# File 24: Multi-Boxing and Fleet Coordination

**Layer 5: Practical Implementations**

---

## Table of Contents
1. [Introduction to Multi-Boxing](#introduction-to-multi-boxing)
2. [LavishScript Relay System](#lavishscript-relay-system)
3. [Fleet Management](#fleet-management)
4. [Character Coordination Patterns](#character-coordination-patterns)
5. [Master-Slave Architectures](#master-slave-architectures)
6. [Resource Sharing Patterns](#resource-sharing-patterns)
7. [Fleet Combat Coordination](#fleet-combat-coordination)
8. [Mining Fleet Coordination](#mining-fleet-coordination)
9. [Event-Driven Coordination](#event-driven-coordination)
10. [Complete Working Examples](#complete-working-examples)
11. [Common Problems](#common-problems)

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

> **📡 Alternative: IRC Coordination:** For multi-computer coordination or remote monitoring, IRC can be used alongside or instead of relay/uplink. See File 28 (Relay_System_and_IPC.md) IRC Bridge Integration section and `__CRITICAL_NEWEST_ISXIM_Reference.md` for IRC-based fleet coordination patterns.

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

## Summary

This file covered:

1. **LavishScript Relay System**: Inter-session communication, event broadcasting, Uplink networking
2. **Fleet Management**: Invite/accept patterns, CharID resolution, fleet warp capability
3. **Character Coordination**: Synchronized actions, position reporting, status sharing
4. **Master-Slave Architectures**: Centralized control, autonomous with coordination, leader-follower
5. **Resource Sharing**: Bookmark sharing, target coordination, cargo capacity tracking
6. **Fleet Combat**: Primary calling, logistics reps, tackle coordination
7. **Mining Fleet**: Orca-centric operations, boosting, belt depletion
8. **Event-Driven Coordination**: Custom event registry, state change events
9. **Working Examples**: Mining fleet (hauler + miners), combat fleet with FC
10. **Common Problems**: Relay debugging, session discovery, race conditions, Uplink issues

**Key Takeaway**: Multi-boxing in EVE requires **robust event-driven architecture** with **defensive programming** - always handle edge cases like disconnections, missing fleet members, and event delivery failures.

**This completes Layer 5: Practical Implementations!** The next layer (Layer 6) will cover Advanced Topics including performance optimization, memory management, and large-scale botting operations.

---

**File Statistics**:
- **Lines**: ~2400
- **Code Examples**: 30+
- **Complete Implementations**: 2
- **EVEBot Patterns Referenced**: obj_Fleet.iss, relay patterns from multiple files

**Project Milestone**: Layer 5 Complete (60% of total project) 🎉
