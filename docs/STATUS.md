# Rampage Clone — STATUS

**Phase:** 1 (faithful replica)
**Last updated:** 2026-07-28 — end of Session 1
**Last session:** Session 1 — Core skeleton
**Next session:** Session 2 — Buildings & climbing

---

## Current state

**Playable?** Barely — one monster walks, jumps and punches the air on an empty street. Nothing to hit yet.

**What exists:**
- `index.html` — the game. Single self-contained file, no build step, opens directly in a browser.
- `docs/design/` — Phase 1 GDD split into 12 per-system files + inferred-items list
- `docs/STATUS.md` — this file
- `docs/sessions/SESSION-PLAN.md` — ordered session list with success criteria
- `CLAUDE.md` — project rules

**How to run:** open `index.html`. Press **F / G / Enter** at the title → pick a monster with **A/D**, confirm with **F** → play.
**P1 controls:** A/D move · W/S (read, unused until climbing) · **G** jump · **F** punch.
P2 (arrows + `.` `/`) and P3 (IJKL + O/P) key maps exist as data but no second monster spawns yet — `CONFIG.players.activeAtStart` is 1 until Session 10.

| System | State |
|---|---|
| Game loop & state machine | ✅ fixed timestep + interpolated render; BOOT→TITLE→CHARACTER_SELECT→PLAY, later states stubbed |
| Monster movement | 🟨 walk / jump / punch anim on flat ground. No climbing, no rooftops |
| Buildings & climbing | ⬜ not started |
| Destruction & collapse | ⬜ not started — punch hitbox exists and hits nothing |
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

## What's next

**Session 2 — Buildings & climbing.** `Building` entity as a grid of destructible cells, a hardcoded day layout, climbing a face, rooftop walking, and jump-to-grab.

Seams already in place for it:
- `Stance` in `index.html` is `{ GROUND, AIR }` — add `CLING` and `ROOF` to that enum rather than starting a second state machine.
- `Monster.update` does its street contact in one clearly-marked block; that's where building faces/roofs replace the flat-ground test.
- `InputManager` already reads `up`/`down` per player; climbing needs no input work.
- `EntityManager` has no spatial query yet. Add one when buildings need it — not before.

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
| 9 | **Movement during a punch** — new in Session 1 | `Monster.update` | GDD is silent. Implemented as: punch **roots** the monster on the ground, airborne momentum kept. Verify |
| 10 | **Air control** — new in Session 1 | `CONFIG.monster.airControl` | 0.9 of walk speed while airborne. No documented rule. Interacts with jump-to-grab in Session 2 |

Port decisions (not fidelity claims, recorded so they aren't mistaken for one): keyboard mapping, and the 512×384 logical playfield (MCR-3 was 512×480). Both listed in `docs/design/13-inferred-items.md`.

---

## Session log

| # | Session | Date | Outcome |
|---|---|---|---|
| 0 | Repo scaffold & design-doc split | 2026-07-28 | ✅ |
| 1 | Core skeleton | 2026-07-28 | ✅ all success criteria verified |

---

## Notes for the next session

_(Anything the next session needs to know that isn't obvious from the code — deferred decisions, known rough edges, things deliberately left unfinished.)_

- **`window.RAMPAGE`** exposes `{ CONFIG, game, GameState, Entity, EntityManager, Monster, InputManager, Stance }` for console poking and for the headless harness. Keep it exported.
- **Debug readout** at the bottom of the screen (`CONFIG.debug.enabled`) is Session-1 scaffolding, including the `CROSS` stopwatch used to prove frame-rate independence. Session 8 replaces it with the real HUD; the crossing timer can go then.
- **Jump apex is ~40 px against a 46 px monster** — under one body height. If Session 2's jump-to-grab feels bad, raise `CONFIG.monster.jumpSpeed` rather than reworking the physics.
- **`DAY_CLEAR` and `GAME_OVER` stubs have no `update`** — they're intentional dead ends until Session 9. Nothing transitions into them yet. `DAY_INTRO` passes straight through to `PLAY`.
- **Punch hits nothing.** `Monster.getPunchBox()` returns the hitbox and it's drawn in debug yellow; Session 3 is the first thing to test against it.
- **No spatial partitioning, no camera, no audio, no `LevelData`** — deliberately not built, since no Phase-1 mechanic needs them yet.
- Nothing was committed to git this session.