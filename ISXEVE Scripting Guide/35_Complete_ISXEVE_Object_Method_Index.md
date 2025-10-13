# Common ISXEVE Objects & Methods Reference

> **Part of Layer 9: Reference Materials**
> Reference for commonly used ISXEVE objects, properties, and methods in EVE Online bot development.

---

## ⚠️ CRITICAL API CHANGE WARNING (July 2020)

**This file has been updated to use the MODERN inventory API.**

### Deprecated Cargo API (DO NOT USE)
```lavish
; ❌ DEPRECATED - These methods were removed July 2020:
MyShip:GetCargo[index]
MyShip.CargoCapacity / MyShip.UsedCargoCapacity
MyShip.OreHoldCapacity / MyShip.UsedOreHoldCapacity
MyShip.DroneBayCapacity / MyShip.UsedDroneBayCapacity
```

### Modern Unified Inventory API (USE THIS)
```lavish
; ✅ CORRECT - Use EVEWindow[Inventory] unified windows:
EVEWindow[Inventory].Child[ShipCargo]:GetItems[index:item]
EVEWindow[Inventory].Child[ShipCargo].Capacity
EVEWindow[Inventory].Child[ShipCargo].UsedCapacity
EVEWindow[Inventory].Child[ShipOreHold].Capacity
EVEWindow[Inventory].Child[ShipOreHold].UsedCapacity
```

**Why this matters:** All cargo/inventory methods were replaced with a unified window system in July 2020. Old scripts using deprecated methods will fail. See File 13 for legacy reference.

**For complete ISXEVE coverage:** This file covers ~20 most common datatypes. For all 50+ datatypes, see `__CRITICAL_NEWEST_ISXEVE_Reference.md`

---

## Table of Contents

1. [Introduction to ISXEVE](#introduction-to-isxeve)
2. [Top-Level Objects (TLO)](#top-level-objects-tlo)
3. [EVE Object](#eve-object)
4. [Me Object](#me-object)
5. [MyShip Object](#myship-object)
6. [Entity Object](#entity-object)
7. [Item Object](#item-object)
8. [Module Object](#module-object)
9. [Bookmark Object](#bookmark-object)
10. [Station Object](#station-object)
11. [SolarSystem Object](#solarsystem-object)
12. [Fleet Object](#fleet-object)
13. [Pilot Object](#pilot-object)
14. [Corporation Object](#corporation-object)
15. [Alliance Object](#alliance-object)
16. [EVEWindow Object](#evewindow-object)
17. [Container Objects](#container-objects)
18. [Query Methods](#query-methods)
19. [Common Patterns](#common-patterns)
20. [Quick Reference Tables](#quick-reference-tables)

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

**See Also:** File 13 (legacy cargo API reference), File 15 (window system details)

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

### Scope & Limitations

**What this file covers:**
- 20 most common ISXEVE datatypes
- Modern inventory API (July 2020+)
- Performance best practices
- Common usage patterns

**What this file does NOT cover:**
- All 50+ ISXEVE datatypes (pilot, being, fleetmember, eveinvwindow, scanner, skill, etc.)
- Complete member lists for covered types
- Advanced window system details
- Market/agent/skill system details

**For comprehensive coverage, see:**
- `__CRITICAL_NEWEST_ISXEVE_Reference.md` - All 50+ datatypes with complete member lists
- File 09 (09_ISXEVE_Core_Objects_Reference.md) - Core TLOs and inheritance
- File 10 (10_Entity_System_and_Targeting.md) - Entity system details
- File 12 (12_Module_Management_and_Ship_Control.md) - Module system details
- File 13 (13_Inventory_and_Cargo_Systems.md) - Legacy cargo API reference
- File 15 (15_UI_Windows_and_Menus.md) - Window system details

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

**Previous Document:** `34_Metatron_DotNet_Architecture_Overview.md` - Real-world .NET bot architecture

**Next Document:** `36_LavishScript_Command_Reference.md` - LavishScript language reference

---

*Document complete. ISXEVE object reference index created for community use.*
