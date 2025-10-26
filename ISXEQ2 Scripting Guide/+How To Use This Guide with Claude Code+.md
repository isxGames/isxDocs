# How to Use This Guide with Claude Code

This document explains how to reference the ISXEQ2 Scripting Guide when working with Claude Code for InnerSpace/ISXEQ2 script development.   This document is NOT intended to be read by Claude Code for any sort of reference -- it's intended for human users only.

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
I need help with ISXEQ2 scripting. Please read the documentation at:
<path-to-guide>/ISXEQ2 Scripting Guide

Then help me [describe your task].
```

**Example (replace the path with your actual location):**
```
I need help with ISXEQ2 scripting. Please read the documentation at:
C:\MyScripts\Documentation\ISXEQ2 Scripting Guide

Then help me create a script that casts healing spells when health drops below 50%.
```

**Common Locations:**
- `C:\InnerSpace\Documentation\ISXEQ2 Scripting Guide`
- `D:\Games\InnerSpace\Docs\ISXEQ2 Scripting Guide`
- `C:\Users\YourName\Documents\ISXEQ2 Scripting Guide`

### Option 2: Reference Specific Guides

If you know which guide you need, reference it directly:

```
Please reference the ISXEQ2 guide at <path-to-guide>/ISXEQ2 Scripting Guide/[FILENAME].md
to help me with [your task].
```

**Common Files:**
- `00_MASTER_GUIDE.md` - Quick API reference lookup
- `02_Quick_Start_Guide.md` - Getting started
- `03_API_Reference.md` - Complete API documentation
- `06_Working_Examples.md` - Working code examples
- `14_Advanced_Scripting_Patterns.md` - Production patterns
- `17_Navigation_Library_Patterns.md` - Navigation/pathfinding

**Example (replace with your path):**
```
Please reference D:\InnerSpace\Docs\ISXEQ2 Scripting Guide\06_Working_Examples.md
to help me create a combat rotation script.
```

### Option 3: Start from the Master Guide

For comprehensive help:

```
Please read the master guide at:
<path-to-guide>/ISXEQ2 Scripting Guide/00_MASTER_GUIDE.md

This contains links to all other documentation. Then help me [your task].
```

---

## Custom Command Setup (Advanced Method)

### What is a Custom Command?

A custom Claude Code command allows you to type `/isxeq2` instead of providing the full path each time. This is faster and more convenient for frequent ISXEQ2 development.

### Setup Instructions

**Step 1: Locate Your Working Directory**

First, determine where you want to create the custom command. This should be a directory where you frequently work with InnerSpace scripts. For example:
- `C:\InnerSpace\Scripts\` (if you work from the InnerSpace Scripts folder)
- `C:\Dev\InnerSpace\` (if you have a dedicated development folder)
- `D:\MyProjects\EQ2Scripts\` (any custom location)

**Step 2: Create the Commands Directory**

In your chosen working directory, create the `.claude` subdirectory structure:

```
<your-working-directory>\
└── .claude\
    └── commands\
        └── isxeq2.md
```

**Examples:**
```
C:\InnerSpace\Scripts\.claude\commands\isxeq2.md
D:\Dev\EQ2\.claude\commands\isxeq2.md
```

**Step 3: Create the Command File**

Create the `isxeq2.md` file in the commands directory with this content (be sure to update the path in the template below):

```markdown
---
description: Load ISXEQ2 scripting knowledge from comprehensive guide
---

You are an expert InnerSpace/ISXEQ2 script developer with access to comprehensive documentation.

# Knowledge Base Location

The complete ISXEQ2 Scripting Guide is located at:
**<REPLACE_WITH_YOUR_PATH>\ISXEQ2 Scripting Guide**

(Example: C:\InnerSpace\Documentation\ISXEQ2 Scripting Guide)

# Your Role

You help users create, debug, and optimize scripts for InnerSpace using the ISXEQ2 extension for EverQuest 2.

# Available Documentation (20 Guides)

## Getting Started
- **00_MASTER_GUIDE.md** - Master reference hub with all APIs organized by category
- **01_LavishScript_Fundamentals.md** - Complete LavishScript programming guide
- **02_Quick_Start_Guide.md** - Quick start tutorial

## Core Documentation
- **03_API_Reference.md** - Complete API reference (all TLOs, datatypes, commands, events)
- **04_Core_Concepts.md** - Essential concepts (datatypes, queries, async, inheritance)

## Practical Guides
- **05_Patterns_And_Best_Practices.md** - Coding patterns and best practices
- **06_Working_Examples.md** - Real-world runnable examples
- **07_Advanced_Patterns_And_Examples.md** - Advanced patterns (mouse, XML, broker, navigation)

## UI Development
- **08_LavishGUI1_UI_Guide.md** - LavishGUI 1 (XML-based, legacy)
- **09_Advanced_LGUI1_Patterns.md** - Advanced LGUI1 patterns from production scripts
- **10_LavishGUI2_UI_Guide.md** - LavishGUI 2 (JSON-based, modern - recommended)
- **11_LavishGUI1_to_LavishGUI2_Migration.md** - Migration guide with common issues and solutions
- **12_LGUI2_Scaling_System.md** - Dynamic UI scaling system for LGUI2

