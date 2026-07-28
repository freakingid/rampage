# Rampage Clone — STATUS

**Phase:** 1 (faithful replica)
**Last updated:** 2026-07-28 — end of Session 2
**Last session:** Session 2 — Buildings & climbing
**Next session:** Session 3 — Destruction & collapse

---

## Current state

**Playable?** Getting there — a monster walks a street of five buildings, climbs their faces, walks their roofs, and jumps between them. Nothing can be damaged yet.

**What exists:**
- `index.html` — the game. Single self-contained file, no build step, opens directly in a browser.
- `docs/design/` — Phase 1 GDD split into 12 per-system files + inferred-items list
- `docs/STATUS.md` — this file
- `docs/sessions/SESSION-PLAN.md` — ordered session list with success criteria
- `CLAUDE.md` — project rules

**How to run:** open `index.html`. Press **F / G / Enter** at the title → pick a monster with **A/D**, confirm with **F** → play.
**P1 controls:** A/D move · **W** climb up / grab a face · **S** climb down / leave a roof · **G** jump · **F** punch.
Jump-to-grab: jump toward a building and hold **W** in mid-air.
P2 (arrows + `.` `/`) and P3 (IJKL + O/P) key maps exist as data but no second monster spawns yet — `CONFIG.players.activeAtStart` is 1 until Session 10.

| System | State |
|---|---|
| Game loop & state machine | ✅ fixed timestep + interpolated render; BOOT→TITLE→CHARACTER_SELECT→PLAY, later states stubbed |
| Monster movement | ✅ walk / jump / punch anim, climb, cling, rooftop walk, jump-to-grab |
| Buildings & climbing | ✅ `Building` cell grid + hardcoded 5-building day + full ground↔face↔roof traversal |
| Destruction & collapse | ⬜ not started — cells carry state + HP, punch hitbox exists and hits nothing |
| Eating & health | ⬜ not started |
| Civilians & designated victims | ⬜ not started — victim ids are in `CONFIG.monsterTypes` |
| Military — Guardsmen, helicopters | ⬜ not started |
| Military — tanks, police, paratroopers, lightning | ⬜ not started |
| Scoring & HUD | 🟨 3-column HUD strip reserved (names only). No score, no energy bars |
| Day system & progression | ⬜ not started |
| 2-player & monster-vs-monster | 🟨 input + player records sized for 3; only P1 spawns |
| Art & audio pass | ⬜ not started — placeholder rectangles, no audio |

---

## What changed last session

Session 2 — buildings and climbing, all inside `index.html`:

- **`Building extends Entity`** (`02-movement-destruction` section). Anchored like every other entity (x = centre, y = base), so its base is always `world.groundY` and `top` is the roof line. Holds a `cells[row][col]` grid of `{ state, hp }` with `CellState = INTACT | CRACKED | HOLE | GONE`. **Nothing damages cells yet** — the model and the per-state art exist so Session 3 only has to add the damage path. Helpers: `cellAt`, `cellRect`, `cellIndexAt`, `overlapWidth`, `roofY`.
- **`CONFIG.buildings`** — cell size, cell HP, palettes, window-grid art values, and `dayLayout`: a hardcoded 5-building row (3×7, 4×5, 3×9, 5×6, 3×8 cells) with explicit gaps, filling the street exactly from `leftBound` to `rightBound`. Session 9 swaps this for `LevelData`.
- **Four-stance traversal.** `Stance` is now `{ GROUND, AIR, CLING, ROOF }` — one state machine, as the Session 1 notes asked. `Monster.update` dispatches to one handler per stance (`updateGround` / `updateAir` / `updateCling` / `updateRoof`); each is responsible for leaving the monster at a legal position. The full transition graph is documented in a comment above the enum.
- **Climbing** (§3.1): up on the street in front of a face attaches; up/down climbs; sideways traverses the face; the body is clamped flush inside the frontage. **Rooftop walking**: walk the top, step off the edge, jump between buildings. **Jump-to-grab**: hold up in mid-air near a face to attach, with a short lockout so jumping *off* a face doesn't instantly re-grab it.
- **`Game.spawnBuildings` / `startDay`** plus two spatial queries traversal actually needs — `faceTargetFor` (best-overlap grabbable face) and `roofLandingAt` (swept roof crossing). Linear scans over `game.buildings`; no spatial grid, since five buildings don't warrant one.

