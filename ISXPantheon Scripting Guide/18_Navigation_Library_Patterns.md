# Navigation Library Patterns

**Pathfinding and navigation patterns built on LavishNav, the platform-agnostic InnerSpace navigation system.**

---

## Table of Contents

1. [Overview](#overview)
2. [LavishNav Integration](#lavishnav-integration)
3. [Path Planning with Dijkstra](#path-planning-with-dijkstra)
4. [Region-Based Navigation](#region-based-navigation)
5. [Position and Movement Execution (planned)](#position-and-movement-execution-planned)
6. [Door Automation (planned)](#door-automation-planned)
7. [Aggro Detection Integration (planned)](#aggro-detection-integration-planned)
8. [Precision Management](#precision-management)
9. [Performance Optimization](#performance-optimization)
10. [Complete Working Example](#complete-working-example)

---

## Overview

LavishNav is InnerSpace's region-based pathfinding system. It is platform-agnostic — the region graph, shortest-path planning, and precision/throttling mechanics work independently of any specific game. This guide documents those mechanics, plus how a navigation library is structured.

**LavishNav mechanics (platform-agnostic, documented here):**
- LavishNav region graph and pathfinding integration
- Dijkstra shortest-path algorithm
- Region-based mapping system
- Dual-precision waypoint/destination management
- CPU-optimized pulse throttling

**Movement-dependent pieces (planned — require game-data surface not yet implemented):**
- Reading the player's position/heading and issuing movement inputs
- Collision/steep-terrain detection against the live world
- Stuck detection and recovery (depends on player position over time)
- Door/object automation (depends on in-world entity interaction)
- Aggro detection integration (depends on entity enumeration)

> **NOTE.** The LavishNav region/pathfinding mechanics in this guide are agnostic and can be studied today. However, a fully working navigation bot also needs to read the player's position and drive movement — that surface (a local-player datatype plus movement inputs) is on the ISXPantheon roadmap but not yet implemented. All of that execution-layer content is collected under one section, [Position and Movement Execution (planned)](#position-and-movement-execution-planned). Where an agnostic example needs a start position, it uses a `point3f` placeholder (`This.CurrentLoc`) that the planned local-player surface will eventually fill.

---

## LavishNav Integration

### Pattern: LavishNav Region System

**Concept:** Use LavishNav's built-in pathfinding engine with region-based navigation

```lavishscript
objectdef Nav
{
    variable NavMapper Mapper
    variable lnavpath Path
    variable index:NavPath NavigationPath
    variable lnavregionref CurrentRegion
    variable point3f NavDestination

    method LoadMap()
    {
        Mapper.UseLSO:Set[${UsingLSO}]
        Mapper:LoadMap
        This:Output["Navigation Map Loaded"]
    }

    method UseLSO(bool UseIt)
    {
        if ${UseIt}
        {
            This.UsingLSO:Set[TRUE]
            Mapper.UseLSO:Set[TRUE]
            This:Output["Utilizing LSO File"]
        }
        else
        {
            This.UsingLSO:Set[FALSE]
            Mapper.UseLSO:Set[FALSE]
            This:Output["Utilizing XML Files"]
        }
    }
}
```

**File Format Options:**
```lavishscript
; LSO binary format (faster loading)
if ${UseLSO}
{
    LNavRegion[${ZoneText}]:Export[-lso,"${ZonesDir}${ZoneText}"]
    LoadedZoneRegion:Import[-lso,"${ZonesDir}${ZoneText}"]
}
; XML format (human-readable)
else
{
    LNavRegion[${ZoneText}]:Export["${ZonesDir}${ZoneText}.xml"]
    LoadedZoneRegion:Import["${ZonesDir}${ZoneText}.xml"]
}
```

**Key Techniques:**
- `lnavpath` datatype for path storage
- `lnavregionref` for region references
- LSO vs XML file formats
- Zone-specific map files

---

## Path Planning with Dijkstra

### Pattern: Dijkstra Shortest Path

**Find optimal path between two points:**

```lavishscript
method PopulatePath(float X, float Y, float Z)
{
    variable dijkstrapathfinder PathFinder
    variable lnavregionref CurrentRegion
    variable lnavregionref ZoneRegion
    variable lnavregionref DestZoneRegion
    variable lnavregionref DestinationRegion

    ; Get zone region
    ZoneRegion:SetRegion[${LNavRegion[${Mapper.ZoneText}]}]
    DestZoneRegion:SetRegion[${LavishNav.FindRegion[${Mapper.ZoneText}]}]

    ; Find containing regions. CurrentLoc is a point3f holding the start position;
    ; reading it from the live player is planned (see "Position and Movement Execution
    ; (planned)" below) -- for the pathfinding logic itself any point3f works.
    CurrentRegion:SetRegion[${ZoneRegion.BestContainer[${This.CurrentLoc}].ID}]
    DestinationRegion:SetRegion[${DestZoneRegion.BestContainer[${X},${Math.Calc[${Y}+1]},${Z}].ID}]

    ; Calculate shortest path
    PathFinder:SelectPath[${CurrentRegion.FQN},${DestinationRegion.FQN},This.Path]

    This:Debug["PopulatePath(): Destination: ${DestinationRegion.FQN} -- Hops: ${This.Path.Hops}"]
}
```

**Convert to navigation waypoints:**

```lavishscript
method PopulateNavigationPath()
{
    variable int Index = 2

    This.NavigationPath:Clear

    ; Convert lnavpath to index of waypoints
    do
    {
        This:Debug["Adding [${This.Path.Region[${Index}].CenterPoint}] - [${This.Path.Region[${Index}].FQN}]"]
        This.NavigationPath:Insert[${This.Path.Region[${Index}].CenterPoint},0,${This.Path.Region[${Index}].FQN}]
    }
    while ${Index:Inc} <= ${This.Path.Hops}

    This:Debug["${NavigationPath.Used} hops used."]
}
```

**Custom Path Waypoint Object:**

```lavishscript
objectdef NavPath
{
    variable point3f Location
    variable int Method
    variable string FQN

    method Initialize(float X, float Y, float Z, int MoveType, string Label)
    {
        This.Location:Set[${X},${Y},${Z}]
        This.Method:Set[${MoveType}]
        This.FQN:Set[${Label}]
    }
}
```

**Key Techniques:**
- `dijkstrapathfinder` datatype
- `SelectPath` method for pathfinding
- `BestContainer` to find containing region
- `+1` to Y coordinate for region lookup
- Store path as index of custom objects

---

## Region-Based Navigation

### Pattern: Check Path Availability

**Verify path exists before attempting navigation:**

```lavishscript
member:bool AvailablePath(float X, float Y, float Z)
{
    variable dijkstrapathfinder PathFinder
    variable lnavpath Path
    variable lnavregionref CurrentRegion
    variable lnavregionref ZoneRegion
    variable lnavregionref DestZoneRegion
    variable lnavregionref DestinationRegion

    if ${X}==0 && ${Y}==0 && ${Z}==0
    {
        return FALSE
    }

    ; Short distance = direct path likely available. A live-world collision/steep
    ; check would refine this (planned -- see "Position and Movement Execution
    ; (planned)" below); the agnostic graph test below is what runs today.
    if ${Math.Distance[${This.CurrentLoc},${X},${Y},${Z}]} < 40
    {
        return TRUE
    }

    ; Check for navigation path
    Path:Clear
    ZoneRegion:SetRegion[${LNavRegion[${Mapper.ZoneText}]}]
    DestZoneRegion:SetRegion[${LavishNav.FindRegion[${Mapper.ZoneText}]}]
    CurrentRegion:SetRegion[${ZoneRegion.BestContainer[${This.CurrentLoc}].ID}]
    DestinationRegion:SetRegion[${DestZoneRegion.BestContainer[${X},${Math.Calc[${Y}+1]},${Z}].ID}]
    PathFinder:SelectPath[${CurrentRegion.FQN},${DestinationRegion.FQN},Path]

    if ${Path.Hops}
    {
        return TRUE
    }
    else
    {
        return FALSE
    }
}

member:bool AvailablePathToRegion(string Region)
{
    variable lnavregionref DestinationRegion
    DestinationRegion:SetRegion[${Region}]
    return ${This.AvailablePath[${DestinationRegion.CenterPoint}]}
}
```

**Usage:**
```lavishscript
; Check before navigating
if ${Nav.AvailablePath[${X},${Y},${Z}]}
{
    Nav:MoveToLoc[${X},${Y},${Z}]
}
else
{
    echo "No path available!"
}
```

### Pattern: Find Closest Region

**Select nearest region from multiple options:**

```lavishscript
member:string ClosestRegionToPoint(float PointX, float PointY, float PointZ, ... RegionNames)
{
    variable string ClosestRegion
    variable float64 ClosestRegionDistance = 0
    variable float64 CurrentRegionDistance = 0
    variable int Pos

    if ${RegionNames.Size} > 0
    {
        if ${RegionNames.Size} == 1
        {
            return ${RegionNames[1]}
        }

        ; Check first region
        Pos:Set[1]
        CurrentRegionDistance:Set[${This.DistanceFromPointToRegion[${PointX},${PointY},${PointZ}, ${RegionNames[${Pos}]}]}]
        ClosestRegionDistance:Set[${CurrentRegionDistance}]
        ClosestRegion:Set[${RegionNames[${Pos}]}]

        ; Compare remaining regions
        while ${Pos:Inc} <= ${RegionNames.Size}
        {
            CurrentRegionDistance:Set[${This.DistanceFromPointToRegion[${PointX},${PointY},${PointZ}, ${RegionNames[${Pos}]}]}]

            if ${CurrentRegionDistance} != -1 && \
                ${CurrentRegionDistance} < ${ClosestRegionDistance}
            {
                ClosestRegionDistance:Set[${CurrentRegionDistance}]
                ClosestRegion:Set[${RegionNames[${Pos}]}]
            }
        }

        return ${ClosestRegion}
    }

    return ""
}

member:float64 DistanceFromPointToRegion(float PointX, float PointY, float PointZ, string RegionName)
{
    variable lnavregionref DestinationRegion
    DestinationRegion:SetRegion[${RegionName}]

    if ${DestinationRegion.Region(exists)}
    {
        return ${Math.Distance[${PointX},${PointY},${PointZ},${DestinationRegion.CenterPoint.X},${DestinationRegion.CenterPoint.Y},${DestinationRegion.CenterPoint.Z}]}
    }

    return -1
}
```

**Usage:**
```lavishscript
; Find the closest named region to a point, then move to it.
; Here the point is a literal coordinate; sourcing it from a live entity's
; position is planned (entity enumeration is not yet implemented).
variable string ClosestStation
ClosestStation:Set[${Nav.ClosestRegionToPoint[100.0,5.0,250.0,"Station 1","Station 2","Station 3"]}]
Nav:MoveToRegion[${ClosestStation}]
```

---

## Position and Movement Execution (planned)

> **PLANNED — NOT YET IMPLEMENTED.** Everything in this section EXECUTES navigation: reading the player's live position/heading, checking collision and steep terrain against the world, issuing movement inputs, and detecting/recovering from a stuck player. All of that requires a local-player datatype (a `${Me}`-style top-level object) and movement APIs that are PLANNED on the ISXPantheon roadmap but are not in the current build. In the source, `${Me}` exists only as a reserved (commented-out) top-level object; there is no working player-position member today. The member names used in the example below (for instance `Loc`, `X`/`Y`/`Z`, `CheckCollision`, `IsMoving`) are **provisional and illustrative of the technique only — they are NOT a confirmed Pantheon API and WILL change.** Do not write production scripts against them yet.

The agnostic LavishNav graph and pathfinding mechanics documented above (region graph, Dijkstra path selection, region validation, dual precision, pulse throttling) run today and produce a planned route. What this section adds — and what is still planned — is the execution layer that drives the player along that route: a `MoveToLoc` method that decides between direct movement and pathfinding, a stuck-detection check that compares position over time and launches a recovery script, and the per-pulse "are we there yet / are we stuck" logic. Once a local-player position source and movement inputs ship, that execution layer plugs into the route the agnostic mechanics already build.

### Pattern: Movement Execution (illustrative, planned API)

The single example below illustrates the whole execution shape: validate the destination, take the direct path when close (a collision check would gate this), otherwise fall back to the planned route, and detect a stalled position to trigger recovery. Treat every player-position read here as a placeholder for the planned local-player surface.

```lavishscript
; ILLUSTRATIVE ONLY -- the ${Me.*} reads below stand in for a PLANNED
; local-player position source; member names are provisional and will change.
method MoveToLoc(float X, float Y, float Z)
{
    ; Validate destination
    if ${X}==0 && ${Y}==0 && ${Z}==0
        return

    ; Already at destination? (player position is planned surface)
    if ${Math.Distance[${Me.Loc},${X},${Y},${Z}]} < ${This.PRECISION}
    {
        This:Output["Already here, not moving."]
        This:ClearPath
        This.MeMoving:Set[FALSE]
        return
    }

    This:ClearPath

    ; Close + no collision = direct movement; otherwise use the planned route
    ; produced by the agnostic pathfinding mechanics above.
    if ${Math.Distance[${Me.Loc},${X},${Y},${Z}]} < 20 && !${Me.CheckCollision[${X},${Y},${Z}]}
    {
        This.NavDestination:Set[${X},${Y},${Z}]
        This.MeMoving:Set[TRUE]
        return
    }
    else
    {
        This:PopulatePath[${X},${Y},${Z}]
        if ${This.Path.Hops} > 0
        {
            This:PopulateNavigationPath
            This.NavDestination:Set[${X},${Y},${Z}]
            This.MeMoving:Set[TRUE]
        }
    }
}

; Stuck detection (in Pulse): if the player has not moved while navigation is
; active, launch the external recovery script. Both the position read and the
; "is moving" check are planned surface.
if !${Me.IsMoving}
{
    if ${This.NotMovingPassCount} > 4
    {
        This:StopRunning
        runscript "${LavishScript.HomeDirectory}/Scripts/Navigation/NavObstacleHandler.iss" "${This.AUTORUN}" "${This.MOVEFORWARD}" "${This.MOVEBACKWARD}" "${This.STRAFELEFT}" "${This.STRAFERIGHT}" "${This.BackupTime}" "${This.StrafeTime}"
        This.NotMovingPassCount:Set[0]
        return
    }
    This.NotMovingPassCount:Inc
}
```

**What is agnostic vs planned here:**
- Agnostic (runs today): the decision structure (validate, direct-vs-route, throttled re-check), the `PopulatePath`/`PopulateNavigationPath` route building, and the external recovery-script launch mechanism.
- Planned (not yet available): every player-position/heading read, the collision/steep-terrain check, the "is moving" state, and the movement inputs that actually drive the player.

---

## Door Automation (planned)

> **PLANNED — NOT YET IMPLEMENTED.** Detecting and activating in-world objects such as doors requires entity enumeration (to find a nearby door) and an entity interaction method (to activate it) — neither is available in the current ISXPantheon build. The tracking technique a door-opener would use is itself agnostic (a `collection:string` keyed by entity ID, so each door is only activated once per trip, cleared when the destination is reached), but the entity detection and the activation call it depends on are planned. Do not write production scripts against it yet.

## Aggro Detection Integration (planned)

> **PLANNED — NOT YET IMPLEMENTED.** Pausing navigation when hostile entities engage the player requires entity enumeration and entity hostility/threat state — that surface is on the ISXPantheon roadmap but is not yet available. The control-flow scaffolding is agnostic: poll on a timer, set a flag, skip the navigation pulse while the flag is set, and run a post-event check (such as scanning for nearby loot) once it clears. But the hostility check and the nearby-object scan both depend on planned entity surface. Do not write production scripts against it yet.

## Precision Management

### Pattern: Dual Precision System

**Different precision for waypoints vs destination:**

The two precision thresholds are plain configuration values and are agnostic. They are *consumed* by the planned movement-execution layer, which compares them against the live player position (a planned surface — see "Position and Movement Execution (planned)" above). `CurrentLoc` below is a `point3f` standing in for that planned position read.

```lavishscript
variable float gPrecision = 2              ; Waypoint precision
variable float DestinationPrecision = 5    ; Final destination precision

; Moving through waypoints (CurrentLoc = planned live-position read)
if ${Math.Distance[${This.CurrentLoc},${This.NavigationPath.Get[1].Location}]} <= ${This.gPrecision}
{
    This:Debug["Within ${This.gPrecision}m of waypoint -- removing from path"]
    This.NavigationPath:Remove[1]
    This.NavigationPath:Collapse
}

; Arriving at destination
if ${This.DestinationDistance} < ${This.DestinationPrecision}
{
    This:Debug["Within ${This.DestinationPrecision}m of destination -- ending movement"]
    This:StopRunning
    This.MeMoving:Set[FALSE]
    This:ClearPath
    return
}
```

**Why Two Precision Values?**
- **gPrecision (2m)**: Tight precision for waypoints to avoid getting stuck
- **DestinationPrecision (5m)**: Looser precision for destination to avoid "bouncing"

**Configurable Values:**
```lavishscript
; Script can override defaults
Nav.gPrecision:Set[1.5]                    ; Tighter for small areas
Nav.DestinationPrecision:Set[4]            ; Minimum to avoid bouncing

; Context-specific adjustments (planned: keying off a live zone identifier
; requires a zone datatype that is not yet surfaced — use a config value today)
if ${TightSpaces}
    Nav.DestinationPrecision:Set[3]        ; Tighter for confined areas
```

---

## Performance Optimization

### Pattern: CPU-Saving Pulse Throttling

**Skip frames to reduce CPU usage:**

```lavishscript
variable int SKIPNAV
variable int SkipNavTime = 50

method Pulse()
{
    ; Only process every SkipNavTime pulses
    if ${This.SKIPNAV} < ${This.SkipNavTime}
    {
        This.SKIPNAV:Inc
        return
    }
    This.SKIPNAV:Set[0]

    ; ... navigation logic ...
}
```

**Usage in Main Loop:**
```lavishscript
; With wait 1 in loop
Nav.SkipNavTime:Set[50]
do
{
    Nav:Pulse
    wait 1
}
while ${Nav.Moving}

; With waitframe in loop
Nav.SkipNavTime:Set[0]   ; Process every frame
do
{
    Nav:Pulse
    waitframe
}
while ${Nav.Moving}
```

**Timer-Based Throttling:**
```lavishscript
variable int DirectMovingToTimer = 250
variable int MovingTo_Timer

; Only update direct movement every 250ms
if ${LavishScript.RunningTime}-${This.MovingTo_Timer} > ${This.DirectMovingToTimer}
{
    This:MoveTo[${This.MovingTo_X},${This.MovingTo_Y},${This.MovingTo_Z},${This.MovingTo_Precision}]
    This.MovingTo_Timer:Set[${LavishScript.RunningTime}]
}
```

---

## Complete Working Example

### Simple Navigation Bot

```lavishscript
; SimpleNav.iss - Basic navigation automation
; Usage: run SimpleNav "Region Name"

#include "${LavishScript.HomeDirectory}/Scripts/Navigation/Nav_Lib.iss"

variable Nav Nav
variable bool Verbose = TRUE

function main(string regionname)
{
    if !${ISXPantheon.IsReady}
    {
        echo "ISXPantheon not ready!"
        Script:End
    }

    if !${regionname.Length}
    {
        echo "Usage: run SimpleNav \"Region Name\""
        Script:End
    }

    ; Initialize navigation
    Nav:UseLSO[FALSE]           ; Use XML map files
    Nav:LoadMap

    ; Configure precision
    Nav.gPrecision:Set[2]
    Nav.DestinationPrecision:Set[5]
    Nav.SkipNavTime:Set[50]
    Nav.SmartDestinationDetection:Set[TRUE]

    ; Configure movement keys (default WASD)
    Nav.AUTORUN:Set["num lock"]
    Nav.MOVEFORWARD:Set["w"]
    Nav.MOVEBACKWARD:Set["s"]
    Nav.STRAFELEFT:Set["q"]
    Nav.STRAFERIGHT:Set["e"]

    echo "=== SimpleNav ==="
    echo "Moving to region: ${regionname}"

    ; Validate region exists
    if !${Nav.RegionExists[${regionname}]}
    {
        echo "Error: Region '${regionname}' not found!"
        Script:End
    }

    ; Start navigation
    Nav:MoveToRegion[${regionname}]

    ; Main navigation loop
    do
    {
        Nav:Pulse
        wait 1
    }
    while ${Nav.Moving}

    echo "=== Arrived at ${regionname} ==="
    Script:End
}
```

**Usage:**
```
run SimpleNav "Bank 1"
run SimpleNav "Broker"
run SimpleNav "Forge 3"
```

### Advanced Navigation with Waypoints

```lavishscript
; MultiStop.iss - Navigate through multiple waypoints
; Usage: run MultiStop

#include "${LavishScript.HomeDirectory}/Scripts/Navigation/Nav_Lib.iss"

variable Nav Nav
variable bool Verbose = TRUE
variable index:string Waypoints

function main()
{
    while !${ISXPantheon.IsReady}
        wait 10

    ; Initialize
    Nav:UseLSO[FALSE]
    Nav:LoadMap
    Nav.gPrecision:Set[2]
    Nav.DestinationPrecision:Set[5]

    ; Build waypoint list
    Waypoints:Insert["Bank 1"]
    Waypoints:Insert["Broker"]
    Waypoints:Insert["Wholesaler"]
    Waypoints:Insert["Forge 1"]

    echo "=== Multi-Stop Navigation ==="
    echo "Waypoints: ${Waypoints.Used}"

    variable int i
    for (i:Set[1]; ${i} <= ${Waypoints.Used}; i:Inc)
    {
        echo "Moving to waypoint ${i}: ${Waypoints[${i}]}"

        ; Validate region
        if !${Nav.RegionExists[${Waypoints[${i}]}]}
        {
            echo "Warning: Region '${Waypoints[${i}]}' not found - skipping"
            continue
        }

        ; Check if path available
        if !${Nav.AvailablePathToRegion[${Waypoints[${i}]}]}
        {
            echo "Warning: No path to '${Waypoints[${i}]}' - skipping"
            continue
        }

        ; Navigate to waypoint
        Nav:MoveToRegion[${Waypoints[${i}]}]

        do
        {
            Nav:Pulse
            wait 1
        }
        while ${Nav.Moving}

        echo "Arrived at ${Waypoints[${i}]}"

        ; Wait at waypoint
        wait 20
    }

    echo "=== Tour Complete ==="
    Script:End
}
```

### Navigation with Aggro Handling (planned)

> **PLANNED — NOT YET IMPLEMENTED.** A combat-aware navigation loop — pause the navigation pulse while hostile entities are engaged, resume when clear, and optionally scan for nearby loot afterward — depends on entity enumeration and hostility/threat state that are not yet available. The loop structure (pulse, check a hostility flag, wait, re-check, then resume) is the same scaffolding shown in the Aggro Detection Integration section above; it cannot be made to actually detect combat until that surface ships. Build the route logic now with the agnostic LavishNav mechanics, and add the combat gating once entity surface is available.

---

## Summary

### Key Patterns Learned

**LavishNav mechanics (runnable today):**

1. **LavishNav Integration**: Use built-in pathfinding with the region system
2. **Dijkstra Pathfinding**: Shortest-path algorithm with `lnavpath`
3. **Region Validation**: Check existence and path availability before moving
4. **Dual Precision**: Different precision for waypoints vs destination
5. **Performance Optimization**: Pulse throttling and timer-based updates

**Movement-dependent (planned — need game-data surface):**

6. **Position and Movement Execution**: Read the live player position, choose direct-vs-route movement, run collision/steep checks, and detect/recover from a stalled player — all under one planned section
7. **Door Automation**: Activate in-world objects once using collection tracking
8. **Aggro Integration**: Pause navigation during combat

### Best Practices

- Always load the map before navigating (`Nav:LoadMap`)
- Validate region existence before navigation
- Use dual precision (tight for waypoints, loose for destination)
- Throttle `Pulse()` calls to reduce CPU usage
- Clear the path when the destination is reached
- Provide a fallback to the nearest region when unmapped
- Build route logic now with the agnostic LavishNav mechanics; add collision, stuck, door, and aggro gating once the player-position and entity surface ships

### Configuration Variables

**Essential Settings:**
```lavishscript
Nav.gPrecision:Set[2]                      ; Waypoint precision (1-3m)
Nav.DestinationPrecision:Set[5]            ; Destination precision (4-6m)
Nav.SkipNavTime:Set[50]                    ; Pulse throttle (0-100)
Nav.SmartDestinationDetection:Set[TRUE]    ; Direct movement optimization
Nav.DirectMovingToTimer:Set[250]           ; Update frequency (ms)
Nav.IgnoreAggro:Set[FALSE]                 ; Enable aggro detection (planned — requires entity surface)
```

**Movement Keys:**
```lavishscript
Nav.AUTORUN:Set["num lock"]
Nav.MOVEFORWARD:Set["w"]
Nav.MOVEBACKWARD:Set["s"]
Nav.STRAFELEFT:Set["q"]
Nav.STRAFERIGHT:Set["e"]
Nav.TURNLEFT:Set["a"]
Nav.TURNRIGHT:Set["d"]
```

### Related Documentation

- [15_Advanced_Scripting_Patterns.md](15_Advanced_Scripting_Patterns.md) - LavishSettings and infrastructure patterns
- [03_API_Reference.md](03_API_Reference.md) - Real and planned datatypes
