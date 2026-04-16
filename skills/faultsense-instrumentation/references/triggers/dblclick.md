---
title: fs-trigger="dblclick"
description: Start an assertion when the user double-clicks the element.
category: trigger
name: dblclick
since: 0.1.0
status: stable
---

# `fs-trigger="dblclick"`

Fires on a native `dblclick` event. Use for UI patterns that genuinely require a double-click — inline-edit activation, select-and-open, open-details gestures.

## Syntax

```html
<li fs-assert="todos/enter-edit-mode" fs-trigger="dblclick"
    fs-assert-visible=".todo-edit-input">
  Write the docs
</li>
```

## When to use it

- Inline edit activation on list items
- Open-details gestures where single-click has a different meaning
- Legacy desktop-style UI patterns ported to the web

For most web UI, a single [`click`](click.md) is the right choice. `dblclick` is niche.

## Example

```html
<li class="todo-item"
  fs-assert="todos/enter-edit-mode"
  fs-trigger="dblclick"
  fs-assert-updated=".todo-item[data-mode=edit]">
  Write the docs
</li>
```

## Pairs well with

- [`fs-assert-visible`](../assertions/visible.md) — reveal an edit input
- [`fs-assert-updated`](../assertions/updated.md) — flip the element to edit mode

## Gotchas

- **Mobile and touch devices.** Native `dblclick` is unreliable on touch. If you support touch, prefer a long-press or a tap-to-edit pattern with a single [`click`](click.md).
- **Timing.** Browsers fire two `click` events before the `dblclick` — if you also have a `click` assertion on the same element, both will fire. Put the trigger on distinct elements or use different assertion keys.

## See also

- [`click`](click.md) — the single-click alternative
- [Triggers index](../triggers.md)
