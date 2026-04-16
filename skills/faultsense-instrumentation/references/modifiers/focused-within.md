---
title: "[focused-within=true|false]"
description: Check whether focus is anywhere inside this element's subtree.
category: modifier
name: focused-within
since: 0.1.0
status: stable
---

# `[focused-within=true|false]`

Checks whether focus is anywhere inside the element's subtree. Implemented as `element.matches(':focus-within')`.

## Syntax

```
fs-assert-<type>="<selector>[focused-within=true]"
fs-assert-<type>="<selector>[focused-within=false]"
```

## When to use it

- Container-level focus contracts — "the modal should contain focus"
- Tab-trap verification on modals and drawers
- Verifying focus didn't escape a focus-locked region
- Broader than [`focused`](focused.md) — care about the subtree, not a specific element

## Example — modal contains focus

```html
<div class="modal-overlay"
  fs-assert="modal/focus-trapped"
  fs-trigger="mount"
  fs-assert-visible=".modal-overlay[focused-within=true]">
</div>
```

Passes when the modal is visible AND focus is somewhere inside it (any input, button, or focusable descendant).

## Example — assert focus escaped

```html
<button
  fs-assert="dropdown/close-returns-focus"
  fs-trigger="click"
  fs-assert-updated=".dropdown[focused-within=false]">
  Close
</button>
```

Passes when the dropdown is updated AND focus has moved outside it.

## Semantics

- **Point-in-time CSS `:focus-within` check.** Uses native `matches(':focus-within')` which the browser computes from the focus tree.
- **Includes the element itself.** If the instrumented element has focus directly, `focused-within` is also true.
- **Works across shadow DOM boundaries** in browsers that correctly implement `:focus-within` with shadow DOM.

## Pairs well with

- [`fs-assert-visible`](../assertions/visible.md) — verify visibility AND internal focus
- [`fs-trigger="mount"`](../triggers/mount.md) — verify focus was set when the container mounted
- [Self-referencing selectors](../assertions/self-referencing.md) — check the instrumented container itself

## Gotchas

- **Container without any focusable descendants.** A `<div>` with only text is never `focused-within=true`. Add `tabindex="-1"` to make the container itself focusable if that's the desired contract.
- **Initial focus timing.** Like [`focused`](focused.md), focus management races with assertion timing. Expect to pair with a small [timeout](../assertions/timeout.md) or with an event trigger.

## See also

- [`focused`](focused.md) — for element-specific focus
- [Modifiers index](../modifiers.md)
