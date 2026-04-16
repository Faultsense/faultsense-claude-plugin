---
title: fs-assert-emitted
description: Resolves when a matching CustomEvent fires on document.
category: assertion
name: emitted
since: 0.1.0
status: stable
---

# `fs-assert-emitted`

Resolves when a `CustomEvent` with the given name fires on `document`. Use to assert that your app dispatches the expected events after a user action — payment-complete, cart-updated, auth-changed.

## Syntax

```
fs-assert-emitted="<eventName>"
fs-assert-emitted="<eventName>[detail-matches=key:pattern]"
```

## When to use it

- Event-bus or pub/sub flows where "the event fired" is a correctness contract
- Analytics contract verification ("clicking Buy dispatches `purchase:init`")
- Decoupled verification — assert the event from any element, regardless of who dispatched

## Example

```html
<button fs-assert="checkout/payment" fs-trigger="click"
  fs-assert-emitted="payment:complete[detail-matches=orderId:\d+]">
  Pay Now
</button>
```

Passes when a `payment:complete` CustomEvent fires on `document` AND its `detail.orderId` matches the regex `\d+`.

## `detail-matches` on `emitted` uses regex

Unlike [`fs-trigger="event:..."`](../triggers/event.md), which uses shallow string equality, `detail-matches` on `emitted` is **regex matching**:

```
fs-assert-emitted="order:created[detail-matches=total:\$\d+\.\d{2}]"
```

This matches `detail.total === "$12.50"`, `"$1.00"`, etc.

## Synchronous dispatch limitation

If the `CustomEvent` is dispatched **synchronously** inside the same click handler, the event fires before the `emitted` assertion listener is created. The listener is attached when the assertion is initialized — which happens after the trigger handler runs.

Fix: use async dispatch.

```js
// BAD — synchronous dispatch
button.addEventListener('click', () => {
  doWork();
  document.dispatchEvent(new CustomEvent('work:done'));  // fires before listener exists
});

// GOOD — async dispatch
button.addEventListener('click', () => {
  doWork();
  queueMicrotask(() => {
    document.dispatchEvent(new CustomEvent('work:done'));
  });
});

// GOOD — HTMX HX-Trigger response header (async by design)
// server sets: HX-Trigger: {"work:done": {"ok": true}}
```

## Pairs well with

- [`fs-trigger="click"`](../triggers/click.md) — user actions that should dispatch events
- [`fs-trigger="submit"`](../triggers/submit.md) — form submits that fire events on success
- [`fs-assert-visible`](visible.md) — verify the UI caught up alongside the event
- [Conditional assertions](conditional.md) — `emitted-success` + `added-error` branches

## Gotchas

- **Not compatible with MPA mode.** MPA assertions are persisted across page navigations and resolved on the next load. CustomEvents don't survive a navigation. The agent warns and ignores `fs-assert-mpa="true"` on `emitted` assertions.
- **Synchronous dispatch in the trigger handler** (see above) — the event fires before the listener is created. Use async dispatch.
- **Detail-matches regex vs trigger detail-matches equality.** Same syntax, different semantics. On the [trigger](../triggers/event.md) side it's string equality; on the `emitted` side it's regex. This is easy to trip on when copying between the two.
- **Dispatching on `window` instead of `document`.** The agent listens on `document`. Events dispatched on `window` don't bubble — they won't be caught.

## See also

- [`fs-trigger="event:<name>"`](../triggers/event.md) — the trigger counterpart
- [HTMX HX-Trigger header](../frameworks/htmx.md) — the canonical async-dispatch path for server-rendered apps
- [Assertions index](../assertions.md)
