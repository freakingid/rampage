<!-- Source: RAMPAGE-GDD-PHASE1.md — Section 13: Inferred items to verify (running list) -->

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
11. **Movement during a punch** — the GDD says nothing about whether a monster can walk while punching. Session 1 implements: punching **roots** the monster while it's on the ground (`vx = 0` for the punch duration), but airborne momentum is kept. **Session 2 extends the same rule to the two new surfaces**: punching also pins a monster clinging to a face and one standing on a roof. Verify against the arcade. (§3.1, §3.2)
12. **Air control** — no documented rule for steering mid-jump. Session 1 uses `CONFIG.monster.airControl = 0.9` (90% of walk speed while airborne). Interacts with jump-to-grab in Session 2. (§3.1)
13. **Climb speed** — no documented rate for climbing a face. Session 2 uses `CONFIG.monster.climbSpeed = 60` px/s, slower than the 84 px/s walk. Pure feel; retune in Session 12. (§3.1)
14. **Lateral movement while clinging** — §3.1 only says "up/down climbs it". Session 2 also lets a clinging monster traverse **sideways** along the face (`climbSideSpeed = 60` px/s), clamped so the body can never slide off the frontage. Rationale: the 8-way stick, and punching cells across a building's width (§3.2) needs it. Verify whether the arcade allowed it, and at what speed. (§3.1, §3.2)
15. **The street is a lane in front of the buildings** — the GDD never says whether a building's base blocks ground movement. Session 2 implements it as a lane: a monster walks the full street past every building and only interacts with one by climbing it. Consequence: buildings are solid only for CLING (body pinned flush to the face) and ROOF (feet on the roof line). Believed faithful — the original draws monsters in front of building bases — but it is an inference. To keep this consistent, **every building must be taller than the jump apex** so a street jump can never land on a roof; `CONFIG.buildings.dayLayout` is constrained accordingly. (§3.1)
16. **Roof-edge and detach rules** — undocumented details Session 2 had to pick: a monster leaves a roof when its **centre** passes the frontage edge (so it can stand half over the edge first); jumping off a face applies a short re-grab lockout (`grabLockout = 0.18 s`) so a jump-to-grab doesn't instantly re-attach to the wall just left; and jump-to-grab needs `grabMinOverlap = 10` px of body over the frontage. All feel values. (§3.1)

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
