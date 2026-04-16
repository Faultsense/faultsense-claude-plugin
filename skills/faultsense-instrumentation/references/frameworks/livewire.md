---
title: Livewire
description: Instrumenting Laravel Livewire components with fs-* attributes — morph swaps, wire:model, actions.
category: framework
name: livewire
since: 0.1.0
status: stable
---

# Livewire

Livewire is Laravel's full-stack reactivity framework. It re-renders server-side templates on every user action and patches the DOM client-side via morphdom. `fs-*` attributes live directly in your Blade templates.

## Install

Load the agent in your layout (`resources/views/layouts/app.blade.php`):

```blade
<head>
  <script src="{{ asset('js/faultsense-agent.min.js') }}" defer></script>
  @livewireScripts
  <script>
    document.addEventListener('DOMContentLoaded', () => {
      Faultsense.init({
        releaseLabel: '{{ config('app.release_label') }}',
        collectorURL: '/events',
        apiKey: '{{ env('FAULTSENSE_KEY') }}',
      });
    });
  </script>
</head>
```

## Morph swaps — use `updated`

Livewire uses morphdom for all DOM updates. Elements preserve identity across re-renders — attributes and children mutate in place. This is [PAT-04 morphdom preserved-identity](../mutation-patterns.md#pat-04-morphdom-preserved-identity).

**The canonical assertion type for Livewire mutations is [`fs-assert-updated`](../assertions/updated.md)**, not `added`. An item is not "added" — it's an in-place patch that makes the existing element look different.

```blade
<button
  wire:click="increment"
  fs-assert="counter/increment"
  fs-trigger="click"
  fs-assert-updated="#count[text-matches=\d+]">
  +
</button>
<span id="count">{{ $count }}</span>
```

Clicking the button triggers Livewire, which re-renders the component server-side and morphs the response into the DOM. `#count`'s text changes in place — [`updated`](../assertions/updated.md) with [`text-matches`](../modifiers/text-matches.md) catches it.

## wire:model and form controls

`wire:model` binds a form control to a server-side property. The update fires on blur or change by default.

```blade
<input type="text"
  wire:model="search"
  fs-assert="search/filter"
  fs-trigger="blur"
  fs-assert-updated=".results[count-min=0]" />
```

## Actions and DOM mutations

`wire:click` fires an action on the server; the server re-renders and returns HTML; Livewire morphs it in. Track actions via `wire:click` + `fs-trigger="click"`:

```blade
<button
  wire:click="deleteTodo({{ $todo->id }})"
  fs-assert="todos/delete"
  fs-trigger="click"
  fs-assert-removed=".todo-item">
  Delete
</button>
```

Under morphdom, a deletion removes the element from the morph tree — [`removed`](../assertions/removed.md) fires correctly.

## Dynamic assertion values

Blade's expression syntax works the same as ERB/EJS for computing the expected next state:

```blade
<input type="checkbox"
  wire:click="toggle({{ $todo->id }})"
  fs-assert="todos/toggle"
  fs-trigger="change"
  fs-assert-updated=".todo-item[classlist=completed:{{ !$todo->completed ? 'true' : 'false' }}]" />
```

## Gotchas

- **Use `updated`, not `added`, for item mutations.** Livewire morphs — identity is preserved. `fs-assert-added=".todo-item"` will never match on a state toggle.
- **`wire:key` matters.** Livewire uses `wire:key` to match elements during morph. Without stable keys on list items, morphdom may reorder and surprise your assertions. Always set `wire:key="{{ $todo->id }}"` on list items.
- **Full component re-renders** can produce large mutation batches. The agent's [PAT-07 microtask batching](../mutation-patterns.md#pat-07-microtask-batching) handles this — every record is fanned out through the resolver.
- **Alpine inside Livewire.** Livewire ships with Alpine. Use Alpine's CSP build for strict-CSP production environments.

## See also

- [Frameworks index](../frameworks.md)
- [Mutation pattern PAT-04](../mutation-patterns.md#pat-04-morphdom-preserved-identity)
- [Alpine notes](alpine.md)
