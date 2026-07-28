# Rampage Clone — Phase 1 Session Plan

Twelve sessions from empty repo to a faithful, playable Phase 1 clone. **One session = one fresh Claude Code session = one mechanic.**

Each spec lists: what it builds, which design files to read, testable success criteria, and what's explicitly *not* in it. Ordering matters — each session assumes the ones before it exist.

Exact paste-ready prompts and model settings for each session are in `IMPLEMENTATION-NOTES.md`.

---

## Session 0 — Repo scaffold & design-doc split

**Goal:** Turn the single GDD into the working repo structure. No gameplay code.

**Reads:** the full GDD (this is the one session that reads all of it).

**Builds:**
- Split the GDD into `docs/design/00-core-loop.md` … `12-parking-lot.md` per the split table in GDD §0, plus `13-inferred-items.md` from §13.
- Each file keeps its section's content verbatim — no rewriting, summarizing, or "improving."
- Add `CLAUDE.md`, `docs/STATUS.md`, `docs/sessions/SESSION-PLAN.md`.

**Done when:**
- Every GDD section appears in exactly one file, content intact.
- No section's rules are split across two files.
- `docs/design/13-inferred-items.md` lists all 10 inferred items.
- Nothing was added to or removed from the design content.

**Out of scope:** any `index.html`, any code at all.

**Model / effort:** Sonnet, default effort. This is mechanical file surgery.

---

## Session 1 — Core skeleton

**Goal:** A monster you can walk and jump around an empty street, on a correct game loop.

**Reads:** `10-architecture.md`, `01-monsters.md`, `02-movement-destruction.md` (movement half only).

**Builds:**
- `index.html`: canvas, inline script, commented section headers mirroring `docs/design/`.
- `CONFIG` object holding every tunable (speeds, jump, gravity, timings).
- Fixed-timestep loop with accumulator + interpolated render (per architecture §11.1).
- Game-state machine: `BOOT → TITLE → CHARACTER_SELECT → PLAY` (later states stubbed).
- `Entity` base class; `Monster` extends it, parameterized by `monsterType`.
- `InputManager` keyed **per player, sized for 3**, with P1 wired to keys.
- One monster: walk left/right, jump, punch animation (hits nothing yet), placeholder rectangle art.

**Done when:**
- Opening `index.html` directly in a browser shows a titled screen, and a key advances to a controllable monster.
- Movement is frame-rate independent: the monster crosses the screen in the same wall-clock time whether the tab runs at 60 or 144 Hz.
- All three monster types selectable and visibly distinct (color/label is enough) with **identical** movement values.
- Console is clean. No tunable number appears outside `CONFIG`.

**Out of scope:** buildings, climbing, enemies, health, score, HUD beyond a debug readout.

**Model / effort:** **Opus**, default effort. This session sets the architecture every later session builds on.

---

## Session 2 — Buildings & climbing

**Goal:** A city of buildings the monster can climb and stand on.

**Reads:** `02-movement-destruction.md`, `10-architecture.md`.

**Builds:**
- `Building` entity: a grid of destructible cells (per-cell state + HP), variable width/height, drawn as a window grid.
- A day's worth of buildings laid out across the street (hardcoded layout is fine here — data-driven days come in Session 9).
- Climbing: up/down on a building face while adjacent; cling state.
- Rooftop traversal: walk on top of a building.
- Jump-to-grab: jump toward a face and hold up to attach mid-air.

**Done when:**
- The monster can climb a building face, reach the roof, walk it, and climb down the far side.
- Jump-to-grab attaches reliably when jumping into a face.
- The monster cannot walk through buildings or float off a face.
- Transitions ground↔face↔roof never leave the monster stuck or clipped.

**Out of scope:** damaging buildings, collapse, contents of windows.

**Model / effort:** Sonnet, default. Bump to Opus if state transitions get fiddly.

---

## Session 3 — Destruction & collapse

**Goal:** Punch buildings apart and bring them down.

