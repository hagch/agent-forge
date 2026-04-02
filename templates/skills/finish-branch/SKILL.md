---
name: finish-branch
description: >
  Use when a feature is fully implemented, verified, and documented.
  Pushes the branch with all changes (code + docs). Triggers: feature
  complete, ready to push, finish feature, push branch.
---

# Finish Branch — Push & Complete

Push the feature branch with all changes — code, tests, and documentation.

## Prerequisites

Before starting, verify:
1. CURRENT-STATE.md shows `phase: generate-docs` with status `completed`
2. Verification report exists and verdict is PASS
3. User documentation has been generated and committed

## Step 1: Pre-Check

```bash
# 1. All tests green?
<run project test command>

# 2. What branch are we on?
git branch --show-current

# 3. What will be pushed?
git log --oneline main..HEAD
git diff --stat main..HEAD
```

<pre-check>
If tests fail → STOP. Do not proceed. Go back to verify-feature.
All code, tests, AND docs must be committed before pushing.
</pre-check>

## Step 2: Push

Push the branch with all changes:

```bash
git push -u origin <branch-name>
```

Report to user:
- Branch name
- Number of commits
- Summary of what's included (features, tests, docs)
- Remote URL

## Step 3: Post-Push Cleanup

1. **Update CURRENT-STATE.md:**
   ```yaml
   phase: done
   status: completed
   integration: "pushed to origin/<branch-name>"
   human_checkpoint_pending: no
   last_updated: <ISO 8601 timestamp>
   ```

2. **Update feature registry:**
   - Add/update entry in `docs/agentforge/FEATURES.md`
   - Update `docs/agentforge/SITEMAP.md` if new user-facing routes were added
   - Update `docs/agentforge/DECISIONS.md` if architectural decisions were made
   - Optionally update `docs/agentforge/overview.md` and `docs/agentforge/workflows.md` if architecture changed

3. **Regenerate sidebar:**
   ```bash
   bash scripts/generate-sidebar.sh
   ```

4. **Commit registry updates:**
   ```bash
   git add docs/agentforge/
   git commit -m "docs: update feature registry for <feature>"
   git push origin <branch-name>
   ```

5. **Clean up worktree** (if one was created during development):
   ```bash
   git worktree remove <worktree-path>
   ```

## Red Flags

| Signal | What To Do |
|--------|------------|
| Tests fail during pre-check | Go back to verify-feature |
| Uncommitted changes exist | Commit everything before pushing |
| Docs not generated | Go back to generate-docs |
| Push fails | Check remote access, `gh auth status` |

---

## Terminal State

After successful push:

> **Invoke `agentforge:self-improve` skill** to run the retrospective and improvement cycle.
