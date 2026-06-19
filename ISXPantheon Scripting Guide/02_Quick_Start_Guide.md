# ISXPantheon Quick Start Guide

**Estimated Time:** 5-10 minutes

Get up and running with ISXPantheon scripting in minutes!

> **IMPORTANT.** ISXPantheon is in early development. Two areas are live today: `${ISXPantheon}` (extension control + helpers) and `${Login}` (login state, realm enumeration, and the login screen's buttons/input fields). In-world game-data objects like the local player and target are **planned but not yet implemented** — see [03_API_Reference.md - Planned API](03_API_Reference.md#planned-api). This guide builds its working examples on the real surface and clearly flags any planned snippet.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Verify Installation](#verify-installation)
- [Your First Script](#your-first-script)
- [Understanding the Example](#understanding-the-example)
- [Common First Scripts](#common-first-scripts)
- [Common Beginner Mistakes](#common-beginner-mistakes)
- [Next Steps](#next-steps)
- [Troubleshooting](#troubleshooting)

---

<!-- CLAUDE_SKIP_START -->
## Prerequisites

Before you begin, ensure you have:

1. **InnerSpace** installed and working
2. **ISXPantheon extension** loaded in your Pantheon: Rise of the Fallen session
3. **Pantheon: Rise of the Fallen** running
4. Basic familiarity with file editing (any text editor will work)

---

## Verify Installation

### Step 1: Check ISXPantheon is Loaded

1. Open the InnerSpace console (click the InnerSpace icon in your system tray)
2. Type the following command and press Enter:

```
echo ${ISXPantheon(exists)}
```

**Expected Result:** You should see `TRUE`.

**If you see `FALSE`:** ISXPantheon is not loaded. Load it with:

```
extension isxpantheon
```

### Step 2: Test Basic API Access

Type these commands one at a time to verify the API is working:

```
echo ${ISXPantheon.IsReady}
echo ${ISXPantheon.Version}
echo ${ISXPantheon.APIVersion}
```

**Expected Results:**
- First command shows `TRUE` once the extension has finished loading
- Second command shows the extension version string
- Third command shows the API version string

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
; A simple extension-info display script
;*****************************************************

function main()
{
    ; Wait for ISXPantheon to be ready
    while !${ISXPantheon.IsReady}
    {
        wait 10
    }

    echo "========================================"
    echo "   ISXPantheon Information"
    echo "========================================"
    echo "Version:     ${ISXPantheon.Version}"
    echo "API Version: ${ISXPantheon.APIVersion}"
    echo "Ready:       ${ISXPantheon.IsReady}"
    echo "========================================"
    echo ""
    echo "Script complete!"
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

Every ISXPantheon script starts with a `main()` function. This is the entry point that runs when you execute the script.

### Waiting for ISXPantheon

```lavishscript
while !${ISXPantheon.IsReady}
{
    wait 10
}
```

This ensures ISXPantheon is fully loaded before doing any work. Always include this at the start of your scripts.

### Accessing Extension Data

```lavishscript
echo "Version: ${ISXPantheon.Version}"
echo "Ready:   ${ISXPantheon.IsReady}"
```

- `${ISXPantheon}` is a **Top-Level Object (TLO)** that represents the extension
- `.Version`, `.IsReady` are **members** of the `isxpantheon` datatype
- `${...}` syntax evaluates the expression and returns the value

### NULL Checks

```lavishscript
if ${ISXPantheon(exists)}
{
    echo "Extension version: ${ISXPantheon.Version}"
}
```

Always check if an object exists before accessing its members. This prevents errors when the object doesn't exist.

### Comments

```lavishscript
; This is a comment
```

Lines starting with `;` are comments and are ignored by the script engine.

---

## Common First Scripts

### Example 2: Custom Variable Store

A script that stores and reads back a custom variable through the extension:

```lavishscript
; CustomVars.iss
; Demonstrates ISXPantheon custom variables

function main()
{
    while !${ISXPantheon.IsReady}
        wait 10

    ; Store a value
    ISXPantheon:SetCustomVariable[Greeting,"Hello, Pantheon"]
    ISXPantheon:SetCustomVariable[Count,5]

    ; Read it back (typed)
    echo "Greeting: ${ISXPantheon.GetCustomVariable[Greeting]}"
    echo "Count:    ${ISXPantheon.GetCustomVariable[Count,int]}"

    ; Clean up
    ISXPantheon:ClearAllCustomVariables
    echo "Done."
}
```

**To run:** `run CustomVars`

### Example 3: HTTP Request

Fetch a URL and print the response. The result arrives asynchronously via the `isxGames_onHTTPResponse` event:

```lavishscript
; FetchUrl.iss
; Demonstrates the GetURL command and its response event

function main()
{
    while !${ISXPantheon.IsReady}
        wait 10

    Event[isxGames_onHTTPResponse]:AttachAtom[OnHTTP]

    echo "Requesting..."
    GetURL "https://example.com"

    ; Keep the script alive long enough to receive the response
    wait 100
}

atom OnHTTP(int Size, string URL, string IPAddress, int ResponseCode, float TransferTime, string ResponseText, string ParsedBody)
{
    echo "Response ${ResponseCode} from ${URL} (${Size} bytes in ${TransferTime}s)"
}
```

**Usage:** `run FetchUrl`

### Example 4: In-World Game-Data Access (Planned)

> **PLANNED — NOT YET IMPLEMENTED.** ISXPantheon's in-world game-data surface (local player, entities, abilities, quests) is on the roadmap but is not available in the current build. The live surface today is `${ISXPantheon}` plus `${Login}` (the login/realm/login-UI surface — see [03_API_Reference.md - login](03_API_Reference.md#login) and the Login Screen example in [06_Working_Examples.md](06_Working_Examples.md#login-screen-state-realms-buttons-fields)). When in-world game-data TLOs and datatypes are implemented, examples will be added here.

---

## Common Beginner Mistakes

### 1. Forgetting NULL Checks

```lavishscript
; BAD - Will error if the object doesn't exist
echo ${ISXPantheon.Version}

; GOOD - Checks existence first
if ${ISXPantheon(exists)}
    echo ${ISXPantheon.Version}
```

### 2. Case Sensitivity in Strings

```lavishscript
; LavishScript member names are case-insensitive
echo ${ISXPantheon.Version}    ; Works
echo ${ISXPantheon.version}    ; Works

; But string comparisons ARE case-sensitive
if ${MyString.Equal["Bob"]}    ; Only matches "Bob", not "bob"
if ${MyString.Equal["bob"]}    ; Only matches "bob", not "Bob"
```

### 3. Not Waiting for the Extension to be Ready

```lavishscript
; BAD - the extension may not be loaded yet
echo ${ISXPantheon.Version}

; GOOD - wait for IsReady first
while !${ISXPantheon.IsReady}
    wait 10

echo ${ISXPantheon.Version}
```

---

<!-- CLAUDE_SKIP_START -->
## Next Steps

Congratulations! You've created and run your first ISXPantheon script. Here's what to explore next:

### 1. Learn LavishScript (If Needed)
If you're new to LavishScript, read **[01_LavishScript_Fundamentals.md](01_LavishScript_Fundamentals.md)** to understand:
- Variables, functions, and objects
- Loops and conditionals
- Collections (index type)
- Web requests and audio

For exhaustive command/datatype/TLO lookup tables (one canonical entry per feature, with links to the Lavish Software wiki), use the companion reference **[01b_LavishScript_Reference.md](01b_LavishScript_Reference.md)**.

### 2. Learn Core ISXPantheon Concepts
Read **[04_Core_Concepts.md](04_Core_Concepts.md)** to understand:
- Datatypes and inheritance
- Variables and scoping
- Events and atoms
- NULL checks and async data

### 3. Study Working Examples
Browse **[06_Working_Examples.md](06_Working_Examples.md)** for code examples built on the real surface.

### 4. Master the API
Bookmark **[00_MASTER_GUIDE.md](00_MASTER_GUIDE.md)** and **[03_API_Reference.md](03_API_Reference.md)** for quick lookups of:
- The `${ISXPantheon}` datatype members and methods
- Commands and events
- The planned-feature roadmap

### 5. Learn Best Practices
Read **[05_Patterns_And_Best_Practices.md](05_Patterns_And_Best_Practices.md)** to write clean, efficient, robust scripts.

### 6. Advanced Topics
Explore specialized guides:
- **[07_Advanced_Patterns_And_Examples.md](07_Advanced_Patterns_And_Examples.md)** - Advanced patterns
- **[10_LavishGUI2_UI_Guide.md](10_LavishGUI2_UI_Guide.md)** - Create modern UIs
- **[13_JSON_Guide.md](13_JSON_Guide.md)** - Work with JSON data
- **[14_LavishMachine_Guide.md](14_LavishMachine_Guide.md)** - Async task execution
<!-- CLAUDE_SKIP_END -->

---

<!-- CLAUDE_SKIP_START -->
## Quick Reference Card

### Essential TLOs

| TLO | Returns | Status | Description |
|-----|---------|--------|-------------|
| `${ISXPantheon}` | isxpantheon | REAL | Extension info, control, utilities |
| `${Login}` | login | REAL | Login / realm-selection screen |
| `${CharSelect}` | charselect | REAL | Character-selection screen (NULL unless at that scene) |
| `${CharCreate}` | charcreate | REAL | Character-creation screen (NULL unless at that scene) |
| `${Pantheon}` | pantheon | REAL | Game-wide info + render/camera control (resolution, FPS limit, VSync, cameras) |

`${Me}` and `${Radar}` exist in the source only as reserved (commented-out) top-level objects; they are not registered and return nothing today. Additional game-data TLOs are planned.

### Essential Commands

```lavishscript
echo <text>                  ; Output to console
GetURL <url>                 ; HTTP GET (async response event)
PostURL <url> <body>         ; HTTP POST (async response event)
wait <deciseconds>           ; Wait (10 = 1 second)
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
; Wait for the extension to be ready
while !${ISXPantheon.IsReady}
    wait 10

; Loop
while ${Condition}
{
    ; Code
    wait 10
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
- **New to LavishScript?** Read [01_LavishScript_Fundamentals.md](01_LavishScript_Fundamentals.md) (tutorial); use [01b_LavishScript_Reference.md](01b_LavishScript_Reference.md) for exhaustive command/datatype/TLO lookup
<!-- CLAUDE_SKIP_END -->

---

## Troubleshooting

### Problem: "ISXPantheon is not defined"

**Solution:**
```lavishscript
extension isxpantheon
```

### Problem: Script won't run

**Check:**
1. File has `.iss` extension
2. File is in `InnerSpace/Scripts/` directory
3. Using correct syntax: `run ScriptName` (no `.iss`)

### Problem: Members return nothing

**Cause:** ISXPantheon not ready yet, or you are accessing a planned (not-yet-implemented) feature.

**Solution:** Always check `${ISXPantheon.IsReady}` first:
```lavishscript
while !${ISXPantheon.IsReady}
{
    wait 10
}
```

If a game-data member (local player, target, entities, abilities, quests) returns nothing, it is likely **planned** — see [03_API_Reference.md - Planned API](03_API_Reference.md#planned-api).

### Problem: "Object doesn't exist" errors

**Cause:** Trying to access an object that doesn't exist

**Solution:** Always check `(exists)`:
```lavishscript
if ${ISXPantheon(exists)}
{
    echo "${ISXPantheon.Version}"
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
