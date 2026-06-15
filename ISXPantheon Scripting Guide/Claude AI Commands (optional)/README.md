# ISXPantheon-Expert - Quick Start Guide

## Architecture

The ISXPantheon system uses a **coordinator/worker architecture**:

| Component | Role |
|-----------|------|
| `/isxpantheon` command (coordinator) | Handles user interaction, asks clarifying questions, delegates heavy work to the agent |
| `ISXPantheon-Expert` agent (worker) | Does the actual work: reads documentation, analyzes scripts, creates/edits files |

This conserves context in the main conversation while maintaining thoroughness.

---

## Installation

1. Copy `ISXPantheon-Expert.md` to `~/.claude/agents/`
2. Copy `isxpantheon.md` to `~/.claude/commands/`
3. Edit `ISXPantheon-Expert.md` and update the **Local Paths** section at the top:
   - `SCRIPTS_DIR`: Your InnerSpace Scripts directory
   - `GUIDE_DIR`: Your ISXPantheon Scripting Guide directory
4. Edit `isxpantheon.md` and update the **User Configuration** table at the top:
   - **Scripts Directory**: Your InnerSpace Scripts directory
   - **Guide Directory**: Your ISXPantheon Scripting Guide directory

---

## How to Use

Simply invoke `/isxpantheon` and describe what you need:

```
/isxpantheon I need a script that fetches a JSON status from a web endpoint and reacts to it
```

The coordinator will:
1. Ask clarifying questions if needed
2. Delegate heavy work to the ISXPantheon-Expert agent
3. Summarize results

You can also spawn the agent directly via Task tool:

```
Task(subagent_type="ISXPantheon-Expert", prompt="Create a script that fetches a URL...")
```

---

## What the Agent Does

- Reads your comprehensive guide (the numbered documentation files plus the README and quick-start)
- Knows your directories (Scripts, Guide location)
- Follows best practices (readiness gating, NULL checks, pulse architecture)
- Handles all tasks (creating, editing, debugging, refactoring scripts)
- Applies correct API usage against the real ISXPantheon surface, and flags planned features as not-yet-implemented
- Ensures code quality (multi-timer pulses, error handling, naming conventions)
- Can spawn sub-subagents for complex research tasks

---

## Context Efficiency

The architecture is designed to conserve context:

- Coordinator stays lightweight (no large file reads)
- Agent reads documentation in isolated context
- Large script analysis happens in agent context
- Only results/summaries return to main conversation