**Reads:** `02-movement-destruction.md`, `05-scoring.md` (attribution rule only).

**Builds:**
- Punch damages the targeted cell: intact → cracked (partial break) → hole (full break) → gone.
- Punch works from ground and while clinging.
- Collapse trigger per GDD §3.3: cell-HP threshold **or** bottom row destroyed → `SHAKING` state → collapse to rubble + dust puff.
- Roof-drop collapse: landing on a sufficiently weakened building finishes it.
- Fall damage: a monster on/clinging to a collapsing building takes a large hit (placeholder value — health lands next session).
- Collapse **attribution**: record which monster caused it (the 2,500-point rule depends on this).

**Done when:**
- Repeated punches visibly progress a cell through all damage states.
- A building reliably enters SHAKING before collapsing, with a window long enough to jump clear.
- Staying on a collapsing building triggers the fall-damage path; jumping clear avoids it.
- Rubble remains and blocks/supports nothing incorrectly.
- The collapse event carries a `causedBy` reference.

**Out of scope:** scoring numbers, window contents, military-caused collapse.

**Model / effort:** **Opus**, default. This is the most interconnected system in the game and the collapse threshold is an inferred mechanic that needs judgment.

---

## Session 4 — Eating & health

**Goal:** The health economy — take damage, eat to recover.

**Reads:** `03-eating-health.md`.

**Builds:**
- `HealthSystem`: per-monster energy bar; all damage sources routed through one `applyDamage(source)` so values stay in `CONFIG`.
- Defeat: energy hits zero → revert-to-human → walk off screen (state exists; continue flow comes in Session 9).
- `Pickup` entity revealed by punching cells: food (turkey, hamburger, milk, fruit, toast) heals.
- Bad items: poison, cactus, dynamite (with its ~2–3s exposure timer), and live electrical items (lit bulb, TV on, toaster, neon) shock.
- Photographer and bathtub-bather knock-off behaviors.
- Debug health readout on screen (real HUD is Session 8).

**Done when:**
- Punching cells reveals items; eating food raises the bar and bad items lower it, all values sourced from `CONFIG`.
- Exposed dynamite detonates on its timer and knocks the monster off the building.
- Photographer flash and bather water both knock the monster off a face.
- Energy reaching zero triggers the revert-to-human walk-off exactly once, cleanly.
- Every damage source in GDD §4.1 has a code path, even if some aren't reachable yet.

**Out of scope:** civilians as food (next session), scoring, the real HUD.

**Model / effort:** Sonnet, default. Mostly data-and-wiring work against a well-specified design section.

---

## Session 5 — Civilians & designated victims

**Goal:** People in windows: edible, grabbable, and one special victim per monster.

**Reads:** `01-monsters.md` (§2.3), `03-eating-health.md`.

**Builds:**
- `Human` entity with a role flag (civilian / designated victim / window-soldier placeholder).
- Civilians appear in windows and flee on the street; can be punched or eaten (heals).
- Designated victims: woman→George, middle-aged man→Lizzie, businessman→Ralph.
- Grab/hold: monster holds its designated victim; victim struggles free after a random hold time; can then be eaten.
- While holding: National Guardsmen stop firing (flagged inferred — single-source claim).

**Done when:**
- Each monster can grab only its own designated victim; the wrong victim is eaten or punched instead of held.
- The hold → struggle-free → eat sequence completes without leaving orphaned entities.
- Eating a civilian restores health per `CONFIG`.
- The "Guardsmen stop firing while holding" hook exists even though Guardsmen arrive next session.

**Out of scope:** the 4,000–6,000 point award (Session 8), soldier behavior.

**Model / effort:** Sonnet, default.

---

## Session 6 — Military A: Guardsmen & helicopters

**Goal:** The two threats present on every single day.

**Reads:** `04-military-hazards.md`.

