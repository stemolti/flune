# Figma design engine — capabilities, limits, setup

Read this before working on any `--fig` flow. It is the source of truth for what the Figma
engine can and cannot do; skills and agents must not claim capabilities beyond this document.

## The `.fig` reality

- `.fig` is a **proprietary, undocumented binary format**. No tool outside Figma can author a
  valid `.fig` file. Community reverse-engineering exists but is fragile and NOT a supported
  path — openflune never uses it.
- The **Figma REST API cannot create or edit design content** (read-oriented: files, nodes,
  images, comments). Programmatic authoring exists only in the **Plugin API**, which runs
  inside Figma.
- Consequence: openflune produces **Figma-native deliverables** (imports and bridge-driven
  edits). The `.fig` file itself is always saved by the user from Figma
  (File → Save local copy). This is a hard constraint, not a temporary limitation.

## Integration levels

### Level 1 — Import package (zero setup, always available)

`/openflune:design --fig` produces `<designPath>/<slug>/`:

| File | Role | How it enters Figma |
|---|---|---|
| `tokens.json` | Design tokens, Tokens Studio format | Tokens Studio plugin (free) → variables/styles |
| `frames/*.svg` | Pixel-exact reference frames per screen × state | Drag onto the canvas (editable vectors; **no auto layout/components** — those get rebuilt per `figma-craft`) |
| `DESIGN.md` | Implementation spec (screens, components, tokens) | Read by humans and by the implement pipeline |
| `interactions.md` | Motion contract (Saffer blocks, source-tagged values) | Guides prototype wiring and code |
| `figma-import.md` | Numbered import steps | — |

### Level 2 — Live bridge (optional, user-configured)

Community MCP bridges pair an MCP server with a **plugin running inside the user's open Figma
file** (typically over a local WebSocket), giving the agent real authoring: frames, auto
layout, components, variables. Separately, import services (e.g. HTML-to-Figma converters)
exist as paid products with their own APIs.

Bridge names, capabilities, and availability **change over time — verify at setup, never
assume**:

1. The user installs and connects a bridge MCP server + its Figma plugin, per that project's
   own docs.
2. Record the server name in `.claude/config.json` → `figma.bridge` so Phase 0.5 probes the
   right server first.
3. The engine discovers tool names **at runtime** and uses only what the probe advertises.

If no bridge is connected, the engine automatically stays at Level 1 — it never fails for a
missing bridge and never fakes bridge output.

## Config schema

```json
"figma": {
  "enabled": true,
  "designPath": "designs/figma/",
  "bridge": "talk-to-figma"
}
```

- `figma.enabled` — gate for the `--fig` engine (the design skill self-configures this key
  with user consent on first use if absent).
- `figma.designPath` — where design packages live. Default `"designs/figma/"`.
- `figma.bridge` — optional: MCP server name of a connected bridge; absent = Level 1.

## Versioning policy

- The **package is the committed source of truth** — it is diffable, reviewable, and
  regenerable.
- The `.fig` may be committed alongside (`<designPath>/<slug>.fig`) after the user saves it
  from Figma; treat it as a binary artifact (no merges — last write wins; keep packages small
  and per-feature so this rarely hurts).

## Anti-hallucination rules (binding for all Figma work)

1. Never invent Figma MCP tool names, plugin names as facts, motion values, or links. Probe,
   read, or mark `TODO`.
2. Never tag a motion value `figma-panel` unless it was actually read from a connected Figma
   file's animation panel.
3. Never claim a `.fig` was created or updated by the engine — only the user's save does that.
4. When a capability is unavailable, say so in one factual sentence and continue at Level 1.
