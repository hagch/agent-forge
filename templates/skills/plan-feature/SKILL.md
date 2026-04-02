---
name: plan-feature
description: >
  Use when a challenged Design Spec is ready for implementation planning.
  Creates ShapeUp-style vertical slices with micro-tasks, pseudocode, and
  a task DAG. Triggers: challenge complete, ready to plan, concept approved
  and challenged, create implementation plan.
---

# Plan Feature — ShapeUp Vertical Slices + Task DAG

You are the implementation planner. Given a challenged Design Spec, you create a plan that dev agents can execute slice-by-slice without asking questions.

**Iron Law:** No placeholders. Every slice has pseudocode, every task has acceptance criteria, every file path is real. If you write "TBD", "TODO", or "similar to above" — you have failed.

## On Startup

1. Read the Design Spec from `docs/agentforge/features/<domain>/<slug>/concept.md`
2. Read the Challenge Log from `docs/agentforge/features/<domain>/<slug>/challenge-concept.md`
3. Read CLAUDE.md for project code standards
4. Read the existing codebase: file structure, naming conventions, existing patterns
5. Update CURRENT-STATE.md: `phase: plan-feature`

## This Phase Is Autonomous

**Do NOT ask the user questions.** If something is ambiguous, make a reasonable decision, document it in the plan under "Planning Decisions", and move on. The plan goes directly to challenge-plan for review — no user checkpoint here.

## Step 1: ShapeUp Pitch

Frame the work using the ShapeUp pitch structure:

```markdown
## Pitch

**Problem:** <What user problem does this solve — from the spec>
**Appetite:** <Time budget — S (1-2 days), M (3-5 days), L (1-2 weeks)>
**Solution:** <High-level approach — the chosen architecture from the spec>
**Rabbit Holes:** <Areas of complexity that could consume disproportionate time>
**No-Gos:** <What we explicitly will NOT build — from the spec's scope boundaries>
```

## Step 2: Identify Vertical Slices

Each slice is one **end-to-end piece** of the feature: DB -> API -> UI -> Test. Each slice is independently testable and delivers visible progress.

### Slice Identification Rules
- A slice delivers ONE user-visible behavior or ONE infrastructure prerequisite
- A slice can be demo'd or tested on its own (even if ugly)
- Slices are ordered from highest-risk to lowest-risk (build the hardest thing first)
- First slice should be a "spike slice" if there is significant technical uncertainty

### Slice Format

```markdown
## SLICE-<N>: <Title>

**Goal:** <What this slice delivers — one sentence>
**E2E Scope:** <What layers it touches: DB | API | Service | UI | Test>
**Depends On:** [SLICE-<IDs>]
**Acceptance Criteria:**
- Given <precondition>, When <action>, Then <expected result>
- Given ...
- ...

### File Boundaries
- CREATE: <exact/path/to/file.ext> — <purpose>
- MODIFY: <exact/path/to/file.ext> — <what changes>
- ...

### Pseudocode

\`\`\`pseudocode
// <Layer: Entity/Migration>
// <What this code does conceptually, NOT real syntax>
// Enough detail that a dev agent understands the approach
// without copy-pasting

// <Layer: Service>
// ...

// <Layer: Controller/API>
// ...

// <Layer: UI Component>
// ...
\`\`\`

### Micro-Tasks (TDD Cycles)

Each micro-task is one Red-Green-Refactor cycle (2-5 minutes of agent work).

#### SLICE-<N>.1: <Task Title>
- **Type:** RED — Write failing test
- **File:** <exact/path/to/test.ext>
- **What to test:** <Specific behavior — one assertion focus>
- **Skills:** [tdd-cycle]

#### SLICE-<N>.2: <Task Title>
- **Type:** GREEN — Make it pass
- **File:** <exact/path/to/implementation.ext>
- **What to implement:** <Minimal code to pass the test>
- **Skills:** [tdd-cycle]

#### SLICE-<N>.3: <Task Title>
- **Type:** REFACTOR — Clean up
- **What to refactor:** <Specific improvement>
- **Skills:** [tdd-cycle]

#### SLICE-<N>.4: <Task Title>
- **Type:** COMMIT
- **Message:** `<type>(<scope>): <description>`
```

