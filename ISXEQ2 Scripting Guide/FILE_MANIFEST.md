<!-- CLAUDE_SKIP_START -->
# ISXEQ2 Scripting Guide - File Manifest

**Project:** ISXEQ2 Scripting Guide
**Version:** 3.0
**Generated:** 2025-10-25

---

## Documentation Files Created

### Core Navigation
| File | Status | Description |
|------|--------|-------------|
| **README.md** | ✅ Complete | Main navigation hub, overview, and getting started guide |
| **02_Quick_Start_Guide.md** | ✅ Complete | 5-10 minute tutorial for beginners with working examples |
| **FILE_MANIFEST.md** | ✅ Complete | Complete file listing and project statistics |
| **+How To Use This Guide with Claude Code+.md** | ✅ Complete | Human-readable instructions for using the guide with Claude Code |

### Comprehensive Documentation
| File | Status | Description |
|------|--------|-------------|
| **00_MASTER_GUIDE.md** | ✅ Complete | Master reference with organized links to all APIs, commands, events |
| **01_LavishScript_Fundamentals.md** | ✅ Complete | Complete LavishScript programming guide (variables, functions, objects, loops, index, webrequest, audio) |
| **02_Quick_Start_Guide.md** | ✅ Complete | Quick start tutorial for beginners |
| **03_API_Reference.md** | ✅ Complete | Complete API documentation (all TLOs, datatypes, commands, 38 events) |
| **04_Core_Concepts.md** | ✅ Complete | Essential ISXEQ2 concepts (datatypes, queries, async, inheritance) |
| **05_Patterns_And_Best_Practices.md** | ✅ Complete | Coding patterns from [EQ2Bot](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Bot) analysis |
| **06_Working_Examples.md** | ✅ Complete | Real-world code examples for common tasks |
| **07_Advanced_Patterns_And_Examples.md** | ✅ Complete | Advanced patterns (mouse automation, XML config, broker, navigation) |
| **08_LavishGUI1_UI_Guide.md** | ✅ Complete | LavishGUI 1 UI creation guide (XML-based, legacy) |
| **09_Advanced_LGUI1_Patterns.md** | ✅ Complete | Real-world LGUI1 patterns from [MyPrices](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/MyPrices) script |
| **10_LavishGUI2_UI_Guide.md** | ✅ Complete | LavishGUI 2 UI creation guide (JSON-based, modern) with scalable title bars |
| **11_LavishGUI1_to_LavishGUI2_Migration.md** | ✅ Complete | Migration guide from LGUI1 to LGUI2 with scalable title bars and limitations |
| **12_LGUI2_Scaling_System.md** | ✅ Complete | Dynamic UI scaling system for LGUI2 interfaces with custom title bar patterns |
| **13_JSON_Guide.md** | ✅ Complete | Complete JSON guide (parsing, serialization, file I/O) |
| **14_LavishMachine_Guide.md** | ✅ Complete | Asynchronous task system (audio, web requests, custom tasks) |
| **15_Advanced_Scripting_Patterns.md** | ✅ Complete | Production-grade patterns from [EQ2OgreFree](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2OgreFree) and [EQ2Track](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Track) (14 patterns total) |
| **16_Utility_Script_Patterns.md** | ✅ Complete | Utility patterns from EQ2BJCommon (18 patterns: timers, tracking, validation, etc.) |
| **17_Crafting_Script_Patterns.md** | ✅ Complete | Crafting automation patterns from [EQ2Craft](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Craft) (13 patterns: cmd-line, navigation, localization, queue mgmt, writs, etc.) |
| **18_Navigation_Library_Patterns.md** | ✅ Complete | Navigation and pathfinding patterns from EQ2Nav (10 patterns: LavishNav, Dijkstra, regions, collision, stuck detection, doors, aggro, precision, optimization) |
| **19_DotNet_Development.md** | ✅ Complete | Scripts vs .NET decision-and-orientation guide — language/tooling/performance/debugging/deployment tradeoffs, ISXEQ2-specific .NET interop patterns (Extension helpers, domain wrappers, no `Execute(ExecuteCommand)` enum), `ISXEQ2Wrapper.dll` namespace map |

**Total Documentation:** 23 files (20 numbered guides + README + FILE_MANIFEST + How To Use Guide)

---

## Source Materials Analyzed

