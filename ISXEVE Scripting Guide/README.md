# ISXEVE Scripting Guide

**Language:** LavishScript
**Extension:** ISXEVE for InnerSpace
**Target Game:** EVE Online

---

## Overview

This comprehensive guide provides complete documentation for creating, debugging, and maintaining scripts for the ISXEVE InnerSpace extension. Whether you're new to ISXEVE scripting or an experienced developer, this guide will help you understand the API, learn best practices, and implement powerful automation for EVE Online.

**ISXEVE** is an InnerSpace extension that exposes EVE Online's game data and functionality through a structured LavishScript API. It allows you to:

- Access character, ship, and module information
- Automate combat rotations and ship actions
- Interact with game UI elements and windows
- React to game events in real-time
- Manage inventory, cargo, and market operations
- Create complex multi-character automation
- Coordinate fleet operations

---

## Documentation Structure

### Getting Started

1. **[00_MASTER_GUIDE.md](00_MASTER_GUIDE.md)** - Master Reference & Navigation Hub
   Complete overview of the ISXEVE API with organized links to all datatypes, commands, and events. **Start here for quick reference and navigation.**

2. **[01_LavishScript_Fundamentals.md](01_LavishScript_Fundamentals.md)** - LavishScript Basics (For beginners!)
   Tutorial-style introduction to LavishScript programming. Concepts and intuition: variables, functions, objects, loops, conditionals, atoms, events, triggers, aliases, modules, input emulation, web requests, audio, and best practices. **If you're new to LavishScript, read this before the Quick Start Guide.** For the exhaustive reference inventory, see file 01b below.

3. **[01b_LavishScript_Reference.md](01b_LavishScript_Reference.md)** - LavishScript and Inner Space Reference
   Pure-reference companion to file 01. Exhaustive inventory of every LavishScript-core and Inner-Space-core command, datatype, and Top-Level Object, with one canonical entry per feature linked to the Lavish Software wiki. Use this when you want a flat lookup table rather than tutorial prose.

4. **[02_Quick_Start_Guide.md](02_Quick_Start_Guide.md)** - Your First ISXEVE Script
   Get up and running in 5-10 minutes with your first ISXEVE script. Includes installation verification and a simple working example. **Assumes basic LavishScript knowledge.**

### Core Documentation

4. **[03_API_Reference.md](03_API_Reference.md)** - Complete API Documentation
   Exhaustive reference for all Top-Level Objects (TLOs), datatypes, members, methods, and commands. Covers entities, ships, modules, inventory, UI, and more.

5. **[04_Core_Concepts.md](04_Core_Concepts.md)** - Fundamental Concepts
   Essential concepts every ISXEVE scripter needs to know: datatypes, TLOs, game mechanics, extension architecture, and EVE-specific patterns.

### Practical Guides

6. **[05_Patterns_And_Best_Practices.md](05_Patterns_And_Best_Practices.md)** - Development Patterns
   Proven coding patterns, main loop architecture, error handling strategies, and best practices learned from analyzing production scripts like [EVEBot](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable).

7. **[06_Working_Examples.md](06_Working_Examples.md)** - Practical Examples
   Real-world, runnable code examples covering common tasks: target locking, module activation, inventory management, autopilot, and event-driven scripts.

8. **[07_Advanced_Patterns_And_Examples.md](07_Advanced_Patterns_And_Examples.md)** - Advanced Patterns
   Advanced patterns from production scripts: multi-boxing, relay/IPC systems, configuration management, fleet coordination, and complex automation.

### UI and System Guides (Game-Agnostic)

9. **[08_LavishGUI1_UI_Guide.md](08_LavishGUI1_UI_Guide.md)** - LavishGUI 1 UI Creation
   Complete guide to creating custom user interfaces with LavishGUI 1 XML for InnerSpace scripts.

10. **[09_Advanced_LGUI1_Patterns.md](09_Advanced_LGUI1_Patterns.md)** - Advanced LavishGUI 1 Patterns
   Real-world patterns from production scripts: UI element navigation, dynamic updates, script communication, and advanced list management.

11. **[10_LavishGUI2_UI_Guide.md](10_LavishGUI2_UI_Guide.md)** - LavishGUI 2 UI Creation
    Modern JSON-based UI system guide. Covers LGUI2 fundamentals, JSON packages, templates, data binding, and complete working examples.

