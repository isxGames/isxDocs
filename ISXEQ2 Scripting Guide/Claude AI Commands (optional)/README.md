# ISXEQ2-Expert - Quick Start Guide

## Architecture

The ISXEQ2 system uses a **coordinator/worker architecture**:

| Component | Role |
|-----------|------|
| `/isxeq2` command (coordinator) | Handles user interaction, asks clarifying questions, delegates heavy work to the agent |
| `ISXEQ2-Expert` agent (worker) | Does the actual work: reads documentation, analyzes scripts, creates/edits files |

This conserves context in the main conversation while maintaining thoroughness.

---

## Installation

1. Copy `ISXEQ2-Expert.md` to `~/.claude/agents/`
2. Copy `isxeq2.md` to `~/.claude/commands/`
3. Edit `ISXEQ2-Expert.md` and update the **Local Paths** section at the top:
   - `SCRIPTS_DIR`: Your InnerSpace Scripts directory
   - `GUIDE_DIR`: Your ISXEQ2 Scripting Guide directory
   - `CHANGES_FILE`: The ISXEQ2Changes.txt file from your ISXEQ2 installation
   - `QUICK_REF_FILE`: Path to `ISXEQ2_QuickReference.md` (in the parent `isxDocs` directory)
4. Edit `isxeq2.md` and update the **User Configuration** table at the top:
   - **Scripts Directory**: Your InnerSpace Scripts directory
   - **Guide Directory**: Your ISXEQ2 Scripting Guide directory
   - **Changes File**: The ISXEQ2Changes.txt file from your ISXEQ2 installation
   - **Quick Reference**: Path to `ISXEQ2_QuickReference.md`

---

## How to Use

Simply invoke `/isxeq2` and describe what you need:

```
/isxeq2 I need a script that monitors my health and alerts me
```

The coordinator will:
1. Ask clarifying questions if needed
2. Delegate heavy work to the ISXEQ2-Expert agent
3. Summarize results

You can also spawn the agent directly via Task tool:

```
Task(subagent_type="ISXEQ2-Expert", prompt="Create a health monitoring script...")
```

---

## What the Agent Does

- Reads your comprehensive guide (19 documentation files + quick reference)
- Knows your directories (Scripts, Guide location)
- Follows best practices (EQ2Bot patterns, NULL checks, async data handling)
- Handles all tasks (creating, editing, debugging, refactoring scripts)
- Applies correct API usage (TLOs, datatypes, queries, events, commands)
- Ensures code quality (multi-timer pulses, error handling, naming conventions)
- Can spawn sub-subagents for complex research tasks

---

## Context Efficiency

The architecture is designed to conserve context:

- Coordinator stays lightweight (no large file reads)
- Agent reads documentation in isolated context
- Large script analysis happens in agent context
- Only results/summaries return to main conversation
