# Advanced Patterns And Examples

**Purpose:** Advanced patterns for multi-boxing, relay/IPC communication, and configuration management
**Audience:** Developers building complex multi-client automation systems

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
11. [Complete Working Examples (Fleet)](#complete-working-examples)
12. [Common Problems (Fleet)](#common-problems)

### Relay System and IPC
13. [Relay System and Inter-Process Communication](#relay-system-and-inter-process-communication-ipc)
14. [LavishScript Relay Basics](#relay-basics)
15. [Event System Foundation](#event-system)
16. [Basic Relay Patterns](#basic-patterns)
17. [UplinkManager System](#uplink-system)
18. [Fleet Coordination Patterns](#fleet-patterns)
19. [IRC Bridge Integration](#irc-bridge)
20. [Uplink Networking](#uplink-networking)
21. [Performance Considerations (Relay)](#performance)
22. [Best Practices (Relay)](#best-practices)
23. [Complete Working Examples (Relay)](#examples)

### Configuration Management
24. [Configuration and Settings Management](#configuration-and-settings-management)
25. [LavishSettings Foundation](#lavishsettings)
26. [EVEBot Configuration Architecture](#evebot-config)
27. [Tehbot Configuration Architecture](#tehbot-config)
28. [Configuration Patterns](#config-patterns)
29. [XML File Structure](#xml-structure)
30. [Config Macros and Helpers](#macros)
31. [UI Integration](#ui-integration)
32. [Per-Character Settings](#per-character)
33. [Config Migration](#migration)
34. [Fleet Config Synchronization](#fleet-sync)
35. [isxSQLite Configuration Storage](#isxsqlite-config)
36. [Best Practices (Config)](#best-practices-1)
37. [Complete Examples (Config)](#examples-1)

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

## Configuration and Settings Management

---

## LavishSettings Foundation

### What is LavishSettings?

**LavishSettings** is LavishScript's hierarchical configuration API:
- Stores settings in XML files
- Supports nested Sets and Settings
- Import/Export to/from XML
- Runtime modification with auto-save

### Basic LavishSettings API

```lavish
; Create a settings object
LavishSettings:AddSet[MySettings]

; Add a nested set
LavishSettings[MySettings]:AddSet[Character1]

; Add individual settings
LavishSettings[MySettings].FindSet[Character1]:AddSetting[Name, "Jovehn"]
LavishSettings[MySettings].FindSet[Character1]:AddSetting[HomeStation, "Jita 4-4"]
LavishSettings[MySettings].FindSet[Character1]:AddSetting[IsActive, TRUE]

; Read settings
variable string name = ${LavishSettings[MySettings].FindSet[Character1].FindSetting[Name]}
variable bool active = ${LavishSettings[MySettings].FindSet[Character1].FindSetting[IsActive]}

; Export to XML
LavishSettings[MySettings]:Export["config/MyConfig.xml"]

; Import from XML
LavishSettings[MySettings]:Import["config/MyConfig.xml"]

; Remove settings
LavishSettings[MySettings]:Clear
LavishSettings[MySettings]:Remove
```

### Settings Hierarchy

```
LavishSettings[MySettings]                    ; Root settings object
    └── FindSet[Character1]                   ; Character set
        ├── FindSetting[Name]                 ; Individual setting
        ├── FindSetting[HomeStation]          ; Individual setting
        └── FindSet[Behavior]                 ; Nested set
            ├── FindSetting[Mode]             ; Nested setting
            └── FindSetting[Target]           ; Nested setting
```

### settingsetref Type

**settingsetref** is a reference to a settings set, used for efficient access:

```lavish
objectdef obj_MyConfig
{
    variable settingsetref CharacterRef

    method Initialize()
    {
        ; Set reference to a deeply nested set
        CharacterRef:Set[${LavishSettings[MySettings].FindSet[Character1]}]

        ; Now access is simpler
        variable string name = ${This.CharacterRef.FindSetting[Name]}
    }
}
```

---

## EVEBot Configuration Architecture

### Overview

EVEBot uses a sophisticated multi-layer configuration system:

**Architecture:**
```
obj_Configuration_BaseConfig                  ; File I/O, core functionality
    ├── Manages ${Me.Name} Config.xml
    ├── Import/Export
    └── Save on shutdown

obj_Configuration                             ; Main config container
    ├── Common       (obj_Configuration_Common)
    ├── Combat       (obj_Configuration_Combat)
    ├── Miner        (obj_Configuration_Miner)
    ├── Hauler       (obj_Configuration_Hauler)
    ├── Missioneer   (obj_Configuration_Missioneer)
    ├── Fleet        (obj_Configuration_Fleet)
    └── [etc...]
```

### BaseConfig Implementation

```lavish
objectdef obj_Configuration_BaseConfig
{
    variable float ConfigVersion = 1.0
    variable filepath CONFIG_PATH = "${Script.CurrentDirectory}/Config"
    variable string CONFIG_FILE = "${Me.Name} Config.xml"
    variable settingsetref BaseRef

    method Initialize()
    {
        ; Remove old settings
        LavishSettings[EVEBotSettings]:Remove

        ; Create new settings
        LavishSettings:AddSet[EVEBotSettings]
        LavishSettings[EVEBotSettings]:AddSet[${Me.Name}]

        ; Check if config file exists
        if !${CONFIG_PATH.FileExists[${CONFIG_FILE}]}
        {
            Logger:Log["${CONFIG_FILE} not found - creating new"]
            ; Config modules will set defaults
        }
        else
        {
            Logger:Log["Configuration file is ${CONFIG_FILE}"]
            LavishSettings[EVEBotSettings]:Import[${CONFIG_PATH}/${CONFIG_FILE}]
        }

        ; Set reference to character's settings
        BaseRef:Set[${LavishSettings[EVEBotSettings].FindSet[${Me.Name}]}]
    }

    method Shutdown()
    {
        This:Save
        LavishSettings[EVEBotSettings]:Remove
    }

    method Save()
    {
        Logger:Log["Saving configuration to ${CONFIG_FILE}"]
        LavishSettings[EVEBotSettings]:Export[${CONFIG_PATH}/${CONFIG_FILE}]
    }
}
```

### Main Config Object

```lavish
objectdef obj_Configuration
{
    ; Sub-configuration objects
    variable obj_Configuration_Common Common
    variable obj_Configuration_Combat Combat
    variable obj_Configuration_Miner Miner
    variable obj_Configuration_Hauler Hauler
    variable obj_Configuration_Missioneer Missioneer
    variable obj_Configuration_Fleet Fleet

    method Shutdown()
    {
        ; Sub-configs don't need individual shutdown
        ; BaseConfig handles save
    }

    method Save()
    {
        ; Delegate to BaseConfig
        BaseConfig:Save
    }
}
```

**Usage:**

```lavish
; Access is clean and organized
variable string behavior = ${Config.Common.CurrentBehavior}
variable int minShield = ${Config.Combat.MinimumShieldPct}
variable bool useDrones = ${Config.Miner.UseMiningDrones}

; Set values
Config.Common:SetCurrentBehavior["Miner"]
Config.Combat:SetMinimumShieldPct[50]
Config.Miner:SetUseMiningDrones[TRUE]

; Save to disk
Config:Save
```

### Sub-Configuration Pattern

```lavish
objectdef obj_Configuration_Common
{
    variable string SetName = "Common"
    variable settingsetref Ref

    method Initialize()
    {
        ; Check if set exists in config file
        if !${BaseConfig.BaseRef.FindSet[${This.SetName}](exists)}
        {
            Logger:Log["Warning: ${This.SetName} settings missing - initializing"]
            This:Set_Default_Values
        }
        else
        {
            ; Set exists - get reference
            Ref:Set[${BaseConfig.BaseRef.FindSet[${This.SetName}]}]
        }
    }

    method Set_Default_Values()
    {
        ; Create the set
        BaseConfig.BaseRef:AddSet[${This.SetName}]
        Ref:Set[${BaseConfig.BaseRef.FindSet[${This.SetName}]}]

        ; Add default settings
        This.Ref:AddSetting[Home Station, 1]
        This.Ref:AddSetting[CurrentBehavior, "Idle"]
        This.Ref:AddSetting[AutoLogin, TRUE]
        This.Ref:AddSetting[Maximum Runtime, 0]
        This.Ref:AddSetting[Use Sound, FALSE]
        This.Ref:AddSetting[Disable 3D, FALSE]
    }

    ; Member to read
    member:string CurrentBehavior()
    {
        return ${This.Ref.FindSetting[CurrentBehavior, "Idle"]}
    }

    ; Method to write
    method SetCurrentBehavior(string value)
    {
        This.Ref:AddSetting[CurrentBehavior, ${value}]
    }

    ; More settings...
    member:int HomeStation()
    {
        return ${This.Ref.FindSetting[Home Station, 1]}
    }

    method SetHomeStation(int value)
    {
        This.Ref:AddSetting[Home Station, ${value}]
    }

    member:bool AutoLogin()
    {
        return ${This.Ref.FindSetting[AutoLogin, TRUE]}
    }

    method SetAutoLogin(bool value)
    {
        This.Ref:AddSetting[AutoLogin, ${value}]
    }
}
```

---

## Tehbot Configuration Architecture

### Overview

[Tehbot](https://github.com/isxGames/Tehbot) uses a cleaner inheritance-based pattern:

**Architecture:**
```
obj_Configuration_Manager                     ; Singleton manager
    ├── CONFIG_FILE
    ├── ConfigRoot (settingsetref)
    └── Initialize/Shutdown/Save

obj_Configuration_Base                        ; Base class
    ├── SetName
    ├── ConfigRef (member)
    └── Set_Default_Values (method)

obj_Configuration_Common : obj_Configuration_Base
obj_Configuration_Behavior : obj_Configuration_Base
obj_Configuration_Combat : obj_Configuration_Base
[etc...]
```

### ConfigManager Singleton

```lavish
objectdef obj_Configuration_Manager
{
    variable string CONFIG_FILE = "${Me.Name} Config.xml"
    variable filepath CONFIG_PATH = "${Script.CurrentDirectory}/config"
    variable settingsetref ConfigRoot

    method Initialize()
    {
        ; Handle alternative character name
        if ${EVEExtension.Character.Length}
        {
            CONFIG_FILE:Set["${EVEExtension.Character} Config.xml"]
        }

        ; Clear and recreate settings
        LavishSettings[TehbotSettings]:Clear
        LavishSettings:AddSet[TehbotSettings]

        ; Add character set
        if ${EVEExtension.Character.Length}
        {
            LavishSettings[TehbotSettings]:AddSet[${EVEExtension.Character}]
        }
        else
        {
            LavishSettings[TehbotSettings]:AddSet[${Me.Name}]
        }

        ; Import if exists
        if ${CONFIG_PATH.FileExists["${CONFIG_PATH}/${CONFIG_FILE}"]}
        {
            LavishSettings[TehbotSettings]:Import["${CONFIG_PATH}/${CONFIG_FILE}"]
        }

        ; Set root reference
        if ${EVEExtension.Character.Length}
        {
            ConfigRoot:Set[${LavishSettings[TehbotSettings].FindSet[${EVEExtension.Character}]}]
        }
        else
        {
            ConfigRoot:Set[${LavishSettings[TehbotSettings].FindSet[${Me.Name}]}]
        }
    }

    method Shutdown()
    {
        This:Save
        LavishSettings[TehbotSettings]:Clear
    }

    method Save()
    {
        LavishSettings[TehbotSettings]:Export["${CONFIG_PATH}/${CONFIG_FILE}"]
    }
}

; Global instantiation
variable(global) obj_Configuration_Manager ConfigManager
```

### Base Configuration Class

```lavish
objectdef obj_Configuration_Base
{
    variable string SetName = ""

    method Initialize(string name)
    {
        SetName:Set[${name}]

        ; Check if set exists
        if !${ConfigManager.ConfigRoot.FindSet[${This.SetName}](exists)}
        {
            Logger:Log["${This.SetName} settings missing - initializing"]
            ConfigManager.ConfigRoot:AddSet[${This.SetName}]
            This:Set_Default_Values
        }
    }

    ; Get reference to this config's set
    member:settingsetref ConfigRef()
    {
        return ${ConfigManager.ConfigRoot.FindSet[${This.SetName}]}
    }

    ; Override in derived classes
    method Set_Default_Values()
    {
        ; Nothing - override in derived classes
    }
}
```

### Derived Configuration Classes

```lavish
objectdef obj_Configuration_Common inherits obj_Configuration_Base
{
    method Initialize()
    {
        ; Call parent with set name
        This[parent]:Initialize["Common"]
    }

    method Set_Default_Values()
    {
        ; This.ConfigRef is provided by base class
        This.ConfigRef:AddSetting[Tehbot_Mode, "MiniMode"]
        This.ConfigRef:AddSetting[ActiveTab, "Status"]
        This.ConfigRef:AddSetting[LogLevelBar, LOG_INFO]
        This.ConfigRef:AddSetting[AutoStart, FALSE]
        This.ConfigRef:AddSetting[Disable3D, FALSE]
        This.ConfigRef:AddSetting[DisableUI, FALSE]
    }

    ; Settings using Setting() macro (defined elsewhere)
    Setting(string, Tehbot_Mode, SetTehbot_Mode)
    Setting(bool, AutoStart, SetAutoStart)
    Setting(bool, Disable3D, SetDisable3D)
    Setting(bool, DisableUI, SetDisableUI)
    Setting(string, ActiveTab, SetActiveTab)
    Setting(int, LogLevelBar, SetLogLevelBar)
}
```

**Setting() Macro:**

```lavish
#define Setting(type, name, setMethod) \
    member:type name() \
    { \
        return ${This.ConfigRef.FindSetting[#name]} \
    } \
    method setMethod(type value) \
    { \
        This.ConfigRef:AddSetting[#name, ${value}] \
    }
```

---

## Configuration Patterns

### Pattern 1: Member/Method Pair

**Every config value needs:**
- **Member:** Read-only accessor
- **Method:** Write accessor that updates config

```lavish
; MEMBER - Read value
member:int MinimumShieldPct()
{
    return ${This.ConfigRef.FindSetting[MinimumShieldPct, 50]}
                                                          ; ^^^ Default value
}

; METHOD - Write value
method SetMinimumShieldPct(int value)
{
    This.ConfigRef:AddSetting[MinimumShieldPct, ${value}]
}

; Usage
variable int shield = ${Config.Combat.MinimumShieldPct}    ; Read
Config.Combat:SetMinimumShieldPct[75]                      ; Write
```

### Pattern 2: Default Values

**Always provide defaults** in case setting doesn't exist:

```lavish
; Inline default
member:string CurrentBehavior()
{
    return ${This.ConfigRef.FindSetting[CurrentBehavior, "Idle"]}
}

; Constant default
member:int MaxTargets()
{
    return ${This.ConfigRef.FindSetting[MaxTargets, ${EVE.MaxLockedTargets}]}
}

; Calculated default
member:int OrbitDistance()
{
    return ${This.ConfigRef.FindSetting[OrbitDistance, ${Math.Calc[${MyShip.MaxTargetRange} * 0.8]}]}
}
```

### Pattern 3: Nested Sets

For complex configuration with sub-categories:

```lavish
objectdef obj_Configuration_Miner
{
    variable settingsetref Ref
    variable settingsetref OreTypesRef

    method Initialize()
    {
        Ref:Set[${BaseConfig.BaseRef.FindSet["Miner"]}]

        ; Check if nested set exists
        if !${This.Ref.FindSet["ORE_Types"](exists)}
        {
            This.Ref:AddSet["ORE_Types"]
        }

        OreTypesRef:Set[${This.Ref.FindSet["ORE_Types"]}]
    }

    method Set_Default_Values()
    {
        ; Main miner settings
        This.Ref:AddSetting[Strip Mine, TRUE]
        This.Ref:AddSetting[Ice Mining, FALSE]
        This.Ref:AddSetting[Cargo Threshold, 11500]

        ; Nested ore types
        This.Ref:AddSet["ORE_Types"]
        OreTypesRef:Set[${This.Ref.FindSet["ORE_Types"]}]

        ; Add each ore type
        This.OreTypesRef:AddSetting["Veldspar", 1]
        This.OreTypesRef:AddSetting["Scordite", 1]
        This.OreTypesRef:AddSetting["Pyroxeres", 1]
        This.OreTypesRef:AddSetting["Plagioclase", 1]
        ; ... etc
    }

    ; Check if ore type enabled
    member:bool OreEnabled(string oreName)
    {
        return ${This.OreTypesRef.FindSetting[${oreName}, 0]}
    }

    ; Enable/disable ore type
    method SetOreEnabled(string oreName, bool enabled)
    {
        This.OreTypesRef:AddSetting[${oreName}, ${If[${enabled}, 1, 0]}]
    }
}
```

### Pattern 4: Config Lists (Collections)

Store lists of items:

```lavish
objectdef obj_Configuration_Targets
{
    method Initialize()
    {
        if !${This.ConfigRef.FindSet["PriorityTargets"](exists)}
        {
            This.ConfigRef:AddSet["PriorityTargets"]
            This:Set_Default_Priority_Targets
        }
    }

    method Set_Default_Priority_Targets()
    {
        variable settingsetref PriorityRef
        PriorityRef:Set[${This.ConfigRef.FindSet["PriorityTargets"]}]

        ; Each setting is a priority target (name, priority)
        PriorityRef:AddSetting["Dire Pithi Arrogator", 10]
        PriorityRef:AddSetting["Guardian Agent", 10]
        PriorityRef:AddSetting["Pithi Saboteur", 8]
        PriorityRef:AddSetting["Frigate", 5]
    }

    ; Get priority for target name
    member:int GetTargetPriority(string targetName)
    {
        return ${This.ConfigRef.FindSet["PriorityTargets"].FindSetting[${targetName}, 0]}
    }

    ; Get all priority targets
    method GetPriorityTargets(index:string targetList)
    {
        variable iterator Setting
        This.ConfigRef.FindSet["PriorityTargets"]:GetSettingIterator[Setting]

        if ${Setting:First(exists)}
        {
            do
            {
                targetList:Insert["${Setting.Key}"]
            }
            while ${Setting:Next(exists)}
        }
    }
}
```

---

## XML File Structure

### EVEBot XML Structure

```xml
<?xml version='1.0' encoding='UTF-8'?>
<!-- Generated by LavishSettings v2 -->
<InnerSpaceSettings>
    <Set Name="Jovehn">                          <!-- Character Name -->
        <Set Name="Common">                      <!-- Config Category -->
            <Setting Name="Home Station">Jita 4-4</Setting>
            <Setting Name="CurrentBehavior">Miner</Setting>
            <Setting Name="AutoLogin">TRUE</Setting>
            <Setting Name="Maximum Runtime">8</Setting>
            <Setting Name="Disable 3D">FALSE</Setting>
        </Set>

        <Set Name="Combat">                      <!-- Combat Category -->
            <Setting Name="MinimumShieldPct">50</Setting>
            <Setting Name="MinimumArmorPct">70</Setting>
            <Setting Name="MinimumCapPct">25</Setting>
            <Setting Name="Launch Combat Drones">TRUE</Setting>
            <Setting Name="OrbitDistance">30000</Setting>
        </Set>

        <Set Name="Miner">                       <!-- Miner Category -->
            <Setting Name="Strip Mine">TRUE</Setting>
            <Setting Name="Cargo Threshold">11500</Setting>
            <Setting Name="Ice Mining">FALSE</Setting>

            <Set Name="ORE_Types">               <!-- Nested Set -->
                <Setting Name="Veldspar">1</Setting>
                <Setting Name="Scordite">1</Setting>
                <Setting Name="Pyroxeres">1</Setting>
                <Setting Name="Plagioclase">1</Setting>
                <Setting Name="Omber">1</Setting>
                <Setting Name="Kernite">1</Setting>
                <!-- ... more ore types ... -->
            </Set>

            <Set Name="ICE_Types">               <!-- Another Nested Set -->
                <Setting Name="Blue Ice">1</Setting>
                <Setting Name="Clear Icicle">1</Setting>
                <Setting Name="Dark Glitter">1</Setting>
                <!-- ... more ice types ... -->
            </Set>
        </Set>

        <Set Name="Fleet">                       <!-- Fleet Category -->
            <Setting Name="FleetMode">TRUE</Setting>
            <Setting Name="MasterName">Hauler Main</Setting>
            <Setting Name="IsMaster">FALSE</Setting>
        </Set>
    </Set>
</InnerSpaceSettings>
```

### File Naming Convention

```
EVEBot:
    Jovehn Config.xml
    Jovehn2 Config.xml
    Miner Alt Config.xml
    Hauler Main Config.xml

Tehbot:
    MyCharacter Config.xml
    Combat Pilot Config.xml
```

**Pattern:** `${Me.Name} Config.xml`

### Multiple Config Files

```
Config/
    ├── Jovehn Config.xml                        ; Character config
    ├── Jovehn Blacklist.xml                     ; Character blacklist
    ├── Jovehn Whitelist.xml                     ; Character whitelist
    ├── Jovehn Mission Cache.xml                 ; Mission data
    └── Launcher_example.xml                     ; Fleet launcher config
```

---

## Config Macros and Helpers

### EVEBot Define_ConfigItem Macro

```lavish
#macro Define_ConfigItem(_Type, _Key, _DefaultValue)
    ; Member to read
    member:_Type _Key()
    {
        return ${This.Ref.FindSetting[_Key, _DefaultValue]}
    }

    ; Method to write
    method _Key(_Type Value)
    {
        This.Ref:AddSetting[_Key, ${Value}]
    }
#endmac
```

**Usage:**

```lavish
objectdef obj_Configuration_Combat
{
    variable settingsetref Ref

    ; Instead of writing member/method pairs manually...
    Define_ConfigItem(int, MinimumShieldPct, 50)
    Define_ConfigItem(int, MinimumArmorPct, 70)
    Define_ConfigItem(int, MinimumCapPct, 25)
    Define_ConfigItem(bool, LaunchCombatDrones, TRUE)
    Define_ConfigItem(int, OrbitDistance, 30000)
}

; Creates:
; member:int MinimumShieldPct() { return ${This.Ref.FindSetting[MinimumShieldPct, 50]} }
; method MinimumShieldPct(int Value) { This.Ref:AddSetting[MinimumShieldPct, ${Value}] }
; ... etc for each config item
```

### Tehbot Setting Macro

```lavish
#define Setting(type, name, setMethod) \
    member:type name() \
    { \
        return ${This.ConfigRef.FindSetting[#name]} \
    } \
    method setMethod(type value) \
    { \
        This.ConfigRef:AddSetting[#name, ${value}] \
    }
```

**Usage:**

```lavish
objectdef obj_Configuration_Common inherits obj_Configuration_Base
{
    Setting(string, Tehbot_Mode, SetTehbot_Mode)
    Setting(bool, AutoStart, SetAutoStart)
    Setting(bool, Disable3D, SetDisable3D)
    Setting(int, LogLevelBar, SetLogLevelBar)
}

; Creates:
; member:string Tehbot_Mode() { return ${This.ConfigRef.FindSetting[Tehbot_Mode]} }
; method SetTehbot_Mode(string value) { This.ConfigRef:AddSetting[Tehbot_Mode, ${value}] }
; ... etc
```

### Config Helper Functions

```lavish
; Check if setting exists
member:bool SettingExists(string settingName)
{
    return ${This.ConfigRef.FindSetting[${settingName}](exists)}
}

; Get setting with fallback
member:string GetSetting(string settingName, string defaultValue)
{
    if ${This:SettingExists[${settingName}]}
    {
        return ${This.ConfigRef.FindSetting[${settingName}]}
    }
    return ${defaultValue}
}

; Ensure setting has value
method EnsureSetting(string settingName, string defaultValue)
{
    if !${This:SettingExists[${settingName}]}
    {
        This.ConfigRef:AddSetting[${settingName}, ${defaultValue}]
    }
}
```

---

## UI Integration

### LavishGUI Config Binding

Config values can be bound to UI elements for real-time editing:

```xml
<!-- EVEBot UI XML -->
<Tab Name="Common" Template='EveTabTemplate'>
    <ComboBox Name='CurrentBehavior' Template='EveComboTemplate'>
        <X>10</X>
        <Y>50</Y>
        <Width>200</Width>
        <Height>25</Height>
        <Items>
            <Item Value='Miner' Text='Miner'/>
            <Item Value='Hauler' Text='Hauler'/>
            <Item Value='Combat' Text='Combat'/>
            <Item Value='Idle' Text='Idle'/>
        </Items>
        <SelectedItem>${Config.Common.CurrentBehavior}</SelectedItem>
        <OnSelect>
            Config.Common:SetCurrentBehavior["${This.SelectedItem.Value}"]
        </OnSelect>
    </ComboBox>

    <CheckBox Name='AutoLogin' Template='EveCheckBoxTemplate'>
        <X>10</X>
        <Y>100</Y>
        <Text>Auto Login</Text>
        <Checked>${Config.Common.AutoLogin}</Checked>
        <OnLeftClick>
            Config.Common:SetAutoLogin[${This.Checked}]
        </OnLeftClick>
    </CheckBox>

    <TextEntry Name='MaxRuntime' Template='EveTextEntryTemplate'>
        <X>10</X>
        <Y>150</Y>
        <Width>100</Width>
        <Height>25</Height>
        <Text>${Config.Common.MaxRuntime}</Text>
        <OnChange>
            Config.Common:SetMaxRuntime[${This.Text}]
        </OnChange>
    </TextEntry>
</Tab>
```

### UI Update Pattern

```lavish
objectdef obj_ConfigUI
{
    method Initialize()
    {
        Event[EVENT_ONFRAME]:AttachAtom[This:UpdateUI]
    }

    method UpdateUI()
    {
        ; Update UI elements with config values
        UIElement[CurrentBehavior@Common]:SetText[${Config.Common.CurrentBehavior}]
        UIElement[MinShield@Combat]:SetText[${Config.Combat.MinimumShieldPct}]
        UIElement[AutoLogin@Common]:SetChecked[${Config.Common.AutoLogin}]
    }

    method OnBehaviorChanged()
    {
        ; Get value from UI
        variable string newBehavior = ${UIElement[CurrentBehavior@Common].SelectedItem.Value}

        ; Update config
        Config.Common:SetCurrentBehavior[${newBehavior}]

        ; Save to disk
        Config:Save

        echo "Behavior changed to: ${newBehavior}"
    }
}
```

### Config Tabs Pattern

Organize config UI into tabs by category:

```xml
<TabControl Name='ConfigTabs'>
    <Tab Name='Common'>
        <!-- General settings: behavior, home station, runtime -->
    </Tab>
    <Tab Name='Combat'>
        <!-- Combat settings: tank thresholds, orbit, drones -->
    </Tab>
    <Tab Name='Mining'>
        <!-- Mining settings: ore types, ice, cargo threshold -->
    </Tab>
    <Tab Name='Hauling'>
        <!-- Hauling settings: routes, autopilot, delivery -->
    </Tab>
    <Tab Name='Fleet'>
        <!-- Fleet settings: master/slave, coordination -->
    </Tab>
</TabControl>
```

---

## Per-Character Settings

### Character-Specific Config Files

**Pattern:** One config file per character

```lavish
objectdef obj_Configuration_BaseConfig
{
    variable string CONFIG_FILE = "${Me.Name} Config.xml"

    method Initialize()
    {
        ; Each character has their own file
        LavishSettings:AddSet[EVEBotSettings]
        LavishSettings[EVEBotSettings]:AddSet[${Me.Name}]

        ; Import character's config
        if ${CONFIG_PATH.FileExists[${CONFIG_FILE}]}
        {
            LavishSettings[EVEBotSettings]:Import[${CONFIG_PATH}/${CONFIG_FILE}]
        }
    }
}
```

**Result:**
```
Config/
    Miner Main Config.xml
    Hauler Alt Config.xml
    Combat Pilot Config.xml
```

### Shared Config Pattern

For settings that should be shared across all characters:

```lavish
objectdef obj_Configuration_Shared
{
    variable string SHARED_CONFIG = "SharedConfig.xml"
    variable settingsetref SharedRef

    method Initialize()
    {
        ; Create separate settings object for shared config
        LavishSettings:AddSet[SharedSettings]

        if ${CONFIG_PATH.FileExists[${SHARED_CONFIG}]}
        {
            LavishSettings[SharedSettings]:Import[${CONFIG_PATH}/${SHARED_CONFIG}]
        }

        SharedRef:Set[${LavishSettings[SharedSettings]}]
    }

    ; Shared settings accessible to all characters
    member:string FleetChannel()
    {
        return ${This.SharedRef.FindSetting[Fleet Channel, "FleetChat"]}
    }

    member:string IRCServer()
    {
        return ${This.SharedRef.FindSetting[IRC Server, "irc.example.com"]}
    }

    member:string MasterName()
    {
        return ${This.SharedRef.FindSetting[Master Name, "FleetCommander"]}
    }
}
```

### Character Roles Pattern

Define character roles in config:

```lavish
objectdef obj_Configuration_Character
{
    method Set_Default_Values()
    {
        This.ConfigRef:AddSetting[CharacterRole, "DPS"]
        This.ConfigRef:AddSetting[CharacterPriority, 1]
        This.ConfigRef:AddSetting[FleetPosition, "DPS"]
    }

    member:string CharacterRole()
    {
        return ${This.ConfigRef.FindSetting[CharacterRole, "DPS"]}
    }

    method SetCharacterRole(string role)
    {
        This.ConfigRef:AddSetting[CharacterRole, ${role}]
    }
}

; Usage in fleet coordination
method AssignFleetRoles()
{
    switch ${Config.Character.CharacterRole}
    {
        case FC
            This:SetupFleetCommander
            break
        case DPS
            This:SetupDPSShip
            break
        case Logi
            This:SetupLogisticsShip
            break
        case Scout
            This:SetupScoutShip
            break
    }
}
```

---

## Config Migration

### Version Tracking

```lavish
objectdef obj_Configuration_BaseConfig
{
    variable float ConfigVersion = 2.1

    method Initialize()
    {
        ; Import existing config
        LavishSettings[EVEBotSettings]:Import[${CONFIG_FILE}]

        ; Check version
        variable float fileVersion = ${BaseRef.FindSetting[ConfigVersion, 1.0]}

        if ${fileVersion} < ${This.ConfigVersion}
        {
            Logger:Log["Config version ${fileVersion} -> ${This.ConfigVersion}, migrating"]
            This:MigrateConfig[${fileVersion}]
        }
    }

    method MigrateConfig(float fromVersion)
    {
        ; Migrate from 1.0 to 2.0
        if ${fromVersion} < 2.0
        {
            This:Migrate_1_0_to_2_0
        }

        ; Migrate from 2.0 to 2.1
        if ${fromVersion} < 2.1
        {
            This:Migrate_2_0_to_2_1
        }

        ; Update version number
        BaseRef:AddSetting[ConfigVersion, ${This.ConfigVersion}]
        This:Save
    }

    method Migrate_1_0_to_2_0()
    {
        Logger:Log["Migrating config 1.0 -> 2.0"]

        ; Rename settings
        variable settingsetref CommonRef = ${BaseRef.FindSet["Common"]}

        if ${CommonRef.FindSetting["Bot Mode Name"](exists)}
        {
            ; Old key: "Bot Mode Name"
            ; New key: "CurrentBehavior"
            variable string oldValue = ${CommonRef.FindSetting["Bot Mode Name"]}
            CommonRef:AddSetting[CurrentBehavior, ${oldValue}]
            CommonRef.FindSetting["Bot Mode Name"]:Remove
            CommonRef.FindSetting["Bot Mode"]:Remove
        }
    }

    method Migrate_2_0_to_2_1()
    {
        Logger:Log["Migrating config 2.0 -> 2.1"]

        ; Add new settings with defaults
        variable settingsetref CombatRef = ${BaseRef.FindSet["Combat"]}

        if !${CombatRef.FindSetting["EnableDroneDefense"](exists)}
        {
            CombatRef:AddSetting[EnableDroneDefense, TRUE]
        }
    }
}
```

### Real Example from EVEBot

```lavish
; From obj_Configuration_Common:
method Initialize()
{
    if ${This.Ref.FindSetting[Bot Mode Name](exists)}
    {
        ; The previous key was present, migrate it to the new one and delete it
        This.Ref:AddSetting[CurrentBehavior, ${This.Ref.FindSetting[Bot Mode Name]}]
        This.Ref.FindSetting[Bot Mode]:Remove
        This.Ref.FindSetting[Bot Mode Name]:Remove
        Logger:Log["Configuration: Migrating config value: Bot Mode Name -> Behavior (${This.Ref.FindSetting[CurrentBehavior]})", LOG_ECHOTOO]
    }
}
```

### Backup Before Migration

```lavish
method MigrateConfig(float fromVersion)
{
    ; Backup config before migration
    variable string backupFile = "${CONFIG_FILE}.v${fromVersion}.backup"
    LavishSettings[EVEBotSettings]:Export[${CONFIG_PATH}/${backupFile}]
    Logger:Log["Config backed up to ${backupFile}"]

    ; Perform migration
    This:Migrate_${fromVersion}_to_${This.ConfigVersion}

    ; Save migrated config
    This:Save
}
```

---

## Fleet Config Synchronization

### Broadcasting Config Changes

Use relay to synchronize config across fleet:

```lavish
objectdef obj_ConfigSync
{
    method Initialize()
    {
        LavishScript:RegisterEvent[Fleet_Config_Update]
        Event[Fleet_Config_Update]:AttachAtom[This:OnConfigUpdate]
    }

    ; Master broadcasts config change
    method BroadcastConfigChange(string category, string setting, string value)
    {
        relay "all other" -event Fleet_Config_Update "${category}" "${setting}" "${value}"
    }

    ; Slaves receive and apply
    atom OnConfigUpdate(string category, string setting, string value)
    {
        echo "Config update: ${category}.${setting} = ${value}"

        switch ${category}
        {
            case Common
                This:UpdateCommonConfig["${setting}", "${value}"]
                break
            case Combat
                This:UpdateCombatConfig["${setting}", "${value}"]
                break
            case Fleet
                This:UpdateFleetConfig["${setting}", "${value}"]
                break
        }
    }

    method UpdateCommonConfig(string setting, string value)
    {
        switch ${setting}
        {
            case CurrentBehavior
                Config.Common:SetCurrentBehavior["${value}"]
                break
            case MaxRuntime
                Config.Common:SetMaxRuntime[${value}]
                break
        }

        Config:Save
    }
}
```

### Master Config Pattern

Fleet master sets config for all slaves:

```lavish
objectdef obj_FleetMaster
{
    ; Set fleet-wide behavior
    method SetFleetBehavior(string behavior)
    {
        ; Set locally
        Config.Common:SetCurrentBehavior[${behavior}]

        ; Broadcast to fleet
        relay all -event Fleet_Config_Update "Common" "CurrentBehavior" "${behavior}"

        echo "Fleet behavior set to: ${behavior}"
    }

    ; Set fleet-wide combat threshold
    method SetFleetMinShield(int shieldPct)
    {
        Config.Combat:SetMinimumShieldPct[${shieldPct}]
        relay all -event Fleet_Config_Update "Combat" "MinimumShieldPct" "${shieldPct}"

        echo "Fleet min shield set to: ${shieldPct}%"
    }

    ; Synchronize all config to fleet
    method SyncConfigToFleet()
    {
        echo "Syncing config to entire fleet..."

        ; Common settings
        relay all -event Fleet_Config_Update "Common" "CurrentBehavior" "${Config.Common.CurrentBehavior}"
        relay all -event Fleet_Config_Update "Common" "MaxRuntime" "${Config.Common.MaxRuntime}"

        ; Combat settings
        relay all -event Fleet_Config_Update "Combat" "MinimumShieldPct" "${Config.Combat.MinimumShieldPct}"
        relay all -event Fleet_Config_Update "Combat" "MinimumArmorPct" "${Config.Combat.MinimumArmorPct}"

        ; Fleet settings
        relay all -event Fleet_Config_Update "Fleet" "MasterName" "${Me.Name}"

        echo "Config sync complete"
    }
}
```

---

## isxSQLite Configuration Storage

### Overview

**isxSQLite** provides an alternative to XML-based configuration using SQLite databases:

**Advantages over XML/LavishSettings:**
- ✅ Better performance for large configs
- ✅ Easier querying with SQL
- ✅ Built-in data validation (column types)
- ✅ Transaction support for atomic updates
- ✅ Can store historical data (versioning)
- ✅ Better for complex data structures

**Disadvantages:**
- ❌ Requires isxSQLite extension
- ❌ More complex setup
- ❌ Not human-editable (binary format)
- ❌ Requires manual database schema design

### Basic Database Config Pattern

```lavish
objectdef obj_ConfigDatabase
{
    variable sqlitedb ConfigDB
    variable string DBPath = "${Script.CurrentDirectory}/config/botconfig.db"

    method Initialize()
    {
        ; Check if isxSQLite loaded
        if !${ISXSQLite(exists)}
        {
            echo "ERROR: isxSQLite not loaded - run: extension isxsqlite"
            return FALSE
        }

        ; Wait for ready
        while !${ISXSQLite.IsReady}
            waitframe

        ; Open/create database
        ConfigDB:Set[${SQLite.OpenDB["BotConfig", "${This.DBPath}"]}]

        if !${ConfigDB.ID(exists)}
        {
            echo "ERROR: Failed to open config database"
            return FALSE
        }

        ; Initialize schema
        This:InitializeSchema

        echo "Config database ready: ${This.DBPath}"
        return TRUE
    }

    method InitializeSchema()
    {
        ; Create settings table if doesn't exist
        if !${ConfigDB.TableExists["settings"]}
        {
            ConfigDB:ExecDML["CREATE TABLE settings (
                category TEXT NOT NULL,
                key TEXT NOT NULL,
                value TEXT,
                type TEXT,
                PRIMARY KEY (category, key)
            );"]

            ; Insert defaults
            This:LoadDefaults
        }
    }

    method LoadDefaults()
    {
        ; Use transaction for bulk insert
        variable index:string DML

        DML:Insert["INSERT OR REPLACE INTO settings VALUES ('General', 'BotMode', 'Idle', 'string');"]
        DML:Insert["INSERT OR REPLACE INTO settings VALUES ('General', 'HomeStation', 'Jita 4-4', 'string');"]
        DML:Insert["INSERT OR REPLACE INTO settings VALUES ('General', 'AutoStart', 'FALSE', 'bool');"]

        DML:Insert["INSERT OR REPLACE INTO settings VALUES ('Combat', 'MinShield', '50', 'int');"]
        DML:Insert["INSERT OR REPLACE INTO settings VALUES ('Combat', 'MinArmor', '70', 'int');"]
        DML:Insert["INSERT OR REPLACE INTO settings VALUES ('Combat', 'MinCap', '25', 'int');"]

        ConfigDB:ExecDMLTransaction[DML]
    }

    method Close()
    {
        if ${ConfigDB.ID(exists)}
            ConfigDB:Close
    }

    ; Get setting value
    member:string Get(string category, string key)
    {
        ; Escape inputs to prevent SQL injection
        variable string SafeCategory = "${SQLite.Escape_String[${category}]}"
        variable string SafeKey = "${SQLite.Escape_String[${key}]}"

        variable sqlitequery Query
        Query:Set[${ConfigDB.ExecQuery["SELECT value FROM settings WHERE category='${SafeCategory}' AND key='${SafeKey}'"]}]

        variable string Value = ""
        if ${Query.NumRows} > 0
            Value:Set["${Query.GetFieldValue[1]}"]

        Query:Finalize
        return "${Value}"
    }

    member:int GetInt(string category, string key)
    {
        variable string val = "${This.Get[${category}, ${key}]}"
        if ${val.Length} > 0
            return ${val}
        return 0
    }

    member:bool GetBool(string category, string key)
    {
        variable string val = "${This.Get[${category}, ${key}]}"
        return ${val.Equal["TRUE"]}
    }

    ; Set setting value
    method Set(string category, string key, string value, string type)
    {
        variable string SafeCategory = "${SQLite.Escape_String[${category}]}"
        variable string SafeKey = "${SQLite.Escape_String[${key}]}"
        variable string SafeValue = "${SQLite.Escape_String[${value}]}"

        ConfigDB:ExecDML["INSERT OR REPLACE INTO settings VALUES ('${SafeCategory}', '${SafeKey}', '${SafeValue}', '${type}');"]
    }
}
```

### Config Object with Database Backend

```lavish
objectdef obj_Config_General
{
    ; Get values
    member:string BotMode()
    {
        variable string val = "${DBConfig.Get["General", "BotMode"]}"
        if ${val.Length} == 0
            return "Idle"
        return "${val}"
    }

    member:string HomeStation()
    {
        variable string val = "${DBConfig.Get["General", "HomeStation"]}"
        if ${val.Length} == 0
            return "Jita 4-4"
        return "${val}"
    }

    member:bool AutoStart()
    {
        return ${DBConfig.GetBool["General", "AutoStart"]}
    }

    ; Set values
    method SetBotMode(string mode)
    {
        DBConfig:Set["General", "BotMode", "${mode}", "string"]
    }

    method SetHomeStation(string station)
    {
        DBConfig:Set["General", "HomeStation", "${station}", "string"]
    }

    method SetAutoStart(bool value)
    {
        DBConfig:Set["General", "AutoStart", "${If[${value}, TRUE, FALSE]}", "bool"]
    }
}

objectdef obj_Config_Combat
{
    member:int MinShield()
    {
        variable int val = ${DBConfig.GetInt["Combat", "MinShield"]}
        if ${val} == 0
            return 50
        return ${val}
    }

    method SetMinShield(int value)
    {
        DBConfig:Set["Combat", "MinShield", "${value}", "int"]
    }

    ; ... similar for other combat settings
}

; Main config object
objectdef obj_Config
{
    variable obj_Config_General General
    variable obj_Config_Combat Combat
}

; Global instances
variable(global) obj_ConfigDatabase DBConfig
variable(global) obj_Config Config

; Usage
DBConfig:Initialize
echo "Bot Mode: ${Config.General.BotMode}"
Config.General:SetBotMode["Mining"]
DBConfig:Close
```

### Hybrid Approach: XML + Database

Use XML for user-editable settings, database for runtime/computed data:

```lavish
objectdef obj_HybridConfig
{
    ; User settings (XML)
    variable obj_ConfigManager XMLConfig

    ; Runtime data (Database)
    variable obj_ConfigDatabase DBConfig

    method Initialize()
    {
        ; Initialize both
        XMLConfig:Initialize
        DBConfig:Initialize

        ; Store runtime info in database
        This:UpdateRuntimeInfo
    }

    method UpdateRuntimeInfo()
    {
        ; Track session statistics
        DBConfig:Set["Runtime", "LastStartTime", "${Time.Timestamp}", "int64"]
        DBConfig:Set["Runtime", "CharacterName", "${Me.Name}", "string"]
        DBConfig:Set["Runtime", "Version", "4.5.2", "string"]
    }

    ; User preferences from XML
    member:string BotMode()
    {
        return ${XMLConfig.General.BotMode}
    }

    ; Runtime stats from database
    member:int64 LastStartTime()
    {
        return ${DBConfig.GetInt["Runtime", "LastStartTime"]}
    }
}
```

### Config with History/Versioning

Track config changes over time:

```lavish
objectdef obj_ConfigWithHistory
{
    variable sqlitedb ConfigDB

    method InitializeSchema()
    {
        ; Current config table
        ConfigDB:ExecDML["CREATE TABLE IF NOT EXISTS settings (
            category TEXT NOT NULL,
            key TEXT NOT NULL,
            value TEXT,
            updated_at INTEGER,
            PRIMARY KEY (category, key)
        );"]

        ; History table
        ConfigDB:ExecDML["CREATE TABLE IF NOT EXISTS settings_history (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            category TEXT,
            key TEXT,
            old_value TEXT,
            new_value TEXT,
            changed_at INTEGER,
            changed_by TEXT
        );"]
    }

    method Set(string category, string key, string newValue)
    {
        ; Get old value
        variable string oldValue = "${This.Get[${category}, ${key}]}"

        ; Update current value
        variable string SafeCat = "${SQLite.Escape_String[${category}]}"
        variable string SafeKey = "${SQLite.Escape_String[${key}]}"
        variable string SafeVal = "${SQLite.Escape_String[${newValue}]}"

        ConfigDB:ExecDML["INSERT OR REPLACE INTO settings VALUES (
            '${SafeCat}',
            '${SafeKey}',
            '${SafeVal}',
            ${Time.Timestamp}
        );"]

        ; Record history
        variable string SafeOld = "${SQLite.Escape_String[${oldValue}]}"
        variable string SafeName = "${SQLite.Escape_String[${Me.Name}]}"

        ConfigDB:ExecDML["INSERT INTO settings_history (category, key, old_value, new_value, changed_at, changed_by) VALUES (
            '${SafeCat}',
            '${SafeKey}',
            '${SafeOld}',
            '${SafeVal}',
            ${Time.Timestamp},
            '${SafeName}'
        );"]
    }

    ; Get config history for a setting
    method GetHistory(string category, string key)
    {
        variable sqlitequery Query
        Query:Set[${ConfigDB.ExecQuery["SELECT old_value, new_value, changed_at, changed_by
            FROM settings_history
            WHERE category='${SQLite.Escape_String[${category}]}' AND key='${SQLite.Escape_String[${key}]}'
            ORDER BY changed_at DESC
            LIMIT 10"]}]

        echo "History for ${category}.${key}:"
        if ${Query.NumRows} > 0
        {
            do
            {
                echo "  ${Query.GetFieldValue[4]} changed '${Query.GetFieldValue[1]}' -> '${Query.GetFieldValue[2]}' at ${Query.GetFieldValue[3]}"
                Query:NextRow
            }
            while !${Query.LastRow}
        }

        Query:Finalize
    }
}
```

### Performance Comparison

**XML (LavishSettings):**
```lavish
; Read: ~100 settings in ~5ms
; Write: ~100 settings in ~50ms (export entire file)
; Good for: < 1000 settings, human-editable
```

**Database (isxSQLite):**
```lavish
; Read: ~100 settings in ~2ms (query)
; Write: ~100 settings in ~1ms (transaction)
; Good for: > 1000 settings, complex queries, history
```

### When to Use isxSQLite Config

**Use isxSQLite when:**
- Storing large amounts of configuration data (> 500 settings)
- Need complex queries (e.g., "all ore types with priority > 5")
- Want configuration history/audit trail
- Sharing config data between multiple scripts
- Need transactional updates
- Storing runtime statistics alongside config

**Use XML/LavishSettings when:**
- Simple, small configuration files
- Need human-editability
- Following existing bot patterns (EVEBot, Tehbot)
- Don't want extra dependency on isxSQLite

---

## Best Practices

### 1. Always Provide Defaults

```lavish
; GOOD - has default
member:int MinShield()
{
    return ${This.ConfigRef.FindSetting[MinShield, 50]}
}

; BAD - no default, crashes if missing
member:int MinShield()
{
    return ${This.ConfigRef.FindSetting[MinShield]}
}
```

### 2. Validate Config Values

```lavish
method SetMinimumShieldPct(int value)
{
    ; Validate range
    if ${value} < 0 || ${value} > 100
    {
        Logger:Log["ERROR: Invalid shield pct ${value}, must be 0-100"]
        return
    }

    This.ConfigRef:AddSetting[MinimumShieldPct, ${value}]
}

method SetOrbitDistance(int distance)
{
    ; Validate against ship capabilities
    if ${distance} > ${MyShip.MaxTargetRange}
    {
        Logger:Log["WARNING: Orbit distance ${distance}m exceeds targeting range ${MyShip.MaxTargetRange}m"]
        distance:Set[${MyShip.MaxTargetRange}]
    }

    This.ConfigRef:AddSetting[OrbitDistance, ${distance}]
}
```

### 3. Save After Critical Changes

```lavish
method SetCurrentBehavior(string behavior)
{
    ; Update config
    This.ConfigRef:AddSetting[CurrentBehavior, ${behavior}]

    ; Save immediately for critical setting
    BaseConfig:Save

    echo "Behavior changed to ${behavior} and saved"
}
```

### 4. Use Inheritance for Related Configs

```lavish
; Base behavior config
objectdef obj_Configuration_BehaviorBase inherits obj_Configuration_Base
{
    method Set_Default_Values()
    {
        ; Common to all behaviors
        This.ConfigRef:AddSetting[Enabled, TRUE]
        This.ConfigRef:AddSetting[Priority, 1]
    }

    Setting(bool, Enabled, SetEnabled)
    Setting(int, Priority, SetPriority)
}

; Specific behavior configs inherit common settings
objectdef obj_Configuration_Miner inherits obj_Configuration_BehaviorBase
{
    method Initialize()
    {
        This[parent]:Initialize["Miner"]
        ; Miner-specific init
    }

    method Set_Default_Values()
    {
        ; Call parent for common settings
        This[parent]:Set_Default_Values

        ; Add miner-specific settings
        This.ConfigRef:AddSetting[StripMine, TRUE]
        This.ConfigRef:AddSetting[CargoThreshold, 11500]
    }
}
```

### 5. Encrypt Sensitive Data

```lavish
; WARNING: LavishScript has limited encryption

member:string LoginPassword()
{
    ; Stored in plain text in XML - not secure!
    return ${This.ConfigRef.FindSetting[Login Password, ""]}
}

; Better: Use InnerSpace's secure storage
member:string LoginPassword()
{
    ; Store in InnerSpace settings, not script config
    return ${LavishScript.Settings.FindSetting[SecurePassword, ""]}
}

; Or base64 encode (obscurity, not security)
method SetLoginPassword(string password)
{
    variable string encoded = ${password.Base64Encode}
    This.ConfigRef:AddSetting[Login Password, ${encoded}]
}

member:string LoginPassword()
{
    variable string encoded = ${This.ConfigRef.FindSetting[Login Password, ""]}
    return ${encoded.Base64Decode}
}
```

### 6. Group Related Settings

```lavish
; GOOD - organized into sets
<Set Name="Combat">
    <Set Name="Tank">
        <Setting Name="MinShield">50</Setting>
        <Setting Name="MinArmor">70</Setting>
        <Setting Name="MinCap">25</Setting>
    </Set>
    <Set Name="Movement">
        <Setting Name="OrbitDistance">30000</Setting>
        <Setting Name="KeepAtRange">FALSE</Setting>
    </Set>
</Set>

; Access
variable int minShield = ${Config.Combat.Tank.MinShield}
variable int orbitDist = ${Config.Combat.Movement.OrbitDistance}
```

### 7. Document Config Files

```xml
<?xml version='1.0' encoding='UTF-8'?>
<!--
    EVEBot Configuration for: Jovehn
    Last Modified: 2024-01-15
    Bot -->
<InnerSpaceSettings>
    <Set Name="Jovehn">
        <!-- Common Settings -->
        <Set Name="Common">
            <!-- Current active behavior: Miner, Hauler, Combat, Idle -->
            <Setting Name="CurrentBehavior">Miner</Setting>

            <!-- Maximum runtime in hours (0 = unlimited) -->
            <Setting Name="Maximum Runtime">8</Setting>
        </Set>

        <!-- Combat Settings -->
        <Set Name="Combat">
            <!-- Shield threshold to flee (0-100) -->
            <Setting Name="MinimumShieldPct">50</Setting>
        </Set>
    </Set>
</InnerSpaceSettings>
```

---

## Complete Examples

### Example 1: Complete Config System

```lavish
/* ========== CONFIG MANAGER ========== */
objectdef obj_MyBot_ConfigManager
{
    variable string CONFIG_FILE = "${Me.Name} Config.xml"
    variable filepath CONFIG_PATH = "${Script.CurrentDirectory}/config"
    variable settingsetref ConfigRoot
    variable float ConfigVersion = 1.5

    method Initialize()
    {
        ; Setup LavishSettings
        LavishSettings[MyBotSettings]:Remove
        LavishSettings:AddSet[MyBotSettings]
        LavishSettings[MyBotSettings]:AddSet[${Me.Name}]

        ; Import existing config
        if ${CONFIG_PATH.FileExists[${CONFIG_FILE}]}
        {
            echo "Loading config: ${CONFIG_FILE}"
            LavishSettings[MyBotSettings]:Import[${CONFIG_PATH}/${CONFIG_FILE}]
        }
        else
        {
            echo "Creating new config: ${CONFIG_FILE}"
        }

        ; Set root reference
        ConfigRoot:Set[${LavishSettings[MyBotSettings].FindSet[${Me.Name}]}]

        ; Check version and migrate if needed
        variable float fileVersion = ${ConfigRoot.FindSetting[ConfigVersion, 1.0]}
        if ${fileVersion} < ${This.ConfigVersion}
        {
            This:MigrateConfig[${fileVersion}]
        }

        ; Update version
        ConfigRoot:AddSetting[ConfigVersion, ${This.ConfigVersion}]
    }

    method Shutdown()
    {
        This:Save
        LavishSettings[MyBotSettings]:Remove
    }

    method Save()
    {
        echo "Saving config to ${CONFIG_FILE}"
        LavishSettings[MyBotSettings]:Export[${CONFIG_PATH}/${CONFIG_FILE}]
    }

    method MigrateConfig(float fromVersion)
    {
        echo "Migrating config from v${fromVersion} to v${This.ConfigVersion}"

        ; Backup
        variable string backup = "${CONFIG_FILE}.v${fromVersion}.backup"
        LavishSettings[MyBotSettings]:Export[${CONFIG_PATH}/${backup}]

        ; Migration logic here
        if ${fromVersion} < 1.5
        {
            ; Add new settings, rename old ones, etc.
        }
    }
}

/* ========== BASE CONFIG CLASS ========== */
objectdef obj_MyBot_ConfigBase
{
    variable string SetName = ""

    method Initialize(string name)
    {
        SetName:Set[${name}]

        if !${ConfigManager.ConfigRoot.FindSet[${This.SetName}](exists)}
        {
            echo "Creating config set: ${This.SetName}"
            ConfigManager.ConfigRoot:AddSet[${This.SetName}]
            This:Set_Default_Values
        }
    }

    member:settingsetref ConfigRef()
    {
        return ${ConfigManager.ConfigRoot.FindSet[${This.SetName}]}
    }

    method Set_Default_Values()
    {
        ; Override in derived classes
    }
}

/* ========== GENERAL CONFIG ========== */
objectdef obj_MyBot_Config_General inherits obj_MyBot_ConfigBase
{
    method Initialize()
    {
        This[parent]:Initialize["General"]
    }

    method Set_Default_Values()
    {
        This.ConfigRef:AddSetting[BotMode, "Idle"]
        This.ConfigRef:AddSetting[HomeStation, "Jita 4-4"]
        This.ConfigRef:AddSetting[AutoStart, FALSE]
        This.ConfigRef:AddSetting[MaxRuntime, 0]
    }

    member:string BotMode()
    {
        return ${This.ConfigRef.FindSetting[BotMode, "Idle"]}
    }

    method SetBotMode(string mode)
    {
        This.ConfigRef:AddSetting[BotMode, ${mode}]
        ConfigManager:Save
    }

    member:string HomeStation()
    {
        return ${This.ConfigRef.FindSetting[HomeStation, "Jita 4-4"]}
    }

    method SetHomeStation(string station)
    {
        This.ConfigRef:AddSetting[HomeStation, ${station}]
    }

    member:bool AutoStart()
    {
        return ${This.ConfigRef.FindSetting[AutoStart, FALSE]}
    }

    method SetAutoStart(bool value)
    {
        This.ConfigRef:AddSetting[AutoStart, ${value}]
    }

    member:int MaxRuntime()
    {
        return ${This.ConfigRef.FindSetting[MaxRuntime, 0]}
    }

    method SetMaxRuntime(int hours)
    {
        if ${hours} < 0
        {
            echo "ERROR: Invalid runtime ${hours}"
            return
        }
        This.ConfigRef:AddSetting[MaxRuntime, ${hours}]
    }
}

/* ========== COMBAT CONFIG ========== */
objectdef obj_MyBot_Config_Combat inherits obj_MyBot_ConfigBase
{
    method Initialize()
    {
        This[parent]:Initialize["Combat"]
    }

    method Set_Default_Values()
    {
        This.ConfigRef:AddSetting[MinShield, 50]
        This.ConfigRef:AddSetting[MinArmor, 70]
        This.ConfigRef:AddSetting[MinCap, 25]
        This.ConfigRef:AddSetting[OrbitDistance, 30000]
        This.ConfigRef:AddSetting[UseDrones, TRUE]
    }

    member:int MinShield()
    {
        return ${This.ConfigRef.FindSetting[MinShield, 50]}
    }

    method SetMinShield(int value)
    {
        if ${value} < 0 || ${value} > 100
        {
            echo "ERROR: Shield pct must be 0-100"
            return
        }
        This.ConfigRef:AddSetting[MinShield, ${value}]
    }

    member:int MinArmor()
    {
        return ${This.ConfigRef.FindSetting[MinArmor, 70]}
    }

    method SetMinArmor(int value)
    {
        if ${value} < 0 || ${value} > 100
        {
            echo "ERROR: Armor pct must be 0-100"
            return
        }
        This.ConfigRef:AddSetting[MinArmor, ${value}]
    }

    member:int MinCap()
    {
        return ${This.ConfigRef.FindSetting[MinCap, 25]}
    }

    method SetMinCap(int value)
    {
        if ${value} < 0 || ${value} > 100
        {
            echo "ERROR: Cap pct must be 0-100"
            return
        }
        This.ConfigRef:AddSetting[MinCap, ${value}]
    }

    member:int OrbitDistance()
    {
        return ${This.ConfigRef.FindSetting[OrbitDistance, 30000]}
    }

    method SetOrbitDistance(int distance)
    {
        This.ConfigRef:AddSetting[OrbitDistance, ${distance}]
    }

    member:bool UseDrones()
    {
        return ${This.ConfigRef.FindSetting[UseDrones, TRUE]}
    }

    method SetUseDrones(bool value)
    {
        This.ConfigRef:AddSetting[UseDrones, ${value}]
    }
}

/* ========== MAIN CONFIG OBJECT ========== */
objectdef obj_MyBot_Config
{
    variable obj_MyBot_Config_General General
    variable obj_MyBot_Config_Combat Combat

    method Save()
    {
        ConfigManager:Save
    }
}

/* ========== USAGE ========== */
variable(global) obj_MyBot_ConfigManager ConfigManager
variable(global) obj_MyBot_Config Config

function main()
{
    ; Initialize config system
    ConfigManager:Initialize

    ; Access config
    echo "Bot Mode: ${Config.General.BotMode}"
    echo "Home Station: ${Config.General.HomeStation}"
    echo "Min Shield: ${Config.Combat.MinShield}%"

    ; Change config
    Config.General:SetBotMode["Combat"]
    Config.Combat:SetMinShield[60]

    ; Save
    Config:Save

    ; Shutdown
    ConfigManager:Shutdown
}
```

### Example 2: UI-Driven Config

```lavish
/* ========== CONFIG UI ========== */
objectdef obj_ConfigUI
{
    method Initialize()
    {
        This:CreateUI
    }

    method CreateUI()
    {
        ; Create config window
        ui -load config_ui.xml
    }

    ; Called when behavior dropdown changes
    method OnBehaviorChanged()
    {
        variable string newBehavior = ${UIElement[BehaviorCombo].SelectedItem.Value}
        Config.General:SetBotMode[${newBehavior}]
        echo "Behavior changed to: ${newBehavior}"
    }

    ; Called when min shield slider changes
    method OnMinShieldChanged()
    {
        variable int newValue = ${UIElement[MinShieldSlider].Value}
        Config.Combat:SetMinShield[${newValue}]
        UIElement[MinShieldText]:SetText["${newValue}%"]
    }

    ; Called when Save button clicked
    method OnSaveClicked()
    {
        Config:Save
        echo "Configuration saved!"
        UIElement[SaveButton]:SetText["Saved!"]
        wait 20
        UIElement[SaveButton]:SetText["Save Config"]
    }

    ; Update UI from config (called periodically)
    method UpdateUIFromConfig()
    {
        UIElement[BehaviorCombo]:SelectItem[${Config.General.BotMode}]
        UIElement[MinShieldSlider]:SetValue[${Config.Combat.MinShield}]
        UIElement[MinShieldText]:SetText["${Config.Combat.MinShield}%"]
        UIElement[AutoStartCheckbox]:SetChecked[${Config.General.AutoStart}]
    }
}
```

**config_ui.xml:**

```xml
<?xml version='1.0' encoding='UTF-8'?>
<LGUI2>
    <window name='ConfigWindow' width='400' height='600'>
        <label>Bot Configuration</label>

        <!-- Behavior Selection -->
        <label y='50'>Behavior:</label>
        <combobox name='BehaviorCombo' x='100' y='50' width='200'>
            <item value='Idle'>Idle</item>
            <item value='Mining'>Mining</item>
            <item value='Combat'>Combat</item>
            <item value='Hauling'>Hauling</item>
            <onselect>ConfigUI:OnBehaviorChanged</onselect>
        </combobox>

        <!-- Min Shield Slider -->
        <label y='100'>Min Shield %:</label>
        <slider name='MinShieldSlider' x='100' y='100' width='200' min='0' max='100'>
            <onchange>ConfigUI:OnMinShieldChanged</onchange>
        </slider>
        <label name='MinShieldText' x='310' y='100'>50%</label>

        <!-- Auto Start Checkbox -->
        <checkbox name='AutoStartCheckbox' x='10' y='150'>
            <label>Auto Start</label>
            <onclick>Config.General:SetAutoStart[${This.Checked}]</onclick>
        </checkbox>

        <!-- Save Button -->
        <button name='SaveButton' x='150' y='550' width='100'>
            <label>Save Config</label>
            <onclick>ConfigUI:OnSaveClicked</onclick>
        </button>
    </window>
</LGUI2>
```

### Example 3: Fleet Config Sync

```lavish
/* ========== FLEET CONFIG SYNC ========== */
objectdef obj_FleetConfigSync
{
    variable bool IsMaster = FALSE
    variable string MasterName

    method Initialize()
    {
        This.IsMaster:Set[${Config.Fleet.IsMaster}]
        This.MasterName:Set["${Config.Fleet.MasterName}"]

        ; Register events
        LavishScript:RegisterEvent[Fleet_Config_Sync]
        Event[Fleet_Config_Sync]:AttachAtom[This:OnConfigSync]
    }

    ; Master: Sync config to all slaves
    method SyncToFleet()
    {
        if !${This.IsMaster}
        {
            echo "ERROR: Only master can sync config"
            return
        }

        echo "Syncing config to fleet..."

        ; Build config string: category|setting|value
        variable string configData = ""

        ; General
        configData:Concat["General|BotMode|${Config.General.BotMode};"]
        configData:Concat["General|MaxRuntime|${Config.General.MaxRuntime};"]

        ; Combat
        configData:Concat["Combat|MinShield|${Config.Combat.MinShield};"]
        configData:Concat["Combat|MinArmor|${Config.Combat.MinArmor};"]
        configData:Concat["Combat|OrbitDistance|${Config.Combat.OrbitDistance};"]

        ; Broadcast
        relay "all other" -event Fleet_Config_Sync "${configData}"

        echo "Config sync complete"
    }

    ; Slave: Receive and apply config
    atom OnConfigSync(string configData)
    {
        if ${This.IsMaster}
            return

        echo "Receiving config from master..."

        ; Parse config string
        variable int i = 1
        variable string configItem

        while ${configData.Token[${i}, ";"](exists)}
        {
            configItem:Set["${configData.Token[${i}, ";"]}"]

            variable string category = ${configItem.Token[1, "|"]}
            variable string setting = ${configItem.Token[2, "|"]}
            variable string value = ${configItem.Token[3, "|"]}

            This:ApplyConfigItem["${category}", "${setting}", "${value}"]

            i:Inc
        }

        ; Save applied config
        Config:Save

        echo "Config sync applied"
    }

    method ApplyConfigItem(string category, string setting, string value)
    {
        switch ${category}
        {
            case General
                switch ${setting}
                {
                    case BotMode
                        Config.General:SetBotMode["${value}"]
                        break
                    case MaxRuntime
                        Config.General:SetMaxRuntime[${value}]
                        break
                }
                break

            case Combat
                switch ${setting}
                {
                    case MinShield
                        Config.Combat:SetMinShield[${value}]
                        break
                    case MinArmor
                        Config.Combat:SetMinArmor[${value}]
                        break
                    case OrbitDistance
                        Config.Combat:SetOrbitDistance[${value}]
                        break
                }
                break
        }

        echo "Applied: ${category}.${setting} = ${value}"
    }
}
```

---

## Summary

### Key Takeaways

1. **LavishSettings Foundation**
   - Hierarchical Sets and Settings
   - Import/Export XML
   - settingsetref for efficient access

2. **Config Architecture Patterns**
   - BaseConfig for file I/O
   - Category-based sub-configs
   - Member/method pairs for access

3. **Configuration Best Practices**
   - Always provide defaults
   - Validate values
   - Version and migrate
   - Per-character files

4. **UI Integration**
   - Bind config to UI elements
   - Real-time updates
   - Save on critical changes

5. **Fleet Coordination**
   - Broadcast config changes via relay
   - Master syncs to slaves
   - Validate before applying
