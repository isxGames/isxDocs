# Main Loop and State Machines - Complete Bot Architecture Reference

**Purpose**: This file documents how to structure the main execution loop of EVE Online bots, including state machine patterns, timing, and bot lifecycle management. This is THE most critical architectural file for building functional bots.

**Target Reader**: AI learning to build EVE bots from scratch

> **Note on Code Examples**: This file contains illustrative examples that may include deprecated API patterns (e.g., `MyShip.CargoFreeSpace`, `MyShip.UsedCargoCapacity`). For production code, always use the modern `EVEWindow[Inventory].Child[ShipCargo]` API documented in Files 13 and 15. The complete working example at the end has been updated to use modern APIs where critical.

**Prerequisites**:
- Files 01-16 (Foundation, Scripting, ISXEVE API)
- Understanding of infinite loops and event systems
- Knowledge of ISXEVE objects (Me, MyShip, EVE)

**Wiki References**:
- LavishScript Events: `LavishScriptWiki/LavishScript/Events.html`
- ISXEVE onFrame event: `IsxeveWiki/Events.html`
- Frame timing: `LavishScriptWiki/LavishScript/Commands/waitframe.html`

---

## Table of Contents

1. [Bot Lifecycle Overview](#bot-lifecycle-overview)
2. [Main Loop Architectures](#main-loop-architectures)
3. [Simple Polling Loop Pattern](#simple-polling-loop-pattern)
4. [Behavior-Based State Machine](#behavior-based-state-machine)
5. [Event-Driven Architecture](#event-driven-architecture)
6. [State Machine Concepts](#state-machine-concepts)
7. [State Transition Patterns](#state-transition-patterns)
8. [Timing and Frame Management](#timing-and-frame-management)
9. [Safety-First Architecture](#safety-first-architecture)
10. [Common Patterns from Example Bots](#common-patterns-from-example-bots)
11. [Performance Considerations](#performance-considerations)
12. [Common Mistakes and Gotchas](#common-mistakes-and-gotchas)

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

**Architecture**: Event-driven pulse with independent modules

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

### Next Files

- **File 18**: Decision Making and Logic Patterns (How to choose targets, prioritize actions)
- **File 19**: Error Handling and Recovery (What to do when things go wrong)
- **File 20**: Performance and Timing (Optimization, profiling, efficiency)

---

**File Complete**: Main loop and state machine patterns fully documented with working examples from Yamfa, EVEBot, and Tehbot.
