---
name: state-management
description: >
  Use when reading, writing, or reconstructing CURRENT-STATE.md for
  session continuity. Handles state transitions, task graph updates,
  and restart recovery. Triggers: phase change, task completion, session
  start, session resume.
---

# State Management

Manage CURRENT-STATE.md for session continuity across restarts.

## CURRENT-STATE.md Location

`docs/memory/features/<domain>/<slug>/CURRENT-STATE.md`

## When to Update

- Every phase transition (brainstorm → challenge → planning → ...)
- Every task status change (pending → in_progress → completed/failed)
- Every verification round
- Every problem logged
- When setting/clearing human checkpoints

## State Template

```markdown
# Current State: <feature-name>

## Meta
- **Branch:** feature/<slug>
- **Phase:** <current phase>
- **Phase Status:** pending | in_progress | completed
- **Human Checkpoint Pending:** yes | no
- **Last Updated:** <ISO 8601 timestamp>

## Task Graph

### <TASK-ID>: <title>
- **Status:** pending | in_progress | completed | failed
- **Agent:** <agent name>
- **Skills:** <skills to load>
- **Context:** <files to load>
- **Blocked by:** <task IDs or —>
- **Attempts:** <number>
- **Guardrail:** <if failed: what went wrong>

## Verification
- **Round:** <N> / max 2
- **Blockers:** <list or —>

## Problems
- <ID>: <title> (<agent>, <resolved|open>)
```

## Restart Recovery

When reading CURRENT-STATE.md on startup:

1. Parse the Meta section for phase and status
2. Parse the Task Graph for individual task states
3. Determine next action:
   - `human_checkpoint_pending: yes` → inform user, wait
   - Task with `status: in_progress` → redispatch agent with guardrails
   - Task with `status: failed` and attempts < 3 → retry with guardrail
   - Task with `status: failed` and attempts >= 3 → escalate to user
   - All tasks `completed` → advance to next phase
4. Reconstruct native tasks via TaskCreate + addBlockedBy

## Archive

When a feature is complete:
1. Set phase to `done`, phase_status to `completed`
2. Set human_checkpoint_pending to `no`
3. Update last_updated timestamp
4. The file remains as history in the feature directory
