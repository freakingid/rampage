# Rampage Clone — STATUS

**Phase:** 1 (faithful replica)
**Last updated:** 2026-07-28 — end of Session 3
**Last session:** Session 3 — Destruction & collapse
**Next session:** Session 4 — Eating & health

---

## Current state

**Playable?** Yes, in the sense that matters — you can now knock the whole city down. A monster walks a street of five buildings, climbs their faces, walks their roofs, punches sections apart, and brings buildings down on top of itself if it doesn't jump clear. No health, no score, no enemies yet, so nothing pushes back.

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
| Destruction & collapse | ✅ punch damage from ground/face, STANDING→SHAKING→RUBBLE, roof-drop, dust puff, fall damage, `causedBy` attribution |
| Eating & health | 🟨 `Monster.applyDamage(amount, source)` is the funnel Session 4 fills in; only fall damage routes through it today, into a `damageTaken` counter |
| Civilians & designated victims | ⬜ not started — victim ids are in `CONFIG.monsterTypes` |
| Military — Guardsmen, helicopters | ⬜ not started |
| Military — tanks, police, paratroopers, lightning | ⬜ not started |
| Scoring & HUD | 🟨 3-column HUD strip reserved (names only). No score, no energy bars. Break kinds and collapse attribution are recorded and waiting for Session 8 |
| Day system & progression | ⬜ not started |
| 2-player & monster-vs-monster | 🟨 input + player records sized for 3; only P1 spawns |
| Art & audio pass | ⬜ not started — placeholder rectangles, no audio |

---

## What changed last session

Session 3 — destruction and collapse, all inside `index.html`:

- **Cell damage (§3.2).** `Building.damageCell(col, row, amount, byMonster, causeKind)` drives a cell down through `INTACT → CRACKED → HOLE → GONE`. HP is spread evenly across the three non-GONE states by `cellStateForHp`, so `CONFIG.buildings.cellHp` stays a free tunable — at the current value of 3 that's one punch per state. It returns the §6 break kind (`'partial'` on the first crack, `'full'` when the hole opens, `null` for the final removal, which §6 has no row for). Nothing reads the return value until Session 8.
- **Punch → cell (§3.2).** `Monster.resolvePunch` fires once per punch, at full fist extension, from whatever position the monster ends the tick at. `Game.cellTargetFor` picks the still-standing cell that overlaps the punch's target rect most, skipping `GONE` sections so repeated hits chew inwards instead of hitting air. See the "which cell" note below — the target rect is *not* the same box as the drawn fist, and that's deliberate.
- **`BuildingState = { STANDING, SHAKING, RUBBLE }` (§3.3).** `standing` is true for STANDING *and* SHAKING — the shaking window is a warning, not a removal, so a building stays fully climbable while it comes down. Traversal now gates on `standing` rather than `alive`; the entity stays `alive` as rubble because it still draws the mound (and Session 4 hangs items off it).
- **Collapse triggers (§3.3).** Either half the building's total cell HP is gone, or every cell of the bottom row is GONE. Both thresholds are in `CONFIG.buildings` with the reasoning written out there. **Both numbers are inferred** — see below.
- **Roof-drop collapse (§3.2).** An `AIR → ROOF` *landing* on a building already below `roofDropStructureFraction` starts the collapse. Only a landing does it, so walking onto a roof from a face is safe, and the monster is left standing on a shaking building — which is the point.
- **Fall damage (§3.3).** `Building.collapse` detaches every monster still attached to it, drops it (AIR, `vy = 0`, no knockback), and calls `Monster.applyDamage(CONFIG.monster.fallDamage, 'FALL')`. `applyDamage` is the single funnel Session 4's HealthSystem takes over; today it just accumulates into `damageTaken`.
- **Attribution (§6).** `Building.causedBy` / `causeKind` are set the moment the building starts shaking — the blow that doomed it owns the collapse, and a later hit during the shake can't steal the credit. On collapse a record goes onto `Game.collapseEvents` (`{ building, buildingIndex, causedBy, causeKind, simTime }`). `causedBy` is `null` for a third-party collapse, which is exactly what will make Session 6's demolition charges pay nobody.
- **`DustPuff extends Entity`** (16 per collapse) and a rubble mound, both drawn from fixed patterns rather than RNG so a collapse looks identical every run — which also keeps the headless tests deterministic. Rubble is scenery: not climbable, not landable, doesn't block the street.
- **`Game.dayCleared()`** — one-liner, true when every building is rubble. Session 9 acts on it; the debug strip shows it now.
- **Debug strip** gained per-building structure %/state, the collapse-attribution list, the monster's `damageTaken`, and a second punch overlay showing the rect that actually damages cells (red) versus the drawn fist (yellow).

