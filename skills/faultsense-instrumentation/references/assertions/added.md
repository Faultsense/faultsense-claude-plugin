---
title: fs-assert-added
description: Resolves when a new element matching the selector appears in the DOM.
category: assertion
name: added
since: 0.1.0
status: stable
---

# `fs-assert-added`

Resolves when a new element matching the selector is inserted into the DOM after the trigger fires. Mutation-observed — captures the exact insertion moment via `MutationObserver`.

## Syntax

```
fs-assert-added="<selector>"
fs-assert-added="<selector>[modifier=value]..."
```

## When to use it

Use when the target element **doesn't exist yet** and will be created by the triggered action. The canonical choice for:

- Adding items to lists (todos, cart, notifications)
- Showing a new overlay, modal, or toast
- Rendering a success or error element that wasn't in the DOM before
- Appending search results

If the element already exists and its content changes, use [`updated`](updated.md) instead. This is the #1 instrumentation mistake.

## Example

```html
<button
  fs-assert="todos/add-item"
  fs-trigger="click"
  fs-assert-added=".todo-item">
  Add
</button>
```

Passes when a new `.todo-item` element is inserted into the DOM after the click.

## Pre-existing matches don't pass

If a matching element is already in the DOM at the moment the trigger fires, `added` does **not** pass on that. It waits for an actual insertion event. This is intentional — a pre-existing match isn't "added by this trigger" and reporting it as a pass would mask real failures.

See [mutation pattern PAT-01](../mutation-patterns.md#pat-01-pre-existing-target) for the regression lock that guarantees this behavior.

## Pairs well with

- [`fs-trigger="click"`](../triggers/click.md) — canonical user interaction
- [`fs-trigger="submit"`](../triggers/submit.md) — form submissions that add a confirmation element
- [`[count-min=1]`](../modifiers/count.md) — assert at least N new elements appeared
- [`[text-matches=...]`](../modifiers/text-matches.md) — verify the new element's content
- [Conditional assertions](conditional.md) — branch on `added-success` vs `added-error`
- [`fs-assert-mutex="each"`](mutex.md) — group with a failure branch
- [`fs-assert-oob`](oob.md) — fire side-effect checks when this passes

## Gotchas

- **Using `added` when `updated` is correct.** Class toggles, text changes, and attribute updates on existing elements are [`updated`](updated.md), not added. `fs-assert-added=".foo.complete"` will never match if `.foo` already exists and merely gains the `complete` class.
- **Using `added` on HTMX `hx-swap="morph:outerHTML"`.** Idiomorph patches the element in place; identity is preserved. The correct type is [`updated`](updated.md). See the [HTMX swap table](../frameworks/htmx.md).
- **Broad selectors in lists.** Under a standard (non-morph) `outerHTML` swap the old and new elements briefly coexist. Prefer specific ids over class selectors: `fs-assert-added="#todo-123[classlist=completed:true]"` over `fs-assert-added=".todo-item[classlist=completed:true]"`.
- **With OOB or invariant.** State types work with [OOB](oob.md) and [invariant](../triggers/invariant.md); `added` is a state type and is fine for both. `updated` / `loaded` are event types and miss mutations that already happened — prefer `added` / `removed` / `visible` / `hidden` for OOB and invariant.

## See also

- [`updated`](updated.md) — for in-place content changes
- [`removed`](removed.md) — the complementary type
- [Mutation pattern catalog](../mutation-patterns.md) — how the agent handles DOM shapes
- [Assertions index](../assertions.md)
