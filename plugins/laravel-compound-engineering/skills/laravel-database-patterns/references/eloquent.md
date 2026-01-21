# Laravel Eloquent Patterns

<core_principle>
**Eloquent models are expressive, not bloated.** Models should have rich public APIs with scopes, relationships, and accessors - but delegate heavy business logic to Actions.
</core_principle>

<scopes>
## Query Scopes

**Local Scopes:**
```php
class User extends Model
{
    public function scopeActive(Builder $query): Builder
    {
        return $query->where('is_active', true);
    }

    public function scopeByRole(Builder $query, string $role): Builder
    {
        return $query->where('role', $role);
    }

    public function scopeCreatedBetween(Builder $query, Carbon $start, Carbon $end): Builder
    {
        return $query->whereBetween('created_at', [$start, $end]);
    }

    public function scopeSearch(Builder $query, ?string $term): Builder
    {
        if (!$term) {
            return $query;
        }

        return $query->where(function ($q) use ($term) {
            $q->where('name', 'like', "%{$term}%")
              ->orWhere('email', 'like', "%{$term}%");
        });
    }
}

// Usage - chainable and expressive
$users = User::active()
    ->byRole('admin')
    ->createdBetween(now()->subMonth(), now())
    ->search($request->input('q'))
    ->get();
```

**Global Scopes:**
```php
class User extends Model
{
    protected static function booted(): void
    {
        static::addGlobalScope('active', function (Builder $query) {
            $query->where('is_active', true);
        });
    }
}

// Remove global scope when needed
User::withoutGlobalScope('active')->get();
```
</scopes>

<accessors>
## Accessors & Mutators

**Modern Syntax (Laravel 9+):**
```php
use Illuminate\Database\Eloquent\Casts\Attribute;

class User extends Model
{
    // Get and set transformations
    protected function name(): Attribute
    {
        return Attribute::make(
            get: fn (string $value) => ucfirst($value),
            set: fn (string $value) => strtolower($value)
        );
    }

    // Computed attribute (get only)
    protected function fullName(): Attribute
    {
        return Attribute::make(
            get: fn () => "{$this->first_name} {$this->last_name}"
        );
    }

    // Cached computed attribute
    protected function expensiveCalculation(): Attribute
    {
        return Attribute::make(
            get: fn () => $this->calculateExpensiveValue()
        )->shouldCache();
    }
}
```

**Appending to JSON:**
```php
class User extends Model
{
    protected $appends = ['full_name'];

    protected function fullName(): Attribute
    {
        return Attribute::make(
            get: fn () => "{$this->first_name} {$this->last_name}"
        );
    }
}
```
</accessors>

<casts>
## Attribute Casting

```php
class User extends Model
{
    protected function casts(): array
    {
        return [
            'email_verified_at' => 'datetime',
            'is_active' => 'boolean',
            'settings' => 'array',
            'metadata' => 'collection',
            'birthday' => 'date:Y-m-d',
            'options' => AsArrayObject::class,
            'address' => AddressCast::class,
        ];
    }
}
```

**Custom Cast:**
```php
class AddressCast implements CastsAttributes
{
    public function get(Model $model, string $key, mixed $value, array $attributes): Address
    {
        return new Address(json_decode($value, true));
    }

    public function set(Model $model, string $key, mixed $value, array $attributes): string
    {
        return json_encode($value->toArray());
    }
}
```
</casts>

<events>
## Model Events

```php
class User extends Model
{
    protected static function booted(): void
    {
        // Before create
        static::creating(function (User $user) {
            $user->uuid = Str::uuid();
        });

        // After create
        static::created(function (User $user) {
            $user->profile()->create();
        });

        // Before delete
        static::deleting(function (User $user) {
            $user->posts()->delete();
        });
    }
}
```

**Available Events:**
- `retrieved` - After model retrieved from database
- `creating` / `created` - Before/after insert
- `updating` / `updated` - Before/after update
- `saving` / `saved` - Before/after insert or update
- `deleting` / `deleted` - Before/after delete
- `restoring` / `restored` - Soft delete restore
- `replicating` - When model is replicated
</events>

<relationships>
## Relationship Best Practices

```php
class User extends Model
{
    // HasMany - plural name
    public function posts(): HasMany
    {
        return $this->hasMany(Post::class);
    }

    // BelongsTo - singular name
    public function company(): BelongsTo
    {
        return $this->belongsTo(Company::class);
    }

    // HasOne - singular name
    public function profile(): HasOne
    {
        return $this->hasOne(Profile::class);
    }

    // BelongsToMany - plural name
    public function roles(): BelongsToMany
    {
        return $this->belongsToMany(Role::class)
            ->withTimestamps()
            ->withPivot('expires_at');
    }

    // HasManyThrough
    public function postComments(): HasManyThrough
    {
        return $this->hasManyThrough(Comment::class, Post::class);
    }

    // Constrained relationship
    public function publishedPosts(): HasMany
    {
        return $this->hasMany(Post::class)->published();
    }
}
```
</relationships>

<eager_loading>
## Eager Loading (Prevent N+1)

**The N+1 Problem:**
```php
// Bad - N+1 queries (1 for users + N for posts)
$users = User::all();
foreach ($users as $user) {
    echo $user->posts->count(); // Query per user!
}
```

**Solutions:**

```php
// with() - Load relations
$users = User::with('posts')->get();
foreach ($users as $user) {
    echo $user->posts->count(); // No additional queries
}

// withCount() - Count without loading
$users = User::withCount('posts')->get();
foreach ($users as $user) {
    echo $user->posts_count; // Single query total
}

// Nested eager loading
$users = User::with([
    'posts' => function ($query) {
        $query->published()->latest();
    },
    'posts.comments',
    'profile',
])->get();

// Constrained eager loading
$users = User::with('posts:id,user_id,title')->get();
```

