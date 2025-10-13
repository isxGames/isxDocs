# Variables, Data Types, and Scope
## Complete Guide to LavishScript Variable System

**Purpose**: Exhaustive coverage of variables, types, and scope in LavishScript
**Audience**: AI mastering LavishScript variable mechanics
**Prerequisites**: Read files 04 and 05

---

## Table of Contents

1. [Variable Declaration Deep Dive](#variable-declaration-deep-dive)
2. [Scope Modifiers Explained](#scope-modifiers-explained)
3. [Primitive Types Complete Reference](#primitive-types-complete-reference)
4. [Complex Types Mastery](#complex-types-mastery)
5. [Type Conversion and Coercion](#type-conversion-and-coercion)
6. [Variable Lifetime](#variable-lifetime)
7. [Memory and Performance](#memory-and-performance)
8. [Best Practices](#best-practices)

---

## Variable Declaration Deep Dive

### Full Syntax

**Complete Declaration Syntax**:
```lavishscript
variable(<scope_modifier>) <type> <name>
variable(<scope_modifier>) <type> <name> = <initial_value>
```

**Components**:
- `<scope_modifier>`: Optional - local, global, globalkeep, script, readonly
- `<type>`: Required - int, string, entity, etc.
- `<name>`: Required - variable name
- `<initial_value>`: Optional - initial value

### Declaration Examples

```lavishscript
; Minimal (local scope, default value)
variable int MyNumber

; With initial value
variable int MyNumber = 42

; With scope modifier
variable(global) int GlobalNumber = 100

; Multiple modifiers (rare)
variable(readonly) int Constant = 999

; Complex types
variable index:entity EntityList
variable iterator MyIterator
variable settingsetref Config
```

### Default Values

**Types Have Default Values**:
```lavishscript
variable int MyInt              ; Default: 0
variable bool MyBool            ; Default: FALSE
variable string MyString        ; Default: "" (empty string)
variable float MyFloat          ; Default: 0.0
variable int64 MyInt64          ; Default: 0

; Complex types default to NULL/uninitialized
variable entity MyEntity        ; Default: NULL
variable index:int MyIndex      ; Default: empty index
```

---

## Scope Modifiers Explained

### Local Scope (Default)

**Characteristics**:
- Default if no modifier specified
- Function/block scope
- Destroyed when function exits
- NOT accessible from other functions

**Example**:
```lavishscript
function FunctionA()
{
    variable int LocalVar = 10  ; Local to FunctionA
    echo "${LocalVar}"          ; Works
}

function FunctionB()
{
    echo "${LocalVar}"          ; ERROR - LocalVar doesn't exist here
}
```

**Block Scope**:
```lavishscript
function TestScope()
{
    variable int Outer = 10

    if TRUE
    {
        variable int Inner = 20
        echo "${Outer}"         ; Works - can see outer
        echo "${Inner}"         ; Works - in same block
    }

    echo "${Outer}"             ; Works
    echo "${Inner}"             ; ERROR - Inner out of scope
}
```

### Global Scope

**Characteristics**:
- Accessible anywhere in script
- Survives until script ends
- Shared across all functions
- Can be modified by any function

**Declaration**:
```lavishscript
variable(global) int GlobalCounter = 0
variable(global) string GlobalState = "Idle"
```

**Usage Across Functions**:
```lavishscript
variable(global) int GlobalCounter = 0

function IncrementCounter()
{
    GlobalCounter:Inc
    echo "Counter: ${GlobalCounter}"
}

function main()
{
    call IncrementCounter  ; Counter: 1
    call IncrementCounter  ; Counter: 2
    call IncrementCounter  ; Counter: 3
}
```

**Common Use Cases**:
```lavishscript
; State management
variable(global) string CurrentState = "Idle"

; Shared data
variable(global) int64 ActiveTargetID = 0

; Configuration
variable(global) int MaxTargets = 5

; Script control
variable(global) bool ScriptRunning = TRUE
```

### GlobalKeep Scope

**Characteristics**:
- Like global, but **persists across script reloads**
- Survives `Script:End` and `Script:Start`
- Used for settings and persistent state
- Expensive to create (allocates differently)

**Declaration**:
```lavishscript
variable(globalkeep) settingsetref Config
variable(globalkeep) int TimesRun = 0
variable(globalkeep) string LastSession = ""
```

**Persistence Demonstration**:
```lavishscript
; First run:
variable(globalkeep) int TimesRun = 0
TimesRun:Inc
echo "Times run: ${TimesRun}"  ; Outputs: 1

; Script ends, user runs again:
; Variable still exists with value 1
TimesRun:Inc
echo "Times run: ${TimesRun}"  ; Outputs: 2
```

**Typical Usage**:
```lavishscript
; Configuration object
variable(globalkeep) settingsetref BotConfig

; Session tracking
variable(globalkeep) int SessionCount = 0

; Persistent stats
variable(globalkeep) int64 TotalISKEarned = 0
```

### Script Scope

**Characteristics**:
- Accessible via `Script[scriptname]` TLO
- Enables inter-script communication
- Used with relay system
- Visible to other running scripts

**Declaration**:
```lavishscript
variable(script) bool ReadyState = FALSE
variable(script) string CurrentActivity = "Idle"
variable(script) int64 ActiveTarget = 0
```

**Accessing from Another Script**:
```lavishscript
; In Script A (MyBot.iss):
variable(script) bool IsReady = TRUE

; In Script B (Monitor.iss):
if ${Script[MyBot](exists)}
{
    echo "MyBot ready state: ${Script[MyBot].VariableScope.IsReady}"
}
```

**With Relay System**:
```lavishscript
; Script variables used for relay data passing
variable(global) string YamfaRelayedTargets
variable(global) int64 YamfaRelayedActive

; Relay atom updates these
atom(script) ReceiveTargets(string targets, int64 active)
{
    YamfaRelayedTargets:Set["${targets}"]
    YamfaRelayedActive:Set[${active}]
}
```

### ReadOnly Scope

**Characteristics**:
- Cannot be modified after initialization
- Const/immutable
- Good for constants
- Compile-time enforcement (limited)

**Declaration**:
```lavishscript
variable(readonly) int MAX_TARGETS = 10
variable(readonly) string VERSION = "1.0.0"
variable(readonly) float PI = 3.14159
```

**Attempting to Modify** (will error):
```lavishscript
variable(readonly) int Constant = 100
Constant:Set[200]  ; ERROR - cannot modify readonly variable
```

---

## Primitive Types Complete Reference

### bool (Boolean)

**Size**: 1 byte (effectively)
**Range**: TRUE or FALSE
**Default**: FALSE

**Declaration**:
```lavishscript
variable bool MyBool = TRUE
variable bool AnotherBool = FALSE
```

**Members**:
```lavishscript
${MyBool.Not}           ; Logical NOT (returns opposite)
${MyBool.Toggle}        ; Toggle value (returns opposite)
```

**Methods**:
```lavishscript
MyBool:Set[TRUE]        ; Set to TRUE
MyBool:Set[FALSE]       ; Set to FALSE
MyBool:Toggle           ; Toggle value (modifies variable)
```

**Type Conversion**:
```lavishscript
; Int to bool
variable int Number = 1
variable bool BoolValue = ${Number}  ; Non-zero = TRUE, zero = FALSE

; String to bool
variable string Str = "TRUE"
variable bool BoolValue = ${Str}     ; "TRUE" or "FALSE" strings work
```

**Usage in Conditions**:
```lavishscript
variable bool IsReady = TRUE

if ${IsReady}
{
    echo "Ready"
}

if !${IsReady}
{
    echo "Not ready"
}

; With method
if ${SomeCondition.Not}
{
    echo "Condition is false"
}
```

### int (32-bit Integer)

**Size**: 4 bytes
**Range**: -2,147,483,648 to 2,147,483,647
**Default**: 0

**Declaration**:
```lavishscript
variable int MyInt = 42
variable int Negative = -100
variable int Zero = 0
```

**Members**:
```lavishscript
${MyInt.Inc}            ; Returns value + 1 (doesn't modify)
${MyInt.Dec}            ; Returns value - 1 (doesn't modify)
```

**Methods**:
```lavishscript
MyInt:Set[42]           ; Set value
MyInt:Inc               ; Increment (modifies variable)
MyInt:Dec               ; Decrement (modifies variable)
MyInt:Inc[5]            ; Increment by 5
MyInt:Dec[3]            ; Decrement by 3
```

**Arithmetic**:
```lavishscript
variable int A = 10
variable int B = 3

; In expressions (requires Math.Calc for complex operations)
variable int Sum = ${Math.Calc[${A} + ${B}]}        ; 13
variable int Difference = ${Math.Calc[${A} - ${B}]} ; 7
variable int Product = ${Math.Calc[${A} * ${B}]}    ; 30
variable int Quotient = ${Math.Calc[${A} / ${B}]}   ; 3
variable int Remainder = ${Math.Calc[${A} % ${B}]}  ; 1
```

**Overflow**:
```lavishscript
variable int Max = 2147483647
Max:Inc
echo "${Max}"  ; Wraps to -2147483648
```

### int64 (64-bit Integer)

**Size**: 8 bytes
**Range**: -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807
**Default**: 0

**When to Use**:
- Entity IDs (always int64)
- Character IDs
- Large ISK amounts
- Timestamps

**Declaration**:
```lavishscript
variable int64 EntityID = ${SomeEntity.ID}
variable int64 BigNumber = 123456789012345
variable int64 Wallet = ${Me.Wallet}
```

**Same Methods as int**:
```lavishscript
BigNumber:Set[9999999999]
BigNumber:Inc
BigNumber:Dec
```

**Critical Use Cases**:
```lavishscript
; Storing entity IDs
variable int64 TargetID = ${Target.ID}

; Later retrieval
if ${Entity[${TargetID}](exists)}
{
    echo "Target still exists: ${Entity[${TargetID}].Name}"
}

; ISK tracking
variable int64 StartWallet = ${Me.Wallet}
; ... bot runs ...
variable int64 EndWallet = ${Me.Wallet}
variable int64 Profit = ${Math.Calc[${EndWallet} - ${StartWallet}]}
```

### float (Floating Point)

**Size**: 4 bytes (32-bit float)
**Range**: ±3.4 × 10^38 (approximately)
**Precision**: ~7 decimal digits
**Default**: 0.0

**Declaration**:
```lavishscript
variable float MyFloat = 3.14159
variable float Distance = ${Entity.Distance}
variable float Percentage = 0.75
```

**Members**:
```lavishscript
${MyFloat.Int}              ; Convert to int (truncate)
${MyFloat.Round}            ; Round to nearest int
${MyFloat.Precision[2]}     ; Round to 2 decimal places
${MyFloat.Ceiling}          ; Round up
${MyFloat.Floor}            ; Round down
```

**Precision Example**:
```lavishscript
variable float Distance = 12345.6789
echo "${Distance.Precision[0]}"  ; 12346
echo "${Distance.Precision[1]}"  ; 12345.7
echo "${Distance.Precision[2]}"  ; 12345.68
echo "${Distance.Precision[3]}"  ; 12345.679
```

**Rounding**:
```lavishscript
variable float Value = 3.7
echo "${Value.Floor}"    ; 3
echo "${Value.Round}"    ; 4
echo "${Value.Ceiling}"  ; 4

variable float Value2 = 3.2
echo "${Value2.Floor}"   ; 3
echo "${Value2.Round}"   ; 3
echo "${Value2.Ceiling}" ; 4
```

**Arithmetic**:
```lavishscript
variable float A = 10.5
variable float B = 3.2

variable float Sum = ${Math.Calc[${A} + ${B}]}  ; 13.7
variable float Product = ${Math.Calc[${A} * ${B}]}  ; 33.6
```

**Common Uses**:
```lavishscript
; Distances
variable float Distance = ${Entity.Distance}

; HP/Cap percentages
variable float ShieldPct = ${MyShip.ShieldPct}

; Ranges
variable float OptimalRange = ${Module.OptimalRange}
```

### string (Text)

**Size**: Variable (dynamic)
**Max Length**: Effectively unlimited
**Default**: "" (empty string)

**Declaration**:
```lavishscript
variable string MyString = "Hello World"
variable string Empty = ""
variable string WithVariable = "${CharName} is in ${SystemName}"
```

**Members** (comprehensive):
```lavishscript
${MyString.Length}              ; String length
${MyString.Upper}               ; UPPERCASE
${MyString.Lower}               ; lowercase
${MyString.Token[1,":"}         ; First token split by ":"
${MyString.Token[2,":"}         ; Second token split by ":"
${MyString.Left[5]}             ; First 5 chars
${MyString.Right[5]}            ; Last 5 chars
${MyString.Mid[3,5]}            ; 5 chars starting at position 3
${MyString.Find["substring"]}   ; Position of substring (1-indexed, 0 if not found)
${MyString.Equal["other"]}      ; Case-sensitive equality
${MyString.NotEqual["other"]}   ; Case-sensitive inequality
${MyString.Compare["other"]}    ; Comparison (-1, 0, 1)
${MyString.Int}                 ; Convert to int
${MyString.Float}               ; Convert to float
```

**Methods**:
```lavishscript
MyString:Set["New Value"]               ; Set value
MyString:Concat[" More Text"]           ; Append
MyString:Insert[5, "INSERT"]            ; Insert at position
MyString:Replace["old", "new"]          ; Replace first occurrence
MyString:ReplaceSubString["old", "new"] ; Replace all occurrences
MyString:Clear[]                        ; Empty string
```

**String Building**:
```lavishscript
variable string Message = ""
Message:Set["Bot: "]
Message:Concat["${Me.Name}"]
Message:Concat[" in system "]
Message:Concat["${Me.SolarSystemID}"]
echo "${Message}"
; Output: Bot: CharacterName in system 30000142
```

**Token Splitting**:
```lavishscript
variable string CSV = "Apple,Banana,Cherry"
echo "${CSV.Token[1,","]}"  ; Apple
echo "${CSV.Token[2,","]}"  ; Banana
echo "${CSV.Token[3,","]}"  ; Cherry

; Common use: Parsing relay data
variable string TargetList = "12345,67890,11223"
variable string FirstID = "${TargetList.Token[1,","]}"
variable string SecondID = "${TargetList.Token[2,","]}"
```

**String Searching**:
```lavishscript
variable string Name = "Caldari Navy Hookbill"

if ${Name.Find["Navy"]}
{
    echo "This is a Navy ship (position: ${Name.Find["Navy"]})"
}

if ${Name.Find["Guristas"]} == 0
{
    echo "Not a Guristas ship"
}
```

**String Comparison**:
```lavishscript
variable string A = "Hello"
variable string B = "hello"

if ${A.Equal["${B}"]}
{
    echo "Equal (case-sensitive)"
}
else
{
    echo "Not equal - different case"
}

; Case-insensitive comparison (use Upper or Lower)
if ${A.Upper.Equal["${B.Upper}"]}
{
    echo "Equal (case-insensitive)"
}
```

---

## Complex Types Mastery

### index (Collection/Array)

**What It Is**:
- Dynamic array/list
- Can hold any type
- 1-indexed (CRITICAL!)
- Ordered collection

**Declaration**:
```lavishscript
variable index:int IntList
variable index:string StringList
variable index:entity EntityList
variable index:int64 IDList
```

**Adding Items**:
```lavishscript
IntList:Insert[10]
IntList:Insert[20]
IntList:Insert[30]

; Can insert at position
IntList:Insert[2, 15]  ; Insert 15 at position 2
; Now: [10, 15, 20, 30]
```

**Accessing Items**:
```lavishscript
; 1-INDEXED!
echo "${IntList.Get[1]}"  ; First item: 10
echo "${IntList.Get[2]}"  ; Second item: 15
echo "${IntList.Get[3]}"  ; Third item: 20

; Out of bounds returns NULL/0
echo "${IntList.Get[999]}"  ; 0 (doesn't exist)
```

**Members**:
```lavishscript
${IntList.Used}         ; Number of items
${IntList.Size}         ; Allocated size (capacity)
${IntList.Get[N]}       ; Get item at index N (1-indexed)
```

**Methods**:
```lavishscript
IntList:Insert[value]           ; Add to end
IntList:Insert[pos, value]      ; Insert at position
IntList:Set[pos, value]         ; Replace at position
IntList:Remove[pos]             ; Remove at position
IntList:Clear[]                 ; Remove all items
IntList:Collapse[]              ; Remove NULLs and compact
IntList:Reverse[]               ; Reverse order
IntList:Sort[]                  ; Sort (ascending)
```

**Iteration Pattern**:
```lavishscript
variable index:string Names
Names:Insert["Alice"]
Names:Insert["Bob"]
Names:Insert["Charlie"]

variable int i = 1
while ${i} <= ${Names.Used}
{
    echo "Name ${i}: ${Names.Get[${i}]}"
    i:Inc
}
```

**Common Use**:
```lavishscript
; Store entity IDs for processing
variable index:int64 TargetIDList

variable index:entity NPCs
EVE:QueryEntities[NPCs, "IsNPC"]

variable iterator NPC
NPCs:GetIterator[NPC]

if ${NPC:First(exists)}
{
    do
    {
        TargetIDList:Insert[${NPC.Value.ID}]
    }
    while ${NPC:Next(exists)}
}

; Later: Process stored IDs
variable int i = 1
while ${i} <= ${TargetIDList.Used}
{
    variable int64 ID = ${TargetIDList.Get[${i}]}
    if ${Entity[${ID}](exists)}
    {
        echo "Processing: ${Entity[${ID}].Name}"
    }
    i:Inc
}
```

### iterator (Traversal Object)

**What It Is**:
- Used to iterate through indexes
- Cursor-based traversal
- Stateful (tracks position)

**Declaration**:
```lavishscript
variable iterator MyIterator
```

**Getting Iterator from Index**:
```lavishscript
variable index:string Names
Names:Insert["Alice"]
Names:Insert["Bob"]

variable iterator NameIter
Names:GetIterator[NameIter]
```

**Iteration Pattern**:
```lavishscript
if ${NameIter:First(exists)}
{
    do
    {
        echo "Name: ${NameIter.Value}"
        echo "Index: ${NameIter.Key}"
    }
    while ${NameIter:Next(exists)}
}
```

**Members**:
```lavishscript
${Iterator.Value}       ; Current item value
${Iterator.Key}         ; Current item index (1-based)
```

**Methods**:
```lavishscript
Iterator:First(exists)      ; Move to first, return true if exists
Iterator:Next(exists)       ; Move to next, return true if exists
Iterator:Last(exists)       ; Move to last, return true if exists
Iterator:Previous(exists)   ; Move to previous, return true if exists
Iterator:Reset[]            ; Reset to beginning (before first)
```

**Reverse Iteration**:
```lavishscript
if ${NameIter:Last(exists)}
{
    do
    {
        echo "Name (reverse): ${NameIter.Value}"
    }
    while ${NameIter:Previous(exists)}
}
```

**Modifying During Iteration** (dangerous):
```lavishscript
; CAUTION: Modifying index during iteration can cause issues
variable index:int Numbers
Numbers:Insert[1]
Numbers:Insert[2]
Numbers:Insert[3]

variable iterator Num
Numbers:GetIterator[Num]

if ${Num:First(exists)}
{
    do
    {
        ; Be careful modifying the index here
        ; Can cause iterator to skip items or crash
    }
    while ${Num:Next(exists)}
}
```

### point3f (3D Coordinate)

**What It Is**:
- 3D position in space
- X, Y, Z coordinates
- Used for spatial calculations

**Declaration**:
```lavishscript
variable point3f Position
variable point3f MyPosition
```

**Setting Values**:
```lavishscript
Position:Set[1000, 2000, 3000]

; From entity
MyPosition:Set[${MyShip.ToEntity.X}, ${MyShip.ToEntity.Y}, ${MyShip.ToEntity.Z}]
```

**Members**:
```lavishscript
${Position.X}               ; X coordinate
${Position.Y}               ; Y coordinate
${Position.Z}               ; Z coordinate
${Position.Distance[${OtherPoint}]}  ; Distance to another point
```

**Distance Calculation**:
```lavishscript
variable point3f PosA
variable point3f PosB

PosA:Set[0, 0, 0]
PosB:Set[100, 200, 300]

variable float Distance = ${PosA.Distance[${PosB}]}
echo "Distance: ${Distance}m"
```

**Common Use**:
```lavishscript
; Calculate distance between two entities without ISXEVE helper
function CalculateDistance(int64 entityA, int64 entityB)
{
    variable point3f PosA
    variable point3f PosB

    PosA:Set[${Entity[${entityA}].X}, ${Entity[${entityA}].Y}, ${Entity[${entityA}].Z}]
    PosB:Set[${Entity[${entityB}].X}, ${Entity[${entityB}].Y}, ${Entity[${entityB}].Z}]

    return ${PosA.Distance[${PosB}]}
}
```

### time (Timestamp)

**What It Is**:
- Unix timestamp (seconds since epoch)
- Used for timing and cooldowns

**Declaration**:
```lavishscript
variable time Now = ${Time.Timestamp}
variable time LastAction
```

**Members**:
```lavishscript
${Now}                  ; Timestamp (int64)
${Now.Date}             ; Date string
${Now.Time12}           ; 12-hour time
${Now.Time24}           ; 24-hour time
${Now.Year}             ; Year
${Now.Month}            ; Month
${Now.Day}              ; Day
```

**Cooldown Pattern**:
```lavishscript
variable time LastAction = ${Time.Timestamp}

function CanDoAction()
{
    variable time Now = ${Time.Timestamp}
    variable int64 TimeSince = ${Math.Calc[${Now} - ${LastAction}]}

    ; 5 second cooldown
    if ${TimeSince} >= 5
    {
        return TRUE
    }

    return FALSE
}

function DoAction()
{
    if !${CanDoAction}
    {
        echo "Action on cooldown"
        return
    }

    ; Do action
    echo "Action executed"

    ; Update timestamp
    LastAction:Set[${Time.Timestamp}]
}
```

---

## Type Conversion and Coercion

### Implicit Conversion

**Auto-Conversion Cases**:
```lavishscript
; Int to String (common)
variable int Number = 42
variable string Str = "${Number}"  ; "42"

; String to Int (if valid)
variable string NumStr = "123"
variable int Num = ${NumStr}       ; 123

; Bool to Int
variable bool Flag = TRUE
variable int Value = ${Flag}       ; 1 (TRUE=1, FALSE=0)
```

### Explicit Conversion

**String Methods**:
```lavishscript
variable string Str = "42"
variable int I = ${Str.Int}         ; 42
variable float F = ${Str.Float}     ; 42.0

variable string FloatStr = "3.14"
variable float F2 = ${FloatStr.Float}  ; 3.14
```

**Float Methods**:
```lavishscript
variable float F = 3.7
variable int I = ${F.Int}           ; 3 (truncate)
variable int Rounded = ${F.Round}   ; 4 (round)
```

**Manual ToString**:
```lavishscript
; Wrap in ${}
variable int Number = 42
variable string Str = "${Number}"

; Or explicitly
Str:Set["${Number}"]
```

---

## Variable Lifetime

### Local Variables

**Created**: When execution reaches declaration
**Destroyed**: When function/block exits

```lavishscript
function TestLifetime()
{
    echo "Before variable created"

    variable int LocalVar = 10

    echo "Variable created: ${LocalVar}"

    ; Variable destroyed when function returns
}
; LocalVar no longer exists here
```

### Global Variables

**Created**: When execution reaches declaration
**Destroyed**: When script ends

```lavishscript
variable(global) int GlobalVar = 100

function main()
{
    echo "${GlobalVar}"  ; 100

    GlobalVar:Set[200]

    ; GlobalVar persists until script ends
}
```

### GlobalKeep Variables

**Created**: First time script runs
**Destroyed**: Never (persists across script reloads)

```lavishscript
variable(globalkeep) int PersistentCounter = 0
PersistentCounter:Inc

; Even if script ends and restarts, counter retains value
```

---

## Memory and Performance

### Memory Usage

**Primitive Types** (approximate):
- bool: 1 byte
- int: 4 bytes
- int64: 8 bytes
- float: 4 bytes
- string: Variable (minimum ~20 bytes overhead + content)

**Complex Types**:
- index: ~100+ bytes overhead + contents
- iterator: ~20 bytes
- point3f: 12 bytes (3 floats)

### Performance Considerations

**Avoid Excessive Global Variables**:
```lavishscript
; BAD - hundreds of globals
variable(global) int GlobalVar1
variable(global) int GlobalVar2
; ... x 1000

; BETTER - use data structure
variable(global) index:int AllValues
```

**String Concatenation**:
```lavishscript
; SLOW - creates new string each time
variable string Result = ""
variable int i = 0
while ${i} < 1000
{
    Result:Concat["Item${i}"]  ; Very slow for large loops
    i:Inc
}

; BETTER - build list, join once
variable index:string Items
variable int i = 0
while ${i} < 1000
{
    Items:Insert["Item${i}"]
    i:Inc
}
```

---

## Best Practices

### Naming Conventions

**Readable Names**:
```lavishscript
; GOOD
variable int TargetCount
variable string MasterSession
variable bool IsReady

; BAD
variable int tc
variable string ms
variable bool r
```

**Scope Indication** (optional pattern):
```lavishscript
; Prefix globals with g_
variable(global) int g_TargetCount

; Prefix locals with no prefix
variable int targetCount

; Constants in UPPER_CASE
variable(readonly) int MAX_TARGETS = 10
```

### Initialization

**Always Initialize Important Variables**:
```lavishscript
; GOOD - explicit initialization
variable int Counter = 0
variable string State = "Idle"
variable bool IsRunning = FALSE

; RISKY - relies on defaults
variable int Counter
variable string State
```

### Type Choice

**Use Appropriate Types**:
```lavishscript
; Entity IDs - MUST use int64
variable int64 EntityID = ${Target.ID}  ; CORRECT

; Counts - int is fine
variable int NPCCount = ${NPCs.Used}

; Distances - float
variable float Distance = ${Entity.Distance}

; Flags - bool
variable bool IsLocked = ${Entity.IsLockedTarget}
```

### 🔴 CRITICAL: Variable Scope Bug in do-while Loops

**DISCOVERED: 2025-10-09**

**THE BUG**:
Variables declared INSIDE do-while loops become NULL/0 when accessed in if/elseif conditions!

**WRONG (will fail silently)**:
```lavishscript
do
{
    variable int groupID = ${ModIter.Value.ToItem.GroupID}  ; Gets value: 52
    variable string name = "${ModIter.Value.Name}"          ; Gets value: "Warp Disruptor II"

    if ${groupID} == 52  ; ← FAILS! groupID is now 0/NULL
    {
        echo "${name}"   ; ← FAILS! name is now ""
        ; Never executes even though groupID WAS 52!
    }
}
while ${ModIter:Next(exists)}
```

**CORRECT (always works)**:
```lavishscript
do
{
    ; Access properties DIRECTLY - don't store in variables
    if ${ModIter.Value.ToItem.GroupID} == 52  ; ← WORKS!
    {
        echo "${ModIter.Value.Name}"           ; ← WORKS!
        Collection:Insert[${ModIter.Value.ID}]
    }
}
while ${ModIter:Next(exists)}
```

**WHY THIS HAPPENS**:
- LavishScript has undocumented quirky scope rules
- Variables declared inside do-while loops lose their values when accessed in conditional branches
- This is NOT documented anywhere in official LavishScript documentation
- Discovered through debugging YAMFA module initialization (all modules showed as UNRECOGNIZED despite having correct GroupIDs)

**THE RULE**:
**NEVER declare variables inside do-while loops if you need to use them in if/elseif conditions.**
**ALWAYS access iterator/object properties directly instead.**

**Real-World Example (from YAMFA)**:
```lavishscript
; ❌ WRONG - All modules showed as UNRECOGNIZED
variable index:module AllModules
variable iterator ModIter
MyShip:GetModules[AllModules]
AllModules:GetIterator[ModIter]

if ${ModIter:First(exists)}
{
    do
    {
        variable int groupID = ${ModIter.Value.ToItem.GroupID}
        variable string name = "${ModIter.Value.Name}"

        if ${groupID} == 52        ; FAILS - groupID is 0
        {
            echo "Tackle module"   ; Never executes
        }
    }
    while ${ModIter:Next(exists)}
}

; ✅ CORRECT - All modules correctly categorized
if ${ModIter:First(exists)}
{
    do
    {
        ; Access directly
        if ${ModIter.Value.ToItem.GroupID} == 52
        {
            echo "${ModIter.Value.Name}"  ; Works perfectly
            ModuleList:Insert[${ModIter.Value.ID}]
        }
    }
    while ${ModIter:Next(exists)}
}
```

**Alternative (if you MUST use variables)**:
Declare variables OUTSIDE the loop:
```lavishscript
; Declare BEFORE loop
variable int groupID
variable string name

do
{
    ; Set values
    groupID:Set[${ModIter.Value.ToItem.GroupID}]
    name:Set["${ModIter.Value.Name}"]

    ; This MIGHT work (untested), but direct access is safer
    if ${groupID} == 52
    {
        echo "${name}"
    }
}
while ${ModIter:Next(exists)}
```

**RECOMMENDATION**: Always use direct property access in iterators. It's more reliable and avoids this bug entirely.

---

**END OF FILE**
**Next File**: 07_Control_Flow_and_Logic.md
