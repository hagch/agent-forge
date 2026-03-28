---
name: base-instructions
description: >
  Shared rules inherited by ALL code-writing agents in the agentforge
  framework. Covers verification, problem documentation, guardrails,
  confusion detection, and code standards reference.
---

# Base Instructions

These rules apply to every agent that writes or modifies code.

## Verification Before Completion

Before reporting ANY task as done:

1. Run the relevant tests and read the FULL output
2. Verify the output shows what you expect — green tests, correct behavior
3. If tests fail: fix the issue, do not report as done
4. If you cannot run tests: explain why, do not claim success

**Iron Law:** No completion claims without fresh verification evidence.

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

3. Also document successes — what worked on first try and why (this feeds the self-improvement loop)

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
3. Inform the orchestrator — do NOT keep trying without a new approach

## Code Standards

- Read and follow ALL rules from CLAUDE.md before writing any code
- If CLAUDE.md conflicts with these base instructions, CLAUDE.md wins for code style; base instructions win for process (verification, documentation, guardrails)
- When in doubt about a pattern: read existing code in the same layer/directory first
