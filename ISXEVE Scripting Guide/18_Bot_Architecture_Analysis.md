# Bot Architecture Analysis

**Purpose:** Architecture analysis of EVEBot and [Tehbot](https://github.com/isxGames/Tehbot) production bots
**Audience:** Developers studying production bot architectures and design patterns

---

## Table of Contents

### EVEBot Analysis
1. [EVEBot Architecture Analysis](#evebot-architecture-analysis)
2. [EVEBot Overview](#evebot-overview)
3. [Project Structure](#project-structure)
4. [Initialization System](#initialization-system)
5. [Core Object Architecture](#core-object-architecture)
6. [Behavior System](#behavior-system)
7. [Pulse and Timing System](#pulse-and-timing-system)
8. [Configuration System](#configuration-system)
9. [Master-Slave Coordination](#master-slave-coordination)
10. [Key Design Patterns](#key-design-patterns)
11. [Strengths and Weaknesses](#strengths-and-weaknesses)
12. [Code Reuse Guide](#code-reuse-guide)

### Yamfa (Reference Pointer)
13. [Yamfa Fleet Assist (Reference)](#yamfa-fleet-assist-reference)

### Tehbot Analysis
14. [Tehbot Combat Analysis](#tehbot-combat-analysis)
15. [Tehbot Overview](#tehbot-overview)
16. [StateQueue Architecture](#statequeue-architecture)
17. [MiniMode System](#minimode-system)
18. [Dynamic Behavior and MiniMode Registry](#dynamic-behavior-and-minimode-registry)
19. [Mission Data Files as Per-Mission Scripts](#mission-data-files-as-per-mission-scripts)
20. [Comparison: Tehbot vs EVEBot vs Yamfa](#comparison-tehbot-vs-evebot-vs-yamfa)
21. [Key Patterns](#key-patterns)
22. [When to Use Which Architecture](#when-to-use-which-architecture)
23. [Community Takeaways](#community-takeaways)

---

## EVEBot Architecture Analysis

---

## EVEBot Overview

### What is EVEBot?

**[EVEBot](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable)** is the most comprehensive and mature open-source LavishScript bot for EVE Online. It has been in development since ~2008 and supports:

- **Mining**: Solo and fleet mining with Orca support
- **Hauling**: Jetcan pickup, station-to-station transport
- **Combat**: Ratting, missions, anomalies
- **Logistics**: Guardian support, remote reps
- **Freighting**: Large-scale hauling operations
- **Multi-boxing**: Master/slave fleet coordination

### Why Study EVEBot?

1. **Proven Architecture**: Battle-tested over 15+ years
2. **Comprehensive Coverage**: Shows solutions to nearly every EVE automation problem
3. **Modular Design**: Clean separation of concerns
4. **Real-World Patterns**: Actual production code, not toy examples
5. **Community Foundation**: Base for hundreds of derivative bots

**Critical Note**: EVEBot was last actively maintained around 2015-2017. Some patterns are outdated, but the architecture remains sound.

---

## Project Structure

### Directory Layout

```
EVEBot/
├── EVEBot.iss                    # Main entry point
├── Branches/
│   └── Stable/
│       ├── EVEBot.iss            # Stable branch entry
│       ├── core/                 # Core objects
│       │   ├── defines.iss       # Constants and #defines
│       │   ├── obj_EVEBot.iss    # Main bot controller
│       │   ├── obj_Logger.iss    # Logging system
│       │   ├── obj_Configuration.iss  # Config system
│       │   ├── obj_Ship.iss      # Ship control
│       │   ├── obj_Combat.iss    # Combat system
│       │   ├── obj_Asteroids.iss # Asteroid selection
│       │   ├── obj_Cargo.iss     # Cargo management
│       │   ├── obj_Station.iss   # Station operations
│       │   ├── obj_Targets.iss   # Target management
│       │   ├── obj_Drones.iss    # Drone control
│       │   ├── obj_Fleet.iss     # Fleet coordination
│       │   ├── obj_Social.iss    # Social/standings
│       │   ├── obj_Autopilot.iss # Navigation
│       │   ├── obj_Inventory.iss # Inventory windows
│       │   ├── obj_Bookmarks.iss # Bookmark management
│       │   ├── obj_Safespots.iss # Safe spot generation
│       │   ├── obj_Belts.iss     # Belt management
│       │   ├── obj_Agents.iss    # Agent interaction
│       │   ├── obj_Missions.iss  # Mission running
│       │   ├── obj_Market.iss    # Market operations
│       │   └── ... (30+ core objects total)
│       │
│       ├── Behaviors/            # Bot behaviors
│       │   ├── obj_Miner.iss     # Mining behavior
│       │   ├── obj_Hauler.iss    # Hauling behavior
│       │   ├── obj_Orca.iss      # Orca support
│       │   ├── obj_Missioneer.iss# Mission running
│       │   ├── obj_Ratter.iss    # NPC ratting
│       │   ├── obj_AnomRatter.iss# Anomaly ratting
│       │   ├── obj_Guardian.iss  # Logistics
│       │   ├── obj_Freighter.iss # Freighter hauling
│       │   ├── obj_Courier.iss   # Courier missions
│       │   ├── obj_Scavenger.iss # Salvaging
│       │   └── obj_StealthHauler.iss # Covert ops hauling
│       │
│       └── Threads/              # Background threads
│           ├── Defense_Drone.iss # Auto-defend with drones
│           └── ...
│
├── Data/                         # Data files
│   ├── MissionXML/              # Mission databases
│   └── ...
│
└── Config/                       # Configuration files
    └── EVEBot.xml               # Main configuration
```

**Logger implementations:** For the full `obj_Logger` code (EVEBot and Tehbot variants with log levels, file output, and color-coded console), see [20_Debugging_And_Troubleshooting.md](20_Debugging_And_Troubleshooting.md) under Logging Frameworks.

### File Organization Philosophy

**Core Objects** (`core/`):
- **Purpose**: Reusable components that provide services
- **Pattern**: Single responsibility - each object does ONE thing well
- **Examples**: obj_Ship (ship control), obj_Cargo (cargo operations)
- **Usage**: Called by behaviors as needed

**Behaviors** (`Behaviors/`):
- **Purpose**: Complete bot modes (Miner, Hauler, etc.)
- **Pattern**: State machine with SetState() and ProcessState()
- **Examples**: obj_Miner (mining operations), obj_Hauler (hauling loop)
- **Usage**: User selects one behavior to run

**Threads** (`Threads/`):
- **Purpose**: Always-on background tasks
- **Pattern**: Independent scripts that run in parallel
- **Examples**: Defense_Drone.iss (auto-defend)
- **Usage**: Automatically launched at startup

---

## Initialization System

### Startup Sequence

File: `EVEBot.iss` (lines 1-200+)

```lavishscript
function main()
{
    // 1. CHECK PREREQUISITES
    #if ${ISXEVE(exists)}
    #else
        #error EVEBot requires ISXEVE to be loaded before running
    #endif

    // 2. INCLUDE FILES
    #include core/defines.iss              // Constants first

    // 3. BASE REQUIREMENTS (order matters!)
    #include core/obj_Logger.iss           // Logging first (everyone needs it)
    #include core/obj_Configuration.iss    // Config second
    #include core/obj_EVEBot.iss           // Main controller third

    // 4. SUPPORT FILES (alphabetical, no dependencies)
    #include core/obj_Asteroids.iss
    #include core/obj_Ship.iss
    #include core/obj_Station.iss
    #include core/obj_Cargo.iss
    // ... 25+ more includes

    // 5. BEHAVIORS (dynamically loaded)
    #includeoptional Behaviors/_includes.iss

    // 6. SET TURBO MODE
    Turbo 4000    // Process 4000 frames per iteration

    // 7. DISABLE ENTITY CACHE (performance)
    ISXEVE:Debug_SetEntityCacheDisabled[TRUE]

    // 8. CREATE CORE OBJECTS
    call CreateVariable Logger obj_Logger global

    // 9. LOAD CONFIGURATION
    declarevariable Config obj_Configuration script

    // 10. CREATE MAIN BOT CONTROLLER
    call CreateVariable EVEBot obj_EVEBot global

    // 11. LOAD CORE OBJECTS (30+ objects)
    declarevariable Ship obj_Ship script
    declarevariable Station obj_Station script
    declarevariable Cargo obj_Cargo script
    // ...

    // 12. LOAD BEHAVIORS (dynamic discovery)
    Logger:Log[" Loading Behaviors...", LOG_ECHOTOO]
    call LoadBehaviors "Behaviors" "${Script.CurrentDirectory}/Behaviors"

    // 13. LOAD BACKGROUND THREADS
    Logger:Log[" Loading Threads...", LOG_ECHOTOO]
    call LoadThreads "Threads" "${Script.CurrentDirectory}/Threads"

    // 14. MARK BOT AS LOADED
    EVEBot.Loaded:Set[TRUE]

    // 15. MAIN LOOP
    while TRUE
    {
        call ${Config.Common.CurrentBehavior}.ProcessState
        wait 1
    }
}
```

### Dynamic Behavior Loading

File: `EVEBot.iss` (lines 59-84)

```lavishscript
function LoadBehaviors(string Label, string Path)
{
    variable int Position = 0
    variable filelist Files
    variable string NewObjectName
    variable string NewVarName

    // Get all .iss files in Behaviors folder
    Files:GetFiles["${Path}"]

    while (${Position:Inc}<=${Files.Files})
    {
        // Skip special files
        if ${Files.File[${Position}].Filename.NotEqual["_includes.iss"]} &&
           ${Files.File[${Position}].Filename.NotEqual["_variables.iss"]}
        {
            // Extract object name from filename
            // "obj_Miner.iss" -> "obj_Miner"
            NewObjectName:Set[${Files.File[${Position}].Filename.Left[-4]}]

            // Extract variable name
            // "obj_Miner" -> "Miner"
            NewVarName:Set[${NewObjectName.Right[-4]}]

            Logger:Log["   ${NewVarName}", LOG_ECHOTOO]

            // Dynamically create global variable
            call CreateVariable ${NewVarName} ${NewObjectName} script

            // Track loaded behaviors
            Behaviors.Loaded:Add[${NewVarName}]
        }
    }
}
```

**Why This Matters**:
- Add new behavior by simply dropping .iss file in Behaviors folder
- No need to modify main script
- Clean plugin architecture

### Thread Loading

File: `EVEBot.iss` (lines 86-111)

```lavishscript
function LoadThreads(string Label, string Path)
{
    variable int Position = 0
    variable filelist Files

    Files:GetFiles["${Path}"]

    while (${Position:Inc}<=${Files.Files})
    {
        if ${Files.File[${Position}].Size} > 0
        {
            Log:Concat["${Files.File[${Position}].Filename.Left[-4]} "]

            // Launch thread in background
            TimedCommand 0 runscript "${Files.File[${Position}].FullPath}" ${Script.Filename}
            wait 1
        }
    }
}
```

**Pattern**: Threads are independent scripts that run in parallel with main bot.

---

## Core Object Architecture

### Base Class Pattern

File: `core/obj_BaseClass.iss` (standard pattern)

```lavishscript
objectdef obj_BaseClass
{
    variable string LogPrefix = "BaseClass"
    variable obj_PulseTimer PulseTimer

    method Initialize()
    {
        LogPrefix:Set["${This.ObjectName}"]
        PulseTimer:SetIntervals[1.0,1.0]
    }

    method Shutdown()
    {
        // Override in derived classes
    }
}
```

**All core objects inherit from obj_BaseClass**:
- Provides standard LogPrefix
- Provides PulseTimer for timing
- Standardizes Initialize/Shutdown pattern

### EVEBot Main Controller

File: `core/obj_EVEBot.iss` (lines 10-300+)

```lavishscript
objectdef obj_EVEBot inherits obj_BaseClass
{
    // STATE VARIABLES
    variable bool ReturnToStation = FALSE     // Emergency flag
    variable bool _Paused = TRUE              // Pause state
    variable bool Disabled = FALSE            // Complete shutdown flag
    variable bool Loaded = FALSE              // Initialization complete

    // THREAD MANAGEMENT
    variable index:string Threads             // Track background threads

    // MASTER/SLAVE SUPPORT
    variable obj_PulseTimer LastMasterQuery
    variable string MasterName
    variable bool IsMaster = FALSE

    method Initialize()
    {
        LogPrefix:Set["${This.ObjectName}"]
        PulseTimer:SetIntervals[4.0,5.0]      // 4-5 second pulse

        // Register custom events
        LavishScript:RegisterEvent[EVENT_EVEBOT_ONFRAME]
        LavishScript:RegisterEvent[EVENT_EVEBOT_ONFRAME_INSPACE]
        Event[EVENT_ONFRAME]:AttachAtom[This:Pulse]

        // Master/slave events
        LavishScript:RegisterEvent[EVEBot_Master_Query]
        Event[EVEBot_Master_Query]:AttachAtom[This:Event_Master_Query]

        LavishScript:RegisterEvent[EVEBot_Master_Notify]
        Event[EVEBot_Master_Notify]:AttachAtom[This:Event_Master_Notify]
    }

    method Pulse()
    {
        if !${This.Loaded} || ${This.Disabled}
        {
            return
        }

        if ${This.PulseTimer.Ready}
        {
            // MASTER/SLAVE COORDINATION
            if ${Config.Miner.GroupMode}
            {
                if ${This.MasterName.Length} == 0 && ${This.LastMasterQuery.Ready}
                {
                    // Query for master
                    relay all -event EVEBot_Master_Query
                }
            }

            // UI RENDERING CONTROL
            if ${Config.Common.DisableUI} && ${EVE.IsUIDisplayOn}
            {
                EVE:ToggleUIDisplay
                Logger:Log["Disabling UI Rendering"]
            }

            // 3D RENDERING CONTROL (performance)
            if ${Me.InSpace}
            {
                if ${Config.Common.Disable3D} && ${EVE.Is3DDisplayOn}
                {
                    EVE:Toggle3DDisplay
                    Logger:Log["Disabling 3D Rendering"]
                }
            }

            if !${This._Paused}
            {
                // AUTO-CLOSE DIALOGS
                EVE:CloseAllMessageBoxes
                EVE:CloseAllChatInvites

                // DOWNTIME DETECTION
                if !${This.ReturnToStation}
                {
                    if ( ${This.GameHour} == 10 &&
                       ( ${This.GameMinute} >= 50 && ${This.GameMinute} <= 57) )
                    {
                        Logger:Log["EVE downtime approaching, pausing operations", LOG_CRITICAL]
                        This.ReturnToStation:Set[TRUE]
                    }

                    // MAX RUNTIME CHECK
                    variable int Hours = ${Math.Calc[(${Script.RunningTime}/1000/60/60)%60].Int}

                    if ${Config.Common.MaxRuntime} > 0 && ${Config.Common.MaxRuntime} <= ${Hours}
                    {
                        Logger:Log["Maximum runtime exceeded, pausing operations", LOG_CRITICAL]
                        This.ReturnToStation:Set[TRUE]
                    }
                }
            }

            This.PulseTimer:Update
        }
    }

    // PAUSE/RESUME SYSTEM
    member:bool Paused()
    {
        if ${This._Paused} || ${Script.Paused}
        {
            return TRUE
        }

        if !${ISXEVE.IsSafe}    // Session changing
        {
            return TRUE
        }

        return FALSE
    }

    method Pause(string Reason)
    {
        This._Paused:Set[TRUE]
        This:PauseThreads
        Logger:Log["\agPaused\ax: ${Reason}", LOG_ECHOTOO]
        Script:Pause
    }

    method Resume(string Reason)
    {
        This.ReturnToStation:Set[FALSE]
        Script:Resume
        This:ResumeThreads
        This._Paused:Set[FALSE]
        Logger:Log["Resumed: ${Reason}", LOG_ECHOTOO]
    }

    // THREAD MANAGEMENT
    method PauseThreads()
    {
        variable int i
        for (i:Set[1]; ${i} <= ${Threads.Used}; i:Inc)
        {
            Logger:Log[" Pausing ${Threads.Get[${i}]} thread..."]
            Script[${Threads.Get[${i}]}]:Pause
        }
    }

    method ResumeThreads()
    {
        variable int i
        for (i:Set[1]; ${i} <= ${Threads.Used}; i:Inc)
        {
            Logger:Log[" Resuming ${Threads.Get[${i}]} thread..."]
            Script[${Threads.Get[${i}]}]:Resume
        }
    }

    // UTILITY METHODS
    member:string MetersToKM_Str(float64 Meters)
    {
        if ${Meters(exists)} && ${Meters} > 0
        {
            return "${Math.Calc[${Meters} / 1000].Centi}km"
        }
        else
        {
            return "0km"
        }
    }

    member:string ISK_To_Str(float64 Total)
    {
        if ${Total(exists)}
        {
            if ${Total} > 1000000000
            {
                return "${Math.Calc[${Total}/100000000].Precision[3]}b isk"
            }
            elseif ${Total} > 1000000
            {
                return "${Math.Calc[${Total}/1000000].Precision[2]}m isk"
            }
            elseif ${Total} > 1000
            {
                return "${Math.Calc[${Total}/1000].Int}k isk"
            }
            else
            {
                return "${Total.Int} isk"
            }
        }
        return "0 isk"
    }
}
```

**Key Responsibilities**:
1. **Lifecycle Management**: Pause/Resume, thread control
2. **Emergency Handling**: ReturnToStation flag for hard stops
3. **Performance**: UI/3D rendering toggle
4. **Downtime Detection**: Auto-pause before server restart
5. **Utility Functions**: Formatting helpers

---

## Behavior System

### Behavior Interface

All behaviors follow a common generic template built around a `CurrentState` string, a `NextPulse` timer, and the `Initialize` / `Pulse` / `SetState` / `ProcessState` lifecycle inherited from `obj_BaseClass`.

The canonical behavior template, with full `obj_BaseClass` inheritance, `Initialize`/`Shutdown` lifecycle, `SetState`/`ProcessState` dispatch, and safety-check ordering, is shown under [Example: Creating New Behavior](#example-creating-new-behavior) later in this guide.

### Miner Behavior Example

File: `Behaviors/obj_Miner.iss` (simplified)

```lavishscript
objectdef obj_Miner inherits obj_BaseClass
{
    variable string CurrentState = "IDLE"
    variable time NextPulse
    variable int PulseIntervalInSeconds = 2

    variable int64 CurrentAsteroid = 0
    variable bool AtBookmark = FALSE

    method SetState()
    {
        // SAFETY FIRST
        if ${Me.InStation}
        {
            This.CurrentState:Set["INSTATION"]
            return
        }

        if !${Me.InSpace}
        {
            This.CurrentState:Set["UNKNOWN"]
            return
        }

        // EMERGENCY CHECKS
        if ${EVEBot.ReturnToStation}
        {
            This.CurrentState:Set["FLEE"]
            return
        }

        if ${Social.PossibleHostiles}
        {
            This.CurrentState:Set["FLEE"]
            relay all -event EVEBot_HARDSTOP "${Me.Name} - ${Config.Common.CurrentBehavior} (Hostiles)"
            EVEBot.ReturnToStation:Set[TRUE]
            return
        }

        if ${Ship.IsPod}
        {
            This.CurrentState:Set["FLEE"]
            relay all -event EVEBot_HARDSTOP "${Me.Name} - ${Config.Common.CurrentBehavior} (InPod)"
            EVEBot.ReturnToStation:Set[TRUE]
            return
        }

        // NORMAL OPERATION
        if ${Ship.CargoFreeSpace} < ${Config.Miner.CargoThreshold}
        {
            This.CurrentState:Set["CARGOFULL"]
            return
        }

        This.CurrentState:Set["MINE"]
    }

    function ProcessState()
    {
        switch ${This.CurrentState}
        {
            case IDLE
                break

            case INSTATION
                call Station.Undock
                break

            case FLEE
                call This.Flee
                break

            case CARGOFULL
                if ${Config.Miner.DeliverToOrca}
                {
                    call This.DeliverToOrca
                }
                else
                {
                    call This.ReturnToStation
                }
                break

            case MINE
                call This.Mine
                break
        }
    }

    function Mine()
    {
        // Get asteroid if needed
        if ${This.CurrentAsteroid} == 0 || !${Entity[${This.CurrentAsteroid}](exists)}
        {
            This.CurrentAsteroid:Set[${Asteroids.GetNext}]
        }

        if ${This.CurrentAsteroid} == 0
        {
            echo "No asteroids available"
            return
        }

        // Lock asteroid
        if !${Entity[${This.CurrentAsteroid}].IsLockedTarget}
        {
            Entity[${This.CurrentAsteroid}]:LockTarget
            return
        }

        // Activate mining lasers
        Ship:Activate_MiningLasers
    }
}
```

**State Flow**:
```
IDLE → INSTATION (undock) → MINE → CARGOFULL → RETURNTOSTATION → INSTATION
                                ↓
                              FLEE (hostiles)
```

### Hauler Behavior Example

File: `Behaviors/obj_Hauler.iss` (we analyzed this earlier)

**States**:
- `IDLE` - Waiting
- `HARDSTOP` - Emergency dock
- `FLEE` - Danger, get to safety
- `BASE` - In station, unload
- `HAUL` - Pick up cargo
- `DROPOFF` - Cargo full, return

**Pattern**: Safety checks → Determine action → Execute

---

## Pulse and Timing System

### PulseTimer Object

File: `core/obj_PulseTimer.iss`

```lavishscript
objectdef obj_PulseTimer
{
    variable float NextPulse
    variable float MinIntervalInSeconds
    variable float MaxIntervalInSeconds

    method SetIntervals(float Min, float Max)
    {
        This.MinIntervalInSeconds:Set[${Min}]
        This.MaxIntervalInSeconds:Set[${Max}]
    }

    member:bool Ready()
    {
        if ${Time.Timestamp} >= ${This.NextPulse}
        {
            return TRUE
        }
        return FALSE
    }

    method Update()
    {
        variable float RandomInterval
        RandomInterval:Set[${Math.Rand[${Math.Calc[${This.MaxIntervalInSeconds} - ${This.MinIntervalInSeconds}]}]}]
        RandomInterval:Set[${Math.Calc[${RandomInterval} + ${This.MinIntervalInSeconds}]}]

        This.NextPulse:Set[${Math.Calc[${Time.Timestamp} + ${RandomInterval}]}]
    }
}
```

**Usage Pattern**:
```lavishscript
variable obj_PulseTimer MyTimer

MyTimer:SetIntervals[2.0, 3.0]    // 2-3 second random interval

method Pulse()
{
    if ${MyTimer.Ready}
    {
        // Do work

        MyTimer:Update    // Schedule next pulse
    }
}
```

**Why Randomization?**: Prevents bot detection by varying timing. All bots pulsing at exact 2.000 second intervals = suspicious.

### Multi-Level Timing

EVEBot uses tiered pulse rates:

```
EVENT_ONFRAME (every frame)
    ↓
EVEBot.Pulse (4-5 seconds)
    ↓
Behavior.Pulse (2-3 seconds)
    ↓
Individual actions (immediate)
```

**Rationale**:
- Frame-level events for critical checks (session changes)
- EVEBot pulse for system-level tasks (downtime check, UI toggle)
- Behavior pulse for bot logic
- Immediate execution for time-sensitive actions

---

## Configuration System

### Configuration Architecture

File: `core/obj_Configuration.iss` (simplified)

```lavishscript
objectdef obj_Configuration
{
    variable obj_Configuration_Common Common
    variable obj_Configuration_Miner Miner
    variable obj_Configuration_Hauler Hauler
    variable obj_Configuration_Combat Combat
    variable obj_Configuration_Fleet Fleet
    // ... more config sections

    method Initialize()
    {
        // Load from XML
        This:Load["${Script.CurrentDirectory}/Config/EVEBot.xml"]
    }

    method Load(string filename)
    {
        // Parse XML and populate config objects
        LavishSettings:Import[${filename}]

        // Each config section loads its own settings
        This.Common:Load
        This.Miner:Load
        This.Hauler:Load
        // ...
    }

    method Save()
    {
        This.Common:Save
        This.Miner:Save
        This.Hauler:Save

        LavishSettings:Export["${Script.CurrentDirectory}/Config/EVEBot.xml"]
    }
}

objectdef obj_Configuration_Common
{
    variable string CurrentBehavior = "Miner"
    variable bool DisableUI = FALSE
    variable bool Disable3D = FALSE
    variable int MaxRuntime = 0

    method Load()
    {
        // Load from LavishSettings
        This.CurrentBehavior:Set["${LavishSettings[EVEBot].FindSet[Common].FindSetting[CurrentBehavior]}"]
        This.DisableUI:Set[${LavishSettings[EVEBot].FindSet[Common].FindSetting[DisableUI,FALSE]}]
        // ...
    }

    method Save()
    {
        LavishSettings[EVEBot].FindSet[Common]:AddSetting[CurrentBehavior, "${This.CurrentBehavior}"]
        LavishSettings[EVEBot].FindSet[Common]:AddSetting[DisableUI, ${This.DisableUI}]
        // ...
    }
}
```

### XML Configuration Format

File: `Config/EVEBot.xml`

```xml
<?xml version='1.0' encoding='UTF-8'?>
<InnerSpaceSettings>
    <Set Name="EVEBot">
        <Set Name="Common">
            <Setting Name="CurrentBehavior">Miner</Setting>
            <Setting Name="DisableUI">FALSE</Setting>
            <Setting Name="Disable3D">FALSE</Setting>
            <Setting Name="MaxRuntime">0</Setting>
        </Set>

        <Set Name="Miner">
            <Setting Name="CargoThreshold">27000</Setting>
            <Setting Name="DeliverToOrca">FALSE</Setting>
            <Setting Name="MiningSystemBookmark">My Mining Belt</Setting>
            <Setting Name="DeliveryLocation">Jita IV - Moon 4</Setting>
            <Setting Name="OrcaMode">FALSE</Setting>
            <Setting Name="MasterMode">FALSE</Setting>
            <Setting Name="GroupMode">FALSE</Setting>
        </Set>

        <Set Name="Hauler">
            <Setting Name="HaulerModeName">Service Fleet Members</Setting>
            <Setting Name="DeliveryLocation">Jita IV - Moon 4</Setting>
            <Setting Name="OrcaPilotName">MyOrca</Setting>
        </Set>
    </Set>
</InnerSpaceSettings>
```

**Pattern**: Hierarchical sets with typed settings. LavishSettings provides XML→object mapping.

---

## Master-Slave Coordination

### Master Query Pattern

File: `core/obj_EVEBot.iss` (lines 91-99)

```lavishscript
// In EVEBot.Pulse():
if ${Config.Miner.GroupMode}
{
    if ${This.MasterName.Length} == 0 && ${This.LastMasterQuery.Ready}
    {
        This.LastMasterQuery:SetIntervals[5.0,5.0]
        This.LastMasterQuery:Update

        // Ask all sessions: "Who is the master?"
        relay all -event EVEBot_Master_Query
    }
}
```

### Master Response

```lavishscript
atom Event_Master_Query()
{
    // If I'm configured as master, announce it
    if ${Config.Miner.MasterMode}
    {
        relay all -event EVEBot_Master_Notify "${Me.Name}"
    }
}

atom Event_Master_Notify(string MasterName)
{
    // Store master name
    This.MasterName:Set["${MasterName}"]
    echo "Master is: ${This.MasterName}"
}
```

### Asteroid Claiming (Master/Slave)

File: `core/obj_Asteroids.iss` (lines 520-570)

```lavishscript
// Before selecting asteroid, claim it
if ${Config.Miner.GroupMode}
{
    // Broadcast claim to all miners
    relay all "Event[EVEBot_ClaimAsteroid]:Execute[${Me.ID}, ${AsteroidIterator.Value.ID}]"
}

// Store claimed asteroid
This.ClaimedAsteroids:Set[${AsteroidIterator.Value.ID}, ${Me.ID}]
```

**Flow**:
1. Master queries → Identifies master
2. Miners claim asteroids → Broadcast to fleet
3. Other miners avoid claimed asteroids
4. When finished, release claim

---

## Key Design Patterns

### Pattern 1: Layered Pulse System

See [Multi-Level Timing](#multi-level-timing) for the full diagram and rationale. In short: tiered pulse rates (frame event → system pulse 4-5s → behavior pulse 2-3s) balance performance, responsiveness, and timing randomization.

### Pattern 2: SetState/ProcessState Separation

```lavishscript
method SetState()
{
    // PURE DECISION LOGIC
    // No actions, just determine state
    if <condition>
        This.CurrentState:Set["STATE"]
}

function ProcessState()
{
    // PURE ACTIONS
    // Execute based on state
    switch ${This.CurrentState}
    {
        case STATE
            call This.DoAction
    }
}
```

**Benefits**:
- Clean separation of concerns
- Easy to debug (know exactly why state was chosen)
- Testable (SetState is deterministic)

### Pattern 3: Emergency Flag Cascade

```lavishscript
// Any module can set emergency flag
EVEBot.ReturnToStation:Set[TRUE]

// All behaviors check flag first
method SetState()
{
    if ${EVEBot.ReturnToStation}
    {
        This.CurrentState:Set["FLEE"]
        return
    }
    // ... normal logic
}
```

**Benefits**:
- Immediate response to danger
- Works across all behaviors
- Simple to implement

### Pattern 4: Relay Event Coordination

```lavishscript
// Broadcaster
relay all -event CustomEvent ${parameter1} ${parameter2}

// Listeners
LavishScript:RegisterEvent[CustomEvent]
Event[CustomEvent]:AttachAtom[This:OnCustomEvent]

atom OnCustomEvent(type1 param1, type2 param2)
{
    // Handle event
}
```

**Benefits**:
- Decoupled communication
- Works across sessions (multi-boxing)
- Extensible (add listeners without modifying broadcaster)

### Pattern 5: Dynamic Loading

```lavishscript
// Scan folder for .iss files
Files:GetFiles["${Path}"]

// Load each file
while (${Position:Inc}<=${Files.Files})
{
    call CreateVariable ${VarName} ${ObjectName} script
}
```

**Benefits**:
- Plugin architecture
- Add features without modifying core
- Clean codebase organization

---

## Strengths and Weaknesses

### Strengths

1. **Modular Architecture**
   - Clean separation between core and behaviors
   - Reusable core objects
   - Easy to add new behaviors

2. **Robust Error Handling**
   - Emergency flag system
   - Hard stop on hostiles/pod
   - Downtime detection

3. **Performance Optimizations**
   - Tiered pulse system
   - UI/3D rendering toggle
   - Entity cache disabled for speed

4. **Multi-Boxing Support**
   - Master/slave coordination
   - Resource claiming (asteroids)
   - Relay-based communication

5. **Comprehensive Coverage**
   - 30+ core objects
   - 11+ behaviors
   - Handles nearly every EVE scenario

6. **Configuration System**
   - XML-based persistence
   - Hierarchical organization
   - Type-safe settings

### Weaknesses

1. **Code Age (2008-2017)**
   - Some ISXEVE APIs deprecated
   - Patterns may be outdated
   - Does not leverage the ISXEVEWrapper .NET API (available for C#-based bots such as Metatron); EVEBot is LavishScript-only

2. **Inconsistent Error Handling**
   - Some functions return bool, some don't
   - Silent failures in some areas
   - No centralized exception handling

3. **Performance Issues**
   - Still uses `wait` commands extensively
   - Could use more caching
   - Entity queries not optimized

4. **Limited Documentation**
   - Code comments sparse
   - No architecture docs (until now!)
   - Learning curve is steep

5. **Tight Coupling in Places**
   - Behaviors directly access global objects
   - Hard to test in isolation
   - Config system tightly bound to behaviors

6. **Threading Model**
   - Threads are independent scripts (heavy)
   - No shared memory (must use relay)
   - Limited coordination between threads

---

## Code Reuse Guide

### What to Reuse

**Highly Recommended**:
1. **Core Object Pattern** (`obj_BaseClass` inheritance)
2. **PulseTimer System** (random interval timing)
3. **SetState/ProcessState Pattern** (state machines)
4. **Dynamic Behavior Loading** (plugin architecture)
5. **Emergency Flag System** (`ReturnToStation`)
6. **Configuration Architecture** (XML + LavishSettings)

**Good Ideas**:
1. **Asteroid Claiming** (resource coordination)
2. **Master/Slave Events** (fleet coordination)
3. **Utility Functions** (MetersToKM_Str, ISK_To_Str)
4. **Downtime Detection** (auto-pause before restart)

### What to Avoid

**Outdated Patterns**:
1. **Heavy Thread Usage** - Use event-driven instead
2. **Excessive `wait` Commands** - Use timers/events
3. **Global Object Access** - Pass dependencies explicitly
4. **Silent Failures** - Always return status codes

**Anti-Patterns**:
1. **No Function Return Values** - Always indicate success/failure
2. **Tight Coupling** - Use interfaces or dependency injection
3. **Hardcoded Values** - Use constants or config
4. **Mixed Concerns** - Keep UI/logic/data separate

### Modernization Tips 🔧

**If Adapting EVEBot Code**:

1. **Add Return Values**:
   ```lavishscript
   // Old (EVEBot style)
   function DoSomething()
   {
       // ... work
   }
   
   // New (better)
   function:bool DoSomething()
   {
       if <error>
           return FALSE
       // ... work
       return TRUE
   }
   ```

2. **Use Event-Driven Over Polling**:
   ```lavishscript
   // Old (EVEBot style)
   while TRUE
   {
       if ${SomeCondition}
           call DoAction
       wait 10
   }
   
   // New (better)
   Event[EVE_OnSomeEvent]:AttachAtom[This:DoAction]
   ```

3. **Cache Entity Queries**:
   ```lavishscript
   // Old (EVEBot style - queries every time)
   if ${Entity[GroupID = 123](exists)}
       // ...
   
   // New (better - cache results)
   if ${This.CachedEntities.Used} == 0
   {
       EVE:QueryEntities[This.CachedEntities, "GroupID = 123"]
   }
   ```

4. **Dependency Injection**:
   ```lavishscript
   // Old (EVEBot style - global access)
   objectdef obj_Behavior
   {
       function DoWork()
       {
           Ship:Activate_Weapon    // Global Ship object
       }
   }
   
   // New (better - inject dependency)
   objectdef obj_Behavior
   {
       variable obj_Ship ShipController
   
       method Initialize(obj_Ship ship)
       {
           This.ShipController:Set[${ship}]
       }
   
       function DoWork()
       {
           This.ShipController:Activate_Weapon
       }
   }
   ```

### Example: Creating New Behavior

Based on EVEBot patterns, here's a template:

```lavishscript
// MyNewBehavior.iss
objectdef obj_MyNewBehavior inherits obj_BaseClass
{
    variable string CurrentState = "IDLE"
    variable time NextPulse
    variable int PulseIntervalInSeconds = 2

    method Initialize()
    {
        LogPrefix:Set["${This.ObjectName}"]

        // Register in behavior system
        Behaviors.Loaded:Add["MyNewBehavior"]

        Event[EVENT_ONFRAME]:AttachAtom[This:Pulse]

        Logger:Log["${LogPrefix}: Initialized", LOG_MINOR]
    }

    method Shutdown()
    {
        Event[EVENT_ONFRAME]:DetachAtom[This:Pulse]
    }

    method Pulse()
    {
        // Check if paused
        if ${EVEBot.Paused}
        {
            return
        }

        // Check if we're the active behavior
        if !${Config.Common.CurrentBehavior.Equal["MyNewBehavior"]}
        {
            return
        }

        // Pulse timing
        if ${Time.Timestamp} >= ${This.NextPulse.Timestamp}
        {
            This:SetState

            This.NextPulse:Set[${Time.Timestamp}]
            This.NextPulse.Second:Inc[${This.PulseIntervalInSeconds}]
            This.NextPulse:Update
        }
    }

    method SetState()
    {
        // SAFETY CHECKS FIRST
        if ${Me.InStation}
        {
            This.CurrentState:Set["INSTATION"]
            return
        }

        if ${EVEBot.ReturnToStation}
        {
            This.CurrentState:Set["FLEE"]
            return
        }

        if ${Social.PossibleHostiles}
        {
            This.CurrentState:Set["FLEE"]
            relay all -event EVEBot_HARDSTOP "${Me.Name} - MyNewBehavior (Hostiles)"
            EVEBot.ReturnToStation:Set[TRUE]
            return
        }

        // NORMAL LOGIC
        This.CurrentState:Set["DOWORK"]
    }

    function ProcessState()
    {
        switch ${This.CurrentState}
        {
            case IDLE
                break

            case INSTATION
                call Station.Undock
                break

            case FLEE
                call This.Flee
                break

            case DOWORK
                call This.DoWork
                break
        }
    }

    function DoWork()
    {
        // Your behavior logic here
        echo "Doing work!"
    }

    function Flee()
    {
        if ${Me.InStation}
            return

        // Find safe station
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
```

**To Add to EVEBot**:
1. Save as `Behaviors/obj_MyNewBehavior.iss`
2. EVEBot will auto-load it
3. Set `Config.Common.CurrentBehavior = "MyNewBehavior"`
4. Run!

---

## Yamfa Fleet Assist (Reference)

For a complete working fleet-assist bot as a study reference, see `Scripts\++isxScripts++\EVE-Online\Scripts\Yamfa\Yamfa.iss` — a single-file multiboxing coordinator built around `objToon` / `objShip` / `objConfig` with a pulse-chain driven by each object's `.Processed` member and relay routing via `Config.Attribute[MasterSession]`. It is deliberately narrower in scope than EVEBot or Tehbot, so the script itself is the best documentation; no architectural deep-dive is provided here.

---

## Tehbot Combat Analysis

---

## Tehbot Overview

### What is Tehbot?

**[Tehbot](https://github.com/isxGames/Tehbot)** = Advanced mission runner and combat bot with unique StateQueue architecture.

**Primary Features**:
- Mission running (auto-accept, auto-complete)
- Abyssal deadspace
- Combat anomalies
- Mining support
- Salvaging
- Advanced target prioritization

**Architecture Philosophy**: "MiniModes" - Small independent modules that coordinate through global variables.

### File Structure

Directory listing from the current Tehbot repository:

```
Tehbot/
├── Tehbot.iss                    # Main entry (orchestrator)
├── Tehbot.xml                    # Main UI
├── TehbotUpdater.cs              # Auto-updater
├── TehbotUpdater.xml
│
├── core/                         # Core objects
│   ├── Defines.iss               # Constants
│   ├── Macros.iss                # Macros
│   ├── obj_Tehbot.iss            # Main controller
│   ├── obj_StateQueue.iss        # State queue pattern
│   ├── obj_Configuration.iss     # Config system
│   ├── obj_Client.iss            # Client/session management
│   ├── obj_Login.iss             # Login / session lifecycle
│   ├── obj_Busy.iss              # Busy-state tracking
│   ├── obj_Logger.iss            # Logging
│   ├── obj_TehbotUI.iss          # UI helpers
│   ├── obj_Module.iss            # Module control
│   ├── obj_ModuleList.iss        # Classified module indexes
│   ├── obj_Ship.iss              # Ship wrapper
│   ├── obj_Cargo.iss             # Cargo handling
│   ├── obj_Drones.iss            # Drone control
│   ├── obj_Move.iss              # Movement
│   ├── obj_TargetList.iss        # Target enumeration/filtering
│   ├── obj_PrioritizedTargets.iss# Target prioritization
│   ├── obj_NPCData.iss           # NPC database
│   ├── obj_FactionData.iss       # Faction database
│   ├── obj_MissionParser.iss     # Mission text parsing
│   ├── obj_Dynamic.iss           # Dynamic minimode registration
│   └── obj_Utility.iss           # Miscellaneous utilities
│
├── behavior/                     # Full behaviors
│   ├── Mission.iss               # Mission running
│   ├── Salvager.iss              # Salvaging behavior
│   └── MiniMode.iss              # Minimode-only runtime host
│
├── minimode/                     # MiniModes (small feature modules)
│   ├── AutoModule.iss            # Auto-activate hardeners / reps / gang links / etc.
│   ├── AutoThrust.iss            # Speed control
│   ├── Automate.iss              # General automation helpers
│   ├── DroneControl.iss          # Drone management
│   ├── FightOrFlight.iss         # Safety / retreat system
│   ├── InstaWarp.iss             # Insta-warp maneuvers
│   ├── Salvage.iss               # Auto-salvage
│   └── UndockWarp.iss            # Undock-and-warp automation
│
├── config/                       # (user config, populated at runtime)
│
└── data/                         # Static data
    ├── NPCData.xml
    ├── FactionData.xml
    ├── DroneData.xml
    └── MissionDataExample.iss
```

**Note on targeting:** Tehbot does NOT ship a dedicated "TargetManager" minimode. Target selection is split across `core/obj_TargetList.iss` (enumeration / filtering) and `core/obj_PrioritizedTargets.iss` (priority logic), and behaviors / minimodes call into those objects directly rather than going through a separate targeting minimode.

**Note on combat optimization:** There is no `obj_CombatComputer.iss` in the real Tehbot. Ammo / damage decisions live inside the behaviors and minimodes themselves using the static `data/NPCData.xml` and `data/FactionData.xml` tables loaded by `obj_NPCData` and `obj_FactionData`.

### Architecture Type

**Tehbot** = **Hybrid Architecture**:
- Core objects (like EVEBot)
- Full behaviors (like EVEBot)
- StateQueue pattern (unique to Tehbot)
- MiniMode modules (unique to Tehbot)
- Global variable coordination (simple but effective)

**Complexity Scale**:
```
Yamfa (Simple)  ←→  Tehbot (Medium)  ←→  EVEBot (Complex)
   1 file            ~36 files            50+ files
   No states         StateQueue           State machines
   Monolithic        MiniModes            Full modularity
```

---

## StateQueue Architecture

### What is StateQueue?

**StateQueue** = Queue-based state machine where states are queued and executed in order.

**Unlike traditional state machines**:
- Traditional: `SetState()` → determines ONE state → `ProcessState()` executes it
- StateQueue: Queue multiple states → Execute in order → Auto-advance

### StateQueue Implementation

File: `core/obj_StateQueue.iss` (lines 1-150)

```lavishscript
objectdef obj_State
{
    variable string Name          // State name
    variable int Frequency        // How often to pulse this state (ms)
    variable string Args          // Arguments to state function

    method Initialize(string arg_Name, int arg_Frequency, string arg_Args)
    {
        Name:Set[${arg_Name}]
        Frequency:Set[${arg_Frequency}]
        Args:Set["${arg_Args.Escape}"]
    }
}

objectdef obj_StateQueue inherits obj_Logger
{
    variable queue:obj_State States     // Queue of states
    variable obj_State CurState         // Current executing state

    variable int NextPulse
    variable int PulseFrequency = 2000  // Default 2 seconds
    variable bool IsIdle = TRUE

    method Initialize()
    {
        CurState:Set["Idle", 100, ""]
        IsIdle:Set[TRUE]
        Event[ISXEVE_onFrame]:AttachAtom[This:Pulse]
    }

    method Pulse()
    {
        if ${LavishScript.RunningTime} >= ${This.NextPulse}
        {
            if (${Tehbot.Paused.Equal["FALSE"]} && ${Client.Ready.Equal["TRUE"]})
            {
                // Execute current state function
                if ${This.${CurState.Name}[${CurState.Args}]}
                {
                    // State returned TRUE = complete, advance to next
                    if ${States.Used} == 0
                    {
                        This:QueueState["Idle", 100]
                        IsIdle:Set[TRUE]
                    }

                    // Dequeue and set next state
                    CurState:Set[${States.Peek.Name}, ${States.Peek.Frequency}, "${States.Peek.Args.Escape}"]
                    States:Dequeue
                }
            }

            // Set next pulse time (current state frequency + random delta)
            This.NextPulse:Set[${Math.Calc[${LavishScript.RunningTime} + ${CurState.Frequency} + ${Math.Rand[500]}]}]
        }
    }

    method QueueState(string arg_Name, int arg_Frequency=-1, string arg_Args="")
    {
        variable int var_Frequency

        if ${arg_Frequency} == -1
        {
            var_Frequency:Set[${This.PulseFrequency}]
        }
        else
        {
            var_Frequency:Set[${arg_Frequency}]
        }

        States:Queue[${arg_Name},${var_Frequency},"${arg_Args.Escape}"]
        This.IsIdle:Set[FALSE]
    }

    method InsertState(string arg_Name, int arg_Frequency=-1, string arg_Args="")
    {
        // Insert at front of queue (priority)
        variable queue:obj_State tempStates

        // Save existing states
        variable iterator StateIterator
        States:GetIterator[StateIterator]
        if ${StateIterator:First(exists)}
        {
            do
            {
                tempStates:Queue[${StateIterator.Value.Name},${StateIterator.Value.Frequency},"${StateIterator.Value.Args.Escape}"]
            }
            while ${StateIterator:Next(exists)}
        }

        States:Clear

        // Resolve frequency (default to PulseFrequency if not supplied)
        variable int var_Frequency
        if ${arg_Frequency} == -1
        {
            var_Frequency:Set[${This.PulseFrequency}]
        }
        else
        {
            var_Frequency:Set[${arg_Frequency}]
        }

        // Insert new state at front
        States:Queue[${arg_Name},${var_Frequency},"${arg_Args.Escape}"]

        // Re-queue existing states
        tempStates:GetIterator[StateIterator]
        if ${StateIterator:First(exists)}
        {
            do
            {
                States:Queue[${StateIterator.Value.Name},${StateIterator.Value.Frequency},"${StateIterator.Value.Args.Escape}"]
            }
            while ${StateIterator:Next(exists)}
        }

        This.IsIdle:Set[FALSE]
    }

    member:bool Idle()
    {
        return TRUE    // Always returns TRUE to advance queue
    }
}
```

### StateQueue Usage Pattern

**Behaviors inherit from obj_StateQueue**:

```lavishscript
objectdef obj_Mission inherits obj_StateQueue
{
    method Initialize()
    {
        This[parent]:Initialize    // Call parent (StateQueue) init
    }

    // Define state functions (must return bool)
    member:bool StartMission(string args)
    {
        echo "Starting mission..."

        // Accept mission from agent
        call Agent.AcceptMission

        // Queue next states
        This:QueueState["WarpToMission"]     // State 2
        This:QueueState["ClearPocket"]       // State 3
        This:QueueState["CompleteMission"]   // State 4

        return TRUE    // State complete, advance to next
    }

    member:bool WarpToMission(string args)
    {
        echo "Warping to mission..."

        call Mission.WarpToMission

        return TRUE
    }

    member:bool ClearPocket(string args)
    {
        echo "Clearing pocket..."

        if ${This.PocketCleared}
        {
            return TRUE    // Pocket clear, advance
        }

        call Combat.Fight

        return FALSE    // Not done yet, stay in this state
    }

    member:bool CompleteMission(string args)
    {
        echo "Completing mission..."

        call Agent.CompleteMission

        return TRUE
    }

    member:bool PocketCleared()
    {
        variable index:entity hostiles
        EVE:QueryEntities[hostiles, "IsNPC && !IsMoribund && Distance < 200000"]

        return ${Math.Calc[${hostiles.Used} == 0]}
    }
}

// Usage:
Mission:QueueState["StartMission"]

// Execution flow:
// StartMission → queues WarpToMission, ClearPocket, CompleteMission
// WarpToMission → executes, returns TRUE, advances
// ClearPocket → executes repeatedly until pocket clear (returns TRUE)
// CompleteMission → executes, returns TRUE, queue empty → Idle
```

### StateQueue Flow Diagram

```
Initial:
  Queue: [Empty]
  Current: Idle

Mission:QueueState["StartMission"]:
  Queue: [StartMission]
  Current: Idle
    ↓
  Pulse → Execute Idle → TRUE → Dequeue
  Queue: []
  Current: StartMission
    ↓
  Pulse → Execute StartMission → queues 3 more states → TRUE → Dequeue
  Queue: [WarpToMission, ClearPocket, CompleteMission]
  Current: WarpToMission
    ↓
  Pulse → Execute WarpToMission → TRUE → Dequeue
  Queue: [ClearPocket, CompleteMission]
  Current: ClearPocket
    ↓
  Pulse → Execute ClearPocket → FALSE (not done) → Stay in state
  Queue: [CompleteMission]
  Current: ClearPocket
    ↓
  Pulse → Execute ClearPocket → FALSE (still fighting)
  ...
    ↓
  Pulse → Execute ClearPocket → TRUE (pocket clear!) → Dequeue
  Queue: []
  Current: CompleteMission
    ↓
  Pulse → Execute CompleteMission → TRUE → Dequeue
  Queue: []
  Current: Idle
```

### Advantages of StateQueue

**Dynamic State Planning**
- Can queue states mid-execution
- Can insert high-priority states at front
- Self-modifying behavior

**Clear Sequencing**
- States execute in order
- Easy to visualize flow
- Predictable behavior

**Reentrant States**
- State returns FALSE → stays in state
- State returns TRUE → advances
- Perfect for "wait until done" patterns

**Frequency Per State**
- Each state has its own pulse frequency
- Combat state: 100ms
- Travel state: 3000ms
- Optimizes performance

### Disadvantages of StateQueue

**Complexity**
- Harder to understand than simple state machine
- Queue manipulation can be tricky
- Debugging is difficult (what's in queue?)

**Global State**
- States share scope
- Hard to isolate state logic
- Can lead to unexpected interactions

**No State Parameters (Properly)**
- Args are strings, require parsing
- Type safety issues
- Error-prone

**When to Use StateQueue**:
- Complex multi-step sequences (missions, abyssals)
- Dynamic state insertion needed
- Reentrant "wait until done" states
- Different pulse frequencies per state

**When NOT to Use StateQueue**:
- Simple bots (overkill)
- Reactive behaviors (better with SetState/ProcessState)
- State machine with few states

---

## MiniMode System

### What are MiniModes?

**MiniModes** = Small, focused modules that run independently and coordinate through shared object references.

**Real minimodes shipped with Tehbot** (from the `minimode/` directory):
- `AutoModule` — auto-activate hardeners / shield boosters / armor reps / gang links / cloaks
- `AutoThrust` — speed and propulsion control
- `Automate` — general-purpose automation helpers
- `DroneControl` — drone deployment and management
- `FightOrFlight` — combat vs. retreat safety logic
- `InstaWarp` — insta-warp maneuvers
- `Salvage` — auto-salvage wrecks
- `UndockWarp` — undock-and-warp automation

**Pattern**: Each minimode is an `obj_StateQueue` subclass that pulses independently at its own `PulseFrequency` and registers through `DynamicAddMiniMode`.

### Minimode Registration and Coordination

Tehbot does not rely on a large pool of named `global` variables for minimode coordination. Minimodes register themselves into a shared registry via `DynamicAddMiniMode(...)` from `core/obj_Dynamic.iss`, and the registry (`obj_Dynamic`) owns collections of known behaviors and minimodes. Each minimode is a StateQueue that pulses on its own `PulseFrequency` and reaches for shared state through well-known object references (`Tehbot`, `Config`, `Drones`, `Ship`, `NPCData`, `FactionData`, `PrioritizedTargets`, `ActiveNPC` / `obj_TargetList` instances) rather than a flat variable namespace.

Where cross-module coordination is needed — "what is the current offense target?", "am I in combat?", "should drones engage?" — the source of truth tends to be a member on one of those shared objects (e.g. a minimode reads `PrioritizedTargets.GetBestTarget` or consults its own `obj_TargetList`), not a top-level `global` primitive.

### Where Targeting Lives

Target selection is not itself a minimode in Tehbot. The core objects `obj_TargetList` (enumeration / filtering) and `obj_PrioritizedTargets` (priority rules) do the work, and whichever behavior or minimode needs a target calls into those objects and writes the chosen ID into a shared global (`CurrentOffenseTarget`). The closest thing to a "targeting minimode" in spirit is the targeting-related block inside behaviors like `behavior/Mission.iss`, not a standalone file.

### MiniMode Example: DroneControl

Sketch of the real `minimode/DroneControl.iss` shape (simplified — consult the real file for full detail):

```lavishscript
objectdef obj_DroneControl inherits obj_StateQueue
{
    variable obj_Configuration_DroneControl Config
    variable obj_TargetList ActiveNPC
    variable int64 currentTarget = 0

    method Initialize()
    {
        This[parent]:Initialize
        PulseFrequency:Set[1000]
        This.NonGameTiedPulse:Set[TRUE]
        DynamicAddMiniMode("DroneControl", "DroneControl")
    }

    method Start()
    {
        This:QueueState["DroneControl"]
    }

    member:bool DroneControl(string args)
    {
        ; Pick best NPC target from own obj_TargetList
        ; (real code picks by class, range, and configured drone loadout)
        variable int64 target = ${This.ActiveNPC.GetBestTarget}

        if ${target} == 0
        {
            call This.RecallDrones
            return FALSE
        }

        ; Launch / redirect drones as needed
        if !${Me.GetActiveDrones.Count}
            call This.LaunchDrones
        call This.SendDronesToTarget ${target}

        return FALSE    ; Stay in state, keep controlling
    }
}
```

Key points:

- Inherits `obj_StateQueue` and registers through `DynamicAddMiniMode` on initialization.
- Holds its own `obj_TargetList` (`ActiveNPC`) rather than reading a global "current target" variable.
- Owns an `obj_Configuration_DroneControl` for runtime-tunable settings (sentry usage, drone-count cap, engage range).
- Pulses independently at its own `PulseFrequency`.

### Trade-offs of the MiniMode Pattern

Advantages:
- **Modularity.** Each minimode is a single file with a single responsibility and can be enabled / disabled independently.
- **Reusability.** The same minimode (e.g. `AutoModule`, `DroneControl`) plugs into multiple behaviors without modification.
- **Simple lifecycle.** StateQueue inheritance + `DynamicAddMiniMode` registration means no bespoke wiring per module.

Costs:
- **Implicit coupling.** A behavior "just knows" that `DroneControl` will pick its own targets from `obj_TargetList` and activate the right drones — changing that contract quietly breaks callers.
- **Pulse-timing assumptions.** Minimodes pulse at their own `PulseFrequency`; code that assumes a specific ordering between two modules can desync when those frequencies differ.
- **Discoverability.** Shared state lives on object references (`Drones`, `NPCData`, per-minimode `obj_TargetList` instances) rather than in one obvious place, so new contributors need to trace several files to understand a coordination path.

A common "modernization" suggestion is to replace the implicit shared-reference style with explicit message passing (a small `obj_MessageBus` with typed post/consume). That buys discoverability at the cost of boilerplate; whether it's worth it depends on project size.

---

## Dynamic Behavior and MiniMode Registry

**Reference implementation:** see `Scripts\Tehbot\core\obj_Dynamic.iss` and `Scripts\Tehbot\core\Macros.iss`.

Tehbot has no hardcoded list of behaviors or minimodes. Every behavior file (e.g. `Mission.iss`, `Salvager.iss`, `MiniMode.iss`) and every minimode file (`AutoModule.iss`, `DroneControl.iss`, `FightOrFlight.iss`, `InstaWarp.iss`, `UndockWarp.iss`, ...) self-registers into a single runtime registry when it's loaded. The registry then drives the UI dropdowns and the dispatch layer.

### The registration macro

Both registration entry points are macros defined in `core/Macros.iss`:

```lavishscript
#macro DynamicAddBehavior(name, displayname)
    Dynamic:AddBehavior[name, displayname, ${String[_FILE_].Escape}]
#endmac

#macro DynamicAddMiniMode(name, displayname)
    Dynamic:AddMiniMode[name, displayname, ${String[_FILE_].Escape}]
#endmac
```

The `${String[_FILE_]}` capture means the registry automatically remembers the source file path of each registered item, so nothing needs to be hand-wired in a central table.

Each behavior / minimode invokes the appropriate macro from its `Initialize` method, typically one line:

```lavishscript
; Inside a behavior's Initialize()
DynamicAddBehavior("Mission", "Combat Missions")

; Inside a minimode's Initialize()
DynamicAddMiniMode("DroneControl", "DroneControl")
```

The first argument is the internal name (which must match the declared object name so the registry can resolve method calls against it); the second is the human-readable label that populates the UI dropdown.

### The registry object

`obj_Dynamic` holds two keyed collections, one for behaviors and one for minimodes, each keyed by internal name and carrying the display name and source path for every registered item. It also owns an `obj_Configuration_Dynamic` that persists which minimodes are currently enabled.

Two methods bridge the registry to the UI: `Dynamic:PopulateBehaviors` and `Dynamic:PopulateMiniModes`. They clear the relevant UI list elements and rewalk the collections, adding each entry's display name / internal name pair. For minimodes, the population step also consults the persisted "enabled minimodes" config set and immediately calls `<MiniModeName>:Start` on the ones that were enabled in the previous run — so enable-state survives restarts with no extra code.

### Runtime activation and deactivation

```lavishscript
method ActivateMiniMode(string name)
{
    This.Config:AddMiniMode[${name.Escape}]
    ${name}:Start
}

method DeactivateMiniMode(string name)
{
    This.Config:RemMiniMode[${name.Escape}]
    ${name}:Stop
}
```

The UI calls these by the user's selection. `${name}:Start` expands into a method call on the live object; because the internal name in `DynamicAddMiniMode(...)` matches the `declarevariable` of the minimode object, the dispatch resolves without any hand-written switch statement.

### Why this beats a hardcoded selector

A naive alternative is a central `if` ladder:

```lavishscript
; ANTI-PATTERN
if ${Config.Mode.Equal["Mission"]}
    call RunMission
elseif ${Config.Mode.Equal["Salvager"]}
    call RunSalvager
elseif ${Config.Mode.Equal["MyCustomBehavior"]}
    call RunMyCustomBehavior
```

Every new behavior requires editing that ladder, the UI dropdown, and the configuration schema. The registry pattern collapses all three into a single `DynamicAddBehavior(...)` line inside the new behavior's own file:

```lavishscript
; Dropping a new file minimode/MyCustomBehavior.iss and adding one #include
; line in Tehbot.iss is all that's needed. The UI dropdown, enable/disable
; persistence, and dispatch all pick it up automatically.
objectdef obj_MyCustomBehavior inherits obj_StateQueue
{
    method Initialize()
    {
        This[parent]:Initialize
        PulseFrequency:Set[1000]
        DynamicAddMiniMode("MyCustomBehavior", "My Custom Behavior")
    }
    ; ... state definitions ...
}
```

The trade-off is a small amount of discoverability: the "list of all behaviors" is no longer in one place — it's whatever the loader happens to `#include`. In practice that's fine because the `#include` list in the main script still gives a single place to read.

---

## Mission Data Files as Per-Mission Scripts

**Reference implementation:** see `Scripts\Tehbot\behavior\Mission.iss` and `Scripts\Tehbot\data\MissionDataExample.iss`.

Tehbot's mission behavior is deliberately generic — it knows how to fly to an agent, accept a mission, warp to the mission site, kill NPCs, loot, deliver, and turn in — but it knows nothing specific about individual missions ("The Blockade", "Dread Pirate Scarlet", ...). Per-mission knowledge lives in a separate `.iss` file under `data/`, loaded at runtime.

### How the file is loaded

The Mission behavior's `Start` method clears its per-mission collections, reads a config attribute naming the data file, and `runscript`s it:

```lavishscript
variable filepath MissionData = "${Script[Tehbot].CurrentDirectory}/data/${Config.MissionFile}"
runscript "${MissionData}"
```

`runscript` executes the file's `function main()` in the context of the running Tehbot script. The main function's only job is to populate collections that the Mission behavior will consult later.

### What a data file contains

Each mission data file is a regular LavishScript `function main()` that performs a series of `:Insert` / `:Set` / `:Add` calls into script-scoped collections on the live Mission object. A trimmed example:

```lavishscript
function main()
{
    ; Agents to cycle through
    Script[Tehbot].VariableScope.Mission.AgentList:Insert["Guy"]
    Script[Tehbot].VariableScope.Mission.AgentList:Insert["Dude"]

    ; Factions whose missions to decline
    Script[Tehbot].VariableScope.Mission.DontFightFaction:Insert["Amarr"]

    ; Per-mission damage type (or "Auto" to detect from faction logo)
    Script[Tehbot].VariableScope.Mission.DamageType:Set["The Blockade", "Auto"]
    Script[Tehbot].VariableScope.Mission.DamageType:Set["Dread Pirate Scarlet", "Kinetic"]

    ; A mission-specific kill-this-exact-entity requirement
    Script[Tehbot].VariableScope.Mission.TargetToDestroy:Set["The Right Hand Of Zazzmatazz", \
        "Name = \"Outpost Headquarters\""]

    ; Mission-specific loot / delivery / gate-key requirements
    Script[Tehbot].VariableScope.Mission.ContainerToLoot:Set["Worlds Collide", \
        "Name = \"Damaged Heron\" && !IsWreckEmpty"]
    Script[Tehbot].VariableScope.Mission.GateKey:Set["Dread Pirate Scarlet", "Gate Key"]

    ; Missions to skip entirely
    Script[Tehbot].VariableScope.Mission.BlackListedMission:Add["Surprise Surprise"]
}
```

Collections populated this way include agent list, damage-type override table, target-to-destroy search strings, container-to-loot search strings, gate-key item names and their source containers, deliver-item container specs, faction blacklist, and mission blacklist.

### How the bot consumes the data

During its pulse, the Mission behavior reads the name of the current mission, then looks it up in the collections by key:

```lavishscript
if ${DamageType.Element[${missionName}](exists)}
    damageType:Set[${DamageType.Element[${missionName}].Lower}]

if ${TargetToDestroy.Element[${missionName}](exists)}
    targetToDestroy:Set[${TargetToDestroy.Element[${missionName}]}]
```

Missions that aren't in the collections simply fall through to generic behavior (or are skipped, depending on policy).

### Why this beats hardcoding

Hardcoding mission knowledge inside the behavior file has three problems: (1) the behavior file grows linearly with the number of missions the bot supports, (2) updating a single mission's damage type requires a code edit and redeploy, and (3) different users end up maintaining divergent forks because their agent lists and faction preferences differ.

The data-file pattern sidesteps all three. The behavior is stable, shipped code; each user supplies their own `MyMissions.iss` under `data/` with their agents, their factional preferences, and any per-mission overrides they've tuned. The config setting `MissionFile` names which data file to load, so a user can keep several profiles (e.g. `lowsec.iss`, `highsec.iss`) and switch between them with a dropdown.

The cost is that the data file is executable LavishScript rather than a passive data format — a malformed data file can crash the bot just like a malformed behavior file. In practice this is acceptable because the data file is authored by the same person running the bot.

---

## Comparison: Tehbot vs EVEBot vs Yamfa

### Architecture Summary

| Aspect | Yamfa | Tehbot | EVEBot |
|--------|-------|--------|--------|
| **Files** | 1 | ~36 | 50+ |
| **Main Pattern** | Simple loop | StateQueue | State machine |
| **Modularity** | None | MiniModes | Core objects + Behaviors |
| **Coordination** | Relay events | Global variables | Function calls |
| **Complexity** | Simple | Medium | Complex |
| **Learning Curve** | Easy | Medium | Steep |
| **Use Case** | Fleet assist | Combat/missions | Everything |
| **Unique Feature** | Hysteresis | StateQueue | Full modularity |

### Code Comparison

**Yamfa (Simple Loop)**:
```lavishscript
function main()
{
    while TRUE
    {
        if ${IsMaster}
        {
            call MasterPulse    ; Get targets, relay
        }
        else
        {
            call SlavePulse     ; Lock targets, follow
        }

        wait ${Math.Rand[2,6]}
    }
}
```

**Tehbot (StateQueue)**:
```lavishscript
objectdef obj_Mission inherits obj_StateQueue
{
    member:bool StartMission()
    {
        This:QueueState["WarpTo"]
        This:QueueState["ClearPocket"]
        This:QueueState["Complete"]
        return TRUE
    }
}

; States execute in order, auto-advance
```

**EVEBot (State Machine)**:
```lavishscript
objectdef obj_Miner
{
    method SetState()
    {
        if <condition>
            This.CurrentState:Set["STATE"]
    }

    function ProcessState()
    {
        switch ${This.CurrentState}
        {
            case STATE
                call DoAction
        }
    }
}
```

### Performance Comparison

No benchmark numbers are published in this guide. Startup time, memory usage, and CPU usage vary substantially by ship fit, configuration, module count, number of tracked entities, and which behaviors / minimodes are active — so any single number would be misleading.

Qualitatively, a monolithic single-file script (Yamfa-style) starts and runs with the least overhead; a hybrid framework (Tehbot) sits in the middle because it loads a moderate number of core objects and minimodes at startup; and a fully modular framework (EVEBot) has the highest startup cost and resident footprint because it loads core objects, behavior objects, threads, and library modules. Maintainability trades off inversely: more files generally means more isolation and easier targeted changes, at the cost of a steeper initial read.

If you need concrete numbers for your use case, measure on your own machine with your own configuration — `Script.RunningTime`, `${Script[name].MemoryUsage}`-style probes, and Windows Task Manager will all give you representative readings.

### Scalability

**Yamfa**: Does NOT scale
- Adding features = rewriting core
- No plugin system
- Monolithic design

**Tehbot**: Scales moderately
- Add features via MiniModes
- Some plugin capability
- Global variables limit scalability

**EVEBot**: Scales excellently
- Add features via new behaviors/objects
- Full plugin architecture
- Clean interfaces

---

## Key Patterns

### Pattern 1: StateQueue (Tehbot)

**When to use**:
- Complex multi-step sequences
- Dynamic state insertion
- Reentrant states

**Example**:
```lavishscript
This:QueueState["Step1"]    ; Add states
This:QueueState["Step2"]
This:QueueState["Step3"]
This:InsertState["Urgent"]  ; Insert at front

; Execution: Urgent → Step1 → Step2 → Step3
```

### Pattern 2: MiniMode Coordination (Tehbot)

**When to use**:
- Independent modules need to coordinate
- Simple message passing
- Don't want complex architecture

**Example**:
```lavishscript
; Module A sets global
CurrentTarget:Set[${targetID}]
TargetReady:Set[TRUE]

; Module B reads global
if ${TargetReady}
{
    call ShootTarget ${CurrentTarget}
}
```

### Pattern 3: Hysteresis and Change Detection (Yamfa)

For the full code and rationale, see [Code Strengths](#code-strengths) in the Yamfa analysis above (patterns: Hysteresis for Stability, Change Detection Before Relay, Command Caching, Random Timing).

**When to use these Yamfa patterns**:
- Hysteresis: prevent state flickering, smooth transitions, anti-spam
- Change detection: reduce network/relay traffic by only broadcasting on actual change
- Command caching: avoid re-issuing identical movement/action commands
- Random timing: anti-detection, natural pacing

### Pattern 4: Behavior + Core (EVEBot)

**When to use**:
- Large project
- Team development
- Long-term maintenance

**Example**:
```lavishscript
; Core objects = services
obj_Ship, obj_Cargo, obj_Combat

; Behaviors = use services
objectdef obj_Miner
{
    function Mine()
    {
        call Ship.Activate_MiningLasers
        call Cargo.CheckFull
    }
}
```

---

## When to Use Which Architecture

### Use Yamfa-Style (Single File) When:

- Bot has ONE specific job
- Code < 1000 lines
- Solo developer
- Learning/prototyping
- Need quick iteration

**Example Use Cases**:
- Fleet assist (targeting sync)
- Auto-follow bot
- Bookmark organizer
- Market monitor

### Use Tehbot-Style (StateQueue + MiniModes) When:

- Complex sequences (missions, abyssals)
- Multiple independent features
- Need dynamic state management
- Code 1000-5000 lines
- Medium complexity

**Example Use Cases**:
- Mission runner
- Abyssal runner
- Anomaly bot with safety features
- Mining bot with combat defense

### Use EVEBot-Style (Full Modularity) When:

- Multi-purpose bot
- Code > 5000 lines
- Team development
- Long-term maintenance
- Maximum flexibility

**Example Use Cases**:
- Full automation suite
- Multi-behavior bot (mining, combat, hauling)
- Framework for derivative bots
- Commercial bot

---

## Community Takeaways

### Key Lessons from Tehbot

1. **StateQueue is Powerful for Sequences**
   - Perfect for missions (step 1 → step 2 → step 3)
   - Self-modifying behavior (queue states mid-execution)
   - Reentrant states (return FALSE to stay in state)

2. **MiniModes = Simple Modularity**
   - Small, focused modules
   - Coordination via shared object references (not a flat global-variable namespace)
   - Easy to add features

3. **Static Data Tables for Damage / Target Decisions**
   - `data/NPCData.xml` and `data/FactionData.xml` loaded at startup by `obj_NPCData` / `obj_FactionData`
   - Avoids per-encounter database queries
   - Good fit when the bot's enemy set is known up front

### Patterns to Adopt

**StateQueue for Sequences**
```lavishscript
This:QueueState["Step1"]
This:QueueState["Step2"]
; Auto-executes in order
```

**MiniMode Pattern**
```lavishscript
; Small independent modules, each inheriting obj_StateQueue
obj_DroneControl
obj_AutoModule
obj_FightOrFlight
; Coordinate via shared object references and per-module obj_TargetList
```

**Data-Table-Driven Decisions**
```lavishscript
; Load static lookup tables at startup instead of querying per-decision
NPCData:LoadData    ; from data/NPCData.xml
; Then consult in-memory
NPCData.NPCType[${TargetGroupID}]
```

### Patterns to Avoid

**Ad-Hoc Global Variables for Coordination**
```lavishscript
; Flat global namespace couples modules implicitly
; Prefer a shared object reference or a typed message bus
```

**StateQueue for Simple Bots**
```lavishscript
; Overkill for simple targeting bot
; Use simple loop instead
```

---

## Summary

### Tehbot Architecture

**Core Innovation**: StateQueue + MiniModes

**StateQueue**:
- Queue-based state machine
- States execute in order
- Reentrant states (return FALSE to stay)
- Dynamic state insertion

**MiniModes**:
- Small independent modules, each an `obj_StateQueue` subclass
- Register via `DynamicAddMiniMode` into `obj_Dynamic`
- Coordinate through shared object references, not a flat global namespace

**Static Data Layer**:
- `data/NPCData.xml`, `data/FactionData.xml`, `data/DroneData.xml` loaded once
- Minimodes and behaviors consult the in-memory tables for NPC classification, faction preferences, and drone loadouts

### Comparison to Others

**vs. Yamfa**: More complex but more capable
**vs. EVEBot**: Less complex but less scalable

### Best Use Cases

**Tehbot-style perfect for**:
- Mission running (sequences via StateQueue)
- Multi-step combat encounters
- Medium complexity bots (1000-5000 lines)

### For the Community

**Learn from Tehbot**:
1. StateQueue for sequences
2. MiniMode pattern for modularity
3. Static data tables over per-decision database queries

**Avoid**:
1. Flat global-variable coordination
2. StateQueue for genuinely simple bots


Three bot architectures analyzed:
- **EVEBot**: Full modular framework
- **Yamfa**: Simple single-file
- **Tehbot**: Hybrid StateQueue + MiniModes

The community now has three proven patterns to choose from based on project needs!
