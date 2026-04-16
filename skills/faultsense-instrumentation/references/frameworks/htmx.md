---
title: HTMX
description: Instrumenting HTMX apps with fs-* attributes — swap strategies, OOB swaps, HX-Trigger, error responses, focus management.
category: framework
name: htmx
since: 0.1.0
status: stable
---

# HTMX

HTMX ships server-rendered HTML fragments and swaps them into the DOM on user actions. `fs-*` attributes are authored in your server templates (EJS, ERB, Jinja, Handlebars, Nunjucks, etc.) and arrive in the DOM as plain HTML. No framework glue required — but several HTMX-specific patterns are worth knowing because they shape which assertion type is correct.

## Install

Load Faultsense before HTMX in your layout's `<head>`:

```html
<head>
  <script src="/faultsense-agent.min.js" defer></script>
  <script src="/htmx.min.js" defer></script>
  <script>
    document.addEventListener('DOMContentLoaded', () => {
      Faultsense.init({
        releaseLabel: '2.4.1',
        collectorURL: '/events',
        apiKey: 'fs_secret_...',
      });
    });
  </script>
</head>
```

The agent's wait-for-pass resolver handles HTMX's transient swap states (`htmx-swapping`, `htmx-added`, `htmx-settling`) automatically. No special-casing needed in your modifier checks.

## Template example

```ejs
<!-- views/todos/index.ejs -->
<button
  hx-post="/todos"
  hx-target="#todo-list"
  hx-swap="beforeend"
  fs-assert="todos/add-item"
  fs-trigger="click"
  fs-assert-added=".todo-item">
  Add
</button>

<ul id="todo-list">
  <% todos.forEach(function (todo) { %>
    <%- include('partials/todo-item', { todo: todo }) %>
  <% }) %>
</ul>
```

The `fs-*` attributes live directly in the server template. HTMX handles the swap; Faultsense observes the DOM mutation and resolves the assertion.

## The swap-strategy table

**`added` vs `updated` depends on the swap strategy, not the markup.** This is the #1 HTMX instrumentation mistake.

| Swap | DOM effect | Correct type |
|---|---|---|
| `hx-swap="outerHTML"` (no morph) | Old element removed, new element inserted | [`fs-assert-added`](../assertions/added.md) |
| `hx-swap="innerHTML"` | Parent keeps identity, children replaced | [`fs-assert-updated`](../assertions/updated.md) on parent, or [`fs-assert-added`](../assertions/added.md) on new children |
| `hx-swap="morph:outerHTML"` (idiomorph) | Target element is patched **in place** — same DOM node, attributes/class/children mutate | [`fs-assert-updated`](../assertions/updated.md) |
| `hx-swap="morph:innerHTML"` | Parent and children patched in place | [`fs-assert-updated`](../assertions/updated.md) |
| `hx-swap="beforeend"` / `"afterbegin"` | New child appended, existing siblings untouched | [`fs-assert-added`](../assertions/added.md) |

The common trap: a button with `hx-swap="morph:outerHTML"` toggling a row between view mode (`.todo-item`) and edit mode (`.todo-item-edit`) looks like "a new edit row appeared" — but with idiomorph, it's the same `<div id="todo-1">` getting its class and children patched. Use [`fs-assert-updated=".todo-item-edit"`](../assertions/updated.md), not `fs-assert-added`.

## 1. Use `HX-Location`, not `HX-Redirect`, for SPA navigation

`HX-Redirect` triggers a hard page reload — the agent re-initializes from scratch and pending assertions are lost. `HX-Location` does a client-side fetch + pushState, preserving the agent session.

```js
// GOOD — preserves the agent session
res.set('HX-Location', '/dashboard').status(204).end();

// BAD — hard reload loses pending assertions
res.set('HX-Redirect', '/dashboard').status(204).end();
```

## 2. Error responses don't swap by default

HTMX drops 4xx/5xx response bodies and fires `htmx:responseError`. If you declare [`fs-assert-added-error=".error-msg"`](../assertions/conditional.md) on a trigger, the expected `.error-msg` will never appear because the error fragment never enters the DOM.

Three options:

1. **Return 200 with an error fragment** and use `HX-Retarget` / `HX-Reswap` headers to route it to a dedicated error slot. Simplest — no extension required.
2. **Install the `response-targets` extension** and use `hx-target-error="#error-slot"` or `hx-target-5xx="#error-slot"` on the trigger.
3. **Reclassify status codes** via `htmx.config.responseHandling` so HTMX swaps error statuses by default.

```js
// Option 1 — server returns 200 + error fragment, retargeted
res.set('HX-Retarget', '#add-error-slot');
res.set('HX-Reswap', 'innerHTML');
res.render('partials/add-todo-error', { error: 'Todo cannot be empty' });
```

## 3. HX-Trigger header is the async dispatch path for `fs-assert-emitted`

