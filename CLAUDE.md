# Rampage Clone — Phase 1 (faithful replica)

## What this is
A close-to-exact HTML5 Canvas + vanilla JS clone of the 1986 Bally Midway arcade game *Rampage*.
Phase 1 = faithful replica only. New mechanics and objects come in later phases — not yet.

**Build target:** one self-contained `index.html` (inline `<canvas>` + `<script>`), no build step, no dependencies. It must run by opening the file directly in a browser.

## Where things live
- `index.html` — the game. The only source file.
- `docs/design/` — the design doc, split by system. **Read only the file(s) the current task needs.**
  - `00-core-loop.md` · `01-monsters.md` · `02-movement-destruction.md` · `03-eating-health.md`
  - `04-military-hazards.md` · `05-scoring.md` · `06-levels-progression.md` · `07-multiplayer.md`
  - `08-hud-ui.md` · `09-art-audio.md` · `10-architecture.md` · `99-parking-lot.md`
  - `13-inferred-items.md` — every mechanic we guessed at rather than confirmed.
- `docs/STATUS.md` — running log. Current state, last session's changes, what's next, open questions.
- `docs/sessions/SESSION-PLAN.md` — the ordered session list and each session's success criteria.

## Rules

**Scope**
- The design doc is ground truth. Don't re-derive, second-guess, or "improve on" mechanics it specifies.
- Stay inside Phase 1. Good ideas beyond it go to `docs/design/99-parking-lot.md`, not into code.
- One feature/mechanic per session. If a request spans more than one, say so and propose a split before writing code.
- Don't build systems no current Phase 1 mechanic uses. No frameworks, no ECS, no asset pipeline, no networking.

**Code**
- Make targeted edits. Don't regenerate `index.html` wholesale unless a rewrite is genuinely necessary — and say why first.
- Keep `index.html` organized in commented sections that mirror the `docs/design/` split, so any system is findable by name.
- All tunable numbers (damage, health, speeds, timings, spawn rates, thresholds) go in the single `CONFIG` object at the top. Never hardcode a tunable inline.
- New entity types extend the existing base `Entity` — that's the Phase 2 extension seam. Don't fork parallel hierarchies.
- Input, HUD, and scoring are built for **up to 3 players** even though only 2 are wired up in Phase 1. Never hardcode a 2-player assumption.

**Fidelity**
- If you're inferring how an original Rampage mechanic worked rather than confirming it, say so explicitly — in chat, in a code comment, and in `docs/design/13-inferred-items.md`. Never present a guess as settled.
- Don't invent mechanics from *Rampage: World Tour* (1997) or any home port. Only the 1986 arcade original counts. In particular: **all three monsters are mechanically identical** — same speed, jump, punch damage, and health. Per-monster stats are a World Tour thing and are out of scope.

**Session hygiene**
- Update `docs/STATUS.md` before ending a session: current state, what changed, what's next, anything guessed at.
- Verify your work runs before declaring done — open the file, check the console is clean, confirm the session's success criteria in `SESSION-PLAN.md` actually pass.
- Say so if a session is running long enough that a clean stop and a fresh session would be cheaper.

**Communication**
- Be concise: what changed and why. Save detail for genuinely non-obvious decisions.
- Flag it rather than guessing silently when the design doc is ambiguous or silent on something you need.