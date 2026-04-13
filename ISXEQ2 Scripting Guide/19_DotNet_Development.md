# .NET Development for ISXEQ2

## Scripts vs .NET Programs

This guide compares the two supported development models for ISXEQ2 automation: plain LavishScript scripts (`.iss` text files, interpreted) and compiled .NET programs (`.exe` / `.dll` assemblies written in C#/VB.NET/F#, loaded via `dotnet <program>` inside InnerSpace). It is a decision-and-orientation guide, not an API reference — for the full .NET wrapper surface see the wrapper's own help file at `ISXEQ2Wrapper/ISXEQ2Wrapper.chm`.

The ISXEQ2 wrapper is shipped as `ISXEQ2Wrapper.dll` with root namespace `EQ2.ISXEQ2`. 

## Table of Contents
1. [Fundamental Differences](#fundamental-differences)
2. [LavishScript Scripts (.iss files)](#lavishscript-scripts-iss-files)
3. [.NET Programs (.exe/.dll files)](#net-programs-exedll-files)
4. [Language Comparison](#language-comparison)
5. [Development Workflow](#development-workflow)
6. [Performance Comparison](#performance-comparison)
7. [Debugging Capabilities](#debugging-capabilities)
8. [Deployment and Distribution](#deployment-and-distribution)
9. [Maintenance and Updates](#maintenance-and-updates)
10. [Hybrid Approaches](#hybrid-approaches)
11. [Migration Strategies](#migration-strategies)
12. [Decision Matrix](#decision-matrix)
13. [ISXEQ2 .NET Wrapper Reference](#isxeq2-net-wrapper-reference)
14. [Comparison Table](#comparison-table)
15. [Summary](#summary)

---

## Fundamental Differences

### Scripts (.iss files)

**What They Are:**
- Plain text files containing LavishScript code
- Interpreted at runtime by the LavishScript engine
- Executed in InnerSpace via `run <scriptname>`

**Technology:**
- **Language:** LavishScript
- **Execution:** Interpreted
- **Compilation:** None
- **Runtime:** LavishScript engine (C++)

**Examples from the ISXEQ2 ecosystem:**
- [EQ2Bot](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Bot) — full-featured combat bot (main script + 20+ class routines + 39 UI files)
- [EQ2Craft](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Craft) — tradeskill automation
- [EQ2Navigation](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Navigation) — navigation/pathfinding library (`EQ2Nav`)
- [MyPrices](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/MyPrices) — broker automation

### .NET Programs (.exe/.dll)

Compiled .NET assemblies (C#, VB.NET, or F#) loaded into InnerSpace via `dotnet <program>`. See [.NET Programs (.exe/.dll files)](#net-programs-exedll-files) below for characteristics and development cycle.

---

## LavishScript Scripts (.iss files)

### Characteristics

- **No compilation** — edit and run immediately; instant iteration cycle
- **ISXEQ2 native** — the language ISXEQ2 was designed for; all TLOs, datatypes, commands, and events are LavishScript-first
- **Simple deployment** — just copy `.iss` text files
- **Slower execution** — interpreted (not compiled); debugging is echo/logs only

For the complete feature-by-feature comparison, see the [Comparison Table](#comparison-table) later in this guide.

### Structure Example (simple single-file script)

```
MyBot.iss                          ; Single file script

Contents:
- Variable declarations
- objectdef definitions
- Function/atom definitions
- Main execution loop
- Event handlers

Execution: run MyBot
```

### Structure Example (EQ2Bot)

```
EQ2Bot/
├── EQ2Bot.iss                     ; Main entry point
├── EQ2BotLib.iss                  ; Shared library
├── Class Routines/
│   ├── Guardian.iss               ; Class-specific rotations
│   ├── Warlock.iss
│   └── ...                        ; 20+ class files
├── UI/
│   ├── EQ2Bot.xml                 ; LGUI1 UI
│   └── ...                        ; 39 UI files
└── Profiles/
    └── CharacterName.xml          ; Per-character settings

Execution: run EQ2Bot
```

### Development Cycle

The full edit → save → run → debug iteration cycle for LavishScript development is documented in the [Development Workflow](#development-workflow) section below.

---

## .NET Programs (.exe/.dll files)

### Characteristics

- **Compiled to IL, JIT-compiled at runtime** — significantly faster than interpreted LavishScript for CPU-bound work
- **Full IDE support** — Visual Studio / Rider with IntelliSense, breakpoints, watch windows, refactoring
- **Type safety** — compile-time error detection catches whole categories of bugs before the game ever starts
- **Compilation required** — edit → build → copy → run cycle is slower than LavishScript's immediate-iteration model

For the complete feature-by-feature comparison, see the [Comparison Table](#comparison-table) later in this guide.

### Structure Example (generic .NET bot layout)

```
MyBot/
├── MyBot/                         ; Main application
│   ├── ActionModules/
│   │   ├── Movement.cs            ; Movement logic
│   │   ├── Offensive.cs           ; Ability/spell control
│   │   ├── Defense.cs             ; Tank/heal management
│   │   └── Targeting.cs           ; Target acquisition
│   ├── BehaviorModules/
│   │   ├── Combat.cs              ; Combat behavior
│   │   ├── Crafting.cs            ; Crafting behavior
│   │   └── QuestRunner.cs         ; Quest automation
│   ├── Core/
│   │   ├── CharacterState.cs      ; Cached Me state
│   │   ├── ActorCache.cs          ; Actor caching
│   │   └── AbilityQueue.cs        ; Cast queue
│   ├── Program.cs                 ; Entry point
│   └── MyBot.csproj               ; Project file
├── MyBot.Core/
│   ├── Loader.cs                  ; App initialization
│   └── Interfaces/
│       ├── IActorProvider.cs
│       └── IAbilityQueue.cs
└── MyBot.sln                      ; Solution file

Execution: dotnet MyBot
```

> **Note:** The author is not aware of a high-profile open-source ISXEQ2 .NET bot comparable to ISXEVE's Metatron2. Most active ISXEQ2 automation is in LavishScript. The layout above is illustrative of conventional .NET project organization; it is not based on a specific existing bot.

### Development Cycle

The full edit → build → run → debug iteration cycle for .NET development is documented in the [Development Workflow](#development-workflow) section below.

---

## Language Comparison

### Type Systems

**LavishScript:**
```lavishscript
; Weakly typed - runtime type checking
variable string myString = "hello"
variable int myInt = 123
variable float myFloat = 45.67

; Type errors caught at runtime
myInt:Set["not a number"]    ; Runtime error
```

**C# (.NET):**
```csharp
// Strongly typed - compile-time checking
string myString = "hello";
int myInt = 123;
float myFloat = 45.67f;

// Type errors caught at compile time
myInt = "not a number";    // Compiler error
```

### Object-Oriented Programming

**LavishScript:**
```lavishscript
; Basic OOP - objectdef
objectdef obj_Combat
{
    variable string Name
    variable float HealthPct

    method Initialize(string name)
    {
        This.Name:Set[${name}]
    }

    member:float GetHealth()
    {
        return ${Me.CurrentHealth}
    }

    method CastHeal()
    {
        Me.Ability["Minor Healing"]:Use
    }
}

; Usage
variable obj_Combat MyCombat
MyCombat:Initialize["Guardian"]
```

**C# (.NET):**
```csharp
// Full OOP - class, interface, inheritance
using EQ2.ISXEQ2;
using EQ2.ISXEQ2.CharacterActor;

public class Combat
{
    public string Name { get; set; }
    public float HealthPct { get; private set; }

    public Combat(string name)
    {
        Name = name;
    }

    public float GetHealth()
    {
        Character me = Extension.Me();
        return me.CurrentHealth;
    }

    public void CastHeal()
    {
        // Issue a game command through the LavishScript engine
        Extension.EQ2Execute("useability", "Minor Healing");
    }
}

// Usage
var myCombat = new Combat("Guardian");
```

### Collections

**LavishScript:**
```lavishscript
; Limited collection types
variable index:actor Actors
variable collection:int ActorIDs
variable queue:string Messages

; Usage
EQ2:GetActors[Actors, Range, 50, NPC]
ActorIDs:Set[${ID}, 1]
Messages:Queue["Hello"]
```

**C# (.NET):**
```csharp
// Rich collection library
using System.Collections.Generic;
using System.Linq;
using EQ2.ISXEQ2.CharacterActor;

List<Actor> actors = new List<Actor>();
Dictionary<int, int> actorIDs = new Dictionary<int, int>();
Queue<string> messages = new Queue<string>();

// LINQ support
var hostiles = actors.Where(a => a.IsAggro).OrderBy(a => a.Distance);
```

### Error Handling

**LavishScript:**
```lavishscript
; Limited error handling - no try/catch
if !${Actor[${actorID}](exists)}
{
    echo "Error: Actor doesn't exist"
    return
}
```

**C# (.NET):**
```csharp
// Full exception handling
try
{
    Actor actor = Extension.Actor(actorID);
    actor.DoTarget();
}
catch (Exception ex)
{
    LogError($"Failed to target actor: {ex.Message}");
}
finally
{
    // Cleanup
}
```

### Async Programming

**LavishScript:**
```lavishscript
; No async/await - use blocking waits
Me.Ability["Mend Wounds"]:Use
wait 20 ${Me.CastingSpell}
wait 50 !${Me.CastingSpell}
```

**C# (.NET):**
```csharp
// Async/await support
public async Task<bool> CastAndWaitAsync(string abilityName)
{
    Extension.EQ2Execute("useability", abilityName);

    // Non-blocking wait
    await Task.Delay(200);

    return !Extension.Me().CastingSpell;
}

var result = await CastAndWaitAsync("Mend Wounds");
```

---

## Development Workflow

### Script Development (LavishScript)

**Tools:**
- **Editor:** Notepad++, VS Code, Sublime Text
- **Testing:** In-game only
- **Debugging:** `echo` statements, log files
- **Version Control:** Manual copies or Git

**Workflow:**
```
Edit -> Save -> Run -> Test -> Debug (echo) -> Repeat
```

**Pros:**
- Immediate feedback
- Simple tools
- No build step
- Easy sharing (copy `.iss` files)

**Cons:**
- Limited syntax highlighting
- No code completion
- Manual debugging only
- No refactoring tools

### .NET Development

**Tools:**
- **IDE:** Visual Studio (Community/Professional/Enterprise), JetBrains Rider, VS Code + C# Dev Kit
- **Testing:** In-game + unit tests (NUnit, xUnit, MSTest)
- **Debugging:** Full debugger — breakpoints, watches, stack traces, immediate window
- **Version Control:** Integrated Git

**Workflow:**
```
Edit -> Save -> Build -> Copy DLL -> Run -> Test -> Debug (breakpoints) -> Repeat
```

**Pros:**
- Syntax highlighting, IntelliSense, auto-complete
- Refactoring tools
- Advanced debugging
- Unit testing
- Rich ecosystem (NuGet)

**Cons:**
- Build time (seconds per build)
- More complex project setup
- Deployment overhead (multiple DLLs)
- Steeper learning curve

---

## Performance Comparison

### Execution Speed

**LavishScript:**
- Interpreted — substantially slower than compiled .NET for pure computation
- No JIT optimization
- String operations especially slow
- Per-TLO queries reasonable (native ISXEQ2 C++ implementation)

**.NET:**
- JIT-compiled to native code
- Order-of-magnitude faster for tight loops and CPU-bound work
- First-call JIT overhead amortized quickly

### Memory Usage

**LavishScript:**
- Low baseline for simple scripts
- Higher for complex multi-file bots (e.g. EQ2Bot)
- Limited GC — long-running scripts' footprint can grow over time

**.NET:**
- Baseline CLR overhead added on top of the hosted process
- Generational GC keeps heap bounded under steady-state load
- Figures vary widely by workload — measure, don't assume

### Startup Time

**LavishScript:**
- Near-instant — parse text, run

**.NET:**
- Noticeably slower than scripts
- CLR initialization, assembly loading, and JIT all add overhead
- Exact timing varies by application size

---

## Debugging Capabilities

### Script Debugging

**Available Tools:**
```lavishscript
; echo statements
echo "Current state: ${This.CurrentState}"
echo "Target: ${Actor[${actorID}].Name}"

; Log files
LavishScript:Echo["Debug: Processing actor ${actorID}"]

; Variable inspection (manual)
method DebugDump()
{
    echo "=== Debug ==="
    echo "State: ${This.CurrentState}"
    echo "Health: ${Me.CurrentHealth}/${Me.MaxHealth}"
    echo "InCombat: ${Me.InCombat}"
    echo "============="
}
```

**Limitations:**
- No breakpoints
- No watch windows
- No stack traces
- No step-through
- Manual variable dumping

### .NET Debugging

**Breakpoints:**
```csharp
public void ProcessActor(int actorID)
{
    // Set breakpoint (F9)
    Actor actor = Extension.Actor(actorID);

    // Execution pauses; inspect actor, actorID, locals, call stack
}
```

**Watch Windows:**
```
Watches:
- actor.Name
- actor.Distance
- actor.IsLocked
- CastQueue.Count
```

**Immediate Window:**
```csharp
// Execute expressions while paused
> actor.DoTarget()
> Console.WriteLine(actor.Name)
```

**Exception Helper:**
```
Unhandled exception at line 245:
System.NullReferenceException: Object reference not set...

Stack trace:
  at MyBot.ActionModules.Targeting.ProcessActor(Int32 actorID) line 245
  at MyBot.BehaviorModules.Combat.Pulse() line 89
  at MyBot.Core.ModuleBase.Execute() line 34
```

---

## Deployment and Distribution

### Script Deployment

**Steps:**
1. Copy `.iss` files to the InnerSpace Scripts directory
2. Copy any config/data files
3. Run via `run ScriptName`

**Distribution:**
```
YourBot/
├── YourBot.iss
├── Lib/
│   ├── obj_Config.iss
│   └── obj_Logger.iss
└── README.txt

Zip and share.
```

**Pros:**
- Dead simple
- No extra dependencies (ISXEQ2 assumed present)
- Users can read/modify the source

**Cons:**
- Source is visible (if that matters)
- No source protection

### .NET Deployment

**Steps:**
1. Build solution in Release
2. Copy output DLLs
3. Copy dependencies (`ISXEQ2Wrapper.dll`, any NuGet deps)
4. Copy config files
5. Test on target machine

**Distribution:**
```
MyBot/
├── MyBot.exe
├── MyBot.Core.dll
├── Newtonsoft.Json.dll         ; Example NuGet dependency
├── ISXEQ2Wrapper.dll           ; ISXEQ2 .NET wrapper
├── LavishScriptAPI.dll         ; InnerSpace / LavishScript .NET API
└── Config/
    └── Settings.xml

Zip and share.
```

**Pros:**
- Compiled (source not directly readable)
- Professional packaging
- Versioned assemblies

**Cons:**
- Multiple files to ship
- Dependency/version management
- Harder for users to tweak

---

## Maintenance and Updates

### Script Maintenance

**Updating:**
```
1. Edit .iss file
2. Save
3. Users download new .iss
4. Replace old file
5. Restart script
```

**Version Management:**
```lavishscript
variable string VERSION = "2.5"

method Initialize()
{
    echo "YourBot v${VERSION} starting"
}
```

**Pros:** Instant updates, diffable, easy to patch.
**Cons:** No automatic updates, users must manually download.

### .NET Maintenance

**Updating:**
```
1. Edit .cs files
2. Save and build
3. Test build
4. Create release package
5. Users download new DLLs
6. Replace DLLs
7. Restart program
```

**Version Management:**
```csharp
// Modern SDK-style .csproj:
//   <PropertyGroup>
//     <Version>2.5.0.0</Version>
//   </PropertyGroup>

var version = Assembly.GetExecutingAssembly().GetName().Version;
Console.WriteLine($"MyBot v{version}");
```

**Auto-Update Sketch:**
```csharp
public class AutoUpdater
{
    public async Task<bool> CheckForUpdates()
    {
        // Download version manifest
        // Compare versions
        // Download new DLLs
        // Replace on restart
        return true;
    }
}
```

**Pros:** Professional versioning, auto-update possible, release management.
**Cons:** Complex update process, must replace all DLLs, more potential for breakage.

---

## Hybrid Approaches

### Calling .NET from Scripts

**Pattern:** A .NET program, launched via `dotnet <program>`, registers LavishScript-visible objects that scripts can then access. The .NET side defines LavishScript-exposed types using the InnerSpace .NET API; the script side uses them exactly like any other LavishScript object.

```csharp
// .NET program (started from LavishScript with: dotnet MyUtilities)
//
// Inside the program, register a LavishScript object that exposes members
// and methods. Scripts then see it as ${MyUtilities.something}.
//
// Consult the InnerSpace .NET API documentation for the current
// registration API — the exact type/method names have changed across
// InnerSpace versions, so the authoritative reference is Lavish's own
// docs, not this guide.
```

```lavishscript
; LavishScript side: start the .NET program, then use whatever TLO/object
; it registers. Example assumes the program registers a "MyUtilities" TLO.
dotnet MyUtilities

echo ${MyUtilities.SomeMember}
```

**Use Cases:**
- Complex calculations offloaded to .NET
- Use .NET libraries (JSON parsing, HTTP requests, crypto)
- Leverage C# performance for specific bottlenecks

> **Note:** There is no generic `${DotNet.Invoke[...]}` TLO. To call .NET code from LavishScript you must run a .NET program that registers its own LavishScript-visible object model. See the InnerSpace .NET API docs for the current registration mechanism.

### Calling Scripts from .NET (ISXEQ2 interop model)

**⚠ Important — ISXEQ2 differs from ISXEVE here.** ISXEVE uses a universal `EVE.Execute(ExecuteCommand)` method backed by a large `ExecuteCommand` enum. **ISXEQ2 does NOT.** There is no `ExecuteCommand` enum and no universal `Execute(enum)` method anywhere in `ISXEQ2Wrapper`. If you're cross-referencing ISXEVE .NET examples, this section is where the two diverge.

The ISXEQ2 interop model has **three layers**:

**1. Read-side: `LavishScript.Objects.GetObject(...)` for TLOs**

This is identical to ISXEVE. Every wrapper class internally calls `LavishScript.Objects.GetObject("TLOName", ...args)` to obtain a handle. For example, from the wrapper's `Extension` class (paraphrased):

```csharp
// Inside ISXEQ2Wrapper/Extension.cs
return new Character(LavishScript.Objects.GetObject("Me"));
return new Actor(LavishScript.Objects.GetObject("Actor", id.ToString(CultureInfo.InvariantCulture)));
return new Utility.EQ2(LavishScript.Objects.GetObject("EQ2"));
return new Zone(LavishScript.Objects.GetObject("Zone"));
```

You can use `LavishScript.Objects.GetObject(...)` directly for anything the wrapper doesn't already expose.

**2. Action-side Layer A: `Extension` static helpers**

`EQ2.ISXEQ2.Extension` (in `Extension.cs`) is a `public static class` with thin wrappers over specific LavishScript commands. These are the closest ISXEQ2 equivalent to ISXEVE's `ExecuteCommand` enum — but they're individual static methods, not enum values.

Verified signatures from the wrapper source:

```csharp
public static int EQ2Execute(params string[] command);         // Runs: EQ2Execute <args>
public static int EQ2Press(string args);                       // Runs: EQ2Press <args>
public static int Broker(string name);                         // Runs: broker name <name>
public static int Face();                                      // Runs: Face
public static int Face(ActorType type);                        // Runs: Face <type>
public static int Face(float heading);                         // Runs: Face <heading>
public static int Face(float x, float z);                      // Runs: Face <x> <z>
public static int Face(float x, float y, float z);             // Runs: Face <x> <y> <z>
public static int Face(string name);                           // Runs: Face <name>
public static int Radar(RadarCommand command, string name = null);
public static int Target(TargetType type);                     // Runs: Target <type>
public static int Target(int id);                              // Runs: Target <id>
public static int Target(string name);                         // Runs: Target <name>
```

And TLO getters (all return wrapper objects):

```csharp
public static Character Me();
public static Actor Actor(int id);
public static Actor Actor(string search);
public static Actor Target();
public static Zone Zone();
public static Utility.EQ2 EQ2();
public static Utility.ISXEQ2 ISXEQ2();
public static Radar Radar(int index);
public static Radar Radar(string name);
// ...plus EQ2Mail, EQ2Loc, Vendor, LootWindow, ChoiceWindow, ReplyDialog,
//    RewardWindow, ExamineItemWindow, EQ2UIPage, EQ2DataSourceContainer, Achievement
```

**3. Action-side Layer B: domain methods on wrapper classes**

Many actions live as instance methods on specific domain wrapper classes rather than on `Extension`. For example, `EQ2.ISXEQ2.Utility.EQ2` (the wrapper for the `${EQ2}` TLO) exposes:

```csharp
public bool AcceptPendingQuest();
public bool AcceptReward();
public bool ConfirmZoneTeleporterDestination();
public bool DeclinePendingQuest();
public IEnumerable<Actor> GetActors(params string[] args);
public IEnumerable<string> GetPersistentZones();
public bool SetAmbientLight(float ambientPct);
public bool SetMasterVolume(float volPct);
public bool ShowAllOnScreenAnnouncements();
// ...and more
```

Similarly, `Character`, `Actor`, `Item`, `Vendor`, `BrokerWindow`, etc. each expose their own methods.

**Putting it together:**

```csharp
using EQ2.ISXEQ2;
using EQ2.ISXEQ2.CharacterActor;
using EQ2.ISXEQ2.Utility;

public class ScriptInterop
{
    public void TargetAndAttack(int actorId)
    {
        // Get a handle to an actor via Extension TLO getter
        Actor actor = Extension.Actor(actorId);
        if (actor == null) return;

        // Read-side: properties map to LavishScript members
        if (actor.Distance > 30.0f) return;

        // Action-side Layer A: Extension static helper wraps a game command
        Extension.Target(actorId);

        // Action-side Layer B: instance method on a domain wrapper
        actor.DoTarget();
        actor.DoubleClick();
    }

    public void AcceptQuestIfPending()
    {
        EQ2 eq2 = Extension.EQ2();
        if (eq2.PendingQuestName != null)
        {
            eq2.AcceptPendingQuest();
        }
    }

    public void RunArbitraryEq2Command()
    {
        // The universal escape hatch: EQ2Execute wraps any EQ2 command
        Extension.EQ2Execute("useability", "Minor Healing");
        Extension.EQ2Execute("useabilityonplayer", "Heal", "PartyMember1");
    }
}
```

**Enums used by these APIs (all domain-scoped, not universal):**

| Enum | File | Used by |
|------|------|---------|
| `ActorType` | `Extension.cs` | `Face(ActorType)` |
| `AnnouncementSound` | `Extension.cs` | `EQ2Announce` helpers |
| `ChatType` | `Extension.cs` | Character chat methods |
| `EQ2MailStatus` | `Extension.cs` | `EQ2Mail` lookups |
| `EquipSlot` | `Extension.cs` | Equipment-slot addressing |
| `RadarCommand` | `Extension.cs` | `Radar(RadarCommand, name)` |
| `TargetType` | `Extension.cs` | `Target(TargetType)` |
| `CommandType` | `CharacterActor/Actor.cs` | Actor-specific commands |
| `EffectType` | `CharacterActor/Character.cs` | Effect queries |
| `CoinType` | `CharacterActor/Character.cs` | Money conversion |
| `InvType` | `CharacterActor/Character.cs` | Inventory queries |
| `ElementType` | `UI/EQ2UIElement.cs` | UI element types |
| `BenefitToggle` | `Utility/ISXEQ2.cs` | ISXEQ2 benefit toggles |
| `NextFreeType` | `InventoryConsignment/Item.cs` | Free-slot lookup |

> **Note on arbitrary command strings:** Invoking raw LavishScript command strings (e.g. `run`, `endscript`) from .NET uses the LavishScript-engine-level `LavishScript.ExecuteCommand(...)` helper from `LavishScriptAPI.dll` — that's what every `Extension` helper above is a thin wrapper around. Use it directly if you need to fire a command the wrapper doesn't already expose.

**Use Cases:**
- .NET app orchestrating scripts
- .NET reading live script state
- Mixed architecture

---

## Migration Strategies

### Script to .NET Migration

**When to Migrate:**
- Performance becomes critical
- Need advanced language features (async/await, LINQ, generics)
- Want professional tooling
- Planning commercial distribution or team development

**Migration Path:**

**Phase 1: Planning**
```
1. Document script architecture
2. Identify core objects/functions
3. Map to .NET classes
4. Plan module structure
```

**Phase 2: Foundation**
```
1. Create .NET project (net4.x or modern .NET as appropriate for your InnerSpace build)
2. Reference ISXEQ2Wrapper.dll and LavishScriptAPI.dll
3. Implement core classes (CharacterState, ActorCache, etc.)
4. Port configuration system
```

**Phase 3: Incremental Port**
```
1. Port one behavior at a time
2. Test each behavior in-game
3. Run script alongside .NET during transition
4. Gradually shift functionality to .NET
```

**Phase 4: Completion**
```
1. Port all behaviors
2. Retire script
3. Polish .NET app
4. Package for distribution
```

**Example Migration:**

```lavishscript
; Original Script: obj_Combat
objectdef obj_Combat
{
    variable string CurrentState = "IDLE"

    method Pulse()
    {
        This:SetState
        This:ProcessState
    }

    method SetState()
    {
        ; ... state logic ...
    }

    method ProcessState()
    {
        ; ... processing logic ...
    }
}
```

```csharp
// .NET Port: Combat.cs
// (ModuleBase / ShouldPulse here is hypothetical — your own bot would
// supply its own base class. The ISXEQ2 wrapper doesn't impose a bot
// framework; it just exposes the ISXEQ2 API surface.)
namespace MyBot.BehaviorModules
{
    public class Combat : ModuleBase
    {
        private string _currentState = "IDLE";

        public override void Pulse()
        {
            if (!ShouldPulse()) return;

            SetState();
            ProcessState();
        }

        private void SetState()
        {
            // ... state logic ...
        }

        private void ProcessState()
        {
            // ... processing logic ...
        }
    }
}
```

---

## Decision Matrix

### When to Use Scripts

**Use LavishScript Scripts When:**

- [ ] Prototyping or proof-of-concept
- [ ] Simple automation (< 1000 lines)
- [ ] Learning ISXEQ2 / EQ2 automation
- [ ] Need rapid iteration
- [ ] Sharing with the ISXEQ2 LavishScript community (by far the larger pool)
- [ ] Don't know C#/.NET
- [ ] Want simple deployment

**Examples:** simple hauler, loot logger, AFK alarm, specific-quest helper, crafting helper.

### When to Use .NET

**Use .NET Programs When:**

- [ ] Complex logic (> 5000 lines)
- [ ] Performance-critical computation (pathfinding, big queries, cryptography)
- [ ] Need advanced features (async/await, LINQ, web APIs, third-party libs)
- [ ] Professional/commercial project
- [ ] Team development
- [ ] Long-term maintenance
- [ ] Want full debugging tools
- [ ] Already fluent in C#

**Examples:** full-featured combat bot, trading/broker analyzer with external APIs, multi-character fleet coordinator with database backend.

### Hybrid Approach

**Use Hybrid (Scripts + .NET) When:**

- [ ] Want best of both worlds
- [ ] Performance-critical algorithms need .NET, orchestration lives in scripts
- [ ] Rapid prototyping in scripts, production-grade pieces in .NET
- [ ] Incremental migration strategy

**Example split:**
```
Main bot / orchestration: .NET
Per-zone quick tweaks, config: .iss scripts
Heavy computation (pathfinding, market analysis): .NET
UI: LGUI2 JSON + small .iss glue
```

---

## ISXEQ2 .NET Wrapper Reference

### Assembly and Namespace

| Property | Value |
|----------|-------|
| Assembly | `ISXEQ2Wrapper.dll` |
| Root namespace | `EQ2.ISXEQ2` |
| Source repo | `ISXEQ2Wrapper/` (shipped alongside ISXEQ2) |
| API reference | `ISXEQ2Wrapper/ISXEQ2Wrapper.chm` |
| Project file | `ISXEQ2Wrapper/EQ2.ISXEQ2.csproj` |

### Sub-namespace layout

| Sub-namespace | Contents (representative) |
|---------------|---------------------------|
| `EQ2.ISXEQ2` | `Extension` static class (TLO getters + command helpers), shared enums |
| `EQ2.ISXEQ2.AbilityEffect` | `Ability`, `AbilityEffect`, `Achievement`, `Class`, `Effect`, `Maintained` |
| `EQ2.ISXEQ2.CharacterActor` | `Character`, `Actor` |
| `EQ2.ISXEQ2.Events` | `EQ2Event` and event wrappers |
| `EQ2.ISXEQ2.Helpers` | `LavishScriptObjectExtensions`, `Utils` |
| `EQ2.ISXEQ2.InventoryConsignment` | `Item`, `Vendor`, `BrokerWindow`, `Consignment`, etc. |
| `EQ2.ISXEQ2.Recipe` | `Recipe`, `Component` |
| `EQ2.ISXEQ2.UI` | `EQ2UIElement`, `EQ2UIPage`, `EQ2Window`, window wrappers |
| `EQ2.ISXEQ2.Utility` | `EQ2`, `ISXEQ2`, `Zone`, `EQ2Location`, `EQ2Mail`, `Radar` |

### Canonical interop patterns (summary)

- **Get a TLO handle:** `LavishScript.Objects.GetObject("TLOName", ...args)` (or use an `Extension` getter).
- **Read a member:** wrapper property (e.g. `Me.CurrentHealth`) or `handle.GetMember<T>("MemberName")` for anything not wrapped.
- **Run an EQ2 command:** `Extension.EQ2Execute("cmd", "arg1", "arg2", ...)` or a domain-specific helper like `Extension.Face(...)`, `Extension.Target(...)`, `Extension.Radar(...)`.
- **Run any LavishScript command:** `LavishScript.ExecuteCommand("cmd args...")` directly from `LavishScriptAPI.dll`.

---

## Comparison Table

| Feature | LavishScript Scripts | .NET Programs |
|---------|----------------------|---------------|
| Language | LavishScript | C#, VB.NET, F# |
| Execution | Interpreted | Compiled (JIT) |
| Performance | Slow for CPU-bound work | Fast (native-speed JIT) |
| Startup | Near-instant | Slower (CLR init + JIT) |
| IDE support | Basic editors | Visual Studio / Rider (full) |
| Debugging | echo / logs only | Breakpoints, watches, stack traces |
| Type safety | Runtime | Compile-time |
| IntelliSense | None | Full |
| Unit testing | None | Full (NUnit, xUnit, MSTest) |
| Async/await | No | Yes |
| LINQ | No | Yes |
| Error handling | Limited | Full try/catch/finally |
| Build time | None | Seconds |
| Deployment | Copy `.iss` files | Copy DLL bundle |
| Updates | Replace `.iss` | Replace DLLs |
| Learning curve | Easy | Moderate to hard |
| Code sharing | Easy (text files) | Harder (binaries) |
| Source protection | None | Compiled |
| Best for | Simple bots, prototypes | Complex bots, production |

---

## Summary

### Key Takeaways

1. **Scripts are simple.** Instant gratification, low barrier to entry, ideal for learning and for small-to-medium bots. The overwhelming majority of public ISXEQ2 automation ([EQ2Bot](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Bot), [EQ2Craft](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Craft), [EQ2Navigation](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Navigation), [MyPrices](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/MyPrices)) is LavishScript-based.

2. **.NET is powerful.** Full debugger, LINQ, async, unit tests, and arbitrary third-party libraries. Better for complex, performance-critical, or commercially-maintained projects.

3. **ISXEQ2's .NET interop model is NOT the same as ISXEVE's.** There is no universal `Execute(ExecuteCommand)` method and no `ExecuteCommand` enum. Use:
   - `LavishScript.Objects.GetObject(...)` for TLO handles,
   - `Extension.*` static helpers for common game commands,
   - instance methods on specific wrapper classes (`Character`, `Actor`, `Utility.EQ2`, `Item`, `Vendor`, ...) for domain actions,
   - `Extension.EQ2Execute(...)` or `LavishScript.ExecuteCommand(...)` as the general-purpose escape hatches.

4. **Choose based on need.** Simple bot or learning? Script. Complex/production/performance-critical? .NET. Can't pick? Hybrid — orchestrate in scripts, compute in .NET.

5. **Definitive API reference:** `ISXEQ2Wrapper/ISXEQ2Wrapper.chm`. The source tree under `ISXEQ2Wrapper/` is also small enough to read directly — recommended when writing non-trivial .NET against ISXEQ2.
