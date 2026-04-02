---
name: orchestrate
description: >
  Interactive feature development pipeline. Guides the user through the
  8-phase agentforge skill chain with human checkpoints. Each phase invokes
  a dedicated skill. Triggers: "let's build a feature", "new feature",
  "orchestrate", "start the pipeline", "plan and implement".
---

# Orchestrate — Interactive Feature Pipeline

You are now the orchestrator. You drive the agentforge pipeline by **invoking skills in sequence**, each skill handling one phase. You manage state, transitions, and human checkpoints.

## Skill Chain

```
/brainstorm → /challenge-concept → /plan-feature → /challenge-plan → /develop-slices → /verify-feature → /finish-branch → /self-improve
```

Each skill explicitly invokes the next when it completes. Your job is to:
1. Initialize state and feature directory
2. Start the chain by invoking the first skill
3. Handle session resumption if interrupted
4. Enforce human checkpoints between phases

## On Startup — Session Continuity

1. Search for active work: `docs/agentforge/features/*/CURRENT-STATE.md`
2. If a CURRENT-STATE.md with status != `done` exists:
   - Read it to determine current phase and task status
   - Inform the user immediately what the current state is
   - If `human_checkpoint_pending: yes` → present what needs approval
   - Resume by invoking the skill for the current phase
3. If no active work → ask user what feature they want to build

## Phase Flow

### Phase 1: Brainstorm
1. Create feature directory: `docs/agentforge/features/<domain>/<slug>/`
2. Create `CURRENT-STATE.md` with `phase: brainstorm`
3. **Invoke `agentforge:brainstorm` skill**
4. Skill runs interactively (Socratic dialogue + Research + UX agents)
5. Skill produces Design-Spec and invokes challenge-concept

### Phase 2: Challenge Concept
- Invoked automatically by brainstorm skill
- `agentforge:challenge-concept` dispatches 4 parallel agents
- Results presented sequentially to user
- Spec updated with resolutions
- Invokes plan-feature when done

### Phase 3: Plan Feature
- Invoked automatically by challenge-concept skill
- `agentforge:plan-feature` creates ShapeUp slices + task DAG
- No user review — goes directly to challenge-plan

### Phase 3.5: Challenge Plan
- Invoked automatically by plan-feature skill
- `agentforge:challenge-plan` dispatches 4 parallel agents against codebase
- BLOCKER/HIGH auto-resolved (max 1 round)
- Invokes develop-slices when done

### Phase 4: Development
- Invoked automatically by challenge-plan skill
- `agentforge:develop-slices` executes slice by slice
- Per slice: specialist agents (DB → Backend → Frontend → Test → DevOps)
- Per slice: 3-stage review (Spec → Quality → Mini-Verify incl. DAU)
- Slices without dependencies run in parallel
- Escalation to user after 3 failures on same task
- Invokes verify-feature when all slices done

### Phase 5: Verification
- Invoked automatically by develop-slices skill
- `agentforge:verify-feature` dispatches 6 parallel verification agents
- **Human Checkpoint:** Verification report presented, user must approve
- BLOCKER/HIGH routed back to specialists (max 2 rounds)
- Invokes finish-branch after approval

### Phase 6: Finish Branch
- Invoked automatically by verify-feature skill
- `agentforge:finish-branch` presents integration options (PR/Merge/Keep/Discard)
- **Human Checkpoint:** User chooses integration approach
- Invokes self-improve when done

### Phase 7: Self-Improve
- Invoked automatically by finish-branch skill
- `agentforge:self-improve` runs retro + research + health check
- Documentation auto-applied, framework changes proposed

## Human Checkpoints

| After Phase | What Needs Approval |
|------------|-------------------|
| Brainstorm | Design-Spec (functional + UX, per section) |
| Challenge Concept | Resolutions for each finding |
| Verify Feature | Verification report — PASS/FAIL |
| Finish Branch | Integration choice (PR/Merge/Keep/Discard) |

## State Management

CURRENT-STATE.md tracks:
```yaml
feature: <name>
phase: brainstorm | challenge-concept | plan-feature | challenge-plan | development | verification | finish-branch | self-improve | done
human_checkpoint_pending: yes | no
slices:
  SLICE-1: not_started | in_progress | done
  SLICE-2: not_started | in_progress | done
last_updated: <ISO timestamp>
```

Update after every phase transition and slice completion.

## Error Handling

| Situation | Action |
|-----------|--------|
| Skill invocation fails | Retry once, then escalate to user |
| Agent fails 3x same error | Present full context to user with suggested fix |
| Agent context full | Save state, spawn fresh agent with guardrails |
| BLOCKER in verification | Route to specialist, max 2 rounds, then escalate |
| Unexpected error | Inform user immediately with context |

## Escalation Format

When escalating to the user:
1. What was attempted (phase, task, skill)
2. What failed (actual error messages, test output)
3. What was tried to fix it (attempts, guardrails applied)
4. What the likely root cause is
5. Suggested next steps — let the user decide

## Rules
- ALWAYS read CURRENT-STATE.md before doing anything
- ALWAYS update CURRENT-STATE.md after every phase change
- NEVER skip a phase — follow the skill chain in order
- NEVER continue past a human checkpoint without explicit user approval
- NEVER auto-apply changes to agents, skills, CLAUDE.md, or hooks
- Report progress at natural milestones
- Each skill is self-contained — trust it to do its job and invoke the next skill
- Use the user's language for communication
