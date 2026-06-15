# ISXPantheon Patterns and Best Practices

Proven LavishScript patterns and best practices for building robust ISXPantheon scripts. The patterns below are platform-generic (they apply to any InnerSpace extension); they are illustrated with the real ISXPantheon surface where one exists, and with clearly-labeled planned placeholders where the game data surface is still on the roadmap.

> **NOTE — surface maturity.** The live ISXPantheon surface today is `${ISXPantheon}` (version / `IsReady` / custom variables / currency / rounding) plus the `GetURL` / `PostURL` commands. Game-data objects such as the local player, targets, abilities, and inventory are **(planned — not yet implemented)**. Examples that reference those are marked accordingly and must not be treated as runnable today.

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
AoEMode:Set[${ConfigSet.FindSet[General].FindSetting[Cast AoE Spells,FALSE]}]
AutoFollowMode:Set[${ConfigSet.FindSet[Extras].FindSetting[Auto Follow Mode,FALSE]}]
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

### File Structure (Modular Bot Layout)

```lavishscript
;*****************************************************
; ModuleName.iss
; by Author
; Version History:
; 20240101a - Description of changes
; 20231201a - Initial version
;*****************************************************

; ============================================
; ===  Include Guards and Includes        ===
; ============================================

#ifndef _ModuleFile_
#define _ModuleFile_

#include "${LavishScript.HomeDirectory}/Scripts/MyBot/Routines/CommonLib.iss"

; ============================================
; ===  Global Declarations                ===
; ============================================

declare ClassFileVersion int script 20240101
declare AoEMode bool script FALSE
; ... more declarations

; ============================================
; ===  Initialization                     ===
; ============================================

function Module_Declaration()
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
declare PrimaryShield string script "Reinforced Tower Shield"
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

**Reduce repetitive typing by caching a deep object reference into a local:**
```lavishscript
; Instead of repeating a long object chain:
echo ${SomeObject.Child.Detail.FieldA}
echo ${SomeObject.Child.Detail.FieldB}
echo ${SomeObject.Child.Detail.FieldC}

; Cache the reference once, then reuse the shorthand:
variable string CachedName = "${ISXPantheon.GetCustomVariable[PrimaryWeaponName]}"

echo ${CachedName}

; The same idea applies to any future game object: assign it to a typed
; local variable once, then access its members through the shorthand.
```

---

## Error Handling Strategies

### Existence Checks (Most Important!)

**Always check before accessing.** Querying a member off an object that does not exist is the single most common source of script errors. The `(exists)` test is the universal guard:

```lavishscript
; BAD - assumes the object is present
GameObject["Foo"]:DoSomething

; GOOD - guard with (exists), then guard the action condition
if ${GameObject["Foo"](exists)}
{
    if ${GameObject["Foo"].IsReady}
    {
        GameObject["Foo"]:DoSomething
    }
}
```

> **PLANNED — NOT YET IMPLEMENTED.** Game-data objects are on the ISXPantheon roadmap. The `(exists)` / `.IsReady` pattern shown here is the correct shape to use once those objects are surfaced, but no game-data members are available in the current build — the only object with working members today is `${ISXPantheon}`.

### Compound Safety Checks

**Check multiple conditions with `&&` so a later access never runs on a missing object:**
```lavishscript
; Generic, platform-safe example: only act when BOTH conditions hold
variable bool ReadyFlag = ${ISXPantheon.GetCustomVariable[ActionReady,bool]}

if ${ISXPantheon.IsReady} && ${ReadyFlag}
{
    echo "All preconditions met - proceeding"
}
```

> **PLANNED — NOT YET IMPLEMENTED.** A typical game-data compound check (object exists AND is ready AND a resource threshold is met) follows the same `&&` chaining shape, but the player/ability objects it would reference are not yet surfaced.

### Object Validation

**Validate an object exists and is in a usable state before acting on it:**
```lavishscript
; Generic shape - confirm presence, then confirm a usability flag
if ${GameObject["Subject"](exists)} && !${GameObject["Subject"].IsDisabled}
{
    GameObject["Subject"]:Activate
}
```

> **PLANNED — NOT YET IMPLEMENTED.** Validating a targetable entity (exists, not dead, in range) before acting on it follows this shape; entity/target objects are planned, not available today.

### Collection Validation

**Check an iterator is valid before iterating:**
```lavishscript
variable collection:string Names
variable iterator NameIt

