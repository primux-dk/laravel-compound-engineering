---
name: laravel-setup
description: This skill should be used to configure a Laravel project for use with the compound-engineering plugin's review workflows. Generates .claude/compound-engineering.local.md with Laravel-specific agent configuration.
user-invocable: true
disable-model-invocation: true
allowed-tools: Read, Write, Bash, Glob, Grep
---

# Laravel Setup for Compound Engineering

Configure this Laravel project to use Laravel-specific review agents with the upstream compound-engineering plugin.

## Prerequisites

Verify the upstream compound-engineering plugin is installed. Check for its agents directory:

```bash
ls ~/.claude/plugins/compound-engineering/agents/ 2>/dev/null
```

If not found, inform the user:

> The upstream **compound-engineering** plugin is required. Install it first:
> ```
> claude /plugin install compound-engineering
> ```
> Then run `/laravel-setup` again.

## Detection

Confirm this is a Laravel project by checking for ANY of:
- `artisan` file in the project root
- `composer.json` containing `laravel/framework`

If neither is found, warn the user and ask whether to proceed anyway.

## Configuration

Create `.claude/compound-engineering.local.md` in the project root with this content:

```markdown
# Laravel Project Configuration

## Review Agents

Use these Laravel-specialized agents for code review workflows:

review_agents:
  - taylor-otwell-reviewer
  - data-integrity-guardian
  - data-migration-expert
  - github-code-researcher
  - lint

plan_review_agents:
  - taylor-otwell-reviewer

## Review Context

This is a Laravel project using the TALL stack (Tailwind CSS, Alpine.js, Livewire, Laravel).

Key conventions:
- Follow Taylor Otwell's coding style (Actions pattern, thin controllers, expressive APIs)
- Use Pest for testing
- Use Laravel Pint for code formatting
- Database changes require migration review by data-integrity-guardian
- All code review should consider Laravel conventions and best practices
```

## Post-Setup

After creating the file, confirm success:

> Created `.claude/compound-engineering.local.md`
>
> The upstream compound-engineering plugin will now use Laravel-specific agents for review workflows.
>
> **Agents configured:**
> - `taylor-otwell-reviewer` — Laravel code review (Taylor Otwell's perspective)
> - `data-integrity-guardian` — Database migration and integrity review
> - `data-migration-expert` — Data migration validation
> - `github-code-researcher` — Open source pattern research
> - `lint` — PHP/Blade linting with Laravel Pint
