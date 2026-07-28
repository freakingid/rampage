<!-- Source: RAMPAGE-GDD-PHASE1.md — Section 3: Movement & destruction -->

## 3. Movement & destruction

**Controls (per monster):** an 8-way direction input + **Punch** + **Jump**. (See §9/§11 for keyboard mapping.) **[FAITHFUL]**

### 3.1 Movement & traversal

- **Ground:** left/right walks along the street.
- **Climbing:** when adjacent to a building face, up/down climbs it; the monster clings to the wall.
- **Jump:** vertical or directional leap. **Jump toward a building and hold up to grab it** — this is the fast way to cross from building to building and to start a climb mid-air.
- **Rooftops:** the monster can stand and walk on the top of a building until it's destroyed. **[FAITHFUL]**

### 3.2 Destroying a building

A building is a **vertical stack of destructible cells** (a grid of windows/wall sections). It comes down through accumulated damage:

- **Punch a wall section** (from the ground or while clinging to the face) to rip out that section. A punch that only cracks a section is a *partial* break; a punch that opens a full hole is a *full* break (these score differently — §6). Repeated hits destroy successive sections. **[FAITHFUL]**
- **Roof-drop collapse:** weaken a building, then **jump onto its roof** — the monster's weight finishes it off and it collapses. **[FAITHFUL]**
- **Third-party collapse:** buildings can also be brought down by **National Guard demolition charges at the base**, **helicopter bombs**, and **dynamite** hidden in walls. Important scoring rule: **the 2,500-point "building destroyed" bonus is awarded only if the player's own monster caused the collapse** (§6). **[FAITHFUL]**

### 3.3 Collapse trigger & consequences

- **Trigger:** a building collapses once enough of its structure is gone (a critical amount of cell damage / loss of lower support). **[INFERRED]** — the exact threshold isn't documented. Phase 1 model: each cell has HP; when total remaining structure drops below a threshold **or** the bottom row is destroyed, the building enters a short "shaking" state, then collapses into a rubble pile with a dust puff.
- **Rubble:** collapsed buildings leave a rubble mound on the street that can still contain exposed food/items to grab.
- **Falling damage:** if a monster is **still clinging to (or standing on) a building when it collapses, it takes a large chunk of damage** from the fall. The shaking state is the tell — jump clear in time to avoid it. **[FAITHFUL]**
- **Day clears** when all buildings on the board are rubble (§7). **[FAITHFUL]**

**Phase 1 scope:** cell-grid buildings with partial/full break states, roof-drop collapse, third-party collapse with correct point attribution, shaking→collapse with fall damage. Multiple building heights/widths; feature tiles (bridges, trains, piers) are level dressing handled in §7.
