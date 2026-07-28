# Rampage Clone — Game Design Document (Phase 1)

**Target:** Browser game (HTML5 Canvas + vanilla JavaScript), implemented via Claude Code.
**Goal:** A close-to-exact, playable replica of the 1986 Bally Midway arcade game *Rampage* (designers Brian Colin & Jeff Nauman).
**Version:** 0.1 · **Date:** 2026-07-01 · **Scope:** Phase 1 (faithful replica). Later phases add monsters/hazards/mechanics.

---

## 0. How to read (and later split) this document

Each numbered section below is **self-contained**: the rules for one mechanic live entirely inside one section, so this file can later be split into per-system files with no cross-cutting edits. Suggested future split:

| Section | Future file |
|---|---|
| 1 Core loop | `design/00-core-loop.md` |
| 2 Monsters | `design/01-monsters.md` |
| 3 Movement & destruction | `design/02-movement-destruction.md` |
| 4 Eating & health | `design/03-eating-health.md` |
| 5 Military & hazards | `design/04-military-hazards.md` |
| 6 Scoring | `design/05-scoring.md` |
| 7 Level structure & progression | `design/06-levels-progression.md` |
| 8 Multiplayer | `design/07-multiplayer.md` |
| 9 HUD/UI | `design/08-hud-ui.md` |
| 10 Art & audio | `design/09-art-audio.md` |
| 11 Technical architecture | `design/10-architecture.md` |
| 12 Phase 2+ parking lot | `design/99-parking-lot.md` |

**Confidence markers used below:**
- **[FAITHFUL]** — confirmed against sources; implement as written.
- **[INFERRED]** — not documented in accessible sources; a reasoned guess to tune during implementation. All of these are also collected in §13.

A running list of everything marked **[INFERRED]** is in **§13** so it's easy to verify later.

---

## 1. Core loop

Moment to moment, the player **is a giant monster demolishing a city block** while an escalating military tries to stop them. The loop is:

1. **Destroy buildings** — climb them, punch chunks out of them, and bring them down. A day (level) ends when every building on screen is rubble.
2. **Feed to survive** — punching buildings exposes food and civilians inside; eating them refills the health/energy bar that enemy fire drains.
3. **Fend off the military** — soldiers, vehicles, and aircraft attack continuously; the player dodges, out-positions, and swats them (often using the environment).
4. **Clear the day → next city** — a brief newspaper-headline interstitial, then a new cityscape with slightly more threat.

Why it's fun (design pillars to protect): **cathartic destruction** (everything on screen can be smashed), **legible chaos** (a 2D side view where the whole board is visible), **risk/reward tempo** (stopping to eat is safety *and* vulnerability), and **social play** (2+ monsters cooperating and sabotaging each other on the same screen). There is **no puzzle and no fail state for "wrong" actions** — the only lose condition is running out of health. Keep sessions short and immediately re-enterable. **[FAITHFUL]**

**Phase 1 scope:** implement the full loop for a single day and looping day-to-day progression (see §7). Everything the loop needs — buildings, eating, health, military, scoring, HUD, 1–2 players — is in scope. Nothing outside the loop is.

---

## 2. Monsters (George, Lizzie, Ralph)

Three playable monsters, all former humans mutated by an experiment gone wrong:
- **George** — King Kong–style giant ape (mutated by an experimental mega-vitamin).
- **Lizzie** — Godzilla/Ymir-style giant lizard (mutated by a radioactive lake).
- **Ralph** — giant werewolf (mutated by a food additive in a sausage/hot dog). **[FAITHFUL]**

### 2.1 Mechanical parity — the key design decision

In the **1986 arcade original, the three monsters are mechanically identical**: same movement speed, same jump, same punch damage, same health pool, same collision size. The only differences are **sprite art**, **punch-animation flavor** (George swings fists, Lizzie snaps with jaws, Ralph swipes with paws — same effect), and each monster's **designated victim** (§2.3). Asset reuse reinforces this: in the original, George and Ralph are the *same body with a head swap and shared palette*. **[FAITHFUL]**

> ⚠ Common misconception: many home-port manuals and FAQs claim "George climbs/jumps best, Lizzie is fastest, Ralph is strongest." That per-character stat differentiation belongs to the **1997 sequel *Rampage: World Tour*, not the 1986 original.** For a faithful Phase-1 clone, **do not** give the monsters different stats. (If you ever want to confirm empirically, observe the arcade ROM in MAME.) Flagged in §13.

