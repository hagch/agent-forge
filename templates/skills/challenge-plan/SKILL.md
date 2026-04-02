---
name: challenge-plan
description: >
  Use when an implementation plan needs validation against the real codebase
  before development starts. 4 parallel agents with full codebase access
  review architecture, completeness, maintainability, and risk. Triggers:
  plan complete, plan ready for review, validate implementation plan.
---

# Challenge Plan — Codebase-Aware Technical Validation

You are the plan challenge orchestrator. Your job is to validate the implementation plan against the **real codebase** — not in theory, but by reading actual code, checking actual file paths, and verifying actual patterns.

**Iron Law:** Every agent in this phase has codebase access and MUST read actual code. Findings based on assumptions about the codebase (without reading it) are invalid.

## Difference from Concept Challenge

| Concept Challenge | Plan Challenge |
|-------------------|----------------|
| "What can go wrong with the design?" | "Will this plan actually work in this codebase?" |
| Reviews the spec abstractly | Reads real code, checks real paths |
| Security, performance, DX, DAU lenses | Architecture, completeness, maintainability, risk lenses |
| Fixes go into the spec | Fixes go into the plan |

## On Startup

1. Read the implementation plan from `docs/agentforge/features/<domain>/<slug>/plan.md`
2. Read the Design Spec from `docs/agentforge/features/<domain>/<slug>/concept.md`
3. Read CURRENT-STATE.md to confirm phase
4. Update CURRENT-STATE.md: `phase: challenge-plan`

## Step 1: Dispatch 4 Parallel Agents

All agents have full codebase access. They MUST read actual files.

<agents>
<agent name="architecture-reviewer">
  Model: sonnet | Tools: Read, Glob, Grep
  Perspective: Architecture Alignment
  Instructions:
  - Read the pseudocode in each slice and compare against ACTUAL code in the codebase
  - For every file path in the plan: verify it exists (or that the parent directory exists for CREATE)
  - Check: Does the pseudocode match existing architectural patterns?
  - Check: Is the plan building something that already exists? Search for similar code
  - Check: Are there reuse opportunities the plan missed? (existing utilities, services, components)
  - Check: Layer violations — does the plan respect the existing layer boundaries?
  - Check: Circular dependencies — would the plan create any?
  - Check: Dependency direction — do new dependencies flow in the right direction?
  - For each finding: reference the actual file/line in the codebase that proves the point
  - Output: findings in the standard format (see below)
</agent>
<agent name="completeness-reviewer">
  Model: sonnet | Tools: Read, Glob, Grep
  Perspective: Completeness
  Instructions:
  - Cross-reference EVERY acceptance criterion in the Design Spec against plan slices
  - Check: Are there spec requirements with no corresponding slice?
  - Check: Are there edge cases from the Challenge Log that are not addressed?
  - Check: Missing slices — database migrations, config changes, i18n, permissions?
  - Check: Are error handling paths covered in the micro-tasks?
  - Check: Are all CRUD operations covered (not just Create, but also Read/Update/Delete)?
  - Check: Integration points — does the plan handle all connections to existing features?
  - Create a traceability matrix: Spec Requirement -> Slice -> Micro-Tasks
  - Output: findings in the standard format + traceability matrix
</agent>
<agent name="maintainability-reviewer">
  Model: sonnet | Tools: Read, Glob, Grep
  Perspective: Maintainability
  Instructions:
  - Read existing code in the same layers the plan targets
  - Check: Does the plan follow the SAME patterns? (naming, structure, error handling)
  - Check: Coupling — are planned components appropriately decoupled?
  - Check: Cohesion — does each planned file have a single responsibility?
  - Check: Testability — can each component be tested in isolation as planned?
  - Check: Naming consistency — do planned names match the naming conventions in the codebase?
  - Check: Is there existing test infrastructure (factories, fixtures, helpers) the plan should use?
  - Search for ACTUAL examples of similar patterns in the codebase and compare
  - Output: findings in the standard format
</agent>
<agent name="risk-ordering-reviewer">
  Model: sonnet | Tools: Read, Glob, Grep
  Perspective: Risk and Ordering
  Instructions:
  - Validate the task DAG:
    * Are dependencies correct? (Does SLICE-3 truly depend on SLICE-2?)
    * Are parallel groups truly independent? (No shared state, no file conflicts)
    * Is the ordering risk-first? (Hardest/riskiest slice first)
  - Identify the single biggest implementation risk — where is the plan most likely to fail?
  - Check: Should there be a spike-slice (proof-of-concept) before the full implementation?
  - Check: Are micro-tasks correctly sized? (2-5 min each — flag tasks that seem too large)
  - Check: Agent assignment — are the right agents assigned to the right slices?
  - Check: File ownership boundaries — do parallel slices edit the same files? (conflict risk)
  - Output: findings in the standard format + revised DAG if needed
</agent>
</agents>

## Step 2: Finding Format (All Agents Use This)

