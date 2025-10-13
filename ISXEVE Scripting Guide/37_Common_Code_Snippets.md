# Common Code Snippets

> **Part of Layer 9: Reference Materials**
> Copy-paste ready code snippets for common EVE Online bot operations.

---

## Table of Contents

1. [Entity Management](#entity-management)
2. [Targeting](#targeting)
3. [Movement](#movement)
4. [Combat](#combat)
5. [Mining](#mining)
6. [Salvaging & Looting](#salvaging--looting)
7. [Docking & Undocking](#docking--undocking)
8. [Inventory Management](#inventory-management)
9. [Fleet Operations](#fleet-operations)
10. [Station Trading](#station-trading)
11. [Safety & Defense](#safety--defense)
12. [Performance & Profiling](#performance--profiling)
13. [Logging & Debugging](#logging--debugging)
14. [Configuration](#configuration)
15. [Utility Functions](#utility-functions)

---

## Entity Management

### Find Closest Entity (LavishScript)

```lavish
function GetClosestNPC()
{
    variable int64 ClosestID = 0
    variable float ClosestDist2 = 999999999999  ; Squared distance

    variable index:entity NPCs
    EVE:QueryEntities[NPCs, "CategoryID = 11 && IsNPC = TRUE && Distance < 100000"]

    variable iterator NPC
    NPCs:GetIterator[NPC]

    if ${NPC:First(exists)}
    {
        do
        {
            ; Use Distance2 (squared distance) for comparisons - 3-5x faster
            if ${NPC.Value.Distance2} < ${ClosestDist2}
            {
                ClosestID:Set[${NPC.Value.ID}]
                ClosestDist2:Set[${NPC.Value.Distance2}]
            }
        }
        while ${NPC:Next(exists)}
    }

    return ${ClosestID}
}
```

**Performance Note:** This function uses `Distance2` (squared distance) instead of `Distance` for comparisons. Distance2 is 3-5x faster because it avoids expensive square root calculations. When finding the closest entity, we only need to compare distances, not calculate the actual distance value.

### Find Closest Entity (.NET)

```csharp
public long GetClosestNPC()
{
    var npcs = _entityProvider.EntityWrappers
        .Where(e => e.IsNPC)
        .Where(e => e.CategoryID == (int)CategoryIDs.Entity)
        .Where(e => e.Distance < 100000)
        .OrderBy(e => e.Distance)
        .ToList();

    if (npcs.Any())
    {
        return npcs.First().ID;
    }

    return 0;
}
```

### Filter Entities by Type (LavishScript)

```lavish
; Get all asteroids within 50km
function GetAsteroids()
{
    variable index:entity Asteroids
    EVE:QueryEntities[Asteroids, "CategoryID = 25 && Distance < 50000"]

    echo "Found ${Asteroids.Used} asteroids"
    return ${Asteroids.Used}
}

; Get all wrecks
function GetWrecks()
{
    variable index:entity Wrecks
    EVE:QueryEntities[Wrecks, "GroupID = 226 && Distance < 100000"]

    echo "Found ${Wrecks.Used} wrecks"
    return ${Wrecks.Used}
}

; Get all players (not self)
function GetPlayers()
{
    variable index:entity Players
    EVE:QueryEntities[Players, "CategoryID = 6 && IsPC = TRUE"]

    variable iterator Player
    Players:GetIterator[Player]

    variable int Count = 0

    if ${Player:First(exists)}
    {
        do
        {
            if ${Player.Value.ID} != ${Me.CharID}
            {
                echo "Player: ${Player.Value.Name}"
                Count:Inc
            }
        }
        while ${Player:Next(exists)}
    }

    return ${Count}
}
```

### Safe Entity Access (LavishScript)

```lavish
function SafeEntityOperation(int64 EntityID)
{
    ; Always check exists before access
    if !${Entity[${EntityID}](exists)}
    {
        echo "ERROR: Entity ${EntityID} doesn't exist"
        return FALSE
    }

    ; Check distance
    if ${Entity[${EntityID}].Distance} > 100000
    {
        echo "ERROR: Entity too far (${Entity[${EntityID}].Distance}m)"
        return FALSE
    }

    ; Safe to operate
    echo "Entity: ${Entity[${EntityID}].Name}"
    return TRUE
}
```

---

## Targeting

### Lock Target (LavishScript)

```lavish
function LockTarget(int64 EntityID)
{
    ; Check entity exists
    if !${Entity[${EntityID}](exists)}
    {
        echo "ERROR: Entity doesn't exist"
        return FALSE
    }

    ; Check if already locked
    if ${Entity[${EntityID}].IsLockedTarget}
    {
        echo "Already locked: ${Entity[${EntityID}].Name}"
        return TRUE
    }

    ; Check target limit
    if ${Me.TargetCount} >= ${Me.MaxLockedTargets}
    {
        echo "ERROR: At max targets (${Me.TargetCount}/${Me.MaxLockedTargets})"
        return FALSE
    }

    ; Check range
    if ${Entity[${EntityID}].Distance} > ${Me.TargetingRange}
    {
        echo "ERROR: Out of targeting range"
        return FALSE
    }

    ; Lock target
    Entity[${EntityID}]:LockTarget
    echo "Locking ${Entity[${EntityID}].Name}..."

    ; Wait for lock
    variable int Timeout = 0
    while !${Entity[${EntityID}].IsLockedTarget} && ${Timeout} < 50
    {
        wait 10
        Timeout:Inc
    }

    if ${Entity[${EntityID}].IsLockedTarget}
    {
        echo "Locked: ${Entity[${EntityID}].Name}"
        return TRUE
    }
    else
    {
        echo "ERROR: Lock timeout"
        return FALSE
    }
}
```

### Unlock All Targets (LavishScript)

```lavish
function UnlockAllTargets()
{
    variable index:int64 LockedTargets
    Me:GetTargets[LockedTargets]

    variable iterator Target
    LockedTargets:GetIterator[Target]

    if ${Target:First(exists)}
    {
        do
        {
            if ${Entity[${Target.Value}](exists)}
            {
                Entity[${Target.Value}]:UnlockTarget
                echo "Unlocking ${Entity[${Target.Value}].Name}"
            }
        }
        while ${Target:Next(exists)}
    }

    wait 10
    echo "All targets unlocked"
}
```

### Target Priority Queue (.NET)

```csharp
public class TargetPriorityQueue
{
    private List<PriorityTarget> _targets = new List<PriorityTarget>();

    public void QueueTarget(long entityId, int priority, double distance)
    {
        // Don't add duplicates
        if (_targets.Any(t => t.EntityId == entityId))
            return;

        _targets.Add(new PriorityTarget
        {
            EntityId = entityId,
            Priority = priority,
            Distance = distance,
            QueuedAt = DateTime.Now
        });

        // Sort by priority (descending), then distance (ascending)
        _targets = _targets
            .OrderByDescending(t => t.Priority)
            .ThenBy(t => t.Distance)
            .ToList();
    }

    public long? GetHighestPriorityTarget()
    {
        if (!_targets.Any())
            return null;

        var target = _targets.First();

        // Check if entity still exists
        if (!_entityProvider.EntityWrappersById.ContainsKey(target.EntityId))
        {
            _targets.Remove(target);
            return GetHighestPriorityTarget(); // Recursive
        }

        return target.EntityId;
    }

    public void RemoveTarget(long entityId)
    {
        _targets.RemoveAll(t => t.EntityId == entityId);
    }
}

public class PriorityTarget
{
    public long EntityId { get; set; }
    public int Priority { get; set; }
    public double Distance { get; set; }
    public DateTime QueuedAt { get; set; }
}
```

---

## Movement

### Orbit Entity (LavishScript)

```lavish
function OrbitEntity(int64 EntityID, int Distance)
{
    if !${Entity[${EntityID}](exists)}
    {
        echo "ERROR: Entity doesn't exist"
        return FALSE
    }

    Entity[${EntityID}]:Orbit[${Distance}]
    echo "Orbiting ${Entity[${EntityID}].Name} at ${Distance}m"

    ; Activate afterburner/MWD
    call ActivatePropulsionModule

    return TRUE
}

function ActivatePropulsionModule()
{
    variable index:module PropMods
    MyShip:GetModulesByGroupID[PropMods, 46]  ; Afterburner
    MyShip:GetModulesByGroupID[PropMods, 51]  ; MWD

    variable iterator Mod
    PropMods:GetIterator[Mod]

    if ${Mod:First(exists)}
    {
        if !${Mod.Value.IsActive} && ${Mod.Value.IsOnline}
        {
            Mod.Value:Activate
            echo "Activated propulsion module"
        }
    }
}
```

### Warp to Entity (LavishScript)

```lavish
function WarpToEntity(int64 EntityID, int Distance)
{
    if !${Entity[${EntityID}](exists)}
    {
        echo "ERROR: Entity doesn't exist"
        return FALSE
    }

    echo "Warping to ${Entity[${EntityID}].Name} at ${Distance}m..."
    Entity[${EntityID}]:WarpTo[${Distance}]

    ; Wait to enter warp
    wait 50 ${Me.ToEntity.Mode} == 3

    if ${Me.ToEntity.Mode} != 3
    {
        echo "ERROR: Failed to enter warp"
        return FALSE
    }

    echo "In warp..."

    ; Wait to exit warp (max 5 minutes)
    wait 3000 ${Me.ToEntity.Mode} != 3

    if ${Me.ToEntity.Mode} == 3
    {
        echo "ERROR: Still in warp after timeout"
        return FALSE
    }

    echo "Arrived at ${Entity[${EntityID}].Name}"
    return TRUE
}
```

### Warp to Bookmark (LavishScript)

```lavish
function WarpToBookmark(string BookmarkName, int Distance)
{
    if !${EVE.Bookmark["${BookmarkName}"](exists)}
    {
        echo "ERROR: Bookmark doesn't exist: ${BookmarkName}"
        return FALSE
    }

    echo "Warping to bookmark: ${BookmarkName} at ${Distance}m..."
    EVE.Bookmark["${BookmarkName}"]:WarpTo[${Distance}]

    ; Wait to enter warp
    wait 50 ${Me.ToEntity.Mode} == 3

    if ${Me.ToEntity.Mode} != 3
    {
        echo "ERROR: Failed to enter warp"
        return FALSE
    }

    ; Wait to exit warp
    wait 3000 ${Me.ToEntity.Mode} != 3

    echo "Arrived at bookmark: ${BookmarkName}"
    return TRUE
}
```

### Movement Queue (.NET)

```csharp
public class MovementQueue
{
    private Queue<Destination> _destinations = new Queue<Destination>();
    private Destination _currentDestination = null;

    public void QueueWarpToEntity(long entityId, int distance)
    {
        _destinations.Enqueue(new Destination
        {
            Type = DestinationType.Entity,
            EntityId = entityId,
            Distance = distance
        });
    }

    public void QueueOrbit(long entityId, int distance)
    {
        _destinations.Enqueue(new Destination
        {
            Type = DestinationType.Orbit,
            EntityId = entityId,
            Distance = distance
        });
    }

    public void Process()
    {
        // If no current destination, get next
        if (_currentDestination == null && _destinations.Any())
        {
            _currentDestination = _destinations.Dequeue();
            ExecuteDestination(_currentDestination);
        }

        // Check if current destination complete
        if (_currentDestination != null)
        {
            if (IsDestinationComplete(_currentDestination))
            {
                _currentDestination = null;
            }
        }
    }

    private void ExecuteDestination(Destination dest)
    {
        if (!_entityProvider.EntityWrappersById.ContainsKey(dest.EntityId))
        {
            LogMessage("Entity doesn't exist, skipping destination");
            _currentDestination = null;
            return;
        }

        var entity = _entityProvider.EntityWrappersById[dest.EntityId];

        switch (dest.Type)
        {
            case DestinationType.WarpTo:
                entity.WarpTo(dest.Distance);
                break;

            case DestinationType.Orbit:
                entity.Orbit(dest.Distance);
                break;

            case DestinationType.Approach:
                entity.Approach();
                break;
        }
    }
}
```

---

## Combat

### Activate Weapons on Target (LavishScript)

```lavish
function ActivateWeaponsOnTarget(int64 TargetID)
{
    if !${Entity[${TargetID}].IsLockedTarget}
    {
        echo "ERROR: Target not locked"
        return FALSE
    }

    ; Get all weapon modules (turrets + launchers)
    variable index:module Weapons
    MyShip:GetTurretModules[Weapons]
    MyShip:GetLauncherModules[Weapons]

    variable iterator Weapon
    Weapons:GetIterator[Weapon]

    if ${Weapon:First(exists)}
    {
        do
        {
            ; Only activate if online, not active, and in range
            if ${Weapon.Value.IsOnline} && !${Weapon.Value.IsActive}
            {
                variable float MaxRange = ${Math.Calc[${Weapon.Value.OptimalRange} + ${Weapon.Value.FalloffRange}]}

                if ${Entity[${TargetID}].Distance} < ${MaxRange}
                {
                    Weapon.Value:Activate[${TargetID}]
                    echo "Activated ${Weapon.Value.ToItem.Name} on ${Entity[${TargetID}].Name}"
                }
            }
        }
        while ${Weapon:Next(exists)}
    }

    return TRUE
}
```

### Simple Combat Loop (LavishScript)

```lavish
function DoCombat()
{
    ; Get NPCs within range
    variable index:entity NPCs
    EVE:QueryEntities[NPCs, "CategoryID = 11 && IsNPC = TRUE && Distance < 100000"]

    if ${NPCs.Used} == 0
    {
        echo "No NPCs on grid"
        return
    }

    ; Lock closest NPC
    variable int64 TargetID
    call GetClosestNPC TargetID

    if ${TargetID} > 0
    {
        call LockTarget ${TargetID}
    }

    ; Orbit target
    if ${Entity[${TargetID}](exists)} && ${Entity[${TargetID}].IsLockedTarget}
    {
        call OrbitEntity ${TargetID} 15000
    }

    ; Activate weapons
    call ActivateWeaponsOnTarget ${TargetID}

    ; Deploy drones
    call DeployDrones

    ; Activate drones on target
    call ActivateDronesOnTarget ${TargetID}
}
```

### Combat State Machine (.NET)

```csharp
public enum CombatState
{
    Idle,
    Acquiring,
    Engaging,
    Fleeing
}

public class CombatModule
{
    private CombatState _state = CombatState.Idle;
    private long? _currentTargetId = null;

    public void Pulse()
    {
        switch (_state)
        {
            case CombatState.Idle:
                // Check for threats
                var npcs = GetNPCsInRange();
                if (npcs.Any())
                {
                    _state = CombatState.Acquiring;
                }
                break;

            case CombatState.Acquiring:
                // Select and lock target
                _currentTargetId = GetHighestPriorityTarget();
                if (_currentTargetId.HasValue)
                {
                    LockTarget(_currentTargetId.Value);
                    _state = CombatState.Engaging;
                }
                break;

            case CombatState.Engaging:
                if (!_currentTargetId.HasValue)
                {
                    _state = CombatState.Idle;
                    break;
                }

                // Check target still exists
                if (!_entityProvider.EntityWrappersById.ContainsKey(_currentTargetId.Value))
                {
                    _currentTargetId = null;
                    _state = CombatState.Acquiring;
                    break;
                }

                // Orbit target
                OrbitTarget(_currentTargetId.Value, 15000);

                // Activate weapons
                ActivateWeapons(_currentTargetId.Value);

                // Deploy and activate drones
                DeployDrones();
                ActivateDrones(_currentTargetId.Value);

                // Check if need to flee
                if (ShouldFlee())
                {
                    _state = CombatState.Fleeing;
                }
                break;

            case CombatState.Fleeing:
                // Warp to safe spot
                WarpToSafeSpot();
                _state = CombatState.Idle;
                break;
        }
    }
}
```

---

## Mining

### Mine Asteroid (LavishScript)

```lavish
function MineAsteroid(int64 AsteroidID)
{
    if !${Entity[${AsteroidID}](exists)}
    {
        echo "ERROR: Asteroid doesn't exist"
        return FALSE
    }

    ; Lock asteroid
    call LockTarget ${AsteroidID}

    if !${Entity[${AsteroidID}].IsLockedTarget}
    {
        echo "ERROR: Failed to lock asteroid"
        return FALSE
    }

    ; Activate mining lasers
    variable index:module MiningLasers
    MyShip:GetMiningLasers[MiningLasers]

    variable iterator Laser
    MiningLasers:GetIterator[Laser]

    if ${Laser:First(exists)}
    {
        do
        {
            if ${Laser.Value.IsOnline} && !${Laser.Value.IsActive}
            {
                if ${Entity[${AsteroidID}].Distance} < 25000
                {
                    Laser.Value:Activate[${AsteroidID}]
                    echo "Activated ${Laser.Value.ToItem.Name}"
                }
            }
        }
        while ${Laser:Next(exists)}
    }

    return TRUE
}
```

### Check Cargo Full (Modern API - LavishScript)

**⚠️ NOTE:** Old `MyShip.CargoCapacity` / `UsedCargoCapacity` members are DEPRECATED. Use EVEWindow[Inventory] instead.

```lavish
function IsCargoFull()
{
    ; Open inventory window if not open
    if !${EVEWindow[Inventory](exists)}
        EVE:Execute[CmdOpenInventory]

    wait 5

    if !${EVEWindow[Inventory](exists)}
    {
        echo "ERROR: Cannot open inventory window"
        return FALSE
    }

    ; Check ship cargo
    variable eveinvchildwindow Cargo = EVEWindow[Inventory].Child[ShipCargo]

    if ${Cargo(exists)} && ${Cargo.Capacity} > 0
    {
        variable float CargoPercent = ${Math.Calc[${Cargo.UsedCapacity} / ${Cargo.Capacity} * 100]}

        if ${CargoPercent} >= 90
        {
            echo "Cargo full: ${CargoPercent.Int}%"
            return TRUE
        }
    }

    return FALSE
}

function IsOreHoldFull()
{
    ; Open inventory window if not open
    if !${EVEWindow[Inventory](exists)}
        EVE:Execute[CmdOpenInventory]

    wait 5

    if !${EVEWindow[Inventory](exists)}
    {
        echo "ERROR: Cannot open inventory window"
        return FALSE
    }

    ; Check ore hold
    variable eveinvchildwindow OreHold = EVEWindow[Inventory].Child[ShipOreHold]

    if ${OreHold(exists)} && ${OreHold.Capacity} > 0
    {
        variable float OrePercent = ${Math.Calc[${OreHold.UsedCapacity} / ${OreHold.Capacity} * 100]}

        if ${OrePercent} >= 90
        {
            echo "Ore hold full: ${OrePercent.Int}%"
            return TRUE
        }
    }

    return FALSE
}

function HasCargoSpace(float RequiredSpace)
{
    ; Check if ship has required cargo space
    if !${EVEWindow[Inventory](exists)}
        EVE:Execute[CmdOpenInventory]

    wait 5

    if ${EVEWindow[Inventory](exists)}
    {
        variable eveinvchildwindow Cargo = EVEWindow[Inventory].Child[ShipCargo]
        if ${Cargo(exists)}
        {
            variable float FreeSpace = ${Math.Calc[${Cargo.Capacity} - ${Cargo.UsedCapacity}]}
            return ${Math.Calc[${FreeSpace} >= ${RequiredSpace}]}
        }
    }

    return FALSE
}
```

### Mining Loop (.NET)

**⚠️ NOTE:** This example uses deprecated cargo capacity properties for simplicity. In production .NET code, use ISXEVE's EVEWindow inventory access or implement a modern inventory wrapper.

```csharp
public class MiningModule
{
    public void Pulse()
    {
        // Check if cargo/ore hold full
        if (IsOreHoldFull())
        {
            WarpToStation();
            UnloadOre();
            WarpBackToField();
            return;
        }

        // Get asteroids (use Distance2 for performance)
        var asteroids = _entityProvider.EntityWrappers
            .Where(e => e.CategoryID == (int)CategoryIDs.Asteroid)
            .Where(e => e.Distance2 < (25000 * 25000))  // Distance2 is faster
            .OrderBy(e => e.Distance2)  // Sort by squared distance
            .ToList();

        if (!asteroids.Any())
        {
            LogMessage("No asteroids in range, warping to next belt");
            WarpToNextBelt();
            return;
        }

        // Lock asteroids (up to max targets)
        var lockedCount = _meCache.TargetCount;
        foreach (var asteroid in asteroids)
        {
            if (lockedCount >= _meCache.MaxLockedTargets)
                break;

            if (!asteroid.IsLockedTarget)
            {
                asteroid.LockTarget();
                lockedCount++;
            }
        }

        // Activate mining lasers
        foreach (var laser in _ship.MiningLaserModules)
        {
            if (!laser.IsActive && laser.IsOnline)
            {
                // Find locked asteroid in range
                var target = asteroids
                    .Where(a => a.IsLockedTarget)
                    .Where(a => a.Distance2 < (25000 * 25000))  // Use Distance2
                    .FirstOrDefault();

                if (target != null)
                {
                    laser.Activate(target.ID);
                }
            }
        }
    }

    // NOTE: Cargo capacity properties deprecated July 2020
    // For production code, use EVEWindow[Inventory] inventory access
    private bool IsOreHoldFull()
    {
        // DEPRECATED: UsedOreHoldCapacity / OreHoldCapacity
        // Modern approach: Access EVEWindow[Inventory].Child[ShipOreHold].UsedCapacity

        // Simplified example (deprecated API):
        var percentUsed = (_ship.UsedOreHoldCapacity / _ship.OreHoldCapacity) * 100;
        return percentUsed >= 90;

        // Production code should use inventory window access instead
    }
}
```

---

## Salvaging & Looting

### Salvage Wreck (LavishScript)

```lavish
function SalvageWreck(int64 WreckID)
{
    if !${Entity[${WreckID}](exists)}
    {
        echo "ERROR: Wreck doesn't exist"
        return FALSE
    }

    ; Check if empty
    if ${Entity[${WreckID}].IsEmpty}
    {
        echo "Wreck is empty"
        return FALSE
    }

    ; Lock wreck
    call LockTarget ${WreckID}

    ; Activate salvagers
    variable index:module Salvagers
    MyShip:GetSalvagerModules[Salvagers]

    variable iterator Salvager
    Salvagers:GetIterator[Salvager]

    if ${Salvager:First(exists)}
    {
        do
        {
            if ${Salvager.Value.IsOnline} && !${Salvager.Value.IsActive}
            {
                Salvager.Value:Activate[${WreckID}]
                echo "Salvaging ${Entity[${WreckID}].Name}"
            }
        }
        while ${Salvager:Next(exists)}
    }

    return TRUE
}
```

### Loot Wreck (LavishScript)

```lavish
function LootWreck(int64 WreckID)
{
    if !${Entity[${WreckID}](exists)}
    {
        return FALSE
    }

    ; Open wreck
    Entity[${WreckID}]:Open
    wait 20

    ; Get wreck cargo window
    variable int64 WreckWindowID
    ; Find active inventory window showing wreck
    ; (Simplified - actual implementation more complex)

    ; Loot all
    ; EVEWindow[${WreckWindowID}]:StackAll

    echo "Looted ${Entity[${WreckID}].Name}"
    return TRUE
}
```

### Salvage & Loot Loop (.NET)

```csharp
public void SalvageAndLoot()
{
    var wrecks = _entityProvider.EntityWrappers
        .Where(e => e.GroupID == (int)GroupIDs.Wreck)
        .Where(e => !e.IsEmpty)
        .Where(e => e.Distance < 100000)
        .OrderBy(e => e.Distance)
        .ToList();

    if (!wrecks.Any())
    {
        LogMessage("No wrecks to salvage");
        return;
    }

    foreach (var wreck in wrecks)
    {
        // Approach wreck
        if (wreck.Distance > 2500)
        {
            wreck.Approach();
            wait 20 wreck.Distance < 2500
        }

        // Lock wreck
        if (!wreck.IsLockedTarget && _meCache.TargetCount < _meCache.MaxLockedTargets)
        {
            wreck.LockTarget();
        }

        // Activate salvager
        var salvager = _ship.SalvagerModules.FirstOrDefault(s => !s.IsActive && s.IsOnline);
        if (salvager != null && wreck.IsLockedTarget)
        {
            salvager.Activate(wreck.ID);
        }

        // Loot wreck
        if (wreck.Distance < 2500)
        {
            wreck.Open();
            LootAll(wreck.ID);
        }
    }
}
```

---

## Docking & Undocking

### Dock at Station (LavishScript)

```lavish
function DockAtStation(int64 StationID)
{
    if !${Entity[${StationID}](exists)}
    {
        echo "ERROR: Station doesn't exist"
        return FALSE
    }

    echo "Docking at ${Entity[${StationID}].Name}..."

    ; Warp to station if far
    if ${Entity[${StationID}].Distance} > 200000
    {
        call WarpToEntity ${StationID} 0
    }

    ; Approach station
    if ${Entity[${StationID}].Distance} > 1000
    {
        Entity[${StationID}]:Approach
        wait 300 ${Entity[${StationID}].Distance} < 1000
    }

    ; Request dock
    Entity[${StationID}]:Dock
    echo "Docking request sent"

    ; Wait to dock
    wait 300 ${Me.InStation}

    if ${Me.InStation}
    {
        echo "Docked successfully"
        return TRUE
    }
    else
    {
        echo "ERROR: Docking timeout"
        return FALSE
    }
}
```

### Undock from Station (LavishScript)

```lavish
function UndockFromStation()
{
    if !${Me.InStation}
    {
        echo "ERROR: Not in station"
        return FALSE
    }

    echo "Undocking from ${Me.Station.Name}..."

    Me.Station:Undock
    wait 300 ${Me.InSpace}

    if ${Me.InSpace}
    {
        echo "Undocked successfully"
        return TRUE
    }
    else
    {
        echo "ERROR: Undock timeout"
        return FALSE
    }
}
```

---

## Inventory Management

**⚠️ WARNING: Deprecated API Below**

Most inventory management methods shown here use DEPRECATED APIs (July 2020). See File 09 and File 13 for modern `EVEWindow[Inventory]` patterns.

### Get Cargo Items (Modern API - LavishScript)

```lavish
function GetCargoItems()
{
    ; Open inventory window if not open
    if !${EVEWindow[Inventory](exists)}
        EVE:Execute[CmdOpenInventory]

    wait 10

    ; Get cargo items via inventory window
    variable index:item CargoItems
    if ${EVEWindow[Inventory](exists)}
    {
        EVEWindow[Inventory].Child[ShipCargo]:GetItems[CargoItems]

        ; Iterate using iterator
        variable iterator Item
        CargoItems:GetIterator[Item]

        if ${Item:First(exists)}
        {
            do
            {
                echo "${Item.Value.Name} x${Item.Value.Quantity}"
            }
            while ${Item:Next(exists)}
        }

        return ${CargoItems.Used}
    }

    return 0
}
```

### Stack All Items (Modern API - LavishScript)

```lavish
function StackAllCargo()
{
    ; Open inventory window
    if !${EVEWindow[Inventory](exists)}
        EVE:Execute[CmdOpenInventory]

    wait 10

    ; Stack cargo via inventory child window
    if ${EVEWindow[Inventory](exists)}
    {
        EVEWindow[Inventory].Child[ShipCargo]:StackAll
        echo "Stacking all cargo items"
        wait 20
        return TRUE
    }

    return FALSE
}
```

### Check Cargo Capacity (Modern API)

**⚠️ NOTE:** Use modern EVEWindow[Inventory] API for cargo checks (see examples above).

```lavish
function GetCargoPercentUsed()
{
    ; Modern API - use inventory window
    if !${EVEWindow[Inventory](exists)}
        EVE:Execute[CmdOpenInventory]

    wait 5

    if ${EVEWindow[Inventory](exists)}
    {
        variable eveinvchildwindow Cargo = EVEWindow[Inventory].Child[ShipCargo]
        if ${Cargo(exists)} && ${Cargo.Capacity} > 0
        {
            variable float PercentUsed = ${Math.Calc[${Cargo.UsedCapacity} / ${Cargo.Capacity} * 100]}
            return ${PercentUsed.Int}
        }
    }

    return 0
}
```

### ⚠️ DEPRECATED: Legacy Inventory Examples

```lavish
; DEPRECATED - DO NOT USE IN NEW SCRIPTS
; These examples shown only for understanding old code

; OLD API (DEPRECATED):
; function StackAllCargo_OLD()
; {
;     variable index:item CargoItems
;     MyShip:GetCargo[CargoItems]  ; DEPRECATED
;     ...
; }

; MODERN REPLACEMENT:
; Use EVEWindow[Inventory].Child[ShipCargo] as shown above
```

---

## Fleet Operations

### Get Fleet Members (LavishScript)

```lavish
function GetFleetMembers()
{
    if !${Me.Fleet(exists)}
    {
        echo "Not in fleet"
        return 0
    }

    variable index:fleetmember FleetMembers
    Me.Fleet:GetMembers[FleetMembers]

    variable iterator Member
    FleetMembers:GetIterator[Member]

    variable int Count = 0

    if ${Member:First(exists)}
    {
        do
        {
            echo "Fleet member: ${Member.Value.ToPilot.Name}"
            echo "  CharID: ${Member.Value.CharID}"
            echo "  ShipType: ${Member.Value.ToFleetMember.ShipType}"
            echo "  System: ${Member.Value.ToFleetMember.SolarSystemID}"
            Count:Inc
        }
        while ${Member:Next(exists)}
    }

    return ${Count}
}
```

### Broadcast to Fleet (LavishScript)

```lavish
function BroadcastToFleet(string Message)
{
    ; Send relay to all fleet members
    relay "all other" Event[FleetMessage]:Execute["${Me.Name}", "${Message}"]
}

; Event handler (in other fleet members' scripts)
Event[FleetMessage]:AttachAtom[FleetMessage_Handler]

atom FleetMessage_Handler(string SenderName, string Message)
{
    echo "Fleet message from ${SenderName}: ${Message}"
}
```

### Warp to Fleet Member (.NET)

```csharp
public void WarpToFleetMember(int charId)
{
    if (!_meCache.Fleet.Any())
    {
        LogMessage("Not in fleet");
        return;
    }

    var member = _meCache.Fleet.FirstOrDefault(m => m.CharID == charId);
    if (member == null)
    {
        LogMessage($"Fleet member {charId} not found");
        return;
    }

    LogMessage($"Warping to fleet member {member.Name}");

    // Queue warp to fleet member
    _movement.QueueDestination(new Destination(DestinationTypes.FleetMember, member.CharID)
    {
        Distance = 0
    });
}
```

---

## Safety & Defense

### Check for Hostile Players (LavishScript)

```lavish
function CheckForHostiles()
{
    variable index:entity Players
    EVE:QueryEntities[Players, "CategoryID = 6 && IsPC = TRUE"]

    variable iterator Player
    Players:GetIterator[Player]

    variable int HostileCount = 0

    if ${Player:First(exists)}
    {
        do
        {
            if ${Player.Value.ID} != ${Me.CharID}
            {
                ; Check standings (simplified)
                ; In reality, use proper standings check

                echo "WARNING: Player detected: ${Player.Value.Name}"
                HostileCount:Inc
            }
        }
        while ${Player:Next(exists)}
    }

    if ${HostileCount} > 0
    {
        echo "ALERT: ${HostileCount} player(s) on grid!"
        return TRUE
    }

    return FALSE
}
```

### Warp to Safe Spot (LavishScript)

```lavish
function WarpToSafeSpot()
{
    ; Get safe spot bookmark
    if ${EVE.Bookmark["Safe Spot"](exists)}
    {
        echo "Warping to safe spot..."
        call WarpToBookmark "Safe Spot" 0
        return TRUE
    }

    ; No safe spot, warp to random celestial
    echo "No safe spot bookmark, warping to planet..."
    variable index:entity Celestials
    EVE:QueryEntities[Celestials, "CategoryID = 2"]  ; Planets

    if ${Celestials.Used} > 0
    {
        variable int Random = ${Math.Rand[${Celestials.Used}]}
        Random:Inc  ; 1-based index

        variable int64 CelestialID = ${Celestials[${Random}].ID}
        call WarpToEntity ${CelestialID} 100000
        return TRUE
    }

    echo "ERROR: No safe locations found!"
    return FALSE
}
```

### Auto-Flee on Threat (.NET)

```csharp
public class DefenseModule
{
    private bool _isFleeing = false;

    public void Pulse()
    {
        if (_isFleeing)
            return;

        // Check for hostile players
        var hostiles = _entityProvider.EntityWrappers
            .Where(e => e.IsPC)
            .Where(e => e.ID != _meCache.ShipId)
            .Where(e => !IsFleetMember(e.Name))
            .Where(e => !IsBlue(e.CharID))
            .ToList();

        if (hostiles.Any())
        {
            LogMessage($"ALERT: {hostiles.Count} hostile(s) on grid!", LogSeverityTypes.Critical);
            Flee();
        }

        // Check shield/armor damage
        var shieldPercent = (_ship.Shield / _ship.MaxShield) * 100;
        var armorPercent = (_ship.Armor / _ship.MaxArmor) * 100;

        if (shieldPercent < 30 || armorPercent < 50)
        {
            LogMessage("ALERT: Low HP, fleeing!", LogSeverityTypes.Critical);
            Flee();
        }
    }

    private void Flee()
    {
        _isFleeing = true;

        // Recall drones
        RecallDrones();

        // Warp to safe spot
        WarpToSafeSpot();

        // Dock at station if in same system
        if (_config.SafeStationId > 0)
        {
            DockAtStation(_config.SafeStationId);
        }

        _isFleeing = false;
    }
}
```

---

## Performance & Profiling

### Simple Profiler (LavishScript)

```lavish
objectdef obj_Profiler
{
    variable int64 StartTime
    variable string OperationName

    method Start(string Name)
    {
        OperationName:Set["${Name}"]
        StartTime:Set[${Time.Timestamp}]
    }

    method End()
    {
        variable int64 ElapsedMs = ${Math.Calc[${Time.Timestamp} - ${This.StartTime}]}

        echo "${This.OperationName} took ${ElapsedMs}ms"

        if ${ElapsedMs} > 100
        {
            echo "WARNING: ${This.OperationName} exceeded 100ms threshold!"
        }
    }
}

; Usage
variable obj_Profiler Profiler

Profiler:Start["GetEntities"]
; ... operation ...
Profiler:End
```

### Performance Tracker (.NET)

```csharp
public class PerformanceTracker
{
    private Dictionary<string, List<TimeSpan>> _measurements = new Dictionary<string, List<TimeSpan>>();
    private Stopwatch _stopwatch = new Stopwatch();
    private string _currentOperation;

    public void Start(string operationName)
    {
        _currentOperation = operationName;
        _stopwatch.Restart();
    }

    public void End()
    {
        _stopwatch.Stop();

        if (!_measurements.ContainsKey(_currentOperation))
        {
            _measurements[_currentOperation] = new List<TimeSpan>();
        }

        _measurements[_currentOperation].Add(_stopwatch.Elapsed);

        // Keep only last 100 samples
        if (_measurements[_currentOperation].Count > 100)
        {
            _measurements[_currentOperation].RemoveAt(0);
        }

        // Warn if too slow
        if (_stopwatch.ElapsedMilliseconds > 100)
        {
            LogMessage($"WARNING: {_currentOperation} took {_stopwatch.ElapsedMilliseconds}ms", LogSeverityTypes.Critical);
        }
    }

    public void PrintReport()
    {
        foreach (var kvp in _measurements.OrderByDescending(x => x.Value.Average(t => t.TotalMilliseconds)))
        {
            var avg = kvp.Value.Average(t => t.TotalMilliseconds);
            var max = kvp.Value.Max(t => t.TotalMilliseconds);
            var min = kvp.Value.Min(t => t.TotalMilliseconds);

            LogMessage($"{kvp.Key}: Avg={avg:F2}ms, Min={min:F2}ms, Max={max:F2}ms");
        }
    }
}

// Usage
_profiler.Start("GetEntities");
// ... operation ...
_profiler.End();
```

---

## Logging & Debugging

### Logger Object (LavishScript)

```lavish
objectdef obj_Logger
{
    variable string LogFile = "${Script.CurrentDirectory}/bot.log"

    method Log(string Message, int Level)
    {
        variable string Timestamp = "${Time.Date} ${Time.Time24}"
        variable string LevelStr

        switch ${Level}
        {
            case 0
                LevelStr:Set["DEBUG"]
                break
            case 1
                LevelStr:Set["INFO"]
                break
            case 2
                LevelStr:Set["WARN"]
                break
            case 3
                LevelStr:Set["ERROR"]
                break
        }

        variable string LogLine = "[${Timestamp}] [${LevelStr}] ${Message}"

        ; Echo to console
        echo "${LogLine}"

        ; Write to file
        redirect -append "${This.LogFile}" echo "${LogLine}"
    }

    method Debug(string Message)
    {
        This:Log["${Message}", 0]
    }

    method Info(string Message)
    {
        This:Log["${Message}", 1]
    }

    method Warn(string Message)
    {
        This:Log["${Message}", 2]
    }

    method Error(string Message)
    {
        This:Log["${Message}", 3]
    }
}

; Usage
variable(global) obj_Logger Logger

Logger:Info["Bot started"]
Logger:Warn["Low capacitor"]
Logger:Error["Failed to lock target"]
```

---

## Configuration

### Config Manager (LavishScript)

```lavish
objectdef obj_Config
{
    variable settingsetref ConfigRef

    method Load(string ConfigFile)
    {
        LavishSettings:AddSet[BotConfig]

        if ${ConfigFile(exists)}
        {
            LavishSettings[BotConfig]:Import["${ConfigFile}"]
            echo "Config loaded from ${ConfigFile}"
        }
        else
        {
            echo "Config file not found, using defaults"
            This:SetDefaults
        }

        ConfigRef:Set[${LavishSettings[BotConfig]}]
    }

    method SetDefaults()
    {
        LavishSettings[BotConfig]:AddSetting[MiningRange, 25000]
        LavishSettings[BotConfig]:AddSetting[SafeSpotBookmark, "Safe Spot"]
        LavishSettings[BotConfig]:AddSetting[AutoFlee, TRUE]
    }

    method Save(string ConfigFile)
    {
        LavishSettings[BotConfig]:Export["${ConfigFile}"]
        echo "Config saved to ${ConfigFile}"
    }

    member:int GetInt(string Key)
    {
        return ${ConfigRef.FindSetting[${Key}]}
    }

    member:string GetString(string Key)
    {
        return "${ConfigRef.FindSetting[${Key}]}"
    }

    member:bool GetBool(string Key)
    {
        return ${ConfigRef.FindSetting[${Key}]}
    }
}

; Usage
variable(global) obj_Config Config

Config:Load["${Script.CurrentDirectory}/config.xml"]

variable int MiningRange = ${Config.GetInt[MiningRange]}
variable string SafeSpot = "${Config.GetString[SafeSpotBookmark]}"
variable bool AutoFlee = ${Config.GetBool[AutoFlee]}
```

### Config Manager with isxSQLite (LavishScript)

```lavish
objectdef obj_ConfigDatabase
{
    variable sqlitedb ConfigDB
    variable string DBPath = "${Script.CurrentDirectory}/botconfig.db"

    method Initialize()
    {
        ; Check if isxSQLite loaded
        if !${ISXSQLite(exists)}
        {
            echo "ERROR: isxSQLite not loaded"
            return FALSE
        }

        ; Wait for isxSQLite to be ready
        while !${ISXSQLite.IsReady}
            waitframe

        ; Open/create database
        ConfigDB:Set[${SQLite.OpenDB["BotConfig", "${This.DBPath}"]}]

        if !${ConfigDB.ID(exists)}
        {
            echo "ERROR: Failed to open config database"
            return FALSE
        }

        ; Create config table if doesn't exist
        if !${ConfigDB.TableExists["settings"]}
        {
            ConfigDB:ExecDML["CREATE TABLE settings (key TEXT PRIMARY KEY, value TEXT, type TEXT);"]
            This:SetDefaults
            echo "Config database initialized with defaults"
        }

        echo "Config loaded from database: ${This.DBPath}"
        return TRUE
    }

    method SetDefaults()
    {
        ; Insert default settings
        variable index:string DML
        DML:Insert["INSERT OR REPLACE INTO settings VALUES ('MiningRange', '25000', 'int');"]
        DML:Insert["INSERT OR REPLACE INTO settings VALUES ('SafeSpotBookmark', 'Safe Spot', 'string');"]
        DML:Insert["INSERT OR REPLACE INTO settings VALUES ('AutoFlee', 'TRUE', 'bool');"]
        DML:Insert["INSERT OR REPLACE INTO settings VALUES ('FleetMode', 'FALSE', 'bool');"]

        ConfigDB:ExecDMLTransaction[DML]
    }

    member:string Get(string Key)
    {
        variable sqlitequery Query
        Query:Set[${ConfigDB.ExecQuery["SELECT value FROM settings WHERE key='${SQLite.Escape_String[${Key}]}'"]}]

        if ${Query.NumRows} > 0
        {
            variable string Value = "${Query.GetFieldValue[1]}"
            Query:Finalize
            return "${Value}"
        }

        Query:Finalize
        return ""
    }

    member:int GetInt(string Key)
    {
        return ${This.Get["${Key}"]}
    }

    member:bool GetBool(string Key)
    {
        variable string Val = "${This.Get["${Key}"]}"
        if ${Val.Equal["TRUE"]}
            return TRUE
        return FALSE
    }

    method Set(string Key, string Value, string Type)
    {
        ConfigDB:ExecDML["INSERT OR REPLACE INTO settings VALUES ('${SQLite.Escape_String[${Key}]}', '${SQLite.Escape_String[${Value}]}', '${Type}');"]
    }

    method Close()
    {
        if ${ConfigDB.ID(exists)}
            ConfigDB:Close
    }
}

; Usage
variable(global) obj_ConfigDatabase DBConfig

DBConfig:Initialize

; Get values
variable int MiningRange = ${DBConfig.GetInt["MiningRange"]}
variable string SafeSpot = "${DBConfig.Get["SafeSpotBookmark"]}"
variable bool AutoFlee = ${DBConfig.GetBool["AutoFlee"]}

; Set values
DBConfig:Set["MiningRange", "30000", "int"]
DBConfig:Set["FleetMode", "TRUE", "bool"]

; Close on exit
DBConfig:Close
```

### Session Stats Tracker with isxSQLite (LavishScript)

```lavish
objectdef obj_SessionStats
{
    variable sqlitedb StatsDB
    variable int64 SessionID

    method Initialize()
    {
        ; Open stats database
        StatsDB:Set[${SQLite.OpenDB["Stats", "./stats.db"]}]

        ; Create tables
        if !${StatsDB.TableExists["sessions"]}
        {
            StatsDB:ExecDML["CREATE TABLE sessions (id INTEGER PRIMARY KEY, character TEXT, start_time INTEGER, end_time INTEGER, isk_earned REAL);"]
            StatsDB:ExecDML["CREATE TABLE kills (id INTEGER PRIMARY KEY, session_id INTEGER, npc_name TEXT, bounty REAL, timestamp INTEGER);"]
        }

        ; Start new session
        variable string CharName = "${SQLite.Escape_String[${Me.Name}]}"
        StatsDB:ExecDML["INSERT INTO sessions (character, start_time) VALUES ('${CharName}', ${Time.Timestamp});"]

        ; Get session ID
        variable sqlitequery Query
        Query:Set[${StatsDB.ExecQuery["SELECT last_insert_rowid();"]}]
        SessionID:Set[${Query.GetFieldValue[1, int]}]
        Query:Finalize

        echo "Session ${SessionID} started"
    }

    method LogKill(string NPCName, float Bounty)
    {
        variable string SafeName = "${SQLite.Escape_String[${NPCName}]}"
        StatsDB:ExecDML["INSERT INTO kills (session_id, npc_name, bounty, timestamp) VALUES (${SessionID}, '${SafeName}', ${Bounty}, ${Time.Timestamp});"]
    }

    method EndSession(float TotalISK)
    {
        StatsDB:ExecDML["UPDATE sessions SET end_time=${Time.Timestamp}, isk_earned=${TotalISK} WHERE id=${SessionID};"]
        echo "Session ${SessionID} ended - Earned ${TotalISK} ISK"
    }

    method GetTotalEarnings()
    {
        variable sqlitequery Query
        Query:Set[${StatsDB.ExecQuery["SELECT SUM(isk_earned) FROM sessions WHERE character='${SQLite.Escape_String[${Me.Name}]}';"]}]

        variable float Total = 0
        if ${Query.NumRows} > 0 && !${Query.FieldIsNULL[1]}
            Total:Set[${Query.GetFieldValue[1, float]}]

        Query:Finalize
        return ${Total}
    }
}

; Usage
variable(global) obj_SessionStats Stats

Stats:Initialize

; Log kills
Stats:LogKill["Sansha Commander", 1500000]
Stats:LogKill["Sansha Battletower", 3000000]

; End session
Stats:EndSession[${TotalISKEarned}]

; Get all-time earnings
echo "Total lifetime earnings: ${Stats.GetTotalEarnings} ISK"
```

---

## Utility Functions

### Distance vs Distance2 Performance (LavishScript)

**CRITICAL OPTIMIZATION:** Use `Distance2` (squared distance) for range comparisons - 3-5x faster than `Distance`.

```lavish
; ❌ SLOW - Distance requires expensive sqrt() calculation
function IsEntityInRange_SLOW(int64 EntityID, float MaxRange)
{
    if !${Entity[${EntityID}](exists)}
        return FALSE

    if ${Entity[${EntityID}].Distance} < ${MaxRange}
        return TRUE

    return FALSE
}

; ✅ FAST - Distance2 is raw squared distance (no sqrt)
function IsEntityInRange_FAST(int64 EntityID, float MaxRange)
{
    if !${Entity[${EntityID}](exists)}
        return FALSE

    ; Compare squared distances (much faster)
    variable float MaxRange2 = ${Math.Calc[${MaxRange} * ${MaxRange}]}

    if ${Entity[${EntityID}].Distance2} < ${MaxRange2}
        return TRUE

    return FALSE
}
```

**Why this matters:**
- `Distance` = sqrt(x² + y² + z²) → expensive square root operation
- `Distance2` = x² + y² + z² → no square root, just multiplication/addition
- Performance gain: 3-5x faster in tight loops with many entities
- Use Distance2 for: range checks, finding closest entity, distance comparisons
- Use Distance only when: you need the actual distance value for display/logging

### Calculate Distance (LavishScript)

```lavish
function CalculateDistance(float X1, float Y1, float Z1, float X2, float Y2, float Z2)
{
    return ${Math.Distance[${X1}, ${Y1}, ${Z1}, ${X2}, ${Y2}, ${Z2}]}
}

; Usage
variable float Dist = ${CalculateDistance[${Me.ToEntity.X}, ${Me.ToEntity.Y}, ${Me.ToEntity.Z}, ${Entity[${targetID}].X}, ${Entity[${targetID}].Y}, ${Entity[${targetID}].Z}]}
```

### Format ISK (LavishScript)

```lavish
function FormatISK(float Amount)
{
    if ${Amount} >= 1000000000
    {
        return "${Math.Calc[${Amount} / 1000000000]}B ISK"
    }
    elseif ${Amount} >= 1000000
    {
        return "${Math.Calc[${Amount} / 1000000]}M ISK"
    }
    elseif ${Amount} >= 1000
    {
        return "${Math.Calc[${Amount} / 1000]}K ISK"
    }
    else
    {
        return "${Amount} ISK"
    }
}

; Usage
echo "Balance: ${FormatISK[${Me.Wallet}]}"
```

### Random Delay (.NET)

```csharp
public class RandomDelay
{
    private Random _random = new Random();

    public void Wait(int minMs, int maxMs)
    {
        var delay = _random.Next(minMs, maxMs);
        Thread.Sleep(delay);
    }
}

// Usage
_randomDelay.Wait(1000, 3000);  // Wait 1-3 seconds
```

---

## Summary

This collection provides **ready-to-use code snippets** for the most common EVE Online bot operations.

### Quick Reference

**Entity Management:**
- Get closest NPC
- Filter by type
- Safe entity access

**Combat:**
- Lock targets
- Activate weapons
- Combat state machine

**Mining:**
- Mine asteroids
- Check cargo full
- Mining loop

**Movement:**
- Warp to entity/bookmark
- Orbit entity
- Movement queue

**Safety:**
- Check for hostiles
- Auto-flee
- Warp to safe spot

**Utilities:**
- Logging
- Profiling
- Configuration
- Distance calculation

### Usage Tips

1. **Copy snippets** into your own scripts
2. **Modify as needed** for your specific use case
3. **Combine snippets** to create complex behaviors
4. **Test thoroughly** before live use
5. **Add error handling** for production

---

**Previous Document:** `36_LavishScript_Command_Reference.md` - LavishScript language reference

**Next Document:** `38_Wiki_Navigation_Guide.md` - Final navigation guide

---

*Document complete. Common code snippets compiled for community use.*
