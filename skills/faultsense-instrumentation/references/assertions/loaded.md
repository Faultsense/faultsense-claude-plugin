---
title: fs-assert-loaded
description: Resolves when a media element finishes loading.
category: assertion
name: loaded
since: 0.1.0
status: stable
---

# `fs-assert-loaded`

Resolves when a media element (`img`, `video`, `iframe`) matching the selector finishes loading. Combines the native `load` event with a `.complete` check for the cached case.

## Syntax

```
fs-assert-loaded="<selector>"
```

## When to use it

- Hero images that must actually fetch for the page to be "ready"
- Video or iframe embeds where a broken resource is a visible failure
- Media-heavy UI where load state is a correctness signal

## Example

```html
<img src="/hero.webp" class="hero"
  fs-assert="home/hero-loaded"
  fs-trigger="mount"
  fs-assert-loaded="img.hero">
```

Passes when the image finishes loading — either via the `load` event or because it was already cached at registration time.

## Cached resources

The agent checks `img.complete && img.naturalWidth > 0` at registration time, so cached images pass immediately even if the native `load` event fired before the assertion existed. Uncached resources wait for the `load` event.

## Pairs well with

- [`fs-trigger="load"`](../triggers/load.md) — the natural event trigger
- [`fs-trigger="mount"`](../triggers/mount.md) — verify just-mounted media loads
- [Conditional assertions](conditional.md) — pair with an `error` branch via [`fs-trigger="error"`](../triggers/error.md) on a separate element

## Gotchas

- **Not compatible with OOB or invariant.** `loaded` is an event type — OOB and invariant assertions need state types that evaluate against the current DOM. Use [`visible`](visible.md) or [`added`](added.md) for those contexts.
- **iframe load fires on every navigation inside.** Scope the assertion key appropriately.
- **Lazy-loaded images.** If an `<img loading="lazy">` hasn't scrolled into view, `load` never fires. Either disable lazy loading for the asserted image or trigger a scroll before the assertion.

## See also

- [`fs-trigger="load"`](../triggers/load.md)
- [`fs-trigger="error"`](../triggers/error.md)
- [Assertions index](../assertions.md)
