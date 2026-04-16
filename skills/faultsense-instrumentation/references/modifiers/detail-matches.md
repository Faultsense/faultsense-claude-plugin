---
title: "[detail-matches=...]"
description: Filter a CustomEvent by inspecting `event.detail`. Syntax is shared between `fs-trigger="event:..."` and `fs-assert-emitted`, but semantics differ — string equality on triggers, regex on assertions.
category: modifier
name: detail-matches
since: 0.1.0
status: stable
---

# `[detail-matches=<key>:<value>]`

Filters a `CustomEvent` by inspecting its `event.detail` object. The only modifier that pairs with the event surface — used with [`fs-trigger="event:<name>"`](../triggers/event.md) to decide whether a dispatch counts as the trigger, and with [`fs-assert-emitted`](../assertions/emitted.md) to decide whether a matching dispatch counts as a pass.

## Syntax

```
fs-trigger="event:<name>[detail-matches=<key>:<value>]"
fs-assert-emitted="<name>[detail-matches=<key>:<pattern>]"
```

Multiple keys are comma-separated inside a **single** bracket (all must match — AND semantics):

```
fs-trigger="event:cart-updated[detail-matches=action:add,source:user]"
```

Do **not** chain as `[detail-matches=a:1][detail-matches=b:2]` — the parser stores each bracket's modifier value under the same key, so the second bracket overwrites the first. The comma form above is the only AND.

## Semantics differ by site

The same syntax does two different things depending on where it's attached:

| Site | Behavior |
|---|---|
| `fs-trigger="event:..."` | **Shallow string equality.** Each `value` must match `String(event.detail[key])` exactly. No regex. |
| `fs-assert-emitted="..."` | **Regex match.** Each `value` is a regex that is tested against `String(event.detail[key])`. |

This is the modifier's biggest gotcha and the reason it has its own doc page. Copying a `detail-matches` expression between a trigger and an assertion will change its matching semantics.

```html
<!-- TRIGGER — string equality. Matches detail.action === "add" exactly. -->
<div fs-assert="cart/on-add"
  fs-trigger="event:cart-updated[detail-matches=action:add]"
  fs-assert-updated="#cart-count">
</div>

<!-- ASSERTION — regex. Matches detail.orderId against \d+. -->
<button fs-assert="checkout/pay" fs-trigger="click"
  fs-assert-emitted="payment:complete[detail-matches=orderId:\d+]">
  Pay Now
</button>
```

## When to use it

- Filter a shared event channel by action: `event:cart-updated[detail-matches=action:add]` vs. `action:remove`
- Validate a numeric or ID field's shape in `fs-assert-emitted`: `[detail-matches=orderId:\d+]`, `[detail-matches=total:\$\d+\.\d{2}]`
- Distinguish multiple variants of the same event name that differ only in their detail payload

## Missing keys — semantics differ

Both sides stringify the detail value with `String(...)` before comparing, but they diverge on what happens when the named `key` isn't present on `event.detail`:

- **Trigger:** the lookup produces `undefined`, which `String()` coerces to the literal string `"undefined"`. A matcher like `[detail-matches=x:undefined]` would technically match a missing `x` — in practice matches against missing keys fail because nobody asserts against the literal string `"undefined"`.
- **Assertion:** an explicit `!(key in detail)` check rejects missing keys before the regex runs. Missing keys always fail, regardless of the matcher's value.

If you care about missing-key handling being deterministic, use `fs-assert-emitted` rather than filtering at the trigger site.

## Primitive `event.detail`

When `event.detail` is a primitive (`detail: 5`, `detail: "ok"`) rather than an object, the matcher must have **exactly one** key-value pair. The key is ignored; only the value is compared against `String(detail)`:

- Trigger side: `String(detail) === value` (string equality).
- Assertion side: `new RegExp(value).test(String(detail))` (regex).

Primitives with more than one matcher always fail. If your app's events dispatch both primitive and object detail payloads on the same event name, prefer object detail consistently.

## Pairs well with

- [`fs-trigger="event:<name>"`](../triggers/event.md) — the filter-a-trigger use case
- [`fs-assert-emitted`](../assertions/emitted.md) — the validate-a-dispatched-event use case
- [Patterns cookbook #11](../patterns.md#11-custom-event-trigger--emitted-assertion)

## Gotchas

- **Confusing the two semantics.** String equality on triggers, regex on `emitted`. `[detail-matches=status:success]` is an exact-string check on a trigger; add a regex metacharacter (`.`, `\d`, etc.) and an `emitted` assertion will match but the trigger won't.
- **Repeated bracket chains are NOT additive.** `[detail-matches=a:1][detail-matches=b:2]` stores `"b:2"` — the second bracket wins. Use `[detail-matches=a:1,b:2]` for AND.
- **Only top-level keys.** `[detail-matches=user.id:123]` will look for a literal key named `"user.id"` on `event.detail`, which almost certainly doesn't exist. Flatten your detail or dispatch a separate event.
- **Number-to-string coercion.** `{ detail: { count: 5 } }` is checked as `"5"` (string). Fine for exact matches, but watch your regex if you assumed a numeric type.

## See also

- [`fs-trigger="event:<name>"`](../triggers/event.md)
- [`fs-assert-emitted`](../assertions/emitted.md)
- [Modifiers index](../modifiers.md)
