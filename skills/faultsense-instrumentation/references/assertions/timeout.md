---
title: fs-assert-timeout
description: Opt-in per-assertion SLA timeout — fails if the expected outcome doesn't happen in time.
category: feature
name: timeout
since: 0.1.0
status: stable
---

# `fs-assert-timeout`

Opt-in per-assertion SLA. The assertion fails if it hasn't resolved within the given number of milliseconds. There is **no default timeout** — assertions without `fs-assert-timeout` wait until the DOM resolves them or the GC sweep cleans them up.

## Syntax

```
fs-assert-timeout="<ms>"
```

```html
<button fs-assert="checkout/submit" fs-trigger="click"
  fs-assert-added=".confirmation"
  fs-assert-timeout="2000">
  Submit
</button>
```

## When to use it

- Network operations with a visible SLA (payment, signup, search)
- Performance contracts ("this must resolve within 500ms")
- Stability windows for [`stable`](stable.md) assertions that need a tight bound
- Any flow where a slow response is itself a failure signal

Don't add timeouts to every assertion. The absence of a timeout is not a bug — most assertions resolve naturally from DOM mutations and get cleaned up by the GC sweep if they don't.

## The lifecycle (no timeout)

Without `fs-assert-timeout`:

1. **Trigger fires.** Assertion is created, pending.
2. **DOM resolves it.** The mutation observer or query check satisfies the assertion, which reports pass.
3. **OR: GC sweep.** After `config.gcInterval` (default 5 seconds) the background sweep cleans up stale assertions. They do NOT report as failed — they're swept silently.
4. **OR: page unload.** On `pagehide`, assertions older than `config.unloadGracePeriod` (default 2 seconds) report as failed via `sendBeacon`. Fresh assertions are dropped.

## The lifecycle (with timeout)

With `fs-assert-timeout="2000"`:

1. **Trigger fires.** Assertion is created, pending.
2. **Timer starts.** A 2-second timer begins.
3. **DOM resolves it first** → pass.
4. **OR: timer fires first** → fail with a timeout reason. Reports to the collector as a performance SLA miss.

The timeout is competitive with the natural resolution path. Whichever happens first wins.

## Re-trigger tracking

When a user re-triggers an action while its assertion is still pending (e.g., clicking a laggy button twice), the new trigger timestamp is recorded in an `attempts[]` array on the assertion and reported in the final payload. This is how rage-click analysis works on the collector side.

## Pairs well with

- [Conditional assertions](conditional.md) — network-backed outcomes deserve an SLA
- [`fs-assert-mutex="each"`](mutex.md) — race success vs error under a bounded window
- [`fs-assert-stable`](stable.md) — explicit stability windows
- [`fs-trigger="click"`](../triggers/click.md) / [`fs-trigger="submit"`](../triggers/submit.md) — user actions with network latency

## Gotchas

- **Not compatible with [`invariant`](../triggers/invariant.md).** Invariants are perpetual and ignore `fs-assert-timeout`.
- **Not compatible with [`after`](after.md).** `after` resolves immediately — there's nothing to time out.
- **Too-tight timeouts generate noise.** Start loose (e.g., 5 seconds), measure real-world pass rates, then tighten if the feature's p95 is much lower. A timeout that fails for 20% of real users is worse than no timeout at all.

## Config-level knobs

The agent exposes two timing knobs at init time (see [configuration](../configuration.md)):

| Option | Default | Controls |
|---|---|---|
| `gcInterval` | 5000 ms | How often stale assertions without explicit timeouts are swept |
| `unloadGracePeriod` | 2000 ms | How long an assertion has to mature before page-unload counts it as a failure |

## See also

- [Configuration](../configuration.md#gcinterval)
- [`fs-assert-stable`](stable.md) — the canonical explicit-window partner
- [Assertions index](../assertions.md)
