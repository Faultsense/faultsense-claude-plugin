---
title: Solid
description: Instrumenting SolidJS components with fs-* attributes — JSX binding, signals, reactive values.
category: framework
name: solid
since: 0.1.0
status: stable
---

# Solid

SolidJS uses JSX, but unlike React, Solid compiles JSX to real DOM operations — there's no virtual DOM. `fs-*` attributes pass through to native elements without configuration.

## Install

Load the agent via a `<script>` tag. Initialize in your entry point:

```jsx
// src/index.jsx
import { render } from 'solid-js/web';
import App from './App';

render(() => <App />, document.getElementById('root'));

Faultsense.init({
  releaseLabel: import.meta.env.VITE_RELEASE,
  collectorURL: '/events',
  apiKey: import.meta.env.VITE_FS_KEY,
});
```

## JSX binding

Static attributes are plain strings:

```jsx
<button
  fs-assert="cart/add-item"
  fs-trigger="click"
  fs-assert-updated="#cart-count"
  onClick={handleAddToCart}>
  Add to cart
</button>
```

Dynamic attributes use expressions. Solid's fine-grained reactivity re-evaluates only the parts that change:

```jsx
<input type="checkbox"
  fs-assert="todos/toggle"
  fs-trigger="change"
  fs-assert-updated={`.todo-item[classlist=completed:${!todo.completed}]`}
  checked={todo.completed}
  onChange={onToggle} />
```

## Signals and reactivity

Solid's signals produce fine-grained DOM updates — often a single `characterData` or `attributes` mutation with no ancestor changes. The agent handles this via PAT-06 text-only mutation, which promotes characterData record targets to their `parentElement` so [`text-matches`](../modifiers/text-matches.md) modifiers work without targeting text nodes directly.

```jsx
const [count, setCount] = createSignal(0);

<button
  fs-assert="counter/increment"
  fs-trigger="click"
  fs-assert-updated='#count[text-matches=\d+]'
  onClick={() => setCount(c => c + 1)}>
  +
</button>
<div id="count">{count()}</div>
```

When `setCount` triggers, Solid updates just the text node inside `#count`. The agent's `updated` resolver sees the text mutation and matches against `#count`.

## Dynamic assertion values

Compute the expected next state in the expression:

```jsx
<input type="checkbox"
  fs-assert="todos/toggle-complete"
  fs-trigger="change"
  fs-assert-updated={`.todo-item[classlist=completed:${!todo.completed}]`} />
```

## Gotchas

- **No virtual DOM means no unnecessary re-renders.** Mutations are precise, which is good for assertion correctness but means broad selectors sometimes miss the mutation target. Prefer specific selectors.
- **Components don't forward props automatically.** Similar to React, if you wrap a native element in a Solid component, you need to spread `props` onto the DOM element: `<button {...props}>`.
- **Reactivity inside `onMount`** runs after initial render. Assertions with [`fs-trigger="mount"`](../triggers/mount.md) fire when Solid inserts the element into the DOM.

## See also

- [Frameworks index](../frameworks.md)
- [Dynamic assertion values](../patterns.md#dynamic-assertion-values)
- Mutation pattern PAT-06 — how the agent handles fine-grained text mutations
