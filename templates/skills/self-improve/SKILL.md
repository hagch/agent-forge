---
name: self-improve
description: >
  Use to manually trigger framework improvement. Runs feature retro (if
  invoked after /finish-branch), compares with Superpowers updates, searches
  online for new patterns, scans for community skills, and checks framework
  health. Triggers: /self-improve command, after /finish-branch, periodic
  maintenance, after noticing framework issues.
---

# /self-improve — Feature Retro & Manual Improvement Trigger

Run a feature retrospective (after `/finish-branch`) and/or a comprehensive improvement scan.

## Step 0: Feature Retro

If invoked after `/finish-branch` (i.e., a feature was just completed), analyze what happened:

### 0.1: Read Agent Logs
Read ALL `## Agent Log` entries from the completed feature doc at `docs/agentforge/features/<domain>/<slug>/README.md`.

Collect:
- All **Problem** entries (with agent name, task, root cause, solution)
- All **Success** entries (with agent name, task, what worked)
- All **Guardrail** entries (what NOT to do)

### 0.2: Analyze Patterns
Identify recurring patterns across all agent logs:

- **Rework patterns:** Which agents had the most Problems? Which review stage caught the most issues?
- **Spec gaps:** Were there MISSING or PARTIALLY_IMPLEMENTED findings from spec-reviewer? → Brainstorming needs improvement
- **Quality patterns:** Which dimensions scored below 7 in quality-reviewer? → Dev agent instructions need improvement
- **DAU issues:** What frustration > 3 moments did dau-tester find? → UX-researcher or frontend agent needs improvement
- **Verification blockers:** What BLOCKER/HIGH issues did verifier find? → Which phase should have caught this earlier?
- **Agent stuck patterns:** Did any agent hit the 3-attempt escalation limit? → Instructions unclear or missing context

### 0.3: Generate Improvement Proposals
For each pattern found, generate a specific improvement proposal:

```markdown
### Retro Finding: <title>

**Pattern:** <what happened repeatedly>
**Affected agents:** <list>
**Root cause:** <why this happened — missing instruction? unclear spec? wrong skill?>
**Proposed fix:** <specific change to agent/skill/base-instructions>
**Apply to:** <exact file path>
```

### 0.4: Present Retro Summary
Present to user:

```markdown
## Feature Retro — <feature name>

### What Went Well
- <successes from agent logs>

### What Caused Rework
- <problems, grouped by pattern>

### Improvement Proposals
<list of proposals from 0.3>

### Metrics
- Total Problems logged: X
- Total Successes logged: X
- Agents with most issues: <list>
- Review stage that caught most: <spec-compliance | quality | mini-verify | full-verify>
- Average DAU frustration: X.X
```

If no active feature was just completed (manual `/self-improve` run), skip Step 0 and start at Step 1.

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