**The transition design, since it's the part most likely to break.** Every paired transition happens at an *identical* y on both sides — ground↔cling both at `groundY`, cling↔roof both at `building.top`. Nothing moves when a stance changes, so there is nothing to clip through or pop out of. The cost is that each boundary needs a directional guard or the two states flip-flop every tick: `CLING→ROOF` requires up *not* being overridden by down, `CLING→GROUND` requires up *not* held. Roof→cling routes through `attachToFace` because a roof lets the centre reach the very edge — half a body further out than a face allows — and without the re-clamp the monster hung off the side of the wall for a tick. (That was a real bug; the fuzz test below caught it.)

Session 1 — built `index.html` from scratch:
- **`CONFIG`** at the top of the file holds every tunable: view, loop, world, monster, monster types, art proportions, player key maps, UI layout, debug.
- **Fixed-timestep loop** (`CONFIG.loop.fixedDt` = 1/60) with an accumulator, a `maxFrameTime` clamp against the tab-out death spiral, and an interpolated render pass driven by leftover accumulator.
- **Game state machine**: `BOOT → TITLE → CHARACTER_SELECT → PLAY`; `DAY_INTRO`, `DAY_CLEAR`, `GAME_OVER` exist as stubs that draw a "SESSION 9" placeholder. All transitions go through `Game.setState`.
- **`Entity` base class** (position anchored at centre-x / feet, `prevX`/`prevY` for interpolation, `alive`) plus `EntityManager` (add/update/despawn/`ofType`).
- **`Monster extends Entity`**, parameterized by `monsterType`. One class, one stat set, all three types.
- **`InputManager`** with per-player key sets sized to `CONFIG.players.maxPlayers` (3). Held state for movement, edge-triggered state for punch/jump/confirm, consumed by exactly one fixed step. Window blur releases everything.
- **Character select** with three cards, per-player cursors, and a 3-column HUD strip where empty slots stay reserved.

**Verification.** `index.html` was executed headlessly in Node with a DOM/canvas shim (the real inline script, not a copy):
- Crossing the street left wall → right wall takes **5.6500 s at 30, 60 and 144 Hz — spread 0.000000 s**. Jump apex (38.25 px) and airborne duration (34 ticks) also identical across all three rates.
- Every state's draw path runs without throwing, including the punch-arm and punch-hitbox paths.
- Console clean — no errors or warnings at any refresh rate.
- All three monster types produce **byte-identical position/stance/animation traces** over the same scripted input, and the type table contains no stat fields.

Not done in a real browser (none available in this environment) — worth one manual open to confirm it looks right.

---

## Verification — Session 2

Same method as Session 1: the **real** inline script from `index.html` executed headlessly in Node under a DOM/canvas shim, driven tick by tick. 55 checks, all passing.

Against the SESSION-PLAN Session 2 success criteria:

| Criterion | Result |
|---|---|
| Climb a face, reach the roof, walk it, climb down the far side | ✅ verified from **every column of every building, in both directions** (32 runs) |
| Jump-to-grab attaches reliably when jumping into a face | ✅ attaches in all 8 gap trials (each street gap, jumping either way) |
| Cannot walk through buildings or float off a face | ✅ see the note below on what "walk through" means here; float-off covered by the invariant |
| Ground↔face↔roof transitions never leave the monster stuck or clipped | ✅ see below |

The stuck/clipped criterion is the one that got real effort:

