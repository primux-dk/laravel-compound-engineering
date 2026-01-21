---
name: laravel-livewire-patterns
description: This skill should be used when building Livewire v3 components, Form Objects, and Flux UI interfaces. It applies when creating reactive UI components, handling form validation, building TALL stack applications, or implementing real-time features. Triggers on Livewire, Flux UI, Form Objects, TALL stack, or Alpine.js integration mentions.
---

<objective>
Provide comprehensive patterns for building reactive interfaces with Livewire v3, including component architecture, Form Objects for validation, and Flux UI Pro integration. Focuses on thin orchestration - components delegate business logic to Actions.
</objective>

<essential_principles>
## Core Philosophy

**Livewire components are thin orchestration layers** - just like controllers.

- Components manage UI state and user interactions
- All business logic delegates to Actions
- Form Objects handle validation
- Flux UI provides consistent, accessible components

**The TALL Stack:**
- **T**ailwind CSS - Utility-first styling
- **A**lpine.js - Lightweight reactivity
- **L**ivewire - Full-stack reactive components
- **L**aravel - Backend framework
</essential_principles>

<intake>
What are you building?

1. **Components** - Livewire component structure, lifecycle, attributes
2. **Form Objects** - Validation, real-time validation, multi-step forms
3. **Flux UI** - Pro components, forms, tables, modals, navigation
4. **Real-time** - Live search, polling, events, component communication
5. **Code Review** - Review Livewire code against patterns

**Specify a number or describe your task.**
</intake>

<routing>
| Response | Reference to Read |
|----------|-------------------|
| 1, "component" | [components.md](./references/components.md) |
| 2, "form", "validation" | [form-objects.md](./references/form-objects.md) |
| 3, "flux", "ui" | [flux-ui.md](./references/flux-ui.md) |
| 4, "real-time", "live", "event" | [components.md](./references/components.md) |
| 5, "review" | Read all references, then review code |

**After reading relevant references, apply patterns to the user's code.**
</routing>

<quick_reference>
## Component Structure

```php
namespace App\Livewire\User;

use App\Livewire\Forms\UserForm;
use App\Actions\User\CreateUser;
use Livewire\Component;
use Livewire\Attributes\Title;

#[Title('Create User')]
class Create extends Component
{
    public UserForm $form;

    public function save(CreateUser $createUser)
    {
        $user = $this->form->store($createUser);
        $this->dispatch('user-created', userId: $user->id);

        return redirect()->route('users.show', $user);
    }

    public function render()
    {
        return view('livewire.user.create');
    }
}
```

## Form Object

```php
namespace App\Livewire\Forms;

use App\Actions\User\CreateUser;
use Livewire\Attributes\Validate;
use Livewire\Form;

class UserForm extends Form
{
    #[Validate('required|min:3')]
    public string $name = '';

    #[Validate('required|email|unique:users')]
    public string $email = '';

    public function store(CreateUser $action): User
    {
        $validated = $this->validate();
        $user = $action->handle($validated);
        $this->reset();
        return $user;
    }
}
```

## Flux UI Form

```blade
<flux:form wire:submit="save">
    <flux:field>
        <flux:label>Name</flux:label>
        <flux:input wire:model="form.name" />
        <flux:error name="form.name" />
    </flux:field>

    <flux:button type="submit" variant="primary">
        Save
    </flux:button>
</flux:form>
```

## Key Attributes

```php
#[Title('Page Title')]           // Set page title
#[Layout('layouts.app')]         // Set layout
#[Lazy]                          // Lazy load component
#[On('event-name')]              // Listen for events
#[Url(as: 'q')]                  // Sync with URL
#[Locked]                        // Prevent tampering
#[Computed(cache: true)]         // Cached computed property
#[Validate('required')]          // Validation rule
```

## Naming Conventions

| Component | Convention | Example |
|-----------|------------|---------|
| Classes | PascalCase | `CreatePost`, `UserProfile` |
| Blade files | kebab-case | `create-post.blade.php` |
| Invocation | kebab-case | `<livewire:create-post />` |
| Form Objects | `{Model}Form` | `UserForm`, `PostForm` |
</quick_reference>

<reference_index>
## Domain Knowledge

All detailed patterns in `references/`:

| File | Topics |
|------|--------|
| [components.md](./references/components.md) | Structure, lifecycle, events, communication, lazy loading |
| [form-objects.md](./references/form-objects.md) | Validation, real-time validation, multi-step, file uploads |
| [flux-ui.md](./references/flux-ui.md) | Layout, forms, tables, modals, navigation, customization |
</reference_index>

<success_criteria>
Livewire code follows patterns when:
- Components are thin orchestration layers
- Business logic lives in Actions
- Form Objects handle all validation
- Flux UI components used consistently
- Proper attributes applied (#[Title], #[Lazy], etc.)
- Events used for component communication
- Real-time validation with `validateOnly()`
- Loading states shown during operations
</success_criteria>
