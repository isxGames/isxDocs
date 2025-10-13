# Control Flow and Logic
## Complete Guide to Branching and Looping

**Purpose**: Exhaustive coverage of control flow structures in LavishScript
**Audience**: AI mastering decision-making and loops
**Prerequisites**: Files 04-06

---

## Table of Contents

1. [Conditional Statements](#conditional-statements)
2. [Logical Operators](#logical-operators)
3. [Comparison Operators](#comparison-operators)
4. [While Loops](#while-loops)
5. [Do-While Loops](#do-while-loops)
6. [For Loops (Limited)](#for-loops-limited)
7. [Loop Control](#loop-control)
8. [Switch Alternatives](#switch-alternatives)
9. [Complex Conditions](#complex-conditions)
10. [Decision Trees](#decision-trees)

---

## Conditional Statements

### Basic if Statement

**Syntax**:
```lavishscript
if <condition>
{
    ; Code executes if condition TRUE
}
```

**Examples**:
```lavishscript
; Simple boolean check
if ${IsReady}
{
    echo "Ready"
}

; Comparison
if ${Counter} > 10
{
    echo "Counter exceeded 10"
}

; Existence check
if ${Entity[${TargetID}](exists)}
{
    echo "Entity exists"
}

; Method return value
if ${MyShip.Module[1].IsActive}
{
    echo "Module is active"
}
```

### if-else Statement

**Syntax**:
```lavishscript
if <condition>
{
    ; Code if TRUE
}
else
{
    ; Code if FALSE
}
```

**Examples**:
```lavishscript
if ${MyShip.ShieldPct} > 50
{
    echo "Shields good"
}
else
{
    echo "Shields low - retreat!"
}

; Nested in function (modern inventory API)
function CheckCargo()
{
    ; Check if inventory window available
    if ${EVEWindow[Inventory](exists)}
    {
        if ${EVEWindow[Inventory].Child[ShipCargo].FreeSpace} > 100
        {
            return TRUE  ; Space available
        }
        else
        {
            return FALSE  ; Cargo full
        }
    }

    return FALSE  ; Inventory not accessible
}
```

### if-elseif-else Chain

**Syntax**:
```lavishscript
if <condition1>
{
    ; Code if condition1 TRUE
}
elseif <condition2>
{
    ; Code if condition2 TRUE (and condition1 FALSE)
}
elseif <condition3>
{
    ; Code if condition3 TRUE (and above FALSE)
}
else
{
    ; Code if all conditions FALSE
}
```

**Extensive Example**:
```lavishscript
function DetermineAction()
{
    ; Priority-based decision making
    if ${MyShip.ShieldPct} < 25
    {
        echo "CRITICAL SHIELDS - WARP OUT"
        call WarpToSafe
    }
    elseif ${MyShip.CapacitorPct} < 20
    {
        echo "CRITICAL CAP - EMERGENCY PROTOCOL"
        call EmergencyCap
    }
    elseif ${UnderAttack} && ${AttackerCount} > 3
    {
        echo "OUTNUMBERED - RETREAT"
        call WarpToSafe
    }
    elseif ${NPCCount} > 0
    {
        echo "NPCS PRESENT - ENGAGE"
        call EngageCombat
    }
    elseif ${WreckCount} > 0
    {
        echo "WRECKS PRESENT - LOOT"
        call LootWrecks
    }
    else
    {
        echo "IDLE - LOOKING FOR WORK"
        call FindNewAnomaly
    }
}
```

**Priority Matters**:
```lavishscript
; First matching condition executes
if ${HP} < 50
{
    echo "HP below 50"
}
elseif ${HP} < 75
{
    echo "HP below 75"  ; Never executes if HP < 50
}

; This means if HP = 30:
; - First condition matches (30 < 50)
; - Executes first block
; - Skips second condition check
```

### Nested if Statements

**Multiple Levels**:
```lavishscript
if ${Me.InSpace}
{
    echo "In space"

    if ${Me.InFleet}
    {
        echo "In fleet"

        if ${IsMaster}
        {
            echo "Master in fleet in space"
            call MasterLogic
        }
        else
        {
            echo "Slave in fleet in space"
            call SlaveLogic
        }
    }
    else
    {
        echo "Not in fleet"
        call SoloLogic
    }
}
else
{
    echo "In station"
    call StationLogic
}
```

**Flattening Nested Conditions** (better):
```lavishscript
; Instead of deep nesting:
if ${Me.InSpace}
{
    if ${HasTarget}
    {
        if ${Target.Distance} < 10000
        {
            call Shoot
        }
    }
}

; Use early returns to flatten:
function ProcessCombat()
{
    if !${Me.InSpace}
    {
        return  ; Not in space, exit
    }

    if !${HasTarget}
    {
        return  ; No target, exit
    }

    if ${Target.Distance} >= 10000
    {
        return  ; Too far, exit
    }

    ; All conditions met
    call Shoot
}
```

---

## Logical Operators

### AND Operator (&&)

**Syntax**: `condition1 && condition2`
**Evaluation**: Both must be TRUE

```lavishscript
; Both conditions must be TRUE
if ${InSpace} && ${HasTarget}
{
    echo "In space AND have target"
}

; Multiple conditions
if ${Shield} > 50 && ${Cap} > 30 && ${IsLocked}
{
    echo "All conditions met"
}

; With parentheses
if (${A} > 5 && ${B} > 10) && ${C} < 20
{
    echo "Complex AND logic"
}
```

**Short-Circuit Evaluation**:
```lavishscript
; If first condition FALSE, second not evaluated
if ${Entity[${ID}](exists)} && ${Entity[${ID}].Distance} < 10000
{
    ; Safe - existence checked first
    ; If entity doesn't exist, second condition NOT evaluated
    ; Avoids crash
}
```

**Common Patterns**:
```lavishscript
; Existence + property check
if ${Target(exists)} && ${Target.IsLockedTarget}
{
    call ShootTarget
}

; Range checks
if ${Distance} > 1000 && ${Distance} < 50000
{
    echo "In optimal range"
}

; State + availability
if ${State.Equal["Combat"]} && ${MyShip.Module[1].IsOnline}
{
    Module[1]:Activate[]
}
```

### OR Operator (||)

**Syntax**: `condition1 || condition2`
**Evaluation**: At least one must be TRUE

```lavishscript
; Either condition TRUE
if ${Shield} < 25 || ${Armor} < 25
{
    echo "Low HP - retreat!"
}

; Multiple conditions
if ${IsIdle} || ${IsReturning} || ${IsDocked}
{
    echo "Not in combat"
}

; With parentheses
if (${A} > 100 || ${B} > 100) || ${C} > 100
{
    echo "At least one exceeded 100"
}
```

**Short-Circuit Evaluation**:
```lavishscript
; If first TRUE, second not evaluated
if ${IsMaster} || ${SomeExpensiveCheck}
{
    ; If IsMaster TRUE, SomeExpensiveCheck NOT called
}
```

**Common Patterns**:
```lavishscript
; Flee conditions (any trigger flee)
if ${Shield} < 20 || ${Players} > 0 || ${IsScrammed}
{
    call FleeToSafe
}

; State machine (multiple valid states)
if ${State.Equal["Idle"]} || ${State.Equal["Waiting"]} || ${State.Equal["Docked"]}
{
    call CheckForWork
}

; Type checking (enemy types)
if ${IsNPC} || ${IsHostilePlayer}
{
    call AddToTargetQueue
}
```

### NOT Operator (!)

**Syntax**: `!condition`
**Evaluation**: Inverts boolean value

```lavishscript
; Invert boolean
if !${IsReady}
{
    echo "Not ready"
}

; Invert comparison
if !${Shield} > 50  ; Same as ${Shield} <= 50
{
    echo "Shield not greater than 50"
}

; Invert method return
if !${Entity[${ID}](exists)}
{
    echo "Entity doesn't exist"
}

; Double negative (NOT recommended)
if !!${SomeBool}  ; Just use ${SomeBool}
{
    echo "Unnecessarily complex"
}
```

**Common Patterns**:
```lavishscript
; Existence check (common!)
if !${Entity[${ID}](exists)}
{
    echo "Entity despawned"
    return
}

; Inverting flags
if !${ScriptRunning}
{
    echo "Script stopped"
    break
}

; State checks
if !${State.Equal["Combat"]}
{
    echo "Not in combat"
}
```

### Combined Operators

**AND + OR**:
```lavishscript
; Grouping with parentheses
if (${A} > 5 && ${B} > 10) || (${C} > 100)
{
    ; (A>5 AND B>10) OR (C>100)
}

; Multiple groups
if (${Shield} < 25 || ${Armor} < 25) && ${SafeSpotExists}
{
    ; (Low HP) AND (Safe spot available)
    call WarpToSafe
}
```

**Complex Combat Logic**:
```lavishscript
if (${Target(exists)} && ${Target.Distance} < ${WeaponRange}) || ${OverkillOK}
{
    ; Shoot if:
    ; - Target exists AND in range
    ; OR
    ; - Overkill allowed (fire anyway)

    call ActivateWeapons
}
```

**Defensive Checks**:
```lavishscript
; Check all safety conditions
if ${Me.InSpace} && !${IsDocked} && ${ISXEVE.IsReady} && ${MyShip(exists)}
{
    ; Safe to run bot logic
    call ProcessBot
}
else
{
    echo "Waiting for safe conditions"
}
```

---

## Comparison Operators

### Numeric Comparisons

**Operators**:
- `>` - Greater than
- `<` - Less than
- `>=` - Greater or equal
- `<=` - Less or equal
- `==` - Equal
- `!=` - Not equal

**Examples**:
```lavishscript
variable int A = 10
variable int B = 5

if ${A} > ${B}       ; TRUE (10 > 5)
{
    echo "A greater than B"
}

if ${A} < ${B}       ; FALSE
{
    echo "A less than B"
}

if ${A} >= 10        ; TRUE (10 >= 10)
{
    echo "A at least 10"
}

if ${A} <= 5         ; FALSE (10 <= 5)
{
    echo "A at most 5"
}

if ${A} == 10        ; TRUE
{
    echo "A equals 10"
}

if ${A} != ${B}      ; TRUE (10 != 5)
{
    echo "A not equal to B"
}
```

**Float Comparisons**:
```lavishscript
variable float Distance = ${Entity.Distance}

if ${Distance} < 10000.0
{
    echo "Within 10km"
}

if ${Distance} >= 50000.0
{
    echo "50km or further"
}

; Beware float precision issues
variable float F1 = 0.1
variable float F2 = 0.2
variable float Sum = ${Math.Calc[${F1} + ${F2}]}

if ${Sum} == 0.3  ; May be FALSE due to float precision!
{
    echo "Exact match"
}

; Better: Range check
if ${Sum} > 0.29 && ${Sum} < 0.31
{
    echo "Approximately 0.3"
}
```

### String Comparisons

**Equal Method**:
```lavishscript
variable string A = "Hello"
variable string B = "Hello"
variable string C = "hello"

if ${A.Equal["${B}"]}    ; TRUE
{
    echo "Strings equal"
}

if ${A.Equal["${C}"]}    ; FALSE (case-sensitive)
{
    echo "Strings equal"
}
```

**Case-Insensitive**:
```lavishscript
; Compare uppercase versions
if ${A.Upper.Equal["${C.Upper}"]}  ; TRUE
{
    echo "Strings equal (case-insensitive)"
}
```

**Contains / Find**:
```lavishscript
variable string Name = "Caldari Navy Hookbill"

if ${Name.Find["Navy"]}  ; Returns position (> 0) if found
{
    echo "This is a Navy ship"
}

if ${Name.Find["Guristas"]} == 0  ; Not found
{
    echo "Not a Guristas ship"
}
```

**Common Pattern - State Checking**:
```lavishscript
if ${CurrentState.Equal["Combat"]}
{
    call ProcessCombat
}
elseif ${CurrentState.Equal["Mining"]}
{
    call ProcessMining
}
```

### Boolean Comparisons

**Direct Use** (preferred):
```lavishscript
variable bool IsReady = TRUE

if ${IsReady}  ; Preferred
{
    echo "Ready"
}

if ${IsReady} == TRUE  ; Redundant but valid
{
    echo "Ready"
}

if !${IsReady}  ; Preferred for negative
{
    echo "Not ready"
}

if ${IsReady} == FALSE  ; Redundant
{
    echo "Not ready"
}
```

**Method Returns**:
```lavishscript
; Method returns bool
if ${Entity[${ID}](exists)}
{
    ; Exists
}

; Object member is bool
if ${MyShip.Module[1].IsActive}
{
    ; Module is active
}
```

---

## While Loops

### Basic While Loop

**Syntax**:
```lavishscript
while <condition>
{
    ; Loop body
    wait <delay>  ; CRITICAL - prevents CPU maxing
}
```

**Simple Counter Loop**:
```lavishscript
variable int Counter = 0

while ${Counter} < 10
{
    echo "Counter: ${Counter}"
    Counter:Inc
    wait 5
}
```

**Condition-Based Loop**:
```lavishscript
; Wait until in range
while ${Entity[${TargetID}].Distance} > 2500
{
    ; Keep approaching
    wait 10

    ; Safety: Check still exists
    if !${Entity[${TargetID}](exists)}
    {
        echo "Target despawned"
        break
    }
}
```

### Infinite Loop (Main Loop Pattern)

**Script Main Loop**:
```lavishscript
variable bool Running = TRUE

while ${Running}
{
    ; Main logic
    call ProcessBot

    ; CRITICAL: Must have wait
    wait 20  ; 2 seconds
}
```

**With Exit Condition**:
```lavishscript
while TRUE
{
    ; Check exit condition
    if ${ShouldStop}
    {
        break  ; Exit loop
    }

    call DoWork
    wait 10
}
```

### Wait-Until Pattern

**Wait for Condition**:
```lavishscript
function WaitUntilLocked(int64 targetID, int timeout)
{
    variable int counter = 0

    while !${Entity[${targetID}].IsLockedTarget} && ${counter} < ${timeout}
    {
        wait 10
        counter:Inc

        ; Check entity still exists
        if !${Entity[${targetID}](exists)}
        {
            echo "Target despawned while waiting"
            return FALSE
        }
    }

    ; Check if successful or timeout
    if ${Entity[${targetID}].IsLockedTarget}
    {
        return TRUE
    }
    else
    {
        echo "Lock timed out"
        return FALSE
    }
}
```

**Generic Wait Function**:
```lavishscript
function WaitForCondition(int timeoutSeconds)
{
    variable int timeout = ${Math.Calc[${timeoutSeconds} * 10]}
    variable int counter = 0

    while <your_condition_here> && ${counter} < ${timeout}
    {
        wait 10
        counter:Inc
    }

    if ${counter} >= ${timeout}
    {
        echo "Timeout waiting for condition"
        return FALSE
    }

    return TRUE
}
```

### Complex While Conditions

**Multiple Conditions**:
```lavishscript
; AND conditions
while ${Running} && ${InSpace} && ${TargetExists}
{
    call ProcessCombat
    wait 10
}

; OR conditions
while ${State.Equal["Combat"]} || ${State.Equal["Looting"]}
{
    call ProcessActiveState
    wait 10
}

; Complex mixed
while (${Shield} > 25 && ${Cap} > 20) && !${PlayersPresent}
{
    call ContinueCombat
    wait 10
}
```

---

## Do-While Loops

### Basic Do-While

**Syntax**:
```lavishscript
do
{
    ; Loop body (executes at least once)
}
while <condition>
```

**Guaranteed Execution**:
```lavishscript
variable int Counter = 100

; This executes once even though condition FALSE
do
{
    echo "Counter: ${Counter}"  ; Prints once
    Counter:Inc
}
while ${Counter} < 10  ; FALSE (100 < 10), so exits
```

### Iterator Pattern (Most Common Use)

**Standard Iterator Pattern**:
```lavishscript
variable index:entity NPCs
EVE:QueryEntities[NPCs, "IsNPC"]

variable iterator NPC
NPCs:GetIterator[NPC]

; Do-while is perfect for iterators
if ${NPC:First(exists)}
{
    do
    {
        echo "NPC: ${NPC.Value.Name}"

        ; Process NPC
        call ProcessNPC ${NPC.Value.ID}

    }
    while ${NPC:Next(exists)}
}
```

**Why Do-While for Iterators**:
```lavishscript
; First(exists) moves to first element AND returns TRUE if exists
; So we know element exists when entering loop
; Then Next(exists) checks for next element

; This pattern guarantees we process each element exactly once
```

**Alternative While Pattern** (less common):
```lavishscript
; Can also use while, but more awkward
variable iterator NPC
NPCs:GetIterator[NPC]

NPC:First(exists)

while ${NPC.Value(exists)}
{
    echo "NPC: ${NPC.Value.Name}"

    if !${NPC:Next(exists)}
    {
        break
    }
}
```

### Menu-Driven Loop

**Retry Until Success**:
```lavishscript
variable bool Success = FALSE

do
{
    ; Attempt action
    Success:Set[${AttemptDock}]

    if !${Success}
    {
        echo "Dock failed, retrying in 5 seconds"
        wait 50
    }
}
while !${Success}

echo "Docked successfully"
```

---

## For Loops (Limited)

### LavishScript Has NO True For Loop

**No Built-In For Syntax**:
```lavishscript
; This DOESN'T WORK in LavishScript:
; for (int i = 0; i < 10; i++)  ; NOT VALID!
; {
;     echo "i = ${i}"
; }
```

### While Loop Replacement

**Counter Loop with While**:
```lavishscript
; Equivalent to: for (i = 0; i < 10; i++)
variable int i = 0

while ${i} < 10
{
    echo "i = ${i}"
    i:Inc
    wait 5
}
```

**Counting Down**:
```lavishscript
; Equivalent to: for (i = 10; i > 0; i--)
variable int i = 10

while ${i} > 0
{
    echo "i = ${i}"
    i:Dec
    wait 5
}
```

**Step Increment**:
```lavishscript
; Equivalent to: for (i = 0; i < 100; i += 5)
variable int i = 0

while ${i} < 100
{
    echo "i = ${i}"
    i:Inc[5]  ; Increment by 5
    wait 5
}
```

---

## Loop Control

### break Statement

**Exit Loop Early**:
```lavishscript
while TRUE
{
    call DoWork

    ; Exit condition
    if ${ShouldStop}
    {
        echo "Breaking out of loop"
        break  ; Immediately exits while loop
    }

    wait 10
}
echo "Loop exited"
```

**Search and Exit**:
```lavishscript
; Find first NPC and exit
variable index:entity NPCs
EVE:QueryEntities[NPCs, "IsNPC"]

variable iterator NPC
NPCs:GetIterator[NPC]

variable int64 FoundID = 0

if ${NPC:First(exists)}
{
    do
    {
        if ${NPC.Value.Name.Find["Battleship"]}
        {
            echo "Found battleship: ${NPC.Value.Name}"
            FoundID:Set[${NPC.Value.ID}]
            break  ; Stop searching, found target
        }
    }
    while ${NPC:Next(exists)}
}
```

**Nested Loop Break**:
```lavishscript
; break only exits innermost loop
variable int i = 0

while ${i} < 10
{
    variable int j = 0

    while ${j} < 10
    {
        if ${j} == 5
        {
            break  ; Exits inner loop only
        }
        j:Inc
    }

    i:Inc
}
```

### continue Statement

**Skip to Next Iteration**:
```lavishscript
variable int i = 0

while ${i} < 10
{
    i:Inc

    ; Skip even numbers
    if ${Math.Calc[${i} % 2]} == 0
    {
        continue  ; Skip rest of loop, go to next iteration
    }

    echo "Odd number: ${i}"
    wait 5
}
; Output: 1, 3, 5, 7, 9
```

**Filtering Iterator**:
```lavishscript
variable iterator NPC
NPCs:GetIterator[NPC]

if ${NPC:First(exists)}
{
    do
    {
        ; Skip non-existent entities
        if !${NPC.Value(exists)}
        {
            echo "Entity despawned, skipping"
            continue
        }

        ; Skip locked targets
        if ${NPC.Value.IsLockedTarget}
        {
            continue
        }

        ; Process unlocked NPCs
        echo "Processing: ${NPC.Value.Name}"
        call ProcessNPC ${NPC.Value.ID}

    }
    while ${NPC:Next(exists)}
}
```

### return Statement (Function Exit)

**Early Return**:
```lavishscript
function CheckTarget(int64 targetID)
{
    ; Guard clauses with early return
    if !${Entity[${targetID}](exists)}
    {
        echo "Target doesn't exist"
        return FALSE  ; Exit function early
    }

    if ${Entity[${targetID}].IsMoribund}
    {
        echo "Target is dead"
        return FALSE
    }

    if ${Entity[${targetID}].Distance} > 100000
    {
        echo "Target too far"
        return FALSE
    }

    ; All checks passed
    return TRUE
}
```

---

## Switch Alternatives

### No Native Switch

**LavishScript Doesn't Have Switch**:
```lavishscript
; This DOESN'T WORK:
; switch (${State})  ; NOT VALID!
; {
;     case "Idle":
;         ...
;     case "Combat":
;         ...
; }
```

### if-elseif Chain (Replacement)

**State Machine Pattern**:
```lavishscript
function ProcessState()
{
    if ${State.Equal["Idle"]}
    {
        call ProcessIdle
    }
    elseif ${State.Equal["Combat"]}
    {
        call ProcessCombat
    }
    elseif ${State.Equal["Mining"]}
    {
        call ProcessMining
    }
    elseif ${State.Equal["Returning"]}
    {
        call ProcessReturning
    }
    elseif ${State.Equal["Docked"]}
    {
        call ProcessDocked
    }
    else
    {
        echo "ERROR: Unknown state ${State}"
        State:Set["Idle"]  ; Reset to safe state
    }
}
```

### Function Table Pattern (Advanced)

**Using Function Calls**:
```lavishscript
; Define handler functions
function HandleIdle()
{
    echo "Handling Idle"
}

function HandleCombat()
{
    echo "Handling Combat"
}

; Dispatch based on state
function DispatchState()
{
    if ${State.Equal["Idle"]}
    {
        call HandleIdle
    }
    elseif ${State.Equal["Combat"]}
    {
        call HandleCombat
    }
    ; ... etc
}
```

---

## Complex Conditions

### Readability with Parentheses

**Explicit Grouping**:
```lavishscript
; Without parentheses (hard to read)
if ${A} > 5 && ${B} > 10 || ${C} < 20 && ${D} == 30
{
    ; What's the precedence?
}

; With parentheses (clear intent)
if ((${A} > 5 && ${B} > 10) || (${C} < 20 && ${D} == 30))
{
    ; Clear: (A AND B) OR (C AND D)
}
```

### De Morgan's Laws

**NOT Distribution**:
```lavishscript
; NOT (A AND B) = (NOT A) OR (NOT B)
if !(${A} && ${B})
{
    ; Same as:
    ; if !${A} || !${B}
}

; NOT (A OR B) = (NOT A) AND (NOT B)
if !(${A} || ${B})
{
    ; Same as:
    ; if !${A} && !${B}
}
```

**Practical Example**:
```lavishscript
; Check if NOT (in space AND have target)
if !(${Me.InSpace} && ${HasTarget})
{
    echo "Either not in space OR no target"
}

; Equivalent to:
if !${Me.InSpace} || !${HasTarget}
{
    echo "Either not in space OR no target"
}
```

---

## Decision Trees

### Combat Priority Example

**Tiered Decision Making**:
```lavishscript
function DetermineCombatAction()
{
    ; TIER 1: Critical survival checks
    if ${MyShip.ShieldPct} < 15
    {
        echo "CRITICAL SHIELDS - EMERGENCY WARP"
        call WarpToSafe
        return
    }

    if ${MyShip.CapacitorPct} < 10
    {
        echo "CRITICAL CAP - SHUT DOWN MODULES"
        call EmergencyCapSave
        return
    }

    ; TIER 2: Tactical threats
    if ${PlayerCount} > 0 && !${PlayerIsFriendly}
    {
        echo "HOSTILE PLAYER - FLEE"
        call WarpToSafe
        return
    }

    if ${NPCCount} > 5 && ${MyShip.ShieldPct} < 50
    {
        echo "TOO MANY NPCS - RETREAT"
        call WarpToSafe
        return
    }

    ; TIER 3: Normal combat operations
    if ${NPCCount} > 0
    {
        echo "ENGAGING NPCS"
        call EngageNPCs
        return
    }

    ; TIER 4: Cleanup
    if ${WreckCount} > 0
    {
        echo "LOOTING WRECKS"
        call LootWrecks
        return
    }

    ; TIER 5: Idle
    echo "IDLE - LOOKING FOR WORK"
    call FindNewAnomaly
}
```

---

**END OF FILE**
**Next File**: 08_Functions_Atoms_and_Code_Organization.md
