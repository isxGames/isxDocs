# ISXEQ2 Quick Start Guide

**Estimated Time:** 5-10 minutes

Get up and running with ISXEQ2 scripting in minutes!

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Verify Installation](#verify-installation)
- [Your First Script](#your-first-script)
- [Understanding the Example](#understanding-the-example)
- [Next Steps](#next-steps)

---

<!-- CLAUDE_SKIP_START -->
## Prerequisites

Before you begin, ensure you have:

1. **InnerSpace** installed and working
2. **ISXEQ2 extension** loaded in your EverQuest 2 session
3. **EverQuest 2** running and logged into a character
4. Basic familiarity with file editing (any text editor will work)

---

## Verify Installation

### Step 1: Check ISXEQ2 is Loaded

1. Open the InnerSpace console (click the InnerSpace icon in your system tray)
2. Type the following command and press Enter:

```
echo ${ISXEQ2(exists)}
```

**Expected Result:** You should see `TRUE`.

**If you see `FALSE`:** ISXEQ2 is not loaded. Load it with:

```
extension isxeq2
```

### Step 2: Test Basic API Access

Type these commands one at a time to verify the API is working:

```
echo ${Me.Name}
echo ${Me.Level}
echo ${Me.CurrentHealth}
```

**Expected Results:**
- First command shows your character name
- Second command shows your character level
- Third command shows your current health value

If all three work, you're ready to write scripts!
<!-- CLAUDE_SKIP_END -->

---

## Your First Script

### Step 1: Create the Script File

1. Navigate to your InnerSpace Scripts folder (typically in your InnerSpace installation directory under `Scripts\`)

2. Create a new file called `MyFirstScript.iss`

3. Open it in your text editor (Notepad, Notepad++, VS Code, etc.)

### Step 2: Write the Script

Copy and paste this code into your new file:

```lavishscript
;*****************************************************
; MyFirstScript.iss
; A simple character info display script
;*****************************************************

function main()
{
    ; Wait for ISXEQ2 to be ready
    while !${ISXEQ2.IsReady}
    {
        wait 10
    }

    echo "========================================"
    echo "   Character Information"
    echo "========================================"
    echo "Name: ${Me.Name}"
    echo "Level: ${Me.Level}"
    echo "Class: ${Me.SubClass}"
    echo "Race: ${Me.Race}"
    echo ""
    echo "Health: ${Me.CurrentHealth} / ${Me.MaxHealth}"
    echo "Power: ${Me.CurrentPower} / ${Me.MaxPower}"
    echo ""
    echo "Current Zone: ${Zone.Name}"
    echo "Position: ${Me.X}, ${Me.Y}, ${Me.Z}"
    echo "Heading: ${Me.Heading}"
    echo ""

    ; Check target
    if ${Target(exists)}
    {
        echo "Current Target: ${Target.Name}"
        echo "Target Level: ${Target.Level}"
        echo "Target Distance: ${Target.Distance}"
        echo "Target Type: ${Target.Type}"
    }
    else
    {
        echo "No target selected"
    }

    echo "========================================"
    echo ""
    echo "Script complete! Try targeting something and run again."
}
```

### Step 3: Save and Run the Script

1. Save the file

2. In the InnerSpace console, type:

```
run MyFirstScript
```

3. Watch the output appear in the console!

---

## Understanding the Example

Let's break down the key parts of this script:

### The main() Function

```lavishscript
function main()
{
    ; Your code here
}
```

Every ISXEQ2 script starts with a `main()` function. This is the entry point that runs when you execute the script.

### Waiting for ISXEQ2

```lavishscript
while !${ISXEQ2.IsReady}
{
    wait 10
}
```

This ensures ISXEQ2 is fully loaded before accessing game data. Always include this at the start of your scripts.

### Accessing Character Data

```lavishscript
echo "Name: ${Me.Name}"
echo "Level: ${Me.Level}"
echo "Health: ${Me.CurrentHealth} / ${Me.MaxHealth}"
```

- `${Me}` is a **Top-Level Object (TLO)** that represents your character
- `.Name`, `.Level`, `.CurrentHealth` are **members** of the `char` datatype
- `${...}` syntax evaluates the expression and returns the value

### NULL Checks

```lavishscript
if ${Target(exists)}
{
    echo "Current Target: ${Target.Name}"
}
```

Always check if an object exists before accessing its members. This prevents errors when the target doesn't exist.

### Comments

```lavishscript
; This is a comment
```

Lines starting with `;` are comments and are ignored by the script engine.

---

## Common First Scripts

### Example 2: Health Monitor

A script that continuously monitors health percentage:

```lavishscript
; HealthMonitor.iss
; Monitors health percentage and warns if low

variable bool Running = TRUE

function main()
{
    ; Wait for ISXEQ2
    while !${ISXEQ2.IsReady}
    {
        wait 10
    }

    echo "Health Monitor started"
    echo "Press End Script to stop"

    ; Main loop
    while ${Running}
    {
        variable float HealthPct = ${Math.Calc[${Me.CurrentHealth}*100/${Me.MaxHealth}]}

        ; Check health levels
        if ${HealthPct} < 25
        {
            echo "WARNING: Health CRITICAL (${HealthPct.Int}%)"
        }
        elseif ${HealthPct} < 50
        {
            echo "CAUTION: Health LOW (${HealthPct.Int}%)"
        }

        ; Wait 2 seconds between checks
        wait 20
    }

    echo "Health Monitor stopped"
}
```

**To stop:** In InnerSpace console, type: `endscript HealthMonitor`

### Example 3: Target Info

Display information about your current target:

```lavishscript
; TargetInfo.iss
; Displays information about current target

function main()
{
    while !${ISXEQ2.IsReady}
    {
        wait 10
    }

    ; Check if we have a target
    if !${Target(exists)}
    {
        echo "No target selected!"
        return
    }

    echo "====================================="
    echo "TARGET INFORMATION"
    echo "====================================="

    echo "Name: ${Target.Name}"
    echo "Level: ${Target.Level}"
    echo "Type: ${Target.Type}"
    echo "Distance: ${Target.Distance}"
    echo "Health: ${Target.Health}%"
    echo "Con Color: ${Target.ConColor}"

    ; Check if NPC or player
    if ${Target.IsEpic}
    {
        echo "Category: EPIC NPC"
    }
    elseif ${Target.Type.Equal["NPC"]}
    {
        echo "Category: NPC"
    }
    elseif ${Target.Type.Equal["PC"]}
    {
        echo "Category: Player Character"
    }

    ; Aggro status
    if ${Target.IsAggro}
    {
        echo "Status: AGGRESSIVE"
    }
    else
    {
        echo "Status: Not Aggressive"
    }

    echo "====================================="
}
```

**Usage:**
1. Target something in game
2. Run: `run TargetInfo`

### Example 4: Auto-Target Nearest

Automatically target the nearest NPC:

```lavishscript
; AutoTarget.iss
; Automatically targets nearest NPC

function main()
{
    while !${ISXEQ2.IsReady}
    {
        wait 10
    }

    ; Find nearest actor
    variable index:actor Actors
    variable iterator ActorIt

    EQ2:QueryActors[Actors, "Type = NPC"]
    Actors:GetIterator[ActorIt]

    if !${ActorIt:First(exists)}
    {
        echo "No NPCs found nearby"
        return
    }

    variable float NearestDistance = 999999
    variable int NearestID = 0

    ; Find nearest
    do
    {
        if ${ActorIt.Value.Distance} < ${NearestDistance}
        {
            NearestDistance:Set[${ActorIt.Value.Distance}]
            NearestID:Set[${ActorIt.Value.ID}]
        }
    }
    while ${ActorIt:Next(exists)}

    ; Target it
    if ${NearestID} > 0
    {
        echo "Targeting: ${Actor[${NearestID}].Name} (${NearestDistance}m away)"
        EQ2Execute /target_id ${NearestID}
    }
}
```

---

## Experiment and Learn

Try modifying the script to explore more:

### Example 1: Show Inventory

Add this before the final echo:

```lavishscript
echo ""
echo "Inventory Slots Free: ${Me.InventorySlotsFree}"
echo "Bank Slots Free: ${Me.BankSlotsFree}"
```

### Example 2: List Abilities

Add this to show your first few abilities:

```lavishscript
echo ""
echo "First 5 Abilities:"
variable int i
for (i:Set[1]; ${i} <= 5 && ${i} <= ${Me.NumAbilities}; i:Inc)
{
    if ${Me.Ability[${i}].IsReady}
        echo "${i}. ${Me.Ability[${i}].ToAbilityInfo.Name} - READY"
    else
        echo "${i}. ${Me.Ability[${i}].ToAbilityInfo.Name} - ${Me.Ability[${i}].TimeUntilReady}s"
}
```

### Example 3: Show Equipped Items

Add this to see your equipped gear:

```lavishscript
echo ""
echo "Equipped Weapon: ${Me.Equipment[primary].Name}"
echo "Equipped Chest: ${Me.Equipment[chest].Name}"
```

---

<!-- CLAUDE_SKIP_START -->
## Common Beginner Mistakes

### 1. Forgetting NULL Checks

```lavishscript
; BAD - Will error if no target
echo ${Target.Name}

; GOOD - Checks existence first
if ${Target(exists)}
    echo ${Target.Name}
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
variable item MyItem
MyItem:Set[${Me.Inventory[5]}]

; BAD - May not be loaded yet
echo ${MyItem.ToItemInfo.Description}

; GOOD - Wait for it to load
if !${MyItem.IsItemInfoAvailable}
    wait 10 ${MyItem.IsItemInfoAvailable}

echo ${MyItem.ToItemInfo.Description}
```
<!-- CLAUDE_SKIP_END -->

---

<!-- CLAUDE_SKIP_START -->
## Next Steps

Congratulations! You've created and run your first ISXEQ2 script. Here's what to explore next:

### 1. Learn LavishScript (If Needed)
If you're new to LavishScript, read **[01_LavishScript_Fundamentals.md](01_LavishScript_Fundamentals.md)** to understand:
- Variables, functions, and objects
- Loops and conditionals
- Collections (index type)
- Web requests and audio

### 2. Learn Core ISXEQ2 Concepts
Read **[04_Core_Concepts.md](04_Core_Concepts.md)** to understand:
- Datatypes and inheritance
- Variables and scoping
- Query syntax
- Events and atoms

### 3. Study Working Examples
Browse **[06_Working_Examples.md](06_Working_Examples.md)** for:
- Inventory management
- Casting abilities
- UI interaction
- Event-driven scripts
- Combat automation

### 4. Master the API
Bookmark **[00_MASTER_GUIDE.md](00_MASTER_GUIDE.md)** and **[03_API_Reference.md](03_API_Reference.md)** for quick lookups of:
- All datatypes and their members
- All commands
- All events
- Complete method signatures

### 5. Learn Best Practices
Read **[05_Patterns_And_Best_Practices.md](05_Patterns_And_Best_Practices.md)** to write:
- Clean, maintainable code
- Efficient scripts
- Robust error handling
- Professional naming conventions

### 6. Advanced Topics
Explore specialized guides:
- **[07_Advanced_Patterns_And_Examples.md](07_Advanced_Patterns_And_Examples.md)** - Advanced patterns
- **[09_LavishGUI2_UI_Guide.md](09_LavishGUI2_UI_Guide.md)** - Create modern UIs
- **[11_JSON_Guide.md](11_JSON_Guide.md)** - Work with JSON data
- **[12_LavishMachine_Guide.md](12_LavishMachine_Guide.md)** - Async task execution
<!-- CLAUDE_SKIP_END -->

---

<!-- CLAUDE_SKIP_START -->
## Quick Reference Card

### Essential TLOs

| TLO | Returns | Description |
|-----|---------|-------------|
| `${Me}` | char | Your character |
| `${Target}` | actor | Current target |
| `${Zone}` | zone | Current zone |
| `${Actor[name]}` | actor | Actor by name |
| `${EQ2}` | eq2 | Game utilities |

### Essential Commands

```lavishscript
echo <text>                  ; Output to console
EQ2Execute <command>         ; Execute EQ2 command
Target <name/type>           ; Target an actor
Face                         ; Face target
wait <ms>                    ; Wait milliseconds
run <script>                 ; Run a script
```

### Essential Checks

```lavishscript
${Object(exists)}            ; Check if object exists
${Value.Equal[text]}         ; String comparison
${Number} > ${OtherNumber}   ; Numeric comparison
${Boolean}                   ; TRUE/FALSE check
```

### Essential Patterns

```lavishscript
; Wait for condition
wait 50 ${Condition}

; Loop
while ${Condition}
{
    ; Code
}

; For loop
variable int i
for (i:Set[1]; ${i} <= 10; i:Inc)
{
    ; Code
}

; NULL check
if ${Object(exists)}
{
    ; Safe to access
}
```
<!-- CLAUDE_SKIP_END -->

---

<!-- CLAUDE_SKIP_START -->
## Getting Help

- **Need an example?** Browse [06_Working_Examples.md](06_Working_Examples.md)
- **API question?** Look up in [00_MASTER_GUIDE.md](00_MASTER_GUIDE.md) or [03_API_Reference.md](03_API_Reference.md)
- **Need advanced patterns?** Check [07_Advanced_Patterns_And_Examples.md](07_Advanced_Patterns_And_Examples.md)
- **New to LavishScript?** Read [01_LavishScript_Fundamentals.md](01_LavishScript_Fundamentals.md)
<!-- CLAUDE_SKIP_END -->

---

<!-- CLAUDE_SKIP_START -->
## Practice Exercises

Try building these simple scripts to practice:

### Exercise 1: Health Monitor
Create a script that checks your health every 2 seconds and echoes a warning if it drops below 50%.

**Solution:**

```lavishscript
function main()
{
    while !${ISXEQ2.IsReady}
        wait 10

    variable bool Running = TRUE

    while ${Running}
    {
        variable float HealthPercent
        HealthPercent:Set[${Math.Calc[${Me.CurrentHealth}*100/${Me.MaxHealth}]}]

        if ${HealthPercent} < 50
            echo "WARNING: Health at ${HealthPercent.Int}%!"

        wait 2000
    }
}
```

### Exercise 2: Target Info
Create a script that displays detailed information about your current target.

**Solution:**

```lavishscript
function main()
{
    while !${ISXEQ2.IsReady}
        wait 10

    if !${Target(exists)}
    {
        echo "No target selected"
        return
    }

    echo "===== Target Information ====="
    echo "Name: ${Target.Name}"
    echo "Level: ${Target.Level}"
    echo "Type: ${Target.Type}"
    echo "Health: ${Target.Health}%"
    echo "Distance: ${Target.Distance}"
    echo "Con Color: ${Target.ConColor}"

    if ${Target.IsAggro}
        echo "Status: AGGRESSIVE"
    else
        echo "Status: Neutral"
}
```

### Exercise 3: Inventory Counter
Create a script that counts how many items you have in your inventory.

**Solution:**

```lavishscript
function main()
{
    while !${ISXEQ2.IsReady}
        wait 10

    variable index:item Items
    variable iterator ItemIt
    variable int ItemCount = 0

    Me:QueryInventory[Items,"Location == \"Inventory\""]
    Items:GetIterator[ItemIt]

    if ${ItemIt:First(exists)}
    {
        do
        {
            ItemCount:Inc
        }
        while ${ItemIt:Next(exists)}
    }

    echo "You have ${ItemCount} items in your inventory"
    echo "Free slots: ${Me.InventorySlotsFree}"
}
```
<!-- CLAUDE_SKIP_END -->

---

## Troubleshooting

### Problem: "ISXEQ2 is not defined"

**Solution:**
```lavishscript
extension isxeq2
```

### Problem: Script won't run

**Check:**
1. File has `.iss` extension
2. File is in `InnerSpace/Scripts/` directory
3. Using correct syntax: `run ScriptName` (no `.iss`)

### Problem: "${Me} is NULL" or similar

**Cause:** ISXEQ2 not ready yet

**Solution:** Always check `${ISXEQ2.IsReady}` first:
```lavishscript
while !${ISXEQ2.IsReady}
{
    wait 10
}
```

### Problem: "Object doesn't exist" errors

**Cause:** Trying to access object that doesn't exist

**Solution:** Always check `(exists)`:
```lavishscript
if ${Target(exists)}
{
    ; Safe to use
    echo "${Target.Name}"
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
- Open console: Click InnerSpace system tray icon
- Or check InnerSpace log file

---

*Part of ISXEQ2 Scripting Guide*
