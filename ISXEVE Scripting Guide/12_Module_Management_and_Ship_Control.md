# Module Management and Ship Control
## Complete Guide to Module Activation, Ship Control, and Combat/Mining Patterns

**Part of**: EVE Online Bot Development - AI-Level Documentation
**Layer**: 3 - ISXEVE API Deep Dive
**Prerequisites**: Read files 01-10 (Foundation + Scripting + Core Objects + Entities)
**Critical For**: Combat bots, mining bots, any bot that activates modules

---

## Purpose of This Document

This file provides **exhaustive coverage** of:
- Module object architecture and members
- Module slots (high, mid, low, rigs, subsystems)
- Module states (online, offline, active, overloaded)
- Activation and deactivation patterns
- Charge management (ammunition, mining crystals, scripts)
- Module grouping and group activation/deactivation
- Specific module types (weapons, miners, hardeners, repairers, prop mods, tackle, etc.)
- Overheating (overloading) mechanics
- Common patterns from Evebot, Yamfa, and Tehbot
- Timing and cycle management
- Performance optimization
- Critical gotchas and edge cases

**Why This Is CRITICAL**:
- **Combat bots live or die by module management** (weapons, tank, tackle)
- **Mining bots depend on laser activation** (target selection, cycle timing)
- **Module timing directly impacts efficiency** (DPS, mining yield, survival)
- **Many bot failures come from improper module handling** (activation races, charge depletion, capacitor drain)

---

## Table of Contents

