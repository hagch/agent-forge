---
name: challenge-concept
description: >
  Use when a Design Spec is complete and needs stress-testing from multiple
  technical perspectives before planning. 4-lens parallel review: Security,
  Performance, DX, DAU-Stress. Triggers: concept approved, spec complete,
  brainstorm done, ready for challenge.
---

# Challenge Concept — 4-Lens Stress Test

You are the challenge orchestrator. Your job is to find what can go wrong with the approved Design Spec **technically** — before a single line of planning begins.

**Iron Law:** Every finding must be specific, actionable, and reference a concrete part of the spec. Vague findings waste everyone's time.

## Difference from Brainstorming

| Brainstorming | Concept Challenge |
|---------------|-------------------|
| "What should it do?" | "What can go wrong?" |
| Functional + UX focus | Security, performance, DX, edge cases |
| Interactive with user | Parallel agents, then sequential review |
| Builds the spec | Stress-tests the spec |

## On Startup

1. Read the Design Spec from `docs/agentforge/features/<domain>/<slug>/concept.md`
2. Read CURRENT-STATE.md to confirm phase
3. Update CURRENT-STATE.md: `phase: challenge-concept`

## Step 1: Dispatch 4 Parallel Agents

Launch all 4 agents simultaneously for time efficiency:

<agents>
<agent name="security-challenger">
  Model: sonnet | Tools: Read, Glob, Grep, WebSearch
  Perspective: Security
  Instructions:
  - Review EVERY component in the spec through an attacker's lens
  - Check OWASP Top 10 categories against each endpoint/interaction
  - For each finding: describe an explicit exploit scenario step-by-step
  - Confidence scoring: only report findings with confidence >= 0.7
  - Hard exclusions: do NOT report theoretical risks without a concrete scenario
  - Check: authentication bypass, authorization escalation, injection vectors,
    data exposure, tenant isolation, CSRF, insecure defaults
  - Output: findings in the standard format (see below)
</agent>
<agent name="performance-challenger">
  Model: sonnet | Tools: Read, Glob, Grep
  Perspective: Performance
  Instructions:
  - For every data operation in the spec, reason about scale:
    "With 10K users doing this concurrently, what happens?"
  - Check: N+1 queries, unbounded operations, missing pagination,
    missing caching, large payloads, expensive computations on hot paths
  - For each finding: include a load scenario with concrete numbers
  - Require: "With N=X, this operation would take approximately Y"
  - Check existing codebase for performance patterns already in use
  - Output: findings in the standard format
</agent>
<agent name="dx-challenger">
  Model: sonnet | Tools: Read, Glob, Grep
  Perspective: Developer Experience
  Instructions:
  - Review spec through the "next developer" lens — someone joins in 6 months
  - Testability checklist: can each component be tested in isolation?
  - Consistency enforcement: does the spec follow existing project patterns?
  - Onboarding readiness: could a developer implement this from the spec alone?
  - Check: over-engineering, unnecessary new abstractions, naming inconsistency,
    tight coupling, missing error handling strategy, unclear data flows
  - Compare against existing patterns in the codebase (read actual code)
  - Output: findings in the standard format
</agent>
<agent name="dau-stress-challenger">
  Model: sonnet | Tools: Read, Glob, Grep
  Perspective: DAU-Stress (Dumbest Assumable User under stress)
  Instructions:
  - Adopt persona: non-technical, impatient user who reads nothing
  - For each user-facing interaction in the spec, simulate:
    * Double-click on every button
    * Hit back button mid-flow
    * Open same page in two tabs
    * Try on mobile
    * Enter garbage data
    * Walk away mid-process and come back
  - Frustration rating per finding: 1 (minor annoyance) to 5 (user rage-quits)
  - Only report findings with frustration >= 3
  - Check: missing loading states, confusing error messages, no empty states,
    unclear navigation, broken flows on interruption
  - Output: findings in the standard format
</agent>
</agents>

## Step 2: Finding Format (All Agents Use This)

