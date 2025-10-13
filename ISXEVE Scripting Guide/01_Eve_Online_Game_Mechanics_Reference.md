# EVE Online Game Mechanics Reference
## For Bot Development and Automation

**Purpose**: Comprehensive reference of Eve Online game mechanics relevant to bot development
**Audience**: AI/automation systems that need to understand game behavior
**Wiki References**: See IsxeveWiki for API access to these mechanics

---

## Table of Contents

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

**Next Steps**:
- Read `02_InnerSpace_Platform_Overview.md` to understand the bot hosting environment
- Read `03_ISXEVE_Extension_Architecture.md` to understand the game API
- Read `04_LavishScript_Language_Complete_Reference.md` to learn the scripting language

---

**END OF FILE**
**Next File**: 02_InnerSpace_Platform_Overview.md
