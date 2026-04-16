---
title: fs-assert-after
description: Resolves when all referenced parent assertion keys have already passed. Sequence validation.
category: assertion
name: after
since: 0.1.0
status: stable
---

# `fs-assert-after`

Resolves immediately at creation time — passes if every referenced parent assertion has already passed, fails otherwise. Use to validate that user actions happened in the correct order.

## Syntax

```
fs-assert-after="<key>"
fs-assert-after="<key1>,<key2>,..."
```

Comma-separated parent keys, AND semantics: every listed parent must have passed for `after` to pass.

## When to use it

- Multi-step wizards ("Step 3 requires Step 2 which requires Step 1")
- Checkout funnels ("Submit payment requires add-to-cart")
- Multi-click workflows that must happen in order
- Analytics funnel validation without network instrumentation

## Example

```html
<!-- Step 1 -->
<button fs-assert="checkout/add-to-cart" fs-trigger="click"
  fs-assert-added=".cart-item">Add to cart</button>

<!-- Step 2: must have added to cart first -->
<button fs-assert="checkout/submit-payment" fs-trigger="click"
  fs-assert-after="checkout/add-to-cart"
  fs-assert-visible=".confirmation">Pay</button>
```

If the user clicks Pay without adding to cart, `checkout/submit-payment/after` fails with a sequence violation — even if the `.confirmation` element still appears.

## Semantics

- **Immediate resolution.** `after` doesn't wait. At trigger time, it checks the stored "has passed" state of every parent key. If all passed, it passes. Otherwise it fails.
- **Independent data point.** `after` produces its own assertion event alongside any DOM assertions on the same element. A sequence violation with a healthy-looking UI shows up as a failed `after` assertion and a passing `visible` assertion — both signals are valuable.
- **Recovery on re-trigger.** A failed `after` can recover if the parent assertion later passes and the trigger fires again.
- **Chaining.** A → B → C works naturally: each `after` checks only its direct parent.

## Pairs well with

- [`fs-trigger="click"`](../triggers/click.md) — the canonical trigger
- [`fs-trigger="submit"`](../triggers/submit.md) — form-based wizards
- Any DOM assertion type on the same element — `after` runs alongside, not in place of

## Gotchas

- **Don't combine with [`fs-trigger="invariant"`](../triggers/invariant.md).** Invariants skip the immediate resolution path that `after` depends on — it would never be checked.
- **Parent keys must be stable across releases.** If you rename `checkout/add-to-cart` to `checkout/cart-add`, every `after` referencing the old key fails until you update them.
- **"Has passed" is session-scoped.** Parent state is tracked in memory, not in localStorage. Navigating to a new page via a hard reload resets the state. For cross-page sequences, use [`fs-assert-mpa="true"`](mpa.md) on both the parent and the `after` assertion.

## See also

- [Patterns cookbook #5](../patterns.md#5-multi-step-wizard-with-sequence-validation)
- [Assertions index](../assertions.md)
