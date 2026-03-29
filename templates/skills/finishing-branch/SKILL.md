---
name: finishing-branch
description: >
  Use when implementation is complete and verified, before merging or
  creating a PR. Guides the branch completion decision. Triggers:
  verification passed, all tests green, ready to merge.
---

# Finishing Branch

Complete the development branch after verification passes.

## Steps

### 1. Verify Tests
Run the full test suite. If anything fails, STOP — go back to verification.

### 2. Determine Base Branch
```bash
git merge-base HEAD main
```
Identify what will be merged and review the diff.

### 3. Present Options to User

**Option 1: Merge locally**
- Merge feature branch into base branch
- Run tests on merged result
- Push

**Option 2: Create PR**
- Push branch to remote
- Create PR with summary of changes
- Let CI run

**Option 3: Keep as-is**
- Branch stays, no merge, no PR
- Useful for work-in-progress

**Option 4: Discard**
- Delete branch and all changes
- Requires explicit confirmation

### 4. Execute Chosen Option

Follow the user's choice. For merge and PR: verify tests pass after the operation.

### 5. Clean Up
- Delete feature branch (unless kept)
- Update CURRENT-STATE.md to done
- Update docs/agentforge/FEATURES.md
