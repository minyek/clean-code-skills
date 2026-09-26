---
name: clean-general
description: Use when reviewing code for quality issues — duplicated logic, magic numbers, obscured intent, deep nesting, Law of Demeter violations, or functions doing too much.
when_to_use: |
  Also trigger on: duplicated logic across files or branches (G5), magic numbers or hardcoded values (G25), long if/elif chains that should be polymorphism (G23), chained property access like `a.b.c.d` (G36), functions juggling multiple responsibilities (G30), clever one-liners whose intent is not obvious (G16).
---

# General Clean Code Principles

## Overview

Code should be clear, not clever. Every piece of knowledge has one authoritative representation. Functions do one thing.

## Critical Rules

### G5: DRY (Don't Repeat Yourself)

```python
# Bad — duplication
ca_total = subtotal * 1.0825
ny_total = subtotal * 1.07

# Good — single source of truth
TAX_RATES = {"CA": 0.0825, "NY": 0.07}
def calculate_total(subtotal: float, state: str) -> float:
    return subtotal * (1 + TAX_RATES[state])
```

### G16: No Obscured Intent

```python
# Bad — what does this do?
return (x & 0x0F) << 4 | (y & 0x0F)

# Good — obvious intent
return pack_coordinates(x, y)
```

### G23: Prefer Polymorphism to If/Else

```python
# Bad — will grow forever
def calculate_pay(employee):
    if employee.type == "SALARIED":
        return employee.salary
    elif employee.type == "HOURLY":
        return employee.hours * employee.rate

# Good — open/closed principle
class SalariedEmployee:
    def calculate_pay(self): return self.salary

class HourlyEmployee:
    def calculate_pay(self): return self.hours * self.rate
```

### G25: Named Constants, Not Magic Numbers

```python
# Bad
if elapsed_time > 86400: ...

# Good
SECONDS_PER_DAY = 86400
if elapsed_time > SECONDS_PER_DAY: ...
```

### G30: Functions Do One Thing

If you can extract another function, your function does more than one thing.

### G36: Law of Demeter (No Train Wrecks)

```python
# Bad — reaching through multiple objects
output_dir = context.options.scratch_dir.absolute_path

# Good — one dot
output_dir = context.get_scratch_dir()
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Extracting a constant for a one-off value | Inline single-use values; extract only when reused or non-obvious (G25) |
| Reaching for polymorphism with only two cases | If there will only ever be 2-3 variants, a conditional is fine; abstract on the third (G23) |
| DRY-ing two snippets that look similar but mean different things | Coincidental duplication isn't duplication — keep them separate (G5) |
| Adding a method on the caller to satisfy Law of Demeter | Move the behaviour to the object that owns the data, don't just hide the chain (G36) |

## Enforcement Checklist

- [ ] No duplication (G5)
- [ ] Clear intent, no magic numbers (G16, G25)
- [ ] Polymorphism over conditionals (G23)
- [ ] Functions do one thing (G30)
- [ ] No Law of Demeter violations (G36)
- [ ] Boundary conditions handled (G3)
- [ ] Dead code removed (G9)

**For systematic file-level review**, use the `clean-code-review` skill which covers all G-rules plus Fowler smells, Python anti-patterns, and test design.