**Builds:**
- `Enemy` base with a `behavior` field; `Projectile` entity (bullets, dynamite, bombs).
- National Guardsman: pops from windows to shoot and throw dynamite; edible from a window; street variant that runs in and plants a demolition charge at a building's base.
- Helicopter: flies over, turns, dives to strafe (early behavior) **and** drops bombs (later behavior) — both implemented, selected by a difficulty flag.
- Third-party collapse: charges and bombs damage buildings, and collapses they cause are attributed to *them*, not the player.
- `SpawnDirector` with per-day cadence values in `CONFIG`.

**Done when:**
- Guardsmen appear in windows, fire, throw dynamite, and can be eaten to remove the threat.
- Helicopters perform both strafing and bombing runs; toggling the difficulty flag switches behavior.
- A demolition charge can bring down a building, and that collapse is *not* credited to the player.
- Holding a designated victim visibly stops Guardsman fire (the Session 5 hook now observable).

**Out of scope:** ground vehicles, paratroopers, lightning.

**Model / effort:** Sonnet, default.

---

## Session 7 — Military B: tanks, police, paratroopers, lightning

**Goal:** Fill out the threat roster and the escalation curve.

**Reads:** `04-military-hazards.md`.

**Builds:**
- Tank: slow, armored, heavy shells with large knockback.
- Police car: faster movement and fire rate.
- Paratrooper: lands on a roof, defends it, rapid-fires when a monster is on that building.
- Lightning cloud: drifts over, fires bolts, heavy damage.
- Environmental weapons: manhole cover, flower pot, safe — knocked loose and dropped onto ground/air units.
- Escalation scalars in `CONFIG` along the five axes in GDD §5.2.

**Done when:**
- Shell hits knock the monster back a meaningful distance, including off buildings.
- Paratroopers only harass the monster on their own building.
- Lightning fires on schedule and damages on hit.
- A dropped object destroys a ground unit.
- Raising the difficulty scalar visibly increases unit count/aggression without code changes.

**Out of scope:** scoring, boats/trains (tied to level features in Session 9).

**Model / effort:** Sonnet, default.

---

## Session 8 — Scoring & HUD

**Goal:** Everything that's happening becomes legible and rewarded.

**Reads:** `05-scoring.md`, `08-hud-ui.md`.

**Builds:**
- `ScoreSystem` implementing the full GDD §6 table.
- Attribution enforced: the 2,500 building bonus only pays the monster that caused the collapse.
- Designated-victim hold bonus (4,000–6,000).
- HUD: per-monster score + energy bar + name, laid out in **three columns** with unused slots reserved.
- Title screen, character select, and a score readout that survives state transitions.

**Done when:**
- Every scoring row in GDD §6 is reachable and awards the correct value.
- A building destroyed by a demolition charge awards 0 to the player; the same building punched down awards 2,500.
- The HUD renders correctly with 1, 2, and 3 active monsters without layout changes.
- Scores persist across a day transition.

**Out of scope:** day progression itself, art polish.

**Model / effort:** Sonnet, default.

---

## Session 9 — Day system & progression

**Goal:** The game becomes a game: clear a city, advance, escalate, loop.

**Reads:** `06-levels-progression.md`, `08-hud-ui.md`.

**Builds:**
- `LevelData`: plain data array of day definitions (building count/heights, feature flags, enemy set, difficulty scalars).
- First slice of authored days: the Peoria opening + Plano as the 2-building day; the rest as fillable data rows.
- Day clear detection (all buildings rubble) → newspaper-headline interstitial → next day.
- Looping: day index wraps; later passes raise lightning/police per §7.2.
- Mega-vitamin event at each 128-day milestone: full heal + 5,000 points.
- Defeat → continue prompt during the walk-off window → rejoin.
- Level features where cheap: bridge, riverway (underwater damage), train/trolley, pier/boater — plus their scoring rows.

