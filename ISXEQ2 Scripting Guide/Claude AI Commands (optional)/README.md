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

> **Note:** The coordinator (`isxeq2.md`) has no paths - only the agent needs updating.

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

- Reads your comprehensive guide (17 documentation files)
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
