---
name: tailwindcss-v4
description: This skill should be used when styling with Tailwind CSS v4. It applies when adding styles, working with responsive design, dark mode, gradients, spacing, layout, or any visual changes. Triggers on CSS, Tailwind, styling, classes, restyle, or UI change mentions. Provides v4-specific patterns including CSS-first configuration, replaced utilities, and modern layout patterns.
---

<objective>
Provide Tailwind CSS v4 patterns and prevent use of deprecated v3 utilities. Covers CSS-first configuration, replaced utilities, and modern layout patterns.
</objective>

<essential_principles>
## Core Rules

- **Always use Tailwind CSS v4** — avoid deprecated v3 utilities
- **Check existing conventions** before introducing new patterns
- **Extract repeated patterns** into components matching project conventions (Blade, Vue, React)
- **Use `gap` over margins** for spacing between siblings
- **Add dark mode variants** when the project uses dark mode
</essential_principles>

<quick_reference>
## CSS-First Configuration

Tailwind v4 uses CSS-first configuration with `@theme` — no `tailwind.config.js` needed:

```css
@import "tailwindcss";

@theme {
  --color-brand: oklch(0.72 0.11 178);
  --font-display: "Satoshi", sans-serif;
}
```

## Import Syntax

```diff
- @tailwind base;
- @tailwind components;
- @tailwind utilities;
+ @import "tailwindcss";
```

## Replaced Utilities (v3 → v4)

These v3 utilities are removed. Use the replacements:

| Deprecated (v3) | Replacement (v4) |
|-----------------|------------------|
| `bg-opacity-*` | `bg-black/*` (e.g. `bg-black/50`) |
| `text-opacity-*` | `text-black/*` |
| `border-opacity-*` | `border-black/*` |
| `divide-opacity-*` | `divide-black/*` |
| `ring-opacity-*` | `ring-black/*` |
| `placeholder-opacity-*` | `placeholder-black/*` |
| `flex-shrink-*` | `shrink-*` |
| `flex-grow-*` | `grow-*` |
| `overflow-ellipsis` | `text-ellipsis` |
| `decoration-slice` | `box-decoration-slice` |
| `decoration-clone` | `box-decoration-clone` |

Opacity values remain numeric (e.g. `bg-blue-500/75`).

## Not Supported in v4

- `corePlugins` configuration
- `tailwind.config.js` (use CSS `@theme` instead)
- `@tailwind` directives (use `@import` instead)

## Spacing

Use `gap` utilities instead of margins for siblings:

```html
<div class="flex gap-8">
    <div>Item 1</div>
    <div>Item 2</div>
</div>
```

## Dark Mode

Match existing project dark mode patterns using the `dark:` variant:

```html
<div class="bg-white dark:bg-gray-900 text-gray-900 dark:text-white">
    Content adapts to color scheme
</div>
```

## Common Layout Patterns

**Flexbox:**
```html
<div class="flex items-center justify-between gap-4">
    <div>Left</div>
    <div>Right</div>
</div>
```

**Grid:**
```html
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
    <div>Card 1</div>
    <div>Card 2</div>
    <div>Card 3</div>
</div>
```

**Container:**
```html
<div class="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8">
    <!-- Centered content -->
</div>
```
</quick_reference>

<common_pitfalls>
## Common Pitfalls

- Using deprecated v3 utilities (`bg-opacity-*`, `flex-shrink-*`, etc.)
- Using `@tailwind` directives instead of `@import "tailwindcss"`
- Creating `tailwind.config.js` instead of using CSS `@theme` directive
- Using margins for spacing between siblings instead of `gap`
- Forgetting dark mode variants when the project uses dark mode
- Including opacity utilities with separate classes instead of slash syntax
</common_pitfalls>

<success_criteria>
Tailwind code follows v4 patterns when:
- No deprecated v3 utilities are used
- Configuration is in CSS with `@theme`, not `tailwind.config.js`
- Import uses `@import "tailwindcss"` not `@tailwind` directives
- `gap` used for sibling spacing, not margins
- Dark mode variants present where project requires them
- Opacity uses slash syntax (`bg-blue-500/50`)
</success_criteria>
