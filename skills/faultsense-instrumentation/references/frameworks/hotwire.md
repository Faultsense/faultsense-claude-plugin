---
title: Hotwire (Turbo + Stimulus)
description: Instrumenting Turbo Streams, Turbo Frames, and Stimulus controllers with fs-* attributes.
category: framework
name: hotwire
since: 0.1.0
status: stable
---

# Hotwire (Turbo + Stimulus)

Hotwire is Rails' default frontend stack. Turbo handles page navigation and partial updates via Turbo Frames and Turbo Streams; Stimulus handles progressive enhancement with lightweight JS controllers. `fs-*` attributes are authored in your ERB templates and reach the DOM as plain HTML.

## Install

Load the agent in your layout (`app/views/layouts/application.html.erb`):

```erb
<!DOCTYPE html>
<html>
<head>
  <%= javascript_include_tag "application", "faultsense-agent", defer: true %>
  <script>
    document.addEventListener("turbo:load", () => {
      Faultsense.init({
        releaseLabel: "<%= ENV['RELEASE_LABEL'] %>",
        collectorURL: "/events",
        apiKey: "<%= ENV['FAULTSENSE_KEY'] %>",
      });
    });
  </script>
</head>
```

Use `turbo:load` instead of `DOMContentLoaded`. Turbo hijacks navigation — `DOMContentLoaded` fires only on full page loads, not on Turbo Drive visits. `turbo:load` fires on both.

## Turbo Drive (page navigation)

Turbo Drive intercepts link clicks and form submissions, fetches the response via XHR, and merges the new `<body>` and `<head>` into the current document. Identity of elements outside the swapped regions is preserved — your instrumented elements survive navigation.

**Do not use [`fs-assert-mpa="true"`](../assertions/mpa.md)** under Turbo Drive. Turbo Drive doesn't fire a real unload, so MPA assertions never reload. A regular DOM assertion will wait for the next view and resolve naturally.

## Turbo Frames

Turbo Frames replace a specific element's contents with fetched HTML. The frame tag (`<turbo-frame id="...">`) remains; its children are swapped.

```erb
<%= turbo_frame_tag "todo-list" do %>
  <%= render @todos %>
<% end %>
```

Assertions inside a frame see their subtree replaced on each navigation — use [`fs-assert-added`](../assertions/added.md) to catch new items after a fetch, or [`fs-assert-updated`](../assertions/updated.md) on elements that preserve identity (the frame itself, containers that render as-is).

## Turbo Streams

Turbo Streams push HTML fragments to the client with named actions: `append`, `prepend`, `replace`, `update`, `remove`, `before`, `after`. The action determines the DOM mutation shape, which determines the correct assertion type.

| Turbo action | DOM effect | Correct type |
|---|---|---|
| `append` | New child appended | [`fs-assert-added`](../assertions/added.md) |
| `prepend` | New child prepended | [`fs-assert-added`](../assertions/added.md) |
| `replace` | Target element replaced (outerHTML) | [`fs-assert-added`](../assertions/added.md) on the new element |
| `update` | Target's innerHTML replaced (wrapper preserved) | [`fs-assert-updated`](../assertions/updated.md) on wrapper, or [`fs-assert-added`](../assertions/added.md) on new children |
| `remove` | Target removed | [`fs-assert-removed`](../assertions/removed.md) |
| `before` / `after` | Sibling inserted next to target | [`fs-assert-added`](../assertions/added.md) |
| `refresh` (morph) | In-place patch via idiomorph | [`fs-assert-updated`](../assertions/updated.md) |

```erb
<!-- app/views/todos/create.turbo_stream.erb -->
<%= turbo_stream.append "todo-list", partial: "todos/todo", locals: { todo: @todo } %>
```

Triggering element:

```erb
<%= button_to "Add",
      todos_path,
      method: :post,
      data: { "turbo-stream": true },
      "fs-assert": "todos/add-item",
      "fs-trigger": "click",
      "fs-assert-added": ".todo-item" %>
```

## Turbo 8 morphing (`refresh="morph"`)

Turbo 8 introduced page-morph refreshes via idiomorph. The full page re-renders on the server and the client patches the DOM in place — identity is preserved. Under a morph refresh, use [`fs-assert-updated`](../assertions/updated.md), not `added`. See PAT-04 morphdom preserved-identity.

## Stimulus controllers

Stimulus attaches JS behavior via `data-controller`, `data-action`, `data-<controller>-<name>-value` attributes. `fs-*` attributes live alongside Stimulus attributes without conflict:

```erb
<button
  data-controller="cart"
  data-action="click->cart#addItem"
  fs-assert="cart/add-item"
  fs-trigger="click"
  fs-assert-updated="#cart-count">
  Add
</button>
```

The Stimulus action fires on click; Faultsense sees the same click via its capture-phase listener. Both run — neither interferes.

## Dynamic assertion values from ERB

The expected-next-state pattern works naturally with ERB:

```erb
<%= check_box_tag "completed",
      todo.id,
      todo.completed,
      "fs-assert": "todos/toggle-complete",
      "fs-trigger": "change",
      "fs-assert-updated": ".todo-item[classlist=completed:#{!todo.completed}]" %>
```

## Gotchas

- **Use `turbo:load`, not `DOMContentLoaded`.** Drive visits don't fire `DOMContentLoaded`.
- **MPA mode doesn't apply under Turbo Drive.** Drive is a long-lived session — regular assertions work across virtual navigations.
- **Turbo 8 morph refreshes are `updated`, not `added`.** Same as HTMX's `morph:outerHTML`.
- **Turbo Frame navigation replaces the frame's children.** An instrumented element inside a frame is lost on frame nav — re-render it with instrumentation intact on the server side.

## See also

- [Frameworks index](../frameworks.md)
- Mutation pattern PAT-04 morphdom preserved-identity
- [HTMX notes](htmx.md) — many of the same patterns apply
