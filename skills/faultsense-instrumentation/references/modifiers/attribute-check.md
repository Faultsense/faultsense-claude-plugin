---
title: Attribute checks (unreserved keys)
description: Any bracket key that isn't a reserved modifier is treated as an attribute check — regex full match against the element's attribute.
category: modifier
name: attribute-check
since: 0.1.0
status: stable
---

# Attribute checks (unreserved modifier keys)

Any bracket key that isn't one of the reserved modifiers ([`text-matches`](text-matches.md), [`value-matches`](value-matches.md), [`checked`](checked.md), [`disabled`](disabled.md), [`focused`](focused.md), [`focused-within`](focused-within.md), [`count`](count.md), [`classlist`](classlist.md)) is treated as an **attribute check**. The key is the attribute name, the value is a regex that **fully matches** (auto-anchored) the element's attribute value.

## Syntax

```
fs-assert-<type>="<selector>[<attribute>=<regex>]"
fs-assert-<type>="<selector>[<attribute>=<regex>][<attribute>=<regex>]..."
```

## Anchoring — full match (auto-anchored)

Attribute checks are auto-wrapped in `^(?:...)$`. The regex must match the **entire** attribute value, not a substring:

```
[data-state=active]        → matches data-state="active" exactly
[data-state=active|ready]  → matches "active" OR "ready" (alternation still full-matches)
[src=/img/new\.png]        → must match the entire src attribute
```

This is the opposite of [`text-matches`](text-matches.md) and [`value-matches`](value-matches.md), which are partial-match. The difference exists because text content is usually a long string where partial match is useful, while attribute values are usually exact tokens where full match prevents accidental substring hits.

## When to use it

- `aria-*` state — `aria-expanded=true`, `aria-selected=false`, `aria-hidden=false`
- `data-*` state machines — `data-state=open`, `data-phase=loading|ready`
- URL attributes — `src`, `href`
- Form attribute state — `type=password`, `required`

## Example — aria-expanded

```html
<button
  fs-assert="accordion/expand"
  fs-trigger="click"
  fs-assert-updated='[aria-expanded=true]'>
  Toggle
</button>
```

Self-referencing assertion — checks the button's own `aria-expanded` flips to `true`.

## Example — data-state machine

```html
<div class="wizard"
  fs-assert="wizard/enter-review"
  fs-trigger="click"
  fs-assert-updated='.wizard[data-phase=review]'>
</div>
```

Passes when `.wizard` has `data-phase="review"` exactly.

## Example — multiple attribute checks chained

```html
<img fs-assert="hero/replaced" fs-trigger="mount"
  fs-assert-loaded='img.hero[src=/img/new.png][width=1200][alt=New hero image]'>
```

All three attribute checks must pass. Any mismatch fails the modifier.

## Example — alternation

```html
<div
  fs-assert="onboarding/step-ready"
  fs-trigger="invariant"
  fs-assert-visible='[data-state=ready|complete]'>
</div>
```

Passes when `data-state` is exactly `ready` or exactly `complete`. The alternation is evaluated under full-match, so both sides must represent a complete attribute value.

## Semantics

- **Reads `element.getAttribute(name)`.** Missing attributes return `null`, which fails the regex.
- **Regex is auto-anchored** with `^(?:...)$` — no partial matches.
- **Multiple attribute checks chain with AND semantics.** All must pass.
- **Chained modifiers mix freely** — reserved and unreserved in any order.

## Pairs well with

- [`fs-assert-updated`](../assertions/updated.md) — the canonical pairing for attribute changes
- [`fs-assert-visible`](../assertions/visible.md) — verify attribute state on a visible element
- [Self-referencing selectors](../assertions/self-referencing.md) — check the instrumented element's own attributes
- [`classlist`](classlist.md) — when the state is a class, not an attribute

## Gotchas

- **Using CSS attribute selectors inside modifier values.** The bracket parser treats `[` as a modifier delimiter. `[data-id="123"] .btn[disabled=true]` gets misparsed. Use id or class selectors in the selector part, put attribute checks in modifier brackets.
- **Escaping slashes in the value.** Attribute values like URLs contain slashes, which are fine as-is — the regex engine doesn't care. But dots need escaping: `[src=/img/new\.png]`.
- **Partial-match expectations.** If you expect `[data-state=active]` to match `data-state="is-active"`, it doesn't — attribute checks are full-match. Use `[data-state=.*active.*]` or switch to a classlist-style check.
- **Dynamic attribute values in React JSX.** Remember to string-interpolate, not template-literal the whole attribute: `fs-assert-updated={`[aria-expanded=${isOpen}]`}`.

## See also

- [`text-matches`](text-matches.md) — for text content (partial match)
- [`classlist`](classlist.md) — when class presence matters
- [Modifiers index](../modifiers.md)
