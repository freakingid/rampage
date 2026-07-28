<!-- Source: RAMPAGE-GDD-PHASE1.md — Section 2: Monsters (George, Lizzie, Ralph) -->

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
