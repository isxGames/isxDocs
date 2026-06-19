# ISXPantheon Scripting Guide

**Language:** LavishScript
**Extension:** ISXPantheon for InnerSpace
**Target Game:** Pantheon: Rise of the Fallen

---

## Overview

This guide provides documentation for creating, debugging, and maintaining scripts for the ISXPantheon InnerSpace extension. Whether you're new to ISXPantheon scripting or an experienced developer, it will help you understand the API, learn best practices, and use the LavishScript and LavishGUI platform effectively.

**ISXPantheon** is an InnerSpace extension for Pantheon: Rise of the Fallen, built on the same LavishScript foundation as the other isxGames extensions. Today it exposes a small, stable surface through the `${ISXPantheon}` Top-Level Object, plus the `GetURL`/`PostURL` HTTP commands and the `isxGames_onHTTPResponse` event. With it you can:

- Query the extension's version and readiness state
- Store and retrieve custom variables
- Format currency strings and round numeric values
- Make HTTP requests and react to the responses
- Build custom user interfaces with LavishGUI 1 and LavishGUI 2
- Use the full LavishScript / InnerSpace platform (JSON, LavishMachine, LavishSettings, LavishNav, triggers, timers)

Game-data automation is **planned but not yet implemented**. The `pantheon`, `entity`, `ability`, and `quest` datatypes are registered but have no working members yet; `${Me}` and `${Radar}` exist only as reserved (commented-out) top-level objects; and a `Where` command, a `Radar` command, and a `Crafting` TLO/datatype are named on the roadmap. Where this guide describes planned features, they are clearly marked as planned — do not write production scripts against them yet.

---

## Documentation Structure

### Getting Started

1. **[00_MASTER_GUIDE.md](00_MASTER_GUIDE.md)** - Master Reference & Navigation Hub
   Overview of the ISXPantheon API with organized links to the real surface, plus the planned roadmap. **Start here for quick reference and navigation.**

2. **[01_LavishScript_Fundamentals.md](01_LavishScript_Fundamentals.md)** - LavishScript Basics (For beginners!)
   Tutorial-style introduction to LavishScript programming. Covers variables, functions, objects, loops, conditionals, and all language fundamentals. **If you're new to LavishScript, read this before the Quick Start Guide.**

3. **[01b_LavishScript_Reference.md](01b_LavishScript_Reference.md)** - LavishScript and Inner Space Reference
   Exhaustive command, datatype, and Top-Level Object inventory for LavishScript core, Inner Space core, and bundled first-party subsystem packages (LavishGUI, LavishGUI 2, LavishNav, LavishSettings, LavishMachine). One canonical entry per feature with links to the authoritative Lavish Software wiki page. **Use this for lookup-style questions; use file 01 for conceptual/tutorial questions.**

4. **[02_Quick_Start_Guide.md](02_Quick_Start_Guide.md)** - Your First ISXPantheon Script
   Get up and running in 5-10 minutes with your first ISXPantheon script. Includes installation verification and a simple working example. **Assumes basic LavishScript knowledge.**

### Core Documentation

4. **[03_API_Reference.md](03_API_Reference.md)** - Complete API Documentation
   Reference for the Top-Level Objects (TLOs), datatypes, members, methods, commands, and events that ISXPantheon currently exposes, plus a clearly marked Planned API section for roadmap features.

5. **[04_Core_Concepts.md](04_Core_Concepts.md)** - Fundamental Concepts
   Essential concepts every ISXPantheon scripter needs to know: datatypes, inheritance, NULL checks, query syntax, events, and more.

### Practical Guides

6. **[05_Patterns_And_Best_Practices.md](05_Patterns_And_Best_Practices.md)** - Development Patterns
   Proven coding patterns, naming conventions, error handling strategies, and best practices for LavishScript automation.

7. **[06_Working_Examples.md](06_Working_Examples.md)** - Practical Examples
   Runnable code examples built on the real surface: version/readiness checks, custom variables, currency formatting, HTTP requests, and UI interaction. Game-data examples are marked as planned.

