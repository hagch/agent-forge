---
name: develop-slices
description: >
  Use when executing an approved implementation plan slice by slice.
  Each slice dispatches specialist agents, follows strict TDD, uses
  conventional commits, and passes 3-stage review before completion.
  Triggers: plan approved, start development, execute slices,
  implement feature, development phase.
---

# Develop Slices — Slice-by-Slice Implementation

Execute the approved implementation plan. Each slice gets specialist agents, strict TDD, conventional commits, and a 3-stage review gate. No slice is done until all 3 reviews pass.

## Prerequisites

Before starting, verify:
1. An approved plan exists in the feature doc with a task-DAG
2. CURRENT-STATE.md exists with `phase: development`
3. A feature branch is active
4. Read `base-instructions.md` — all rules apply throughout

## Environment Detection

<check>
Detect whether Agent Teams is available:
- If CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1 is set → use parallel TeamCreate for independent slices
- Otherwise → use sequential subagent dispatch via TaskCreate
</check>

## Slice Execution Model

```
┌─────────────────────────────────────────────────────────────────────┐
│ Per Slice                                                           │
│                                                                     │
│  DB-Agent ──→ Backend-Agent ──→ Frontend-Agent ──→ Test-Agent       │
│  (if needed)  (if needed)       (if needed)        (E2E/Integration)│
│                                                                     │
│  DevOps-Agent runs in parallel when no dependencies on above        │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ After ALL micro-tasks complete:                              │    │
│  │  Review 1: Spec-Compliance ──→ Fix Loop (max 2)             │    │
│  │  Review 2: Code-Quality    ──→ Fix Loop (max 2)             │    │
│  │  Review 3: Mini-Verification (3 passes) ──→ Fix Loop (max 2)│    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                     │
│  ✓ Slice complete → update CURRENT-STATE.md → next slice            │
└─────────────────────────────────────────────────────────────────────┘

Between slices WITHOUT dependencies → PARALLEL
Between slices WITH dependencies   → SEQUENTIAL
```

---

## Step 1: Parse the Plan

1. Read the task-DAG from the feature doc
2. Group tasks into slices (the plan defines slice boundaries)
3. Build a dependency graph between slices
4. Identify which slices can run in parallel (no mutual dependencies)
5. For each slice, identify which specialist agents are needed

## Step 2: Execute Each Slice

For every slice, follow the sequence below.

### 2a. Dispatch Specialist Agents

Dispatch micro-tasks to the appropriate specialist agent in dependency order:

<agents>

**DB-Agent** — Migrations, schema changes, indexes, seed data
- Runs FIRST within a slice (other agents depend on schema)
- Produces: migration files, updated schema, seed data

**Backend-Agent** — Entities, services, controllers, API endpoints, business logic
- Runs AFTER DB-Agent (needs schema)
- Produces: API endpoints, service layer, business logic

**Frontend-Agent** — Components, pages, forms, state management, UI patterns
- Runs AFTER Backend-Agent (needs API contracts)
- Produces: UI components, pages, client-side state

**Test-Agent** — E2E tests, integration tests
- Runs AFTER Frontend-Agent and Backend-Agent (needs endpoints + UI)
- Produces: E2E test suites, integration test suites

**DevOps-Agent** — Docker, CI/CD, config, infrastructure
- Runs in PARALLEL with others when no direct dependency
- Produces: config files, CI pipelines, Docker changes

</agents>

### 2b. TDD per Micro-Task

Every micro-task follows strict TDD. Load the `tdd-cycle` skill.

```
RED ──→ GREEN ──→ REFACTOR ──→ COMMIT ──→ NEXT
```

<tdd-isolation>

**Three-Agent TDD Isolation Pattern:**
When feasible, separate test writing from implementation:
1. Test-writer agent writes the failing test with NO knowledge of implementation details
2. Implementer agent sees ONLY the failing test and writes minimal code to pass
3. This prevents tests that merely mirror implementation

For simple micro-tasks, a single agent may run the full TDD cycle.

</tdd-isolation>

### 2c. Conventional Commits

After EVERY TDD cycle, create a conventional commit:

```
<type>(<scope>): <description>
```

| Type | When |
|------|------|
| `feat` | New functionality |
| `fix` | Bug fix |
| `test` | Adding or updating tests only |
| `refactor` | Code restructuring, no behavior change |
| `chore` | Config, dependencies, tooling |

Rules:
- Imperative mood: "add", not "added" or "adds"
- Title under 50 characters
- Scope = module or component name
- One commit per TDD cycle, not per file

### 2d. PostToolUse Hooks

If PostToolUse hooks are configured (auto-run tests after Edit/Write):
- Let the hooks run — do not suppress them
- Read hook output before proceeding
- If hooks show test failures, fix BEFORE continuing

### 2e. Token Budget Awareness

<token-budget>
Monitor context usage during long slices:
- At ~85% context capacity: STOP, save full state to CURRENT-STATE.md, and request a fresh agent instance
- The fresh agent reads CURRENT-STATE.md and resumes from the last completed micro-task
- Include guardrails from any problems encountered so far
</token-budget>

---

## Step 3: 3-Stage Slice Review

After ALL micro-tasks in a slice are complete, run 3 sequential review stages. A slice is NOT done until all 3 pass.

### Review 1: Spec-Compliance Review

<spec-compliance>

**Goal:** Does the implementation match the spec exactly?

