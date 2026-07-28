<!-- Source: RAMPAGE-GDD-PHASE1.md — Section 5: Military & hazards (threat roster + escalation) -->

## 5. Military & hazards (threat roster + escalation)

The military attacks continuously. Present **every day**: National Guardsmen and helicopters. Other units appear per the day schedule (§7). **[FAITHFUL]**

### 5.1 Enemy roster & attack patterns

| Enemy | Behavior | Notes |
|---|---|---|
| **National Guardsman** | Pops out of building windows to **shoot** and **throw dynamite**; some run in from the screen edge to plant **demolition charges at a building's base**. | The bread-and-butter threat, everywhere. Can be eaten out of windows (removes threat + heals). |
| **Helicopter** | Air support, every day. **Early:** flies over, turns, and **dives to strafe** (rapid burst). **Later:** **drops bombs** from altitude. | Can be destroyed on its approach. Bombs can be baited onto buildings to collapse them. |
| **Tank** | Slow, heavily armored; fires **heavy shells** that do big damage and **large knockback** (can knock a monster off a building or across the screen). | Punish between shots or drop objects on it. |
| **Police car** | Like a tank but **moves and fires faster**. | |
| **Paratrooper** | Lands on a rooftop and **defends that building**; **rapid-fires** when the monster is on it. | Scale fast and eat it. |
| **Photographer** | (Also a building hazard, §4.3) flash **knocks the monster off** the building. | |
| **Lightning cloud** | Drifts over the city and fires **lightning bolts**; heavy damage on hit. | A hazard rather than a unit; scheduled per day. |
| **Boats / boater** | On pier/riverway days; smashable. | Level-feature-dependent. |
| **Trains / trolleys** | On train-feature days; smashable back and forth ("train pong" in multiplayer). | |

**[FAITHFUL]** for all rows. **[INFERRED]:** precise fire rates, projectile speeds, spawn cadences, and damage-per-hit are not documented — tune during implementation (§13).

### 5.2 How threat escalates over time

Difficulty ramps across days along five axes (all **[FAITHFUL]** in direction; exact rates **[INFERRED]**):
1. **More buildings** to clear (roughly 3 early → 4–6 later).
2. **More and faster ground units** — tanks and police introduced and stacked.
3. **Helicopters shift from strafing to bombing.**
4. **Lightning appears and recurs more often.**
5. **On later loops of the city list, more lightning and more police** than the first pass.

**Phase 1 scope:** implement Guardsmen + helicopters (both strafe and bomb behaviors), tanks, police, paratroopers, lightning, and the "environmental weapon" interactions (drop manhole cover / flower pot / safe onto ground/air units — §6). Boats/trains are tied to level features (§7) — include if their feature is implemented in Phase 1; otherwise flag as deferred there.
