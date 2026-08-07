---
name: figma-craft
description: Professional Figma construction rules — file organization, auto layout, components and variants, variables/tokens, prototype wiring, dev handoff. Use when building or reviewing Figma files or Figma design packages so that professional designers can take over the file without cleanup.
---

# Figma Craft — construction rules

The bar: a designer hired tomorrow opens the file and finds it organized the way a mature
product team's design system is organized. Every rule below is checkable — the design-critic
agent uses this file as a rubric.

## File organization

- Fixed page order: `📄 Cover` → `🧩 Components` → `📱 Screens` → `🔀 Flows` → `🧪 Playground` → `🗄 Archive`.
  Small projects may merge Flows into Screens; never scatter components across screen pages.
- Flows read **left → right**, states of the same screen stack **top → bottom**.
- Frames sit on an **8pt grid**; frame x/y positions are multiples of 8.
- Use Sections to group related frames per feature; name them after the feature, not "Group 12".

## Naming

- **Zero default names.** No "Frame 427", "Rectangle 12", "Component 3" anywhere.
- Components: slash taxonomy — `Button/Primary`, `Card/Run/Compact`. Variant properties use
  `Property=Value` (`State=Hover`, `Size=Large`).
- Layers: semantic names (`icon-pause`, `label-distance`, `bg-map`), kebab-case, consistent
  across the whole file.
- **Smart Animate depends on identical layer names across frames** — naming is prototype
  infrastructure, not cosmetics. Name layers before wiring any prototype.

## Auto layout

- Every frame that contains content uses auto layout. Absolute positioning only for true
  overlays (FAB, toast, badge-on-avatar).
- Hug/Fill discipline: text hugs, containers fill; fixed sizes only when the design demands it
  (e.g. a 44pt touch target).
- Spacing and padding come from the spacing tokens — no hand-typed magic numbers.
- Set min/max widths on text containers so translations don't break layouts.

## Components and variants

- Anything used twice is a component. Anything with states is a component **with variants**.
- Interactive components ship the full state set: `Default / Hover / Pressed / Focus /
  Disabled` (plus `Loading` where async).
- Use component properties (boolean, text, instance-swap) instead of variant explosion; expose
  nested instances rather than duplicating wrappers.
- Private base pattern: `.base/Button` holds structure; public variants wrap it.
- Every component has a **description** (usage, do/don't) — Dev Mode surfaces it.

## Variables and tokens

- Three tiers: **primitive** (`blue-600`) → **semantic** (`action/primary`) → component-level
  where needed. Layers bind to **semantic** tokens, never to primitives, never to raw hex.
- Modes for light/dark on the semantic collection; a layer never hardcodes a per-mode value.
- The repo-committed `tokens.json` (Tokens Studio format) is the file of record; Figma
  variables are generated from it, not the other way around.

## Prototype wiring

- Named flows with explicit starting points — one flow per user journey.
- Smart Animate between frames whose layers share names; use spring presets and **record the
  stiffness/damping/mass values shown in Figma's animation panel** in the interaction spec —
  those exact values port to code.
- Overlays for sheets/dialogs (not duplicated full screens); "After delay" only for system
  events, never to fake data loading in user-triggered flows.

## Accessibility annotations (in-file)

- Touch targets ≥ 44×44pt — annotate any control that looks smaller but has a larger hit area.
- Text contrast AA (4.5:1 body, 3:1 large) — verify against real token values, don't eyeball.
- Note focus order and VoiceOver labels for icon-only controls on the frame's annotation layer.

## Dev handoff

- Mark sections "Ready for dev" only when: states complete, tokens bound, descriptions filled,
  interaction spec written.
- Link the code path (component file) and the spec (`DESIGN.md`, `interactions.md`) in the
  component/section description so Dev Mode shows them.
