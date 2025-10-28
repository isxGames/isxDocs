# ISXEQ2 Patterns and Best Practices

Proven patterns and best practices learned from analyzing production ISXEQ2 scripts, particularly [EQ2Bot](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Bot) (7,500+ lines) and its 20+ class routines.

---

## Table of Contents

1. [Naming Conventions](#naming-conventions)
2. [Code Organization](#code-organization)
3. [Variable Declaration Patterns](#variable-declaration-patterns)
4. [Error Handling Strategies](#error-handling-strategies)
5. [Performance Patterns](#performance-patterns)
6. [Function Design](#function-design)
7. [Event Handling Patterns](#event-handling-patterns)
8. [Configuration Management](#configuration-management)
9. [Common Anti-Patterns](#common-anti-patterns)
10. [Professional Script Structure](#professional-script-structure)

---

## Naming Conventions

### Variables

**Boolean Flags:**
```lavishscript
; Use descriptive PascalCase with Mode/Active/Enabled suffix
declare AoEMode bool script FALSE
declare FullAutoMode bool script TRUE
declare BuffingActive bool script FALSE
declare DebugEnabled bool script FALSE
```

**Counters and Timers:**
```lavishscript
; Use descriptive name + Timer/Counter suffix
declare ClassPulseTimer int64 script 0
declare EquipmentChangeTimer int64 script 0
declare AttackCounter int script 0
declare LoopIterations int script 0
```

**Action Names:**
```lavishscript
; Use descriptive PascalCase
declare CurrentAction string script "Buff_Self"
variable string ActionName = "AoE_Taunt1"
variable string SpellAction = "Melee_Attack2"
```

**Collections:**
```lavishscript
; Indicate the type being collected
declare MezSpells collection:int script
declare DoNotPullList collection:string script
declare VimBuffsOn collection:string script
```

**Local Variables:**
```lavishscript
; Use descriptive lowercase or short abbreviations
variable int temphl
variable int grpheal
variable int buffmember
variable string tempnme
```

### Settings/INI Keys

**Use human-readable names with spaces:**
```lavishscript
AoEMode:Set[${CharacterSet.FindSet[${Me.SubClass}].FindSetting[Cast AoE Spells,FALSE]}]
AutoFollowMode:Set[${CharacterSet.FindSet[EQ2BotExtras].FindSetting[Auto Follow Mode,FALSE]}]
```

### Functions

**Use descriptive verbs:**
```lavishscript
function CheckHeals()
function CastSpellRange(int start, int finish)
function ProcessBuffQueue()
function HandleCombatRotation()
```

---

## Code Organization

### File Structure (Based on EQ2Bot)

```lavishscript
;*****************************************************
; ClassName.iss
; by Author
; Version History:
; 20240101a - Description of changes
; 20231201a - Initial version
;*****************************************************

; ============================================
; ===  Include Guards and Includes        ===
; ============================================

#ifndef _ClassFile_
#define _ClassFile_

#include "${LavishScript.HomeDirectory}/Scripts/EQ2Bot/Class Routines/EQ2BotLib.iss"

; ============================================
; ===  Global Declarations                ===
; ============================================

declare ClassFileVersion int script 20240101
declare AoEMode bool script FALSE
; ... more declarations

; ============================================
; ===  Initialization                     ===
; ============================================

function Class_Declaration()
{
    ; Load settings, initialize UI, setup variables
}

; ============================================
; ===  Core Functions                     ===
; ============================================

function Pulse()
{
    ; Called every ~500-2000ms for continuous checks
}

function Buff_Init()
{
    ; Initialize out-of-combat buffs
}

function Buff_Routine(int xAction)
{
    ; Execute buff actions
}

function Combat_Init()
{
    ; Initialize combat rotations
}

function Combat_Routine(int xAction)
{
    ; Execute combat actions
}

function PostCombat_Init()
{
    ; Initialize post-combat recovery
}

function Post_Combat_Routine(int xAction)
{
    ; Execute post-combat actions
}

; ============================================
; ===  Event Handlers                     ===
; ============================================

function Have_Aggro()
{
    ; Called when gaining aggro
}

function Lost_Aggro()
{
    ; Called when losing aggro
}

; ============================================
; ===  Helper Functions                   ===
; ============================================

function CheckHeals()
{
    ; Helper for heal checks
}

function ProcessDebuffs()
{
    ; Helper for debuff logic
}

; End of include guard
#endif
```

### Section Headers

**Use clear visual separators:**
```lavishscript
;===================================================
;===        Keyboard Configuration              ====
;===================================================

;===================================================
;===        Buff Configuration                  ====
;===================================================

;===================================================
;===        Combat Rotation Setup               ====
;===================================================
```

---

## Variable Declaration Patterns

### Script-Scoped (Persistent)

**Use for settings and state that must persist:**
```lavishscript
; Boolean flags
declare AoEMode bool script FALSE
declare FullAutoMode bool script TRUE
declare DoHOs bool script TRUE

; Configuration
declare TowerShield string script "Guardian's Tower Shield"
declare OffensiveStance string script "Offensive Stance"

; Arrays indexed by priority
declare PreAction collection:string script
declare PreSpellRange collection:int script
declare PreMobHealth collection:int script

; Timers (use int64 for timestamps)
declare ClassPulseTimer int64 script 0
declare EquipmentChangeTimer int64 script 0
```

### Local-Scoped (Temporary)

**Use for calculations and temporary values:**
```lavishscript
function CheckHeals()
{
    ; Local variables at top of function
    declare temphl int local
    declare grpheal int local 0
    declare lowest int local 0
    declare BuffTarget string local

    ; Function logic below
    ; ...
}
```

### Shorthand Variables Pattern

**Reduce repetitive typing:**
```lavishscript
; Instead of:
echo ${Me.Equipment[primary].ToItemInfo.DamageRating}
echo ${Me.Equipment[primary].ToItemInfo.Delay}
echo ${Me.Equipment[primary].ToItemInfo.Range}

; Use:
variable item MyWeapon
MyWeapon:Set[${Me.Equipment[primary]}]

if ${MyWeapon.IsItemInfoAvailable}
{
    echo ${MyWeapon.ToItemInfo.DamageRating}
    echo ${MyWeapon.ToItemInfo.Delay}
    echo ${MyWeapon.ToItemInfo.Range}
}
```

---

## Error Handling Strategies

### Existence Checks (Most Important!)

**Always check before accessing:**
```lavishscript
; BAD
Me.Inventory["Potion"]:Use

; GOOD
if ${Me.Inventory[ExactName,"Health Potion"](exists)}
{
    if ${Me.Inventory[ExactName,"Health Potion"].IsReady}
    {
        Me.Inventory[ExactName,"Health Potion"]:Use
    }
}
```

### Compound Safety Checks

**Check multiple conditions:**
```lavishscript
; Check item exists AND is ready before use
if ${Me.Inventory[ExactName,"Brock's Thermal Shocker"](exists)} && \
   ${Me.Inventory[ExactName,"Brock's Thermal Shocker"].IsReady}
{
    Me.Inventory[ExactName,"Brock's Thermal Shocker"]:Use
}

; Check ability exists AND is ready
if ${Me.Ability[${SpellType[156]}](exists)} && \
   ${Me.Ability[${SpellType[156]}].IsReady} && \
   ${Me.Health} < 60
{
    call CastSpellRange 156
}
```

### Actor Validation

**Validate actor before targeting:**
```lavishscript
; Check actor exists and is valid
if ${Actor[${BuffTarget.Token[2,:]},${BuffTarget.Token[1,:]}].Name(exists)} && \
   !${Actor[${BuffTarget.Token[2,:]},${BuffTarget.Token[1,:]}].IsDead}
{
    call CastSpellRange ${PreSpellRange[${xAction},1]} 0 0 0 ${Actor[...].ID}
}
```

### Collection Validation

**Check collection before iterating:**
```lavishscript
variable index:item Items
variable iterator ItemIt

Me:QueryInventory[Items,"Name =- Potion"]
Items:GetIterator[ItemIt]

; Check iterator is valid
if ${ItemIt:First(exists)}
{
    do
    {
        ; Process item
        echo ${ItemIt.Value.Name}
    }
    while ${ItemIt:Next(exists)}
}
else
{
    echo "No items found"
}
```

### Async Data Timeouts

**Use timeouts to prevent infinite waits:**
```lavishscript
variable item MyItem
MyItem:Set[${Me.Inventory[5]}]

if !${MyItem.IsItemInfoAvailable}
{
    variable int Timeout = 0
    do
    {
        waitframe
    }
    while !${MyItem.IsItemInfoAvailable} && ${Timeout:Inc} < 1500
}

; Check if we timed out
if ${MyItem.IsItemInfoAvailable}
{
    echo ${MyItem.ToItemInfo.Description}
}
else
{
    echo "Timed out waiting for item info"
}
```

---

## Performance Patterns

### Multi-Timer Pulse System (EQ2Bot Pattern)

**Different check frequencies for efficiency:**
```lavishscript
; Main loop with multiple timers
while ${StartBot}
{
    ; 1-second checks (frequent)
    if ${Script.RunningTime} >= ${Math.Calc64[${MainPulse1SecondTimer}+1000]}
    {
        MainPulse1SecondTimer:Set[${Script.RunningTime}]
        ; Quick checks: health, power, immediate threats
    }

    ; 2-second checks (moderate)
    if ${Script.RunningTime} >= ${Math.Calc64[${MainPulse2SecondTimer}+2000]}
    {
        MainPulse2SecondTimer:Set[${Script.RunningTime}]
        ; Buff checks, ability cooldowns
    }

    ; 5-second checks (less frequent)
    if ${Script.RunningTime} >= ${Math.Calc64[${MainPulse5SecondTimer}+5000]}
    {
        MainPulse5SecondTimer:Set[${Script.RunningTime}]
        ; Equipment checks, long-term monitoring
    }

    ; Class-specific pulse
    if ${Script[Class].Defined}
    {
        call Pulse
    }

    wait 10  ; Prevent excessive CPU usage
}
```

### Lazy Evaluation

**Only check when needed:**
```lavishscript
; BAD - Checks every loop
while ${Running}
{
    if ${Me.Health} < 50
        call UseHeal
    if ${Me.Power} < 30
        call UsePowerRegen
    wait 10
}

; GOOD - Only check periodically
while ${Running}
{
    if ${Script.RunningTime} >= ${Math.Calc64[${LastHealthCheck}+2000]}
    {
        if ${Me.Health} < 50
            call UseHeal
        if ${Me.Power} < 30
            call UsePowerRegen
        LastHealthCheck:Set[${Script.RunningTime}]
    }
    wait 10
}
```

### Early Returns

**Exit functions early when conditions aren't met:**
```lavishscript
function CastBuff()
{
    ; Guard clauses at the top
    if !${Me(exists)}
        return

    if ${Me.InCombat}
        return

    if !${BuffingEnabled}
        return

    ; Main logic only runs if all checks pass
    ; ... buffing logic here
}
```

### Minimize API Calls

**Cache frequently accessed values:**
```lavishscript
; BAD - Multiple API calls
if ${Me.CurrentHealth} < 5000
{
    echo "Health: ${Me.CurrentHealth}"
    call Emergency  Heal ${Me.CurrentHealth}
}

; GOOD - Cache the value
variable int CurrentHP
CurrentHP:Set[${Me.CurrentHealth}]

if ${CurrentHP} < 5000
{
    echo "Health: ${CurrentHP}"
    call EmergencyHeal ${CurrentHP}
}
```

---

## Function Design

### Template Pattern (Class Compatibility)

**All class files implement the same interface:**
```lavishscript
; Required functions for class compatibility
function Class_Declaration()
{
    ; Setup and initialization
}

function Pulse()
{
    ; Called periodically for continuous checks
}

function Buff_Init()
{
    ; Initialize buff actions
}

function Buff_Routine(int xAction)
{
    ; Execute buff by index
}

function Combat_Init()
{
    ; Initialize combat rotations
}

function Combat_Routine(int xAction)
{
    ; Execute combat action by index
}

function PostCombat_Init()
{
    ; Initialize post-combat recovery
}

function Post_Combat_Routine(int xAction)
{
    ; Execute recovery action by index
}

; Event handlers
function Have_Aggro()
function Lost_Aggro()
function MA_Dead()
```

### Action Dispatch Pattern

**Use arrays indexed by priority:**
```lavishscript
; Configuration arrays
variable string PreAction[40]           ; Action names
variable int PreSpellRange[40,5]        ; Spell indices
variable int PreMobHealth[40,2]         ; Health thresholds

; Populate actions
PreAction[1]:Set["Self_Buff"]
PreSpellRange[1,1]:Set[10]  ; Spell index
PreMobHealth[1,1]:Set[0]    ; Min health
PreMobHealth[1,2]:Set[100]  ; Max health

PreAction[2]:Set["Group_Buff"]
PreSpellRange[2,1]:Set[20]
PreSpellRange[2,2]:Set[25]

; Dispatch function
function Buff_Routine(int xAction)
{
    switch ${PreAction[${xAction}]}
    {
        case Self_Buff
            call CastSpellRange ${PreSpellRange[${xAction},1]}
            break

        case Group_Buff
            call CastSpellRange ${PreSpellRange[${xAction},1]} ${PreSpellRange[${xAction},2]}
            break

        case Avoidance_Target
            if !${BuffTarget.Equal["None"]} && ${Actor[${BuffTarget}].Name(exists)}
            {
                call CastSpellRange ${PreSpellRange[${xAction},1]} 0 0 0 ${Actor[${BuffTarget}].ID}
            }
            break
    }
}
```

### Parameter Patterns

**Consistent parameter ordering:**
```lavishscript
; CastSpellRange - standard parameter pattern from EQ2Bot
function CastSpellRange(int start, int finish, int rangetype, int quadrant, int TargetID, bool notall, int refreshtimer, bool castwhilemoving, bool cannotmove, bool oncooldown, bool instancespell)
{
    ; Positional arguments (most common):
    ; start - Spell index (e.g., 170)
    ; finish - Range end (e.g., 171) or 0 for single spell
    ; rangetype - 0=any, 1=close, 3=ranged
    ; quadrant - 0=any, 1=behind, 2=front, 3=flank
    ; TargetID - Target actor ID or 0 for self

    ; Optional parameters with sensible defaults
    if !${notall}
        notall:Set[FALSE]
    if !${refreshtimer}
        refreshtimer:Set[0]
    ; ... etc
}

; Common usage patterns:
call CastSpellRange 156                              ; Single spell, self
call CastSpellRange 156 0 0 0 0                      ; Explicit self target
call CastSpellRange 170 171 0 0 ${KillTarget}       ; Range with target
call CastSpellRange 322 0 1 0 ${KillTarget}         ; Position-specific
```

---

## Event Handling Patterns

### Chat Trigger Pattern

```lavishscript
; Define triggers (in initialization)
AddTrigger AutoFollowTank "\\aPC @*@ @*@:@sender@\\/a tells@*@Follow Me@*@"
AddTrigger ReceivedTell "\\aPC @*@ @*@:@Sender@\\/a tells you,@Message@"

; Attach event
Event[EQ2_onIncomingChatText]:AttachAtom[ChatText]

; Event atom handler
atom ChatText(string line, string Sender, string Message)
{
    ; Check if line matches trigger patterns
    if ${ReceivedTell.Matches[${line}]}
    {
        call ReceivedTell "${line}" "${Sender}" "${Message}"
    }

    if ${AutoFollowTank.Matches[${line}]}
    {
        call StartFollowing "${Sender}"
    }
}
```

### Window Appearance Pattern

```lavishscript
; Attach window event
Event[EQ2_onLootWindowAppeared]:AttachAtom[OnLootWindow]

; Event handler with delay
atom OnLootWindow(uint WindowID)
{
    ; Wait for window to fully populate
    wait 5

    ; Validate window still exists
    if !${LootWindow(exists)}
        return

    ; Process loot
    if ${LootWindow.NumItems} > 0
    {
        LootWindow:LootAll
        echo "Looted ${LootWindow.NumItems} items"
    }

    ; Close window
    LootWindow:Close
}
```

### State Change Pattern

```lavishscript
; Track previous state
declare PreviousHealth int script 100

; Monitor for changes
function Pulse()
{
    if ${Me.Health} != ${PreviousHealth}
    {
        ; Health changed - react
        if ${Me.Health} < 30 && ${PreviousHealth} >= 30
        {
            echo "Entered low health threshold!"
            call EmergencyHeal
        }

        PreviousHealth:Set[${Me.Health}]
    }
}
```

---

## Configuration Management

### Settings Loading Pattern

```lavishscript
function Class_Declaration()
{
    ; Initialize character settings
    CharacterSet:AddSet[${Me.SubClass}]
    CharacterSet:AddSet[EQ2BotExtras]

    ; Load setting values with defaults
    AoEMode:Set[${CharacterSet.FindSet[${Me.SubClass}].FindSetting[Cast AoE Spells,FALSE]}]
    AutoFollowMode:Set[${CharacterSet.FindSet[EQ2BotExtras].FindSetting[Auto Follow Mode,FALSE]}]
    MaxBuffDistance:Set[${CharacterSet.FindSet[${Me.SubClass}].FindSetting[Max Buff Distance,30]}]

    ; Load UI elements
    ui -load -parent "Class@EQ2Bot Tabs@EQ2 Bot" -skin EQ2-Green "${PATH_UI}/${Me.SubClass}.xml"
}
```

### XML Import Pattern

```lavishscript
; Load spell list from XML
LavishSettings[EQ2Bot]:AddSet[OtherSpells]
LavishSettings[EQ2Bot].FindSet[OtherSpells]:Import[${PATH_SPELL_LIST}/${Me.SubClass}.xml]

; Get iterator for spells
LavishSettings[EQ2Bot].FindSet[OtherSpells].FindSet[${Me.SubClass}]:GetSettingIterator[SpellIterator]

; Populate collection from XML
variable collection:int MezSpells

if ${SpellIterator:First(exists)}
{
    do
    {
        variable string tempnme = "${SpellIterator.Key}"
        variable int iLevel = ${tempnme.Token[1, ]}
        variable int iType = ${tempnme.Token[2, ]}
        variable string SpellName = "${SpellIterator.Value}"

        switch ${iType}
        {
            case 92
            case 352
                MezSpells:Set[${SpellName},${iLevel}]
                break
        }
    }
    while ${SpellIterator:Next(exists)}
}
```

### UI Binding Pattern

```lavishscript
; Get selected value from dropdown
variable string SelectedTarget
SelectedTarget:Set[${UIElement[cbBuffAvoidanceGroupMember@Class@EQ2Bot Tabs@EQ2 Bot].SelectedItem.Text}]

; Check checkbox state
if ${UIElement[cbEnableDebug@Class@EQ2Bot Tabs@EQ2 Bot].Checked}
{
    DebugMode:Set[TRUE]
}
```

---

## Common Anti-Patterns

### Anti-Pattern 1: No NULL Checks

```lavishscript
; WRONG - Will error if target doesn't exist
echo ${Target.Name}
Target:DoTarget

; RIGHT - Check first
if ${Target(exists)}
{
    echo ${Target.Name}
    Target:DoTarget
}
```

### Anti-Pattern 2: Ignoring Async Data

```lavishscript
; WRONG - May not be loaded yet
echo ${Me.Inventory[5].ToItemInfo.Description}

; RIGHT - Check availability
if ${Me.Inventory[5].IsItemInfoAvailable}
{
    echo ${Me.Inventory[5].ToItemInfo.Description}
}
else
{
    wait 10 ${Me.Inventory[5].IsItemInfoAvailable}
    echo ${Me.Inventory[5].ToItemInfo.Description}
}
```

### Anti-Pattern 3: Tight Loops Without Wait

```lavishscript
; WRONG - Consumes 100% CPU
while ${Running}
{
    ; Continuous checking with no delay
    if ${Me.Health} < 50
        call Heal
}

; RIGHT - Add wait to prevent CPU spike
while ${Running}
{
    if ${Me.Health} < 50
        call Heal
    wait 10  ; Prevent CPU overload
}
```

### Anti-Pattern 4: Repeated Complex Lookups

```lavishscript
; WRONG - Repeats same lookup
if ${Me.Equipment[primary].ToItemInfo.DamageRating} > 500
{
    echo ${Me.Equipment[primary].ToItemInfo.DamageRating}
    echo ${Me.Equipment[primary].ToItemInfo.Delay}
}

; RIGHT - Cache the value
variable item PrimaryWeapon
PrimaryWeapon:Set[${Me.Equipment[primary]}]

if ${PrimaryWeapon.IsItemInfoAvailable} && ${PrimaryWeapon.ToItemInfo.DamageRating} > 500
{
    echo ${PrimaryWeapon.ToItemInfo.DamageRating}
    echo ${PrimaryWeapon.ToItemInfo.Delay}
}
```

### Anti-Pattern 5: Magic Numbers

```lavishscript
; WRONG - Unclear what 50 means
if ${Me.Health} < 50
    call EmergencyHeal

; RIGHT - Use named constants
declare LOW_HEALTH_THRESHOLD int script 50

if ${Me.Health} < ${LOW_HEALTH_THRESHOLD}
    call EmergencyHeal
```

---

## Professional Script Structure

### Complete Example

```lavishscript
;*****************************************************
; MyBot.iss - Professional Bot Template
; by Author
; Version 1.0.0 - 2024-01-01
;*****************************************************

; ============================================
; ===  Script Configuration               ===
; ============================================

declare SCRIPT_VERSION string script "1.0.0"
declare DEBUG_MODE bool script FALSE

; ============================================
; ===  Constants                          ===
; ============================================

declare LOW_HEALTH_THRESHOLD int script 30
declare LOW_POWER_THRESHOLD int script 20
declare PULSE_INTERVAL int script 500

; ============================================
; ===  State Variables                    ===
; ============================================

declare Running bool script TRUE
declare InCombat bool script FALSE
declare LastPulseTime int64 script 0

; ============================================
; ===  Main Entry Point                   ===
; ============================================

function main()
{
    ; Wait for ISXEQ2
    while !${ISXEQ2.IsReady}
        wait 10

    ; Initialize
    call Initialize

    ; Register events
    call RegisterEvents

    echo "MyBot ${SCRIPT_VERSION} started"

    ; Main loop
    while ${Running}
    {
        call MainPulse
        wait 10
    }

    ; Cleanup
    call Shutdown
}

; ============================================
; ===  Initialization                     ===
; ============================================

function Initialize()
{
    ; Load configuration
    call LoadConfiguration

    ; Setup UI
    call SetupUI

    ; Initialize subsystems
    call InitializeCombat
    call InitializeBuffing
}

; ============================================
; ===  Main Logic                         ===
; ============================================

function MainPulse()
{
    ; Throttle pulse frequency
    if ${Script.RunningTime} < ${Math.Calc64[${LastPulseTime}+${PULSE_INTERVAL}]}
        return

    LastPulseTime:Set[${Script.RunningTime}]

    ; Safety check
    if !${Me(exists)}
        return

    ; Update state
    call UpdateCombatState

    ; Execute appropriate routine
    if ${InCombat}
    {
        call CombatRoutine
    }
    else
    {
        call IdleRoutine
    }

    ; Continuous checks
    call CheckHealth
    call CheckPower
}

; ============================================
; ===  Combat System                      ===
; ============================================

function UpdateCombatState()
{
    InCombat:Set[${Me.InCombat}]
}

function CombatRoutine()
{
    ; Combat logic here
}

function IdleRoutine()
{
    ; Idle/buffing logic here
}

; ============================================
; ===  Health/Power Management            ===
; ============================================

function CheckHealth()
{
    if ${Me.Health} < ${LOW_HEALTH_THRESHOLD}
    {
        call EmergencyHeal
    }
}

function CheckPower()
{
    if ${Me.Power} < ${LOW_POWER_THRESHOLD}
    {
        call UsePowerRegen
    }
}

; ============================================
; ===  Event Handlers                     ===
; ============================================

function RegisterEvents()
{
    Event[EQ2_onIncomingChatText]:AttachAtom[OnChat]
    Event[EQ2_StartedZoning]:AttachAtom[OnZoneStart]
    Event[EQ2_FinishedZoning]:AttachAtom[OnZoneEnd]
}

atom OnChat(int ChatType, string Message, string Speaker, string Target, string SpeakerIsNPC, string ChannelName, int SpeakerID, int TargetID, string UnkString1)
{
    ; Handle chat events
}

atom OnZoneStart()
{
    echo "Zoning started..."
    Running:Set[FALSE]
}

atom OnZoneEnd(string TimeInSeconds)
{
    echo "Arrived in ${Zone.Name} (${TimeInSeconds}s)"
    Running:Set[TRUE]
}

; ============================================
; ===  Utility Functions                  ===
; ============================================

function DebugLog(string message)
{
    if ${DEBUG_MODE}
        echo "[DEBUG] ${message}"
}

; ============================================
; ===  Shutdown                           ===
; ============================================

function Shutdown()
{
    ; Detach events
    Event[EQ2_onIncomingChatText]:DetachAtom[OnChat]
    Event[EQ2_StartedZoning]:DetachAtom[OnZoneStart]
    Event[EQ2_FinishedZoning]:DetachAtom[OnZoneEnd]

    echo "MyBot shutdown complete"
}
```

---

## Summary of Best Practices

1. **✅ Always check (exists)** before accessing objects
2. **✅ Use multi-timer pulse** systems for efficiency
3. **✅ Cache frequently accessed** values
4. **✅ Use early returns** to reduce nesting
5. **✅ Check async data** availability before access
6. **✅ Name variables** descriptively
7. **✅ Organize code** with clear sections
8. **✅ Handle errors** gracefully with timeouts
9. **✅ Use templates** for consistency
10. **✅ Document** with version history and comments

---

*Part of ISXEQ2 Scripting Guide*
*Analysis based on EQ2Bot: https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Bot*
