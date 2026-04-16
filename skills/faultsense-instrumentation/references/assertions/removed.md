---
title: fs-assert-removed
description: Resolves when an element matching the selector is removed from the DOM.
category: assertion
name: removed
since: 0.1.0
status: stable
---

# `fs-assert-removed`

Resolves when an element matching the selector is removed from the DOM after the trigger fires. Mutation-observed.

## Syntax

```
fs-assert-removed="<selector>"
```

## When to use it

- Deleting items from a list (todos, cart, notifications)
- Closing a modal whose content is unmounted (not just hidden)
- Dismissing a toast by removal
- Cleanup flows where an element should be torn down

If the element stays in the DOM but is hidden (display:none, visibility:hidden), use [`hidden`](hidden.md) instead.

## Example

```html
<button
  fs-assert="todos/remove-item"
  fs-trigger="click"
  fs-assert-removed=".todo-item">
  Delete
</button>
```

Passes when a matching `.todo-item` is removed from the DOM after the click.

## Pre-existing matches matter

Unlike [`added`](added.md), `removed` naturally depends on a pre-existing match — you can't remove what isn't there. The assertion watches the `removedElements` mutation list and resolves when an element matching the selector is removed.

## Pairs well with

- [`fs-trigger="click"`](../triggers/click.md) — delete buttons
- [`fs-trigger="unmount"`](../triggers/unmount.md) — verify cleanup on teardown
- [Conditional assertions](conditional.md) — `removed-success` + `added-error` branches
- [`fs-assert-mutex="each"`](mutex.md) — group the success + error branches

## Gotchas

- **Using `removed` when `hidden` is correct.** If your modal uses `display: none` rather than unmounting, the element is still in the DOM. Use [`hidden`](hidden.md).
- **Batched removals.** A single click may remove many elements in the same microtask. `removed` resolves as soon as ANY matching element is removed — use [`[count=N]`](../modifiers/count.md) for cardinality checks at the removal moment (but note `count` is evaluated on the current DOM, not the removed set).

## See also

- [`added`](added.md) — the complementary type
- [`hidden`](hidden.md) — for "still in DOM, not visible" checks
- [Assertions index](../assertions.md)
