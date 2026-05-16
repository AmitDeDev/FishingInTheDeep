# Fishing In The Deep — Revised Technical Implementation Plan

## 1. Core Systems and Architecture

### Systems Overview

| System | Responsibility |
|--------|---------------|
| **Game State Machine** | Manages phase transitions through the game loop |
| **Render Engine** | Canvas 2D drawing with pseudo-3D colorful visuals |
| **Input Handler** | Pointer events (mouse/touch) for play click and hook dragging |
| **Gauge System** | Min-max-min pendulum, depth calculation on stop |
| **Hook & Line System** | Descent, pause, ascent, fast return, drag control |
| **Fish Spawner** | Places fish based on selectedDepth, rarity by depth |
| **Economy System** | Coins, upgrade purchases, fish-to-coin conversion |
| **Counter System** | Coin/fish counter visibility swapping with precise timing |
| **Camera System** | Follows hook vertically through the water |
| **FX System** | Particles, bounce numbers, rarity titles, coin bursts |

### Game State Machine

```
READY → DESCENDING → PAUSE_AT_DEPTH → ASCENDING_CATCH → [FAST_RETURN] → SUMMARY_FISH → SUMMARY_TOTAL → READY or END_SCREEN
```

| State | Description |
|-------|-------------|
| **READY** | Gauge pointer is already swinging. Upgrade buttons visible. Coin counter visible. One click on Play stops gauge, calculates depth, transitions immediately to DESCENDING. |
| **DESCENDING** | Hook drops to selectedDepth. Camera follows. Fish counter replaces coin counter. Fish are spawned for this throw. |
| **PAUSE_AT_DEPTH** | Hook holds at selectedDepth. Round 1: 2-second pause with tutorial text "Move the hook to catch fish". Rounds 2–3: 0.5–1 second pause. |
| **ASCENDING_CATCH** | Hook rises toward surface. Player drags hook left/right to catch fish. Fish counter updates on each catch. If caughtFish.length == maxFish → transition to FAST_RETURN. |
| **FAST_RETURN** | Hook speeds to surface. Catching disabled. Caught fish remain attached. Entered only if max fish reached during ASCENDING_CATCH. |
| **SUMMARY_FISH** | Fish jump one-by-one with value popups, rarity titles, and coin burst effects. |
| **SUMMARY_TOTAL** | Total sum displays with counting animation. Coins fly from total to coin counter. Counter updates. |
| **END_SCREEN** | After round 3 summary completes. Shows final earnings and celebratory visuals. |

Note: If max fish is NOT reached, hook simply rises to surface normally during ASCENDING_CATCH (no FAST_RETURN state needed), then transitions to SUMMARY_FISH.

### Counter Visibility Rules

| Phase | Coin Counter | Fish Counter |
|-------|-------------|--------------|
| READY | Visible | Hidden |
| DESCENDING | Hidden | Visible (starts at 0/maxFish) |
| PAUSE_AT_DEPTH | Hidden | Visible |
| ASCENDING_CATCH | Hidden | Visible (updates on catch) |
| FAST_RETURN | Hidden | Visible |
| Just before surface | Visible | Hidden (swap happens just before hook arrives) |
| SUMMARY_FISH | Visible | Hidden |
| SUMMARY_TOTAL | Visible (updating) | Hidden |
| END_SCREEN | Visible (final) | Hidden |

### Architecture (single index.html)

```
index.html
├── <style> — Minimal CSS for overlay positioning
├── <canvas id="game"> — All game rendering
├── <div> overlays — Coin counter, fish counter, upgrade buttons, titles
└── <script>
    ├── CONFIG — Constants, upgrade tables, rarity tables
    ├── State — Single state object
    ├── Gauge module
    ├── Hook module
    ├── Fish module (spawning + collision)
    ├── Economy module
    ├── FX module (particles, tweens)
    ├── Render module (draw functions per entity type)
    ├── Update loop (requestAnimationFrame)
    └── Input handlers (pointerdown, pointermove, pointerup)
```

---

## 2. Gameplay Flow

### READY State

The player sees:
- Colorful scene: sky, water surface, fisherman on dock (pseudo-3D 2D shapes with gradients/shadows).
- **Coin counter** in upper area showing current coins.
- **Two upgrade buttons**:
  - Max Fish — shows next tier and cost. Greyed out if unaffordable.
  - Max Depth — shows next tier and cost. Greyed out if unaffordable.
