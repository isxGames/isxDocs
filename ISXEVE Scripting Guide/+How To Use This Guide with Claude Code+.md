<!-- CLAUDE_SKIP_START -->
# How to Use This Guide with Claude Code

This document explains how to set up and use the ISXEVE Scripting Guide with Claude Code for InnerSpace/ISXEVE script development. This document is NOT intended to be read by Claude Code for any sort of reference -- it's intended for human users only.

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Installation](#installation)
3. [Using the /isxeve Command](#using-the-isxeve-command)
4. [How It Works](#how-it-works)
5. [Example Usage](#example-usage)
6. [Tips for Best Results](#tips-for-best-results)
7. [Alternative: Direct Path Reference](#alternative-direct-path-reference)
8. [Troubleshooting](#troubleshooting)

---

## Architecture Overview

The ISXEVE Claude Code integration uses a **coordinator/worker architecture** designed to conserve context while maintaining thoroughness:

```
┌─────────────────────────────────────────────────────────────────┐
│                     Your Conversation                           │
│                                                                 │
│  You ──► /isxeve ──► Coordinator ──► ISXEVE-Expert Agent       │
│                         │                    │                  │
│                    (lightweight)        (isolated context)      │
│                         │                    │                  │
│                    Asks questions       Reads docs              │
│                    Plans approach       Analyzes code           │
│                    Summarizes           Creates/edits files     │
│                         │                    │                  │
│                         ◄────── Results ─────┘                  │
└─────────────────────────────────────────────────────────────────┘
```

**Benefits:**
- **Context Conservation**: Large documentation files (up to 11,500 lines) are read in isolated agent context, not your main conversation
- **Thoroughness**: The agent has full access to all 23 documentation files
- **Efficiency**: Coordinator stays lightweight; only results return to main conversation
- **Sub-subagent Support**: For very large research tasks, the agent can spawn additional subagents

---

## Installation

The command and agent files are located in `Claude AI Commands (optional)/` within this guide directory.

### Step 1: Copy the Agent File

Copy `Claude AI Commands (optional)/ISXEVE-Expert.md` to your Claude agents directory:

```
~/.claude/agents/ISXEVE-Expert.md
```

On Windows, this is typically:
```
C:\Users\<YourUsername>\.claude\agents\ISXEVE-Expert.md
```

### Step 2: Copy the Command File

Copy `Claude AI Commands (optional)/isxeve.md` to your Claude commands directory:

```
~/.claude/commands/isxeve.md
```

On Windows, this is typically:
```
C:\Users\<YourUsername>\.claude\commands\isxeve.md
```

### Step 3: Update the Agent's Local Paths

Edit `ISXEVE-Expert.md` and update the **Local Paths** section at the very top of the file:

```
## Local Paths (UPDATE THESE FOR YOUR SYSTEM)

SCRIPTS_DIR: C:\Dev\InnerSpace\Scripts\
GUIDE_DIR:   C:\Dev\InnerSpace\isxDocs\ISXEVE Scripting Guide\
```

Change these paths to match where you have:
- **SCRIPTS_DIR**: Your InnerSpace Scripts directory
- **GUIDE_DIR**: This ISXEVE Scripting Guide directory

**Note:** The coordinator file (`isxeve.md`) has no paths to update - only the agent file needs configuration.

### Step 4: Verify Installation

1. Start a new Claude Code session
2. Type `/` and look for `isxeve` in the command list
3. If it appears, you're ready to go!

---

## Using the /isxeve Command

Once installed, simply invoke the command and describe what you need:

```
/isxeve I need a script that warps to asteroid belts and mines ore.
```

The coordinator will:
1. **Ask clarifying questions** if your request is ambiguous
2. **Delegate the heavy work** to the ISXEVE-Expert agent
3. **Summarize the results** and ask if you need anything else

---

## How It Works

### The Coordinator (`/isxeve`)

The coordinator is a lightweight prompt that:
- Understands user requests
- Asks clarifying questions when needed
- Delegates actual work to the ISXEVE-Expert agent
- Synthesizes and summarizes results

It does NOT read documentation files directly, keeping your main conversation context clean.

### The Worker Agent (ISXEVE-Expert)

The agent runs in an isolated context and:
- Reads documentation files (all 23 guides)
- Analyzes existing scripts
- Creates and edits script files
- Debugs issues
- Has full edit authority

**Large File Handling:** The agent knows which documentation files are large (3,000+ lines) and can spawn sub-subagents to read them if needed:
- `03_API_Reference.md` (~11,500 lines)
- `10_LavishGUI2_UI_Guide.md` (~7,600 lines)
- `05_Patterns_And_Best_Practices.md` (~7,500 lines)
- `07_Advanced_Patterns_And_Examples.md` (~7,000 lines)
- `17_Fleet_Operations.md` (~5,900 lines)
- `11_LavishGUI1_to_LavishGUI2_Migration.md` (~5,000 lines)
- `16_Mining_And_Hauling.md` (~4,000 lines)
- `18_Bot_Architecture_Analysis.md` (~3,800 lines)
- `20_Debugging_And_Troubleshooting.md` (~3,500 lines)
- `01_LavishScript_Fundamentals.md` (~3,000 lines)

---

## Example Usage

### Example 1: Creating a New Script

```
/isxeve

I need to create a script that:
1. Warps to asteroid belts
2. Targets and mines asteroids
3. Returns to station when cargo is full
```

The coordinator will ask clarifying questions like:
- "Should this handle multiple ore types or focus on specific ores?"
- "Do you need fleet support for a hauler alt?"
- "Should it include defensive measures for hostiles?"

Then it delegates to the agent to create the script.

### Example 2: Debugging Existing Code

```
/isxeve

My script crashes when I try to access ${Entity[TypeID,12345].Name}. Here's the code:
[paste your code]

What's wrong?
```

The agent will analyze your code, reference the API documentation, and identify the issue (likely missing NULL check or entity not existing).

### Example 3: Learning a Specific Topic

```
/isxeve I want to learn how to use the relay system for multi-boxing coordination. Show me examples.
```

The agent will read the Fleet Operations documentation and provide examples with explanations.

### Example 4: Combat Automation

```
/isxeve Help me create a ratting script that uses drones and manages tank.
```

The agent will reference the Combat Automation guide and create the script.

---

## Tips for Best Results

### Be Specific About Your Needs

**Good:**
```
/isxeve

I need a script that monitors nearby belts for ore anomalies, warps to them,
and mines the most valuable ore first. I want to use modern state machine patterns.
```

**Less Effective:**
```
/isxeve Help with mining.
```

### Mention Your Experience Level

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

It's not working as expected. The targeting loop never finds any rats.
```

### Ask for Modern Patterns

```
/isxeve Show me the modern way to manage combat targets using state machines
like Tehbot does.
```

---

## Alternative: Direct Path Reference

If you don't want to set up the command/agent system, you can still reference the guide directly:

```
I need help with ISXEVE scripting. Please read the documentation at:
C:\Path\To\ISXEVE Scripting Guide

Then help me [describe your task].
```

**However, this approach:**
- Consumes more context in your main conversation
- Requires specifying the path each time
- Doesn't benefit from the coordinator/worker architecture

The custom command setup is recommended for regular ISXEVE development.

---

## Troubleshooting

### Command Not Appearing

If `/isxeve` doesn't appear in the command list:

1. Verify the file exists at: `~/.claude/commands/isxeve.md`
2. Restart Claude Code
3. Check that the file has proper markdown formatting

### Agent Not Working

If the agent doesn't seem to work properly:

1. Verify the agent file exists at: `~/.claude/agents/ISXEVE-Expert.md`
2. Check that the Local Paths section has valid paths
3. Ensure the GUIDE_DIR path points to this guide directory

### Documentation Not Found

If the agent reports it can't find documentation:

1. Verify GUIDE_DIR in the agent file points to the correct location
2. Check that all documentation files exist in that directory
3. Make sure paths don't have trailing spaces

---

## File Reference

### Files in `Claude AI Commands (optional)/`

| File | Purpose |
|------|---------|
| `isxeve.md` | Coordinator command - copy to `~/.claude/commands/` |
| `ISXEVE-Expert.md` | Worker agent - copy to `~/.claude/agents/` |
| `README.md` | Quick reference for installation |

### Documentation Files (23 total)

| File | Lines | Description |
|------|-------|-------------|
| `01_LavishScript_Fundamentals.md` | ~3,000 | Language basics |
| `02_Quick_Start_Guide.md` | ~750 | Getting started |
| `03_API_Reference.md` | ~11,500 | Complete API docs |
| `04_Core_Concepts.md` | ~860 | Core concepts |
| `05_Patterns_And_Best_Practices.md` | ~7,500 | Best practices |
| `06_Working_Examples.md` | ~1,150 | Working examples |
| `07_Advanced_Patterns_And_Examples.md` | ~7,000 | Advanced patterns |
| `08_LavishGUI1_UI_Guide.md` | ~1,090 | Legacy UI (XML-based) |
| `09_Advanced_LGUI1_Patterns.md` | ~910 | Advanced LGUI1 patterns |
| `10_LavishGUI2_UI_Guide.md` | ~7,600 | Modern UI (recommended) |
| `11_LavishGUI1_to_LavishGUI2_Migration.md` | ~5,000 | UI migration |
| `12_LGUI2_Scaling_System.md` | ~1,100 | UI scaling |
| `13_JSON_Guide.md` | ~1,700 | JSON usage |
| `14_LavishMachine_Guide.md` | ~1,950 | Async tasks |
| `15_Combat_Automation.md` | ~2,900 | Combat patterns |
| `16_Mining_And_Hauling.md` | ~4,000 | Mining patterns |
| `17_Fleet_Operations.md` | ~5,900 | Fleet coordination |
| `18_Bot_Architecture_Analysis.md` | ~3,800 | Bot analysis |
| `19_DotNet_Development.md` | ~1,050 | .NET development |
| `20_Debugging_And_Troubleshooting.md` | ~3,500 | Debugging |
| `21_Advanced_Scripting_Patterns.md` | ~2,500 | Production patterns |
| `22_Utility_Script_Patterns.md` | ~800 | Utility patterns |

---

*This guide helps you make the most of the ISXEVE Scripting Guide when working with Claude Code. The coordinator/worker architecture ensures efficient context usage while providing comprehensive documentation access.*
<!-- CLAUDE_SKIP_END -->
