# Base Game Leader Trait Picker (Stellaris 4.5+)

Allows manual leader trait selection upon level-up in the **Stellaris Base Game** (without requiring the *Galactic Paragons* DLC).

## Features
- **Zero Vanilla Overwrites:** Hooks cleanly into on_leader_level_up to revert the random C++ auto-picked trait (or Tier II/III auto-upgrade) and open an interactive **Leader Story Event** window with the leader's portrait.
- **203 Supported Traits:**
  - Includes all **186 Base Game** level-up traits across Commanders, Scientists, and Officials (including base-game-exclusive traits and ree_or_veteran traits).
  - Dynamically supports **17 non-Paragon DLC traits** (*Ancient Relics*, *Overlord*, *First Contact*, *Astral Planes*, *Grand Archive*, and *Shadows of the Shroud*) when those DLCs are enabled.
- **Categorized Trait Menus:**
  - **Upgrade Existing Trait (Tier II / III)**
  - **Field Traits** (2 role-specific sub-categories per class)
  - **Council Traits** (2 role-specific sub-categories per class)
  - **Auto-Pick (Random Valid Trait)** for quick assignment
- **Save-Game Compatible:** Automatically syncs existing leaders' traits when loading an ongoing save game (on_single_player_save_game_load).
- **Negative Traits Untouched:** Preserves the vanilla yearly negative trait pulse system (paragon_2.1020).
