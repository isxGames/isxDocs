# ISXEVE-Expert - Quick Start Guide

## Architecture

The ISXEVE system uses a **coordinator/worker architecture**:

| Component | Role |
|-----------|------|
| `/isxeve` command (coordinator) | Handles user interaction, asks clarifying questions, delegates heavy work to the agent |
| `ISXEVE-Expert` agent (worker) | Does the actual work: reads documentation, analyzes scripts, creates/edits files |

This conserves context in the main conversation while maintaining thoroughness.

---

## Installation

1. Copy [ISXEVE-Expert.md](ISXEVE-Expert.md) to `~/.claude/agents/`
2. Copy [isxeve.md](isxeve.md) to `~/.claude/commands/`
3. Edit [ISXEVE-Expert.md](ISXEVE-Expert.md) and update the **Local Paths** section at the top:
   - `SCRIPTS_DIR`: Your InnerSpace Scripts directory
   - `GUIDE_DIR`: Your ISXEVE Scripting Guide directory
   - `CHANGES_FILE`: The ISXEVEChanges.txt file from your ISXEVE installation
   - `QUICK_REF_FILE`: Path to [ISXEVE_QuickReference.md](../../ISXEVE_QuickReference.md) (in the parent `isxDocs` directory)
4. Edit [isxeve.md](isxeve.md) and update the **User Configuration** table at the top:
   - **Scripts Directory**: Your InnerSpace Scripts directory
   - **Guide Directory**: Your ISXEVE Scripting Guide directory
   - **Changes File**: The ISXEVEChanges.txt file from your ISXEVE installation
   - **Quick Reference**: Path to [ISXEVE_QuickReference.md](../../ISXEVE_QuickReference.md)

---

## How to Use

Simply invoke `/isxeve` and describe what you need:

```
/isxeve I need a script that manages my mining operation
```

The coordinator will:
1. Ask clarifying questions if needed
2. Delegate heavy work to the ISXEVE-Expert agent
3. Summarize results

You can also spawn the agent directly via Task tool:

```
Task(subagent_type="ISXEVE-Expert", prompt="Create a mining script...")
```

---

## What the Agent Does

- Reads your comprehensive guide (22 numbered guides plus meta/index files + quick reference)
- Knows your directories (Scripts, Guide, Changes File, Quick Reference)
- Follows best practices (EVEBot, Yamfa, Tehbot, Metatron patterns, NULL checks, async data handling)
- Handles all tasks (creating, editing, debugging, refactoring scripts)
- Applies correct API usage (TLOs, datatypes, entities, modules, movement)
- Ensures code quality (multi-timer pulses, error handling, naming conventions)
- Can spawn sub-subagents for complex research tasks

---

## Context Efficiency

The architecture is designed to conserve context:

- Coordinator stays lightweight (no large file reads)
- Agent reads documentation in isolated context
- Large script analysis happens in agent context
- Only results/summaries return to main conversation

ISXEVE has several large files that the agent will delegate to sub-subagents when needed, including the massive API Reference.