### Primary Sources

| Source | Purpose |
|--------|---------|
| **Official ISXEQ2 Reference** | Complete API reference |
| **ISXEQ2 Source Code** | Architecture analysis |
| **[EQ2Bot](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Bot) Main Script** | Pattern analysis |
| **[EQ2Bot](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Bot) Library** | Best practices |
| **[EQ2Bot](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Bot) Class Routines** | Implementation patterns |
| **[MyPrices](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/MyPrices) Script** | Advanced LGUI1 patterns |
| **[EQ2OgreFree](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2OgreFree) Scripts** | Advanced production patterns |
| **[EQ2Track](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Track) Script** | UI patterns, collections, file discovery |
| **[EQ2BJCommon](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2BJCommon) Scripts** | Utility patterns |
| **[EQ2Craft](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Craft) Script** | Crafting automation patterns |
| **[EQ2Navigation](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Navigation) Library** | Pathfinding and navigation |
| **Example Scripts** | Usage examples |
| **LavishGUI 1 Documentation** | Legacy UI system |
| **LavishGUI 2 Documentation** | Modern UI system |
| **[LERN/LGUI2](https://github.com/LavishSoftware/LERN/tree/master/LGUI2) Examples** | LGUI2 patterns |
| **[LERN/JSON](https://github.com/LavishSoftware/LERN/tree/master/JSON) Examples** | JSON usage |
| **[LERN/LMAC](https://github.com/LavishSoftware/LERN/tree/master/LMAC) Documentation** | Async tasks |
| **[LERN/LS](https://github.com/LavishSoftware/LERN/tree/master/LS) Tutorial Series** | LavishScript fundamentals |
| **[LERN/Lists](https://github.com/LavishSoftware/LERN/tree/master/Lists) Examples** | Index type usage |
| **[LERN/Web](https://github.com/LavishSoftware/LERN/tree/master/Web) Examples** | Web request patterns |
| **[LERN/Audio](https://github.com/LavishSoftware/LERN/tree/master/Audio) Examples** | Audio system usage |

### Source Code Analysis Results

#### ISXEQ2 Extension Architecture
- **Total Source Files:** 100+ C++ files
- **Datatypes Defined:** 60+ datatypes
- **Events Available:** 38 events
- **Commands Implemented:** 15+ commands
- **Top-Level Objects:** 21 TLOs

#### API Coverage Statistics
| Category | Count | Documentation Status |
|----------|-------|---------------------|
| **Core TLOs** | 21 | ✅ Documented in reference |
| **Datatypes** | 60+ | ✅ All documented |
| **Actor Datatypes** | 5 | ✅ Documented |
| **Item Datatypes** | 5 | ✅ Documented |
| **Ability Datatypes** | 2 | ✅ Documented |
| **Effect Datatypes** | 4 | ✅ Documented |
| **Quest Datatypes** | 2 | ✅ Documented |
| **Commerce Datatypes** | 3 | ✅ Documented |
| **Crafting Datatypes** | 5 | ✅ Documented |
| **Window Datatypes** | 14 | ✅ Documented |
| **Widget Datatypes** | 13 | ✅ Documented |
| **Commands** | 15 | ✅ Documented |
| **Events** | 48 | ✅ Documented |

---

## Documentation Statistics

### Content Breakdown

| Section | Content Type | Coverage |
|---------|--------------|----------|
| **Getting Started** | Tutorial | ✅ Complete |
| **API Reference** | Technical Reference | ✅ Complete |
| **Architecture** | Technical Deep-Dive | ✅ Complete |
| **Concepts** | Educational | ✅ Complete |
| **Best Practices** | Patterns | ✅ Complete |
| **Examples** | Code Samples | ✅ Complete |
| **Troubleshooting** | Problem-Solution | ✅ Complete |

### Code Examples Extracted

| Example Category | Count | Source |
|-----------------|-------|--------|
| **Basic Information Access** | 10+ | ISXEQ2 Reference |
| **Inventory Management** | 8+ | ISXEQ2 Reference + EQ2Bot |
| **Ability Casting** | 12+ | EQ2Bot Class Routines |
| **Event Handling** | 15+ | ISXEQ2 Reference + Example Scripts |
| **UI Interaction** | 10+ | ISXEQ2 Reference |
| **Quest Management** | 5+ | ISXEQ2 Reference |
| **Crafting** | 6+ | EQ2Bot + EQ2Craft |
| **Commerce/Broker** | 4+ | ISXEQ2 Reference |
| **Advanced Queries** | 8+ | ISXEQ2 Reference |
| **Movement/Navigation** | 3+ | EQ2Bot |

**Total Runnable Examples:** 80+ across all categories

### Patterns Identified (From EQ2Bot Analysis)

| Pattern Category | Patterns Found |
|-----------------|----------------|
| **Variable Declarations** | Script-scoped vs local scoping patterns |
| **Function Structures** | Template pattern for class compatibility |
| **Event Handling** | Chat triggers, atom handlers, event attachments |
| **Main Loop Patterns** | Multi-timer throttling, pulse architecture |
| **Naming Conventions** | PascalCase for bools, descriptive action names |
| **Code Organization** | File structure, section headers, function ordering |
| **Error Handling** | NULL checks, collection validation, timeout patterns |
| **Configuration** | UI-driven settings, XML imports, character-specific overrides |
| **Collections** | Population patterns, iterator usage |
| **Ability Casting** | CastSpellRange positional arguments, readiness checks |

---

## Key Findings from Analysis

### Architecture Insights

1. **Detour-Based Monitoring**: ISXEQ2 hooks game functions non-invasively to monitor state
2. **Event-Driven Design**: 38 events cover all major game activities
3. **Typed Object System**: Strong typing with inheritance hierarchies
4. **Async Data Loading**: Many detailed info objects require async loading
5. **Modular Organization**: Clear separation between datatypes, commands, events, and utilities

### Best Practices (From EQ2Bot)

1. **Multi-Timer Pulse System**: Different check frequencies (1s, 2s, 5s, 10s) for efficiency
2. **Template Pattern**: All class files implement identical function signatures
3. **Action Dispatch Arrays**: Priority-indexed arrays drive spell rotations
4. **Safety-First Coding**: Extensive existence checks prevent crashes
5. **Configuration-Driven**: Heavy UI integration for real-time tuning
6. **Code Reuse**: Shared library (EQ2BotLib.iss) prevents duplication

### Common Script Patterns

1. **Wait for ISXEQ2**: Always check `${ISXEQ2.IsReady}` before accessing API
2. **NULL Validation**: Check `(exists)` before accessing object members
3. **Async Waits**: Use timeouts when waiting for `IsItemInfoAvailable`, etc.
4. **Query Syntax**: Complex filtering with `==`, `!=`, `>`, `<`, `=-`, `=~` operators
5. **Event Atoms**: React to game events with attached atom handlers
6. **Iterator Pattern**: Standard way to loop through collections

---

## Documentation Completion Status

### ✅ Completed Documentation (All Core Files)

All essential documentation files have been completed:

1. **00_MASTER_GUIDE.md** - ✅ Complete navigation and quick reference
2. **01_LavishScript_Fundamentals.md** - ✅ Complete beginner's guide to LavishScript
3. **02_Quick_Start_Guide.md** - ✅ Complete quick start tutorial
4. **03_API_Reference.md** - ✅ Complete API documentation
5. **04_Core_Concepts.md** - ✅ Complete ISXEQ2 concepts guide
6. **05_Patterns_And_Best_Practices.md** - ✅ Complete patterns from EQ2Bot
7. **06_Working_Examples.md** - ✅ Complete working examples
8. **07_Advanced_Patterns_And_Examples.md** - ✅ Complete advanced patterns
9. **08_LavishGUI1_UI_Guide.md** - ✅ Complete legacy UI guide
10. **09_Advanced_LGUI1_Patterns.md** - ✅ Complete real-world LGUI1 patterns
11. **10_LavishGUI2_UI_Guide.md** - ✅ Complete modern UI guide
12. **11_LavishGUI1_to_LavishGUI2_Migration.md** - ✅ Complete migration guide with limitations
13. **12_LGUI2_Scaling_System.md** - ✅ Complete UI scaling system guide
14. **13_JSON_Guide.md** - ✅ Complete JSON guide
15. **14_LavishMachine_Guide.md** - ✅ Complete async task guide
16. **15_Advanced_Scripting_Patterns.md** - ✅ Complete production patterns guide
17. **16_Utility_Script_Patterns.md** - ✅ Complete utility patterns from EQ2BJCommon
18. **17_Crafting_Script_Patterns.md** - ✅ Complete crafting automation patterns from EQ2Craft
19. **18_Navigation_Library_Patterns.md** - ✅ Complete navigation and pathfinding patterns from EQ2Nav
20. **19_DotNet_Development.md** - ✅ Complete Scripts vs .NET decision guide with ISXEQ2 interop patterns

### Recent Additions (Version 2.0-3.1)

- **Version 3.1**: .NET Development guide (file 19) — Scripts vs .NET decision guide
- **Version 3.0**: Utility, Crafting, and Navigation pattern guides (files 16-18)
- **Version 2.9**: LGUI2 Scaling System guide
- **Version 2.5**: Advanced LGUI1 Patterns and LGUI2 limitations documentation
- **Version 2.4**: Advanced Scripting Patterns from EQ2OgreFree analysis
- **Version 2.3**: LavishScript Fundamentals with index, webrequest, and audio
- **Version 2.2**: LavishMachine (LMAC) async task guide
- **Version 2.1**: JSON guide with complete examples
- **Version 2.0**: LavishGUI 2 guide and migration guide

---

## Usage Recommendations

### For AI/LLM Use

This documentation set is specifically designed to serve as a knowledge base for AI assistants helping with ISXEQ2 script development:

1. **Use README.md** for navigation and determining which document to reference
2. **Use 01_LavishScript_Fundamentals.md** for LavishScript language basics
3. **Use 02_Quick_Start_Guide.md** when user needs ISXEQ2 introduction
4. **Use 03_API_Reference.md** for API lookups
5. **Use 04_Core_Concepts.md** for explaining ISXEQ2 fundamentals
6. **Use 06_Working_Examples.md** for code generation
7. **Use 13_JSON_Guide.md** for JSON operations
8. **Use 14_LavishMachine_Guide.md** for async tasks
9. **Use 15_Advanced_Scripting_Patterns.md** for production-grade patterns (multi-threading, navigation, modern APIs)

### For Human Developers

1. **Complete Beginners**: Start with [01_LavishScript_Fundamentals.md](01_LavishScript_Fundamentals.md) to learn the language
2. **ISXEQ2 Beginners**: Start with [02_Quick_Start_Guide.md](02_Quick_Start_Guide.md), then read [04_Core_Concepts.md](04_Core_Concepts.md)
3. **Intermediate**: Focus on [05_Patterns_And_Best_Practices.md](05_Patterns_And_Best_Practices.md), then [06_Working_Examples.md](06_Working_Examples.md)
4. **Advanced**: Study [07_Advanced_Patterns_And_Examples.md](07_Advanced_Patterns_And_Examples.md), [15_Advanced_Scripting_Patterns.md](15_Advanced_Scripting_Patterns.md), and specialized guides (JSON, LMAC, LGUI2)
5. **All Levels**: Bookmark [00_MASTER_GUIDE.md](00_MASTER_GUIDE.md) and [03_API_Reference.md](03_API_Reference.md) for quick lookups

---

## Quality Assurance

### Documentation Standards Met

- ✅ Table of contents with anchor links in all major files
- ✅ Clear headings and hierarchy
- ✅ Working code examples (tested syntax)
- ✅ Cross-references between sections
- ✅ File paths referenced where applicable
- ✅ Version information in footers

### Code Example Standards

- ✅ Complete and runnable examples
- ✅ Include necessary setup
- ✅ Show realistic use cases
- ✅ Include error handling
- ✅ Clear comments explaining logic
- ✅ Follow LavishScript conventions

### Accuracy

- ✅ All API references from official ISXEQ2Changes.txt
- ✅ Architecture details from actual source code
- ✅ Patterns from real production scripts (EQ2Bot)
- ✅ No invented/guessed API members
- ✅ All code examples use documented APIs only

---

## Success Criteria Status

| Criteria | Status |
|----------|--------|
| **Complete beginners can learn LavishScript** | ✅ Achieved (01_LavishScript_Fundamentals.md) |
| **ISXEQ2 beginners can start in 5-10 minutes** | ✅ Achieved (02_Quick_Start_Guide.md) |
| **Users can find any API quickly** | ✅ Achieved (00_MASTER_GUIDE.md + 03_API_Reference.md) |
| **All major features have examples** | ✅ Achieved (06_Working_Examples.md) |
| **Best practices are documented** | ✅ Achieved (05_Patterns_And_Best_Practices.md) |
| **Advanced patterns available** | ✅ Achieved (07_Advanced_Patterns_And_Examples.md) |
| **Modern UI development supported** | ✅ Achieved (10_LavishGUI2_UI_Guide.md) |
| **UI scaling system available** | ✅ Achieved (12_LGUI2_Scaling_System.md) |
| **JSON operations documented** | ✅ Achieved (13_JSON_Guide.md) |
| **Async tasks documented** | ✅ Achieved (14_LavishMachine_Guide.md) |
| **Production-grade patterns documented** | ✅ Achieved (15_Advanced_Scripting_Patterns.md) |
| **Utility script patterns documented** | ✅ Achieved (16_Utility_Script_Patterns.md) |
| **Crafting automation patterns documented** | ✅ Achieved (17_Crafting_Script_Patterns.md) |
| **Navigation and pathfinding documented** | ✅ Achieved (18_Navigation_Library_Patterns.md) |

---

## Documentation Structure

All documentation files are contained within this ISXEQ2 Scripting Guide directory:

- 22 total markdown files (19 numbered guides + README + FILE_MANIFEST + How To Use Guide)
- Covers beginner to advanced topics
- Includes LavishScript fundamentals through production-grade patterns
- Now includes LGUI2 scaling system with scalable title bars for dynamic UI resizing

---

## Version History

- **v3.1 (2026-04-12)**
  - Added [19_DotNet_Development.md](19_DotNet_Development.md) — Scripts vs .NET decision guide
  - Documents ISXEQ2-specific .NET interop (`LavishScript.Objects.GetObject`, `Extension` helpers, domain wrapper methods)
  - Explicitly contrasts with ISXEVE's `Execute(ExecuteCommand)` pattern (does NOT exist in `ISXEQ2Wrapper`)
  - Includes `ISXEQ2Wrapper.dll` namespace/assembly reference

- **v3.0 (2025-10-25)**
  - **MAJOR UPDATE:** Scalable Title Bars now standard for all LGUI2 scripts
  - Discovered correct `onCloseButtonClick` event for window close handling
  - Documented complete custom title bar pattern (EQ2BotCommander lines 1-88)
  - Updated all 4 guides with scalable title bar documentation:
    - 10_LavishGUI2_UI_Guide.md: Added custom titleBar to Window section
    - 11_LavishGUI1_to_LavishGUI2_Migration.md: Added scalable window pattern
    - 12_LGUI2_Scaling_System.md: Added "Creating Scalable Title Bars" section
    - EQ2Bot/UI/LGUI2_MIGRATION_GUIDE.md: Added section 1.2 as recommended default
  - Established 2x scaling baseline (font 40, padding 12, border 2) as standard
  - Corrected event documentation: Added 50+ events from DefaultSkin.json analysis

- **v2.9 (2025-10-25)**
  - Added LGUI2 Scaling System guide
  - Analyzed EQ2BotCommander LGUI1→LGUI2 migration with scaling implementation
  - Complete UI scaling patterns: dynamic JSON preprocessing, window position preservation, percentage-to-absolute conversion, button width adjustment (85%), font injection/scaling
  - Real-world example: EQ2BotCommander scaled 2x (800x800→1600x1600)
  - Updated 10_LavishGUI2_UI_Guide.md with scaling system references
  - Updated 11_LavishGUI1_to_LavishGUI2_Migration.md with scaling enhancement section
  - Renumbered files 13-18 to insert new guide after migration guide

- **v2.8 (2025-10-23)**
  - Added Navigation Library Patterns guide
  - Analyzed EQ2Navigation library: EQ2Nav_Lib.iss, EQ2NavMapper_Lib.iss, EQ2NavAggressionHandler.iss, EQ2NavObstacleHandler.iss, EQ2NavFaceClass_Lib.iss
  - 10 major navigation patterns: LavishNav integration, Dijkstra pathfinding algorithm, region-based navigation, collision detection, stuck detection and recovery, door automation, aggro detection integration, direct vs pathfinding movement, dual precision system, performance optimization
  - 3 complete working examples: SimpleNav (basic navigation), MultiStop (waypoint navigation), SafeNav (navigation with aggro handling)
  - Updated README navigation with 10 new navigation pattern links
  - Updated Advanced LGUI1 Patterns with 8 new patterns from EQ2Bot UI (main UI + 39 class UIs)
  - Updated Advanced LGUI1 Patterns with 14 new patterns from EQ2Craft UI

- **v2.7 (2025-10-23)**
  - Added Crafting Script Patterns guide
  - Analyzed EQ2Craft script (main + includes)
  - 13 crafting patterns: command-line argument parsing, navigation integration, localization/multi-language support, UI state management, recipe queue management, crafting window monitoring, event-driven crafting loops, device/station targeting, writ automation, durability/quality management, reaction arts system, progress tracking/statistics, file-based configuration
  - Complete working example: SimpleCraft automation script
  - Updated README navigation with 10 new crafting pattern links
  - Updated all navigation and learning paths

- **v2.6 (2025-10-23)**
  - Added Utility Script Patterns guide
  - Analyzed EQ2BJCommon scripts: bjauction, bjlooter, bjmagic, bjshuffle, bjxpbot
  - 18 utility patterns: custom timer objects, time calculations, currency tracking, position tracking, character-specific configs, dynamic file loading, prioritized item lists, event-based text parsing, EQ2DataSourceContainer, randomized movement, item verification, ApplyVerb, ReplyDialog, audio alerts, input validation, progressive XP tracking, script existence checking, ExecuteQueued loops
  - Complete working examples: auto-potion script, treasure chest hunter, multi-type XP tracker
  - Updated README navigation with 18 new utility pattern links
  - Updated all navigation and learning paths

- **v2.5 (2025-10-23)**
  - Added Advanced LGUI1 Patterns guide
  - Analyzed [MyPrices](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/MyPrices) broker script
  - Real-world patterns: FindChild navigation, alpha-based show/hide, Script:QueueCommand, settings integration, color-coded lists
  - Documented LGUI2 limitation: custom C++ element types not supported
  - Updated migration guide with Known Limitations section about radar migration failure
  - Renumbered files 09-14 to insert new guide after LGUI1 guide
  - Updated all navigation and learning paths

- **v2.4 (2025-10-21)**
  - Added Advanced Scripting Patterns guide
  - Analyzed 43 EQ2OgreFree script files and EQ2Track script
  - 14 patterns total: multi-threading, LavishSettings, LavishNav, timer objects, UI sync, modern EQ2:GetActors, triggers, controller pattern, dynamic variables, injectable UI, UI initialization guards, collection-based exclusion, file-based discovery, zone-aware auto-configuration
  - Replaced deprecated CustomActorArray with EQ2:GetActors throughout all guides
  - Production-grade patterns from real-world scripts
  - Updated all navigation and learning paths

- **v2.3 (2025-10-21)**
  - Added LavishScript Fundamentals guide
  - Analyzed 19 LERN/LS tutorial lessons (https://github.com/LavishSoftware/LERN/tree/master/LS)
  - Added Collections/Lists (index), Web Requests, and Audio sections
  - Prerequisite guide for complete beginners
  - Updated all navigation and learning paths

- **v2.2 (2025-10-21)**
  - Added LavishMachine (LMAC) guide
  - Analyzed 8 LERN/LMAC files (https://github.com/LavishSoftware/LERN/tree/master/LMAC)
  - Task managers, custom tasks, controller pattern
  - Audio and web request tasks
  - Complete async task examples

- **v2.1 (2025-10-21)**
  - Added JSON guide
  - Analyzed 12 LERN/JSON files (https://github.com/LavishSoftware/LERN/tree/master/JSON)
  - Serialization/deserialization
  - File I/O and configuration management

- **v2.0 (2025-10-21)**
  - Added LavishGUI 2 guide
  - Added LGUI1 to LGUI2 migration guide
  - Analyzed 60+ LERN/LGUI2 files (https://github.com/LavishSoftware/LERN/tree/master/LGUI2)
  - Modern JSON-based UI development

- **v1.0 (2024)**
  - Initial comprehensive documentation
  - Complete API reference
  - Core concepts and patterns
  - Working examples
  - LavishGUI 1 documentation
  - Advanced patterns

<!-- CLAUDE_SKIP_END -->
