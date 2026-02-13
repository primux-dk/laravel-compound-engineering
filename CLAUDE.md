# Laravel Compound Engineering Plugin

This repository is a Claude Code plugin marketplace that distributes the `laravel-compound-engineering` plugin — a Laravel companion to the upstream [compound-engineering](https://github.com/EveryInc/compound-engineering-plugin) plugin.

## Repository Structure

```
laravel-compound-engineering-plugin/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace catalog
├── src/                          # CLI tool (convert/install plugins)
├── tests/                        # CLI test suite
└── plugins/
    └── laravel-compound-engineering/
        ├── .claude-plugin/
        │   └── plugin.json       # Plugin metadata
        ├── agents/               # 5 Laravel-specific agents
        ├── skills/               # 8 Laravel/TALL stack skills
        ├── README.md             # Plugin documentation
        └── CHANGELOG.md          # Version history
```

## Philosophy: Compounding Engineering

**Each unit of engineering work should make subsequent units of work easier—not harder.**

When working on this repository, follow the compounding engineering process:

1. **Plan** → Understand the change needed and its impact
2. **Delegate** → Use AI tools to help with implementation
3. **Assess** → Verify changes work as expected
4. **Codify** → Update this CLAUDE.md with learnings

## Claude Code Extension Architecture (v2.1.3+)

> **Important:** As of Claude Code 2.1.3, commands and skills have been unified into a single system.

### The Three Extension Types

| Type | Purpose | Invocation | Context |
|------|---------|------------|---------|
| **Skills** | Reusable knowledge & prompts | `/skill` or auto-discovered | Same conversation |
| **Agents** | Isolated specialized workers | `@agent` or auto-delegated | Separate context |
| **Hooks** | Deterministic enforcement | Automatic on events | N/A |

### Skills: The Unified Approach

Commands and skills are now the same mechanism. The Skill tool handles both user-invoked (`/command`) and model-invoked (auto-discovered) capabilities.

**Structure options:**

```
# Simple skill (single file)
skills/commit.md

# Complex skill (directory with supporting files)
skills/livewire/
├── SKILL.md
├── PATTERNS.md
└── scripts/
```

**Frontmatter controls:**

```yaml
---
name: my-skill
description: What it does and when to use it

# Visibility Controls
user-invocable: true              # Show in / menu (default: true)
disable-model-invocation: false   # Prevent auto-discovery (default: false)

# Tool Restrictions
allowed-tools: Read, Grep, Glob   # Limit available tools (optional)

# Other Options
model: claude-3-5-haiku-20241022  # Override model (optional)
---
```

**Mapping old commands to new skills:**

| Desired Behavior | Frontmatter Configuration |
|------------------|---------------------------|
| Manual `/command` only | `disable-model-invocation: true` |
| Auto-discovered only | `user-invocable: false` |
| Both (hybrid) | Default - no flags needed |

### Agents vs Skills

| Aspect | Skills | Agents |
|--------|--------|--------|
| **Context** | Same conversation | Isolated context window |
| **Effect** | Injects knowledge | Delegates task, returns summary |
| **Tool access** | Inherits or restricts | Fully configurable |
| **Nesting** | Can chain | Cannot spawn other subagents |

**Use agents when you need:**
- Context isolation (exploration shouldn't pollute main conversation)
- Different tool permissions (read-only reviewer, restricted deployer)
- Cost control (route simple tasks to Haiku)
- Parallel/background execution

**Agent frontmatter:**

```yaml
---
name: code-reviewer
description: Review code for quality and security
tools: Read, Grep, Glob       # Allowed tools
model: haiku                  # Model override
skills: skill-1, skill-2     # Inject skills at startup
---
```

### Hooks: Deterministic Enforcement

Hooks are NOT part of the skill unification. They provide deterministic control where CLAUDE.md and skills are suggestions.

```
CLAUDE.md saying "don't edit .env" → Parsed by LLM → Maybe followed
PreToolUse hook blocking .env edits → Always runs → Operation blocked
```

**Hook events:** `PreToolUse`, `PostToolUse`, `Stop`, `SubagentStop`, `Notification`

---

## Working with This Repository

### Updating the Laravel Plugin

When agents or skills are added/removed, follow this checklist:

#### 1. Count all components accurately

```bash
# Count agents
find plugins/laravel-compound-engineering/agents -name "*.md" | wc -l

# Count skills
ls -d plugins/laravel-compound-engineering/skills/*/ 2>/dev/null | wc -l
```

#### 2. Update ALL description strings with correct counts

The description appears in multiple places and must match everywhere:

- [ ] `plugins/laravel-compound-engineering/.claude-plugin/plugin.json` → `description` field
- [ ] `.claude-plugin/marketplace.json` → plugin `description` field
- [ ] `plugins/laravel-compound-engineering/README.md` → intro paragraph

Format: `"X agents and Y skills for Laravel, Livewire, Pest, and TALL stack development."`

#### 3. Update version numbers

Bump the version in:

- [ ] `plugins/laravel-compound-engineering/.claude-plugin/plugin.json` → `version`
- [ ] `.claude-plugin/marketplace.json` → plugin `version`

#### 4. Update documentation

- [ ] `plugins/laravel-compound-engineering/README.md` → list all components
- [ ] `plugins/laravel-compound-engineering/CHANGELOG.md` → document changes

#### 5. Validate JSON files

```bash
jq . .claude-plugin/marketplace.json
jq . plugins/laravel-compound-engineering/.claude-plugin/plugin.json
```

#### 6. Verify before committing

```bash
# Ensure counts match actual files
find plugins/laravel-compound-engineering/agents -name "*.md" | wc -l
ls -d plugins/laravel-compound-engineering/skills/*/ | wc -l
```

### Marketplace.json Structure

The marketplace.json follows the official Claude Code spec:

```json
{
  "name": "marketplace-identifier",
  "owner": {
    "name": "Owner Name",
    "url": "https://github.com/owner"
  },
  "metadata": {
    "description": "Marketplace description",
    "version": "1.0.0"
  },
  "plugins": [
    {
      "name": "plugin-name",
      "description": "Plugin description",
      "version": "1.0.0",
      "author": { ... },
      "homepage": "https://...",
      "tags": ["tag1", "tag2"],
      "source": "./plugins/plugin-name"
    }
  ]
}
```

**Only include fields that are in the official spec.** Do not add custom fields like:

- `downloads`, `stars`, `rating` (display-only)
- `categories`, `featured_plugins`, `trending` (not in spec)
- `type`, `verified`, `featured` (not in spec)

### Plugin.json Structure

Each plugin has its own plugin.json with metadata:

```json
{
  "name": "plugin-name",
  "version": "1.0.0",
  "description": "Plugin description",
  "author": { ... },
  "keywords": ["keyword1", "keyword2"]
}
```

## Testing Changes

### Test Locally

1. Install the marketplace locally:

   ```bash
   claude /plugin marketplace add /path/to/laravel-compound-engineering-plugin
   ```

2. Install the plugin:

   ```bash
   claude /plugin install laravel-compound-engineering
   ```

3. Test agents and skills:
   ```bash
   claude /laravel-setup
   claude agent taylor-otwell-reviewer "test message"
   ```

### Validate JSON

```bash
jq . .claude-plugin/marketplace.json
jq . plugins/laravel-compound-engineering/.claude-plugin/plugin.json
```

## Common Tasks

### Adding a New Agent

1. Create `plugins/laravel-compound-engineering/agents/<category>/new-agent.md`
2. Update plugin.json description with new agent count
3. Update marketplace.json description with new agent count
4. Update README.md agent table
5. Update CHANGELOG.md
6. Test with `claude agent new-agent "test"`

### Adding a New Skill

1. Create skill directory: `plugins/laravel-compound-engineering/skills/skill-name/`
2. Add `SKILL.md` with frontmatter (`name`, `description`)
3. Update plugin.json description with new skill count
4. Update marketplace.json description with new skill count
5. Update README.md skill table
6. Update CHANGELOG.md
7. Test with `claude skill skill-name`

### Updating Tags/Keywords

Tags should reflect the Laravel focus:

- Use: `laravel`, `php`, `livewire`, `pest`, `tall-stack`, `compound-engineering`
- Avoid: Generic workflow tags (those belong to upstream)

## Commit Conventions

Follow these patterns for commit messages:

- `Add [agent/command name]` - Adding new functionality
- `Remove [agent/command name]` - Removing functionality
- `Update [file] to [what changed]` - Updating existing files
- `Fix [issue]` - Bug fixes
- `Simplify [component] to [improvement]` - Refactoring

Include the Claude Code footer:

```
🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

## Resources to search for when needing more information

- [Claude Code Plugin Documentation](https://docs.claude.com/en/docs/claude-code/plugins)
- [Plugin Marketplace Documentation](https://docs.claude.com/en/docs/claude-code/plugin-marketplaces)
- [Plugin Reference](https://docs.claude.com/en/docs/claude-code/plugins-reference)

## Key Learnings

_This section captures important learnings as we work on this repository._

### 2026-02-13: v2.0.0 — Slimmed to Laravel-only companion

Upstream compound-engineering now supports per-project agent configuration via `.local.md`. Removed all generic components (~88% of the plugin) and restructured as a focused Laravel supplement.

**Learning:** When upstream provides generic functionality, don't maintain a fork. Focus on domain-specific expertise (Laravel patterns, Taylor Otwell's style) and let upstream handle workflows.

### 2026-01-17: Claude Code 2.1.3 unified commands and skills

Commands and skills are now the same mechanism — both handled by the Skill tool. Frontmatter controls visibility: `user-invocable` (show in / menu) and `disable-model-invocation` (prevent auto-discovery). Agents remain separate (isolated context). Hooks remain separate (deterministic enforcement).

**Learning:** When creating new commands, prefer creating them as skills with `disable-model-invocation: true` for manual-only behavior.

### 2024-11-22: Component counts must match across files

The counts appear in multiple places (plugin.json, marketplace.json, README.md) and must all match. Always count actual files before updating descriptions.

### 2024-10-09: Simplified marketplace.json to match official spec

**Learning:** Stick to the official spec. Custom fields may confuse users or break compatibility with future versions.
