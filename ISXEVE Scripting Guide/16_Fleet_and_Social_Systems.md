# Fleet and Social Systems
## Complete Guide to Fleet Management, Social Interactions, and Yamfa-Style Coordination

**Part of**: EVE Online Bot Development - AI-Level Documentation
**Layer**: 3 - ISXEVE API Deep Dive
**Prerequisites**: Read files 01-13, 15 (Foundation + Core Objects + Entities + UI + Functions/Relay)
**Critical For**: Yamfa fleet assist bots, multi-boxing coordination, fleet combat bots

---

## Purpose of This Document

This file provides **exhaustive coverage** of:
- Fleet membership and management
- Fleet member objects and iteration
- Fleet commands (invite, kick, broadcast, fleet warp)
- Fleet positions (fleet boss, wing commander, squad commander)
- Social interactions (standings, contacts, corp/alliance)
- Local chat monitoring (enemy detection)
- Relay coordination patterns for fleet bots (Yamfa master/slave pattern)
- Fleet targeting coordination
- Fleet position and distance queries
- Common patterns from Yamfa and multi-boxing scripts
- Critical timing and safety patterns
- Fleet-specific gotchas

**Why This Is CRITICAL for Yamfa**:
- **Yamfa = Fleet assist bot** - Master coordinates slave targeting
- **Relay atoms enable fleet communication** (covered in File 08, applied here)
- **Fleet member queries** determine who to assist
- **Target broadcasting** via relay is core Yamfa pattern
- Understanding fleet API is **essential** for any fleet coordination bot

---

## Table of Contents

