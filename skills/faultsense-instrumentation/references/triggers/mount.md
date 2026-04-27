---
title: fs-trigger="mount"
description: Start an assertion when the element is added to the DOM.
category: trigger
name: mount
since: 0.1.0
status: stable
---

# `fs-trigger="mount"`

Fires once when the element is inserted into the DOM. Use for page-load checks, lazy-loaded section validation, and any "the feature should be present when this element appears" contract.

## Syntax

```html
<div
  fs-assert="layout/hero-loaded"
  fs-trigger="mount"
  fs-assert-visible=".hero-image">
</div>
```

## When to use it

- Page-load validation ("the nav should be visible when it mounts")
- Lazy-loaded section checks ("when this island hydrates, assert its contents")
- One-shot lifecycle checks that [invariant](invariant.md) would make too chatty

**Mount vs invariant.** `mount` fires once, at insertion. [`invariant`](invariant.md) fires perpetually. For "should always be true" use invariant. For "should be true when the element appears" use mount.

## Example

```html
<section
  fs-assert="checkout/payment-form-loaded"
  fs-trigger="mount"
  fs-assert-visible=".card-number-input">
  <!-- lazy-loaded payment form -->
</section>
```

## Pairs well with

- [`fs-assert-visible`](../assertions/visible.md) — verify the section rendered with the right elements
- [`fs-assert-added`](../assertions/added.md) — verify sub-elements appear after mount
- [`fs-assert-after`](../assertions/after.md) — gate on a prior assertion before this mount counts

## Gotchas

- **Pre-existing elements.** If the element is already in the DOM when the agent initializes (i.e., on `DOMContentLoaded`), mount fires at init time — treat it as first-render validation.
- **Hydration.** An SSR-rendered element that the client hydrates is NOT re-mounted. Its attributes change, but identity is preserved. Mount does not re-fire on hydration. Use [`invariant`](invariant.md) for "assert across hydration."
- **Re-mounts (React StrictMode, key changes).** Frameworks that unmount and remount the same logical element fire mount multiple times. Each fire starts a fresh assertion.

## See also

- [`unmount`](unmount.md) — the complementary trigger
- [`invariant`](invariant.md) — perpetual alternative
- [Triggers index](../triggers.md)
