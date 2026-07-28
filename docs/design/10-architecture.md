<!-- Source: RAMPAGE-GDD-PHASE1.md — Section 11: Technical architecture -->

## 11. Technical architecture

**Stack:** HTML5 Canvas 2D + vanilla JavaScript, **single HTML file** for Phase 1 (inline `<canvas>` + `<script>`), organized into clearly commented sections that **map 1:1 to the future file split**. No build step, no external libraries required.

### 11.1 Main loop

Fixed-timestep update with a render pass (decoupled so behavior is frame-rate independent):

```
requestAnimationFrame(frame):
  accumulate elapsed time
  while accumulator >= FIXED_DT:   // e.g. 1/60 s
      update(FIXED_DT)             // input → AI → physics → collisions → resolve
      accumulator -= FIXED_DT
  render(interpolationAlpha)       // draw current game state
```

`update` order each tick: read input → update monsters → update enemies/hazards → update projectiles → update buildings (damage/collapse) → resolve collisions → resolve eating/pickups → update score/health → check day-clear / defeat.

### 11.2 State management (game state machine)

A single top-level state enum drives which update/render runs:
`BOOT → TITLE/ATTRACT → CHARACTER_SELECT → DAY_INTRO (headline) → PLAY → DAY_CLEAR → (loop to DAY_INTRO) / GAME_OVER (with CONTINUE window)`.
Keep transitions in one place. Persist run-level data (scores, current day index, active players) across states.

### 11.3 Entity model (extensible by design)

A small base `Entity` (position, velocity, bounds, `update`, `draw`, `alive`) with subclasses. This is the main extension seam for Phase 2+ (new monster/enemy/hazard/item = new subclass + a data entry; no core rewrite):

- `Monster` — parameterized by `monsterType` (sprite set, punch anim, designated-victim id). **One class for all three** (§2).
- `Building` — a grid of destructible **cells** (per-cell HP + state), collapse logic, rubble/contents.
- `Human` — civilians, designated victims, and window-soldiers share this with a role flag (edible, harmful-on-flash, etc.).
- `Enemy` — Guardsman, Helicopter, Tank, Police, Paratrooper via a `behavior` field/strategy; Lightning as a hazard variant.
- `Projectile` — bullets, shells, thrown dynamite, bombs.
- `Vehicle` — cars/trains/boats (smashables + threats where applicable).
- `Pickup` — food, valuables, electrical items (with on/off variant), poison, cactus.

Suggested managers/modules (each = a future file section): `InputManager` (maps **per-player** key sets; built for up to 3 players from day one), `EntityManager` (spawn/update/despawn, spatial queries), `CollisionSystem`, `BuildingSystem` (damage/collapse), `SpawnDirector` (per-day enemy schedule + escalation scalars — the hook for §5.2/§7), `ScoreSystem`, `HealthSystem`, `HUD`/`Renderer`, `AudioManager`, and a `LevelData` module (**plain data array of day definitions**: building count/heights, feature flags, enemy set, difficulty multipliers — see §7 so adding cities is content-only).

### 11.4 Data-driven config (so tuning ≠ code surgery)

Centralize the **[INFERRED]** tuning values (damage per source, health totals, regen amounts, movement/jump/punch timing, spawn cadences, collapse threshold) into one `CONFIG`/balance object. This is where all §13 unknowns get dialed in without touching logic.

**Phase 1 scope:** one HTML file, fixed-timestep loop, the state machine above, the entity model, per-player input for up to 3 players (2 wired to keys now), a data-driven `LevelData` list, and a central `CONFIG`. Don't build systems no Phase-1 mechanic uses (no ECS framework, no networking, no asset pipeline).
