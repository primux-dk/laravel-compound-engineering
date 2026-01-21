# Flux UI Pro Component Patterns

<principles>
## Core Principles

1. **Consistency** - Use Flux components throughout the application
2. **Accessibility** - Flux components include ARIA attributes by default
3. **Responsive** - All components are mobile-first responsive
4. **Customization** - Extend with Tailwind classes when needed
</principles>

<layout>
## Layout Components

**Application Shell:**
```blade
<flux:layout>
    <flux:sidebar>
        <flux:logo href="/" />

        <flux:nav>
            <flux:nav-item href="/dashboard" icon="home">
                Dashboard
            </flux:nav-item>
            <flux:nav-item href="/users" icon="users">
                Users
            </flux:nav-item>
        </flux:nav>
    </flux:sidebar>

    <flux:main>
        <flux:header>
            <flux:heading>{{ $title }}</flux:heading>

            <flux:actions>
                <flux:button variant="primary" icon="plus">
                    New Item
                </flux:button>
            </flux:actions>
        </flux:header>

        <flux:content>
            {{ $slot }}
        </flux:content>
    </flux:main>
</flux:layout>
```

**Page Structure:**
```blade
<flux:page>
    <flux:page-header
        title="Users"
        description="Manage your application users"
    >
        <flux:button variant="primary" wire:click="create">
            Create User
        </flux:button>
    </flux:page-header>

    <flux:page-body>
        {{-- Page content --}}
    </flux:page-body>
</flux:page>
```
</layout>

<forms>
## Form Components

**Complete Form:**
```blade
<flux:form wire:submit="save">
    <flux:form-section>
        <flux:heading size="lg">User Information</flux:heading>

        <flux:field>
            <flux:label>Full Name</flux:label>
            <flux:input wire:model="form.name" />
            <flux:error name="form.name" />
            <flux:hint>Enter your full legal name</flux:hint>
        </flux:field>

        <flux:field>
            <flux:label>Email Address</flux:label>
            <flux:input type="email" wire:model="form.email" />
            <flux:error name="form.email" />
        </flux:field>

        <flux:field>
            <flux:label>Bio</flux:label>
            <flux:textarea wire:model="form.bio" rows="4" />
            <flux:error name="form.bio" />
        </flux:field>
    </flux:form-section>

    <flux:form-footer>
        <flux:button variant="ghost" type="button" wire:click="cancel">
            Cancel
        </flux:button>
        <flux:button variant="primary" type="submit">
            Save Changes
        </flux:button>
    </flux:form-footer>
</flux:form>
```

**Input Types:**
```blade
{{-- Text --}}
<flux:input wire:model="name" placeholder="Enter name" />

{{-- Password --}}
<flux:input type="password" wire:model="password" />

{{-- Email --}}
<flux:input type="email" wire:model="email" />

{{-- Search with icon --}}
<flux:input
    type="search"
    wire:model.live.debounce="search"
    placeholder="Search..."
    icon="search"
/>
```

**Select Components:**
```blade
{{-- Basic Select --}}
<flux:select wire:model="form.role">
    <flux:option value="">Choose a role</flux:option>
    <flux:option value="admin">Administrator</flux:option>
    <flux:option value="editor">Editor</flux:option>
</flux:select>

{{-- Dynamic Options --}}
<flux:select wire:model="form.country_id">
    <flux:option value="">Select Country</flux:option>
    @foreach($countries as $country)
        <flux:option value="{{ $country->id }}">
            {{ $country->name }}
        </flux:option>
    @endforeach
</flux:select>

{{-- Searchable Select --}}
<flux:select.search
    wire:model="form.user_id"
    :options="$users"
    option-label="name"
    option-value="id"
    placeholder="Search users..."
/>
```

**Checkboxes and Radios:**
```blade
{{-- Single Checkbox --}}
<flux:checkbox wire:model="form.terms">
    I agree to the terms and conditions
</flux:checkbox>

{{-- Checkbox Group --}}
<flux:checkbox-group wire:model="form.permissions">
    <flux:checkbox value="read">Read</flux:checkbox>
    <flux:checkbox value="write">Write</flux:checkbox>
    <flux:checkbox value="delete">Delete</flux:checkbox>
</flux:checkbox-group>

{{-- Radio Group --}}
<flux:radio-group wire:model="form.plan">
    <flux:radio value="basic">
        <flux:label>Basic Plan</flux:label>
        <flux:description>$10/month</flux:description>
    </flux:radio>
    <flux:radio value="pro">
        <flux:label>Pro Plan</flux:label>
        <flux:description>$30/month</flux:description>
    </flux:radio>
</flux:radio-group>
```