; Populate the collection from any source (config, query, etc.)
Names:Set["alpha","alpha"]
Names:Set["beta","beta"]
Names:GetIterator[NameIt]

; Check iterator is valid before the loop
if ${NameIt:First(exists)}
{
    do
    {
        echo ${NameIt.Value}
    }
    while ${NameIt:Next(exists)}
}
else
{
    echo "No entries found"
}
```

### Async Data Timeouts

**Use timeouts to prevent infinite waits when data loads asynchronously.** Many extension members populate on a background frame; never block forever waiting on them:

```lavishscript
; Generic async-wait shape with a bounded retry count
variable int Timeout = 0

while !${ISXPantheon.IsReady} && ${Timeout:Inc} < 1500
{
    waitframe
}

if ${ISXPantheon.IsReady}
{
    echo "Extension ready after ${Timeout} frames"
}
else
{
    echo "Timed out waiting for the extension"
}
```

The same bounded-wait shape applies whenever you wait on a member that populates on a background frame. The other asynchronous mechanism ISXPantheon provides today is HTTP: `GetURL` / `PostURL` deliver their results through the `isxGames_onHTTPResponse` event rather than a polled member.

---

## Performance Patterns

### Multi-Timer Pulse System

**Different check frequencies for efficiency.** This pattern is platform-generic: it uses only `${Script.RunningTime}` and `${Math.Calc64[...]}`, so it works today regardless of which game objects are surfaced.

```lavishscript
; Main loop with multiple timers
while ${StartBot}
{
    ; 1-second checks (frequent)
    if ${Script.RunningTime} >= ${Math.Calc64[${MainPulse1SecondTimer}+1000]}
    {
        MainPulse1SecondTimer:Set[${Script.RunningTime}]
        ; Quick checks: high-priority state, immediate reactions
    }

    ; 2-second checks (moderate)
    if ${Script.RunningTime} >= ${Math.Calc64[${MainPulse2SecondTimer}+2000]}
    {
        MainPulse2SecondTimer:Set[${Script.RunningTime}]
        ; Periodic state, cooldown bookkeeping
    }

    ; 5-second checks (less frequent)
    if ${Script.RunningTime} >= ${Math.Calc64[${MainPulse5SecondTimer}+5000]}
    {
        MainPulse5SecondTimer:Set[${Script.RunningTime}]
        ; Slow-changing state, long-term monitoring
    }

    ; Module-specific pulse
    if ${Script[Module].Defined}
    {
        call Pulse
    }

    wait 10  ; Prevent excessive CPU usage
}
```

### Lazy Evaluation

**Only do expensive work when needed.** Gate costly checks behind a timer instead of running them every loop iteration:

```lavishscript
; BAD - runs the expensive check every single loop
while ${Running}
{
    call ExpensiveStateCheck
    wait 10
}

; GOOD - only run the expensive check every 2 seconds
while ${Running}
{
    if ${Script.RunningTime} >= ${Math.Calc64[${LastStateCheck}+2000]}
    {
        call ExpensiveStateCheck
        LastStateCheck:Set[${Script.RunningTime}]
    }
    wait 10
}
```

> **PLANNED — NOT YET IMPLEMENTED.** In a combat script the gated work would read player health/power and react; those player members are planned, so the example above uses a generic `ExpensiveStateCheck` placeholder.

### Early Returns

**Exit functions early when conditions aren't met:**
```lavishscript
function DoWork()
{
    ; Guard clauses at the top
    if !${ISXPantheon.IsReady}
        return

    if !${WorkEnabled}
        return

    ; Main logic only runs if all checks pass
    ; ... work here
}
```

> **PLANNED — NOT YET IMPLEMENTED.** A real combat routine would add guards like "skip if no local player" or "skip while in combat"; those player-state members are planned. The guard-clause structure itself is what matters here.

### Minimize API Calls

**Cache frequently accessed values** instead of re-evaluating the same object chain repeatedly:
```lavishscript
; BAD - reads the same value three times
if ${ISXPantheon.GetCustomVariable[Threshold,int]} < 5000
{
    echo "Threshold: ${ISXPantheon.GetCustomVariable[Threshold,int]}"
    call Process ${ISXPantheon.GetCustomVariable[Threshold,int]}
}

