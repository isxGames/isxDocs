# Performance and Timing - Bot Optimization Reference

**Purpose**: This file documents how to optimize EVE Online bot performance - timing, caching, query optimization, and resource management. Well-optimized bots are faster, more responsive, and less detectable.

**Target Reader**: AI learning to build efficient EVE bots

**Prerequisites**:
- Files 01-19 (Foundation, API, Main Loops, Decision Making, Error Handling)
- Understanding of performance concepts (caching, throttling, profiling)
- Knowledge of timing commands (wait, waitframe, Frame)

**Wiki References**:
- LavishScript timing: `LavishScriptWiki/LavishScript/Commands/wait.html`
- Script profiling: `LavishScriptWiki/DataType/script.html#EnableProfiling`
- Turbo mode: `LavishScriptWiki/LavishScript/Commands/turbo.html`

---

## Table of Contents

1. [Performance Overview](#performance-overview)
2. [Timing Fundamentals](#timing-fundamentals)
3. [Loop Optimization](#loop-optimization)
4. [Query Optimization](#query-optimization)
5. [Caching Strategies](#caching-strategies)
6. [CPU Usage Management](#cpu-usage-management)
7. [Memory Management](#memory-management)
8. [Profiling and Measurement](#profiling-and-measurement)
9. [Common Performance Bottlenecks](#common-performance-bottlenecks)
10. [Optimization Patterns](#optimization-patterns)
11. [Real Examples from Bots](#real-examples-from-bots)
12. [Performance Checklist](#performance-checklist)

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

### Performance Goals

- **Main loop**: 10-60 Hz
- **Entity queries**: < 100ms, cached for 1+ seconds
- **CPU usage**: < 10% average
- **Memory**: < 100MB
- **Response latency**: < 1 second

---

**File Complete**: Performance and timing optimization fully documented with real examples and best practices.

**Layer 4 COMPLETE!** All bot architecture patterns documented.