- **Play button** with an integrated gauge bar.
- **Gauge pointer** is already animating: swinging in a pendulum motion (ease-in-out at edges).

The gauge layout is **min–max–min**:
- Left edge = minDepth (maxDepth − 2m).
- Center = maxDepth.
- Right edge = minDepth (maxDepth − 2m).
- selectedDepth is determined by how close the pointer is to center when stopped.

**One click** on the Play button:
1. Stops the gauge pointer.
2. Calculates selectedDepth based on pointer position (distance from center maps linearly to depth between minDepth and maxDepth).
3. Swaps coin counter → fish counter.
4. Begins hook descent immediately.

No two-click flow. Gauge is always animating in READY; click = commit.

### DESCENDING

- Hook drops from surface toward selectedDepth.
- Camera scrolls down, following the hook.
- Water darkens in bands as depth increases.
- Fish are visible at their spawn depths as the hook passes them.
- Fish counter shows 0/maxFish with a fill bar (empty).

### PAUSE_AT_DEPTH

- Hook reaches selectedDepth and holds still.
- **Round 1**: Pause for 2 seconds. Tutorial text appears: "Move the hook to catch fish". Text fades out, then hook begins ascent.
- **Rounds 2–3**: Pause for 0.5–1 second, then hook begins ascent.

### ASCENDING_CATCH

- Hook moves upward at a steady pace.
- Player controls hook's horizontal position by dragging/moving pointer left-right.
- Fish swim at various horizontal positions; collision with hook = catch.
- On catch:
  - Fish visually attaches to hook (stacks below previous catches).
  - Bouncy "+value" number pops up from the fish.
  - Fish counter increments; fill bar updates proportionally.
  - If fish is rare/amazing/legendary: title popup appears ("Rare!", "Amazing!", "Legendary!").
- If `caughtFish.length == maxFish`:
  - Immediately transition to FAST_RETURN.
- Otherwise, hook continues rising to surface normally.

### FAST_RETURN (conditional)

- Triggered only when max fish is reached.
- Hook speed increases significantly.
- Catching is disabled — hook passes through remaining fish.
- Caught fish remain visually attached to hook.
- Fish counter remains visible during fast return.
- Just before hook reaches surface: fish counter swaps back to coin counter.

### Surface Arrival → SUMMARY_FISH

- Coin counter reappears just before hook + fish arrive at surface.
- Transition to SUMMARY_FISH.

### SUMMARY_FISH

- Each caught fish is presented one-by-one:
  - Fish performs a jumping/bouncing animation.
  - Its coin value is displayed.
  - If it has a rarity title (Rare!, Amazing!, Legendary!), the title appears.
  - Coin burst particle effect fires from the fish.
- After all fish have been animated, transition to SUMMARY_TOTAL.

### SUMMARY_TOTAL

- Total earnings for this throw displayed with a counting-up animation.
- Coins fly from the total number to the coin counter.
- Coin counter updates to new total.
- If round < 3: return to READY for next round (Play button reappears, gauge resumes swinging).
- If round == 3: transition to END_SCREEN after summary completes.

### END_SCREEN

- Full summary for round 3 plays out first (SUMMARY_FISH + SUMMARY_TOTAL).
- Then final screen displays total earnings across all 3 rounds.
- Celebratory visual effects.

---

## 3. Entities and Data

### Game State

```
state = {
  phase: string,            // READY, DESCENDING, PAUSE_AT_DEPTH, ASCENDING_CATCH, FAST_RETURN, SUMMARY_FISH, SUMMARY_TOTAL, END_SCREEN
  round: number,            // 1, 2, or 3
  coins: number,            // current coins (starts 100)
  maxFish: number,          // current upgrade level (starts 6)
  maxDepth: number,         // current upgrade level in meters (starts 5)
  selectedDepth: number,    // depth chosen by gauge this throw
  caughtFish: [],           // fish caught this throw
  totalEarnings: number,    // cumulative coins earned across all rounds
  pauseTimer: number        // countdown for PAUSE_AT_DEPTH duration
}
```

### Gauge

```
gauge = {
  position: number,     // 0.0 to 1.0 (0 = left edge, 0.5 = center, 1.0 = right edge)
  direction: number,    // +1 or -1
  speed: number,        // base oscillation speed
  running: boolean      // true in READY, false once clicked
}
```

