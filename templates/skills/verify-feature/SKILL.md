---
name: verify-feature
description: >
  Use when all slices are implemented and slice-level reviews have
  passed. Final quality gate that verifies the complete feature as
  a whole, not individual slices. Triggers: all slices done, feature
  verification, final quality gate, pre-merge review.
---

# Verify Feature — Full-Feature Quality Gate

Verify the complete feature across all slices. This is the final gate before branch completion. Unlike slice-level reviews that check parts, this checks the WHOLE.

## Prerequisites

Before starting, verify:
1. CURRENT-STATE.md shows all slices completed with reviews passed
2. All slice-level reviews (spec-compliance, code-quality, mini-verification) are PASS
3. Feature branch has all commits from development phase
4. Read `base-instructions.md` — the Iron Law applies with full force here

## Verification Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                  6 Parallel Verification Passes               │
│                                                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │ 1. Cross-    │  │ 2. Arch      │  │ 3. Security  │       │
│  │    Slice     │  │    Consistency│  │              │       │
│  │    Integration│  │              │  │              │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │ 4. Perf      │  │ 5. DAU Full  │  │ 6. Compound  │       │
│  │              │  │    Journey   │  │    Gate      │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
│                                                               │
│  All 6 run in PARALLEL → findings merged → fix routing        │
└──────────────────────────────────────────────────────────────┘
```

Dispatch all 6 passes in parallel (Agent Teams if available, otherwise sequential subagents).

---

## Pass 1: Cross-Slice Integration

<cross-slice-integration>

**Question:** Do the slices work together as a coherent feature?

Check:
- API contracts consistent between slices (request/response shapes match)
- Data flow correct end-to-end: from user input through all slices to final output
- Shared state (database, cache, session) used consistently across slices
- No conflicting migrations or schema assumptions between slices
- Event/message contracts match between producers and consumers
- Error propagation: does a failure in slice N surface correctly in slice M?

Evidence required:
- Run integration tests that span multiple slices
- Trace at least one complete data flow through ALL slices
- Paste test output showing cross-slice tests pass

Per-finding format:
```markdown
#### [BLOCKER|HIGH|MEDIUM|LOW] <Title>
- **Pass:** Cross-Slice Integration
- **Slices Affected:** <slice IDs>
- **Issue:** <what breaks between slices>
- **Evidence:** <test output, code reference>
- **Fix:** <concrete change, which slice/file>
```

</cross-slice-integration>

## Pass 2: Architecture Consistency

<architecture-consistency>

**Question:** Does the new code fit the existing codebase?

Check:
- New patterns consistent with existing codebase conventions (read CLAUDE.md)
- No duplicate abstractions across slices (e.g., two different HTTP clients, two validation approaches)
- Dependency direction correct: no circular imports, no layer violations
- File placement follows project structure conventions
- Naming conventions consistent with existing code
- No unnecessary new dependencies introduced
- Configuration follows existing patterns (env vars, config files)

Layer violation detection:
- UI must not import directly from DB/repository layer
- Business logic must not depend on framework-specific types
- Shared utilities must not import from feature modules

Per-finding format:
```markdown
#### [BLOCKER|HIGH|MEDIUM|LOW] <Title>
- **Pass:** Architecture Consistency
- **File:** <path:line>
- **Issue:** <what is inconsistent>
- **Existing Pattern:** <how the codebase does it>
- **Fix:** <concrete change to match existing pattern>
```

</architecture-consistency>

## Pass 3: Security

<security>

**Question:** Does this feature introduce security vulnerabilities?

Categorical checklist — check EVERY category:

| Category | What to Verify |
|----------|----------------|
| Input Validation | All user input validated and sanitized at entry point |
| Authentication | Auth required on all new endpoints? Correct roles? |
| Authorization | Users can only access their own data? Horizontal privilege escalation blocked? |
| Injection | SQL injection, XSS, command injection, template injection — across ALL slices |
| Data Exposure | Sensitive data in logs, error messages, API responses, client-side state? |
| Cryptography | Passwords hashed? Tokens secure? No secrets in code? |
| CSRF/CORS | Cross-origin protections in place for state-changing operations? |
| Rate Limiting | Abuse-prone endpoints rate-limited? |

Rules:
- For each finding, describe a concrete EXPLOIT SCENARIO (not just "this could be a problem")
- Confidence threshold: only report findings with confidence >= 0.7
- Below 0.7 confidence = too speculative, do not report
- Evidence before assertions: show the vulnerable code, explain the attack vector

Per-finding format:
```markdown
#### [BLOCKER|HIGH|MEDIUM|LOW] <Title>
- **Pass:** Security
- **Category:** <from checklist>
- **File:** <path:line>
- **Exploit Scenario:** <step-by-step how an attacker exploits this>
- **Evidence:** <vulnerable code snippet>
- **Fix:** <concrete remediation>
```

</security>

## Pass 4: Performance

<performance>

**Question:** Will this feature perform well under realistic load?

Check:
- Database queries: N+1 problems, missing indexes, unbounded result sets
- Cross-slice queries: do multi-slice flows generate excessive DB roundtrips?
- Caching strategy coherent across slices (no stale cache, no cache stampede)
- Big-O implications: any O(n^2) or worse in data processing paths?
- Payload sizes: API responses proportional to what the client needs?
- Concurrency: race conditions, deadlocks under concurrent access?
- Resource cleanup: connections closed, streams drained, temp files removed?

Load scenario reasoning:
- Consider 10x current load — what breaks first?
- Identify the hot path and analyze its complexity
- Check for operations that scale with data size (not just request count)

Per-finding format:
```markdown
#### [BLOCKER|HIGH|MEDIUM|LOW] <Title>
- **Pass:** Performance
- **File:** <path:line>
- **Scenario:** <load pattern that triggers the issue>
- **Impact:** <what degrades — latency, memory, throughput>
- **Evidence:** <code reference, Big-O analysis>
- **Fix:** <concrete optimization>
```

</performance>

## Pass 5: DAU Full Journey

<dau-full-journey>

**Question:** Can a non-technical, impatient user complete the entire feature flow?

Load the `dau-ui-test` skill for the persona and test script pattern.

This is NOT per-slice UI testing. This is the COMPLETE user journey through ALL slices combined.

Test script:
1. **Discovery** — Start from the app's main entry point. Can the user FIND the feature?
2. **Onboarding** — First interaction with the feature. Is it obvious what to do?
3. **Happy Path** — Complete the full flow end-to-end through all slices. Every step clear?
4. **Error Recovery** — Break things deliberately. Are error messages human-readable? Can the user recover?
5. **Edge Cases** — Long input, special characters, back button, multiple tabs, refresh mid-flow
6. **Mobile** — Full journey on 375px viewport. All interactions reachable? No horizontal scroll?
7. **Accessibility** — Full journey via keyboard only. WCAG 2.1 AA compliance. Screen reader landmarks.

Frustration Rating (per step):
- 1 = Smooth, intuitive
- 2 = Minor confusion, self-recoverable
- 3 = Noticeable friction, might need help
- 4 = Significant confusion, likely to abandon
- 5 = Cannot complete without external help

Verdict: PASS if no step exceeds frustration 3, and average frustration <= 2.

Per-finding format:
```markdown
#### [BLOCKER|HIGH|MEDIUM|LOW] <Title>
- **Pass:** DAU Full Journey
- **Journey Step:** <discovery|onboarding|happy-path|error-recovery|edge-cases|mobile|accessibility>
- **Frustration:** <1-5>
- **What Happened:** <step-by-step user experience>
- **Expected:** <what the user expected to happen>
- **Fix:** <concrete UX change>
```

</dau-full-journey>

## Pass 6: Compound Gate

<compound-gate>

**Question:** Do ALL tests pass and is the feature shippable?

This is the hard gate — no subjective judgment, just evidence.

1. Run the FULL test suite (unit + integration + E2E)
2. Paste the COMPLETE test output — do not summarize or truncate
3. Count: total tests, passed, failed, skipped
4. If ANY test fails → automatic FAIL, no exceptions
5. Build succeeds with no warnings treated as errors
6. Linting passes (if configured)

Verdict format:
```markdown
### Compound Gate Verdict

