# Rampage Clone — Implementation Notes

Setup, model settings, and exact paste-ready prompts for each Claude Code session.

---

## 1. Repo setup (do this once, before Session 0)

```
rampage/
├── CLAUDE.md
├── index.html                    ← created in Session 1
└── docs/
    ├── STATUS.md
    ├── RAMPAGE-GDD-PHASE1.md     ← the single GDD; Session 0 splits it
    ├── design/                   ← populated by Session 0
    └── sessions/
        └── SESSION-PLAN.md
```

Drop in `CLAUDE.md`, `docs/STATUS.md`, `docs/sessions/SESSION-PLAN.md`, and the GDD, then `cd rampage && claude`.

Worth doing: `git init` and commit at the end of every session. It's the cheapest possible undo when a session goes sideways, and it makes "revert to last known good" a real option instead of a rewrite.

---

## 2. Model and effort settings

Two independent dials. **Model = how capable. Effort = how thorough.** Per Anthropic's guidance, <cite index="29-1">effort controls not just thinking time but how many files Claude reads, how much it verifies, and how far it pushes through a multi-step task before checking back in with you</cite>.

### The models, framed usefully

<cite index="29-1">Fable is a specialist who's seen problems almost no one else has, Opus is the expert, and Sonnet is a really good generalist — and the effort level decides how much time any of them spends on your task.</cite>

**Recommendation for this project:**

| Session | Model | Why |
|---|---|---|
| 0 — doc split | Sonnet | Mechanical file surgery |
| **1 — core skeleton** | **Opus** | Sets the architecture everything else inherits |
| 2 — buildings/climbing | Sonnet | Well-specified, contained |
| **3 — destruction/collapse** | **Opus** | Most interconnected system; inferred collapse rule needs judgment |
| 4 — eating/health | Sonnet | Data and wiring against a clear spec |
| 5 — civilians | Sonnet | Contained |
| 6 — military A | Sonnet | Contained |
| 7 — military B | Sonnet | Contained |
| 8 — scoring/HUD | Sonnet | Table-driven |
| **9 — day system** | **Opus** | Data modeling + progression state machine |
| **10 — multiplayer** | **Opus** | Touches every system; surfaces single-player assumptions |
| 11 — art/audio | Sonnet | Volume work |
| 12 — balance | Sonnet | Tuning |

### Effort

**Leave it at default.** <cite index="29-1">Anthropic's guidance is that for most tasks you should use the model's default effort level — the level where Claude scales its token usage to what most people would want to spend on a task</cite>, and <cite index="29-1">effort is better treated as a general preference based on the kind of work you do than a task-by-task decision</cite>.

### When something goes wrong

The diagnostic that matters: <cite index="29-1">did Claude not know enough, or did it not try hard enough?</cite>

- **Skipped a file, didn't verify, bailed on a refactor partway** → raise **effort**.
- **Had all the context, clearly tried, still confidently wrong** → raise **model**.
- <cite index="29-1">But first check upstream — if you're raising effort on a task that shouldn't need it, the fix is often in your context, your CLAUDE.md, or how the task was scoped.</cite>

### Where Fable 5 earns its cost here

Save it. <cite index="29-1">Fable pulls furthest ahead on long, multi-step work — in Anthropic's testing it finished jobs Opus and Sonnet couldn't reach at any effort level — and it also costs the most per token, which is the other reason to save it for work that needs it.</cite>

Realistic triggers on this project:
- A bug that spans destruction + multiplayer + collapse attribution and has survived two Opus attempts.
- Session 12 fidelity work where several inferred mechanics interact and tuning one breaks another.
- A late-phase refactor touching every system at once.

Reaching for Fable on Session 2 is paying specialist rates for generalist work. Reaching for it on a cross-system bug at hour three of debugging is exactly right.

### Mechanics

- `/model` — interactive picker; sets model **and** effort. Takes effect next turn.
- `claude --model opus` — set at launch for a session.
- `/status` — check what you're currently running.
- `/context` — check what's eating your context window.

---

## 3. Session prompts

Start each in a **fresh** Claude Code session. `CLAUDE.md` loads automatically — don't restate its rules.

---

### Session 0 — Repo scaffold & design-doc split
*Sonnet, default effort*

