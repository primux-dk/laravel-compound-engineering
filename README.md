# Laravel Compound Engineering Plugin

Laravel-specific agents and skills for the [Compound Engineering Plugin](https://github.com/EveryInc/compound-engineering-plugin). Provides expert knowledge for Laravel, Livewire, Pest, and TALL stack development.

## Install

First, install the upstream compound-engineering plugin for workflow commands and generic agents:

```bash
/plugin marketplace add https://github.com/EveryInc/compound-engineering-plugin
/plugin install compound-engineering
```

Then add this marketplace and install the Laravel companion:

```bash
/plugin marketplace add https://github.com/primux-dk/laravel-compound-engineering
/plugin install laravel-compound-engineering
```

## Quick Start

Run `/laravel-setup` in your Laravel project to configure the upstream plugin's review workflows with Laravel-specific agents.

This generates `.claude/compound-engineering.local.md` pre-configured with:
- `taylor-otwell-reviewer` for code review
- `data-integrity-guardian` and `data-migration-expert` for database changes
- `github-code-researcher` for finding real-world patterns
- `lint` for PHP/Blade quality checks

## What's Included

### 5 Agents

| Agent | Purpose |
|-------|---------|
| `taylor-otwell-reviewer` | Laravel code review from Taylor Otwell's perspective |
| `data-integrity-guardian` | Database migration and data integrity review |
| `data-migration-expert` | Validate ID mappings, column renames, data transformations |
| `github-code-researcher` | Search GitHub for open source implementations via `gh` CLI |
| `lint` | PHP/Blade linting with Laravel Pint |

### 8 Skills

| Skill | Purpose |
|-------|---------|
| `taylor-otwell-style` | Taylor Otwell's PHP/Laravel coding patterns |
| `laravel-livewire-patterns` | Livewire v4, Form Objects, Flux UI Pro |
| `laravel-database-patterns` | Migrations, transactions, Eloquent optimization |
| `pest-testing` | Pest 4 testing patterns |
| `tailwindcss-v4` | Tailwind CSS v4 patterns |
| `spatie-laravel-package-writer` | Spatie-style Laravel packages |
| `web-interface-review` | Web UI accessibility and UX review |
| `laravel-setup` | Configure upstream plugin for Laravel (`/laravel-setup`) |

## How It Works

This plugin provides **Laravel expertise** while the upstream compound-engineering plugin provides **workflows**. Use them together:

```
/workflows:plan    → Plans your feature (upstream)
/workflows:work    → Builds it (upstream)
/workflows:review  → Reviews with Laravel agents (upstream + this plugin)
```

The review agents from this plugin are automatically used when configured via `/laravel-setup`.

## CLI Tool

Convert Claude Code plugins to other AI assistant formats:

```bash
# Convert to OpenCode or Codex format
bunx @primux/laravel-compound-plugin convert --target opencode
bunx @primux/laravel-compound-plugin convert --target codex

# Sync personal config
bunx @primux/laravel-compound-plugin sync --target opencode
```

## Philosophy

Built on the **Compound Engineering** philosophy by [Every Inc](https://every.to):

> **Each unit of engineering work should make subsequent units easier—not harder.**

Read more:
- [Compound engineering: how Every codes with agents](https://every.to/chain-of-thought/compound-engineering-how-every-codes-with-agents)
- [The story behind compounding engineering](https://every.to/source-code/my-ai-had-already-fixed-the-code-before-i-saw-it)

## Attribution

This project builds on the [Compound Engineering Plugin](https://github.com/EveryInc/compound-engineering-plugin) created by [Kieran Klaassen](https://github.com/kieranklaassen) at [Every Inc](https://every.to).

## License

MIT
