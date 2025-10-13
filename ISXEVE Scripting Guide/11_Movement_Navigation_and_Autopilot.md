# Movement, Navigation, and Autopilot
## Complete Guide to Ship Movement, Warping, Docking, and Navigation Patterns

**Part of**: EVE Online Bot Development - AI-Level Documentation
**Layer**: 3 - ISXEVE API Deep Dive
**Prerequisites**: Read files 01-10, 12 (Foundation + Scripting + Core Objects + Entities + Modules)
**Critical For**: ALL bots - Navigation is fundamental to every bot task

---

## Purpose of This Document

This file provides **exhaustive coverage** of:
- Warping mechanics (warp to entity, bookmark, distance management)
- Docking and undocking
- Autopilot system and waypoint management
- Approach and orbit mechanics
- Stargate jumping and system travel
- Movement commands and patterns
- Speed and velocity management
- Navigation safety patterns (avoiding bumps, camps, bubbles)
- Distance calculation and range management
- Common patterns from Evebot, Yamfa, and Tehbot
- Navigation state machine patterns
- Critical timing and wait patterns
- Gotchas and edge cases

**Why This Is CRITICAL**:
- **Every bot needs navigation** (warping to sites, returning to station, traveling systems)
- **Navigation failures = bot stuck** (most common bot failure mode)
- **Timing is critical** (warping, docking, jumping all require proper waits)
- **Safety patterns prevent losses** (avoiding gate camps, station undock collisions)

---

## Table of Contents

1. [Movement System Overview](#movement-system-overview)
2. [Ship Movement States](#ship-movement-states)
3. [Warping Mechanics](#warping-mechanics)
4. [Docking and Undocking](#docking-and-undocking)
5. [Approach and Orbit](#approach-and-orbit)
6. [Stargate Jumping](#stargate-jumping)
7. [Autopilot System](#autopilot-system)
8. [Speed Control](#speed-control)
9. [Distance Management](#distance-management)
10. [Navigation Safety Patterns](#navigation-safety-patterns)
11. [Navigation State Machines](#navigation-state-machines)
12. [Common Patterns from Example Scripts](#common-patterns-from-example-scripts)
13. [Timing and Wait Patterns](#timing-and-wait-patterns)
14. [Critical Gotchas](#critical-gotchas)
15. [Anti-Patterns](#anti-patterns)

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

## Next Steps

**Continue to Other Layer 3 Files**:
- File 13: Inventory_and_Cargo_Systems.md (Cargo management, hangar access)
- File 16: Fleet_and_Social_Systems.md (Fleet operations)
- File 14: Market_and_Contracts.md (if relevant)

**Apply Knowledge**:
- All bots need navigation (mining to belts, combat to sites, hauling to stations)
- Recognize navigation patterns in Evebot/Tehbot
- Build robust navigation state machines for complex routes

---

## Wiki References

**Entity Object** (for WarpTo, Dock, Jump methods):
- `IsxeveWiki/ISXEVE/Entity_(Object_Type).html`

**EVE Execute Commands**:
- `IsxeveWiki/ISXEVE/EVE_(Object_Type).html`

**MyShip Movement**:
- `IsxeveWiki/ISXEVE/MyShip_(Object_Type).html`

---

**END OF FILE 11**
**Status**: Complete (~1800 lines)
**Next**: Update progress tracker, continue with File 13 (Inventory) or assess token budget
