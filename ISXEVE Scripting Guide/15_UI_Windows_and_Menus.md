# UI Windows and Menus
## Complete Guide to EVE UI Interaction via ISXEVE

**Part of**: EVE Online Bot Development - AI-Level Documentation
**Layer**: 3 - ISXEVE API Deep Dive
**Prerequisites**: Read files 01-08 (Foundation + Scripting Fundamentals)
**Critical For**: All bot tasks involving UI (market, inventory, contracts, stations, fitting, etc.)

---

## Purpose of This Document

This file provides **exhaustive coverage** of:
- EVE's UI window system architecture
- EVEWindow object hierarchy and methods
- EVE:Execute command (the primary UI interaction mechanism)
- Common UI commands and patterns
- Window finding, validation, and navigation
- UI state checking and waiting patterns
- Click simulation and menu interaction
- Real-world patterns from Evebot, Yamfa, and Tehbot
- Critical gotchas and timing issues

**Why This Is CRITICAL**:
- **95% of bot actions require UI interaction** (inventory, market, contracts, fitting, etc.)
- EVE:Execute is the **ONLY** way to trigger UI actions from scripts
- UI timing and state validation are **major sources of bot failures**
- Example scripts have **significant UI bugs** (race conditions, missing waits)

---

## Table of Contents

