---
title: fs-trigger="online"
description: Start an assertion when browser connectivity is restored.
category: trigger
name: online
since: 0.1.0
status: stable
---

# `fs-trigger="online"`

Fires when the browser regains connectivity (`window.online` event). Use to verify reconnection UI and offline-mode recovery paths.

## Syntax

```html
<div
  fs-assert="network/reconnect-sync"
  fs-trigger="online"
  fs-assert-visible=".sync-in-progress">
</div>
```

## When to use it

- Verifying that offline-queued mutations flush on reconnect
- Reconnect toast or indicator UI
- Syncing presence, cart state, draft content when the network returns

## Example

```html
<div id="app-root"
  fs-assert="network/reconnect-banner"
  fs-trigger="online"
  fs-assert-removed=".offline-banner">
</div>
```

When the browser comes back online, assert the offline banner is gone.

## Pairs well with

- [`fs-assert-removed`](../assertions/removed.md) — offline indicator disappears
- [`fs-assert-visible`](../assertions/visible.md) — reconnect toast appears
- [`fs-assert-added`](../assertions/added.md) — a sync-progress indicator is created
- [`fs-assert-after`](../assertions/after.md) — sequence against the earlier offline event

## Gotchas

- **Browser-reported connectivity is unreliable.** The event reflects what the OS/browser thinks, which may differ from actual reachability. Treat online/offline signals as indicators, not ground truth.
- **Fires on `window`, not your element.** The trigger works by attaching a document-level listener; place it on any element.

## See also

- [`offline`](offline.md) — the complementary trigger
- [Triggers index](../triggers.md)
