# Debugging And Troubleshooting

**Purpose:** Debugging techniques and common problem solutions for ISXEVE scripts
**Audience:** Developers troubleshooting and debugging EVE automation

---

## Table of Contents

### Debugging Techniques
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

### Common Problems
13. [Common Problems and Solutions](#common-problems-and-solutions)
14. [Entity and Targeting Problems](#entity-problems)
15. [Module and Ship Control Issues](#module-issues)
16. [Navigation and Movement Problems](#navigation-problems)
17. [Inventory and Cargo Issues](#inventory-issues)
18. [Fleet and Relay Problems](#fleet-problems)
19. [Performance and Memory Issues](#performance-issues)
20. [Configuration Problems](#config-problems)
21. [ISXEVE Bugs and Limitations](#isxeve-bugs)
22. [Common Scripting Mistakes](#scripting-mistakes)
23. [Recovery and Failsafe Strategies](#recovery-strategies)
24. [Debugging Workflows](#debugging-workflows)
25. [Production Deployment Issues](#deployment-issues)

---

## Logging Frameworks

**Architectural context:** For how these loggers fit into EVEBot and Tehbot's startup sequence, object hierarchy, and `#include` chain, see [18_Bot_Architecture_Analysis.md](18_Bot_Architecture_Analysis.md) under Project Structure and Initialization System.

### Why Logging Matters

**Critical for EVE bots because:**
- Scripts run for hours/days without supervision
- Need post-mortem analysis when things go wrong
- Performance bottlenecks must be identified
- Fleet coordination requires synchronized logs
- CCP detections need to be diagnosed

### obj_Logger Pattern (EVEBot)

EVEBot uses a sophisticated logging system:

```lavishscript
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

```lavishscript
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

## Log Levels and Filtering

### EVEBot Log Levels

```lavishscript
; Define log level constants
#define LOG_MINOR       0    ; Minor info - log only, don't print
#define LOG_STANDARD    1    ; Standard - log and print to screen
#define LOG_CRITICAL    2    ; Critical - log, print, alert
#define LOG_ECHOTOO     3    ; Echo to console too
#define LOG_DEBUG       4    ; Debug logging (conditional)
```

### Log Method with Levels

```lavishscript
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

```lavishscript
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

```lavishscript
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

## Console Output and Echo

### Basic Echo

**InnerSpace console output:**

```lavishscript
; Simple console output
echo "Hello World"

; With variables
echo "Target: ${Entity[${targetID}].Name} at ${Entity[${targetID}].Distance}m"

; Formatted
echo "Shield: ${MyShip.ShieldPct.Precision[1]}%"
```

### Echo with Formatting

```lavishscript
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

```lavishscript
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

```lavishscript
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

## File-Based Logging

### Redirect to File

```lavishscript
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

See [Example 1: Complete Logging System](#example-1-complete-logging-system) for a production-grade logger (`obj_ComprehensiveLogger`) that includes `Initialize`, `WriteHeader`, `Log`, and `LogError` plus level filtering, color-coded output, and file rotation.

### Separate Log Files by Purpose

```lavishscript
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

```lavishscript
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

## Performance Profiling

### Timestamp-Based Profiling

The basic timestamp profiler (`Start`/`End` methods with `collection:float` timings and a >1.0s slow-operation warning) is documented in its canonical, production-grade form as `obj_PerformanceProfiler` under [Example 2: Performance Profiler](#example-2-performance-profiler). That version adds `TotalTimes`, `CallCounts`, and `PrintReport` on top of the same core pattern.

### Frame Time Profiling

```lavishscript
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

The standalone call-counting pattern (`collection:int` with create-if-missing + increment + iterator-based `PrintStats`) is integrated into the canonical `obj_PerformanceProfiler` under [Example 2: Performance Profiler](#example-2-performance-profiler). That version combines call counting with start/end timing and a `PrintReport` method.

### Memory Usage Tracking

LavishScript does not expose script-memory diagnostics -- there is no `Script.MemUsage` member or equivalent. If you need to track your bot's memory footprint over time, use OS-level tools (Task Manager, Process Explorer, `Get-Process` in PowerShell) against the InnerSpace host process. Spikes in that number correlate with collections that are not being cleared, event atoms that are not being detached, or unbounded log buffers -- the common LavishScript leak sources.

For script-level leak detection without true memory numbers, track collection sizes as a proxy: log `${MyCache.Used}`, `${RegisteredEvents.Used}`, etc. on a periodic tick and look for unbounded growth.
```

---

## Common Debugging Patterns

### Pattern 1: Trace Logging

**Log every state transition:**

```lavishscript
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

```lavishscript
method DebugDump()
{
    echo "=== Debug Dump ==="
    echo "CurrentState: ${This.CurrentState}"
    echo "InSpace: ${Me.InSpace}"
    echo "InStation: ${Me.InStation}"
    echo "CargoUsed: ${MyShip.UsedCargoCapacity}"
    echo "CargoFree: ${Math.Calc[${MyShip.CargoCapacity} - ${MyShip.UsedCargoCapacity}]}"
    echo "Shield: ${MyShip.ShieldPct.Precision[1]}%"
    echo "Targets: ${Me.TargetCount}"
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

```lavishscript
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

```lavishscript
method CheckCondition()
{
    if ${MyShip.ShieldPct} < 20
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

```lavishscript
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

## Error Detection and Diagnosis

### ISXEVE Error Checking

The standard ISXEVE preflight checks (extension loaded, `IsSafe`, character exists, InSpace/InStation) are documented as a reusable function under [Session Validation](#session-validation) in the ISXEVE Debugging Tools section.

### Entity Validation

```lavishscript
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

```lavishscript
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

```lavishscript
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

## ISXEVE Debugging Tools

### ISXEVE:Debug_LogMsg

**Built-in ISXEVE debug logging:**

```lavishscript
; Enable debug logging
ISXEVE:Debug_LogMsg["MyModule", "Starting processing"]

; With categorization
ISXEVE:Debug_LogMsg["TargetManager", "Locking target ${targetID}"]
ISXEVE:Debug_LogMsg["Movement", "Warping to ${bookmark}"]
ISXEVE:Debug_LogMsg["Combat", "Activating weapons on ${Entity[${targetID}].Name}"]
```

### Disabling ISXEVE's Entity Cache

**Reference implementation:** see `EVEBot/Branches/Stable/EVEBot.iss` (near the top of `function main`).

ISXEVE caches entity query results between calls to reduce API cost. This is almost always what you want, but it can mask a class of bugs where your script thinks an entity is still on grid when the game client has already removed it. When you suspect a stale-cache issue -- e.g. `Entity[${id}](exists)` returns TRUE for something the overview clearly no longer shows -- disable the cache at startup to force every query to go fresh to the game:

```lavishscript
function main()
{
    Turbo 4000
    ISXEVE:Debug_SetEntityCacheDisabled[TRUE]   ; debugging only
    ; ... rest of main ...
}
```

Trade-off: every entity query now performs full API work, which slows down tight loops noticeably (especially multi-pulse `EVE:QueryEntities` iterations). Leave this on only while diagnosing a suspected stale-cache bug; re-enable caching for production by removing the call or passing `FALSE`.

### ISXEVE Diagnostics

```lavishscript
function PrintISXEVEDiagnostics()
{
    echo "=== ISXEVE Diagnostics ==="
    echo "Version: ${ISXEVE.Version}"
    echo "IsReady: ${ISXEVE.IsReady}"
    echo "IsSafe: ${ISXEVE.IsSafe}"
    echo "=========================="
}
```

### Session Validation

```lavishscript
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

## Real-Time Monitoring

> **📡 IRC Remote Monitoring:** For remote bot monitoring and alerting, IRC can send critical log messages, status updates, and alerts to an IRC channel. See File 28 (Relay_System_and_IPC.md) IRC Bridge Integration section and `__CRITICAL_NEWEST_ISXIM_Reference.md` for implementation details. EVEBot already implements this pattern (lines 173-177).

### Status Display

```lavishscript
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
        echo "│ Shield: ${MyShip.ShieldPct.Precision[0]}%   Armor: ${MyShip.ArmorPct.Precision[0]}%"
        echo "│ Cap: ${MyShip.CapacitorPct.Precision[0]}%      Cargo: ${Ship.CargoFull.Precision[0]}%"
        echo "├─────────────────────────────────────┤"
        echo "│ Targets: ${Me.TargetCount}/${Me.MaxLockedTargets}"
        echo "│ Uptime: ${Math.Calc[${Script.RunningTime}/1000/60].Int} minutes"
        echo "└─────────────────────────────────────┘"
    }

    variable int LastStatusPrint = 0

    method Pulse()
    {
        ; Update every 5 seconds. Gate on elapsed-since-last rather than
        ; `RunningTime % 5000 < 100` -- the modulo form is fragile: if the
        ; pulse rate is faster than 100 ms it fires multiple times per
        ; window; if slower, it skips the window entirely.
        if ${Math.Calc[${Script.RunningTime} - ${This.LastStatusPrint}]} >= 5000
        {
            This.LastStatusPrint:Set[${Script.RunningTime}]
            This:PrintStatus
        }
    }
}
```

### Metrics Collection

```lavishscript
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

## Troubleshooting Guide

### Common Issues and Solutions

#### Issue: Script Won't Start

The startup diagnostic checklist (ISXEVE loaded, character in game, ready state) follows the same validation pattern as [Session Validation](#session-validation) above. For production use, add a config-directory existence check (`Script.CurrentDirectory.FileExists["config"]`) with auto-creation as a post-validation step.

#### Issue: Entity Not Found

Use `ValidateEntity(entityID)` from [Entity Validation](#entity-validation) to check whether the entity exists, then `DebugEntity(entityID)` from [Pattern 3: Entity Debugging](#pattern-3-entity-debugging) to dump its state if it does.

**If the entity ID is 0:** the variable was never initialized or the entity despawned before its ID was stored.

**If the entity genuinely doesn't exist**, common causes are:
1. Entity despawned (asteroid mined out, NPC killed)
2. Entity warped away
3. You warped away (left the grid)
4. Grid changed (session change)

**Solution:** Re-query for a new entity using `EVE:QueryEntities` or `EVE:GetEntities`.

#### Issue: Targeting Fails

The target-lock diagnostic checklist (TargetCount vs MaxLockedTargets, entity exists, IsLockedTarget, BeingTargeted, Distance vs MaxTargetRange) is implemented as a production-grade `SafeLockTarget` function under [Problem 2: Target Lock Fails Silently](#problem-2-target-lock-fails-silently). That version performs all the same checks AND actually attempts the lock with verification -- use it both for diagnosis and as the canonical lock helper.

#### Issue: Slow Performance

```lavishscript
function DiagnosePerformance()
{
    echo "=== Performance Diagnostics ==="

    ; Check script runtime
    variable float runtime = ${Math.Calc[${Script.RunningTime} / 1000]}
    echo "Script Uptime: ${runtime.Precision[0]}s"

    ; (Memory usage is not exposed by LavishScript -- use OS-level tools
    ; like Process Explorer or PowerShell Get-Process against the InnerSpace
    ; host if you need a real memory number. As a script-side proxy, log
    ; collection sizes -- unbounded growth in any `.Used` count is the
    ; usual signature of a leak.)

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

## Best Practices

### 1. Log Levels Appropriately

```lavishscript
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

```lavishscript
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

```lavishscript
; GOOD - de-duplicated
method CheckShield()
{
    variable static int lastShieldPct = -1

    if ${MyShip.ShieldPct.Int} != ${lastShieldPct}
    {
        Logger:Log["Shield: ${MyShip.ShieldPct.Precision[0]}%"]
        lastShieldPct:Set[${MyShip.ShieldPct.Int}]
    }
}

; BAD - logs every frame
method CheckShield()
{
    Logger:Log["Shield: ${MyShip.ShieldPct.Precision[0]}%"]
}
```

### 4. Use Structured Logging

```lavishscript
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

```lavishscript
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

```lavishscript
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

```lavishscript
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

## Complete Examples

### Example 1: Complete Logging System

```lavishscript
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

```lavishscript
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

## Common Problems and Solutions

---

## Entity and Targeting Problems

### Problem 1: Entity Doesn't Exist After Query

**Symptom:**
```lavishscript
EVE:QueryEntities[Targets, "CategoryID = CATEGORYID_ENTITY"]
Entity[${Targets.Get[1]}]:LockTarget    ; ERROR: Entity doesn't exist!
```

**Cause:** Entity despawned between query and action (asteroid mined out, NPC killed, warp away)

**Solution:**

```lavishscript
; ALWAYS validate before acting
method LockEntity(int64 entityID)
{
    ; Validate exists
    if !${Entity[${entityID}](exists)}
    {
        Logger:Log["Entity ${entityID} no longer exists"]
        return FALSE
    }

    ; Validate ID is still valid
    if ${Entity[${entityID}].ID} == 0
    {
        Logger:Log["Entity ${entityID} has ID=0 (despawned)"]
        return FALSE
    }

    ; Now safe to lock
    Entity[${entityID}]:LockTarget
    return TRUE
}
```

### Problem 2: Target Lock Fails Silently

**Symptom:** Call `LockTarget` but target never locks

**Causes:**
- Already at max locked targets
- Target out of range
- Target not targetable (structure, etc.)
- Ship ECM jammed

**Solution:**

```lavishscript
method SafeLockTarget(int64 targetID)
{
    ; Check target count
    if ${Me.TargetCount} >= ${Me.MaxLockedTargets}
    {
        Logger:Log["At max targets (${Me.TargetCount}/${Me.MaxLockedTargets})"]
        return FALSE
    }

    ; Validate entity
    if !${Entity[${targetID}](exists)}
    {
        Logger:Log["Entity doesn't exist"]
        return FALSE
    }

    ; Check already locked/targeting
    if ${Entity[${targetID}].IsLockedTarget}
    {
        return TRUE    ; Already locked
    }

    if ${Entity[${targetID}].BeingTargeted}
    {
        Logger:Log["Already targeting, waiting..."]
        return FALSE    ; Wait for lock to complete
    }

    ; Check range
    if ${Entity[${targetID}].Distance} > ${MyShip.MaxTargetRange}
    {
        Logger:Log["Out of range: ${Entity[${targetID}].Distance}m > ${MyShip.MaxTargetRange}m"]
        return FALSE
    }

    ; Attempt lock
    Entity[${targetID}]:LockTarget

    ; Wait briefly and verify
    wait 5

    if ${Entity[${targetID}].BeingTargeted}
    {
        return TRUE    ; Lock initiated successfully
    }

    Logger:Log["Lock failed - possible jam or targeting error"]
    return FALSE
}
```

### Problem 3: QueryEntities Returns Too Many Results

**Symptom:** Script lags when querying entities in busy systems

**Cause:** Inefficient query - returns thousands of entities

**Solution:**

```lavishscript
; BAD - returns ALL entities
EVE:QueryEntities[AllStuff]    ; Thousands of results!

; GOOD - specific query
EVE:QueryEntities[Rats, "CategoryID = CATEGORYID_ENTITY && IsNPC"]

; BETTER - even more specific
EVE:QueryEntities[Rats, "CategoryID = CATEGORYID_ENTITY && IsNPC && Distance < 150000"]

; BEST - cache and reuse
variable index:entity CachedRats
variable float LastQueryTime = 0

function GetRats()
{
    ; Only re-query every 5 seconds
    if ${Math.Calc[${Time.Timestamp} - ${LastQueryTime}]} > 5
    {
        EVE:QueryEntities[CachedRats, "CategoryID = CATEGORYID_ENTITY && IsNPC && Distance < 150000"]
        LastQueryTime:Set[${Time.Timestamp}]
    }

    return ${CachedRats}
}
```

### Problem 4: Entity.Distance Suddenly Changes Massively

**Symptom:** Entity shows 10km away, next frame shows 250,000km

**Cause:** Grid change or warp - entity moved to different location

**Solution:**

```lavishscript
method ValidateEntityDistance(int64 entityID, float maxDistance)
{
    if !${Entity[${entityID}](exists)}
    {
        return FALSE
    }

    ; Check reasonable distance
    if ${Entity[${entityID}].Distance} > ${maxDistance}
    {
        Logger:Log["Entity ${Entity[${entityID}].Name} moved out of range (${Entity[${entityID}].Distance}m)"]
        return FALSE
    }

    ; Check for grid change (sudden massive distance jump)
    variable static float lastDistance = 0

    if ${lastDistance} > 0
    {
        variable float distanceChange = ${Math.Calc[${Entity[${entityID}].Distance} - ${lastDistance}]}

        if ${Math.Calc[${distanceChange}].Abs} > 100000    ; 100km sudden change
        {
            Logger:Log["Grid change detected - entity moved ${distanceChange}m instantly"]
            lastDistance:Set[0]
            return FALSE
        }
    }

    lastDistance:Set[${Entity[${entityID}].Distance}]
    return TRUE
}
```

---

## Module and Ship Control Issues

### Problem 5: Module Won't Activate

**Symptom:** `Module:Click` does nothing

**Common Causes:**
1. Module offline
2. No capacitor
3. No target (for targeted modules)
4. No charges (for guns)
5. Module still cycling

**Complete Diagnostic:**

```lavishscript
function DiagnoseModuleActivation(int64 moduleID)
{
    if !${Me.GetModule[${moduleID}](exists)}
    {
        echo "✗ Module doesn't exist"
        return
    }

    variable module M = ${Me.GetModule[${moduleID}]}

    echo "=== Module Diagnostic ==="
    echo "Name: ${M.ToItem.Name}"
    echo "IsOnline: ${M.IsOnline}"
    echo "IsActive: ${M.IsActive}"
    echo "IsActivatable: ${M.IsActivatable}"
    echo "IsDeactivating: ${M.IsDeactivating}"
    echo "IsReloadingAmmo: ${M.IsReloadingAmmo}"
    echo "IsGoingOnline: ${M.IsGoingOnline}"
    echo "CurrentCharges: ${M.Charge.Quantity}"
    echo "Ship Cap: ${MyShip.CapacitorPct.Precision[0]}%"
    echo "========================="

    ; Specific fixes
    if !${M.IsOnline}
    {
        echo "FIX: Module OFFLINE - put online manually or script:Module:SetOnline"
    }

    if ${M.IsActive}
    {
        echo "INFO: Module already active"
    }

    if !${M.IsActivatable}
    {
        echo "PROBLEM: Module not activatable"

        ; Check if needs target
        if ${M.ToItem.Group.Find["Remote"](exists)} || ${M.ToItem.Group.Find["Weapon"](exists)}
        {
            if ${Me.TargetCount} == 0
            {
                echo "FIX: Module needs target, none locked"
            }
        }

        ; Check capacitor
        if ${MyShip.CapacitorPct} < 5
        {
            echo "FIX: Insufficient capacitor"
        }

        ; Check charges
        if ${M.ToItem.Group.Find["Projectile"](exists)} || ${M.ToItem.Group.Find["Hybrid"](exists)}
        {
            if ${M.Charge.Quantity} == 0
            {
                echo "FIX: Out of ammo - reload"
            }
        }
    }

    if ${M.IsReloadingAmmo}
    {
        echo "INFO: Module reloading - wait ${Math.Calc[10 - ${M.ReloadTimeLeft}].Int}s"
    }
}
```

### Problem 6: Weapons Don't Shoot at Target

**Symptom:** Modules activate but don't apply to target

**Cause:** Target not locked, or wrong target active

**Solution:**

```lavishscript
method EngageTarget(int64 targetID)
{
    ; Lock target first
    if !${Entity[${targetID}].IsLockedTarget}
    {
        Logger:Log["Target not locked - locking first"]
        Entity[${targetID}]:LockTarget
        wait 50 ${Entity[${targetID}].IsLockedTarget}

        if !${Entity[${targetID}].IsLockedTarget}
        {
            Logger:Log["Failed to lock target"]
            return FALSE
        }
    }

    ; Make active target
    Entity[${targetID}]:MakeActiveTarget
    wait 5

    ; Verify active
    if ${Me.ActiveTarget.ID} != ${targetID}
    {
        Logger:Log["Failed to make target active"]
        return FALSE
    }

    ; Now activate weapons
    Ship:Activate_Weapons

    return TRUE
}
```

### Problem 7: Ship.Activate_* Methods Miss Some Modules

**Symptom:** Only some weapons activate when calling `Ship:Activate_Weapons`

**Cause:** EVEBot Ship object methods are helper methods that may not find all modules

**Solution:**

```lavishscript
; Manual activation - more reliable
method ActivateAllWeapons()
{
    variable iterator Module
    MyShip.Modules:GetIterator[Module]

    if ${Module:First(exists)}
    {
        do
        {
            ; Check if weapon module
            if ${Module.Value.ToItem.Group.Find["Projectile Weapon"](exists)} ||
               ${Module.Value.ToItem.Group.Find["Energy Weapon"](exists)} ||
               ${Module.Value.ToItem.Group.Find["Hybrid Weapon"](exists)} ||
               ${Module.Value.ToItem.Group.Find["Missile Launcher"](exists)}
            {
                ; Activate if not active
                if ${Module.Value.IsActivatable} && !${Module.Value.IsActive}
                {
                    Module.Value:Click
                    wait 1
                }
            }
        }
        while ${Module:Next(exists)}
    }
}
```

---

## Navigation and Movement Problems

### Problem 8: WarpTo Fails Silently

**Symptom:** Call `WarpTo` but ship doesn't warp

**Causes:**
- Warp scrambled
- Warp disrupted
- Not aligned
- Insufficient capacitor
- Docking request active

**Solution:**

```lavishscript
method SafeWarpTo(int64 destID, int distance)
{
    ; Check if can warp
    if !${Me.ToEntity.CanWarp}
    {
        Logger:Log["Cannot warp - likely scrambled/disrupted"]

        ; Check for scrams
        variable index:entity Entities
        EVE:QueryEntities[Entities, "IsTargetingMe && Distance < 50000"]

        if ${Entities.Used} > 0
        {
            Logger:Log["Hostile within 50km - likely scrambled"]
        }

        return FALSE
    }

    ; Check capacitor
    if ${MyShip.CapacitorPct} < 10
    {
        Logger:Log["Insufficient capacitor for warp"]
        return FALSE
    }

    ; Cancel any active approach/orbit
    EVE:Execute[CmdStopShip]
    wait 5

    ; Initiate warp
    Entity[${destID}]:WarpTo[${distance}]

    ; Wait for warp to start
    wait 50 ${Me.ToEntity.Mode} == 3

    if ${Me.ToEntity.Mode} == 3
    {
        Logger:Log["Warp initiated successfully"]
        return TRUE
    }

    Logger:Log["Warp failed to initiate"]
    return FALSE
}
```

### Problem 9: Autopilot Gets Stuck

**Symptom:** EVE:Execute[CmdSetDestination] sets destination but ship doesn't autopilot

**Cause:** EVE's autopilot must be manually activated, or use Navigator for scripted autopilot

**Solution:**

```lavishscript
; Option 1: Activate EVE's built-in autopilot (not recommended - slow)
EVE:Execute[CmdSetDestination, ${destinationID}]
EVE:Execute[CmdToggleAutopilot]    ; Toggle on

; Option 2: Use ISXEVE Navigator (recommended - faster, more control)
Navigator:Destination:Set[${destinationID}]
Navigator:FlyToDestination

; Option 3: Manual autopilot with control (best for bots)
function AutopilotTo(int destinationSystemID)
{
    variable index:int Path
    EVE:GetToDestinationPath[Path]

    variable iterator System
    Path:GetIterator[System]

    if ${System:First(exists)}
    {
        do
        {
            ; Warp to stargate
            call WarpToStargate ${System.Value}

            ; Jump through
            call JumpThroughGate

            ; Wait in new system
            wait 50 ${Me.InSpace}
        }
        while ${System:Next(exists)}
    }
}
```

### Problem 10: Approach Never Reaches Target

**Symptom:** Ship approaches target forever, never stops

**Cause:** Default approach range is 0m (unreachable for large objects)

**Solution:**

```lavishscript
; BAD - approaches to 0m (impossible)
Entity[${targetID}]:Approach

; GOOD - specify reasonable range
Entity[${targetID}]:Approach[2500]    ; Stop at 2500m

; BETTER - approach with timeout
function ApproachWithTimeout(int64 targetID, int range, int timeoutSeconds)
{
    Entity[${targetID}]:Approach[${range}]

    variable float startTime = ${Time.Timestamp}

    ; Wait until in range or timeout
    while ${Entity[${targetID}].Distance} > ${range}
    {
        ; Check timeout
        if ${Math.Calc[${Time.Timestamp} - ${startTime}]} > ${timeoutSeconds}
        {
            Logger:Log["Approach timeout after ${timeoutSeconds}s"]
            EVE:Execute[CmdStopShip]
            return FALSE
        }

        ; Check still exists
        if !${Entity[${targetID}](exists)}
        {
            Logger:Log["Target disappeared during approach"]
            EVE:Execute[CmdStopShip]
            return FALSE
        }

        wait 10
    }

    Logger:Log["Reached target"]
    EVE:Execute[CmdStopShip]
    return TRUE
}
```

---

## Inventory and Cargo Issues

### Problem 11: Item:MoveTo Fails

**Symptom:** `Item:MoveTo` returns but item doesn't move

**Causes:**
- Target cargo full
- Item already in destination
- Cargo not open
- Item is fitted module
- Insufficient permissions

**Solution:**

```lavishscript
method SafeMoveTo(int64 itemID, string destination, int quantity)
{
    ; Locate the item by scanning the ship's cargo index.
    variable index:item cargo
    variable iterator ci
    variable int foundIdx = 0
    MyShip:GetCargo[cargo]
    cargo:GetIterator[ci]
    if ${ci:First(exists)}
    {
        do
        {
            if ${ci.Value.ID.Equal[${itemID}]}
            {
                foundIdx:Set[${ci.Key}]
                break
            }
        }
        while ${ci:Next(exists)}
    }

    if ${foundIdx} == 0
    {
        Logger:Log["Item ${itemID} not found in cargo"]
        return FALSE
    }

    variable item TheItem = ${cargo.Get[${foundIdx}]}

    ; Check destination capacity
    switch ${destination}
    {
        case MyHangar
            variable float hangarFree = ${Math.Calc[${Me.Station.ItemHangar.Capacity} - ${Me.Station.ItemHangar.UsedCapacity}]}

            if ${TheItem.Volume} > ${hangarFree}
            {
                Logger:Log["Hangar full - need ${TheItem.Volume}m3, have ${hangarFree}m3"]
                return FALSE
            }
            break

        case OtherCargo
            ; Assumes other cargo (Orca fleet hangar, etc.) is open
            if !${EVEWindow[ByName, "Fleet Hangar"](exists)}
            {
                Logger:Log["Other cargo window not open"]
                return FALSE
            }
            break
    }

    ; Attempt move
    Logger:Log["Moving ${TheItem.Name} x${quantity} to ${destination}"]
    TheItem:MoveTo[${destination}, ${quantity}]

    ; Wait for move
    wait 20

    ; Verify moved -- re-scan cargo for the ID; if the remaining quantity is
    ; below the requested move, consider it successful (partial stacks move OK).
    variable int remaining = 0
    MyShip:GetCargo[cargo]
    cargo:GetIterator[ci]
    if ${ci:First(exists)}
    {
        do
        {
            if ${ci.Value.ID.Equal[${itemID}]}
            {
                remaining:Set[${ci.Value.Quantity}]
                break
            }
        }
        while ${ci:Next(exists)}
    }
    if ${remaining} < ${quantity}
    {
        Logger:Log["Move successful"]
        return TRUE
    }

    Logger:Log["Move may have failed - item still in cargo"]
    return FALSE
}
```

### Problem 12: Can't Access Station Hangar

**Symptom:** `Me.Station.ItemHangar` returns NULL

**Cause:** Not docked, or hangar not loaded yet

**Solution:**

```lavishscript
function WaitForHangar(int timeoutSeconds)
{
    if !${Me.InStation}
    {
        Logger:Log["Not in station"]
        return FALSE
    }

    variable float startTime = ${Time.Timestamp}

    ; Wait for hangar to load
    while !${Me.Station.ItemHangar(exists)}
    {
        if ${Math.Calc[${Time.Timestamp} - ${startTime}]} > ${timeoutSeconds}
        {
            Logger:Log["Timeout waiting for hangar"]
            return FALSE
        }

        wait 10
    }

    ; Hangar loaded
    wait 5    ; Extra safety wait
    return TRUE
}

; Usage
function UnloadCargo()
{
    ; Dock first
    call Dock

    ; Wait for hangar
    call WaitForHangar 30
    if !${Return}
    {
        Logger:Log["Failed to access hangar"]
        return
    }

    ; Now safe to unload using modern inventory API
    if ${EVEWindow[Inventory](exists)}
    {
        ; Modern API: Transfer cargo items to hangar
        variable index:item cargoItems
        EVEWindow[Inventory].Child[ShipCargo]:GetItems[cargoItems]

        variable iterator item
        cargoItems:GetIterator[item]

        if ${item:First(exists)}
        {
            do
            {
                item.Value:MoveTo[MyItemHangar]
                wait 5
            }
            while ${item:Next(exists)}
        }
    }
}
```

### Problem 13: Cargo Operations Fail After Docking/Undocking

**Symptom:** Cargo operations work in space, fail after docking

**Cause:** Inventory window state may change when docking/undocking

**Solution:**

```lavishscript
; Modern API: Always ensure inventory window is open and valid
function GetCargoSpace()
{
    ; Ensure inventory window is open
    if !${EVEWindow[Inventory](exists)}
    {
        EVE:Execute[OpenInventory]
        wait 20
    }

    if !${EVEWindow[Inventory](exists)}
    {
        Logger:Log["Cannot access inventory"]
        return 0
    }

    ; Check if we're in station or space
    if ${Me.InStation}
    {
        ; In station - cargo still accessed via ShipCargo
        return ${EVEWindow[Inventory].Child[ShipCargo].FreeSpace}
    }
    else
    {
        ; In space - use ShipCargo
        return ${EVEWindow[Inventory].Child[ShipCargo].FreeSpace}
    }
}

; Usage
call GetCargoSpace
variable float cargoFree = ${Return}
Logger:Log["Cargo free: ${cargoFree}m3"]
```

---

## Fleet and Relay Problems

### Problem 14: Relay Events Not Received

**Symptom:** Relay broadcasts but other sessions don't respond

**Causes:**
- Event not registered
- Atom not attached
- Wrong event name
- Typo in relay command

**Solution:**

```lavishscript
; Sender (Master)
method BroadcastPrimary(int64 targetID)
{
    ; Make sure using correct event name
    relay all -event FC_Primary_Target ${targetID}
    ;                 ^^^^^^^^^^^^^^^^ Must match exactly
}

; Receiver (Slave) - MUST match exactly
method Initialize()
{
    ; Register event
    LavishScript:RegisterEvent[FC_Primary_Target]
    ;                            ^^^^^^^^^^^^^^^^ Must match exactly

    ; Attach handler
    Event[FC_Primary_Target]:AttachAtom[This:OnPrimaryTarget]
    ;     ^^^^^^^^^^^^^^^^                    ^^^^^^^^^^^^^^
}

atom OnPrimaryTarget(int64 targetID)
{
    echo "Received primary: ${targetID}"
    Entity[${targetID}]:LockTarget
}

; DEBUG: Verify event registration.
; LavishScript does not expose an event-enumeration API; track your
; registrations script-side by appending to a collection at RegisterEvent
; time, then iterate that collection here.
variable index:string RegisteredEvents

method RegisterBotEvent(string eventName)
{
    LavishScript:RegisterEvent[${eventName}]
    RegisteredEvents:Insert[${eventName}]
}

function CheckEvents()
{
    echo "Registered events:"
    variable iterator Ev
    RegisteredEvents:GetIterator[Ev]

    if ${Ev:First(exists)}
    {
        do
        {
            echo "  ${Ev.Value}"
        }
        while ${Ev:Next(exists)}
    }
}
```

### Problem 15: Relay Sends But Parameter Missing

**Symptom:** Event fires but parameters are NULL or empty

**Cause:** Incorrect parameter types or escaping

**Solution:**

```lavishscript
; WRONG - string parameter not quoted
relay all -event MyEvent ${someString}
; Receiver gets: empty or just first word

; RIGHT - quote string parameters
relay all -event MyEvent "${someString}"

; Example: Target name with spaces
relay all -event TargetInfo "${Entity[${targetID}].Name}" ${Entity[${targetID}].Distance}

; Receiver
atom OnTargetInfo(string targetName, float distance)
{
    ; Now targetName is complete, even with spaces
    echo "Target: ${targetName} at ${distance}m"
}
```

### Problem 16: Master-Slave Desync

**Symptom:** Slaves get out of sync with master

**Cause:** Missed heartbeats or relay messages

**Solution:**

```lavishscript
objectdef obj_FleetSync
{
    variable time LastMasterHeartbeat
    variable int HeartbeatTimeout = 10    ; Seconds

    method Initialize()
    {
        LavishScript:RegisterEvent[Master_Heartbeat]
        Event[Master_Heartbeat]:AttachAtom[This:OnHeartbeat]
    }

    variable int LastHeartbeat = 0

    ; Master: Broadcast heartbeat
    method MasterPulse()
    {
        if ${Config.Fleet.IsMaster}
        {
            ; Send state every 2 seconds. Gate on elapsed-since-last, not
            ; on `Timestamp % 2` -- the modulo form fires on every pulse
            ; during the matching second (Timestamp has 1s resolution).
            if ${Math.Calc[${Time.Timestamp} - ${This.LastHeartbeat}]} >= 2
            {
                This.LastHeartbeat:Set[${Time.Timestamp}]
                relay all -event Master_Heartbeat "${This.CurrentState}" ${Time.Timestamp}
            }
        }
    }

    ; Slave: Receive heartbeat
    atom OnHeartbeat(string masterState, float timestamp)
    {
        This.LastMasterHeartbeat:Set[${timestamp}]
        This.MasterState:Set[${masterState}]
    }

    ; Slave: Check for timeout
    method CheckMasterAlive()
    {
        if ${Config.Fleet.IsMaster}
        {
            return TRUE    ; We are master
        }

        variable float timeSinceHeartbeat = ${Math.Calc[${Time.Timestamp} - ${This.LastMasterHeartbeat.Timestamp}]}

        if ${timeSinceHeartbeat} > ${This.HeartbeatTimeout}
        {
            Logger:Log["Master timeout - ${timeSinceHeartbeat}s since last heartbeat", LOG_CRITICAL]
            Logger:Log["Entering autonomous mode"]

            ; Go autonomous
            This:EnterAutonomousMode
            return FALSE
        }

        return TRUE
    }
}
```

---

## Performance and Memory Issues

### Problem 17: Script Slows Down Over Time

**Symptom:** Script runs fast initially, becomes sluggish after hours

**Cause:** Memory leak - collections growing unbounded

**Solution:**

```lavishscript
; BAD - collection never cleared
objectdef obj_LeakyBot
{
    variable collection:entity AllTargetsEverSeen

    method Pulse()
    {
        variable index:entity CurrentTargets
        EVE:QueryEntities[CurrentTargets, "CategoryID = CATEGORYID_ENTITY"]

        variable iterator Target
        CurrentTargets:GetIterator[Target]

        if ${Target:First(exists)}
        {
            do
            {
                ; Leak! Adds forever, never removes
                This.AllTargetsEverSeen:Set[${Target.Value.ID}, ${Target.Value}]
            }
            while ${Target:Next(exists)}
        }
    }
}

; GOOD - periodic cleanup
objectdef obj_CleanBot
{
    variable collection:entity RecentTargets
    variable time LastCleanup

    method Pulse()
    {
        ; Periodic cleanup
        if ${Math.Calc[${Time.Timestamp} - ${This.LastCleanup.Timestamp}]} > 300    ; Every 5 minutes
        {
            This.RecentTargets:Clear
            This.LastCleanup:Set[${Time.Timestamp}]
            Logger:Log["Cleaned target cache"]
        }

        ; ... normal processing ...
    }
}
```

### Problem 18: Entity Queries Lag

**Symptom:** `QueryEntities` takes seconds to complete

**Cause:** Too many entities on grid (1000+), or query too broad

**Solution:**

```lavishscript
; BAD - queries all entities every frame
method Pulse()
{
    variable index:entity AllStuff
    EVE:QueryEntities[AllStuff]    ; Can be thousands!

    ; ... process ...
}

; GOOD - cache and throttle
objectdef obj_CachedQueries
{
    variable index:entity CachedRats
    variable float LastRatQuery = 0
    variable float RatQueryInterval = 5.0    ; Seconds

    method GetRats()
    {
        ; Only query every 5 seconds
        if ${Math.Calc[${Time.Timestamp} - ${This.LastRatQuery}]} > ${This.RatQueryInterval}
        {
            ; Specific query
            EVE:QueryEntities[This.CachedRats, "CategoryID = CATEGORYID_ENTITY && IsNPC && Distance < 150000"]
            This.LastRatQuery:Set[${Time.Timestamp}]
            Logger:Log["Queried ${This.CachedRats.Used} rats"]
        }

        return ${This.CachedRats}
    }
}
```

### Problem 19: High Memory Usage

**Symptom:** Script uses 200MB+ memory

**Causes:**
- Too many cached objects
- Large index/collection variables
- Storing full entity objects instead of IDs

**Solution:**

```lavishscript
; BAD - stores full entity objects
variable collection:entity AllRats    ; Each entity is large object

; GOOD - stores only IDs
variable collection:int64 AllRatIDs   ; Just 64-bit integers

; Access pattern
method ProcessRat(int64 ratID)
{
    ; Retrieve entity when needed
    if ${Entity[${ratID}](exists)}
    {
        ; Process
    }
    else
    {
        ; Remove from collection
        AllRatIDs:Remove[${ratID}]
    }
}
```

---

## Configuration Problems

### Problem 20: Config Changes Don't Save

**Symptom:** Change config values but they reset on reload

**Cause:** Config not saved to file

**Solution:**

```lavishscript
method SetConfigValue(string value)
{
    ; Update in memory
    Config.MyValue:Set[${value}]

    ; MUST save to persist!
    Config:Save

    ; Or use BaseConfig
    BaseConfig:Save
}
```

### Problem 21: Config File Corrupt After Crash

**Symptom:** XML parse errors when loading config

**Cause:** Script crashed while writing config file

**Solution:**

```lavishscript
; Backup before saving
method SafeSave()
{
    ; Create backup
    variable string backupFile = "${CONFIG_FILE}.backup"

    ; Copy current to backup
    if ${CONFIG_PATH.FileExists[${CONFIG_FILE}]}
    {
        System:Copy["${CONFIG_PATH}/${CONFIG_FILE}", "${CONFIG_PATH}/${backupFile}"]
    }

    ; Save new config
    LavishSettings[MyBotSettings]:Export["${CONFIG_PATH}/${CONFIG_FILE}"]

    Logger:Log["Config saved (backup created)"]
}

; Recovery
method LoadConfig()
{
    if !${CONFIG_PATH.FileExists[${CONFIG_FILE}]}
    {
        Logger:Log["Config file missing - checking for backup"]

        variable string backupFile = "${CONFIG_FILE}.backup"

        if ${CONFIG_PATH.FileExists[${backupFile}]}
        {
            Logger:Log["Restoring from backup"]
            System:Copy["${CONFIG_PATH}/${backupFile}", "${CONFIG_PATH}/${CONFIG_FILE}"]
        }
    }

    LavishSettings[MyBotSettings]:Import["${CONFIG_PATH}/${CONFIG_FILE}"]
}
```

---

## ISXEVE Bugs and Limitations

### Known Issue 1: Me.GetTargets Returns Stale Targets

**Symptom:** `Me.GetTargets` includes targets that are no longer locked

**Workaround:**

```lavishscript
; Validate each target from GetTargets
variable index:entity Targets
Me:GetTargets[Targets]

variable iterator Target
Targets:GetIterator[Target]

if ${Target:First(exists)}
{
    do
    {
        ; ALWAYS validate IsLockedTarget
        if ${Target.Value.IsLockedTarget}
        {
            ; Safe to use
            This:ProcessTarget[${Target.Value.ID}]
        }
        else
        {
            Logger:Log["Target ${Target.Value.ID} in GetTargets but not locked (stale)"]
        }
    }
    while ${Target:Next(exists)}
}
```

### Known Issue 2: Module.IsActivatable Unreliable

**Symptom:** `Module.IsActivatable` returns TRUE but Click() fails

**Workaround:**

Use the canonical `ActivateModule` from [Module Activation Errors](#module-activation-errors) with two modifications: (1) **skip the `IsActivatable` check** -- it can return TRUE even when activation will fail, and (2) **add an `IsDeactivating` check** -- attempting activation while the module is mid-deactivation will silently fail. The root cause is that `IsActivatable` doesn't account for the deactivation-in-progress state.

### Known Issue 3: Entity.Distance Can Be Wrong After Warp

**Symptom:** After warping, entity distances show very large or 0

**Cause:** Grid change - entities not updated yet

**Workaround:**

```lavishscript
function WaitForGridLoad()
{
    ; After warp, wait for grid to stabilize
    wait 50    ; Basic wait

    ; Verify entities loaded
    variable int retries = 0

    while ${retries} < 10
    {
        variable index:entity NearbyStuff
        EVE:QueryEntities[NearbyStuff, "Distance < 150000"]

        if ${NearbyStuff.Used} > 0
        {
            ; Found entities - grid loaded
            return TRUE
        }

        wait 10
        retries:Inc
    }

    return FALSE
}

; Use after warping
function WarpAndSettle(int64 destID)
{
    Entity[${destID}]:WarpTo[0]
    wait 50 ${Me.ToEntity.Mode} == 3

    ; Wait for warp to complete
    wait 50 ${Me.ToEntity.Mode} != 3

    ; CRITICAL: Wait for grid to stabilize
    call WaitForGridLoad

    ; Now entity distances are reliable
}
```

---

## Common Scripting Mistakes

### Mistake 1: Not Checking Existence

```lavishscript
; WRONG - assumes entity exists
Entity[${targetID}]:LockTarget    ; CRASH if doesn't exist!

; RIGHT - always validate
if ${Entity[${targetID}](exists)}
{
    Entity[${targetID}]:LockTarget
}
```

### Mistake 2: Infinite Loops Without Wait

```lavishscript
; WRONG - freezes InnerSpace
while TRUE
{
    echo "Looping"
}

; RIGHT - include wait
while TRUE
{
    echo "Looping"
    wait 1    ; CRITICAL!
}
```

### Mistake 3: Variable Scope Confusion

```lavishscript
; WRONG - local variable, lost after function
function GetTarget()
{
    variable int64 targetID = ${Entity["CategoryID = CATEGORYID_ENTITY"].ID}
    return ${targetID}
}

call GetTarget
variable int64 myTarget = ${Return}    ; Works

echo ${targetID}    ; ERROR: targetID doesn't exist here!

; RIGHT - use return value
call GetTarget
variable int64 myTarget = ${Return}
echo ${myTarget}    ; Works
```

### Mistake 4: Forgetting to Detach Events

```lavishscript
; WRONG - memory leak
objectdef obj_LeakyObject
{
    method Initialize()
    {
        Event[MyEvent]:AttachAtom[This:OnMyEvent]
        ; No shutdown method!
    }
}

; RIGHT - always detach
objectdef obj_CleanObject
{
    method Initialize()
    {
        Event[MyEvent]:AttachAtom[This:OnMyEvent]
    }

    method Shutdown()
    {
        Event[MyEvent]:DetachAtom[This:OnMyEvent]
    }
}
```

### Mistake 5: Race Conditions with Async Operations

```lavishscript
; WRONG - doesn't wait for lock
Entity[${targetID}]:LockTarget
Ship:Activate_Weapons    ; Target not locked yet!

; RIGHT - wait for lock
Entity[${targetID}]:LockTarget
wait 50 ${Entity[${targetID}].IsLockedTarget}

if ${Entity[${targetID}].IsLockedTarget}
{
    Ship:Activate_Weapons
}
```

---

## Recovery and Failsafe Strategies

### Strategy 1: Automatic Recovery from Stuck States

```lavishscript
objectdef obj_StuckDetector
{
    variable string LastState = ""
    variable time StateChangeTime
    variable int StuckThresholdSeconds = 300    ; 5 minutes

    method CheckStuck()
    {
        ; Detect state change
        if !${This.CurrentState.Equal[${This.LastState}]}
        {
            This.LastState:Set["${This.CurrentState}"]
            This.StateChangeTime:Set[${Time.Timestamp}]
            return FALSE
        }

        ; Same state - check how long
        variable float timeInState = ${Math.Calc[${Time.Timestamp} - ${This.StateChangeTime.Timestamp}]}

        if ${timeInState} > ${This.StuckThresholdSeconds}
        {
            Logger:Log["STUCK DETECTED: In ${This.CurrentState} for ${timeInState}s", LOG_CRITICAL]
            This:RecoverFromStuck
            return TRUE
        }

        return FALSE
    }

    method RecoverFromStuck()
    {
        Logger:Log["Attempting automatic recovery"]

        ; Generic recovery actions
        switch ${This.CurrentState}
        {
            case WARPING
                ; Stuck warping - might be autopilot issue
                EVE:Execute[CmdStopShip]
                This.CurrentState:Set["IDLE"]
                break

            case MINING
                ; Stuck mining - might be targeting issue; unlock everything
                variable index:entity LockedTargets
                variable iterator T
                Me:GetTargets[LockedTargets]
                LockedTargets:GetIterator[T]
                if ${T:First(exists)}
                {
                    do
                    {
                        T.Value:UnlockTarget
                    }
                    while ${T:Next(exists)}
                }
                This.CurrentState:Set["IDLE"]
                break

            case HAULING
                ; Stuck hauling - go to station
                This.CurrentState:Set["DOCKING"]
                break

            default
                ; Unknown state - return to base
                This.CurrentState:Set["RETURN_HOME"]
                break
        }

        ; Reset state timer
        This.StateChangeTime:Set[${Time.Timestamp}]
    }
}
```

### Strategy 2: Dead Man's Switch

```lavishscript
objectdef obj_DeadMansSwitch
{
    variable time LastUserInput
    variable int TimeoutMinutes = 30

    method Pulse()
    {
        ; Check for user activity
        if ${Mouse.X} != ${This.LastMouseX} || ${Mouse.Y} != ${This.LastMouseY}
        {
            This.LastUserInput:Set[${Time.Timestamp}]
            This.LastMouseX:Set[${Mouse.X}]
            This.LastMouseY:Set[${Mouse.Y}]
        }

        ; Check timeout
        variable float minutesSinceInput = ${Math.Calc[(${Time.Timestamp} - ${This.LastUserInput.Timestamp}) / 60]}

        if ${minutesSinceInput} > ${This.TimeoutMinutes}
        {
            Logger:Log["Dead man's switch: No user input for ${minutesSinceInput} minutes", LOG_CRITICAL]
            This:EmergencyShutdown
        }
    }

    method EmergencyShutdown()
    {
        Logger:Log["Emergency shutdown initiated"]

        ; Dock if in space
        if ${Me.InSpace}
        {
            This:EmergencyDock
        }

        ; Pause script
        Script:Pause
    }
}
```

### Strategy 3: Periodic Restarts

```lavishscript
function main()
{
    ; Check runtime
    variable int maxRuntimeHours = 8

    while TRUE
    {
        ; Run bot
        call ProcessBotLogic

        ; Check runtime
        variable float hoursRunning = ${Math.Calc[${Script.RunningTime} / 1000 / 60 / 60]}

        if ${hoursRunning} >= ${maxRuntimeHours}
        {
            Logger:Log["Max runtime ${maxRuntimeHours}h reached - restarting"]

            ; Dock safely
            call SafeDock

            ; End script (launcher will restart)
            Script:End
        }

        wait 1
    }
}
```

---

## Debugging Workflows

### Workflow 1: Systematic Issue Diagnosis

```
1. REPRODUCE
   - Can you make it happen again?
   - What are exact steps?

2. ISOLATE
   - Does it happen in specific state?
   - Specific ship/fit?
   - Specific system/situation?

3. LOG
   - Add logging around problem area
   - Log all relevant variables
   - Log state transitions

4. VALIDATE
   - Are entities valid?
   - Is ISXEVE.IsSafe?
   - Are modules online?

5. SIMPLIFY
   - Comment out complex logic
   - Test minimal case
   - Add back complexity gradually

6. FIX
   - Add validation
   - Add error handling
   - Add fallback logic

7. TEST
   - Run for extended period
   - Test edge cases
   - Monitor logs
```

### Workflow 2: Performance Investigation

```
1. PROFILE
   - Add performance profiler
   - Identify slow methods

2. MEASURE
   - Time critical operations
   - Count iterations
   - Track memory usage

3. OPTIMIZE
   - Cache frequent queries
   - Reduce iteration count
   - Clear unused collections

4. VERIFY
   - Re-measure
   - Compare before/after
   - Monitor long-term
```

---

## Production Deployment Issues

### Issue: Script Works in Testing, Fails in Production

**Common Causes:**
- Different system/environment
- Different ship/fit
- Network lag
- Higher entity count

**Solution:**

```lavishscript
; Add environment detection
function DetectEnvironment()
{
    call CountEntities
    variable int entityCount = ${Return}

    echo "=== Environment Detection ==="
    echo "System: ${Me.SolarSystem.Name}"
    echo "Security: ${Me.SolarSystem.Security}"
    echo "Ship: ${MyShip.ToEntity.Name}"
    echo "Entities on Grid: ${entityCount}"
    echo "FPS: ${Display.FPS}"
    echo "============================="

    ; Adjust settings based on environment
    if ${entityCount} > 500
    {
        Logger:Log["High entity count - increasing query intervals"]
        This.QueryInterval:Set[10.0]    ; Slower queries
    }
}

function CountEntities()
{
    variable index:entity All
    EVE:QueryEntities[All]
    return ${All.Used}
}
```

### Issue: Fleet Coordination Breaks Down

**Symptom:** Master-slave coordination works 1-on-1, fails with 10+ slaves

**Cause:** Relay message flooding, race conditions

**Solution:**

```lavishscript
; Throttle broadcasts
objectdef obj_ThrottledBroadcaster
{
    variable float LastBroadcast = 0
    variable float BroadcastInterval = 0.5    ; Max 2 broadcasts/second

    method BroadcastTargets()
    {
        ; Check if too soon
        if ${Math.Calc[${Time.Timestamp} - ${This.LastBroadcast}]} < ${This.BroadcastInterval}
        {
            return    ; Skip this broadcast
        }

        ; Broadcast
        relay "all other" -event TargetUpdate "${targetData}"
        This.LastBroadcast:Set[${Time.Timestamp}]
    }
}

; Add sequence numbers to detect missed messages
method BroadcastWithSequence()
{
    This.SequenceNumber:Inc

    relay "all other" -event TargetUpdate ${This.SequenceNumber} "${targetData}"
}

atom OnTargetUpdate(int sequence, string targetData)
{
    ; Check for missed messages
    if ${sequence} != ${Math.Calc[${This.LastSequence} + 1]}
    {
        Logger:Log["Missed ${Math.Calc[${sequence} - ${This.LastSequence} - 1]} relay messages"]

        ; Request full state
        relay "${MasterName}" -event RequestFullState
    }

    This.LastSequence:Set[${sequence}]
    This:ProcessTargetData["${targetData}"]
}
```

---

## Summary

### Top 10 Most Common Issues

1. **Entity validation** - Always check `(exists)` before using
2. **Module activation** - Check IsOnline, cap, target, ammo
3. **Infinite loops** - Always include `wait` in loops
4. **Event handling** - Register events AND attach atoms
5. **Memory leaks** - Clear collections periodically
6. **Query performance** - Cache and throttle entity queries
7. **Race conditions** - Wait for async operations to complete
8. **Config persistence** - Must call Save() to persist changes
9. **Relay parameters** - Quote string parameters properly
10. **Grid changes** - Validate after warps/jumps

### Quick Diagnostic Checklist

```
□ Is ISXEVE loaded? (${ISXEVE(exists)})
□ Is game safe? (${ISXEVE.IsSafe})
□ Is character loaded? (${Me(exists)})
□ Are entities validated? (${Entity[ID](exists)})
□ Are events registered? (LavishScript:RegisterEvent)
□ Are atoms attached? (Event:AttachAtom)
□ Are atoms detached in shutdown? (Event:DetachAtom)
□ Do loops have waits? (wait 1 minimum)
□ Is config saved? (Config:Save or BaseConfig:Save)
□ Are collections cleared? (Collection:Clear periodically)
```
