# Base Game Leader Trait Picker (Stellaris 4.5+)

Allows manual leader trait selection upon level-up in the **Stellaris Base Game** (without requiring the *Galactic Paragons* DLC).

## How It Works & Engine Architecture

In Stellaris v4.5.3 (Cygnus), leader trait selection behavior is governed by the C++ engine binary (`stellaris.exe`):
1. **Day 1 Starting Bonus Leaders (Native Yellow `[+]` Drawer):**
   - Leaders initialized during galaxy generation with bonus starting levels (e.g., *Warbots* +2 Commander level, *Vaults of Knowledge*, *Distinguished Admiralty*) have their unspent trait selections (`m_available_trait_selections`) populated directly at creation (`create_leader`), bypassing `CLeader::LevelUp`.
   - On Day 1, these starting leaders natively display the glowing yellow `[+]` button and open the native 4-option trait drawer. The mod protects these initial leaders (`has_global_flag = game_started`, `has_base_skill > 1`, `has_experience > 0`) so players can use the native drawer on Day 1.
2. **Mid-Game Level-Ups (4-Option Weighted Event Popup Dialog):**
   - For any mid-game level-up from XP or events without the *Galactic Paragons* DLC, `CLeader::ShouldAutopickTraits()` in `stellaris.exe` is hardcoded to return `true` (`if (!HasDLC(GALACTIC_PARAGONS)) return true;`). As a result, the C++ engine automatically assigns a random trait and clears unspent picks *before* script on-actions run.
   - The mod intercepts the level-up via `on_leader_level_up`, immediately reverts the auto-assigned random trait, dynamically weights 4 eligible traits matching the game's define (`NON_PARAGON_TRAIT_OPTIONS_ON_LEVEL_UP = 4`), and presents an interactive **Leader Story Event Popup** (`bgltp.10`).
   - The dialog displays the leader's portrait, title, and current skill level, and presents:
     - **Existing Trait Upgrades (Tier II & III)** at the top.
     - **4 Role-Weighted Traits** matching leader class, council position, ethics, and unlocked technologies.
     - An optional **Browse All Categories** browser to inspect or select any valid trait.
   - Multiple queued level-ups (e.g. from large quest XP rewards) are queued cleanly via `bgltp_pending_picks`.

---

## Features
- **4-Option Role-Weighted Level-Up Trait Selection:**
  - Full player choice for all leader level-ups across Commanders, Scientists, Officials, Rulers, and Gestalt Central Nodes.
- **Existing Trait Upgrades Supported:**
  - Upgrading existing traits (Tier I -> Tier II -> Tier III) is always offered at the top of the selection dialog when eligible.
- **Full Gestalt Consciousness Support:**
  - Primary Link, Cognitive Node, Regulatory Node, Growth Node, and Legion Node are fully eligible for trait picks. Culled or reformatted nodes start fresh and gain picks normally.
- **Zero Background Loops (Event-Driven Trait Synchronization):**
  - Newly picked traits are automatically registered into the known traits registry via native game hooks:
    - Monthly country pulse (`on_monthly_pulse_country`)
    - Leader assignment & recruitment (`on_leader_assigned`, `on_leader_hired`)
    - Save game loading (`on_game_start_country`)
    - Anomaly completion (`anomaly_success`)
    - Planet surveys (`on_survey_planet`)
    - Archaeological site stage completion (`on_arch_stage_finished`, `on_arch_site_finished`)
    - Astral rift completion (`on_astral_rift_exploration_complete`)
    - Fleet combat victory/exit (`on_fleet_exit_battle`)
- **203 Supported Traits (Base Game + Non-Paragon DLCs):**
  - Supports all 186 Base Game level-up traits plus 17 traits from *Ancient Relics*, *Overlord*, *First Contact*, *Astral Planes*, *Grand Archive*, and *Shadows of the Shroud*.
- **Save-Game Compatible & Negative Traits Untouched:**
  - Negative traits are rolled by the vanilla yearly pulse (`paragon_2.1020`) based on leader age/lifespan and are never touched or altered.
- **Built-in Dev Tool (`event bgltp.99`):**
  - Allows console testing with instant exact level-up for the Ruler, random Commander, Scientist, Official, or ALL owned leaders.
