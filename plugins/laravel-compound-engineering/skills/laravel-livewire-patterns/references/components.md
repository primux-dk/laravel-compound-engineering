# Livewire Component Patterns

<core_principle>
**Livewire components are thin orchestration layers** - They manage UI state and user interactions, delegating all business logic to Actions, just like Controllers.
</core_principle>

<directory_structure>
## Organization

```
app/Livewire/
├── Auth/
│   ├── Login.php
│   ├── Register.php
│   └── PasswordReset.php
├── Dashboard/
│   ├── Stats.php
│   └── ActivityFeed.php
├── Posts/
│   ├── Create.php
│   ├── Edit.php
│   ├── Index.php
│   └── Show.php
├── Forms/               # Form Objects
│   ├── PostForm.php
│   └── UserForm.php
└── Shared/
    ├── Navigation.php
    └── SearchBar.php
```

```
resources/views/livewire/
├── auth/
│   ├── login.blade.php
│   └── register.blade.php
├── posts/
│   ├── create.blade.php
│   └── index.blade.php
└── shared/
    └── navigation.blade.php
```
</directory_structure>

<basic_component>
## Basic Component Structure

```php
namespace App\Livewire\User;

use App\Livewire\Forms\UserForm;
use App\Actions\User\CreateUser;
use Livewire\Component;
use Livewire\Attributes\Title;
use Livewire\Attributes\Layout;

#[Title('Create User')]
#[Layout('layouts.app')]
class Create extends Component
{
    public UserForm $form;

    public function save(CreateUser $createUser)
    {
        // Form object handles validation
        $user = $this->form->store($createUser);

        $this->dispatch('user-created', userId: $user->id);

        return redirect()
            ->route('users.show', $user)
            ->with('message', __('User created successfully'));
    }

    public function render()
    {
        return view('livewire.user.create');
    }
}
```

**Blade template:**
```blade
<div>
    <form wire:submit="save">
        <div>
            <label for="name">Name</label>
            <input type="text" id="name" wire:model="form.name">
            @error('form.name') <span>{{ $message }}</span> @enderror
        </div>

        <button type="submit">Save</button>
    </form>
</div>
```
</basic_component>

<attributes>
## Livewire Attributes

```php
#[Title('Page Title')]              // Set page <title>
#[Layout('layouts.app')]            // Set layout
#[Lazy]                             // Lazy load component
#[On('event-name')]                 // Listen for events
#[Url(as: 'q')]                     // Sync property with URL
#[Locked]                           // Prevent client tampering
#[Computed]                         // Computed property
#[Computed(cache: true)]            // Cached computed property
#[Validate('required|email')]       // Validation rule
```

**Usage examples:**
```php
class UserSearch extends Component
{
    #[Url(as: 'q')]
    public string $search = '';

    #[Locked]
    public int $userId;

    #[Computed(cache: true)]
    public function user()
    {
        return User::find($this->userId);
    }

    #[On('user-updated')]
    public function refreshUser()
    {
        unset($this->user); // Clear cache
    }
}
```
</attributes>

<real_time_features>
## Real-time Features

**Live Search:**
```php
class UserSearch extends Component
{
    #[Url(as: 'q')]
    public string $search = '';

    public function render(SearchUsers $searchUsers)
    {
        $results = strlen($this->search) >= 3
            ? $searchUsers->handle($this->search)
            : collect();

        return view('livewire.user-search', [
            'results' => $results
        ]);
    }
}
```

**Debounced input:**
```blade
<input type="text" wire:model.live.debounce.300ms="search">
```

**Polling:**
```blade
<div wire:poll.5s>
    @foreach($activities as $activity)
        <div>{{ $activity->description }}</div>
    @endforeach
</div>
```
</real_time_features>

<component_communication>
## Component Communication

**Parent to child (events):**
```php
// Parent dispatches
$this->dispatch('post-deleted', postId: $postId);

// Child listens
#[On('post-deleted')]
public function handlePostDeleted($postId)
{
    if ($this->post->id === $postId) {
        $this->hidden = true;
    }
}
```

