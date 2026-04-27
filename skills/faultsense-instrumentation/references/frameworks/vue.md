---
title: Vue
description: Instrumenting Vue components with fs-* attributes — template binding, inheritAttrs, dynamic expressions.
category: framework
name: vue
since: 0.1.0
status: stable
---

# Vue

Vue passes unknown attributes through to DOM elements via `inheritAttrs` (default: `true`). `fs-*` attributes work without configuration — write them directly in the template.

## Install

Load the agent via a `<script>` tag in `index.html` or your app entry. Initialize after `app.mount()`.

```js
// src/main.js
import { createApp } from 'vue';
import App from './App.vue';

createApp(App).mount('#app');

Faultsense.init({
  releaseLabel: import.meta.env.VITE_RELEASE,
  collectorURL: '/events',
  apiKey: import.meta.env.VITE_FS_KEY,
});
```

## Template binding

Static attributes are plain HTML:

```vue
<template>
  <button fs-assert="cart/add-item" fs-trigger="click"
    fs-assert-updated="#cart-count"
    @click="handleAddToCart">
    Add to cart
  </button>
</template>
```

Dynamic attributes use `:attr` binding:

```vue
<template>
  <input type="checkbox"
    fs-assert="todos/toggle"
    fs-trigger="change"
    :fs-assert-updated="`.todo-item[classlist=completed:${!todo.completed}]`"
    v-model="todo.completed" />
</template>
```

## `inheritAttrs`

Vue components with a single root DOM element inherit parent-passed attributes automatically. `fs-*` attributes passed to a component reach its root element:

```vue
<!-- Parent -->
<MyButton fs-assert="..." fs-trigger="click" fs-assert-added=".result">Click</MyButton>

<!-- MyButton.vue — fs-* attributes land on the <button> root -->
<template>
  <button class="my-button">
    <slot />
  </button>
</template>
```

**Multi-root components** (Vue 3 fragments) and components with `inheritAttrs: false` need manual forwarding via `$attrs` or `v-bind="$attrs"` on the element you want to instrument.

## Dynamic assertion values

For bidirectional toggles, compute the expected next state in the binding:

```vue
<template>
  <input type="checkbox"
    fs-assert="todos/toggle-complete"
    fs-trigger="change"
    :fs-assert-updated="`.todo-item[classlist=completed:${!todo.completed}]`"
    v-model="todo.completed" />
</template>
```

When Vue re-renders on prop changes, the attribute value is recomputed — the assertion always reflects the expected post-toggle state.

## SSR (Nuxt, Vue SSR)

SSR-rendered elements are hydrated, not re-mounted. [`fs-trigger="mount"`](../triggers/mount.md) doesn't re-fire on hydration. Use [`fs-trigger="invariant"`](../triggers/invariant.md) for "should be true across hydration" contracts. See PAT-09 hydration upgrade.

## Gotchas

- **`inheritAttrs: false` components drop `fs-*`.** Use `v-bind="$attrs"` on the element you want to instrument.
- **Multi-root components** (fragments) need explicit routing of `fs-*` attributes to the right root.
- **Scoped styles generate hashed classes.** If you use `<style scoped>` and instrument against the generated class names, your selectors become fragile. Prefer stable `data-*` attributes for assertions.
- **Teleport (`<Teleport>`)** moves the subtree to another part of the DOM. The agent still sees mutations because the observer is rooted at `document.body`, but relative selectors inside a teleported subtree may target unexpected elements.

## See also

- [Frameworks index](../frameworks.md)
- [Dynamic assertion values](../patterns.md#dynamic-assertion-values)
- [Patterns cookbook](../patterns.md)
