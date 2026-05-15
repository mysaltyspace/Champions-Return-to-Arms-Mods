# Champions: Return to Arms — PCSX2 Mods

PCSX2 `.pnach` patch mods for **Champions: Return to Arms** (PS2, NTSC-U).

**Game serial:** `SLUS-20973` | **CRC:** `4028A55F`

---

## Mods

### INT Scaling
`custom patches/SLUS-20973_4028A55F.SpellDamageScaling.pnach`

Makes spell damage scale with the caster's **Intelligence (INT)** stat using diminishing returns. Works in 1–4 player sessions — each player's spells scale off their own INT.

#### Scaling table

| INT | Bonus |
|-----|-------|
| 20  | +0% (base threshold) |
| 50  | +15% |
| 100 | +40% |
| 150 | +60% |
| 200 | +80% |
| 300 | +110% |
| 400 | +130% |

Tiers: **+0.5%/pt** from 20–100 · **+0.4%/pt** from 100–200 · **+0.3%/pt** from 200–300 · **+0.2%/pt** above 300

#### What scales
- Skills that directly deal damage or DoT such as Bolt of Shock, Cone of Fire, Cone of Frost, Fire Storm, Wizard Beam, Tagar's Insects, Roar (Berserker), Holy Strike (Cleric), etc.

#### What does NOT scale
- Basic attacks (blunt, slashing, bows, thrown)
- Shield Bash — has its own optional armor scaling mod (see below)
- Weapon enchantment elemental damage — has its own separate mod (see below)
- Socketed gem elemental damage (unaffected by design)
- Disease/Poison Weapons elemental damage (base game bug — deals no elemental damage; fixed by the **Disease Blade/Poison Weapons Fix** in `SLUS-20973_4028A55F.WeaponEnchantFixAndScaling.pnach`)

> **Note:** This patch is a work in progress. If you find a damage type that is incorrectly scaled or not scaled, please open an issue.

---

### Weapon Enchant INT Scaling
`custom patches/SLUS-20973_4028A55F.WeaponEnchantFixAndScaling.pnach`

Makes the elemental damage bonus from **Fire Weapons**, **Cold Weapons**, and **Lightning Weapons** spells scale with **Intelligence (INT)**. Uses the same tiered diminishing-returns formula as the INT Scaling mod. Works independently — enable with or without INT Scaling.

#### What scales
- Fire Weapons elemental hit bonus
- Cold Weapons elemental hit bonus
- Lightning Weapons elemental hit bonus

#### What does NOT scale
- Socketed gem elemental damage (intentionally excluded — different damage source)
- Disease/Poison Weapons — see Disease Blade/Poison Weapons Fix section below
- Base weapon physical damage

#### How it works
Three hook points intercept the game's elemental-merge instruction for each enchant type. At each hook, a small scanner reads the weapon's enchant slot array (up to 4 slots per weapon) and checks the type code stored in each slot. If a Fire/Cold/Lightning Weapons spell buff is found in any slot, INT scaling is applied; if only socket gems are present the damage passes through unchanged. This handles any weapon in your inventory regardless of how many enchants are active or which slot the buff occupies. Each hook then reads P1's INT and applies the same four-tier bonus before merging the scaled value into the total damage.

#### Scaling table (same as INT Scaling)

| INT | Bonus |
|-----|-------|
| 20  | +0% (base threshold) |
| 50  | +15% |
| 100 | +40% |
| 150 | +60% |
| 200 | +80% |
| 300 | +110% |
| 400 | +130% |

> **Note:** Each player's elemental damage scales off their own INT. Verified working in 2-player; 3- and 4-player use the same formula.

---

### Disease Blade/Poison Weapons Fix
`custom patches/SLUS-20973_4028A55F.WeaponEnchantFixAndScaling.pnach` — second checkbox

Fixes two bugs affecting **Disease Blade** (Shadow Knight) and **Poison Weapon** (Shaman):

**On-hit damage fix** — both spells deal zero elemental damage in the base game. A flag in the player entity struct has bit 17 set for disease/poison hits, causing `z_un_001e8230` (the universal weapon-enchant elemental handler) to exit early before computing any damage. This patch NOPs that early-exit branch (`001E82A0`) and adds a damage-computation hook (`001E9818`) using the same LCG RNG and resistance formula as the Fire/Cold/Lightning handlers.

**Ally buff timer fix** — the ally AoE version of both spells flickers on and off while allies stand near the caster. Two root causes: (1) the buff-application function only runs once every 30 frames, creating a 0.5s re-application gap; (2) the ally buff timer is written from a weapon-stat-derived value (~52 ticks) that expires before the next re-application cycle. This patch removes the 30-frame throttle and sets the ally timer to 60 ticks (~1s). The buff is now stable while the ally is in range and drops within ~1s when they move out of range or the spell ends.

