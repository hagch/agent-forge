---
name: systematic-debugging
description: >
  Use when encountering any bug, test failure, or unexpected behavior
  during implementation. 4-phase root cause analysis before proposing
  fixes. Triggers: test failure, runtime error, unexpected behavior,
  stuck for more than 2 attempts.
---

# Systematic Debugging

**Iron Law:** NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST.

## Phase 1 — Root Cause Investigation

1. **Read the error** — the FULL error message, stack trace, and context. Do not skim.
2. **Reproduce consistently** — can you trigger it every time? If not, find the conditions.
3. **Track recent changes** — what changed since it last worked? `git diff`, `git log`
4. **Gather evidence** — add logging, print statements, or breakpoints to narrow down WHERE the failure occurs.

## Phase 2 — Pattern Analysis

1. **Find a working example** — is there similar code in the project that works correctly?
2. **Compare** — what is different between the working example and the failing code?
3. **Check documentation** — does the framework/library documentation explain this behavior?

## Phase 3 — Hypothesis and Testing

1. **Form a hypothesis** — "I think X fails because Y"
2. **Design a minimal test** — what is the smallest change that would confirm or deny this?
3. **Test the hypothesis** — make ONLY that change, observe the result
4. **If confirmed** → proceed to Phase 4
5. **If denied** → return to Phase 1 with new information

## Phase 4 — Implementation

1. Write a failing test that captures the bug
2. Implement the fix — SINGLE change, not multiple
3. Run the failing test — it must now pass
4. Run ALL tests — nothing else must break
5. Document the fix in the Agent Log

## Escalation Rule

**If you have tried 3 different fixes and none work:**

STOP. You are likely misunderstanding the root cause. Document:
- What you tried (all 3 approaches)
- What happened each time
- Your current understanding of the problem

Then escalate to the orchestrator. Do NOT try a 4th fix.

## Red Flags — You Are Debugging Wrong If:

| What You Are Doing | What To Do Instead |
|---------------------|-------------------|
| Changing random things to see what happens | Form a hypothesis first |
| Fixing without understanding the cause | Phase 1 — investigate |
| Making multiple changes at once | One change at a time |
| Ignoring the error message | Read it. All of it. |
| "It works on my machine" | Reproduce the exact failing conditions |
| Googling the error before reading it | Read first, search second |
