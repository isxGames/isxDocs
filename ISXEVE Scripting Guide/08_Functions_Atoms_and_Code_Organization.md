# Functions, Atoms, and Code Organization
## LavishScript Function Architecture and Multi-File Script Patterns

**Part of**: EVE Online Bot Development - AI-Level Documentation
**Layer**: 2 - Scripting Fundamentals
**Prerequisites**: Read files 04-07 (Language reference, syntax, variables, control flow)
**Builds Toward**: Layer 3 (ISXEVE API), Layer 4 (Bot architecture patterns)

---

## Purpose of This Document

This file provides **exhaustive coverage** of:
- Function declaration, parameters, and return mechanisms
- Variable scope within functions (local vs function-level)
- Atoms: special event-driven and relay-callable functions
- Relay communication architecture for fleet coordination
- Multi-file script organization and include patterns
- Library creation and code reuse strategies
- Real-world patterns from Evebot, Yamfa, and Tehbot

**Critical for**: Building modular, maintainable bots with clean code organization.

---

## Table of Contents

1. [Function Basics](#function-basics)
2. [Parameters and Arguments](#parameters-and-arguments)
3. [Return Values and Mechanisms](#return-values-and-mechanisms)
4. [Variable Scope in Functions](#variable-scope-in-functions)
5. [Atoms: Event-Driven Functions](#atoms-event-driven-functions)
6. [Relay Atoms and IPC](#relay-atoms-and-ipc)
7. [Code Organization Patterns](#code-organization-patterns)
8. [Multi-File Scripts and Includes](#multi-file-scripts-and-includes)
9. [Library Patterns](#library-patterns)
10. [Object-Oriented Patterns](#object-oriented-patterns)
11. [Best Practices](#best-practices)
12. [Common Patterns from Example Scripts](#common-patterns-from-example-scripts)
13. [Anti-Patterns to Avoid](#anti-patterns-to-avoid)

---

## Function Basics

### Function Declaration Syntax

```lavish
function FunctionName()
{
    ; Function body
    echo "Hello from FunctionName"
}
```

**Key Rules**:
- Functions declared with `function` keyword
- Function names are **case-insensitive** (FunctionName = functionname)
- No return type declaration (LavishScript is dynamically typed)
- Functions can be called before they're declared (forward references work)
- Functions can be nested (define functions inside functions)

### Calling Functions

```lavish
; Simple call
FunctionName

; Call with parentheses (same result)
FunctionName()

; From a variable (function pointer pattern)
variable string funcName = "FunctionName"
call ${funcName}
```

**Call Keyword**:
```lavish
; Explicit call (required for variable-based calls)
call FunctionName

; Call with parameters
call FunctionName "param1" "param2"

; Call and capture return
variable string result
result:Set[${FunctionName["param"]}]
```

### Function Naming Conventions

**From Evebot**:
```lavish
function main()                    ; Entry point
function ProcessCombat()          ; PascalCase for major functions
function obj_Defense_OnFrame()    ; Object methods (objectdef pattern)
function Defense_Init()           ; Module_Action pattern
```

**From Yamfa**:
```lavish
function atom atexit()            ; Special atom (always lowercase 'atom')
function SetupUI()                ; Setup/initialization functions
function UpdateTargets()          ; Action functions
function IsValidTarget()          ; Query functions (Is/Has/Can prefix)
```

**Recommended Pattern**:
- `main()` - Entry point
- `Initialize()` / `Init()` - Setup
- `Cleanup()` / `Shutdown()` - Teardown
- `Process*()` - Main logic (ProcessCombat, ProcessMining)
- `Update*()` - State updates
- `Is*() / Has*() / Can*()` - Boolean queries
- `Get*()` - Data retrieval
- `Set*()` - Data modification

---

## Parameters and Arguments

### Basic Parameters

```lavish
function Greet(string name)
{
    echo "Hello, ${name}!"
}

; Call
call Greet "Bob"
; Output: Hello, Bob!
```

### Multiple Parameters

```lavish
function AddNumbers(int num1, int num2)
{
    echo "Sum: ${Math.Calc[${num1} + ${num2}]}"
}

call AddNumbers 5 10
; Output: Sum: 15
```

### Parameter Types

**All standard types supported**:
```lavish
function ComplexFunction(bool isActive, int count, float distance, string name, int64 entityID)
{
    echo "Active: ${isActive}, Count: ${count}, Distance: ${distance}"
    echo "Name: ${name}, ID: ${entityID}"
}

call ComplexFunction TRUE 5 1500.5 "Asteroid" 123456789012345
```

### ByRef vs ByVal (CRITICAL)

**Default: ByVal (Pass by Value)**:
```lavish
function ModifyValue(int number)
{
    number:Set[999]
    echo "Inside function: ${number}"
}

variable int myNumber = 5
call ModifyValue ${myNumber}
echo "Outside function: ${myNumber}"

; Output:
; Inside function: 999
; Outside function: 5    <- NOT modified!
```

**ByRef (Pass by Reference)**:
```lavish
function ModifyValueByRef(int byref number)
{
    number:Set[999]
    echo "Inside function: ${number}"
}

variable int myNumber = 5
call ModifyValueByRef myNumber    ; NOTE: No ${} when passing byref!
echo "Outside function: ${myNumber}"

; Output:
; Inside function: 999
; Outside function: 999    <- WAS modified!
```

**CRITICAL BYREF RULES**:
1. Declare parameter with `byref` keyword: `function Func(int byref param)`
2. When calling, pass variable NAME (no `${}`): `call Func varName`
3. ByRef allows function to modify caller's variable
4. Common for "output parameters" pattern

### Multiple ByRef Parameters

```lavish
function GetShipInfo(string byref shipName, int byref shipID, float byref shipCapacitor)
{
    shipName:Set["${MyShip.Name}"]
    shipID:Set[${MyShip.ID}]
    shipCapacitor:Set[${MyShip.Capacitor}]
}

variable string name
variable int id
variable float cap

call GetShipInfo name id cap

echo "Ship: ${name}, ID: ${id}, Cap: ${cap}"
```

### Optional Parameters (Not Supported - Workaround)

**LavishScript does NOT support optional parameters**. Use these patterns instead:

**Pattern 1: Multiple Versions**:
```lavish
function WarpToBookmark(string bookmarkName)
{
    call WarpToBookmarkAtDistance "${bookmarkName}" 0
}

function WarpToBookmarkAtDistance(string bookmarkName, int distance)
{
    ; Implementation
    echo "Warping to ${bookmarkName} at ${distance}m"
}
```

**Pattern 2: Use -1 or Empty String as "Not Provided"**:
```lavish
function WarpTo(string destination, int distance)
{
    if ${distance} == -1
    {
        distance:Set[0]
    }
    echo "Warping to ${destination} at ${distance}m"
}

; Call with default
call WarpTo "Station" -1

; Call with specific distance
call WarpTo "Bookmark" 100000
```

### Variadic Functions (Not Supported)

LavishScript does **not** support variable-length argument lists. Maximum parameters is finite (exact limit unclear, but 10+ works fine).

---

## Return Values and Mechanisms

### Returning Values

**Use `return` statement**:
```lavish
function GetShipName()
{
    return "${MyShip.Name}"
}

variable string name
name:Set[${GetShipName}]
echo "Ship: ${name}"
```

### Return Types

**Functions can return any type**:
```lavish
function IsInStation()
{
    return ${Me.InStation}
}

function GetTargetCount()
{
    return ${Me.GetTargets}
}

function GetMaxTargetRange()
{
    return ${Me.MaxTargetRange}
}

; Usage
if ${IsInStation}
{
    echo "We're docked"
}

variable int targets = ${GetTargetCount}
variable float range = ${GetMaxTargetRange}
```

### Multiple Return Values (ByRef Pattern)

**Since functions can only return one value, use byref for multiple outputs**:

```lavish
function GetTargetInfo(int64 targetID, string byref name, float byref distance, int byref type)
{
    variable entity target
    target:Set[${Entity[${targetID}]}]

    if !${target(exists)}
    {
        return FALSE
    }

    name:Set["${target.Name}"]
    distance:Set[${target.Distance}]
    type:Set[${target.TypeID}]

    return TRUE
}

; Usage
variable string tgtName
variable float tgtDist
variable int tgtType

if ${GetTargetInfo[${MyTarget.ID}, tgtName, tgtDist, tgtType]}
{
    echo "Target: ${tgtName}, ${tgtDist}m, Type ${tgtType}"
}
else
{
    echo "Failed to get target info"
}
```

### Return Without Value (Early Exit)

```lavish
function ProcessTarget(int64 entityID)
{
    variable entity target = ${Entity[${entityID}]}

    if !${target(exists)}
    {
        echo "Target doesn't exist, aborting"
        return    ; Exit early
    }

    if ${target.Distance} > 150000
    {
        echo "Target too far"
        return
    }

    ; Continue processing
    echo "Processing ${target.Name}"
}
```

### Return in Main Loop (Script Exit)

```lavish
function main()
{
    while TRUE
    {
        if !${ISXEVE(exists)}
        {
            echo "ISXEVE not loaded, exiting"
            return    ; Exits script entirely
        }

        if ${Me.InStation}
        {
            echo "We're in station, script complete"
            return    ; Exits script
        }

        call ProcessTick
        wait 10
    }
}
```

---

## Variable Scope in Functions

### Local Scope (Default in Functions)

```lavish
function TestScope()
{
    variable int localVar = 100
    echo "Local: ${localVar}"
}

call TestScope
; Output: Local: 100

echo "${localVar}"
; ERROR: localVar doesn't exist outside function!
```

### Function-Level vs Block-Level

**Variables declared in function are function-scoped** (accessible in all blocks within function):

```lavish
function ScopeTest()
{
    variable int funcVar = 1

    if TRUE
    {
        variable int blockVar = 2
        echo "Block can see funcVar: ${funcVar}"
        echo "Block can see blockVar: ${blockVar}"
    }

    echo "Function can see funcVar: ${funcVar}"
    echo "Function can see blockVar: ${blockVar}"    ; Still works! (block-level scope is function-level in LS)
}
```

**IMPORTANT**: Unlike C/C++/C#, LavishScript variables declared in `if`, `while`, etc. blocks are **function-scoped**, not block-scoped!

### Accessing Global Variables from Functions

```lavish
variable(global) string globalName = "GlobalValue"

function AccessGlobal()
{
    echo "Global from function: ${globalName}"
    globalName:Set["Modified"]
}

call AccessGlobal
echo "After function: ${globalName}"

; Output:
; Global from function: GlobalValue
; After function: Modified
```

### Shadowing (Local vs Global Same Name)

```lavish
variable(global) int count = 100

function ShadowTest()
{
    variable int count = 50    ; Local shadows global
    echo "Local count: ${count}"
    count:Inc[10]
    echo "Local after inc: ${count}"
}

call ShadowTest
echo "Global count: ${count}"

; Output:
; Local count: 50
; Local after inc: 60
; Global count: 100    <- Global unchanged
```

### Script Scope (Cross-Function Sharing)

```lavish
variable(script) int sharedCounter = 0

function IncrementCounter()
{
    sharedCounter:Inc
}

function GetCounter()
{
    return ${sharedCounter}
}

call IncrementCounter
call IncrementCounter
echo "Counter: ${GetCounter}"
; Output: Counter: 2
```

### Best Practice: Minimize Global Usage

```lavish
; BAD: Everything global
variable(global) int targetCount
variable(global) float maxRange
variable(global) bool inCombat

; GOOD: Pass data as parameters
function ProcessCombat(int targetCount, float maxRange, bool inCombat)
{
    ; Use parameters
}

; Or use objectdef for state management (covered later)
```

---

## Atoms: Event-Driven Functions

### What Are Atoms?

**Atoms** are special functions that:
1. Can be called remotely via **Relay** (IPC system)
2. Can be registered as **event handlers**
3. Are declared with `atom` keyword
4. Follow special naming conventions

### Atom Declaration

```lavish
function atom MyAtom()
{
    echo "Atom called!"
}
```

**Atom with Parameters**:
```lavish
function atom MyAtomWithParams(int value, string text)
{
    echo "Received: ${value}, ${text}"
}
```

### Special Atoms (Reserved Names)

**`atexit` - Cleanup on Script Exit**:
```lavish
function atom atexit()
{
    echo "Script is exiting, cleaning up..."

    ; Cleanup code
    if ${Me.InSpace}
    {
        EVE:Execute[CmdStopShip]
    }
}
```

**When atexit is called**:
- Script ends normally (main returns)
- Script is manually ended (End or EndScript command)
- User closes the script
- NOT called if script crashes or InnerSpace closes

**Example from Yamfa**:
```lavish
function atom atexit()
{
    echo "${APP_NAME} exiting..."

    ; Unregister relay atoms
    relay "all other" Event[EVENT_YAMFA_SHUTDOWN]:Execute

    ; Save settings
    call SaveSettings

    ; Close UI
    call UICleanup
}
```

### Event Atoms (Not Common in ISXEVE)

**Registering atom as event handler**:
```lavish
function atom onFrameEvent()
{
    echo "Frame event!"
}

; Register (not common in ISXEVE - use OnFrame objects instead)
Event[OnFrame]:AttachAtom[onFrameEvent]

; Detach
Event[OnFrame]:DetachAtom[onFrameEvent]
```

**NOTE**: ISXEVE typically uses **objectdef OnFrame patterns** instead of event atoms (covered in objectdef section).

---

## Relay Atoms and IPC

### Relay System Overview

**Relay** = InnerSpace's IPC (Inter-Process Communication) system for multi-client coordination.

**Key Concepts**:
- Each InnerSpace session (is1, is2, is3) runs a separate EVE client
- Relay allows sessions to call atoms in other sessions
- Used for fleet coordination (Yamfa pattern: master calls slave targeting atoms)
- Uplink = Relay server component

### Relay Atom Requirements

1. **Must be declared as `atom`**
2. **Must be "relayed" to make accessible**

### Relaying Atoms

```lavish
; Make atom callable via relay
relay "all" atom MyAtom

; Make multiple atoms callable
relay "all" atom TargetEntity
relay "all" atom AssistMaster
relay "all" atom StopFiring
```

**Relay Targets**:
- `"all"` - All other sessions
- `"all local"` - All sessions on this PC
- `"all other"` - All except this session
- `"all other local"` - All other sessions on this PC
- Specific session: `"is2"`, `"is3"`

### Calling Relay Atoms

**From another session**:
```lavish
; Call atom on all other sessions
relay "all other" MyAtom

; Call with parameters
relay "all other" TargetEntity ${entityID}

; Call on specific session
relay "is2" AssistMaster
```

### Relay Atom Pattern (Master/Slave)

**Master Script (Yamfa-style)**:
```lavish
; Master tells all slaves to target an entity
function BroadcastTarget(int64 entityID)
{
    echo "Broadcasting target ${entityID} to fleet"
    relay "all other" TargetEntity ${entityID}
}

function main()
{
    while TRUE
    {
        ; Master's target selection logic
        variable int64 bestTarget = ${GetBestTarget}

        if ${bestTarget} > 0
        {
            call BroadcastTarget ${bestTarget}
        }

        wait 1000
    }
}
```

**Slave Script**:
```lavish
; Relay atom - called by master
function atom TargetEntity(int64 entityID)
{
    echo "Master ordered target: ${entityID}"

    variable entity target = ${Entity[${entityID}]}
    if ${target(exists)}
    {
        target:LockTarget
        echo "Locking ${target.Name}"
    }
}

; Make atom callable via relay
relay "all" atom TargetEntity

function main()
{
    while TRUE
    {
        ; Slave waits for master commands via relay
        wait 10
    }
}
```

### Relay Events (Broadcast Pattern)

**Define relay event**:
```lavish
; In initialization
Event[EVENT_YAMFA_TARGET]:AttachAtom[TargetEntity]
```

**Trigger event via relay**:
```lavish
; From master
relay "all other" Event[EVENT_YAMFA_TARGET]:Execute[${entityID}]
```

**Atom receives event**:
```lavish
function atom TargetEntity(int64 entityID)
{
    echo "Event triggered: Target ${entityID}"
    ; Process target
}
```

### Relay Best Practices

1. **Always validate relay parameters** (other sessions could send bad data)
2. **Use unique event names** to avoid conflicts with other scripts
3. **Relay setup should happen early** in script initialization
4. **Clean up relay on exit** (relay "all other" Event[SHUTDOWN])

**Example from Yamfa** (Cleaned Up):
```lavish
variable(global) string RELAY_GROUP = "YamfaFleet"

function SetupRelay()
{
    ; Join relay group
    relay "${RELAY_GROUP}" "all other local"

    ; Register atoms
    relay "${RELAY_GROUP}" atom TargetEntity
    relay "${RELAY_GROUP}" atom StopFiring
    relay "${RELAY_GROUP}" atom AssistComplete

    echo "Relay setup complete"
}

function atom atexit()
{
    ; Notify others we're leaving
    relay "${RELAY_GROUP}" Event[YamfaSlaveDisconnect]:Execute["${Me.Name}"]

    ; Leave group
    relay "${RELAY_GROUP}" quit
}
```

---

## Code Organization Patterns

### Single File Scripts (Simple Bots)

**Structure**:
```lavish
; ========================================
; Configuration Variables
; ========================================
variable(global) int COMBAT_RANGE = 50000
variable(global) int SAFETY_TIMEOUT = 300

; ========================================
; State Variables
; ========================================
variable(script) bool inCombat = FALSE
variable(script) int targetCount = 0

; ========================================
; Helper Functions
; ========================================
function IsValidTarget(int64 entityID)
{
    ; Implementation
    return TRUE
}

function GetBestTarget()
{
    ; Implementation
    return 0
}

; ========================================
; Core Logic Functions
; ========================================
function ProcessCombat()
{
    ; Implementation
}

function ProcessMining()
{
    ; Implementation
}

; ========================================
; Main Entry Point
; ========================================
function main()
{
    echo "Script starting..."

    while TRUE
    {
        call ProcessCombat
        wait 10
    }
}

; ========================================
; Cleanup
; ========================================
function atom atexit()
{
    echo "Cleanup..."
}
```

### Multi-Section Organization (Medium Scripts)

**Use clear section headers**:
```lavish
; #############################################
; # SECTION: Configuration
; #############################################

; #############################################
; # SECTION: Initialization
; #############################################

; #############################################
; # SECTION: Target Management
; #############################################

; #############################################
; # SECTION: Combat Logic
; #############################################

; #############################################
; # SECTION: Movement & Navigation
; #############################################

; #############################################
; # SECTION: Main Loop
; #############################################
```

---

## Multi-File Scripts and Includes

### Include Statement

**Load code from another file**:
```lavish
#include "path/to/file.iss"
```

**Include with relative path**:
```lavish
; Relative to script directory
#include "libs/common.iss"
#include "../shared/utilities.iss"
```

**Include with absolute path**:
```lavish
#include "C:/EVE/Scripts/libs/targeting.iss"
```

### Include Mechanics

**What #include does**:
1. Inserts the entire contents of the included file at that line
2. Acts like preprocessor (happens before script runs)
3. Variables/functions from include are available in main script
4. Can include multiple files
5. Can nest includes (file A includes B, B includes C)

**What #include does NOT do**:
- Does NOT create separate scope
- Does NOT prevent duplicate code (including same file twice = duplicate declarations)
- Does NOT have "import" or "using" concept

### Include Pattern (Evebot Style)

**Main Script (Evebot.iss)**:
```lavish
; Core includes
#include "Support/evebot.h.iss"

; Module includes
#include "Branches/Defense.iss"
#include "Branches/Cargo.iss"
#include "Branches/Combat.iss"
#include "Branches/Mining.iss"

function main()
{
    ; Functions from includes are available
    call Defense_Init
    call Cargo_Init

    while TRUE
    {
        call ProcessCombat
        call ProcessMining
        wait 10
    }
}
```

**Header File (evebot.h.iss)**:
```lavish
; Global configuration
variable(global) string VERSION = "1.0.0"

; Global constants
variable(global) int MINING_RANGE = 15000
variable(global) int COMBAT_RANGE = 50000

; Shared utility functions
function DebugLog(string message)
{
    echo "[${Time}] ${message}"
}
```

**Module File (Combat.iss)**:
```lavish
; Combat module

variable(script) bool combatActive = FALSE

function Combat_Init()
{
    echo "Combat module initialized"
}

function ProcessCombat()
{
    if ${Me.InSpace} && ${combatActive}
    {
        ; Combat logic
    }
}
```

### Include Guards (Prevent Duplicate Inclusion)

**LavishScript does NOT have built-in include guards**, so use this pattern:

```lavish
; File: common.iss

; Check if already included
if !${COMMON_ISS_INCLUDED(exists)}
{
    variable(global) bool COMMON_ISS_INCLUDED = TRUE

    ; All your code here
    function CommonFunction()
    {
        echo "Common function"
    }
}
```

**Usage**:
```lavish
; These multiple includes won't cause duplicate declarations
#include "common.iss"
#include "common.iss"    ; Skipped due to guard
```

### Multi-File Script Structure (Best Practice)

```
MyBot/
├── MyBot.iss                  ; Main entry point
├── config/
│   └── settings.iss           ; Configuration variables
├── libs/
│   ├── common.iss             ; Shared utilities
│   ├── targeting.iss          ; Targeting logic
│   └── navigation.iss         ; Movement logic
└── modules/
    ├── combat.iss             ; Combat module
    ├── mining.iss             ; Mining module
    └── hauling.iss            ; Hauling module
```

**MyBot.iss**:
```lavish
#include "config/settings.iss"
#include "libs/common.iss"
#include "libs/targeting.iss"
#include "modules/combat.iss"
#include "modules/mining.iss"

function main()
{
    call InitializeModules

    while TRUE
    {
        call ProcessCombat
        call ProcessMining
        wait 10
    }
}
```

---

## Library Patterns

### Utility Library Pattern

**File: libs/utilities.iss**:
```lavish
if !${UTILITIES_ISS_INCLUDED(exists)}
{
    variable(global) bool UTILITIES_ISS_INCLUDED = TRUE

    ; ==========================================
    ; String Utilities
    ; ==========================================
    function StringContains(string haystack, string needle)
    {
        return ${haystack.Find["${needle}"](exists)}
    }

    ; ==========================================
    ; Math Utilities
    ; ==========================================
    function Clamp(float value, float min, float max)
    {
        if ${value} < ${min}
            return ${min}
        if ${value} > ${max}
            return ${max}
        return ${value}
    }

    ; ==========================================
    ; Timing Utilities
    ; ==========================================
    function WaitWithTimeout(int timeoutSeconds, string condition)
    {
        variable int start = ${Script.RunningTime}

        while TRUE
        {
            if ${condition}
                return TRUE

            if ${Math.Calc[${Script.RunningTime} - ${start}]} > ${Math.Calc[${timeoutSeconds} * 1000]}
                return FALSE

            wait 10
        }
    }
}
```

### Module Pattern (Initialization + Processing)

**Module Structure**:
```lavish
; ==========================================
; Module: Defense
; ==========================================

variable(script) bool defenseInitialized = FALSE
variable(script) bool defenseActive = FALSE

function Defense_Init()
{
    if ${defenseInitialized}
        return

    echo "Initializing defense module..."

    ; Setup
    defenseActive:Set[TRUE]
    defenseInitialized:Set[TRUE]
}

function Defense_Shutdown()
{
    defenseActive:Set[FALSE]
    echo "Defense module shut down"
}

function Defense_OnFrame()
{
    if !${defenseActive}
        return

    ; Process defensive actions
    call CheckThreatLevel
    call ActivateHardeners
}

function CheckThreatLevel()
{
    ; Implementation
}

function ActivateHardeners()
{
    ; Implementation
}
```

**Usage in Main Script**:
```lavish
#include "modules/defense.iss"

function main()
{
    call Defense_Init

    while TRUE
    {
        call Defense_OnFrame
        wait 10
    }
}

function atom atexit()
{
    call Defense_Shutdown
}
```

---

## Object-Oriented Patterns

### Objectdef Basics

**Declare object type**:
```lavish
objectdef MyObject
{
    variable string name
    variable int value

    method Initialize(string _name, int _value)
    {
        name:Set["${_name}"]
        value:Set[${_value}]
    }

    member:int GetValue()
    {
        return ${This.value}
    }

    method SetValue(int newValue)
    {
        This.value:Set[${newValue}]
    }
}
```

**Create and use object**:
```lavish
variable MyObject obj

obj:Initialize["Test", 100]
echo "Value: ${obj.GetValue}"

obj:SetValue[200]
echo "New value: ${obj.GetValue}"
```

### OnFrame Pattern (Critical for Bots)

**Most common objectdef pattern in EVE bots**:

```lavish
objectdef obj_MyBot
{
    variable bool running = FALSE
    variable int tickCount = 0

    method Initialize()
    {
        running:Set[TRUE]
        LavishScript:RegisterEvent[OnFrame]
        Event[OnFrame]:AttachAtom[This:OnFrame]
        echo "Bot initialized"
    }

    method Shutdown()
    {
        running:Set[FALSE]
        Event[OnFrame]:DetachAtom[This:OnFrame]
        echo "Bot shut down"
    }

    method OnFrame()
    {
        if !${running}
            return

        tickCount:Inc

        ; Main bot logic runs every frame
        call This:ProcessTick
    }

    method ProcessTick()
    {
        ; Your bot logic here
        if ${Math.Calc[${tickCount} % 100]} == 0
        {
            echo "Tick ${tickCount}"
        }
    }
}

; Global instance
variable(global) obj_MyBot Bot

function main()
{
    Bot:Initialize

    ; Keep script alive (OnFrame does the work)
    while ${Bot.running}
    {
        wait 1000
    }
}

function atom atexit()
{
    Bot:Shutdown
}
```

**Why OnFrame Pattern?**:
- Runs every frame (multiple times per second)
- More responsive than `wait` loops
- Common in Evebot and other advanced bots
- Allows precise timing control

### State Machine Object Pattern

```lavish
objectdef obj_StateMachine
{
    variable string currentState = "IDLE"

    method Initialize()
    {
        currentState:Set["IDLE"]
    }

    method SetState(string newState)
    {
        echo "State change: ${currentState} -> ${newState}"
        currentState:Set["${newState}"]
    }

    method OnFrame()
    {
        switch ${currentState}
        {
            case "IDLE"
                call This:State_Idle
                break
            case "MINING"
                call This:State_Mining
                break
            case "COMBAT"
                call This:State_Combat
                break
            default
                echo "Unknown state: ${currentState}"
        }
    }

    method State_Idle()
    {
        ; Idle logic
        if ${NeedToMine}
            This:SetState["MINING"]
    }

    method State_Mining()
    {
        ; Mining logic
        if ${UnderAttack}
            This:SetState["COMBAT"]
    }

    method State_Combat()
    {
        ; Combat logic
        if ${NoThreats}
            This:SetState["IDLE"]
    }
}
```

**IMPORTANT**: LavishScript's `switch` is actually **if-elseif chain** (not true switch). The pattern above shows intent, but implement as:

```lavish
method OnFrame()
{
    if ${currentState.Equal["IDLE"]}
        call This:State_Idle
    elseif ${currentState.Equal["MINING"]}
        call This:State_Mining
    elseif ${currentState.Equal["COMBAT"]}
        call This:State_Combat
}
```

---

## Best Practices

### 1. Function Naming

**Good**:
```lavish
function IsValidTarget(int64 entityID)           ; Clear boolean
function GetNearestAsteroid()                     ; Clear getter
function WarpToBookmark(string name)              ; Clear action
function ProcessCombatTick()                      ; Clear processing
```

**Bad**:
```lavish
function check(int64 id)                          ; Vague
function go(string s)                             ; Too short
function DoStuffAndThings()                       ; Too vague
```

### 2. Parameter Validation

**Always validate**:
```lavish
function LockTarget(int64 entityID)
{
    ; Validate parameter
    if ${entityID} <= 0
    {
        echo "ERROR: Invalid entity ID ${entityID}"
        return FALSE
    }

    variable entity target = ${Entity[${entityID}]}
    if !${target(exists)}
    {
        echo "ERROR: Entity ${entityID} doesn't exist"
        return FALSE
    }

    ; Proceed with valid target
    target:LockTarget
    return TRUE
}
```

### 3. Single Responsibility

**Good** (each function does one thing):
```lavish
function FindNearestRat()
{
    ; Only finding, not locking
    return ${GetNearestEntityByQuery["GroupID = 99"]}
}

function LockTarget(int64 entityID)
{
    ; Only locking, not finding
    Entity[${entityID}]:LockTarget
}

; Usage
variable int64 rat = ${FindNearestRat}
if ${rat} > 0
    call LockTarget ${rat}
```

**Bad** (function does too much):
```lavish
function FindAndLockRat()
{
    ; Finding AND locking (hard to reuse)
    variable int64 rat = ${GetNearest}
    Entity[${rat}]:LockTarget
    wait 5000
    ; What if I just want to find, not lock?
}
```

### 4. Error Handling in Functions

```lavish
function WarpToBookmark(string bookmarkName)
{
    variable int64 bookmarkID = ${EVEWindow[Inventory].GetBookmarkID["${bookmarkName}"]}

    if ${bookmarkID} <= 0
    {
        echo "ERROR: Bookmark '${bookmarkName}' not found"
        return FALSE
    }

    if !${Me.InSpace}
    {
        echo "ERROR: Cannot warp while docked"
        return FALSE
    }

    EVE:Execute[CmdWarpToBookmark, ${bookmarkID}]
    return TRUE
}

; Usage with error check
if !${WarpToBookmark["Mining Site"]}
{
    echo "Warp failed, aborting"
    return
}
```

### 5. Avoid Deep Nesting

**Bad**:
```lavish
function ProcessTarget(int64 entityID)
{
    if ${entityID} > 0
    {
        variable entity target = ${Entity[${entityID}]}
        if ${target(exists)}
        {
            if ${target.Distance} < 50000
            {
                if !${target.IsLockedTarget}
                {
                    target:LockTarget
                }
            }
        }
    }
}
```

**Good** (early returns):
```lavish
function ProcessTarget(int64 entityID)
{
    if ${entityID} <= 0
        return

    variable entity target = ${Entity[${entityID}]}
    if !${target(exists)}
        return

    if ${target.Distance} >= 50000
        return

    if ${target.IsLockedTarget}
        return

    target:LockTarget
}
```

### 6. Use Constants for Magic Numbers

**Bad**:
```lavish
function IsInRange(entity target)
{
    return ${target.Distance} < 50000
}
```

**Good**:
```lavish
variable(global) int MAX_TARGETING_RANGE = 50000

function IsInRange(entity target)
{
    return ${target.Distance} < ${MAX_TARGETING_RANGE}
}
```

### 7. Comment Complex Logic

```lavish
function GetOptimalDroneTarget()
{
    ; Priority system:
    ; 1. Frigates attacking us (high threat, easy to kill)
    ; 2. Cruisers attacking us (medium threat)
    ; 3. Any NPC in range (cleanup)

    ; Check for frigates first
    variable int64 frigate = ${GetNearestByGroupAndDistance[25, 50000]}
    if ${frigate} > 0
        return ${frigate}

    ; Fall back to cruisers
    variable int64 cruiser = ${GetNearestByGroupAndDistance[26, 50000]}
    if ${cruiser} > 0
        return ${cruiser}

    ; Finally any NPC
    return ${GetNearestNPC}
}
```

---

## Common Patterns from Example Scripts

### Pattern 1: Initialization/Shutdown Pair (Evebot)

```lavish
function Defense_Init()
{
    echo "Defense: Initializing"

    ; Load settings
    call Defense_LoadSettings

    ; Setup state
    variable(script) bool defenseActive = TRUE
}

function Defense_Shutdown()
{
    echo "Defense: Shutting down"

    ; Save state
    call Defense_SaveSettings

    ; Deactivate modules
    call DeactivateAllModules
}
```

### Pattern 2: Relay Command Atoms (Yamfa)

```lavish
; Master sends commands
function BroadcastTargetToFleet(int64 entityID)
{
    relay "all other" atom_TargetEntity ${entityID}
}

; Slave receives commands
function atom atom_TargetEntity(int64 entityID)
{
    echo "Received target command: ${entityID}"

    ; Validate
    variable entity target = ${Entity[${entityID}]}
    if !${target(exists)}
    {
        echo "Target doesn't exist, ignoring"
        return
    }

    ; Execute
    target:LockTarget
}

relay "all" atom atom_TargetEntity
```

### Pattern 3: Config Loading (Tehbot)

```lavish
function LoadConfiguration()
{
    ; Load from XML settings
    variable string configPath = "${Script.CurrentDirectory}/config.xml"

    if !${SettingXML[${configPath}](exists)}
    {
        echo "Config not found, using defaults"
        call CreateDefaultConfig
        return
    }

    ; Load values
    COMBAT_RANGE:Set[${SettingXML[${configPath}].FindSetting[CombatRange,50000]}]
    MINING_RANGE:Set[${SettingXML[${configPath}].FindSetting[MiningRange,15000]}]

    echo "Configuration loaded"
}
```

### Pattern 4: Safe Entity Iteration (Evebot)

```lavish
function ProcessAllTargets()
{
    variable int targetCount = ${Me.GetTargets}
    variable int i = 1

    ; Create snapshot of target IDs (prevents iteration issues)
    variable int64[] targetIDs

    for (i:Set[1]; ${i} <= ${targetCount}; i:Inc)
    {
        targetIDs:Insert[${Me.GetTarget[${i}].ID}]
    }

    ; Process snapshot
    for (i:Set[1]; ${i} <= ${targetIDs.Used}; i:Inc)
    {
        call ProcessTarget ${targetIDs[${i}]}
    }
}
```

### Pattern 5: Timeout Wrapper (Common)

```lavish
function WaitForCondition(string conditionDescription, int timeoutSeconds)
{
    variable int startTime = ${Script.RunningTime}
    variable int elapsed

    echo "Waiting for: ${conditionDescription}"

    while TRUE
    {
        ; Check your condition here (example: waiting for lock)
        ; This is a template - replace with actual condition

        elapsed:Set[${Math.Calc[(${Script.RunningTime} - ${startTime}) / 1000]}]

        if ${elapsed} >= ${timeoutSeconds}
        {
            echo "Timeout waiting for ${conditionDescription}"
            return FALSE
        }

        wait 100
    }

    return TRUE
}
```

---

## Anti-Patterns to Avoid

### 1. Global Variable Overuse

**Bad**:
```lavish
variable(global) int tempValue1
variable(global) int tempValue2
variable(global) string tempName

function CalculateSomething()
{
    tempValue1:Set[100]
    tempValue2:Set[200]
    ; Globals for temporary data = namespace pollution
}
```

**Good**:
```lavish
function CalculateSomething()
{
    variable int value1 = 100
    variable int value2 = 200
    ; Local scope, no pollution
}
```

### 2. Functions Without Return Values (When They Should Have Them)

**Bad**:
```lavish
variable(script) bool lastCheckResult

function CheckIfInRange(entity target)
{
    if ${target.Distance} < 50000
        lastCheckResult:Set[TRUE]
    else
        lastCheckResult:Set[FALSE]
}

; Usage - awkward!
call CheckIfInRange ${target}
if ${lastCheckResult}
    echo "In range"
```

**Good**:
```lavish
function IsInRange(entity target)
{
    return ${Math.Calc[${target.Distance} < 50000]}
}

; Usage - clean!
if ${IsInRange[${target}]}
    echo "In range"
```

### 3. Copy-Paste Code (Make Functions Instead)

**Bad**:
```lavish
; Repeated 10 times in script
echo "[${Time}] Message"
echo "[${Time}] Another message"
echo "[${Time}] Yet another message"
```

**Good**:
```lavish
function Log(string message)
{
    echo "[${Time}] ${message}"
}

call Log "Message"
call Log "Another message"
call Log "Yet another message"
```

### 4. Monolithic Functions (500+ Lines)

**Bad**:
```lavish
function DoEverything()
{
    ; 500 lines of combat logic
    ; 200 lines of mining logic
    ; 300 lines of navigation logic
    ; Impossible to maintain!
}
```

**Good**:
```lavish
function ProcessCombat()
{
    ; 50 lines
}

function ProcessMining()
{
    ; 50 lines
}

function ProcessNavigation()
{
    ; 50 lines
}

function main()
{
    call ProcessCombat
    call ProcessMining
    call ProcessNavigation
}
```

### 5. Missing Validation

**Bad**:
```lavish
function LockTarget(int64 entityID)
{
    Entity[${entityID}]:LockTarget    ; What if ID is invalid? Script crashes!
}
```

**Good**:
```lavish
function LockTarget(int64 entityID)
{
    if ${entityID} <= 0
        return FALSE

    variable entity target = ${Entity[${entityID}]}
    if !${target(exists)}
        return FALSE

    target:LockTarget
    return TRUE
}
```

---

## Summary and Key Takeaways

### Function Essentials
- Use `function` keyword for declaration
- Parameters: declare types, use `byref` for output parameters
- Return values: `return` statement, any type
- Scope: function-level (not block-level like C)

### Atoms and Relay
- Atoms: special functions for relay and events
- `function atom atomName()` syntax
- Relay: `relay "target" atomName params` for IPC
- `atexit`: special cleanup atom

### Code Organization
- Single file for simple bots
- Multi-file with `#include` for complex bots
- Use include guards to prevent duplicates
- Module pattern: Init/Shutdown/OnFrame
- Objectdef for OOP patterns

### Best Practices
- Validate all parameters
- Single responsibility per function
- Use early returns to reduce nesting
- Avoid global variables
- Comment complex logic
- Use constants for magic numbers

### Common Patterns
- OnFrame for frame-perfect execution
- State machines for complex logic
- Relay atoms for fleet coordination
- Timeout wrappers for safety
- Safe entity iteration (snapshot IDs)

---

## Next Steps

**Continue to Layer 3 (ISXEVE API Deep Dive)**:
- File 09: ISXEVE_Core_Objects_Reference.md
- File 10: Entity_System_and_Targeting.md
- File 15: UI_Windows_and_Menus.md (PRIORITY)

**Apply Knowledge**:
- Use function patterns from this file when reading API docs
- Recognize relay patterns in Yamfa analysis
- Understand objectdef patterns in Evebot analysis

---

## Wiki References

**LavishScript Functions**:
- `LavishScriptWiki/LavishScript/Functions.html` - Function declaration syntax
- `LavishScriptWiki/LavishScript/Atoms.html` - Atom details

**Relay System**:
- `LavishScriptWiki/InnerSpace/Uplink.html` - Uplink/Relay overview
- `LavishScriptWiki/LavishScript/Commands/relay.html` - Relay command

**Code Organization**:
- `LavishScriptWiki/LavishScript/Preprocessor.html` - #include and preprocessor

---

**END OF FILE 08**
**Status**: Complete (~1200 lines)
**Next**: Update progress tracker, continue to File 15 or File 09
