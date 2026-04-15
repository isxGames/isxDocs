# Advanced Scripting Patterns

**Purpose:** Production-grade patterns for script organization, main loops, state machines, error handling, relay communication, and configuration management
**Audience:** Intermediate to advanced scripters building complex automation

---

## Table of Contents

1. [Script Organization Patterns](#script-organization-patterns)
2. [Main Loop Patterns](#main-loop-patterns)
3. [Entity Processing Patterns](#entity-processing-patterns)
4. [State Machine Patterns](#state-machine-patterns)
5. [Unified Movement Facade](#unified-movement-facade)
6. [Error Handling Patterns](#error-handling-patterns)
7. [Timing and Wait Patterns](#timing-and-wait-patterns)
8. [Relay Communication Patterns](#relay-communication-patterns)
9. [Configuration Patterns](#configuration-patterns)
10. [Logging and Debug Patterns](#logging-and-debug-patterns)
11. [Common Anti-Patterns](#common-anti-patterns)

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
variable(global) settingsetref ConfigRef

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
    ; Create (or attach to) the root settings set by name
    LavishSettings:AddSet[${SCRIPT_NAME}.Config]
    ConfigRef:Set[${LavishSettings[${SCRIPT_NAME}.Config]}]

    ; Import existing XML if present; seed defaults otherwise and write it out
    variable string ConfigFile = "Scripts/${SCRIPT_NAME}/config.xml"
    if ${ConfigFile.FileExists}
    {
        LavishSettings[${SCRIPT_NAME}.Config]:Import["${ConfigFile}"]
    }
    else
    {
        echo "No config found, creating defaults"
        call CreateDefaultConfig
        LavishSettings[${SCRIPT_NAME}.Config]:Export["${ConfigFile}"]
    }

    ; Read a value via FindSetting (the second arg is the default if the key is absent)
    MasterSession:Set["${LavishSettings[${SCRIPT_NAME}.Config].FindSetting[MasterSession, "is1"]}"]
}

function CreateDefaultConfig()
{
    LavishSettings[${SCRIPT_NAME}.Config]:AddSetting[MasterSession, "is1"]
    LavishSettings[${SCRIPT_NAME}.Config]:AddSetting[MaxTargets, 5]
    LavishSettings[${SCRIPT_NAME}.Config]:AddSetting[OrbitDistance, 15000]
}

;==============================================================================
; MAIN LOOP
;==============================================================================
function main()
{
    ; Dependency check — runtime only; the preprocessor cannot see runtime objects
    if !${ISXEVE(exists)}
    {
        echo "ERROR: ISXEVE extension not loaded. Run 'ext isxeve' first."
        Script:End
        return
    }

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
; atom(script) EmergencyStop() — see "Emergency Stop Pattern" section below for full implementation
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

; Include-once guard — preprocessor symbols, evaluated at script-load time
#ifndef TARGETINGLIB_LOADED
#define TARGETINGLIB_LOADED 1

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

## Unified Movement Facade

**Reference implementation:** see Tehbot's `core/obj_Move.iss`.

**Motivation.** A non-trivial bot needs to move to *many* kinds of destinations: a named bookmark, an in-space entity, a different solar system, a fleet member who may be several jumps away, an accel gate, an agent's station, a mission bookmark. Writing a separate state machine for each destination type produces scattered, inconsistent logic — each call site duplicates "am I docked?", "am I already warping?", "is there a gate in the way?", "did I arrive yet?". The unified movement facade collapses all of that behind one object so callers issue *one* command and poll *one* flag.

**Public surface.** `obj_Move` (which itself inherits `obj_StateQueue`) exposes a small, uniform set of dispatch methods — one per destination type:

| Method | Destination |
|---|---|
| `Bookmark[label, IgnoreGate, Distance, FleetWarp]` | Named bookmark (or station by name) |
| `Entity[ID, Distance, FleetWarp]` | Any entity in space |
| `System[SystemID]` | Another solar system (uses autopilot + waypoint) |
| `Fleetmember[charID, IgnoreGate, Distance]` | Follow a fleet member, bounce-warping as needed |
| `Gate[entityID]` | Approach and activate a stargate / accel gate |
| `Agent[name]` / `Agent[index]` | Travel to an agent's station |
| `AgentBookmark[id]` | Travel to a mission bookmark |
| `Approach[ID, distance]` | Sub-warp approach using afterburner/MWD |
| `Orbit[ID, distance]` | Enter a stable orbit |
| `SaveSpot` / `GotoSavedSpot` / `RemoveSavedSpot` | Ad-hoc safespot bookmarking |
| `Stop` | Cancel the current movement and clear the queue |

Every dispatch method does the same three things: validate that the destination exists, set `Traveling` to `TRUE`, and `QueueState` a private per-destination-type `*Move` member (e.g. `BookmarkMove`, `EntityMove`, `SystemMove`, `FleetmemberMove`) that carries out the actual multi-step work.

**The `Traveling` flag.** A single `bool Traveling` on `obj_Move` doubles as a mutex and a completion signal:

- Every public dispatch method early-returns if `${Move.Traveling}` is already `TRUE`. This prevents a caller from accidentally stacking conflicting movement commands.
- The private `*Move` state members set `Traveling:Set[FALSE]` only when they detect arrival (docked at the right station, within range of the fleet member, in the right system, within 150 km of the bookmark/entity, etc.).
- Callers therefore don't need to know anything about warp phases, gate jumps, undocks, or docking animations — they just poll one boolean.

**Internally-handled transitions.** Each `*Move` member knows how to walk through the full journey:

- **Docked but need to move?** Issue `Undock`, return `FALSE` (stay queued).
- **Already warping?** Return `FALSE` — wait it out.
- **Wrong solar system?** Delegate to `TravelToSystem`, which sets a waypoint and toggles autopilot.
- **Gate in the way (and `IgnoreGate` is false)?** Recursively call `This:Gate[...]` with `CalledFromMove=TRUE` so the gate sub-state doesn't clear `Traveling` on its own completion.
- **In warp range of the destination?** Warp (honoring `FleetWarp` if the caller has fleet/wing/squad command).
- **Final destination is a station/structure?** Hand off to `DockAtStation`.
- **Arrived?** Log "Reached", clear `Traveling`, return `TRUE`.

**Caller pattern.** Because the facade absorbs every intermediate state, call sites become uniformly trivial — fire-and-poll:

```lavishscript
; Issue one command — could be any of bookmark / entity / system / fleetmember
Move:Bookmark["MyDestination"]

; Poll until the facade reports completion
while ${Move.Traveling}
{
    wait 20
}

; At this point we are docked / at the bookmark / in the target system / next to the fleetmate.
; The caller never had to know whether the journey involved undocking, a gate jump, or a fleet warp.
```

The same three-line shape works for `Move:Entity[${targetID}]`, `Move:System[${destID}]`, or `Move:Fleetmember[${charID}]` — that uniformity is the whole point of the facade.

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
    Entity[${newTargetID}]:MakeActiveTarget

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
    variable index:entity Targets
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
        Entity[${activeTargetID}]:MakeActiveTarget
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

**Core LavishSettings pattern** (load or create):
```lavishscript
variable(globalkeep) settingsetref BotConfig

; Create (or re-attach to) the named root set; LavishSettings persists across scripts
LavishSettings:AddSet[MyBot.Config]
BotConfig:Set[${LavishSettings[MyBot.Config]}]

variable string ConfigFile = "Scripts/MyBot/config.xml"
if ${ConfigFile.FileExists}
{
    ; Load existing config
    LavishSettings[MyBot.Config]:Import["${ConfigFile}"]
}
else
{
    ; Seed defaults with AddSetting[Key, Value]; nested sections use AddSet[Name]
    LavishSettings[MyBot.Config]:AddSetting[MasterSession, "is1"]
    LavishSettings[MyBot.Config]:AddSetting[MaxTargets, 5]
    LavishSettings[MyBot.Config]:AddSet[Combat]
    LavishSettings[MyBot.Config].FindSet[Combat]:AddSetting[OrbitDistance, 15000]
    LavishSettings[MyBot.Config]:Export["${ConfigFile}"]
}

; Read values via FindSetting (second arg = default when the key is absent)
variable string Master = "${LavishSettings[MyBot.Config].FindSetting[MasterSession, "is1"]}"
variable int MaxTgt   = ${LavishSettings[MyBot.Config].FindSetting[MaxTargets, 5]}
```

For the full config architecture -- sub-configuration objects, derived classes, the `Setting()` / `Define_ConfigItem()` macros, config migration, and validation best practices -- see the [EVEBot Configuration Architecture](07_Advanced_Patterns_And_Examples.md#evebot-configuration-architecture) and [Tehbot Configuration Architecture](07_Advanced_Patterns_And_Examples.md#tehbot-configuration-architecture) chapters in 07_Advanced_Patterns_And_Examples.md.

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
