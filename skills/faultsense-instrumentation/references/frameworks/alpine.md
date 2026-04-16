---
title: Alpine
description: Instrumenting Alpine.js apps with fs-* attributes — directive-only, server-rendered HTML.
category: framework
name: alpine
since: 0.1.0
status: stable
---

# Alpine.js

Alpine is a directive-only framework — it operates on server-rendered HTML and provides reactivity via the `x-*` directive set. `fs-*` attributes live alongside Alpine directives in the same markup and work without any integration code.

## Install

Load Faultsense and Alpine in your layout's `<head>`:

```html
<head>
  <script src="/faultsense-agent.min.js" defer></script>
  <script src="/alpine.min.js" defer></script>
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

Alpine binds to elements with `x-data`, `x-show`, `x-on`, etc. on `DOMContentLoaded`. Faultsense binds to `fs-*` attributes on the same event. Order doesn't matter — both scan the DOM.

## Template

Write `fs-*` alongside Alpine directives. They're just attributes, so there's no conflict:

```html
<div x-data="{ count: 0 }">
  <button
    fs-assert="counter/increment"
    fs-trigger="click"
    fs-assert-updated='#count[text-matches=\d+]'
    @click="count++">
    +
  </button>
  <span id="count" x-text="count">0</span>
</div>
```

When the user clicks, Alpine increments `count` and re-renders the `x-text` binding. The agent's `updated` resolver sees the text mutation and resolves.

## Dynamic assertion values with `:fs-*`

Alpine's `:attr` syntax binds a JS expression to any attribute. Use it to compute the expected next state for toggles:

```html
<div x-data="{ completed: false }">
  <input type="checkbox"
    fs-assert="todos/toggle"
    fs-trigger="change"
    :fs-assert-updated="`.todo-item[classlist=completed:${!completed}]`"
    x-model="completed" />
</div>
```

## CSP considerations

The agent uses strict CSP — no inline expressions, no `eval`. **Alpine's default build evaluates `x-*` directives as inline JS expressions, which requires `'unsafe-eval'` in your CSP.** For strict CSP environments, use Alpine's CSP-compliant build:

```html
<script src="/alpine-csp.min.js" defer></script>
```

With the CSP build, Alpine expressions are pre-compiled via a register function rather than evaluated at runtime. This is the supported production configuration — it also happens to be what Faultsense requires internally.

## Gotchas

- **CSP build is required for strict CSP environments.** The default Alpine build uses `eval`. See above.
- **Transitions (`x-transition`)** produce mutation records as classes toggle. The agent's wait-for-pass resolver handles the intermediate states (see [PAT-02 delayed-commit mutation](../mutation-patterns.md#pat-02-delayed-commit-mutation)) so you don't need to special-case transition classes.
- **`x-show="false"`** hides via `display: none`, so [`fs-assert-hidden`](../assertions/hidden.md) is the correct type. `x-if="false"` removes the element — use [`fs-assert-removed`](../assertions/removed.md).

## See also

- [Frameworks index](../frameworks.md)
- [Mutation pattern PAT-02](../mutation-patterns.md#pat-02-delayed-commit-mutation) — how transitions are handled
- [Patterns cookbook](../patterns.md)
