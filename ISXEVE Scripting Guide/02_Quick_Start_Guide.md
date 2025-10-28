# ISXEVE Quick Start Guide
## Your First ISXEVE Script in 5-10 Minutes

**Target Audience:** Developers who know LavishScript basics and want to start writing ISXEVE scripts quickly.

**Prerequisites:**
- InnerSpace installed
- ISXEVE extension installed
- EVE Online running
- Basic LavishScript knowledge (if not, read [01_LavishScript_Fundamentals.md](01_LavishScript_Fundamentals.md) first)

---

## Table of Contents

1. [Installation Verification](#installation-verification)
2. [Your First Script](#your-first-script)
3. [Understanding the Script](#understanding-the-script)
4. [Running the Script](#running-the-script)
5. [Common First Scripts](#common-first-scripts)
6. [Next Steps](#next-steps)
7. [Troubleshooting](#troubleshooting)

---

## Installation Verification

Before writing scripts, verify ISXEVE is loaded:

### Step 1: Launch InnerSpace and EVE

1. Start InnerSpace
2. Launch EVE Online through InnerSpace
3. Log into a character
4. Open the InnerSpace console (default: Ctrl+I)

### Step 2: Verify ISXEVE Extension

In the InnerSpace console, type:

```lavishscript
echo ${ISXEVE(exists)}
```

**Expected output:** `TRUE`

If you get `FALSE`, ISXEVE is not loaded. Load it with:

```lavishscript
ext load isxeve
```

### Step 3: Test Basic TLOs

Verify you can access game data:

```lavishscript
echo ${Me.Name}
echo ${MyShip.Name}
echo ${Me.InSpace}
```

**Expected output:**
```
CharacterName
ShipName
TRUE (or FALSE if docked)
```

If these work, you're ready to script!

---

## Your First Script

Let's create a simple script that displays character and ship information.

### Step 1: Create Script File

Create a new file: `InnerSpace/Scripts/MyFirstScript.iss`

### Step 2: Write the Script

Copy this code into `MyFirstScript.iss`:

```lavishscript
; MyFirstScript.iss
; Displays character and ship information

function main()
{
    ; Wait for ISXEVE to be ready
    while !${ISXEVE.IsReady}
    {
        echo "Waiting for ISXEVE..."
        wait 10
    }

    echo "====================================="
    echo "CHARACTER INFORMATION"
    echo "====================================="

    ; Character info
    echo "Name: ${Me.Name}"
    echo "Corporation: ${Me.Corp}"

    ; Check if in alliance
    if ${Me.Alliance(exists)}
    {
        echo "Alliance: ${Me.Alliance}"
    }
    else
    {
        echo "Alliance: None"
    }

    ; Wallet
    echo "Wallet: ${Me.Wallet} ISK"

    ; Location
    if ${Me.InSpace}
    {
        echo "Location: In space"
    }
    else
    {
        echo "Location: Docked"
    }

    echo ""
    echo "====================================="
    echo "SHIP INFORMATION"
    echo "====================================="

    ; Ship info
    echo "Ship: ${MyShip.Name}"
    echo "Shield: ${MyShip.ShieldPct}%"
    echo "Armor: ${MyShip.ArmorPct}%"
    echo "Structure: ${MyShip.StructurePct}%"
    echo "Capacitor: ${MyShip.CapacitorPct}%"

    ; Cargo
    echo "Cargo: ${MyShip.UsedCargoCapacity}/${MyShip.CargoCapacity} m3"

    echo ""
    echo "====================================="
    echo "Script complete!"
}
```

### Step 3: Run the Script

In the InnerSpace console:

```lavishscript
run MyFirstScript
```

**Expected output:**
```
=====================================
CHARACTER INFORMATION
=====================================
Name: YourCharName
Corporation: Your Corp
Alliance: Your Alliance
Wallet: 1234567.89 ISK
Location: In space

=====================================
SHIP INFORMATION
=====================================
Ship: Dominix
Shield: 100%
Armor: 100%
Structure: 100%
Capacitor: 95.5%
Cargo: 150/400 m3

=====================================
Script complete!
```

**Congratulations!** You've written your first ISXEVE script!

---

## Understanding the Script

Let's break down the key concepts:

### 1. ISXEVE Ready Check

```lavishscript
while !${ISXEVE.IsReady}
{
    echo "Waiting for ISXEVE..."
    wait 10
}
```

**Why:** ISXEVE loads game data asynchronously. Always wait for `IsReady` before accessing game data.

### 2. Top-Level Objects (TLOs)

```lavishscript
${Me.Name}      ; Character TLO
${MyShip.Name}  ; Ship TLO
```

**What:** TLOs are globally accessible objects provided by ISXEVE.

### 3. Existence Checking

```lavishscript
if ${Me.Alliance(exists)}
{
    echo "Alliance: ${Me.Alliance}"
}
```

**Why:** Not all characters have alliances. Check `(exists)` before accessing optional data.

### 4. Object Members

```lavishscript
${MyShip.ShieldPct}     ; Member (property)
${MyShip.CargoCapacity} ; Member (property)
```

**What:** Members return values from objects (read-only).

---

## Common First Scripts

### Example 2: Shield Monitor

A script that continuously monitors shield percentage:

```lavishscript
; ShieldMonitor.iss
; Monitors shield percentage and warns if low

variable bool Running = TRUE

function main()
{
    ; Wait for ISXEVE
    while !${ISXEVE.IsReady}
    {
        wait 10
    }

    echo "Shield Monitor started"
    echo "Press End Script to stop"

    ; Main loop
    while ${Running}
    {
        variable float ShieldPct = ${MyShip.ShieldPct}

        ; Check shield levels
        if ${ShieldPct} < 25
        {
            echo "WARNING: Shields CRITICAL (${ShieldPct}%)"
        }
        elseif ${ShieldPct} < 50
        {
            echo "CAUTION: Shields LOW (${ShieldPct}%)"
        }

        ; Wait 2 seconds between checks
        wait 20
    }

    echo "Shield Monitor stopped"
}
```

**To stop:** In InnerSpace console, type: `endscript ShieldMonitor`

### Example 3: Target Info

Display information about your current target:

```lavishscript
; TargetInfo.iss
; Displays information about current target

function main()
{
    while !${ISXEVE.IsReady}
    {
        wait 10
    }

    ; Check if we have a target
    if !${MyShip.ActiveTarget(exists)}
    {
        echo "No active target!"
        return
    }

    ; Get target as entity
    variable entity Target = ${MyShip.ActiveTarget.ToEntity}

    echo "====================================="
    echo "TARGET INFORMATION"
    echo "====================================="

    echo "Name: ${Target.Name}"
    echo "Distance: ${Target.Distance} meters"
    echo "Type: ${Target.Type}"
    echo "Group: ${Target.Group}"

    ; Check if NPC or player
    if ${Target.IsNPC}
    {
        echo "Category: NPC"
    }
    elseif ${Target.IsPC}
    {
        echo "Category: Player Character"
    }

    ; Targeting status
    if ${Target.IsLockedTarget}
    {
        echo "Status: LOCKED"
    }
    else
    {
        echo "Status: Not locked"
    }

    echo "====================================="
}
```

**Usage:**
1. Target something in game
2. Run: `run TargetInfo`

### Example 4: Auto-Lock Target

Automatically lock your active target:

```lavishscript
; AutoLock.iss
; Automatically locks current target

function main()
{
    while !${ISXEVE.IsReady}
    {
        wait 10
    }

    ; Check if we have a target
    if !${MyShip.ActiveTarget(exists)}
    {
        echo "No active target!"
        return
    }

    ; Get target entity
    variable entity Target = ${MyShip.ActiveTarget.ToEntity}

    echo "Locking target: ${Target.Name}"

    ; Check if already locked
    if ${Target.IsLockedTarget}
    {
        echo "Target already locked!"
        return
    }

    ; Lock the target
    Target:LockTarget[]

    ; Wait for lock (max 15 seconds)
    variable int counter = 0
    while !${Target.IsLockedTarget} && ${counter} < 150
    {
        wait 10
        counter:Inc
    }

    ; Check result
    if ${Target.IsLockedTarget}
    {
        echo "Target locked successfully!"
    }
    else
    {
        echo "Failed to lock target (timeout)"
    }
}
```

---

<!-- CLAUDE_SKIP_START -->
## Running Scripts

### Console Commands

```lavishscript
; Run a script
run ScriptName

; Run with parameters
run ScriptName "param1" "param2"

; End a running script
endscript ScriptName

; List running scripts
scripts

; Reload a script (if changed)
runscript ScriptName -reload
```

### Script Organization

Recommended folder structure:
```
InnerSpace/
  Scripts/
    MyBot/
      MyBot.iss           ; Main script
      Lib/
        Targeting.iss     ; Library functions
        Movement.iss
        Combat.iss
      Config/
        Settings.xml      ; Configuration
```
<!-- CLAUDE_SKIP_END -->

---

<!-- CLAUDE_SKIP_START -->
## Next Steps

### Beginner Path

1. **Learn more LavishScript** - [01_LavishScript_Fundamentals.md](01_LavishScript_Fundamentals.md)
2. **Explore the API** - [03_API_Reference.md](03_API_Reference.md)
3. **Study core concepts** - [04_Core_Concepts.md](04_Core_Concepts.md)
4. **Try working examples** - [06_Working_Examples.md](06_Working_Examples.md)

### Intermediate Path

1. **Learn patterns** - [05_Patterns_And_Best_Practices.md](05_Patterns_And_Best_Practices.md)
2. **Build simple automation** - [06_Working_Examples.md](06_Working_Examples.md)
3. **Study bot architectures** - [18_Bot_Architecture_Analysis.md](18_Bot_Architecture_Analysis.md)

### Advanced Path

1. **Combat automation** - [15_Combat_Automation.md](15_Combat_Automation.md)
2. **Mining automation** - [16_Mining_And_Hauling.md](16_Mining_And_Hauling.md)
3. **Fleet coordination** - [17_Fleet_Operations.md](17_Fleet_Operations.md)
4. **Advanced patterns** - [07_Advanced_Patterns_And_Examples.md](07_Advanced_Patterns_And_Examples.md)
<!-- CLAUDE_SKIP_END -->

---

<!-- CLAUDE_SKIP_START -->
## Troubleshooting

### Problem: "ISXEVE is not defined"

**Solution:**
```lavishscript
ext load isxeve
```

### Problem: Script won't run

**Check:**
1. File has `.iss` extension
2. File is in `InnerSpace/Scripts/` directory
3. Using correct syntax: `run ScriptName` (no `.iss`)

### Problem: "${Me} is NULL" or similar

**Cause:** ISXEVE not ready yet

**Solution:** Always check `${ISXEVE.IsReady}` first:
```lavishscript
while !${ISXEVE.IsReady}
{
    wait 10
}
```

### Problem: "Entity doesn't exist" errors

**Cause:** Trying to access object that doesn't exist

**Solution:** Always check `(exists)`:
```lavishscript
if ${Entity[${ID}](exists)}
{
    ; Safe to use
    echo "${Entity[${ID}].Name}"
}
```

### Problem: Script locks up/freezes

**Cause:** Infinite loop without `wait`

**Solution:** Always use `wait` in loops:
```lavishscript
; WRONG - freezes
while TRUE
{
    call DoStuff
}

; CORRECT
while TRUE
{
    call DoStuff
    wait 10  ; CRITICAL!
}
```

### Problem: Can't see console output

**Solution:**
- Open console: Ctrl+I (default)
- Or check InnerSpace log file
<!-- CLAUDE_SKIP_END -->

---

<!-- CLAUDE_SKIP_START -->
## Quick Reference Card

### Essential Patterns

```lavishscript
; Wait for ISXEVE
while !${ISXEVE.IsReady}
{
    wait 10
}

; Check existence
if ${Object(exists)}
{
    ; Use object
}

; Main loop
while ${Running}
{
    ; Do stuff
    wait 20  ; 2 second pulse
}

; Iterate collection
variable iterator Item
Collection:GetIterator[Item]
if ${Item:First(exists)}
{
    do
    {
        ; Process Item.Value
    }
    while ${Item:Next(exists)}
}
```

### Common TLOs

```lavishscript
${Me}               ; Your character
${MyShip}           ; Your ship
${EVE}              ; Game system
${Local}            ; Local chat
${Station}          ; Current station
${Entity[id]}       ; Entity by ID
```

### Common Operations

```lavishscript
; Lock target
Entity[${ID}]:LockTarget[]

; Activate module
MyShip.Module[1]:Activate[]

; Approach entity
Entity[${ID}]:Approach[]

; Open cargo
MyShip:Open[]

; Stop ship
EVE:Execute[CmdStopShip]
```
<!-- CLAUDE_SKIP_END -->

---

<!-- CLAUDE_SKIP_START -->
## Common Beginner Mistakes

### 1. Forgetting NULL Checks

```lavishscript
; BAD - Will error if no target
echo ${MyShip.ActiveTarget.Name}

; GOOD - Checks existence first
if ${MyShip.ActiveTarget(exists)}
    echo ${MyShip.ActiveTarget.Name}
```

### 2. Case Sensitivity in Strings

```lavishscript
; LavishScript member names are case-insensitive
echo ${Me.Name}    ; Works
echo ${Me.name}    ; Works
echo ${ME.NAME}    ; Works

; But string comparisons ARE case-sensitive
if ${Me.Name.Equal["Bob"]}    ; Only matches "Bob", not "bob"
if ${Me.Name.Equal["bob"]}    ; Only matches "bob", not "Bob"
```

### 3. Not Waiting for Async Data

```lavishscript
; Some data loads asynchronously
variable entity Target

; BAD - May not be ready yet
Target:Set[${Entity[${ID}]}]
echo ${Target.Name}

; GOOD - Check if ready first
while !${ISXEVE.IsReady}
    wait 10

Target:Set[${Entity[${ID}]}]
if ${Target(exists)}
    echo ${Target.Name}
```
<!-- CLAUDE_SKIP_END -->

---

## Practice Exercises

Try building these simple scripts to practice:

### Exercise 1: Capacitor Monitor
Create a script that checks your capacitor every 2 seconds and echoes a warning if it drops below 30%.

**Solution:**

```lavishscript
function main()
{
    while !${ISXEVE.IsReady}
        wait 10

    variable bool Running = TRUE
    echo "Capacitor Monitor started"

    while ${Running}
    {
        variable float CapPct = ${MyShip.CapacitorPct}

        if ${CapPct} < 30
            echo "WARNING: Capacitor at ${CapPct.Int}%!"
        elseif ${CapPct} < 50
            echo "CAUTION: Capacitor at ${CapPct.Int}%"

        wait 2000
    }
}
```

### Exercise 2: Entity Info Display
Create a script that displays detailed information about your current target.

**Solution:**

```lavishscript
function main()
{
    while !${ISXEVE.IsReady}
        wait 10

    if !${MyShip.ActiveTarget(exists)}
    {
        echo "No target selected"
        return
    }

    variable entity Target = ${MyShip.ActiveTarget.ToEntity}

    echo "===== Target Information ====="
    echo "Name: ${Target.Name}"
    echo "Distance: ${Target.Distance}m"
    echo "Type: ${Target.Type}"
    echo "Group: ${Target.Group}"

    if ${Target.IsNPC}
        echo "Category: NPC"
    elseif ${Target.IsPC}
        echo "Category: Player"

    if ${Target.IsLockedTarget}
        echo "Status: LOCKED"
    else
        echo "Status: Not locked"
}
```

### Exercise 3: Cargo Space Checker
Create a script that reports how much cargo space you have used and remaining.

**Solution:**

```lavishscript
function main()
{
    while !${ISXEVE.IsReady}
        wait 10

    variable float UsedCapacity = ${MyShip.UsedCargoCapacity}
    variable float TotalCapacity = ${MyShip.CargoCapacity}
    variable float FreeCapacity = ${Math.Calc[${TotalCapacity}-${UsedCapacity}]}
    variable float PercentUsed = ${Math.Calc[${UsedCapacity}*100/${TotalCapacity}]}

    echo "===== Cargo Report ====="
    echo "Used: ${UsedCapacity} m3"
    echo "Free: ${FreeCapacity} m3"
    echo "Total: ${TotalCapacity} m3"
    echo "Capacity: ${PercentUsed.Int}% full"

    if ${PercentUsed} > 90
        echo "WARNING: Cargo almost full!"
}
```
---

<!-- CLAUDE_SKIP_START -->
## Congratulations!

You've completed the ISXEVE Quick Start Guide! You now know how to:

- ✓ Verify ISXEVE installation
- ✓ Write basic ISXEVE scripts
- ✓ Access character and ship data
- ✓ Use TLOs and object members
- ✓ Handle existence checking
- ✓ Run and debug scripts

---

*Last Updated: 2025-10-26*
<!-- CLAUDE_SKIP_END -->
