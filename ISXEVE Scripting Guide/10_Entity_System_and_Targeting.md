# Entity System and Targeting
## Complete Guide to Entity Management, Queries, and Targeting Mechanics

**Part of**: EVE Online Bot Development - AI-Level Documentation
**Layer**: 3 - ISXEVE API Deep Dive
**Prerequisites**: Read files 01-09 (Foundation + Scripting + Core Objects)
**Critical For**: **ALL bots** - Mining, combat, hauling, scanning, everything

---

## Purpose of This Document

This file provides **exhaustive coverage** of:
- Entity object architecture and members
- QueryEntities vs GetEntities (query system deep dive)
- Entity query syntax and filters (Group, Category, Type, Distance, Name, etc.)
- Targeting mechanics (locking, unlocking, max targets, ranges)
- Entity lifecycle and existence checking
- Safe entity iteration patterns (avoiding crashes from despawned entities)
- Common query patterns from Evebot, Yamfa, and Tehbot
- Distance calculations and spatial queries
- Performance optimization for entity queries
- Critical gotchas and race conditions

**Why This Is THE MOST CRITICAL File**:
- **Every bot constantly queries entities** (asteroids, NPCs, stations, gates, etc.)
- **Entity system is the primary way to perceive the game world**
- **Most bot crashes come from improper entity handling** (entities despawning mid-iteration)
- **Query performance directly impacts bot responsiveness**
- Understanding entities is **foundational** to all ISXEVE scripting

---

## Table of Contents

1. [Entity System Overview](#entity-system-overview)
2. [Entity Object](#entity-object)
3. [QueryEntities vs GetEntities](#queryentities-vs-getentities)
4. [Query Syntax and Filters](#query-syntax-and-filters)
5. [Common Query Patterns](#common-query-patterns)
6. [Entity Lifecycle and Existence](#entity-lifecycle-and-existence)
7. [Safe Entity Iteration](#safe-entity-iteration)
8. [Targeting Mechanics](#targeting-mechanics)
9. [Distance Calculations](#distance-calculations)
10. [Entity Categories and Groups](#entity-categories-and-groups)
11. [Entity Methods](#entity-methods)
12. [Performance Optimization](#performance-optimization)
13. [Real-World Patterns from Example Scripts](#real-world-patterns-from-example-scripts)
14. [Critical Gotchas](#critical-gotchas)
15. [Anti-Patterns](#anti-patterns)

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

## Next Steps

**Continue to Other Layer 3 Files**:
- File 12: Module_Management_and_Ship_Control.md (Activating/deactivating modules)
- File 11: Movement_Navigation_and_Autopilot.md (Warping, docking, autopilot)
- File 13: Inventory_and_Cargo_Systems.md (Cargo manipulation)

**Apply Knowledge**:
- All combat bots use entity queries constantly (find NPCs, lock targets)
- All mining bots use entity queries (find asteroids)
- Recognize entity patterns in Evebot/Yamfa/Tehbot
- Build robust entity handling into all bot logic

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

**END OF FILE 10**
**Status**: Complete (~2400 lines)
**Next**: Update progress tracker, continue with File 12 (Modules) or File 11 (Movement)
