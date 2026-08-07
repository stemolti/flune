---
name: design-figma
description: Figma engine for /openflune:design --fig. Interactive design reasoning that produces Figma-native deliverables (Tokens Studio JSON, importable SVG frames, DESIGN.md, interaction specs) and drives a live Figma bridge MCP when one is configured. Loaded by the design skill's Engine Selection step — not invoked directly.
---

# Figma Design Engine

## Reality constraints (non-negotiable — every claim in outputs must respect these)

1. **`.fig` is a proprietary, undocumented binary format.** Nothing outside Figma can author a
   valid `.fig` file. Never claim to generate one, never attempt to write one byte-by-byte.
2. **The Figma REST API cannot create or edit design content** (it is read-oriented). Design
   authoring from outside Figma happens only through:
   - a **bridge MCP** paired with a plugin running inside the user's open Figma file, or
   - **Figma-native imports**: SVG (editable vectors, but no auto layout/components) and
     Tokens Studio JSON (tokens → Figma variables/styles via the Tokens Studio plugin).
3. **Never hardcode or invent Figma MCP tool names.** Bridge servers vary; discover tools at
   runtime. If a capability is unavailable, degrade to the import package and say so plainly.
4. The **committed source of truth is the design package** (files below). The `.fig` itself is
   saved by the user from Figma (File → Save local copy) — prompt for it exactly like the
   Pencil save reminder, never pretend it happened automatically.

## Deliverable model — two levels, auto-selected in Phase 0.5

| Level | When | Output |
|---|---|---|
| **bridge** | A Figma-capable MCP server is connected and probes OK | Design built live inside the open Figma file (pages, auto-layout frames, components, variables) per the `figma-craft` skill — **plus** the import package below (still the committed spec) |
| **package** | No bridge available | `<designPath>/<slug>/` containing `tokens.json`, `frames/*.svg`, `DESIGN.md`, `interactions.md`, `figma-import.md` |

## Phase 0 — Context Loading

**Config check**: identical to the design skill — read `.claude/config.json`; if missing, stop:
"openflune is not configured for this project. Run `/openflune:configure` first to set up."

**Figma gating**: read the `figma` key:

- `figma.enabled: true` → use `figma.designPath` (default `"designs/figma/"` if absent).
- Key **absent** → self-configure with consent. Ask via `AskUserQuestion`:
  > "Figma design workflows aren't configured for this project yet. Enable them?"
  Options: "Enable (designs/figma/)", "Enable with custom path" (+ description field), "Cancel".
  On enable, merge `"figma": { "enabled": true, "designPath": "<path>" }` into
  `.claude/config.json` **preserving every existing key**. On "Cancel" → **Stop.**
- `figma.enabled: false` → stop: "Figma design workflows are disabled for this project.
  Re-enable via `/openflune:configure` or edit `.claude/config.json`."

## Phase 0.5 — Bridge Discovery

Determine `$FIGMA_LEVEL`:

1. If `.claude/config.json` has `figma.bridge` (a user-recorded MCP server name), probe that
   server first with its lightest discovery/read call.
2. Otherwise inspect the available MCP tools for a Figma-capable server (names beginning
   `mcp__figma`, or a community bridge under whatever server name the user configured).
3. **Probe succeeds** → `$FIGMA_LEVEL = "bridge"`. Note the discovered tool names — use only
   those, exactly as advertised.
4. **No server / probe fails** → `$FIGMA_LEVEL = "package"`. Tell the user once, factually:
   "No Figma bridge MCP is connected — I'll produce the import package. See `docs/figma.md`
   for live-editing setup." **Never fail the run because a bridge is missing.**

## Argument Parsing

Same rules as the design skill: detect a leading `--mobbin` token → `$MOBBIN_MODE`; then
**ticket mode** (first token matches `^#?\d+$` — fetch via `gh issue view` after reading the
`shell-rules` skill; look for a **Design Direction** section) or **ticketless mode** (the
remaining string is the design description). Read relevant `docs/<topic>.md` entries.

## Phase 1 — Attachments

Ticket mode only: follow the `attachments` reference skill (discover, present, download, load).
Ticketless mode: skip.

## Phase 2 — Design Understanding

Forced reasoning phase — no files are created yet.

**Load the knowledge skills first** (read all three in full):

- `${CLAUDE_PLUGIN_ROOT}/skills/figma-craft/SKILL.md` — how the Figma file/package must be built
- `${CLAUDE_PLUGIN_ROOT}/skills/ux-patterns/SKILL.md` — states, platform conventions, accessibility
- `${CLAUDE_PLUGIN_ROOT}/skills/interaction-design/SKILL.md` — interaction/micro-interaction vocabulary and spec format

**Classify the design type** using the same table as the design skill (screen/page, component,
dashboard, landing-page, form/wizard, slides).

**Propose-first questioning** — `AskUserQuestion`, one at a time, max 5, skip anything the
Design Direction already answers:

1. **Scope validation** — "I'll design [specific thing] containing [proposed elements]. Match?"
2. **Design system discovery** — Glob `<designPath>/**/tokens.json` and existing package dirs.
   Found → propose reusing those tokens/components as the system; multiple → let the user pick;
   none → designing from scratch (say so).
3. **Visual direction** — propose a specific aesthetic (from Design Direction if present).
4. **Screen states** (screens/forms only, multiSelect) — empty / populated / error / loading.
5. **Interactions in scope** — propose 2–4 named patterns from the `interaction-design`
   glossary that fit this design ("morph on submit, hold-to-confirm on delete…"); multiSelect
   plus "None — static design".

