# CLAUDE.md — Trial of Notch

## Project Overview

**Trial of Notch** (创世神审判) is a boss-driven Minecraft modpack/mod for **Minecraft 1.20.1** on **Forge 47.2.x**. It combines RPG progression, custom bosses, skill effects, dimension exploration, and artifact weapons into a single cohesive experience. The primary author is **benyexi**.

The mod is distributed as `TrialOfNotch-v1.0.0.zip` — an integrated modpack archive meant for launchers like HMCL or CurseForge.

## Key Dependencies

- **KubeJS** 1901.6.3+ — All game logic (boss behavior, skills, drop tables, systems) is written as KubeJS scripts
- **Patchouli** 1.20.1-81+ — In-game guidebooks (boss manual, weapon manual, dimension manual)
- **Forge** 47.2.x — Mod loader

## Repository Structure

```
TrialOfNotch-v1.0.0.zip          # Distributable modpack archive
README.md                         # Project README (mostly empty at repo root)
CLAUDE.md                         # This file

# Inside the zip / modpack:
TrialOfNotch-v1.0.0/
├── kubejs/
│   ├── server_scripts/           # Server-side KubeJS scripts (core game logic)
│   │   ├── boss/
│   │   │   ├── notch.js          # Notch (final boss) — spawn effects, rage phase at 33% HP
│   │   │   ├── executed_dragon.js # Execution Dragon — two rage phases (50%, 33%)
│   │   │   └── wither_titan.js   # Wither Titan — kill tracking, Wither Sla spawn trigger
│   │   ├── skills/
│   │   │   ├── doom_burst.js     # "Destroy All" skill — Warden sonic boom + ground shake, 90s CD
│   │   │   └── inferno_slash.js  # "Inferno Taichi Slash" — red/blue flame rings, 60s CD
│   │   ├── system/
│   │   │   └── player_tags.js    # Dynamic rainbow title system — "[创世者]" after defeating Notch
│   │   ├── warehouse/
│   │   │   └── system.js         # Overflow inventory storage (giveOrStore function)
│   │   └── drop_system.js        # Boss loot tables — items & blocks on boss kill
│   └── cilent_scripts/           # Client-side KubeJS scripts (note: typo "cilent")
│       └── warehouse_ui.js       # Warehouse UI button on inventory screen
├── assets/trial_of_notch/
│   ├── lang/
│   │   └── zh_cn.json            # Chinese localization keys for items, bosses, UI
│   └── patchouli_books/
│       ├── boos_manual/          # Boss guidebook (note: typo "boos")
│       │   ├── book.json
│       │   ├── categories/bosses.json
│       │   └── entries/          # notch.json, executed_dragon.json, wither_sla.json, wither_titan.json
│       ├── dimension_manual/     # Dimension guidebook
│       │   ├── book.json
│       │   ├── categories/dimension.json
│       │   └── entries/inferno_realm.json
│       └── weapon_manual/        # Weapon guidebook
│           ├── book.json
│           ├── categories/weapon.json
│           └── entries/          # doom_blade.json, god_sword.json, hell_blade.json
├── data/trial_of_notch/
│   └── advancements/
│       ├── arrival_of_creator.json   # Triggers on first tick (welcome advancement)
│       └── defeat_notch.json         # Triggers on killing yourmod:notch
├── mods/                         # (empty — users add mod JARs here)
├── screenshots/                  # (empty — for promotional screenshots)
├── logo..png                     # Modpack logo (note: double dot in filename)
└── README.md                     # Full project README with install instructions
```

## Game Design & Content

### Bosses (5 total)

| Boss | Entity ID | HP | Rage Phases | Drops |
|---|---|---|---|---|
| Wither Titan (凋零泰坦) | `yourmod:wither_titan` | — | None | Immortal Armor (2.87%), Doom Blade (0.43%) |
| Wither Sla (凋零斯拉) | `yourmod:wither_sla` | — | — | All blocks + Infinite Armor set |
| Execution Dragon (执行之龙) | `yourmod:executed_dragon` | — | Normal (50% HP), Extreme (33% HP) | All blocks + Hell Blade |
| Notch (创世神) | `yourmod:notch` | 100,000,000 | Rage at 33% HP (0.5s attack interval) | All artifacts + "[创世者]" rainbow title |

### Weapons

| Weapon | Item ID | Skill | Cooldown |
|---|---|---|---|
| Doom Blade (毁灭之剑) | `yourmod:doom_blade` | Destroy All — Warden sonic boom + ground shake, 100-block AoE | 90s |
| Hell Blade (炼狱之剑) | `yourmod:hell_blade` | Inferno Taichi Slash — rotating red/blue flame circles | 60s |
| God Sword (终极斩神剑) | — | Soulbound, max enchants (32767), cannot be dropped | — |

### Armor Sets

- **Immortal Set** (`yourmod:immortal_*`) — helmet, chestplate, leggings, boots
- **Infinite Set** (`yourmod:infinite_*`) — helmet, chestplate, leggings, boots

