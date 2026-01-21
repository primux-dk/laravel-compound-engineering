---
name: laravel-database-patterns
description: This skill should be used when working with Laravel database operations, migrations, transactions, and Eloquent optimization. It applies when creating migrations, implementing transactions, optimizing queries, preventing N+1 problems, or implementing database locking patterns. Triggers on database, migration, transaction, Eloquent, query optimization, or N+1 mentions.
---

<objective>
Provide comprehensive patterns for database operations in Laravel, including migrations, transactions, Eloquent best practices, and query optimization. Focuses on performance, data integrity, and Laravel conventions.
</objective>

<essential_principles>
## Core Philosophy

**Database operations must be safe, performant, and reversible.**

- **Migrations are immutable** - Never modify migrations once run in production
- **Transactions for integrity** - Wrap related operations to ensure all-or-nothing
- **Eager loading by default** - Prevent N+1 queries with `with()` and `withCount()`
- **Index strategically** - Index foreign keys and WHERE clause columns

**Taylor's approach to databases:**
- Eloquent over raw queries (unless performance requires it)
- Models with rich scopes and relationships
- Factories for test data
- Database queues over Redis (Laravel 11+)
</essential_principles>

<intake>
What are you working with?

1. **Migrations** - Schema changes, column types, indexes, naming conventions
2. **Transactions** - Wrapping operations, retries, savepoints, locking
3. **Eloquent** - Scopes, accessors, events, relationships, optimization
4. **Query Optimization** - N+1 prevention, eager loading, chunking, caching
5. **Code Review** - Review database code against patterns

**Specify a number or describe your task.**
</intake>

<routing>
| Response | Reference to Read |
|----------|-------------------|
| 1, "migration", "schema" | [migrations.md](./references/migrations.md) |
| 2, "transaction", "lock" | [transactions.md](./references/transactions.md) |
| 3, "eloquent", "model", "scope" | [eloquent.md](./references/eloquent.md) |
| 4, "optimization", "N+1", "performance" | [eloquent.md](./references/eloquent.md) |
| 5, "review" | Read all references, then review code |

**After reading relevant references, apply patterns to the user's code.**
</routing>

<quick_reference>
## Migration Structure

```php
return new class extends Migration
{
    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->timestamps();

            $table->index('email');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('users');
    }
};
```

## Transaction Wrapper

```php
use Illuminate\Support\Facades\DB;

return DB::transaction(function () use ($data) {
    $user = User::create($data);
    $user->profile()->create(['bio' => '']);
    return $user;
});
```

## Eager Loading (Prevent N+1)

```php
// Bad - N+1 queries
$users = User::all();
foreach ($users as $user) {
    echo $user->posts->count(); // Query per user!
}

// Good - Single query for relations
$users = User::with('posts')->get();

// Better - Count without loading
$users = User::withCount('posts')->get();
foreach ($users as $user) {
    echo $user->posts_count;
}
```

## Model Scopes

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
}

// Usage: User::active()->byRole('admin')->get();
```

## Common Column Types

```php
$table->id();                        // Auto-increment BIGINT
$table->string('name', 100);         // VARCHAR(100)
$table->text('description');         // TEXT
$table->decimal('price', 8, 2);      // DECIMAL(8,2)
$table->boolean('is_active');        // BOOLEAN
$table->json('settings');            // JSON
$table->timestamp('verified_at');    // TIMESTAMP nullable
$table->timestamps();                // created_at, updated_at
$table->foreignId('user_id')->constrained()->cascadeOnDelete();
```

## Naming Conventions

| Component | Convention | Example |
|-----------|------------|---------|
| Tables | Plural snake_case | `users`, `team_members` |
| Pivot tables | Alphabetical singular | `role_user`, `post_tag` |
| Foreign keys | `{model}_id` | `user_id`, `team_id` |
| Migrations | `YYYY_MM_DD_HHMMSS_action_table` | `2025_01_30_create_users_table` |
</quick_reference>

<reference_index>
## Domain Knowledge

All detailed patterns in `references/`:

| File | Topics |
|------|--------|
| [migrations.md](./references/migrations.md) | Schema, column types, indexes, naming, seeders, factories |
| [transactions.md](./references/transactions.md) | Basic transactions, manual control, retries, locking |
| [eloquent.md](./references/eloquent.md) | Scopes, accessors, events, optimization, N+1 prevention |
</reference_index>

<success_criteria>
Database code follows patterns when:
- Migrations are reversible with proper `down()` methods
- Foreign keys and queried columns are indexed
- Related operations wrapped in transactions
- No N+1 queries (eager loading used)
- Factories exist for all models
- Scopes used for reusable query logic
- Select only needed columns for performance
- Raw queries use parameterized bindings
</success_criteria>
