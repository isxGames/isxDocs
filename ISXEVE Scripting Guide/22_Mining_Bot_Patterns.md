# Mining Bot Patterns - Practical Implementation Guide

**Purpose**: This file provides complete, working patterns for mining bots - from simple belt miners to advanced Orca-supported fleet operations. These are battle-tested patterns extracted from EVEBot.

**Target Reader**: AI implementing mining automation

**Prerequisites**:
- Files 01-21 (All foundation, API, architecture, combat)
- Understanding of mining mechanics (File 01)
- Knowledge of inventory and cargo (File 13)

**Wiki References**:
- Mining modules: `IsxeveWiki/DataType/module.html`
- Asteroids: `IsxeveWiki/DataType/entity.html`
- Cargo holds: `IsxeveWiki/DataType/ship.html#Cargo`

---

## Table of Contents

1. [Mining Bot Overview](#mining-bot-overview)
2. [Mining Fundamentals](#mining-fundamentals)
3. [Belt Selection and Navigation](#belt-selection-and-navigation)
4. [Asteroid Selection Patterns](#asteroid-selection-patterns)
5. [Survey Scanner Usage](#survey-scanner-usage)
6. [Mining Laser Activation](#mining-laser-activation)
7. [Cargo Management](#cargo-management)
8. [Station Return and Unloading](#station-return-and-unloading)
9. [Ice Mining Patterns](#ice-mining-patterns)
10. [Orca-Supported Mining](#orca-supported-mining)
11. [Complete Working Examples](#complete-working-examples)
12. [Common Mining Problems](#common-mining-problems)

---

## Mining Bot Overview

Mining bots automate ore/ice extraction from asteroid belts and anomalies.

### Mining Bot Types

| Type | Purpose | Ship Examples |
|------|---------|---------------|
| Solo Belt Miner | Mine solo in asteroid belts | Retriever, Covetor |
| Ice Miner | Harvest ice in ice belts | Skiff, Mackinaw |
| Anomaly Miner | Mine in ore anomalies | Hulk, Covetor |
| Fleet Miner | Coordinate with Orca/Rorqual | Any mining barge |
| AFK Miner | Passive mining (Procurer tank) | Procurer |

### Mining Loop Architecture

```
┌─────────────┐
│ Bot Pulse   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Safety      │ Check hostiles, tank
│ Checks      │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Cargo Check │ Full? Return to station
│             │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Location    │ In belt? Warp to belt
│ Check       │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Asteroid    │ Find/lock best asteroid
│ Selection   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Mine        │ Activate miners, wait
│             │
└─────────────┘
```

---

## Mining Fundamentals

### Ore Types and Values

```lavish
; ===== ORE TYPE PRIORITIES =====

variable(global) collection:int OreValues

function LoadOreValues()
{
    ; High value (prioritize)
    OreValues:Set["Jaspet", 100]
    OreValues:Set["Hemorphite", 90]
    OreValues:Set["Kernite", 80]
    OreValues:Set["Hedbergite", 70]

    ; Medium value
    OreValues:Set["Plagioclase", 40]
    OreValues:Set["Pyroxeres", 30]
    OreValues:Set["Scordite", 20]

    ; Low value (if nothing else)
    OreValues:Set["Veldspar", 10]
}

function GetOreValue(string oreName)
{
    ; Check each ore type
    variable iterator OreType
    OreValues:GetIterator[OreType]

    if ${OreType:First(exists)}
        do
        {
            if ${oreName.Find["${OreType.Key}"]}
            {
                return ${OreType.Value}
            }
        }
        while ${OreType:Next(exists)}

    return 0
}
```

### Mining Module Types

```lavish
; ===== MINING MODULE DETECTION =====

function IsMiningModule(item module)
{
    ; Check module group
    if ${module.ToItem.Group.Equal["Mining Laser"]} || \
       ${module.ToItem.Group.Equal["Strip Miner"]} || \
       ${module.ToItem.Group.Equal["Ice Harvester"]}
    {
        return TRUE
    }

    return FALSE
}

function GetOptimalMiningRange()
{
    ; Check ship modules for mining range
    variable int i
    variable float maxRange = 0

    for (i:Set[1]; ${i} <= ${MyShip.ModuleCount}; i:Inc)
    {
        variable item module = ${MyShip.Module[${i}]}

        if ${IsMiningModule[module]}
        {
            ; Get module optimal range (simplified - real check would use module.OptimalRange)
            variable float moduleRange = 15000  ; Default mining laser range

            if ${module.ToItem.Name.Find["Strip"]}
            {
                moduleRange:Set[20000]  ; Strip miners longer range
            }

            if ${moduleRange} > ${maxRange}
            {
                maxRange:Set[${moduleRange}]
            }
        }
    }

    return ${maxRange}
}
```

---

## Belt Selection and Navigation

### Random Belt Selection

```lavish
; ===== RANDOM BELT SELECTION =====

function WarpToRandomBelt()
{
    variable index:entity Belts
    variable iterator Belt

    ; Find all asteroid belts
    EVE:QueryEntities[Belts, "GroupID = GROUPID_ASTEROIDBELT"]
    Belts:GetIterator[Belt]

    if ${Belts.Used} == 0
    {
        echo "ERROR: No asteroid belts found in system"
        return FALSE
    }

    ; Pick random belt
    variable int randomIdx = ${Math.Rand[1,${Belts.Used}]}
    variable entity targetBelt = ${Belts.Get[${randomIdx}]}

    echo "Warping to ${targetBelt.Name}"
    targetBelt:WarpTo[0]

    wait 100  ; Wait for warp to complete

    return TRUE
}
```

### Bookmark-Based Belt Selection

**EVEBot Pattern**: Use bookmarked mining locations

```lavish
; ===== BOOKMARK BELT SELECTION =====
; Based on EVEBot obj_Asteroids.iss

variable string BeltBookmarkPrefix = "Mining -"
variable int64 LastUsedBookmarkID = 0

function WarpToBookmarkedBelt()
{
    ; Refresh bookmarks
    EVE:RefreshBookmarks
    wait 10

    ; Get all bookmarks
    variable index:bookmark Bookmarks
    EVE:GetBookmarks[Bookmarks]

    ; Filter to mining bookmarks in current system
    variable iterator Bookmark
    Bookmarks:GetIterator[Bookmark]

    variable index:bookmark FilteredBookmarks

    if ${Bookmark:First(exists)}
        do
        {
            ; Check if mining bookmark
            if !${Bookmark.Value.Label.Find["${BeltBookmarkPrefix}"]}
                continue

            ; Check if in current system
            if ${Bookmark.Value.SolarSystemID} != ${Me.SolarSystemID}
                continue

            ; Check if too close (already there)
            if ${Bookmark.Value.Distance} < 150000  ; Within warp range
                continue

            ; Check if last used
            if ${Bookmark.Value.ID} == ${LastUsedBookmarkID}
                continue

            ; Add to filtered list
            FilteredBookmarks:Insert[${Bookmark.Value}]
        }
        while ${Bookmark:Next(exists)}

    ; No bookmarks found
    if ${FilteredBookmarks.Used} == 0
    {
        echo "No suitable mining bookmarks found"
        return FALSE
    }

    ; Pick random from filtered
    variable int randomIdx = ${Math.Rand[1,${FilteredBookmarks.Used}]}
    variable bookmark targetBookmark = ${FilteredBookmarks.Get[${randomIdx}]}

    echo "Warping to bookmark: ${targetBookmark.Label}"
    EVE:WarpToBookMarkID[${targetBookmark.ID}]

    LastUsedBookmarkID:Set[${targetBookmark.ID}]

    wait 100  ; Wait for warp

    return TRUE
}
```

### Anomaly Selection

```lavish
; ===== ANOMALY ORE SITE SELECTION =====

function WarpToOreAnomaly()
{
    ; Scan for probe results (ore anomalies)
    variable index:systemsite ProbeResults
    EVE:GetProbeResults[ProbeResults]

    variable iterator Site
    ProbeResults:GetIterator[Site]

    variable index:systemsite OreAnomalies

    if ${Site:First(exists)}
        do
        {
            ; Check if ore anomaly
            if ${Site.Value.Name.Find["Ore"]} || \
               ${Site.Value.Name.Find["Asteroid"]}
            {
                OreAnomalies:Insert[${Site.Value}]
            }
        }
        while ${Site:Next(exists)}

    if ${OreAnomalies.Used} == 0
    {
        echo "No ore anomalies found"
        return FALSE
    }

    ; Pick first ore anomaly
    variable systemsite targetAnomaly = ${OreAnomalies.Get[1]}

    echo "Warping to anomaly: ${targetAnomaly.Name}"
    EVE:WarpToSite[${targetAnomaly.ID}]

    wait 100

    return TRUE
}
```

---

## Asteroid Selection Patterns

### Pattern 1: Nearest Asteroid

```lavish
; ===== NEAREST ASTEROID SELECTION =====

function GetNearestAsteroid()
{
    variable index:entity Asteroids
    variable iterator Asteroid

    ; Query all asteroids
    EVE:QueryEntities[Asteroids, "CategoryID = CATEGORYID_ASTEROID"]
    Asteroids:GetIterator[Asteroid]

    ; Find nearest unlocked
    if ${Asteroid:First(exists)}
        do
        {
            ; Skip if already locked
            if ${Asteroid.Value.IsLockedTarget} || ${Asteroid.Value.BeingTargeted}
                continue

            ; Check if in range
            if ${Asteroid.Value.Distance} > ${GetOptimalMiningRange}
                continue

            ; Found good asteroid
            return ${Asteroid.Value.ID}
        }
        while ${Asteroid:Next(exists)}

    return 0
}
```

### Pattern 2: By Ore Type Priority

```lavish
; ===== ORE TYPE PRIORITY SELECTION =====

function GetBestAsteroidByOreType()
{
    variable index:entity Asteroids
    variable iterator Asteroid

    EVE:QueryEntities[Asteroids, "CategoryID = CATEGORYID_ASTEROID && Distance < ${GetOptimalMiningRange}"]
    Asteroids:GetIterator[Asteroid]

    variable int64 BestAsteroid = 0
    variable int BestValue = 0

    if ${Asteroid:First(exists)}
        do
        {
            ; Skip locked
            if ${Asteroid.Value.IsLockedTarget} || ${Asteroid.Value.BeingTargeted}
                continue

            ; Get ore value
            variable int oreValue = ${GetOreValue["${Asteroid.Value.Name}"]}

            ; Update best
            if ${oreValue} > ${BestValue}
            {
                BestValue:Set[${oreValue}]
                BestAsteroid:Set[${Asteroid.Value.ID}]
            }
        }
        while ${Asteroid:Next(exists)}

    if ${BestAsteroid} > 0
    {
        echo "Best asteroid: ${Entity[${BestAsteroid}].Name} (value: ${BestValue})"
    }

    return ${BestAsteroid}
}
```

### Pattern 3: Grouped Asteroid Selection

**EVEBot Pattern**: Select asteroids close to each other

```lavish
; ===== GROUPED ASTEROID SELECTION =====
; Based on EVEBot obj_Asteroids.iss NearestAsteroid

variable float GROUP_RANGE_MULTIPLIER = 0.75

function GetGroupedAsteroid()
{
    variable index:entity Asteroids
    variable iterator Asteroid

    EVE:QueryEntities[Asteroids, "CategoryID = CATEGORYID_ASTEROID && Distance < ${GetOptimalMiningRange}"]
    Asteroids:GetIterator[Asteroid]

    if !${Asteroid:First(exists)}
        return 0

    ; Get currently locked asteroids
    variable index:entity LockedAsteroids
    Me:GetTargets[LockedAsteroids]

    ; If no locked asteroids, just return nearest
    if ${LockedAsteroids.Used} == 0
    {
        do
        {
            if !${Asteroid.Value.IsLockedTarget} && !${Asteroid.Value.BeingTargeted}
            {
                return ${Asteroid.Value.ID}
            }
        }
        while ${Asteroid:Next(exists)}

        return 0
    }

    ; Find asteroid close to ALL currently locked asteroids
    variable float maxDistFromLocked = ${Math.Calc[${GetOptimalMiningRange} * ${GROUP_RANGE_MULTIPLIER}]}

    Asteroid:First

    do
    {
        ; Skip locked
        if ${Asteroid.Value.IsLockedTarget} || ${Asteroid.Value.BeingTargeted}
            continue

        ; Check distance to ALL locked asteroids
        variable bool TooFarFromGroup = FALSE
        variable iterator LockedAsteroid
        LockedAsteroids:GetIterator[LockedAsteroid]

        if ${LockedAsteroid:First(exists)}
            do
            {
                variable float distToLocked = ${Asteroid.Value.DistanceTo[${LockedAsteroid.Value.ID}]}

                if ${distToLocked} > ${maxDistFromLocked}
                {
                    TooFarFromGroup:Set[TRUE]
                    break
                }
            }
            while ${LockedAsteroid:Next(exists)}

        ; If close to all locked asteroids, this is good!
        if !${TooFarFromGroup}
        {
            echo "Found grouped asteroid: ${Asteroid.Value.Name}"
            return ${Asteroid.Value.ID}
        }
    }
    while ${Asteroid:Next(exists)}

    echo "No grouped asteroids found, selecting nearest"
    return ${GetNearestAsteroid}
}
```

---

## Survey Scanner Usage

### Activate Survey Scanner

```lavish
; ===== SURVEY SCANNER ACTIVATION =====

function ActivateSurveyScanner()
{
    variable int i
    for (i:Set[1]; ${i} <= ${MyShip.ModuleCount}; i:Inc)
    {
        variable item module = ${MyShip.Module[${i}]}

        if ${module.ToItem.Group.Equal["Survey Scanner"]}
        {
            if !${module.IsActive} && ${module.IsOnline}
            {
                echo "Activating survey scanner"
                module:Activate
                return TRUE
            }
        }
    }

    return FALSE
}
```

### Process Survey Data

**EVEBot Pattern**: Event-driven survey data processing

```lavish
; ===== SURVEY DATA PROCESSING =====

variable index:entity SurveyedAsteroids
variable int LastSurveyResultCount = 0
variable int LastSurveyTime = 0

; Register event handler
function InitializeSurveyScanner()
{
    LavishScript:RegisterEvent[EVE_OnSurveyScanData]
    Event[EVE_OnSurveyScanData]:AttachAtom[OnSurveyData]
}

; Event handler
atom OnSurveyData(int resultCount)
{
    echo "Survey scan completed: ${resultCount} asteroids"
    LastSurveyResultCount:Set[${resultCount}]
    LastSurveyTime:Set[${LavishScript.RunningTime}]

    call ProcessSurveyResults
}

function ProcessSurveyResults()
{
    ; Clear old survey data
    SurveyedAsteroids:Clear

    ; Get all asteroids
    variable index:entity Asteroids
    EVE:QueryEntities[Asteroids, "CategoryID = CATEGORYID_ASTEROID"]

    ; Survey scanner marks asteroids with quantity data
    variable iterator Asteroid
    Asteroids:GetIterator[Asteroid]

    if ${Asteroid:First(exists)}
        do
        {
            ; Only add asteroids with quantity data (surveyed)
            if ${Asteroid.Value.Quantity} > 0
            {
                SurveyedAsteroids:Insert[${Asteroid.Value}]
            }
        }
        while ${Asteroid:Next(exists)}

    echo "Surveyed ${SurveyedAsteroids.Used} asteroids"
}

function GetBestSurveyedAsteroid()
{
    ; Get highest quantity asteroid
    variable iterator Asteroid
    SurveyedAsteroids:GetIterator[Asteroid]

    variable int64 BestAsteroid = 0
    variable int BestQuantity = 0

    if ${Asteroid:First(exists)}
        do
        {
            ; Skip locked
            if ${Asteroid.Value.IsLockedTarget} || ${Asteroid.Value.BeingTargeted}
                continue

            ; Check in range
            if ${Asteroid.Value.Distance} > ${GetOptimalMiningRange}
                continue

            ; Update best
            if ${Asteroid.Value.Quantity} > ${BestQuantity}
            {
                BestQuantity:Set[${Asteroid.Value.Quantity}]
                BestAsteroid:Set[${Asteroid.Value.ID}]
            }
        }
        while ${Asteroid:Next(exists)}

    if ${BestAsteroid} > 0
    {
        echo "Best surveyed: ${Entity[${BestAsteroid}].Name} (${BestQuantity} units)"
    }

    return ${BestAsteroid}
}
```

---

## Mining Laser Activation

### Basic Mining

```lavish
; ===== BASIC MINING LASER ACTIVATION =====

function ActivateMinersOnTarget(int64 asteroidID)
{
    if !${Entity[${asteroidID}](exists)}
    {
        echo "ERROR: Asteroid ${asteroidID} doesn't exist"
        return FALSE
    }

    if !${Entity[${asteroidID}].IsLockedTarget}
    {
        echo "ERROR: Asteroid not locked"
        return FALSE
    }

    ; Activate all mining lasers
    variable int i
    variable int activated = 0

    for (i:Set[1]; ${i} <= ${MyShip.ModuleCount}; i:Inc)
    {
        variable item module = ${MyShip.Module[${i}]}

        if ${IsMiningModule[module]}
        {
            if !${module.IsActive} && ${module.IsOnline}
            {
                echo "Activating ${module.ToItem.Name} on ${Entity[${asteroidID}].Name}"
                module:Activate[${asteroidID}]
                activated:Inc
                wait 5  ; Small delay between activations
            }
        }
    }

    echo "Activated ${activated} mining modules"
    return TRUE
}
```

### Automatic Asteroid Depletion Handling

```lavish
; ===== ASTEROID DEPLETION DETECTION =====

variable int64 CurrentMiningTarget = 0

function ManageMiningTarget()
{
    ; Check if current target still exists
    if ${CurrentMiningTarget} > 0
    {
        if !${Entity[${CurrentMiningTarget}](exists)}
        {
            echo "Asteroid ${CurrentMiningTarget} depleted"
            CurrentMiningTarget:Set[0]

            ; Unlock depleted asteroid
            ; (automatically happens when asteroid disappears)
        }
    }

    ; Need new target
    if ${CurrentMiningTarget} == 0
    {
        ; Get best asteroid
        CurrentMiningTarget:Set[${GetBestAsteroidByOreType}]

        if ${CurrentMiningTarget} == 0
        {
            echo "No asteroids available to mine"
            return FALSE
        }

        ; Lock new target
        echo "Locking asteroid: ${Entity[${CurrentMiningTarget}].Name}"
        Entity[${CurrentMiningTarget}]:LockTarget
        wait 20  ; Wait for lock

        if ${Entity[${CurrentMiningTarget}].IsLockedTarget}
        {
            ; Activate miners
            call ActivateMinersOnTarget ${CurrentMiningTarget}
        }
    }

    return TRUE
}
```

---

## Cargo Management

### Cargo Status Check

```lavish
; ===== CARGO STATUS CHECKING =====
; ⚠️ Modern Inventory API (July 2020+)

function ShouldReturnToStation()
{
    ; Open inventory if needed
    if !${EVEWindow[Inventory](exists)}
    {
        EVE:Execute[CmdOpenInventory]
        wait 20
    }

    variable eveinvchildwindow Cargo = EVEWindow[Inventory].Child[ShipCargo]
    variable float cargoFreeSpace = ${Math.Calc[${Cargo.Capacity} - ${Cargo.UsedCapacity}]}
    variable float cargoCapacity = ${Cargo.Capacity}

    ; Cargo 95% full (leave small buffer)
    if ${cargoFreeSpace} < ${Math.Calc[${cargoCapacity} * 0.05]}
    {
        echo "Cargo full (${cargoFreeSpace}m³ free of ${cargoCapacity}m³)"
        return TRUE
    }

    return FALSE
}

function GetCargoPercentFull()
{
    ; Open inventory if needed
    if !${EVEWindow[Inventory](exists)}
    {
        EVE:Execute[CmdOpenInventory]
        wait 20
    }

    variable eveinvchildwindow Cargo = EVEWindow[Inventory].Child[ShipCargo]
    variable float used = ${Cargo.UsedCapacity}
    variable float capacity = ${Cargo.Capacity}

    return ${Math.Calc[(${used} / ${capacity}) * 100]}
}
```

### Approach Asteroid Pattern

```lavish
; ===== APPROACH ASTEROID FOR MINING =====

variable int64 ApproachingAsteroid = 0
variable int ApproachStartTime = 0

function ApproachAsteroidIfNeeded(int64 asteroidID)
{
    variable entity asteroid = ${Entity[${asteroidID}]}

    if !${asteroid(exists)}
        return FALSE

    variable float distance = ${asteroid.Distance}
    variable float optimalRange = ${GetOptimalMiningRange}

    ; Already in range
    if ${distance} <= ${optimalRange}
    {
        ; Stop approaching if was
        if ${ApproachingAsteroid} == ${asteroidID}
        {
            EVE:Execute[CmdStopShip]
            ApproachingAsteroid:Set[0]
        }

        return TRUE
    }

    ; Out of range, approach
    if ${ApproachingAsteroid} != ${asteroidID}
    {
        echo "Approaching ${asteroid.Name} (${distance}m away)"
        asteroid:Approach[${optimalRange}]
        ApproachingAsteroid:Set[${asteroidID}]
        ApproachStartTime:Set[${LavishScript.RunningTime}]
    }
    else
    {
        ; Check approach timeout (60 seconds)
        if ${Math.Calc[${LavishScript.RunningTime} - ${ApproachStartTime}]} > 60000
        {
            echo "Approach timeout, selecting different asteroid"
            ApproachingAsteroid:Set[0]
            return FALSE
        }
    }

    return FALSE  ; Not in range yet
}
```

---

## Station Return and Unloading

### Return to Station

```lavish
; ===== RETURN TO STATION PATTERN =====

variable string HomeStation = "Jita IV - Moon 4 - Caldari Navy Assembly Plant"

function ReturnToStation()
{
    echo "Returning to station"

    ; Stop mining
    EVE:Execute[CmdStopAllModules]
    wait 10

    ; Clear targets
    EVE:Execute[CmdClearTargets]
    wait 10

    ; Find home station
    variable entity station = ${Entity["GroupID = GROUPID_STATION && Name = \"${HomeStation}\""]}

    if !${station(exists)}
    {
        echo "ERROR: Home station not found, docking at nearest"
        station:Set[${Entity["GroupID = GROUPID_STATION && IsNearestStation"]}]
    }

    if !${station(exists)}
    {
        echo "ERROR: No station found in system!"
        return FALSE
    }

    ; Warp to station
    if ${station.Distance} > 150000
    {
        echo "Warping to ${station.Name}"
        station:WarpTo[0]
        wait 100  ; Wait for warp
    }

    ; Dock
    echo "Docking at ${station.Name}"
    station:Dock
    wait 50  ; Wait for docking

    if ${Me.InStation}
    {
        echo "Docked successfully"
        return TRUE
    }

    echo "ERROR: Failed to dock"
    return FALSE
}
```

### Unload Ore to Station Hangar

```lavish
; ===== UNLOAD ORE TO HANGAR =====
; ⚠️ Modern Inventory API (July 2020+)

function UnloadOre()
{
    if !${Me.InStation}
    {
        echo "ERROR: Not in station, cannot unload"
        return FALSE
    }

    echo "Unloading ore to station hangar"

    ; Open inventory window
    if !${EVEWindow[Inventory](exists)}
    {
        EVE:Execute[CmdOpenInventory]
        wait 20
    }

    ; Open item hangar
    EVE:Execute[OpenHangarFloor]
    wait 20

    ; Get cargo items
    variable index:item CargoItems
    EVEWindow[Inventory].Child[ShipCargo]:GetItems[CargoItems]

    variable iterator Item
    CargoItems:GetIterator[Item]

    variable int itemsMoved = 0

    if ${Item:First(exists)}
        do
        {
            ; Check if ore (CategoryID 25) or ice (CategoryID 423)
            if ${Item.Value.CategoryID} == 25 || ${Item.Value.CategoryID} == 423
            {
                echo "Moving ${Item.Value.Quantity} x ${Item.Value.Name}"
                Item.Value:MoveTo[MyStationHangar, ${Item.Value.Quantity}]
                itemsMoved:Inc
                wait 10  ; Wait between moves
            }
        }
        while ${Item:Next(exists)}

    echo "Unloaded ${itemsMoved} item stacks"

    return TRUE
}
```

### Complete Hauling Cycle

```lavish
; ===== COMPLETE HAUL CYCLE =====

function PerformHaulingCycle()
{
    ; Return to station
    if !${ReturnToStation}
    {
        echo "ERROR: Failed to return to station"
        return FALSE
    }

    ; Unload ore
    if !${UnloadOre}
    {
        echo "ERROR: Failed to unload ore"
        return FALSE
    }

    ; Wait a moment
    wait 20

    ; Undock
    echo "Undocking"
    EVE:Execute[CmdExitStation]
    wait 100  ; Wait for undock

    if ${Me.InSpace}
    {
        echo "Undocked successfully"

        ; Return to mining belt
        call WarpToBookmarkedBelt

        return TRUE
    }

    echo "ERROR: Failed to undock"
    return FALSE
}
```

---

## Ice Mining Patterns

### Ice Detection

```lavish
; ===== ICE DETECTION =====

function IsIceBelt()
{
    ; Check for ice entities
    variable index:entity IceAsteroids
    EVE:QueryEntities[IceAsteroids, "CategoryID = 423"]  ; Ice category

    if ${IceAsteroids.Used} > 0
    {
        return TRUE
    }

    return FALSE
}

function GetNearestIce()
{
    variable index:entity IceAsteroids
    variable iterator Ice

    EVE:QueryEntities[IceAsteroids, "CategoryID = 423 && Distance < ${GetOptimalMiningRange}"]
    IceAsteroids:GetIterator[Ice]

    if ${Ice:First(exists)}
    {
        if !${Ice.Value.IsLockedTarget} && !${Ice.Value.BeingTargeted}
        {
            return ${Ice.Value.ID}
        }
    }

    return 0
}
```

### Ice Mining Cycle

```lavish
; ===== ICE MINING CYCLE =====

function MineIce()
{
    ; Get ice target
    variable int64 iceID = ${GetNearestIce}

    if ${iceID} == 0
    {
        echo "No ice found, checking for ice harvesters"

        ; Check if have ice harvesters
        variable bool haveIceHarvesters = FALSE
        variable int i

        for (i:Set[1]; ${i} <= ${MyShip.ModuleCount}; i:Inc)
        {
            if ${MyShip.Module[${i}].ToItem.Group.Equal["Ice Harvester"]}
            {
                haveIceHarvesters:Set[TRUE]
                break
            }
        }

        if !${haveIceHarvesters}
        {
            echo "ERROR: No ice harvesters fitted!"
            return FALSE
        }

        echo "No ice available in range"
        return FALSE
    }

    ; Lock ice
    if !${Entity[${iceID}].IsLockedTarget}
    {
        echo "Locking ice: ${Entity[${iceID}].Name}"
        Entity[${iceID}]:LockTarget
        wait 20
    }

    ; Activate ice harvesters
    variable int i
    for (i:Set[1]; ${i} <= ${MyShip.ModuleCount}; i:Inc)
    {
        variable item module = ${MyShip.Module[${i}]}

        if ${module.ToItem.Group.Equal["Ice Harvester"]}
        {
            if !${module.IsActive} && ${module.IsOnline}
            {
                echo "Activating ${module.ToItem.Name}"
                module:Activate[${iceID}]
                wait 5
            }
        }
    }

    return TRUE
}
```

---

## Orca-Supported Mining

**EVEBot Pattern**: Mine and deliver to Orca

### Orca Detection

```lavish
; ===== ORCA DETECTION =====

variable string OrcaPilotName = "Orca Alt"
variable int64 OrcaEntityID = 0

function FindOrca()
{
    ; Find Orca by pilot name
    variable entity orca = ${Entity["Name = \"${OrcaPilotName}\""]}

    if ${orca(exists)}
    {
        OrcaEntityID:Set[${orca.ID}]
        echo "Found Orca: ${orca.Name} (${orca.Distance}m away)"
        return TRUE
    }

    echo "Orca not found on grid"
    OrcaEntityID:Set[0]
    return FALSE
}
```

### Jetcan to Orca Transfer

```lavish
; ===== ORCA DELIVERY PATTERN =====
; ⚠️ Modern Inventory API (July 2020+)

function DeliverToOrca()
{
    ; Find Orca
    if !${FindOrca}
    {
        echo "ERROR: Orca not on grid"
        return FALSE
    }

    variable entity orca = ${Entity[${OrcaEntityID}]}

    ; Check distance
    if ${orca.Distance} > 2500
    {
        echo "Approaching Orca (${orca.Distance}m away)"
        orca:Approach[2000]
        wait 50  ; Wait to get closer
    }

    ; Open inventory window
    if !${EVEWindow[Inventory](exists)}
    {
        EVE:Execute[CmdOpenInventory]
        wait 20
    }

    ; Open Orca fleet hangar
    echo "Opening Orca fleet hangar"
    orca:OpenCargo

    wait 30  ; Wait for window

    ; Transfer ore to Orca
    variable index:item CargoItems
    EVEWindow[Inventory].Child[ShipCargo]:GetItems[CargoItems]

    variable iterator Item
    CargoItems:GetIterator[Item]

    if ${Item:First(exists)}
        do
        {
            ; Transfer ore/ice
            if ${Item.Value.CategoryID} == 25 || ${Item.Value.CategoryID} == 423
            {
                echo "Transferring ${Item.Value.Quantity} x ${Item.Value.Name} to Orca"

                ; Move to Orca fleet hangar
                Item.Value:MoveTo[OtherCargo, ${Item.Value.Quantity}]
                wait 10
            }
        }
        while ${Item:Next(exists)}

    echo "Delivery to Orca complete"
    return TRUE
}
```

---

## Complete Working Examples

### Example 1: Simple Solo Miner

```lavish
; ===== SIMPLE SOLO MINER =====

variable bool BotRunning = TRUE
variable string CurrentState = "IDLE"
variable int64 MiningTarget = 0

function main()
{
    ; Startup
    while !${ISXEVE.IsReady}
    {
        wait 10
    }

    while !${Me(exists)} || !${MyShip(exists)}
    {
        wait 10
    }

    ; Initialize
    echo "Simple Miner starting"
    call LoadOreValues

    ; Main loop
    while ${BotRunning}
    {
        if ${Me.InSpace}
        {
            call MiningPulse
        }
        elseif ${Me.InStation}
        {
            call StationPulse
        }

        wait ${Math.Rand[5,10]}
        waitframe
    }
}

function MiningPulse()
{
    ; Safety check
    if ${CheckForHostiles}
    {
        call EmergencyDock
        return
    }

    ; Check cargo
    if ${ShouldReturnToStation}
    {
        CurrentState:Set["HAULING"]
        call PerformHaulingCycle
        CurrentState:Set["MINING"]
        return
    }

    ; Mine
    call ManageMiningTarget
}

function StationPulse()
{
    ; ⚠️ Modern Inventory API (July 2020+)
    ; Open inventory to check cargo
    if !${EVEWindow[Inventory](exists)}
    {
        EVE:Execute[CmdOpenInventory]
        wait 20
    }

    ; Unload if have ore
    if ${EVEWindow[Inventory].Child[ShipCargo].UsedCapacity} > 0
    {
        call UnloadOre
    }

    ; Undock
    wait 20
    EVE:Execute[CmdExitStation]
    wait 100
}
```

### Example 2: Advanced Survey Miner

```lavish
; ===== ADVANCED SURVEY MINER =====

variable bool BotRunning = TRUE
variable string MiningMode = "ORE"  ; ORE or ICE
variable int BeltChangeTime = 0

function main()
{
    ; Startup
    while !${ISXEVE.IsReady}
    {
        wait 10
    }

    while !${Me(exists)} || !${MyShip(exists)}
    {
        wait 10
    }

    ; Initialize
    echo "Advanced Survey Miner starting"
    call LoadOreValues
    call InitializeSurveyScanner

    ; Main loop
    while ${BotRunning}
    {
        if ${Me.InSpace}
        {
            call AdvancedMiningPulse
        }
        elseif ${Me.InStation}
        {
            call StationPulse
        }

        wait ${Math.Rand[8,15]}
        waitframe
    }
}

function AdvancedMiningPulse()
{
    ; Safety
    if ${CheckForHostiles}
    {
        call ReturnToStation
        return
    }

    ; Cargo check
    if ${ShouldReturnToStation}
    {
        call PerformHaulingCycle
        return
    }

    ; Survey scan if available
    call ActivateSurveyScanner

    ; Select best surveyed asteroid
    variable int64 bestAsteroid

    if ${SurveyedAsteroids.Used} > 0
    {
        bestAsteroid:Set[${GetBestSurveyedAsteroid}]
    }
    else
    {
        bestAsteroid:Set[${GetBestAsteroidByOreType}]
    }

    if ${bestAsteroid} == 0
    {
        ; No asteroids, change belt every 5 minutes
        if ${Math.Calc[${LavishScript.RunningTime} - ${BeltChangeTime}]} > 300000
        {
            echo "Belt depleted, moving to new belt"
            call WarpToBookmarkedBelt
            BeltChangeTime:Set[${LavishScript.RunningTime}]
        }

        return
    }

    ; Mine best asteroid
    if !${Entity[${bestAsteroid}].IsLockedTarget}
    {
        Entity[${bestAsteroid}]:LockTarget
        wait 20
    }

    call ActivateMinersOnTarget ${bestAsteroid}
}
```

---

## Common Mining Problems

### Problem 1: Miners Won't Activate

```lavish
function DiagnoseMinerFailure(int moduleIndex, int64 asteroidID)
{
    variable item module = ${MyShip.Module[${moduleIndex}]}

    ; Module offline
    if !${module.IsOnline}
    {
        echo "ERROR: Miner offline"
        return "OFFLINE"
    }

    ; Asteroid not locked
    if !${Entity[${asteroidID}].IsLockedTarget}
    {
        echo "ERROR: Asteroid not locked"
        return "NOT_LOCKED"
    }

    ; Out of range
    if ${Entity[${asteroidID}].Distance} > ${GetOptimalMiningRange}
    {
        echo "ERROR: Out of range (${Entity[${asteroidID}].Distance}m)"
        return "OUT_OF_RANGE"
    }

    ; Wrong module type (ice harvester on ore)
    if ${module.ToItem.Group.Equal["Ice Harvester"]} && ${Entity[${asteroidID}].CategoryID} != 423
    {
        echo "ERROR: Ice harvester on ore asteroid"
        return "WRONG_TYPE"
    }

    return "UNKNOWN"
}
```

### Problem 2: Cargo Management Issues

```lavish
; ⚠️ Modern Inventory API (July 2020+)
function DiagnoseCargoIssue()
{
    ; Open inventory window if not open
    if !${EVEWindow[Inventory](exists)}
    {
        echo "Opening inventory window"
        EVE:Execute[CmdOpenInventory]
        wait 20
    }

    ; Check if inventory window exists
    if !${EVEWindow[Inventory](exists)}
    {
        echo "ERROR: Cannot open inventory window"
        return "NO_WINDOW"
    }

    ; Get cargo child window
    variable eveinvchildwindow Cargo = EVEWindow[Inventory].Child[ShipCargo]

    if !${Cargo(exists)}
    {
        echo "ERROR: Cannot access ship cargo"
        return "NO_ACCESS"
    }

    ; Check cargo space
    variable float freeSpace = ${Math.Calc[${Cargo.Capacity} - ${Cargo.UsedCapacity}]}
    if ${freeSpace} <= 0
    {
        echo "Cargo completely full"
        return "FULL"
    }

    ; Check if cargo accessible (has capacity data)
    if ${Cargo.Capacity} <= 0
    {
        echo "WARNING: Cargo capacity data missing, refreshing inventory"
        EVEWindow[Inventory]:Close
        wait 10
        EVE:Execute[CmdOpenInventory]
        wait 20
        return "REFRESHED"
    }

    return "OK"
}
```

---

## Summary

### Key Takeaways

1. **Belt Selection**:
   - Random belts (simple)
   - Bookmarked belts (organized)
   - Ore anomalies (high value)

2. **Asteroid Selection**:
   - Nearest (simple, fast)
   - By ore type (value prioritization)
   - Grouped (efficiency - minimize movement)
   - Survey data (optimal quantity)

3. **Mining Cycle**:
   - Lock asteroid
   - Activate miners
   - Wait for depletion
   - Switch to next asteroid

4. **Cargo Management**:
   - Monitor cargo space
   - Return at 95% full
   - Unload to station hangar

5. **Ice Mining**:
   - Different category (423 vs 25)
   - Requires ice harvesters
   - Same basic pattern

6. **Orca Support**:
   - Detect Orca on grid
   - Transfer to fleet hangar
   - No station trips needed

### Next File

- **File 23**: Hauling_and_Logistics.md (Hauler bots, autopilot, station-to-station transport)

---

**File Complete**: Mining bot patterns fully documented with working examples from EVEBot.