Depth calculation:
```
distanceFromCenter = abs(gauge.position - 0.5) * 2    // 0.0 at center, 1.0 at edges
selectedDepth = maxDepth - (distanceFromCenter * 2)   // center = maxDepth, edges = maxDepth - 2
```

### Hook

```
hook = {
  x: number,            // horizontal position (player-controlled in ASCENDING_CATCH)
  y: number,            // vertical world position
  speed: number,        // current vertical movement speed
  targetDepth: number,  // selectedDepth for this throw
  fishAttached: []      // visual references to caught fish for stacking display
}
```

### Fish Entity

```
fish = {
  x: number,            // horizontal position
  y: number,            // depth in world coords
  type: string,         // "common", "rare", "amazing", "legendary"
  value: number,        // coin reward
  size: number,         // visual radius
  color: string,        // primary fill color
  accentColor: string,  // gradient/detail color
  swimSpeed: number,    // horizontal movement speed
  direction: number,    // +1 or -1
  caught: boolean       // true once hooked
}
```

### Camera

```
camera = {
  y: number             // world Y offset for vertical scrolling
}
```

### Upgrade Tables

```
UPGRADES = {
  maxFish: [
    { level: 6, cost: 0 },
    { level: 7, cost: 50 },
    { level: 8, cost: 100 },
    { level: 9, cost: 200 },
    { level: 10, cost: 400 }
  ],
  maxDepth: [
    { level: 5, cost: 0 },
    { level: 10, cost: 50 },
    { level: 15, cost: 75 },
    { level: 20, cost: 150 },
    { level: 30, cost: 300 }
  ]
}
```

### Rarity Distribution (by selectedDepth)

| selectedDepth Range | Common % | Rare % | Amazing % | Legendary % |
|---------------------|----------|--------|-----------|-------------|
| 3–5m               | 80       | 15     | 4         | 1           |
| 6–15m              | 60       | 25     | 10        | 5           |
| 16–30m             | 40       | 28     | 19        | 13          |

### Fish Value by Rarity

| Rarity | Coin Value Range |
|--------|-----------------|
| Common | 5–10 |
| Rare | 15–25 |
| Amazing | 30–50 |
| Legendary | 60–100 |

### Fish Spawning Rules

- Fish are spawned when entering DESCENDING, based on selectedDepth for that throw.
- Spawn count: at least `maxFish * 2` fish between surface and selectedDepth.
- Fish distributed across depth layers so player always has opportunity to catch maxFish.
- Deeper selectedDepth → higher proportion of rare/amazing/legendary fish.
- Fish are placed at random X positions within the playable width.

---

## 4. Step-by-Step Implementation Plan

### Step 1: Scaffold & Render Loop

- Create `index.html` with `<canvas>`, minimal `<style>`, and `<script>`.
- Set up `requestAnimationFrame` loop with delta-time tracking.
- Draw a colorful layered background: sky gradient, water surface line, water gradient (lighter at top, darker below).
- Implement basic canvas resize handling.
- **Test**: Double-click file → browser shows colorful sky + water background, no errors in console.

### Step 2: State Machine & Phase Rendering

- Implement state object with `phase` field.
- Create an `update(dt)` function that switches on `state.phase`.
- Create a `render()` function that draws phase-appropriate visuals.
- Draw a temporary phase label on screen for debugging.
- **Test**: Can set phase to any value; render loop shows correct phase label.

### Step 3: READY Screen — Fisherman, Coins, Upgrades

- Draw fisherman on dock as a colorful pseudo-3D shape (rounded body, hat, rod — using gradients, shadows, outlines).
- Draw coin counter (top area, coin icon + number).
- Draw two upgrade buttons with current level, next tier, and cost.
- Implement purchase logic: click button → if affordable, deduct coins, advance level, update button display. Grey out if unaffordable or maxed.
- **Test**: Start with 100 coins, buy Max Depth (50 coins), counter shows 50, button advances to next tier.

### Step 4: Gauge System

- Draw gauge bar on/near the Play button: left-center-right layout representing min-max-min.
- Visual markers showing min and max depth zones.
- Animate pointer: oscillates 0→1→0 using sinusoidal motion with ease-in-out at ends.
- Pointer starts swinging immediately in READY state.
- **One click** on Play area:
  1. `gauge.running = false` (pointer stops).
  2. Calculate `selectedDepth` from pointer position (center = maxDepth, edges = maxDepth − 2).
  3. Transition to DESCENDING.
