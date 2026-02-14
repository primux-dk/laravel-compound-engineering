# Primux Team Conventions

Team-specific conventions that build on Taylor Otwell's Laravel philosophy. These are strict rules — violations should be flagged in review.

## Code Style

### No Comments
**NO COMMENTS** in code unless explicitly requested. Let code be self-documenting through clear naming. Exception: PHPDoc blocks for complex methods with array shapes.

### Happy Path Last
Handle error conditions first, success case last. Avoid `else` — use early returns.

```php
// Correct - happy path last
if (! $user) {
    return null;
}

if (! $user->isActive()) {
    return null;
}

// Process active user...

// Wrong - nested conditions
if ($user) {
    if ($user->isActive()) {
        // Process...
    }
}
```

### PHP 8.4 Style
- **Constructor property promotion**: `public function __construct(public GitHub $github) {}`
- **Short nullable**: `?string` not `string|null`
- **Explicit return types** on all methods including `void`
- **String interpolation** over concatenation: `"Hello {$name}"` not `'Hello ' . $name`
- **Curly braces always** on control structures, even single-line
- **PSR-12** coding standards
- **No empty constructors** with zero parameters

### Ternary Operators
Short ternaries on one line, multi-line when complex:

```php
$name = $isFoo ? 'foo' : 'bar';

$result = $object instanceof Model
    ? $object->name
    : 'A default value';
```

## Architecture

### Base Action Class
All actions extend a base `Action` class with event support:

```php
abstract class Action
{
    protected array $beforeEvents = [];
    protected array $afterEvents = [];

    abstract public function handle();

    public function execute(...$args)
    {
        $this->dispatchBeforeEvents(...$args);
        $result = $this->handle(...$args);
        $this->dispatchAfterEvents($result, ...$args);
        return $result;
    }

    public static function fake(): FakeAction
    {
        return app()->instance(static::class, new FakeAction(static::class));
    }
}
```

### Action Naming
Verb-first, no "Action" suffix:
- `CreateUser`, `PublishPost`, `ProcessPayment`
- NOT `UserCreateAction`, `CreateUserAction`

### DTOs (Simple, No Packages)
Simple readonly classes with `fromRequest()` and `toArray()`:

```php
class CreateUserDTO
{
    public function __construct(
        public readonly string $name,
        public readonly string $email,
        public readonly string $password,
        public readonly ?string $phone = null,
    ) {}

    public static function fromRequest(array $validated): self
    {
        return new self(
            name: $validated['name'],
            email: $validated['email'],
            password: $validated['password'],
            phone: $validated['phone'] ?? null,
        );
    }

    public function toArray(): array
    {
        return [
            'name' => $this->name,
            'email' => $this->email,
            'password' => $this->password,
            'phone' => $this->phone,
        ];
    }
}
```

### Code Boundaries
Controllers and Livewire Components are thin orchestration layers:

1. **Controller path**: FormRequest → Controller → Action → Model
2. **Livewire path**: Form Object → Component → Action → Model

Both delegate ALL business logic to Actions. Choose based on UI needs, not business logic.

### One FormRequest Per Controller
Single FormRequest handles both store and update with method checks:

```php
class UserRequest extends FormRequest
{
    public function rules(): array
    {
        $rules = [
            'name' => ['required', 'string', 'max:255'],
        ];

        if ($this->isMethod('POST')) {
            $rules['email'] = ['required', 'email', 'unique:users'];
        }

        if ($this->isMethod('PUT') || $this->isMethod('PATCH')) {
            $rules['email'] = ['required', 'email', 'unique:users,email,' . $this->route('user')->id];
        }

        return $rules;
    }
}
```

## Enums

All enums must be string-backed with required methods:

```php
enum OrderStatus: string
{
    case Pending = 'pending';
    case Processing = 'processing';
    case Completed = 'completed';

    public function label(): string
    {
        return match ($this) {
            self::Pending => __('Pending'),
            self::Processing => __('Processing'),
            self::Completed => __('Completed'),
        };
    }

    public function color(): string
    {
        return match ($this) {
            self::Pending => 'yellow',
            self::Processing => 'blue',
            self::Completed => 'green',
        };
    }

    public function icon(): string
    {
        return match ($this) {
            self::Pending => 'clock',
            self::Processing => 'arrow-path',
            self::Completed => 'check-circle',
        };
    }
}
```

Rules:
- Cases are **TitleCase** (`Pending`, not `PENDING`)
- Values are **lowercase strings** for database storage
- Always cast in models via `casts()` method
- Required methods: `label()`, `color()`, `icon()` (when applicable)

## Database

### Mass Assignment Unguarded
Models do NOT have `$fillable` or `$guarded`. Mass assignment is unguarded globally in `AppServiceProvider`.

### Migrations: No `down()` Method
Only write `up()` methods. Never include `down()`.

### Prefer `Model::query()` Over `DB::`
Use Eloquent models and relationships. Avoid `DB::` facade for queries.

## Internationalization

All user-facing text uses `__()` helper with string-based translations (not key-based):

```php
__('Welcome to our application')
__('Hello, :name!', ['name' => $user->name])

// In Blade
{{ __('Welcome to our application') }}

// NEVER use @lang directive or key-based translations
```

## Alpine.js — CRITICAL
**Never manually initialize Alpine.js.** Alpine is included and managed by Livewire 3. Never add `import Alpine from 'alpinejs'` or `Alpine.start()` to app.js.

## Formatting & Linting
Run `vendor/bin/pint --dirty` before committing. Do not use `--test` flag.

## Testing
- 100% coverage goal
- Integration/feature tests over unit tests
- Run coverage with: `herd coverage ./vendor/bin/pest --coverage`
- Use Pest v4 syntax: `test()`, `it()`, `expect()`, `describe()`
- Happy path first, then 1-2 critical edge cases

## Git Commits
- `feat:` for new features
- `fix:` for bug fixes
- `refactor:` for code improvements
- `test:` for test additions
- Run lints and tests before committing

## File Naming
- Actions: `{Verb}{ActionPerformed}.php` (e.g., `CreateUser.php`)
- Events: tense-based (`UserRegistering.php`, `UserRegistered.php`)
- Listeners: action + `Listener` suffix (`SendInvitationMailListener.php`)
- Commands: action + `Command` suffix (`PublishScheduledPostsCommand.php`)
- Mailables: purpose + `Mail` suffix (`AccountActivatedMail.php`)
- Enums: descriptive name, no prefix (`OrderStatus.php`)
