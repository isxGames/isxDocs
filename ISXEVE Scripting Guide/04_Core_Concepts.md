# Core Concepts

**Purpose:** Essential ISXEVE concepts covering EVE game mechanics, InnerSpace platform, and ISXEVE extension architecture
**Audience:** Developers learning to build InnerSpace-based EVE automation

---

## Table of Contents

### EVE Online Game Mechanics
1. [Core Game Concepts](#core-game-concepts)
2. [Space and Movement](#space-and-movement)
3. [Ships and Modules](#ships-and-modules)
4. [Targeting and Combat](#targeting-and-combat)
5. [Cargo and Inventory](#cargo-and-inventory)
6. [Market and Economy](#market-and-economy)
7. [Mining and Resources](#mining-and-resources)
8. [NPCs and Anomalies](#npcs-and-anomalies)
9. [Stations and Structures](#stations-and-structures)
10. [Fleet Mechanics](#fleet-mechanics)
11. [Standings and Security](#standings-and-security)
12. [Critical Timing and Delays](#critical-timing-and-delays)

### InnerSpace Platform
13. [InnerSpace Platform Overview](#innerspace-platform-overview)
14. [What Is InnerSpace](#what-is-innerspace)
15. [Architecture and Components](#architecture-and-components)
16. [LavishScript Engine](#lavishscript-engine)
17. [Extension System](#extension-system)
18. [Sessions and Multi-Client](#sessions-and-multi-client)
19. [Uplink and Relay](#uplink-and-relay)
20. [Script Execution Model](#script-execution-model)
21. [Console and Commands](#console-and-commands)
22. [File System and Paths](#file-system-and-paths)
23. [Performance and Limitations (InnerSpace)](#performance-and-limitations)

### ISXEVE Extension
24. [ISXEVE Extension Architecture](#isxeve-extension-architecture)
25. [What Is ISXEVE](#what-is-isxeve)
26. [How ISXEVE Works](#how-isxeve-works)
27. [Core Object Hierarchy](#core-object-hierarchy)
28. [Object Types and TLOs](#object-types-and-tlos)
29. [Members vs Methods](#members-vs-methods)
30. [Entity System](#entity-system)
31. [UI Interaction](#ui-interaction)
32. [Event System](#event-system)
33. [Common Patterns](#common-patterns)
34. [Limitations and Constraints](#limitations-and-constraints)

---

## Core Game Concepts

### The Client-Server Architecture

**Critical Understanding**:
- EVE is a **server-authoritative** MMO
- Client only displays what server tells it
- **Cannot send fake commands** - server validates everything
- Bot automation works by **reading client state** and **issuing valid commands**

**Implications for Botting**:
- Must wait for server confirmation (actions not instant)
- Network latency affects timing (typically 50-300ms)
- Server can lag, causing delays (1-5 seconds possible)
- Must handle "action failed" responses gracefully

### The Game Loop

**Server Tick Rate**: 1 Hz (1 second per tick)
**Client Update Rate**: Variable, 60 FPS typical

**What this means**:
- Movement updates every ~1 second from server
- Visual client interpolates between updates
- Command responses take 0.1-2 seconds typically
- **Never assume instant response** - always verify state change

### Session States

**Possible States**:
1. **Logged Out** - No character in game
2. **Character Selection** - At character select screen
3. **In Station** - Docked at station/structure
4. **In Space** - Character in space (flying)
5. **Transitioning** - Undocking, jumping, warping

**Bot Must Handle All States**:
```lavishscript
; Check if we're in space before trying to target
if !${Me.InSpace}
{
    echo "Not in space - cannot execute space commands"
    return
}
```

---

## Space and Movement

### Solar Systems and Space

**Hierarchy**:
- **Universe** (single game world)
  - **Regions** (large areas, ~30-100 systems each)
    - **Constellations** (5-15 systems)
      - **Solar Systems** (the playable space units)

**Solar System Structure**:
- Central star (usually)
- Planets (can have moons)
- Asteroid belts
- Stargates (connect to other systems)
- Stations/Structures (player docking points)
- Anomalies (scan sites, combat sites)
- Empty space (vast distances)

**Coordinate System**:
- **3D Cartesian** coordinates (X, Y, Z)
- Measured in **meters**
- Distances can be **millions of kilometers** (astronomical units)
- Use `Entity.X`, `Entity.Y`, `Entity.Z` for positions

### Movement Types

#### 1. Normal Flight (Sub-warp)
**Speed**: 50 - 5,000 m/s depending on ship
**Use**: Close-range maneuvering, combat, mining
**Commands**:
- `Entity:Approach[]` - Fly straight to entity
- `Entity:Orbit[distance]` - Circle around entity
- `Entity:KeepAtRange[distance]` - Maintain distance
- `MyShip:StopShip[]` - Stop all movement

**Physics**:
- Ships have **mass** and **inertia**
- Takes time to accelerate/decelerate (align time)
- Angular velocity matters for combat (transversal)
- Propulsion modules increase max speed

**Critical Timing**:
```lavishscript
; Approaching takes time - must wait
Entity:Approach[]
wait 10  ; Wait 1 second
while ${Entity.Distance} > 2500
{
    wait 10
    ; Check if we're still approaching
    if !${MyShip.ToEntity.IsApproaching}
    {
        Entity:Approach[]
    }
}
```

#### 2. Warping (Inter-system Fast Travel)
**Speed**: 3 AU/s typical (millions of km/s)
**Use**: Travel between planets, gates, stations
**Requirements**:
- Ship must **align** (point toward destination)
- Align time = f(ship mass, inertia, skills)
- Typical align: 2-15 seconds
- Cannot warp if **scrambled** (enemy tackle)

**Warp Mechanics**:
```
Enter Warp Command
    ↓
Ship Aligns (2-15 seconds)
    ↓
Enters Warp (acceleration phase)
    ↓
Warp Tunnel (constant speed)
    ↓
Deceleration Phase
    ↓
Drops Out of Warp
```

**Bot Considerations**:
```lavishscript
; Warp to entity
Entity:WarpTo[0]
wait 30  ; Wait 3 seconds for align/warp entry

; Must wait until we drop out
while ${Me.ToEntity.Mode} == 3  ; Mode 3 = Warping
{
    wait 10
    ; Timeout check after 2 minutes
    if ${counter} > 1200
    {
        echo "Warp took too long - stuck?"
        break
    }
}
echo "Warp complete"
```

**Warp Distances**:
- `WarpTo[0]` - Warp to 0km (as close as possible)
- `WarpTo[100000]` - Warp to 100km
- Default is typically 15km for celestials

#### 3. Jumping (Stargate Travel)
**Speed**: Instant teleport (loading screen)
**Use**: Travel between solar systems
**Mechanics**:
- Must be within 2,500m of gate
- Activate gate
- Loading screen (2-10 seconds)
- Appear on other side (session change)
- **Session timer** prevents immediate jump back (30s - 10min)

**Critical Bot Code**:
```lavishscript
; Jump through gate
variable int RetryCount = 0
Stargate:Approach[]
wait 20

; Wait until in range
while ${Stargate.Distance} > 2500 && ${RetryCount} < 60
{
    wait 10
    RetryCount:Inc
}

if ${Stargate.Distance} <= 2500
{
    Stargate:Activate[]
    wait 20

    ; Wait for session change
    variable int SessionID = ${Session}
    while ${Session} == ${SessionID} && ${RetryCount} < 300
    {
        wait 10
        RetryCount:Inc
    }

    if ${Session} != ${SessionID}
    {
        echo "Jump successful - new session ${Session}"
        ; Wait for grid load
        wait 50
    }
}
```

#### 4. Docking/Undocking
**Docking**:
- Must be within docking range (varies by structure, typically 0-5000m)
- Instant transition to station interior
- **Session change** (similar to jumping)

**Undocking**:
- Instant appearance outside station
- **Invulnerability timer** (30 seconds) - cannot be targeted
- Ship appears ~1km from station undock point

**Bot Pattern**:
```lavishscript
; Dock at station
Station:Dock[]
wait 100  ; Wait 10 seconds for docking

; Verify we're in station
if ${Me.InStation}
{
    echo "Docked successfully"
}
else
{
    echo "Dock failed or timed out"
}

; Undock
EVE:Execute[CmdExitStation]
wait 100  ; Wait for undock

if ${Me.InSpace}
{
    echo "Undocked successfully"
    ; Still have invuln for ~30s
}
```

---

## Ships and Modules

### Ship Classes and Sizes

**Ship Size Categories** (affects various mechanics):
1. **Frigates** - Smallest, fastest, low HP (~5k-10k)
2. **Destroyers** - Small, high DPS, fragile
3. **Cruisers** - Medium, balanced, versatile (~30k-60k HP)
4. **Battlecruisers** - Large, tough, good damage
5. **Battleships** - Largest sub-capital, high DPS, tanky (~100k-200k HP)
6. **Capitals** - Titans, Supercarriers, etc. (special mechanics, rare for bots)

**Size Affects**:
- Warp speed (smaller = faster)
- Align time (smaller = quicker)
- Signature radius (smaller = harder to hit)
- Module fitting (each ship has CPU/powergrid limits)

### Module Types

**High Slot Modules** (offensive/utility):
- **Weapons**: Turrets (guns), Launchers (missiles), Drones
- **Mining Lasers**: Extract ore from asteroids
- **Salvagers**: Extract loot from wrecks
- **Cloaking Devices**: Make ship invisible (special mechanics)

**Medium Slot Modules** (navigation/ewar):
- **Propulsion**: Afterburner (AB), Microwarpdrive (MWD)
- **Shield Modules**: Shield extenders, boosters, hardeners
- **EWAR**: Target painters, sensor dampeners, etc.
- **Tackle**: Warp scramblers, disruptors, stasis webifiers

**Low Slot Modules** (defense/damage mods):
- **Armor**: Plates, repairers, hardeners
- **Damage Mods**: Increase weapon damage
- **Ship Modifications**: Inertia stabilizers, nanofibers, etc.

**Rig Slots** (permanent modifications):
- Cannot be removed without destroying
- Increase various ship attributes
- **Not activatable** - passive bonuses only

### Module States

**States**:
1. **Offline** - Not ready, cannot activate
2. **Online** - Ready to activate
3. **Active** - Currently running
4. **Overloaded** - Running at 150% with heat damage
5. **Deactivating** - Cycling down

**Cycling**:
- Most modules have **cycle time** (e.g., 5 seconds)
- Module activates → runs for cycle time → deactivates
- Some modules **auto-repeat** (guns, miners)
- Some are **one-shot** per cycle (armor repairers)

**Bot Code Pattern**:
```lavishscript
; Activate module
Module:Activate[]

; Check if it activated
wait 5  ; Small delay for server response
if ${Module.IsActive}
{
    echo "Module ${Module.ToItem.Name} is now active"
}
else
{
    echo "Module failed to activate"
}

; Some modules need to be kept active
while ${TargetExists} && ${Module.IsOnline}
{
    if !${Module.IsActive}
    {
        Module:Activate[]
        wait 5
    }
    wait 10  ; Check every second
}
```

### Capacitor (Ship Energy)

**Critical Resource**:
- Powers all active modules
- Regenerates over time (30s-10min to full)
- Running out = **cap empty** = modules deactivate
- Must manage carefully in combat

**Cap Calculations**:
- Each module has activation cost (e.g., 50 GJ)
- Cap recharge is **non-linear** (fastest at 25-30% cap)
- **Cap stable** = regeneration ≥ consumption

**Bot Must Monitor**:
```lavishscript
; Check capacitor before activating expensive module
if ${MyShip.Capacitor} > 500
{
    ; Safe to activate
    ExpensiveModule:Activate[]
}
else
{
    echo "Cap too low - waiting"
    ; Deactivate some modules to recover
    PropulsionModule:Deactivate[]
}

; Cap percentage
if ${MyShip.CapacitorPct} < 25
{
    echo "CRITICAL: Cap below 25% - emergency protocol"
}
```

---

## Targeting and Combat

### Targeting System

**Targeting Limits**:
- **Max Targets**: Determined by ship + skills (typically 5-10)
- **Lock Time**: 1-15 seconds depending on ship sensors, target signature
- **Max Range**: Ship sensor strength (typically 50-250km)

**Target Lock Process**:
```
Issue Lock Command
    ↓
Lock Timer Starts (1-15s)
    ↓
Target Locked (can now shoot/activate modules on it)
    ↓
Target Can Be Unlocked (instant)
```

**Bot Targeting Pattern**:
```lavishscript
; Lock target
variable int64 TargetID = ${NPC.ID}
NPC:LockTarget[]

; Wait for lock to complete
variable int LockTimeout = 0
while !${NPC.IsLockedTarget} && ${LockTimeout} < 150
{
    wait 10
    LockTimeout:Inc

    ; Check if target still exists
    if !${Entity[${TargetID}](exists)}
    {
        echo "Target disappeared while locking"
        return
    }
}

if ${NPC.IsLockedTarget}
{
    echo "Target locked - engaging"
    ; Now can activate weapons
}
else
{
    echo "Lock timed out or failed"
}
```

**Target Switching**:
- Bots often need to switch targets (primary dies, etc.)
- Must unlock old target before locking new (if at max targets)
- Locking has cooldown - cannot spam lock commands

### Weapon Systems

#### Turrets (Guns)
**Types**: Lasers, Projectiles (autocannons/artillery), Hybrids (blasters/railguns)

**Mechanics**:
- **Tracking**: Can gun track the target's angular velocity?
- **Optimal Range**: Distance where damage is best
- **Falloff**: Damage decreases beyond optimal
- **Damage Formula**: Complex, depends on tracking, range, signature

**Tracking Equation (Simplified)**:
```
Hit Chance = f(Angular Velocity, Signature Radius, Tracking Speed, Range)
```

**Angular Velocity**:
- **Critical Concept**: How fast target crosses your field of view
- Target moving toward/away = low angular (easy to hit)
- Target moving perpendicular = high angular (hard to hit)
- **Orbiting** creates angular velocity (reduces incoming damage)

**Bot Must Understand**:
```lavishscript
; For a battleship shooting a frigate:
; - Frigate orbiting at 500m/s at 5km range
; - Angular velocity = very high
; - Battleship tracking = poor
; - Result: Can't hit frigate
;
; Solution: Orbit at optimal range to create our own angular
MyShip.ActiveTarget:Orbit[20000]  ; Orbit at 20km
```

#### Missiles
**Types**: Rockets, Light/Heavy/Cruise missiles, Torpedoes

**Mechanics**:
- **No tracking** - always flies toward target
- **Explosion Velocity & Radius**: Determines damage application
- **Flight Time**: Missiles take time to reach target (1-15 seconds)
- **Damage**: Applied when missile impacts

**Key Difference from Turrets**:
- Don't need to track
- Damage based on target signature vs explosion radius
- Delayed damage (flight time)

**Bot Considerations**:
```lavishscript
; Missiles have flight time
Module[TypeID,Launcher]:Activate[]
; Missile launches immediately
wait 5

; But damage won't apply for several seconds
; Target might die from someone else's damage
; Missile will "miss" and deal 0 damage
;
; Can't stop missile once launched
; This is called "overkilling" - multiple missiles on dying target
```

#### Drones
**Special Mechanics**:
- **Separate entities** - drones are NPC-like objects in space
- Launch from **drone bay**
- Fly to target and orbit
- Must be commanded to engage

**Drone Bandwidth**:
- Each ship has bandwidth limit (e.g., 50 Mbit/s)
- Light drones = 5 Mbit, Medium = 10, Heavy = 25, etc.
- Cannot exceed bandwidth

**Drone Control**:
```lavishscript
; Launch drones
MyShip:LaunchAllDrones[]
wait 30  ; Wait for launch

; Verify drones in space
if ${Me.GetDrones} > 0
{
    ; Command drones to engage
    MyShip.ActiveTarget:EngageMyTarget[]
    wait 10
}

; Later: Return drones
EVE:Execute[CmdDronesReturnToBay]
wait 30  ; Wait for return
while ${Me.GetDrones} > 0
{
    wait 10
    ; Timeout after 30 seconds
}
```

**Drone AI**:
- Drones have simple AI
- Will chase target until target dies or recalled
- Can get "stuck" aggressing wrong targets
- **Must micromanage** in complex scenarios

### Damage Types and Resistances

**Four Damage Types**:
1. **EM** (Electromagnetic)
2. **Thermal**
3. **Kinetic**
4. **Explosive**

**Resistance System**:
- Each ship has resistances to each damage type
- Resistance = damage reduction percentage
- Example: 50% EM resistance = take half EM damage

**NPC Damage Patterns**:
- **Sansha**: EM/Thermal
- **Blood Raiders**: EM/Thermal
- **Guristas**: Kinetic/Thermal
- **Serpentis**: Kinetic/Thermal
- **Angel**: Explosive/Kinetic
- **Mercenary**: All types

**Bot Should Know**:
```lavishscript
; Example: Fighting Guristas (Kinetic/Thermal damage)
; Should fit Kinetic/Thermal hardeners
;
; Check NPC faction:
if ${NPC.Owner.Name.Find["Guristas"]}
{
    ; Activate kinetic hardener
    KineticHardener:Activate[]
}
```

### Aggression and Targeting Priority

**NPC Aggression**:
- NPCs usually aggro on **first to attack** or **closest**
- Switching aggro is complex (based on DPS, distance, ship size)
- **Trigger NPCs**: Some NPCs trigger spawns when killed

**Bot Targeting Priority**:
```lavishscript
; Typical priority order:
; 1. Frigates (fast, annoying, low HP)
; 2. Cruisers (medium threat)
; 3. Battleships (high HP, slow)
;
; OR prioritize by:
; - Closest (safest - reduces range to threats)
; - Lowest HP (kill faster)
; - Highest DPS (most dangerous)
;
; Example: Sort by distance
variable iterator Target
EVE:QueryEntities[Target, "(CategoryID = CATEGORY_ENTITY) && IsNPC && !IsMoribund"]
Target:GetTargets
if ${Target:First(exists)}
{
    ; Sort by distance
    ; (LavishScript doesn't have built-in sort, must implement)
}
```

---

## Cargo and Inventory

### Inventory System Structure

**Hierarchy**:
```
Character
├── Ship (Active Ship)
│   ├── Cargo Hold
│   ├── Ore Hold (mining ships)
│   ├── Drone Bay
│   ├── Fighter Bay (carriers)
│   └── Fuel Bay (some ships)
├── Items (on character - when in station)
├── Ship Hangar (other ships in station)
├── Item Hangar (items in station)
└── Corporation Hangars (if available)

Station/Structure
├── Item Hangar (personal)
├── Ship Hangar (personal)
├── Corporation Hangars
└── Corporation Ship Hangar
```

**Critical IDs**:
- Each inventory location has unique ID
- Must reference correct location when moving items
- Example: `${MyShip.ID}` for ship cargo

### Cargo Capacity

**Limits**:
- Each container has **maximum volume** (m³)
- Items have volume (e.g., Veldspar = 0.1 m³)
- Cannot exceed capacity

**Bot Must Check** (modern inventory API):
```lavishscript
; Check cargo space before looting
if ${EVEWindow[Inventory](exists)}
{
    variable float cargoFree = ${EVEWindow[Inventory].Child[ShipCargo].FreeSpace}

    if ${cargoFree} > 100
    {
        ; Safe to loot wreck
        Wreck:Open[]
        wait 20
        Wreck:LootAll[]
    }
    else
    {
        echo "Cargo full - cannot loot wreck"
        ; Return to station to unload
    }
}
```

### Item Movement

**Operations**:
1. **Open** - Open a container (station hangar, wreck, etc.)
2. **Move** - Move items between locations
3. **Stack** - Combine stacks of same item
4. **Split** - Split stack into smaller stacks

**Timing Critical**:
- Opening containers takes time (server delay)
- Moving items not instant
- Must wait for actions to complete

**Example**:
```lavishscript
; Move ore from ship cargo to station hangar
; MUST be docked for this

; Open cargo hold
MyShip:Open[]
wait 20  ; CRITICAL WAIT

; Get items of type "Veldspar"
variable index:item CargoItems
MyShip:GetCargo[CargoItems]

variable iterator CargoItem
CargoItems:GetIterator[CargoItem]

if ${CargoItem:First(exists)}
{
    do
    {
        if ${CargoItem.Value.Name.Equal["Veldspar"]}
        {
            ; Move to item hangar
            CargoItem.Value:MoveTo[${Me.Station.ID}, MyItemHangar]
            wait 5  ; Small delay per item
        }
    }
    while ${CargoItem:Next(exists)}
}

echo "Ore unloaded"
```

### Wrecks and Looting

**Wreck Mechanics**:
- NPCs leave wrecks when killed
- Wrecks contain loot (modules, ammo, salvage)
- Wrecks can be **salvaged** for materials
- Wrecks despawn after 2 hours

**Loot Rights**:
- Yellow wreck = can loot freely
- Blue wreck = owned by corpmate
- White wreck = abandoned (anyone can loot)
- Red wreck = stealing if you loot (suspect flag)

**Bot Looting**:
```lavishscript
; Find wrecks
variable index:entity Wrecks
variable iterator Wreck
EVE:QueryEntities[Wrecks, "GroupID = GROUP_WRECK"]

Wrecks:GetIterator[Wreck]
if ${Wreck:First(exists)}
{
    do
    {
        ; Approach wreck
        Wreck.Value:Approach[]
        while ${Wreck.Value.Distance} > 2500
        {
            wait 10
        }

        ; Open and loot
        Wreck.Value:Open[]
        wait 30  ; MUST WAIT for container to open

        Wreck.Value:LootAll[]
        wait 20
    }
    while ${Wreck:Next(exists)}
}
```

---

## Mining and Resources

### Asteroid Belts

**Structure**:
- Each solar system has 0-20+ asteroid belts
- Belts contain asteroids (ore/ice)
- Belts listed in Overview
- Persistent locations (always same spot)

**Asteroid Types**:
- Different ores (Veldspar, Scordite, Pyroxeres, etc.)
- Each ore has different value and minerals
- Higher-value ores in lower-security space

### Mining Mechanics

**Process**:
```
Lock Asteroid
    ↓
Activate Mining Laser
    ↓
Laser Cycles (60-180 seconds)
    ↓
Ore Added to Cargo (or Ore Hold)
    ↓
Repeat until asteroid depleted or cargo full
```

**Cycle Time**:
- Mining lasers typically 60-180 second cycles
- Cannot interrupt cycle without wasting it
- Must wait for full cycle to complete

**Bot Mining Loop** (modern inventory API):
```lavishscript
; Basic mining loop - check cargo space using modern API
function GetCargoFreeSpace()
{
    if ${EVEWindow[Inventory](exists)}
    {
        return ${EVEWindow[Inventory].Child[ShipCargo].FreeSpace}
    }
    return 0
}

while ${GetCargoFreeSpace} > 100
{
    ; Find asteroid
    variable index:entity Asteroids
    EVE:QueryEntities[Asteroids, "GroupID = GROUP_ASTEROID"]

    if ${Asteroids.Used} == 0
    {
        echo "No asteroids in belt"
        break
    }

    ; Get first asteroid
    variable entity Roid = ${Asteroids.Get[1]}

    ; Approach and lock
    Roid:Approach[]
    wait 10
    Roid:LockTarget[]
    wait 20

    ; Orbit at mining range
    Roid:Orbit[1000]
    wait 10

    ; Activate miners
    call ActivateMiningLasers

    ; Wait for cargo to fill or asteroid to deplete
    while ${GetCargoFreeSpace} > 100 && ${Roid(exists)}
    {
        wait 50  ; Check every 5 seconds

        ; Check if lasers still active
        if !${MiningLaser.IsActive}
        {
            ; Asteroid depleted or laser deactivated
            echo "Laser inactive - switching target"
            break
        }
    }

    ; Deactivate lasers
    call DeactivateMiningLasers
    Roid:UnlockTarget[]
}

echo "Cargo full - returning to station"
```

### Ice Mining

**Different from Ore**:
- Ice asteroids are **huge** (take many minutes to deplete)
- Ice takes more cargo space
- Ice mining lasers are separate modules
- Ice belts **respawn** on a 4-hour timer

**Special Considerations**:
- Typically mine same ice rock for extended time
- Less target switching than ore mining
- Higher value but slower

---

## NPCs and Anomalies

### NPC Types

**Categories**:
1. **Belt Rats** - Spawn in asteroid belts randomly
2. **Mission Rats** - Spawn in mission sites (mission-specific)
3. **Anomaly Rats** - Spawn in combat anomalies
4. **Roaming Rats** - Wander solar system

**NPC Behavior**:
- Aggro on players (attack)
- Can warp away if low HP (rare)
- Drop loot when killed
- Respawn on timers

### Combat Anomalies

**What Are They**:
- Scannable (or combat scanner probe) sites
- Contain waves of NPCs
- Clear all waves = site complete
- Reward: Bounties + loot

**Anomaly Types**:
- **Havens, Sanctums** (high-end)
- **Hubs, Rallies** (medium)
- **Hideaways, Forsaken** (lower)

**Wave Mechanics**:
```
Warp to Anomaly
    ↓
Initial Wave Spawns (5-15 NPCs)
    ↓
Kill All NPCs in Wave
    ↓
Next Wave Spawns (30-60 second delay)
    ↓
Repeat until all waves clear
    ↓
Anomaly Despawns
    ↓
Loot Wrecks
```

**Bot Must Wait for Waves**:
```lavishscript
; After killing all rats, wait for next wave
variable int WaveWaitCounter = 0
while ${WaveWaitCounter} < 40  ; Wait 40 seconds
{
    ; Check if new NPCs spawned
    variable index:entity NPCs
    EVE:QueryEntities[NPCs, "IsNPC && !IsMoribund"]

    if ${NPCs.Used} > 0
    {
        echo "New wave spawned - engaging"
        break
    }

    wait 10
    WaveWaitCounter:Inc
}

if ${NPCs.Used} == 0
{
    echo "No new wave - anomaly complete"
    ; Loot wrecks and warp out
}
```

### Bounties

**Bounty System**:
- Killing NPCs pays ISK bounty
- Bounty credited instantly to wallet
- Different NPC types have different bounties
- Primary income for combat bots

**Bot Tracking** (Optional):
```lavishscript
; Track wallet balance before/after
variable int64 StartingBalance = ${Me.Wallet}

; Run anomaly...

variable int64 EndingBalance = ${Me.Wallet}
variable int64 Profit = ${Math.Calc[${EndingBalance} - ${StartingBalance}]}
echo "Earned ${Profit} ISK in bounties"
```

---

## Stations and Structures

### Station Types

**NPC Stations**:
- Permanent, cannot be destroyed
- Safe havens
- Market hubs
- Instant docking

**Player Structures** (Citadels, etc.):
- Can be destroyed
- May have docking restrictions
- May deny docking

**Bot Must Check Access**:
```lavishscript
; Before warping to structure
if !${Structure.HasDockingAccess}
{
    echo "Cannot dock at this structure - denied"
    ; Find alternative station
}
```

### Station Services

**Available Services**:
- **Repair** - Fix ship damage
- **Reprocessing** - Turn ore into minerals
- **Market** - Buy/sell items
- **Contracts** - Player-to-player trades
- **Fitting** - Change ship modules
- **Insurance** - Insure ship

**Bot Use Cases**:
- Unload ore → sell on market
- Repair ship after combat
- Restock ammo from market
- Change to different ship

---

## Fleet Mechanics

### Fleet Structure

**Hierarchy**:
```
Fleet Commander (FC)
├── Wing Commander 1
│   ├── Squad Commander 1
│   │   ├── Member 1
│   │   ├── Member 2
│   │   └── Member 3
│   └── Squad Commander 2
└── Wing Commander 2
```

**Roles**:
- **Fleet Commander**: Top authority
- **Wing Commander**: Manages wing
- **Squad Commander**: Manages squad (up to 10 members)
- **Member**: Regular fleet member

### Fleet Broadcasts

**Broadcast System**:
- **Need Shield** - Request shield reps
- **Need Armor** - Request armor reps
- **Need Capacitor** - Request cap
- **In Position** - Signaling ready
- **Enemy Spotted** - Alert fleet
- **Target** - Primary target broadcast

**Bot Can Monitor**:
```lavishscript
; Check fleet broadcasts
; Note: ISXEVE API for this may be limited
; Typically need relay system for fleet coordination
```

### Fleet Benefits

**Bonuses**:
- Shared Overview (see fleet members' targets)
- Command bursts (fleet bonuses)
- Coordinated warps
- Shared bookmarks (corp/alliance)

**Multi-Boxing Fleets**:
- Common for bot fleets
- Master bot commands, slaves follow
- **Relay system** coordinates actions (see Yamfa example)

---

## Standings and Security

### Standing System

**Standing Values**: -10.0 to +10.0

**Standing Types**:
1. **Personal** - Your standing to entity
2. **Corporation** - Your corp's standing
3. **Alliance** - Your alliance's standing
4. **Entity to You** - Their standing to you

**Effects**:
- Negative standings = hostile (will attack)
- Positive standings = friendly
- Security status affects high-sec access

### Security Status

**System Security**:
- **1.0 to 0.5** - High security (CONCORD protection)
- **0.4 to 0.1** - Low security (limited protection)
- **0.0 and below** - Null security (no protection)

**CONCORD**:
- NPC police in high-sec
- Kills criminals (players who attack illegally)
- Response time: Instant in 1.0, slower in 0.5

**Bot Considerations**:
- High-sec safest but lower rewards
- Low/null-sec dangerous but profitable
- Must handle PvP threats in low/null

---

## Critical Timing and Delays

### Server Response Times

**Typical Delays**:
- Command → Server Response: **100-500ms**
- Container Open: **500-2000ms**
- Lock Target: **1000-15000ms**
- Module Activate: **100-500ms**
- Dock/Undock: **2000-10000ms**
- Jump Gate: **2000-10000ms**

### Wait Patterns

**Anti-Pattern** (Bad):
```lavishscript
; DON'T DO THIS - Too fast, doesn't wait for server
Entity:LockTarget[]
Module:Activate[]  ; Tries to activate before lock completes - FAILS
```

**Correct Pattern**:
```lavishscript
; DO THIS - Wait for lock
Entity:LockTarget[]
wait 10  ; Initial delay
while !${Entity.IsLockedTarget}
{
    wait 10
}
; Now locked - can activate
Module:Activate[]
```

**Timeout Pattern**:
```lavishscript
; BEST PRACTICE - Wait with timeout
Entity:LockTarget[]
variable int Timeout = 0
while !${Entity.IsLockedTarget} && ${Timeout} < 150
{
    wait 10
    Timeout:Inc
}

if ${Timeout} >= 150
{
    echo "Lock timed out - target may be out of range"
    return
}
```

### Pulse/Loop Timing

**Recommended**:
- Main loop: **1-2 second pulse** (wait 10-20)
- Combat loop: **0.5-1 second pulse** (wait 5-10)
- UI checks: **2-5 second pulse** (wait 20-50)

**Why Not Faster**:
- Server updates every ~1 second anyway
- Faster loops waste CPU
- Can trigger anti-bot detection (spam)

**Example Main Loop**:
```lavishscript
while ${ScriptRunning}
{
    ; State machine or decision logic here
    call ProcessCombat
    call ProcessMining
    call CheckThreats

    ; Pulse delay
    wait 20  ; 2 seconds
}
```

---

## Summary for Bot Developers

**Critical Concepts**:
1. **Server Authority** - Cannot fake actions, must wait for responses
2. **Timing** - Everything takes time, must wait appropriately
3. **State Management** - Track ship state, space state, module states
4. **Entity System** - Everything is an entity (ships, NPCs, asteroids)
5. **Coordinates** - 3D space, distances in meters
6. **Targeting** - Must lock before shooting, limited target slots
7. **Capacitor** - Energy resource, must manage carefully
8. **Cargo** - Limited space, must return to station
9. **Modules** - Activation states, cycle times, cap costs
10. **NPCs** - Respawn timers, aggression mechanics, loot

---

## InnerSpace Platform Overview

**The Foundation for Game Automation**

---

## What Is InnerSpace

### Core Purpose

**InnerSpace** is a **game automation platform** that:
- Hosts multiple game clients simultaneously
- Provides scripting engine (LavishScript)
- Enables memory reading from game processes
- Allows extensions to add game-specific APIs
- Facilitates inter-process communication (IPC)

**Critical Understanding**:
- InnerSpace is **NOT** a cheat engine or memory editor
- It is **NOT** injecting fake data into the game
- It **IS** reading game memory and issuing valid commands
- It **IS** a legitimate (though ToS-gray) automation tool

### The Ecosystem

```
InnerSpace (Host Platform)
    ↓
LavishScript (Scripting Engine)
    ↓
ISXEVE (EVE Online Extension)
    ↓
Your Bot Script (.iss file)
    ↓
EVE Online Client (Game)
```

**Flow of Control**:
1. InnerSpace launches EVE client
2. InnerSpace injects itself into EVE process
3. ISXEVE extension loads, reads EVE memory
4. Your script runs in LavishScript engine
5. Script uses ISXEVE to read game state
6. Script issues commands to EVE UI
7. EVE client sends commands to server

---

## Architecture and Components

### InnerSpace Components

**Core Modules**:
1. **Inner Space.exe** - Main executable, process manager
2. **LavishScript.dll** - Scripting engine
3. **Extensions/** - Game-specific plugins (ISXEVE, etc.)
4. **Scripts/** - User scripts and libraries
5. **Uplink/** - IPC server for relay communication

**Directory Structure**:
```
InnerSpace/
├── Inner Space.exe           # Main program
├── ISXStealth.dll            # Stealth module (hides from detection)
├── Extensions/
│   ├── ISXEVE.dll            # EVE Online extension
│   ├── ISXStealth.dll        # Anti-detection
│   └── [other game extensions]
├── Scripts/
│   ├── EVEBot/               # Evebot script
│   ├── Yamfa/                # Your fleet assist script
│   ├── Common/               # Shared libraries
│   └── [your scripts]
├── Uplink/                   # Relay server files
└── Settings/                 # Configuration files
```

### Process Model

**Single InnerSpace, Multiple Games**:
```
InnerSpace.exe (Parent Process)
    ├── eve.exe (Session 1 - Character A)
    ├── eve.exe (Session 2 - Character B)
    └── eve.exe (Session 3 - Character C)
```

**Critical Concepts**:
- Each EVE client = separate process
- Each process = separate "session" in InnerSpace
- InnerSpace coordinates all sessions
- Sessions can communicate via Relay

---

## LavishScript Engine

### What Is LavishScript

**LavishScript** is:
- **Scripting language** embedded in InnerSpace
- **Interpreted** (not compiled)
- **Game-agnostic** core with **game-specific extensions**
- **Event-driven** and **imperative**

**Language Characteristics**:
- Similar to C-style syntax (braces, semicolons)
- Dynamic typing (variables don't have fixed types)
- Object-oriented features (objects, methods)
- NO classes (objects are created via extensions or atoms)

**Example**:
```lavishscript
; This is a comment
variable int MyNumber = 42
variable string MyText = "Hello"

if ${MyNumber} > 40
{
    echo "Number is greater than 40"
}
```

### Script File Types

**File Extensions**:
- **.iss** - InnerSpace Script (main script files)
- **.xml** - Configuration files (settings, GUIs)

**Script Structure**:
```lavishscript
; Header: Check dependencies
#if !${ISXEVE(exists)}
    echo "ISXEVE extension required"
    Script:End
#endif

; Variables
variable bool Running = TRUE
variable int Counter = 0

; Functions
function DoSomething()
{
    echo "Doing something"
}

; Main entry point
function main()
{
    echo "Script starting"

    while ${Running}
    {
        call DoSomething
        wait 10
        Counter:Inc
    }

    echo "Script ending"
}
```

### Variable System

**Variable Declaration**:
```lavishscript
; Local variable
variable int LocalVar = 10

; Global variable (accessible across script scopes)
variable(global) int GlobalVar = 20

; Persistent variable (survives script reload)
variable(globalkeep) string PersistentVar = "Saved"

; Script variable (visible to other scripts via Script[scriptname])
variable(script) bool ScriptVar = TRUE
```

**Scope Rules**:
- Local = function/block scope
- Global = entire script scope
- Globalkeep = persists across script reloads
- Script = accessible via relay/uplink

### Data Types

**Primitive Types**:
```lavishscript
variable bool MyBool = TRUE              ; Boolean
variable int MyInt = 42                  ; 32-bit integer
variable int64 MyBigInt = 123456789      ; 64-bit integer
variable float MyFloat = 3.14            ; Floating point
variable string MyString = "Hello"       ; String
```

**Complex Types**:
```lavishscript
; Index (collection/array)
variable index:int MyIndex
MyIndex:Insert[10]
MyIndex:Insert[20]
echo "First element: ${MyIndex.Get[1]}"  ; 1-indexed!

; Iterator (for traversing collections)
variable iterator MyIterator
MyIndex:GetIterator[MyIterator]

; Point3f (3D coordinate)
variable point3f MyPoint
MyPoint:Set[100, 200, 300]

; Time (timestamp)
variable time MyTime = ${Time.Timestamp}
```

### Object System

**Everything Is An Object**:
```lavishscript
; Numbers are objects with methods
variable int MyNumber = 42
echo "Incremented: ${MyNumber.Inc}"  ; Returns 43

; Strings are objects
variable string MyString = "hello world"
echo "Uppercase: ${MyString.Upper}"  ; "HELLO WORLD"
echo "Length: ${MyString.Length}"    ; 11

; Even TRUE/FALSE are objects
variable bool MyBool = TRUE
echo "Negated: ${MyBool.Not}"  ; FALSE
```

**Object Members**:
- **Properties**: Values retrieved from object (e.g., `${MyString.Length}`)
- **Methods**: Actions performed by object (e.g., `MyString:Set["new value"]`)
- **Indices**: Access by index/key (e.g., `${MyIndex.Get[1]}`)

**Syntax**:
- **Get value**: `${Object.Property}` or `${Object.Method}`
- **Set value**: `Object:Method[parameters]`
- **Method call**: `Object:Method[]` (empty brackets for no params)

---

## Extension System

### What Are Extensions

**Extensions** are DLLs that:
- Add new object types to LavishScript
- Provide game-specific APIs
- Read game memory
- Expose game data as LavishScript objects

**ISXEVE Extension**:
- THE critical extension for EVE bots
- Provides 100+ object types
- Reads EVE client memory
- Exposes: ships, modules, entities, UI, etc.

**Loading Extensions**:
```lavishscript
; Check if extension loaded
#if ${ISXEVE(exists)}
    echo "ISXEVE is loaded"
#else
    echo "ISXEVE not loaded - exiting"
    Script:End
#endif
```

**Extension-Provided Objects**:
```lavishscript
; These objects come from ISXEVE extension:
${Me}                 ; Your character
${MyShip}             ; Your ship
${EVE}                ; EVE client object
${Entity[ID]}         ; Entities in space
${Station}            ; Current station
```

### Extension vs Script

**What Extensions Do**:
- Read memory directly
- Provide C++-speed performance
- Expose complex game structures
- Handle memory layout changes (patches)

**What Scripts Do**:
- Use extension-provided objects
- Implement bot logic
- Make decisions
- Issue commands

**Division of Labor**:
```
Extension (ISXEVE.dll)
    ↓ Provides Objects ↓
Script (YourBot.iss)
    ↓ Uses Objects ↓
Game (eve.exe)
```

---

## Sessions and Multi-Client

### Session Concept

**What Is A Session**:
- A **session** = one game client instance
- Each session has unique ID
- Sessions are numbered: is1, is2, is3, etc.
- Current session ID: `${Session}`

**Session Names**:
```lavishscript
; Current session
echo "Running in session ${Session}"

; Check if running in specific session
if ${Session.Equal["is1"]}
{
    echo "This is the master session"
}
```

### Multi-Boxing

**Common Pattern**:
- **Master** session controls logic
- **Slave** sessions follow master
- Communication via **Relay** system

**Example Setup**:
```
Session is1 (Master)
    - Makes targeting decisions
    - Broadcasts targets to slaves

Session is2 (Slave)
    - Listens for target broadcasts
    - Locks and shoots targets

Session is3 (Slave)
    - Listens for target broadcasts
    - Locks and shoots targets
```

**Session-Specific Scripts**:
```lavishscript
; Different behavior per session
if ${Session.Equal["is1"]}
{
    ; Master logic
    call MasterBehavior
}
else
{
    ; Slave logic
    call SlaveBehavior
}
```

---

## Uplink and Relay

### Uplink Service

**What Is Uplink**:
- **IPC server** built into InnerSpace
- Allows scripts to communicate across sessions
- Runs locally on your PC
- NOT internet-connected by default

**Relay = Uplink Communication**:
- "Relay" refers to sending data via Uplink
- Scripts can "relay" commands/data to each other
- Real-time, low-latency

### Relay System

**How It Works**:
```
Session is1 (Sender)
    ↓
    relay "all other" MyAtom "param1" "param2"
    ↓
Uplink Server
    ↓
Session is2, is3, is4 (Receivers)
    ↓
    Execute MyAtom function with parameters
```

**Relay Syntax**:
```lavishscript
; In sender script:
relay "all other" FunctionName "arg1" "arg2"

; This calls FunctionName in ALL other sessions
; With arguments "arg1" and "arg2"

; Specific session:
relay "is2" FunctionName "arg1"

; All except current:
relay "all other" FunctionName

; All including current:
relay "all" FunctionName
```

**Relay Receiver** (Atom):
```lavishscript
; Define an atom to receive relay
atom(script) FunctionName(string arg1, string arg2)
{
    echo "Received relay: ${arg1}, ${arg2}"
    ; Do something with the data
}
```

### Typical Relay Use Case: Fleet Assist

**Yamfa Example** (simplified):
```lavishscript
; MASTER session (is1):
function BroadcastTargets()
{
    ; Get my locked targets
    variable string TargetList = ""
    ; ... build target ID list ...

    ; Relay to all slaves
    relay "all other" ReceiveTargets "${TargetList}"
}

; SLAVE sessions (is2, is3, etc.):
atom(script) ReceiveTargets(string targetIDs)
{
    echo "Master says lock these targets: ${targetIDs}"
    ; Parse target IDs and lock them
}
```

**Why This Works**:
- Master makes targeting decisions (complex logic)
- Slaves just lock what master says (simple logic)
- No need for external communication (Redis, files, etc.)
- Real-time, same-machine communication

---

## Script Execution Model

### Running Scripts

**Launch Methods**:

**1. Console Command**:
```
run MyScript
```
- Opens console
- Types `run MyScript`
- Script executes

**2. Autostart**:
- Configure in InnerSpace settings
- Script auto-runs when session starts
- Good for production bots

**3. From Another Script**:
```lavishscript
; Start another script
Script[OtherScript]:Start[]

; End another script
Script[OtherScript]:End[]
```

### Script Lifecycle

**Execution Flow**:
```
Script Launched
    ↓
Global Variables Initialized
    ↓
main() Function Called (if exists)
    ↓
Script Runs (main loop, events, etc.)
    ↓
Script:End called OR main() returns
    ↓
Script Terminates
```

**Important**:
- **NO AUTOMATIC MAIN LOOP** - You create it!
- Script ends when it reaches end of file (unless in infinite loop)
- Must use `wait` commands to avoid CPU lockup

**Typical Script Structure**:
```lavishscript
; Init
function main()
{
    echo "Starting bot"
    call Initialize

    ; Main loop
    variable bool Running = TRUE
    while ${Running}
    {
        call Pulse
        wait 20  ; CRITICAL - prevents CPU maxing
    }

    call Shutdown
    echo "Bot ended"
}
```

### The Wait Command

**Critical Command**: `wait`

**Syntax**:
```lavishscript
wait <deciseconds>
```
- **decisecond** = 1/10 of a second = 100ms
- `wait 10` = wait 1 second
- `wait 5` = wait 0.5 seconds

**Why Wait Is Mandatory**:
```lavishscript
; BAD - NEVER DO THIS
while TRUE
{
    ; No wait - script runs MILLIONS of times per second
    ; Maxes CPU, crashes InnerSpace
}

; GOOD - ALWAYS DO THIS
while TRUE
{
    ; Process logic
    wait 10  ; Waits 1 second each loop iteration
}
```

**Typical Wait Times**:
- Main loop: `wait 10-20` (1-2 seconds)
- Combat loop: `wait 5-10` (0.5-1 seconds)
- Waiting for action: `wait 30-100` (3-10 seconds)

---

## Console and Commands

### InnerSpace Console

**What Is It**:
- Text console built into InnerSpace
- Appears as overlay in game client
- Receive script output (`echo` commands)
- Issue commands manually

**Opening Console**:
- Default hotkey: **Ctrl+Shift+I** (varies by config)
- Console appears over game

**Console Commands**:
```
run ScriptName           ; Start a script
end ScriptName           ; Stop a script
runscript path/to/file   ; Run script from path
echo Hello World         ; Print to console
ext                      ; List loaded extensions
relay all other uplink   ; Test relay communication
```

### Echo Command

**Output to Console**:
```lavishscript
echo "Hello from script"
echo "My ship: ${MyShip.ToEntity.Name}"

; Concatenation
variable int Value = 42
echo "The value is: ${Value}"
```

**Echo Is Your Debug Tool**:
```lavishscript
; Debug current state
echo "DEBUG: Current HP = ${MyShip.Shield}"
echo "DEBUG: Target locked? ${Target.IsLockedTarget}"
```

### Script Object

**Control Scripts**:
```lavishscript
; End current script
Script:End

; Pause script
Script:Pause

; Resume script
Script:Resume

; Check if another script running
if ${Script[EVEBot](exists)}
{
    echo "EVEBot is running"
}
```

---

## File System and Paths

### Script Paths

**Default Script Directory**:
```
InnerSpace/Scripts/
```

**Running Scripts**:
```lavishscript
; If file is at: InnerSpace/Scripts/MyBot/MyBot.iss
; Run with:
run MyBot/MyBot

; Or:
runscript MyBot/MyBot.iss
```

**Including Other Files**:
```lavishscript
; Include another script file
#include MyBot/Library/Targeting.iss
#include Common/Utilities.iss

; Now can use functions from those files
```

### File Operations

**Reading Files**:
```lavishscript
; Open file for reading
variable file MyFile = "Data/config.txt"
MyFile:Open[]

; Read line
variable string Line
if ${MyFile:Read[Line]}
{
    echo "Read: ${Line}"
}

MyFile:Close[]
```

**Writing Files**:
```lavishscript
; Open file for writing
variable file LogFile = "Logs/botlog.txt"
LogFile:Open[append]  ; Append mode

; Write line
LogFile:Write["Bot started at ${Time}\n"]

LogFile:Close[]
```

### Settings Files (XML)

**Persistent Configuration**:
```lavishscript
; Create settings object
variable(globalkeep) settingsetref MySettings
MySettings:Set[MyBot.Config, XML]

; Add settings
MySettings:Add[MasterCharacter, "CharacterName"]
MySettings:Add[FollowDistance, 1000]

; Save to file
MySettings:Save["Scripts/MyBot/config.xml"]

; Later: Load from file
MySettings:Load["Scripts/MyBot/config.xml"]

; Retrieve values
variable string Master = "${MySettings.Get[MasterCharacter]}"
```

---

## Performance and Limitations

### CPU and Memory

**Script Performance**:
- LavishScript is **interpreted** - slower than compiled C++/C#
- Extensions are **compiled C++** - fast
- Heavy computation should use extension features when possible

**Avoid**:
```lavishscript
; BAD - Querying thousands of entities every frame
while ${Running}
{
    variable index:entity AllEntities
    EVE:QueryEntities[AllEntities]  ; Expensive!
    ; Process...
    wait 5  ; Running 2x per second - too fast for this query
}
```

**Better**:
```lavishscript
; GOOD - Query at reasonable interval
while ${Running}
{
    variable index:entity AllEntities
    EVE:QueryEntities[AllEntities]  ; Expensive!
    ; Process...
    wait 20  ; Running every 2 seconds - reasonable
}
```

---

## Critical Concepts Summary

**Key Understandings**:

1. **InnerSpace = Platform**
   - Hosts game clients
   - Provides scripting engine
   - Enables IPC via relay

2. **LavishScript = Language**
   - Interpreted scripting language
   - Object-oriented
   - Dynamic typing
   - Extension-based

3. **ISXEVE = Extension**
   - Game-specific API
   - Reads EVE memory
   - Exposes game objects to scripts

4. **Sessions = Instances**
   - Each game client = one session
   - Sessions can communicate via relay
   - Multi-boxing uses multiple sessions

5. **Wait = Mandatory**
   - Must use `wait` in loops
   - Prevents CPU maxing
   - Accounts for server latency

6. **Relay = IPC**
   - Inter-script communication
   - Real-time, same-machine
   - Critical for fleet coordination

7. **Scripts vs Extensions**
   - Scripts = logic (your code)
   - Extensions = API (game access)
   - Extensions are fast, scripts are flexible

8. **Execution Model**
   - No automatic main loop
   - You create the loop
   - Script ends when code ends

---

## ISXEVE Extension Architecture

**The Bridge Between Scripts and EVE Online**

---

## What Is ISXEVE

### Definition

**ISXEVE** = **I**nner**S**pace E**X**tension for **EVE** Online

**Purpose**:
- Reads EVE client memory
- Exposes game data as LavishScript objects
- Provides methods to interact with game
- Handles EVE-specific logic

**NOT ISXEVE**:
- Does NOT inject fake data into game
- Does NOT modify game memory (read-only)
- Does NOT send network packets
- Does NOT bypass server authority

**What ISXEVE Actually Does**:
```
EVE Client Memory (Game State)
    ↓ ISXEVE Reads ↓
LavishScript Objects (Bot-Accessible Data)
    ↓ Bot Script Logic ↓
EVE UI Commands (Mouse/Keyboard Simulation)
    ↓ EVE Client Processes ↓
Server Commands (Normal Game Protocol)
```

### Why ISXEVE Is Necessary

**Without ISXEVE**:
- Would need to OCR (read pixels) to see game state
- Would need complex image recognition
- Would be extremely slow and fragile
- Would be human-level reaction times only

**With ISXEVE**:
- Instant access to all game data
- Precise coordinates, IDs, values
- Millisecond reaction times
- Structured, reliable data

**Example Comparison**:
```lavishscript
; WITHOUT ISXEVE (impossible in LavishScript):
; 1. Take screenshot
; 2. Run image recognition on shield bar
; 3. Estimate shield % from pixel colors
; 4. Hope it's accurate
;
; WITH ISXEVE (actual code):
variable float ShieldPct = ${MyShip.ShieldPct}
; Instant, precise, reliable
```

---

## How ISXEVE Works

### Memory Reading

**The Process**:
1. EVE Client stores game state in memory (C++ objects)
2. ISXEVE knows memory layouts (reverse-engineered)
3. ISXEVE reads memory addresses
4. ISXEVE creates LavishScript objects from memory data
5. Your script accesses these objects

**Memory Offsets**:
```
EVE Memory Address 0x12345678:
    ShipID: 1000000001234
    ShipName: "Dominix"
    ShieldHP: 8542.5
    ArmorHP: 12000.0
    ...

ISXEVE Reads This:
    ${MyShip.ID} = 1000000001234
    ${MyShip.ToEntity.Name} = "Dominix"
    ${MyShip.Shield} = 8542.5
    ${MyShip.Armor} = 12000.0
```

**Patch Resistance**:
- CCP patches EVE regularly
- Memory layouts change
- ISXEVE must be updated for each patch
- **Your scripts don't need updates** (ISXEVE abstracts the changes)

### Object Creation

**ISXEVE Registers Object Types**:
```cpp
// In ISXEVE C++ code (simplified):
RegisterTopLevelObject("Me", &GetMeObject);
RegisterTopLevelObject("MyShip", &GetMyShipObject);
RegisterTopLevelObject("EVE", &GetEVEObject);
```

**LavishScript Can Then Use**:
```lavishscript
; These "magically" exist because ISXEVE registered them
${Me}        ; Character object
${MyShip}    ; Ship object
${EVE}       ; EVE client object
```

### Update Cycle

**ISXEVE Updates**:
- ISXEVE reads memory every frame (60+ times per second)
- Caches data to reduce memory reads
- Invalidates cache when things change

**Script Access**:
```lavishscript
; When you access ${MyShip.Shield}:
; 1. ISXEVE checks cache (is it still valid?)
; 2. If cache invalid, read memory
; 3. If cache valid, return cached value
; 4. Return value to script

; This happens in microseconds
```

---

## Core Object Hierarchy

### Top-Level Objects (TLOs)

**What Are TLOs**:
- **Top-Level Objects** = objects accessible directly (not through another object)
- Start with `${}` syntax
- Provided by ISXEVE extension

**Most Important TLOs**:

```lavishscript
${Me}                   ; Your character
${MyShip}               ; Your ship
${EVE}                  ; EVE client/game state
${Entity[ID]}           ; Entity by ID
${Entity[Name]}         ; Entity by name
${Station}              ; Current station (if docked)
${Local}                ; Local chat
${Config}               ; ISXEVE configuration
${ISXEVE}               ; ISXEVE extension object
```

**Usage Examples**:
```lavishscript
; Character name
echo "${Me.Name}"

; Ship name
echo "${MyShip.ToEntity.Name}"

; Current solar system
echo "${Me.SolarSystemID}"

; Capacitor percentage
echo "${MyShip.CapacitorPct}%"
```

### Object Chaining

**Members Return Objects**:
```lavishscript
; ${Me} is an object
; ${Me.ToEntity} returns an entity object
; ${Me.ToEntity.Name} returns a string object

variable entity MyEntity = ${Me.ToEntity}
variable string MyName = ${MyEntity.Name}

; Or chain:
variable string MyName = ${Me.ToEntity.Name}
```

**Deep Chains**:
```lavishscript
; Get active target's owner's alliance name
${MyShip.ActiveTarget.ToEntity.Owner.Alliance.Name}

; Breakdown:
; MyShip                   -> ship object
; .ActiveTarget            -> module object
; .ToEntity                -> entity object
; .Owner                   -> owner object
; .Alliance                -> alliance object
; .Name                    -> string
```

### Type System

**Every Object Has A Type**:
```lavishscript
; Check type
echo "${Me.Type}"           ; Outputs: "me"
echo "${MyShip.Type}"       ; Outputs: "ship"
echo "${Entity[123].Type}"  ; Outputs: "entity"

; Type checking
if ${MyShip.Type.Equal["ship"]}
{
    echo "Yes, it's a ship"
}
```

**Common Types**:
- `me` - Character
- `ship` - Ship
- `entity` - Entity in space
- `item` - Inventory item
- `module` - Ship module
- `bookmark` - Bookmark
- `agent` - Agent (mission)
- `window` - UI window

---

## Object Types and TLOs

### The "me" Object

**${Me}** = Your character

**Key Members**:
```lavishscript
${Me.Name}              ; Character name (string)
${Me.CharID}            ; Character ID (int64)
${Me.CorporationID}     ; Corp ID (int)
${Me.AllianceID}        ; Alliance ID (int)
${Me.InSpace}           ; In space? (bool)
${Me.InStation}         ; In station? (bool)
${Me.SolarSystemID}     ; Current system ID (int)
${Me.ToEntity}          ; Character as entity (entity)
${Me.Wallet}            ; ISK balance (int64)
```

**Methods**:
```lavishscript
Me.GetTargets           ; Get index of locked targets
Me.GetFleetMembers      ; Get index of fleet members
```

**Example Usage**:
```lavishscript
if ${Me.InSpace}
{
    echo "${Me.Name} is in space in system ${Me.SolarSystemID}"
}
else if ${Me.InStation}
{
    echo "${Me.Name} is docked"
}
```

### The "ship" Object

**${MyShip}** = Your current ship

**Critical Members**:
```lavishscript
${MyShip.ID}                    ; Ship entity ID (int64)
${MyShip.ToEntity}              ; Ship as entity (entity)
${MyShip.Shield}                ; Current shield HP (float)
${MyShip.ShieldPct}             ; Shield % (float)
${MyShip.Armor}                 ; Current armor HP (float)
${MyShip.ArmorPct}              ; Armor % (float)
${MyShip.Structure}             ; Current structure HP (float)
${MyShip.StructurePct}          ; Structure % (float)
${MyShip.Capacitor}             ; Current cap (float)
${MyShip.CapacitorPct}          ; Cap % (float)
${MyShip.MaxTargetRange}        ; Max targeting range (float)
${MyShip.MaxLockedTargets}      ; Max lockable targets (int)
```

**Cargo Access** (⚠️ Modern API - use EVEWindow[Inventory]):
```lavishscript
; MODERN (July 2020+): Use inventory windows
${EVEWindow[Inventory].Child[ShipCargo].FreeSpace}   ; Free cargo space
${EVEWindow[Inventory].Child[ShipCargo].Capacity}    ; Total cargo capacity
${EVEWindow[Inventory].Child[ShipCargo].UsedSpace}   ; Used cargo space

; DEPRECATED (legacy reference only):
; ${MyShip.CargoFreeSpace}, ${MyShip.CargoCapacity} removed July 2020
; See File 13 for modern inventory API patterns
```

**Module Access**:
```lavishscript
${MyShip.Module[ID]}            ; Module by ID
${MyShip.Module[Index]}         ; Module by slot index
${MyShip.Module[TypeID,Group]}  ; Module by type and group
```

**Methods**:
```lavishscript
MyShip:Open[]                   ; Open cargo
MyShip:GetModules[]             ; Get index of modules
MyShip:GetCargo[]               ; Get index of cargo items
MyShip:SetActiveTarget[]        ; Set active target
MyShip:LaunchAllDrones[]        ; Launch all drones
```

### The "EVE" Object

**${EVE}** = EVE client state

**Critical Members**:
```lavishscript
${EVE.Bookmark[ID]}             ; Bookmark by ID
${EVE.Bookmark[Name]}           ; Bookmark by name
${EVE.Station[ID]}              ; Station by ID
```

**Critical Methods**:
```lavishscript
EVE:QueryEntities[]             ; Query entities (MOST USED)
EVE:Execute[]                   ; Execute EVE command
EVE:GetBookmarks[]              ; Get index of bookmarks
```

**EVE:Execute Examples**:
```lavishscript
; Dock at station
EVE:Execute[CmdExitStation]     ; Undock

; UI commands
EVE:Execute[OpenShipHangar]     ; Open ship hangar
EVE:Execute[OpenCargoHold]      ; Open cargo
```

### The "entity" Object

**Most Important Object Type** (covered in detail later)

**${Entity[]}** = Access entity by ID or name

```lavishscript
variable entity Target = ${Entity[1234567890]}
variable entity Station = ${Entity["Jita IV - Moon 4 - Caldari Navy Assembly Plant"]}

; Entity members
${Target.ID}                    ; Unique ID (int64)
${Target.Name}                  ; Name (string)
${Target.Distance}              ; Distance in meters (float)
${Target.IsLockedTarget}        ; Locked by me? (bool)
${Target.IsNPC}                 ; Is this an NPC? (bool)
${Target.GroupID}               ; Group ID (int)
${Target.CategoryID}            ; Category ID (int)
```

---

## Members vs Methods

### Syntax Distinction

**Get Value (Member)**:
```lavishscript
; Use ${} to GET a value
variable float Shield = ${MyShip.Shield}
echo "Shield: ${MyShip.Shield}"
```

**Call Method (Action)**:
```lavishscript
; Use : to CALL a method
Entity:LockTarget[]
MyShip:Open[]
Module:Activate[]
```

### Members (Properties)

**Characteristics**:
- Read-only (usually)
- Return data from game state
- No side effects
- Fast (memory read)

**Examples**:
```lavishscript
${Me.Name}                  ; Read character name
${MyShip.Shield}            ; Read shield HP
${Entity.Distance}          ; Read distance
${Module.IsActive}          ; Read module state
```

**Some Members Take Parameters**:
```lavishscript
${MyShip.Module[1]}         ; Get module in slot 1
${Entity["Frigate"]}        ; Get entity by name
${Math.Calc[5 + 3]}         ; Calculate expression
```

### Methods (Actions)

**Characteristics**:
- Perform action in game
- Can modify game state
- May have side effects
- Take time to execute (server delay)

**Examples**:
```lavishscript
Entity:LockTarget[]                 ; Lock entity
Entity:Approach[]                   ; Approach entity
Entity:Orbit[5000]                  ; Orbit at 5km
Module:Activate[]                   ; Activate module
Module:Deactivate[]                 ; Deactivate module
MyShip:SetActiveTarget[ID]          ; Set active target
```

### Return Values

**Methods Can Return Values**:
```lavishscript
; Some methods return bool (success/failure)
if ${Entity:LockTarget[]}
{
    echo "Lock command sent successfully"
}
else
{
    echo "Lock command failed"
}
```

**But Most Don't**:
```lavishscript
; Most methods return nothing, just execute
Entity:Approach[]
; No return value to check

; Must verify action worked via members:
wait 10
if ${Entity.Distance} < ${OldDistance}
{
    echo "Approach working"
}
```

---

## Entity System

### What Are Entities

**Entity** = Anything in space with coordinates

**Entity Types**:
- Player ships
- NPC ships
- Asteroids
- Wrecks
- Stations
- Stargates
- Planets
- Containers
- Drones
- Missiles (projectiles)

**ALL are entity objects**

### Entity Identification

**Unique ID**:
```lavishscript
; Every entity has unique int64 ID
variable int64 EntityID = ${SomeEntity.ID}

; Can retrieve entity by ID later
variable entity Same = ${Entity[${EntityID}]}
```

**GroupID and CategoryID**:
```lavishscript
; GroupID = specific type (e.g., "Asteroid Belt", "Battleship", "Stargate")
; CategoryID = broad category (e.g., CATEGORY_ENTITY, CATEGORY_CELESTIAL)

; Example IDs (from EVE static data):
; GroupID 15 = Station
; GroupID 10 = Stargate
; GroupID 25 = Frigate
; CategoryID 11 = Ship
; CategoryID 2 = Celestial
```

**Filtering Entities**:
```lavishscript
; Query all NPCs
variable index:entity NPCs
EVE:QueryEntities[NPCs, "CategoryID = CATEGORY_ENTITY && IsNPC"]

; Query all asteroids
variable index:entity Roids
EVE:QueryEntities[Roids, "GroupID = GROUP_ASTEROID"]

; Query all wrecks
variable index:entity Wrecks
EVE:QueryEntities[Wrecks, "GroupID = GROUP_WRECK"]
```

### QueryEntities (Critical Method)

**Syntax**:
```lavishscript
EVE:QueryEntities[OutputIndex, "FilterExpression"]
```

**Parameters**:
- `OutputIndex`: index:entity variable to fill with results
- `FilterExpression`: string expression to filter entities

**Filter Syntax**:
```lavishscript
; Simple filter
"IsNPC"                     ; All NPCs

; Comparison
"Distance < 50000"          ; Within 50km

; Logical operators
"IsNPC && Distance < 50000" ; NPCs within 50km
"IsNPC || IsPC"             ; NPCs or players

; Grouping
"(IsNPC || IsPC) && Distance < 100000"

; Constants
"CategoryID = CATEGORY_ENTITY"
"GroupID = GROUP_WRECK"
```

**Common Filters**:
```lavishscript
; Hostiles (NPCs or unfriendly players)
"IsNPC || (IsPC && !IsFleetMember)"

; Lockable targets
"!IsLockedTarget && Distance < ${MyShip.MaxTargetRange}"

; Lootable wrecks
"GroupID = GROUP_WRECK && Distance < 100000"

; Minable asteroids
"GroupID = GROUP_ASTEROID && Distance < 50000"
```

**Example**:
```lavishscript
; Get all NPCs within 100km
variable index:entity NPCs
EVE:QueryEntities[NPCs, "IsNPC && Distance < 100000"]

; Iterate through results
variable iterator NPC
NPCs:GetIterator[NPC]

if ${NPC:First(exists)}
{
    do
    {
        echo "NPC: ${NPC.Value.Name} at ${NPC.Value.Distance}m"
    }
    while ${NPC:Next(exists)}
}
```

### Entity Members (Comprehensive)

**Identification**:
```lavishscript
${Entity.ID}                ; Unique ID (int64)
${Entity.Name}              ; Name (string)
${Entity.TypeID}            ; Type ID (int)
${Entity.GroupID}           ; Group ID (int)
${Entity.CategoryID}        ; Category ID (int)
```

**Position**:
```lavishscript
${Entity.X}                 ; X coordinate (float)
${Entity.Y}                 ; Y coordinate (float)
${Entity.Z}                 ; Z coordinate (float)
${Entity.Distance}          ; Distance from me (float)
```

**State**:
```lavishscript
${Entity.IsLockedTarget}    ; Am I locking this? (bool)
${Entity.IsTargetingMe}     ; Is it targeting me? (bool)
${Entity.IsNPC}             ; Is this an NPC? (bool)
${Entity.IsPC}              ; Is this a player? (bool)
${Entity.IsMoribund}        ; Is it exploding/dead? (bool)
${Entity.IsWarpingAway}     ; Is it warping away? (bool)
```

**Ownership**:
```lavishscript
${Entity.Owner}             ; Owner object
${Entity.OwnerID}           ; Owner ID (int)
${Entity.Owner.Name}        ; Owner name (string)
${Entity.Owner.CorpID}      ; Owner corp ID (int)
${Entity.Owner.AllianceID}  ; Owner alliance ID (int)
```

**Combat**:
```lavishscript
${Entity.Shield}            ; Current shield (float)
${Entity.ShieldPct}         ; Shield % (float)
${Entity.Armor}             ; Current armor (float)
${Entity.ArmorPct}          ; Armor % (float)
```

### Entity Methods (Critical Actions)

**Targeting**:
```lavishscript
Entity:LockTarget[]         ; Lock this entity
Entity:UnlockTarget[]       ; Unlock this entity
Entity:MakeActiveTarget[]   ; Set as active target
```

**Movement**:
```lavishscript
Entity:Approach[]           ; Approach entity
Entity:Orbit[distance]      ; Orbit at distance (meters)
Entity:KeepAtRange[distance]; Keep at distance
Entity:WarpTo[distance]     ; Warp to entity at distance
Entity:Dock[]               ; Dock (if station/structure)
```

**Interaction**:
```lavishscript
Entity:Open[]               ; Open (wreck, container, etc.)
Entity:LootAll[]            ; Loot everything
Entity:Activate[]           ; Activate (gate, etc.)
```

---

## UI Interaction

### EVE:Execute Command

**Purpose**: Execute EVE UI commands

**Common Commands**:
```lavishscript
EVE:Execute[CmdExitStation]             ; Undock
EVE:Execute[OpenShipHangar]             ; Open ship hangar
EVE:Execute[OpenCargoHold]              ; Open cargo hold
EVE:Execute[OpenDroneBay]               ; Open drone bay
EVE:Execute[CmdDronesReturnToBay]       ; Return drones
EVE:Execute[CmdDronesEngage]            ; Engage target with drones
EVE:Execute[CmdStopShip]                ; Stop ship
```

**Full List**:
- See IsxeveWiki for complete EVE:Execute command list
- Hundreds of commands available

### Windows

**Window Object**:
```lavishscript
; Access UI windows
variable index:window Windows
EVE:GetWindows[Windows]

; Check if specific window open
if ${Window["Local"](exists)}
{
    echo "Local chat is open"
}
```

**Common Window Names**:
- "Local" - Local chat
- "Inventory" - Inventory window
- "Ship Hangar" - Ship hangar
- "Market" - Market window

---

## Event System

### ISXEVE Events

**What Are Events**:
- Notifications when things happen in game
- Your script can subscribe (listen) to events
- Event fires → your function/atom called

**Common Events**:
```lavishscript
; On entity appeared
Event[OnFrame]:AttachAtom[OnFrameHandler]

; On target changed
Event[OnActiveTargetChanged]:AttachAtom[OnTargetChanged]
```

**Example**:
```lavishscript
; Define atom to handle event
atom OnTargetChanged()
{
    echo "Active target changed to: ${MyShip.ActiveTarget.ToEntity.Name}"
}

; Attach atom to event
Event[OnActiveTargetChanged]:AttachAtom[OnTargetChanged]

; Later: Detach
Event[OnActiveTargetChanged]:DetachAtom[OnTargetChanged]
```

**Available Events**:
- OnFrame - Every frame update
- OnActiveTargetChanged - Active target changed
- OnMyShipShieldsUpdate - Shield HP changed
- Many more (see IsxeveWiki)

---

## Common Patterns

### Existence Checking

**CRITICAL Pattern**:
```lavishscript
; ALWAYS check if object exists before using
if ${Entity[${TargetID}](exists)}
{
    ; Safe to use
    echo "${Entity[${TargetID}].Name}"
}
else
{
    echo "Entity doesn't exist (dead/left grid)"
}
```

**Why Necessary**:
- Entities can despawn (NPCs die, ships warp, etc.)
- Accessing non-existent object = crash or error
- Must ALWAYS check (exists) for dynamic objects

### Iterator Pattern

**Iterate Through Index**:
```lavishscript
variable index:entity Entities
EVE:QueryEntities[Entities, "IsNPC"]

variable iterator Entity
Entities:GetIterator[Entity]

if ${Entity:First(exists)}
{
    do
    {
        ; Process each entity
        echo "NPC: ${Entity.Value.Name}"

        ; Check if still exists (could despawn during iteration)
        if !${Entity.Value(exists)}
        {
            echo "Entity despawned during iteration!"
            continue
        }

        ; Do something with entity
        Entity.Value:LockTarget[]
    }
    while ${Entity:Next(exists)}
}
```

### Distance Checking

**Before Approaching**:
```lavishscript
; Check distance before approaching
if ${Entity.Distance} > 2500
{
    Entity:Approach[]
    wait 10

    ; Wait until in range
    while ${Entity.Distance} > 2500 && ${Entity(exists)}
    {
        wait 10
    }
}

if ${Entity.Distance} <= 2500
{
    ; In range, do action
    Entity:Open[]
}
```

---

## Limitations and Constraints

### What ISXEVE Cannot Do

**Server Authority**:
- Cannot fake actions to server
- Cannot teleport ship
- Cannot spawn items
- Cannot change standings
- Cannot bypass game rules

**UI Limitations**:
- Cannot read text from UI labels directly (in most cases)
- Cannot easily detect all UI states
- Some windows hard to interact with

**Timing**:
- No control over server response times
- Cannot force instant actions
- Must wait for server confirmation

### What ISXEVE Can Do

**Reading**:
- All entity data (positions, IDs, states)
- All ship data (HP, cap, modules)
- All inventory data
- Character data (wallet, skills, etc.)
- UI state (some windows)

**Actions**:
- All normal player actions (via UI commands)
- Lock, shoot, move, loot, dock, etc.
- Anything a player can do

**The Rule**:
> If a player can do it by clicking, ISXEVE can do it by scripting

---
