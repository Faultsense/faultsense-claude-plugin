---
title: Conditional assertions
description: One element, multiple possible outcomes. Append a condition key to any assertion type to declare alternative success paths.
category: feature
name: conditional
since: 0.1.0
status: stable
---

# Conditional assertions

Some user actions have more than one valid outcome. Submitting a form can succeed, fail validation, or hit a server error. Clicking a search button can return results, an empty state, or a rate-limited message. Conditional assertions let you declare all the possibilities on a single element — the first outcome whose selector matches wins, the others are dismissed.

## Syntax

Append a `{condition-key}` suffix to any assertion type:

```
fs-assert-{type}-{condition-key}="<selector>"
```

```html
<button fs-assert="auth/login" fs-trigger="click"
  fs-assert-added-success=".dashboard"
  fs-assert-added-error=".error-msg">
  Log in
</button>
```

One of `added-success` or `added-error` wins. The other is dismissed and never reported.

## Condition key rules

- **Freeform lowercase alphanumeric with hyphens.** Examples: `success`, `error`, `empty`, `rate-limited`, `validation-error`, `server-error`.
- **Avoid assertion type names** as condition keys. `added`, `removed`, `updated`, `visible`, `hidden`, `loaded`, `oob`, `oob-fail`, `stable`, `emitted`, `after` are parsed as assertion types, not condition keys. Using them leads to confusing parser errors.
- **Keys are grouped by element + assertion type.** Multiple conditional siblings with the same type on the same element form a sibling group — first to match wins.

## Sibling groups — same type, different conditions

```html
<button fs-assert="search/execute" fs-trigger="click"
  fs-assert-added-results=".search-results"
  fs-assert-added-empty=".no-results"
  fs-assert-added-error=".search-error">
  Search
</button>
```

Three `added` conditionals, same element. First matching selector wins. The other two are dismissed.

## Cross-type conditional groups

To group conditionals **across** different types on the same element, add [`fs-assert-mutex`](mutex.md). Without mutex, cross-type conditionals (`added-success` + `removed-success`) resolve independently — each producing its own pass/fail signal.

See [mutex](mutex.md) for the full semantics.

## Full outcome coverage — conditionals + OOB

A single conditional branch can have multiple expectations via [OOB](oob.md). For example, a delete should remove the item AND show a success toast:

```html
<!-- Primary: conditional on the trigger -->
<button fs-assert="todos/remove-item" fs-trigger="click"
  fs-assert-mutex="each"
  fs-assert-removed-success=".todo-item"
  fs-assert-added-error=".error-msg">
  Delete
</button>

<!-- Secondary: OOB checks the toast appeared on successful delete -->
<div class="toast-container"
  fs-assert="todos/delete-toast"
  fs-assert-oob="todos/remove-item"
  fs-assert-visible=".success-toast">
</div>
```

## Dismissed vs failed

**Dismissed** means the assertion was retired without producing a signal — it's not a failure. A conditional's losing siblings are dismissed, not failed. This means:

- Dismissed assertions don't appear in the collector's failure metrics
- [`fs-assert-oob-fail`](oob.md) does NOT fire on dismissed conditionals — only on timeouts, GC sweep, SLA misses, and explicit failures

## Pairs well with

- [`fs-assert-mutex`](mutex.md) — required for cross-type conditional grouping
- [OOB assertions](oob.md) — multi-check validation per conditional branch
- [`fs-assert-timeout`](timeout.md) — network outcomes deserve an explicit SLA

## Gotchas

- **Using reserved words as condition keys.** `success` and `error` are fine. `added`, `removed`, and other assertion type names are not.
- **Expecting AND semantics within a sibling group.** First-match-wins is OR semantics. For AND, use [OOB](oob.md) on a secondary element or [`mutex="conditions"`](mutex.md) with same-key grouping.
- **Condition key collisions between types.** Without mutex, `added-success` and `removed-success` on the same element are independent — they don't race. Use [`fs-assert-mutex="each"`](mutex.md) or [`fs-assert-mutex="conditions"`](mutex.md) if you need them to compete.
- **React JSX boolean drop.** React drops bare boolean attributes. Always use explicit string values — `fs-assert-added-success=".dashboard"`, never `fs-assert-added-success`. See the [React boolean attribute trap](../frameworks/react.md#the-boolean-attribute-trap).

## See also

- [`mutex`](mutex.md) — cross-type grouping
- [OOB assertions](oob.md) — secondary checks
- [Patterns cookbook #6](../patterns.md#6-form-submit-with-conditional-outcomes)
- [Assertions index](../assertions.md)
