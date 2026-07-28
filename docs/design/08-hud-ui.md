<!-- Source: RAMPAGE-GDD-PHASE1.md — Section 9: HUD / UI -->

## 9. HUD / UI

On-screen, always visible (top of screen, one column per active monster): **[FAITHFUL]**

- **Per-monster score.**
- **Per-monster energy/health bar** (with the monster's name/label).
- (P3 column reserved even in 2P builds — leave layout room.)

Other UI states:
- **Title / attract screen** (Phase 1: minimal title + "press to start").
- **Character select** — pick any of George/Lizzie/Ralph per player; unused slots joinable later.
- **Day interstitial** — city name + comedic newspaper headline (§7).
- **Continue prompt** — appears during a defeated monster's walk-off window (§7.4).
- **Mega-vitamin bonus flourish** at each 128-day milestone.

**[INFERRED]:** exact arcade HUD art/fonts and whether health shows descriptive labels (§4.1) — Phase 1 uses clean, readable placeholders (bars + numeric score). Keep HUD rendering isolated so a faithful art pass can drop in later. Design the HUD to read clearly for **up to 3 columns** from the start.

**Phase 1 scope:** score + health bars for all active monsters, title, character select, day interstitial, continue prompt, bonus flourish. No pause menus/options screens required.