Synchronous `document.dispatchEvent` inside a click handler fires BEFORE the [emitted](../assertions/emitted.md) assertion listener is registered. In HTMX, use the `HX-Trigger` response header — HTMX dispatches the CustomEvent on `document.body` after `htmx:afterSettle`, giving the assertion listener time to register.

```js
res.set('HX-Trigger', JSON.stringify({ 'payment:complete': { orderId: '1234' } }));
```

```html
<button
  fs-assert="checkout/payment"
  fs-trigger="click"
  fs-assert-emitted="payment:complete[detail-matches=orderId:\d+]">
  Pay Now
</button>
```

## 4. `fs-assert-oob` is NOT `hx-swap-oob`

Same name, orthogonal concepts:

- [**`fs-assert-oob`**](../assertions/oob.md) (Faultsense) — assertion routing. Fires a secondary assertion on a side-effect element when a named parent passes or fails.
- **`hx-swap-oob`** (HTMX) — DOM delivery. Allows a single server response to replace multiple DOM targets in one round trip, not just the primary swap target.

You can use one, both, or neither. They don't interact.

### Preserve instrumentation across OOB swaps

Prefer `hx-swap-oob="innerHTML:#target"` over `hx-swap-oob="true"` (which is `outerHTML`). innerHTML replaces only the children, leaving the target element and its `fs-*` attributes intact across swaps. outerHTML replaces the element — any `fs-*` attributes you don't re-render on the server are lost.

```ejs
<!-- GOOD: #todo-count keeps its fs-* instrumentation across the OOB swap -->
<span hx-swap-oob="innerHTML:#todo-count"><%= uncompleted %>/<%= todos.length %> remaining</span>

<!-- RISKY: replaces the element, losing fs-assert-oob unless re-rendered -->
<div id="todo-count" hx-swap-oob="true">...</div>
```

## 5. Focus is not reliable on swapped-in inputs — call `.focus()` explicitly

HTMX preserves focus on swapped elements that have a matching `id` pre- and post-swap, but it does NOT consistently focus newly-added inputs. `autofocus` works in some browsers on dynamic insertion and not others. Assertions using [`[focused=true]`](../modifiers/focused.md) on swapped content need an explicit `.focus()` call.

The cleanest pattern: ship a small inline `<script>` in the swap partial itself. HTMX executes inline scripts in swapped content **synchronously during the swap**, which runs *before* the MutationObserver callback microtask fires — so by the time the resolver evaluates `focused=true`, `document.activeElement === input` is already true.

```ejs
<!-- In your edit partial -->
<div id="todo-<%= todo.id %>" class="todo-item" data-status="editing">
  <input class="todo-edit-input" autofocus ... />
  ...
</div>
<script>
  ;(function () {
    var el = document.getElementById('todo-<%= todo.id %>');
    var input = el && el.querySelector('.todo-edit-input');
    if (input) {
      input.focus();
      if (input.select) input.select();
    }
  })();
</script>
```

The leading `;` on the IIFE defends against ASI if another script tag precedes this one in the swap pipeline. `autofocus` is a belt-and-suspenders fallback for browsers that honor it on dynamic insertion.

Alternatively, use `hx-on::after-settle` on the button that triggers the swap — but inline script in the partial keeps the focus logic co-located with the element that needs it.

## 6. MPA mode is for hard navigation only

[`fs-assert-mpa="true"`](../assertions/mpa.md) persists an assertion across a real page navigation. The agent writes it to `localStorage` on `pagehide` and reloads it on the next real `DOMContentLoaded`.

**Do not use `fs-assert-mpa="true"` on hx-boosted routes.** hx-boost never fires a real unload, so MPA assertions created under boost are written to storage but never reloaded — they orphan. Under hx-boost the agent session is long-lived, so you don't need MPA mode: a regular DOM assertion can wait for the new view to render and resolve naturally.

MPA is only appropriate when a subset of your app does actual hard navigation (e.g., a legacy flow wrapped inside an otherwise SPA app, or a form that POSTs to a different origin).

## 7. Scope `stable` assertions OUTSIDE the swap target

[`fs-assert-stable="#foo"`](../assertions/stable.md) fails if any mutation touches `#foo`'s subtree. If `#foo` is inside your `hx-target`, every swap mutates it and stable always fails.

Place stability sentinels OUTSIDE the swap target, or use OOB with a narrow `innerHTML:#foo` swap that updates only text content.

## 8. Form submit and click both fire under HTMX

HTMX calls `preventDefault()` on the native submit event AFTER listeners have already run. Faultsense's capture-phase listeners see the event first — both [`fs-trigger="submit"`](../triggers/submit.md) on the `<form>` and [`fs-trigger="click"`](../triggers/click.md) on the submit button work unchanged. Pick one per assertion.

## 9. Default `hx-target` is the triggering element itself

`<button hx-post="/x">Submit</button>` without an explicit `hx-target` replaces the button's own innerHTML with the response. If you add `fs-assert-updated` on that button, it will fire — but the button is being mutated, so the assertion target needs care. For most cases, set an explicit `hx-target` (e.g., `hx-target="#result-container"`) and assert against that target.

