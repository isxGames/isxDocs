# ISXPantheon Core Concepts

Essential concepts every ISXPantheon scripter needs to understand.

> **IMPORTANT.** ISXPantheon is in early development. Two areas are live today: `${ISXPantheon}` (extension control + helpers) and `${Login}` (login state, realm enumeration, and the login screen's buttons/input fields, via the `login`/`realm`/`uibutton`/`uitext`/`uiinputfield`/`uicolor` datatypes that inherit from the base `object` type). In-world game-data objects (local player, target, entities, items, abilities, quests) and game events are **planned but not yet implemented** — see [03_API_Reference.md - Planned API](03_API_Reference.md#planned-api). The concepts below are universal LavishScript/extension concepts; examples that reference in-world game data are clearly flagged as planned and will not run today.

---

## Table of Contents

1. [Datatypes and Objects](#datatypes-and-objects)
2. [Top-Level Objects (TLOs)](#top-level-objects-tlos)
3. [Datatype Inheritance](#datatype-inheritance)
4. [Members and Methods](#members-and-methods)
5. [NULL Checks and Existence Validation](#null-checks-and-existence-validation)
6. [Variable Declarations and Scoping](#variable-declarations-and-scoping)
7. [Asynchronous Data Loading](#asynchronous-data-loading)
8. [Collections and Iterators](#collections-and-iterators)
9. [Events and Atoms](#events-and-atoms)
10. [LavishScript Basics](#lavishscript-basics)

---

## Datatypes and Objects

### What is a Datatype?

In ISXPantheon, every piece of data has a specific **datatype** that defines what information it contains and what actions you can perform on it.

Think of a datatype as a blueprint or template:

```lavishscript
; ${ISXPantheon} is an "isxpantheon" datatype
echo ${ISXPantheon.Version}     ; isxpantheon has a Version member
echo ${ISXPantheon.IsReady}     ; isxpantheon has an IsReady member
```

### Common Datatypes

| Datatype | Represents | Example | Status |
|----------|------------|---------|--------|
| **isxpantheon** | The extension | `${ISXPantheon}` | REAL |
| **pantheon** | Reserved game object | `${Pantheon}` | Registered (empty) |
| **entity** | Any entity in the world | (no TLO yet) | Registered (empty) |
| **ability** | A character ability/skill | (no TLO yet) | Registered (empty) |
| **quest** | A quest entry | (no TLO yet) | Registered (empty) |

The `entity`, `ability`, and `quest` datatypes are registered today but expose no working members yet. `${Me}` and `${Radar}` exist in the source only as reserved (commented-out) top-level objects.

**Key Point:** You can't mix datatypes. An object of one datatype won't have another datatype's members.

---

## Top-Level Objects (TLOs)

**Top-Level Objects (TLOs)** are your entry points into extension and game data. They're globally accessible and always available.

### Essential TLOs

```lavishscript
${ISXPantheon}           ; Extension info, control, utilities (isxpantheon datatype) — REAL
${Pantheon}              ; Reserved game object (pantheon datatype) — registered, empty
```

### Planned Game TLOs

> **PLANNED — NOT YET IMPLEMENTED.** Additional game-data top-level objects are on the roadmap but are not available in the current build. The API is provisional and WILL change. `${Me}` and `${Radar}` exist in the source only as reserved (commented-out) top-level objects — they are not registered and return nothing today. Do not write production scripts against any of this surface.

### Using TLOs

```lavishscript
; TLOs are accessed directly
echo ${ISXPantheon.Version}

; Some TLOs take parameters (the helper members on ${ISXPantheon} do)
echo ${ISXPantheon.GetCustomVariable[MyFlag,int]}
```

**Key Point:** TLOs don't need to be declared - they're always available.

---

## Datatype Inheritance

Many datatypes **inherit** from other datatypes, gaining all their members and methods. A derived datatype exposes all of its parent's members in addition to its own.

### Inheritance Hierarchy

```
parent datatype
  ↓
derived datatype has all parent members PLUS its own derived-specific members
```

A concrete inheritance chain already exists in the live login surface: the `login`, `realm`, `uibutton`, `uitext`, and `uiinputfield` datatypes all **inherit from the base `object` type**, so each of them exposes every member of `object` (currently none) in addition to its own. As `object` gains members later, every one of those derived types picks them up automatically. See [03_API_Reference.md - object](03_API_Reference.md#object) for the full login/UI datatype family.

The planned in-world game-data datatypes are also intended to use inheritance — for example, the local-player datatype is expected to inherit from the generic entity datatype, so the local player would expose all entity members plus player-specific ones. Those types are not implemented yet.

> **PLANNED — NOT YET IMPLEMENTED.** The in-world game-data datatypes are expected to use inheritance once implemented — for example, a local-player datatype inheriting from the generic entity datatype — but no concrete members exist for them in the live surface today. Member names will be documented here only after they are implemented.

For the underlying LavishScript inheritance mechanics (how `objectdef ... inherits` works), see [01_LavishScript_Fundamentals.md](01_LavishScript_Fundamentals.md).

**Key Point:** Understanding inheritance helps you know what members are available — today for the login/UI datatypes that inherit from `object`, and later once the in-world game-data types are implemented.

---

## Members and Methods

Every ISXPantheon datatype exposes two kinds of access: **members** (read values with `.`) and **methods** (perform actions with `:`). For the underlying LavishScript concepts and how to define your own members/methods on custom `objectdef` types, see [01_LavishScript_Fundamentals.md - Members](01_LavishScript_Fundamentals.md#members) and [01_LavishScript_Fundamentals.md - Methods](01_LavishScript_Fundamentals.md#methods). For the exhaustive object-types catalogue (every built-in datatype with its members/methods and a link to the canonical wiki page), see [01b §3 Object Types](01b_LavishScript_Reference.md#3-object-types).

### Member Access

```lavishscript
${ISXPantheon.Version}      ; isxpantheon member - returns a version string
${ISXPantheon.IsReady}      ; isxpantheon member - returns TRUE/FALSE
${ISXPantheon.GetCustomVariable[MyFlag,int]}   ; member with a parameter
```

### Method Calls

```lavishscript
ISXPantheon:SetCustomVariable[MyFlag,1]    ; Set a custom variable
ISXPantheon:ClearAllCustomVariables        ; Clear all custom variables
ISXPantheon:Reload[10]                     ; Reload the extension after a delay
```

**Syntax Summary:**

```lavishscript
${Object.Member}         ; Get a value from an object
Object:Method            ; Perform an action on an object
Object:Method[param]     ; Perform an action with a parameter
```

**Key Point:** Many methods require the object to exist - always pair method calls with `(exists)` checks (see next section).

---

## NULL Checks and Existence Validation

**THE MOST IMPORTANT CONCEPT:** Always check if something exists before accessing it.

### The Problem

```lavishscript
; BAD - Will error if the object doesn't exist
echo ${SomeObject.Name}

; Console output:
; Invalid type
```

### The Solution: (exists) Check

```lavishscript
; GOOD - Check first
if ${ISXPantheon(exists)}
{
    echo ${ISXPantheon.Version}
}
else
{
    echo "Extension not loaded"
}
```

### Common NULL Check Patterns

```lavishscript
; Check the extension is present
if ${ISXPantheon(exists)}
    echo ${ISXPantheon.Version}

; Check a collection has items
if ${MyItems.Used} > 0
    echo "Found ${MyItems.Used} items"
```

When the planned game-data TLOs exist, the same `(exists)` pattern will apply to them. Until then, use it only against the real `${ISXPantheon}` object as shown above.

### Defensive Coding Pattern

```lavishscript
function SafeVersionInfo()
{
    ; Guard clause - exit early if the extension isn't ready
    if !${ISXPantheon(exists)} || !${ISXPantheon.IsReady}
    {
        echo "ISXPantheon not ready"
        return
    }

    ; Now safe to access
    echo "Version: ${ISXPantheon.Version}"
    echo "API:     ${ISXPantheon.APIVersion}"
}
```

**Key Point:** **ALWAYS** use `(exists)` checks. This prevents most script errors.

---

## Variable Declarations and Scoping

For the fundamentals of declaring variables, variable types, scopes (script vs local vs global), and assignment, see [01_LavishScript_Fundamentals.md - Variables and Data Types](01_LavishScript_Fundamentals.md#variables-and-data-types) and [01_LavishScript_Fundamentals.md - Variable Scope](01_LavishScript_Fundamentals.md#best-practices-summary).

This section covers how variables are used with ISXPantheon datatypes.

### Declaring Datatype Variables

ISXPantheon datatypes can be stored in local or script-scoped variables just like primitive types. The planned game-data datatypes will be usable the same way once implemented:

```lavishscript
variable string Ver
variable index:string Names              ; Collection of strings
variable iterator ResultIt               ; Iterator for collections
```

```lavishscript
; PLANNED — these collection element types depend on the planned datatypes
; variable index:entity NearbyEntities
```

### Common Pattern: Shorthand Variable

Assigning an object to a variable avoids repeating long member paths:

```lavishscript
; Cache a frequently-read value
variable string Ver
Ver:Set[${ISXPantheon.Version}]

echo "Running version ${Ver}"
```

**Key Point:** Script-scoped variables are useful for persistent state (settings, timers, flags) across a script's lifetime; local variables should be preferred for temporary calculations inside functions.

---

## Asynchronous Data Loading

Some data is delivered **asynchronously** (in the background) rather than being returned by the call that requests it. ISXPantheon has one such mechanism today: HTTP requests. The `GetURL` and `PostURL` commands return immediately, and the response arrives later through the `isxGames_onHTTPResponse` event. You attach an atom to that event, fire the request, and handle the result when the event triggers — this is event-driven, not poll-an-availability-member.

### The Extension Readiness Gate

Separately from HTTP, wait for the extension itself to finish loading before doing anything at script start:

```lavishscript
; Wait until the extension has finished loading before doing anything
while !${ISXPantheon.IsReady}
    waitframe
```

### Asynchronous HTTP Responses (Event-Driven)

`GetURL` / `PostURL` hand back control right away; the body comes back later via the `isxGames_onHTTPResponse` event. Attach an atom, send the request, and process the response inside the atom:

```lavishscript
atom OnHTTP(int Size, string URL, string IPAddress, int ResponseCode, float TransferTime, string ResponseText, string ParsedBody)
{
    echo "HTTP ${ResponseCode} from ${URL} (${Size} bytes)"
}

Event[isxGames_onHTTPResponse]:AttachAtom[OnHTTP]
GetURL "https://example.com"
```

See [Events and Atoms](#events-and-atoms) below for the full HTTP event flow.

The loading semantics of any future game-data APIs are not defined yet, so this guide does not describe how they will deliver data.

**Key Point:** Wait for `${ISXPantheon.IsReady}` at script start. The only asynchronous data ISXPantheon delivers today is HTTP responses, handled through the `isxGames_onHTTPResponse` event.

---

## Collections and Iterators

When you need to process multiple results, you store them in a **collection** and walk them with an **iterator**.

For the core LavishScript `index` and `collection` types themselves - including `Insert`, `Remove`, `Collapse`, `Clear`, `.Used`, and `AsJSON`/`FromJSON` serialization - see [01_LavishScript_Fundamentals.md - Collections and Lists](01_LavishScript_Fundamentals.md#collections-and-lists). The full container catalogue (`array`, `queue`, `stack`, `set`, `variablescope`, `objectcontainer`) is in [01b §3.5 Containers](01b_LavishScript_Reference.md#35-containers).

### Standard Iterator Pattern

```lavishscript
; Step 1: Declare collection and iterator
variable index:string MyList
variable iterator ListIt

; Step 2: Populate the collection
MyList:Insert["alpha"]
MyList:Insert["beta"]
MyList:Insert["gamma"]

; Step 3: Get iterator
MyList:GetIterator[ListIt]

; Step 4: Loop through results
if ${ListIt:First(exists)}
{
    do
    {
        echo "Item: ${ListIt.Value}"
    }
    while ${ListIt:Next(exists)}
}
```

### Collection Members

```lavishscript
echo ${MyList.Used}       ; Number of items in collection
echo ${MyList.Size}       ; Total capacity of collection

if ${MyList.Used} == 0
    echo "No items"
```

When the planned entity-enumeration API arrives, the same iterator pattern will be used to walk a collection of entities. That enumeration is **planned — not yet implemented**; see [03_API_Reference.md - Planned API](03_API_Reference.md#planned-api).

**Key Point:** The iterator pattern is the standard way to process collection results.

---

## Events and Atoms

LavishScript extensions can fire **events** when specific actions occur. You attach **atom** handlers to respond to these events.

Today ISXPantheon fires only the framework HTTP-response event. Game events (entity spawn/despawn, chat, zoning, etc.) are **planned — not yet implemented**.

### Event Registration

```lavishscript
; Attach your handler to an event
Event[isxGames_onHTTPResponse]:AttachAtom[OnHTTP]
```

### Atom Handler

```lavishscript
; Define what happens when the event fires
atom OnHTTP(int Size, string URL, string IPAddress, int ResponseCode, float TransferTime, string ResponseText, string ParsedBody)
{
    echo "HTTP ${ResponseCode} from ${URL} (${Size} bytes)"
}
```

### Complete Event Example

```lavishscript
function main()
{
    ; Wait for ISXPantheon
    while !${ISXPantheon.IsReady}
        wait 10

    ; Register the HTTP response event
    Event[isxGames_onHTTPResponse]:AttachAtom[OnHTTP]

    echo "Event handler active"

    GetURL "https://example.com"

    ; Keep the script running long enough to receive the response
    wait 100
}

atom OnHTTP(int Size, string URL, string IPAddress, int ResponseCode, float TransferTime, string ResponseText, string ParsedBody)
{
    echo "Response ${ResponseCode} from ${URL}"
}
```

### Event Detachment

```lavishscript
; Stop listening to the event
Event[isxGames_onHTTPResponse]:DetachAtom[OnHTTP]
```

### Planned Game Events

> **PLANNED — NOT YET IMPLEMENTED.** Game events (entity spawn/despawn, incoming chat text, zoning, level change, etc.) are on the roadmap but are not wired up in the current build. They will never fire today, so do not register handlers for them yet.

**Key Point:** Event-driven scripts are reactive and efficient — once game events exist, this same attach/atom pattern will apply to them.

---

## LavishScript Basics

This guide focuses on ISXPantheon-specific concepts. For core LavishScript syntax - control flow (`if`/`while`/`for`), string manipulation, math (`${Math.Calc[...]}`), wait commands, function declaration/calling, and comparisons - see the dedicated LavishScript reference:

- [01_LavishScript_Fundamentals.md - Conditional Branching](01_LavishScript_Fundamentals.md#conditional-branching)
- [01_LavishScript_Fundamentals.md - Loops](01_LavishScript_Fundamentals.md#loops)
- [01_LavishScript_Fundamentals.md - Wait Commands](01_LavishScript_Fundamentals.md#wait-commands)
- [01_LavishScript_Fundamentals.md - Functions](01_LavishScript_Fundamentals.md#functions)
- [01_LavishScript_Fundamentals.md - Return Values](01_LavishScript_Fundamentals.md#return-values)

The remaining sections of this file apply those fundamentals to ISXPantheon-specific tasks.

---

## Putting It All Together

Here's an example that demonstrates multiple core concepts on the real surface:

```lavishscript
;*****************************************************
; Core Concepts Demo Script
;*****************************************************

; Script-scoped variables
declare Running bool script TRUE
declare LastCheck int script 0

function main()
{
    ; Wait for ISXPantheon (async readiness gate)
    while !${ISXPantheon.IsReady}
        wait 10

    ; Register the HTTP response event
    Event[isxGames_onHTTPResponse]:AttachAtom[OnHTTP]

    echo "Core Concepts Demo Started (version ${ISXPantheon.Version})"

    ; Main loop
    while ${Running}
    {
        ; Periodic action every 2 seconds
        if ${Script.RunningTime} >= ${Math.Calc64[${LastCheck}+2000]}
        {
            call PeriodicTask
            LastCheck:Set[${Script.RunningTime}]
        }

        wait 10
    }
}

function PeriodicTask()
{
    ; NULL check (existence validation)
    if !${ISXPantheon(exists)}
        return

    ; Local variable for calculation
    variable int Rounded
    Rounded:Set[${ISXPantheon.Round[int,47,5]}]

    echo "Rounded value: ${Rounded}"
}

; Event atom handler
atom OnHTTP(int Size, string URL, string IPAddress, int ResponseCode, float TransferTime, string ResponseText, string ParsedBody)
{
    echo "HTTP response ${ResponseCode} from ${URL}"
}
```

---

## Summary of Core Concepts

1. **Datatypes** - Every object has a specific type with defined members/methods
2. **TLOs** - Global entry points (`${ISXPantheon}` today; additional game-data TLOs are planned)
3. **Inheritance** - Some datatypes inherit members from parent types
4. **Members vs Methods** - `.Member` gets values, `:Method` performs actions
5. **NULL Checks** - Always use `(exists)` before accessing objects
6. **Scoping** - Script vs local variables have different lifetimes
7. **Async Loading** - Wait for `${ISXPantheon.IsReady}`; for planned game data, check `Is*Available` first
8. **Collections** - Results stored in collections, accessed via iterators
9. **Events** - React to actions with atom handlers (HTTP today; game events planned)
