---
title: React
description: Instrumenting React components with fs-* attributes — JSX binding, TypeScript augmentation, dynamic values.
category: framework
name: react
since: 0.1.0
status: stable
---

# React

React passes unknown HTML attributes through to DOM elements on native elements (`<button>`, `<form>`, `<div>`, etc.). `fs-*` attributes work without any binding configuration — just write them in JSX.

## Install

Standard [installation](../installation.md) applies. Load the agent via a `<script>` tag in your HTML template or inside `index.html`; call `Faultsense.init()` once React has mounted.

```jsx
// src/index.jsx
import { createRoot } from 'react-dom/client';
import App from './App';

createRoot(document.getElementById('root')).render(<App />);

// After hydration, init the agent
Faultsense.init({
  releaseLabel: import.meta.env.VITE_RELEASE_LABEL,
  collectorURL: '/events',
  apiKey: import.meta.env.VITE_FS_API_KEY,
});
```

## JSX binding

`fs-*` attributes are plain strings in JSX. React forwards them to the DOM on native elements.

```jsx
<button
  fs-assert="cart/add-item"
  fs-trigger="click"
  fs-assert-updated="#cart-count"
  onClick={handleAddToCart}>
  Add to cart
</button>
```

## TypeScript augmentation

TypeScript flags `fs-*` attributes as unknown props on native elements. Augment React's `HTMLAttributes` interface once to silence the warnings:

```typescript
// src/types/faultsense.d.ts
declare namespace React {
  interface HTMLAttributes<T> {
    'fs-assert'?: string
    'fs-trigger'?: string
    'fs-assert-added'?: string
    'fs-assert-removed'?: string
    'fs-assert-updated'?: string
    'fs-assert-visible'?: string
    'fs-assert-hidden'?: string
    'fs-assert-loaded'?: string
    'fs-assert-stable'?: string
    'fs-assert-emitted'?: string
    'fs-assert-after'?: string
    'fs-assert-timeout'?: string
    'fs-assert-mpa'?: string
    'fs-assert-mutex'?: string
    'fs-assert-oob'?: string
    'fs-assert-oob-fail'?: string
    // Conditional variants follow the fs-assert-<type>-<condition> pattern:
    // 'fs-assert-added-success'?: string, 'fs-assert-added-error'?: string, etc.
  }
}
```

Place the file anywhere TypeScript can find it (`src/types/` is conventional). No import needed — it augments the global namespace automatically.

## The boolean attribute trap

**React drops custom attributes with boolean `true` values.** Always use explicit string values for `fs-*` attributes in JSX:

```jsx
// Correct
<button fs-assert-mutex="each">

// WRONG — React drops or emits value="true" inconsistently
<button fs-assert-mutex>
```

This applies to [`fs-assert-mpa`](../assertions/mpa.md) and [`fs-assert-mutex`](../assertions/mutex.md) most commonly. Use `"true"` and `"each"` explicitly.

## Dynamic assertion values

For bidirectional toggles, compute the expected **next** state in the attribute value using a template literal:

```jsx
<input type="checkbox"
  fs-assert="todos/toggle-complete"
  fs-trigger="change"
  fs-assert-updated={`.todo-item[classlist=completed:${!todo.completed}]`}
  checked={todo.completed}
  onChange={onToggle} />
```

When `todo.completed` is `false`, the assertion checks for `completed:true` (checking). When `true`, it checks for `completed:false` (unchecking). Compute the negation in the template — the agent doesn't track "state before the click."

## Custom components

Native elements pass `fs-*` through automatically. Custom components need to forward props to the root DOM element:

```jsx
function Button({ children, ...props }) {
  return (
    <button {...props} className="btn">
      {children}
    </button>
  );
}

// Works because Button spreads props onto the native button
<Button fs-assert="..." fs-trigger="click" fs-assert-added=".result">Click</Button>
```

If your component wraps the root element or filters props, make sure `fs-*` props reach the DOM element or instrument the rendered element directly from the parent.

## StrictMode and double-mount

React 18 `StrictMode` double-mounts components in development (insert → unmount → insert). The agent handles this naturally via its PAT-05 detach-reattach resolution — `added` assertions pass on the final insert, not the intermediate states.

## SSR and hydration

An SSR-rendered element that the client hydrates is **not** re-mounted. Its attributes change, but DOM identity is preserved. [`fs-trigger="mount"`](../triggers/mount.md) does NOT re-fire on hydration. Use [`fs-trigger="invariant"`](../triggers/invariant.md) for "should be true across hydration" contracts, or [`fs-assert-updated`](../assertions/updated.md) if you want to catch the exact hydration moment.

See PAT-09 hydration upgrade for the regression lock.

## Gotchas

- **Bare boolean attribute drop** (above) — always use explicit strings.
- **Forgetting to forward props** on custom components — `fs-*` gets dropped at the component boundary.
- **`className` toggles look like new elements.** Toggling `.complete` on an existing `<div>` is [`updated`](../assertions/updated.md), not [`added`](../assertions/added.md). The element's identity is preserved — only its class list changes.
- **Keyed list reorders** (React 18) trigger detach-reattach on the moved items. `added` assertions can fire on the reinsert — scope keys to make intent clear.

## See also

- [Frameworks index](../frameworks.md)
- [Patterns cookbook](../patterns.md)
- [Dynamic assertion values](../patterns.md#dynamic-assertion-values)