- **Legality invariant, asserted after every single tick.** CLING ⇒ attached, y within `[roof, street]`, body fully inside the frontage. ROOF ⇒ attached, y exactly the roof line, centre over the frontage. GROUND/AIR ⇒ not attached, never below the street. Plus playfield bounds and finite coordinates, always. Held for **146,000+ ticks** of randomised input across 13 seeds (all four stances heavily exercised: cling 55k, air 38k, roof 22k, ground 2k ticks). This is what caught the roof-edge clamp bug described above.
- **No stuck state, exhaustively.** From every cell-aligned cling position on every building (both directly and after jumping off), and from every roof position, holding *down* returns the monster to the street. 300+ start positions, no exceptions.
- **No tunnelling.** A 4,000 px/s fall — more than a whole building per step — still lands on the roof, because the roof test is swept across the step rather than sampled.
- **A street jump can never reach a roof.** Asserted for all five buildings, and enforced structurally: `CONFIG.buildings` documents that every building must be taller than the jump apex, and the test recomputes the apex from `CONFIG` and checks the layout against it. This is what keeps the street a lane in front of the buildings rather than a set of ramps.
- **Frame-rate independence still holds** — the climb to a roof takes identical sim time at 1, 3 and 9 fixed steps per frame.
- **Every draw path runs clean** — all seven game states, all four cell damage states, all four stances, punching and not.
- **Two monsters on one building** climb independently (forward-check for Session 10).

A frame was also rasterised from the game's real draw calls to confirm it looks right: five buildings with window grids across the street, a monster standing flush on a roof, one clinging mid-face, one on the street in a gap, and hand-set CRACKED/HOLE/GONE cells rendering distinctly. Still not opened in a real browser (none in this environment) — worth one manual open.

---

## What's next

**Session 3 — Destruction & collapse.** Punch damages cells, collapse threshold, SHAKING state, rubble, fall damage, `causedBy` attribution.

Seams already in place for it:
- `Building.cells[row][col]` is `{ state, hp }` with all four `CellState` values already defined **and already drawn** — `drawCell` renders INTACT, CRACKED, HOLE and GONE distinctly today. Session 3 sets the state; it doesn't need new art.
- `Building.cellIndexAt(x, y)` and `cellRect(col, row)` map between world space and the grid — that's the bridge from `Monster.getPunchBox()` to the cell a punch lands on.
- `CONFIG.buildings.cellHp` exists (3) and is a placeholder. The collapse threshold is item #5 in `13-inferred-items.md` and is still unimplemented.
- `Monster.updateCling` and `updateRoof` already bail to AIR if their building stops being `alive` — the collapse path can despawn a building without stranding whoever was on it. Fall damage still needs adding at that point.
- `Game.buildings` is the query list; a collapsed building should be removed from it (or filtered by `alive`, which every query already does).

Success criteria are in `docs/sessions/SESSION-PLAN.md`.

---

## Open questions / things guessed at

Anything implemented from an inference rather than a confirmed source gets logged here **and** in `docs/design/13-inferred-items.md`.

| # | Item | Where | Status |
|---|---|---|---|
| 1 | Damage values, health-bar size, regen amounts | `CONFIG` | not yet implemented — placeholder pending |
| 2 | Movement speed, jump arc, punch cadence | `CONFIG.monster` | **first-pass values set in Session 1** — walk 84 px/s, jump 270 px/s vs gravity 900 (≈40 px apex, 0.6 s airtime), punch 0.20 s. All guesses; retune in Session 12 |
| 3 | Building collapse threshold | destruction | not yet implemented |
| 4 | Enemy fire rates, projectile speeds, spawn cadences | `CONFIG` | not yet implemented |
| 5 | Health meter as bar vs. descriptive labels | HUD | decided: plain bar for Phase 1 |
| 6 | Monster palettes / exact colors | `CONFIG.monsterTypes` | **placeholders picked in Session 1** (brown / green / grey) — distinct, not claimed accurate. Deferred to art pass |
| 7 | "Holding designated victim → only Guardsmen stop firing" | civilians | single-source claim, implement as written |
| 8 | Monster-vs-monster "punch forces drop" | multiplayer | implement drop-on-hit, verify later |
| 9 | **Movement during a punch** — new in Session 1 | `Monster.update` | GDD is silent. Punch **roots** the monster on the ground; airborne momentum kept. **Session 2 extended the same rule to cling and roof.** Verify |
| 10 | **Air control** — new in Session 1 | `CONFIG.monster.airControl` | 0.9 of walk speed while airborne. No documented rule. Interacts with jump-to-grab — worked out fine, gaps are crossable |
| 11 | **Climb speed** — new in Session 2 | `CONFIG.monster.climbSpeed` | 60 px/s up/down a face, vs 84 px/s walk. Undocumented; pure feel. Retune in Session 12 |
| 12 | **Sideways movement while clinging** — new in Session 2 | `CONFIG.monster.climbSideSpeed` | §3.1 only specifies up/down. Implemented as face traversal at 60 px/s, clamped inside the frontage. Punching cells across a building's width needs it — but verify the original allowed it |
| 13 | **The street is a lane in front of the buildings** — new in Session 2 | `Building` / `Monster.updateGround` | GDD never says whether a building base blocks ground movement. Implemented as a lane: the monster walks the full street past every building. Believed faithful, but it's an inference — and it's the reason every building must be taller than the jump apex |
| 14 | **Roof-edge and re-grab rules** — new in Session 2 | `CONFIG.monster` | Leaves a roof when the **centre** passes the edge; `grabLockout` 0.18 s after jumping off a face; `grabMinOverlap` 10 px to grab. All feel values, none documented |

