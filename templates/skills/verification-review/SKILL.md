---
name: verification-review
description: >
  Use when reviewing implemented code for quality, security, and plan
  adherence. Provides structured multi-pass review with backend,
  frontend, architecture perspectives. Triggers: code review, verification
  phase, quality gate.
---

# Verification Review

Multi-pass code review with structured perspectives.

## Pass Structure

Complete each pass fully before starting the next:

1. **Backend Pass** — Correctness, patterns, security, performance
2. **Frontend Pass** — UI components, i18n, states, responsive, a11y
3. **Architecture Pass** — Plan adherence, consistency, coupling

## Per-Finding Format

```markdown
#### [BLOCKER|HIGH|MEDIUM|LOW] <Title>
- **Pass:** Backend | Frontend | Architecture
- **File:** <path:line>
- **Issue:** <What is wrong — be specific>
- **Fix:** <Concrete change needed>
```

## What to Check Per Pass

### Backend
- Tests match expected behavior?
- Project patterns followed? (Check CLAUDE.md)
- Input validation on all user-facing endpoints?
- Auth checks present?
- No N+1 queries, unbounded operations?
- Error handling uses i18n codes?

### Frontend
- Project's component library used? No raw HTML?
- All text uses i18n? Both language files updated?
- Loading, error, empty states present?
- Forms use project form handling?
- Responsive at mobile breakpoints?

### Architecture
- Implementation matches the plan?
- New code consistent with existing patterns?
- No unnecessary new abstractions?
- Names clear and self-documenting?
- Files in correct directories?

## Evidence Rule

**Iron Law:** No completion claims without fresh verification evidence.

For every claim, show the evidence:
- "Tests pass" → paste test output
- "Build succeeds" → paste build output
- "Feature works" → describe what you ran and what you observed
