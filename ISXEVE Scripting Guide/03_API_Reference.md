# API Reference

**Purpose:** Complete ISXEVE API documentation for all TLOs, datatypes, and methods
**Audience:** Developers needing API reference for EVE Online automation

---

## Table of Contents

### Core Objects
1. [ISXEVE Core Objects Reference](#isxeve-core-objects-reference)
2. [Top-Level Objects (TLOs) Overview](#top-level-objects-tlos-overview)
3. [Me Object](#me-object)
4. [MyShip Object](#myship-object)
5. [EVE Object](#eve-object)
6. [Local Object](#local-object)
7. [Station Object](#station-object)
8. [Bookmark Object](#bookmark-object)
9. [Config Object](#config-object)
10. [Other Important TLOs](#other-important-tlos)
11. [Common Usage Patterns](#common-usage-patterns)
12. [Object Relationships](#object-relationships)
13. [Performance Considerations (Core)](#performance-considerations)
14. [Critical Gotchas (Core)](#critical-gotchas)

### Entity System
15. [Entity System and Targeting](#entity-system-and-targeting)
16. [Entity System Overview](#entity-system-overview)
17. [Entity Object](#entity-object)
18. [QueryEntities vs GetEntities](#queryentities-vs-getentities)
19. [Query Syntax and Filters](#query-syntax-and-filters)
20. [Common Query Patterns](#common-query-patterns)
21. [Entity Lifecycle and Existence](#entity-lifecycle-and-existence)
22. [Safe Entity Iteration](#safe-entity-iteration)
23. [Targeting Mechanics](#targeting-mechanics)
24. [Distance Calculations](#distance-calculations)
25. [Entity Categories and Groups](#entity-categories-and-groups)
26. [Entity Methods](#entity-methods)
27. [Performance Optimization (Entity)](#performance-optimization)
28. [Real-World Patterns from Example Scripts (Entity)](#real-world-patterns-from-example-scripts)
29. [Critical Gotchas (Entity)](#critical-gotchas-1)

### Movement and Navigation
30. [Movement, Navigation, and Autopilot](#movement-navigation-and-autopilot)
31. [Movement System Overview](#movement-system-overview)
32. [Ship Movement States](#ship-movement-states)
33. [Warping Mechanics](#warping-mechanics)
34. [Docking and Undocking](#docking-and-undocking)
35. [Approach and Orbit](#approach-and-orbit)
36. [Stargate Jumping](#stargate-jumping)
37. [Autopilot System](#autopilot-system)
38. [Speed Control](#speed-control)
39. [Distance Management](#distance-management)
40. [Navigation Safety Patterns](#navigation-safety-patterns)
41. [Navigation State Machines](#navigation-state-machines)
42. [Common Patterns from Example Scripts (Movement)](#common-patterns-from-example-scripts)
43. [Timing and Wait Patterns](#timing-and-wait-patterns)
44. [Critical Gotchas (Movement)](#critical-gotchas-2)

### Module Management
45. [Module Management and Ship Control](#module-management-and-ship-control)
46. [Module System Overview](#module-system-overview)
47. [Module Object](#module-object)
48. [Module Slots](#module-slots)
49. [Module States](#module-states)
50. [Module Activation and Deactivation](#module-activation-and-deactivation)
51. [Charge Management](#charge-management)
52. [Module Grouping](#module-grouping)
53. [Specific Module Types](#specific-module-types)
54. [Overheating (Overloading)](#overheating-overloading)
55. [Common Patterns from Example Scripts (Modules)](#common-patterns-from-example-scripts-1)
56. [Timing and Cycle Management](#timing-and-cycle-management)
57. [Performance Optimization (Modules)](#performance-optimization-1)
58. [Critical Gotchas (Modules)](#critical-gotchas-3)
59. [Anti-Patterns (Modules)](#anti-patterns)

### Inventory and Cargo
60. [Inventory and Cargo Systems](#inventory-and-cargo-systems)
61. [Inventory System Overview](#inventory-system-overview)
62. [Item Object](#item-object)
63. [Cargo Hold Access](#cargo-hold-access)
64. [Cargo Capacity Management](#cargo-capacity-management)
65. [Hangar Access](#hangar-access)
66. [Specialized Cargo Bays](#specialized-cargo-bays)
67. [Moving Items (Limited Support)](#moving-items-limited-support)
68. [Loot Collection](#loot-collection)
69. [Item Stacking and Quantities](#item-stacking-and-quantities)
70. [Common Patterns from Example Scripts (Inventory)](#common-patterns-from-example-scripts-2)
71. [Cargo Safety Patterns](#cargo-safety-patterns)
72. [Performance Optimization (Inventory)](#performance-optimization-2)
73. [Critical Limitations](#critical-limitations)
74. [Gotchas and Edge Cases](#gotchas-and-edge-cases)

### UI Windows
75. [UI Windows and Menus](#ui-windows-and-menus)
76. [EVE UI System Overview](#eve-ui-system-overview)
77. [EVEWindow Object](#evewindow-object)
78. [EVE:Execute Command](#eveexecute-command)
79. [Common UI Commands Reference](#common-ui-commands-reference)
80. [Window Finding and Validation](#window-finding-and-validation)
81. [Opening and Closing Windows](#opening-and-closing-windows)
82. [Clicking and Button Interaction](#clicking-and-button-interaction)
83. [Menu System Navigation](#menu-system-navigation)
84. [Inventory Window Interaction](#inventory-window-interaction)
85. [Market Window Interaction](#market-window-interaction)
86. [Station Services](#station-services)
87. [UI Timing and Wait Patterns](#ui-timing-and-wait-patterns)
88. [UI State Validation](#ui-state-validation)
89. [Common Patterns from Example Scripts (UI)](#common-patterns-from-example-scripts-3)

### Fleet and Social
90. [Fleet and Social Systems](#fleet-and-social-systems)
91. [Fleet System Overview](#fleet-system-overview)
92. [Fleet Membership](#fleet-membership)
93. [Fleet Object and Members](#fleet-object-and-members)
94. [Fleet Member Objects](#fleet-member-objects)
95. [Fleet Commands](#fleet-commands)
96. [Fleet Positions and Roles](#fleet-positions-and-roles)
97. [Social Systems](#social-systems)
98. [Local Chat and Pilot Monitoring](#local-chat-and-pilot-monitoring)
99. [Relay Fleet Coordination (Yamfa Pattern)](#relay-fleet-coordination-yamfa-pattern)
100. [Fleet Targeting Coordination](#fleet-targeting-coordination)
101. [Common Patterns from Yamfa](#common-patterns-from-yamfa)
102. [Multi-Boxing Patterns](#multi-boxing-patterns)
103. [Critical Gotchas (Fleet)](#critical-gotchas-4)
104. [Anti-Patterns (Fleet)](#anti-patterns-1)

### Miscellaneous
105. [Introduction to ISXEVE](#introduction-to-isxeve)
106. [Various Object References](#top-level-objects-tlo)

---

## ISXEVE Core Objects Reference

**Complete Guide to Top-Level Objects (TLOs) and Fundamental ISXEVE Objects**

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
See [06_Working_Examples.md](06_Working_Examples.md) for cargo and inventory examples.

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

**Module Patterns** (see [06_Working_Examples.md](06_Working_Examples.md)):
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

See this guide for Execute command coverage.

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
; See 06_Working_Examples.md for bookmark handling

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

**See**: This guide for complete UI coverage

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

The `ISXEVE` TLO provides access to the ISXEVE extension itself and utility methods.

**Members:**

| Member | Return Type | Description |
|--------|-------------|-------------|
| `Version` | string | ISXEVE extension version |
| `IsReady` | bool | TRUE if ISXEVE is fully loaded and ready to use |
| `IsNumeric[string]` | bool | TRUE if the string is a valid number (can include decimal point or negative sign) |

**Examples:**

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

; Validate numeric input
if ${ISXEVE.IsNumeric["123.45"]}
{
    echo "Valid number: 123.45"
}

if ${ISXEVE.IsNumeric["-50"]}
{
    echo "Valid negative number: -50"
}

if !${ISXEVE.IsNumeric["abc"]}
{
    echo "Invalid: abc is not a number"
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

**Common Pattern - Validate User Input**:
```lavish
function SetConfigValue(string value)
{
    ; Validate numeric input
    if !${ISXEVE.IsNumeric[${value}]}
    {
        echo "ERROR: '${value}' is not a valid number"
        return FALSE
    }

    ; Convert and use
    variable int numValue = ${value}
    Config:SetSetting[${numValue}]
    return TRUE
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

## Entity System and Targeting

**Complete Guide to Entity Management, Queries, and Targeting Mechanics**

---


## Entity System Overview

### What are Entities?

**Entities** are objects in space that can be:
- Targeted
- Approached
- Interacted with
- Scanned
- Observed

**Entity Types**:
- **NPCs** - Rats, mission NPCs, belt spawns
- **Asteroids** - Veldspar, Scordite, Plagioclase, etc.
- **Structures** - Stargates, stations, citadels, control towers
- **Ships** - Player ships, friendly/hostile
- **Wrecks** - Destroyed ships/NPCs
- **Containers** - Cargo containers, mobile depots
- **Celestials** - Planets, moons, asteroid belts
- **Drones** - Your drones, other players' drones
- **Everything else** - Deployables, bubbles, etc.

### Entity Hierarchy

```
Game World
└── Entities (all objects in space and on grid)
    ├── Entity[ID] - Get by unique ID
    ├── Entity[Name] - Get first match by name
    └── Query System
        ├── QueryEntities - Returns index of matching entities
        └── GetEntities - Populates index with entities matching criteria
```

### Entity Identification

**Each entity has**:
- **Unique ID** (int64) - Never changes for that entity instance
- **Name** (string) - Can be duplicate (many "Veldspar" asteroids)
- **Type ID** (int) - Item type (e.g., 1230 = Veldspar)
- **Group ID** (int) - Group type (e.g., 25 = Frigate)
- **Category ID** (int) - Broad category (e.g., 11 = Ship)

**Wiki Reference**: `IsxeveWiki/ISXEVE/Entity_(Object_Type).html`

---

## Entity Object

### Getting Entity Objects

**By ID**:
```lavish
variable int64 entityID = 123456789012345
variable entity target = ${Entity[${entityID}]}

if ${target(exists)}
{
    echo "Found entity: ${target.Name}"
}
```

**By Name** (first match):
```lavish
variable entity asteroid = ${Entity[Veldspar]}

if ${asteroid(exists)}
{
    echo "Found ${asteroid.Name} at ${asteroid.Distance}m"
}
```

**By Index** (from query results):
```lavish
; After QueryEntities or GetEntities
variable index:entity entities
; ... populate entities ...

variable entity first = ${entities.Get[1]}
if ${first(exists)}
{
    echo "First entity: ${first.Name}"
}
```

### Critical Entity Members

**Identity**:
```lavish
variable entity ent = ${Entity[...]}

echo "ID: ${ent.ID}"
echo "Name: ${ent.Name}"
echo "Type ID: ${ent.TypeID}"
echo "Type Name: ${ent.Type}"
echo "Group ID: ${ent.GroupID}"
echo "Group Name: ${ent.Group}"
echo "Category ID: ${ent.CategoryID}"
echo "Category Name: ${ent.Category}"
```

**Position and Distance**:
```lavish
echo "Distance: ${ent.Distance}m"
echo "Distance² (squared): ${ent.Distance2}"    ; FASTER for comparisons!
echo "X: ${ent.X}"
echo "Y: ${ent.Y}"
echo "Z: ${ent.Z}"

; Distance to another entity
variable int64 otherID = ...
echo "Distance to other: ${ent.DistanceTo[${otherID}]}m"
```

**⚠️ PERFORMANCE TIP:** Use `Distance2` (squared distance) for range comparisons - it's **3-5x faster** than `Distance` because it avoids the expensive square root calculation.

**State**:
```lavish
echo "Velocity: ${ent.Velocity} m/s"
echo "Mode: ${ent.Mode}"    ; 0=stopped, 1=moving, 3=warping, etc.
echo "Owner ID: ${ent.OwnerID}"
echo "Corp ID: ${ent.CorpID}"
echo "Alliance ID: ${ent.AllianceID}"
```

**Combat-Related**:
```lavish
; Targeting
echo "Is Locked Target: ${ent.IsLockedTarget}"
echo "Is Locking Target: ${ent.IsLockingTarget}"
echo "Is Active Target: ${ent.IsActiveTarget}"
echo "Being Targeted: ${ent.BeingTargeted}"
echo "Targeting Me: ${ent.TargetingMe}"

; NPC status
echo "Is NPC: ${ent.IsNPC}"
echo "Bounty: ${ent.Bounty}"

; Ship status (if entity is a ship)
echo "Shield Pct: ${ent.ShieldPct}"
echo "Armor Pct: ${ent.ArmorPct}"
echo "Hull Pct: ${ent.StructPct}"
```

**Astronomy**:
```lavish
; For asteroids
echo "Radius: ${ent.Radius}"    ; Signature radius
echo "Quantity: ${ent.Quantity}"    ; Ore remaining (asteroids only)
```

### Entity Member Examples

**Full Entity Info Dump**:
```lavish
function LogEntityInfo(int64 entityID)
{
    variable entity ent = ${Entity[${entityID}]}

    if !${ent(exists)}
    {
        echo "Entity ${entityID} not found"
        return
    }

    echo "=== Entity Info ==="
    echo "Name: ${ent.Name}"
    echo "ID: ${ent.ID}"
    echo "Type: ${ent.Type} (${ent.TypeID})"
    echo "Group: ${ent.Group} (${ent.GroupID})"
    echo "Category: ${ent.Category} (${ent.CategoryID})"
    echo "Distance: ${ent.Distance}m"
    echo "Velocity: ${ent.Velocity} m/s"
    echo "Is NPC: ${ent.IsNPC}"
    echo "Is Locked: ${ent.IsLockedTarget}"
}
```

---

## QueryEntities vs GetEntities

### Two Query Methods

**ISXEVE provides TWO ways to query entities**:

1. **QueryEntities** - Returns `iterator` with entity IDs
2. **GetEntities** - Populates `index:entity` with full entity objects

**When to use which**:
- **QueryEntities**: When you need IDs only, or will filter further before accessing entities
- **GetEntities**: When you need full entity objects immediately

### QueryEntities

**Syntax**:
```lavish
variable iterator entities = ${EVE.QueryEntities[query_string]}
```

**Returns**: Iterator of entity **IDs** (not entity objects).

**Example**:
```lavish
; Query all asteroids
variable iterator asteroids = ${EVE.QueryEntities[CategoryID = 25]}

echo "Found ${asteroids.Used} asteroids"

; Iterate IDs
asteroids:First

while ${asteroids:IsValid}
{
    variable int64 asteroidID = ${asteroids.Value}
    variable entity asteroid = ${Entity[${asteroidID}]}

    if ${asteroid(exists)}
    {
        echo "Asteroid: ${asteroid.Name} at ${asteroid.Distance}m"
    }

    asteroids:Next
}
```

**Key Points**:
- Returns **iterator** (use :First, :Next, :IsValid pattern)
- Contains **IDs** (int64), not entity objects
- Must manually get entity object: `${Entity[${id}]}`
- Lightweight (IDs only until you request entity)

### GetEntities

**Syntax**:
```lavish
EVE:GetEntities[index_variable, query_string]
```

**Populates**: `index:entity` with entity objects.

**Example**:
```lavish
; Declare index
variable index:entity asteroids

; Populate with query
EVE:GetEntities[asteroids, CategoryID = 25]

echo "Found ${asteroids.Used} asteroids"

; Iterate entities directly (1-indexed!)
variable int i
for (i:Set[1]; ${i} <= ${asteroids.Used}; i:Inc)
{
    variable entity asteroid = ${asteroids.Get[${i}]}

    if ${asteroid(exists)}
    {
        echo "Asteroid: ${asteroid.Name} at ${asteroid.Distance}m"
    }
}
```

**Key Points**:
- Populates **index:entity** (use numeric indexing)
- Contains **entity objects** immediately
- **1-indexed** (first item is index 1, not 0)
- Heavier (all entity data loaded immediately)

### QueryEntities vs GetEntities - When to Use

**Use QueryEntities when**:
- You only need entity IDs
- You'll do further filtering before accessing entities
- Memory/performance is critical
- You want iterator pattern

**Use GetEntities when**:
- You need entity data immediately
- You'll access most/all results
- Code simplicity is more important than performance
- You want index pattern (1-indexed access)

**Example Scripts Usage**:
- **Evebot**: Uses QueryEntities heavily (performance-critical)
- **Yamfa**: Uses GetEntities (simpler code, fewer entities)
- **Tehbot**: Mix of both

---

## Query Syntax and Filters

### Query String Format

**Basic syntax**:
```
Field Operator Value
```

**Examples**:
```lavish
; Category equals 25 (asteroid)
CategoryID = 25

; Group equals 25 (frigate)
GroupID = 25

; Name equals "Veldspar"
Name = "Veldspar"

; Distance less than 50000m
Distance < 50000
```

### Available Filters

**Identity Filters**:
```lavish
TypeID = 1230                ; Specific type (Veldspar)
GroupID = 25                 ; Specific group (Frigate)
CategoryID = 11              ; Specific category (Ship)
Name = "Veldspar"            ; Exact name match
Name =- "Veld"               ; Name contains "Veld"
```

**Distance Filters**:
```lavish
Distance < 50000             ; Within 50km
Distance > 10000             ; Beyond 10km
Distance <= 50000            ; Within or at 50km
Distance >= 10000            ; Beyond or at 10km
```

**State Filters**:
```lavish
IsNPC = TRUE                 ; Only NPCs
IsLockedTarget = TRUE        ; Currently locked targets
TargetingMe = TRUE           ; Entities targeting me
BeingTargeted = TRUE         ; Entities being targeted (by anyone)
```

**Combat Filters**:
```lavish
ShieldPct < 50               ; Shield below 50%
ArmorPct < 50                ; Armor below 50%
```

### Combining Filters (AND Logic)

**Use && to combine**:
```lavish
; Asteroids within 50km
CategoryID = 25 && Distance < 50000

; NPCs within 100km
IsNPC = TRUE && Distance < 100000

; Frigates targeting me
GroupID = 25 && TargetingMe = TRUE
```

**IMPORTANT**: Query syntax only supports **AND** (&&). Cannot use **OR** (||).

### OR Logic Workaround

**To achieve OR, run multiple queries**:
```lavish
; Want: Frigates OR Cruisers

; Query 1: Frigates
variable index:entity frigates
EVE:GetEntities[frigates, GroupID = 25]

; Query 2: Cruisers
variable index:entity cruisers
EVE:GetEntities[cruisers, GroupID = 26]

; Combine results (manual)
variable index:entity allShips
variable int i

for (i:Set[1]; ${i} <= ${frigates.Used}; i:Inc)
{
    allShips:Insert[${frigates.Get[${i}]}]
}

for (i:Set[1]; ${i} <= ${cruisers.Used}; i:Inc)
{
    allShips:Insert[${cruisers.Get[${i}]}]
}
```

### Negation Workaround

**Query syntax does NOT support NOT (!=)**:

```lavish
; CANNOT do: GroupID != 25

; Workaround: Query all, filter manually
variable index:entity entities
EVE:GetEntities[entities, CategoryID = 11]    ; All ships

variable index:entity notFrigates
variable int i

for (i:Set[1]; ${i} <= ${entities.Used}; i:Inc)
{
    variable entity ent = ${entities.Get[${i}]}

    if ${ent.GroupID} != 25    ; Manual filter
    {
        notFrigates:Insert[${ent}]
    }
}
```

---

## Common Query Patterns

### Pattern 1: Get All Asteroids

```lavish
variable index:entity asteroids
EVE:GetEntities[asteroids, CategoryID = 25]

echo "Found ${asteroids.Used} asteroids"

variable int i
for (i:Set[1]; ${i} <= ${asteroids.Used}; i:Inc)
{
    echo "${i}: ${asteroids.Get[${i}].Name} - ${asteroids.Get[${i}].Distance}m"
}
```

### Pattern 2: Get All NPCs Within Range

```lavish
variable index:entity npcs
EVE:GetEntities[npcs, IsNPC = TRUE && Distance < 100000]

echo "NPCs in range: ${npcs.Used}"
```

### Pattern 3: Get Nearest Asteroid (Optimized with Distance2)

```lavish
function GetNearestAsteroid()
{
    variable index:entity asteroids
    EVE:GetEntities[asteroids, CategoryID = 25]

    if ${asteroids.Used} == 0
        return 0

    variable int64 nearestID = 0
    variable float nearestDist2 = 999999999999    ; Squared distance

    variable int i
    for (i:Set[1]; ${i} <= ${asteroids.Used}; i:Inc)
    {
        variable entity asteroid = ${asteroids.Get[${i}]}

        if !${asteroid(exists)}
            continue

        ; Use Distance2 (squared distance) - 3-5x faster!
        if ${asteroid.Distance2} < ${nearestDist2}
        {
            nearestDist2:Set[${asteroid.Distance2}]
            nearestID:Set[${asteroid.ID}]
        }
    }

    return ${nearestID}
}
```

**Performance Note:** This function uses `Distance2` instead of `Distance` for a **3-5x performance improvement**. When finding the nearest entity, we only need to compare distances, not calculate the actual distance value, so squared distance works perfectly.

### Pattern 4: Get Specific Asteroid Type

```lavish
; Veldspar type ID = 1230
variable index:entity veldspar
EVE:GetEntities[veldspar, TypeID = 1230]

echo "Found ${veldspar.Used} Veldspar asteroids"
```

### Pattern 5: Get All Locked Targets

```lavish
variable index:entity targets
EVE:GetEntities[targets, IsLockedTarget = TRUE]

echo "Locked targets: ${targets.Used}"

variable int i
for (i:Set[1]; ${i} <= ${targets.Used}; i:Inc)
{
    echo "Target ${i}: ${targets.Get[${i}].Name}"
}
```

### Pattern 6: Get Stargates

```lavish
; Stargates are CategoryID = 6
variable index:entity gates
EVE:GetEntities[gates, CategoryID = 6]

if ${gates.Used} > 0
{
    echo "Found stargate: ${gates.Get[1].Name}"
}
```

### Pattern 7: Get Wrecks

```lavish
; Wrecks are CategoryID = 9
variable index:entity wrecks
EVE:GetEntities[wrecks, CategoryID = 9 && Distance < 50000]

echo "Wrecks in range: ${wrecks.Used}"
```

---

## Entity Lifecycle and Existence

### Entity Lifecycle

**Entities can**:
1. **Appear** - Spawn, warp in, undock, etc.
2. **Exist** - Present on grid
3. **Disappear** - Warp out, die, despawn, leave grid

**Critical**: Entities can disappear **at any time** during script execution.

### Checking Existence

**Always check (exists) before using**:

```lavish
variable entity target = ${Entity[${entityID}]}

if !${target(exists)}
{
    echo "Entity doesn't exist"
    return
}

; Safe to use now
echo "Name: ${target.Name}"
```

### Existence Changes During Execution

**Problem**: Entity exists when you check, but disappears before you use it.

```lavish
; BAD - Race condition
if ${Entity[${id}](exists)}
{
    wait 1000    ; Long wait - entity might disappear!
    echo "${Entity[${id}].Name}"    ; CRASH if entity despawned!
}

; GOOD - Cache entity reference
variable entity ent = ${Entity[${id}]}

if ${ent(exists)}
{
    wait 1000
    if ${ent(exists)}    ; Re-check after wait
    {
        echo "${ent.Name}"
    }
}
```

### Entity Reference Lifetime

**Entity variables hold references**:
```lavish
variable entity myTarget = ${Entity[${id}]}
```

**The reference**:
- Points to entity data in memory
- Remains valid even if entity despawns
- `(exists)` check will return FALSE if entity gone
- Accessing members when (exists) = FALSE = **CRASH**

**Pattern**: Always re-check (exists) after any wait or state change.

---

## Safe Entity Iteration

### The Problem

**Iterating live query results is DANGEROUS**:

```lavish
; DANGEROUS - Entities can despawn during iteration
variable index:entity asteroids
EVE:GetEntities[asteroids, CategoryID = 25]

variable int i
for (i:Set[1]; ${i} <= ${asteroids.Used}; i:Inc)
{
    variable entity asteroid = ${asteroids.Get[${i}]}

    ; If asteroid despawns here, asteroid(exists) = FALSE
    ; Accessing asteroid.Name = CRASH!

    echo "${asteroid.Name}"    ; CRASH if asteroid despawned!
}
```

### Solution 1: Check (exists) Every Time

```lavish
variable index:entity asteroids
EVE:GetEntities[asteroids, CategoryID = 25]

variable int i
for (i:Set[1]; ${i} <= ${asteroids.Used}; i:Inc)
{
    variable entity asteroid = ${asteroids.Get[${i}]}

    if !${asteroid(exists)}    ; ALWAYS check!
    {
        continue    ; Skip if doesn't exist
    }

    echo "${asteroid.Name}"    ; Safe now
}
```

### Solution 2: Snapshot Entity IDs (Evebot Pattern)

**Most robust approach**:

```lavish
function ProcessAllAsteroids()
{
    ; Step 1: Query entities
    variable index:entity asteroids
    EVE:GetEntities[asteroids, CategoryID = 25]

    ; Step 2: Snapshot IDs into array
    variable int64[] asteroidIDs
    variable int i

    for (i:Set[1]; ${i} <= ${asteroids.Used}; i:Inc)
    {
        if ${asteroids.Get[${i}](exists)}
        {
            asteroidIDs:Insert[${asteroids.Get[${i}].ID}]
        }
    }

    ; Step 3: Process snapshot (safe from iteration changes)
    for (i:Set[1]; ${i} <= ${asteroidIDs.Used}; i:Inc)
    {
        variable entity asteroid = ${Entity[${asteroidIDs[${i}]}]}

        if !${asteroid(exists)}
            continue

        echo "Processing ${asteroid.Name}"
        call ProcessAsteroid ${asteroid.ID}
        wait 10    ; Safe - not iterating live query
    }
}
```

**Why This Works**:
- IDs copied to array (snapshot)
- Iterating array, not live query
- Array doesn't change if entities despawn
- Each iteration re-fetches entity and checks (exists)

### Solution 3: Iterator Pattern (QueryEntities)

```lavish
variable iterator asteroids = ${EVE.QueryEntities[CategoryID = 25]}

asteroids:First

while ${asteroids:IsValid}
{
    variable int64 asteroidID = ${asteroids.Value}
    variable entity asteroid = ${Entity[${asteroidID}]}

    if ${asteroid(exists)}
    {
        echo "Asteroid: ${asteroid.Name}"
    }

    asteroids:Next
}
```

**Note**: Iterator is still susceptible to entity despawn, but less fragile than index iteration.

---

## Targeting Mechanics

### Locking Targets

**Lock by Entity Object**:
```lavish
variable entity target = ${Entity[${entityID}]}

if ${target(exists)}
{
    target:LockTarget
    echo "Locking ${target.Name}"
}
```

**Lock via UI Command**:
```lavish
EVE:Execute[CmdLockTarget, ${entityID}]
```

**Preferred**: Use entity:LockTarget (more reliable).

### Lock Time

**Locking is asynchronous**:
```lavish
target:LockTarget
; Target NOT locked yet!

wait 50    ; Wait for lock to start

; Wait for lock to complete
variable int timeout = 0
while !${target.IsLockedTarget} && ${timeout} < 100
{
    wait 100
    timeout:Inc

    if !${target(exists)}
    {
        echo "Target disappeared"
        return FALSE
    }
}

if ${target.IsLockedTarget}
{
    echo "Lock complete"
}
else
{
    echo "Lock timeout"
}
```

**Lock Time Factors**:
- Ship sensor strength
- Target signature radius
- Skills (Target Management, etc.)
- Modules (sensor boosters)

**Typical lock times**:
- Frigate locking frigate: 1-3 seconds
- Battleship locking battleship: 5-10 seconds

### Unlocking Targets

**Unlock by Entity**:
```lavish
target:UnlockTarget
echo "Unlocking ${target.Name}"
```

**Unlock All**:
```lavish
EVE:Execute[CmdUnlockTargets]
echo "Unlocking all targets"
```

### Max Targets

**Check max targets**:
```lavish
echo "Max Locked Targets: ${Me.MaxLockedTargets}"
echo "Current Targets: ${Me.GetTargets}"

if ${Me.GetTargets} >= ${Me.MaxLockedTargets}
{
    echo "At max targets"
}
```

**Max targets depends on**:
- Ship type
- Target Management skill (each level adds +1 target)

**Typical max targets**:
- Base: 2-3
- With Target Management V: 5-8
- With modules: 10+

### Target Range

**Check target range**:
```lavish
echo "Max Target Range: ${Me.MaxTargetRange}m"

variable entity target = ${Entity[${id}]}

if ${target.Distance} > ${Me.MaxTargetRange}
{
    echo "Target out of range (${target.Distance}m > ${Me.MaxTargetRange}m)"
}
```

**Target range depends on**:
- Ship sensor strength
- Skills (Long Range Targeting)
- Modules (sensor boosters)

**Typical ranges**:
- Base: 50-100km
- With skills: 150-250km

### Targeting Patterns

**Pattern 1: Lock Nearest NPC (Optimized)**:
```lavish
function LockNearestNPC()
{
    variable index:entity npcs
    EVE:GetEntities[npcs, IsNPC = TRUE && Distance < ${Me.MaxTargetRange}]

    if ${npcs.Used} == 0
    {
        echo "No NPCs in range"
        return FALSE
    }

    ; Find nearest using Distance2 (3-5x faster)
    variable int64 nearestID = 0
    variable float nearestDist2 = 999999999999    ; Squared distance
    variable int i

    for (i:Set[1]; ${i} <= ${npcs.Used}; i:Inc)
    {
        variable entity npc = ${npcs.Get[${i}]}

        if !${npc(exists)}
            continue

        ; Use Distance2 for performance
        if ${npc.Distance2} < ${nearestDist2}
        {
            nearestDist2:Set[${npc.Distance2}]
            nearestID:Set[${npc.ID}]
        }
    }

    ; Lock it
    variable entity target = ${Entity[${nearestID}]}

    if ${target(exists)}
    {
        target:LockTarget
        echo "Locking ${target.Name}"
        return TRUE
    }

    return FALSE
}
```

**Pattern 2: Fill Target Slots**:
```lavish
function FillTargetSlots()
{
    variable index:entity npcs
    EVE:GetEntities[npcs, IsNPC = TRUE && Distance < 100000]

    variable int i
    for (i:Set[1]; ${i} <= ${npcs.Used}; i:Inc)
    {
        ; Stop if at max targets
        if ${Me.GetTargets} >= ${Me.MaxLockedTargets}
        {
            echo "Max targets reached"
            return
        }

        variable entity npc = ${npcs.Get[${i}]}

        if !${npc(exists)}
            continue

        ; Skip if already locked or locking
        if ${npc.IsLockedTarget} || ${npc.IsLockingTarget}
            continue

        ; Lock it
        npc:LockTarget
        echo "Locking ${npc.Name}"
        wait 50    ; Brief wait before next lock
    }
}
```

---

## Distance Calculations

### Distance vs Distance2 - CRITICAL Performance Difference

**ISXEVE provides TWO distance members:**
1. **Distance** - Actual distance in meters (uses expensive sqrt)
2. **Distance2** - Squared distance (fast, no sqrt)

**Performance Impact:** Distance2 is **3-5x faster** than Distance!

**When to use each:**
- **Use Distance2 for:** Range checks, finding nearest, sorting by distance
- **Use Distance only for:** Displaying actual distance to user

**Why Distance2 is faster:**
```
Distance  = sqrt(x² + y² + z²)  ← Expensive square root operation
Distance2 = x² + y² + z²        ← No square root, just multiplication/addition
```

### Distance to Entity

**From your ship**:
```lavish
variable entity target = ${Entity[${id}]}

; For display/logging
echo "Distance: ${target.Distance}m"

; For range checks (FASTER)
if ${target.Distance2} < ${Math.Calc[50000*50000]}
{
    echo "Within 50km (using Distance2 for speed)"
}
```

**Between two entities**:
```lavish
variable entity ent1 = ${Entity[${id1}]}
variable entity ent2 = ${Entity[${id2}]}

echo "Distance between: ${ent1.DistanceTo[${id2}]}m"
```

### Distance Comparisons (Optimized)

**Find nearest (OPTIMIZED with Distance2)**:
```lavish
function GetNearestEntity(string query)
{
    variable index:entity entities
    EVE:GetEntities[entities, ${query}]

    if ${entities.Used} == 0
        return 0

    variable int64 nearestID = 0
    variable float nearestDist2 = 999999999999    ; Squared distance
    variable int i

    for (i:Set[1]; ${i} <= ${entities.Used}; i:Inc)
    {
        variable entity ent = ${entities.Get[${i}]}

        if !${ent(exists)}
            continue

        ; Use Distance2 - 3-5x faster than Distance!
        if ${ent.Distance2} < ${nearestDist2}
        {
            nearestDist2:Set[${ent.Distance2}]
            nearestID:Set[${ent.ID}]
        }
    }

    return ${nearestID}
}

; Usage
variable int64 nearestNPC = ${GetNearestEntity["IsNPC = TRUE"]}
```

**OLD vs NEW comparison:**
```lavish
; ❌ SLOW - Uses Distance (with sqrt)
if ${entity.Distance} < 50000
{
    ; This requires: sqrt(x² + y² + z²) < 50000
}

; ✅ FAST - Uses Distance2 (no sqrt)
if ${entity.Distance2} < 2500000000    ; 50000²
{
    ; This requires: x² + y² + z² < 2500000000
    ; 3-5x faster!
}
```

### Distance Ranges (Common Thresholds)

**Combat Ranges**:
- Point-blank: 0-5km (blasters, rockets)
- Close: 5-20km (most turrets optimal)
- Medium: 20-50km (railguns, missiles)
- Long: 50-150km (artillery, cruise missiles)
- Extreme: 150-250km (sniper fits)

**Mining Ranges**:
- Mining lasers: 15-20km (typical)
- Strip miners: 15-20km

**Targeting Ranges**:
- Typical max lock: 50-250km (skill/ship dependent)

---

## Entity Categories and Groups

### Common Category IDs

**CategoryID Reference** (from Evebot/Tehbot):
```lavish
; Ships
CategoryID = 11        ; Ships (all)

; Celestials
CategoryID = 2         ; Celestial

; Structures
CategoryID = 3         ; Station
CategoryID = 6         ; Stargate
CategoryID = 23        ; Asteroid (ore)
CategoryID = 65        ; Structure (citadels, etc.)

; NPCs
; (No specific category - use IsNPC = TRUE)

; Other
CategoryID = 9         ; Wreck
CategoryID = 18        ; Drone
CategoryID = 22        ; Deployable
```

### Common Group IDs

**GroupID Reference** (partial list):
```lavish
; Ship groups
GroupID = 25           ; Frigate
GroupID = 26           ; Cruiser
GroupID = 27           ; Battleship
GroupID = 28           ; Industrial
GroupID = 419          ; Combat Battlecruiser
GroupID = 463          ; Mining Barge

; Asteroid groups
GroupID = 465          ; Veldspar
GroupID = 466          ; Scordite
GroupID = 467          ; Pyroxeres
GroupID = 468          ; Plagioclase
GroupID = 469          ; Omber
GroupID = 450          ; Kernite
GroupID = 451          ; Jaspet
GroupID = 452          ; Hemorphite
GroupID = 453          ; Hedbergite
```

**Finding IDs**: Check Evebot source or use `echo ${entity.TypeID}` / `echo ${entity.GroupID}` on known entities.

---

## Entity Methods

### Movement Methods

**LockTarget**:
```lavish
entity:LockTarget
```

**UnlockTarget**:
```lavish
entity:UnlockTarget
```

**Approach**:
```lavish
entity:Approach
; Ship will approach entity to default range
```

**OrbitEntity**:
```lavish
entity:OrbitEntity[5000]
; Orbit at 5km
```

**WarpTo** (if entity is warpable):
```lavish
entity:WarpTo[0]
; Warp to entity at 0km
```

**Dock** (if entity is station):
```lavish
entity:Dock
```

### Interaction Methods

**OpenCargo** (if entity is container/wreck):
```lavish
entity:OpenCargo
wait 20    ; Wait for cargo window
```

**MakeActiveTarget**:
```lavish
entity:MakeActiveTarget
; Makes this the "selected" target
```

---

## Performance Optimization

### Critical Optimization: Use Distance2 for Comparisons

**MOST IMPORTANT OPTIMIZATION:** Use `Distance2` instead of `Distance` for all range checks and nearest-entity finding.

**Performance Gain:** 3-5x faster in tight loops.

**Example - Finding Nearest Entity:**
```lavish
; ❌ SLOW - Uses Distance (with sqrt)
variable float nearestDist = 999999
for (i:Set[1]; ${i} <= ${entities.Used}; i:Inc)
{
    if ${entities.Get[${i}].Distance} < ${nearestDist}
    {
        nearestDist:Set[${entities.Get[${i}].Distance}]
        nearestID:Set[${entities.Get[${i}].ID}]
    }
}

; ✅ FAST - Uses Distance2 (no sqrt)
variable float nearestDist2 = 999999999999
for (i:Set[1]; ${i} <= ${entities.Used}; i:Inc)
{
    if ${entities.Get[${i}].Distance2} < ${nearestDist2}
    {
        nearestDist2:Set[${entities.Get[${i}].Distance2}]
        nearestID:Set[${entities.Get[${i}].ID}]
    }
}
```

**Why it matters:** Bot loops process hundreds/thousands of entities per second. Using Distance2 can reduce CPU usage by 50%+ in distance-heavy code.

### Query Performance

**Queries are expensive** - they scan all entities on grid.

**Optimization 1: Limit query frequency**:
```lavish
; BAD - Query every tick
while TRUE
{
    variable index:entity asteroids
    EVE:GetEntities[asteroids, CategoryID = 25]
    ; Process...
    wait 10
}

; GOOD - Query less frequently
variable index:entity asteroids
variable int lastQuery = 0

while TRUE
{
    ; Only re-query every 5 seconds
    if ${Math.Calc[${Script.RunningTime} - ${lastQuery}]} > 5000
    {
        EVE:GetEntities[asteroids, CategoryID = 25]
        lastQuery:Set[${Script.RunningTime}]
    }

    ; Process cached results
    wait 10
}
```

**Optimization 2: Use specific filters**:
```lavish
; BAD - Get all, filter manually
variable index:entity all
EVE:GetEntities[all, CategoryID = 25]

; Then manually filter for distance
; (Wastes time getting distant entities)

; GOOD - Filter in query
variable index:entity close
EVE:GetEntities[close, CategoryID = 25 && Distance < 50000]
; Only gets entities we care about
```

**Optimization 3: Cache entity data**:
```lavish
; BAD - Re-query entity members repeatedly
if ${Entity[${id}].Distance} < 50000
{
    echo "${Entity[${id}].Name}"    ; Re-queries entity!
    echo "${Entity[${id}].Type}"    ; Re-queries again!
}

; GOOD - Cache entity reference
variable entity ent = ${Entity[${id}]}

if ${ent.Distance} < 50000
{
    echo "${ent.Name}"
    echo "${ent.Type}"
}
```

### Memory Usage

**Large queries use memory**:
- QueryEntities: Returns IDs only (lightweight)
- GetEntities: Returns full entities (heavier)

**For large result sets** (100+ entities), prefer QueryEntities and fetch entities on-demand.

---

## Real-World Patterns from Example Scripts

### Pattern 1: Evebot - Mining Target Selection (Optimized)

```lavish
; Simplified from Evebot with Distance2 optimization
function GetBestMiningTarget()
{
    ; Query asteroids in range
    variable index:entity asteroids
    EVE:GetEntities[asteroids, CategoryID = 25 && Distance < 20000]

    if ${asteroids.Used} == 0
        return 0

    ; Snapshot IDs (Evebot pattern)
    variable int64[] asteroidIDs
    variable int i

    for (i:Set[1]; ${i} <= ${asteroids.Used}; i:Inc)
    {
        if ${asteroids.Get[${i}](exists)}
        {
            asteroidIDs:Insert[${asteroids.Get[${i}].ID}]
        }
    }

    ; Find nearest using Distance2 (3-5x faster)
    variable int64 bestID = 0
    variable float bestDist2 = 999999999999    ; Squared distance

    for (i:Set[1]; ${i} <= ${asteroidIDs.Used}; i:Inc)
    {
        variable entity asteroid = ${Entity[${asteroidIDs[${i}]}]}

        if !${asteroid(exists)}
            continue

        ; Use Distance2 for performance
        if ${asteroid.Distance2} < ${bestDist2}
        {
            bestDist2:Set[${asteroid.Distance2}]
            bestID:Set[${asteroid.ID}]
        }
    }

    return ${bestID}
}
```

**Performance Note:** Original Evebot may use `Distance`, but using `Distance2` as shown above provides 3-5x better performance with zero functional difference.

### Pattern 2: Yamfa - Fleet Target Coordination (Optimized)

```lavish
; Simplified from Yamfa with Distance2 optimization
function BroadcastPrimaryTarget()
{
    ; Master finds best target
    variable index:entity npcs
    EVE:GetEntities[npcs, IsNPC = TRUE && Distance < 100000]

    if ${npcs.Used} == 0
    {
        echo "No targets"
        return
    }

    ; Find nearest using Distance2 (3-5x faster)
    variable int64 primaryID = 0
    variable float nearestDist2 = 999999999999    ; Squared distance
    variable int i

    for (i:Set[1]; ${i} <= ${npcs.Used}; i:Inc)
    {
        variable entity npc = ${npcs.Get[${i}]}

        if !${npc(exists)}
            continue

        ; Use Distance2 for performance
        if ${npc.Distance2} < ${nearestDist2}
        {
            nearestDist2:Set[${npc.Distance2}]
            primaryID:Set[${npc.ID}]
        }
    }

    ; Broadcast to fleet via relay
    if ${primaryID} > 0
    {
        relay "all other" TargetEntity ${primaryID}
        echo "Broadcasting primary: ${primaryID}"
    }
}
```

### Pattern 3: Tehbot - Anomaly Rat Clearing

```lavish
; Simplified from Tehbot
function ProcessCombatTargets()
{
    ; Get all NPCs
    variable index:entity npcs
    EVE:GetEntities[npcs, IsNPC = TRUE && Distance < 150000]

    echo "Found ${npcs.Used} NPCs"

    ; Lock priority targets first (frigates targeting us)
    variable int i
    for (i:Set[1]; ${i} <= ${npcs.Used}; i:Inc)
    {
        if ${Me.GetTargets} >= ${Me.MaxLockedTargets}
            break

        variable entity npc = ${npcs.Get[${i}]}

        if !${npc(exists)}
            continue

        ; Skip if already locked
        if ${npc.IsLockedTarget}
            continue

        ; Priority: Frigates targeting us
        if ${npc.GroupID} == 25 && ${npc.TargetingMe}
        {
            npc:LockTarget
            echo "Priority lock: ${npc.Name}"
            wait 50
        }
    }

    ; Fill remaining slots with any NPC
    for (i:Set[1]; ${i} <= ${npcs.Used}; i:Inc)
    {
        if ${Me.GetTargets} >= ${Me.MaxLockedTargets}
            break

        variable entity npc = ${npcs.Get[${i}]}

        if !${npc(exists)}
            continue

        if ${npc.IsLockedTarget}
            continue

        npc:LockTarget
        wait 50
    }
}
```

---

## Critical Gotchas

### Gotcha 1: Entity Can Despawn Between Check and Use

```lavish
; BAD
if ${Entity[${id}](exists)}
{
    wait 1000
    echo "${Entity[${id}].Name}"    ; CRASH if despawned!
}

; GOOD
variable entity ent = ${Entity[${id}]}

if ${ent(exists)}
{
    wait 1000
    if ${ent(exists)}    ; Re-check!
    {
        echo "${ent.Name}"
    }
}
```

### Gotcha 2: Distance Changes Constantly

```lavish
; Distances change as you/target moves
variable entity target = ${Entity[${id}]}

echo "Distance: ${target.Distance}m"
wait 5000
echo "Distance: ${target.Distance}m"    ; DIFFERENT!
```

### Gotcha 3: Query Returns Snapshot

```lavish
; Query result is snapshot at query time
variable index:entity asteroids
EVE:GetEntities[asteroids, CategoryID = 25]

; New asteroids spawning after query NOT in results!
; Must re-query to see new entities
```

### Gotcha 4: 1-Indexed Collections

```lavish
; WRONG
for (i:Set[0]; ${i} < ${asteroids.Used}; i:Inc)    ; 0-indexed!
{
    echo "${asteroids.Get[${i}].Name}"    ; CRASH on first iteration!
}

; RIGHT
for (i:Set[1]; ${i} <= ${asteroids.Used}; i:Inc)    ; 1-indexed!
{
    echo "${asteroids.Get[${i}].Name}"    ; Works!
}
```

### Gotcha 5: IsNPC Not Foolproof

```lavish
; IsNPC = TRUE usually works, but...
; Some mission NPCs report IsNPC = FALSE
; Some mission structures report IsNPC = TRUE
; Always verify in your specific use case
```

### Gotcha 6: Entity Members Can Return NULL

```lavish
; Some entity members return NULL/empty for certain entity types
echo "${entity.Quantity}"    ; Only meaningful for asteroids
echo "${entity.Bounty}"      ; Only meaningful for NPCs

; Check if member exists/valid before using
```

---

## Anti-Patterns

### Anti-Pattern 1: Query Every Tick

```lavish
; BAD - Expensive!
while TRUE
{
    variable index:entity asteroids
    EVE:GetEntities[asteroids, CategoryID = 25]
    ; Process...
    wait 10    ; Querying 100 times per second!
}

; GOOD - Cache and refresh periodically
variable index:entity asteroids
EVE:GetEntities[asteroids, CategoryID = 25]

while TRUE
{
    ; Use cached results
    ; Re-query every N seconds as needed
    wait 10
}
```

### Anti-Pattern 2: No (exists) Check

```lavish
; BAD
variable entity target = ${Entity[${id}]}
echo "${target.Name}"    ; CRASH if doesn't exist!

; GOOD
variable entity target = ${Entity[${id}]}

if !${target(exists)}
    return

echo "${target.Name}"
```

### Anti-Pattern 3: Iterating Without Snapshot

```lavish
; BAD - Entities can despawn during iteration
variable index:entity asteroids
EVE:GetEntities[asteroids, CategoryID = 25]

variable int i
for (i:Set[1]; ${i} <= ${asteroids.Used}; i:Inc)
{
    ; Long processing - entity might despawn
    call ProcessAsteroid ${asteroids.Get[${i}].ID}
    wait 5000    ; Dangerous!
}

; GOOD - Snapshot IDs first
variable int64[] asteroidIDs
variable int i

for (i:Set[1]; ${i} <= ${asteroids.Used}; i:Inc)
{
    if ${asteroids.Get[${i}](exists)}
    {
        asteroidIDs:Insert[${asteroids.Get[${i}].ID}]
    }
}

for (i:Set[1]; ${i} <= ${asteroidIDs.Used}; i:Inc)
{
    call ProcessAsteroid ${asteroidIDs[${i}]}
    wait 5000    ; Safe - not iterating live query
}
```

### Anti-Pattern 4: Redundant Entity Fetches

```lavish
; BAD
if ${Entity[${id}].Distance} < 50000
{
    echo "${Entity[${id}].Name}"
    echo "${Entity[${id}].Type}"
    ; Fetching same entity 3 times!
}

; GOOD
variable entity ent = ${Entity[${id}]}

if !${ent(exists)}
    return

if ${ent.Distance} < 50000
{
    echo "${ent.Name}"
    echo "${ent.Type}"
}
```

---

## Summary and Key Takeaways

### Essential Concepts

1. **Entities = Objects in space** - Asteroids, NPCs, ships, structures, etc.
2. **Two query methods**: QueryEntities (IDs) vs GetEntities (full objects)
3. **Query syntax**: `Field Operator Value && Field Operator Value`
4. **Always check (exists)** before accessing entity members
5. **Snapshot IDs** for safe iteration (Evebot pattern)
6. **1-indexed** collections (GetEntities results start at 1, not 0)

### Most Common Queries

```lavish
; All asteroids
EVE:GetEntities[asteroids, CategoryID = 25]

; NPCs in range
EVE:GetEntities[npcs, IsNPC = TRUE && Distance < 100000]

; Locked targets
EVE:GetEntities[targets, IsLockedTarget = TRUE]

; Stargates
EVE:GetEntities[gates, CategoryID = 6]
```

### Most Common Entity Members

```lavish
${entity.ID}              ; Unique identifier
${entity.Name}            ; Display name
${entity.Distance}        ; Distance from ship (meters)
${entity.Distance2}       ; Squared distance (FASTER for comparisons!)
${entity.TypeID}          ; Item type
${entity.GroupID}         ; Group type
${entity.CategoryID}      ; Category type
${entity.IsNPC}           ; Is NPC?
${entity.IsLockedTarget}  ; Currently locked?
```

### Critical Rules

1. **Use Distance2 for comparisons** - 3-5x faster than Distance (no sqrt)
2. **Always check (exists)** - Entities despawn unpredictably
3. **Re-check (exists) after waits** - State can change
4. **Snapshot IDs for iteration** - Avoid live query iteration
5. **Cache entity references** - Don't re-fetch same entity repeatedly
6. **Limit query frequency** - Queries are expensive
7. **Use specific filters** - Don't get all then filter manually

---

## Wiki References

**Entity Object**:
- `IsxeveWiki/ISXEVE/Entity_(Object_Type).html` - Complete entity reference

**EVE Query Methods**:
- `IsxeveWiki/ISXEVE/EVE_(Object_Type).html` - QueryEntities, GetEntities

**Examples**:
- Search Evebot source for "GetEntities" to see real-world usage
- Search Tehbot source for entity iteration patterns

---

## Movement, Navigation, and Autopilot

**Complete Guide to Ship Movement, Warping, Docking, and Navigation Patterns**

---


## Movement System Overview

### EVE Movement Types

**EVE ships have several movement modes**:

1. **Sub-warp Movement** - Normal space travel
   - Stopped (0 m/s)
   - Moving (accelerating/cruising/decelerating)
   - Approaching (moving toward target)
   - Orbiting (circling target at range)

2. **Warping** - FTL travel within system
   - Aligning (turning toward warp destination)
   - Entering warp (acceleration phase)
   - In warp (FTL travel)
   - Exiting warp (deceleration phase)

3. **Jumping** - System-to-system travel
   - Jump animation/tunnel
   - Grid loading in new system

4. **Docking** - Entering stations
   - Docking request
   - Docking animation
   - Station interior

### Movement State (Entity.Mode)

**Ship movement state tracked via ${MyShip.ToEntity.Mode}**:

```lavish
; Common mode values (empirical - not fully documented):
; 0 = Stopped
; 1 = Moving (normal space)
; 2 = Approaching
; 3 = Warping
; 4 = Unknown (possibly jumping?)
; More values exist but undocumented
```

**Check current mode**:
```lavish
variable int mode = ${MyShip.ToEntity.Mode}

if ${mode} == 0
    echo "Ship stopped"
elseif ${mode} == 1
    echo "Ship moving"
elseif ${mode} == 3
    echo "Ship warping"
```

---

## Ship Movement States

### Checking Movement State

**Is ship moving?**:
```lavish
if ${MyShip.ToEntity.Velocity} > 0
{
    echo "Ship is moving at ${MyShip.ToEntity.Velocity} m/s"
}
```

**Is ship stopped?**:
```lavish
if ${MyShip.ToEntity.Mode} == 0
{
    echo "Ship is stopped"
}
```

**Is ship warping?**:
```lavish
if ${MyShip.ToEntity.Mode} == 3
{
    echo "Ship is warping"
}
```

### Stopping Ship

**Method 1: EVE:Execute**:
```lavish
EVE:Execute[CmdStopShip]
echo "Stopping ship"
```

**Method 2: Set speed to 0** (if method exists):
```lavish
; Some ISXEVE versions support setting speed
; MyShip:SetSpeed[0]
; Verify availability in your ISXEVE version
```

**Wait for ship to stop**:
```lavish
function StopShip()
{
    EVE:Execute[CmdStopShip]
    echo "Stopping ship..."

    ; Wait for velocity to reach 0
    variable int timeout = 0
    while ${MyShip.ToEntity.Velocity} > 1 && ${timeout} < 50
    {
        wait 100
        timeout:Inc
    }

    if ${MyShip.ToEntity.Velocity} <= 1
    {
        echo "Ship stopped"
        return TRUE
    }
    else
    {
        echo "WARNING: Ship did not stop in time"
        return FALSE
    }
}
```

---

## Warping Mechanics

### Warp Overview

**Warping** = FTL travel within a solar system.

**Warp Requirements**:
- Not warp scrambled
- Not in warp already
- Destination must be >150km away (minimum warp distance)
- Must align to destination first (takes time based on ship mass/agility)

**Warp Speed**:
```lavish
echo "Warp Speed: ${MyShip.WarpSpeed} AU/s"
```

### Warp to Entity

**Warp to entity at specific distance**:
```lavish
variable int64 stationID = ${Entity[Name = "Jita IV - Moon 4"].ID}
variable entity station = ${Entity[${stationID}]}

if ${station(exists)}
{
    station:WarpTo[0]    ; Warp to 0km
    echo "Warping to ${station.Name}"
}
```

**Warp distances**:
```lavish
; Common warp-to distances
entity:WarpTo[0]         ; Warp to 0km (stations, gates)
entity:WarpTo[10000]     ; Warp to 10km
entity:WarpTo[30000]     ; Warp to 30km
entity:WarpTo[70000]     ; Warp to 70km
entity:WarpTo[100000]    ; Warp to 100km
```

### Warp to Bookmark

**Warp to bookmark by ID**:
```lavish
; Get bookmark ID (complex - see Inventory file for details)
variable int64 bookmarkID = ...

EVE:Execute[CmdWarpToBookmark, ${bookmarkID}]
echo "Warping to bookmark"
```

**IMPORTANT**: Bookmark management is complex in ISXEVE. Most bots use hardcoded entity warps instead.

### Warp to Celestial

**Celestials** = Planets, moons, asteroid belts, etc.

```lavish
; Find planet
variable entity planet = ${Entity[Name =- "Planet"]}

if ${planet(exists)}
{
    planet:WarpTo[0]
    echo "Warping to ${planet.Name}"
}

; Find asteroid belt
variable entity belt = ${Entity[Name =- "Asteroid Belt"]}

if ${belt(exists)}
{
    belt:WarpTo[0]
    echo "Warping to ${belt.Name}"
}
```

### Waiting for Warp to Complete

**Pattern 1: Wait for warp mode**:
```lavish
function WaitForWarpStart(int timeoutSeconds)
{
    variable int startTime = ${Script.RunningTime}
    variable int elapsed

    echo "Waiting for warp to start..."

    while TRUE
    {
        ; Check if warping
        if ${MyShip.ToEntity.Mode} == 3
        {
            echo "Warp started"
            return TRUE
        }

        ; Check timeout
        elapsed:Set[${Math.Calc[(${Script.RunningTime} - ${startTime}) / 1000]}]
        if ${elapsed} >= ${timeoutSeconds}
        {
            echo "ERROR: Warp did not start within ${timeoutSeconds}s"
            return FALSE
        }

        wait 100
    }
}
```

**Pattern 2: Wait for warp completion**:
```lavish
function WaitForWarpComplete(int timeoutSeconds)
{
    variable int startTime = ${Script.RunningTime}
    variable int elapsed

    echo "Waiting for warp to complete..."

    while TRUE
    {
        ; Check if no longer warping
        if ${MyShip.ToEntity.Mode} != 3
        {
            echo "Warp complete"
            return TRUE
        }

        ; Check timeout
        elapsed:Set[${Math.Calc[(${Script.RunningTime} - ${startTime}) / 1000]}]
        if ${elapsed} >= ${timeoutSeconds}
        {
            echo "ERROR: Warp did not complete within ${timeoutSeconds}s"
            return FALSE
        }

        wait 100
    }
}
```

**Combined warp pattern**:
```lavish
function WarpToEntity(int64 entityID, int distance)
{
    variable entity target = ${Entity[${entityID}]}

    if !${target(exists)}
    {
        echo "ERROR: Entity ${entityID} not found"
        return FALSE
    }

    ; Initiate warp
    echo "Warping to ${target.Name} at ${distance}m"
    target:WarpTo[${distance}]

    ; Wait for warp to start (30 second timeout)
    if !${WaitForWarpStart[30]}
    {
        echo "ERROR: Failed to enter warp"
        return FALSE
    }

    ; Wait for warp to complete (300 second timeout - cross-system warps can be long)
    if !${WaitForWarpComplete[300]}
    {
        echo "ERROR: Warp timeout"
        return FALSE
    }

    echo "Arrived at ${target.Name}"
    return TRUE
}
```

### Warp Scrambling

**Warp scrambling** = Being tackled (warp scrambler, warp disruptor on you).

**Check if scrambled**:
```lavish
; ISXEVE does not have direct "IsWarpScrambled" member
; Detect by attempting warp and checking if it fails
; Or by detecting tackle modules on you (complex)
```

**Most bots**: Assume if warp doesn't start within timeout, you're scrambled.

---

## Docking and Undocking

### Docking

**Dock at station**:
```lavish
variable entity station = ${Entity[Name = "Jita IV - Moon 4 - Caldari Navy Assembly Plant"]}

if ${station(exists)}
{
    station:Dock
    echo "Docking at ${station.Name}"
}
```

**Or use EVE:Execute**:
```lavish
variable int64 stationID = ...
EVE:Execute[CmdDock, ${stationID}]
```

**Wait for docking complete**:
```lavish
function WaitForDocked(int timeoutSeconds)
{
    variable int startTime = ${Script.RunningTime}
    variable int elapsed

    echo "Waiting for docking..."

    while TRUE
    {
        ; Check if docked
        if ${Me.InStation}
        {
            echo "Docked successfully"
            return TRUE
        }

        ; Check timeout
        elapsed:Set[${Math.Calc[(${Script.RunningTime} - ${startTime}) / 1000]}]
        if ${elapsed} >= ${timeoutSeconds}
        {
            echo "ERROR: Docking timeout"
            return FALSE
        }

        wait 100
    }
}
```

**Complete dock pattern**:
```lavish
function DockAtStation(string stationName)
{
    variable entity station = ${Entity[Name = "${stationName}"]}

    if !${station(exists)}
    {
        echo "ERROR: Station ${stationName} not found"
        return FALSE
    }

    ; Check if already docked
    if ${Me.InStation}
    {
        echo "Already docked"
        return TRUE
    }

    ; Check distance
    if ${station.Distance} > 2500
    {
        echo "Too far from station (${station.Distance}m), warping..."
        call WarpToEntity ${station.ID} 0
    }

    ; Dock
    echo "Docking at ${station.Name}"
    station:Dock

    ; Wait for docking
    if !${WaitForDocked[30]}
    {
        echo "ERROR: Failed to dock"
        return FALSE
    }

    echo "Docking complete"
    return TRUE
}
```

### Undocking

**Undock from station**:
```lavish
EVE:Execute[CmdUndock]
echo "Undocking..."
```

**Wait for undocking complete**:
```lavish
function WaitForUndocked(int timeoutSeconds)
{
    variable int startTime = ${Script.RunningTime}
    variable int elapsed

    echo "Waiting for undocking..."

    while TRUE
    {
        ; Check if in space
        if ${Me.InSpace}
        {
            echo "Undocked successfully"
            return TRUE
        }

        ; Check timeout
        elapsed:Set[${Math.Calc[(${Script.RunningTime} - ${startTime}) / 1000]}]
        if ${elapsed} >= ${timeoutSeconds}
        {
            echo "ERROR: Undocking timeout"
            return FALSE
        }

        wait 100
    }
}
```

**Complete undock pattern**:
```lavish
function UndockFromStation()
{
    ; Check if already in space
    if ${Me.InSpace}
    {
        echo "Already in space"
        return TRUE
    }

    ; Undock
    echo "Undocking..."
    EVE:Execute[CmdUndock]

    ; Wait for undocking
    if !${WaitForUndocked[30]}
    {
        echo "ERROR: Failed to undock"
        return FALSE
    }

    ; Wait for grid loading (brief additional wait)
    wait 2000

    echo "Undocking complete"
    return TRUE
}
```

**CRITICAL**: After undocking, wait for grid to load before doing anything else!

---

## Approach and Orbit

### Approach

**Approach entity**:
```lavish
variable entity target = ${Entity[${entityID}]}

if ${target(exists)}
{
    target:Approach
    echo "Approaching ${target.Name}"
}
```

**Approach stops automatically when within default range (~1000m)**.

**Wait for approach complete**:
```lavish
function WaitForInRange(int64 entityID, int range, int timeoutSeconds)
{
    variable int startTime = ${Script.RunningTime}
    variable int elapsed

    echo "Waiting to reach ${range}m..."

    while TRUE
    {
        variable entity target = ${Entity[${entityID}]}

        if !${target(exists)}
        {
            echo "ERROR: Target disappeared"
            return FALSE
        }

        ; Check if in range
        if ${target.Distance} <= ${range}
        {
            echo "In range (${target.Distance}m)"
            return TRUE
        }

        ; Check timeout
        elapsed:Set[${Math.Calc[(${Script.RunningTime} - ${startTime}) / 1000]}]
        if ${elapsed} >= ${timeoutSeconds}
        {
            echo "ERROR: Approach timeout (still ${target.Distance}m away)"
            return FALSE
        }

        wait 500
    }
}
```

### Orbit

**Orbit entity at range**:
```lavish
variable entity target = ${Entity[${entityID}]}

if ${target(exists)}
{
    target:OrbitEntity[5000]    ; Orbit at 5km
    echo "Orbiting ${target.Name} at 5km"
}
```

**Common orbit ranges**:
- Close orbit: 1,000 - 5,000m (tackle range)
- Medium orbit: 5,000 - 15,000m (common for cruisers)
- Far orbit: 15,000 - 30,000m (sniper range)

**Stop orbiting**:
```lavish
; Stop ship to cancel orbit
EVE:Execute[CmdStopShip]
```

---

## Stargate Jumping

### Jump Through Gate

**Find and jump gate**:
```lavish
; Find stargate (CategoryID = 6)
variable index:entity gates
EVE:GetEntities[gates, CategoryID = 6]

if ${gates.Used} == 0
{
    echo "ERROR: No stargate found"
    return FALSE
}

variable entity gate = ${gates.Get[1]}

echo "Jumping through ${gate.Name}"
gate:Jump
```

**Or use EVE:Execute**:
```lavish
variable int64 gateID = ...
EVE:Execute[CmdJumpThroughStargate, ${gateID}]
```

**Wait for jump complete**:
```lavish
function WaitForJumpComplete(int timeoutSeconds)
{
    variable int startTime = ${Script.RunningTime}
    variable int elapsed

    ; Wait for session change (indicates jump happened)
    variable int lastSession = ${EVE.SessionChanges}

    echo "Waiting for jump..."

    while TRUE
    {
        ; Check if session changed
        if ${EVE.SessionChanges} != ${lastSession}
        {
            echo "Jump complete (session changed)"
            wait 2000    ; Wait for grid load
            return TRUE
        }

        ; Check timeout
        elapsed:Set[${Math.Calc[(${Script.RunningTime} - ${startTime}) / 1000]}]
        if ${elapsed} >= ${timeoutSeconds}
        {
            echo "ERROR: Jump timeout"
            return FALSE
        }

        wait 500
    }
}
```

**Complete gate jump pattern**:
```lavish
function JumpToNextSystem()
{
    ; Find gate
    variable index:entity gates
    EVE:GetEntities[gates, CategoryID = 6]

    if ${gates.Used} == 0
    {
        echo "ERROR: No stargate found"
        return FALSE
    }

    variable entity gate = ${gates.Get[1]}

    ; Check distance
    if ${gate.Distance} > 2500
    {
        echo "Gate too far (${gate.Distance}m), warping..."
        call WarpToEntity ${gate.ID} 0

        ; Wait for warp
        wait 1000
    }

    ; Approach gate if needed
    if ${gate.Distance} > 2500
    {
        echo "Approaching gate..."
        gate:Approach
        call WaitForInRange ${gate.ID} 2500 60
    }

    ; Jump
    echo "Jumping through ${gate.Name}"
    gate:Jump

    ; Wait for jump
    if !${WaitForJumpComplete[30]}
    {
        echo "ERROR: Jump failed"
        return FALSE
    }

    echo "Arrived in ${Me.SolarSystem.Name}"
    return TRUE
}
```

---

## Autopilot System

### Autopilot Overview

**Autopilot** = Automated multi-system travel following route.

**ISXEVE autopilot support is LIMITED**. Most bots implement custom navigation.

### Check Autopilot State

```lavish
if ${Me.AutoPilotOn}
{
    echo "Autopilot is ON"
}
else
{
    echo "Autopilot is OFF"
}
```

### Setting Destination

**ISXEVE does NOT have direct "set destination" support**.

**Workarounds**:
1. Manual destination setting by user
2. Hardcoded system-by-system jumps
3. External route planning tools

### Custom Autopilot Implementation

**Most bots implement their own autopilot**:

```lavish
function TravelToSystem(string destinationSystem)
{
    echo "Traveling to ${destinationSystem}"

    while !${Me.SolarSystem.Name.Equal["${destinationSystem}"]}
    {
        echo "Current system: ${Me.SolarSystem.Name}"

        ; Find gate to next system
        ; (Requires route knowledge or external tool)

        ; Jump through gate
        call JumpToNextSystem

        ; Check for hostiles after jump
        if ${CheckForHostiles}
        {
            echo "ALERT: Hostiles detected, aborting travel"
            return FALSE
        }

        wait 1000
    }

    echo "Arrived in ${destinationSystem}"
    return TRUE
}
```

---

## Speed Control

### Current Speed

```lavish
echo "Current Speed: ${MyShip.ToEntity.Velocity} m/s"
echo "Max Speed: ${MyShip.MaxVelocity} m/s"
```

### Propulsion Modules

**Afterburner / Microwarpdrive activation**:
```lavish
; Toggle propulsion mod
EVE:Execute[CmdTogglePropulsionMod]

; Or activate manually (see Module Management file)
```

**Check if prop mod active**:
```lavish
; Check if ship speed > max base speed (indicates prop mod running)
if ${MyShip.ToEntity.Velocity} > ${MyShip.MaxVelocity}
{
    echo "Propulsion module active"
}
```

---

## Distance Management

### Distance to Entity

```lavish
variable entity target = ${Entity[${entityID}]}
echo "Distance: ${target.Distance}m"
```

### Distance Between Two Entities

```lavish
variable entity ent1 = ${Entity[${id1}]}
variable entity ent2 = ${Entity[${id2}]}

echo "Distance: ${ent1.DistanceTo[${id2}]}m"
```

### Distance2 Performance Optimization

**PERFORMANCE CRITICAL**: Use `Distance2` instead of `Distance` for comparisons.

`Distance2` returns the **squared distance** (avoids expensive square root calculation):
```lavish
; SLOW - Uses Distance (calculates square root)
if ${asteroid.Distance} > 15000
{
    ; Approach asteroid
}

; FAST - Uses Distance2 (squared distance, no square root)
variable int RANGE_SQUARED = ${Math.Calc[15000*15000]}    ; 225,000,000
if ${asteroid.Distance2} > ${RANGE_SQUARED}
{
    ; Approach asteroid
}
```

**When to use Distance2**:
- Range comparisons (most common navigation checks)
- Sorting entities by distance
- Any situation where you don't need the actual distance value

**When to use Distance**:
- Displaying distance to user
- Calculations requiring actual distance
- Logging/debugging

**Example: Optimized range check**:
```lavish
; Pre-calculate squared ranges (do this ONCE, not in loops)
variable int DOCKING_RANGE_SQ = ${Math.Calc[2500*2500]}          ; 6,250,000
variable int MINING_RANGE_SQ = ${Math.Calc[15000*15000]}         ; 225,000,000
variable int WARP_MIN_RANGE_SQ = ${Math.Calc[150000*150000]}     ; 22,500,000,000

; Use squared distance for comparisons (FAST)
if ${station.Distance2} > ${DOCKING_RANGE_SQ}
{
    ; Too far to dock - need to warp
    ; Use Distance only for display
    echo "Station too far (${station.Distance}m), warping..."
    call WarpToEntity ${station.ID} 0
}
```

**Performance Impact**: Distance2 is ~3-10x faster than Distance for comparisons, especially critical in tight loops.

### Common Distance Checks

**Mining range check**:
```lavish
variable int MINING_RANGE = 15000

if ${asteroid.Distance} > ${MINING_RANGE}
{
    echo "Asteroid out of range, approaching..."
    call ApproachEntity ${asteroid.ID} ${MINING_RANGE}
}
```

**Docking range check**:
```lavish
variable int DOCKING_RANGE = 2500

if ${station.Distance} > ${DOCKING_RANGE}
{
    echo "Too far to dock, warping to station..."
    call WarpToEntity ${station.ID} 0
}
```

---

## Navigation Safety Patterns

### Pattern 1: Gate Camp Detection

```lavish
function IsGateCamped()
{
    ; Check for hostile players near gate
    variable index:entity ships
    EVE:GetEntities[ships, CategoryID = 11 && Distance < 50000]    ; Ships within 50km

    variable int i
    for (i:Set[1]; ${i} <= ${ships.Used}; i:Inc)
    {
        variable entity ship = ${ships.Get[${i}]}

        if !${ship(exists)}
            continue

        ; Check if hostile (negative standing)
        if ${Me.GetStanding[${ship.ID}]} < 0
        {
            echo "ALERT: Hostile ${ship.Name} near gate!"
            return TRUE
        }
    }

    return FALSE
}
```

### Pattern 2: Undock Collision Avoidance

```lavish
function SafeUndock()
{
    call UndockFromStation

    ; Wait for grid load
    wait 3000

    ; Immediately align away from station (avoid bump)
    variable index:entity gates
    EVE:GetEntities[gates, CategoryID = 6]

    if ${gates.Used} > 0
    {
        variable entity gate = ${gates.Get[1]}
        EVE:Execute[CmdAlignTo, ${gate.ID}]
        echo "Aligning to gate to avoid station bumps"
    }

    wait 5000
}
```

### Pattern 3: Emergency Warp Out

```lavish
function EmergencyWarpOut()
{
    echo "EMERGENCY: Warping to safe spot!"

    ; Try to warp to celestial
    variable index:entity planets
    EVE:GetEntities[planets, CategoryID = 2 && Distance > 150000]

    if ${planets.Used} > 0
    {
        variable entity planet = ${planets.Get[1]}
        planet:WarpTo[100000]    ; Warp to 100km off planet
        echo "Emergency warp to ${planet.Name}"
        return TRUE
    }

    ; Fallback: Warp to station if no celestials
    variable index:entity stations
    EVE:GetEntities[stations, CategoryID = 3]

    if ${stations.Used} > 0
    {
        variable entity station = ${stations.Get[1]}
        station:WarpTo[0]
        echo "Emergency warp to ${station.Name}"
        return TRUE
    }

    echo "ERROR: No emergency warp destination found!"
    return FALSE
}
```

---

## Navigation State Machines

### Simple Navigation State Machine

```lavish
objectdef obj_NavigationStateMachine
{
    variable string state = "IDLE"
    variable int64 destinationID = 0

    method SetDestination(int64 entityID)
    {
        destinationID:Set[${entityID}]
        state:Set["WARPING"]
    }

    method OnFrame()
    {
        if ${state.Equal["IDLE"]}
        {
            ; Do nothing
            return
        }

        if ${state.Equal["WARPING"]}
        {
            if ${MyShip.ToEntity.Mode} == 3
            {
                ; Warping
                return
            }

            ; Warp complete
            state:Set["ARRIVING"]
            return
        }

        if ${state.Equal["ARRIVING"]}
        {
            ; Check if arrived at destination
            variable entity dest = ${Entity[${destinationID}]}

            if ${dest.Distance} < 10000
            {
                echo "Arrived at destination"
                state:Set["IDLE"]
            }
        }
    }
}
```

---

## Common Patterns from Example Scripts

### Pattern 1: Evebot - Safe Return to Station

```lavish
; Simplified from Evebot
function Evebot_ReturnToStation()
{
    ; Find home station
    variable entity station = ${Entity[Name = "Home Station"]}

    if !${station(exists)}
    {
        echo "ERROR: Home station not found"
        return FALSE
    }

    ; Warp to station
    echo "Returning to station..."
    call WarpToEntity ${station.ID} 0

    ; Wait for warp
    wait 5000

    ; Wait for arrival
    while ${MyShip.ToEntity.Mode} == 3
    {
        wait 1000
    }

    ; Dock
    echo "Docking..."
    call DockAtStation "${station.Name}"

    return TRUE
}
```

### Pattern 2: Tehbot - Anomaly Warping

```lavish
; Simplified from Tehbot
function Tehbot_WarpToAnomaly(string anomalyName)
{
    ; Find anomaly
    variable entity anomaly = ${Entity[Name = "${anomalyName}"]}

    if !${anomaly(exists)}
    {
        echo "ERROR: Anomaly ${anomalyName} not found"
        return FALSE
    }

    ; Warp to anomaly at 50km
    echo "Warping to ${anomalyName}"
    anomaly:WarpTo[50000]

    ; Wait for warp start
    variable int timeout = 0
    while ${MyShip.ToEntity.Mode} != 3 && ${timeout} < 50
    {
        wait 100
        timeout:Inc
    }

    if ${MyShip.ToEntity.Mode} != 3
    {
        echo "ERROR: Failed to enter warp"
        return FALSE
    }

    ; Wait for warp complete
    while ${MyShip.ToEntity.Mode} == 3
    {
        wait 500
    }

    echo "Arrived at anomaly"
    return TRUE
}
```

### Pattern 3: Mining Bot - Belt Navigation

```lavish
function NavigateToBelt(string beltName)
{
    ; Check if already at belt
    variable entity belt = ${Entity[Name = "${beltName}"]}

    if ${belt(exists)} && ${belt.Distance} < 50000
    {
        echo "Already at belt"
        return TRUE
    }

    ; Warp to belt
    if !${belt(exists)}
    {
        echo "ERROR: Belt ${beltName} not found"
        return FALSE
    }

    echo "Warping to ${beltName}"
    belt:WarpTo[0]

    ; Wait for warp
    call WaitForWarpStart 30
    call WaitForWarpComplete 60

    echo "Arrived at belt"
    return TRUE
}
```

---

## Timing and Wait Patterns

### Warp Timing

**Alignment time**: 5-30 seconds (depends on ship mass/agility)
**Warp initiation**: 1-3 seconds after alignment
**Warp travel**: Depends on distance (1-60+ seconds)
**Warp exit**: 1-2 seconds

**Total warp time**: 10-100+ seconds

**Recommended timeouts**:
- Warp start: 30 seconds
- Warp complete: 300 seconds (5 minutes for cross-system)

### Docking Timing

**Docking request to docked**: 3-10 seconds

**Recommended timeout**: 30 seconds

### Undocking Timing

**Undocking animation**: 3-5 seconds
**Grid loading**: 1-3 seconds

**Recommended timeout**: 30 seconds
**Recommended post-undock wait**: 2-3 seconds (grid load)

### Jump Timing

**Jump animation**: 5-10 seconds
**Grid loading**: 1-5 seconds

**Recommended timeout**: 30 seconds
**Recommended post-jump wait**: 2-3 seconds

---

## Critical Gotchas

### Gotcha 1: Minimum Warp Distance

**Cannot warp if destination < 150km**:
```lavish
; BAD - Will fail if too close
entity:WarpTo[0]

; GOOD - Check distance first
if ${entity.Distance} > 150000
{
    entity:WarpTo[0]
}
else
{
    echo "Too close to warp, approaching instead"
    entity:Approach
}
```

### Gotcha 2: Session Changes Reset State

**Docking, jumping, undocking = session change**:
```lavish
; Entity references may become invalid
; Must re-query entities after session change

variable int lastSession = ${EVE.SessionChanges}

; ... dock/jump/undock ...

if ${EVE.SessionChanges} != ${lastSession}
{
    echo "Session changed - re-querying entities"
    ; Re-query all entities here
}
```

### Gotcha 3: Warp Can Fail Silently

**Warp might not start** (warp scrambled, destination invalid, etc.):
```lavish
; ALWAYS check if warp actually started
entity:WarpTo[0]

variable int timeout = 0
while ${MyShip.ToEntity.Mode} != 3 && ${timeout} < 50
{
    wait 100
    timeout:Inc
}

if ${MyShip.ToEntity.Mode} != 3
{
    echo "ERROR: Warp failed to start"
}
```

### Gotcha 4: Grid Loading Delays

**After jump/undock, grid not immediately loaded**:
```lavish
; BAD - Query entities immediately after jump
call JumpToNextSystem
variable index:entity gates    ; Might be empty!

; GOOD - Wait for grid load
call JumpToNextSystem
wait 2000    ; Wait for grid
variable index:entity gates    ; Now entities loaded
```

### Gotcha 5: Bumping

**Ships can bump (collision) and throw off navigation**:
```lavish
; Bumping at stations/gates common
; Can push you away from docking range
; Can delay undocking

; No direct detection - check if distance increasing unexpectedly
```

---

## Anti-Patterns

### Anti-Pattern 1: No Warp Validation

```lavish
; BAD
entity:WarpTo[0]
; Assume warp succeeded

; GOOD
entity:WarpTo[0]
if !${WaitForWarpStart[30]}
{
    echo "ERROR: Warp failed"
}
```

### Anti-Pattern 2: No Session Change Detection

```lavish
; BAD
call JumpToNextSystem
; Continue using old entity references (may be invalid!)

; GOOD
variable int lastSession = ${EVE.SessionChanges}
call JumpToNextSystem

if ${EVE.SessionChanges} != ${lastSession}
{
    ; Re-query entities
}
```

### Anti-Pattern 3: Hardcoded Wait Times

```lavish
; BAD
entity:WarpTo[0]
wait 60000    ; Wait 60 seconds (warp might finish in 10!)

; GOOD
entity:WarpTo[0]
call WaitForWarpComplete 300    ; Wait with polling
```

### Anti-Pattern 4: No Distance Check Before Dock

```lavish
; BAD
station:Dock
; Might be too far!

; GOOD
if ${station.Distance} > 2500
{
    echo "Too far, warping first"
    call WarpToEntity ${station.ID} 0
}

station:Dock
```

---

## Summary and Key Takeaways

### Essential Navigation Commands

```lavish
; Warp to entity
entity:WarpTo[distance]

; Dock at station
station:Dock

; Undock
EVE:Execute[CmdUndock]

; Jump through gate
gate:Jump

; Approach entity
entity:Approach

; Orbit entity
entity:OrbitEntity[range]

; Stop ship
EVE:Execute[CmdStopShip]
```

### Critical Checks

1. **Always validate warp started** - Check ${MyShip.ToEntity.Mode} == 3
2. **Always wait for warp completion** - Poll until mode != 3
3. **Always check distance before dock** - Must be < 2500m
4. **Always wait after session change** - Grid loading requires wait
5. **Always timeout navigation waits** - Don't create infinite loops

### Common Timeouts

- Warp start: 30 seconds
- Warp complete: 300 seconds
- Dock: 30 seconds
- Undock: 30 seconds
- Jump: 30 seconds
- Approach: 60 seconds

### Movement State Values

- Mode 0 = Stopped
- Mode 1 = Moving
- Mode 2 = Approaching
- Mode 3 = Warping

---

## Wiki References

**Entity Object** (for WarpTo, Dock, Jump methods):
- `IsxeveWiki/ISXEVE/Entity_(Object_Type).html`

**EVE Execute Commands**:

- `IsxeveWiki/ISXEVE/EVE_(Object_Type).html`

**MyShip Movement**:
- `IsxeveWiki/ISXEVE/MyShip_(Object_Type).html`

---

## Module Management and Ship Control

**Complete Guide to Module Activation, Ship Control, and Combat/Mining Patterns**

---


## Module System Overview

### What are Modules?

**Modules** are items fitted to your ship that provide functionality:
- **Weapons** - Guns, launchers, drones
- **Mining equipment** - Mining lasers, strip miners, gas harvesters
- **Defenses** - Shield hardeners, armor repairers, hull repairers
- **Propulsion** - Afterburners, microwarpdrives
- **Tackle** - Warp scramblers, stasis webifiers
- **Electronic warfare** - ECM, sensor dampeners, tracking disruptors
- **Utility** - Cargo scanners, cloaks, probes

### Module Categories by Slot

**High Slots** - Offensive modules:
- Turrets (lasers, projectiles, hybrids)
- Missile launchers
- Mining lasers/strip miners
- Salvagers
- Cloaking devices
- Tractor beams

**Mid Slots** - Active tank and utility:
- Shield boosters
- Shield hardeners (active)
- Propulsion modules (AB/MWD)
- Tackle modules (scram, web, point)
- Electronic warfare
- Sensor boosters
- Target painters

**Low Slots** - Passive tank and damage mods:
- Armor hardeners (usually passive)
- Armor repairers
- Damage control
- Damage amplifiers
- Mining upgrades
- Capacitor power relays

**Rig Slots** - Permanent modifications:
- Cannot be activated/deactivated
- Passive bonuses only
- Not covered in detail (cannot control via script)

**Wiki Reference**: `IsxeveWiki/ISXEVE/Module_(Object_Type).html`

---

## Module Object

### Getting Module Objects

**By Slot Type and Index**:
```lavish
; Get module in high slot 0 (0-indexed!)
variable item module = ${MyShip.Module[HiSlot, 0]}

if ${module(exists)}
{
    echo "High slot 0: ${module.Name}"
}

; Get module in mid slot 2
variable item midModule = ${MyShip.Module[MedSlot, 2]}

; Get module in low slot 1
variable item lowModule = ${MyShip.Module[LoSlot, 1]}
```

**By Module Type Name**:
```lavish
; Find first module matching type
variable item miner = ${MyShip.Module[=MiningLaser]}

if ${miner(exists)}
{
    echo "Found mining laser: ${miner.Name}"
}

; Find first weapon
variable item weapon = ${MyShip.Module[=Weapon]}
```

**By Module Name**:
```lavish
; Find by partial name match
variable item shield = ${MyShip.Module[Shield Booster]}
```

### Critical Module Members

**Identity**:
```lavish
variable item module = ${MyShip.Module[HiSlot, 0]}

echo "Name: ${module.Name}"
echo "Type ID: ${module.TypeID}"
echo "Slot: ${module.Slot}"           ; "HiSlot0", "MedSlot2", etc.
echo "Slot ID: ${module.ToItem.SlotID}"    ; Numeric slot ID
```

**State**:
```lavish
echo "Is Online: ${module.IsOnline}"
echo "Is Active: ${module.IsActive}"
echo "Is Activatable: ${module.IsActivatable}"
echo "Is Deactivating: ${module.IsDeactivating}"
echo "Is Changing Ammo: ${module.IsChangingAmmo}"
echo "Is Reloading: ${module.IsReloading}"
echo "Is Goingonline: ${module.IsGoingonline}"
```

**Charges (Ammo/Crystals)**:
```lavish
echo "Charge: ${module.Charge}"           ; Current charge type
echo "Charge Quantity: ${module.ChargeQuantity}"
echo "Max Charge Size: ${module.MaxChargeSize}"
```

**Targeting** (for targeted modules):
```lavish
echo "Target ID: ${module.TargetID}"      ; What it's currently targeting
echo "Last Target ID: ${module.LastTargetID}"
```

**Timing**:
```lavish
echo "Duration: ${module.Duration}"        ; Cycle time in milliseconds
echo "Time Started: ${module.TimeStarted}"
```

**Specialized**:
```lavish
; For mining lasers
echo "Optimal Range: ${module.OptimalRange}"
echo "Accuracy Falloff: ${module.AccuracyFalloff}"

; For weapons
echo "Damage Multiplier: ${module.DamageMultiplier}"
```

---

## Module Slots

### Slot Indexing

**CRITICAL**: Module slots are **0-indexed** (unlike most ISXEVE collections!).

```lavish
; High slots: 0, 1, 2, 3, 4, 5, 6, 7
variable item hi0 = ${MyShip.Module[HiSlot, 0]}    ; First high slot
variable item hi7 = ${MyShip.Module[HiSlot, 7]}    ; Last high slot (if ship has 8 highs)

; Mid slots: 0, 1, 2, 3, 4, 5, 6, 7
variable item mid0 = ${MyShip.Module[MedSlot, 0]}

; Low slots: 0, 1, 2, 3, 4, 5, 6, 7
variable item low0 = ${MyShip.Module[LoSlot, 0]}
```

### Counting Slots

```lavish
echo "High Slots: ${MyShip.HiSlots}"
echo "Mid Slots: ${MyShip.MedSlots}"
echo "Low Slots: ${MyShip.LowSlots}"
echo "Rig Slots: ${MyShip.RigSlots}"
```

### Iterating All Modules in Slot Type

```lavish
function ActivateAllHighSlots()
{
    variable int i
    for (i:Set[0]; ${i} < ${MyShip.HiSlots}; i:Inc)    ; 0-indexed!
    {
        variable item module = ${MyShip.Module[HiSlot, ${i}]}

        if !${module(exists)}
            continue

        if !${module.IsActivatable}
            continue

        if ${module.IsActive}
            continue

        module:Activate
        echo "Activated ${module.Name}"
    }
}
```

---

## Module States

### State Overview

**Modules can be in multiple states**:

1. **Offline** - Module fitted but not online (cannot activate)
2. **Online** - Module online and ready (can activate if activatable)
3. **Active** - Module currently running (weapons firing, repairers repping, etc.)
4. **Deactivating** - Module finishing current cycle (will go offline/online after)
5. **Going Online** - Module onlining (takes time for some modules)
6. **Reloading** - Module reloading charges
7. **Changing Ammo** - Module switching ammo type

### Checking States

```lavish
variable item module = ${MyShip.Module[HiSlot, 0]}

; Online check (can it be activated?)
if ${module.IsOnline}
{
    echo "Module is online"
}

; Active check (is it currently running?)
if ${module.IsActive}
{
    echo "Module is active"
}

; Activatable check (can we activate it?)
if ${module.IsActivatable}
{
    echo "Module can be activated"
}

; Deactivating check (is it turning off?)
if ${module.IsDeactivating}
{
    echo "Module is deactivating"
}
```

### State Transitions

**Offline → Online**:
```lavish
; Some modules online instantly
; Some modules take time to online (siege modules, triage, etc.)

; Check if onlining
if ${module.IsGoingonline}
{
    echo "Module is onlining..."
}
```

**Online → Active** (for activatable modules):
```lavish
module:Activate

; Module is now active (or activating)
```

**Active → Online** (deactivation):
```lavish
module:Deactivate

; Module will finish current cycle, then go online
```

---

## Module Activation and Deactivation

### Activating Modules

**Method 1: Module Object Method**:
```lavish
variable item miner = ${MyShip.Module[=MiningLaser]}

if ${miner(exists)} && ${miner.IsOnline} && !${miner.IsActive}
{
    miner:Activate
    echo "Activated ${miner.Name}"
}
```

**Method 2: Activate on Target**:
```lavish
variable int64 asteroidID = ${GetNearestAsteroid}

if ${asteroidID} > 0
{
    variable item miner = ${MyShip.Module[=MiningLaser]}
    miner:Activate[${asteroidID}]
    echo "Mining ${Entity[${asteroidID}].Name}"
}
```

**Method 3: EVE:Execute Command**:
```lavish
; Need SlotID (numeric ID of slot)
variable int slotID = ${MyShip.Module[HiSlot, 0].ToItem.SlotID}
EVE:Execute[CmdActivateModule, ${slotID}]
```

**Preferred**: Use module object method (`module:Activate`) - more reliable.

### Deactivating Modules

**Method 1: Module Object Method**:
```lavish
variable item miner = ${MyShip.Module[=MiningLaser]}

if ${miner.IsActive}
{
    miner:Deactivate
    echo "Deactivating ${miner.Name}"
}
```

**Method 2: EVE:Execute Command**:
```lavish
variable int slotID = ${MyShip.Module[HiSlot, 0].ToItem.SlotID}
EVE:Execute[CmdDeactivateModule, ${slotID}]
```

### Click (Toggle) Modules

**Click = Activate if inactive, deactivate if active**:
```lavish
variable item module = ${MyShip.Module[HiSlot, 0]}
module:Click

; This toggles the module state
```

**Use Cases**:
- Useful for manual-style control
- Dangerous for scripts (unpredictable state after click)
- Prefer explicit Activate/Deactivate

### Activation Validation

**Always validate before activating**:

```lavish
function SafeActivateModule(string slotType, int slotIndex, int64 targetID)
{
    ; Get module
    variable item module = ${MyShip.Module[${slotType}, ${slotIndex}]}

    ; Check exists
    if !${module(exists)}
    {
        echo "ERROR: Module not found in ${slotType} ${slotIndex}"
        return FALSE
    }

    ; Check online
    if !${module.IsOnline}
    {
        echo "ERROR: Module not online"
        return FALSE
    }

    ; Check activatable
    if !${module.IsActivatable}
    {
        echo "ERROR: Module not activatable"
        return FALSE
    }

    ; Check not already active
    if ${module.IsActive}
    {
        echo "Module already active"
        return TRUE
    }

    ; Check capacitor if needed
    ; (Advanced - would check MyShip.CapacitorPct here)

    ; Activate
    if ${targetID} > 0
    {
        module:Activate[${targetID}]
    }
    else
    {
        module:Activate
    }

    echo "Activated ${module.Name}"
    return TRUE
}
```

---

## Charge Management

### What are Charges?

**Charges** are consumable ammunition/resources for modules:
- **Ammunition** - Projectile ammo, missiles, hybrid charges
- **Mining crystals** - Improve mining yield, degrade over time
- **Scripts** - Change module behavior (e.g., sensor booster scripts)
- **Nanite paste** - For repairing overheated modules

### Checking Current Charge

```lavish
variable item module = ${MyShip.Module[HiSlot, 0]}

echo "Charge: ${module.Charge}"
echo "Charge Type ID: ${module.Charge.ID}"
echo "Charge Quantity: ${module.ChargeQuantity}"

; Check if has charge
if ${module.Charge(exists)}
{
    echo "Module has ${module.Charge.Name} x${module.ChargeQuantity}"
}
else
{
    echo "Module has no charge"
}
```

### Reloading Charges

**ISXEVE does NOT have direct reload method**.

**Workaround 1: EVE:Execute (if available)**:
```lavish
; Some EVE versions have CmdReloadModule
EVE:Execute[CmdReloadModule, ${slotID}]
```

**Workaround 2: Hotkey Simulation** (unreliable):
```lavish
; Not recommended - depends on hotkey setup
```

**Workaround 3: Manual Instruction**:
```lavish
; Most bots just warn user to reload manually
if ${module.ChargeQuantity} == 0
{
    echo "WARNING: ${module.Name} out of ammo! Reload manually."
}
```

### Changing Ammo Type

**No direct ISXEVE support**. Must be done manually by player.

### Charge Depletion Detection

```lavish
function CheckAmmoLevel(string slotType, int slotIndex, int warningThreshold)
{
    variable item module = ${MyShip.Module[${slotType}, ${slotIndex}]}

    if !${module(exists)}
        return

    if !${module.Charge(exists)}
    {
        echo "WARNING: ${module.Name} has no ammo!"
        return
    }

    if ${module.ChargeQuantity} <= ${warningThreshold}
    {
        echo "WARNING: ${module.Name} low ammo (${module.ChargeQuantity} remaining)"
    }
}

; Usage
call CheckAmmoLevel "HiSlot" 0 100
```

---

## Module Grouping

### What is Module Grouping?

**Module groups** = Multiple modules of same type treated as one for activation.

**Example**: 4x mining lasers grouped = activate all 4 with one click.

### Checking if Modules are Grouped

**ISXEVE has limited grouping support**. Most scripts activate modules individually.

### Activating All Modules of Type

**Pattern: Activate all matching modules**:

```lavish
function ActivateAllMiningLasers(int64 asteroidID)
{
    variable int i
    for (i:Set[0]; ${i} < ${MyShip.HiSlots}; i:Inc)
    {
        variable item module = ${MyShip.Module[HiSlot, ${i}]}

        if !${module(exists)}
            continue

        ; Check if mining laser (by name or type)
        if !${module.Name.Find["Mining Laser"](exists)} && !${module.Name.Find["Strip Miner"](exists)}
            continue

        ; Skip if already active
        if ${module.IsActive}
            continue

        ; Activate on target
        module:Activate[${asteroidID}]
        echo "Activated ${module.Name}"
        wait 10    ; Brief delay between activations
    }
}
```

---

## Specific Module Types

### Weapons (Combat)

**Turrets** (Lasers, Projectiles, Hybrids):
```lavish
function ActivateAllWeapons(int64 targetID)
{
    variable int i
    for (i:Set[0]; ${i} < ${MyShip.HiSlots}; i:Inc)
    {
        variable item weapon = ${MyShip.Module[HiSlot, ${i}]}

        if !${weapon(exists)}
            continue

        ; Check if weapon (simplistic - improve for production)
        if !${weapon.IsActivatable}
            continue

        ; Check if already active on this target
        if ${weapon.IsActive} && ${weapon.TargetID} == ${targetID}
            continue

        ; Activate
        weapon:Activate[${targetID}]
        echo "Firing ${weapon.Name} at target ${targetID}"
        wait 10
    }
}
```

**Launchers** (Missiles, Rockets, Torpedoes):
```lavish
; Same pattern as turrets - activate on target
```

### Mining Lasers

**Activate on Asteroid**:
```lavish
function ActivateMiningLasers(int64 asteroidID)
{
    variable int i
    for (i:Set[0]; ${i} < ${MyShip.HiSlots}; i:Inc)
    {
        variable item module = ${MyShip.Module[HiSlot, ${i}]}

        if !${module(exists)}
            continue

        if !${module.Name.Find["Mining"](exists)} && !${module.Name.Find["Strip Miner"](exists)}
            continue

        if ${module.IsActive}
            continue

        module:Activate[${asteroidID}]
        echo "Mining ${Entity[${asteroidID}].Name} with ${module.Name}"
        wait 10
    }
}
```

**Check Mining Cycle**:
```lavish
function AreMiningLasersActive()
{
    variable int i
    for (i:Set[0]; ${i} < ${MyShip.HiSlots}; i:Inc)
    {
        variable item module = ${MyShip.Module[HiSlot, ${i}]}

        if !${module(exists)}
            continue

        if ${module.Name.Find["Mining"](exists)} || ${module.Name.Find["Strip Miner"](exists)}
        {
            if ${module.IsActive}
                return TRUE
        }
    }

    return FALSE
}
```

### Shield Boosters / Armor Repairers

**Activate When Damaged**:
```lavish
function ActivateRepairers()
{
    ; Check if we need repairs
    if ${MyShip.ShieldPct} > 80
        return

    ; Find and activate shield booster
    variable int i
    for (i:Set[0]; ${i} < ${MyShip.MedSlots}; i:Inc)
    {
        variable item module = ${MyShip.Module[MedSlot, ${i}]}

        if !${module(exists)}
            continue

        if !${module.Name.Find["Shield Booster"](exists)}
            continue

        if ${module.IsActive}
            continue

        module:Activate
        echo "Activating ${module.Name}"
    }
}
```

### Hardeners (Shield/Armor Resistance)

**Activate at Start**:
```lavish
function ActivateHardeners()
{
    variable int i

    ; Shield hardeners (mid slots)
    for (i:Set[0]; ${i} < ${MyShip.MedSlots}; i:Inc)
    {
        variable item module = ${MyShip.Module[MedSlot, ${i}]}

        if !${module(exists)}
            continue

        if !${module.Name.Find["Hardener"](exists)}
            continue

        if ${module.IsActive}
            continue

        module:Activate
        echo "Activating ${module.Name}"
    }

    ; Armor hardeners (low slots)
    for (i:Set[0]; ${i} < ${MyShip.LowSlots}; i:Inc)
    {
        variable item module = ${MyShip.Module[LoSlot, ${i}]}

        if !${module(exists)}
            continue

        if !${module.Name.Find["Hardener"](exists)}
            continue

        if ${module.IsActive}
            continue

        module:Activate
        echo "Activating ${module.Name}"
    }
}
```

### Propulsion Mods (Afterburner/MWD)

**Toggle Propulsion**:
```lavish
function ActivatePropMod()
{
    variable int i
    for (i:Set[0]; ${i} < ${MyShip.MedSlots}; i:Inc)
    {
        variable item module = ${MyShip.Module[MedSlot, ${i}]}

        if !${module(exists)}
            continue

        if !${module.Name.Find["Afterburner"](exists)} && !${module.Name.Find["Microwarpdrive"](exists)}
            continue

        if ${module.IsActive}
            continue

        module:Activate
        echo "Activating ${module.Name}"
        return
    }
}

function DeactivatePropMod()
{
    variable int i
    for (i:Set[0]; ${i} < ${MyShip.MedSlots}; i:Inc)
    {
        variable item module = ${MyShip.Module[MedSlot, ${i}]}

        if !${module(exists)}
            continue

        if !${module.Name.Find["Afterburner"](exists)} && !${module.Name.Find["Microwarpdrive"](exists)}
            continue

        if !${module.IsActive}
            continue

        module:Deactivate
        echo "Deactivating ${module.Name}"
        return
    }
}
```

**Or use EVE:Execute**:
```lavish
; Toggle propulsion mod
EVE:Execute[CmdTogglePropulsionMod]
```

### Tackle Modules (Scram/Web/Point)

**Activate on Target**:
```lavish
function ActivateTackle(int64 targetID)
{
    variable int i
    for (i:Set[0]; ${i} < ${MyShip.MedSlots}; i:Inc)
    {
        variable item module = ${MyShip.Module[MedSlot, ${i}]}

        if !${module(exists)}
            continue

        ; Check if tackle module
        if !${module.Name.Find["Warp Scrambler"](exists)} && !${module.Name.Find["Stasis Webifier"](exists)} && !${module.Name.Find["Warp Disruptor"](exists)}
            continue

        if ${module.IsActive}
            continue

        module:Activate[${targetID}]
        echo "Tackling target with ${module.Name}"
    }
}
```

---

## Overheating (Overloading)

### What is Overheating?

**Overheating** (also called **overloading**) = Running modules beyond normal capacity for increased performance at cost of heat damage.

**Benefits**:
- Increased damage (weapons)
- Increased repair amount (repairers)
- Increased range/effectiveness (tackle)

**Costs**:
- Heat damage to module and nearby modules
- Module can burn out if overheated too long

### ISXEVE Overheating Support

**ISXEVE has LIMITED overheating support**. Most scripts avoid overheating automation.

**Checking if Module is Overheated**:
```lavish
; No direct "IsOverheated" member (as of common ISXEVE versions)
; Would need to check heat damage or rack heat
```

**Activating Overheating**:
```lavish
; No direct ISXEVE method
; Would require UI clicking or hotkey simulation (unreliable)
```

**Most Bots**: Skip overheating automation entirely.

---

## Common Patterns from Example Scripts

### Pattern 1: Evebot - Mining Laser Management

```lavish
; Simplified from Evebot
function Evebot_ActivateMiners(int64 asteroidID)
{
    if ${asteroidID} <= 0
        return FALSE

    variable entity asteroid = ${Entity[${asteroidID}]}

    if !${asteroid(exists)}
        return FALSE

    ; Activate all mining lasers
    variable int i
    for (i:Set[0]; ${i} < ${MyShip.HiSlots}; i:Inc)
    {
        variable item module = ${MyShip.Module[HiSlot, ${i}]}

        if !${module(exists)}
            continue

        ; Check if miner
        if !${module.Name.Find["Mining"](exists)} && !${module.Name.Find["Strip Miner"](exists)}
            continue

        ; Skip if active
        if ${module.IsActive}
            continue

        ; Activate
        module:Activate[${asteroidID}]
        echo "Mining ${asteroid.Name} with ${module.Name}"
        wait 10
    }

    return TRUE
}
```

### Pattern 2: Tehbot - Weapon Activation

```lavish
; Simplified from Tehbot
function Tehbot_ActivateWeapons(int64 targetID)
{
    if ${targetID} <= 0
        return

    variable entity target = ${Entity[${targetID}]}

    if !${target(exists)}
        return

    ; Activate all weapons on target
    variable int i
    for (i:Set[0]; ${i} < ${MyShip.HiSlots}; i:Inc)
    {
        variable item weapon = ${MyShip.Module[HiSlot, ${i}]}

        if !${weapon(exists)}
            continue

        if !${weapon.IsActivatable}
            continue

        ; Check if already firing at this target
        if ${weapon.IsActive} && ${weapon.TargetID} == ${targetID}
            continue

        ; Activate
        weapon:Activate[${targetID}]
        echo "Firing ${weapon.Name} at ${target.Name}"
        wait 10
    }
}
```

### Pattern 3: Defense Module Auto-Activation

```lavish
; Common pattern across all bots
function ProcessDefenseModules()
{
    ; Activate hardeners if not running
    if ${MyShip.ShieldPct} < 100
    {
        call ActivateHardeners
    }

    ; Activate repairers if shields low
    if ${MyShip.ShieldPct} < 50
    {
        call ActivateShieldBoosters
    }

    ; Emergency: Overheat repairers if critical
    if ${MyShip.ShieldPct} < 20
    {
        echo "CRITICAL: Low shields!"
        ; Would activate overheating here if supported
    }
}
```

---

## Timing and Cycle Management

### Module Cycles

**Most modules run in cycles**:
- Weapons: Fire every N seconds
- Mining lasers: Mine every N seconds
- Repairers: Repair every N seconds

**Cycle Time**:
```lavish
variable item module = ${MyShip.Module[HiSlot, 0]}
echo "Cycle time: ${module.Duration}ms"
```

### Waiting for Cycle Completion

**Pattern: Wait for mining cycle**:
```lavish
function WaitForMiningCycle()
{
    ; Check if any miner is active
    variable bool anyActive = FALSE
    variable int i

    for (i:Set[0]; ${i} < ${MyShip.HiSlots}; i:Inc)
    {
        variable item module = ${MyShip.Module[HiSlot, ${i}]}

        if !${module(exists)}
            continue

        if ${module.Name.Find["Mining"](exists)} || ${module.Name.Find["Strip Miner"](exists)}
        {
            if ${module.IsActive}
            {
                anyActive:Set[TRUE]
                break
            }
        }
    }

    if !${anyActive}
        return

    ; Wait for cycle to complete (miners go inactive)
    echo "Waiting for mining cycle..."

    variable int timeout = 0
    while ${anyActive} && ${timeout} < 300    ; 30 second timeout
    {
        wait 100

        anyActive:Set[FALSE]
        for (i:Set[0]; ${i} < ${MyShip.HiSlots}; i:Inc)
        {
            variable item module = ${MyShip.Module[HiSlot, ${i}]}

            if !${module(exists)}
                continue

            if ${module.Name.Find["Mining"](exists)} || ${module.Name.Find["Strip Miner"](exists)}
            {
                if ${module.IsActive}
                {
                    anyActive:Set[TRUE]
                    break
                }
            }
        }

        timeout:Inc
    }

    echo "Mining cycle complete"
}
```

---

## Performance Optimization

### Cache Module References

**Don't re-fetch modules repeatedly**:

```lavish
; BAD - Re-fetches module every check
while TRUE
{
    if ${MyShip.Module[HiSlot, 0].IsActive}
    {
        echo "Active: ${MyShip.Module[HiSlot, 0].Name}"
    }
    wait 10
}

; GOOD - Cache module reference
variable item miner = ${MyShip.Module[HiSlot, 0]}

while TRUE
{
    if ${miner.IsActive}
    {
        echo "Active: ${miner.Name}"
    }
    wait 10
}
```

### Minimize Module Iteration

**Don't iterate all modules every tick if not needed**:

```lavish
; BAD - Iterates every tick
while TRUE
{
    call ActivateAllMiningLasers
    wait 10    ; Checking 100 times per second!
}

; GOOD - Only activate when needed
variable bool lasersActive = TRUE

while TRUE
{
    if !${lasersActive}
    {
        call ActivateAllMiningLasers
        lasersActive:Set[TRUE]
    }

    ; Check if still active (less frequently)
    ; ...

    wait 10
}
```

---

## Critical Gotchas

### Gotcha 1: Slot Indexing is 0-Based

```lavish
; WRONG - 1-indexed assumption
variable item module = ${MyShip.Module[HiSlot, 1]}    ; This is SECOND high slot!

; RIGHT - 0-indexed
variable item module = ${MyShip.Module[HiSlot, 0]}    ; First high slot
```

### Gotcha 2: Module Can Despawn (Ship Destruction/Refit)

```lavish
; BAD - Module reference can become invalid
variable item miner = ${MyShip.Module[HiSlot, 0]}

; ... later ...
miner:Activate    ; CRASH if ship changed/destroyed!

; GOOD - Re-fetch or check exists
if ${miner(exists)}
{
    miner:Activate
}
```

### Gotcha 3: Activation is Asynchronous

```lavish
; BAD - Assumes instant activation
module:Activate
if ${module.IsActive}    ; Might be FALSE immediately after!
{
    echo "Active"
}

; GOOD - Wait for activation
module:Activate
wait 50

if ${module.IsActive}
{
    echo "Active"
}
```

### Gotcha 4: Charge Quantity Can Change Mid-Cycle

```lavish
; Charges consumed during cycles
; Check quantity BEFORE activation if critical
```

### Gotcha 5: Some Modules Cannot Be Activated via Script

```lavish
; Cloaking devices, jump drives, etc. may not respond to Activate
; Test each module type empirically
```

---

## Anti-Patterns

### Anti-Pattern 1: No State Check Before Activation

```lavish
; BAD
module:Activate

; GOOD
if ${module(exists)} && ${module.IsOnline} && !${module.IsActive}
{
    module:Activate
}
```

### Anti-Pattern 2: Activating Every Tick

```lavish
; BAD - Spamming activation
while TRUE
{
    module:Activate    ; Already active! Wasted calls!
    wait 10
}

; GOOD - Check state first
while TRUE
{
    if !${module.IsActive}
    {
        module:Activate
    }
    wait 10
}
```

### Anti-Pattern 3: No Ammo Check

```lavish
; BAD
module:Activate[${target}]
; Might fail if no ammo!

; GOOD
if ${module.Charge(exists)} && ${module.ChargeQuantity} > 0
{
    module:Activate[${target}]
}
else
{
    echo "Out of ammo!"
}
```

---

## Summary and Key Takeaways

### Essential Concepts

1. **Modules = Ship equipment** (weapons, miners, tank, utility)
2. **Slots**: High (offense), Mid (active tank/utility), Low (passive tank/damage mods)
3. **0-indexed slots** (unlike most ISXEVE collections!)
4. **Module states**: Offline, Online, Active, Deactivating, Reloading
5. **Activation**: `module:Activate` or `module:Activate[targetID]`
6. **Deactivation**: `module:Deactivate`

### Most Common Module Operations

```lavish
; Get module
variable item module = ${MyShip.Module[HiSlot, 0]}

; Check state
if ${module.IsOnline} && !${module.IsActive}

; Activate (untargeted)
module:Activate

; Activate (targeted)
module:Activate[${targetID}]

; Deactivate
module:Deactivate

; Check charge
if ${module.Charge(exists)}
    echo "Ammo: ${module.ChargeQuantity}"
```

### Critical Rules

1. **Slots are 0-indexed** (first slot = 0, not 1)
2. **Always check (exists)** before using module
3. **Always check IsOnline** before activating
4. **Check IsActive** to avoid redundant activations
5. **Wait after activation** before checking IsActive
6. **Monitor ammo/charges** for consumable modules
7. **Cache module references** for performance

---

## Wiki References

**Module Object**:
- `IsxeveWiki/ISXEVE/Module_(Object_Type).html` - Complete module reference

**Item Object** (modules inherit from item):
- `IsxeveWiki/ISXEVE/Item_(Object_Type).html` - Item object reference

**MyShip Module Access**:
- `IsxeveWiki/ISXEVE/MyShip_(Object_Type).html` - MyShip.Module reference

---

## Complete Guide to Cargo Management, Hangar Access, and Item Handling

---

## ⚠️ CRITICAL API CHANGE WARNING (July 2020) ⚠️

**THIS DOCUMENT CONTAINS DEPRECATED API EXAMPLES!**

Most cargo/inventory methods shown in this file were **DEPRECATED in July 2020** and should **NOT** be used in new scripts:

**DEPRECATED (Do Not Use):**
- `${MyShip.GetCargo}` → Use `EVEWindow[Inventory].Child[ShipCargo]:GetItems[index:item]`
- `${MyShip.Cargo[#]}` → Use iterator pattern on index:item
- `${MyShip.UsedOreHoldCapacity}` → Use `EVEWindow[Inventory].Child[ShipOreHold].UsedCapacity`
- `${MyShip.OreHoldCapacity}` → Use `EVEWindow[Inventory].Child[ShipOreHold].Capacity`
- All `Get*HoldCargo` methods → Use unified inventory window children

**MODERN API (Use This Instead):**
```lavish
; Open inventory window
if !${EVEWindow[Inventory](exists)}
    EVE:Execute[CmdOpenInventory]

wait 10

; Access cargo via inventory window
variable index:item CargoItems
EVEWindow[Inventory].Child[ShipCargo]:GetItems[CargoItems]

; Iterate using iterator
variable iterator CargoIterator
CargoItems:GetIterator[CargoIterator]
if ${CargoIterator:First(exists)}
{
    do
    {
        echo "${CargoIterator.Value.Name} x${CargoIterator.Value.Quantity}"
    }
    while ${CargoIterator:Next(exists)}
}

; Check capacity (still works)
echo "Cargo: ${MyShip.UsedCargoCapacity} / ${MyShip.CargoCapacity} m³"
```

**Why This Section Still Exists:**

1. **Understanding legacy scripts** - Evebot and other old scripts use this API
2. **Conceptual understanding** - The patterns are still valid, just the API changed
3. **Migration reference** - Helps convert old scripts to new API

**Recommendation**: Read this file to understand inventory concepts, then refer to:
- This guide - Modern cargo API examples throughout

**For New Scripts**: Use the modern `EVEWindow[Inventory]` pattern exclusively!

---

## Inventory and Cargo Systems

**Complete Guide to Inventory, Cargo Management, and Item Handling**

---


## Inventory System Overview

### EVE Inventory Structure

**Inventory locations**:
- **Ship cargo hold** - Main cargo bay
- **Ship ore hold** - Mining ships (Venture, Mining Barge, Exhumer)
- **Ship specialized holds** - Fleet hangar, fuel bay, command center hold
- **Station hangars** - Item hangar, ship hangar, corp hangars
- **Containers** - Cargo containers, secure containers, audit log containers
- **Wrecks** - Destroyed ships/NPCs (can be looted)

**Inventory hierarchy**:
```
Station
├── Item Hangar
├── Ship Hangar
├── Corp Hangars (1-7 divisions)
└── Active Ship
    ├── Cargo Hold
    ├── Ore Hold (mining ships)
    ├── Drones
    └── Modules
```

### ISXEVE Inventory Limitations

**CRITICAL**: ISXEVE has **LIMITED** inventory manipulation capabilities:

**What ISXEVE CAN do**:
- ✅ Query cargo contents (item names, quantities, types)
- ✅ Check cargo capacity (used, max, free space)
- ✅ Access ship cargo, ore hold
- ✅ Query hangar contents (limited)
- ✅ Open cargo windows

**What ISXEVE CANNOT do easily**:
- ❌ Drag/drop items (no direct support)
- ❌ Stack items (no direct method)
- ❌ Repackage items (no support)
- ❌ Reliable item movement between containers
- ❌ Market transactions (very limited)

**Result**: Most cargo bots focus on **reading cargo state** rather than manipulating items. Item movement typically done manually or via complex UI clicking.

**Wiki Reference**: `IsxeveWiki/ISXEVE/Item_(Object_Type).html`

---

## Item Object

### Getting Item Objects

**Items in cargo**:
```lavish
; Get cargo item count
variable int itemCount = ${MyShip.GetCargo}

; Iterate items (1-indexed!)
variable int i
for (i:Set[1]; ${i} <= ${itemCount}; i:Inc)
{
    variable item cargoItem = ${MyShip.Cargo[${i}]}

    if ${cargoItem(exists)}
    {
        echo "Item ${i}: ${cargoItem.Name} x${cargoItem.Quantity}"
    }
}
```

**By item ID**:
```lavish
variable int64 itemID = ...
variable item myItem = ${Item[${itemID}]}

if ${myItem(exists)}
{
    echo "Item: ${myItem.Name}"
}
```

### Item Members

**Identity**:
```lavish
variable item cargoItem = ${MyShip.Cargo[1]}

echo "Name: ${cargoItem.Name}"
echo "ID: ${cargoItem.ID}"
echo "Type ID: ${cargoItem.TypeID}"
echo "Group ID: ${cargoItem.GroupID}"
echo "Category ID: ${cargoItem.CategoryID}"
```

**Quantity and Volume**:
```lavish
echo "Quantity: ${cargoItem.Quantity}"
echo "Volume: ${cargoItem.Volume}"        ; Volume per unit
echo "Total Volume: ${Math.Calc[${cargoItem.Quantity} * ${cargoItem.Volume}]}"
```

**Location**:
```lavish
echo "Location ID: ${cargoItem.LocationID}"    ; Where item is located
echo "Container ID: ${cargoItem.ContainerID}"  ; Container holding item
```

**Modules** (if item is a module):
```lavish
; If item is fitted module, has additional module members
; See 06_Working_Examples.md for module examples

echo "Slot: ${cargoItem.Slot}"
echo "Is Active: ${cargoItem.IsActive}"
```

---

## Cargo Hold Access

### Cargo Capacity

**Check cargo capacity**:
```lavish
echo "Cargo Used: ${MyShip.UsedCargoCapacity} m³"
echo "Cargo Max: ${MyShip.CargoCapacity} m³"
echo "Cargo Free: ${MyShip.FreeCargoCapacity} m³"
```

**Calculate cargo percent full**:
```lavish
variable float cargoPercent = ${Math.Calc[${MyShip.UsedCargoCapacity} * 100.0 / ${MyShip.CargoCapacity}]}

echo "Cargo: ${cargoPercent}% full"

if ${cargoPercent} >= 90
{
    echo "WARNING: Cargo nearly full!"
}
```

### Iterating Cargo Items

**Pattern 1: Simple iteration**:
```lavish
function ListCargoContents()
{
    variable int itemCount = ${MyShip.GetCargo}

    if ${itemCount} == 0
    {
        echo "Cargo is empty"
        return
    }

    echo "Cargo contains ${itemCount} item stacks:"

    variable int i
    for (i:Set[1]; ${i} <= ${itemCount}; i:Inc)
    {
        variable item cargoItem = ${MyShip.Cargo[${i}]}

        if ${cargoItem(exists)}
        {
            echo "${i}: ${cargoItem.Name} x${cargoItem.Quantity} (${Math.Calc[${cargoItem.Quantity} * ${cargoItem.Volume}]} m³)"
        }
    }
}
```

**Pattern 2: Find specific item type**:
```lavish
function GetCargoQuantity(string itemName)
{
    variable int totalQuantity = 0
    variable int itemCount = ${MyShip.GetCargo}
    variable int i

    for (i:Set[1]; ${i} <= ${itemCount}; i:Inc)
    {
        variable item cargoItem = ${MyShip.Cargo[${i}]}

        if !${cargoItem(exists)}
            continue

        if ${cargoItem.Name.Equal["${itemName}"]}
        {
            totalQuantity:Inc[${cargoItem.Quantity}]
        }
    }

    return ${totalQuantity}
}

; Usage
variable int veldsparQuantity = ${GetCargoQuantity["Veldspar"]}
echo "Veldspar in cargo: ${veldsparQuantity}"
```

### Opening Cargo Window

**Open cargo hold UI**:
```lavish
EVE:Execute[CmdOpenCargoHold]
wait 20    ; Wait for window to open

; Verify window opened
if ${EVEWindow[ByItemID,${MyShip.ID}](exists)}
{
    echo "Cargo window opened"
}
```

---

## Cargo Capacity Management

### Cargo Full Detection

**Pattern 1: Percentage threshold**:
```lavish
function IsCargoFull(int threshold)
{
    variable float usedPercent = ${Math.Calc[${MyShip.UsedCargoCapacity} * 100.0 / ${MyShip.CargoCapacity}]}

    return ${Math.Calc[${usedPercent} >= ${threshold}]}
}

; Usage
if ${IsCargoFull[90]}
{
    echo "Cargo 90% full - returning to station"
    call ReturnToStation
}
```

**Pattern 2: Absolute space check**:
```lavish
function HasCargoSpace(float requiredSpace)
{
    return ${Math.Calc[${MyShip.FreeCargoCapacity} >= ${requiredSpace}]}
}

; Usage - Check if space for one more Veldspar cycle
variable float veldsparVolume = 16.0    ; Veldspar volume per unit
variable float miningYield = 500        ; Typical yield per cycle

if !${HasCargoSpace[${Math.Calc[${veldsparVolume} * ${miningYield}]}]}
{
    echo "No space for next mining cycle"
    call ReturnToStation
}
```

### Cargo Optimization

**Calculate optimal mining cycles before full**:
```lavish
function GetRemainingMiningCycles()
{
    ; Mining laser stats
    variable float oreVolumePerUnit = 16.0    ; Veldspar
    variable float yieldPerCycle = 500         ; Units per cycle
    variable float volumePerCycle = ${Math.Calc[${oreVolumePerUnit} * ${yieldPerCycle}]}

    ; Calculate cycles
    variable float remainingCycles = ${Math.Calc[${MyShip.FreeCargoCapacity} / ${volumePerCycle}]}

    return ${Math.Calc[${remainingCycles}]}    ; Returns int
}

; Usage
variable int cycles = ${GetRemainingMiningCycles}

if ${cycles} <= 1
{
    echo "Less than 1 cycle remaining - returning to station"
}
```

---

## Hangar Access

### Item Hangar (Station)

**IMPORTANT**: Direct hangar item access via ISXEVE is **VERY LIMITED**.

**Check if in station**:
```lavish
if !${Me.InStation}
{
    echo "Must be docked to access hangar"
    return
}
```

**Open item hangar UI**:
```lavish
EVE:Execute[CmdOpenHangarFloor]
wait 50    ; Wait for window to open

; Hangar window access is complex - see Evebot for examples
```

**Get hangar items** (limited support):
```lavish
; MyShip.GetHangarItems returns count
variable int hangarItemCount = ${MyShip.GetHangarItems}

echo "Items in hangar: ${hangarItemCount}"

; Accessing individual items is complex and unreliable
; Most bots avoid hangar item manipulation
```

### Ship Hangar

**Open ship hangar**:
```lavish
EVE:Execute[CmdOpenHangarFloor]
wait 50
```

**Switching ships** (limited ISXEVE support):
```lavish
; No direct "switch ship" method
; Must use UI interaction (complex and fragile)
```

### Corp Hangar

**Open corp hangar division**:
```lavish
; Division 1-7
EVE:Execute[OpenCorpHangar, 1]
wait 50

echo "Opened corp hangar division 1"
```

**Corp hangar access** is **VERY COMPLEX** in ISXEVE - Evebot has extensive corp hangar code but it's fragile.

---

## Specialized Cargo Bays

### Ore Hold (Mining Ships)

**Mining ships** (Venture, Mining Barge, Exhumer, some industrials) have specialized ore holds.

**Check ore hold capacity**:
```lavish
echo "Ore Hold Used: ${MyShip.UsedOreHoldCapacity} m³"
echo "Ore Hold Max: ${MyShip.OreHoldCapacity} m³"
echo "Ore Hold Free: ${Math.Calc[${MyShip.OreHoldCapacity} - ${MyShip.UsedOreHoldCapacity}]} m³"
```

**Check if ship has ore hold**:
```lavish
if ${MyShip.OreHoldCapacity} > 0
{
    echo "Ship has ore hold (${MyShip.OreHoldCapacity} m³)"
}
else
{
    echo "Ship uses standard cargo hold for ore"
}
```

**Mining bots**: Use ore hold capacity instead of cargo capacity:
```lavish
function IsOreHoldFull(int threshold)
{
    if ${MyShip.OreHoldCapacity} <= 0
    {
        ; No ore hold - use cargo instead
        return ${IsCargoFull[${threshold}]}
    }

    variable float usedPercent = ${Math.Calc[${MyShip.UsedOreHoldCapacity} * 100.0 / ${MyShip.OreHoldCapacity}]}

    return ${Math.Calc[${usedPercent} >= ${threshold}]}
}
```

### Other Specialized Bays

**Fleet Hangar** (Orca, Rorqual):
```lavish
echo "Fleet Hangar Capacity: ${MyShip.FleetHangarCapacity}"
; Access methods limited
```

**Specialized bay detection**:
```lavish
; Check various bay capacities to determine ship type
if ${MyShip.OreHoldCapacity} > 0
    echo "Mining ship"
if ${MyShip.FleetHangarCapacity} > 0
    echo "Command ship (Orca/Rorqual)"
```

---

## Moving Items (Limited Support)

### ISXEVE Item Movement Limitations

**ISXEVE does NOT have reliable item movement methods**:
- No drag/drop support
- No "Move to" method for most item types
- Must use UI clicking (fragile, version-dependent)

**What example scripts do**:
- **Evebot**: Complex UI window manipulation (very fragile)
- **Most bots**: Avoid item movement entirely
- **Workaround**: Manual item movement by player

### Theoretical Item Movement

**Some item types MAY have MoveTo method** (not guaranteed):
```lavish
variable item cargoItem = ${MyShip.Cargo[1]}

; Check if MoveTo exists (rare)
if ${cargoItem.MoveTo(exists)}
{
    ; Try to move to item hangar
    cargoItem:MoveTo[${MyShip.ID}, ItemHangar]
    wait 100
}
else
{
    echo "MoveTo method not available"
}
```

**Recommendation**: **Avoid scripting item movement**. Design bots to work without moving items, or have player manually move items.

---

## Loot Collection

### Looting Wrecks

**Wrecks** = Destroyed ships/NPCs that may contain loot.

**Find wrecks**:
```lavish
; Wrecks are CategoryID = 9
variable index:entity wrecks
EVE:GetEntities[wrecks, CategoryID = 9 && Distance < 50000]

echo "Found ${wrecks.Used} wrecks"
```

**Open wreck cargo**:
```lavish
variable entity wreck = ${wrecks.Get[1]}

if ${wreck(exists)}
{
    wreck:OpenCargo
    wait 50    ; Wait for cargo window to open

    echo "Opened ${wreck.Name} cargo"
}
```

**Check wreck contents** (LIMITED):
```lavish
; Wreck cargo access via ISXEVE is very limited
; Most bots just open wreck and let player manually loot
```

**Loot all** (UI method - unreliable):
```lavish
; No direct ISXEVE "loot all" method
; Would require UI window clicking (fragile)
```

**Practical approach**: Most combat bots **skip looting** or rely on player to manually loot.

---

## Item Stacking and Quantities

### Item Stacking

**EVE auto-stacks** identical items in cargo.

**Example**:
- 100x Veldspar (stack 1)
- 50x Veldspar (stack 2)
- Both appear as separate cargo items until manually stacked

**ISXEVE does NOT provide stack method** - must be done via UI.

### Counting Total Quantity

**Count all of item type** (across multiple stacks):
```lavish
function GetTotalCargoQuantity(string itemName)
{
    variable int total = 0
    variable int itemCount = ${MyShip.GetCargo}
    variable int i

    for (i:Set[1]; ${i} <= ${itemCount}; i:Inc)
    {
        variable item cargoItem = ${MyShip.Cargo[${i}]}

        if !${cargoItem(exists)}
            continue

        if ${cargoItem.Name.Equal["${itemName}"]}
        {
            total:Inc[${cargoItem.Quantity}]
        }
    }

    return ${total}
}

; Usage
variable int totalVeldspar = ${GetTotalCargoQuantity["Veldspar"]}
echo "Total Veldspar in cargo: ${totalVeldspar} units"
```

---

## Common Patterns from Example Scripts

### Pattern 1: Evebot - Mining Cargo Check

```lavish
; Simplified from Evebot
function Evebot_CheckCargoFull()
{
    ; Check ore hold if available
    if ${MyShip.OreHoldCapacity} > 0
    {
        variable float oreHoldPercent = ${Math.Calc[${MyShip.UsedOreHoldCapacity} * 100.0 / ${MyShip.OreHoldCapacity}]}

        if ${oreHoldPercent} >= 90
        {
            echo "Ore hold 90% full - returning to station"
            return TRUE
        }
    }
    else
    {
        ; Check cargo hold
        variable float cargoPercent = ${Math.Calc[${MyShip.UsedCargoCapacity} * 100.0 / ${MyShip.CargoCapacity}]}

        if ${cargoPercent} >= 90
        {
            echo "Cargo 90% full - returning to station"
            return TRUE
        }
    }

    return FALSE
}
```

### Pattern 2: Cargo Dump at Station

```lavish
; Common pattern - bot returns to station, player manually unloads
function UnloadCargoAtStation()
{
    ; Ensure docked
    if !${Me.InStation}
    {
        echo "ERROR: Not in station"
        return FALSE
    }

    ; Open cargo hold
    EVE:Execute[CmdOpenCargoHold]
    wait 50

    ; Open item hangar
    EVE:Execute[CmdOpenHangarFloor]
    wait 50

    ; Instruct player
    echo "========================================="
    echo "CARGO FULL - Please manually move ore from ship to hangar"
    echo "Press ENTER when complete..."
    echo "========================================="

    ; Wait for player confirmation
    ; (Some scripts pause here and wait for user input)

    return TRUE
}
```

### Pattern 3: Pre-Mining Capacity Check

```lavish
function CanMineCycle()
{
    ; Estimate volume of next mining cycle
    variable float oreVolumePerUnit = 16.0
    variable float estimatedYield = 500
    variable float cycleVolume = ${Math.Calc[${oreVolumePerUnit} * ${estimatedYield}]}

    ; Check if enough space
    if ${MyShip.FreeCargoCapacity} < ${cycleVolume}
    {
        echo "Insufficient cargo space for mining cycle"
        return FALSE
    }

    return TRUE
}
```

---

## Cargo Safety Patterns

### Pattern 1: Prevent Jettison

**Jettisoning cargo** = Dropping items into space (creates container).

**ISXEVE does NOT have direct jettison method** - but avoid UI clicks that might jettison.

**Safety**: Always check cargo space BEFORE activating mining lasers.

### Pattern 2: Cargo Overflow Protection

```lavish
function SafeMiningCycle(int64 asteroidID)
{
    ; Check cargo space before mining
    if !${CanMineCycle}
    {
        echo "Cargo full - aborting mining"
        return FALSE
    }

    ; Activate mining lasers
    call ActivateMiningLasers ${asteroidID}

    return TRUE
}
```

### Pattern 3: Emergency Cargo Dump

```lavish
function EmergencyCargoDump()
{
    ; If cargo full and can't dock, jettison least valuable items
    ; ISXEVE DOES NOT SUPPORT THIS - would need UI interaction
    ; Most bots just abort and return to station

    echo "EMERGENCY: Cargo full, returning to station immediately"
    call ReturnToStation
}
```

---

## Performance Optimization

### Cache Cargo Queries

**Cargo queries can be expensive**:

```lavish
; BAD - Query cargo multiple times
if ${MyShip.GetCargo} > 0
{
    echo "Cargo items: ${MyShip.GetCargo}"    ; Queried again!

    variable int i
    for (i:Set[1]; ${i} <= ${MyShip.GetCargo}; i:Inc)    ; Queried again!
    {
        ; ...
    }
}

; GOOD - Cache result
variable int cargoItemCount = ${MyShip.GetCargo}

if ${cargoItemCount} > 0
{
    echo "Cargo items: ${cargoItemCount}"

    variable int i
    for (i:Set[1]; ${i} <= ${cargoItemCount}; i:Inc)
    {
        ; ...
    }
}
```

### Minimize Cargo Iteration

**Don't iterate cargo every tick**:

```lavish
; BAD - Iterate every frame
while TRUE
{
    call ListCargoContents    ; Expensive!
    wait 10
}

; GOOD - Iterate only when needed
variable bool cargoChanged = FALSE

while TRUE
{
    ; Only list cargo if something changed
    if ${cargoChanged}
    {
        call ListCargoContents
        cargoChanged:Set[FALSE]
    }

    wait 10
}
```

---

## Critical Limitations

### Limitation 1: No Reliable Item Movement

**ISXEVE cannot reliably move items**:
- No drag/drop
- MoveTo method rarely available
- UI clicking is fragile

**Solution**: Design bots to avoid item movement, or use manual player intervention.

### Limitation 2: No Hangar Item Access

**Cannot reliably query hangar contents**:
- GetHangarItems returns count only
- Individual item access unreliable
- Corp hangar access very complex

**Solution**: Avoid hangar automation. Focus on ship cargo only.

### Limitation 3: No Loot Automation

**Cannot automate looting wrecks**:
- Can open wreck cargo
- Cannot "loot all" or move items from wreck to ship

**Solution**: Skip looting in combat bots, or pause for manual looting.

### Limitation 4: No Reprocessing/Market

**Cannot script reprocessing or market sales**:
- No reprocess method
- Market interaction very limited

**Solution**: Bots focus on collection, player handles selling/reprocessing.

---

## Gotchas and Edge Cases

### Gotcha 1: Cargo Item Indices Change

**Adding/removing items changes indices**:

```lavish
; DANGEROUS - Index changes if items added/removed
variable int itemCount = ${MyShip.GetCargo}

variable int i
for (i:Set[1]; ${i} <= ${itemCount}; i:Inc)
{
    variable item cargoItem = ${MyShip.Cargo[${i}]}

    ; If you remove this item, indices shift!
    ; Next iteration might skip items or crash
}
```

**Solution**: Snapshot item IDs if manipulating cargo during iteration (rare in ISXEVE).

### Gotcha 2: Volume Rounding

**Volume calculations can have floating-point errors**:

```lavish
; Account for small rounding errors
variable float requiredSpace = 1000.5
variable float freeSpace = ${MyShip.FreeCargoCapacity}

; BAD
if ${freeSpace} >= ${requiredSpace}

; BETTER - Add small buffer
if ${freeSpace} >= ${Math.Calc[${requiredSpace} + 0.1]}
```

### Gotcha 3: Ore Hold vs Cargo Hold Confusion

**Some ships have ore hold, some don't**:

```lavish
; MUST check which to use
function GetFreeMiningSpace()
{
    if ${MyShip.OreHoldCapacity} > 0
    {
        return ${Math.Calc[${MyShip.OreHoldCapacity} - ${MyShip.UsedOreHoldCapacity}]}
    }
    else
    {
        return ${MyShip.FreeCargoCapacity}
    }
}
```

### Gotcha 4: Session Change Invalidates Items

**Docking/undocking/jumping = session change**:

```lavish
variable item cargoItem = ${MyShip.Cargo[1]}

; ... dock ...

; cargoItem reference may now be invalid!
; Re-query after session change
```

---

## Summary and Key Takeaways

### What ISXEVE CAN Do

- ✅ Read cargo contents (item names, quantities, types)
- ✅ Check cargo capacity (used, free, max)
- ✅ Access ore hold capacity
- ✅ Open cargo windows
- ✅ Count cargo items

### What ISXEVE CANNOT Do

- ❌ Move items between containers (reliably)
- ❌ Stack items
- ❌ Loot wrecks (reliably)
- ❌ Access hangar items (reliably)
- ❌ Reprocess ore
- ❌ Sell items on market

### Essential Cargo Checks

```lavish
; Cargo capacity
echo "Cargo: ${MyShip.UsedCargoCapacity} / ${MyShip.CargoCapacity} m³"

; Cargo percent full
variable float percent = ${Math.Calc[${MyShip.UsedCargoCapacity} * 100.0 / ${MyShip.CargoCapacity}]}

; Ore hold (mining ships)
if ${MyShip.OreHoldCapacity} > 0
    echo "Ore Hold: ${MyShip.UsedOreHoldCapacity} / ${MyShip.OreHoldCapacity} m³"

; Iterate cargo
variable int count = ${MyShip.GetCargo}
variable int i
for (i:Set[1]; ${i} <= ${count}; i:Inc)
{
    echo "${MyShip.Cargo[${i}].Name} x${MyShip.Cargo[${i}].Quantity}"
}
```

### Critical Rules

1. **Always check cargo before mining** - Prevent overflow
2. **Use ore hold if available** - Mining ships have dedicated ore hold
3. **Avoid item movement scripts** - ISXEVE support is poor
4. **Design bots for read-only cargo** - Query state, don't manipulate
5. **Manual player intervention for unloading** - Most reliable approach

---

## Wiki References

**Item Object**:
- `IsxeveWiki/ISXEVE/Item_(Object_Type).html` - Complete item reference

**MyShip Cargo**:
- `IsxeveWiki/ISXEVE/MyShip_(Object_Type).html` - Cargo methods and members

**Examples**:
- Search Evebot for "GetCargo" and "CargoCapacity" for real patterns

---

## UI Windows and Menus

**Complete Guide to EVE UI Interaction via ISXEVE**

---


## EVE UI System Overview

### How EVE's UI Works

**Key Concepts**:
1. **Everything is a window** - Inventory, market, ship hangar, even the HUD elements
2. **Windows have names** - Usually descriptive (e.g., "inventory", "market", "fitting")
3. **Windows have unique IDs** - Multiple instances of same window type (e.g., multiple cargo containers)
4. **EVE:Execute triggers actions** - Cannot directly click/manipulate UI, must use Execute commands
5. **UI is asynchronous** - Commands don't complete instantly, must wait for state changes
6. **Server authority** - Many UI actions require server round-trip (delays)

### Window Types

**Common Window Types**:
- `inventory` - Inventory/cargo windows
- `market` - Market window
- `fitting` - Ship fitting window
- `station` - Station services
- `fleetwindow` - Fleet window
- `contractswindow` - Contracts
- `charactersheet` - Character sheet
- `overview` - Overview (target list)
- `selected` - Selected item window

**Window Hierarchy**:
```
EVE (top-level UI system)
└── Windows (collection of all open windows)
    ├── EVEWindow[Name] - Find by name
    ├── EVEWindow[ID] - Find by ID
    └── Methods to query, open, close
```

### UI Interaction Model

**Script CANNOT directly**:
- Click buttons
- Drag items
- Type in text fields
- Manipulate UI elements

**Script CAN**:
- Execute UI commands via `EVE:Execute[CommandName, parameters]`
- Query window state via `EVEWindow` objects
- Check if windows are open
- Get window positions/sizes
- Wait for UI state changes

---

## EVEWindow Object

### Getting EVEWindow Objects

**By Name**:
```lavish
variable evwindow inventoryWin = ${EVEWindow[inventory]}

if ${inventoryWin(exists)}
{
    echo "Inventory window exists"
}
```

**By Partial Name (First Match)**:
```lavish
; Finds first window with "invent" in name
variable evwindow win = ${EVEWindow[invent]}
```

**By ID**:
```lavish
variable int64 windowID = 123456789
variable evwindow win = ${EVEWindow[${windowID}]}
```

**All Windows**:
```lavish
; Get count of all open windows
variable int count = ${EVEWindow.Count}

; Iterate all windows
variable int i
for (i:Set[1]; ${i} <= ${EVEWindow.Count}; i:Inc)
{
    echo "Window ${i}: ${EVEWindow[${i}].Name}"
}
```

### EVEWindow Members

**Common Members**:
```lavish
variable evwindow win = ${EVEWindow[inventory]}

; Basic info
echo "Name: ${win.Name}"
echo "ID: ${win.ID}"
echo "Text: ${win.Text}"         ; Window title text
echo "Valid: ${win(exists)}"

; State
echo "Minimized: ${win.Minimized}"
echo "Visible: ${win.Visible}"

; Position/size
echo "X: ${win.X}"
echo "Y: ${win.Y}"
echo "Width: ${win.Width}"
echo "Height: ${win.Height}"
```

**Child Windows**:
```lavish
; Get child windows (e.g., tabs within inventory)
variable int childCount = ${win.ChildCount}

variable int i
for (i:Set[1]; ${i} <= ${childCount}; i:Inc)
{
    echo "Child ${i}: ${win.Child[${i}].Name}"
}
```

**Buttons**:
```lavish
; Get button count
echo "Buttons: ${win.ButtonCount}"

; Get button by index
echo "Button 1: ${win.Button[1].Text}"

; Check if button exists by text
if ${win.Button["OK"](exists)}
{
    echo "OK button exists"
}
```

### EVEWindow Methods

**Common Methods**:
```lavish
; Close window
win:Close

; Minimize window
win:Minimize

; Maximize window
win:Maximize

; Click button by text
win:ClickButton["OK"]

; Click button by index
win:ClickButton[1]

; Set window position
win:SetPosition[100, 200]

; Set window size
win:SetSize[400, 300]
```

### Window Existence Checking

**CRITICAL PATTERN** - Always check if window exists before using:

```lavish
variable evwindow win = ${EVEWindow[inventory]}

if !${win(exists)}
{
    echo "ERROR: Inventory window not found"
    return FALSE
}

; Safe to use win now
echo "Inventory: ${win.Name}"
```

**Why This Matters**:
- Attempting to use non-existent window members/methods = script crash
- Windows can close unexpectedly (player action, server disconnect, etc.)
- Window names can change between EVE versions

---

## EVE:Execute Command

### Overview

**EVE:Execute** is the **primary mechanism** for all UI interaction in ISXEVE.

**Syntax**:
```lavish
EVE:Execute[CommandName, param1, param2, ...]
```

**Key Rules**:
1. Commands are **case-insensitive** (CmdOpenInventory = cmdopeninventory)
2. Parameters vary by command (some have none, some require IDs/names)
3. **Execution is asynchronous** - command returns immediately, UI updates later
4. **No return value** - Cannot tell if command succeeded from Execute call
5. Must **validate result** by checking UI state after wait

### Common Execute Commands Categories

**Opening Windows**:
- `CmdOpenInventory`
- `CmdOpenMarket`
- `CmdOpenFitting`
- `CmdOpenStationPanel`
- `CmdOpenHangarFloor`
- `CmdOpenCargoHold`

**Ship/Module Control**:
- `CmdActivateModule`
- `CmdDeactivateModule`
- `CmdTogglePropulsionMod`
- `CmdStopShip`

**Navigation**:
- `CmdWarpToBookmark`
- `CmdDock`
- `CmdUndock`
- `CmdJumpThroughStargate`

**Item Interaction**:
- `OpenCorpHangar`
- `TakeItems`
- `TrashItems`

**Menu Actions**:
- `CmdShowInfo`
- `CmdLookAtItem`

### Finding Command Names

**Methods**:
1. **Wiki**: `IsxeveWiki/ISXEVE/EVE_(Object_Type).html` - Lists many commands
2. **Trial and Error**: Try logical names (CmdOpen*, Cmd*, Open*)
3. **Example Scripts**: Search Evebot/Yamfa/Tehbot for `EVE:Execute` usage
4. **InnerSpace Console**: Some commands shown in console when you manually use UI

**IMPORTANT**: Wiki documentation is **incomplete**. Many working commands are undocumented. Example scripts are the best reference.

---

## Common UI Commands Reference

### Inventory Commands

**Open Inventory**:
```lavish
EVE:Execute[CmdOpenInventory]
wait 20    ; Wait for window to open
```

**Open Cargo Hold**:
```lavish
EVE:Execute[CmdOpenCargoHold]
wait 20
```

**Open Ship Hangar** (in station):
```lavish
EVE:Execute[CmdOpenHangarFloor]
wait 20
```

**Open Corp Hangar**:
```lavish
; paramter = division number (1-7)
EVE:Execute[OpenCorpHangar, 1]
wait 20
```

**Open Container by ID**:
```lavish
variable int64 containerID = ${Entity[Name = "My Container"].ID}
EVE:Execute[OpenCargoHold, ${containerID}]
wait 20
```

### Module Commands

**Activate Module**:
```lavish
; Parameter = module slot index (0-7 for high, 0-7 for mid, 0-7 for low)
variable int slotID = ${MyShip.Module[=MiningLaser].ToItem.SlotID}
EVE:Execute[CmdActivateModule, ${slotID}]
```

**Deactivate Module**:
```lavish
EVE:Execute[CmdDeactivateModule, ${slotID}]
```

**Toggle Propulsion Mod** (afterburner/MWD):
```lavish
EVE:Execute[CmdTogglePropulsionMod]
```

### Ship Commands

**Stop Ship**:
```lavish
EVE:Execute[CmdStopShip]
```

**Align to Entity/Bookmark**:
```lavish
; Parameter = entity ID or bookmark ID
EVE:Execute[CmdAlignTo, ${entityID}]
```

### Docking/Undocking

**Dock at Station**:
```lavish
variable int64 stationID = ${Entity[Name = "Jita IV - Moon 4"](exists),ID}
EVE:Execute[CmdDock, ${stationID}]
```

**Undock**:
```lavish
EVE:Execute[CmdUndock]
```

**Jump Through Stargate**:
```lavish
variable int64 gateID = ${Entity[CategoryID = 6](exists),ID}    ; Category 6 = stargates
EVE:Execute[CmdJumpThroughStargate, ${gateID}]
```

### Warping

**Warp to Bookmark**:
```lavish
; Need bookmark ID (finding bookmarks is complex, covered in inventory section)
variable int64 bookmarkID = ${EVEWindow[ByCaption,inventory].GetBookmarkID["Mining Site"]}
EVE:Execute[CmdWarpToBookmark, ${bookmarkID}]
```

**Warp to Zero** (on selected entity):
```lavish
; Must have entity selected first
EVE:Execute[CmdWarpToSelection]
```

### Targeting

**Lock Target**:
```lavish
; Entity method is preferred, but can use Execute
variable int64 targetID = ${Entity[...].ID}
EVE:Execute[CmdLockTarget, ${targetID}]
```

**Unlock Target**:
```lavish
EVE:Execute[CmdUnlockTarget, ${targetID}]
```

**Unlock All**:
```lavish
EVE:Execute[CmdUnlockTargets]
```

### Menu/Info Commands

**Show Info Window**:
```lavish
variable int64 itemID = ${Entity[...].ID}
EVE:Execute[CmdShowInfo, ${itemID}]
wait 20    ; Wait for info window
```

**Look At** (camera focus):
```lavish
EVE:Execute[CmdLookAtItem, ${entityID}]
```

---

## Window Finding and Validation

### Finding Windows by Name

**Exact Name**:
```lavish
variable evwindow win = ${EVEWindow[inventory]}

if !${win(exists)}
{
    echo "Inventory window not found"
    return FALSE
}
```

**Partial Name** (First Match):
```lavish
; Matches "inventory", "inventoryPrimary", etc.
variable evwindow win = ${EVEWindow[invent]}
```

**By Caption** (Window Title):
```lavish
; Some windows identified by title text
variable evwindow win = ${EVEWindow[ByCaption,Item Hangar]}
```

### Finding Specific Inventory Windows

**Primary Inventory**:
```lavish
variable evwindow win = ${EVEWindow[inventory]}
```

**Cargo Hold**:
```lavish
; Cargo of YOUR ship
variable evwindow win = ${EVEWindow[MyShipCargo]}

; Alternative
win:Set[${EVEWindow[ByItemID,${MyShip.ID}]}]
```

**Ship Hangar** (in station):
```lavish
variable evwindow win = ${EVEWindow[ShipHangar]}

; Alternative
win:Set[${EVEWindow[ByCaption,Ship Hangar]}]
```

**Corp Hangar Division**:
```lavish
; Find corp hangar division 1
variable evwindow win = ${EVEWindow[ByCaption,Corporation Hangar]}

; Check child windows for specific division
; (This is complex - Evebot has extensive corp hangar code)
```

### Validating Window State

**Check if Open and Ready**:
```lavish
function IsWindowReady(string windowName)
{
    variable evwindow win = ${EVEWindow[${windowName}]}

    if !${win(exists)}
        return FALSE

    if ${win.Minimized}
        return FALSE

    ; Some windows need additional checks
    ; (e.g., market window loaded, inventory items loaded)

    return TRUE
}

; Usage
if ${IsWindowReady["inventory"]}
{
    echo "Inventory ready"
}
```

**Wait for Window to Open**:
```lavish
function WaitForWindow(string windowName, int timeoutSeconds)
{
    variable int startTime = ${Script.RunningTime}
    variable int elapsed

    while TRUE
    {
        if ${EVEWindow[${windowName}](exists)}
        {
            echo "Window ${windowName} opened"
            return TRUE
        }

        elapsed:Set[${Math.Calc[(${Script.RunningTime} - ${startTime}) / 1000]}]
        if ${elapsed} >= ${timeoutSeconds}
        {
            echo "Timeout waiting for ${windowName}"
            return FALSE
        }

        wait 100
    }
}

; Usage
EVE:Execute[CmdOpenInventory]
if !${WaitForWindow["inventory", 10]}
{
    echo "Failed to open inventory"
    return
}
```

---

## Opening and Closing Windows

### Opening Windows Safely

**Pattern**:
1. Check if already open
2. If not, execute open command
3. Wait for window to appear
4. Validate window is ready

**Example**:
```lavish
function OpenInventory()
{
    ; Already open?
    if ${EVEWindow[inventory](exists)}
    {
        echo "Inventory already open"
        return TRUE
    }

    ; Open it
    echo "Opening inventory..."
    EVE:Execute[CmdOpenInventory]

    ; Wait for it
    variable int attempts = 0
    while ${attempts} < 20
    {
        wait 100
        if ${EVEWindow[inventory](exists)}
        {
            echo "Inventory opened successfully"
            return TRUE
        }
        attempts:Inc
    }

    echo "ERROR: Failed to open inventory"
    return FALSE
}
```

### Closing Windows

**Close by EVEWindow Object**:
```lavish
variable evwindow win = ${EVEWindow[inventory]}

if ${win(exists)}
{
    win:Close
    echo "Closed inventory"
}
```

**Close All Windows of Type** (useful for cleanup):
```lavish
function CloseAllWindowsByName(string windowName)
{
    variable int i
    variable int count = ${EVEWindow.Count}

    ; Iterate backwards (closing changes indices)
    for (i:Set[${count}]; ${i} >= 1; i:Dec)
    {
        if ${EVEWindow[${i}].Name.Find[${windowName}](exists)}
        {
            echo "Closing ${EVEWindow[${i}].Name}"
            EVEWindow[${i}]:Close
            wait 10
        }
    }
}

; Usage
call CloseAllWindowsByName "inventory"
```

### Cleanup Pattern (atexit)

```lavish
function atom atexit()
{
    echo "Cleaning up UI..."

    ; Close windows we opened
    if ${EVEWindow[inventory](exists)}
        EVEWindow[inventory]:Close

    if ${EVEWindow[market](exists)}
        EVEWindow[market]:Close

    ; Stop ship (safety)
    if ${Me.InSpace}
        EVE:Execute[CmdStopShip]
}
```

---

## Clicking and Button Interaction

### Clicking Buttons in Windows

**Click by Button Text**:
```lavish
variable evwindow win = ${EVEWindow[inventory]}

if ${win.Button["OK"](exists)}
{
    win:ClickButton["OK"]
    echo "Clicked OK"
}
```

**Click by Button Index**:
```lavish
; Click first button
win:ClickButton[1]
```

**Validate Button Exists Before Clicking**:
```lavish
function ClickButton(string windowName, string buttonText)
{
    variable evwindow win = ${EVEWindow[${windowName}]}

    if !${win(exists)}
    {
        echo "ERROR: Window ${windowName} not found"
        return FALSE
    }

    if !${win.Button[${buttonText}](exists)}
    {
        echo "ERROR: Button '${buttonText}' not found in ${windowName}"
        return FALSE
    }

    win:ClickButton[${buttonText}]
    echo "Clicked '${buttonText}' in ${windowName}"
    return TRUE
}

; Usage
call ClickButton "inventory" "OK"
```

### Finding Buttons

**List All Buttons**:
```lavish
function ListWindowButtons(string windowName)
{
    variable evwindow win = ${EVEWindow[${windowName}]}

    if !${win(exists)}
    {
        echo "Window not found"
        return
    }

    echo "Buttons in ${windowName}:"
    variable int i
    for (i:Set[1]; ${i} <= ${win.ButtonCount}; i:Inc)
    {
        echo "  ${i}: ${win.Button[${i}].Text}"
    }
}

; Usage (for debugging)
call ListWindowButtons "inventory"
```

### Button Click Timing

**CRITICAL**: Button clicks are **asynchronous** and may trigger server actions.

```lavish
; BAD - No wait after click
win:ClickButton["OK"]
; Window might still be open here!

; GOOD - Wait for UI to respond
win:ClickButton["OK"]
wait 50    ; Minimum wait for UI update

; BETTER - Wait and validate
win:ClickButton["OK"]
wait 50

variable int attempts = 0
while ${win(exists)} && ${attempts} < 20
{
    wait 100
    attempts:Inc
}

if ${win(exists)}
{
    echo "WARNING: Window still open after clicking OK"
}
```

---

## Menu System Navigation

### Right-Click Menus (Context Menus)

**IMPORTANT**: ISXEVE has **limited** support for right-click menu interaction.

**Opening Context Menu on Entity** (unreliable):
```lavish
; This is NOT well-supported
; Prefer using direct commands instead
```

**Alternative: Use Direct Commands**:
```lavish
; Instead of right-click -> "Look At"
EVE:Execute[CmdLookAtItem, ${entityID}]

; Instead of right-click -> "Show Info"
EVE:Execute[CmdShowInfo, ${entityID}]

; Instead of right-click -> "Lock Target"
Entity[${entityID}]:LockTarget
```

### Neocom Menu (Left Sidebar)

**Opening Neocom Items**:
```lavish
; Most Neocom items have dedicated Execute commands

; Inventory
EVE:Execute[CmdOpenInventory]

; Market
EVE:Execute[CmdOpenMarket]

; Fitting
EVE:Execute[CmdOpenFitting]

; Character sheet
EVE:Execute[CmdOpenCharactersheet]
```

---

## Inventory Window Interaction

### Inventory Window Structure

**Inventory Hierarchy**:
```
Inventory Window
├── Tree (left side) - Locations
│   ├── Item Hangar
│   ├── Ship Hangar
│   ├── Corp Hangars (1-7)
│   └── Active Ship (cargo, drones, etc.)
└── Content (right side) - Items in selected location
```

### Opening Inventory Locations

**Open Inventory**:
```lavish
EVE:Execute[CmdOpenInventory]
wait 20
```

**Switch to Item Hangar**:
```lavish
; This is complex - inventory locations accessed via ChildWindow
; Example from Evebot:
variable evwindow invWin = ${EVEWindow[inventory]}
variable evwindow itemHangar = ${invWin.ChildWindow[itemHangar]}

if ${itemHangar(exists)}
{
    echo "Item hangar accessible"
}
```

**Open Ship Hangar**:
```lavish
EVE:Execute[CmdOpenHangarFloor]
wait 20
```

**Open Corp Hangar Division**:
```lavish
; Division 1-7
EVE:Execute[OpenCorpHangar, 1]
wait 20
```

### Accessing Inventory Items

**CRITICAL**: Direct item manipulation is **complex** and **fragile**.

**⚠️ WARNING:** Old `MyShip.GetCargo` / `MyShip.Cargo[#]` are DEPRECATED (July 2020). Use modern EVEWindow[Inventory] API.

**Get Items in Cargo** (Modern API - July 2020+):
```lavish
; MODERN API: Use EVEWindow[Inventory].Child[ShipCargo]
if !${EVEWindow[Inventory](exists)}
{
    EVE:Execute[CmdOpenInventory]
    wait 20
}

if ${EVEWindow[Inventory](exists)}
{
    variable index:item CargoItems
    EVEWindow[Inventory].Child[ShipCargo]:GetItems[CargoItems]

    variable iterator Item
    CargoItems:GetIterator[Item]

    if ${Item:First(exists)}
    {
        do
        {
            echo "Item: ${Item.Value.Name} (${Item.Value.Quantity})"
        }
        while ${Item:Next(exists)}
    }
}
```

**Inventory Window Item Access** (Advanced, from Evebot):
```lavish
; This is VERY complex - Evebot has entire modules for inventory management
; Involves ChildWindow access, item iteration, etc.
; DEPRECATED: MyShip.Cargo, MyShip.GetHangarItems (July 2020)
; MODERN: Use EVEWindow[Inventory].Child[name] pattern shown above
```

### Moving Items (Drag/Drop Simulation)

**ISXEVE does NOT support drag/drop**. Must use game's right-click menu or hotkeys.

**Workarounds**:
1. **Use item.MoveTo method** (if item type supports it)
2. **Use EVE:Execute[OpenCargoHold]** to open destination, then item methods
3. **Some Evebot patterns** use complex window/button clicking (fragile)

**Example from Evebot** (simplified):
```lavish
; This is an approximation - real Evebot code is much more complex
function MoveItemToCargoHold(int64 itemID)
{
    ; Get item
    variable item myItem = ${Item[${itemID}]}

    if !${myItem(exists)}
        return FALSE

    ; Use MoveTo if available (not always available)
    if ${myItem.MoveTo(exists)}
    {
        myItem:MoveTo[${MyShip.ID}, CargoHold]
        wait 50
        return TRUE
    }

    ; Otherwise, must use UI clicking (complex, fragile)
    echo "Cannot move item - no MoveTo method"
    return FALSE
}
```

---

## Market Window Interaction

### Opening Market

```lavish
EVE:Execute[CmdOpenMarket]
wait 50    ; Market window is slow to load
```

**Validate Market Loaded**:
```lavish
function IsMarketReady()
{
    variable evwindow market = ${EVEWindow[market]}

    if !${market(exists)}
        return FALSE

    ; Additional checks (market data loaded, etc.) would go here
    ; Evebot has complex market validation

    return TRUE
}
```

### Market Commands

**IMPORTANT**: Market interaction via ISXEVE is **extremely limited** and **fragile**.

**Why Market Bots are Difficult**:
- Market window has complex internal structure (tabs, filters, item list, etc.)
- Few dedicated Execute commands for market actions
- Clicking specific items in market list is unreliable
- Price data access is limited

**What Example Scripts Do**:
- Evebot has market modules, but they're **complex** and **fragile**
- Most avoid market automation entirely
- Prefer using API/web for market data (outside ISXEVE scope)

**Basic Market Pattern** (from Evebot, simplified):
```lavish
function OpenMarketForItem(string itemName)
{
    ; Open market
    if !${EVEWindow[market](exists)}
    {
        EVE:Execute[CmdOpenMarket]
        wait 100
    }

    ; Beyond this, item searching/filtering is very complex
    ; and not well-supported by ISXEVE

    echo "Market opened, but item search is not automated"
}
```

---

## Station Services

### Opening Station Panel

```lavish
; In station only
if ${Me.InStation}
{
    EVE:Execute[CmdOpenStationPanel]
    wait 20
}
```

**Station Services Available**:
- Repair
- Reprocessing
- Insurance
- Clone services
- Corporation offices
- (Access methods vary, poorly documented)

### Repair Services

**Opening Repair Window**:
```lavish
; Must be in station
EVE:Execute[CmdOpenRepairShop]
wait 50
```

**Repairing** (using window):
```lavish
variable evwindow repair = ${EVEWindow[repairshop]}

if ${repair(exists)}
{
    ; Click "Repair All" button (if it exists)
    if ${repair.Button["Repair All"](exists)}
    {
        repair:ClickButton["Repair All"]
        wait 50
    }
}
```

### Reprocessing

**Opening Reprocessing**:
```lavish
EVE:Execute[CmdOpenReprocessingPlant]
wait 50
```

### Insurance

**Limited support**. Most bots skip insurance automation.

---

## UI Timing and Wait Patterns

### Why Timing Matters

**UI Operations are Asynchronous**:
1. `EVE:Execute` returns immediately (doesn't wait for completion)
2. UI updates after network round-trip (server authority)
3. Complex windows load in stages (window opens, then data loads)
4. Button clicks trigger actions that take time

**Failure Modes if No Waits**:
- Script checks for window before it appears (false negative)
- Script clicks button before window fully loaded (click fails silently)
- Script tries to access items before inventory data loaded (crashes)

### Minimum Wait Times (Empirical)

**After Opening Window**:
```lavish
EVE:Execute[CmdOpenInventory]
wait 20    ; Minimum for simple windows

EVE:Execute[CmdOpenMarket]
wait 50    ; Market is slower

EVE:Execute[CmdOpenFitting]
wait 30    ; Fitting window has complex load
```

**After Clicking Button**:
```lavish
win:ClickButton["OK"]
wait 50    ; Allow UI to process
```

**After Module Activation**:
```lavish
EVE:Execute[CmdActivateModule, ${slotID}]
wait 10    ; Module activation is fast (local UI update)
```

**After Warp Command**:
```lavish
EVE:Execute[CmdWarpToBookmark, ${bookmarkID}]
wait 100   ; Ship state changes (server round-trip)

; Then wait for warp to complete (separate check)
while !${MyShip.ToEntity.Mode} == 3    ; Mode 3 = Warping
{
    wait 100
}
echo "Warp initiated"
```

### Timeout Pattern (Essential)

```lavish
function WaitForWindowWithTimeout(string windowName, int timeoutSeconds)
{
    variable int startTime = ${Script.RunningTime}
    variable int elapsed

    while TRUE
    {
        ; Check condition
        if ${EVEWindow[${windowName}](exists)}
            return TRUE

        ; Check timeout
        elapsed:Set[${Math.Calc[(${Script.RunningTime} - ${startTime}) / 1000]}]
        if ${elapsed} >= ${timeoutSeconds}
        {
            echo "TIMEOUT: Window ${windowName} did not appear in ${timeoutSeconds}s"
            return FALSE
        }

        wait 100
    }
}

; Usage
EVE:Execute[CmdOpenInventory]
if !${WaitForWindowWithTimeout["inventory", 10]}
{
    echo "Failed to open inventory"
    return
}
```

### Wait Between UI Actions (Rate Limiting)

**Important**: Don't spam UI commands too fast or game may lag/ignore commands.

```lavish
; BAD - Commands too fast
EVE:Execute[CmdOpenInventory]
EVE:Execute[CmdOpenMarket]
EVE:Execute[CmdOpenFitting]
; All three might fail!

; GOOD - Wait between commands
EVE:Execute[CmdOpenInventory]
wait 50

EVE:Execute[CmdOpenMarket]
wait 50

EVE:Execute[CmdOpenFitting]
wait 50
```

---

## UI State Validation

### Always Validate Before and After

**Before UI Action**:
```lavish
function ClickButtonSafe(string windowName, string buttonText)
{
    ; Validate window exists
    variable evwindow win = ${EVEWindow[${windowName}]}
    if !${win(exists)}
    {
        echo "ERROR: Window not found"
        return FALSE
    }

    ; Validate button exists
    if !${win.Button[${buttonText}](exists)}
    {
        echo "ERROR: Button not found"
        return FALSE
    }

    ; Safe to click
    win:ClickButton[${buttonText}]
    return TRUE
}
```

**After UI Action**:
```lavish
function CloseWindowSafe(string windowName)
{
    variable evwindow win = ${EVEWindow[${windowName}]}

    if !${win(exists)}
    {
        echo "Window already closed"
        return TRUE
    }

    ; Close it
    win:Close
    wait 50

    ; Validate it closed
    if ${EVEWindow[${windowName}](exists)}
    {
        echo "WARNING: Window did not close"
        return FALSE
    }

    echo "Window closed successfully"
    return TRUE
}
```

### State Machine Pattern for Complex UI

```lavish
objectdef obj_UIStateMachine
{
    variable string state = "IDLE"

    method SetState(string newState)
    {
        echo "UI State: ${state} -> ${newState}"
        state:Set["${newState}"]
    }

    method ProcessMarketPurchase()
    {
        if ${state.Equal["IDLE"]}
        {
            echo "Opening market..."
            EVE:Execute[CmdOpenMarket]
            This:SetState["OPENING_MARKET"]
            return
        }

        if ${state.Equal["OPENING_MARKET"]}
        {
            if ${EVEWindow[market](exists)}
            {
                echo "Market opened"
                This:SetState["SEARCHING_ITEM"]
            }
            return
        }

        if ${state.Equal["SEARCHING_ITEM"]}
        {
            ; Complex search logic here...
            This:SetState["CLICKING_BUY"]
            return
        }

        ; etc.
    }
}
```

---

## Common Patterns from Example Scripts

### Pattern 1: Safe Window Open (Evebot)

```lavish
function Evebot_OpenInventory()
{
    if ${EVEWindow[inventory](exists)}
    {
        return TRUE
    }

    echo "Opening inventory"
    EVE:Execute[CmdOpenInventory]

    variable int counter = 0
    while !${EVEWindow[inventory](exists)} && ${counter} < 50
    {
        wait 10
        counter:Inc
    }

    if !${EVEWindow[inventory](exists)}
    {
        echo "ERROR: Could not open inventory"
        return FALSE
    }

    return TRUE
}
```

### Pattern 2: Inventory Item Iteration (Modern API)

**⚠️ NOTE:** Original Yamfa uses deprecated `MyShip.GetCargo`. Updated to modern API below.

```lavish
function ProcessCargoItems()
{
    ; Open inventory window
    if !${EVEWindow[Inventory](exists)}
    {
        EVE:Execute[CmdOpenInventory]
        wait 20
    }

    if !${EVEWindow[Inventory](exists)}
    {
        echo "ERROR: Cannot open inventory"
        return
    }

    ; MODERN API: Get cargo items via inventory window
    variable index:item CargoItems
    EVEWindow[Inventory].Child[ShipCargo]:GetItems[CargoItems]

    echo "Cargo has ${CargoItems.Used} items"

    variable iterator Item
    CargoItems:GetIterator[Item]

    if ${Item:First(exists)}
    {
        do
        {
            echo "Item: ${Item.Value.Name} (Qty: ${Item.Value.Quantity})"
        }
        while ${Item:Next(exists)}
    }
}
```

### Pattern 3: Button Click with Retry (Tehbot - Simplified)

```lavish
function ClickButtonWithRetry(string windowName, string buttonText, int maxAttempts)
{
    variable int attempt = 0

    while ${attempt} < ${maxAttempts}
    {
        attempt:Inc

        variable evwindow win = ${EVEWindow[${windowName}]}

        if !${win(exists)}
        {
            echo "Attempt ${attempt}: Window not found, waiting..."
            wait 100
            continue
        }

        if !${win.Button[${buttonText}](exists)}
        {
            echo "Attempt ${attempt}: Button not found, waiting..."
            wait 100
            continue
        }

        ; Click and return success
        win:ClickButton[${buttonText}]
        echo "Button clicked successfully"
        return TRUE
    }

    echo "ERROR: Failed to click button after ${maxAttempts} attempts"
    return FALSE
}
```

---

## Critical Gotchas and Issues

### Gotcha 1: Window Names Change Between EVE Versions

**Problem**: CCP sometimes changes window internal names.

**Solution**: Use multiple name attempts or ByCaption:
```lavish
variable evwindow win = ${EVEWindow[inventory]}

if !${win(exists)}
{
    win:Set[${EVEWindow[ByCaption,Inventory]}]
}

if !${win(exists)}
{
    echo "ERROR: Cannot find inventory window"
}
```

### Gotcha 2: Multiple Windows with Same Name

**Problem**: Opening multiple cargo containers = multiple "inventory" windows.

**Solution**: Use ByItemID or track window IDs:
```lavish
; Open container and get its specific window
variable int64 containerID = ${Entity[Name = "Container"].ID}

EVE:Execute[OpenCargoHold, ${containerID}]
wait 20

; Find window by item ID
variable evwindow containerWin = ${EVEWindow[ByItemID,${containerID}]}

if ${containerWin(exists)}
{
    echo "Found container window: ${containerWin.Name}"
}
```

### Gotcha 3: Buttons Without Text

**Problem**: Some buttons have no text (icon-only).

**Solution**: Use button index (fragile) or avoid:
```lavish
; Click first button (might be "OK")
win:ClickButton[1]
```

### Gotcha 4: Windows Not Fully Loaded

**Problem**: Window appears but data not loaded yet.

**Solution**: Add extra waits or check data presence:
```lavish
EVE:Execute[CmdOpenMarket]
wait 50    ; Initial wait

; Additional validation
variable evwindow market = ${EVEWindow[market]}
variable int attempts = 0

; Wait for market to have content (example - actual check varies)
while ${attempts} < 20
{
    ; Actual content check would go here (complex)
    wait 100
    attempts:Inc
}
```

### Gotcha 5: Race Conditions (Clicking Too Soon)

**Problem**: Script clicks before UI processes previous action.

**Example**:
```lavish
; BAD
win:ClickButton["Next"]
win:ClickButton["Confirm"]    ; Might click before "Next" processed!

; GOOD
win:ClickButton["Next"]
wait 50
win:ClickButton["Confirm"]
```

### Gotcha 6: Server Lag

**Problem**: High latency = longer waits needed.

**Solution**: Increase timeouts, add retry logic:
```lavish
; If players have high ping, increase waits
variable int UI_WAIT = 50    ; Normal
variable int UI_WAIT = 100   ; High latency

EVE:Execute[CmdOpenInventory]
wait ${UI_WAIT}
```

---

## Anti-Patterns to Avoid

### Anti-Pattern 1: No Wait After Execute

**Bad**:
```lavish
EVE:Execute[CmdOpenInventory]
if ${EVEWindow[inventory](exists)}    ; Will be FALSE!
{
    echo "Inventory open"
}
```

**Good**:
```lavish
EVE:Execute[CmdOpenInventory]
wait 20

if ${EVEWindow[inventory](exists)}
{
    echo "Inventory open"
}
```

### Anti-Pattern 2: No Existence Check

**Bad**:
```lavish
variable evwindow win = ${EVEWindow[inventory]}
echo "Name: ${win.Name}"    ; CRASH if window doesn't exist!
```

**Good**:
```lavish
variable evwindow win = ${EVEWindow[inventory]}

if !${win(exists)}
{
    echo "ERROR: Window not found"
    return
}

echo "Name: ${win.Name}"    ; Safe
```

### Anti-Pattern 3: No Timeout on Wait Loops

**Bad**:
```lavish
while !${EVEWindow[inventory](exists)}
{
    wait 100
}
; Infinite loop if window never appears!
```

**Good**:
```lavish
variable int attempts = 0
while !${EVEWindow[inventory](exists)} && ${attempts} < 50
{
    wait 100
    attempts:Inc
}

if !${EVEWindow[inventory](exists)}
{
    echo "ERROR: Timeout"
}
```

### Anti-Pattern 4: Relying on Fixed Button Indices

**Bad**:
```lavish
win:ClickButton[3]    ; What if CCP rearranges buttons?
```

**Good**:
```lavish
win:ClickButton["OK"]    ; Text is more stable
```

### Anti-Pattern 5: Complex UI Automation Without State Tracking

**Bad**:
```lavish
; Trying to do complex market purchase in linear code
EVE:Execute[CmdOpenMarket]
; ...100 lines of brittle clicking...
```

**Good**:
```lavish
; Use state machine pattern (see earlier section)
; Allows recovery from failures, retries, etc.
```

---

## Summary and Key Takeaways

### Essential Rules for UI Interaction

1. **Always use EVE:Execute** - Cannot directly manipulate UI
2. **Always wait after Execute** - UI updates are asynchronous
3. **Always validate window existence** - Scripts crash if window doesn't exist
4. **Always use timeouts** - Don't create infinite loops
5. **Always check state after actions** - Execute doesn't return success/failure

### Common UI Workflow

```lavish
; 1. Open window
EVE:Execute[CmdOpenInventory]
wait 20

; 2. Validate it opened
if !${EVEWindow[inventory](exists)}
{
    echo "ERROR: Failed to open"
    return FALSE
}

; 3. Perform action
EVEWindow[inventory]:ClickButton["OK"]
wait 50

; 4. Validate result
if ${EVEWindow[inventory](exists)}
{
    echo "WARNING: Window still open"
}
```

### Most Critical Commands

- `CmdOpenInventory` - Inventory access
- `CmdOpenCargoHold` - Cargo access
- `CmdOpenHangarFloor` - Ship hangar
- `CmdActivateModule` / `CmdDeactivateModule` - Module control
- `CmdDock` / `CmdUndock` - Station interaction
- `CmdStopShip` - Emergency stop

### Most Critical Objects

- `${EVEWindow[name]}` - Find windows
- `${EVEWindow[...](exists)}` - Validate existence
- `EVEWindow:Close` - Close windows
- `EVEWindow:ClickButton` - Click buttons

### Timing Guidelines

- Simple window open: 20ms wait
- Complex window (market, fitting): 50ms+ wait
- After button click: 50ms wait
- After warp/dock command: 100ms+ wait
- Always add timeout to loops (10-20 second max)

---

## Wiki References

**EVE Object**:
- `IsxeveWiki/ISXEVE/EVE_(Object_Type).html` - EVE:Execute command reference

**EVEWindow Object**:
- `IsxeveWiki/ISXEVE/EVEWindow_(Object_Type).html` - Window object reference

**Common Commands** (scattered across wiki):
- Search for "Cmd*" in wiki for command names
- Example scripts are better reference than wiki (wiki is incomplete)

---

## Fleet and Social Systems

**Complete Guide to Fleet Management, Social Interactions, and Yamfa-Style Coordination**

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

**See [07_Advanced_Patterns_And_Examples.md](07_Advanced_Patterns_And_Examples.md) for relay atom details**.

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

## Wiki References

**Fleet Object**:
- `IsxeveWiki/ISXEVE/Fleet_(Object_Type).html` - Fleet object reference

**Fleet Member Object**:
- `IsxeveWiki/ISXEVE/FleetMember_(Object_Type).html` - Fleet member reference

**Local Object**:
- `IsxeveWiki/ISXEVE/Local_(Object_Type).html` - Local chat reference

**Relay System** (see [07_Advanced_Patterns_And_Examples.md](07_Advanced_Patterns_And_Examples.md)):
- `LavishScriptWiki/LavishScript/Commands/relay.html` - Relay command reference

---


## Introduction to ISXEVE

### What is ISXEVE?

**ISXEVE** is a C++/C# extension for InnerSpace that provides read/write access to EVE Online's memory.

**Key Characteristics:**
- Memory-based (no network calls)
- Real-time game state access
- Both LavishScript and .NET support
- Closed-source, legacy codebase
- No updates expected

**Version Check:**

```lavish
; LavishScript
if ${ISXEVE(exists)} && ${ISXEVE.IsReady}
{
    echo "ISXEVE version: ${ISXEVE.Version}"
}
```

```csharp
// C# .NET
if (ISXEVE.IsReady)
{
    Console.WriteLine($"ISXEVE version: {ISXEVE.Version}");
}
```

### Data Type Convention

ISXEVE uses a specific naming convention:

| Type | Example | Description |
|------|---------|-------------|
| **Singular** | `Entity` | Single object |
| **Plural** | `Entities` | Collection/Iterator |
| **Index** | `Entity[123]` | Access by ID |
| **TLO** | `EVE`, `Me`, `MyShip` | Top-Level Object (global) |

### DataType Inheritance

**CRITICAL CONCEPT:** ISXEVE datatypes use inheritance, meaning child types have ALL members of their parent types.

**Major Inheritance Chains:**
```
character → pilot → being
  └─ ${Me} is a character, which inherits all pilot and being members

module → item → iteminfo
  └─ ${Module} has all item properties (Name, TypeID, etc.)

fleetmember → pilot → being
  └─ Fleet members have all pilot/being properties

entity → (entitywormhole, entityplayerstructure, attacker, jammer)
  └─ Specialized entity types with extra properties
```

**Example - Understanding ${Me}:**
```lavish
; Me is character type, but inherits from pilot and being
echo ${Me.Name}              ; character member
echo ${Me.CharID}            ; pilot member (inherited)
echo ${Me.ToEntity}          ; being member (inherited)
```

**Why this matters:** If documentation says "pilot has member X", then character (which inherits from pilot) also has member X. This explains why ${Me} has more members than just those listed as "character" members.

**Type Conversions:**
```lavish
${Me.ToEntity}           ; character → entity
${Me.ToFleetMember}      ; character → fleetmember (if in fleet)
${Entity.ToPilot}        ; entity → pilot (if it's a pilot)
${Module.ToItem}         ; module → item
```

---

## Top-Level Objects (TLO)

### Always Available Globals

These objects are available **everywhere** without declaration:

| Object | Type | Description |
|--------|------|-------------|
| `EVE` | EVE | Main EVE interface |
| `Me` | Pilot | Current pilot/character |
| `MyShip` | Ship | Current ship |
| `ISXEVE` | ISXEVE | Extension itself |
| `Game` | Game | Game client state |
| `Session` | string | Session name (InnerSpace) |

**Example Usage:**

```lavish
; All available immediately
echo "Pilot: ${Me.Name}"
echo "Ship: ${MyShip.ToEntity.Name}"
echo "System: ${Me.SolarSystemID}"
echo "Station: ${Me.Station.Name}"
```

### Additional TLOs (Indexed Access)

These TLOs require an index or name parameter:

| Object | Type | Description |
|--------|------|-------------|
| `EVEWindow[...]` | EVEWindow | Access UI windows by caption/name/ID |
| `Entity[id]` | Entity | Access entity by ID |
| `Being[id]` | Being | Access being (player/NPC) by ID |
| `Chat[id]` or `Chat[name]` | ChatWindow | Access chat channel |
| `Universe[id]` or `Universe[name]` | Universe object | Access universe objects |
| `Local` | ChatWindow | Local chat (shortcut) |
| `CharSelect` | CharSelect | Character selection screen |
| `EVETime` | EVETime | EVE time object |
| `Overview` | Overview | Overview object |

**EVEWindow Examples:**
```lavish
; Access inventory window (CRITICAL for modern cargo API)
if ${EVEWindow[Inventory](exists)}
{
    echo "Inventory window is open"
    EVEWindow[Inventory].Child[ShipCargo]:GetItems[CargoItems]
}

; Access window by caption
if ${EVEWindow[ByCaption, "Item hangar"](exists)}
{
    echo "Item hangar is open"
}

; Access window by item ID
if ${EVEWindow[byItemID, ${stationID}](exists)}
{
    echo "Station services window is open"
}
```

**Entity/Being Direct Access:**
```lavish
; Access entity directly by ID (alternative to iteration)
variable int64 targetID = 123456789
if ${Entity[${targetID}](exists)}
{
    echo "Entity: ${Entity[${targetID}].Name}"
    echo "Distance: ${Entity[${targetID}].Distance}"
}

; Access being by ID
if ${Being[${charID}](exists)}
{
    echo "Being: ${Being[${charID}].Name}"
}
```

**Chat Access:**
```lavish
; Access local chat
if ${Local(exists)}
{
    echo "Local member count: ${Local.MemberCount}"
}

; Access chat by channel ID
if ${Chat[${channelID}](exists)}
{
    echo "Channel: ${Chat[${channelID}].Name}"
}

; Access chat by name
if ${Chat["My Chat Channel"](exists)}
{
    echo "Found channel by name"
}
```

**CharSelect (Login Screen):**
```lavish
; During character selection
if ${CharSelect(exists)}
{
    echo "At character selection screen"
    echo "Characters available: ${CharSelect.CharCount}"
}
```

**EVETime:**
```lavish
; EVE time
echo "EVE time: ${EVETime.Time}"
echo "EVE date: ${EVETime.Date}"
```

---

## EVE Object

### Overview

The `EVE` object provides access to global EVE game state and utilities.

### Properties

#### Progress Window

```lavish
; Check if progress window is open (loading screen)
if ${EVE.IsProgressWindowOpen}
{
    echo "Loading: ${EVE.ProgressWindowTitle}"
}
```

**Properties:**
- `EVE.IsProgressWindowOpen` → bool
- `EVE.ProgressWindowTitle` → string
- `EVE.ProgressWindowPercentage` → int (0-100)

#### Bookmarks

```lavish
; Access bookmark by name
if ${EVE.Bookmark["My Safe Spot"](exists)}
{
    echo "Bookmark ID: ${EVE.Bookmark["My Safe Spot"].ID}"
    echo "Solar System: ${EVE.Bookmark["My Safe Spot"].SolarSystemID}"
    echo "Location: ${EVE.Bookmark["My Safe Spot"].X}, ${EVE.Bookmark["My Safe Spot"].Y}, ${EVE.Bookmark["My Safe Spot"].Z}"
}

; Access bookmark by ID
variable int64 bookmarkID = 123456789
if ${EVE.Bookmark[${bookmarkID}](exists)}
{
    echo "Bookmark: ${EVE.Bookmark[${bookmarkID}].Label}"
}
```

**Bookmark Properties:**
- `EVE.Bookmark[name]` → Bookmark object
- `EVE.Bookmark[id]` → Bookmark object
- `EVE.BookmarkCount` → int

### Methods

#### Entity Queries

```lavish
; Query all entities
variable iterator Entity
EVE:QueryEntities[Entities]
Entities:GetIterator[Entity]

if ${Entity:First(exists)}
{
    do
    {
        echo "${Entity.Value.Name} (${Entity.Value.ID})"
    }
    while ${Entity:Next(exists)}
}

; Query with filter
variable index:entity Asteroids
EVE:QueryEntities[Asteroids, "CategoryID = CATEGORYID_ASTEROID"]

echo "Found ${Asteroids.Used} asteroids"
```

**Query Syntax:**
```
EVE:QueryEntities[index, "filter"]

Filters:
- CategoryID = value
- GroupID = value
- TypeID = value
- Distance < value
- Distance > value
- Name = "value"
- IsNPC = TRUE/FALSE
- IsPC = TRUE/FALSE
```

#### Location Names

```lavish
; Get name by ID
echo "Location: ${EVE.GetLocationNameByID[${stationID}]}"
echo "System: ${EVE.GetLocationNameByID[${solarSystemID}]}"
```

#### Item Database

```lavish
; Get item info by type ID
echo "Item name: ${EVE.GetItemName[${typeID}]}"
echo "Item group: ${EVE.GetGroupName[${groupID}]}"
echo "Item category: ${EVE.GetCategoryName[${categoryID}]}"
```

### Complete EVE Object Reference

| Property/Method | Returns | Description |
|-----------------|---------|-------------|
| `EVE.IsProgressWindowOpen` | bool | Loading screen visible |
| `EVE.ProgressWindowTitle` | string | Loading screen title |
| `EVE.Bookmark[name]` | Bookmark | Get bookmark by name |
| `EVE.Bookmark[id]` | Bookmark | Get bookmark by ID |
| `EVE.BookmarkCount` | int | Number of bookmarks |
| `EVE:QueryEntities[index, filter]` | - | Query entities with filter |
| `EVE.GetLocationNameByID[id]` | string | Get location name |
| `EVE.GetItemName[typeID]` | string | Get item name |
| `EVE.GetGroupName[groupID]` | string | Get group name |

---

## Me Object

### Overview

The `Me` object represents the current pilot (your character).

### Location Properties

```lavish
; Where am I?
echo "In Station: ${Me.InStation}"
echo "In Space: ${Me.InSpace}"
echo "Solar System: ${Me.SolarSystemID}"
echo "Station: ${Me.StationID}"

; Navigation state
echo "Warping: ${Me.ToEntity.Mode} == 3"  ; Mode 3 = Warp
echo "Autopilot: ${Me.AutoPilotOn}"
echo "Velocity: ${Me.ToEntity.Velocity}"
```

**Properties:**
- `Me.InStation` → bool
- `Me.InSpace` → bool
- `Me.SolarSystemID` → int
- `Me.StationID` → int64
- `Me.Station` → Station object
- `Me.AutoPilotOn` → bool

### Character Properties

```lavish
; Character info
echo "Name: ${Me.Name}"
echo "Char ID: ${Me.CharID}"
echo "Corp ID: ${Me.CorpID}"
echo "Alliance ID: ${Me.AllianceID}"

; Skills
echo "Skill points: ${Me.SkillPoints}"
echo "Has skill: ${Me.HasSkill[${skillID}]}"
echo "Skill level: ${Me.Skill[${skillID}].Level}"
```

**Properties:**
- `Me.Name` → string
- `Me.CharID` → int
- `Me.CorpID` → int
- `Me.AllianceID` → int
- `Me.SkillPoints` → int
- `Me.Skill[id]` → Skill object

### Wallet & Assets

```lavish
; Wallet
echo "Balance: ${Me.Wallet} ISK"

; Cargo
echo "Cargo capacity: ${Me.Ship.CargoCapacity}"
echo "Cargo used: ${Me.Ship.UsedCargoCapacity}"
echo "Cargo free: ${Math.Calc[${Me.Ship.CargoCapacity} - ${Me.Ship.UsedCargoCapacity}]}"
```

**Properties:**
- `Me.Wallet` → double (ISK balance)
- `Me.Ship` → Ship object (same as MyShip)

### ToEntity Conversion

```lavish
; Me as entity (for position, mode, etc.)
echo "Position: ${Me.ToEntity.X}, ${Me.ToEntity.Y}, ${Me.ToEntity.Z}"
echo "Mode: ${Me.ToEntity.Mode}"  ; 0=unknown, 1=stop, 2=orbit, 3=warp, etc.
echo "Velocity: ${Me.ToEntity.Velocity} m/s"
echo "Max Velocity: ${Me.ToEntity.MaxVelocity} m/s"
```

**Modes:**
- 0 = Unknown
- 1 = Stop
- 2 = Orbit
- 3 = Warp
- 4 = Approach

### Fleet

```lavish
; Fleet membership
if ${Me.Fleet(exists)}
{
    echo "Fleet members: ${Me.Fleet.MemberCount}"

    ; Iterate fleet members
    variable iterator FleetMember
    Me.Fleet:GetMembers[FleetMembers]
    FleetMembers:GetIterator[FleetMember]

    if ${FleetMember:First(exists)}
    {
        do
        {
            echo "Member: ${FleetMember.Value.ToPilot.Name}"
            echo "  CharID: ${FleetMember.Value.CharID}"
            echo "  ShipType: ${FleetMember.Value.ToFleetMember.ShipType}"
            echo "  SolarSystem: ${FleetMember.Value.ToFleetMember.SolarSystemID}"
        }
        while ${FleetMember:Next(exists)}
    }
}
```

**Fleet Properties:**
- `Me.Fleet` → Fleet object
- `Me.Fleet.MemberCount` → int
- `Me.Fleet.Member[id]` → FleetMember object

### Targeting

```lavish
; Targeting info
echo "Locked targets: ${Me.TargetCount}"
echo "Max targets: ${Me.MaxLockedTargets}"
echo "Targeting range: ${Me.TargetingRange}"

; Get locked target IDs
variable iterator Target
Me:GetTargets[Targets]
Targets:GetIterator[Target]

if ${Target:First(exists)}
{
    do
    {
        echo "Target: ${Entity[${Target.Value}].Name} (${Target.Value})"
    }
    while ${Target:Next(exists)}
}
```

**Targeting Properties:**
- `Me.TargetCount` → int
- `Me.MaxLockedTargets` → int
- `Me.TargetingRange` → double
- `Me.GetTargets[index]` → Fills index with target IDs

### Complete Me Object Reference

| Property/Method | Returns | Description |
|-----------------|---------|-------------|
| `Me.Name` | string | Character name |
| `Me.CharID` | int | Character ID |
| `Me.CorpID` | int | Corporation ID |
| `Me.AllianceID` | int | Alliance ID |
| `Me.InStation` | bool | In station |
| `Me.InSpace` | bool | In space |
| `Me.SolarSystemID` | int | Current system |
| `Me.StationID` | int64 | Current station |
| `Me.Station` | Station | Station object |
| `Me.Wallet` | double | ISK balance |
| `Me.ToEntity` | Entity | Self as entity |
| `Me.AutoPilotOn` | bool | Autopilot state |
| `Me.Fleet` | Fleet | Fleet object |
| `Me.TargetCount` | int | Locked targets |
| `Me.MaxLockedTargets` | int | Max targets |
| `Me.Ship` | Ship | Current ship (=MyShip) |
| `Me:GetTargets[index]` | - | Fill index with targets |

---

## MyShip Object

### Overview

The `MyShip` object represents your current ship.

**Note:** `MyShip` and `Me.Ship` are identical objects.

### Ship Properties

```lavish
; Basic info
echo "Ship name: ${MyShip.ToEntity.Name}"
echo "Ship type: ${MyShip.ToEntity.TypeID}"
echo "Ship group: ${MyShip.ToEntity.GroupID}"

; Capacitor
echo "Capacitor: ${MyShip.Capacitor} / ${MyShip.MaxCapacitor}"
echo "Cap %: ${Math.Calc[${MyShip.Capacitor} / ${MyShip.MaxCapacitor} * 100]}"

; HP
echo "Shield: ${MyShip.Shield} / ${MyShip.MaxShield}"
echo "Armor: ${MyShip.Armor} / ${MyShip.MaxArmor}"
echo "Structure: ${MyShip.Structure} / ${MyShip.MaxStructure}"

; Movement
echo "Max velocity: ${MyShip.MaxVelocity}"
echo "Max warp speed: ${MyShip.MaxWarpSpeed}"
echo "Warp capacitor need: ${MyShip.WarpCapacitorNeed}"
```

**Properties:**
- `MyShip.Capacitor` → double
- `MyShip.MaxCapacitor` → double
- `MyShip.Shield` → double
- `MyShip.MaxShield` → double
- `MyShip.Armor` → double
- `MyShip.MaxArmor` → double
- `MyShip.Structure` → double
- `MyShip.MaxStructure` → double
- `MyShip.MaxVelocity` → double
- `MyShip.MaxWarpSpeed` → double

### Cargo (Modern API - July 2020+)

**⚠️ WARNING:** Old `MyShip.CargoCapacity` / `MyShip:GetCargo[]` methods were DEPRECATED in July 2020. Use EVEWindow[Inventory] instead.

```lavish
; ✅ MODERN API - Access cargo via inventory window
if ${EVEWindow[Inventory](exists)}
{
    ; Ship cargo
    variable int64 cargoCapacity = ${EVEWindow[Inventory].Child[ShipCargo].Capacity}
    variable int64 cargoUsed = ${EVEWindow[Inventory].Child[ShipCargo].UsedCapacity}
    echo "Cargo: ${cargoUsed} / ${cargoCapacity}"

    ; Ore hold
    if ${EVEWindow[Inventory].Child[ShipOreHold](exists)}
    {
        variable int64 oreCapacity = ${EVEWindow[Inventory].Child[ShipOreHold].Capacity}
        variable int64 oreUsed = ${EVEWindow[Inventory].Child[ShipOreHold].UsedCapacity}
        echo "Ore hold: ${oreUsed} / ${oreCapacity}"
    }

    ; Drone bay
    if ${EVEWindow[Inventory].Child[ShipDroneBay](exists)}
    {
        variable int64 droneCapacity = ${EVEWindow[Inventory].Child[ShipDroneBay].Capacity}
        variable int64 droneUsed = ${EVEWindow[Inventory].Child[ShipDroneBay].UsedCapacity}
        echo "Drone bay: ${droneUsed} / ${droneCapacity}"
    }
}
else
{
    echo "ERROR: Inventory window not open"
}
```

**Modern Inventory Access:**
- `EVEWindow[Inventory].Child[ShipCargo].Capacity` → int64
- `EVEWindow[Inventory].Child[ShipCargo].UsedCapacity` → int64
- `EVEWindow[Inventory].Child[ShipOreHold].Capacity` → int64
- `EVEWindow[Inventory].Child[ShipOreHold].UsedCapacity` → int64
- `EVEWindow[Inventory].Child[ShipDroneBay].Capacity` → int64
- `EVEWindow[Inventory].Child[ShipDroneBay].UsedCapacity` → int64

**See Also:** This guide covers both legacy and modern cargo APIs

### Modules

```lavish
; Get all modules
variable iterator Module
MyShip:GetModules[Modules]
Modules:GetIterator[Module]

if ${Module:First(exists)}
{
    do
    {
        echo "${Module.Value.ToItem.Name} - Active: ${Module.Value.IsActive}"
    }
    while ${Module:Next(exists)}
}

; Get modules by group
variable index:module TurretModules
MyShip:GetTurretModules[TurretModules]
echo "Turrets: ${TurretModules.Used}"

variable index:module MiningLasers
MyShip:GetMiningLasers[MiningLasers]
echo "Mining lasers: ${MiningLasers.Used}"
```

**Module Methods:**
- `MyShip:GetModules[index]` → All modules
- `MyShip:GetTurretModules[index]` → Turrets
- `MyShip:GetLauncherModules[index]` → Launchers
- `MyShip:GetMiningLasers[index]` → Mining lasers
- `MyShip:GetShieldBoosters[index]` → Shield boosters
- `MyShip:GetArmorRepairers[index]` → Armor repairers

### Drones

```lavish
; Drones in bay
variable index:item DronesInBay
MyShip:GetDronesInBay[DronesInBay]
echo "Drones in bay: ${DronesInBay.Used}"

; Drones in space
variable index:activedrone DronesInSpace
MyShip:GetActiveDrones[DronesInSpace]
echo "Drones in space: ${DronesInSpace.Used}"

; Max drones
echo "Max drones: ${MyShip.MaxActiveDrones}"
```

**Drone Properties:**
- `MyShip.MaxActiveDrones` → int
- `MyShip:GetDronesInBay[index]` → Item index
- `MyShip:GetActiveDrones[index]` → ActiveDrone index

### ToEntity Conversion

```lavish
; Ship as entity
echo "Ship ID: ${MyShip.ToEntity.ID}"
echo "Ship position: ${MyShip.ToEntity.X}, ${MyShip.ToEntity.Y}, ${MyShip.ToEntity.Z}"
echo "Ship velocity: ${MyShip.ToEntity.Velocity}"
echo "Ship mode: ${MyShip.ToEntity.Mode}"
```

### Complete MyShip Object Reference

| Property/Method | Returns | Description |
|-----------------|---------|-------------|
| `MyShip.Capacitor` | double | Current capacitor |
| `MyShip.MaxCapacitor` | double | Max capacitor |
| `MyShip.Shield` | double | Current shield HP |
| `MyShip.MaxShield` | double | Max shield HP |
| `MyShip.Armor` | double | Current armor HP |
| `MyShip.MaxArmor` | double | Max armor HP |
| `MyShip.Structure` | double | Current hull HP |
| `MyShip.MaxStructure` | double | Max hull HP |
| ~~`MyShip.CargoCapacity`~~ | ~~double~~ | ⚠️ DEPRECATED - Use EVEWindow[Inventory] |
| ~~`MyShip.UsedCargoCapacity`~~ | ~~double~~ | ⚠️ DEPRECATED - Use EVEWindow[Inventory] |
| ~~`MyShip.OreHoldCapacity`~~ | ~~double~~ | ⚠️ DEPRECATED - Use EVEWindow[Inventory] |
| `MyShip.MaxVelocity` | double | Max velocity |
| `MyShip.MaxWarpSpeed` | double | Max warp speed |
| `MyShip.MaxActiveDrones` | int | Max drones |
| `MyShip.ToEntity` | Entity | Ship as entity |
| `MyShip:GetModules[index]` | - | All modules |
| `MyShip:GetTurretModules[index]` | - | Turret modules |
| `MyShip:GetMiningLasers[index]` | - | Mining lasers |
| `MyShip:GetActiveDrones[index]` | - | Drones in space |

---

## Entity Object

### Overview

The `Entity` object represents any object in space (ships, asteroids, structures, etc.).

### Entity Access

```lavish
; By ID
variable int64 entityID = 123456789
if ${Entity[${entityID}](exists)}
{
    echo "Entity: ${Entity[${entityID}].Name}"
}

; By iteration
variable iterator Entity
EVE:QueryEntities[Entities]
Entities:GetIterator[Entity]

if ${Entity:First(exists)}
{
    do
    {
        ; Entity.Value is the entity object
        echo "${Entity.Value.Name}"
    }
    while ${Entity:Next(exists)}
}
```

### Identity Properties

```lavish
; Basic identity
echo "ID: ${Entity[${entityID}].ID}"
echo "Name: ${Entity[${entityID}].Name}"
echo "TypeID: ${Entity[${entityID}].TypeID}"
echo "GroupID: ${Entity[${entityID}].GroupID}"
echo "CategoryID: ${Entity[${entityID}].CategoryID}"
```

**Properties:**
- `Entity.ID` → int64
- `Entity.Name` → string
- `Entity.TypeID` → int
- `Entity.GroupID` → int
- `Entity.CategoryID` → int

### Type Checks

```lavish
; Entity type
echo "IsNPC: ${Entity[${entityID}].IsNPC}"
echo "IsPC: ${Entity[${entityID}].IsPC}"
echo "IsBillboard: ${Entity[${entityID}].IsBillboard}"

; Specific types
if ${Entity[${entityID}].CategoryID} == CATEGORYID_SHIP
{
    echo "This is a ship"
}

if ${Entity[${entityID}].CategoryID} == CATEGORYID_ASTEROID
{
    echo "This is an asteroid"
}

if ${Entity[${entityID}].GroupID} == GROUPID_WRECK
{
    echo "This is a wreck"
}
```

**Type Properties:**
- `Entity.IsNPC` → bool
- `Entity.IsPC` → bool
- `Entity.IsBillboard` → bool (bookmark)

### Position & Distance

```lavish
; Position
echo "X: ${Entity[${entityID}].X}"
echo "Y: ${Entity[${entityID}].Y}"
echo "Z: ${Entity[${entityID}].Z}"

; Distance
echo "Distance: ${Entity[${entityID}].Distance}"
echo "Distance to me: ${Math.Distance[${Me.ToEntity.X}, ${Me.ToEntity.Y}, ${Me.ToEntity.Z}, ${Entity[${entityID}].X}, ${Entity[${entityID}].Y}, ${Entity[${entityID}].Z}]}"

; In range check
if ${Entity[${entityID}].Distance} < 50000
{
    echo "Within 50km"
}
```

**Position Properties:**
- `Entity.X` → double
- `Entity.Y` → double
- `Entity.Z` → double
- `Entity.Distance` → double (distance to player)

### Movement & Mode

```lavish
; Movement
echo "Velocity: ${Entity[${entityID}].Velocity}"
echo "Mode: ${Entity[${entityID}].Mode}"

; Approaching
if ${Entity[${entityID}].Approaching(exists)}
{
    echo "Approaching: ${Entity[${entityID}].Approaching.Name}"
}
```

**Movement Properties:**
- `Entity.Velocity` → double
- `Entity.Mode` → int (0=unknown, 1=stop, 2=orbit, 3=warp, etc.)
- `Entity.Approaching` → Entity (what it's approaching)

### Targeting

```lavish
; Lock status
echo "Is locked: ${Entity[${entityID}].IsLockedTarget}"
echo "Being targeted: ${Entity[${entityID}].BeingTargeted}"
echo "Being jammed: ${Entity[${entityID}].IsJamming}"
echo "Being warp disrupted: ${Entity[${entityID}].IsWarpScrambling}"
```

**Targeting Properties:**
- `Entity.IsLockedTarget` → bool
- `Entity.BeingTargeted` → bool
- `Entity.IsJamming` → bool (targeting us)
- `Entity.IsWarpScrambling` → bool (scramming us)

### Actions

```lavish
; Lock target
if !${Entity[${entityID}].IsLockedTarget}
{
    Entity[${entityID}]:LockTarget
    echo "Locking ${Entity[${entityID}].Name}..."
}

; Unlock target
if ${Entity[${entityID}].IsLockedTarget}
{
    Entity[${entityID}]:UnlockTarget
    echo "Unlocking ${Entity[${entityID}].Name}..."
}

; Approach
Entity[${entityID}]:Approach
echo "Approaching ${Entity[${entityID}].Name}..."

; Orbit
Entity[${entityID}]:Orbit[15000]
echo "Orbiting ${Entity[${entityID}].Name} at 15km..."

; Open (wrecks, containers)
Entity[${entityID}]:Open
echo "Opening ${Entity[${entityID}].Name}..."

; Warp to
Entity[${entityID}]:WarpTo[0]  ; 0 = at 0km, 10000 = at 10km, etc.
echo "Warping to ${Entity[${entityID}].Name}..."
```

**Action Methods:**
- `Entity:LockTarget` → Lock entity
- `Entity:UnlockTarget` → Unlock entity
- `Entity:Approach` → Approach entity
- `Entity:Orbit[distance]` → Orbit at distance
- `Entity:Open` → Open container/wreck
- `Entity:WarpTo[distance]` → Warp to entity at distance

### Container/Wreck Properties

```lavish
; For wrecks/containers
if ${Entity[${entityID}].GroupID} == GROUPID_WRECK
{
    echo "Is empty: ${Entity[${entityID}].IsEmpty}"
    echo "Is being salvaged: ${Entity[${entityID}].IsBeingSalvaged}"
    echo "Is being tractored: ${Entity[${entityID}].IsBeingTractored}"
}
```

**Container Properties:**
- `Entity.IsEmpty` → bool
- `Entity.IsBeingSalvaged` → bool
- `Entity.IsBeingTractored` → bool

### Complete Entity Object Reference

| Property/Method | Returns | Description |
|-----------------|---------|-------------|
| `Entity.ID` | int64 | Entity ID |
| `Entity.Name` | string | Entity name |
| `Entity.TypeID` | int | Type ID |
| `Entity.GroupID` | int | Group ID |
| `Entity.CategoryID` | int | Category ID |
| `Entity.Distance` | double | Distance to player (meters) |
| `Entity.Distance2` | double | Squared distance (FASTER for comparisons) |
| `Entity.X/Y/Z` | double | Position |
| `Entity.Velocity` | double | Velocity m/s |
| `Entity.Mode` | int | Movement mode |
| `Entity.IsNPC` | bool | Is NPC |
| `Entity.IsPC` | bool | Is player |
| `Entity.IsLockedTarget` | bool | Is locked |
| `Entity.BeingTargeted` | bool | Being targeted |
| `Entity.IsEmpty` | bool | Container empty |
| `Entity:LockTarget` | - | Lock target |
| `Entity:UnlockTarget` | - | Unlock target |
| `Entity:Approach` | - | Approach entity |
| `Entity:Orbit[distance]` | - | Orbit at distance |
| `Entity:WarpTo[distance]` | - | Warp to entity |
| `Entity:Open` | - | Open container |

---

## Module Object

### Overview

The `Module` object represents a module fitted to your ship.

### Module Access

```lavish
; Get all modules
variable iterator Module
MyShip:GetModules[Modules]
Modules:GetIterator[Module]

if ${Module:First(exists)}
{
    do
    {
        echo "Module: ${Module.Value.ToItem.Name}"
        echo "  Slot: ${Module.Value.Slot}"
        echo "  Active: ${Module.Value.IsActive}"
    }
    while ${Module:Next(exists)}
}

; Access module by slot
variable module MyModule
MyModule:Set[${MyShip.Module[${slotNumber}]}]
```

### Module Properties

```lavish
; Basic info
echo "Name: ${Module.ToItem.Name}"
echo "Slot: ${Module.Slot}"
echo "Type: ${Module.ToItem.TypeID}"
echo "Group: ${Module.ToItem.GroupID}"

; State
echo "Is online: ${Module.IsOnline}"
echo "Is active: ${Module.IsActive}"
echo "Is activating: ${Module.IsActivating}"
echo "Is deactivating: ${Module.IsDeactivating}"
echo "Is changing ammo: ${Module.IsChangingAmmo}"
echo "Is reloading: ${Module.IsReloading}"

; Stats
echo "Optimal range: ${Module.OptimalRange}"
echo "Falloff: ${Module.FalloffRange}"
echo "Max range: ${Math.Calc[${Module.OptimalRange} + ${Module.FalloffRange}]}"
echo "Cycle time: ${Module.Duration}"
echo "Activation cost: ${Module.ActivationCost}"
```

**Properties:**
- `Module.Slot` → int
- `Module.ToItem` → Item object
- `Module.IsOnline` → bool
- `Module.IsActive` → bool
- `Module.IsActivating` → bool
- `Module.IsDeactivating` → bool
- `Module.IsChangingAmmo` → bool
- `Module.IsReloading` → bool
- `Module.OptimalRange` → double
- `Module.FalloffRange` → double
- `Module.Duration` → int (cycle time ms)
- `Module.ActivationCost` → double (cap cost)

### Charges (Ammo)

```lavish
; Charge info
if ${Module.Charge(exists)}
{
    echo "Loaded: ${Module.Charge.TypeID}"
    echo "Quantity: ${Module.CurrentCharges}"
    echo "Max: ${Module.MaxCharges}"
}
else
{
    echo "No charges loaded"
}
```

**Charge Properties:**
- `Module.Charge` → Item (loaded charge)
- `Module.CurrentCharges` → int
- `Module.MaxCharges` → int

### Actions

```lavish
; Activate module
if !${Module.IsActive} && ${Module.IsOnline}
{
    Module:Activate
    echo "Activating ${Module.ToItem.Name}..."
}

; Activate on target
Module:Activate[${targetID}]
echo "Activating ${Module.ToItem.Name} on ${Entity[${targetID}].Name}..."

; Deactivate
if ${Module.IsActive}
{
    Module:Deactivate
    echo "Deactivating ${Module.ToItem.Name}..."
}

; Online/Offline
Module:SetOnline
Module:SetOffline

; Change ammo
Module:ChangeAmmo[${chargeTypeID}]
echo "Changing ammo to type ${chargeTypeID}..."
```

**Action Methods:**
- `Module:Activate` → Activate module
- `Module:Activate[targetID]` → Activate on target
- `Module:Deactivate` → Deactivate module
- `Module:SetOnline` → Set online
- `Module:SetOffline` → Set offline
- `Module:ChangeAmmo[typeID]` → Change ammo type

### Special Module Types

**Mining Lasers:**
```lavish
; Check if mining laser
if ${Module.ToItem.GroupID} == GROUPID_MININGLASER
{
    echo "This is a mining laser"
    echo "Mining amount: ${Module.MiningAmount}"
}
```

**Turrets:**
```lavish
; Check if turret
if ${Module.ToItem.CategoryID} == CATEGORYID_MODULE && ${Module.ToItem.GroupID} == GROUPID_PROJECTILETURRET
{
    echo "This is a projectile turret"
    echo "Tracking speed: ${Module.TrackingSpeed}"
}
```

**Missile Launchers:**
```lavish
; Check if launcher
if ${Module.ToItem.GroupID} == GROUPID_MISSILELAUNCHER
{
    echo "This is a missile launcher"
}
```

### Complete Module Object Reference

| Property/Method | Returns | Description |
|-----------------|---------|-------------|
| `Module.Slot` | int | Slot number |
| `Module.ToItem` | Item | Module as item |
| `Module.IsOnline` | bool | Is online |
| `Module.IsActive` | bool | Is active |
| `Module.IsActivating` | bool | Is activating |
| `Module.IsDeactivating` | bool | Is deactivating |
| `Module.IsChangingAmmo` | bool | Is changing ammo |
| `Module.IsReloading` | bool | Is reloading |
| `Module.OptimalRange` | double | Optimal range |
| `Module.FalloffRange` | double | Falloff range |
| `Module.Duration` | int | Cycle time (ms) |
| `Module.ActivationCost` | double | Cap cost |
| `Module.Charge` | Item | Loaded charge |
| `Module.CurrentCharges` | int | Charges loaded |
| `Module.MaxCharges` | int | Max charges |
| `Module:Activate` | - | Activate module |
| `Module:Activate[targetID]` | - | Activate on target |
| `Module:Deactivate` | - | Deactivate module |
| `Module:ChangeAmmo[typeID]` | - | Change ammo |

---

## Item Object

### Overview

The `Item` object represents any item (in cargo, hangar, or fitted).

### Item Access

**⚠️ NOTE:** Use modern EVEWindow[Inventory] API to get items, not deprecated `MyShip:GetCargo[]`

```lavish
; ✅ MODERN API - Get items from cargo via inventory window
if ${EVEWindow[Inventory](exists)}
{
    variable index:item CargoItems
    EVEWindow[Inventory].Child[ShipCargo]:GetItems[CargoItems]

    variable iterator Item
    CargoItems:GetIterator[Item]

    if ${Item:First(exists)}
    {
        do
        {
            echo "${Item.Value.Name} x${Item.Value.Quantity}"
        }
        while ${Item:Next(exists)}
    }
}
```

### Item Properties

```lavish
; Identity
echo "ID: ${Item.ID}"
echo "Name: ${Item.Name}"
echo "TypeID: ${Item.TypeID}"
echo "GroupID: ${Item.GroupID}"
echo "CategoryID: ${Item.CategoryID}"

; Quantity
echo "Quantity: ${Item.Quantity}"
echo "Volume: ${Item.Volume}"
echo "Total volume: ${Math.Calc[${Item.Quantity} * ${Item.Volume}]}"

; Location
echo "LocationID: ${Item.LocationID}"
echo "Flag: ${Item.Flag}"  ; Cargo=5, DroneBay=87, OreHold=134, etc.
```

**Properties:**
- `Item.ID` → int64
- `Item.Name` → string
- `Item.TypeID` → int
- `Item.GroupID` → int
- `Item.CategoryID` → int
- `Item.Quantity` → int
- `Item.Volume` → double
- `Item.LocationID` → int64
- `Item.Flag` → int (inventory flag)

### Actions

```lavish
; Move item
Item:MoveTo[${locationID}, ${locationFlag}]

; Move to cargo
Item:MoveTo[${MyShip.ID}, 5]  ; 5 = Cargo flag

; Move to ore hold
Item:MoveTo[${MyShip.ID}, 134]  ; 134 = Ore hold flag

; Jettison
Item:Jettison
echo "Jettisoning ${Item.Name}..."

; Stack all
Item:StackAll
```

**Action Methods:**
- `Item:MoveTo[locationID, flag]` → Move item
- `Item:Jettison` → Jettison to space
- `Item:StackAll` → Stack all of this type

### Inventory Flags

| Flag | Location | Description |
|------|----------|-------------|
| 4 | Hangar | Station hangar |
| 5 | Cargo | Ship cargo |
| 7 | Module | Fitted module |
| 87 | DroneBay | Drone bay |
| 90 | ShipHangar | Ship hangar |
| 134 | OreHold | Ore hold |
| 142 | FleetHangar | Fleet hangar |

### Complete Item Object Reference

| Property/Method | Returns | Description |
|-----------------|---------|-------------|
| `Item.ID` | int64 | Item ID |
| `Item.Name` | string | Item name |
| `Item.TypeID` | int | Type ID |
| `Item.GroupID` | int | Group ID |
| `Item.CategoryID` | int | Category ID |
| `Item.Quantity` | int | Quantity |
| `Item.Volume` | double | Volume per unit |
| `Item.LocationID` | int64 | Location ID |
| `Item.Flag` | int | Inventory flag |
| `Item:MoveTo[locationID, flag]` | - | Move item |
| `Item:Jettison` | - | Jettison item |
| `Item:StackAll` | - | Stack all of type |

---

## Bookmark Object

### Overview

The `Bookmark` object represents a bookmark (location marker).

### Bookmark Access

```lavish
; By name
if ${EVE.Bookmark["My Safe Spot"](exists)}
{
    variable bookmark MyBookmark
    MyBookmark:Set[${EVE.Bookmark["My Safe Spot"]}]
}

; By ID
if ${EVE.Bookmark[${bookmarkID}](exists)}
{
    variable bookmark MyBookmark
    MyBookmark:Set[${EVE.Bookmark[${bookmarkID}]}]
}

; Iterate all bookmarks
variable iterator Bookmark
EVE:GetBookmarks[Bookmarks]
Bookmarks:GetIterator[Bookmark]

if ${Bookmark:First(exists)}
{
    do
    {
        echo "${Bookmark.Value.Label} (${Bookmark.Value.ID})"
    }
    while ${Bookmark:Next(exists)}
}
```

### Bookmark Properties

```lavish
; Identity
echo "ID: ${Bookmark.ID}"
echo "Label: ${Bookmark.Label}"
echo "Note: ${Bookmark.Note}"
echo "Creator: ${Bookmark.CreatorID}"

; Location
echo "Solar System: ${Bookmark.SolarSystemID}"
echo "ItemID: ${Bookmark.ItemID}"  ; Station/structure ID if docked
echo "TypeID: ${Bookmark.TypeID}"  ; 5=station, 0=space, etc.
echo "Position: ${Bookmark.X}, ${Bookmark.Y}, ${Bookmark.Z}"

; Checks
if ${Bookmark.ItemID} > 0
{
    echo "This bookmark is a station/structure"
}
else
{
    echo "This bookmark is in space"
}
```

**Properties:**
- `Bookmark.ID` → int64
- `Bookmark.Label` → string
- `Bookmark.Note` → string
- `Bookmark.CreatorID` → int
- `Bookmark.SolarSystemID` → int
- `Bookmark.ItemID` → int64 (0 if in space)
- `Bookmark.TypeID` → int
- `Bookmark.X/Y/Z` → double

### Actions

```lavish
; Warp to bookmark
Bookmark:WarpTo[0]  ; 0 = at 0km
Bookmark:WarpTo[100000]  ; at 100km

; Warp to and dock
if ${Bookmark.ItemID} > 0
{
    Bookmark:WarpTo[0]
    wait 50 ${Me.ToEntity.Mode} == 3  ; Wait for warp
    wait 300 ${Me.ToEntity.Mode} != 3  ; Wait to exit warp
    wait 10
    call Station.DockAtStation ${Bookmark.ItemID}
}

; Delete bookmark
Bookmark:Remove

; Get ToEntity (if in space)
if ${Bookmark.ToEntity(exists)}
{
    echo "Bookmark entity: ${Bookmark.ToEntity.Name}"
    echo "Distance: ${Bookmark.ToEntity.Distance}"
}
```

**Action Methods:**
- `Bookmark:WarpTo[distance]` → Warp to bookmark
- `Bookmark:Remove` → Delete bookmark
- `Bookmark.ToEntity` → Entity (if in space)

### Complete Bookmark Object Reference

| Property/Method | Returns | Description |
|-----------------|---------|-------------|
| `Bookmark.ID` | int64 | Bookmark ID |
| `Bookmark.Label` | string | Bookmark name |
| `Bookmark.Note` | string | Bookmark note |
| `Bookmark.SolarSystemID` | int | Solar system |
| `Bookmark.ItemID` | int64 | Station/structure ID |
| `Bookmark.TypeID` | int | Type ID |
| `Bookmark.X/Y/Z` | double | Position |
| `Bookmark.ToEntity` | Entity | Bookmark as entity |
| `Bookmark:WarpTo[distance]` | - | Warp to bookmark |
| `Bookmark:Remove` | - | Delete bookmark |

---

## Station Object

### Overview

The `Station` object represents a station or structure.

### Station Access

```lavish
; Current station
if ${Me.InStation}
{
    echo "Docked at: ${Me.Station.Name}"
    echo "Station ID: ${Me.Station.ID}"
}

; By entity
if ${Entity[${stationID}].GroupID} == GROUPID_STATION
{
    echo "Station: ${Entity[${stationID}].Name}"
}
```

### Station Properties

```lavish
; Identity
echo "ID: ${Station.ID}"
echo "Name: ${Station.Name}"
echo "TypeID: ${Station.TypeID}"
echo "GroupID: ${Station.GroupID}"

; Owner
echo "OwnerID: ${Station.OwnerID}"
echo "Corp: ${Station.Corporation.Name}"
```

**Properties:**
- `Station.ID` → int64
- `Station.Name` → string
- `Station.TypeID` → int
- `Station.GroupID` → int
- `Station.OwnerID` → int
- `Station.Corporation` → Corporation object

### Actions

```lavish
; Dock
Station:Dock
echo "Docking at ${Station.Name}..."

; Undock
if ${Me.InStation}
{
    Me.Station:Undock
    echo "Undocking..."
    wait 50 ${Me.InSpace}
}
```

**Action Methods:**
- `Station:Dock` → Dock at station
- `Station:Undock` → Undock from station

### Complete Station Object Reference

| Property/Method | Returns | Description |
|-----------------|---------|-------------|
| `Station.ID` | int64 | Station ID |
| `Station.Name` | string | Station name |
| `Station.TypeID` | int | Type ID |
| `Station.GroupID` | int | Group ID |
| `Station.OwnerID` | int | Owner ID |
| `Station:Dock` | - | Dock at station |
| `Station:Undock` | - | Undock |

---

## Query Methods

### EntityQuery Syntax

```lavish
; Basic query
EVE:QueryEntities[index, "filter"]

; Examples:
EVE:QueryEntities[Asteroids, "CategoryID = ${CATEGORYID_ASTEROID}"]
EVE:QueryEntities[NPCs, "CategoryID = ${CATEGORYID_ENTITY} && IsNPC = TRUE"]
EVE:QueryEntities[Wrecks, "GroupID = ${GROUPID_WRECK}"]
EVE:QueryEntities[CloseEntities, "Distance < 50000"]
```

### Filter Operators

| Operator | Example | Description |
|----------|---------|-------------|
| `=` | `CategoryID = 25` | Equal |
| `!=` | `CategoryID != 25` | Not equal |
| `<` | `Distance < 50000` | Less than |
| `>` | `Distance > 50000` | Greater than |
| `<=` | `Distance <= 50000` | Less than or equal |
| `>=` | `Distance >= 50000` | Greater than or equal |
| `&&` | `IsNPC = TRUE && Distance < 100000` | AND |
| `||` | `CategoryID = 2 || CategoryID = 25` | OR |

### Common Query Patterns

**Get NPCs within range:**
```lavish
variable index:entity NPCs
EVE:QueryEntities[NPCs, "CategoryID = ${CATEGORYID_ENTITY} && IsNPC = TRUE && Distance < 100000"]
echo "Found ${NPCs.Used} NPCs"
```

**Get asteroids:**
```lavish
variable index:entity Asteroids
EVE:QueryEntities[Asteroids, "CategoryID = ${CATEGORYID_ASTEROID} && Distance < 50000"]
echo "Found ${Asteroids.Used} asteroids"
```

**Get wrecks:**
```lavish
variable index:entity Wrecks
EVE:QueryEntities[Wrecks, "GroupID = ${GROUPID_WRECK} && Distance < 100000"]
echo "Found ${Wrecks.Used} wrecks"
```

**Get players:**
```lavish
variable index:entity Players
EVE:QueryEntities[Players, "CategoryID = ${CATEGORYID_SHIP} && IsPC = TRUE"]
echo "Found ${Players.Used} players"
```

---

## Common Patterns

### Safe Entity Check

```lavish
; Always check exists before accessing
variable int64 targetID = 123456789

if ${Entity[${targetID}](exists)}
{
    echo "Entity exists: ${Entity[${targetID}].Name}"
    Entity[${targetID}]:LockTarget
}
else
{
    echo "Entity doesn't exist (probably died)"
}
```

### Distance Check

```lavish
; Check distance before action
if ${Entity[${targetID}](exists)}
{
    if ${Entity[${targetID}].Distance} < 50000
    {
        Entity[${targetID}]:LockTarget
    }
    else
    {
        echo "Too far: ${Entity[${targetID}].Distance}m"
    }
}
```

### Module Activation Pattern

```lavish
; Safe module activation
if ${Module.IsOnline} && !${Module.IsActive} && !${Module.IsActivating}
{
    if ${MyShip.Capacitor} > ${Module.ActivationCost}
    {
        if ${Entity[${targetID}](exists)}
        {
            if ${Entity[${targetID}].Distance} < ${Module.OptimalRange}
            {
                Module:Activate[${targetID}]
            }
        }
    }
}
```

### Wait for Warp

```lavish
; Warp and wait
Entity[${targetID}]:WarpTo[0]
echo "Warping..."

wait 50 ${Me.ToEntity.Mode} == 3  ; Wait to enter warp
wait 300 ${Me.ToEntity.Mode} != 3  ; Wait to exit warp (max 30 sec)

if ${Me.ToEntity.Mode} == 3
{
    echo "ERROR: Still in warp after 30 seconds"
}
else
{
    echo "Arrived at destination"
}
```

### Inventory Iteration (Modern API)

**⚠️ WARNING:** Old `MyShip:GetCargo[]` is DEPRECATED. Use EVEWindow[Inventory] instead.

```lavish
; ✅ MODERN API - Get all cargo items via inventory window
if ${EVEWindow[Inventory](exists)}
{
    variable index:item CargoItems
    EVEWindow[Inventory].Child[ShipCargo]:GetItems[CargoItems]

    variable iterator Item
    CargoItems:GetIterator[Item]

    if ${Item:First(exists)}
    {
        do
        {
            echo "${Item.Value.Name}"
            echo "  Quantity: ${Item.Value.Quantity}"
            echo "  Volume: ${Math.Calc[${Item.Value.Quantity} * ${Item.Value.Volume}]}"
        }
        while ${Item:Next(exists)}
    }
    else
    {
        echo "Cargo is empty"
    }
}
else
{
    echo "ERROR: Inventory window not open"
    ; Open inventory if needed: EVE:Execute[OpenInventory]
}
```

---

## Quick Reference Tables

### Category IDs

| ID | Name | Description |
|----|------|-------------|
| 2 | Celestial | Planets, moons, suns |
| 6 | Ship | Player ships |
| 11 | Entity | NPCs |
| 22 | Deployable | Deployable structures |
| 23 | Starbase | POS structures |
| 25 | Asteroid | Asteroids |
| 65 | Structure | Citadels, stations |
| 87 | Fighter | Fighters |

### Group IDs

| ID | Name | Description |
|----|------|-------------|
| 12 | Container | Secure containers |
| 14 | Asteroid | Asteroid belt |
| 226 | Wreck | Ship wreck |
| 365 | Station | NPC station |
| 941 | Mobile Warp Disruptor | Warp bubble |
| 1246 | Mobile Tractor Unit | MTU |

### Type IDs (Common)

| ID | Name | Description |
|----|------|-------------|
| 34 | Tritanium | Common ore mineral |
| 35 | Pyerite | Common ore mineral |
| 36 | Mexallon | Common ore mineral |
| 1228 | Veldspar | Common asteroid |
| 17470 | Scordite | Common asteroid |

### Inventory Flags

| Flag | Name | Description |
|------|------|-------------|
| 4 | Hangar | Station hangar |
| 5 | Cargo | Ship cargo |
| 7 | Module | Fitted module |
| 87 | DroneBay | Drone bay |
| 90 | ShipHangar | Ship hangar |
| 134 | OreHold | Ore hold |
| 142 | FleetHangar | Fleet hangar |

### Movement Modes

| Mode | Name | Description |
|------|------|-------------|
| 0 | Unknown | Unknown state |
| 1 | Stop | Stopped |
| 2 | Orbit | Orbiting |
| 3 | Warp | In warp |
| 4 | Approach | Approaching |

---

## Performance Best Practices

### Use Distance2 for Range Comparisons

**CRITICAL OPTIMIZATION:** Use `Distance2` (squared distance) instead of `Distance` when comparing ranges.

```lavish
; ❌ SLOW - Distance requires expensive sqrt() calculation
if ${Entity.Distance} < 50000
{
    ; do something
}

; ✅ FAST - Distance2 is raw squared distance (no sqrt)
if ${Entity.Distance2} < ${Math.Calc[50000*50000]}
{
    ; do something
}
```

**Why this matters:** `Distance` = sqrt(x² + y² + z²) requires expensive square root. `Distance2` = x² + y² + z² is much faster. For range checks, compare squared values.

**Performance gain:** ~3-5x faster in tight loops with many entities.

### Cache Entity Queries

```lavish
; ❌ BAD - Querying every frame
while TRUE
{
    variable index:entity NPCs
    EVE:QueryEntities[NPCs, "CategoryID = 11"]
    ; ... process NPCs ...
    wait 1
}

; ✅ GOOD - Query periodically, cache results
variable index:entity NPCs
variable int lastQueryFrame = 0

while TRUE
{
    if ${Math.Calc[${Script.RunningTime} - ${lastQueryFrame}]} > 50
    {
        EVE:QueryEntities[NPCs, "CategoryID = 11"]
        lastQueryFrame:Set[${Script.RunningTime}]
    }
    ; ... process cached NPCs ...
    wait 1
}
```

### Use Query Filters

```lavish
; ❌ SLOW - Iterate all entities manually
variable iterator AllEntities
EVE:QueryEntities[AllEntities]
; ... iterate and filter in script ...

; ✅ FAST - Let ISXEVE filter in C++
variable index:entity CloseNPCs
EVE:QueryEntities[CloseNPCs, "CategoryID = 11 && IsNPC = TRUE && Distance < 100000"]
```

**Why this matters:** ISXEVE's C++ filtering is ~100x faster than LavishScript loops.

---

## Summary

This reference covers **the most commonly used ~20 ISXEVE datatypes** in EVE Online bot development.

### Key Takeaways

1. **Use modern inventory API** - Old `MyShip:GetCargo[]` is DEPRECATED (July 2020)
2. **Always check `(exists)`** before accessing entities, bookmarks, or any object that can disappear
3. **Use Distance2 for comparisons** - 3-5x faster than Distance in tight loops
4. **Use query filters** - Let ISXEVE filter in C++ instead of LavishScript loops
5. **Cache entity queries** - Don't query every frame, cache and refresh periodically
6. **Handle errors gracefully** - Entities die, modules fail, windows close

### Most Important Objects

- `EVE` - Global interface, queries, bookmarks
- `EVEWindow[Inventory]` - CRITICAL for modern cargo/inventory access
- `Me` - Current pilot, location, fleet, targeting
- `MyShip` - Current ship, modules, HP/cap stats
- `Entity` - Anything in space (NPCs, asteroids, wrecks, players)
- `Module` - Fitted modules, activation, stats
- `Item` - Inventory items, cargo, hangar
- `Bookmark` - Location bookmarks

### Performance Tips

- ✅ Use `Entity.Distance2` for range comparisons (3-5x faster)
- ✅ Use `EVE:QueryEntities[]` with filters (C++ speed)
- ✅ Cache entity queries, refresh periodically (not every frame)
- ✅ Check `(exists)` before re-accessing cached entities
- ❌ Don't use deprecated cargo API (`MyShip:GetCargo[]`)
- ❌ Don't iterate all entities manually
- ❌ Don't query entities every frame

---

*Last Updated: 2025-10-26*
*Part of ISXEVE Scripting Guide*
