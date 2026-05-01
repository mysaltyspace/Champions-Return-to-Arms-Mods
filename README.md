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
- All direct damage spells (Bolt of Shock, etc.)
- AoE spells (Cone of Fire, Cone of Frost, etc.)
- DoT ticks (Fire Storm, etc.)
- Beam spells (Wizard Beam, etc.)
- Berserker force/shout abilities (Roar, etc.)
- Holy Strike (Cleric)

#### What does NOT scale
- Melee attacks (blunt, slashing)
- Ranged attacks (bows, thrown weapons)
- Cyclone and other melee AoE skills
- Shield Bash — has its own optional scaling mod (see below)
- Weapon enchantment elemental damage (Fire Weapons, Cold Weapons, Lightning Weapons spells) — these are merged with the weapon's physical hit before the scaling point and cannot currently be distinguished

> **Note:** This patch is a work in progress. If you find a damage type that is incorrectly scaled or not scaled, please open an issue.

---

### Shield Bash Defense Scaling
`custom patches/SLUS-20973_4028A55F.INTScaling.pnach` — second checkbox in PCSX2

Makes **Shield Bash** damage scale with the Paladin's **total armor**. Rewards armor-focused builds without making the ability overpowered at low gear levels.

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

1. Copy the `.pnach` file to your PCSX2 **`patches/`** folder
   - Default location: `Documents/PCSX2/patches/`
   - Do **not** put it in the `cheats/` folder
2. In PCSX2, enable patches: **System → Enable Patches**
3. Boot the game — the patch is active immediately on load

The `.pnach` file contains two independently toggleable sections shown as checkboxes in PCSX2:
- **INT Scaling** — spell damage scales with INT (standalone)
- **Shield Bash Defense Scaling** — Shield Bash scales with armor (requires INT Scaling also enabled)

---

## Credits

Patch written with Claude code by mysaltyspace (https://github.com/mysaltyspace).

Inspired by [skeewirt's SnowblindMods](https://github.com/skeewirt/SnowblindMods) for Champions of Norrath.