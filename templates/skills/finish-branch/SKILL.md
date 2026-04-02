---
name: finish-branch
description: >
  Use when a feature is fully implemented and verified, ready for
  integration. Presents branch completion options: PR, local merge,
  keep, or discard. Triggers: verification passed, feature complete,
  ready to merge, create PR, finish feature.
---

# Finish Branch — Feature Integration

Integrate the finished, verified feature work. Simple and direct — no overengineering.

## Prerequisites

Before starting, verify:
1. CURRENT-STATE.md shows `phase: verification` with status `completed`
2. Verification report exists and verdict is PASS (or PASS WITH NOTES with user approval)
3. Feature branch is active and up to date

## Step 1: Pre-Check

Run these checks before presenting options:

```bash
# 1. All tests green?
<run project test command>

# 2. What branch are we on?
git branch --show-current

# 3. What is the base branch?
git merge-base --fork-point main HEAD

# 4. What will be integrated?
git log --oneline main..HEAD
git diff --stat main..HEAD
```

<pre-check>
If tests fail → STOP. Do not proceed. Go back to verify-feature.
If verification report is not PASS → STOP. Do not proceed.
Both conditions must be satisfied before continuing.
</pre-check>

## Step 2: Present Options

Present all 4 options to the user. Explain what each does. Wait for their choice.

---

### Option 1: Create PR (Default)

**What it does:** Push the branch and create a GitHub Pull Request.

Steps:
1. Push the feature branch to remote:
   ```bash
   git push -u origin <branch-name>
   ```

2. Create the PR using `gh`:
   ```bash
   gh pr create --title "<conventional-commit-title>" --body "<body>"
   ```

3. PR body format:
   ```markdown
   ## Summary
   <From the feature concept doc — what this feature does and why>

   ## Changes
   <From the slice summaries — what was built, organized by slice>

   ### Slice 1: <title>
   - <key changes>

   ### Slice 2: <title>
   - <key changes>

   ## Test Plan
   <From the verification report — what was tested and how>

   - [ ] Unit tests: <count> passing
   - [ ] Integration tests: <count> passing
   - [ ] E2E tests: <count> passing
   - [ ] DAU journey: tested, frustration <= 2

   ## Verification Report
   <Summary from verify-feature — verdict, accepted risks, key findings>

   ## Accepted Risks
   <MEDIUM/LOW findings accepted during verification, or "None">
   ```

4. PR title: use the feature's primary conventional commit type
   - New feature → `feat(<scope>): <description>`
   - Fix → `fix(<scope>): <description>`
   - Under 70 characters

5. Report the PR URL to the user

---

### Option 2: Merge Locally

**What it does:** Merge the feature branch into the base branch locally.

Steps:
1. Checkout the base branch:
   ```bash
   git checkout main
   ```

2. Merge the feature branch:
   ```bash
   git merge <branch-name> --no-ff
   ```
   Use `--no-ff` to preserve the merge commit for history.

3. Run the full test suite on the merged result:
   ```bash
   <run project test command>
   ```

4. If tests fail after merge → revert the merge, report to user:
   ```bash
   git merge --abort
   ```

5. If tests pass → push:
   ```bash
   git push origin main
   ```

6. Delete the feature branch:
   ```bash
   git branch -d <branch-name>
   git push origin --delete <branch-name>
   ```

---

### Option 3: Keep Branch

**What it does:** Leave everything as-is. No merge, no PR, no cleanup.

Steps:
1. Report the current state to the user:
   - Branch name
   - Worktree location (if using worktrees)
   - Number of commits ahead of base
   - Last commit message
2. Do NOT delete anything
3. Do NOT merge anything

---

### Option 4: Discard

**What it does:** Delete the feature branch and all its commits. Destructive and irreversible.

Steps:
1. **Require explicit confirmation.** Ask the user to type "discard" to confirm.
2. Do NOT proceed on "yes", "ok", or "sure" — the word "discard" is required.
3. After confirmation:
   ```bash
   git checkout main
   git branch -D <branch-name>
   ```
4. If the branch was pushed to remote:
   ```bash
   git push origin --delete <branch-name>
   ```
5. Clean up worktree if one was created:
   ```bash
   git worktree remove <worktree-path>
   ```

---

## Step 3: Post-Integration Cleanup

After executing the chosen option (except "Keep"):

1. **Clean up worktree** (if one was created during development):
   ```bash
   git worktree remove <worktree-path>
   ```

2. **Update CURRENT-STATE.md:**
   ```markdown
   ## Meta
   - **Phase:** done
   - **Phase Status:** completed
   - **Human Checkpoint Pending:** no
   - **Integration:** <PR #123 | merged locally | kept | discarded>
   - **Last Updated:** <ISO 8601 timestamp>
   ```

3. **Update feature registry:**
   - Add/update entry in `docs/agentforge/FEATURES.md`
   - Update `docs/agentforge/SITEMAP.md` if new pages were created
   - Run `bash scripts/generate-sidebar.sh` if docs changed

---

## Red Flags — Integration Is Going Wrong If:

| Signal | What To Do |
|--------|------------|
| Tests fail during pre-check | Go back to verify-feature — do not integrate broken code |
| Merge conflicts | Resolve conflicts, run full test suite, verify before completing merge |
| PR creation fails | Check remote access, branch push status, gh auth status |
| User hesitates on discard | Suggest "Keep" instead — discard is irreversible |

---

## Terminal State

After successful integration:

> **Invoke `agentforge:self-improve` skill** to run the retrospective and improvement cycle.
