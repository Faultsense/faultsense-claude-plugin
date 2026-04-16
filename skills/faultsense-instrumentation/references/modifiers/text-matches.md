---
title: "[text-matches=...]"
description: Match the element's text content against a regex — partial match, unanchored.
category: modifier
name: text-matches
since: 0.1.0
status: stable
---

# `[text-matches=<pattern>]`

Partial-match regex check against the element's text content. The most common modifier — use it whenever you care about the rendered text, not just the presence of an element.

## Syntax

```
fs-assert-<type>="<selector>[text-matches=<regex>]"
```

## Anchoring — partial match

`text-matches` is **unanchored**. The pattern can match anywhere in the text content:

```
[text-matches=\d+]          → matches "3 items"
[text-matches=Order #\d+]   → matches "Your Order #1234 is ready"
[text-matches=^Exact$]      → anchored explicitly, matches only "Exact"
```

If you need the whole string to match, anchor with `^` and `$` explicitly.

## When to use it

- Counter values: `[text-matches=\d+]`
- Status strings: `[text-matches=(Pending|Complete|Failed)]`
- Order confirmation numbers: `[text-matches=Order #\d+]`
- Per-item state: `[text-matches=2 of \d+ remaining]`
- Any dynamic text content that needs to match a pattern, not a literal

## Example

```html
<button fs-assert="counter/increment" fs-trigger="click"
  fs-assert-updated='#counter[text-matches=Count: [1-9]\d*]'>
  Increment
</button>
<div id="counter">Count: 0</div>
```

Passes when `#counter`'s text content contains "Count: " followed by a positive integer (1 through 9 followed by any number of digits — explicitly not `Count: 0`).

## Self-referencing

Omit the selector to check the instrumented element itself:

```html
<div id="todo-count"
  fs-assert="todos/count-displayed"
  fs-trigger="mount"
  fs-assert-updated="[text-matches=\d+/\d+ remaining]">
```

## Pairs well with

- [`fs-assert-updated`](../assertions/updated.md) — the canonical pairing for text changes
- [`fs-assert-added`](../assertions/added.md) — verify new element's rendered text
- [`fs-assert-visible`](../assertions/visible.md) — verify visible text
- Attribute checks — chain with `[data-state=...]` or similar

## Gotchas

- **Using a literal dynamic value.** `[text-matches=42]` matches the literal string "42", which works until the value changes. Use a regex pattern: `[text-matches=\d+]`.
- **Forgetting this is partial match.** `[text-matches=Success]` matches "Login Success", "Success!", "Unsuccessful" (contains "success" if case-insensitive), and more. Anchor explicitly if you need exactness.
- **Regex escape in HTML.** Backslashes in attribute values don't need double-escaping in plain HTML, but do in JSX/JS string templates. `fs-assert-updated="[text-matches=\d+]"` in HTML, `fs-assert-updated={"[text-matches=\\d+]"}` in a JS template string.
- **`text-matches` reads `textContent`, not `innerText`.** It includes text inside hidden elements (`display:none` descendants' text is still in `textContent`). If that matters, target a more specific child.

## See also

- [`value-matches`](value-matches.md) — for form control `.value` (also partial-match regex)
- [Attribute checks](attribute-check.md) — full-match regex for attributes
- [Modifiers index](../modifiers.md)
