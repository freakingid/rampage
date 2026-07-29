<!-- Source: RAMPAGE-GDD-PHASE1.md — Section 13: Inferred items to verify (running list) -->

## 13. Inferred items to verify (running list)

Everything Phase 1 is guessing at rather than confirming — dial these in during implementation or verify against the arcade ROM:

1. **Monster mechanical parity** — treated as *identical* (faithful to 1986; the "different stats" belief is from *World Tour*/port manuals). High confidence; confirmable in MAME. (§2.1)
2. **Damage values / health-bar size / regen amounts** — not documented; placeholder in `CONFIG`. (§4.1)
3. **Movement speed, jump arc, punch cadence** — not documented numerically; tune to feel. (§3, §11.4)
4. **Health-meter representation** — bar vs. descriptive labels ("Excellent/Groggy/…"); labels confirmed for *World Tour* only. Phase 1: bar. (§4.1, §9)
5. **Building collapse threshold** — exact trigger undocumented; using per-cell HP + bottom-row/critical-mass rule. (§3.3) **Session 3 picked the numbers.** `CONFIG.buildings`: `collapseStructureFraction = 0.5` (half the building's total cell HP gone brings it down), `collapseOnBottomRowGone = true` (every bottom-row cell GONE brings it down regardless of the rest), `shakeDuration = 1.2 s` for the SHAKING window, and `roofDropStructureFraction = 0.8` for §3.2's roof-drop. Consequences on the current day layout: the bottom-row route costs 9–15 connecting punches and is deliberately the fastest — it is the same "loss of lower support" the Guard's demolition charges exploit in §3.2 — the roof-drop route costs 12–18 punches plus a landing, and punching a building apart anywhere costs 30–45. All four numbers are judgement calls; Session 12 retunes them. Also inferred within this: **`cellHp` (3) is spread evenly across the three non-GONE damage states**, so one punch = one state at the current value and the constant stays free to change.
6. **Monster palettes/exact colors** — sources conflict (esp. George); pick distinct palettes, treat exact hues as polish. (§10)
7. **"Holding designated victim → only Guardsmen stop firing"** — from a single walkthrough; implement as the rule, verify. (§2.3)
8. **Monster-vs-monster "punch to force drop"** — implemented as drop-on-hit; verify exact original behavior. (§8.2)
9. **Full city list authoring** — Phase 1 ships the day *system* + a first slice of authored days (Peoria opening, Plano as the 2-building day); remaining 128 cities are content to fill in. (§7.2)
10. **Enemy fire rates / projectile speeds / spawn cadences** — undocumented; tune. (§5.1)
11. **Movement during a punch** — the GDD says nothing about whether a monster can walk while punching. Session 1 implements: punching **roots** the monster while it's on the ground (`vx = 0` for the punch duration), but airborne momentum is kept. **Session 2 extends the same rule to the two new surfaces**: punching also pins a monster clinging to a face and one standing on a roof. Verify against the arcade. (§3.1, §3.2)
12. **Air control** — no documented rule for steering mid-jump. Session 1 uses `CONFIG.monster.airControl = 0.9` (90% of walk speed while airborne). Interacts with jump-to-grab in Session 2. (§3.1)
13. **Climb speed** — no documented rate for climbing a face. Session 2 uses `CONFIG.monster.climbSpeed = 60` px/s, slower than the 84 px/s walk. Pure feel; retune in Session 12. (§3.1)
14. **Lateral movement while clinging** — §3.1 only says "up/down climbs it". Session 2 also lets a clinging monster traverse **sideways** along the face (`climbSideSpeed = 60` px/s), clamped so the body can never slide off the frontage. Rationale: the 8-way stick, and punching cells across a building's width (§3.2) needs it. Verify whether the arcade allowed it, and at what speed. (§3.1, §3.2)
15. **The street is a lane in front of the buildings** — the GDD never says whether a building's base blocks ground movement. Session 2 implements it as a lane: a monster walks the full street past every building and only interacts with one by climbing it. Consequence: buildings are solid only for CLING (body pinned flush to the face) and ROOF (feet on the roof line). Believed faithful — the original draws monsters in front of building bases — but it is an inference. To keep this consistent, **every building must be taller than the jump apex** so a street jump can never land on a roof; `CONFIG.buildings.dayLayout` is constrained accordingly. (§3.1)
16. **Roof-edge and detach rules** — undocumented details Session 2 had to pick: a monster leaves a roof when its **centre** passes the frontage edge (so it can stand half over the edge first); jumping off a face applies a short re-grab lockout (`grabLockout = 0.18 s`) so a jump-to-grab doesn't instantly re-attach to the wall just left; and jump-to-grab needs `grabMinOverlap = 10` px of body over the frontage. All feel values. (§3.1)
17. **Which cell a punch lands on** — new in Session 3. §3.2 says a punch "rips out that section" but never says which section. Implemented as: the punch's target rect runs from the monster's **own centre** to `punchReach` past its leading edge, and the cell hit is the still-standing cell that rect overlaps most (tie-broken toward the facing direction); already-GONE sections are skipped rather than absorbing the punch, so repeated hits chew into the section behind. Reaching back to the centre rather than starting at the leading edge is load-bearing: a clinging monster is pinned flush inside the frontage, so a purely forward reach would punch clean past the building at the outermost column and that column could never be destroyed from a face. Two side effects worth verifying against the arcade: a monster standing on the street punches the **second** row up, not the bottom row, because it is more than two storeys tall and punches at chest height (the bottom row goes next, once that one is gone); and a punch thrown from the AIR damages cells too, since no stance restriction is applied. (§3.2)
18. **Score for removing an already-holed section** — new in Session 3. §6 has exactly two punch rows, "partial break" (25) and "full break / hole" (225). The third transition, HOLE → GONE, has no row. `Building.damageCell` returns `'partial'` on the first crack, `'full'` when the hole opens, and `null` for the final removal — i.e. the literal reading, where clearing a holed section scores nothing. Session 8 owns the decision when it wires up the ScoreSystem. (§3.2, §6)
19. **Rubble is scenery** — new in Session 3. §3.3 says a collapsed building "leaves a rubble mound on the street that can still contain exposed food/items to grab", and says nothing about whether it blocks or supports anything. Phase 1 implements it as pure scenery: it cannot be climbed, cannot be landed on, and does not block ground movement — consistent with the street-as-a-lane model in #15. Session 4 hangs exposed items off it. (§3.3)
20. **Fall damage is a straight drop** — new in Session 3. §3.3 says a monster caught on a collapsing building "takes a large chunk of damage from the fall" but describes no knockback. Implemented as: detach to AIR with `vy = 0` and let gravity do the rest — no upward pop, no lateral throw. The damage value (`CONFIG.monster.fallDamage = 30`) is a placeholder pending Session 4's health economy; see #2. (§3.3)

---

## Not arcade inferences — port decisions

These aren't guesses about the 1986 original; they're choices the original had no equivalent for. Recorded so they aren't mistaken for fidelity claims.

- **Keyboard mapping** — the arcade used an 8-way joystick + Punch + Jump per player. Phase 1 maps P1 to WASD/F/G, P2 to arrows/`.`/`/`, P3 to IJKL/O/P (`CONFIG.players.slots`, Session 1).
- **Playfield resolution** — 512×384 logical px, integer-upscaled to the window. MCR-3 hardware was 512×480; the shorter playfield is a windowing choice, not a fidelity claim. (`CONFIG.view`, Session 1)

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
