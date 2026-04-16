---
title: fs-assert-mutex
description: Control how conditional siblings compete across assertion types on the same element.
category: feature
name: mutex
since: 0.1.0
status: stable
---

# `fs-assert-mutex`

Controls mutual exclusion of [conditional assertions](conditional.md) across different types on the same element. Without mutex, same-type conditionals (`added-success` / `added-error`) race naturally while cross-type conditionals (`removed-success` / `added-error`) resolve independently. `fs-assert-mutex` lets you override that default.

## Syntax

```
fs-assert-mutex="<mode>"
```

Four modes:

| Mode | Semantics |
|---|---|
| `type` | Same-type conditionals race, cross-type are independent (this is the default when `fs-assert-mutex` is omitted) |
| `each` | **All** conditionals on the element race as one group — first to resolve wins, all others dismissed |
| `conditions` | Condition keys form outcome groups — when one key wins, assertions with different keys are dismissed but same-key assertions resolve independently |
| `<key1>,<key2>` | Selective — only listed condition keys compete; unlisted keys resolve independently |

`fs-assert-mutex=""` is **invalid** and logs a warning. Always provide an explicit mode.

## When to use it

- **`each`** — "exactly one of these outcomes should win, then stop" — the canonical choice for delete flows (removed OR error) and login flows (dashboard OR error).
- **`conditions`** — "success needs several things, but if error wins it cancels them all" — useful when one condition key implies multiple assertion types.
- **selective (`success,error`)** — "these two race, but let this third assertion resolve independently" — useful for analytics pixels that should always fire regardless of success/error.

## `mutex="each"` — first wins, all others dismissed

All conditionals race. First to resolve wins. All others are dismissed.

```html
<button fs-assert="todos/remove-item" fs-trigger="click"
  fs-assert-mutex="each"
  fs-assert-removed-success=".todo-item"
  fs-assert-added-error=".error-msg"
  fs-assert-timeout="5000">
  Delete
</button>
```

If `.todo-item` is removed, `removed-success` wins and `added-error` is dismissed. If the server returns an error and `.error-msg` appears first, `added-error` wins and `removed-success` is dismissed.

## `mutex="conditions"` — condition keys are outcome groups

Condition keys form groups. When one key wins, assertions with **different** keys are dismissed. Same-key assertions resolve independently.

```html
<button fs-assert="todos/add-item" fs-trigger="click"
  fs-assert-mutex="conditions"
  fs-assert-added-success=".todo-item"
  fs-assert-emitted-success="todo:added"
  fs-assert-added-error=".add-error">
  Add
</button>
```

Two outcome groups: `success` and `error`.

- When `success` wins: both `added-success` and `emitted-success` resolve independently (first-to-resolve starts the group, siblings in the same group still need to pass). `added-error` is dismissed.
- When `error` wins: both success assertions are dismissed.

This is the right choice when "success" means "N things must happen" but "error" means "a totally different path was taken."

## `mutex="<key1>,<key2>"` — selective competition

Only listed condition keys race. Unlisted keys resolve independently.

```html
<button fs-assert="checkout/submit" fs-trigger="click"
  fs-assert-mutex="success,error"
  fs-assert-added-success=".confirmation"
  fs-assert-added-error=".error-msg"
  fs-assert-updated-analytics="#tracking-pixel">
  Submit
</button>
```

`success` and `error` are mutually exclusive. `analytics` resolves independently — the tracking pixel should always update regardless of checkout outcome.

## Default (`mutex` omitted or `mutex="type"`)

Same-type conditionals race (`added-success` vs `added-error`), cross-type conditionals are independent (`removed-success` + `added-error` each produce their own signal). This is fine when you genuinely want both cross-type outcomes to resolve, but surprising when you expected them to compete.

## Pairs well with

- [Conditional assertions](conditional.md) — mutex only matters when conditionals exist
- [`fs-assert-timeout`](timeout.md) — bounds the race so dismissed siblings don't linger
- [OOB assertions](oob.md) — multi-check validation per winning branch

## Gotchas

- **`fs-assert-mutex=""` is invalid.** The agent warns and treats it as if omitted.
- **React JSX boolean drop.** React drops bare boolean attributes. Write `fs-assert-mutex="each"`, not `fs-assert-mutex`.
- **Expecting `mutex="each"` to resolve multiple success siblings together.** Each dismisses every other sibling on first match. If you need "both `added-success` and `visible-success` to pass together," use `mutex="conditions"` with the same condition key, or use [OOB](oob.md).
- **Invariant and MPA compatibility.** Mutex works with user triggers. Invariant-triggered assertions don't support conditional keys; MPA assertions don't support `emitted` siblings.

## See also

- [Conditional assertions](conditional.md) — how condition keys work
- [OOB assertions](oob.md) — independent secondary checks
- [Patterns cookbook #6](../patterns.md#6-form-submit-with-conditional-outcomes)
- [Assertions index](../assertions.md)
