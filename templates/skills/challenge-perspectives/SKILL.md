---
name: challenge-perspectives
description: >
  Use when reviewing a concept or implementation plan from multiple
  perspectives. Provides structured role switching through Security,
  Performance, DX, and DAU viewpoints. Triggers: concept review,
  plan review, design challenge.
---

# Challenge Perspectives

A structured approach to stress-testing concepts and plans from 4 viewpoints.

## How to Use

1. Complete each perspective FULLY before moving to the next
2. For each perspective, go through EVERY component in the concept/plan
3. Document findings immediately with severity
4. After all 4 perspectives: summarize total findings by severity

## Perspective Order

1. **Security** — Can it be attacked?
2. **Performance** — Will it be slow?
3. **Developer Experience** — Will it be maintainable?
4. **DAU** — Will it confuse users?

## Per Finding Template

```markdown
### [BLOCKER|HIGH|MEDIUM|LOW] <Title>
- **Perspective:** Security | Performance | DX | DAU
- **Scenario:** <Step-by-step description>
- **Affected Component:** <Specific part>
- **Required Change:** <Concrete fix>
```

## Severity Decision Guide

- Ask: "What happens if we ship without fixing this?"
  - Data loss, security breach, complete feature failure → **BLOCKER**
  - Degraded performance, confusing UX, missing error handling → **HIGH**
  - Inconsistency, minor inefficiency, suboptimal pattern → **MEDIUM**
  - Cosmetic, preference, nice-to-have → **LOW**

## Rules
- Be specific and actionable — vague findings waste time
- Do not fabricate findings — if a perspective yields nothing, say so
- Reference actual components/files from the concept/plan
- "Required Change" must be concrete enough to implement