- **Test**: Pointer swings smoothly. Clicking stops it. Clicking near center gives high depth, near edges gives low depth. Values are correct.

### Step 5: DESCENDING — Hook, Camera, Counter Swap

- On entering DESCENDING: swap coin counter → fish counter (0/maxFish with fill bar).
- Spawn hook at surface. Move hook downward at steady speed toward selectedDepth.
- Camera.y follows hook.y so hook stays in view.
- Water background darkens as depth increases (render depth-based color bands).
- Draw hook + line as colorful shapes (hook arc, line up to top of screen).
- **Test**: After gauge click, coin counter disappears, fish counter appears at 0. Hook descends smoothly, camera follows, water darkens.

### Step 6: Fish Spawning

- On entering DESCENDING, spawn fish for this throw:
  - Count: `maxFish * 2` minimum (spread between surface and selectedDepth).
  - Use rarity table based on selectedDepth to assign types.
  - Random X positions within canvas width. Random Y positions distributed across depth range.
- Draw fish as pseudo-3D colorful shapes: ellipses with gradient fills, darker outline, small eye, tail fin. Color varies by rarity (common = blues/greens, rare = purple/pink, amazing = gold, legendary = rainbow/prismatic).
- Animate fish swimming horizontally (oscillate or bounce off edges).
- **Test**: Fish appear at various depths. Deeper selectedDepth shows more colorful/rare fish. Fish swim left/right smoothly.

### Step 7: PAUSE_AT_DEPTH & Tutorial

- When hook.y reaches selectedDepth, enter PAUSE_AT_DEPTH.
- Start a timer: 2 seconds for round 1, random 0.5–1s for rounds 2–3.
- Round 1: show tutorial text overlay "Move the hook to catch fish" (centered, stylized, fades in).
- When timer expires: hide tutorial text, transition to ASCENDING_CATCH.
- **Test**: Hook stops at depth. Round 1 shows tutorial for 2s. Round 2+ shows brief pause. Then hook starts ascending.

### Step 8: ASCENDING_CATCH — Hook Control & Fish Catching

- Hook moves upward at steady speed.
- Player controls hook.x via pointer events (pointermove → hook.x tracks pointer X, clamped to canvas bounds).
- Collision detection each frame: circle-circle between hook and uncaught fish.
- On collision (catch):
  - `fish.caught = true`, add to `state.caughtFish`.
  - Fish attaches to hook visually (offset below hook, stacked).
  - Fish counter increments, fill bar updates.
  - Spawn bouncy "+value" floating text at fish position.
  - If fish type is rare/amazing/legendary: spawn title popup ("Rare!", "Amazing!", "Legendary!").
- If `state.caughtFish.length >= state.maxFish`: transition to FAST_RETURN.
- If hook reaches surface without hitting max: transition directly to pre-summary (swap counter, then SUMMARY_FISH).
- **Test**: Can drag hook horizontally. Catching fish updates counter. Rarity titles appear. Max fish triggers fast return.

### Step 9: FAST_RETURN

- Increase hook vertical speed (3–4x normal ascent speed).
- Disable collision detection (no more catches).
- Caught fish remain attached to hook visually.
- Fish counter remains visible.
- When hook approaches surface (within a threshold): swap fish counter → coin counter.
- When hook reaches surface: transition to SUMMARY_FISH.
- **Test**: After max fish caught, hook zooms up ignoring remaining fish. Counter swaps near surface.

### Step 10: SUMMARY_FISH

- Display caught fish one-by-one in sequence (timed, ~1s each):
  - Fish performs a bounce/jump animation.
  - Coin value displayed next to it.
  - If rare+: rarity title appears with flair.
  - Coin burst particle effect from the fish.
- After all fish presented: transition to SUMMARY_TOTAL.
- **Test**: Each fish animates in sequence with correct values and titles. Particle effects fire.

### Step 11: SUMMARY_TOTAL & Round Progression

- Calculate total coins earned this throw (sum of all caught fish values).
- Display total with a counting-up animation (0 → total over ~1.5s).
- Animate coins flying from the total number toward the coin counter.
- Coin counter increments to new value.
- Add throw earnings to `state.totalEarnings`.
- If `state.round < 3`:
  - Increment round.
  - Reset throw state (caughtFish, selectedDepth, gauge resumes).
  - Transition to READY.
- If `state.round == 3`:
  - Transition to END_SCREEN.
