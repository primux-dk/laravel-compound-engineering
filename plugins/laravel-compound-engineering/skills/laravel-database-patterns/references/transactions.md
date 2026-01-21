# Laravel Transaction Patterns

<core_principle>
**Transactions ensure data integrity** - Either all operations succeed, or none do. Use transactions whenever multiple related database operations must succeed together.
</core_principle>

<basic_transaction>
## Basic Transaction

```php
use Illuminate\Support\Facades\DB;

// Simple closure-based transaction
DB::transaction(function () use ($data) {
    $user = User::create($data);
    $profile = Profile::create(['user_id' => $user->id]);
    return $user;
});

// With return value
$user = DB::transaction(function () use ($data) {
    $user = User::create($data);
    $user->assignRole('customer');
    return $user;
});
```

**When to use:**
- Creating related records (user + profile)
- Financial operations (debit + credit)
- Any operation that must be atomic
</basic_transaction>

<manual_control>
## Manual Transaction Control

```php
DB::beginTransaction();

try {
    $order = Order::create($orderData);
    $payment = Payment::process($paymentData);

    if ($payment->failed()) {
        throw new PaymentException('Payment failed');
    }

    DB::commit();
    return $order;
} catch (\Exception $e) {
    DB::rollBack();
    throw $e;
}
```

**Use manual control when:**
- Need to check intermediate results
- Complex branching logic
- Multiple potential failure points
</manual_control>

<retries>
## Transaction with Retries

```php
// Retry up to 5 times on deadlock
DB::transaction(function () use ($data) {
    User::lockForUpdate()->find($data['id'])->update($data);
}, attempts: 5);
```

**Deadlocks occur when:**
- Two transactions try to lock same rows in different order
- High concurrency on same records
- Long-running transactions

**Retry guidelines:**
- Default is 1 attempt (no retries)
- 3-5 retries usually sufficient
- Add exponential backoff for critical operations
</retries>

<savepoints>
## Nested Transactions (Savepoints)

```php
DB::transaction(function () {
    $user = User::create($userData);

    try {
        DB::transaction(function () use ($user) {
            // This creates a savepoint
            $user->teams()->attach($teamId);
            $user->notifyTeamJoined();
        });
    } catch (\Exception $e) {
        // Only the inner transaction rolls back
        // $user still created
        Log::warning('Team notification failed', ['error' => $e->getMessage()]);
    }
});
```

**Savepoint behavior:**
- Inner transaction failure only rolls back to savepoint
- Outer transaction continues
- Useful for optional operations that shouldn't fail the whole transaction
</savepoints>

<pessimistic_locking>
## Pessimistic Locking

**Lock for Update (Exclusive Lock):**
```php
DB::transaction(function () {
    // Locks the row - other transactions wait
    $user = User::lockForUpdate()->find(1);

    $user->balance -= 100;
    $user->save();
});
```

**Shared Lock (Read Lock):**
```php
DB::transaction(function () {
    // Prevents updates but allows reads
    $user = User::sharedLock()->find(1);

    // Read operations
    $balance = $user->balance;
});
```

**When to use:**
- `lockForUpdate()` - When you'll modify the record
- `sharedLock()` - When you need consistent read during transaction
</pessimistic_locking>

<optimistic_locking>
## Optimistic Locking

```php
class Order extends Model
{
    public function updateWithVersionCheck(array $data): bool
    {
        $currentVersion = $this->version;

        $updated = DB::table($this->getTable())
            ->where('id', $this->id)
            ->where('version', $currentVersion)
            ->update([
                ...$data,
                'version' => $currentVersion + 1,
            ]);

        if (!$updated) {
            throw new OptimisticLockException(
                'Record was modified by another process'
            );
        }

        return true;
    }
}
```

**Migration for version column:**
```php
$table->unsignedInteger('version')->default(1);
```

