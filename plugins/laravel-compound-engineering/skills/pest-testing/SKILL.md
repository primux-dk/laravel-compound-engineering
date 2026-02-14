---
name: pest-testing
description: This skill should be used when writing tests with Pest 4 for Laravel applications. It applies when creating unit, feature, or browser tests, working with assertions, datasets, mocking, architecture tests, or debugging test failures. Triggers on test, Pest, TDD, expects, assertion, coverage, browser test, or smoke test mentions.
---

<objective>
Provide comprehensive patterns for testing Laravel applications with Pest 4, including feature tests, browser testing, architecture tests, datasets, and mocking. Follows Taylor Otwell's testing philosophy: high-level feature tests over unit tests, PEST over PHPUnit.
</objective>

<essential_principles>
## Core Philosophy

**Test behavior, not implementation.**

- **Feature tests first** — Test from the user's perspective
- **Pest over PHPUnit** — Expressive, minimal syntax
- **Factories for data** — Never hardcode test data
- **Never delete tests** — Tests are core application code
- **Specific assertions** — Use `assertSuccessful()` not `assertStatus(200)`

**Taylor's approach to testing:**
- High-level feature tests that cover real user flows
- Use factories extensively
- Test the happy path first, then 1-2 critical edge cases
- Keep tests readable — they're documentation
- **100% coverage goal** — run coverage with: `herd coverage ./vendor/bin/pest --coverage`
- Use Pest v4 syntax: `test()`, `it()`, `expect()`, `describe()`
</essential_principles>

<intake>
What are you testing?

1. **Feature Tests** — HTTP requests, controllers, API endpoints
2. **Browser Tests** — Real browser interactions, forms, navigation (Pest 4)
3. **Architecture Tests** — Code conventions, naming, structure enforcement
4. **Datasets & Mocking** — Parameterized tests, fakes, mocks
5. **Code Review** — Review test code against patterns

**Specify a number or describe your task.**
</intake>

<quick_reference>
## Creating Tests

```bash
# Create a Pest test
php artisan make:test UserRegistrationTest --pest

# Create a unit test
php artisan make:test CalculatorTest --pest --unit
```

## Running Tests

**IMPORTANT:** Always prefix with `APP_ENV=testing` to ensure the correct environment. Claude Code and other tools may set their own `APP_ENV` which overrides `.env.testing`. The explicit prefix guarantees tests run against the testing database and config.

### Workflow: Start Fast, Narrow Down on Failure

**Step 1 — Run everything in parallel:**
```bash
APP_ENV=testing ./vendor/bin/pest --parallel --compact
```

**Step 2 — If failures, stop on first error to isolate:**
```bash
APP_ENV=testing ./vendor/bin/pest --bail
```

**Step 3 — Re-run only the previously failed tests:**
```bash
APP_ENV=testing ./vendor/bin/pest --retry
```

**Step 4 — Filter to a specific test or file:**
```bash
APP_ENV=testing ./vendor/bin/pest --filter="test description"
APP_ENV=testing ./vendor/bin/pest tests/Feature/UserTest.php
```

### CLI Flags Reference

| Flag | Purpose |
|------|---------|
| `--parallel` | Run tests concurrently across CPU cores (fastest full run) |
| `--parallel --processes=8` | Parallel with explicit process count |
| `--bail` | Stop on first failure or error (isolate problems fast) |
| `--retry` | Re-run previously failed tests first |
| `--dirty` | Only run tests with uncommitted git changes |
| `--filter="name"` | Run tests matching a regex pattern |
| `--group=integration` | Run tests in a specific group |
| `--exclude-group=browser` | Skip tests in a specific group |
| `--compact` | Minimal output — only show failures |
| `--profile` | Show slowest tests for optimization |
| `--coverage` | Generate code coverage report (requires Xdebug/PCOV) |
| `--todo` | List all `todo()` tests |

### Common Combinations

```bash
# Fast development loop — only changed tests
APP_ENV=testing ./vendor/bin/pest --dirty

# Full CI run with parallel and compact output
APP_ENV=testing ./vendor/bin/pest --parallel --compact

# Debug a specific failure
APP_ENV=testing ./vendor/bin/pest --bail --filter="creates user"

# Find slow tests to optimize
APP_ENV=testing ./vendor/bin/pest --profile

# Coverage report (with herd)
APP_ENV=testing herd coverage ./vendor/bin/pest --coverage
```

## Basic Test Structure

```php
it('registers a new user', function () {
    $response = $this->postJson('/api/register', [
        'name' => 'John Doe',
        'email' => 'john@example.com',
        'password' => 'password123',
        'password_confirmation' => 'password123',
    ]);

    $response->assertSuccessful();
    $this->assertDatabaseHas('users', ['email' => 'john@example.com']);
});
```

## Specific Assertions

Always prefer specific assertions over generic `assertStatus()`:

| Use | Instead of |
|-----|------------|
| `assertSuccessful()` | `assertStatus(200)` |
| `assertNotFound()` | `assertStatus(404)` |
| `assertForbidden()` | `assertStatus(403)` |
| `assertUnauthorized()` | `assertStatus(401)` |
| `assertUnprocessable()` | `assertStatus(422)` |

## Expect API

```php
expect($user->name)->toBe('John');
expect($users)->toHaveCount(3);
expect($user->email)->toContain('@');
expect($user->is_admin)->toBeFalse();
expect($value)->toBeNull();
expect($collection)->each->toBeInstanceOf(User::class);
```

