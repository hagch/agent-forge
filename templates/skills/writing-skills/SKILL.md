---
name: writing-skills
description: >
  Use when creating new skills or improving existing ones during the
  self-improvement phase. Applies TDD principles to process documentation.
  Triggers: self-improvement identifies need for new skill, agent prompt
  needs improvement, new pattern discovered.
---

# Writing Skills

Apply TDD to skill creation: test first, then write, then bulletproof.

## When to Create a Skill

- A pattern repeats across multiple features
- An agent keeps making the same type of mistake
- A new workflow is established that should be reusable

## When NOT to Create a Skill

- It is a one-time procedure
- It duplicates an existing skill
- It is project-specific (put it in CLAUDE.md instead)

## Skill Structure

```markdown
---
name: <kebab-case-name>
description: >
  Use when <specific trigger conditions>.
  Triggers: <comma-separated list of when to load this skill>.
---

# <Skill Name>

<Core principle or iron law>

## <Main Sections>

<Workflow, rules, examples, templates>

## Red Flags

<What violations look like>

## Rules

<Non-negotiable constraints>
```

## CSO — Claude Search Optimization

The `description` field is CRITICAL for discovery. It must describe WHEN to use the skill, not WHAT it does.

**BAD:** "A comprehensive guide to debugging with 4 phases and root cause analysis"
**GOOD:** "Use when encountering any bug, test failure, or unexpected behavior. Triggers: test failure, runtime error, stuck for more than 2 attempts."

## Quality Checklist

- [ ] Name is kebab-case, descriptive, verb-first
- [ ] Description starts with "Use when" and lists triggers
- [ ] Has clear iron law or core principle
- [ ] Red flags section exists
- [ ] No placeholders or TODOs
- [ ] Tested: does an agent follow it correctly?
