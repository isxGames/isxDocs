# ISXPantheon Advanced Patterns and Examples

Collection of advanced, platform-generic LavishScript patterns useful for building ISXPantheon scripts. The mechanics below (mouse automation, LavishSettings XML, command-line parsing, triggers, extension management, logging, TTS) are part of LavishScript / InnerSpace and work today regardless of which game-data objects ISXPantheon has surfaced.

> **NOTE — surface maturity.** The live ISXPantheon surface today is `${ISXPantheon}` (version / `IsReady` / custom variables / currency / rounding) plus the `GetURL` / `PostURL` commands. Patterns here that would otherwise reference a game object (player name, server name, broker/merchant windows, raid roster, in-game navigation) are re-based onto generic placeholders or marked **(planned — not yet implemented)**. Do not treat planned shapes as runnable.

---

## Table of Contents

1. [Mouse Automation](#mouse-automation)
2. [LavishSettings XML Configuration](#lavishsettings-xml-configuration)
3. [Command-Line Argument Parsing](#command-line-argument-parsing)
4. [Trigger-Based Automation](#trigger-based-automation)
5. [Game-Data Automation (Planned)](#game-data-automation-planned)
6. [Localization Support](#localization-support)
7. [Navigation System Integration](#navigation-system-integration)
8. [Script Include Patterns](#script-include-patterns)
9. [Debugging and Logging](#debugging-and-logging)
10. [Text-to-Speech Integration](#text-to-speech-integration)
11. [Extension Management](#extension-management)

---

## Mouse Automation

### Pattern: Capturing and Clicking UI Elements

Scripts often need to interact with UI elements not directly accessible via the extension API. Mouse automation allows clicking specific screen coordinates.

#### Basic Mouse Clicking

```lavishscript
variable int ToClickX
variable int ToClickY

function CaptureMousePosition()
{
    ; Get current mouse position
    ToClickX:Set[${Mouse.X}]
    ToClickY:Set[${Mouse.Y}]

    echo "Mouse position captured: ${ToClickX}, ${ToClickY}"
}

function ClickStoredPosition()
{
    ; Move mouse to stored position
    Mouse:SetPosition[${ToClickX},${ToClickY}]
    wait 20  ; Small delay for UI to respond

    ; Click
    Mouse:LeftClick

    echo "Clicked at position: ${ToClickX}, ${ToClickY}"
}
```

#### Advanced: Saving Mouse Position to Configuration

```lavishscript
variable(script) int ToClickX
variable(script) int ToClickY
variable(script) settingsetref Options

function ResetMouseClickLocation()
{
    variable int i = 1

    ; Verify mouse is over the target UI element
    if (!${UIElement[TargetElement@MyWindow].IsMouseOver})
    {
        echo "Please hover mouse over the target UI element."
        return "ENDSCRIPT"
    }

    ; Countdown before capturing
    echo "Capturing mouse position in 5 seconds..."
    do
    {
        wait 10
        echo "${i}..."
    }
    while ${i:Inc} <= 5

    ; Capture position
    ToClickX:Set[${Mouse.X}]
    ToClickY:Set[${Mouse.Y}]

    ; Save to configuration
    Options:AddSetting[ToClickX,${ToClickX}]
    Options:AddSetting[ToClickY,${ToClickY}]
    LavishSettings[Config File]:Export[${configfile}]

    echo "'Add' button set at ${ToClickX}, ${ToClickY}"
    return "OK"
}

function UseStoredMouseClick()
{
    ; Load from configuration
    ToClickX:Set[${Options.FindSetting[ToClickX,0]}]
    ToClickY:Set[${Options.FindSetting[ToClickY,0]}]

    if (${ToClickX} == 0 && ${ToClickY} == 0)
    {
        call ResetMouseClickLocation
        return
    }

    ; Store current mouse position to restore later
    variable int SavedMouseX = ${Mouse.X}
    variable int SavedMouseY = ${Mouse.Y}

    ; Click the target position
    Mouse:SetPosition[${ToClickX},${ToClickY}]

    ; Wait until mouse is over target
    if (!${UIElement[TargetElement@MyWindow].IsMouseOver})
    {
        do
        {
            waitframe
        }
        while !${UIElement[TargetElement@MyWindow].IsMouseOver}
    }

    wait 5
    Mouse:LeftClick
    wait 5

    ; Restore original mouse position
    Mouse:SetPosition[${SavedMouseX},${SavedMouseY}]

    ; Wait until mouse is no longer over target
    if (${UIElement[TargetElement@MyWindow].IsMouseOver})
    {
        do
        {
            waitframe
        }
        while ${UIElement[TargetElement@MyWindow].IsMouseOver}
    }
}
```

#### Key Lessons:

- **Always add delays** after mouse movements (`wait 20` or `waitframe`)
- **Verify mouse position** using `UIElement.IsMouseOver` before/after clicking
- **Store and restore** original mouse position to avoid disrupting user
- **Save to config** for reusable automation across sessions
- **Provide setup mode** for users to capture UI positions

---

## LavishSettings XML Configuration

### Pattern: Persistent Configuration Management

LavishSettings provides XML-based configuration storage for persistent data across script runs.

#### Basic XML Import/Export

```lavishscript
variable settingsetref Configuration
variable filepath ConfigPath
variable string configfile

function InitializeConfiguration()
{
    ; Set config file path (use a per-profile name; substitute your own identifier)
    ConfigPath:Set["${LavishScript.HomeDirectory}/Scripts/MyScript/"]
    configfile:Set[${ConfigPath}Profile_Config.xml]

    ; Create settings structure
    LavishSettings:AddSet[MyScript Config]
    LavishSettings[MyScript Config]:Clear
    LavishSettings[MyScript Config]:AddSet[Options]
    LavishSettings[MyScript Config]:AddSet[Characters]

    ; Import existing configuration
    LavishSettings[MyScript Config]:Import[${configfile}]

    ; Get reference to settings
    Configuration:Set[${LavishSettings[MyScript Config].FindSet[Options]}]
}

function SaveConfiguration()
{
    ; Export to XML file
    LavishSettings[MyScript Config]:Export[${configfile}]
    echo "Configuration saved to ${configfile}"
}
```

#### Advanced: Nested Settings with Collections

```lavishscript
variable settingsetref ProfileData
variable settingsetref KnownItemsList
variable string profileconfigfile
variable string itemcachefile

function LoadProfileData()
{
    ; Profile-specific data tracking (substitute your own profile identifier)
    profileconfigfile:Set["${ConfigPath}Profile_Data.xml"]

    LavishSettings:AddSet[Config File]
    LavishSettings[Config File]:Import[${profileconfigfile}]

    ProfileData:Set[${LavishSettings[Config File].FindSet[Profile]}]

    ; Shared cache (reused across profiles)
    itemcachefile:Set["${ConfigPath}KnownItems_Cache.xml"]

    LavishSettings:AddSet[KnownItems File]
    LavishSettings[KnownItems File]:Import[${itemcachefile}]

    KnownItemsList:Set[${LavishSettings[KnownItems File].FindSet[KnownItems]}]
}

function CacheItemStatus(string ItemName, bool Flag)
{
    ; Add to cache for faster future lookups
    KnownItemsList:AddSetting["${ItemName}",${Flag}]

    ; Export to cache file
    LavishSettings[KnownItems File]:Export[${itemcachefile}]
}

function CheckCachedItem(string ItemName)
{
    ; Check if we've already recorded this item
    variable bool CachedValue

    CachedValue:Set[${KnownItemsList.FindSetting["${ItemName}",FALSE]}]

    if ${CachedValue}
        return TRUE
    else
        return FALSE
}
```

#### Settings Iteration Pattern

```lavishscript
variable settingsetref pAliases
variable iterator AIterator
variable string AliasName
variable string CanonicalName

function LoadAliasMappings()
{
    pAliases:Set[${LavishSettings[MyScript].FindSet[Aliases]}]

    pAliases:GetSetIterator[AIterator]
    if ${AIterator:First(exists)}
    {
        do
        {
            AliasName:Set[${AIterator.Key}]
            CanonicalName:Set[${pAliases.FindSet[${AliasName}].FindSetting[Canonical]}]

            echo "${AliasName} maps to ${CanonicalName}"
        }
        while ${AIterator:Next(exists)}
    }
}

function SaveAliasMapping(string Alias, string Canonical)
{
    pAliases:AddSet[${Alias}]
    pAliases.FindSet[${Alias}]:AddSetting[Canonical,${Canonical}]

    LavishSettings[MyScript]:Export[${configfile}]
}
```

#### Key Lessons:

- **Per-profile configs**: Use a stable profile identifier in the filename
- **Shared caches**: Separate caches for data shared across profiles
- **Clear before import**: Use `:Clear` to reset settings before importing
- **Nested structures**: Create hierarchical settings with `:AddSet`
- **Iterator pattern**: Use `:GetSetIterator` to walk through all settings
- **FindSetting with default**: Provide default value if setting doesn't exist

---

## Command-Line Argument Parsing

### Pattern: Flexible Script Parameters

Scripts often need to accept various command-line arguments for flexibility.

#### Basic Argument Parsing

```lavishscript
function main(... Args)
{
    variable int Iterator = 1
    variable bool DoExtra = FALSE
    variable bool VerboseMode = FALSE
    variable string TargetName

    ; Check if any arguments provided
    if ${Args.Size} > 0
    {
        do
        {
            ; Check for flags
            if (${Args[${Iterator}].Equal[-EXTRA]})
                DoExtra:Set[TRUE]
            elseif (${Args[${Iterator}].Equal[-VERBOSE]})
                VerboseMode:Set[TRUE]
            elseif (${Args[${Iterator}].Equal[-v]})
                VerboseMode:Set[TRUE]
            else
                ; Assume it's the target name
                TargetName:Set[${Args[${Iterator}]}]
        }
        while ${Iterator:Inc} <= ${Args.Size}
    }

    echo "DoExtra: ${DoExtra}"
    echo "VerboseMode: ${VerboseMode}"
    echo "TargetName: ${TargetName}"
}
```

#### Advanced: Named Parameters with Values

```lavishscript
function main(... Args)
{
    variable float MinValue = 0
    variable float MaxValue = 0
    variable int Budget = 0
    variable int TotalToProcess = 0

    if ${Args.Size} > 0
    {
        ; Special command: reset stored mouse position
        if (${Args[1].Equal[ResetMouseClickLocation]})
        {
            call ResetMouseClickLocation
            if (${Return.Equal[ENDSCRIPT]})
                return
        }
        else
        {
            ; Positional parameters
            MinValue:Set[${Args[1]}]
            MaxValue:Set[${Args[2]}]
            Budget:Set[${Args[3]}]
            TotalToProcess:Set[${Args[4]}]
        }
    }
    else
    {
        ; No arguments - display usage
        echo "Syntax: run MyScript MinValue MaxValue Budget TotalToProcess"
        echo "        run MyScript ResetMouseClickLocation"
        echo "---"
        echo "Notes:  1.  'Budget' is an integer"
        echo "        2.  'MinValue' and 'MaxValue' are float values"
        return
    }
}
```

#### Advanced: Token-Based Named Parameters

```lavishscript
function main(... Args)
{
    variable int count = 0
    variable string JobName
    variable string JobDate
    variable bool StatsOnly = FALSE
    variable bool SummaryOnly = FALSE
    variable index:string Excluded

    ; Parse all arguments
    while ${count:Inc}<=${Args.Size}
    {
        ; Named parameter: Job=<name>
        if ${Args[${count}].Token[1,=].Equal[Job]}
            JobName:Set[${Args[${count}].Token[2,=]}]

        ; Named parameter: Date=<date>
        elseif ${Args[${count}].Token[1,=].Equal[Date]}
            JobDate:Set[${Args[${count}].Token[2,=]}]

        ; Flag parameter
        elseif ${Args[${count}].Find[-StatsOnly]}
            StatsOnly:Set[TRUE]

        elseif (${Args[${count}].Find[-SummaryOnly]} || ${Args[${count}].Find[-SumOnly]})
            SummaryOnly:Set[TRUE]

        ; Alias mapping: Alias=Canonical
        elseif ${Args[${count}].Find[=]}
        {
            echo "Adding '${Args[${count}].Token[1,=]}' as alias of '${Args[${count}].Token[2,=]}'"
            call SaveAliasMapping "${Args[${count}].Token[1,=]}" "${Args[${count}].Token[2,=]}"
            return
        }

        ; Anything else is treated as an excluded name
        else
        {
            Excluded:Insert[${Args[${count}]}]
            echo "${Args[${count}]} is excluded"
        }
    }
}
```

#### Switch-Based Flag Parser

A common pattern is a `switch` statement over a variable-argument list (`... Tokens`) to parse flags such as `-vv`, `-debug`, `-sort`, `-v`, `-q`, `-start`, `-load`, and `-hideui`, while the `default` case accumulates any non-flag tokens into a job-name string. This is a fully generic LavishScript pattern.

#### Key Lessons:

- **Use `... Args` pattern** for variable argument count
- **`Args.Size`** gives number of arguments
- **Token parsing**: Use `.Token[index,delimiter]` for `Key=Value` parameters
- **Switch statement**: Clean way to handle multiple flags
- **Default clause**: Capture unrecognized arguments for flexible usage
- **Usage display**: Show syntax when no args or invalid args provided
- **Case-insensitive checks**: Use `.Equal[...]` for string comparison

---

## Trigger-Based Automation

### Pattern: Autoharvest with Wildcard Triggers

The trigger system parses incoming chat text and queues handler function calls. For the canonical reference covering placeholders (`@TYPE@`, `@NUMBER@`, `@RAW@`, `@NAME@`, `@ZONE@`), handler signatures, and full examples for harvest/combat/group/tell/zone/quest triggers, see [15_Advanced_Scripting_Patterns.md](15_Advanced_Scripting_Patterns.md#trigger-system-for-chat-parsing).

The example below is unique in two ways: it uses the `@*@` wildcard form (match any text before/after the literal) and it demonstrates a timer-gated `ExecuteQueued` pump so trigger-driven actions are processed at a controlled interval rather than every frame.

**Source:** autoharvest.iss

```lavishscript
variable(script) int triggerCheckTimer = 5
variable time triggerTime

function init()
{
    ; Register triggers for harvest chat messages
    AddTrigger Harvest "@*@You mined@*@"
    AddTrigger Harvest "@*@You mine@*@"
    AddTrigger Harvest "@*@You fail to mine@*@"
    AddTrigger Harvest "@*@You acquire@*@"
    AddTrigger Harvest "@*@You gather@*@"
    AddTrigger Harvest "@*@You failed to gather@*@"
    AddTrigger Harvest "@*@You fish@*@"
    AddTrigger Harvest "@*@You catch@*@"

    echo "Harvest triggers registered"
}

function main()
{
    call init

    do
    {
        call Check_Triggers
        waitframe
    }
    while 1
}

; Handler function called when a trigger matches.
; NOTE: the trigger pipeline (AddTrigger / ExecuteQueued / QueuedCommands)
; is a real platform feature and works today. The game-data action inside
; the handler is PLANNED - NOT YET IMPLEMENTED (entity/target/combat objects
; are not surfaced yet), so it is shown as a placeholder.
function Harvest(string Line)
{
    echo "Harvest trigger fired: ${Line}"

    ; PLANNED: act on the matched line once entity/target objects exist,
    ; e.g. double-click the targeted resource when out of combat.
}

; Check queued commands from triggers
function Check_Triggers()
{
    ; Only check every triggerCheckTimer seconds
    if ${Math.Calc[${Time.Timestamp}-${triggerTime.Timestamp}]}>=${triggerCheckTimer}
    {
        if ${QueuedCommands}
        {
            do
            {
                ExecuteQueued
            }
            while ${QueuedCommands}
        }

        triggerTime:Set[${Time.Timestamp}]
    }
}
```

#### Key Lessons:

- **Wildcards**: `@*@` matches any text before/after a literal fragment -- useful when you only care that a message occurred, not its details
- **Periodic checking**: Don't drain the trigger queue every frame -- gate `ExecuteQueued` behind a timer to batch work
- **ExecuteQueued loop**: Wrap in `do { ExecuteQueued } while ${QueuedCommands}` to drain all pending matches in one pass

---

## Game-Data Automation (Planned)

> **PLANNED — NOT YET IMPLEMENTED.** The automation patterns that previously lived here — bank/coin management, broker search-and-buy, examine-window text parsing, collection journaling, and raid-attendance tracking — all depend on game-data objects (player currency, target/entity objects, merchant/broker/examine windows, raid roster) that are **not surfaced in the current build**. They have been removed rather than carried as non-runnable code. When the corresponding objects are surfaced, these workflows will return as runnable patterns.

The platform mechanics they relied on are real and usable today, documented elsewhere in this guide:

- **Mouse automation** to drive UI elements — see [Mouse Automation](#mouse-automation) above.
- **Persistent configuration** via LavishSettings XML — see [LavishSettings XML Configuration](#lavishsettings-xml-configuration) above.
- **Collections / iterators** for name-to-count bookkeeping (attendance-style tallies) — these work today against any data source; only the game-roster input is planned.
- **`wait <frames> <condition>`** to wait up to a bounded time for an async condition — fully usable now.

When you build a game-data workflow later, combine those mechanics with the (planned) entity/window objects from the API Reference's Planned API section.

## Localization Support

### Pattern: Multi-Language Script Support

Driving a script's user-facing strings from a configurable locale key, so the same script can run in multiple languages.

#### Localization Pattern

```lavishscript
variable string sLocalization
variable string Label_Start
variable string Label_Stop
variable string Label_Ready

function SetLocalization(string Locale)
{
    switch ${Locale}
    {
        case French
            sLocalization:Set["French"]
            Label_Start:Set["Démarrer"]
            Label_Stop:Set["Arrêter"]
            Label_Ready:Set["Prêt"]
            break

        default
            ; English (default)
            sLocalization:Set["English"]
            Label_Start:Set["Start"]
            Label_Stop:Set["Stop"]
            Label_Ready:Set["Ready"]
            break
    }
}

function main()
{
    ; Pick the locale from a custom variable (or any config source)
    call SetLocalization "${ISXPantheon.GetCustomVariable[Locale]}"

    ; Use the localized strings
    echo "${Label_Ready}"
}
```

#### Key Lessons:

- **Config-driven locale**: Read the active locale from a custom variable or config file
- **Switch by locale**: Different string tables for different languages
- **String variables**: Store all localizable text in variables
- **Default clause**: Fallback to English if the locale is not recognized
- **Non-ASCII strings**: LavishScript string literals can hold accented text directly

---

## Navigation System Integration

### Pattern: Region-Based Pathfinding

> **PLANNED — NOT YET IMPLEMENTED.** Automated movement and pathfinding depend on the player position/movement objects and a Pantheon navigation library, neither of which is surfaced in the current build. The region-load / validate-regions / move-to-region shape below is illustrative only and is not runnable today.

The underlying LavishNav region/path primitives are part of InnerSpace and are platform-generic; the missing piece is the game-side movement surface that drives the player toward a path. When that is available, a navigation flow will follow roughly this shape:

```lavishscript
; PLANNED - illustrative only, not runnable today
variable Nav Nav

function main()
{
    ; Initialize navigation and load the current-area map
    Nav:LoadMap

    ; Move toward a named region, pulsing until movement completes
    Nav:MoveToRegion["DestinationA"]

    do
    {
        Nav:Pulse
        wait 0
    }
    while ${ISXPantheon(exists)} && ${Nav.Moving}

    echo "Arrived at DestinationA"
}
```

#### Key Lessons:

- **LavishNav is platform**: The region/path graph primitives exist in InnerSpace today
- **Movement surface is planned**: Driving the player along a path needs the (planned) player/movement objects
- **Pulse loop**: A navigation library is pulsed each frame while moving
- **Bounded by extension presence**: Guard the loop with `${ISXPantheon(exists)}` so it stops if the extension unloads

---

## Script Include Patterns

### Pattern: Modular Script Organization

Organizing scripts with includes for code reuse.

#### Include Patterns

```lavishscript
; Standard includes
#include "${LavishScript.HomeDirectory}/Scripts/MyBot/Lib/Navigation.iss"
#include "${LavishScript.HomeDirectory}/Scripts/MyBot/Include/Search.iss"

; Optional includes (don't fail if missing)
#includeoptional "${LavishScript.HomeDirectory}/Scripts/Common/Debug.iss"
#includeoptional "${LavishScript.HomeDirectory}/Scripts/Common/MovementKeys.iss"

function main()
{
    ; Use included functionality
    Debug:Echo["Script starting..."]
}
```

#### Key Lessons:

- **#include**: Includes another script file
- **#includeoptional**: Include if exists, don't error if missing
- **LavishScript.HomeDirectory**: Path to InnerSpace installation
- **Relative paths**: Include paths relative to Scripts folder
- **Shared libraries**: Common code in separate include files

---

## Debugging and Logging

### Pattern: Advanced Debug Output

Professional debugging and logging patterns.

#### Debug Pattern

```lavishscript
variable bool V1 = FALSE   ; Verbose mode
variable bool V2 = FALSE   ; Very verbose mode
variable bool EnableDebug = FALSE
variable bool EchoToConsole = FALSE

; Custom echo helper
atom(script) ChatEcho(... Params)
{
    if !${Params.Size}
        return

    if ${EchoToConsole} && ${V1}
    {
        echo ${Params.Expand}
    }
    elseif ${V1}
    {
        echo ${Params.Expand}
    }
}

; Error echo (always shows unless -q quiet mode)
atom(script) ErrorEcho(... Params)
{
    if !${Params.Size}
        return

    if !${Quiet}
    {
        ChatEcho ${Params.Expand}
    }
}

function main()
{
    ; Disable debugging initially
    Script:DisableDebugging

    ; Set debug file location
    Debug:SetFilename["${LavishScript.HomeDirectory}/Scripts/MyBot/MyBot_Debug.txt"]

    if ${EnableDebug}
    {
        Debug:Enable
    }

    ; Enable debug logging
    Debug:SetPrefix[]
    Debug:SetEchoAlsoLogs[TRUE]

    ; Use debug output
    Debug:Echo["Script initializing..."]
    ChatEcho "Verbose message"
    ErrorEcho "Error occurred!"
}
```

#### Log-File Debug Pattern

```lavishscript
#include Common/Debug.iss

variable(script) string sFileName

function main()
{
    ; Initialize debug system
    Debug:Enable
    Debug:SetPrefix[]
    Debug:SetEchoAlsoLogs[TRUE]
    Debug:SetFilename["MyBot.txt"]

    ; Headers
    Debug:Echo["-------- MyBot Run --------"]
    Debug:Echo["(Created ${Time.Date} at ${Time})"]
    Debug:Echo["-"]

    ; Debug output
    Debug:Echo["- Initializing..."]

    ; Log at end
    Debug:Echo["---------------------------"]
    Debug:Log["\n\n\n"]
}
```

#### Key Lessons:

- **atom(script) pattern**: Create reusable echo functions
- **... Params**: Variable parameter list
- **Params.Size**: Number of parameters
- **Params.Expand**: Expand all parameters into string
- **echo**: Echoes to the console
- **Debug:SetFilename**: Set log file location
- **Debug:SetEchoAlsoLogs[TRUE]**: Write to both console and log
- **Debug:SetPrefix[]**: Set empty prefix
- **Time.Date and Time**: Get current date/time
- **Verbosity levels**: V1 for verbose, V2 for very verbose

---

## Text-to-Speech Integration

### Pattern: Audio Feedback for Long-Running Scripts

Provide audio notifications using text-to-speech.

#### TTS Pattern

```lavishscript
variable bool EnableTTS = FALSE

atom(script) ChatSay(... Params)
{
    if !${Params.Used} || !${EnableTTS}
        return

    Debug:Echo["ChatSay '${Params.Expand}'"]

    if ${TTS.IsReady}
        speak "${Params.Expand}"
}

function main()
{
    ; Enable TTS from configuration
    EnableTTS:Set[${Configuration.FindSetting[EnableTTS,FALSE]}]

    ; Use TTS
    ChatSay "Task complete"
    ChatSay "Out of resources"
    ChatSay "Script finished"
}
```

#### Key Lessons:

- **TTS.IsReady**: Check if text-to-speech is available
- **speak command**: Triggers text-to-speech
- **EnableTTS flag**: User configurable on/off
- **Audio notifications**: Useful for AFK or long-running tasks

---

## Extension Management

### Pattern: Loading and Checking Extensions

Managing ISXPantheon and other extensions.

#### Extension Loading Pattern

```lavishscript
function main()
{
    ; Try to load ISXPantheon if not already loaded
    if !${Extension[ISXPantheon](exists)}
        ext isxpantheon

    ; Check if ISXPantheon is loaded
    if ${Extension[ISXPantheon](exists)}
    {
        echo "Waiting for ISXPantheon to get ready..."
        wait 300 ${ISXPantheon.IsReady}
        echo "ISXPantheon should now be Ready!"
        wait 40
    }
    else
    {
        echo "No ISXPantheon extension loaded!"
        wait 70
    }

    ; Use ISXPantheon if available
    if ${Extension[ISXPantheon](exists)}
    {
        echo "Using ISXPantheon features..."
        do
        {
            waitframe
        }
        while !${ISXPantheon.IsReady} && ${Extension[ISXPantheon](exists)}
    }
    else
    {
        echo "Using Lavish-only features..."
    }
}
```

#### ISXPantheon Readiness Pattern

```lavishscript
function main()
{
    if !${ISXPantheon.IsReady}
    {
        ErrorEcho "ISXPantheon has not been loaded! Ending script..."
        Script:End
    }

    ; Use a custom variable as cross-script state
    ISXPantheon:SetCustomVariable[Started,1]

    echo "ISXPantheon ${ISXPantheon.Version} ready"
}
```

#### Key Lessons:

- **Extension[name](exists)**: Check if extension loaded
- **ext extensionname**: Load an extension
- **ISXPantheon.IsReady**: Wait for ISXPantheon to fully initialize
- **wait with timeout**: `wait 300 ${condition}` = wait up to 30 seconds
- **Graceful degradation**: Provide fallback when extension unavailable
- **Custom variables**: Use `:SetCustomVariable` / `.GetCustomVariable` for cross-script state

---

<!-- CLAUDE_SKIP_START -->
## Summary of New Patterns

### Patterns Added from Additional Script Analysis:

1. **Mouse Automation** - UI clicking, position storage, mouse restoration
2. **LavishSettings XML** - Configuration persistence, nested settings, iteration
3. **Command-Line Parsing** - Flag parsing, named parameters, token-based args
4. **Trigger-Based Automation** - Chat pattern matching, queued commands
5. **Game-Data Automation (Planned)** - bank/broker/examine/collection/raid workflows, pending game surface
6. **Localization** - Config-driven multi-language support
7. **Region-Based Pathfinding (Planned)** - LavishNav region graph, pending movement surface
8. **Script Includes** - Modular organization, optional includes
9. **Advanced Debugging** - Multi-level verbosity, file logging
10. **Text-to-Speech** - Audio notifications
11. **Extension Management** - Loading, checking, graceful degradation
<!-- CLAUDE_SKIP_END -->

---

## Cross-References

- **Core Concepts:** [04_Core_Concepts.md](04_Core_Concepts.md)
- **Best Practices:** [05_Patterns_And_Best_Practices.md](05_Patterns_And_Best_Practices.md)
- **Working Examples:** [06_Working_Examples.md](06_Working_Examples.md)
- **API Reference:** [03_API_Reference.md](03_API_Reference.md)