**Implementation consequence:** one `Monster` class, parameterized by a `monsterType` enum that selects sprite sheet, punch animation, and designated-victim id. No per-type stat tables in Phase 1. This keeps balance trivial for any player combination and makes adding a 4th monster later a pure content addition.

### 2.2 Monster capabilities (shared by all three)

- Walk left/right on the ground and on rooftops.
- Climb up/down the face of a building.
- Jump (and jump-to-grab a building to combine leaping + climbing).
- Punch in the joystick direction, or straight ahead if neutral. Punching hits buildings, enemies, items, and other monsters.
- Grab/eat items and people from windows and rubble.
Full movement and destruction rules are in §3; eating/health rules are in §4. **[FAITHFUL]**

### 2.3 Designated victims

Each monster has one "designated victim" — a specific civilian type it can pluck from a window and hold:
- **George → the woman (lady in red).**
- **Lizzie → the middle-aged man (yellow shirt / white boxers).**
- **Ralph → the businessman (in a suit).**

While a monster holds its designated victim, **only the National Guardsmen stop firing** (other threats keep attacking). After a random hold time the victim struggles free (a few uppercuts to the monster's jaw), and the monster can then eat them for a large point bonus (see §6). **[FAITHFUL]** — the "only Guardsmen stop firing" detail comes from a single walkthrough; treat as the rule but verify (§13).

**Phase 1 scope:** implement the designated-victim grab/hold/eat for all three monsters. The "hold to score big" is a scoring feature, not required for the core loop, but it's cheap and characterful — include it.

---

## 3. Movement & destruction

**Controls (per monster):** an 8-way direction input + **Punch** + **Jump**. (See §9/§11 for keyboard mapping.) **[FAITHFUL]**

### 3.1 Movement & traversal

- **Ground:** left/right walks along the street.
- **Climbing:** when adjacent to a building face, up/down climbs it; the monster clings to the wall.
- **Jump:** vertical or directional leap. **Jump toward a building and hold up to grab it** — this is the fast way to cross from building to building and to start a climb mid-air.
- **Rooftops:** the monster can stand and walk on the top of a building until it's destroyed. **[FAITHFUL]**

### 3.2 Destroying a building

A building is a **vertical stack of destructible cells** (a grid of windows/wall sections). It comes down through accumulated damage:

- **Punch a wall section** (from the ground or while clinging to the face) to rip out that section. A punch that only cracks a section is a *partial* break; a punch that opens a full hole is a *full* break (these score differently — §6). Repeated hits destroy successive sections. **[FAITHFUL]**
- **Roof-drop collapse:** weaken a building, then **jump onto its roof** — the monster's weight finishes it off and it collapses. **[FAITHFUL]**
- **Third-party collapse:** buildings can also be brought down by **National Guard demolition charges at the base**, **helicopter bombs**, and **dynamite** hidden in walls. Important scoring rule: **the 2,500-point "building destroyed" bonus is awarded only if the player's own monster caused the collapse** (§6). **[FAITHFUL]**

### 3.3 Collapse trigger & consequences

- **Trigger:** a building collapses once enough of its structure is gone (a critical amount of cell damage / loss of lower support). **[INFERRED]** — the exact threshold isn't documented. Phase 1 model: each cell has HP; when total remaining structure drops below a threshold **or** the bottom row is destroyed, the building enters a short "shaking" state, then collapses into a rubble pile with a dust puff.
- **Rubble:** collapsed buildings leave a rubble mound on the street that can still contain exposed food/items to grab.
- **Falling damage:** if a monster is **still clinging to (or standing on) a building when it collapses, it takes a large chunk of damage** from the fall. The shaking state is the tell — jump clear in time to avoid it. **[FAITHFUL]**
- **Day clears** when all buildings on the board are rubble (§7). **[FAITHFUL]**

**Phase 1 scope:** cell-grid buildings with partial/full break states, roof-drop collapse, third-party collapse with correct point attribution, shaking→collapse with fall damage. Multiple building heights/widths; feature tiles (bridges, trains, piers) are level dressing handled in §7.

---

## 4. Eating & health

### 4.1 The health/energy system

Each monster has a single **energy bar** (health). Enemy attacks and hazards drain it; eating refills it. **When the bar empties, the monster reverts to its (naked) human form and walks off the side of the screen** — that monster is out. **[FAITHFUL]**

**Damage sources** (how the bar goes down):
- Being **shot** by National Guardsmen (bullets) and hit by thrown **dynamite**.
- Being hit by **tank / police shells**.
- **Falling** — from a collapsing building, or being **knocked off** by an attack.
- **Punches from another monster** (multiplayer — §8).
- **Lightning** strikes; **electrocution** from grabbing "live" electrical items (§4.3).
- **Going underwater** (river/pier levels).
- **Eating a bad item** (§4.3). **[FAITHFUL]**

**[INFERRED]** exact per-source damage values, total bar size, and regen amounts are not documented; Phase 1 uses tuned placeholder values (see §13). Tune so a day is survivable but attrition is real.

**[INFERRED]** The original may have shown descriptive meter states ("Excellent/Groggy/Weak"–style labels) — that's confirmed for *World Tour*, unclear for the 1986 arcade. Phase 1 default: a simple depleting bar; labels optional (§13).

### 4.2 Food & healthy items (restore health)

Found inside buildings (exposed by punching) and in windows/rubble. Eating restores a bit of energy:
**turkey** (cooling in windowsills), **hamburger** (mid-building), **milk**, **bowl of fruit** (bananas/apples), **toast** (wait for it to pop from a toaster). **[FAITHFUL]**

### 4.3 Bad items & environmental hazards inside buildings (drain health)

Grabbing/eating these hurts the monster — and because punching is fast, a greedy player will sometimes grab the wrong thing:
- **Poison** (skull-and-crossbones) — big health loss.
- **Cactus** — doesn't agree with the monster.
- **Dynamite** in a wall — on exposure you have ~2–3 seconds to leave before it blasts you off the building; eating it just hurts you.
- **Live electrical items** — **lit light bulb**, **TV that's on**, **toaster with no toast**, **neon sign** — shock the monster. (When *off*, TV/light bulb/neon can instead be knocked out for points — §6.)
- **Photographer** (in windows) — a camera flash knocks the monster off the building; eat them before the flash.
- **Bathtub with a person** — the bather blasts water to knock the monster off; eat them fast.
- **Toilet / empty bathtub** — no benefit if eaten (junk grabs). **[FAITHFUL]**

**Phase 1 scope:** implement food (heal), the bad/electrical items (damage/shock), photographer and bathtub "knock-off" behaviors, and dynamite's exposure-timer. Civilians and soldiers as edible are covered in §5/§6 (eating a soldier both removes a threat and heals — keep that dual role).

---

## 5. Military & hazards (threat roster + escalation)

The military attacks continuously. Present **every day**: National Guardsmen and helicopters. Other units appear per the day schedule (§7). **[FAITHFUL]**

### 5.1 Enemy roster & attack patterns

| Enemy | Behavior | Notes |
|---|---|---|
| **National Guardsman** | Pops out of building windows to **shoot** and **throw dynamite**; some run in from the screen edge to plant **demolition charges at a building's base**. | The bread-and-butter threat, everywhere. Can be eaten out of windows (removes threat + heals). |
| **Helicopter** | Air support, every day. **Early:** flies over, turns, and **dives to strafe** (rapid burst). **Later:** **drops bombs** from altitude. | Can be destroyed on its approach. Bombs can be baited onto buildings to collapse them. |
| **Tank** | Slow, heavily armored; fires **heavy shells** that do big damage and **large knockback** (can knock a monster off a building or across the screen). | Punish between shots or drop objects on it. |
| **Police car** | Like a tank but **moves and fires faster**. | |
| **Paratrooper** | Lands on a rooftop and **defends that building**; **rapid-fires** when the monster is on it. | Scale fast and eat it. |
| **Photographer** | (Also a building hazard, §4.3) flash **knocks the monster off** the building. | |
| **Lightning cloud** | Drifts over the city and fires **lightning bolts**; heavy damage on hit. | A hazard rather than a unit; scheduled per day. |
| **Boats / boater** | On pier/riverway days; smashable. | Level-feature-dependent. |
| **Trains / trolleys** | On train-feature days; smashable back and forth ("train pong" in multiplayer). | |

**[FAITHFUL]** for all rows. **[INFERRED]:** precise fire rates, projectile speeds, spawn cadences, and damage-per-hit are not documented — tune during implementation (§13).

### 5.2 How threat escalates over time

Difficulty ramps across days along five axes (all **[FAITHFUL]** in direction; exact rates **[INFERRED]**):
1. **More buildings** to clear (roughly 3 early → 4–6 later).
2. **More and faster ground units** — tanks and police introduced and stacked.
3. **Helicopters shift from strafing to bombing.**
4. **Lightning appears and recurs more often.**
5. **On later loops of the city list, more lightning and more police** than the first pass.

**Phase 1 scope:** implement Guardsmen + helicopters (both strafe and bomb behaviors), tanks, police, paratroopers, lightning, and the "environmental weapon" interactions (drop manhole cover / flower pot / safe onto ground/air units — §6). Boats/trains are tied to level features (§7) — include if their feature is implemented in Phase 1; otherwise flag as deferred there.

---

## 6. Scoring

Points come from destruction, eating, and grabbing valuables. Arcade values below are **[FAITHFUL]** (from the arcade scoring table). **No extra lives are ever awarded** (continues are coin/credit-based — §7).

| Action / item | Points |
|---|---|
| Punch — partial break in building | 25 |
| Punch — full break / hole in building | 225 |
| **Building destroyed (by your own monster only)** | 2,500 |
| Eat a civilian | 500 |
| Food item (turkey, milk, fruit, hamburger, toast) | 175 |
| Hold your **designated victim** | 4,000–6,000 |
| Bag of loot | 100–500 |
| Safe (after opening) | 100–500 |
| Flower pot | 500 |
| Light bulb (off) | 500 |
| Television (off) | 500 |
| Manhole cover (per hit) | 500 |
| Train (per hit) | 500 |
| Neon sign | 1,000 |
| Boater | 750 |
| Helicopter | 750 |
| Photographer | 750 |
| Police car | 750 |
| Car — parked then takes off quickly | 750 |
| Car — moving slowly | 200 |
| Tank | 200 |
| Car — parked | 100 |
| National Guardsman | 50 |
| Paratrooper | 50 |
| **Mega-vitamin bonus** (every 128 days, full-heal event) | 5,000 |

Key rule to preserve: **the 2,500 building bonus is only credited to the monster that actually caused the collapse** — buildings knocked down by demolition charges, bombs, or another monster don't pay you. **[FAITHFUL]**

**Phase 1 scope:** per-monster score, all destruction/eating/valuable point awards above, correct collapse attribution, and the 128-day mega-vitamin bonus. Items whose scoring depends on unimplemented level features (boater/train) follow those features (§7).

---

## 7. Level structure & progression

### 7.1 Unit of play: the "day"

Each **day = one city screen** with a fixed number of buildings to level (2–6). Clear all buildings → the day ends → a brief **newspaper-headline interstitial** → next day. **[FAITHFUL]**

### 7.2 City list, looping, and the arcade quirks

- **128 unique days**, running as a set list of North American cities. **[FAITHFUL]**
- The arcade list **starts in Peoria, IL (day 1)** and **ends in Plano, IL (day 128)**; Plano is the only day with just **2 buildings** (an in-joke — the co-designer's hometown). **[FAITHFUL]**
- The 128-day sequence **runs six times total** (the first pass plus five repeats) for **768 days**, then resets to day 1. Later passes escalate (more lightning/police). **[FAITHFUL]** — sources phrase this as "repeats five times"; 6 × 128 = 768.
- **Mega-vitamin bonus:** surviving to the end of each 128-day block **fully restores health and awards 5,000 points** (days 128, 256, 384, 512, 640, 768). **[FAITHFUL]**
- (Note: some *home ports* use a different list, e.g. San Jose → Los Angeles. We follow the **arcade** list for faithfulness.)

### 7.3 Per-day features (level dressing)

Days can include set-piece features: **bridge**, **riverway** (water hazard — going under drains health), **piers + boater**, and **train/trolley**. These change the board layout and add feature-specific smashables/hazards. **[FAITHFUL]**

### 7.4 Win / lose / continue

- **Advance:** clear all buildings on a day. **[FAITHFUL]**
- **No true "win":** the arcade loops indefinitely (resets after 768). Reaching that is the de facto endgame. **[FAITHFUL]**
- **Lose (per monster):** energy hits zero → monster reverts to a human and walks off screen. In multiplayer, remaining monsters continue. **[FAITHFUL]**
- **Continue ("buy back in"):** if the player re-inserts a credit **before the human figure finishes walking off screen**, that monster rejoins and play continues. **[FAITHFUL]**

**Phase 1 scope (pragmatic):**
- Implement the **day → clear → headline → next day** loop with looping progression and the mega-vitamin heal event.
- **[INFERRED]/decision:** it is not necessary to author all 128 named cities to be "faithful and playable." Phase 1 should implement the **day *system*** (data-driven day definitions: building count, feature flags, enemy set, difficulty scalars) and ship a **first slice of authored days** (start with the Peoria→… opening and Plano as the 2-building day), with the remaining cities as a data table to fill in. Structure the day list as a plain data array so adding cities is content-only. Flagged in §13.
- Implement the browser analogue of "buy back in" (a **Continue prompt / key press** during the human-walk-off window; no coins). Keep the walk-off timer as the continue window.
- Headlines: a small rotating pool of the game's comedic headlines shown on the interstitial (paraphrased/original text is fine — do not reproduce long copyrighted strings verbatim; a handful of short original headlines in the same spirit is acceptable for Phase 1).

---

## 8. Multiplayer (local, matching the original)

The arcade is **3-player simultaneous**, one control panel per monster, on **one shared screen** — a defining feature. Players can **cooperate or compete**, and can **join at any time** (the "join the action" jump-button start). **[FAITHFUL]**

### 8.1 Phase 1 target

- **Local 2-player** on one keyboard (per the project brief), sharing one screen and one camera.
- **Architecture must support up to 3 players** (arcade parity) so adding P3 later is config-only — see §11. Any player may pick any of the three monsters at select. **[FAITHFUL intent]**

### 8.2 Monster-vs-monster interactions (faithful)

- **Monsters can punch each other, dealing real damage** to the other's energy bar. **[FAITHFUL]**
- **Competition for food/valuables** — whoever grabs it gets it; punching a monster can make it drop what it's holding / interrupt a grab. **[INFERRED]** on the exact "punch to make them drop" rule — implement drop-on-hit as the faithful-feeling default; verify (§13).
- **Cannibalism of the defeated:** when a monster's energy is depleted and it reverts to human form, **another player can eat that human** before it walks off screen (points + heal). **[FAITHFUL]**
- **Cooperative smashing set-pieces** like "train pong" (two monsters batting a train back and forth for repeated points). **[FAITHFUL]**

**Phase 1 scope:** shared-screen 2P, monster-vs-monster punch damage, competitive item pickup, and eat-the-downed-human. "Train pong" comes for free if trains/§7 features are in. No online play (that's §12).

---

## 9. HUD / UI

On-screen, always visible (top of screen, one column per active monster): **[FAITHFUL]**

- **Per-monster score.**
- **Per-monster energy/health bar** (with the monster's name/label).
- (P3 column reserved even in 2P builds — leave layout room.)

Other UI states:
- **Title / attract screen** (Phase 1: minimal title + "press to start").
- **Character select** — pick any of George/Lizzie/Ralph per player; unused slots joinable later.
- **Day interstitial** — city name + comedic newspaper headline (§7).
- **Continue prompt** — appears during a defeated monster's walk-off window (§7.4).
- **Mega-vitamin bonus flourish** at each 128-day milestone.

**[INFERRED]:** exact arcade HUD art/fonts and whether health shows descriptive labels (§4.1) — Phase 1 uses clean, readable placeholders (bars + numeric score). Keep HUD rendering isolated so a faithful art pass can drop in later. Design the HUD to read clearly for **up to 3 columns** from the start.

**Phase 1 scope:** score + health bars for all active monsters, title, character select, day interstitial, continue prompt, bonus flourish. No pause menus/options screens required.

---

## 10. Art & audio direction

**Recommended direction: a faithful pixel-art homage** — cartoony, exaggerated, comedic (monsters mug at the "camera"; the tone is B-movie, not horror). This is both true to the original and the most practical for a solo dev, because it needs only small, chunky sprites and tiled buildings. **[FAITHFUL tone]**

**Practical art plan for a solo dev:**
- **Monsters:** one silhouette-distinct sprite set each. Exploit the original's trick — **George and Ralph can share a body rig with a head/palette swap**; Lizzie is the distinct one. Animations needed: idle, walk, climb, jump, punch, grab/hold, eat, hit-react, and the **revert-to-human walk-off**.
- **Buildings:** tile-based facades with **damage states per cell** (intact → cracked → hole → gone), plus a rubble mound and a **dust-puff** on collapse (the dust also conveniently hides animation seams — the original did exactly this).
- **Enemies/items:** small readable icons; palette-driven "on/off" variants for electrical items (lit vs dark) so the hazard read is instant.
- **[INFERRED]:** exact arcade palettes (e.g., George's body color is described inconsistently across sources) — pick a clear, distinct palette per monster and treat exact-hue fidelity as a later polish item (§13).

**Audio (keep minimal in Phase 1):** monster roars, punch/impact, building-collapse rumble, explosion, eat/heal blip, damage/shock zap, and a short day-clear sting. Placeholder or lightweight synthesized SFX are fine for Phase 1; a faithful pass can come later. Keep an audio manager abstraction so swapping assets is trivial (§11).

**Phase 1 scope:** functional, consistent placeholder-to-homage art and a small SFX set. Do not block gameplay work on art polish. No music required for Phase 1.

---

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

---

## 12. Phase 2+ parking lot (flagged extension points, not designed)

Lightweight list — future directions the architecture leaves room for. Not in Phase 1; noted so ideas have a home:

- **More monsters:** Larry (giant rat, from the Lynx port), Tiny, or originals. (Pure content via `monsterType` + `LevelData`.)
- **Expanded moveset:** kicks and special moves (as introduced in *World Tour*), e.g. a "mega jump" roof-slam, jump-kick vs. aircraft.
- **Per-monster stats (optional):** if ever desired, a stats table keyed by `monsterType` — deliberately excluded from Phase 1 for faithfulness.
- **Power-ups / mega-food / bonus items** beyond the arcade set.
- **Boss/heavy units:** attack robots (e.g. *World Tour*'s ED-209-like enemies).
- **Full water/underwater behavior**, weather/day-night variation, destructible set-pieces beyond bridges/trains.
- **3rd local player** wired up (architecture already supports it), and **online multiplayer**.
- **Full 128-city authored list** + richer headline pool + between-city map screen.
- **Score-attack / redemption / challenge modes**; high-score persistence.
- **Mobile/touch controls** and responsive canvas scaling.
- **Faithful art/audio pass:** exact palettes, arcade-accurate HUD, roars/music, meter labels.

---

## 13. Inferred items to verify (running list)

Everything Phase 1 is guessing at rather than confirming — dial these in during implementation or verify against the arcade ROM:

1. **Monster mechanical parity** — treated as *identical* (faithful to 1986; the "different stats" belief is from *World Tour*/port manuals). High confidence; confirmable in MAME. (§2.1)
2. **Damage values / health-bar size / regen amounts** — not documented; placeholder in `CONFIG`. (§4.1)
3. **Movement speed, jump arc, punch cadence** — not documented numerically; tune to feel. (§3, §11.4)
4. **Health-meter representation** — bar vs. descriptive labels ("Excellent/Groggy/…"); labels confirmed for *World Tour* only. Phase 1: bar. (§4.1, §9)
5. **Building collapse threshold** — exact trigger undocumented; using per-cell HP + bottom-row/critical-mass rule. (§3.3)
6. **Monster palettes/exact colors** — sources conflict (esp. George); pick distinct palettes, treat exact hues as polish. (§10)
7. **"Holding designated victim → only Guardsmen stop firing"** — from a single walkthrough; implement as the rule, verify. (§2.3)
8. **Monster-vs-monster "punch to force drop"** — implemented as drop-on-hit; verify exact original behavior. (§8.2)
9. **Full city list authoring** — Phase 1 ships the day *system* + a first slice of authored days (Peoria opening, Plano as the 2-building day); remaining 128 cities are content to fill in. (§7.2)
10. **Enemy fire rates / projectile speeds / spawn cadences** — undocumented; tune. (§5.1)

---

## Sources consulted

- Wikipedia — *Rampage (video game)* (development notes: head-swap/shared palette; 128 days × loops; designers; inspirations).
- Rampage Wiki / Fandom — *Rampage (1986)* (designated victims per monster; Peoria→Plano; mega-vitamin; 768-day reset; damage sources).
- StrategyWiki — *Rampage* (controls; MCR-3 hardware; 768 days / cycle repeats).
- GameFAQs — *Rampage* arcade guide by War_Doc (full scoring table; enemy roster; bad/healthy/good items; day-by-day chart; 128-day heal milestones; headlines).
- Arcade-History — *Rampage (1986)* (release; 768 levels; Plano detail; damage-source summary; "buy back in").
- Grokipedia — *Rampage (1986 video game)* / *(franchise)* (control scheme; punch-animation-per-monster flavor; asset reuse for Ralph).
- The King of Grabs; PrimeTime Amusements; Dinogame GG history (identical-mechanics summary; collapse/fall-damage; climb-and-punch; three-player social design).
- ComicBook.com; Digital Press (revert-to-human and "unless another player eats them first"; *World Tour* differences that clarify what is *not* in the 1986 original).