```markdown
### [BLOCKER|HIGH|MEDIUM|LOW] <Title>
- **Perspective:** Security | Performance | DX | DAU-Stress
- **Affected Component:** <Specific section/component from the spec>
- **Scenario:** <Step-by-step description of what goes wrong>
- **Impact:** <What happens if we ship without fixing this>
- **Required Change:** <Concrete, actionable fix for the spec>
- **Confidence:** <0.0-1.0 for Security> | <Frustration 1-5 for DAU>
```

## Severity Decision Guide

Ask: "What happens if we ship without fixing this?"

| Severity | Criteria | Examples |
|----------|----------|---------|
| BLOCKER | Data loss, security breach, complete feature failure | Auth bypass, SQL injection, data corruption |
| HIGH | Degraded experience, missing critical handling | N+1 on list page, no error state, confusing flow |
| MEDIUM | Inconsistency, suboptimal but functional | Naming mismatch, missing cache, minor UX friction |
| LOW | Cosmetic, preference, nice-to-have | Style inconsistency, verbose logging, minor optimization |

## Step 3: Present Results to User (Sequential, One Perspective at a Time)

After all agents complete, present results ONE perspective at a time:

### For each perspective:

1. **Summary header:**
   ```
   ## Security Review: X findings (Y BLOCKER, Z HIGH, ...)
   ```
2. **Present all findings** for this perspective, ordered by severity (BLOCKER first)
3. **For each finding, ask the user to decide:**
   - **Fix in spec** — incorporate the required change into the Design Spec
   - **Accept risk** — acknowledge the issue, document why we accept it
   - **Consciously ignore** — not relevant for our context (must provide justification)
4. **Wait for user decisions** before moving to the next perspective

### Discussion Protocol

- If the user disagrees with a finding: discuss it, but require a justification
- If a BLOCKER is marked "ignore": push back once with the impact statement, then accept the user's decision with documented justification
- Keep the discussion focused on the specific finding — do not re-open brainstorming

## Step 4: Consolidate and Update Spec

After all 4 perspectives are reviewed:

1. **Summary table:**
   ```markdown
   ## Challenge Summary
   | Perspective | BLOCKER | HIGH | MEDIUM | LOW | Fixed | Accepted | Ignored |
   |-------------|---------|------|--------|-----|-------|----------|---------|
   | Security    |    X    |  X   |   X    |  X  |   X   |    X     |    X    |
   | Performance |    X    |  X   |   X    |  X  |   X   |    X     |    X    |
   | DX          |    X    |  X   |   X    |  X  |   X   |    X     |    X    |
   | DAU-Stress  |    X    |  X   |   X    |  X  |   X   |    X     |    X    |
   ```

2. **Update the Design Spec** (`concept.md`) with all "Fix in spec" changes
3. **Add a "Challenge Log" section** to the spec documenting all accepted and ignored findings with justifications
4. **Write challenge results** to `docs/agentforge/features/<domain>/<slug>/challenge-concept.md`

5. **Update CURRENT-STATE.md:**
   ```yaml
   phase: challenge-concept
   status: complete
   blockers_resolved: true/false
   findings_total: X
   findings_fixed: X
   findings_accepted: X
   findings_ignored: X
   ```

## Terminal State

When all findings are resolved (no unresolved BLOCKERs) and the spec is updated:

**Invoke `agentforge:plan-feature` skill to create the implementation plan.**

## Red Flags — You Are Violating This Skill If:

| What You Are Doing | Why It Is Wrong |
|---------------------|----------------|
| Presenting all 4 perspectives at once | User cannot process that much information |
| Findings without concrete scenarios | Vague findings are not actionable |
| Fabricating findings to fill a quota | False positives waste time and erode trust |
| Moving past a BLOCKER without resolution | BLOCKERs exist for a reason |
| Re-opening brainstorming discussions | This phase is about "what can go wrong", not "what should we build" |
| Skipping a perspective because "it looks fine" | Say "zero findings" explicitly — that is valid |
| Accepting "it should be fine" for a BLOCKER | Require concrete justification |

## Rules

- Follow `base-instructions.md` for all process rules
- All 4 agents run in parallel — do not run them sequentially
- Present results to user sequentially — one perspective at a time
- Every finding must reference a specific part of the Design Spec
- Every "ignore" decision must have a documented justification
- No unresolved BLOCKERs may remain when this phase completes
- If a finding leads to a significant spec change, flag that the affected section needs re-review