## Phase 2.7 — Mobbin Reference Gathering

**Only if `$MOBBIN_MODE` is `true`.** Follow **Phase 2.7 of the design skill verbatim**: the
paid-feature gate on `mobbin.enabled`, runtime tool discovery (never hardcode Mobbin tool
names), rate-limit discipline, reference presentation and selection into `$MOBBIN_REFERENCES`.
Cite **only** links actually returned by the Mobbin server.

## Phase 3 — Package Creation

Create `<designPath>/<slug>/` (slug from ticket title or description). All paths absolute.

### 3A — `tokens.json` (Tokens Studio format)

- `global` set: primitives — color ramps, typography (family/size/weight/lineHeight), spacing
  scale, radii.
- `semantic` set: purpose tokens (`bg.surface`, `text.primary`, `action.primary`…) that
  **reference** primitives — no raw values in the semantic set.
- When dark mode is in scope: two theme sets mapping the same semantic names.
- Reuse the existing `tokens.json` when Phase 2 found one — extend, don't fork.

### 3B — `frames/*.svg`

One SVG per screen × state, at exact target dimensions (e.g. 390×844 for iOS), text as real
`<text>` elements, meaningful `id` attributes on groups (they become layer names on import).
These import into Figma as **editable vectors without auto layout or components** — they are
visual reference frames. The component build happens inside Figma per `figma-craft` (via
bridge, or by a designer following the package). State this plainly in `figma-import.md`.

### 3C — `interactions.md`

Delegate to the **interaction-spec-writer** agent (Task tool) with: approved scope, selected
patterns from Phase 2 Q5, screen states, and the package path. On return, **verify** every
motion value carries a source tag (`figma-panel` / `platform-default` / `TODO-tune`) — reject
and re-run if any value is untagged. Skip this step entirely if the user chose "None — static
design".

### 3D — `DESIGN.md`

Reuse the structure of `${CLAUDE_PLUGIN_ROOT}/templates/design-spec.md`, adapted for Figma:
screens table references `frames/<file>.svg` instead of Pencil node IDs; components table maps
package components to framework names (framework from `stack.frontend` in config, as in the
design skill); tokens table generated from `tokens.json`. If `$MOBBIN_REFERENCES` is non-empty,
append the `## Design References (Mobbin)` section exactly as the design skill specifies.

### 3E — `figma-import.md`

Numbered, exact steps for a human:

1. Import the SVGs (drag onto the Figma canvas).
2. Install the **Tokens Studio** plugin → load `tokens.json` → apply as variables/styles.
3. Rebuild components with auto layout per the `figma-craft` checklist (linked).
4. Wire prototypes per `interactions.md` — **Smart Animate requires identical layer names
   across frames**; name layers before wiring.
5. If the team versions `.fig`: File → Save local copy → `<designPath>/<slug>.fig`, commit it
   alongside the package.

### Bridge mode (additionally, when `$FIGMA_LEVEL = "bridge"`)

Execute the build inside the user's open Figma file via the discovered bridge tools, following
`figma-craft`: page structure, auto-layout frames, components with variants, variables from
`tokens.json`. Batch independent operations; read state back at decision boundaries. If the
bridge offers screenshot/export, use it for Phase 4 visual checks. The package files are
**still written** — they remain the committed spec.

## Phase 4 — Review Loop

1. Run the **design-critic** agent (Task tool) on the package: pass the package path, the
   chosen direction, and the three knowledge skills as the rubric. Fix genuine defects it
   finds; re-run once after fixes.
2. Present to the user via `AskUserQuestion` — same semantics as the design skill:
   **Approve** → Phase 5 · **Request changes** → apply, re-validate, re-present ·
   **Start over** → back to Phase 2 visual direction.

## Phase 5 — Report

- Package path and file list; `$FIGMA_LEVEL` used; key design decisions; Mobbin references
  (if any); screens/components/tokens counts.
- End with the `.fig` note:
  > "The committed source of truth is the design package. To version the `.fig` too: open the
  > design in Figma, File → Save local copy into `<designPath>/`, and commit it alongside."

**Ticket mode label discipline**: add "Working" at the start of Phase 2, as the design skill does.

## Phase 6 — Commit

Same rules as the design skill Phase 6: current branch, **no push, no PR**. Stage only
`<designPath>/<slug>/` (and the `.fig` only if the user confirmed they saved it there):

```bash
git add <designPath>/<slug>/ && git commit -m "feat(design): <description>"
```

Ticket mode: reference `#<ticket-id>` in the body; swap label "Working" → "Designed".
Error recovery: identical to the design skill (show commands on commit failure; labels are
non-blocking). Then **STOP** — do not offer implementation. Final message mirrors the design
skill ("Run `/openflune:implement <ticket-id>` when ready" in ticket mode).

## Anti-slop guardrails (apply throughout every phase)

- Capability claims about Figma come only from this skill, `docs/figma.md`, or a runtime
  probe — never from memory of blog posts or model priors.
- Never fabricate: Figma tool names, Mobbin links, motion values, token values "that look
  right". Unknown → ask the user or mark `TODO` explicitly.
- Prefer fewer, denser artifacts. No placeholder sections, no lorem ipsum, no empty tables,
  no states drawn "for completeness" that the user did not request.
