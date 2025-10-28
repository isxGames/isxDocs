# ISXEQ2 Advanced Patterns and Examples

Complete collection of advanced patterns discovered from production ISXEQ2 scripts including [EQ2Craft](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Craft), [EQ2RaidAttendance](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2RaidAttendance), and 40+ example scripts.

---

## Table of Contents

1. [Mouse Automation](#mouse-automation)
2. [LavishSettings XML Configuration](#lavishsettings-xml-configuration)
3. [Command-Line Argument Parsing](#command-line-argument-parsing)
4. [Trigger-Based Automation](#trigger-based-automation)
5. [Banking Automation](#banking-automation)
6. [Broker Window Automation](#broker-window-automation)
7. [Collection Processing](#collection-processing)
8. [Raid Attendance Tracking](#raid-attendance-tracking)
9. [Examine Window Interaction](#examine-window-interaction)
10. [Localization Support](#localization-support)
11. [Navigation System Integration](#navigation-system-integration)
12. [Script Include Patterns](#script-include-patterns)
13. [Debugging and Logging](#debugging-and-logging)
14. [Text-to-Speech Integration](#text-to-speech-integration)
15. [Extension Management](#extension-management)

---

## Mouse Automation

### Pattern: Capturing and Clicking UI Elements

Scripts often need to interact with UI elements not directly accessible via ISXEQ2. Mouse automation allows clicking specific screen coordinates.

#### Basic Mouse Clicking

**Source:** collect.iss example script

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

**Source:** Production script patterns

```lavishscript
variable(script) int ToClickX
variable(script) int ToClickY
variable(script) settingsetref Options

function ResetMouseClickLocation()
{
    variable int i = 1

    ; Verify mouse is over the target UI element
    if (!${EQ2UIPage[Journals,JournalsQuest].IsMouseOver})
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
    variable int BrokerWindowMouseX = ${Mouse.X}
    variable int BrokerWindowMouseY = ${Mouse.Y}

    ; Click the target position
    Mouse:SetPosition[${ToClickX},${ToClickY}]

    ; Wait until mouse is over target
    if (!${EQ2UIPage[Journals,JournalsQuest].IsMouseOver})
    {
        do
        {
            waitframe
        }
        while !${EQ2UIPage[Journals,JournalsQuest].IsMouseOver}
    }

    wait 5
    Mouse:LeftClick
    wait 5

    ; Restore original mouse position
    Mouse:SetPosition[${BrokerWindowMouseX},${BrokerWindowMouseY}]

    ; Wait until mouse is no longer over target
    if (${EQ2UIPage[Journals,JournalsQuest].IsMouseOver})
    {
        do
        {
            waitframe
        }
        while ${EQ2UIPage[Journals,JournalsQuest].IsMouseOver}
    }
}
```

#### Key Lessons:

- **Always add delays** after mouse movements (`wait 20` or `waitframe`)
- **Verify mouse position** using `EQ2UIPage.IsMouseOver` before/after clicking
- **Store and restore** original mouse position to avoid disrupting user
- **Save to config** for reusable automation across sessions
- **Provide setup mode** for users to capture UI positions

---

## LavishSettings XML Configuration

### Pattern: Persistent Configuration Management

LavishSettings provides XML-based configuration storage for persistent data across script runs.

#### Basic XML Import/Export

**Source:** [EQ2RaidAttendance](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2RaidAttendance) and production scripts

```lavishscript
variable settingsetref Configuration
variable filepath ConfigPath
variable string configfile

function InitializeConfiguration()
{
    ; Set config file path
    ConfigPath:Set["${LavishScript.HomeDirectory}/Scripts/MyScript/"]
    configfile:Set[${ConfigPath}${Me.Name}-${EQ2.ServerName}_Config.xml]

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

**Source:** Production script patterns

```lavishscript
variable settingsetref Collections
variable settingsetref IsCollectibleList
variable string playerconfigfile
variable string collectiblecachefile

function LoadCollectionData()
{
    ; Player-specific collection tracking
    playerconfigfile:Set["${ConfigPath}${Me.Name}-${EQ2.ServerName}_Collections.xml"]

    LavishSettings:AddSet[Config File]
    LavishSettings[Config File]:Import[${playerconfigfile}]

    Collections:Set[${LavishSettings[Config File].FindSet[${Me.Name}]}]

    ; Collectible cache (shared across characters)
    collectiblecachefile:Set["${ConfigPath}IsCollectible_Cache.xml"]

    LavishSettings:AddSet[IsCollectible File]
    LavishSettings[IsCollectible File]:Import[${collectiblecachefile}]

    IsCollectibleList:Set[${LavishSettings[IsCollectible File].FindSet[IsCollectible]}]
}

function CacheCollectibleStatus(string ItemName, bool IsCollectible)
{
    ; Add to cache for faster future lookups
    IsCollectibleList:AddSetting["${ItemName}",${IsCollectible}]

    ; Export to cache file
    LavishSettings[IsCollectible File]:Export[${collectiblecachefile}]
}

function CheckCachedCollectible(string ItemName)
{
    ; Check if we've already determined collectible status
    variable bool CachedValue

    CachedValue:Set[${IsCollectibleList.FindSetting["${ItemName}",FALSE]}]

    if ${CachedValue}
        return TRUE
    else
        return FALSE
}
```

#### Settings Iteration Pattern

**Source:** EQ2RaidAttendance.iss

```lavishscript
variable settingsetref pAlts
variable iterator AIterator
variable string AltName
variable string MainName

function LoadAltMappings()
{
    pAlts:Set[${LavishSettings[EQ2RaidAttendance].FindSet[Alts]}]

    pAlts:GetSetIterator[AIterator]
    if ${AIterator:First(exists)}
    {
        do
        {
            AltName:Set[${AIterator.Key}]
            MainName:Set[${pAlts.FindSet[${AltName}].FindSetting[Main]}]

            echo "${AltName} is an alt of ${MainName}"
        }
        while ${AIterator:Next(exists)}
    }
}

function SaveAltMapping(string AltCharName, string MainCharName)
{
    pAlts:AddSet[${AltCharName}]
    pAlts.FindSet[${AltCharName}]:AddSetting[Main,${MainCharName}]

    LavishSettings[EQ2RaidAttendance]:Export[${configfile}]
}
```

#### Key Lessons:

- **Per-character configs**: Use `${Me.Name}-${EQ2.ServerName}` in filename
- **Shared caches**: Separate caches for data shared across characters
- **Clear before import**: Use `:Clear` to reset settings before importing
- **Nested structures**: Create hierarchical settings with `:AddSet`
- **Iterator pattern**: Use `:GetSetIterator` to walk through all settings
- **FindSetting with default**: Provide default value if setting doesn't exist

---

## Command-Line Argument Parsing

### Pattern: Flexible Script Parameters

Scripts often need to accept various command-line arguments for flexibility.

#### Basic Argument Parsing

**Source:** [EQ2RaidAttendance](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2RaidAttendance) and production scripts

```lavishscript
function main(... Args)
{
    variable int Iterator = 1
    variable bool DoMastercrafted = FALSE
    variable bool VerboseMode = FALSE
    variable string TargetName

    ; Check if any arguments provided
    if ${Args.Size} > 0
    {
        do
        {
            ; Check for flags
            if (${Args[${Iterator}].Equal[-MASTERCRAFTED]})
                DoMastercrafted:Set[TRUE]
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

    echo "DoMastercrafted: ${DoMastercrafted}"
    echo "VerboseMode: ${VerboseMode}"
    echo "TargetName: ${TargetName}"
}
```

#### Advanced: Named Parameters with Values

**Source:** Production script patterns

```lavishscript
function main(... Args)
{
    variable float MinGold = 0
    variable float MaxGold = 0
    variable int SpendingMoney = 0
    variable int TotalItemsToBuy = 0

    if ${Args.Size} > 0
    {
        ; Special command: Reset mouse position
        if (${Args[1].Equal[ResetMouseClickLocation]})
        {
            call ResetMouseClickLocation
            if (${Return.Equal[ENDSCRIPT]})
                return
        }
        else
        {
            ; Positional parameters
            MinGold:Set[${Args[1]}]
            MaxGold:Set[${Args[2]}]
            SpendingMoney:Set[${Args[3]}]
            TotalItemsToBuy:Set[${Args[4]}]
        }
    }
    else
    {
        ; No arguments - display usage
        echo "Syntax: run BuyShineys MinGold MaxGold SpendingMoney TotalItemsToBuy"
        echo "        run BuyShineys ResetMouseClickLocation"
        echo "---"
        echo "Notes:  1.  'SpendingMoney' is in gold pieces (integer)"
        echo "        2.  'MinGold' and 'MaxGold' are float values (base price before commission)"
        return
    }
}
```

#### Advanced: Token-Based Named Parameters

**Source:** EQ2RaidAttendance.iss, EQ2Craft.iss

```lavishscript
function main(... Args)
{
    variable int count = 0
    variable string RaidName
    variable string RaidDate
    variable bool StatsOnly = FALSE
    variable bool SnapshotOnly = FALSE
    variable index:string Sitters

    ; Parse all arguments
    while ${count:Inc}<=${Args.Size}
    {
        ; Named parameter: Raid=<name>
        if ${Args[${count}].Token[1,=].Equal[Raid]}
            RaidName:Set[${Args[${count}].Token[2,=]}]

        ; Named parameter: Date=<date>
        elseif ${Args[${count}].Token[1,=].Equal[Date]}
            RaidDate:Set[${Args[${count}].Token[2,=]}]

        ; Flag parameter
        elseif ${Args[${count}].Find[-StatsOnly]}
            StatsOnly:Set[TRUE]

        elseif (${Args[${count}].Find[-SnapshotOnly]} || ${Args[${count}].Find[-SSOnly]})
            SnapshotOnly:Set[TRUE]

        ; Alt mapping: Alt=Main
        elseif ${Args[${count}].Find[=]}
        {
            echo "Adding '${Args[${count}].Token[1,=]}' as alt of '${Args[${count}].Token[2,=]}'"
            call SaveAltMapping "${Args[${count}].Token[1,=]}" "${Args[${count}].Token[2,=]}"
            return
        }

        ; Anything else is treated as a sitter name
        else
        {
            Sitters:Insert[${Args[${count}]}]
            echo "${Args[${count}]} is sitting out today"
        }
    }
}
```

#### EQ2Craft Command-Line Switch Pattern

**Source:** EQ2Craft.iss

```lavishscript
function main(... recipeFavourite)
{
    variable int argIndex=1
    variable bool V1 = FALSE
    variable bool V2 = FALSE
    variable bool EnableDebug = FALSE
    variable bool AllowWritSort = FALSE
    variable bool Quiet = FALSE
    variable bool startrecipe = FALSE
    variable bool skipwait = FALSE
    variable bool CraftLiteMode = FALSE
    variable bool HideUI = FALSE
    variable string CmdLineQueue

    while ${argIndex} <= ${recipeFavourite.Used}
    {
        switch ${recipeFavourite[${argIndex}]}
        {
            case -vv
                V2:Set[TRUE]
                break
            case -debug
                EnableDebug:Set[TRUE]
                Debug:Enable
                break
            case -sort
                AllowWritSort:Set[TRUE]
                break
            case -v
                V1:Set[TRUE]
                break
            case -q
                Quiet:Set[TRUE]
                break
            case -chat
                Verbose:Set[TRUE]
                break
            case -start
                startrecipe:Set[TRUE]
                break
            case -load
                skipwait:Set[TRUE]
                break
            case -lite
                CraftLiteMode:Set[TRUE]
                break
            case -hideui
                HideUI:Set[TRUE]
                break
            case -buffer
            case -script
                ; Ignore these flags
                break
            default
                ; Everything else is part of the queue
                CmdLineQueue:Set[${CmdLineQueue} ${recipeFavourite[${argIndex}]}]
        }
        argIndex:Inc
    }
}
```

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

### Pattern: Chat-Based Event Automation

Triggers allow scripts to respond to chat messages automatically.

#### Basic Trigger Pattern

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
    }
    while 1  ; Loop endlessly
}

; Harvest function called when trigger matches
function Harvest(string Line)
{
    ; If target is a resource and not in combat, harvest it
    if ${Target.Type.Equal[resource]} && !${Me.InCombat}
    {
        Actor[${Me.Target}]:DoubleClick
    }
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

- **AddTrigger syntax**: `AddTrigger FunctionName "pattern"`
- **Wildcards**: Use `@*@` to match any text before/after
- **Periodic checking**: Don't execute triggers every frame - use timer
- **ExecuteQueued**: Processes all queued trigger commands
- **Function signature**: Trigger function receives `string Line` parameter
- **Endless loop**: Use `while 1` or `while 0 < 1` for continuous monitoring

---

## Banking Automation

### Pattern: Automated Coin Management

Simple pattern for depositing/withdrawing coins at bank.

#### Basic Banking

**Source:** AutoBank.iss

```lavishscript
function main()
{
    ; Target the banker
    Actor[nokillnpc]:DoTarget
    wait 1

    ; Face the target
    Target:DoFace
    wait 1

    ; Open bank window
    Target:DoubleClick
    wait 1

    ; Deposit all coins
    Me:BankDeposit[p,${Me.Platinum}]
    Me:BankDeposit[g,${Me.Gold}]
    Me:BankDeposit[s,${Me.Silver}]
    Me:BankDeposit[c,${Me.Copper}]

    ; Withdraw specific amounts
    Me:BankWithdraw[p,${Me.Platinum}]
    Me:BankWithdraw[g,${Me.Gold}]
}
```

#### Key Lessons:

- **Actor[nokillnpc]**: Targets friendly NPCs only (excludes mobs)
- **:DoFace**: Face the target before interaction
- **:DoubleClick**: Opens interaction window
- **:BankDeposit[coin_type, amount]**: Coin types are p, g, s, c
- **:BankWithdraw[coin_type, amount]**: Same syntax as deposit
- **Wait between actions**: Give server time to process each request

---

## Broker Window Automation

### Pattern: Automated Broker Searches and Purchases

Complex pattern for searching broker and automating purchases.

#### Advanced Broker Automation

**Source:** Production script patterns

```lavishscript
variable float MinGold
variable float MaxGold
variable int SpendingMoney
variable int TotalItemsToBuy
variable int TotalBoughtItems = 0
variable float TotalCost = 0
variable int PageNum

function AutoBuyFromBroker()
{
    ; Target the broker
    if (${Target.ID} <= 0)
        Actor[xzrange,15,yrange,2,guild,"Broker"]:DoTarget

    wait 5

    if (${Target.ID} <= 0)
        target broker

    wait 5

    if (${Target.ID} <= 0)
    {
        echo "Unable to auto-target the broker."
        return
    }

    ; Open broker window
    Target:DoubleClick
    wait 5

    ; Perform broker search
    ; MinPrice and MaxPrice are in copper (gold * 10000)
    broker MinLevel 0 MaxLevel 105 Sort ByPriceAsc MinPrice ${Math.Calc[${MinGold}*10000]} MaxPrice ${Math.Calc[${MaxGold}*10000]} -type Collectible

    wait 20
    wait 20 ${BrokerWindow.SearchResult[1](exists)}

    ; Process search results
    if ${BrokerWindow.NumSearchResults}
    {
        PageNum:Set[1]
        do
        {
            ; Go to specific page if not page 1
            if ${PageNum}>1
            {
                BrokerWindow:GotoSearchPage[${PageNum}]
                wait 200 ${BrokerWindow.SearchResult[1].SerialNumber} != ${tmpInt64}
            }

            ; Process items on current page
            variable int i = 1
            do
            {
                echo "Checking ${BrokerWindow.SearchResult[${i}].Name}..."

                ; Check if price exceeds maximum
                if ${BrokerWindow.SearchResult[${i}].Price} > ${Math.Calc[${MaxGold}*100]}
                {
                    echo "Reached first item out of price range"
                    return
                }

                ; Check if we've bought enough items
                if ${TotalBoughtItems} == ${TotalItemsToBuy}
                {
                    echo "Reached maximum number of items to buy"
                    return
                }

                ; Check quantity available
                if (${BrokerWindow.SearchResult[${i}].Quantity} <= 0)
                {
                    echo "${BrokerWindow.SearchResult[${i}].Name} quantity is 0, skipping"
                    continue
                }

                ; Buy the item
                variable float CurrentItemPrice = ${BrokerWindow.SearchResult[${i}].Price}
                variable float CurrentItemPriceAsGold = ${Math.Calc[${CurrentItemPrice}/100]}
                variable int TmpQuantity = ${BrokerWindow.SearchResult[${i}].Quantity}

                BrokerWindow.SearchResult[${i}]:Buy[1]

                TotalBoughtItems:Inc
                TotalCost:Inc[${CurrentItemPriceAsGold}]

                echo "Bought ${BrokerWindow.SearchResult[${i}].Name} for ${CurrentItemPriceAsGold.Precision[3]} gold"
                echo "${TotalBoughtItems} bought for total of ${TotalCost.Precision[3]} gold"

                ; Wait for purchase to complete
                wait 5
                wait 200 ${BrokerWindow.SearchResult[${i}].Quantity} == ${Math.Calc[${TmpQuantity}-1]}

                ; Check spending limit
                if ${SpendingMoney} > 0
                {
                    if ${TotalCost} > ${SpendingMoney}
                    {
                        echo "We have spent more than our allocation (${SpendingMoney}g)"
                        return
                    }
                }
            }
            while ${i:Inc}<=${BrokerWindow.NumSearchResults}

            ; Save SerialNumber to detect page change
            variable int64 tmpInt64 = ${BrokerWindow.SearchResult[1].SerialNumber}
        }
        while ${PageNum:Inc}<=${BrokerWindow.TotalSearchPages}
    }
}
```

#### Key Lessons:

- **Actor search with range**: `Actor[xzrange,15,yrange,2,guild,"Broker"]`
- **broker command**: Use broker search command with parameters
- **Price conversion**: Broker API uses different units (price in copper/100)
- **Pagination**: Use `:GotoSearchPage[page]` and `TotalSearchPages`
- **SerialNumber tracking**: Detect when page has loaded new results
- **Quantity checking**: Verify item quantity before attempting purchase
- **`:Buy[quantity]`**: Purchase specified quantity
- **Wait with condition**: `wait 200 ${condition}` waits up to 20 seconds for condition

---

## Examine Window Interaction

### Pattern: Reading Examine Window Text

Examining items to read their properties from the examine window.

#### Examine Window Text Parsing

**Source:** Production script patterns

```lavishscript
variable int ExamineWaitCounter = 0
variable string TmpString

function CheckIfCollected(string ItemName)
{
    ; Examine the item
    BrokerWindow.SearchResult[${i}]:Examine

    ExamineWaitCounter:Set[0]
    do
    {
        wait 5

        ; Check if examine window opened for correct item
        if (${ExamineItemWindow(exists)})
        {
            TmpString:Set[${ExamineItemWindow.ToItem.Name}]
            if (${TmpString.Equal[${ItemName}]})
                break
        }

        echo "Waiting on ExamineItemWindow to open..."
        ExamineWaitCounter:Inc

        ; Retry examine after several attempts
        if ${ExamineWaitCounter} > 5
        {
            BrokerWindow.SearchResult[${i}]:Examine
            ExamineWaitCounter:Set[0]
        }

        wait 3
    }
    while !${TmpString.Equal[${ItemName}]}

    ; Check examine window text for "already collected"
    if (${ExamineItemWindow.TextVector} >= 2)
    {
        if (${ExamineItemWindow.TextVector[2].Type.Equal[Text]} && ${ExamineItemWindow.TextVector[2].Label.Find[already collected]} > 0)
        {
            echo "${ItemName} has already been collected."
            eq2execute /close_top_window
            wait 5
            return TRUE
        }
    }

    eq2execute /close_top_window
    wait 5
    return FALSE
}
```

#### Key Lessons:

- **:Examine method**: Opens examine window for item/ability
- **ExamineItemWindow**: TLO for examine window
- **ToItem member**: Gets item datatype from examine window
- **TextVector**: Array of text elements in examine window
- **Retry logic**: Re-examine if window doesn't open promptly
- **Name verification**: Ensure examine window is for correct item
- **Close window**: Use `/close_top_window` to close examine window
- **Text searching**: Use `.Find[text]` to search within label

---

## Collection Processing

### Pattern: Automated Collection Adding

Processing inventory items and adding collectibles to journal.

#### Collection Auto-Add

**Source:** Production script patterns

```lavishscript
variable string TmpString
variable int WaitCounter

function AddToCollections(string ItemName, float ItemPrice)
{
    ; Only auto-add if item is cheap (less than 2 plat)
    if (${ItemPrice} >= 200)
        return

    ; Wait for item to appear in inventory after purchase
    if (!${Me.Inventory[${ItemName}](exists)})
    {
        WaitCounter:Set[0]
        do
        {
            if (${WaitCounter} == 50)
                echo "Waiting for item to appear in inventory..."

            waitframe
            WaitCounter:Inc

            if (${WaitCounter} > 500)
            {
                echo "Giving up (perhaps being sold by the player?)"
                return
            }
        }
        while !${Me.Inventory[${ItemName}](exists)}
    }

    echo "Adding '${ItemName}' to collectibles..."

    ; Store broker window mouse position
    variable int BrokerWindowMouseX = ${Mouse.X}
    variable int BrokerWindowMouseY = ${Mouse.Y}

    ; Examine the item
    Me.Inventory[${ItemName}]:Examine
    wait 5

    ; Click the "Add to Journal" button (stored position)
    Mouse:SetPosition[${ToClickX},${ToClickY}]

    ; Wait until mouse is over the button
    if (!${EQ2UIPage[Journals,JournalsQuest].IsMouseOver})
    {
        do
        {
            waitframe
        }
        while !${EQ2UIPage[Journals,JournalsQuest].IsMouseOver}
    }

    wait 5
    Mouse:LeftClick
    wait 5

    ; Restore mouse to broker window
    Mouse:SetPosition[${BrokerWindowMouseX},${BrokerWindowMouseY}]
    waitframe

    ; Wait until mouse is no longer over journal
    if (${EQ2UIPage[Journals,JournalsQuest].IsMouseOver})
    {
        do
        {
            waitframe
        }
        while ${EQ2UIPage[Journals,JournalsQuest].IsMouseOver}
    }
}
```

#### Key Lessons:

- **Inventory existence check**: Verify item exists before interacting
- **Timeout pattern**: Give up after reasonable wait time
- **Frame-based waiting**: Use `waitframe` for tight polling loops
- **Mouse position preservation**: Save and restore to avoid disrupting user
- **UI hover checking**: Use `IsMouseOver` to verify mouse position
- **Sequential waits**: Wait between examine and click for UI responsiveness

---

## Raid Attendance Tracking

### Pattern: Raid Member Tracking and Statistics

Track raid membership across multiple sessions with alt support.

#### Raid Snapshot and Statistics

**Source:** EQ2RaidAttendance.iss

```lavishscript
variable(script) collection:uint CurMembers
variable(script) collection:uint TotalMembers
variable(script) collection:string Alts
variable(script) index:string Sitters
variable(script) uint RaidCount
variable(script) string RaidName

function TakeSnapShot()
{
    variable iterator SIterator
    variable string PlayerName
    variable int SSCount
    variable int i = 1
    variable int Counter = 0

    echo "Taking a Snapshot of today's raid..."

    ; Process people in current raid
    if (${Me.Raid} > 0)
    {
        do
        {
            if ${Me.Raid[${i}].Name(exists)}
            {
                ; Check if already counted
                if (${CurMembers.Element[${Me.Raid[${i}].Name}].Name(exists)})
                {
                    ; Increment snapshot count
                    SSCount:Set[${CurMembers.Element[${Me.Raid[${i}].Name}]}]
                    CurMembers:Set[${Me.Raid[${i}].Name},${Math.Calc[${SSCount}+1].Precision[0]}]
                    echo "${Me.Raid[${i}].Name} already exists (${Math.Calc[${SSCount}+1].Precision[0]})"
                }
                else
                {
                    ; First snapshot for this member
                    CurMembers:Set[${Me.Raid[${i}].Name},1]
                    echo "${Me.Raid[${i}].Name} added to raid (1)"
                }
                Counter:Inc
            }
        }
        while ${i:Inc} <= ${Me.Raid}
    }

    echo "${Counter} people found in current raid force"

    ; Process people sitting out
    Sitters:GetIterator[SIterator]
    if ${SIterator:First(exists)}
    {
        do
        {
            PlayerName:Set[${SIterator.Value}]

            if (${CurMembers.Element[${PlayerName}](exists)})
            {
                SSCount:Set[${CurMembers.Element[${PlayerName}]}]
                CurMembers:Set[${PlayerName},${Math.Calc[${SSCount}+1].Precision[0]}]
            }
            else
            {
                CurMembers:Set[${PlayerName},1]
                echo "${PlayerName} added to raid (sitting out)"
            }
        }
        while ${SIterator:Next(exists)}
    }

    echo "Snapshot complete :: ${CurMembers.Used} raid members counted"
}

function GenerateStats()
{
    variable iterator SIterator
    variable string PlayerName
    variable string MainName
    variable uint PlayerRaidCount
    variable float Percentage
    variable float AverageCounter = 0
    variable float Average

    echo "Generating Statistics..."

    ; Calculate attendance percentages
    TotalMembers:GetIterator[SIterator]
    if ${SIterator:First(exists)}
    {
        do
        {
            PlayerName:Set[${SIterator.Key}]
            PlayerRaidCount:Set[${SIterator.Value}]

            ; Calculate percentage
            Percentage:Set[${Math.Calc[${PlayerRaidCount}/${RaidCount}*100]}]

            echo "${PlayerName} attended ${PlayerRaidCount} of ${RaidCount} raids (${Percentage.Precision[2]}%)"

            ; Track for average calculation
            AverageCounter:Set[${Math.Calc[${AverageCounter}+${Percentage}]}]
        }
        while ${SIterator:Next(exists)}
    }

    ; Calculate average attendance
    Average:Set[${Math.Calc[${AverageCounter}/${TotalMembers.Used}]}]
    echo "Average attendance rate is ${Average.Precision[2]}%"
}
```

#### Alt Tracking Pattern

**Source:** EQ2RaidAttendance.iss

```lavishscript
variable(script) collection:string Alts
variable(script) collection:string AltCreditCheck

function ProcessAltAttendance()
{
    variable string PlayerName
    variable string MainName
    variable uint PlayerRaidCount
    variable bool AlreadyCredited

    ; Check if this player is an alt
    if (${Alts.Element[${PlayerName}](exists)})
    {
        MainName:Set[${Alts.Element[${PlayerName}]}]

        echo "${PlayerName} :: **ALT**"

        ; Check if main character is already in this raid
        if ${pRaids.FindSet[${RaidName}].FindSet[${MainName}](exists)}
        {
            echo "${MainName} is already counted in this raid. Skipping..."
            return
        }

        ; Credit the main character
        if (${TotalMembers.Element[${MainName}](exists)})
        {
            ; Check if we already credited this main in this raid
            if (!${AltCreditCheck.Element[${MainName}](exists)})
            {
                PlayerRaidCount:Set[${TotalMembers.Element[${MainName}]}]
                TotalMembers:Set[${MainName},${Math.Calc[${PlayerRaidCount}+1].Precision[0]}]
                echo "Adding to ${MainName}'s total (${Math.Calc[${PlayerRaidCount}+1].Precision[0]})"

                ; Mark that we've credited this main
                AltCreditCheck:Set[${MainName},${PlayerName}]
            }
            else
            {
                echo "${MainName} already given credit for one alt this raid"
            }
        }
        else
        {
            TotalMembers:Set[${MainName},1]
            echo "Adding to ${MainName}'s total (1)"
            AltCreditCheck:Set[${MainName},${PlayerName}]
        }
    }
}
```

#### Key Lessons:

- **Collection usage**: `collection:uint` for name->count mapping
- **Element existence**: Check `.Element[key](exists)` before accessing
- **Me.Raid**: Provides raid member count and access by index
- **Me.Raid[index].Name**: Get raid member name
- **Snapshot counting**: Track multiple snapshots per raid
- **Alt credit logic**: Prevent double-counting when alt and main both attend
- **AltCreditCheck**: Per-raid tracking to ensure one credit per main
- **Statistics calculation**: Generate percentages and averages

---

## Localization Support

### Pattern: Multi-Language Script Support

Supporting multiple EQ2 servers with different languages.

#### Localization Pattern

**Source:** EQ2Craft.iss

```lavishscript
variable string sLocalization
variable string RushOrder_GuildTag
variable string WorkOrder_GuildTag
variable string Broker_GuildTag
variable string Wholesaler
variable string WritInitialConvo
variable string StoveAndKeg
variable string Forge

function SetLocalization()
{
    switch ${EQ2.ServerName}
    {
        case Storms
            ; French server
            Debug:Echo["Utilizing French Localization"]
            sLocalization:Set["French"]

            RushOrder_GuildTag:Set["ordres urgents"]
            WorkOrder_GuildTag:Set["ordres urgents"]
            Broker_GuildTag:Set["Négociant"]
            Wholesaler:Set["Wholesaler"]

            WritInitialConvo:Set["Je voudrais "]

            StoveAndKeg:Set["Cuisinière et baril"]
            Forge:Set["Forge"]
            break

        default
            ; English (default)
            sLocalization:Set["English"]

            RushOrder_GuildTag:Set["Rush Orders"]
            WorkOrder_GuildTag:Set["Work Orders"]
            Broker_GuildTag:Set["Broker"]
            Wholesaler:Set["Wholesaler"]

            WritInitialConvo:Set["I would like"]

            StoveAndKeg:Set["Stove & Keg"]
            Forge:Set["Forge"]
            break
    }
}

function main()
{
    ; Set localization first
    call SetLocalization

    ; Use localized strings
    Actor[guild,"${Broker_GuildTag}"]:DoTarget

    ; In NPC dialog
    eq2execute /say ${WritInitialConvo} a work order
}
```

#### Key Lessons:

- **EQ2.ServerName**: Detect which server the character is on
- **Switch by server**: Different localizations for different servers
- **String variables**: Store all localizable text in variables
- **Default clause**: Fallback to English if server not recognized
- **Guild tag targeting**: Many NPCs identified by guild tag
- **Dialog text**: NPC conversations may differ by language

---

## Navigation System Integration

### Pattern: Using EQ2Nav for Pathfinding

Integration with the EQ2Nav navigation system for automated movement.

#### Navigation Setup

**Source:** EQ2Craft.iss

```lavishscript
#include "${LavishScript.HomeDirectory}/Scripts/EQ2Navigation/EQ2Nav_Lib.iss"

variable EQ2Nav Nav

function main()
{
    ; Initialize Navigation System
    Nav:UseLSO[FALSE]
    Nav:LoadMap

    ; Validate that navigation regions exist
    call ValidateRegions

    ; Use navigation
    call NavigateToForge
}

function ValidateRegions()
{
    variable set Regions
    variable iterator Region

    ; Add all expected regions
    Regions:Add[RushOrder]
    Regions:Add[WorkOrder]
    Regions:Add[Broker]
    Regions:Add[Forge 1]
    Regions:Add[Forge 2]
    Regions:Add[Forge 3]

    ; Check each region
    Regions:GetIterator[Region]
    Region:First
    do
    {
        if ${Nav.RegionExists[${Region.Value}]}
        {
            echo "Found: ${Region.Value} Path: ${Nav.AvailablePathToRegion[${Region.Value}]}"
        }
        else
        {
            echo "Missing: ${Region.Value}"
        }
    }
    while ${Region:Next(exists)}
}

function NavigateToForge()
{
    ; Start navigation
    Nav:MoveToRegion["Forge 1"]

    ; Wait until navigation complete
    do
    {
        Nav:Pulse
        wait 0
    }
    while ${ISXEQ2(exists)} && ${Nav.Moving}

    echo "Arrived at Forge 1"
}
```

#### Key Lessons:

- **#include pattern**: Include navigation library
- **Nav:UseLSO[FALSE]**: Disable LSO mode
- **Nav:LoadMap**: Load navigation map for current zone
- **Nav:RegionExists[name]**: Check if region is defined
- **Nav:MoveToRegion[name]**: Start navigation to region
- **Nav:Pulse**: Must be called in loop during navigation
- **Nav.Moving**: Boolean indicating navigation in progress
- **set datatype**: Use for collection of unique strings

---

## Script Include Patterns

### Pattern: Modular Script Organization

Organizing scripts with includes for code reuse.

#### Include Patterns

**Source:** EQ2Craft.iss

```lavishscript
; Standard includes
#include "${LavishScript.HomeDirectory}/Scripts/EQ2Navigation/EQ2Nav_Lib.iss"
#include "${LavishScript.HomeDirectory}/Scripts/EQ2Craft/Include/Search.iss"

; Optional includes (don't fail if missing)
#includeoptional "${LavishScript.HomeDirectory}/Scripts/EQ2Common/Debug.iss"
#includeoptional "${LavishScript.HomeDirectory}/Scripts/EQ2Common/MovementKeys.iss"

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

**Source:** EQ2Craft.iss, EQ2RaidAttendance.iss

```lavishscript
variable bool V1 = FALSE   ; Verbose mode
variable bool V2 = FALSE   ; Very verbose mode
variable bool EnableDebug = FALSE
variable bool EchoInChat = FALSE

; Custom echo for chat window
atom(script) ChatEcho(... Params)
{
    if !${Params.Size}
        return

    if ${EchoInChat} && ${V1}
    {
        eq2echo ${Params.Expand}
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
    Debug:SetFilename["${LavishScript.HomeDirectory}/Scripts/EQ2Craft/Craft_Debug.txt"]

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

#### EQ2RaidAttendance Debug Pattern

**Source:** EQ2RaidAttendance.iss

```lavishscript
#include EQ2Common/Debug.iss

variable(script) string sFileName

function main()
{
    ; Initialize debug system
    Debug:Enable
    Debug:SetPrefix[]
    Debug:SetEchoAlsoLogs[TRUE]
    Debug:SetFilename["EQ2RaidAttendance.txt"]

    ; Headers
    Debug:Echo["-------- EQ2 Raid Attendance --------"]
    Debug:Echo["(Created ${Time.Date} at ${Time})"]
    Debug:Echo["-"]

    ; Debug output
    Debug:Echo["- Initializing..."]
    Debug:Echo["-- ${Args[${count}]} is sitting out today"]

    ; Log at end
    Debug:Echo["-----------------------------------"]
    Debug:Log["\n\n\n"]
}
```

#### Key Lessons:

- **atom(script) pattern**: Create reusable echo functions
- **... Params**: Variable parameter list
- **Params.Size**: Number of parameters
- **Params.Expand**: Expand all parameters into string
- **eq2echo**: Echoes to EQ2 chat window
- **echo**: Echoes to console only
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

**Source:** EQ2Craft.iss

```lavishscript
variable bool EnableTTS = FALSE

atom(script) ChatSay(... Params)
{
    if !${Params.Used} || !${EnableTTS}
        return

    Debug:Echo["EQ2Craft::ChatSay '${Params.Expand}'"]

    if ${TTS.IsReady}
        speak "${Params.Expand}"
}

function main()
{
    ; Enable TTS from configuration
    EnableTTS:Set[${Configuration.FindSetting[EnableTTS,FALSE]}]

    ; Use TTS
    ChatSay "Crafting complete"
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

Managing ISXEQ2 and other extensions.

#### Extension Loading Pattern

**Source:** EQ2AutoLogin.iss

```lavishscript
function main()
{
    ; Try to load ISXEQ2 if not already loaded
    if !${Extension[ISXEQ2](exists)}
        ext isxeq2

    ; Check if ISXEQ2 is loaded
    if ${Extension[ISXEQ2](exists)}
    {
        echo "Waiting for ISXEQ2 to get ready..."
        wait 300 ${ISXEQ2.IsReady}
        echo "ISXEQ2 should now be Ready!"
        wait 40
    }
    else
    {
        echo "No ISXEQ2 extension loaded!"
        wait 70
    }

    ; Use ISXEQ2 if available
    if ${Extension[ISXEQ2](exists)}
    {
        echo "Using ISXEQ2 features..."
        do
        {
            waitframe
        }
        while !${ISXEQ2.IsReady} && ${Extension[ISXEQ2](exists)}
    }
    else
    {
        echo "Using Lavish-only features..."
    }
}
```

#### ISXEQ2 Readiness Pattern

**Source:** EQ2Craft.iss

```lavishscript
function main()
{
    if !${ISXEQ2.IsReady}
    {
        ErrorEcho "ISXEQ2 has not been loaded! Ending script..."
        Script:End
    }

    ; Clear caches
    ISXEQ2:SetCustomVariable[SRO,0]
    ISXEQ2:ClearAbilitiesCache

    ; Force recipe book to open for data load
    if (!${Me.Recipe[1](exists)})
    {
        ChatEcho "Opening Recipe Book to Retrieve Data from Server"
        EQ2Execute /toggletradeskills
        wait 50
        EQ2Execute /toggletradeskills
    }
}
```

#### Key Lessons:

- **Extension[name](exists)**: Check if extension loaded
- **ext extensionname**: Load an extension
- **ISXEQ2.IsReady**: Wait for ISXEQ2 to fully initialize
- **wait with timeout**: `wait 300 ${condition}` = wait up to 30 seconds
- **Graceful degradation**: Provide fallback when extension unavailable
- **Cache clearing**: ISXEQ2:ClearAbilitiesCache for fresh data
- **Force data load**: Open/close windows to trigger server data

---

## Summary of New Patterns

### Patterns Added from Additional Script Analysis:

1. **Mouse Automation** - UI clicking, position storage, mouse restoration
2. **LavishSettings XML** - Configuration persistence, nested settings, iteration
3. **Command-Line Parsing** - Flag parsing, named parameters, token-based args
4. **Trigger-Based Automation** - Chat pattern matching, queued commands
5. **Banking Automation** - Coin management, NPC targeting
6. **Broker Automation** - Search, pagination, purchasing
7. **Examine Window** - Text parsing, retry logic
8. **Collection Processing** - Auto-adding to journal, mouse coordination
9. **Raid Attendance** - Member tracking, alt handling, statistics
10. **Localization** - Multi-language support, server detection
11. **Navigation Integration** - EQ2Nav usage, region validation
12. **Script Includes** - Modular organization, optional includes
13. **Advanced Debugging** - Multi-level verbosity, file logging
14. **Text-to-Speech** - Audio notifications
15. **Extension Management** - Loading, checking, graceful degradation

---

## Cross-References

- **Core Concepts:** [03_Core_Concepts.md](03_Core_Concepts.md)
- **Best Practices:** [04_Patterns_And_Best_Practices.md](04_Patterns_And_Best_Practices.md)
- **Working Examples:** [05_Working_Examples.md](05_Working_Examples.md)
- **API Reference:** [01_API_Reference.md](01_API_Reference.md)

---

*Part of ISXEQ2 Scripting Guide*
