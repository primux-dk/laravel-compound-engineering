---
name: laravel-setup
description: This skill should be used to configure a Laravel project for use with the compound-engineering plugin's review workflows. Generates compound-engineering.local.md with Laravel-specific agent configuration.
user-invocable: true
disable-model-invocation: true
allowed-tools: Read, Write, Bash, Glob, Grep
---

# Laravel Setup for Compound Engineering

Configure this Laravel project to use Laravel-specific review agents with the upstream compound-engineering plugin.

## Step 1: Check Existing Config

Read `compound-engineering.local.md` in the project root. If it exists, display current settings and use AskUserQuestion:

```
question: "Settings file already exists. What would you like to do?"
header: "Config"
options:
  - label: "Reconfigure"
    description: "Overwrite with Laravel defaults"
  - label: "View current"
    description: "Show the file contents, then stop"
  - label: "Cancel"
    description: "Keep current settings"
```

If "View current": read and display the file, then stop.
If "Cancel": stop.

## Step 2: Detect Laravel

Confirm this is a Laravel project by checking for ANY of:
- `artisan` file in the project root
- `composer.json` containing `laravel/framework`

If neither is found, warn the user and ask whether to proceed anyway.

## Step 3: Write Config

Write `compound-engineering.local.md` in the **project root** (not `.claude/`) with YAML frontmatter:

```markdown
---
review_agents: [primux-laravel-reviewer, data-integrity-guardian, data-migration-expert, github-code-researcher, lint]
plan_review_agents: [primux-laravel-reviewer, code-simplicity-reviewer]
---

# Review Context

Laravel project using the TALL stack (Tailwind CSS, Alpine.js, Livewire, Laravel).

Key conventions:
- Follow Taylor Otwell's coding style (Actions pattern, thin controllers, expressive APIs)
- Use Pest for testing
- Use Laravel Pint for code formatting
- Database changes require migration review by data-integrity-guardian
- All code review should consider Laravel conventions and best practices
```

## Step 4: Confirm

```
Saved to compound-engineering.local.md

Stack:        Laravel (TALL)
Review depth: Thorough
Agents:       5 configured
              primux-laravel-reviewer
              data-integrity-guardian
              data-migration-expert
              github-code-researcher
              lint

Plan review:  primux-laravel-reviewer
              code-simplicity-reviewer

Tip: Edit the "Review Context" section to add project-specific instructions.
     Re-run /laravel-setup anytime to reconfigure.
```
