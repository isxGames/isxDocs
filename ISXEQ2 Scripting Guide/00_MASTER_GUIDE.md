# ISXEQ2 Master Guide - Quick Reference

Complete quick reference for ISXEQ2 scripting. For detailed information, see the individual documentation files.

---

## Quick Links

- **[README.md](README.md)** - Start here for navigation
- **[02_Quick_Start_Guide.md](02_Quick_Start_Guide.md)** - Get started in 5-10 minutes
- **[04_Core_Concepts.md](04_Core_Concepts.md)** - Fundamental concepts
- **[05_Patterns_And_Best_Practices.md](05_Patterns_And_Best_Practices.md)** - Best practices
- **[06_Working_Examples.md](06_Working_Examples.md)** - Code examples
- **[15_Advanced_Scripting_Patterns.md](15_Advanced_Scripting_Patterns.md)** - Production-grade patterns

---

## Essential TLOs (Top-Level Objects)

| TLO | Type | Common Usage |
|-----|------|--------------|
| `${Me}` | char | `${Me.Name}`, `${Me.Level}`, `${Me.CurrentHealth}` |
| `${Target}` | actor | `${Target.Name}`, `${Target.Distance}`, `${Target.Health}` |
| `${Zone}` | zone | `${Zone.Name}`, `${Zone.ID}` |
| `${Actor[name]}` | actor | `${Actor[Gnoll].ID}`, `${Actor[ID,12345].Name}` |
| `${EQ2}` | eq2 | `${EQ2.Zoning}`, `${EQ2.ServerName}` |
| `${ISXEQ2}` | isxeq2 | `${ISXEQ2.IsReady}`, `${ISXEQ2.Version}` |

---

## Character (Me) - Most Used Members

### Stats
```lavishscript
${Me.Name}                  ; Character name
${Me.Level}                 ; Level
${Me.SubClass}              ; Class (e.g., "Guardian")
${Me.CurrentHealth}         ; Current HP
${Me.MaxHealth}             ; Maximum HP
${Me.CurrentPower}          ; Current power/mana
${Me.MaxPower}              ; Maximum power
${Me.Platinum}              ; Platinum coins
${Me.Gold}                  ; Gold coins
${Me.Silver}                ; Silver coins
${Me.Copper}                ; Copper coins
```

### State
```lavishscript
${Me.InCombat}              ; TRUE if in combat
${Me.CastingSpell}          ; TRUE if casting
${Me.IsMoving}              ; TRUE if moving
${Me.AutoAttackOn}          ; TRUE if auto-attack active
${Me.InZone}                ; TRUE if fully loaded in zone
${Me.Grouped}               ; TRUE if in group
${Me.IsGroupLeader}         ; TRUE if group leader
```

### Position
```lavishscript
${Me.X}                     ; X coordinate
${Me.Y}                     ; Y coordinate
${Me.Z}                     ; Z coordinate
${Me.Heading}               ; Heading (0-360)
${Me.Distance[x,y,z]}       ; Distance to coordinates
```

### Inventory
```lavishscript
${Me.Inventory[5]}                        ; Item in slot 5
${Me.Inventory[ExactName,"Item Name"]}   ; Item by exact name
${Me.Equipment[primary]}                  ; Primary weapon
${Me.Equipment[chest]}                    ; Chest armor
${Me.InventorySlotsFree}                  ; Free inventory slots
```

### Abilities
```lavishscript
${Me.Ability["Ability Name"]}             ; Ability by name
${Me.Ability[1]}                          ; Ability by index
${Me.NumAbilities}                        ; Total abilities
```

### Group
```lavishscript
${Me.Group[1]}              ; First group member
${Me.GroupCount}            ; Number in group
```

---

## Actor (Target, NPCs, PCs) - Most Used Members

### Identity
```lavishscript
${Actor.Name}               ; Name
${Actor.Level}              ; Level
${Actor.Type}               ; PC, NPC, NamedNPC, Pet, etc.
${Actor.Class}              ; Class name
${Actor.ID}                 ; Unique ID
```

