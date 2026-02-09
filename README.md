# Laravel Compound Engineering Plugin

A Claude Code plugin for Laravel and PHP development, making each unit of engineering work easier than the last.

This is a Laravel-focused fork of the [Compound Engineering Plugin](https://github.com/EveryInc/compound-engineering-plugin) by [Every Inc](https://every.to).

## Install

```bash
/plugin marketplace add https://github.com/primux-dk/laravel-compound-engineering-plugin
/plugin install compound-engineering
```

## Getting Started: Your First Feature

Already have Claude Code installed and a Laravel project ready? Here's how to build your first feature from start to finish.

### 1. Plan your feature

Don't try to build everything at once. Pick one feature and describe it clearly:

```
/workflows:plan
```

Write out what you want to build. The plan command turns your description into a structured implementation plan with steps, files to create, and considerations.

### 2. Review the plan

Read through the generated plan. If something needs adjusting, use the `/deepen-plan` command to enrich it with research, or just ask Claude Code to change specific parts.

### 3. Start building

When the plan looks good, start the work:

```
/workflows:work
```

Ask it to create a **Git worktree** if you plan to keep working on other things in parallel. The work command picks up where the plan left off and executes it step by step.

### 4. Review the code

Once the work is done, review what was built:

```
/workflows:review
```

This runs multiple specialized review agents in parallel — checking architecture, security, performance, and code quality. If you want to track the findings, explicitly ask it to create todos from the review.

### 5. Address findings

Look at the todos from the review. Delete the ones you disagree with. Then use `/triage` to prioritize them, or run `/resolve_parallel` to let Claude Code fix them all in parallel.

### 6. Create a PR and merge

Ask Claude Code to create a pull request. Go to GitHub, skim the code one more time to build your understanding of the architecture, then merge it.

### 7. Clean up and repeat

Delete the worktree, and start the cycle again with your next feature.

```
Plan → Work → Review → Fix → PR → Merge → Repeat
```

### When you have a vague idea

If you're not sure what the feature should look like yet, start with brainstorming instead:

```
/workflows:brainstorm
```

This walks you through questions to clarify what you actually need before committing to a plan. Once the brainstorm is clear, move to `/workflows:plan`.

## Workflow Commands

| Command | Purpose |
|---------|---------|
| `/workflows:brainstorm` | Explore a vague idea before committing to a plan |
| `/workflows:plan` | Turn feature ideas into detailed implementation plans |
| `/workflows:work` | Execute plans with worktrees and task tracking |
| `/workflows:review` | Multi-agent code review before merging |
| `/workflows:compound` | Document learnings to make future work easier |

Each cycle compounds: plans inform future plans, reviews catch more issues, patterns get documented.

## Philosophy

This plugin is built on the **Compound Engineering** philosophy created by [Every Inc](https://every.to):

> **Each unit of engineering work should make subsequent units easier—not harder.**

Traditional development accumulates technical debt. Every feature adds complexity. The codebase becomes harder to work with over time.

Compound engineering inverts this. 80% is in planning and review, 20% is in execution:
- Plan thoroughly before writing code
- Review to catch issues and capture learnings
- Codify knowledge so it's reusable
- Keep quality high so future changes are easy

Read more about the philosophy:
- [Compound engineering: how Every codes with agents](https://every.to/chain-of-thought/compound-engineering-how-every-codes-with-agents)
- [The story behind compounding engineering](https://every.to/source-code/my-ai-had-already-fixed-the-code-before-i-saw-it)

## Learn More

- [Full component reference](plugins/compound-engineering/README.md) - all agents, commands, skills

## Attribution

This project is a fork of the [Compound Engineering Plugin](https://github.com/EveryInc/compound-engineering-plugin) created by [Kieran Klaassen](https://github.com/kieranklaassen) at [Every Inc](https://every.to). We're grateful for their pioneering work in AI-powered development workflows.

## License

MIT