**Done when:**
- Destroying all buildings advances the day with an interstitial, and the next day loads from data.
- Adding a new city to `LevelData` requires zero code changes.
- The continue prompt appears during walk-off and rejoining actually restores play.
- Fast-forwarding the day index to 128 fires the mega-vitamin heal and bonus.
- Difficulty scalars visibly differ between day 5 and day 100.

**Out of scope:** authoring all 128 cities; multiplayer.

**Model / effort:** **Opus**, default. Data modeling plus the progression/continue state machine — the ambiguity is worth the capability.

---

## Session 10 — Local 2-player & monster-vs-monster

**Goal:** The thing that made the original a social game.

**Reads:** `07-multiplayer.md`.

**Builds:**
- Second player wired to a distinct key set; shared screen, shared camera.
- Monster-vs-monster punches deal real damage.
- Competitive pickups: first to grab wins; a punch makes a monster drop what it's holding (flagged inferred).
- Cannibalism: a defeated monster's human form can be eaten by another player before it exits (points + heal).
- Join-in-progress: P2 can join mid-day.
- Verify the 3-player-ready assumption still holds (P3 addable by config alone).

**Done when:**
- Two monsters play simultaneously with independent scores, health, and controls.
- Punching the other monster drains its bar; being punched forces a drop.
- One player can eat the other's downed human form for points and health.
- Both can be on the same building without collision or climbing bugs.
- Enabling a third player requires only a config change.

**Out of scope:** online play, split screen.

**Model / effort:** **Opus**, default. Multiplayer touches every system at once; this is where single-player assumptions surface.

---

## Session 11 — Art & audio pass

**Goal:** Make it read as Rampage instead of as rectangles.

**Reads:** `09-art-audio.md`.

**Builds:**
- Monster sprites/animation states: idle, walk, climb, jump, punch, grab/hold, eat, hit-react, revert-to-human. George and Ralph share a rig with a head/palette swap.
- Building damage-state tiles, rubble, dust puff on collapse.
- Enemy and item icons; on/off variants for electrical items.
- `AudioManager` + SFX set: roar, punch, collapse, explosion, eat, zap, day-clear sting.
- Arcade-styled HUD pass.

**Done when:**
- Every animation state has art and transitions cleanly.
- Electrical items are readable as on vs. off at a glance.
- Every SFX in GDD §10 fires on its trigger and can be muted.
- Frame rate holds at 60 with a full city and full enemy load.

**Out of scope:** music, exact arcade palette matching (polish item).

**Model / effort:** Sonnet, default. Volume work, not hard reasoning.

---

## Session 12 — Balance & fidelity pass

**Goal:** Close out the inferred-items list and make it feel right.

**Reads:** `13-inferred-items.md`, plus whichever system files a given item touches.

**Builds:**
- Tune every `CONFIG` placeholder against actual play: damage, health, regen, speeds, collapse threshold, spawn cadence.
- Walk `13-inferred-items.md` item by item; resolve, revise, or confirm-as-unknowable and mark accordingly.
- Fix accumulated rough edges from prior sessions' STATUS notes.

**Done when:**
- A full day is survivable but attritional — health matters without being punishing.
- Every item in `13-inferred-items.md` is marked resolved, tuned, or explicitly still-open.
- No `CONFIG` value is still a first-guess placeholder.
- A run of 5+ consecutive days plays without a crash or soft-lock.

**Out of scope:** anything in the Phase 2 parking lot.

**Model / effort:** Sonnet at default for tuning; **Fable** if a cross-system bug resists two Opus attempts.

---

## Sequencing notes

- **Sessions 1, 3, 9, 10 are the architecture-defining ones.** Those are the Opus sessions; the rest are execution against a settled shape.
- If a session's success criteria don't all pass, **don't roll into the next session** — fix or explicitly defer in STATUS first. Deferred items compound fast.
- Sessions 6 and 7 are one design section split in two deliberately. If Session 6 comes in light, pulling paratroopers forward is fine; don't pull the whole of 7.
- If a session balloons past what fits comfortably, stop, update STATUS, and start fresh. A clean handoff beats a long context.