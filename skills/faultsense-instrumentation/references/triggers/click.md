---
title: fs-trigger="click"
description: Start an assertion when the user clicks the element or any of its descendants.
category: trigger
name: click
since: 0.1.0
status: stable
---

# `fs-trigger="click"`

Starts an assertion when the user clicks the instrumented element. By far the most common trigger — use it for buttons, links, and any clickable UI.

## Syntax

```html
<button fs-assert="feature/action" fs-trigger="click" fs-assert-added=".result">
  Do the thing
</button>
```

## When to use it

- Buttons that trigger an action
- Anchor links, including in-app navigation
- Any `div`, `span`, or custom element that has a click handler
- Submit buttons inside forms (either `click` on the button or `submit` on the form — either works)

## Example

```html
<button
  fs-assert="todos/add-item"
  fs-trigger="click"
  fs-assert-added=".todo-item">
  Add
</button>
```

Passes when a new `.todo-item` appears in the DOM after the click.

## Descendant clicks work

The agent walks up from `event.target` to the nearest `fs-trigger` ancestor via `closest()`. You can put the instrumentation on a `<button>` and not worry about clicks landing on nested icons, spans, or text — they all resolve to the button.

```html
<button
  fs-assert="cart/add-item"
  fs-trigger="click"
  fs-assert-updated="#cart-count">
  <svg class="cart-icon">…</svg>
  Add to cart
</button>
```

Clicks on the SVG icon still fire the assertion. No `pointer-events: none` hack needed.

## Pairs well with

- [`fs-assert-added`](../assertions/added.md) — the most common pairing (clicked the button, new element appeared)
- [`fs-assert-updated`](../assertions/updated.md) — clicked the button, existing element content changed
- [`fs-assert-removed`](../assertions/removed.md) — clicked the button, element is gone
- [`fs-assert-emitted`](../assertions/emitted.md) — clicked the button, custom event fired (use async dispatch)
- [Conditional assertions](../assertions/conditional.md) — `added-success` / `added-error` for branch outcomes
- [`fs-assert-mutex`](../assertions/mutex.md) — group cross-type conditionals together

## Gotchas

- **Placing `fs-trigger="click"` on a container `<div>` that wraps multiple unrelated children.** It fires for ANY click inside, producing noisy assertions. Put the trigger on the specific element the user is meant to interact with.
- **HTMX form submit buttons also fire `click`.** HTMX calls `preventDefault()` on submit AFTER listeners have run, so both `fs-trigger="click"` on the button and `fs-trigger="submit"` on the form work. Pick one.
- **Synchronous `dispatchEvent` in the click handler** fires before an [emitted](../assertions/emitted.md) assertion listener is registered. Use async dispatch (setTimeout, queueMicrotask, or an HTMX `HX-Trigger` response header).

## See also

- [Triggers index](../triggers.md)
- [Placement rules](../triggers.md#placement-rules)