### Stats
```lavishscript
${Actor.Health}             ; Health percentage
${Actor.Power}              ; Power percentage
${Actor.Distance}           ; Distance from Me
${Actor.Heading}            ; Heading
```

### State
```lavishscript
${Actor.IsDead}             ; TRUE if dead
${Actor.IsAggro}            ; TRUE if aggressive
${Actor.InCombat}           ; TRUE if in combat
${Actor.IsCamping}          ; TRUE if camping out
${Actor.IsMoving}           ; TRUE if moving
```

### Targeting
```lavishscript
${Actor.Target}             ; Actor's target (returns actor)
${Actor.Pet}                ; Actor's pet (returns actor)
```

---

## Item - Most Used Members

### Basic Info
```lavishscript
${Item.Name}                ; Item name
${Item.ID}                  ; Item ID
${Item.Quantity}            ; Stack quantity
${Item.Slot}                ; Inventory slot
```

### Item Info (Detailed)
```lavishscript
${Item.IsItemInfoAvailable}              ; Check if detailed info loaded
${Item.ToItemInfo.Description}           ; Description
${Item.ToItemInfo.Type}                  ; Item type
${Item.ToItemInfo.Level}                 ; Required level
${Item.ToItemInfo.DamageRating}          ; Weapon damage
```

### State
```lavishscript
${Item.IsReady}             ; Not on cooldown
${Item.TimeUntilReady}      ; Seconds until ready
${Item.IsContainer}         ; Is a container/bag
```

### Methods
```lavishscript
Item:Use                    ; Use the item
Item:Equip                  ; Equip the item
Item:Destroy                ; Destroy the item
```

---

## Ability - Most Used Members

### Basic Info
```lavishscript
${Ability.IsReady}                       ; Ready to cast
${Ability.TimeUntilReady}                ; Seconds until ready
${Ability.IsAbilityInfoAvailable}        ; Detailed info loaded
```

### Ability Info (Detailed)
```lavishscript
${Ability.ToAbilityInfo.Name}            ; Ability name
${Ability.ToAbilityInfo.PowerCost}       ; Power cost
${Ability.ToAbilityInfo.CastingTime}     ; Cast time (seconds)
${Ability.ToAbilityInfo.Range}           ; Maximum range
${Ability.ToAbilityInfo.RecastTime}      ; Recast time
```

### Methods
```lavishscript
Ability:Use                 ; Cast the ability
Ability:Examine             ; Examine the ability
```

---

## Common Commands

### Game Commands (via EQ2Execute)
```lavishscript
EQ2Execute /say Hello
EQ2Execute /target Gnoll
EQ2Execute /useability "Holy Strike"
EQ2Execute /auto_attack
EQ2Execute /stopfollow
EQ2Execute /camp
```

### ISXEQ2 Commands
```lavishscript
Target NPC                  ; Target nearest NPC
Target "gnoll scout"        ; Target by name
Face                        ; Face current target
Face 180                    ; Face heading 180
Where NPC                   ; List all NPCs
Radar                       ; Show all actors on radar
```

---

## Query Syntax

### Operators
```lavishscript
==              ; Equals
!=              ; Not equals
>               ; Greater than
<               ; Less than
>=              ; Greater or equal
<=              ; Less or equal
=-              ; Contains (case-insensitive)
!-              ; Does not contain
=~              ; Regex match
!~              ; Regex not match
&&              ; And
||              ; Or
```

### Examples
```lavishscript
; Find actors
EQ2:GetActors[Index,"Distance < 50 && Level == 120"]

; Find inventory items
Me:QueryInventory[Items,"Name =- \"Potion\" && Quantity > 1"]

; Find ready abilities
Me:QueryAbilities[Abilities,"IsReady && IsReady"]

; Find effects
Me:QueryEffects[Effects,"IsBeneficial && Duration > 60"]
```

---

## Essential Patterns

### NULL Check Pattern
```lavishscript
if ${Target(exists)}
{
    echo ${Target.Name}
}
```

### Async Data Loading Pattern
```lavishscript
if !${Item.IsItemInfoAvailable}
{
    wait 10 ${Item.IsItemInfoAvailable}
}

if ${Item.IsItemInfoAvailable}
{
    echo ${Item.ToItemInfo.Description}
}
```