---

### Shield Bash Defense Scaling
`custom patches/SLUS-20973_4028A55F.SpellDamageScaling.pnach` — second checkbox in PCSX2

Makes **Shield Bash** damage scale with **total armor** (Cleric, Barbarian, Shadow Knight). Rewards armor-focused builds without making the ability overpowered at low gear levels.

> **Requires the INT Scaling checkbox to also be enabled** — the Shield Bash section reuses the INT Scaling hook and register save/restore code.

#### Formula

```
damage = (base + 10 + 0.04 × armor) × (1.7 + 0.0015 × armor)
```

The flat `+10` and `×1.7` base give a small early-game bump even with no armor investment. The armor terms grow the damage progressively as you stack defense.

#### Scaling examples (Shield Bash Lv20, ~125 base damage)

| Total armor | Damage |
|-------------|--------|
| 0           | ~230   |
| 200         | ~286   |
| 400         | ~347   |
| 524         | ~388   |
| 800         | ~484   |
| 1000        | ~560   |

---

### 2H Weapon Damage Bonus
`custom patches/SLUS-20973_4028A55F.WeaponRebalance.pnach` — first checkbox

Two-handed melee weapons deal **+20% damage** across the board — physical and elemental. Bows and thrown weapons are unaffected.

#### What gets the bonus
- **Physical damage** — min and max are both scaled before the random roll and before crit
- **Elemental damage from socket gems** — fire, cold, lightning sockets on the equipped weapon
- **Elemental damage from spell enchants** — Fire Weapons, Cold Weapons, Lightning Weapons
- **Disease Blade / Poison Weapon elemental** — if the Disease Blade/Poison Weapons Fix is also enabled

#### What does NOT get the bonus
- 1H melee weapons
- Bows, thrown weapons
- Spells (use INT Scaling mod for those)

#### Stacking with Weapon Enchant INT Scaling
When both mods are active, elemental enchant damage stacks multiplicatively:
`elemental = base × 1.2 × (1 + INT_bonus)`

---

### 1H Attack Speed Bonus
`custom patches/SLUS-20973_4028A55F.WeaponRebalance.pnach` — second checkbox

One-handed melee weapons swung without a shield or off-hand weapon attack **25% faster**. Stacks additively with Battle Cry and other speed buffs (same as one rank of Battle Cry 25%).

Dual-wielding and shield users are unaffected — the bonus only applies when the off-hand slot is empty.

---

## Installation

1. Copy the desired `.pnach` file(s) to your PCSX2 **`patches/`** folder
   - Default location: `Documents/PCSX2/patches/`
   - Do **not** put them in the `cheats/` folder
2. In PCSX2, open **Game Properties → Patches** and check each patch you want enabled
3. Do a fresh boot of the game

### Patch files

| File | Contents |
|------|----------|
| `SLUS-20973_4028A55F.SpellDamageScaling.pnach` | INT Scaling + Shield Bash Defense Scaling (two checkboxes) |
| `SLUS-20973_4028A55F.WeaponEnchantFixAndScaling.pnach` | Weapon Enchant INT Scaling + Disease Blade/Poison Weapons Fix (two checkboxes) |
| `SLUS-20973_4028A55F.WeaponRebalance.pnach` | 2H Damage Bonus + 1H Attack Speed Bonus (two checkboxes) |

`SpellDamageScaling.pnach` sections:
- **INT Scaling** — spell damage scales with INT (standalone)
- **Shield Bash Defense Scaling** — Shield Bash scales with armor (requires INT Scaling also enabled)

`WeaponEnchantFixAndScaling.pnach` sections:
- **Weapon Enchant INT Scaling** — Fire/Cold/Lightning Weapons elemental scales with INT (standalone)
- **Disease Blade/Poison Weapons Fix** — fixes on-hit elemental damage and ally buff flicker for Disease Blade and Poison Weapon (standalone)

`WeaponRebalance.pnach` sections:
- **2H Damage Bonus** — +20% physical and elemental damage for 2H weapons (standalone)
- **1H Attack Speed Bonus** — +25% attack speed for unshielded 1H weapons (standalone)

All mods are fully independent and can be mixed freely.

---

## Credits

Patch written with Claude code by mysaltyspace (https://github.com/mysaltyspace).

Inspired by [skeewirt's SnowblindMods](https://github.com/skeewirt/SnowblindMods) for Champions of Norrath.