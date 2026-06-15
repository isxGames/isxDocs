# Advanced ISXPantheon Scripting Patterns

**Comprehensive guide to production-grade, platform-agnostic scripting techniques**

This guide covers advanced LavishScript patterns and techniques used in complex, multi-threaded automation systems. The infrastructure patterns here (multi-threading, timer objects, LavishSettings, UI synchronization, triggers, dynamic loading) are platform-agnostic and apply to any InnerSpace extension. Patterns that depend on live game data (entity enumeration, zone-aware configuration) are flagged as planned, since that surface is not yet implemented in ISXPantheon.

---

## Table of Contents

1. [Multi-Threaded Script Architecture](#multi-threaded-script-architecture)
2. [LavishSettings Configuration Management](#lavishsettings-configuration-management)
3. [LavishNav Navigation System](#lavishnav-navigation-system)
4. [Reusable Timer Objects](#reusable-timer-objects)
5. [UI Synchronization Patterns](#ui-synchronization-patterns)
6. [Entity Enumeration (planned)](#entity-enumeration-planned)
7. [Trigger System for Chat Parsing](#trigger-system-for-chat-parsing)
8. [Script Controller Pattern](#script-controller-pattern)
9. [Dynamic Variable Declaration](#dynamic-variable-declaration)
10. [Injectable UI Components](#injectable-ui-components)
11. [UI Initialization Guard Pattern](#11-ui-initialization-guard-pattern)
12. [Collection-Based Exclusion Lists](#12-collection-based-exclusion-lists)
13. [Dynamic File-Based Configuration Discovery](#13-dynamic-file-based-configuration-discovery)
14. [Context-Aware Auto-Configuration (planned)](#14-context-aware-auto-configuration-planned)

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
    ; Stop all movement immediately (issue your stop-movement input here)
    press -release ${Forward}
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

### Complete Example: Multi-Threaded Worker Pattern

This example demonstrates the multi-threading architecture itself — a controller thread coordinating two worker threads through global state, with cooperative pause/stop and cross-thread `ExecuteAtom` calls. It uses a generic work queue rather than any game-specific data, so the pattern is fully runnable today. Once game-data surface ships, the "find work" step in the scanner thread can be replaced with a real query.

```lavishscript
;===========================================
; WorkerController.iss - Main Controller
;===========================================
variable(global) bool WorkStop=FALSE
variable(global) bool WorkPause=FALSE
variable(global) int  PendingJob=0
variable(global) string JobCommand=NONE

function main()
{
    ; Start worker threads
    if !${Script[JobScanner](exists)}
        runscript "${LavishScript.HomeDirectory}/Scripts/JobScanner"

    if !${Script[JobProcessor](exists)}
        runscript "${LavishScript.HomeDirectory}/Scripts/JobProcessor"

    ; Wait for both threads to start (up to 5 seconds)
    wait 50 ${Script[JobScanner](exists)} && ${Script[JobProcessor](exists)}

    echo "Worker started - all threads running"

    while !${WorkStop}
    {
        ; Paused check
        while ${WorkPause} && !${WorkStop}
            wait 10

        ; If the scanner queued a job, hand it to the processor
        if ${PendingJob} > 0
        {
            Script[JobProcessor]:ExecuteAtom[BeginJob]
            wait 50 ${JobCommand.Equal[NONE]}
            PendingJob:Set[0]
        }

        wait 10
    }

    ; Shutdown
    WorkStop:Set[TRUE]
    wait 100
    echo "Worker stopped"
}

;===========================================
; JobScanner.iss - Work Detection Thread
;===========================================
function main()
{
    variable int NextJob=0

    while !${WorkStop}
    {
        if !${WorkPause} && ${PendingJob}==0
        {
            ; Replace this block with a real query once game-data surface ships.
            ; For now, generate a synthetic job id to demonstrate the handoff.
            NextJob:Inc
            PendingJob:Set[${NextJob}]
            JobCommand:Set[PROCESS]
        }

        wait 5  ; Scan every half second
    }
}

;===========================================
; JobProcessor.iss - Work Execution Thread
;===========================================
function main()
{
    while !${WorkStop}
    {
        if ${JobCommand.Equal[PROCESS]}
        {
            call ProcessJob ${PendingJob}
            JobCommand:Set[NONE]
        }

        wait 10
    }
}

atom BeginJob()
{
    ; Cross-thread trigger from the controller; the loop above does the work.
    return
}

function ProcessJob(int JobID)
{
    echo "Processing job ${JobID}"
    wait 20

    return
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
LavishSettings[MyApplication].FindSet[Users]:AddSet[${Session}]

; Get reference to this session's settings
UserSettings:Set[${LavishSettings[MyApplication].FindSet[Users].FindSet[${Session}]}]

; Import from file
UserSettings:Import["${Script.CurrentDirectory}/Config/${Session}.xml"]

; Read settings
if ${UserSettings.FindSetting[EnableAutoLoot,TRUE]}
    echo "Auto-loot is enabled"

variable int LootRadius
LootRadius:Set[${UserSettings.FindSetting[LootRadius,50]}]

; Write settings
UserSettings.FindSetting[EnableAutoLoot]:Set[TRUE]
UserSettings.FindSetting[LootRadius]:Set[75]

; Export to file
UserSettings:Export["${Script.CurrentDirectory}/Config/${Session}.xml"]
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
    LavishSettings[BotApplication].FindSet[Characters]:AddSet[${Session}]

    BotConfig:Set[${LavishSettings[BotApplication].FindSet[Characters].FindSet[${Session}]}]

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
    ConfigFile:Set["${Script.CurrentDirectory}/Configs/${Session}.xml"]

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
    ConfigFile:Set["${Script.CurrentDirectory}/Configs/${Session}.xml"]
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

LavishNav is InnerSpace's pre-mapped, region-based pathfinding system. It provides intelligent navigation that handles obstacles, elevation, and complex routes using zone-specific map data.

**Capabilities covered by the navigation library:**

- Dijkstra shortest-path planning over mapped regions (`dijkstrapathfinder`, `lnavpath`, `lnavregionref`, `point3f`)
- Region lookup via `LNavRegion[]`, `LavishNav.FindRegion`, and `BestContainer`
- Zone map loading from XML or LSO binary format
- Collision and steep-terrain detection for direct-movement fallback
- Multi-level stuck detection and external obstacle-recovery scripts
- Smart selection between direct movement and pathfinding based on distance
- Dual-precision management (tight waypoint precision, looser destination precision)
- CPU-saving pulse throttling
- Complete working examples (single-region and multi-waypoint navigation)

For the full guide with patterns, code examples, and production-grade best practices, see [18_Navigation_Library_Patterns.md](18_Navigation_Library_Patterns.md).

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
; Professional multi-timer architecture
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
<ISUI>
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
</ISUI>
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
    ui -reload "${LavishScript.HomeDirectory}/Scripts/MyBot/UI/BotUI.xml"

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
    Config:Export["${Script.CurrentDirectory}/Config/${Session}.xml"]

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
<ISUI>
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
</ISUI>
**/
```

### Best Practices

1. **Always wait for UI ready flag** before accessing elements
2. **Use global variables** for UI element IDs
3. **Signal changes** with a `SettingsChanged` flag
4. **Batch updates** - don't save on every change, use a flag
5. **OnLoad/OnUnload** - handle UI lifecycle properly

---

## Entity Enumeration (planned)

> **PLANNED — NOT YET IMPLEMENTED.** A game-data surface is on the ISXPantheon roadmap but is not available in the current build. The `entity` datatype is registered but has no working members yet, and `${Me}`/`${Radar}` exist only as reserved (commented-out) top-level objects. No member names or query signatures are documented here because none are finalized — do not write production scripts against this surface yet.

### Overview

Once a game-data surface ships, the standard LavishScript pattern for working with a collection of objects is to populate an `index` and walk it with an `iterator`. That `index`/`iterator` mechanic is platform-agnostic and is covered generically in [01_LavishScript_Fundamentals.md](01_LavishScript_Fundamentals.md). The throttling guidance below applies to any periodic scan you build on top of it.

### Performance Considerations (applies once a game-data surface ships)

The throttling pattern below is platform-agnostic and good practice for any periodic scan, including a future entity enumeration call. Never scan every frame; gate scans behind a timer.

```lavishscript
; DON'T scan every frame (BAD - too frequent)
; DO use a timer object to throttle periodic scans
variable TimerObject ScanTimer
ScanTimer:Set[500]  ; Scan every half second

while 1
{
    if !${ScanTimer.TimeLeft}
    {
        ; ... perform the periodic scan here ...
        ScanTimer:Set[500]
    }
    waitframe
}
```

---

## Trigger System for Chat Parsing

### Overview

The trigger system allows automatic parsing of chat messages using pattern matching with named placeholders. When a trigger pattern matches, it calls a function with extracted values as parameters. The trigger mechanic itself (`AddTrigger`, placeholders, callback signatures) is platform-agnostic and works today against any text stream you feed it.

> **NOTE.** The example trigger strings below (harvest, combat, chat, zone/quest) are illustrative of the *technique*. The exact in-game chat-line formats for Pantheon: Rise of the Fallen are not enumerated here — replace the pattern strings with the real lines you observe in-game. Triggers that react to game events depend on game-data surface that is planned but not yet wired (see the Entity Enumeration section).

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
- `@NAME@` - Entity/item name
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

; Called when a "You found a ..." collectible line appears
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

    ; Check if rare (adapt the rarity keywords to the game's own naming)
    if ${ItemName.Find[Rare]} || ${ItemName.Find[Legendary]}
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
    ; NOTE: issuing in-game commands/responses (invite, tell, etc.) requires
    ; game-action surface that is planned but not yet implemented in ISXPantheon.
    if ${Message.Find[invite]}
    {
        echo "Would respond to invite request from ${SenderName}"
    }

    ; Command system
    if ${Message.Find[!status]}
    {
        echo "Would report status to ${SenderName}"
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

; Optional text — register a base pattern plus a longer variant for the same event
AddTrigger ExperienceGain "You gain @NUMBER@ experience"
AddTrigger ExperienceGainBonus "You gain @NUMBER@ experience (@NUMBER@ bonus)"

function ExperienceGain(string Line, int XP)
{
    TotalXP:Set[${Math.Calc[${TotalXP}+${XP}]}]
}

function ExperienceGainBonus(string Line, int XP, int Bonus)
{
    TotalXP:Set[${Math.Calc[${TotalXP}+${XP}+${Bonus}]}]
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

The controller pattern manages a shared resource (such as a cached periodic scan) and automatically cleans up when no scripts are using it. This prevents resource leaks and ensures efficient operation.

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

### Advanced: Shared Scan Controller

This is the same reference-counting controller applied to a periodic scan shared across scripts: each client registers the range it cares about, the controller scans at the widest requested range on a timer, and shuts itself down when the last client unregisters. The scan call itself is a placeholder — wire it to a real entity query once that surface ships (see the Entity Enumeration section).

```lavishscript
;===========================================
; SharedScanController.iss
;===========================================

variable(global) collection:int ScriptsUsingArray  ; ScriptName -> Distance
variable(global) int MaxDistance = 0

function main()
{
    if !${ArrayControllerOb(exists)}
        declarevariable ArrayControllerOb ArrayControllerObject global

    echo "Shared Scan Controller started"

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
            ; Update the shared scan results here (planned entity query goes here)
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

This pattern shares the result of an expensive periodic query across multiple scripts so it is computed once and read by many. The cache contents below are generic placeholders; the two query calls in `Update` are where a real game-data query would go once that surface ships (planned).

```lavishscript
;===========================================
; DataCacheController.iss - Shares expensive queries
;===========================================

variable(global) collection:string ActiveUsers
variable(global) index:string CachedSetA
variable(global) index:string CachedSetB
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
        InventoryScanTimer:Set[2000]  ; Update set A every 2 seconds
        EnemyScanTimer:Set[500]        ; Update set B every half second
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

        ; Update set A (planned: populate from a real game-data query)
        if !${InventoryScanTimer.TimeLeft}
        {
            InventoryScanTimer:Set[2000]
        }

        ; Update set B (planned: populate from a real game-data query)
        if !${EnemyScanTimer.TimeLeft}
        {
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
        ; Access the shared cached sets directly
        echo "Set A entries: ${CachedSetA.Used}"
        echo "Set B entries: ${CachedSetB.Used}"

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

This pattern creates objects conditionally based on a runtime value. Here the value is a user-configured role read from a custom variable, so it does not depend on game-data surface. Once a local-player datatype ships you could switch on the player's actual class instead.

```lavishscript
;===========================================
; Create objects based on a configured role
;===========================================

function InitializeCombatObjects()
{
    ; Always create base combat object
    if !${CombatOb(exists)}
        declarevariable CombatOb CombatObject script

    ; Role-specific objects, driven by a user-set custom variable
    variable string Role = "${ISXPantheon.GetCustomVariable[Role]}"
    switch ${Role}
    {
        case dps
            if !${NukerOb(exists)}
                declarevariable NukerOb NukerObject script
            break

        case tank
            if !${TankOb(exists)}
                declarevariable TankOb TankObject script
            break

        case healer
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
<ISUI>
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
</ISUI>
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
        ; Update status display using the real ${ISXPantheon} surface
        UIElement[TxtVersion]:SetText[${ISXPantheon.Version}]
        UIElement[TxtAPIVersion]:SetText[${ISXPantheon.APIVersion}]
        UIElement[TxtReady]:SetText[${ISXPantheon.IsReady}]
        UIElement[TxtRunTime]:SetText[${Script.RunningTime}]

        wait 100
    }

    echo "Status tab unloading"
}

;===========================================
; StatusTab.xml
;===========================================
/**
<?xml version='1.0' encoding='UTF-8'?>
<ISUI>
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
        <text x='10' y='10'>Version:</text>
        <text name='TxtVersion' x='100' y='10'></text>

        <text x='10' y='35'>API Version:</text>
        <text name='TxtAPIVersion' x='100' y='35'></text>

        <text x='10' y='60'>Ready:</text>
        <text name='TxtReady' x='100' y='60'></text>

        <text x='10' y='85'>Run Time:</text>
        <text name='TxtRunTime' x='100' y='85'></text>
    </panel>
</ISUI>
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
            echo "Loading plugin: ${PluginFiles.File[${i}].Filename}"
            runscript "${PluginDir}/${PluginFiles.File[${i}].Filename}" ${MainTabsID} ${TabPos}
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

### Complete Example: Guarded Combobox Initialization

```lavishscript
;===========================================
; Full guarded-initialization example
;===========================================

variable bool TrackListCombo_executeOnSelect = FALSE

function main()
{
    ; Guard flag starts as FALSE
    TrackListCombo_executeOnSelect:Set[FALSE]

    ; Load UI (OnLoad fires and populates combo)
    ui -reload "${Script.CurrentDirectory}/UI/MyTool.xml"

    ; Perform additional initialization
    variable filelist ListFiles
    variable int Count = 0

    ListFiles:GetFiles[${Script.CurrentDirectory}/Saved Lists/\*.xml]
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


### Why Collections Over Arrays

```lavishscript
; BAD: Array-based exclusion (O(n) lookup)
variable string ExcludedIDs[1000]
variable int ExcludedCount = 0

; Check requires loop through entire array
function IsExcluded(int ID)
{
    variable int i = 1
    while ${i} <= ${ExcludedCount}
    {
        if ${ExcludedIDs[${i}].Equal[${ID}]}
            return TRUE
        i:Inc
    }
    return FALSE
}

; GOOD: Collection-based exclusion (O(1) lookup)
variable collection:string ExcludedIDs

; Instant lookup!
if ${ExcludedIDs.Element[${ID}](exists)}
    return TRUE
```

### Basic Pattern

```lavishscript
;===========================================
; ID exclusion list with a collection
;===========================================

variable collection:string ExcludedIDs

function Exclude(int ID, string Label)
{
    if ${ExcludedIDs.Element[${ID}](exists)}
    {
        echo "Already excluded: ${Label}"
        return
    }

    ; Add to exclusion list
    ExcludedIDs:Set[${ID},${Label}]
    echo "Excluded ${Label} (${ID})"
}

function IsExcluded(int ID)
{
    ; O(1) instant lookup
    return ${ExcludedIDs.Element[${ID}](exists)}
}

function ClearExcludedIDs()
{
    ExcludedIDs:Clear
}
```

### Integration with a Scan Loop

> **PLANNED — NOT YET IMPLEMENTED.** The entity scan call below is provisional; entity enumeration is not yet available (see the Entity Enumeration section). The exclusion-collection technique itself is agnostic — only the source of the IDs being filtered is planned.

```lavishscript
;===========================================
; Skip excluded IDs during a scan
;===========================================

function ScanEntities()
{
    variable index:int Candidates
    variable iterator CandidateIt

    ; PLANNED: populate Candidates from a real entity query
    Candidates:GetIterator[CandidateIt]

    if ${CandidateIt:First(exists)}
    {
        do
        {
            ; Fast exclusion check
            if ${ExcludedIDs.Element[${CandidateIt.Value}](exists)}
            {
                ; Skip this id entirely
                continue
            }

            ; Process candidate
            echo "Processing: ${CandidateIt.Value}"
        }
        while ${CandidateIt:Next(exists)}
    }
}
```

### UI Integration

```lavishscript
;===========================================
; Add/remove an id to the exclusion list via UI
;===========================================

objectdef _ListInterface
{
    method ExcludeID(int ID, string Label)
    {
        if ${ExcludedIDs.Element[${ID}](exists)}
        {
            echo "${Label} already excluded"
            return
        }

        ; Add to exclusion list
        ExcludedIDs:Set[${ID},${Label}]

        ; Remove from tracking list if present
        call RemoveFromTrackingList[${ID}]
    }

    method UnExcludeID(int ID)
    {
        if ${ExcludedIDs.Element[${ID}](exists)}
        {
            ExcludedIDs:Erase[${ID}]
            echo "Removed id ${ID} from exclusion list"
        }
    }
}
```

### XML Button

```xml
<Button Name='ExcludeSelected'>
    <X>134</X>
    <Y>50</Y>
    <Width>51</Width>
    <Height>20</Height>
    <AutoTooltip>Exclude the selected id and no longer track it</AutoTooltip>
    <Text>Exclude</Text>
    <OnLeftClick>
        ListInterface:ExcludeID[${SelectedID},"${SelectedLabel}"]
    </OnLeftClick>
</Button>
```

### Persistence Pattern

```lavishscript
;===========================================
; Save/Load exclusion list
;===========================================

function SaveExcludedIDs()
{
    variable iterator ExcludedIt
    variable settingsetref ExcludeList

    LavishSettings:AddSet[ExcludeList]
    ExcludeList:Set[${LavishSettings[ExcludeList]}]

    ExcludedIDs:GetIterator[ExcludedIt]
    if ${ExcludedIt:First(exists)}
    {
        do
        {
            ; Key = ID, Value = Label
            ExcludeList:AddSetting[${ExcludedIt.Key},${ExcludedIt.Value}]
        }
        while ${ExcludedIt:Next(exists)}
    }

    ExcludeList:Export["${Script.CurrentDirectory}/ExcludedIDs.xml"]
    LavishSettings[ExcludeList]:Remove
}

function LoadExcludedIDs()
{
    variable iterator SettingIt
    variable settingsetref ExcludeList

    LavishSettings:AddSet[ExcludeList]
    ExcludeList:Set[${LavishSettings[ExcludeList]}]
    ExcludeList:Import["${Script.CurrentDirectory}/ExcludedIDs.xml"]

    ExcludeList:GetSettingIterator[SettingIt]
    if ${SettingIt:First(exists)}
    {
        do
        {
            ; Restore to collection
            ExcludedIDs:Set[${SettingIt.Key},${SettingIt.Value}]
        }
        while ${SettingIt:Next(exists)}
    }

    LavishSettings[BadActorList]:Remove
}
```

### Advanced: Temporary Exclusion

```lavishscript
;===========================================
; Time-based temporary exclusion
;===========================================

variable collection:int ExcludedExpiry

function ExcludeTemporary(int ID, int DurationSeconds)
{
    variable int ExpiryTime
    ExpiryTime:Set[${Math.Calc[${Script.RunningTime} + (${DurationSeconds} * 1000)]}]

    ExcludedIDs:Set[${ID},"temp"]
    ExcludedExpiry:Set[${ID},${ExpiryTime}]
}

function CleanupExpiredExclusions()
{
    variable iterator ExpiryIt

    ExcludedExpiry:GetIterator[ExpiryIt]
    if ${ExpiryIt:First(exists)}
    {
        do
        {
            if ${Script.RunningTime} >= ${ExpiryIt.Value}
            {
                ; Expired, remove from both collections
                ExcludedIDs:Erase[${ExpiryIt.Key}]
                ExcludedExpiry:Erase[${ExpiryIt.Key}]
            }
        }
        while ${ExpiryIt:Next(exists)}
    }
}

; Call periodically:
timedcommand 50 Script:QueueCommand[call CleanupExpiredExclusions]
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
   ExcludedIDs:Set[${EntityID},"..."]        ; Good
   ExcludedIDs:Set[${EntityName},"..."]      ; Bad (names can duplicate)
   ```

3. **Value storage:**
   ```lavishscript
   ; Store useful context in value
   ExcludedIDs:Set[${EntityID},${EntityName}]       ; Name for logging
   CooldownCache:Set[${ActionName},${ExpiryTime}]   ; Expiry time
   ```

---

## 13. Dynamic File-Based Configuration Discovery

### Overview

The File Discovery pattern dynamically loads configurations from the filesystem at runtime, allowing users to create new configs without modifying code. This pattern is essential for user-extensible systems.


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

function LoadPrefixedConfigs(string Prefix)
{
    variable filelist MatchFiles
    variable int Count = 0

    ; Only get files starting with the given prefix
    MatchFiles:GetFiles[${LavishScript.HomeDirectory}/Scripts/MyScript/Profiles/${Prefix}_\*.xml]

    echo "Found ${MatchFiles.Files} configs for prefix ${Prefix}"

    while ${Count:Inc} <= ${MatchFiles.Files}
    {
        echo "Loading: ${MatchFiles.File[${Count}].Filename}"
        call LoadConfig["${MatchFiles.File[${Count}].FullPath}"]
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
       Session Configs/
         ${Session}.xml
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

## 14. Context-Aware Auto-Configuration (planned)

> **PLANNED - NOT YET IMPLEMENTED.** Auto-loading a configuration when the player's game context changes depends on game-data surface that is on the ISXPantheon roadmap but is not available in the current build. No member names, top-level objects, or event names are documented here because none are finalized. Do not write production scripts against this surface yet.

### Overview

Context-aware auto-configuration loads context-specific settings automatically when the player's game context changes. The file-discovery and matching mechanics used to implement it are platform-agnostic (see the previous section) and can be studied today; only the trigger - a game event that signals the context change - and the context key it provides are planned. Once a suitable game event and context member are surfaced, the implementation would attach an atom to the event and match discovered config files against the current context key.

## Additional Resources

- **LavishScript Reference:** http://www.lavishsoft.com/wiki/LavishScript
- **ISXPantheon API Reference:** [03_API_Reference.md](03_API_Reference.md)
- **Advanced Patterns and Examples:** [07_Advanced_Patterns_And_Examples.md](07_Advanced_Patterns_And_Examples.md)

---

## Summary

This guide covered 14 production-grade, platform-agnostic LavishScript patterns:

### Infrastructure Patterns:

1. **Multi-Threading** - Worker threads with cross-script communication
2. **LavishSettings** - Hierarchical XML configuration management
3. **LavishNav** - Pathfinding and navigation mechanics
4. **Timer Objects** - Reusable timing with pulse architecture
5. **UI Synchronization** - Safe UI loading and settings sync
6. **Entity Enumeration (planned)** - Filtered nearby-entity queries (roadmap)
7. **Trigger System** - Automatic chat/text parsing with callbacks
8. **Controller Pattern** - Shared resource management with auto-cleanup
9. **Dynamic Declaration** - Runtime object creation with proper scoping
10. **Injectable UI** - Modular, plugin-based UI architecture

### UI / Data Patterns:

11. **UI Initialization Guard** - Prevent premature event firing during UI setup
12. **Collection-Based Exclusion** - Fast O(1) exclusion-list lookups
13. **File-Based Discovery** - Dynamic configuration loading from filesystem
14. **Context-Aware Auto-Config (planned)** - Auto-load context-specific settings (roadmap)

The infrastructure patterns here apply to any InnerSpace extension. Patterns that depend on live game data are flagged as planned and will become usable once that surface ships in ISXPantheon.
