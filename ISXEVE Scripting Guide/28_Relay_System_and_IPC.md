# File 28: Relay System and Inter-Process Communication (IPC)

**Layer 7: Advanced Topics - Part 1 of 4**

## Table of Contents
1. [LavishScript Relay Basics](#relay-basics)
2. [Event System Foundation](#event-system)
3. [Basic Relay Patterns](#basic-patterns)
4. [UplinkManager System](#uplink-system)
5. [Fleet Coordination Patterns](#fleet-patterns)
6. [IRC Bridge Integration](#irc-bridge)
7. [Uplink Networking](#uplink-networking)
8. [Performance Considerations](#performance)
9. [Best Practices](#best-practices)
10. [Complete Working Examples](#examples)

---

## LavishScript Relay Basics <a name="relay-basics"></a>

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

## Event System Foundation <a name="event-system"></a>

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

## Basic Relay Patterns <a name="basic-patterns"></a>

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

## UplinkManager System <a name="uplink-system"></a>

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

## Fleet Coordination Patterns <a name="fleet-patterns"></a>

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

## IRC Bridge Integration <a name="irc-bridge"></a>

> **📚 Complete ISXIM Documentation:** See `__CRITICAL_NEWEST_ISXIM_Reference.md` for full IRC extension reference including all TLOs, datatypes, events, and usage patterns.

### Overview (Tehbot ChatRelay)

Tehbot's **ChatRelay** bridges EVE sessions with IRC for:
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

## Uplink Networking <a name="uplink-networking"></a>

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

## Performance Considerations <a name="performance"></a>

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

## Best Practices <a name="best-practices"></a>

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

## Complete Working Examples <a name="examples"></a>

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

### What's Next?

**File 29:** Configuration and Settings Management
- Config file architecture
- XML config parsing
- UI-driven configuration
- Behavior switching
- Per-character settings
- Fleet-wide config sync

### Navigation

[← Previous: File 27 - Tehbot Combat Analysis](27_Tehbot_Combat_Analysis.md) | [Next: File 29 - Configuration Management →](29_Configuration_and_Settings_Management.md)

---

*Layer 7 Progress: 1/4 Complete (25%)*
*Total Documentation Progress: 27/38 Files (71%)*