## Advanced Topics
- **13_JSON_Guide.md** - JSON parsing, serialization, file I/O
- **14_LavishMachine_Guide.md** - Asynchronous tasks (audio, web requests)
- **15_Advanced_Scripting_Patterns.md** - Production patterns (multi-threading, LavishSettings, LavishNav)
- **16_Utility_Script_Patterns.md** - Utility patterns (timers, tracking, validation)
- **17_Crafting_Script_Patterns.md** - Crafting automation patterns
- **18_Navigation_Library_Patterns.md** - Navigation and pathfinding patterns
- **README.md** - Complete guide overview with quick navigation

# Instructions

When a user invokes `/isxeq2`, you should:

1. **Acknowledge** that you have access to the ISXEQ2 Scripting Guide
2. **Read the appropriate documentation** based on the user's needs:
   - For beginners: Start with `00_MASTER_GUIDE.md` or `02_Quick_Start_Guide.md`
   - For API lookups: Use `03_API_Reference.md`
   - For specific tasks: Reference relevant guides (navigation, crafting, UI, etc.)
   - For patterns: Use the advanced pattern guides
3. **Provide accurate help** based on the official documentation
4. **Reference specific files** when providing code examples or explanations

# Key Principles

- Always use modern patterns (e.g., `EQ2:GetActors` NOT deprecated `CustomActorArray`)
- Check for NULL with `(exists)` before accessing object members
- Wait for `${ISXEQ2.IsReady}` before using the API
- Use LavishGUI 2 (JSON) for new UIs, not LavishGUI 1 (XML)
- Follow best practices from the guides (proper error handling, timeouts, etc.)

# Response Format

After reading the relevant documentation, help the user with:
- Clear explanations
- Working code examples
- References to specific guide sections (e.g., "See 06_Working_Examples.md:123")
- Best practices and common pitfalls

# If User Needs Are Unclear

Ask clarifying questions:
- Are you creating a new script or debugging existing code?
- What specific task are you trying to accomplish?
- Are you familiar with LavishScript basics?
- Do you need UI development help?

# Available Documentation Coverage

The guide covers:
- ✅ Complete API reference (60+ datatypes, 21 TLOs, 48 events)
- ✅ LavishScript fundamentals
- ✅ Working examples for all common tasks
- ✅ Advanced patterns from production scripts (EQ2Bot, EQ2Craft, EQ2Nav, EQ2AFKAlarm)
- ✅ UI development (both LGUI1 and LGUI2)
- ✅ LGUI1 to LGUI2 migration guide with 12+ common issues and solutions
- ✅ Dynamic UI scaling system for LGUI2
- ✅ Navigation and pathfinding
- ✅ Crafting automation
- ✅ JSON and async tasks
- ✅ 35,248 lines of documentation across 20 guides
```

**Step 3: Verify the Command**

In Claude Code, type `/` and you should see `isxeq2` in the list of available commands.

### Using the Custom Command

Once set up, simply type:

```
/isxeq2
```

Then describe what you need help with. Claude will automatically load the documentation and assist you.

**Example:**
```
/isxeq2

I need to create a script that:
1. Monitors my health
2. Casts a heal spell when below 50%
3. Uses a potion if the spell is on cooldown
```

---

## Example Usage

### Example 1: Creating a New Script

**Without Custom Command:**
```
I need help with ISXEQ2 scripting. Please read:
<path-to-guide>\ISXEQ2 Scripting Guide

Then help me create a script that automatically loots nearby chests.
```
(Replace `<path-to-guide>` with your actual path, e.g., `C:\InnerSpace\Documentation`)

**With Custom Command:**
```
/isxeq2

Help me create a script that automatically loots nearby chests.
```

### Example 2: Debugging Existing Code

**Without Custom Command:**
```
Please reference <path-to-guide>\ISXEQ2 Scripting Guide\03_API_Reference.md

My script crashes when I try to access ${Actor[chest].Name}. Why?
```
(Replace `<path-to-guide>` with your actual path)

**With Custom Command:**
```
/isxeq2

My script crashes when I try to access ${Actor[chest].Name}. Why?
```

### Example 3: Learning a Specific Topic

**Without Custom Command:**
```
Please read <path-to-guide>\ISXEQ2 Scripting Guide\17_Navigation_Library_Patterns.md

Teach me how to use the EQ2Nav library for pathfinding.
```
(Replace `<path-to-guide>` with your actual path)

**With Custom Command:**
```
/isxeq2

Teach me how to use the EQ2Nav library for pathfinding.
```

### Example 4: Getting Started as a Beginner

**Without Custom Command:**
```
I'm new to ISXEQ2 scripting. Please read:
<path-to-guide>\ISXEQ2 Scripting Guide\01_LavishScript_Fundamentals.md
and
<path-to-guide>\ISXEQ2 Scripting Guide\02_Quick_Start_Guide.md

Then guide me through creating my first script.
```
(Replace `<path-to-guide>` with your actual path)

**With Custom Command:**
```
/isxeq2