Port decisions (not fidelity claims, recorded so they aren't mistaken for one): keyboard mapping, and the 512×384 logical playfield (MCR-3 was 512×480). Both listed in `docs/design/13-inferred-items.md`.

---

## Session log

| # | Session | Date | Outcome |
|---|---|---|---|
| 0 | Repo scaffold & design-doc split | 2026-07-28 | ✅ |
| 1 | Core skeleton | 2026-07-28 | ✅ all success criteria verified |
| 2 | Buildings & climbing | 2026-07-28 | ✅ all success criteria verified |

---

## Notes for the next session

_(Anything the next session needs to know that isn't obvious from the code — deferred decisions, known rough edges, things deliberately left unfinished.)_

- **`window.RAMPAGE`** exposes `{ CONFIG, game, GameState, Entity, EntityManager, Monster, InputManager, Stance, Building, CellState }` for console poking and for the headless harness. Keep it exported — the headless tests drive the game entirely through it.
- **The four-stance handlers each own their exit conditions.** If Session 3 adds a way to leave a surface (a building collapsing under you, knockback), add it *inside* the relevant handler and make sure the monster ends the tick at a legal position for whatever stance it lands in. The invariant the tests assert is in the Verification section above — reuse it.
- **`CONFIG.buildings.dayLayout` has a height constraint**, documented in `CONFIG`: every building must be taller than the jump apex. If Session 9's `LevelData` ever authors a short building, a monster will be able to jump from the street onto its roof, which breaks the street-as-a-lane model. Either keep the constraint or decide deliberately to model the lane some other way.
- **Rooftop walking uses `walkSpeed`, climbing uses `climbSpeed`** — deliberately different, both guesses.
- **No damage anywhere yet.** `CONFIG.buildings.cellHp` is set and every cell has `hp`, but nothing decrements it. `Monster.getPunchBox()` still hits nothing.
- **Debug readout** at the bottom of the screen (`CONFIG.debug.enabled`) is Session-1 scaffolding, including the `CROSS` stopwatch used to prove frame-rate independence. Session 8 replaces it with the real HUD; the crossing timer can go then.
- **Jump apex is ~40 px against a 46 px monster** — under one body height. It's enough to clear every gap in the current layout. Raising `CONFIG.monster.jumpSpeed` is still the right lever if jumps feel bad, but it now has a second consequence: it raises the minimum legal building height (see the point above).
- **`DAY_CLEAR` and `GAME_OVER` stubs have no `update`** — they're intentional dead ends until Session 9. Nothing transitions into them yet. `DAY_INTRO` passes straight through to `PLAY`.
- **Punch hits nothing.** `Monster.getPunchBox()` returns the hitbox and it's drawn in debug yellow; Session 3 is the first thing to test against it.
- **No spatial partitioning, no camera, no audio, no `LevelData`** — deliberately not built, since no Phase-1 mechanic needs them yet. `Game.faceTargetFor` / `roofLandingAt` are linear scans over five buildings; that's the whole spatial system and it's enough.
- Nothing was committed to git in Sessions 1 or 2.