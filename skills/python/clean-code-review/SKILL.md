---
name: clean-code-review
description: Use when reviewing or fixing individual files for code quality issues — naming smells, function design, dead code, duplication, commented-out code, primitive obsession, mutable defaults, broad exceptions, test flakiness, or missing boundary tests.
---

# Clean Code Review — File-Level Checklist

## Overview

Systematic file-level code quality review using Robert Martin's Clean Code catalog, Fowler's refactoring smells, and test design principles. Find violations, fix them, report by rule number.

## When to Use

- Reviewing changed files during code review
- Cleaning up a file after implementation
- Spot-checking code quality on any file

**Not for:** Architectural or cross-file issues — review those across the whole change first.

## Quick Reference

| Category | Key Rules | Common Violations |
|----------|-----------|-------------------|
| **Names** | N1-N7 | Vague names (`d`, `tmp`), encodings (`str_name`), missing side-effect description |
| **Functions** | F1-F4 | >3 args, boolean flags, dead functions |
| **Comments** | C1-C5 | Redundant comments, commented-out code, metadata in comments |
| **General** | G3-G36 | Duplication (G5), magic numbers (G25), mixed abstraction (G34), train wrecks (G36) |
| **Smells** | RF4-RF7 | Repeated switches, data clumps, primitive obsession, middle man |
| **GPU** | GPU1-GPU5 | Tensor loops, graph breaks, missing compile tests, CPU-tensor in forward, per-batch host syncs |
| **Python** | PY1-PY5 | Mutable defaults, broad `except`, mutable class attrs, bare `assert` |
| **Tests** | T1-T9 | Missing boundary tests, slow tests, trivial behaviour left untested |
| **Test Design** | TD1-TD6 | Untestable code, implementation-coupled tests, non-determinism |

## How To Use

1. Review each changed file against the checklists below
2. Fix every violation found
3. Assign severity: **blocking** (bugs/major pain) · **important** (significant improvement) · **suggestion** (alternative approach) · **nit** (minor)
4. Report each fix with rule number, severity, and location
5. Note strengths — what was done well

## Names (N1-N7)

- [ ] **N1**: Descriptive, reveals intent — no `d`, `x`, `proc`, `tmp`
- [ ] **N2**: Right abstraction level — not implementation-specific
- [ ] **N3**: Standard nomenclature (domain terms, patterns)
- [ ] **N4**: Unambiguous — clear what's being operated on
- [ ] **N5**: Length matches scope — short for loops, long for globals
- [ ] **N6**: No encodings — no `str_`, `lst_`, `i_` prefixes
- [ ] **N7**: Describes side effects — `get_or_create_config()` not `get_config()` if it creates

## Functions (F1-F4)

- [ ] **F1**: Max 3 arguments — use dataclass for more
- [ ] **F2**: No output arguments — return values, don't mutate args
- [ ] **F3**: No flag/boolean arguments — split into separate functions
- [ ] **F4**: No dead/uncalled functions — delete, Git remembers

## Comments (C1-C5)

*Canonical source: the `clean-comments` skill. Keep these checks in sync with it; load it when a file's comments need deeper scrutiny.*

