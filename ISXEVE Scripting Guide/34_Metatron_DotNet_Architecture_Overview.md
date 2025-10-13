# Metatron .NET Architecture Overview

> **Part of Layer 8: .NET Bridge**
> Real-world case study of a professional EVE Online .NET bot architecture.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture Philosophy](#architecture-philosophy)
3. [Module-Based Design](#module-based-design)
4. [Pulse-Driven Execution](#pulse-driven-execution)
5. [Dependency Injection](#dependency-injection)
6. [Entity Management System](#entity-management-system)
7. [Queue-Based Systems](#queue-based-systems)
8. [Threading and Async Patterns](#threading-and-async-patterns)
9. [Configuration Management](#configuration-management)
10. [ESI API Integration](#esi-api-integration)
11. [Event System](#event-system)
12. [Performance and Profiling](#performance-and-profiling)
13. [Code Organization](#code-organization)
14. [Design Patterns Used](#design-patterns-used)
15. [Lessons Learned](#lessons-learned)

---

## Project Overview

### Statistics

**Metatron** is a large-scale EVE Online automation bot written in C# (.NET Framework 4.8).

| Metric | Value |
|--------|-------|
| **Language** | C# (.NET Framework 4.8) |
| **Files** | 3,000+ |
| **Lines of Code** | 100,000+ |
| **Development Time** | 7+ years (2018-2025) |
| **Developers** | 5+ core contributors |
| **Users** | 100+ active users |
| **Supported Behaviors** | Combat, Mining, Missions, Salvaging, Hauling, Freighting |

### Technology Stack

```
Platform: InnerSpace (game automation host)
Framework: .NET Framework 4.8 (not .NET Core/8)
Language: C# 7.3
IDE: Visual Studio 2019/2022
Libraries:
  - ISXEVE (EVE memory reading)
  - LavishScript (scripting integration)
  - Newtonsoft.Json (JSON parsing)
  - protobuf-net (binary serialization)
  - System.Net.Http (ESI API calls)
```

### Project Structure

```
Metatron_spacekoala/
├── Metatron/                     # Main executable
│   ├── ActionModules/            # Low-level actions (Movement, Offensive, Defense)
│   ├── BehaviorModules/          # High-level behaviors (Ratting, Mining, Missions)
│   ├── Core/                     # Core systems (Ship, EntityCache, etc.)
│   └── Program.cs                # Entry point
│
├── Metatron.Core/                # Shared library
│   ├── Config/                   # Configuration classes
│   ├── Interfaces/               # Dependency injection interfaces
│   ├── CustomEventArgs/          # Event argument classes
│   └── Loader.cs                 # Application initialization
│
└── isxGamesPatcher/              # Auto-updater utility
```

---

## Architecture Philosophy

### Core Principles

1. **Module-Based Design**
   - Every feature is a module
   - Modules are self-contained
   - Clear separation of concerns

2. **Pulse-Driven Execution**
   - No blocking operations
   - Every module pulses periodically
   - Configurable pulse frequency

3. **Dependency Injection**
   - Constructor injection
   - Interface-based contracts
   - Testable and maintainable

4. **Queue-Based State Management**
   - Target queue for combat/mining
   - Destination queue for movement
   - Event queue for communication

5. **Performance First**
   - < 5ms pulse time (100x faster than LavishScript)
   - Profiling built-in
   - Optimized LINQ queries

### Design Goals

**Primary Goals:**
- ✅ Fast execution (< 10ms per pulse)
- ✅ Reliable automation (24/7 operation)
- ✅ Maintainable codebase (team of developers)
- ✅ Extensible architecture (easy to add features)

**Anti-Goals:**
- ❌ Not a framework (specific to Metatron)
- ❌ Not cross-platform (.NET Framework only)
- ❌ Not backward compatible (breaking changes OK)

---

## Module-Based Design

### Module Hierarchy

All modules inherit from `ModuleBase`, which provides:
- Logging (LogMessage, LogException, LogTrace)
- Profiling (StartPulseProfiling, EndPulseProfiling)
- Pulse management (ShouldPulse, PulseFrequency)

```csharp
public abstract class ModuleBase
{
    public string ModuleName { get; protected set; }
    public int PulseFrequency { get; protected set; } = 1;
    public bool IsEnabled { get; protected set; } = true;

    private DateTime _nextPulse = DateTime.MinValue;
    private Stopwatch _pulseProfiler = new Stopwatch();

    // Pulse control
    protected bool ShouldPulse()
    {
        if (!IsEnabled) return false;
        if (DateTime.Now < _nextPulse) return false;

        _nextPulse = DateTime.Now.AddSeconds(PulseFrequency);
        return true;
    }

    // Abstract pulse method - implemented by each module
    public abstract void Pulse();

    // Profiling
    protected void StartPulseProfiling()
    {
        _pulseProfiler.Restart();
    }

    protected void EndPulseProfiling()
    {
        _pulseProfiler.Stop();
        // Log if pulse took too long
        if (_pulseProfiler.ElapsedMilliseconds > 100)
        {
            LogMessage("Pulse", LogSeverityTypes.Critical,
                $"Pulse took {_pulseProfiler.ElapsedMilliseconds}ms!");
        }
    }

    // Logging
    protected void LogMessage(string methodName, LogSeverityTypes severity, string message, params object[] args)
    {
        Core.Metatron.Logging.LogMessage(ModuleName, new LogEventArgs(severity, methodName,
            string.Format(message, args)));
    }
}
```

### Module Types

#### 1. Action Modules (Low-Level)

**Location:** `ActionModules/`

**Purpose:** Execute specific game actions

**Examples:**
- **Movement.cs** - Ship movement (orbit, approach, warp)
- **Offensive.cs** - Weapon activation, combat
- **Defense.cs** - Shield/armor repair, fleeing
- **Targeting.cs** - Target locking/unlocking
- **Drones.cs** - Drone deployment and control
- **NonOffensive.cs** - Mining, salvaging, tractoring

**Characteristics:**
- Pulse every frame (PulseFrequency = 1)
- React immediately to game state
- No high-level decision making
- Consume queues (target queue, destination queue)

**Example - Movement Module:**

```csharp
internal sealed class Movement : ModuleBase, ICriticallyBlocking, IMovement
{
    private readonly List<Destination> _destinationQueue = new List<Destination>();
    private readonly IEntityProvider _entityProvider;
    private readonly IMeCache _meCache;
    private readonly IShip _ship;

    public Movement(IEntityProvider entityProvider, IMeCache meCache, IShip ship)
    {
        _entityProvider = entityProvider;
        _meCache = meCache;
        _ship = ship;

        ModuleName = "Movement";
        PulseFrequency = 1;  // Every frame
    }

    public override void Pulse()
    {
        if (!ShouldPulse()) return;

        StartPulseProfiling();

        // Don't move if in warp
        if (_meCache.ToEntity.Mode == (int)Modes.Warp)
        {
            EndPulseProfiling();
            return;
        }

        // Process destination queue
        if (_destinationQueue.Count > 0)
        {
            ProcessDestinationQueue();
        }

        // Manage propulsion module
        if (MovementType == MovementTypes.Approach &&
            _meCache.Ship.CapacitorPct >= 50)
        {
            _ship.ActivateModuleList(_ship.AfterBurnerModules, false);
        }
        else
        {
            DeactivatePropulsionMods();
        }

        EndPulseProfiling();
    }

    public void QueueDestination(Destination destination)
    {
        _destinationQueue.Add(destination);
        LogMessage("QueueDestination", LogSeverityTypes.Debug,
            $"Queued destination: {destination.DestinationType}");
    }

    private void ProcessDestinationQueue()
    {
        var destination = _destinationQueue.First();

        switch (destination.DestinationType)
        {
            case DestinationTypes.Entity:
                ProcessEntityDestination(destination);
                break;
            case DestinationTypes.BookMark:
                ProcessBookMarkDestination(destination);
                break;
            case DestinationTypes.SolarSystem:
                ProcessSolarSystem(destination);
                break;
        }
    }
}
```

#### 2. Behavior Modules (High-Level)

**Location:** `BehaviorModules/`

**Purpose:** Orchestrate action modules to achieve goals

**Examples:**
- **Ratting.cs** - Anomaly/belt combat
- **Mining.cs** - Asteroid mining operations
- **Salvaging.cs** - Wreck salvaging
- **MissionRunner.cs** - Mission automation
- **CombatAssist.cs** - Fleet combat support
- **Freighter.cs** - Hauling operations

**Characteristics:**
- Pulse every 2-5 seconds (PulseFrequency = 2-5)
- Make high-level decisions
- Queue actions for action modules
- State machine driven

**Example - Ratting Module:**

```csharp
internal class Ratting : BehaviorBase
{
    private RattingStates _rattingState = RattingStates.Idle;
    private ChangeAnomalyStates _changeAnomalyState;
    private long? _selectedAnomalyId;

    // Dependencies (injected via constructor)
    private readonly ISocial _social;
    private readonly IMeCache _meCache;
    private readonly IBookmarks _bookmarks;
    private readonly IRattingConfiguration _rattingConfiguration;
    private readonly IAnomalyProvider _anomalyProvider;
    private readonly IEntityProvider _entityProvider;
    private readonly IMovement _movement;
    private readonly IShip _ship;
    private readonly ITargetQueue _targetQueue;

    public Ratting(ISocial social, IMeCache meCache, IBookmarks bookmarks,
        IRattingConfiguration rattingConfiguration, IAnomalyProvider anomalyProvider,
        IEntityProvider entityProvider, IMovement movement, IShip ship,
        ITargetQueue targetQueue)
    {
        _social = social;
        _meCache = meCache;
        _bookmarks = bookmarks;
        _rattingConfiguration = rattingConfiguration;
        _anomalyProvider = anomalyProvider;
        _entityProvider = entityProvider;
        _movement = movement;
        _ship = ship;
        _targetQueue = targetQueue;

        ModuleName = "Ratting";
        PulseFrequency = 2;  // Every 2 seconds
    }

    public override void Pulse()
    {
        if (!ShouldPulse()) return;

        StartPulseProfiling();

        // Don't pulse if moving
        if (_movement.IsMoving && _movement.MovementType != MovementTypes.Approach)
        {
            EndPulseProfiling();
            return;
        }

        // Get entities on grid
        var npcs = _entityProvider.EntityWrappers
            .Where(e => e.IsNPC)
            .Where(e => e.Distance < 250000)
            .ToList();

        var players = _entityProvider.EntityWrappers
            .Where(e => e.IsPC)
            .Where(e => e.ID != _meCache.ShipId)
            .ToList();

        // Check for threats
        bool npcCheck = CheckNpcs(npcs);
        bool pcCheck = CheckPcs(players);

        // Update state machine
        SetPulseState();

        // Execute state
        ProcessPulseState();

        EndPulseProfiling();
    }

    private void SetPulseState()
    {
        // Fleeing?
        if (Core.Metatron.Defense.IsFleeing)
        {
            _rattingState = RattingStates.Defend;
            return;
        }

        // Error state?
        if (_rattingState == RattingStates.Error)
        {
            // Check if new anomalies available
            var anomalies = GetAvailableAnomalies();
            if (anomalies.Any())
            {
                _rattingState = RattingStates.Traveling;
            }
            return;
        }

        // Normal state machine logic...
    }

    private void ProcessPulseState()
    {
        switch (_rattingState)
        {
            case RattingStates.Idle:
                // Start ratting
                _rattingState = RattingStates.Traveling;
                break;

            case RattingStates.Traveling:
                // Warp to anomaly
                ProcessChangeAnomaly();
                break;

            case RattingStates.KillRats:
                // Combat logic
                QueueCombatTargets();
                break;

            case RattingStates.Salvaging:
                // Salvage wrecks
                QueueSalvageTargets();
                break;

            case RattingStates.Error:
                // Go to safe spot
                _movement.QueueDestination(new Destination(
                    DestinationTypes.SafeSpot));
                break;
        }
    }
}
```

#### 3. Core Modules

**Location:** `Core/`

**Purpose:** Foundational services and caching

**Examples:**
- **Ship.cs** - Ship state and modules
- **EntityCache.cs** - Entity caching
- **EntityProvider.cs** - Entity registry
- **AllianceCache.cs** - Alliance data from ESI
- **CorporationCache.cs** - Corporation data from ESI
- **MeCache.cs** - Player ship state
- **TargetQueue.cs** - Target management

**Characteristics:**
- Pulse frequency varies (1-10 seconds)
- Maintain cached data
- Provide services to other modules
- Thread-safe

**Example - Ship Module:**

```csharp
internal sealed class Ship : ModuleBase, IShip
{
    // Module lists (categorized)
    private readonly List<IModule> _allModules = new List<IModule>();
    private readonly List<IModule> _turretModules = new List<IModule>();
    private readonly List<IModule> _launcherModules = new List<IModule>();
    private readonly List<IModule> _miningLaserModules = new List<IModule>();
    private readonly List<IModule> _afterBurnerModules = new List<IModule>();
    private readonly List<IModule> _shieldBoosterModules = new List<IModule>();

    // Public read-only collections
    public ReadOnlyCollection<IModule> AllModules => _allModules.AsReadOnly();
    public ReadOnlyCollection<IModule> TurretModules => _turretModules.AsReadOnly();
    public ReadOnlyCollection<IModule> MiningLaserModules => _miningLaserModules.AsReadOnly();

    // Cached ranges
    private double _maxWeaponRange = -1;
    private double _maxMiningRange = -1;

    public Ship(IIsxeveProvider isxeveProvider, IMeCache meCache)
    {
        _isxeveProvider = isxeveProvider;
        _meCache = meCache;

        ModuleName = "Ship";
        PulseFrequency = 1;  // Every frame
    }

    public override void Pulse()
    {
        if (!ShouldPulse()) return;

        StartPulseProfiling();

        // Update module lists
        UpdateModuleLists();

        // Update cached values
        UpdateCachedRanges();

        EndPulseProfiling();
    }

    private void UpdateModuleLists()
    {
        _allModules.Clear();
        _turretModules.Clear();
        _launcherModules.Clear();
        _miningLaserModules.Clear();

        foreach (var module in _meCache.Ship.Modules)
        {
            _allModules.Add(module);

            if (module.GroupID == (int)GroupIDs.ProjectileTurret ||
                module.GroupID == (int)GroupIDs.HybridTurret ||
                module.GroupID == (int)GroupIDs.EnergyTurret)
            {
                _turretModules.Add(module);
            }
            else if (module.GroupID == (int)GroupIDs.MissileLauncher)
            {
                _launcherModules.Add(module);
            }
            else if (module.GroupID == (int)GroupIDs.MiningLaser)
            {
                _miningLaserModules.Add(module);
            }
        }
    }

    public double GetMaximumWeaponRange()
    {
        if (_maxWeaponRange < 0)
        {
            _maxWeaponRange = _turretModules
                .Union(_launcherModules)
                .Max(m => m.OptimalRange + m.FalloffRange);
        }

        return _maxWeaponRange;
    }

    public void ActivateModuleList(ReadOnlyCollection<IModule> modules, bool immediate)
    {
        foreach (var module in modules)
        {
            if (!module.IsActive && module.IsOnline)
            {
                module.Activate();
                if (immediate) break;
            }
        }
    }
}
```

### Module Registration

Modules register themselves in their constructor:

```csharp
// Action modules
ModuleManager.ModulesToPulse.Add(this);

// Behavior modules
BehaviorManager.BehaviorsToPulse.Add(BotModes.Ratting, this);

// Critically blocking modules
ModuleManager.CriticallyBlockingModules.Add(this);
```

Main loop pulses all registered modules:

```csharp
public class Core
{
    public void MainLoop()
    {
        while (true)
        {
            // Pulse all action modules
            foreach (var module in ModuleManager.ModulesToPulse)
            {
                module.Pulse();
            }

            // Pulse active behavior module
            var activeBehavior = BehaviorManager.BehaviorsToPulse[CurrentBotMode];
            activeBehavior?.Pulse();

            // Frame delay
            Thread.Sleep(10);  // ~100 FPS
        }
    }
}
```

---

## Pulse-Driven Execution

### Pulse Philosophy

**Key Concept:** No blocking operations allowed.

Every module pulses periodically and makes incremental progress.

**Example - Bad (Blocking):**

```csharp
// ❌ BAD - Blocks for 5 seconds
public void DockAtStation()
{
    station.Dock();
    Thread.Sleep(5000);  // Wait for docking
    // Now in station
}
```

**Example - Good (Pulse-Driven):**

```csharp
// ✅ GOOD - Incremental progress
private DockingStates _dockingState = DockingStates.Idle;

public override void Pulse()
{
    if (!ShouldPulse()) return;

    switch (_dockingState)
    {
        case DockingStates.Idle:
            // Not docking
            break;

        case DockingStates.RequestDock:
            station.Dock();
            _dockingState = DockingStates.Waiting;
            break;

        case DockingStates.Waiting:
            if (_meCache.InStation)
            {
                _dockingState = DockingStates.Complete;
                LogMessage("Pulse", LogSeverityTypes.Standard, "Docking complete!");
            }
            break;

        case DockingStates.Complete:
            _dockingState = DockingStates.Idle;
            break;
    }
}
```

### Pulse Frequency Tuning

| Module | Frequency | Reason |
|--------|-----------|--------|
| Movement | 1 (every frame) | Immediate reaction required |
| Targeting | 1 (every frame) | Lock/unlock must be fast |
| Offensive | 1 (every frame) | Weapon activation critical |
| Defense | 1 (every frame) | Survival depends on speed |
| Ship | 1 (every frame) | Module state changes fast |
| Ratting | 2 (every 2 sec) | Decision making can be slower |
| Mining | 3 (every 3 sec) | Mining cycles are long |
| Social | 5 (every 5 sec) | Standings don't change often |
| Statistics | 10 (every 10 sec) | Performance tracking |

**Rule of Thumb:**
- Action modules: 1 (every frame)
- Behavior modules: 2-5 (every 2-5 seconds)
- Core caching: 5-10 (every 5-10 seconds)

---

## Dependency Injection

### Constructor Injection Pattern

All dependencies are injected via constructor:

```csharp
internal class Ratting : BehaviorBase
{
    // Private readonly fields
    private readonly ISocial _social;
    private readonly IMeCache _meCache;
    private readonly IBookmarks _bookmarks;
    private readonly IRattingConfiguration _rattingConfiguration;
    private readonly IAnomalyProvider _anomalyProvider;
    private readonly IEntityProvider _entityProvider;
    private readonly IMovement _movement;
    private readonly IShip _ship;

    // Constructor - all dependencies injected
    public Ratting(
        ISocial social,
        IMeCache meCache,
        IBookmarks bookmarks,
        IRattingConfiguration rattingConfiguration,
        IAnomalyProvider anomalyProvider,
        IEntityProvider entityProvider,
        IMovement movement,
        IShip ship)
    {
        _social = social;
        _meCache = meCache;
        _bookmarks = bookmarks;
        _rattingConfiguration = rattingConfiguration;
        _anomalyProvider = anomalyProvider;
        _entityProvider = entityProvider;
        _movement = movement;
        _ship = ship;
    }
}
```

### Interface-Based Contracts

Every service has an interface:

```csharp
// IMovement.cs
public interface IMovement
{
    bool IsMoving { get; }
    MovementTypes MovementType { get; }

    void QueueDestination(Destination destination);
    void StopCurrentMovement(bool dequeue);
}

// IShip.cs
public interface IShip
{
    ReadOnlyCollection<IModule> AllModules { get; }
    ReadOnlyCollection<IModule> TurretModules { get; }

    double GetMaximumWeaponRange();
    void ActivateModuleList(ReadOnlyCollection<IModule> modules, bool immediate);
}

// IEntityProvider.cs
public interface IEntityProvider
{
    List<IEntityWrapper> EntityWrappers { get; }
    Dictionary<long, IEntityWrapper> EntityWrappersById { get; }
}
```

### Dependency Registration

Dependencies are registered in `Loader.cs`:

```csharp
public class Loader
{
    public static void Load()
    {
        // Core services
        var isxeveProvider = new IsxeveProvider();
        var entityProvider = new EntityProvider(isxeveProvider);
        var meCache = new MeCache(isxeveProvider);
        var ship = new Ship(isxeveProvider, meCache);

        // Action modules
        var movement = new Movement(isxeveProvider, entityProvider, meCache, ship);
        var offensive = new Offensive(entityProvider, meCache, ship, movement);
        var targeting = new Targeting(entityProvider, meCache, ship);

        // Behavior modules
        var social = new Social(meCache);
        var rattingConfig = new RattingConfiguration();
        var anomalyProvider = new AnomalyProvider(isxeveProvider);
        var bookmarks = new Bookmarks(isxeveProvider);
        var targetQueue = new TargetQueue();

        var ratting = new Ratting(
            social,
            meCache,
            bookmarks,
            rattingConfig,
            anomalyProvider,
            entityProvider,
            movement,
            ship,
            targeting,
            targetQueue
        );

        // Register in Core
        Core.Metatron.Movement = movement;
        Core.Metatron.Offensive = offensive;
        Core.Metatron.Ratting = ratting;
    }
}
```

### Benefits

✅ **Testable:** Mock interfaces for unit tests
✅ **Maintainable:** Clear dependencies
✅ **Flexible:** Easy to swap implementations
✅ **Discoverable:** See all dependencies in constructor

---

## Entity Management System

### Entity Lifecycle

```
Game Entity Spawns
    ↓
ISXEVE detects entity
    ↓
EntityProvider.UpdateEntities() called
    ↓
Create EntityWrapper (wraps ISXEVE entity)
    ↓
Add to EntityWrappers list
    ↓
Add to EntityWrappersById dictionary
    ↓
Entity available to all modules
    ↓
Entity despawns/dies in game
    ↓
ISXEVE reports entity gone
    ↓
EntityProvider removes from lists
    ↓
Entity no longer accessible
```

### EntityProvider Implementation

```csharp
public class EntityProvider : ModuleBase, IEntityProvider
{
    private readonly IIsxeveProvider _isxeveProvider;

    // Two collections for different access patterns
    public List<IEntityWrapper> EntityWrappers { get; private set; }
    public Dictionary<long, IEntityWrapper> EntityWrappersById { get; private set; }

    public EntityProvider(IIsxeveProvider isxeveProvider)
    {
        _isxeveProvider = isxeveProvider;

        EntityWrappers = new List<IEntityWrapper>();
        EntityWrappersById = new Dictionary<long, IEntityWrapper>();

        ModuleName = "EntityProvider";
        PulseFrequency = 1;  // Update every frame
    }

    public override void Pulse()
    {
        if (!ShouldPulse()) return;

        StartPulseProfiling();

        // Clear old entities
        EntityWrappers.Clear();
        EntityWrappersById.Clear();

        // Get fresh entities from ISXEVE
        var entities = _isxeveProvider.EVE.GetEntities();

        foreach (var entity in entities)
        {
            var wrapper = new EntityWrapper(entity);

            EntityWrappers.Add(wrapper);
            EntityWrappersById[entity.ID] = wrapper;
        }

        EndPulseProfiling();
    }
}
```

### EntityWrapper Interface

```csharp
public interface IEntityWrapper
{
    // Identity
    long ID { get; }
    string Name { get; }
    int TypeID { get; }
    int GroupID { get; }
    int CategoryID { get; }

    // State
    double Distance { get; }
    bool IsNPC { get; }
    bool IsPC { get; }
    bool IsLockedTarget { get; }
    bool BeingTargeted { get; }

    // Actions
    void LockTarget();
    void UnlockTarget();
    void Approach();
    void Orbit(int distance);
    void Open();
}
```

### Safe Entity Access Pattern

**Always check entity exists before accessing:**

```csharp
// ❌ BAD - Entity might not exist
public void OrbitTarget(long targetId)
{
    var entity = _entityProvider.EntityWrappersById[targetId];  // Crash if not found!
    entity.Orbit(15000);
}

// ✅ GOOD - Check exists first
public void OrbitTarget(long targetId)
{
    if (!_entityProvider.EntityWrappersById.ContainsKey(targetId))
    {
        LogMessage("OrbitTarget", LogSeverityTypes.Debug,
            $"Entity {targetId} no longer exists.");
        return;
    }

    var entity = _entityProvider.EntityWrappersById[targetId];
    entity.Orbit(15000);
}

// ✅ BETTER - TryGetValue pattern
public void OrbitTarget(long targetId)
{
    if (_entityProvider.EntityWrappersById.TryGetValue(targetId, out var entity))
    {
        entity.Orbit(15000);
    }
    else
    {
        LogMessage("OrbitTarget", LogSeverityTypes.Debug,
            $"Entity {targetId} no longer exists.");
    }
}
```

### LINQ Query Patterns

Efficient entity filtering using LINQ:

```csharp
// Get all NPCs within weapon range
var npcs = _entityProvider.EntityWrappers
    .Where(e => e.IsNPC)
    .Where(e => e.Distance < _ship.GetMaximumWeaponRange())
    .OrderBy(e => e.Distance)
    .ToList();

// Get specific entity types
var asteroids = _entityProvider.EntityWrappers
    .Where(e => e.CategoryID == (int)CategoryIDs.Asteroid)
    .Where(e => e.GroupID == (int)GroupIDs.Veldspar)
    .Where(e => e.Distance < 25000)
    .ToList();

// Get wrecks to salvage
var wrecks = _entityProvider.EntityWrappers
    .Where(e => e.GroupID == (int)GroupIDs.Wreck)
    .Where(e => !e.IsEmpty)
    .Where(e => e.Distance < 100000)
    .OrderBy(e => e.Distance)
    .Take(5)
    .ToList();
```

**Performance:** These queries take < 1ms even with 1000+ entities

---

## Queue-Based Systems

### Target Queue

**Purpose:** Manage what to target (combat, mining, salvaging)

```csharp
public class QueueTarget
{
    public long Id { get; set; }              // Entity ID
    public TargetTypes Type { get; set; }     // Kill, Mine, LootSalvage
    public int Priority { get; set; }         // Higher = more important
    public double SubPriority { get; set; }   // Tiebreaker (distance)
    public DateTime TimeQueued { get; set; }  // When added
}

public enum TargetTypes
{
    Kill,           // Combat target
    Mine,           // Asteroid
    LootSalvage,    // Wreck
    Approach        // Just approach
}
```

**Implementation:**

```csharp
public class TargetQueue : ITargetQueue
{
    private readonly List<QueueTarget> _targets = new List<QueueTarget>();

    public ReadOnlyCollection<QueueTarget> Targets => _targets.AsReadOnly();

    public void QueueTarget(QueueTarget target)
    {
        // Don't queue duplicates
        if (_targets.Any(t => t.Id == target.Id))
            return;

        _targets.Add(target);

        // Sort by priority, then subpriority
        _targets.Sort((a, b) =>
        {
            int priorityCompare = b.Priority.CompareTo(a.Priority);
            if (priorityCompare != 0)
                return priorityCompare;

            return a.SubPriority.CompareTo(b.SubPriority);
        });
    }

    public void DequeueTarget(long entityId)
    {
        _targets.RemoveAll(t => t.Id == entityId);
    }

    public QueueTarget GetHighestPriorityTarget(TargetTypes type)
    {
        return _targets
            .Where(t => t.Type == type)
            .OrderByDescending(t => t.Priority)
            .ThenBy(t => t.SubPriority)
            .FirstOrDefault();
    }
}
```

**Usage in Ratting:**

```csharp
// Queue combat targets
foreach (var npc in npcs)
{
    _targetQueue.QueueTarget(new QueueTarget
    {
        Id = npc.ID,
        Type = TargetTypes.Kill,
        Priority = GetNpcPriority(npc),  // E.g. 100 for frigates, 200 for cruisers
        SubPriority = npc.Distance,      // Closer = lower subpriority = higher precedence
        TimeQueued = DateTime.Now
    });
}

// Get highest priority target
var primaryTarget = _targetQueue.GetHighestPriorityTarget(TargetTypes.Kill);
if (primaryTarget != null)
{
    // Lock and shoot
    if (!_entityProvider.EntityWrappersById[primaryTarget.Id].IsLockedTarget)
    {
        _entityProvider.EntityWrappersById[primaryTarget.Id].LockTarget();
    }
}
```

### Destination Queue

**Purpose:** Manage where to move (orbit, warp, dock)

```csharp
public class Destination
{
    public DestinationTypes DestinationType { get; set; }
    public long EntityId { get; set; }        // Entity to move to
    public long BookMarkId { get; set; }      // Bookmark to move to
    public int SolarSystemId { get; set; }    // System to jump to
    public double Distance { get; set; }      // How close to get
    public ApproachTypes ApproachType { get; set; }  // Orbit/Approach
    public bool Dock { get; set; }            // Dock when arrived?
}

public enum DestinationTypes
{
    Entity,        // Move to entity
    BookMark,      // Move to bookmark
    SolarSystem,   // Jump to system
    Undock,        // Undock from station
    SafeSpot       // Go to safe spot
}

public enum ApproachTypes
{
    Approach,      // Get within distance
    Orbit,         // Orbit at distance
    None           // Just warp to
}
```

**Usage in Offensive:**

```csharp
// Queue orbit for combat target
Core.Metatron.Movement.QueueDestination(new Destination(
    DestinationTypes.Entity,
    activeTarget.ID)
{
    Distance = 15000,
    ApproachType = ApproachTypes.Orbit
});
```

**Processing in Movement:**

```csharp
private void ProcessDestinationQueue()
{
    if (_destinationQueue.Count == 0) return;

    var destination = _destinationQueue.First();

    switch (destination.DestinationType)
    {
        case DestinationTypes.Entity:
            ProcessEntityDestination(destination);
            break;

        case DestinationTypes.BookMark:
            ProcessBookMarkDestination(destination);
            break;

        case DestinationTypes.SolarSystem:
            ProcessSolarSystem(destination);
            break;
    }
}

private void ProcessEntityDestination(Destination destination)
{
    // Check entity exists
    if (!_entityProvider.EntityWrappersById.ContainsKey(destination.EntityId))
    {
        // Entity died - check if it was combat orbit
        if (destination.ApproachType == ApproachTypes.Orbit && destination.Distance > 0)
        {
            // Combat target died - maintain velocity for next target
            LogMessage("ProcessEntityDestination", LogSeverityTypes.Debug,
                "Combat target died; maintaining velocity.");
            DequeueDestination(destination);
            return;
        }

        // Not combat - stop ship
        StopCurrentMovement(true);
        return;
    }

    var entity = _entityProvider.EntityWrappersById[destination.EntityId];

    // Check approach type
    if (destination.ApproachType == ApproachTypes.Orbit)
    {
        ProcessEntityOrbit(destination, entity);
    }
    else if (destination.ApproachType == ApproachTypes.Approach)
    {
        ProcessEntityApproach(destination, entity);
    }
}
```

---

## Threading and Async Patterns

### ThreadPool Pattern (Primary)

Metatron uses `ThreadPool.QueueUserWorkItem` for background operations:

```csharp
// AllianceCache.cs
public void GetAllianceInfo(int allianceId)
{
    // Queue work on thread pool
    ThreadPool.QueueUserWorkItem(TryGetAllianceInfo,
        new AllianceStateInfo(allianceId));
}

private void TryGetAllianceInfo(object stateInfo)
{
    var info = (AllianceStateInfo)stateInfo;

    try
    {
        var url = $"https://esi.evetech.net/latest/alliances/{info.AllianceId}/";

        // Synchronous HTTP call (on thread pool thread)
        var response = _httpClient.GetAsync(url).GetAwaiter().GetResult();
        response.EnsureSuccessStatusCode();

        var json = response.Content.ReadAsStringAsync().GetAwaiter().GetResult();
        var alliance = JsonConvert.DeserializeObject<ESIAlliance>(json);

        // Cache result
        lock (_cacheLock)
        {
            _allianceCache[info.AllianceId] = alliance;
        }
    }
    catch (HttpRequestException ex)
    {
        if (ex.StatusCode == HttpStatusCode.ServiceUnavailable)
        {
            // ESI is down, retry later
            LogMessage("TryGetAllianceInfo", LogSeverityTypes.Debug,
                $"ESI unavailable for alliance {info.AllianceId}, will retry.");
        }
    }
}
```

### Why Not async/await?

**Reasons:**
1. **Codebase is synchronous** - Pulse-driven model doesn't need async
2. **Complexity** - async/await adds complexity without benefit
3. **.NET Framework 4.8** - async/await available but not widely used in legacy codebase
4. **ThreadPool works** - Simple and effective for background tasks

**When async IS used:**

```csharp
// ESI API calls (in newer code)
private static readonly HttpClient _httpClient = new HttpClient
{
    Timeout = TimeSpan.FromSeconds(30)
};

public async Task<ESIAlliance> GetAllianceInfoAsync(int allianceId)
{
    var url = $"https://esi.evetech.net/latest/alliances/{allianceId}/";

    var response = await _httpClient.GetAsync(url);
    response.EnsureSuccessStatusCode();

    var json = await response.Content.ReadAsStringAsync();
    return JsonConvert.DeserializeObject<ESIAlliance>(json);
}
```

But called synchronously in pulse methods:

```csharp
public override void Pulse()
{
    // Call async method synchronously
    var alliance = GetAllianceInfoAsync(allianceId).GetAwaiter().GetResult();
}
```

### Thread Safety

**Thread-safe collections:**

```csharp
using System.Collections.Concurrent;

private ConcurrentDictionary<int, Alliance> _allianceCache =
    new ConcurrentDictionary<int, Alliance>();

// Thread-safe add
_allianceCache[allianceId] = alliance;

// Thread-safe read
if (_allianceCache.TryGetValue(allianceId, out var alliance))
{
    // Use alliance
}
```

**Lock-based synchronization:**

```csharp
private readonly object _cacheLock = new object();
private Dictionary<int, Alliance> _allianceCache = new Dictionary<int, Alliance>();

// Thread-safe add
lock (_cacheLock)
{
    _allianceCache[allianceId] = alliance;
}

// Thread-safe read
lock (_cacheLock)
{
    if (_allianceCache.TryGetValue(allianceId, out var alliance))
    {
        return alliance;
    }
}
```

---

## Configuration Management

### Configuration Architecture

**Hierarchical structure:**

```
Configuration (root)
├── MainConfiguration
├── DefensiveConfiguration
├── RattingConfiguration
├── MiningConfiguration
├── MissionConfiguration
├── SocialConfiguration
├── CargoConfiguration
├── FleetConfiguration
└── MovementConfiguration
```

### Base Configuration Class

```csharp
public abstract class ConfigurationBase
{
    protected readonly string ConfigPath;

    protected ConfigurationBase(string configPath)
    {
        ConfigPath = configPath;
    }

    public abstract void Load();
    public abstract void Save();
}
```

### Example - RattingConfiguration

```csharp
public interface IRattingConfiguration
{
    bool IsAnomalyMode { get; set; }
    bool RunCombatAnomalies { get; set; }
    bool OnlyUseBookMarks { get; set; }
    int MinimumAnomalyValue { get; set; }
    List<string> AnomalyNamesToRun { get; set; }
}

public class RattingConfiguration : ConfigurationBase, IRattingConfiguration
{
    public bool IsAnomalyMode { get; set; }
    public bool RunCombatAnomalies { get; set; }
    public bool OnlyUseBookMarks { get; set; }
    public int MinimumAnomalyValue { get; set; }
    public List<string> AnomalyNamesToRun { get; set; }

    public RattingConfiguration() : base("Config/Ratting.json")
    {
        AnomalyNamesToRun = new List<string>();
    }

    public override void Load()
    {
        if (!File.Exists(ConfigPath))
        {
            LoadDefaults();
            Save();
            return;
        }

        var json = File.ReadAllText(ConfigPath);
        var config = JsonConvert.DeserializeObject<RattingConfiguration>(json);

        IsAnomalyMode = config.IsAnomalyMode;
        RunCombatAnomalies = config.RunCombatAnomalies;
        OnlyUseBookMarks = config.OnlyUseBookMarks;
        MinimumAnomalyValue = config.MinimumAnomalyValue;
        AnomalyNamesToRun = config.AnomalyNamesToRun;
    }

    public override void Save()
    {
        var json = JsonConvert.SerializeObject(this, Formatting.Indented);
        File.WriteAllText(ConfigPath, json);
    }

    private void LoadDefaults()
    {
        IsAnomalyMode = true;
        RunCombatAnomalies = true;
        OnlyUseBookMarks = false;
        MinimumAnomalyValue = 1000000;
        AnomalyNamesToRun = new List<string>
        {
            "Forsaken Hub",
            "Forsaken Rally Point",
            "Forsaken Den"
        };
    }
}
```

### JSON Configuration File

```json
{
  "IsAnomalyMode": true,
  "RunCombatAnomalies": true,
  "OnlyUseBookMarks": false,
  "MinimumAnomalyValue": 1000000,
  "AnomalyNamesToRun": [
    "Forsaken Hub",
    "Forsaken Rally Point",
    "Forsaken Den",
    "Haven",
    "Sanctum"
  ]
}
```

### Usage in Modules

```csharp
public class Ratting : BehaviorBase
{
    private readonly IRattingConfiguration _rattingConfiguration;

    public Ratting(IRattingConfiguration rattingConfiguration)
    {
        _rattingConfiguration = rattingConfiguration;
    }

    public override void Pulse()
    {
        // Access config values
        if (_rattingConfiguration.IsAnomalyMode)
        {
            // Run anomalies
            RunAnomalies();
        }
        else
        {
            // Run belts
            RunBelts();
        }
    }

    private bool ShouldRunAnomaly(string anomalyName)
    {
        return _rattingConfiguration.AnomalyNamesToRun.Contains(anomalyName);
    }
}
```

---

## ESI API Integration

### HttpClient Pattern

```csharp
public class AllianceCache : ModuleBase, IAllianceCache
{
    // Static HttpClient (singleton pattern for connection pooling)
    private static readonly HttpClient _httpClient = new HttpClient
    {
        Timeout = TimeSpan.FromSeconds(30)
    };

    private readonly ConcurrentDictionary<int, ESIAlliance> _allianceCache =
        new ConcurrentDictionary<int, ESIAlliance>();

    public void GetAllianceInfo(int allianceId)
    {
        // Check cache first
        if (_allianceCache.ContainsKey(allianceId))
            return;

        // Queue background fetch
        ThreadPool.QueueUserWorkItem(TryGetAllianceInfo,
            new AllianceStateInfo(allianceId));
    }

    private void TryGetAllianceInfo(object stateInfo)
    {
        var info = (AllianceStateInfo)stateInfo;

        try
        {
            var url = $"https://esi.evetech.net/latest/alliances/{info.AllianceId}/?datasource=tranquility";

            // Synchronous call (but on thread pool thread)
            var response = _httpClient.GetAsync(url).GetAwaiter().GetResult();

            if (!response.IsSuccessStatusCode)
            {
                if (response.StatusCode == HttpStatusCode.ServiceUnavailable)
                {
                    // ESI down, silently fail
                    return;
                }

                LogMessage("TryGetAllianceInfo", LogSeverityTypes.Critical,
                    $"ESI returned {response.StatusCode} for alliance {info.AllianceId}");
                return;
            }

            var json = response.Content.ReadAsStringAsync().GetAwaiter().GetResult();
            var alliance = JsonConvert.DeserializeObject<ESIAlliance>(json);

            // Cache result
            _allianceCache[info.AllianceId] = alliance;

            // Serialize to disk for persistence
            SaveCacheToDisk();

            LogMessage("TryGetAllianceInfo", LogSeverityTypes.Standard,
                $"Successfully fetched alliance {alliance.Name} ({info.AllianceId})");
        }
        catch (HttpRequestException ex)
        {
            LogException(ex, "TryGetAllianceInfo", $"Failed to fetch alliance {info.AllianceId}");
        }
        catch (System.Threading.Tasks.TaskCanceledException ex)
        {
            LogMessage("TryGetAllianceInfo", LogSeverityTypes.Debug,
                $"Timeout fetching alliance {info.AllianceId}");
        }
        catch (Exception ex)
        {
            LogException(ex, "TryGetAllianceInfo", $"Unexpected error for alliance {info.AllianceId}");
        }
    }
}
```

### ESI Data Models

```csharp
// ESIAlliance.cs
public class ESIAlliance
{
    [JsonProperty("alliance_id")]
    public int AllianceId { get; set; }

    [JsonProperty("name")]
    public string Name { get; set; }

    [JsonProperty("ticker")]
    public string Ticker { get; set; }

    [JsonProperty("executor_corporation_id")]
    public int ExecutorCorporationId { get; set; }

    [JsonProperty("date_founded")]
    public DateTime DateFounded { get; set; }
}

// ESICorporation.cs
public class ESICorporation
{
    [JsonProperty("corporation_id")]
    public int CorporationId { get; set; }

    [JsonProperty("name")]
    public string Name { get; set; }

    [JsonProperty("ticker")]
    public string Ticker { get; set; }

    [JsonProperty("alliance_id")]
    public int? AllianceId { get; set; }

    [JsonProperty("ceo_id")]
    public int CeoId { get; set; }
}
```

### Error Handling Strategy

```csharp
private void TryGetData(object stateInfo)
{
    try
    {
        var response = _httpClient.GetAsync(url).GetAwaiter().GetResult();

        // Check status code
        if (!response.IsSuccessStatusCode)
        {
            switch (response.StatusCode)
            {
                case HttpStatusCode.ServiceUnavailable:
                    // ESI is down - silently fail and retry later
                    return;

                case HttpStatusCode.NotFound:
                    // Entity doesn't exist - log and cache null
                    LogMessage("TryGetData", LogSeverityTypes.Debug,
                        $"Entity not found: {entityId}");
                    _cache[entityId] = null;
                    return;

                case HttpStatusCode.TooManyRequests:
                    // Rate limited - wait and retry
                    Thread.Sleep(1000);
                    // Recursive retry (with counter to prevent infinite loop)
                    if (retryCount < 3)
                        TryGetData(stateInfo, retryCount + 1);
                    return;

                default:
                    // Unexpected error - log as critical
                    LogMessage("TryGetData", LogSeverityTypes.Critical,
                        $"ESI returned {response.StatusCode}");
                    return;
            }
        }

        // Success - parse JSON
        var json = response.Content.ReadAsStringAsync().GetAwaiter().GetResult();
        var data = JsonConvert.DeserializeObject<DataType>(json);

        _cache[entityId] = data;
    }
    catch (HttpRequestException ex)
    {
        // Network error
        LogException(ex, "TryGetData", $"Network error for {entityId}");
    }
    catch (System.Threading.Tasks.TaskCanceledException ex)
    {
        // Timeout
        LogMessage("TryGetData", LogSeverityTypes.Debug,
            $"Timeout for {entityId}");
    }
    catch (JsonException ex)
    {
        // JSON parse error
        LogException(ex, "TryGetData", $"JSON parse error for {entityId}");
    }
}
```

### Cache Persistence

```csharp
private void SaveCacheToDisk()
{
    try
    {
        var serializer = new DataContractSerializer(typeof(ConcurrentDictionary<int, ESIAlliance>));

        using (var stream = File.Create("Cache/Alliances.bin"))
        {
            Serializer.Serialize(stream, _allianceCache);
        }

        LogMessage("SaveCacheToDisk", LogSeverityTypes.Debug,
            $"Saved {_allianceCache.Count} alliances to disk.");
    }
    catch (Exception ex)
    {
        LogException(ex, "SaveCacheToDisk", "Failed to save alliance cache.");
    }
}

private void LoadCacheFromDisk()
{
    if (!File.Exists("Cache/Alliances.bin"))
        return;

    try
    {
        using (var stream = File.OpenRead("Cache/Alliances.bin"))
        {
            _allianceCache = Serializer.Deserialize<ConcurrentDictionary<int, ESIAlliance>>(stream);
        }

        LogMessage("LoadCacheFromDisk", LogSeverityTypes.Standard,
            $"Loaded {_allianceCache.Count} alliances from disk.");
    }
    catch (Exception ex)
    {
        LogException(ex, "LoadCacheFromDisk", "Failed to load alliance cache.");
    }
}
```

---

## Event System

### Custom Event Args

```csharp
// SessionChangedEventArgs.cs
public class SessionChangedEventArgs : EventArgs
{
    public bool InStation { get; set; }
    public bool InSpace { get; set; }
    public int SolarSystemId { get; set; }
    public string SolarSystemName { get; set; }
}

// WalletStatisticsUpdatedEventArgs.cs
public class WalletStatisticsUpdatedEventArgs : EventArgs
{
    public double IskPerHour { get; set; }
    public double IskThisSession { get; set; }
    public TimeSpan SessionTime { get; set; }
}
```

### Event Publishers

```csharp
public class MeCache : ModuleBase, IMeCache
{
    // Event declaration
    public event EventHandler<SessionChangedEventArgs> SessionChanged;

    private int _lastSolarSystemId = -1;
    private bool _wasInStation = false;

    public override void Pulse()
    {
        if (!ShouldPulse()) return;

        // Check for session change
        bool sessionChanged = false;

        if (_meCache.SolarSystemId != _lastSolarSystemId)
        {
            sessionChanged = true;
            _lastSolarSystemId = _meCache.SolarSystemId;
        }

        if (_meCache.InStation != _wasInStation)
        {
            sessionChanged = true;
            _wasInStation = _meCache.InStation;
        }

        if (sessionChanged)
        {
            // Raise event
            SessionChanged?.Invoke(this, new SessionChangedEventArgs
            {
                InStation = _meCache.InStation,
                InSpace = _meCache.InSpace,
                SolarSystemId = _meCache.SolarSystemId,
                SolarSystemName = _meCache.SolarSystemName
            });
        }
    }
}
```

### Event Subscribers

```csharp
public class Statistics : ModuleBase
{
    private readonly IMeCache _meCache;

    public Statistics(IMeCache meCache)
    {
        _meCache = meCache;

        // Subscribe to session changed event
        _meCache.SessionChanged += OnSessionChanged;
    }

    private void OnSessionChanged(object sender, SessionChangedEventArgs e)
    {
        LogMessage("OnSessionChanged", LogSeverityTypes.Standard,
            $"Session changed: {(e.InStation ? "In Station" : "In Space")}, System: {e.SolarSystemName}");

        // Reset statistics for new session
        if (e.InStation)
        {
            ResetSessionStatistics();
        }
    }
}
```

---

## Performance and Profiling

### Built-in Profiling

Every module has profiling built-in:

```csharp
public override void Pulse()
{
    if (!ShouldPulse()) return;

    StartPulseProfiling();  // Start stopwatch

    // Do work...

    EndPulseProfiling();  // Stop stopwatch and log if too slow
}

protected void EndPulseProfiling()
{
    _pulseProfiler.Stop();

    if (_pulseProfiler.ElapsedMilliseconds > 100)
    {
        LogMessage("Pulse", LogSeverityTypes.Critical,
            $"{ModuleName} pulse took {_pulseProfiler.ElapsedMilliseconds}ms! (threshold: 100ms)");
    }

    // Store in profiling stats
    Core.Metatron.Statistics.RecordPulseTime(ModuleName, _pulseProfiler.Elapsed);
}
```

### Performance Statistics

```csharp
public class Statistics : ModuleBase
{
    private readonly Dictionary<string, List<TimeSpan>> _pulseTimesByModule =
        new Dictionary<string, List<TimeSpan>>();

    public void RecordPulseTime(string moduleName, TimeSpan pulseTime)
    {
        if (!_pulseTimesByModule.ContainsKey(moduleName))
        {
            _pulseTimesByModule[moduleName] = new List<TimeSpan>();
        }

        _pulseTimesByModule[moduleName].Add(pulseTime);

        // Keep only last 100 samples
        if (_pulseTimesByModule[moduleName].Count > 100)
        {
            _pulseTimesByModule[moduleName].RemoveAt(0);
        }
    }

    public Dictionary<string, double> GetAveragePulseTimesByModule()
    {
        return _pulseTimesByModule.ToDictionary(
            kvp => kvp.Key,
            kvp => kvp.Value.Average(t => t.TotalMilliseconds)
        );
    }

    public void LogPerformanceReport()
    {
        var report = GetAveragePulseTimesByModule();

        LogMessage("LogPerformanceReport", LogSeverityTypes.Standard,
            "=== Performance Report ===");

        foreach (var kvp in report.OrderByDescending(kvp => kvp.Value))
        {
            LogMessage("LogPerformanceReport", LogSeverityTypes.Standard,
                $"{kvp.Key}: {kvp.Value:F2}ms average");
        }
    }
}
```

**Example Output:**

```
=== Performance Report ===
EntityProvider: 2.34ms average
Movement: 1.87ms average
Ship: 1.45ms average
Offensive: 0.98ms average
Targeting: 0.67ms average
Ratting: 0.45ms average
Defense: 0.23ms average
```

### Optimization Techniques

**1. LINQ Optimization:**

```csharp
// ❌ SLOW - Multiple iterations
var npcs = _entityProvider.EntityWrappers
    .Where(e => e.IsNPC)
    .ToList()
    .Where(e => e.Distance < 100000)
    .ToList()
    .OrderBy(e => e.Distance)
    .ToList();

// ✅ FAST - Single iteration
var npcs = _entityProvider.EntityWrappers
    .Where(e => e.IsNPC && e.Distance < 100000)
    .OrderBy(e => e.Distance)
    .ToList();
```

**2. Caching:**

```csharp
// Cache expensive calculations
private double _maxWeaponRange = -1;

public double GetMaximumWeaponRange()
{
    if (_maxWeaponRange < 0)
    {
        _maxWeaponRange = _turretModules
            .Union(_launcherModules)
            .Max(m => m.OptimalRange + m.FalloffRange);
    }

    return _maxWeaponRange;
}

// Invalidate cache when modules change
private void OnModulesChanged()
{
    _maxWeaponRange = -1;  // Force recalculation next time
}
```

**3. Early Returns:**

```csharp
public override void Pulse()
{
    if (!ShouldPulse()) return;  // Early return

    if (_meCache.InWarp) return;  // Early return

    if (!_ship.IsInventoryReady) return;  // Early return

    // Do expensive work only if necessary
    ProcessCombat();
}
```

---

## Code Organization

### File Naming Conventions

```
Modules: [ModuleName].cs
  - Movement.cs
  - Offensive.cs
  - Ratting.cs

Interfaces: I[InterfaceName].cs
  - IMovement.cs
  - IShip.cs
  - IEntityProvider.cs

Configuration: [Name]Configuration.cs
  - RattingConfiguration.cs
  - DefensiveConfiguration.cs

Event Args: [Name]EventArgs.cs
  - SessionChangedEventArgs.cs
  - WalletStatisticsUpdatedEventArgs.cs

Enums: [Name].cs (no special prefix)
  - DestinationTypes.cs
  - ApproachTypes.cs
  - TargetTypes.cs
```

### Folder Structure Best Practices

```
Metatron/
├── ActionModules/           # One module per file
│   ├── Movement.cs
│   ├── Offensive.cs
│   ├── Defense.cs
│   ├── Targeting.cs
│   ├── Drones.cs
│   └── NonOffensive.cs
│
├── BehaviorModules/         # One behavior per file
│   ├── Ratting.cs
│   ├── Mining.cs
│   ├── Salvaging.cs
│   ├── MissionRunner.cs
│   └── CombatAssist.cs
│
├── Core/                    # Core systems
│   ├── Ship.cs
│   ├── EntityProvider.cs
│   ├── EntityCache.cs
│   ├── AllianceCache.cs
│   ├── CorporationCache.cs
│   ├── MeCache.cs
│   └── TargetQueue.cs
│
└── Program.cs               # Entry point
```

### Namespace Conventions

```csharp
// Action modules
namespace Metatron.ActionModules
{
    internal sealed class Movement : ModuleBase, IMovement
    {
        // ...
    }
}

// Behavior modules
namespace Metatron.BehaviorModules
{
    internal class Ratting : BehaviorBase
    {
        // ...
    }
}

// Core systems
namespace Metatron.Core
{
    internal sealed class Ship : ModuleBase, IShip
    {
        // ...
    }
}

// Interfaces
namespace Metatron.Core.Interfaces
{
    public interface IMovement
    {
        // ...
    }
}

// Configuration
namespace Metatron.Core.Config
{
    public class RattingConfiguration : ConfigurationBase
    {
        // ...
    }
}
```

---

## Design Patterns Used

### 1. Module Pattern

Every feature is a self-contained module:

```csharp
public abstract class ModuleBase
{
    public string ModuleName { get; protected set; }
    public abstract void Pulse();
}

// Usage
public class Movement : ModuleBase
{
    public override void Pulse()
    {
        // Module logic
    }
}
```

### 2. Dependency Injection

Constructor injection for all dependencies:

```csharp
public class Ratting : BehaviorBase
{
    private readonly IMovement _movement;
    private readonly IShip _ship;

    public Ratting(IMovement movement, IShip ship)
    {
        _movement = movement;
        _ship = ship;
    }
}
```

### 3. Observer Pattern

Event-driven communication:

```csharp
// Publisher
public event EventHandler<SessionChangedEventArgs> SessionChanged;

SessionChanged?.Invoke(this, new SessionChangedEventArgs { ... });

// Subscriber
_meCache.SessionChanged += OnSessionChanged;
```

### 4. Strategy Pattern

Interchangeable behavior modules:

```csharp
public interface IBehavior
{
    void Pulse();
}

public class Ratting : IBehavior { }
public class Mining : IBehavior { }
public class Salvaging : IBehavior { }

// Usage
IBehavior activeBehavior = BehaviorManager.BehaviorsToPulse[CurrentBotMode];
activeBehavior.Pulse();
```

### 5. Queue Pattern

Queue-based state management:

```csharp
// Target queue
_targetQueue.QueueTarget(new QueueTarget { ... });
var target = _targetQueue.GetHighestPriorityTarget(TargetTypes.Kill);

// Destination queue
_destinationQueue.Add(new Destination { ... });
ProcessDestinationQueue();
```

### 6. State Machine Pattern

State-driven behaviors:

```csharp
private RattingStates _rattingState = RattingStates.Idle;

private void ProcessPulseState()
{
    switch (_rattingState)
    {
        case RattingStates.Idle:
            // ...
            break;
        case RattingStates.Traveling:
            // ...
            break;
        case RattingStates.KillRats:
            // ...
            break;
    }
}
```

### 7. Singleton Pattern

Single instance of core services:

```csharp
public static class Core
{
    public static IMovement Movement { get; set; }
    public static IShip Ship { get; set; }
    public static IEntityProvider EntityProvider { get; set; }
}

// Usage
Core.Metatron.Movement.QueueDestination(...);
```

### 8. Factory Pattern

Entity wrapper creation:

```csharp
public class EntityWrapper : IEntityWrapper
{
    public EntityWrapper(ISXEVE.Entity entity)
    {
        // Wrap ISXEVE entity
    }
}

// Usage
var wrapper = new EntityWrapper(isxeveEntity);
```

### 9. Repository Pattern

Caching and data access:

```csharp
public interface IAllianceCache
{
    ESIAlliance GetAlliance(int allianceId);
}

public class AllianceCache : IAllianceCache
{
    private readonly ConcurrentDictionary<int, ESIAlliance> _cache;

    public ESIAlliance GetAlliance(int allianceId)
    {
        if (_cache.TryGetValue(allianceId, out var alliance))
            return alliance;

        // Fetch from ESI...
        return null;
    }
}
```

### 10. Facade Pattern

Core singleton provides simple interface:

```csharp
public static class Core
{
    // Complex subsystems hidden behind simple interface
    public static IMovement Movement { get; set; }
    public static IShip Ship { get; set; }
    public static IEntityProvider EntityProvider { get; set; }
}

// Simple usage
Core.Metatron.Movement.QueueDestination(...);
Core.Metatron.Ship.ActivateModuleList(...);
```

---

## Lessons Learned

### What Worked Well

✅ **Module-based architecture**
- Easy to add new behaviors
- Clear separation of concerns
- Maintainable by team

✅ **Pulse-driven execution**
- No blocking operations
- Responsive to game state
- Easy to reason about

✅ **Dependency injection**
- Testable code
- Clear dependencies
- Easy to mock for tests

✅ **Queue-based systems**
- Simple state management
- Priority-based processing
- Easy to debug

✅ **Interface-based contracts**
- Flexible implementations
- Easy to swap modules
- Clear API boundaries

### What Didn't Work

❌ **Global singleton (Core.Metatron)**
- Tight coupling
- Hard to test
- Should use proper DI container

❌ **Mixed sync/async patterns**
- Confusing for new developers
- Should pick one approach
- async/await not consistently used

❌ **Large module files**
- Some modules 2000+ lines
- Hard to navigate
- Should split into smaller classes

❌ **Inconsistent error handling**
- Some modules throw exceptions
- Others return null/false
- Should standardize

❌ **Limited unit testing**
- Most code not tested
- Hard to refactor safely
- Should have written tests from start

### Recommendations for New Projects

**1. Use a DI Container**

Instead of manual registration:

```csharp
// ❌ Manual (current approach)
var movement = new Movement(entityProvider, meCache, ship);
var ratting = new Ratting(movement, ship, ...);

// ✅ DI Container (better)
services.AddSingleton<IMovement, Movement>();
services.AddSingleton<IRatting, Ratting>();

var serviceProvider = services.BuildServiceProvider();
var ratting = serviceProvider.GetService<IRatting>();
```

**2. Choose async/await or ThreadPool (not both)**

```csharp
// ✅ Pick one approach and stick to it

// Option 1: Full async/await
public async Task PulseAsync()
{
    var alliance = await GetAllianceAsync(allianceId);
}

// Option 2: ThreadPool only
public void Pulse()
{
    ThreadPool.QueueUserWorkItem(GetAlliance, allianceId);
}
```

**3. Split large files**

```csharp
// ✅ Better organization

// Ratting.cs (main logic)
public partial class Ratting : BehaviorBase
{
    public override void Pulse() { }
}

// Ratting.EntityManagement.cs (entity logic)
public partial class Ratting
{
    private void GetEntities() { }
    private void CheckNpcs() { }
}

// Ratting.AnomalySelection.cs (anomaly logic)
public partial class Ratting
{
    private void SelectAnomaly() { }
    private void MoveToAnomaly() { }
}
```

**4. Standardize error handling**

```csharp
// ✅ Consistent error handling

public class Result<T>
{
    public bool Success { get; set; }
    public T Value { get; set; }
    public string Error { get; set; }
}

public Result<QueueTarget> GetHighestPriorityTarget()
{
    try
    {
        var target = _targets.OrderByDescending(t => t.Priority).FirstOrDefault();

        if (target == null)
        {
            return new Result<QueueTarget>
            {
                Success = false,
                Error = "No targets in queue"
            };
        }

        return new Result<QueueTarget>
        {
            Success = true,
            Value = target
        };
    }
    catch (Exception ex)
    {
        return new Result<QueueTarget>
        {
            Success = false,
            Error = ex.Message
        };
    }
}
```

**5. Write unit tests from start**

```csharp
// ✅ Unit test example

[TestClass]
public class TargetQueueTests
{
    [TestMethod]
    public void QueueTarget_AddsDuplicateOnce()
    {
        // Arrange
        var queue = new TargetQueue();
        var target = new QueueTarget { Id = 123, Priority = 100 };

        // Act
        queue.QueueTarget(target);
        queue.QueueTarget(target);  // Duplicate

        // Assert
        Assert.AreEqual(1, queue.Targets.Count);
    }

    [TestMethod]
    public void GetHighestPriorityTarget_ReturnsPriority()
    {
        // Arrange
        var queue = new TargetQueue();
        queue.QueueTarget(new QueueTarget { Id = 1, Priority = 50 });
        queue.QueueTarget(new QueueTarget { Id = 2, Priority = 100 });
        queue.QueueTarget(new QueueTarget { Id = 3, Priority = 75 });

        // Act
        var target = queue.GetHighestPriorityTarget(TargetTypes.Kill);

        // Assert
        Assert.AreEqual(2, target.Id);  // Highest priority
    }
}
```

---

## Summary

**Metatron demonstrates a successful .NET architecture for EVE Online automation:**

### Key Takeaways

1. **Module-based design** works well for game automation
2. **Pulse-driven execution** avoids blocking and maintains responsiveness
3. **Dependency injection** makes code testable and maintainable
4. **Queue-based systems** simplify state management
5. **Interface-based contracts** provide flexibility
6. **Performance profiling** is essential for real-time systems
7. **ESI integration** requires proper async/thread handling
8. **Configuration management** enables user customization

### By The Numbers

- **3,000+ files** - Large codebase but well organized
- **100,000+ lines** - Comprehensive feature set
- **< 5ms pulse time** - 100x faster than LavishScript equivalent
- **7+ years** - Proven longevity and maintainability
- **100+ users** - Successfully supports community

### Final Thoughts

Metatron shows that .NET is the right choice for complex EVE automation:
- ✅ Performance: 100x faster than scripts
- ✅ Maintainability: Team of developers can collaborate
- ✅ Extensibility: Easy to add new features
- ✅ Reliability: Runs 24/7 without issues

**This architecture can serve as a blueprint for other EVE bot projects.**

---

**Previous Document:** `33_When_to_Use_DotNet_Instead.md` - Decision criteria for .NET vs LavishScript

**Next Document:** `35_Complete_ISXEVE_Object_Method_Index.md` - Reference index of all ISXEVE objects

---

*Document complete. Architecture overview captured from real-world production bot.*
