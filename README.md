# RPG-MAKER-MZ
## ARPG Map Combat System for RPG Maker MZ

A real-time **map-based ARPG combat framework** for **RPG Maker MZ**.

This project replaces traditional turn-based encounters with direct combat on the map.  
Players attack in real time, cast weapon-based skills, fight event-driven enemies, build combo count, and collect loot directly from the map.

Email: `algarloner@gmail.com`  
Discord: `CFpanda`

---

## Overview

This project is a modular ARPG combat framework built for **RPG Maker MZ**.

Its core design is based on:

- **Real-time map combat**
- **Attack Object driven hit detection**
- **Event-based enemies linked to database battlers**
- **Weapon-based skill switching**
- **Independent skill cooldowns**
- **Combo extension**
- **Map loot drop system**

Instead of applying damage immediately when a skill is used, skills first create an **Attack Object**.  
Damage is only calculated when that object actually reaches and collides with a target.

This makes the system more suitable for action RPG gameplay such as:

- melee slash attacks
- arrow / projectile skills
- enemy projectile attacks
- combo-based combat
- future multi-hit or piercing extensions

---

## Core Modules

### CF_Attack.js
Handles player normal attack input and basic attack flow.

Features:
- Normal attack input on **A**
- Basic attack cooldown
- Front-target detection
- Weapon-dependent normal attack skill selection
- Shared map combat entry point

### CF_AttackObject.js
The core combat execution module.

Features:
- Creates and updates **Attack Objects**
- Supports different attack object types such as:
  - `slash`
  - `arrow`
  - `punch`
- Handles movement, range, collision, and lifetime
- Calculates:
  - hit / miss
  - evasion
  - critical hits
  - variance-based damage
- Works for both:
  - player attacking enemies
  - enemies attacking player

### CF_Enemy.js
Converts map events into database-driven enemies.

Features:
- Event enemy binding using note tags such as:
  - `<enemy:id>`
- Maps event enemies to RPG Maker database enemies
- Reads enemy action settings from database
- Selects usable skills using enemy action rating logic
- Supports:
  - direct enemy skills
  - Attack Object based enemy skills
- Displays:
  - enemy HP / MP bars
  - damage popups
  - MISS popups
  - skill name popups

### CF_PlayerSkillCast.js
Handles player skill casting on the map.

Features:
- Uses **Q / W / E / R** as skill hotkeys
- Switches skill sets automatically by weapon type
- Checks cooldown before casting
- Consumes MP / TP
- Displays floating skill name text
- Sends skills into the Attack Object combat pipeline

### CF_ComboSystem.js
Combo extension for real-time combat.

Features:
- Tracks combo hits when attack objects actually land
- Breaks combo after inactivity
- Shows combo text near the player
- Grants temporary attack bonus based on combo count

### CF_EnemyLoot.js
Map loot drop extension.

Features:
- Enemies drop loot directly onto the map
- Drop icons scatter around defeated enemies
- Nearby items are picked up automatically
- Supports:
  - items
  - weapons
  - armors
  - gold

---

## Combat Flow

### Player Normal Attack
1. Press **A**
2. The system checks the equipped weapon
3. A normal attack skill is selected
4. The skill creates an Attack Object
5. The Attack Object moves / plays on the map
6. Damage is calculated only when collision occurs

### Player Skill Cast
1. Press **Q / W / E / R**
2. The system checks current weapon type
3. A weapon-specific skill ID is selected
4. Cooldown and resource checks are performed
5. MP / TP is consumed
6. An Attack Object is generated
7. Skill name popup appears
8. Cooldown begins

### Enemy Attack
1. A map event is marked as an enemy with `<enemy:id>`
2. The event is linked to database enemy data
3. Enemy actions are evaluated
4. A valid skill is selected
5. The skill either:
   - applies direct damage, or
   - creates an Attack Object
6. The result is displayed on the map through combat UI feedback

---

## Weapon-Based Skill Mapping

Current built-in mapping follows weapon type.

### Normal Attack
- No weapon → Skill **1**
- Weapon Type **1** → Skill **2**
- Weapon Type **2** → Skill **7**

### QWER Skill Sets

#### Sword (Weapon Type 1)
- Q → Skill **3**
- W → Skill **4**
- E → Skill **5**
- R → Skill **6**

#### Bow (Weapon Type 2)
- Q → Skill **8**
- W → Skill **9**
- E → Skill **10**
- R → Skill **11**

You can expand or edit this mapping directly in the plugin code.

---

## Note Tags

### Enemy Event
```xml
<enemy:1>

Binds a map event to database enemy ID 1.

Attack Object Type
<attackObject:slash>
<attackObject:arrow>
<attackObject:punch>

Defines what type of Attack Object the skill creates.

Range
<range:3>

Defines the line range used by the attack or skill.

Cooldown
<cooldown:60>

Defines cooldown in frames.
60 frames ≈ 1 second.

Sound Effect
<se:Slash1>
<se:Slash1,90,100,0>

Optional skill sound playback definition.

UI Feedback

This framework already supports map-based combat feedback:

enemy HP bar
enemy MP bar
damage popup
MISS popup
skill name popup
combo popup

This allows the whole combat loop to stay on the map without switching to the default battle scene.

Loot System

Enemy loot is dropped directly onto the map instead of going through a traditional battle result screen.

Features:

loot icon generation on enemy death
scattered drop positions
automatic pickup when player approaches
support for item / weapon / armor / gold rewards

This keeps exploration, combat, and reward flow inside one continuous ARPG loop.

Recommended Plugin Structure

Recommended load order:

CF_Attack.js
CF_AttackObject.js
CF_Enemy.js
CF_PlayerSkillCast.js
CF_ComboSystem.js
CF_EnemyLoot.js

If you add custom extensions such as motion plugins, dash combo plugins, or special projectile plugins, place them below the core modules.

Project Scope

This project currently focuses on:

map-based real-time combat
event-driven enemies
weapon-dependent skill casting
Attack Object based hit resolution
combo combat feedback
map loot integration

This is not a replacement for RPG Maker MZ's default turn-based battle scene.
It is a separate ARPG combat framework intended for action gameplay directly on the map.

License

This project is released for learning, research, and non-commercial use only.

You may:

study the source code
modify it for personal learning
use it in free or non-commercial projects

You may not:

use this code in commercial games
sell or redistribute this project for profit
use derivative works in commercial products without permission

Copyright (c) 2026 CFpanda. All rights reserved.

For commercial licensing, contact the author separately:

Email: algarloner@gmail.com
Discord: CFpanda

This framework is designed to be expandable.

Possible extensions include:

motion systems
special projectile logic
dash attack skills
piercing mechanics
boss AI patterns
HUD / toolbar systems
state / buff combat integration

The goal of this project is to provide a reusable foundation for real-time action combat in RPG Maker MZ.

## License

This project is source-available and provided for personal learning, research, and non-commercial use only.

Commercial use is strictly prohibited without a separate written license from the author.

For commercial licensing:
- Email: algarloner@gmail.com
- Discord: CFpanda
