<!-- Source: RAMPAGE-GDD-PHASE1.md — Section 4: Eating & health -->

## 4. Eating & health

### 4.1 The health/energy system

Each monster has a single **energy bar** (health). Enemy attacks and hazards drain it; eating refills it. **When the bar empties, the monster reverts to its (naked) human form and walks off the side of the screen** — that monster is out. **[FAITHFUL]**

**Damage sources** (how the bar goes down):
- Being **shot** by National Guardsmen (bullets) and hit by thrown **dynamite**.
- Being hit by **tank / police shells**.
- **Falling** — from a collapsing building, or being **knocked off** by an attack.
- **Punches from another monster** (multiplayer — §8).
- **Lightning** strikes; **electrocution** from grabbing "live" electrical items (§4.3).
- **Going underwater** (river/pier levels).
- **Eating a bad item** (§4.3). **[FAITHFUL]**

**[INFERRED]** exact per-source damage values, total bar size, and regen amounts are not documented; Phase 1 uses tuned placeholder values (see §13). Tune so a day is survivable but attrition is real.

**[INFERRED]** The original may have shown descriptive meter states ("Excellent/Groggy/Weak"–style labels) — that's confirmed for *World Tour*, unclear for the 1986 arcade. Phase 1 default: a simple depleting bar; labels optional (§13).

### 4.2 Food & healthy items (restore health)

Found inside buildings (exposed by punching) and in windows/rubble. Eating restores a bit of energy:
**turkey** (cooling in windowsills), **hamburger** (mid-building), **milk**, **bowl of fruit** (bananas/apples), **toast** (wait for it to pop from a toaster). **[FAITHFUL]**

### 4.3 Bad items & environmental hazards inside buildings (drain health)

Grabbing/eating these hurts the monster — and because punching is fast, a greedy player will sometimes grab the wrong thing:
- **Poison** (skull-and-crossbones) — big health loss.
- **Cactus** — doesn't agree with the monster.
- **Dynamite** in a wall — on exposure you have ~2–3 seconds to leave before it blasts you off the building; eating it just hurts you.
- **Live electrical items** — **lit light bulb**, **TV that's on**, **toaster with no toast**, **neon sign** — shock the monster. (When *off*, TV/light bulb/neon can instead be knocked out for points — §6.)
- **Photographer** (in windows) — a camera flash knocks the monster off the building; eat them before the flash.
- **Bathtub with a person** — the bather blasts water to knock the monster off; eat them fast.
- **Toilet / empty bathtub** — no benefit if eaten (junk grabs). **[FAITHFUL]**

**Phase 1 scope:** implement food (heal), the bad/electrical items (damage/shock), photographer and bathtub "knock-off" behaviors, and dynamite's exposure-timer. Civilians and soldiers as edible are covered in §5/§6 (eating a soldier both removes a threat and heals — keep that dual role).
