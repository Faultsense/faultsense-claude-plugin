---
title: "[classlist=class:bool,...]"
description: Assert that one or more classes are present or absent on the element.
category: modifier
name: classlist
since: 0.1.0
status: stable
---

# `[classlist=<class>:<bool>,...]`

Presence/absence check on the element's class list. Format: comma-separated `name:true|false` pairs.

## Syntax

```
fs-assert-<type>="<selector>[classlist=active:true]"
fs-assert-<type>="<selector>[classlist=active:true,hidden:false]"
fs-assert-<type>="<selector>[classlist=loading:false,ready:true,error:false]"
```

## When to use it

- Class-based state machines — `.active`, `.selected`, `.completed`, `.expanded`
- Transition states — `.loading` should flip to `.ready`
- Error indicators — `.error` should not be present after recovery
- Anything that uses CSS classes as state flags

## Example — toggle completion

```html
<input type="checkbox"
  fs-assert="todos/toggle-complete"
  fs-trigger="change"
  fs-assert-updated=".todo-item[classlist=completed:true]">
```

Passes when `.todo-item` has the `completed` class after the change event.

## Example — transition state

```html
<button fs-assert="data/load" fs-trigger="click"
  fs-assert-updated=".data-panel[classlist=loading:false,ready:true]">
  Reload
</button>
```

Passes when `.data-panel` is **not** loading AND **is** ready — both conditions must hold.

## Semantics

- **Checks `element.classList.contains(name)`.** True means the class is present; false means it's not.
- **All entries must pass.** A single failing entry means the whole modifier fails.
- **Case-sensitive class names** (matches DOM behavior).
- **`true` and `false`** are case-insensitive as values. Other values log a warning.

## Pairs well with

- [`fs-assert-updated`](../assertions/updated.md) — the canonical pairing for class toggles
- [`fs-assert-visible`](../assertions/visible.md) — verify visible class state
- [Dynamic assertion values](../patterns.md#dynamic-assertion-values) — compute `!currentState` for toggles

## Gotchas

- **Dynamic expected state with React/Vue/Svelte.** For bidirectional toggles, compute the expected next state in the template: `[classlist=completed:${!todo.completed}]`. Hardcoding `completed:true` breaks as soon as the user untoggles.
- **CSS modules and scoped classes.** Frameworks that generate hashed class names (CSS modules, Vue scoped styles) make `classlist` checks fragile. Prefer stable data attributes (`data-state=open`) for state signals.
- **Checking for a class that doesn't exist.** `[classlist=neverpresent:false]` passes trivially — the class is indeed not present. Be explicit about which classes matter and include all relevant ones.

## See also

- [`fs-assert-updated`](../assertions/updated.md) — the canonical pairing
- [Attribute checks](attribute-check.md) — alternative for data-attribute state
- [Modifiers index](../modifiers.md)