### Collection Iterator Pattern
```lavishscript
variable index:item Items
variable iterator ItemIt

Me:QueryInventory[Items,"Name =- Potion"]
Items:GetIterator[ItemIt]

if ${ItemIt:First(exists)}
{
    do
    {
        echo ${ItemIt.Value.Name}
    }
    while ${ItemIt:Next(exists)}
}
```

### Event Handler Pattern
```lavishscript
; Register event
Event[EQ2_onIncomingChatText]:AttachAtom[OnChat]

; Define atom handler
atom OnChat(int ChatType, string Message, string Speaker, ...)
{
    if ${ChatType} == 7  ; Tell
    {
        echo "Tell from ${Speaker}: ${Message}"
    }
}
```

### Wait for ISXEQ2 Pattern
```lavishscript
function main()
{
    while !${ISXEQ2.IsReady}
        wait 10

    ; Script logic here
}
```

---

## Common Events

| Event | When It Fires | Arguments |
|-------|---------------|-----------|
| `EQ2_ActorSpawned` | Actor appears | ID, Name, Level, Type |
| `EQ2_ActorDespawned` | Actor disappears | ID, Name |
| `EQ2_onIncomingChatText` | Chat message received | ChatType, Message, Speaker, ... |
| `EQ2_onInventoryUpdate` | Inventory changes | None |
| `EQ2_onLootWindowAppeared` | Loot window opens | WindowID |
| `EQ2_StartedZoning` | Zoning begins | None |
| `EQ2_FinishedZoning` | Zoning completes | TimeInSeconds |
| `EQ2_onQuestOffered` | Quest offered | Name, Description, Level, StatusReward |
| `EQ2_onLevelChange` | Character levels up | OldLevel, NewLevel |

---

## Control Flow

### If Statement
```lavishscript
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
```

### While Loop
```lavishscript
while ${Condition}
{
    ; Code
    wait 10
}
```

### For Loop
```lavishscript
variable int i
for (i:Set[1]; ${i} <= 10; i:Inc)
{
    ; Code
}
```

### Switch Statement
```lavishscript
switch ${Variable}
{
    case Value1
        ; Code
        break
    case Value2
        ; Code
        break
    default
        ; Code
        break
}
```

---

## Variable Declarations

### Basic Types
```lavishscript
declare MyInt int script 100
declare MyFloat float script 3.14
declare MyString string script "Hello"
declare MyBool bool script TRUE
```

### Object Types
```lavishscript
variable item MyItem
variable actor MyActor
variable ability MyAbility
```

### Collections
```lavishscript
variable index:item Items
variable collection:string Names
variable iterator MyIterator
```

### Scopes
```lavishscript
declare ScriptVar int script 0      ; Persists across functions
variable LocalVar int local 0        ; Only exists in function
```

---

## String Manipulation

```lavishscript
${String.Length}                ; Length
${String.Left[5]}              ; First 5 characters
${String.Right[3]}             ; Last 3 characters
${String.Upper}                ; UPPERCASE
${String.Lower}                ; lowercase
${String.Find["text"]}         ; Position of "text" (0 if not found)
${String.Equal["text"]}        ; TRUE if equal (case-sensitive)
${String.Replace["old","new"]} ; Replace text
${String.Token[2,:]}           ; Second token separated by ':'
```

---

## Math Operations

```lavishscript
${Math.Calc[10+5]}             ; 15
${Math.Calc[10-5]}             ; 5
${Math.Calc[10*5]}             ; 50
${Math.Calc[10/5]}             ; 2
${Math.Calc[(10+5)*2]}         ; 30
${Math.Rand[100]}              ; Random 0-99
${Math.Rand[100]:Inc}          ; Random 1-100
```

---

## Wait Commands

```lavishscript
wait 1000                      ; Wait 1 second (1000 milliseconds)
wait 50 ${Condition}           ; Wait up to 5 seconds for condition
waitframe                      ; Wait one frame (~16ms)
```

---

## Common Window TLOs

