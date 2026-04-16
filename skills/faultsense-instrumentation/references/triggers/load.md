---
title: fs-trigger="load"
description: Start an assertion when a media resource finishes loading.
category: trigger
name: load
since: 0.1.0
status: stable
---

# `fs-trigger="load"`

Fires on the native `load` event of media elements. Use for image, video, or iframe load validation.

## Syntax

```html
<img src="/hero.webp"
  fs-assert="home/hero-loaded"
  fs-trigger="load"
  fs-assert-loaded="img.hero">
```

## When to use it

- Hero image load validation
- Video or iframe loaded-successfully contracts
- Anything where "did the resource actually fetch?" is a visible correctness signal

## Example

```html
<video src="/demo.mp4" controls
  fs-assert="demo/video-loaded"
  fs-trigger="load"
  fs-assert-loaded="video">
</video>
```

## Pairs well with

- [`fs-assert-loaded`](../assertions/loaded.md) — the natural partner for this trigger
- [Conditional assertions](../assertions/conditional.md) — pair with [`error`](error.md) for success/failure branches

## Gotchas

- **Cached resources.** Browsers fire `load` synchronously for cached resources. If the image is in cache when the agent initializes, the event may fire before the assertion is registered. Prefer [`fs-assert-loaded`](../assertions/loaded.md) which also checks `.complete` for the cached case.
- **iframe `load` is noisy.** Every navigation inside the iframe fires another `load`. Scope the assertion key appropriately.

## See also

- [`error`](error.md) — the failure counterpart
- [`fs-assert-loaded`](../assertions/loaded.md)
- [Triggers index](../triggers.md)