8. **[07_Advanced_Patterns_And_Examples.md](07_Advanced_Patterns_And_Examples.md)** - Advanced Patterns
   Advanced platform patterns: mouse automation, LavishSettings XML, command-line parsing, triggers, and navigation integration.

9. **[08_LavishGUI1_UI_Guide.md](08_LavishGUI1_UI_Guide.md)** - LavishGUI 1 UI Creation (Legacy)
   Complete guide to creating custom user interfaces with XML (LavishGUI 1). Learn windows, buttons, checkboxes, tabs, events, script-to-UI interaction, templates, and skinning. **Note:** LavishGUI 1 is legacy - use LavishGUI 2 for new scripts.

10. **[09_Advanced_LGUI1_Patterns.md](09_Advanced_LGUI1_Patterns.md)** - Advanced LavishGUI 1 Patterns
    Real-world LavishGUI 1 patterns: deep UI navigation with FindChild, alpha-based show/hide, Script:QueueCommand communication, settings integration, color-coded lists, and dynamic element visibility.

11. **[10_LavishGUI2_UI_Guide.md](10_LavishGUI2_UI_Guide.md)** - LavishGUI 2 UI Creation (Modern)
    Complete guide to creating custom user interfaces with JSON (LavishGUI 2). Covers all element types, data binding, event handlers, templates, styling, and advanced features. **Recommended for all new scripts.**

12. **[11_LavishGUI1_to_LavishGUI2_Migration.md](11_LavishGUI1_to_LavishGUI2_Migration.md)** - Migration Guide
    Comprehensive guide for converting existing LavishGUI 1 XML scripts to LavishGUI 2 JSON. Includes element mapping, event handler conversion, data binding migration, complete examples, and documentation of known limitations (custom C++ element types not supported).

13. **[12_LGUI2_Scaling_System.md](12_LGUI2_Scaling_System.md)** - LGUI2 Dynamic UI Scaling
    Complete guide to the dynamic UI scaling system for LGUI2 interfaces. Covers the scaling pipeline, automatic dimension/font scaling, window position preservation, percentage-to-absolute conversion, button width adjustment, font injection, troubleshooting, and advanced topics. **Reusable library for all LGUI2 scripts.**

14. **[13_JSON_Guide.md](13_JSON_Guide.md)** - JSON in LavishScript
    Complete guide to working with JSON in LavishScript. Covers JSON data types, creating/accessing/modifying JSON, objects and arrays, file I/O, object serialization (AsJSON), deserialization (FromJSON), collections, and best practices.

15. **[14_LavishMachine_Guide.md](14_LavishMachine_Guide.md)** - LavishMachine (LMAC) Asynchronous Tasks
    Complete guide to using LavishMachine for asynchronous task execution. Covers task managers, built-in task types (echo, audio, web requests, chains), creating custom task types, task states and lifecycle, controller pattern, error handling, and complete working examples.

16. **[15_Advanced_Scripting_Patterns.md](15_Advanced_Scripting_Patterns.md)** - Production-Grade Advanced Patterns
    Advanced platform patterns: multi-threaded architecture, LavishSettings configuration, LavishNav navigation, timer objects, UI synchronization, trigger systems, controller pattern, dynamic variables, injectable UI, UI initialization guards, collection-based exclusion, and file-based discovery.

17. **[16_Utility_Script_Patterns.md](16_Utility_Script_Patterns.md)** - Utility Script Patterns
    Practical utility patterns: custom timer objects, time calculations, dynamic file loading, prioritized lists, event text parsing, randomized movement, audio alerts, input validation, and ExecuteQueued patterns.

18. **[17_Crafting_Script_Patterns.md](17_Crafting_Script_Patterns.md)** - Crafting Automation Patterns (Planned)
    Roadmap stub for crafting automation. Crafting is a planned ISXPantheon feature and is not yet implemented; this file describes the intended direction rather than a working API. Do not write production crafting scripts yet.

