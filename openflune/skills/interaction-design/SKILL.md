---
name: interaction-design
description: Interaction and micro-interaction design — the Saffer model, named pattern glossary, motion spec format, Figma prototype mechanics (Smart Animate, spring presets), porting motion values to code without inventing them. Use when specifying, designing, or reviewing interactions and micro-interactions.
---

# Interaction Design — vocabulary and spec discipline

## The model (Dan Saffer)

Every micro-interaction is specified in four parts. A spec missing any part is incomplete:

1. **Trigger** — what starts it (tap, long-press, drag, scroll threshold, system event).
2. **Rules** — what happens: states, conditions, what is and isn't allowed.
3. **Feedback** — what the user perceives: visual + haptic (+ sound, rarely).
4. **Loops & modes** — how it evolves: does it repeat, decay, change after N uses?

## Pattern glossary (shared names — use these, don't paraphrase)

| Name | Definition |
|---|---|
| **Spring / overshoot** | Motion passes the target and settles back (physical feel) |
| **Squash & stretch** | Compress on press, release with a spring |
| **Morph** | An element transforms into another without disappearing |
| **Hold-to-confirm** | Destructive/critical action requires a long-press with visible progress; early release springs back with feedback |
| **Rubber banding** | Elastic resistance at scroll/drag limits |
| **Shared element transition** | An element travels between two screens during navigation |
| **Staggered reveal** | Items enter in sequence with a small per-item delay |
| **Skeleton / shimmer** | Structure-shaped placeholder pulsing while content loads |
| **Pull-to-refresh** | Drag past a threshold to reload, with threshold feedback |
| **Progress morph** | A progress indicator transforms into its result state (✓ / ✗) |
| **Color state flip** | A full-surface color change encodes a state change |
| **Parallax** | Layers move at different speeds to convey depth |

## Motion spec format (one block per interaction in `interactions.md`)

```markdown
### <Interaction name>
- **Where**: <screen / component>
- **Trigger**: <tap | long-press (N ms) | drag | system event>
- **Pattern**: <glossary name(s)>
- **Rules**: <states and conditions, incl. what cancels it>
- **Motion**: <duration ms + easing, or spring {stiffness, damping, mass}> [source: figma-panel | platform-default | TODO-tune]
- **Feedback**: <visual> + <haptic: none | impact-light/medium/heavy | notification-success/warning/error | selection>
- **Cancel/fallback**: <what an aborted gesture does — must have feedback, never a silent no-op>
- **Reduce Motion variant**: <the non-moving equivalent, e.g. cross-fade>
- **Loops & modes**: <or "none">
```

## Source tags — the anti-invention rule

**Every motion value carries a source tag.** Allowed tags:

- `[source: figma-panel]` — read from the stiffness/damping/mass or duration/easing shown in
  the Figma prototype animation panel of the actual file.
- `[source: platform-default]` — the platform's stock timing (e.g. iOS navigation transition);
  name the platform primitive, don't restate its numbers from memory.
- `[source: TODO-tune]` — a starting guess that MUST be tuned on a real device before ship.

A number with no tag is a defect. Never present an invented value as measured.

## Figma prototype mechanics

- **Smart Animate matches layers by name** — identical layer names across frames are a hard
  prerequisite. Wire nothing before names are settled (see `figma-craft`).
- Spring presets (Gentle, Quick, Bouncy, Slow) are starting points; the panel shows their
  stiffness/damping/mass — record those values in the spec, then port them.
- Drag triggers for swipe gestures; overlays for sheets; "After delay" only for genuine
  system-driven events.
- A Figma prototype demonstrates **intent**; the spec captures the **contract**; code is the
  implementation. All three must name the same patterns.

## Porting to code

- Springs: carry `{stiffness, damping, mass}` across as-is (e.g. React Native Reanimated
  `withSpring`) and tag `TODO-tune` — engines differ, the device decides.
- Durations: milliseconds + named curve (`ease-out`, `spring`), never "fast"/"smooth".
- Animations must run off the JS/main thread where the stack allows (Reanimated worklets,
  CSS transforms) — an interaction spec that requires per-frame JS is a defect.
- Haptic vocabulary (iOS): confirm/engage → `impact` (light/medium/heavy by weight of the
  action) · outcome → `notification` (success/warning/error) · scrubbing/selection ticks →
  `selection`. Haptics fire at the animation's key moment, not at gesture start.