; GOOD - read once, reuse the local
variable int Threshold
Threshold:Set[${ISXPantheon.GetCustomVariable[Threshold,int]}]

if ${Threshold} < 5000
{
    echo "Threshold: ${Threshold}"
    call Process ${Threshold}
}
```

---

## Function Design

### Template Pattern (Pluggable Modules)

**Every interchangeable module implements the same interface,** so the host loop can call them uniformly regardless of which module is loaded:
```lavishscript
; Required functions for module compatibility
function Module_Declaration()
{
    ; Setup and initialization
}

function Pulse()
{
    ; Called periodically for continuous checks
}

function Phase1_Init()
{
    ; Initialize first-phase actions
}

function Phase1_Routine(int xAction)
{
    ; Execute first-phase action by index
}

function Phase2_Init()
{
    ; Initialize second-phase actions
}

function Phase2_Routine(int xAction)
{
    ; Execute second-phase action by index
}

function Recovery_Init()
{
    ; Initialize recovery actions
}

function Recovery_Routine(int xAction)
{
    ; Execute recovery action by index
}

; State-transition handlers
function OnEngage()
function OnDisengage()
function OnFailure()
```

### Action Dispatch Pattern

**Use arrays indexed by priority,** then dispatch on the action name with a `switch`:
```lavishscript
; Configuration arrays
variable string PreAction[40]           ; Action names
variable int PreParam[40,5]             ; Action parameters
variable int PreThreshold[40,2]         ; Trigger thresholds

; Populate actions
PreAction[1]:Set["Self_Action"]
PreParam[1,1]:Set[10]       ; Primary parameter
PreThreshold[1,1]:Set[0]    ; Min threshold
PreThreshold[1,2]:Set[100]  ; Max threshold

PreAction[2]:Set["Group_Action"]
PreParam[2,1]:Set[20]
PreParam[2,2]:Set[25]

; Dispatch function
function Action_Routine(int xAction)
{
    switch ${PreAction[${xAction}]}
    {
        case Self_Action
            call DoAction ${PreParam[${xAction},1]}
            break

        case Group_Action
            call DoAction ${PreParam[${xAction},1]} ${PreParam[${xAction},2]}
            break

        case Conditional_Action
            if !${SelectedName.Equal["None"]}
            {
                call DoAction ${PreParam[${xAction},1]}
            }
            break
    }
}
```

> **PLANNED — NOT YET IMPLEMENTED.** A combat dispatcher would resolve `Conditional_Action` against a live target entity (exists / not dead / in range) before acting; entity/target objects are planned. The array-plus-`switch` dispatch structure shown here is fully usable today.

### Parameter Patterns

**Consistent parameter ordering** makes a heavily-overloaded function easy to call. Put the required positional arguments first, give optional trailing parameters sensible defaults:
```lavishscript
; Standard parameter pattern: required first, optional with defaults
function DoAction(int start, int finish, int mode, int variant, int subjectID, bool flagA, int retryTimer)
{
    ; Positional arguments (most common):
    ; start     - Primary action index
    ; finish    - Range end, or 0 for a single action
    ; mode      - 0=default, 1=close, 3=extended
    ; variant   - 0=any, 1..3 = specific variant
    ; subjectID - Subject id, or 0 for self

    ; Optional parameters with sensible defaults
    if !${flagA}
        flagA:Set[FALSE]
    if !${retryTimer}
        retryTimer:Set[0]
    ; ... etc
}

