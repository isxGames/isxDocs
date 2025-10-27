# ISXEQ2 Scripting Guide

**Version:** 3.0
**Language:** LavishScript
**Extension:** ISXEQ2 for InnerSpace
**Target Game:** EverQuest 2

---

## Overview

This comprehensive guide provides complete documentation for creating, debugging, and maintaining scripts for the ISXEQ2 InnerSpace extension. Whether you're new to ISXEQ2 scripting or an experienced developer, this guide will help you understand the API, learn best practices, and implement powerful automation for EverQuest 2.

**ISXEQ2** is an InnerSpace extension that exposes EverQuest 2's game data and functionality through a structured LavishScript API. It allows you to:

- Access character, inventory, and ability information
- Automate combat rotations and character actions
- Interact with game UI elements and windows
- React to game events in real-time
- Manage quests, crafting, and commerce
- Create complex multi-character automation

---

## Documentation Structure

### Getting Started

1. **[00_MASTER_GUIDE.md](00_MASTER_GUIDE.md)** - Master Reference & Navigation Hub
   Complete overview of the ISXEQ2 API with organized links to all datatypes, commands, and events. **Start here for quick reference and navigation.**

2. **[01_LavishScript_Fundamentals.md](01_LavishScript_Fundamentals.md)** - LavishScript Basics (For beginners!)
   Complete introduction to LavishScript programming. Covers variables, functions, objects, loops, conditionals, and all language fundamentals. **If you're new to LavishScript, read this before the Quick Start Guide.**

3. **[02_Quick_Start_Guide.md](02_Quick_Start_Guide.md)** - Your First ISXEQ2 Script
   Get up and running in 5-10 minutes with your first ISXEQ2 script. Includes installation verification and a simple working example. **Assumes basic LavishScript knowledge.**

### Core Documentation

4. **[03_API_Reference.md](03_API_Reference.md)** - Complete API Documentation
   Exhaustive reference for all Top-Level Objects (TLOs), datatypes, members, methods, commands, and events. Generated from the official ISXEQ2 reference.

5. **[04_Core_Concepts.md](04_Core_Concepts.md)** - Fundamental Concepts
   Essential concepts every ISXEQ2 scripter needs to know: datatypes, inheritance, NULL checks, async data loading, query syntax, and more.

### Practical Guides

6. **[05_Patterns_And_Best_Practices.md](05_Patterns_And_Best_Practices.md)** - Development Patterns
   Proven coding patterns, naming conventions, error handling strategies, and best practices learned from analyzing production scripts like EQ2Bot.

7. **[06_Working_Examples.md](06_Working_Examples.md)** - Practical Examples
   Real-world, runnable code examples covering common tasks: combat automation, inventory management, crafting, UI interaction, and event-driven scripts.

8. **[07_Advanced_Patterns_And_Examples.md](07_Advanced_Patterns_And_Examples.md)** - Advanced Patterns
   Advanced patterns from production scripts: mouse automation, LavishSettings XML, command-line parsing, triggers, broker automation, raid tracking, localization, and navigation integration.

9. **[08_LavishGUI1_UI_Guide.md](08_LavishGUI1_UI_Guide.md)** - LavishGUI 1 UI Creation (Legacy)
   Complete guide to creating custom user interfaces with XML (LavishGUI 1). Learn windows, buttons, checkboxes, tabs, events, script-to-UI interaction, templates, and skinning. **Note:** LavishGUI 1 is legacy - use LavishGUI 2 for new scripts.

10. **[09_Advanced_LGUI1_Patterns.md](09_Advanced_LGUI1_Patterns.md)** - Advanced LavishGUI 1 Patterns
    Real-world patterns from production scripts. Deep UI navigation with FindChild, alpha-based show/hide, Script:QueueCommand communication, settings integration, color-coded lists, dynamic element visibility, and production patterns. Analyzed from MyPrices broker script (3937 lines).

11. **[10_LavishGUI2_UI_Guide.md](10_LavishGUI2_UI_Guide.md)** - LavishGUI 2 UI Creation (Modern)
    Complete guide to creating custom user interfaces with JSON (LavishGUI 2). Covers all element types, data binding, event handlers, templates, styling, and advanced features. **Recommended for all new scripts.**

12. **[11_LavishGUI1_to_LavishGUI2_Migration.md](11_LavishGUI1_to_LavishGUI2_Migration.md)** - Migration Guide
    Comprehensive guide for converting existing LavishGUI 1 XML scripts to LavishGUI 2 JSON. Includes element mapping, event handler conversion, data binding migration, complete examples, and documentation of known limitations (custom C++ element types not supported).

