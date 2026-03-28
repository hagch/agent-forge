---
name: tdd-cycle
description: >
  Use when implementing any feature or bugfix. Enforces strict
  Red-Green-Refactor cycle. Triggers: writing code, implementing tasks,
  fixing bugs, adding features.
---

# TDD Cycle

**Iron Law:** NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST.

## The Cycle

```
   ┌─────────────────────────────────────────┐
   │                                         │
   ▼                                         │
 RED ──→ GREEN ──→ REFACTOR ──→ COMMIT ──→ NEXT
 Write    Write     Clean up    Save        Loop
 failing  minimal   without     progress    back
 test     code      changing
                    behavior
```

## RED — Write a Failing Test

1. Write a test that describes ONE specific behavior
2. The test name IS the documentation: `shouldRejectNegativeAmount`, `rendersErrorWhenApiReturns500`
3. Run the test
4. **VERIFY IT FAILS FOR THE RIGHT REASON**
   - Wrong reason: syntax error, missing import, wrong test setup → fix the test
   - Right reason: the functionality does not exist yet → proceed to GREEN

## GREEN — Minimal Implementation

1. Write the MINIMUM code to make the test pass
2. Do NOT write code that no test requires
3. Do NOT anticipate future requirements
4. Run the test — it must pass
5. Run ALL related tests — nothing must break

## REFACTOR — Clean Up

1. Look at what you just wrote with fresh eyes
2. Can you simplify? Remove duplication? Improve names?
3. Make changes WITHOUT altering behavior
4. Run ALL tests after every refactoring change
5. If tests break during refactor → revert, try again smaller

## COMMIT

After each Red-Green-Refactor cycle:
```bash
git add <test-file> <implementation-file>
git commit -m "<type>: <description>"
```

## Red Flags — You Are Violating TDD If:

| What You Are Doing | Why It Is Wrong |
|---------------------|----------------|
| Writing implementation before the test | You do not know what "correct" looks like |
| Writing more code than the test requires | YAGNI — you are guessing at requirements |
| Test passes on first run | Either the test is wrong or the code already existed |
| Skipping the RED verification | You cannot know the test works if you never saw it fail |
| Refactoring and adding features simultaneously | One thing at a time |
| Not running tests after refactoring | Refactoring can introduce bugs |
| "This is too simple to test" | Simple things become complex. Test it. |
| "I will write tests after" | You will not. And if you do, they will test your implementation, not the requirement. |

## Integration with Agent Log

After each TDD cycle:
- If a problem occurred: document in `## Agent Log` → Problems
- If it went smoothly: document in `## Agent Log` → Successes
- If a guardrail was relevant: note that you followed it and whether it helped