; Common usage patterns:
call DoAction 156                       ; Single action, self
call DoAction 156 0 0 0 0               ; Explicit self subject
call DoAction 170 171 0 0 ${SubjectID}  ; Range with subject
call DoAction 322 0 1 0 ${SubjectID}    ; Variant-specific
```

---

## Event Handling Patterns

### Chat Trigger Pattern

Triggers match incoming chat text against patterns registered with `AddTrigger` and queue named handler functions for later execution via `ExecuteQueued`. This enables chat-driven automation such as auto-follow on request, auto-response to tells, loot tracking, and combat event reactions. For the canonical reference -- including placeholder syntax (`@TYPE@`, `@NUMBER@`, `@RAW@`, `@NAME@`, `@ZONE@`), handler function signatures, and complete examples covering harvesting, combat, group/tell, and zone/quest triggers -- see [15_Advanced_Scripting_Patterns.md](15_Advanced_Scripting_Patterns.md#trigger-system-for-chat-parsing). For a working `@*@` wildcard example with a timer-gated `ExecuteQueued` pump, see [07_Advanced_Patterns_And_Examples.md](07_Advanced_Patterns_And_Examples.md#trigger-based-automation).

### Window / Game-Event Pattern

> **PLANNED — NOT YET IMPLEMENTED.** ISXPantheon does not yet fire game events (window-appeared, spawn/despawn, incoming chat, zoning). The event-driven shape below is the pattern you will use once events are surfaced; it is not runnable today. The general LavishScript event mechanics (`Event[...]:AttachAtom`, `atom` handlers) are real and can be used with platform/script events now.

```lavishscript
; PLANNED shape - attach to a game event, validate, react, then clean up
Event[SomeGameEvent]:AttachAtom[OnSomeEvent]

atom OnSomeEvent(uint ID)
{
    ; Wait for associated state to settle
    wait 5

    ; Validate the subject still exists before acting
    if !${GameObject[${ID}](exists)}
        return

    ; ... react to the event ...
}
```

### State Change Pattern

**Track the previous value and only react on a transition,** not on every poll. This shape is generic; here it watches a custom variable:

```lavishscript
; Track previous state
declare PreviousValue int script 100

; Monitor for changes
function Pulse()
{
    variable int CurrentValue = ${ISXPantheon.GetCustomVariable[WatchedValue,int]}

    if ${CurrentValue} != ${PreviousValue}
    {
        ; Value changed - react only on the downward crossing
        if ${CurrentValue} < 30 && ${PreviousValue} >= 30
        {
            echo "Crossed the low threshold!"
            call OnThresholdCrossed
        }

        PreviousValue:Set[${CurrentValue}]
    }
}
```

> **PLANNED — NOT YET IMPLEMENTED.** In a combat script the watched value would be player health; that member is planned. The transition-detection structure is what matters and works today against any pollable value.

---

## Configuration Management

### Settings Loading Pattern

```lavishscript
function Module_Declaration()
{
    ; Initialize configuration sets
    ConfigSet:AddSet[Profile]
    ConfigSet:AddSet[Extras]

    ; Load setting values with defaults
    AoEMode:Set[${ConfigSet.FindSet[Profile].FindSetting[Cast AoE Spells,FALSE]}]
    AutoFollowMode:Set[${ConfigSet.FindSet[Extras].FindSetting[Auto Follow Mode,FALSE]}]
    MaxRange:Set[${ConfigSet.FindSet[Profile].FindSetting[Max Range,30]}]

    ; Load UI elements
    ui -load -parent "Main@MyBot Tabs@MyBot" -skin Default "${PATH_UI}/Profile.xml"
}
```

### XML Import Pattern

```lavishscript
; Load an entry list from XML
LavishSettings[MyBot]:AddSet[Entries]
LavishSettings[MyBot].FindSet[Entries]:Import[${PATH_DATA}/Profile.xml]

; Get iterator for the entries
LavishSettings[MyBot].FindSet[Entries].FindSet[Profile]:GetSettingIterator[EntryIterator]

; Populate collection from XML
variable collection:int SelectedEntries

if ${EntryIterator:First(exists)}
{
    do
    {
        variable string tempnme = "${EntryIterator.Key}"
        variable int iLevel = ${tempnme.Token[1, ]}
        variable int iType = ${tempnme.Token[2, ]}
        variable string EntryName = "${EntryIterator.Value}"

        switch ${iType}
        {
            case 92
            case 352
                SelectedEntries:Set[${EntryName},${iLevel}]
                break
        }
    }
    while ${EntryIterator:Next(exists)}
}
```

### UI Binding Pattern

```lavishscript
; Get selected value from dropdown
variable string SelectedItem
SelectedItem:Set[${UIElement[cbSelection@Main@MyBot Tabs@MyBot].SelectedItem.Text}]

