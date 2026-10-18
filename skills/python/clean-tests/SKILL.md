---
name: clean-tests
description: Use when writing, fixing, or reviewing tests — missing boundary tests, slow tests, flaky tests, skipped tests, tests with multiple asserts, or tests coupled to implementation details.
when_to_use: |
  Also trigger on: slow or flaky tests, `@pytest.mark.skip` without a clear reason, tests that only cover the happy path, tests with multiple assertions about different concepts, missing boundary cases (empty input, off-by-one, page zero), or asks about "coverage gap", "edge case", "did we test".
---

# Clean Tests (T1-T9)

## Overview

Tests should be fast, independent, and test one concept each. Test everything that could break, especially boundary conditions.

## Rules

### T1: Test Everything That Could Break

```python
# Bad — only happy path
def test_divide():
    assert divide(10, 2) == 5

# Good — edge cases too
def test_divide_normal():
    assert divide(10, 2) == 5

def test_divide_by_zero():
    with pytest.raises(ZeroDivisionError):
        divide(10, 0)

def test_divide_negative():
    assert divide(-10, 2) == -5
```

### T2: Use Coverage Tools

```bash
pytest --cov=myproject --cov-report=term-missing
```

Aim for meaningful coverage, not 100%.

### T3: Test Trivial Behaviour

Small behaviours still deserve a test: a default that changes what the code does, an edge a one-liner handles. Assert the behaviour the value drives, not the value itself.

```python
def test_new_user_cannot_delete_others():
    user = User(name="Alice")
    # A member can't delete others, so this asserts the default's effect.
    assert not user.can_delete_others()
```

### T4: Ignored Test = Ambiguity Question

Don't use `@pytest.mark.skip` to hide problems: fix the test. Skipping or deleting a test needs the user's approval.

### T5: Test Boundary Conditions

Bugs congregate at boundaries. Test them explicitly.

```python
def test_pagination_boundaries():
    items = list(range(100))
    assert paginate(items, page=1, size=10) == items[0:10]     # First page
    assert paginate(items, page=10, size=10) == items[90:100]   # Last page
    assert paginate(items, page=11, size=10) == []              # Beyond last
    assert paginate([], page=1, size=10) == []                  # Empty list
    with pytest.raises(ValueError):
        paginate(items, page=0, size=10)                        # Invalid
```

### T6: Exhaustively Test Near Bugs

When you find a bug, write tests for all similar cases. Bugs cluster.

### T7: Patterns of Failure Are Revealing

When tests fail, look for patterns — they point to deeper issues.

### T8: Coverage Patterns Are Revealing

Untested code paths often reveal design problems. If you can't easily test a function, it probably does too much.

### T9: Tests Must Be Fast

Slow tests don't get run. Keep unit tests under 100ms each.

```python
# Bad — hits real database
def test_user_creation():
    db = connect_to_database()  # Slow!
    ...

# Good — in-memory
def test_user_creation():
    db = InMemoryDatabase()
    ...
```

## FIRST Principles

- **Fast**: Run quickly
- **Independent**: Don't depend on each other
- **Repeatable**: Same result every time, any environment
- **Self-Validating**: Pass or fail, no manual inspection
- **Timely**: Written before or with the code

## One Concept Per Test

```python
# Bad — multiple concepts
def test_user():
    user = User("Alice", "alice@example.com")
    assert user.name == "Alice"
    assert user.is_valid()
    user.activate()
    assert user.is_active

# Good — one concept each
def test_user_stores_name(): ...
def test_new_user_is_valid(): ...
def test_user_can_be_activated(): ...
```

## Quick Reference

| Rule | Principle |
|------|-----------|
| T1 | Test everything that could break |
| T2 | Use coverage tools |
| T3 | Test trivial behaviour |
| T4 | Ignored test = ambiguity |
| T5 | Test boundary conditions |
| T6 | Exhaustively test near bugs |
| T7 | Patterns in failures reveal deeper issues |
| T8 | Coverage gaps reveal design problems |
| T9 | Tests must be fast (< 100ms) |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Tests pass but assert nothing meaningful | Every test must fail if the production code is wrong; review each assert |
| Shared mutable fixture state between tests | Use function-scoped fixtures or explicit resets; flakiness compounds across tests |
| Asserting implementation details (mock call counts) instead of observable behaviour | Assert outcomes, not internals; tests should survive refactors |
| One test method asserting many concepts | Split per concept (see One Concept Per Test); failure messages stay diagnostic |
| Catching the bug only in the test that exposed it | Apply T6 — search for sibling cases and pin them too |

**SEE ALSO:** `clean-code-review` — test design principles (testability, determinism, no test logic) in section TD1-TD6.