```markdown
### [BLOCKER|HIGH|MEDIUM|LOW] <Title>
- **Perspective:** Architecture | Completeness | Maintainability | Risk-Ordering
- **Affected Slice:** SLICE-<N> (or "Plan-wide")
- **Evidence:** <Actual file path and what you found when reading it>
- **Issue:** <What is wrong — specific and concrete>
- **Required Change:** <Exact modification to the plan>
```

## Severity Decision Guide

| Severity | Criteria |
|----------|----------|
| BLOCKER | Plan will fail during execution — wrong file paths, duplicate code, missing critical slice, broken DAG |
| HIGH | Plan will produce poor results — wrong patterns, missing edge cases, suboptimal ordering |
| MEDIUM | Plan works but could be better — reuse opportunity missed, naming inconsistency |
| LOW | Minor improvements — slightly better ordering, cosmetic naming preference |

## Step 3: Auto-Resolution (Max 1 Iteration)

After all agents complete:

### BLOCKER and HIGH findings:

1. Automatically adjust the plan to resolve BLOCKER and HIGH findings
2. Apply changes to the plan document
3. Re-run a quick validation ONLY on the changed sections (not all 4 agents again)
4. If BLOCKERs persist after 1 iteration: **escalate to the user**

### Escalation Format

```markdown
## Plan Challenge Escalation

**Unresolved BLOCKERs after auto-adjustment:**

### BLOCKER: <Title>
- **Issue:** <What is wrong>
- **What was tried:** <How the plan was adjusted>
- **Why it still blocks:** <Why the adjustment was insufficient>
- **Options:**
  1. <Option A with trade-offs>
  2. <Option B with trade-offs>
  3. Go back to brainstorming for this area

**Decision needed from user.**
```

### MEDIUM and LOW findings:

- Document in the plan under a "Known Trade-offs" section
- No plan changes required — these are informational

## Step 4: Present Final Plan to User

After auto-resolution (or escalation):

1. **Summary of changes:**
   ```markdown
   ## Plan Challenge Results
   | Perspective | BLOCKER | HIGH | MEDIUM | LOW | Auto-Fixed | Documented |
   |-------------|---------|------|--------|-----|------------|------------|
   | Architecture |   X    |  X   |   X    |  X  |     X      |     X      |
   | Completeness |   X    |  X   |   X    |  X  |     X      |     X      |
   | Maintainability | X   |  X   |   X    |  X  |     X      |     X      |
   | Risk-Ordering |  X    |  X   |   X    |  X  |     X      |     X      |
   ```

2. **Traceability matrix** (from completeness reviewer)
3. **Revised DAG** (if changed by risk-ordering reviewer)
4. **The final plan** — present to user for approval

## Step 5: Development Dispatch Decision

Before invoking the next skill, determine the execution strategy:

<execution-strategy>
Check if Agent Teams beta feature is active:
- If CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1 is set:
  → Use Agent Teams for parallel slice execution
- Otherwise:
  → Use subagent-driven development (sequential with parallel groups)

Document the chosen strategy in CURRENT-STATE.md.
Note: There is NO option for inline execution — development always uses agents.
</execution-strategy>

## Output Location

Write the challenge results to:
```
docs/agentforge/features/<domain>/<slug>/challenge-plan.md
```

Update the plan (`plan.md`) with auto-fixed changes.

Update CURRENT-STATE.md:
```yaml
phase: challenge-plan
status: complete
artifact: challenge-plan.md
findings_total: X
auto_fixed: X
documented: X
escalated: X
execution_strategy: agent-teams | subagent
```

## Terminal State

When all BLOCKERs are resolved and the user approves the final plan:

**Invoke `agentforge:develop-slices` skill to begin implementation.**

## Red Flags — You Are Violating This Skill If:

| What You Are Doing | Why It Is Wrong |
|---------------------|----------------|
| Findings without reading actual code | This phase is about codebase reality, not theory |
| "The plan looks correct" without evidence | Show the file you read, the pattern you compared |
| Auto-fixing more than 1 iteration | Diminishing returns — escalate to the user |
| Changing the Design Spec | This phase modifies the plan, not the spec |
| Skipping the traceability matrix | You cannot prove completeness without it |
| Allowing parallel slices that edit the same file | File conflicts will break development |
| Not checking if file paths actually exist | Agents will fail at the first task |
| Running agents sequentially instead of parallel | Wastes time — all 4 are independent |

## Rules

- Follow `base-instructions.md` for all process rules
- ALL review agents MUST have codebase access and MUST read actual files
- Every finding MUST include evidence from the codebase (file path + what was found)
- Auto-fix BLOCKER/HIGH — max 1 iteration, then escalate
- MEDIUM/LOW are documented, not fixed
- The plan is the artifact that changes — never modify the Design Spec in this phase
- Traceability matrix is mandatory — no completeness claim without it
- File path verification is mandatory — glob for every CREATE/MODIFY path in the plan
- Parallel groups must be validated for file ownership conflicts
