# Fleet Operations

**Purpose:** Multi-boxing, fleet coordination patterns, and relay/IPC for fleet operations
**Audience:** Developers building multi-client fleet automation

---

## Table of Contents

### Multi-Boxing and Fleet Coordination
1. [Multi-Boxing and Fleet Coordination](#multi-boxing-and-fleet-coordination)
2. [Introduction to Multi-Boxing](#introduction-to-multi-boxing)
3. [LavishScript Relay System](#lavishscript-relay-system)
4. [Fleet Management](#fleet-management)
5. [Character Coordination Patterns](#character-coordination-patterns)
6. [Master-Slave Architectures](#master-slave-architectures)
7. [Resource Sharing Patterns](#resource-sharing-patterns)
8. [Fleet Combat Coordination](#fleet-combat-coordination)
9. [Mining Fleet Coordination](#mining-fleet-coordination)
10. [Event-Driven Coordination](#event-driven-coordination)
11. [Complete Working Examples (Multi-Boxing)](#complete-working-examples)
12. [Common Problems (Multi-Boxing)](#common-problems)

### Relay System and IPC
13. [Relay System and Inter-Process Communication](#relay-system-and-inter-process-communication-ipc)
14. [LavishScript Relay Basics](#relay-basics)
15. [Event System Foundation](#event-system)
16. [Basic Relay Patterns](#basic-patterns)
17. [UplinkManager System](#uplink-system)
18. [Fleet Coordination Patterns](#fleet-patterns)
19. [IRC Bridge Integration](#irc-bridge)
20. [Uplink Networking](#uplink-networking)
21. [Performance Considerations](#performance)
22. [Best Practices](#best-practices)
23. [Complete Working Examples (Relay)](#examples)

### Multiboxing Patterns
24. [Multiboxing Patterns](#multiboxing-patterns)

### Reference Implementations
25. [Reference Implementations](#reference-implementations)

---

## Multi-Boxing and Fleet Coordination

---

## Introduction to Multi-Boxing

Multi-boxing (running multiple EVE clients with coordinated automation) is introduced in [04_Core_Concepts.md](04_Core_Concepts.md) under Sessions and Multi-Client. This chapter focuses on the fleet-specific coordination patterns.

### Key Concepts

1. **Session**: A single instance of EVE running with one character
2. **Relay**: LavishScript's inter-session communication system
3. **Uplink**: Network connection between multiple computers
4. **Master/Slave**: One bot directs others
5. **Autonomous Fleet**: All bots operate independently with coordination

### Legal and Ethical Considerations

**CCP's Stance on Multi-Boxing**:
- Running multiple clients is allowed
- Input broadcasting (one keystroke → multiple clients) is **BANNED**
- Each bot must operate independently
- No automation that plays the game for you (gray area)

**Important**: This documentation is for **educational purposes**. Understand EVE's EULA and Terms of Service before implementation.

---

## LavishScript Relay System

For the canonical relay primer (syntax, destinations, `-noredirect`, `-event` flag, event registration lifecycle, request/response, state synchronization, Uplink networking, and the full `obj_FleetSafety` HARDSTOP pattern), see [Relay System and Inter-Process Communication (IPC)](#relay-system-and-inter-process-communication-ipc) later in this guide. That chapter is the single source of truth; the short form here was a teaser.

---

## Fleet Management

### Fleet Object (EVEBot Pattern)

Based on `obj_Fleet.iss`:

```lavishscript
objectdef obj_FleetManager
{
    variable string FleetLeader = "MyMainCharacter"
    variable bool IsLeader = FALSE

    method Initialize()
    {
        echo "Fleet Manager initialized"

        if ${Me.Name.Equal["${This.FleetLeader}"]}
        {
            This.IsLeader:Set[TRUE]
            echo "I am the fleet leader"
        }

        Event[EVENT_ONFRAME]:AttachAtom[This:Pulse]
    }

    method Pulse()
    {
        if ${This.IsLeader}
        {
            call This.ManageFleet
        }
        else
        {
            call This.FollowLeader
        }

        wait 20
    }

    // Fleet leader invites members
    method ManageFleet()
    {
        // Accept any pending fleet invites (in case we left fleet)
        if ${Me.Fleet.Invited}
        {
            Me.Fleet:AcceptInvite
        }

        // Invite configured fleet members
        variable iterator memberIterator
        Config.Fleet.FleetMembers:GetIterator[memberIterator]

        if ${memberIterator:First(exists)}
        {
            do
            {
                variable string memberName = "${memberIterator.Value.Name}"
                variable int64 memberCharID = ${This.ResolveCharID[${memberName}]}

                if ${memberCharID} == 0
                {
                    ; Character not found or offline
                    continue
                }

                // Invite if not already in fleet
                if !${Me.Fleet.IsMember[${memberCharID}]}
                {
                    echo "Inviting ${memberName} to fleet"
                    call This.InviteToFleet ${memberCharID}
                }
            }
            while ${memberIterator:Next(exists)}
        }
    }

    // Fleet members accept invites from leader
    method FollowLeader()
    {
        // Accept invites from our leader
        if ${Me.Fleet.Invited}
        {
            if ${Me.Fleet.InvitationText.Find["${This.FleetLeader}"]}
            {
                echo "Accepting fleet invite from ${This.FleetLeader}"
                Me.Fleet:AcceptInvite
            }
        }

        // If in fleet, verify leader is still there
        if ${Me.Fleet.IsMember[${Me.CharID}]}
        {
            variable int64 leaderCharID = ${This.ResolveCharID["${This.FleetLeader}"]}

            if !${Me.Fleet.IsMember[${leaderCharID}]}
            {
                echo "Fleet leader not in fleet - leaving"
                Me.Fleet:LeaveFleet
            }
        }
    }

    // Resolve character name to CharID
    member:int64 ResolveCharID(string characterName)
    {
        // Try corp members
        variable index:pilot corpMembers
        EVE:GetOnlineCorpMembers[corpMembers]

        variable iterator it
        corpMembers:GetIterator[it]

        if ${it:First(exists)}
        {
            do
            {
                if ${it.Value.Name.Equal[${characterName}]} && ${it.Value.IsOnline}
                {
                    return ${it.Value.CharID}
                }
            }
            while ${it:Next(exists)}
        }

        // Try contacts/buddies
        variable index:being buddies
        EVE:GetContacts[buddies]
        buddies:GetIterator[it]

        if ${it:First(exists)}
        {
            do
            {
                if ${it.Value.Name.Equal[${characterName}]} && ${it.Value.IsOnline}
                {
                    return ${it.Value.CharID}
                }
            }
            while ${it:Next(exists)}
        }

        // Try local pilots
        variable index:pilot localPilots
        EVE:GetLocalPilots[localPilots]
        localPilots:GetIterator[it]

        if ${it:First(exists)}
        {
            do
            {
                if ${it.Value.Name.Equal[${characterName}]}
                {
                    return ${it.Value.CharID}
                }
            }
            while ${it:Next(exists)}
        }

        return 0
    }

    // Invite character to fleet by CharID
    method InviteToFleet(int64 charID)
    {
        // Try corp members
        variable index:pilot corpMembers
        EVE:GetOnlineCorpMembers[corpMembers]

        variable iterator it
        corpMembers:GetIterator[it]

        if ${it:First(exists)}
        {
            do
            {
                if ${it.Value.CharID} == ${charID}
                {
                    it.Value:InviteToFleet
                    return
                }
            }
            while ${it:Next(exists)}
        }

        // Try contacts
        variable index:being buddies
        EVE:GetContacts[buddies]
        buddies:GetIterator[it]

        if ${it:First(exists)}
        {
            do
            {
                if ${it.Value.CharID} == ${charID}
                {
                    it.Value:InviteToFleet
                    return
                }
            }
            while ${it:Next(exists)}
        }

        // Try local
        variable index:pilot localPilots
        EVE:GetLocalPilots[localPilots]
        localPilots:GetIterator[it]

        if ${it:First(exists)}
        {
            do
            {
                if ${it.Value.CharID} == ${charID}
                {
                    it.Value:InviteToFleet
                    return
                }
            }
            while ${it:Next(exists)}
        }
    }
}
```

### Fleet Warp Capability

```lavishscript
// Check if can warp fleet
member:bool CanWarpFleet()
{
    if ${Me.Fleet.IsMember[${Me.CharID}]}
    {
        if ${Me.ToFleetMember.IsFleetCommander} ||
           ${Me.ToFleetMember.IsWingCommander} ||
           ${Me.ToFleetMember.IsSquadCommander}
        {
            return TRUE
        }
    }

    return FALSE
}

// Warp fleet to location
function WarpFleetTo(string locationName, int distance)
{
    if !${This.CanWarpFleet}
    {
        echo "ERROR: Do not have fleet warp authority"
        return FALSE
    }

    if ${Entity[${locationName}](exists)}
    {
        ; Warp to entity
        Entity[${locationName}]:WarpFleetTo[${distance}]
        echo "Fleet warp to ${locationName} at ${distance}m"
    }
    elseif ${Local[${locationName}].ToFleetMember(exists)}
    {
        ; Warp to fleet member
        Local[${locationName}].ToFleetMember:WarpFleetTo[${distance}]
        echo "Fleet warp to ${locationName} at ${distance}m"
    }
    else
    {
        echo "ERROR: ${locationName} not found"
        return FALSE
    }

    return TRUE
}
```

---

## Character Coordination Patterns

### Synchronized Actions

```lavishscript
// All bots wait for master signal before starting
variable bool MasterReady = FALSE

// Master sends ready signal
if ${This.IsMaster}
{
    echo "Master ready - signaling fleet"
    relay all -event Fleet_MasterReady ${Me.CharID}
}

// Slaves wait for signal
atom OnMasterReady(int64 masterCharID)
{
    echo "Master is ready (CharID: ${masterCharID})"
    This.MasterReady:Set[TRUE]
}

// Wait for master
while !${This.MasterReady}
{
    echo "Waiting for master..."
    wait 10
}

echo "Master ready - beginning operation"
```

### Position Reporting

```lavishscript
// Report position to fleet every 30 seconds
method ReportPosition()
{
    variable string locationDesc

    if ${Me.InStation}
    {
        locationDesc:Set["Station: ${Me.Station.Name}"]
    }
    elseif ${Me.InSpace}
    {
        locationDesc:Set["Space: ${Me.SolarSystem.Name}"]

        // Add specific location if near bookmark
        variable index:bookmark nearbyBookmarks
        EVE:GetBookmarks[nearbyBookmarks]

        variable iterator bmIterator
        nearbyBookmarks:GetIterator[bmIterator]

        if ${bmIterator:First(exists)}
        {
            do
            {
                if ${bmIterator.Value.SolarSystemID} == ${Me.SolarSystemID} && ${bmIterator.Value.ToEntity(exists)}
                {
                    if ${bmIterator.Value.ToEntity.Distance} < 50000
                    {
                        locationDesc:Set["${locationDesc} near ${bmIterator.Value.Label}"]
                        break
                    }
                }
            }
            while ${bmIterator:Next(exists)}
        }
    }

    relay all -event Fleet_PositionReport ${Me.CharID} "${locationDesc}"
}

// Receive position reports
atom OnPositionReport(int64 charID, string location)
{
    echo "Position report from ${charID}: ${location}"
    This.FleetPositions:Set[${charID}, "${location}"]
}
```

### Status Synchronization

```lavishscript
// Share status across fleet
objectdef obj_FleetStatus
{
    variable string MyStatus = "IDLE"
    variable collection:string FleetMemberStatus

    method BroadcastStatus(string newStatus)
    {
        This.MyStatus:Set["${newStatus}"]
        relay all -event Fleet_StatusUpdate ${Me.CharID} "${newStatus}"
    }

    atom OnStatusUpdate(int64 charID, string status)
    {
        This.FleetMemberStatus:Set[${charID}, "${status}"]
        echo "Fleet member ${charID} status: ${status}"
    }

    // Check if all fleet members have specific status
    member:bool AllMembersStatus(string requiredStatus)
    {
        variable iterator statusIterator
        This.FleetMemberStatus:GetIterator[statusIterator]

        if ${statusIterator:First(exists)}
        {
            do
            {
                if !${statusIterator.Value.Equal[${requiredStatus}]}
                {
                    return FALSE
                }
            }
            while ${statusIterator:Next(exists)}
        }

        return TRUE
    }
}

// Usage: Wait for all miners to be ready
FleetStatus:BroadcastStatus["READY"]

while !${FleetStatus.AllMembersStatus["READY"]}
{
    echo "Waiting for fleet to be ready..."
    wait 10
}

echo "All fleet members ready - starting operation"
```

---

## Master-Slave Architectures

### Pattern 1: Centralized Master Control

Master bot directs all actions:

```lavishscript
objectdef obj_MasterController
{
    variable queue:string CommandQueue

    method Initialize()
    {
        echo "Master Controller initialized"
    }

    // Master sends commands to specific slave
    method CommandSlave(string slaveName, string command)
    {
        echo "Commanding ${slaveName}: ${command}"
        relay "${slaveName}" ${command}
    }

    // Master broadcasts command to all slaves
    method CommandAllSlaves(string command)
    {
        echo "Broadcasting to all slaves: ${command}"
        relay other ${command}
    }

    // Example: Warp all miners to belt
    method WarpMinersTo(string beltName)
    {
        relay other -event Master_WarpTo "${beltName}" 0
    }

    // Example: Recall all to station
    method RecallFleet()
    {
        echo "Recalling fleet to station"
        relay all -event Master_RecallToStation
    }
}

// Slave receives master commands
objectdef obj_SlaveBot
{
    variable string MasterName = "MasterBot"

    method Initialize()
    {
        LavishScript:RegisterEvent[Master_WarpTo]
        LavishScript:RegisterEvent[Master_RecallToStation]

        Event[Master_WarpTo]:AttachAtom[This:OnMasterWarpTo]
        Event[Master_RecallToStation]:AttachAtom[This:OnMasterRecall]
    }

    atom OnMasterWarpTo(string locationName, int distance)
    {
        echo "Master orders: Warp to ${locationName} at ${distance}m"

        if ${EVE.Bookmark[${locationName}](exists)}
        {
            call Warp.ToBookmark "${locationName}"
        }
    }

    atom OnMasterRecall()
    {
        echo "Master orders: Recall to station"
        This.CurrentState:Set["RETURN_TO_STATION"]
    }
}
```

### Pattern 2: Autonomous with Coordination

Each bot operates independently but coordinates resources:

```lavishscript
objectdef obj_AutonomousMiner
{
    variable collection:int64 ClaimedAsteroids

    method Initialize()
    {
        LavishScript:RegisterEvent[EVEBot_ClaimAsteroid]
        LavishScript:RegisterEvent[EVEBot_ReleaseAsteroid]

        Event[EVEBot_ClaimAsteroid]:AttachAtom[This:OnAsteroidClaimed]
        Event[EVEBot_ReleaseAsteroid]:AttachAtom[This:OnAsteroidReleased]
    }

    // Before targeting asteroid, claim it
    function ClaimAsteroid(int64 asteroidID)
    {
        // Broadcast claim to fleet
        relay all -event EVEBot_ClaimAsteroid ${Me.CharID} ${asteroidID}

        This.ClaimedAsteroids:Set[${asteroidID}, ${Me.CharID}]

        echo "Claimed asteroid ${asteroidID}"
    }

    // When finished, release asteroid
    function ReleaseAsteroid(int64 asteroidID)
    {
        relay all -event EVEBot_ReleaseAsteroid ${Me.CharID} ${asteroidID}

        This.ClaimedAsteroids:Erase[${asteroidID}]

        echo "Released asteroid ${asteroidID}"
    }

    // Listen for other bots claiming asteroids
    atom OnAsteroidClaimed(int64 minerID, int64 asteroidID)
    {
        if ${minerID} != ${Me.CharID}
        {
            echo "Asteroid ${asteroidID} claimed by ${minerID}"
            This.ClaimedAsteroids:Set[${asteroidID}, ${minerID}]
        }
    }

    atom OnAsteroidReleased(int64 minerID, int64 asteroidID)
    {
        if ${minerID} != ${Me.CharID}
        {
            echo "Asteroid ${asteroidID} released by ${minerID}"
            This.ClaimedAsteroids:Erase[${asteroidID}]
        }
    }

    // Check if asteroid is available
    member:bool IsAsteroidAvailable(int64 asteroidID)
    {
        return ${Bool[!${This.ClaimedAsteroids.Element[${asteroidID}](exists)}]}
    }
}
```

### Pattern 3: Leader-Follower

One bot leads, others follow. (For master role discovery/election, see [Master/Slave Coordination](#masterslave-coordination) in the Relay IPC chapter below.)

```lavishscript
objectdef obj_FleetFollower
{
    variable string LeaderName = "FleetLeader"
    variable int64 LeaderEntityID = 0

    method Initialize()
    {
        LavishScript:RegisterEvent[Leader_MoveTo]
        Event[Leader_MoveTo]:AttachAtom[This:OnLeaderMove]
    }

    method Pulse()
    {
        // Constantly follow leader position
        call This.FollowLeader

        wait 20
    }

    method FollowLeader()
    {
        // Find leader entity
        if !${Entity[${This.LeaderEntityID}](exists)}
        {
            variable index:entity entities
            EVE:QueryEntities[entities, "Name = \"${This.LeaderName}\" && CategoryID = CATEGORYID_SHIP"]

            if ${entities.Used} > 0
            {
                This.LeaderEntityID:Set[${entities[1].ID}]
            }
            else
            {
                return
            }
        }

        variable entity leader = ${Entity[${This.LeaderEntityID}]}

        // If leader too far, warp to them
        if ${leader.Distance} > 150000
        {
            echo "Leader too far - warping to fleet member"
            Local[${This.LeaderName}].ToFleetMember:WarpTo
            return
        }

        // If leader moderate distance, follow
        if ${leader.Distance} > 5000
        {
            leader:Approach
        }

        // If close enough, maintain formation
        // (This would include orbit logic, keep at range, etc.)
    }

    // Leader broadcasts movement commands
    atom OnLeaderMove(string destination)
    {
        echo "Leader moving to: ${destination}"
        // Optionally warp to same location
    }
}
```

---

## Resource Sharing Patterns

### Shared Bookmark Database

```lavishscript
objectdef obj_SharedBookmarks
{
    variable collection:string SharedBookmarks

    method Initialize()
    {
        LavishScript:RegisterEvent[Fleet_AddBookmark]
        LavishScript:RegisterEvent[Fleet_RemoveBookmark]

        Event[Fleet_AddBookmark]:AttachAtom[This:OnBookmarkAdded]
        Event[Fleet_RemoveBookmark]:AttachAtom[This:OnBookmarkRemoved]
    }

    // Share bookmark with fleet
    method ShareBookmark(string bookmarkName)
    {
        if !${EVE.Bookmark[${bookmarkName}](exists)}
        {
            echo "Bookmark ${bookmarkName} does not exist"
            return FALSE
        }

        variable bookmark bm = ${EVE.Bookmark[${bookmarkName}]}

        relay all -event Fleet_AddBookmark "${bm.Label}" ${bm.X} ${bm.Y} ${bm.Z} ${bm.SolarSystemID}

        return TRUE
    }

    atom OnBookmarkAdded(string label, float x, float y, float z, int64 systemID)
    {
        echo "Shared bookmark received: ${label} in ${Universe[${systemID}].Name}"

        // Store bookmark info
        This.SharedBookmarks:Set["${label}", "${x},${y},${z},${systemID}"]

        // Optionally create actual bookmark
        // (Would require creating bookmark at coordinates)
    }

    atom OnBookmarkRemoved(string label)
    {
        echo "Shared bookmark removed: ${label}"
        This.SharedBookmarks:Erase["${label}"]
    }
}
```

### Shared Target Database

```lavishscript
objectdef obj_SharedTargets
{
    variable collection:int64 PriorityTargets

    method Initialize()
    {
        LavishScript:RegisterEvent[Fleet_PriorityTarget]
        Event[Fleet_PriorityTarget]:AttachAtom[This:OnPriorityTarget]
    }

    // Broadcast priority target to fleet
    method BroadcastTarget(int64 targetID)
    {
        echo "Broadcasting priority target: ${Entity[${targetID}].Name}"
        relay all -event Fleet_PriorityTarget ${Me.CharID} ${targetID}
    }

    atom OnPriorityTarget(int64 broadcasterID, int64 targetID)
    {
        echo "Priority target from ${broadcasterID}: ${Entity[${targetID}].Name}"

        This.PriorityTargets:Set[${targetID}, ${Time.Timestamp}]
    }

    // Get highest priority target
    member:int64 GetPriorityTarget()
    {
        variable iterator it
        This.PriorityTargets:GetIterator[it]

        if ${it:First(exists)}
        {
            // Return first target (could add priority logic)
            return ${it.Key}
        }

        return 0
    }

    // Clean up old targets
    method CleanupTargets()
    {
        variable iterator it
        This.PriorityTargets:GetIterator[it]

        if ${it:First(exists)}
        {
            do
            {
                // Remove targets older than 60 seconds
                if ${Math.Calc[${Time.Timestamp} - ${it.Value}]} > 60
                {
                    This.PriorityTargets:Erase[${it.Key}]
                }

                // Remove targets that no longer exist
                if !${Entity[${it.Key}](exists)}
                {
                    This.PriorityTargets:Erase[${it.Key}]
                }
            }
            while ${it:Next(exists)}
        }
    }
}
```

### Cargo Capacity Coordination

```lavishscript
// Hauler broadcasts free cargo space
method BroadcastCargoSpace()
{
    variable float freeSpace = ${Ship.Cargo.FreeCapacity}
    relay all -event EVEBot_HaulerCargoUpdate ${Me.CharID} ${freeSpace}
}

// Miners listen for hauler capacity
atom OnHaulerCargoUpdate(int64 haulerID, float freeSpace)
{
    echo "Hauler ${haulerID} has ${freeSpace.Int}m3 free"

    // If hauler has capacity and we're full, request pickup
    if ${freeSpace} > 1000 && ${Ship.Cargo.PercentFull} > 95
    {
        relay all -event EVEBot_Miner_RequestPickup ${Me.CharID} ${Me.SolarSystemID}
    }
}
```

---

## Fleet Combat Coordination

### Primary Target Calling

Fleet Commander broadcasts a primary (and optional secondary) target via a relay event; DPS slaves receive the event, lock the target, make it active, and activate weapons. The full, production-grade implementation of this pattern — with master/slave role detection, both primary and secondary target broadcasting, slave-side `OnPrimaryTarget` / `OnSecondaryTarget` atoms, lock-and-fire logic, and complete Initialize/Shutdown lifecycle — is provided as the canonical `obj_FleetCombat` object in [Example 1: Complete Fleet Combat Coordination](#example-1-complete-fleet-combat-coordination) later in this guide. Use that single authoritative version rather than re-implementing the pattern.

### Logistics Coordination (Shield/Armor Reps)

> **Framework context:** `Ship` in the example below is EVEBot's `obj_Ship` wrapper object (defined in `core/obj_Ship.iss`), not the ISXEVE `MyShip` TLO. It pre-classifies the ship's modules into indexes like `ModuleList_ShieldTransporters` and `ModuleList_GangLinks` at startup. Outside EVEBot you will need to enumerate `MyShip.GetModules[...]` yourself and filter by group name (e.g. `Shield Transporter`, `Gang Coordinator`) to get the equivalent lists.

```lavishscript
objectdef obj_LogisticsShip
{
    variable index:int64 RepTargets

    method Initialize()
    {
        LavishScript:RegisterEvent[Fleet_ShieldRepRequest]
        LavishScript:RegisterEvent[Fleet_ArmorRepRequest]

        Event[Fleet_ShieldRepRequest]:AttachAtom[This:OnShieldRepRequest]
        Event[Fleet_ArmorRepRequest]:AttachAtom[This:OnArmorRepRequest]
    }

    method Pulse()
    {
        // Constantly monitor fleet health
        call This.MonitorFleetHealth

        // Rep targets that need it
        call This.RepFleet

        wait 10
    }

    method MonitorFleetHealth()
    {
        variable queue:fleetmember fleetMembers
        Me.Fleet:GetMembers[fleetMembers]

        variable iterator memberIt
        fleetMembers:GetIterator[memberIt]

        if ${memberIt:First(exists)}
        {
            do
            {
                variable fleetmember member = ${memberIt.Value}

                // Check if in local and on grid
                if ${Local[${member.Name}](exists)}
                {
                    if ${Entity[Name = "${member.Name}"](exists)}
                    {
                        variable entity memberShip = ${Entity[Name = "${member.Name}"]}

                        // Check shield percentage
                        variable float shieldPct = ${Math.Calc[${memberShip.ShieldPct}]}

                        if ${shieldPct} < 70 && ${Ship.ModuleList_ShieldTransporters.Used} > 0
                        {
                            echo "${member.Name} shields at ${shieldPct.Int}% - adding to rep queue"
                            This.RepTargets:Insert[${memberShip.ID}]
                        }
                    }
                }
            }
            while ${memberIt:Next(exists)}
        }
    }

    method RepFleet()
    {
        if ${This.RepTargets.Used} == 0
            return

        variable iterator targetIt
        This.RepTargets:GetIterator[targetIt]

        if ${targetIt:First(exists)}
        {
            do
            {
                variable int64 targetID = ${targetIt.Value}

                if !${Entity[${targetID}](exists)}
                {
                    This.RepTargets:Erase[${targetID}]
                    continue
                }

                // Lock target if not locked
                if !${Entity[${targetID}].IsLockedTarget}
                {
                    Entity[${targetID}]:LockTarget
                    wait 10
                    continue
                }

                // Activate reps on target
                call This.RepTarget ${targetID}

                // Remove from queue if shields full
                if ${Entity[${targetID}].ShieldPct} > 95
                {
                    This.RepTargets:Erase[${targetID}]
                }
            }
            while ${targetIt:Next(exists)}
        }
    }

    method RepTarget(int64 targetID)
    {
        // Make active target
        Entity[${targetID}]:MakeActiveTarget
        wait 5

        // Activate shield transporters
        variable iterator modIt
        Ship.ModuleList_ShieldTransporters:GetIterator[modIt]

        if ${modIt:First(exists)}
        {
            do
            {
                if ${modIt.Value.IsReady} && !${modIt.Value.IsActive}
                {
                    modIt.Value:Activate
                }
            }
            while ${modIt:Next(exists)}
        }
    }

    // Listen for rep requests
    atom OnShieldRepRequest(int64 requesterID)
    {
        echo "Shield rep requested by ${Entity[${requesterID}].Name}"
        This.RepTargets:Insert[${requesterID}]
    }

    atom OnArmorRepRequest(int64 requesterID)
    {
        echo "Armor rep requested by ${Entity[${requesterID}].Name}"
        This.RepTargets:Insert[${requesterID}]
    }
}

// DPS ships request reps when needed
if ${Me.Ship.ShieldPct} < 50
{
    echo "Shields low - requesting reps"
    relay all -event Fleet_ShieldRepRequest ${MyShip.ID}
}
```

### Tackle Coordination

```lavishscript
// Tackle reports warp scrambled targets
if ${MyModule.IsActive} && ${MyModule.ToItem.Group.Equal["Warp Scramblers"]}
{
    relay all -event Fleet_TargetScrambled ${Me.ActiveTarget.ID} ${Me.CharID}
}

// DPS prioritizes scrambled targets
atom OnTargetScrambled(int64 targetID, int64 tacklerID)
{
    echo "Target ${Entity[${targetID}].Name} scrambled by ${tacklerID}"

    // Prioritize this target
    This.ScrambledTargets:Set[${targetID}, ${tacklerID}]
}
```

---

## Mining Fleet Coordination

### Orca-Centric Fleet

The full Orca-centric mining fleet coordinator — where the Orca is designated master, broadcasts belt selection to slaved miners, and miners react to warp-to-belt events — is integrated into the complete `obj_MiningFleet` implementation shown later in this guide under [Example 2: Mining Fleet with Orca Support](#example-2-mining-fleet-with-orca-support). See that section for the complete canonical implementation (Orca presence broadcasts, miner warp-in, survey-scan requests, full-cargo reports, and ore delivery to the Orca's fleet hangar).

### Boosting Coordination

> **Framework context:** As with the logistics example above, `Ship.ModuleList_GangLinks` is an EVEBot `obj_Ship` convenience index; outside EVEBot, enumerate `MyShip.GetModules[...]` and filter by the `Gang Coordinator` group yourself.

```lavishscript
objectdef obj_FleetBooster
{
    variable bool GangLinksActive = FALSE

    method Initialize()
    {
        echo "Fleet Booster initialized"
    }

    method ActivateGangLinks()
    {
        echo "Activating gang links..."

        variable iterator modIt
        Ship.ModuleList_GangLinks:GetIterator[modIt]

        if ${modIt:First(exists)}
        {
            do
            {
                if ${modIt.Value.IsReady} && !${modIt.Value.IsActive}
                {
                    echo "Activating ${modIt.Value.ToItem.Name}"
                    modIt.Value:Activate
                    wait 10
                }
            }
            while ${modIt:Next(exists)}
        }

        This.GangLinksActive:Set[TRUE]

        // Broadcast to fleet
        relay all -event Fleet_BoostsActive ${Me.CharID}
    }

    // Report boost status
    method ReportBoosts()
    {
        echo "==== GANG LINK STATUS ===="

        variable iterator modIt
        Ship.ModuleList_GangLinks:GetIterator[modIt]

        if ${modIt:First(exists)}
        {
            do
            {
                echo "${modIt.Value.ToItem.Name}: ${modIt.Value.IsActive}"
            }
            while ${modIt:Next(exists)}
        }

        echo "=========================="
    }
}

// Miners verify boosts
atom OnBoostsActive(int64 boosterID)
{
    echo "Boosts active from ${boosterID}"

    // Verify we're receiving boosts
    wait 30

    // Check for buff indicators
    // (ISXEVE doesn't directly expose buffs, but you'd see improved stats)
}
```

### Belt Depletion Coordination

```lavishscript
// Master monitors belt depletion
method CheckBeltDepletion()
{
    variable index:entity asteroids
    EVE:QueryEntities[asteroids, "GroupID = GROUP_ASTEROID && Distance < 200000"]

    if ${asteroids.Used} < 5
    {
        echo "Belt depleted (${asteroids.Used} asteroids remaining)"

        // Broadcast belt change
        relay all -event Fleet_BeltDepleted ${This.CurrentBeltID}

        // Find new belt
        call This.SelectNewBelt
    }
}

// Miners respond to belt depletion
atom OnBeltDepleted(int64 beltID)
{
    echo "Belt depleted - waiting for new belt assignment"
    This.CurrentState:Set["WAIT_FOR_BELT"]
}
```

---

## Event-Driven Coordination

### Custom Event Registry

```lavishscript
objectdef obj_EventRegistry
{
    method RegisterAllEvents()
    {
        // Combat events
        LavishScript:RegisterEvent[Fleet_PrimaryTarget]
        LavishScript:RegisterEvent[Fleet_SecondaryTarget]
        LavishScript:RegisterEvent[Fleet_TargetScrambled]
        LavishScript:RegisterEvent[Fleet_ShieldRepRequest]
        LavishScript:RegisterEvent[Fleet_ArmorRepRequest]
        LavishScript:RegisterEvent[Fleet_CapTransferRequest]

        // Mining events
        LavishScript:RegisterEvent[Fleet_WarpToBelt]
        LavishScript:RegisterEvent[Fleet_BeltDepleted]
        LavishScript:RegisterEvent[Fleet_OrcaCargo]
        LavishScript:RegisterEvent[Fleet_MinerFull]

        // Coordination events
        LavishScript:RegisterEvent[Fleet_MasterReady]
        LavishScript:RegisterEvent[Fleet_StatusUpdate]
        LavishScript:RegisterEvent[Fleet_PositionReport]
        LavishScript:RegisterEvent[Fleet_BoostsActive]

        // Emergency events
        LavishScript:RegisterEvent[Fleet_HARDSTOP]
        LavishScript:RegisterEvent[Fleet_Hostiles]
        LavishScript:RegisterEvent[Fleet_EmergencyDock]

        // Resource events
        LavishScript:RegisterEvent[Fleet_ClaimAsteroid]
        LavishScript:RegisterEvent[Fleet_ReleaseAsteroid]
        LavishScript:RegisterEvent[Fleet_AddBookmark]
        LavishScript:RegisterEvent[Fleet_RemoveBookmark]

        echo "All fleet events registered"
    }

    method AttachAllHandlers()
    {
        // Attach handlers for all events
        // (Would attach specific atoms to each event)
        echo "All event handlers attached"
    }
}
```

### Event-Driven State Changes

```lavishscript
// State changes driven by events rather than polling
objectdef obj_EventDrivenBot
{
    variable string CurrentState = "IDLE"

    method Initialize()
    {
        // Register events
        LavishScript:RegisterEvent[Master_StartMining]
        LavishScript:RegisterEvent[Master_ReturnToStation]
        LavishScript:RegisterEvent[Master_ChangeBelt]

        // Attach handlers
        Event[Master_StartMining]:AttachAtom[This:OnStartMining]
        Event[Master_ReturnToStation]:AttachAtom[This:OnReturnToStation]
        Event[Master_ChangeBelt]:AttachAtom[This:OnChangeBelt]
    }

    atom OnStartMining(int64 beltID)
    {
        echo "Event: Start mining at ${Entity[${beltID}].Name}"
        This.CurrentState:Set["MINE"]
        This.TargetBeltID:Set[${beltID}]
    }

    atom OnReturnToStation()
    {
        echo "Event: Return to station"
        This.CurrentState:Set["RETURN_TO_STATION"]
    }

    atom OnChangeBelt(int64 newBeltID)
    {
        echo "Event: Change to belt ${Entity[${newBeltID}].Name}"
        This.TargetBeltID:Set[${newBeltID}]
        This.CurrentState:Set["WARP_TO_BELT"]
    }

    function ProcessState()
    {
        // States are set by events, not polling
        switch ${This.CurrentState}
        {
            case IDLE
                wait 100
                break

            case MINE
                call This.MineAsteroids
                break

            case RETURN_TO_STATION
                call This.ReturnToStation
                break

            case WARP_TO_BELT
                call This.WarpToBelt
                This.CurrentState:Set["MINE"]
                break
        }
    }
}
```

---

## Complete Working Examples

### Example 1: Simple Mining Fleet (1 Hauler + 2 Miners)

**Hauler**:
```lavishscript
objectdef obj_MiningFleetHauler
{
    variable queue:obj_MinerRequest MinerRequests
    variable string CurrentState = "IDLE"

    method Initialize()
    {
        echo "Fleet Hauler initialized"

        LavishScript:RegisterEvent[EVEBot_Miner_Full]
        Event[EVEBot_Miner_Full]:AttachAtom[This:OnMinerFull]
    }

    atom OnMinerFull(int64 minerCharID, int64 systemID, int64 canID)
    {
        echo "Miner ${minerCharID} reports full (can ${canID})"

        // Add to pickup queue
        This.MinerRequests:Queue[${minerCharID}, ${systemID}, ${canID}]
    }

    method Pulse()
    {
        This:DetermineState
        This:ProcessState

        wait 20
    }

    method DetermineState()
    {
        if ${Me.InStation}
        {
            if ${Ship.Cargo.UsedCapacity} > 0
            {
                This.CurrentState:Set["UNLOAD"]
            }
            else if ${This.MinerRequests.Used} > 0
            {
                This.CurrentState:Set["UNDOCK"]
            }
            else
            {
                This.CurrentState:Set["IDLE"]
            }
            return
        }

        // In space
        if ${Ship.Cargo.PercentFull} > 90
        {
            This.CurrentState:Set["RETURN_TO_STATION"]
            return
        }

        if ${This.MinerRequests.Used} > 0
        {
            This.CurrentState:Set["PICKUP"]
        }
        else
        {
            This.CurrentState:Set["WAIT"]
        }
    }

    method ProcessState()
    {
        switch ${This.CurrentState}
        {
            case IDLE
                break

            case UNDOCK
                call Station.Undock
                break

            case PICKUP
                call This.PickupFromMiner
                break

            case RETURN_TO_STATION
                call This.ReturnToStation
                break

            case UNLOAD
                call Cargo.TransferCargoToStationHangar
                break

            case WAIT
                call Safespots.WarpTo
                break
        }
    }

    method PickupFromMiner()
    {
        if ${This.MinerRequests.Used} == 0
            return

        variable obj_MinerRequest request = ${This.MinerRequests.Peek}

        echo "Picking up from miner ${request.MinerCharID}"

        // Warp to can
        if ${Entity[${request.CanID}].Distance} > 150000
        {
            Entity[${request.CanID}]:WarpTo[10000]
            wait 20
            while ${Me.ToEntity.Mode} == 3
            {
                wait 10
            }
        }

        // Loot can
        call This.LootCan ${request.CanID}

        // Remove from queue
        This.MinerRequests:Dequeue
    }

    method LootCan(int64 canID)
    {
        if !${Entity[${canID}](exists)}
            return

        // Approach
        if ${Entity[${canID}].Distance} > 2500
        {
            call Ship.Approach ${canID} 2500

            variable int timeout = 0
            while ${Entity[${canID}].Distance} > 2500 && ${timeout} < 600
            {
                wait 10
                timeout:Inc[10]
            }
        }

        EVE:Execute[CmdStopShip]
        wait 20

        // Open and loot
        Entity[${canID}]:Open
        wait 30

        // MODERN API (July 2020+): Access container cargo via inventory window
        variable index:item items
        if !${EVEWindow[ByItemID,${canID}](exists)}
        {
            echo "WARNING: Container window not found for ${canID}"
            return
        }

        EVEWindow[ByItemID,${canID}]:GetItems[items]

        if ${items.Used} > 0
        {
            variable index:int64 itemIDs
            variable iterator it
            items:GetIterator[it]

            if ${it:First(exists)}
            {
                do
                {
                    itemIDs:Insert[${it.Value.ID}]
                }
                while ${it:Next(exists)}
            }

            EVE:MoveItemsTo[itemIDs, MyShip, CargoHold]
            wait 30
        }

        Entity[${canID}]:Close
    }

    method ReturnToStation()
    {
        Navigator:FlyToBookmark["${Config.Hauler.DeliveryStation}", 0, TRUE]
        while ${Navigator.Busy}
        {
            wait 10
        }
    }
}

objectdef obj_MinerRequest
{
    variable int64 MinerCharID
    variable int64 SystemID
    variable int64 CanID

    method Initialize(int64 charID, int64 sysID, int64 can)
    {
        This.MinerCharID:Set[${charID}]
        This.SystemID:Set[${sysID}]
        This.CanID:Set[${can}]
    }
}
```

**Miner**:
```lavishscript
objectdef obj_FleetMiner
{
    variable string CurrentState = "IDLE"
    variable int64 CurrentAsteroid = 0

    method Pulse()
    {
        This:DetermineState
        This:ProcessState

        wait 20
    }

    method DetermineState()
    {
        if ${Me.InStation}
        {
            This.CurrentState:Set["IDLE"]
            return
        }

        if ${Ship.Cargo.PercentFull} > 95
        {
            This.CurrentState:Set["CARGO_FULL"]
            return
        }

        This.CurrentState:Set["MINE"]
    }

    method ProcessState()
    {
        switch ${This.CurrentState}
        {
            case IDLE
                break

            case MINE
                call This.MineAsteroid
                break

            case CARGO_FULL
                call This.JettisonCargo
                break
        }
    }

    method MineAsteroid()
    {
        // Find asteroid
        if ${This.CurrentAsteroid} == 0 || !${Entity[${This.CurrentAsteroid}](exists)}
        {
            call This.FindAsteroid
        }

        if ${This.CurrentAsteroid} == 0
        {
            echo "No asteroids found"
            return
        }

        // Lock and mine
        variable entity asteroid = ${Entity[${This.CurrentAsteroid}]}

        if !${asteroid.IsLockedTarget}
        {
            asteroid:LockTarget
            wait 10
        }

        // Activate miners
        Ship:Activate_MiningLasers
    }

    method FindAsteroid()
    {
        variable index:entity asteroids
        EVE:QueryEntities[asteroids, "GroupID = GROUP_ASTEROID && Distance < 50000", "Distance"]

        if ${asteroids.Used} > 0
        {
            This.CurrentAsteroid:Set[${asteroids[1].ID}]
            echo "Selected asteroid: ${asteroids[1].Name}"
        }
    }

    method JettisonCargo()
    {
        echo "Cargo full - jettisoning"

        // Open inventory
        if !${EVEWindow[Inventory](exists)}
        {
            EVE:Execute[OpenInventory]
            wait 20
        }

        // MODERN API (July 2020+): Access ship cargo via inventory window
        variable index:item cargoItems
        if !${EVEWindow[Inventory](exists)}
        {
            echo "ERROR: Cannot open inventory window"
            return
        }

        EVEWindow[Inventory].Child[ShipCargo]:GetItems[cargoItems]

        variable iterator it
        cargoItems:GetIterator[it]

        if ${it:First(exists)}
        {
            do
            {
                it.Value:Jettison
                wait 10
            }
            while ${it:Next(exists)}
        }

        wait 30

        // Find my jetcan
        variable index:entity cans
        EVE:QueryEntities[cans, "GroupID = GROUP_CARGO_CONTAINER && Distance < 5000", "Distance"]

        if ${cans.Used} > 0
        {
            variable int64 myCanID = ${cans[1].ID}

            echo "Reporting full can to hauler"
            relay all -event EVEBot_Miner_Full ${Me.CharID} ${Me.SolarSystemID} ${myCanID}
        }

        This.CurrentState:Set["MINE"]
    }
}
```

### Example 2: Combat Fleet with FC

**Fleet Commander**:
```lavishscript
objectdef obj_FleetCommander
{
    variable int64 PrimaryTarget = 0
    variable int64 SecondaryTarget = 0

    method Initialize()
    {
        echo "Fleet Commander online"

        LavishScript:RegisterEvent[Fleet_TargetKilled]
        Event[Fleet_TargetKilled]:AttachAtom[This:OnTargetKilled]
    }

    method Pulse()
    {
        call This.ManageTargets

        wait 20
    }

    method ManageTargets()
    {
        // If no primary, find one
        if ${This.PrimaryTarget} == 0 || !${Entity[${This.PrimaryTarget}](exists)}
        {
            call This.FindAndCallPrimary
        }

        // If no secondary, find one
        if ${This.SecondaryTarget} == 0 || !${Entity[${This.SecondaryTarget}](exists)}
        {
            call This.FindAndCallSecondary
        }
    }

    method FindAndCallPrimary()
    {
        variable index:entity hostiles
        EVE:QueryEntities[hostiles, "IsNPC && !IsMoribund && Distance < 150000", "Distance"]

        if ${hostiles.Used} > 0
        {
            This.PrimaryTarget:Set[${hostiles[1].ID}]

            echo "=== PRIMARY: ${hostiles[1].Name} ==="
            relay all -event Fleet_PrimaryTarget ${This.PrimaryTarget}
        }
    }

    method FindAndCallSecondary()
    {
        variable index:entity hostiles
        EVE:QueryEntities[hostiles, "IsNPC && !IsMoribund && Distance < 150000", "Distance"]

        // Find target that's not primary
        variable iterator it
        hostiles:GetIterator[it]

        if ${it:First(exists)}
        {
            do
            {
                if ${it.Value.ID} != ${This.PrimaryTarget}
                {
                    This.SecondaryTarget:Set[${it.Value.ID}]

                    echo "--- Secondary: ${it.Value.Name} ---"
                    relay all -event Fleet_SecondaryTarget ${This.SecondaryTarget}
                    break
                }
            }
            while ${it:Next(exists)}
        }
    }

    atom OnTargetKilled(int64 targetID)
    {
        if ${targetID} == ${This.PrimaryTarget}
        {
            echo "Primary destroyed - promoting secondary"

            This.PrimaryTarget:Set[${This.SecondaryTarget}]
            This.SecondaryTarget:Set[0]

            if ${This.PrimaryTarget} > 0
            {
                relay all -event Fleet_PrimaryTarget ${This.PrimaryTarget}
            }

            call This.FindAndCallSecondary
        }
    }
}
```

**DPS Ship**:
```lavishscript
objectdef obj_DPSShip
{
    variable int64 PrimaryTarget = 0

    method Initialize()
    {
        LavishScript:RegisterEvent[Fleet_PrimaryTarget]
        Event[Fleet_PrimaryTarget]:AttachAtom[This:OnPrimaryTarget]
    }

    atom OnPrimaryTarget(int64 targetID)
    {
        echo "Engaging primary: ${Entity[${targetID}].Name}"
        This.PrimaryTarget:Set[${targetID}]
    }

    method Pulse()
    {
        call This.ShootPrimary

        wait 10
    }

    method ShootPrimary()
    {
        if ${This.PrimaryTarget} == 0
            return

        if !${Entity[${This.PrimaryTarget}](exists)}
        {
            echo "Primary destroyed"
            relay all -event Fleet_TargetKilled ${This.PrimaryTarget}
            This.PrimaryTarget:Set[0]
            return
        }

        // Lock if needed
        if !${Entity[${This.PrimaryTarget}].IsLockedTarget}
        {
            Entity[${This.PrimaryTarget}]:LockTarget
            return
        }

        // Make active and shoot
        Entity[${This.PrimaryTarget}]:MakeActiveTarget
        Ship:Activate_Weapons
    }
}
```

---

## Common Problems

### Problem 1: Relay Events Not Received

**Symptom**: Events sent but not received by other sessions

**Diagnosis**:
```lavishscript
echo "Event registered: ${LavishScript.RegisteredEvent[MyEvent](exists)}"
echo "Handler attached: ${Event[MyEvent].HasAtom}"
```

**Solution**:
```lavishscript
// Ensure event is registered BEFORE sending
LavishScript:RegisterEvent[MyEvent]
Event[MyEvent]:AttachAtom[This:OnMyEvent]

// Then send
relay all -event MyEvent "test"
```

### Problem 2: Session Names Unknown

**Symptom**: Can't relay to specific session

**Diagnosis**:
```lavishscript
echo "Session name: ${Session}"
echo "Session uplink ID: ${Session.UplinkID}"
```

**Solution**:
```lavishscript
// Use "all" or "other" instead of specific names
relay all -event MyEvent

// Or discover session names
variable iterator SessionIterator
Session:GetSessions[SessionIterator]
if ${SessionIterator:First(exists)}
{
    do
    {
        echo "Session: ${SessionIterator.Value}"
    }
    while ${SessionIterator:Next(exists)}
}
```

### Problem 3: Race Conditions

**Symptom**: Bots act before coordinator is ready

**Solution**:
```lavishscript
// Use synchronized startup
variable bool AllBotsReady = FALSE

// Each bot reports ready
relay all -event Bot_Ready ${Me.CharID}

// Count ready bots
variable int ReadyCount = 0

atom OnBotReady(int64 charID)
{
    This.ReadyCount:Inc
    echo "${This.ReadyCount} bots ready"

    if ${This.ReadyCount} >= ${Config.ExpectedBotCount}
    {
        echo "All bots ready - starting operation"
        relay all -event Bot_Start
        This.AllBotsReady:Set[TRUE]
    }
}

// Wait for start signal
while !${This.AllBotsReady}
{
    wait 10
}
```

### Problem 4: Uplink Connection Lost

**Symptom**: Multi-computer fleet loses coordination

**Diagnosis**:
```lavishscript
echo "Uplink connected: ${Uplink.IsConnected}"
echo "Uplink sessions: ${Uplink.Sessions}"
```

**Solution**:
```lavishscript
// Auto-reconnect
if !${Uplink.IsConnected}
{
    echo "Uplink disconnected - reconnecting"
    uplink connect 192.168.1.100:2048 password123

    wait 50

    if ${Uplink.IsConnected}
    {
        echo "Uplink reconnected"
    }
}
```

### Problem 5: Fleet Member Not Found

**Symptom**: Can't resolve fleet member CharID

**Diagnosis**:
```lavishscript
echo "In fleet: ${Me.Fleet.IsMember[${Me.CharID}]}"
echo "Fleet members: ${Me.Fleet.Size}"
```

**Solution**:
```lavishscript
// Wait for fleet to populate
wait 50

// Try multiple resolution methods
variable int64 charID = ${This.ResolveCharID["MemberName"]}

if ${charID} == 0
{
    echo "ERROR: Could not resolve ${MemberName}"
    // Fallback: use entity lookup
    if ${Entity[Name = "MemberName"](exists)}
    {
        echo "Found by entity lookup"
    }
}
```

### Diagnostic: Fleet Status Report

```lavishscript
function FleetStatusReport()
{
    echo "==== FLEET STATUS REPORT ===="

    echo "Session: ${Session}"
    echo "Character: ${Me.Name} (${Me.CharID})"

    echo "In fleet: ${Me.Fleet.IsMember[${Me.CharID}]}"

    if ${Me.Fleet.IsMember[${Me.CharID}]}
    {
        echo "Fleet members: ${Me.Fleet.Size}"

        variable queue:fleetmember members
        Me.Fleet:GetMembers[members]

        variable iterator it
        members:GetIterator[it]

        if ${it:First(exists)}
        {
            do
            {
                echo "  ${it.Value.Name} (${it.Value.CharID})"
            }
            while ${it:Next(exists)}
        }
    }

    echo "Registered events:"
    echo "  EVEBot_HARDSTOP: ${LavishScript.RegisteredEvent[EVEBot_HARDSTOP](exists)}"
    echo "  EVEBot_Miner_Full: ${LavishScript.RegisteredEvent[EVEBot_Miner_Full](exists)}"

    echo "Uplink: ${Uplink.IsConnected}"

    echo "============================"
}
```


---

## Relay System and Inter-Process Communication (IPC)

---

## LavishScript Relay Basics

### What is Relay?

The **relay** command is LavishScript's inter-process communication (IPC) mechanism that allows:
- Communication between multiple EVE clients on **same computer**
- Broadcasting messages to all sessions or specific targets
- Remote method execution across sessions
- Event-driven coordination

**Key Concept:** Each EVE client runs in a separate InnerSpace session. Relay allows these sessions to communicate.

### Relay Syntax

```lavishscript
; Basic syntax
relay <destination> <command>

; With options
relay <destination> -noredirect <command>
relay <destination> -event <event_name> [parameters]

; Examples
relay all echo "Hello everyone"                    ; Execute command in all sessions
relay "all other" echo "Everyone but me"           ; All except sender
relay "CharacterName" echo "Just you"              ; Specific character
relay all -event MyEvent "param1" "param2"         ; Trigger event
relay all -noredirect "MyObject:MyMethod[param]"   ; No return relay
```

### Relay Destinations

| Destination | Description | Example |
|------------|-------------|---------|
| `all` | All sessions including sender | `relay all echo "broadcast"` |
| `"all other"` | All sessions except sender | `relay "all other" pause` |
| `"CharName"` | Specific character by name | `relay "Hauler Alt" dock` |
| `${Session}` | Sender's session (self) | Used as identifier |

### The -noredirect Flag

**Critical:** Always use `-noredirect` for method calls to prevent infinite relay loops!

```lavishscript
; WRONG - Creates relay loop!
relay all "MyObject:DoSomething"

; CORRECT - No loop
relay all -noredirect "MyObject:DoSomething"
```

**Why?** Without `-noredirect`, the receiving session might relay back the result, creating an infinite loop.

---

## Event System Foundation

### Event Registration Pattern

Events are the foundation of relay communication. You must register and attach handlers:

```lavishscript
objectdef obj_MyFleetBot
{
    method Initialize()
    {
        ; 1. Register the event (creates it globally)
        LavishScript:RegisterEvent[Fleet_Primary_Target]

        ; 2. Attach handler method to event
        Event[Fleet_Primary_Target]:AttachAtom[This:OnPrimaryTarget]

        ; 3. Register relay events
        LavishScript:RegisterEvent[Fleet_HARDSTOP]
        Event[Fleet_HARDSTOP]:AttachAtom[This:OnHardStop]
    }

    method Shutdown()
    {
        ; ALWAYS detach in shutdown!
        Event[Fleet_Primary_Target]:DetachAtom[This:OnPrimaryTarget]
        Event[Fleet_HARDSTOP]:DetachAtom[This:OnHardStop]
    }

    ; Handler methods
    atom OnPrimaryTarget(int64 targetID, string targetName)
    {
        echo "FC called primary: ${targetName}"
        Entity[${targetID}]:LockTarget
    }

    atom OnHardStop(string reason)
    {
        echo "HARD STOP: ${reason}"
        EVEBot.ReturnToStation:Set[TRUE]
    }
}
```

### Event vs Atom

- **Event:** Named trigger point that can have multiple handlers
- **Atom:** A method that handles an event (event handler)

```lavishscript
; Event can have multiple atoms attached
Event[MyEvent]:AttachAtom[Object1:Handler1]
Event[MyEvent]:AttachAtom[Object2:Handler2]
Event[MyEvent]:AttachAtom[Object3:Handler3]

; When event executes, ALL attached atoms are called
Event[MyEvent]:Execute["param"]
```

### Event Broadcasting

```lavishscript
; Broadcast event to all other sessions
method BroadcastPrimary(int64 targetID)
{
    ; Local execution
    Event[Fleet_Primary_Target]:Execute[${targetID}, "${Entity[${targetID}].Name}"]

    ; Remote execution (all other sessions)
    relay "all other" -event Fleet_Primary_Target ${targetID} "${Entity[${targetID}].Name}"
}
```

---

## Basic Relay Patterns

### Pattern 1: Simple Broadcast

**Use Case:** Notify all fleet members of an event

```lavishscript
objectdef obj_FleetBroadcaster
{
    method Initialize()
    {
        LavishScript:RegisterEvent[Fleet_Warp]
        Event[Fleet_Warp]:AttachAtom[This:OnFleetWarp]
    }

    ; Master broadcasts warp command
    method CommandWarp(int64 destinationID)
    {
        echo "FC: Fleet warp to ${Entity[${destinationID}].Name}"
        relay all -event Fleet_Warp ${destinationID}
    }

    ; All members (including master) receive
    atom OnFleetWarp(int64 destinationID)
    {
        if !${IsMaster}
        {
            echo "Following fleet warp"
            Entity[${destinationID}]:WarpTo[0]
        }
    }
}
```

### Pattern 2: Request/Response

**Use Case:** Query information from fleet members

```lavishscript
objectdef obj_FleetQuery
{
    variable int ResponseCount = 0

    method Initialize()
    {
        ; Register both request and response events
        LavishScript:RegisterEvent[Fleet_Shield_Request]
        LavishScript:RegisterEvent[Fleet_Shield_Response]

        Event[Fleet_Shield_Request]:AttachAtom[This:OnShieldRequest]
        Event[Fleet_Shield_Response]:AttachAtom[This:OnShieldResponse]
    }

    ; Master: Request shield status from all
    method RequestShieldStatus()
    {
        This.ResponseCount:Set[0]
        relay "all other" -event Fleet_Shield_Request
    }

    ; Slave: Respond with shield status
    atom OnShieldRequest()
    {
        variable float shieldPct = ${MyShip.ShieldPct}
        relay "${MasterName}" -event Fleet_Shield_Response "${Me.Name}" ${shieldPct}
    }

    ; Master: Collect responses
    atom OnShieldResponse(string charName, float shieldPct)
    {
        This.ResponseCount:Inc
        echo "${charName}: Shield at ${shieldPct.Precision[1]}%"

        if ${shieldPct} < 50
        {
            echo "WARNING: ${charName} low shields!"
        }
    }
}
```

### Pattern 3: State Synchronization

**Use Case:** Keep fleet members in sync

The fleet state synchronization pattern (master broadcasts state via relay, all members receive and switch behavior) is implemented as `obj_FleetStatus` under [Status Synchronization](#status-synchronization) in the Character Coordination section. That version includes `AllMembersStatus` collection tracking — a superset of the simple `obj_FleetSync` shown previously.

---

## UplinkManager System

### Overview

EVEBot's **UplinkManager** is a sophisticated automatic peer discovery and coordination system:

**Key Features:**
- **Automatic Registration:** Sessions auto-discover each other
- **Heartbeat System:** Sessions ping every 30 seconds, pruned if silent for 35 seconds
- **Dynamic Variables:** Insert custom data into peer objects at runtime
- **Skill Sharing:** Broadcast skill levels for fleet role assignment

### Architecture

```lavishscript
; Registered session object
objectdef obj_RegisteredSession
{
    variable string SessionName        ; InnerSpace session name
    variable int CharID               ; Character ID
    variable string CharName          ; Character name
    variable string Behavior          ; Current bot behavior
    variable time LastPing            ; Last heartbeat timestamp

    ; Fleet command skills (auto-populated)
    variable int SkillLevel_Leadership = -1
    variable int SkillLevel_Wing_Command
    variable int SkillLevel_Fleet_Command
    variable int SkillLevel_Mining_Foreman

    ; Dynamic variables inserted at runtime via RelayInfo
}

; Main uplink manager
objectdef obj_UplinkManager
{
    variable int MaxHeartBeat = 35    ; Seconds before session pruned
    variable index:obj_RegisteredSession RegisteredSessions

    ; Pulse every 0.5-1.0 seconds
    method Pulse()
    {
        if !${Initialized}
        {
            ; First pulse: register and send skills
            This:RelayRegistration["all"]
            This:RelayMySkills["all"]
            Initialized:Set[TRUE]
        }
        else
        {
            ; Subsequent pulses: heartbeat and prune
            This:RelayRegistration["all", TRUE]
            This:PrunePeerSessions
        }
    }
}
```

### Registration and Heartbeat

```lavishscript
; Step 1: Broadcast registration to all peers
method RelayRegistration(string Destination, bool Update=FALSE)
{
    relay "${Destination}" -noredirect \
        "UplinkManager:UpdatePeerSession[${Session},${Me.CharID},${Me.Name},${Config.Common.CurrentBehavior}]"
}

; Step 2: Each peer receives and stores registration
method UpdatePeerSession(string RemoteSessionName, int CharID, string CharName, string Behavior)
{
    variable iterator RegisteredSession
    This.RegisteredSessions:GetIterator[RegisteredSession]

    if ${RegisteredSession:First(exists)}
    {
        do
        {
            ; If session exists, just update timestamp (heartbeat)
            if ${RegisteredSession.Value.SessionName.Equal[${RemoteSessionName}]}
            {
                RegisteredSession.Value.Behavior:Set[${Behavior}]
                RegisteredSession.Value.LastPing:Set[${Time.Timestamp}]
                return
            }
        }
        while ${RegisteredSession:Next(exists)}
    }

    ; New session - add to index
    This.RegisteredSessions:Insert[${RemoteSessionName}, ${CharID}, ${CharName}, ${Behavior}, ${Time.Timestamp}]

    ; Request skills from new session
    relay "${RemoteSessionName}" -noredirect "UplinkManager:RequestSkills[${Session}]"
    Logger:Log["Registered ${RemoteSessionName}:${Behavior}"]
}

; Step 3: Prune dead sessions
method PrunePeerSessions()
{
    variable iterator RegisteredSession
    This.RegisteredSessions:GetIterator[RegisteredSession]

    if ${RegisteredSession:First(exists)}
    {
        do
        {
            ; Check if heartbeat expired
            if ${Math.Calc[${Time.Timestamp} - ${RegisteredSession.Value.LastPing.Timestamp}].Int} > ${This.MaxHeartBeat}
            {
                Logger:Log["Removed ${RegisteredSession.Value.SessionName} (timeout)"]
                This.RegisteredSessions:Remove[${RegisteredSession.Key}]
                This.RegisteredSessions:Collapse
                return
            }
        }
        while ${RegisteredSession:Next(exists)}
    }
}
```

### Skill Sharing

```lavishscript
; Request skills from peer
method RequestSkills(string Requester)
{
    This:RelayMySkills[${Requester}]
}

; Broadcast skills to destination
method RelayMySkills(string Destination="all")
{
    relay "${Destination}" -noredirect \
        "UplinkManager:UpdatePeerSkills[${Session}, \
            ${If[${Me.Skill[Leadership](exists)}, ${Me.Skill[Leadership].Level}, 0]}, \
            ${If[${Me.Skill["Wing Command"](exists)}, ${Me.Skill["Wing Command"].Level}, 0]}, \
            ${If[${Me.Skill["Fleet Command"](exists)}, ${Me.Skill["Fleet Command"].Level}, 0]}, \
            ${If[${Me.Skill["Mining Foreman"](exists)}, ${Me.Skill["Mining Foreman"].Level}, 0]}, \
            ${If[${Me.Skill["Armored Warfare"](exists)}, ${Me.Skill["Armored Warfare"].Level}, 0]}]"
}

; Receive and store peer skills
method UpdatePeerSkills(string RemoteSessionName, int Leadership, int Wing_Command, int Fleet_Command,
                        int Mining_Foreman, int Armored_Warfare)
{
    variable iterator RegisteredSession
    This.RegisteredSessions:GetIterator[RegisteredSession]

    if ${RegisteredSession:First(exists)}
    {
        do
        {
            if ${RegisteredSession.Value.SessionName.Equal[${RemoteSessionName}]}
            {
                ; Store all skill levels
                RegisteredSession.Value.SkillLevel_Leadership:Set[${Leadership}]
                RegisteredSession.Value.SkillLevel_Wing_Command:Set[${Wing_Command}]
                RegisteredSession.Value.SkillLevel_Fleet_Command:Set[${Fleet_Command}]
                RegisteredSession.Value.SkillLevel_Mining_Foreman:Set[${Mining_Foreman}]
                RegisteredSession.Value.SkillLevel_Armored_Warfare:Set[${Armored_Warfare}]
                RegisteredSession.Value.LastPing:Set[${Time.Timestamp}]
                return
            }
        }
        while ${RegisteredSession:Next(exists)}
    }
}
```

### Dynamic Variable Insertion

**Powerful Feature:** Insert custom variables into peer objects at runtime!

```lavishscript
; Sender: Broadcast custom info
method RelayInfo(string VarName, string VarType, string Value)
{
    relay "all other" -noredirect \
        "UplinkManager:UpdateInfo[${Session}, ${VarName}, ${VarType}, ${Value}]"
}

; Receiver: Create or update variable dynamically
method UpdateInfo(string RemoteSessionName, string VarName, string VarType, string Value)
{
    variable iterator RegisteredSession
    This.RegisteredSessions:GetIterator[RegisteredSession]

    if ${RegisteredSession:First(exists)}
    {
        do
        {
            if ${RegisteredSession.Value.SessionName.Equal[${RemoteSessionName}]}
            {
                ; Check if variable exists
                if ${RegisteredSession.Value.${VarName}(exists)}
                {
                    ; Update existing
                    RegisteredSession.Value.${VarName}:Set[${Value}]
                }
                else
                {
                    ; CREATE NEW VARIABLE AT RUNTIME!
                    RegisteredSession.Value.VariableScope:CreateVariable[${VarType}, "${VarName}", "${Value}"]
                }

                RegisteredSession.Value.LastPing:Set[${Time.Timestamp}]
                Logger:Log["${RegisteredSession.Value.CharName}: ${VarName}=${Value}"]
                return
            }
        }
        while ${RegisteredSession:Next(exists)}
    }
}
```

### Using UplinkManager

```lavishscript
; Example: Orca broadcasts need for hauler
method RequestHauler()
{
    ; Broadcast custom variable to all peers
    UplinkManager:RelayInfo["NeedHauler", "bool", "TRUE"]
    UplinkManager:RelayInfo["OrcaLocation", "string", "${Me.ToEntity.ID}"]
}

; Example: Hauler checks for Orca needing service
method CheckForOrca()
{
    variable iterator RegisteredSession
    UplinkManager.RegisteredSessions:GetIterator[RegisteredSession]

    if ${RegisteredSession:First(exists)}
    {
        do
        {
            ; Check if peer has NeedHauler variable (dynamically inserted!)
            variable string VarName = "NeedHauler"
            if ${RegisteredSession.Value.${VarName}(exists)}
            {
                if ${RegisteredSession.Value.${VarName}}
                {
                    echo "Orca ${RegisteredSession.Value.CharName} needs hauler!"
                    variable string LocVar = "OrcaLocation"
                    if ${RegisteredSession.Value.${LocVar}(exists)}
                    {
                        This:WarpToOrca[${RegisteredSession.Value.${LocVar}}]
                    }
                }
            }
        }
        while ${RegisteredSession:Next(exists)}
    }
}
```

---

## Fleet Coordination Patterns

### HARDSTOP Pattern (Emergency Dock)

**Used by:** EVEBot Orca, Guardian, Miner

**Purpose:** Broadcast critical danger, all fleet members dock immediately

```lavishscript
objectdef obj_FleetSafety
{
    method Initialize()
    {
        LavishScript:RegisterEvent[EVEBot_HARDSTOP]
        Event[EVEBot_HARDSTOP]:AttachAtom[This:OnHardStop]
    }

    ; Any fleet member can trigger HARDSTOP
    method TriggerHardStop(string reason)
    {
        Logger:Log["HARD STOP: ${reason}", LOG_CRITICAL]

        ; Set local flag
        EVEBot.ReturnToStation:Set[TRUE]

        ; Notify entire fleet
        relay all -event EVEBot_HARDSTOP "${Me.Name} - ${Config.Common.CurrentBehavior} (${reason})"
    }

    ; All members receive and react
    atom OnHardStop(string message)
    {
        echo "=== HARD STOP RECEIVED ==="
        echo "Reason: ${message}"
        echo "=========================="

        ; Set local flag to dock
        EVEBot.ReturnToStation:Set[TRUE]

        ; State machine will handle actual docking
    }

    ; Example triggers from different behaviors
    method CheckHostiles()
    {
        if ${Social.PossibleHostiles}
        {
            This:TriggerHardStop["Hostiles"]
        }
    }

    method CheckPod()
    {
        if ${Ship.IsPod}
        {
            This:TriggerHardStop["InPod"]
        }
    }

    method CheckCargoStuck()
    {
        if ${CargoNotChanging}
        {
            This:TriggerHardStop["Cargo Stuck"]
        }
    }
}
```

**Real Example from obj_Orca.iss:**

```lavishscript
; Orca detects hostiles
if ${Social.PossibleHostiles}
{
    This.CurrentState:Set["HARDSTOP"]
    Logger:Log["HARD STOP: Possible hostiles, notifying fleet"]
    relay all -event EVEBot_HARDSTOP "${Me.Name} - ${Config.Common.CurrentBehavior} (Hostiles)"
    EVEBot.ReturnToStation:Set[TRUE]
    return
}

; Guardian detects pod
if ${Ship.IsPod}
{
    This.CurrentState:Set["HARDSTOP"]
    Logger:Log["HARD STOP: Ship in a pod, notifying fleet that I failed them"]
    relay all -event EVEBot_HARDSTOP "${Me.Name} - ${Config.Common.CurrentBehavior} (InPod)"
    EVEBot.ReturnToStation:Set[TRUE]
    return
}
```

### Master/Slave Coordination

**Pattern 1: Master Election (EVEBot)**

(For movement-based following once the master is known, see [Pattern 3: Leader-Follower](#pattern-3-leader-follower) in Master-Slave Architectures above.)

```lavishscript
objectdef obj_MasterElection
{
    variable string MasterName
    variable bool IsMaster = FALSE
    variable obj_PulseTimer LastMasterQuery

    method Initialize()
    {
        LavishScript:RegisterEvent[EVEBot_Master_Query]
        LavishScript:RegisterEvent[EVEBot_Master_Notify]

        Event[EVEBot_Master_Query]:AttachAtom[This:Event_Master_Query]
        Event[EVEBot_Master_Notify]:AttachAtom[This:Event_Master_Notify]

        This.LastMasterQuery:SetIntervals[5.0, 5.0]
    }

    method Pulse()
    {
        ; If in group mode and no master known, query for master
        if ${Config.Miner.GroupMode}
        {
            if ${This.MasterName.Length} == 0 && ${This.LastMasterQuery.Ready}
            {
                This.LastMasterQuery:Update
                relay all -event EVEBot_Master_Query
            }
        }
    }

    ; Someone is looking for master
    atom Event_Master_Query()
    {
        if ${Config.Miner.MasterMode}
        {
            ; I am master - notify requester
            relay all -event EVEBot_Master_Notify "${Me.Name}"
        }
    }

    ; Master announced themselves
    atom Event_Master_Notify(string masterName)
    {
        if ${masterName.Length} > 0
        {
            This.MasterName:Set[${masterName}]
            This.IsMaster:Set[FALSE]
            echo "Master is: ${masterName}"
        }
        else
        {
            ; Master shutdown
            This.MasterName:Set[""]
            echo "Master disconnected"
        }
    }

    ; On shutdown, if I'm master, notify slaves
    method Shutdown()
    {
        if ${Config.Miner.MasterMode}
        {
            relay all -event EVEBot_Master_Notify ""
        }
    }
}
```

**Pattern 2: Target Sharing (Yamfa)**

Yamfa's master-slave target-sharing pattern broadcasts the master's locked-target list and current primary to all slaves each pulse, slaves lock the relayed IDs and engage the relayed primary. The full production-grade implementation of this pattern — with master/slave role detection, primary/secondary broadcasting, slave-side `OnPrimaryTarget` atom, lock-and-fire logic, and complete Initialize/Shutdown lifecycle — is provided as the canonical `obj_FleetCombat` object in [Example 1: Complete Fleet Combat Coordination](#example-1-complete-fleet-combat-coordination) later in this guide. Prefer that single authoritative version over re-implementing the Yamfa relay loop directly.

### Orca Service Pattern

**Scenario:** Miners request Orca services (survey scan, shield boosts)

```lavishscript
objectdef obj_OrcaService
{
    method Initialize()
    {
        LavishScript:RegisterEvent[Orca_Survey_Request]
        LavishScript:RegisterEvent[Orca_Shield_Request]

        Event[Orca_Survey_Request]:AttachAtom[This:OnSurveyRequest]
        Event[Orca_Shield_Request]:AttachAtom[This:OnShieldRequest]
    }

    ; MINER: Request survey scan
    method RequestSurveyScan(int64 asteroidID)
    {
        relay "${OrcaName}" -event Orca_Survey_Request ${Me.CharID} ${asteroidID}
    }

    ; ORCA: Handle survey request
    atom OnSurveyRequest(int requesterID, int64 asteroidID)
    {
        if !${Config.Common.CurrentBehavior.Equal["Orca"]}
            return

        echo "Survey request from ${requesterID}"

        ; Lock and survey asteroid
        if ${Entity[${asteroidID}](exists)}
        {
            if !${Entity[${asteroidID}].IsLockedTarget}
            {
                Entity[${asteroidID}]:LockTarget
                wait 30 ${Entity[${asteroidID}].IsLockedTarget}
            }

            if ${Entity[${asteroidID}].IsLockedTarget}
            {
                Ship:Activate_SurveyScanners
            }
        }
    }

    ; MINER: Request shield boost
    method RequestShieldBoost()
    {
        if ${MyShip.ShieldPct} < 50
        {
            relay "${OrcaName}" -event Orca_Shield_Request ${Me.ToEntity.ID} ${MyShip.ShieldPct}
        }
    }

    ; ORCA: Provide shield boost
    atom OnShieldRequest(int64 shipID, float shieldPct)
    {
        if !${Config.Common.CurrentBehavior.Equal["Orca"]}
            return

        echo "Shield boost request: ${Entity[${shipID}].Name} at ${shieldPct.Precision[1]}%"

        ; Lock ship if needed
        if ${Entity[${shipID}](exists)} && !${Entity[${shipID}].IsLockedTarget}
        {
            Entity[${shipID}]:LockTarget
        }

        ; Activate shield transfer
        variable iterator Module
        MyShip.Modules:GetIterator[Module]

        if ${Module:First(exists)}
        {
            do
            {
                if ${Module.Value.ToItem.Group.Equal["Shield Transporter"]}
                {
                    Module.Value:Click
                    break
                }
            }
            while ${Module:Next(exists)}
        }
    }
}
```

---

## IRC Bridge Integration

> **📚 Complete ISXIM Documentation:** See `__CRITICAL_NEWEST_ISXIM_Reference.md` for full IRC extension reference including all TLOs, datatypes, events, and usage patterns.

### Overview (Tehbot ChatRelay)

[Tehbot](https://github.com/isxGames/Tehbot)'s **ChatRelay** bridges EVE sessions with IRC for:
- Fleet coordination via IRC channel
- Remote monitoring and control
- Multi-computer coordination (across network via IRC server)
- Persistent chat logs
- Remote bot command & control
- Status reporting to IRC channel

### Architecture

```lavishscript
objectdef obj_ChatRelay
{
    variable bool IsConnected = FALSE
    variable queue:string Buffer        ; Message queue for throttling
    variable bool Throttle = FALSE      ; Reconnect throttle

    method Initialize()
    {
        ; Register IRC events from ISXIM extension
        Event[IRC_ReceivedChannelMsg]:AttachAtom[This:IRC_ReceivedChannelMsg]
        Event[IRC_ReceivedPrivateMsg]:AttachAtom[This:IRC_ReceivedPrivateMsg]
        Event[IRC_KickedFromChannel]:AttachAtom[This:IRC_KickedFromChannel]
    }

    method Start()
    {
        if ${Config.UseIRC}
        {
            ; Load ISXIM extension
            if !${Extension[ISXIM]}
            {
                ext -require ISXIM
            }

            This:Connect
        }
    }
}
```

### IRC Connection

```lavishscript
function Connect()
{
    Logger:Log["Connecting to IRC"]

    if ${Throttle}
    {
        ; Disconnect if throttled (reconnecting too fast)
        IRCUser[${Config.IRCUsername}]:Disconnect
    }
    Throttle:Set[FALSE]

    ; Connect to IRC server
    IRC:Connect[${Config.IRCServer}, ${Config.IRCUsername}, ${Config.IRCPort}, ${Config.IRCPassword}]
}

; Main pulse
member:bool ChatRelay()
{
    if !${Config.UseIRC}
        return FALSE

    ; Connect if needed
    if ${IRC.NumUsers} < 1
    {
        call This.Connect
    }

    ; Wait for connection
    if ${IRCUser[${Config.IRCUsername}].IsConnecting}
    {
        return FALSE
    }

    ; Join channel if connected
    if ${IRCUser[${Config.IRCUsername}](exists)} && ${IRCUser[${Config.IRCUsername}].IsConnected}
    {
        if ${IRCUser[${Config.IRCUsername}].NumChannelsIn} < 1
        {
            IRCUser[${Config.IRCUsername}]:Join[${Config.IRCChannel}]
        }

        ; Send queued messages (one per pulse for throttling)
        if ${This.Buffer.Peek(exists)}
        {
            This:SendMessage["${This.Buffer.Peek}"]
            This.Buffer:Dequeue
        }
    }

    return FALSE
}
```

### Message Handling

```lavishscript
; Receive channel message from IRC
method IRC_ReceivedChannelMsg(string User, string Channel, string From, string Message)
{
    echo "[${User} - ${Channel}] -- (${From}) ${Message}"

    ; Parse for commands
    if ${Message.Left[1].Equal["!"]}
    {
        This:ProcessCommand["${Message}", "${From}"]
    }
}

; Process IRC commands
method ProcessCommand(string message, string sender)
{
    variable string command = ${message.Token[1, " "]}

    switch ${command}
    {
        case !status
            This:ReportStatus
            break

        case !dock
            EVEBot.ReturnToStation:Set[TRUE]
            This:Say["${Me.Name}: Docking..."]
            break

        case !pause
            EVEBot:Pause["IRC Command"]
            This:Say["${Me.Name}: Paused"]
            break

        case !resume
            EVEBot:Resume["IRC Command"]
            This:Say["${Me.Name}: Resumed"]
            break
    }
}

; Send message to IRC (queued)
method QueueMessage(string msg)
{
    This.Buffer:Queue["${msg}"]
}

; Actual send (throttled via queue)
method SendMessage(string msg)
{
    if ${IRCUser[${Config.IRCUsername}].IsConnected}
    {
        IRCUser[${Config.IRCUsername}].Channel[${Config.IRCChannel}]:Say["${msg}"]
    }
}

; Public method for scripts to broadcast to IRC
function Say(string msg)
{
    This:QueueMessage["${msg}"]

    if !${IRCUser[${Config.IRCUsername}].IsConnected}
    {
        call This.Connect
    }
}
```

### Nickserv Authentication

```lavishscript
method IRC_ReceivedNotice(string User, string From, string To, string Message)
{
    ; Handle Nickserv authentication
    if ${From.Equal[Nickserv]}
    {
        if ${Message.Find[This nickname is registered and protected]}
        {
            ; Send password to Nickserv
            if ${To.Equal[${Config.IRCUsername}]}
            {
                IRCUser[${Config.IRCUsername}]:PM[Nickserv, "identify ${Config.IRCPassword}"]
            }
            return
        }
        elseif ${Message.Find[Password accepted]}
        {
            echo "[${To}] Identify with Nickserv successful"
            return
        }
        elseif ${Message.Find[Password incorrect]}
        {
            echo "Incorrect password while attempting to identify ${To} with Nickserv"
            return
        }
    }
}
```

### Integration Example

```lavishscript
; Report fleet status to IRC
method ReportFleetStatus()
{
    variable string status = ""

    ; Build status report
    status:Concat["${Me.Name}: "]
    status:Concat["State=${CurrentState} "]
    status:Concat["Shield=${MyShip.ShieldPct.Precision[0]}% "]
    status:Concat["Cap=${MyShip.CapacitorPct.Precision[0]}% "]
    status:Concat["Cargo=${Ship.CargoFull.Precision[0]}%"]

    ; Send to IRC
    ChatRelay:Say["${status}"]
}

; Emergency broadcast
method EmergencyBroadcast(string reason)
{
    ChatRelay:Say["!!! EMERGENCY: ${Me.Name} - ${reason} !!!"]

    ; Also relay to other EVE sessions
    relay all -event EVEBot_HARDSTOP "${reason}"
}
```

---

## Uplink Networking

### Multi-Computer Coordination

**Uplink** extends relay across network - communicate between different physical computers!

### Uplink Architecture

```
Computer 1 (Mining Rig)          Computer 2 (Combat Rig)
┌──────────────────┐            ┌──────────────────┐
│  Miner 1         │            │  Combat 1        │
│  Miner 2         │◄──Uplink──►│  Combat 2        │
│  Hauler          │            │  Logi            │
└──────────────────┘            └──────────────────┘
```

### Uplink Setup

```lavishscript
; In InnerSpace console on Computer 1:
uplink create MyFleet
uplink MyFleet connect 192.168.1.100:54321

; In InnerSpace console on Computer 2:
uplink create MyFleet
uplink MyFleet listen 54321
```

### Uplink Relay Syntax

```lavishscript
; Relay across uplink
uplink relay all echo "Message to all computers"
uplink relay "CharName@Computer2" echo "Specific char on remote computer"

; Uplink event broadcast
uplink relay all -event Fleet_Warp ${destinationID}
```

### Uplink in Code

```lavishscript
objectdef obj_UplinkCoordination
{
    method Initialize()
    {
        ; Same event registration - uplink relay works automatically!
        LavishScript:RegisterEvent[Fleet_Primary_Target]
        Event[Fleet_Primary_Target]:AttachAtom[This:OnPrimaryTarget]
    }

    ; FC on Computer 1 broadcasts to all computers
    method CallPrimary(int64 targetID)
    {
        ; Local relay (same computer)
        relay all -event Fleet_Primary_Target ${targetID}

        ; Uplink relay (all computers)
        uplink relay all -event Fleet_Primary_Target ${targetID}
    }

    ; DPS ships on all computers receive
    atom OnPrimaryTarget(int64 targetID)
    {
        echo "Primary target: ${Entity[${targetID}].Name}"
        Entity[${targetID}]:LockTarget
    }
}
```

### Uplink UpdateClient (EVEBot)

**Advanced:** Uplink can call methods on remote computer's running script

```lavishscript
; From obj_Callback.iss - broadcasts ship status across uplink
method Pulse()
{
    if ${Config.Common.Callback} && ${EVEBot.SessionValid}
    {
        ; This updates a monitoring application on another computer!
        uplink UpdateClient \
            "${Me.Name}" \
            "${MyShip.ShieldPct}" \
            "${MyShip.ArmorPct}" \
            "${MyShip.CapacitorPct}" \
            "${Defense.Hide}" \
            "${Defense.HideReason}" \
            "${Me.ActiveTarget.Name}" \
            "${EVEBot.Paused}" \
            "${Config.Common.Behavior}" \
            "${MyShip}" \
            "${Session}"
    }
}
```

### Network Topology Patterns

**Pattern 1: Star (Hub and Spoke)**

```
        Computer 1 (Hub - FC)
              /  |  \
             /   |   \
    Comp 2   Comp 3   Comp 4
    (DPS)    (DPS)    (Logi)
```

```lavishscript
; Computer 1 (Hub) setup
uplink create FleetHub
uplink FleetHub listen 54321

; Computers 2-4 (Spokes) setup
uplink create FleetHub
uplink FleetHub connect 192.168.1.100:54321
```

**Pattern 2: Mesh (Full Connectivity)**

```
    Comp 1 ─────── Comp 2
      │  \       /  │
      │   \     /   │
      │    Comp 3   │
      │       │     │
    Comp 4 ──────── Comp 5
```

```lavishscript
; Each computer connects to all others
uplink create FleetMesh
uplink FleetMesh listen 54321
uplink FleetMesh connect 192.168.1.101:54322
uplink FleetMesh connect 192.168.1.102:54323
; ... etc
```

---

## Performance Considerations

### Relay Throttling

**Problem:** Broadcasting too frequently causes lag

**Solution 1: Pulse Timer**

```lavishscript
objectdef obj_ThrottledRelay
{
    variable obj_PulseTimer RelayTimer

    method Initialize()
    {
        ; Relay max once per 0.5-1.0 seconds
        This.RelayTimer:SetIntervals[0.5, 1.0]
    }

    method Pulse()
    {
        if ${This.RelayTimer.Ready}
        {
            This:BroadcastStatus
            This.RelayTimer:Update
        }
    }

    method BroadcastStatus()
    {
        relay "all other" -event Fleet_Status "${Me.Name}" ${MyShip.ShieldPct}
    }
}
```

**Solution 2: Change Detection**

```lavishscript
objectdef obj_SmartRelay
{
    variable int64 LastPrimaryTarget = 0

    method CheckPrimaryChange()
    {
        ; Only relay if primary changed
        if ${Me.ActiveTarget.ID} != ${This.LastPrimaryTarget}
        {
            This.LastPrimaryTarget:Set[${Me.ActiveTarget.ID}]

            ; Broadcast change
            relay all -event Fleet_Primary_Changed ${Me.ActiveTarget.ID}
        }
    }
}
```

**Solution 3: Message Queuing**

```lavishscript
objectdef obj_QueuedRelay
{
    variable queue:string MessageQueue
    variable obj_PulseTimer SendTimer

    method Initialize()
    {
        ; Send one message per 0.1 seconds
        This.SendTimer:SetIntervals[0.1, 0.1]
    }

    ; Queue message for sending
    method QueueRelay(string destination, string message)
    {
        This.MessageQueue:Queue["${destination}|${message}"]
    }

    ; Process queue
    method Pulse()
    {
        if ${This.SendTimer.Ready} && ${This.MessageQueue.Peek(exists)}
        {
            variable string msg = "${This.MessageQueue.Peek}"
            variable string dest = ${msg.Token[1, "|"]}
            variable string content = ${msg.Token[2, "|"]}

            relay "${dest}" ${content}

            This.MessageQueue:Dequeue
            This.SendTimer:Update
        }
    }
}
```

### Event Loop Integration

**Best Practice:** Attach to EVENT_ONFRAME for automatic pulsing

```lavishscript
objectdef obj_RelayManager
{
    method Initialize()
    {
        ; Pulse with EVEBot frame event
        Event[EVENT_ONFRAME]:AttachAtom[This:Pulse]

        ; Set intervals
        This.PulseTimer:SetIntervals[1.0, 2.0]
    }

    method Pulse()
    {
        ; Early exit if not ready
        if !${This.PulseTimer.Ready}
            return

        ; Do relay work
        This:ProcessRelayQueue
        This:BroadcastStatus

        ; Update timer
        This.PulseTimer:Update
    }

    method Shutdown()
    {
        ; ALWAYS detach!
        Event[EVENT_ONFRAME]:DetachAtom[This:Pulse]
    }
}
```

### Bandwidth Considerations

**Problem:** Large data broadcasts cause network lag

**Solution: Compress data**

```lavishscript
; BAD - sends full entity objects
relay all -event Targets "${Entity[${ID1}]}" "${Entity[${ID2}]}" "${Entity[${ID3}]}"

; GOOD - sends only IDs
relay all -event Targets "${ID1},${ID2},${ID3}"

; Parse on receive
atom OnTargets(string targetIDs)
{
    variable int i = 1
    variable string idList = "${targetIDs}"

    while ${idList.Token[${i}, ","](exists)}
    {
        variable int64 targetID = ${idList.Token[${i}, ","]}
        This:ProcessTarget[${targetID}]
        i:Inc
    }
}
```

---

## Best Practices

### 1. Always Use -noredirect for Method Calls

```lavishscript
; WRONG - can create relay loops
relay all "MyObject:MyMethod"

; RIGHT - no relay loop
relay all -noredirect "MyObject:MyMethod"
```

### 2. Always Detach Events in Shutdown

```lavishscript
method Initialize()
{
    Event[MyEvent]:AttachAtom[This:OnMyEvent]
}

method Shutdown()
{
    ; CRITICAL - detach or memory leak!
    Event[MyEvent]:DetachAtom[This:OnMyEvent]
}
```

### 3. Validate Relay Data

```lavishscript
atom OnTargetRelay(int64 targetID)
{
    ; Validate before using
    if !${Entity[${targetID}](exists)}
    {
        echo "Invalid target ID relayed: ${targetID}"
        return
    }

    ; Safe to use
    Entity[${targetID}]:LockTarget
}
```

### 4. Use Heartbeat for Session Tracking

```lavishscript
objectdef obj_SessionTracker
{
    variable collection:time LastSeen
    variable int TimeoutSeconds = 30

    method Pulse()
    {
        This:BroadcastHeartbeat
        This:PruneDeadSessions
    }

    method BroadcastHeartbeat()
    {
        relay all -event Fleet_Heartbeat "${Me.Name}"
    }

    atom OnHeartbeat(string charName)
    {
        This.LastSeen:Set[${charName}, ${Time.Timestamp}]
    }

    method PruneDeadSessions()
    {
        variable iterator Session
        This.LastSeen:GetIterator[Session]

        if ${Session:First(exists)}
        {
            do
            {
                if ${Math.Calc[${Time.Timestamp} - ${Session.Value.Timestamp}]} > ${TimeoutSeconds}
                {
                    echo "Session ${Session.Key} timed out"
                    This.LastSeen:Erase[${Session.Key}]
                }
            }
            while ${Session:Next(exists)}
        }
    }
}
```

### 5. Version Compatibility

```lavishscript
objectdef obj_VersionedRelay
{
    variable string ProtocolVersion = "2.1"

    method Initialize()
    {
        LavishScript:RegisterEvent[Fleet_Versioned_Event]
        Event[Fleet_Versioned_Event]:AttachAtom[This:OnVersionedEvent]
    }

    method BroadcastWithVersion(string data)
    {
        relay all -event Fleet_Versioned_Event "${This.ProtocolVersion}" "${data}"
    }

    atom OnVersionedEvent(string version, string data)
    {
        ; Check version compatibility
        if !${version.Equal[${This.ProtocolVersion}]}
        {
            echo "WARNING: Received event from incompatible version ${version}"
            return
        }

        ; Process data
        This:ProcessData["${data}"]
    }
}
```

### 6. Error Handling

```lavishscript
atom OnFleetCommand(string command, string params)
{
    ; Validate command exists
    if !${This.ValidCommands.Contains[${command}]}
    {
        echo "Unknown command received: ${command}"
        return
    }

    ; Try-catch pattern using conditional execution
    if ${This.${command}(exists)}
    {
        call This.${command} "${params}"
    }
    else
    {
        echo "ERROR: Command method ${command} not found"
    }
}
```

### 7. Security - Don't Trust Remote Input

```lavishscript
atom OnRemoteBookmark(string bookmarkName)
{
    ; DANGER - arbitrary bookmark warp!
    EVE.ExecuteCommand["/warp ${bookmarkName}"]

    ; SAFER - validate bookmark exists and is safe
    if ${EVE.Bookmark[${bookmarkName}](exists)}
    {
        variable int solarSystemID = ${EVE.Bookmark[${bookmarkName}].SolarSystemID}

        ; Check security
        if ${solarSystemID} == ${Me.SolarSystemID}
        {
            call Ship.WarpToBookMarkName "${bookmarkName}"
        }
        else
        {
            echo "Rejected cross-system bookmark warp"
        }
    }
}
```

### 8. Master Failover

```lavishscript
objectdef obj_MasterFailover
{
    variable string MasterName
    variable time LastMasterPing
    variable int MasterTimeout = 10

    method CheckMasterAlive()
    {
        if ${Math.Calc[${Time.Timestamp} - ${This.LastMasterPing.Timestamp}]} > ${MasterTimeout}
        {
            echo "Master ${This.MasterName} timed out!"
            This:ElectNewMaster
        }
    }

    method ElectNewMaster()
    {
        ; Simple election: lowest CharID becomes master
        variable int lowestID = ${Me.CharID}
        variable string newMaster = "${Me.Name}"

        variable iterator Peer
        UplinkManager.RegisteredSessions:GetIterator[Peer]

        if ${Peer:First(exists)}
        {
            do
            {
                if ${Peer.Value.CharID} < ${lowestID}
                {
                    lowestID:Set[${Peer.Value.CharID}]
                    newMaster:Set["${Peer.Value.CharName}"]
                }
            }
            while ${Peer:Next(exists)}
        }

        if ${newMaster.Equal[${Me.Name}]}
        {
            echo "I am the new master!"
            This:BecomeMaster
        }
        else
        {
            echo "New master: ${newMaster}"
            This.MasterName:Set[${newMaster}]
        }
    }
}
```

---

## Complete Working Examples

### Example 1: Complete Fleet Combat Coordination

```lavishscript
objectdef obj_FleetCombat
{
    ; Master variables
    variable int64 PrimaryTarget = 0
    variable int64 SecondaryTarget = 0
    variable bool IsMaster = FALSE

    ; Slave variables
    variable string MasterName

    method Initialize()
    {
        ; Register all fleet events
        LavishScript:RegisterEvent[FC_Primary_Target]
        LavishScript:RegisterEvent[FC_Secondary_Target]
        LavishScript:RegisterEvent[FC_Fleet_Warp]
        LavishScript:RegisterEvent[FC_Anchor_Request]
        LavishScript:RegisterEvent[FC_EWAR_Assignment]

        ; Attach handlers
        Event[FC_Primary_Target]:AttachAtom[This:OnPrimaryTarget]
        Event[FC_Secondary_Target]:AttachAtom[This:OnSecondaryTarget]
        Event[FC_Fleet_Warp]:AttachAtom[This:OnFleetWarp]
        Event[FC_Anchor_Request]:AttachAtom[This:OnAnchorRequest]
        Event[FC_EWAR_Assignment]:AttachAtom[This:OnEWARAssignment]

        ; Check if this character is FC
        This.IsMaster:Set[${Config.Fleet.IsFC}]
        This.MasterName:Set["${Config.Fleet.FCName}"]
    }

    method Shutdown()
    {
        Event[FC_Primary_Target]:DetachAtom[This:OnPrimaryTarget]
        Event[FC_Secondary_Target]:DetachAtom[This:OnSecondaryTarget]
        Event[FC_Fleet_Warp]:DetachAtom[This:OnFleetWarp]
        Event[FC_Anchor_Request]:DetachAtom[This:OnAnchorRequest]
        Event[FC_EWAR_Assignment]:DetachAtom[This:OnEWARAssignment]
    }

    ; ===== MASTER METHODS =====

    method CallPrimary(int64 targetID)
    {
        if !${This.IsMaster}
            return

        This.PrimaryTarget:Set[${targetID}]
        echo "FC: PRIMARY -> ${Entity[${targetID}].Name}"

        relay all -event FC_Primary_Target ${targetID} "${Entity[${targetID}].Name}"
    }

    method CallSecondary(int64 targetID)
    {
        if !${This.IsMaster}
            return

        This.SecondaryTarget:Set[${targetID}]
        echo "FC: SECONDARY -> ${Entity[${targetID}].Name}"

        relay all -event FC_Secondary_Target ${targetID} "${Entity[${targetID}].Name}"
    }

    method CommandFleetWarp(int64 destinationID)
    {
        if !${This.IsMaster}
            return

        echo "FC: FLEET WARP -> ${Entity[${destinationID}].Name}"

        ; FC warps immediately
        Entity[${destinationID}]:WarpTo[0]

        ; Fleet follows
        relay "all other" -event FC_Fleet_Warp ${destinationID}
    }

    method AssignEWAR(string charName, string ewarType, int64 targetID)
    {
        if !${This.IsMaster}
            return

        echo "FC: ${charName} -> ${ewarType} on ${Entity[${targetID}].Name}"

        relay "${charName}" -event FC_EWAR_Assignment "${ewarType}" ${targetID}
    }

    ; ===== SLAVE HANDLERS =====

    atom OnPrimaryTarget(int64 targetID, string targetName)
    {
        if ${This.IsMaster}
            return

        echo ">>> PRIMARY: ${targetName}"
        This.PrimaryTarget:Set[${targetID}]

        ; DPS ships lock and shoot
        if ${Config.Fleet.Role.Equal["DPS"]}
        {
            if ${Entity[${targetID}](exists)}
            {
                Entity[${targetID}]:LockTarget
                wait 30 ${Entity[${targetID}].IsLockedTarget}

                if ${Entity[${targetID}].IsLockedTarget}
                {
                    Ship:Activate_Weapons
                    Drones:EngageTarget[${targetID}]
                }
            }
        }
    }

    atom OnSecondaryTarget(int64 targetID, string targetName)
    {
        if ${This.IsMaster}
            return

        echo ">>> SECONDARY: ${targetName}"
        This.SecondaryTarget:Set[${targetID}]

        ; Pre-lock for faster switching
        if ${Config.Fleet.Role.Equal["DPS"]}
        {
            if ${Entity[${targetID}](exists)} && !${Entity[${targetID}].IsLockedTarget}
            {
                Entity[${targetID}]:LockTarget
            }
        }
    }

    atom OnFleetWarp(int64 destinationID)
    {
        if ${This.IsMaster}
            return

        echo ">>> FLEET WARPING to ${Entity[${destinationID}].Name}"

        ; Stop current action
        EVE:Execute[CmdStopShip]

        ; Warp to destination
        Entity[${destinationID}]:WarpTo[0]
    }

    atom OnAnchorRequest()
    {
        if ${This.IsMaster}
            return

        ; Anchor ships orbit FC at 2500m
        if ${Config.Fleet.Role.Equal["Logi"]} || ${Config.Fleet.Role.Equal["EWAR"]}
        {
            variable int64 fcID = ${Entity["Name = \"${This.MasterName}\""].ID}

            if ${fcID} > 0
            {
                echo ">>> Anchoring on FC"
                Entity[${fcID}]:Orbit[2500]
            }
        }
    }

    atom OnEWARAssignment(string ewarType, int64 targetID)
    {
        if ${This.IsMaster}
            return

        echo ">>> EWAR Assignment: ${ewarType} on ${Entity[${targetID}].Name}"

        ; Lock target
        if ${Entity[${targetID}](exists)} && !${Entity[${targetID}].IsLockedTarget}
        {
            Entity[${targetID}]:LockTarget
            wait 30 ${Entity[${targetID}].IsLockedTarget}
        }

        ; Activate EWAR module
        if ${Entity[${targetID}].IsLockedTarget}
        {
            switch ${ewarType}
            {
                case WEB
                    Ship:Activate_Webs
                    break
                case SCRAM
                    Ship:Activate_Scrams
                    break
                case JAM
                    Ship:Activate_Jammers
                    break
                case DAMP
                    Ship:Activate_Damps
                    break
                case PAINTER
                    Ship:Activate_Painters
                    break
            }
        }
    }

    ; ===== COMBAT LOGIC =====

    method ProcessCombat()
    {
        ; FC autonomous targeting
        if ${This.IsMaster}
        {
            This:MasterTargeting
        }
        ; Slaves follow FC targets
        else
        {
            This:SlaveTargeting
        }
    }

    method MasterTargeting()
    {
        ; Find best primary
        variable int64 bestTarget = ${This:GetBestTarget}

        if ${bestTarget} != ${This.PrimaryTarget}
        {
            This:CallPrimary[${bestTarget}]
        }

        ; Find secondary (next threat)
        variable int64 secondaryTarget = ${This:GetSecondBestTarget}

        if ${secondaryTarget} != ${This.SecondaryTarget}
        {
            This:CallSecondary[${secondaryTarget}]
        }
    }

    method SlaveTargeting()
    {
        ; Lock primary if not locked
        if ${This.PrimaryTarget} > 0
        {
            if ${Entity[${This.PrimaryTarget}](exists)} && !${Entity[${This.PrimaryTarget}].IsLockedTarget}
            {
                Entity[${This.PrimaryTarget}]:LockTarget
            }
        }

        ; Lock secondary if not locked
        if ${This.SecondaryTarget} > 0
        {
            if ${Entity[${This.SecondaryTarget}](exists)} && !${Entity[${This.SecondaryTarget}].IsLockedTarget}
            {
                Entity[${This.SecondaryTarget}]:LockTarget
            }
        }
    }

    member:int64 GetBestTarget()
    {
        ; Priority 1: Threats targeting me
        ; NOTE: ISXEVE only exposes `entity.IsTargetingMe` (is this entity
        ; targeting ME). There is no primitive for "is entity X targeting
        ; entity Y", so we cannot directly detect threats targeting the FC
        ; from a slave's perspective. The slave-side best we can do is
        ; prioritize entities targeting the slave itself, and rely on the
        ; FC relaying its own threat list separately if fleet-wide threat
        ; awareness is needed.
        variable index:entity Threats
        variable iterator Threat

        EVE:QueryEntities[Threats, "CategoryID = CATEGORYID_ENTITY && IsNPC"]
        Threats:GetIterator[Threat]

        if ${Threat:First(exists)}
        {
            do
            {
                if ${Threat.Value.IsTargetingMe}
                {
                    return ${Threat.Value.ID}
                }
            }
            while ${Threat:Next(exists)}
        }

        ; Priority 2: Closest
        variable int64 closestID = ${Entity["CategoryID = CATEGORYID_ENTITY && IsNPC"].ID}
        return ${closestID}
    }
}
```

### Example 2: Mining Fleet with Orca Support

```lavishscript
objectdef obj_MiningFleet
{
    variable bool IsOrca = FALSE
    variable string OrcaName
    variable bool OrcaInBelt = FALSE

    method Initialize()
    {
        This.IsOrca:Set[${MyShip.Group.Equal["Industrial Command Ship"]}]
        This.OrcaName:Set["${Config.Mining.OrcaName}"]

        ; Register events
        LavishScript:RegisterEvent[Orca_In_Belt]
        LavishScript:RegisterEvent[Orca_Left_Belt]
        LavishScript:RegisterEvent[Orca_Survey_Data]
        LavishScript:RegisterEvent[Miner_Request_Survey]
        LavishScript:RegisterEvent[Miner_Full_Cargo]

        ; Attach handlers
        Event[Orca_In_Belt]:AttachAtom[This:OnOrcaInBelt]
        Event[Orca_Left_Belt]:AttachAtom[This:OnOrcaLeftBelt]
        Event[Orca_Survey_Data]:AttachAtom[This:OnSurveyData]
        Event[Miner_Request_Survey]:AttachAtom[This:OnSurveyRequest]
        Event[Miner_Full_Cargo]:AttachAtom[This:OnMinerFullCargo]
    }

    ; ===== ORCA METHODS =====

    method ArriveInBelt()
    {
        if !${This.IsOrca}
            return

        echo "Orca: Arrived in belt, notifying miners"
        relay all -event Orca_In_Belt "${Me.ToEntity.ID}"
    }

    method LeaveBelt()
    {
        if !${This.IsOrca}
            return

        echo "Orca: Leaving belt, notifying miners"
        relay all -event Orca_Left_Belt
    }

    method BroadcastSurveyData(int64 asteroidID, int quantity)
    {
        if !${This.IsOrca}
            return

        ; Broadcast survey results to fleet
        relay all -event Orca_Survey_Data ${asteroidID} ${quantity} "${Entity[${asteroidID}].Name}"
    }

    atom OnSurveyRequest(string requester, int64 asteroidID)
    {
        if !${This.IsOrca}
            return

        echo "Orca: Survey request from ${requester}"

        ; Lock and survey asteroid
        if ${Entity[${asteroidID}](exists)}
        {
            if !${Entity[${asteroidID}].IsLockedTarget}
            {
                Entity[${asteroidID}]:LockTarget
                wait 30 ${Entity[${asteroidID}].IsLockedTarget}
            }

            if ${Entity[${asteroidID}].IsLockedTarget}
            {
                Ship:Activate_SurveyScanners
            }
        }
    }

    atom OnMinerFullCargo(string minerName, int64 minerID)
    {
        if !${This.IsOrca}
            return

        echo "Orca: ${minerName} has full cargo"

        ; Open fleet hangar for delivery
        if !${EVEWindow[ByName, "Fleet Hangar"](exists)}
        {
            Me.ToEntity:OpenHangar
        }
    }

    ; ===== MINER METHODS =====

    method RequestSurvey(int64 asteroidID)
    {
        if ${This.IsOrca}
            return

        relay "${This.OrcaName}" -event Miner_Request_Survey "${Me.Name}" ${asteroidID}
    }

    method NotifyFullCargo()
    {
        if ${This.IsOrca}
            return

        if ${This.OrcaInBelt}
        {
            echo "Miner: Cargo full, notifying Orca"
            relay "${This.OrcaName}" -event Miner_Full_Cargo "${Me.Name}" ${Me.ToEntity.ID}
        }
    }

    atom OnOrcaInBelt(int64 orcaID)
    {
        if ${This.IsOrca}
            return

        echo "Miner: Orca arrived in belt!"
        This.OrcaInBelt:Set[TRUE]

        ; Warp to Orca
        Entity[${orcaID}]:WarpTo[0]
        wait 50 !${Me.ToEntity.Mode} == 3

        ; Request survey on current target
        if ${Me.ActiveTarget.ID} > 0
        {
            This:RequestSurvey[${Me.ActiveTarget.ID}]
        }
    }

    atom OnOrcaLeftBelt()
    {
        if ${This.IsOrca}
            return

        echo "Miner: Orca left belt"
        This.OrcaInBelt:Set[FALSE]
    }

    atom OnSurveyData(int64 asteroidID, int quantity, string asteroidName)
    {
        if ${This.IsOrca}
            return

        echo "Miner: Survey - ${asteroidName} has ${quantity} units"

        ; Store survey data for asteroid selection logic
        AsteroidSurveyData:Set[${asteroidID}, ${quantity}]
    }

    ; ===== MINING LOGIC =====

    method DeliverToOrca()
    {
        variable int64 orcaID = ${Entity["Name = \"${This.OrcaName}\""].ID}

        if ${orcaID} == 0
        {
            echo "Orca not found!"
            return FALSE
        }

        ; Approach Orca
        if ${Entity[${orcaID}].Distance} > 2500
        {
            Entity[${orcaID}]:Approach[2500]
            wait 50 ${Entity[${orcaID}].Distance} <= 2500
        }

        ; Open Orca fleet hangar
        Entity[${orcaID}]:OpenCargo
        wait 30 ${EVEWindow[ByName, "Fleet Hangar"](exists)}

        if ${EVEWindow[ByName, "Fleet Hangar"](exists)}
        {
            ; Transfer ore to Orca
            variable iterator Item
            MyShip.Cargo:GetIterator[Item]

            if ${Item:First(exists)}
            {
                do
                {
                    if ${Item.Value.Group.Equal["Ice"]} || ${Item.Value.CategoryID} == CATEGORYID_ORE
                    {
                        Item.Value:MoveTo[OtherCargo, ${Item.Value.Quantity}]
                    }
                }
                while ${Item:Next(exists)}
            }

            wait 20
            echo "Ore delivered to Orca"
            return TRUE
        }

        return FALSE
    }
}
```

### Example 3: Uplink Multi-Computer Coordination

```lavishscript
objectdef obj_MultiComputerFleet
{
    variable string ComputerRole     ; "DPS", "Logi", "Mining", "Hauling"
    variable string FleetCommander
    variable bool IsFC = FALSE

    method Initialize()
    {
        This.ComputerRole:Set["${Config.Fleet.ComputerRole}"]
        This.FleetCommander:Set["${Config.Fleet.FCName}"]
        This.IsFC:Set[${Me.Name.Equal[${This.FleetCommander}]}]

        ; Setup uplink
        This:SetupUplink

        ; Register cross-computer events
        LavishScript:RegisterEvent[Fleet_Operation_Start]
        LavishScript:RegisterEvent[Fleet_Operation_End]
        LavishScript:RegisterEvent[Fleet_Status_Request]
        LavishScript:RegisterEvent[Fleet_Status_Report]

        Event[Fleet_Operation_Start]:AttachAtom[This:OnOperationStart]
        Event[Fleet_Operation_End]:AttachAtom[This:OnOperationEnd]
        Event[Fleet_Status_Request]:AttachAtom[This:OnStatusRequest]
        Event[Fleet_Status_Report]:AttachAtom[This:OnStatusReport]
    }

    method SetupUplink()
    {
        ; Already configured via InnerSpace console
        echo "Using uplink for multi-computer coordination"
    }

    ; FC starts operation (broadcasts to all computers)
    method StartOperation(string operation)
    {
        if !${This.IsFC}
            return

        echo "FC: Starting operation - ${operation}"

        ; Broadcast to same computer
        relay all -event Fleet_Operation_Start "${operation}"

        ; Broadcast to all computers on network
        uplink relay all -event Fleet_Operation_Start "${operation}"
    }

    ; FC ends operation
    method EndOperation()
    {
        if !${This.IsFC}
            return

        echo "FC: Ending operation"

        relay all -event Fleet_Operation_End
        uplink relay all -event Fleet_Operation_End
    }

    ; FC requests status from all computers
    method RequestFleetStatus()
    {
        if !${This.IsFC}
            return

        echo "FC: Requesting fleet status from all computers"

        ; Local
        relay "all other" -event Fleet_Status_Request

        ; Remote computers
        uplink relay all -event Fleet_Status_Request
    }

    ; All characters receive operation start
    atom OnOperationStart(string operation)
    {
        echo ">>> Starting operation: ${operation}"

        switch ${operation}
        {
            case MINING
                This:StartMiningOperation
                break
            case COMBAT
                This:StartCombatOperation
                break
            case HAULING
                This:StartHaulingOperation
                break
        }
    }

    ; All characters receive operation end
    atom OnOperationEnd()
    {
        echo ">>> Operation complete, returning to base"
        This:ReturnToStation
    }

    ; Respond to status request
    atom OnStatusRequest()
    {
        ; Build status report
        variable string status = "${Me.Name}|${This.ComputerRole}|${CurrentState}|${MyShip.ShieldPct.Precision[0]}|${Ship.CargoFull.Precision[0]}"

        ; Send to FC (same computer)
        relay "${This.FleetCommander}" -event Fleet_Status_Report "${status}"

        ; Send to FC (might be on different computer)
        uplink relay "${This.FleetCommander}" -event Fleet_Status_Report "${status}"
    }

    ; FC collects status reports
    atom OnStatusReport(string statusData)
    {
        if !${This.IsFC}
            return

        ; Parse status: Name|Role|State|Shield|Cargo
        variable string name = ${statusData.Token[1, "|"]}
        variable string role = ${statusData.Token[2, "|"]}
        variable string state = ${statusData.Token[3, "|"]}
        variable float shield = ${statusData.Token[4, "|"]}
        variable float cargo = ${statusData.Token[5, "|"]}

        echo "Status: ${name} (${role}) - State: ${state}, Shield: ${shield}%, Cargo: ${cargo}%"

        ; Take action based on status
        if ${shield} < 50
        {
            echo "WARNING: ${name} low shields!"
        }

        if ${cargo} > 90 && ${role.Equal["Mining"]}
        {
            echo "INFO: ${name} cargo nearly full"
        }
    }

    ; Computer-specific operations
    method StartMiningOperation()
    {
        if ${This.ComputerRole.Equal["Mining"]}
        {
            ; Mining computer undocks miners
            relay all -noredirect "Miner:Start"
        }
        elseif ${This.ComputerRole.Equal["Hauling"]}
        {
            ; Hauling computer prepares haulers
            relay all -noredirect "Hauler:PrepareForPickup"
        }
    }

    method StartCombatOperation()
    {
        if ${This.ComputerRole.Equal["DPS"]}
        {
            relay all -noredirect "Combat:StartRatting"
        }
        elseif ${This.ComputerRole.Equal["Logi"]}
        {
            relay all -noredirect "Logi:StartRepairs"
        }
    }
}
```

---

## Multiboxing Patterns

**Reference implementation:** `Scripts\++isxScripts++\EVE-Online\Scripts\Yamfa\Yamfa.iss` — see this script as a canonical single-file multiboxing coordinator.

### Multiboxing vs. Fleet Coordination

These two problem spaces look similar but aren't the same thing, and the best tool for each differs:

- **Multiboxing** — one player, multiple EVE clients, typically on the same physical machine, all running under one InnerSpace instance. Coordination is inter-session over the local InnerSpace uplink: near-zero latency, ordered delivery, trusted endpoints. The problem is chiefly "make N clients behave as one player's hands."
- **Fleet coordination** — multiple human players, possibly on different machines, cooperating via EVE's own fleet chat / broadcasts / watchlist plus optional cross-machine uplinks. Latency is higher, trust is weaker, and the EVE fleet API (fleet commander, wingman lookup, `FleetMember:WarpTo`) does most of the heavy lifting.

The patterns below are for the first case. They assume you control every session, every session runs your script, and `relay` to any named session Just Works with no network hop.

### Session Identification and Discovery

Multiboxing coordination is meaningless until each session knows the names of the others. InnerSpace session names are the addressable unit for `relay <SessionName> "..."`, so the two practical questions are:

1. How does each session learn what its own session name is?
2. How do slave sessions learn the master's session name (and vice versa)?

The conventional answer is a **config-driven bootstrap**: each session reads a saved config attribute at startup identifying the master, and derives its own role by comparing its character name (or session name) against the configured master. Abstract example:

```lavishscript
; At startup, decide role from config
if ${Config.Attribute[MasterName].Equal[${Me.Name}]}
    Role:Set[Master]
else
    Role:Set[Slave]

; Slaves remember the master's session name so they can send targeted relays
variable string MasterSession = ${Config.Attribute[MasterSession]}
```

A master session is effectively discovered-by-convention: the config stores `MasterName` (the character who is master) and `MasterSession` (the InnerSpace session that character is logged into), and every other session reads those attributes on boot. When the master changes, the config is updated on all sessions — typically by the master itself relaying new values, or by a manual edit before launch.

Auto-detection from EVE fleet state (fleet-commander lookup) is possible but fragile in a pure multiboxing context, because the player may not be fleeted at all while, e.g., soloing a mission across two clients.

### Relay Over the Same-Machine Uplink

Inside one InnerSpace instance, `relay` to a named session is cheap: it's an in-process message, ordered, and reliable. This is very different from cross-machine uplink, which pays real network cost and can reorder or drop.

Two useful targeting patterns:

```lavishscript
; Fan out to every session on this uplink (including sender unless -noredirect)
relay all "Script[myBot].VariableScope.Commands:Set[\"Lock,${entityID}\"]"

; Targeted to one specific session
relay "${MasterSession}" -echo "${Me.Name} is ready"

; Fan out but skip own session
relay all -noredirect "Event[MyBotAlert]:Execute[\"incoming\"]"
```

Two distinct payload styles are in common use:

- **Event-style** — sender calls `Event[...]:Execute[...]`, receivers must have pre-registered and attached an atom. Clean separation of concerns, but requires lifecycle discipline (register on startup, detach on shutdown).
- **Direct state injection** — sender writes into a known script-scoped variable or collection on the receiver, e.g. `relay all "Script[myBot].VariableScope.Commands:Set[...]"`. No event registration is needed because the receiving script is already polling that collection in its pulse. Simpler, and the pattern real Yamfa uses.

The injection style trades a little explicitness for a lot less boilerplate and is a reasonable choice when both ends of the relay are the same script.

### Master-Drives Patterns

The core shape of every multibox bot is the same: the master makes all interesting decisions, and each slave does the minimum work needed to mirror those decisions in its own client. Three patterns cover the majority of what players actually want.

**1. Target mirroring.** The master observes its own targeting list (what's currently being locked, what's locked, and what's currently active) and broadcasts one command per entity to the slaves. Slaves receive the command, verify the entity still exists in their local grid, range-check it, and lock/activate it.

```lavishscript
; Master side (runs in master's pulse, when Role == Master)
variable index:entity MyTargets
Me:GetTargets[MyTargets]
variable iterator it
MyTargets:GetIterator[it]
if ${it:First(exists)}
    do
    {
        call RelayCommand "Lock" ${it.Value.ID}
    }
    while ${it:Next(exists)}

if ${Me.ActiveTarget(exists)}
{
    call RelayCommand "Lock"   ${Me.ActiveTarget.ID}
    call RelayCommand "Target" ${Me.ActiveTarget.ID}
}

; Slave side (processes queued commands each pulse)
; For each queued command, act only if locally valid
if !${Entity[${targetID}](exists)}
    return
if ${Entity[${targetID}].IsLockedTarget} || ${Entity[${targetID}].BeingTargeted}
    return
if ${Entity[${targetID}].Distance} > ${MyShip.MaxTargetRange}
    return
Entity[${targetID}]:LockTarget
```

The slave-side guards are the load-bearing part: the master has already decided the target is worth locking, but only the slave knows its own lock slots, targeting range, and whether the entity is even on its overview grid. Never blindly execute master commands.

**2. Follow-the-leader warp.** When the master enters warp, slaves must either already be in warp or arrive shortly after or they get left behind. The cleanest implementation piggybacks on EVE's own fleet-member warp: if you're in a fleet with the master, slaves can warp directly to the master by name via Local chat lookup.

```lavishscript
; Slave side: if master is in my Local and is a fleet member, warp to them
if ${Local[${Config.Attribute[MasterName]}](exists)}
    Local[${Config.Attribute[MasterName]}].ToFleetMember:WarpTo
```

This avoids the scatter problem of every slave trying to resolve the master's destination independently. Trigger it on a relayed "I am warping" announcement from the master, or on detecting that the master's entity has left your grid.

**3. Positioning on the master.** While on grid with the master, slaves typically hold station via `Orbit`, `KeepAtRange`, or `Approach` on the master's entity, with the distance read from config.

```lavishscript
; Slave side, while on grid with master and not currently approaching something else
if !${Me.ToEntity.Approaching(exists)}
{
    if ${Entity[Name =- "${Config.Attribute[MasterName]}"](exists)}
    {
        Entity[Name =- "${Config.Attribute[MasterName]}"]:Orbit[${Config.Attribute[Orbit]}]
    }
}
```

The `=-` partial-name match is convenient because EVE entity names sometimes have trailing suffixes (ship type, corp ticker), but be aware it can match multiple entities in crowded local — prefer exact match or an ID lookup if you have the master's ship ID. The guard against `Me.ToEntity.Approaching(exists)` prevents clobbering an already-issued movement command every pulse.

### Pitfalls

- **Race conditions on warp.** Master decides to warp and immediately broadcasts "warping" while some slaves are mid-activation on a module, mid-lock, or still processing a queued command. If the slave enters warp with an activating module it can get stuck in a half-processed state. Mitigate by having slaves clear local command queues on a "master warping" signal, and tolerate a brief desync while each slave's own pulse catches up.
- **Stale-state propagation.** The master broadcasts a target ID; by the time the slave processes it, the entity may no longer exist on the slave's grid (warped off, popped, de-cloaked elsewhere). Every slave-side consumer must treat every relayed ID as a hint, not a command — re-verify existence, distance, and lock-eligibility before acting.
- **Master disconnects or crashes.** Slaves that continue orbiting / locking based on last-known master state will keep doing so indefinitely. Give slaves a watchdog: if no command or heartbeat from the master's session has arrived in N seconds, stop active behaviors and fall back to a safe default (dock, warp to safe, or just idle).
- **Name collisions.** The master-identification-by-character-name pattern breaks if two characters share a name prefix and you use partial match, or if the master character changes ship (name reuse across pilots is rare but ship-name collisions happen). Prefer `MasterSession` (an InnerSpace identifier you control) for relay targeting, and use character name only for in-game entity lookups where you have no alternative.
- **Slave-to-slave drift.** If each slave independently decides how to handle a relayed "Lock" command, two slaves on the same grid can end up with different target priorities and different active targets. This is usually fine (and often desired for DPS spread), but if you need synchronized firing — e.g., everyone on the same primary — broadcast a dedicated `Target` command after the `Lock` and make slaves explicitly `MakeActiveTarget` on it.

---

## Reference Implementations

For a complete working fleet-assist bot as a study reference, see `Scripts\++isxScripts++\EVE-Online\Scripts\Yamfa\Yamfa.iss` — a single-file multiboxing coordinator using an `objToon` / `objShip` / `objConfig` pulse architecture, with relay routing driven by `Config.Attribute[MasterSession]`. EVEBot and Tehbot (referenced throughout this guide) provide heavier, modular counterpoints for comparison.

---

## Summary

### Key Takeaways

1. **Relay is IPC Foundation**
   - Enables multi-session coordination
   - Event-driven architecture
   - Supports same-computer and network communication

2. **Event System is Critical**
   - Always register events before use
   - Always detach in shutdown
   - Use -noredirect for method calls

3. **UplinkManager Pattern**
   - Automatic peer discovery
   - Heartbeat and pruning
   - Dynamic variable insertion

4. **Fleet Coordination Patterns**
   - HARDSTOP for emergencies
   - Master/slave for leadership
   - Request/response for queries

5. **Performance Matters**
   - Throttle relay broadcasts
   - Queue messages
   - Compress data

6. **Uplink for Multi-Computer**
   - Star or mesh topology
   - Same relay/event syntax
   - Enables large-scale coordination
