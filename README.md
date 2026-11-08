# Base Game Leader Trait Picker (Stellaris 4.5+)

Allows manual leader trait selection upon level-up in the **Stellaris Base Game** (without requiring the *Galactic Paragons* DLC).

## Branches
- **`main` (Default / Recommended):** Uses the **Native C++ Yellow `[+]` Trait Picker UI** with a level-up toast notification reminder. When a leader levels up, their card in the Leaders view lights up with the glowing yellow `[+]` button, opening the native 4-trait selection drawer.
- **`event-popup-picker`:** Alternative mode using a **Story Event Popup** upon level-up. Displays a 4-trait role-weighted dialog with an optional category browser to hand-pick any valid trait.

---

## Features on `main`
- **Native Stellaris Yellow `[+]` Trait Picker:**
  - When a leader levels up via XP (`on_leader_level_up`), the mod reverts the random trait auto-assigned by the C++ engine and triggers the native C++ trait selection drawer (`available_trait = 4 weighted options + existing trait upgrades`).
- **Level-Up Toast Notification:**
  - Sends a standard toast message notification (`MESSAGE_LEADER_LEVEL_UP`) alerting the player that the leader leveled up and has a trait selection waiting.
- **Zero Background Loops (Monthly Pulse & Assignment Sync):**
  - Newly picked traits are automatically registered via native game triggers: on the 1st of every month (`on_monthly_pulse_country`), on leader assignment/reassignment (`on_leader_assigned`), upon hiring (`on_leader_hired`), and on game load. No custom background timers or event loops are needed.
- **Starting Skill Bonus Compatible (*War Machines*, *Distinguished Admiralty*, etc.):**
  - Leaders generated in the recruitment pool with higher starting levels keep their rolled traits without re-prompting.
- **Save-Game Compatible & Negative Traits Untouched.**
- **Built-in Dev Tool (`event bgltp.99`):**
  - Allows granting exact XP to reach (Next Level - 1) or nudging with +5 XP for fast in-game testing.