## Step 3: Task DAG Visualization

After all slices, provide the dependency graph:

```markdown
## Task DAG

### Dependency Graph

SLICE-1 (Foundation) ──→ SLICE-2 (Core Logic) ──→ SLICE-4 (UI)
                    └──→ SLICE-3 (Secondary)  ──→ SLICE-5 (Integration)
                                                        ↓
                                                   SLICE-6 (E2E)

### Parallel Groups

| Group | Slices | Can Start After |
|-------|--------|-----------------|
| 1     | [SLICE-1] | Immediately |
| 2     | [SLICE-2, SLICE-3] | SLICE-1 complete |
| 3     | [SLICE-4, SLICE-5] | SLICE-2 complete |
| 4     | [SLICE-6] | All previous complete |

### Agent Assignment

| Slice | Agent | Reason |
|-------|-------|--------|
| SLICE-1 | dev-backend | DB + API layer |
| SLICE-2 | dev-backend | Service logic |
| SLICE-3 | dev-frontend | UI components |
| ... | ... | ... |
```

## Step 4: Planning Decisions

Document every ambiguous decision you made:

```markdown
## Planning Decisions

| # | Decision | Alternatives Considered | Rationale |
|---|----------|------------------------|-----------|
| 1 | <What you decided> | <Other options> | <Why this choice> |
| 2 | ... | ... | ... |
```

## Step 5: Risk Assessment

```markdown
## Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| <What could go wrong during implementation> | H/M/L | H/M/L | <How the plan handles it> |
| ... | ... | ... | ... |
```

## Step 6: Self-Review Checklist

Before writing the plan, verify:

- [ ] Every acceptance criterion from the Design Spec has a corresponding slice
- [ ] Every slice has pseudocode (not just a description)
- [ ] Every micro-task specifies the exact file to create or modify
- [ ] File paths are real — checked against the actual codebase
- [ ] Type/method/component names are consistent across slices
- [ ] Each micro-task can be understood without reading other tasks
- [ ] At least one E2E test slice exists
- [ ] Challenge findings marked "Fix in spec" are addressed in the plan
- [ ] No "TBD", "TODO", "similar to above", or "add appropriate..."
- [ ] Commit messages follow conventional commits format
- [ ] Skills field references only existing skills (tdd-cycle, systematic-debugging)

## Output Location

Write the implementation plan to:
```
docs/agentforge/features/<domain>/<slug>/plan.md
```

Update CURRENT-STATE.md:
```yaml
phase: plan-feature
status: complete
artifact: plan.md
slices_total: X
micro_tasks_total: X
estimated_appetite: S|M|L
```

## Terminal State

When the plan is complete and self-reviewed:

**Invoke `agentforge:challenge-plan` skill to validate the plan against the real codebase.**

Do NOT present the plan to the user — it goes directly to challenge-plan. The user will see the final plan after it survives the challenge.

## Red Flags — You Are Violating This Skill If:

| What You Are Doing | Why It Is Wrong |
|---------------------|----------------|
| Asking the user questions | This phase is autonomous — decide and document |
| Writing horizontal slices (all DB, then all API, then all UI) | Vertical slices deliver testable increments |
| Pseudocode that is actually real code | Agents should understand the approach, not copy-paste |
| Tasks without file paths | Agents do not know where to write |
| File paths that do not exist in the codebase | Agents will create files in wrong locations |
| "Similar to SLICE-1" in SLICE-3 | Agents may read slices out of order — repeat the content |
| Skipping the E2E test slice | You cannot verify the feature works end-to-end |
| Starting with the easiest slice | Build the riskiest thing first to fail fast |

## Rules

- Follow `base-instructions.md` for all process rules
- Load `plan-creation` skill for detailed task format reference
- Vertical slices only — every slice touches DB-to-UI (or the relevant subset)
- Pseudocode, not real code — enough to understand the approach
- Micro-tasks are TDD cycles: RED -> GREEN -> REFACTOR -> COMMIT
- Each micro-task is 2-5 minutes of agent work — split if larger
- Risk-first ordering: hardest/riskiest slice first
- Conventional commits in all commit message templates
- Reference existing code patterns in the codebase — do not invent new patterns
