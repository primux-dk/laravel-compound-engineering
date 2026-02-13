# Changelog

All notable changes to the compound-engineering plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.0.0] - 2026-02-13

### Changed

- **Restructured as Laravel companion plugin** — Users install upstream [compound-engineering](https://github.com/EveryInc/compound-engineering-plugin) for generic workflows, this plugin provides Laravel expertise only

### Added

- **`laravel-setup` skill** — Run `/laravel-setup` to generate `.claude/compound-engineering.local.md` pre-configured with Laravel agents for the upstream plugin's review workflows

### Removed

- **All 27 commands** — Now provided by upstream compound-engineering plugin (`/workflows:plan`, `/workflows:review`, `/lfg`, etc.)
- **21 generic agents** — Now provided by upstream:
  - Design (3): figma-design-sync, design-implementation-reviewer, design-iterator
  - Research (5): best-practices-researcher, framework-docs-researcher, git-history-analyzer, learnings-researcher, repo-research-analyst
  - Review (10): agent-native-reviewer, architecture-strategist, code-simplicity-reviewer, deployment-verification-agent, julik-frontend-races-reviewer, kieran-python-reviewer, kieran-typescript-reviewer, pattern-recognition-specialist, performance-oracle, security-sentinel
  - Workflow (3): bug-reproduction-validator, pr-comment-resolver, spec-flow-analyzer
- **13 generic skills** — Now provided by upstream: agent-browser, agent-native-architecture, brainstorming, compound-docs, create-agent-skills, document-review, file-todos, frontend-design, gemini-imagegen, git-worktree, orchestrating-swarms, rclone, skill-creator
- **Context7 MCP server** — Removed from plugin.json; add manually if needed

### Retained

- **5 agents**: primux-laravel-reviewer, data-integrity-guardian, data-migration-expert, github-code-researcher, lint
- **8 skills**: taylor-otwell-style, laravel-livewire-patterns, laravel-database-patterns, pest-testing, tailwindcss-v4, spatie-laravel-package-writer, web-interface-review, laravel-setup (new)

### Migration

1. Install upstream: `claude /plugin install compound-engineering`
2. Update this plugin: `claude /plugin update laravel-compound-engineering`
3. Run `/laravel-setup` in your Laravel projects to configure review agents

### Summary

- 5 agents, 0 commands, 8 skills

---

## [1.8.1] - 2026-02-10

### Fixed

- **`/deepen-plan` command** - Fix context window exhaustion from unbounded agent spawning
  - Capped total background agents at 8 (was unlimited, commonly spawning 15-40+)
  - Replaced "run ALL agents" instructions with relevance-based filtering per step
  - Agents now write results to `/tmp/deepen-plan/{agent-name}.md` (keeps outputs off orchestrator context until synthesis)
  - Switched from non-blocking `TaskOutput` polling to single blocking calls (eliminates token-wasting poll loops)
  - Combined learnings + research into grouped agents (2-3 instead of one per section/learning)
  - Added domain summary in Step 1 for consistent filtering across all later steps
  - Added agent budget labels per step: Skills (2-3), Research+Learnings (2-3), Review (2-3)

### Summary

- 26 agents, 27 commands, 20 skills, 1 MCP server

---

## [1.8.0] - 2026-02-10

### Added

- **`/triage-prs` command** - Triage all open PRs with parallel agents, label, group, and review one-by-one
  - Detects repo from current directory or `$ARGUMENTS`
  - Batches PRs by theme (bug fixes, features, docs, stale)
  - Spawns parallel review agents per batch
  - Cross-references issues to PRs
  - Compiles triage report with recommended actions
  - Interactive one-by-one merge/comment/close/skip flow
  - Post-triage cleanup and summary

### Changed

- **`create-agent-skills` skill** - Rewrite to document unified commands/skills system
  - Documents that commands and skills are now the same mechanism
  - Adds new frontmatter fields: `disable-model-invocation`, `user-invocable`, `context`, `agent`
  - Adds invocation control table and dynamic context injection docs
  - Adds `$ARGUMENTS` and subagent (`context: fork`) documentation

- **`skill-structure.md` reference** - Fix incorrect XML tags recommendation
  - Was recommending XML tags over markdown headings (wrong)
  - Now correctly recommends standard markdown headings
  - Updated validation checklist and anti-patterns

- **`official-spec.md` reference** - Complete 2026 specification update
  - Documents commands/skills merger
  - Adds complete frontmatter reference with all new fields
  - Adds invocation control, string substitutions, dynamic context injection

### Summary

- 26 agents, 27 commands, 20 skills, 1 MCP server

---

## [1.7.0] - 2026-02-09

### Changed

- **Agent descriptions** - Trimmed 25 verbose agent descriptions (~38,860 → ~5,200 chars) ([upstream PR #161](https://github.com/EveryInc/compound-engineering-plugin/pull/161))
  - Replaced multi-paragraph descriptions with concise 1-2 sentence summaries
  - Moved `<example>` blocks from YAML frontmatter into agent body as `<examples>` tags
  - Affected agents: all design (3), research (6), review (13), and workflow (3) agents

- **Command visibility** - Added `disable-model-invocation: true` to 18 commands with side effects
  - Prevents auto-discovery of orchestrators (`lfg`, `slfg`), diagnostic tools (`doctor`), and side-effect commands (`changelog`, `deploy-docs`, `triage`, etc.)
  - Keeps 8 core workflow commands auto-discoverable (`workflows:plan`, `workflows:work`, `workflows:review`, etc.)

- **Skill visibility** - Added `disable-model-invocation: true` to 5 specialized skills
  - `orchestrating-swarms`, `skill-creator`, `compound-docs`, `file-todos`, `create-agent-skills`
  - Keeps 15 auto-discoverable skills for Laravel, testing, styling, and design patterns

### Summary

- 26 agents, 26 commands, 20 skills, 1 MCP server
- Plugin context reduced from ~46,679 to ~9,800 chars (~79% reduction)

---

## [1.6.0] - 2026-02-09

### Added

- **`sync` command** - Sync personal Claude Code config to OpenCode or Codex ([PR #123](https://github.com/EveryInc/compound-engineering-plugin/pull/123))
  - Parses `~/.claude/skills/` for personal skills (supports symlinks)
  - Parses `~/.claude/settings.json` for MCP servers
  - Syncs skills as symlinks (single source of truth, instant updates)
  - Converts MCP to JSON (OpenCode) or TOML (Codex)

### Fixed

- **Config backup** - Backup existing config files before overwriting ([PR #119](https://github.com/EveryInc/compound-engineering-plugin/pull/119))
  - `backupFile()` utility creates timestamped backups (e.g., `config.toml.bak.2026-02-09T...`)
  - Backs up `config.toml` (Codex) and `opencode.json` (OpenCode) before writing

- **`git-worktree` skill** - Remove confirmation prompt when creating worktrees ([PR #144](https://github.com/EveryInc/compound-engineering-plugin/pull/144))
  - Removes interactive `y/n` prompt that blocked Claude from creating worktrees on first try

- **`git-worktree` skill** - Detect worktrees where `.git` is a file, not a directory ([PR #159](https://github.com/EveryInc/compound-engineering-plugin/pull/159))
  - Changed `-d` to `-e` check — in worktrees `.git` is a file containing a `gitdir:` pointer

### Summary

- 26 agents, 26 commands, 20 skills, 1 MCP server
- CLI bumped to 0.2.0

---

## [1.5.2] - 2026-02-09

### Fixed

- **OpenCode converter** - Fix crash when hook entries have no matcher ([PR #160](https://github.com/EveryInc/compound-engineering-plugin/pull/160))
  - Made `matcher` optional in `ClaudeHookMatcher` type (hooks like `SessionStart` and `SubagentStop` don't need one)
  - Guarded all `matcher.matcher` accesses in `claude-to-opencode.ts` with nullish coalescing

### Summary

- 26 agents, 26 commands, 20 skills, 1 MCP server

---

## [1.5.1] - 2026-02-09

### Fixed

- **`/workflows:compound` command** - Prevent subagents from writing intermediary files ([PR #150](https://github.com/EveryInc/compound-engineering-plugin/pull/150))
  - Restructured from flat parallel strategy to two-phase orchestration (research → assembly)
  - Added `<critical_requirement>` block forbidding subagent file writes
  - Removed Documentation Writer as a subagent; orchestrator handles assembly in Phase 2
  - Added "Common Mistakes to Avoid" table for quick reference
  - Simplified success output

### Summary

- 26 agents, 26 commands, 20 skills, 1 MCP server

---

## [1.5.0] - 2026-02-08

### Added

- **`pest-testing` skill** - Comprehensive Pest 4 testing patterns
  - Feature tests, browser testing, smoke testing
  - Architecture tests for enforcing code conventions
  - Datasets for parameterized tests
  - Mocking patterns (Mail, Queue, Notification fakes)
  - Livewire component testing patterns
  - Specific assertion recommendations (assertSuccessful over assertStatus)

- **`tailwindcss-v4` skill** - Tailwind CSS v4 development patterns
  - CSS-first configuration with `@theme` directive
  - Import syntax change (`@import "tailwindcss"` replaces `@tailwind` directives)
  - Complete deprecated utility replacement table (12 utilities)
  - Gap-over-margins pattern, dark mode, layout patterns
  - Common pitfalls for v3→v4 migration

### Changed

- **`laravel-livewire-patterns` skill** - Upgraded from Livewire v3 to v4
  - Added component formats: Single-File (SFC), Multi-File (MFC), view-based
  - Added `livewire:convert` command documentation
  - Added Islands (`@island`) for isolated update regions
  - Added async actions (`wire:click.async`, `#[Async]`)
  - Added deferred/bundled loading (`defer`, `lazy.bundle`)
  - Added new directives: `wire:sort`, `wire:intersect`, `wire:ref`, `.renderless`, `.preserve-scroll`
  - Added v3→v4 breaking changes section (config renames, wire:model behavior, JS API changes)
  - Added JavaScript interceptor system (`interceptMessage`, `interceptRequest`)
  - Added common pitfalls section
  - Preserved existing Actions pattern and thin components philosophy

- **Flux UI reference** - Added component list and icon guidance
  - Full list of all available Pro components (47 components)
  - Heroicons as default icon set with search guidance
  - Lucide fallback with `php artisan flux:icon` import command

### Summary

- 26 agents, 26 commands, 20 skills, 1 MCP server

---

## [1.4.0] - 2026-02-06

### Added

- **`orchestrating-swarms` skill** - Comprehensive guide to multi-agent orchestration
  - Covers primitives: Agent, Team, Teammate, Leader, Task, Inbox, Message, Backend
  - Documents two spawning methods: subagents vs teammates
  - Explains all 13 TeammateTool operations
  - Includes orchestration patterns: Parallel Specialists, Pipeline, Self-Organizing Swarm
  - Details spawn backends: in-process, tmux, iterm2
  - Provides complete workflow examples
- **`/slfg` command** - Swarm-enabled variant of `/lfg` that uses swarm mode for parallel execution
- **`github-code-researcher` agent** - Search GitHub for open source implementations and patterns
  - Uses `gh search code` to find real-world code examples in popular repos
  - Uses `gh search repos` to discover reference projects by stars, topic, and language
  - Uses `gh search issues`/`prs` to find community solutions and discussions
  - Fetches actual file contents via `gh api` for code analysis
  - Includes trusted organization list (spatie, laravel, filamentphp, etc.)
  - Complements best-practices-researcher (articles) and framework-docs-researcher (docs)

### Changed

- **`/workflows:work` command** - Added optional Swarm Mode section for parallel execution with coordinated agents

---

## [1.3.4] - 2026-01-28

### Improved

- **`/workflows:brainstorm` command** - Better question flow
  - Added "Ask more questions" option at handoff phase
  - Clarified that Claude should ask the user questions (not wait for user to ask)
  - Added requirement to resolve ALL open questions before offering to proceed to planning

---

## [1.3.3] - 2026-01-22

### Added

- **`learnings-researcher` agent** - Search institutional learnings in `docs/solutions/` ([PR #106](https://github.com/EveryInc/compound-engineering-plugin/pull/106))
  - Grep-first filtering strategy for efficiency with 100+ files
  - Category-based narrowing to reduce search scope
  - Parallel Grep calls with synonym support (OR patterns)
  - Case-insensitive matching with `-i=true`
  - Frontmatter-only reads before full document reads
  - Always checks critical patterns file
  - Uses `haiku` model for speed

### Changed

- **`/workflows:plan`** - Added learnings-researcher to local research phase ([PR #106](https://github.com/EveryInc/compound-engineering-plugin/pull/106))
  - Now runs repo-research-analyst AND learnings-researcher in parallel
  - Consolidates institutional learnings (gotchas, patterns) into research output

- **Search tool standardization** - Updated agents to use built-in Grep tool ([PR #106](https://github.com/EveryInc/compound-engineering-plugin/pull/106))
  - `repo-research-analyst`: Use built-in Grep/Glob/Read tools, add TypeScript ast-grep example
  - `pattern-recognition-specialist`: Use built-in Grep tool, keep ast-grep mention generic

### Fixed

- **`compound-docs` skill** - Fixed severity enum from `[critical, moderate, minor]` to `[critical, high, medium, low]`

### Summary

- 25 agents, 25 commands, 16 skills, 1 MCP server

---

## [1.3.2] - 2026-01-22

### Changed

- **Standardized filename conventions** for plans and brainstorms ([PR #105](https://github.com/EveryInc/compound-engineering-plugin/pull/105))
  - Plans: `docs/plans/YYYY-MM-DD-<type>-<name>-plan.md` (added date prefix and `-plan` suffix)
  - Brainstorms: `docs/brainstorms/YYYY-MM-DD-<topic>-brainstorm.md` (added `-brainstorm` suffix)
  - Added YAML frontmatter (title, type, date) to all plan templates
  - Updated `/deepen-plan` to use new paths and deepened file naming
  - Updated brainstorming skill with output location

### Summary

- 24 agents, 25 commands, 16 skills, 1 MCP server

---

## [1.3.1] - 2026-01-22

### Added

- **OpenCode/Codex CLI Tooling** - Convert Claude Code plugins to other AI assistant formats
  - `laravel-compound-plugin convert` - Convert plugins to OpenCode or Codex format
  - `laravel-compound-plugin install` - Install and convert in one step
  - `laravel-compound-plugin list` - List available plugins
  - Supports OpenCode (opencode.json, agents/, plugins/, skills/)
  - Supports OpenAI Codex (~/.codex/, prompts/, skills/, config.toml, AGENTS.md)
  - Full test suite with fixtures
  - Based on [PR #104](https://github.com/EveryInc/compound-engineering-plugin/pull/104)

### Summary

- 24 agents, 25 commands, 16 skills, 1 MCP server
- CLI tooling for multi-assistant support

---

## [1.3.0] - 2026-01-21

### Added

- `/workflows:brainstorm` command - Explore WHAT to build before planning HOW
  - Collaborative discovery through dialogue
  - Proposes 2-3 approaches with trade-offs
  - Outputs to `docs/brainstorms/YYYY-MM-DD-<topic>.md`
  - Integrates with `/workflows:plan` (auto-detects brainstorm context)

- `brainstorming` skill - Questioning techniques and YAGNI principles
  - Five Whys technique for root cause discovery
  - Problem framing, scope, and approach questions
  - YAGNI red flags and counter-questions
  - Approach exploration pattern with trade-off analysis

- `laravel-livewire-patterns` skill - Comprehensive patterns for Livewire v3, Form Objects, and Flux UI
  - Component structure, lifecycle, attributes (#[Title], #[Lazy], #[On], etc.)
  - Form Objects for validation and data handling
  - Flux UI Pro components (forms, tables, modals, navigation)
  - Real-time features (live search, polling, events)
  - Reference files: `components.md`, `form-objects.md`, `flux-ui.md`

- `laravel-database-patterns` skill - Database operations, migrations, transactions, and Eloquent optimization
  - Migration patterns with naming conventions and column types
  - Transaction handling (closure, manual, retries, savepoints, locking)
  - Eloquent best practices (scopes, accessors, events, relationships)
  - Query optimization (eager loading, N+1 prevention, chunking)
  - Reference files: `migrations.md`, `transactions.md`, `eloquent.md`

### Changed

- `/workflows:plan` - Add interactive refinement for better planning
  - **Idea Refinement**: Before research, ask clarifying questions using AskUserQuestion
  - **Research Validation**: After research, summarize findings and validate alignment
  - Skip option when description is already detailed
  - Inspired by [PR #88](https://github.com/EveryInc/compound-engineering-plugin/pull/88)

- `/workflows:work` - Merge improvements from upstream [PR #93](https://github.com/EveryInc/compound-engineering-plugin/pull/93)
  - **Branch Detection**: Smart detection of current branch and repository default (main/master)
  - **Contextual Setup**: Different guidance based on whether you're on a feature branch or default branch
  - **Safety Guard**: Never commit directly to default branch without explicit user permission
  - **Incremental Commits**: New section with decision matrix for when to commit
  - Renumbered Phase 2 steps to accommodate new section

- `/workflows:plan` - Unified output paths and brainstorm integration
  - Output path changed: `plans/` → `docs/plans/`
  - Detects and uses existing brainstorm documents
  - Suggests `/workflows:brainstorm` for complex/unclear requirements

- `/workflows:plan` - Smart research decision logic ([PR #100](https://github.com/EveryInc/compound-engineering-plugin/pull/100))
  - **Step 1.5 Research Decision**: Decides on external research based on context signals
  - High-risk topics (security, payments, APIs) → always research
  - Strong local context → skip external research
  - Uncertainty or new territory → research
  - Repo research (Step 1) always runs first (fast, local)
  - External research (Step 1.5b) is now conditional

- `/workflows:review` and `git-worktree` skill - Clearer branch detection wording ([PR #104](https://github.com/EveryInc/compound-engineering-plugin/pull/104))
  - Changed "PR branch" to "target branch (PR branch or requested branch)"
  - Clarifies behavior when reviewing specific branches vs PRs

- Research agents - Mandatory API deprecation validation ([PR #102](https://github.com/EveryInc/compound-engineering-plugin/pull/102))
  - `framework-docs-researcher` - Added step 2: MANDATORY Deprecation Check
  - `best-practices-researcher` - Added Phase 1.5: MANDATORY Deprecation Check
  - Prevents recommending deprecated APIs (e.g., Google Photos API scopes deprecated March 2025)
  - "5 minutes of validation saves hours debugging dead APIs"

- Enhanced `taylor-otwell-style` skill
  - Stronger emphasis on Actions pattern and thin controllers
  - Added i18n/translation patterns reference (`i18n.md`)
  - Updated routing to include i18n as option 8

- Updated agents to use Laravel skills
  - `primux-laravel-reviewer` now includes: `taylor-otwell-style`, `laravel-livewire-patterns`, `laravel-database-patterns`
  - `data-integrity-guardian` now includes: `laravel-database-patterns`
  - `data-migration-expert` now includes: `laravel-database-patterns`

### Summary

- 24 agents, 25 commands, 16 skills, 1 MCP server

---

## [1.2.0] - 2026-01-18

### Added

- `web-interface-review` skill - Review web UI code against Vercel's Web Interface Guidelines
  - Checks accessibility, focus states, forms, animations, typography
  - Validates performance patterns, navigation, touch interactions, i18n
  - Outputs findings in VS Code-compatible `file:line` format
  - Full reference documentation in `references/guidelines.md`

### Summary

- 24 agents, 20 commands, 13 skills, 1 MCP server

---

## [1.1.0] - 2026-01-18

### Added

- `/plugin:doctor` command - Check plugin requirements and install missing dependencies
  - Checks core requirements: PHP, Composer, Node.js, npm, Python, pip
  - Checks Context7 MCP server configuration
  - Checks skill-specific tools: agent-browser, rclone, GEMINI_API_KEY, google-genai, pillow
  - Checks Laravel project tools: Pint, PEST, PHPStan
  - Offers to configure Context7 MCP and install missing dependencies interactively

### Summary

- 24 agents, 20 commands, 12 skills, 1 MCP server

---

## [1.0.0] - 2026-01-18

### Changed

This release converts the plugin from Ruby/Rails to Laravel/PHP.

**Skills:**
- Replaced `dhh-rails-style` with `taylor-otwell-style` - Laravel/PHP coding conventions based on Taylor Otwell's philosophy
- Replaced `andrew-kane-gem-writer` with `spatie-laravel-package-writer` - Laravel package development following Spatie patterns

**Agents:**
- Replaced `dhh-rails-reviewer` with `primux-laravel-reviewer` - Laravel code review from Taylor Otwell's perspective
- Updated `lint` agent - Now uses Laravel Pint, PHPStan, and composer audit (only lints changed files)
- Updated `framework-docs-researcher` - Uses composer instead of bundler
- Updated `performance-oracle` - Eloquent optimization, queue jobs
- Updated `security-sentinel` - Laravel request validation, fillable/guarded, @csrf
- Updated `data-migration-expert` - Artisan commands instead of rake tasks
- Updated `deployment-verification-agent` - Artisan migrate, Eloquent/Tinker examples
- Updated `bug-reproduction-validator` - Laravel app references

**Commands:**
- Updated `review.md` - Removed rails-turbo-expert, uses primux-laravel-reviewer
- Updated `deepen-plan.md` - Simplified skill references (no paths needed)
- Updated `compound.md`, `work.md`, `plan_review.md` - Laravel reviewer references

### Removed

- `kieran-rails-reviewer` agent - primux-laravel-reviewer covers Laravel
- `ankane-readme-writer` agent - spatie-laravel-package-writer covers Laravel packages
- `dhh-rails-style` skill directory
- `rails-turbo-expert` reference

### Summary

- 24 agents, 23 commands, 12 skills, 1 MCP server

---

## Attribution

This plugin is forked from the [Compound Engineering Plugin](https://github.com/EveryInc/compound-engineering-plugin) by Every Inc, created by Kieran Klaassen. The Laravel adaptations build upon their pioneering work in AI-powered development workflows.
