# Livewire Form Objects

<overview>
## What Form Objects Handle

- Input validation
- Data transformation
- Error messages
- Interaction with Actions

**What they DO NOT handle:**
- Business logic (that's for Actions)
- Database operations (that's for Actions/Models)
- External API calls (that's for Services via Actions)
</overview>

<basic_form>
## Basic Form Object

```php
namespace App\Livewire\Forms;

use App\Actions\User\CreateUser;
use App\Actions\User\UpdateUser;
use App\Models\User;
use Livewire\Attributes\Validate;
use Livewire\Form;

class UserForm extends Form
{
    public ?User $user = null;

    #[Validate('required|min:3|max:255')]
    public string $name = '';

    #[Validate('required|email')]
    public string $email = '';

    #[Validate('sometimes|nullable|min:8|confirmed')]
    public string $password = '';

    public string $password_confirmation = '';

    #[Validate('nullable|string|max:500')]
    public string $bio = '';

    /**
     * Initialize form from existing model
     */
    public function setUser(User $user): void
    {
        $this->user = $user;
        $this->fill($user->only(['name', 'email', 'bio']));
    }

    /**
     * Store a new user
     */
    public function store(CreateUser $action): User
    {
        $validated = $this->validate();
        $user = $action->handle($validated);
        $this->reset();
        return $user;
    }

    /**
     * Update existing user
     */
    public function update(User $user, UpdateUser $action): User
    {
        $validated = $this->validate();
        $user = $action->handle($user, $validated);
        $this->reset();
        return $user;
    }
}
```
</basic_form>

<using_in_components>
## Using Form Objects in Components

```php
namespace App\Livewire\User;

use App\Livewire\Forms\UserForm;
use App\Actions\User\CreateUser;
use App\Actions\User\UpdateUser;
use App\Models\User;
use Livewire\Component;

class CreateOrEdit extends Component
{
    public UserForm $form;
    public ?User $user = null;

    public function mount(?User $user = null)
    {
        if ($user) {
            $this->user = $user;
            $this->form->setUser($user);
        }
    }

    public function save(CreateUser $createUser, UpdateUser $updateUser)
    {
        if ($this->user) {
            $user = $this->form->update($this->user, $updateUser);
            $message = __('User updated successfully');
        } else {
            $user = $this->form->store($createUser);
            $message = __('User created successfully');
        }

        return redirect()
            ->route('users.show', $user)
            ->with('message', $message);
    }

    public function render()
    {
        return view('livewire.user.create-or-edit');
    }
}
```
</using_in_components>

<dynamic_rules>
## Dynamic Validation Rules

```php
use Illuminate\Validation\Rule;

class PostForm extends Form
{
    public ?Post $post = null;
    public string $title = '';
    public string $slug = '';
    public string $content = '';
    public int $category_id = 0;

    public function rules(): array
    {
        return [
            'title' => ['required', 'min:5', 'max:255'],
            'slug' => [
                'required',
                'alpha_dash',
                $this->post
                    ? Rule::unique('posts')->ignore($this->post->id)
                    : 'unique:posts'
            ],
            'content' => ['required', 'min:10'],
            'category_id' => ['required', 'exists:categories,id'],
        ];
    }

    public function messages(): array
    {
        return [
            'title.required' => __('Please provide a post title'),
            'slug.unique' => __('This URL slug is already in use'),
            'category_id.required' => __('Please select a category'),
        ];
    }
}
```
</dynamic_rules>

<real_time_validation>
## Real-time Validation

```php
class RegistrationForm extends Form
{
    #[Validate('required|min:3')]
    public string $name = '';

    #[Validate('required|email|unique:users,email')]
    public string $email = '';

    #[Validate('required|min:8|regex:/[A-Z]/|regex:/[0-9]/')]
    public string $password = '';

    /**
     * Validate a single field in real-time
     */
    public function updated($propertyName): void
    {
        $this->validateOnly($propertyName);
    }

    /**
     * Custom email availability check
     */
    public function updatedEmail(): void
    {
        $this->validateOnly('email');

        if (!$this->errors()->has('email') && $this->email) {
            if (User::where('email', $this->email)->exists()) {
                $this->addError('email', __('This email is already registered.'));
            }
        }
    }
}
```
</real_time_validation>

<multi_step_form>
## Multi-step Form

```php
class CheckoutForm extends Form
{
    public int $currentStep = 1;

    // Step 1: Cart
    #[Validate('required|array|min:1')]
    public array $items = [];

    // Step 2: Shipping
    #[Validate('required|string')]
    public string $shippingName = '';

    #[Validate('required|string')]
    public string $shippingAddress = '';

    #[Validate('required|string|size:5')]
    public string $shippingZip = '';

    // Step 3: Payment
    #[Validate('required|exists:payment_methods,id')]
    public int $paymentMethodId;

    public function validateStep(): bool
    {
        $rules = match($this->currentStep) {
            1 => ['items' => 'required|array|min:1'],
            2 => [
                'shippingName' => 'required|string',
                'shippingAddress' => 'required|string',
                'shippingZip' => 'required|string|size:5',
            ],
            3 => ['paymentMethodId' => 'required|exists:payment_methods,id'],
            default => []
        };

        $this->validate($rules);
        return true;
    }

    public function nextStep(): void
    {
        if ($this->validateStep()) {
            $this->currentStep++;
        }
    }

    public function previousStep(): void
    {
        $this->currentStep--;
    }

    public function submit(ProcessCheckout $action): Order
    {
        $validated = $this->validate();
        $order = $action->handle($validated);
        $this->reset();
        return $order;
    }
}
```
</multi_step_form>

<file_uploads>
## File Upload Forms

```php
use Livewire\WithFileUploads;
use Livewire\Features\SupportFileUploads\TemporaryUploadedFile;

class ProfileForm extends Form
{
    use WithFileUploads;

    #[Validate('required|min:3')]
    public string $name = '';

    #[Validate('nullable|image|max:2048')]
    public ?TemporaryUploadedFile $avatar = null;

    public function setProfile(User $user): void
    {
        $this->fill($user->only(['name', 'email']));
    }

    public function update(User $user, UpdateProfile $action): User
    {
        $validated = $this->validate();

        if ($this->avatar) {
            $validated['avatar_path'] = $this->avatar->store('avatars', 'public');
        }

        $user = $action->handle($user, $validated);
        $this->reset();
        return $user;
    }

    /**
     * Preview avatar before saving
     */
    public function getAvatarPreviewProperty(): ?string
    {
        return $this->avatar?->temporaryUrl();
    }
}
```
</file_uploads>

<dependent_fields>
## Dependent Fields

```php
class LocationForm extends Form
{
    #[Validate('required|exists:countries,id')]
    public int $countryId = 0;

    #[Validate('required|exists:states,id')]
    public int $stateId = 0;

    #[Validate('required|exists:cities,id')]
    public int $cityId = 0;

    public function getStatesProperty(): Collection
    {
        if (!$this->countryId) {
            return collect();
        }
        return State::where('country_id', $this->countryId)->get();
    }

    public function getCitiesProperty(): Collection
    {
        if (!$this->stateId) {
            return collect();
        }
        return City::where('state_id', $this->stateId)->get();
    }

    public function updatedCountryId(): void
    {
        $this->stateId = 0;
        $this->cityId = 0;
    }

    public function updatedStateId(): void
    {
        $this->cityId = 0;
    }
}
```
</dependent_fields>

<testing>
## Testing Form Objects

```php
use App\Livewire\Forms\UserForm;
use App\Livewire\User\Create;
use Livewire\Livewire;

it('validates form fields', function () {
    Livewire::test(Create::class)
        ->set('form.name', 'Jo') // Too short
        ->set('form.email', 'invalid') // Invalid email
        ->call('save')
        ->assertHasErrors([
            'form.name' => 'min',
            'form.email' => 'email',
        ]);
});

it('stores user through form', function () {
    Livewire::test(Create::class)
        ->set('form.name', 'John Doe')
        ->set('form.email', 'john@example.com')
        ->set('form.password', 'password123')
        ->set('form.password_confirmation', 'password123')
        ->call('save')
        ->assertHasNoErrors()
        ->assertRedirect('/users');

    $this->assertDatabaseHas('users', [
        'email' => 'john@example.com'
    ]);
});

it('validates in real-time', function () {
    Livewire::test(Create::class)
        ->set('form.email', 'john@example.com')
        ->assertHasNoErrors('form.email')
        ->set('form.email', 'invalid')
        ->assertHasErrors(['form.email' => 'email']);
});
```
</testing>

<best_practices>
## Best Practices

1. **Keep forms focused** - One form per purpose
2. **Use DTOs** - Transform validated data before passing to Actions
3. **Reset after operations** - Always `$this->reset()` after successful operations
4. **Clear error messages** - Use `messages()` for user-friendly errors
5. **Computed properties** - For derived values like totals
</best_practices>
