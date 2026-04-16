---
title: fs-trigger="unmount"
description: Start an assertion when the element is removed from the DOM.
category: trigger
name: unmount
since: 0.1.0
status: stable
---

# `fs-trigger="unmount"`

Fires when the element is removed from the DOM. Use for cleanup validation — verify that removing a feature also removes its side effects.

## Syntax

```html
<div class="modal"
  fs-assert="modal/unmount-cleanup"
  fs-trigger="unmount"
  fs-assert-removed=".backdrop">
</div>
```

## When to use it

- Cleanup validation when a section is torn down (modals, drawers, lazy-loaded pages)
- "When X disappears, Y should also disappear" contracts

## Example

```html
<div id="editor-panel"
  fs-assert="editor/unmount-saves-draft"
  fs-trigger="unmount"
  fs-assert-added=".draft-saved-toast">
</div>
```

When the editor panel is removed, assert that a "draft saved" toast appears elsewhere.

## Pairs well with

- [`fs-assert-removed`](../assertions/removed.md) — verify a related element is gone
- [`fs-assert-added`](../assertions/added.md) — verify a cleanup notification appeared
- [`fs-assert-after`](../assertions/after.md) — sequence check against a prior close action

## Gotchas

- **Timing.** The assertion is created at the moment of removal. If the side effect happens in the same microtask, use [OOB](../assertions/oob.md) with `fs-assert-oob-fail` or a different trigger — unmount is a commit moment, not a window.
- **Full page unloads don't fire unmount.** The agent's unload grace-period model takes over (see [timeout](../assertions/timeout.md)).

## See also

- [`mount`](mount.md) — the complementary trigger
- [OOB assertions](../assertions/oob.md) — for side-effect validation
- [Triggers index](../triggers.md)