## Factories

```php
it('shows user profile', function () {
    $user = User::factory()->create();

    $this->actingAs($user)
        ->get("/users/{$user->id}")
        ->assertSuccessful()
        ->assertSee($user->name);
});

it('lists active users', function () {
    User::factory()->count(3)->create(['is_active' => true]);
    User::factory()->count(2)->create(['is_active' => false]);

    $this->getJson('/api/users?active=true')
        ->assertSuccessful()
        ->assertJsonCount(3, 'data');
});
```

## Mocking

```php
use function Pest\Laravel\mock;

it('sends welcome email on registration', function () {
    Mail::fake();

    $this->postJson('/api/register', [
        'name' => 'John',
        'email' => 'john@example.com',
        'password' => 'password123',
        'password_confirmation' => 'password123',
    ]);

    Mail::assertSent(WelcomeEmail::class);
});

it('dispatches job after order', function () {
    Queue::fake();

    // ... create order ...

    Queue::assertPushed(ProcessOrder::class);
});
```

## Datasets

Use datasets for parameterized tests:

```php
it('validates required fields', function (string $field) {
    $data = User::factory()->raw([$field => '']);

    $this->postJson('/api/users', $data)
        ->assertUnprocessable()
        ->assertJsonValidationErrorFor($field);
})->with(['name', 'email', 'password']);

it('processes payment methods', function (string $method, int $expectedFee) {
    $order = Order::factory()->create(['payment_method' => $method]);

    expect($order->processing_fee)->toBe($expectedFee);
})->with([
    'credit card' => ['credit_card', 250],
    'bank transfer' => ['bank_transfer', 0],
    'paypal' => ['paypal', 150],
]);
```

## Livewire Component Tests

```php
use Livewire\Livewire;

it('creates user through component', function () {
    Livewire::test(CreateUser::class)
        ->set('form.name', 'John Doe')
        ->set('form.email', 'john@example.com')
        ->call('save')
        ->assertDispatched('user-created')
        ->assertRedirect('/users');

    $this->assertDatabaseHas('users', ['email' => 'john@example.com']);
});

it('validates form in real-time', function () {
    Livewire::test(CreateUser::class)
        ->set('form.email', 'invalid')
        ->assertHasErrors(['form.email' => 'email']);
});
```
</quick_reference>

<browser_testing>
## Browser Testing (Pest 4)

Browser tests run in real browsers for full integration testing. They live in `tests/Browser/`.

```php
it('completes checkout flow', function () {
    $user = User::factory()->create();

    $this->actingAs($user);

    $page = visit('/products');

    $page->assertSee('Products')
        ->assertNoJavaScriptErrors()
        ->click('Add to Cart')
        ->click('Checkout')
        ->fill('card_number', '4242424242424242')
        ->fill('expiry', '12/28')
        ->fill('cvc', '123')
        ->click('Pay Now')
        ->assertSee('Order confirmed!');
});
```

**Key browser test capabilities:**
- Click, type, scroll, select, submit, drag-and-drop, touch gestures
- Test across browsers (Chrome, Firefox, Safari)
- Test viewport sizes (mobile, tablet, desktop)
- Switch color schemes (light/dark mode)
- Take screenshots for debugging

## Smoke Testing

Quickly validate multiple pages render without errors:

```php
$pages = visit(['/', '/about', '/contact', '/pricing']);

$pages->assertNoJavaScriptErrors()->assertNoConsoleLogs();
```

## Visual Regression Testing

Capture and compare screenshots to detect unintended visual changes.
</browser_testing>

<architecture_tests>
## Architecture Tests

Enforce code conventions and project structure with Pest architecture tests:

```php
arch('controllers have correct suffix')
    ->expect('App\Http\Controllers')
    ->toExtendNothing()
    ->toHaveSuffix('Controller');

arch('actions are invokable or have handle method')
    ->expect('App\Actions')
    ->toHaveMethod('handle');

arch('models extend base model')
    ->expect('App\Models')
    ->toExtend('Illuminate\Database\Eloquent\Model');

arch('no debugging statements')
    ->expect(['dd', 'dump', 'ray', 'var_dump'])
    ->not->toBeUsed();

arch('strict types in all files')
    ->expect('App')
    ->toUseStrictTypes();
```
</architecture_tests>

<best_practices>
## Best Practices

1. **One assertion concern per test** — Test one behavior, use multiple assertions for that behavior
2. **Descriptive test names** — `it('prevents duplicate email registration')` not `it('test1')`
3. **Use factories** — Never hardcode IDs, emails, or test data
4. **Assert database state** — Use `assertDatabaseHas`/`assertDatabaseMissing`
5. **Fake externals** — `Mail::fake()`, `Queue::fake()`, `Notification::fake()`
6. **Run filtered tests first** — `--filter=testName` before running full suite
7. **Always check for JS errors** in browser tests — `assertNoJavaScriptErrors()`
</best_practices>

<success_criteria>
Test code follows patterns when:
- All tests use Pest syntax (not PHPUnit)
- Specific assertions used over generic status codes
- Factories used for all test data
- External services faked (Mail, Queue, Notification)
- Browser tests check for JS errors
- Architecture tests enforce project conventions
- Test names describe behavior clearly
- No hardcoded IDs or test data
</success_criteria>
