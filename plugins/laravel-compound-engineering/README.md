# Laravel Compound Engineering Plugin

Laravel-specific knowledge for the [Compound Engineering](https://github.com/EveryInc/compound-engineering-plugin) plugin. Provides 5 agents and 8 skills for Laravel, Livewire, Pest, and TALL stack development.

## Quick Start

Run `/laravel-setup` in your Laravel project to configure review workflows with Laravel-specific agents.

> **Note:** This is a companion to the [compound-engineering](https://github.com/EveryInc/compound-engineering-plugin) plugin. Install that first for workflow commands.

## Agents

| Agent | Category | Description |
|-------|----------|-------------|
| `primux-laravel-reviewer` | Review | Laravel code review using Taylor Otwell's conventions |
| `data-integrity-guardian` | Review | Database migration and data integrity review |
| `data-migration-expert` | Review | Validate ID mappings, column renames, and data transformations |
| `github-code-researcher` | Research | Search GitHub for open source implementations via `gh` CLI |
| `lint` | Workflow | PHP/Blade linting with Laravel Pint |

### Injected Skills

These agents automatically load relevant skills:

- **primux-laravel-reviewer** injects: `taylor-otwell-style`, `laravel-livewire-patterns`, `laravel-database-patterns`
- **data-integrity-guardian** injects: `laravel-database-patterns`
- **data-migration-expert** injects: `laravel-database-patterns`

## Skills

### Laravel Patterns

| Skill | Description |
|-------|-------------|
| `taylor-otwell-style` | Taylor Otwell's PHP/Laravel coding style (Actions, thin controllers, i18n) |
| `laravel-livewire-patterns` | Livewire v4 components, Form Objects, Flux UI Pro patterns |
| `laravel-database-patterns` | Database migrations, transactions, Eloquent optimization |
| `pest-testing` | Pest 4 testing: browser tests, architecture tests, datasets |
| `tailwindcss-v4` | Tailwind CSS v4 patterns and v3 migration guide |
| `spatie-laravel-package-writer` | Laravel package development following Spatie patterns |

### Tools

| Skill | Description |
|-------|-------------|
| `web-interface-review` | Review web UI for accessibility, performance, and UX compliance |
| `laravel-setup` | Configure upstream plugin to use Laravel agents (`/laravel-setup`) |

## Version History

See [CHANGELOG.md](CHANGELOG.md) for detailed version history.

## Attribution

This plugin builds on the [Compound Engineering Plugin](https://github.com/EveryInc/compound-engineering-plugin) by Every Inc, created by Kieran Klaassen.

## License

MIT
