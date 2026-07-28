<!-- Source: RAMPAGE-GDD-PHASE1.md — Section 1: Core loop -->

## 1. Core loop

Moment to moment, the player **is a giant monster demolishing a city block** while an escalating military tries to stop them. The loop is:

1. **Destroy buildings** — climb them, punch chunks out of them, and bring them down. A day (level) ends when every building on screen is rubble.
2. **Feed to survive** — punching buildings exposes food and civilians inside; eating them refills the health/energy bar that enemy fire drains.
3. **Fend off the military** — soldiers, vehicles, and aircraft attack continuously; the player dodges, out-positions, and swats them (often using the environment).
4. **Clear the day → next city** — a brief newspaper-headline interstitial, then a new cityscape with slightly more threat.

Why it's fun (design pillars to protect): **cathartic destruction** (everything on screen can be smashed), **legible chaos** (a 2D side view where the whole board is visible), **risk/reward tempo** (stopping to eat is safety *and* vulnerability), and **social play** (2+ monsters cooperating and sabotaging each other on the same screen). There is **no puzzle and no fail state for "wrong" actions** — the only lose condition is running out of health. Keep sessions short and immediately re-enterable. **[FAITHFUL]**

**Phase 1 scope:** implement the full loop for a single day and looping day-to-day progression (see §7). Everything the loop needs — buildings, eating, health, military, scoring, HUD, 1–2 players — is in scope. Nothing outside the loop is.
