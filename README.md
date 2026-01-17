# Laravel Compound Engineering Plugin

A Claude Code plugin for Laravel and PHP development, making each unit of engineering work easier than the last.

This is a Laravel-focused fork of the [Compound Engineering Plugin](https://github.com/EveryInc/compound-engineering-plugin) by [Every Inc](https://every.to).

## Install

```bash
/plugin marketplace add https://github.com/primux-dk/laravel-compound-engineering-plugin
/plugin install compound-engineering
```

## Workflow

```
Plan → Work → Review → Compound → Repeat
```

| Command | Purpose |
|---------|---------|
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
