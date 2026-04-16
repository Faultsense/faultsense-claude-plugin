---
title: fs-trigger="keydown"
description: Start an assertion on keyboard input — any key, a specific key, or a modifier combo.
category: trigger
name: keydown
since: 0.1.0
status: stable
---

# `fs-trigger="keydown"`

Fires on keyboard input on the instrumented element. Two forms:

- **`keydown`** — any key
- **`keydown:<key>`** — a specific key or modifier combo

## Syntax

```html
<!-- Any key -->
<input fs-assert="editor/keypress" fs-trigger="keydown" fs-assert-updated=".draft-count">

<!-- A specific key -->
<div fs-assert="modal/close-escape" fs-trigger="keydown:Escape"
     fs-assert-removed=".modal" tabindex="0">

<!-- A modifier combo -->
<textarea fs-assert="editor/save" fs-trigger="keydown:ctrl+s"
          fs-assert-visible=".saved-indicator"></textarea>
```

## Key syntax

- **Named keys** match `KeyboardEvent.key`: `Escape`, `Enter`, `ArrowUp`, `Tab`, `Backspace`, `Delete`, `F1`…`F12`, single printable characters (`a`, `/`, `?`).
- **Modifier combos** chain with `+`: `ctrl+s`, `shift+enter`, `alt+f4`, `meta+k`. Order-insensitive.
- **Case-insensitive** on the key name.

## When to use it

- Keyboard shortcuts (Cmd+K command palette, Ctrl+S save, Esc close)
- Key-specific flows (Enter to submit, Arrow keys to navigate a list)
- Global hotkeys attached to a focusable element

## Example

```html
<div id="command-palette-root" tabindex="0"
  fs-assert="commands/open-palette"
  fs-trigger="keydown:meta+k"
  fs-assert-visible=".command-palette">
</div>
```

## Pairs well with

- [`fs-assert-visible`](../assertions/visible.md) — a keyboard-activated overlay appears
- [`fs-assert-removed`](../assertions/removed.md) — Escape closes a modal
- [`[focused=true]`](../modifiers/focused.md) — verify focus landed on the right element after the key press

## Gotchas

- **Keyboard events fire on the focused element.** Put `fs-trigger="keydown:<key>"` on the focusable element or make the container focusable with `tabindex="0"`.
- **Browser-reserved shortcuts.** `ctrl+n`, `ctrl+t`, `ctrl+w`, `cmd+q` and similar don't reach the page — the browser handles them first.
- **Repeat keys.** Holding a key fires multiple keydown events. Each starts a fresh assertion.

## See also

- [Triggers index](../triggers.md)
- [Placement rules](../triggers.md#placement-rules)
