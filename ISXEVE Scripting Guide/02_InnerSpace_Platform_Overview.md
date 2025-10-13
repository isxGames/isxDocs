# InnerSpace Platform Overview
## The Foundation for Game Automation

**Purpose**: Comprehensive understanding of InnerSpace - the platform that hosts and executes EVE automation scripts
**Audience**: AI systems learning to develop InnerSpace-based bots
**Wiki Reference**: LavishScriptWiki for detailed InnerSpace documentation

---

## Table of Contents

1. [What Is InnerSpace](#what-is-innerspace)
2. [Architecture and Components](#architecture-and-components)
3. [LavishScript Engine](#lavishscript-engine)
4. [Extension System](#extension-system)
5. [Sessions and Multi-Client](#sessions-and-multi-client)
6. [Uplink and Relay](#uplink-and-relay)
7. [Script Execution Model](#script-execution-model)
8. [Console and Commands](#console-and-commands)
9. [File System and Paths](#file-system-and-paths)
10. [Performance and Limitations](#performance-and-limitations)

---

## What Is InnerSpace

### Core Purpose

**InnerSpace** is a **game automation platform** that:
- Hosts multiple game clients simultaneously
- Provides scripting engine (LavishScript)
- Enables memory reading from game processes
- Allows extensions to add game-specific APIs
- Facilitates inter-process communication (IPC)

**Critical Understanding**:
- InnerSpace is **NOT** a cheat engine or memory editor
- It is **NOT** injecting fake data into the game
- It **IS** reading game memory and issuing valid commands
- It **IS** a legitimate (though ToS-gray) automation tool

### The Ecosystem

```
InnerSpace (Host Platform)
    ↓
LavishScript (Scripting Engine)
    ↓
ISXEVE (EVE Online Extension)
    ↓
Your Bot Script (.iss file)
    ↓
EVE Online Client (Game)
```

**Flow of Control**:
1. InnerSpace launches EVE client
2. InnerSpace injects itself into EVE process
3. ISXEVE extension loads, reads EVE memory
4. Your script runs in LavishScript engine
5. Script uses ISXEVE to read game state
6. Script issues commands to EVE UI
7. EVE client sends commands to server

---

## Architecture and Components

### InnerSpace Components

**Core Modules**:
1. **Inner Space.exe** - Main executable, process manager
2. **LavishScript.dll** - Scripting engine
3. **Extensions/** - Game-specific plugins (ISXEVE, etc.)
4. **Scripts/** - User scripts and libraries
5. **Uplink/** - IPC server for relay communication

**Directory Structure**:
```
InnerSpace/
├── Inner Space.exe           # Main program
├── ISXStealth.dll            # Stealth module (hides from detection)
├── Extensions/
│   ├── ISXEVE.dll            # EVE Online extension
│   ├── ISXStealth.dll        # Anti-detection
│   └── [other game extensions]
├── Scripts/
│   ├── EVEBot/               # Evebot script
│   ├── Yamfa/                # Your fleet assist script
│   ├── Common/               # Shared libraries
│   └── [your scripts]
├── Uplink/                   # Relay server files
└── Settings/                 # Configuration files
```

### Process Model

**Single InnerSpace, Multiple Games**:
```
InnerSpace.exe (Parent Process)
    ├── eve.exe (Session 1 - Character A)
    ├── eve.exe (Session 2 - Character B)
    └── eve.exe (Session 3 - Character C)
```

**Critical Concepts**:
- Each EVE client = separate process
- Each process = separate "session" in InnerSpace
- InnerSpace coordinates all sessions
- Sessions can communicate via Relay

---

## LavishScript Engine

### What Is LavishScript

**LavishScript** is:
- **Scripting language** embedded in InnerSpace
- **Interpreted** (not compiled)
- **Game-agnostic** core with **game-specific extensions**
- **Event-driven** and **imperative**

**Language Characteristics**:
- Similar to C-style syntax (braces, semicolons)
- Dynamic typing (variables don't have fixed types)
- Object-oriented features (objects, methods)
- NO classes (objects are created via extensions or atoms)

**Example**:
```lavishscript
; This is a comment
variable int MyNumber = 42
variable string MyText = "Hello"

if ${MyNumber} > 40
{
    echo "Number is greater than 40"
}
```

### Script File Types

**File Extensions**:
- **.iss** - InnerSpace Script (main script files)
- **.xml** - Configuration files (settings, GUIs)

**Script Structure**:
```lavishscript
; Header: Check dependencies
#if !${ISXEVE(exists)}
    echo "ISXEVE extension required"
    Script:End
#endif

; Variables
variable bool Running = TRUE
variable int Counter = 0

; Functions
function DoSomething()
{
    echo "Doing something"
}

; Main entry point
function main()
{
    echo "Script starting"

    while ${Running}
    {
        call DoSomething
        wait 10
        Counter:Inc
    }

    echo "Script ending"
}
```

### Variable System

**Variable Declaration**:
```lavishscript
; Local variable
variable int LocalVar = 10

; Global variable (accessible across script scopes)
variable(global) int GlobalVar = 20

; Persistent variable (survives script reload)
variable(globalkeep) string PersistentVar = "Saved"

; Script variable (visible to other scripts via Script[scriptname])
variable(script) bool ScriptVar = TRUE
```

**Scope Rules**:
- Local = function/block scope
- Global = entire script scope
- Globalkeep = persists across script reloads
- Script = accessible via relay/uplink

### Data Types

**Primitive Types**:
```lavishscript
variable bool MyBool = TRUE              ; Boolean
variable int MyInt = 42                  ; 32-bit integer
variable int64 MyBigInt = 123456789      ; 64-bit integer
variable float MyFloat = 3.14            ; Floating point
variable string MyString = "Hello"       ; String
```

**Complex Types**:
```lavishscript
; Index (collection/array)
variable index:int MyIndex
MyIndex:Insert[10]
MyIndex:Insert[20]
echo "First element: ${MyIndex.Get[1]}"  ; 1-indexed!

; Iterator (for traversing collections)
variable iterator MyIterator
MyIndex:GetIterator[MyIterator]

; Point3f (3D coordinate)
variable point3f MyPoint
MyPoint:Set[100, 200, 300]

; Time (timestamp)
variable time MyTime = ${Time.Timestamp}
```

### Object System

**Everything Is An Object**:
```lavishscript
; Numbers are objects with methods
variable int MyNumber = 42
echo "Incremented: ${MyNumber.Inc}"  ; Returns 43

; Strings are objects
variable string MyString = "hello world"
echo "Uppercase: ${MyString.Upper}"  ; "HELLO WORLD"
echo "Length: ${MyString.Length}"    ; 11

; Even TRUE/FALSE are objects
variable bool MyBool = TRUE
echo "Negated: ${MyBool.Not}"  ; FALSE
```

**Object Members**:
- **Properties**: Values retrieved from object (e.g., `${MyString.Length}`)
- **Methods**: Actions performed by object (e.g., `MyString:Set["new value"]`)
- **Indices**: Access by index/key (e.g., `${MyIndex.Get[1]}`)

**Syntax**:
- **Get value**: `${Object.Property}` or `${Object.Method}`
- **Set value**: `Object:Method[parameters]`
- **Method call**: `Object:Method[]` (empty brackets for no params)

---

## Extension System

### What Are Extensions

**Extensions** are DLLs that:
- Add new object types to LavishScript
- Provide game-specific APIs
- Read game memory
- Expose game data as LavishScript objects

**ISXEVE Extension**:
- THE critical extension for EVE bots
- Provides 100+ object types
- Reads EVE client memory
- Exposes: ships, modules, entities, UI, etc.

**Loading Extensions**:
```lavishscript
; Check if extension loaded
#if ${ISXEVE(exists)}
    echo "ISXEVE is loaded"
#else
    echo "ISXEVE not loaded - exiting"
    Script:End
#endif
```

**Extension-Provided Objects**:
```lavishscript
; These objects come from ISXEVE extension:
${Me}                 ; Your character
${MyShip}             ; Your ship
${EVE}                ; EVE client object
${Entity[ID]}         ; Entities in space
${Station}            ; Current station
```

### Extension vs Script

**What Extensions Do**:
- Read memory directly
- Provide C++-speed performance
- Expose complex game structures
- Handle memory layout changes (patches)

**What Scripts Do**:
- Use extension-provided objects
- Implement bot logic
- Make decisions
- Issue commands

**Division of Labor**:
```
Extension (ISXEVE.dll)
    ↓ Provides Objects ↓
Script (YourBot.iss)
    ↓ Uses Objects ↓
Game (eve.exe)
```

---

## Sessions and Multi-Client

### Session Concept

**What Is A Session**:
- A **session** = one game client instance
- Each session has unique ID
- Sessions are numbered: is1, is2, is3, etc.
- Current session ID: `${Session}`

**Session Names**:
```lavishscript
; Current session
echo "Running in session ${Session}"

; Check if running in specific session
if ${Session.Equal["is1"]}
{
    echo "This is the master session"
}
```

### Multi-Boxing

**Common Pattern**:
- **Master** session controls logic
- **Slave** sessions follow master
- Communication via **Relay** system

**Example Setup**:
```
Session is1 (Master)
    - Makes targeting decisions
    - Broadcasts targets to slaves

Session is2 (Slave)
    - Listens for target broadcasts
    - Locks and shoots targets

Session is3 (Slave)
    - Listens for target broadcasts
    - Locks and shoots targets
```

**Session-Specific Scripts**:
```lavishscript
; Different behavior per session
if ${Session.Equal["is1"]}
{
    ; Master logic
    call MasterBehavior
}
else
{
    ; Slave logic
    call SlaveBehavior
}
```

---

## Uplink and Relay

### Uplink Service

**What Is Uplink**:
- **IPC server** built into InnerSpace
- Allows scripts to communicate across sessions
- Runs locally on your PC
- NOT internet-connected by default

**Relay = Uplink Communication**:
- "Relay" refers to sending data via Uplink
- Scripts can "relay" commands/data to each other
- Real-time, low-latency

### Relay System

**How It Works**:
```
Session is1 (Sender)
    ↓
    relay "all other" MyAtom "param1" "param2"
    ↓
Uplink Server
    ↓
Session is2, is3, is4 (Receivers)
    ↓
    Execute MyAtom function with parameters
```

**Relay Syntax**:
```lavishscript
; In sender script:
relay "all other" FunctionName "arg1" "arg2"

; This calls FunctionName in ALL other sessions
; With arguments "arg1" and "arg2"

; Specific session:
relay "is2" FunctionName "arg1"

; All except current:
relay "all other" FunctionName

; All including current:
relay "all" FunctionName
```

**Relay Receiver** (Atom):
```lavishscript
; Define an atom to receive relay
atom(script) FunctionName(string arg1, string arg2)
{
    echo "Received relay: ${arg1}, ${arg2}"
    ; Do something with the data
}
```

### Typical Relay Use Case: Fleet Assist

**Yamfa Example** (simplified):
```lavishscript
; MASTER session (is1):
function BroadcastTargets()
{
    ; Get my locked targets
    variable string TargetList = ""
    ; ... build target ID list ...

    ; Relay to all slaves
    relay "all other" ReceiveTargets "${TargetList}"
}

; SLAVE sessions (is2, is3, etc.):
atom(script) ReceiveTargets(string targetIDs)
{
    echo "Master says lock these targets: ${targetIDs}"
    ; Parse target IDs and lock them
}
```

**Why This Works**:
- Master makes targeting decisions (complex logic)
- Slaves just lock what master says (simple logic)
- No need for external communication (Redis, files, etc.)
- Real-time, same-machine communication

---

## Script Execution Model

### Running Scripts

**Launch Methods**:

**1. Console Command**:
```
run MyScript
```
- Opens console
- Types `run MyScript`
- Script executes

**2. Autostart**:
- Configure in InnerSpace settings
- Script auto-runs when session starts
- Good for production bots

**3. From Another Script**:
```lavishscript
; Start another script
Script[OtherScript]:Start[]

; End another script
Script[OtherScript]:End[]
```

### Script Lifecycle

**Execution Flow**:
```
Script Launched
    ↓
Global Variables Initialized
    ↓
main() Function Called (if exists)
    ↓
Script Runs (main loop, events, etc.)
    ↓
Script:End called OR main() returns
    ↓
Script Terminates
```

**Important**:
- **NO AUTOMATIC MAIN LOOP** - You create it!
- Script ends when it reaches end of file (unless in infinite loop)
- Must use `wait` commands to avoid CPU lockup

**Typical Script Structure**:
```lavishscript
; Init
function main()
{
    echo "Starting bot"
    call Initialize

    ; Main loop
    variable bool Running = TRUE
    while ${Running}
    {
        call Pulse
        wait 20  ; CRITICAL - prevents CPU maxing
    }

    call Shutdown
    echo "Bot ended"
}
```

### The Wait Command

**Critical Command**: `wait`

**Syntax**:
```lavishscript
wait <deciseconds>
```
- **decisecond** = 1/10 of a second = 100ms
- `wait 10` = wait 1 second
- `wait 5` = wait 0.5 seconds

**Why Wait Is Mandatory**:
```lavishscript
; BAD - NEVER DO THIS
while TRUE
{
    ; No wait - script runs MILLIONS of times per second
    ; Maxes CPU, crashes InnerSpace
}

; GOOD - ALWAYS DO THIS
while TRUE
{
    ; Process logic
    wait 10  ; Waits 1 second each loop iteration
}
```

**Typical Wait Times**:
- Main loop: `wait 10-20` (1-2 seconds)
- Combat loop: `wait 5-10` (0.5-1 seconds)
- Waiting for action: `wait 30-100` (3-10 seconds)

---

## Console and Commands

### InnerSpace Console

**What Is It**:
- Text console built into InnerSpace
- Appears as overlay in game client
- Receive script output (`echo` commands)
- Issue commands manually

**Opening Console**:
- Default hotkey: **Ctrl+Shift+I** (varies by config)
- Console appears over game

**Console Commands**:
```
run ScriptName           ; Start a script
end ScriptName           ; Stop a script
runscript path/to/file   ; Run script from path
echo Hello World         ; Print to console
ext                      ; List loaded extensions
relay all other uplink   ; Test relay communication
```

### Echo Command

**Output to Console**:
```lavishscript
echo "Hello from script"
echo "My ship: ${MyShip.ToEntity.Name}"

; Concatenation
variable int Value = 42
echo "The value is: ${Value}"
```

**Echo Is Your Debug Tool**:
```lavishscript
; Debug current state
echo "DEBUG: Current HP = ${MyShip.Shield}"
echo "DEBUG: Target locked? ${Target.IsLockedTarget}"
```

### Script Object

**Control Scripts**:
```lavishscript
; End current script
Script:End

; Pause script
Script:Pause

; Resume script
Script:Resume

; Check if another script running
if ${Script[EVEBot](exists)}
{
    echo "EVEBot is running"
}
```

---

## File System and Paths

### Script Paths

**Default Script Directory**:
```
InnerSpace/Scripts/
```

**Running Scripts**:
```lavishscript
; If file is at: InnerSpace/Scripts/MyBot/MyBot.iss
; Run with:
run MyBot/MyBot

; Or:
runscript MyBot/MyBot.iss
```

**Including Other Files**:
```lavishscript
; Include another script file
#include MyBot/Library/Targeting.iss
#include Common/Utilities.iss

; Now can use functions from those files
```

### File Operations

**Reading Files**:
```lavishscript
; Open file for reading
variable file MyFile = "Data/config.txt"
MyFile:Open[]

; Read line
variable string Line
if ${MyFile:Read[Line]}
{
    echo "Read: ${Line}"
}

MyFile:Close[]
```

**Writing Files**:
```lavishscript
; Open file for writing
variable file LogFile = "Logs/botlog.txt"
LogFile:Open[append]  ; Append mode

; Write line
LogFile:Write["Bot started at ${Time}\n"]

LogFile:Close[]
```

### Settings Files (XML)

**Persistent Configuration**:
```lavishscript
; Create settings object
variable(globalkeep) settingsetref MySettings
MySettings:Set[MyBot.Config, XML]

; Add settings
MySettings:Add[MasterCharacter, "CharacterName"]
MySettings:Add[FollowDistance, 1000]

; Save to file
MySettings:Save["Scripts/MyBot/config.xml"]

; Later: Load from file
MySettings:Load["Scripts/MyBot/config.xml"]

; Retrieve values
variable string Master = "${MySettings.Get[MasterCharacter]}"
```

---

## Performance and Limitations

### CPU and Memory

**Script Performance**:
- LavishScript is **interpreted** - slower than compiled C++/C#
- Extensions are **compiled C++** - fast
- Heavy computation should use extension features when possible

**Avoid**:
```lavishscript
; BAD - Querying thousands of entities every frame
while ${Running}
{
    variable index:entity AllEntities
    EVE:QueryEntities[AllEntities]  ; Expensive!
    ; Process...
    wait 5  ; Running 2x per second - too fast for this query
}
```

**Better**:
```lavishscript
; GOOD - Query at reasonable interval
while ${Running}
{
    variable index:entity AllEntities
    EVE:QueryEntities[AllEntities]  ; Expensive!
    ; Process...
    wait 20  ; Running every 2 seconds - reasonable
}
```

### Memory Limitations

**32-bit Process**:
- InnerSpace/LavishScript are 32-bit applications
- Limited to ~2GB RAM per process
- Running many sessions can exhaust memory

**Solutions**:
- Keep scripts lean
- Don't store huge data structures
- Clear old data periodically

### Network Latency

**Server Communication**:
- EVE is online game - server delays inevitable
- **Must design for latency** in bot logic
- Always use waits and timeouts

**Typical Latencies**:
- Local network: 10-50ms
- US to US: 50-100ms
- US to EU: 100-200ms
- High load: 200-500ms+

### Anti-Detection

**ISXStealth**:
- Extension that hides InnerSpace from game
- Prevents game from detecting memory reading
- **Not perfect** - CCP can still detect patterns

**Bot Detection Risks**:
- Repetitive behavior (exact same timings)
- Superhuman reaction times
- 24/7 operation
- Obvious patterns (warp, kill, loot, repeat)

**Mitigation**:
- Add randomness to timings
- Vary behavior slightly
- Don't bot 24/7
- Act human-like

---

## Critical Concepts Summary

**Key Understandings**:

1. **InnerSpace = Platform**
   - Hosts game clients
   - Provides scripting engine
   - Enables IPC via relay

2. **LavishScript = Language**
   - Interpreted scripting language
   - Object-oriented
   - Dynamic typing
   - Extension-based

3. **ISXEVE = Extension**
   - Game-specific API
   - Reads EVE memory
   - Exposes game objects to scripts

4. **Sessions = Instances**
   - Each game client = one session
   - Sessions can communicate via relay
   - Multi-boxing uses multiple sessions

5. **Wait = Mandatory**
   - Must use `wait` in loops
   - Prevents CPU maxing
   - Accounts for server latency

6. **Relay = IPC**
   - Inter-script communication
   - Real-time, same-machine
   - Critical for fleet coordination

7. **Scripts vs Extensions**
   - Scripts = logic (your code)
   - Extensions = API (game access)
   - Extensions are fast, scripts are flexible

8. **Execution Model**
   - No automatic main loop
   - You create the loop
   - Script ends when code ends

---

## Next Steps

**Now you understand the platform. Next:**
- Read `03_ISXEVE_Extension_Architecture.md` to learn how the EVE API works
- Read `04_LavishScript_Language_Complete_Reference.md` for deep language knowledge
- Then move to practical scripting fundamentals

**You should be able to answer**:
- What is InnerSpace's role?
- What is LavishScript's role?
- What is ISXEVE's role?
- How do sessions communicate?
- Why must scripts use `wait`?
- What's the difference between scripts and .NET programs?

**Script vs .NET Teaser**:
- **Scripts**: `.iss` files, run via `run ScriptName`
- **.NET Programs**: `.exe`/`.dll` files, run via `dotnet ProgramName`
- Both use InnerSpace/ISXEVE, different execution models
- More detail in file `32_Scripts_vs_DotNet_Programs.md`

---

**END OF FILE**
**Next File**: 03_ISXEVE_Extension_Architecture.md
