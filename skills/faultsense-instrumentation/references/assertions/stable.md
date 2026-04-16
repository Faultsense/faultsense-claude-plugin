---
title: fs-assert-stable
description: Passes when the matched element's subtree is NOT mutated during the timeout window.
category: assertion
name: stable
since: 0.1.0
status: stable
---

# `fs-assert-stable`

The temporal inverse of [`updated`](updated.md). Passes when the matched element's subtree is **not** mutated during the timeout window. Any mutation within the window fails the assertion.

## Syntax

```
fs-assert-stable="<selector>"
```

## When to use it

- Flicker detection — "after I add to cart, the price should not update again for 500ms"
- Settlement checks — "after the modal opens, the form shouldn't re-render during the transition"
- Perpetual "never mutate" guards on immutable UI (legal notice, static footer)

## Example — flicker detection (OOB + stable)

The most useful pattern. Pair [`stable`](stable.md) with [OOB](oob.md) so the stability window begins after the primary mutation:

```html
<!-- Primary: add to cart -->
<button fs-assert="cart/add-item" fs-trigger="click"
  fs-assert-updated="#cart-total">
  Add to Cart
</button>

<!-- OOB: verify price doesn't flicker after the cart update -->
<div
  fs-assert="cart/price-stable"
  fs-assert-oob="cart/add-item"
  fs-assert-stable="#cart-total"
  fs-assert-timeout="500">
</div>
```

## Example — perpetual immutability

```html
<div id="legal-notice"
  fs-assert="layout/legal-stable"
  fs-trigger="invariant"
  fs-assert-stable="#legal-notice">
</div>
```

This fails if `#legal-notice` or any descendant is ever mutated.

## Timeout semantics

`stable` is unique: it commits on the first mutation (failure) rather than waiting for a match. Without [`fs-assert-timeout`](timeout.md), `stable` passes via the GC sweep (default 5s) — use an explicit timeout for tighter stability windows.

- **With OOB:** the timeout window starts when the parent assertion passes, not at page load.
- **With invariant:** runs perpetually for the page lifetime. Any mutation, any time, is a failure.
- **With user triggers:** window starts at trigger fire. Rare — OOB is almost always the right pairing.

## Pairs well with

- [OOB](oob.md) — the canonical pairing for flicker detection
- [`fs-trigger="invariant"`](../triggers/invariant.md) — perpetual immutability guards
- [`fs-assert-timeout`](timeout.md) — explicit stability window

## Gotchas

- **Scope matters.** `fs-assert-stable="#foo"` fails on ANY mutation in `#foo`'s subtree. If `#foo` is inside your app's root swap target, every re-render mutates it. Place stability sentinels OUTSIDE swap targets, or narrow the selector to an element that isn't part of routine re-renders.
- **Framework transitions.** CSS animations via class toggles count as mutations. If your UI animates into place, the animation class toggle will fail a `stable` check targeting the same element. Target the inner stable content instead.
- **Not a "nothing happened" check.** `stable` only cares about the selected subtree. Events, network calls, and mutations elsewhere in the DOM are ignored.

## See also

- [`updated`](updated.md) — the temporal complement
- [`fs-assert-oob`](oob.md) — the canonical pairing
- [Patterns cookbook #12](../patterns.md#12-stable-assertion-no-flickering)
- [Assertions index](../assertions.md)