1. [EVE UI System Overview](#eve-ui-system-overview)
2. [EVEWindow Object](#evewindow-object)
3. [EVE:Execute Command](#eveexecute-command)
4. [Common UI Commands Reference](#common-ui-commands-reference)
5. [Window Finding and Validation](#window-finding-and-validation)
6. [Opening and Closing Windows](#opening-and-closing-windows)
7. [Clicking and Button Interaction](#clicking-and-button-interaction)
8. [Menu System Navigation](#menu-system-navigation)
9. [Inventory Window Interaction](#inventory-window-interaction)
10. [Market Window Interaction](#market-window-interaction)
11. [Station Services](#station-services)
12. [UI Timing and Wait Patterns](#ui-timing-and-wait-patterns)
13. [UI State Validation](#ui-state-validation)
14. [Common Patterns from Example Scripts](#common-patterns-from-example-scripts)
15. [Critical Gotchas and Issues](#critical-gotchas-and-issues)
16. [Anti-Patterns to Avoid](#anti-patterns-to-avoid)

---

## EVE UI System Overview

### How EVE's UI Works

**Key Concepts**:
1. **Everything is a window** - Inventory, market, ship hangar, even the HUD elements
2. **Windows have names** - Usually descriptive (e.g., "inventory", "market", "fitting")
3. **Windows have unique IDs** - Multiple instances of same window type (e.g., multiple cargo containers)
4. **EVE:Execute triggers actions** - Cannot directly click/manipulate UI, must use Execute commands
5. **UI is asynchronous** - Commands don't complete instantly, must wait for state changes
6. **Server authority** - Many UI actions require server round-trip (delays)

### Window Types

**Common Window Types**:
- `inventory` - Inventory/cargo windows
- `market` - Market window
- `fitting` - Ship fitting window
- `station` - Station services
- `fleetwindow` - Fleet window
- `contractswindow` - Contracts
- `charactersheet` - Character sheet
- `overview` - Overview (target list)
- `selected` - Selected item window

**Window Hierarchy**:
```
EVE (top-level UI system)
└── Windows (collection of all open windows)
    ├── EVEWindow[Name] - Find by name
    ├── EVEWindow[ID] - Find by ID
    └── Methods to query, open, close
```

### UI Interaction Model

**Script CANNOT directly**:
- Click buttons
- Drag items
- Type in text fields
- Manipulate UI elements

**Script CAN**:
- Execute UI commands via `EVE:Execute[CommandName, parameters]`
- Query window state via `EVEWindow` objects
- Check if windows are open
- Get window positions/sizes
- Wait for UI state changes

---

## EVEWindow Object

### Getting EVEWindow Objects

**By Name**:
```lavish
variable evwindow inventoryWin = ${EVEWindow[inventory]}

if ${inventoryWin(exists)}
{
    echo "Inventory window exists"
}
```

**By Partial Name (First Match)**:
```lavish
; Finds first window with "invent" in name
variable evwindow win = ${EVEWindow[invent]}
```

**By ID**:
```lavish
variable int64 windowID = 123456789
variable evwindow win = ${EVEWindow[${windowID}]}
```

**All Windows**:
```lavish
; Get count of all open windows
variable int count = ${EVEWindow.Count}

; Iterate all windows
variable int i
for (i:Set[1]; ${i} <= ${EVEWindow.Count}; i:Inc)
{
    echo "Window ${i}: ${EVEWindow[${i}].Name}"
}
```

### EVEWindow Members

**Common Members**:
```lavish
variable evwindow win = ${EVEWindow[inventory]}

; Basic info
echo "Name: ${win.Name}"
echo "ID: ${win.ID}"
echo "Text: ${win.Text}"         ; Window title text
echo "Valid: ${win(exists)}"

; State
echo "Minimized: ${win.Minimized}"
echo "Visible: ${win.Visible}"

; Position/size
echo "X: ${win.X}"
echo "Y: ${win.Y}"
echo "Width: ${win.Width}"
echo "Height: ${win.Height}"
```

**Child Windows**:
```lavish
; Get child windows (e.g., tabs within inventory)
variable int childCount = ${win.ChildCount}

variable int i
for (i:Set[1]; ${i} <= ${childCount}; i:Inc)
{
    echo "Child ${i}: ${win.Child[${i}].Name}"
}
```

**Buttons**:
```lavish
; Get button count
echo "Buttons: ${win.ButtonCount}"

; Get button by index
echo "Button 1: ${win.Button[1].Text}"

; Check if button exists by text
if ${win.Button["OK"](exists)}
{
    echo "OK button exists"
}
```

### EVEWindow Methods

**Common Methods**:
```lavish
; Close window
win:Close

; Minimize window
win:Minimize

; Maximize window
win:Maximize

; Click button by text
win:ClickButton["OK"]

; Click button by index
win:ClickButton[1]

; Set window position
win:SetPosition[100, 200]

; Set window size
win:SetSize[400, 300]
```

### Window Existence Checking

**CRITICAL PATTERN** - Always check if window exists before using:

```lavish
variable evwindow win = ${EVEWindow[inventory]}

if !${win(exists)}
{
    echo "ERROR: Inventory window not found"
    return FALSE
}

; Safe to use win now
echo "Inventory: ${win.Name}"
```

**Why This Matters**:
- Attempting to use non-existent window members/methods = script crash
- Windows can close unexpectedly (player action, server disconnect, etc.)
- Window names can change between EVE versions

---

## EVE:Execute Command

### Overview

**EVE:Execute** is the **primary mechanism** for all UI interaction in ISXEVE.

**Syntax**:
```lavish
EVE:Execute[CommandName, param1, param2, ...]
```

**Key Rules**:
1. Commands are **case-insensitive** (CmdOpenInventory = cmdopeninventory)
2. Parameters vary by command (some have none, some require IDs/names)
3. **Execution is asynchronous** - command returns immediately, UI updates later
4. **No return value** - Cannot tell if command succeeded from Execute call
5. Must **validate result** by checking UI state after wait

### Common Execute Commands Categories

**Opening Windows**:
- `CmdOpenInventory`
- `CmdOpenMarket`
- `CmdOpenFitting`
- `CmdOpenStationPanel`
- `CmdOpenHangarFloor`
- `CmdOpenCargoHold`

**Ship/Module Control**:
- `CmdActivateModule`
- `CmdDeactivateModule`
- `CmdTogglePropulsionMod`
- `CmdStopShip`

**Navigation**:
- `CmdWarpToBookmark`
- `CmdDock`
- `CmdUndock`
- `CmdJumpThroughStargate`

**Item Interaction**:
- `OpenCorpHangar`
- `TakeItems`
- `TrashItems`

**Menu Actions**:
- `CmdShowInfo`
- `CmdLookAtItem`

### Finding Command Names

**Methods**:
1. **Wiki**: `IsxeveWiki/ISXEVE/EVE_(Object_Type).html` - Lists many commands
2. **Trial and Error**: Try logical names (CmdOpen*, Cmd*, Open*)
3. **Example Scripts**: Search Evebot/Yamfa/Tehbot for `EVE:Execute` usage
4. **InnerSpace Console**: Some commands shown in console when you manually use UI

**IMPORTANT**: Wiki documentation is **incomplete**. Many working commands are undocumented. Example scripts are the best reference.

---

## Common UI Commands Reference

### Inventory Commands

**Open Inventory**:
```lavish
EVE:Execute[CmdOpenInventory]
wait 20    ; Wait for window to open
```

**Open Cargo Hold**:
```lavish
EVE:Execute[CmdOpenCargoHold]
wait 20
```

**Open Ship Hangar** (in station):
```lavish
EVE:Execute[CmdOpenHangarFloor]
wait 20
```

**Open Corp Hangar**:
```lavish
; paramter = division number (1-7)
EVE:Execute[OpenCorpHangar, 1]
wait 20
```

**Open Container by ID**:
```lavish
variable int64 containerID = ${Entity[Name = "My Container"].ID}
EVE:Execute[OpenCargoHold, ${containerID}]
wait 20
```

### Module Commands

**Activate Module**:
```lavish
; Parameter = module slot index (0-7 for high, 0-7 for mid, 0-7 for low)
variable int slotID = ${MyShip.Module[=MiningLaser].ToItem.SlotID}
EVE:Execute[CmdActivateModule, ${slotID}]
```

**Deactivate Module**:
```lavish
EVE:Execute[CmdDeactivateModule, ${slotID}]
```

**Toggle Propulsion Mod** (afterburner/MWD):
```lavish
EVE:Execute[CmdTogglePropulsionMod]
```

### Ship Commands

**Stop Ship**:
```lavish
EVE:Execute[CmdStopShip]
```

**Align to Entity/Bookmark**:
```lavish
; Parameter = entity ID or bookmark ID
EVE:Execute[CmdAlignTo, ${entityID}]
```

### Docking/Undocking

**Dock at Station**:
```lavish
variable int64 stationID = ${Entity[Name = "Jita IV - Moon 4"](exists),ID}
EVE:Execute[CmdDock, ${stationID}]
```

**Undock**:
```lavish
EVE:Execute[CmdUndock]
```

**Jump Through Stargate**:
```lavish
variable int64 gateID = ${Entity[CategoryID = 6](exists),ID}    ; Category 6 = stargates
EVE:Execute[CmdJumpThroughStargate, ${gateID}]
```

### Warping

**Warp to Bookmark**:
```lavish
; Need bookmark ID (finding bookmarks is complex, covered in inventory section)
variable int64 bookmarkID = ${EVEWindow[ByCaption,inventory].GetBookmarkID["Mining Site"]}
EVE:Execute[CmdWarpToBookmark, ${bookmarkID}]
```

**Warp to Zero** (on selected entity):
```lavish
; Must have entity selected first
EVE:Execute[CmdWarpToSelection]
```

### Targeting

**Lock Target**:
```lavish
; Entity method is preferred, but can use Execute
variable int64 targetID = ${Entity[...].ID}
EVE:Execute[CmdLockTarget, ${targetID}]
```

**Unlock Target**:
```lavish
EVE:Execute[CmdUnlockTarget, ${targetID}]
```

**Unlock All**:
```lavish
EVE:Execute[CmdUnlockTargets]
```

### Menu/Info Commands

**Show Info Window**:
```lavish
variable int64 itemID = ${Entity[...].ID}
EVE:Execute[CmdShowInfo, ${itemID}]
wait 20    ; Wait for info window
```

**Look At** (camera focus):
```lavish
EVE:Execute[CmdLookAtItem, ${entityID}]
```

---

## Window Finding and Validation

### Finding Windows by Name

**Exact Name**:
```lavish
variable evwindow win = ${EVEWindow[inventory]}

if !${win(exists)}
{
    echo "Inventory window not found"
    return FALSE
}
```

**Partial Name** (First Match):
```lavish
; Matches "inventory", "inventoryPrimary", etc.
variable evwindow win = ${EVEWindow[invent]}
```

**By Caption** (Window Title):
```lavish
; Some windows identified by title text
variable evwindow win = ${EVEWindow[ByCaption,Item Hangar]}
```

### Finding Specific Inventory Windows

**Primary Inventory**:
```lavish
variable evwindow win = ${EVEWindow[inventory]}
```

**Cargo Hold**:
```lavish
; Cargo of YOUR ship
variable evwindow win = ${EVEWindow[MyShipCargo]}

; Alternative
win:Set[${EVEWindow[ByItemID,${MyShip.ID}]}]
```

**Ship Hangar** (in station):
```lavish
variable evwindow win = ${EVEWindow[ShipHangar]}

; Alternative
win:Set[${EVEWindow[ByCaption,Ship Hangar]}]
```

**Corp Hangar Division**:
```lavish
; Find corp hangar division 1
variable evwindow win = ${EVEWindow[ByCaption,Corporation Hangar]}

; Check child windows for specific division
; (This is complex - Evebot has extensive corp hangar code)
```

### Validating Window State

**Check if Open and Ready**:
```lavish
function IsWindowReady(string windowName)
{
    variable evwindow win = ${EVEWindow[${windowName}]}

    if !${win(exists)}
        return FALSE

    if ${win.Minimized}
        return FALSE

    ; Some windows need additional checks
    ; (e.g., market window loaded, inventory items loaded)

    return TRUE
}

; Usage
if ${IsWindowReady["inventory"]}
{
    echo "Inventory ready"
}
```

**Wait for Window to Open**:
```lavish
function WaitForWindow(string windowName, int timeoutSeconds)
{
    variable int startTime = ${Script.RunningTime}
    variable int elapsed

    while TRUE
    {
        if ${EVEWindow[${windowName}](exists)}
        {
            echo "Window ${windowName} opened"
            return TRUE
        }

        elapsed:Set[${Math.Calc[(${Script.RunningTime} - ${startTime}) / 1000]}]
        if ${elapsed} >= ${timeoutSeconds}
        {
            echo "Timeout waiting for ${windowName}"
            return FALSE
        }

        wait 100
    }
}

; Usage
EVE:Execute[CmdOpenInventory]
if !${WaitForWindow["inventory", 10]}
{
    echo "Failed to open inventory"
    return
}
```

---

## Opening and Closing Windows

### Opening Windows Safely

**Pattern**:
1. Check if already open
2. If not, execute open command
3. Wait for window to appear
4. Validate window is ready

**Example**:
```lavish
function OpenInventory()
{
    ; Already open?
    if ${EVEWindow[inventory](exists)}
    {
        echo "Inventory already open"
        return TRUE
    }

    ; Open it
    echo "Opening inventory..."
    EVE:Execute[CmdOpenInventory]

    ; Wait for it
    variable int attempts = 0
    while ${attempts} < 20
    {
        wait 100
        if ${EVEWindow[inventory](exists)}
        {
            echo "Inventory opened successfully"
            return TRUE
        }
        attempts:Inc
    }

    echo "ERROR: Failed to open inventory"
    return FALSE
}
```

### Closing Windows

**Close by EVEWindow Object**:
```lavish
variable evwindow win = ${EVEWindow[inventory]}

if ${win(exists)}
{
    win:Close
    echo "Closed inventory"
}
```

**Close All Windows of Type** (useful for cleanup):
```lavish
function CloseAllWindowsByName(string windowName)
{
    variable int i
    variable int count = ${EVEWindow.Count}

    ; Iterate backwards (closing changes indices)
    for (i:Set[${count}]; ${i} >= 1; i:Dec)
    {
        if ${EVEWindow[${i}].Name.Find[${windowName}](exists)}
        {
            echo "Closing ${EVEWindow[${i}].Name}"
            EVEWindow[${i}]:Close
            wait 10
        }
    }
}

; Usage
call CloseAllWindowsByName "inventory"
```

### Cleanup Pattern (atexit)

```lavish
function atom atexit()
{
    echo "Cleaning up UI..."

    ; Close windows we opened
    if ${EVEWindow[inventory](exists)}
        EVEWindow[inventory]:Close

    if ${EVEWindow[market](exists)}
        EVEWindow[market]:Close

    ; Stop ship (safety)
    if ${Me.InSpace}
        EVE:Execute[CmdStopShip]
}
```

---

## Clicking and Button Interaction

### Clicking Buttons in Windows

**Click by Button Text**:
```lavish
variable evwindow win = ${EVEWindow[inventory]}

if ${win.Button["OK"](exists)}
{
    win:ClickButton["OK"]
    echo "Clicked OK"
}
```

**Click by Button Index**:
```lavish
; Click first button
win:ClickButton[1]
```

**Validate Button Exists Before Clicking**:
```lavish
function ClickButton(string windowName, string buttonText)
{
    variable evwindow win = ${EVEWindow[${windowName}]}

    if !${win(exists)}
    {
        echo "ERROR: Window ${windowName} not found"
        return FALSE
    }

    if !${win.Button[${buttonText}](exists)}
    {
        echo "ERROR: Button '${buttonText}' not found in ${windowName}"
        return FALSE
    }

    win:ClickButton[${buttonText}]
    echo "Clicked '${buttonText}' in ${windowName}"
    return TRUE
}

; Usage
call ClickButton "inventory" "OK"
```

### Finding Buttons

**List All Buttons**:
```lavish
function ListWindowButtons(string windowName)
{
    variable evwindow win = ${EVEWindow[${windowName}]}

    if !${win(exists)}
    {
        echo "Window not found"
        return
    }

    echo "Buttons in ${windowName}:"
    variable int i
    for (i:Set[1]; ${i} <= ${win.ButtonCount}; i:Inc)
    {
        echo "  ${i}: ${win.Button[${i}].Text}"
    }
}

; Usage (for debugging)
call ListWindowButtons "inventory"
```

### Button Click Timing

**CRITICAL**: Button clicks are **asynchronous** and may trigger server actions.

```lavish
; BAD - No wait after click
win:ClickButton["OK"]
; Window might still be open here!

; GOOD - Wait for UI to respond
win:ClickButton["OK"]
wait 50    ; Minimum wait for UI update

; BETTER - Wait and validate
win:ClickButton["OK"]
wait 50

variable int attempts = 0
while ${win(exists)} && ${attempts} < 20
{
    wait 100
    attempts:Inc
}

if ${win(exists)}
{
    echo "WARNING: Window still open after clicking OK"
}
```

---

## Menu System Navigation

### Right-Click Menus (Context Menus)

**IMPORTANT**: ISXEVE has **limited** support for right-click menu interaction.

**Opening Context Menu on Entity** (unreliable):
```lavish
; This is NOT well-supported
; Prefer using direct commands instead
```

**Alternative: Use Direct Commands**:
```lavish
; Instead of right-click -> "Look At"
EVE:Execute[CmdLookAtItem, ${entityID}]

; Instead of right-click -> "Show Info"
EVE:Execute[CmdShowInfo, ${entityID}]

; Instead of right-click -> "Lock Target"
Entity[${entityID}]:LockTarget
```

### Neocom Menu (Left Sidebar)

**Opening Neocom Items**:
```lavish
; Most Neocom items have dedicated Execute commands

; Inventory
EVE:Execute[CmdOpenInventory]

; Market
EVE:Execute[CmdOpenMarket]

; Fitting
EVE:Execute[CmdOpenFitting]

; Character sheet
EVE:Execute[CmdOpenCharactersheet]
```

---

## Inventory Window Interaction

### Inventory Window Structure

**Inventory Hierarchy**:
```
Inventory Window
├── Tree (left side) - Locations
│   ├── Item Hangar
│   ├── Ship Hangar
│   ├── Corp Hangars (1-7)
│   └── Active Ship (cargo, drones, etc.)
└── Content (right side) - Items in selected location
```

### Opening Inventory Locations

**Open Inventory**:
```lavish
EVE:Execute[CmdOpenInventory]
wait 20
```

**Switch to Item Hangar**:
```lavish
; This is complex - inventory locations accessed via ChildWindow
; Example from Evebot:
variable evwindow invWin = ${EVEWindow[inventory]}
variable evwindow itemHangar = ${invWin.ChildWindow[itemHangar]}

if ${itemHangar(exists)}
{
    echo "Item hangar accessible"
}
```

**Open Ship Hangar**:
```lavish
EVE:Execute[CmdOpenHangarFloor]
wait 20
```

**Open Corp Hangar Division**:
```lavish
; Division 1-7
EVE:Execute[OpenCorpHangar, 1]
wait 20
```

### Accessing Inventory Items

**CRITICAL**: Direct item manipulation is **complex** and **fragile**.

**⚠️ WARNING:** Old `MyShip.GetCargo` / `MyShip.Cargo[#]` are DEPRECATED (July 2020). Use modern EVEWindow[Inventory] API.

**Get Items in Cargo** (Modern API - July 2020+):
```lavish
; MODERN API: Use EVEWindow[Inventory].Child[ShipCargo]
if !${EVEWindow[Inventory](exists)}
{
    EVE:Execute[CmdOpenInventory]
    wait 20
}

if ${EVEWindow[Inventory](exists)}
{
    variable index:item CargoItems
    EVEWindow[Inventory].Child[ShipCargo]:GetItems[CargoItems]

    variable iterator Item
    CargoItems:GetIterator[Item]

    if ${Item:First(exists)}
    {
        do
        {
            echo "Item: ${Item.Value.Name} (${Item.Value.Quantity})"
        }
        while ${Item:Next(exists)}
    }
}
```

**Inventory Window Item Access** (Advanced, from Evebot):
```lavish
; This is VERY complex - Evebot has entire modules for inventory management
; Involves ChildWindow access, item iteration, etc.
; DEPRECATED: MyShip.Cargo, MyShip.GetHangarItems (July 2020)
; MODERN: Use EVEWindow[Inventory].Child[name] pattern shown above
```

### Moving Items (Drag/Drop Simulation)

**ISXEVE does NOT support drag/drop**. Must use game's right-click menu or hotkeys.

**Workarounds**:
1. **Use item.MoveTo method** (if item type supports it)
2. **Use EVE:Execute[OpenCargoHold]** to open destination, then item methods
3. **Some Evebot patterns** use complex window/button clicking (fragile)

**Example from Evebot** (simplified):
```lavish
; This is an approximation - real Evebot code is much more complex
function MoveItemToCargoHold(int64 itemID)
{
    ; Get item
    variable item myItem = ${Item[${itemID}]}

    if !${myItem(exists)}
        return FALSE

    ; Use MoveTo if available (not always available)
    if ${myItem.MoveTo(exists)}
    {
        myItem:MoveTo[${MyShip.ID}, CargoHold]
        wait 50
        return TRUE
    }

    ; Otherwise, must use UI clicking (complex, fragile)
    echo "Cannot move item - no MoveTo method"
    return FALSE
}
```

---

## Market Window Interaction

### Opening Market

```lavish
EVE:Execute[CmdOpenMarket]
wait 50    ; Market window is slow to load
```

**Validate Market Loaded**:
```lavish
function IsMarketReady()
{
    variable evwindow market = ${EVEWindow[market]}

    if !${market(exists)}
        return FALSE

    ; Additional checks (market data loaded, etc.) would go here
    ; Evebot has complex market validation

    return TRUE
}
```

### Market Commands

**IMPORTANT**: Market interaction via ISXEVE is **extremely limited** and **fragile**.

**Why Market Bots are Difficult**:
- Market window has complex internal structure (tabs, filters, item list, etc.)
- Few dedicated Execute commands for market actions
- Clicking specific items in market list is unreliable
- Price data access is limited

**What Example Scripts Do**:
- Evebot has market modules, but they're **complex** and **fragile**
- Most avoid market automation entirely
- Prefer using API/web for market data (outside ISXEVE scope)

**Basic Market Pattern** (from Evebot, simplified):
```lavish
function OpenMarketForItem(string itemName)
{
    ; Open market
    if !${EVEWindow[market](exists)}
    {
        EVE:Execute[CmdOpenMarket]
        wait 100
    }

    ; Beyond this, item searching/filtering is very complex
    ; and not well-supported by ISXEVE

    echo "Market opened, but item search is not automated"
}
```

---

## Station Services

### Opening Station Panel

```lavish
; In station only
if ${Me.InStation}
{
    EVE:Execute[CmdOpenStationPanel]
    wait 20
}
```

**Station Services Available**:
- Repair
- Reprocessing
- Insurance
- Clone services
- Corporation offices
- (Access methods vary, poorly documented)

### Repair Services

**Opening Repair Window**:
```lavish
; Must be in station
EVE:Execute[CmdOpenRepairShop]
wait 50
```

**Repairing** (using window):
```lavish
variable evwindow repair = ${EVEWindow[repairshop]}

if ${repair(exists)}
{
    ; Click "Repair All" button (if it exists)
    if ${repair.Button["Repair All"](exists)}
    {
        repair:ClickButton["Repair All"]
        wait 50
    }
}
```

### Reprocessing

**Opening Reprocessing**:
```lavish
EVE:Execute[CmdOpenReprocessingPlant]
wait 50
```

### Insurance

**Limited support**. Most bots skip insurance automation.

---

## UI Timing and Wait Patterns

### Why Timing Matters

**UI Operations are Asynchronous**:
1. `EVE:Execute` returns immediately (doesn't wait for completion)
2. UI updates after network round-trip (server authority)
3. Complex windows load in stages (window opens, then data loads)
4. Button clicks trigger actions that take time

**Failure Modes if No Waits**:
- Script checks for window before it appears (false negative)
- Script clicks button before window fully loaded (click fails silently)
- Script tries to access items before inventory data loaded (crashes)

### Minimum Wait Times (Empirical)

**After Opening Window**:
```lavish
EVE:Execute[CmdOpenInventory]
wait 20    ; Minimum for simple windows

EVE:Execute[CmdOpenMarket]
wait 50    ; Market is slower

EVE:Execute[CmdOpenFitting]
wait 30    ; Fitting window has complex load
```

**After Clicking Button**:
```lavish
win:ClickButton["OK"]
wait 50    ; Allow UI to process
```

**After Module Activation**:
```lavish
EVE:Execute[CmdActivateModule, ${slotID}]
wait 10    ; Module activation is fast (local UI update)
```

**After Warp Command**:
```lavish
EVE:Execute[CmdWarpToBookmark, ${bookmarkID}]
wait 100   ; Ship state changes (server round-trip)

; Then wait for warp to complete (separate check)
while !${MyShip.ToEntity.Mode} == 3    ; Mode 3 = Warping
{
    wait 100
}
echo "Warp initiated"
```

### Timeout Pattern (Essential)

```lavish
function WaitForWindowWithTimeout(string windowName, int timeoutSeconds)
{
    variable int startTime = ${Script.RunningTime}
    variable int elapsed

    while TRUE
    {
        ; Check condition
        if ${EVEWindow[${windowName}](exists)}
            return TRUE

        ; Check timeout
        elapsed:Set[${Math.Calc[(${Script.RunningTime} - ${startTime}) / 1000]}]
        if ${elapsed} >= ${timeoutSeconds}
        {
            echo "TIMEOUT: Window ${windowName} did not appear in ${timeoutSeconds}s"
            return FALSE
        }

        wait 100
    }
}

; Usage
EVE:Execute[CmdOpenInventory]
if !${WaitForWindowWithTimeout["inventory", 10]}
{
    echo "Failed to open inventory"
    return
}
```

### Wait Between UI Actions (Rate Limiting)

**Important**: Don't spam UI commands too fast or game may lag/ignore commands.

```lavish
; BAD - Commands too fast
EVE:Execute[CmdOpenInventory]
EVE:Execute[CmdOpenMarket]
EVE:Execute[CmdOpenFitting]
; All three might fail!

; GOOD - Wait between commands
EVE:Execute[CmdOpenInventory]
wait 50

EVE:Execute[CmdOpenMarket]
wait 50

EVE:Execute[CmdOpenFitting]
wait 50
```

---

## UI State Validation

### Always Validate Before and After

**Before UI Action**:
```lavish
function ClickButtonSafe(string windowName, string buttonText)
{
    ; Validate window exists
    variable evwindow win = ${EVEWindow[${windowName}]}
    if !${win(exists)}
    {
        echo "ERROR: Window not found"
        return FALSE
    }

    ; Validate button exists
    if !${win.Button[${buttonText}](exists)}
    {
        echo "ERROR: Button not found"
        return FALSE
    }

    ; Safe to click
    win:ClickButton[${buttonText}]
    return TRUE
}
```

**After UI Action**:
```lavish
function CloseWindowSafe(string windowName)
{
    variable evwindow win = ${EVEWindow[${windowName}]}

    if !${win(exists)}
    {
        echo "Window already closed"
        return TRUE
    }

    ; Close it
    win:Close
    wait 50

    ; Validate it closed
    if ${EVEWindow[${windowName}](exists)}
    {
        echo "WARNING: Window did not close"
        return FALSE
    }

    echo "Window closed successfully"
    return TRUE
}
```

### State Machine Pattern for Complex UI

```lavish
objectdef obj_UIStateMachine
{
    variable string state = "IDLE"

    method SetState(string newState)
    {
        echo "UI State: ${state} -> ${newState}"
        state:Set["${newState}"]
    }

    method ProcessMarketPurchase()
    {
        if ${state.Equal["IDLE"]}
        {
            echo "Opening market..."
            EVE:Execute[CmdOpenMarket]
            This:SetState["OPENING_MARKET"]
            return
        }

        if ${state.Equal["OPENING_MARKET"]}
        {
            if ${EVEWindow[market](exists)}
            {
                echo "Market opened"
                This:SetState["SEARCHING_ITEM"]
            }
            return
        }

        if ${state.Equal["SEARCHING_ITEM"]}
        {
            ; Complex search logic here...
            This:SetState["CLICKING_BUY"]
            return
        }

        ; etc.
    }
}
```

---

## Common Patterns from Example Scripts

### Pattern 1: Safe Window Open (Evebot)

```lavish
function Evebot_OpenInventory()
{
    if ${EVEWindow[inventory](exists)}
    {
        return TRUE
    }

    echo "Opening inventory"
    EVE:Execute[CmdOpenInventory]

    variable int counter = 0
    while !${EVEWindow[inventory](exists)} && ${counter} < 50
    {
        wait 10
        counter:Inc
    }

    if !${EVEWindow[inventory](exists)}
    {
        echo "ERROR: Could not open inventory"
        return FALSE
    }

    return TRUE
}
```

### Pattern 2: Inventory Item Iteration (Modern API)

**⚠️ NOTE:** Original Yamfa uses deprecated `MyShip.GetCargo`. Updated to modern API below.

```lavish
function ProcessCargoItems()
{
    ; Open inventory window
    if !${EVEWindow[Inventory](exists)}
    {
        EVE:Execute[CmdOpenInventory]
        wait 20
    }

    if !${EVEWindow[Inventory](exists)}
    {
        echo "ERROR: Cannot open inventory"
        return
    }

    ; MODERN API: Get cargo items via inventory window
    variable index:item CargoItems
    EVEWindow[Inventory].Child[ShipCargo]:GetItems[CargoItems]

    echo "Cargo has ${CargoItems.Used} items"

    variable iterator Item
    CargoItems:GetIterator[Item]

    if ${Item:First(exists)}
    {
        do
        {
            echo "Item: ${Item.Value.Name} (Qty: ${Item.Value.Quantity})"
        }
        while ${Item:Next(exists)}
    }
}
```

### Pattern 3: Button Click with Retry (Tehbot - Simplified)

```lavish
function ClickButtonWithRetry(string windowName, string buttonText, int maxAttempts)
{
    variable int attempt = 0

    while ${attempt} < ${maxAttempts}
    {
        attempt:Inc

        variable evwindow win = ${EVEWindow[${windowName}]}

        if !${win(exists)}
        {
            echo "Attempt ${attempt}: Window not found, waiting..."
            wait 100
            continue
        }

        if !${win.Button[${buttonText}](exists)}
        {
            echo "Attempt ${attempt}: Button not found, waiting..."
            wait 100
            continue
        }

        ; Click and return success
        win:ClickButton[${buttonText}]
        echo "Button clicked successfully"
        return TRUE
    }

    echo "ERROR: Failed to click button after ${maxAttempts} attempts"
    return FALSE
}
```

---

## Critical Gotchas and Issues

### Gotcha 1: Window Names Change Between EVE Versions

**Problem**: CCP sometimes changes window internal names.

**Solution**: Use multiple name attempts or ByCaption:
```lavish
variable evwindow win = ${EVEWindow[inventory]}

if !${win(exists)}
{
    win:Set[${EVEWindow[ByCaption,Inventory]}]
}

if !${win(exists)}
{
    echo "ERROR: Cannot find inventory window"
}
```

### Gotcha 2: Multiple Windows with Same Name

**Problem**: Opening multiple cargo containers = multiple "inventory" windows.

**Solution**: Use ByItemID or track window IDs:
```lavish
; Open container and get its specific window
variable int64 containerID = ${Entity[Name = "Container"].ID}

EVE:Execute[OpenCargoHold, ${containerID}]
wait 20

; Find window by item ID
variable evwindow containerWin = ${EVEWindow[ByItemID,${containerID}]}

if ${containerWin(exists)}
{
    echo "Found container window: ${containerWin.Name}"
}
```

### Gotcha 3: Buttons Without Text

**Problem**: Some buttons have no text (icon-only).

**Solution**: Use button index (fragile) or avoid:
```lavish
; Click first button (might be "OK")
win:ClickButton[1]
```

### Gotcha 4: Windows Not Fully Loaded

**Problem**: Window appears but data not loaded yet.

**Solution**: Add extra waits or check data presence:
```lavish
EVE:Execute[CmdOpenMarket]
wait 50    ; Initial wait

; Additional validation
variable evwindow market = ${EVEWindow[market]}
variable int attempts = 0

; Wait for market to have content (example - actual check varies)
while ${attempts} < 20
{
    ; Actual content check would go here (complex)
    wait 100
    attempts:Inc
}
```

### Gotcha 5: Race Conditions (Clicking Too Soon)

**Problem**: Script clicks before UI processes previous action.

**Example**:
```lavish
; BAD
win:ClickButton["Next"]
win:ClickButton["Confirm"]    ; Might click before "Next" processed!

; GOOD
win:ClickButton["Next"]
wait 50
win:ClickButton["Confirm"]
```

### Gotcha 6: Server Lag

**Problem**: High latency = longer waits needed.

**Solution**: Increase timeouts, add retry logic:
```lavish
; If players have high ping, increase waits
variable int UI_WAIT = 50    ; Normal
variable int UI_WAIT = 100   ; High latency

EVE:Execute[CmdOpenInventory]
wait ${UI_WAIT}
```

---

## Anti-Patterns to Avoid

### Anti-Pattern 1: No Wait After Execute

**Bad**:
```lavish
EVE:Execute[CmdOpenInventory]
if ${EVEWindow[inventory](exists)}    ; Will be FALSE!
{
    echo "Inventory open"
}
```

**Good**:
```lavish
EVE:Execute[CmdOpenInventory]
wait 20

if ${EVEWindow[inventory](exists)}
{
    echo "Inventory open"
}
```

### Anti-Pattern 2: No Existence Check

**Bad**:
```lavish
variable evwindow win = ${EVEWindow[inventory]}
echo "Name: ${win.Name}"    ; CRASH if window doesn't exist!
```

**Good**:
```lavish
variable evwindow win = ${EVEWindow[inventory]}

if !${win(exists)}
{
    echo "ERROR: Window not found"
    return
}

echo "Name: ${win.Name}"    ; Safe
```

### Anti-Pattern 3: No Timeout on Wait Loops

**Bad**:
```lavish
while !${EVEWindow[inventory](exists)}
{
    wait 100
}
; Infinite loop if window never appears!
```

**Good**:
```lavish
variable int attempts = 0
while !${EVEWindow[inventory](exists)} && ${attempts} < 50
{
    wait 100
    attempts:Inc
}

if !${EVEWindow[inventory](exists)}
{
    echo "ERROR: Timeout"
}
```

### Anti-Pattern 4: Relying on Fixed Button Indices

**Bad**:
```lavish
win:ClickButton[3]    ; What if CCP rearranges buttons?
```

**Good**:
```lavish
win:ClickButton["OK"]    ; Text is more stable
```

### Anti-Pattern 5: Complex UI Automation Without State Tracking

**Bad**:
```lavish
; Trying to do complex market purchase in linear code
EVE:Execute[CmdOpenMarket]
; ...100 lines of brittle clicking...
```

**Good**:
```lavish
; Use state machine pattern (see earlier section)
; Allows recovery from failures, retries, etc.
```

---

## Summary and Key Takeaways

### Essential Rules for UI Interaction

1. **Always use EVE:Execute** - Cannot directly manipulate UI
2. **Always wait after Execute** - UI updates are asynchronous
3. **Always validate window existence** - Scripts crash if window doesn't exist
4. **Always use timeouts** - Don't create infinite loops
5. **Always check state after actions** - Execute doesn't return success/failure

### Common UI Workflow

```lavish
; 1. Open window
EVE:Execute[CmdOpenInventory]
wait 20

; 2. Validate it opened
if !${EVEWindow[inventory](exists)}
{
    echo "ERROR: Failed to open"
    return FALSE
}

; 3. Perform action
EVEWindow[inventory]:ClickButton["OK"]
wait 50

; 4. Validate result
if ${EVEWindow[inventory](exists)}
{
    echo "WARNING: Window still open"
}
```

### Most Critical Commands

- `CmdOpenInventory` - Inventory access
- `CmdOpenCargoHold` - Cargo access
- `CmdOpenHangarFloor` - Ship hangar
- `CmdActivateModule` / `CmdDeactivateModule` - Module control
- `CmdDock` / `CmdUndock` - Station interaction
- `CmdStopShip` - Emergency stop

### Most Critical Objects

- `${EVEWindow[name]}` - Find windows
- `${EVEWindow[...](exists)}` - Validate existence
- `EVEWindow:Close` - Close windows
- `EVEWindow:ClickButton` - Click buttons

### Timing Guidelines

- Simple window open: 20ms wait
- Complex window (market, fitting): 50ms+ wait
- After button click: 50ms wait
- After warp/dock command: 100ms+ wait
- Always add timeout to loops (10-20 second max)

---

## Next Steps

**Continue to Other Layer 3 Files**:
- File 09: ISXEVE_Core_Objects_Reference.md (${Me}, ${MyShip}, ${EVE})
- File 10: Entity_System_and_Targeting.md (QueryEntities, targeting patterns)
- File 12: Module_Management_and_Ship_Control.md (Module control patterns)

**Apply Knowledge**:
- All bot tasks require UI interaction
- Use patterns from this file when implementing inventory, market, station logic
- Recognize timing issues in example scripts (common source of bugs)

---

## Wiki References

**EVE Object**:
- `IsxeveWiki/ISXEVE/EVE_(Object_Type).html` - EVE:Execute command reference

**EVEWindow Object**:
- `IsxeveWiki/ISXEVE/EVEWindow_(Object_Type).html` - Window object reference

**Common Commands** (scattered across wiki):
- Search for "Cmd*" in wiki for command names
- Example scripts are better reference than wiki (wiki is incomplete)

---

**END OF FILE 15**
**Status**: Complete (~1800 lines)
**Next**: Update progress tracker, continue with File 09, 10, or others
