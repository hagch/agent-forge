---
name: plan-creation
description: >
  Use when creating implementation plans from approved concepts.
  Produces granular task-DAGs with dependencies, skills, context
  files, and acceptance criteria. Triggers: planning phase, task
  breakdown, implementation strategy.
---

# Plan Creation

Create implementation plans as task-DAGs that agents can execute without questions.

## Task Granularity

Each task should be completable in a single agent context (2-15 minutes of focused work). If you find yourself writing a task description longer than 10 lines, split it.

**Each step in a task is one action:**
- "Write the failing test" — one step
- "Run it to verify it fails" — one step
- "Implement the minimal code" — one step
- "Run tests to verify green" — one step
- "Commit" — one step

## Task Format

```markdown
### Task <PREFIX>-<N>: <Title>

- **Description:** <Concrete, implementable description>
- **Files:**
  - CREATE: <exact/path/to/file>
  - MODIFY: <exact/path/to/file>
- **Skills:** [tdd-cycle, systematic-debugging]
- **Context:** [<path/to/relevant/doc>, <path/to/example/code>]
- **Depends on:** [<TASK-IDs>]
- **Produces:** <What downstream tasks need from this>
- **Acceptance:** <Specific, testable — "tests pass" or "endpoint returns 201">
- **Complexity:** S | M | L
```

**Prefixes:** BE (backend), FE (frontend), T (test), E2E (end-to-end), OPS (devops)

## Dependency Graph

After all tasks, visualize:
1. Text-based DAG showing task flow
2. Parallel groups (tasks that can run simultaneously)

## No Placeholders Rule

These are plan failures — never write:
- "TBD", "TODO", "implement later"
- "Add appropriate error handling"
- "Write tests for the above" (without specifying WHICH tests)
- "Similar to Task N" (repeat the content — agents may read tasks out of order)

## Self-Review Checklist

After writing the plan:
1. Does every acceptance criterion in the concept have a task?
2. Are all file paths real and correct?
3. Do type/method names match across tasks?
4. Can every task be understood without reading other tasks?
5. Is there at least one E2E test task?