```
Read docs/RAMPAGE-GDD-PHASE1.md and split it into per-system files under docs/design/,
following the split table in section 0 of that document.

Rules for the split:
- Copy content verbatim. Do not rewrite, summarize, condense, or "improve" anything.
- Every section lands in exactly one file. No mechanic's rules split across two files.
- Also create docs/design/13-inferred-items.md from section 13.
- Give each file a one-line header naming which GDD section it came from.
- Leave docs/RAMPAGE-GDD-PHASE1.md in place as the archived original.

When done, list the files created and confirm every GDD section is accounted for.
Then update docs/STATUS.md.
```

---

### Session 1 — Core skeleton
*Opus, default effort*

```
Session 1 from docs/sessions/SESSION-PLAN.md: the core skeleton.

Read docs/design/10-architecture.md and docs/design/01-monsters.md first, plus the
movement portion of docs/design/02-movement-destruction.md.

Build index.html as a single self-contained file that runs by opening it directly
in a browser — no build step, no dependencies, no ES modules.

Include:
- A CONFIG object at the top holding every tunable number
- Fixed-timestep loop with accumulator and interpolated render
- Game state machine: BOOT → TITLE → CHARACTER_SELECT → PLAY, with later states stubbed
- Entity base class, and Monster extending it, parameterized by monsterType
- InputManager with per-player key sets, structured for 3 players, P1 wired up
- One controllable monster: walk, jump, punch animation. Placeholder rectangle art is fine.
- Commented section headers matching the docs/design/ file names

All three monsters must use identical movement values — they differ only by sprite/label.

Before you finish, verify against the Session 1 success criteria in SESSION-PLAN.md,
especially that movement is frame-rate independent. Then update docs/STATUS.md.
```

---

### Session 2 — Buildings & climbing
*Sonnet, default effort*

```
Session 2 from docs/sessions/SESSION-PLAN.md: buildings and climbing.

Read docs/design/02-movement-destruction.md.

Add to index.html:
- A Building entity: grid of destructible cells with per-cell state and HP, variable
  width and height, drawn as a window grid
- A hardcoded row of buildings across the street (data-driven days come in Session 9)
- Climbing up/down a building face with a cling state
- Rooftop walking
- Jump-to-grab: jumping into a face while holding up attaches the monster

Cells should carry damage state now even though nothing damages them yet — Session 3
builds on this.

The thing most likely to go wrong is the ground↔face↔roof transitions. Make sure the
monster can never get stuck, clipped, or floating. Verify against the Session 2 success
criteria, then update docs/STATUS.md.
```

---

### Session 3 — Destruction & collapse
*Opus, default effort*

```
Session 3 from docs/sessions/SESSION-PLAN.md: destruction and collapse.

Read docs/design/02-movement-destruction.md, plus the collapse-attribution rule in
docs/design/05-scoring.md.

Add to index.html:
- Punching damages the targeted cell: intact → cracked → hole → gone
- Punching works from the ground and while clinging to a face
- Collapse trigger per the design doc: cell-HP threshold or bottom row destroyed →
  SHAKING state → collapse to rubble with a dust puff
- Roof-drop collapse: landing on a weakened building finishes it off
- Fall damage for a monster on or clinging to a collapsing building (placeholder value
  in CONFIG; the health system arrives in Session 4)
- Collapse events must record which monster caused them — Session 8's 2,500-point rule
  depends on this attribution

Note: the exact collapse threshold is an inferred mechanic, not a confirmed one. Flag
what you chose in a code comment, in chat, and in docs/design/13-inferred-items.md.

The SHAKING window needs to be long enough that jumping clear is a real skill check.
Verify against the Session 3 success criteria, then update docs/STATUS.md.
```

---

### Session 4 — Eating & health
*Sonnet, default effort*

```
Session 4 from docs/sessions/SESSION-PLAN.md: eating and health.

Read docs/design/03-eating-health.md.

Add to index.html:
- A HealthSystem with a per-monster energy bar, where every damage source routes through
  a single applyDamage(source) function and all values live in CONFIG
- Defeat: energy reaching zero triggers revert-to-human and a walk-off (the continue
  flow itself comes in Session 9)
- A Pickup entity revealed by punching cells
- Food that heals: turkey, hamburger, milk, fruit, toast
- Bad items: poison, cactus, dynamite with its exposure timer, and live electrical items
  (lit bulb, TV on, toaster, neon sign)
- Photographer flash and bathtub-bather water: both knock the monster off a building
- A temporary debug health readout — the real HUD is Session 8

Every damage source listed in the design doc should have a code path even if some aren't
reachable until later sessions.

Verify against the Session 4 success criteria, then update docs/STATUS.md.
```

---

### Session 5 — Civilians & designated victims
*Sonnet, default effort*

