# Laravel Migration Patterns

<naming>
## Migration Naming

```
YYYY_MM_DD_HHMMSS_descriptive_name.php

Examples:
2025_01_30_000001_create_users_table.php
2025_01_30_000002_add_status_to_orders_table.php
2025_01_30_000003_create_role_user_pivot_table.php
```

**Naming patterns:**
- `create_{table}_table` - New table
- `add_{column}_to_{table}_table` - Add column
- `remove_{column}_from_{table}_table` - Remove column
- `modify_{column}_in_{table}_table` - Change column type
- `create_{table1}_{table2}_table` - Pivot table (alphabetical)
</naming>

<structure>
## Migration Structure

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password');
            $table->boolean('is_active')->default(true);
            $table->rememberToken();
            $table->timestamps();

            // Indexes
            $table->index('email');
            $table->index('created_at');
        });
    }

};
```

**Best Practices:**
1. **Only write `up()` methods** — never include `down()`
2. Add indexes for foreign keys and commonly queried columns
3. Use appropriate column types
4. Set sensible defaults
5. **Never modify migrations once run in production**
</structure>

<column_types>
## Column Types Reference

```php
// Primary Keys
$table->id();                        // Auto-increment BIGINT
$table->uuid('id')->primary();       // UUID as primary key
$table->ulid('id')->primary();       // ULID as primary key

// Strings
$table->string('name');              // VARCHAR(255) default
$table->string('name', 100);         // VARCHAR(100)
$table->char('code', 2);             // CHAR(2)
$table->text('description');         // TEXT
$table->mediumText('content');       // MEDIUMTEXT
$table->longText('body');            // LONGTEXT

// Numbers
$table->integer('age');              // INTEGER
$table->tinyInteger('level');        // TINYINT
$table->smallInteger('votes');       // SMALLINT
$table->mediumInteger('count');      // MEDIUMINT
$table->bigInteger('views');         // BIGINT
$table->unsignedBigInteger('amount');// UNSIGNED BIGINT

// Decimals
$table->decimal('price', 8, 2);      // DECIMAL(8,2)
$table->float('rating', 8, 2);       // FLOAT(8,2)
$table->double('precision', 8, 2);   // DOUBLE(8,2)

// Dates & Times
$table->date('birth_date');          // DATE
$table->time('start_time');          // TIME
$table->datetime('published_at');    // DATETIME
$table->timestamp('verified_at');    // TIMESTAMP
$table->timestamps();                // created_at, updated_at
$table->softDeletes();               // deleted_at

// Booleans
$table->boolean('is_active');        // BOOLEAN

// JSON
$table->json('settings');            // JSON
$table->jsonb('metadata');           // JSONB (PostgreSQL)

// Binary
$table->binary('data');              // BLOB

// Enums
$table->enum('status', ['draft', 'published', 'archived']);
```
</column_types>

<indexes>
## Indexing Strategies

```php
Schema::table('users', function (Blueprint $table) {
    // Single column index
    $table->index('email');

    // Named index
    $table->index('email', 'users_email_idx');

    // Composite index (order matters!)
    $table->index(['last_name', 'first_name']);

    // Unique index
    $table->unique('email');

    // Full-text index (MySQL)
    $table->fullText('bio');

    // Spatial index
    $table->spatialIndex('location');
});
```

**Indexing Guidelines:**
- Always index foreign keys
- Index columns used in WHERE clauses
- Index columns used in ORDER BY
- Composite indexes: put most selective column first
- Don't over-index (slows writes)
</indexes>

<foreign_keys>
## Foreign Key Constraints

```php
// Modern syntax (Laravel 7+)
$table->foreignId('user_id')->constrained()->cascadeOnDelete();

// Expanded form
$table->foreignId('user_id')
    ->constrained('users')
    ->onUpdate('cascade')
    ->onDelete('cascade');

// Nullable foreign key
$table->foreignId('team_id')->nullable()->constrained()->nullOnDelete();

// Legacy syntax
$table->unsignedBigInteger('user_id');
$table->foreign('user_id')->references('id')->on('users');
```

**Cascade Options:**
- `cascadeOnDelete()` - Delete related records
- `cascadeOnUpdate()` - Update foreign key when parent changes
- `nullOnDelete()` - Set to NULL when parent deleted
- `restrictOnDelete()` - Prevent parent deletion if children exist
</foreign_keys>

<modifying>
## Modifying Columns

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

// Add column
Schema::table('users', function (Blueprint $table) {
    $table->string('phone')->nullable()->after('email');
});

// Modify column (requires doctrine/dbal)
Schema::table('users', function (Blueprint $table) {
    $table->string('name', 50)->change();
});

// Rename column
Schema::table('users', function (Blueprint $table) {
    $table->renameColumn('name', 'full_name');
});

// Drop column
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn('phone');
    // Or multiple: $table->dropColumn(['phone', 'fax']);
});

// Drop index
Schema::table('users', function (Blueprint $table) {
    $table->dropIndex('users_email_index');
    $table->dropUnique('users_email_unique');
    $table->dropForeign('users_team_id_foreign');
});
```
</modifying>

<seeders>
## Database Seeding

**Seeder Structure:**
```php
namespace Database\Seeders;

use App\Models\User;
use Illuminate\Database\Seeder;

class UserSeeder extends Seeder
{
    public function run(): void
    {
        // Create admin user
        User::create([
            'name' => 'Admin',
            'email' => 'admin@example.com',
            'password' => bcrypt('password'),
            'is_admin' => true,
        ]);

        // Create test users with factory
        User::factory()->count(10)->create();
    }
}
```

**DatabaseSeeder:**
```php
class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        $this->call([
            UserSeeder::class,
            TeamSeeder::class,
            PostSeeder::class,
        ]);
    }
}
```
</seeders>

<factories>
## Model Factories

```php
namespace Database\Factories;

use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Str;

class UserFactory extends Factory
{
    protected $model = User::class;

    public function definition(): array
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'email_verified_at' => now(),
            'password' => bcrypt('password'),
            'remember_token' => Str::random(10),
        ];
    }

    // State modifications
    public function unverified(): static
    {
        return $this->state(fn () => [
            'email_verified_at' => null,
        ]);
    }

    public function admin(): static
    {
        return $this->state(fn () => [
            'is_admin' => true,
        ]);
    }
}
```

**Factory Usage:**
```php
// Single model
$user = User::factory()->create();

// Multiple models
$users = User::factory()->count(10)->create();

// With states
$admin = User::factory()->admin()->create();

// With relationships
$user = User::factory()
    ->has(Post::factory()->count(3))
    ->create();

// Override attributes
$user = User::factory()->create([
    'name' => 'Custom Name',
]);
```
</factories>

<testing>
## Testing Migrations

```php
use Illuminate\Foundation\Testing\RefreshDatabase;

uses(RefreshDatabase::class);

it('creates users table with correct columns', function () {
    expect(Schema::hasTable('users'))->toBeTrue();
    expect(Schema::hasColumn('users', 'email'))->toBeTrue();
    expect(Schema::hasColumn('users', 'name'))->toBeTrue();
});

it('enforces unique email constraint', function () {
    User::factory()->create(['email' => 'test@example.com']);

    expect(fn () => User::factory()->create(['email' => 'test@example.com']))
        ->toThrow(QueryException::class);
});
```
</testing>