I'm a complete beginner. Guide me through creating my first ISXEQ2 script.
```

---

## Tips for Best Results

### Be Specific About Your Needs

**Good:**
```
/isxeq2

I need a script that monitors my target's health and automatically
assists when it drops below 20%. I want to use modern patterns.
```

**Less Effective:**
```
/isxeq2

Help with combat.
```

### Mention Your Experience Level

This helps Claude choose the right documentation:

```
/isxeq2

I'm familiar with LavishScript but new to ISXEQ2. I need help understanding
how to query for nearby NPCs using the Actor TLO.
```

### Reference Existing Code When Debugging

```
/isxeq2

Here's my code:
[paste code]

It's not working as expected. Can you help debug it?
```

### Ask for Specific Documentation Sections

```
/isxeq2

I'm trying to understand the difference between lnavpath and lnavregionref.
Please reference the navigation guide and explain with examples.
```

### Request Modern Patterns

```
/isxeq2

Show me the modern way to search for actors. I know CustomActorArray
is deprecated.
```

### Ask for Complete Examples

```
/isxeq2

Give me a complete working example of a script that uses events to
react when I enter combat, with proper error handling.
```

---

## Common Tasks and Recommended Guides

| Task | Recommended Guide(s) |
|------|---------------------|
| **First time using ISXEQ2** | 01_LavishScript_Fundamentals.md, 02_Quick_Start_Guide.md |
| **API reference lookup** | 00_MASTER_GUIDE.md, 03_API_Reference.md |
| **Combat automation** | 06_Working_Examples.md, 05_Patterns_And_Best_Practices.md |
| **Inventory management** | 06_Working_Examples.md, 03_API_Reference.md |
| **Creating a UI** | 10_LavishGUI2_UI_Guide.md (modern) or 08_LavishGUI1_UI_Guide.md (legacy) |
| **Navigation/Pathfinding** | 17_Navigation_Library_Patterns.md |
| **Crafting automation** | 16_Crafting_Script_Patterns.md |
| **Understanding events** | 04_Core_Concepts.md, 06_Working_Examples.md |
| **Advanced patterns** | 14_Advanced_Scripting_Patterns.md, 15_Utility_Script_Patterns.md |
| **JSON usage** | 12_JSON_Guide.md |
| **Async tasks** | 13_LavishMachine_Guide.md |

---

## Troubleshooting

### Command Not Appearing

If `/isxeq2` doesn't appear in the command list:

1. Verify the file exists at: `<your-working-directory>\.claude\commands\isxeq2.md`
2. Restart Claude Code
3. Make sure the file has the proper frontmatter (the `---` section at the top)
4. Check that the `.claude` directory is in your project root

### Claude Doesn't Load Documentation

If Claude doesn't seem to be using the guide:

1. Be explicit in your request: "Please read the ISXEQ2 Scripting Guide first"
2. Reference specific files when possible
3. Use the `/isxeq2` command if you've set it up
4. Verify the path in your command file is correct (check the path you specified in the isxeq2.md file)

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
I need help with ISXEQ2 scripting. Please read the documentation at:
<path-to-guide>\ISXEQ2 Scripting Guide

My experience level: [beginner/intermediate/advanced]

Task: [Describe what you want to do]

Current code (if any): [Paste code or write "starting from scratch"]

Specific questions: [List any specific questions]
```
(Replace `<path-to-guide>` with your actual path, e.g., `C:\InnerSpace\Documentation`)

### With Custom Command:
```
/isxeq2

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

- **Guide Location:** `<path-to-guide>\ISXEQ2 Scripting Guide\`
  - Example: `C:\InnerSpace\Documentation\ISXEQ2 Scripting Guide\`
  - Example: `D:\Games\InnerSpace\Docs\ISXEQ2 Scripting Guide\`
- **Command File (if using):** `<your-working-directory>\.claude\commands\isxeq2.md`
  - Example: `C:\InnerSpace\Scripts\.claude\commands\isxeq2.md`
  - Example: `D:\Dev\EQ2\.claude\commands\isxeq2.md`
- **InnerSpace Scripts:** Typically found in your InnerSpace installation
  - Example: `C:\InnerSpace\Scripts\`
  - Example: `D:\Games\InnerSpace\Scripts\`

### Documentation Statistics

- **Version:** 3.0
- **Total Guides:** 20 comprehensive guides
- **Total Documentation:** 35,248 lines
- **Coverage:** Complete ISXEQ2 API, LavishScript fundamentals, UI development (LGUI1 & LGUI2), advanced patterns
- **Scripts Analyzed:** EQ2Bot, EQ2Craft, EQ2Nav, MyPrices, EQ2BJCommon, EQ2OgreFree, EQ2Track, EQ2AFKAlarm

---

## Version Information

**Document Version:** 1.1
**ISXEQ2 Guide Version:** 3.0
**Last Updated:** 2025-10-25

---

*This guide is designed to help you make the most of the ISXEQ2 Scripting Guide when working with Claude Code. Whether you use the simple path reference method or set up the custom command, you'll have access to comprehensive documentation covering all aspects of ISXEQ2 script development.*