**Which cell a punch lands on, since it's the non-obvious decision.** The damaging rect runs from the monster's *own centre* to `punchReach` past its leading edge — not from the leading edge outward like the drawn fist. That's load-bearing rather than cosmetic: a clinging monster is pinned flush inside the frontage (Session 2), so at the outermost column a purely forward reach punches clean past the building and that column could never be destroyed from a face. Two side effects fell out of it, both flagged as inferred: a monster on the street hits the **second** row up rather than the bottom row (it's more than two storeys tall and punches at chest height — the bottom row goes next, once that one is clear), and a punch thrown from the air damages cells too, since no stance restriction is applied.

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

## Verification — Session 3

Same method as Sessions 1 and 2: the **real** inline script from `index.html` executed headlessly in Node under a DOM/canvas shim, driven tick by tick through the game's own `InputManager`. **133 checks, all passing.**

Against the SESSION-PLAN Session 3 success criteria:

| Criterion | Result |
|---|---|
| Repeated punches visibly progress a cell through all damage states | ✅ `INTACT→CRACKED→HOLE→GONE` on real punches with real input; a fourth punch on a GONE cell is a no-op; all four states already had distinct art from Session 2 |
| A building reliably enters SHAKING before collapsing, with a window long enough to jump clear | ✅ both trigger routes tested; window measured at exactly 1.200000 s. See the escape sweep below |
| Staying on a collapsing building triggers the fall-damage path; jumping clear avoids it | ✅ from both ROOF and CLING, with `lastDamageSource === 'FALL'` |
| Rubble remains and blocks/supports nothing incorrectly | ✅ not climbable, not a face target, not a roof landing, doesn't block the street, does no damage, can't be punched |
| The collapse event carries a `causedBy` reference | ✅ punch, roof-drop and third-party cases, including that a third-party hit *during* the shake can't steal the credit |

**The escape sweep** — the criterion most worth real effort, since "long enough to jump clear" is a judgement call. Every building × ROOF and CLING × every reaction delay in the window, running a competent-player policy (head for an open edge and jump; jump again if you land back on it):

- **Reacting within half a second: 310/310 escapes.** No building, no stance, no delay in that range gets you killed.
- **Across the whole 1.2 s window: 709/710.** The single death is jumping from the dead centre of the 5-wide roof at exactly 0.58 s in — the jump doesn't clear the footprint, lands back on the roof one tick before it falls. That's correct emergent behaviour, not a bug, and it's the kind of thing that makes the window a skill check rather than a formality.
- **Not jumping: 10/30 escape.** Walking off a narrow roof works; climbing down works only from low on a short building. From the top of the 9-row building, climbing down is never fast enough (3.3 s of climb against a 1.2 s window). Jumping is the answer, which is what the design intends.
- The last tick of the window is genuinely too late — buildings update before monsters, so the collapse resolves before the jump input is read. Deliberate and consistent.

Balance, measured rather than asserted (all inferred, all for Session 12 to retune):

| Building | Bottom-row route | Structure-threshold route | Roof-drop route |
|---|---|---|---|
| 3×7 | 9 punches | 32 | 13 + a landing |
| 4×5 | 12 | 30 | 12 + a landing |
| 3×9 | 9 | 41 | 17 + a landing |
| 5×6 | 15 | 45 | 18 + a landing |
| 3×8 | 9 | 36 | 15 + a landing |

Two full end-to-end runs, driven entirely by real key input with no reaching into the model:

- **From the street:** building 0 down in **18 connecting punches over 5.7 s**, bottom two rows gone at 71% structure remaining — the bottom-row route.
- **From the face:** climb the 3×9, sweep it with punches, **68 punches over 45 s**, collapse fires at 49% structure with the bottom row still intact — the structure-threshold route. Attributed to GEORGE, and the monster was still clinging when it went, so it took the fall damage. This is the whole headline loop in one run: climb → punch → weaken → collapse → attribution → fall damage.

Three genuinely differentiated routes, which is what §3.2/§3.3 describe.

Also checked, as regression cover on Sessions 1 and 2:

- **Frame-rate independence still holds.** Street crossing: 5.650000 s at 1, 3 and 9 fixed steps per frame — spread 0. The SHAKING window: 1.200000 s at all three.
- **The Session 2 legality invariant still holds with collapses in the mix** — 48,000 ticks of randomised input across 8 seeds, asserted every tick, zero violations, including the two new ones (never clinging to rubble, never standing on rubble). Buildings actually came down during those runs.
- **All three monster types still produce byte-identical traces**, now including building damage and `damageTaken`. The type table still has no stat fields.
- **Console clean** — 3,000 ticks with interleaved renders through STANDING, SHAKING, RUBBLE and live dust, no warnings or errors. Every game state still draws; every `fillRect` got finite coordinates.
- **Every column of every building is punchable from a face**, including the outermost ones — the check that justifies the target-rect decision above.

