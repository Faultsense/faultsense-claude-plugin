---
title: Out-of-band (OOB) assertions
description: Fire secondary assertions on side-effect elements when a parent assertion passes or fails.
category: feature
name: oob
since: 0.1.0
status: stable
---

# Out-of-band (OOB) assertions

OOB assertions fire when a referenced parent assertion passes or fails. Use them for side-effect elements — count labels, totals, toasts, error indicators — that should be verified after a primary action resolves, without prop-drilling state through the component tree.

## Syntax

OOB replaces `fs-trigger`:

```html
<div
  fs-assert="feature/secondary-check"
  fs-assert-oob="primary-key"
  fs-assert-visible=".target">
</div>
```

Two attributes:

| Attribute | Fires when |
|---|---|
| `fs-assert-oob="<key1>,<key2>"` | Any listed parent **passes** |
| `fs-assert-oob-fail="<key1>,<key2>"` | Any listed parent **fails** (timeout, GC, SLA miss). Dismissed conditionals do NOT trigger `oob-fail`. |

Both can coexist on the same element as independent triggers. Multiple parent keys are comma-separated (OR — fires if any parent matches).

## When to use it

- Count labels in a separate component that reflect list state after add/remove
- Toast containers that verify a confirmation message appeared after an action
- Error indicators tested against a parent's failure path
- Stability checks that begin after a primary mutation settles
- Cross-component side-effect validation without dependency injection

## Example — count label

```html
<!-- Primary actions in various components -->
<button fs-assert="todos/add-item" fs-trigger="click"
  fs-assert-added=".todo-item">Add</button>

<button fs-assert="todos/remove-item" fs-trigger="click"
  fs-assert-removed=".todo-item">Delete</button>

<!-- OOB count label (separate component, no prop drilling) -->
<div id="todo-count"
  fs-assert="todos/count-updated"
  fs-assert-oob="todos/add-item,todos/remove-item"
  fs-assert-visible="[text-matches=\d+/\d+ remaining]">
  2/3 remaining
</div>
```

When either parent passes, the OOB assertion evaluates its `visible` check against the current DOM and reports pass/fail.

## Example — multi-check per conditional branch

The primary element has conditional branches; OOB verifies a secondary check only on the success branch:

```html
<button fs-assert="todos/remove-item" fs-trigger="click"
  fs-assert-mutex="each"
  fs-assert-removed-success=".todo-item"
  fs-assert-added-error=".error-msg">
  Delete
</button>

<div class="toast-container"
  fs-assert="todos/delete-toast"
  fs-assert-oob="todos/remove-item"
  fs-assert-visible=".success-toast">
</div>
```

The toast OOB fires when `todos/remove-item` passes (whichever condition wins). If you only want it on the success branch, scope your trigger with [`fs-assert-mutex="conditions"`](mutex.md) or use `fs-assert-oob-fail` on a companion element to verify the error path.

## Example — `oob-fail`

```html
<button fs-assert="todos/add-item" fs-trigger="click"
  fs-assert-added=".todo-item"
  fs-assert-timeout="3000">
  Add
</button>

<!-- Verify an error indicator is visible when the add fails -->
<div id="error-check"
  fs-assert="todos/add-error-check"
  fs-assert-oob-fail="todos/add-item"
  fs-assert-visible=".error-indicator">
</div>
```

When `todos/add-item` times out or fails, the OOB fail fires and checks that `.error-indicator` is visible.

## Semantics

- **No chaining.** OOB passing does not trigger further OOB. One hop.
- **Self-referencing supported.** Omit the selector to check the OOB element itself.
- **Multi-parent.** Comma-separated keys mean "fire when any parent matches."
- **State types only.** Use [`visible`](visible.md), [`hidden`](hidden.md), [`added`](added.md), [`removed`](removed.md), [`stable`](stable.md). Event types ([`updated`](updated.md), [`loaded`](loaded.md), [`emitted`](emitted.md)) require witnessing a mutation or event — but by the time the OOB assertion is created, the parent already resolved and the mutation already happened. Event types will miss it.
- **Dismissed parents don't fire `oob-fail`.** Dismissed assertions (losing conditional siblings) are not failures. Only real failures — timeout, GC, SLA — trigger `oob-fail`.

## Pairs well with

- [Conditional assertions](conditional.md) — OOB is the canonical multi-check for a conditional winner
- [`fs-assert-mutex`](mutex.md) — group conditional siblings before OOB fires
- [`fs-assert-stable`](stable.md) — the classic "no flicker after parent resolves" pattern

## Gotchas

- **Using event types with OOB.** [`updated`](updated.md), [`loaded`](loaded.md), and [`emitted`](emitted.md) will miss the mutation because the parent already passed. Use state types.
- **OOB for per-item assertions in lists.** OOB broadcasts to ALL elements matching `fs-assert-oob="key"`. Use OOB for singleton side-effects (count displays, toasts). For per-item state, use multiple assertion types on the trigger element itself.
- **HTMX `hx-swap-oob` is a different concept.** Same name, no interaction — one is Faultsense routing, the other is HTMX DOM delivery. See [HTMX framework notes](../frameworks/htmx.md).
- **OOB requires the element to exist when the parent resolves.** If the OOB element is itself conditionally rendered and not yet in the DOM, its assertion isn't created. Use a long-lived container.

## See also

- [Patterns cookbook #9](../patterns.md#9-oob-side-effect-validation-count-label)
- [`fs-assert-stable`](stable.md) — the canonical OOB partner
- [Assertions index](../assertions.md)