13. **[12_LGUI2_Scaling_System.md](12_LGUI2_Scaling_System.md)** - LGUI2 Dynamic UI Scaling
    Complete guide to the dynamic UI scaling system for LGUI2 interfaces. Covers the scaling pipeline, automatic dimension/font scaling, window position preservation, percentage-to-absolute conversion, button width adjustment, font injection, troubleshooting, and advanced topics. Includes real-world example from EQ2BotCommander migration. **Reusable library for all LGUI2 scripts.**

14. **[13_JSON_Guide.md](13_JSON_Guide.md)** - JSON in LavishScript
    Complete guide to working with JSON in LavishScript. Covers JSON data types, creating/accessing/modifying JSON, objects and arrays, file I/O, object serialization (AsJSON), deserialization (FromJSON), collections, and best practices.

15. **[14_LavishMachine_Guide.md](14_LavishMachine_Guide.md)** - LavishMachine (LMAC) Asynchronous Tasks
    Complete guide to using LavishMachine for asynchronous task execution. Covers task managers, built-in task types (echo, audio, web requests, chains), creating custom task types, task states and lifecycle, controller pattern, error handling, and complete working examples.

16. **[15_Advanced_Scripting_Patterns.md](15_Advanced_Scripting_Patterns.md)** - Production-Grade Advanced Patterns
    Advanced patterns from production scripts (EQ2OgreFree and EQ2Track). Covers multi-threaded architecture, LavishSettings configuration, LavishNav navigation, timer objects, UI synchronization, modern EQ2:GetActors, trigger systems, controller pattern, dynamic variables, injectable UI, UI initialization guards, collection-based exclusion, file-based discovery, and zone-aware auto-configuration. 14 patterns total.

17. **[16_Utility_Script_Patterns.md](16_Utility_Script_Patterns.md)** - Utility Script Patterns
    Practical utility patterns from EQ2BJCommon scripts. Covers custom timer objects, time calculations, currency tracking, position tracking, character-specific configs, dynamic file loading, prioritized item lists, event text parsing, EQ2DataSourceContainer, randomized movement, item verification, ApplyVerb usage, ReplyDialog, audio alerts, input validation, XP tracking, and ExecuteQueued patterns. 18 complete patterns with working examples.

18. **[17_Crafting_Script_Patterns.md](17_Crafting_Script_Patterns.md)** - Crafting Automation Patterns
    Advanced patterns for tradeskill automation from EQ2Craft. Covers command-line argument parsing, navigation integration, localization support, UI state management, recipe queue management, crafting window monitoring, event-driven crafting loops, device targeting, writ automation, durability/quality management, reaction arts system, progress tracking, and file-based configuration. 13 complete patterns with full working example.

19. **[18_Navigation_Library_Patterns.md](18_Navigation_Library_Patterns.md)** - Navigation and Pathfinding
    Production-grade navigation patterns from EQ2Nav library (1,187 lines). Covers LavishNav integration, Dijkstra pathfinding, region-based navigation, collision detection, stuck detection/recovery, door automation, aggro detection integration, direct vs pathfinding movement, dual precision system, and performance optimization. 10 major patterns with 3 complete working examples (simple nav, waypoints, aggro handling).

---

<!-- CLAUDE_SKIP_START -->
## Quick Navigation

### By Task