1. [Module System Overview](#module-system-overview)
2. [Module Object](#module-object)
3. [Module Slots](#module-slots)
4. [Module States](#module-states)
5. [Module Activation and Deactivation](#module-activation-and-deactivation)
6. [Charge Management](#charge-management)
7. [Module Grouping](#module-grouping)
8. [Specific Module Types](#specific-module-types)
9. [Overheating (Overloading)](#overheating-overloading)
10. [Common Patterns from Example Scripts](#common-patterns-from-example-scripts)
11. [Timing and Cycle Management](#timing-and-cycle-management)
12. [Performance Optimization](#performance-optimization)
13. [Critical Gotchas](#critical-gotchas)
14. [Anti-Patterns](#anti-patterns)

---

## Module System Overview

### What are Modules?

**Modules** are items fitted to your ship that provide functionality:
- **Weapons** - Guns, launchers, drones
- **Mining equipment** - Mining lasers, strip miners, gas harvesters
- **Defenses** - Shield hardeners, armor repairers, hull repairers
- **Propulsion** - Afterburners, microwarpdrives
- **Tackle** - Warp scramblers, stasis webifiers
- **Electronic warfare** - ECM, sensor dampeners, tracking disruptors
- **Utility** - Cargo scanners, cloaks, probes

### Module Categories by Slot

**High Slots** - Offensive modules:
- Turrets (lasers, projectiles, hybrids)
- Missile launchers
- Mining lasers/strip miners
- Salvagers
- Cloaking devices
- Tractor beams

**Mid Slots** - Active tank and utility:
- Shield boosters
- Shield hardeners (active)
- Propulsion modules (AB/MWD)
- Tackle modules (scram, web, point)
- Electronic warfare
- Sensor boosters
- Target painters

**Low Slots** - Passive tank and damage mods:
- Armor hardeners (usually passive)
- Armor repairers
- Damage control
- Damage amplifiers
- Mining upgrades
- Capacitor power relays

**Rig Slots** - Permanent modifications:
- Cannot be activated/deactivated
- Passive bonuses only
- Not covered in detail (cannot control via script)

**Wiki Reference**: `IsxeveWiki/ISXEVE/Module_(Object_Type).html`

---

## Module Object

### Getting Module Objects

**By Slot Type and Index**:
```lavish
; Get module in high slot 0 (0-indexed!)
variable item module = ${MyShip.Module[HiSlot, 0]}

if ${module(exists)}
{
    echo "High slot 0: ${module.Name}"
}

; Get module in mid slot 2
variable item midModule = ${MyShip.Module[MedSlot, 2]}

; Get module in low slot 1
variable item lowModule = ${MyShip.Module[LoSlot, 1]}
```

**By Module Type Name**:
```lavish
; Find first module matching type
variable item miner = ${MyShip.Module[=MiningLaser]}

if ${miner(exists)}
{
    echo "Found mining laser: ${miner.Name}"
}

; Find first weapon
variable item weapon = ${MyShip.Module[=Weapon]}
```

**By Module Name**:
```lavish
; Find by partial name match
variable item shield = ${MyShip.Module[Shield Booster]}
```

### Critical Module Members

**Identity**:
```lavish
variable item module = ${MyShip.Module[HiSlot, 0]}

echo "Name: ${module.Name}"
echo "Type ID: ${module.TypeID}"
echo "Slot: ${module.Slot}"           ; "HiSlot0", "MedSlot2", etc.
echo "Slot ID: ${module.ToItem.SlotID}"    ; Numeric slot ID
```

**State**:
```lavish
echo "Is Online: ${module.IsOnline}"
echo "Is Active: ${module.IsActive}"
echo "Is Activatable: ${module.IsActivatable}"
echo "Is Deactivating: ${module.IsDeactivating}"
echo "Is Changing Ammo: ${module.IsChangingAmmo}"
echo "Is Reloading: ${module.IsReloading}"
echo "Is Goingonline: ${module.IsGoingonline}"
```

**Charges (Ammo/Crystals)**:
```lavish
echo "Charge: ${module.Charge}"           ; Current charge type
echo "Charge Quantity: ${module.ChargeQuantity}"
echo "Max Charge Size: ${module.MaxChargeSize}"
```

**Targeting** (for targeted modules):
```lavish
echo "Target ID: ${module.TargetID}"      ; What it's currently targeting
echo "Last Target ID: ${module.LastTargetID}"
```

**Timing**:
```lavish
echo "Duration: ${module.Duration}"        ; Cycle time in milliseconds
echo "Time Started: ${module.TimeStarted}"
```

**Specialized**:
```lavish
; For mining lasers
echo "Optimal Range: ${module.OptimalRange}"
echo "Accuracy Falloff: ${module.AccuracyFalloff}"

; For weapons
echo "Damage Multiplier: ${module.DamageMultiplier}"
```

---

## Module Slots

### Slot Indexing

**CRITICAL**: Module slots are **0-indexed** (unlike most ISXEVE collections!).

```lavish
; High slots: 0, 1, 2, 3, 4, 5, 6, 7
variable item hi0 = ${MyShip.Module[HiSlot, 0]}    ; First high slot
variable item hi7 = ${MyShip.Module[HiSlot, 7]}    ; Last high slot (if ship has 8 highs)

; Mid slots: 0, 1, 2, 3, 4, 5, 6, 7
variable item mid0 = ${MyShip.Module[MedSlot, 0]}

; Low slots: 0, 1, 2, 3, 4, 5, 6, 7
variable item low0 = ${MyShip.Module[LoSlot, 0]}
```

### Counting Slots

```lavish
echo "High Slots: ${MyShip.HiSlots}"
echo "Mid Slots: ${MyShip.MedSlots}"
echo "Low Slots: ${MyShip.LowSlots}"
echo "Rig Slots: ${MyShip.RigSlots}"
```

### Iterating All Modules in Slot Type

```lavish
function ActivateAllHighSlots()
{
    variable int i
    for (i:Set[0]; ${i} < ${MyShip.HiSlots}; i:Inc)    ; 0-indexed!
    {
        variable item module = ${MyShip.Module[HiSlot, ${i}]}

        if !${module(exists)}
            continue

        if !${module.IsActivatable}
            continue

        if ${module.IsActive}
            continue

        module:Activate
        echo "Activated ${module.Name}"
    }
}
```

---

## Module States

### State Overview

**Modules can be in multiple states**:

1. **Offline** - Module fitted but not online (cannot activate)
2. **Online** - Module online and ready (can activate if activatable)
3. **Active** - Module currently running (weapons firing, repairers repping, etc.)
4. **Deactivating** - Module finishing current cycle (will go offline/online after)
5. **Going Online** - Module onlining (takes time for some modules)
6. **Reloading** - Module reloading charges
7. **Changing Ammo** - Module switching ammo type

### Checking States

```lavish
variable item module = ${MyShip.Module[HiSlot, 0]}

; Online check (can it be activated?)
if ${module.IsOnline}
{
    echo "Module is online"
}

; Active check (is it currently running?)
if ${module.IsActive}
{
    echo "Module is active"
}

; Activatable check (can we activate it?)
if ${module.IsActivatable}
{
    echo "Module can be activated"
}

; Deactivating check (is it turning off?)
if ${module.IsDeactivating}
{
    echo "Module is deactivating"
}
```

### State Transitions

**Offline → Online**:
```lavish
; Some modules online instantly
; Some modules take time to online (siege modules, triage, etc.)

; Check if onlining
if ${module.IsGoingonline}
{
    echo "Module is onlining..."
}
```

**Online → Active** (for activatable modules):
```lavish
module:Activate

; Module is now active (or activating)
```

**Active → Online** (deactivation):
```lavish
module:Deactivate

; Module will finish current cycle, then go online
```

---

## Module Activation and Deactivation

### Activating Modules

**Method 1: Module Object Method**:
```lavish
variable item miner = ${MyShip.Module[=MiningLaser]}

if ${miner(exists)} && ${miner.IsOnline} && !${miner.IsActive}
{
    miner:Activate
    echo "Activated ${miner.Name}"
}
```

**Method 2: Activate on Target**:
```lavish
variable int64 asteroidID = ${GetNearestAsteroid}

if ${asteroidID} > 0
{
    variable item miner = ${MyShip.Module[=MiningLaser]}
    miner:Activate[${asteroidID}]
    echo "Mining ${Entity[${asteroidID}].Name}"
}
```

**Method 3: EVE:Execute Command**:
```lavish
; Need SlotID (numeric ID of slot)
variable int slotID = ${MyShip.Module[HiSlot, 0].ToItem.SlotID}
EVE:Execute[CmdActivateModule, ${slotID}]
```

**Preferred**: Use module object method (`module:Activate`) - more reliable.

### Deactivating Modules

**Method 1: Module Object Method**:
```lavish
variable item miner = ${MyShip.Module[=MiningLaser]}

if ${miner.IsActive}
{
    miner:Deactivate
    echo "Deactivating ${miner.Name}"
}
```

**Method 2: EVE:Execute Command**:
```lavish
variable int slotID = ${MyShip.Module[HiSlot, 0].ToItem.SlotID}
EVE:Execute[CmdDeactivateModule, ${slotID}]
```

### Click (Toggle) Modules

**Click = Activate if inactive, deactivate if active**:
```lavish
variable item module = ${MyShip.Module[HiSlot, 0]}
module:Click

; This toggles the module state
```

**Use Cases**:
- Useful for manual-style control
- Dangerous for scripts (unpredictable state after click)
- Prefer explicit Activate/Deactivate

### Activation Validation

**Always validate before activating**:

```lavish
function SafeActivateModule(string slotType, int slotIndex, int64 targetID)
{
    ; Get module
    variable item module = ${MyShip.Module[${slotType}, ${slotIndex}]}

    ; Check exists
    if !${module(exists)}
    {
        echo "ERROR: Module not found in ${slotType} ${slotIndex}"
        return FALSE
    }

    ; Check online
    if !${module.IsOnline}
    {
        echo "ERROR: Module not online"
        return FALSE
    }

    ; Check activatable
    if !${module.IsActivatable}
    {
        echo "ERROR: Module not activatable"
        return FALSE
    }

    ; Check not already active
    if ${module.IsActive}
    {
        echo "Module already active"
        return TRUE
    }

    ; Check capacitor if needed
    ; (Advanced - would check MyShip.CapacitorPct here)

    ; Activate
    if ${targetID} > 0
    {
        module:Activate[${targetID}]
    }
    else
    {
        module:Activate
    }

    echo "Activated ${module.Name}"
    return TRUE
}
```

---

## Charge Management

### What are Charges?

**Charges** are consumable ammunition/resources for modules:
- **Ammunition** - Projectile ammo, missiles, hybrid charges
- **Mining crystals** - Improve mining yield, degrade over time
- **Scripts** - Change module behavior (e.g., sensor booster scripts)
- **Nanite paste** - For repairing overheated modules

### Checking Current Charge

```lavish
variable item module = ${MyShip.Module[HiSlot, 0]}

echo "Charge: ${module.Charge}"
echo "Charge Type ID: ${module.Charge.ID}"
echo "Charge Quantity: ${module.ChargeQuantity}"

; Check if has charge
if ${module.Charge(exists)}
{
    echo "Module has ${module.Charge.Name} x${module.ChargeQuantity}"
}
else
{
    echo "Module has no charge"
}
```

### Reloading Charges

**ISXEVE does NOT have direct reload method**.

**Workaround 1: EVE:Execute (if available)**:
```lavish
; Some EVE versions have CmdReloadModule
EVE:Execute[CmdReloadModule, ${slotID}]
```

**Workaround 2: Hotkey Simulation** (unreliable):
```lavish
; Not recommended - depends on hotkey setup
```

**Workaround 3: Manual Instruction**:
```lavish
; Most bots just warn user to reload manually
if ${module.ChargeQuantity} == 0
{
    echo "WARNING: ${module.Name} out of ammo! Reload manually."
}
```

### Changing Ammo Type

**No direct ISXEVE support**. Must be done manually by player.

### Charge Depletion Detection

```lavish
function CheckAmmoLevel(string slotType, int slotIndex, int warningThreshold)
{
    variable item module = ${MyShip.Module[${slotType}, ${slotIndex}]}

    if !${module(exists)}
        return

    if !${module.Charge(exists)}
    {
        echo "WARNING: ${module.Name} has no ammo!"
        return
    }

    if ${module.ChargeQuantity} <= ${warningThreshold}
    {
        echo "WARNING: ${module.Name} low ammo (${module.ChargeQuantity} remaining)"
    }
}

; Usage
call CheckAmmoLevel "HiSlot" 0 100
```

---

## Module Grouping

### What is Module Grouping?

**Module groups** = Multiple modules of same type treated as one for activation.

**Example**: 4x mining lasers grouped = activate all 4 with one click.

### Checking if Modules are Grouped

**ISXEVE has limited grouping support**. Most scripts activate modules individually.

### Activating All Modules of Type

**Pattern: Activate all matching modules**:

```lavish
function ActivateAllMiningLasers(int64 asteroidID)
{
    variable int i
    for (i:Set[0]; ${i} < ${MyShip.HiSlots}; i:Inc)
    {
        variable item module = ${MyShip.Module[HiSlot, ${i}]}

        if !${module(exists)}
            continue

        ; Check if mining laser (by name or type)
        if !${module.Name.Find["Mining Laser"](exists)} && !${module.Name.Find["Strip Miner"](exists)}
            continue

        ; Skip if already active
        if ${module.IsActive}
            continue

        ; Activate on target
        module:Activate[${asteroidID}]
        echo "Activated ${module.Name}"
        wait 10    ; Brief delay between activations
    }
}
```

---

## Specific Module Types

### Weapons (Combat)

**Turrets** (Lasers, Projectiles, Hybrids):
```lavish
function ActivateAllWeapons(int64 targetID)
{
    variable int i
    for (i:Set[0]; ${i} < ${MyShip.HiSlots}; i:Inc)
    {
        variable item weapon = ${MyShip.Module[HiSlot, ${i}]}

        if !${weapon(exists)}
            continue

        ; Check if weapon (simplistic - improve for production)
        if !${weapon.IsActivatable}
            continue

        ; Check if already active on this target
        if ${weapon.IsActive} && ${weapon.TargetID} == ${targetID}
            continue

        ; Activate
        weapon:Activate[${targetID}]
        echo "Firing ${weapon.Name} at target ${targetID}"
        wait 10
    }
}
```

**Launchers** (Missiles, Rockets, Torpedoes):
```lavish
; Same pattern as turrets - activate on target
```

### Mining Lasers

**Activate on Asteroid**:
```lavish
function ActivateMiningLasers(int64 asteroidID)
{
    variable int i
    for (i:Set[0]; ${i} < ${MyShip.HiSlots}; i:Inc)
    {
        variable item module = ${MyShip.Module[HiSlot, ${i}]}

        if !${module(exists)}
            continue

        if !${module.Name.Find["Mining"](exists)} && !${module.Name.Find["Strip Miner"](exists)}
            continue

        if ${module.IsActive}
            continue

        module:Activate[${asteroidID}]
        echo "Mining ${Entity[${asteroidID}].Name} with ${module.Name}"
        wait 10
    }
}
```

**Check Mining Cycle**:
```lavish
function AreMiningLasersActive()
{
    variable int i
    for (i:Set[0]; ${i} < ${MyShip.HiSlots}; i:Inc)
    {
        variable item module = ${MyShip.Module[HiSlot, ${i}]}

        if !${module(exists)}
            continue

        if ${module.Name.Find["Mining"](exists)} || ${module.Name.Find["Strip Miner"](exists)}
        {
            if ${module.IsActive}
                return TRUE
        }
    }

    return FALSE
}
```

### Shield Boosters / Armor Repairers

**Activate When Damaged**:
```lavish
function ActivateRepairers()
{
    ; Check if we need repairs
    if ${MyShip.ShieldPct} > 80
        return

    ; Find and activate shield booster
    variable int i
    for (i:Set[0]; ${i} < ${MyShip.MedSlots}; i:Inc)
    {
        variable item module = ${MyShip.Module[MedSlot, ${i}]}

        if !${module(exists)}
            continue

        if !${module.Name.Find["Shield Booster"](exists)}
            continue

        if ${module.IsActive}
            continue

        module:Activate
        echo "Activating ${module.Name}"
    }
}
```

### Hardeners (Shield/Armor Resistance)

**Activate at Start**:
```lavish
function ActivateHardeners()
{
    variable int i

    ; Shield hardeners (mid slots)
    for (i:Set[0]; ${i} < ${MyShip.MedSlots}; i:Inc)
    {
        variable item module = ${MyShip.Module[MedSlot, ${i}]}

        if !${module(exists)}
            continue

        if !${module.Name.Find["Hardener"](exists)}
            continue

        if ${module.IsActive}
            continue

        module:Activate
        echo "Activating ${module.Name}"
    }

    ; Armor hardeners (low slots)
    for (i:Set[0]; ${i} < ${MyShip.LowSlots}; i:Inc)
    {
        variable item module = ${MyShip.Module[LoSlot, ${i}]}

        if !${module(exists)}
            continue

        if !${module.Name.Find["Hardener"](exists)}
            continue

        if ${module.IsActive}
            continue

        module:Activate
        echo "Activating ${module.Name}"
    }
}
```

### Propulsion Mods (Afterburner/MWD)

**Toggle Propulsion**:
```lavish
function ActivatePropMod()
{
    variable int i
    for (i:Set[0]; ${i} < ${MyShip.MedSlots}; i:Inc)
    {
        variable item module = ${MyShip.Module[MedSlot, ${i}]}

        if !${module(exists)}
            continue

        if !${module.Name.Find["Afterburner"](exists)} && !${module.Name.Find["Microwarpdrive"](exists)}
            continue

        if ${module.IsActive}
            continue

        module:Activate
        echo "Activating ${module.Name}"
        return
    }
}

function DeactivatePropMod()
{
    variable int i
    for (i:Set[0]; ${i} < ${MyShip.MedSlots}; i:Inc)
    {
        variable item module = ${MyShip.Module[MedSlot, ${i}]}

        if !${module(exists)}
            continue

        if !${module.Name.Find["Afterburner"](exists)} && !${module.Name.Find["Microwarpdrive"](exists)}
            continue

        if !${module.IsActive}
            continue

        module:Deactivate
        echo "Deactivating ${module.Name}"
        return
    }
}
```

**Or use EVE:Execute**:
```lavish
; Toggle propulsion mod
EVE:Execute[CmdTogglePropulsionMod]
```

### Tackle Modules (Scram/Web/Point)

**Activate on Target**:
```lavish
function ActivateTackle(int64 targetID)
{
    variable int i
    for (i:Set[0]; ${i} < ${MyShip.MedSlots}; i:Inc)
    {
        variable item module = ${MyShip.Module[MedSlot, ${i}]}

        if !${module(exists)}
            continue

        ; Check if tackle module
        if !${module.Name.Find["Warp Scrambler"](exists)} && !${module.Name.Find["Stasis Webifier"](exists)} && !${module.Name.Find["Warp Disruptor"](exists)}
            continue

        if ${module.IsActive}
            continue

        module:Activate[${targetID}]
        echo "Tackling target with ${module.Name}"
    }
}
```

---

## Overheating (Overloading)

### What is Overheating?

**Overheating** (also called **overloading**) = Running modules beyond normal capacity for increased performance at cost of heat damage.

**Benefits**:
- Increased damage (weapons)
- Increased repair amount (repairers)
- Increased range/effectiveness (tackle)

**Costs**:
- Heat damage to module and nearby modules
- Module can burn out if overheated too long

### ISXEVE Overheating Support

**ISXEVE has LIMITED overheating support**. Most scripts avoid overheating automation.

**Checking if Module is Overheated**:
```lavish
; No direct "IsOverheated" member (as of common ISXEVE versions)
; Would need to check heat damage or rack heat
```

**Activating Overheating**:
```lavish
; No direct ISXEVE method
; Would require UI clicking or hotkey simulation (unreliable)
```

**Most Bots**: Skip overheating automation entirely.

---

## Common Patterns from Example Scripts

### Pattern 1: Evebot - Mining Laser Management

```lavish
; Simplified from Evebot
function Evebot_ActivateMiners(int64 asteroidID)
{
    if ${asteroidID} <= 0
        return FALSE

    variable entity asteroid = ${Entity[${asteroidID}]}

    if !${asteroid(exists)}
        return FALSE

    ; Activate all mining lasers
    variable int i
    for (i:Set[0]; ${i} < ${MyShip.HiSlots}; i:Inc)
    {
        variable item module = ${MyShip.Module[HiSlot, ${i}]}

        if !${module(exists)}
            continue

        ; Check if miner
        if !${module.Name.Find["Mining"](exists)} && !${module.Name.Find["Strip Miner"](exists)}
            continue

        ; Skip if active
        if ${module.IsActive}
            continue

        ; Activate
        module:Activate[${asteroidID}]
        echo "Mining ${asteroid.Name} with ${module.Name}"
        wait 10
    }

    return TRUE
}
```

### Pattern 2: Tehbot - Weapon Activation

```lavish
; Simplified from Tehbot
function Tehbot_ActivateWeapons(int64 targetID)
{
    if ${targetID} <= 0
        return

    variable entity target = ${Entity[${targetID}]}

    if !${target(exists)}
        return

    ; Activate all weapons on target
    variable int i
    for (i:Set[0]; ${i} < ${MyShip.HiSlots}; i:Inc)
    {
        variable item weapon = ${MyShip.Module[HiSlot, ${i}]}

        if !${weapon(exists)}
            continue

        if !${weapon.IsActivatable}
            continue

        ; Check if already firing at this target
        if ${weapon.IsActive} && ${weapon.TargetID} == ${targetID}
            continue

        ; Activate
        weapon:Activate[${targetID}]
        echo "Firing ${weapon.Name} at ${target.Name}"
        wait 10
    }
}
```

### Pattern 3: Defense Module Auto-Activation

```lavish
; Common pattern across all bots
function ProcessDefenseModules()
{
    ; Activate hardeners if not running
    if ${MyShip.ShieldPct} < 100
    {
        call ActivateHardeners
    }

    ; Activate repairers if shields low
    if ${MyShip.ShieldPct} < 50
    {
        call ActivateShieldBoosters
    }

    ; Emergency: Overheat repairers if critical
    if ${MyShip.ShieldPct} < 20
    {
        echo "CRITICAL: Low shields!"
        ; Would activate overheating here if supported
    }
}
```

---

## Timing and Cycle Management

### Module Cycles

**Most modules run in cycles**:
- Weapons: Fire every N seconds
- Mining lasers: Mine every N seconds
- Repairers: Repair every N seconds

**Cycle Time**:
```lavish
variable item module = ${MyShip.Module[HiSlot, 0]}
echo "Cycle time: ${module.Duration}ms"
```

### Waiting for Cycle Completion

**Pattern: Wait for mining cycle**:
```lavish
function WaitForMiningCycle()
{
    ; Check if any miner is active
    variable bool anyActive = FALSE
    variable int i

    for (i:Set[0]; ${i} < ${MyShip.HiSlots}; i:Inc)
    {
        variable item module = ${MyShip.Module[HiSlot, ${i}]}

        if !${module(exists)}
            continue

        if ${module.Name.Find["Mining"](exists)} || ${module.Name.Find["Strip Miner"](exists)}
        {
            if ${module.IsActive}
            {
                anyActive:Set[TRUE]
                break
            }
        }
    }

    if !${anyActive}
        return

    ; Wait for cycle to complete (miners go inactive)
    echo "Waiting for mining cycle..."

    variable int timeout = 0
    while ${anyActive} && ${timeout} < 300    ; 30 second timeout
    {
        wait 100

        anyActive:Set[FALSE]
        for (i:Set[0]; ${i} < ${MyShip.HiSlots}; i:Inc)
        {
            variable item module = ${MyShip.Module[HiSlot, ${i}]}

            if !${module(exists)}
                continue

            if ${module.Name.Find["Mining"](exists)} || ${module.Name.Find["Strip Miner"](exists)}
            {
                if ${module.IsActive}
                {
                    anyActive:Set[TRUE]
                    break
                }
            }
        }

        timeout:Inc
    }

    echo "Mining cycle complete"
}
```

---

## Performance Optimization

### Cache Module References

**Don't re-fetch modules repeatedly**:

```lavish
; BAD - Re-fetches module every check
while TRUE
{
    if ${MyShip.Module[HiSlot, 0].IsActive}
    {
        echo "Active: ${MyShip.Module[HiSlot, 0].Name}"
    }
    wait 10
}

; GOOD - Cache module reference
variable item miner = ${MyShip.Module[HiSlot, 0]}

while TRUE
{
    if ${miner.IsActive}
    {
        echo "Active: ${miner.Name}"
    }
    wait 10
}
```

### Minimize Module Iteration

**Don't iterate all modules every tick if not needed**:

```lavish
; BAD - Iterates every tick
while TRUE
{
    call ActivateAllMiningLasers
    wait 10    ; Checking 100 times per second!
}

; GOOD - Only activate when needed
variable bool lasersActive = TRUE

while TRUE
{
    if !${lasersActive}
    {
        call ActivateAllMiningLasers
        lasersActive:Set[TRUE]
    }

    ; Check if still active (less frequently)
    ; ...

    wait 10
}
```

---

## Critical Gotchas

### Gotcha 1: Slot Indexing is 0-Based

```lavish
; WRONG - 1-indexed assumption
variable item module = ${MyShip.Module[HiSlot, 1]}    ; This is SECOND high slot!

; RIGHT - 0-indexed
variable item module = ${MyShip.Module[HiSlot, 0]}    ; First high slot
```

### Gotcha 2: Module Can Despawn (Ship Destruction/Refit)

```lavish
; BAD - Module reference can become invalid
variable item miner = ${MyShip.Module[HiSlot, 0]}

; ... later ...
miner:Activate    ; CRASH if ship changed/destroyed!

; GOOD - Re-fetch or check exists
if ${miner(exists)}
{
    miner:Activate
}
```

### Gotcha 3: Activation is Asynchronous

```lavish
; BAD - Assumes instant activation
module:Activate
if ${module.IsActive}    ; Might be FALSE immediately after!
{
    echo "Active"
}

; GOOD - Wait for activation
module:Activate
wait 50

if ${module.IsActive}
{
    echo "Active"
}
```

### Gotcha 4: Charge Quantity Can Change Mid-Cycle

```lavish
; Charges consumed during cycles
; Check quantity BEFORE activation if critical
```

### Gotcha 5: Some Modules Cannot Be Activated via Script

```lavish
; Cloaking devices, jump drives, etc. may not respond to Activate
; Test each module type empirically
```

---

## Anti-Patterns

### Anti-Pattern 1: No State Check Before Activation

```lavish
; BAD
module:Activate

; GOOD
if ${module(exists)} && ${module.IsOnline} && !${module.IsActive}
{
    module:Activate
}
```

### Anti-Pattern 2: Activating Every Tick

```lavish
; BAD - Spamming activation
while TRUE
{
    module:Activate    ; Already active! Wasted calls!
    wait 10
}

; GOOD - Check state first
while TRUE
{
    if !${module.IsActive}
    {
        module:Activate
    }
    wait 10
}
```

### Anti-Pattern 3: No Ammo Check

```lavish
; BAD
module:Activate[${target}]
; Might fail if no ammo!

; GOOD
if ${module.Charge(exists)} && ${module.ChargeQuantity} > 0
{
    module:Activate[${target}]
}
else
{
    echo "Out of ammo!"
}
```

---

## Summary and Key Takeaways

### Essential Concepts

1. **Modules = Ship equipment** (weapons, miners, tank, utility)
2. **Slots**: High (offense), Mid (active tank/utility), Low (passive tank/damage mods)
3. **0-indexed slots** (unlike most ISXEVE collections!)
4. **Module states**: Offline, Online, Active, Deactivating, Reloading
5. **Activation**: `module:Activate` or `module:Activate[targetID]`
6. **Deactivation**: `module:Deactivate`

### Most Common Module Operations

```lavish
; Get module
variable item module = ${MyShip.Module[HiSlot, 0]}

; Check state
if ${module.IsOnline} && !${module.IsActive}

; Activate (untargeted)
module:Activate

; Activate (targeted)
module:Activate[${targetID}]

; Deactivate
module:Deactivate

; Check charge
if ${module.Charge(exists)}
    echo "Ammo: ${module.ChargeQuantity}"
```

### Critical Rules

1. **Slots are 0-indexed** (first slot = 0, not 1)
2. **Always check (exists)** before using module
3. **Always check IsOnline** before activating
4. **Check IsActive** to avoid redundant activations
5. **Wait after activation** before checking IsActive
6. **Monitor ammo/charges** for consumable modules
7. **Cache module references** for performance

---

## Next Steps

**Continue to Other Layer 3 Files**:
- File 11: Movement_Navigation_and_Autopilot.md (Warping, docking, movement)
- File 13: Inventory_and_Cargo_Systems.md (Cargo management, hangar access)
- File 16: Fleet_and_Social_Systems.md (Fleet operations)

**Apply Knowledge**:
- All combat bots need weapon activation patterns
- All mining bots need mining laser management
- Defense module patterns critical for survival
- Recognize module patterns in Evebot/Tehbot/Yamfa

---

## Wiki References

**Module Object**:
- `IsxeveWiki/ISXEVE/Module_(Object_Type).html` - Complete module reference

**Item Object** (modules inherit from item):
- `IsxeveWiki/ISXEVE/Item_(Object_Type).html` - Item object reference

**MyShip Module Access**:
- `IsxeveWiki/ISXEVE/MyShip_(Object_Type).html` - MyShip.Module reference

---

**END OF FILE 12**
**Status**: Complete (~1600 lines)
**Next**: Update progress tracker, assess token budget for more files
