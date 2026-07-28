<!-- Source: RAMPAGE-GDD-PHASE1.md — Section 10: Art & audio direction -->

## 10. Art & audio direction

**Recommended direction: a faithful pixel-art homage** — cartoony, exaggerated, comedic (monsters mug at the "camera"; the tone is B-movie, not horror). This is both true to the original and the most practical for a solo dev, because it needs only small, chunky sprites and tiled buildings. **[FAITHFUL tone]**

**Practical art plan for a solo dev:**
- **Monsters:** one silhouette-distinct sprite set each. Exploit the original's trick — **George and Ralph can share a body rig with a head/palette swap**; Lizzie is the distinct one. Animations needed: idle, walk, climb, jump, punch, grab/hold, eat, hit-react, and the **revert-to-human walk-off**.
- **Buildings:** tile-based facades with **damage states per cell** (intact → cracked → hole → gone), plus a rubble mound and a **dust-puff** on collapse (the dust also conveniently hides animation seams — the original did exactly this).
- **Enemies/items:** small readable icons; palette-driven "on/off" variants for electrical items (lit vs dark) so the hazard read is instant.
- **[INFERRED]:** exact arcade palettes (e.g., George's body color is described inconsistently across sources) — pick a clear, distinct palette per monster and treat exact-hue fidelity as a later polish item (§13).

**Audio (keep minimal in Phase 1):** monster roars, punch/impact, building-collapse rumble, explosion, eat/heal blip, damage/shock zap, and a short day-clear sting. Placeholder or lightweight synthesized SFX are fine for Phase 1; a faithful pass can come later. Keep an audio manager abstraction so swapping assets is trivial (§11).

**Phase 1 scope:** functional, consistent placeholder-to-homage art and a small SFX set. Do not block gameplay work on art polish. No music required for Phase 1.