| Metric | Result |
|--------|--------|
| Unit Tests | <X> passed, <Y> failed, <Z> skipped |
| Integration Tests | <X> passed, <Y> failed, <Z> skipped |
| E2E Tests | <X> passed, <Y> failed, <Z> skipped |
| Build | PASS / FAIL |
| Lint | PASS / FAIL / N/A |

**Verdict:** PASS / FAIL
**Evidence:** <link to test output above>
```

</compound-gate>

---

## Finding Routing

After all 6 passes complete, merge findings and route by severity:

### BLOCKER or HIGH

1. Identify the responsible specialist agent (from development phase)
2. Route findings to that agent with:
   - The specific findings
   - The affected files
   - The required fix (from the reviewer)
3. Agent applies fix and re-runs affected tests
4. Re-run ONLY the affected verification pass (not all 6)
5. Max 2 fix rounds, then escalate to user

### MEDIUM or LOW

1. Document in the verification report
2. Present to user at the checkpoint
3. User decides: fix now, fix later, or accept

### Confidence Threshold

<confidence>
- Only report findings with confidence >= 0.7
- Below 0.7 = too speculative to act on
- Each finding must be discrete and actionable
- "Evidence before assertions" — if you cannot point to specific code, do not report it
- False positives waste developer time — when in doubt, leave it out
</confidence>

---

## Verification Report

After all passes complete (and fixes applied), compile the report:

```markdown
# Verification Report: <feature-name>

