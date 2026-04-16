---
title: Phoenix LiveView
description: Instrumenting Phoenix LiveView with fs-* attributes — morph patches, phx bindings, live actions.
category: framework
name: liveview
since: 0.1.0
status: stable
---

# Phoenix LiveView

Phoenix LiveView keeps a persistent WebSocket connection to the server and pushes HTML diffs as state changes. The client morphs them into the DOM via its own morphing implementation — identity is preserved across updates. `fs-*` attributes live in your HEEx templates and survive morph updates.

## Install

Load the agent in your root layout (`lib/my_app_web/components/layouts/root.html.heex`):

```heex
<head>
  <script phx-track-static type="text/javascript" src={~p"/assets/faultsense-agent.min.js"} defer />
  <script phx-track-static type="text/javascript" src={~p"/assets/app.js"} defer />
  <script>
    document.addEventListener("phx:page-loading-stop", () => {
      Faultsense.init({
        releaseLabel: "<%= System.get_env("RELEASE_LABEL") %>",
        collectorURL: "/events",
        apiKey: "<%= System.get_env("FAULTSENSE_KEY") %>",
      });
    });
  </script>
</head>
```

Use `phx:page-loading-stop` to initialize — it fires after LiveView has settled the initial mount.

## Morph updates — use `updated`

LiveView's client morphs incoming diffs into the DOM, preserving element identity where possible. The canonical assertion type is [`fs-assert-updated`](../assertions/updated.md) — elements are patched in place, not added.

This is the same shape as Livewire ([PAT-04 morphdom preserved-identity](../mutation-patterns.md#pat-04-morphdom-preserved-identity)).

```heex
<button
  phx-click="increment"
  fs-assert="counter/increment"
  fs-trigger="click"
  fs-assert-updated="#count[text-matches=\d+]">
  +
</button>
<span id="count"><%= @count %></span>
```

## `phx-click` and actions

```heex
<button
  phx-click="delete_todo"
  phx-value-id={@todo.id}
  fs-assert="todos/delete"
  fs-trigger="click"
  fs-assert-removed={".todo-item[data-todo-id='#{@todo.id}']"}>
  Delete
</button>
```

`phx-click` sends an event over the WebSocket; the server re-renders; the client morphs the response. A deletion removes the element — [`removed`](../assertions/removed.md) fires.

## Form handling

```heex
<.form for={@form} phx-submit="save">
  <input name={@form[:email].name} value={@form[:email].value}
    fs-assert="signup/email-valid"
    fs-trigger="blur"
    fs-assert-updated=".email-field[classlist=valid:true]" />

  <button type="submit"
    fs-assert="signup/submit"
    fs-trigger="click"
    fs-assert-visible=".welcome">
    Sign up
  </button>
</.form>
```

## Dynamic assertion values

HEEx lets you interpolate expressions inside attributes:

```heex
<input type="checkbox"
  phx-click="toggle"
  phx-value-id={@todo.id}
  checked={@todo.completed}
  fs-assert="todos/toggle"
  fs-trigger="change"
  fs-assert-updated={".todo-item[classlist=completed:#{!@todo.completed}]"} />
```

## LiveView `stream` patches

`Phoenix.LiveView.stream/3` manages large lists efficiently by maintaining a keyed structure and pushing only deltas. New items are inserted, removed items are deleted — but identity is preserved for untouched rows. Use [`fs-assert-added`](../assertions/added.md) for new stream entries, [`fs-assert-removed`](../assertions/removed.md) for removals.

## Live navigation (`live_redirect`, `live_patch`)

Live navigation updates the URL and LiveView state without a full page load. The socket stays open; the agent session stays alive. **Don't use [`fs-assert-mpa="true"`](../assertions/mpa.md) with live navigation** — the agent never sees an unload. Regular DOM assertions work across live nav.

## Gotchas

- **Use `updated`, not `added`, for in-place mutations.** LiveView morphs.
- **`phx-update="append"` vs `phx-update="prepend"` vs `phx-update="replace"`** determine the DOM shape. `append` / `prepend` add children (use `added`); `replace` clears and refills (use `added` on new children, or `updated` on the parent).
- **JS hooks (`phx-hook`)** run custom JS per element. They can mutate the DOM in ways LiveView doesn't know about — the agent still observes via MutationObserver, so assertions still resolve.
- **Stream containers need `phx-update="stream"` on the parent.** This enables the keyed diffing that LiveView uses for `stream/3` collections.

## See also

- [Frameworks index](../frameworks.md)
- [Mutation pattern PAT-04](../mutation-patterns.md#pat-04-morphdom-preserved-identity)
- [Livewire notes](livewire.md) — similar patterns apply