```lavishscript
${LootWindow}                  ; Loot window
${LootWindow.NumItems}         ; Number of items
LootWindow:LootAll             ; Loot everything

${RewardWindow}                ; Reward selection window
${RewardWindow.NumRewards}     ; Number of rewards
RewardWindow:AcceptReward[linkid]

${MerchantWindow}              ; Merchant window
${MerchantWindow.Item[1]}      ; First merchant item
MerchantWindow.Item[1]:Buy[5]  ; Buy 5 of item

${QuestJournalWindow}          ; Quest journal
${QuestJournalWindow.NumActiveQuests}
${QuestJournalWindow.ActiveQuest[1]}
```

---

## Datatype Inheritance

Understanding inheritance helps you know what members are available:

```
actor
  ├─ char (Me)
  └─ groupmember (Me.Group[1])

eq2window
  ├─ journalquestwindow (QuestJournalWindow)
  ├─ merchantwindow (MerchantWindow)
  └─ eq2clonewindow
      ├─ lootwindow (LootWindow)
      ├─ rewardwindow (RewardWindow)
      └─ containerwindow (ContainerWindow)

eq2baseobject
  └─ eq2widget
      ├─ eq2button
      ├─ eq2text
      ├─ eq2icon
      └─ eq2uipage
```

---

## Function Syntax

### Basic Function
```lavishscript
function MyFunction()
{
    echo "Function called"
}

call MyFunction
```

### Function with Parameters
```lavishscript
function Greet(string name, int level)
{
    echo "Hello ${name}, level ${level}!"
}

call Greet "Bob" 120
```

### Function with Return
```lavishscript
function GetHealth() returns int
{
    return ${Me.CurrentHealth}
}

variable int HP
HP:Set[${GetHealth}]
```

---

## Debugging Tips

### Echo Debugging
```lavishscript
echo "Debug: Variable = ${MyVariable}"
echo "Debug: Target exists? ${Target(exists)}"
```

### Conditional Debug
```lavishscript
declare DEBUG_MODE bool script TRUE

if ${DEBUG_MODE}
    echo "Debug: Reached checkpoint 1"
```

### Type Checking
```lavishscript
echo "Type of Me: ${Me.Type}"
echo "Type of Target: ${Target.Type}"
```

---

## Best Practices Checklist

- ✅ Always check `(exists)` before accessing objects
- ✅ Check `Is*InfoAvailable` before accessing detailed info
- ✅ Use `wait 10` in loops to prevent CPU spikes
- ✅ Cache frequently accessed values in variables
- ✅ Use early returns to reduce nesting
- ✅ Name variables descriptively
- ✅ Wait for `${ISXEQ2.IsReady}` at script start
- ✅ Add comments for complex logic
- ✅ Use timeouts when waiting for async data
- ✅ Handle errors gracefully

---

## Quick Troubleshooting

| Problem | Solution |
|---------|----------|
| "Invalid type" error | Add `(exists)` check before access |
| Script hangs | Add `wait` in loops |
| Can't access detailed info | Check `Is*InfoAvailable` first |
| Variable always empty | Check variable scope (script vs local) |
| Event not firing | Verify event name and attach syntax |
| Ability won't cast | Check `IsReady` and existence |

---

<!-- CLAUDE_SKIP_START -->
## Resource Files

- **Official Reference:** `..\+ISXEQ2_Reference+.md`
- **Example Scripts:** https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts
- **EQ2Bot:** https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Bot

---

## For More Information

- **Getting Started:** [02_Quick_Start_Guide.md](02_Quick_Start_Guide.md)
- **Fundamental Concepts:** [04_Core_Concepts.md](04_Core_Concepts.md)
- **Code Examples:** [06_Working_Examples.md](06_Working_Examples.md)
- **Best Practices:** [05_Patterns_And_Best_Practices.md](05_Patterns_And_Best_Practices.md)
- **Production Patterns:** [15_Advanced_Scripting_Patterns.md](15_Advanced_Scripting_Patterns.md)
<!-- CLAUDE_SKIP_END -->

---

*Part of ISXEQ2 Scripting Guide*
