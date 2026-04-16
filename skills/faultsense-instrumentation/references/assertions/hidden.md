---
title: fs-assert-hidden
description: Resolves when the matched element exists but has no layout dimensions.
category: assertion
name: hidden
since: 0.1.0
status: stable
---

# `fs-assert-hidden`

Resolves when an element matching the selector exists in the DOM but has zero layout dimensions. Query-based — runs a point-in-time `querySelector` and layout check, not a mutation watch. The complement of [`visible`](visible.md).

## Syntax

```
fs-assert-hidden="<selector>"
```

## When to use it

- Collapsible or closeable UI where the element persists but isn't visible (accordions, drawers, modals with `display: none`)
- "This should never appear" invariants (error banners, stuck loading spinners)
- Verifying a close/cancel action hides an existing element

If the element is removed entirely (not just hidden), use [`removed`](removed.md) instead.

## Example

```html
<button
  fs-assert="modal/close"
  fs-trigger="click"
  fs-assert-hidden=".modal-overlay">
  Close
</button>
```

Passes when `.modal-overlay` has zero layout dimensions after the click.

## Invariant use — "should never appear"

`fs-assert-hidden` combined with `fs-trigger="invariant"` is the idiomatic way to guard against unwanted UI:

```html
<div
  fs-assert="layout/no-error-banner"
  fs-trigger="invariant"
  fs-assert-hidden=".global-error-banner">
</div>
```

This fails the moment `.global-error-banner` gains dimensions, and recovers when it goes hidden again.

## Pairs well with

- [`fs-trigger="click"`](../triggers/click.md) — close, dismiss, collapse
- [`fs-trigger="invariant"`](../triggers/invariant.md) — canonical "should never appear" pattern
- [OOB assertions](oob.md) — state-type choice for side-effect checks
- [`[classlist=...]`](../modifiers/classlist.md) — combine with a class-state check

## Gotchas

- **Element must exist in the DOM.** If the element is removed, `hidden` does not pass — use [`removed`](removed.md) instead.
- **`opacity: 0` is NOT hidden.** The element still has dimensions. Use a [`[classlist=hidden:true]`](../modifiers/classlist.md) or attribute check for opacity-based hiding.

## See also

- [`visible`](visible.md) — the complementary type
- [`removed`](removed.md) — for "no longer in the DOM"
- [Assertions index](../assertions.md)
