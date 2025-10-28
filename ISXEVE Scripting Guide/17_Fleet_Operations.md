# Fleet Operations

**Purpose:** Multi-boxing, fleet coordination patterns, Yamfa analysis, and relay/IPC for fleet operations
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

### Yamfa Analysis
13. [Yamfa Fleet Assist Analysis](#yamfa-fleet-assist-analysis)
14. [Yamfa Overview](#yamfa-overview)
15. [Architecture Comparison](#architecture-comparison)
16. [Master-Slave Pattern](#master-slave-pattern)
17. [Hysteresis and Timing](#hysteresis-and-timing)
18. [Relay Communication](#relay-communication)
19. [Target Management](#target-management)
20. [Movement and Following](#movement-and-following)
21. [Code Strengths](#code-strengths)
22. [Code Weaknesses and Fixes](#code-weaknesses-and-fixes)
23. [Improvement Roadmap](#improvement-roadmap)
24. [Lessons for the Community](#lessons-for-the-community)

### Relay System and IPC
25. [Relay System and Inter-Process Communication](#relay-system-and-inter-process-communication-ipc)
26. [LavishScript Relay Basics](#relay-basics)
27. [Event System Foundation](#event-system)
28. [Basic Relay Patterns](#basic-patterns)
29. [UplinkManager System](#uplink-system)
30. [Fleet Coordination Patterns](#fleet-patterns)
31. [IRC Bridge Integration](#irc-bridge)
32. [Uplink Networking](#uplink-networking)
33. [Performance Considerations](#performance)
34. [Best Practices](#best-practices)
35. [Complete Working Examples (Relay)](#examples)

---

## Multi-Boxing and Fleet Coordination

---

## Introduction to Multi-Boxing

### What is Multi-Boxing?

Multi-boxing is running multiple EVE clients simultaneously on one or more computers, each controlled by automation. This allows:

- **Mining Fleets**: One Orca + multiple miners working in coordination
- **Combat Fleets**: DPS ships + logi + tackle working together
- **Hauler Support**: Dedicated hauler servicing multiple miners
- **Market Operations**: Multiple traders across different trade hubs
- **Boosting**: Command ship providing bonuses to fleet

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

### Relay Basics

LavishScript's `relay` command sends messages between sessions on the same computer or across Uplink network.

**Syntax**:
```lavish
relay <target> <command>
relay <target> -event <event_name> <parameters>
```

**Targets**:
- `all` - All sessions
- `other` - All other sessions (not self)
- `"SessionName"` - Specific session by name
- `local` - Only local computer sessions
- `uplink` - All Uplink network sessions

### Basic Relay Examples

```lavish
// Send command to all sessions
relay all echo "Hello from ${Me.Name}"

// Send command to specific session
relay "EVESession1" echo "Message from ${Me.Name}"

// Send command to other sessions (not self)
relay other echo "Alert from ${Me.Name}"

// Send event to all sessions
relay all -event MyCustomEvent ${Me.CharID} ${Me.SolarSystemID}
```

### Receiving Relay Events

```lavish
// Register custom event
LavishScript:RegisterEvent[MyCustomEvent]

// Attach atom to handle event
Event[MyCustomEvent]:AttachAtom[This:OnMyCustomEvent]

// Event handler
atom OnMyCustomEvent(int64 charID, int64 systemID)
{
    echo "Received event from CharID: ${charID} in system ${systemID}"
}

// Cleanup
Event[MyCustomEvent]:DetachAtom[This:OnMyCustomEvent]
```

### Event Registration Pattern

```lavish
method Initialize()
{
    // Register all custom events
    LavishScript:RegisterEvent[EVEBot_HARDSTOP]
    LavishScript:RegisterEvent[EVEBot_Miner_Full]
    LavishScript:RegisterEvent[EVEBot_Master_InBelt]
    LavishScript:RegisterEvent[EVEBot_AttackerReport]

    // Attach handlers
    Event[EVEBot_HARDSTOP]:AttachAtom[This:OnHardStop]
    Event[EVEBot_Miner_Full]:AttachAtom[This:OnMinerFull]
    Event[EVEBot_Master_InBelt]:AttachAtom[This:OnMasterInBelt]
    Event[EVEBot_AttackerReport]:AttachAtom[This:OnAttackerReport]
}

method Shutdown()
{
    // Detach all handlers
    Event[EVEBot_HARDSTOP]:DetachAtom[This:OnHardStop]
    Event[EVEBot_Miner_Full]:DetachAtom[This:OnMinerFull]
    Event[EVEBot_Master_InBelt]:DetachAtom[This:OnMasterInBelt]
    Event[EVEBot_AttackerReport]:DetachAtom[This:OnAttackerReport]
}
```

### Emergency Stop Broadcast (EVEBot Pattern)

```lavish
// When any bot detects danger, broadcast to all
if ${Social.PossibleHostiles}
{
    echo "HOSTILES DETECTED - Broadcasting emergency stop"
    relay all -event EVEBot_HARDSTOP "${Me.Name} - ${Config.Common.CurrentBehavior} (Hostiles)"
    EVEBot.ReturnToStation:Set[TRUE]
}

// All bots listen for emergency stop
atom OnHardStop(string message)
{
    echo "HARD STOP received: ${message}"
    EVEBot.ReturnToStation:Set[TRUE]
    This.CurrentState:Set["FLEE"]
}
```

### Uplink for Multi-Computer Coordination

**Setup Uplink Server** (on one computer):
```lavish
uplink create MyFleetUplink password123
uplink start MyFleetUplink
```

**Connect from Other Computers**:
```lavish
uplink connect 192.168.1.100:2048 password123
```

**Send Commands Across Network**:
```lavish
// From computer A, command all computers
relay uplink echo "Message from ${Me.Name} to entire network"
```

---

## Fleet Management

### Fleet Object (EVEBot Pattern)

Based on `obj_Fleet.iss`:

```lavish
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

```lavish
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

    if ${Local[${locationName}](exists)}
    {
        if ${Entity[${locationName}](exists)}
        {
            // Warp to entity
            Entity[${locationName}]:FleetWarpTo[${distance}]
            echo "Fleet warp to ${locationName} at ${distance}m"
        }
        elseif ${Local[${locationName}].ToFleetMember(exists)}
        {
            // Warp to fleet member
            Local[${locationName}].ToFleetMember:FleetWarpTo[${distance}]
            echo "Fleet warp to ${locationName} at ${distance}m"
        }
    }
    else
    {
        echo "ERROR: ${locationName} not found"
        return FALSE
    }

    return TRUE
}
```

### Fleet Broadcast Monitoring

```lavish
// Watch for fleet broadcasts
atom OnFleetBroadcast(string broadcastType, int64 broadcasterID)
{
    echo "Fleet broadcast: ${broadcastType} from ${broadcasterID}"

    switch ${broadcastType}
    {
        case "NeedShieldRepair"
            echo "Shield repair needed on ${Entity[${broadcasterID}].Name}"
            call This.RepShields ${broadcasterID}
            break

        case "NeedArmorRepair"
            echo "Armor repair needed on ${Entity[${broadcasterID}].Name}"
            call This.RepArmor ${broadcasterID}
            break

        case "NeedCapacitor"
            echo "Capacitor needed on ${Entity[${broadcasterID}].Name}"
            call This.TransferCap ${broadcasterID}
            break

        case "InPosition"
            echo "${Entity[${broadcasterID}].Name} reports in position"
            break

        case "HoldPosition"
            echo "Fleet broadcast: Hold position"
            EVE:Execute[CmdStopShip]
            break

        case "WarpTo"
            echo "Fleet broadcast: Warp to target"
            // Handled by FleetWarpTo
            break
    }
}

// Register fleet broadcast handler
Event[EVE_OnFleetBroadcast]:AttachAtom[This:OnFleetBroadcast]
```

---

## Character Coordination Patterns

### Synchronized Actions

```lavish
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

```lavish
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
                if ${bmIterator.Value.SolarSystemID} == ${Me.SolarSystemID}
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

```lavish
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

```lavish
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

```lavish
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

One bot leads, others follow:

```lavish
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

```lavish
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

```lavish
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

```lavish
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

```lavish
objectdef obj_FCTargetCaller
{
    variable int64 PrimaryTarget = 0
    variable int64 SecondaryTarget = 0

    // FC calls primary target
    method CallPrimary(int64 targetID)
    {
        if !${Entity[${targetID}](exists)}
        {
            echo "ERROR: Target does not exist"
            return FALSE
        }

        This.PrimaryTarget:Set[${targetID}]

        echo "CALLING PRIMARY: ${Entity[${targetID}].Name}"
        relay all -event FC_PrimaryTarget ${targetID}

        return TRUE
    }

    // Call secondary (backup target)
    method CallSecondary(int64 targetID)
    {
        This.SecondaryTarget:Set[${targetID}]

        echo "CALLING SECONDARY: ${Entity[${targetID}].Name}"
        relay all -event FC_SecondaryTarget ${targetID}
    }

    // Auto-call next primary when current dies
    atom OnTargetDestroyed(int64 targetID)
    {
        if ${targetID} == ${This.PrimaryTarget}
        {
            echo "Primary destroyed - calling secondary as new primary"

            if ${This.SecondaryTarget} > 0 && ${Entity[${This.SecondaryTarget}](exists)}
            {
                call This.CallPrimary ${This.SecondaryTarget}
            }
            else
            {
                echo "Finding new primary..."
                call This.FindAndCallPrimary
            }
        }
    }

    function FindAndCallPrimary()
    {
        // Find highest priority target
        variable index:entity hostiles
        EVE:QueryEntities[hostiles, "IsNPC && !IsMoribund && Distance < 200000", "Distance"]

        if ${hostiles.Used} > 0
        {
            call This.CallPrimary ${hostiles[1].ID}
        }
    }
}

// DPS ships respond to primary calls
objectdef obj_DPSShip
{
    variable int64 PrimaryTarget = 0

    method Initialize()
    {
        LavishScript:RegisterEvent[FC_PrimaryTarget]
        Event[FC_PrimaryTarget]:AttachAtom[This:OnPrimaryTarget]
    }

    atom OnPrimaryTarget(int64 targetID)
    {
        echo "PRIMARY CALLED: ${Entity[${targetID}].Name}"

        This.PrimaryTarget:Set[${targetID}]

        // Lock and shoot primary
        call This.EngagePrimary
    }

    function EngagePrimary()
    {
        if !${Entity[${This.PrimaryTarget}](exists)}
        {
            echo "Primary no longer exists"
            return
        }

        // Lock target
        if !${Entity[${This.PrimaryTarget}].IsLockedTarget} &&
           !${Entity[${This.PrimaryTarget}].BeingTargeted}
        {
            Entity[${This.PrimaryTarget}]:LockTarget
            wait 10
        }

        // Wait for lock
        variable int timeout = 0
        while !${Entity[${This.PrimaryTarget}].IsLockedTarget} && ${timeout} < 100
        {
            wait 10
            timeout:Inc[10]
        }

        // Make active target and activate weapons
        Entity[${This.PrimaryTarget}]:MakeActiveTarget
        wait 10

        Ship:Activate_Weapons
    }
}
```

### Logistics Coordination (Shield/Armor Reps)

```lavish
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

                        if ${shieldPct} < 70 && ${Ship.ModuleList_ShieldTransporter.Used} > 0
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
        Ship.ModuleList_ShieldTransporter:GetIterator[modIt]

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

```lavish
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

```lavish
objectdef obj_MiningFleetCoordinator
{
    variable string OrcaPilotName = "MyOrca"
    variable int64 CurrentBeltID = 0

    method Initialize()
    {
        // Orca = Master
        // Miners = Slaves

        if ${Me.Name.Equal["${This.OrcaPilotName}"]}
        {
            echo "I am the Orca - initializing as fleet master"
            call This.InitializeOrcaMaster
        }
        else
        {
            echo "I am a miner - initializing as slave"
            call This.InitializeMinerSlave
        }
    }

    method InitializeOrcaMaster()
    {
        // Orca selects belt and broadcasts to fleet
        call This.SelectBelt

        // Broadcast belt location
        relay all -event Fleet_WarpToBelt ${This.CurrentBeltID}

        // Warp to belt
        Entity[${This.CurrentBeltID}]:WarpTo[30000]
    }

    method SelectBelt()
    {
        variable index:entity belts
        EVE:QueryEntities[belts, "GroupID = GROUP_ASTEROID_BELT"]

        if ${belts.Used} > 0
        {
            This.CurrentBeltID:Set[${belts[1].ID}]
            echo "Selected belt: ${belts[1].Name}"
        }
    }

    method InitializeMinerSlave()
    {
        // Wait for Orca to broadcast belt
        LavishScript:RegisterEvent[Fleet_WarpToBelt]
        Event[Fleet_WarpToBelt]:AttachAtom[This:OnWarpToBelt]
    }

    atom OnWarpToBelt(int64 beltID)
    {
        echo "Orca orders: Warp to belt ${Entity[${beltID}].Name}"

        Entity[${beltID}]:WarpTo[0]
        wait 20

        while ${Me.ToEntity.Mode} == 3
        {
            wait 10
        }

        // Start mining
        This.CurrentState:Set["MINE"]
    }
}
```

### Boosting Coordination

```lavish
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
        Ship.ModuleList_GangCoordinator:GetIterator[modIt]

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
        Ship.ModuleList_GangCoordinator:GetIterator[modIt]

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

```lavish
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

```lavish
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

```lavish
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
```lavish
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
```lavish
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
            EVE:Execute[CmdOpenInventory]
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
```lavish
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
```lavish
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
```lavish
echo "Event registered: ${LavishScript.RegisteredEvent[MyEvent](exists)}"
echo "Handler attached: ${Event[MyEvent].HasAtom}"
```

**Solution**:
```lavish
// Ensure event is registered BEFORE sending
LavishScript:RegisterEvent[MyEvent]
Event[MyEvent]:AttachAtom[This:OnMyEvent]

// Then send
relay all -event MyEvent "test"
```

### Problem 2: Session Names Unknown

**Symptom**: Can't relay to specific session

**Diagnosis**:
```lavish
echo "Session name: ${Session}"
echo "Session uplink ID: ${Session.UplinkID}"
```

**Solution**:
```lavish
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
```lavish
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
```lavish
echo "Uplink connected: ${Uplink.IsConnected}"
echo "Uplink sessions: ${Uplink.Sessions}"
```

**Solution**:
```lavish
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
```lavish
echo "In fleet: ${Me.Fleet.IsMember[${Me.CharID}]}"
echo "Fleet members: ${Me.Fleet.MemberCount}"
```

**Solution**:
```lavish
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

```lavish
function FleetStatusReport()
{
    echo "==== FLEET STATUS REPORT ===="

    echo "Session: ${Session}"
    echo "Character: ${Me.Name} (${Me.CharID})"

    echo "In fleet: ${Me.Fleet.IsMember[${Me.CharID}]}"

    if ${Me.Fleet.IsMember[${Me.CharID}]}
    {
        echo "Fleet members: ${Me.Fleet.MemberCount}"

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

## Yamfa Fleet Assist Analysis

---

## Yamfa Overview

### What is Yamfa?

**[Yamfa](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/Yamfa)** - **Y**et **A**nother **M**ulti **F**leet **A**ssist - A lightweight fleet assist bot for target synchronization and fleet following.

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

```lavish
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

```lavish
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

```lavish
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

```lavish
; Event can have multiple atoms attached
Event[MyEvent]:AttachAtom[Object1:Handler1]
Event[MyEvent]:AttachAtom[Object2:Handler2]
Event[MyEvent]:AttachAtom[Object3:Handler3]

; When event executes, ALL attached atoms are called
Event[MyEvent]:Execute["param"]
```

### Event Broadcasting

```lavish
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

```lavish
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

```lavish
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
        variable float shieldPct = ${MyShip.Shield.Pct}
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

```lavish
objectdef obj_FleetSync
{
    variable string FleetState = "IDLE"

    method Initialize()
    {
        LavishScript:RegisterEvent[Fleet_State_Change]
        Event[Fleet_State_Change]:AttachAtom[This:OnStateChange]
    }

    ; Master: Change fleet state
    method SetFleetState(string newState)
    {
        if !${IsMaster}
            return

        This.FleetState:Set[${newState}]
        echo "FC: Fleet state -> ${newState}"

        ; Broadcast to all
        relay all -event Fleet_State_Change "${newState}"
    }

    ; All: React to state change
    atom OnStateChange(string newState)
    {
        This.FleetState:Set[${newState}]

        switch ${newState}
        {
            case MINING
                This:StartMining
                break
            case HAULING
                This:StartHauling
                break
            case COMBAT
                This:StartCombat
                break
            case DOCKED
                This:ReturnToStation
                break
        }
    }
}
```

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

```lavish
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

```lavish
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

```lavish
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
            ${If[${Me.Skill[Wing Command](exists)}, ${Me.Skill[Wing Command].Level}, 0]}, \
            ${If[${Me.Skill[Fleet Command](exists)}, ${Me.Skill[Fleet Command].Level}, 0]}, \
            ${If[${Me.Skill[Mining Foreman](exists)}, ${Me.Skill[Mining Foreman].Level}, 0]}, \
            ${If[${Me.Skill[Armored Warfare](exists)}, ${Me.Skill[Armored Warfare].Level}, 0]}]"
}

; Receive and store peer skills
method UpdatePeerSkills(string RemoteSessionName, int Leadership, int Wing_Command, int Fleet_Command,
                        int Armored_Warfare, int Information_Warfare, int Mining_Foreman)
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

```lavish
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

```lavish
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

```lavish
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

```lavish
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

```lavish
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

```lavish
objectdef obj_YamfaTargetRelay
{
    ; Master variables
    variable string CurrentTargetList = ""
    variable int64 PrimaryTarget = 0

    ; Slave variables
    variable string RelayedTargets = ""
    variable int64 RelayedPrimary = 0
    variable bool NewRelayData = FALSE

    method Initialize()
    {
        LavishScript:RegisterEvent[YamfaTargets]
        Event[YamfaTargets]:AttachAtom[This:OnYamfaTargets]
    }

    ; MASTER: Broadcast targets every pulse
    method BroadcastTargets()
    {
        if !${IsMaster}
            return

        ; Build target ID list
        This.CurrentTargetList:Set[""]
        variable iterator Target
        MyTargets:GetIterator[Target]

        if ${Target:First(exists)}
        {
            do
            {
                This.CurrentTargetList:Concat["${Target.Value.ID},"]
            }
            while ${Target:Next(exists)}
        }

        ; Broadcast to slaves
        relay "all other" -noredirect \
            "Event[YamfaTargets]:Execute[\"${This.CurrentTargetList}\",${This.PrimaryTarget}]"
    }

    ; SLAVE: Receive relayed targets
    atom OnYamfaTargets(string targetIDs, int64 primaryTarget)
    {
        if ${IsMaster}
            return

        This.RelayedTargets:Set[${targetIDs}]
        This.RelayedPrimary:Set[${primaryTarget}]
        This.NewRelayData:Set[TRUE]
    }

    ; SLAVE: Process relayed targets
    method ProcessRelayedTargets()
    {
        if !${This.NewRelayData}
            return

        This.NewRelayData:Set[FALSE]

        ; Parse comma-separated target IDs
        variable string TargetList = "${This.RelayedTargets}"
        variable int NumTargets = 0

        ; Count targets
        while ${TargetList.Find[","](exists)}
        {
            variable int64 targetID = ${TargetList.Token[1, ","]}

            if ${Entity[${targetID}](exists)} && !${Entity[${targetID}].BeingTargeted}
            {
                Entity[${targetID}]:LockTarget
            }

            TargetList:Set[${TargetList.Right[${Math.Calc[${TargetList.Length} - ${TargetList.Find[","]}]}]}]
            NumTargets:Inc
        }

        echo "Locked ${NumTargets} targets from master"
    }
}
```

### Orca Service Pattern

**Scenario:** Miners request Orca services (survey scan, shield boosts)

```lavish
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
        if ${MyShip.Shield.Pct} < 50
        {
            relay "${OrcaName}" -event Orca_Shield_Request ${Me.ToEntity.ID} ${MyShip.Shield.Pct}
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

```lavish
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

```lavish
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

```lavish
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

```lavish
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

```lavish
; Report fleet status to IRC
method ReportFleetStatus()
{
    variable string status = ""

    ; Build status report
    status:Concat["${Me.Name}: "]
    status:Concat["State=${CurrentState} "]
    status:Concat["Shield=${MyShip.Shield.Pct.Precision[0]}% "]
    status:Concat["Cap=${MyShip.Capacitor.Pct.Precision[0]}% "]
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

```lavish
; In InnerSpace console on Computer 1:
uplink create MyFleet
uplink MyFleet connect 192.168.1.100:54321

; In InnerSpace console on Computer 2:
uplink create MyFleet
uplink MyFleet listen 54321
```

### Uplink Relay Syntax

```lavish
; Relay across uplink
uplink relay all echo "Message to all computers"
uplink relay "CharName@Computer2" echo "Specific char on remote computer"

; Uplink event broadcast
uplink relay all -event Fleet_Warp ${destinationID}
```

### Uplink in Code

```lavish
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

```lavish
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

```lavish
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

```lavish
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

```lavish
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
        relay "all other" -event Fleet_Status "${Me.Name}" ${MyShip.Shield.Pct}
    }
}
```

**Solution 2: Change Detection**

```lavish
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

```lavish
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

```lavish
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

```lavish
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

```lavish
; WRONG - can create relay loops
relay all "MyObject:MyMethod"

; RIGHT - no relay loop
relay all -noredirect "MyObject:MyMethod"
```

### 2. Always Detach Events in Shutdown

```lavish
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

```lavish
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

```lavish
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

```lavish
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

```lavish
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

```lavish
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

```lavish
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

```lavish
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
        ; Priority 1: Targets shooting FC
        variable int64 fcID = ${Entity["Name = \"${This.MasterName}\""].ID}
        variable index:entity Threats
        variable iterator Threat

        EVE:QueryEntities[Threats, "CategoryID = CATEGORYID_ENTITY && IsNPC"]
        Threats:GetIterator[Threat]

        if ${Threat:First(exists)}
        {
            do
            {
                if ${Threat.Value.IsTargetingMe} || ${Threat.Value.IsActivelyTargeting[${fcID}]}
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

```lavish
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

```lavish
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
        variable string status = "${Me.Name}|${This.ComputerRole}|${CurrentState}|${MyShip.Shield.Pct.Precision[0]}|${Ship.CargoFull.Precision[0]}"

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
