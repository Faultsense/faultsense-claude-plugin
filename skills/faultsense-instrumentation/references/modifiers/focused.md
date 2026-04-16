---
title: "[focused=true|false]"
description: Check whether the element currently has focus — document.activeElement equality.
category: modifier
name: focused
since: 0.1.0
status: stable
---

# `[focused=true|false]`

Checks whether the element currently holds document focus. Implemented as `document.activeElement === element`.

## Syntax

```
fs-assert-<type>="<selector>[focused=true]"
fs-assert-<type>="<selector>[focused=false]"
```

## When to use it

- Reveal-and-focus flows where an input should auto-focus after a modal opens
- Keyboard navigation verification — "after Tab, focus should land on the next input"
- Confirming focus moves correctly after a toast dismiss or modal close

## Example — auto-focus on modal open

```html
<button
  fs-assert="search/open-modal"
  fs-trigger="click"
  fs-assert-visible=".search-modal .search-input[focused=true]">
  Search
</button>
```

Passes when the search modal is visible AND the search input inside it has focus.

## Semantics

- **Point-in-time check.** The modifier reads `document.activeElement` at the moment of evaluation. It does not listen to focus events.
- **A single focus at a time.** `focused=true` can only be true for one element per document.
- **IFrames have their own active elements.** Focus inside an iframe means the iframe is the active element from the parent document's perspective.

## Pairs well with

- [`fs-assert-visible`](../assertions/visible.md) — verify visibility AND focus in a single assertion
- [`fs-trigger="focus"`](../triggers/focus.md) — explicitly assert focus landed
- [Self-referencing selectors](../assertions/self-referencing.md) — check the instrumented element itself
- [`[focused-within=true]`](focused-within.md) — for "focus is anywhere inside this subtree"

## Gotchas

- **Focus races with assertion timing.** In frameworks that insert an element and focus it in separate microtasks, the assertion may run between the two. Use [`fs-assert-updated`](../assertions/updated.md) or a small [timeout](../assertions/timeout.md) to give the focus call a chance to land.
- **`.focus()` called on a non-focusable element is a no-op.** Divs and spans without `tabindex` can't hold focus. The assertion will silently fail.
- **Browser quirks on dynamic insertion.** `autofocus` on elements inserted after initial page load is unreliable. Some browsers honor it, some don't. Call `.focus()` explicitly if the contract matters. See [HTMX framework notes](../frameworks/htmx.md) for the swapped-input pattern.

## See also

- [`focused-within`](focused-within.md) — for subtree focus
- [`fs-trigger="focus"`](../triggers/focus.md) — the focus trigger
- [Modifiers index](../modifiers.md)
