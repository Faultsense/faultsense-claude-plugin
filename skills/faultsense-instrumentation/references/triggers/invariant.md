---
title: fs-trigger="invariant"
description: Continuously monitor a condition that should always hold — only failures and recoveries are reported.
category: trigger
name: invariant
since: 0.1.0
status: stable
---

# `fs-trigger="invariant"`

Continuously monitors a condition for the lifetime of the page. Unlike user-triggered assertions, invariants stay **pending** as long as the condition holds — only failures (violations) and recoveries (pass after failure) are reported to the collector.

## Syntax

```html
<nav
  fs-assert="layout/nav-visible"
  fs-trigger="invariant"
  fs-assert-visible="#main-nav">
</nav>
```

## Semantics

- **Quiet when healthy.** While the condition holds, no collector traffic.
- **Loud when broken.** A violation fires a failure report immediately.
- **Recoveries are reported.** A failed invariant that later passes again sends a recovery signal.
- **Auto-passed on unload.** On page close, any still-pending invariant is sent as the "all clear" signal.
- **No timeout.** Invariants are perpetual — they live for the page lifetime and ignore [`fs-assert-timeout`](../assertions/timeout.md).

## When to use it

- Page-level contracts ("the nav should always be visible", "no error banner should ever appear")
- Continuous layout validation across route changes in an SPA
- Guardrails around legal / compliance UI that should always be present
- Cross-cutting invariants (no toast stack overflow, no stuck loading spinners)

## Example

```html
<!-- Nav must always be visible -->
<nav
  fs-assert="layout/nav-visible"
  fs-trigger="invariant"
  fs-assert-visible=".main-nav">
</nav>

<!-- Error banner must never appear -->
<div
  fs-assert="layout/no-error-banner"
  fs-trigger="invariant"
  fs-assert-hidden=".global-error-banner">
</div>

<!-- Immutable legal notice -->
<div id="legal-notice"
  fs-assert="layout/legal-stable"
  fs-trigger="invariant"
  fs-assert-stable="#legal-notice">
</div>
```

## Pairs well with

- [`fs-assert-visible`](../assertions/visible.md) — the primary pairing
- [`fs-assert-hidden`](../assertions/hidden.md) — for "should never appear" contracts
- [`fs-assert-stable`](../assertions/stable.md) — "should never be mutated" contracts

## Gotchas

- **Don't combine with [`fs-assert-after`](../assertions/after.md).** Invariants skip the immediate resolution path that `after` depends on — it would never be checked. Combining the two logs a warning.
- **Event-based types are discouraged.** [`updated`](../assertions/updated.md), [`loaded`](../assertions/loaded.md), and [`emitted`](../assertions/emitted.md) require witnessing a mutation or event, but invariants evaluate perpetually against current state. The agent allows it but warns. Use state types (`visible`, `hidden`, `added`, `removed`, `stable`).
- **Conditional siblings and MPA are not supported** on invariant-triggered assertions.

## See also

- [`mount`](mount.md) — the one-shot alternative
- [Assertions index](../assertions.md)
- [Patterns cookbook #10](../patterns.md#10-invariant-continuous-monitoring)