```
Session 5 from docs/sessions/SESSION-PLAN.md: civilians and designated victims.

Read the designated-victim section of docs/design/01-monsters.md and
docs/design/03-eating-health.md.

Add to index.html:
- A Human entity with a role flag: civilian, designated victim, or window-soldier
  (the soldier role is a placeholder until Session 6)
- Civilians in windows and fleeing on the street; punchable or edible, and eating heals
- Designated victims: woman for George, middle-aged man for Lizzie, businessman for Ralph
- Grab and hold: the monster holds its designated victim, the victim struggles free after
  a random hold time, and can then be eaten
- While holding, National Guardsmen stop firing — build the hook now, it becomes
  observable in Session 6

That "only Guardsmen stop firing" detail comes from a single source. Treat it as the rule
but note it in docs/design/13-inferred-items.md.

No point values yet — scoring is Session 8. Verify against the Session 5 success criteria,
then update docs/STATUS.md.
```

---

### Session 6 — Military A: Guardsmen & helicopters
*Sonnet, default effort*

```
Session 6 from docs/sessions/SESSION-PLAN.md: National Guardsmen and helicopters — the
two threats present on every day.

Read docs/design/04-military-hazards.md.

Add to index.html:
- An Enemy base class with a behavior field, and a Projectile entity for bullets,
  dynamite, and bombs
- National Guardsman: pops from windows to shoot and throw dynamite, edible from a window,
  plus a street variant that runs in and plants a demolition charge at a building's base
- Helicopter: implement both behaviors — the early strafing run (fly over, turn, dive)
  and the later bombing run — selected by a difficulty flag
- Charges and bombs damage buildings, and any collapse they cause is attributed to them,
  not to the player. This must not award the player the building bonus later.
- A SpawnDirector with cadence values in CONFIG

Fire rates, projectile speeds, and spawn cadences aren't documented anywhere — pick
reasonable values, put them in CONFIG, and log them as inferred.

Verify against the Session 6 success criteria, then update docs/STATUS.md.
```

---

### Session 7 — Military B: tanks, police, paratroopers, lightning
*Sonnet, default effort*

```
Session 7 from docs/sessions/SESSION-PLAN.md: the rest of the threat roster.

Read docs/design/04-military-hazards.md.

Add to index.html:
- Tank: slow, armored, heavy shells with large knockback
- Police car: faster movement and fire rate than the tank
- Paratrooper: lands on a roof, defends that specific building, rapid-fires when a
  monster is on it
- Lightning cloud: drifts across and fires bolts for heavy damage
- Environmental weapons: manhole cover, flower pot, and safe can be knocked loose and
  dropped onto ground and air units
- Difficulty scalars in CONFIG covering the five escalation axes in the design doc

Knockback should be strong enough to knock a monster off a building — that's a real part
of the original's threat model.

Verify against the Session 7 success criteria, then update docs/STATUS.md.
```

---

### Session 8 — Scoring & HUD
*Sonnet, default effort*

```
Session 8 from docs/sessions/SESSION-PLAN.md: scoring and HUD.

Read docs/design/05-scoring.md and docs/design/08-hud-ui.md.

Add to index.html:
- A ScoreSystem implementing the full scoring table from the design doc, exactly as
  specified — don't round, adjust, or "balance" the arcade values
- Strict collapse attribution: the 2,500 building bonus pays only the monster that
  actually caused the collapse. A building brought down by a demolition charge or a
  helicopter bomb pays nothing.
- The designated-victim hold bonus
- A HUD with per-monster score, energy bar, and name, laid out in three columns with
  unused player slots reserved
- A real title screen and character select

Test the attribution both ways: punch a building down (2,500) and let a charge take the
same building down (0).

Verify against the Session 8 success criteria, then update docs/STATUS.md.
```

---

### Session 9 — Day system & progression
*Opus, default effort*

```
Session 9 from docs/sessions/SESSION-PLAN.md: the day system and progression.

Read docs/design/06-levels-progression.md and docs/design/08-hud-ui.md.

Add to index.html:
- LevelData: a plain data array of day definitions — building count and heights, feature
  flags, enemy set, difficulty scalars. Adding a city must require zero code changes.
- An authored first slice of days: the Peoria opening, and Plano as the 2-building day.
  The remaining cities are empty data rows to fill in later — do not author all 128.
- Day clear detection (all buildings rubble) → newspaper headline interstitial → next day
- Looping: the day index wraps, and later passes raise lightning and police counts
- The mega-vitamin event at each 128-day milestone: full heal plus 5,000 points
- Defeat → continue prompt during the walk-off window → rejoin on input
- Level features where they're cheap: bridge, riverway with underwater damage,
  train/trolley, pier and boater, plus their scoring rows

For the headlines, write a handful of short original ones in the same comedic spirit
rather than reproducing the arcade's text.

Add a debug way to jump to an arbitrary day so the 128-day milestone is testable without
playing 128 days.

Verify against the Session 9 success criteria, then update docs/STATUS.md.
```

