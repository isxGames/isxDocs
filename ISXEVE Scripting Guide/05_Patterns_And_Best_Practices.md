# Patterns And Best Practices

**Purpose:** Main loop patterns, decision making, error handling, and performance optimization for EVE bots
**Audience:** Developers building production-quality automation

> **Note on Code Examples**: This file contains illustrative examples that may include deprecated API patterns (e.g., `MyShip.CargoFreeSpace`, `MyShip.UsedCargoCapacity`). For production code, always use the modern `EVEWindow[Inventory].Child[ShipCargo]`. The complete working example at the end has been updated to use modern APIs where critical.

---

## Table of Contents

### Main Loop and State Machines
1. [Main Loop and State Machines](#main-loop-and-state-machines)
2. [Bot Lifecycle Overview](#bot-lifecycle-overview)
3. [Main Loop Architectures](#main-loop-architectures)
4. [Simple Polling Loop Pattern](#simple-polling-loop-pattern)
5. [Behavior-Based State Machine](#behavior-based-state-machine)
6. [Event-Driven Architecture](#event-driven-architecture)
7. [State Machine Concepts](#state-machine-concepts)
8. [State Transition Patterns](#state-transition-patterns)
9. [Timing and Frame Management](#timing-and-frame-management)
10. [Safety-First Architecture](#safety-first-architecture)
11. [Common Patterns from Example Bots](#common-patterns-from-example-bots)
12. [Performance Considerations (Main Loop)](#performance-considerations)
13. [Common Mistakes and Gotchas](#common-mistakes-and-gotchas)

### Decision Making and Logic
14. [Decision Making and Logic Patterns](#decision-making-and-logic-patterns)
15. [Decision-Making Overview](#decision-making-overview)
16. [Target Selection Patterns](#target-selection-patterns)
17. [Priority Systems](#priority-systems)
18. [Condition Checking Patterns](#condition-checking-patterns)
19. [Decision Trees](#decision-trees)
20. [Weighted Scoring Systems](#weighted-scoring-systems)
21. [Real Examples from Bots (Decision)](#real-examples-from-bots)
22. [Mining Decision Logic](#mining-decision-logic)
23. [Combat Decision Logic](#combat-decision-logic)
24. [Movement Decision Logic](#movement-decision-logic)
25. [Common Patterns (Decision)](#common-patterns)
26. [Performance Considerations (Decision)](#performance-considerations-1)

### Error Handling and Recovery
27. [Error Handling and Recovery](#error-handling-and-recovery)
28. [Error Handling Overview](#error-handling-overview)
29. [Types of Errors](#types-of-errors)
30. [Error Detection Patterns](#error-detection-patterns)
31. [Logging Patterns](#logging-patterns)
32. [Recovery Strategies](#recovery-strategies)
33. [Retry Logic](#retry-logic)
34. [Graceful Degradation](#graceful-degradation)
35. [Emergency Procedures](#emergency-procedures)
36. [Common Errors and Solutions](#common-errors-and-solutions)
37. [Failsafes and Sanity Checks](#failsafes-and-sanity-checks)
38. [Real Examples from Bots (Error)](#real-examples-from-bots-1)
39. [Testing Error Handling](#testing-error-handling)

### Performance and Timing
40. [Performance and Timing](#performance-and-timing)
41. [Performance Overview](#performance-overview)
42. [Timing Fundamentals](#timing-fundamentals)
43. [Loop Optimization](#loop-optimization)
44. [Query Optimization](#query-optimization)
45. [Caching Strategies](#caching-strategies)
46. [CPU Usage Management](#cpu-usage-management)
47. [Memory Management](#memory-management)
48. [Profiling and Measurement](#profiling-and-measurement)
49. [Common Performance Bottlenecks](#common-performance-bottlenecks)
50. [Optimization Patterns](#optimization-patterns)
51. [Real Examples from Bots (Performance)](#real-examples-from-bots-2)
52. [Performance Checklist](#performance-checklist)

---

## Main Loop and State Machines

**Wiki References**:
- LavishScript Events: `LavishScriptWiki/LavishScript/Events.html`
- ISXEVE onFrame event: `IsxeveWiki/Events.html`
- Frame timing: `LavishScriptWiki/LavishScript/Commands/waitframe.html`

---

## Bot Lifecycle Overview

Every EVE bot follows the same basic lifecycle:

```
┌─────────────┐
│  STARTUP    │ Wait for ISXEVE, Me, MyShip to exist
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ INITIALIZE  │ Load config, create objects, register events
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  MAIN LOOP  │ ◄──┐ Execute bot logic repeatedly
└──────┬──────┘    │
       │           │
       └───────────┘
       │
       ▼
┌─────────────┐
│  SHUTDOWN   │ Save config, cleanup, detach events
└─────────────┘
```

### Minimal Bot Template

```lavish
; ===== MINIMAL BOT TEMPLATE =====

variable bool ScriptRunning = TRUE

function main()
{
    ; 1. WAIT FOR ISXEVE
    while !${ISXEVE.IsReady}
    {
        echo "Waiting for ISXEVE..."
        wait 10
    }

    ; 2. WAIT FOR CHARACTER AND SHIP
    while !${Me(exists)} || !${MyShip(exists)}
    {
        echo "Waiting for character and ship..."
        wait 10
    }

    ; 3. INITIALIZE
    echo "Initializing bot for ${Me.Name}"
    call Initialize

    ; 4. MAIN LOOP
    while ${ScriptRunning}
    {
        if ${Me.InSpace} && ${ISXEVE.IsReady} && ${Me(exists)}
        {
            call BotPulse
        }

        waitframe
    }

    ; 5. SHUTDOWN (automatic via atexit)
}

function Initialize()
{
    echo "Bot initialized"
    ; Load config, create UI, register events, etc.
}

function BotPulse()
{
    ; This runs every frame while in space
    ; Put your bot logic here
}

function atexit()
{
    echo "Bot shutting down"
    ScriptRunning:Set[FALSE]
    ; Save config, cleanup, etc.
}
```

**Key Points**:
- **ALWAYS** wait for ISXEVE.IsReady before accessing game objects
- **ALWAYS** wait for Me and MyShip to exist
- **ALWAYS** check safety conditions before each pulse
- Use `atexit()` for cleanup (called when script ends)
- Use `waitframe` to prevent infinite loop from freezing

---

## Main Loop Architectures

There are three main loop architectures used in EVE bots:

| Pattern | Example | Complexity | Use Case |
|---------|---------|------------|----------|
| Simple Polling Loop | Yamfa | Low | Single-purpose bots (fleet assist, simple miners) |
| Behavior State Machine | EVEBot | Medium | Multi-mode bots (mining, combat, hauling) |
| Event-Driven | Tehbot | High | Complex modular bots with independent systems |

### Comparison

```lavish
; ===== SIMPLE POLLING LOOP =====
while ${ScriptRunning}
{
    if ${Me.InSpace}
    {
        call BotPulse
    }
    wait 5
    waitframe
}

; ===== BEHAVIOR STATE MACHINE =====
while TRUE
{
    if !${Paused} && ${${CurrentBehavior}(exists)}
    {
        call ${CurrentBehavior}.ProcessState
    }
    wait 5
}

; ===== EVENT-DRIVEN =====
; In Initialize:
Event[ISXEVE_onFrame]:AttachAtom[This:Pulse]

; Pulse runs automatically every frame:
method Pulse()
{
    if ${LavishScript.RunningTime} >= ${NextPulseTime}
    {
        ; Do work
        NextPulseTime:Set[${Math.Calc[${LavishScript.RunningTime} + ${PulseInterval}]}]
    }
}
```

---

## Simple Polling Loop Pattern

**Best For**: Single-purpose bots with straightforward logic

**Example**: Yamfa fleet assist bot

### Full Pattern

```lavish
; ===== YAMFA-STYLE SIMPLE LOOP =====

variable bool ScriptRunning = TRUE
variable int LastActionTime = 0
variable int LastCleanupTime = 0

function main()
{
    ; Startup
    while !${ISXEVE.IsReady}
    {
        echo "Waiting for ISXEVE..."
        wait 10
    }

    while !${Me(exists)} || !${MyShip(exists)}
    {
        echo "Waiting for character and ship..."
        wait 10
    }

    ; Initialize
    echo "Initializing bot for ${Me.Name}"
    call Initialize

    ; Main loop
    while ${ScriptRunning}
    {
        ; Safety checks FIRST
        if ${Me.InSpace} && ${ISXEVE.IsReady} && ${Me(exists)}
        {
            call MainPulse
        }

        ; Process queued commands
        if ${QueuedCommands}
        {
            ExecuteQueued
        }

        ; Periodic cleanup (every 5 minutes)
        if ${Math.Calc[${LavishScript.RunningTime} - ${LastCleanupTime}]} > 300000
        {
            call PeriodicCleanup
            LastCleanupTime:Set[${LavishScript.RunningTime}]
        }

        ; Random delay + frame wait
        wait ${Math.Rand[2,6]}
        waitframe
    }
}

function Initialize()
{
    echo "Loading configuration..."
    call LoadConfig

    echo "Loading UI..."
    call LoadUI

    echo "Registering relay events..."
    LavishScript:RegisterEvent[BotCommand]
    Event[BotCommand]:AttachAtom[OnBotCommand]

    echo "Initialization complete"
}

function MainPulse()
{
    ; Check for hostiles (safety first!)
    if ${CheckForHostiles}
    {
        call EmergencyRetreat
        return
    }

    ; Normal operation
    call ProcessTargets
    call ProcessModules
    call ProcessMovement

    ; Update UI periodically (not every frame!)
    if ${Math.Calc[${LavishScript.RunningTime} - ${LastActionTime}]} > 500
    {
        call UpdateUI
        LastActionTime:Set[${LavishScript.RunningTime}]
    }
}

function atexit()
{
    echo "Shutting down..."
    ScriptRunning:Set[FALSE]

    Event[BotCommand]:DetachAtom[OnBotCommand]
    call SaveConfig

    echo "Shutdown complete"
}
```

**Yamfa.iss Real Example** (lines 76-120):

```lavish
function main()
{
    ; Wait for ISXEVE to be ready
    while !${ISXEVE.IsReady}
    {
        echo "Waiting for ISXEVE..."
        wait 10
    }

    ; Wait for character and ship
    while !${Me(exists)} || !${MyShip(exists)}
    {
        echo "Waiting for character and ship..."
        wait 10
    }

    ; Initialize core systems
    echo "Initializing Yamfa for ${Me.Name}"
    call Initialize

    ; Main loop
    while ${ScriptRunning}
    {
        if ${Me.InSpace} && ${ISXEVE.IsReady} && ${Me(exists)}
        {
            call MainPulse
        }

        ; Process queued commands
        if ${QueuedCommands}
        {
            ExecuteQueued
        }

        ; Periodic cleanup every 5 minutes
        if ${Math.Calc[${LavishScript.RunningTime} - ${LastCleanupTime}]} > 300000
        {
            call PeriodicCleanup
            LastCleanupTime:Set[${LavishScript.RunningTime}]
        }

        wait ${Math.Rand[2,6]}
        waitframe
    }
}
```

### Key Features

1. **Single control flow**: One loop, easy to understand
2. **Explicit timing**: You control when things run
3. **Simple debugging**: Easy to add echo statements and trace execution
4. **Periodic tasks**: Use runtime checks for infrequent operations

### When to Use

- ✅ Fleet assist bots (Yamfa)
- ✅ Simple miners (single mode)
- ✅ Passive monitoring bots
- ❌ Multi-behavior bots (use Behavior State Machine instead)
- ❌ Complex modular bots (use Event-Driven instead)

---

## Behavior-Based State Machine

**Best For**: Multi-mode bots that switch between different behaviors

**Example**: EVEBot (mining, hauling, salvaging, missions)

### Concept

Instead of one big MainPulse, create separate "behavior" objects for each bot mode. The main loop delegates to the active behavior.

```
┌──────────────┐
│  Main Loop   │
└──────┬───────┘
       │
       ▼
┌──────────────┐     ┌────────────┐
│ CurrentBeh.  │────▶│  Behavior  │
│ ProcessState │     │   Object   │
└──────────────┘     └─────┬──────┘
                           │
       ┌───────────────────┼───────────────────┐
       ▼                   ▼                   ▼
   ┌────────┐        ┌─────────┐        ┌──────────┐
   │ Mining │        │ Hauling │        │ Salvage  │
   │ Behavior│       │ Behavior│        │ Behavior │
   └────────┘        └─────────┘        └──────────┘
```

### Full Pattern

```lavish
; ===== BEHAVIOR STATE MACHINE PATTERN =====

variable string CurrentBehavior = "Mining"
variable bool Paused = TRUE

; Behavior objects
objectdef obj_MiningBehavior
{
    variable string CurrentState = "IDLE"

    method ProcessState()
    {
        switch ${This.CurrentState}
        {
            case IDLE
                call This.State_Idle
                break
            case MINING
                call This.State_Mining
                break
            case HAULING
                call This.State_Hauling
                break
            default
                echo "Unknown state: ${This.CurrentState}"
                This.CurrentState:Set["IDLE"]
        }
    }

    method State_Idle()
    {
        ; Check if we should start mining
        if ${MyShip.CargoFreeSpace} > 1000 && ${Me.InSpace}
        {
            echo "Transitioning to MINING"
            This.CurrentState:Set["MINING"]
        }
    }

    method State_Mining()
    {
        ; Mine asteroids
        call LockAsteroid
        call ActivateMiners

        ; Check if cargo full
        if ${MyShip.CargoFreeSpace} < 100
        {
            echo "Cargo full, transitioning to HAULING"
            This.CurrentState:Set["HAULING"]
        }
    }

    method State_Hauling()
    {
        ; Return to station
        call DockAtStation
        call UnloadCargo

        echo "Hauling complete, transitioning to IDLE"
        This.CurrentState:Set["IDLE"]
    }
}

objectdef obj_CombatBehavior
{
    variable string CurrentState = "IDLE"

    method ProcessState()
    {
        switch ${This.CurrentState}
        {
            case IDLE
                call This.State_Idle
                break
            case COMBAT
                call This.State_Combat
                break
            case LOOTING
                call This.State_Looting
                break
        }
    }

    method State_Idle()
    {
        ; Find targets
        if ${GetNPCCount} > 0
        {
            This.CurrentState:Set["COMBAT"]
        }
    }

    method State_Combat()
    {
        call EngageTargets

        if ${GetNPCCount} == 0
        {
            This.CurrentState:Set["LOOTING"]
        }
    }

    method State_Looting()
    {
        call LootWrecks
        This.CurrentState:Set["IDLE"]
    }
}

; Global behavior instances
variable(global) obj_MiningBehavior Mining
variable(global) obj_CombatBehavior Combat

function main()
{
    ; Startup
    while !${ISXEVE.IsReady}
    {
        wait 10
    }
    while !${Me(exists)} || !${MyShip(exists)}
    {
        wait 10
    }

    ; Initialize
    echo "Bot loaded - behavior set to ${CurrentBehavior}"

    ; Main loop - delegate to current behavior
    while TRUE
    {
        if !${Paused} && ${${CurrentBehavior}(exists)}
        {
            ; Call the behavior's ProcessState method
            call ${CurrentBehavior}.ProcessState
        }

        ; Random delay between state ticks
        wait ${Math.Calc[5 + (${Math.Rand[400]}/100)]}
    }
}

; Switch behaviors dynamically
function SwitchToCombat()
{
    CurrentBehavior:Set["Combat"]
    echo "Switched to Combat behavior"
}

function SwitchToMining()
{
    CurrentBehavior:Set["Mining"]
    echo "Switched to Mining behavior"
}
```

**EVEBot.iss Real Example** (Branches/Stable/EVEBot.iss, lines 253-263):

```lavish
while TRUE
{
    if !${EVEBot.Paused} && \
        !${EVEBot.Disabled} && \
        ${${Config.Common.CurrentBehavior}(exists)}
    {
        call ${Config.Common.CurrentBehavior}.ProcessState
    }
    ; Random delay between ProcessState calls
    wait ${Math.Calc[5 + (${Math.Rand[399]}/100)]}
}
```

**How EVEBot Loads Behaviors**:
1. Each behavior is in a separate file (Behaviors/obj_Miner.iss, Behaviors/obj_Hauler.iss, etc.)
2. At startup, EVEBot loads all behavior files and creates global instances
3. Config.Common.CurrentBehavior contains the behavior name (e.g., "Miner")
4. Main loop calls: `${Miner}.ProcessState` (dynamic variable reference!)

### Key Features

1. **Separation of concerns**: Each behavior is independent
2. **Easy to add new modes**: Create new behavior object, add to list
3. **Dynamic switching**: Change CurrentBehavior at runtime
4. **Shared infrastructure**: All behaviors use same Ship, Cargo, etc. objects

### When to Use

- ✅ Multi-mode bots (mining + hauling + salvaging)
- ✅ Mission runners (different logic per mission type)
- ✅ Bots that switch between activities
- ❌ Very simple single-purpose bots (overkill)

---

## Event-Driven Architecture

**Best For**: Complex modular bots with independent subsystems

**Example**: Tehbot (combat anomalies, missions, mining, abyssal)

### Concept

Instead of a polling loop, attach to ISXEVE's onFrame event. This fires automatically every game frame, and your Pulse method runs.

```
ISXEVE onFrame Event ──▶ Pulse() method called automatically
                         │
                         ▼
                    Check timer
                         │
                         ▼
                    If time to pulse:
                         │
                         ├──▶ Module 1 pulse
                         ├──▶ Module 2 pulse
                         ├──▶ Module 3 pulse
                         └──▶ Update next pulse time
```

### Full Pattern

```lavish
; ===== EVENT-DRIVEN ARCHITECTURE =====

objectdef obj_Bot
{
    variable bool Paused = TRUE
    variable int NextPulse = 0
    variable int PulseIntervalInMilliseconds = 2000

    method Initialize()
    {
        echo "Attaching to onFrame event..."
        Event[ISXEVE_onFrame]:AttachAtom[This:Pulse]

        ; Initialize subsystems
        TargetManager:Initialize
        ModuleManager:Initialize
        MovementManager:Initialize
    }

    method Shutdown()
    {
        echo "Detaching from onFrame event..."
        Event[ISXEVE_onFrame]:DetachAtom[This:Pulse]

        ; Shutdown subsystems
        TargetManager:Shutdown
        ModuleManager:Shutdown
        MovementManager:Shutdown
    }

    ; This runs EVERY FRAME (60+ times per second!)
    method Pulse()
    {
        ; Immediate return if paused
        if ${This.Paused}
        {
            return
        }

        ; Safety checks
        if !${Me(exists)} || !${MyShip(exists)} || !${ISXEVE.IsReady}
        {
            return
        }

        ; Throttle to desired pulse rate
        if ${LavishScript.RunningTime} >= ${This.NextPulse}
        {
            ; Do actual work
            call This.DoPulse

            ; Schedule next pulse with random jitter
            This.NextPulse:Set[${Math.Calc[${LavishScript.RunningTime} + ${This.PulseIntervalInMilliseconds} + ${Math.Rand[500]}]}]
        }
    }

    method DoPulse()
    {
        ; Pulse all subsystems
        TargetManager:Pulse
        ModuleManager:Pulse
        MovementManager:Pulse

        ; Update UI
        UI:Update
    }

    method Pause()
    {
        This.Paused:Set[TRUE]
        echo "Bot paused"
    }

    method Resume()
    {
        This.Paused:Set[FALSE]
        echo "Bot resumed"
    }
}

; Global bot instance
variable(global) obj_Bot Bot

function main()
{
    ; Startup
    while !${ISXEVE.IsReady}
    {
        wait 10
    }
    while !${Me(exists)} || !${MyShip(exists)}
    {
        wait 10
    }

    ; Initialize
    Bot:Initialize

    ; Main loop does NOTHING except wait!
    ; All work happens in event-driven Pulse()
    while TRUE
    {
        wait 10
    }
}

function atexit()
{
    Bot:Shutdown
}
```

**Tehbot Real Example** (core/obj_Tehbot.iss, lines 8-31):

```lavish
objectdef obj_Tehbot
{
    variable bool Paused = TRUE
    variable int NextPulse
    variable int PulseIntervalInMilliseconds = 2000

    method Initialize()
    {
        Event[ISXEVE_onFrame]:AttachAtom[This:Pulse]
    }

    method Shutdown()
    {
        Event[ISXEVE_onFrame]:DetachAtom[This:Pulse]
    }

    method Pulse()
    {
        if ${Paused}
        {
            return
        }

        if ${LavishScript.RunningTime} >= ${This.NextPulse}
        {
            This.NextPulse:Set[${Math.Calc[${LavishScript.RunningTime} + ${This.PulseIntervalInMilliseconds} + ${Math.Rand[500]}]}]
            ; (Real work happens here in actual Tehbot)
        }
    }
}
```

**Tehbot Main Loop** (Tehbot.iss, lines 211-214):

```lavish
while TRUE
{
    wait 10
}
```

Yes, that's it! All work happens in the event-driven Pulse method.

### Key Features

1. **Event-driven**: Pulse runs automatically every frame
2. **Throttling**: Use timer to control actual work frequency
3. **Modular**: Easy to add independent subsystems
4. **Efficient**: Can respond to events immediately if needed

### When to Use

- ✅ Complex bots with many independent systems
- ✅ Bots that need fast reaction times
- ✅ Modular architecture (drop-in components)
- ❌ Simple bots (unnecessary complexity)
- ❌ Beginners (harder to debug event flow)

---

## State Machine Concepts

A **state machine** is a pattern where your bot can be in exactly ONE state at a time, and transitions between states based on conditions.

### State Machine Diagram

```
              ┌────────┐
              │  IDLE  │◄────────────────┐
              └───┬────┘                 │
                  │                      │
                  │ asteroids found      │
                  ▼                      │
              ┌─────────┐                │
              │ MINING  │                │ cargo unloaded
              └────┬────┘                │
                   │                     │
                   │ cargo full          │
                   ▼                     │
              ┌─────────┐                │
              │ HAULING │────────────────┘
              └─────────┘
```

### State Machine Implementation

```lavish
objectdef obj_Miner
{
    variable string CurrentState = "IDLE"

    method ProcessState()
    {
        ; Dispatch to current state handler
        switch ${This.CurrentState}
        {
            case IDLE
                call This.State_Idle
                break
            case MINING
                call This.State_Mining
                break
            case HAULING
                call This.State_Hauling
                break
            case FLEEING
                call This.State_Fleeing
                break
            default
                echo "ERROR: Unknown state ${This.CurrentState}"
                This.CurrentState:Set["IDLE"]
        }
    }

    method State_Idle()
    {
        echo "State: IDLE - Waiting for conditions"

        ; Check for hostiles FIRST (safety)
        if ${CheckForHostiles}
        {
            This.CurrentState:Set["FLEEING"]
            return
        }

        ; Check if we can mine
        if ${MyShip.CargoFreeSpace} > 1000 && ${Me.InSpace}
        {
            echo "Transitioning IDLE -> MINING"
            This.CurrentState:Set["MINING"]
            return
        }
    }

    method State_Mining()
    {
        echo "State: MINING - Extracting ore"

        ; Safety check EVERY state
        if ${CheckForHostiles}
        {
            EVE:Execute[CmdStopAllModules]
            This.CurrentState:Set["FLEEING"]
            return
        }

        ; Find and lock asteroid
        if ${Me.TargetCount} == 0
        {
            call This.LockAsteroid
        }

        ; Activate miners
        call This.ActivateMiners

        ; Check if cargo full
        if ${MyShip.CargoFreeSpace} < 100
        {
            echo "Cargo full, transitioning MINING -> HAULING"
            EVE:Execute[CmdStopAllModules]
            This.CurrentState:Set["HAULING"]
            return
        }
    }

    method State_Hauling()
    {
        echo "State: HAULING - Returning to station"

        ; Dock if not docked
        if !${Me.InStation}
        {
            call This.DockAtStation
            return
        }

        ; Unload cargo
        call This.UnloadOre

        ; Transition back to mining
        echo "Hauling complete, transitioning HAULING -> IDLE"
        This.CurrentState:Set["IDLE"]
    }

    method State_Fleeing()
    {
        echo "State: FLEEING - Emergency retreat!"

        ; Warp to safe spot
        if ${Me.InSpace}
        {
            call This.WarpToSafe
            return
        }

        ; If safe, return to idle
        if !${CheckForHostiles}
        {
            echo "Safe now, transitioning FLEEING -> IDLE"
            This.CurrentState:Set["IDLE"]
        }
    }
}
```

### Common Bot States

| State | Purpose | Typical Actions |
|-------|---------|----------------|
| STARTUP | Initial bot setup | Wait for ISXEVE, load config, create objects |
| IDLE | Waiting for work | Check conditions, decide next state |
| ACTIVE | Main operation | Mining, combat, hauling, etc. |
| TRAVELING | Moving between locations | Warp, dock, undock, autopilot |
| FLEEING | Emergency escape | Warp away, dock, cloak |
| ERROR | Something went wrong | Log error, attempt recovery, or abort |
| SHUTDOWN | Clean exit | Save config, detach events, cleanup |

---

## State Transition Patterns

### Priority-Based Transitions

**ALWAYS** check safety conditions before normal operations:

```lavish
method ProcessState()
{
    ; 1. SAFETY CHECKS FIRST (highest priority)
    if ${CheckForHostiles}
    {
        This.CurrentState:Set["FLEEING"]
        return
    }

    if ${MyShip.ShieldPct} < 25
    {
        This.CurrentState:Set["FLEEING"]
        return
    }

    ; 2. ERROR CONDITIONS
    if ${MyShip.CargoFreeSpace} < 0  ; Should never happen!
    {
        This.CurrentState:Set["ERROR"]
        return
    }

    ; 3. NORMAL STATE TRANSITIONS
    switch ${This.CurrentState}
    {
        case IDLE
            call This.State_Idle
            break
        case MINING
            call This.State_Mining
            break
    }
}
```

### Hysteresis (Preventing State Flapping)

**Problem**: State changes rapidly back and forth (MINING -> HAULING -> MINING -> HAULING)

**Solution**: Use different thresholds for entering and leaving a state

```lavish
; BAD: No hysteresis
if ${MyShip.CargoFreeSpace} < 100
{
    This.CurrentState:Set["HAULING"]
}

; GOOD: Hysteresis
; Enter hauling when cargo is 90% full (100m³ free)
; Don't leave hauling until completely empty

method State_Mining()
{
    if ${MyShip.CargoFreeSpace} < 100  ; 90% full
    {
        This.CurrentState:Set["HAULING"]
    }
}

method State_Hauling()
{
    if ${MyShip.CargoFreeSpace} >= ${MyShip.CargoCapacity}  ; 100% empty
    {
        This.CurrentState:Set["IDLE"]
    }
}
```

### Timer-Based Transitions

Don't transition until a condition has been true for a minimum duration:

```lavish
variable int MiningStartTime = 0
variable int MIN_MINING_DURATION = 60000  ; 60 seconds

method State_Mining()
{
    ; Record start time
    if ${This.MiningStartTime} == 0
    {
        This.MiningStartTime:Set[${LavishScript.RunningTime}]
    }

    ; Don't allow transition until minimum duration
    if ${MyShip.CargoFreeSpace} < 100
    {
        if ${Math.Calc[${LavishScript.RunningTime} - ${This.MiningStartTime}]} >= ${MIN_MINING_DURATION}
        {
            This.CurrentState:Set["HAULING"]
            This.MiningStartTime:Set[0]
        }
        else
        {
            echo "Cargo full but mining for ${Math.Calc[(${LavishScript.RunningTime} - ${This.MiningStartTime}) / 1000]}s (min ${MIN_MINING_DURATION/1000}s)"
        }
    }
}
```

### Condition Accumulation

Require multiple conditions before transitioning:

```lavish
method State_Idle()
{
    ; Require ALL of these to start mining:
    if ${Me.InSpace} && \
       ${MyShip.CargoFreeSpace} > 1000 && \
       !${CheckForHostiles} && \
       ${GetNearbyAsteroidCount} > 0 && \
       ${MyShip.ShieldPct} > 75
    {
        This.CurrentState:Set["MINING"]
    }
}
```

---

## Timing and Frame Management

### Frame vs Wait vs Waitframe

| Command | Duration | Use Case |
|---------|----------|----------|
| `wait 10` | 1 decisecond (100ms) | Long delays, waiting for actions |
| `waitframe` | 1 frame (~16ms at 60fps) | End of main loop to prevent freeze |
| `Frame` | 1 frame | Same as waitframe (older syntax) |

### Frame Timing Example

```lavish
; ===== FRAME TIMING PATTERNS =====

; PATTERN 1: Wait + Waitframe (Yamfa style)
while ${ScriptRunning}
{
    call BotPulse

    wait ${Math.Rand[2,6]}  ; 200-600ms random delay
    waitframe                ; Also wait for next frame
}

; PATTERN 2: Just Wait (EVEBot style)
while TRUE
{
    call BehaviorPulse

    wait ${Math.Calc[5 + (${Math.Rand[400]}/100)]}  ; 5.00-9.00ms
}

; PATTERN 3: Event-driven with throttle (Tehbot style)
method Pulse()
{
    ; This runs EVERY frame
    if ${LavishScript.RunningTime} >= ${NextPulseTime}
    {
        call DoPulse
        NextPulseTime:Set[${Math.Calc[${LavishScript.RunningTime} + 2000]}]  ; 2 seconds
    }
}
```

### Why Random Jitter?

```lavish
; BAD: Fixed interval
wait 5

; GOOD: Random jitter
wait ${Math.Rand[2,6]}

; BETTER: Percentage-based jitter
variable int baseInterval = 2000
wait ${Math.Calc[${baseInterval} + ${Math.Rand[500]}]}
```

**Reasons for jitter**:
1. **Prevents patterns**: Server can't detect "bot always acts exactly every 2000ms"
2. **Distributes load**: Multiple bots don't all act simultaneously
3. **Appears more human**: Real players don't have perfect timing

### Periodic Tasks

For tasks that should run infrequently (UI updates, cleanup, config saves):

```lavish
variable int LastUIUpdate = 0
variable int LastCleanup = 0
variable int LastConfigSave = 0

function BotPulse()
{
    ; Core logic runs EVERY pulse
    call ProcessTargets
    call ProcessModules

    ; UI update every 500ms
    if ${Math.Calc[${LavishScript.RunningTime} - ${LastUIUpdate}]} > 500
    {
        call UpdateUI
        LastUIUpdate:Set[${LavishScript.RunningTime}]
    }

    ; Cleanup every 5 minutes
    if ${Math.Calc[${LavishScript.RunningTime} - ${LastCleanup}]} > 300000
    {
        call PeriodicCleanup
        LastCleanup:Set[${LavishScript.RunningTime}]
    }

    ; Save config every 10 minutes
    if ${Math.Calc[${LavishScript.RunningTime} - ${LastConfigSave}]} > 600000
    {
        call SaveConfig
        LastConfigSave:Set[${LavishScript.RunningTime}]
    }
}
```

**Yamfa Real Example** (lines 97-120):

```lavish
while ${ScriptRunning}
{
    if ${Me.InSpace} && ${ISXEVE.IsReady} && ${Me(exists)}
    {
        call MainPulse
    }

    ; Process queued commands
    if ${QueuedCommands}
    {
        ExecuteQueued
    }

    ; Periodic cleanup every 5 minutes
    if ${Math.Calc[${LavishScript.RunningTime} - ${LastCleanupTime}]} > 300000
    {
        call PeriodicCleanup
        LastCleanupTime:Set[${LavishScript.RunningTime}]
    }

    wait ${Math.Rand[2,6]}
    waitframe
}
```

---

## Safety-First Architecture

**CRITICAL PRINCIPLE**: Check safety conditions BEFORE normal operations, EVERY pulse.

### Safety Check Order

```
1. Is ISXEVE ready?
2. Does Me exist?
3. Does MyShip exist?
4. Are there hostiles?
5. Is ship damaged?
6. Is cargo full? (for some bots)
7. Is in station when should be in space?
8. Normal operation...
```

### Safety-First Pulse Example

```lavish
function BotPulse()
{
    ; ===== LAYER 1: CRITICAL SAFETY =====

    ; Check ISXEVE ready (should never fail in pulse, but be safe)
    if !${ISXEVE.IsReady}
    {
        echo "WARNING: ISXEVE not ready!"
        return
    }

    ; Check Me exists
    if !${Me(exists)}
    {
        echo "WARNING: Me does not exist!"
        return
    }

    ; Check MyShip exists
    if !${MyShip(exists)}
    {
        echo "WARNING: MyShip does not exist!"
        return
    }

    ; ===== LAYER 2: HOSTILE DETECTION =====

    ; Check for hostiles in local
    if ${CheckForHostilesInLocal}
    {
        echo "ALERT: Hostiles in local!"
        call EmergencyDock
        return
    }

    ; Check for hostiles on grid
    if ${CheckForHostilesOnGrid}
    {
        echo "ALERT: Hostiles on grid!"
        call EmergencyWarpOut
        return
    }

    ; ===== LAYER 3: SHIP INTEGRITY =====

    ; Check shield/armor/hull
    if ${MyShip.ShieldPct} < 25
    {
        echo "WARNING: Shields low (${MyShip.ShieldPct}%)"
        call EmergencyRetreat
        return
    }

    if ${MyShip.ArmorPct} < 50
    {
        echo "WARNING: Armor damage (${MyShip.ArmorPct}%)"
        call EmergencyRetreat
        return
    }

    if ${MyShip.StructurePct} < 75
    {
        echo "CRITICAL: Hull damage (${MyShip.StructurePct}%)"
        call EmergencyRetreat
        return
    }

    ; ===== LAYER 4: LOCATION SANITY =====

    ; Check if in expected location
    if !${Me.InSpace} && ${BotMode.Equal["Mining"]}
    {
        echo "ERROR: Should be in space but in station"
        call Undock
        return
    }

    ; ===== LAYER 5: NORMAL OPERATION =====

    ; All safety checks passed, do normal bot work
    call ProcessTargets
    call ProcessModules
    call ProcessMovement
}
```

### Hostile Detection Pattern

```lavish
; ===== HOSTILE DETECTION =====

variable(global) index:string HostilePilots

function LoadHostileList()
{
    ; Load from config or standings
    HostilePilots:Insert["BadGuy1"]
    HostilePilots:Insert["BadGuy2"]

    ; Or load from standings
    variable index:pilot LocalPilots
    EVE:GetLocalPilots[LocalPilots]

    variable iterator Pilot
    LocalPilots:GetIterator[Pilot]

    if ${Pilot:First(exists)}
        do
        {
            variable float standing = ${Me.GetStanding[${Pilot.Value.CharID}]}
            if ${standing} < 0
            {
                HostilePilots:Insert["${Pilot.Value.Name}"]
            }
        }
        while ${Pilot:Next(exists)}
}

function CheckForHostilesInLocal()
{
    ; Check if any hostile pilots in local
    variable int i
    for (i:Set[1]; ${i} <= ${HostilePilots.Used}; i:Inc)
    {
        if ${Local.Pilot["${HostilePilots[${i}]}"](exists)}
        {
            echo "HOSTILE DETECTED: ${HostilePilots[${i}]}"
            return TRUE
        }
    }

    return FALSE
}

function CheckForHostilesOnGrid()
{
    ; Check entities on grid for hostile players
    variable index:entity Entities
    variable iterator Entity

    EVE:QueryEntities[Entities, "IsPilot"]
    Entities:GetIterator[Entity]

    if ${Entity:First(exists)}
        do
        {
            if ${Entity.Value.IsPC} && !${Entity.Value.IsFriendly}
            {
                variable float standing = ${Me.GetStanding[${Entity.Value.CharID}]}
                if ${standing} < 0
                {
                    echo "HOSTILE ON GRID: ${Entity.Value.Name}"
                    return TRUE
                }
            }
        }
        while ${Entity:Next(exists)}

    return FALSE
}
```

### Emergency Actions

```lavish
function EmergencyDock()
{
    echo "EMERGENCY: Docking immediately"

    ; Stop all modules
    EVE:Execute[CmdStopAllModules]
    wait 5

    ; Clear targets
    EVE:Execute[CmdClearTargets]
    wait 5

    ; Recall drones
    if ${MyShip.UsedDroneBayCapacity} > 0
    {
        EVE:Execute[DroneReturnAndOrbit]
        wait 30  ; Wait for drones
    }

    ; Dock at nearest station
    variable entity Station = ${Entity["GroupID = GROUPID_STATION && IsNearestStation"]}
    if ${Station(exists)}
    {
        echo "Docking at ${Station.Name}"
        Station:Dock
    }
    else
    {
        echo "ERROR: No station found, warping to safe instead"
        call EmergencyWarpOut
    }
}

function EmergencyWarpOut()
{
    echo "EMERGENCY: Warping out"

    ; Stop modules
    EVE:Execute[CmdStopAllModules]
    wait 5

    ; Clear targets
    EVE:Execute[CmdClearTargets]
    wait 5

    ; Warp to random celestial
    variable index:entity Celestials
    variable iterator Celestial

    EVE:QueryEntities[Celestials, "GroupID = GROUPID_PLANET || GroupID = GROUPID_MOON"]
    Celestials:GetIterator[Celestial]

    if ${Celestial:First(exists)}
    {
        ; Pick random celestial
        variable int randomIdx = ${Math.Rand[1,${Celestials.Used}]}
        echo "Warping to ${Celestials.Get[${randomIdx}].Name}"
        Celestials.Get[${randomIdx}]:WarpTo
    }
}
```

---

## Common Patterns from Example Bots

### Yamfa: Simple Master/Slave Fleet Bot

**Architecture**: Simple polling loop with role-based pulse

```lavish
function MainPulse()
{
    if !${IsReady} || !${ISXEVE.IsReady} || !${Me(exists)}
        return

    ; Determine role and call appropriate pulse
    if ${Me.Name.Equal["${MASTER_CHARACTER_NAME}"]}
    {
        call MasterPulse
    }
    else
    {
        call SlavePulse
    }

    ; Update UI periodically
    if ${Math.Calc[${LavishScript.RunningTime} - ${LastActionTime}]} > 500
    {
        call UpdateUI
        LastActionTime:Set[${LavishScript.RunningTime}]
    }
}
```

**Key Pattern**: Role-based dispatch in single pulse function

### EVEBot: Multi-Behavior State Machine

**Architecture**: Behavior-based with ProcessState delegation

```lavish
; Main loop
while TRUE
{
    if !${EVEBot.Paused} && \
        !${EVEBot.Disabled} && \
        ${${Config.Common.CurrentBehavior}(exists)}
    {
        call ${Config.Common.CurrentBehavior}.ProcessState
    }
    wait ${Math.Calc[5 + (${Math.Rand[399]}/100)]}
}

; Example behavior object
objectdef obj_Miner
{
    variable string CurrentState = "Idle"

    method ProcessState()
    {
        ; Process queue
        This.m_StateQueue:ProcessQueue

        ; Safety checks
        if ${Social.IsHostileInLocal}
        {
            call Station.Dock
            return
        }

        ; Execute current state
        switch ${This.CurrentState}
        {
            case Idle
                call This.State_Idle
                break
            case Mining
                call This.State_Mining
                break
            case Hauling
                call This.State_Hauling
                break
        }
    }
}
```

**Key Pattern**: Centralized behavior objects with state machines

### Tehbot: Event-Driven Modular

**Architecture**: Event-driven pulse with independent modules (see [Tehbot](https://github.com/isxGames/Tehbot))

```lavish
; Attach to onFrame event
method Initialize()
{
    Event[ISXEVE_onFrame]:AttachAtom[This:Pulse]
}

; Pulse runs every frame
method Pulse()
{
    if ${Paused}
        return

    if ${LavishScript.RunningTime} >= ${This.NextPulse}
    {
        ; Pulse all modules
        AutoModule:Pulse
        TargetManager:Pulse
        DroneControl:Pulse
        FightOrFlight:Pulse

        This.NextPulse:Set[${Math.Calc[${LavishScript.RunningTime} + ${PulseIntervalInMilliseconds} + ${Math.Rand[500]}]}]
    }
}

; Main loop does nothing
while TRUE
{
    wait 10
}
```

**Key Pattern**: Event-driven with modular independent pulsers

---

## Performance Considerations

### CPU Usage

```lavish
; BAD: No delay, 100% CPU
while TRUE
{
    call BotPulse
    ; No wait! CPU at 100%!
}

; GOOD: Reasonable delay
while TRUE
{
    call BotPulse
    wait 5  ; ~50ms delay
    waitframe
}

; BEST: Adaptive delay based on workload
while TRUE
{
    variable int pulseStart = ${LavishScript.RunningTime}

    call BotPulse

    variable int pulseTime = ${Math.Calc[${LavishScript.RunningTime} - ${pulseStart}]}

    ; If pulse took <10ms, wait longer
    if ${pulseTime} < 10
    {
        wait ${Math.Calc[20 - ${pulseTime}]}
    }

    waitframe
}
```

### Expensive Operation Throttling

Don't do expensive operations every pulse:

```lavish
; BAD: Query all entities every pulse
function BotPulse()
{
    variable index:entity AllEntities
    EVE:QueryEntities[AllEntities]  ; EXPENSIVE! 100-1000 entities
    ; Process...
}

; GOOD: Query once per second
variable int LastEntityQuery = 0
variable index:entity CachedEntities

function BotPulse()
{
    if ${Math.Calc[${LavishScript.RunningTime} - ${LastEntityQuery}]} > 1000
    {
        EVE:QueryEntities[CachedEntities]
        LastEntityQuery:Set[${LavishScript.RunningTime}]
    }

    ; Use cached entities
    ; Process CachedEntities...
}
```

### Index vs Iterator Performance

```lavish
; SLOW: Direct index access in loop (1-indexed!)
variable int i
for (i:Set[1]; ${i} <= ${Targets.Used}; i:Inc)
{
    echo "${Targets.Get[${i}].Name}"
}

; FAST: Iterator
variable iterator Target
Targets:GetIterator[Target]

if ${Target:First(exists)}
    do
    {
        echo "${Target.Value.Name}"
    }
    while ${Target:Next(exists)}
```

### Avoid Repeated Calculations

```lavish
; BAD: Calculate same value multiple times
if ${Math.Calc[${MyShip.CargoCapacity} - ${MyShip.UsedCargoCapacity}]} < 100
{
    echo "Cargo free: ${Math.Calc[${MyShip.CargoCapacity} - ${MyShip.UsedCargoCapacity}]}"
}

; GOOD: Calculate once
variable float cargoFree = ${Math.Calc[${MyShip.CargoCapacity} - ${MyShip.UsedCargoCapacity}]}
if ${cargoFree} < 100
{
    echo "Cargo free: ${cargoFree}"
}

; BEST: Use built-in member
if ${MyShip.CargoFreeSpace} < 100
{
    echo "Cargo free: ${MyShip.CargoFreeSpace}"
}
```

---

## Common Mistakes and Gotchas

### Mistake 1: Infinite Loop Without Frame Wait

```lavish
; BAD: Will freeze InnerSpace!
while TRUE
{
    call BotPulse
    ; No wait or waitframe!
}

; GOOD:
while TRUE
{
    call BotPulse
    waitframe  ; REQUIRED!
}
```

**Why**: LavishScript scripts run in the main UI thread. If you don't yield control with `wait` or `waitframe`, InnerSpace freezes.

### Mistake 2: Not Checking ISXEVE.IsReady

```lavish
; BAD: Crash if ISXEVE not ready
function main()
{
    echo "${Me.Name}"  ; CRASH if ISXEVE not ready!
}

; GOOD:
function main()
{
    while !${ISXEVE.IsReady}
    {
        echo "Waiting for ISXEVE..."
        wait 10
    }

    echo "${Me.Name}"  ; Safe
}
```

### Mistake 3: State Flapping (No Hysteresis)

```lavish
; BAD: Rapid state changes
if ${MyShip.CargoFreeSpace} < 100
    CurrentState:Set["HAULING"]

if ${MyShip.CargoFreeSpace} > 90
    CurrentState:Set["MINING"]

; Result: MINING -> HAULING -> MINING -> HAULING (oscillates at ~95m³)

; GOOD: Wide hysteresis
if ${MyShip.CargoFreeSpace} < 50
    CurrentState:Set["HAULING"]

if ${MyShip.CargoFreeSpace} >= ${MyShip.CargoCapacity}
    CurrentState:Set["MINING"]
```

### Mistake 4: Not Snapshotting Collections

```lavish
; BAD: Collection size changes during iteration
variable int i
for (i:Set[1]; ${i} <= ${Me.Fleet.MemberCount}; i:Inc)
{
    ; If member leaves, MemberCount decreases, loop breaks early!
    echo "${Me.Fleet.Member[${i}].Name}"
}

; GOOD: Snapshot size first
variable int memberCount = ${Me.Fleet.MemberCount}
variable int i
for (i:Set[1]; ${i} <= ${memberCount}; i:Inc)
{
    if ${Me.Fleet.Member[${i}](exists)}
    {
        echo "${Me.Fleet.Member[${i}].Name}"
    }
}
```

### Mistake 5: Forgetting to Detach Events

```lavish
; BAD: Event stays attached after script ends
function main()
{
    Event[ISXEVE_onFrame]:AttachAtom[MyPulse]

    while TRUE
    {
        wait 10
    }

    ; Script ends, but event still attached!
}

; GOOD: Detach in atexit
function main()
{
    Event[ISXEVE_onFrame]:AttachAtom[MyPulse]

    while TRUE
    {
        wait 10
    }
}

function atexit()
{
    Event[ISXEVE_onFrame]:DetachAtom[MyPulse]
}
```

### Mistake 6: Using `wait` in Atoms

```lavish
; BAD: Atoms cannot use wait!
atom MyAtom()
{
    echo "Starting"
    wait 10  ; ERROR: Atoms cannot wait!
    echo "Done"
}

; GOOD: Schedule delayed action
atom MyAtom()
{
    echo "Starting"
    TimedCommand 10 "echo Done"  ; Schedule for 1 second later
}

; OR: Set a flag and check in main loop
atom MyAtom()
{
    NeedToDoSomething:Set[TRUE]
    DoSomethingTime:Set[${Math.Calc[${LavishScript.RunningTime} + 100]}]
}

function MainLoop()
{
    if ${NeedToDoSomething} && ${LavishScript.RunningTime} >= ${DoSomethingTime}
    {
        call DoSomething
        NeedToDoSomething:Set[FALSE]
    }
}
```

### Mistake 7: Not Handling Ship Changes

```lavish
; BAD: Caches ship ID at startup
variable int64 MyShipID = ${MyShip.ID}

function BotPulse()
{
    if ${Entity[${MyShipID}](exists)}  ; Fails if you change ships!
    {
        ; ...
    }
}

; GOOD: Always use ${MyShip}
function BotPulse()
{
    if ${MyShip(exists)}  ; Always current ship
    {
        echo "Ship: ${MyShip.Name}"
    }
}
```

### Mistake 8: Racing Against Game Actions

```lavish
; BAD: Assume instant action
Asteroid:LockTarget
if ${Asteroid.IsLockedTarget}  ; FALSE! Lock takes time
{
    echo "Locked"
}

; GOOD: Wait for lock
Asteroid:LockTarget
wait 20  ; Wait 2 seconds
if ${Asteroid.IsLockedTarget}
{
    echo "Locked"
}

; BETTER: Check in next pulse
function LockAsteroid()
{
    if !${Asteroid.IsLockedTarget} && !${Asteroid.BeingTargeted}
    {
        Asteroid:LockTarget
    }
}

function ActivateMiners()
{
    ; This runs every pulse, so it will activate when lock completes
    if ${Asteroid.IsLockedTarget}
    {
        ; Activate miners...
    }
}
```

---

## Decision Trees for Architecture Choice

### Choose Simple Polling Loop If:

- ✅ Single-purpose bot (one mode)
- ✅ Beginner programmer
- ✅ Simple logic (< 500 lines)
- ✅ Fleet coordination (master/slave roles)
- ✅ Passive monitoring bot

**Example Use Cases**:
- Fleet assist bot (Yamfa)
- Simple miner (one asteroid belt)
- Station trader
- Market monitor

### Choose Behavior State Machine If:

- ✅ Multi-mode bot (mining, hauling, combat, etc.)
- ✅ Intermediate programmer
- ✅ Need to switch between activities
- ✅ Complex state transitions
- ✅ Shared infrastructure between modes

**Example Use Cases**:
- EVEBot (mining + hauling + salvaging)
- Mission runner (different logic per mission)
- Multifunction combat bot

### Choose Event-Driven If:

- ✅ Complex modular architecture
- ✅ Advanced programmer
- ✅ Many independent subsystems
- ✅ Need fast reaction times
- ✅ Drop-in components

**Example Use Cases**:
- Tehbot (combat, missions, mining, abyssal, all modular)
- Comprehensive automation suite
- Framework for multiple bots

---

## Complete Working Example: Mining Bot

Here's a complete, working mining bot using Simple Polling Loop + State Machine:

```lavish
; ===== SIMPLE MINING BOT - COMPLETE EXAMPLE =====

#if !${Extension[ISXEVE](exists)}
    echo "ISXEVE Extension required - exiting"
    Script:End
#endif

; ===== GLOBAL VARIABLES =====
variable bool ScriptRunning = TRUE
variable string CurrentState = "IDLE"
variable int LastUIUpdate = 0
variable int LastCleanup = 0
variable int64 CurrentAsteroid = 0
variable string HomeStation = "Jita IV - Moon 4 - Caldari Navy Assembly Plant"

; ===== MAIN ENTRY POINT =====
function main()
{
    ; Wait for ISXEVE
    while !${ISXEVE.IsReady}
    {
        echo "Waiting for ISXEVE..."
        wait 10
    }

    ; Wait for character and ship
    while !${Me(exists)} || !${MyShip(exists)}
    {
        echo "Waiting for character and ship..."
        wait 10
    }

    ; Initialize
    echo "SimpleMiner initialized for ${Me.Name}"

    ; Main loop
    while ${ScriptRunning}
    {
        ; Safety checks first
        if ${Me(exists)} && ${MyShip(exists)} && ${ISXEVE.IsReady}
        {
            call BotPulse
        }

        ; Periodic cleanup
        if ${Math.Calc[${LavishScript.RunningTime} - ${LastCleanup}]} > 300000
        {
            call PeriodicCleanup
            LastCleanup:Set[${LavishScript.RunningTime}]
        }

        wait ${Math.Rand[2,6]}
        waitframe
    }
}

; ===== MAIN PULSE =====
function BotPulse()
{
    ; Safety checks
    if ${CheckForHostiles}
    {
        echo "HOSTILES DETECTED! Docking..."
        CurrentState:Set["FLEEING"]
    }

    if ${MyShip.ShieldPct} < 30
    {
        echo "Low shields! Docking..."
        CurrentState:Set["FLEEING"]
    }

    ; State machine
    switch ${CurrentState}
    {
        case IDLE
            call State_Idle
            break
        case UNDOCKING
            call State_Undocking
            break
        case TRAVELING
            call State_Traveling
            break
        case MINING
            call State_Mining
            break
        case HAULING
            call State_Hauling
            break
        case FLEEING
            call State_Fleeing
            break
    }

    ; Update UI every 500ms
    if ${Math.Calc[${LavishScript.RunningTime} - ${LastUIUpdate}]} > 500
    {
        call UpdateUI
        LastUIUpdate:Set[${LavishScript.RunningTime}]
    }
}

; ===== STATE: IDLE =====
function State_Idle()
{
    echo "State: IDLE"

    ; If in station, undock
    if ${Me.InStation}
    {
        CurrentState:Set["UNDOCKING"]
        return
    }

    ; If in space with cargo space, start mining
    if ${Me.InSpace} && ${MyShip.CargoFreeSpace} > 1000
    {
        CurrentState:Set["MINING"]
        return
    }

    ; If cargo full, haul
    if ${MyShip.CargoFreeSpace} < 100
    {
        CurrentState:Set["HAULING"]
        return
    }
}

; ===== STATE: UNDOCKING =====
function State_Undocking()
{
    echo "State: UNDOCKING"

    if ${Me.InStation}
    {
        EVE:Execute[CmdExitStation]
        wait 100  ; Wait 10 seconds for undock
    }

    if ${Me.InSpace}
    {
        CurrentState:Set["TRAVELING"]
    }
}

; ===== STATE: TRAVELING =====
function State_Traveling()
{
    echo "State: TRAVELING to belt"

    ; Find nearest asteroid belt
    variable entity Belt = ${Entity["GroupID = GROUPID_ASTEROIDBELT && IsNearestAsteroidBelt"]}

    if !${Belt(exists)}
    {
        echo "ERROR: No asteroid belt found!"
        CurrentState:Set["IDLE"]
        return
    }

    ; If not at belt, warp
    if ${Belt.Distance} > 150000
    {
        echo "Warping to ${Belt.Name}"
        Belt:WarpTo[0]
        wait 100  ; Wait for warp
        return
    }

    ; At belt, start mining
    if ${Belt.Distance} < 150000
    {
        echo "Arrived at belt"
        CurrentState:Set["MINING"]
    }
}

; ===== STATE: MINING =====
function State_Mining()
{
    echo "State: MINING (Cargo: ${MyShip.UsedCargoCapacity}/${MyShip.CargoCapacity})"

    ; Check if cargo full
    if ${MyShip.CargoFreeSpace} < 50
    {
        echo "Cargo full, hauling"
        EVE:Execute[CmdStopAllModules]
        CurrentState:Set["HAULING"]
        return
    }

    ; Lock asteroid if not locked
    if ${Me.TargetCount} == 0
    {
        call LockNearestAsteroid
        return
    }

    ; Activate miners
    call ActivateMiners
}

function LockNearestAsteroid()
{
    variable index:entity Asteroids
    variable iterator Asteroid

    ; Find asteroids
    EVE:QueryEntities[Asteroids, "CategoryID = CATEGORYID_ASTEROID"]
    Asteroids:GetIterator[Asteroid]

    ; Lock closest
    if ${Asteroid:First(exists)}
    {
        echo "Locking ${Asteroid.Value.Name}"
        Asteroid.Value:LockTarget
        CurrentAsteroid:Set[${Asteroid.Value.ID}]
    }
}

function ActivateMiners()
{
    if ${CurrentAsteroid} == 0
        return

    if !${Entity[${CurrentAsteroid}](exists)}
    {
        echo "Asteroid depleted"
        CurrentAsteroid:Set[0]
        return
    }

    ; Activate all mining lasers on asteroid
    variable int i
    for (i:Set[1]; ${i} <= ${MyShip.ModuleCount}; i:Inc)
    {
        variable item module = ${MyShip.Module[${i}]}

        if ${module.ToItem.Group.Equal["Mining Laser"]}
        {
            if !${module.IsActive} && !${module.IsOnline}
            {
                echo "Activating ${module.ToItem.Name}"
                module:Activate[${CurrentAsteroid}]
                wait 5
            }
        }
    }
}

; ===== STATE: HAULING =====
function State_Hauling()
{
    echo "State: HAULING"

    ; If in space, dock
    if ${Me.InSpace}
    {
        variable entity Station = ${Entity["GroupID = GROUPID_STATION && Name = \"${HomeStation}\""]}

        if ${Station(exists)}
        {
            if ${Station.Distance} > 150000
            {
                echo "Warping to ${Station.Name}"
                Station:WarpTo[0]
                wait 100
            }
            else
            {
                echo "Docking at ${Station.Name}"
                Station:Dock
                wait 100
            }
        }
        else
        {
            echo "ERROR: Station not found!"
            CurrentState:Set["IDLE"]
        }

        return
    }

    ; If in station, unload
    if ${Me.InStation}
    {
        call UnloadOre
        CurrentState:Set["IDLE"]
    }
}

function UnloadOre()
{
    echo "Unloading ore to station hangar"

    ; ⚠️ Modern Inventory API (July 2020+)
    ; Open inventory window
    if !${EVEWindow[Inventory](exists)}
    {
        EVE:Execute[CmdOpenInventory]
        wait 20
    }

    ; Open item hangar
    EVE:Execute[OpenHangarFloor]
    wait 20

    ; Move all ore (CategoryID 25)
    variable index:item CargoItems
    EVEWindow[Inventory].Child[ShipCargo]:GetItems[CargoItems]

    variable iterator Item
    CargoItems:GetIterator[Item]

    if ${Item:First(exists)}
        do
        {
            if ${Item.Value.CategoryID} == 25  ; Asteroid category
            {
                echo "Moving ${Item.Value.Quantity} x ${Item.Value.Name}"
                Item.Value:MoveTo[MyStationHangar,${Item.Value.Quantity}]
                wait 10
            }
        }
        while ${Item:Next(exists)}

    echo "Unload complete"
}

; ===== STATE: FLEEING =====
function State_Fleeing()
{
    echo "State: FLEEING - Emergency dock!"

    EVE:Execute[CmdStopAllModules]
    wait 5

    variable entity Station = ${Entity["GroupID = GROUPID_STATION && IsNearestStation"]}

    if ${Station(exists)}
    {
        if ${Station.Distance} > 150000
        {
            Station:WarpTo[0]
            wait 100
        }
        else
        {
            Station:Dock
            wait 100
        }
    }

    if ${Me.InStation}
    {
        echo "Safe in station"
        CurrentState:Set["IDLE"]
    }
}

; ===== SAFETY FUNCTIONS =====
function CheckForHostiles()
{
    ; Simple check: any PC that's not me
    variable index:entity Entities
    variable iterator Entity

    EVE:QueryEntities[Entities, "IsPC"]
    Entities:GetIterator[Entity]

    if ${Entity:First(exists)}
        do
        {
            if ${Entity.Value.CharID} != ${Me.CharID}
            {
                echo "Hostile player detected: ${Entity.Value.Name}"
                return TRUE
            }
        }
        while ${Entity:Next(exists)}

    return FALSE
}

; ===== UTILITY FUNCTIONS =====
function UpdateUI()
{
    ; Update UI here if you have one
    echo "[${CurrentState}] Cargo: ${MyShip.UsedCargoCapacity}/${MyShip.CargoCapacity} m³"
}

function PeriodicCleanup()
{
    echo "Periodic cleanup"
    ; Clear old target references
    if ${CurrentAsteroid} > 0 && !${Entity[${CurrentAsteroid}](exists)}
    {
        CurrentAsteroid:Set[0]
    }
}

; ===== SHUTDOWN =====
function atexit()
{
    echo "SimpleMiner shutting down"
    ScriptRunning:Set[FALSE]
    EVE:Execute[CmdStopAllModules]
}
```

**To run this bot**:
1. Save as `SimpleMiner.iss`
2. In InnerSpace console: `runscript SimpleMiner`
3. Bot will undock, warp to belt, mine until cargo full, return to station, repeat

---

## Summary

### Key Takeaways

1. **Three main loop patterns**:
   - Simple Polling Loop (easiest, single-purpose)
   - Behavior State Machine (medium, multi-mode)
   - Event-Driven (hardest, modular)

2. **Always follow bot lifecycle**:
   - STARTUP → INITIALIZE → MAIN LOOP → SHUTDOWN

3. **Safety-first architecture**:
   - Check hostiles BEFORE normal operations
   - Check ship integrity BEFORE activating modules
   - Check ISXEVE.IsReady, Me(exists), MyShip(exists) EVERY pulse

4. **State machines prevent spaghetti code**:
   - One state at a time
   - Clear transition conditions
   - Use hysteresis to prevent flapping

5. **Timing is critical**:
   - ALWAYS use `waitframe` or `wait` in loops
   - Add random jitter to appear human
   - Throttle expensive operations

6. **Common mistakes to avoid**:
   - Infinite loops without frame wait
   - Not checking ISXEVE.IsReady
   - State flapping
   - Racing against game actions
   - Forgetting to detach events

---

## Decision Making and Logic Patterns

**Wiki References**:
- Entity queries: `IsxeveWiki/DataType/eve.html#QueryEntities`
- Boolean logic: `LavishScriptWiki/LavishScript/Data_Sequences.html`
- Math operations: `LavishScriptWiki/TopLevelObjects/Math.html`

**⚠️ API Note:** Some examples use simplified cargo checks (`MyShip.CargoFreeSpace`) to illustrate decision logic. For production code, use modern inventory API: `EVEWindow[Inventory].Child[ShipCargo].FreeSpace`. See Files 13 and 15 for modern cargo handling.

---

## Decision-Making Overview

Bot intelligence comes from **making good decisions** based on **available information**. The decision-making process follows this flow:

```
┌──────────────┐
│ Gather Data  │ Query entities, check ship status, read environment
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Filter Data  │ Remove invalid/irrelevant options
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Prioritize   │ Rank remaining options by importance
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Choose Best  │ Select highest priority option
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Execute      │ Perform chosen action
└──────────────┘
```

### Key Principles

1. **Safety First**: Always check hostile/damage conditions before normal operations
2. **Priority Order**: Process high-priority options before low-priority
3. **Validity Checking**: Ensure chosen option still exists/is valid before acting
4. **State Awareness**: Different decisions in different states (idle vs active vs fleeing)
5. **Performance**: Don't re-query/re-calculate unnecessarily

---

## Target Selection Patterns

Target selection is the most common decision-making task for bots. The pattern is:

1. **Query** all potential targets
2. **Filter** to valid targets only
3. **Prioritize** by importance/threat/value
4. **Select** highest priority target
5. **Validate** before locking

### Pattern 1: Simple Closest Target

```lavish
function GetClosestNPC()
{
    variable index:entity NPCs
    variable iterator NPC

    ; Query all NPCs within range
    EVE:QueryEntities[NPCs, "CategoryID = CATEGORYID_ENTITY && IsNPC && !IsMoribund && Distance < ${MyShip.MaxTargetRange}"]

    NPCs:GetIterator[NPC]

    ; QueryEntities automatically sorts by distance
    ; First result is closest
    if ${NPC:First(exists)}
    {
        return ${NPC.Value.ID}
    }

    return 0  ; No target found
}
```

**Use Case**: Simple mining bots, basic combat bots

### Pattern 2: Priority List Matching

```lavish
; ===== PRIORITY TARGET PATTERN =====
; Based on EVEBot obj_Targets.iss

variable(global) index:string PriorityTargets
variable(global) index:string NormalTargets

function LoadTargetPriorities()
{
    ; Priority targets (scramblers, webifiers, jammers) - kill FIRST
    PriorityTargets:Insert["Dire Pithi Arrogator"]  ; web/scram
    PriorityTargets:Insert["Dire Pithi Imputor"]    ; web/scram
    PriorityTargets:Insert["Dire Pithi Infiltrator"]; web/scram
    PriorityTargets:Insert["Dire Pithi Saboteur"]   ; jamming
    PriorityTargets:Insert["Guardian Agent"]        ; web/scram
    PriorityTargets:Insert["Guardian Scout"]        ; web/scram

    ; Normal targets - kill after priority
    NormalTargets:Insert["Pithi Arrogator"]
    NormalTargets:Insert["Pithi Infiltrator"]
    NormalTargets:Insert["Guristas Imputor"]
}

function GetBestCombatTarget()
{
    variable index:entity Entities
    variable iterator Entity

    ; Query all hostile NPCs
    EVE:QueryEntities[Entities, "CategoryID = CATEGORYID_ENTITY && IsNPC && !IsMoribund && Distance < ${MyShip.MaxTargetRange}"]
    Entities:GetIterator[Entity]

    if !${Entity:First(exists)}
        return 0

    ; PASS 1: Look for priority targets
    do
    {
        variable int i
        for (i:Set[1]; ${i} <= ${PriorityTargets.Used}; i:Inc)
        {
            if ${Entity.Value.Name.Find["${PriorityTargets[${i}]}"]}
            {
                echo "PRIORITY TARGET: ${Entity.Value.Name}"
                return ${Entity.Value.ID}
            }
        }
    }
    while ${Entity:Next(exists)}

    ; PASS 2: No priority targets, take closest normal target
    Entity:First

    return ${Entity.Value.ID}
}
```

**EVEBot Real Example** (obj_Targets.iss, lines 92-115):

```lavish
PriorityTargets:Insert["Dire Pithi Destructor"]
PriorityTargets:Insert["Dire Pithi Wrecker"]
PriorityTargets:Insert["Dire Pithi Plunderer"]
PriorityTargets:Insert["Factory Defense Battery"] 		/* web/scram */
PriorityTargets:Insert["Dire Pithi Arrogator"] 		/* web/scram */
PriorityTargets:Insert["Dire Pithi Despoiler"] 		/* Jamming */
PriorityTargets:Insert["Dire Pithi Imputor"] 		/* web/scram */
PriorityTargets:Insert["Dire Pithi Infiltrator"] 	/* web/scram */
PriorityTargets:Insert["Dire Pithi Invader"] 		/* web/scram */
PriorityTargets:Insert["Dire Pithi Saboteur"] 		/* Jamming */
```

### Pattern 3: Weighted Scoring

```lavish
; ===== WEIGHTED SCORING PATTERN =====

function GetBestTargetScored()
{
    variable index:entity Entities
    variable iterator Entity

    ; Query all NPCs
    EVE:QueryEntities[Entities, "CategoryID = CATEGORYID_ENTITY && IsNPC && !IsMoribund && Distance < ${MyShip.MaxTargetRange}"]
    Entities:GetIterator[Entity]

    if !${Entity:First(exists)}
        return 0

    variable int64 BestTarget = 0
    variable float BestScore = 0

    ; Score each entity
    do
    {
        variable float score = 0

        ; Distance (closer = higher score)
        score:Set[${Math.Calc[10000 / ${Entity.Value.Distance}]}]

        ; Shield %  (lower shields = higher score, prioritize damaged)
        if ${Entity.Value.ShieldPct} < 50
        {
            score:Inc[${Math.Calc[100 - ${Entity.Value.ShieldPct}]}]
        }

        ; Ship size (frigates = 100, cruisers = 50, battleships = 10)
        switch ${Entity.Value.GroupID}
        {
            case ${GROUPID_FRIGATES}
                score:Inc[100]
                break
            case ${GROUPID_CRUISERS}
                score:Inc[50]
                break
            case ${GROUPID_BATTLESHIPS}
                score:Inc[10]
                break
        }

        ; Priority target bonus
        if ${IsPriorityTarget["${Entity.Value.Name}"]}
        {
            score:Inc[1000]  ; Huge bonus!
        }

        ; Already locked penalty (don't re-pick same target)
        if ${Entity.Value.IsLockedTarget} || ${Entity.Value.BeingTargeted}
        {
            score:Dec[500]
        }

        ; Update best
        if ${score} > ${BestScore}
        {
            BestScore:Set[${score}]
            BestTarget:Set[${Entity.Value.ID}]
        }
    }
    while ${Entity:Next(exists)}

    if ${BestTarget} > 0
    {
        echo "Best target: ${Entity[${BestTarget}].Name} (score: ${BestScore})"
    }

    return ${BestTarget}
}

function IsPriorityTarget(string name)
{
    variable int i
    for (i:Set[1]; ${i} <= ${PriorityTargets.Used}; i:Inc)
    {
        if ${name.Find["${PriorityTargets[${i}]}"]}
        {
            return TRUE
        }
    }

    return FALSE
}
```

### Pattern 4: Query String Building

**Tehbot Style** - Build complex query strings dynamically

```lavish
; ===== DYNAMIC QUERY STRING BUILDING =====
; Based on Tehbot obj_TargetList.iss

objectdef obj_TargetSelector
{
    variable index:string QueryStringList
    variable index:entity ResultEntities
    variable set ExcludeTargetID
    variable set ExcludeNameParts
    variable int MaxRange = 20000
    variable int MinRange = 0

    method ClearQueryStrings()
    {
        QueryStringList:Clear
    }

    method AddQueryString(string QueryString)
    {
        QueryStringList:Insert["${QueryString.Escape}"]
    }

    method AddAllNPCs()
    {
        variable string QueryString = "CategoryID = CATEGORYID_ENTITY && IsNPC && !IsMoribund && !("

        ; Exclude specific groups (concord, convoy, structures, etc.)
        QueryString:Concat["GroupID = GROUP_CONCORDDRONE ||"]
        QueryString:Concat["GroupID = GROUP_CONVOYDRONE ||"]
        QueryString:Concat["GroupID = GROUP_CONVOY ||"]
        QueryString:Concat["GroupID = GROUP_LARGECOLLIDABLEOBJECT ||"]
        QueryString:Concat["GroupID = GROUP_LARGECOLLIDABLESHIP ||"]
        QueryString:Concat["GroupID = GROUP_SPAWNCONTAINER ||"]
        QueryString:Concat["GroupID = GROUP_DEADSPACEOVERSEERSSTRUCTURE ||"]
        QueryString:Concat["GroupID = GROUP_LARGECOLLIDABLESTRUCTURE"]
        QueryString:Concat[")"]

        This:AddQueryString["${QueryString.Escape}"]
    }

    method AddTargetingMe()
    {
        This:AddQueryString["Distance < 150000 && IsTargetingMe && IsNPC && !IsMoribund"]
    }

    method AddPCTargetingMe()
    {
        This:AddQueryString["Distance < 150000 && IsTargetingMe && !IsFleetMember && IsPC && !IsMoribund"]
    }

    method AddExcludeByID(int64 EntityID)
    {
        ExcludeTargetID:Add[${EntityID}]
    }

    method AddExcludeByNamePart(string NamePart)
    {
        ExcludeNameParts:Add["${NamePart}"]
    }

    method UpdateTargetList()
    {
        ResultEntities:Clear

        variable iterator QueryIterator
        QueryStringList:GetIterator[QueryIterator]

        ; Execute each query
        if ${QueryIterator:First(exists)}
            do
            {
                variable index:entity TempEntities
                variable iterator Entity

                EVE:QueryEntities[TempEntities, "${QueryIterator.Value.Escape}"]
                TempEntities:GetIterator[Entity]

                ; Add to result list with filtering
                if ${Entity:First(exists)}
                    do
                    {
                        ; Filter by range
                        if ${Entity.Value.Distance} < ${MinRange} || ${Entity.Value.Distance} > ${MaxRange}
                            continue

                        ; Filter by excluded IDs
                        if ${ExcludeTargetID.Contains[${Entity.Value.ID}]}
                            continue

                        ; Filter by excluded name parts
                        variable bool excludeByName = FALSE
                        variable iterator NamePart
                        ExcludeNameParts:GetIterator[NamePart]

                        if ${NamePart:First(exists)}
                            do
                            {
                                if ${Entity.Value.Name.Find["${NamePart.Value}"]}
                                {
                                    excludeByName:Set[TRUE]
                                    break
                                }
                            }
                            while ${NamePart:Next(exists)}

                        if ${excludeByName}
                            continue

                        ; Add to result
                        ResultEntities:Insert[${Entity.Value}]
                    }
                    while ${Entity:Next(exists)}
            }
            while ${QueryIterator:Next(exists)}
    }

    member:int64 GetBestTarget()
    {
        call This.UpdateTargetList

        ; Results are sorted by distance by default
        if ${ResultEntities.Used} > 0
        {
            return ${ResultEntities.Get[1].ID}
        }

        return 0
    }
}
```

**Tehbot Real Example** (obj_TargetList.iss, lines 130-177):

```lavish
method AddAllNPCs()
{
    variable string QueryString="CategoryID = CATEGORYID_ENTITY && IsNPC && !IsMoribund && !("

    ; Abyssal switch for ignoring the really distant cans if we aren't using an MTU
    if !${Abyssal.Config.UseMTU}
    {
        QueryString:Concat["TypeID = 49663 ||"]
        QueryString:Concat["TypeID = 49662 ||"]
        QueryString:Concat["TypeID = 49661 ||"]
    }
    ;Exclude Groups here
    QueryString:Concat["GroupID = GROUP_CONCORDDRONE ||"]
    QueryString:Concat["GroupID = GROUP_CONVOYDRONE ||"]
    QueryString:Concat["GroupID = GROUP_CONVOY ||"]
    QueryString:Concat["GroupID = GROUP_LARGECOLLIDABLEOBJECT ||"]
    QueryString:Concat["GroupID = GROUP_LARGECOLLIDABLESHIP ||"]
    QueryString:Concat["GroupID = GROUP_SPAWNCONTAINER ||"]
    QueryString:Concat["GroupID = CATEGORYID_ORE ||"]
    QueryString:Concat["GroupID = GROUP_DEADSPACEOVERSEERSSTRUCTURE ||"]
    QueryString:Concat["GroupID = GROUP_LARGECOLLIDABLESTRUCTURE ||"]
    ; ... more exclusions ...

    This:AddQueryString["${QueryString.Escape}"]
}
```

---

## Priority Systems

Priority systems allow bots to handle multiple concerns simultaneously.

### Pattern 1: Layered Priority

```lavish
; ===== LAYERED PRIORITY PATTERN =====

function GetNextAction()
{
    ; LAYER 1: SAFETY (highest priority)
    if ${CheckForHostiles}
    {
        echo "ACTION: Flee (hostiles detected)"
        return "FLEE"
    }

    if ${MyShip.ShieldPct} < 30
    {
        echo "ACTION: Dock (low shields)"
        return "DOCK"
    }

    ; LAYER 2: CRITICAL OPERATIONS
    if ${MyShip.CargoFreeSpace} < 50
    {
        echo "ACTION: Haul (cargo full)"
        return "HAUL"
    }

    ; LAYER 3: NORMAL OPERATIONS
    if ${GetNearbyNPCCount} > 0 && ${BotMode.Equal["Combat"]}
    {
        echo "ACTION: Engage (NPCs found)"
        return "COMBAT"
    }

    if ${GetNearbyAsteroidCount} > 0 && ${BotMode.Equal["Mining"]}
    {
        echo "ACTION: Mine (asteroids found)"
        return "MINE"
    }

    ; LAYER 4: IDLE
    echo "ACTION: Idle (nothing to do)"
    return "IDLE"
}
```

### Pattern 2: Numeric Priority

```lavish
; ===== NUMERIC PRIORITY PATTERN =====

objectdef obj_Task
{
    variable string Name
    variable int Priority  ; Higher = more important
    variable string Action

    method Initialize(string name, int priority, string action)
    {
        This.Name:Set["${name}"]
        This.Priority:Set[${priority}]
        This.Action:Set["${action}"]
    }
}

variable(global) index:obj_Task TaskList

function AddTask(string name, int priority, string action)
{
    TaskList:Insert[${name},${priority},${action}]
}

function GetHighestPriorityTask()
{
    if ${TaskList.Used} == 0
        return ""

    variable iterator Task
    TaskList:GetIterator[Task]

    variable string BestTask = ""
    variable int BestPriority = 0

    if ${Task:First(exists)}
        do
        {
            if ${Task.Value.Priority} > ${BestPriority}
            {
                BestPriority:Set[${Task.Value.Priority}]
                BestTask:Set["${Task.Value.Action}"]
            }
        }
        while ${Task:Next(exists)}

    return "${BestTask}"
}

; Usage example
function BotPulse()
{
    TaskList:Clear

    ; Add tasks with priorities
    if ${CheckForHostiles}
    {
        call AddTask "Flee" 1000 "FLEE"
    }

    if ${MyShip.ShieldPct} < 30
    {
        call AddTask "Dock" 900 "DOCK"
    }

    if ${MyShip.CargoFreeSpace} < 50
    {
        call AddTask "Haul" 500 "HAUL"
    }

    if ${GetNearbyNPCCount} > 0
    {
        call AddTask "Combat" 300 "COMBAT"
    }

    ; Get highest priority task
    variable string nextAction = ${GetHighestPriorityTask}

    switch ${nextAction}
    {
        case FLEE
            call EmergencyFlee
            break
        case DOCK
            call DockForRepairs
            break
        case HAUL
            call ReturnToStation
            break
        case COMBAT
            call EngageCombat
            break
    }
}
```

### Pattern 3: Priority Queues

```lavish
; ===== PRIORITY QUEUE PATTERN =====

variable(global) index:int64 PriorityTargets
variable(global) index:int64 NormalTargets
variable(global) index:int64 LowPriorityTargets

function CategorizeTargets()
{
    PriorityTargets:Clear
    NormalTargets:Clear
    LowPriorityTargets:Clear

    variable index:entity Entities
    variable iterator Entity

    EVE:QueryEntities[Entities, "CategoryID = CATEGORYID_ENTITY && IsNPC && !IsMoribund"]
    Entities:GetIterator[Entity]

    if ${Entity:First(exists)}
        do
        {
            ; Categorize by name matching
            if ${Entity.Value.Name.Find["Dire"]} || ${Entity.Value.Name.Find["Guardian"]}
            {
                PriorityTargets:Insert[${Entity.Value.ID}]
            }
            elseif ${Entity.Value.Name.Find["Hauler"]} || ${Entity.Value.Name.Find["Transport"]}
            {
                LowPriorityTargets:Insert[${Entity.Value.ID}]
            }
            else
            {
                NormalTargets:Insert[${Entity.Value.ID}]
            }
        }
        while ${Entity:Next(exists)}

    echo "Targets - Priority: ${PriorityTargets.Used}, Normal: ${NormalTargets.Used}, Low: ${LowPriorityTargets.Used}"
}

function GetNextTarget()
{
    call CategorizeTargets

    ; Take from priority queue first
    if ${PriorityTargets.Used} > 0
    {
        return ${PriorityTargets.Get[1]}
    }

    ; Then normal queue
    if ${NormalTargets.Used} > 0
    {
        return ${NormalTargets.Get[1]}
    }

    ; Finally low priority queue
    if ${LowPriorityTargets.Used} > 0
    {
        return ${LowPriorityTargets.Get[1]}
    }

    return 0
}
```

---

## Condition Checking Patterns

### Pattern 1: Guard Clauses

```lavish
; ===== GUARD CLAUSE PATTERN =====

function ProcessCombat()
{
    ; Guard clauses - early returns for invalid states
    if !${ISXEVE.IsReady}
    {
        echo "ISXEVE not ready"
        return
    }

    if !${Me.InSpace}
    {
        echo "Not in space"
        return
    }

    if ${Me.InWarp}
    {
        echo "In warp, waiting"
        return
    }

    if ${MyShip.ToEntity.IsCloaked}
    {
        echo "Cloaked, cannot engage"
        return
    }

    ; All checks passed, do combat
    call FindAndEngageTargets
}
```

### Pattern 2: Compound Conditions

```lavish
; ===== COMPOUND CONDITION PATTERN =====

; BAD: Nested ifs (hard to read)
if ${Me.InSpace}
{
    if ${ISXEVE.IsReady}
    {
        if !${Me.InWarp}
        {
            if ${MyShip.ShieldPct} > 50
            {
                call ProcessCombat
            }
        }
    }
}

; GOOD: Compound condition with AND operator
if ${Me.InSpace} && \
   ${ISXEVE.IsReady} && \
   !${Me.InWarp} && \
   ${MyShip.ShieldPct} > 50
{
    call ProcessCombat
}

; GOOD: Stored in boolean variable
variable bool CanEngage = ${Me.InSpace} && ${ISXEVE.IsReady} && !${Me.InWarp} && ${MyShip.ShieldPct} > 50

if ${CanEngage}
{
    call ProcessCombat
}
```

### Pattern 3: Condition Functions

```lavish
; ===== CONDITION FUNCTION PATTERN =====

function IsSafeToOperate()
{
    ; Check all safety conditions
    if !${ISXEVE.IsReady}
        return FALSE

    if !${Me(exists)} || !${MyShip(exists)}
        return FALSE

    if ${CheckForHostiles}
        return FALSE

    if ${MyShip.ShieldPct} < 30
        return FALSE

    if ${Me.InWarp}
        return FALSE

    return TRUE
}

function CanMine()
{
    if !${IsSafeToOperate}
        return FALSE

    if !${Me.InSpace}
        return FALSE

    if ${MyShip.CargoFreeSpace} < 100
        return FALSE

    if ${GetNearbyAsteroidCount} == 0
        return FALSE

    return TRUE
}

function CanEngage()
{
    if !${IsSafeToOperate}
        return FALSE

    if !${Me.InSpace}
        return FALSE

    if ${Me.TargetedByCount} > 0 && ${MyShip.ToEntity.GroupID} != GROUPID_BATTLESHIPS
        return FALSE  ; Being targeted and not in battleship

    if ${GetNearbyNPCCount} == 0
        return FALSE

    return TRUE
}

; Usage
function BotPulse()
{
    if !${IsSafeToOperate}
    {
        echo "Not safe, idling"
        return
    }

    if ${CanMine}
    {
        call ProcessMining
    }
    elseif ${CanEngage}
    {
        call ProcessCombat
    }
    else
    {
        call ProcessIdle
    }
}
```

---

## Decision Trees

Decision trees help visualize complex decision-making logic.

### Example: Mining Bot Decision Tree

```
                        ┌──────────────┐
                        │ Bot Pulse    │
                        └──────┬───────┘
                               │
                ┌──────────────┴──────────────┐
                │ Hostiles detected?          │
                └──────┬──────────────┬───────┘
                  YES  │              │ NO
                       ▼              ▼
                ┌──────────┐   ┌─────────────┐
                │ FLEE     │   │ Low shields?│
                └──────────┘   └──────┬──────┘
                                 YES  │  NO
                                      ▼      ▼
                               ┌──────────┐ ┌──────────────┐
                               │ DOCK     │ │ Cargo full?  │
                               └──────────┘ └──────┬───────┘
                                              YES │ NO
                                                  ▼  ▼
                                           ┌──────────┐ ┌────────────┐
                                           │ HAUL     │ │ In space?  │
                                           └──────────┘ └──────┬─────┘
                                                          YES │ NO
                                                              ▼  ▼
                                                       ┌─────────┐ ┌────────┐
                                                       │ MINE    │ │ UNDOCK │
                                                       └─────────┘ └────────┘
```

### Implementation

```lavish
function MiningBotDecisionTree()
{
    ; Level 1: Safety check
    if ${CheckForHostiles}
    {
        echo "Decision: FLEE (hostiles detected)"
        return "FLEE"
    }

    ; Level 2: Ship integrity
    if ${MyShip.ShieldPct} < 30
    {
        echo "Decision: DOCK (low shields)"
        return "DOCK"
    }

    ; Level 3: Cargo status
    if ${MyShip.CargoFreeSpace} < 50
    {
        echo "Decision: HAUL (cargo full)"
        return "HAUL"
    }

    ; Level 4: Location
    if !${Me.InSpace}
    {
        echo "Decision: UNDOCK (in station)"
        return "UNDOCK"
    }

    ; Level 5: Normal operation
    echo "Decision: MINE (all checks passed)"
    return "MINE"
}
```

### Example: Combat Bot Decision Tree

```
                          ┌─────────────┐
                          │ Bot Pulse   │
                          └──────┬──────┘
                                 │
                    ┌────────────┴────────────┐
                    │ Hostiles on grid?       │
                    └────────┬────────────┬───┘
                        NO   │            │ YES
                             ▼            ▼
                      ┌──────────┐  ┌─────────────────┐
                      │ Wrecks?  │  │ Low HP or outnumbered? │
                      └────┬─────┘  └────────┬────────────┬───┘
                       YES │ NO        YES   │            │ NO
                           ▼  ▼              ▼            ▼
                     ┌─────────┐ ┌────┐ ┌──────┐  ┌────────────┐
                     │ LOOT    │ │IDLE│ │RETREAT│  │ Priority?  │
                     └─────────┘ └────┘ └──────┘  └─────┬──────┘
                                                     YES │ NO
                                                         ▼  ▼
                                                  ┌──────────┐ ┌────────┐
                                                  │ KILL PRI │ │ KILL   │
                                                  └──────────┘ └────────┘
```

### Implementation

```lavish
function CombatBotDecisionTree()
{
    variable int npcCount = ${GetNearbyNPCCount}

    ; No NPCs - check for loot or idle
    if ${npcCount} == 0
    {
        if ${GetNearbyWreckCount} > 0
        {
            echo "Decision: LOOT (no enemies, wrecks present)"
            return "LOOT"
        }

        echo "Decision: IDLE (no enemies, no wrecks)"
        return "IDLE"
    }

    ; NPCs present - assess threat
    variable int hpPercent = ${MyShip.ShieldPct}
    if ${MyShip.ArmorPct} < ${hpPercent}
    {
        hpPercent:Set[${MyShip.ArmorPct}]
    }

    ; Check if outnumbered or low HP
    if ${hpPercent} < 40 || ${npcCount} > 5
    {
        echo "Decision: RETREAT (${hpPercent}% HP, ${npcCount} enemies)"
        return "RETREAT"
    }

    ; Check for priority targets
    if ${GetPriorityTargetCount} > 0
    {
        echo "Decision: KILL_PRIORITY (${GetPriorityTargetCount} priority targets)"
        return "KILL_PRIORITY"
    }

    echo "Decision: KILL (${npcCount} enemies, no priorities)"
    return "KILL"
}
```

---

## Weighted Scoring Systems

For complex decisions with multiple factors, weighted scoring provides nuanced selection.

### Example: Best Asteroid Selection

```lavish
; ===== WEIGHTED ASTEROID SCORING =====

function GetBestAsteroid()
{
    variable index:entity Asteroids
    variable iterator Asteroid

    EVE:QueryEntities[Asteroids, "CategoryID = CATEGORYID_ASTEROID"]
    Asteroids:GetIterator[Asteroid]

    if !${Asteroid:First(exists)}
        return 0

    variable int64 BestAsteroid = 0
    variable float BestScore = 0

    do
    {
        variable float score = 0

        ; Distance factor (closer = better)
        ; Score: 0-100 based on distance (0m = 100, 50km = 0)
        variable float distanceScore = ${Math.Calc[100 - (${Asteroid.Value.Distance} / 500)]}
        if ${distanceScore} < 0
        {
            distanceScore:Set[0]
        }
        score:Inc[${distanceScore}]

        ; Ore type factor
        variable string oreName = "${Asteroid.Value.Name}"

        if ${oreName.Find["Veldspar"]}
        {
            score:Inc[10]  ; Low value
        }
        elseif ${oreName.Find["Scordite"]}
        {
            score:Inc[20]
        }
        elseif ${oreName.Find["Pyroxeres"]}
        {
            score:Inc[30]
        }
        elseif ${oreName.Find["Plagioclase"]}
        {
            score:Inc[40]
        }
        elseif ${oreName.Find["Kernite"]}
        {
            score:Inc[80]  ; High value
        }
        elseif ${oreName.Find["Jaspet"]}
        {
            score:Inc[100]  ; Highest value
        }

        ; Quantity factor (larger = better)
        variable int quantity = ${Asteroid.Value.Quantity}
        variable float quantityScore = ${Math.Calc[${quantity} / 1000]}
        if ${quantityScore} > 50
        {
            quantityScore:Set[50]  ; Cap at 50
        }
        score:Inc[${quantityScore}]

        ; Already locked penalty
        if ${Asteroid.Value.IsLockedTarget}
        {
            score:Dec[500]  ; Heavy penalty
        }

        ; Update best
        if ${score} > ${BestScore}
        {
            BestScore:Set[${score}]
            BestAsteroid:Set[${Asteroid.Value.ID}]
        }
    }
    while ${Asteroid:Next(exists)}

    if ${BestAsteroid} > 0
    {
        echo "Best asteroid: ${Entity[${BestAsteroid}].Name} (score: ${BestScore.Precision[2]})"
    }

    return ${BestAsteroid}
}
```

### Example: Yamfa Master Target Selection

**Real Yamfa Pattern** (Yamfa.iss, lines 210-310):

```lavish
function MasterPulse()
{
    variable int CurrentTime = ${Math.Calc[${LavishScript.RunningTime} / 100]}
    variable index:entity Targets
    variable iterator Target
    variable index:int64 CurrentTargets

    ; Get all currently locked/locking targets
    EVE:QueryEntities[Targets]
    Targets:GetIterator[Target]

    if ${Target:First(exists)}
        do
        {
            if ${Target.Value.IsLockedTarget} || ${Target.Value.BeingTargeted}
            {
                CurrentTargets:Insert[${Target.Value.ID}]
                ; Update last seen time for this target
                MasterTargetTimers:Set[${Target.Value.ID}, ${CurrentTime}]
            }
        }
        while ${Target:Next(exists)}

    ; Apply hysteresis - keep targets for MASTER_HOLD_TIME after they disappear
    variable iterator ExistingTarget
    MasterTargetSet:GetIterator[ExistingTarget]

    if ${ExistingTarget:First(exists)}
        do
        {
            variable int64 TargetID = ${MasterTargetSet.Get[${ExistingTarget.Key}]}
            variable int LastSeen = ${MasterTargetTimers.Get[${TargetID}]}

            ; If target still exists in current set, keep it
            variable bool StillExists = FALSE
            variable int i
            for (i:Set[1] ; ${i} <= ${CurrentTargets.Used} ; i:Inc)
            {
                if ${CurrentTargets.Get[${i}]} == ${TargetID}
                {
                    StillExists:Set[TRUE]
                    break
                }
            }

            ; If not in current set but within hold time AND still locked, keep it
            if !${StillExists}
            {
                if ${Math.Calc[${CurrentTime} - ${LastSeen}]} < ${MASTER_HOLD_TIME} && \
                   ${Entity[${TargetID}](exists)} && \
                   (${Entity[${TargetID}].IsLockedTarget} || ${Entity[${TargetID}].BeingTargeted})
                {
                    CurrentTargets:Insert[${TargetID}]
                }
            }
        }
        while ${ExistingTarget:Next(exists)}

    ; Update master target set
    MasterTargetSet:Clear
    variable int j
    for (j:Set[1] ; ${j} <= ${CurrentTargets.Used} ; j:Inc)
    {
        MasterTargetSet:Insert[${CurrentTargets.Get[${j}]}]
    }

    ; Get active target
    MasterActiveTarget:Set[0]
    if ${Me.ActiveTarget(exists)}
        MasterActiveTarget:Set[${Me.ActiveTarget.ID}]

    ; Relay to fleet...
}
```

**Key Pattern**: Yamfa uses **hysteresis with timers** to prevent rapid target changes. Targets are kept for 700ms after unlocking to allow slaves to catch up.

---

## Real Examples from Bots

### EVEBot: Priority Target System

**File**: `obj_Targets.iss`

**Pattern**: Explicit priority lists with name matching

```lavish
; Priority targets will be targeted (and killed)
; before other targets, they often do special things
; which we cant use (scramble / web / damp / etc)

PriorityTargets:Insert["Dire Pithi Destructor"]
PriorityTargets:Insert["Dire Pithi Wrecker"]
PriorityTargets:Insert["Dire Pithi Plunderer"]
PriorityTargets:Insert["Factory Defense Battery"]  /* web/scram */
PriorityTargets:Insert["Dire Pithi Arrogator"]    /* web/scram */
PriorityTargets:Insert["Dire Pithi Despoiler"]    /* Jamming */

; Special targets will trigger an alert
; This should include haulers / faction / officers

SpecialTargets:Insert["Estamel Tharchon"]  ; Guristas officer
SpecialTargets:Insert["Kaikka Peunato"]
SpecialTargets:Insert["Thon Eney"]
SpecialTargets:Insert["Vepas Minimala"]

SpecialTargets:Insert["Dread Guristas"]   ; Faction spawns
SpecialTargets:Insert["Shadow Serpentis"]
SpecialTargets:Insert["True Sansha"]
SpecialTargets:Insert["Dark Blood"]

SpecialTargets:Insert["Courier"]  ; Haulers (low threat, valuable loot)
SpecialTargets:Insert["Ferrier"]
SpecialTargets:Insert["Gatherer"]
SpecialTargets:Insert["Harvester"]
```

**Usage Pattern**:

```lavish
function GetNextCombatTarget()
{
    variable index:entity Entities
    variable iterator Entity

    EVE:QueryEntities[Entities, "IsNPC && !IsMoribund"]
    Entities:GetIterator[Entity]

    ; Pass 1: Check for priority targets
    if ${Entity:First(exists)}
        do
        {
            if ${IsPriorityTarget["${Entity.Value.Name}"]}
            {
                return ${Entity.Value.ID}
            }
        }
        while ${Entity:Next(exists)}

    ; Pass 2: Check for special targets (alerts)
    Entity:First
    do
    {
        if ${IsSpecialTarget["${Entity.Value.Name}"]}
        {
            call AlertSpecialTarget "${Entity.Value.Name}"
            return ${Entity.Value.ID}
        }
    }
    while ${Entity:Next(exists)}

    ; Pass 3: Take closest normal target
    Entity:First
    return ${Entity.Value.ID}
}
```

### Tehbot: Query String Exclusion System

**File**: `obj_TargetList.iss`

**Pattern**: Build complex exclusion query string, then filter results

```lavish
method AddAllNPCs()
{
    variable string QueryString="CategoryID = CATEGORYID_ENTITY && IsNPC && !IsMoribund && !("

    ; Exclude specific groups
    QueryString:Concat["GroupID = GROUP_CONCORDDRONE ||"]
    QueryString:Concat["GroupID = GROUP_CONVOYDRONE ||"]
    QueryString:Concat["GroupID = GROUP_CONVOY ||"]
    QueryString:Concat["GroupID = GROUP_LARGECOLLIDABLEOBJECT ||"]
    QueryString:Concat["GroupID = GROUP_SPAWNCONTAINER ||"]
    QueryString:Concat["GroupID = GROUP_DEADSPACEOVERSEERSSTRUCTURE"]
    QueryString:Concat[")"]

    This:AddQueryString["${QueryString.Escape}"]
}

method AddTargetExceptionByID(int64 ID)
{
    ExcludeTargetID:Add[${ID}]

    ; Remove from existing lists
    variable iterator RemoveIterator
    TargetList:GetIterator[RemoveIterator]
    if ${RemoveIterator:First(exists)}
    {
        do
        {
            if ${RemoveIterator.Value.ID.Equal[${ID}]}
            {
                TargetList:Remove[${RemoveIterator.Key}]
            }
        }
        while ${RemoveIterator:Next(exists)}
    }

    ; Unlock if locked
    if ${Entity[${ID}].IsLockedTarget}
    {
        Entity[${ID}]:UnlockTarget
    }
}
```

---

## Mining Decision Logic

### Decision Flow

```
1. Is safe to mine? (No hostiles, sufficient HP)
   NO → Flee/Dock
   YES ↓

2. Have cargo space?
   NO → Return to station
   YES ↓

3. Have locked asteroid?
   NO → Find and lock best asteroid
   YES ↓

4. Is asteroid depleted?
   YES → Unlock, go to step 3
   NO ↓

5. Are miners active?
   NO → Activate miners
   YES → Continue mining
```

### Implementation

```lavish
function ProcessMining()
{
    ; Safety check
    if !${IsSafeToMine}
    {
        call EmergencyDock
        return
    }

    ; Cargo check
    if ${MyShip.CargoFreeSpace} < 100
    {
        call ReturnToStation
        return
    }

    ; Asteroid lock check
    if ${Me.TargetCount} == 0
    {
        variable int64 bestAsteroid = ${GetBestAsteroid}
        if ${bestAsteroid} > 0
        {
            Entity[${bestAsteroid}]:LockTarget
        }
        return
    }

    ; Get current locked asteroid
    variable int64 currentAsteroid = ${Me.GetTargets.Get[1].ID}

    ; Check if depleted
    if !${Entity[${currentAsteroid}](exists)} || ${Entity[${currentAsteroid}].Distance} > 20000
    {
        echo "Asteroid depleted or out of range, unlocking"
        Entity[${currentAsteroid}]:UnlockTarget
        return
    }

    ; Activate miners
    call ActivateMinersOn ${currentAsteroid}
}

function ActivateMinersOn(int64 asteroidID)
{
    variable int i
    for (i:Set[1]; ${i} <= ${MyShip.ModuleCount}; i:Inc)
    {
        variable item module = ${MyShip.Module[${i}]}

        if ${module.ToItem.Group.Equal["Mining Laser"]} || \
           ${module.ToItem.Group.Equal["Strip Miner"]}
        {
            if !${module.IsActive} && ${module.IsOnline}
            {
                echo "Activating ${module.ToItem.Name}"
                module:Activate[${asteroidID}]
                wait 10
            }
        }
    }
}
```

---

## Combat Decision Logic

### Decision Flow

```
1. Is safe to engage? (No overwhelming force, sufficient HP)
   NO → Retreat
   YES ↓

2. Are priority targets present?
   YES → Lock and kill priority targets first
   NO ↓

3. Are targets within range?
   NO → Approach closer target
   YES ↓

4. Lock targets (up to max)

5. Activate weapons on best target

6. All targets dead?
   YES → Look for loot
   NO → Return to step 2
```

### Implementation

```lavish
function ProcessCombat()
{
    ; Safety check
    if !${IsSafeToCombat}
    {
        call EmergencyRetreat
        return
    }

    ; Get priority targets
    variable index:int64 priorityTargets
    call GetPriorityTargets priorityTargets

    if ${priorityTargets.Used} > 0
    {
        echo "Engaging ${priorityTargets.Used} priority targets"
        call EngageTargets priorityTargets
        return
    }

    ; Get normal targets
    variable index:int64 normalTargets
    call GetNormalTargets normalTargets

    if ${normalTargets.Used} > 0
    {
        echo "Engaging ${normalTargets.Used} normal targets"
        call EngageTargets normalTargets
        return
    }

    ; No targets, check for loot
    if ${GetNearbyWreckCount} > 0
    {
        call ProcessLooting
    }
}

function GetPriorityTargets(index:int64 outTargets)
{
    outTargets:Clear

    variable index:entity Entities
    variable iterator Entity

    EVE:QueryEntities[Entities, "IsNPC && !IsMoribund && Distance < ${MyShip.MaxTargetRange}"]
    Entities:GetIterator[Entity]

    if ${Entity:First(exists)}
        do
        {
            if ${IsPriorityTarget["${Entity.Value.Name}"]}
            {
                outTargets:Insert[${Entity.Value.ID}]
            }
        }
        while ${Entity:Next(exists)}
}

function EngageTargets(index:int64 targetIDs)
{
    ; Lock targets
    variable int i
    for (i:Set[1]; ${i} <= ${targetIDs.Used} && ${Me.TargetCount} < ${Me.MaxLockedTargets}; i:Inc)
    {
        variable int64 targetID = ${targetIDs.Get[${i}]}

        if !${Entity[${targetID}].IsLockedTarget} && !${Entity[${targetID}].BeingTargeted}
        {
            Entity[${targetID}]:LockTarget
            wait 5
        }
    }

    ; Activate weapons on first locked target
    variable int64 activeTarget = 0
    for (i:Set[1]; ${i} <= ${targetIDs.Used}; i:Inc)
    {
        if ${Entity[${targetIDs.Get[${i}]}].IsLockedTarget}
        {
            activeTarget:Set[${targetIDs.Get[${i}]}]
            break
        }
    }

    if ${activeTarget} > 0
    {
        call ActivateWeaponsOn ${activeTarget}
    }
}
```

---

## Movement Decision Logic

### Decision: Approach, Orbit, or Keep Range?

```lavish
function GetMovementCommand(int64 targetID)
{
    if ${targetID} == 0 || !${Entity[${targetID}](exists)}
        return "NONE"

    variable entity target = ${Entity[${targetID}]}
    variable float distance = ${target.Distance}
    variable float optimalRange = ${GetWeaponOptimalRange}

    ; Too far - approach
    if ${distance} > ${Math.Calc[${optimalRange} * 1.5]}
    {
        echo "Movement: APPROACH (${distance}m > ${optimalRange}m optimal)"
        return "APPROACH"
    }

    ; Too close - keep range
    if ${distance} < ${Math.Calc[${optimalRange} * 0.5]}
    {
        echo "Movement: KEEPRANGE (${distance}m < ${optimalRange}m optimal)"
        return "KEEPRANGE"
    }

    ; In range - orbit
    echo "Movement: ORBIT (${distance}m ≈ ${optimalRange}m optimal)"
    return "ORBIT"
}

function ExecuteMovementCommand(string command, int64 targetID)
{
    if !${Entity[${targetID}](exists)}
        return

    switch ${command}
    {
        case APPROACH
            Entity[${targetID}]:Approach
            break
        case ORBIT
            Entity[${targetID}]:Orbit[${GetWeaponOptimalRange}]
            break
        case KEEPRANGE
            Entity[${targetID}]:KeepAtRange[${GetWeaponOptimalRange}]
            break
    }
}
```

---

## Common Patterns

### Pattern 1: Fallback Chain

```lavish
; Try preferred option, fall back to alternatives

function GetTarget()
{
    variable int64 target

    ; Try: Priority targets
    target:Set[${GetPriorityTarget}]
    if ${target} > 0
        return ${target}

    ; Try: Active target (don't switch unnecessarily)
    if ${Me.ActiveTarget(exists)}
        return ${Me.ActiveTarget.ID}

    ; Try: Already locked target
    if ${Me.TargetCount} > 0
        return ${Me.GetTargets.Get[1].ID}

    ; Try: Closest NPC
    target:Set[${GetClosestNPC}]
    if ${target} > 0
        return ${target}

    ; No target found
    return 0
}
```

### Pattern 2: Time-Based Decisions

```lavish
; Make different decisions based on time elapsed

variable int MiningStartTime = 0
variable int MIN_MINING_DURATION = 120000  ; 2 minutes

function ShouldReturnToStation()
{
    ; Don't return if mining less than 2 minutes
    if ${MiningStartTime} > 0
    {
        variable int miningDuration = ${Math.Calc[${LavishScript.RunningTime} - ${MiningStartTime}]}

        if ${miningDuration} < ${MIN_MINING_DURATION}
        {
            echo "Mining for ${miningDuration}ms, minimum ${MIN_MINING_DURATION}ms"
            return FALSE
        }
    }

    ; Cargo full
    if ${MyShip.CargoFreeSpace} < 50
    {
        return TRUE
    }

    return FALSE
}
```

### Pattern 3: Threshold with Hysteresis

```lavish
; Different thresholds for entering/exiting state

variable string ShieldState = "NORMAL"

function CheckShieldState()
{
    ; Enter CRITICAL when shields drop below 30%
    if ${ShieldState.Equal["NORMAL"]} && ${MyShip.ShieldPct} < 30
    {
        echo "Shield state: NORMAL -> CRITICAL"
        ShieldState:Set["CRITICAL"]
        call DockForRepairs
        return
    }

    ; Exit CRITICAL when shields recover above 80%
    if ${ShieldState.Equal["CRITICAL"]} && ${MyShip.ShieldPct} > 80
    {
        echo "Shield state: CRITICAL -> NORMAL"
        ShieldState:Set["NORMAL"]
        return
    }
}
```

---

## Performance Considerations

### 1. Cache Expensive Queries

```lavish
; BAD: Query every pulse
function BotPulse()
{
    variable int npcCount = ${GetNPCCount}  ; Queries entities every time
    if ${npcCount} > 0
    {
        call ProcessCombat
    }
}

; GOOD: Cache and refresh periodically
variable int CachedNPCCount = 0
variable int LastNPCQuery = 0

function BotPulse()
{
    ; Refresh every 1 second
    if ${Math.Calc[${LavishScript.RunningTime} - ${LastNPCQuery}]} > 1000
    {
        CachedNPCCount:Set[${GetNPCCount}]
        LastNPCQuery:Set[${LavishScript.RunningTime}]
    }

    if ${CachedNPCCount} > 0
    {
        call ProcessCombat
    }
}
```

### 2. Short-Circuit Evaluation

```lavish
; BAD: All conditions evaluated
if ${ExpensiveCheck1} && ${ExpensiveCheck2} && ${ExpensiveCheck3}
{
    ; ...
}

; GOOD: Cheap checks first, short-circuit on failure
if ${Me.InSpace} && ${ISXEVE.IsReady} && ${ExpensiveCheck1} && ${ExpensiveCheck2}
{
    ; Me.InSpace and ISXEVE.IsReady are cheap, checked first
    ; If they fail, expensive checks never run
}
```

### 3. Limit Search Scope

```lavish
; BAD: Query all entities
EVE:QueryEntities[Entities, "IsNPC"]

; GOOD: Query only nearby entities
EVE:QueryEntities[Entities, "IsNPC && Distance < ${MyShip.MaxTargetRange}"]

; BETTER: Query specific category/group
EVE:QueryEntities[Entities, "CategoryID = CATEGORYID_ENTITY && IsNPC && Distance < ${MyShip.MaxTargetRange}"]
```

---

## Summary

### Key Takeaways

1. **Decision-making is layered**:
   - Safety checks FIRST (hostiles, damage)
   - Critical operations SECOND (cargo full, repairs needed)
   - Normal operations THIRD (combat, mining)
   - Idle LAST

2. **Target selection patterns**:
   - Closest (simple, fast)
   - Priority lists (explicit threat ranking)
   - Weighted scoring (nuanced, multi-factor)
   - Query string building (flexible, filterable)

3. **Priority systems**:
   - Layered priority (safety > critical > normal > idle)
   - Numeric priority (explicit scoring)
   - Priority queues (separate high/normal/low)

4. **Condition checking**:
   - Guard clauses (early return on invalid state)
   - Compound conditions (AND/OR logic)
   - Condition functions (reusable checks)

5. **Performance**:
   - Cache expensive queries
   - Short-circuit evaluation (cheap checks first)
   - Limit search scope (distance filters)

---

## Error Handling and Recovery

**Wiki References**:
- LavishScript echo: `LavishScriptWiki/LavishScript/Commands/echo.html`
- File I/O for logging: `LavishScriptWiki/TopLevelObjects/File.html`
- Script timing: `LavishScriptWiki/TopLevelObjects/LavishScript.html`

---

## Error Handling Overview

Errors in EVE bots fall into two categories:

1. **Code Errors**: Bugs, logic flaws, invalid operations
2. **Game State Errors**: Unexpected game conditions, UI failures, network issues

### Error Handling Principles

1. **Detect Early**: Check for error conditions before they cause problems
2. **Log Everything**: Record what happened, when, and why
3. **Fail Gracefully**: Don't crash, degrade functionality instead
4. **Recover Automatically**: Attempt to fix the problem without user intervention
5. **Alert When Necessary**: Notify user of critical failures

### Error Handling Flow

```
┌──────────────┐
│ Operation    │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Detect Error?│──NO──▶ Continue normally
└──────┬───────┘
   YES │
       ▼
┌──────────────┐
│ Log Error    │ Record what happened
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Can Recover? │──NO──▶ Alert user, graceful degradation
└──────┬───────┘
   YES │
       ▼
┌──────────────┐
│ Attempt Fix  │ Retry, reset state, workaround
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Verify Fixed?│──NO──▶ Log failure, try alternative
└──────┬───────┘
   YES │
       ▼
┌──────────────┐
│ Continue     │
└──────────────┘
```

---

## Types of Errors

### Code Errors

| Error Type | Cause | Example |
|------------|-------|---------|
| Null Reference | Accessing non-existent object | `${Entity[999999].Name}` when entity doesn't exist |
| Index Out of Bounds | Accessing invalid index | `${Index.Get[0]}` (indices are 1-based!) |
| Type Mismatch | Wrong data type | Passing string to function expecting int |
| Division by Zero | Math error | `${Math.Calc[100 / ${SomeVar}]}` when SomeVar = 0 |
| Infinite Loop | No wait/waitframe | `while TRUE { echo "stuck!" }` |

### Game State Errors

| Error Type | Cause | Example |
|------------|-------|---------|
| ISXEVE Not Ready | Extension loading | `${Me.Name}` called before ISXEVE ready |
| UI Window Closed | Player closed window | Opening cargo when player closed it |
| Entity Despawned | Target died/warped | Activating module on dead target |
| Inventory Locked | Another operation in progress | Moving items during active drag |
| Network Disconnect | Lost connection | Game disconnected mid-operation |
| Warp Scrambled | Player movement restricted | Trying to warp while scrambled |
| Module Offline | Ship fitting changed | Activating module that's offline |

---

## Error Detection Patterns

### Pattern 1: Existence Checking

```lavish
; ===== EXISTENCE CHECKING PATTERN =====

; BAD: Assume entity exists
function BadTargetLock(int64 entityID)
{
    Entity[${entityID}]:LockTarget  ; CRASH if entity doesn't exist!
}

; GOOD: Check existence first
function GoodTargetLock(int64 entityID)
{
    if !${Entity[${entityID}](exists)}
    {
        echo "ERROR: Entity ${entityID} does not exist, cannot lock"
        return FALSE
    }

    Entity[${entityID}]:LockTarget
    return TRUE
}

; BETTER: Check existence AND validity
function BestTargetLock(int64 entityID)
{
    ; Check exists
    if !${Entity[${entityID}](exists)}
    {
        echo "ERROR: Entity ${entityID} does not exist"
        return FALSE
    }

    ; Check distance
    if ${Entity[${entityID}].Distance} > ${MyShip.MaxTargetRange}
    {
        echo "ERROR: Entity ${entityID} out of range (${Entity[${entityID}].Distance}m)"
        return FALSE
    }

    ; Check not already locked
    if ${Entity[${entityID}].IsLockedTarget}
    {
        echo "WARNING: Entity ${entityID} already locked"
        return TRUE  ; Not an error, just already done
    }

    ; All checks passed
    Entity[${entityID}]:LockTarget
    return TRUE
}
```

### Pattern 2: Sanity Checking

```lavish
; ===== SANITY CHECKING PATTERN =====

function MoveCargoToHangar(int quantity)
{
    ; Sanity check: Quantity
    if ${quantity} <= 0
    {
        echo "ERROR: Invalid quantity ${quantity}"
        return FALSE
    }

    ; Sanity check: ISXEVE ready
    if !${ISXEVE.IsReady}
    {
        echo "ERROR: ISXEVE not ready"
        return FALSE
    }

    ; Sanity check: In station
    if !${Me.InStation}
    {
        echo "ERROR: Not in station, cannot access hangar"
        return FALSE
    }

    ; Sanity check: Inventory window accessible
    if !${EVEWindow[Inventory](exists)}
    {
        EVE:Execute[OpenInventory]
        wait 20

        if !${EVEWindow[Inventory](exists)}
        {
            echo "ERROR: Cannot open inventory window"
            return FALSE
        }
    }

    ; All checks passed, proceed
    variable index:item Items
    EVEWindow[Inventory].Child[ShipCargo]:GetItems[Items]
    ; ... move items ...

    return TRUE
}
```

### Pattern 3: State Validation

```lavish
; ===== STATE VALIDATION PATTERN =====

function ValidateBotState()
{
    variable string errorMessage = ""

    ; Check ISXEVE
    if !${ISXEVE.IsReady}
    {
        errorMessage:Concat["ISXEVE not ready; "]
    }

    ; Check character
    if !${Me(exists)}
    {
        errorMessage:Concat["Me does not exist; "]
    }

    ; Check ship
    if !${MyShip(exists)}
    {
        errorMessage:Concat["MyShip does not exist; "]
    }

    ; Check location
    if !${Me.InSpace} && !${Me.InStation}
    {
        errorMessage:Concat["Invalid location; "]
    }

    ; If any errors, log and return false
    if ${errorMessage.Length} > 0
    {
        echo "ERROR: Bot state invalid: ${errorMessage}"
        return FALSE
    }

    return TRUE
}

function BotPulse()
{
    ; Validate state every pulse
    if !${ValidateBotState}
    {
        echo "Bot state invalid, waiting..."
        return
    }

    ; State valid, proceed with bot logic
    call ProcessBotLogic
}
```

---

## Logging Patterns

### Pattern 1: Simple Echo Logging

```lavish
; ===== SIMPLE ECHO LOGGING =====

function Log(string level, string message)
{
    echo "${Time} [${level}] ${message}"
}

; Usage
call Log "INFO" "Bot started"
call Log "WARNING" "Low shields: ${MyShip.ShieldPct}%"
call Log "ERROR" "Failed to lock target"
call Log "CRITICAL" "Emergency dock initiated"
```

### Pattern 2: File Logging

```lavish
; ===== FILE LOGGING PATTERN =====

variable(global) file LogFile

function InitializeLogging()
{
    variable string logPath = "${Script.CurrentDirectory}/Logs/BotLog_${Time.Date}_${Time.Time}.txt"

    ; Create log file
    LogFile:SetFilename["${logPath}"]
    LogFile:Open[append]

    call Log "INFO" "=== Bot Started ==="
    call Log "INFO" "Character: ${Me.Name}"
    call Log "INFO" "Session: ${Session}"
}

function Log(string level, string message)
{
    variable string logLine = "${Time.Date} ${Time.Time} [${level}] ${message}"

    ; Echo to console
    echo "${logLine}"

    ; Write to file
    if ${LogFile.IsOpen}
    {
        LogFile:Write["${logLine}\n"]
    }
}

function CloseLogging()
{
    call Log "INFO" "=== Bot Stopped ==="

    if ${LogFile.IsOpen}
    {
        LogFile:Close[]
    }
}

function atexit()
{
    call CloseLogging
}
```

### Pattern 3: Structured Logging with Levels

```lavish
; ===== STRUCTURED LOGGING PATTERN =====

variable(global) string LOG_LEVEL = "INFO"  ; DEBUG, INFO, WARNING, ERROR, CRITICAL

variable(global) int LEVEL_DEBUG = 0
variable(global) int LEVEL_INFO = 1
variable(global) int LEVEL_WARNING = 2
variable(global) int LEVEL_ERROR = 3
variable(global) int LEVEL_CRITICAL = 4

function GetLogLevel(string levelName)
{
    switch ${levelName}
    {
        case DEBUG
            return ${LEVEL_DEBUG}
        case INFO
            return ${LEVEL_INFO}
        case WARNING
            return ${LEVEL_WARNING}
        case ERROR
            return ${LEVEL_ERROR}
        case CRITICAL
            return ${LEVEL_CRITICAL}
    }

    return ${LEVEL_INFO}
}

function Log(string level, string message)
{
    ; Check if should log based on current log level
    variable int messageLevelNum = ${GetLogLevel["${level}"]}
    variable int currentLevelNum = ${GetLogLevel["${LOG_LEVEL}"]}

    if ${messageLevelNum} < ${currentLevelNum}
    {
        return  ; Don't log, below threshold
    }

    variable string logLine = "${Time.Date} ${Time.Time} [${level}] ${message}"

    ; Color code based on level
    switch ${level}
    {
        case DEBUG
            echo "\ag${logLine}\ax"  ; Green
            break
        case INFO
            echo "${logLine}"  ; White
            break
        case WARNING
            echo "\ay${logLine}\ax"  ; Yellow
            break
        case ERROR
            echo "\ar${logLine}\ax"  ; Red
            break
        case CRITICAL
            echo "\am${logLine}\ax"  ; Magenta
            break
    }

    ; Write to file
    if ${LogFile.IsOpen}
    {
        LogFile:Write["${logLine}\n"]
    }
}

; Usage
LOG_LEVEL:Set["WARNING"]  ; Only log WARNING or higher

call Log "DEBUG" "Checking target lock"  ; Not logged
call Log "INFO" "Target locked"          ; Not logged
call Log "WARNING" "Low ammo"            ; Logged
call Log "ERROR" "Failed to dock"        ; Logged
call Log "CRITICAL" "Emergency retreat"  ; Logged
```

### Pattern 4: Context Logging

```lavish
; ===== CONTEXT LOGGING PATTERN =====

function LogContext(string level, string message)
{
    variable string context = ""

    ; Add location context
    if ${Me.InSpace}
    {
        context:Concat["[InSpace ${Me.SolarSystemID}] "]
    }
    elseif ${Me.InStation}
    {
        context:Concat["[InStation ${Me.StationID}] "]
    }

    ; Add ship context
    if ${MyShip(exists)}
    {
        context:Concat["[${MyShip.ToEntity.Name}] "]
    }

    ; Add state context
    if ${CurrentState.NotNULLOrEmpty}
    {
        context:Concat["[State: ${CurrentState}] "]
    }

    call Log "${level}" "${context}${message}"
}

; Usage
call LogContext "INFO" "Mining started"
; Output: [InSpace 30000142] [Retriever] [State: MINING] Mining started

call LogContext "ERROR" "Failed to lock asteroid"
; Output: [InSpace 30000142] [Retriever] [State: MINING] Failed to lock asteroid
```

---

## Recovery Strategies

### Strategy 1: Retry with Backoff

```lavish
; ===== RETRY WITH BACKOFF PATTERN =====

function RetryOperation(string operationName, int maxRetries, int baseDelay)
{
    variable int attempt = 0
    variable int delay = ${baseDelay}

    while ${attempt} < ${maxRetries}
    {
        attempt:Inc

        call Log "INFO" "Attempting ${operationName} (${attempt}/${maxRetries})"

        ; Try operation
        if ${DoOperation["${operationName}"]}
        {
            call Log "INFO" "${operationName} succeeded on attempt ${attempt}"
            return TRUE
        }

        ; Failed, wait before retry
        if ${attempt} < ${maxRetries}
        {
            call Log "WARNING" "${operationName} failed, retrying in ${delay}ms"
            wait ${delay}

            ; Exponential backoff
            delay:Set[${Math.Calc[${delay} * 2]}]
        }
    }

    call Log "ERROR" "${operationName} failed after ${maxRetries} attempts"
    return FALSE
}

function DoOperation(string operationName)
{
    switch ${operationName}
    {
        case "LockTarget"
            return ${TryLockTarget}
        case "Dock"
            return ${TryDock}
        case "Warp"
            return ${TryWarp}
    }

    return FALSE
}

; Usage
if !${RetryOperation["LockTarget", 3, 1000]}
{
    echo "Unable to lock target after 3 attempts, aborting"
    CurrentState:Set["ERROR"]
}
```

### Strategy 2: State Reset

```lavish
; ===== STATE RESET PATTERN =====

function ResetBotState()
{
    call Log "WARNING" "Resetting bot state"

    ; Clear all targets
    EVE:Execute[CmdClearTargets]
    wait 10

    ; Stop all modules
    EVE:Execute[CmdStopAllModules]
    wait 10

    ; Stop ship
    EVE:Execute[CmdStopShip]
    wait 10

    ; Clear cached data
    CachedTargets:Clear
    CachedAsteroids:Clear
    CurrentTarget:Set[0]

    ; Reset to IDLE state
    CurrentState:Set["IDLE"]

    call Log "INFO" "Bot state reset complete"
}

; Usage
function BotPulse()
{
    if ${DetectStuckState}
    {
        call Log "ERROR" "Bot appears stuck, resetting state"
        call ResetBotState
        return
    }

    ; Normal pulse...
}

function DetectStuckState()
{
    ; If in same state for > 5 minutes, probably stuck
    if ${CurrentState.Equal["${LastState}"]}
    {
        if ${Math.Calc[${LavishScript.RunningTime} - ${StateChangeTime}]} > 300000
        {
            return TRUE
        }
    }
    else
    {
        LastState:Set["${CurrentState}"]
        StateChangeTime:Set[${LavishScript.RunningTime}]
    }

    return FALSE
}
```

### Strategy 3: Fallback Actions

```lavish
; ===== FALLBACK ACTIONS PATTERN =====

function DockAtStation()
{
    ; Try: Dock at current station bookmark
    if ${TryDockAtBookmark["Home Station"]}
    {
        return TRUE
    }

    call Log "WARNING" "Failed to dock at bookmark, trying nearest station"

    ; Try: Dock at nearest station
    if ${TryDockAtNearest}
    {
        return TRUE
    }

    call Log "WARNING" "Failed to dock at nearest, trying any station"

    ; Try: Dock at any station in system
    if ${TryDockAtAnyStation}
    {
        return TRUE
    }

    call Log "ERROR" "Unable to dock at any station"

    ; Fallback: Emergency safe logoff
    call EmergencySafeLogoff

    return FALSE
}

function TryDockAtBookmark(string bookmarkName)
{
    ; ... implementation ...
    return FALSE
}

function TryDockAtNearest()
{
    variable entity station = ${Entity["GroupID = GROUPID_STATION && IsNearestStation"]}

    if !${station(exists)}
    {
        return FALSE
    }

    if ${station.Distance} > 1000000000  ; > 1M km
    {
        station:WarpTo[0]
        wait 100  ; Wait for warp
    }

    station:Dock
    wait 50  ; Wait for docking

    return ${Me.InStation}
}

function TryDockAtAnyStation()
{
    variable index:entity stations
    variable iterator station

    EVE:QueryEntities[stations, "GroupID = GROUPID_STATION"]
    stations:GetIterator[station]

    if ${station:First(exists)}
    {
        if ${station.Value.Distance} > 1000000000
        {
            station.Value:WarpTo[0]
            wait 100
        }

        station.Value:Dock
        wait 50

        return ${Me.InStation}
    }

    return FALSE
}

function EmergencySafeLogoff()
{
    call Log "CRITICAL" "Initiating emergency safe logoff"

    ; Stop everything
    EVE:Execute[CmdStopAllModules]
    wait 10

    EVE:Execute[CmdStopShip]
    wait 10

    ; Safe logoff
    EVE:Execute[CmdLogOff]
}
```

---

## Retry Logic

### Pattern 1: Simple Retry

```lavish
; ===== SIMPLE RETRY PATTERN =====

function LockTargetWithRetry(int64 entityID, int maxAttempts)
{
    variable int attempts = 0

    while ${attempts} < ${maxAttempts}
    {
        attempts:Inc

        if !${Entity[${entityID}](exists)}
        {
            call Log "ERROR" "Entity ${entityID} no longer exists"
            return FALSE
        }

        Entity[${entityID}]:LockTarget
        wait 20  ; Wait 2 seconds for lock

        if ${Entity[${entityID}].IsLockedTarget}
        {
            call Log "INFO" "Locked ${Entity[${entityID}].Name} on attempt ${attempts}"
            return TRUE
        }

        call Log "WARNING" "Lock attempt ${attempts} failed, retrying"
    }

    call Log "ERROR" "Failed to lock target after ${maxAttempts} attempts"
    return FALSE
}
```

### Pattern 2: Retry Until Condition

```lavish
; ===== RETRY UNTIL CONDITION PATTERN =====

function WaitForCondition(string conditionName, int timeout)
{
    variable int startTime = ${LavishScript.RunningTime}

    while ${Math.Calc[${LavishScript.RunningTime} - ${startTime}]} < ${timeout}
    {
        if ${CheckCondition["${conditionName}"]}
        {
            call Log "INFO" "${conditionName} satisfied after ${Math.Calc[${LavishScript.RunningTime} - ${startTime}]}ms"
            return TRUE
        }

        wait 10  ; Check every second
    }

    call Log "ERROR" "${conditionName} not satisfied after ${timeout}ms"
    return FALSE
}

function CheckCondition(string conditionName)
{
    switch ${conditionName}
    {
        case "InSpace"
            return ${Me.InSpace}
        case "InStation"
            return ${Me.InStation}
        case "NotInWarp"
            return !${Me.InWarp}
        case "TargetLocked"
            return ${Me.TargetCount} > 0
        case "CargoOpen"
            return ${EVEWindow["Cargo Hold"](exists)}
    }

    return FALSE
}

; Usage
if !${WaitForCondition["InSpace", 60000]}
{
    call Log "ERROR" "Failed to enter space after 60 seconds"
    return
}
```

### Pattern 3: Conditional Retry

```lavish
; ===== CONDITIONAL RETRY PATTERN =====

function ActivateModuleWithRetry(int moduleIndex, int64 targetID)
{
    variable int maxAttempts = 3
    variable int attempt = 0

    while ${attempt} < ${maxAttempts}
    {
        attempt:Inc

        ; Check preconditions
        if !${MyShip.Module[${moduleIndex}](exists)}
        {
            call Log "ERROR" "Module ${moduleIndex} does not exist"
            return FALSE
        }

        if !${MyShip.Module[${moduleIndex}].IsOnline}
        {
            call Log "ERROR" "Module ${moduleIndex} is offline"
            return FALSE
        }

        if ${MyShip.Module[${moduleIndex}].IsActive}
        {
            call Log "INFO" "Module ${moduleIndex} already active"
            return TRUE
        }

        ; Try activation
        MyShip.Module[${moduleIndex}]:Activate[${targetID}]
        wait 10  ; Wait for activation

        ; Check if successful
        if ${MyShip.Module[${moduleIndex}].IsActive}
        {
            call Log "INFO" "Module ${moduleIndex} activated on attempt ${attempt}"
            return TRUE
        }

        ; Failed, check why
        if ${MyShip.CapacitorPct} < 10
        {
            call Log "ERROR" "Cannot activate module: insufficient capacitor"
            return FALSE  ; No point retrying
        }

        call Log "WARNING" "Module activation failed (attempt ${attempt}), retrying"
    }

    call Log "ERROR" "Failed to activate module after ${maxAttempts} attempts"
    return FALSE
}
```

---

## Graceful Degradation

### Pattern 1: Feature Disabling

```lavish
; ===== FEATURE DISABLING PATTERN =====

variable bool DronesEnabled = TRUE
variable bool MarketEnabled = TRUE
variable bool UIEnabled = TRUE

function ProcessCombat()
{
    call LockTargets
    call ActivateWeapons

    ; Try to use drones
    if ${DronesEnabled}
    {
        if !${TryUseDrones}
        {
            call Log "ERROR" "Drones failed, disabling drone feature"
            DronesEnabled:Set[FALSE]
        }
    }

    call ProcessMovement
}

function TryUseDrones()
{
    if !${MyShip.DroneCapacity} > 0
    {
        return FALSE  ; Ship has no drones
    }

    ; Try to launch drones
    if ${MyShip.UsedDroneBayCapacity} > 0 && ${EVE.GetTargetDrones.Used} == 0
    {
        EVE:Execute[DroneReturnAndOrbit]

        wait 30  ; Wait for drones

        if ${EVE.GetTargetDrones.Used} > 0
        {
            return TRUE
        }
        else
        {
            call Log "ERROR" "Drone launch failed"
            return FALSE
        }
    }

    return TRUE
}
```

### Pattern 2: Reduced Functionality

```lavish
; ===== REDUCED FUNCTIONALITY PATTERN =====

variable string BotMode = "FULL"  ; FULL, REDUCED, MINIMAL

function BotPulse()
{
    ; Safety checks always run
    if ${CheckForHostiles}
    {
        call EmergencyDock
        return
    }

    ; Mode-dependent features
    switch ${BotMode}
    {
        case FULL
            call ProcessFullMode
            break
        case REDUCED
            call ProcessReducedMode
            break
        case MINIMAL
            call ProcessMinimalMode
            break
    }
}

function ProcessFullMode()
{
    ; All features
    call ProcessTargeting
    call ProcessModules
    call ProcessDrones
    call ProcessMovement
    call ProcessLooting
    call UpdateUI
}

function ProcessReducedMode()
{
    ; Essential features only
    call ProcessTargeting
    call ProcessModules
    call ProcessMovement
    ; No drones, no looting, minimal UI updates
}

function ProcessMinimalMode()
{
    ; Bare minimum
    call ProcessTargeting
    call ProcessModules
    ; Just shoot and run
}

function DegradeBotMode()
{
    switch ${BotMode}
    {
        case FULL
            call Log "WARNING" "Degrading to REDUCED mode"
            BotMode:Set["REDUCED"]
            break
        case REDUCED
            call Log "WARNING" "Degrading to MINIMAL mode"
            BotMode:Set["MINIMAL"]
            break
        case MINIMAL
            call Log "CRITICAL" "Already in MINIMAL mode, cannot degrade further"
            break
    }
}
```

### Pattern 3: Adaptive Behavior

```lavish
; ===== ADAPTIVE BEHAVIOR PATTERN =====

variable int FailureCount = 0
variable int MAX_FAILURES = 5

function ProcessMining()
{
    if !${TryLockAsteroid}
    {
        FailureCount:Inc
        call Log "WARNING" "Failed to lock asteroid (${FailureCount}/${MAX_FAILURES})"

        if ${FailureCount} >= ${MAX_FAILURES}
        {
            call Log "ERROR" "Too many failures, adapting strategy"
            call AdaptMiningStrategy
        }

        return
    }

    ; Success, reset counter
    FailureCount:Set[0]

    call ActivateMiners
}

function AdaptMiningStrategy()
{
    ; Strategy 1: Try different belt
    call Log "INFO" "Adaptation: Warping to different belt"
    if ${WarpToRandomBelt}
    {
        FailureCount:Set[0]
        return
    }

    ; Strategy 2: Try different system
    call Log "INFO" "Adaptation: Jumping to different system"
    if ${JumpToNeighborSystem}
    {
        FailureCount:Set[0]
        return
    }

    ; Strategy 3: Dock and wait
    call Log "WARNING" "Adaptation: Docking and pausing bot"
    call DockAtStation
    BotPaused:Set[TRUE]
}
```

---

## Emergency Procedures

### Pattern 1: Emergency Dock

```lavish
; ===== EMERGENCY DOCK PATTERN =====

function EmergencyDock()
{
    call Log "CRITICAL" "EMERGENCY DOCK INITIATED"

    ; Stop everything immediately
    EVE:Execute[CmdStopAllModules]
    wait 5

    EVE:Execute[CmdClearTargets]
    wait 5

    ; Recall drones if any
    if ${MyShip.UsedDroneBayCapacity} > 0
    {
        EVE:Execute[DroneReturnAndOrbit]
        wait 30  ; Wait for drones
    }

    ; Find nearest station
    variable entity station = ${Entity["GroupID = GROUPID_STATION && IsNearestStation"]}

    if !${station(exists)}
    {
        call Log "ERROR" "No station found, attempting safe logoff"
        call EmergencySafeLogoff
        return FALSE
    }

    ; Warp if needed
    if ${station.Distance} > 150000
    {
        call Log "INFO" "Warping to ${station.Name}"
        station:WarpTo[0]

        ; Wait for warp to complete
        variable int warpStart = ${LavishScript.RunningTime}
        while ${Me.InWarp} && ${Math.Calc[${LavishScript.RunningTime} - ${warpStart}]} < 60000
        {
            wait 10
        }

        if ${Me.InWarp}
        {
            call Log "ERROR" "Warp timeout, safe logoff"
            call EmergencySafeLogoff
            return FALSE
        }
    }

    ; Dock
    call Log "INFO" "Docking at ${station.Name}"
    station:Dock

    ; Wait for docking
    variable int dockStart = ${LavishScript.RunningTime}
    while !${Me.InStation} && ${Math.Calc[${LavishScript.RunningTime} - ${dockStart}]} < 30000
    {
        wait 10
    }

    if ${Me.InStation}
    {
        call Log "INFO" "EMERGENCY DOCK SUCCESSFUL"
        return TRUE
    }
    else
    {
        call Log "ERROR" "Dock timeout, safe logoff"
        call EmergencySafeLogoff
        return FALSE
    }
}
```

### Pattern 2: Emergency Warp

```lavish
; ===== EMERGENCY WARP PATTERN =====

function EmergencyWarp()
{
    call Log "CRITICAL" "EMERGENCY WARP INITIATED"

    ; Stop modules
    EVE:Execute[CmdStopAllModules]
    wait 5

    ; Clear targets
    EVE:Execute[CmdClearTargets]
    wait 5

    ; Check if scrambled
    if ${Me.ToEntity.IsWarpScrambled}
    {
        call Log "ERROR" "WARP SCRAMBLED! Cannot emergency warp"
        call EmergencyDock  ; Try to dock instead
        return FALSE
    }

    ; Find safe warp destination
    variable int64 warpTarget = ${GetSafeWarpDestination}

    if ${warpTarget} == 0
    {
        call Log "ERROR" "No safe warp destination found"
        return FALSE
    }

    ; Warp
    Entity[${warpTarget}]:WarpTo
    call Log "INFO" "Warping to ${Entity[${warpTarget}].Name}"

    ; Wait for warp
    wait 50

    if ${Me.InWarp}
    {
        call Log "INFO" "EMERGENCY WARP SUCCESSFUL"
        return TRUE
    }
    else
    {
        call Log "ERROR" "Emergency warp failed"
        return FALSE
    }
}

function GetSafeWarpDestination()
{
    ; Try: Bookmarked safe spot
    variable index:bookmark safes
    EVE:GetBookmarks[safes]

    variable iterator safe
    safes:GetIterator[safe]

    if ${safe:First(exists)}
        do
        {
            if ${safe.Value.Label.Find["Safe"]}
            {
                return ${safe.Value.ItemID}
            }
        }
        while ${safe:Next(exists)}

    ; Try: Random celestial
    variable index:entity celestials
    EVE:QueryEntities[celestials, "GroupID = GROUPID_PLANET || GroupID = GROUPID_MOON"]

    if ${celestials.Used} > 0
    {
        variable int randomIdx = ${Math.Rand[1,${celestials.Used}]}
        return ${celestials.Get[${randomIdx}].ID}
    }

    return 0
}
```

### Pattern 3: Emergency Shutdown

```lavish
; ===== EMERGENCY SHUTDOWN PATTERN =====

function EmergencyShutdown(string reason)
{
    call Log "CRITICAL" "EMERGENCY SHUTDOWN: ${reason}"

    ; Stop all bot operations
    BotRunning:Set[FALSE]

    ; Stop all ship activity
    EVE:Execute[CmdStopAllModules]
    wait 10

    EVE:Execute[CmdStopShip]
    wait 10

    EVE:Execute[CmdClearTargets]
    wait 10

    ; Dock if possible
    if ${Me.InSpace}
    {
        call EmergencyDock
    }

    ; Save state
    call SaveBotState

    ; Close UI
    if ${UIElement[BotWindow](exists)}
    {
        ui -unload "${Script.CurrentDirectory}/ui.xml"
    }

    call Log "CRITICAL" "EMERGENCY SHUTDOWN COMPLETE"

    ; End script
    Script:End
}
```

---

## Common Errors and Solutions

### Error 1: Target Despawned Mid-Operation

**Symptoms**: Module activation fails, targeting errors

**Solution**:

```lavish
function SafeModuleActivate(int moduleIndex, int64 targetID)
{
    ; Validate target exists
    if !${Entity[${targetID}](exists)}
    {
        call Log "WARNING" "Target ${targetID} despawned"
        return FALSE
    }

    ; Validate target locked
    if !${Entity[${targetID}].IsLockedTarget}
    {
        call Log "WARNING" "Target ${targetID} not locked"
        return FALSE
    }

    ; Validate module exists
    if !${MyShip.Module[${moduleIndex}](exists)}
    {
        call Log "ERROR" "Module ${moduleIndex} does not exist"
        return FALSE
    }

    ; Activate
    MyShip.Module[${moduleIndex}]:Activate[${targetID}]

    return TRUE
}
```

### Error 2: Inventory Lock

**Symptoms**: Cannot move items, operations fail

**Solution**:

```lavish
function SafeMoveItem(item itemToMove, string destination, int quantity)
{
    ; Check if item exists
    if !${itemToMove(exists)}
    {
        call Log "ERROR" "Item does not exist"
        return FALSE
    }

    ; Wait for inventory to be ready
    variable int waitTime = 0
    while ${EVE.InventoryOperationInProgress} && ${waitTime} < 10000
    {
        wait 10
        waitTime:Inc[100]
    }

    if ${EVE.InventoryOperationInProgress}
    {
        call Log "ERROR" "Inventory locked after 10 seconds"
        return FALSE
    }

    ; Move item
    itemToMove:MoveTo[${destination},${quantity}]

    ; Wait for operation to complete
    wait 10

    return TRUE
}
```

### Error 3: UI Window Not Found

**Symptoms**: Cannot interact with UI, operations fail

**Solution**:

```lavish
function OpenCargoWithRetry(int maxAttempts)
{
    variable int attempt = 0

    while ${attempt} < ${maxAttempts}
    {
        attempt:Inc

        ; Check if already open
        if ${EVEWindow["Cargo Hold"](exists)}
        {
            call Log "INFO" "Cargo already open"
            return TRUE
        }

        ; Open cargo
        EVE:Execute[OpenCargoHoldOfActiveShip]
        wait 20  ; Wait for window

        ; Check if opened
        if ${EVEWindow["Cargo Hold"](exists)}
        {
            call Log "INFO" "Cargo opened on attempt ${attempt}"
            return TRUE
        }

        call Log "WARNING" "Cargo open failed (attempt ${attempt})"
    }

    call Log "ERROR" "Failed to open cargo after ${maxAttempts} attempts"
    return FALSE
}
```

### Error 4: Stuck in Warp

**Symptoms**: Me.InWarp stays true indefinitely

**Solution**:

```lavish
function WarpToWithTimeout(int64 entityID, int timeout)
{
    if !${Entity[${entityID}](exists)}
    {
        return FALSE
    }

    Entity[${entityID}]:WarpTo

    ; Wait for warp to complete
    variable int startTime = ${LavishScript.RunningTime}

    while ${Me.InWarp} && ${Math.Calc[${LavishScript.RunningTime} - ${startTime}]} < ${timeout}
    {
        wait 10
    }

    ; Check if stuck
    if ${Me.InWarp}
    {
        call Log "ERROR" "Stuck in warp after ${timeout}ms"

        ; Try to fix
        EVE:Execute[CmdStopShip]
        wait 50

        if ${Me.InWarp}
        {
            call Log "ERROR" "Still stuck in warp, emergency reset"
            call ResetBotState
            return FALSE
        }
    }

    call Log "INFO" "Warp completed"
    return TRUE
}
```

### Error 5: Module Won't Activate

**Symptoms**: Module activation command does nothing

**Solutions**:

```lavish
function DiagnoseModuleFailure(int moduleIndex)
{
    variable item module = ${MyShip.Module[${moduleIndex}]}

    if !${module(exists)}
    {
        call Log "ERROR" "Module ${moduleIndex} does not exist"
        return "NOT_EXISTS"
    }

    if !${module.IsOnline}
    {
        call Log "ERROR" "Module offline"
        return "OFFLINE"
    }

    if ${module.IsActive}
    {
        call Log "INFO" "Module already active"
        return "ALREADY_ACTIVE"
    }

    if ${MyShip.CapacitorPct} < 10
    {
        call Log "ERROR" "Insufficient capacitor (${MyShip.CapacitorPct}%)"
        return "NO_CAP"
    }

    if ${module.IsReloadingAmmo}
    {
        call Log "WARNING" "Module reloading ammo"
        return "RELOADING"
    }

    call Log "WARNING" "Unknown module failure reason"
    return "UNKNOWN"
}
```

---

## Failsafes and Sanity Checks

### Pattern 1: Watchdog Timer

```lavish
; ===== WATCHDOG TIMER PATTERN =====

variable int LastPulseTime = 0
variable int WATCHDOG_TIMEOUT = 30000  ; 30 seconds

function WatchdogCheck()
{
    variable int timeSinceLastPulse = ${Math.Calc[${LavishScript.RunningTime} - ${LastPulseTime}]}

    if ${timeSinceLastPulse} > ${WATCHDOG_TIMEOUT}
    {
        call Log "CRITICAL" "Watchdog timeout: No pulse for ${timeSinceLastPulse}ms"
        call EmergencyShutdown "Watchdog timeout"
    }
}

function BotPulse()
{
    ; Update watchdog
    LastPulseTime:Set[${LavishScript.RunningTime}]

    ; Normal pulse operations...
}

; In main loop, check watchdog periodically
function main()
{
    while ${BotRunning}
    {
        call BotPulse
        call WatchdogCheck

        waitframe
    }
}
```

### Pattern 2: State Timeout

```lavish
; ===== STATE TIMEOUT PATTERN =====

variable string CurrentState = "IDLE"
variable int StateEnterTime = 0
variable collection:int StateTimeouts

function InitializeStateTimeouts()
{
    ; Set maximum time allowed in each state (milliseconds)
    StateTimeouts:Set["IDLE", 600000]      ; 10 minutes
    StateTimeouts:Set["MINING", 1800000]   ; 30 minutes
    StateTimeouts:Set["HAULING", 300000]   ; 5 minutes
    StateTimeouts:Set["COMBAT", 900000]    ; 15 minutes
    StateTimeouts:Set["FLEEING", 60000]    ; 1 minute
}

method ChangeState(string newState)
{
    if !${newState.Equal["${CurrentState}"]}
    {
        call Log "INFO" "State change: ${CurrentState} -> ${newState}"
        CurrentState:Set["${newState}"]
        StateEnterTime:Set[${LavishScript.RunningTime}]
    }
}

function CheckStateTimeout()
{
    variable int timeInState = ${Math.Calc[${LavishScript.RunningTime} - ${StateEnterTime}]}
    variable int maxTime = ${StateTimeouts.Get["${CurrentState}"]}

    if ${timeInState} > ${maxTime}
    {
        call Log "ERROR" "State timeout: In ${CurrentState} for ${timeInState}ms (max ${maxTime}ms)"
        call ResetBotState
        return TRUE
    }

    return FALSE
}
```

### Pattern 3: Sanity Bounds

```lavish
; ===== SANITY BOUNDS PATTERN =====

function ValidateNumericValue(string valueName, float value, float min, float max)
{
    if ${value} < ${min} || ${value} > ${max}
    {
        call Log "ERROR" "${valueName} out of bounds: ${value} (expected ${min}-${max})"
        return FALSE
    }

    return TRUE
}

function ValidateShipState()
{
    ; Shield should be 0-100%
    if !${ValidateNumericValue["ShieldPct", ${MyShip.ShieldPct}, 0, 100]}
    {
        call Log "CRITICAL" "Invalid shield percentage!"
        return FALSE
    }

    ; Armor should be 0-100%
    if !${ValidateNumericValue["ArmorPct", ${MyShip.ArmorPct}, 0, 100]}
    {
        call Log "CRITICAL" "Invalid armor percentage!"
        return FALSE
    }

    ; Cargo free space should be <= capacity (modern inventory API)
    if ${EVEWindow[Inventory](exists)}
    {
        variable float cargoFree = ${EVEWindow[Inventory].Child[ShipCargo].FreeSpace}
        variable float cargoCapacity = ${EVEWindow[Inventory].Child[ShipCargo].Capacity}

        if ${cargoFree} > ${cargoCapacity}
        {
            call Log "ERROR" "Cargo free space (${cargoFree}) > capacity (${cargoCapacity})"
            return FALSE
        }
    }

    return TRUE
}
```

---

## Real Examples from Bots

### Yamfa: Relay Error Handling

```lavish
; From Yamfa.iss - Relay data handling with validation

atom(script) YamfaTargetsRelay(string targetIDs, int64 relayactivetarget)
{
    echo "Relay received: ${targetIDs} with active: ${relayactivetarget}"

    ; Validate data
    if ${targetIDs.Length} > 1000
    {
        echo "ERROR: Relay data too large, ignoring"
        return
    }

    YamfaRelayedTargets:Set[${targetIDs}]
    YamfaRelayedActive:Set[${relayactivetarget}]
    YamfaNewRelayData:Set[TRUE]
}

function ProcessRelayedTargets(string targetIDs, int64 activeTarget, int currentTime)
{
    ; Handle empty relay
    if ${targetIDs.Length} == 0
    {
        SlaveTargetSet:Clear
        EVE:Execute[CmdClearTargets]
        SlaveActiveTarget:Set[0]
        return
    }

    ; Validate relay data
    ; ... parse and validate ...
}
```

### EVEBot: Module Activation with Error Checking

```lavish
; Typical EVEBot pattern for module activation

function ActivateModule(int moduleIndex, int64 targetID)
{
    ; Check module exists
    if !${MyShip.Module[${moduleIndex}](exists)}
    {
        Logger:Log["Module ${moduleIndex} does not exist", LOG_ERROR]
        return FALSE
    }

    ; Check module online
    if !${MyShip.Module[${moduleIndex}].IsOnline}
    {
        Logger:Log["Module ${moduleIndex} is offline", LOG_WARNING]
        return FALSE
    }

    ; Check target exists
    if ${targetID} > 0 && !${Entity[${targetID}](exists)}
    {
        Logger:Log["Target ${targetID} does not exist", LOG_WARNING]
        return FALSE
    }

    ; Activate
    MyShip.Module[${moduleIndex}]:Activate[${targetID}]

    return TRUE
}
```

---

## Summary

### Key Takeaways

1. **Error Detection**:
   - Check existence before accessing objects
   - Validate state before operations
   - Use sanity checks for bounds/ranges

2. **Logging**:
   - Log levels: DEBUG, INFO, WARNING, ERROR, CRITICAL
   - Context logging (location, ship, state)
   - File logging for persistence

3. **Recovery Strategies**:
   - Retry with exponential backoff
   - State reset when stuck
   - Fallback actions (primary → secondary → emergency)

4. **Graceful Degradation**:
   - Disable failing features
   - Reduce functionality under errors
   - Adapt behavior based on failures

5. **Emergency Procedures**:
   - Emergency dock (safest)
   - Emergency warp (if scrambled)
   - Emergency shutdown (last resort)

6. **Common Errors**:
   - Target despawned: Always check existence
   - Inventory locked: Wait for operations to complete
   - UI window not found: Retry with timeout
   - Stuck in warp: Timeout and force stop
   - Module won't activate: Diagnose specific cause

7. **Failsafes**:
   - Watchdog timer (detect hung bot)
   - State timeout (detect stuck states)
   - Sanity bounds (detect invalid values)


---

## Performance and Timing

**Wiki References**:
- LavishScript timing: `LavishScriptWiki/LavishScript/Commands/wait.html`
- Script profiling: `LavishScriptWiki/DataType/script.html#EnableProfiling`
- Turbo mode: `LavishScriptWiki/LavishScript/Commands/turbo.html`

---

## Performance Overview

Bot performance affects three critical areas:

1. **Responsiveness**: How quickly the bot reacts to game events
2. **CPU Usage**: How much system resources the bot consumes
3. **Detectability**: Performance patterns that may reveal automation

### Performance Goals

| Metric | Target | Poor | Good | Excellent |
|--------|--------|------|------|-----------|
| Main loop frequency | 10-60 Hz | < 1 Hz | 10-20 Hz | 30-60 Hz |
| Entity query time | < 100ms | > 500ms | 100-200ms | < 50ms |
| CPU usage | < 10% | > 30% | 10-20% | < 5% |
| Memory usage | < 100MB | > 500MB | 100-200MB | < 50MB |
| Response latency | < 1s | > 5s | 1-2s | < 500ms |

### Performance Trade-offs

```
         ┌────────────┐
         │ Fast       │
         │ Responsive │
         └─────┬──────┘
               │
               │ High CPU
               ▼
         ┌────────────┐
         │ Optimized  │
         │ Efficient  │
         └─────┬──────┘
               │
               │ Complex code
               ▼
         ┌────────────┐
         │ Simple     │
         │ Readable   │
         └────────────┘
```

**Balance**: Aim for "good enough" performance - optimize bottlenecks, not everything.

---

## Timing Fundamentals

### Command Comparison

```lavish
; ===== TIMING COMMAND COMPARISON =====

; wait N - Wait N deciseconds (1 decisecond = 100ms)
wait 10  ; Wait 1 second (10 * 100ms)
wait 5   ; Wait 500ms
wait 1   ; Wait 100ms

; waitframe - Wait for next game frame (~16ms at 60fps)
waitframe  ; Single frame wait

; Frame - Alias for waitframe (older syntax)
Frame  ; Single frame wait

; Turbo N - Set maximum commands per frame
Turbo 1000  ; Execute up to 1000 commands per frame
Turbo 100   ; Execute up to 100 commands per frame (default)
```

### Timing Precision

```lavish
; ===== TIMING PRECISION EXAMPLES =====

; High precision using LavishScript.RunningTime
variable int startTime = ${LavishScript.RunningTime}

; Do something...

variable int elapsed = ${Math.Calc[${LavishScript.RunningTime} - ${startTime}]}
echo "Operation took ${elapsed}ms"

; Lower precision using Time.Timestamp
variable int startTimestamp = ${Time.Timestamp}

; Do something...

variable int elapsedSeconds = ${Math.Calc[${Time.Timestamp} - ${startTimestamp}]}
echo "Operation took ${elapsedSeconds} seconds"
```

### Frame Rate Considerations

```lavish
; ===== FRAME RATE ADAPTIVE TIMING =====

variable float currentFPS = ${Display.FPS}
variable int frameTime = ${Math.Calc[1000 / ${currentFPS}]}

echo "Running at ${currentFPS} FPS (${frameTime}ms per frame)"

; Adaptive delay based on FPS
if ${currentFPS} > 50
{
    ; High FPS, can afford shorter delays
    wait 2  ; 200ms
}
elseif ${currentFPS} > 30
{
    ; Medium FPS
    wait 5  ; 500ms
}
else
{
    ; Low FPS, use longer delays to reduce load
    wait 10  ; 1 second
}
```

---

## Loop Optimization

### Pattern 1: Proper Frame Wait

```lavish
; ===== PROPER FRAME WAIT PATTERN =====

; BAD: No wait - 100% CPU!
while TRUE
{
    call BotPulse
    ; MISSING WAIT! Infinite loop!
}

; BAD: Wait without waitframe
while TRUE
{
    call BotPulse
    wait 5
    ; Works but can cause frame skipping
}

; GOOD: Wait + waitframe
while TRUE
{
    call BotPulse
    wait ${Math.Rand[2,6]}  ; Random jitter
    waitframe               ; Frame synchronization
}

; BEST: Waitframe + conditional wait
while TRUE
{
    variable int pulseStart = ${LavishScript.RunningTime}

    call BotPulse

    variable int pulseTime = ${Math.Calc[${LavishScript.RunningTime} - ${pulseStart}]}

    ; If pulse was fast, wait longer
    if ${pulseTime} < 10
    {
        wait ${Math.Calc[20 - ${pulseTime}]}
    }

    waitframe
}
```

### Pattern 2: Conditional Processing

```lavish
; ===== CONDITIONAL PROCESSING PATTERN =====

; BAD: Process everything every pulse
function BotPulse()
{
    call ProcessTargets      ; Expensive!
    call ProcessModules      ; Expensive!
    call ProcessInventory    ; Expensive!
    call ProcessUI           ; Expensive!
    call ProcessMarket       ; Expensive!
}

; GOOD: Conditional processing
variable int LastTargetUpdate = 0
variable int LastModuleUpdate = 0
variable int LastInventoryUpdate = 0
variable int LastUIUpdate = 0

function BotPulse()
{
    ; Process targets every 500ms
    if ${Math.Calc[${LavishScript.RunningTime} - ${LastTargetUpdate}]} > 500
    {
        call ProcessTargets
        LastTargetUpdate:Set[${LavishScript.RunningTime}]
    }

    ; Process modules every 200ms
    if ${Math.Calc[${LavishScript.RunningTime} - ${LastModuleUpdate}]} > 200
    {
        call ProcessModules
        LastModuleUpdate:Set[${LavishScript.RunningTime}]
    }

    ; Process inventory every 5 seconds
    if ${Math.Calc[${LavishScript.RunningTime} - ${LastInventoryUpdate}]} > 5000
    {
        call ProcessInventory
        LastInventoryUpdate:Set[${LavishScript.RunningTime}]
    }

    ; Process UI every second
    if ${Math.Calc[${LavishScript.RunningTime} - ${LastUIUpdate}]} > 1000
    {
        call ProcessUI
        LastUIUpdate:Set[${LavishScript.RunningTime}]
    }
}
```

### Pattern 3: Early Exit

```lavish
; ===== EARLY EXIT PATTERN =====

; BAD: Check all conditions
function BotPulse()
{
    variable bool inSpace = ${Me.InSpace}
    variable bool isReady = ${ISXEVE.IsReady}
    variable bool meExists = ${Me(exists)}
    variable bool shipExists = ${MyShip(exists)}

    if ${inSpace} && ${isReady} && ${meExists} && ${shipExists}
    {
        call ProcessBot
    }
}

; GOOD: Early exit (guard clauses)
function BotPulse()
{
    ; Exit early on cheap checks
    if !${ISXEVE.IsReady}
        return

    if !${Me(exists)}
        return

    if !${MyShip(exists)}
        return

    if !${Me.InSpace}
        return

    ; All checks passed, do expensive work
    call ProcessBot
}
```

---

## Query Optimization

### Pattern 1: Reduce Query Scope

```lavish
; ===== QUERY SCOPE REDUCTION =====

; BAD: Query everything
EVE:QueryEntities[Entities, "IsNPC"]
; Returns 100-1000+ entities across entire system!

; BETTER: Add distance filter
EVE:QueryEntities[Entities, "IsNPC && Distance < ${MyShip.MaxTargetRange}"]
; Returns only entities within targeting range

; BEST: Multiple filters
EVE:QueryEntities[Entities, "CategoryID = CATEGORYID_ENTITY && IsNPC && !IsMoribund && Distance < ${MyShip.MaxTargetRange}"]
; Returns only relevant, alive NPCs within range
```

### Pattern 2: Query Result Reuse

```lavish
; ===== QUERY RESULT REUSE =====

; BAD: Multiple identical queries
function ProcessTargets()
{
    ; Query 1
    variable index:entity NPCs
    EVE:QueryEntities[NPCs, "IsNPC && Distance < 50000"]

    echo "NPC count: ${NPCs.Used}"
}

function ProcessDrones()
{
    ; Query 2 - SAME QUERY!
    variable index:entity NPCs
    EVE:QueryEntities[NPCs, "IsNPC && Distance < 50000"]

    ; Launch drones...
}

; GOOD: Query once, reuse result
variable(global) index:entity CachedNPCs
variable int LastNPCQuery = 0

function UpdateNPCCache()
{
    if ${Math.Calc[${LavishScript.RunningTime} - ${LastNPCQuery}]} > 1000
    {
        EVE:QueryEntities[CachedNPCs, "IsNPC && Distance < 50000"]
        LastNPCQuery:Set[${LavishScript.RunningTime}]
    }
}

function ProcessTargets()
{
    call UpdateNPCCache
    echo "NPC count: ${CachedNPCs.Used}"
}

function ProcessDrones()
{
    call UpdateNPCCache
    ; Use CachedNPCs...
}
```

### Pattern 3: Progressive Filtering

```lavish
; ===== PROGRESSIVE FILTERING =====

; BAD: Complex single query
EVE:QueryEntities[Targets, "IsNPC && !IsMoribund && Distance < 50000 && ShieldPct < 50 && Name =- \"Guristas\""]
; Complex query, slow server-side processing

; GOOD: Simple query + client-side filtering
variable index:entity AllNPCs
variable index:entity FilteredNPCs

; Simple fast query
EVE:QueryEntities[AllNPCs, "IsNPC && Distance < 50000"]

; Client-side filtering (fast, local)
variable iterator NPC
AllNPCs:GetIterator[NPC]

if ${NPC:First(exists)}
    do
    {
        ; Filter: Not dead
        if ${NPC.Value.IsMoribund}
            continue

        ; Filter: Low shields
        if ${NPC.Value.ShieldPct} >= 50
            continue

        ; Filter: Faction
        if !${NPC.Value.Name.Find["Guristas"]}
            continue

        ; Passed all filters
        FilteredNPCs:Insert[${NPC.Value}]
    }
    while ${NPC:Next(exists)}
```

---

## Caching Strategies

### Pattern 1: Time-Based Cache

```lavish
; ===== TIME-BASED CACHE PATTERN =====

objectdef obj_EntityCache
{
    variable index:entity CachedEntities
    variable int LastUpdate = 0
    variable int CacheDuration = 1000  ; 1 second

    method Update()
    {
        if ${Math.Calc[${LavishScript.RunningTime} - ${This.LastUpdate}]} > ${This.CacheDuration}
        {
            EVE:QueryEntities[This.CachedEntities, "IsNPC && Distance < 100000"]
            This.LastUpdate:Set[${LavishScript.RunningTime}]
        }
    }

    member:index:entity GetEntities()
    {
        call This.Update
        return This.CachedEntities
    }

    member:int Count()
    {
        call This.Update
        return ${This.CachedEntities.Used}
    }
}

variable(global) obj_EntityCache NPCCache

; Usage
function ProcessTargets()
{
    echo "NPC Count: ${NPCCache.Count}"

    variable iterator NPC
    NPCCache.GetEntities:GetIterator[NPC]

    if ${NPC:First(exists)}
        do
        {
            echo "${NPC.Value.Name}"
        }
        while ${NPC:Next(exists)}
}
```

### Pattern 2: Invalidation Cache

```lavish
; ===== INVALIDATION CACHE PATTERN =====

objectdef obj_TargetCache
{
    variable index:entity CachedTargets
    variable bool IsDirty = TRUE

    method Invalidate()
    {
        This.IsDirty:Set[TRUE]
    }

    method Update()
    {
        if ${This.IsDirty}
        {
            EVE:QueryEntities[This.CachedTargets, "IsNPC && Distance < 50000"]
            This.IsDirty:Set[FALSE]
        }
    }

    member:index:entity GetTargets()
    {
        call This.Update
        return This.CachedTargets
    }
}

variable(global) obj_TargetCache TargetCache

; Usage
function BotPulse()
{
    ; Invalidate cache on state change
    if !${CurrentState.Equal["${LastState}"]}
    {
        TargetCache:Invalidate
        LastState:Set["${CurrentState}"]
    }

    ; Use cached targets
    variable iterator Target
    TargetCache.GetTargets:GetIterator[Target]

    ; Process targets...
}
```

### Pattern 3: Lazy Evaluation

```lavish
; ===== LAZY EVALUATION PATTERN =====

objectdef obj_LazyValue
{
    variable bool Computed = FALSE
    variable int CachedValue = 0

    method Invalidate()
    {
        This.Computed:Set[FALSE]
    }

    member:int Get()
    {
        if !${This.Computed}
        {
            ; Expensive computation
            This.CachedValue:Set[${ExpensiveCalculation}]
            This.Computed:Set[TRUE]
        }

        return ${This.CachedValue}
    }
}

variable(global) obj_LazyValue NPCCount

function ExpensiveCalculation()
{
    variable index:entity NPCs
    EVE:QueryEntities[NPCs, "IsNPC"]
    return ${NPCs.Used}
}

; Usage
function BotPulse()
{
    ; Only computes when accessed
    if ${NPCCount.Get} > 0
    {
        call ProcessCombat
    }

    ; Invalidate on location change
    if ${Me.SolarSystemID} != ${LastSystemID}
    {
        NPCCount:Invalidate
        LastSystemID:Set[${Me.SolarSystemID}]
    }
}
```

---

## CPU Usage Management

### Pattern 1: Turbo Control

```lavish
; ===== TURBO CONTROL PATTERN =====

; Default turbo: 100 commands/frame
Turbo 100

; HIGH CPU usage during startup (fast initialization)
function Initialize()
{
    echo "Starting initialization..."
    Turbo 4000  ; High turbo for fast startup

    ; Load all data
    call LoadConfiguration
    call LoadUI
    call LoadBehaviors

    echo "Initialization complete"
    Turbo 100  ; Back to normal
}

; LOW CPU usage during normal operation
function BotPulse()
{
    Turbo 100  ; Normal turbo

    call ProcessBotLogic
}

; VERY LOW CPU usage when idle/paused
function IdlePulse()
{
    Turbo 10  ; Very low turbo

    wait 100  ; Wait 10 seconds
}
```

**EVEBot Real Example** (Stable/EVEBot.iss, lines 134-168):

```lavish
function main()
{
    ; Set turbo to 4000 per frame for startup.
    Turbo 4000
    ; ... initialization ...

    turbo 8000
    Logger:Log[" Loading EVEDBs...", LOG_ECHOTOO]
    ; ... load databases ...
    turbo 4000

    ; ... more initialization ...

    Turbo 125  ; Back to low for normal operation
}
```

### Pattern 2: Throttled Processing

```lavish
; ===== THROTTLED PROCESSING PATTERN =====

objectdef obj_ThrottledProcessor
{
    variable int MinInterval = 1000  ; Minimum 1 second between calls
    variable int LastCall = 0

    method Process()
    {
        variable int now = ${LavishScript.RunningTime}

        if ${Math.Calc[${now} - ${This.LastCall}]} >= ${This.MinInterval}
        {
            call This.DoWork
            This.LastCall:Set[${now}]
        }
    }

    method DoWork()
    {
        ; Expensive work here
        echo "Processing..."
    }
}

variable(global) obj_ThrottledProcessor TargetUpdater
variable(global) obj_ThrottledProcessor UIUpdater
variable(global) obj_ThrottledProcessor InventoryManager

function BotPulse()
{
    ; These will only run once per second
    TargetUpdater:Process
    UIUpdater:Process
    InventoryManager:Process
}
```

### Pattern 3: Work Spreading

```lavish
; ===== WORK SPREADING PATTERN =====

; Instead of processing all targets at once, spread over multiple frames

variable index:int64 PendingTargets
variable int TargetsPerFrame = 2

function ProcessTargetsGradually()
{
    ; Process up to N targets per frame
    variable int processed = 0

    while ${PendingTargets.Used} > 0 && ${processed} < ${TargetsPerFrame}
    {
        variable int64 targetID = ${PendingTargets.Get[1]}
        PendingTargets:Remove[1]

        call ProcessSingleTarget ${targetID}

        processed:Inc
    }

    echo "Processed ${processed} targets, ${PendingTargets.Used} remaining"
}

function AddTargets()
{
    ; Add all targets to pending queue
    variable index:entity Entities
    variable iterator Entity

    EVE:QueryEntities[Entities, "IsNPC"]
    Entities:GetIterator[Entity]

    if ${Entity:First(exists)}
        do
        {
            PendingTargets:Insert[${Entity.Value.ID}]
        }
        while ${Entity:Next(exists)}
}

; Usage
function BotPulse()
{
    ; Refill queue if empty
    if ${PendingTargets.Used} == 0
    {
        call AddTargets
    }

    ; Process a few per frame
    call ProcessTargetsGradually
}
```

---

## Memory Management

### Pattern 1: Index Cleanup

```lavish
; ===== INDEX CLEANUP PATTERN =====

; BAD: Never clear, grows indefinitely
variable(global) index:entity AllTargets

function AddTarget(int64 entityID)
{
    AllTargets:Insert[${Entity[${entityID}]}]
    ; Never removed! Memory leak!
}

; GOOD: Periodic cleanup
variable(global) index:entity AllTargets
variable int LastCleanup = 0

function AddTarget(int64 entityID)
{
    AllTargets:Insert[${Entity[${entityID}]}]
}

function CleanupTargets()
{
    variable iterator Target
    AllTargets:GetIterator[Target]

    if ${Target:First(exists)}
        do
        {
            ; Remove dead/invalid targets
            if !${Target.Value(exists)} || ${Target.Value.IsMoribund}
            {
                AllTargets:Remove[${Target.Key}]
            }
        }
        while ${Target:Next(exists)}

    AllTargets:Collapse  ; Compact index
}

function BotPulse()
{
    ; Cleanup every 5 minutes
    if ${Math.Calc[${LavishScript.RunningTime} - ${LastCleanup}]} > 300000
    {
        call CleanupTargets
        LastCleanup:Set[${LavishScript.RunningTime}]
    }
}
```

**Yamfa Real Example** (Yamfa.iss, lines 707-736):

```lavish
function PeriodicCleanup()
{
    echo "Performing periodic cleanup..."

    ; Clean up old target timers
    if ${Me.Name.Equal["${MASTER_CHARACTER_NAME}"]}
    {
        MasterTargetTimers:Clear
    }
    else
    {
        variable int CurrentTime = ${Math.Calc[${LavishScript.RunningTime} / 100]}
        variable iterator Timer
        SlaveTargetTimers:GetIterator[Timer]

        if ${Timer:First(exists)}
            do
            {
                ; 5 seconds old
                if ${Math.Calc[${CurrentTime} - ${Timer.Value}]} > 50
                {
                    SlaveTargetTimers:Remove[${Timer.Key}]
                }
            }
            while ${Timer:Next(exists)}
    }

    call SaveConfig
    call UpdateUI
    echo "Cleanup complete"
}
```

### Pattern 2: String Pooling

```lavish
; ===== STRING POOLING PATTERN =====

; BAD: Create new strings repeatedly
function LogMessage()
{
    echo "${Time} [INFO] Bot pulse"  ; Creates new string every time
}

; GOOD: Reuse strings
variable(global) string LOG_PREFIX = "[INFO]"
variable(global) string LOG_WARNING = "[WARNING]"
variable(global) string LOG_ERROR = "[ERROR]"

function LogMessage(string level, string message)
{
    echo "${Time} ${level} ${message}"  ; Reuses level strings
}

; Usage
call LogMessage "${LOG_PREFIX}" "Bot pulse"
call LogMessage "${LOG_WARNING}" "Low shields"
call LogMessage "${LOG_ERROR}" "Target lost"
```

### Pattern 3: Object Pooling

```lavish
; ===== OBJECT POOLING PATTERN =====

; Instead of creating new objects, reuse from pool

objectdef obj_TargetInfo
{
    variable int64 EntityID
    variable string Name
    variable float Distance
    variable bool InUse = FALSE

    method Reset()
    {
        This.EntityID:Set[0]
        This.Name:Set[""]
        This.Distance:Set[0]
        This.InUse:Set[FALSE]
    }

    method Initialize(int64 entityID)
    {
        This.EntityID:Set[${entityID}]
        This.Name:Set["${Entity[${entityID}].Name}"]
        This.Distance:Set[${Entity[${entityID}].Distance}]
        This.InUse:Set[TRUE]
    }
}

variable(global) index:obj_TargetInfo TargetPool

function InitializeTargetPool(int poolSize)
{
    variable int i
    for (i:Set[1]; ${i} <= ${poolSize}; i:Inc)
    {
        TargetPool:Insert[obj_TargetInfo]
    }
}

function GetTargetInfo(int64 entityID)
{
    ; Find unused object in pool
    variable int i
    for (i:Set[1]; ${i} <= ${TargetPool.Used}; i:Inc)
    {
        if !${TargetPool.Get[${i}].InUse}
        {
            TargetPool.Get[${i}]:Initialize[${entityID}]
            return ${i}
        }
    }

    ; Pool exhausted, create new (shouldn't happen if pool sized correctly)
    TargetPool:Insert[obj_TargetInfo]
    TargetPool.Get[${TargetPool.Used}]:Initialize[${entityID}]
    return ${TargetPool.Used}
}

function ReleaseTargetInfo(int poolIndex)
{
    TargetPool.Get[${poolIndex}]:Reset
}
```

---

## Profiling and Measurement

### Pattern 1: Basic Timing

```lavish
; ===== BASIC TIMING PATTERN =====

function MeasureOperation(string operationName)
{
    variable int startTime = ${LavishScript.RunningTime}

    ; Do operation
    call ${operationName}

    variable int elapsed = ${Math.Calc[${LavishScript.RunningTime} - ${startTime}]}

    echo "${operationName} took ${elapsed}ms"
}

; Usage
call MeasureOperation "ProcessTargets"
call MeasureOperation "ProcessModules"
call MeasureOperation "ProcessInventory"
```

### Pattern 2: Performance Counter

```lavish
; ===== PERFORMANCE COUNTER PATTERN =====

objectdef obj_PerfCounter
{
    variable string Name
    variable int TotalCalls = 0
    variable int TotalTime = 0
    variable int MinTime = 999999
    variable int MaxTime = 0
    variable int LastTime = 0

    method Initialize(string name)
    {
        This.Name:Set["${name}"]
    }

    method StartMeasure()
    {
        This.LastTime:Set[${LavishScript.RunningTime}]
    }

    method EndMeasure()
    {
        variable int elapsed = ${Math.Calc[${LavishScript.RunningTime} - ${This.LastTime}]}

        This.TotalCalls:Inc
        This.TotalTime:Inc[${elapsed}]

        if ${elapsed} < ${This.MinTime}
        {
            This.MinTime:Set[${elapsed}]
        }

        if ${elapsed} > ${This.MaxTime}
        {
            This.MaxTime:Set[${elapsed}]
        }
    }

    member:float AverageTime()
    {
        if ${This.TotalCalls} == 0
            return 0

        return ${Math.Calc[${This.TotalTime} / ${This.TotalCalls}]}
    }

    method PrintStats()
    {
        echo "=== ${This.Name} ==="
        echo "Calls: ${This.TotalCalls}"
        echo "Total Time: ${This.TotalTime}ms"
        echo "Average: ${This.AverageTime.Precision[2]}ms"
        echo "Min: ${This.MinTime}ms"
        echo "Max: ${This.MaxTime}ms"
    }
}

variable(global) obj_PerfCounter PerfTargets
variable(global) obj_PerfCounter PerfModules

function Initialize()
{
    PerfTargets:Initialize["ProcessTargets"]
    PerfModules:Initialize["ProcessModules"]
}

function ProcessTargets()
{
    PerfTargets:StartMeasure

    ; Target processing logic...

    PerfTargets:EndMeasure
}

function ProcessModules()
{
    PerfModules:StartMeasure

    ; Module processing logic...

    PerfModules:EndMeasure
}

function ShowPerformanceStats()
{
    PerfTargets:PrintStats
    PerfModules:PrintStats
}
```

### Pattern 3: Script Profiling

```lavish
; ===== SCRIPT PROFILING PATTERN =====

; Enable LavishScript profiling
Script:EnableProfiling

; Run bot for a while...

; Dump profiling data
Redirect ProfileData.txt Script:DumpProfiling

; Disable profiling
Script:DisableProfiling

; ProfileData.txt will contain timing for every function/atom
```

---

## Common Performance Bottlenecks

### Bottleneck 1: Excessive Entity Queries

**Problem**:

```lavish
; BAD: Query entities every pulse
function BotPulse()
{
    variable index:entity NPCs
    EVE:QueryEntities[NPCs, "IsNPC"]  ; SLOW! 100-500ms per query

    echo "NPCs: ${NPCs.Used}"
}
```

**Solution**:

```lavish
; GOOD: Cache query results
variable(global) index:entity CachedNPCs
variable int LastNPCQuery = 0

function BotPulse()
{
    if ${Math.Calc[${LavishScript.RunningTime} - ${LastNPCQuery}]} > 1000
    {
        EVE:QueryEntities[CachedNPCs, "IsNPC"]
        LastNPCQuery:Set[${LavishScript.RunningTime}]
    }

    echo "NPCs: ${CachedNPCs.Used}"
}
```

### Bottleneck 2: Inventory Operations

**Problem**:

```lavish
; BAD: Access cargo every pulse (DEPRECATED API - for illustration)
function BotPulse()
{
    ; SLOW! Opens inventory window every pulse
    if !${EVEWindow[Inventory](exists)}
    {
        EVE:Execute[OpenInventory]
    }

    variable index:item Items
    EVEWindow[Inventory].Child[ShipCargo]:GetItems[Items]

    echo "Items: ${Items.Used}"
}
```

**Solution**:

```lavish
; GOOD: Cache cargo state, update only when needed (modern inventory API)
variable int CachedCargoUsed = 0
variable int LastCargoCheck = 0

function UpdateCargoCache()
{
    if ${Math.Calc[${LavishScript.RunningTime} - ${LastCargoCheck}]} > 5000
    {
        if ${EVEWindow[Inventory](exists)}
        {
            CachedCargoUsed:Set[${EVEWindow[Inventory].Child[ShipCargo].UsedSpace}]
            LastCargoCheck:Set[${LavishScript.RunningTime}]
        }
    }
}

function BotPulse()
{
    call UpdateCargoCache

    echo "Cargo used: ${CachedCargoUsed}"
}
```

### Bottleneck 3: Nested Loops

**Problem**:

```lavish
; BAD: O(n²) complexity
function FindMatchingTargets()
{
    variable index:entity NPCs
    variable index:string PriorityNames

    EVE:QueryEntities[NPCs, "IsNPC"]

    variable int i
    variable int j

    ; O(NPCs * PriorityNames) - SLOW!
    for (i:Set[1]; ${i} <= ${NPCs.Used}; i:Inc)
    {
        for (j:Set[1]; ${j} <= ${PriorityNames.Used}; j:Inc)
        {
            if ${NPCs.Get[${i}].Name.Find["${PriorityNames.Get[${j}]}"]}
            {
                echo "Match: ${NPCs.Get[${i}].Name}"
            }
        }
    }
}
```

**Solution**:

```lavish
; GOOD: Use set for O(1) lookup
variable(global) set PriorityNameSet

function LoadPriorityNames()
{
    PriorityNameSet:Add["Dire Pithi"]
    PriorityNameSet:Add["Guardian"]
    PriorityNameSet:Add["Guristas"]
}

function FindMatchingTargets()
{
    variable index:entity NPCs
    EVE:QueryEntities[NPCs, "IsNPC"]

    variable iterator NPC
    NPCs:GetIterator[NPC]

    ; O(n) complexity
    if ${NPC:First(exists)}
        do
        {
            ; Check if name contains any priority keyword
            if ${IsPriorityTarget["${NPC.Value.Name}"]}
            {
                echo "Match: ${NPC.Value.Name}"
            }
        }
        while ${NPC:Next(exists)}
}

function IsPriorityTarget(string name)
{
    variable iterator Keyword
    PriorityNameSet:GetIterator[Keyword]

    if ${Keyword:First(exists)}
        do
        {
            if ${name.Find["${Keyword.Value}"]}
            {
                return TRUE
            }
        }
        while ${Keyword:Next(exists)}

    return FALSE
}
```

### Bottleneck 4: String Operations

**Problem**:

```lavish
; BAD: String concatenation in loop
variable string result = ""
variable int i

for (i:Set[1]; ${i} <= 1000; i:Inc)
{
    result:Concat["${i} "]  ; Creates new string each iteration!
}
```

**Solution**:

```lavish
; GOOD: Build array, join once
variable index:string numbers

variable int i
for (i:Set[1]; ${i} <= 1000; i:Inc)
{
    numbers:Insert["${i}"]
}

; Join at end (if needed)
variable string result = "${numbers.Get[1]}"
for (i:Set[2]; ${i} <= ${numbers.Used}; i:Inc)
{
    result:Concat[" ${numbers.Get[${i}]}"]
}
```

---

## Optimization Patterns

### Pattern 1: Lazy Loading

```lavish
; ===== LAZY LOADING PATTERN =====

; Don't load data until needed

objectdef obj_LazyData
{
    variable bool Loaded = FALSE
    variable index:string Data

    method Load()
    {
        if !${This.Loaded}
        {
            echo "Loading expensive data..."

            ; Expensive load operation
            call This.DoExpensiveLoad

            This.Loaded:Set[TRUE]
        }
    }

    method DoExpensiveLoad()
    {
        ; Load from file, database, etc.
        variable int i
        for (i:Set[1]; ${i} <= 10000; i:Inc)
        {
            This.Data:Insert["Item ${i}"]
        }
    }

    member:index:string GetData()
    {
        call This.Load
        return This.Data
    }
}

variable(global) obj_LazyData Configuration

; Usage
function Initialize()
{
    echo "Bot initialized (config not yet loaded)"
}

function FirstTimeNeedConfig()
{
    ; Config loads on first access
    echo "Config items: ${Configuration.GetData.Used}"
}
```

### Pattern 2: Memoization

```lavish
; ===== MEMOIZATION PATTERN =====

; Cache function results

variable(global) collection:float DistanceCache

function GetDistanceToTarget(int64 targetID)
{
    ; Check cache first
    if ${DistanceCache.Element["${targetID}"](exists)}
    {
        return ${DistanceCache.Get["${targetID}"]}
    }

    ; Not cached, calculate
    if !${Entity[${targetID}](exists)}
    {
        return -1
    }

    variable float distance = ${Entity[${targetID}].Distance}

    ; Store in cache
    DistanceCache:Set["${targetID}", ${distance}]

    return ${distance}
}

function InvalidateDistanceCache()
{
    DistanceCache:Clear
}

; Usage
function BotPulse()
{
    ; First call calculates, subsequent calls use cache
    echo "Target 1 distance: ${GetDistanceToTarget[123456]}"
    echo "Target 1 distance: ${GetDistanceToTarget[123456]}"  ; Cached!

    ; Invalidate cache when position changes
    if ${Me.InWarp}
    {
        call InvalidateDistanceCache
    }
}
```

### Pattern 3: Batch Processing

```lavish
; ===== BATCH PROCESSING PATTERN =====

; Process multiple items at once instead of one at a time

variable(global) index:int64 PendingLocks

function QueueTargetLock(int64 entityID)
{
    PendingLocks:Insert[${entityID}]
}

function ProcessLockQueue()
{
    if ${PendingLocks.Used} == 0
        return

    echo "Processing ${PendingLocks.Used} pending locks"

    ; Lock all at once
    variable int i
    for (i:Set[1]; ${i} <= ${PendingLocks.Used} && ${Me.TargetCount} < ${Me.MaxLockedTargets}; i:Inc)
    {
        variable int64 targetID = ${PendingLocks.Get[${i}]}

        if ${Entity[${targetID}](exists)} && !${Entity[${targetID}].IsLockedTarget}
        {
            Entity[${targetID}]:LockTarget
        }
    }

    PendingLocks:Clear
}

; Usage
function FindTargets()
{
    variable index:entity NPCs
    EVE:QueryEntities[NPCs, "IsNPC && Distance < 50000"]

    variable iterator NPC
    NPCs:GetIterator[NPC]

    ; Queue all for locking
    if ${NPC:First(exists)}
        do
        {
            call QueueTargetLock ${NPC.Value.ID}
        }
        while ${NPC:Next(exists)}
}

function BotPulse()
{
    call FindTargets
    call ProcessLockQueue  ; Batch lock
}
```

---

## Real Examples from Bots

### Tehbot: Throttled Pulse

**File**: `obj_Tehbot.iss`

```lavish
objectdef obj_Tehbot
{
    variable bool Paused = TRUE
    variable int NextPulse
    variable int PulseIntervalInMilliseconds = 2000

    method Pulse()
    {
        if ${Paused}
        {
            return
        }

        if ${LavishScript.RunningTime} >= ${This.NextPulse}
        {
            ; Do work here

            ; Schedule next pulse with jitter
            This.NextPulse:Set[${Math.Calc[${LavishScript.RunningTime} + ${PulseIntervalInMilliseconds} + ${Math.Rand[500]}]}]
        }
    }
}
```

**Pattern**: Throttle pulse to every 2-2.5 seconds with random jitter

### Yamfa: Periodic Cleanup

**File**: `Yamfa.iss` (lines 707-736)

```lavish
function PeriodicCleanup()
{
    echo "Performing periodic cleanup..."

    ; Clean up old target timers
    if ${Me.Name.Equal["${MASTER_CHARACTER_NAME}"]}
    {
        MasterTargetTimers:Clear
    }
    else
    {
        variable int CurrentTime = ${Math.Calc[${LavishScript.RunningTime} / 100]}
        variable iterator Timer
        SlaveTargetTimers:GetIterator[Timer]

        if ${Timer:First(exists)}
            do
            {
                if ${Math.Calc[${CurrentTime} - ${Timer.Value}]} > 50  ; 5 seconds old
                {
                    SlaveTargetTimers:Remove[${Timer.Key}]
                }
            }
            while ${Timer:Next(exists)}
    }

    call SaveConfig
    call UpdateUI
    echo "Cleanup complete"
}
```

**Pattern**: Periodic cleanup (every 5 minutes) to prevent memory leaks

### EVEBot: Startup Turbo

**File**: `Stable/EVEBot.iss` (lines 134-232)

```lavish
function main()
{
    ; Set turbo to 4000 per frame for startup.
    Turbo 4000

    ; ... fast initialization ...

    turbo 8000
    Logger:Log[" Loading EVEDBs...", LOG_ECHOTOO]
    ; ... load databases ...
    turbo 4000

    ; ... more initialization ...

    Turbo 125  ; Back to normal for operation

    ; Main loop
    while TRUE
    {
        if !${EVEBot.Paused}
        {
            call ${Config.Common.CurrentBehavior}.ProcessState
        }
        wait ${Math.Calc[5 + (${Math.Rand[399]}/100)]}
    }
}
```

**Pattern**: High turbo during startup, low turbo during normal operation

---

## Performance Checklist

### Pre-Launch Checklist

- [ ] Main loop has `waitframe` or `wait`
- [ ] No infinite loops without wait
- [ ] Entity queries cached (not every pulse)
- [ ] Expensive operations throttled
- [ ] UI updates limited to 1-2 Hz
- [ ] Inventory operations minimized
- [ ] String operations optimized
- [ ] Memory cleanup implemented
- [ ] Turbo set appropriately (100-125 for normal operation)

### Optimization Checklist

- [ ] Profile code to find bottlenecks
- [ ] Query scope reduced (distance filters, category filters)
- [ ] Query results reused
- [ ] Client-side filtering used
- [ ] Caching implemented for expensive operations
- [ ] Periodic cleanup prevents memory leaks
- [ ] Nested loops avoided or optimized
- [ ] Early exit patterns used
- [ ] Work spreading implemented for large tasks

### Detection Avoidance Checklist

- [ ] Random jitter in timing (${Math.Rand[]})
- [ ] Variable delays (not fixed intervals)
- [ ] Human-like reaction times (200-1000ms)
- [ ] No perfect patterns (e.g., always exactly 2000ms)
- [ ] CPU usage reasonable (< 20%)
- [ ] No frame skipping or stuttering

---

## Summary

### Key Takeaways

1. **Timing**:
   - Always use `waitframe` or `wait` in loops
   - Add random jitter to avoid patterns
   - Use `LavishScript.RunningTime` for precise timing

2. **Loop Optimization**:
   - Early exit on invalid states
   - Conditional processing (not everything every pulse)
   - Adaptive delays based on workload

3. **Query Optimization**:
   - Reduce scope (distance, category filters)
   - Cache results (time-based or invalidation-based)
   - Client-side filtering
   - Reuse query results

4. **Caching**:
   - Time-based cache (expire after N ms)
   - Invalidation cache (invalidate on state change)
   - Lazy evaluation (compute on first access)

5. **CPU Management**:
   - Use `Turbo` appropriately (high for startup, low for operation)
   - Throttle expensive operations
   - Spread work over multiple frames

6. **Memory Management**:
   - Periodic cleanup of indices/collections
   - String pooling
   - Object pooling
   - Collapse indices after removals

7. **Common Bottlenecks**:
   - Excessive entity queries → Cache
   - Inventory operations → Minimize
   - Nested loops → Use sets for O(1) lookup
   - String operations → Build arrays, join once

8. **Profiling**:
   - Use `Script:EnableProfiling` for detailed analysis
   - Performance counters for tracking
   - Basic timing for quick checks
