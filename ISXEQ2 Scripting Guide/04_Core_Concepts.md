# ISXEQ2 Core Concepts

Essential concepts every ISXEQ2 scripter needs to understand.

---

## Table of Contents

1. [Datatypes and Objects](#datatypes-and-objects)
2. [Top-Level Objects (TLOs)](#top-level-objects-tlos)
3. [Datatype Inheritance](#datatype-inheritance)
4. [Members and Methods](#members-and-methods)
5. [NULL Checks and Existence Validation](#null-checks-and-existence-validation)
6. [Variable Declarations and Scoping](#variable-declarations-and-scoping)
7. [Asynchronous Data Loading](#asynchronous-data-loading)
8. [Query Syntax](#query-syntax)
9. [Collections and Iterators](#collections-and-iterators)
10. [Events and Atoms](#events-and-atoms)
11. [LavishScript Basics](#lavishscript-basics)

---

## Datatypes and Objects

### What is a Datatype?

In ISXEQ2, every piece of game data has a specific **datatype** that defines what information it contains and what actions you can perform on it.

Think of a datatype as a blueprint or template:

```lavishscript
; ${Me} is a "character" datatype
echo ${Me.Name}          ; character has a Name member
echo ${Me.Level}         ; character has a Level member

; ${Target} is an "actor" datatype
echo ${Target.Name}      ; actor has a Name member
echo ${Target.Health}    ; actor has a Health member

; ${Me.Inventory[5]} is an "item" datatype
echo ${Me.Inventory[5].Name}      ; item has a Name member
echo ${Me.Inventory[5].Quantity}  ; item has a Quantity member
```

### Common Datatypes

| Datatype | Represents | Example |
|----------|------------|---------|
| **character** | Your character | `${Me}` |
| **actor** | Any NPC, PC, or object in the world | `${Target}`, `${Actor[Gnoll]}` |
| **item** | Inventory item | `${Me.Inventory[5]}` |
| **ability** | Spell or combat art | `${Me.Ability[1]}` |
| **effect** | Buff or debuff | `${Me.Effect[1]}` |
| **quest** | Quest information | `${QuestJournalWindow.ActiveQuest[1]}` |
| **zone** | Zone/area information | `${Zone}` |

**Key Point:** You can't mix datatypes. An "item" object won't have "actor" members.

---

## Top-Level Objects (TLOs)

**Top-Level Objects (TLOs)** are your entry points into the game's data. They're globally accessible and always available.

### Essential TLOs

```lavishscript
${Me}                    ; Your character (character datatype)
${Target}                ; Current target (actor datatype)
${Zone}                  ; Current zone (zone datatype)
${Actor[name/id]}        ; Specific actor (actor datatype)
${EQ2}                   ; Game utilities (eq2 datatype)
${ISXEQ2}                ; Extension info (isxeq2 datatype)
```

### Window TLOs

```lavishscript
${LootWindow}            ; Loot window
${RewardWindow}          ; Reward selection window
${MerchantWindow}        ; Merchant window
${QuestJournalWindow}    ; Quest journal
${ContainerWindow}       ; Container window
```

### Using TLOs

```lavishscript
; TLOs are accessed directly
echo ${Me.Name}

; Some TLOs take parameters
echo ${Actor[Gnoll].Name}              ; By name
echo ${Actor[ID,12345].Name}           ; By ID
echo ${EQ2UIPage[MapWindow].IsVisible} ; By window name
```

**Key Point:** TLOs don't need to be declared - they're always available.

---

## Datatype Inheritance

Many datatypes **inherit** from other datatypes, gaining all their members and methods.

### Inheritance Hierarchy

```
character inherits from actor
  ↓
character has all actor members PLUS its own character-specific members
```

**Example:**

```lavishscript
; "actor" datatype has these members:
${Target.Name}      ; actor member
${Target.Health}    ; actor member
${Target.Level}     ; actor member

; "character" inherits from "actor", so ${Me} has all actor members:
${Me.Name}          ; inherited from actor
${Me.Health}        ; inherited from actor
${Me.Level}         ; inherited from actor

; Plus character-specific members:
${Me.Platinum}      ; only character has this
${Me.Inventory[5]}  ; only character has this
${Me.Ability[1]}    ; only character has this
```

### Common Inheritance Patterns

| Datatype | Inherits From | Meaning |
|----------|---------------|---------|
| **character** | **actor** | Characters have all actor properties plus inventory, abilities, etc. |
| **groupmember** | **actor** | Group members have all actor properties plus group-specific data |
| **eq2clonewindow** | **eq2window** | Clone windows have all window properties plus clone-specific features |
| **eq2button** | **eq2widget** → **eq2baseobject** | Buttons have widget properties, which have base object properties |

**Key Point:** Understanding inheritance helps you know what members are available.

---

## Members and Methods

Datatypes have two types of access:

### Members (Properties)

Members are **read-only values** accessed with a dot:

```lavishscript
${Me.Name}           ; Returns "Mycharacter"
${Me.Level}          ; Returns 120
${Target.Distance}   ; Returns 15.3
```

### Methods (Actions)

Methods are **actions** you can perform, accessed with a colon:

```lavishscript
Me.Inventory[5]:Use              ; Use the item
Me.Ability["Holy Strike"]:Use    ; Cast the ability
Target:DoTarget                  ; Target the actor
LootWindow:LootAll              ; Loot everything
```

**Method with Parameters:**

```lavishscript
Me:Face[180]                     ; Face heading 180
Me:BankDeposit[1000]            ; Deposit 1000 copper
MerchantWindow.Item[1]:Buy[5]   ; Buy 5 of item 1
```

**Syntax Summary:**

```lavishscript
${Object.Member}         ; Get a value (read)
Object:Method            ; Perform an action (write)
Object:Method[param]     ; Perform an action with parameter
```

---

## NULL Checks and Existence Validation

**THE MOST IMPORTANT CONCEPT:** Always check if something exists before accessing it.

### The Problem

```lavishscript
; BAD - Will error if no target
echo ${Target.Name}

; Console output:
; Invalid type
```

### The Solution: (exists) Check

```lavishscript
; GOOD - Check first
if ${Target(exists)}
{
    echo ${Target.Name}
}
else
{
    echo "No target"
}
```

### Common NULL Check Patterns

```lavishscript
; Check target exists
if ${Target(exists)}
    echo ${Target.Name}

; Check inventory slot has item
if ${Me.Inventory[5](exists)}
    Me.Inventory[5]:Use

; Check ability exists
if ${Me.Ability["Holy Strike"](exists)}
    Me.Ability["Holy Strike"]:Use

; Check window is open
if ${LootWindow(exists)}
    LootWindow:LootAll

; Check collection has items
if ${MyItems.Used} > 0
    echo "Found ${MyItems.Used} items"
```

### Defensive Coding Pattern

```lavishscript
function SafeTargetInfo()
{
    ; Guard clause - exit early if no target
    if !${Target(exists)}
    {
        echo "No target selected"
        return
    }

    ; Now safe to access target
    echo "Target: ${Target.Name}"
    echo "Level: ${Target.Level}"
    echo "Distance: ${Target.Distance}"
}
```

**Key Point:** **ALWAYS** use `(exists)` checks. This prevents 90% of script errors.

---

## Variable Declarations and Scoping

### Variable Scopes

LavishScript has two variable scopes:

#### Script-Scoped (Global)

```lavishscript
declare MyVariable int script 100

; Available throughout entire script
; Persists across function calls
; Survives loops and code blocks
```

**Use for:** Settings, counters, flags that need to persist

#### Local-Scoped (Function)

```lavishscript
function MyFunction()
{
    declare MyVariable int local 100

    ; Only available in this function
    ; Destroyed when function ends
    ; Each call creates a new instance
}
```

**Use for:** Temporary calculations, loop counters, function-specific data

### Variable Types

```lavishscript
declare MyInt int local 100              ; Integer
declare MyFloat float local 3.14         ; Floating point
declare MyString string local "Hello"    ; String
declare MyBool bool local TRUE           ; Boolean
declare MyItem item local                ; Object (uninitialized)
```

### Variable Assignment

```lavishscript
; Using :Set method
MyVariable:Set[100]
MyString:Set["New Value"]

; Using = operator (integers only)
MyInt:Set[${Math.Calc[10+5]}]
```

### Common Pattern: Shorthand Variable

```lavishscript
; Instead of repeatedly typing long paths:
echo ${Me.Equipment[primary].ToItemInfo.DamageRating}
echo ${Me.Equipment[primary].ToItemInfo.Delay}

; Use a variable:
variable item MyWeapon
MyWeapon:Set[${Me.Equipment[primary]}]

echo ${MyWeapon.ToItemInfo.DamageRating}
echo ${MyWeapon.ToItemInfo.Delay}
```

---

## Asynchronous Data Loading

Some game data loads **asynchronously** (in the background) and may not be immediately available.

### The Problem

```lavishscript
; Immediately after accessing an item, detailed info might not be loaded
variable item MyItem
MyItem:Set[${Me.Inventory[5]}]

echo ${MyItem.ToItemInfo.Description}  ; Might be empty!
```

### The Solution: Check Availability

```lavishscript
variable item MyItem
MyItem:Set[${Me.Inventory[5]}]

; Check if detailed info is loaded
if !${MyItem.IsItemInfoAvailable}
{
    ; Wait up to 3 seconds for it to load
    variable int Timer = 0
    while !${MyItem.IsItemInfoAvailable} && ${Timer:Inc} < 1500
    {
        waitframe
    }
}

; Now safe to access
if ${MyItem.IsItemInfoAvailable}
    echo ${MyItem.ToItemInfo.Description}
```

### Data That Requires Async Loading

| Datatype | Check Member | Detailed Object |
|----------|--------------|-----------------|
| **item** | `IsItemInfoAvailable` | `ToItemInfo` |
| **ability** | `IsAbilityInfoAvailable` | `ToAbilityInfo` |
| **effect** | `IsEffectInfoAvailable` | `ToEffectInfo` |
| **actoreffect** | `IsEffectInfoAvailable` | `ToEffectInfo` |
| **recipe** | `IsRecipeInfoAvailable` | `ToRecipeInfo` |

### Standard Async Wait Pattern

```lavishscript
; Request the data
Me:RequestEffectsInfo

; Wait a moment for the server to respond
wait 5

; Then check availability before accessing
if ${Me.Effect[1].IsEffectInfoAvailable}
{
    echo ${Me.Effect[1].ToEffectInfo.Description}
}
```

**Key Point:** Always check `Is*InfoAvailable` before accessing `To*Info` members.

---

## Query Syntax

ISXEQ2 provides powerful query capabilities to filter and search collections.

### Query Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `==` | Equals | `Level == 120` |
| `!=` | Not equals | `Type != "PC"` |
| `>` | Greater than | `Distance > 50` |
| `<` | Less than | `Health < 30` |
| `>=` | Greater or equal | `Level >= 110` |
| `<=` | Less or equal | `Quantity <= 5` |
| `=-` | Contains (case-insensitive) | `Name =- "Potion"` |
| `!-` | Does not contain | `Name !- "Junk"` |
| `=~` | Regex match | `Name =~ "^Gold"` |
| `!~` | Regex not match | `Name !~ "Trash"` |
| `&&` | And | `Level == 120 && Type == "NPC"` |
| `||` | Or | `Type == "NPC" || Type == "NamedNPC"` |

### Query Examples

```lavishscript
; Find actors within 50 meters at level 120
variable index:actor NearbyActors
EQ2:GetActors[NearbyActors,"Distance < 50 && Level == 120"]

; Find inventory items containing "Potion"
variable index:item Potions
Me:QueryInventory[Potions,"Name =- Potion"]

; Find ready abilities
variable index:ability ReadyAbilities
Me:QueryAbilities[ReadyAbilities,"IsReady"]

; Find detrimental effects
variable index:effect Detriments
Me:QueryEffects[Detriments,"!IsBeneficial"]

; Complex query with multiple conditions
variable index:item Weapons
Me:QueryInventory[Weapons,"Type =- \"Weapon\" && ToItemInfo.Level >= 100"]
```

### Modern Actor Collection (EQ2:GetActors)

**IMPORTANT:** Use `EQ2:GetActors` instead of the deprecated `CustomActorArray` system.

```lavishscript
; Get actors using modern method (populates index directly)
variable index:actor Actors
variable iterator ActorIt

EQ2:GetActors[Actors,Range,50,NPC]

; Iterate through results
Actors:GetIterator[ActorIt]
if ${ActorIt:First(exists)}
{
    do
    {
        echo "${ActorIt.Value.Name} at ${ActorIt.Value.Distance}m"
    }
    while ${ActorIt:Next(exists)}
}

echo "Total actors found: ${Actors.Used}"
```

**Key Point:** `EQ2:GetActors` is the modern, recommended method. See [13_Advanced_Scripting_Patterns.md](13_Advanced_Scripting_Patterns.md#modern-eq2getactors-usage) for comprehensive usage.

---

## Collections and Iterators

When queries return multiple results, they populate a **collection**. You iterate through collections using an **iterator**.

### Standard Iterator Pattern

```lavishscript
; Step 1: Declare collection and iterator
variable index:item MyItems
variable iterator ItemIterator

; Step 2: Populate collection with query
Me:QueryInventory[MyItems,"Name =- Potion"]

; Step 3: Get iterator
MyItems:GetIterator[ItemIterator]

; Step 4: Loop through results
if ${ItemIterator:First(exists)}
{
    do
    {
        echo "Found: ${ItemIterator.Value.Name}"
        echo "Quantity: ${ItemIterator.Value.Quantity}"
    }
    while ${ItemIterator:Next(exists)}
}
```

### Collection Members

```lavishscript
echo ${MyItems.Used}       ; Number of items in collection
echo ${MyItems.Size}       ; Total capacity of collection

if ${MyItems.Used} == 0
    echo "No items found"
```

### Alternative: For Loop Pattern

```lavishscript
; Get quests into collection
variable index:quest Quests
QuestJournalWindow:GetActiveQuests[Quests]

; Loop by index
variable int i
for (i:Set[1]; ${i} <= ${Quests.Used}; i:Inc)
{
    echo "Quest ${i}: ${Quests.Get[${i}].Name}"
}
```

**Key Point:** The iterator pattern is the standard way to process query results.

---

## Events and Atoms

ISXEQ2 provides 48+ **events** that fire when specific game actions occur. You attach **atom** handlers to respond to these events.

### Event Registration

```lavishscript
; Attach your handler to an event
Event[EQ2_ActorSpawned]:AttachAtom[OnActorSpawned]
```

### Atom Handler

```lavishscript
; Define what happens when event fires
atom OnActorSpawned(string ID, string Name, string Level, string Type)
{
    echo "Actor spawned: ${Name} (${Type}, Level ${Level})"

    ; React to specific actors
    if ${Type.Equal["NamedNPC"]}
    {
        echo "NAMED SPAWN DETECTED!"
    }
}
```

### Complete Event Example

```lavishscript
function main()
{
    ; Wait for ISXEQ2
    while !${ISXEQ2.IsReady}
        wait 10

    ; Register events
    Event[EQ2_onIncomingChatText]:AttachAtom[OnChat]
    Event[EQ2_onLootWindowAppeared]:AttachAtom[OnLoot]

    echo "Event handlers active"

    ; Keep script running
    wait 99999999
}

; Chat event handler
atom OnChat(int ChatType, string Message, string Speaker, string Target, string SpeakerIsNPC, string ChannelName, int SpeakerID, int TargetID, string UnkString1)
{
    ; React to tells
    if ${ChatType} == 7  ; Tell
    {
        echo "Received tell from ${Speaker}: ${Message}"
    }
}

; Loot window event handler
atom OnLoot(uint WindowID)
{
    echo "Loot window appeared!"
    wait 5
    LootWindow:LootAll
    echo "Looted all items"
}
```

### Event Detachment

```lavishscript
; Stop listening to event
Event[EQ2_onIncomingChatText]:DetachAtom[OnChat]
```

### Common Events

| Event | When It Fires |
|-------|---------------|
| `EQ2_ActorSpawned` | Actor appears in world |
| `EQ2_onIncomingChatText` | Chat message received |
| `EQ2_onInventoryUpdate` | Inventory changes |
| `EQ2_onLootWindowAppeared` | Loot window opens |
| `EQ2_StartedZoning` | Zoning begins |
| `EQ2_FinishedZoning` | Zoning completes |
| `EQ2_onQuestOffered` | Quest offered |
| `EQ2_onLevelChange` | Character levels up |

**Key Point:** Event-driven scripts are reactive and efficient.

---

## LavishScript Basics

Quick reference for essential LavishScript syntax.

### Control Flow

```lavishscript
; If statement
if ${Condition}
{
    ; Code
}
elseif ${OtherCondition}
{
    ; Code
}
else
{
    ; Code
}

; While loop
while ${Condition}
{
    ; Code
}

; Do-While loop
do
{
    ; Code
}
while ${Condition}

; For loop
variable int i
for (i:Set[1]; ${i} <= 10; i:Inc)
{
    ; Code
}
```

### Comparisons

```lavishscript
; Numeric
${Value} > 10
${Value} < 100
${Value} == 50
${Value} != 25
${Value} >= 10
${Value} <= 100

; Boolean
${Boolean}           ; TRUE check
!${Boolean}          ; FALSE check (negation)

; String
${String.Equal["Text"]}         ; Exact match (case-sensitive)
${String.NotEqual["Text"]}      ; Not equal
${String.Find["Sub"]}           ; Contains substring
${String.Left[5].Equal["Start"]}  ; First 5 chars
```

### String Manipulation

```lavishscript
${String.Length}                ; Length
${String.Left[5]}              ; First 5 characters
${String.Right[3]}             ; Last 3 characters
${String.Mid[2,5]}             ; 5 chars starting at position 2
${String.Upper}                ; UPPERCASE
${String.Lower}                ; lowercase
${String.Find["sub"]}          ; Position of substring (or 0 if not found)
${String.Replace["old","new"]} ; Replace text
```

### Math

```lavishscript
${Math.Calc[10+5]}             ; 15
${Math.Calc[10-5]}             ; 5
${Math.Calc[10*5]}             ; 50
${Math.Calc[10/5]}             ; 2
${Math.Calc[(10+5)*2]}         ; 30
${Math.Rand[100]}              ; Random 0-99
${Math.Rand[100]:Inc}          ; Random 1-100
```

### Wait Commands

```lavishscript
wait 1000                      ; Wait 1 second (1000ms)
wait 50 ${Condition}           ; Wait up to 5 seconds for condition
waitframe                      ; Wait one frame (~16ms)
```

### Functions

```lavishscript
function MyFunction()
{
    echo "Function called"
}

function MyFunctionWithParams(string name, int level)
{
    echo "Name: ${name}, Level: ${level}"
}

function MyFunctionWithReturn() returns int
{
    return 42
}

; Calling functions
call MyFunction
call MyFunctionWithParams "Bob" 120
variable int result
result:Set[${MyFunctionWithReturn}]
```

---

## Putting It All Together

Here's an example that demonstrates multiple core concepts:

```lavishscript
;*****************************************************
; Core Concepts Demo Script
;*****************************************************

; Script-scoped variables
declare Running bool script TRUE
declare LastHealthCheck int script 0

function main()
{
    ; Wait for ISXEQ2 (async loading)
    while !${ISXEQ2.IsReady}
        wait 10

    ; Register event
    Event[EQ2_onInventoryUpdate]:AttachAtom[OnInventoryChanged]

    echo "Core Concepts Demo Started"

    ; Main loop
    while ${Running}
    {
        ; Health check every 2 seconds
        if ${Script.RunningTime} >= ${Math.Calc64[${LastHealthCheck}+2000]}
        {
            call CheckHealth
            LastHealthCheck:Set[${Script.RunningTime}]
        }

        wait 10
    }
}

function CheckHealth()
{
    ; NULL check (existence validation)
    if !${Me(exists)}
        return

    ; Local variable for calculation
    variable float HealthPercent
    HealthPercent:Set[${Math.Calc[${Me.CurrentHealth}*100/${Me.MaxHealth}]}]

    ; Conditional logic
    if ${HealthPercent} < 30
    {
        echo "WARNING: Low health (${HealthPercent.Int}%)"
        call UseHealthPotion
    }
}

function UseHealthPotion()
{
    ; Query inventory (collection)
    variable index:item Potions
    variable iterator PotionIt

    Me:QueryInventory[Potions,"Name =- \"Health Potion\" && Location == \"Inventory\""]
    Potions:GetIterator[PotionIt]

    ; Iterator pattern
    if ${PotionIt:First(exists)}
    {
        ; Async data check
        if !${PotionIt.Value.IsItemInfoAvailable}
            wait 10 ${PotionIt.Value.IsItemInfoAvailable}

        ; Method call
        PotionIt.Value:Use
        echo "Used ${PotionIt.Value.Name}"
    }
    else
    {
        echo "No health potions found"
    }
}

; Event atom handler
atom OnInventoryChanged()
{
    echo "Inventory was updated"
}
```

---

## Summary of Core Concepts

1. **Datatypes** - Every object has a specific type with defined members/methods
2. **TLOs** - Global entry points (${Me}, ${Target}, ${Zone}, etc.)
3. **Inheritance** - Some datatypes inherit members from parent types
4. **Members vs Methods** - `.Member` gets values, `:Method` performs actions
5. **NULL Checks** - Always use `(exists)` before accessing objects
6. **Scoping** - Script vs local variables have different lifetimes
7. **Async Loading** - Check `Is*InfoAvailable` before accessing detailed info
8. **Queries** - Powerful filtering with operators (==, !=, =-, etc.)
9. **Collections** - Results stored in collections, accessed via iterators
10. **Events** - React to game actions with atom handlers

---

## Next Steps

- **Practice:** Try the examples in this guide
- **Reference:** Use [01_API_Reference.md](01_API_Reference.md) to look up specific datatypes
- **Examples:** Study [05_Working_Examples.md](05_Working_Examples.md) for real-world code
- **Patterns:** Read [04_Patterns_And_Best_Practices.md](04_Patterns_And_Best_Practices.md) for best practices

---

*Part of ISXEQ2 Scripting Guide*
