---
title: Self-referencing selectors
description: Omit the selector and provide only modifiers to check the instrumented element itself.
category: feature
name: self-referencing
since: 0.1.0
status: stable
---

# Self-referencing selectors

When the assertion target is the instrumented element itself, omit the selector and provide only [modifiers](../modifiers.md). The empty selector before `[` means "check this element."

## Syntax

```
fs-assert-<type>="[modifier=value][modifier=value]..."
```

## When to use it

- A counter element asserts its own updated text
- A form control asserts its own value or checked state
- An accordion asserts its own `aria-expanded` attribute
- Any case where the instrumented element is both the trigger target (or mount point) and the thing being checked

## Example — self-updating counter

```html
<div id="todo-count"
  fs-assert="todos/count-displayed"
  fs-trigger="mount"
  fs-assert-updated="[text-matches=\d+/\d+ remaining]">
  2/3 remaining
</div>
```

The `fs-assert-updated` attribute has no selector — the modifiers check the `#todo-count` element itself.

## Example — form control state

```html
<input type="checkbox"
  fs-assert="consent/checked"
  fs-trigger="change"
  fs-assert-updated="[checked=true]">
```

The assertion checks the checkbox that was clicked. No need to duplicate the selector.

## Example — OOB on self

```html
<div class="toast-container"
  fs-assert="global/toast-appeared"
  fs-assert-oob="any-parent-key"
  fs-assert-visible="[classlist=active:true]">
</div>
```

The OOB assertion checks the toast container itself, not a descendant.

## Semantics

- **Empty selector** — just the modifier brackets. The parser recognizes a leading `[` as "start of modifiers with no selector."
- **All modifiers apply to the element itself.** `text-matches` checks the element's text content, `classlist` checks its classes, attribute checks check its attributes.
- **Works on every assertion type.** `visible`, `hidden`, `updated`, `added` (rarely useful — the element already exists), `removed`, `stable`, `loaded`.

## Pairs well with

- [`[text-matches=...]`](../modifiers/text-matches.md) — the canonical self-referencing modifier
- [`[checked=true]`](../modifiers/checked.md) — form control state
- [`[value-matches=...]`](../modifiers/value-matches.md) — form control value
- [`[classlist=...]`](../modifiers/classlist.md) — class state
- [`[focused=true]`](../modifiers/focused.md) — focus state
- Attribute checks — `[aria-expanded=true]`, `[data-state=open]`

## Gotchas

- **Can't use `[count=N]` self-referencing.** Count of self is always 1 — the modifier requires an explicit selector to match a collection. See [count](../modifiers/count.md).
- **Can't mix self-referencing with a selector.** The empty-selector form is all-or-nothing. `div[text-matches=...]` targets any descendant `<div>`, not the element itself — self-referencing requires the selector to be fully empty.
- **Dynamic content on the same element.** If the element's content changes frequently, [`updated`](updated.md) with self-referencing can race. Prefer an explicit selector to a stable child when precision matters.

## See also

- [Modifiers](../modifiers.md) — index of every modifier
- [Assertions index](../assertions.md)