**File Upload:**
```blade
<flux:field>
    <flux:label>Profile Photo</flux:label>
    <flux:file wire:model="form.avatar" accept="image/*">
        <flux:button variant="secondary" icon="upload">
            Choose File
        </flux:button>
    </flux:file>

    @if ($form->avatar)
        <flux:image-preview :src="$form->avatar->temporaryUrl()" />
    @endif

    <flux:error name="form.avatar" />
</flux:field>
```
</forms>

<buttons>
## Buttons

**Variants:**
```blade
<flux:button variant="primary">Save</flux:button>
<flux:button variant="secondary">Cancel</flux:button>
<flux:button variant="danger">Delete</flux:button>
<flux:button variant="ghost">Learn More</flux:button>
```

**Sizes:**
```blade
<flux:button variant="primary" size="sm">Small</flux:button>
<flux:button variant="primary">Default</flux:button>
<flux:button variant="primary" size="lg">Large</flux:button>
```

**With Icons:**
```blade
<flux:button variant="primary" icon="plus">
    Create New
</flux:button>

{{-- Icon Only --}}
<flux:button.icon icon="trash" variant="danger" />
```

**Loading State:**
```blade
<flux:button wire:click="save" wire:loading.attr="disabled">
    <flux:spinner wire:loading wire:target="save" />
    <span wire:loading.remove wire:target="save">Save</span>
    <span wire:loading wire:target="save">Saving...</span>
</flux:button>
```

**Button Groups:**
```blade
<flux:button-group>
    <flux:button variant="secondary">Previous</flux:button>
    <flux:button variant="primary">Next</flux:button>
</flux:button-group>
```
</buttons>

<tables>
## Data Tables

```blade
<flux:table>
    <flux:thead>
        <flux:tr>
            <flux:th>Name</flux:th>
            <flux:th>Email</flux:th>
            <flux:th>Role</flux:th>
            <flux:th></flux:th>
        </flux:tr>
    </flux:thead>
    <flux:tbody>
        @foreach($users as $user)
            <flux:tr wire:key="user-{{ $user->id }}">
                <flux:td>{{ $user->name }}</flux:td>
                <flux:td>{{ $user->email }}</flux:td>
                <flux:td>
                    <flux:badge variant="{{ $user->is_admin ? 'primary' : 'secondary' }}">
                        {{ $user->role }}
                    </flux:badge>
                </flux:td>
                <flux:td class="text-right">
                    <flux:dropdown>
                        <flux:button.icon icon="dots-vertical" variant="ghost" />

                        <flux:menu>
                            <flux:menu-item wire:click="edit({{ $user->id }})">
                                Edit
                            </flux:menu-item>
                            <flux:menu-item wire:click="delete({{ $user->id }})" variant="danger">
                                Delete
                            </flux:menu-item>
                        </flux:menu>
                    </flux:dropdown>
                </flux:td>
            </flux:tr>
        @endforeach
    </flux:tbody>
</flux:table>

{{-- Pagination --}}
{{ $users->links('flux::pagination') }}
```
</tables>

<cards>
## Cards

```blade
<flux:card>
    <flux:card-header>
        <flux:heading>Card Title</flux:heading>
        <flux:card-actions>
            <flux:button.icon icon="dots-horizontal" variant="ghost" />
        </flux:card-actions>
    </flux:card-header>

    <flux:card-body>
        Card content goes here
    </flux:card-body>

    <flux:card-footer>
        <flux:button variant="primary">Action</flux:button>
    </flux:card-footer>
</flux:card>

{{-- Stats Cards --}}
<flux:stats>
    <flux:stat>
        <flux:stat-label>Total Users</flux:stat-label>
        <flux:stat-value>1,234</flux:stat-value>
        <flux:stat-change variant="success">+12%</flux:stat-change>
    </flux:stat>
</flux:stats>
```
</cards>

<feedback>
## Feedback Components

**Alerts:**
```blade
<flux:alert variant="success" dismissible>
    <flux:alert-icon icon="check-circle" />
    <flux:alert-content>
        Operation completed successfully!
    </flux:alert-content>
</flux:alert>

<flux:alert variant="danger">
    <flux:alert-icon icon="exclamation-circle" />
    <flux:alert-content>
        <flux:alert-title>Error</flux:alert-title>
        There was a problem processing your request.
    </flux:alert-content>
</flux:alert>
```

