---
name: self-improve
description: >
  Use to manually trigger framework improvement. Compares with Superpowers
  updates, searches online for new patterns, scans for community skills,
  and checks framework health. Triggers: /self-improve command, periodic
  maintenance, after noticing framework issues.
---

# /self-improve — Manual Improvement Trigger

Run a comprehensive improvement scan independent of a feature cycle.

## Step 1: Plugin Template Diff

Compare local agent/skill files against plugin templates:

1. Read each template from the plugin's `templates/` directory
2. Read the corresponding local file in `.claude/agents/` or `.claude/skills/`
3. Classify:
   - **UNCHANGED** — local matches template → eligible for auto-update
   - **LOCALLY MODIFIED** — local differs from template → preserve local changes
   - **NEW IN PLUGIN** — template exists but no local copy → offer to install

If plugin was recently updated, present template diff:
```markdown
### Plugin Template Updates

#### <filename>
- **Plugin change:** <what changed>
- **Local change:** <what was modified locally>
- **Recommendation:** AUTO-UPDATE | MERGE | KEEP LOCAL
```

## Step 2: Superpowers Diff

If Superpowers plugin is installed:

1. Read current Superpowers skill versions
2. Compare with our adapted skills:
   - `brainstorm` ← based on Superpowers `brainstorming`
   - `challenge-concept` ← agentforge-specific (4-lens stress test on design spec)
   - `plan-feature` ← based on Superpowers `writing-plans` + ShapeUp slices
   - `challenge-plan` ← agentforge-specific (4-lens stress test on implementation plan)
   - `develop-slices` ← based on Superpowers `subagent-driven-development` + vertical slices
   - `verify-feature` ← based on Superpowers `verification-before-completion` + `requesting-code-review`
   - `finish-branch` ← based on Superpowers `finishing-a-development-branch`
   - `tdd-cycle` ← based on Superpowers `test-driven-development`
   - `systematic-debugging` ← based on Superpowers `systematic-debugging`
   - `writing-skills` ← based on Superpowers `writing-skills`
3. Identify improvements in Superpowers we have not adopted
4. Report with recommendation

## Step 3: Online Research

Dispatch the `researcher` agent with:

```markdown
Search for recent developments (last 3 months) in:
1. Claude Code development patterns and best practices
2. Multi-agent coding frameworks and orchestration
3. AI-assisted software development workflows
4. New Claude Code features, plugins, skills
5. Updates to: Ralph loops, Agent Teams, Compound AI patterns

Also search for:
- Claude Code changelog / release notes
- Anthropic blog posts about Claude Code
- Community discussions about agent team patterns
```

## Step 4: Framework Health Check

Read all local framework files and check:

### Agent Consistency
- Do all agents reference base-instructions.md?
- Are tool lists up to date?
- Are model assignments appropriate (opus for complex, sonnet for focused)?
- Are there references to files/skills that don't exist?

### Skill Quality
- Do all skills have proper frontmatter (name, description with "Use when")?
- Are there skills that are never referenced by any agent?
- Are there gaps (agents mention skills that don't exist)?

### Documentation Health
- Is FEATURES.md up to date? (compare with actual code)
- Is SITEMAP.md up to date? (compare with actual routes)
- Are there feature docs with status != DONE that have no CURRENT-STATE.md?

### CLAUDE.md Drift
- Read CLAUDE.md and compare with actual code patterns
- Are there new patterns in code not documented in CLAUDE.md?
- Are there rules in CLAUDE.md that code no longer follows?

## Step 5: Generate Report

```markdown
## Self-Improvement Report — <date>

### Plugin Template Updates
<diff results from Step 1>

### Superpowers Updates
<diff results from Step 2>

### Online Research Findings
<researcher results from Step 3>

### Framework Health
<issues from Step 4>

### Recommendations

#### AUTO-APPLY (committed directly)
- <documentation changes only>

#### PROPOSE (require your approval)
- <CLAUDE.md, base-instructions, hook changes with rationale>
```

## Step 6: Apply / Propose

- **AUTO-APPLY:** Only documentation (feature docs, registries) → edit and commit
  - Commit message: `docs: self-improve scan <date>`
- **PROPOSE:** Agent definitions, skills, CLAUDE.md, base instructions, hooks → present to user, wait for explicit approval before applying
- **PLUGIN UPDATES:** Unchanged local files → update from template, commit
  - Commit message: `chore: update from agentforge template v<version>`
