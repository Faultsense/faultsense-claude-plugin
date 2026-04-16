---
title: fs-assert-route
description: Resolves when the current URL matches a pattern — pathname, query params, and hash fragment are each regex-matched.
category: assertion
name: route
since: 0.1.0
status: stable
---

# `fs-assert-route`

Resolves when `window.location` matches the given URL pattern. Pathname, query parameters, and hash fragment are each validated independently as anchored regex. Use to assert that a user action navigates to the expected route — after a client-side router transition, an HTMX boost, a form submit with `PushState`, or a full-page navigation that was preserved via MPA mode.

## Syntax

```
fs-assert-route="<pathname-regex>"
fs-assert-route="<pathname-regex>?<param>=<value-regex>&<param>=<value-regex>..."
fs-assert-route="<pathname-regex>#<hash-regex>"
fs-assert-route="<pathname-regex>?<param>=<value-regex>#<hash-regex>"
```

Everything looks like a URL but each part is a regex — **pathname**, each **query-param value**, and the **hash** (without `#`). Plain URLs without regex metacharacters are valid patterns that match literally.

## Anchoring — full match

Every regex part is auto-wrapped in `^...$`. The pathname must match the whole `window.location.pathname`, each declared query param must match its value end-to-end, and the hash (if declared) must match the whole `window.location.hash.slice(1)`.

```
/dashboard              → matches exactly /dashboard, not /dashboard/settings
/dashboard/.+           → matches /dashboard/anything, not /dashboard itself
/users/\d+              → matches /users/123, not /users/me
```

## Query params — order-independent, declared-only

The pattern's query params are matched against the current URL via `URLSearchParams`, so parameter order doesn't matter. Undeclared params in the URL are ignored — the pattern only asserts on the params you list.

```
fs-assert-route="/callback?code=.*&state=\w+"
```

Passes for `/callback?state=xyz&code=abc123` (order reversed) and `/callback?code=abc123&state=xyz&debug=1` (extra param ignored). Fails for `/callback?code=abc123` (missing `state`) and `/callback?code=abc123&state=xyz!` (`state` doesn't match `\w+`).

## Hash fragments

The leading `#` is stripped before matching. Your pattern matches the raw fragment text.

```
fs-assert-route="/docs#section-\d+"
```

Passes for `/docs#section-4`. Fails for `/docs` (no hash at all) and `/docs#section-intro` (doesn't match `\d+`).

## When to use it

- Client-side router transitions (React Router, Vue Router, TanStack Router, Next.js, SvelteKit)
- HTMX `hx-push-url="true"` or `hx-boost` transitions
- Login redirects after form submit
- OAuth callback handling — `/auth/callback?code=...&state=...`
- Any assertion that says "the user ended up at the right place"

## Example

```html
<button fs-assert="auth/login" fs-trigger="click"
  fs-assert-route-success="/dashboard"
  fs-assert-added-error=".login-error">
  Sign in
</button>
```

Conditional branches: on success the URL becomes `/dashboard`; on failure, an error element appears.

## Pairs well with

- [`fs-trigger="click"`](../triggers/click.md) — navigation triggered by clicking a link or button
- [`fs-trigger="submit"`](../triggers/submit.md) — form submits that push a new URL
- [`fs-trigger="event:<name>"`](../triggers/event.md) — route changes driven by custom events
- [`fs-assert-mpa`](mpa.md) — persist the assertion across a full-page navigation and resolve on the next page load
- [Conditional assertions](conditional.md) — `route-success` vs `added-error` branches

## Gotchas

- **Regex metacharacters in pathnames.** `/dashboard.html` matches `/dashboard-html`, `/dashboardxhtml`, and more because `.` is a regex wildcard. Escape with `/dashboard\.html` if the literal dot matters.
- **Query params are declared-only, not exhaustive.** If the user navigates to `/page?a=1&b=2` and the pattern is `/page?a=1`, the assertion passes — extra params are ignored. If the strict absence of extra params matters, include them all in the pattern.
- **Hash regex runs on the raw fragment without `#`.** `fs-assert-route="/page#foo"` checks that `window.location.hash` is `#foo` — do NOT include the `#` in your pattern.
- **Invalid regex in any part.** The agent validates the pattern before creating the assertion and logs a `console.warn` if any part fails to compile. A failed pattern never becomes an assertion, so it never resolves — watch the console when authoring.
- **Not mutation-observed.** `route` resolves via `window.location` reads, not MutationObserver. It's re-evaluated after each navigation-like event. If you're asserting on DOM state caused by the route change, pair `route` with a DOM-type assertion using conditional branches or OOB.

## See also

- [`fs-assert-mpa`](mpa.md) — for route assertions that span a full-page navigation
- [Conditional assertions](conditional.md) — route-success vs added-error branches
- [Assertions index](../assertions.md)
