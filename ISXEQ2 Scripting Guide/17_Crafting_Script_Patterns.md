# Crafting Script Patterns

**Advanced patterns for tradeskill automation from EQ2Craft**

---

## Table of Contents

1. [Overview](#overview)
2. [Command-Line Argument Parsing](#command-line-argument-parsing)
3. [Navigation Integration](#navigation-integration)
4. [Localization and Multi-Language Support](#localization-and-multi-language-support)
5. [UI State Management](#ui-state-management)
6. [Recipe Queue Management](#recipe-queue-management)
7. [Crafting Window Monitoring](#crafting-window-monitoring)
8. [Event-Driven Crafting Loop](#event-driven-crafting-loop)
9. [Device and Station Targeting](#device-and-station-targeting)
10. [Writ Automation](#writ-automation)
11. [Durability and Quality Management](#durability-and-quality-management)
12. [Reaction Arts System](#reaction-arts-system)
13. [Progress Tracking and Statistics](#progress-tracking-and-statistics)
14. [File-Based Configuration](#file-based-configuration)
15. [Complete Working Example](#complete-working-example)

---

## Overview

EQ2Craft is a sophisticated crafting automation script that demonstrates production-grade patterns for tradeskill automation. This guide extracts the most valuable patterns for use in your own scripts.

**Source:** https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Craft

**Key Features Demonstrated:**
- Command-line argument parsing for flexible script invocation
- Multi-language localization support
- Complex UI state management
- Navigation system integration
- Event-driven crafting loops
- Recipe queue management
- Statistics tracking

---

## Command-Line Argument Parsing

### Pattern: Flexible Script Invocation

EQ2Craft supports multiple command-line options for different use cases:

```lavishscript
function main(... recipeFavourite)
{
    variable int argIndex = 1
    variable bool V1 = FALSE           ; Verbose mode
    variable bool V2 = FALSE           ; Debug mode
    variable bool EnableDebug = FALSE
    variable bool AllowWritSort = FALSE
    variable bool startrecipe = FALSE
    variable bool skipwait = FALSE
    variable bool CraftLiteMode = FALSE
    variable bool HideUI = FALSE
    variable string CmdLineQueue

    ; Parse command-line arguments
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
            ; Accumulate unknown args as queue name
            CmdLineQueue:Set[${CmdLineQueue} ${recipeFavourite[${argIndex}]}]
        }
        argIndex:Inc
    }

    ; Validate argument combinations
    if ${CmdLineQueue.Length} == 0
    {
        skipwait:Set[FALSE]
        startrecipe:Set[FALSE]
    }
    elseif !${skipwait}
    {
        skipwait:Set[TRUE]
        startrecipe:Set[TRUE]
    }
}
```

**Usage Examples:**
```
run EQ2Craft -v                           ; Verbose mode
run EQ2Craft -debug MyQueue               ; Debug mode with queue
run EQ2Craft -lite -hideui MyQueue        ; Lite mode, no UI
run EQ2Craft -start -load MyQueue         ; Auto-start with queue
```

**Key Techniques:**
- `... recipeFavourite` accepts variable number of arguments
- `${recipeFavourite.Used}` gets argument count
- Switch statement for flag parsing
- Default case accumulates non-flag arguments
- Post-parse validation for argument combinations

---

## Navigation Integration

### Pattern: EQ2Nav Integration

EQ2Craft integrates with the EQ2Nav navigation library:

```lavishscript
#include "${LavishScript.HomeDirectory}/Scripts/EQ2Navigation/EQ2Nav_Lib.iss"

variable EQ2Nav Nav

function main()
{
    ; Initialize navigation
    Nav:UseLSO[FALSE]
    Nav:LoadMap
    Nav.SmartDestinationDetection:Set[FALSE]
    Nav.BackupTime:Set[3]
    Nav.StrafeTime:Set[3]

    ; Navigate to a region
    call MovetoDevice "Forge" 1
}

function MovetoDevice(string devicename, int devicenum)
{
    variable string ldevicename

    ; Convert device name to localized name
    switch ${devicename}
    {
        case Forge
            ldevicename:Set[${Forge}]  ; From localization
            break
        case Stove and Keg
            devicename:Set["Stove & Keg"]
            ldevicename:Set[${StoveAndKeg}]
            break
    }

    ; Set navigation precision
    Nav.gPrecision:Set[1]
    Nav.DestinationPrecision:Set[2]

    ; Special handling for tradeskill instances
    if ${Zone.ShortName.Find[tradeskill]}
        Nav.DestinationPrecision:Set[3]

    ; Navigate to the region
    if ${devicenum}
    {
        if ${Nav.RegionExists[${devicename} ${devicenum}]}
            Nav:MoveToRegion["${devicename} ${devicenum}"]
        else
            Debug:Echo[NAV WARNING: Missing region: ${devicename} ${devicenum}]
    }
    else
    {
        if ${Nav.RegionExists[${devicename}]}
            Nav:MoveToRegion["${devicename}"]
        else
            Debug:Echo[NAV WARNING: Missing region: ${devicename}]
    }

    ; Pulse navigation until complete
    do
    {
        Nav:Pulse
        wait 0
    }
    while ${ISXEQ2(exists)} && ${Nav.Moving}

    ; Stop any residual movement
    waitframe
    call StopRunning
    waitframe
}

function StopRunning()
{
    if ${Me.IsMoving}
    {
        press -hold "${Nav.BACKWARD}"
        wait 2
        press -release "${Nav.BACKWARD}"
    }
}
```

**Key Techniques:**
- `Nav:LoadMap` initializes navigation data
- `Nav.RegionExists` validates regions before navigation
- `Nav:MoveToRegion` starts navigation to named region
- `Nav:Pulse` in loop processes navigation frames
- `${Nav.Moving}` checks if navigation is in progress
- Adjust precision based on zone type

### Pattern: Region Validation

Validate navigation regions at startup:

```lavishscript
function ValidateRegions()
{
    variable set Regions
    variable iterator Region

    ; Build list of expected regions
    Regions:Add[RushOrder]
    Regions:Add[WorkOrder]
    Regions:Add[Wholesaler]
    Regions:Add[Broker]
    Regions:Add[Invoices]

    variable int Counter
    for (Counter:Set[1]; ${Counter} <= 3; Counter:Inc)
    {
        Regions:Add[Invoices ${Counter}]
        Regions:Add[Chemistry Table ${Counter}]
        Regions:Add[Work Bench ${Counter}]
        Regions:Add[Sewing Table & Mannequin ${Counter}]
        Regions:Add[Woodworking Table ${Counter}]
        Regions:Add[Stove & Keg ${Counter}]
        Regions:Add[Forge ${Counter}]
        Regions:Add[Engraved Desk ${Counter}]
    }

    ; Validate each region
    Regions:GetIterator[Region]
    Region:First
    do
    {
        if ${Nav.RegionExists[${Region.Value}]}
        {
            Debug:Echo["Found: ${Region.Value} - Path Available: ${Nav.AvailablePathToRegion[${Region.Value}]}"]
        }
        else
        {
            Debug:Echo["Missing: ${Region.Value}"]
        }
    }
    while ${Region:Next(exists)}
}
```

---

## Localization and Multi-Language Support

### Pattern: Server-Based Localization

Support multiple languages based on server:

```lavishscript
variable string sLocalization
variable string RushOrder_GuildTag
variable string WorkOrder_GuildTag
variable string Wholesaler
variable string WritInitialConvo
variable string StoveAndKeg
variable string SewingTableAndMannequin
variable string Forge
; ... more localized strings

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
            Wholesaler:Set["Wholesaler"]
            WritInitialConvo:Set["Je voudrais "]

            StoveAndKeg:Set["Cuisinière et baril"]
            SewingTableAndMannequin:Set["Table de couture avec un mannequin"]
            Forge:Set["Forge"]
            break

        default
            ; English servers
            sLocalization:Set["English"]

            RushOrder_GuildTag:Set["Rush Orders"]
            WorkOrder_GuildTag:Set["Work Orders"]
            Wholesaler:Set["Wholesaler"]
            WritInitialConvo:Set["I would like"]

            StoveAndKeg:Set["Stove & Keg"]
            SewingTableAndMannequin:Set["Sewing Table & Mannequin"]
            Forge:Set["Forge"]
            break
    }
}

function main()
{
    ; Initialize localization
    call SetLocalization

    ; Use localized strings
    if ${Actor[xzrange,10,yrange,2,guild,${RushOrder_GuildTag}].Name(exists)}
    {
        echo "Found Rush Order agent"
    }
}
```

**Key Techniques:**
- Use `${EQ2.ServerName}` to detect server
- Store all UI text in variables
- Switch statement for different languages
- Centralized function for all localization

---

## UI State Management

### Pattern: Mode Switching

EQ2Craft has "full" and "lite" modes with different UI states:

```lavishscript
variable bool CraftLite = FALSE
variable bool CraftLiteMode = FALSE

function main()
{
    if ${CraftLiteMode}
    {
        Craft:SetMode[lite]
        UIElement[Main Frame@Craft Selection]:Hide
        UIElement[Craft Selection]:Minimize
        UIElement[CraftLite@Titlebar@Craft Selection]:SetText[Craft Full]
    }

    while 1
    {
        ; Normal mode processing
        if !${CraftLite}
        {
            ; Full UI loop
            do
            {
                call ProcessTriggers
                call CheckInventory
                Craft:InitGUI
            }
            while !${startrecipe}
        }

        ; Lite mode processing
        while ${CraftLite}
        {
            waitframe
            call ProcessTriggers

            if ${roundstart} && ${craftingready}
            {
                ; Simplified crafting loop
                call ProcessArts
            }
        }
    }
}
```

### Pattern: UI Visibility Control

Control which UI elements are shown based on state:

```lavishscript
; Hide crafting UI elements
UIElement[Craft Selection].FindUsableChild[Start Crafting,commandbutton]:Hide
UIElement[Craft Selection].FindUsableChild[Save List,commandcheckbox]:Hide
UIElement[Craft Selection].FindUsableChild[Add Recipe,commandbutton]:Hide

; Show progress UI
UIElement[Craft Selection].FindUsableChild[Process Recipe,variablegauge]:Show
UIElement[Craft Selection].FindUsableChild[Gauge Label,text]:Show

; Update progress
gaugelevel:Set[0.5]
UIElement[Craft Selection].FindUsableChild[Process Recipe,variablegauge].FindChild[Fill,gauge]:SetLevel[${gaugelevel}]
UIElement[Craft Selection].FindUsableChild[Gauge Label,text]:SetText[Processing Recipe...]
```

---

## Recipe Queue Management

### Pattern: Multi-Dimensional Recipe Arrays

Store recipe queue data in multi-dimensional arrays:

```lavishscript
variable float MakeQty[2,200]          ; Quantity to make
variable int MakeCnt[2]                ; Count of recipes in each tier
variable string MakeName[2,200]        ; Recipe names
variable int64 MakeID[2,200]           ; Recipe IDs
variable string MakeKnowledge[2,200]   ; Knowledge type (Artisan, etc.)
variable string MakeDevice[2,200]      ; Crafting device needed
variable int MakeQlt[2,200]            ; Quality level (1-4)
variable string MakeFuelName[2,200]    ; Fuel required
variable int MakeFuelCnt[2,200]        ; Fuel quantity
variable int MakeProduce[2,200]        ; Items produced per combine
variable int MakeLevel[2,200]          ; Recipe level

; Process queue
variable int xvar = 2
variable int xval

do
{
    xval:Set[${MakeCnt[${xvar}]}]
    do
    {
        if ${MakeCnt[${xvar}]} && ${MakeQty[${xvar},${xval}]} > 0 && ${MakeName[${xvar},${xval}].Length}
        {
            ; Process this recipe
            call Craft ${MakeID[${xvar},${xval}]} ${MakeQty[${xvar},${xval}].Ceil} ${MakeQlt[${xvar},${xval}]} ${xvar} ${xval}
        }
    }
    while ${xval:Dec} > 0
}
while ${xvar:Dec} > 0
```

**Why Two Tiers?**
- Tier 1: Regular recipes
- Tier 2: Rare recipes (different durability settings)

### Pattern: Recipe Queue from Settings

Load recipes from saved queue files:

```lavishscript
variable settingsetref CraftQueue
variable settingsetref LastSavedQueue

function LoadQueue(string queuename)
{
    variable filepath QueuePath = "${LavishScript.HomeDirectory}/Scripts/EQ2Craft/Queues"

    ; Load the queue file
    LavishSettings:AddSet[Craft Queue File]
    LavishSettings[Craft Queue File]:Import[${QueuePath}/${queuename}.xml]
    LavishSettings[Craft Queue File]:GetSet[Craft Queue File]:GetSetIterator[sIterator]

    ; Clear existing queue
    FailedRecipe:Clear

    ; Process each recipe in queue
    if ${sIterator:First(exists)}
    {
        do
        {
            ; sIterator.Key = recipe name
            ; sIterator.Value = quantity
            FailedRecipe:Insert["${sIterator.Key} ${sIterator.Value}"]
        }
        while ${sIterator:Next(exists)}
    }

    return TRUE
}

function SaveQueue()
{
    variable int tempvar = 0
    variable string tmplist

    CraftQueue:Clear

    ; Build queue from UI listbox
    while ${UIElement[Craft Selection].FindUsableChild[Craft List,listbox].Item[${tempvar:Inc}](exists)}
    {
        tmplist:Set[${UIElement[Craft Selection].FindUsableChild[Craft List,listbox].Item[${tempvar}]}]
        ; Format: "RecipeName|Quantity"
        CraftQueue:AddSetting[${tmplist.Token[1,|]},${tmplist.Token[2,|]}]
    }

    CraftQueue:Export[${QueuePath}/${Me.TSSubClass}-Last_Crafted_Queue.xml]
    CraftQueue:Clear
}
```

---

## Crafting Window Monitoring

### Pattern: UI Page Detection

Monitor crafting UI state by checking specific UI elements:

```lavishscript
variable bool roundstart = FALSE
variable bool complete = FALSE
variable int CurrentQuality = 0
variable int CurrentProgress = 0
variable int CurrentDurability = 0
variable string CurrentReactive

; Check if we're in crafting mode
if ${EQ2UIPage[Tradeskills,Tradeskills].Child[text,Tradeskills.TabPages.Craft.Create.RecipeName](exists)}
{
    ; We're on the crafting page
    craftingrecipe:Set[${EQ2UIPage[Tradeskills,Tradeskills].Child[text,Tradeskills.TabPages.Craft.Create.RecipeName].GetProperty[LocalText]}]
    craftingknowledge:Set[${Me.Recipe[${craftingrecipe}].Knowledge}]
}

; Check if crafting has started
if ${EQ2UIPage[Tradeskills,Tradeskills].Child[button,Tradeskills.TabPages.Craft.Create.Stop](exists)}
{
    ; Stop button exists = crafting is active
    roundstart:Set[TRUE]
}

; Check for completion
if ${EQ2UIPage[Tradeskills,Tradeskills].Child[button,Tradeskills.TabPages.Craft.Create.Repeat](exists)}
{
    ; Repeat button exists = crafting is complete
    complete:Set[TRUE]
}
```

### Pattern: Page Stuck Detection

Detect when UI is stuck in preparation screen:

```lavishscript
objectdef EQ2Craft
{
    method CheckPageStuck() returns bool
    {
        ; Check if we're stuck on the prep page
        if ${EQ2UIPage[Tradeskills,Tradeskills].Child[text,TradeSkills.TabPages.Craft.prepare.summarypage.pccount].GetProperty[LocalText](exists)}
        {
            variable string pText
            variable int currentqty
            variable int maxqty

            pText:Set[${EQ2UIPage[Tradeskills,Tradeskills].Child[text,TradeSkills.TabPages.Craft.prepare.summarypage.pccount].GetProperty[LocalText]}]

            ; Text format: "current / max"
            currentqty:Set[${pText.Token[1,/]}]
            maxqty:Set[${pText.Token[2,/]}]

            ; If current != max, we're stuck
            if ${currentqty} != ${maxqty}
            {
                ErrorEcho "Missing components for recipe!"
                return TRUE
            }
        }

        return FALSE
    }
}
```

---

## Event-Driven Crafting Loop

### Pattern: Trigger-Based Updates

Use event triggers to detect crafting state changes:

```lavishscript
objectdef EQ2Craft
{
    method InitTriggers()
    {
        ; These triggers fire when UI elements appear/change
        ; Implementation varies based on ISXEQ2 version
    }

    method InitEvents()
    {
        Event[EQ2_onIncomingText]:AttachAtom[OnIncomingText]
    }
}

atom OnIncomingText(string Text, string ChannelName, string SpeakerName)
{
    ; Detect crafting events from game text
    if ${Text.Find["You have created"]}
    {
        complete:Set[TRUE]

        ; Extract item name
        variable string ItemName
        ItemName:Set[${Text.Token[4," "]}]  ; "You have created [item]"
        ItemsCreated:Insert[${ItemName}]
    }

    if ${Text.Find["You have failed to create"]}
    {
        complete:Set[TRUE]
    }
}

function ProcessTriggers()
{
    ; Check for updates from triggers
    ; This is called frequently in the main loop
    FlushQueued
}
```

### Pattern: Round Start Detection

Detect when a new crafting round starts:

```lavishscript
; Event fires when round starts
atom OnRoundStart()
{
    roundstart:Set[TRUE]

    ; Reset counters
    CurrentQuality:Set[0]
    timer:Set[${Time.Timestamp}]
    chktotdur:Set[TRUE]
}

; In main crafting loop
while ${CurrentQuality} < ${QualityResult}
{
    if ${roundstart} && !${complete}
    {
        ; Cancel buffs from previous round
        Craft:CancelBuffs
        wait 3

        roundstart:Set[FALSE]
        timer:Set[${Time.Timestamp}]

        ; Calculate durability
        if ${chktotdur}
        {
            TotalDurability:Set[${Math.Calc[${CurrentDurability}-${ChangeinDur}]}]
            chktotdur:Set[FALSE]
        }

        ; Process reaction arts
        call ProcessReactionArts
    }

    ; Check for timeout
    if ${Time.Timestamp} - ${timer} > 30
    {
        ErrorEcho "Crafting timeout!"
        break
    }

    waitframe
}
```

---

## Device and Station Targeting

### Pattern: Actor Search with Fallbacks

Find crafting stations with multiple fallback strategies:

```lavishscript
function MovetoDevice(string devicename, int devicenum)
{
    variable bool InGuildHall = FALSE

    if ${Zone.ShortName.Find[guildhall]} > 0
        InGuildHall:Set[TRUE]

    ; Strategy 1: Check if device is nearby
    if ${Actor[${ldevicename},xzrange,${DistToTable},yrange,2].Name(exists)}
    {
        if ${InGuildHall}
        {
            ; Try elaborate table first
            ElaborateTable:Set["Elaborate ${ldevicename}"]

            if !${Actor[ExactName,${ElaborateTable},xzrange,${DistToTable},yrange,2].CheckCollision}
            {
                Actor[ExactName,${ElaborateTable}]:DoTarget
                wait 10 "${Target.ID}==${Actor[ExactName,${ElaborateTable}].ID}"
                if ${Target.Name.Equal[${ElaborateTable}]}
                    face
                wait 10
                lastdevice:Set[${ldevicename}]
                return
            }

            ; Try guild crafting station
            GuildCraftingStation:Set["Guild Crafting Station: ${ldevicename}"]

            if !${Actor[ExactName,${GuildCraftingStation},xzrange,${DistToTable},yrange,2].CheckCollision}
            {
                Actor[ExactName,${GuildCraftingStation}]:DoTarget
                wait 10 "${Target.ID}==${Actor[ExactName,${GuildCraftingStation}].ID}"
                if ${Target.Name.Equal[${GuildCraftingStation}]}
                    face
                wait 10
                lastdevice:Set[${ldevicename}]
                return
            }

            ; Try regular table
            if !${Actor[ExactName,${ldevicename},xzrange,${DistToTable},yrange,2].CheckCollision}
            {
                Actor[ExactName,${ldevicename}]:DoTarget
                wait 10 "${Target.ID}==${Actor[${ldevicename}].ID}"
                if ${Target.Name.Equal[${ldevicename}]}
                    face
                wait 10
                lastdevice:Set[${ldevicename}]
                return
            }
        }
        else
        {
            ; Not in guild hall, simple target
            Actor[ExactName,${ldevicename}]:DoTarget
            wait 10 "${Target.ID}==${Actor[${ldevicename}].ID}"
            if ${Target.Name.Equal[${ldevicename}]}
                face
            wait 10
            lastdevice:Set[${ldevicename}]
            return
        }
    }

    ; Strategy 2: Navigate to device region
    if ${Nav.RegionExists[${devicename} ${devicenum}]}
        Nav:MoveToRegion["${devicename} ${devicenum}"]
}
```

**Key Techniques:**
- Check for collision before targeting (`CheckCollision`)
- Try multiple name variations (Elaborate, Guild, regular)
- Use exact name matching (`ExactName`)
- Verify target after DoTarget
- Face the target after acquiring it

### Pattern: Broker/Vendor Targeting

Special handling for NPCs with guild tags:

```lavishscript
if ${devicename.Equal[Broker]}
{
    ; Try tradeskill instance broker
    if ${Actor[xzrange,10,yrange,2,guild,${Broker_GuildTag}].Name(exists)} && ${Actor[xzrange,10,yrange,2,guild,${Broker_GuildTag}].Type.Equal[NoKill NPC]}
    {
        Actor[xzrange,10,yrange,2,guild,${Broker_GuildTag}]:DoTarget
        wait 10 "${Target.ID}==${Actor[xzrange,10,yrange,2,guild,${Broker_GuildTag}].ID}"
        if ${Target.Guild.Equal[${Broker_GuildTag}]}
            face
        wait 10

        ; Move closer if needed
        if ${Target.Distance} > 7
        {
            press "${Nav.AUTORUN}"
            do
            {
                waitframe
            }
            while ${Target.Distance} > 7
            wait 1
            press "${Nav.AUTORUN}"
            wait 2
        }

        lastdevice:Set[Broker]
        return
    }

    ; Try guild hall broker
    elseif ${Actor[xzrange,10,yrange,2,guild,${Broker_GuildTag2}].Name(exists)} && ${Actor[xzrange,10,yrange,2,guild,${Broker_GuildTag2}].Type.Equal[NoKill NPC]}
    {
        ; Similar logic with ${Broker_GuildTag2}
    }
}
```

---

## Writ Automation

### Pattern: Writ Quest Acceptance

Automate accepting tradeskill writs:

```lavishscript
function GetWrit()
{
    variable int tcount
    variable string guildtag

    ; Hide UI elements
    UIElement[Craft Selection].FindUsableChild[Create Work Order,commandbutton]:Hide
    UIElement[Craft Selection].FindUsableChild[Create Rush Order,commandbutton]:Hide

    ; Navigate to writ agent
    if ${ISXEQ2.GetCustomVariable[SRO,bool]}
    {
        call MovetoDevice "RushOrder"
        guildtag:Set[Rush Orders]
        wait 2
        if ${Target.ID} <= 0
            Actor[xzrange,10,yrange,2,guild,${RushOrder_GuildTag}]:DoTarget
    }
    else
    {
        call MovetoDevice "WorkOrder"
        guildtag:Set[Work Orders]
        wait 2
        if ${Target.ID} <= 0
            Actor[xzrange,10,yrange,2,guild,${WorkOrder_GuildTag}]:DoTarget
    }

    if ${Target.ID} <= 0
    {
        ErrorEcho "Unable to find writ agent"
        Script:End
    }

    ; Hail the agent
    Target:DoubleClick
    wait 15

    ; Navigate conversation
    if !${EQ2UIPage[ProxyActor,Conversation].Child[composite,replies].Child[1].GetProperty[LocalText].Left[12].Equal[${WritInitialConvo}]}
    {
        EQ2UIPage[ProxyActor,Conversation].Child[composite,replies].Child[1]:LeftClick
        wait 6
        Target:DoubleClick
        wait 15
    }

    ; Select writ tier
    if ${Tier} <= 0
        Tier:Set[1]

    EQ2UIPage[ProxyActor,Conversation].Child[composite,replies].Child[${Tier}]:LeftClick
    wait 10

    ; Accept quest
    if ${RewardWindow(exists)}
        call AcceptQuest
    else
        wait 40

    if ${RewardWindow(exists)}
        call AcceptQuest
    else
    {
        ErrorEcho "Failed to accept writ quest"
        Script:End
    }

    wait 15

    ; Close conversation
    EQ2UIPage[ProxyActor,Conversation].Child[composite,replies].Child[1]:LeftClick
    wait 15
    EQ2UIPage[ProxyActor,Conversation].Child[composite,replies].Child[1]:LeftClick
    wait 6

    ; Move to invoice desk
    call MovetoDevice "Invoices"
    wait 5

    ; Find and click work order clipboard/desk
    if ${Actor[xzrange,10,${WorkOrderClipboard}].Name(exists)}
        Actor[${WorkOrderClipboard}]:DoubleClick
    elseif ${Actor[xzrange,10,${WorkOrdersDesk}].Name(exists)}
        Actor[${WorkOrdersDesk}]:DoubleClick
    else
        Actor[${Desk}]:DoubleClick

    wait 20
    wait 30 !${Me.CastingSpell}
    FlushQueued

    if !${ReplyDialog(exists)}
        call WaitForResume "Unable to find order clipboard"

    ; Pick first option
    ReplyDialog:Choose[1]
    wait 8

    EQ2Execute /close_top_window

    WritTrigger:Set[TRUE]
    wait 100 ${GotWrit}
}

function AcceptQuest()
{
    if ${RewardWindow.Child[text,Title].GetProperty[localtext].Find["New Quest"]}
    {
        if ${EQ2UIPage[PopUp,RewardPack].Child[button,RewardPack.ButtonComposite.Accept](exists)}
            EQ2UIPage[PopUp,RewardPack].Child[button,RewardPack.ButtonComposite.Accept]:LeftClick
        else
            EQ2UIPage[PopUp,RewardPack].Child[button,RewardPack.Accept]:LeftClick
    }
    else
    {
        echo "ERROR - Expected 'New Quest' title but got '${RewardWindow.Child[text,Title].GetProperty[localtext]}'"
    }
}
```

### Pattern: Writ Tracking

Track writ completion:

```lavishscript
variable bool GotWrit = FALSE
variable bool WritTrigger = FALSE
variable int WritCount
variable int WritQty

; In main loop after crafting
if ${GotWrit}
{
    if ${RewardWindow(exists)}
    {
        wait 5
        RewardWindow:AcceptReward
    }

    Craft:ClearAllRecipes[0]

    ; Decrement writ counter
    ISXEQ2:SetCustomVariable[CWC,${Math.Calc[${ISXEQ2.GetCustomVariable["CWC",int]}-1]}]

    if ${ISXEQ2.GetCustomVariable["CWC",int]} < 1
    {
        startrecipe:Set[FALSE]
        ISXEQ2:SetCustomVariable[SRO,0]
        ISXEQ2:SetCustomVariable[SWO,0]

        ChatEcho "No more Writs to Make!"
        ISXEQ2:SetCustomVariable[CWC,-1]
        UIElement[Craft Selection].FindUsableChild[Writs Remaining,text]:SetText[""]
    }
    else
    {
        ChatEcho "Remaining Writs: ${ISXEQ2.GetCustomVariable["CWC",int]}"
        UIElement[Craft Selection].FindUsableChild[Writs Remaining,text]:SetText["${ISXEQ2.GetCustomVariable["CWC",int]} Writs remaining"]
    }
}
```

---

## Durability and Quality Management

### Pattern: Durability Calculation

Track durability changes to optimize reaction arts:

```lavishscript
variable int TotalDurability
variable int CurrentDurability
variable int ChangeinDur
variable bool chktotdur = TRUE

; On first round
if ${chktotdur}
{
    TotalDurability:Set[${Math.Calc[${CurrentDurability}-${ChangeinDur}]}]
    chktotdur:Set[FALSE]
}

; Calculate durability percentage
variable int tempdur
tempdur:Set[((${CurrentDurability}/${TotalDurability}*100)-80)/20*100]

; Use durability to decide which arts to cast
if ${tempdur} > ${Durability[${tempvar2},2]}
{
    ; High durability - use tier 1 arts
    call CastReaction 1 ${counter}
}
else
{
    ; Low durability - use tier 2 arts
    call CastReaction 2 ${counter}
}

; Check if we need emergency durability boost
if ${tempdur} < ${Durability[${tempvar2},1]} && ${Me.Power} > ${Durability[${tempvar2},3]}
{
    call CastReaction 2 3  ; Emergency durability art
}
```

### Pattern: Quality Targeting

Stop crafting when target quality is reached:

```lavishscript
variable int QualityResult  ; Target quality (1-4)
variable int CurrentQuality

; Main crafting loop
while ${CurrentQuality} < ${QualityResult}
{
    if ${roundstart} && !${complete}
    {
        ; Process reaction arts
        call ProcessArts
    }

    if ${complete}
        break

    waitframe
}

; Stop the craft when quality is achieved
if !${complete} && ${CurrentQuality}
{
    EQ2UIPage[Tradeskills,Tradeskills].Child[button,Tradeskills.TabPages.Craft.Create.Stop]:LeftClick
    wait 15
}
```

---

## Reaction Arts System

### Pattern: LavishSettings-Based Reaction Arts

Store reaction art mappings in XML:

**ReactionArts.xml:**
```xml
<InnerSpaceSettings>
    <Set Name="Reaction Arts">
        <Set Name="Artisan">
            <Setting Name="Finished Edge">1</Setting>
            <Setting Name="Strong Grain">2</Setting>
            <Setting Name="Weak Grain">3</Setting>
        </Set>
        <Set Name="Scholar">
            <Setting Name="Precise Folds">1</Setting>
            <Setting Name="Straight Grain">2</Setting>
            <Setting Name="Ragged Edge">3</Setting>
        </Set>
    </Set>
</InnerSpaceSettings>
```

**Loading and using:**
```lavishscript
variable settingsetref ReactionArts
variable string CurrentReactive

function main()
{
    ; Load reaction arts XML
    LavishSettings:AddSet[Reaction Arts]
    LavishSettings[Reaction Arts]:Import["${RecipePath}/ReactionArts.xml"]
    ReactionArts:Set[${LavishSettings[Reaction Arts].FindSet[Reaction Arts]}]
}

; In crafting loop
if ${roundstart}
{
    ; Look up counter for this reaction
    counter:Set[${LavishSettings[Reaction Arts].FindSet[${MakeKnowledge[${var1},${var2}]}].FindSetting[${CurrentReactive},0]}]

    if ${counter}
    {
        ; Counter found - cast appropriate arts
        if ${counter} == 1
        {
            call CastReaction 1 1  ; Counter type 1
            call CastReaction 1 2  ; Follow-up
        }
        elseif ${counter} == 2
        {
            call CastReaction 1 2  ; Counter type 2
            call CastReaction 1 1  ; Follow-up
        }
        elseif ${counter} == 3
        {
            call CastReaction 1 1  ; Counter type 3
            call CastReaction 1 2  ; Multiple arts
        }
    }
    else
    {
        ; Unknown reaction - log it
        if ${CurrentReactive.Length}
        {
            EQ2Echo [${Time.Date} ${Time}] Unknown Reaction Art - ${CurrentReactive}...Knowledge - ${MakeKnowledge[${var1},${var2}]}\n >> UnknownReactionArts.txt
        }

        ; Fall back to standard arts
        call ProcessArts
    }
}
```

### Pattern: Skill Casting

Cast tradeskill abilities:

```lavishscript
variable string TSSpell[4,3]  ; [tier,slot]

; Load skills for this knowledge
TSSpell[1,1]:Set["${LavishSettings[Skills].FindSet[${craftingknowledge}].FindSetting[1 1]}"]
TSSpell[1,2]:Set["${LavishSettings[Skills].FindSet[${craftingknowledge}].FindSetting[1 2]}"]
TSSpell[1,3]:Set["${LavishSettings[Skills].FindSet[${craftingknowledge}].FindSetting[1 3]}"]
TSSpell[2,1]:Set["${LavishSettings[Skills].FindSet[${craftingknowledge}].FindSetting[2 1]}"]
TSSpell[2,2]:Set["${LavishSettings[Skills].FindSet[${craftingknowledge}].FindSetting[2 2]}"]
TSSpell[2,3]:Set["${LavishSettings[Skills].FindSet[${craftingknowledge}].FindSetting[2 3]}"]

function CastReaction(int tier, int slot)
{
    if ${TSSpell[${tier},${slot}].Length} == 0
        return

    ; Wait for previous cast to finish
    while ${Me.CastingSpell}
        wait 1

    ; Cast the ability
    EQ2Execute /useability "${TSSpell[${tier},${slot}]}"

    ; Wait for cast to start
    wait 2

    ; Wait for cast to finish
    while ${Me.CastingSpell}
        wait 1
}

function ProcessArts()
{
    ; Cast standard progression arts
    call CastReaction 1 1
    call CastReaction 1 2
    call CastReaction 1 3
}
```

---

## Progress Tracking and Statistics

### Pattern: Statistics Tracking

Track crafting statistics:

```lavishscript
variable string StatFuelNme[200]
variable int StatFuelCnt[200]
variable int StatFuelTot[200]
variable int fuelcnt

variable string StatResNme[200]
variable int StatResCnt[200]
variable int StatResTot[200]
variable int rescnt

variable string StatCraftNme[200]
variable int StatCraftCnt[200,4]  ; [index,quality]
variable int StatCraftTot[2,200]
variable int craftcnt

; Update statistics after crafting
function UpdateStatistics(string recipename, int quality, int quantity)
{
    variable int i
    variable bool found = FALSE

    ; Find existing entry
    for (i:Set[1]; ${i} <= ${craftcnt}; i:Inc)
    {
        if ${StatCraftNme[${i}].Equal[${recipename}]}
        {
            StatCraftCnt[${i},${quality}]:Inc[${quantity}]
            StatCraftTot[1,${i}]:Inc[${quantity}]
            found:Set[TRUE]
            break
        }
    }

    ; Create new entry if not found
    if !${found}
    {
        craftcnt:Inc
        StatCraftNme[${craftcnt}]:Set[${recipename}]
        StatCraftCnt[${craftcnt},${quality}]:Set[${quantity}]
        StatCraftTot[1,${craftcnt}]:Set[${quantity}]
    }
}

; Display statistics
function DisplayStatistics()
{
    variable int i

    echo "=== Crafting Statistics ==="
    echo "Recipes Crafted:"
    for (i:Set[1]; ${i} <= ${craftcnt}; i:Inc)
    {
        echo "  ${StatCraftNme[${i}]}: ${StatCraftTot[1,${i}]} total"
        echo "    Q1: ${StatCraftCnt[${i},1]} | Q2: ${StatCraftCnt[${i},2]} | Q3: ${StatCraftCnt[${i},3]} | Q4: ${StatCraftCnt[${i},4]}"
    }
}
```

### Pattern: Progress UI Updates

Update UI with current progress:

```lavishscript
objectdef EQ2Craft
{
    method UpdateProgress(int var1, int var2)
    {
        ; Update the current recipe's quantity
        MakeQty[${var1},${var2}]:Dec

        ; Update total queued counter
        call UpdateTotal

        ; Update UI
        This:ProgressGUI
    }

    method ProgressGUI()
    {
        ; Update progress display
        UIElement[Craft Selection].FindUsableChild[Current Recipe,text]:SetText[${MakeName[${xvar},${xval}]}]
        UIElement[Craft Selection].FindUsableChild[Recipe Count,text]:SetText[${MakeQty[${xvar},${xval}].Ceil}]
        UIElement[Craft Selection].FindUsableChild[Total Queued,text]:SetText[${TotalQueued}]
    }
}

atom UpdateTotal()
{
    variable int xvar
    variable int xval

    TotalQueued:Set[0]
    xvar:Set[2]

    do
    {
        xval:Set[${MakeCnt[${xvar}]}]
        do
        {
            if ${MakeCnt[${xvar}]} && ${MakeQty[${xvar},${xval}]} > 0 && ${MakeName[${xvar},${xval}].Length}
            {
                TotalQueued:Inc
            }
        }
        while ${xval:Dec} > 0
    }
    while ${xvar:Dec} > 0
}
```

---

## File-Based Configuration

### Pattern: Character-Specific Config

Save/load character-specific settings:

```lavishscript
variable settingsetref Configuration
variable filepath ConfigPath = "${LavishScript.HomeDirectory}/Scripts/EQ2Craft/Character Config/"
variable string configfile

function main()
{
    ; Build config filename
    configfile:Set["${ConfigPath}${Me.Name}_${EQ2.ServerName}.xml"]

    ; Load configuration
    LavishSettings:AddSet[Craft Config File]
    LavishSettings[Craft Config File]:Import[${configfile}]
    Configuration:Set[${LavishSettings[Craft Config File].FindSet[Craft Config File]}]

    ; Read settings
    WritCount:Set[${Configuration.FindSetting[How many Writs to create per craft session?]}]
    WritQty:Set[${Configuration.FindSetting[How many Recipes per Writ?]}]
    CampTimer:Set[${Configuration.FindSetting[Camp out after a specified time has elapsed for a crafting session?]}]
    PowerRegen:Set[${Configuration.FindSetting[Amount of Power to Regenerate before crafting a recipe?]}]

    ; ... rest of script
}

function SaveConfiguration()
{
    ; Update settings
    Configuration.FindSetting[How many Writs to create per craft session?]:Set[${WritCount}]
    Configuration.FindSetting[How many Recipes per Writ?]:Set[${WritQty}]
    Configuration.FindSetting[Camp out after a specified time has elapsed for a crafting session?]:Set[${CampTimer}]
    Configuration.FindSetting[Amount of Power to Regenerate before crafting a recipe?]:Set[${PowerRegen}]

    ; Export to file
    LavishSettings[Craft Config File]:Export[${configfile}]
}
```

**Example Config XML:**
```xml
<InnerSpaceSettings>
    <Set Name="Craft Config File">
        <Setting Name="How many Writs to create per craft session?">5</Setting>
        <Setting Name="How many Recipes per Writ?">6</Setting>
        <Setting Name="Camp out after a specified time has elapsed for a crafting session?">60</Setting>
        <Setting Name="Amount of Power to Regenerate before crafting a recipe?">80</Setting>
        <Setting Name="Time to wait between combines?">2</Setting>
        <Setting Name="Pather Precision">2.0</Setting>
    </Set>
</InnerSpaceSettings>
```

### Pattern: Global Data Files

Load shared data files:

```lavishscript
variable filepath RecipePath = "${LavishScript.HomeDirectory}/Scripts/EQ2Craft/Recipe Data/"
variable settingsetref VendorBought
variable settingsetref Harvests
variable settingsetref Rares

function LoadGlobalData()
{
    ; Load common resources
    LavishSettings:AddSet[Common Resources]
    LavishSettings[Common Resources]:Import["${RecipePath}/Common.xml"]
    VendorBought:Set[${LavishSettings[Common Resources].FindSet[Common Resources]}]

    ; Load harvest resources
    LavishSettings:AddSet[Harvest Resources]
    LavishSettings[Harvest Resources]:Import["${RecipePath}/Resources.xml"]
    Harvests:Set[${LavishSettings[Harvest Resources].FindSet[Harvest Resources]}]

    ; Load rare resources
    LavishSettings:AddSet[Rare Resources]
    LavishSettings[Rare Resources]:Import["${RecipePath}/Resources.xml"]
    Rares:Set[${LavishSettings[Rare Resources].FindSet[Rare Resources]}]
}

; Use the data
function IsVendorBought(string itemname)
{
    if ${VendorBought.FindSetting[${itemname}](exists)}
        return TRUE
    return FALSE
}
```

---

## Complete Working Example

### Simple Auto-Crafter

A simplified example demonstrating key patterns:

```lavishscript
; SimpleCraft.iss - Simplified crafting automation
; Usage: run SimpleCraft "Recipe Name" 10

#include "${LavishScript.HomeDirectory}/Scripts/EQ2Navigation/EQ2Nav_Lib.iss"

variable EQ2Nav Nav
variable int CurrentQuality = 0
variable int TargetQuality = 4
variable bool roundstart = FALSE
variable bool complete = FALSE
variable string craftingrecipe
variable string craftingknowledge
variable string TSSpell[2,3]

function main(string recipename, int quantity)
{
    if !${ISXEQ2.IsReady}
    {
        echo "ISXEQ2 not ready!"
        Script:End
    }

    if !${recipename.Length}
    {
        echo "Usage: run SimpleCraft \"Recipe Name\" quantity"
        Script:End
    }

    ; Initialize navigation
    Nav:UseLSO[FALSE]
    Nav:LoadMap

    ; Initialize events
    Event[EQ2_onIncomingText]:AttachAtom[OnIncomingText]

    echo "=== SimpleCraft ==="
    echo "Recipe: ${recipename}"
    echo "Quantity: ${quantity}"

    ; Verify recipe exists
    if !${Me.Recipe[${recipename}](exists)}
    {
        echo "You don't know the recipe: ${recipename}"
        Script:End
    }

    ; Get recipe info
    variable recipe myrecipe
    myrecipe:Set[${Me.Recipe[${recipename}]}]

    ; Wait for recipe info
    if !${myrecipe.IsRecipeInfoAvailable}
    {
        echo "Loading recipe info..."
        wait 50 ${myrecipe.IsRecipeInfoAvailable}
    }

    if !${myrecipe.IsRecipeInfoAvailable}
    {
        echo "Failed to load recipe info!"
        Script:End
    }

    ; Navigate to device
    call NavigateToDevice "${myrecipe.ToRecipeInfo.Device}"

    ; Load skills
    craftingknowledge:Set[${myrecipe.Knowledge}]
    call LoadSkills

    ; Craft the items
    variable int i
    for (i:Set[1]; ${i} <= ${quantity}; i:Inc)
    {
        echo "Crafting ${i}/${quantity}..."
        call CraftOne ${myrecipe.ID}

        ; Wait between combines
        wait 20
    }

    echo "=== Crafting Complete ==="
    Script:End
}

function NavigateToDevice(string devicename)
{
    echo "Moving to ${devicename}..."

    ; Map device names to regions
    switch ${devicename}
    {
        case Forge
            Nav:MoveToRegion["Forge 1"]
            break
        case Stove and Keg
            Nav:MoveToRegion["Stove & Keg 1"]
            break
        case Work Bench
            Nav:MoveToRegion["Work Bench 1"]
            break
        default
            echo "Unknown device: ${devicename}"
            Script:End
    }

    ; Wait for navigation to complete
    do
    {
        Nav:Pulse
        wait 0
    }
    while ${Nav.Moving}

    echo "Arrived at ${devicename}"
}

function LoadSkills()
{
    ; Load skill data from file
    LavishSettings:AddSet[Skills]
    LavishSettings[Skills]:Import["${LavishScript.HomeDirectory}/Scripts/EQ2Craft/Recipe Data/Skills.xml"]

    ; Load skills for this knowledge type
    TSSpell[1,1]:Set["${LavishSettings[Skills].FindSet[${craftingknowledge}].FindSetting[1 1]}"]
    TSSpell[1,2]:Set["${LavishSettings[Skills].FindSet[${craftingknowledge}].FindSetting[1 2]}"]
    TSSpell[1,3]:Set["${LavishSettings[Skills].FindSet[${craftingknowledge}].FindSetting[1 3]}"]
}

function CraftOne(int64 recipeID)
{
    ; Reset state
    complete:Set[FALSE]
    roundstart:Set[FALSE]
    CurrentQuality:Set[0]

    ; Start the craft
    EQ2Execute /createfromrecipe ${recipeID}
    wait 15

    ; Click Begin
    if ${EQ2UIPage[Tradeskills,Tradeskills].Child[button,Tradeskills.TabPages.Craft.Create.Begin](exists)}
    {
        EQ2UIPage[Tradeskills,Tradeskills].Child[button,Tradeskills.TabPages.Craft.Create.Begin]:LeftClick
        wait 10
    }

    ; Wait for roundstart event
    wait 50 ${roundstart}

    if !${roundstart}
    {
        echo "Failed to start crafting!"
        return
    }

    ; Main crafting loop
    variable int timeout = ${Time.Timestamp}
    while ${CurrentQuality} < ${TargetQuality} && !${complete}
    {
        if ${roundstart}
        {
            roundstart:Set[FALSE]

            ; Cast arts
            call CastArt 1 1
            call CastArt 1 2
            call CastArt 1 3
        }

        ; Check timeout
        if ${Time.Timestamp} - ${timeout} > 30
        {
            echo "Crafting timeout!"
            break
        }

        waitframe
    }

    ; Stop when quality reached
    if ${CurrentQuality} >= ${TargetQuality} && !${complete}
    {
        if ${EQ2UIPage[Tradeskills,Tradeskills].Child[button,Tradeskills.TabPages.Craft.Create.Stop](exists)}
        {
            EQ2UIPage[Tradeskills,Tradeskills].Child[button,Tradeskills.TabPages.Craft.Create.Stop]:LeftClick
            wait 10
        }
    }

    ; Wait for completion
    wait 50 ${complete}
}

function CastArt(int tier, int slot)
{
    if ${TSSpell[${tier},${slot}].Length} == 0
        return

    ; Wait for previous cast
    while ${Me.CastingSpell}
        wait 1

    ; Cast the ability
    EQ2Execute /useability "${TSSpell[${tier},${slot}]}"
    wait 2

    ; Wait for cast to finish
    while ${Me.CastingSpell}
        wait 1
}

atom OnIncomingText(string Text, string ChannelName, string SpeakerName)
{
    ; Detect round start
    if ${Text.Find["The round has started"]}
    {
        roundstart:Set[TRUE]
    }

    ; Detect completion
    if ${Text.Find["You have created"]} || ${Text.Find["You have failed"]}
    {
        complete:Set[TRUE]
    }

    ; Detect quality increase
    if ${Text.Find["pristine"]}
        CurrentQuality:Set[4]
    elseif ${Text.Find["mastercraft"]}
        CurrentQuality:Set[3]
    elseif ${Text.Find["handcrafted"]}
        CurrentQuality:Set[2]
    elseif ${Text.Find["artisan"]}
        CurrentQuality:Set[1]
}
```

**Usage:**
```
run SimpleCraft "Iron Chainmail Chestguard" 10
```

**Skills.xml Example:**
```xml
<InnerSpaceSettings>
    <Set Name="Skills">
        <Set Name="Armorer">
            <Setting Name="1 1">Smelt</Setting>
            <Setting Name="1 2">Shape Metal</Setting>
            <Setting Name="1 3">Refine Metal</Setting>
        </Set>
        <Set Name="Weaponsmith">
            <Setting Name="1 1">Forge</Setting>
            <Setting Name="1 2">Temper</Setting>
            <Setting Name="1 3">Sharpen</Setting>
        </Set>
    </Set>
</InnerSpaceSettings>
```

---

## Summary

### Key Patterns Learned

1. **Command-Line Parsing**: Flexible script invocation with multiple options
2. **Navigation Integration**: EQ2Nav integration with region validation
3. **Localization**: Server-based multi-language support
4. **UI State Management**: Complex mode switching and visibility control
5. **Queue Management**: Multi-dimensional arrays for recipe queues
6. **Window Monitoring**: UI element detection for state tracking
7. **Event-Driven Loop**: Trigger-based crafting automation
8. **Device Targeting**: Multi-fallback actor search strategies
9. **Writ Automation**: Complete quest acceptance and tracking
10. **Durability/Quality**: Calculated art selection based on metrics
11. **Reaction Arts**: LavishSettings-based art mapping system
12. **Statistics**: Comprehensive tracking and reporting
13. **File Configuration**: Character-specific and global data files

### Best Practices

- ✅ Always validate arguments and provide usage help
- ✅ Use navigation libraries instead of manual movement
- ✅ Support multiple languages with localization
- ✅ Store data in external XML files for easy updates
- ✅ Use multi-dimensional arrays for complex data
- ✅ Implement timeouts for all wait conditions
- ✅ Provide multiple fallback strategies for actor targeting
- ✅ Track statistics for debugging and optimization
- ✅ Use events for state change detection
- ✅ Separate configuration from code

### Related Documentation

- [14_Advanced_Scripting_Patterns.md](14_Advanced_Scripting_Patterns.md) - LavishSettings, Navigation
- [05_Patterns_And_Best_Practices.md](05_Patterns_And_Best_Practices.md) - General best practices
- [03_API_Reference.md](03_API_Reference.md) - Recipe, Me, Actor datatypes

---

*Part of the ISXEQ2 Scripting Guide - Advanced Patterns Series*
*Based on EQ2Craft v9.5 - https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Craft*