; Check checkbox state
if ${UIElement[cbEnableDebug@Main@MyBot Tabs@MyBot].Checked}
{
    DebugMode:Set[TRUE]
}
```

---

## Common Anti-Patterns

### Anti-Pattern 1: No NULL Checks

```lavishscript
; WRONG - Will error if the object doesn't exist
echo ${GameObject["Subject"].Name}
GameObject["Subject"]:Activate

; RIGHT - Check first
if ${GameObject["Subject"](exists)}
{
    echo ${GameObject["Subject"].Name}
    GameObject["Subject"]:Activate
}
```

### Anti-Pattern 2: Reading Data Before It Is Ready

```lavishscript
; WRONG - reading extension members before the extension has loaded
echo ${ISXPantheon.Version}

; RIGHT - wait for the readiness gate first
while !${ISXPantheon.IsReady}
    waitframe

echo ${ISXPantheon.Version}
```

For HTTP, do not poll for a result at all — `GetURL` / `PostURL` deliver their response through the `isxGames_onHTTPResponse` event. Attach an atom and handle the result there (see [04_Core_Concepts.md - Asynchronous Data Loading](04_Core_Concepts.md#asynchronous-data-loading)).

### Anti-Pattern 3: Tight Loops Without Wait

```lavishscript
; WRONG - Consumes 100% CPU
while ${Running}
{
    ; Continuous checking with no delay
    call PollState
}

; RIGHT - Add wait to prevent CPU spike
while ${Running}
{
    call PollState
    wait 10  ; Prevent CPU overload
}
```

### Anti-Pattern 4: Repeated Complex Lookups

```lavishscript
; WRONG - repeats the same deep lookup
if ${SomeObject.Detail.Rating} > 500
{
    echo ${SomeObject.Detail.Rating}
    echo ${SomeObject.Detail.Delay}
}

; RIGHT - cache the reference, then read members once each
variable string Cached
Cached:Set[${SomeObject.Detail.Rating}]

if ${Cached} > 500
{
    echo ${Cached}
    echo ${SomeObject.Detail.Delay}
}
```

### Anti-Pattern 5: Magic Numbers

```lavishscript
; WRONG - Unclear what 50 means
if ${WatchedValue} < 50
    call OnLow

; RIGHT - Use named constants
declare LOW_THRESHOLD int script 50

if ${WatchedValue} < ${LOW_THRESHOLD}
    call OnLow
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
    ; Wait for ISXPantheon
    while !${ISXPantheon.IsReady}
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
    if !${ISXPantheon.IsReady}
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
    call CheckResources
}

; ============================================
; ===  Combat System                      ===
; ============================================

function UpdateCombatState()
{
    ; PLANNED: read live combat state once the player object is surfaced.
    ; For now, drive this from a custom variable so the template runs today.
    InCombat:Set[${ISXPantheon.GetCustomVariable[InCombat,bool]}]
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
; ===  Resource Management                ===
; ============================================

function CheckResources()
{
    ; PLANNED: read live player health/power here once those members exist.
    variable int Watched = ${ISXPantheon.GetCustomVariable[WatchedValue,int]}

    if ${Watched} < ${LOW_HEALTH_THRESHOLD}
    {
        call OnLowResource
    }
}

; ============================================
; ===  Event Handlers                     ===
; ============================================

function RegisterEvents()
{
    ; PLANNED - NOT YET IMPLEMENTED: ISXPantheon does not yet fire game events
    ; (chat, zoning, spawn/despawn). Attach handlers here once they are surfaced.
    ; Platform/script events (e.g. via LavishScript) can be attached now.
}

atom OnNotice(string Message)
{
    ; Handle an incoming notice (placeholder for a future game event)
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
    ; Detach any handlers attached in RegisterEvents (none wired today - see PLANNED note there)

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
