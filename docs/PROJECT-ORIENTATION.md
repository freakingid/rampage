# Rampage Clone — Project Orientation

Read this first. It explains what this project is and what's available to you.

## The project

A browser-based, close-to-exact clone of the 1986 Bally Midway arcade game *Rampage*.
HTML5 Canvas + vanilla JavaScript, single self-contained `index.html`, no build step.
Solo developer. Implementation happens in **Claude Code**; this claude.ai Project is where design and planning happen.

**Current phase:** Phase 1 — faithful replica only. New mechanics and objects are deferred to later phases and live in the design doc's parking lot.

## Two separate systems

| | claude.ai Project (here) | Claude Code (the repo) |
|---|---|---|
| Role | Design HQ — design decisions, planning, debugging discussion, writing session specs | Build HQ — writes and edits the actual game code |
| Context | Project knowledge + whatever I attach | `CLAUDE.md` + files it reads on demand |

They share nothing automatically. Decisions made here only reach Claude Code if they get written into the design doc, the session plan, or a session prompt.

## What's in Project knowledge

- **The Game Design Document** — ground truth for Phase 1. Twelve sections plus a list of inferred items (§13) covering everything guessed at rather than confirmed. Don't re-derive or "improve on" mechanics it already specifies.
- **SESSION-PLAN.md** — the 13 ordered build sessions, each with success criteria and scope boundaries.
- **IMPLEMENTATION-NOTES.md** — paste-ready Claude Code prompts per session, plus model/effort guidance.
- **CLAUDE.md** — the rules Claude Code operates under in the repo.

## What is NOT here, and why

`docs/STATUS.md` and `index.html` change every session, so they're attached per conversation rather than stored here — a stale copy would be actively misleading. **If a question depends on the current build state and neither is attached, ask for them rather than guessing.**

## Repo layout

```
rampage/
├── CLAUDE.md
├── index.html                     ← the entire game
└── docs/
    ├── STATUS.md                  ← running log; attach to chats
    ├── RAMPAGE-GDD-PHASE1.md      ← archived single-file original
    ├── IMPLEMENTATION-NOTES.md
    ├── design/                    ← GDD split per system, read selectively by Claude Code
    └── sessions/SESSION-PLAN.md
```

## Conventions worth knowing

- **One mechanic per session.** If something spans several, propose splitting it.
- **Fidelity marking:** `[FAITHFUL]` = confirmed against sources. `[INFERRED]` = a reasoned guess. Never present an inference as settled; new inferences get logged to `docs/design/13-inferred-items.md`.
- **All three monsters are mechanically identical** in the 1986 original — same speed, jump, punch damage, health. Per-monster stats come from *Rampage: World Tour* (1997) and are out of scope. This misconception is widespread in FAQs and port manuals; don't reintroduce it.
- **All tunable numbers live in one `CONFIG` object** in `index.html`. Nothing tunable is hardcoded inline.
- **Input, HUD, and scoring are built for 3 players** even though Phase 1 only wires up 2, matching the arcade original.

## Maintenance

When the GDD or session plan is revised, **replace** the copy here rather than uploading a second version — two copies drifting apart is the main failure mode for this setup.