Review the current generated plan at:

docs/plans/01_generated_plan.md

Compare it carefully against the PRD file at:

1_PRD_Fishing_In_The_Deep.md

Create a revised technical implementation plan and save it as:

docs/plans/02_generated_plan_revised.md

Do not write code yet.
Do not modify the PRD.
Do not modify index.html.
Do not manually edit the first plan file.

The PRD is the authoritative source. The revised plan must fix these issues:

1. Gauge behavior

The gauge must be min-max-min, not simple min-to-max.

Clarify that:
- Left edge = minDepth.
- Center = maxDepth.
- Right edge = minDepth.
- minDepth = maxDepth - 2m.
- selectedDepth is based on distance from the center.

2. Play interaction

Avoid a two-click flow.

The gauge should already be visible and moving in the READY screen.

One Play click should:
- stop the gauge pointer,
- calculate selectedDepth,
- replace the coin counter with the fish counter,
- start hook descent.

3. State machine clarity

Replace the current flow with a clearer PRD-aligned flow:

READY
-> DESCENDING
-> PAUSE_AT_DEPTH
-> ASCENDING_CATCH
-> FAST_RETURN_TO_SURFACE, only if max fish is reached
-> SUMMARY_FISH
-> SUMMARY_TOTAL
-> READY or END_SCREEN

Make it clear that catching fish happens during ASCENDING_CATCH.

4. Counter timing

Clarify the exact counter behavior:
- Coin counter is visible initially.
- Fish counter is hidden initially.
- Fish counter replaces coin counter when fishing starts / hook descends.
- Fish counter updates with each caught fish.
- If max fish is reached deep underwater, fish counter remains visible during fast return.
- Coin counter returns just before the hook + caught fish reach the surface.

5. Fish spawning

Fix fish spawning so it is based on selectedDepth, not only maxDepth.

Clarify that:
- Fish for the current throw should spawn mostly between surface and selectedDepth.
- There should be enough catchable fish, at least maxFish * 2.
- Deeper selectedDepth should increase the chance of Rare, Amazing, and Legendary fish.

6. selectedDepth vs maxDepth

Clearly separate:
- maxDepth = maximum possible depth from upgrades.
- selectedDepth = actual depth chosen by the gauge for the current throw.

The hook descends to selectedDepth.

7. First-round tutorial

Clarify that after the hook reaches selectedDepth:
- Round 1 pauses for 2 seconds.
- Tutorial text appears: “move the hook to catch fish”.
- Then the title disappears and the hook starts ascending.
- Later rounds pause only 0.5–1 second.

8. Max fish behavior

Clarify that when caughtFish reaches maxFish:
- catching is disabled,
- hook speeds up to the surface,
- no more fish can be collected,
- already caught fish remain attached to the hook.

9. Summary and round progression

Split the summary into:
- SUMMARY_FISH: fish jump one-by-one, values/titles appear, coin bursts play.
- SUMMARY_TOTAL: total count-up, coins fly to coin counter, coin counter updates.

Round 3 should still show the full summary. Only after round 3 summary finishes should the game move to END_SCREEN.

Keep the good parts from the first plan:
- single index.html,
- Canvas 2D,
- colorful pseudo-3D visuals,
- vanilla HTML/CSS/JavaScript only,
- no frameworks, imports, modules, CDNs, build tools, Three.js, or WebGL,
- pointer input for mouse/touch,
- upgrades,
- economy,
- camera follow,
- rarity titles,
- fish attach to hook,
- bouncy value popups,
- coin burst effects,
- final QA checklist.

Keep the revised plan concise, practical, and implementation-ready.

Important: Do not simply append these notes to the old plan. Rewrite the plan cleanly so the revised version reads like one coherent implementation plan.

Required sections:
1. Core Systems and Architecture
2. Gameplay Flow
3. Entities and Data
4. Step-by-Step Implementation Plan
5. Final PRD Verification Checklist