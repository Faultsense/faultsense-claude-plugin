---
title: Astro
description: Instrumenting Astro sites with fs-* attributes — SSR pages, hydration islands, framework mix.
category: framework
name: astro
since: 0.1.0
status: stable
---

# Astro

Astro is a component-based SSR framework with "islands architecture" — most of the page is static HTML, interactive components are opt-in via `client:*` directives and hydrate client-side. `fs-*` attributes work on both static HTML and hydrated islands.

## Install

Load the agent in your layout (`src/layouts/Layout.astro`):

```astro
---
const releaseLabel = import.meta.env.RELEASE_LABEL;
---
<html>
  <head>
    <script src="/faultsense-agent.min.js" defer></script>
    <script is:inline define:vars={{ releaseLabel }}>
      document.addEventListener('DOMContentLoaded', () => {
        Faultsense.init({
          releaseLabel: releaseLabel,
          collectorURL: '/events',
          apiKey: 'fs_secret_...',
        });
      });
    </script>
  </head>
  <body>
    <slot />
  </body>
</html>
```

`is:inline` + `define:vars` is Astro's CSP-friendly way to pass build-time values into an inline script without string interpolation.

## Static HTML pages

Non-island pages are plain SSR HTML. `fs-*` attributes work without any client-side JS framework:

```astro
---
// src/pages/contact.astro
---
<Layout>
  <form
    fs-assert="contact/submit"
    fs-trigger="submit"
    fs-assert-mpa="true"
    fs-assert-visible=".success">
    <input type="email" name="email" required />
    <button type="submit">Send</button>
  </form>
</Layout>
```

MPA mode is the right choice here — Astro's default behavior for forms is a full page navigation.

## Hydration islands

Islands are components that hydrate client-side via `client:load`, `client:idle`, `client:visible`, or `client:only`. Each island is a standalone React / Vue / Svelte / Solid / Preact component.

**During hydration, the island's root element's identity is preserved.** The server-rendered HTML is the starting state; the client framework takes over and patches attributes / children as state changes. [`fs-trigger="mount"`](../triggers/mount.md) does NOT re-fire on hydration — the element is already in the DOM.

For "assert something is true once hydrated," use [`fs-trigger="invariant"`](../triggers/invariant.md) or [`fs-assert-updated`](../assertions/updated.md) depending on whether you want continuous monitoring or a one-shot check at the hydration moment.

```astro
---
import Counter from '../components/Counter.jsx';
---
<Counter client:load
  fs-assert="home/counter-hydrated"
  fs-trigger="invariant"
  fs-assert-visible=".counter-ready" />
```

Note: `fs-*` attributes passed to a framework component reach the DOM only if the component forwards props. For React components, see [React notes](react.md).

## Framework mix

Astro lets you mix React, Vue, Svelte, Solid, and Preact islands in the same site. The framework-specific notes for each apply to the islands that use them:

- [React](react.md) — the boolean attribute trap, TypeScript augmentation
- [Vue](vue.md) — `inheritAttrs`
- [Svelte](svelte.md) — `$$restProps` pass-through
- [Solid](solid.md) — JSX binding

## Gotchas

- **CSP and inline scripts.** Astro supports `is:inline` + `define:vars` for build-time values without string interpolation. Prefer external scripts where possible; the `<script>` tag without a directive is bundled and hashed, which is CSP-friendly.
- **`client:only` islands** skip SSR entirely and render only on the client. Elements inside them don't exist until hydration — use [`fs-assert-added`](../assertions/added.md) or [`fs-trigger="mount"`](../triggers/mount.md) for child elements.
- **Image optimization.** Astro's built-in `<Image>` component generates picture/source markup. `fs-assert-loaded` works on the eventual `<img>` element but not on the `<picture>` wrapper.
- **Partial hydration + assertions in `<script>` tags.** Inline scripts inside `.astro` files run once at page load, before island hydration. Don't reference island state from inline scripts.

## See also

- [Frameworks index](../frameworks.md)
- Mutation pattern PAT-09 hydration upgrade
- [React notes](react.md) — React islands
- [Vue notes](vue.md) — Vue islands
- [Svelte notes](svelte.md) — Svelte islands
