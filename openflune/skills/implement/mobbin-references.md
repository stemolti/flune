# Mobbin Reference Gathering

Read this file only when `$MOBBIN_MODE` is `true` and `hasPlanFile` is `false`, from the Mobbin Reference Gathering step of the implement pipeline.

Mobbin's MCP server surfaces real-world, shipped UI screens and flows so the implementation can be grounded in patterns that ship in production apps — without a manual design phase (no Pencil, no DESIGN.md). It is a **paid** (Mobbin Pro/Team/Enterprise), OAuth-authenticated remote server. Read `${CLAUDE_PLUGIN_ROOT}/docs/mobbin.md` before proceeding — it covers setup, auth, rate limits, and prompting best practices.

All steps run in the main agent. Store the outcome as `$MOBBIN_REFERENCES`.

## Step A — Paid-feature Gate

Read `mobbin.enabled` from `.claude/config.json` (already loaded in the Context section). Do **not** check `pencil.enabled` — this flow exists precisely for work that skips the Pencil design phase.

- **If `mobbin.enabled` is `true`** → proceed to Step B.
- **If `mobbin` is absent or `mobbin.enabled` is not `true`** → Mobbin is not enabled for this project. Tell the user:
  > "`--mobbin` requires a paid Mobbin plan (Pro/Team/Enterprise) and one-time setup. Enable it with `/openflune:configure` (turn on Mobbin design references), then authenticate with:
  > `claude mcp add mobbin --scope user --transport http https://api.mobbin.com/mcp`
  > followed by `/mcp` → **Authenticate**."
  Then ask via `AskUserQuestion`:
  > "How do you want to proceed?"
  Options:
  - **"Continue without Mobbin"** → set `$MOBBIN_MODE = false` and return to the pipeline (Label "Working").
  - **"Stop so I can set up Mobbin"** → **Stop.** The user re-runs `/openflune:implement --mobbin <ticket-id | description>` after configuring.

## Step B — Connectivity & Auth Check

Probe the Mobbin server with one lightweight `mcp__mobbin` call (list/probe the available tools; use whichever search/discovery tool the server advertises with a minimal query). This both verifies the connection and discovers the exact tool names at runtime (Mobbin does not publish stable tool names — never hardcode them).

- **If the probe succeeds** → note the discovered tool name(s) and proceed to Step C.
- **If the probe fails** (server not connected, or `mcp__mobbin` tools unavailable / unauthorized) → the Mobbin MCP server is not connected or not authenticated. Tell the user:
  > "The Mobbin MCP server isn't connected or authenticated. Run:
  > `claude mcp add mobbin --scope user --transport http https://api.mobbin.com/mcp`
  > then `/mcp` → **Authenticate** (a browser window opens to sign in to Mobbin). Once it shows `mobbin: connected`, re-run `/openflune:implement --mobbin <ticket>`."
  Then **Stop.** Do not continue the pipeline — the user must authenticate and re-run.

## Step C — Query Mobbin

Build a **context-rich, specific** natural-language query. Unlike the design skill, there is no design-classification phase — derive the inputs from what this pipeline already has:

- **Feature area**: the ticket title and the context-gatherer digest's summary bullets (ticket mode), or the task description (ticketless mode).
- **UI surface type**, inferred from the ticket/description: screen, form/wizard, dashboard, component, landing page, etc.
- **Platform and framework**: the project's stack from `.claude/config.json` (for monorepos, the affected project's stack); web vs. mobile.

Prefer concrete product language over generic terms (e.g., "mobile onboarding flow with progressive step indicators for a fintech KYC screen" beats "onboarding screen").

Respect Mobbin's rate limit (**60 requests / 60 seconds per user**):
- Issue at most a **couple of batched queries** — do not loop tool calls tightly.
- If a call returns HTTP `429`, read the `Retry-After` value and wait that many seconds before a single retry; if it fails again, tell the user Mobbin is rate-limited and offer (via `AskUserQuestion`) to continue without Mobbin.

## Step D — Present References

Summarize the returned references — for each, the app/screen name, the pattern it illustrates, and its Mobbin link. Then present via `AskUserQuestion` (multiSelect=true):

> "Mobbin returned these real-world references for [task]. Which patterns should the implementation follow? (Links included so you can review them yourself.)"

Options: one per returned reference (app/screen + one-line pattern note), plus **"None — proceed without references"**.

Store the user's selected references (name, pattern, link) as `$MOBBIN_REFERENCES` for the planner delegation and plan-file persistence in Phase 1. If the user picks "None", set `$MOBBIN_REFERENCES` to empty — the plan file then gets no `mobbin` front matter field and no `## Design References (Mobbin)` section.
