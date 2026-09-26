---
name: clean-names
description: Use when naming, renaming, or fixing names of variables, functions, classes, or modules — vague names, implementation-leaking names, Hungarian notation, or names that hide side effects.
when_to_use: |
  Also trigger on: single-letter or cryptic identifiers (`d`, `x`, `proc`), Hungarian notation (`str_name`, `lst_users`, `i_count`), `I`-prefixed classes, function names that hide side effects (e.g. `get_config` that also writes a file), non-standard abbreviations, or asks like "rename this", "what does this variable mean", "clearer name".
---

# Clean Names (N1-N7)

## Overview

Names should reveal intent. If a name requires a comment, it doesn't reveal its intent.

## Rules

### N1: Choose Descriptive Names

```python
# Bad — what is d?
d = 86400

# Good — obvious meaning
SECONDS_PER_DAY = 86400
```

### N2: Right Abstraction Level

Don't expose implementation in the name.

```python
# Bad
def get_dict_of_user_ids_to_names(): ...

# Good
def get_user_directory(): ...
```

### N3: Standard Nomenclature

Use domain terms, design patterns, and well-known conventions.

```python
class UserFactory:
    def create(self, data): ...

def calculate_amortization(principal, rate, term): ...
```

### N4: Unambiguous Names

```python
# Bad
def rename(old, new): ...

# Good
def rename_file(old_path: Path, new_path: Path): ...
```

### N5: Length Matches Scope

Short names for tiny scopes. Long, descriptive names for globals and module-level.

```python
total = sum(x for x in numbers)          # Fine — tiny scope
MAX_RETRY_ATTEMPTS_BEFORE_FAILURE = 5     # Good — module level
MAX = 5                                    # Bad — too short for module level
```

### N6: No Encodings

Don't encode type or scope information. Modern editors make this unnecessary.

```python
# Bad — Hungarian notation
str_name = "Alice"
lst_users = []

# Good
name = "Alice"
users = []
```

### N7: Describe Side Effects

```python
# Bad — hides file creation
def get_config(): ...

# Good — reveals behaviour
def get_or_create_config(): ...
```

## Quick Reference

| Rule | Principle | Example |
|------|-----------|---------|
| N1 | Descriptive names | `SECONDS_PER_DAY` not `d` |
| N2 | Right abstraction | `get_user_directory()` not `get_dict_of_...` |
| N3 | Standard nomenclature | `UserFactory`, `calculate_amortization` |
| N4 | Unambiguous | `rename_file(old_path, new_path)` |
| N5 | Length matches scope | Short for loops, long for globals |
| N6 | No encodings | `users` not `lst_users` |
| N7 | Describe side effects | `get_or_create_config()` |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Renaming a public API name without checking callers | Grep first; renames have blast radius |
| Choosing a name that documents the current implementation | Name by intent/role, not by mechanism — so the name survives refactors |
| Single-letter loop indices in long bodies | Acceptable in tight scope; switch to a real name once the loop body grows past ~5 lines |
| Abbreviating to save keystrokes (`usr`, `cfg`, `mgr`) | Spell it out — modern editors autocomplete; readers don't |
| Names that lie (`get_user()` that also writes to a cache) | If it has side effects, say so (`load_user_and_cache()`) — see N7 |
