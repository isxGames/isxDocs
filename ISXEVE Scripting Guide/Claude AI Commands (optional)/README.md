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

1. Copy `ISXEVE-Expert.md` to `~/.claude/agents/`
2. Copy `isxeve.md` to `~/.claude/commands/`
3. Edit `ISXEVE-Expert.md` and update the **Local Paths** section at the top:
   - `SCRIPTS_DIR`: Your InnerSpace Scripts directory
   - `GUIDE_DIR`: Your ISXEVE Scripting Guide directory

> **Note:** The coordinator (`isxeve.md`) has no paths - only the agent needs updating.

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

- Reads your comprehensive guide (23 documentation files)
- Knows your directories (Scripts, Guide location)
- Follows best practices (EVEBot patterns, NULL checks, async data handling)
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

ISXEVE has 10 large files (3,000+ lines) that the agent will delegate to sub-subagents when needed, including the massive API Reference (~11,500 lines).