| Task | Documentation |
|------|---------------|
| **Learn LavishScript basics** | [01_LavishScript_Fundamentals.md](01_LavishScript_Fundamentals.md) |
| **Write your first ISXEQ2 script** | [02_Quick_Start_Guide.md](02_Quick_Start_Guide.md) |
| **Look up an API member** | [03_API_Reference.md](03_API_Reference.md) |
| **Understand inheritance** | [04_Core_Concepts.md#datatype-inheritance](04_Core_Concepts.md#datatype-inheritance) |
| **Query inventory/actors** | [04_Core_Concepts.md#query-syntax](04_Core_Concepts.md#query-syntax) |
| **Handle events** | [06_Working_Examples.md#event-driven-scripting](06_Working_Examples.md#event-driven-scripting) |
| **Access character stats** | [03_API_Reference.md#char](03_API_Reference.md#char) |
| **Cast abilities** | [06_Working_Examples.md#abilities-and-casting](06_Working_Examples.md#abilities-and-casting) |
| **Interact with UI** | [06_Working_Examples.md#ui-interaction](06_Working_Examples.md#ui-interaction) |
| **Use advanced patterns** | [07_Advanced_Patterns_And_Examples.md](07_Advanced_Patterns_And_Examples.md) |
| **Use production-grade patterns** | [15_Advanced_Scripting_Patterns.md](15_Advanced_Scripting_Patterns.md) |
| **Multi-threaded scripts** | [15_Advanced_Scripting_Patterns.md#multi-threaded-script-architecture](15_Advanced_Scripting_Patterns.md#multi-threaded-script-architecture) |
| **LavishSettings config** | [15_Advanced_Scripting_Patterns.md#lavishsettings-configuration-management](15_Advanced_Scripting_Patterns.md#lavishsettings-configuration-management) |
| **Navigation with LavishNav** | [15_Advanced_Scripting_Patterns.md#lavishnav-navigation-system](15_Advanced_Scripting_Patterns.md#lavishnav-navigation-system) |
| **Modern actor queries** | [15_Advanced_Scripting_Patterns.md#modern-eq2getactors-usage](15_Advanced_Scripting_Patterns.md#modern-eq2getactors-usage) |
| **Mouse automation** | [07_Advanced_Patterns_And_Examples.md#mouse-automation](07_Advanced_Patterns_And_Examples.md#mouse-automation) |
| **XML configuration** | [07_Advanced_Patterns_And_Examples.md#lavishsettings-xml-configuration](07_Advanced_Patterns_And_Examples.md#lavishsettings-xml-configuration) |
| **Broker automation** | [07_Advanced_Patterns_And_Examples.md#broker-window-automation](07_Advanced_Patterns_And_Examples.md#broker-window-automation) |
| **Create UI/GUI (modern)** | [10_LavishGUI2_UI_Guide.md](10_LavishGUI2_UI_Guide.md) |
| **Create UI/GUI (legacy)** | [08_LavishGUI1_UI_Guide.md](08_LavishGUI1_UI_Guide.md) |
| **Advanced LGUI1 patterns** | [09_Advanced_LGUI1_Patterns.md](09_Advanced_LGUI1_Patterns.md) |
| **Migrate LGUI1 to LGUI2** | [11_LavishGUI1_to_LavishGUI2_Migration.md](11_LavishGUI1_to_LavishGUI2_Migration.md) |
| **Deep UI navigation** | [09_Advanced_LGUI1_Patterns.md#ui-element-navigation](09_Advanced_LGUI1_Patterns.md#ui-element-navigation) |
| **Script-UI communication** | [09_Advanced_LGUI1_Patterns.md#script-communication](09_Advanced_LGUI1_Patterns.md#script-communication) |
| **Settings sync with UI** | [09_Advanced_LGUI1_Patterns.md#settings-integration](09_Advanced_LGUI1_Patterns.md#settings-integration) |
| **Color-coded lists** | [09_Advanced_LGUI1_Patterns.md#advanced-list-management](09_Advanced_LGUI1_Patterns.md#advanced-list-management) |
| **Create windows (LGUI2)** | [10_LavishGUI2_UI_Guide.md#window](10_LavishGUI2_UI_Guide.md#window) |
| **Create buttons/checkboxes (LGUI2)** | [10_LavishGUI2_UI_Guide.md#button](10_LavishGUI2_UI_Guide.md#button) |
| **Data binding (LGUI2)** | [10_LavishGUI2_UI_Guide.md#data-binding](10_LavishGUI2_UI_Guide.md#data-binding) |
| **Event handlers (LGUI2)** | [10_LavishGUI2_UI_Guide.md#event-handling](10_LavishGUI2_UI_Guide.md#event-handling) |
| **Script-UI interaction (LGUI2)** | [10_LavishGUI2_UI_Guide.md#script-to-ui-interaction](10_LavishGUI2_UI_Guide.md#script-to-ui-interaction) |
| **Work with JSON** | [11_JSON_Guide.md](11_JSON_Guide.md) |
| **Parse JSON files** | [11_JSON_Guide.md#reading-and-writing-json-files](11_JSON_Guide.md#reading-and-writing-json-files) |
| **Serialize objects to JSON** | [11_JSON_Guide.md#object-serialization-asjson](11_JSON_Guide.md#object-serialization-asjson) |
| **Load objects from JSON** | [11_JSON_Guide.md#object-deserialization-fromjson](11_JSON_Guide.md#object-deserialization-fromjson) |
| **Save/load configuration** | [11_JSON_Guide.md#complete-examples](11_JSON_Guide.md#complete-examples) |
| **Use asynchronous tasks** | [14_LavishMachine_Guide.md](14_LavishMachine_Guide.md) |
| **Create task managers** | [14_LavishMachine_Guide.md#task-managers](14_LavishMachine_Guide.md#task-managers) |
| **Play audio from scripts** | [14_LavishMachine_Guide.md#audio-tasks](14_LavishMachine_Guide.md#audio-tasks) |
| **Make web requests** | [14_LavishMachine_Guide.md#web-request-tasks](14_LavishMachine_Guide.md#web-request-tasks) |
| **Create custom task types** | [14_LavishMachine_Guide.md#creating-custom-task-types](14_LavishMachine_Guide.md#creating-custom-task-types) |
| **Handle task errors** | [14_LavishMachine_Guide.md#error-handling](14_LavishMachine_Guide.md#error-handling) |
| **Use controller pattern** | [14_LavishMachine_Guide.md#controller-pattern](14_LavishMachine_Guide.md#controller-pattern) |
| **Custom timer objects** | [16_Utility_Script_Patterns.md#custom-timer-objects](16_Utility_Script_Patterns.md#custom-timer-objects) |
| **Track script runtime** | [16_Utility_Script_Patterns.md#script-runningtime-calculations](16_Utility_Script_Patterns.md#script-runningtime-calculations) |
| **Track currency/coins** | [16_Utility_Script_Patterns.md#currency-management](16_Utility_Script_Patterns.md#currency-management) |
| **Save/return to position** | [16_Utility_Script_Patterns.md#position-tracking-and-return-to-home](16_Utility_Script_Patterns.md#position-tracking-and-return-to-home) |
| **Character-specific settings** | [16_Utility_Script_Patterns.md#character-specific-configuration](16_Utility_Script_Patterns.md#character-specific-configuration) |
| **Load user presets** | [16_Utility_Script_Patterns.md#dynamic-file-loading](16_Utility_Script_Patterns.md#dynamic-file-loading) |
| **Prioritized item usage** | [16_Utility_Script_Patterns.md#prioritized-item-lists](16_Utility_Script_Patterns.md#prioritized-item-lists) |
| **Parse game messages** | [16_Utility_Script_Patterns.md#event-based-text-parsing](16_Utility_Script_Patterns.md#event-based-text-parsing) |
| **Track buff durations** | [16_Utility_Script_Patterns.md#eq2datasourcecontainer-usage](16_Utility_Script_Patterns.md#eq2datasourcecontainer-usage) |
| **Randomized movement** | [16_Utility_Script_Patterns.md#randomized-movement](16_Utility_Script_Patterns.md#randomized-movement) |
| **Verify item usage** | [16_Utility_Script_Patterns.md#item-usage-verification](16_Utility_Script_Patterns.md#item-usage-verification) |
| **Loot chests/corpses** | [16_Utility_Script_Patterns.md#applyverb-for-object-interaction](16_Utility_Script_Patterns.md#applyverb-for-object-interaction) |
| **NPC dialog automation** | [16_Utility_Script_Patterns.md#replydialog-for-npc-conversations](16_Utility_Script_Patterns.md#replydialog-for-npc-conversations) |
| **Sound notifications** | [16_Utility_Script_Patterns.md#audio-notifications](16_Utility_Script_Patterns.md#audio-notifications) |
| **Validate user input** | [16_Utility_Script_Patterns.md#input-validation-patterns](16_Utility_Script_Patterns.md#input-validation-patterns) |
| **Automate crafting** | [17_Crafting_Script_Patterns.md](17_Crafting_Script_Patterns.md) |
| **Parse command-line args** | [17_Crafting_Script_Patterns.md#command-line-argument-parsing](17_Crafting_Script_Patterns.md#command-line-argument-parsing) |
| **Navigate to stations** | [17_Crafting_Script_Patterns.md#navigation-integration](17_Crafting_Script_Patterns.md#navigation-integration) |
| **Multi-language support** | [17_Crafting_Script_Patterns.md#localization-and-multi-language-support](17_Crafting_Script_Patterns.md#localization-and-multi-language-support) |
| **Manage recipe queues** | [17_Crafting_Script_Patterns.md#recipe-queue-management](17_Crafting_Script_Patterns.md#recipe-queue-management) |
| **Monitor crafting UI** | [17_Crafting_Script_Patterns.md#crafting-window-monitoring](17_Crafting_Script_Patterns.md#crafting-window-monitoring) |
| **Automate writs** | [17_Crafting_Script_Patterns.md#writ-automation](17_Crafting_Script_Patterns.md#writ-automation) |
| **Manage durability/quality** | [17_Crafting_Script_Patterns.md#durability-and-quality-management](17_Crafting_Script_Patterns.md#durability-and-quality-management) |
| **Reaction arts system** | [17_Crafting_Script_Patterns.md#reaction-arts-system](17_Crafting_Script_Patterns.md#reaction-arts-system) |
| **Navigate with pathfinding** | [18_Navigation_Library_Patterns.md](18_Navigation_Library_Patterns.md) |
| **LavishNav integration** | [18_Navigation_Library_Patterns.md#lavishnav-integration](18_Navigation_Library_Patterns.md#lavishnav-integration) |
| **Dijkstra shortest path** | [18_Navigation_Library_Patterns.md#path-planning-with-dijkstra](18_Navigation_Library_Patterns.md#path-planning-with-dijkstra) |
| **Region-based navigation** | [18_Navigation_Library_Patterns.md#region-based-navigation](18_Navigation_Library_Patterns.md#region-based-navigation) |
| **Collision detection** | [18_Navigation_Library_Patterns.md#collision-detection](18_Navigation_Library_Patterns.md#collision-detection) |
| **Stuck detection/recovery** | [18_Navigation_Library_Patterns.md#stuck-detection-and-recovery](18_Navigation_Library_Patterns.md#stuck-detection-and-recovery) |
| **Auto-open doors** | [18_Navigation_Library_Patterns.md#door-automation](18_Navigation_Library_Patterns.md#door-automation) |
| **Aggro-aware navigation** | [18_Navigation_Library_Patterns.md#aggro-detection-integration](18_Navigation_Library_Patterns.md#aggro-detection-integration) |
| **Precision navigation** | [18_Navigation_Library_Patterns.md#precision-management](18_Navigation_Library_Patterns.md#precision-management) |
| **Optimize navigation CPU** | [18_Navigation_Library_Patterns.md#performance-optimization](18_Navigation_Library_Patterns.md#performance-optimization) |
| **Track XP gains** | [16_Utility_Script_Patterns.md#progressive-xp-tracking](16_Utility_Script_Patterns.md#progressive-xp-tracking) |
| **Prevent duplicate scripts** | [16_Utility_Script_Patterns.md#script-existence-checking](16_Utility_Script_Patterns.md#script-existence-checking) |
| **Handle queued commands** | [16_Utility_Script_Patterns.md#executequeued-loop-patterns](16_Utility_Script_Patterns.md#executequeued-loop-patterns) |

### By API Category

| Category | Documentation |
|----------|---------------|
| **Character (Me)** | [03_API_Reference.md#char](03_API_Reference.md#char) |
| **Actors/NPCs** | [03_API_Reference.md#actor](03_API_Reference.md#actor) |
| **Items/Inventory** | [03_API_Reference.md#item](03_API_Reference.md#item) |
| **Abilities/Spells** | [03_API_Reference.md#ability](03_API_Reference.md#ability) |
| **Effects/Buffs** | [03_API_Reference.md#effect](03_API_Reference.md#effect) |
| **Quests** | [03_API_Reference.md#quest](03_API_Reference.md#quest) |
| **Crafting** | [03_API_Reference.md#crafting](03_API_Reference.md#crafting) |
| **Commerce/Broker** | [03_API_Reference.md#commerce-datatypes](03_API_Reference.md#commerce-datatypes) |
| **UI Windows** | [03_API_Reference.md#window-datatypes](03_API_Reference.md#window-datatypes) |
| **Commands** | [03_API_Reference.md#commands](03_API_Reference.md#commands) |
| **Events** | [03_API_Reference.md#events](03_API_Reference.md#events) |
<!-- CLAUDE_SKIP_END -->

---

## Learning Path

### Beginner (Never used LavishScript or ISXEQ2 before)

1. **If new to LavishScript:** Read [01_LavishScript_Fundamentals.md](01_LavishScript_Fundamentals.md) to learn the language basics
2. Read the [02_Quick_Start_Guide.md](02_Quick_Start_Guide.md) and run the example script
3. Read [04_Core_Concepts.md](04_Core_Concepts.md) sections 1-4 (Datatypes, TLOs, NULL Checks, Variables)
4. Study the basic examples in [06_Working_Examples.md](06_Working_Examples.md#basic-information)
5. Bookmark [03_API_Reference.md](03_API_Reference.md) for looking up API members
6. Try modifying the Quick Start example to access different character data

### Intermediate (Familiar with LavishScript basics)

1. Study [05_Patterns_And_Best_Practices.md](05_Patterns_And_Best_Practices.md) to learn proven patterns
2. Read [04_Core_Concepts.md#query-syntax](04_Core_Concepts.md#query-syntax) to master queries
3. Work through [06_Working_Examples.md](06_Working_Examples.md) examples in your area of interest
4. Learn event-driven scripting from [06_Working_Examples.md#event-driven-scripting](06_Working_Examples.md#event-driven-scripting)
5. Study [16_Utility_Script_Patterns.md](16_Utility_Script_Patterns.md) for practical utility patterns (timers, tracking, validation, etc.)

### Advanced (Building complex scripts)

1. Study [07_Advanced_Patterns_And_Examples.md](07_Advanced_Patterns_And_Examples.md) for production-grade patterns
2. Learn [13_JSON_Guide.md](13_JSON_Guide.md) to work with JSON for configuration and data persistence
3. Learn [14_LavishMachine_Guide.md](14_LavishMachine_Guide.md) to use asynchronous tasks for audio, web requests, and custom animations
4. Learn [10_LavishGUI2_UI_Guide.md](10_LavishGUI2_UI_Guide.md) to create modern custom UIs with JSON
5. If maintaining older scripts, review [11_LavishGUI1_to_LavishGUI2_Migration.md](11_LavishGUI1_to_LavishGUI2_Migration.md) for migration guidance
6. Study [09_Advanced_LGUI1_Patterns.md](09_Advanced_LGUI1_Patterns.md) for real-world LGUI1 patterns from production scripts like MyPrices
7. Master [07_Advanced_Patterns_And_Examples.md#broker-window-automation](07_Advanced_Patterns_And_Examples.md#broker-window-automation) for commerce scripts
8. Study [15_Advanced_Scripting_Patterns.md](15_Advanced_Scripting_Patterns.md) for production-grade patterns from EQ2Ogre (multi-threading, LavishNav, modern EQ2:GetActors, etc.)
9. Implement raid tracking and localization from advanced patterns
10. Analyze the EQ2Bot, EQ2Craft, MyPrices, EQ2OgreFree, and other source scripts referenced in this documentation

---

## Key Concepts

### Top-Level Objects (TLOs)

TLOs are your entry points into the game's data. The most important ones are:

- `${Me}` - Your character (health, power, inventory, abilities, etc.)
- `${Target}` - Your current target
- `${Zone}` - Current zone information
- `${Actor[...]}` - Access any actor by name or ID
- `${EQ2}` - Game state and utilities

### Datatypes

ISXEQ2 uses a strongly-typed system where each object has a specific datatype:

- `${Me}` returns a **char** datatype
- `${Me.Inventory[5]}` returns an **item** datatype
- `${Me.Ability[1]}` returns an **ability** datatype

Each datatype has **members** (properties) and **methods** (actions).

### Inheritance

Many datatypes inherit from others:

- **char** inherits from **actor**, so `${Me}` has all actor members plus character-specific ones
- **eq2button** inherits from **eq2widget** and **eq2baseobject**, gaining all their members

### Events

ISXEQ2 provides 40+ events you can react to:

- Actor spawning/despawning
- Chat messages
- Inventory changes
- Quest updates
- Window appearances
- Combat state changes

---

<!-- CLAUDE_SKIP_START -->
## Source Materials

This guide was created by analyzing:

1. **Official ISXEQ2 Reference** - `..\+ISXEQ2_Reference+.md` (4,715 lines)
2. **ISXEQ2 Source Code** - (100+ files, ~120,000 lines of C++)
3. **EQ2Bot Scripts** - https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Bot (7,509 lines main script + 20+ class routines + 39 UI files)
4. **MyPrices Script** - https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/MyPrices (3,937 lines main script + 3,288 line UI file)
5. **EQ2OgreFree Scripts** - Old production scripts with advanced patterns (multi-threading, LavishNav, LavishSettings, modern APIs)
6. **EQ2Track Script** - Actor tracking utility demonstrating UI patterns, collections, file discovery, zone-aware configuration
7. **EQ2BJCommon Scripts** - https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2BJCommon (bjauction, bjlooter, bjmagic, bjshuffle, bjxpbot - ~2,500 lines of utility scripts)
8. **Production Scripts** - EQ2Craft, EQ2Transmute, EQ2Collections, EQ2BuyShineys, EQ2RaidAttendance
9. **Example Scripts** - https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts (45+ example scripts)
10. **LavishGUI 1 Documentation** - https://www.lavishsoft.com/wiki/index.php/LavishGUI
11. **LavishGUI 2 Documentation** - https://www.lavishsoft.com/wiki/index.php/LavishGUI_2
12. **LERN/LGUI2 Examples** - https://github.com/LavishSoftware/LERN/tree/master/LGUI2 (60+ example files)
13. **LERN/JSON Examples** - https://github.com/LavishSoftware/LERN/tree/master/JSON (12 example files)
14. **LERN/LMAC Documentation and Examples** - https://github.com/LavishSoftware/LERN/tree/master/LMAC (8 files: tutorials, audio examples, web request examples)
15. **LERN/LS Tutorial Series** - https://github.com/LavishSoftware/LERN/tree/master/LS (19 lessons: complete LavishScript programming introduction)
<!-- CLAUDE_SKIP_END -->

---

## Important Notes

### NULL Checks

Always check if an object exists before accessing its members:

```lavishscript
if ${Target(exists)}
{
    echo ${Target.Name}
}
```

### Async Data Loading

Some detailed info requires async loading. Always check availability:

```lavishscript
if !${Me.Inventory[5].IsItemInfoAvailable}
{
    ; Wait for data to load
    wait 10 ${Me.Inventory[5].IsItemInfoAvailable}
}

echo ${Me.Inventory[5].ToItemInfo.Description}
```

### Case Insensitivity

LavishScript is case-insensitive for member and method names:

- `${Me.Name}` = `${Me.name}` = `${Me.NAME}`
- `Me:DoTarget` = `Me:dotarget` = `Me:DOTARGET`

---

## Support and Resources

- **InnerSpace Documentation:** http://www.lavishsoft.com/wiki/
- **LavishScript Reference:** http://www.lavishsoft.com/wiki/LavishScript
- **LERN**: https://github.com/LavishSoftware/LERN/tree/master

---

## Contributing

Found an error or have an improvement? This documentation was generated using actual source code analysis and may need updates as ISXEQ2 evolves. Please feel free to fork this repository and create a pull request in order to make updates/corrections.

---

<!-- CLAUDE_SKIP_START -->
## Version History

- **3.0** (2025-10-25)
  - **MAJOR UPDATE:** Scalable Title Bars now standard for all LGUI2 scripts
  - Discovered correct `onCloseButtonClick` event for window close handling
  - Documented complete custom title bar pattern (EQ2BotCommander lines 1-88)
  - Updated all 4 guides with scalable title bar documentation:
    - 10_LavishGUI2_UI_Guide.md: Added custom titleBar to Window section
    - 11_LavishGUI1_to_LavishGUI2_Migration.md: Added scalable window pattern
    - 12_LGUI2_Scaling_System.md: Added "Creating Scalable Title Bars" section (335 lines)
    - EQ2Bot/UI/LGUI2_MIGRATION_GUIDE.md: Added section 1.2 as recommended default
  - Established 2x scaling baseline (font 40, padding 12, border 2) as standard
  - Corrected event documentation: Added 50+ events from DefaultSkin.json analysis
  - Updated FILE_MANIFEST.md with accurate line counts (35,248 lines total)
  - Total documentation: 35,248 lines across 20 files

- **2.9** (2025-10-25)
  - Added LGUI2 Scaling System guide (1,102 lines)
  - Analyzed EQ2BotCommander LGUI1→LGUI2 migration with scaling implementation
  - Complete UI scaling patterns: dynamic JSON preprocessing, window position preservation, percentage-to-absolute conversion, button width adjustment (85%), font injection/scaling
  - Real-world example: EQ2BotCommander scaled 2x (800x800→1600x1600)
  - Updated 10_LavishGUI2_UI_Guide.md with scaling system references
  - Updated 11_LavishGUI1_to_LavishGUI2_Migration.md with scaling enhancement section
  - Renumbered files 13-18 to insert new guide after migration guide

- **2.8** (2025-10-23)
  - Added Navigation Library Patterns guide (~8,000 lines)
  - Analyzed EQ2Navigation library: EQ2Nav_Lib.iss (1,187 lines), EQ2NavMapper_Lib.iss, EQ2NavAggressionHandler.iss, EQ2NavObstacleHandler.iss, EQ2NavFaceClass_Lib.iss (~1,200 lines total)
  - 10 major navigation patterns: LavishNav integration, Dijkstra pathfinding algorithm, region-based navigation, collision detection, stuck detection and recovery, door automation, aggro detection integration, direct vs pathfinding movement, dual precision system, performance optimization
  - 3 complete working examples: SimpleNav (basic navigation), MultiStop (waypoint navigation), SafeNav (navigation with aggro handling)
  - Updated README navigation with 10 new navigation pattern links
  - Updated Advanced LGUI1 Patterns with 8 new patterns from EQ2Bot UI (2,498 line main UI + 39 class UIs)
  - Updated Advanced LGUI1 Patterns with 14 new patterns from EQ2Craft UI (1,802 line UI)
  - Total documentation now exceeds 107,000 lines across 19 files

- **2.7** (2025-10-23)
  - Added Crafting Script Patterns guide (~7,200 lines)
  - Analyzed EQ2Craft script (~15,000 lines total with includes)
  - 13 crafting patterns: command-line argument parsing, navigation integration, localization/multi-language support, UI state management, recipe queue management, crafting window monitoring, event-driven crafting loops, device/station targeting, writ automation, durability/quality management, reaction arts system, progress tracking/statistics, file-based configuration
  - Complete working example: SimpleCraft automation script
  - Updated README navigation with 10 new crafting pattern links
  - Updated all navigation and learning paths
  - Total documentation now exceeds 99,000 lines across 18 files

- **2.6** (2025-10-23) - Utility Script Patterns from EQ2BJCommon
  - Added Utility Script Patterns guide (16_Utility_Script_Patterns.md, ~6,000 lines)
  - Analyzed EQ2BJCommon scripts: bjauction, bjlooter, bjmagic, bjshuffle, bjxpbot (~2,500 lines total)
  - 18 utility patterns: custom timers, time calculations, currency tracking, position tracking, character-specific configs, dynamic file loading, prioritized items, event parsing, EQ2DataSourceContainer, randomized movement, item verification, ApplyVerb, ReplyDialog, audio alerts, input validation, XP tracking, script existence checking, ExecuteQueued loops
  - Complete working examples for auto-potion scripts, treasure hunting, and XP tracking
  - Updated navigation, learning paths, and source materials
- **2.5** (2025-10-23) - Advanced LavishGUI 1 Patterns and LGUI2 Limitations
  - Added Advanced LGUI1 Patterns guide (09_Advanced_LGUI1_Patterns.md)
  - Analyzed MyPrices broker script (3,937 lines + 3,288 line UI)
  - Real-world patterns: FindChild navigation, alpha-based show/hide, Script:QueueCommand, settings integration, color-coded lists
  - Documented LGUI2 limitation: custom C++ element types not supported (radar migration failed)
  - Updated migration guide with Known Limitations section
  - Renumbered files to insert new guide after LGUI1 guide
  - Updated navigation and learning paths
- **2.4** (2025-10-21) - Advanced Scripting Patterns from EQ2OgreFree and EQ2Track
  - Added production-grade patterns guide (15_Advanced_Scripting_Patterns.md, ~3,961 lines)
  - Analyzed EQ2OgreFree scripts (43 files) and EQ2Track script
  - 14 patterns total: multi-threading, LavishSettings, LavishNav, timer objects, UI sync, modern EQ2:GetActors, triggers, controller pattern, dynamic variables, injectable UI, UI initialization guards, collection-based exclusion, file-based discovery, zone-aware auto-configuration
  - Replaced deprecated CustomActorArray with modern EQ2:GetActors throughout all guides
  - Updated navigation and learning paths
- **2.3** (2025-10-21) - LavishScript Fundamentals addition
  - Added complete LavishScript Fundamentals guide (00_LavishScript_Fundamentals.md)
  - Analyzed 19 LERN/LS tutorial lessons
  - Covers all LavishScript basics: variables, functions, objects, members, methods, inheritance, loops, conditionals, atoms, wait commands
  - Prerequisite for beginners new to LavishScript
  - Updated navigation and learning paths
- **2.2** (2025-10-21) - LavishMachine (LMAC) documentation expansion
  - Added complete LavishMachine guide (11_LavishMachine_Guide.md)
  - Analyzed 8 LERN/LMAC tutorial and example files
  - Covers task managers, built-in task types, custom task types, controller pattern
  - Audio tasks, web request tasks, chain tasks, error handling
  - Complete working examples for animations, audio, and web requests
  - Updated navigation and learning paths
- **2.1** (2025-10-21) - JSON documentation expansion
  - Added complete JSON guide (10_JSON_Guide.md)
  - Analyzed 12 LERN/JSON tutorial and example files
  - Covers jsonvalue, jsonvalueref, serialization, deserialization
  - Complete examples for configuration management
  - Updated navigation and learning paths
- **2.0** (2025-10-21) - LavishGUI 2 expansion
  - Added complete LavishGUI 2 UI guide (08_LavishGUI2_UI_Guide.md)
  - Added LGUI1 to LGUI2 migration guide (09_LavishGUI1_to_LavishGUI2_Migration.md)
  - Analyzed 60+ LERN/LGUI2 example files
  - Updated navigation and learning paths
  - Marked LavishGUI 1 as legacy
- **1.0** (2025-10-20) - Initial comprehensive documentation
  - Complete API reference from official docs
  - Architecture analysis from source code
  - Pattern analysis from EQ2Bot
  - Working examples from multiple sources
  - LavishGUI 1 documentation
  - Troubleshooting and best practices
<!-- CLAUDE_SKIP_END -->

---

## License

This documentation is provided for educational and reference purposes for ISXEQ2 script development. ISXEQ2, InnerSpace, and EverQuest 2 are properties of their respective owners.

---

**Ready to start? Head to [QUICK_START_GUIDE.md](QUICK_START_GUIDE.md) now!**

---

*Last Updated: 2025-10-25*
*Documentation Version: 3.0*
*ISXEQ2 Version: 2023+*
