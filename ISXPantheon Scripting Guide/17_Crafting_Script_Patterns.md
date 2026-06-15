# Crafting Script Patterns

**Planned roadmap feature — crafting automation for ISXPantheon.**

---

> **PLANNED — NOT YET IMPLEMENTED.** Crafting automation is a planned ISXPantheon feature. It is not available in the current build, and no crafting API exists yet. Everything below describes the intended direction only. The names, members, and methods are provisional and WILL change. Do not write production scripts against any of it.

## Overview

Crafting in Pantheon: Rise of the Fallen is a player subsystem that ISXPantheon intends to surface for scripting in a future release. A `Crafting` top-level object and a `crafting` datatype are planned, but they are not implemented and their members and methods are not defined yet.

Until then, this document is a roadmap placeholder. There is no recipe queue, no crafting-window automation, no reaction/quality mechanics, and no station/device targeting available today.

## Planned Surface

The following are anticipated, not implemented:

- A **`Crafting` top-level object** **(planned — not yet implemented)** — the planned entry point for the crafting subsystem.
- A **`crafting` datatype** **(planned — not yet implemented)** — the planned return type for that object.

The members and methods of these are not defined yet and are intentionally not described here. They will be documented once the feature is implemented and verified against the extension, following the same conventions as the other game-data datatypes in this guide.

## What to Use Today

There is nothing crafting-specific to use yet. General-purpose infrastructure that a future crafting script will build on is already available and documented elsewhere in this guide:

- Custom timers, time/rate calculations, dynamic file loading, input validation, and queued-command handling — see [16_Utility_Script_Patterns.md](16_Utility_Script_Patterns.md).
- Multi-threading, LavishSettings configuration, UI synchronization, and trigger-based text parsing — see [15_Advanced_Scripting_Patterns.md](15_Advanced_Scripting_Patterns.md).

When the crafting surface lands, this document will be expanded into a full pattern guide. Until then, treat crafting as a roadmap item.
