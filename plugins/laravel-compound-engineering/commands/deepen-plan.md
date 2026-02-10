---
name: deepen-plan
description: Enhance a plan with parallel research agents for each section to add depth, best practices, and implementation details
argument-hint: "[path to plan file]"
---

# Deepen Plan - Power Enhancement Mode

## Introduction

**Note: The current year is 2026.** Use this when searching for recent documentation and best practices.

This command takes an existing plan (from `/workflows:plan`) and enhances each section with parallel research agents. Each major element gets its own dedicated research sub-agent to find:
- Best practices and industry patterns
- Performance optimizations
- UI/UX improvements (if applicable)
- Quality enhancements and edge cases
- Real-world implementation examples

The result is a deeply grounded, production-ready plan with concrete implementation details.

> **CRITICAL CONSTRAINT: Spawn a MAXIMUM of 8 total background agents across ALL steps combined.**
> More agents = context window exhaustion before synthesis. Filter ruthlessly for relevance.
> Each agent MUST write its output to `/tmp/deepen-plan/{agent-name}.md` so results stay off the orchestrator's context until synthesis.

## Agent Output Protocol

Before spawning any agents, create the output directory:

```bash
mkdir -p /tmp/deepen-plan
```

Every agent prompt MUST include this instruction:
```
IMPORTANT: Write your COMPLETE findings to /tmp/deepen-plan/{agent-name}.md
Start the file with a # heading summarizing your role, then write all findings.
Your terminal output should be a brief 1-2 sentence summary only.
```

## Plan File

<plan_path> #$ARGUMENTS </plan_path>

**If the plan path above is empty:**
1. Check for recent plans: `ls -la docs/plans/`
2. Ask the user: "Which plan would you like to deepen? Please provide the path (e.g., `docs/plans/2026-01-15-feat-my-feature-plan.md`)."

Do not proceed until you have a valid plan file path.

## Main Tasks

### 1. Parse and Analyze Plan Structure

<thinking>
First, read and parse the plan to identify each major section that can be enhanced with research.
</thinking>

**Read the plan file and extract:**
- [ ] Overview/Problem Statement
- [ ] Proposed Solution sections
- [ ] Technical Approach/Architecture
- [ ] Implementation phases/steps
- [ ] Code examples and file references
- [ ] Acceptance criteria
- [ ] Any UI/UX components mentioned
- [ ] Technologies/frameworks mentioned (Laravel, Livewire, Alpine.js, Tailwind, Inertia, etc.)
- [ ] Domain areas (data models, APIs, UI, security, performance, etc.)

**Create a section manifest:**
```
Section 1: [Title] - [Brief description of what to research]
Section 2: [Title] - [Brief description of what to research]
...
```

**Create a domain summary** (used for agent filtering in later steps):
```
Primary domains: [e.g., Laravel API, database, frontend]
Technologies: [e.g., Laravel 11, Livewire 4, Tailwind v4]
Concerns: [e.g., performance, security, UI/UX]
```

### 2. Discover and Apply Available Skills (Budget: 2-3 agents)

<thinking>
Discover available skills and match ONLY the most relevant ones to the plan. Cap at 3 skill agents maximum.
</thinking>

**Step 1: Discover ALL available skills from ALL sources**

```bash
# 1. Project-local skills (highest priority - project-specific)
ls .claude/skills/

# 2. User's global skills (~/.claude/)
ls ~/.claude/skills/

# 3. compound-engineering plugin skills
ls ~/.claude/plugins/cache/*/compound-engineering/*/skills/

# 4. ALL other installed plugins - check every plugin for skills
find ~/.claude/plugins/cache -type d -name "skills" 2>/dev/null

# 5. Also check installed_plugins.json for all plugin locations
cat ~/.claude/plugins/installed_plugins.json
```

**Step 2: For each discovered skill, read its SKILL.md to understand what it does**

```bash
# For each skill directory found, read its documentation
cat [skill-path]/SKILL.md
```

**Step 3: Rank skills by relevance to the plan's domain summary**

Score each skill against the plan:
- **High relevance:** Skill's domain directly matches a primary plan technology or concern
- **Medium relevance:** Skill overlaps with a secondary plan concern
- **Low relevance:** Skill's domain doesn't intersect with the plan

**Step 4: Spawn agents for the TOP 2-3 most relevant skills only**

Select the 2-3 highest-relevance skills. Skip everything rated "low relevance."

For each selected skill, spawn a background agent:
```
Task general-purpose (run_in_background: true): "You have the [skill-name] skill available at [skill-path].

YOUR JOB: Use this skill on the plan.

1. Read the skill: cat [skill-path]/SKILL.md
2. Follow the skill's instructions exactly
3. Apply the skill to this content:

[relevant plan section or full plan]

4. Write your COMPLETE findings to /tmp/deepen-plan/skill-[skill-name].md
5. Print a 1-2 sentence summary to stdout.

The skill tells you what to do - follow it. Execute the skill completely."
```

