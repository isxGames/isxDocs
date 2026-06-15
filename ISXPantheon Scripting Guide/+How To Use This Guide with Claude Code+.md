<!-- CLAUDE_SKIP_START -->
# How to Use This Guide with Claude Code

This document explains how to set up and use the ISXPantheon Scripting Guide with Claude Code for InnerSpace/ISXPantheon script development. This document is NOT intended to be read by Claude Code for any sort of reference -- it's intended for human users only.

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Installation](#installation)
3. [Using the /isxpantheon Command](#using-the-isxpantheon-command)
4. [How It Works](#how-it-works)
5. [Example Usage](#example-usage)
6. [Tips for Best Results](#tips-for-best-results)
7. [Alternative: Direct Path Reference](#alternative-direct-path-reference)
8. [Troubleshooting](#troubleshooting)

---

## Architecture Overview

The ISXPantheon Claude Code integration uses a **coordinator/worker architecture** designed to conserve context while maintaining thoroughness:

```
┌─────────────────────────────────────────────────────────────────┐
│                     Your Conversation                           │
│                                                                 │
│  You ──► /isxpantheon ──► Coordinator ──► ISXPantheon-Expert    │
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
- **Thoroughness**: The agent has full access to all of the documentation files
- **Efficiency**: Coordinator stays lightweight; only results return to main conversation
- **Sub-subagent Support**: For very large research tasks, the agent can spawn additional subagents

---

## Installation

The command and agent files are located in `Claude AI Commands (optional)/` within this guide directory. See `README.md` in that directory for a quick-start summary.

### Step 1: Copy the Agent File

Copy `Claude AI Commands (optional)/ISXPantheon-Expert.md` to your Claude agents directory:

```
~/.claude/agents/ISXPantheon-Expert.md
```

On Windows, this is typically:
```
C:\Users\<YourUsername>\.claude\agents\ISXPantheon-Expert.md
```

### Step 2: Copy the Command File

Copy `Claude AI Commands (optional)/isxpantheon.md` to your Claude commands directory:

```
~/.claude/commands/isxpantheon.md
```

On Windows, this is typically:
```
C:\Users\<YourUsername>\.claude\commands\isxpantheon.md
```

### Step 3: Update the Agent's Local Paths

Edit `ISXPantheon-Expert.md` and update the **Local Paths** section at the very top of the file:

```
## Local Paths (UPDATE THESE FOR YOUR SYSTEM)

SCRIPTS_DIR:    C:\Dev\InnerSpace\Scripts\
GUIDE_DIR:      C:\Dev\InnerSpace\isxDocs\ISXPantheon Scripting Guide\
```

Change these paths to match where you have:
- **SCRIPTS_DIR**: Your InnerSpace Scripts directory
- **GUIDE_DIR**: This ISXPantheon Scripting Guide directory

### Step 4: Update the Coordinator's Paths

Edit `isxpantheon.md` and update the **User Configuration** table at the very top of the file:

```
| Setting | Path |
|---------|------|
| **Scripts Directory** | `C:\Dev\InnerSpace\Scripts` |
| **Guide Directory** | `C:\Dev\InnerSpace\isxDocs\ISXPantheon Scripting Guide` |
```

Change these paths to match where you have:
- **Scripts Directory**: Your InnerSpace Scripts directory
- **Guide Directory**: This ISXPantheon Scripting Guide directory

### Step 5: Verify Installation

1. Close and reopen Claude Code (commands and agents are loaded at startup)
2. Type `/` and look for `isxpantheon` in the command list
3. If it appears, you're ready to go!

---

## Using the /isxpantheon Command

Once installed, simply invoke the command and describe what you need:

```
/isxpantheon I need a script that fetches a JSON status from a web endpoint and stores a value.
```

The coordinator will:
1. **Ask clarifying questions** if your request is ambiguous
2. **Delegate the heavy work** to the ISXPantheon-Expert agent
3. **Summarize the results** and ask if you need anything else

---

## How It Works

### The Coordinator (`/isxpantheon`)

The coordinator is a lightweight prompt that:
- Understands user requests
- Asks clarifying questions when needed
- Delegates actual work to the ISXPantheon-Expert agent
- Synthesizes and summarizes results

It does NOT read documentation files directly, keeping your main conversation context clean.

### The Worker Agent (ISXPantheon-Expert)

The agent runs in an isolated context and:
- Reads documentation files (all numbered guides plus the `01b_LavishScript_Reference.md` reference companion to file 01)
- Analyzes existing scripts
- Creates and edits script files
- Debugs issues
- Has full edit authority

**Large File Handling:** The agent knows which documentation files are large and can spawn sub-subagents to read them if needed:
- `01_LavishScript_Fundamentals.md`
- `03_API_Reference.md`
- `10_LavishGUI2_UI_Guide.md`
- `11_LavishGUI1_to_LavishGUI2_Migration.md`
- `15_Advanced_Scripting_Patterns.md`
- `16_Utility_Script_Patterns.md`

---

## Example Usage

### Example 1: Creating a New Script

```
/isxpantheon

I need to create a script that:
1. Waits for the extension to be ready
2. Fetches a JSON status from a web endpoint
3. Stores a value from the response as a custom variable
```

The coordinator will ask clarifying questions like:
- "Which endpoint should it request?"
- "What value from the response do you care about?"
- "Should it poll on a timer or run once?"

Then it delegates to the agent to create the script.

### Example 2: Debugging Existing Code

```
/isxpantheon

My script does nothing after it starts. Here's the code:
[paste your code]

What's wrong?
```

The agent will analyze your code, reference the API documentation, and identify the issue (often a missing `${ISXPantheon.IsReady}` gate or reliance on a planned, not-yet-implemented feature).

### Example 3: Learning a Specific Topic

```
/isxpantheon I want to learn how to use GetURL and the isxGames_onHTTPResponse event. Show me examples.
```

The agent will read the relevant documentation and provide examples with explanations.

### Example 4: UI Development

```
/isxpantheon Help me create a LGUI2 settings panel for my addon with checkboxes and a dropdown.
```

The agent will reference the LGUI2 UI Guide and create the JSON UI definition.

---

## Tips for Best Results

### Be Specific About Your Needs

**Good:**
```
/isxpantheon I need a script that polls a web endpoint every 30 seconds, parses
the JSON response, and updates a LGUI2 label with one of its fields.
```

**Less Effective:**
```
/isxpantheon Help with a script.
```

### Mention Your Experience Level

```
/isxpantheon I'm familiar with LavishScript but new to ISXPantheon. I need help understanding
the custom-variable members on the ${ISXPantheon} TLO.
```

### Reference Existing Code When Debugging

```
/isxpantheon

Here's my code:
[paste code]

It's not working as expected. Nothing happens after the script starts.
```

### Ask About What Is Available Today

```
/isxpantheon Which game-data features are actually implemented right now, and which are still planned?
```

---

## Alternative: Direct Path Reference

If you don't want to set up the command/agent system, you can still reference the guide directly:

```
I need help with ISXPantheon scripting. Please read the documentation at:
C:\Path\To\ISXPantheon Scripting Guide

Then help me [describe your task].
```

**However, this approach:**
- Consumes more context in your main conversation
- Requires specifying the path each time
- Doesn't benefit from the coordinator/worker architecture

The custom command setup is recommended for regular ISXPantheon development.

---

## Troubleshooting

### Command Not Appearing

If `/isxpantheon` doesn't appear in the command list:

1. Verify the file exists at: `~/.claude/commands/isxpantheon.md`
2. Restart Claude Code
3. Check that the file has proper markdown formatting

### Agent Not Working

If the agent doesn't seem to work properly:

1. Verify the agent file exists at: `~/.claude/agents/ISXPantheon-Expert.md`
2. Check that the Local Paths section has valid paths
3. Ensure the GUIDE_DIR path points to this guide directory

### Documentation Not Found

If the agent reports it can't find documentation:

1. Verify GUIDE_DIR in the agent file points to the correct location
2. Check that all documentation files exist in that directory
3. Make sure paths don't have trailing spaces

<!-- CLAUDE_SKIP_END -->
