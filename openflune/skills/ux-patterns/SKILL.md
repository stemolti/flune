---
name: ux-patterns
description: UX heuristics for screens and flows — screen state matrix, platform conventions (iOS/Android), action hierarchy, forms, accessibility, UI copy. Use when designing or reviewing screens, flows, or design packages.
---

# UX Patterns — heuristics and checklists

## Screen state matrix (no state, no ship)

Every screen that shows data ships **all** of: `empty` (first-run, with an invitation to act,
not an apology) · `loading` (skeleton over spinner when structure is known) · `populated` ·
`error` (what happened + what to do, retry affordance). Screens depending on network or
sensors add `offline` / `permission-denied`. A design that only shows the happy populated
state is incomplete — flag it.

## Platform conventions

**iOS**: tab bar for top-level navigation (3–5 tabs); back is a left-edge swipe plus a
top-left chevron — never a bottom "back" button; sheets for scoped tasks, full screens for
journeys; destructive confirmations via action sheet anchored to the trigger; haptics carry
meaning (see `interaction-design`), don't fire decoratively.

**Android** (when in scope): system back must always work and never lose data; bottom nav or
nav drawer per Material; FAB only for THE primary action of a screen.

Don't invent hybrid patterns: pick the platform's native idiom for navigation, pickers,
dialogs, and share surfaces.

## Action hierarchy

- **One primary action per screen.** Everything else is secondary/ghost.
- Destructive actions are protected: confirm dialog for rare actions, hold-to-confirm for
  actions that must stay fast but safe, undo-window when the cost of confirmation is too high.
- Progressive disclosure: advanced options collapsed by default; never make the common case
  pay for the rare one.
- Irreversible + frequent is a design smell — redesign so it's one or the other.

## Forms

- Validate inline on blur, not only on submit; errors sit next to their field.
- **Never clear user input on error.** Never.
- Label above field; placeholder is an example of valid input, not a label.
- Every field earns its place: each removed field measurably raises completion.

## Accessibility (minimum bar, checked against real values)

- Contrast AA: 4.5:1 body text, 3:1 large text/icons — computed from token values.
- Touch targets ≥ 44×44pt (visual size may be smaller with an extended hit area — annotate it).
- Dynamic Type: layouts survive +2 text sizes; numbers-as-hero screens define how they scale.
- Reduce Motion: every meaningful animation has a reduced variant (cross-fade instead of
  movement); specified per interaction in `interactions.md`.
- Icon-only controls carry a VoiceOver/content-description label.

## UI copy

- Sentence case everywhere; CTAs are verb-first, 1–3 words ("Start run", not "Click here to
  begin").
- Errors: what happened + what to do, one sentence, no error codes at the user.
- Empty states invite ("Plan your first run"), never apologize ("No data yet :(").
- No "please", no "successfully", no exclamation marks in system copy.
