# Faultsense Instrumentation — Claude Code plugin

Claude Code plugin that installs the [Faultsense](https://faultsense.com) instrumentation skill. Asks your AI coding agent to reason like an E2E test author when it adds `fs-*` attributes to your HTML.

When installed, the skill activates whenever you ask your agent to "add assertions", "add monitoring", "instrument this component", or "add Faultsense" — and again whenever the agent reviews or modifies existing `fs-*` instrumentation.

## Install

### Via the Faultsense plugin marketplace (recommended)

```shell
/plugin marketplace add Faultsense/faultsense-ai
/plugin install faultsense-instrumentation@faultsense-ai
```

### Direct via `--plugin-dir`

Clone this repo and point Claude Code at it:

```shell
git clone https://github.com/Faultsense/faultsense-claude-plugin.git
claude --plugin-dir ./faultsense-claude-plugin
```

### Non-Claude-Code agents (Cursor, OpenCode, Goose, Gemini CLI, etc.)

The `skills/faultsense-instrumentation/` subdirectory of this repo is a spec-compliant [agentskills.io](https://agentskills.io) skill package. Copy it into your tool's skills location:

```shell
git clone https://github.com/Faultsense/faultsense-claude-plugin.git /tmp/fs-plugin
cp -r /tmp/fs-plugin/skills/faultsense-instrumentation ~/.cursor/skills/
#   or ~/.config/opencode/skills/, ~/.goose/skills/, etc. — path varies by tool
```

The `plugin.json` at the repo root is Claude-Code-specific and is benignly ignored by other tools.

## What the skill teaches your agent

- Trigger selection (`click`, `submit`, `mount`, `invariant`, `event:<name>`) and placement rules
- The `added` vs `updated` vs `visible` distinction — the #1 instrumentation mistake
- Modifier chaining (`text-matches`, `classlist`, `count`, `detail-matches`, attribute checks)
- Conditional outcomes with `fs-assert-<type>-<condition>` suffix patterns and `fs-assert-mutex`
- OOB (out-of-band) assertions for side-effect checks
- Framework-specific binding nuances for React, Vue, Svelte, Alpine, HTMX, Hotwire, Livewire, LiveView, Astro, Solid

## Requires a Faultsense collector

The skill authors `fs-*` attributes; the [Faultsense agent](https://github.com/Faultsense/faultsense-agent) evaluates them in the browser and ships results to a collector. See [faultsense.com](https://faultsense.com) for collector setup.

## About this repo

This repo is a **one-way release projection** from the private Faultsense monorepo. Every commit here corresponds to a `skill/v<semver>` release tag on the source repo. Issues and PRs filed here will not be addressed — please file them upstream.

Each release ships:

- `.claude-plugin/plugin.json` — Claude Code plugin manifest
- `skills/faultsense-instrumentation/` — the agentskills.io-compliant skill package
  - `SKILL.md` — agent-activation layer (decision tree, placement rules, common mistakes)
  - `references/` — offline-complete reference for every `fs-*` trigger, assertion type, modifier, and supported framework
  - `README.md`, `LICENSE`, `package.json`

## License

[FSL-1.1-ALv2](LICENSE). The Functional Source License (FSL-1.1) grants broad use of the skill with a two-year transition to Apache 2.0. See the LICENSE file for the full terms.
