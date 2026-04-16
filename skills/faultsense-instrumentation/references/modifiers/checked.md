---
title: "[checked=true|false]"
description: Check a checkbox or radio button's .checked DOM property.
category: modifier
name: checked
since: 0.1.0
status: stable
---

# `[checked=true|false]`

Reads the element's `.checked` DOM property. Works on checkboxes, radio buttons, and any element that exposes a `.checked` property.

## Syntax

```
fs-assert-<type>="<selector>[checked=true]"
fs-assert-<type>="<selector>[checked=false]"
```

## When to use it

- Toggle flows where the state flip is the correctness signal
- Consent checkboxes
- Settings toggles
- Radio group selection

## Example — consent checkbox

```html
<input type="checkbox"
  fs-assert="signup/accept-terms"
  fs-trigger="change"
  fs-assert-updated="[checked=true]">
```

Passes when the checkbox is checked after the change event.

## Example — toggle with dynamic expected state

For bidirectional toggles, compute the expected next state in your template:

```jsx
// React
<input type="checkbox"
  fs-assert="todos/toggle-complete"
  fs-trigger="change"
  fs-assert-updated={`[checked=${!todo.completed}]`}
  checked={todo.completed} />
```

When the checkbox is currently unchecked (`completed === false`), the assertion expects `checked=true`. When currently checked, it expects `checked=false`.

## Semantics

- **Reads `.checked`**, not the `checked=""` HTML attribute. The HTML attribute is the initial state; `.checked` is the live state.
- **Case-insensitive** on `true`/`false`.
- **Unrecognized values log a warning** and the modifier is treated as no-op.

## Pairs well with

- [`fs-trigger="change"`](../triggers/change.md) — the canonical pairing
- [`fs-trigger="click"`](../triggers/click.md) — label-wrapped checkboxes
- [Self-referencing selectors](../assertions/self-referencing.md) — check the checkbox itself
- [Dynamic assertion values](../patterns.md#dynamic-assertion-values) — flip-expected-state pattern

## Gotchas

- **Using `value-matches` on a checkbox.** `checkbox.value` returns the static `value` attribute, not the checked state. Use `checked`, not `value-matches`, for checkbox/radio state.
- **Label-wrapped checkboxes fire click on both.** Clicking a `<label>` that wraps an `<input>` fires click on both. Place the trigger on the element the user is meant to interact with.
- **Indeterminate state.** HTML allows `checkbox.indeterminate = true` — a third state. `checked` only reads true/false, not indeterminate. The agent does not check `.indeterminate`.

## See also

- [`disabled`](disabled.md) — the disabled-state counterpart
- [`focused`](focused.md) — for focus state
- [Modifiers index](../modifiers.md)
