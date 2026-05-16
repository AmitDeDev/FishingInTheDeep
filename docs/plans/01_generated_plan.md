# Fishing In The Deep — Technical Implementation Plan

## 1. Core Systems and Architecture

### Systems Overview

| System | Responsibility |
|--------|---------------|
| **Game State Machine** | Controls transitions between screens/phases |
| **Render Engine** | Canvas 2D drawing loop with pseudo-3D visuals |
| **Input Handler** | Click/touch events for gauge, hook dragging |
| **Gauge System** | Pendulum animation, depth calculation on stop |
| **Hook & Line System** | Descent/ascent movement, camera follow, drag controls |
| **Fish Spawner** | Places fish at depth layers, handles collision with hook |
| **Economy System** | Coins, upgrades, fish-to-coin conversion |
| **UI Overlay** | Coin counter, fish counter with fill bar, upgrade buttons, summaries |
| **Animation/FX System** | Tweens, particle bursts, bounce numbers, title popups |

### Game State Flow

```
UPGRADE_SCREEN → GAUGE → DESCENDING → FISHING → ASCENDING → SUMMARY → [loop 3 rounds] → END_SCREEN
```

Detailed states:

1. **UPGRADE_SCREEN** — Show coin counter, upgrade buttons (max fish, max depth), Play button with gauge.
2. **GAUGE** — Pendulum pointer animates; player clicks to stop it; depth is calculated.
3. **DESCENDING** — Hook drops to target depth; camera follows; fish counter replaces coin counter. On round 1: pause 2s at bottom with tutorial text.
4. **FISHING** — Hook moves upward; player drags hook left/right to catch fish. Fish counter + fill bar update on catch.
5. **ASCENDING** — If max fish caught, hook speeds up to surface (no more catching). Fish counter swaps back to coin counter just before surface.
6. **SUMMARY** — Fish are tallied one-by-one with bounce animations, rarity titles, coin bursts. Total shown with counter animation. Coins fly to coin counter. Game resets for next round.
7. **END_SCREEN** — After round 3 summary, show final score/total earnings.

### Architecture (single file)

```
index.html
├── <style> — All CSS (minimal, mostly for overlays)
├── <canvas> — Main game rendering
├── <div> overlays — UI elements (counters, buttons, titles)
└── <script>
    ├── Constants & Config
    ├── State object
    ├── Systems (gauge, hook, fish, economy, fx)
    ├── Render functions
    ├── Update loop (requestAnimationFrame)
    └── Input handlers
```

---

## 2. Gameplay Flow

### Initial Screen (UPGRADE_SCREEN)
- Canvas shows: water surface at top, fisherman on dock/boat (simple colorful 2D shape with pseudo-3D shading).
- **Coin counter** displayed (starts at 100).
- Two upgrade buttons visible:
  - **Max Fish**: 6 → 7 (50), 8 (100), 9 (200), 10 (400)
  - **Max Depth**: 5m → 10m (50), 15m (75), 20m (150), 30m (300)
- **Play button** with gauge indicator (pendulum pointer swinging min↔max with ease-in-out).

### Gauge Interaction
- Pointer oscillates between min and max on the gauge bar.
- Min depth = maxDepth − 2m; max depth = current maxDepth upgrade level.
- Player clicks Play to lock the pointer; the stopped position determines hook depth.

### Descent Phase
- Hook drops into water. Camera scrolls down following the hook.
- Coin counter is replaced by fish counter (0/maxFish) with fill bar.
- Water gets darker with depth (gradient layers).
- Fish are visible at various depths as hook passes.
- At target depth: hook pauses briefly (0.5–1s).
  - **Round 1 only**: 2-second pause with tutorial overlay: "Move the hook to catch fish".

### Fishing Phase (Ascent with Control)
- Hook begins moving upward at steady pace.
- Player drags/moves hook left and right (mouse/touch X-axis).
- Fish swim at various horizontal positions; hook collision = catch.
- On catch:
  - Bouncy "+value" number pops from fish.
  - Fish attaches to hook visually.
  - Fish counter increments; fill bar updates.
  - Rare fish show title popup ("Rare!", "Amazing!", "Legendary!").
- If fish counter reaches maxFish: hook speeds up to surface, remaining fish ignored.

### Pre-Surface Transition
- Just before hook reaches surface: fish counter replaced by coin counter.

### Summary Phase
- Fish are displayed one-by-one with jump animation.
- Each fish shows its coin value and any rarity title.
- Coin burst particle effect per fish.
- After all fish animated: total sum displayed with counting animation.
- Coins fly from total to coin counter (counter updates).
- Play button reappears for next round.

### End Screen (after 3 rounds)
- Final total displayed.
- "Game Over" or celebratory message.
- Option to replay (optional, not in PRD but clean UX).

---

## 3. Entities and Data

### Game State

```
state = {
  phase: string,          // current game state
  round: number,          // 1–3
  coins: number,          // player currency (starts 100)
  maxFish: number,        // current max fish capacity (starts 6)
  maxDepth: number,       // current max depth in meters (starts 5)
  selectedDepth: number,  // depth chosen via gauge this throw
  caughtFish: [],         // array of caught fish this throw
  totalEarnings: number   // cumulative across all rounds
}
```

### Fish Entity