**Example selection for a Laravel API plan:**
```
# Selected (high relevance):
- taylor-otwell-style (Laravel code patterns) → agent 1
- laravel-database-patterns (DB optimization) → agent 2

# Skipped (low relevance for an API plan):
- frontend-design (no frontend in plan)
- gemini-imagegen (no image generation)
- rclone (no file uploads)
```

### 3. Discover and Apply Learnings + Launch Research (Budget: 2-3 agents)

<thinking>
Combine learnings discovery and per-section research into fewer agents to stay within budget. Group related plan sections into 1-2 research prompts.
</thinking>

**Step 1: Check for documented learnings**

```bash
# PRIMARY LOCATION - Project learnings
find docs/solutions -name "*.md" -type f 2>/dev/null

# If docs/solutions doesn't exist, check alternate locations:
find .claude/docs -name "*.md" -type f 2>/dev/null
find ~/.claude/docs -name "*.md" -type f 2>/dev/null
```

**Step 2: Read frontmatter to filter learnings**

Each learning file has YAML frontmatter. Read the first ~20 lines to get tags, category, module, and root_cause. Compare against the plan's domain summary.

**SKIP learnings that are clearly not applicable:**
- Plan is frontend-only → skip `database-migrations/` learnings
- Plan is Python → skip `laravel-specific/` learnings
- Plan has no auth → skip `authentication-issues/` learnings

**Step 3: Combine learnings + research into grouped agents**

Instead of one agent per learning and one per plan section, group them:

**Agent A - Research + Learnings for the plan's primary domain:**
```
Task Explore (run_in_background: true): "Research best practices for: [primary plan domain].

ALSO review these learnings files for applicable insights:
[list of 2-5 relevant learning file paths]

For each learning, read the full file and extract applicable insights.

For the research, find:
- Industry standards and conventions
- Performance considerations
- Common pitfalls and how to avoid them
- Real-world implementation examples

Write your COMPLETE findings to /tmp/deepen-plan/research-primary.md
Print a 1-2 sentence summary to stdout."
```

**Agent B - Research for secondary concerns (if the plan has multiple distinct domains):**
```
Task Explore (run_in_background: true): "Research best practices for: [secondary plan concerns, e.g., UI/UX, security, testing].

Cover these plan sections:
[list of 2-3 related section summaries]

Find concrete recommendations, code patterns, and edge cases.

Write your COMPLETE findings to /tmp/deepen-plan/research-secondary.md
Print a 1-2 sentence summary to stdout."
```

**Also use Context7 MCP for framework documentation (runs in orchestrator, no agent needed):**

For any technologies/frameworks mentioned in the plan, query Context7 directly:
```
mcp__plugin_compound-engineering_context7__resolve-library-id: Find library ID for [framework]
mcp__plugin_compound-engineering_context7__query-docs: Query documentation for specific patterns
```

### 4. Run Targeted Review Agents (Budget: 2-3 agents)

<thinking>
Select the 2-3 most relevant review agents based on the plan's domain. Skip agents whose expertise doesn't overlap with the plan.
</thinking>

**Step 1: Discover available agents**

```bash
# 1. Project-local agents
find .claude/agents -name "*.md" 2>/dev/null

# 2. User's global agents
find ~/.claude/agents -name "*.md" 2>/dev/null

# 3. compound-engineering plugin agents (review + research categories)
find ~/.claude/plugins/cache/*/compound-engineering/*/agents/review -name "*.md" 2>/dev/null
find ~/.claude/plugins/cache/*/compound-engineering/*/agents/research -name "*.md" 2>/dev/null
find ~/.claude/plugins/cache/*/compound-engineering/*/agents/design -name "*.md" 2>/dev/null

# 4. ALL other installed plugins
find ~/.claude/plugins/cache -path "*/agents/*.md" 2>/dev/null
```

**For compound-engineering plugin specifically:**
- USE: `agents/review/*` (reviewers)
- USE: `agents/research/*` (researchers)
- USE: `agents/design/*` (design agents)
- SKIP: `agents/workflow/*` (workflow orchestrators)
- SKIP: `agents/docs/*` (not needed for plan deepening)

**Step 2: Select the 2-3 most relevant review agents**

Match agents to the plan's domain summary:

| Plan Domain | Best Review Agents |
|---|---|
| Laravel backend | taylor-otwell-reviewer, architecture-strategist |
| Database/migrations | data-integrity-guardian, data-migration-expert |
| Security/auth | security-sentinel |
| Performance | performance-oracle |
| Frontend/JS | julik-frontend-races-reviewer |
| API design | architecture-strategist |
| Agent/AI features | agent-native-reviewer |
| General code quality | code-simplicity-reviewer, pattern-recognition-specialist |

