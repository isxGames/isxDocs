# File 30: Debugging Techniques and Troubleshooting

**Layer 7: Advanced Topics - Part 3 of 4**

## Table of Contents
1. [Logging Frameworks](#logging-frameworks)
2. [Log Levels and Filtering](#log-levels)
3. [Console Output and Echo](#console-output)
4. [File-Based Logging](#file-logging)
5. [Performance Profiling](#profiling)
6. [Common Debugging Patterns](#debug-patterns)
7. [Error Detection and Diagnosis](#error-detection)
8. [ISXEVE Debugging Tools](#isxeve-tools)
9. [Real-Time Monitoring](#monitoring)
10. [Troubleshooting Guide](#troubleshooting)
11. [Best Practices](#best-practices)
12. [Complete Examples](#examples)

---

## Logging Frameworks <a name="logging-frameworks"></a>

### Why Logging Matters

**Critical for EVE bots because:**
- Scripts run for hours/days without supervision
- Need post-mortem analysis when things go wrong
- Performance bottlenecks must be identified
- Fleet coordination requires synchronized logs
- CCP detections need to be diagnosed

### obj_Logger Pattern (EVEBot)

EVEBot uses a sophisticated logging system:

```lavish
objectdef obj_Logger
{
    variable string LogFile
    variable string StatsLogFile
    variable queue:string ConsoleBuffer
    variable string PreviousMsg        ; For de-duplication

    method Initialize()
    {
        ; Create logs directory
        declare FP filepath "${Script.CurrentDirectory}"
        if !${FP.FileExists["Config"]}
        {
            FP:MakeSubdirectory["Config"]
        }
        FP:Set["${Script.CurrentDirectory}/Config"]
        if !${FP.FileExists["Logs"]}
        {
            FP:MakeSubdirectory["Logs"]
        }

        ; Set log file paths
        This.LogFile:Set["./Config/Logs/${Me.Name}.log"]
        This.StatsLogFile:Set["./Config/Logs/${Me.Name}_Stats.log"]

        This:InitializeLogs
    }

    method InitializeLogs()
    {
        ; Write header
        redirect -append "${This.LogFile}" echo "--------------------------------------------------------------------------------------"
        redirect -append "${This.LogFile}" echo "** EVEBot starting on ${Time.Date} at ${Time.Time24}"
    }
}
```

**Key Features:**
- **Per-character logs:** Each character gets own log file
- **Stats logging:** Separate file for statistics/metrics
- **Console buffer:** Queue messages until UI ready
- **De-duplication:** Prevents spam of repeated messages

### Global Logger Instance

```lavish
; Global logger - accessible everywhere
variable(global) obj_Logger Logger

function main()
{
    ; Initialize logger first
    Logger:Initialize

    ; Now can log anywhere
    Logger:Log["Bot starting"]
}
```

---

## Log Levels and Filtering <a name="log-levels"></a>

### EVEBot Log Levels

```lavish
; Define log level constants
#define LOG_MINOR       0    ; Minor info - log only, don't print
#define LOG_STANDARD    1    ; Standard - log and print to screen
#define LOG_CRITICAL    2    ; Critical - log, print, alert
#define LOG_ECHOTOO     3    ; Echo to console too
#define LOG_DEBUG       4    ; Debug logging (conditional)
```

### Log Method with Levels

```lavish
method Log(string StatusMessage, int Level=LOG_STANDARD, int Indent=0)
{
    variable string msg
    variable bool Filter = FALSE

    ; Filter debug messages unless debug mode enabled
    if ${Level} == LOG_DEBUG
    {
        if EVEBOT_DEBUG == 0
        {
            return    ; Skip debug messages in production
        }

        ; Optionally filter by DEBUG_TARGET
        if ${String["All"].NotEqual[DEBUG_TARGET]} &&
           !${StatusMessage.Token[1, " "].Find[DEBUG_TARGET](exists)}
        {
            return
        }
    }

    ; De-duplicate repeated messages
    if ${StatusMessage.Escape.Equal["${This.PreviousMsg.Escape}"]}
    {
        Filter:Set[TRUE]
    }
    else
    {
        This.PreviousMsg:Set["${StatusMessage.Escape}"]
    }

    ; Build message with timestamp
    msg:Set["${Time.Time24}: "]

    ; Add indentation
    variable int Count
    for (Count:Set[1]; ${Count}<=${Indent}; Count:Inc)
    {
        msg:Concat[" "]
    }

    msg:Concat["${StatusMessage.Escape}"]

    ; Output based on level
    if ${Level} > LOG_MINOR && !${Filter}
    {
        ; Print to UI console
        UIElement[StatusConsole@Status@EVEBotOptionsTab@EVEBot]:Echo["${msg}"]
    }

    if ${Level} == LOG_ECHOTOO
    {
        ; Also echo to InnerSpace console
        echo "${msg}"
    }

    ; Always write to file
    redirect -append "${This.LogFile}" Echo "${msg}"

    ; Critical messages go to IRC too
    if ${Level} == LOG_CRITICAL
    {
        ChatIRC:QueueMessage["${msg}"]
    }
}
```

### Usage Examples

```lavish
; Minor - logged but not displayed (spam reduction)
Logger:Log["Pulse iteration 1423", LOG_MINOR]

; Standard - log and display
Logger:Log["Warping to asteroid belt"]

; Critical - log, display, and alert
Logger:Log["HARD STOP: Hostiles detected!", LOG_CRITICAL]

; Echo to console - useful for development
Logger:Log["Debug: Target distance ${Entity[${targetID}].Distance}", LOG_ECHOTOO]

; Debug - only when EVEBOT_DEBUG=1
Logger:Log["obj_Miner: Checking asteroid ${asteroidID}", LOG_DEBUG]

; With indentation
Logger:Log["Starting mining cycle", LOG_STANDARD]
Logger:Log["Asteroid: ${Entity[${asteroidID}].Name}", LOG_STANDARD, 2]
Logger:Log["Distance: ${Entity[${asteroidID}].Distance}m", LOG_STANDARD, 4]
```

### Tehbot Log Levels

Tehbot uses a different, more modern approach:

```lavish
; Log level constants
#define LOG_DEBUG     0
#define LOG_INFO      1
#define LOG_CRITICAL  2

objectdef obj_Logger
{
    variable string LogLevelBar = LOG_INFO    ; Minimum level to log

    method Log(string CallingModule, string StatusMessage, string Color = "w",
               int level = LOG_INFO, int logLevelBar = LOG_DEBUG)
    {
        ; Filter by level
        if ${level} < ${logLevelBar}
        {
            return
        }

        ; Build formatted message
        variable string MSG
        MSG:Set["${Time.Time24}: "]

        ; Add level prefix
        switch ${level}
        {
            case LOG_DEBUG
                MSG:Concat["DEBUG"]
                break
            case LOG_INFO
                MSG:Concat["INFO"]
                break
            case LOG_CRITICAL
                MSG:Concat["CRITICAL"]
                break
        }

        ; Pad to column 20
        while ${MSG.Length} < 20
        {
            MSG:Concat[" "]
        }

        ; Add module name (max 15 chars)
        if ${CallingModule.Length} > 15
        {
            MSG:Concat["[${CallingModule.Left[15]}]"]
        }
        else
        {
            MSG:Concat["[${CallingModule}]"]
        }

        ; Pad to column 40
        while ${MSG.Length} < 40
        {
            MSG:Concat[" "]
        }

        ; Add message
        MSG:Concat["${StatusMessage.Escape.Replace["\"", ""].Replace["\\", ""]}"]

        ; Write to file
        redirect -append "${This.LogFile}" Echo "${MSG}"

        ; Update UI with color
        UI:Update["${CallingModule.Escape}", "${StatusMessage.Escape}", "${Color}"]
    }

    ; Convenience methods
    method LogInfo(string message)
    {
        Logger:Log[${LogModuleName}, "${message.Escape}", "w", LOG_INFO, ${This.LogLevelBar}]
    }

    method LogDebug(string message)
    {
        Logger:Log[${LogModuleName}, "${message.Escape}", "", LOG_DEBUG, ${This.LogLevelBar}]
    }

    method LogCritical(string message)
    {
        Logger:Log[${LogModuleName}, "${message.Escape}", "r", LOG_CRITICAL, ${This.LogLevelBar}]
    }
}
```

**Formatted Output Example:**

```
14:23:45: INFO     [CombatManager]        Engaging primary target
14:23:46: DEBUG    [TargetMgr]            Locked target ID 1423523523
14:23:47: CRITICAL [DefenseSystem]        Shield at 15% - fleeing!
```

---

## Console Output and Echo <a name="console-output"></a>

### Basic Echo

**InnerSpace console output:**

```lavish
; Simple console output
echo "Hello World"

; With variables
echo "Target: ${Entity[${targetID}].Name} at ${Entity[${targetID}].Distance}m"

; Formatted
echo "Shield: ${MyShip.Shield.Pct.Precision[1]}%"
```

### Echo with Formatting

```lavish
; Color codes (InnerSpace console)
echo "\agGreen text\ax - success"
echo "\arRed text\ax - error"
echo "\ayYellow text\ax - warning"
echo "\aoOrange text\ax - alert"

; Examples
echo "\agWarp successful\ax"
echo "\arCRITICAL: Low shields!\ax"
echo "\ayWARNING: Hostile in local\ax"
```

### Conditional Debugging

```lavish
; Debug flag pattern
variable bool DEBUG_MODE = FALSE

function DebugLog(string message)
{
    if ${DEBUG_MODE}
    {
        echo "[DEBUG] ${message}"
    }
}

; Usage
call DebugLog "Asteroid selection: ${asteroidID}"

; Only outputs if DEBUG_MODE = TRUE
```

### Verbose Levels

```lavish
variable int VERBOSE_LEVEL = 1    ; 0=quiet, 1=normal, 2=verbose, 3=debug

function Log(string message, int minLevel)
{
    if ${VERBOSE_LEVEL} >= ${minLevel}
    {
        echo "[${Time.Time24}] ${message}"
    }
}

; Usage
call Log "Bot starting" 0              ; Always shown
call Log "Warping to belt" 1           ; Normal and above
call Log "Checking asteroid 12345" 2   ; Verbose and above
call Log "Lock attempt #3" 3           ; Debug only
```

---

## File-Based Logging <a name="file-logging"></a>

### Redirect to File

```lavish
; Append to log file
redirect -append "mybot.log" echo "Bot starting at ${Time.Time24}"

; Create/overwrite file
redirect "mybot.log" echo "=== NEW SESSION ==="

; Multiple lines
redirect -append "mybot.log" echo "Line 1"
redirect -append "mybot.log" echo "Line 2"
```

### Log File Structure

**Good practice:**

```lavish
objectdef obj_MyLogger
{
    variable string LogFile = "./logs/${Me.Name}.log"

    method Initialize()
    {
        ; Create logs directory
        declare FP filepath "${Script.CurrentDirectory}"
        if !${FP.FileExists["logs"]}
        {
            FP:MakeSubdirectory["logs"]
        }

        ; Write header
        This:WriteHeader
    }

    method WriteHeader()
    {
        redirect "${This.LogFile}" echo "========================================"
        redirect -append "${This.LogFile}" echo "Session: ${Time.Date} ${Time.Time24}"
        redirect -append "${This.LogFile}" echo "Character: ${Me.Name}"
        redirect -append "${This.LogFile}" echo "Ship: ${MyShip.ToEntity.Name}"
        redirect -append "${This.LogFile}" echo "========================================"
    }

    method Log(string message)
    {
        redirect -append "${This.LogFile}" echo "[${Time.Time24}] ${message}"
    }

    method LogError(string message)
    {
        redirect -append "${This.LogFile}" echo "[${Time.Time24}] ERROR: ${message}"
        echo "\ar[ERROR] ${message}\ax"
    }
}
```

### Separate Log Files by Purpose

```lavish
objectdef obj_AdvancedLogger
{
    variable string MainLog = "./logs/${Me.Name}_main.log"
    variable string ErrorLog = "./logs/${Me.Name}_errors.log"
    variable string StatsLog = "./logs/${Me.Name}_stats.log"
    variable string CombatLog = "./logs/${Me.Name}_combat.log"

    method LogMain(string message)
    {
        redirect -append "${This.MainLog}" echo "[${Time.Time24}] ${message}"
    }

    method LogError(string message)
    {
        ; Write to both main and error log
        redirect -append "${This.MainLog}" echo "[${Time.Time24}] ERROR: ${message}"
        redirect -append "${This.ErrorLog}" echo "[${Time.Time24}] ${message}"
        echo "\ar${message}\ax"
    }

    method LogStat(string statName, string value)
    {
        redirect -append "${This.StatsLog}" echo "[${Time.Time24}] ${statName}: ${value}"
    }

    method LogCombat(string action, int64 targetID)
    {
        variable string msg = "${action} -> ${Entity[${targetID}].Name} (${Entity[${targetID}].ID})"
        redirect -append "${This.CombatLog}" echo "[${Time.Time24}] ${msg}"
    }
}

; Usage
MyLogger:LogMain["Bot started"]
MyLogger:LogError["Failed to lock target"]
MyLogger:LogStat["ISK Earned", "${ISKEarned}"]
MyLogger:LogCombat["Engaged", ${targetID}]
```

### Rotating Log Files

```lavish
objectdef obj_RotatingLogger
{
    variable string CurrentLog

    method Initialize()
    {
        ; Create log file with date/time
        This.CurrentLog:Set["./logs/${Me.Name}_${Time.Date.Replace["\/", "-"]}_${Time.Time24.Replace[":", ""]}.log"]

        This:WriteHeader
    }

    method CheckRotate()
    {
        ; Rotate log every 6 hours
        declare FP filepath "${This.CurrentLog}"

        if ${FP.FileExists["${This.CurrentLog}"]}
        {
            variable int fileAge = ${Math.Calc[${Time.Timestamp} - ${FP.FileTime}]}

            if ${fileAge} > 21600    ; 6 hours in seconds
            {
                ; Start new log file
                This:Initialize
            }
        }
    }
}
```

---

## Performance Profiling <a name="profiling"></a>

### Timestamp-Based Profiling

```lavish
objectdef obj_Profiler
{
    variable float StartTime
    variable collection:float Timings

    method Start(string operation)
    {
        This.Timings:Set[${operation}, ${Time.Timestamp}]
    }

    method End(string operation)
    {
        variable float duration = ${Math.Calc[${Time.Timestamp} - ${This.Timings.Get[${operation}]}]}

        echo "PROFILE: ${operation} took ${duration.Precision[3]}s"

        if ${duration} > 1.0
        {
            Logger:Log["WARNING: Slow operation ${operation}: ${duration.Precision[3]}s", LOG_CRITICAL]
        }
    }
}

; Usage
Profiler:Start["Entity Query"]
EVE:QueryEntities[Targets, "CategoryID = CATEGORYID_ENTITY"]
Profiler:End["Entity Query"]

; Output: PROFILE: Entity Query took 0.342s
```

### Frame Time Profiling

```lavish
objectdef obj_FrameProfiler
{
    variable float LastFrameTime
    variable int SlowFrameCount = 0

    method Pulse()
    {
        variable float frameTime = ${Math.Calc[${Time.Timestamp} - ${This.LastFrameTime}]}

        if ${frameTime} > 0.1    ; 100ms = slow
        {
            This.SlowFrameCount:Inc
            Logger:Log["Slow frame: ${frameTime.Precision[3]}s", LOG_DEBUG]
        }

        This.LastFrameTime:Set[${Time.Timestamp}]
    }

    method GetStats()
    {
        echo "Slow frames: ${This.SlowFrameCount}"
    }
}
```

### Method Call Counter

```lavish
objectdef obj_CallCounter
{
    variable collection:int CallCounts

    method Count(string methodName)
    {
        if !${This.CallCounts.Element[${methodName}](exists)}
        {
            This.CallCounts:Set[${methodName}, 0]
        }

        This.CallCounts:Set[${methodName}, ${Math.Calc[${This.CallCounts.Get[${methodName}]} + 1]}]
    }

    method PrintStats()
    {
        variable iterator Item
        This.CallCounts:GetIterator[Item]

        echo "=== Call Count Statistics ==="

        if ${Item:First(exists)}
        {
            do
            {
                echo "${Item.Key}: ${Item.Value} calls"
            }
            while ${Item:Next(exists)}
        }
    }
}

; Usage in methods
method LockTarget(int64 targetID)
{
    CallCounter:Count["LockTarget"]

    ; ... actual logic ...
}

; After session
CallCounter:PrintStats

; Output:
; === Call Count Statistics ===
; LockTarget: 1523 calls
; ActivateWeapons: 1421 calls
; CheckCargo: 45 calls
```

### Memory Usage Tracking

```lavish
objectdef obj_MemoryTracker
{
    variable int LastMemory

    method Check()
    {
        variable int currentMem = ${Script.MemUsage}
        variable int delta = ${Math.Calc[${currentMem} - ${This.LastMemory}]}

        if ${delta} > 10000000    ; 10MB increase
        {
            echo "WARNING: Memory increased by ${Math.Calc[${delta}/1000000]}MB"
            echo "Current usage: ${Math.Calc[${currentMem}/1000000]}MB"
        }

        This.LastMemory:Set[${currentMem}]
    }
}

; Check periodically
method Pulse()
{
    if ${Math.Calc[${Script.RunningTime} % 60000]} == 0    ; Every minute
    {
        MemoryTracker:Check
    }
}
```

---

## Common Debugging Patterns <a name="debug-patterns"></a>

### Pattern 1: Trace Logging

**Log every state transition:**

```lavish
method SetState()
{
    variable string oldState = "${This.CurrentState}"

    ; ... state logic ...

    if !${This.CurrentState.Equal[${oldState}]}
    {
        Logger:Log["STATE: ${oldState} -> ${This.CurrentState}"]
    }
}

; Output:
; 14:23:45: STATE: IDLE -> WARPING
; 14:23:52: STATE: WARPING -> MINING
; 14:24:15: STATE: MINING -> HAULING
```

### Pattern 2: Variable Dumping

**Log all relevant variables:**

```lavish
method DebugDump()
{
    echo "=== Debug Dump ==="
    echo "CurrentState: ${This.CurrentState}"
    echo "InSpace: ${Me.InSpace}"
    echo "InStation: ${Me.InStation}"
    echo "CargoUsed: ${MyShip.Cargo.UsedCapacity}"
    echo "CargoFree: ${MyShip.Cargo.FreeSpace}"
    echo "Shield: ${MyShip.Shield.Pct.Precision[1]}%"
    echo "Targets: ${Me.GetTargets}"
    echo "=================="
}

; Call when confused
if ${somethingWeird}
{
    This:DebugDump
}
```

### Pattern 3: Entity Debugging

**Log entity details:**

```lavish
function DebugEntity(int64 entityID)
{
    if !${Entity[${entityID}](exists)}
    {
        echo "Entity ${entityID} does not exist!"
        return
    }

    variable entity E = ${Entity[${entityID}]}

    echo "=== Entity ${entityID} ==="
    echo "Name: ${E.Name}"
    echo "Type: ${E.Type}"
    echo "Group: ${E.Group}"
    echo "Category: ${E.CategoryID}"
    echo "Distance: ${E.Distance}m"
    echo "IsLockedTarget: ${E.IsLockedTarget}"
    echo "IsTargetingMe: ${E.IsTargetingMe}"
    echo "IsNPC: ${E.IsNPC}"
    echo "========================"
}

; Usage
call DebugEntity ${Me.ActiveTarget.ID}
```

### Pattern 4: Conditional Breakpoints

**Pause execution when condition met:**

```lavish
method CheckCondition()
{
    if ${MyShip.Shield.Pct} < 20
    {
        echo "\arCRITICAL SHIELD - BREAKPOINT\ax"
        echo "Press any key to continue..."
        waitframe

        ; Force pause for inspection
        Script:Pause
    }
}
```

### Pattern 5: Stack Trace

**Show call path:**

```lavish
objectdef obj_CallStack
{
    variable queue:string Stack

    method Push(string functionName)
    {
        This.Stack:Queue["${functionName}"]
    }

    method Pop()
    {
        if ${This.Stack.Peek(exists)}
        {
            This.Stack:Dequeue
        }
    }

    method Print()
    {
        variable iterator Item
        variable queue:string TempQueue
        TempQueue:Set[${This.Stack}]

        echo "=== Call Stack ==="

        TempQueue:GetIterator[Item]
        if ${Item:First(exists)}
        {
            do
            {
                echo "  ${Item.Value}"
            }
            while ${Item:Next(exists)}
        }

        echo "=================="
    }
}

; Usage
method ProcessState()
{
    CallStack:Push["ProcessState"]

    switch ${This.CurrentState}
    {
        case MINING
            This:DoMining
            break
    }

    CallStack:Pop
}

method DoMining()
{
    CallStack:Push["DoMining"]

    ; If error occurs
    if ${error}
    {
        CallStack:Print
    }

    CallStack:Pop
}

; Output:
; === Call Stack ===
;   ProcessState
;   DoMining
; ==================
```

---

## Error Detection and Diagnosis <a name="error-detection"></a>

### ISXEVE Error Checking

```lavish
; Check if ISXEVE loaded
if !${ISXEVE(exists)}
{
    echo "ERROR: ISXEVE extension not loaded!"
    echo "Run: ext -require isxeve"
    return
}

; Check if EVE ready
if !${ISXEVE.IsSafe}
{
    echo "WARNING: ISXEVE not safe - waiting..."
    wait 50
}

; Check if character loaded
if !${Me(exists)}
{
    echo "ERROR: Character not loaded in game!"
    return
}

; Check if in space
if !${Me.InSpace} && !${Me.InStation}
{
    echo "ERROR: Character state unknown!"
    return
}
```

### Entity Validation

```lavish
function ValidateEntity(int64 entityID)
{
    ; Check exists
    if !${Entity[${entityID}](exists)}
    {
        Logger:LogError["Entity ${entityID} does not exist"]
        return FALSE
    }

    ; Check still valid (not despawned)
    if ${Entity[${entityID}].ID} == 0
    {
        Logger:LogError["Entity ${entityID} has ID=0 (despawned)"]
        return FALSE
    }

    ; Check distance (common issue)
    if ${Entity[${entityID}].Distance} > 250000
    {
        Logger:LogError["Entity ${entityID} out of range: ${Entity[${entityID}].Distance}m"]
        return FALSE
    }

    return TRUE
}

; Usage
if !${This:ValidateEntity[${targetID}]}
{
    This:SelectNewTarget
}
```

### Module Activation Errors

```lavish
method ActivateModule(int64 moduleID)
{
    if !${Me.GetModule[${moduleID}](exists)}
    {
        Logger:LogError["Module ${moduleID} not found"]
        return FALSE
    }

    variable module M = ${Me.GetModule[${moduleID}]}

    ; Check if can activate
    if ${M.IsOnline} == FALSE
    {
        Logger:LogError["Module ${M.ToItem.Name} is OFFLINE"]
        return FALSE
    }

    if ${M.IsActive}
    {
        Logger:Log["Module ${M.ToItem.Name} already active", LOG_MINOR]
        return TRUE
    }

    if ${M.IsActivatable} == FALSE
    {
        Logger:LogError["Module ${M.ToItem.Name} is not activatable"]
        return FALSE
    }

    if ${M.IsReloadingAmmo}
    {
        Logger:Log["Module ${M.ToItem.Name} is reloading", LOG_MINOR]
        return FALSE
    }

    ; Attempt activation
    M:Click

    ; Verify
    wait 10
    if !${M.IsActive}
    {
        Logger:LogError["Module ${M.ToItem.Name} failed to activate"]
        return FALSE
    }

    Logger:Log["Activated ${M.ToItem.Name}"]
    return TRUE
}
```

### Common Error Messages

```lavish
function DiagnoseError(string errorMessage)
{
    ; Parse common errors
    if ${errorMessage.Find["timed out"](exists)}
    {
        Logger:Log["DIAGNOSIS: Operation timeout - server lag or network issues"]
    }
    elseif ${errorMessage.Find["not found"](exists)}
    {
        Logger:Log["DIAGNOSIS: Object no longer exists - entity despawned or moved"]
    }
    elseif ${errorMessage.Find["out of range"](exists)}
    {
        Logger:Log["DIAGNOSIS: Target moved out of range - need to approach"]
    }
    elseif ${errorMessage.Find["jammed"](exists)}
    {
        Logger:Log["DIAGNOSIS: Ship is ECM jammed - cannot lock targets"]
    }
    elseif ${errorMessage.Find["scrambled"](exists)}
    {
        Logger:Log["DIAGNOSIS: Ship is warp scrambled - cannot warp"]
    }
}
```

---

## ISXEVE Debugging Tools <a name="isxeve-tools"></a>

### ISXEVE.Debug_LogMsg

**Built-in ISXEVE debug logging:**

```lavish
; Enable debug logging
ISXEVE.Debug_LogMsg["MyModule", "Starting processing"]

; With categorization
ISXEVE.Debug_LogMsg["TargetManager", "Locking target ${targetID}"]
ISXEVE.Debug_LogMsg["Movement", "Warping to ${bookmark}"]
ISXEVE.Debug_LogMsg["Combat", "Activating weapons on ${Entity[${targetID}].Name}"]
```

### ISXEVE Diagnostics

```lavish
function PrintISXEVEDiagnostics()
{
    echo "=== ISXEVE Diagnostics ==="
    echo "Version: ${ISXEVE.Version}"
    echo "IsSafe: ${ISXEVE.IsSafe}"
    echo "LastError: ${ISXEVE.LastError}"
    echo "MemManager Loaded: ${ISXEVE.MemManager.IsValid}"
    echo "=========================="
}
```

### Session Validation

```lavish
function ValidateSession()
{
    echo "=== Session Validation ==="

    ; Check character
    if ${Me(exists)}
    {
        echo "✓ Character: ${Me.Name}"
    }
    else
    {
        echo "✗ Character: NOT LOADED"
        return FALSE
    }

    ; Check ship
    if ${MyShip(exists)}
    {
        echo "✓ Ship: ${MyShip.ToEntity.Name}"
    }
    else
    {
        echo "✗ Ship: NOT LOADED"
        return FALSE
    }

    ; Check location
    if ${Me.InSpace}
    {
        echo "✓ Location: In Space (${Me.SolarSystemID})"
    }
    elseif ${Me.InStation}
    {
        echo "✓ Location: In Station (${Me.StationID})"
    }
    else
    {
        echo "✗ Location: UNKNOWN"
        return FALSE
    }

    ; Check UI
    if ${EVE(exists)}
    {
        echo "✓ EVE UI: Loaded"
    }
    else
    {
        echo "✗ EVE UI: NOT LOADED"
        return FALSE
    }

    echo "==========================="
    return TRUE
}
```

---

## Real-Time Monitoring <a name="monitoring"></a>

> **📡 IRC Remote Monitoring:** For remote bot monitoring and alerting, IRC can send critical log messages, status updates, and alerts to an IRC channel. See File 28 (Relay_System_and_IPC.md) IRC Bridge Integration section and `__CRITICAL_NEWEST_ISXIM_Reference.md` for implementation details. EVEBot already implements this pattern (lines 173-177).

### Status Display

```lavish
objectdef obj_StatusMonitor
{
    method PrintStatus()
    {
        ; Clear console (optional)
        ; execute cls

        echo "┌─────────────────────────────────────┐"
        echo "│ EVE Bot Status Monitor              │"
        echo "├─────────────────────────────────────┤"
        echo "│ Character: ${Me.Name.Left[20]}"
        echo "│ Ship: ${MyShip.ToEntity.Name.Left[25]}"
        echo "│ State: ${This.CurrentState.Left[20]}"
        echo "├─────────────────────────────────────┤"
        echo "│ Shield: ${MyShip.Shield.Pct.Precision[0]}%   Armor: ${MyShip.Armor.Pct.Precision[0]}%"
        echo "│ Cap: ${MyShip.Capacitor.Pct.Precision[0]}%      Cargo: ${Ship.CargoFull.Precision[0]}%"
        echo "├─────────────────────────────────────┤"
        echo "│ Targets: ${Me.TargetCount}/${Me.MaxLockedTargets}"
        echo "│ Uptime: ${Math.Calc[${Script.RunningTime}/1000/60].Int} minutes"
        echo "└─────────────────────────────────────┘"
    }

    method Pulse()
    {
        ; Update every 5 seconds
        if ${Math.Calc[${Script.RunningTime} % 5000]} < 100
        {
            This:PrintStatus
        }
    }
}
```

### Metrics Collection

```lavish
objectdef obj_Metrics
{
    variable int TargetsDestroyed = 0
    variable int OreMined = 0
    variable float ISKEarned = 0
    variable int Deaths = 0
    variable float StartTime

    method Initialize()
    {
        This.StartTime:Set[${Time.Timestamp}]
    }

    method RecordKill()
    {
        This.TargetsDestroyed:Inc
    }

    method RecordOre(int quantity)
    {
        This.OreMined:Set[${Math.Calc[${This.OreMined} + ${quantity}]}]
    }

    method RecordISK(float amount)
    {
        This.ISKEarned:Set[${Math.Calc[${This.ISKEarned} + ${amount}]}]
    }

    method RecordDeath()
    {
        This.Deaths:Inc
    }

    method PrintReport()
    {
        variable float runtime = ${Math.Calc[${Time.Timestamp} - ${This.StartTime}]}
        variable float hours = ${Math.Calc[${runtime} / 3600]}

        echo "=== Session Report ==="
        echo "Runtime: ${hours.Precision[2]} hours"
        echo "Targets Destroyed: ${This.TargetsDestroyed}"
        echo "Ore Mined: ${This.OreMined} m3"
        echo "ISK Earned: ${Math.Calc[${This.ISKEarned}/1000000].Precision[2]}M"
        echo "Deaths: ${This.Deaths}"
        echo ""
        echo "Kills/Hour: ${Math.Calc[${This.TargetsDestroyed} / ${hours}].Precision[1]}"
        echo "ISK/Hour: ${Math.Calc[${This.ISKEarned} / ${hours} / 1000000].Precision[2]}M"
        echo "====================="
    }
}
```

---

## Troubleshooting Guide <a name="troubleshooting"></a>

### Common Issues and Solutions

#### Issue: Script Won't Start

```lavish
; Diagnostic checklist
function DiagnoseStartupFailure()
{
    echo "=== Startup Diagnostics ==="

    ; Check ISXEVE
    if !${ISXEVE(exists)}
    {
        echo "✗ ISXEVE not loaded"
        echo "FIX: Run 'ext -require isxeve'"
        return
    }
    echo "✓ ISXEVE loaded"

    ; Check character
    if !${Me(exists)}
    {
        echo "✗ Character not in game"
        echo "FIX: Log into EVE first"
        return
    }
    echo "✓ Character: ${Me.Name}"

    ; Check files
    if !${Script.CurrentDirectory.FileExists["config"]}
    {
        echo "⚠ Config directory missing"
        echo "FIX: Creating config directory..."
        ; Auto-fix
        declare FP filepath "${Script.CurrentDirectory}"
        FP:MakeSubdirectory["config"]
    }

    echo "✓ All checks passed"
    echo "==========================="
}
```

#### Issue: Entity Not Found

```lavish
function DebugEntityNotFound(int64 entityID)
{
    echo "=== Entity Not Found Debug ==="
    echo "Searching for: ${entityID}"

    ; Check if ID is valid
    if ${entityID} == 0
    {
        echo "✗ Entity ID is 0 (invalid)"
        echo "CAUSE: Variable not initialized or entity despawned"
        return
    }

    ; Try to find entity
    if ${Entity[${entityID}](exists)}
    {
        echo "✓ Entity exists (false alarm)"
        echo "Name: ${Entity[${entityID}].Name}"
        echo "Distance: ${Entity[${entityID}].Distance}m"
    }
    else
    {
        echo "✗ Entity definitely doesn't exist"
        echo "CAUSES:"
        echo "  1. Entity despawned (asteroid mined out, NPC killed)"
        echo "  2. Entity warped away"
        echo "  3. You warped away"
        echo "  4. Grid changed"
        echo "SOLUTION: Query for new entity"
    }

    echo "=============================="
}
```

#### Issue: Module Won't Activate

```lavish
function DebugModuleActivation(int64 moduleID)
{
    echo "=== Module Activation Debug ==="

    if !${Me.GetModule[${moduleID}](exists)}
    {
        echo "✗ Module ${moduleID} not found"
        echo "CAUSE: Invalid module ID or module not fitted"
        return
    }

    variable module M = ${Me.GetModule[${moduleID}]}

    echo "Module: ${M.ToItem.Name}"
    echo "IsOnline: ${M.IsOnline}"
    echo "IsActive: ${M.IsActive}"
    echo "IsActivatable: ${M.IsActivatable}"
    echo "IsDeactivating: ${M.IsDeactivating}"
    echo "IsReloadingAmmo: ${M.IsReloadingAmmo}"
    echo "IsGoingOnline: ${M.IsGoingOnline}"

    ; Diagnose
    if !${M.IsOnline}
    {
        echo "✗ PROBLEM: Module is OFFLINE"
        echo "SOLUTION: Activate it manually first"
    }
    elseif ${M.IsActive}
    {
        echo "⚠ Module already active (not a problem)"
    }
    elseif !${M.IsActivatable}
    {
        echo "✗ PROBLEM: Module not activatable"
        echo "CAUSES:"
        echo "  - No target (weapons/remote reps)"
        echo "  - No charges (guns without ammo)"
        echo "  - Insufficient capacitor"
        echo "  - Module cycling down"
    }
    elseif ${M.IsReloadingAmmo}
    {
        echo "⚠ Module reloading (wait...)"
    }
    else
    {
        echo "✓ Module SHOULD activate - try again"
    }

    echo "==============================="
}
```

#### Issue: Targeting Fails

```lavish
function DebugTargeting(int64 targetID)
{
    echo "=== Targeting Debug ==="

    ; Check target count
    echo "Current Targets: ${Me.TargetCount}/${Me.MaxLockedTargets}"

    if ${Me.TargetCount} >= ${Me.MaxLockedTargets}
    {
        echo "✗ PROBLEM: At max target limit"
        echo "SOLUTION: Unlock a target first"
        return
    }

    ; Check entity exists
    if !${Entity[${targetID}](exists)}
    {
        echo "✗ PROBLEM: Entity doesn't exist"
        return
    }

    variable entity E = ${Entity[${targetID}]}

    echo "Target: ${E.Name}"
    echo "IsLockedTarget: ${E.IsLockedTarget}"
    echo "BeingTargeted: ${E.BeingTargeted}"
    echo "IsTargetingMe: ${E.IsTargetingMe}"
    echo "Distance: ${E.Distance}m"
    echo "Max Target Range: ${MyShip.MaxTargetRange}m"

    ; Diagnose
    if ${E.IsLockedTarget}
    {
        echo "⚠ Already locked (not a problem)"
    }
    elseif ${E.BeingTargeted}
    {
        echo "⚠ Already targeting (wait...)"
    }
    elseif ${E.Distance} > ${MyShip.MaxTargetRange}
    {
        echo "✗ PROBLEM: Out of range"
        echo "SOLUTION: Approach closer"
    }
    else
    {
        echo "✓ Should be able to lock - try again"
    }

    echo "======================"
}
```

#### Issue: Slow Performance

```lavish
function DiagnosePerformance()
{
    echo "=== Performance Diagnostics ==="

    ; Check script runtime
    variable float runtime = ${Math.Calc[${Script.RunningTime} / 1000]}
    echo "Script Uptime: ${runtime.Precision[0]}s"

    ; Check memory usage
    variable int memMB = ${Math.Calc[${Script.MemUsage} / 1000000]}
    echo "Memory Usage: ${memMB}MB"

    if ${memMB} > 100
    {
        echo "⚠ HIGH MEMORY USAGE"
        echo "CAUSES:"
        echo "  - Memory leak (collections not cleared)"
        echo "  - Too many cached objects"
        echo "SOLUTION: Restart script periodically"
    }

    ; Check entity count
    variable index:entity AllEntities
    EVE:QueryEntities[AllEntities]
    echo "Entities on Grid: ${AllEntities.Used}"

    if ${AllEntities.Used} > 1000
    {
        echo "⚠ HIGH ENTITY COUNT"
        echo "CAUSE: Large grid (busy system/belt)"
        echo "SOLUTION:"
        echo "  - Use more specific queries"
        echo "  - Cache entity lists"
        echo "  - Increase pulse intervals"
    }

    ; Check frame rate
    echo "FPS: ${Display.FPS}"

    if ${Display.FPS} < 10
    {
        echo "⚠ LOW FPS"
        echo "CAUSES:"
        echo "  - UI rendering enabled (disable 3D/UI)"
        echo "  - Too many operations per frame"
        echo "  - InnerSpace CPU throttling"
    }

    echo "==============================="
}
```

---

## Best Practices <a name="best-practices"></a>

### 1. Log Levels Appropriately

```lavish
; GOOD - appropriate levels
Logger:Log["Bot started", LOG_STANDARD]                  ; User wants to see this
Logger:Log["Pulse iteration 1423", LOG_MINOR]             ; Too spammy for display
Logger:Log["HARD STOP: Hostiles!", LOG_CRITICAL]          ; Urgent alert
Logger:Log["Checking asteroid 12345", LOG_DEBUG]          ; Debug only

; BAD - wrong levels
Logger:Log["Pulse iteration 1423", LOG_CRITICAL]          ; Not critical!
Logger:Log["HARD STOP: Hostiles!", LOG_MINOR]             ; Too important to hide!
```

### 2. Include Context in Logs

```lavish
; GOOD - context included
Logger:Log["Warping to ${bookmark} in ${solarSystem}"]
Logger:Log["Locking target ${Entity[${targetID}].Name} at ${Entity[${targetID}].Distance}m"]
Logger:Log["Cargo at ${Ship.CargoFull.Precision[0]}%, threshold is ${CargoThreshold}%"]

; BAD - no context
Logger:Log["Warping"]
Logger:Log["Locking target"]
Logger:Log["Cargo check"]
```

### 3. De-duplicate Repetitive Logs

```lavish
; GOOD - de-duplicated
method CheckShield()
{
    variable static int lastShieldPct = -1

    if ${MyShip.Shield.Pct.Int} != ${lastShieldPct}
    {
        Logger:Log["Shield: ${MyShip.Shield.Pct.Precision[0]}%"]
        lastShieldPct:Set[${MyShip.Shield.Pct.Int}]
    }
}

; BAD - logs every frame
method CheckShield()
{
    Logger:Log["Shield: ${MyShip.Shield.Pct.Precision[0]}%"]
}
```

### 4. Use Structured Logging

```lavish
; GOOD - structured
Logger:Log["[Combat] Engaged primary: ${Entity[${primaryID}].Name}"]
Logger:Log["[Combat] Weapons activated: 8/8"]
Logger:Log["[Combat] Target destroyed: ${bounty} ISK"]

; Makes searching logs easy:
; grep "\[Combat\]" logfile.log

; BAD - unstructured
Logger:Log["Fighting"]
Logger:Log["Guns on"]
Logger:Log["Kill"]
```

### 5. Log State Transitions

```lavish
; ALWAYS log state changes
method SetState()
{
    variable string oldState = "${This.CurrentState}"

    ; ... determine new state ...

    if !${This.CurrentState.Equal[${oldState}]}
    {
        Logger:Log["STATE: ${oldState} -> ${This.CurrentState}"]

        ; Log why if possible
        if ${This.CurrentState.Equal["FLEE"]}
        {
            Logger:Log["  Reason: ${FleeReason}"]
        }
    }
}
```

### 6. Profile Slow Operations

```lavish
; Profile anything that might be slow
method QueryEnemies()
{
    variable float startTime = ${Time.Timestamp}

    EVE:QueryEntities[Enemies, "CategoryID = CATEGORYID_ENTITY && IsNPC"]

    variable float duration = ${Math.Calc[${Time.Timestamp} - ${startTime}]}

    if ${duration} > 0.5
    {
        Logger:Log["WARNING: Slow entity query: ${duration.Precision[3]}s", LOG_CRITICAL]
    }
}
```

### 7. Validate Before Acting

```lavish
; Always validate critical operations
method LockTarget(int64 targetID)
{
    ; Validate entity
    if !${Entity[${targetID}](exists)}
    {
        Logger:LogError["LockTarget: Entity ${targetID} doesn't exist"]
        return FALSE
    }

    ; Validate range
    if ${Entity[${targetID}].Distance} > ${MyShip.MaxTargetRange}
    {
        Logger:LogError["LockTarget: ${Entity[${targetID}].Name} out of range (${Entity[${targetID}].Distance}m)"]
        return FALSE
    }

    ; Attempt lock
    Entity[${targetID}]:LockTarget

    return TRUE
}
```

---

## Complete Examples <a name="examples"></a>

### Example 1: Complete Logging System

```lavish
/* ==================== COMPLETE LOGGING SYSTEM ==================== */

; Log level constants
#define LOG_DEBUG     0
#define LOG_INFO      1
#define LOG_WARNING   2
#define LOG_ERROR     3
#define LOG_CRITICAL  4

objectdef obj_ComprehensiveLogger
{
    variable string LogFile = "./logs/${Me.Name}.log"
    variable string ErrorLog = "./logs/${Me.Name}_errors.log"
    variable string DebugLog = "./logs/${Me.Name}_debug.log"

    variable int LogLevel = LOG_INFO    ; Minimum level to log
    variable bool EnableDebug = FALSE   ; Debug flag

    variable string PreviousMsg
    variable bool DeDuplicate = TRUE

    method Initialize()
    {
        ; Create logs directory
        declare FP filepath "${Script.CurrentDirectory}"
        if !${FP.FileExists["logs"]}
        {
            FP:MakeSubdirectory["logs"]
        }

        ; Write headers
        This:WriteHeader[${This.LogFile}]
        This:WriteHeader[${This.ErrorLog}]

        if ${This.EnableDebug}
        {
            This:WriteHeader[${This.DebugLog}]
        }
    }

    method WriteHeader(string logFile)
    {
        redirect "${logFile}" echo "========================================="
        redirect -append "${logFile}" echo "Session: ${Time.Date} ${Time.Time24}"
        redirect -append "${logFile}" echo "Character: ${Me.Name}"
        redirect -append "${logFile}" echo "Ship: ${MyShip.ToEntity.Name}"
        redirect -append "${logFile}" echo "========================================="
    }

    method Log(string module, string message, int level)
    {
        ; Filter by level
        if ${level} < ${This.LogLevel}
        {
            return
        }

        ; De-duplicate if enabled
        if ${This.DeDuplicate}
        {
            if ${message.Escape.Equal["${This.PreviousMsg.Escape}"]}
            {
                return
            }
            This.PreviousMsg:Set["${message.Escape}"]
        }

        ; Build formatted message
        variable string levelStr
        variable string color

        switch ${level}
        {
            case LOG_DEBUG
                levelStr:Set["DEBUG"]
                color:Set["\ag"]
                break
            case LOG_INFO
                levelStr:Set["INFO"]
                color:Set["\aw"]
                break
            case LOG_WARNING
                levelStr:Set["WARN"]
                color:Set["\ay"]
                break
            case LOG_ERROR
                levelStr:Set["ERROR"]
                color:Set["\ar"]
                break
            case LOG_CRITICAL
                levelStr:Set["CRIT"]
                color:Set["\ao"]
                break
        }

        variable string timestamp = "${Time.Time24}"
        variable string formatted = "${timestamp} [${levelStr}] [${module}] ${message}"

        ; Write to main log
        redirect -append "${This.LogFile}" echo "${formatted}"

        ; Echo to console with color
        echo "${color}${formatted}\ax"

        ; Write to error log if error/critical
        if ${level} >= LOG_ERROR
        {
            redirect -append "${This.ErrorLog}" echo "${formatted}"
        }

        ; Write to debug log if debug enabled
        if ${This.EnableDebug} && ${level} == LOG_DEBUG
        {
            redirect -append "${This.DebugLog}" echo "${formatted}"
        }
    }

    ; Convenience methods
    method LogDebug(string module, string message)
    {
        This:Log["${module}", "${message}", LOG_DEBUG]
    }

    method LogInfo(string module, string message)
    {
        This:Log["${module}", "${message}", LOG_INFO]
    }

    method LogWarning(string module, string message)
    {
        This:Log["${module}", "${message}", LOG_WARNING]
    }

    method LogError(string module, string message)
    {
        This:Log["${module}", "${message}", LOG_ERROR]
    }

    method LogCritical(string module, string message)
    {
        This:Log["${module}", "${message}", LOG_CRITICAL]
    }
}

; Global logger instance
variable(global) obj_ComprehensiveLogger Logger

/* ==================== USAGE ==================== */

function main()
{
    ; Initialize logger
    Logger:Initialize

    ; Set log level (INFO and above)
    Logger.LogLevel:Set[LOG_INFO]

    ; Enable debug logging
    Logger.EnableDebug:Set[TRUE]

    ; Log at different levels
    Logger:LogInfo["Main", "Bot starting"]
    Logger:LogDebug["Main", "Debug mode enabled"]
    Logger:LogWarning["Main", "Low capacitor detected"]
    Logger:LogError["Main", "Failed to lock target"]
    Logger:LogCritical["Main", "HARD STOP - Hostiles detected!"]
}

/* OUTPUT (main log):
14:23:45 [INFO] [Main] Bot starting
14:23:46 [WARN] [Main] Low capacitor detected
14:23:47 [ERROR] [Main] Failed to lock target
14:23:48 [CRIT] [Main] HARD STOP - Hostiles detected!

OUTPUT (error log):
14:23:47 [ERROR] [Main] Failed to lock target
14:23:48 [CRIT] [Main] HARD STOP - Hostiles detected!

OUTPUT (debug log):
14:23:46 [DEBUG] [Main] Debug mode enabled
*/
```

### Example 2: Performance Profiler

```lavish
/* ==================== PERFORMANCE PROFILER ==================== */

objectdef obj_PerformanceProfiler
{
    variable collection:float StartTimes
    variable collection:float TotalTimes
    variable collection:int CallCounts

    method Start(string operation)
    {
        This.StartTimes:Set[${operation}, ${Time.Timestamp}]
    }

    method End(string operation)
    {
        if !${This.StartTimes.Element[${operation}](exists)}
        {
            echo "ERROR: End() called without Start() for ${operation}"
            return
        }

        variable float duration = ${Math.Calc[${Time.Timestamp} - ${This.StartTimes.Get[${operation}]}]}

        ; Update totals
        if !${This.TotalTimes.Element[${operation}](exists)}
        {
            This.TotalTimes:Set[${operation}, 0]
            This.CallCounts:Set[${operation}, 0]
        }

        This.TotalTimes:Set[${operation}, ${Math.Calc[${This.TotalTimes.Get[${operation}]} + ${duration}]}]
        This.CallCounts:Set[${operation}, ${Math.Calc[${This.CallCounts.Get[${operation}]} + 1]}]

        ; Warn if slow
        if ${duration} > 1.0
        {
            echo "\ayWARNING: Slow operation ${operation}: ${duration.Precision[3]}s\ax"
        }
    }

    method PrintReport()
    {
        echo "┌─────────────────────────────────────────────────────────┐"
        echo "│              Performance Profile Report                 │"
        echo "├─────────────────────────────────────────────────────────┤"

        variable iterator Item
        This.TotalTimes:GetIterator[Item]

        if ${Item:First(exists)}
        {
            do
            {
                variable string op = "${Item.Key}"
                variable float totalTime = ${Item.Value}
                variable int calls = ${This.CallCounts.Get[${op}]}
                variable float avgTime = ${Math.Calc[${totalTime} / ${calls}]}

                echo "│ ${op.Left[20]}"
                echo "│   Calls: ${calls}"
                echo "│   Total: ${totalTime.Precision[3]}s"
                echo "│   Average: ${avgTime.Precision[4]}s"
                echo "├─────────────────────────────────────────────────────────┤"
            }
            while ${Item:Next(exists)}
        }

        echo "└─────────────────────────────────────────────────────────┘"
    }
}

; Global profiler
variable(global) obj_PerformanceProfiler Profiler

/* ==================== USAGE ==================== */

function main()
{
    ; Profile entity query
    Profiler:Start["EntityQuery"]
    EVE:QueryEntities[Targets, "CategoryID = CATEGORYID_ENTITY"]
    Profiler:End["EntityQuery"]

    ; Profile targeting
    Profiler:Start["Targeting"]
    call LockAllTargets
    Profiler:End["Targeting"]

    ; Profile combat
    Profiler:Start["Combat"]
    call DoCombat
    Profiler:End["Combat"]

    ; Print report
    Profiler:PrintReport
}

/* OUTPUT:
┌─────────────────────────────────────────────────────────┐
│              Performance Profile Report                 │
├─────────────────────────────────────────────────────────┤
│ EntityQuery
│   Calls: 152
│   Total: 23.451s
│   Average: 0.1543s
├─────────────────────────────────────────────────────────┤
│ Targeting
│   Calls: 89
│   Total: 12.234s
│   Average: 0.1375s
├─────────────────────────────────────────────────────────┤
│ Combat
│   Calls: 89
│   Total: 451.123s
│   Average: 5.0688s
├─────────────────────────────────────────────────────────┤
└─────────────────────────────────────────────────────────┘
*/
```

---

## Summary

### Key Takeaways

1. **Logging is Essential**
   - Scripts run unattended for hours
   - Post-mortem analysis critical
   - Multiple log levels filter noise

2. **Performance Profiling Matters**
   - Identify bottlenecks early
   - Profile slow operations
   - Track metrics over time

3. **Error Handling**
   - Validate before acting
   - Log errors with context
   - Provide actionable diagnostics

4. **Debugging Tools**
   - ISXEVE built-in diagnostics
   - Custom profilers
   - Real-time monitoring

5. **Best Practices**
   - Log appropriately by level
   - Include context
   - De-duplicate spam
   - Profile slow paths

### What's Next?

**File 31:** Common Problems and Solutions
- Known ISXEVE bugs and workarounds
- Script compatibility issues
- Performance optimization tips
- Common mistakes and fixes
- Fleet coordination debugging
- Recovery from failures

### Navigation

[← Previous: File 29 - Configuration Management](29_Configuration_and_Settings_Management.md) | [Next: File 31 - Common Problems →](31_Common_Problems_and_Solutions.md)

---

*Layer 7 Progress: 3/4 Complete (75%)*
*Total Documentation Progress: 29/37 Files (78.4%)*