19. **[18_Navigation_Library_Patterns.md](18_Navigation_Library_Patterns.md)** - Navigation and Pathfinding
    LavishNav navigation patterns: LavishNav integration, Dijkstra pathfinding, region-based navigation, collision detection, stuck detection/recovery, direct vs pathfinding movement, dual precision system, and performance optimization. Game-data-dependent pieces (aggro handling, door automation) are marked as planned.

20. **[19_DotNet_Development.md](19_DotNet_Development.md)** - .NET Development (Scripts vs .NET)
    Decision-and-orientation guide comparing LavishScript `.iss` scripts against compiled .NET programs. Covers language, tooling, performance, debugging, deployment, and maintenance tradeoffs. A dedicated ISXPantheon .NET interop layer is planned, not yet available.

21. **[20_Debugging_And_Troubleshooting.md](20_Debugging_And_Troubleshooting.md)** - Debugging and Troubleshooting (Capstone)
    Diagnostic capstone covering readiness checks with `${ISXPantheon.IsReady}`, logging strategies with `redirect`, performance profiling, `${Script.*}` introspection, session validation, and a common-problems catalog (most centered on distinguishing real surface from planned features). Cross-links to existing coverage in 02/04/05/07/15/18 rather than duplicating.

---

<!-- CLAUDE_SKIP_START -->
## Quick Navigation

### By Task