- **Test**: Total counts up correctly. Coins fly to counter. Counter updates. Next round begins or game ends after round 3.

### Step 12: END_SCREEN

- Show final screen: total earnings across all 3 rounds.
- Celebratory visuals (particle shower, bold text, colorful background flash).
- Optionally: "Play Again" button that resets all state to initial values.
- **Test**: After round 3 summary, end screen displays correct cumulative total.

### Step 13: Visual Polish & Juice

- Add pseudo-3D depth to all shapes: drop shadows, gradient fills, highlights, outlines.
- Particle effects: water splash when hook enters/exits water, sparkles on catch, coin shower in summary.
- Smooth easing on all transitions (fade phases in/out).
- Make gauge feel satisfying (slight overshoot bounce when pointer stops).
- Fish have subtle bobbing animation even while swimming.
- Hook line sways slightly during descent.
- Screen shake on legendary catches.
- Vibrant color palette: warm golds for coins, cool blues for water, bright accents for rare fish.
- **Test**: Full playthrough feels colorful, juicy, and polished. No jarring transitions.

### Step 14: Final QA & Edge Cases

- Complete 3-round playthrough end-to-end.
- Verify all items in the PRD Verification Checklist below.
- Test with 0 fish caught in a round (summary still works, 0 coins awarded).
- Test buying all upgrades (buttons show maxed state).
- Test gauge at exact edges and exact center.
- Test on different browser window sizes.
- Ensure no external dependencies (file works offline, double-click to open).

---

## 5. Final PRD Verification Checklist

| # | PRD Requirement | Verification |
|---|----------------|--------------|
| 1 | Player starts with 100 coins | Coin counter shows 100 on first load |
| 2 | Max fish starts at 6 | Fish counter denominator shows 6 initially |
| 3 | Max depth starts at 5m | Gauge center represents 5m depth |
| 4 | Max fish upgrade costs: 50, 100, 200, 400 | Each tier costs correct amount |
| 5 | Max depth upgrade costs: 50, 75, 150, 300 | Each tier costs correct amount |
| 6 | Gauge is min-max-min with pendulum motion | Pointer swings edge→center→edge with easing |
| 7 | Min depth = max depth − 2m | Edges of gauge always equal maxDepth − 2 |
| 8 | 3 rounds total, then end screen | Game ends after round 3 summary |
| 9 | Coin counter visible initially, fish counter hidden | READY shows only coin counter |
| 10 | Fish counter replaces coin counter when fishing starts | Swap happens at start of DESCENDING |
| 11 | Fish counter has fill bar | Bar fills proportionally with catches |
| 12 | Hook descends to selectedDepth | Hook stops at gauge-determined depth |
| 13 | Camera follows hook | View scrolls with hook during descent/ascent |
| 14 | Hook pauses at depth (0.5–1s normal, 2s round 1) | Timer-controlled pause per round |
| 15 | Round 1 tutorial: "move the hook to catch fish" | Text appears during 2s pause, then disappears |
| 16 | Hook ascends, player drags side to side | Horizontal pointer control during ASCENDING_CATCH |
| 17 | Fish caught shows bouncy value amount | "+N" floats up from caught fish |
| 18 | Rare/amazing/legendary titles shown | Popup text on qualifying catches |
| 19 | Higher depth = better rarity chance | Rarity table scales with selectedDepth |
| 20 | Caught fish attach to hook visually | Fish stack below hook during ascent |
| 21 | Max fish reached → hook speeds to surface | FAST_RETURN state triggered |
| 22 | No more fish caught during speed-up | Collision disabled in FAST_RETURN |
| 23 | Fish counter → coin counter swap just before surface | Swap at threshold before hook arrives |
| 24 | Summary: fish jump with values and titles | SUMMARY_FISH animates each fish |
| 25 | Summary: coin burst effect per fish | Particles fire from each fish |
| 26 | Summary: total count-up animation | Number increments visually |
| 27 | Summary: coins fly to coin counter | Animated coin flight to counter |
| 28 | Coin counter updates after flight | Counter reflects new total |
| 29 | Play button reappears for next round | READY state restores after summary (rounds 1–2) |
| 30 | End screen after round 3 | Shows only after round 3 SUMMARY_TOTAL completes |
| 31 | Pseudo-3D visuals, colorful, fun | Gradients, shadows, outlines, vibrant colors |
| 32 | Single index.html, no dependencies | File runs standalone in browser |
