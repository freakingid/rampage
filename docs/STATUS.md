# Rampage Clone — STATUS

**Phase:** 1 (faithful replica)
**Last updated:** _(date)_ — end of Session 0
**Last session:** Session 0 — Repo scaffold & design-doc split
**Next session:** Session 1 — Core skeleton (loop, state machine, one controllable monster)

---

## Current state

**Playable?** Not yet — no gameplay code exists.

**What exists:**
- `docs/design/` — Phase 1 GDD split into 12 per-system files + inferred-items list
- `docs/STATUS.md` — this file
- `docs/sessions/SESSION-PLAN.md` — ordered session list with success criteria
- `CLAUDE.md` — project rules
- `index.html` — not yet created

**Systems implemented:** none.

| System | State |
|---|---|
| Game loop & state machine | ⬜ not started |
| Monster movement | ⬜ not started |
| Buildings & climbing | ⬜ not started |
| Destruction & collapse | ⬜ not started |
| Eating & health | ⬜ not started |
| Civilians & designated victims | ⬜ not started |
| Military — Guardsmen, helicopters | ⬜ not started |
| Military — tanks, police, paratroopers, lightning | ⬜ not started |
| Scoring & HUD | ⬜ not started |
| Day system & progression | ⬜ not started |
| 2-player & monster-vs-monster | ⬜ not started |
| Art & audio pass | ⬜ not started |

---

## What changed last session

Session 0 (setup only, no gameplay code):
- Split the single Phase 1 GDD into per-system files under `docs/design/`
- Created `CLAUDE.md`, `docs/STATUS.md`, `docs/sessions/SESSION-PLAN.md`

---

## What's next

**Session 1 — Core skeleton.** Fixed-timestep loop, game-state machine, `CONFIG` object, `Entity` base, and one controllable monster walking/jumping on a flat street. No buildings, no enemies.

Success criteria are in `docs/sessions/SESSION-PLAN.md`.

---

## Open questions / things guessed at

Anything implemented from an inference rather than a confirmed source gets logged here **and** in `docs/design/13-inferred-items.md`.

| # | Item | Where | Status |
|---|---|---|---|
| 1 | Damage values, health-bar size, regen amounts | `CONFIG` | not yet implemented — placeholder pending |
| 2 | Movement speed, jump arc, punch cadence | `CONFIG` | not yet implemented — tune to feel |
| 3 | Building collapse threshold | destruction | not yet implemented |
| 4 | Enemy fire rates, projectile speeds, spawn cadences | `CONFIG` | not yet implemented |
| 5 | Health meter as bar vs. descriptive labels | HUD | decided: plain bar for Phase 1 |
| 6 | Monster palettes / exact colors | art | deferred to art pass |
| 7 | "Holding designated victim → only Guardsmen stop firing" | civilians | single-source claim, implement as written |
| 8 | Monster-vs-monster "punch forces drop" | multiplayer | implement drop-on-hit, verify later |

---

## Session log

| # | Session | Date | Outcome |
|---|---|---|---|
| 0 | Repo scaffold & design-doc split | | ✅ |

---

## Notes for the next session

_(Anything the next session needs to know that isn't obvious from the code — deferred decisions, known rough edges, things deliberately left unfinished.)_

- Nothing yet.