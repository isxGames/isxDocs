<!-- CLAUDE_SKIP_START -->
# How to Use This Guide with Claude Code

This document explains how to set up and use the ISXEQ2 Scripting Guide with Claude Code for InnerSpace/ISXEQ2 script development. This document is NOT intended to be read by Claude Code for any sort of reference -- it's intended for human users only.

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Installation](#installation)
3. [Using the /isxeq2 Command](#using-the-isxeq2-command)
4. [How It Works](#how-it-works)
5. [Example Usage](#example-usage)
6. [Tips for Best Results](#tips-for-best-results)
7. [Alternative: Direct Path Reference](#alternative-direct-path-reference)
8. [Troubleshooting](#troubleshooting)

---

## Architecture Overview

The ISXEQ2 Claude Code integration uses a **coordinator/worker architecture** designed to conserve context while maintaining thoroughness:

```
┌─────────────────────────────────────────────────────────────────┐
│                     Your Conversation                           │
│                                                                 │
│  You ──► /isxeq2 ──► Coordinator ──► ISXEQ2-Expert Agent       │
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
- **Context Conservation**: Large documentation files (up to 7,500 lines) are read in isolated agent context, not your main conversation
- **Thoroughness**: The agent has full access to all 19 documentation files
- **Efficiency**: Coordinator stays lightweight; only results return to main conversation
- **Sub-subagent Support**: For very large research tasks, the agent can spawn additional subagents

---

## Installation

The command and agent files are located in `Claude AI Commands (optional)/` within this guide directory.

### Step 1: Copy the Agent File

Copy `Claude AI Commands (optional)/ISXEQ2-Expert.md` to your Claude agents directory:

```
~/.claude/agents/ISXEQ2-Expert.md
```

On Windows, this is typically:
```
C:\Users\<YourUsername>\.claude\agents\ISXEQ2-Expert.md
```

### Step 2: Copy the Command File

Copy `Claude AI Commands (optional)/isxeq2.md` to your Claude commands directory:

```
~/.claude/commands/isxeq2.md
```

On Windows, this is typically:
```
C:\Users\<YourUsername>\.claude\commands\isxeq2.md
```

### Step 3: Update the Agent's Local Paths

Edit `ISXEQ2-Expert.md` and update the **Local Paths** section at the very top of the file:

```
## Local Paths (UPDATE THESE FOR YOUR SYSTEM)

SCRIPTS_DIR: C:\Dev\InnerSpace\Scripts\
GUIDE_DIR:   C:\Dev\InnerSpace\isxDocs\ISXEQ2 Scripting Guide\
```

Change these paths to match where you have:
- **SCRIPTS_DIR**: Your InnerSpace Scripts directory
- **GUIDE_DIR**: This ISXEQ2 Scripting Guide directory

**Note:** The coordinator file (`isxeq2.md`) has no paths to update - only the agent file needs configuration.

### Step 4: Verify Installation

1. Start a new Claude Code session
2. Type `/` and look for `isxeq2` in the command list
3. If it appears, you're ready to go!

---

## Using the /isxeq2 Command

Once installed, simply invoke the command and describe what you need:

```
/isxeq2 I need a script that monitors my health and casts heals when below 50%.
```

The coordinator will:
1. **Ask clarifying questions** if your request is ambiguous
2. **Delegate the heavy work** to the ISXEQ2-Expert agent
3. **Summarize the results** and ask if you need anything else

---

## How It Works

### The Coordinator (`/isxeq2`)

The coordinator is a lightweight prompt that:
- Understands user requests
- Asks clarifying questions when needed
- Delegates actual work to the ISXEQ2-Expert agent
- Synthesizes and summarizes results

It does NOT read documentation files directly, keeping your main conversation context clean.

### The Worker Agent (ISXEQ2-Expert)

The agent runs in an isolated context and:
- Reads documentation files (all 19 guides)
- Analyzes existing scripts
- Creates and edits script files
- Debugs issues
- Has full edit authority

**Large File Handling:** The agent knows which documentation files are large (3,000+ lines) and can spawn sub-subagents to read them if needed:
- `01_LavishScript_Fundamentals.md` (~3,000 lines)
- `03_API_Reference.md` (~3,400 lines)
- `10_LavishGUI2_UI_Guide.md` (~7,500 lines)
- `15_Advanced_Scripting_Patterns.md` (~4,000 lines)
- `16_Utility_Script_Patterns.md` (~3,200 lines)

---

## Example Usage

### Example 1: Creating a New Script

```
/isxeq2

I need to create a script that:
1. Monitors my health
2. Casts a heal spell when below 50%
3. Uses a potion if the spell is on cooldown
```

The coordinator will ask clarifying questions like:
- "Should this be a standalone script or integrate with EQ2Bot?"
- "What heal spell should it use?"
- "Should it check for combat state?"

Then it delegates to the agent to create the script.

### Example 2: Debugging Existing Code

```
/isxeq2

My script crashes when I try to access ${Actor[chest].Name}. Here's the code:
[paste your code]

What's wrong?
```

The agent will analyze your code, reference the API documentation, and identify the issue (likely missing NULL check).

### Example 3: Learning a Specific Topic

```
/isxeq2 I want to learn how to use EQ2:GetActors for actor scanning. Show me examples.
```

The agent will read the relevant documentation and provide examples with explanations.

### Example 4: UI Development

```
/isxeq2 Help me create a LGUI2 settings panel for my addon with checkboxes and a dropdown.
```

The agent will reference the LGUI2 UI Guide and create the JSON UI definition.

---

## Tips for Best Results

### Be Specific About Your Needs

**Good:**
```
/isxeq2 I need a script that monitors my target's health and automatically
assists when it drops below 20%. I want to use the modern EQ2:GetActors pattern.
```

**Less Effective:**
```
/isxeq2 Help with combat.
```

### Mention Your Experience Level

```
/isxeq2 I'm familiar with LavishScript but new to ISXEQ2. I need help understanding
how to query for nearby NPCs using the Actor TLO.
```

### Reference Existing Code When Debugging

```
/isxeq2

Here's my code:
[paste code]

It's not working as expected. The loop never finds any actors.
```

### Ask for Modern Patterns

```
/isxeq2 Show me the modern way to search for actors. I know CustomActorArray is deprecated.
```

---

## Alternative: Direct Path Reference

If you don't want to set up the command/agent system, you can still reference the guide directly:

```
I need help with ISXEQ2 scripting. Please read the documentation at:
C:\Path\To\ISXEQ2 Scripting Guide

Then help me [describe your task].
```

**However, this approach:**
- Consumes more context in your main conversation
- Requires specifying the path each time
- Doesn't benefit from the coordinator/worker architecture

The custom command setup is recommended for regular ISXEQ2 development.

---

## Troubleshooting

### Command Not Appearing

If `/isxeq2` doesn't appear in the command list:

1. Verify the file exists at: `~/.claude/commands/isxeq2.md`
2. Restart Claude Code
3. Check that the file has proper markdown formatting

### Agent Not Working

If the agent doesn't seem to work properly:

1. Verify the agent file exists at: `~/.claude/agents/ISXEQ2-Expert.md`
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
| `isxeq2.md` | Coordinator command - copy to `~/.claude/commands/` |
| `ISXEQ2-Expert.md` | Worker agent - copy to `~/.claude/agents/` |
| `README.md` | Quick reference for installation |

### Documentation Files (19 total)

| File | Lines | Description |
|------|-------|-------------|
| `01_LavishScript_Fundamentals.md` | ~3,000 | Language basics |
| `02_Quick_Start_Guide.md` | ~750 | Getting started |
| `03_API_Reference.md` | ~3,400 | Complete API docs |
| `04_Core_Concepts.md` | ~860 | Core concepts |
| `05_Patterns_And_Best_Practices.md` | ~1,100 | Best practices |
| `06_Working_Examples.md` | ~1,150 | Working examples |
| `07_Advanced_Patterns_And_Examples.md` | ~1,600 | Advanced patterns |
| `08_LavishGUI1_UI_Guide.md` | ~1,300 | Legacy UI (XML-based) |
| `09_Advanced_LGUI1_Patterns.md` | ~1,450 | Advanced LGUI1 patterns |
| `10_LavishGUI2_UI_Guide.md` | ~7,500 | Modern UI (recommended) |
| `11_LavishGUI1_to_LavishGUI2_Migration.md` | ~4,900 | UI migration |
| `12_LGUI2_Scaling_System.md` | ~1,100 | UI scaling |
| `13_JSON_Guide.md` | ~1,700 | JSON usage |
| `14_LavishMachine_Guide.md` | ~1,950 | Async tasks |
| `15_Advanced_Scripting_Patterns.md` | ~4,000 | Production patterns |
| `16_Utility_Script_Patterns.md` | ~3,200 | Utility patterns |
| `17_Crafting_Script_Patterns.md` | ~1,700 | Crafting patterns |
| `18_Navigation_Library_Patterns.md` | ~1,050 | Navigation/pathfinding |

---

*This guide helps you make the most of the ISXEQ2 Scripting Guide when working with Claude Code. The coordinator/worker architecture ensures efficient context usage while providing comprehensive documentation access.*
<!-- CLAUDE_SKIP_END -->
