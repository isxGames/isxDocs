# ISXEVE Scripting Guide

**Version:** 2.0
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
   Complete introduction to LavishScript programming. Covers variables, functions, objects, loops, conditionals, and all language fundamentals. **If you're new to LavishScript, read this before the Quick Start Guide.**

3. **[02_Quick_Start_Guide.md](02_Quick_Start_Guide.md)** - Your First ISXEVE Script
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

8. **[08_LavishGUI1_UI_Guide.md](08_LavishGUI1_UI_Guide.md)** - LavishGUI 1 UI Creation
   Complete guide to creating custom user interfaces with LavishGUI 1 XML for InnerSpace scripts.

9. **[09_Advanced_LGUI1_Patterns.md](09_Advanced_LGUI1_Patterns.md)** - Advanced LavishGUI 1 Patterns
   Real-world patterns from production scripts: UI element navigation, dynamic updates, script communication, and advanced list management.

10. **[10_LavishGUI2_UI_Guide.md](10_LavishGUI2_UI_Guide.md)** - LavishGUI 2 UI Creation
    Modern JSON-based UI system guide. Covers LGUI2 fundamentals, JSON packages, templates, data binding, and complete working examples.

11. **[11_LavishGUI1_to_LavishGUI2_Migration.md](11_LavishGUI1_to_LavishGUI2_Migration.md)** - LGUI1 to LGUI2 Migration
    Convert existing LavishGUI 1 XML scripts to LavishGUI 2 JSON. Includes migration patterns, common issues, and side-by-side comparisons.

12. **[12_LGUI2_Scaling_System.md](12_LGUI2_Scaling_System.md)** - UI Scaling System
    Add dynamic UI scaling to LGUI2-based scripts. Reusable library for consistent UI scaling across different screen resolutions.

13. **[13_JSON_Guide.md](13_JSON_Guide.md)** - JSON in LavishScript
    Complete guide to JSON in LavishScript: creating, accessing, modifying JSON data, file I/O, serialization, and complete examples.

14. **[14_LavishMachine_Guide.md](14_LavishMachine_Guide.md)** - LavishMachine (LMAC)
    LavishMachine Virtual Machine guide: async operations, time-based animations, audio control, web requests, task sequencing, and custom task types.

### Specialized Automation Guides