```
fish = {
  x: number,             // horizontal position
  y: number,             // depth position (world coords)
  type: string,          // common, rare, amazing, legendary
  value: number,         // coin reward
  size: number,          // visual size
  color: string,         // fill color
  speed: number,         // horizontal swim speed
  direction: number,     // +1 or -1
  caught: boolean        // whether hooked
}
```

### Hook Entity

```
hook = {
  x: number,            // horizontal position (player-controlled during fishing)
  y: number,            // vertical position (world coords)
  targetDepth: number,  // where it stops descending
  speed: number,        // current vertical speed
  fishOnHook: []        // references to caught fish (for visual stacking)
}
```

### Upgrade Definitions

```
upgrades = {
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

### Fish Type Distribution (by depth)

| Depth Range | Common % | Rare % | Amazing % | Legendary % |
|-------------|----------|--------|-----------|-------------|
| 0–5m        | 80       | 15     | 4         | 1           |
| 5–15m       | 60       | 25     | 10        | 5           |
| 15–30m      | 40       | 30     | 18        | 12          |

### Gauge

```
gauge = {
  pointerPosition: number,  // 0.0–1.0 normalized
  direction: number,        // +1 or -1
  speed: number,            // oscillation speed
  running: boolean          // whether pointer is animating
}
```

### Camera

```
camera = {
  y: number   // vertical offset for scrolling the world
}
```

---

## 4. Step-by-Step Implementation Plan

### Step 1: Scaffold & Render Loop
- Create `index.html` with canvas, basic CSS, and a `requestAnimationFrame` loop.
- Draw a colored gradient background (sky + water).
- **Test**: File opens in browser, shows animated water gradient.

### Step 2: Game State Machine
- Implement state transitions (UPGRADE_SCREEN → GAUGE → DESCENDING → FISHING → ASCENDING → SUMMARY → END_SCREEN).
- Wire a simple phase indicator on screen.
- **Test**: Can manually trigger state changes; UI reflects current phase.

### Step 3: Upgrade Screen & Economy
- Draw coin counter, two upgrade buttons with costs.
- Implement purchase logic (deduct coins, advance upgrade level, grey out unaffordable).
- Draw the fisherman as a colorful pseudo-3D shape on a dock/boat.
- **Test**: Start with 100 coins, buy an upgrade, coins deducted, button updates to next tier.

### Step 4: Gauge System
- Draw the gauge bar (min–max) with a pointer.
- Animate pointer in pendulum motion (sinusoidal with ease-in-out).
- On click: stop pointer, calculate selectedDepth from pointer position.
- Min depth = maxDepth − 2, mapped linearly.
- **Test**: Pointer swings smoothly, clicking locks it, depth value is correct.

### Step 5: Hook Descent & Camera
- Spawn hook at surface; move it downward to selectedDepth.
- Camera follows hook (scroll world Y offset).
- Water darkens with depth (layered gradient bands).
- Draw the hook + line as a simple colorful shape.
- Swap coin counter → fish counter (0/maxFish) with fill bar.
- **Test**: After gauge stop, hook descends smoothly, camera follows, darkening visible.

### Step 6: Fish Spawner
- Populate fish entities between surface and maxDepth at random positions.
- Fish swim left/right at varying speeds.
- Use depth-based rarity distribution for type/value assignment.
- Draw fish as colorful pseudo-3D shapes (ellipses with gradient fills, eye dots, tail fins).
- **Test**: Fish visible at various depths as camera scrolls down; fish move horizontally.

### Step 7: Fishing Phase (Ascent + Catching)
- After brief pause at bottom (+ tutorial on round 1), hook ascends.
- Player drags hook horizontally (mousemove / touchmove on X-axis).
- Collision detection: hook hitbox vs fish hitbox.
- On catch: fish marked caught, attached to hook visually, counter increments, fill bar updates.
- Bounce "+value" number effect. Rarity title popup for rare+.
- If caughtFish.length >= maxFish: speed up ascent, disable catching.
- **Test**: Can drag hook, catch fish, see counter update, hook speeds up at max.

### Step 8: Summary Phase
- After hook reaches surface: swap fish counter → coin counter.
- Animate each caught fish one-by-one: jump animation, show value, rarity title, coin burst particles.
- Show total with counting animation.
- Coin-fly effect to coin counter; counter updates.
- After summary, if round < 3: return to UPGRADE_SCREEN. If round == 3: go to END_SCREEN.
- **Test**: Full summary plays out, coins awarded, next round begins or game ends.

### Step 9: End Screen
- Display final total earnings with celebratory text/effects.
- Optionally show a "Play Again" button to reset all state.
- **Test**: After 3 rounds, end screen shows with correct total.

### Step 10: Juice & Polish
- Add particle effects (coin bursts, water splashes, catch sparkles).
- Ease all transitions (fade between phases).
- Add subtle screen shake on big catches.
- Make fish colors vibrant and varied per rarity.
- Add shadow/outline pseudo-3D effects to fisherman, hook, fish.
- Ensure the gauge feels satisfying (slight bounce on stop).
- **Test**: Full playthrough feels fun, juicy, and complete. All 3 rounds work end-to-end.

### Step 11: Final QA
- Verify all PRD requirements are met (3 rounds, upgrade costs, gauge min/max logic, fish counter swap timing, tutorial on first round, rarity titles, speed-up on max fish).
- Test edge cases: can't buy unaffordable upgrades, 0 fish caught in a round, all fish caught instantly.
- Test on different screen sizes (responsive canvas scaling).
- **Done**: Single `index.html` file, double-click to play.
