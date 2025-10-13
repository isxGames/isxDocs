# LavishScript Command Reference

> **Part of Layer 9: Reference Materials**
> Complete reference for LavishScript language syntax, commands, and patterns.

---

## Table of Contents

1. [Introduction to LavishScript](#introduction-to-lavishscript)
2. [Variables](#variables)
3. [Data Types](#data-types)
4. [Operators](#operators)
5. [Control Flow](#control-flow)
6. [Functions](#functions)
7. [Object-Oriented Programming](#object-oriented-programming)
8. [Collections](#collections)
9. [Commands](#commands)
10. [Math Operations](#math-operations)
11. [String Operations](#string-operations)
12. [File I/O](#file-io)
13. [Threading](#threading)
14. [Relay (IPC)](#relay-ipc)
15. [Events](#events)
16. [Script Management](#script-management)
17. [Common Patterns](#common-patterns)
18. [Best Practices](#best-practices)

---

## Introduction to LavishScript

### What is LavishScript?

**LavishScript** is a C++-based scripting language designed for game automation in InnerSpace.

**Key Characteristics:**
- Interpreted (no compilation)
- Weakly typed
- C-like syntax
- Object-oriented support
- Built-in threading
- Extension-based (ISXEVE, etc.)

### Basic Script Structure

```lavish
; Include files (optional)
#include "support/lib.iss"

; Global variables
variable(global) int MyGlobalVar

; Entry point
function main()
{
    ; Script logic here
    echo "Hello, EVE!"
}
```

### Execution

```bash
; Command line
run "path/to/script.iss"

; With parameters
run "path/to/script.iss" "param1" "param2"
```

### Comments

```lavish
; Single-line comment

/* Multi-line
   comment */
```

---

## Variables

### Declaration Syntax

```lavish
; Basic declaration
variable <type> <name>
variable <type> <name> = <value>

; Examples
variable int MyInt
variable int MyInt = 42
variable string MyString = "Hello"
variable bool MyBool = TRUE
```

### Variable Scopes

**Local (default):**
```lavish
function main()
{
    variable int MyLocal = 10
    ; MyLocal only exists in this function
}
```

**Global:**
```lavish
variable(global) int MyGlobal = 100

function main()
{
    echo "${MyGlobal}"  ; Accessible everywhere
}
```

**Script:**
```lavish
variable(script) int MyScript = 50
; Accessible to all functions in this script
```

### Variable Access

```lavish
; Get value
echo "${MyVariable}"

; Set value
MyVariable:Set[newValue]

; Increment/Decrement
MyVariable:Inc        ; MyVariable++
MyVariable:Inc[5]     ; MyVariable += 5
MyVariable:Dec        ; MyVariable--
MyVariable:Dec[3]     ; MyVariable -= 3
```

### Type Declaration Patterns

```lavish
; Primitive types
variable int Count = 0
variable float Distance = 123.45
variable string Name = "MyName"
variable bool Flag = TRUE

; Complex types
variable int64 EntityID = 123456789
variable index:entity Entities
variable iterator Entity
variable collection:int Numbers

; Object types
variable obj_MyObject MyObject
variable module MyModule
variable entity MyEntity
```

---

## Data Types

### Primitive Types

| Type | Description | Example |
|------|-------------|---------|
| `int` | 32-bit integer | `variable int X = 42` |
| `int64` | 64-bit integer | `variable int64 ID = 123456789` |
| `float` | Floating point | `variable float F = 3.14` |
| `double` | Double precision | `variable double D = 3.14159` |
| `bool` | Boolean | `variable bool Flag = TRUE` |
| `string` | String | `variable string S = "text"` |

### Complex Types

**Index (Array):**
```lavish
; Index of type
variable index:entity Entities
variable index:int Numbers
variable index:string Names

; Access by numeric index
echo "${Entities[1].Name}"
echo "${Numbers[0]}"

; Size
echo "Count: ${Entities.Used}"
```

**Iterator:**
```lavish
; Iterator for index
variable iterator Entity
Entities:GetIterator[Entity]

; Iterate
if ${Entity:First(exists)}
{
    do
    {
        echo "${Entity.Value.Name}"
    }
    while ${Entity:Next(exists)}
}
```

**Collection:**
```lavish
; Collection with key-value pairs
variable collection:int MyCollection

; Set value
MyCollection:Set[key1, 100]
MyCollection:Set[key2, 200]

; Get value
echo "${MyCollection.Get[key1]}"  ; 100

; Iterate
variable iterator Item
MyCollection:GetIterator[Item]

if ${Item:First(exists)}
{
    do
    {
        echo "Key: ${Item.Key}, Value: ${Item.Value}"
    }
    while ${Item:Next(exists)}
}
```

### Type Conversion

```lavish
; String to int
variable string NumStr = "42"
variable int Num = ${NumStr}

; Int to string
variable int Count = 100
variable string CountStr = "${Count}"

; Float to int (truncate)
variable float F = 3.99
variable int I = ${F}  ; I = 3
```

---

## Operators

### Arithmetic Operators

```lavish
; Math.Calc for expressions
variable int Result

Result:Set[${Math.Calc[5 + 3]}]      ; 8
Result:Set[${Math.Calc[10 - 4]}]     ; 6
Result:Set[${Math.Calc[6 * 7]}]      ; 42
Result:Set[${Math.Calc[15 / 3]}]     ; 5
Result:Set[${Math.Calc[17 % 5]}]     ; 2 (modulo)
Result:Set[${Math.Calc[2 ^ 8]}]      ; 256 (power)

; Complex expressions
Result:Set[${Math.Calc[(5 + 3) * 2 - 1]}]  ; 15
```

### Comparison Operators

```lavish
; Equal
if ${A} == ${B}
if ${Name.Equal["MyName"]}

; Not equal
if ${A} != ${B}
if !${Name.Equal["MyName"]}

; Greater/Less
if ${A} > ${B}
if ${A} < ${B}
if ${A} >= ${B}
if ${A} <= ${B}
```

### Logical Operators

```lavish
; AND
if ${A} > 0 && ${B} < 100

; OR
if ${A} == 1 || ${B} == 2

; NOT
if !${Flag}
```

### String Operators

```lavish
; Concatenate
variable string Full = "${First}${Second}"
Full:Concat[" more text"]

; Equal
if ${Str1.Equal["${Str2}"]}

; Contains
if ${Str.Find["substring"]}

; Length
echo "Length: ${Str.Length}"
```

---

## Control Flow

### If-Else

```lavish
; Simple if
if ${Count} > 0
{
    echo "Count is positive"
}

; If-else
if ${Flag}
{
    echo "True"
}
else
{
    echo "False"
}

; If-elseif-else
if ${Value} < 0
{
    echo "Negative"
}
elseif ${Value} == 0
{
    echo "Zero"
}
else
{
    echo "Positive"
}

; Nested if
if ${A} > 0
{
    if ${B} > 0
    {
        echo "Both positive"
    }
}
```

### Switch-Case

```lavish
; Switch statement
switch ${Value}
{
    case 1
        echo "One"
        break
    case 2
        echo "Two"
        break
    case 3
    case 4
        echo "Three or Four"
        break
    default
        echo "Other"
        break
}
```

### While Loop

```lavish
; While loop
variable int Count = 0
while ${Count} < 10
{
    echo "Count: ${Count}"
    Count:Inc
}

; Infinite loop
while TRUE
{
    ; Loop logic

    ; Break condition
    if ${SomeCondition}
    {
        break
    }
}
```

### Do-While Loop

```lavish
; Do-while loop
variable int Count = 0
do
{
    echo "Count: ${Count}"
    Count:Inc
}
while ${Count} < 10
```

### For Loop (via Iterator)

```lavish
; Iterate index
variable index:entity Entities
EVE:QueryEntities[Entities, "CategoryID = 25"]

variable iterator Entity
Entities:GetIterator[Entity]

if ${Entity:First(exists)}
{
    do
    {
        echo "${Entity.Value.Name}"
    }
    while ${Entity:Next(exists)}
}
```

### Break and Continue

```lavish
; Break - exit loop
while TRUE
{
    if ${ShouldExit}
    {
        break
    }
}

; Continue - next iteration
variable int Index = 0
while ${Index} < 10
{
    Index:Inc

    if ${Index} == 5
    {
        continue  ; Skip 5
    }

    echo "${Index}"
}
```

---

## Functions

### Function Declaration

```lavish
; Basic function
function MyFunction()
{
    echo "Hello from function"
}

; Function with parameters
function Add(int A, int B)
{
    variable int Result = ${Math.Calc[${A} + ${B}]}
    echo "Result: ${Result}"
}

; Function with return (via call)
function GetValue()
{
    return 42
}
```

### Function Calls

```lavish
; Direct call
MyFunction

; Call with parameters
Add 5 10

; Call and capture return
variable int Value
call GetValue Value
echo "Value: ${Value}"

; Call in expression
variable int Sum
call Add 3 7 Sum
```

### Parameters

```lavish
; Positional parameters
function Process(string Name, int Count)
{
    echo "Name: ${Name}, Count: ${Count}"
}

; Variable parameters (...)
function PrintAll(... Args)
{
    echo "Param 1: ${Param[1]}"
    echo "Param 2: ${Param[2]}"
    echo "Total params: ${ParamCount}"
}

; Optional parameters
function DoThing(string Name, int Count = 10)
{
    ; If Count not provided, defaults to 10
}
```

### Return Values

```lavish
; Return via call
function Calculate(int A, int B)
{
    return ${Math.Calc[${A} + ${B}]}
}

; Usage
variable int Result
call Calculate 5 3 Result
echo "Result: ${Result}"  ; 8
```

### Main Function

```lavish
; Entry point
function main(... Args)
{
    echo "Script started"
    echo "Args: ${ParamCount}"

    ; Access parameters
    if ${ParamCount} > 0
    {
        echo "First arg: ${Param[1]}"
    }
}
```

### Exit Function

```lavish
; Called when script ends
function atexit()
{
    echo "Script ending, cleanup..."
}
```

---

## Object-Oriented Programming

### Object Definition

```lavish
; Define object type
objectdef obj_MyObject
{
    ; Variables (members)
    variable string Name
    variable int Count = 0
    variable bool Active = FALSE

    ; Constructor
    method Initialize(string InitName)
    {
        Name:Set["${InitName}"]
        echo "Object initialized: ${Name}"
    }

    ; Methods
    method DoSomething()
    {
        echo "Doing something with ${Name}"
        Count:Inc
    }

    method GetCount()
    {
        return ${Count}
    }

    ; Method with parameters
    method SetData(string NewName, int NewCount)
    {
        Name:Set["${NewName}"]
        Count:Set[${NewCount}]
    }
}
```

### Object Usage

```lavish
; Declare object variable
variable obj_MyObject MyObj

; Initialize
MyObj:Initialize["TestObject"]

; Call methods
MyObj:DoSomething

; Get return value
variable int CurrentCount
call MyObj.GetCount CurrentCount
echo "Count: ${CurrentCount}"

; Access members
echo "Name: ${MyObj.Name}"
echo "Count: ${MyObj.Count}"
echo "Active: ${MyObj.Active}"
```

### This Keyword

```lavish
objectdef obj_Example
{
    variable int Value = 10

    method UseThis()
    {
        ; Access own members with This
        echo "Value: ${This.Value}"
        This.Value:Inc
    }

    method CallOther()
    {
        ; Call own methods with This
        This.UseThis
    }
}
```

### Object Inheritance (Limited)

LavishScript has **no inheritance**. Use composition instead:

```lavish
objectdef obj_Logger
{
    method Log(string Message)
    {
        echo "${Time.Time24}: ${Message}"
    }
}

objectdef obj_MyClass
{
    variable obj_Logger Logger

    method Initialize()
    {
        ; Composition - MyClass has a Logger
    }

    method DoWork()
    {
        Logger:Log["Doing work..."]
    }
}
```

---

## Collections

### Index (Array)

```lavish
; Create index
variable index:int Numbers
variable index:string Names
variable index:entity Entities

; Add items
Numbers:Insert[42]
Numbers:Insert[100]
Names:Insert["Alice"]
Names:Insert["Bob"]

; Access by index (1-based!)
echo "${Numbers[1]}"  ; 42
echo "${Names[2]}"    ; Bob

; Size
echo "Count: ${Numbers.Used}"

; Clear
Numbers:Clear

; Remove
Numbers:Erase[1]  ; Remove first item
```

### Collection (Key-Value)

```lavish
; Create collection
variable collection:int MyCollection

; Set values
MyCollection:Set[key1, 100]
MyCollection:Set[key2, 200]
MyCollection:Set[key3, 300]

; Get values
echo "${MyCollection.Get[key1]}"  ; 100

; Check exists
if ${MyCollection.Element[key2](exists)}
{
    echo "key2 exists"
}

; Erase
MyCollection:Erase[key1]

; Clear all
MyCollection:Clear
```

### Iterator

```lavish
; Get iterator for index
variable index:string Names
Names:Insert["Alice"]
Names:Insert["Bob"]
Names:Insert["Charlie"]

variable iterator Name
Names:GetIterator[Name]

; Iterate
if ${Name:First(exists)}
{
    do
    {
        echo "${Name.Value}"
    }
    while ${Name:Next(exists)}
}

; Iterator for collection
variable collection:int MyCollection
MyCollection:Set[a, 10]
MyCollection:Set[b, 20]

variable iterator Item
MyCollection:GetIterator[Item]

if ${Item:First(exists)}
{
    do
    {
        echo "Key: ${Item.Key}, Value: ${Item.Value}"
    }
    while ${Item:Next(exists)}
}
```

---

## Commands

### Echo (Print)

```lavish
; Simple echo
echo "Hello, World!"

; With variables
echo "Count: ${Count}"

; Concatenation
echo "Name: ${Name}, Age: ${Age}"
```

### Wait

```lavish
; Wait milliseconds
wait 100  ; Wait 100ms

; Wait with condition
wait 50 ${Me.InSpace}  ; Wait up to 5 seconds for Me.InSpace to be true

; Wait forever until condition
wait ${SomeCondition}
```

### Execute

```lavish
; Execute command
execute "echo Hello"

; Execute with variables
execute "echo ${MyVar}"
```

### Redirect

```lavish
; Redirect output to file
redirect "output.txt" echo "This goes to file"

; Append to file
redirect -append "log.txt" echo "This appends to log"

; Redirect variable
variable string Output
redirect Output echo "Captured"
echo "${Output}"  ; Shows captured text
```

### TimedCommand

```lavish
; Execute command after delay
timedcommand 50 echo "After 5 seconds"

; With variable
timedcommand ${DelayMs} MyFunction
```

---

## Math Operations

### Math.Calc

```lavish
; Basic arithmetic
variable int Result

Result:Set[${Math.Calc[5 + 3]}]           ; 8
Result:Set[${Math.Calc[10 - 4]}]          ; 6
Result:Set[${Math.Calc[6 * 7]}]           ; 42
Result:Set[${Math.Calc[15 / 3]}]          ; 5
Result:Set[${Math.Calc[17 % 5]}]          ; 2
Result:Set[${Math.Calc[2 ^ 8]}]           ; 256

; Complex expressions
Result:Set[${Math.Calc[(10 + 5) * 2 / 3]}]  ; 10

; With variables
Result:Set[${Math.Calc[${A} + ${B} * ${C}]}]
```

### Math Functions

```lavish
; Absolute value
variable int Abs = ${Math.Abs[-5]}  ; 5

; Square root
variable float Sqrt = ${Math.Sqrt[16]}  ; 4

; Power
variable int Pow = ${Math.Pow[2, 8]}  ; 256

; Min/Max
variable int Min = ${Math.Min[5, 10]}  ; 5
variable int Max = ${Math.Max[5, 10]}  ; 10

; Random
variable int Rand = ${Math.Rand[100]}  ; 0-99
```

### Math.Distance

```lavish
; 2D distance
variable float Dist2D = ${Math.Distance[${X1}, ${Y1}, ${X2}, ${Y2}]}

; 3D distance
variable float Dist3D = ${Math.Distance[${X1}, ${Y1}, ${Z1}, ${X2}, ${Y2}, ${Z2}]}

; Example - distance to entity
variable float DistToTarget = ${Math.Distance[
    ${Me.ToEntity.X}, ${Me.ToEntity.Y}, ${Me.ToEntity.Z},
    ${Entity[${targetID}].X}, ${Entity[${targetID}].Y}, ${Entity[${targetID}].Z}
]}
```

---

## String Operations

### String Methods

```lavish
variable string Str = "Hello World"

; Length
echo "${Str.Length}"  ; 11

; Uppercase/Lowercase
echo "${Str.Upper}"   ; HELLO WORLD
echo "${Str.Lower}"   ; hello world

; Substring
echo "${Str.Left[5]}"    ; Hello
echo "${Str.Right[5]}"   ; World
echo "${Str.Mid[6, 5]}"  ; World

; Find
echo "${Str.Find["World"]}"  ; 7 (position)

; Replace
echo "${Str.Replace[World, EVE]}"  ; Hello EVE

; Equal
if ${Str.Equal["Hello World"]}
{
    echo "Match!"
}
```

### String Concatenation

```lavish
; Via variable
variable string Full = "${Part1}${Part2}"

; Via Concat method
Full:Set["Hello"]
Full:Concat[" "]
Full:Concat["World"]
echo "${Full}"  ; Hello World
```

### String Conversion

```lavish
; Int to string
variable int Num = 42
variable string NumStr = "${Num}"

; String to int
variable string Str = "123"
variable int Num = ${Str}

; Hex to int
variable string Hex = "0xFF"
variable int Val = ${Hex}  ; 255
```

---

## File I/O

### File Exists

```lavish
; Check if file exists
if ${Script.CurrentDirectory}/myfile.txt(exists)
{
    echo "File exists"
}
```

### Read File

```lavish
; Read entire file
variable string FileContent
redirect FileContent System:Run["type \"${Script.CurrentDirectory}/file.txt\""]
echo "${FileContent}"

; Read line by line (via settingset)
; No native readline in LavishScript - use LavishSettings instead
```

### Write File

```lavish
; Write to file (overwrite)
redirect "${Script.CurrentDirectory}/output.txt" echo "Line 1"

; Append to file
redirect -append "${Script.CurrentDirectory}/output.txt" echo "Line 2"

; Write variable
variable string Data = "Some data"
redirect "${Script.CurrentDirectory}/data.txt" echo "${Data}"
```

### LavishSettings (XML File I/O)

```lavish
; Create setting set
LavishSettings:AddSet[MySettings]

; Add values
LavishSettings[MySettings]:AddSetting[Name, "MyName"]
LavishSettings[MySettings]:AddSetting[Count, 42]

; Save to file
LavishSettings[MySettings]:Export["${Script.CurrentDirectory}/settings.xml"]

; Load from file
LavishSettings[MySettings]:Import["${Script.CurrentDirectory}/settings.xml"]

; Read values
variable string Name = "${LavishSettings[MySettings].FindSetting[Name]}"
variable int Count = ${LavishSettings[MySettings].FindSetting[Count]}

; Clear
LavishSettings[MySettings]:Clear
LavishSettings:Remove[MySettings]
```

---

## Threading

### Script Execution

```lavish
; Run another script
run "path/to/script.iss"

; Run with parameters
run "path/to/script.iss" "param1" "param2"

; Run in background
runscript "path/to/script.iss"

; End script
endscript "scriptname"
```

### Wait for Script

```lavish
; Wait for script to finish
run "myscript.iss"
wait ${Script[myscript](exists)} == FALSE
echo "Script finished"
```

### Script Information

```lavish
; Check if script running
if ${Script[myscript](exists)}
{
    echo "Script is running"
}

; Script path
echo "${Script.CurrentDirectory}"

; Script name
echo "${Script.Filename}"
```

---

## Relay (IPC)

### Relay Command

```lavish
; Send to all other sessions
relay "all other" echo "Hello from ${Me.Name}"

; Send to specific session
relay "CharName" echo "Message for CharName"

; Send to all
relay "all" echo "Broadcast to everyone"

; Execute command remotely
relay "other" Ship:Approach[${targetID}]

; With -noredirect (fire and forget)
relay "all other" -noredirect event[MyEvent]:Execute[param1, param2]
```

### Event Registration (for Relay)

```lavish
; Register event
Event[MyEvent]:AttachAtom[MyEvent_Handler]

; Event handler
atom MyEvent_Handler(string Param1, string Param2)
{
    echo "Event received: ${Param1}, ${Param2}"
}

; Execute event
Event[MyEvent]:Execute["value1", "value2"]

; Detach event
Event[MyEvent]:DetachAtom[MyEvent_Handler]
```

### Common Relay Patterns

**Broadcast to fleet:**
```lavish
; Send command to all fleet members
relay "all other" Event[FleetCommand]:Execute["dock", "${stationID}"]
```

**Request/Response:**
```lavish
; Request
relay "Hauler" Event[RequestOrePickup]:Execute[${Me.Name}, ${Me.SolarSystemID}]

; Response (in Hauler script)
Event[RequestOrePickup]:AttachAtom[HandleOreRequest]

atom HandleOreRequest(string RequesterName, int SolarSystemID)
{
    echo "Pickup request from ${RequesterName} in ${SolarSystemID}"
    ; Process request...
}
```

---

## Events

### Event System

```lavish
; Register event
Event[MyEvent]:AttachAtom[MyHandler]

; Event handler atom
atom MyHandler(string Data)
{
    echo "Event fired: ${Data}"
}

; Execute event
Event[MyEvent]:Execute["test data"]

; Detach event
Event[MyEvent]:DetachAtom[MyHandler]
```

### Built-in Events

**OnFrame:**
```lavish
; Execute every frame
Event[OnFrame]:AttachAtom[OnFrame_Handler]

atom OnFrame_Handler()
{
    ; Called every frame
}
```

**Custom Events:**
```lavish
; Create custom event
Event[OnTargetKilled]:AttachAtom[TargetKilled_Handler]

atom TargetKilled_Handler(int64 EntityID, string EntityName)
{
    echo "Killed: ${EntityName} (${EntityID})"
}

; Fire event
Event[OnTargetKilled]:Execute[${targetID}, "${targetName}"]
```

---

## Script Management

### Script Control

```lavish
; End current script
Script:End

; End other script
endscript "scriptname"

; Pause script
Script:Pause

; Resume script
Script:Resume
```

### Script Information

```lavish
; Script path
echo "${Script.CurrentDirectory}"
echo "${Script.Path}"

; Script name
echo "${Script.Filename}"

; Check if script exists
if ${Script[myscript](exists)}
{
    echo "Script running"
}
```

---

## Common Patterns

### Entity Loop Pattern

```lavish
; Query and iterate entities
variable index:entity NPCs
EVE:QueryEntities[NPCs, "CategoryID = 11 && IsNPC = TRUE && Distance < 100000"]

variable iterator NPC
NPCs:GetIterator[NPC]

if ${NPC:First(exists)}
{
    do
    {
        ; Check entity still exists
        if ${Entity[${NPC.Value.ID}](exists)}
        {
            echo "NPC: ${NPC.Value.Name} (${NPC.Value.Distance}m)"
        }
    }
    while ${NPC:Next(exists)}
}
```

### Wait for Condition Pattern

```lavish
; Wait with timeout
variable int Timeout = 0

while !${DesiredCondition} && ${Timeout} < 100
{
    wait 10
    Timeout:Inc
}

if ${Timeout} >= 100
{
    echo "Timeout!"
}
else
{
    echo "Condition met!"
}
```

### Safe Entity Access Pattern

```lavish
; Always check exists before access
variable int64 targetID = 123456789

if ${Entity[${targetID}](exists)}
{
    if ${Entity[${targetID}].Distance} < 50000
    {
        Entity[${targetID}]:LockTarget
    }
}
else
{
    echo "Entity doesn't exist"
}
```

### Error Handling Pattern

```lavish
; No try-catch in LavishScript, use validation
function SafeOperation(int64 EntityID)
{
    ; Validate inputs
    if ${EntityID} <= 0
    {
        echo "ERROR: Invalid entity ID"
        return FALSE
    }

    ; Check preconditions
    if !${Entity[${EntityID}](exists)}
    {
        echo "ERROR: Entity doesn't exist"
        return FALSE
    }

    ; Perform operation
    Entity[${EntityID}]:LockTarget
    return TRUE
}
```

### State Machine Pattern

```lavish
variable int State = 0

; STATE_IDLE = 0
; STATE_MOVING = 1
; STATE_COMBAT = 2

function main()
{
    while TRUE
    {
        switch ${State}
        {
            case 0
                ; Idle state
                if ${ShouldStartCombat}
                {
                    State:Set[2]
                }
                break

            case 1
                ; Moving state
                if ${Me.ToEntity.Mode} != 3  ; Not warping
                {
                    State:Set[2]
                }
                break

            case 2
                ; Combat state
                call DoCombat
                if ${NoMoreTargets}
                {
                    State:Set[0]
                }
                break
        }

        wait 10
    }
}
```

---

## Best Practices

### 1. Variable Naming

```lavish
; ✅ Good - descriptive names
variable int TargetCount
variable float OptimalRange
variable string PilotName

; ❌ Bad - cryptic names
variable int tc
variable float r
variable string n
```

### 2. Function Organization

```lavish
; ✅ Good - small, focused functions
function GetClosestNPC()
{
    ; Single responsibility
    ; Returns closest NPC
}

function LockTarget(int64 EntityID)
{
    ; Single responsibility
    ; Locks specified target
}

; ❌ Bad - giant function doing everything
function DoEverything()
{
    ; 500 lines of mixed logic
}
```

### 3. Error Checking

```lavish
; ✅ Good - check before use
if ${Entity[${targetID}](exists)}
{
    if ${Entity[${targetID}].Distance} < ${MyShip.MaxTargetRange}
    {
        Entity[${targetID}]:LockTarget
    }
}

; ❌ Bad - assume entity exists
Entity[${targetID}]:LockTarget  ; CRASH if entity doesn't exist
```

### 4. Avoid Global Variables

```lavish
; ❌ Bad - global pollution
variable(global) int Count
variable(global) string Status

; ✅ Good - use objects for state
objectdef obj_BotState
{
    variable int Count
    variable string Status
}

variable(global) obj_BotState BotState
```

### 5. Use Objects for Organization

```lavish
; ✅ Good - organized in object
objectdef obj_Combat
{
    variable int TargetCount

    method FindTargets()
    {
        ; Combat logic
    }

    method LockTarget(int64 EntityID)
    {
        ; Targeting logic
    }
}

; ❌ Bad - scattered functions
function Combat_FindTargets()
function Combat_LockTarget()
variable int Combat_TargetCount
```

### 6. Comment Complex Logic

```lavish
; ✅ Good - explain why
; Calculate angular velocity to avoid damage
; Higher angular = less damage from turrets
variable float AngularVelocity = ${Math.Calc[${MyShip.ToEntity.Velocity} / ${Entity[${targetID}].Distance}]}

if ${AngularVelocity} < 0.01
{
    ; Too slow, orbit closer
    Entity[${targetID}]:Orbit[5000]
}

; ❌ Bad - no explanation
variable float av = ${Math.Calc[${MyShip.ToEntity.Velocity} / ${Entity[${targetID}].Distance}]}
if ${av} < 0.01
{
    Entity[${targetID}]:Orbit[5000]
}
```

### 7. Use Constants

```lavish
; ✅ Good - named constants
variable int WARP_MODE = 3
variable int MAX_TARGETS = 5
variable int MINING_RANGE = 25000

if ${Me.ToEntity.Mode} == ${WARP_MODE}
{
    echo "In warp"
}

; ❌ Bad - magic numbers
if ${Me.ToEntity.Mode} == 3
{
    echo "In warp"
}
```

### 8. Validate Parameters

```lavish
; ✅ Good - validate inputs
function WarpToEntity(int64 EntityID)
{
    if ${EntityID} <= 0
    {
        echo "ERROR: Invalid entity ID: ${EntityID}"
        return FALSE
    }

    if !${Entity[${EntityID}](exists)}
    {
        echo "ERROR: Entity doesn't exist: ${EntityID}"
        return FALSE
    }

    Entity[${EntityID}]:WarpTo[0]
    return TRUE
}

; ❌ Bad - no validation
function WarpToEntity(int64 EntityID)
{
    Entity[${EntityID}]:WarpTo[0]  ; May crash
}
```

---

## Summary

LavishScript is a **powerful but quirky** scripting language designed for EVE Online automation.

### Key Takeaways

1. **Weakly typed** - No compile-time type checking
2. **No inheritance** - Use composition for code reuse
3. **1-based indexing** - Arrays/indices start at 1, not 0
4. **No try-catch** - Validate inputs and check conditions
5. **Limited string operations** - Basic string handling only
6. **Event-driven** - Use events for callbacks and IPC
7. **Relay for IPC** - Cross-session communication
8. **Objects for organization** - Structure code with objectdef

### When to Use vs .NET

**Use LavishScript when:**
- Simple automation (< 1000 lines)
- Quick prototyping
- Learning EVE botting
- Solo developer

**Use .NET when:**
- Complex logic (> 5000 lines)
- Performance critical
- Team development
- Long-term project

### Most Important Concepts

- **Variables**: Declare with `variable`, access with `${Var}`, set with `Var:Set[]`
- **Functions**: Define with `function`, call directly or with `call`
- **Objects**: Define with `objectdef`, use for organization
- **Collections**: Use `index` for arrays, `collection` for key-value
- **Events**: Use for callbacks and relay communication
- **Relay**: Cross-session IPC with `relay "session" command`

---

**Previous Document:** `35_Complete_ISXEVE_Object_Method_Index.md` - ISXEVE object reference

**Next Document:** `37_Common_Code_Snippets.md` - Reusable code patterns

---

*Document complete. LavishScript command reference created for community use.*
