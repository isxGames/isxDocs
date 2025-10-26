# Navigation Library Patterns

**Advanced pathfinding and navigation from EQ2Nav**

---

## Table of Contents

1. [Overview](#overview)
2. [LavishNav Integration](#lavishnav-integration)
3. [Path Planning with Dijkstra](#path-planning-with-dijkstra)
4. [Region-Based Navigation](#region-based-navigation)
5. [Collision Detection](#collision-detection)
6. [Stuck Detection and Recovery](#stuck-detection-and-recovery)
7. [Door Automation](#door-automation)
8. [Aggro Detection Integration](#aggro-detection-integration)
9. [Direct vs Pathfinding Movement](#direct-vs-pathfinding-movement)
10. [Precision Management](#precision-management)
11. [Performance Optimization](#performance-optimization)
12. [Complete Working Example](#complete-working-example)

---

## Overview

EQ2Nav is a production-grade navigation library that provides sophisticated pathfinding and movement automation.

**Source:** https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Navigation

**Key Features:**
- LavishNav pathfinding integration
- Dijkstra shortest path algorithm
- Region-based mapping system
- Collision and steep terrain detection
- Automatic stuck detection and recovery
- Door automation
- Aggro detection integration
- CPU-optimized pulse system
- Direct movement fallback

**Files:**
- `EQ2Nav_Lib.iss` (1,187 lines) - Main navigation engine
- `EQ2NavMapper_Lib.iss` - Map creation and management
- `EQ2NavAggressionHandler.iss` - Combat detection
- `EQ2NavObstacleHandler.iss` - Stuck recovery
- `EQ2NavFaceClass_Lib.iss` - Facing logic

---

## LavishNav Integration

### Pattern: LavishNav Region System

**Concept:** Use LavishNav's built-in pathfinding engine with region-based navigation

```lavishscript
objectdef EQ2Nav
{
    variable EQ2Mapper Mapper
    variable lnavpath Path
    variable index:EQ2NavPath NavigationPath
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

    ; Find containing regions
    CurrentRegion:SetRegion[${ZoneRegion.BestContainer[${Me.Loc}].ID}]
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
objectdef EQ2NavPath
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

    ; Short distance + no collision = direct path available
    if ${Math.Distance[${Me.Loc},${X},${Y},${Z}]} < 40 && \
        !${Me.CheckCollision[${X},${Y},${Z}]} && \
        !${Mapper.Topography.IsSteep[${Me.Loc},${X},${Y},${Z}]}
    {
        return TRUE
    }

    ; Check for navigation path
    Path:Clear
    ZoneRegion:SetRegion[${LNavRegion[${Mapper.ZoneText}]}]
    DestZoneRegion:SetRegion[${LavishNav.FindRegion[${Mapper.ZoneText}]}]
    CurrentRegion:SetRegion[${ZoneRegion.BestContainer[${Me.Loc}].ID}]
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
; Find closest station from list
variable string ClosestStation
ClosestStation:Set[${Nav.ClosestRegionToPoint[${Actor[Banker].X},${Actor[Banker].Y},${Actor[Banker].Z},"Bank 1","Bank 2","Bank 3"]}]
Nav:MoveToRegion[${ClosestStation}]
```

---

## Collision Detection

### Pattern: Multi-Check Movement Validation

**Check collision, steep terrain, and distance:**

```lavishscript
method MoveToLoc(float X, float Y, float Z)
{
    ; Validate destination
    if ${X}==0 && ${Y}==0 && ${Z}==0
    {
        return
    }

    ; Already at destination?
    if ${Math.Distance[${Me.Loc},${X},${Y},${Z}]} < ${This.PRECISION}
    {
        This:Output["Already here, not moving."]
        This:ClearPath
        This.MeMoving:Set[FALSE]
        return
    }

    ; Prevent duplicate path to same destination
    if ${This.NavigationPath.Get[1](exists)}
    {
        if ${This.NavDestination.X}==${X} && ${This.NavDestination.Y}==${Y} && ${This.NavDestination.Z}==${Z}
        {
            This:Output["Error: Calling again to same destination. Aborting!"]
            This.MeMoving:Set[FALSE]
            return
        }
    }

    This:ClearPath

    ; Close + no collision = direct movement
    if ${Math.Distance[${Me.Loc},${X},${Y},${Z}]} < 20 && !${Me.CheckCollision[${X},${Y},${Z}]}
    {
        This:Debug["Moving to ${X},${Y},${Z} directly."]
        This.MovingToNearestRegion:Set[TRUE]
        This.MovingTo_X:Set[${X}]
        This.MovingTo_Y:Set[${Y}]
        This.MovingTo_Z:Set[${Z}]
        This.NavDestination:Set[${X},${Y},${Z}]
        This.MeMoving:Set[TRUE]
        return
    }

    ; Use pathfinding
    else
    {
        This:PopulatePath[${X},${Y},${Z}]

        if ${This.Path.Hops} > 0
        {
            This:PopulateNavigationPath
            This.NavDestination:Set[${X},${Y},${Z}]
            This.MeMoving:Set[TRUE]
            return
        }
        else
        {
            This:Output["Not enough mapping data. Moving to nearest mapped point..."]
            This:MoveToNearestRegion[${X},${Y},${Z}]
        }
    }
}
```

**Key Decision Points:**
1. Distance < 20m + no collision = direct movement
2. Distance ≥ 20m = use pathfinding
3. No path found = move to nearest region

---

## Stuck Detection and Recovery

### Pattern: Multi-Level Stuck Detection

**Track movement and trigger recovery:**

```lavishscript
variable int NotMovingPassCount
variable float CheckX
variable float CheckY
variable float CheckZ
variable int CheckLocSet
variable int CheckLocPassCount

method MoveTo(float X, float Y, float Z, float fPrecision)
{
    ; Record current position
    This.CheckX:Set[${Me.X}]
    This.CheckY:Set[${Me.Y}]
    This.CheckZ:Set[${Me.Z}]
    This.CheckLocPassCount:Inc
    This.CheckLocSet:Set[${Time.Timestamp}]

    ; ... movement logic ...
}

; In Pulse() method
if ${This.CheckLocPassCount} > 2
{
    if ${Math.Calc64[${Time.Timestamp}-${CheckLocSet}]} < 2
    {
        ; Haven't moved more than 0.1m in 2 seconds
        if ${Math.Distance[${Me.Loc},${This.CheckX},${This.CheckY},${This.CheckZ}]} < 0.1
        {
            This:Debug["We must be stuck...handling."]
            This:StopRunning
            runscript "${LavishScript.HomeDirectory}/Scripts/EQ2Navigation/EQ2NavObstacleHandler.iss" "${This.AUTORUN}" "${This.MOVEFORWARD}" "${This.MOVEBACKWARD}" "${This.STRAFELEFT}" "${This.STRAFERIGHT}" "${This.BackupTime}" "${This.StrafeTime}"
            This.CheckLocPassCount:Set[0]
            return
        }
    }
}

; Check if not moving during navigation
if !${Me.IsMoving}
{
    ; If we have not moved in more than 4 pulses, then we must be stuck
    if ${This.NotMovingPassCount} > 4
    {
        This:Debug["We must be stuck...handling. (NotMovingPassCount: ${This.NotMovingPassCount})"]
        This:StopRunning
        runscript "${LavishScript.HomeDirectory}/Scripts/EQ2Navigation/EQ2NavObstacleHandler.iss" "${This.AUTORUN}" "${This.MOVEFORWARD}" "${This.MOVEBACKWARD}" "${This.STRAFELEFT}" "${This.STRAFERIGHT}" "${This.BackupTime}" "${This.StrafeTime}"
        This.NotMovingPassCount:Set[0]
        return
    }
    This.NotMovingPassCount:Inc
}
else
{
    ; If we have moved more than 0.1m, then reset the NotMovingPassCount
    if ${Math.Abs[${This.NextHopDistance}-${This.HopDistanceCheck}]} > 0.1
        This.NotMovingPassCount:Set[0]
}
```

**Recovery Script Pattern:**
```lavishscript
; EQ2NavObstacleHandler.iss launched with parameters
runscript "EQ2NavObstacleHandler.iss" \
    "${AUTORUN_KEY}" \
    "${FORWARD_KEY}" \
    "${BACKWARD_KEY}" \
    "${STRAFELEFT_KEY}" \
    "${STRAFERIGHT_KEY}" \
    "${BackupTime}" \
    "${StrafeTime}"
```

**Key Techniques:**
- Position tracking every MoveTo call
- Time-based stuck detection (2 seconds)
- Pass count stuck detection (4+ pulses)
- Distance-based movement verification (0.1m threshold)
- External obstacle handler script

---

## Door Automation

### Pattern: Auto-Click Nearby Doors

**Track and click doors once:**

```lavishscript
variable collection:string DoorsOpenedThisTrip

; In Pulse() method
variable int DoorID
DoorID:Set[${Actor[door,xzrange,5,yrange,1].ID}]

if ${DoorID(exists)}
{
    if !${This.DoorsOpenedThisTrip.Element[${DoorID}](exists)}
    {
        Actor[id,${DoorID}]:DoubleClick
        This.DoorsOpenedThisTrip:Set[${DoorID},"Door"]
    }
}

; Clear when movement completes
This.DoorsOpenedThisTrip:Clear
```

**Key Techniques:**
- `collection:string` to track opened doors
- Use Actor ID as key
- XZ range 5m, Y range 1m for door detection
- Clear collection when destination reached
- Prevents double-clicking same door

---

## Aggro Detection Integration

### Pattern: External Aggro Handler

**Pause navigation when aggro detected:**

```lavishscript
#include "${LavishScript.HomeDirectory}/Scripts/EQ2Navigation/EQ2NavAggressionHandler.iss"

variable bool IgnoreAggro = FALSE
variable bool AggroDetected
variable int AggroDetectionTimer

method CheckAggro()
{
    ; Skip if configured to ignore aggro
    if ${This.IgnoreAggro}
        return

    if ${MobCheck.Detect}
    {
        This:Output["Agression Detected"]
        This:StopRunning
        This.AggroDetected:Set[TRUE]
    }
    else
        This.AggroDetected:Set[FALSE]

    AggroDetectionTimer:Set[${Time.Timestamp}]
}

; In Pulse() method
variable bool PreviousAggro
PreviousAggro:Set[${AggroDetected}]

; Check for aggression every 3 seconds
if ${Math.Calc64[${Time.Timestamp}-${AggroDetectionTimer}]} > 3
    This:CheckAggro

if ${AggroDetected}
    return  ; Skip navigation pulse
else
{
    if ${PreviousAggro}
    {
        This:CheckLoot  ; Check for loot after combat
        return
    }
}
```

**Loot Detection:**
```lavishscript
variable bool LootNearby

method CheckLoot()
{
    if ${Actor[chest,radius,15].Name(exists)} || ${Actor[corpse,radius,15].Name(exists)}
    {
        This:Debug["Loot nearby..."]
        This.LootNearby:Set[TRUE]
    }
    else
        This.LootNearby:Set[FALSE]
}
```

**Key Techniques:**
- `MobCheck` object from aggression handler
- Time-based checking (every 3 seconds)
- Pause navigation when aggro detected
- Resume navigation when clear
- Automatic loot detection after combat

---

## Direct vs Pathfinding Movement

### Pattern: Smart Movement Selection

**Choose direct or pathfinding based on distance:**

```lavishscript
method MoveTo(float X, float Y, float Z, float fPrecision)
{
    ; No navigation path = direct movement
    if !${This.NavigationPath.Get[1](exists)} || ${This.NavigationPath.Used} == 0
    {
        ; Check if we're within precision
        if ${Math.Distance[${Me.X},${Me.Y},${Me.Z},${X},${Y},${Z}]} <= ${DestinationPrecision}
        {
            This:StopRunning
            face ${X} ${Y} ${Z}
            This.MeMoving:Set[FALSE]
            return
        }

        ; Distance > 12m and path exists = use pathfinding
        if ${Math.Distance[${Me.X},${Me.Y},${Me.Z},${X},${Y},${Z}]} > 12 && ${This.UseMapping}
        {
            if ${This.PointsConnect[${Me.X},${Me.Y},${Me.Z},${X},${Y},${Z}]}
            {
                This:Debug["Destination > 10m and path exists -- using it."]
                This.MovingToNearestRegion:Set[FALSE]
                This:MoveToLocNoMapping[${X},${Y},${Z},${This.gPrecision}]
                return
            }
        }

        ; Direct movement
        face ${X} ${Y} ${Z}
        This:StartRunning
        This.MeMoving:Set[TRUE]
        return
    }

    ; Have navigation path = use it
    if ${Math.Distance[${Me.X},${Me.Y},${Me.Z},${X},${Y},${Z}]} > ${fPrecision} || \
        ${Mapper.IsSteep[${X},${Y},${Z},${Me.Loc}]} || \
        ${Me.CheckCollision[${X},${Y},${Z}]}
    {
        face ${X} ${Y} ${Z}
        This:StartRunning
        This.MeMoving:Set[TRUE]
    }
    else
    {
        ; Within precision - stop
        This:StopRunning
        This.MeMoving:Set[FALSE]
        face ${X} ${Y} ${Z}

        if ${This.NavigationPath.Get[1](exists)}
        {
            This:ClearPath
        }
        return
    }
}
```

**Smart Destination Detection:**
```lavishscript
; Within 10m + no collision = skip to destination
if ${SmartDestinationDetection}
{
    if ${This.DestinationDistance} <= 10 && !${Me.CheckCollision[${This.NavDestination}]}
    {
        This:Debug["Within 10m of destination and no obstructions -- moving directly"]
        face ${This.NavDestination.X} ${This.NavDestination.Y} ${This.NavDestination.Z}
        Dest:Set[${This.NavDestination}]
        This:ClearPath
        This.MeMoving:Set[TRUE]
        This:MoveTo[${Dest},${This.gPrecision}]
        return
    }
}
```

---

## Precision Management

### Pattern: Dual Precision System

**Different precision for waypoints vs destination:**

```lavishscript
variable float gPrecision = 2              ; Waypoint precision
variable float DestinationPrecision = 5    ; Final destination precision

; Moving through waypoints
if ${Math.Distance[${Me.Loc},${This.NavigationPath.Get[1].Location}]} <= ${This.gPrecision}
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
    face ${This.NavDestination}
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
Nav.gPrecision:Set[1.5]                    ; Tighter for small zones
Nav.DestinationPrecision:Set[4]            ; Minimum to avoid bouncing

; Zone-specific adjustments
if ${Zone.ShortName.Find[tradeskill]}
    Nav.DestinationPrecision:Set[3]        ; Tighter for indoor zones
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
; With wait 0.5 in loop
Nav.SkipNavTime:Set[50]
do
{
    Nav:Pulse
    wait 0.5
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

#include "${LavishScript.HomeDirectory}/Scripts/EQ2Navigation/EQ2Nav_Lib.iss"

variable EQ2Nav Nav
variable bool Verbose = TRUE

function main(string regionname)
{
    if !${ISXEQ2.IsReady}
    {
        echo "ISXEQ2 not ready!"
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
        wait 0.5
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

#include "${LavishScript.HomeDirectory}/Scripts/EQ2Navigation/EQ2Nav_Lib.iss"

variable EQ2Nav Nav
variable bool Verbose = TRUE
variable index:string Waypoints

function main()
{
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
            wait 0.5
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

### Navigation with Aggro Handling

```lavishscript
; SafeNav.iss - Navigation with combat integration

#include "${LavishScript.HomeDirectory}/Scripts/EQ2Navigation/EQ2Nav_Lib.iss"

variable EQ2Nav Nav
variable bool Verbose = TRUE

function main(string destination)
{
    ; Initialize
    Nav:UseLSO[FALSE]
    Nav:LoadMap

    ; Enable aggro detection
    Nav.IgnoreAggro:Set[FALSE]

    echo "=== Safe Navigation ==="
    echo "Destination: ${destination}"

    ; Start navigation
    Nav:MoveToRegion[${destination}]

    ; Main loop with combat checks
    do
    {
        Nav:Pulse

        ; Check if aggro detected
        if ${Nav.AggroDetected}
        {
            echo "Aggro detected - pausing navigation"

            ; Wait for combat to end
            while ${Nav.AggroDetected}
            {
                ; Your combat logic here
                wait 5
                Nav:CheckAggro  ; Recheck aggro status
            }

            echo "Aggro cleared - resuming navigation"

            ; Check for loot
            if ${Nav.LootNearby}
            {
                echo "Loot detected nearby"
                ; Your looting logic here
                wait 20
            }
        }

        wait 0.5
    }
    while ${Nav.Moving}

    echo "=== Arrived Safely ==="
    Script:End
}
```

---

## Summary

### Key Patterns Learned

1. **LavishNav Integration**: Use built-in pathfinding with region system
2. **Dijkstra Pathfinding**: Shortest path algorithm with lnavpath
3. **Region Validation**: Check existence and path availability before moving
4. **Collision Detection**: Multiple checks (collision, steep, distance)
5. **Stuck Detection**: Multi-level detection (time, position, pass count)
6. **Door Automation**: Auto-click doors once using collection tracking
7. **Aggro Integration**: Pause navigation during combat
8. **Direct vs Pathfinding**: Smart selection based on distance
9. **Dual Precision**: Different precision for waypoints vs destination
10. **Performance Optimization**: Pulse throttling and timer-based updates

### Best Practices

- ✅ Always load map before navigating (`Nav:LoadMap`)
- ✅ Validate region existence before navigation
- ✅ Use dual precision (tight for waypoints, loose for destination)
- ✅ Implement aggro detection for safe navigation
- ✅ Track opened doors to prevent double-clicking
- ✅ Use stuck detection with external recovery script
- ✅ Throttle Pulse() calls to reduce CPU usage
- ✅ Clear path when destination reached
- ✅ Check collision before direct movement
- ✅ Provide fallback to nearest region when unmapped

### Configuration Variables

**Essential Settings:**
```lavishscript
Nav.gPrecision:Set[2]                      ; Waypoint precision (1-3m)
Nav.DestinationPrecision:Set[5]            ; Destination precision (4-6m)
Nav.SkipNavTime:Set[50]                    ; Pulse throttle (0-100)
Nav.SmartDestinationDetection:Set[TRUE]    ; Direct movement optimization
Nav.DirectMovingToTimer:Set[250]           ; Update frequency (ms)
Nav.IgnoreAggro:Set[FALSE]                 ; Enable aggro detection
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

- [14_Advanced_Scripting_Patterns.md](14_Advanced_Scripting_Patterns.md) - LavishSettings patterns
- [16_Crafting_Script_Patterns.md](16_Crafting_Script_Patterns.md) - Navigation usage examples
- [03_API_Reference.md](03_API_Reference.md) - Actor, Me datatypes

---

*Part of the ISXEQ2 Scripting Guide - Advanced Patterns Series*
*Based on EQ2Nav v2.x - https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Navigation*
