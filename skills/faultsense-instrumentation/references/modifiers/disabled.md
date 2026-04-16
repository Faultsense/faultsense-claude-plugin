---
title: "[disabled=true|false]"
description: Check the element's disabled state — native .disabled property plus aria-disabled="true".
category: modifier
name: disabled
since: 0.1.0
status: stable
---

# `[disabled=true|false]`

Checks the element's disabled state. Reads both the native `.disabled` DOM property (`button`, `input`, `select`, `textarea`, etc.) AND `aria-disabled="true"` for ARIA-annotated non-native disables.

## Syntax

```
fs-assert-<type>="<selector>[disabled=true]"
fs-assert-<type>="<selector>[disabled=false]"
```

## When to use it

- Submit button becomes disabled while a request is in flight
- Form controls disable when a prerequisite isn't met
- Verify a control is re-enabled after an action completes
- ARIA-disabled custom components (role="button" divs, etc.)

## Example — disabled while submitting

```html
<button type="submit"
  fs-assert="signup/submit-disables"
  fs-trigger="click"
  fs-assert-updated="[disabled=true]">
  Create account
</button>
```

Passes when the button becomes disabled after the click (presumably because the submit handler marks it so while the request is in flight).

## Example — re-enabled after completion

```html
<div
  fs-assert="signup/submit-reenables"
  fs-assert-oob="signup/request-completes"
  fs-assert-visible="button[type=submit][disabled=false]">
</div>
```

OOB check that re-enablement happened.

## Semantics

The modifier passes if **either** is true:

1. `element.disabled === true` (native form control property), OR
2. `element.getAttribute('aria-disabled') === 'true'`

For `[disabled=false]`, the modifier passes if both are false.

This matches how screen readers and assistive tech treat disabled state — a custom button with `aria-disabled="true"` is functionally equivalent to a native `<button disabled>`.

## Pairs well with

- [`fs-assert-updated`](../assertions/updated.md) — catch the exact moment the state flips
- [`fs-assert-visible`](../assertions/visible.md) — verify the current disabled state
- [Self-referencing selectors](../assertions/self-referencing.md) — check the element itself

## Gotchas

- **Custom components without `aria-disabled`.** A `<div role="button" class="is-disabled">` with only visual styling isn't disabled per this modifier. Either add `aria-disabled="true"` (recommended for accessibility) or use a [`[classlist=is-disabled:true]`](classlist.md) check.
- **Pointer-events CSS.** `pointer-events: none` is not disabled state — it just blocks clicks. Use `aria-disabled` for the accessibility contract.
- **Readonly is not disabled.** A `readonly` text input is still focusable and submittable. `disabled=true` won't match a readonly input.

## See also

- [`checked`](checked.md) — for checkbox/radio state
- [`focused`](focused.md) — for focus state
- [Modifiers index](../modifiers.md)
