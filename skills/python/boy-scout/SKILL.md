---
name: boy-scout
description: Use after completing a code change or before committing — applies the boy-scout rule to leave touched code cleaner than found.
---

# The Boy Scout Rule

> "Always check a module in cleaner than when you checked it out." — Robert C. Martin

## Overview

Every code change should leave the code a little better than you found it. Not perfect — just better. Over time, better compounds into excellent.

## When to Use

- Every time you touch code for any reason
- After fixing a bug — clean up what's around it
- During implementation — improve what you see as you go

**Not for:** Massive refactors unrelated to your task. Keep changes proportional.

## Quick Wins (Do Immediately)

- Rename a poorly named variable (**SEE ALSO:** `clean-names`)
- Delete a redundant comment (**SEE ALSO:** `clean-comments`)
- Remove dead code or unused imports
- Replace a magic number with a named constant
- Extract a deeply nested block into a well-named function

## Deeper Improvements (When Time Allows)

- Split a multi-responsibility function (**SEE ALSO:** `clean-functions`)
- Remove duplication (DRY) (**SEE ALSO:** `clean-general`)
- Add missing boundary checks
- Improve test coverage (**SEE ALSO:** `clean-tests`)

## Example

```python
# Asked to fix a bug in this function — don't just fix and leave:
def proc(d, x, flag=False):
    # process data
    for i in d:
        if i > 0:
            if flag:
                x.append(i * 1.0825)  # tax
            else:
                x.append(i)
    return x

# Leave it cleaner:
TAX_RATE = 0.0825

def process_positive_values(
    values: list[float],
    apply_tax: bool = False
) -> list[float]:
    """Filter positive values, optionally applying tax."""
    rate = 1 + TAX_RATE if apply_tax else 1
    return [v * rate for v in values if v > 0]
```

Fixed: N1 (names), G25 (magic number), F2 (no mutation), C4 (useful docstring), P3 (type hints).

## Skill Orchestration

| Task | Trigger Skill |
|------|---------------|
| Writing/reviewing any Python | `python-clean-code` (master reference) |
| Naming variables, functions, classes | `clean-names` |
| Writing or editing comments | `clean-comments` |
| Creating or refactoring functions | `clean-functions` |
| Reviewing code quality | `clean-general` |
| Writing or reviewing tests | `clean-tests` |
| Systematic file-level review | `clean-code-review` (full checklist) |

## AI Behaviour

When working on code, complete the requested task first. Then make the small cleanups the touched code warrants — none, if it is already clean — using the matching specialized skill, and report each with rule number and severity (e.g., "[nit] N1: renamed `d` to `daily_returns`").

When reviewing code:
1. Load `python-clean-code` for comprehensive rule checking
2. Use `clean-code-review` for systematic file-level review with severity labels
3. Suggest incremental improvements, not complete rewrites