## Summary
- **Total Findings:** <count by severity>
- **Fix Rounds:** <how many fix iterations were needed>
- **Test Results:** <total passed / failed / skipped>
- **Verdict:** PASS / FAIL / PASS WITH NOTES

## Pass Results

### 1. Cross-Slice Integration: PASS / FAIL
<findings or "No issues found">

### 2. Architecture Consistency: PASS / FAIL
<findings or "No issues found">

### 3. Security: PASS / FAIL
<findings or "No issues found">

### 4. Performance: PASS / FAIL
<findings or "No issues found">

### 5. DAU Full Journey: PASS / FAIL
<frustration summary, findings or "No issues found">

### 6. Compound Gate: PASS / FAIL
<test counts, build status>

## Accepted Risks
<MEDIUM/LOW findings that user accepted>

## DAU Journey Summary
<end-to-end user experience narrative>
```

## User Checkpoint

After the verification report is compiled:

1. Present the full report to the user
2. Highlight any accepted risks (MEDIUM/LOW findings)
3. Ask for explicit approval to proceed
4. Update CURRENT-STATE.md: `human_checkpoint_pending: yes`
5. Wait for approval before continuing

---

## Red Flags — Verification Is Going Wrong If:

| Signal | What To Do |
|--------|------------|
| Reviewer claims "looks good" without running tests | Violation of Iron Law — re-run with evidence requirement |
| Findings lack specific file/line references | Too vague to act on — reviewer must be concrete |
| Fix round introduces new failures | Stop, revert fix, re-analyze |
| Same finding keeps recurring across fix rounds | Likely architectural issue — escalate to user |
| Compound Gate fails but reviewer says PASS | Compound Gate overrules — tests are the truth |
| DAU journey not actually tested in browser | Must use real browser interaction, not imagination |

---

## Terminal State

When the verification report is PASS (or PASS WITH NOTES after user approval):

> **Invoke `agentforge:finish-branch` skill** to integrate the verified feature.