**Default Eager Loading:**
```php
class User extends Model
{
    protected $with = ['profile'];
}
```

**Lazy Eager Loading:**
```php
$users = User::all();
$users->load('posts', 'comments');
```
</eager_loading>

<query_optimization>
## Query Optimization

**Select Only What You Need:**
```php
// Bad - Fetches all columns
$users = User::all();

// Good - Select specific columns
$users = User::select('id', 'name', 'email')->get();

// With relationships
$users = User::with('posts:id,user_id,title')
    ->select('id', 'name')
    ->get();
```

**Chunking Large Datasets:**
```php
// Process in chunks (memory efficient)
User::chunk(100, function ($users) {
    foreach ($users as $user) {
        // Process user
    }
});

// Chunk by ID (safer for updates)
User::chunkById(100, function ($users) {
    foreach ($users as $user) {
        $user->update(['processed' => true]);
    }
});

// Cursor for maximum memory efficiency
foreach (User::cursor() as $user) {
    // Process user - only one model in memory at a time
}
```

**Avoiding Queries in Loops:**
```php
// Bad - Query in loop
foreach ($userIds as $id) {
    $user = User::find($id);
    // Process
}

// Good - Single query
$users = User::whereIn('id', $userIds)->get()->keyBy('id');
foreach ($userIds as $id) {
    $user = $users[$id];
    // Process
}
```
</query_optimization>

<caching>
## Query Caching

```php
// Cache expensive queries
$users = Cache::remember('active-users', 3600, function () {
    return User::active()->with('profile')->get();
});

// Cache with tags (Redis/Memcached)
$user = Cache::tags(['users'])->remember("user.{$id}", 3600, function () use ($id) {
    return User::with('posts')->find($id);
});

// Invalidate on update
protected static function booted(): void
{
    static::saved(function (User $user) {
        Cache::tags(['users'])->flush();
    });
}
```
</caching>

<raw_queries>
## Raw Queries (When Necessary)

**Use raw queries for:**
- Complex aggregations
- Database-specific features
- Performance-critical operations

```php
// Parameterized query (safe)
$users = DB::select('SELECT * FROM users WHERE role = ?', ['admin']);

// Named bindings
$users = DB::select(
    'SELECT * FROM users WHERE role = :role',
    ['role' => 'admin']
);

// Raw expressions in Eloquent
$users = User::select([
    'name',
    DB::raw('COUNT(posts.id) as post_count'),
])
->leftJoin('posts', 'users.id', '=', 'posts.user_id')
->groupBy('users.id', 'users.name')
->get();

// NEVER do this (SQL injection risk)
$users = DB::select("SELECT * FROM users WHERE role = '$role'"); // ❌
```
</raw_queries>

<anti_patterns>
## Anti-Patterns to Avoid

**❌ N+1 Queries:**
```php
// Bad
$users = User::all();
foreach ($users as $user) {
    echo $user->posts->count();
}

// Good
$users = User::withCount('posts')->get();
```

**❌ Loading Unnecessary Data:**
```php
// Bad
$user = User::with(['posts', 'comments', 'likes'])->first();

// Good - Load only what you need
$user = User::select('id', 'name')->first();
```

**❌ Queries Inside Accessors:**
```php
// Bad - Query every time attribute accessed
protected function postsCount(): Attribute
{
    return Attribute::make(
        get: fn () => $this->posts()->count()
    );
}

// Good - Use withCount() at query time
User::withCount('posts')->get();
```

**❌ Business Logic in Models:**
```php
// Bad - Model doing too much
class User extends Model
{
    public function createSubscription($plan)
    {
        $this->subscriptions()->create([...]);
        $this->notify(new SubscriptionCreated());
        event(new UserSubscribed($this));
        // Send to Stripe, etc.
    }
}

// Good - Delegate to Action
class CreateSubscription
{
    public function handle(User $user, Plan $plan): Subscription
    {
        return DB::transaction(function () use ($user, $plan) {
            $subscription = $user->subscriptions()->create([...]);
            event(new UserSubscribed($user, $subscription));
            return $subscription;
        });
    }
}
```
</anti_patterns>

<performance_monitoring>
## Performance Monitoring

```php
// Enable query log in development
DB::enableQueryLog();

// ... run queries

dd(DB::getQueryLog());

// Detect N+1 in development
// Add to AppServiceProvider
if (app()->environment('local')) {
    DB::listen(function ($query) {
        Log::debug($query->sql, $query->bindings);
    });
}
```

**Use Laravel Debugbar or Telescope for:**
- Query count per request
- Slow query detection
- Duplicate query detection
- N+1 detection
</performance_monitoring>

<testing>
## Testing Eloquent

```php
use Illuminate\Foundation\Testing\RefreshDatabase;

uses(RefreshDatabase::class);

it('returns only active users', function () {
    User::factory()->count(3)->create(['is_active' => true]);
    User::factory()->count(2)->create(['is_active' => false]);

    $users = User::active()->get();

    expect($users)->toHaveCount(3);
});

it('eager loads relationships', function () {
    $user = User::factory()->has(Post::factory()->count(3))->create();

    DB::enableQueryLog();

    $loaded = User::with('posts')->find($user->id);
    $loaded->posts->count();

    // Should be exactly 2 queries (users + posts)
    expect(DB::getQueryLog())->toHaveCount(2);
});

it('prevents N+1 queries', function () {
    User::factory()->count(5)->has(Post::factory()->count(2))->create();

    DB::enableQueryLog();

    User::withCount('posts')->get()->each->posts_count;

    // Should be exactly 1 query
    expect(DB::getQueryLog())->toHaveCount(1);
});
```
</testing>
