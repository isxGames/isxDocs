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

Every ISXEQ2 datatype exposes two kinds of access: **members** (read values with `.`) and **methods** (perform actions with `:`). For the underlying LavishScript concepts and how to define your own members/methods on custom `objectdef` types, see [01_LavishScript_Fundamentals.md - Members](01_LavishScript_Fundamentals.md#members) and [01_LavishScript_Fundamentals.md - Methods](01_LavishScript_Fundamentals.md#methods).

This section focuses on how members and methods appear on ISXEQ2 datatypes.

### ISXEQ2 Member Access

```lavishscript
${Me.Name}           ; character member - returns "Mycharacter"
${Me.Level}          ; character member - returns 120
${Target.Distance}   ; actor member - returns 15.3
```

### ISXEQ2 Method Calls

```lavishscript
Me.Inventory[5]:Use              ; Use the item
Me.Ability["Holy Strike"]:Use    ; Cast the ability
Target:DoTarget                  ; Target the actor
LootWindow:LootAll               ; Loot everything
```

**Method with Parameters:**

```lavishscript
Me:Face[180]                     ; Face heading 180
Me:BankDeposit[c,1000]           ; Deposit 1000 copper
MerchantWindow.MerchantInventory[1]:Buy[5]  ; Buy 5 of item 1
```

**ISXEQ2 Syntax Summary:**

```lavishscript
${Object.Member}         ; Get a value from an ISXEQ2 object
Object:Method            ; Perform an action on an ISXEQ2 object
Object:Method[param]     ; Perform an action with a parameter
```

**Key Point:** Many ISXEQ2 methods require the object to exist - always pair method calls with `(exists)` checks (see next section).

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

For the fundamentals of declaring variables, variable types, scopes (script vs local vs global), and assignment, see [01_LavishScript_Fundamentals.md - Variables and Data Types](01_LavishScript_Fundamentals.md#variables-and-data-types) and [01_LavishScript_Fundamentals.md - Variable Scope](01_LavishScript_Fundamentals.md#best-practices-summary).

This section covers how variables are used with ISXEQ2 datatypes.

### Declaring ISXEQ2 Datatype Variables

ISXEQ2 datatypes can be stored in local or script-scoped variables just like primitive types:

```lavishscript
variable item MyWeapon
variable actor MyTarget
variable index:actor NearbyActors        ; Collection of actors
variable index:item Potions              ; Collection of items
variable iterator ResultIt               ; Iterator for collections
```

`index:actor`, `index:item`, `index:ability`, etc. are the collection types used with `EQ2:GetActors`, `Me:QueryInventory`, `Me:QueryAbilities`, and similar population methods (see [Query Syntax](#query-syntax) and [Collections and Iterators](#collections-and-iterators)).

### Common Pattern: Shorthand Variable

Assigning an ISXEQ2 object to a variable avoids repeating long member paths and, for async-loaded types, gives you a stable handle to call `Is*InfoAvailable` against:

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

**Key Point:** Script-scoped variables are useful for persistent state (settings, timers, flags) across an ISXEQ2 script's lifetime; local variables should be preferred for temporary calculations inside functions.

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
EQ2:QueryActors[NearbyActors, Distance < 50 && Level == 120]

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

Use `EQ2:GetActors` (not the deprecated `CustomActorArray`) to populate an `index:actor` collection with filtered actors, then walk it with an iterator:

```lavishscript
variable index:actor Actors
variable iterator ActorIt
EQ2:GetActors[Actors,Range,50,NPC]
Actors:GetIterator[ActorIt]
```

For the full API reference including all filter types (`Range`, `Type`, `Zone`, combined filters), complete examples (harvesting, nearest-enemy, group scan), and performance throttling guidance, see [15_Advanced_Scripting_Patterns.md - Modern EQ2:GetActors Usage](15_Advanced_Scripting_Patterns.md#modern-eq2getactors-usage).

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
atom OnChat(int ChatType, string Message, string Speaker, string Target, string SpeakerIsNPC, string ChannelName, int SpeakerID, int TargetID)
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

This guide focuses on ISXEQ2-specific concepts. For core LavishScript syntax - control flow (`if`/`while`/`for`), string manipulation, math (`${Math.Calc[...]}`), wait commands, function declaration/calling, and comparisons - see the dedicated LavishScript reference:

- [01_LavishScript_Fundamentals.md - Conditional Branching](01_LavishScript_Fundamentals.md#conditional-branching)
- [01_LavishScript_Fundamentals.md - Loops](01_LavishScript_Fundamentals.md#loops)
- [01_LavishScript_Fundamentals.md - Wait Commands](01_LavishScript_Fundamentals.md#wait-commands)
- [01_LavishScript_Fundamentals.md - Functions](01_LavishScript_Fundamentals.md#functions)
- [01_LavishScript_Fundamentals.md - Return Values](01_LavishScript_Fundamentals.md#return-values)

The remaining sections of this file apply those fundamentals to ISXEQ2-specific tasks.

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
