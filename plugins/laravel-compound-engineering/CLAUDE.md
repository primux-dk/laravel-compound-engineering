# Laravel Compound Engineering Plugin Development

## Versioning Requirements

**IMPORTANT**: Every change to this plugin MUST include updates to all three files:

1. **`.claude-plugin/plugin.json`** - Bump version using semver
2. **`CHANGELOG.md`** - Document changes using Keep a Changelog format
3. **`README.md`** - Verify/update component counts and tables

### Version Bumping Rules

- **MAJOR** (1.0.0 → 2.0.0): Breaking changes, major reorganization
- **MINOR** (1.0.0 → 1.1.0): New agents or skills
- **PATCH** (1.0.0 → 1.0.1): Bug fixes, doc updates, minor improvements

### Pre-Commit Checklist

Before committing ANY changes:

- [ ] Version bumped in `.claude-plugin/plugin.json`
- [ ] CHANGELOG.md updated with changes
- [ ] README.md component counts verified
- [ ] README.md tables accurate (agents, skills)
- [ ] plugin.json description matches current counts

### Directory Structure

```
agents/
├── research/   # Research agents
├── review/     # Code review agents
└── workflow/   # Workflow automation agents

skills/
└── */          # Skill directories with SKILL.md
```

## Skill Compliance Checklist

When adding or modifying skills, verify compliance:

### YAML Frontmatter (Required)

- [ ] `name:` present and matches directory name (lowercase-with-hyphens)
- [ ] `description:` present and uses **third person** ("This skill should be used when...")

### Reference Links (Required if references/ exists)

- [ ] All files in `references/` are linked as `[filename.md](./references/filename.md)`
- [ ] No bare backtick references like `` `references/file.md` `` - use proper markdown links

### Writing Style

- [ ] Use imperative/infinitive form (verb-first instructions)
- [ ] Avoid second person ("you should") - use objective language

## Documentation

See `docs/solutions/plugin-versioning-requirements.md` for detailed versioning workflow.
