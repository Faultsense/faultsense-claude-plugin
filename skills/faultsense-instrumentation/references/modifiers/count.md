---
title: "[count=N], [count-min=N], [count-max=N]"
description: Cardinality check on the selector's match set — exact, lower bound, or upper bound.
category: modifier
name: count
since: 0.1.0
status: stable
aliases: [count-min, count-max]
---

# `[count=N]`, `[count-min=N]`, `[count-max=N]`

Cardinality modifiers. They check how many elements match the selector via `querySelectorAll(selector).length`.

## Syntax

```
fs-assert-<type>="<selector>[count=N]"
fs-assert-<type>="<selector>[count-min=N]"
fs-assert-<type>="<selector>[count-max=N]"
```

| Modifier | Passes when |
|---|---|
| `[count=N]` | Exactly N elements match |
| `[count-min=N]` | At least N elements match |
| `[count-max=N]` | At most N elements match |

Chain bounds to express ranges:

```
fs-assert-<type>="<selector>[count-min=3][count-max=10]"
```

## When to use it

- Search result verification — "at least one result appeared"
- List cardinality — "exactly five items remain"
- Pagination checks — "no more than 20 results on one page"
- Toast queue limits — "at most three toasts visible"

## Example — search has results

```html
<button fs-assert="search/execute" fs-trigger="click"
  fs-assert-added=".result-card[count-min=1]">
  Search
</button>
```

Passes when at least one `.result-card` appears after the click.

## Example — exact count after delete

```html
<div
  fs-assert="todos/item-count"
  fs-assert-oob="todos/remove-item"
  fs-assert-visible=".todo-item[count=4]">
</div>
```

OOB check that exactly four `.todo-item` elements remain after a delete.

## Example — range bounds

```html
<div
  fs-assert="feed/page-size"
  fs-trigger="mount"
  fs-assert-visible=".feed-item[count-min=10][count-max=20]">
</div>
```

Passes when between 10 and 20 feed items are present.

## Semantics

- **Evaluated via `querySelectorAll(selector).length`.** Uses the top-level `document.querySelectorAll`, not a subtree query.
- **Counts the total match set**, not per-mutation delta. `[count-min=1]` on [`added`](../assertions/added.md) passes the moment at least one element matches, regardless of how it got there.
- **Integer values.** Non-integer or negative values log a warning and the modifier becomes no-op.

## Pairs well with

- [`fs-assert-added`](../assertions/added.md) — "at least N new elements after the action"
- [`fs-assert-visible`](../assertions/visible.md) — "exactly N elements visible right now"
- [`fs-assert-removed`](../assertions/removed.md) — "N or fewer remaining after remove"
- [OOB](../assertions/oob.md) — cardinality audits on side-effect containers

## Gotchas

- **`count` with self-referencing.** Count of self is always 1. The modifier requires an explicit selector that can match a collection. `fs-assert-visible="[count=3]"` is nonsensical and the modifier is ignored.
- **Global queries.** `querySelectorAll` runs on the document, not the subtree. `fs-assert-visible=".item[count=3]"` counts every `.item` on the page, not just children of the instrumented element. Use more specific selectors if you need scoped counts.
- **Counting invisible elements.** `count` doesn't care about layout — hidden and offscreen elements count as matches. Combine with [`fs-assert-visible`](../assertions/visible.md) for visible cardinality.

## See also

- [`classlist`](classlist.md) — class-state checks
- [Modifiers index](../modifiers.md)
