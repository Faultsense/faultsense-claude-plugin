---
title: fs-trigger="event:<name>"
description: Start an assertion when a named CustomEvent fires on document, optionally filtered by event.detail properties.
category: trigger
name: event
since: 0.1.0
status: stable
---

# `fs-trigger="event:<name>"`

Starts an assertion when a named `CustomEvent` fires on `document`. Use to hook into your app's own event bus — cart updates, auth state changes, pub/sub channels — without caring which element dispatched it.

## Syntax

```html
<!-- Any dispatch of the named event triggers the assertion -->
<div fs-assert="cart/sync"
  fs-trigger="event:cart-updated"
  fs-assert-visible="#cart-count[text-matches=\d+]">
</div>

<!-- With detail-matches: only trigger when event.detail properties match -->
<div fs-assert="cart/item-added"
  fs-trigger="event:cart-updated[detail-matches=action:increment]"
  fs-assert-updated="#cart-count">
</div>
```

## When to use it

- App-wide state changes dispatched via CustomEvents
- Decoupled cross-component verification (any component dispatches, any element can assert)
- HTMX `HX-Trigger` response headers that fan out state updates after a swap
- Anywhere your app already has an event bus and you want to piggyback on it

## `detail-matches` on triggers

`detail-matches` filters which dispatches count as a trigger. Format: `[detail-matches=key:value]`. Multiple chains: `[detail-matches=a:1][detail-matches=b:2]`.

- **Shallow string equality.** `key` is a top-level property name on `event.detail`; `value` is the literal string it must equal. No regex, no nested paths.
- **Missing keys fail the match.** A dispatch without the named detail property won't trigger the assertion.
- **This is different from the `emitted` assertion.** [`fs-assert-emitted`](../assertions/emitted.md) uses regex for `detail-matches`, not string equality.

## Example

```html
<!-- App dispatches this after a server sync -->
<script>
  document.dispatchEvent(new CustomEvent('cart-updated', {
    detail: { action: 'add', itemId: '123' }
  }));
</script>

<!-- Triggered only on 'add' events -->
<div fs-assert="cart/on-add"
  fs-trigger="event:cart-updated[detail-matches=action:add]"
  fs-assert-visible=".cart-count[text-matches=\d+]">
</div>
```

## Pairs well with

- [`fs-assert-visible`](../assertions/visible.md) — verify UI reflects the state change
- [`fs-assert-updated`](../assertions/updated.md) — verify a cross-component element mutated
- [`fs-assert-emitted`](../assertions/emitted.md) — assert a downstream event also fires
- [`fs-assert-after`](../assertions/after.md) — sequence after a prior assertion

## Gotchas

- **Shallow equality only.** `detail-matches` on triggers does string equality, not regex. `[detail-matches=count:5]` matches `detail.count === "5"` (string) or `5` (number coerced to string), not `>= 5`.
- **Events fire on `document`.** The agent attaches a single delegated listener at `document` level. Elements using `fs-trigger="event:..."` don't need to be the dispatch target — any element can assert on the event.
- **Mixing up trigger vs emitted detail-matches semantics.** On `fs-trigger="event:..."`, `detail-matches` is string equality. On [`fs-assert-emitted`](../assertions/emitted.md), it's regex. Both use the same syntax, different semantics.

## See also

- [`fs-assert-emitted`](../assertions/emitted.md) — the assertion-type counterpart
- [Patterns cookbook #11](../patterns.md#11-custom-event-trigger--emitted-assertion)
- [HTMX framework notes — HX-Trigger header](../frameworks/htmx.md)
