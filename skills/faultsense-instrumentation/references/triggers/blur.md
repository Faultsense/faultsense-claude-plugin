---
title: fs-trigger="blur"
description: Start an assertion when the element loses focus.
category: trigger
name: blur
since: 0.1.0
status: stable
---

# `fs-trigger="blur"`

Fires on the native `blur` event (via `focusout` capture so it reaches the instrumented element). Use for validation and "user finished with this field" semantics.

## Syntax

```html
<input type="email"
  fs-assert="signup/validate-email"
  fs-trigger="blur"
  fs-assert-updated=".email-field[classlist=valid:true]">
```

## When to use it

- Field-level validation that runs after the user leaves the field
- Save-on-blur semantics (inline edit, autosave)
- "Did the user actually finish?" gates on form sections

## Example

```html
<input type="text" name="username"
  fs-assert="signup/check-username"
  fs-trigger="blur"
  fs-assert-visible-available=".username-available"
  fs-assert-visible-taken=".username-taken">
```

## Pairs well with

- [`fs-assert-updated`](../assertions/updated.md) — field marks itself valid/invalid
- [`fs-assert-visible`](../assertions/visible.md) — a validation message appears
- [Conditional assertions](../assertions/conditional.md) — branch on `available` vs `taken`
- [`[value-matches=...]`](../modifiers/value-matches.md) — verify the committed value via self-referencing

## Gotchas

- **Blur on text inputs also triggers `change`.** If you have both a `blur` trigger and a `change` trigger on the same field, both fire when the user leaves. Pick one.
- **Click-on-submit patterns lose focus.** Clicking a submit button blurs the input — the blur assertion fires alongside the submit.

## See also

- [`focus`](focus.md) — the complementary trigger
- [`change`](change.md) — commit-semantics alternative
- [Triggers index](../triggers.md)