**Toasts (from component):**
```php
public function save()
{
    // Save logic...

    $this->dispatch('flux:toast', [
        'variant' => 'success',
        'message' => __('Settings saved successfully'),
    ]);
}
```

```blade
<flux:toaster position="top-right" />
```

**Modals:**
```blade
<flux:modal wire:model="showDeleteModal">
    <flux:modal-header>
        Confirm Deletion
    </flux:modal-header>

    <flux:modal-body>
        Are you sure you want to delete this user?
    </flux:modal-body>

    <flux:modal-footer>
        <flux:button variant="ghost" wire:click="$set('showDeleteModal', false)">
            Cancel
        </flux:button>
        <flux:button variant="danger" wire:click="confirmDelete">
            Delete User
        </flux:button>
    </flux:modal-footer>
</flux:modal>
```
</feedback>

<navigation>
## Navigation

**Tabs:**
```blade
<flux:tabs wire:model="activeTab">
    <flux:tab-list>
        <flux:tab value="profile">Profile</flux:tab>
        <flux:tab value="settings">Settings</flux:tab>
        <flux:tab value="billing">Billing</flux:tab>
    </flux:tab-list>

    <flux:tab-panels>
        <flux:tab-panel value="profile">
            {{-- Profile content --}}
        </flux:tab-panel>
        <flux:tab-panel value="settings">
            {{-- Settings content --}}
        </flux:tab-panel>
    </flux:tab-panels>
</flux:tabs>
```

**Breadcrumbs:**
```blade
<flux:breadcrumbs>
    <flux:breadcrumb href="/">Home</flux:breadcrumb>
    <flux:breadcrumb href="/users">Users</flux:breadcrumb>
    <flux:breadcrumb>Edit Profile</flux:breadcrumb>
</flux:breadcrumbs>
```

**Dropdowns:**
```blade
<flux:dropdown>
    <flux:button variant="secondary">
        Options
        <flux:icon icon="chevron-down" />
    </flux:button>

    <flux:menu>
        <flux:menu-item wire:click="edit">Edit</flux:menu-item>
        <flux:menu-item wire:click="duplicate">Duplicate</flux:menu-item>
        <flux:menu-separator />
        <flux:menu-item wire:click="delete" variant="danger">
            Delete
        </flux:menu-item>
    </flux:menu>
</flux:dropdown>
```
</navigation>

<loading>
## Loading States

**Spinners:**
```blade
<flux:spinner />
<flux:spinner size="sm" />
<flux:spinner size="lg" />

<div wire:loading>
    <flux:spinner />
</div>
```

**Skeleton Loaders:**
```blade
<flux:skeleton>
    <flux:skeleton-line />
    <flux:skeleton-line width="75%" />
    <flux:skeleton-line width="50%" />
</flux:skeleton>
```
</loading>

<utilities>
## Utility Components

**Tooltips:**
```blade
<flux:tooltip content="This is helpful information">
    <flux:button variant="ghost">Hover me</flux:button>
</flux:tooltip>
```

**Popovers:**
```blade
<flux:popover>
    <flux:popover-trigger>
        <flux:button>Open Popover</flux:button>
    </flux:popover-trigger>

    <flux:popover-content>
        <flux:heading size="sm">Popover Title</flux:heading>
        <p>Popover content goes here</p>
    </flux:popover-content>
</flux:popover>
```
</utilities>

<customization>
## Customization

**Extending with Tailwind:**
```blade
<flux:button variant="primary" class="w-full lg:w-auto">
    Full Width on Mobile
</flux:button>

<flux:card class="bg-gradient-to-r from-blue-500 to-purple-600 text-white">
    <flux:card-body>
        Gradient Card
    </flux:card-body>
</flux:card>
```

**Custom Themes:**
```css
/* resources/css/flux-theme.css */
:root {
    --flux-primary: theme('colors.indigo.600');
    --flux-primary-hover: theme('colors.indigo.700');
    --flux-secondary: theme('colors.gray.600');
    --flux-danger: theme('colors.red.600');
    --flux-success: theme('colors.green.600');
}
```
</customization>

<best_practices>
## Best Practices

1. **Use Flux consistently** - Don't mix with raw HTML forms
2. **Leverage built-in validation** - Use `flux:error` components
3. **Show loading states** - Use spinners and skeletons
4. **Follow accessibility** - Flux handles ARIA attributes
5. **Use semantic variants** - primary, secondary, danger, etc.
</best_practices>
