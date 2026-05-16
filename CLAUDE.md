# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Goal

A single-file playable called "Fishing In The Deep" — a casual fishing game where the player earns coins by catching fish across 3 rounds. Process documentation is part of the submission.

## Important Files

| File | Purpose |
|------|---------|
| `1_PRD_Fishing_In_The_Deep.md` | Authoritative game design spec. Do not modify. |
| `docs/plans/02_generated_plan_revised.md` | Chosen technical implementation plan. |
| `docs/plans/03_plan_selection_notes.md` | Plan selection reasoning. |
| `index.html` | Final playable (single file, not yet created). |

## Hard Technical Constraints

- Final output: one `index.html` file, runs by double-clicking in browser.
- Vanilla HTML + CSS + plain JavaScript inside that single file.
- Canvas 2D rendering with colorful pseudo-3D visuals (gradients, shadows, outlines, depth scaling).
- No React, Vue, Angular, Svelte, npm, bundlers, build tools, modules, imports, CDNs, Three.js, WebGL, or external frameworks.
- The PRD is authoritative and must not be modified.

## Workflow Rules

1. Read the PRD and chosen plan (`02_generated_plan_revised.md`) before implementation.
2. Do not write game code until explicitly asked.
3. Explain planned changes before editing files.
4. Summarize changes after editing.
5. Keep git commits meaningful and descriptive.
6. Preserve the initial AI-generated implementation as a distinct commit or tag before iterating.

## Key PRD Requirements (must not be missed)

- 100 starting coins.
- 3 fishing rounds, then end screen.
- Upgrades — max fish: 6→7(50), 8(100), 9(200), 10(400). Max depth: 5m→10m(50), 15m(75), 20m(150), 30m(300).
- Gauge is min-max-min (edges = minDepth, center = maxDepth, minDepth = maxDepth − 2m).
- One-click Play: gauge is already swinging; click stops it, calculates depth, starts descent.
- Coin counter visible initially; fish counter (with fill bar) replaces it when fishing starts.
- Hook descends to selectedDepth (not maxDepth).
- First-round pause: 2 seconds with tutorial "move the hook to catch fish". Rounds 2–3: 0.5–1s.
- Hook ascends with player drag control (left/right).
- Caught fish attach to hook visually.
- Max fish reached → hook speeds to surface, catching disabled, fish stay attached.
- Fish counter swaps back to coin counter just before hook reaches surface.
- Summary: fish jump one-by-one with values/titles/coin bursts, then total count-up, coins fly to counter.
- End screen shows after round 3 summary completes.

## Testing

- Primary method: open in browser, manual playtesting.
- Use the webapp-testing skill if/when available.
- Check browser console for errors after each change.
- Verify a full 3-round playthrough works end-to-end.
