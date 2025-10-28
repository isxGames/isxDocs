# Mining And Hauling

**Purpose:** Mining and hauling bot patterns for ore extraction and logistics automation
**Audience:** Developers building mining and hauling bots

---

## Table of Contents

### Mining Bot Patterns
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
11. [Complete Working Examples (Mining)](#complete-working-examples)
12. [Common Mining Problems](#common-mining-problems)

### Hauling and Logistics
13. [Hauling and Logistics Bot Patterns](#hauling-and-logistics-bot-patterns)
14. [Introduction to Hauling](#introduction-to-hauling)
15. [Autopilot and Navigation](#autopilot-and-navigation)
16. [Station Operations](#station-operations)
17. [Basic Hauler State Machine](#basic-hauler-state-machine)
18. [Fleet Hauling Patterns](#fleet-hauling-patterns)
19. [Orca Service Patterns](#orca-service-patterns)
20. [Stealth Hauling](#stealth-hauling)
21. [Route Planning and Safety](#route-planning-and-safety)
22. [Cargo Optimization](#cargo-optimization)
23. [Complete Working Examples (Hauling)](#complete-working-examples-1)
24. [Common Problems (Hauling)](#common-problems)

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

## Hauling and Logistics Bot Patterns

> **Note on Code Examples**: This file contains EVEBot patterns that use custom wrapper objects like `Ship` and `Cargo`. These are **NOT** the deprecated `MyShip.Cargo` API. The `Ship` object is EVEBot's abstraction layer that internally uses modern APIs. Direct references to `MyShip:GetCargo` or `MyShip.Cargo` have been updated to use the modern `EVEWindow[Inventory]` API.

---

## Introduction to Hauling

### What is Hauling in EVE?

Hauling is the process of transporting cargo from one location to another. This includes:

- **Mining Support**: Picking up ore from miners and delivering to station
- **Market Hauling**: Moving goods between trade hubs
- **Contract Hauling**: Delivering packages for ISK
- **Fleet Logistics**: Supporting operations with ammunition and supplies
- **Salvage Collection**: Retrieving loot from combat sites

### Hauler Ship Types

```lavish
// Common hauler ship groups
define GROUP_INDUSTRIAL 19
define GROUP_TRANSPORT_SHIP 380        // Deep Space Transport (DST)
define GROUP_BLOCKADE_RUNNER 1202      // Covert Ops hauler
define GROUP_FREIGHTER 513
define GROUP_JUMP_FREIGHTER 902

// Check ship type
if ${MyShip.GroupID} == GROUP_BLOCKADE_RUNNER
{
    echo "Using stealth hauling mode"
}
```

### Key Hauling Concepts

1. **Cargo Capacity**: How much you can carry (m3)
2. **Align Time**: How fast you can warp (critical for safety)
3. **Tank**: Can you survive a gank attempt?
4. **Jump Range**: For jump freighters
5. **Warp Speed**: How fast you travel between gates

---

## Autopilot and Navigation

### Basic Autopilot Object (EVEBot Pattern)

Based on `obj_Autopilot.iss`:

```lavish
objectdef obj_Autopilot
{
    variable int       Destination
    variable int       IsLowSecRoute
    variable index:int Path
    variable iterator  PathIterator

    method Initialize()
    {
        This.IsLowSecRoute:Set[-999]
        Logger:Log["obj_Autopilot: Initialized", LOG_MINOR]
    }

    // Set autopilot destination by system ID
    method SetDestination(int64 id)
    {
        if ${id.Equal[${This.Destination}]}
        {
            return
        }

        Logger:Log["obj_Autopilot: Setting destination to ${Universe[${id}]}"]
        This.Destination:Set[${id}]
        Universe[${id}]:SetDestination
        This.IsLowSecRoute:Set[-999]
    }

    // Check if route goes through low-sec
    member:bool LowSecRoute()
    {
        if ${This.IsLowSecRoute} == -999
        {
            EVE:GetToDestinationPath[This.Path]
            This.Path:GetIterator[This.PathIterator]

            if ${This.PathIterator:First(exists)}
            {
                do
                {
                    if ${This.PathIterator.Value} > 0 &&
                       ${Universe[${This.PathIterator.Value}].Security} <= 0.45
                    {
                        Logger:Log["obj_Autopilot: Low-Sec system found"]
                        This.IsLowSecRoute:Set[TRUE]
                        break
                    }
                }
                while ${This.PathIterator:Next(exists)}
            }
        }

        return ${This.IsLowSecRoute}
    }
}
```

### Travel to System Function

```lavish
function TravelToSystem(int64 solarSystemID)
{
    if ${Me.SolarSystemID} == ${solarSystemID}
    {
        echo "Already in destination system"
        return TRUE
    }

    // Set autopilot destination
    Universe[${solarSystemID}]:SetDestination

    // Get the path
    variable index:int pathToDestination
    EVE:GetToDestinationPath[pathToDestination]

    echo "Route has ${pathToDestination.Used} jumps"

    // Travel each jump
    variable int jumpCount = 1
    while ${pathToDestination.Used} > 0
    {
        if ${Me.InStation}
        {
            call Station.Undock
            wait 50 ${Me.InSpace}
        }

        echo "Jump ${jumpCount}/${Math.Calc[${jumpCount} + ${pathToDestination.Used} - 1]}"

        // Take the next jump
        call This.TravelOneJump

        jumpCount:Inc

        // Recalculate path
        EVE:GetToDestinationPath[pathToDestination]
    }

    echo "Arrived at ${Me.SolarSystem.Name}"
    return TRUE
}
```

### Single Jump Navigation

```lavish
function TravelOneJump()
{
    // Get current system and next system in route
    variable index:int route
    EVE:GetToDestinationPath[route]

    if ${route.Used} == 0
    {
        echo "Already at destination"
        return
    }

    variable int nextSystemID = ${route[1]}
    variable string nextSystemName = "${Universe[${nextSystemID}].Name}"

    echo "Next system: ${nextSystemName}"

    // Find the stargate to that system
    variable index:entity stargates
    EVE:QueryEntities[stargates, "GroupID = GROUP_STARGATE && Name = \"${nextSystemName}\""]

    if ${stargates.Used} == 0
    {
        echo "ERROR: Cannot find stargate to ${nextSystemName}"
        return FALSE
    }

    variable int64 gateID = ${stargates[1].ID}

    // Warp to gate
    echo "Warping to ${stargates[1].Name}"
    stargates[1]:WarpTo[0]

    // Wait for warp
    wait 20
    while ${Me.ToEntity.Mode} == 3
    {
        wait 10
    }

    // Wait until in jump range
    wait 50
    variable int timeout = 0
    while ${Entity[${gateID}].Distance} > 2500 && ${timeout} < 600
    {
        wait 10
        timeout:Inc[10]
    }

    if ${timeout} >= 600
    {
        echo "ERROR: Timeout waiting to reach gate"
        return FALSE
    }

    // Jump
    echo "Jumping to ${nextSystemName}"
    Entity[${gateID}]:Jump

    // Wait for session change
    wait 50

    return TRUE
}
```

### Warp to Bookmark

```lavish
function WarpToBookmark(string bookmarkName)
{
    if !${EVE.Bookmark[${bookmarkName}](exists)}
    {
        echo "ERROR: Bookmark '${bookmarkName}' does not exist"
        return FALSE
    }

    echo "Warping to bookmark: ${bookmarkName}"

    EVE.Bookmark[${bookmarkName}]:WarpTo

    // Wait for warp to start
    wait 30

    // Wait for warp to complete
    while ${Me.ToEntity.Mode} == 3
    {
        wait 10
    }

    echo "Arrived at ${bookmarkName}"
    return TRUE
}
```

---

## Station Operations

### Docking at Station

```lavish
objectdef obj_Station
{
    // Dock at specific station by entity ID
    function DockAtStation(int64 stationID)
    {
        if ${Me.InStation}
        {
            if ${Me.StationID} == ${stationID}
            {
                echo "Already docked at target station"
                return TRUE
            }
            else
            {
                call This.Undock
                wait 50 ${Me.InSpace}
            }
        }

        if !${Entity[${stationID}](exists)}
        {
            echo "ERROR: Station entity ${stationID} does not exist"
            return FALSE
        }

        variable float distance = ${Entity[${stationID}].Distance}

        // If too far, warp to station
        if ${distance} > 200000
        {
            echo "Warping to station (${Math.Calc[${distance}/1000].Int}km away)"
            Entity[${stationID}]:WarpTo[0]
            wait 20
            while ${Me.ToEntity.Mode} == 3
            {
                wait 10
            }
        }

        // Approach if needed
        if ${Entity[${stationID}].Distance} > 2500
        {
            echo "Approaching station"
            Entity[${stationID}]:Approach

            variable int timeout = 0
            while ${Entity[${stationID}].Distance} > 2500 && ${timeout} < 600
            {
                wait 10
                timeout:Inc[10]
            }
        }

        // Dock
        echo "Docking at ${Entity[${stationID}].Name}"
        Entity[${stationID}]:Dock

        // Wait for docking
        wait 100
        variable int dockTimeout = 0
        while !${Me.InStation} && ${dockTimeout} < 600
        {
            wait 10
            dockTimeout:Inc[10]
        }

        if ${Me.InStation}
        {
            echo "Docked successfully"
            return TRUE
        }
        else
        {
            echo "ERROR: Docking failed"
            return FALSE
        }
    }

    // Undock from current station
    function Undock()
    {
        if !${Me.InStation}
        {
            echo "Not in station"
            return TRUE
        }

        echo "Undocking from ${Me.Station.Name}"

        EVE:Execute[CmdExitStation]

        wait 50
        variable int timeout = 0
        while ${Me.InStation} && ${timeout} < 300
        {
            wait 10
            timeout:Inc[10]
        }

        if ${Me.InSpace}
        {
            echo "Undocked successfully"
            return TRUE
        }
        else
        {
            echo "ERROR: Undocking failed"
            return FALSE
        }
    }
}
```

### Cargo Transfer to Station Hangar

Based on EVEBot `obj_Cargo.iss`:

```lavish
function TransferCargoToStationHangar()
{
    if !${Me.InStation}
    {
        echo "ERROR: Must be in station to transfer cargo"
        return FALSE
    }

    echo "Transferring cargo to station hangar"

    // Open ship cargo
    call Inventory.ShipCargo.Activate
    wait 20

    // Get all items in cargo
    variable index:item cargoItems
    Inventory.ShipCargo:GetItems[cargoItems]

    if ${cargoItems.Used} == 0
    {
        echo "No cargo to transfer"
        return TRUE
    }

    echo "Moving ${cargoItems.Used} item stacks to hangar"

    // Build list of item IDs
    variable index:int64 itemIDs
    variable iterator cargoIterator
    cargoItems:GetIterator[cargoIterator]

    if ${cargoIterator:First(exists)}
    {
        do
        {
            itemIDs:Insert[${cargoIterator.Value.ID}]
        }
        while ${cargoIterator:Next(exists)}
    }

    // Move all items to station hangar
    EVE:MoveItemsTo[itemIDs, ${Me.Station.ID}, Hangar]

    wait 30

    echo "Cargo transfer complete"
    return TRUE
}
```

### Cargo Transfer from Station to Ship

```lavish
function TransferHangarItemsToShip(string itemName, int quantity)
{
    if !${Me.InStation}
    {
        echo "ERROR: Must be in station"
        return FALSE
    }

    // Open station hangar
    call Inventory.StationHangar.Activate ${Me.Station.ID}
    wait 20

    // Find the item
    variable index:item hangarItems
    Inventory.StationHangar:GetItems[hangarItems, "Name = \"${itemName}\""]

    if ${hangarItems.Used} == 0
    {
        echo "ERROR: Item '${itemName}' not found in hangar"
        return FALSE
    }

    echo "Moving ${quantity} x ${itemName} to ship cargo"

    // Move to ship
    hangarItems[1]:MoveTo[MyShip, CargoHold, ${quantity}]

    wait 20

    return TRUE
}
```

---

## Basic Hauler State Machine

### Hauler States

Based on EVEBot `obj_Hauler.iss`:

```lavish
// Hauler states:
// IDLE        - Waiting in station (nothing to do)
// HARDSTOP    - Emergency dock and stay docked
// FLEE        - Danger detected, get to safety
// BASE        - In station, unload cargo
// HAUL        - In space, collecting cargo
// DROPOFF     - Cargo full, returning to station
```

### Simple Hauler Object

```lavish
objectdef obj_SimpleHauler
{
    variable string CurrentState = "IDLE"
    variable int PulseInterval = 2
    variable time NextPulse

    variable string DeliveryStation = "Jita IV - Moon 4 - Caldari Navy Assembly Plant"
    variable float CargoThreshold = 0.9    // Return when 90% full

    method Initialize()
    {
        echo "SimpleHauler initialized"
        Event[EVENT_ONFRAME]:AttachAtom[This:Pulse]
    }

    method Pulse()
    {
        if ${Time.Timestamp} >= ${This.NextPulse.Timestamp}
        {
            This:SetState

            This.NextPulse:Set[${Time.Timestamp}]
            This.NextPulse.Second:Inc[${This.PulseInterval}]
            This.NextPulse:Update
        }
    }

    method SetState()
    {
        // SAFETY CHECKS FIRST

        // If in station and hard stop requested, stay idle
        if ${Me.InStation}
        {
            if ${EVEBot.ReturnToStation}
            {
                This.CurrentState:Set["IDLE"]
                return
            }
        }
        else
        {
            // In space

            // Hard stop requested - get to station
            if ${EVEBot.ReturnToStation}
            {
                This.CurrentState:Set["HARDSTOP"]
                return
            }

            // Check for hostiles
            if ${Social.PossibleHostiles}
            {
                This.CurrentState:Set["HARDSTOP"]
                echo "HARD STOP: Hostiles detected"
                EVEBot.ReturnToStation:Set[TRUE]
                relay all -event EVEBot_HARDSTOP "${Me.Name} - Hauler (Hostiles)"
                return
            }

            // Check if podded
            if ${Ship.IsPod}
            {
                This.CurrentState:Set["HARDSTOP"]
                echo "HARD STOP: I'm in a pod!"
                EVEBot.ReturnToStation:Set[TRUE]
                return
            }
        }

        // Check for soft flee conditions
        if !${Social.IsSafe} && !${EVEBot.ReturnToStation}
        {
            if !${Me.InStation}
            {
                This.CurrentState:Set["FLEE"]
                echo "FLEE: Unsafe player detected"
            }
            else
            {
                This.CurrentState:Set["IDLE"]
            }
            return
        }

        // NORMAL OPERATION

        // If in station, unload
        if ${Me.InStation}
        {
            This.CurrentState:Set["BASE"]
            return
        }

        // If cargo full, return to station
        if ${This.HaulerFull}
        {
            This.CurrentState:Set["DROPOFF"]
            return
        }

        // Otherwise, haul cargo
        This.CurrentState:Set["HAUL"]
    }

    // Check if cargo is full enough to return
    member:bool HaulerFull()
    {
        variable float cargoUsed = ${Ship.Cargo.UsedCapacity}
        variable float cargoMax = ${Ship.Cargo.Capacity}

        if ${cargoMax} == 0
            return TRUE

        variable float cargoPercent = ${Math.Calc[${cargoUsed} / ${cargoMax}]}

        return ${Math.Calc[${cargoPercent} >= ${This.CargoThreshold}]}
    }

    function ProcessState()
    {
        switch ${This.CurrentState}
        {
            case IDLE
                // Just wait
                break

            case HARDSTOP
                // Emergency dock
                call This.EmergencyDock
                break

            case FLEE
                // Flee to safety
                call This.FleeToSafety
                break

            case BASE
                // Unload cargo at station
                call This.UnloadCargo
                break

            case HAUL
                // Pick up cargo (overridden by subclasses)
                break

            case DROPOFF
                // Return to delivery station
                call This.ReturnToStation
                break
        }
    }

    function EmergencyDock()
    {
        if ${Me.InStation}
        {
            return
        }

        // Try panic bookmark first
        if ${EVE.Bookmark["${Config.Miner.PanicLocation}"](exists)}
        {
            Navigator:FlyToBookmark["${Config.Miner.PanicLocation}", 0, TRUE]
            while ${Navigator.Busy}
            {
                wait 10
            }
            return
        }

        // Try delivery station
        if ${EVE.Bookmark["${This.DeliveryStation}"](exists)}
        {
            Navigator:FlyToBookmark["${This.DeliveryStation}", 0, TRUE]
            while ${Navigator.Busy}
            {
                wait 10
            }
            return
        }

        // Any station in system
        variable index:entity stations
        EVE:QueryEntities[stations, "GroupID = 15 || GroupID = 1657"]

        if ${stations.Used} > 0
        {
            echo "Docking at ${stations[1].Name}"
            call Station.DockAtStation ${stations[1].ID}
            return
        }

        // No stations - warp to safe spot
        call Safespots.WarpTo
        wait 30
    }

    function FleeToSafety()
    {
        // Same as EmergencyDock but will resume after threat passes
        call This.EmergencyDock
    }

    function UnloadCargo()
    {
        if !${Me.InStation}
        {
            return
        }

        // Transfer cargo to hangar
        call Cargo.TransferCargoToStationHangar

        // Undock
        call Station.Undock
        wait 20 ${Me.InSpace}
    }

    function ReturnToStation()
    {
        // Navigate to delivery station
        if !${EVE.Bookmark["${This.DeliveryStation}"](exists)}
        {
            echo "ERROR: Delivery station bookmark not found"
            return
        }

        Navigator:FlyToBookmark["${This.DeliveryStation}", 0, TRUE]
        while ${Navigator.Busy}
        {
            wait 10
        }
    }
}
```

---

## Fleet Hauling Patterns

### Hauling for Fleet Members

Pick up jetcans from miners in your fleet.

```lavish
objectdef obj_FleetHauler inherits obj_SimpleHauler
{
    variable queue:entity JetCans
    variable int64 ApproachingID = 0
    variable int TimeStartedApproaching = 0

    // Override the HAUL state
    function ProcessState()
    {
        if ${This.CurrentState.Equal["HAUL"]}
        {
            call This.HaulForFleet
        }
        else
        {
            // Use parent implementation for other states
            call obj_SimpleHauler.ProcessState
        }
    }

    function HaulForFleet()
    {
        // Build list of fleet members
        variable queue:fleetmember fleetMembers
        variable iterator memberIterator

        Me.Fleet:GetMembers[fleetMembers]
        fleetMembers:GetIterator[memberIterator]

        if ${memberIterator:First(exists)}
        {
            do
            {
                // Skip self
                if ${memberIterator.Value.CharID} == ${Me.CharID}
                    continue

                // Check if in local
                if ${Local[${memberIterator.Value.Name}](exists)}
                {
                    call This.WarpToFleetMemberAndLoot ${memberIterator.Value.CharID}
                }
            }
            while ${memberIterator:Next(exists)}
        }

        // If nothing to do, wait at safe spot
        if ${This.JetCans.Used} == 0
        {
            call Safespots.WarpTo
        }
    }

    function WarpToFleetMemberAndLoot(int64 memberCharID)
    {
        // Get character from CharID
        variable string memberName

        variable queue:fleetmember allMembers
        Me.Fleet:GetMembers[allMembers]

        variable iterator it
        allMembers:GetIterator[it]
        if ${it:First(exists)}
        {
            do
            {
                if ${it.Value.CharID} == ${memberCharID}
                {
                    memberName:Set["${it.Value.Name}"]
                    break
                }
            }
            while ${it:Next(exists)}
        }

        if !${Local[${memberName}](exists)}
        {
            echo "Fleet member ${memberName} not in local"
            return
        }

        // Check if on grid
        if !${Local[${memberName}].ToEntity.ID(exists)}
        {
            echo "Warping to ${memberName}"
            Local[${memberName}].ToFleetMember:WarpTo
            wait 30
            while ${Me.ToEntity.Mode} == 3
            {
                wait 10
            }
            return
        }

        // On grid - build jetcan list and loot
        call This.BuildJetCanList ${Local[${memberName}].ToEntity.ID}
        call This.LootJetCans
    }

    function BuildJetCanList(int64 nearEntityID)
    {
        This.JetCans:Clear

        // Find all jetcans near the entity
        variable index:entity cans
        EVE:QueryEntities[cans, "GroupID = GROUP_CARGO_CONTAINER && Distance < 150000"]

        if ${cans.Used} == 0
            return

        variable iterator canIterator
        cans:GetIterator[canIterator]

        if ${canIterator:First(exists)}
        {
            do
            {
                // Check distance from target entity
                if ${canIterator.Value.DistanceTo[${nearEntityID}]} < 100000
                {
                    This.JetCans:Queue[${canIterator.Value}]
                }
            }
            while ${canIterator:Next(exists)}
        }

        echo "Found ${This.JetCans.Used} jetcans to loot"
    }

    function LootJetCans()
    {
        while ${This.JetCans.Peek(exists)}
        {
            variable entity can = ${This.JetCans.Peek}

            // Check if still exists
            if !${can(exists)}
            {
                This.JetCans:Dequeue
                continue
            }

            echo "Looting ${can.Name} at ${Math.Calc[${can.Distance}/1000].Int}km"

            // If far away, use tractor beam
            if ${can.Distance} >= 5000
            {
                call This.TractorAndLoot ${can.ID}
            }
            // If moderate distance, approach
            elseif ${can.Distance} >= LOOT_RANGE
            {
                call Ship.Approach ${can.ID} LOOT_RANGE
                wait 50
            }

            // Wait until in loot range
            variable int timeout = 0
            while ${Entity[${can.ID}].Distance} > LOOT_RANGE && ${timeout} < 400
            {
                wait 10
                timeout:Inc[10]
            }

            // Stop ship
            if ${Me.ToEntity.Velocity} > 10
            {
                EVE:Execute[CmdStopShip]
                wait 20
            }

            // Loot the can
            call This.LootEntity ${can.ID}

            // Remove from queue
            This.JetCans:Dequeue

            // Check if cargo full
            if ${This.HaulerFull}
            {
                echo "Cargo full, stopping loot cycle"
                break
            }
        }
    }

    function TractorAndLoot(int64 canID)
    {
        if !${Entity[${canID}](exists)}
            return

        variable float tractorRange = ${Ship.OptimalTractorRange}
        variable float targetRange = ${Ship.OptimalTargetingRange}

        if ${tractorRange} == 0
        {
            // No tractor, just approach
            call Ship.Approach ${canID} LOOT_RANGE
            return
        }

        variable float approachRange = ${tractorRange}
        if ${tractorRange} > ${targetRange}
        {
            approachRange:Set[${targetRange}]
        }

        // Approach into tractor range
        if ${Entity[${canID}].Distance} > ${approachRange}
        {
            call Ship.Approach ${canID} ${approachRange}
            wait 30
        }

        // Lock target
        Entity[${canID}]:LockTarget
        wait 10 ${Entity[${canID}].BeingTargeted} || ${Entity[${canID}].IsLockedTarget}

        variable int timeout = 0
        while !${Entity[${canID}].IsLockedTarget} && ${timeout} < 300
        {
            wait 10
            timeout:Inc[10]
        }

        if !${Entity[${canID}].IsLockedTarget}
        {
            echo "Failed to lock ${Entity[${canID}].Name}"
            return
        }

        // Make active target
        Entity[${canID}]:MakeActiveTarget
        wait 10

        // Activate tractor beams
        Ship:Activate_Tractor

        // Wait for can to get close
        timeout:Set[0]
        while ${Entity[${canID}].Distance} > LOOT_RANGE && ${timeout} < 600
        {
            wait 10
            timeout:Inc[10]
        }

        // Deactivate tractor
        Ship:Deactivate_Tractor
        wait 10
    }

    function LootEntity(int64 entityID)
    {
        if !${Entity[${entityID}](exists)}
            return

        echo "Opening cargo of ${Entity[${entityID}].Name}"

        Entity[${entityID}]:Open
        wait 30

        // MODERN API (July 2020+): Access container cargo via inventory window
        variable index:item containerItems
        if !${EVEWindow[ByItemID,${entityID}](exists)}
        {
            echo "WARNING: Container window not found for ${entityID}"
            return
        }

        EVEWindow[ByItemID,${entityID}]:GetItems[containerItems]

        if ${containerItems.Used} == 0
        {
            echo "Container is empty"
            Entity[${entityID}]:Close
            return
        }

        echo "Transferring ${containerItems.Used} item stacks"

        // Build item ID list
        variable index:int64 itemIDs
        variable iterator itemIterator
        containerItems:GetIterator[itemIterator]

        if ${itemIterator:First(exists)}
        {
            do
            {
                itemIDs:Insert[${itemIterator.Value.ID}]
            }
            while ${itemIterator:Next(exists)}
        }

        // Move to ship cargo
        EVE:MoveItemsTo[itemIDs, MyShip, CargoHold]
        wait 30

        Entity[${entityID}]:Close

        echo "Loot complete"
    }
}
```

### Listen for Miner Full Events

EVEBot uses relay events to notify haulers:

```lavish
method Initialize()
{
    Event[EVENT_ONFRAME]:AttachAtom[This:Pulse]

    // Register for miner events
    LavishScript:RegisterEvent[EVEBot_Miner_Full]
    Event[EVEBot_Miner_Full]:AttachAtom[This:OnMinerFull]
}

atom OnMinerFull(int64 fleetMemberID, int64 solarSystemID, int64 canID)
{
    echo "Miner reports full: FleetMember=${fleetMemberID}, System=${solarSystemID}, Can=${canID}"

    // Add to pickup queue
    This.FullMiners:Set[${canID}, ${fleetMemberID}, ${solarSystemID}, ${canID}]
}

// Miner sends event when full
if ${Ship.Cargo.PercentFull} > 95
{
    relay all -event EVEBot_Miner_Full ${Me.CharID} ${Me.SolarSystemID} ${MyCan.ID}
}
```

---

## Orca Service Patterns

### Delivering Ore to Orca

Miners deliver ore to an Orca's fleet hangar instead of station:

```lavish
objectdef obj_OrcaDelivery
{
    variable string OrcaPilotName = "MyOrca"
    variable int64 OrcaEntityID = 0

    function FindOrca()
    {
        // Try to find Orca by pilot name
        variable index:entity orcas
        EVE:QueryEntities[orcas, "Name = \"${This.OrcaPilotName}\" && CategoryID = CATEGORYID_SHIP"]

        if ${orcas.Used} > 0
        {
            This.OrcaEntityID:Set[${orcas[1].ID}]
            echo "Found Orca: ${orcas[1].Name} at ${Math.Calc[${orcas[1].Distance}/1000].Int}km"
            return TRUE
        }

        echo "Orca not found on grid"
        return FALSE
    }

    function DeliverToOrca()
    {
        if !${This.FindOrca}
        {
            echo "Cannot deliver - Orca not found"
            return FALSE
        }

        variable entity orca = ${Entity[${This.OrcaEntityID}]}

        // Approach Orca if needed
        if ${orca.Distance} > 2500
        {
            echo "Approaching Orca"
            orca:Approach

            variable int timeout = 0
            while ${Entity[${This.OrcaEntityID}].Distance} > 2500 && ${timeout} < 600
            {
                wait 10
                timeout:Inc[10]
            }

            EVE:Execute[CmdStopShip]
            wait 20
        }

        // Open Orca cargo (fleet hangar)
        echo "Opening Orca fleet hangar"
        orca:OpenCargo
        wait 30

        // Get ore from ship
        call Inventory.ShipGeneralMiningHold.Activate
        wait 20

        variable index:item oreItems
        Inventory.ShipGeneralMiningHold:GetItems[oreItems]

        if ${oreItems.Used} == 0
        {
            echo "No ore to transfer"
            return TRUE
        }

        echo "Transferring ${oreItems.Used} ore stacks to Orca"

        // Transfer each item
        variable iterator oreIterator
        oreItems:GetIterator[oreIterator]

        if ${oreIterator:First(exists)}
        {
            do
            {
                // Move to Orca's fleet hangar (OtherCargo)
                oreIterator.Value:MoveTo[OtherCargo, ${oreIterator.Value.Quantity}]
                wait 10
            }
            while ${oreIterator:Next(exists)}
        }

        wait 20

        echo "Delivery to Orca complete"
        return TRUE
    }
}

// Usage in miner:
if ${Config.Miner.DeliverToOrca}
{
    call OrcaDelivery.DeliverToOrca
}
else
{
    call This.ReturnToStation
}
```

### Orca Hauler Pattern

Orca hauls ore from fleet hangar to station:

```lavish
objectdef obj_OrcaHauler
{
    variable float CargoThreshold = 35000    // Return when fleet hangar has 35k m3

    function ServiceOrca()
    {
        // Wait until Orca has enough cargo to justify hauling
        if ${This.OrcaCargo} < ${This.CargoThreshold}
        {
            echo "Orca cargo: ${This.OrcaCargo.Int}m3 (threshold: ${This.CargoThreshold.Int}m3)"
            return
        }

        echo "Orca is ready for pickup (${This.OrcaCargo.Int}m3)"

        // Find Orca
        if !${This.FindOrca}
        {
            echo "Cannot find Orca"
            return
        }

        // Warp to Orca if needed
        if !${Entity["${Config.Hauler.OrcaPilotName}"](exists)}
        {
            echo "Warping to Orca"
            Local["${Config.Hauler.OrcaPilotName}"].ToFleetMember:WarpTo
            wait 30
            while ${Me.ToEntity.Mode} == 3
            {
                wait 10
            }
        }

        // Approach Orca
        call This.ApproachOrca

        // Open Orca's fleet hangar
        Entity["${Config.Hauler.OrcaPilotName}"]:OpenCargo
        wait 30

        // Transfer from Orca to hauler
        call This.TakeOreFromOrca

        // Close Orca cargo
        EVE:Execute[CloseInventory]
    }

    function TakeOreFromOrca()
    {
        // OtherCargo = Fleet Hangar of target ship
        variable index:item orcaCargo

        // This gets items from the open fleet hangar window
        variable index:eveinvchildwindow childWindows
        EVEWindow[Inventory]:GetChildWindows[childWindows]

        variable iterator windowIterator
        childWindows:GetIterator[windowIterator]

        // Find the fleet hangar window
        if ${windowIterator:First(exists)}
        {
            do
            {
                if ${windowIterator.Value.Name.Find["Fleet Hangar"]}
                {
                    windowIterator.Value:GetItems[orcaCargo]
                    break
                }
            }
            while ${windowIterator:Next(exists)}
        }

        if ${orcaCargo.Used} == 0
        {
            echo "No ore in Orca fleet hangar"
            return
        }

        echo "Taking ${orcaCargo.Used} ore stacks from Orca"

        // Build item ID list
        variable index:int64 itemIDs
        variable iterator oreIterator
        orcaCargo:GetIterator[oreIterator]

        if ${oreIterator:First(exists)}
        {
            do
            {
                itemIDs:Insert[${oreIterator.Value.ID}]
            }
            while ${oreIterator:Next(exists)}
        }

        // Move to hauler cargo
        EVE:MoveItemsTo[itemIDs, MyShip, CargoHold]
        wait 30

        echo "Ore pickup complete"
    }

    // Listen for Orca cargo updates
    atom OnOrcaCargoUpdate(float cargoAmount)
    {
        This.OrcaCargo:Set[${cargoAmount}]
        echo "Orca cargo updated: ${This.OrcaCargo.Int}m3"
    }
}

// Orca reports its cargo level
method ReportCargoLevel()
{
    variable float fleetHangarUsed = ${Ship.FleetHangar.UsedCapacity}
    relay all -event EVEBot_Orca_Cargo ${fleetHangarUsed}
}
```

---

## Stealth Hauling

### Blockade Runner / Covert Ops Hauling

Based on EVEBot `obj_StealthHauler.iss`:

```lavish
objectdef obj_StealthHauler
{
    variable index:int RouteToDestination
    variable iterator  RouteIterator

    function ProcessState()
    {
        if ${Station.Docked}
        {
            call Station.Undock
            return
        }

        if !${Ship.HasCovOpsCloak}
        {
            echo "ERROR: This ship does not have a Covert Ops cloak!"
            return FALSE
        }

        // Get route
        if ${This.RouteToDestination.Used} == 0
        {
            EVE:GetToDestinationPath[This.RouteToDestination]
            This.RouteToDestination:GetIterator[This.RouteIterator]
            This.RouteIterator:First

            if ${This.RouteToDestination.Used} == 0
            {
                // At destination - cloak and move
                call This.CloakAndMove
                return
            }
        }

        // Take next jump
        if ${This.RouteIterator.Value(exists)}
        {
            call This.StealthJump ${This.RouteIterator.Value}
            This.RouteIterator:Next

            // Recalculate route
            This.RouteToDestination:Clear
        }
    }

    function StealthJump(int nextSystemID)
    {
        variable string nextSystemName = "${Universe[${nextSystemID}].Name}"

        echo "Stealth jumping to: ${nextSystemName}"

        // Find stargate
        variable index:entity stargates
        EVE:QueryEntities[stargates, "GroupID = GROUP_STARGATE && Name = \"${nextSystemName}\""]

        if ${stargates.Used} == 0
        {
            echo "ERROR: Cannot find stargate to ${nextSystemName}"
            return FALSE
        }

        variable int64 gateID = ${stargates[1].ID}

        // STEALTH PROTOCOL:

        // 1. Set speed to near-max (adds randomization to avoid detection)
        variable float randomSpeed = ${Math.Calc[90 + ${Math.Rand[9]} + ${Math.Calc[0.10 * ${Math.Rand[9]}]}]}
        echo "Setting speed to ${randomSpeed.Int}%"
        Me:SetVelocity[${randomSpeed}]

        // 2. Activate afterburner
        Ship:Activate_AfterBurner
        wait 5

        // 3. Activate cloak
        echo "Activating cloak"
        do
        {
            Ship:Activate_Cloak
            wait 10
        }
        while !${Me.ToEntity.IsCloaked}

        // 4. Wait if warp scrambled
        if ${Me.ToEntity.IsWarpScrambled}
        {
            echo "WARNING: Warp scrambled! Waiting..."
            do
            {
                wait 10
            }
            while ${Me.ToEntity.IsWarpScrambled}
        }

        // 5. Warp to gate (while cloaked)
        echo "Warping to gate (cloaked)"
        Navigator:FlyToEntityID[${gateID}, 0]
        while ${Navigator.Busy}
        {
            wait 10
        }

        // 6. Decloak before jumping
        echo "Decloaking for jump"
        Ship:Deactivate_Cloak
        do
        {
            wait 10
        }
        while ${Me.ToEntity.IsCloaked}

        wait 5

        // 7. Jump
        echo "Jumping to ${nextSystemName}"
        Entity[${gateID}]:Jump
        wait 50

        // 8. After jump, immediately cloak again
        call This.CloakAndMove

        return TRUE
    }

    function CloakAndMove()
    {
        // Set random speed
        variable float randomSpeed = ${Math.Calc[90 + ${Math.Rand[9]} + ${Math.Calc[0.10 * ${Math.Rand[9]}]}]}
        Me:SetVelocity[${randomSpeed}]
        wait 5

        // Activate afterburner
        Ship:Activate_AfterBurner
        wait 5

        // Cloak
        Ship:Activate_Cloak

        echo "Cloaked and moving"
    }
}
```

### Cloak-Warp Trick

For travel in hostile space:

```lavish
function CloakWarpTrick()
{
    // After jumping through gate:
    // 1. Don't move - gate cloak protects you
    // 2. Create a bookmark 150km+ in random direction
    // 3. Align to bookmark
    // 4. Let gate cloak drop
    // 5. IMMEDIATELY activate covert ops cloak (before getting locked)
    // 6. Warp to bookmark (can warp while cloaked with covops)
    // 7. From bookmark, warp to next gate

    echo "Using cloak-warp trick"

    // Wait for jump session change
    wait 50

    // Create safe spot bookmark
    variable int randomDirection = ${Math.Rand[360]}
    variable int randomDistance = ${Math.Calc[150000 + ${Math.Rand[50000]}]}

    // Align to random direction
    echo "Aligning to safe direction"
    // (In practice, create bookmark manually or use existing safe spot)

    // Wait for gate cloak to drop (60 seconds)
    variable int countdown = 55
    while ${countdown} > 0
    {
        echo "Gate cloak: ${countdown}s"
        wait 10
        countdown:Dec[10]
    }

    echo "Gate cloak dropping - activating covops cloak NOW"

    // Spam cloak activation
    Ship:Activate_Cloak
    wait 5
    Ship:Activate_Cloak
    wait 5
    Ship:Activate_Cloak

    // Verify cloaked
    if !${Me.ToEntity.IsCloaked}
    {
        echo "ERROR: Failed to cloak! EMERGENCY WARP"
        call Safespots.WarpTo
        return FALSE
    }

    echo "Successfully cloaked"

    // Now warp to safe spot (cloaked warp is safe)
    call Safespots.WarpTo

    return TRUE
}
```

---

## Route Planning and Safety

### Calculate Route Value

Determine if a route is safe:

```lavish
function AnalyzeRoute(int64 destinationSystemID)
{
    variable index:int route
    EVE:GetToDestinationPath[route]

    echo "Analyzing route to ${Universe[${destinationSystemID}].Name}"
    echo "Jumps: ${route.Used}"

    variable int lowSecJumps = 0
    variable int nullSecJumps = 0
    variable float minSecurity = 1.0

    variable iterator it
    route:GetIterator[it]

    if ${it:First(exists)}
    {
        do
        {
            variable float security = ${Universe[${it.Value}].Security}

            echo "  ${Universe[${it.Value}].Name} (${security.Precision[2]})"

            if ${security} < ${minSecurity}
            {
                minSecurity:Set[${security}]
            }

            if ${security} <= 0.45 && ${security} > 0
            {
                lowSecJumps:Inc
            }
            elseif ${security} <= 0
            {
                nullSecJumps:Inc
            }
        }
        while ${it:Next(exists)}
    }

    echo "Route analysis:"
    echo "  Low-sec jumps: ${lowSecJumps}"
    echo "  Null-sec jumps: ${nullSecJumps}"
    echo "  Minimum security: ${minSecurity.Precision[2]}"

    if ${nullSecJumps} > 0
    {
        echo "WARNING: Route includes NULL-SEC - Use covops hauler or find alternate route"
        return FALSE
    }

    if ${lowSecJumps} > 0
    {
        echo "WARNING: Route includes LOW-SEC - Use tanked hauler or travel in fast ship"
        return FALSE
    }

    echo "Route is HIGH-SEC - Safe for normal haulers"
    return TRUE
}
```

### Set Waypoint Preferences

```lavish
// Prefer shorter routes (may go through low-sec)
EVE:SetWaypointPreference[Shorter]

// Prefer safer routes (stay in high-sec even if longer)
EVE:SetWaypointPreference[Safer]

// Example: Always use safer routes for valuable cargo
if ${Ship.Cargo.Value} > 1000000000  // 1 billion ISK
{
    echo "Valuable cargo - using safer route"
    EVE:SetWaypointPreference[Safer]
}
else
{
    EVE:SetWaypointPreference[Shorter]
}
```

### Gank Avoidance

```lavish
function CheckGankRisk()
{
    // Common gank systems (Uedama, Niarja, etc.)
    variable collection:string gankSystems
    gankSystems:Set["Uedama", TRUE]
    gankSystems:Set["Niarja", TRUE]
    gankSystems:Set["Sivala", TRUE]

    if ${gankSystems.Element["${Me.SolarSystem.Name}"](exists)}
    {
        echo "WARNING: You are in a known gank system!"

        // Check local count
        if ${Local.Count} > 20
        {
            echo "DANGER: High local count (${Local.Count}) - possible gank fleet"
            return TRUE
        }
    }

    // Check ship value vs tank
    variable float cargoValue = ${Ship.Cargo.Value}
    variable float ehp = ${Ship.GetEHP}

    // Rule of thumb: If cargo value / EHP > 10000, you're at risk
    variable float riskFactor = ${Math.Calc[${cargoValue} / ${ehp}]}

    if ${riskFactor} > 10000
    {
        echo "WARNING: High gank risk factor: ${riskFactor.Int}"
        echo "  Cargo value: ${Math.Calc[${cargoValue} / 1000000].Int}M ISK"
        echo "  EHP: ${ehp.Int}"
        return TRUE
    }

    return FALSE
}

// Usage
if ${CheckGankRisk}
{
    echo "Gank risk detected - recommend aborting or using alternate route"
}
```

---

## Cargo Optimization

### Calculate Cargo Value

```lavish
; ⚠️ Modern Inventory API (July 2020+)
member:float CargoValue()
{
    ; Open inventory if needed
    if !${EVEWindow[Inventory](exists)}
    {
        EVE:Execute[CmdOpenInventory]
        wait 20
    }

    variable index:item cargoItems
    EVEWindow[Inventory].Child[ShipCargo]:GetItems[cargoItems]

    variable float totalValue = 0
    variable iterator itemIterator
    cargoItems:GetIterator[itemIterator]

    if ${itemIterator:First(exists)}
    {
        do
        {
            variable float itemValue = ${Math.Calc[${itemIterator.Value.Quantity} * ${itemIterator.Value.BasePrice}]}
            totalValue:Set[${Math.Calc[${totalValue} + ${itemValue}]}]
        }
        while ${itemIterator:Next(exists)}
    }

    return ${totalValue}
}

// Usage
echo "Cargo value: ${Math.Calc[${This.CargoValue} / 1000000].Int}M ISK"
```

### Prioritize Valuable Items

When cargo is limited, prioritize high-value items:

```lavish
function LoadCargoByValue()
{
    // Get all items from station hangar
    call Inventory.StationHangar.Activate ${Me.Station.ID}
    wait 20

    variable index:item hangarItems
    Inventory.StationHangar:GetItems[hangarItems]

    // Calculate value per m3 for each item
    variable collection:float itemValuePerM3

    variable iterator itemIterator
    hangarItems:GetIterator[itemIterator]

    if ${itemIterator:First(exists)}
    {
        do
        {
            variable float value = ${Math.Calc[${itemIterator.Value.Quantity} * ${itemIterator.Value.BasePrice}]}
            variable float volume = ${Math.Calc[${itemIterator.Value.Quantity} * ${itemIterator.Value.Volume}]}

            if ${volume} > 0
            {
                variable float valuePerM3 = ${Math.Calc[${value} / ${volume}]}
                itemValuePerM3:Set[${itemIterator.Value.ID}, ${valuePerM3}]
            }
        }
        while ${itemIterator:Next(exists)}
    }

    // Sort by value per m3 (descending)
    // (LavishScript doesn't have built-in sort, so implement priority loading manually)

    // Load highest value items first
    variable float cargoFree = ${Ship.Cargo.FreeCapacity}

    // Simple greedy algorithm: take items in order until cargo full
    hangarItems:GetIterator[itemIterator]
    if ${itemIterator:First(exists)}
    {
        do
        {
            variable float itemVolume = ${Math.Calc[${itemIterator.Value.Quantity} * ${itemIterator.Value.Volume}]}

            if ${itemVolume} <= ${cargoFree}
            {
                echo "Loading: ${itemIterator.Value.Name} (${itemVolume.Int}m3, ${Math.Calc[${itemValuePerM3.Element[${itemIterator.Value.ID}]}].Int} ISK/m3)"

                itemIterator.Value:MoveTo[MyShip, CargoHold]
                wait 10

                cargoFree:Set[${Math.Calc[${cargoFree} - ${itemVolume}]}]
            }
        }
        while ${itemIterator:Next(exists)}
    }

    echo "Cargo loaded: ${Math.Calc[${Ship.Cargo.PercentFull}].Int}% full"
}
```

### Calculate Optimal Load

```lavish
function CalculateOptimalLoad(string oreName)
{
    variable float oreVolume = ${Item[${oreName}].Volume}
    variable float cargoCapacity = ${Ship.Cargo.Capacity}

    variable int maxLoad = ${Math.Calc[${cargoCapacity} / ${oreVolume}].Int}

    echo "${Me.Ship.Name} can carry ${maxLoad} units of ${oreName}"
    echo "Total volume: ${Math.Calc[${maxLoad} * ${oreVolume}].Int}m3"

    return ${maxLoad}
}

// Usage
call This.CalculateOptimalLoad "Veldspar"
```

---

## Complete Working Examples

### Example 1: Simple Station-to-Station Hauler

```lavish
objectdef obj_SimpleStationHauler
{
    variable string PickupStation = "Jita IV - Moon 4 - Caldari Navy Assembly Plant"
    variable string DeliveryStation = "Amarr VIII (Oris) - Emperor Family Academy"

    variable string CurrentState = "IDLE"

    method Initialize()
    {
        echo "Simple Station Hauler initialized"
        echo "Pickup: ${This.PickupStation}"
        echo "Delivery: ${This.DeliveryStation}"
    }

    method Start()
    {
        echo "Starting hauling operation"

        while TRUE
        {
            This:DetermineState
            This:ExecuteState

            wait 20
        }
    }

    method DetermineState()
    {
        // Safety checks
        if !${Me.InStation} && ${Social.PossibleHostiles}
        {
            This.CurrentState:Set["FLEE"]
            return
        }

        // If in pickup station with empty cargo, load
        if ${Me.InStation} && ${Me.Station.Name.Equal["${This.PickupStation}"]}
        {
            if ${Ship.Cargo.UsedCapacity} < 100
            {
                This.CurrentState:Set["LOAD"]
                return
            }
        }

        // If in delivery station with cargo, unload
        if ${Me.InStation} && ${Me.Station.Name.Equal["${This.DeliveryStation}"]}
        {
            if ${Ship.Cargo.UsedCapacity} > 0
            {
                This.CurrentState:Set["UNLOAD"]
                return
            }
        }

        // If in wrong station, travel
        if ${Me.InStation}
        {
            // Have cargo? Go to delivery
            if ${Ship.Cargo.UsedCapacity} > 0
            {
                This.CurrentState:Set["TRAVEL_TO_DELIVERY"]
            }
            else
            {
                This.CurrentState:Set["TRAVEL_TO_PICKUP"]
            }
            return
        }

        // If in space, wait for navigation to complete
        This.CurrentState:Set["TRAVELING"]
    }

    method ExecuteState()
    {
        switch ${This.CurrentState}
        {
            case IDLE
                echo "Idle"
                break

            case LOAD
                echo "Loading cargo from ${This.PickupStation}"
                call This.LoadCargo
                break

            case UNLOAD
                echo "Unloading cargo at ${This.DeliveryStation}"
                call Cargo.TransferCargoToStationHangar
                break

            case TRAVEL_TO_PICKUP
                echo "Traveling to pickup station"
                call This.TravelToPickup
                break

            case TRAVEL_TO_DELIVERY
                echo "Traveling to delivery station"
                call This.TravelToDelivery
                break

            case TRAVELING
                // Just wait for navigation
                wait 100
                break

            case FLEE
                echo "FLEE: Hostiles detected!"
                call This.EmergencyDock
                break
        }
    }

    function LoadCargo()
    {
        // Load everything from hangar
        call Inventory.StationHangar.Activate ${Me.Station.ID}
        wait 20

        variable index:item hangarItems
        Inventory.StationHangar:GetItems[hangarItems]

        variable iterator itemIterator
        hangarItems:GetIterator[itemIterator]

        if ${itemIterator:First(exists)}
        {
            do
            {
                itemIterator.Value:MoveTo[MyShip, CargoHold]
                wait 10
            }
            while ${itemIterator:Next(exists)}
        }

        echo "Cargo loaded: ${Ship.Cargo.PercentFull.Int}% full"
    }

    function TravelToPickup()
    {
        if !${EVE.Bookmark["${This.PickupStation}"](exists)}
        {
            echo "ERROR: Pickup station bookmark not found"
            return FALSE
        }

        Navigator:FlyToBookmark["${This.PickupStation}", 0, TRUE]
        while ${Navigator.Busy}
        {
            wait 10
        }
    }

    function TravelToDelivery()
    {
        if !${EVE.Bookmark["${This.DeliveryStation}"](exists)}
        {
            echo "ERROR: Delivery station bookmark not found"
            return FALSE
        }

        Navigator:FlyToBookmark["${This.DeliveryStation}", 0, TRUE]
        while ${Navigator.Busy}
        {
            wait 10
        }
    }

    function EmergencyDock()
    {
        if ${Me.InStation}
            return

        // Find nearest station
        variable index:entity stations
        EVE:QueryEntities[stations, "GroupID = 15 || GroupID = 1657", "Distance"]

        if ${stations.Used} > 0
        {
            call Station.DockAtStation ${stations[1].ID}
        }
        else
        {
            call Safespots.WarpTo
        }
    }
}

// Usage
variable obj_SimpleStationHauler Hauler

function main()
{
    Hauler:Initialize
    Hauler:Start
}
```

### Example 2: Mining Support Hauler

Complete hauler that supports a mining fleet:

```lavish
objectdef obj_MiningHauler
{
    variable string CurrentState = "IDLE"
    variable string DeliveryStation = "Jita IV - Moon 4 - Caldari Navy Assembly Plant"
    variable string MiningSystem = "Ikoskio"

    variable queue:int64 FullMinerCans

    method Initialize()
    {
        echo "Mining Hauler initialized"

        Event[EVENT_ONFRAME]:AttachAtom[This:Pulse]

        // Listen for miner full events
        LavishScript:RegisterEvent[EVEBot_Miner_Full]
        Event[EVEBot_Miner_Full]:AttachAtom[This:OnMinerFull]
    }

    atom OnMinerFull(int64 fleetMemberID, int64 systemID, int64 canID)
    {
        if ${systemID} == ${EVE.Bookmark["${This.MiningSystem}"].SolarSystemID}
        {
            echo "Miner ${fleetMemberID} reports full can"
            This.FullMinerCans:Queue[${canID}]
        }
    }

    method Pulse()
    {
        This:DetermineState
        This:ExecuteState

        wait 20
    }

    method DetermineState()
    {
        // Safety
        if !${Me.InStation} && ${Social.PossibleHostiles}
        {
            This.CurrentState:Set["FLEE"]
            return
        }

        if ${Ship.IsPod}
        {
            This.CurrentState:Set["PODDED"]
            return
        }

        // If cargo full, return to station
        if !${Me.InStation} && ${Ship.Cargo.PercentFull} > 90
        {
            This.CurrentState:Set["RETURN_TO_STATION"]
            return
        }

        // If in delivery station with cargo, unload
        if ${Me.InStation} && ${Me.Station.Name.Equal["${This.DeliveryStation}"]}
        {
            if ${Ship.Cargo.UsedCapacity} > 0
            {
                This.CurrentState:Set["UNLOAD"]
                return
            }
            else
            {
                This.CurrentState:Set["RETURN_TO_MINING"]
                return
            }
        }

        // If in mining system, collect cans
        if ${Me.SolarSystem.Name.Equal["${This.MiningSystem}"]}
        {
            if ${This.FullMinerCans.Used} > 0
            {
                This.CurrentState:Set["COLLECT_CANS"]
                return
            }
            else
            {
                This.CurrentState:Set["WAIT_AT_SAFE"]
                return
            }
        }

        This.CurrentState:Set["TRAVELING"]
    }

    method ExecuteState()
    {
        switch ${This.CurrentState}
        {
            case IDLE
                break

            case COLLECT_CANS
                call This.CollectCans
                break

            case RETURN_TO_STATION
                call This.ReturnToStation
                break

            case UNLOAD
                call This.UnloadCargo
                break

            case RETURN_TO_MINING
                call This.ReturnToMining
                break

            case WAIT_AT_SAFE
                call This.WaitAtSafe
                break

            case FLEE
                echo "FLEE!"
                call This.EmergencyDock
                break

            case PODDED
                echo "I'm in a pod - docking immediately"
                call This.EmergencyDock
                break

            case TRAVELING
                wait 100
                break
        }
    }

    function CollectCans()
    {
        if ${This.FullMinerCans.Used} == 0
            return

        variable int64 canID = ${This.FullMinerCans.Peek}

        if !${Entity[${canID}](exists)}
        {
            echo "Can ${canID} no longer exists"
            This.FullMinerCans:Dequeue
            return
        }

        echo "Collecting can ${Entity[${canID}].Name} at ${Math.Calc[${Entity[${canID}].Distance}/1000].Int}km"

        // Warp if needed
        if ${Entity[${canID}].Distance} > 150000
        {
            Entity[${canID}]:WarpTo[50000]
            wait 20
            while ${Me.ToEntity.Mode} == 3
            {
                wait 10
            }
        }

        // Approach
        if ${Entity[${canID}].Distance} > LOOT_RANGE
        {
            call Ship.Approach ${canID} LOOT_RANGE

            variable int timeout = 0
            while ${Entity[${canID}].Distance} > LOOT_RANGE && ${timeout} < 600
            {
                wait 10
                timeout:Inc[10]
            }
        }

        // Stop ship
        EVE:Execute[CmdStopShip]
        wait 20

        // Loot
        call This.LootCan ${canID}

        // Remove from queue
        This.FullMinerCans:Dequeue
    }

    function LootCan(int64 canID)
    {
        Entity[${canID}]:Open
        wait 30

        variable index:item canItems
        Entity[${canID}]:GetCargo[canItems]

        if ${canItems.Used} == 0
        {
            echo "Can is empty"
            Entity[${canID}]:Close
            return
        }

        // Build item list
        variable index:int64 itemIDs
        variable iterator itemIterator
        canItems:GetIterator[itemIterator]

        if ${itemIterator:First(exists)}
        {
            do
            {
                itemIDs:Insert[${itemIterator.Value.ID}]
            }
            while ${itemIterator:Next(exists)}
        }

        // Move to ship
        EVE:MoveItemsTo[itemIDs, MyShip, CargoHold]
        wait 30

        Entity[${canID}]:Close

        echo "Can looted"
    }

    function ReturnToStation()
    {
        echo "Returning to station"
        Navigator:FlyToBookmark["${This.DeliveryStation}", 0, TRUE]
        while ${Navigator.Busy}
        {
            wait 10
        }
    }

    function UnloadCargo()
    {
        echo "Unloading cargo"
        call Cargo.TransferCargoToStationHangar
        wait 20
    }

    function ReturnToMining()
    {
        echo "Returning to mining system"
        call Station.Undock
        wait 50 ${Me.InSpace}

        Navigator:FlyToBookmark["${This.MiningSystem}", 0]
        while ${Navigator.Busy}
        {
            wait 10
        }
    }

    function WaitAtSafe()
    {
        if ${Me.InSpace}
        {
            call Safespots.WarpTo
        }

        wait 100
    }

    function EmergencyDock()
    {
        if ${Me.InStation}
            return

        variable index:entity stations
        EVE:QueryEntities[stations, "GroupID = 15 || GroupID = 1657", "Distance"]

        if ${stations.Used} > 0
        {
            call Station.DockAtStation ${stations[1].ID}
        }
    }
}
```

---

## Common Problems

### Problem 1: Navigator Gets Stuck

**Symptom**: Bot warps to location but Navigator.Busy never becomes false

**Diagnosis**:
```lavish
echo "Navigator.Busy: ${Navigator.Busy}"
echo "Navigator.DestinationID: ${Navigator.DestinationID}"
echo "Navigator.DestinationType: ${Navigator.DestinationType}"
```

**Solution**:
```lavish
// Add timeout to navigation waits
Navigator:FlyToBookmark["MyBookmark", 0, TRUE]

variable int timeout = 0
while ${Navigator.Busy} && ${timeout} < 600
{
    wait 10
    timeout:Inc[10]
}

if ${timeout} >= 600
{
    echo "Navigation timeout - clearing Navigator"
    Navigator:Clear
}
```

### Problem 2: Cargo Transfer Fails

**Symptom**: Items don't move from cargo to hangar

**Diagnosis**:
```lavish
echo "In station: ${Me.InStation}"
echo "Station ID: ${Me.StationID}"
echo "Cargo used: ${Ship.Cargo.UsedCapacity}"
```

**Solution**:
```lavish
// Ensure windows are activated
call Inventory.ShipCargo.Activate
wait 30

call Inventory.StationHangar.Activate ${Me.Station.ID}
wait 30

// Check that activation worked
if !${Inventory.ShipCargo.IsCurrent}
{
    echo "ERROR: Failed to activate ship cargo"
    return FALSE
}
```

### Problem 3: Can't Find Fleet Member

**Symptom**: "Fleet member not on grid" but they're in system

**Diagnosis**:
```lavish
echo "Fleet member in local: ${Local[${FleetMemberName}](exists)}"
echo "Fleet member entity: ${Entity[Name = \"${FleetMemberName}\"](exists)}"
echo "Fleet member ID: ${Local[${FleetMemberName}].ToEntity.ID}"
```

**Solution**:
```lavish
// Use ToFleetMember for reliable warping
if ${Local[${FleetMemberName}](exists)}
{
    if ${Local[${FleetMemberName}].ToFleetMember(exists)}
    {
        Local[${FleetMemberName}].ToFleetMember:WarpTo
    }
}
```

### Problem 4: Autopilot Goes Through Low-Sec

**Symptom**: Route calculation includes dangerous systems

**Solution**:
```lavish
// Set route preference to safer
EVE:SetWaypointPreference[Safer]

// Verify route before traveling
if ${Autopilot.LowSecRoute}
{
    echo "ERROR: Route includes low-sec - cannot travel safely"
    return FALSE
}
```

### Problem 5: Tractor Beam Doesn't Activate

**Symptom**: Items too far to loot but tractor doesn't work

**Diagnosis**:
```lavish
echo "Has tractor: ${Ship.ModuleList_Tractor.Used}"
echo "Tractor range: ${Ship.OptimalTractorRange}"
echo "Target locked: ${Entity[${TargetID}].IsLockedTarget}"
```

**Solution**:
```lavish
// Ensure target is locked and active
Entity[${targetID}]:LockTarget
wait 10 ${Entity[${targetID}].BeingTargeted}

variable int timeout = 0
while !${Entity[${targetID}].IsLockedTarget} && ${timeout} < 300
{
    wait 10
    timeout:Inc[10]
}

if !${Entity[${targetID}].IsLockedTarget}
{
    echo "Failed to lock target"
    return FALSE
}

// Make active target
Entity[${targetID}]:MakeActiveTarget
wait 10

// Now activate tractor
Ship:Activate_Tractor
```

### Problem 6: Session Change Errors

**Symptom**: "Session change timeout" when docking/jumping

**Solution**:
```lavish
// Always wait after session changes
Entity[${gateID}]:Jump
wait 50    // Wait for session change to START

// Then wait for completion
variable int timeout = 0
while ${Me.InStation} && ${timeout} < 300
{
    wait 10
    timeout:Inc[10]
}
```

### Diagnostic: Hauler Status Report

```lavish
function DiagnosticReport()
{
    echo "==== HAULER DIAGNOSTIC REPORT ===="
    echo "Location: ${Me.SolarSystem.Name} (${Me.SolarSystem.Security.Precision[2]})"
    echo "In Station: ${Me.InStation}"
    if ${Me.InStation}
    {
        echo "  Station: ${Me.Station.Name}"
    }

    echo "Ship: ${Me.Ship.Name}"
    echo "  Cargo: ${Ship.Cargo.UsedCapacity.Int}/${Ship.Cargo.Capacity.Int}m3 (${Ship.Cargo.PercentFull.Int}%)"
    echo "  Cargo value: ${Math.Calc[${This.CargoValue} / 1000000].Int}M ISK"

    echo "State: ${This.CurrentState}"
    echo "Navigator busy: ${Navigator.Busy}"

    echo "Fleet members: ${Me.Fleet.MemberCount}"
    echo "Local count: ${Local.Count}"
    echo "Hostiles: ${Social.PossibleHostiles}"

    echo "===================================="
}
```

---

## Summary

This file covered:

1. **Autopilot and Navigation**: Route planning, system-to-system travel, bookmark warping
2. **Station Operations**: Docking, undocking, cargo transfer patterns
3. **Hauler State Machine**: Safety-first design with FLEE and HARDSTOP states
4. **Fleet Hauling**: Jetcan pickup, tractor beams, on-demand service
5. **Orca Service**: Fleet hangar transfers, cargo monitoring
6. **Stealth Hauling**: Covops cloak mechanics, gate jump protocol
7. **Route Safety**: Low-sec detection, gank risk calculation
8. **Cargo Optimization**: Value calculation, priority loading
9. **Working Examples**: Station hauler, mining support hauler
10. **Common Problems**: Navigation timeouts, cargo transfer issues, session changes

**Key Takeaway**: Hauling bots prioritize **SAFETY FIRST** - always check for hostiles, have emergency dock procedures, and use the FLEE state liberally. A successful hauler is one that doesn't get blown up!
