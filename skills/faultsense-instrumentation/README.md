# `@faultsense/skill-instrumentation`

Claude Code / [agentskills.io](https://agentskills.io) skill that teaches AI coding agents how to instrument web apps with [Faultsense](https://faultsense.com) `fs-*` attributes for production correctness monitoring.

When installed, the skill activates whenever you ask your coding agent to "add assertions", "add monitoring", "instrument this component", or "add Faultsense" — and again whenever the agent reviews or modifies existing `fs-*` instrumentation. The skill provides the decision tree for which trigger to use, which assertion type to reach for, and which modifiers to chain, so the instrumentation the agent produces reflects what "correct" actually means for your feature.

## Install

### Claude Code — via the Faultsense marketplace (recommended)

```shell
/plugin marketplace add Faultsense/faultsense-ai
/plugin install faultsense-instrumentation@faultsense-ai
```

Once installed, ask your agent anything Faultsense-shaped — e.g., "add assertions to this checkout form" — and it will activate automatically.

### Claude Code — direct plugin install

```shell
claude --plugin-dir /path/to/cloned/faultsense-claude-plugin
```

Clone the public projection repo:
```shell
git clone https://github.com/Faultsense/faultsense-claude-plugin.git
```

### Cursor, OpenCode, Goose, Gemini CLI, and other agentskills.io-compatible agents

Copy the `skills/faultsense-instrumentation/` subdirectory of the plugin repo into your tool's skills directory:

```shell
git clone https://github.com/Faultsense/faultsense-claude-plugin.git /tmp/fs-plugin
cp -r /tmp/fs-plugin/skills/faultsense-instrumentation ~/.cursor/skills/
#   or ~/.config/opencode/skills/, ~/.goose/skills/, etc. — path varies by tool
```

The `skills/faultsense-instrumentation/` directory is a spec-compliant [agentskills.io](https://agentskills.io) skill package — the `plugin.json` at the repo root is Claude-Code-specific and is ignored by other tools.

## What the skill teaches your agent

- **Trigger selection.** When to use `click` vs `submit` vs `mount` vs `invariant` vs `event:<name>`. Placement rules for nested elements.
- **Assertion-type selection.** The critical `added` vs `updated` vs `visible` distinction that's the #1 instrumentation mistake. When mutation-observed resolvers beat point-in-time `querySelector`.
- **Modifier chaining.** How to refine with `text-matches`, `classlist`, `count`, attribute checks, and per-framework nuances (React JSX, HTMX swap semantics, Vue/Svelte reactive bindings).
- **Conditional outcomes.** `fs-assert-<type>-<condition>` suffix patterns and `fs-assert-mutex` for multiple possible results.
- **OOB (out-of-band) assertions.** When to fire side-effect checks against the parent assertion's pass/fail.
- **Framework integration.** Per-framework binding specifics for React, Vue, Svelte, Alpine, HTMX, Hotwire, Livewire, LiveView, Astro, Solid.

## Requires a running Faultsense collector

The skill authors `fs-*` attributes; the [agent runtime](https://github.com/Faultsense/faultsense-agent) evaluates them in the browser and ships results to a collector. See [faultsense.com/docs](https://faultsense.com) for collector setup.

## License

FSL-1.1-ALv2 (see [`LICENSE`](LICENSE)).
