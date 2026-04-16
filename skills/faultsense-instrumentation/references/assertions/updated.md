---
title: fs-assert-updated
description: Resolves when the matched element or its subtree is mutated — text, attributes, or children change in place.
category: assertion
name: updated
since: 0.1.0
status: stable
---

# `fs-assert-updated`

Resolves when the matched element's text, attributes, or children mutate in place. Mutation-observed — the element identity is preserved; only its content changes.

## Syntax

```
fs-assert-updated="<selector>"
fs-assert-updated="<selector>[modifier=value]..."
```

## When to use it

Use when the target element **already exists** and the triggered action will modify it. The canonical choice for:

- Class toggles on an existing element (`.active`, `.completed`, `.expanded`)
- Text content changes on counters, status indicators, labels
- Attribute changes (`aria-expanded`, `data-state`, `src`)
- Idiomorph or morphdom in-place patches

If a new element is going to be created, use [`added`](added.md) instead.

## Example

```html
<button
  fs-assert="counter/increment"
  fs-trigger="click"
  fs-assert-updated='#counter[text-matches=\d+]'>
  Increment
</button>
<div id="counter">0</div>
```

Passes when `#counter`'s text changes to match the regex after the click.

## Mutation record shape

`updated` resolves from mutation records whose `target` matches the selector — either `attributes` / `characterData` records directly on the element, or `childList` records where the element is the parent. The resolver walks text-node mutations up to their parent element via `parentElement`, so a text-only change satisfies `updated` with [`text-matches`](../modifiers/text-matches.md) without having to target the text node directly (see [PAT-06](../mutation-patterns.md#pat-06-text-only-mutation)).

## Pairs well with

- [`fs-trigger="click"`](../triggers/click.md) — clicks that mutate an existing counter, label, or status
- [`fs-trigger="change"`](../triggers/change.md) — form controls flipping state
- [`[text-matches=...]`](../modifiers/text-matches.md) — verify new text
- [`[classlist=...]`](../modifiers/classlist.md) — verify class changes
- [`[data-state=...]`](../modifiers/attribute-check.md) — verify data-attr changes
- [Dynamic assertion values](../patterns.md#dynamic-assertion-values) — compute the expected next state in the template

## Gotchas

- **Using `updated` on HTMX `hx-swap="outerHTML"`.** outerHTML replaces the element wholesale. The new node lands in `addedElements`, not `updatedElements`, and `updated` will never resolve. Use [`added`](added.md). See the [HTMX swap strategy table](../frameworks/htmx.md#the-swap-strategy-table).
- **Using `updated` with OOB or invariant.** Both need a witnessed mutation. OOB assertions are created when a parent resolves — the mutation already happened. Invariants evaluate perpetually — they don't wait for a new mutation. Use [`visible`](visible.md), [`hidden`](hidden.md), [`added`](added.md), or [`removed`](removed.md) for those cases.
- **Broad selectors.** `updated` waits for a mutation on the first matching element. In lists with many similar items, prefer specific ids or data attributes to avoid matching the wrong element.

## See also

- [`added`](added.md) — for new elements
- [`stable`](stable.md) — the temporal inverse (passes when NOT updated)
- [Mutation pattern PAT-04](../mutation-patterns.md#pat-04-morphdom-preserved-identity) — morphdom in-place patches
- [Assertions index](../assertions.md)