- [ ] **C1**: No metadata, ephemeral context, **change-history narration**, **contrast with roads not taken**, or **justification that correct code is correct** — author names, dates, ticket/issue/PR refs (`<ISSUE-ID>`, #123), commit hashes, "added for X flow" rationale, "used by Y" cross-references, "we used to… / previously… / now instead of… / this replaces…" descriptions of how the code changed, "rather than… / instead of… / we don't…" framing against an approach the code doesn't take, and "…because this must be thread-safe / or we'd NPE / so a malformed message can't crash us" defences of an implementation that simply meets its contract. Comments describe what the code does *now*, positively; history lives in Git / the PR, the road not taken is invisible to a future reader, and a fix's reasoning belongs in the commit message. A comment accurate about the current code that still contrasts it with the past or a rejected alternative, or merely argues the code is right, fails C1 — keep it only when it names a constraint the reader could not derive and would otherwise break (an external quirk, an ordering requirement, a measured cost).
- [ ] **C2**: No obsolete comments describing changed/removed code
- [ ] **C3**: No redundant comments restating the code
- [ ] **C4**: Comments explain WHY not WHAT, well-written and **concise** — the fewest lines that convey the why; cut clauses that restate the code, filler, or over-explanation. A multi-line comment must earn each line (tighten or refactor if it doesn't).
- [ ] **C5**: No commented-out code — delete, Git remembers

## Diagnostic Messages (DM1)

- [ ] **DM1**: New log / error / panic / exception messages match the file or module's existing wording template — verb, tense, capitalization, and format-arg style. Sample 2–3 sibling calls in the file before adding one, and flag deviations (e.g. `"X task failed: {err}"` where siblings say `"failed to <action>: {err}"`; `"Failed to…"` capitalized where the file uses lowercase; positional `{}` + `.display()` where siblings inline `{err}`). Consistency here is what lets a reader grep and scan logs; a lone off-pattern message is noise.

## General (G-rules)

- [ ] **G3**: Boundary conditions handled explicitly
- [ ] **G5**: DRY — no duplicated logic
- [ ] **G9**: No dead code — unreachable branches, unused variables
- [ ] **G10**: Variables declared near usage
- [ ] **G16**: No obscured intent — clarity over cleverness
- [ ] **G19**: Explanatory variables for complex expressions
- [ ] **G23**: Polymorphism over long if/else chains
- [ ] **G25**: Named constants, not magic numbers
- [ ] **G28**: Conditionals encapsulated in well-named functions
- [ ] **G29**: No negative conditionals (`if is_valid()` not `if not is_invalid()`)
- [ ] **G30**: Functions do one thing
- [ ] **G34**: One abstraction level per function
- [ ] **G36**: Law of Demeter — no train wrecks (`a.b.c.d`)

## Fowler's Smells (RF4-RF7)

- [ ] **RF4**: No repeated switches — same if/else chain in multiple places. Use polymorphism or dispatch dict.
- [ ] **RF5**: No data clumps — same 3+ variables together in multiple places. Extract to dataclass.
- [ ] **RF6**: No primitive obsession — strings/ints/floats for domain concepts. Create value objects.
- [ ] **RF7**: No middle man — class that delegates nearly everything. Remove if >50% methods are pass-through.

## GPU & torch.compile Compatibility (GPU1-GPU5)

- [ ] **GPU1**: No Python for-loops over tensor dimensions — use vectorized ops (`cumsum`, `gather`, `scatter`, broadcasting). Loops cause graph breaks and prevent GPU parallelism.
- [ ] **GPU2**: torch.compile(fullgraph=True) compatible — no graph breaks (dynamic Python control flow on tensor values, data-dependent shapes, Python-side list accumulation from tensors). Test with `backend="eager"`.
- [ ] **GPU3**: New model heads/layers must have `@pytest.mark.extended` compile smoke tests — forward pass, backward pass, and multi-dim input.
- [ ] **GPU4**: No CPU-only tensor creation in forward paths — `torch.empty(0)` sentinels, `torch.tensor(...)` without device. Use input tensor's device or `register_buffer`.
- [ ] **GPU5**: No per-batch GPU→CPU sync points in hot paths. `.item()`, `.tolist()`, `.cpu()`/`.to("cpu")` followed by `.numpy()`, and `bool(tensor)`/`int(tensor)`/`float(tensor)` coercions all force the GPU to drain its work queue. Forbidden inside `training_step`, `validation_step`, `test_step`, `predict_step`, `forward`, `on_*_batch_start`, `on_*_batch_end`, and any helper they call per batch. Cause: choppy/low GPU utilisation, especially visible in validation. Allowed patterns: defer the sync to epoch-end (accumulate 0-d GPU tensors with `+=` / `torch.maximum`, drain once in `on_*_epoch_end`); one-batch lag (queue this batch's check, consume next batch's result so the GPU has time to finish async); single batched `.tolist()` over a whole 1-D tensor instead of N `.item()`s in a loop. Note: `non_blocking=True` on `.to("cpu")` is silently synchronous unless the destination is pinned memory — don't rely on it without `pin_memory()`.

## Python Anti-Patterns (PY1-PY5)

- [ ] **PY1**: No mutable default arguments — `def foo(items=[])` shares across calls. Use `None`.
- [ ] **PY2**: No broad exception catching — bare `except:` or `except Exception` that swallows errors. Catch specific.
- [ ] **PY3**: No mutable class attributes — `class Foo: items = []` shared across instances. Move to `__init__`.
- [ ] **PY4**: No string formatting for SQL/commands — use parameterized queries, `subprocess` with list args.
- [ ] **PY5**: No bare `assert` in production — stripped with `-O`. Use `if/raise`.

## Tests (T1-T9)

Apply when reviewing test files:

- [ ] **T1**: Everything that could break is tested
- [ ] **T3**: Trivial behaviour tested by its effect, not by restating a value
- [ ] **T5**: Boundary conditions tested
- [ ] **T6**: Near-bug areas exhaustively tested — bugs cluster
- [ ] **T9**: Fast tests (< 100ms for unit tests)
- [ ] **FIRST**: Fast, Independent, Repeatable, Self-Validating, Timely
- [ ] One concept per test function

## Test Design (TD1-TD6)

Apply when reviewing test files:

- [ ] **TD1**: Code testable in isolation — dependencies injectable, no hard-wired I/O
- [ ] **TD2**: Tests verify behaviour (outputs, effects), not implementation (internal calls, sequences)
- [ ] **TD3**: Right test level — unit for logic, integration for component interaction
- [ ] **TD4**: Failure modes tested — invalid input, missing data, permission denied. Not just happy path.
- [ ] **TD5**: No test logic — no if/else, loops, or try/catch in tests. Straight-line code only.
- [ ] **TD6**: Deterministic — no `datetime.now()`, random values, or uncontrolled network/filesystem

## Reporting Format

```
### file_path

**Strengths:**
- Clean separation of parsing and validation logic

**Violations found and fixed:**
- [blocking] G5 (line 42): Extracted duplicated validation into `validate_input()`
- [important] RF6 (line 23): Replaced raw float for currency with `Money` dataclass
- [nit] C5 (line 88): Deleted commented-out legacy code

**No violations:** F1-F4, PY1-PY5
```
