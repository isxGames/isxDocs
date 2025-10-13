# File 26: Yamfa Fleet Assist Analysis

**Layer 6: Script Analysis (Learning from Examples)**

---

## Table of Contents
1. [Yamfa Overview](#yamfa-overview)
2. [Architecture Comparison](#architecture-comparison)
3. [Master-Slave Pattern](#master-slave-pattern)
4. [Hysteresis and Timing](#hysteresis-and-timing)
5. [Relay Communication](#relay-communication)
6. [Target Management](#target-management)
7. [Movement and Following](#movement-and-following)
8. [Code Strengths](#code-strengths)
9. [Code Weaknesses and Fixes](#code-weaknesses-and-fixes)
10. [Improvement Roadmap](#improvement-roadmap)
11. [Lessons for the Community](#lessons-for-the-community)

---

## Yamfa Overview

### What is Yamfa?

**Y**et **A**nother **M**ulti **F**leet **A**ssist - A lightweight fleet assist bot for target synchronization and fleet following.

**Primary Purpose**:
1. **Master** locks targets → broadcasts to slaves
2. **Slaves** lock same targets → follow master's ship
3. **Result**: Coordinated DPS on same targets

**Use Cases**:
- Multi-boxing missions (all ships shoot same target)
- Incursion fleets (coordinated targeting)
- PvP support (follow FC, shoot primary)
- Mining defense (all shoot rats together)

### Single-File Architecture

Unlike EVEBot's 30+ files, Yamfa is **845 lines in one file**:

```
Yamfa.iss (845 lines total)
├── Constants & Variables (lines 1-61)
├── Relay Event Handler (lines 62-71)
├── Main Entry Point (lines 72-120)
├── Initialization (lines 121-179)
├── Main Pulse (lines 180-205)
├── Master Pulse (lines 206-310)
├── Slave Pulse (lines 311-439)
├── Movement Functions (lines 440-495)
├── Config & UI (lines 496-589)
├── Console Commands (lines 590-657)
├── Retreat Functions (lines 658-737)
├── Shutdown (lines 738-761)
├── UI Event Handlers (lines 762-799)
└── Hotkey Relay (lines 800-845)
```

**Philosophy**: Simplicity over modularity. Entire bot fits in one file for easy understanding.

---

## Architecture Comparison

### Yamfa vs. EVEBot

| Aspect | Yamfa | EVEBot |
|--------|-------|--------|
| **Files** | 1 file (845 lines) | 30+ files (~15,000+ lines) |
| **Purpose** | Fleet assist only | Full automation (mining, combat, hauling) |
| **Complexity** | Simple | Complex |
| **State Machine** | No states | Comprehensive states |
| **Error Handling** | Minimal | Extensive |
| **Modularity** | None (monolithic) | Highly modular |
| **Learning Curve** | Easy | Steep |
| **Maintenance** | Easy (one file) | Complex (many files) |
| **Extensibility** | Limited | Highly extensible |

### When to Use Each Pattern

**Use Yamfa-style (single file) when**:
- Bot has ONE specific purpose
- Code is < 1000 lines
- You want simplicity over features
- Learning/prototyping

**Use EVEBot-style (modular) when**:
- Multi-purpose bot
- Code > 2000 lines
- Team development
- Long-term maintenance needed

---

## Master-Slave Pattern

### Role Determination

File: `Yamfa.iss` (lines 152-160)

```lavish
; Determine role based on character name
if ${Me.Name.Equal["${MASTER_CHARACTER_NAME}"]}
{
    echo "${Me.Name} is MASTER"
}
else
{
    echo "${Me.Name} is SLAVE"
    MasterName:Set[${MASTER_CHARACTER_NAME}]
}
```

**Critical Issue**: Hardcoded master name!

```lavish
variable string MASTER_CHARACTER_NAME = "YourCharacterName"
```

**Problem**: Can't change master without editing code.

**Better Approach**:
```lavish
; In config:
variable bool IsMaster = FALSE    // Set via config

; Or auto-detect:
if ${Me.Fleet.IsMember[${Me.CharID}]} && ${Me.ToFleetMember.IsFleetCommander}
{
    echo "${Me.Name} is MASTER (Fleet Commander)"
    IsMaster:Set[TRUE]
}
```

### Master-Slave Data Flow

```
MASTER                                      SLAVE
  ↓                                          ↓
Lock targets                           Wait for relay
  ↓                                          ↓
Query locked/locking                   Receive target IDs
  ↓                                          ↓
Build target ID list                   Parse target list
  ↓                                          ↓
Get active target                      Lock each target
  ↓                                          ↓
Relay to all slaves  ────────────>    Set active target
  ↓                                          ↓
Wait for next pulse                    Follow master ship
```

---

## Hysteresis and Timing

### What is Hysteresis?

**Hysteresis** = Delay before state changes to prevent rapid flickering.

**Yamfa's Hysteresis**:
```lavish
; Master holds targets for 0.7 seconds after they disappear
variable int MASTER_HOLD_TIME = 7  ; 7 deciseconds = 0.7 seconds

; Slave unlocks targets 0.8 seconds after master stops broadcasting
variable int SLAVE_UNLOCK_TIME = 8  ; 8 deciseconds = 0.8 seconds
```

### Why This Matters

**Without Hysteresis**:
```
Master: Lock → Unlock (momentary) → Lock
         ↓
Slave:  Lock → Unlock → Lock → Unlock → Lock
        (Spam! Wastes CPU, looks suspicious)
```

**With Hysteresis**:
```
Master: Lock → Unlock (momentary) → Lock
         ↓
Slave:  Lock → [HOLD 0.7s] → Still locked (target came back)
        (Smooth, stable targeting)
```

### Master Hysteresis Implementation

File: `Yamfa.iss` (lines 233-266)

```lavish
; Apply hysteresis - keep targets for MASTER_HOLD_TIME after they disappear
variable iterator ExistingTarget
MasterTargetSet:GetIterator[ExistingTarget]

if ${ExistingTarget:First(exists)}
    do
    {
        variable int64 TargetID = ${MasterTargetSet.Get[${ExistingTarget.Key}]}
        variable int LastSeen = ${MasterTargetTimers.Get[${TargetID}]}

        ; Check if target still in current lock list
        variable bool StillExists = FALSE
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
            if ${Math.Calc[${CurrentTime} - ${LastSeen}]} < ${MASTER_HOLD_TIME} &&
               ${Entity[${TargetID}](exists)} &&
               (${Entity[${TargetID}].IsLockedTarget} || ${Entity[${TargetID}].BeingTargeted})
            {
                CurrentTargets:Insert[${TargetID}]
            }
        }
    }
    while ${ExistingTarget:Next(exists)}
```

**Logic**:
1. Check each previously broadcast target
2. If target not in current lock list:
   - Check if less than 0.7 seconds since last seen
   - Check if entity still exists and is locked/locking
   - If yes to both → Keep in broadcast set
3. Prevents flickering when targets momentarily drop

### Timing Constants (Deciseconds)

```lavish
; All timing in deciseconds (1 decisecond = 100ms = 0.1 second)

variable int MASTER_HOLD_TIME = 7      ; 0.7s - hold targets after disappear
variable int SLAVE_UNLOCK_TIME = 8     ; 0.8s - unlock if absent from master
variable int SLAVE_LOCK_COOLDOWN = 2   ; 0.2s - cooldown between lock attempts
variable int RELAY_MIN_INTERVAL = 10   ; 1.0s - minimum time between relays
```

**Why Deciseconds?**
- More precise than seconds (100ms resolution)
- Less overhead than milliseconds
- Perfect for bot timing (human reaction ~200-300ms)

**Conversion**:
```lavish
; Current time in deciseconds
variable int CurrentTime = ${Math.Calc[${LavishScript.RunningTime} / 100]}

; Check if 0.7 seconds passed
if ${Math.Calc[${CurrentTime} - ${LastTime}]} > 7
{
    ; 0.7 seconds elapsed
}
```

---

## Relay Communication

### Relay Event Pattern

File: `Yamfa.iss` (lines 174-176, 65-71, 298)

**Registration** (Both master and slave):
```lavish
; Register custom event
LavishScript:RegisterEvent[YamfaTargets]

; Attach handler
Event[YamfaTargets]:AttachAtom[YamfaTargetsRelay]
```

**Event Handler**:
```lavish
atom(script) YamfaTargetsRelay(string targetIDs, int64 relayactivetarget)
{
    echo "Relay received: ${targetIDs} with active: ${relayactivetarget}"

    ; Store in global variables (atom can't modify local vars directly)
    YamfaRelayedTargets:Set[${targetIDs}]
    YamfaRelayedActive:Set[${relayactivetarget}]

    ; Flag for processing in main loop
    YamfaNewRelayData:Set[TRUE]
}
```

**Master Broadcast**:
```lavish
; Build target list: "ID1|ID2|ID3"
variable string TargetIDList = ""
for (j:Set[1] ; ${j} <= ${MasterTargetSet.Used} ; j:Inc)
{
    if ${j} > 1
        TargetIDList:Concat["|"]    ; Pipe delimiter
    TargetIDList:Concat[${MasterTargetSet.Get[${j}]}]
}

; Relay to all other sessions
relay "all other" -noredirect "Event[YamfaTargets]:Execute[\"${TargetIDList}\",${MasterActiveTarget}]"
```

**Key Points**:
- `-noredirect` prevents feedback loop (master receiving own relay)
- `"all other"` targets all sessions except sender
- Pipe `|` delimiter separates target IDs
- Active target sent separately (int64)

### Relay Data Processing

**Master Side** (lines 282-309):
```lavish
; Build target list string
variable string TargetIDList = ""
for (j:Set[1] ; ${j} <= ${MasterTargetSet.Used} ; j:Inc)
{
    if ${j} > 1
        TargetIDList:Concat["|"]
    TargetIDList:Concat[${MasterTargetSet.Get[${j}]}]
}

; Create hash of current state for change detection
variable string CurrentHash = "${TargetIDList}:${MasterActiveTarget}"

; Relay if changed OR minimum interval passed
if !${CurrentHash.Equal[${LastRelayedTargetHash}]} ||
   ${Math.Calc[${CurrentTime} - ${LastRelayTime}]} >= ${RELAY_MIN_INTERVAL}
{
    if ${MasterTargetSet.Used} > 0
    {
        relay "all other" -noredirect "Event[YamfaTargets]:Execute[\"${TargetIDList}\",${MasterActiveTarget}]"
        echo "Relaying ${MasterTargetSet.Used} targets"
    }
    else
    {
        ; Empty target set = clear all targets
        relay "all other" -noredirect "Event[YamfaTargets]:Execute[\"\",0]"
        echo "Relaying empty target set"
    }

    LastRelayedTargetHash:Set[${CurrentHash}]
    LastRelayTime:Set[${CurrentTime}]
}
```

**Change Detection Pattern**:
- Hash = "${TargetIDList}:${ActiveTarget}"
- Only relay if hash changed OR 1 second passed
- Reduces spam, improves performance

**Slave Side** (lines 341-390):
```lavish
function ProcessRelayedTargets(string targetIDs, int64 activeTarget, int currentTime)
{
    ; Handle empty relay (master cleared targets)
    if ${targetIDs.Length} == 0
    {
        SlaveTargetSet:Clear
        EVE:Execute[CmdClearTargets]    ; Clear all locks
        SlaveActiveTarget:Set[0]
        return
    }

    ; Clear and rebuild
    SlaveTargetSet:Clear

    ; Check for multiple targets (pipe-separated)
    if ${targetIDs.Find["|"]} > 0
    {
        ; Parse pipe-separated list
        variable int parseIdx = 1
        variable string currentID

        while ${parseIdx} <= 20    ; Safety limit
        {
            currentID:Set[${targetIDs.Token[${parseIdx},"|"]}]

            if ${currentID.Length} > 0
            {
                SlaveTargetSet:Insert[${currentID}]
                echo "Added target ${parseIdx}: ${currentID}"
            }
            else
            {
                break
            }

            parseIdx:Inc
        }
    }
    else
    {
        ; Single target
        SlaveTargetSet:Insert[${targetIDs}]
        echo "Added single target: ${targetIDs}"
    }

    SlaveActiveTarget:Set[${activeTarget}]
    echo "SlaveTargetSet now has ${SlaveTargetSet.Used} targets"
}
```

**Parsing Pattern**:
- Check if `|` exists in string
- Use `.Token[index, "|"]` to split by pipe
- Safety limit (20 targets max) prevents infinite loop
- Handles both single and multiple targets

---

## Target Management

### Master Target Management

File: `Yamfa.iss` (lines 210-310)

```lavish
function MasterPulse()
{
    variable int CurrentTime = ${Math.Calc[${LavishScript.RunningTime} / 100]}
    variable index:entity Targets
    variable iterator Target
    variable index:int64 CurrentTargets

    ; 1. GET ALL CURRENTLY LOCKED/LOCKING TARGETS
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

    ; 2. APPLY HYSTERESIS (shown earlier)
    ; Keep targets for MASTER_HOLD_TIME after they disappear

    ; 3. UPDATE MASTER TARGET SET
    MasterTargetSet:Clear
    for (j:Set[1] ; ${j} <= ${CurrentTargets.Used} ; j:Inc)
    {
        MasterTargetSet:Insert[${CurrentTargets.Get[${j}]}]
    }

    ; 4. GET ACTIVE TARGET
    MasterActiveTarget:Set[0]
    if ${Me.ActiveTarget(exists)}
        MasterActiveTarget:Set[${Me.ActiveTarget.ID}]

    ; 5. RELAY TO SLAVES (shown earlier)
}
```

**Flow**:
1. Query all entities
2. Filter to locked/locking targets
3. Apply hysteresis (keep disappearing targets)
4. Update master set
5. Get active target
6. Relay to slaves

### Slave Target Management

File: `Yamfa.iss` (lines 393-439)

```lavish
function SlaveTargetManagement(int currentTime)
{
    echo "=== SlaveTargetManagement ==="
    echo "Targets in set: ${SlaveTargetSet.Used}"

    ; TRY TO LOCK EVERYTHING IN SLAVETARGETSET
    variable int lockIdx
    for (lockIdx:Set[1] ; ${lockIdx} <= ${SlaveTargetSet.Used} ; lockIdx:Inc)
    {
        variable int64 TargetID
        TargetID:Set[${SlaveTargetSet.Get[${lockIdx}]}]
        echo "Checking target ${lockIdx}: ID=${TargetID}"

        if ${Entity[${TargetID}](exists)}
        {
            if !${Entity[${TargetID}].IsLockedTarget} && !${Entity[${TargetID}].BeingTargeted}
            {
                Entity[${TargetID}]:LockTarget
            }
        }
    }

    ; SET ACTIVE TARGET TO MATCH MASTER
    if ${SlaveActiveTarget} > 0 && ${Entity[${SlaveActiveTarget}](exists)}
    {
        ; Only set active if it's actually locked
        if ${Entity[${SlaveActiveTarget}].IsLockedTarget}
        {
            if !${Me.ActiveTarget(exists)} || ${Me.ActiveTarget.ID} != ${SlaveActiveTarget}
            {
                Entity[${SlaveActiveTarget}]:MakeActiveTarget
            }
        }
    }
}
```

**Logic**:
1. Loop through all targets from master
2. If target exists and not locked → Lock it
3. If active target from master exists and is locked → Make it active
4. Simple, direct, effective

**Issue**: No cooldown between lock attempts (will spam LockTarget every pulse).

**Fix**:
```lavish
; Add cooldown tracking
variable index:int64 LastLockAttempt

; Before locking:
if ${Entity[${TargetID}](exists)}
{
    variable int LastAttempt = ${LastLockAttempt.Get[${TargetID}]}

    if !${Entity[${TargetID}].IsLockedTarget} && !${Entity[${TargetID}].BeingTargeted}
    {
        ; Only lock if cooldown passed
        if ${Math.Calc[${CurrentTime} - ${LastAttempt}]} > ${SLAVE_LOCK_COOLDOWN}
        {
            Entity[${TargetID}]:LockTarget
            LastLockAttempt:Set[${TargetID}, ${CurrentTime}]
        }
    }
}
```

---

## Movement and Following

### Follow Master Pattern

File: `Yamfa.iss` (lines 444-495)

```lavish
function FollowMaster()
{
    if ${MovementMode.Equal[None]} || !${Me.InSpace} || !${ISXEVE.IsReady} || !${Me(exists)}
        return

    variable entity Master
    variable bool MasterFound = FALSE

    ; FIND MASTER ENTITY
    if !${MasterName.Equal[]}
    {
        if ${Entity["Name = \"${MasterName}\""](exists)}
        {
            Master:Set[${Entity["Name = \"${MasterName}\""]}]
            MasterFound:Set[TRUE]
        }
    }

    if !${MasterFound}
        return

    ; EXECUTE MOVEMENT COMMAND
    switch ${MovementMode}
    {
        case Orbit
            if !${CurrentMovementCommand.Equal["Orbit:${Master.ID}"]}
            {
                Master:Orbit[${FollowDistance}]
                CurrentMovementCommand:Set["Orbit:${Master.ID}"]
                wait ${Math.Rand[8,15]}    ; Randomized delay
            }
            break

        case KeepRange
            if !${CurrentMovementCommand.Equal["KeepRange:${Master.ID}"]}
            {
                Master:KeepAtRange[${FollowDistance}]
                CurrentMovementCommand:Set["KeepRange:${Master.ID}"]
                wait ${Math.Rand[8,15]}
            }
            break

        case Approach
            if !${CurrentMovementCommand.Equal["Approach:${Master.ID}"]}
            {
                Master:Approach
                CurrentMovementCommand:Set["Approach:${Master.ID}"]
                wait ${Math.Rand[8,15]}
            }
            break
    }
}
```

**Command Caching Pattern**:
```lavish
variable string CurrentMovementCommand = ""

; Only issue command if different from last command
if !${CurrentMovementCommand.Equal["Orbit:${Master.ID}"]}
{
    Master:Orbit[${FollowDistance}]
    CurrentMovementCommand:Set["Orbit:${Master.ID}"]
    wait ${Math.Rand[8,15]}
}
```

**Why Cache?**:
- Prevents spam (don't orbit every pulse)
- Reduces CPU load
- Looks more human (doesn't re-issue same command)

**Randomized Wait**:
```lavish
wait ${Math.Rand[8,15]}    ; Random 8-15 deciseconds (0.8-1.5 seconds)
```

**Why Random?**:
- Anti-detection: Bots with exact timing = suspicious
- Prevents synchronization artifacts (all slaves moving at exact same time)

### Movement Modes

File: `Yamfa.iss` (lines 594-633)

```lavish
function SetOrbit(int Distance=1000)
{
    MovementMode:Set[Orbit]
    FollowDistance:Set[${Distance}]
    CurrentMovementCommand:Set[""]    ; Clear cache to force new command
    call SaveConfig
    call UpdateUI
    echo "Orbit mode at ${Distance}m"
}

function SetKeepRange(int Distance=1000)
{
    MovementMode:Set[KeepRange]
    FollowDistance:Set[${Distance}]
    CurrentMovementCommand:Set[""]
    call SaveConfig
    call UpdateUI
    echo "Keep range mode at ${Distance}m"
}

function SetApproach(int Distance=500)
{
    MovementMode:Set[Approach]
    FollowDistance:Set[${Distance}]
    CurrentMovementCommand:Set[""]
    call SaveConfig
    call UpdateUI
    echo "Approach mode at ${Distance}m"
}

function StopMovement()
{
    MovementMode:Set[None]
    if ${ISXEVE.IsReady} && ${Me(exists)}
    {
        EVE:Execute[CmdStopShip]
    }
    call UpdateUI
    echo "Movement stopped"
}
```

**Usage** (from console):
```
run yamfa
; Wait for init

; In console:
SetOrbit 5000       ; Orbit master at 5000m
SetKeepRange 10000  ; Keep at range 10000m
SetApproach         ; Approach master
StopMovement        ; Stop following
```

---

## Code Strengths

### ✅ Excellent Patterns

**1. Hysteresis for Stability**
```lavish
; Don't immediately drop targets - wait 0.7 seconds
if ${Math.Calc[${CurrentTime} - ${LastSeen}]} < ${MASTER_HOLD_TIME}
{
    ; Keep target even though it disappeared momentarily
}
```
**Why Good**: Prevents flickering, reduces spam, looks more human.

**2. Change Detection Before Relay**
```lavish
; Hash current state
variable string CurrentHash = "${TargetIDList}:${MasterActiveTarget}"

; Only relay if changed
if !${CurrentHash.Equal[${LastRelayedTargetHash}]}
{
    relay "all other" ...
    LastRelayedTargetHash:Set[${CurrentHash}]
}
```
**Why Good**: Reduces network traffic, improves performance, prevents relay spam.

**3. Command Caching**
```lavish
; Only issue new command if different
if !${CurrentMovementCommand.Equal["Orbit:${Master.ID}"]}
{
    Master:Orbit[${FollowDistance}]
    CurrentMovementCommand:Set["Orbit:${Master.ID}"]
}
```
**Why Good**: Prevents command spam, reduces server load, looks more natural.

**4. Random Timing**
```lavish
wait ${Math.Rand[2,6]}    ; Main loop
wait ${Math.Rand[8,15]}   ; Movement commands
```
**Why Good**: Anti-detection, prevents synchronization, more human-like.

**5. Global Variable Relay Pattern**
```lavish
; Atom can't modify function variables directly
atom(script) YamfaTargetsRelay(string targetIDs, int64 relayactivetarget)
{
    ; Store in globals
    YamfaRelayedTargets:Set[${targetIDs}]
    YamfaRelayedActive:Set[${relayactivetarget}]
    YamfaNewRelayData:Set[TRUE]    ; Flag for processing
}

; Process in main loop
if ${YamfaNewRelayData}
{
    call ProcessRelayedTargets "${YamfaRelayedTargets}" ${YamfaRelayedActive}
    YamfaNewRelayData:Set[FALSE]
}
```
**Why Good**: Atoms can't directly modify local variables. This pattern bridges the gap.

---

## Code Weaknesses and Fixes

### ❌ Critical Issues

**1. Hardcoded Master Name**

**Current**:
```lavish
variable string MASTER_CHARACTER_NAME = "YourCharacterName"
```

**Problem**: Must edit code to change master.

**Fix**:
```lavish
; In Initialize():
function Initialize()
{
    ; Load from config
    call LoadConfig

    ; Determine role from config OR fleet status
    if ${Config.IsMaster}
    {
        echo "${Me.Name} is MASTER (from config)"
    }
    elseif ${Me.Fleet.IsMember[${Me.CharID}]} && ${Me.ToFleetMember.IsFleetCommander}
    {
        echo "${Me.Name} is MASTER (Fleet Commander)"
    }
    else
    {
        echo "${Me.Name} is SLAVE"

        ; Get master name from fleet
        if ${Me.Fleet.IsMember[${Me.CharID}]}
        {
            variable int64 fcID = ${Me.Fleet.FleetCommanderID}
            variable queue:fleetmember members
            Me.Fleet:GetMembers[members]

            variable iterator it
            members:GetIterator[it]
            if ${it:First(exists)}
            {
                do
                {
                    if ${it.Value.CharID} == ${fcID}
                    {
                        MasterName:Set["${it.Value.Name}"]
                        break
                    }
                }
                while ${it:Next(exists)}
            }
        }
    }
}
```

**2. No Error Handling**

**Current**:
```lavish
Entity[${TargetID}]:LockTarget
; What if it fails? What if entity doesn't exist anymore?
```

**Fix**:
```lavish
if ${Entity[${TargetID}](exists)}
{
    if ${Entity[${TargetID}].Distance} < ${Ship.OptimalTargetingRange}
    {
        Entity[${TargetID}]:LockTarget
    }
    else
    {
        echo "Target ${TargetID} out of range (${Entity[${TargetID}].Distance.Int}m)"
    }
}
else
{
    echo "Target ${TargetID} no longer exists"
    ; Remove from set
    SlaveTargetSet:Erase[${TargetID}]
}
```

**3. No Lock Cooldown on Slave**

**Current**: Spams LockTarget every pulse.

**Fix** (shown earlier):
```lavish
variable index:int64 LastLockAttempt

if ${Math.Calc[${CurrentTime} - ${LastLockAttempt.Get[${TargetID}]}]} > ${SLAVE_LOCK_COOLDOWN}
{
    Entity[${TargetID}]:LockTarget
    LastLockAttempt:Set[${TargetID}, ${CurrentTime}]
}
```

**4. Entity Query Performance**

**Current**:
```lavish
; Master queries ALL entities every pulse
EVE:QueryEntities[Targets]
```

**Problem**: Expensive query when you only need locked targets.

**Fix**:
```lavish
; Only query locked/locking targets
EVE:QueryEntities[Targets, "IsLockedTarget || BeingTargeted"]
```

**5. No Targeting Range Check**

**Current**: Tries to lock targets that may be out of range.

**Fix**:
```lavish
if ${Entity[${TargetID}].Distance} < ${Ship.OptimalTargetingRange}
{
    Entity[${TargetID}]:LockTarget
}
else
{
    echo "Target ${TargetID} out of range: ${Entity[${TargetID}].Distance.Int}m (max: ${Ship.OptimalTargetingRange.Int}m)"
}
```

---

## Improvement Roadmap

### Phase 1: Bug Fixes (Immediate)

1. **Fix Hardcoded Master**
   - Load from config or detect from fleet

2. **Add Error Handling**
   - Check entity exists before operations
   - Check targeting range
   - Handle relay failures

3. **Add Lock Cooldown**
   - Prevent lock spam on slaves

4. **Optimize Entity Queries**
   - Query only locked targets, not all entities

### Phase 2: Feature Enhancements

1. **Active Target Priority**
   ```lavish
   ; Slaves prioritize locking master's active target first
   if ${SlaveActiveTarget} > 0 && !${Entity[${SlaveActiveTarget}].IsLockedTarget}
   {
       Entity[${SlaveActiveTarget}]:LockTarget
       wait 10
   }

   ; Then lock others
   ```

2. **Distance-Based Following**
   ```lavish
   ; Follow closer if in combat, further if traveling
   if ${Ship.InCombat}
   {
       FollowDistance:Set[5000]    ; Close for logi/DPS
   }
   else
   {
       FollowDistance:Set[15000]   ; Far to avoid bumping
   }
   ```

3. **Emergency Scatter**
   ```lavish
   ; If master broadcasts danger, all warp to random safe spots
   relay "all other" -event Yamfa_Emergency

   atom Yamfa_Emergency()
   {
       echo "EMERGENCY: Scattering!"
       call RetreatSingle
   }
   ```

### Phase 3: Advanced Features

1. **Formation Flying**
   ```lavish
   ; Slaves arrange in formation around master
   variable int MySlaveNumber = 1  ; From config
   variable float FormationAngle = ${Math.Calc[360 / ${TotalSlaves} * ${MySlaveNumber}]}

   ; Orbit at angle
   Master:OrbitAtAngle[${FollowDistance}, ${FormationAngle}]
   ```

2. **Target Prioritization**
   ```lavish
   ; Lock priority targets first (webs, jams, etc.)
   variable index:int64 PriorityTargets

   ; Master broadcasts priority
   relay "all other" -event Yamfa_PriorityTarget ${PriorityTargetID}
   ```

3. **Multi-Master Support**
   ```lavish
   ; Wing commanders can also broadcast targets
   if ${Me.ToFleetMember.IsWingCommander}
   {
       ; Broadcast to wing only
       relay "wing" -event Yamfa_WingTargets "${TargetList}"
   }
   ```

---

## Lessons for the Community

### What Yamfa Teaches

**1. Simplicity Can Be Effective**
- 845 lines does the job
- No need for complex architecture for simple tasks
- Single file = easy to understand and modify

**2. Hysteresis Prevents Flickering**
- Don't immediately react to state changes
- Hold state for short duration (0.5-1 second)
- Reduces spam, looks more human

**3. Change Detection Saves Bandwidth**
- Hash current state
- Only transmit if changed
- Massively reduces relay spam

**4. Command Caching Reduces Load**
- Don't re-issue same command
- Store last command, compare before issuing new one
- Looks more natural, reduces server load

**5. Random Timing is Critical**
- Exact timing = bot detection
- Random delays (within range) = looks human
- Apply to waits, movements, all actions

### Patterns to Reuse

**✅ Use These**:
- Hysteresis pattern for state stability
- Change detection before broadcasting
- Command caching to prevent spam
- Random timing throughout
- Global variable relay pattern

**❌ Avoid These**:
- Hardcoded configuration values
- No error handling
- Querying all entities when you need subset
- Spam actions without cooldowns
- Single point of failure (hardcoded master)

### Example: Building Your Own Fleet Assist

Based on Yamfa patterns:

```lavish
; MyFleetAssist.iss

; CONFIGURATION
variable bool IsMaster = FALSE        ; From config
variable string MasterName = ""       ; Auto-detect from fleet

; TARGET TRACKING
variable index:int64 MyTargets        ; Targets I'm broadcasting/following
variable index:int64 TargetTimers     ; Last seen time per target
variable int HOLD_TIME = 7            ; 0.7 second hysteresis

; RELAY
variable string LastRelayHash = ""
variable int LastRelayTime = 0
variable int RELAY_INTERVAL = 10      ; 1 second minimum

; MAIN LOOP
function main()
{
    call Initialize

    while TRUE
    {
        if ${Me.InSpace} && ${ISXEVE.IsReady}
        {
            if ${IsMaster}
            {
                call MasterPulse
            }
            else
            {
                call SlavePulse
            }
        }

        wait ${Math.Rand[2,6]}
    }
}

function Initialize()
{
    ; Determine role from fleet
    if ${Me.Fleet.IsMember[${Me.CharID}]} && ${Me.ToFleetMember.IsFleetCommander}
    {
        IsMaster:Set[TRUE]
        echo "${Me.Name} is MASTER (Fleet Commander)"
    }
    else
    {
        echo "${Me.Name} is SLAVE"
        ; Get master name from fleet commander
        call DetectMaster
    }

    ; Register relay event
    LavishScript:RegisterEvent[MyFleetTargets]
    Event[MyFleetTargets]:AttachAtom[This:OnTargetsRelay]
}

function DetectMaster()
{
    if ${Me.Fleet.IsMember[${Me.CharID}]}
    {
        variable queue:fleetmember members
        Me.Fleet:GetMembers[members]

        variable iterator it
        members:GetIterator[it]
        if ${it:First(exists)}
        {
            do
            {
                if ${it.Value.IsFleetCommander}
                {
                    MasterName:Set["${it.Value.Name}"]
                    echo "Master detected: ${MasterName}"
                    break
                }
            }
            while ${it:Next(exists)}
        }
    }
}

function MasterPulse()
{
    variable int CurrentTime = ${Math.Calc[${LavishScript.RunningTime} / 100]}

    ; Get locked targets with hysteresis
    call UpdateTargetsWithHysteresis ${CurrentTime}

    ; Build target list
    variable string targetList = ""
    variable int i
    for (i:Set[1]; ${i} <= ${MyTargets.Used}; i:Inc)
    {
        if ${i} > 1
            targetList:Concat["|"]
        targetList:Concat[${MyTargets.Get[${i}]}]
    }

    ; Create hash
    variable string currentHash = "${targetList}:${Me.ActiveTarget.ID}"

    ; Relay if changed or interval passed
    if !${currentHash.Equal[${LastRelayHash}]} ||
       ${Math.Calc[${CurrentTime} - ${LastRelayTime}]} >= ${RELAY_INTERVAL}
    {
        relay "all other" -noredirect "Event[MyFleetTargets]:Execute[\"${targetList}\",${Me.ActiveTarget.ID}]"

        LastRelayHash:Set[${currentHash}]
        LastRelayTime:Set[${CurrentTime}]
    }
}

function SlavePulse()
{
    ; Lock targets from MyTargets
    variable int i
    for (i:Set[1]; ${i} <= ${MyTargets.Used}; i:Inc)
    {
        variable int64 targetID = ${MyTargets.Get[${i}]}

        if ${Entity[${targetID}](exists)} &&
           !${Entity[${targetID}].IsLockedTarget} &&
           !${Entity[${targetID}].BeingTargeted}
        {
            Entity[${targetID}]:LockTarget
        }
    }
}

atom(script) OnTargetsRelay(string targetIDs, int64 activeTarget)
{
    ; Clear and rebuild
    MyTargets:Clear

    if ${targetIDs.Find["|"]} > 0
    {
        variable int i = 1
        variable string id
        while ${i} <= 20
        {
            id:Set[${targetIDs.Token[${i},"|"]}]
            if ${id.Length} > 0
            {
                MyTargets:Insert[${id}]
            }
            else
            {
                break
            }
            i:Inc
        }
    }
    elseif ${targetIDs.Length} > 0
    {
        MyTargets:Insert[${targetIDs}]
    }
}
```

---

## Summary

### Yamfa Architecture

**Core Concept**: Master broadcasts locked targets → Slaves lock same targets

**Key Techniques**:
1. **Hysteresis** - Hold state for 0.7s to prevent flickering
2. **Change Detection** - Only relay when targets actually change
3. **Command Caching** - Don't spam same command
4. **Random Timing** - Varies delays for anti-detection
5. **Relay Events** - LavishScript inter-session communication

### Strengths

✅ Simple (845 lines, one file)
✅ Effective (does the job well)
✅ Hysteresis prevents spam
✅ Change detection reduces traffic
✅ Easy to understand and modify

### Weaknesses

❌ Hardcoded master name
❌ No error handling
❌ No lock cooldown
❌ Inefficient entity queries
❌ No targeting range checks

### For the Community

**Use Yamfa as**:
- Template for fleet assist bots
- Example of hysteresis pattern
- Reference for relay communication
- Starting point (then improve it!)

**Improve by**:
- Auto-detecting master from fleet
- Adding error handling
- Implementing lock cooldowns
- Optimizing queries
- Adding range checks

**Learn from**:
- Simplicity is powerful
- Hysteresis prevents problems
- Change detection saves resources
- Random timing looks human
- One file can be enough!

---

**File Statistics**:
- **Lines**: ~2100
- **Code Examples**: 25+
- **Patterns Documented**: 10+
- **Improvements Suggested**: 15+

**Yamfa Status**: Simple, effective fleet assist. Perfect learning example. Needs error handling and configuration improvements, but core patterns are excellent!

**Next**: File 27 will analyze Tehbot's combat system for advanced combat automation patterns.
