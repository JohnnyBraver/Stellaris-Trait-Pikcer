# Base Game Leader Trait Picker (Stellaris 4.5+)

Allows manual leader trait selection upon level-up in the **Stellaris Base Game** (without requiring the *Galactic Paragons* DLC).

## Features
- **Triggers the Native Stellaris Yellow [+] Trait Picker:**
  - When a leader levels up via XP (on_leader_level_up), the mod reverts the random trait auto-assigned by the C++ engine and executes the set_skill = (level - 1) + dd_skill = 1 trick used in vanilla paragon_events.txt to unlock the **native C++ yellow [+] trait selection drawer** (NON_PARAGON_TRAIT_OPTIONS_ON_LEVEL_UP = 4 weighted options + existing trait upgrades).
- **Automatic 4-Trait Weighted Popup Fallback:**
  - If the C++ engine ever auto-assigns a trait during dd_skill = 1, gltp.1 immediately detects and reverts it on the same tick and opens a **Leader Story Event** popup offering **4 role-weighted random eligible traits** + **existing Tier II / III trait upgrades** (with an optional button to browse the full categorized catalog).
- **203 Supported Traits (Base Game + Non-Paragon DLCs):**
  - Supports all **186 Base Game** level-up traits plus **17 non-Paragon DLC traits** (*Ancient Relics*, *Overlord*, *First Contact*, *Astral Planes*, *Grand Archive*, and *Shadows of the Shroud*) when those DLCs are active.
- **Starting Skill Bonus Compatible (*War Machines*, *Distinguished Admiralty*, etc.):**
  - Leaves initial recruitment traits untouched while syncing traits picked via the native yellow [+] button every 2 in-game days (gltp.6).
- **Save-Game Compatible & Negative Traits Untouched.**
