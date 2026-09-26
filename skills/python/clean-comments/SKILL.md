---
name: clean-comments
description: Use when writing, editing, or reviewing comments and docstrings — redundant comments, metadata in comments, stale comments, or commented-out code.
when_to_use: |
  Also trigger on: commented-out code blocks, TODO/FIXME banners, author/ticket/date metadata in comments, docstrings that no longer match the code, redundant comments that restate the code (e.g. `i += 1  # increment i`), or asks like "is this comment useful", "why is this block commented".
---

# Clean Comments (C1-C5)

## Overview

The best comment is the code itself. If you need a comment to explain what code does, refactor first, comment last.

## Rules

### C1: No Inappropriate Information

Comments shouldn't hold metadata or ephemeral context. Use Git history / PR descriptions for author names, change history, ticket/issue/PR references (PROJ-123, #123), commit hashes, and dates. "Added for X flow" rationale and "used by Y" cross-references rot fast — they belong in PR descriptions, not code. Comments are for technical notes about code only.

When an existing file is full of ticket-tagged comments, that is debt to be cleaned up — not a pattern to extend.

**No change-history narration.** A comment must describe what the code *does now*, never what it used to do or how it changed. Delete the before/now contrast — "we used to…", "previously…", "no longer…", "changed from X to Y", "instead of the old…", "this replaces…" — even when it is accurate: the contrast lives in the commit and the diff, and it rots once the old behaviour is forgotten. Keep only the forward-looking constraint a future reader needs, stated with no reference to the past:

```
// Bad — narrates the change (history belongs in git)
// We used to read this inline, but that blocked the UI thread; it now runs in a task.

// Good — states the standing constraint, present tense, no before/after
// Runs in a background task: reading this inline would block the UI thread.
```

**State what the code does, not what it avoids.** Don't frame a comment by contrast with an approach the code *doesn't* take — "rather than threaded through every caller's task", "instead of stat-ing each item", "we don't resolve symlinks here". The road not taken is invisible to a future reader and just rots; describe the present behaviour and its standing reason positively. Keep the contrast *only* when the rejected alternative is the load-bearing rationale a reader genuinely needs (e.g. "a `String` key, not the `Path`, because two paths can canonicalize to one") — and even then state it once, tersely.

```
// Bad — defines itself against a road not taken
// Build the item here on the worker thread rather than in the drain on the GUI thread.

// Good — states what it does and why
// Build the item on the worker thread: this read_dir + stat would block the GUI thread.
```

**Don't justify why correct code is necessary.** A present-tense, positive, history-free comment can still do nothing but argue that the code is right. Correct code needs no defence in the source. If the surrounding API is *meant* to be thread-safe, a lock or a concurrent collection is simply the implementation; "we use a concurrent map here because this is accessed from multiple threads" tells a reader nothing the contract didn't already promise. Likewise the reasoning behind a fix — what was broken, why this handles it, what would go wrong without it — belongs in the **commit message**, where it is attached to the change that needed it, not stapled to the code forever.

```java
// Bad — argues that correct code is correct
// This class is thread-safe, so we must use a ConcurrentHashMap rather than a HashMap.
// Guard against a null runner, otherwise we would NPE here.
// Validate the size first so that a malformed message cannot crash the parser.

// Good — delete it. The contract, the guard, and the validation speak for themselves.
```

The test is what a reader *loses* by deleting the comment. Delete it when the reason is already implied by the type, the contract, the language, or the code on the next line. Keep it only when it names a constraint the reader could not derive and would otherwise break — an external quirk, an ordering requirement, a measured cost:

```java
// Good — a constraint the reader cannot derive and would break
// The feed restates the full order book after a reconnect, so accumulating deltas here double-counts.
```

### C2: Delete Obsolete Comments

If a comment describes code that no longer exists or works differently, delete it immediately. Stale comments are "floating islands of irrelevance and misdirection."

### C3: No Redundant Comments

```python
# Bad — the code already says this
i += 1  # increment i
user.save()  # save the user

# Good — explains WHY, not WHAT
i += 1  # compensate for zero-indexing in display
```

### C4: Write Comments Well

If a comment is worth writing, write it well:
- Choose words carefully
- Use correct grammar
- Don't ramble or state the obvious
- Be brief

**Be concise — make every line earn its place.** Use the fewest lines that convey the WHY. Before keeping a multi-line comment, cut: clauses that restate the adjacent code, filler ("basically", "note that", "in order to"), and re-explanations of the obvious. A three-line comment that says what one line could is not concise — tighten it. If the point genuinely can't fit in a sentence or two, the code likely needs refactoring (see the overview), not a longer comment. This is a soft, judgement-based check (unlike the concrete C1/C5 rules), so apply it deliberately to every comment you write or review.

### C5: Never Commit Commented-Out Code

```python
# DELETE THIS
# def old_calculate_tax(income):
#     return income * 0.15
```

Delete it. Git remembers everything.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Deleting a comment that explains a non-obvious "why" (e.g. workaround for a known bug) | Keep WHY comments; only delete WHAT-restating comments |
| Leaving `TODO` / `FIXME` comments instead of filing tickets | File a ticket, link it externally if needed, don't TODO inline |
| Rewriting an obsolete comment to "match" the new code instead of deleting it | If the comment adds no insight beyond the code, delete it (C2 + C3) |
| Treating a file full of ticket-tagged comments as the convention | That's debt — follow C1, don't extend the pattern |
| Keeping a comment because it's *accurate*, even though it narrates "we used to X, now Y" | Accuracy isn't the test — history doesn't belong in code. Strip the before/now contrast; keep only the present-tense constraint (C1) |
| Explaining why the code has to be correct — "thread-safe here because callers are concurrent", "check for null or we'd NPE" | Correct code needs no defence. Delete; the contract and the guard already say it (C1) |
| Recording why a fix works, next to the fix | Put it in the commit message, where it stays attached to the change (C1) |

## Quick Reference

| Rule | Principle | Fix |
|------|-----------|-----|
| C1 | No metadata, change-history ("we used to…"), contrast with roads not taken ("rather than…"), or justification that correct code is correct ("thread-safe because…", "check this or we'd crash") | Delete — use Git and the commit message; state the present behaviour positively, and defend nothing |
| C2 | No obsolete comments | Delete immediately |
| C3 | No redundant comments | Delete or explain WHY |
| C4 | Write well if you must | Be brief, precise |
| C5 | No commented-out code | Delete — Git remembers |
