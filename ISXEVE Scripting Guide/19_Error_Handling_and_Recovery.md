# Error Handling and Recovery - Bot Resilience Reference

**Purpose**: This file documents how to detect, handle, and recover from errors in EVE Online bots. Robust error handling is what separates a brittle prototype from a reliable production bot.

**Target Reader**: AI learning to build resilient EVE bots

**Prerequisites**:
- Files 01-18 (Foundation, API, Main Loops, Decision Making)
- Understanding of error states and exceptions
- Knowledge of logging and debugging

**Wiki References**:
- LavishScript echo: `LavishScriptWiki/LavishScript/Commands/echo.html`
- File I/O for logging: `LavishScriptWiki/TopLevelObjects/File.html`
- Script timing: `LavishScriptWiki/TopLevelObjects/LavishScript.html`

---

## Table of Contents

1. [Error Handling Overview](#error-handling-overview)
2. [Types of Errors](#types-of-errors)
3. [Error Detection Patterns](#error-detection-patterns)
4. [Logging Patterns](#logging-patterns)
5. [Recovery Strategies](#recovery-strategies)
6. [Retry Logic](#retry-logic)
7. [Graceful Degradation](#graceful-degradation)
8. [Emergency Procedures](#emergency-procedures)
9. [Common Errors and Solutions](#common-errors-and-solutions)
10. [Failsafes and Sanity Checks](#failsafes-and-sanity-checks)
11. [Real Examples from Bots](#real-examples-from-bots)
12. [Testing Error Handling](#testing-error-handling)

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

### Next File

- **File 20**: Performance and Timing (Optimization, profiling, efficiency)

---

**File Complete**: Error handling and recovery patterns fully documented with real examples and best practices.
