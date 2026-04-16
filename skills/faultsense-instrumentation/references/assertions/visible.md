---
title: fs-assert-visible
description: Resolves when the matched element exists and has layout dimensions.
category: assertion
name: visible
since: 0.1.0
status: stable
---

# `fs-assert-visible`

Resolves when an element matching the selector exists in the DOM AND has non-zero layout dimensions. Query-based — runs a point-in-time `querySelector` and layout check, not a mutation watch.

## Syntax

```
fs-assert-visible="<selector>"
fs-assert-visible="<selector>[modifier=value]..."
```

## When to use it

- The target element exists in the DOM but is currently hidden (modal overlay, tab panel, collapsed drawer) and the action should reveal it
- Verifying a mount-triggered element has layout
- [Invariant](../triggers/invariant.md)-triggered "should always be visible" contracts

If the element doesn't exist yet and will be created, use [`added`](added.md). If you need to catch the exact moment of a state change (rather than the current state), use [`updated`](updated.md).

## Example

```html
<button
  fs-assert="tabs/switch-settings"
  fs-trigger="click"
  fs-assert-visible=".tab-content[data-tab='settings']">
  Settings
</button>
```

Passes when the settings panel has non-zero layout dimensions after the click.

## Resolves immediately on current state

`visible` passes as soon as the query returns a match AND the match has dimensions. If the current state already satisfies the assertion at trigger time, it passes immediately — unlike [`added`](added.md), which waits for a mutation.

This makes `visible` a good fit for OOB and invariant: both evaluate against the current DOM and don't need to witness a new mutation.

## What "has layout dimensions" means

The agent checks `offsetWidth > 0 && offsetHeight > 0`. This catches:

- `display: none` (fails)
- `visibility: hidden` (fails — dimensions are 0)
- Element detached from DOM (fails — query doesn't match)
- Zero-size element via CSS (fails)

It does **not** check:

- `opacity: 0` (still has dimensions — passes)
- Off-screen positioning (still has dimensions — passes)
- Occluded by another element (still has dimensions — passes)

For precise visibility semantics, layer a [`[classlist=visible:true]`](../modifiers/classlist.md) or [`[data-state=open]`](../modifiers/attribute-check.md) check on top.

## Pairs well with

- [`fs-trigger="click"`](../triggers/click.md) — reveal flows
- [`fs-trigger="hover"`](../triggers/hover.md) — tooltips and preview overlays
- [`fs-trigger="invariant"`](../triggers/invariant.md) — the canonical invariant partner
- [`fs-trigger="mount"`](../triggers/mount.md) — verify just-mounted UI has layout
- [OOB assertions](oob.md) — state-type choice for side-effect checks
- [`[text-matches=...]`](../modifiers/text-matches.md) — verify visible text on the same element (self-referencing)
- [`[count=N]`](../modifiers/count.md) — verify exact cardinality of visible elements

## Gotchas

- **Short-lived elements.** If the element is created AND destroyed within the timeout window, `visible` can miss it — the query only runs at creation time and in response to mutation batches. Use [`added`](added.md) for elements with short lifetimes.
- **Using `visible` when `added` is correct.** If the element is conditionally rendered (not in the DOM until the action), `visible` will pass as soon as the query matches, but the timing may be fragile. `added` is more precise.
- **CSS animations and transitions.** An element that animates from `opacity: 0` to `opacity: 1` is "visible" the whole time — it had dimensions throughout the transition. Use a [`[classlist=...]`](../modifiers/classlist.md) or [`[aria-hidden=false]`](../modifiers/attribute-check.md) modifier if you need the animation endpoint.

## See also

- [`hidden`](hidden.md) — the complementary type
- [`added`](added.md) — for elements that don't yet exist
- [Assertions index](../assertions.md)