A frame was rasterised from the game's real draw calls to confirm it looks right: an intact building, one with mixed CRACKED/HOLE/GONE cells, a shaking building with a monster on its roof, a fresh rubble mound with the dust plume still rising, and a settled one. Still **not opened in a real browser** (none in this environment) — worth one manual open.

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

**Session 4 — Eating & health.** The health economy: per-monster energy bar, all damage routed through one call, defeat/revert-to-human, `Pickup` entities revealed by punching cells, bad items, dynamite.

Seams already in place for it:
- **`Monster.applyDamage(amount, source)` is the funnel.** Every damage source is supposed to route through it; Session 3 replaced its body with a `damageTaken` counter and calls it from exactly one place (fall damage). Session 4 swaps the body for real energy bookkeeping and the defeat check. Keep the signature.
- **`CONFIG.monster.fallDamage` (30) is a placeholder** sitting in the monster block because there was no health block to put it in. Session 4 should decide whether damage values move into a `CONFIG.health` section — if so, move this one with them.
- **`Building.damageCell` is where a punch lands**, so it's the natural place to reveal a `Pickup` when a cell opens — the break kind it already returns tells you whether a hole just opened.
- **Rubble is drawn but inert.** §3.3 says a rubble mound "can still contain exposed food/items to grab"; nothing hangs off it yet.
- `Building.cellRect(col, row)` gives the world-space rect of any cell, which is where a revealed item belongs.

