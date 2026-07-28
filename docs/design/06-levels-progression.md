<!-- Source: RAMPAGE-GDD-PHASE1.md — Section 7: Level structure & progression -->

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