| Task | Documentation |
|------|---------------|
| **Learn LavishScript basics (tutorial)** | [01_LavishScript_Fundamentals.md](01_LavishScript_Fundamentals.md) |
| **Look up a LavishScript / Inner Space command, datatype, or TLO (reference)** | [01b_LavishScript_Reference.md](01b_LavishScript_Reference.md) |
| **Write your first ISXPantheon script** | [02_Quick_Start_Guide.md](02_Quick_Start_Guide.md) |
| **Look up an API member** | [03_API_Reference.md](03_API_Reference.md) |
| **Understand inheritance** | [04_Core_Concepts.md#datatype-inheritance](04_Core_Concepts.md#datatype-inheritance) |
| **Make HTTP requests** | [06_Working_Examples.md](06_Working_Examples.md) |
| **Use the ${ISXPantheon} TLO** | [03_API_Reference.md](03_API_Reference.md) |
| **Interact with UI** | [06_Working_Examples.md](06_Working_Examples.md) |
| **Use advanced patterns** | [07_Advanced_Patterns_And_Examples.md](07_Advanced_Patterns_And_Examples.md) |
| **Use production-grade patterns** | [15_Advanced_Scripting_Patterns.md](15_Advanced_Scripting_Patterns.md) |
| **Multi-threaded scripts** | [15_Advanced_Scripting_Patterns.md](15_Advanced_Scripting_Patterns.md) |
| **LavishSettings config** | [15_Advanced_Scripting_Patterns.md](15_Advanced_Scripting_Patterns.md) |
| **Navigation with LavishNav** | [18_Navigation_Library_Patterns.md](18_Navigation_Library_Patterns.md) |
| **Mouse automation** | [07_Advanced_Patterns_And_Examples.md](07_Advanced_Patterns_And_Examples.md) |
| **XML configuration** | [07_Advanced_Patterns_And_Examples.md](07_Advanced_Patterns_And_Examples.md) |
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
| **Work with JSON** | [13_JSON_Guide.md](13_JSON_Guide.md) |
| **Parse JSON files** | [13_JSON_Guide.md#reading-and-writing-json-files](13_JSON_Guide.md#reading-and-writing-json-files) |
| **Serialize objects to JSON** | [13_JSON_Guide.md#object-serialization-asjson](13_JSON_Guide.md#object-serialization-asjson) |
| **Load objects from JSON** | [13_JSON_Guide.md#object-deserialization-fromjson](13_JSON_Guide.md#object-deserialization-fromjson) |
| **Save/load configuration** | [13_JSON_Guide.md#complete-examples](13_JSON_Guide.md#complete-examples) |
| **Use asynchronous tasks** | [14_LavishMachine_Guide.md](14_LavishMachine_Guide.md) |
| **Create task managers** | [14_LavishMachine_Guide.md#task-managers](14_LavishMachine_Guide.md#task-managers) |
| **Play audio from scripts** | [14_LavishMachine_Guide.md#audio-tasks](14_LavishMachine_Guide.md#audio-tasks) |
| **Make web requests** | [14_LavishMachine_Guide.md#web-request-tasks](14_LavishMachine_Guide.md#web-request-tasks) |
| **Create custom task types** | [14_LavishMachine_Guide.md#creating-custom-task-types](14_LavishMachine_Guide.md#creating-custom-task-types) |
| **Handle task errors** | [14_LavishMachine_Guide.md#error-handling](14_LavishMachine_Guide.md#error-handling) |
| **Use controller pattern** | [14_LavishMachine_Guide.md#controller-pattern](14_LavishMachine_Guide.md#controller-pattern) |
| **Custom timer objects** | [16_Utility_Script_Patterns.md](16_Utility_Script_Patterns.md) |
| **Track script runtime** | [16_Utility_Script_Patterns.md](16_Utility_Script_Patterns.md) |
| **Load user presets** | [16_Utility_Script_Patterns.md](16_Utility_Script_Patterns.md) |
| **Parse text with triggers** | [16_Utility_Script_Patterns.md](16_Utility_Script_Patterns.md) |
| **Randomized movement** | [16_Utility_Script_Patterns.md](16_Utility_Script_Patterns.md) |
| **Sound notifications** | [16_Utility_Script_Patterns.md](16_Utility_Script_Patterns.md) |
| **Validate user input** | [16_Utility_Script_Patterns.md](16_Utility_Script_Patterns.md) |
| **Crafting automation (planned)** | [17_Crafting_Script_Patterns.md](17_Crafting_Script_Patterns.md) |
| **Parse command-line args** | [17_Crafting_Script_Patterns.md](17_Crafting_Script_Patterns.md) |
| **Navigate with pathfinding** | [18_Navigation_Library_Patterns.md](18_Navigation_Library_Patterns.md) |
| **LavishNav integration** | [18_Navigation_Library_Patterns.md](18_Navigation_Library_Patterns.md) |
| **Collision detection** | [18_Navigation_Library_Patterns.md](18_Navigation_Library_Patterns.md) |
| **Stuck detection/recovery** | [18_Navigation_Library_Patterns.md](18_Navigation_Library_Patterns.md) |
| **Optimize navigation CPU** | [18_Navigation_Library_Patterns.md](18_Navigation_Library_Patterns.md) |
| **Prevent duplicate scripts** | [16_Utility_Script_Patterns.md](16_Utility_Script_Patterns.md) |
| **Handle queued commands** | [16_Utility_Script_Patterns.md](16_Utility_Script_Patterns.md) |
| **Debug a misbehaving script** | [20_Debugging_And_Troubleshooting.md](20_Debugging_And_Troubleshooting.md) |
| **Profile script performance** | [20_Debugging_And_Troubleshooting.md](20_Debugging_And_Troubleshooting.md) |
| **Validate session state** | [20_Debugging_And_Troubleshooting.md](20_Debugging_And_Troubleshooting.md) |

### By API Category

| Category | Documentation |
|----------|---------------|
| **${ISXPantheon} TLO** | [03_API_Reference.md](03_API_Reference.md) |
| **${Login} TLO + login/realm/UI datatypes** | [03_API_Reference.md](03_API_Reference.md) |
| **Commands (GetURL/PostURL)** | [03_API_Reference.md](03_API_Reference.md) |
| **Events (isxGames_onHTTPResponse)** | [03_API_Reference.md](03_API_Reference.md) |
| **Planned API (roadmap)** | [03_API_Reference.md](03_API_Reference.md) |

---

## Learning Path

### Beginner (Never used LavishScript or ISXPantheon before)

1. **If new to LavishScript:** Read [01_LavishScript_Fundamentals.md](01_LavishScript_Fundamentals.md) to learn the language basics; bookmark [01b_LavishScript_Reference.md](01b_LavishScript_Reference.md) for exhaustive command/datatype/TLO lookup
2. Read the [02_Quick_Start_Guide.md](02_Quick_Start_Guide.md) and run the example script
3. Read [04_Core_Concepts.md](04_Core_Concepts.md) sections 1-4 (Datatypes, TLOs, NULL Checks, Variables)
4. Study the basic examples in [06_Working_Examples.md](06_Working_Examples.md)
5. Bookmark [03_API_Reference.md](03_API_Reference.md) for looking up API members
6. Try modifying the Quick Start example to read different `${ISXPantheon}` members

### Intermediate (Familiar with LavishScript basics)

1. Study [05_Patterns_And_Best_Practices.md](05_Patterns_And_Best_Practices.md) to learn proven patterns
2. Read [04_Core_Concepts.md#datatype-inheritance](04_Core_Concepts.md#datatype-inheritance) to master datatypes and inheritance
3. Work through [06_Working_Examples.md](06_Working_Examples.md) examples in your area of interest
4. Learn event-driven scripting and HTTP handling from [06_Working_Examples.md](06_Working_Examples.md)
5. Study [16_Utility_Script_Patterns.md](16_Utility_Script_Patterns.md) for practical utility patterns (timers, validation, etc.)

### Advanced (Building complex scripts)

1. Study [07_Advanced_Patterns_And_Examples.md](07_Advanced_Patterns_And_Examples.md) for production-grade patterns
2. Learn [13_JSON_Guide.md](13_JSON_Guide.md) to work with JSON for configuration and data persistence
3. Learn [14_LavishMachine_Guide.md](14_LavishMachine_Guide.md) to use asynchronous tasks for audio, web requests, and custom animations
4. Learn [10_LavishGUI2_UI_Guide.md](10_LavishGUI2_UI_Guide.md) to create modern custom UIs with JSON
5. If maintaining older scripts, review [11_LavishGUI1_to_LavishGUI2_Migration.md](11_LavishGUI1_to_LavishGUI2_Migration.md) for migration guidance
6. Study [09_Advanced_LGUI1_Patterns.md](09_Advanced_LGUI1_Patterns.md) for real-world LGUI1 patterns
7. Study [15_Advanced_Scripting_Patterns.md](15_Advanced_Scripting_Patterns.md) for production-grade platform patterns (multi-threading, LavishNav, triggers, etc.)
8. Study [18_Navigation_Library_Patterns.md](18_Navigation_Library_Patterns.md) for LavishNav pathfinding
<!-- CLAUDE_SKIP_END -->

---

## Key Concepts

### Top-Level Objects (TLOs)

TLOs are your entry points into the extension's data. Today the live ones are:

- `${ISXPantheon}` - The extension itself: version, readiness, custom variables, currency formatting, rounding, and lifecycle methods
- `${Login}` - The login / realm-selection screen: login state, realm enumeration, and the login screen's buttons and input fields (via the `login`, `realm`, `uibutton`, `uitext`, `uiinputfield`, and `uicolor` datatypes, which inherit from the base `object` type)
- `${CharSelect}` - The character-selection screen: enumerate and choose characters, enter the world, delete a character, or move to character creation (via `charselect`, `charselect-character`, and `uitoggle`). NULL unless at the character-selection scene
- `${CharCreate}` - The character-creation screen: set name, race, class, gender, appearance (sliders), and spend attribute points (via `charcreate`, `uislider`, `uitoggle`, `uiattributeselection`, and `uiattributeselector`). NULL unless at the character-creation scene
- `${Pantheon}` - Game-wide info and render/camera control: resolution, render quality, FPS limit, VSync, full-screen mode, and camera enumeration (via the `uicamera` datatype)

The `entity`, `ability`, and `quest` datatypes are registered by name but have no working members yet. `${Me}` and `${Radar}` exist in the source only as reserved (commented-out) top-level objects. Additional in-world game-data TLOs are **planned** — see the Planned API section of [03_API_Reference.md](03_API_Reference.md).

### Datatypes

ISXPantheon uses a strongly-typed system where each object has a specific datatype:

- `${ISXPantheon}` returns an **isxpantheon** datatype
- `${ISXPantheon.GetCustomVariable[name]}` returns a typed value

Each datatype has **members** (properties) and **methods** (actions).

### Inheritance

Datatypes can inherit from others, gaining the parent's members. See [04_Core_Concepts.md - Datatype Inheritance](04_Core_Concepts.md#datatype-inheritance) for the full explanation and inheritance chain examples.

### Events

ISXPantheon currently fires one event you can react to:

- `isxGames_onHTTPResponse` - Fired when a `GetURL` / `PostURL` request completes

Game events are **planned** and not yet wired up in the current build.

---

## Important Notes

### NULL Checks

Always check if an object exists before accessing its members:

```lavishscript
if ${ISXPantheon(exists)}
{
    echo ${ISXPantheon.Version}
}
```

### Wait for Readiness

The extension needs time to initialize after it loads. Gate every script on `IsReady` before touching the API:

```lavishscript
while !${ISXPantheon.IsReady}
    wait 10

echo "ISXPantheon ${ISXPantheon.Version} ready"
```

### Case Insensitivity

LavishScript is case-insensitive for member and method names:

- `${ISXPantheon.Version}` = `${ISXPantheon.version}` = `${ISXPantheon.VERSION}`
- `ISXPantheon:Reload` = `ISXPantheon:reload` = `ISXPantheon:RELOAD`

---

## Support and Resources

- **InnerSpace Documentation:** http://www.lavishsoft.com/wiki/
- **LavishScript Reference:** http://www.lavishsoft.com/wiki/LavishScript
- **LERN**: https://github.com/LavishSoftware/LERN/tree/master

---

## Contributing

Found an error or have an improvement? This documentation will need updates as ISXPantheon evolves and planned features ship. Please feel free to fork this repository and create a pull request in order to make updates/corrections.

---

<!-- CLAUDE_SKIP_START -->
## Version History

- **0.3** — Character select / create and render-camera surface documented
  - Documents the now-live `${CharSelect}` and `${CharCreate}` TLOs and their supporting datatypes (`charselect`, `charselect-character`, `charcreate`, `uislider`, `uitoggle`, `uiattributeselection`, `uiattributeselector`): enumerating and choosing characters, entering the world, and driving the character-creation screen (name, race, class, gender, appearance sliders, and attribute spending) — all in [03_API_Reference.md](03_API_Reference.md)
  - Documents the now-live render/camera surface on the `${Pantheon}` datatype (camera enumeration via the new `uicamera` datatype, plus `ResolutionWidth`/`ResolutionHeight`, `RenderQuality`, `FPSLimit`, `VSyncCount`, `FullScreenMode`, `FrameCount`, `NumCameras`, and the `RestoreCameras` / `SetFPSLimit` methods); `SetFPSLimit` also disables VSync because Unity ignores the FPS limit while VSync is on
  - Notes that GUI sliders always have a minimum value of -1 by engine design
  - Changes the `uibutton` `Label` member's return type from `string` to a `uitext` object
  - In-world game-data features (entities, abilities, quests, crafting, navigation, radar, `${Me}`, `${Radar}`) remain planned

- **0.2** — Login / realm / login-UI surface documented
  - Documents the now-live `${Login}` TLO and its supporting datatypes (`login`, `realm`, `uibutton`, `uitext`, `uiinputfield`, `uicolor`, and the base `object` they inherit from): login state, realm enumeration, and reading/driving the login screen's buttons and input fields
  - Adds a read-only Login Screen worked example
  - In-world game-data features (entities, abilities, quests, crafting, navigation, radar, `${Me}`, `${Radar}`, `${Pantheon}` members) remain planned

- **0.1** — Initial ISXPantheon guide
  - First conversion of the shared LavishScript / LavishGUI platform guide for the ISXPantheon extension
  - Documents the current live surface (`${ISXPantheon}` TLO, `GetURL`/`PostURL` commands, `isxGames_onHTTPResponse` event) and marks all game-data features as planned

---

**Ready to start? Head to [02_Quick_Start_Guide.md](02_Quick_Start_Guide.md) now!**

<!-- CLAUDE_SKIP_END -->