**Not done, deliberately left for their own sessions:** scoring numbers (Session 8 reads `Game.collapseEvents` and `damageCell`'s return value), third-party collapse sources (Session 6 calls `beginShaking(null, 'thirdParty')` — the path is already there and tested), and day-clear transition (Session 9; `Game.dayCleared()` already returns the right answer, nothing acts on it).

Success criteria are in `docs/sessions/SESSION-PLAN.md`.

---

## Open questions / things guessed at

Anything implemented from an inference rather than a confirmed source gets logged here **and** in `docs/design/13-inferred-items.md`.

| # | Item | Where | Status |
|---|---|---|---|
| 1 | Damage values, health-bar size, regen amounts | `CONFIG` | **`CONFIG.monster.fallDamage = 30` set in Session 3** as the first placeholder. Everything else pending Session 4 |
| 2 | Movement speed, jump arc, punch cadence | `CONFIG.monster` | **first-pass values set in Session 1** — walk 84 px/s, jump 270 px/s vs gravity 900 (≈40 px apex, 0.6 s airtime), punch 0.20 s. All guesses; retune in Session 12 |
| 3 | **Building collapse threshold** | `CONFIG.buildings` | **chosen in Session 3, and the biggest guess in the session.** `collapseStructureFraction 0.5` · `collapseOnBottomRowGone true` · `roofDropStructureFraction 0.8` · `shakeDuration 1.2 s`. The GDD says outright that the real trigger is undocumented and proposes this model; the four numbers are mine. Measured consequences are in the Verification section. Retune in Session 12 |
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
| 15 | **Which cell a punch lands on** — new in Session 3 | `Monster.punchTargetRect` / `Game.cellTargetFor` | Target rect runs from the monster's **own centre** to `punchReach` past its leading edge; hits the still-standing cell it overlaps most; GONE cells are skipped rather than absorbing the hit. Reaching back to the centre is what makes the outermost column breakable from a face. Side effects to verify: a street punch hits the **second** row up, not the bottom row; and an **airborne** punch damages cells |
| 16 | **Score for HOLE → GONE** — new in Session 3 | `Building.damageCell` | §6 has rows for "partial break" and "full break/hole" only. Clearing an already-holed section returns `null` — the literal reading, worth nothing. Session 8 decides for real |
| 17 | **Rubble is scenery** — new in Session 3 | `Building.drawRubble` | Not climbable, not landable, doesn't block the street. §3.3 only says it's a mound that can hold items. Consistent with #13 (street-as-a-lane) |
| 18 | **Fall damage is a straight drop** — new in Session 3 | `Building.collapse` | Detach to AIR with `vy = 0`, no knockback pop, no lateral throw. §3.3 describes damage but no knockback |
| 19 | **Cell HP spread across damage states** — new in Session 3 | `cellStateForHp` | `cellHp` is divided evenly across the three non-GONE states, so one punch = one state at the current value of 3 and the constant stays tunable. The GDD names the states but not their HP curve |

Port decisions (not fidelity claims, recorded so they aren't mistaken for one): keyboard mapping, and the 512×384 logical playfield (MCR-3 was 512×480). Both listed in `docs/design/13-inferred-items.md`.

---

## Session log

| # | Session | Date | Outcome |
|---|---|---|---|
| 0 | Repo scaffold & design-doc split | 2026-07-28 | ✅ |
| 1 | Core skeleton | 2026-07-28 | ✅ all success criteria verified |
| 2 | Buildings & climbing | 2026-07-28 | ✅ all success criteria verified |
| 3 | Destruction & collapse | 2026-07-28 | ✅ all success criteria verified (123 headless checks) |

---

## Notes for the next session

_(Anything the next session needs to know that isn't obvious from the code — deferred decisions, known rough edges, things deliberately left unfinished.)_

- **`window.RAMPAGE`** exposes `{ CONFIG, game, GameState, Entity, EntityManager, Monster, InputManager, Stance, Building, CellState, BuildingState, DustPuff, cellStateForHp }` for console poking and for the headless harness. Keep it exported — the headless tests drive the game entirely through it.
- **Update order matters now.** Buildings are added to the entity list before monsters, so a building's `update` (and therefore its collapse) runs before any monster's in the same tick. That's what makes the last tick of the SHAKING window unescapable, and it means `Building.collapse` can safely reach into monsters and change their stance — nothing has moved yet that tick. Don't reorder `spawnBuildings` / `spawnMonsters` without thinking about this.
- **Traversal gates on `building.standing`, not `building.alive`.** A rubble building is still an `alive` entity (it draws the mound and will hold items in Session 4). Anything new that asks "can I be on this building?" must use `standing`.
- **The four-stance handlers each own their exit conditions.** Session 3 added a fifth way to leave a surface — the building coming down underneath you — and did it in `Building.collapse` rather than in a stance handler, because the building is what knows. The handlers keep a `!b.standing → detach` safety net. Anything else that removes a surface should do the same. The invariant the tests assert is in the Verification section above — reuse it.
- **`CONFIG.buildings.dayLayout` has a height constraint**, documented in `CONFIG`: every building must be taller than the jump apex. If Session 9's `LevelData` ever authors a short building, a monster will be able to jump from the street onto its roof, which breaks the street-as-a-lane model. Either keep the constraint or decide deliberately to model the lane some other way.
- **Rooftop walking uses `walkSpeed`, climbing uses `climbSpeed`** — deliberately different, both guesses.
- **`getPunchBox()` and `punchTargetRect()` are different boxes on purpose.** The first is the fist the art draws; the second is what damages cells and reaches further back. Debug draws both (yellow / red). Sessions 5, 6 and 10 need to decide which one hits humans, enemies and other monsters — probably `getPunchBox()`, since the centre-reaching trick exists only to solve a building-frontage problem.
- **Debug readout** at the bottom of the screen (`CONFIG.debug.enabled`) is Session-1/3 scaffolding: state, FPS, monster telemetry, the `CROSS` stopwatch, per-building structure %/state, and the collapse-attribution list. Session 8 replaces it with the real HUD; the crossing timer can go then. It's now up to 5 lines and close to the bottom of the screen.
- **Jump apex is ~40 px against a 46 px monster** — under one body height. It's enough to clear every gap in the current layout. Raising `CONFIG.monster.jumpSpeed` is still the right lever if jumps feel bad, but it now has a second consequence: it raises the minimum legal building height (see the point above).
- **`DAY_CLEAR` and `GAME_OVER` stubs have no `update`** — they're intentional dead ends until Session 9. Nothing transitions into them yet. `DAY_INTRO` passes straight through to `PLAY`. **`Game.dayCleared()` now returns a real answer** — you can flatten the city and nothing happens.
- **`Game.collapseEvents` is never drained.** It's reset per day in `startDay()` and grows by one entry per collapse (max five a day at the moment). Session 8 should decide whether the ScoreSystem consumes it or just reads it.
- **No spatial partitioning, no camera, no audio, no `LevelData`** — deliberately not built, since no Phase-1 mechanic needs them yet. `Game.faceTargetFor` / `roofLandingAt` / `cellTargetFor` are linear scans over five buildings; that's the whole spatial system and it's enough.
- **The collapse sound is missing, obviously** — Session 11 owns audio. The dust puff is the only collapse feedback right now besides the shake.
- Sessions 1 and 2 are committed; Session 3 is not committed yet.