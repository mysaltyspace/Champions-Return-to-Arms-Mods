# Champions: Return to Arms — PCSX2 Mods

PCSX2 `.pnach` patch mods for **Champions: Return to Arms** (PS2, NTSC-U).

**Game serial:** `SLUS-20973` | **CRC:** `4028A55F`

---

## Mods

### INT Scaling
`custom patches/SLUS-20973_4028A55F.INTScaling.pnach`

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
- Disease/Poison Weapons elemental damage (base game bug — deals no elemental damage)

> **Note:** This patch is a work in progress. If you find a damage type that is incorrectly scaled or not scaled, please open an issue.

---

### Weapon Enchant INT Scaling
`custom patches/SLUS-20973_4028A55F.WeaponEnchantINTScaling.pnach`

Makes the elemental damage bonus from **Fire Weapons**, **Cold Weapons**, and **Lightning Weapons** spells scale with **Intelligence (INT)**. Uses the same tiered diminishing-returns formula as the INT Scaling mod. Works independently — enable with or without INT Scaling.

#### What scales
- Fire Weapons elemental hit bonus
- Cold Weapons elemental hit bonus
- Lightning Weapons elemental hit bonus

#### What does NOT scale
- Socketed gem elemental damage (intentionally excluded — different damage source)
- Disease/Poison Weapons — see Disease Blade On-Hit Fix section below
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

### Disease Blade On-Hit Fix
`custom patches/SLUS-20973_4028A55F.WeaponEnchantINTScaling.pnach` — second checkbox

Fixes **Disease Blade** (Shadow Knight) and **Poison Weapon** (Shaman) dealing no on-hit elemental damage. The base game has a bug in `z_un_001e8230` (the universal weapon-enchant elemental handler): a flag in the player entity struct has bit 17 set for disease/poison hits, causing the function to exit early before computing any damage. This patch NOPs that early-exit branch (`001E82A0`), allowing disease and poison hits to proceed through the same elemental damage path used by fire, cold, and lightning.

---

### Shield Bash Defense Scaling
`custom patches/SLUS-20973_4028A55F.INTScaling.pnach` — second checkbox in PCSX2

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

## Installation

1. Copy the desired `.pnach` file(s) to your PCSX2 **`patches/`** folder
   - Default location: `Documents/PCSX2/patches/`
   - Do **not** put them in the `cheats/` folder
2. In PCSX2, enable patches: **System → Enable Patches**
3. Boot the game — patches are active immediately on load

### Patch files

| File | Contents |
|------|----------|
| `SLUS-20973_4028A55F.INTScaling.pnach` | INT Scaling + Shield Bash Defense Scaling (two checkboxes) |
| `SLUS-20973_4028A55F.WeaponEnchantINTScaling.pnach` | Weapon Enchant INT Scaling (one checkbox) |

`INTScaling.pnach` has two independently toggleable sections:
- **INT Scaling** — spell damage scales with INT (standalone)
- **Shield Bash Defense Scaling** — Shield Bash scales with armor (requires INT Scaling also enabled)

All three mods are fully independent and can be mixed freely.

---

## Credits

Patch written with Claude code by mysaltyspace (https://github.com/mysaltyspace).

Inspired by [skeewirt's SnowblindMods](https://github.com/skeewirt/SnowblindMods) for Champions of Norrath.