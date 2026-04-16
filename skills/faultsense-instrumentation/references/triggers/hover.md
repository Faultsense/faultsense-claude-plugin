---
title: fs-trigger="hover"
description: Start an assertion when the mouse enters the element.
category: trigger
name: hover
since: 0.1.0
status: stable
aliases: [mouseenter]
---

# `fs-trigger="hover"`

Fires when the mouse enters the element. Implemented as an alias for `mouseenter` (non-bubbling, once per enter — not the noisier `mouseover`).

## Syntax

```html
<div class="product-card"
  fs-assert="catalog/show-quick-view"
  fs-trigger="hover"
  fs-assert-visible=".quick-view-overlay">
```

## When to use it

- Hover-activated tooltips or quick-view overlays
- Preview-on-hover UI (image thumbnails expanding, video auto-play)
- Menu drawers that expand on pointer entry

## Example

```html
<button class="help-button"
  fs-assert="help/show-tooltip"
  fs-trigger="hover"
  fs-assert-visible=".tooltip">
  ?
</button>
```

## Pairs well with

- [`fs-assert-visible`](../assertions/visible.md) — tooltip or preview appears
- [`fs-assert-added`](../assertions/added.md) — lazily-mounted preview is inserted
- [`fs-assert-timeout`](../assertions/timeout.md) — cap how fast the hover reaction needs to be

## Gotchas

- **Touch devices.** There's no hover on touch — the assertion will never fire on phones and tablets. Mirror it with a [`click`](click.md)- or [`focus`](focus.md)-triggered variant if you need mobile coverage.
- **Rapid enter/leave.** Flicking the cursor in and out can produce many trigger fires in quick succession. Each starts a fresh assertion.

## See also

- [`click`](click.md) — touch-safe alternative
- [`focus`](focus.md) — keyboard-accessible alternative
- [Triggers index](../triggers.md)
