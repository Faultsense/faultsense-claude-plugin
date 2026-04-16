---
title: "[value-matches=...]"
description: Match a form control's .value property against a regex — partial match, unanchored.
category: modifier
name: value-matches
since: 0.1.0
status: stable
---

# `[value-matches=<pattern>]`

Partial-match regex check against the form control's live `.value` DOM property. Works on `input`, `textarea`, and `select`. Reads the live value, not the HTML `value=""` attribute.

## Syntax

```
fs-assert-<type>="<selector>[value-matches=<regex>]"
```

## When to use it

- Search and filter inputs whose value drives a dependent query
- Form controls whose committed value matters for correctness
- Any case where "what the user typed" is a correctness signal

## Example — search input

```html
<input type="search" class="search-input"
  fs-assert="search/submit"
  fs-trigger="change"
  fs-assert-updated=".search-input[value-matches=.+]">
```

Passes when the search input's `.value` is one or more characters long.

## `.value` vs the `value` attribute

| Read | Contains |
|---|---|
| `.value` DOM property | The live current value (what the user typed, or what JS set) |
| `value` HTML attribute | The initial/default value (what the server rendered) |

`value-matches` reads `.value`. This is almost always what you want — you care about the state right now, not the initial state.

The practical consequence: typing in an input does not produce an `attributes` mutation on `value`, because `.value` is a DOM property, not an attribute. If you pair `value-matches` with [`fs-assert-updated`](../assertions/updated.md), the mutation observer won't see anything. Use an event trigger like [`change`](../triggers/change.md), [`input`](../triggers/input.md), or [`blur`](../triggers/blur.md).

## Pairs well with

- [`fs-trigger="change"`](../triggers/change.md) — commit-semantics value check
- [`fs-trigger="input"`](../triggers/input.md) — per-keystroke value check
- [`fs-trigger="blur"`](../triggers/blur.md) — value check on focus loss
- [Conditional assertions](../assertions/conditional.md) — branch on valid vs invalid input patterns

## Gotchas

- **Pairing with `fs-assert-updated`.** `.value` isn't an attribute. `MutationObserver` doesn't fire on `.value` changes. Use an event trigger (`change` / `input` / `blur`) and check the value there.
- **Checkboxes and radios.** `checkbox.value` returns the static `value=""` attribute, not the checked state. Use [`[checked=true]`](checked.md) for checkbox/radio state, not `value-matches`.
- **Select menus.** `select.value` returns the selected option's `value` attribute. `value-matches` works, but for multi-select you probably want a [count](count.md) check on `option[selected]` descendants instead.
- **Partial match.** Just like [`text-matches`](text-matches.md), `value-matches` is unanchored. Use `^` / `$` for exactness.

## See also

- [`text-matches`](text-matches.md) — the text-content counterpart
- [`checked`](checked.md) — for checkbox/radio state
- [Modifiers index](../modifiers.md)
