# Advanced Scripting Patterns

**Purpose:** Production-grade patterns for script organization, main loops, state machines, error handling, relay communication, and configuration management
**Audience:** Intermediate to advanced scripters building complex automation

---

## Table of Contents

1. [Script Organization Patterns](#script-organization-patterns)
2. [Main Loop Patterns](#main-loop-patterns)
3. [Entity Processing Patterns](#entity-processing-patterns)
4. [State Machine Patterns](#state-machine-patterns)
5. [Error Handling Patterns](#error-handling-patterns)
6. [Timing and Wait Patterns](#timing-and-wait-patterns)
7. [Relay Communication Patterns](#relay-communication-patterns)
8. [Configuration Patterns](#configuration-patterns)
9. [Logging and Debug Patterns](#logging-and-debug-patterns)
10. [Common Anti-Patterns](#common-anti-patterns)

---

## Script Organization Patterns

### Standard Script Template

**Production-Ready Script Structure**:
```lavishscript
;==============================================================================
; Script Name: MyBot.iss
; Purpose: [Brief description]
; Author: [Your name]
; ; Last Modified: 2025-10-04
;==============================================================================

;==============================================================================
; DEPENDENCY CHECKS
;==============================================================================
#if !${ISXEVE(exists)}
    echo "ERROR: ISXEVE extension required"
    Script:End
#endif

;==============================================================================
; INCLUDES
;==============================================================================
#include MyBot/Libs/Targeting.iss
#include MyBot/Libs/Movement.iss
#include Common/Utilities.iss

;==============================================================================
; GLOBAL CONSTANTS
;==============================================================================
variable(global) string VERSION = "1.0.0"
variable(global) string SCRIPT_NAME = "MyBot"

; State constants
variable(global) string STATE_IDLE = "Idle"
variable(global) string STATE_COMBAT = "Combat"
variable(global) string STATE_RETURNING = "Returning"

; Timing constants (in deciseconds)
variable(global) int PULSE_DELAY = 20        ; 2 seconds
variable(global) int LOCK_TIMEOUT = 150      ; 15 seconds
variable(global) int WARP_TIMEOUT = 1200     ; 2 minutes

;==============================================================================
; GLOBAL VARIABLES
;==============================================================================
variable(global) bool ScriptRunning = TRUE
variable(global) string CurrentState = "${STATE_IDLE}"
variable(global) settingsetref Config

; Session identification
variable(global) bool IsMaster = FALSE
variable(global) string MasterSession = "is1"

;==============================================================================
; INITIALIZATION
;==============================================================================
function Initialize()
{
    echo "========================================"
    echo "${SCRIPT_NAME} v${VERSION}"
    echo "========================================"

    ; Load configuration
    call LoadConfig

    ; Determine if this is master session
    if ${Session.Equal["${MasterSession}"]}
    {
        IsMaster:Set[TRUE]
        echo "Running as MASTER session"
    }
    else
    {
        echo "Running as SLAVE session"
    }

    ; Wait for ISXEVE ready
    while !${ISXEVE.IsReady}
    {
        echo "Waiting for ISXEVE..."
        wait 20
    }

    ; Wait for character in space
    while !${Me.InSpace}
    {
        echo "Waiting to be in space..."
        wait 20
    }

    echo "Initialization complete"
}

;==============================================================================
; CONFIGURATION MANAGEMENT
;==============================================================================
function LoadConfig()
{
    Config:Set[${SCRIPT_NAME}.Config, XML]

    if !${Config:Load["Scripts/${SCRIPT_NAME}/config.xml"]}
    {
        echo "No config found, creating defaults"
        call CreateDefaultConfig
        Config:Save["Scripts/${SCRIPT_NAME}/config.xml"]
    }

    ; Load master session
    MasterSession:Set["${Config.Get[MasterSession]}"]
}

function CreateDefaultConfig()
{
    Config:Add[MasterSession, "is1"]
    Config:Add[MaxTargets, 5]
    Config:Add[OrbitDistance, 15000]
}

;==============================================================================
; MAIN LOOP
;==============================================================================
function main()
{
    call Initialize

    ; Main state machine loop
    while ${ScriptRunning}
    {
        ; Check if still in space
        if !${Me.InSpace}
        {
            echo "Not in space - waiting"
            wait 50
            continue
        }

        ; Process current state
        if ${CurrentState.Equal["${STATE_IDLE}"]}
        {
            call ProcessIdle
        }
        elseif ${CurrentState.Equal["${STATE_COMBAT}"]}
        {
            call ProcessCombat
        }
        elseif ${CurrentState.Equal["${STATE_RETURNING}"]}
        {
            call ProcessReturning
        }
        else
        {
            echo "ERROR: Unknown state ${CurrentState}"
            CurrentState:Set["${STATE_IDLE}"]
        }

        ; Pulse delay
        wait ${PULSE_DELAY}
    }

    call Cleanup
}

;==============================================================================
; STATE HANDLERS
;==============================================================================
function ProcessIdle()
{
    ; Idle state logic
}

function ProcessCombat()
{
    ; Combat state logic
}

function ProcessReturning()
{
    ; Return to station logic
}

;==============================================================================
; CLEANUP
;==============================================================================
function Cleanup()
{
    echo "Shutting down ${SCRIPT_NAME}..."

    ; Save any state if needed

    echo "Shutdown complete"
}

;==============================================================================
; RELAY HANDLERS
;==============================================================================
atom(script) EmergencyStop()
{
    echo "EMERGENCY STOP received from master!"
    ScriptRunning:Set[FALSE]
}
```

### Multi-File Organization

**Project Structure**:
```
Scripts/MyBot/
├── MyBot.iss              # Main script
├── config.xml             # Configuration
├── Libs/
│   ├── Targeting.iss      # Targeting logic
│   ├── Movement.iss       # Movement logic
│   ├── Combat.iss         # Combat logic
│   └── Utilities.iss      # Helper functions
└── States/
    ├── Idle.iss           # Idle state handler
    ├── Combat.iss         # Combat state handler
    └── Returning.iss      # Return state handler
```

**Library File Pattern** (Targeting.iss):
```lavishscript
;==============================================================================
; Library: Targeting.iss
; Purpose: Target locking and management
;==============================================================================

; Only load once
#if !${TargetingLibLoaded(exists)}
variable(global) bool TargetingLibLoaded = TRUE

function LockClosestNPC()
{
    variable index:entity NPCs
    EVE:QueryEntities[NPCs, "IsNPC && !IsLockedTarget && Distance < ${MyShip.MaxTargetRange}"]

    if ${NPCs.Used} == 0
    {
        return FALSE
    }

    ; Lock first (closest due to Distance sort in query)
    variable entity Target = ${NPCs.Get[1]}
    Target:LockTarget[]

    return TRUE
}

; More targeting functions...

#endif
```

---

## Main Loop Patterns

### Basic Pulse Loop

**Simple Main Loop**:
```lavishscript
function main()
{
    variable bool Running = TRUE

    while ${Running}
    {
        ; Check critical conditions
        if !${Me.InSpace}
        {
            wait 50
            continue
        }

        ; Do work
        call ProcessBot

        ; CRITICAL: Wait between pulses
        wait 20  ; 2 second pulse
    }
}
```

### State Machine Loop

**Production State Machine**:
```lavishscript
function main()
{
    variable string State = "Init"
    variable bool Running = TRUE
    variable int ErrorCount = 0

    while ${Running}
    {
        ; State machine with error handling
        switch ${State}
        {
            case "Init":
                if ${ProcessInit}
                {
                    State:Set["Idle"]
                }
                break

            case "Idle":
                call ProcessIdle
                break

            case "Combat":
                if !${ProcessCombat}
                {
                    ; Combat failed, increment error count
                    ErrorCount:Inc
                    if ${ErrorCount} > 3
                    {
                        echo "Too many errors, returning to safe"
                        State:Set["Error"]
                    }
                }
                else
                {
                    ErrorCount:Set[0]  ; Reset on success
                }
                break

            case "Error":
                call HandleError
                State:Set["Idle"]
                break
        }

        wait 20
    }
}
```

### Multi-Role Loop

**Master/Slave Pattern**:
```lavishscript
function main()
{
    call Initialize

    ; Different behavior based on session
    if ${IsMaster}
    {
        call MasterMainLoop
    }
    else
    {
        call SlaveMainLoop
    }

    call Cleanup
}

function MasterMainLoop()
{
    while ${ScriptRunning}
    {
        ; Master decision-making
        call MakeMasterDecisions
        call BroadcastTargets
        call ProcessMasterCombat

        wait 20
    }
}

function SlaveMainLoop()
{
    while ${ScriptRunning}
    {
        ; Slave follows master orders
        call ProcessSlaveOrders
        call ProcessSlaveCombat

        wait 20
    }
}
```

---

## Entity Processing Patterns

### Safe Entity Iteration

**Production-Safe Iteration**:
```lavishscript
function ProcessNPCs()
{
    variable index:entity NPCs
    EVE:QueryEntities[NPCs, "IsNPC && !IsMoribund && Distance < 100000"]

    variable iterator NPC
    NPCs:GetIterator[NPC]

    if ${NPC:First(exists)}
    {
        do
        {
            ; CRITICAL: Check entity still exists
            if !${NPC.Value(exists)}
            {
                echo "DEBUG: NPC despawned during iteration"
                continue
            }

            ; Store ID for safety (in case entity despawns mid-processing)
            variable int64 NPCID = ${NPC.Value.ID}

            ; Process NPC
            call ProcessSingleNPC ${NPCID}

        }
        while ${NPC:Next(exists)}
    }
}

function ProcessSingleNPC(int64 npcID)
{
    ; Double-check existence
    if !${Entity[${npcID}](exists)}
    {
        return
    }

    ; Safe to process
    echo "Processing NPC: ${Entity[${npcID}].Name}"

    ; Do work with NPC...
}
```

### Distance-Filtered Query

**Performance-Optimized Query**:
```lavishscript
function GetTargetsInRange()
{
    variable index:entity Targets

    ; Get max range from ship
    variable float MaxRange = ${MyShip.MaxTargetRange}

    ; Query with distance filter (performance optimization)
    EVE:QueryEntities[Targets, "(IsNPC || (IsPC && !IsFleetMember)) && !IsMoribund && Distance < ${MaxRange}"]

    return ${Targets.Used}
}
```

### Priority Sorting Pattern

**Sort Entities by Priority**:
```lavishscript
function GetHighestPriorityTarget()
{
    variable index:entity NPCs
    EVE:QueryEntities[NPCs, "IsNPC && !IsMoribund"]

    ; Manual sorting (LavishScript doesn't have built-in sort)
    variable int64 BestTargetID = 0
    variable float BestPriority = 999999999

    variable iterator NPC
    NPCs:GetIterator[NPC]

    if ${NPC:First(exists)}
    {
        do
        {
            if !${NPC.Value(exists)}
            {
                continue
            }

            ; Calculate priority (lower = better)
            ; Priority = Distance for closest-first
            variable float Priority = ${NPC.Value.Distance}

            ; OR: Priority based on size (frigates first)
            ; if ${NPC.Value.GroupID} == 25  ; Frigate
            ; {
            ;     Priority:Set[1]
            ; }

            if ${Priority} < ${BestPriority}
            {
                BestPriority:Set[${Priority}]
                BestTargetID:Set[${NPC.Value.ID}]
            }
        }
        while ${NPC:Next(exists)}
    }

    return ${BestTargetID}
}
```

---

## State Machine Patterns

### Enum-Style States

**State Definition Pattern**:
```lavishscript
; Define states as global constants (enum replacement)
variable(global) string STATE_INIT = "Init"
variable(global) string STATE_IDLE = "Idle"
variable(global) string STATE_TRAVELING = "Traveling"
variable(global) string STATE_COMBAT = "Combat"
variable(global) string STATE_LOOTING = "Looting"
variable(global) string STATE_RETURNING = "Returning"
variable(global) string STATE_ERROR = "Error"

variable(global) string CurrentState = "${STATE_INIT}"

; State transition helper
function ChangeState(string newState)
{
    echo "State transition: ${CurrentState} -> ${newState}"
    CurrentState:Set["${newState}"]
}
```

### State Handler Pattern

**Clean State Handlers**:
```lavishscript
function ProcessCurrentState()
{
    if ${CurrentState.Equal["${STATE_IDLE}"]}
    {
        call StateIdle
    }
    elseif ${CurrentState.Equal["${STATE_COMBAT}"]}
    {
        call StateCombat
    }
    elseif ${CurrentState.Equal["${STATE_LOOTING}"]}
    {
        call StateLooting
    }
    ; ... etc
}

function StateIdle()
{
    ; Look for work
    variable index:entity NPCs
    EVE:QueryEntities[NPCs, "IsNPC && Distance < 100000"]

    if ${NPCs.Used} > 0
    {
        echo "NPCs detected - entering combat"
        call ChangeState "${STATE_COMBAT}"
        return
    }

    ; No work, stay idle
}

function StateCombat()
{
    ; Check if combat done
    variable index:entity NPCs
    EVE:QueryEntities[NPCs, "IsNPC && !IsMoribund"]

    if ${NPCs.Used} == 0
    {
        echo "Combat complete - entering looting"
        call ChangeState "${STATE_LOOTING}"
        return
    }

    ; Continue combat
    call ProcessCombatLogic
}
```

### Sub-State Pattern

**Nested State Machines**:
```lavishscript
; Main state
variable string MainState = "Combat"

; Sub-state for combat
variable string CombatSubState = "Targeting"

function StateCombat()
{
    ; Combat sub-state machine
    if ${CombatSubState.Equal["Targeting"]}
    {
        if ${ProcessTargeting}
        {
            CombatSubState:Set["Engaging"]
        }
    }
    elseif ${CombatSubState.Equal["Engaging"]}
    {
        if ${ProcessEngagement}
        {
            CombatSubState:Set["Finishing"]
        }
    }
    elseif ${CombatSubState.Equal["Finishing"]}
    {
        if ${ProcessFinishing}
        {
            ; Combat complete
            call ChangeState "Looting"
            CombatSubState:Set["Targeting"]  ; Reset for next time
        }
    }
}
```

---

## Error Handling Patterns

### Existence Checking

**Always Check Existence**:
```lavishscript
function LockTarget(int64 targetID)
{
    ; CRITICAL: Check existence before accessing
    if !${Entity[${targetID}](exists)}
    {
        echo "ERROR: Target ${targetID} doesn't exist"
        return FALSE
    }

    ; Safe to use
    Entity[${targetID}]:LockTarget[]
    return TRUE
}
```

### Timeout Pattern

**Action with Timeout**:
```lavishscript
function LockTargetWithTimeout(int64 targetID, int timeoutSeconds)
{
    if !${Entity[${targetID}](exists)}
    {
        return FALSE
    }

    ; Issue lock command
    Entity[${targetID}]:LockTarget[]

    ; Wait for lock with timeout
    variable int timeout = ${Math.Calc[${timeoutSeconds} * 10]}  ; Convert to deciseconds
    variable int counter = 0

    while ${counter} < ${timeout}
    {
        ; Check if locked
        if ${Entity[${targetID}](exists)} && ${Entity[${targetID}].IsLockedTarget}
        {
            echo "Target locked in ${Math.Calc[${counter} / 10]} seconds"
            return TRUE
        }

        ; Check if target still exists
        if !${Entity[${targetID}](exists)}
        {
            echo "Target despawned while locking"
            return FALSE
        }

        wait 10
        counter:Inc
    }

    echo "Lock timed out after ${timeoutSeconds} seconds"
    return FALSE
}
```

### Retry Pattern

**Retry with Backoff**:
```lavishscript
function DockWithRetry(int64 stationID, int maxRetries)
{
    variable int attempt = 0

    while ${attempt} < ${maxRetries}
    {
        echo "Dock attempt ${Math.Calc[${attempt} + 1]} of ${maxRetries}"

        if !${Entity[${stationID}](exists)}
        {
            echo "ERROR: Station doesn't exist"
            return FALSE
        }

        ; Try to dock
        Entity[${stationID}]:Dock[]

        ; Wait for docking
        variable int wait = 0
        while ${wait} < 100 && ${Me.InSpace}
        {
            wait 10
            wait:Inc
        }

        ; Check if docked
        if ${Me.InStation}
        {
            echo "Docked successfully"
            return TRUE
        }

        ; Failed, increment and backoff
        attempt:Inc
        variable int backoff = ${Math.Calc[${attempt} * 20]}  ; Increasing delay
        echo "Dock failed, waiting ${Math.Calc[${backoff} / 10]} seconds before retry"
        wait ${backoff}
    }

    echo "Failed to dock after ${maxRetries} attempts"
    return FALSE
}
```

### Graceful Degradation

**Fallback Behavior**:
```lavishscript
function FindSafestBookmark()
{
    ; Try to find safe bookmark
    variable index:bookmark SafeBookmarks
    EVE:GetBookmarks[SafeBookmarks]

    variable iterator BM
    SafeBookmarks:GetIterator[BM]

    if ${BM:First(exists)}
    {
        do
        {
            if ${BM.Value.Title.Find["Safe"]}
            {
                echo "Found safe bookmark: ${BM.Value.Title}"
                return ${BM.Value.ID}
            }
        }
        while ${BM:Next(exists)}
    }

    ; Fallback: No safe bookmark found
    echo "WARNING: No safe bookmark found, using station"

    ; Try to find station
    variable index:entity Stations
    EVE:QueryEntities[Stations, "GroupID = 15"]  ; Station group

    if ${Stations.Used} > 0
    {
        return ${Stations.Get[1].ID}
    }

    ; Final fallback: Just stay in space
    echo "ERROR: No safe location found, staying in current location"
    return 0
}
```

---

## Timing and Wait Patterns

### Adaptive Waiting

**Wait Until Condition Met**:
```lavishscript
function WaitUntilInRange(int64 entityID, float range, int timeoutSeconds)
{
    variable int timeout = ${Math.Calc[${timeoutSeconds} * 10]}
    variable int counter = 0

    while ${counter} < ${timeout}
    {
        if !${Entity[${entityID}](exists)}
        {
            echo "Entity despawned"
            return FALSE
        }

        if ${Entity[${entityID}].Distance} <= ${range}
        {
            echo "In range (${Entity[${entityID}].Distance}m)"
            return TRUE
        }

        wait 10
        counter:Inc
    }

    echo "Timeout waiting to get in range"
    return FALSE
}
```

### Cooldown Management

**Timestamp-Based Cooldowns**:
```lavishscript
; Global cooldown trackers
variable(global) int64 LastTargetSwitch = 0
variable(global) int64 LastOrbitalAdjust = 0

function CanSwitchTarget()
{
    variable int64 Now = ${Time.Timestamp}
    variable int64 TimeSinceSwitch = ${Math.Calc[${Now} - ${LastTargetSwitch}]}

    ; 5 second cooldown
    if ${TimeSinceSwitch} >= 5
    {
        return TRUE
    }

    return FALSE
}

function SwitchToNewTarget(int64 newTargetID)
{
    if !${CanSwitchTarget}
    {
        echo "Target switch on cooldown"
        return FALSE
    }

    ; Do switch
    MyShip:SetActiveTarget[${newTargetID}]

    ; Update cooldown
    LastTargetSwitch:Set[${Time.Timestamp}]

    return TRUE
}
```

### Pulse Counting

**Frame-Based Timers**:
```lavishscript
variable int PulseCounter = 0
variable int ActionEvery = 10  ; Every 10 pulses = 20 seconds at wait 20

function main()
{
    while ${ScriptRunning}
    {
        PulseCounter:Inc

        ; Check every 10 pulses
        if ${Math.Calc[${PulseCounter} % ${ActionEvery}]} == 0
        {
            call PeriodicAction
        }

        wait 20
    }
}
```

---

## Relay Communication Patterns

### Master Broadcast Pattern

**Master Sends Data**:
```lavishscript
; Master script
function BroadcastTargets()
{
    ; Get my locked targets
    variable index:int64 TargetIDs
    variable index:activetarget Targets
    Me:GetTargets[Targets]

    ; Build comma-separated ID list
    variable string IDList = ""
    variable iterator Target
    Targets:GetIterator[Target]

    if ${Target:First(exists)}
    {
        do
        {
            if ${IDList.Length} > 0
            {
                IDList:Concat[","]
            }
            IDList:Concat["${Target.Value.ID}"]
        }
        while ${Target:Next(exists)}
    }

    ; Get active target
    variable int64 ActiveID = 0
    if ${MyShip.ActiveTarget(exists)}
    {
        ActiveID:Set[${MyShip.ActiveTarget.ID}]
    }

    ; Relay to all slaves
    relay "all other" ReceiveTargets "${IDList}" ${ActiveID}
}

; Run every pulse
function main()
{
    while ${ScriptRunning}
    {
        if ${IsMaster}
        {
            call BroadcastTargets
        }

        wait 20
    }
}
```

### Slave Receive Pattern

**Slave Receives Data**:
```lavishscript
; Slave script
atom(script) ReceiveTargets(string targetIDList, int64 activeTargetID)
{
    echo "Received targets: ${targetIDList}, active: ${activeTargetID}"

    ; Parse comma-separated list
    variable string Remaining = "${targetIDList}"

    while ${Remaining.Length} > 0
    {
        variable int CommaPos = ${Remaining.Find[","]}
        variable string IDStr

        if ${CommaPos} > 0
        {
            ; Extract ID before comma
            IDStr:Set["${Remaining.Left[${Math.Calc[${CommaPos} - 1]}]}"]
            ; Remove processed part
            Remaining:Set["${Remaining.Right[${Math.Calc[${Remaining.Length} - ${CommaPos}]}]}"]
        }
        else
        {
            ; Last ID
            IDStr:Set["${Remaining}"]
            Remaining:Set[""]
        }

        ; Lock this target
        variable int64 TargetID = ${IDStr}
        if ${Entity[${TargetID}](exists)} && !${Entity[${TargetID}].IsLockedTarget}
        {
            echo "Locking target: ${TargetID}"
            Entity[${TargetID}]:LockTarget[]
        }
    }

    ; Set active target
    if ${activeTargetID} > 0 && ${Entity[${activeTargetID}](exists)}
    {
        MyShip:SetActiveTarget[${activeTargetID}]
    }
}
```

### Emergency Stop Pattern

**Global Emergency Stop**:
```lavishscript
; Master sends emergency stop
function EmergencyStopAll()
{
    echo "EMERGENCY STOP - broadcasting to all sessions"
    relay "all other" EmergencyStop

    ; Stop self
    ScriptRunning:Set[FALSE]
}

; All scripts receive
atom(script) EmergencyStop()
{
    echo "EMERGENCY STOP received!"
    ScriptRunning:Set[FALSE]
}
```

---

## Configuration Patterns

### XML Settings Pattern

**Complete Config System**:
```lavishscript
variable(globalkeep) settingsetref BotConfig

function LoadConfiguration()
{
    BotConfig:Set[MyBot.Config, XML]

    if !${BotConfig:Load["Scripts/MyBot/config.xml"]}
    {
        echo "Creating default configuration"
        call CreateDefaultConfig
        BotConfig:Save["Scripts/MyBot/config.xml"]
    }

    ; Load settings into variables
    call LoadSettingsIntoVariables
}

function CreateDefaultConfig()
{
    ; Basic settings
    BotConfig:Add[Version, "1.0"]
    BotConfig:Add[MasterSession, "is1"]

    ; Combat settings
    BotConfig:AddSet[Combat]
    BotConfig.FindSet[Combat]:Add[MaxTargets, 5]
    BotConfig.FindSet[Combat]:Add[OrbitDistance, 15000]
    BotConfig.FindSet[Combat]:Add[PreferredDamageType, "Thermal"]

    ; Movement settings
    BotConfig:AddSet[Movement]
    BotConfig.FindSet[Movement]:Add[MaxWarpDistance, 1000000000]
    BotConfig.FindSet[Movement]:Add[DockWhenDamaged, TRUE]

    ; Targeting settings
    BotConfig:AddSet[Targeting]
    BotConfig.FindSet[Targeting]:Add[PriorityMode, "Closest"]
    BotConfig.FindSet[Targeting]:Add[TargetFrigatesFirst, TRUE]
}

function LoadSettingsIntoVariables()
{
    ; Load into global variables for easy access
    MasterSession:Set["${BotConfig.Get[MasterSession]}"]
    MaxTargets:Set[${BotConfig.FindSet[Combat].Get[MaxTargets]}]
    OrbitDistance:Set[${BotConfig.FindSet[Combat].Get[OrbitDistance]}]
    ; ... etc
}
```

---

## Logging and Debug Patterns

### Debug Levels

**Configurable Logging**:
```lavishscript
variable(global) int LOG_LEVEL = 2  ; 0=None, 1=Error, 2=Info, 3=Debug

function LogError(string message)
{
    if ${LOG_LEVEL} >= 1
    {
        echo "[ERROR] ${message}"
    }
}

function LogInfo(string message)
{
    if ${LOG_LEVEL} >= 2
    {
        echo "[INFO] ${message}"
    }
}

function LogDebug(string message)
{
    if ${LOG_LEVEL} >= 3
    {
        echo "[DEBUG] ${message}"
    }
}
```

### Performance Profiling

**Measure Execution Time**:
```lavishscript
function ProfileFunction()
{
    variable int64 StartTime = ${Time.Timestamp}

    ; Do expensive work
    call ExpensiveOperation

    variable int64 EndTime = ${Time.Timestamp}
    variable int64 Duration = ${Math.Calc[${EndTime} - ${StartTime}]}

    echo "ExpensiveOperation took ${Duration} seconds"
}
```

---

## Common Anti-Patterns

### ❌ Tight Loop Without Wait

**WRONG**:
```lavishscript
; DON'T DO THIS
while ${SomeCondition}
{
    call DoWork
    ; NO WAIT - maxes CPU, crashes InnerSpace
}
```

**CORRECT**:
```lavishscript
while ${SomeCondition}
{
    call DoWork
    wait 10  ; ALWAYS wait in loops
}
```

### ❌ No Existence Checking

**WRONG**:
```lavishscript
; DON'T DO THIS
echo "Target: ${Entity[${TargetID}].Name}"
; Crashes if entity doesn't exist
```

**CORRECT**:
```lavishscript
if ${Entity[${TargetID}](exists)}
{
    echo "Target: ${Entity[${TargetID}].Name}"
}
else
{
    echo "Target doesn't exist"
}
```

### ❌ Assuming Action Success

**WRONG**:
```lavishscript
; DON'T DO THIS
Entity:LockTarget[]
; Immediately try to shoot (lock not finished)
Module:Activate[]
```

**CORRECT**:
```lavishscript
Entity:LockTarget[]
wait 10

; Wait for lock
while !${Entity.IsLockedTarget} && ${counter} < 150
{
    wait 10
    counter:Inc
}

if ${Entity.IsLockedTarget}
{
    Module:Activate[]
}
```

---

## Summary

**Key Patterns to Remember**:

1. **Script Organization** - Clean structure, includes, constants
2. **Main Loop** - Always with wait, state machine
3. **Entity Processing** - Safe iteration with existence checks
4. **State Machines** - Clear states, clean transitions
5. **Error Handling** - Timeouts, retries, graceful degradation
6. **Timing** - Adaptive waits, cooldowns, pulse counting
7. **Relay** - Master/slave communication patterns
8. **Configuration** - XML settings, defaults, loading
9. **Logging** - Debug levels, profiling

**Anti-Patterns to Avoid**:
- No wait in loops
- No existence checks
- Assuming instant actions
- No error handling
- Magic numbers (use constants)

---

*Last Updated: 2025-10-26*
*Part of ISXEVE Scripting Guide*
