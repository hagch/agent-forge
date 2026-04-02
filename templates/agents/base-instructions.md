# Base Instructions

These rules apply to every agent that writes or modifies code.

## Iron Law: Evidence Before Assertions

Before reporting ANY task as done:

1. Run the relevant tests and read the FULL output
2. Verify the output shows what you expect — green tests, correct behavior
3. If tests fail: fix the issue, do not report as done
4. If you cannot run tests: explain why, do not claim success

No completion claims without fresh verification evidence.

| Claim | Required Evidence |
|-------|-------------------|
| "Tests pass" | Paste test runner output showing pass count |
| "Feature works" | Run it, show the output |
| "Build succeeds" | Paste build output |
| "Bug is fixed" | Show the failing test now passes |

### Red Flags — You Are Violating This Rule If:

- You say "should work", "probably passes", "seems correct"
- You claim success without running a command
- You trust your own code without executing it
- You mark a task done after writing code but before testing

## TDD Enforcement

No production code without a failing test first. The Red-Green-Refactor cycle is mandatory for every code change.

1. **Red:** Write a failing test that describes the expected behavior
2. **Green:** Write the minimum production code to make the test pass
3. **Refactor:** Clean up while keeping tests green

Every TDD cycle ends with a commit. Do not batch multiple cycles into one commit.

## Conventional Commits

Every commit follows the Conventional Commits format:

```
type(scope): description
```

- **Types:** feat, fix, docs, style, refactor, test, chore
- **Scope:** module, component, or area affected
- **Description:** imperative mood, under 50 characters, lowercase start

Examples:
- `feat(auth): add token refresh on 401 response`
- `fix(users): prevent duplicate email registration`
- `test(orders): add edge case for empty cart checkout`
- `refactor(api): extract validation into middleware`

Each TDD cycle produces one commit. Red-Green-Refactor = one `test(scope)` commit for the test + one `feat|fix(scope)` commit for the implementation, or a single commit if the change is small enough.

## Problem Documentation

When you encounter ANY problem during implementation:

1. Document in the feature doc under `## Agent Log`
2. Use this format:

```
### Problem: <title>
**Agent:** <your agent name>
**Task:** <task ID>
**Context:** What were you trying to do?
**Error:** What happened? (paste actual error)
**Investigation:** What did you try?
**Solution:** What fixed it?
**Root Cause:** Why did this happen?
**Prevention:** How to avoid this in the future?
```

## Success Documentation

Also document what worked on first try and why — this feeds the self-improvement loop:

```
### Success: <title>
**Agent:** <your agent name>
**Task:** <task ID>
**What:** What worked well?
**Why:** Why did it work? (pattern followed, existing example found, etc.)
```

## Guardrails

- Before starting a task: read ALL previous guardrail entries for this task
- A guardrail documents what went wrong in a previous attempt and what NOT to do
- If a guardrail exists: follow its guidance, do not repeat the failed approach
- Document what went wrong and what NOT to do so future attempts avoid the same path
- 3x same error on the same task = STOP and escalate to orchestrator

## Confusion Detection

If you notice any of these signs, STOP immediately:

- You are trying the same approach for the third time
- You are undoing changes you just made
- You are not sure what the task is asking
- Your changes keep breaking other things
- You have been working on the same task for more than 10 minutes without progress

When you stop:
1. Document your current state in the Agent Log
2. Describe what you understand and where you are stuck
3. Escalate with full context — do NOT keep trying without a new approach

## Code Standards

- Read and follow ALL rules from CLAUDE.md before writing any code
- If CLAUDE.md conflicts with these base instructions, CLAUDE.md wins for code style; base instructions win for process (verification, documentation, guardrails)
- When in doubt about a pattern: read existing code in the same layer/directory first

## File Ownership

Only modify files within your declared boundaries. If a task requires changes outside your ownership, document the need and escalate — do not reach into another agent's territory.

## State Management

Update CURRENT-STATE.md after completing work. This is the single source of truth for session continuity. Include: what was done, what remains, any blockers or guardrails discovered.

## Research Patterns

- Use the "Think" tool for complex reasoning before acting
- Context-first: read relevant files before making changes
- Kill criteria: if stuck 3+ iterations on identical error, stop and escalate
