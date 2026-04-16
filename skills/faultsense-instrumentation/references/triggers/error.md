---
title: fs-trigger="error"
description: Start an assertion when a media resource fails to load.
category: trigger
name: error
since: 0.1.0
status: stable
---

# `fs-trigger="error"`

Fires on the native `error` event of a media element — broken image, failed video load, iframe navigation error. Use for visible-failure assertions on resources you depend on.

## Syntax

```html
<img src="/hero.webp"
  fs-assert="home/hero-fallback-shown"
  fs-trigger="error"
  fs-assert-visible=".hero-fallback">
```

## When to use it

- Resources where a broken fetch would be a visible failure (hero, avatar, product image)
- Verifying fallback UI kicks in when a media resource fails
- Dead-link detection for iframes or embedded players

## Example

```html
<img src="/user-avatar.png"
  fs-assert="profile/avatar-broken-shows-initials"
  fs-trigger="error"
  fs-assert-visible=".avatar-initials">
```

## Pairs well with

- [`fs-assert-visible`](../assertions/visible.md) — fallback element becomes visible on error
- [`fs-assert-added`](../assertions/added.md) — fallback element is added when the media 404s
- [Conditional assertions](../assertions/conditional.md) — pair with [`load`](load.md) for success/failure branches

## Gotchas

- **Silent CORS failures.** Some network errors fire `error`, others (especially CORS) fire with no useful detail. The trigger fires in both cases, but the observable failure mode differs.
- **Synthetic monitoring noise.** Bot traffic and link checkers may trigger this constantly. Scope assertion keys to user-facing paths.

## See also

- [`load`](load.md) — the success counterpart
- [Triggers index](../triggers.md)
