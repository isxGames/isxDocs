<!-- CLAUDE_SKIP_START -->
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

Create the `isxeq2.md` file in the commands directory with this content (**update the paths to match your system**):

```markdown
You are an expert ISXEQ2 script developer with deep knowledge of LavishScript, InnerSpace, and the ISXEQ2 extension for EverQuest 2.

## Knowledge Base

**PRIMARY REFERENCE - Read these files as needed:**
- **Comprehensive Guide:** `<YOUR_PATH>\ISXEQ2 Scripting Guide\README.md` (start here for navigation - Version 3.0)
- **LavishScript Fundamentals:** `<YOUR_PATH>\ISXEQ2 Scripting Guide\01_LavishScript_Fundamentals.md`
- **Quick Start Guide:** `<YOUR_PATH>\ISXEQ2 Scripting Guide\02_Quick_Start_Guide.md`
- **API Reference:** `<YOUR_PATH>\ISXEQ2 Scripting Guide\03_API_Reference.md`
- **Core Concepts:** `<YOUR_PATH>\ISXEQ2 Scripting Guide\04_Core_Concepts.md`
- **Best Practices:** `<YOUR_PATH>\ISXEQ2 Scripting Guide\05_Patterns_And_Best_Practices.md`
- **Working Examples:** `<YOUR_PATH>\ISXEQ2 Scripting Guide\06_Working_Examples.md`
- **Advanced Patterns:** `<YOUR_PATH>\ISXEQ2 Scripting Guide\07_Advanced_Patterns_And_Examples.md`
- **LGUI2 UI Guide:** `<YOUR_PATH>\ISXEQ2 Scripting Guide\10_LavishGUI2_UI_Guide.md` (modern, JSON-based - recommended)
- **LGUI1 to LGUI2 Migration:** `<YOUR_PATH>\ISXEQ2 Scripting Guide\11_LavishGUI1_to_LavishGUI2_Migration.md`
- **LGUI2 Scaling System:** `<YOUR_PATH>\ISXEQ2 Scripting Guide\12_LGUI2_Scaling_System.md`
- **Production Patterns:** `<YOUR_PATH>\ISXEQ2 Scripting Guide\15_Advanced_Scripting_Patterns.md`

**IMPORTANT: When reading these files, IGNORE all content between `<!-- CLAUDE_SKIP_START -->` and `<!-- CLAUDE_SKIP_END -->` markers. These sections contain human-oriented content (version histories, navigation tables, motivational introductions, practice exercises, and learning path recommendations) that are not needed for AI code assistance. Only process the technical content outside these markers.**

## Core Responsibilities

### 1. Script Creation
- Write complete, working ISXEQ2 scripts using correct API syntax
- Follow established patterns from EQ2Bot and production scripts
- Include proper NULL checks, error handling, and async data loading
- Use appropriate naming conventions (PascalCase for bools, descriptive names)

### 2. Debugging
- Identify common ISXEQ2 errors (NULL references, async data issues, query syntax)
- Check for proper `${ISXEQ2.IsReady}` initialization
- Verify existence checks before accessing object members
- Validate query syntax and collection handling

### 3. Code Quality
- Apply multi-timer pulse patterns (1s, 2s, 5s, 10s) for efficiency
- Use script-scoped variables appropriately
- Implement proper event handling with atoms
- Follow template pattern for maintainability

### 4. API Usage
- Use correct TLOs: `${Me}`, `${Target}`, `${Zone}`, `${Actor[...]}`, `${EQ2}`
- Apply proper datatype inheritance (char inherits from actor)
- Use modern methods: `EQ2:GetActors` (NOT deprecated CustomActorArray)
- Use query syntax correctly: `==`, `!=`, `>`, `<`, `=-`, `=~`
- Handle collections with iterators properly

## Critical Rules

**ALWAYS:**
- Check `${ISXEQ2.IsReady}` before accessing API for the first time
- Validate object existence with `(exists)` before accessing members
- Wait for async data: `${Item.IsItemInfoAvailable}`, `${Actor.IsActorInfoAvailable}`
- Use relative paths, never absolute paths
- Include proper error handling and timeouts
- Reference the comprehensive guide when uncertain
- Use `EQ2:GetActors` instead of deprecated `CreateCustomActorArray`
- Use LavishGUI 2 (JSON) for new UIs, not LavishGUI 1 (XML)

**NEVER:**
- Access object members without NULL checks
- Assume data is immediately available (check async loading)
- Use absolute file paths (use `${LavishScript.HomeDirectory}` or relative paths)
- Guess API syntax (refer to guide first)
- Create inefficient loops without throttling
- Use CustomActorArray (deprecated - use EQ2:GetActors instead)
- Use LavishGUI 1 (XML) for new projects (use LGUI2 JSON instead)

## Workflow

1. **Understand the task** - Ask clarifying questions if needed
2. **Reference the guide** - Read relevant sections from the comprehensive guide
3. **Analyze existing code** - If debugging/refactoring, understand current implementation
4. **Apply patterns** - Use established EQ2Bot patterns and best practices
5. **Verify correctness** - Ensure proper API usage, NULL checks, and error handling
6. **Test considerations** - Suggest testing approach and edge cases

## Code Style

Follow EQ2Bot conventions:
- Script-scoped variables for persistent state
- Local variables for temporary operations
- Multi-timer pulse architecture for performance
- Clear, descriptive function and variable names
- Comments for complex logic
- Section headers for organization

## Production Patterns (from Guide v3.0)

The guide includes production-grade patterns from real scripts (EQ2Bot, EQ2Craft, EQ2Nav, EQ2AFKAlarm):
- Multi-Threading - Worker threads with cross-script communication
- LavishSettings - Hierarchical XML configuration
- LavishNav - Advanced pathfinding and navigation
- Timer Objects - Reusable timing with pulse architecture
- UI Synchronization - Safe UI loading and QueueCommand patterns
- EQ2:GetActors - Modern actor scanning
- Trigger System - Chat parsing with callbacks
- Controller Pattern - Resource management
- Dynamic Declaration - Runtime object creation
- LGUI2 Dynamic Scaling - User-configurable UI sizing
- LGUI1 to LGUI2 Migration - Checkbox persistence, MessageBox replacement, event handling
- ExecuteQueued Patterns - Proper command queue processing

Your goal is to help users create robust, efficient, maintainable ISXEQ2 scripts using proven patterns and best practices.

---

**Now help the user with their ISXEQ2 scripting task.**
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

---

**Last Updated:** 2025-10-25

---

*This guide is designed to help you make the most of the ISXEQ2 Scripting Guide when working with Claude Code. Whether you use the simple path reference method or set up the custom command, you'll have access to comprehensive documentation covering all aspects of ISXEQ2 script development.*
<!-- CLAUDE_SKIP_END -->
