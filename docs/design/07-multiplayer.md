<!-- Source: RAMPAGE-GDD-PHASE1.md — Section 8: Multiplayer (local, matching the original) -->

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
