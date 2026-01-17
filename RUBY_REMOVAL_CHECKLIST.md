# Ruby Removal Checklist

Files containing Ruby/Rails references that need to be updated to Laravel/PHP.

## Progress: 19/42 files completed

---

## ✅ Skills (dhh-rails-style) - REPLACED WITH taylor-otwell-style
These have been replaced with Laravel equivalents in `skills/taylor-otwell-style/`:

- [x] `plugins/compound-engineering/skills/dhh-rails-style/SKILL.md` → **REPLACED**: `skills/taylor-otwell-style/SKILL.md`
- [x] `plugins/compound-engineering/skills/dhh-rails-style/references/architecture.md` → **REPLACED**: `skills/taylor-otwell-style/references/architecture.md`
- [x] `plugins/compound-engineering/skills/dhh-rails-style/references/controllers.md` → **REPLACED**: `skills/taylor-otwell-style/references/controllers.md`
- [x] `plugins/compound-engineering/skills/dhh-rails-style/references/frontend.md` → **REPLACED**: `skills/taylor-otwell-style/references/frontend.md`
- [x] `plugins/compound-engineering/skills/dhh-rails-style/references/gems.md` → **REPLACED**: `skills/taylor-otwell-style/references/packages.md`
- [x] `plugins/compound-engineering/skills/dhh-rails-style/references/models.md` → **REPLACED**: `skills/taylor-otwell-style/references/models.md`
- [x] `plugins/compound-engineering/skills/dhh-rails-style/references/testing.md` → **REPLACED**: `skills/taylor-otwell-style/references/testing.md`

**✅ DELETED**: `skills/dhh-rails-style/` directory removed

## Agents - Rails-Specific (CONSIDER REMOVING/REPLACING)

- [x] `plugins/compound-engineering/agents/review/kieran-rails-reviewer.md` → **DELETED** (no replacement needed, taylor-otwell-reviewer covers Laravel)
- [x] `plugins/compound-engineering/agents/review/dhh-rails-reviewer.md` → **REPLACED**: `agents/review/taylor-otwell-reviewer.md`
- [x] `plugins/compound-engineering/agents/docs/ankane-readme-writer.md` → **DELETED** (spatie-laravel-package-writer covers Laravel packages)

## ✅ Agents - General (updated Ruby examples to PHP/Laravel)

- [x] `plugins/compound-engineering/agents/research/best-practices-researcher.md` → **UPDATED** (already had Laravel references)
- [x] `plugins/compound-engineering/agents/research/framework-docs-researcher.md` → **UPDATED** (composer, packages)
- [x] `plugins/compound-engineering/agents/research/repo-research-analyst.md` → **UPDATED** (Action class example)
- [x] `plugins/compound-engineering/agents/review/performance-oracle.md` → **UPDATED** (Eloquent, queue jobs)
- [x] `plugins/compound-engineering/agents/review/security-sentinel.md` → **UPDATED** (Laravel request, fillable/guarded)
- [x] `plugins/compound-engineering/agents/review/data-integrity-guardian.md` → **NO CHANGES NEEDED** (generic database concepts)
- [x] `plugins/compound-engineering/agents/review/data-migration-expert.md` → **UPDATED** (artisan commands)
- [x] `plugins/compound-engineering/agents/review/deployment-verification-agent.md` → **UPDATED** (artisan migrate, Eloquent)
- [x] `plugins/compound-engineering/agents/workflow/bug-reproduction-validator.md` → **UPDATED** (Laravel apps)
- [x] `plugins/compound-engineering/agents/workflow/lint.md` → **UPDATED** (Pint, PHPStan, composer audit)

## Commands (update Ruby examples to PHP/Laravel)

- [ ] `plugins/compound-engineering/commands/workflows/compound.md`
- [ ] `plugins/compound-engineering/commands/workflows/review.md`
- [ ] `plugins/compound-engineering/commands/workflows/work.md`
- [ ] `plugins/compound-engineering/commands/deepen-plan.md`
- [ ] `plugins/compound-engineering/commands/reproduce-bug.md`
- [ ] `plugins/compound-engineering/commands/test-browser.md`
- [ ] `plugins/compound-engineering/commands/triage.md`
- [ ] `plugins/compound-engineering/commands/feature-video.md`
- [ ] `plugins/compound-engineering/commands/generate_command.md`
- [ ] `plugins/compound-engineering/commands/plan_review.md`

## Skills - Other (update Ruby examples to PHP/Laravel)

- [ ] `plugins/compound-engineering/skills/file-todos/SKILL.md`
- [ ] `plugins/compound-engineering/skills/file-todos/assets/todo-template.md`
- [ ] `plugins/compound-engineering/skills/compound-docs/SKILL.md`
- [ ] `plugins/compound-engineering/skills/compound-docs/assets/resolution-template.md`
- [ ] `plugins/compound-engineering/skills/compound-docs/references/yaml-schema.md`
- [ ] `plugins/compound-engineering/skills/compound-docs/schema.yaml`
- [ ] `plugins/compound-engineering/skills/agent-native-architecture/SKILL.md`
- [ ] `plugins/compound-engineering/skills/agent-native-architecture/references/mcp-tool-design.md`
- [ ] `plugins/compound-engineering/skills/agent-native-architecture/references/self-modification.md`
- [ ] `plugins/compound-engineering/skills/agent-native-architecture/references/from-primitives-to-domain-tools.md`

## Documentation

- [ ] `plugins/compound-engineering/README.md`
- [ ] `plugins/compound-engineering/CHANGELOG.md`
- [ ] `CLAUDE.md`

---

## Notes

- **dhh-rails-style skill**: This is DHH's Rails style guide. Consider replacing with a Laravel style guide (e.g., Taylor Otwell's style or Spatie conventions).
- **kieran-rails-reviewer**: Rails-specific reviewer. Consider creating a Laravel equivalent.
- **dhh-rails-reviewer**: Rails-specific reviewer. Consider creating a Laravel equivalent.
- **ankane-readme-writer**: Andrew Kane's Ruby gem README style. Already have `spatie-laravel-package-writer` as replacement.

## Decisions Needed

1. ~~Remove `dhh-rails-style` skill entirely or replace with Laravel equivalent?~~ ✅ **DONE**: Replaced with `taylor-otwell-style`
2. ~~Remove `kieran-rails-reviewer` agent or create `kieran-laravel-reviewer`?~~ ✅ **DONE**: Deleted (taylor-otwell-reviewer covers Laravel)
3. ~~Remove `dhh-rails-reviewer` agent or create Laravel equivalent (taylor-otwell-reviewer)?~~ ✅ **DONE**: Replaced with `taylor-otwell-reviewer`
4. ~~Keep `ankane-readme-writer` (for Ruby projects) or remove since we have `spatie-laravel-package-writer`?~~ ✅ **DONE**: Deleted (spatie-laravel-package-writer covers Laravel)