15. **[15_Combat_Automation.md](15_Combat_Automation.md)** - Combat Bot Patterns
    Complete combat automation patterns: target prioritization, weapon systems, tank management, drone control, and analysis of [Tehbot](https://github.com/isxGames/Tehbot) combat patterns.

16. **[16_Mining_And_Hauling.md](16_Mining_And_Hauling.md)** - Mining and Hauling Automation
    Mining bot patterns, hauling logistics, ore management, station trading, and resource optimization.

17. **[17_Fleet_Operations.md](17_Fleet_Operations.md)** - Fleet Coordination
    Fleet coordination patterns, [Yamfa](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/Yamfa) fleet assist analysis, relay systems for multi-character communication, and synchronized operations.

18. **[18_Bot_Architecture_Analysis.md](18_Bot_Architecture_Analysis.md)** - Production Bot Analysis
    In-depth analysis of [EVEBot](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable), [Yamfa](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/Yamfa), and [Tehbot](https://github.com/isxGames/Tehbot) architectures. Learn from battle-tested production scripts.

### Advanced Topics

19. **[19_DotNet_Development.md](19_DotNet_Development.md)** - Scripts vs .NET Programs
    When to use LavishScript vs .NET, [Metatron2](https://github.com/spacekoala420/Metatron2) .NET architecture overview, and hybrid development patterns.

20. **[20_Debugging_And_Troubleshooting.md](20_Debugging_And_Troubleshooting.md)** - Debugging Guide
    Debugging techniques, common problems and solutions, troubleshooting patterns, and diagnostic tools.

21. **[21_Advanced_Scripting_Patterns.md](21_Advanced_Scripting_Patterns.md)** - Advanced Patterns
    Production-grade patterns for script organization, main loops, state machines, error handling, relay communication, and configuration management.

22. **[22_Utility_Script_Patterns.md](22_Utility_Script_Patterns.md)** - Utility Patterns
    Essential utility patterns: custom timers, ISK tracking, position management, input validation, progressive tracking, and complete working examples.

---

<!-- CLAUDE_SKIP_START -->
## Quick Navigation

### By Task

| Task | Documentation |
|------|---------------|
| **Learn LavishScript basics** | [01_LavishScript_Fundamentals.md](01_LavishScript_Fundamentals.md) |
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

### Brand New to EVE Botting?

**Start with these files in order:**

1. **[01_LavishScript_Fundamentals.md](01_LavishScript_Fundamentals.md)** - Learn the language basics
2. **[02_Quick_Start_Guide.md](02_Quick_Start_Guide.md)** - Your first ISXEVE script
3. **[03_API_Reference.md](03_API_Reference.md)** - Core ISXEVE objects (${Me}, ${MyShip}, ${EVE})
4. **[06_Working_Examples.md](06_Working_Examples.md)** - Copy working code examples

**Total Time**: 4-6 hours to get started

---

### Building a Specific Bot?

**Mining Bot**:
- Study **[16_Mining_And_Hauling.md](16_Mining_And_Hauling.md)** - Mining automation patterns
- Reference **[03_API_Reference.md](03_API_Reference.md)** - Entity system and modules
- Copy snippets from **[06_Working_Examples.md](06_Working_Examples.md)** - Mining examples

**Combat Bot**:
- Study **[15_Combat_Automation.md](15_Combat_Automation.md)** - Combat patterns
- Reference **[03_API_Reference.md](03_API_Reference.md)** - Entity targeting and modules
- Study **[18_Bot_Architecture_Analysis.md](18_Bot_Architecture_Analysis.md)** - [Tehbot](https://github.com/isxGames/Tehbot) combat analysis

**Hauling Bot**:
- Study **[16_Mining_And_Hauling.md](16_Mining_And_Hauling.md)** - Hauling and logistics
- Reference **[03_API_Reference.md](03_API_Reference.md)** - Navigation and inventory
- Study **[05_Patterns_And_Best_Practices.md](05_Patterns_And_Best_Practices.md)** - Main loop patterns

**Multi-Boxing Fleet**:
- Study **[17_Fleet_Operations.md](17_Fleet_Operations.md)** - Fleet coordination
- Study **[18_Bot_Architecture_Analysis.md](18_Bot_Architecture_Analysis.md)** - [Yamfa](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/Yamfa) fleet assist analysis
- Study **[07_Advanced_Patterns_And_Examples.md](07_Advanced_Patterns_And_Examples.md)** - Relay/IPC systems
<!-- CLAUDE_SKIP_END -->

---

<!-- CLAUDE_SKIP_START -->
## Learning Paths

### Path 1: Complete Beginner (Never scripted before)

**Time Investment**: 8-12 hours to first working bot

1. **[01_LavishScript_Fundamentals.md](01_LavishScript_Fundamentals.md)** (2-3 hours)
   - Learn variables, functions, loops, conditionals
   - Understand LavishScript syntax

2. **[02_Quick_Start_Guide.md](02_Quick_Start_Guide.md)** (30 minutes)
   - Verify ISXEVE installation
   - Run your first script

3. **[04_Core_Concepts.md](04_Core_Concepts.md)** (1-2 hours)
   - Understand EVE mechanics
   - Learn ISXEVE architecture

4. **[03_API_Reference.md](03_API_Reference.md)** (2-3 hours, skim for now)
   - Bookmark this for reference
   - Understand TLOs (${Me}, ${MyShip}, ${EVE})

5. **[06_Working_Examples.md](06_Working_Examples.md)** (1-2 hours)
   - Copy and run examples
   - Modify them to learn

6. **Start building**: Pick a simple automation project

---

### Path 2: Know LavishScript, New to ISXEVE

**Time Investment**: 4-6 hours to first working bot

1. **[02_Quick_Start_Guide.md](02_Quick_Start_Guide.md)** (15 minutes)
2. **[04_Core_Concepts.md](04_Core_Concepts.md)** (1 hour)
3. **[03_API_Reference.md](03_API_Reference.md)** (2 hours, detailed read)
4. **[05_Patterns_And_Best_Practices.md](05_Patterns_And_Best_Practices.md)** (1 hour)
5. **[06_Working_Examples.md](06_Working_Examples.md)** (1 hour)
6. **Start building**: Choose bot type (mining/combat/fleet)

---

### Path 3: Building Production Bots

**Prerequisites**: Completed Path 1 or 2, have a working simple bot

1. **[05_Patterns_And_Best_Practices.md](05_Patterns_And_Best_Practices.md)**
   - Main loop architecture
   - State machines
   - Error handling

2. **[21_Advanced_Scripting_Patterns.md](21_Advanced_Scripting_Patterns.md)**
   - Script organization
   - Configuration management
   - Advanced patterns

3. **Specialized Automation** (choose your focus):
   - **[15_Combat_Automation.md](15_Combat_Automation.md)** - Combat bots
   - **[16_Mining_And_Hauling.md](16_Mining_And_Hauling.md)** - Mining/hauling
   - **[17_Fleet_Operations.md](17_Fleet_Operations.md)** - Fleet coordination

4. **[18_Bot_Architecture_Analysis.md](18_Bot_Architecture_Analysis.md)**
   - Learn from [EVEBot](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable), [Yamfa](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/Yamfa), [Tehbot](https://github.com/isxGames/Tehbot)
   - Understand production bot architectures

5. **[07_Advanced_Patterns_And_Examples.md](07_Advanced_Patterns_And_Examples.md)**
   - Multi-boxing
   - Relay/IPC systems
   - Complex automation
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
- **ISXEVE Wiki:** IsxeveWiki/ (included in this guide)
- **LavishScript Wiki:** LavishScriptWiki/ (included in this guide)

---

## Contributing

Found an error or have an improvement? This documentation was generated using actual source code analysis and may need updates as ISXEVE evolves. Feel free to contribute corrections and improvements.

---

<!-- CLAUDE_SKIP_START -->
## Version History

- **2.0** (2025-10-27) - Major documentation expansion
  - Reorganized into 23 comprehensive guides (added 7 game-agnostic UI/system guides)
  - Added game-agnostic guides: LavishGUI 1/2, JSON, LavishMachine
  - Added master navigation hub and quick start guide
  - Beginner-friendly LavishScript fundamentals
  - Advanced scripting patterns and utility patterns
  - Merged Extension Reference navigation content into README for single entry point
  - Enhanced cross-references, learning paths, quick start guides, and checklists
  - Improved organization and navigation structure

- **1.0** (2025) - Initial documentation by Spacekoala
  - Comprehensive coverage of ISXEVE API
  - Analysis of [EVEBot](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable), [Yamfa](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/Yamfa), [Tehbot](https://github.com/isxGames/Tehbot) architectures
  - LavishScript fundamentals and patterns
  <!-- CLAUDE_SKIP_END -->

---

## License

This documentation is provided for educational and reference purposes for ISXEVE script development. ISXEVE, InnerSpace, and EVE Online are properties of their respective owners.

---

*Last Updated: 2025-10-26*
*Comprehensive ISXEVE Scripting Reference*