1. [Fleet System Overview](#fleet-system-overview)
2. [Fleet Membership](#fleet-membership)
3. [Fleet Object and Members](#fleet-object-and-members)
4. [Fleet Member Objects](#fleet-member-objects)
5. [Fleet Commands](#fleet-commands)
6. [Fleet Positions and Roles](#fleet-positions-and-roles)
7. [Social Systems](#social-systems)
8. [Local Chat and Pilot Monitoring](#local-chat-and-pilot-monitoring)
9. [Relay Fleet Coordination (Yamfa Pattern)](#relay-fleet-coordination-yamfa-pattern)
10. [Fleet Targeting Coordination](#fleet-targeting-coordination)
11. [Common Patterns from Yamfa](#common-patterns-from-yamfa)
12. [Multi-Boxing Patterns](#multi-boxing-patterns)
13. [Critical Gotchas](#critical-gotchas)
14. [Anti-Patterns](#anti-patterns)

---

## Fleet System Overview

### EVE Fleet Structure

**Fleet hierarchy**:
```
Fleet (up to 256 members)
├── Wing 1 (up to 75 members)
│   ├── Squad 1 (up to 10 members)
│   ├── Squad 2 (up to 10 members)
│   └── Squad 3 (up to 10 members)
├── Wing 2
│   └── ...
└── Wing 3
    └── ...
```

**Fleet roles**:
- **Fleet Boss** - Fleet leader (has full control)
- **Wing Commander** - Wing leader (can manage wing)
- **Squad Commander** - Squad leader (can manage squad)
- **Fleet Member** - Regular member

### Fleet in ISXEVE

**Key TLOs**:
- `${Me.InFleet}` - Am I in a fleet?
- `${Me.Fleet}` - Fleet object (if in fleet)
- `${Me.IsFleetBoss}` - Am I fleet boss?

**Wiki Reference**: `IsxeveWiki/ISXEVE/Fleet_(Object_Type).html`

---

## Fleet Membership

### Checking Fleet Status

```lavish
; Am I in a fleet?
if ${Me.InFleet}
{
    echo "In fleet"
}
else
{
    echo "Not in fleet"
}

; Get fleet ID
if ${Me.InFleet}
{
    echo "Fleet ID: ${Me.FleetID}"
}
```

### Fleet Size

```lavish
if ${Me.InFleet}
{
    echo "Fleet members: ${Me.Fleet.MemberCount}"
}
```

### Joining/Leaving Fleet

**ISXEVE does NOT have direct join/leave methods**. Must be done via UI or player action.

**Workaround**: Player manually creates/joins fleet, then bot operates within fleet.

---

## Fleet Object and Members

### Accessing Fleet Object

```lavish
if !${Me.InFleet}
{
    echo "ERROR: Not in fleet"
    return
}

variable fleet myFleet = ${Me.Fleet}

echo "Fleet has ${myFleet.MemberCount} members"
```

### Fleet Members

**Get member count**:
```lavish
variable int memberCount = ${Me.Fleet.MemberCount}
echo "Fleet size: ${memberCount}"
```

**Iterate fleet members** (1-indexed!):
```lavish
function ListFleetMembers()
{
    if !${Me.InFleet}
    {
        echo "Not in fleet"
        return
    }

    variable int memberCount = ${Me.Fleet.MemberCount}
    echo "Fleet members (${memberCount}):"

    variable int i
    for (i:Set[1]; ${i} <= ${memberCount}; i:Inc)
    {
        variable fleetmember member = ${Me.Fleet.Member[${i}]}

        if ${member(exists)}
        {
            echo "${i}: ${member.Name} (${member.CharID})"
        }
    }
}
```

---

## Fleet Member Objects

### Getting Fleet Member Objects

**By index**:
```lavish
variable fleetmember member = ${Me.Fleet.Member[1]}
```

**By name**:
```lavish
variable fleetmember member = ${Me.Fleet.Member["Bob"]}

if ${member(exists)}
{
    echo "Found ${member.Name}"
}
```

**By character ID**:
```lavish
variable int64 charID = 123456789
variable fleetmember member = ${Me.Fleet.Member[${charID}]}
```

### Fleet Member Properties

```lavish
variable fleetmember member = ${Me.Fleet.Member[1]}

; Identity
echo "Name: ${member.Name}"
echo "Char ID: ${member.CharID}"
echo "Corp: ${member.Corp}"
echo "Alliance: ${member.Alliance}"

; Fleet position
echo "Fleet Boss: ${member.IsFleetBoss}"
echo "Wing Commander: ${member.IsWingCommander}"
echo "Squad Commander: ${member.IsSquadCommander}"

; Location
echo "Solar System ID: ${member.SolarSystemID}"
echo "Station ID: ${member.StationID}"    ; If docked

; Ship
echo "Ship Type ID: ${member.ShipTypeID}"
```

### Fleet Member Methods

**Check if member is in same system**:
```lavish
function IsInSameSystem(string memberName)
{
    variable fleetmember member = ${Me.Fleet.Member["${memberName}"]}

    if !${member(exists)}
        return FALSE

    return ${Math.Calc[${member.SolarSystemID} == ${Me.SolarSystemID}]}
}

; Usage
if ${IsInSameSystem["Bob"]}
{
    echo "Bob is in same system"
}
```

**Get member's entity** (if on grid):
```lavish
function GetFleetMemberEntity(string memberName)
{
    ; Fleet member object does NOT give direct entity access
    ; Must find by name in entity query

    variable entity memberShip = ${Entity[Name = "${memberName}"]}

    if ${memberShip(exists)}
    {
        return ${memberShip.ID}
    }

    return 0
}
```

---

## Fleet Commands

### ISXEVE Fleet Command Limitations

**IMPORTANT**: ISXEVE has **LIMITED** fleet command support.

**What ISXEVE CAN do**:
- ✅ Query fleet membership
- ✅ Query fleet member info
- ✅ Check fleet roles

**What ISXEVE CANNOT do (directly)**:
- ❌ Invite to fleet
- ❌ Kick from fleet
- ❌ Broadcast targets
- ❌ Fleet warp
- ❌ Change fleet positions

**Workaround**: Use **relay atoms** for fleet coordination instead of native fleet commands.

### Fleet Broadcasts (Limited)

**Fleet broadcast types**:
- Target broadcast
- Location broadcast
- Enemy spotted
- Need assistance

**ISXEVE does NOT provide direct broadcast methods**. Must use UI or relay.

### Fleet Warp (Limited)

**Fleet warp** = Fleet boss warps entire fleet to destination.

**No direct ISXEVE method**. Would require UI interaction.

---

## Fleet Positions and Roles

### Checking Roles

**Fleet Boss**:
```lavish
if ${Me.IsFleetBoss}
{
    echo "I am fleet boss"
}

; Alternative
if ${Me.Fleet.IsLeader}
{
    echo "I am fleet leader"
}
```

**Wing Commander**:
```lavish
; Check via fleet member object
variable fleetmember me = ${Me.Fleet.Member[${Me.CharID}]}

if ${me.IsWingCommander}
{
    echo "I am wing commander"
}
```

**Squad Commander**:
```lavish
if ${me.IsSquadCommander}
{
    echo "I am squad commander"
}
```

### Role-Based Logic (Yamfa Master/Slave Pattern)

```lavish
function IsMaster()
{
    ; Define master character name
    variable string MASTER_NAME = "Bob"

    return ${Me.Name.Equal["${MASTER_NAME}"]}
}

function IsSlave()
{
    return ${Math.Calc[!${IsMaster}]}
}

; Usage
if ${IsMaster}
{
    call MasterLogic
}
else
{
    call SlaveLogic
}
```

---

## Social Systems

### Standings

**Standings** = Your relationship with another character/corp/alliance (-10 to +10).

**Get standing toward entity**:
```lavish
variable int64 entityID = ${Entity[...].ID}
variable float standing = ${Me.GetStanding[${entityID}]}

echo "Standing: ${standing}"

; Negative = hostile
if ${standing} < 0
{
    echo "Hostile!"
}

; Positive = friendly
if ${standing} > 0
{
    echo "Friendly"
}
```

**Common standing checks**:
```lavish
function IsHostile(int64 entityID)
{
    return ${Math.Calc[${Me.GetStanding[${entityID}]} < 0]}
}

function IsFriendly(int64 entityID)
{
    return ${Math.Calc[${Me.GetStanding[${entityID}]} > 0]}
}

function IsNeutral(int64 entityID)
{
    variable float standing = ${Me.GetStanding[${entityID}]}
    return ${Math.Calc[${standing} >= 0 && ${standing} <= 0]}
}
```

### Corp and Alliance

**Check corp/alliance membership**:
```lavish
echo "My Corp: ${Me.Corp} (${Me.CorpID})"
echo "My Alliance: ${Me.Alliance} (${Me.AllianceID})"

; Check if entity is in same corp
variable entity ship = ${Entity[...]}

if ${ship.CorpID} == ${Me.CorpID}
{
    echo "Same corp!"
}

; Check if in same alliance
if ${ship.AllianceID} == ${Me.AllianceID} && ${Me.AllianceID} > 0
{
    echo "Same alliance!"
}
```

---

## Local Chat and Pilot Monitoring

### Local Chat Object

**${Local}** = Local chat channel (all pilots in system).

```lavish
echo "Pilots in local: ${Local.PilotCount}"
```

### Iterating Local Pilots

```lavish
function ListLocalPilots()
{
    echo "Pilots in local (${Local.PilotCount}):"

    variable int i
    for (i:Set[1]; ${i} <= ${Local.PilotCount}; i:Inc)
    {
        variable pilot localPilot = ${Local.Pilot[${i}]}

        if ${localPilot(exists)}
        {
            echo "${i}: ${localPilot.Name}"
        }
    }
}
```

### Finding Specific Pilot in Local

```lavish
function IsPilotInLocal(string pilotName)
{
    variable pilot p = ${Local.Pilot["${pilotName}"]}
    return ${p(exists)}
}

; Usage
if ${IsPilotInLocal["Enemy"]}
{
    echo "ALERT: Enemy in local!"
}
```

### Hostile Detection in Local

```lavish
variable(global) string[] HOSTILE_PILOTS
HOSTILE_PILOTS:Insert["Enemy1"]
HOSTILE_PILOTS:Insert["Enemy2"]
HOSTILE_PILOTS:Insert["Enemy3"]

function CheckForHostilesInLocal()
{
    variable int i
    for (i:Set[1]; ${i} <= ${HOSTILE_PILOTS.Used}; i:Inc)
    {
        if ${IsPilotInLocal["${HOSTILE_PILOTS[${i}]}"]}
        {
            echo "ALERT: Hostile ${HOSTILE_PILOTS[${i}]} in local!"
            return TRUE
        }
    }

    return FALSE
}

; Main loop
while TRUE
{
    if ${CheckForHostilesInLocal}
    {
        call EmergencyDock
        break
    }

    wait 1000
}
```

---

## Relay Fleet Coordination (Yamfa Pattern)

### Overview

**Yamfa pattern**: Master bot finds targets, broadcasts target IDs to slaves via relay. Slaves lock/fire on broadcasted targets.

**Key components**:
1. **Relay atoms** - Declared with `function atom`, exposed via `relay` command
2. **Master bot** - Calls relay atoms on other sessions
3. **Slave bots** - Listen for relay calls, execute commanded actions

**See File 08 (Functions/Atoms) for relay atom details**.

### Master Bot Setup

```lavish
; Master declares relay group
variable(global) string RELAY_GROUP = "YamfaFleet"

function Master_Init()
{
    ; Join relay group
    relay "${RELAY_GROUP}" "all other local"

    echo "Master initialized in relay group ${RELAY_GROUP}"
}

; Master finds targets and broadcasts
function Master_FindAndBroadcastTarget()
{
    ; Find best target (e.g., nearest NPC)
    variable index:entity npcs
    EVE:GetEntities[npcs, IsNPC = TRUE && Distance < 100000]

    if ${npcs.Used} == 0
    {
        echo "No targets found"
        return
    }

    ; Find nearest
    variable int64 primaryTarget = 0
    variable float nearestDist = 999999999
    variable int i

    for (i:Set[1]; ${i} <= ${npcs.Used}; i:Inc)
    {
        variable entity npc = ${npcs.Get[${i}]}

        if !${npc(exists)}
            continue

        if ${npc.Distance} < ${nearestDist}
        {
            nearestDist:Set[${npc.Distance}]
            primaryTarget:Set[${npc.ID}]
        }
    }

    ; Broadcast target to slaves
    if ${primaryTarget} > 0
    {
        echo "Broadcasting primary target: ${Entity[${primaryTarget}].Name}"
        relay "${RELAY_GROUP}" TargetEntity ${primaryTarget}
    }
}
```

### Slave Bot Setup

```lavish
; Slave declares relay atoms
function atom TargetEntity(int64 entityID)
{
    echo "Master ordered target: ${entityID}"

    variable entity target = ${Entity[${entityID}]}

    if !${target(exists)}
    {
        echo "Target does not exist, ignoring"
        return
    }

    ; Lock target
    if !${target.IsLockedTarget} && !${target.IsLockingTarget}
    {
        target:LockTarget
        echo "Locking ${target.Name}"
    }
}

function atom StopFiring()
{
    echo "Master ordered cease fire"

    ; Deactivate all weapons
    call DeactivateAllWeapons
}

function Slave_Init()
{
    ; Join relay group
    relay "${RELAY_GROUP}" "all other local"

    ; Register atoms
    relay "${RELAY_GROUP}" atom TargetEntity
    relay "${RELAY_GROUP}" atom StopFiring

    echo "Slave initialized, listening for master commands"
}

function atom atexit()
{
    ; Clean up relay
    relay "${RELAY_GROUP}" quit
}
```

### Complete Master/Slave Pattern

```lavish
; ========================================
; MASTER BOT (Bob)
; ========================================

variable(global) string RELAY_GROUP = "YamfaFleet"

function main()
{
    ; Check if I'm master
    if !${Me.Name.Equal["Bob"]}
    {
        echo "I am not master, exiting"
        return
    }

    ; Init relay
    call Master_Init

    echo "Master bot started"

    while TRUE
    {
        ; Find and broadcast targets
        call Master_FindAndBroadcastTarget

        wait 5000    ; Every 5 seconds
    }
}

; ========================================
; SLAVE BOT (All other chars)
; ========================================

function main()
{
    ; Check if I'm slave
    if ${Me.Name.Equal["Bob"]}
    {
        echo "I am master, exiting (wrong script!)"
        return
    }

    ; Init relay
    call Slave_Init

    echo "Slave bot started, awaiting master commands"

    while TRUE
    {
        ; Slave waits for relay commands
        ; (Relay atoms are called automatically when master broadcasts)

        wait 1000
    }
}
```

---

## Fleet Targeting Coordination

### Assist Targeting Pattern

**Assist** = Lock what another fleet member has locked.

**ISXEVE does NOT have direct "assist" command**, but can query fleet member targets:

```lavish
; This is theoretical - actual implementation depends on ISXEVE version
function AssistFleetMember(string memberName)
{
    ; Find member's entity on grid
    variable entity memberShip = ${Entity[Name = "${memberName}"]}

    if !${memberShip(exists)}
    {
        echo "Member not on grid"
        return
    }

    ; Get member's target (if ISXEVE supports this - not all versions do)
    ; This is HYPOTHETICAL - verify in your ISXEVE version

    ; Alternative: Use relay to broadcast targets instead
}
```

**Yamfa approach**: Use relay instead of native assist.

### Target Switching Coordination

```lavish
; Master decides when to switch targets
function Master_ShouldSwitchTarget(int64 currentTargetID)
{
    variable entity currentTarget = ${Entity[${currentTargetID}]}

    ; Switch if current target dead
    if !${currentTarget(exists)}
        return TRUE

    ; Switch if current target too far
    if ${currentTarget.Distance} > 150000
        return TRUE

    ; Switch if current target low HP (if detectable)
    if ${currentTarget.ShieldPct} < 10 && ${currentTarget.ArmorPct} < 10
        return TRUE

    return FALSE
}

; Master broadcasts new target when needed
variable int64 currentPrimary = 0

while TRUE
{
    if ${Master_ShouldSwitchTarget[${currentPrimary}]}
    {
        call Master_FindAndBroadcastTarget
    }

    wait 2000
}
```

---

## Common Patterns from Yamfa

### Pattern 1: Master Target Selection

```lavish
; Simplified from Yamfa
function Yamfa_GetBestTarget()
{
    ; Priority system:
    ; 1. Frigates targeting fleet members (high threat)
    ; 2. Cruisers/Battlecruisers (medium threat)
    ; 3. Battleships (low threat, high value)

    variable index:entity npcs
    EVE:GetEntities[npcs, IsNPC = TRUE && Distance < 100000]

    if ${npcs.Used} == 0
        return 0

    ; Check for frigates targeting us
    variable int i
    for (i:Set[1]; ${i} <= ${npcs.Used}; i:Inc)
    {
        variable entity npc = ${npcs.Get[${i}]}

        if !${npc(exists)}
            continue

        if ${npc.GroupID} == 25 && ${npc.TargetingMe}    ; Frigate targeting me
        {
            return ${npc.ID}
        }
    }

    ; Fall back to nearest target
    variable int64 nearestID = 0
    variable float nearestDist = 999999999

    for (i:Set[1]; ${i} <= ${npcs.Used}; i:Inc)
    {
        variable entity npc = ${npcs.Get[${i}]}

        if !${npc(exists)}
            continue

        if ${npc.Distance} < ${nearestDist}
        {
            nearestDist:Set[${npc.Distance}]
            nearestID:Set[${npc.ID}]
        }
    }

    return ${nearestID}
}
```

### Pattern 2: Slave Weapon Activation on Relay Target

```lavish
; Slave maintains current target
variable(script) int64 currentTarget = 0

function atom TargetEntity(int64 entityID)
{
    echo "Master assigned target: ${entityID}"

    currentTarget:Set[${entityID}]

    ; Lock target
    variable entity target = ${Entity[${entityID}]}

    if ${target(exists)} && !${target.IsLockedTarget}
    {
        target:LockTarget
    }
}

; Slave main loop - activate weapons on current target
while TRUE
{
    if ${currentTarget} > 0
    {
        variable entity target = ${Entity[${currentTarget}]}

        ; Check target still valid
        if !${target(exists)}
        {
            currentTarget:Set[0]
        }
        elseif ${target.IsLockedTarget}
        {
            ; Activate weapons
            call ActivateAllWeapons ${currentTarget}
        }
    }

    wait 1000
}
```

### Pattern 3: Fleet Position Monitoring

```lavish
function CheckFleetOnGrid()
{
    if !${Me.InFleet}
        return TRUE

    variable int membersOnGrid = 0
    variable int totalMembers = ${Me.Fleet.MemberCount}
    variable int i

    for (i:Set[1]; ${i} <= ${totalMembers}; i:Inc)
    {
        variable fleetmember member = ${Me.Fleet.Member[${i}]}

        if !${member(exists)}
            continue

        ; Check if in same system
        if ${member.SolarSystemID} != ${Me.SolarSystemID}
            continue

        ; Check if on grid (can see their ship)
        variable entity memberShip = ${Entity[Name = "${member.Name}"]}

        if ${memberShip(exists)}
        {
            membersOnGrid:Inc
        }
    }

    echo "Fleet on grid: ${membersOnGrid} / ${totalMembers}"

    ; If too few fleet members on grid, abort
    if ${membersOnGrid} < 3
    {
        echo "WARNING: Most of fleet not on grid!"
        return FALSE
    }

    return TRUE
}
```

---

## Multi-Boxing Patterns

### Session-Based Coordination

**Multi-boxing** = Running multiple EVE clients (is1, is2, is3, etc.).

**Relay enables coordination**:
```lavish
; Each session runs same script
; Script determines role based on character name

function main()
{
    if ${Me.Name.Equal["Bob"]}
    {
        echo "I am master (is1)"
        call MasterMain
    }
    elseif ${Me.Name.Equal["Alice"]}
    {
        echo "I am slave 1 (is2)"
        call SlaveMain
    }
    elseif ${Me.Name.Equal["Charlie"]}
    {
        echo "I am slave 2 (is3)"
        call SlaveMain
    }
    else
    {
        echo "Unknown character, aborting"
        return
    }
}
```

### Synchronized Actions

```lavish
; Master broadcasts synchronized commands
function atom SynchronizedWarp(int64 entityID, int distance)
{
    echo "Fleet warp commanded by master"

    variable entity destination = ${Entity[${entityID}]}

    if ${destination(exists)}
    {
        destination:WarpTo[${distance}]
        echo "Warping to ${destination.Name}"
    }
}

; Master calls on all slaves
relay "all other" SynchronizedWarp ${stationID} 0

; All slaves warp simultaneously
```

---

## Critical Gotchas

### Gotcha 1: Fleet Member Iteration Can Change

**Fleet members join/leave**:

```lavish
; DANGEROUS - Fleet size might change during iteration
variable int memberCount = ${Me.Fleet.MemberCount}

variable int i
for (i:Set[1]; ${i} <= ${memberCount}; i:Inc)
{
    ; If member leaves, memberCount changes!
}

; SAFER - Snapshot member IDs
variable int64[] memberIDs
variable int i

for (i:Set[1]; ${i} <= ${Me.Fleet.MemberCount}; i:Inc)
{
    if ${Me.Fleet.Member[${i}](exists)}
    {
        memberIDs:Insert[${Me.Fleet.Member[${i}].CharID}]
    }
}

; Process snapshot
for (i:Set[1]; ${i} <= ${memberIDs.Used}; i:Inc)
{
    variable fleetmember member = ${Me.Fleet.Member[${memberIDs[${i}]}]}

    if ${member(exists)}
    {
        ; Process member
    }
}
```

### Gotcha 2: Fleet Member Not On Grid

**Fleet member object exists, but their ship isn't visible**:

```lavish
; Fleet member object
variable fleetmember member = ${Me.Fleet.Member["Bob"]}    ; Exists

; Their ship on grid
variable entity memberShip = ${Entity[Name = "Bob"]}      ; Might not exist!

; Always check entity exists
if ${memberShip(exists)}
{
    echo "Bob is on grid at ${memberShip.Distance}m"
}
```

### Gotcha 3: Relay Requires Same Relay Group

**Master and slaves must use same relay group name**:

```lavish
; BAD - Different group names
; Master:
relay "MasterGroup" atom TargetEntity

; Slave:
relay "SlaveGroup" atom TargetEntity
; WILL NOT WORK - different groups!

; GOOD - Same group
; Both:
variable(global) string RELAY_GROUP = "YamfaFleet"
relay "${RELAY_GROUP}" atom TargetEntity
```

### Gotcha 4: Relay Atoms Must Be Registered Before Calling

**Slave must register atom BEFORE master calls it**:

```lavish
; Slave script - MUST register atom first
function atom TargetEntity(int64 entityID)
{
    ; Implementation
}

; THIS MUST HAPPEN EARLY in script initialization
relay "YamfaFleet" atom TargetEntity

; Now master can call it
```

---

## Anti-Patterns

### Anti-Pattern 1: Using Native Fleet Commands (Limited Support)

```lavish
; BAD - Fleet broadcast not supported
; No direct method exists

; GOOD - Use relay instead
relay "all other" TargetEntity ${targetID}
```

### Anti-Pattern 2: Assuming Fleet Member On Grid

```lavish
; BAD
variable fleetmember member = ${Me.Fleet.Member["Bob"]}
variable entity bobShip = ${Entity[Name = "Bob"]}

echo "Distance to Bob: ${bobShip.Distance}"    ; CRASH if not on grid!

; GOOD
if ${bobShip(exists)}
{
    echo "Distance to Bob: ${bobShip.Distance}"
}
```

### Anti-Pattern 3: No Relay Cleanup

```lavish
; BAD - No relay cleanup
function main()
{
    relay "YamfaFleet" "all other local"
    relay "YamfaFleet" atom TargetEntity

    ; Script runs...
}
; Relay group still joined after exit!

; GOOD - Cleanup in atexit
function atom atexit()
{
    relay "YamfaFleet" quit
    echo "Left relay group"
}
```

---

## Summary and Key Takeaways

### Essential Fleet Checks

```lavish
; Am I in fleet?
if ${Me.InFleet}

; Fleet size
variable int members = ${Me.Fleet.MemberCount}

; Am I fleet boss?
if ${Me.IsFleetBoss}

; Iterate fleet members
variable int i
for (i:Set[1]; ${i} <= ${Me.Fleet.MemberCount}; i:Inc)
{
    variable fleetmember m = ${Me.Fleet.Member[${i}]}
    echo "${m.Name}"
}

; Check if pilot in local
if ${Local.Pilot["Enemy"](exists)}
    echo "Enemy in local!"
```

### Yamfa Master/Slave Pattern

**Master**:
1. Find best target
2. Broadcast target ID via relay: `relay "group" TargetEntity ${id}`

**Slave**:
1. Register relay atom: `relay "group" atom TargetEntity`
2. Wait for master commands
3. Lock/fire on broadcasted target

### Critical Rules

1. **Use relay for fleet coordination** - Native fleet commands limited
2. **Always check fleet member entity exists** - Fleet member ≠ on grid
3. **Register relay atoms early** - Before master can call them
4. **Clean up relay on exit** - Use `atexit` atom
5. **Snapshot fleet members** - List can change during iteration
6. **Check Local for hostiles** - Safety pattern for all bots

---

## Next Steps

**Move to Layer 4 (Bot Architecture Patterns)**:
- File 17: Main_Loop_and_State_Machines.md
- File 18: Decision_Making_and_Logic_Patterns.md
- File 19: Error_Handling_and_Recovery.md
- File 20: Performance_and_Timing.md

**Apply Knowledge**:
- Yamfa bots need relay coordination (this file + File 08)
- Multi-boxing needs fleet queries + relay
- Recognize Yamfa patterns in example scripts

---

## Wiki References

**Fleet Object**:
- `IsxeveWiki/ISXEVE/Fleet_(Object_Type).html` - Fleet object reference

**Fleet Member Object**:
- `IsxeveWiki/ISXEVE/FleetMember_(Object_Type).html` - Fleet member reference

**Local Object**:
- `IsxeveWiki/ISXEVE/Local_(Object_Type).html` - Local chat reference

**Relay System** (covered in File 08):
- `LavishScriptWiki/LavishScript/Commands/relay.html` - Relay command reference

---

**END OF FILE 16**
**Status**: Complete (~1400 lines)
**Next**: Update progress tracker, assess if Layer 3 complete, provide final session summary