**Optimistic vs Pessimistic:**
- **Optimistic**: Check version at update time (better for low contention)
- **Pessimistic**: Lock row immediately (better for high contention)
</optimistic_locking>

<patterns>
## Common Transaction Patterns

**Create with Related Records:**
```php
public function handle(array $data): User
{
    return DB::transaction(function () use ($data) {
        $user = User::create([
            'name' => $data['name'],
            'email' => $data['email'],
        ]);

        $user->profile()->create([
            'bio' => $data['bio'] ?? '',
        ]);

        $user->assignRole('customer');

        return $user;
    });
}
```

**Transfer Between Accounts:**
```php
public function transfer(Account $from, Account $to, int $amount): void
{
    DB::transaction(function () use ($from, $to, $amount) {
        $from->lockForUpdate();
        $to->lockForUpdate();

        if ($from->balance < $amount) {
            throw new InsufficientFundsException();
        }

        $from->decrement('balance', $amount);
        $to->increment('balance', $amount);

        Transaction::create([
            'from_account_id' => $from->id,
            'to_account_id' => $to->id,
            'amount' => $amount,
        ]);
    });
}
```

**Bulk Operations:**
```php
DB::transaction(function () use ($orders) {
    foreach ($orders as $orderData) {
        $order = Order::create($orderData);
        $order->items()->createMany($orderData['items']);
    }
});
```
</patterns>

<anti_patterns>
## Anti-Patterns to Avoid

**❌ No Transaction for Related Operations:**
```php
// Bad - Partial failure possible
$order = Order::create($orderData);
$payment = Payment::create($paymentData);
OrderItem::insert($items);
```

**✅ Wrapped in Transaction:**
```php
DB::transaction(function () use ($orderData, $paymentData, $items) {
    $order = Order::create($orderData);
    $payment = Payment::create($paymentData);
    OrderItem::insert($items);
});
```

**❌ Long-Running Transactions:**
```php
// Bad - Holds locks too long
DB::transaction(function () use ($users) {
    foreach ($users as $user) {
        // API call inside transaction!
        $result = Http::post('api/notify', $user->toArray());
        $user->update(['notified' => true]);
    }
});
```

**✅ External Calls Outside Transaction:**
```php
// Good - Only database operations in transaction
$usersToNotify = DB::transaction(function () use ($users) {
    $updated = [];
    foreach ($users as $user) {
        $user->update(['notified_at' => now()]);
        $updated[] = $user;
    }
    return $updated;
});

// External calls after transaction
foreach ($usersToNotify as $user) {
    Http::post('api/notify', $user->toArray());
}
```
</anti_patterns>

<testing>
## Testing Transactions

```php
use Illuminate\Foundation\Testing\RefreshDatabase;

uses(RefreshDatabase::class);

it('rolls back on failure', function () {
    expect(fn () => DB::transaction(function () {
        User::create(['name' => 'Test', 'email' => 'test@example.com']);
        throw new \Exception('Simulated failure');
    }))->toThrow(\Exception::class);

    $this->assertDatabaseMissing('users', ['email' => 'test@example.com']);
});

it('commits on success', function () {
    $user = DB::transaction(function () {
        return User::create(['name' => 'Test', 'email' => 'test@example.com']);
    });

    $this->assertDatabaseHas('users', ['email' => 'test@example.com']);
});

it('handles concurrent access with locking', function () {
    $user = User::factory()->create(['balance' => 100]);

    // Simulate concurrent access
    $process1 = function () use ($user) {
        DB::transaction(function () use ($user) {
            $u = User::lockForUpdate()->find($user->id);
            $u->decrement('balance', 50);
        });
    };

    $process2 = function () use ($user) {
        DB::transaction(function () use ($user) {
            $u = User::lockForUpdate()->find($user->id);
            $u->decrement('balance', 30);
        });
    };

    $process1();
    $process2();

    expect($user->fresh()->balance)->toBe(20);
});
```
</testing>
