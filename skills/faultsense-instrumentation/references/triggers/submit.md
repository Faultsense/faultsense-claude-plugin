---
title: fs-trigger="submit"
description: Start an assertion when a form is submitted.
category: trigger
name: submit
since: 0.1.0
status: stable
---

# `fs-trigger="submit"`

Fires on the native `submit` event — place it on the `<form>` element. A capture-phase listener means the agent sees the submit even if a framework or library calls `preventDefault()` later (HTMX, for instance, suppresses the native submit after its listeners run).

## Syntax

```html
<form
  fs-assert="contact/submit-form"
  fs-trigger="submit"
  fs-assert-visible=".success-message">
  <input name="email" type="email" />
  <button type="submit">Send</button>
</form>
```

## When to use it

- Form submissions — the canonical choice over `click` on the submit button
- Any `<form>` whose submit semantics matter for correctness

Either `fs-trigger="submit"` on the form OR `fs-trigger="click"` on the submit button works. Pick the one that reads clearer for your markup. Don't use both on the same assertion key — you'll get two fires per submit.

## Example

```html
<form
  fs-assert="signup/create-account"
  fs-trigger="submit"
  fs-assert-mutex="each"
  fs-assert-visible-success=".welcome-dashboard"
  fs-assert-added-error=".form-errors">
  …
</form>
```

## Pairs well with

- [Conditional assertions](../assertions/conditional.md) — `success` / `validation-error` / `server-error` branches
- [`fs-assert-mutex="each"`](../assertions/mutex.md) — group cross-type conditionals
- [`fs-assert-timeout`](../assertions/timeout.md) — network operations deserve an explicit SLA
- [`fs-assert-mpa="true"`](../assertions/mpa.md) — persist the assertion across a full page navigation

## Gotchas

- **Enter-to-submit bypasses the submit button.** If you put the trigger on the button instead of the form, pressing Enter inside a text field submits the form without firing the click handler. Prefer `submit` on the form to cover both paths.
- **AJAX submits via `preventDefault`.** Capture listeners see the event first, so the assertion still fires even when JS prevents the real submission.

## See also

- [`click`](click.md) — alternative placement on the submit button
- [Patterns cookbook #6](../patterns.md#6-form-submit-with-conditional-outcomes)
- [Triggers index](../triggers.md)