12. **[11_LavishGUI1_to_LavishGUI2_Migration.md](11_LavishGUI1_to_LavishGUI2_Migration.md)** - LGUI1 to LGUI2 Migration
    Convert existing LavishGUI 1 XML scripts to LavishGUI 2 JSON. Includes migration patterns, common issues, and side-by-side comparisons.

13. **[12_LGUI2_Scaling_System.md](12_LGUI2_Scaling_System.md)** - UI Scaling System
    Add dynamic UI scaling to LGUI2-based scripts. Reusable library for consistent UI scaling across different screen resolutions.

14. **[13_JSON_Guide.md](13_JSON_Guide.md)** - JSON in LavishScript
    Complete guide to JSON in LavishScript: creating, accessing, modifying JSON data, file I/O, serialization, and complete examples.

15. **[14_LavishMachine_Guide.md](14_LavishMachine_Guide.md)** - LavishMachine (LMAC)
    LavishMachine Virtual Machine guide: async operations, time-based animations, audio control, web requests, task sequencing, and custom task types.

### Specialized Automation Guides

16. **[15_Combat_Automation.md](15_Combat_Automation.md)** - Combat Bot Patterns
    Complete combat automation patterns: target prioritization, weapon systems, tank management, drone control, and analysis of [Tehbot](https://github.com/isxGames/Tehbot) combat patterns.

17. **[16_Mining_And_Hauling.md](16_Mining_And_Hauling.md)** - Mining and Hauling Automation
    Mining bot patterns, hauling logistics, ore management, station trading, and resource optimization.

18. **[17_Fleet_Operations.md](17_Fleet_Operations.md)** - Fleet Coordination
    Fleet coordination patterns, [Yamfa](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/Yamfa) fleet assist analysis, relay systems for multi-character communication, and synchronized operations.

19. **[18_Bot_Architecture_Analysis.md](18_Bot_Architecture_Analysis.md)** - Production Bot Analysis
    In-depth analysis of [EVEBot](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable), [Yamfa](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/Yamfa), and [Tehbot](https://github.com/isxGames/Tehbot) architectures. Learn from battle-tested production scripts.

### Advanced Topics

20. **[19_DotNet_Development.md](19_DotNet_Development.md)** - Scripts vs .NET Programs
    When to use LavishScript vs .NET, [Metatron2](https://github.com/spacekoala420/Metatron2) .NET architecture overview, and hybrid development patterns.

21. **[20_Debugging_And_Troubleshooting.md](20_Debugging_And_Troubleshooting.md)** - Debugging Guide
    Debugging techniques, common problems and solutions, troubleshooting patterns, and diagnostic tools.

22. **[21_Advanced_Scripting_Patterns.md](21_Advanced_Scripting_Patterns.md)** - Advanced Patterns
    Production-grade patterns for script organization, main loops, state machines, error handling, relay communication, and configuration management.

23. **[22_Utility_Script_Patterns.md](22_Utility_Script_Patterns.md)** - Utility Patterns
    Essential utility patterns: custom timers, ISK tracking, position management, input validation, progressive tracking, and complete working examples.

---

<!-- CLAUDE_SKIP_START -->
## Quick Navigation

### By Task

| Task | Documentation |
|------|---------------|
| **Learn LavishScript basics (tutorial)** | [01_LavishScript_Fundamentals.md](01_LavishScript_Fundamentals.md) |
| **Look up a LavishScript / Inner Space command, datatype, or TLO (reference)** | [01b_LavishScript_Reference.md](01b_LavishScript_Reference.md) |
| **Write your first ISXEVE script** | [02_Quick_Start_Guide.md](02_Quick_Start_Guide.md) |
| **Look up an API member** | [03_API_Reference.md](03_API_Reference.md) |
| **Understand core concepts** | [04_Core_Concepts.md](04_Core_Concepts.md) |
| **Access character/ship stats** | [03_API_Reference.md#me-object](03_API_Reference.md#me-object) |
| **Control ship modules** | [03_API_Reference.md#module-datatypes](03_API_Reference.md#module-datatypes) |
| **Target and lock entities** | [03_API_Reference.md#entity-datatypes](03_API_Reference.md#entity-datatypes) |
| **Navigate and autopilot** | [03_API_Reference.md#navigation](03_API_Reference.md#navigation) |
| **Manage inventory/cargo** | [03_API_Reference.md#inventory-datatypes](03_API_Reference.md#inventory-datatypes) |
| **Interact with UI** | [03_API_Reference.md#ui-datatypes](03_API_Reference.md#ui-datatypes) |
| **Handle events** | [06_Working_Examples.md#event-driven-scripting](06_Working_Examples.md#event-driven-scripting) |
| **Create UI with LavishGUI 1** | [08_LavishGUI1_UI_Guide.md](08_LavishGUI1_UI_Guide.md) |
| **Create UI with LavishGUI 2** | [10_LavishGUI2_UI_Guide.md](10_LavishGUI2_UI_Guide.md) |
| **Work with JSON** | [13_JSON_Guide.md](13_JSON_Guide.md) |
| **Use LavishMachine (async tasks)** | [14_LavishMachine_Guide.md](14_LavishMachine_Guide.md) |
| **Build combat automation** | [15_Combat_Automation.md](15_Combat_Automation.md) |
| **Build mining automation** | [16_Mining_And_Hauling.md](16_Mining_And_Hauling.md) |
| **Coordinate fleets** | [17_Fleet_Operations.md](17_Fleet_Operations.md) |
| **Multi-box characters** | [07_Advanced_Patterns_And_Examples.md#multi-boxing](07_Advanced_Patterns_And_Examples.md#multi-boxing) |
| **Use relay/IPC** | [07_Advanced_Patterns_And_Examples.md#relay-system](07_Advanced_Patterns_And_Examples.md#relay-system) |
| **Advanced script patterns** | [21_Advanced_Scripting_Patterns.md](21_Advanced_Scripting_Patterns.md) |
| **Utility patterns (timers, tracking)** | [22_Utility_Script_Patterns.md](22_Utility_Script_Patterns.md) |
| **Debug scripts** | [20_Debugging_And_Troubleshooting.md](20_Debugging_And_Troubleshooting.md) |

### By API Category

| Category | Documentation |
|----------|---------------|
| **Character (Me)** | [03_API_Reference.md#me-object](03_API_Reference.md#me-object) |
| **Ship (MyShip)** | [03_API_Reference.md#myship-object](03_API_Reference.md#myship-object) |
| **Entities/NPCs** | [03_API_Reference.md#entity-datatypes](03_API_Reference.md#entity-datatypes) |
| **Modules/Weapons** | [03_API_Reference.md#module-datatypes](03_API_Reference.md#module-datatypes) |
| **Inventory/Cargo** | [03_API_Reference.md#inventory-datatypes](03_API_Reference.md#inventory-datatypes) |
| **UI Windows** | [03_API_Reference.md#ui-datatypes](03_API_Reference.md#ui-datatypes) |
| **Fleet** | [03_API_Reference.md#fleet-datatypes](03_API_Reference.md#fleet-datatypes) |
| **Navigation** | [03_API_Reference.md#navigation](03_API_Reference.md#navigation) |
| **Game (EVE)** | [03_API_Reference.md#eve-object](03_API_Reference.md#eve-object) |
<!-- CLAUDE_SKIP_END -->

---

<!-- CLAUDE_SKIP_START -->
## Quick Start Guides

For task-oriented navigation — "I want to build a combat bot", "I want to implement targeting", "I want to coordinate a fleet" — see [Quick Navigation > By Task](#by-task) above, which maps specific tasks directly to the relevant guide files. For a narrative walkthrough of each guide file's purpose and scope, see [Documentation Structure](#documentation-structure) near the top of this README.

If you're brand new to EVE botting, the shortest path to a running script is [01_LavishScript_Fundamentals.md](01_LavishScript_Fundamentals.md) → [02_Quick_Start_Guide.md](02_Quick_Start_Guide.md) → [03_API_Reference.md](03_API_Reference.md) → [06_Working_Examples.md](06_Working_Examples.md). If you already know what kind of bot you want to build, jump straight to the matching domain guide: [15_Combat_Automation.md](15_Combat_Automation.md), [16_Mining_And_Hauling.md](16_Mining_And_Hauling.md), or [17_Fleet_Operations.md](17_Fleet_Operations.md).
<!-- CLAUDE_SKIP_END -->

---

<!-- CLAUDE_SKIP_START -->
## Learning Paths

Three progression paths depending on your starting point:

- **Complete beginner** (never scripted before) — start with [01_LavishScript_Fundamentals.md](01_LavishScript_Fundamentals.md) for language basics, then [02_Quick_Start_Guide.md](02_Quick_Start_Guide.md) to verify your install, then [04_Core_Concepts.md](04_Core_Concepts.md) for EVE/ISXEVE architecture, then skim [03_API_Reference.md](03_API_Reference.md) to understand TLOs, and finally [06_Working_Examples.md](06_Working_Examples.md) to copy and modify runnable code.
- **LavishScript experienced, new to ISXEVE** — skip to [02_Quick_Start_Guide.md](02_Quick_Start_Guide.md), then [04_Core_Concepts.md](04_Core_Concepts.md), then read [03_API_Reference.md](03_API_Reference.md) in detail, then [05_Patterns_And_Best_Practices.md](05_Patterns_And_Best_Practices.md) and [06_Working_Examples.md](06_Working_Examples.md).
- **Building production bots** (you already have a working simple bot) — focus on [05_Patterns_And_Best_Practices.md](05_Patterns_And_Best_Practices.md) and [21_Advanced_Scripting_Patterns.md](21_Advanced_Scripting_Patterns.md) for architecture, the domain guides [15_Combat_Automation.md](15_Combat_Automation.md) / [16_Mining_And_Hauling.md](16_Mining_And_Hauling.md) / [17_Fleet_Operations.md](17_Fleet_Operations.md) for your focus area, [18_Bot_Architecture_Analysis.md](18_Bot_Architecture_Analysis.md) to learn from [EVEBot](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable), [Yamfa](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/Yamfa), and [Tehbot](https://github.com/isxGames/Tehbot), and [07_Advanced_Patterns_And_Examples.md](07_Advanced_Patterns_And_Examples.md) for multi-boxing and relay/IPC.

For more granular per-task navigation, see [Quick Navigation > By Task](#by-task) above. For narrative summaries of each guide file, see [Documentation Structure](#documentation-structure) near the top of this README.
<!-- CLAUDE_SKIP_END -->

---

## Key Concepts

### Top-Level Objects (TLOs)

TLOs are your entry points into the game's data. The most important ones are:

- `${Me}` - Your character (wallet, skills, standing, etc.)
- `${MyShip}` - Your current ship (health, capacitor, modules, cargo)
- `${EVE}` - Game state and utilities
- `${Local}` - Local chat channel
- `${Station}` - Current station (if docked)
- `${Entity[...]}` - Access any entity by ID or query

### Datatypes

ISXEVE uses a strongly-typed system where each object has a specific datatype:

- `${Me}` returns a **pilot** datatype
- `${MyShip.Cargo[0]}` returns a **item** datatype
- `${MyShip.Module[1]}` returns a **module** datatype

Each datatype has **members** (properties) and **methods** (actions).

### Events

ISXEVE provides events you can react to:

- Entity appearance/disappearance
- Module activation/deactivation
- Inventory changes
- Chat messages
- Target lock events
- And more...

---

<!-- CLAUDE_SKIP_START -->
## Source Materials

This guide was created by analyzing:

1. **ISXEVE Extension** - Official ISXEVE API and documentation
2. **[EVEBot](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable)** - Comprehensive multi-purpose bot (mining, mission running, hauling)
3. **[Yamfa](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/Yamfa)** - Fleet assist and coordination bot
4. **[Tehbot](https://github.com/isxGames/Tehbot)** - Advanced combat automation
5. **[Metatron2](https://github.com/spacekoala420/Metatron2)** - .NET-based bot framework
6. **InnerSpace Documentation** - http://www.lavishsoft.com/wiki/
7. **LavishScript Reference** - http://www.lavishsoft.com/wiki/LavishScript
8. **Production Scripts** - Community-developed automation scripts
<!-- CLAUDE_SKIP_END -->

---

## Important Notes

### Existence Checks

Always check if an object exists before accessing its members:

```lavishscript
if ${Target(exists)}
{
    echo ${Target.Name}
}
```

### NULL Safety

Many ISXEVE members can return NULL. Always validate:

```lavishscript
if ${MyShip.Modules.Get[1](exists)}
{
    MyShip.Modules.Get[1]:Activate
}
```

### Case Insensitivity

LavishScript is case-insensitive for member and method names:

- `${Me.Name}` = `${Me.name}` = `${Me.NAME}`
- `MyShip:Dock` = `MyShip:dock` = `MyShip:DOCK`

---

## Support and Resources

- **InnerSpace Documentation:** http://www.lavishsoft.com/wiki/
- **LavishScript Reference:** http://www.lavishsoft.com/wiki/LavishScript

---

## Contributing

Found an error or have an improvement? This documentation was generated using actual source code analysis and may need updates as ISXEVE evolves. Feel free to contribute corrections and improvements.

---

<!-- CLAUDE_SKIP_START -->
## Version History

- **v3.0 (2026-04-18)**

  - Added guide 19 (DotNet Development) and guide 20 (Debugging and Troubleshooting capstone)
  - Added Module Control Layer chapter to guide 15, Unified Movement Facade to guide 21, Multiboxing Patterns to guide 17, Tehbot dynamic-registry and mission-data patterns to guide 18, GetFallthroughObject metaprogramming to guide 04, and enrichment patterns E3/T10/T13/E9/E10/E11/T14 across guides 05/15/20/21
  - Full source-verification sweep removed ~60 fabricated APIs across every guide: members like `Me.Fleet.MemberCount`, `MyShip.ActiveTarget`, `MyShip.HiSlots`/`MedSlots`, `MyShip.ModuleCount`, chained `Shield.Pct`/`Capacitor.Pct`/`Armor.Pct`; methods like `Me:UnlockAll`, `MyShip:SetActiveTarget`, `EVE.GetTargetDrones`, `EVE:SetWaypointPreference`, `EVE_OnFleetBroadcast`; invalid Module forms (`[=TypeName]`, plain-name, two-arg `[Slot, N]`); fabricated commands (`CmdOpenCharactersheet`, `CmdOpenReprocessingPlant`, `CmdShowInfo`, `CmdLookAtItem`, `CmdDronesLaunch`, `CmdClearTargets`); inverted method names (`FleetWarpTo` → `WarpFleetTo`); and many others
  - Removed fabricated Yamfa and Tehbot chapters from guides 17 and 18; replaced with pointers to the real scripts and grounded Multiboxing patterns in the real Yamfa source
  - Major deduplication pass (~80 commits) cross-referencing canonical implementations instead of re-teaching: HARDSTOP/Emergency primitives, Yamfa / EVEBot / Tehbot main-loop patterns, relay primers, fleet-combat examples, Orca mining patterns, obj_Profiler/obj_CallCounter → obj_PerformanceProfiler, obj_MyLogger → obj_ComprehensiveLogger, obj_FleetSync → obj_FleetStatus, dock/undock and MineAsteroid/WarpToBookmark blocks from guide 06 → guide 16, entity-query caching, Distance2 tutorials, mining-laser activation, SafeModuleActivate → ActivateModuleWithRetry, and the duplicate reference chapter in 03_API_Reference
  - TOC audit and cleanup across the whole KB: fixed broken anchors to match GitHub auto-slug rules (21 in guide 20, 18 in guide 07, 7 in guide 17, 4 in guide 19, Anti-Patterns slug-index errors in guide 03); removed orphan TOC entries (Market and Economy, Testing Error Handling, Station Trading, LavishScript Relay System); added missing entries (Additional Critical TLOs, Decision Trees for Architecture Choice, Complete Working Example: Mining Bot, Local Paths, Quick Reference Card, Common Beginner Mistakes, Practice Exercises, Documentation Navigation, Quick Reference entries in guides 10 and 13); renamed and reordered "Running the Script" → "Running Scripts" in guide 02
  - SKIP-tag hygiene audit: wrapped Practice Exercises section in guide 02 with `CLAUDE_SKIP_START`/`CLAUDE_SKIP_END` markers to keep the section out of agent context
  - Rewrote `ISXEVE_QuickReference.md` end-to-end against `ISXEVEChanges.txt` and then verified against the ISXEVE C++ source; found and flagged 8 distinct changelog-vs-source discrepancies (three broken `Broadcast_*` methods that have been `return false;` stubs since 2012, the `SetectByValue` source typo, the `character:OpenCorpHangar` stale-removal-claim, the `ChangeAmmo` Quantity-arg removal never documented, the entity `Approach[distance]` `argc > 1` off-by-one, and the zero-arg `ChangeAmmo` form that was reverted without a changelog entry). Added `ISXEVE_onFrame` event, `chatchannel.PilotCount`, `jammer.ID`/`GetJams`, and other previously-missing members. Fixed `WarpFactor` → `WarpSpeedMultiplier`, `MyShip:Approach`/`Align` signatures, `Entity` TLO forms, `EVE.Bookmark` lookup semantics, and others
  - Claude AI Commands expanded: registered `ISXEVE_QuickReference.md` in the `/isxeve` controller and `ISXEVE-Expert` agent as a first-class reference; added `QUICK_REF_FILE` to the agent's Local Paths block and the controller's User Configuration table; updated `+How To Use This Guide with Claude Code+.md` Step 3 to cover all four paths; cleaned up the Commands-subdirectory README to use proper markdown hyperlinks and list all four directory references; stipulated foreground-by-default subagent execution
  - EVEBot / Tehbot / Navigator framework-context callouts added throughout guides 07, 16, 17, 18, 20 so scripters using those wrappers (or porting away from them) see where vanilla ISXEVE equivalents live
  - Emoji strip pass across guides 06, 07, 15, 16, 18, 20 replacing Unicode status markers (✓✗⚠📡) with OK/FAIL/WARNING labels and U+2248 with `~=` while preserving ASCII-art box diagrams
  - Fleet coordination corrections: clarified fleet-follow broadcast pattern vs `:WarpFleetTo` FC primitive, guarded `bookmark.ToEntity(exists)`, quoted multi-word skill names in UplinkManager relay, fixed `MonitorFleetHealth` fleet-member-to-entity lookup, added receiver-side `RegisterEvent` to Broadcast-to-Fleet examples
  - LavishScript correctness sweep: quoted echo arguments, converted C-style `//` to `;` comments in guide 16, standardized code-fence language tag to `lavishscript`, hoisted variable declarations out of loop bodies, fixed operator precedence bugs in warp-wait and TryUseDrones, corrected `Ship.CargoFull` as bool (not percent), replaced invalid `${call Fn}` interpolation with separate `call` + `${Return}`, documented LavishScript escape sequences in file operations, replaced `#define` EVEBot identifiers (GROUPID_STATION, MOVE_APPROACH etc.) with numeric literals in guides 07/15/16
  - LGUI1 corrections in guide 09: standardized widget tags to lowercase with a case-convention note, corrected checkbox-event docs with SDK evidence, cited ISXDK LGUIListBox.h + EVEBot.xml for combobox SelectItem usage, dropped empty polymorphic stub
  - Scripting-patterns corrections: `runscript -reload` flag replaced with endscript+runscript pattern, `FleetHangarCapacity` contradiction resolved, `GetHangarItems` clarified as character/station method (not MyShip scalar), Mode 3 = Warping documented at wait-comparison sites, Distance2 performance claim softened, Math.Distance TLO recommended over hand-rolled sqrt, atom-leak-across-reload trap explained in guide 20
  - Bidirectional cross-references added between guides 18 and 20 (obj_Logger coverage), 17 Leader-Follower and Master Election, and Priority-Based Transitions and Layered Priority
  - Linting, typo fixes, and cross-reference integrity: `evwindow` → `evewindow` in guide 03, `Evebot` → `EVEBot`, dead `__CRITICAL_NEWEST_ISXIM_Reference.md` links removed, membermember → membermethod anchor in 07, broken Script-Gap references cleaned up, stale file counts and guide numbering fixed throughout README

- **v2.0 (2025-10-27)**
  
  - Reorganized into 22 numbered guides (added 7 game-agnostic guides)
  - Added 7 game-agnostic UI/system guides from ISXEQ2 documentation
    - LavishGUI 1 UI Guide (08)
    - Advanced LGUI1 Patterns (09)
    - LavishGUI 2 UI Guide (10)
    - LGUI1 to LGUI2 Migration (11)
    - LGUI2 Scaling System (12)
    - JSON Guide (13)
    - LavishMachine Guide (14)
  - Added master navigation hub (00_MASTER_GUIDE.md)
  - Created quick start guide (02_Quick_Start_Guide.md)
  - Replaced LavishScript fundamentals with beginner-friendly version
  - Added advanced scripting patterns (21_Advanced_Scripting_Patterns.md)
  - Added utility script patterns (22_Utility_Script_Patterns.md)
  - Merged Extension Reference navigation content into README for single entry point
  - Enhanced README with comprehensive learning paths, quick start guides, and checklists
  - Improved organization and cross-referencing
  
- **v1.0 (2024)**

  - Initial documentation by Spacekoala
  - Comprehensive ISXEVE API coverage
  - Analysis of EVEBot, Yamfa, Tehbot
  - LavishScript fundamentals and patterns

  <!-- CLAUDE_SKIP_END -->