## 10. `hx-swap="outerHTML"` = `added`, not `updated`

This is the most load-bearing rule for HTMX instrumentation: **when HTMX replaces an element via `outerHTML` swap, the agent sees an `added` event, not an `updated` event.** Matching the React convention of [`fs-assert-updated`](../assertions/updated.md) for in-place mutations will silently fail.

Why: the agent's element resolver uses a type-based switch to decide which mutation list to check. For `updated`, it looks at `updatedElements` — which under a `childList` mutation (the DOM shape an outerHTML swap produces) contains the **parent** of the swapped element, not the element itself. The new element lands in `addedElements`, which is only consulted for `added` (and, separately, for `visible` / `hidden`).

React can use `updated` for list-item class changes because React mutates `className` in place, which fires an `attributes` mutation where `mutation.target` IS the todo item. HTMX replaces the whole element, so the mutation shape is different.

```ejs
<!-- WRONG under outerHTML swap: never resolves -->
<input type="checkbox"
  hx-patch="/todos/<%= todo.id %>/toggle"
  hx-target="closest .todo-item"
  hx-swap="outerHTML"
  fs-assert="todos/toggle"
  fs-trigger="change"
  fs-assert-updated=".todo-item[classlist=completed:<%= !todo.completed %>]">

<!-- RIGHT: outerHTML replaces the item, so the new .todo-item is an added node -->
<input type="checkbox"
  hx-patch="/todos/<%= todo.id %>/toggle"
  hx-target="closest .todo-item"
  hx-swap="outerHTML"
  fs-assert="todos/toggle"
  fs-trigger="change"
  fs-assert-added=".todo-item[classlist=completed:<%= !todo.completed %>]">
```

If you're using `innerHTML` swap to update text content without replacing the wrapper (e.g., `hx-swap-oob="innerHTML:#todo-count"`), `updated` IS correct — the wrapper stays and only its subtree mutates. The rule is specifically about `outerHTML`, `delete`, or any swap that **replaces** the matched element.

## Narrow selectors in lists

During a standard (non-morph) `outerHTML` swap, the old and new elements briefly coexist as the browser processes the mutation batch. Under wait-for-pass this isn't a correctness problem, but broad selectors can match the wrong element. Prefer specific ids over class selectors:

```html
<!-- Good: targets the specific item being toggled -->
fs-assert-added="#todo-123[classlist=completed:true]"

<!-- Risky: matches every .todo-item on the page -->
fs-assert-added=".todo-item[classlist=completed:true]"
```

## Fake checkboxes and icon-in-button patterns

HTMX apps often use `<button>` with nested `<span>` or `<svg>` icons instead of native `<input>` elements. Clicks on the inner icon resolve to the button's `fs-trigger` automatically — put the instrumentation on the `<button>`, not on the icon span.

## Dynamic assertion values from server state

The React JSX pattern of interpolating the expected next state works identically in server templates:

```ejs
<input
  type="checkbox"
  <%= todo.completed ? 'checked' : '' %>
  hx-patch="/todos/<%= todo.id %>/toggle"
  hx-target="closest .todo-item"
  hx-swap="outerHTML"
  fs-assert="todos/toggle-complete"
  fs-trigger="change"
  fs-assert-added=".todo-item[classlist=completed:<%= !todo.completed %>]" />
```

EJS `<%= !todo.completed %>` renders `true` or `false` into the attribute — the same semantics as React JSX `{!todo.completed}`. The server already knows the current state, so compute the negation in the template.

## Don't put multiple IIFEs in one inline `<script>`

Classic JavaScript ASI trap:

```html
<script>
  (function () { /* A */ })()
  (function () { /* B */ })()   // ← parsed as A()(B())() — throws
</script>
```

The parser does not insert a semicolon between `})()` and the next `(`. Under hx-boost swap this is guaranteed to crash because the script is re-evaluated as a unit every time the boost re-runs inline scripts.

Two fixes:

1. **Preferred:** move the JS to an external file in `public/` and load it once from the layout's `<head>`. The `<head>` is outside the swap target under `<body hx-boost="true">`, so it's evaluated exactly once per real page load.
2. **Fallback:** if the script must be inline, wrap everything in a single IIFE, or prefix each inner IIFE with a leading `;`.

The same rule applies to event listeners that need to survive boost: delegate from `document` (or `document.body`, which persists across body innerHTML swaps) rather than from a specific element that will get replaced on the next swap.

## See also

- [Frameworks index](../frameworks.md)
- [Mutation pattern PAT-03 outerHTML replacement](../mutation-patterns.md#pat-03-outerhtml-replacement)
- [Mutation pattern PAT-04 morphdom preserved-identity](../mutation-patterns.md#pat-04-morphdom-preserved-identity)
- [`fs-assert-oob`](../assertions/oob.md)
- [`fs-assert-mpa`](../assertions/mpa.md)
