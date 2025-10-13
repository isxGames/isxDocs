# LavishScript Language Complete Reference
## Exhaustive Language Documentation for AI

**Purpose**: Complete, authoritative reference for LavishScript programming language
**Audience**: AI systems mastering LavishScript for EVE bot development
**Wiki Reference**: LavishScriptWiki/ for additional details

---

## Table of Contents

1. [Language Fundamentals](#language-fundamentals)
2. [Variables and Data Types](#variables-and-data-types)
3. [Operators and Expressions](#operators-and-expressions)
4. [Control Flow](#control-flow)
5. [Functions](#functions)
6. [Atoms and Events](#atoms-and-events)
7. [Object System](#object-system)
8. [Collections (Index and Iterator)](#collections-index-and-iterator)
9. [String Manipulation](#string-manipulation)
10. [Math and Calculations](#math-and-calculations)
11. [File I/O](#file-io)
12. [Settings and Configuration](#settings-and-configuration)
13. [Preprocessor Directives](#preprocessor-directives)
14. [Advanced Patterns](#advanced-patterns)
15. [Common Gotchas](#common-gotchas)

---

## Language Fundamentals

### Script Structure

**Basic Script Template**:
```lavishscript
; Comments start with semicolon

; Preprocessor directives (before any code)
#if ${ISXEVE(exists)}
#else
    echo "ISXEVE required"
    Script:End
#endif

; Global variables
variable bool ScriptRunning = TRUE
variable int Counter = 0

; Functions
function Initialize()
{
    echo "Initializing..."
}

function Cleanup()
{
    echo "Cleaning up..."
}

; Main entry point (optional but recommended)
function main()
{
    call Initialize

    while ${ScriptRunning}
    {
        ; Main loop logic
        wait 20
    }

    call Cleanup
}
```

### Comments

**Single-Line Comments**:
```lavishscript
; This is a comment
echo "Hello"  ; Comment after code
```

**Multi-Line Comments**:
```lavishscript
/*
This is a
multi-line comment
*/
```

**Documentation Comments**:
```lavishscript
;===================================
; Function: DoSomething
; Purpose: Does something important
; Parameters: none
; Returns: bool success
;===================================
function DoSomething()
{
    ; ...
}
```

### Syntax Rules

**Case Sensitivity**:
- **Variables**: Case-sensitive (`MyVar` ≠ `myvar`)
- **Commands**: Case-insensitive (`ECHO` = `echo`)
- **Object members**: Case-sensitive (`${Me.Name}` ≠ `${Me.name}`)

**Whitespace**:
- Whitespace mostly ignored (except in strings)
- Newlines optional (can use ; for statement terminator)
- Indentation for readability only

**Statement Termination**:
```lavishscript
; Newline ends statement
variable int X = 5
variable int Y = 10

; Or use semicolon
variable int X = 5; variable int Y = 10
```

---

## Variables and Data Types

### Variable Declaration

**Syntax**:
```lavishscript
variable <type> <name>
variable <type> <name> = <value>
variable(<scope>) <type> <name> = <value>
```

**Scope Modifiers**:
```lavishscript
; LOCAL (default) - function/block scope
variable int LocalVar = 10

; GLOBAL - accessible anywhere in script
variable(global) int GlobalVar = 20

; GLOBALKEEP - survives script reload
variable(globalkeep) string PersistentVar = "Saved"

; SCRIPT - visible to other scripts via Script[] TLO
variable(script) bool ScriptVisible = TRUE

; READONLY - cannot be modified after initialization
variable(readonly) int ConstantValue = 42
```

**Naming Rules**:
- Must start with letter or underscore
- Can contain letters, digits, underscores
- Cannot be reserved word

**Examples**:
```lavishscript
variable int myNumber
variable string _privateName
variable float Value123
variable bool is_ready

; INVALID:
; variable int 123invalid  ; starts with digit
; variable int my-var      ; contains hyphen
```

### Primitive Types

#### bool (Boolean)
```lavishscript
variable bool MyBool = TRUE
variable bool AnotherBool = FALSE

; Boolean methods
${MyBool.Not}               ; Logical NOT
${MyBool.Toggle}            ; Toggle (TRUE<->FALSE)

; Usage in conditions
if ${MyBool}
{
    echo "True"
}
```

#### int (32-bit Integer)
```lavishscript
variable int MyInt = 42
variable int NegativeInt = -100

; Integer methods
${MyInt.Inc}                ; Increment (returns +1)
${MyInt.Dec}                ; Decrement (returns -1)
MyInt:Inc                   ; Increment (modifies variable)
MyInt:Dec                   ; Decrement (modifies variable)
MyInt:Set[value]            ; Set value

; Usage
variable int Counter = 0
Counter:Inc                 ; Counter now 1
Counter:Inc                 ; Counter now 2
echo "${Counter}"           ; Outputs: 2
```

#### int64 (64-bit Integer)
```lavishscript
variable int64 BigNumber = 123456789012345
variable int64 EntityID = ${SomeEntity.ID}

; Same methods as int
BigNumber:Inc
BigNumber:Set[9999999999]
```

**When to Use int64**:
- Entity IDs (always int64)
- Character IDs
- Large ISK amounts
- Timestamps

#### float (Floating Point)
```lavishscript
variable float MyFloat = 3.14159
variable float Distance = ${Entity.Distance}

; Float methods
${MyFloat.Int}              ; Convert to int (truncate)
${MyFloat.Round}            ; Round to nearest int
${MyFloat.Precision[2]}     ; Round to 2 decimal places

; Usage
variable float Distance = 12345.6789
echo "${Distance.Precision[2]}"  ; Outputs: 12345.68
```

#### string (Text)
```lavishscript
variable string MyString = "Hello World"
variable string Empty = ""

; String members (comprehensive list below)
${MyString.Length}          ; Length of string
${MyString.Upper}           ; Uppercase
${MyString.Lower}           ; Lowercase
${MyString.Left[5]}         ; First 5 characters
${MyString.Right[5]}        ; Last 5 characters

; String methods
MyString:Set["New Value"]   ; Set new value
MyString:Concat[" More"]    ; Append to string

; Usage
variable string Name = "CharacterName"
echo "${Name.Upper}"        ; Outputs: CHARACTERNAME
```

### Complex Types

#### index (Collection)
```lavishscript
; Index is like an array/list
variable index:int MyIntList
variable index:string MyStringList
variable index:entity MyEntityList

; Add items
MyIntList:Insert[10]
MyIntList:Insert[20]
MyIntList:Insert[30]

; Access items (1-INDEXED!)
echo "${MyIntList.Get[1]}"  ; Outputs: 10
echo "${MyIntList.Get[2]}"  ; Outputs: 20

; Size
echo "${MyIntList.Used}"    ; Outputs: 3

; Clear all
MyIntList:Clear[]
```

**CRITICAL**: Indexes are **1-indexed**, not 0-indexed!

#### iterator (For Traversing Collections)
```lavishscript
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
        echo "Name: ${Name.Value}"
    }
    while ${Name:Next(exists)}
}
```

#### point3f (3D Coordinate)
```lavishscript
variable point3f Position
Position:Set[100, 200, 300]

echo "X: ${Position.X}"
echo "Y: ${Position.Y}"
echo "Z: ${Position.Z}"

; Distance to another point
variable point3f OtherPos
OtherPos:Set[150, 250, 350]

variable float Dist = ${Position.Distance[${OtherPos}]}
```

#### time (Timestamp)
```lavishscript
variable time Now = ${Time.Timestamp}

; Time members
${Now}                      ; Seconds since epoch
${Now.Date}                 ; Date string
${Now.Time12}               ; Time (12-hour format)
${Now.Time24}               ; Time (24-hour format)

; Usage
echo "Current time: ${Time.Time24}"
```

### Type Conversion

**Implicit Conversion** (often works):
```lavishscript
variable int MyInt = 42
variable string MyString = "${MyInt}"  ; Int to string

variable string NumberStr = "123"
variable int Number = ${NumberStr}     ; String to int
```

**Explicit Conversion**:
```lavishscript
; String to int
variable string Str = "42"
variable int Num = ${Str.Int}

; Float to int
variable float F = 3.14
variable int I = ${F.Int}              ; Truncate
variable int I2 = ${F.Round}           ; Round
```

---

## Operators and Expressions

### Arithmetic Operators

**Math Operations**:
```lavishscript
; In ${} expressions:
${5 + 3}                    ; Addition: 8
${10 - 4}                   ; Subtraction: 6
${6 * 7}                    ; Multiplication: 42
${20 / 4}                   ; Division: 5
${17 % 5}                   ; Modulo: 2

; With variables:
variable int A = 10
variable int B = 3
echo "${Math.Calc[${A} + ${B}]}"  ; 13
echo "${Math.Calc[${A} * ${B}]}"  ; 30
```

**Math.Calc**:
```lavishscript
; Use Math.Calc for complex expressions
variable int Result = ${Math.Calc[${A} + ${B} * ${C} / ${D}]}

; Supports:
; +, -, *, /, %, ( ), etc.
```

### Comparison Operators

**In Conditionals**:
```lavishscript
; Numeric comparison
if ${A} > ${B}              ; Greater than
if ${A} < ${B}              ; Less than
if ${A} >= ${B}             ; Greater or equal
if ${A} <= ${B}             ; Less or equal
if ${A} == ${B}             ; Equal
if ${A} != ${B}             ; Not equal

; String comparison
if ${String1.Equal["${String2}"]}   ; String equality
if ${String1.Find["substring"]}     ; Contains substring
```

### Logical Operators

**Boolean Logic**:
```lavishscript
; AND
if ${A} > 5 && ${B} < 10
{
    echo "Both conditions true"
}

; OR
if ${A} > 100 || ${B} > 100
{
    echo "At least one > 100"
}

; NOT
if !${IsReady}
{
    echo "Not ready"
}

; Complex
if (${A} > 5 && ${B} > 5) || ${C} > 10
{
    echo "Complex condition met"
}
```

---

## Control Flow

### if / elseif / else

**Basic if**:
```lavishscript
if ${Condition}
{
    ; Code if true
}
```

**if-else**:
```lavishscript
if ${MyShip.ShieldPct} < 50
{
    echo "Shields low!"
}
else
{
    echo "Shields OK"
}
```

**if-elseif-else Chain**:
```lavishscript
if ${MyShip.ShieldPct} < 25
{
    echo "CRITICAL shields!"
}
elseif ${MyShip.ShieldPct} < 50
{
    echo "LOW shields"
}
elseif ${MyShip.ShieldPct} < 75
{
    echo "MODERATE shields"
}
else
{
    echo "GOOD shields"
}
```

**Nested if**:
```lavishscript
if ${Me.InSpace}
{
    if ${MyShip.CapacitorPct} > 50
    {
        ; In space AND cap > 50%
        echo "Ready to engage"
    }
}
```

### while Loop

**Basic while**:
```lavishscript
while ${Condition}
{
    ; Loop body
    wait 10  ; CRITICAL - prevents infinite tight loop
}
```

**Common Pattern**:
```lavishscript
variable bool Running = TRUE
while ${Running}
{
    ; Main loop
    call ProcessLogic

    wait 20  ; 2 second pulse
}
```

**Counter Loop**:
```lavishscript
variable int Counter = 0
while ${Counter} < 10
{
    echo "Counter: ${Counter}"
    Counter:Inc
    wait 10
}
```

**Wait for Condition**:
```lavishscript
; Wait for target to be locked
while !${Target.IsLockedTarget} && ${Counter} < 100
{
    wait 10
    Counter:Inc
}

if ${Counter} >= 100
{
    echo "Timeout - target not locked"
}
```

### do-while Loop

**Execute At Least Once**:
```lavishscript
variable int Counter = 0
do
{
    echo "Counter: ${Counter}"
    Counter:Inc
}
while ${Counter} < 5
```

**Common Use: Iterator**:
```lavishscript
variable iterator Item
MyIndex:GetIterator[Item]

if ${Item:First(exists)}
{
    do
    {
        echo "Item: ${Item.Value}"
    }
    while ${Item:Next(exists)}
}
```

### for Loop (Limited Support)

**LavishScript has LIMITED for loop support**

**Workaround with while**:
```lavishscript
; Instead of: for (i = 0; i < 10; i++)
variable int i = 0
while ${i} < 10
{
    echo "i = ${i}"
    i:Inc
}
```

### switch Statement

**Switch (VERY Limited in LavishScript)**

**Workaround with if-elseif**:
```lavishscript
; No native switch, use if-elseif chain
if ${State.Equal["Idle"]}
{
    call HandleIdle
}
elseif ${State.Equal["Combat"]}
{
    call HandleCombat
}
elseif ${State.Equal["Mining"]}
{
    call HandleMining
}
else
{
    echo "Unknown state: ${State}"
}
```

### break and continue

**break** - Exit loop:
```lavishscript
while TRUE
{
    if ${SomeCondition}
    {
        break  ; Exit loop
    }
    wait 10
}
```

**continue** - Skip to next iteration:
```lavishscript
variable int i = 0
while ${i} < 10
{
    i:Inc

    if ${i} == 5
    {
        continue  ; Skip i=5
    }

    echo "i = ${i}"
}
; Outputs: 1,2,3,4,6,7,8,9,10 (skips 5)
```

### return

**Exit Function Early**:
```lavishscript
function CheckTarget()
{
    if !${Target(exists)}
    {
        echo "No target"
        return  ; Exit function
    }

    ; Continue if target exists
    echo "Target: ${Target.Name}"
}
```

**Return Value**:
```lavishscript
function GetDistance()
{
    if ${Target(exists)}
    {
        return ${Target.Distance}
    }
    else
    {
        return 0
    }
}

; Use returned value
variable float Dist = ${GetDistance}
```

---

## Functions

### Function Declaration

**Basic Function**:
```lavishscript
function MyFunction()
{
    echo "Hello from function"
}

; Call it
call MyFunction
```

**Function with Parameters**:
```lavishscript
function Greet(string name)
{
    echo "Hello, ${name}!"
}

call Greet "CharacterName"
; Outputs: Hello, CharacterName!
```

**Multiple Parameters**:
```lavishscript
function Add(int a, int b)
{
    variable int result = ${Math.Calc[${a} + ${b}]}
    echo "${a} + ${b} = ${result}"
    return ${result}
}

variable int sum = ${Add[5, 3]}
; Outputs: 5 + 3 = 8
; sum = 8
```

### Function Return Values

**Returning Values**:
```lavishscript
function GetShieldPercent()
{
    return ${MyShip.ShieldPct}
}

; Use return value
variable float ShieldPct = ${GetShieldPercent}
echo "Shield: ${ShieldPct}%"
```

**Returning Bool**:
```lavishscript
function IsTargetClose()
{
    if ${Target(exists)} && ${Target.Distance} < 10000
    {
        return TRUE
    }
    else
    {
        return FALSE
    }
}

; Use in condition
if ${IsTargetClose}
{
    echo "Target is close"
}
```

### Calling Functions

**call Statement**:
```lavishscript
call MyFunction
call MyFunction "param1" "param2"
```

**Using Return Value**:
```lavishscript
variable int value = ${MyFunction}
variable string text = ${MyFunction["param"]}
```

**Function Chaining**:
```lavishscript
; Functions can call other functions
function A()
{
    echo "In A"
    call B
}

function B()
{
    echo "In B"
    call C
}

function C()
{
    echo "In C"
}

call A
; Outputs:
; In A
; In B
; In C
```

---

## Atoms and Events

### What Are Atoms

**Atoms** = Special functions that can be:
- Called remotely (via relay)
- Attached to events
- Executed asynchronously

### Atom Declaration

**Syntax**:
```lavishscript
atom <name>(<parameters>)
{
    ; Atom body
}

; Or with visibility modifier:
atom(script) <name>(<parameters>)
{
    ; Visible to other scripts
}
```

**Example**:
```lavishscript
atom MyAtom()
{
    echo "Atom executed"
}

; Call locally
MyAtom
```

### Relay Atoms

**For IPC** (Inter-Process Communication):
```lavishscript
; Sender script (Session is1):
relay "all other" ReceiveTargets "${TargetList}"

; Receiver script (Sessions is2, is3, etc.):
atom(script) ReceiveTargets(string targetIDs)
{
    echo "Received targets: ${targetIDs}"
    ; Parse and process targets
}
```

**Atom Visibility**:
- `atom MyAtom()` - Local only
- `atom(script) MyAtom()` - Accessible via relay

### Event Atoms

**Attach Atom to Event**:
```lavishscript
; Define atom to handle event
atom OnTargetChanged()
{
    echo "Target changed to: ${MyShip.ActiveTarget.ToEntity.Name}"
}

; Attach to event
Event[OnActiveTargetChanged]:AttachAtom[OnTargetChanged]

; When event fires, atom executes automatically

; Later: Detach
Event[OnActiveTargetChanged]:DetachAtom[OnTargetChanged]
```

**Common Events**:
- `OnFrame` - Every frame
- `OnActiveTargetChanged` - Active target changed
- `OnMyShipShieldsUpdate` - Shield HP changed
- Many more (see LavishScriptWiki)

---

## Object System

### Object Access

**Get Object Member**:
```lavishscript
; ${Object.Member} returns value
variable string name = ${Me.Name}
variable float distance = ${Entity.Distance}
```

**Call Object Method**:
```lavishscript
; Object:Method[] performs action
Entity:LockTarget[]
Module:Activate[]
MyShip:Open[]
```

### Object Chaining

**Access Nested Members**:
```lavishscript
; Deep chaining
${MyShip.ActiveTarget.ToEntity.Owner.Alliance.Name}

; Step by step (equivalent):
variable module ActiveModule = ${MyShip.ActiveTarget}
variable entity Target = ${ActiveModule.ToEntity}
variable owner TargetOwner = ${Target.Owner}
variable alliance TargetAlliance = ${TargetOwner.Alliance}
variable string AllianceName = ${TargetAlliance.Name}
```

### Existence Checking

**CRITICAL Pattern**:
```lavishscript
; ALWAYS check (exists) for dynamic objects
if ${Entity[${ID}](exists)}
{
    ; Safe to use
    echo "${Entity[${ID}].Name}"
}
else
{
    echo "Entity doesn't exist"
}
```

**Why Necessary**:
```lavishscript
; NPCs can die/despawn during script execution
variable int64 NPCID = ${SomeNPC.ID}

; ... time passes, NPC dies ...

; This would CRASH:
echo "${Entity[${NPCID}].Name}"  ; ERROR - entity doesn't exist

; This is SAFE:
if ${Entity[${NPCID}](exists)}
{
    echo "${Entity[${NPCID}].Name}"
}
else
{
    echo "NPC is dead/gone"
}
```

### NULL Checking

**Objects Can Be NULL**:
```lavishscript
; Some members return NULL if not applicable
variable alliance MyAlliance = ${Me.ToEntity.Owner.Alliance}

; If character has no alliance, MyAlliance is NULL
if ${MyAlliance(exists)}
{
    echo "Alliance: ${MyAlliance.Name}"
}
else
{
    echo "No alliance"
}
```

---

## Collections (Index and Iterator)

### Index (Collection Type)

**Declaration**:
```lavishscript
variable index:int IntIndex
variable index:string StringIndex
variable index:entity EntityIndex
variable index:item ItemIndex
```

**Adding Items**:
```lavishscript
IntIndex:Insert[10]
IntIndex:Insert[20]
IntIndex:Insert[30]

StringIndex:Insert["Alice"]
StringIndex:Insert["Bob"]
```

**Accessing Items**:
```lavishscript
; 1-INDEXED!
echo "${IntIndex.Get[1]}"   ; 10
echo "${IntIndex.Get[2]}"   ; 20
echo "${IntIndex.Get[3]}"   ; 30

echo "${StringIndex.Get[1]}" ; Alice
```

**Size**:
```lavishscript
echo "Count: ${IntIndex.Used}"  ; 3
```

**Clearing**:
```lavishscript
IntIndex:Clear[]
echo "Count: ${IntIndex.Used}"  ; 0
```

**Removing Items**:
```lavishscript
IntIndex:Remove[2]  ; Remove item at index 2
```

### Iterator (Traversal)

**Basic Iterator Pattern**:
```lavishscript
variable index:string Names
Names:Insert["Alice"]
Names:Insert["Bob"]
Names:Insert["Charlie"]

variable iterator Name
Names:GetIterator[Name]

if ${Name:First(exists)}
{
    do
    {
        echo "Name: ${Name.Value}"
    }
    while ${Name:Next(exists)}
}
; Outputs:
; Name: Alice
; Name: Bob
; Name: Charlie
```

**Iterator Methods**:
- `First(exists)` - Move to first item, return true if exists
- `Next(exists)` - Move to next item, return true if exists
- `Value` - Current item value
- `Key` - Current item index (1-indexed)

**Reverse Iteration**:
```lavishscript
if ${Name:Last(exists)}
{
    do
    {
        echo "Name: ${Name.Value}"
    }
    while ${Name:Previous(exists)}
}
; Outputs in reverse order
```

**Conditional Iteration**:
```lavishscript
variable iterator Entity
NPCs:GetIterator[Entity]

if ${Entity:First(exists)}
{
    do
    {
        ; Check if entity still exists
        if !${Entity.Value(exists)}
        {
            echo "Entity despawned during iteration"
            continue
        }

        ; Process entity
        echo "NPC: ${Entity.Value.Name} at ${Entity.Value.Distance}m"

        ; Can exit early
        if ${Entity.Value.Distance} < 10000
        {
            echo "Found close NPC - stopping search"
            break
        }
    }
    while ${Entity:Next(exists)}
}
```

---

## String Manipulation

### String Members

**Case Conversion**:
```lavishscript
variable string Text = "Hello World"
echo "${Text.Upper}"        ; HELLO WORLD
echo "${Text.Lower}"        ; hello world
```

**Substrings**:
```lavishscript
variable string Text = "Hello World"
echo "${Text.Left[5]}"      ; Hello
echo "${Text.Right[5]}"     ; World
echo "${Text.Mid[6, 5]}"    ; World (start at char 6, take 5 chars)
```

**Length**:
```lavishscript
echo "${Text.Length}"       ; 11
```

**Find**:
```lavishscript
; Returns character position (1-indexed) or 0 if not found
echo "${Text.Find["World"]}"  ; 7

if ${Text.Find["World"]}
{
    echo "Contains 'World'"
}
```

**Equal**:
```lavishscript
if ${Text.Equal["Hello World"]}
{
    echo "Strings are equal"
}

; Case-insensitive:
if ${Text.EqualCS["hello world"]}
{
    echo "Case-sensitive equal"
}
```

**Compare**:
```lavishscript
; Returns 0 if equal, -1 if less, 1 if greater
variable int result = ${Text.Compare["Hello"]}
```

### String Methods

**Set**:
```lavishscript
Text:Set["New Value"]
echo "${Text}"              ; New Value
```

**Concat** (Append):
```lavishscript
Text:Set["Hello"]
Text:Concat[" World"]
echo "${Text}"              ; Hello World
```

**Insert**:
```lavishscript
Text:Set["Hello World"]
Text:Insert[6, "Beautiful "]
echo "${Text}"              ; Hello Beautiful World
```

**Replace**:
```lavishscript
Text:Set["Hello World"]
Text:Replace["World", "Universe"]
echo "${Text}"              ; Hello Universe
```

**ReplaceSubString**:
```lavishscript
; Replace all occurrences
Text:Set["one two one three one"]
Text:ReplaceSubString["one", "1"]
echo "${Text}"              ; 1 two 1 three 1
```

### String Building

**Concatenation with Variables**:
```lavishscript
variable string name = "CharacterName"
variable int age = 42

variable string message = "${name} is ${age} years old"
echo "${message}"
; Outputs: CharacterName is 42 years old
```

**Multi-Line**:
```lavishscript
variable string multiline = "Line 1\nLine 2\nLine 3"
echo "${multiline}"
; Outputs:
; Line 1
; Line 2
; Line 3
```

---

## Math and Calculations

### Math Object

**Math.Calc**:
```lavishscript
; Evaluate mathematical expression
variable int result = ${Math.Calc[5 + 3 * 2]}  ; 11

; With variables
variable int a = 10
variable int b = 5
variable int sum = ${Math.Calc[${a} + ${b}]}   ; 15
variable int product = ${Math.Calc[${a} * ${b}]}  ; 50
```

**Operators in Math.Calc**:
- `+` - Addition
- `-` - Subtraction
- `*` - Multiplication
- `/` - Division
- `%` - Modulo
- `(` `)` - Grouping

**Example**:
```lavishscript
variable float distance = ${Entity.Distance}
variable float speed = 500.0
variable float time = ${Math.Calc[${distance} / ${speed}]}
echo "ETA: ${time} seconds"
```

### Math Functions

**Abs** (Absolute Value):
```lavishscript
variable int negative = -42
echo "${Math.Abs[${negative}]}"  ; 42
```

**Sqrt** (Square Root):
```lavishscript
variable float squareRoot = ${Math.Sqrt[16]}  ; 4.0
```

**Distance Calculation** (3D):
```lavishscript
; Distance between two points
variable float x1 = 100
variable float y1 = 200
variable float z1 = 300

variable float x2 = 150
variable float y2 = 250
variable float z2 = 350

variable float dx = ${Math.Calc[${x2} - ${x1}]}
variable float dy = ${Math.Calc[${y2} - ${y1}]}
variable float dz = ${Math.Calc[${z2} - ${z1}]}

variable float distSquared = ${Math.Calc[${dx}*${dx} + ${dy}*${dy} + ${dz}*${dz}]}
variable float distance = ${Math.Sqrt[${distSquared}]}
```

**Random Numbers**:
```lavishscript
; Random int between 0 and N-1
variable int randomValue = ${Math.Rand[100]}  ; 0-99

; Random float between 0.0 and 1.0
; (Not directly supported - use workaround)
variable float randomFloat = ${Math.Calc[${Math.Rand[1000]} / 1000.0]}
```

---

## File I/O

### File Object

**Declaration**:
```lavishscript
variable file MyFile = "path/to/file.txt"
```

**Opening Files**:
```lavishscript
; Open for reading (default)
MyFile:Open[]

; Open for writing (overwrites)
MyFile:Open[write]

; Open for appending
MyFile:Open[append]
```

**Reading Files**:
```lavishscript
variable file MyFile = "config.txt"
MyFile:Open[]

variable string line
while ${MyFile:Read[line]}
{
    echo "Line: ${line}"
}

MyFile:Close[]
```

**Writing Files**:
```lavishscript
variable file LogFile = "botlog.txt"
LogFile:Open[append]

LogFile:Write["Bot started at ${Time.Time24}\n"]
LogFile:Write["Shield: ${MyShip.ShieldPct}%\n"]

LogFile:Close[]
```

**File Operations**:
```lavishscript
; Seek to position
MyFile:Seek[0]              ; Beginning of file

; Truncate file
MyFile:Truncate[]           ; Clear file content

; Get size
variable int size = ${MyFile.Size}
```

### File Paths

**Relative Paths**:
```lavishscript
; Relative to InnerSpace/Scripts/
variable file F = "MyBot/config.txt"

; Also works:
variable file F = "./MyBot/config.txt"
```

**Absolute Paths**:
```lavishscript
variable file F = "C:/Path/To/File.txt"
```

---

## Settings and Configuration

### SettingSet (XML Configuration)

**Declaration**:
```lavishscript
variable(globalkeep) settingsetref MySettings
```

**Creating Settings**:
```lavishscript
; Initialize
MySettings:Set[MyBot.Config, XML]

; Add settings
MySettings:Add[MasterCharacter, "CharacterName"]
MySettings:Add[FollowDistance, 1000]
MySettings:Add[AutoTarget, TRUE]

; Save to file
MySettings:Save["Scripts/MyBot/config.xml"]
```

**Loading Settings**:
```lavishscript
; Load from file
MySettings:Load["Scripts/MyBot/config.xml"]

; Retrieve values
variable string master = "${MySettings.Get[MasterCharacter]}"
variable int distance = ${MySettings.Get[FollowDistance]}
variable bool autoTarget = ${MySettings.Get[AutoTarget]}
```

**Nested Settings**:
```lavishscript
; Create sub-settings
MySettings:AddSet[Targeting]
MySettings.FindSet[Targeting]:Add[MaxTargets, 5]
MySettings.FindSet[Targeting]:Add[PriorityType, "Closest"]

; Retrieve
variable int maxTargets = ${MySettings.FindSet[Targeting].Get[MaxTargets]}
```

**Settings Persistence**:
```lavishscript
; globalkeep makes settings survive script reload
variable(globalkeep) settingsetref Config

; Settings persist even if script restarts
```

---

## Preprocessor Directives

### #if / #else / #endif

**Conditional Compilation**:
```lavishscript
#if ${ISXEVE(exists)}
    echo "ISXEVE is loaded"
#else
    echo "ISXEVE not loaded - exiting"
    Script:End
#endif
```

**Checking Variables**:
```lavishscript
#if ${Config(exists)}
    echo "Config already loaded"
#else
    ; Initialize config
    variable(global) settingsetref Config
#endif
```

### #include

**Include Other Files**:
```lavishscript
; Include another script file
#include MyBot/Library/Targeting.iss
#include MyBot/Library/Movement.iss

; Now can use functions from those files
call LockTarget
call ApproachEntity
```

**Include Path**:
- Relative to current script directory
- Or relative to InnerSpace/Scripts/

### #define

**Define Constants** (limited support):
```lavishscript
#define MAX_TARGETS 5

; Use in code
variable int maxTargets = MAX_TARGETS
```

---

## Advanced Patterns

### State Machines

**Enum-Style States**:
```lavishscript
; Define states as global variables (poor man's enum)
variable(global) string STATE_IDLE = "Idle"
variable(global) string STATE_COMBAT = "Combat"
variable(global) string STATE_MINING = "Mining"
variable(global) string STATE_RETURNING = "Returning"

variable string CurrentState = "${STATE_IDLE}"

; State machine loop
while ${ScriptRunning}
{
    if ${CurrentState.Equal["${STATE_IDLE}"]}
    {
        call HandleIdle
    }
    elseif ${CurrentState.Equal["${STATE_COMBAT}"]}
    {
        call HandleCombat
    }
    elseif ${CurrentState.Equal["${STATE_MINING}"]}
    {
        call HandleMining
    }
    elseif ${CurrentState.Equal["${STATE_RETURNING}"]}
    {
        call HandleReturning
    }

    wait 20
}

; State transition
function EnterCombat()
{
    echo "Entering combat state"
    CurrentState:Set["${STATE_COMBAT}"]
}
```

### Timers and Cooldowns

**Timestamp-Based Timers**:
```lavishscript
variable int64 LastAction = 0

; Check if enough time passed
variable int64 Now = ${Time.Timestamp}
if ${Math.Calc[${Now} - ${LastAction}]} > 5
{
    ; More than 5 seconds since last action
    call DoAction
    LastAction:Set[${Now}]
}
```

**Counter-Based Timers** (Frame counting):
```lavishscript
variable int CooldownCounter = 0
variable int COOLDOWN_PULSES = 10  ; 10 pulses = ~20 seconds at wait 20

; Main loop
while ${ScriptRunning}
{
    ; Decrement cooldown
    if ${CooldownCounter} > 0
    {
        CooldownCounter:Dec
    }

    ; Check if off cooldown
    if ${CooldownCounter} == 0
    {
        call DoAction
        CooldownCounter:Set[${COOLDOWN_PULSES}]  ; Reset cooldown
    }

    wait 20
}
```

### Retry Logic

**Retry with Timeout**:
```lavishscript
function LockTargetWithRetry(int64 entityID, int maxRetries)
{
    variable int retries = 0

    while ${retries} < ${maxRetries}
    {
        if !${Entity[${entityID}](exists)}
        {
            echo "Entity doesn't exist"
            return FALSE
        }

        Entity[${entityID}]:LockTarget[]
        wait 10

        ; Wait for lock (max 15 seconds)
        variable int lockWait = 0
        while !${Entity[${entityID}].IsLockedTarget} && ${lockWait} < 150
        {
            wait 10
            lockWait:Inc
        }

        ; Check if locked
        if ${Entity[${entityID}].IsLockedTarget}
        {
            echo "Target locked successfully"
            return TRUE
        }

        ; Lock failed, retry
        echo "Lock attempt ${retries} failed, retrying..."
        retries:Inc
        wait 20
    }

    echo "Failed to lock after ${maxRetries} attempts"
    return FALSE
}
```

---

## Common Gotchas

### 1. Arrays Are 1-Indexed

```lavishscript
; WRONG - 0 is invalid index
echo "${MyIndex.Get[0]}"  ; ERROR or returns NULL

; CORRECT - start at 1
echo "${MyIndex.Get[1]}"  ; First element
```

### 2. Must Use ${} For Values

```lavishscript
; WRONG - won't work
variable int X = MyVar

; CORRECT
variable int X = ${MyVar}
```

### 3. String Comparison

```lavishscript
; WRONG - doesn't work
if ${String1} == ${String2}

; CORRECT
if ${String1.Equal["${String2}"]}
```

### 4. Wait Is Mandatory

```lavishscript
; WRONG - infinite tight loop, maxes CPU
while TRUE
{
    call DoStuff
}

; CORRECT - wait between iterations
while TRUE
{
    call DoStuff
    wait 10  ; CRITICAL
}
```

### 5. Object Existence

```lavishscript
; WRONG - crashes if entity doesn't exist
echo "${Entity[${ID}].Name}"

; CORRECT - check existence first
if ${Entity[${ID}](exists)}
{
    echo "${Entity[${ID}].Name}"
}
```

### 6. Case Sensitivity

```lavishscript
; Variables are case-sensitive
variable int MyVar = 10
echo "${myvar}"  ; WRONG - undefined
echo "${MyVar}"  ; CORRECT
```

### 7. Type Mismatches

```lavishscript
; May cause issues
variable int MyInt = 42
variable string MyString = "Hello"
variable int BadIdea = ${MyString}  ; Can't convert

; Be explicit
variable string NumAsString = "${MyInt}"  ; int to string
variable int StringAsInt = ${MyString.Int}  ; string to int (if valid)
```

---

## Summary

**LavishScript Key Concepts**:

1. **Interpreted Language** - Runs in LavishScript engine
2. **Object-Oriented** - Everything is an object
3. **Dynamic Typing** - Variables can change type
4. **1-Indexed Collections** - Arrays/indexes start at 1, not 0
5. **${} for Values** - Use ${} to get variable/object values
6. **: for Actions** - Use : to call methods
7. **wait Is Mandatory** - Must use in loops to prevent CPU lockup
8. **Existence Checking** - Always check (exists) for dynamic objects
9. **Atoms** - Special functions for events and relay
10. **No Native Classes** - Objects provided by extensions

**Critical Patterns**:
- Main loop with wait
- Iterator pattern for collections
- Existence checking before object access
- Retry logic with timeouts
- State machines for bot logic
- Settings for configuration persistence

---

## Next Steps

**Now you have comprehensive LavishScript knowledge. Next:**
- Read `05_LavishScript_Syntax_and_Patterns.md` for practical coding patterns
- Read `06_Variables_DataTypes_and_Scope.md` for deeper variable understanding
- Study example scripts to see language in action
- Practice writing small test scripts

---

**END OF FILE**
**Next File**: 05_LavishScript_Syntax_and_Patterns.md
