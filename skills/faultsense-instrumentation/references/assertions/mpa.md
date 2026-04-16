---
title: fs-assert-mpa
description: Persist an assertion across a full page navigation so it resolves on the next page load.
category: feature
name: mpa
since: 0.1.0
status: stable
---

# `fs-assert-mpa`

Multi-page assertion. Marks an assertion to be persisted to `localStorage` on the current page and resolved on the next page load. Use for flows that straddle a hard navigation — form POSTs that redirect, legacy flows inside an SPA, anywhere a real `pagehide` + new `DOMContentLoaded` happens.

## Syntax

```
fs-assert-mpa="true"
```

## When to use it

- A form submit triggers a real page reload and the success message appears on the next page
- A legacy flow inside an otherwise-SPA app does a hard navigation
- An auth handoff between origins (where the new origin has the agent installed)

For SPA-style client-side navigation (React Router, Vue Router, Turbo Drive), you do NOT need MPA mode — the agent session is long-lived across client-side route changes. A regular DOM assertion will wait for the new view to render and resolve naturally.

## Example

```html
<button
  fs-assert="mpa-form/submit"
  fs-trigger="click"
  fs-assert-mpa="true"
  fs-assert-visible=".success-message">
  Submit
</button>
```

On click, the assertion is serialized to `localStorage`. After the page reload, the agent reads it back and resolves it against the new page's DOM.

## How it works

1. **On trigger fire:** the assertion is created normally AND written to `localStorage` with its current pending state.
2. **On `pagehide`:** the agent's unload handler flushes pending assertions (see [timeout](timeout.md)). MPA-marked assertions are left in storage, not failed.
3. **On the next `DOMContentLoaded`:** the agent reads stored MPA assertions and re-creates them in the new page's assertion manager.
4. **Resolution:** resolves against the new page's DOM via the normal paths — mutation observer for `added` / `removed` / `updated`, query for `visible` / `hidden`.

## Pairs well with

- [`fs-assert-visible`](visible.md) — verify a success message rendered on the next page
- [`fs-assert-added`](added.md) — verify a lazy-loaded element appears
- [`fs-trigger="click"`](../triggers/click.md) — form submissions that redirect
- [`fs-trigger="submit"`](../triggers/submit.md) — same

## Gotchas

- **Not compatible with [`emitted`](emitted.md).** MPA assertions persist to localStorage and resolve on the next page load — where the event will never re-fire. The agent warns and ignores `fs-assert-mpa="true"` on emitted assertions.
- **Not compatible with [`invariant`](../triggers/invariant.md).** Invariants are perpetual for a single page lifetime.
- **Not compatible with [`hx-boost`](../frameworks/htmx.md).** hx-boost never fires a real unload, so MPA assertions under boost are written to storage but never reloaded — they orphan. Under hx-boost, the agent session is long-lived, so you don't need MPA: a regular DOM assertion resolves naturally across the virtual nav.
- **`localStorage` quota.** MPA assertions are tiny but they share the same quota as your app. Clean up is automatic on resolution or timeout.
- **Cross-origin navigation.** MPA survives a same-origin redirect. It does not survive navigation to a different origin — localStorage is origin-scoped.

## See also

- [`timeout`](timeout.md) — how unload handling works
- [HTMX hx-boost](../frameworks/htmx.md) — why MPA doesn't apply under boost
- [Patterns cookbook #7](../patterns.md#7-mpa-navigation)
- [Assertions index](../assertions.md)