Process:
1. Enumerate every requirement from the slice spec
2. For each requirement, classify:

| Status | Meaning |
|--------|---------|
| IMPLEMENTED | Requirement fully met, evidence provided |
| PARTIALLY | Some aspects met, gaps identified |
| MISSING | Not implemented at all |
| CANNOT_VERIFY | Cannot determine without manual testing |

3. Provide evidence for each classification (test output, code reference)
4. Verdict: PASS (all IMPLEMENTED) or FAIL (any PARTIALLY/MISSING)

If FAIL: route findings back to the responsible specialist agent for fixes (same agent instance, not a fresh one). Max 2 fix rounds, then escalate to user.

</spec-compliance>

### Review 2: Code-Quality Review

<code-quality>

**Goal:** Clean code, good patterns, maintainable.

Check each dimension and score 1-10:

| Dimension | What to Check |
|-----------|---------------|
| Complexity | Cyclomatic complexity, nesting depth, function length |
| Duplication | Copy-paste across files, repeated patterns that should be abstracted |
| Naming | Variables, functions, classes — self-documenting? |
| Error Handling | All failure paths covered? Errors propagated correctly? |
| Test Quality | Tests test behavior not implementation? Edge cases covered? |
| Consistency | Matches existing codebase patterns? (Check CLAUDE.md conventions) |

Per-finding format:
```markdown
#### [HIGH|MEDIUM|LOW] <Title>
- **Dimension:** <which dimension>
- **File:** <path:line>
- **Issue:** <specific problem>
- **Fix:** <concrete change>
- **Score Impact:** <how it affects the dimension score>
```

Verdict: PASS (no HIGH findings, average score >= 7) or FAIL.

If FAIL: route to the implementer agent (same instance). Max 2 rounds, then escalate.

</code-quality>

### Review 3: Mini-Verification (3 Passes)

<mini-verification>

**Goal:** The slice works correctly in isolation and with previous slices.

**Pass 1 — Technical:**
- All tests green (unit + integration + E2E for this slice)
- Paste FULL test output as evidence
- Security: no obvious vulnerabilities introduced
- Performance: no N+1 queries, unbounded operations, missing indexes

**Pass 2 — DAU:**
Load the `dau-ui-test` skill if UI changes are in this slice.
- Non-technical impatient user persona
- Can the user find, use, and understand the new functionality?
- Mobile viewport (375px minimum)
- Accessibility: keyboard navigation, focus indicators, WCAG 2.1 AA
- Frustration rating 1-5 (1=smooth, 5=unusable)
- Verdict: PASS if frustration <= 2

**Pass 3 — Integration with Previous Slices:**
- Does this slice break anything from previous slices?
- Run the FULL test suite, not just this slice's tests
- API contracts consistent across slices?
- Data flow correct end-to-end through completed slices?

Verdict: PASS (all 3 passes green) or FAIL.

If FAIL: route to responsible agent. Max 2 rounds, then escalate.

</mini-verification>

---

## Step 4: Post-Slice Housekeeping

After a slice passes all 3 reviews:

1. **Agent Log** — Write to the feature doc:
   ```markdown
   ### Slice <N> Agent Log
   **Problems:** <what went wrong, how it was fixed>
   **Successes:** <what went smoothly, patterns that worked>
   **Guardrails:** <lessons for future slices>
   ```

2. **Update CURRENT-STATE.md:**
   - Mark slice tasks as `completed`
   - Update phase status
   - Record slice review results
   - Unblock dependent slices

3. **Proceed to next slice** (or parallel slices if dependencies allow)

---

## Kill Criteria

<kill-criteria>
If a specialist agent is stuck for 3+ iterations on identical errors:
1. Save all context (error messages, attempts, guardrails) to CURRENT-STATE.md
2. Terminate the stuck agent
3. Dispatch a FRESH agent instance with:
   - The original task description
   - All accumulated guardrails
   - Explicit instruction: "Previous agent failed 3 times on this. Approaches that did NOT work: [list]. Try a different approach."
4. If the fresh agent also fails twice → escalate to user
</kill-criteria>

## Fix Loop Protocol

<fix-loop>
When a review returns FAIL:
1. Route findings to the SAME agent that wrote the code (not a fresh instance)
2. Agent reads findings and applies fixes
3. Agent re-runs affected tests to verify
4. Re-run ONLY the failed review stage (not all 3)
5. Max 2 fix rounds per review stage
6. After 2 failed rounds → escalate to user with:
   - What was attempted
   - What keeps failing
   - Review findings
   - Suggested manual intervention
</fix-loop>

## Red Flags — Development Is Going Wrong If:

| Signal | What To Do |
|--------|------------|
| Agent writes code without failing test first | Stop, revert, restart with TDD |
| Commit contains multiple unrelated changes | Revert, split into separate TDD cycles |
| Test passes on first run (for new functionality) | Test is likely wrong — verify it can fail |
| Agent claims "should work" without test evidence | Violation of base-instructions Iron Law |
| Review findings keep recurring across rounds | Escalate — likely architectural issue |
| Context usage above 85% | Pause, save state, spawn fresh agent |
| Slice has no E2E or integration test | Slice is NOT complete — add tests |

---

## Terminal State

When all slices are complete and all reviews pass:

> **Invoke `agentforge:verify-feature` skill** to run full-feature verification across all slices.