---

### Session 10 — Local 2-player & monster-vs-monster
*Opus, default effort*

```
Session 10 from docs/sessions/SESSION-PLAN.md: local 2-player.

Read docs/design/07-multiplayer.md.

Add to index.html:
- A second player on a distinct key set, sharing one screen and one camera
- Monster-vs-monster punches dealing real damage
- Competitive pickups: first to grab wins, and a punch makes a monster drop what it's
  holding
- Cannibalism: a defeated monster's human form can be eaten by another player before it
  walks off, for points and health
- Join-in-progress: player 2 can join mid-day

This session will surface single-player assumptions baked in earlier. Fix them properly
rather than special-casing player 2 — and confirm that adding a third player still
requires only a config change, since that's the architecture we've been building toward
since Session 1.

The "punch forces a drop" rule is inferred, not confirmed. Note it accordingly.

Verify against the Session 10 success criteria, then update docs/STATUS.md.
```

---

### Session 11 — Art & audio pass
*Sonnet, default effort*

```
Session 11 from docs/sessions/SESSION-PLAN.md: the art and audio pass.

Read docs/design/09-art-audio.md.

Add to index.html:
- Monster sprites and animation states: idle, walk, climb, jump, punch, grab/hold, eat,
  hit-react, revert-to-human. George and Ralph share a body rig with a head and palette
  swap, matching how the original reused assets.
- Building damage-state tiles, rubble, and a dust puff on collapse
- Enemy and item icons, with clearly distinct on/off variants for electrical items
- An AudioManager plus the SFX set: roar, punch, collapse, explosion, eat, zap,
  day-clear sting. Synthesized via WebAudio is fine — no external asset files.
- An arcade-styled HUD pass

Keep everything procedural or inline. The game must still run from a single file opened
directly in a browser.

Check the frame rate holds at 60 with a full city and full enemy load.

Verify against the Session 11 success criteria, then update docs/STATUS.md.
```

---

### Session 12 — Balance & fidelity pass
*Sonnet at default; escalate if needed*

```
Session 12 from docs/sessions/SESSION-PLAN.md: the balance and fidelity pass.

Read docs/design/13-inferred-items.md, plus whichever system files the items touch.

Work through:
- Tune every CONFIG placeholder against actual play: damage values, health pool, regen
  amounts, movement speeds, collapse threshold, spawn cadences
- Walk the inferred-items list one at a time. Mark each resolved, tuned, or explicitly
  still-open. Don't quietly leave anything unmarked.
- Fix accumulated rough edges recorded in docs/STATUS.md from earlier sessions

Target feel: a day should be survivable but attritional. Health should matter without the
game being punishing — the original was designed for short arcade sessions.

Play through at least 5 consecutive days checking for crashes and soft-locks.

Verify against the Session 12 success criteria, then update docs/STATUS.md.
```

---

## 4. Practical workflow notes

**One session per chat.** Fresh Claude Code session each time. The plan and STATUS carry continuity, so history doesn't need to.

**Check the criteria yourself.** Claude verifying its own work is useful but not sufficient. Open the file and confirm the success criteria before moving on — a system that half-works is more expensive three sessions later.

**When a session doesn't finish:** stop it cleanly, have it update STATUS with exactly what's done and what isn't, and start the remainder fresh. Don't push a long session further.

**When you hit a bug across sessions:** paste the symptom, not the diagnosis. "The monster gets stuck when a building collapses while it's climbing" beats "I think the cling state isn't clearing" — the second one narrows the search prematurely.

**The single-file constraint has one real cost.** By Session 10, `index.html` will likely be a few thousand lines. That's fine for targeted edits, but if it starts causing trouble, the escape hatch is splitting into a few `.js` files — with the caveat that ES modules won't load over `file://`, so you'd need a local server (`python3 -m http.server`) and lose double-click-to-play. Worth keeping single-file unless it genuinely breaks down.

**Update the parking lot as you go.** Good ideas will surface mid-session. `docs/design/99-parking-lot.md` is where they go so they neither get lost nor derail Phase 1.