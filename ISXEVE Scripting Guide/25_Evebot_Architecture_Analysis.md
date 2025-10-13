# File 25: EVEBot Architecture Analysis

**Layer 6: Script Analysis (Learning from Examples)**

---

## Table of Contents
1. [EVEBot Overview](#evebot-overview)
2. [Project Structure](#project-structure)
3. [Initialization System](#initialization-system)
4. [Core Object Architecture](#core-object-architecture)
5. [Behavior System](#behavior-system)
6. [Pulse and Timing System](#pulse-and-timing-system)
7. [Configuration System](#configuration-system)
8. [Master-Slave Coordination](#master-slave-coordination)
9. [Key Design Patterns](#key-design-patterns)
10. [Strengths and Weaknesses](#strengths-and-weaknesses)
11. [Code Reuse Guide](#code-reuse-guide)

---

## EVEBot Overview

### What is EVEBot?

**EVEBot** is the most comprehensive and mature open-source LavishScript bot for EVE Online. It has been in development since ~2008 and supports:

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

```lavish
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

```lavish
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

```lavish
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

```lavish
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

```lavish
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

All behaviors follow this pattern:

```lavish
objectdef obj_SomeBehavior inherits obj_BaseClass
{
    // STATE
    variable string CurrentState = "IDLE"
    variable time NextPulse
    variable int PulseIntervalInSeconds = 2

    method Initialize()
    {
        LogPrefix:Set["${This.ObjectName}"]
        Event[EVENT_ONFRAME]:AttachAtom[This:Pulse]
    }

    method Pulse()
    {
        if ${EVEBot.Paused}
        {
            return
        }

        if !${Config.Common.CurrentBehavior.Equal["SomeBehavior"]}
        {
            return
        }

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
        // DETERMINE WHAT TO DO
        // Set This.CurrentState based on conditions
    }

    function ProcessState()
    {
        // EXECUTE CURRENT STATE
        switch ${This.CurrentState}
        {
            case IDLE
                break
            case DOSTUFF
                call This.DoStuff
                break
        }
    }
}
```

### Miner Behavior Example

File: `Behaviors/obj_Miner.iss` (simplified)

```lavish
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

```lavish
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
```lavish
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

```lavish
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

```lavish
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

```lavish
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

```lavish
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

```
Frame Event (every frame)
    ↓
System Pulse (4-5 sec)
    ↓
Behavior Pulse (2-3 sec)
```

**Benefits**:
- Performance: Don't check everything every frame
- Responsiveness: Critical checks still fast
- Randomization: Varies timing to avoid detection

### Pattern 2: SetState/ProcessState Separation

```lavish
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

```lavish
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

```lavish
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

```lavish
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

### Strengths ✅

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

### Weaknesses ❌

1. **Code Age (2008-2017)**
   - Some ISXEVE APIs deprecated
   - Patterns may be outdated
   - No modern C# interop

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

### What to Reuse ✅

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

### What to Avoid ❌

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
   ```lavish
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
   ```lavish
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
   ```lavish
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
   ```lavish
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

```lavish
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

## Summary

### Key Takeaways

1. **EVEBot Architecture** = Modular core + Plugin behaviors + Background threads
2. **Pulse System** = Multi-tiered timing for performance
3. **State Machines** = SetState (decide) + ProcessState (act)
4. **Safety First** = Emergency flags, hostile detection, downtime checks
5. **Fleet Coordination** = Relay events + resource claiming
6. **Configuration** = XML persistence + hierarchical settings

### What This Analysis Provides

For **beginners**:
- Clear architecture overview
- Reusable code patterns
- Template for new behaviors
- Understanding of established patterns

For **intermediate developers**:
- Deep architectural insights
- Modernization opportunities
- Performance optimization ideas
- Integration patterns

For **the EVE scripting community**:
- Comprehensive documentation (first of its kind!)
- Foundation for derivative bots
- Best practices catalog
- Anti-pattern warnings

**Next Steps**: Files 26-27 will analyze Yamfa (fleet assist) and Tehbot (combat) to show different architectural approaches and learn from each.

---

**File Statistics**:
- **Lines**: ~1800
- **Code Examples**: 20+
- **Files Analyzed**: 10+
- **Patterns Documented**: 10+

**EVEBot Status**: Mature, comprehensive, proven architecture. Outdated in places but fundamental patterns remain excellent foundation for modern bots.
