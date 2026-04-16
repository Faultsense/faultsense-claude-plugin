---
title: fs-trigger="focus"
description: Start an assertion when the element receives focus.
category: trigger
name: focus
since: 0.1.0
status: stable
aliases: [focusin]
---

# `fs-trigger="focus"`

Fires when the element gains focus. Implemented as an alias for `focusin`, which bubbles to document-level capture listeners (unlike the non-bubbling `focus` event).

## Syntax

```html
<input type="text"
  fs-assert="search/reveal-suggestions"
  fs-trigger="focus"
  fs-assert-visible=".suggestions-panel">
```

## When to use it

- Reveal-on-focus UI (suggestions dropdown, tooltip, help text)
- First-interaction tracking on a form field
- Onboarding or tour triggers tied to a specific input

## Example

```html
<input type="search"
  fs-assert="header/open-search"
  fs-trigger="focus"
  fs-assert-visible=".search-dropdown">
```

## Pairs well with

- [`fs-assert-visible`](../assertions/visible.md) — a helper panel becomes visible
- [`fs-assert-added`](../assertions/added.md) — a helper panel is lazily mounted on focus
- [`[focused=true]`](../modifiers/focused.md) — verify focus landed on the expected element

## Gotchas

- **Programmatic focus fires the event too.** Calling `.focus()` in JS triggers the assertion. Use a different assertion key or guard against unintended triggers in your handler.
- **Tab-through focus races.** Rapidly tabbing through fields may fire and cancel focus triggers quickly. Increase [`fs-assert-timeout`](../assertions/timeout.md) if race conditions matter.

## See also

- [`blur`](blur.md) — the complementary trigger
- [`[focused=true]`](../modifiers/focused.md) — focus-state modifier
- [Triggers index](../triggers.md)
