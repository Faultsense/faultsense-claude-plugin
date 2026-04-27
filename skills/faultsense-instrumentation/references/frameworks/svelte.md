---
title: Svelte
description: Instrumenting Svelte components with fs-* attributes — template binding, reactive values, SvelteKit.
category: framework
name: svelte
since: 0.1.0
status: stable
---

# Svelte

Svelte passes unknown attributes through on native elements without configuration. `fs-*` attributes work directly in markup.

## Install

Load the agent via a `<script>` tag. Initialize in your entry point or layout:

```svelte
<!-- src/routes/+layout.svelte (SvelteKit) -->
<script>
  import { onMount } from 'svelte';

  onMount(() => {
    Faultsense.init({
      releaseLabel: import.meta.env.VITE_RELEASE,
      collectorURL: '/events',
      apiKey: import.meta.env.VITE_FS_KEY,
    });
  });
</script>
```

## Template binding

Static attributes are plain HTML:

```svelte
<button
  fs-assert="cart/add-item"
  fs-trigger="click"
  fs-assert-updated="#cart-count"
  on:click={handleAddToCart}>
  Add to cart
</button>
```

Dynamic attributes use `{expression}` inside the attribute value:

```svelte
<input type="checkbox"
  fs-assert="todos/toggle"
  fs-trigger="change"
  fs-assert-updated={`.todo-item[classlist=completed:${!todo.completed}]`}
  bind:checked={todo.completed} />
```

## Dynamic assertion values

For bidirectional toggles, compute the expected next state in the expression:

```svelte
<input type="checkbox"
  fs-assert="todos/toggle-complete"
  fs-trigger="change"
  fs-assert-updated={`.todo-item[classlist=completed:${!todo.completed}]`}
  checked={todo.completed}
  on:change={onToggle} />
```

Svelte's reactivity re-computes the expression whenever `todo.completed` changes.

## Custom components

Svelte components don't auto-forward unknown props. If you wrap a native element in a component and want `fs-*` to reach the DOM, use `$$restProps`:

```svelte
<!-- Button.svelte -->
<script>
  // no explicit export for fs-* — they live in $$restProps
</script>

<button class="btn" {...$$restProps}>
  <slot />
</button>
```

Now:

```svelte
<Button fs-assert="..." fs-trigger="click" fs-assert-added=".result">Click</Button>
```

reaches the native button.

## SvelteKit SSR + hydration

SvelteKit SSRs routes and hydrates them client-side. [`fs-trigger="mount"`](../triggers/mount.md) doesn't re-fire on hydration — identity is preserved. Use [`fs-trigger="invariant"`](../triggers/invariant.md) for hydration-spanning contracts or [`fs-assert-updated`](../assertions/updated.md) if you want to catch the hydration moment specifically.

See PAT-09 hydration upgrade.

## Gotchas

- **`$$restProps` is required for pass-through components.** Svelte doesn't spread unknown props by default.
- **`bind:` directives create two-way binding.** The `fs-assert-updated` on a bound checkbox runs on the `change` event, not on every Svelte reactivity tick — which is what you want.
- **Stores triggering mass updates** can fire many mutations in one microtask. The agent's PAT-07 microtask batching handles this.

## See also

- [Frameworks index](../frameworks.md)
- [Dynamic assertion values](../patterns.md#dynamic-assertion-values)
- [Patterns cookbook](../patterns.md)