### Systems

- **Warehouse System**: `giveOrStore()` stores overflow items in `persistentData.inventoryStorage`; client-side UI button in inventory
- **Title System**: Rainbow-cycling "[创世者]" prefix via Minecraft teams, awarded after defeating Notch
- **Advancement System**: Welcome advancement on join, defeat-Notch advancement on kill

### Dimension

- **Inferno Realm** (炼狱维度): Entered via Ancient Debris portal frame, ignited with Flint & Steel while holding God Sword

## Development Conventions

### Language & Localization

- **Primary language**: Chinese (Simplified) — all in-game text, comments, and documentation are in Chinese
- Localization keys are defined in `assets/trial_of_notch/lang/zh_cn.json`
- Item keys follow: `item.yourmod.<item_name>`
- Boss keys follow: `boss.trial_of_notch.<boss_name>`

### KubeJS Scripting Patterns

- **Server scripts** go in `kubejs/server_scripts/` — organized by domain (boss/, skills/, system/, warehouse/)
- **Client scripts** go in `kubejs/cilent_scripts/` (note: existing typo in directory name)
- Entity type checks use string equality: `entity.type == 'yourmod:entity_name'`
- Boss state is stored via `entity.persistentData` (putBoolean, putInt, getBoolean, getInt)
- Player state/cooldowns are stored via `player.persistentData`
- Cooldowns are tracked in ticks (20 ticks = 1 second) and decremented in `ServerEvents.tick`
- Visual effects use `runCommand` with `/particle` and `/playsound`
- Boss announcements use `runCommand` with `/title @a title {...}`
- Minecraft formatting codes (`§c`, `§6`, etc.) are used for colored text

### Event Hooks Used

| Event | Purpose |
|---|---|
| `EntityEvents.spawned` | Boss spawn effects and state initialization |
| `EntityEvents.tick` | Boss rage phase transitions |
| `EntityEvents.death` | Drop tables, kill tracking, title awarding |
| `PlayerEvents.rightClick` | Weapon skill activation |
| `PlayerEvents.loggedIn` | Warehouse initialization |
| `ServerEvents.tick` | Cooldown timers, rainbow title cycling |
| `ClientEvents.lang` | Localization registration |
| `ClientEvents.highPriorityAssets` | UI element registration |
| `ClientEvents.register` | Custom GUI screens |

### Item ID Namespace

All custom items/entities use the `yourmod:` namespace prefix. This is a placeholder that should be replaced with the final mod ID when the mod is packaged as a standalone Forge mod (vs. KubeJS scripts).

### Patchouli Books

- Three books: `boos_manual` (boss guide), `weapon_manual`, `dimension_manual`
- Each book has: `book.json` (metadata), `categories/` (category definitions), `entries/` (content pages)
- Entries use `"type": "text"` pages with inline formatting

### Advancements

- Located in `data/trial_of_notch/advancements/`
- Use standard Minecraft advancement JSON format
- `frame: "challenge"` for all advancements

## Known Issues / Technical Debt

1. **Typo in directory name**: `cilent_scripts` should be `client_scripts`
2. **Typo in book directory**: `boos_manual` should be `boss_manual`
3. **Typo in logo filename**: `logo..png` has double dots
4. **Placeholder namespace**: `yourmod:` needs to be replaced with the actual mod ID (`trial_of_notch:`)
5. **Missing Wither Sla boss script**: `wither_sla` entity is referenced in drops/books but has no behavior script in `boss/`
6. **No damage logic in skills**: `inferno_slash.js` produces particles but does not deal damage to nearby entities (unlike `doom_burst.js`)
7. **God Sword has no script**: The God Sword (终极斩神剑) is described in the guidebook but has no acquisition or behavior script
8. **Empty mods/ directory**: Mod JAR dependencies are not included in the repo

## How to Test

1. Install Minecraft 1.20.1 with Forge 47.2.x
2. Install KubeJS 1901.6.3+ and Patchouli 1.20.1-81+
3. Extract `TrialOfNotch-v1.0.0.zip` into the modpack/instance directory
4. Launch the game — the welcome advancement should trigger immediately
5. Boss entities require the custom entity mod to be registered (not yet included)

## AI Assistant Guidelines

- When modifying KubeJS scripts, follow the existing patterns: use `persistentData` for state, `runCommand` for effects, event-driven architecture
- Preserve Chinese comments and in-game text — this is a Chinese-language mod
- Keep the file organization consistent: boss logic in `boss/`, skills in `skills/`, systems in `system/` or `warehouse/`
- Use the same cooldown pattern: store in `persistentData`, decrement in `ServerEvents.tick`
- When adding new bosses, include: spawn event (effects + state init), tick event (rage phases), death event (drops)
- When adding new weapons, include: right-click skill activation, cooldown management, Patchouli entry, lang key
- Test rage phase thresholds carefully — they rely on HP ratio comparisons
