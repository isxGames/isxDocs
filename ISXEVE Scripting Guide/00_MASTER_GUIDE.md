# ISXEVE Master Guide
## Complete API Reference and Navigation Hub

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

---

## Top-Level Objects (TLOs)

Top-Level Objects are globally accessible entry points to game data.

| TLO | Returns | Description | Documentation |
|-----|---------|-------------|---------------|
| **${Me}** | pilot | Your character/pilot | [03_API_Reference.md#me-object](03_API_Reference.md#me-object) |
| **${MyShip}** | ship | Your current ship | [03_API_Reference.md#myship-object](03_API_Reference.md#myship-object) |
| **${EVE}** | eve | Game universe/UI system | [03_API_Reference.md#eve-object](03_API_Reference.md#eve-object) |
| **${Local}** | chatwindow | Local chat channel | [03_API_Reference.md#local-object](03_API_Reference.md#local-object) |
| **${Station}** | station | Current station (if docked) | [03_API_Reference.md#station-object](03_API_Reference.md#station-object) |
| **${Entity[id]}** | entity | Entity by ID | [03_API_Reference.md#entity-datatypes](03_API_Reference.md#entity-datatypes) |
| **${Config}** | isxeveconfig | ISXEVE configuration | [03_API_Reference.md#config-object](03_API_Reference.md#config-object) |
| **${ISXEVE}** | extension | Extension info | [03_API_Reference.md#isxeve-object](03_API_Reference.md#isxeve-object) |

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
- `${MyShip.Modules}` - Access modules

**Common Methods:**
- `MyShip:Dock[]` - Dock at station
- `MyShip:Undock[]` - Undock from station
- `MyShip:Open[]` - Open cargo hold

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
- `${Module.ChargeQuantity}` - Ammo count
- `${Module.ChargeTypeID}` - Ammo type ID
- `${Module.ToEntity}` - Target entity (if targeted module)

**Module Methods:**
- `Module:Activate[]` - Activate module
- `Module:Deactivate[]` - Deactivate module
- `Module:Online[]` - Online module
- `Module:Offline[]` - Offline module
- `Module:ChangeAmmo[typeID]` - Change ammo type
- `Module:Click[]` - Click module

**Module Access Pattern:**
```lavishscript
; Get all modules
variable index:module ModuleList
EVE:QueryModules[ModuleList]

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
- `Item:MoveTo[locationID, quantity]` - Move item
- `Item:Jettison[]` - Jettison to space

**Cargo Pattern:**
```lavishscript
; Open cargo
MyShip:Open[]
wait 20

; Get cargo items
variable index:item CargoItems
EVE:QueryInventory[CargoItems]

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
- `MyShip:Dock[]` - Dock at station
- `MyShip:Undock[]` - Undock
- `Entity:Approach[]` - Approach entity
- `Entity:Orbit[distance]` - Orbit at distance
- `Entity:KeepAtRange[distance]` - Keep at range
- `EVE:Execute[CmdStopShip]` - Stop ship

**Autopilot:**
- `EVE:SetDestination[solarSystemID]` - Set destination
- `EVE:Execute[CmdToggleAutopilot]` - Toggle autopilot

**Warping:**
- `Entity:WarpTo[distance]` - Warp to entity
- `EVE:Execute[CmdWarpToLocation]` - Warp to location

**Documentation:** [03_API_Reference.md#navigation](03_API_Reference.md#navigation)

---

## UI Windows and Menus

### UI Datatypes

| Datatype | Description | Documentation |
|----------|-------------|---------------|
| **eveuiwindow** | EVE UI window | [03_API_Reference.md#eveuiwindow-datatype](03_API_Reference.md#eveuiwindow-datatype) |
| **chatwindow** | Chat window | [03_API_Reference.md#chatwindow-datatype](03_API_Reference.md#chatwindow-datatype) |

**Common UI Operations:**
- `EVE:Execute[CmdOpenCargoHold]` - Open cargo
- `EVE:Execute[CmdOpenInventory]` - Open inventory
- `EVE:Execute[CmdOpenMarket]` - Open market
- `EVE:CloseAllInventoryWindows[]` - Close all inventory windows

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
- `${Me.Fleet.Members}` - Get members
- `${Me.Fleet.IsBoss}` - Is fleet boss

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
| `CmdOpenCargoHold` | Open cargo |
| `CmdOpenInventory` | Open inventory |
| `CmdOpenMarket` | Open market |
| `CmdDockOrJumpOrActivateGate` | Dock/jump/activate |

**Documentation:** [01_LavishScript_Fundamentals.md#commands](01_LavishScript_Fundamentals.md#commands)

---

## Events

### Common ISXEVE Events

| Event | When Fired | Documentation |
|-------|------------|---------------|
| `OnFrame` | Every frame | [03_API_Reference.md#events](03_API_Reference.md#events) |
| `OnActiveTargetChanged` | Active target changed | [03_API_Reference.md#events](03_API_Reference.md#events) |
| `OnMyShipShieldsUpdate` | Shield HP changed | [03_API_Reference.md#events](03_API_Reference.md#events) |
| `OnMyShipArmorUpdate` | Armor HP changed | [03_API_Reference.md#events](03_API_Reference.md#events) |
| `OnMyShipCapacitorUpdate` | Capacitor changed | [03_API_Reference.md#events](03_API_Reference.md#events) |

**Event Usage:**
```lavishscript
atom OnTargetChanged()
{
    echo "Target changed"
}

Event[OnActiveTargetChanged]:AttachAtom[OnTargetChanged]
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
| **Dock at station** | `MyShip:Dock[]` | `MyShip:Dock[]` |
| **Stop ship** | `EVE:Execute[]` | `EVE:Execute[CmdStopShip]` |
| **Set destination** | `EVE:SetDestination[]` | `EVE:SetDestination[${sysID}]` |

### Common Patterns

**NULL/Existence Checking:**
```lavishscript
if ${Entity[${ID}](exists)}
{
    ; Safe to use entity
}
```

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
| **Complete Beginner** | [01_LavishScript_Fundamentals.md](01_LavishScript_Fundamentals.md) |
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
| **Advanced Patterns** | [22_Advanced_Scripting_Patterns.md](22_Advanced_Scripting_Patterns.md) |
| **Utility Patterns** | [23_Utility_Script_Patterns.md](23_Utility_Script_Patterns.md) |

---

<!-- CLAUDE_SKIP_START -->
---

*Last Updated: 2025-10-26*

*Master Guide for Quick API Reference*
<!-- CLAUDE_SKIP_END -->
