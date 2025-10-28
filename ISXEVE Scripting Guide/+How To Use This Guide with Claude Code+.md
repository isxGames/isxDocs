<!-- CLAUDE_SKIP_START -->
# How to Use This Guide with Claude Code

This document explains how to reference the ISXEVE Scripting Guide when working with Claude Code for InnerSpace/ISXEVE script development. This document is NOT intended to be read by Claude Code for any sort of reference -- it's intended for human users only.

---

## Table of Contents

1. [Quick Reference (Simple Method)](#quick-reference-simple-method)
2. [Custom Command Setup (Advanced Method)](#custom-command-setup-advanced-method)
3. [Example Usage](#example-usage)
4. [Tips for Best Results](#tips-for-best-results)

---

## Quick Reference (Simple Method)

### Option 1: Direct Path Reference

Simply tell Claude Code to read the guide from wherever you've stored it:

```
I need help with ISXEVE scripting. Please read the documentation at:
<path-to-guide>/ISXEVE Scripting Guide

Then help me [describe your task].
```

**Example (replace the path with your actual location):**
```
I need help with ISXEVE scripting. Please read the documentation at:
C:\MyScripts\Documentation\ISXEVE Scripting Guide

Then help me create a script that automatically warps to anomalies and runs combat sites.
```

**Common Locations:**
- `C:\InnerSpace\Documentation\ISXEVE Scripting Guide`
- `D:\Games\InnerSpace\Docs\ISXEVE Scripting Guide`
- `C:\Users\YourName\Documents\ISXEVE Scripting Guide`

### Option 2: Reference Specific Guides

If you know which guide you need, reference it directly:

```
Please reference the ISXEVE guide at <path-to-guide>/ISXEVE Scripting Guide/[FILENAME].md
to help me with [your task].
```

**Common Files:**
- `00_MASTER_GUIDE.md` - Quick API reference lookup
- `02_Quick_Start_Guide.md` - Getting started
- `03_API_Reference.md` - Complete API documentation
- `06_Working_Examples.md` - Working code examples
- `15_Combat_Automation.md` - Combat bot patterns
- `16_Mining_And_Hauling.md` - Mining and hauling automation
- `17_Fleet_Operations.md` - Fleet coordination patterns
- `21_Advanced_Scripting_Patterns.md` - Production patterns

**Example (replace with your path):**
```
Please reference D:\InnerSpace\Docs\ISXEVE Scripting Guide\15_Combat_Automation.md
to help me create a ratting script.
```

### Option 3: Start from the Master Guide

For comprehensive help:

```
Please read the master guide at:
<path-to-guide>/ISXEVE Scripting Guide/00_MASTER_GUIDE.md

This contains links to all other documentation. Then help me [your task].
```

---

## Custom Command Setup (Advanced Method)

### What is a Custom Command?

A custom Claude Code command allows you to type `/isxeve` instead of providing the full path each time. This is faster and more convenient for frequent ISXEVE development.

### Setup Instructions

**Step 1: Locate Your Working Directory**

First, determine where you want to create the custom command. This should be a directory where you frequently work with InnerSpace scripts. For example:
- `C:\InnerSpace\Scripts\` (if you work from the InnerSpace Scripts folder)
- `C:\Dev\InnerSpace\` (if you have a dedicated development folder)
- `D:\MyProjects\EVEScripts\` (any custom location)

**Step 2: Create the Commands Directory**

In your chosen working directory, create the `.claude` subdirectory structure:

```
<your-working-directory>\
└── .claude\
    └── commands\
        └── isxeve.md
```

**Examples:**
```
C:\InnerSpace\Scripts\.claude\commands\isxeve.md
D:\Dev\EVE\.claude\commands\isxeve.md
```

**Step 3: Create the Command File**

Create the `isxeve.md` file in the commands directory with this content (**update the paths to match your system**):

```markdown
You are an expert ISXEVE script developer with deep knowledge of LavishScript, InnerSpace, and the ISXEVE extension for EVE Online.

## Knowledge Base

**PRIMARY REFERENCE - Read these files as needed:**
- **Comprehensive Guide:** `<YOUR_PATH>\ISXEVE Scripting Guide\README.md` (start here for navigation - Version 2.0)
- **LavishScript Fundamentals:** `<YOUR_PATH>\ISXEVE Scripting Guide\01_LavishScript_Fundamentals.md`
- **Quick Start Guide:** `<YOUR_PATH>\ISXEVE Scripting Guide\02_Quick_Start_Guide.md`
- **API Reference:** `<YOUR_PATH>\ISXEVE Scripting Guide\03_API_Reference.md`
- **Core Concepts:** `<YOUR_PATH>\ISXEVE Scripting Guide\04_Core_Concepts.md`
- **Best Practices:** `<YOUR_PATH>\ISXEVE Scripting Guide\05_Patterns_And_Best_Practices.md`
- **Working Examples:** `<YOUR_PATH>\ISXEVE Scripting Guide\06_Working_Examples.md`
- **Advanced Patterns:** `<YOUR_PATH>\ISXEVE Scripting Guide\07_Advanced_Patterns_And_Examples.md`
- **LGUI2 UI Guide:** `<YOUR_PATH>\ISXEVE Scripting Guide\10_LavishGUI2_UI_Guide.md` (modern, JSON-based - recommended)
- **LGUI1 to LGUI2 Migration:** `<YOUR_PATH>\ISXEVE Scripting Guide\11_LavishGUI1_to_LavishGUI2_Migration.md`
- **LGUI2 Scaling System:** `<YOUR_PATH>\ISXEVE Scripting Guide\12_LGUI2_Scaling_System.md`
- **Combat Automation:** `<YOUR_PATH>\ISXEVE Scripting Guide\15_Combat_Automation.md`
- **Mining and Hauling:** `<YOUR_PATH>\ISXEVE Scripting Guide\16_Mining_And_Hauling.md`
- **Fleet Operations:** `<YOUR_PATH>\ISXEVE Scripting Guide\17_Fleet_Operations.md`
- **Bot Architecture Analysis:** `<YOUR_PATH>\ISXEVE Scripting Guide\18_Bot_Architecture_Analysis.md`
- **Production Patterns:** `<YOUR_PATH>\ISXEVE Scripting Guide\21_Advanced_Scripting_Patterns.md`

## Core Responsibilities

### 1. Script Creation
- Write complete, working ISXEVE scripts using correct API syntax
- Follow established patterns from EVEBot, Yamfa, Tehbot, and other production scripts
- Include proper NULL checks, error handling, and async data loading
- Use appropriate naming conventions (PascalCase for bools, descriptive names)

### 2. Debugging
- Identify common ISXEVE errors (NULL references, async data issues, query syntax)
- Check for proper `${ISXEVE.IsReady}` initialization
- Verify existence checks before accessing object members
- Validate query syntax and collection handling

### 3. Code Quality
- Apply multi-timer pulse patterns (1s, 2s, 5s, 10s) for efficiency
- Use script-scoped variables appropriately
- Implement proper event handling with atoms
- Follow template pattern for maintainability

### 4. API Usage
- Use correct TLOs: `${Me}`, `${MyShip}`, `${EVE}`, `${Local}`, `${Station}`, `${Entity[...]}`, `${Agent[...]}`, `${Bookmark[...]}`
- Apply proper datatype inheritance and object relationships
- Handle entity management: targeting, locking, aggro detection
- Use module control: activation, cycling, cooldowns, capacitor management
- Manage inventory: cargo, hangar, item manipulation
- Use movement commands: align, warp, dock, undock, approach
- Handle fleet operations: fleet members, fleet warps, coordination

## Critical Rules

**ALWAYS:**
- Check `${ISXEVE.IsReady}` before accessing API for the first time
- Validate object existence with `(exists)` before accessing members
- Wait for async data when needed (entity data, market info, etc.)
- Use relative paths, never absolute paths
- Include proper error handling and timeouts
- Reference the comprehensive guide when uncertain
- Use LavishGUI 2 (JSON) for new UIs, not LavishGUI 1 (XML)

**NEVER:**
- Access object members without NULL checks
- Assume data is immediately available (check async loading)
- Use absolute file paths (use `${LavishScript.HomeDirectory}` or relative paths)
- Guess API syntax (refer to guide first)
- Create inefficient loops without throttling
- Use LavishGUI 1 (XML) for new projects (use LGUI2 JSON instead)

## Workflow

1. **Understand the task** - Ask clarifying questions if needed
2. **Reference the guide** - Read relevant sections from the comprehensive guide
3. **Analyze existing code** - If debugging/refactoring, understand current implementation
4. **Apply patterns** - Use established EVEBot/Yamfa/Tehbot patterns and best practices
5. **Verify correctness** - Ensure proper API usage, NULL checks, and error handling
6. **Test considerations** - Suggest testing approach and edge cases

## Code Style

Follow EVEBot conventions:
- Script-scoped variables for persistent state
- Local variables for temporary operations
- Multi-timer pulse architecture for performance
- Clear, descriptive function and variable names
- Comments for complex logic
- Section headers for organization

## Production Patterns (from Guide v2.0)

The guide includes production-grade patterns from real scripts ([EVEBot](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable), [Yamfa](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/Yamfa), [Tehbot](https://github.com/isxGames/Tehbot), [Metatron2](https://github.com/spacekoala420/Metatron2)):
- Multi-Threading - Worker threads with cross-script communication
- LavishSettings - Hierarchical XML configuration
- Timer Objects - Reusable timing with pulse architecture
- State Machines - Robust bot control flow
- Relay System - Multi-boxing coordination and IPC
- Combat Management - Target selection, weapon control, tank management
- Mining Operations - Ore scanning, strip miner control, hauler coordination
- Fleet Coordination - Fleet assist, synchronized warps, loot distribution
- Module Management - Cycle tracking, capacitor optimization, cooldown handling
- Inventory Management - Cargo optimization, item queries, container handling
- Movement & Navigation - Warp patterns, safespots, bookmark systems
- LGUI2 Dynamic Scaling - User-configurable UI sizing
- LGUI1 to LGUI2 Migration - Checkbox persistence, MessageBox replacement, event handling

Your goal is to help users create robust, efficient, maintainable ISXEVE scripts using proven patterns and best practices.

---

**Now help the user with their ISXEVE scripting task.**
```

**Step 3: Verify the Command**

In Claude Code, type `/` and you should see `isxeve` in the list of available commands.

### Using the Custom Command

Once set up, simply type:

```
/isxeve
```

Then describe what you need help with. Claude will automatically load the documentation and assist you.

**Example:**
```
/isxeve

I need to create a script that:
1. Warps to asteroid belts
2. Targets and mines asteroids
3. Returns to station when cargo is full
```

---

## Example Usage

### Example 1: Creating a New Script

**Without Custom Command:**
```
I need help with ISXEVE scripting. Please read:
<path-to-guide>\ISXEVE Scripting Guide

Then help me create a script that automatically runs combat anomalies.
```
(Replace `<path-to-guide>` with your actual path, e.g., `C:\InnerSpace\Documentation`)

**With Custom Command:**
```
/isxeve

Help me create a script that automatically runs combat anomalies.
```

### Example 2: Debugging Existing Code

**Without Custom Command:**
```
Please reference <path-to-guide>\ISXEVE Scripting Guide\03_API_Reference.md

My script crashes when I try to access ${Entity[TypeID,12345].Name}. Why?
```
(Replace `<path-to-guide>` with your actual path)

**With Custom Command:**
```
/isxeve

My script crashes when I try to access ${Entity[TypeID,12345].Name}. Why?
```

### Example 3: Learning a Specific Topic

**Without Custom Command:**
```
Please read <path-to-guide>\ISXEVE Scripting Guide\17_Fleet_Operations.md

Teach me how to use the relay system for multi-boxing coordination.
```
(Replace `<path-to-guide>` with your actual path)

**With Custom Command:**
```
/isxeve

Teach me how to use the relay system for multi-boxing coordination.
```

### Example 4: Getting Started as a Beginner

**Without Custom Command:**
```
I'm new to ISXEVE scripting. Please read:
<path-to-guide>\ISXEVE Scripting Guide\01_LavishScript_Fundamentals.md
and
<path-to-guide>\ISXEVE Scripting Guide\02_Quick_Start_Guide.md

Then guide me through creating my first script.
```
(Replace `<path-to-guide>` with your actual path)

**With Custom Command:**
```
/isxeve

I'm a complete beginner. Guide me through creating my first ISXEVE script.
```

---

## Tips for Best Results

### Be Specific About Your Needs

**Good:**
```
/isxeve

I need a script that monitors nearby belts for ore anomalies, warps to them,
and mines the most valuable ore first. I want to use modern patterns.
```

**Less Effective:**
```
/isxeve

Help with mining.
```

### Mention Your Experience Level

This helps Claude choose the right documentation:

```
/isxeve

I'm familiar with LavishScript but new to ISXEVE. I need help understanding
how to scan for entities using the Entity TLO and target selection.
```

### Reference Existing Code When Debugging

```
/isxeve

Here's my code:
[paste code]

It's not working as expected. Can you help debug it?
```

### Ask for Specific Documentation Sections

```
/isxeve

I'm trying to understand the difference between MyShip:Approach and
EVE:Execute[CmdApproach]. Please reference the API guide and explain with examples.
```

### Request Modern Patterns

```
/isxeve

Show me the modern way to manage combat targets using state machines
like Tehbot does.
```

### Ask for Complete Examples

```
/isxeve

Give me a complete working example of a mining script that uses events to
react when asteroids are depleted, with proper error handling.
```

---

## Common Tasks and Recommended Guides

| Task | Recommended Guide(s) |
|------|---------------------|
| **First time using ISXEVE** | 01_LavishScript_Fundamentals.md, 02_Quick_Start_Guide.md |
| **API reference lookup** | 00_MASTER_GUIDE.md, 03_API_Reference.md |
| **Combat automation** | 15_Combat_Automation.md, 05_Patterns_And_Best_Practices.md |
| **Mining automation** | 16_Mining_And_Hauling.md, 06_Working_Examples.md |
| **Fleet coordination** | 17_Fleet_Operations.md, 07_Advanced_Patterns_And_Examples.md |
| **Creating a UI** | 10_LavishGUI2_UI_Guide.md (modern) or 08_LavishGUI1_UI_Guide.md (legacy) |
| **Multi-boxing setup** | 17_Fleet_Operations.md, 07_Advanced_Patterns_And_Examples.md |
| **Understanding bot architecture** | 18_Bot_Architecture_Analysis.md |
| **Advanced patterns** | 21_Advanced_Scripting_Patterns.md, 22_Utility_Script_Patterns.md |
| **JSON usage** | 13_JSON_Guide.md |
| **Async tasks** | 14_LavishMachine_Guide.md |
| **.NET development** | 19_DotNet_Development.md |
| **Debugging** | 20_Debugging_And_Troubleshooting.md |

---

## Troubleshooting

### Command Not Appearing

If `/isxeve` doesn't appear in the command list:

1. Verify the file exists at: `<your-working-directory>\.claude\commands\isxeve.md`
2. Restart Claude Code
3. Make sure the file has the proper frontmatter (the `---` section at the top)
4. Check that the `.claude` directory is in your project root

### Claude Doesn't Load Documentation

If Claude doesn't seem to be using the guide:

1. Be explicit in your request: "Please read the ISXEVE Scripting Guide first"
2. Reference specific files when possible
3. Use the `/isxeve` command if you've set it up
4. Verify the path in your command file is correct (check the path you specified in the isxeve.md file)

### Getting Better Responses

- Provide context about what you're trying to accomplish
- Share relevant code snippets
- Mention if you're getting specific error messages
- Indicate your experience level
- Ask for explanations of concepts you don't understand

---

## Quick Start Template

Copy and paste this template to get started:

### Without Custom Command:
```
I need help with ISXEVE scripting. Please read the documentation at:
<path-to-guide>\ISXEVE Scripting Guide

My experience level: [beginner/intermediate/advanced]

Task: [Describe what you want to do]

Current code (if any): [Paste code or write "starting from scratch"]

Specific questions: [List any specific questions]
```
(Replace `<path-to-guide>` with your actual path, e.g., `C:\InnerSpace\Documentation`)

### With Custom Command:
```
/isxeve

My experience level: [beginner/intermediate/advanced]

Task: [Describe what you want to do]

Current code (if any): [Paste code or write "starting from scratch"]

Specific questions: [List any specific questions]
```

---

## Additional Resources

### Within the Guide

- **README.md** - Complete overview with navigation by task
- **FILE_MANIFEST.md** - Complete file listing and version history
- **00_MASTER_GUIDE.md** - Quick reference organized by category

### File Locations

These are examples - your paths will vary based on where you installed InnerSpace and where you stored the guide:

- **Guide Location:** `<path-to-guide>\ISXEVE Scripting Guide\`
  - Example: `C:\InnerSpace\Documentation\ISXEVE Scripting Guide\`
  - Example: `D:\Games\InnerSpace\Docs\ISXEVE Scripting Guide\`
- **Command File (if using):** `<your-working-directory>\.claude\commands\isxeve.md`
  - Example: `C:\InnerSpace\Scripts\.claude\commands\isxeve.md`
  - Example: `D:\Dev\EVE\.claude\commands\isxeve.md`
- **InnerSpace Scripts:** Typically found in your InnerSpace installation
  - Example: `C:\InnerSpace\Scripts\`
  - Example: `D:\Games\InnerSpace\Scripts\`

---

**Last Updated:** 2025-10-27

---

*This guide is designed to help you make the most of the ISXEVE Scripting Guide when working with Claude Code. Whether you use the simple path reference method or set up the custom command, you'll have access to comprehensive documentation covering all aspects of ISXEVE script development.*
<!-- CLAUDE_SKIP_END -->
