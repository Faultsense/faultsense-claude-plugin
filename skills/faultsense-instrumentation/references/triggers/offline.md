---
title: fs-trigger="offline"
description: Start an assertion when browser connectivity is lost.
category: trigger
name: offline
since: 0.1.0
status: stable
---

# `fs-trigger="offline"`

Fires when the browser loses connectivity (`window.offline` event). Use to verify the offline-mode UI and queued-mutation flow.

## Syntax

```html
<div
  fs-assert="network/offline-banner"
  fs-trigger="offline"
  fs-assert-visible=".offline-banner">
</div>
```

## When to use it

- Offline indicator or banner UI
- Switching to read-only mode on connectivity loss
- Queuing mutations in local storage when the network drops

## Example

```html
<div id="editor"
  fs-assert="editor/offline-autosave"
  fs-trigger="offline"
  fs-assert-visible=".autosave-queued-indicator">
</div>
```

## Pairs well with

- [`fs-assert-visible`](../assertions/visible.md) — offline banner appears
- [`fs-assert-added`](../assertions/added.md) — offline queue UI is lazily mounted
- [`fs-assert-updated`](../assertions/updated.md) — existing UI flips to offline state

## Gotchas

- **Browser-reported connectivity is unreliable** — the event reflects OS/browser heuristics, not actual reachability.
- **Fires on `window`.** Place the trigger on any element; the listener is attached at document level.

## See also

- [`online`](online.md) — the complementary trigger
- [Triggers index](../triggers.md)
