# ISXEVE Core Objects Reference
## Complete Guide to Top-Level Objects (TLOs) and Fundamental ISXEVE Objects

**Part of**: EVE Online Bot Development - AI-Level Documentation
**Layer**: 3 - ISXEVE API Deep Dive
**Prerequisites**: Read files 01-08 (Foundation + Scripting Fundamentals)
**Critical For**: ALL bot scripts - these objects are used constantly

---

## Purpose of This Document

This file provides **exhaustive coverage** of:
- Top-Level Objects (TLOs) concept and architecture
- ${Me} object - Character/pilot information and capabilities
- ${MyShip} object - Current ship state, modules, cargo, defenses
- ${EVE} object - Universe/game system, UI commands, timing
- Other critical TLOs (${Local}, ${Station}, ${Bookmark}, ${Config}, etc.)
- Common usage patterns and best practices
- Real-world examples from Evebot, Yamfa, and Tehbot
- Critical gotchas and timing issues

**Why This Is CRITICAL**:
- **Every script uses these objects** in almost every line of code
- Understanding TLOs is **foundational** to all ISXEVE scripting
- ${Me} and ${MyShip} are the **two most important objects** in ISXEVE
- Incorrect usage = script crashes, failed actions, stuck bots

---

## Table of Contents

1. [Top-Level Objects (TLOs) Overview](#top-level-objects-tlos-overview)
2. [Me Object](#me-object)
3. [MyShip Object](#myship-object)
4. [EVE Object](#eve-object)
5. [Local Object](#local-object)
6. [Station Object](#station-object)
7. [Bookmark Object](#bookmark-object)
8. [Config Object](#config-object)
9. [Other Important TLOs](#other-important-tlos)
10. [Common Usage Patterns](#common-usage-patterns)
11. [Object Relationships](#object-relationships)
12. [Performance Considerations](#performance-considerations)
13. [Critical Gotchas](#critical-gotchas)

---

## Top-Level Objects (TLOs) Overview

### What are TLOs?

**Top-Level Objects** are globally-accessible objects provided by ISXEVE that represent core game systems and state.

**Characteristics**:
- Accessed with `${}` syntax: `${Me}`, `${MyShip}`, `${EVE}`
- Always available (don't need to be declared or initialized)
- Provided by ISXEVE extension (not LavishScript core)
- Represent current game state (updated by ISXEVE's memory reading)
- **Read-only** - cannot modify game state directly, only query it

**Key TLOs**:
```lavish
${Me}           ; Your character/pilot
${MyShip}       ; Your current ship
${EVE}          ; Game universe/UI system
${Local}        ; Local chat channel
${Station}      ; Current station (if docked)
${Config}       ; ISXEVE configuration
${ISXEVE}       ; ISXEVE extension info
```

### TLO vs Regular Objects

**TLO** (Top-Level Object):
```lavish
; No declaration needed
echo "${Me.Name}"    ; Always works (if ISXEVE loaded)
```

**Regular Object** (must declare):
```lavish
; Must declare first
variable entity target = ${Entity[123456]}
echo "${target.Name}"
```

### TLO Existence Checking

**Most TLOs always exist** (if ISXEVE loaded):
```lavish
; ${Me} always exists
echo "${Me.Name}"    ; Safe

; ${MyShip} always exists
echo "${MyShip.Name}"    ; Safe
```

**Some TLOs are conditional**:
```lavish
; ${Station} only exists when docked
if ${Station(exists)}
{
    echo "Docked at ${Station.Name}"
}
else
{
    echo "Not docked"
}
```

---

## Me Object

### Overview

**${Me}** represents your character/pilot and provides:
- Character information (name, ID, corp, alliance)
- Location and state (in space, in station, in fleet)
- Ship state at character level (different from ${MyShip})
- Targeting information
- Skills
- Wallet
- Standings

**Wiki Reference**: `IsxeveWiki/ISXEVE/Me_(Object_Type).html`

### DataType Inheritance

**CRITICAL CONCEPT**: ${Me} is a `character` datatype, which inherits from `pilot`, which inherits from `being`.

```
being (base type)
  ↓
pilot (adds ship/corp/alliance info)
  ↓
character (adds skills/wallet/fleet/targeting) ← ${Me} is this type
```

**What This Means**:
- ${Me} has **all members** from `character`, `pilot`, AND `being` datatypes
- Methods available at any level work on ${Me}
- This is why ${Me} has so many members - it inherits from 3 levels!

**Inherited from `being`**:
- ID, CharID, Name
- IsOnline, IsNPC, IsPC
- Methods: InviteToFleet, GiveMoney

**Inherited from `pilot`**:
- Type, TypeID (ship type)
- Corp, Alliance, AllianceID, AllianceTicker
- Standing, WarFactionID
- IsLimitedEngagement, IsSuspect, IsCriminal
- Methods: SetStanding, InviteToFleet, OpenShowInfo

**Specific to `character`**:
- All the character-specific members below (skills, wallet, fleet, etc.)

**Type Conversions**:
```lavish
; Convert to other types
variable entity MeAsEntity = ${Me.ToEntity}
variable fleetmember MeAsFleetMember = ${Me.ToFleetMember}  ; Only if in fleet
variable pilot MeAsPilot = ${Me.ToPilot}

; These give access to different member sets
echo "My ship type: ${Me.ToEntity.Type}"
echo "My fleet role: ${Me.ToFleetMember.Role}"
```

### Basic Character Info

```lavish
; Identity
echo "Name: ${Me.Name}"
echo "Char ID: ${Me.CharID}"
echo "Corp: ${Me.Corp}"
echo "CorpID: ${Me.CorpID}"
echo "Alliance: ${Me.Alliance}"
echo "AllianceID: ${Me.AllianceID}"

; Character sheet info
echo "Security Status: ${Me.SecurityStatus}"
echo "Bounty: ${Me.Bounty}"
```

### Location State

```lavish
; Where am I?
echo "In Station: ${Me.InStation}"
echo "In Space: ${Me.InSpace}"
echo "System ID: ${Me.SolarSystemID}"
echo "System Name: ${Me.SolarSystem.Name}"
echo "Region ID: ${Me.RegionID}"

; Specific checks
if ${Me.InStation}
{
    echo "We're docked"
}

if ${Me.InSpace}
{
    echo "We're in space"
}
```

**CRITICAL**: Use `${Me.InStation}` and `${Me.InSpace}` constantly to gate logic:
```lavish
; Most combat/movement logic only works in space
if !${Me.InSpace}
{
    echo "Cannot target - not in space"
    return
}

; Station services only work when docked
if !${Me.InStation}
{
    echo "Cannot access hangar - not docked"
    return
}
```

### Fleet State

```lavish
; Fleet membership
echo "In Fleet: ${Me.InFleet}"
echo "Fleet ID: ${Me.FleetID}"
echo "Fleet Size: ${Me.Fleet.MemberCount}"

; Fleet role
echo "Is Fleet Boss: ${Me.IsFleetBoss}"
echo "Is Fleet Leader: ${Me.Fleet.IsLeader}"

; Usage
if ${Me.InFleet}
{
    echo "Fleet member count: ${Me.Fleet.MemberCount}"
}
```

### Targeting Info

```lavish
; Target counts
echo "Max Targets: ${Me.MaxLockedTargets}"
echo "Current Targets: ${Me.GetTargets}"
echo "Max Target Range: ${Me.MaxTargetRange}m"

; Accessing targets (1-indexed!)
variable int i
for (i:Set[1]; ${i} <= ${Me.GetTargets}; i:Inc)
{
    echo "Target ${i}: ${Me.GetTarget[${i}].Name} at ${Me.GetTarget[${i}].Distance}m"
}

; Active target (what you're currently selected on)
if ${Me.ActiveTarget(exists)}
{
    echo "Active target: ${Me.ActiveTarget.Name}"
}
```

**Common Pattern - Get All Target IDs**:
```lavish
function GetAllTargetIDs()
{
    variable int64[] targetIDs
    variable int i
    variable int count = ${Me.GetTargets}

    for (i:Set[1]; ${i} <= ${count}; i:Inc)
    {
        if ${Me.GetTarget[${i}](exists)}
        {
            targetIDs:Insert[${Me.GetTarget[${i}].ID}]
        }
    }

    ; Return count (caller can access targetIDs array)
    return ${targetIDs.Used}
}
```

### Skills

```lavish
; Check if skill is trained
if ${Me.HasSkill["Mining"]}
{
    echo "Mining skill trained"
}

; Get skill level
echo "Mining level: ${Me.Skill["Mining"].Level}"

; Get skill ID
echo "Mining ID: ${Me.Skill["Mining"].ID}"

; Skill training status
echo "Currently Training: ${Me.SkillCurrentlyTraining}"
echo "Training Ends: ${Me.SkillTrainingEnd}"
```

**Common Pattern - Skill Check**:
```lavish
function HasSkillLevel(string skillName, int requiredLevel)
{
    if !${Me.HasSkill["${skillName}"]}
        return FALSE

    if ${Me.Skill["${skillName}"].Level} < ${requiredLevel}
        return FALSE

    return TRUE
}

; Usage
if !${HasSkillLevel["Mining Barge", 3]}
{
    echo "Need Mining Barge III"
    return
}
```

### Wallet

```lavish
; ISK balance
echo "ISK: ${Me.Wallet} ISK"

; Check affordability
variable float itemPrice = 1000000.00

if ${Me.Wallet} < ${itemPrice}
{
    echo "Cannot afford ${itemPrice} ISK (have ${Me.Wallet})"
}
```

### Standings

```lavish
; Get standing toward entity
variable int64 entityID = ${Entity[...].ID}
echo "Standing: ${Me.GetStanding[${entityID}]}"

; Standing is a float from -10.0 to +10.0
; Negative = hostile, Positive = friendly

; Check if entity is hostile
if ${Me.GetStanding[${entityID}]} < 0
{
    echo "Entity is hostile"
}
```

### Other Me Members

```lavish
; Ship-related (overlap with ${MyShip})
echo "Ship ID: ${Me.Ship.ID}"
echo "Ship Name: ${Me.Ship.Name}"
echo "Ship Type ID: ${Me.ToEntity.TypeID}"

; Pod check
if ${Me.InPod}
{
    echo "WARNING: In pod (ship was destroyed)"
}

; Autopilot
echo "Autopilot On: ${Me.AutoPilotOn}"

; Corporation roles
echo "Has Role Accountant: ${Me.HasRole["Accountant"]}"
```

---

## MyShip Object

### Overview

**${MyShip}** represents your current active ship and provides:
- Ship identification and type
- Capacitor, shields, armor, hull status
- Module slots and states
- Cargo and inventory
- Drones
- Speed and navigation state

**Wiki Reference**: `IsxeveWiki/ISXEVE/MyShip_(Object_Type).html`

**CRITICAL**: ${MyShip} is probably the **most used object** in combat/mining bots.

### Basic Ship Info

```lavish
; Identity
echo "Ship Name: ${MyShip.Name}"
echo "Ship ID: ${MyShip.ID}"
echo "Ship Type: ${MyShip.TypeID}"
echo "Ship Type Name: ${MyShip.ToEntity.Type}"
echo "Ship Group: ${MyShip.ToEntity.Group}"

; Example output:
; Ship Name: Bob's Retriever
; Ship ID: 123456789
; Ship Type: 17932
; Ship Type Name: Retriever
; Ship Group: Mining Barge
```

### Defensive Stats

**Shields**:
```lavish
echo "Shield HP: ${MyShip.Shield}"
echo "Max Shield: ${MyShip.MaxShield}"
echo "Shield %: ${MyShip.ShieldPct}"

; Check shield level
if ${MyShip.ShieldPct} < 50
{
    echo "WARNING: Shields below 50%"
}
```

**Armor**:
```lavish
echo "Armor HP: ${MyShip.Armor}"
echo "Max Armor: ${MyShip.MaxArmor}"
echo "Armor %: ${MyShip.ArmorPct}"
```

**Hull** (structure):
```lavish
echo "Hull HP: ${MyShip.Struct}"
echo "Max Hull: ${MyShip.MaxStruct}"
echo "Hull %: ${MyShip.StructPct}"

; Critical damage check
if ${MyShip.StructPct} < 100
{
    echo "CRITICAL: Hull damage! (${MyShip.StructPct}%)"
}
```

**Capacitor**:
```lavish
echo "Capacitor: ${MyShip.Capacitor}"
echo "Max Capacitor: ${MyShip.MaxCapacitor}"
echo "Capacitor %: ${MyShip.CapacitorPct}"

; Cap stable check
if ${MyShip.CapacitorPct} < 20
{
    echo "WARNING: Low capacitor"
}
```

**Common Pattern - Full Defense Status**:
```lavish
function LogDefenseStatus()
{
    echo "=== Ship Status ==="
    echo "Shield: ${MyShip.ShieldPct}%"
    echo "Armor: ${MyShip.ArmorPct}%"
    echo "Hull: ${MyShip.StructPct}%"
    echo "Capacitor: ${MyShip.CapacitorPct}%"
}
```

### Cargo and Inventory

**⚠️ CRITICAL DEPRECATION WARNING (July 2020)**
The following cargo/inventory methods were deprecated and may be removed:
- `${MyShip.GetCargo}` - DEPRECATED
- `${MyShip.Cargo[#]}` - DEPRECATED
- `${MyShip.UsedOreHoldCapacity}` - DEPRECATED
- `${MyShip.OreHoldCapacity}` - DEPRECATED
- All `Get*HoldCargo` methods - DEPRECATED

**Modern Replacement**: Use `EVEWindow[Inventory].Child[name]` pattern (unified inventory system).
See **File 13 (Inventory_and_Cargo_Systems.md)** for complete migration guide.

**Cargo Hold (Modern API)**:
```lavish
; MODERN API: Access via inventory window (recommended)
if ${EVEWindow[Inventory](exists)}
{
    echo "Cargo Used: ${EVEWindow[Inventory].Child[ShipCargo].UsedSpace}"
    echo "Cargo Max: ${EVEWindow[Inventory].Child[ShipCargo].Capacity}"
    echo "Cargo Free: ${EVEWindow[Inventory].Child[ShipCargo].FreeSpace}"

    ; Check if cargo full
    if ${EVEWindow[Inventory].Child[ShipCargo].FreeSpace} < 100
    {
        echo "Cargo nearly full"
    }
}

; LEGACY (may still work but deprecated):
; echo "Cargo Used: ${MyShip.UsedCargoCapacity}"
; echo "Cargo Max: ${MyShip.CargoCapacity}"
; echo "Cargo Free: ${MyShip.FreeCargoCapacity}"

; Access cargo items (MODERN METHOD - requires inventory window)
variable index:item CargoItems
if ${EVEWindow[Inventory](exists)}
{
    EVEWindow[Inventory].Child[ShipCargo]:GetItems[CargoItems]
    echo "Cargo has ${CargoItems.Used} item stacks"

    ; Iterate using iterator (modern pattern)
    variable iterator CargoIterator
    CargoItems:GetIterator[CargoIterator]
    if ${CargoIterator:First(exists)}
    {
        do
        {
            echo "Item: ${CargoIterator.Value.Name} x${CargoIterator.Value.Quantity}"
        }
        while ${CargoIterator:Next(exists)}
    }
}
```

**Ore Hold (Modern API)**:
```lavish
; Access ore hold via inventory window
if ${EVEWindow[Inventory](exists)}
{
    variable eveinvchildwindow OreHold = EVEWindow[Inventory].Child[ShipOreHold]

    if ${OreHold(exists)}
    {
        echo "Ore Hold Used: ${OreHold.UsedCapacity}"
        echo "Ore Hold Max: ${OreHold.Capacity}"

        ; Get items in ore hold
        variable index:item OreItems
        OreHold:GetItems[OreItems]
        echo "Ore stacks: ${OreItems.Used}"
    }
}
```

**Legacy API (Deprecated - Do Not Use in New Code)**:
```lavish
; DEPRECATED - Shown only for understanding old scripts
; variable int itemCount = ${MyShip.GetCargo}
; echo "Item ${i}: ${MyShip.Cargo[${i}].Name}"
;
; Replace with modern inventory window API (see above)
```

**Common Pattern - Check Cargo Full (Modern API)**:
```lavish
function IsCargoFull(int thresholdPercent)
{
    ; MODERN API: Use inventory window
    if !${EVEWindow[Inventory](exists)}
    {
        EVE:Execute[OpenInventory]
        wait 20
    }

    if !${EVEWindow[Inventory](exists)}
    {
        return FALSE  ; Cannot check without inventory
    }

    variable float usedSpace = ${EVEWindow[Inventory].Child[ShipCargo].UsedSpace}
    variable float capacity = ${EVEWindow[Inventory].Child[ShipCargo].Capacity}

    if ${capacity} == 0
    {
        return FALSE  ; Invalid capacity
    }

    variable float usedPercent = ${Math.Calc[${usedSpace} * 100.0 / ${capacity}]}

    if ${usedPercent} >= ${thresholdPercent}
        return TRUE

    return FALSE
}

; Usage
if ${IsCargoFull[90]}
{
    echo "Cargo 90% full, returning to station"
}
```

### Modules

**Module Slots**:
```lavish
; High slots (weapons, mining lasers, salvagers)
echo "High Slots: ${MyShip.HiSlots}"

; Mid slots (shield modules, prop mods, tackle)
echo "Mid Slots: ${MyShip.MedSlots}"

; Low slots (armor, damage mods, mining upgrades)
echo "Low Slots: ${MyShip.LowSlots}"

; Rig slots
echo "Rig Slots: ${MyShip.RigSlots}"
```

**Accessing Modules**:
```lavish
; Get module by slot type and index (0-indexed!)
variable item module = ${MyShip.Module[HiSlot, 0]}

if ${module(exists)}
{
    echo "High slot 0: ${module.Name}"
    echo "Is active: ${module.IsActive}"
    echo "Is online: ${module.IsOnline}"
}

; Get module by type name
variable item miner = ${MyShip.Module[=MiningLaser]}

if ${miner(exists)}
{
    echo "Found mining laser: ${miner.Name}"
}
```

**Module Patterns** (covered in depth in File 12):
```lavish
; Activate module
MyShip.Module[HiSlot, 0]:Activate

; Deactivate module
MyShip.Module[HiSlot, 0]:Deactivate

; Click module (toggle)
MyShip.Module[HiSlot, 0]:Click
```

### Drones

```lavish
; Drone stats
echo "Drones In Bay: ${MyShip.DronesInBay}"
echo "Drones In Space: ${MyShip.DronesInSpace}"
echo "Max Active Drones: ${MyShip.MaxActiveDrones}"
echo "Drone Bandwidth: ${MyShip.UsedDroneBandwidth} / ${MyShip.DroneBandwidth}"

; Drone capacity
echo "Drone Bay Used: ${MyShip.UsedDroneBayCapacity}"
echo "Drone Bay Max: ${MyShip.DroneBayCapacity}"
```

**Drone Methods** (covered in depth in combat file):
```lavish
; Launch all drones
MyShip:LaunchAllDrones

; Return drones
EVE:Execute[CmdReturnDrones]
```

### Speed and Movement

```lavish
; Current speed
echo "Velocity: ${MyShip.ToEntity.Velocity} m/s"

; Max speed
echo "Max Velocity: ${MyShip.MaxVelocity} m/s"

; Ship mass
echo "Mass: ${MyShip.Mass} kg"

; Warp speed
echo "Warp Speed: ${MyShip.WarpSpeed} AU/s"
```

**Movement State** (from ${MyShip.ToEntity}):
```lavish
; Ship mode (stationary, moving, warping)
echo "Mode: ${MyShip.ToEntity.Mode}"

; Mode values:
; 0 = Stopped
; 1 = Moving
; 2 = Approaching
; 3 = Warping
; etc. (see Entity file for full list)

if ${MyShip.ToEntity.Mode} == 3
{
    echo "Ship is warping"
}
```

### ToEntity Member

**${MyShip.ToEntity}** gives access to full **entity** object for your ship:

```lavish
; All entity members available
echo "Distance to target: ${MyShip.ToEntity.DistanceTo[${targetID}]}"
echo "X, Y, Z: ${MyShip.ToEntity.X}, ${MyShip.ToEntity.Y}, ${MyShip.ToEntity.Z}"
echo "Signature Radius: ${MyShip.ToEntity.Radius}"
```

**Why This Matters**: Some ship data is in ${MyShip}, some in ${MyShip.ToEntity}. Know which object has what you need!

---

## EVE Object

### Overview

**${EVE}** represents the game universe/UI system and provides:
- UI command execution (EVE:Execute)
- Game time and server status
- Session information
- Universe queries

**Wiki Reference**: `IsxeveWiki/ISXEVE/EVE_(Object_Type).html`

### Execute Method (CRITICAL)

**EVE:Execute** is the primary UI interaction mechanism:

```lavish
; Open inventory
EVE:Execute[CmdOpenInventory]

; Activate module
EVE:Execute[CmdActivateModule, ${slotID}]

; Warp to bookmark
EVE:Execute[CmdWarpToBookmark, ${bookmarkID}]
```

**See File 15 (UI_Windows_and_Menus.md)** for exhaustive Execute command coverage.

### Time and Server

```lavish
; EVE server time
echo "EVE Time: ${EVE.Time}"

; Session started time
echo "Session Start Time: ${EVE.SessionStartTime}"

; Session duration
variable int sessionSeconds = ${Math.Calc[${EVE.Time} - ${EVE.SessionStartTime}]}
echo "Session duration: ${sessionSeconds} seconds"
```

**Time Format**:
```lavish
; ${EVE.Time} returns a 'time' object
echo "Year: ${EVE.Time.Year}"
echo "Month: ${EVE.Time.Month}"
echo "Day: ${EVE.Time.Day}"
echo "Hour: ${EVE.Time.Hour}"
echo "Minute: ${EVE.Time.Minute}"
echo "Second: ${EVE.Time.Second}"
```

### Session Info

```lavish
; Session change counter
echo "Session Changes: ${EVE.SessionChanges}"

; When you dock/undock/jump, session changes increment
; Useful for detecting when state changes
```

**Session Change Pattern**:
```lavish
variable int lastSessionChange = ${EVE.SessionChanges}

; ... some time later ...

if ${EVE.SessionChanges} != ${lastSessionChange}
{
    echo "Session changed (docked/undocked/jumped)"
    lastSessionChange:Set[${EVE.SessionChanges}]

    ; Re-initialize state that depends on location
    call RefreshLocalData
}
```

### Universe Queries

```lavish
; Get system info by ID
variable int64 systemID = 30000142    ; Jita
echo "System Name: ${EVE.GetSystemName[${systemID}]}"

; Get item type name by ID
variable int typeID = 34
echo "Type Name: ${EVE.GetItemTypeName[${typeID}]}"
```

---

## Local Object

### Overview

**${Local}** represents the local chat channel:

```lavish
; Number of pilots in local
echo "Pilots in local: ${Local.PilotCount}"

; Check if specific pilot in local
if ${Local.Pilot["EnemyName"](exists)}
{
    echo "WARNING: Enemy in local!"
}

; Iterate all pilots
variable int i
for (i:Set[1]; ${i} <= ${Local.PilotCount}; i:Inc)
{
    echo "Pilot ${i}: ${Local.Pilot[${i}].Name}"
}
```

**Common Pattern - Enemy Detection**:
```lavish
variable(global) string[] HOSTILE_PILOTS
HOSTILE_PILOTS:Insert["EnemyOne"]
HOSTILE_PILOTS:Insert["EnemyTwo"]

function CheckForHostiles()
{
    variable int i
    for (i:Set[1]; ${i} <= ${HOSTILE_PILOTS.Used}; i:Inc)
    {
        if ${Local.Pilot["${HOSTILE_PILOTS[${i}]}"](exists)}
        {
            echo "ALERT: Hostile ${HOSTILE_PILOTS[${i}]} in local!"
            return TRUE
        }
    }

    return FALSE
}
```

---

## Station Object

### Overview

**${Station}** represents current station (only exists when docked):

```lavish
; Must check existence first
if !${Station(exists)}
{
    echo "Not docked"
    return
}

; Station info
echo "Station Name: ${Station.Name}"
echo "Station ID: ${Station.ID}"
echo "Station Type: ${Station.TypeID}"
echo "Owner Corp: ${Station.OwnerID}"

; Services
echo "Has Refining: ${Station.HasRefining}"
echo "Has Repair: ${Station.HasRepairShop}"
echo "Has Market: ${Station.HasMarket}"
echo "Has Cloning: ${Station.HasCloning}"
```

**Common Pattern - Station Services Check**:
```lavish
function CanRefineAtStation()
{
    if !${Station(exists)}
    {
        echo "Not in station"
        return FALSE
    }

    if !${Station.HasRefining}
    {
        echo "Station has no refining"
        return FALSE
    }

    return TRUE
}
```

---

## Bookmark Object

### Overview

**Bookmarks** are accessed via special methods (not a direct TLO):

```lavish
; Get bookmark by name (complex - involves inventory window)
; See File 13 (Inventory) for full bookmark handling

; Warp to bookmark (if you have ID)
EVE:Execute[CmdWarpToBookmark, ${bookmarkID}]
```

**Bookmark access is COMPLEX** and fragile. Most scripts avoid bookmark automation or use pre-configured bookmark IDs.

---

## Config Object

### Overview

**${Config}** represents ISXEVE extension configuration:

```lavish
; Check if specific settings enabled
echo "Chat logs enabled: ${Config.ChatLogs}"

; Mostly used for ISXEVE internal settings
; Not commonly used in scripts
```

---

## Additional Critical TLOs

### EVEWindow TLO

**${EVEWindow[...]}** provides access to EVE UI windows (major TLO omission in previous versions):

```lavish
; Access window by name
if ${EVEWindow[Inventory](exists)}
{
    echo "Inventory window is open"
}

; Access window by caption
variable evewindow DroneWindow = ${EVEWindow[ByCaption, "Drones"]}

; Access window by item ID
variable evewindow CargoWindow = ${EVEWindow[ByItemID, ${entityID}]}

; Access active window
echo "Active window: ${EVEWindow[Active].Name}"

; Access local chat
variable evewindow LocalChat = ${EVEWindow[Local]}
```

**Critical for**:
- Inventory access (modern cargo API requires this!)
- Market windows
- Fitting windows
- Agent dialogs
- All UI automation

**See**: File 15 (UI_Windows_and_Menus.md) for complete coverage

### Chat / Chat[id/name] TLO

**${Chat}** provides access to chat system:

```lavish
; Get channel count
echo "Channels: ${Chat.ChannelCount}"

; Access channel by name
variable chatchannel LocalChannel = ${Chat[Local]}

; Access channel by ID
variable chatchannel CorpChannel = ${Chat[123456]}

; Get all channels
variable index:chatchannel AllChannels
Chat:GetChannels[AllChannels]
```

**Common Use**: Monitor chat for commands, enemy detection, fleet coordination

### Universe[id/name] TLO

**${Universe[...]}** provides access to universe objects (systems, regions, planets):

```lavish
; Get solar system by ID
echo "System: ${Universe[30000142].Name}"  ; Jita

; Get system by name
echo "System ID: ${Universe[Jita].ID}"

; Get region/constellation
echo "Region: ${Universe[30000142].Region.Name}"
echo "Constellation: ${Universe[30000142].Constellation.Name}"

; Security status
echo "Security: ${Universe[30000142].Security}"

; Jumps calculation
echo "Jumps to Jita: ${Universe[30000142].JumpsTo[${Me.SolarSystemID}]}"
```

**Performance Note**: First use with string parameter lags (caches hundreds of thousands of names). Use IDs when possible!

### Entity[id/name] TLO

**${Entity[...]}** provides direct entity access (in space only):

```lavish
; Get entity by ID
variable entity Target = ${Entity[123456789]}

if ${Target(exists)}
{
    echo "Entity: ${Target.Name} at ${Target.Distance}m"
}

; Get entity by name (less reliable)
variable entity Gate = ${Entity["Stargate (Jita)"]}
```

**Note**: Only works for entities currently in space. Use `EVE:QueryEntities` for filtered searches.

### Being[id] TLO

**${Being[id]}** provides access to any character/NPC by ID:

```lavish
; Get character information by ID
variable being Pilot = ${Being[123456]}

echo "Name: ${Pilot.Name}"
echo "Is Online: ${Pilot.IsOnline}"
echo "Is NPC: ${Pilot.IsNPC}"
```

### Overview TLO

**${Overview}** provides overview interaction:

```lavish
; Get selected item in overview
if ${Overview.SelectedItem(exists)}
{
    echo "Selected: ${Overview.SelectedItem.Name}"
}

; Clear selection
Overview:ClearSelectedItem
```

### EVETime TLO

**${EVETime}** provides EVE time:

```lavish
; Current EVE time
echo "EVE Time: ${EVETime.DateAndTime}"
echo "Date: ${EVETime.Date}"
echo "Time: ${EVETime.Time}"

; As int64 (FILETIME format)
variable int64 timeStamp = ${EVETime.AsInt64}
```

### CharSelect TLO

**${CharSelect}** provides character selection interface (login screen only):

```lavish
; Check selected character
echo "Selected: ${CharSelect.SelectedChar}"

; Check if character exists
if ${CharSelect.CharExists["MyAlt"]}
{
    CharSelect:ClickCharacter["MyAlt"]
}
```

**Use**: Multi-account automation, character switching

---

## Other Important TLOs

### ISXEVE Object

```lavish
; Extension info
echo "ISXEVE Version: ${ISXEVE.Version}"

; Check if extension loaded
if ${ISXEVE(exists)}
{
    echo "ISXEVE loaded"
}
else
{
    echo "ERROR: ISXEVE not loaded!"
    return
}
```

**Common Pattern - Verify ISXEVE**:
```lavish
function main()
{
    if !${ISXEVE(exists)}
    {
        echo "ERROR: ISXEVE extension not loaded"
        echo "Run: extension isxeve"
        return
    }

    ; Safe to proceed
    echo "ISXEVE ${ISXEVE.Version} loaded"
}
```

### Display Object

```lavish
; Screen resolution
echo "Width: ${Display.Width}"
echo "Height: ${Display.Height}"

; Mostly not needed for scripts (UI handled by EVE:Execute)
```

---

## Common Usage Patterns

### Pattern 1: State Validation

**Always validate state before actions**:

```lavish
function ActivateMiningLaser(int64 asteroidID)
{
    ; Validate in space
    if !${Me.InSpace}
    {
        echo "ERROR: Not in space"
        return FALSE
    }

    ; Validate not in warp
    if ${MyShip.ToEntity.Mode} == 3
    {
        echo "ERROR: Cannot activate while warping"
        return FALSE
    }

    ; Validate capacitor
    if ${MyShip.CapacitorPct} < 10
    {
        echo "ERROR: Insufficient capacitor"
        return FALSE
    }

    ; Safe to proceed
    variable item miner = ${MyShip.Module[=MiningLaser]}
    miner:Activate[${asteroidID}]
    return TRUE
}
```

### Pattern 2: Damage Check (Combat)

```lavish
function ShouldRetreat()
{
    ; Check shields
    if ${MyShip.ShieldPct} < 30
    {
        echo "RETREAT: Shields critical (${MyShip.ShieldPct}%)"
        return TRUE
    }

    ; Check armor (if shields gone)
    if ${MyShip.ShieldPct} < 5 && ${MyShip.ArmorPct} < 50
    {
        echo "RETREAT: Armor damaged"
        return TRUE
    }

    ; Check hull (emergency)
    if ${MyShip.StructPct} < 100
    {
        echo "RETREAT: HULL DAMAGE!"
        return TRUE
    }

    ; Check capacitor
    if ${MyShip.CapacitorPct} < 20
    {
        echo "RETREAT: Low capacitor"
        return TRUE
    }

    return FALSE
}

; Usage in main loop
if ${ShouldRetreat}
{
    call EmergencyWarpOut
    return
}
```

### Pattern 3: Target Management

```lavish
function GetTargetCount()
{
    return ${Me.GetTargets}
}

function HasMaxTargets()
{
    return ${Math.Calc[${Me.GetTargets} >= ${Me.MaxLockedTargets}]}
}

function GetNearestTarget()
{
    variable int i
    variable int64 nearestID = 0
    variable float nearestDist = 999999999

    for (i:Set[1]; ${i} <= ${Me.GetTargets}; i:Inc)
    {
        variable entity target = ${Me.GetTarget[${i}]}

        if !${target(exists)}
            continue

        if ${target.Distance} < ${nearestDist}
        {
            nearestDist:Set[${target.Distance}]
            nearestID:Set[${target.ID}]
        }
    }

    return ${nearestID}
}
```

### Pattern 4: Fleet Coordination (Yamfa Style)

```lavish
function IsMasterCharacter()
{
    ; Define master character name
    variable string MASTER_NAME = "Bob"

    if ${Me.Name.Equal["${MASTER_NAME}"]}
        return TRUE

    return FALSE
}

function AmIInFleetWithMaster()
{
    if !${Me.InFleet}
        return FALSE

    ; Check if master is in fleet
    variable int i
    for (i:Set[1]; ${i} <= ${Me.Fleet.MemberCount}; i:Inc)
    {
        if ${Me.Fleet.Member[${i}].Name.Equal["Bob"]}
            return TRUE
    }

    return FALSE
}
```

---

## Object Relationships

### Me vs MyShip

**Overlap**:
- Some data available in both (e.g., ship ID)
- ${Me} focuses on character/pilot
- ${MyShip} focuses on ship state

**When to use which**:
```lavish
; Character state → use ${Me}
if ${Me.InStation}
if ${Me.InFleet}

; Ship defenses → use ${MyShip}
if ${MyShip.ShieldPct} < 50

; Targeting → use ${Me}
variable int targets = ${Me.GetTargets}

; Cargo → use ${MyShip}
if ${MyShip.FreeCargoCapacity} < 100

; Movement state → use ${MyShip.ToEntity}
if ${MyShip.ToEntity.Mode} == 3
```

### MyShip vs MyShip.ToEntity

**${MyShip}**:
- Ship-specific data (modules, cargo, drones)
- Capacitor, shields, armor, hull
- Ship type/name

**${MyShip.ToEntity}**:
- Position (X, Y, Z)
- Velocity, mode
- Distance calculations
- Generic entity data

**Rule**: If it's about ship state, use ${MyShip}. If it's about position/movement, use ${MyShip.ToEntity}.

---

## Performance Considerations

### Caching TLO Queries

**AVOID re-querying same data multiple times in a tick**:

**Bad**:
```lavish
if ${Me.GetTargets} > 0
{
    echo "Targets: ${Me.GetTargets}"    ; Queried again!

    if ${Me.GetTargets} >= ${Me.MaxLockedTargets}    ; Queried again!
    {
        echo "Max targets"
    }
}
```

**Good**:
```lavish
variable int targetCount = ${Me.GetTargets}

if ${targetCount} > 0
{
    echo "Targets: ${targetCount}"

    if ${targetCount} >= ${Me.MaxLockedTargets}
    {
        echo "Max targets"
    }
}
```

**Why**: Each `${}` query may involve memory reads (slow). Cache results in variables.

### Expensive Queries

**Some members are more expensive than others**:

**Fast** (cached/cheap):
- ${Me.Name}
- ${MyShip.ID}
- ${Me.InStation}

**Slower** (computed):
- ${Me.GetTargets} - Iterates target list
- ${MyShip.GetCargo} - Iterates cargo
- ${Local.PilotCount} - Iterates local pilots

**Rule**: For computed queries, cache the result if used multiple times.

---

## Critical Gotchas

### Gotcha 1: State Can Change Between Checks

**Problem**: Game state changes while script runs.

```lavish
; BAD - Target might disappear between checks
if ${Me.GetTarget[1](exists)}
{
    wait 1000    ; Long wait!
    echo "Target: ${Me.GetTarget[1].Name}"    ; Might crash if target unlocked!
}

; GOOD - Cache entity first
if ${Me.GetTarget[1](exists)}
{
    variable entity target = ${Me.GetTarget[1]}
    wait 1000
    if ${target(exists)}    ; Re-check
    {
        echo "Target: ${target.Name}"
    }
}
```

### Gotcha 2: 1-Indexed Collections

**CRITICAL**: ${Me.GetTarget}, ${MyShip.Cargo}, etc. are **1-indexed**!

```lavish
; WRONG (0-indexed assumption)
variable int i
for (i:Set[0]; ${i} < ${Me.GetTargets}; i:Inc)    ; Starts at 0!
{
    echo "${Me.GetTarget[${i}].Name}"    ; Crash on first iteration!
}

; RIGHT (1-indexed)
for (i:Set[1]; ${i} <= ${Me.GetTargets}; i:Inc)    ; Starts at 1!
{
    echo "${Me.GetTarget[${i}].Name}"    ; Works!
}
```

### Gotcha 3: (exists) is Required for Optional Objects

```lavish
; BAD - Crashes if not docked
echo "${Station.Name}"

; GOOD
if ${Station(exists)}
{
    echo "${Station.Name}"
}
```

### Gotcha 4: Mode Values Vary

**${MyShip.ToEntity.Mode}** values are not well-documented. Common values:

```
0 = Stopped
1 = Moving
2 = Approaching
3 = Warping
```

But other values exist. Always test empirically or check Evebot source.

### Gotcha 5: Capacitor/Shield/etc Can Be Zero

```lavish
; BAD - Division by zero if max is 0
variable float pct = ${Math.Calc[${MyShip.Shield} / ${MyShip.MaxShield} * 100]}

; GOOD - Check for zero
variable float pct = 0
if ${MyShip.MaxShield} > 0
{
    pct:Set[${Math.Calc[${MyShip.Shield} / ${MyShip.MaxShield} * 100]}]
}
```

**NOTE**: CCP provides ShieldPct, CapacitorPct, etc. members that handle this. Use those instead!

---

## Summary and Key Takeaways

### Essential TLOs

- **${Me}** - Character state, location, targeting, fleet, skills, wallet
- **${MyShip}** - Ship state, defenses, modules, cargo, drones
- **${EVE}** - UI commands (Execute), time, session info
- **${Local}** - Local chat, pilot list (enemy detection)
- **${Station}** - Station info and services (when docked)

### Most Common Usage

```lavish
; State checks
if ${Me.InSpace}
if ${Me.InStation}
if ${Me.InFleet}

; Ship status
variable float shieldPct = ${MyShip.ShieldPct}
variable float capPct = ${MyShip.CapacitorPct}
variable float cargoUsed = ${MyShip.UsedCargoCapacity}

; Targeting
variable int targets = ${Me.GetTargets}
variable entity target = ${Me.GetTarget[1]}

; UI interaction
EVE:Execute[CmdOpenInventory]
```

### Critical Rules

1. **Always check (exists)** for optional objects (${Station}, etc.)
2. **Always cache** queries used multiple times
3. **Remember 1-indexing** for collections (GetTarget[1], not [0])
4. **Validate state** before actions (InSpace before targeting, etc.)
5. **Use TLO convenience members** (ShieldPct, not manual math)

---

## Next Steps

**Continue to Other Layer 3 Files**:
- File 10: Entity_System_and_Targeting.md (QueryEntities, entity objects, targeting)
- File 12: Module_Management_and_Ship_Control.md (Module activation patterns)
- File 13: Inventory_and_Cargo_Systems.md (Cargo manipulation, hangar access)

**Apply Knowledge**:
- All scripts use ${Me} and ${MyShip} constantly
- Recognize these patterns in Evebot/Yamfa/Tehbot
- Build defensive checks and state validation into all functions

---

## Wiki References

**Me Object**:
- `IsxeveWiki/ISXEVE/Me_(Object_Type).html` - Complete Me reference

**MyShip Object**:
- `IsxeveWiki/ISXEVE/MyShip_(Object_Type).html` - Complete MyShip reference

**EVE Object**:
- `IsxeveWiki/ISXEVE/EVE_(Object_Type).html` - Complete EVE reference

**Other Objects**:
- `IsxeveWiki/ISXEVE/` - Main index for all ISXEVE objects

---

**END OF FILE 09**
**Status**: Complete (~1700 lines)
**Next**: Update progress tracker, continue with File 10 (Entity/Targeting) or others
