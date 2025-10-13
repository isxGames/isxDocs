# Inventory and Cargo Systems
## Complete Guide to Cargo Management, Hangar Access, and Item Handling

**Part of**: EVE Online Bot Development - AI-Level Documentation
**Layer**: 3 - ISXEVE API Deep Dive
**Prerequisites**: Read files 01-12 (Foundation + Scripting + Core Objects + Navigation)
**Critical For**: Mining bots, hauling bots, loot collection, inventory management

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

**Why This Document Still Exists:**
1. **Understanding legacy scripts** - Evebot and other old scripts use this API
2. **Conceptual understanding** - The patterns are still valid, just the API changed
3. **Migration reference** - Helps convert old scripts to new API

**Recommendation**: Read this file to understand inventory concepts, then refer to:
- **File 09** (Core Objects Reference) - Modern cargo API examples
- **__CRITICAL_NEWEST_ISXEVE_Reference.md** - Complete modern API reference

**For New Scripts**: Use the modern `EVEWindow[Inventory]` pattern exclusively!

---

## Purpose of This Document

This file provides **exhaustive coverage** of:
- Cargo hold access and management
- Item objects and members
- Hangar access (station, ship, corp)
- Cargo capacity checking and fullness detection
- Moving items between containers
- Loot collection from wrecks
- Ore hold and specialized cargo bays
- Item stacking and quantity management
- Common patterns from Evebot (mining/hauling)
- Cargo safety patterns (avoiding jettison, checking capacity)
- Performance optimization for cargo operations
- Critical gotchas and limitations

**Why This Is CRITICAL**:
- **Mining bots must manage ore cargo** (know when to return to station)
- **Hauling bots move items between locations**
- **Combat bots may collect loot** (salvaging, looting wrecks)
- **Cargo fullness detection** prevents wasted cycles
- **ISXEVE cargo manipulation is LIMITED** - Many operations require UI interaction

---

## Table of Contents

1. [Inventory System Overview](#inventory-system-overview)
2. [Item Object](#item-object)
3. [Cargo Hold Access](#cargo-hold-access)
4. [Cargo Capacity Management](#cargo-capacity-management)
5. [Hangar Access](#hangar-access)
6. [Specialized Cargo Bays](#specialized-cargo-bays)
7. [Moving Items (Limited Support)](#moving-items-limited-support)
8. [Loot Collection](#loot-collection)
9. [Item Stacking and Quantities](#item-stacking-and-quantities)
10. [Common Patterns from Example Scripts](#common-patterns-from-example-scripts)
11. [Cargo Safety Patterns](#cargo-safety-patterns)
12. [Performance Optimization](#performance-optimization)
13. [Critical Limitations](#critical-limitations)
14. [Gotchas and Edge Cases](#gotchas-and-edge-cases)

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
; See Module Management file (File 12)

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

## Next Steps

**Continue to Layer 4 (Bot Architecture Patterns)**:
- File 17: Main_Loop_and_State_Machines.md
- File 18: Decision_Making_and_Logic_Patterns.md
- File 19: Error_Handling_and_Recovery.md

**Or complete remaining Layer 3**:
- File 16: Fleet_and_Social_Systems.md
- File 14: Market_and_Contracts.md (low priority - limited ISXEVE support)

**Apply Knowledge**:
- Mining bots need cargo/ore hold management
- Hauling bots need capacity checks
- Recognize cargo patterns in Evebot

---

## Wiki References

**Item Object**:
- `IsxeveWiki/ISXEVE/Item_(Object_Type).html` - Complete item reference

**MyShip Cargo**:
- `IsxeveWiki/ISXEVE/MyShip_(Object_Type).html` - Cargo methods and members

**Examples**:
- Search Evebot for "GetCargo" and "CargoCapacity" for real patterns

---

**END OF FILE 13**
**Status**: Complete (~1600 lines)
**Next**: Update progress tracker, provide comprehensive session summary