Pick the 2-3 agents whose expertise best matches the plan. Skip the rest.

**Step 3: Launch selected agents in parallel (background)**

For each selected agent:
```
Task [agent-type] (run_in_background: true): "Review this plan using your expertise. Apply all your checks and patterns.

Plan content:
---
[full plan content]
---

Write your COMPLETE review findings to /tmp/deepen-plan/review-[agent-name].md
Print a 1-2 sentence summary to stdout."
```

**Launch all selected review agents in a SINGLE message with multiple Task tool calls.**

### 5. Wait for ALL Agents and Synthesize from Files

<thinking>
All agents write to /tmp/deepen-plan/. Wait for them to finish using blocking TaskOutput, then read the files to synthesize.
</thinking>

**Step 1: Wait for all background agents to complete**

For each spawned agent, use a single **blocking** `TaskOutput` call (do NOT poll with non-blocking calls):
```
TaskOutput(task_id=agent_task_id, block=true, timeout=300000)
```

**Step 2: Read all result files**

```bash
ls /tmp/deepen-plan/
```

Then read each result file:
```
Read: /tmp/deepen-plan/skill-taylor-otwell-style.md
Read: /tmp/deepen-plan/research-primary.md
Read: /tmp/deepen-plan/research-secondary.md
Read: /tmp/deepen-plan/review-architecture-strategist.md
... etc
```

**Step 3: Synthesize findings**

**For each agent's findings, extract:**
- [ ] Concrete recommendations (actionable items)
- [ ] Code patterns and examples (copy-paste ready)
- [ ] Anti-patterns to avoid (warnings)
- [ ] Performance considerations (metrics, benchmarks)
- [ ] Security considerations (vulnerabilities, mitigations)
- [ ] Edge cases discovered (handling strategies)
- [ ] Documentation links (references)
- [ ] Skill-specific patterns (from matched skills)
- [ ] Relevant learnings (past solutions that apply)

**Deduplicate and prioritize:**
- Merge similar recommendations from multiple agents
- Prioritize by impact (high-value improvements first)
- Flag conflicting advice for human review
- Group by plan section

### 6. Enhance Plan Sections

<thinking>
Merge research findings back into the plan, adding depth without changing the original structure.
</thinking>

**Enhancement format for each section:**

```markdown
## [Original Section Title]

[Original content preserved]

### Research Insights

**Best Practices:**
- [Concrete recommendation 1]
- [Concrete recommendation 2]

**Performance Considerations:**
- [Optimization opportunity]
- [Benchmark or metric to target]

**Implementation Details:**
```[language]
// Concrete code example from research
```

**Edge Cases:**
- [Edge case 1 and how to handle]
- [Edge case 2 and how to handle]

**References:**
- [Documentation URL 1]
- [Documentation URL 2]
```

### 7. Add Enhancement Summary

At the top of the plan, add a summary section:

```markdown
## Enhancement Summary

**Deepened on:** [Date]
**Sections enhanced:** [Count]
**Agents used:** [Count] ([list agent names])

### Key Improvements
1. [Major improvement 1]
2. [Major improvement 2]
3. [Major improvement 3]

### New Considerations Discovered
- [Important finding 1]
- [Important finding 2]
```

### 8. Update Plan File

**Write the enhanced plan:**
- Preserve original filename
- Add `-deepened` suffix if user prefers a new file
- Update any timestamps or metadata

**Clean up temp files:**
```bash
rm -rf /tmp/deepen-plan
```

## Output Format

Update the plan file in place (or if user requests a separate file, append `-deepened` after `-plan`, e.g., `2026-01-15-feat-auth-plan-deepened.md`).

## Quality Checks

Before finalizing:
- [ ] All original content preserved
- [ ] Research insights clearly marked and attributed
- [ ] Code examples are syntactically correct
- [ ] Links are valid and relevant
- [ ] No contradictions between sections
- [ ] Enhancement summary accurately reflects changes

## Post-Enhancement Options

After writing the enhanced plan, use the **AskUserQuestion tool** to present these options:

**Question:** "Plan deepened at `[plan_path]`. What would you like to do next?"

**Options:**
1. **View diff** - Show what was added/changed
2. **Run `/technical_review`** - Get feedback from reviewers on enhanced plan
3. **Start `/workflows:work`** - Begin implementing this enhanced plan
4. **Deepen further** - Run another round of research on specific sections
5. **Revert** - Restore original plan (if backup exists)

Based on selection:
- **View diff** → Run `git diff [plan_path]` or show before/after
- **`/technical_review`** → Call the /technical_review command with the plan file path
- **`/workflows:work`** → Call the /workflows:work command with the plan file path
- **Deepen further** → Ask which sections need more research, then re-run those agents
- **Revert** → Restore from git or backup

NEVER CODE! Just research and enhance the plan.
