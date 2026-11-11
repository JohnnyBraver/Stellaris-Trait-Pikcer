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
- **Event-Driven Trait Synchronization plus a light daily safety sync (`bgltp.6`):**
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

---

## Code Audit Notes

### Native trait-selection facts (from game script docs and defines)
- Leaders carry **unspent trait selections**. `add_trait = { trait = X consume_selection = yes }` spends one; `create_leader` with `skill = N` and a `traits = { <level> = X }` list spends the selections for those levels. `clone_leader` copies picks from the last created leader.
- There is **no script effect or trigger that grants or counts unspent selections** on an existing leader. `add_skill`/`set_skill` go through the engine level-up, and `add_skill_without_trait_selection` deliberately grants none.
- Day-1 leaders created with bonus levels are the only case that reaches the native `[+]` drawer without the Galactic Paragons DLC. Mid-game level-ups are auto-picked by the engine before `on_leader_level_up` runs. This is observed behavior; the engine function was not located in the binary.
- `autopick_leader_traits` is a valid country code flag and the base UI has an auto-select checkbox, but it was **not shown to have any effect** without Paragons. Do not rely on it.
- The engine level-up toast and its trait text cannot be edited or suppressed from script. `on_leader_level_up` is the earliest scriptable point.
- Defines: `NON_PARAGON_TRAIT_OPTIONS_ON_LEVEL_UP = 4`, `LEADER_TRAIT_SELECTION_LEVELS = 1..10`.

### Known limitation: revert depends on the known-trait registry
`bgltp_revert_auto_trait_effect` removes any tracked trait that is not flagged `bgltp_known_*`. Flags are refreshed only by the sync hooks. A trait added by a vanilla event, civic (`leader.110`), relic or similar after the last sync is wrongly removed at the next level-up.

### XP sources and sync coverage
Synced before XP (hooked): anomaly success, planet survey, archaeology stage/site finished, astral rift completion, fleet exit battle, plus monthly pulse, hire, assign, unassign, spawn and pool add.

**Not hooked** (vanilla `add_experience` in scripts, by area): anomaly event chains that grant XP in later events (`anomaly_events_1`-`4`), Distant Stars, Nomads and arkship events, Biogenesis, Grand Archive origin/events, First Contact effects, Overlord enclaves, Shroud, Megastructures, Cosmic Storms, Infernals, Horizon Signal, Machine Age situations, council agendas and traditions, relics, `on_action_events_3`, and extra arc-site/random-site events.

**Not scriptable:** engine passive XP for councilors, governors and the ruler. The monthly pulse sync races it, and the order is not guaranteed.

### Paths that should not open a picker
`add_skill_without_trait_selection` (starting skills, some origin events), `set_skill` (relics, events, creation) and Paragon-only paths. `bgltp.1` filters only on `has_base_skill > 1` and `has_experience > 0`, so it may offer picks for grants vanilla intends to be pickless if `on_leader_level_up` fires for them.

### Leader types covered
Ruler (including gestalt Primary Link), commanders (fleet-assigned), scientists, officials (councilors, governors, planet-assigned, gestalt nodes). Pool leaders are excluded.

### Daily sync (bgltp.6)
A self-rescheduling 1-day event syncs every owned leader that can still level up (`bgltp_can_level_up_soon_trigger`, window `@bgltp_xp_window = 4500`), covering XP from sources without a hook. The window is based on the largest scripted XP grant (3000, plus headroom for XP modifiers). Because every level gap is <= 2800, all leaders below level 10 currently qualify; lower the window to narrow it. The loop starts once per country (`bgltp_daily_sync_active`) from `bgltp.2`/`bgltp.3`. Cost is roughly 200 trait checks per leader per day for human countries only. The game has no daily country on_action, so this uses a rescheduled event.

### Count-gated scan
The daily pass calls `bgltp_sync_if_changed_effect`, which stores a per-leader signature (`num_traits`*1000 + `num_trait_points`) and runs the full 207-trait scan only when it changes. Script has no on_action or trigger for non-level-up trait gains (only `on_random_trait_added` for species), so the signature is the change detector. Same-day swaps of equal-cost traits are not detected; hire/assign/battle hooks still do the full scan.