**Child to parent:**
```php
// Child dispatches
$this->dispatch('comment-added', commentId: $comment->id);

// Parent listens
#[On('comment-added')]
public function refreshComments()
{
    $this->comments = $this->post->comments()->latest()->get();
}
```

**Global events:**
```php
// Dispatch globally
$this->dispatch('notification',
    type: 'success',
    message: 'Operation completed'
);
```
</component_communication>

<pagination>
## Pagination

```php
use Livewire\WithPagination;

class PostIndex extends Component
{
    use WithPagination;

    public string $search = '';
    public string $sortBy = 'created_at';
    public string $sortDirection = 'desc';

    public function updatingSearch()
    {
        $this->resetPage();
    }

    public function sort($field)
    {
        if ($this->sortBy === $field) {
            $this->sortDirection = $this->sortDirection === 'asc' ? 'desc' : 'asc';
        } else {
            $this->sortBy = $field;
            $this->sortDirection = 'asc';
        }
    }

    public function render()
    {
        $posts = Post::query()
            ->when($this->search, fn($q) =>
                $q->where('title', 'like', "%{$this->search}%")
            )
            ->orderBy($this->sortBy, $this->sortDirection)
            ->paginate(10);

        return view('livewire.post.index', ['posts' => $posts]);
    }
}
```
</pagination>

<lazy_loading>
## Lazy Loading

```php
use Livewire\Attributes\Lazy;

#[Lazy]
class ExpensiveReport extends Component
{
    public $report = null;

    public function mount(GenerateReport $generateReport)
    {
        $this->report = $generateReport->handle(auth()->user());
    }

    public function placeholder()
    {
        return view('livewire.report-skeleton');
    }

    public function render()
    {
        return view('livewire.expensive-report');
    }
}
```
</lazy_loading>

<file_uploads>
## File Uploads

```php
use Livewire\WithFileUploads;
use Livewire\Attributes\Validate;

class AvatarUpload extends Component
{
    use WithFileUploads;

    #[Validate('image|max:1024')]
    public $avatar;

    public function updatedAvatar()
    {
        $this->validateOnly('avatar');
    }

    public function save(UpdateAvatar $updateAvatar)
    {
        $this->validate();

        $path = $updateAvatar->handle(
            auth()->user(),
            $this->avatar
        );

        $this->dispatch('avatar-updated', path: $path);
        $this->reset('avatar');
    }
}
```
</file_uploads>

<authorization>
## Authorization

```php
class EditPost extends Component
{
    public Post $post;

    public function mount(Post $post)
    {
        $this->authorize('update', $post);
        $this->post = $post;
    }

    public function save()
    {
        $this->authorize('update', $this->post);
        // Save logic
    }
}
```
</authorization>

<testing>
## Testing Components

```php
use Livewire\Livewire;

it('creates user through component', function () {
    Livewire::test(Create::class)
        ->set('form.name', 'John Doe')
        ->set('form.email', 'john@example.com')
        ->set('form.password', 'password123')
        ->call('save')
        ->assertDispatched('user-created')
        ->assertRedirect('/users');

    $this->assertDatabaseHas('users', [
        'email' => 'john@example.com'
    ]);
});

it('validates in real-time', function () {
    Livewire::test(Create::class)
        ->set('form.email', 'invalid')
        ->assertHasErrors(['form.email' => 'email']);
});
```
</testing>

<anti_patterns>
## Anti-Patterns to Avoid

**Business logic in components:**
```php
// Bad
public function save()
{
    $user = User::create($this->form->all());
    $user->assignRole('customer');
    Mail::to($user)->send(new WelcomeEmail());
}

// Good
public function save(CreateUser $createUser)
{
    $user = $this->form->store($createUser);
    $this->dispatch('user-created', userId: $user->id);
}
```

**Complex queries in components:**
```php
// Bad
public function render()
{
    $users = User::with(['posts' => fn($q) => $q->published()])
        ->whereHas('subscription')
        ->get();
}

// Good
public function render(GetActiveUsers $getActiveUsers)
{
    return view('livewire.users', [
        'users' => $getActiveUsers->handle()
    ]);
}
```
</anti_patterns>
