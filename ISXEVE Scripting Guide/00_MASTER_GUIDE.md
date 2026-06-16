# ISXEVE Master Guide
**Complete API Reference and Navigation Hub**

**Extension:** ISXEVE for InnerSpace
**Target Game:** EVE Online

---

## Purpose of This Guide

This master guide serves as your **quick reference hub** for all ISXEVE APIs, commands, objects, and documentation. Use this when you need to quickly find:

- A specific object or datatype
- Commands and their syntax
- Events you can respond to
- Links to detailed documentation

**For learning:** Start with [README.md](README.md)
**For quick lookup:** Use this guide

---

## Table of Contents

1. [Top-Level Objects (TLOs)](#top-level-objects-tlos)
2. [Core Datatypes](#core-datatypes)
3. [Entity and Pilot System](#entity-and-pilot-system)
4. [Ship and Module System](#ship-and-module-system)
5. [Inventory and Cargo](#inventory-and-cargo)
6. [Navigation and Movement](#navigation-and-movement)
7. [UI Windows and Menus](#ui-windows-and-menus)
8. [Fleet and Social](#fleet-and-social)
9. [Commands](#commands)
10. [Events](#events)
11. [Quick Reference Tables](#quick-reference-tables)
12. [Documentation Navigation](#documentation-navigation)

---

## Top-Level Objects (TLOs)

Top-Level Objects are globally accessible entry points to game data.

| TLO | Returns | Description | Documentation |
|-----|---------|-------------|---------------|
| **${Me}** | character | Your character (inherits pilot, being) | [03_API_Reference.md#me-object](03_API_Reference.md#me-object) |
| **${MyShip}** | ship | Your current ship | [03_API_Reference.md#myship-object](03_API_Reference.md#myship-object) |
| **${EVE}** | eve | Game universe/UI system | [03_API_Reference.md#eve-object](03_API_Reference.md#eve-object) |
| **${Local[id]}** / **${Local[name]}** | pilot | A pilot in local chat (by CharID or name) | [03_API_Reference.md#local-object](03_API_Reference.md#local-object) |
| **${Me.Station}** | station / structure | Current station (when docked) | [03_API_Reference.md#station-object](03_API_Reference.md#station-object) |
| **${Entity[id]}** | entity | Entity by ID | [03_API_Reference.md#entity-datatypes](03_API_Reference.md#entity-datatypes) |
| **${EVEWindow[name]}** | evewindow | A game window by name | [03_API_Reference.md#window-datatypes](03_API_Reference.md#window-datatypes) |
| **${ISXEVE}** | isxeve | Extension status/info | [03_API_Reference.md#isxeve-object](03_API_Reference.md#isxeve-object) |

**Key Pattern:**
```lavishscript
; Access character
echo "${Me.Name}"

; Access ship
echo "${MyShip.ShieldPct}%"

; Access entity by ID
if ${Entity[${ID}](exists)}
{
    echo "${Entity[${ID}].Name}"
}
```

---

## Core Datatypes

### Character/Pilot

| Datatype | Description | Documentation |
|----------|-------------|---------------|
| **pilot** | Character data (Me) | [03_API_Reference.md#pilot-datatype](03_API_Reference.md#pilot-datatype) |
| **me** | Alias for pilot | [03_API_Reference.md#me-object](03_API_Reference.md#me-object) |

**Common Members:**
- `${Me.Name}` - Character name
- `${Me.InSpace}` - In space vs docked
- `${Me.Corp}` - Corporation name
- `${Me.Alliance}` - Alliance name
- `${Me.Wallet}` - ISK in wallet
- `${Me.ToEntity}` - Get entity object for self

### Ship

| Datatype | Description | Documentation |
|----------|-------------|---------------|
| **ship** | Ship object (MyShip) | [03_API_Reference.md#ship-datatype](03_API_Reference.md#ship-datatype) |
| **myship** | Alias for ship | [03_API_Reference.md#myship-object](03_API_Reference.md#myship-object) |

**Common Members:**
- `${MyShip.Name}` - Ship name
- `${MyShip.ShieldPct}` - Shield percentage
- `${MyShip.ArmorPct}` - Armor percentage
- `${MyShip.StructurePct}` - Structure percentage
- `${MyShip.CapacitorPct}` - Capacitor percentage
- `${MyShip.CargoCapacity}` - Cargo bay size
- `${MyShip.UsedCargoCapacity}` - Cargo used
- `MyShip:GetModules[index:module]` - Populate an index with all fitted modules

**Common Methods:**
- `MyShip:Open[]` - Open cargo hold
- `EVE:Execute[CmdExitStation]` - Undock from station

---

## Entity and Pilot System

### Entity Datatypes

| Datatype | Description | Documentation |
|----------|-------------|---------------|
| **entity** | Any object in space (ships, NPCs, structures) | [03_API_Reference.md#entity-datatype](03_API_Reference.md#entity-datatype) |
| **pilot** | Character/pilot | [03_API_Reference.md#pilot-datatype](03_API_Reference.md#pilot-datatype) |
| **owner** | Entity owner | [03_API_Reference.md#owner-datatype](03_API_Reference.md#owner-datatype) |
| **alliance** | Alliance | [03_API_Reference.md#alliance-datatype](03_API_Reference.md#alliance-datatype) |

**Entity Members:**
- `${Entity.ID}` - Unique entity ID (int64)
- `${Entity.Name}` - Entity name
- `${Entity.Distance}` - Distance to entity
- `${Entity.Type}` - Entity type
- `${Entity.Group}` - Entity group
- `${Entity.IsNPC}` - Is NPC (bool)
- `${Entity.IsPC}` - Is player character (bool)
- `${Entity.Owner}` - Owner object
- `${Entity.IsLockedTarget}` - Is locked (bool)
- `${Entity.IsActiveTarget}` - Is active target (bool)

**Entity Methods:**
- `Entity:LockTarget[]` - Lock target
- `Entity:UnlockTarget[]` - Unlock target
- `Entity:MakeActiveTarget[]` - Set as active target
- `Entity:Approach[]` - Approach entity
- `Entity:Orbit[distance]` - Orbit at distance
- `Entity:KeepAtRange[distance]` - Keep at range

**Targeting Pattern:**
```lavishscript
; Lock entity
if ${Entity[${ID}](exists)} && !${Entity[${ID}].IsLockedTarget}
{
    Entity[${ID}]:LockTarget[]

    ; Wait for lock
    variable int counter = 0
    while !${Entity[${ID}].IsLockedTarget} && ${counter} < 150
    {
        wait 10
        counter:Inc
    }
}
```

---

## Ship and Module System

### Module Datatypes

| Datatype | Description | Documentation |
|----------|-------------|---------------|
| **module** | Ship module/equipment | [03_API_Reference.md#module-datatype](03_API_Reference.md#module-datatype) |
| **charge** | Module ammo/charge | [03_API_Reference.md#charge-datatype](03_API_Reference.md#charge-datatype) |

**Module Members:**
- `${Module.ID}` - Module ID
- `${Module.ToItem.Name}` - Module name
- `${Module.IsOnline}` - Is online (bool)
- `${Module.IsActive}` - Is active (bool)
- `${Module.IsActivatable}` - Can activate (bool)
- `${Module.IsReloadingAmmo}` - Is reloading (bool)
- `${Module.CurrentCharges}` - Ammo count
- `${Module.Charge.TypeID}` - Ammo type ID
- `${Module.Target}` - Target entity (if targeted module)

**Module Methods:**
- `Module:Activate[]` - Activate module
- `Module:Deactivate[]` - Deactivate module
- `Module:PutOnline[]` - Online module
- `Module:PutOffline[]` - Offline module
- `Module:ChangeAmmo[itemID]` - Change ammo type
- `Module:Click[]` - Click module

**Module Access Pattern:**
```lavishscript
; Get all modules
variable index:module ModuleList
MyShip:GetModules[ModuleList]

; Activate first weapon
if ${MyShip.Module[1](exists)} && ${MyShip.Module[1].IsActivatable}
{
    MyShip.Module[1]:Activate[]
}

; Iterate all modules
variable iterator Mod
ModuleList:GetIterator[Mod]
if ${Mod:First(exists)}
{
    do
    {
        if ${Mod.Value.IsActivatable} && !${Mod.Value.IsActive}
        {
            Mod.Value:Activate[]
        }
    }
    while ${Mod:Next(exists)}
}
```

---

## Inventory and Cargo

### Inventory Datatypes

| Datatype | Description | Documentation |
|----------|-------------|---------------|
| **item** | Inventory item | [03_API_Reference.md#item-datatype](03_API_Reference.md#item-datatype) |
| **cargo** | Cargo container | [03_API_Reference.md#cargo-datatype](03_API_Reference.md#cargo-datatype) |
| **hangar** | Station hangar | [03_API_Reference.md#hangar-datatype](03_API_Reference.md#hangar-datatype) |

**Item Members:**
- `${Item.ID}` - Item ID
- `${Item.Name}` - Item name
- `${Item.Quantity}` - Stack size
- `${Item.Volume}` - Volume per unit
- `${Item.TypeID}` - Type ID
- `${Item.GroupID}` - Group ID

**Item Methods:**
- `Item:MoveTo[toID, toDestination, quantity, folder]` - Move item (destination is a canonical slot/destination name)
- `Item:Jettison[]` - Jettison to space

**Cargo Pattern:**
```lavishscript
; Open cargo
MyShip:Open[]
wait 20

; Get cargo items
variable index:item CargoItems
MyShip:GetCargo[CargoItems]

; Iterate cargo
variable iterator Item
CargoItems:GetIterator[Item]
if ${Item:First(exists)}
{
    do
    {
        echo "Item: ${Item.Value.Name} x${Item.Value.Quantity}"
    }
    while ${Item:Next(exists)}
}
```

---

## Navigation and Movement

### Navigation Methods

**Ship Movement:**
- `Entity[stationID]:Dock[]` - Dock at station/citadel
- `EVE:Execute[CmdExitStation]` - Undock
- `Entity:Approach[]` - Approach entity
- `Entity:Orbit[distance]` - Orbit at distance
- `Entity:KeepAtRange[distance]` - Keep at range
- `EVE:Execute[CmdStopShip]` - Stop ship

**Autopilot:**
- `Universe[${solarSystemID}]:SetDestination[]` - Set destination (also on `EVE.Station[id]` / `Bookmark[label]`)
- `EVE:AddWaypoint[solarSystemID]` - Append a waypoint to the route
- `EVE:Execute[CmdToggleAutopilot]` - Toggle autopilot

**Warping:**
- `Entity:WarpTo[distance]` - Warp to entity
- `Bookmark[label]:WarpTo[distance]` - Warp to a bookmark location

**Documentation:** [03_API_Reference.md#navigation](03_API_Reference.md#navigation)

---

## UI Windows and Menus

### UI Datatypes

| Datatype | Description | Documentation |
|----------|-------------|---------------|
| **eveuiwindow** | EVE UI window | [03_API_Reference.md#eveuiwindow-datatype](03_API_Reference.md#eveuiwindow-datatype) |
| **chatwindow** | Chat window | [03_API_Reference.md#chatwindow-datatype](03_API_Reference.md#chatwindow-datatype) |

**Common UI Operations:**
- `EVE:Execute[OpenCargoHoldOfActiveShip]` - Open cargo
- `EVE:Execute[OpenInventory]` - Open inventory
- `EVE:Execute[OpenMarket]` - Open market
- `EVEWindow[Inventory]:Close[]` - Close the inventory window

**Documentation:** [03_API_Reference.md#ui-datatypes](03_API_Reference.md#ui-datatypes)

---

## Fleet and Social

### Fleet Datatypes

| Datatype | Description | Documentation |
|----------|-------------|---------------|
| **fleet** | Fleet information | [03_API_Reference.md#fleet-datatype](03_API_Reference.md#fleet-datatype) |
| **fleetmember** | Fleet member | [03_API_Reference.md#fleetmember-datatype](03_API_Reference.md#fleetmember-datatype) |

**Fleet Operations:**
- `${Me.Fleet}` - Fleet object
- `Me.Fleet:GetMembers[index:fleetmember]` - Populate an index with all fleet members
- `${Me.Fleet.IsFleetCommander}` - You are the fleet commander (bool)

**Documentation:** [03_API_Reference.md#fleet-datatypes](03_API_Reference.md#fleet-datatypes)

---

## Commands

### ISXEVE Commands

| Command | Syntax | Description |
|---------|--------|-------------|
| **relay** | `relay "session" AtomName [params]` | Send atom to another session |
| **waitframe** | `waitframe` | Wait one frame |
| **wait** | `wait <deciseconds>` | Wait time in 1/10 seconds |

### EVE Execute Commands

Common `EVE:Execute[]` commands:

| Command | Description |
|---------|-------------|
| `CmdStopShip` | Stop ship |
| `CmdToggleAutopilot` | Toggle autopilot |
| `OpenCargoHoldOfActiveShip` | Open cargo |
| `OpenInventory` | Open inventory |
| `OpenMarket` | Open market |
| `CmdStargateJump` | Activate/jump stargate |

**Documentation:** [01b_LavishScript_Reference.md#2-commands](01b_LavishScript_Reference.md#2-commands) (exhaustive command inventory) or [01_LavishScript_Fundamentals.md](01_LavishScript_Fundamentals.md) (tutorial-style introduction).

---

## Events

### Common ISXEVE Events

| Event | When Fired | Documentation |
|-------|------------|---------------|
| `ISXEVE_onFrame` | Every ISXEVE pulse (canonical pulse event) | [03_API_Reference.md#events](03_API_Reference.md#events) |
| `EVE_OnChannelMessage` | A chat-channel message is received | [03_API_Reference.md#events](03_API_Reference.md#events) |
| `EVE_OnSurveyScanData` | Survey-scanner results are received | [03_API_Reference.md#events](03_API_Reference.md#events) |
| `isxGames_onHTTPResponse` | An HTTP response is received (`GetURL`/`PostURL`) | [03_API_Reference.md#events](03_API_Reference.md#events) |

**Event Usage:**
```lavishscript
atom OnISXEVEFrame()
{
    if ${Me.ToEntity(exists)} && ${MyShip.ShieldPct} < 30
        echo "Shield low: ${MyShip.ShieldPct.Int}%"
}

Event[ISXEVE_onFrame]:AttachAtom[OnISXEVEFrame]
```

**Documentation:** [03_API_Reference.md#events](03_API_Reference.md#events)

---

## Quick Reference Tables

### Task-Based Quick Reference

| Task | API/Method | Example |
|------|------------|---------|
| **Get character name** | `${Me.Name}` | `echo "${Me.Name}"` |
| **Get ISK wallet** | `${Me.Wallet}` | `echo "${Me.Wallet}"` |
| **Check if in space** | `${Me.InSpace}` | `if ${Me.InSpace}` |
| **Get ship shields** | `${MyShip.ShieldPct}` | `echo "${MyShip.ShieldPct}%"` |
| **Lock target** | `Entity:LockTarget[]` | `Entity[${ID}]:LockTarget[]` |
| **Activate module** | `Module:Activate[]` | `MyShip.Module[1]:Activate[]` |
| **Approach entity** | `Entity:Approach[]` | `Entity[${ID}]:Approach[]` |
| **Orbit entity** | `Entity:Orbit[dist]` | `Entity[${ID}]:Orbit[5000]` |
| **Open cargo** | `MyShip:Open[]` | `MyShip:Open[]` |
| **Dock at station** | `Entity:Dock[]` | `Entity[${StationID}]:Dock[]` |
| **Stop ship** | `EVE:Execute[]` | `EVE:Execute[CmdStopShip]` |
| **Set destination** | `Universe[id]:SetDestination[]` | `Universe[${sysID}]:SetDestination[]` |

### Common Patterns

**NULL/Existence Checking:** See [Existence Checks](README.md#existence-checks) in the README for the canonical `if ${X(exists)}` pattern.

**Wait for Condition:**
```lavishscript
variable int counter = 0
while !${Condition} && ${counter} < 100
{
    wait 10
    counter:Inc
}
```

**Iterate Collection:**
```lavishscript
variable iterator Item
Collection:GetIterator[Item]
if ${Item:First(exists)}
{
    do
    {
        ; Process Item.Value
    }
    while ${Item:Next(exists)}
}
```

---

## Documentation Navigation

### By Learning Level

| Level | Start Here |
|-------|------------|
| **Complete Beginner** | [01_LavishScript_Fundamentals.md](01_LavishScript_Fundamentals.md) (tutorial) |
| **LavishScript / Inner Space lookup** | [01b_LavishScript_Reference.md](01b_LavishScript_Reference.md) (exhaustive command/datatype/TLO reference) |
| **ISXEVE Beginner** | [02_Quick_Start_Guide.md](02_Quick_Start_Guide.md) |
| **Intermediate** | [05_Patterns_And_Best_Practices.md](05_Patterns_And_Best_Practices.md) |
| **Advanced** | [07_Advanced_Patterns_And_Examples.md](07_Advanced_Patterns_And_Examples.md) |
| **Quick Lookup** | This guide (00_MASTER_GUIDE.md) |

### By Bot Type

| Bot Type | Documentation |
|----------|---------------|
| **Combat Bot** | [15_Combat_Automation.md](15_Combat_Automation.md) |
| **Mining Bot** | [16_Mining_And_Hauling.md](16_Mining_And_Hauling.md) |
| **Fleet Coordinator** | [17_Fleet_Operations.md](17_Fleet_Operations.md) |
| **Multi-Purpose** | [18_Bot_Architecture_Analysis.md](18_Bot_Architecture_Analysis.md) |

### By Topic

| Topic | Documentation |
|-------|---------------|
| **API Reference** | [03_API_Reference.md](03_API_Reference.md) |
| **Core Concepts** | [04_Core_Concepts.md](04_Core_Concepts.md) |
| **Working Examples** | [06_Working_Examples.md](06_Working_Examples.md) |
| **UI Creation (LGUI1)** | [08_LavishGUI1_UI_Guide.md](08_LavishGUI1_UI_Guide.md) |
| **UI Creation (LGUI2)** | [10_LavishGUI2_UI_Guide.md](10_LavishGUI2_UI_Guide.md) |
| **JSON in LavishScript** | [13_JSON_Guide.md](13_JSON_Guide.md) |
| **LavishMachine (Async)** | [14_LavishMachine_Guide.md](14_LavishMachine_Guide.md) |
| **Debugging** | [20_Debugging_And_Troubleshooting.md](20_Debugging_And_Troubleshooting.md) |
| **.NET Development** | [19_DotNet_Development.md](19_DotNet_Development.md) |
| **Advanced Patterns** | [21_Advanced_Scripting_Patterns.md](21_Advanced_Scripting_Patterns.md) |
| **Utility Patterns** | [22_Utility_Script_Patterns.md](22_Utility_Script_Patterns.md) |
---
