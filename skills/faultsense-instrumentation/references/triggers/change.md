---
title: fs-trigger="change"
description: Start an assertion when a form control's value commits — after blur for text inputs, on every toggle for checkboxes.
category: trigger
name: change
since: 0.1.0
status: stable
---

# `fs-trigger="change"`

Fires on the native `change` event, which varies subtly by control type:

- **Text inputs** and textareas: `change` fires when the control loses focus AND its value has been modified since it was focused. Use [`input`](input.md) if you want every keystroke.
- **Checkboxes and radios**: fires every time the checked state flips.
- **Select menus**: fires every time a new option is selected.

## Syntax

```html
<input type="checkbox"
  fs-assert="todos/toggle-complete"
  fs-trigger="change"
  fs-assert-updated=".todo-item[classlist=completed:true]">
```

## When to use it

- Checkbox toggles, radio choices, select-menu changes — immediate feedback
- Final-value validation on a text input (use [`blur`](blur.md) if you want the focus event instead)

## Example

```html
<select
  fs-assert="profile/change-plan"
  fs-trigger="change"
  fs-assert-updated="#plan-summary[text-matches=Pro|Enterprise]">
  <option>Free</option>
  <option>Pro</option>
  <option>Enterprise</option>
</select>
```

## Pairs well with

- [`fs-assert-updated`](../assertions/updated.md) — the control's effect updates something in the page
- [`fs-assert-visible`](../assertions/visible.md) — a dependent section appears after the change
- [`[checked=true]`](../modifiers/checked.md) — verify the new checked state on the same element via self-referencing
- [`[value-matches=...]`](../modifiers/value-matches.md) — verify the new value via self-referencing (form controls)
- [Dynamic assertion values](../patterns.md#dynamic-assertion-values) — for bidirectional toggles, compute the expected next state in the template

## Gotchas

- **Text inputs don't fire `change` on every keystroke.** Use [`input`](input.md) for per-keystroke evaluation.
- **The toggle-expected-next-state pattern.** On a toggle, the assertion runs BEFORE the state flip completes visually in some frameworks. Compute `!currentState` in the attribute value so the assertion targets the post-toggle state.

## See also

- [`input`](input.md) — per-keystroke alternative
- [`blur`](blur.md) — trigger on focus loss regardless of value change
- [Triggers index](../triggers.md)
