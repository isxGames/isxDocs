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
- **Context Conservation**: Large documentation files are read in isolated agent context, not your main conversation
- **Thoroughness**: The agent has full access to all 22 numbered guides plus meta/index files
- **Efficiency**: Coordinator stays lightweight; only results return to main conversation
- **Sub-subagent Support**: For very large research tasks, the agent can spawn additional subagents

---

## Installation

The command and agent files are located in the `Claude AI Commands (optional)/` subdirectory within this guide directory. See [Claude AI Commands (optional)/README.md](Claude%20AI%20Commands%20(optional)/README.md) for a quick-start summary.

### Step 1: Copy the Agent File

Copy [Claude AI Commands (optional)/ISXEVE-Expert.md](Claude%20AI%20Commands%20(optional)/ISXEVE-Expert.md) to your Claude agents directory:

```
~/.claude/agents/ISXEVE-Expert.md
```

On Windows, this is typically:
```
C:\Users\<YourUsername>\.claude\agents\ISXEVE-Expert.md
```

### Step 2: Copy the Command File

Copy [Claude AI Commands (optional)/isxeve.md](Claude%20AI%20Commands%20(optional)/isxeve.md) to your Claude commands directory:

```
~/.claude/commands/isxeve.md
```

On Windows, this is typically:
```
C:\Users\<YourUsername>\.claude\commands\isxeve.md
```

### Step 3: Update Paths in Both Files

Both [ISXEVE-Expert.md](Claude%20AI%20Commands%20(optional)/ISXEVE-Expert.md) (agent) and [isxeve.md](Claude%20AI%20Commands%20(optional)/isxeve.md) (coordinator) have a paths block near the top. Update all four paths in **both** files to match your system:

- **Scripts Directory**: Your InnerSpace Scripts directory
- **Guide Directory**: This ISXEVE Scripting Guide directory
- **Changes File**: The ISXEVEChanges.txt file from your ISXEVE installation
- **Quick Reference**: The path to [ISXEVE_QuickReference.md](../ISXEVE_QuickReference.md) (in the parent `isxDocs` directory, OUTSIDE this guide directory)

[ISXEVE-Expert.md](Claude%20AI%20Commands%20(optional)/ISXEVE-Expert.md) uses a **Local Paths** block:

```
## Local Paths (UPDATE THESE FOR YOUR SYSTEM)

SCRIPTS_DIR:    C:\Dev\InnerSpace\Scripts\
GUIDE_DIR:      C:\Dev\InnerSpace\isxDocs\ISXEVE Scripting Guide\
CHANGES_FILE:   C:\Dev\InnerSpace\ISXEVE\Install\x64\Extensions\ISXDK35\ISXEVEChanges.txt
QUICK_REF_FILE: C:\Dev\InnerSpace\isxDocs\ISXEVE_QuickReference.md
```

[isxeve.md](Claude%20AI%20Commands%20(optional)/isxeve.md) uses a **User Configuration** table:

```
| Setting | Path |
|---------|------|
| **Scripts Directory** | `C:\Dev\InnerSpace\Scripts` |
| **Guide Directory** | `C:\Dev\InnerSpace\isxDocs\ISXEVE Scripting Guide` |
| **Changes File** | `C:\Dev\InnerSpace\ISXEVE\Install\x64\Extensions\ISXDK35\ISXEVEChanges.txt` |
| **Quick Reference** | `C:\Dev\InnerSpace\isxDocs\ISXEVE_QuickReference.md` |
```

### Step 4: Verify Installation

1. Close and reopen Claude Code (commands and agents are loaded at startup)
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
- Reads documentation files (all 22 numbered guides plus meta/index files)
- Analyzes existing scripts
- Creates and edits script files
- Debugs issues
- Has full edit authority

**Large File Handling:** The agent knows which documentation files are large (3,000+ lines) and can spawn sub-subagents to read them if needed. For the canonical list — including file sizes and handling guidance for `CHANGES_FILE` — see the **Context Efficiency** section in [ISXEVE-Expert.md](Claude%20AI%20Commands%20(optional)/ISXEVE-Expert.md).

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
2. Verify the command file exists at: `~/.claude/commands/isxeve.md`
3. Check that the **Local Paths** block in the agent and the **User Configuration** table in the coordinator BOTH have valid paths for all four settings (Scripts, Guide, Changes File, Quick Reference)
4. Ensure the `GUIDE_DIR` / Guide Directory path points to this guide directory and `QUICK_REF_FILE` / Quick Reference points to `ISXEVE_QuickReference.md` in the parent `isxDocs` directory

### Documentation Not Found

If the agent reports it can't find documentation:

1. Verify GUIDE_DIR in the agent file points to the correct location
2. Check that all documentation files exist in that directory
3. Make sure paths don't have trailing spaces

<!-- CLAUDE_SKIP_END -->
