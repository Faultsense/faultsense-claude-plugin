---
title: fs-trigger="input"
description: Start an assertion on every keystroke in a text input or textarea.
category: trigger
name: input
since: 0.1.0
status: stable
---

# `fs-trigger="input"`

Fires on the native `input` event — every keystroke, every paste, every IME composition commit. Use for live feedback UI where the page reacts as the user types.

## Syntax

```html
<input type="text"
  fs-assert="search/live-filter"
  fs-trigger="input"
  fs-assert-updated=".results">
```

## When to use it

- Live search or autocomplete where results update as the user types
- Character counters and validation hints that update per keystroke
- Instant-preview inputs (markdown editor, color picker)

For "commit" semantics (only after the user is done typing), use [`change`](change.md) or [`blur`](blur.md) instead.

## Example

```html
<input type="text"
  fs-assert="search/filter"
  fs-trigger="input"
  fs-assert-updated=".results-count[text-matches=\d+ results]">
```

Each keystroke starts a fresh assertion that waits for `.results-count` to update.

## Pairs well with

- [`fs-assert-updated`](../assertions/updated.md) — the live-filtered results mutate
- [`fs-assert-visible`](../assertions/visible.md) — autocomplete dropdown appears
- [`fs-assert-timeout`](../assertions/timeout.md) — cap how long live feedback may take

## Gotchas

- **High frequency.** `input` fires on every keystroke. Multiple assertions for the same key may be in flight at once — if your UI debounces updates, set [`fs-assert-timeout`](../assertions/timeout.md) to an appropriate value or use [`change`](change.md) instead.
- **IME composition.** During a composition (e.g., Japanese input), `input` can fire multiple times per logical character. Expect interim states.

## See also

- [`change`](change.md) — commit-semantics alternative
- [`blur`](blur.md) — trigger on focus loss
- [Triggers index](../triggers.md)
