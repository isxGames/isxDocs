# Advanced ISXEQ2 Scripting Patterns

**Comprehensive guide to production-grade scripting techniques from [EQ2OgreFree](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2OgreFree) and professional scripts**

This guide covers advanced patterns and techniques discovered through analysis of the [EQ2OgreFree](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2OgreFree) script suite (including [EQ2OgreCommon](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2OgreCommon)). These are production-tested patterns used in complex, multi-threaded automation systems.

---

## Table of Contents

1. [Multi-Threaded Script Architecture](#multi-threaded-script-architecture)
2. [LavishSettings Configuration Management](#lavishsettings-configuration-management)
3. [LavishNav Navigation System](#lavishnav-navigation-system)
4. [Reusable Timer Objects](#reusable-timer-objects)
5. [UI Synchronization Patterns](#ui-synchronization-patterns)
6. [Modern EQ2:GetActors Usage](#modern-eq2getactors-usage)
7. [Trigger System for Chat Parsing](#trigger-system-for-chat-parsing)
8. [Script Controller Pattern](#script-controller-pattern)
9. [Dynamic Variable Declaration](#dynamic-variable-declaration)
10. [Injectable UI Components](#injectable-ui-components)

---

## Multi-Threaded Script Architecture

### Overview

Complex automation often requires multiple scripts running simultaneously, each handling a specific task. The multi-threaded pattern allows scripts to communicate via global variables and cross-script atom execution.

### Key Concepts

- **Main Thread**: Coordinates overall logic
- **Worker Threads**: Handle specific tasks (movement, checking, pathing)
- **Global Variables**: Shared state across threads
- **Cross-Script Atoms**: Methods callable from other scripts

### Pattern Implementation

```lavishscript
;===========================================
; Main Script (HarvestMain.iss)
;===========================================
variable(global) bool HarvestStop=FALSE
variable(global) bool HarvestPause=FALSE
variable(global) string NextLocation
variable(global) int CurrentResourceID

function main()
{
    ; Launch worker threads
    if !${Script[HarvestMoveThread](exists)}
        runscript "${LavishScript.HomeDirectory}/Scripts/HarvestMoveThread"

    if !${Script[HarvestCheckThread](exists)}
        runscript "${LavishScript.HomeDirectory}/Scripts/HarvestCheckThread"

    ; Main loop
    while !${HarvestStop}
    {
        while ${HarvestPause}
            wait 10

        ; Main logic here
        call ProcessHarvesting

        wait 10
    }

    ; Signal threads to stop
    HarvestStop:Set[TRUE]
    wait 50  ; Give threads time to cleanup
}

;===========================================
; Worker Thread (HarvestMoveThread.iss)
;===========================================
function main()
{
    while !${HarvestStop}
    {
        if ${NextLocation.NotEqual[none]}
        {
            call MoveToLocation ${NextLocation}
            NextLocation:Set[none]
        }
        wait 10
    }

    echo HarvestMoveThread ending...
}

atom BreakCurrentMovement()
{
    ; Stop all movement immediately
    EQ2Execute /stop
}

;===========================================
; Cross-Script Communication
;===========================================
; From main script, call atom in worker thread:
Script[HarvestMoveThread]:ExecuteAtom[BreakCurrentMovement]

; Set global variable for worker thread:
NextLocation:Set["50,10,75"]

; Check if thread is still running:
if ${Script[HarvestMoveThread](exists)}
    echo "Thread is running"
```

### Best Practices

1. **Always use global variables for inter-thread communication**
2. **Provide stop signals** - all threads should check a global stop flag
3. **Use atoms for immediate actions** - ExecuteAtom runs in target script's context
4. **Wait for thread startup** - use `wait ${Script[ThreadName](exists)}`
5. **Clean shutdown** - signal all threads to stop, then wait briefly

### Complete Example: Harvesting Bot

```lavishscript
;===========================================
; EQ2HarvestBot.iss - Main Controller
;===========================================
variable(global) bool HarvestStop=FALSE
variable(global) bool HarvestPause=FALSE
variable(global) float TargetX
variable(global) float TargetY
variable(global) float TargetZ
variable(global) string MovementCommand=NONE
variable(global) int ResourceToHarvest=0

function main()
{
    ; Start worker threads
    if !${Script[HarvestScanner](exists)}
        runscript "${LavishScript.HomeDirectory}/Scripts/HarvestScanner"

    if !${Script[HarvestMover](exists)}
        runscript "${LavishScript.HomeDirectory}/Scripts/HarvestMover"

    wait ${Script[HarvestScanner](exists)} && ${Script[HarvestMover](exists)}

    echo "Harvest bot started - all threads running"

    while !${HarvestStop}
    {
        ; Paused check
        while ${HarvestPause} && !${HarvestStop}
            wait 10

        ; If scanner found a resource, harvest it
        if ${ResourceToHarvest} > 0 && ${Actor[${ResourceToHarvest}](exists)}
        {
            ; Stop movement
            Script[HarvestMover]:ExecuteAtom[StopMovement]
            wait 10 !${Me.IsMoving}

            ; Harvest
            Actor[${ResourceToHarvest}]:DoTarget
            wait 10 ${Target.ID}==${ResourceToHarvest}
            Actor[${ResourceToHarvest}]:DoubleClick
            wait 10 ${Me.CastingSpell}
            wait 50 !${Me.CastingSpell}

            ; Clear
            ResourceToHarvest:Set[0]
        }

        wait 10
    }

    ; Shutdown
    HarvestStop:Set[TRUE]
    wait 100
    echo "Harvest bot stopped"
}

;===========================================
; HarvestScanner.iss - Resource Detection Thread
;===========================================
variable index:actor Resources
variable iterator ResourceIt

function main()
{
    while !${HarvestStop}
    {
        if !${HarvestPause} && ${ResourceToHarvest}==0
        {
            ; Scan for resources
            EQ2:GetActors[Resources,Range,50,resource]
            Resources:GetIterator[ResourceIt]

            if ${ResourceIt:First(exists)}
            {
                do
                {
                    ; Found a harvestable resource
                    if ${ResourceIt.Value.Type.Equal[resource]} && ${ResourceIt.Value.Distance} < 50
                    {
                        ResourceToHarvest:Set[${ResourceIt.Value.ID}]
                        TargetX:Set[${ResourceIt.Value.X}]
                        TargetY:Set[${ResourceIt.Value.Y}]
                        TargetZ:Set[${ResourceIt.Value.Z}]
                        MovementCommand:Set[MOVE_TO_RESOURCE]
                        break
                    }
                }
                while ${ResourceIt:Next(exists)}
            }
        }

        wait 500  ; Scan every half second
    }
}

;===========================================
; HarvestMover.iss - Movement Thread
;===========================================
function main()
{
    while !${HarvestStop}
    {
        if ${MovementCommand.Equal[MOVE_TO_RESOURCE]} && ${TargetX} > 0
        {
            call NavigateToPoint ${TargetX} ${TargetY} ${TargetZ}
            MovementCommand:Set[NONE]
        }

        wait 10
    }
}

atom StopMovement()
{
    EQ2Execute /stop
    MovementCommand:Set[NONE]
}

function NavigateToPoint(float X, float Y, float Z)
{
    ; Navigation logic here
    while ${Math.Distance[${Me.X},${Me.Y},${Me.Z},${X},${Y},${Z}]} > 3
    {
        if ${MovementCommand.NotEqual[MOVE_TO_RESOURCE]}
            return

        press -hold ${Forward}
        face ${X} ${Z}
        wait 10
    }
    press -release ${Forward}
}
```

---

## LavishSettings Configuration Management

### Overview

LavishSettings provides a hierarchical XML-based configuration system far more powerful than simple file I/O. It supports nested settings, character-specific configs, and automatic persistence.

### Key Datatypes

- **settingsetref**: Reference to a settings set (like a pointer to a config section)
- Methods: `AddSet`, `FindSet`, `FindSetting`, `FindAttribute`, `Import`, `Export`

### Basic Pattern

```lavishscript
variable settingsetref MyAppSettings
variable settingsetref UserSettings

; Create hierarchical structure
LavishSettings:AddSet[MyApplication]
LavishSettings[MyApplication]:AddSet[Users]
LavishSettings[MyApplication].FindSet[Users]:AddSet[${Me.Name}]

; Get reference to this character's settings
UserSettings:Set[${LavishSettings[MyApplication].FindSet[Users].FindSet[${Me.Name}]}]

; Import from file
UserSettings:Import["${Script.CurrentDirectory}/Config/${Me.Name}-${EQ2.ServerName}.xml"]

; Read settings
if ${UserSettings.FindSetting[EnableAutoLoot,TRUE]}
    echo "Auto-loot is enabled"

variable int LootRadius
LootRadius:Set[${UserSettings.FindSetting[LootRadius,50]}]

; Write settings
UserSettings.FindSetting[EnableAutoLoot]:Set[TRUE]
UserSettings.FindSetting[LootRadius]:Set[75]

; Export to file
UserSettings:Export["${Script.CurrentDirectory}/Config/${Me.Name}-${EQ2.ServerName}.xml"]
```

### Complete Example: Bot Configuration System

```lavishscript
;===========================================
; Configuration Manager
;===========================================
variable settingsetref BotConfig
variable settingsetref CombatSettings
variable settingsetref LootSettings
variable settingsetref UISettings

function LoadConfiguration()
{
    ; Build hierarchy: BotApp -> Characters -> [CharName] -> Combat/Loot/UI
    LavishSettings:AddSet[BotApplication]
    LavishSettings[BotApplication]:AddSet[Characters]
    LavishSettings[BotApplication].FindSet[Characters]:AddSet[${Me.Name}]

    BotConfig:Set[${LavishSettings[BotApplication].FindSet[Characters].FindSet[${Me.Name}]}]

    ; Add subsections
    BotConfig:AddSet[Combat]
    BotConfig:AddSet[Loot]
    BotConfig:AddSet[UI]

    ; Get references to subsections
    CombatSettings:Set[${BotConfig.FindSet[Combat]}]
    LootSettings:Set[${BotConfig.FindSet[Loot]}]
    UISettings:Set[${BotConfig.FindSet[UI]}]

    ; Import from XML file
    variable string ConfigFile
    ConfigFile:Set["${Script.CurrentDirectory}/Configs/${Me.Name}-${EQ2.ServerName}.xml"]

    if ${System.FileExists["${ConfigFile}"]}
    {
        BotConfig:Import["${ConfigFile}"]
        echo "Loaded configuration from ${ConfigFile}"
    }
    else
    {
        echo "No config file found, using defaults"
        call SetDefaultSettings
    }

    ; Apply settings to script
    call ApplySettings
}

function SetDefaultSettings()
{
    ; Combat defaults
    CombatSettings.FindSetting[AssistTank]:Set[TRUE]
    CombatSettings.FindSetting[AssistAt]:Set[95]
    CombatSettings.FindSetting[UseAOE]:Set[FALSE]
    CombatSettings.FindSetting[PullDistance]:Set[30]

    ; Loot defaults
    LootSettings.FindSetting[AutoLoot]:Set[TRUE]
    LootSettings.FindSetting[LootRadius]:Set[50]
    LootSettings.FindSetting[LootTreasure]:Set[TRUE]
    LootSettings.FindSetting[LootRares]:Set[TRUE]

    ; UI defaults
    UISettings.FindSetting[ShowMainWindow]:Set[TRUE]
    UISettings.FindSetting[WindowX]:Set[100]
    UISettings.FindSetting[WindowY]:Set[100]
}

function ApplySettings()
{
    ; Read and apply combat settings
    variable bool AssistTank
    variable int AssistAt

    AssistTank:Set[${CombatSettings.FindSetting[AssistTank,TRUE]}]
    AssistAt:Set[${CombatSettings.FindSetting[AssistAt,95]}]

    echo "Combat Settings:"
    echo "  Assist Tank: ${AssistTank}"
    echo "  Assist At: ${AssistAt}%"

    ; Apply to script variables
    script:AssistTankEnabled:Set[${AssistTank}]
    script:AssistPercentage:Set[${AssistAt}]

    ; Read loot settings
    echo "Loot Settings:"
    echo "  Auto Loot: ${LootSettings.FindSetting[AutoLoot,TRUE]}"
    echo "  Loot Radius: ${LootSettings.FindSetting[LootRadius,50]}"
}

function SaveConfiguration()
{
    ; Save current script state to settings
    CombatSettings.FindSetting[AssistTank]:Set[${script:AssistTankEnabled}]
    CombatSettings.FindSetting[AssistAt]:Set[${script:AssistPercentage}]

    ; Export to XML
    variable string ConfigFile
    ConfigFile:Set["${Script.CurrentDirectory}/Configs/${Me.Name}-${EQ2.ServerName}.xml"]
    BotConfig:Export["${ConfigFile}"]

    echo "Configuration saved to ${ConfigFile}"
}

;===========================================
; UI-Driven Settings
;===========================================
function LoadSettingsFromUI()
{
    ; Sync UI checkboxes with settings
    if ${CombatSettings.FindSetting[AssistTank,TRUE]}
        UIElement[ChkAssistTank]:SetChecked

    if ${LootSettings.FindSetting[AutoLoot,TRUE]}
        UIElement[ChkAutoLoot]:SetChecked

    ; Sync text boxes
    UIElement[TxtAssistAt]:SetText[${CombatSettings.FindSetting[AssistAt,95]}]
    UIElement[TxtLootRadius]:SetText[${LootSettings.FindSetting[LootRadius,50]}]
}

function SaveSettingsFromUI()
{
    ; Read from UI and save to settings
    CombatSettings.FindSetting[AssistTank]:Set[${UIElement[ChkAssistTank].Checked}]
    CombatSettings.FindSetting[AssistAt]:Set[${UIElement[TxtAssistAt].Text}]

    LootSettings.FindSetting[AutoLoot]:Set[${UIElement[ChkAutoLoot].Checked}]
    LootSettings.FindSetting[LootRadius]:Set[${UIElement[TxtLootRadius].Text}]

    call SaveConfiguration
}

;===========================================
; XML File Structure Example
;===========================================
/**
<?xml version='1.0' encoding='UTF-8'?>
<InnerSpaceSettings>
    <Set Name="Combat">
        <Setting Name="AssistTank">TRUE</Setting>
        <Setting Name="AssistAt">95</Setting>
        <Setting Name="UseAOE">FALSE</Setting>
        <Setting Name="PullDistance">30</Setting>
    </Set>
    <Set Name="Loot">
        <Setting Name="AutoLoot">TRUE</Setting>
        <Setting Name="LootRadius">50</Setting>
        <Setting Name="LootTreasure">TRUE</Setting>
        <Setting Name="LootRares">TRUE</Setting>
    </Set>
    <Set Name="UI">
        <Setting Name="ShowMainWindow">TRUE</Setting>
        <Setting Name="WindowX">100</Setting>
        <Setting Name="WindowY">100</Setting>
    </Set>
</InnerSpaceSettings>
**/
```

### Benefits

1. **Hierarchical organization** - Group related settings
2. **Character-specific** - Automatic per-character configs
3. **Default values** - `FindSetting[Name,DefaultValue]` pattern
4. **Automatic persistence** - Import/Export handles XML
5. **Type safety** - Settings maintain types

---

## LavishNav Navigation System

### Overview

LavishNav is InnerSpace's powerful pathfinding system that uses pre-mapped zone data for intelligent navigation. It handles obstacles, elevation changes, and complex pathing automatically.

### Key Datatypes

- **dijkstrapathfinder**: Path calculation engine
- **lnavregionref**: Reference to a mapped region
- **lnavpath**: Calculated path with waypoints
- **point3f**: 3D point (X, Y, Z)

### Basic Navigation

```lavishscript
variable dijkstrapathfinder PathFinder
variable lnavregionref CurrentRegion
variable lnavregionref DestinationRegion
variable lnavpath Path
variable float DestX
variable float DestY
variable float DestZ

function NavigateToLocation(float X, float Y, float Z)
{
    DestX:Set[${X}]
    DestY:Set[${Y}]
    DestZ:Set[${Z}]

    ; Calculate path
    PathFinder:GeneratePath[${Me.X},${Me.Y},${Me.Z},${DestX},${DestY},${DestZ},Path]

    if ${Path.Hops}
    {
        echo "Path found with ${Path.Hops} waypoints"
        call FollowPath
    }
    else
    {
        echo "No path available - attempting direct movement"
        call MoveDirectly ${DestX} ${DestY} ${DestZ}
    }
}

function FollowPath()
{
    variable int CurrentHop = 1

    while ${CurrentHop} <= ${Path.Hops}
    {
        ; Get current waypoint
        variable point3f Waypoint
        Path.Hop[${CurrentHop}]:GetPosition[Waypoint]

        ; Move to waypoint
        while ${Math.Distance[${Me.X},${Me.Y},${Me.Z},${Waypoint.X},${Waypoint.Y},${Waypoint.Z}]} > 1.5
        {
            press -hold ${Forward}
            face ${Waypoint.X} ${Waypoint.Z}
            wait 10
        }

        press -release ${Forward}
        CurrentHop:Inc
    }

    echo "Navigation complete"
}
```

### Region-Based Navigation

```lavishscript
variable lnavregionref ZoneRegion

function LoadZoneMap()
{
    ; Load navigation map for current zone
    LNavRegion[${Zone.Name}]:Load
    ZoneRegion:Set[${LNavRegion[${Zone.Name}]}]

    if ${ZoneRegion(exists)}
        echo "Navigation map loaded for ${Zone.Name}"
    else
        echo "WARNING: No navigation map available for ${Zone.Name}"
}

function MoveToRegion(string RegionName)
{
    if !${ZoneRegion.ChildRegion[${RegionName}](exists)}
    {
        echo "Region '${RegionName}' not found in map"
        return
    }

    ; Get destination region
    variable lnavregionref DestRegion
    DestRegion:Set[${ZoneRegion.ChildRegion[${RegionName}]}]

    ; Get center point of region
    variable point3f CenterPoint
    DestRegion:GetCenterPoint[CenterPoint]

    ; Navigate there
    call NavigateToLocation ${CenterPoint.X} ${CenterPoint.Y} ${CenterPoint.Z}
}
```

### Collision Detection

```lavishscript
function CanMoveDirect(float X, float Y, float Z)
{
    ; Check if we can move directly without hitting obstacles
    if !${EQ2.CheckCollision[${Me.X},${Math.Calc[${Me.Y}+1]},${Me.Z},${X},${Math.Calc[${Y}+1]},${Z}]}
    {
        echo "No collision detected - can move directly"
        return TRUE
    }
    else
    {
        echo "Collision detected - need to path"
        return FALSE
    }
}

function SmartNavigate(float X, float Y, float Z)
{
    if ${CanMoveDirect[${X},${Y},${Z}]}
    {
        ; Direct movement
        call MoveDirectly ${X} ${Y} ${Z}
    }
    else
    {
        ; Use pathfinding
        call NavigateToLocation ${X} ${Y} ${Z}
    }
}
```

### Complete Navigation System

```lavishscript
;===========================================
; OgreNav_Lib.inc - Complete Navigation Library (from EQ2OgreFree, not modern-day Ogre)
;===========================================
variable dijkstrapathfinder PathFinder
variable lnavregionref ZoneRegion
variable lnavregionref DestinationRegion
variable lnavpath Path
variable float DestPointX
variable float DestPointY
variable float DestPointZ
variable point3f LastLoc
variable int StuckCounter
variable float Precision = 1.5

function OgreNav(string Location, float X, float Y, float Z)
{
    if ${Location.Equal[Loc]}
    {
        ; Navigate to coordinates
        call MoveToLoc ${X} ${Y} ${Z}
    }
    else
    {
        ; Navigate to named region
        if !${ZoneRegion.ChildRegion[${Location}](exists)}
        {
            echo "Destination '${Location}' not found"
            return
        }

        call MoveToRegion "${Location}"
    }
}

function MoveToLoc(float X, float Y, float Z)
{
    DestPointX:Set[${X}]
    DestPointY:Set[${Y}]
    DestPointZ:Set[${Z}]

    ; Already there?
    if ${Math.Distance[${Me.X},${Me.Y},${Me.Z},${DestPointX},${DestPointY},${DestPointZ}]} < ${Precision}
    {
        echo "Already at destination"
        return
    }

    ; Calculate path
    call OgrePath ${DestPointX} ${DestPointY} ${DestPointZ}

    if ${Path.Hops}
    {
        ; Mapped path available
        call OgreMove
    }
    else
    {
        ; No path - try direct movement
        if ${CanMoveDirect[${DestPointX},${DestPointY},${DestPointZ}]}
        {
            call MoveDirectly ${DestPointX} ${DestPointY} ${DestPointZ}
        }
        else
        {
            echo "Cannot reach destination - no path and collision detected"
        }
    }
}

function OgrePath(float X, float Y, float Z)
{
    ; Generate path using LavishNav
    PathFinder:GeneratePath[${Me.X},${Me.Y},${Me.Z},${X},${Y},${Z},Path]
}

function OgreMove()
{
    variable int CurrentHop = 1
    variable point3f Waypoint

    StuckCounter:Set[0]
    LastLoc:Set[${Me.Loc}]

    while ${CurrentHop} <= ${Path.Hops}
    {
        Path.Hop[${CurrentHop}]:GetPosition[Waypoint]

        ; Move to this waypoint
        while ${Math.Distance[${Me.X},${Me.Y},${Me.Z},${Waypoint.X},${Waypoint.Y},${Waypoint.Z}]} > ${Precision}
        {
            ; Stuck check
            if ${Math.Distance[${Me.X},${Me.Y},${Me.Z},${LastLoc.X},${LastLoc.Y},${LastLoc.Z}]} < 0.5
            {
                StuckCounter:Inc
                if ${StuckCounter} > 10
                {
                    echo "Stuck detected at hop ${CurrentHop}"
                    call HandleStuck
                    StuckCounter:Set[0]
                }
            }
            else
            {
                StuckCounter:Set[0]
                LastLoc:Set[${Me.Loc}]
            }

            press -hold ${Forward}
            face ${Waypoint.X} ${Waypoint.Z}
            wait 10
        }

        CurrentHop:Inc
    }

    press -release ${Forward}
    echo "Navigation complete"
}

function HandleStuck()
{
    ; Unstuck routine
    echo "Attempting to unstuck..."

    press -release ${Forward}
    wait 5

    ; Back up
    press -hold ${Backward}
    wait 500
    press -release ${Backward}

    ; Try moving sideways
    press -hold ${StrafeLeft}
    wait 300
    press -release ${StrafeLeft}

    wait 10
}

function MoveDirectly(float X, float Y, float Z)
{
    ; Raw movement without pathfinding
    while ${Math.Distance[${Me.X},${Me.Y},${Me.Z},${X},${Y},${Z}]} > ${Precision}
    {
        press -hold ${Forward}
        face ${X} ${Z}
        wait 10
    }
    press -release ${Forward}
}

function LoadMap()
{
    ; Load navigation map for current zone
    if ${LNavRegion[${Zone.Name}](exists)}
    {
        LNavRegion[${Zone.Name}]:Load
        ZoneRegion:Set[${LNavRegion[${Zone.Name}]}]
        echo "Loaded navigation map for ${Zone.Name}"
    }
    else
    {
        echo "No navigation map available for ${Zone.Name}"
    }
}
```

---

## Reusable Timer Objects

### Overview

Instead of using inline `wait` commands or tracking time manually, create reusable timer objects that provide clean, reusable timing functionality.

### Basic Timer Object

```lavishscript
objectdef TimerObject
{
    variable uint EndTime

    method Set(uint Milliseconds)
    {
        EndTime:Set[${Milliseconds}+${Script.RunningTime}]
    }

    member:uint TimeLeft()
    {
        if ${Script.RunningTime}>=${EndTime}
            return 0
        return ${Math.Calc[${EndTime}-${Script.RunningTime}]}
    }

    member:bool Expired()
    {
        return ${Script.RunningTime}>=${EndTime}
    }
}
```

### Usage Examples

```lavishscript
; Create timer instances
variable TimerObject AbilityTimer
variable TimerObject HealthCheckTimer
variable TimerObject LootScanTimer

function main()
{
    ; Set timers
    AbilityTimer:Set[2000]      ; 2 seconds
    HealthCheckTimer:Set[1000]  ; 1 second
    LootScanTimer:Set[5000]     ; 5 seconds

    while 1
    {
        ; Check if ability timer expired
        if ${AbilityTimer.Expired}
        {
            call CastAbility
            AbilityTimer:Set[2000]  ; Reset for next cast
        }

        ; Check if health timer expired
        if !${HealthCheckTimer.TimeLeft}
        {
            call CheckHealth
            HealthCheckTimer:Set[1000]
        }

        ; Check if loot scan timer expired
        if ${LootScanTimer.Expired}
        {
            call ScanForLoot
            LootScanTimer:Set[5000]
        }

        wait 10
    }
}
```

### Advanced Timer with Pause/Resume

```lavishscript
objectdef AdvancedTimer
{
    variable uint EndTime
    variable uint PausedTime
    variable uint RemainingTime
    variable bool IsPaused = FALSE

    method Set(uint Milliseconds)
    {
        EndTime:Set[${Milliseconds}+${Script.RunningTime}]
        IsPaused:Set[FALSE]
    }

    member:uint TimeLeft()
    {
        if ${IsPaused}
            return ${RemainingTime}

        if ${Script.RunningTime}>=${EndTime}
            return 0
        return ${Math.Calc[${EndTime}-${Script.RunningTime}]}
    }

    member:bool Expired()
    {
        if ${IsPaused}
            return FALSE
        return ${Script.RunningTime}>=${EndTime}
    }

    method Pause()
    {
        if !${IsPaused}
        {
            RemainingTime:Set[${This.TimeLeft}]
            IsPaused:Set[TRUE]
        }
    }

    method Resume()
    {
        if ${IsPaused}
        {
            EndTime:Set[${RemainingTime}+${Script.RunningTime}]
            IsPaused:Set[FALSE]
        }
    }

    method Reset(uint Milliseconds)
    {
        This:Set[${Milliseconds}]
    }
}
```

### Multi-Timer Pulse System

```lavishscript
;===========================================
; Professional multi-timer architecture (like EQ2Bot)
;===========================================
objectdef TimerObject
{
    variable uint EndTime

    method Set(uint Milliseconds)
    {
        EndTime:Set[${Milliseconds}+${Script.RunningTime}]
    }

    member:uint TimeLeft()
    {
        if ${Script.RunningTime}>=${EndTime}
            return 0
        return ${Math.Calc[${EndTime}-${Script.RunningTime}]}
    }
}

; Timer instances for different frequencies
variable TimerObject Timer_1s
variable TimerObject Timer_2s
variable TimerObject Timer_5s
variable TimerObject Timer_10s

function main()
{
    ; Initialize timers
    Timer_1s:Set[1000]
    Timer_2s:Set[2000]
    Timer_5s:Set[5000]
    Timer_10s:Set[10000]

    while 1
    {
        ; 1 second tasks (frequent checks)
        if !${Timer_1s.TimeLeft}
        {
            call CheckCombatState
            call CheckHealth
            Timer_1s:Set[1000]
        }

        ; 2 second tasks (moderate frequency)
        if !${Timer_2s.TimeLeft}
        {
            call CastAbilities
            call CheckBuffs
            Timer_2s:Set[2000]
        }

        ; 5 second tasks (less frequent)
        if !${Timer_5s.TimeLeft}
        {
            call ScanForLoot
            call CheckGroupMembers
            Timer_5s:Set[5000]
        }

        ; 10 second tasks (infrequent checks)
        if !${Timer_10s.TimeLeft}
        {
            call UpdateStatistics
            call SaveConfiguration
            Timer_10s:Set[10000]
        }

        ; Queue processing
        if ${QueuedCommands}
            ExecuteQueued

        waitframe
    }
}
```

### Benefits

1. **Clean code** - No scattered wait commands
2. **Reusable** - Single timer object definition
3. **Efficient** - Check multiple timers per frame
4. **Flexible** - Pause/resume capability
5. **Professional** - Standard pattern in production scripts

---

## UI Synchronization Patterns

### Overview

When loading UI dynamically, scripts must wait for the UI to be fully loaded before applying settings or accessing UI elements. This pattern prevents crashes and ensures reliability.

### The Problem

```lavishscript
; BAD - UI not loaded yet, will crash
ui -reload "${LavishScript.HomeDirectory}/Scripts/MyBot/MyBotUI.xml"
UIElement[ChkAutoLoot]:SetChecked  ; CRASH! UI not ready
```

### The Solution: Wait for UI Ready Flag

```lavishscript
variable(global) bool MyBotUILoaded = FALSE

function main()
{
    ; Load UI
    ui -reload "${LavishScript.HomeDirectory}/Scripts/MyBot/MyBotUI.xml"

    ; Wait for UI to signal it's ready
    do
    {
        waitframe
    }
    while !${MyBotUILoaded}

    ; NOW safe to access UI
    call LoadSettingsToUI

    ; Main loop
    while ${MyBotUILoaded}
    {
        ; Check for UI changes
        if ${SettingsChanged}
        {
            call SaveSettingsFromUI
            SettingsChanged:Set[FALSE]
        }

        waitframe
    }
}
```

### UI XML: Setting the Ready Flag

```xml
<!-- MyBotUI.xml -->
<LGUI2Package>
    <Script>
        <![CDATA[
        function OnLoad()
        {
            ; Signal to main script that UI is ready
            MyBotUILoaded:Set[TRUE]
        }

        function OnUnload()
        {
            ; Signal UI is closing
            MyBotUILoaded:Set[FALSE]
        }
        ]]>
    </Script>

    <window name="MyBotWindow">
        <!-- UI elements here -->
    </window>
</LGUI2Package>
```

### Complete Example: Settings Synchronization

```lavishscript
;===========================================
; Main Script with UI Sync
;===========================================
variable(global) bool BotUILoaded = FALSE
variable(global) bool SettingsChanged = FALSE
variable settingsetref Config

function main()
{
    ; Load configuration
    call LoadConfiguration

    ; Load UI
    ui -reload -skin EQ2-Green "${LavishScript.HomeDirectory}/Scripts/MyBot/UI/BotUI.xml"

    ; Wait for UI to be fully loaded
    echo "Waiting for UI to load..."
    do
    {
        waitframe
    }
    while !${BotUILoaded}

    echo "UI loaded successfully"

    ; Apply saved settings to UI
    call LoadSettingsToUI

    ; Main loop
    while ${BotUILoaded}
    {
        ; Check if user changed settings in UI
        if ${SettingsChanged}
        {
            call SaveSettingsFromUI
            call ApplyNewSettings
            SettingsChanged:Set[FALSE]
        }

        ; Bot logic here
        call BotPulse

        waitframe
    }

    echo "UI closed - ending script"
}

function LoadSettingsToUI()
{
    ; Sync checkboxes
    if ${Config.FindSetting[AutoLoot,TRUE]}
        UIElement[ChkAutoLoot]:SetChecked
    else
        UIElement[ChkAutoLoot]:SetUnchecked

    if ${Config.FindSetting[AssistTank,TRUE]}
        UIElement[ChkAssistTank]:SetChecked
    else
        UIElement[ChkAssistTank]:SetUnchecked

    ; Sync text fields
    UIElement[TxtAssistAt]:SetText[${Config.FindSetting[AssistAt,95]}]
    UIElement[TxtLootRadius]:SetText[${Config.FindSetting[LootRadius,50]}]

    ; Sync combo boxes
    UIElement[CmbPullMode]:SelectItem[${Config.FindSetting[PullMode,"Tank Only"]}]

    echo "Settings loaded to UI"
}

function SaveSettingsFromUI()
{
    ; Read from UI and save to config
    Config.FindSetting[AutoLoot]:Set[${UIElement[ChkAutoLoot].Checked}]
    Config.FindSetting[AssistTank]:Set[${UIElement[ChkAssistTank].Checked}]
    Config.FindSetting[AssistAt]:Set[${UIElement[TxtAssistAt].Text}]
    Config.FindSetting[LootRadius]:Set[${UIElement[TxtLootRadius].Text}]
    Config.FindSetting[PullMode]:Set[${UIElement[CmbPullMode].SelectedItem.Text}]

    ; Export to file
    Config:Export["${Script.CurrentDirectory}/Config/${Me.Name}.xml"]

    echo "Settings saved from UI"
}

function ApplyNewSettings()
{
    ; Apply changed settings to script behavior
    script:AutoLootEnabled:Set[${Config.FindSetting[AutoLoot,TRUE]}]
    script:AssistTankEnabled:Set[${Config.FindSetting[AssistTank,TRUE]}]
    script:AssistPercentage:Set[${Config.FindSetting[AssistAt,95]}]

    echo "New settings applied to bot"
}

;===========================================
; UI XML with OnLoad/OnUnload
;===========================================
/**
<?xml version='1.0' encoding='UTF-8'?>
<LGUI2Package>
    <Script>
        <![CDATA[
        variable(global) int ChkAutoLootID
        variable(global) int ChkAssistTankID
        variable(global) int TxtAssistAtID
        variable(global) int TxtLootRadiusID
        variable(global) int CmbPullModeID

        function OnLoad()
        {
            ; Store UI element IDs
            ChkAutoLootID:Set[${UIElement[ChkAutoLoot].ID}]
            ChkAssistTankID:Set[${UIElement[ChkAssistTank].ID}]
            TxtAssistAtID:Set[${UIElement[TxtAssistAt].ID}]
            TxtLootRadiusID:Set[${UIElement[TxtLootRadius].ID}]
            CmbPullModeID:Set[${UIElement[CmbPullMode].ID}]

            ; Signal UI is ready
            BotUILoaded:Set[TRUE]

            echo "Bot UI loaded"
        }

        function OnUnload()
        {
            BotUILoaded:Set[FALSE]
            echo "Bot UI unloaded"
        }

        function OnSettingChanged()
        {
            ; Signal that user changed something
            SettingsChanged:Set[TRUE]
        }
        ]]>
    </Script>

    <window name='BotMainWindow' title='My Bot' x='100' y='100' width='400' height='300'>
        <checkbox name='ChkAutoLoot' x='10' y='10'>
            <text>Auto Loot</text>
            <OnLeftClick>OnSettingChanged</OnLeftClick>
        </checkbox>

        <checkbox name='ChkAssistTank' x='10' y='35'>
            <text>Assist Tank</text>
            <OnLeftClick>OnSettingChanged</OnLeftClick>
        </checkbox>

        <text x='10' y='60'>Assist At %:</text>
        <textentry name='TxtAssistAt' x='100' y='60' width='50'>
            <OnChange>OnSettingChanged</OnChange>
        </textentry>

        <text x='10' y='85'>Loot Radius:</text>
        <textentry name='TxtLootRadius' x='100' y='85' width='50'>
            <OnChange>OnSettingChanged</OnChange>
        </textentry>
    </window>
</LGUI2Package>
**/
```

### Best Practices

1. **Always wait for UI ready flag** before accessing elements
2. **Use global variables** for UI element IDs
3. **Signal changes** with a `SettingsChanged` flag
4. **Batch updates** - don't save on every change, use a flag
5. **OnLoad/OnUnload** - handle UI lifecycle properly

---

## Modern EQ2:GetActors Usage

### Overview

`EQ2:GetActors` is the **modern, recommended method** for getting actors. It directly populates an index with filtered actors, replacing the deprecated `CustomActorArray` system.

### Why Use EQ2:GetActors

- ✅ **Modern** - Current ISXEQ2 standard
- ✅ **Efficient** - One call to populate filtered actors
- ✅ **Flexible** - Multiple filter types
- ✅ **Fast** - Built into ISXEQ2 engine
- ❌ **CustomActorArray** - Deprecated, no longer supported

### Basic Usage

```lavishscript
variable index:actor Actors
variable iterator ActorIt

; Get all actors within 50 meters
EQ2:GetActors[Actors,Range,50]

; Iterate through results
Actors:GetIterator[ActorIt]
if ${ActorIt:First(exists)}
{
    do
    {
        echo "Actor: ${ActorIt.Value.Name} (ID: ${ActorIt.Value.ID}) Distance: ${ActorIt.Value.Distance}"
    }
    while ${ActorIt:Next(exists)}
}
```

### Filter Types

```lavishscript
; By Range
EQ2:GetActors[Actors,Range,100]  ; All actors within 100m

; By Type
EQ2:GetActors[Actors,Type,NPC]         ; All NPCs
EQ2:GetActors[Actors,Type,PC]          ; All player characters
EQ2:GetActors[Actors,Type,resource]    ; All harvestable resources
EQ2:GetActors[Actors,Type,door]        ; All doors/clickables

; Combined: Range + Type
EQ2:GetActors[Actors,Range,50,resource]  ; Resources within 50m
EQ2:GetActors[Actors,Range,100,NPC]      ; NPCs within 100m

; By Zone
EQ2:GetActors[Actors,Zone]  ; All actors in current zone
```

### Complete Example: Resource Harvesting

```lavishscript
;===========================================
; Modern Harvesting with EQ2:GetActors
;===========================================
variable index:actor Resources
variable iterator ResourceIt
variable int TargetResource = 0

function ScanForResources()
{
    ; Get all harvestable resources within 50 meters
    EQ2:GetActors[Resources,Range,50,resource]

    echo "Found ${Resources.Used} resources nearby"

    Resources:GetIterator[ResourceIt]
    if ${ResourceIt:First(exists)}
    {
        do
        {
            ; Check if this is a valid harvest target
            if ${IsValidResource[${ResourceIt.Value.ID}]}
            {
                ; Found one! Set as target
                TargetResource:Set[${ResourceIt.Value.ID}]
                echo "Targeting resource: ${ResourceIt.Value.Name} at ${ResourceIt.Value.Distance}m"
                return TRUE
            }
        }
        while ${ResourceIt:Next(exists)}
    }

    return FALSE
}

function IsValidResource(int ActorID)
{
    ; Validate the resource
    if !${Actor[${ActorID}](exists)}
        return FALSE

    ; Check if it's actually a resource
    if !${Actor[${ActorID}].Type.Equal[resource]}
        return FALSE

    ; Check distance
    if ${Actor[${ActorID}].Distance} > 50
        return FALSE

    ; Add your own filters here (node type, etc.)

    return TRUE
}

function HarvestResource(int ActorID)
{
    if !${Actor[${ActorID}](exists)}
        return

    ; Move close
    while ${Actor[${ActorID}].Distance} > 3 && ${Actor[${ActorID}](exists)}
    {
        press -hold ${Forward}
        face ${Actor[${ActorID}].X} ${Actor[${ActorID}].Z}
        wait 10
    }
    press -release ${Forward}

    ; Harvest
    Actor[${ActorID}]:DoTarget
    wait 10 ${Target.ID}==${ActorID}
    Actor[${ActorID}]:DoubleClick
    wait 10 ${Me.CastingSpell}
    wait 50 !${Me.CastingSpell}
}
```

### Finding Specific Actors

```lavishscript
;===========================================
; Find Nearest Enemy
;===========================================
variable index:actor Enemies
variable iterator EnemyIt
variable int NearestEnemy = 0
variable float NearestDistance = 999999

function FindNearestEnemy()
{
    NearestEnemy:Set[0]
    NearestDistance:Set[999999]

    ; Get all NPCs in range
    EQ2:GetActors[Enemies,Range,100,NPC]

    Enemies:GetIterator[EnemyIt]
    if ${EnemyIt:First(exists)}
    {
        do
        {
            ; Check if hostile
            if ${EnemyIt.Value.IsAggro} || ${EnemyIt.Value.ConColor.Equal[red]} || ${EnemyIt.Value.ConColor.Equal[yellow]}
            {
                if ${EnemyIt.Value.Distance} < ${NearestDistance}
                {
                    NearestEnemy:Set[${EnemyIt.Value.ID}]
                    NearestDistance:Set[${EnemyIt.Value.Distance}]
                }
            }
        }
        while ${EnemyIt:Next(exists)}
    }

    if ${NearestEnemy} > 0
    {
        echo "Nearest enemy: ${Actor[${NearestEnemy}].Name} at ${NearestDistance}m"
        return TRUE
    }

    return FALSE
}

;===========================================
; Find Specific Actor by Name
;===========================================
function FindActorByName(string ActorName)
{
    variable index:actor AllActors
    variable iterator ActorIt

    EQ2:GetActors[AllActors,Zone]

    AllActors:GetIterator[ActorIt]
    if ${ActorIt:First(exists)}
    {
        do
        {
            if ${ActorIt.Value.Name.Equal[${ActorName}]}
            {
                echo "Found ${ActorName} at ID ${ActorIt.Value.ID}"
                return ${ActorIt.Value.ID}
            }
        }
        while ${ActorIt:Next(exists)}
    }

    echo "${ActorName} not found"
    return 0
}

;===========================================
; Get All Group Members
;===========================================
function ScanGroupMembers()
{
    variable index:actor GroupMembers
    variable iterator MemberIt

    EQ2:GetActors[GroupMembers,Type,PC]

    echo "Group Members in zone:"
    GroupMembers:GetIterator[MemberIt]
    if ${MemberIt:First(exists)}
    {
        do
        {
            ; Check if in group
            if ${MemberIt.Value.InMyGroup}
            {
                echo "  ${MemberIt.Value.Name} - ${MemberIt.Value.Health}% HP - ${MemberIt.Value.Distance}m"
            }
        }
        while ${MemberIt:Next(exists)}
    }
}
```

### Performance Considerations

```lavishscript
; DON'T scan every frame
while 1
{
    EQ2:GetActors[Actors,Range,100]  ; BAD - too frequent!
    waitframe
}

; DO use timers to throttle scans
variable TimerObject ScanTimer
ScanTimer:Set[500]  ; Scan every half second

while 1
{
    if !${ScanTimer.TimeLeft}
    {
        EQ2:GetActors[Actors,Range,100]  ; GOOD - throttled
        ScanTimer:Set[500]
    }
    waitframe
}
```

---

## Trigger System for Chat Parsing

### Overview

The trigger system allows automatic parsing of chat messages using pattern matching with named placeholders. When a trigger pattern matches, it calls a function with extracted values as parameters.

### Basic Trigger Pattern

```lavishscript
; Define trigger
AddTrigger FunctionName "Pattern with @PLACEHOLDERS@"

; Pattern matches call this function
function FunctionName(string Line, string Param1, int Param2, ...)
{
    echo "Triggered! ${Param1}, ${Param2}"
}
```

### Common Placeholders

- `@TYPE@` - A word (string)
- `@NUMBER@` - A number (int)
- `@RAW@` - Everything up to next placeholder
- `@NAME@` - Actor/item name
- `@ZONE@` - Zone name

### Harvest Tracking Example

```lavishscript
;===========================================
; Harvest Trigger System
;===========================================

function main()
{
    ; Register harvest triggers
    AddTrigger HarvestedItem "You @TYPE@ @NUMBER@ @RAW@ from the @RESOURCENAME@."
    AddTrigger HarvestedRare "You @TYPE@ a @RAW@ from the @RESOURCENAME@."
    AddTrigger HarvestedCollectible "You found a @TYPE@."
    AddTrigger HarvestFailed "You fail to gather anything."

    ; Main loop
    while 1
    {
        ; Triggers fire automatically when chat matches
        waitframe
    }
}

; Called when "You harvest 5 iron cluster from the metal deposit." appears
function HarvestedItem(string Line, string HarvestType, int Amount, string ItemName, string ResourceName)
{
    echo "Harvested ${Amount} ${ItemName} from ${ResourceName}"

    ; Update statistics
    TotalHarvests:Inc
    TotalItems:Set[${Math.Calc[${TotalItems}+${Amount}]}]

    ; Log to file
    call LogHarvest "${ItemName}" ${Amount} "${ResourceName}"
}

; Called when "You harvest a pristine ore from the metal deposit." appears
function HarvestedRare(string Line, string HarvestType, string ItemName, string ResourceName)
{
    echo "*** RARE: ${ItemName} from ${ResourceName} ***"
    TotalRares:Inc
}

; Called when "You found a Qeynosian relic." appears
function HarvestedCollectible(string Line, string CollectibleType)
{
    echo "Found collectible: ${CollectibleType}"
    TotalCollectibles:Inc
}

; Called when "You fail to gather anything." appears
function HarvestFailed(string Line)
{
    echo "Harvest attempt failed"
    TotalFailures:Inc
}
```

### Combat Triggers

```lavishscript
;===========================================
; Combat Event Triggers
;===========================================

function RegisterCombatTriggers()
{
    ; Damage triggers
    AddTrigger IncomingDamage "You are struck by @NAME@ for @NUMBER@ @TYPE@ damage."
    AddTrigger OutgoingDamage "You hit @NAME@ for @NUMBER@ @TYPE@ damage."

    ; Death triggers
    AddTrigger EnemyDeath "@NAME@ has been slain!"
    AddTrigger PlayerDeath "You have been slain by @NAME@!"

    ; Spell triggers
    AddTrigger SpellResisted "Your spell fizzles!"
    AddTrigger SpellInterrupted "Your spell is interrupted!"

    ; Loot triggers
    AddTrigger LootReceived "You receive loot: @RAW@."
}

function IncomingDamage(string Line, string AttackerName, int Damage, string DamageType)
{
    echo "Taking ${Damage} ${DamageType} damage from ${AttackerName}"

    ; Update threat meter
    ThreatMeter[${AttackerName}]:Inc[${Damage}]

    ; Emergency heal if big hit
    if ${Damage} > 5000
    {
        echo "BIG HIT! Emergency heal!"
        call CastEmergencyHeal
    }
}

function OutgoingDamage(string Line, string TargetName, int Damage, string DamageType)
{
    ; Track DPS
    TotalDamage:Set[${Math.Calc[${TotalDamage}+${Damage}]}]
}

function EnemyDeath(string Line, string EnemyName)
{
    echo "${EnemyName} defeated!"
    KillCount:Inc

    ; Clear from threat table
    ThreatMeter:Erase[${EnemyName}]
}

function LootReceived(string Line, string ItemName)
{
    echo "Looted: ${ItemName}"

    ; Check if rare
    if ${ItemName.Find[Fabled]} || ${ItemName.Find[Legendary]}
    {
        echo "*** RARE LOOT: ${ItemName} ***"
        call PlaySound "rare_loot.wav"
    }
}
```

### Group/Tell Triggers

```lavishscript
;===========================================
; Communication Triggers
;===========================================

function RegisterChatTriggers()
{
    ; Group chat
    AddTrigger GroupChat "@NAME@ tells the group, '@RAW@'"

    ; Tells
    AddTrigger IncomingTell "@NAME@ tells you, '@RAW@'"

    ; System messages
    AddTrigger GroupInvite "@NAME@ invites you to join a group."
    AddTrigger GuildInvite "@NAME@ invites you to join @GUILD@."
}

function GroupChat(string Line, string SenderName, string Message)
{
    echo "[Group] ${SenderName}: ${Message}"

    ; Check for commands
    if ${Message.Find[!assist]}
    {
        call AssistPlayer "${SenderName}"
    }

    if ${Message.Find[!heal]}
    {
        call HealPlayer "${SenderName}"
    }
}

function IncomingTell(string Line, string SenderName, string Message)
{
    echo "[Tell from ${SenderName}] ${Message}"

    ; Auto-responses
    if ${Message.Find[invite]}
    {
        EQ2Execute /invite ${SenderName}
    }

    ; Command system
    if ${Message.Find[!status]}
    {
        EQ2Execute /tell ${SenderName} HP: ${Me.Health}% Power: ${Me.Power}%
    }
}
```

### Zone/Quest Triggers

```lavishscript
;===========================================
; Zone and Quest Triggers
;===========================================

function RegisterZoneTriggers()
{
    ; Zone messages
    AddTrigger ZoneEntered "Entering @ZONE@"
    AddTrigger ZoneExited "Leaving @ZONE@"

    ; Quest updates
    AddTrigger QuestComplete "You have completed the quest '@QUEST@'"
    AddTrigger QuestUpdate "Your journal has been updated."

    ; Achievement
    AddTrigger Achievement "You have earned the achievement: @RAW@"
}

function ZoneEntered(string Line, string ZoneName)
{
    echo "Entered zone: ${ZoneName}"

    ; Load zone-specific settings
    call LoadZoneConfiguration "${ZoneName}"

    ; Load navigation map
    call LoadNavigationMap
}

function QuestComplete(string Line, string QuestName)
{
    echo "Completed quest: ${QuestName}"
    CompletedQuests:Inc

    ; Play sound
    call PlaySound "quest_complete.wav"
}

function Achievement(string Line, string AchievementName)
{
    echo "*** ACHIEVEMENT: ${AchievementName} ***"
}
```

### Advanced Pattern Matching

```lavishscript
;===========================================
; Complex Patterns
;===========================================

; Multiple numbers
AddTrigger CriticalHit "You critically hit @NAME@ for @NUMBER@ damage! (@NUMBER@ overkill)"

function CriticalHit(string Line, string TargetName, int Damage, int Overkill)
{
    echo "CRIT! ${Damage} damage (${Overkill} overkill) on ${TargetName}"
}

; Optional text
AddTrigger ExperienceGain "You gain @NUMBER@ experience"
AddTrigger ExperienceGainAA "You gain @NUMBER@ experience (@NUMBER@ AA)"

function ExperienceGain(string Line, int XP)
{
    TotalXP:Set[${Math.Calc[${TotalXP}+${XP}]}]
}

function ExperienceGainAA(string Line, int XP, int AA)
{
    TotalXP:Set[${Math.Calc[${TotalXP}+${XP}]}]
    TotalAA:Set[${Math.Calc[${TotalAA}+${AA}]}]
}
```

### Best Practices

1. **Register triggers early** - Usually in main() before loop
2. **Be specific** - More specific patterns = fewer false matches
3. **Test patterns** - Echo parameters to verify correct parsing
4. **Handle all cases** - Account for variations in messages
5. **Clean up** - Remove triggers when no longer needed

---

## Script Controller Pattern

### Overview

The controller pattern manages shared resources (like custom actor arrays) and automatically cleans up when no scripts are using it. This prevents resource leaks and ensures efficient operation.

### Core Concept

- **Reference counting** - Track which scripts are using the resource
- **Auto-cleanup** - Terminate when reference count hits zero
- **Delayed check** - Use `timedcommand` to avoid same-frame issues

### Basic Controller Script

```lavishscript
;===========================================
; ResourceController.iss
;===========================================

; Global collection tracks which scripts are using this
variable(global) collection:string ScriptsUsingController

function main()
{
    if !${ControllerOb(exists)}
        declarevariable ControllerOb ControllerObject global

    echo "Resource Controller started"

    while 1
        waitframe
}

objectdef ControllerObject
{
    method Load(string ScriptName, string Options)
    {
        ScriptsUsingController:Set[${ScriptName},${Options}]
        echo "Controller: ${ScriptName} registered with options: ${Options}"
        This:UpdateResource
    }

    method UnLoad(string ScriptName)
    {
        if ${ScriptsUsingController.Element[${ScriptName}](exists)}
        {
            ScriptsUsingController:Erase[${ScriptName}]
            echo "Controller: ${ScriptName} unregistered"
            This:UpdateResource
        }
    }

    method UpdateResource()
    {
        if !${ScriptsUsingController.FirstKey(exists)}
        {
            ; No scripts using this anymore - cleanup after delay
            echo "Controller: No more scripts registered - cleaning up in 10 seconds"
            timedcommand 100 ControllerOb:CleanUp
        }
        else
        {
            echo "Controller: ${ScriptsUsingController.Used} scripts registered"
        }
    }

    method CleanUp()
    {
        ; Double-check after delay
        if !${ScriptsUsingController.FirstKey(exists)}
        {
            echo "Controller: Shutting down - no scripts using resource"
            Script:End
        }
        else
        {
            echo "Controller: Scripts registered again, cancelling shutdown"
        }
    }
}
```

### Using the Controller

```lavishscript
;===========================================
; Client Script
;===========================================

function main()
{
    ; Start controller if not running
    if !${Script[ResourceController](exists)}
    {
        runscript "${LavishScript.HomeDirectory}/Scripts/ResourceController"
        waitframe
        wait ${Script[ResourceController](exists)}
        waitframe
    }

    ; Register with controller
    ControllerOb:Load[${Script.Filename},"MyOptions"]

    ; Main loop
    while 1
    {
        ; Use shared resource
        waitframe
    }
}

function OnEnd()
{
    ; Unregister when script ends
    ControllerOb:UnLoad[${Script.Filename}]
}
```

### Advanced: CustomActorArray Controller

```lavishscript
;===========================================
; OgreCustomArrayController.iss  (from EQ2OgreFree, not modern day Ogre)
;===========================================

variable(global) collection:int ScriptsUsingArray  ; ScriptName -> Distance
variable(global) int MaxDistance = 0

function main()
{
    if !${ArrayControllerOb(exists)}
        declarevariable ArrayControllerOb ArrayControllerObject global

    echo "CustomActorArray Controller started"

    while 1
        waitframe
}

objectdef ArrayControllerObject
{
    variable int UpdateInterval = 1000
    variable TimerObject UpdateTimer

    method Load(string ScriptName, int Distance)
    {
        ScriptsUsingArray:Set[${ScriptName},${Distance}]
        echo "Array Controller: ${ScriptName} registered (distance: ${Distance})"
        This:UpdateMaxDistance
    }

    method UnLoad(string ScriptName)
    {
        if ${ScriptsUsingArray.Element[${ScriptName}](exists)}
        {
            ScriptsUsingArray:Erase[${ScriptName}]
            echo "Array Controller: ${ScriptName} unregistered"
            This:UpdateMaxDistance
        }
    }

    method UpdateMaxDistance()
    {
        MaxDistance:Set[0]

        if ${ScriptsUsingArray.FirstKey(exists)}
        {
            do
            {
                if ${ScriptsUsingArray.CurrentValue} > ${MaxDistance}
                    MaxDistance:Set[${ScriptsUsingArray.CurrentValue}]
            }
            while ${ScriptsUsingArray.NextKey(exists)}

            echo "Array Controller: Max distance set to ${MaxDistance}"
        }
        else
        {
            ; No more users - shutdown
            echo "Array Controller: No scripts registered, shutting down in 10 seconds"
            timedcommand 100 ArrayControllerOb:CleanUp
        }
    }

    method Update()
    {
        if ${MaxDistance} > 0 && !${UpdateTimer.TimeLeft}
        {
            ; Update the actor array
            EQ2:GetActors[SharedActors,Range,${MaxDistance}]
            UpdateTimer:Set[${UpdateInterval}]
        }
    }

    method SetUpdateInterval(int Interval)
    {
        UpdateInterval:Set[${Interval}]
    }

    method CleanUp()
    {
        if !${ScriptsUsingArray.FirstKey(exists)}
        {
            echo "Array Controller: Shutting down"
            Script:End
        }
    }
}
```

### Shared Data Pattern

```lavishscript
;===========================================
; DataCacheController.iss - Shares expensive queries
;===========================================

variable(global) collection:string ActiveUsers
variable(global) index:item CachedInventory
variable(global) index:actor CachedEnemies
variable(global) bool DataReady = FALSE

function main()
{
    if !${CacheOb(exists)}
        declarevariable CacheOb DataCacheObject global

    while 1
    {
        CacheOb:Update
        waitframe
    }
}

objectdef DataCacheObject
{
    variable TimerObject InventoryScanTimer
    variable TimerObject EnemyScanTimer

    method Initialize()
    {
        InventoryScanTimer:Set[2000]  ; Update every 2 seconds
        EnemyScanTimer:Set[500]        ; Update every half second
    }

    method Register(string ScriptName)
    {
        ActiveUsers:Set[${ScriptName},"active"]
        echo "Data Cache: ${ScriptName} registered"
    }

    method Unregister(string ScriptName)
    {
        if ${ActiveUsers.Element[${ScriptName}](exists)}
        {
            ActiveUsers:Erase[${ScriptName}]
            echo "Data Cache: ${ScriptName} unregistered"

            if !${ActiveUsers.FirstKey(exists)}
            {
                timedcommand 100 CacheOb:Shutdown
            }
        }
    }

    method Update()
    {
        if ${ActiveUsers.Used} == 0
            return

        ; Update inventory cache
        if !${InventoryScanTimer.TimeLeft}
        {
            Me:QueryInventory[CachedInventory]
            InventoryScanTimer:Set[2000]
        }

        ; Update enemy cache
        if !${EnemyScanTimer.TimeLeft}
        {
            EQ2:GetActors[CachedEnemies,Range,100,NPC]
            EnemyScanTimer:Set[500]
        }

        DataReady:Set[TRUE]
    }

    method Shutdown()
    {
        if !${ActiveUsers.FirstKey(exists)}
        {
            echo "Data Cache: No active users, shutting down"
            Script:End
        }
    }
}

;===========================================
; Client script using cache
;===========================================
function main()
{
    ; Start cache if needed
    if !${Script[DataCacheController](exists)}
        runscript "${LavishScript.HomeDirectory}/Scripts/DataCacheController"

    wait ${Script[DataCacheController](exists)}

    ; Register
    CacheOb:Register[${Script.Filename}]

    ; Wait for first data
    wait ${DataReady}

    ; Use cached data
    while 1
    {
        ; Access CachedInventory and CachedEnemies directly
        echo "Inventory items: ${CachedInventory.Used}"
        echo "Enemies nearby: ${CachedEnemies.Used}"

        wait 1000
    }
}

function OnEnd()
{
    CacheOb:Unregister[${Script.Filename}]
}
```

### Benefits

1. **Resource efficiency** - Only one copy of shared data
2. **Auto-cleanup** - No manual shutdown required
3. **Safe shutdown** - Delayed check prevents race conditions
4. **Scalable** - Handles multiple clients automatically
5. **Professional** - Production-grade pattern

---

## Dynamic Variable Declaration

### Overview

Dynamic variable declaration allows scripts to check if objects/variables exist before creating them, and to declare variables at runtime with specific scopes. This is essential for shared resources and flexible script design.

### Variable Scopes

- **local** - Function-scoped (default)
- **script** - Script-scoped (accessible anywhere in script)
- **global** - Global scope (accessible from other scripts)

### Basic Pattern

```lavishscript
; Check if exists before declaring
if !${MyObject(exists)}
    declarevariable MyObject MyObjectType script

; Now safe to use
MyObject:DoSomething
```

### Shared Object Pattern

```lavishscript
;===========================================
; Navigation object shared across scripts
;===========================================

function InitializeNavigation()
{
    ; Declare if doesn't exist
    if !${Nav(exists)}
        declarevariable Nav WaypointNavigator script

    ; Configure
    Nav.Precision:Set[1.5]
    Nav.TargetRequired:Set[TRUE]
}

;===========================================
; Conditions object pattern
;===========================================

function InitializeConditions()
{
    if !${ConditionsOb(exists)}
        declarevariable ConditionsOb ConditionsObject script

    if !${InformationOb(exists)}
        declarevariable InformationOb InformationObject script
}
```

### Global Shared Objects

```lavishscript
;===========================================
; Main script creates global objects
;===========================================

function main()
{
    ; Create global objects for worker threads
    declarevariable SharedData DataObject global
    declarevariable SharedSettings SettingsObject global

    ; Initialize
    SharedData:Initialize
    SharedSettings:Load

    ; Start worker threads
    runscript WorkerThread1
    runscript WorkerThread2
}

;===========================================
; Worker thread accesses global objects
;===========================================

function main()
{
    ; Objects already exist globally - just use them
    while 1
    {
        ; Access global data
        echo "Shared value: ${SharedData.GetValue}"

        ; Modify global settings
        SharedSettings:UpdateSetting["LastCheck",${Script.RunningTime}]

        waitframe
    }
}
```

### Conditional Object Creation

```lavishscript
;===========================================
; Create objects based on class/configuration
;===========================================

function InitializeCombatObjects()
{
    ; Always create base combat object
    if !${CombatOb(exists)}
        declarevariable CombatOb CombatObject script

    ; Class-specific objects
    switch ${Me.SubClass}
    {
        case Wizard
        case Warlock
            if !${NukerOb(exists)}
                declarevariable NukerOb NukerObject script
            break

        case Guardian
        case Berserker
            if !${TankOb(exists)}
                declarevariable TankOb TankObject script
            break

        case Inquisitor
        case Templar
            if !${HealerOb(exists)}
                declarevariable HealerOb HealerObject script
            break
    }
}
```

### Dynamic Collection Management

```lavishscript
;===========================================
; Runtime collection declaration
;===========================================

function InitializeTracking()
{
    ; Declare collections if needed
    if !${DamageTracking(exists)}
        declarevariable DamageTracking collection:int script

    if !${HealTracking(exists)}
        declarevariable HealTracking collection:int script

    if !${ThreatTable(exists)}
        declarevariable ThreatTable collection:float script
}

function TrackDamage(string TargetName, int Amount)
{
    ; Safe to use - guaranteed to exist
    if ${DamageTracking.Element[${TargetName}](exists)}
        DamageTracking[${TargetName}]:Set[${Math.Calc[${DamageTracking[${TargetName}]}+${Amount}]}]
    else
        DamageTracking:Set[${TargetName},${Amount}]
}
```

### Module Loading Pattern

```lavishscript
;===========================================
; Load optional modules dynamically
;===========================================

function LoadModules()
{
    variable bool LoadCraftingModule = TRUE
    variable bool LoadHarvestModule = TRUE
    variable bool LoadPvPModule = FALSE

    if ${LoadCraftingModule}
    {
        if !${CraftingOb(exists)}
        {
            declarevariable CraftingOb CraftingObject script
            CraftingOb:Initialize
            echo "Crafting module loaded"
        }
    }

    if ${LoadHarvestModule}
    {
        if !${HarvestOb(exists)}
        {
            declarevariable HarvestOb HarvestObject script
            HarvestOb:Initialize
            echo "Harvest module loaded"
        }
    }

    if ${LoadPvPModule}
    {
        if !${PvPOb(exists)}
        {
            declarevariable PvPOb PvPObject script
            PvPOb:Initialize
            echo "PvP module loaded"
        }
    }
}

function UseCrafting()
{
    ; Check if module is loaded
    if ${CraftingOb(exists)}
    {
        CraftingOb:CraftItem["Iron Bar"]
    }
    else
    {
        echo "Crafting module not loaded"
    }
}
```

### Safe Initialization Pattern

```lavishscript
;===========================================
; Guarantee initialization order
;===========================================

function InitializeScript()
{
    echo "Initializing script..."

    ; Step 1: Core objects
    if !${ConfigOb(exists)}
        declarevariable ConfigOb ConfigObject script
    ConfigOb:Load

    ; Step 2: Utility objects
    if !${TimerOb(exists)}
        declarevariable TimerOb TimerManager script
    TimerOb:Initialize

    ; Step 3: Feature objects (depend on config)
    if ${ConfigOb.GetSetting["EnableCombat"]}
    {
        if !${CombatOb(exists)}
            declarevariable CombatOb CombatObject script
        CombatOb:Initialize
    }

    if ${ConfigOb.GetSetting["EnableLoot"]}
    {
        if !${LootOb(exists)}
            declarevariable LootOb LootObject script
        LootOb:Initialize
    }

    echo "Initialization complete"
}
```

### Testing Object Existence

```lavishscript
;===========================================
; Defensive programming with existence checks
;===========================================

function SafeCallNavigation(float X, float Y, float Z)
{
    if ${Nav(exists)}
    {
        Nav:MoveToLoc[${X},${Y},${Z}]
    }
    else
    {
        echo "ERROR: Navigation object not initialized"
        call InitializeNavigation
        Nav:MoveToLoc[${X},${Y},${Z}]
    }
}

function SafeReadSetting(string SettingName)
{
    if ${ConfigOb(exists)}
    {
        return ${ConfigOb.GetSetting["${SettingName}"]}
    }
    else
    {
        echo "WARNING: Config object not initialized, using default"
        return "DEFAULT"
    }
}
```

### Best Practices

1. **Always check existence** before declaring
2. **Use appropriate scope** - script for local, global for shared
3. **Initialize in order** - dependencies before dependents
4. **Document scope** - Comment why global vs script
5. **Clean pattern** - Consistent across all modules

---

## Injectable UI Components

### Overview

Injectable UI allows loading UI components into existing windows dynamically, creating modular, composable interfaces. This is useful for adding tabs to existing UIs or creating plugin-style extensions.

### Basic Injection Pattern

```lavishscript
;===========================================
; Main UI with Tab Container
;===========================================

variable(global) int MainTabControlID

function main()
{
    ; Load main UI shell
    ui -reload "${LavishScript.HomeDirectory}/Scripts/MyBot/UI/MainWindow.xml"

    ; Wait for UI ready
    wait ${MainUILoaded}

    ; Inject additional tabs
    runscript "${LavishScript.HomeDirectory}/Scripts/MyBot/InjectCombatTab" ${MainTabControlID} 1
    runscript "${LavishScript.HomeDirectory}/Scripts/MyBot/InjectLootTab" ${MainTabControlID} 2
    runscript "${LavishScript.HomeDirectory}/Scripts/MyBot/InjectSettingsTab" ${MainTabControlID} 3

    while ${MainUILoaded}
    {
        waitframe
    }
}
```

### Injectable Tab Script

```lavishscript
;===========================================
; InjectCombatTab.iss
;===========================================

function main(int TabControlID, int TabPosition)
{
    if !${TabControlID}
    {
        echo "ERROR: No tab control ID provided"
        return
    }

    echo "Injecting Combat tab into control ${TabControlID} at position ${TabPosition}"

    ; Load the tab UI
    ui -reload "${LavishScript.HomeDirectory}/Scripts/MyBot/UI/CombatTab.xml"

    ; Wait for tab to load
    wait ${CombatTabLoaded}

    ; Inject into parent tab control
    UIElement[${TabControlID}]:InsertTab[${TabPosition},"Combat",${CombatTabID}]

    echo "Combat tab injected successfully"

    ; Keep script running to maintain tab
    while ${CombatTabLoaded} && ${MainUILoaded}
    {
        ; Tab logic here
        waitframe
    }

    echo "Combat tab unloading"
}
```

### Complete Modular UI System

```lavishscript
;===========================================
; MainBot.iss - Main Script
;===========================================

variable(global) bool MainUILoaded = FALSE
variable(global) int MainTabsID

function main()
{
    ; Load shell UI
    ui -reload "${LavishScript.HomeDirectory}/Scripts/BotUI/Shell.xml"

    ; Wait for shell
    do
    {
        waitframe
    }
    while !${MainUILoaded}

    echo "Main UI loaded - injecting tabs"

    ; Inject all tabs
    call InjectAllTabs

    ; Main loop
    while ${MainUILoaded}
    {
        ; Main bot logic
        call BotPulse

        waitframe
    }

    echo "Main UI closed"
}

function InjectAllTabs()
{
    variable int TabPos = 1

    ; Status tab (always first)
    runscript "${LavishScript.HomeDirectory}/Scripts/BotUI/Tabs/StatusTab" ${MainTabsID} ${TabPos}
    TabPos:Inc

    ; Combat tab
    if ${LoadCombatModule}
    {
        runscript "${LavishScript.HomeDirectory}/Scripts/BotUI/Tabs/CombatTab" ${MainTabsID} ${TabPos}
        TabPos:Inc
    }

    ; Loot tab
    if ${LoadLootModule}
    {
        runscript "${LavishScript.HomeDirectory}/Scripts/BotUI/Tabs/LootTab" ${MainTabsID} ${TabPos}
        TabPos:Inc
    }

    ; Settings tab (always last)
    runscript "${LavishScript.HomeDirectory}/Scripts/BotUI/Tabs/SettingsTab" ${MainTabsID} ${TabPos}

    echo "All tabs injected"
}

;===========================================
; Shell.xml - Main UI Container
;===========================================
/**
<?xml version='1.0' encoding='UTF-8'?>
<LGUI2Package>
    <Script>
        <![CDATA[
        variable(global) int MainTabsID

        function OnLoad()
        {
            MainTabsID:Set[${UIElement[MainTabs].ID}]
            MainUILoaded:Set[TRUE]
            echo "Main UI shell loaded"
        }

        function OnUnload()
        {
            MainUILoaded:Set[FALSE]
            echo "Main UI shell unloaded"
        }
        ]]>
    </Script>

    <window name='BotMainWindow' title='My Bot' x='100' y='100' width='500' height='400'>
        <tabs name='MainTabs' x='5' y='25' width='490' height='370'>
            <!-- Tabs will be injected here dynamically -->
        </tabs>

        <button name='BtnClose' x='400' y='5' width='90' height='20'>
            <text>Close</text>
            <OnLeftClick>Script:End</OnLeftClick>
        </button>
    </window>
</LGUI2Package>
**/

;===========================================
; StatusTab.iss - Injectable Tab
;===========================================

variable(global) bool StatusTabLoaded = FALSE
variable(global) int StatusTabPanelID

function main(int ParentTabControlID, int Position)
{
    echo "Loading Status tab..."

    ; Load tab UI
    ui -reload "${LavishScript.HomeDirectory}/Scripts/BotUI/Tabs/StatusTab.xml"

    ; Wait for load
    do
    {
        waitframe
    }
    while !${StatusTabLoaded}

    ; Inject into parent
    UIElement[${ParentTabControlID}]:InsertTab[${Position},"Status",${StatusTabPanelID}]

    echo "Status tab injected at position ${Position}"

    ; Update loop
    while ${StatusTabLoaded} && ${MainUILoaded}
    {
        ; Update status display
        UIElement[TxtCharName]:SetText[${Me.Name}]
        UIElement[TxtHealth]:SetText[${Me.Health}%]
        UIElement[TxtPower]:SetText[${Me.Power}%]
        UIElement[TxtLevel]:SetText[${Me.Level}]

        wait 100
    }

    echo "Status tab unloading"
}

;===========================================
; StatusTab.xml
;===========================================
/**
<?xml version='1.0' encoding='UTF-8'?>
<LGUI2Package>
    <Script>
        <![CDATA[
        variable(global) int StatusTabPanelID

        function OnLoad()
        {
            StatusTabPanelID:Set[${UIElement[StatusPanel].ID}]
            StatusTabLoaded:Set[TRUE]
        }

        function OnUnload()
        {
            StatusTabLoaded:Set[FALSE]
        }
        ]]>
    </Script>

    <panel name='StatusPanel' x='0' y='0' width='480' height='340'>
        <text x='10' y='10'>Character:</text>
        <text name='TxtCharName' x='100' y='10'></text>

        <text x='10' y='35'>Health:</text>
        <text name='TxtHealth' x='100' y='35'></text>

        <text x='10' y='60'>Power:</text>
        <text name='TxtPower' x='100' y='60'></text>

        <text x='10' y='85'>Level:</text>
        <text name='TxtLevel' x='100' y='85'></text>
    </panel>
</LGUI2Package>
**/
```

### Dynamic Tab Loading

```lavishscript
;===========================================
; Load tabs based on configuration
;===========================================

function LoadConfiguredTabs()
{
    variable int TabPos = 1
    variable settingsetref TabSettings

    ; Read tab configuration
    TabSettings:Set[${Config.FindSet[EnabledTabs]}]

    if ${TabSettings.FindSetting[Status,TRUE]}
    {
        runscript "${LavishScript.HomeDirectory}/Scripts/BotUI/Tabs/StatusTab" ${MainTabsID} ${TabPos}
        TabPos:Inc
    }

    if ${TabSettings.FindSetting[Combat,TRUE]}
    {
        runscript "${LavishScript.HomeDirectory}/Scripts/BotUI/Tabs/CombatTab" ${MainTabsID} ${TabPos}
        TabPos:Inc
    }

    if ${TabSettings.FindSetting[Harvest,FALSE]}
    {
        runscript "${LavishScript.HomeDirectory}/Scripts/BotUI/Tabs/HarvestTab" ${MainTabsID} ${TabPos}
        TabPos:Inc
    }

    if ${TabSettings.FindSetting[Advanced,FALSE]}
    {
        runscript "${LavishScript.HomeDirectory}/Scripts/BotUI/Tabs/AdvancedTab" ${MainTabsID} ${TabPos}
        TabPos:Inc
    }
}
```

### Plugin Architecture

```lavishscript
;===========================================
; Plugin loader for extensible UI
;===========================================

function LoadPlugins()
{
    variable string PluginDir
    PluginDir:Set["${LavishScript.HomeDirectory}/Scripts/BotUI/Plugins"]

    ; Get list of plugin files
    variable filelist PluginFiles
    PluginFiles:GetFiles["${PluginDir}","*.iss"]

    if ${PluginFiles.Files}
    {
        variable int TabPos = 10  ; Start plugins after core tabs
        variable int i

        for (i:Set[1]; ${i} <= ${PluginFiles.Files}; i:Inc)
        {
            echo "Loading plugin: ${PluginFiles.File[${i}]}"
            runscript "${PluginDir}/${PluginFiles.File[${i}]}" ${MainTabsID} ${TabPos}
            TabPos:Inc
        }
    }
}

;===========================================
; Example plugin structure
;===========================================
/**
Plugins/CustomFeatureTab.iss:

function main(int ParentTabID, int Position)
{
    ; Load custom UI
    ui -reload "${Script.CurrentDirectory}/CustomFeature.xml"

    wait ${CustomFeatureLoaded}

    ; Inject
    UIElement[${ParentTabID}]:InsertTab[${Position},"Custom Feature",${CustomFeaturePanelID}]

    ; Custom logic
    while ${CustomFeatureLoaded} && ${MainUILoaded}
    {
        call CustomFeatureUpdate
        waitframe
    }
}
**/
```

### Benefits

1. **Modularity** - Each tab is independent
2. **Extensibility** - Easy to add new tabs
3. **Maintainability** - Separate files for each feature
4. **Flexibility** - Load tabs based on configuration
5. **Professional** - Clean plugin architecture

---

## 11. UI Initialization Guard Pattern

### Overview

The UI Initialization Guard pattern prevents unintended event handlers from firing during programmatic UI initialization. When setting dropdown values, checkboxes, or other UI elements from code, their `OnSelect` or `OnChange` events can trigger prematurely, causing incorrect behavior or infinite loops.

**Source:** EQ2Track script (https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Track)

### The Problem

```lavishscript
; BAD: OnSelect fires during initialization
function main()
{
    ui -reload "MyScript.xml"

    ; This triggers OnSelect event!
    UIElement[MyCombo]:SelectItem[1]

    ; This also triggers OnSelect!
    UIElement[MyCombo]:Sort
}

; In XML:
; <OnSelect>
;     LoadConfiguration  ; Fires during init!
; </OnSelect>
```

### The Solution: Guard Flag

```lavishscript
;===========================================
; Use a boolean flag to guard event handlers
;===========================================

variable bool ComboBox_executeOnSelect = FALSE

function main()
{
    ; Load UI
    ui -reload "MyScript.xml"

    ; Set flag to FALSE during initialization
    ComboBox_executeOnSelect:Set[FALSE]

    ; Safe to manipulate UI - events won't fire
    UIElement[MyCombo]:ClearItems
    UIElement[MyCombo]:AddItem["Option 1"]
    UIElement[MyCombo]:AddItem["Option 2"]
    UIElement[MyCombo]:Sort
    UIElement[MyCombo]:SelectItem[1]

    ; Enable event handling after init complete
    ComboBox_executeOnSelect:Set[TRUE]

    ; Now OnSelect events will work normally
}
```

### XML Integration

```xml
<Combobox name='MyCombo'>
    <X>100</X>
    <Y>27</Y>
    <Width>75</Width>
    <Height>20</Height>
    <Sort>Text</Sort>
    <OnLoad>
        ; Initialize combo from saved settings
        MyInterface:UpdateComboList
    </OnLoad>
    <OnSelect>
        ; Guard against initialization events
        if ${Script[MyScript].Variable[ComboBox_executeOnSelect]}
        {
            MyInterface:LoadConfiguration
            This:SetFocus
        }
    </OnSelect>
</Combobox>
```

### Complete Example from EQ2Track

```lavishscript
;===========================================
; Full implementation from EQ2Track
;===========================================

variable bool TrackListCombo_executeOnSelect = FALSE

function main()
{
    ; Guard flag starts as FALSE
    TrackListCombo_executeOnSelect:Set[FALSE]

    ; Load UI (OnLoad fires and populates combo)
    ui -reload "${Script.CurrentDirectory}/UI/EQ2Track.xml"

    ; Perform additional initialization
    variable filelist ListFiles
    variable int Count = 0

    ListFiles:GetFiles[${LavishScript.HomeDirectory}/Scripts/EQ2Track/Saved Lists/\*.xml]
    while ${Count:Inc} <= ${ListFiles.Files}
    {
        UIElement[TrackListCombo]:AddItem[${ListFiles.File[${Count}].Filename.Left[-4]}]
    }

    ; Sort and select first item (without triggering load)
    UIElement[TrackListCombo]:Sort:SelectItem[1]

    ; Enable event handling
    TrackListCombo_executeOnSelect:Set[TRUE]

    ; Main loop
    do
    {
        waitframe
    }
    while 1
}

; In objectdef interface:
method LoadList()
{
    ; This only executes when user selects from combo
    ; Not during initialization!
    This:LoadListByName[${UIElement[TrackListCombo].SelectedItem.Text}]
}
```

### Best Practices

1. **Always use guard flags for:**
   - Dropdown/Combobox initialization
   - Checkbox state setting from config
   - Radio button group setup
   - Slider value restoration

2. **Initialize in correct order:**
   ```lavishscript
   ; 1. Load UI
   ; 2. Set guard = FALSE
   ; 3. Populate/configure UI elements
   ; 4. Set guard = TRUE
   ; 5. Enter main loop
   ```

3. **Clear naming convention:**
   ```lavishscript
   variable bool FilterCombo_executeOnSelect = FALSE
   variable bool SortCombo_executeOnChange = FALSE
   variable bool AutoSave_executeOnCheck = FALSE
   ```

4. **Per-element guards:**
   ```lavishscript
   ; Don't use one global guard for all UI
   ; Use specific guards for each element
   variable bool Combo1_executeOnSelect = FALSE
   variable bool Combo2_executeOnSelect = FALSE
   ```

### Common Mistakes

```lavishscript
; WRONG: Using global "initialized" flag
variable bool UIInitialized = FALSE

; OnSelect handlers check global flag
; Problem: All elements share same flag!

; CORRECT: Per-element flags
variable bool TrackCombo_executeOnSelect = FALSE
variable bool SortCombo_executeOnSelect = FALSE
```

---

## 12. Collection-Based Exclusion Lists

### Overview

Collections provide fast O(1) lookup for maintaining exclusion lists, blacklists, or "seen" tracking. Unlike arrays which require linear searches, collections use hash tables for instant existence checks.

**Source:** EQ2Track script (https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Track)

### Why Collections Over Arrays

```lavishscript
; BAD: Array-based exclusion (O(n) lookup)
variable string BadActors[1000]
variable int BadActorCount = 0

; Check requires loop through entire array
function IsActorBad(int ActorID)
{
    variable int i = 1
    while ${i} <= ${BadActorCount}
    {
        if ${BadActors[${i}].Equal[${ActorID}]}
            return TRUE
        i:Inc
    }
    return FALSE
}

; GOOD: Collection-based exclusion (O(1) lookup)
variable collection:string BadActors

; Instant lookup!
if ${BadActors.Element[${ActorID}](exists)}
    return TRUE
```

### Basic Pattern

```lavishscript
;===========================================
; Actor blacklist with collection
;===========================================

variable collection:string BadActors

function MarkActorAsBad(int ActorID, string ActorName)
{
    if ${BadActors.Element[${ActorID}](exists)}
    {
        echo "Already marked as bad: ${ActorName}"
        return
    }

    ; Add to exclusion list
    BadActors:Set[${ActorID},${ActorName}]
    echo "Marked ${ActorName} (${ActorID}) as bad"
}

function IsActorBad(int ActorID)
{
    ; O(1) instant lookup
    return ${BadActors.Element[${ActorID}](exists)}
}

function ClearBadActors()
{
    BadActors:Clear
}
```

### Integration with Actor Scanning

```lavishscript
;===========================================
; Skip bad actors during scan
;===========================================

function ScanActors()
{
    variable index:actor Actors
    variable iterator ActorIt

    EQ2:GetActors[Actors]
    Actors:GetIterator[ActorIt]

    if ${ActorIt:First(exists)}
    {
        do
        {
            ; Fast exclusion check
            if ${BadActors.Element[${ActorIt.Value.ID}](exists)}
            {
                ; Skip this actor entirely
                continue
            }

            ; Process actor
            echo "Found: ${ActorIt.Value.Name}"
        }
        while ${ActorIt:Next(exists)}
    }
}
```

### UI Integration

```lavishscript
;===========================================
; Mark current target as bad via UI button
;===========================================

objectdef _TrackInterface
{
    method MarkTargetAsBad()
    {
        if !${Target(exists)}
        {
            echo "No target selected"
            return
        }

        if ${BadActors.Element[${Target.ID}](exists)}
        {
            echo "${Target.Name} already marked as bad"
            return
        }

        ; Add to blacklist
        BadActors:Set[${Target.ID},${Target.Name}]

        ; Cancel waypoint if active
        eq2execute "/waypoint_cancel"

        ; Remove from tracking list if present
        call RemoveFromTrackingList[${Target.ID}]
    }

    method UnmarkActor(int ActorID)
    {
        if ${BadActors.Element[${ActorID}](exists)}
        {
            BadActors:Erase[${ActorID}]
            echo "Removed actor ${ActorID} from bad list"
        }
    }
}
```

### XML Button

```xml
<Button Name='MarkBad'>
    <X>134</X>
    <Y>50</Y>
    <Width>51</Width>
    <Height>20</Height>
    <AutoTooltip>Mark Current Target as 'bad' and no longer track</AutoTooltip>
    <Text>Bad Tgt</Text>
    <OnLeftClick>
        if ${Target(exists)}
        {
            TrackInterface:MarkTargetAsBad
        }
    </OnLeftClick>
</Button>
```

### Persistence Pattern

```lavishscript
;===========================================
; Save/Load bad actor list
;===========================================

function SaveBadActors()
{
    variable iterator BadActorIt
    variable settingsetref BadList

    LavishSettings:AddSet[BadActorList]
    BadList:Set[${LavishSettings[BadActorList]}]

    BadActors:GetIterator[BadActorIt]
    if ${BadActorIt:First(exists)}
    {
        do
        {
            ; Key = ActorID, Value = ActorName
            BadList:AddSetting[${BadActorIt.Key},${BadActorIt.Value}]
        }
        while ${BadActorIt:Next(exists)}
    }

    BadList:Export["${Script.CurrentDirectory}/BadActors.xml"]
    LavishSettings[BadActorList]:Remove
}

function LoadBadActors()
{
    variable iterator SettingIt
    variable settingsetref BadList

    LavishSettings:AddSet[BadActorList]
    BadList:Set[${LavishSettings[BadActorList]}]
    BadList:Import["${Script.CurrentDirectory}/BadActors.xml"]

    BadList:GetSettingIterator[SettingIt]
    if ${SettingIt:First(exists)}
    {
        do
        {
            ; Restore to collection
            BadActors:Set[${SettingIt.Key},${SettingIt.Value}]
        }
        while ${SettingIt:Next(exists)}
    }

    LavishSettings[BadActorList]:Remove
}
```

### Advanced: Temporary Blacklist

```lavishscript
;===========================================
; Time-based temporary blacklist
;===========================================

variable collection:int BadActorExpiry

function MarkActorAsBadTemporary(int ActorID, int DurationSeconds)
{
    variable int ExpiryTime
    ExpiryTime:Set[${Math.Calc[${Script.RunningTime} + (${DurationSeconds} * 1000)]}]

    BadActors:Set[${ActorID},"temp"]
    BadActorExpiry:Set[${ActorID},${ExpiryTime}]
}

function CleanupExpiredBadActors()
{
    variable iterator ExpiryIt

    BadActorExpiry:GetIterator[ExpiryIt]
    if ${ExpiryIt:First(exists)}
    {
        do
        {
            if ${Script.RunningTime} >= ${ExpiryIt.Value}
            {
                ; Expired, remove from both collections
                BadActors:Erase[${ExpiryIt.Key}]
                BadActorExpiry:Erase[${ExpiryIt.Key}]
            }
        }
        while ${ExpiryIt:Next(exists)}
    }
}

; Call periodically:
timedcommand 50 Script:QueueCommand[call CleanupExpiredBadActors]
```

### Best Practices

1. **Use collections for:**
   - Blacklists/whitelists
   - "Seen" tracking
   - ID-based lookups
   - Cache invalidation lists

2. **Key selection:**
   ```lavishscript
   ; Use unique, stable IDs as keys
   BadActors:Set[${Actor.ID},"..."]        ; Good
   BadActors:Set[${Actor.Name},"..."]      ; Bad (names can duplicate)
   ```

3. **Value storage:**
   ```lavishscript
   ; Store useful context in value
   BadActors:Set[${ActorID},${ActorName}]           ; Name for logging
   CooldownCache:Set[${AbilityName},${ExpiryTime}]  ; Expiry time
   ```

---

## 13. Dynamic File-Based Configuration Discovery

### Overview

The File Discovery pattern dynamically loads configurations from the filesystem at runtime, allowing users to create new configs without modifying code. This pattern is essential for user-extensible systems.

**Source:** EQ2Track script (https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Track)

### Core Pattern

```lavishscript
;===========================================
; Basic file discovery
;===========================================

variable filelist ConfigFiles
variable int Count = 0

; Get all XML files in directory
ConfigFiles:GetFiles[${LavishScript.HomeDirectory}/Scripts/MyScript/Configs/\*.xml]

; Process each file
while ${Count:Inc} <= ${ConfigFiles.Files}
{
    echo "Found config: ${ConfigFiles.File[${Count}].Filename}"

    ; Strip .xml extension
    variable string ConfigName
    ConfigName:Set[${ConfigFiles.File[${Count}].Filename.Left[-4]}]

    echo "Config name: ${ConfigName}"
}
```

### Populating UI Dropdown

```lavishscript
;===========================================
; Populate combobox with discovered configs
;===========================================

method UpdateConfigCombo()
{
    variable filelist ConfigFiles
    variable int Count = 0

    ; Clear existing items
    UIElement[ConfigCombo]:ClearItems

    ; Discover config files
    ConfigFiles:GetFiles[${LavishScript.HomeDirectory}/Scripts/MyScript/Saved Configs/\*.xml]

    while ${Count:Inc} <= ${ConfigFiles.Files}
    {
        ; Get filename without extension
        variable string ConfigName
        ConfigName:Set[${ConfigFiles.File[${Count}].Filename.Left[-4]}]

        ; Add to dropdown
        UIElement[ConfigCombo]:AddItem[${ConfigName}]
    }

    ; Sort alphabetically and select first
    if ${UIElement[ConfigCombo].Items}
    {
        UIElement[ConfigCombo]:Sort:SelectItem[1]
    }
}
```

### Complete Discovery + Load Pattern

```lavishscript
;===========================================
; Full config discovery and loading system
;===========================================

objectdef _ConfigInterface
{
    method UpdateConfigList()
    {
        variable filelist ConfigFiles
        variable int Count = 0

        UIElement[ConfigList]:ClearItems

        ConfigFiles:GetFiles[${LavishScript.HomeDirectory}/Scripts/MyScript/Configs/\*.xml]

        while ${Count:Inc} <= ${ConfigFiles.Files}
        {
            UIElement[ConfigList]:AddItem[${ConfigFiles.File[${Count}].Filename.Left[-4]}]
        }

        UIElement[ConfigList]:Sort
    }

    method LoadConfig()
    {
        variable string ConfigName
        ConfigName:Set[${UIElement[ConfigList].SelectedItem.Text}]

        This:LoadConfigByName[${ConfigName}]
    }

    method LoadConfigByName(string ConfigName)
    {
        variable settingsetref Config

        if ${ConfigName.Length} == 0
            return

        LavishSettings:AddSet[MyConfig]
        Config:Set[${LavishSettings[MyConfig]}]
        Config:Import["${LavishScript.HomeDirectory}/Scripts/MyScript/Configs/${ConfigName}.xml"]

        ; Process settings
        variable iterator SettingIt
        Config:GetSettingIterator[SettingIt]

        if ${SettingIt:First(exists)}
        {
            do
            {
                echo "${SettingIt.Key} = ${SettingIt.Value}"
                ; Apply settings here
            }
            while ${SettingIt:Next(exists)}
        }

        LavishSettings[MyConfig]:Remove
    }

    method SaveConfig()
    {
        variable string ConfigName
        ConfigName:Set[${UIElement[ConfigNameEntry].Text}]

        if ${ConfigName.Length} == 0
        {
            echo "Error: Config name cannot be empty"
            return
        }

        variable settingsetref Config

        LavishSettings:AddSet[MyConfig]
        Config:Set[${LavishSettings[MyConfig]}]

        ; Add settings to save
        Config:AddSetting["Option1",${Option1Value}]
        Config:AddSetting["Option2",${Option2Value}]

        ; Export to file
        Config:Export["${LavishScript.HomeDirectory}/Scripts/MyScript/Configs/${ConfigName}.xml"]

        LavishSettings[MyConfig]:Remove

        ; Refresh UI list
        This:UpdateConfigList
    }

    method DeleteConfig()
    {
        variable string ConfigName
        ConfigName:Set[${UIElement[ConfigList].SelectedItem.Text}]

        if ${ConfigName.Length} == 0
            return

        rm "${LavishScript.HomeDirectory}/Scripts/MyScript/Configs/${ConfigName}.xml"

        This:UpdateConfigList
    }
}
```

### File Extension Stripping

```lavishscript
;===========================================
; Various methods to strip file extension
;===========================================

; Method 1: Left[-4] for .xml
ConfigName:Set[${ConfigFiles.File[${Count}].Filename.Left[-4]}]

; Method 2: Find last dot and use Left
variable int DotPos
DotPos:Set[${Filename.FindLast["."]]}]
if ${DotPos}
    ConfigName:Set[${Filename.Left[${Math.Calc[${DotPos} - 1]}]}]

; Method 3: Token parsing
ConfigName:Set[${Filename.Token[1,"."]}]
```

### Pattern Matching

```lavishscript
;===========================================
; Load only files matching pattern
;===========================================

function LoadClassConfigs()
{
    variable filelist ClassFiles
    variable int Count = 0

    ; Only get files starting with class name
    ClassFiles:GetFiles[${LavishScript.HomeDirectory}/Scripts/MyScript/Classes/${Me.SubClass}_\*.xml]

    echo "Found ${ClassFiles.Files} configs for ${Me.SubClass}"

    while ${Count:Inc} <= ${ClassFiles.Files}
    {
        echo "Loading: ${ClassFiles.File[${Count}].Filename}"
        call LoadConfig["${ClassFiles.File[${Count}].FullPath}"]
    }
}
```

### Recursive Discovery

```lavishscript
;===========================================
; Search subdirectories
;===========================================

function DiscoverAllConfigs()
{
    variable filelist Configs
    variable filelist SubDirs
    variable int i = 0

    ; Get configs in main directory
    Configs:GetFiles[${LavishScript.HomeDirectory}/Scripts/MyScript/Configs/\*.xml]
    echo "Found ${Configs.Files} configs in main directory"

    ; Get subdirectories
    SubDirs:GetDirectories[${LavishScript.HomeDirectory}/Scripts/MyScript/Configs/\*]

    i:Set[0]
    while ${i:Inc} <= ${SubDirs.Files}
    {
        variable filelist SubConfigs
        SubConfigs:GetFiles[${SubDirs.File[${i}].FullPath}/\*.xml]
        echo "Found ${SubConfigs.Files} configs in ${SubDirs.File[${i}].Filename}"
    }
}
```

### Best Practices

1. **Directory structure:**
   ```
   Scripts/
     MyScript/
       MyScript.iss
       Saved Configs/
         Config1.xml
         Config2.xml
       Character Configs/
         ${Me.Name}.xml
   ```

2. **Always use relative paths:**
   ```lavishscript
   ; Good
   ${LavishScript.HomeDirectory}/Scripts/MyScript/Configs/
   ${Script.CurrentDirectory}/Configs/

   ; Bad
   C:\Program Files\InnerSpace\Scripts\MyScript\Configs/
   ```

3. **Handle missing directories:**
   ```lavishscript
   if !${ConfigFiles.Files}
   {
       echo "No configs found. Creating default..."
       call CreateDefaultConfig
   }
   ```

4. **Validate before loading:**
   ```lavishscript
   if ${ConfigName.Length} == 0
       return
   if ${ConfigName.Equal["NULL"]}
       return
   ```

---

## 14. Zone-Aware Auto-Configuration

### Overview

Zone-Aware Auto-Configuration automatically loads zone-specific settings when entering a new zone. This pattern eliminates manual configuration switching and enables seamless multi-zone automation.

**Source:** EQ2Track script (https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Track)

### Basic Pattern

```lavishscript
;===========================================
; Auto-load config matching current zone
;===========================================

variable bool AutoLoadConfigOnZoning = TRUE

atom(script) EQ2_FinishedZoning(string TimeInSeconds)
{
    if !${AutoLoadConfigOnZoning}
        return

    variable filelist ConfigFiles
    variable int Count = 0
    variable string ConfigName

    ConfigFiles:GetFiles[${LavishScript.HomeDirectory}/Scripts/MyScript/Configs/\*.xml]

    while ${Count:Inc} <= ${ConfigFiles.Files}
    {
        ConfigName:Set[${ConfigFiles.File[${Count}].Filename.Left[-4]}]

        ; Check if config name matches zone short name
        if ${Zone.ShortName.Equal[${ConfigName}]}
        {
            echo "Auto-loading config for zone: ${Zone.ShortName}"
            MyInterface:LoadConfigByName[${ConfigName}]
            return
        }
    }

    echo "No zone-specific config found for ${Zone.ShortName}"
}
```

### Initialization with Zone Check

```lavishscript
;===========================================
; Check for zone config on startup
;===========================================

function main()
{
    variable filelist ConfigFiles
    variable int Count = 0
    variable string ConfigName
    variable bool FoundZoneConfig = FALSE

    ; Initialize UI
    ui -reload "${Script.CurrentDirectory}/UI/MyScript.xml"

    ; Search for zone-specific config
    ConfigFiles:GetFiles[${LavishScript.HomeDirectory}/Scripts/MyScript/Configs/\*.xml]

    while ${Count:Inc} <= ${ConfigFiles.Files}
    {
        ConfigName:Set[${ConfigFiles.File[${Count}].Filename.Left[-4]}]

        if ${Zone.ShortName.Equal[${ConfigName}]}
        {
            echo "Loading zone config: ${ConfigName}"
            MyInterface:LoadConfigByName[${ConfigName}]

            ; Select in UI dropdown
            UIElement[ConfigCombo].ItemByText[${ConfigName}]:Select

            FoundZoneConfig:Set[TRUE]
            break
        }
    }

    ; Fallback to last used config
    if !${FoundZoneConfig}
    {
        variable string LastConfig
        LastConfig:Set[${User.FindSetting["LastConfig",""]}]

        if ${LastConfig.Length} > 0
        {
            echo "Loading last used config: ${LastConfig}"
            UIElement[ConfigCombo].ItemByText[${LastConfig}]:Select
        }
    }

    ; Attach zone event
    Event[EQ2_FinishedZoning]:AttachAtom[EQ2_FinishedZoning]

    ; Main loop
    do
    {
        waitframe
    }
    while 1
}
```

### Zone-Specific Resources

```lavishscript
;===========================================
; Create zone-specific saved lists
;===========================================

; Example: User saves tracking filter as "antonica"
; On zoning to Antonica, automatically loads that filter

method SaveListForCurrentZone()
{
    variable string ZoneName
    ZoneName:Set[${Zone.ShortName}]

    ; Save current settings with zone name
    UIElement[ConfigNameEntry]:SetText[${ZoneName}]
    This:SaveConfig

    echo "Saved config for ${Zone.Name} as '${ZoneName}'"
    echo "This config will auto-load when you enter this zone"
}
```

### Priority System

```lavishscript
;===========================================
; Load most specific config available
;===========================================

atom(script) EQ2_FinishedZoning(string TimeInSeconds)
{
    if !${AutoLoadConfigOnZoning}
        return

    ; Priority 1: Zone + Character specific
    variable string ConfigName
    ConfigName:Set["${Zone.ShortName}_${Me.Name}"]
    if ${ConfigExists[${ConfigName}]}
    {
        echo "Loading character-specific zone config"
        MyInterface:LoadConfigByName[${ConfigName}]
        return
    }

    ; Priority 2: Zone + Class specific
    ConfigName:Set["${Zone.ShortName}_${Me.SubClass}"]
    if ${ConfigExists[${ConfigName}]}
    {
        echo "Loading class-specific zone config"
        MyInterface:LoadConfigByName[${ConfigName}]
        return
    }

    ; Priority 3: Zone general
    ConfigName:Set[${Zone.ShortName}]
    if ${ConfigExists[${ConfigName}]}
    {
        echo "Loading general zone config"
        MyInterface:LoadConfigByName[${ConfigName}]
        return
    }

    ; Priority 4: Default config
    echo "No zone-specific config found, using default"
}

function:bool ConfigExists(string ConfigName)
{
    variable filepath ConfigPath
    ConfigPath:Set["${LavishScript.HomeDirectory}/Scripts/MyScript/Configs/${ConfigName}.xml"]

    return ${ConfigPath.FileExists}
}
```

### User-Friendly Save Function

```lavishscript
;===========================================
; Smart save with zone name suggestion
;===========================================

method SaveConfig()
{
    variable string ConfigName
    ConfigName:Set[${UIElement[ConfigNameEntry].Text}]

    ; If user left name blank, suggest zone name
    if ${ConfigName.Length} == 0
    {
        UIElement[ConfigNameEntry]:SetText[${Zone.ShortName}]
        echo "Suggested config name: ${Zone.ShortName}"
        echo "Save again to confirm, or edit name first"
        return
    }

    ; Perform save
    This:SaveConfigByName[${ConfigName}]

    ; Check if it matches zone
    if ${ConfigName.Equal[${Zone.ShortName}]}
    {
        echo "Config saved with zone name - will auto-load in this zone!"
    }
}
```

### XML UI Integration

```xml
<Button Name='SaveForZone'>
    <X>100</X>
    <Y>10</Y>
    <Width>80</Width>
    <Height>20</Height>
    <Text>Save for Zone</Text>
    <AutoTooltip>Save current settings for this zone (auto-loads on zone entry)</AutoTooltip>
    <OnLeftClick>
        UIElement[ConfigNameEntry]:SetText[${Zone.ShortName}]
        MyInterface:SaveConfig
    </OnLeftClick>
</Button>
```

### Advanced: Zone Category Matching

```lavishscript
;===========================================
; Load configs based on zone type
;===========================================

function GetZoneCategory()
{
    ; Categorize zones
    switch ${Zone.ID}
    {
        case 220  ; Antonica
        case 221  ; Commonlands
        case 222  ; Feerott
            return "Overland"

        case 1145 ; Halls of Fate
        case 1146 ; Palace of Ferzhul
            return "Raid"

        case 395  ; Befallen
        case 396  ; Blackburrow
            return "Dungeon"
    }

    ; Default
    return "General"
}

atom(script) EQ2_FinishedZoning(string TimeInSeconds)
{
    variable string Category
    Category:Set[${GetZoneCategory}]

    ; Try zone-specific first
    if ${ConfigExists[${Zone.ShortName}]}
    {
        MyInterface:LoadConfigByName[${Zone.ShortName}]
        return
    }

    ; Fall back to category config
    if ${ConfigExists[${Category}]}
    {
        echo "Loading ${Category} zone config"
        MyInterface:LoadConfigByName[${Category}]
        return
    }
}
```

### Best Practices

1. **Clear naming convention:**
   ```lavishscript
   ; Zone configs use ${Zone.ShortName}
   ; antonica.xml, commonlands.xml, etc.

   ; Character-specific: zonename_charactername.xml
   ; antonica_mychar.xml

   ; Class-specific: zonename_classname.xml
   ; antonica_guardian.xml
   ```

2. **User feedback:**
   ```lavishscript
   echo "Auto-loaded config: ${ConfigName}"
   echo "To create zone-specific config, click 'Save for Zone'"
   ```

3. **Disable option:**
   ```lavishscript
   ; Always provide way to disable auto-load
   variable bool AutoLoadConfigOnZoning = TRUE

   ; Checkbox in UI to toggle
   ```

4. **Event timing:**
   ```lavishscript
   ; Use EQ2_FinishedZoning, not OnZone
   ; Ensures zone data is fully loaded
   Event[EQ2_FinishedZoning]:AttachAtom[EQ2_FinishedZoning]
   ```

---

## Additional Resources

- **EQ2OgreFree Scripts:** https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2OgreCommon and https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2OgreFree
- **EQ2Track Script:** https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Track
- **LavishScript Reference:** http://www.lavishsoft.com/wiki/LavishScript
- **ISXEQ2 API Reference:** [03_API_Reference.md](03_API_Reference.md)
- **Advanced Patterns from EQ2Bot:** [07_Advanced_Patterns_And_Examples.md](07_Advanced_Patterns_And_Examples.md)

---

## Summary

This guide covered 14 production-grade patterns found in professional ISXEQ2 scripts:

### From EQ2Ogre Analysis:

1. ✅ **Multi-Threading** - Worker threads with cross-script communication
2. ✅ **LavishSettings** - Hierarchical XML configuration management
3. ✅ **LavishNav** - Advanced pathfinding and navigation
4. ✅ **Timer Objects** - Reusable timing with pulse architecture
5. ✅ **UI Synchronization** - Safe UI loading and settings sync
6. ✅ **EQ2:GetActors** - Modern actor scanning (replaces CustomActorArray)
7. ✅ **Trigger System** - Automatic chat parsing with callbacks
8. ✅ **Controller Pattern** - Shared resource management with auto-cleanup
9. ✅ **Dynamic Declaration** - Runtime object creation with proper scoping
10. ✅ **Injectable UI** - Modular, plugin-based UI architecture

### From EQ2Track Analysis:

11. ✅ **UI Initialization Guard** - Prevent premature event firing during UI setup
12. ✅ **Collection-Based Exclusion** - Fast O(1) blacklist/whitelist lookups
13. ✅ **File-Based Discovery** - Dynamic configuration loading from filesystem
14. ✅ **Zone-Aware Auto-Config** - Automatic zone-specific settings

These patterns represent the state-of-the-art in ISXEQ2 scripting and are used in complex, production automation systems. Master these to create professional-grade scripts!

---

*Part of ISXEQ2 Scripting Guide*
