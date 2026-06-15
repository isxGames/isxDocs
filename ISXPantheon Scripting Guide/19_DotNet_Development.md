# .NET Development for ISXPantheon

## Scripts vs .NET Programs

This guide compares the two general development models for InnerSpace automation: plain LavishScript scripts (`.iss` text files, interpreted) and compiled .NET programs (`.exe` / `.dll` assemblies written in C#/VB.NET/F#, loaded via `dotnet <program>` inside InnerSpace). It is a decision-and-orientation guide covering the language, tooling, performance, and deployment tradeoffs between the two models — it is not an API reference.

> **PLANNED — NOT YET IMPLEMENTED.** ISXPantheon does not ship a first-class .NET wrapper assembly today. The Scripts-vs-.NET tradeoffs below are general and apply to any InnerSpace extension, but the strongly-typed `.NET wrapper` interop layer that a mature extension provides (typed TLO getters, domain wrapper classes, etc.) is not available for ISXPantheon yet. Until it ships, all ISXPantheon automation should be written in LavishScript against the real surface (see `03_API_Reference.md`). The generic LavishScript-from-.NET hosting mechanism described under [Hybrid Approaches](#hybrid-approaches) still works, because it is an InnerSpace feature rather than an ISXPantheon one.

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
13. [Comparison Table](#comparison-table)
14. [Summary](#summary)

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

LavishScript is the native automation language for InnerSpace extensions: all TLOs, datatypes, commands, and events are exposed to LavishScript first. For ISXPantheon today, LavishScript is the only supported way to drive the extension (see the planned-.NET note at the top of this guide).

### .NET Programs (.exe/.dll)

Compiled .NET assemblies (C#, VB.NET, or F#) loaded into InnerSpace via `dotnet <program>`. See [.NET Programs (.exe/.dll files)](#net-programs-exedll-files) below for characteristics and development cycle.

---

## LavishScript Scripts (.iss files)

### Characteristics

- **No compilation** — edit and run immediately; instant iteration cycle
- **ISXPantheon native** — the language ISXPantheon is designed for; all TLOs, datatypes, commands, and events are LavishScript-first
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

### Structure Example (multi-file script project)

```
MyBot/
├── MyBot.iss                      ; Main entry point
├── MyBotLib.iss                   ; Shared library
├── Routines/
│   ├── Combat.iss                 ; Behavior-specific routines
│   ├── Movement.iss
│   └── ...                        ; Additional routine files
├── UI/
│   ├── MyBot.xml                  ; LGUI1 UI
│   └── ...                        ; Additional UI files
└── Profiles/
    └── CharacterName.xml          ; Per-character settings

Execution: run MyBot
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
│   │   └── Targeting.cs           ; Target acquisition
│   ├── BehaviorModules/
│   │   └── MainLoop.cs            ; Top-level behavior
│   ├── Core/
│   │   ├── StateCache.cs          ; Cached game state
│   │   └── WorkQueue.cs           ; Action queue
│   ├── Program.cs                 ; Entry point
│   └── MyBot.csproj               ; Project file
├── MyBot.Core/
│   ├── Loader.cs                  ; App initialization
│   └── Interfaces/
│       ├── IStateProvider.cs
│       └── IWorkQueue.cs
└── MyBot.sln                      ; Solution file

Execution: dotnet MyBot
```

> **PLANNED — NOT YET IMPLEMENTED.** The .NET layout above is illustrative of conventional .NET project organization for an InnerSpace bot; it is not based on a specific existing project. There is no published .NET bot for ISXPantheon today because ISXPantheon does not yet expose a first-class .NET wrapper. Most InnerSpace automation in general is written in LavishScript.

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
objectdef obj_Worker
{
    variable string Name
    variable int Iterations

    method Initialize(string name)
    {
        This.Name:Set[${name}]
    }

    member:string GetVersion()
    {
        return ${ISXPantheon.Version}
    }

    method DoWork()
    {
        This.Iterations:Inc
    }
}

; Usage
variable obj_Worker MyWorker
MyWorker:Initialize["Example"]
```

**C# (.NET):**
```csharp
// Full OOP - class, interface, inheritance
public class Worker
{
    public string Name { get; set; }
    public int Iterations { get; private set; }

    public Worker(string name)
    {
        Name = name;
    }

    public string GetVersion()
    {
        // Read a LavishScript member through the InnerSpace .NET API
        // (the exact accessor depends on the LavishScriptAPI version).
        return LavishScript.Objects.GetObject("ISXPantheon").GetMember<string>("Version");
    }

    public void DoWork()
    {
        Iterations++;
    }
}

// Usage
var myWorker = new Worker("Example");
```

### Collections

**LavishScript:**
```lavishscript
; Limited collection types
variable index:int Ids
variable collection:string Settings
variable queue:string Messages

; Usage
Ids:Insert[1]
Settings:Set["mode", "fast"]
Messages:Queue["Hello"]
```

**C# (.NET):**
```csharp
// Rich collection library
using System.Collections.Generic;
using System.Linq;

List<int> ids = new List<int>();
Dictionary<string, string> settings = new Dictionary<string, string>();
Queue<string> messages = new Queue<string>();

// LINQ support
var sorted = ids.Where(id => id > 0).OrderBy(id => id);
```

### Error Handling

**LavishScript:**
```lavishscript
; Limited error handling - no try/catch
if !${ISXPantheon.IsReady}
{
    echo "Error: ISXPantheon is not ready yet"
    return
}
```

**C# (.NET):**
```csharp
// Full exception handling
try
{
    var obj = LavishScript.Objects.GetObject("ISXPantheon");
    bool ready = obj.GetMember<bool>("IsReady");
}
catch (Exception ex)
{
    LogError($"Failed to read ISXPantheon state: {ex.Message}");
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
wait 50 ${ISXPantheon.IsReady}
echo "Extension version: ${ISXPantheon.Version}"
```

**C# (.NET):**
```csharp
// Async/await support
public async Task<bool> WaitForReadyAsync()
{
    var obj = LavishScript.Objects.GetObject("ISXPantheon");

    // Non-blocking wait
    while (!obj.GetMember<bool>("IsReady"))
    {
        await Task.Delay(200);
    }

    return true;
}

var ready = await WaitForReadyAsync();
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
- Per-TLO queries reasonable (native extension C++ implementation)

**.NET:**
- JIT-compiled to native code
- Order-of-magnitude faster for tight loops and CPU-bound work
- First-call JIT overhead amortized quickly

### Memory Usage

**LavishScript:**
- Low baseline for simple scripts
- Higher for complex multi-file bots
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
echo "Ready: ${ISXPantheon.IsReady}"

; Log files
LavishScript:Echo["Debug: iteration ${This.Iterations}"]

; Variable inspection (manual)
method DebugDump()
{
    echo "=== Debug ==="
    echo "State: ${This.CurrentState}"
    echo "Version: ${ISXPantheon.Version}"
    echo "API: ${ISXPantheon.APIVersion}"
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
public void ProcessItem(int itemId)
{
    // Set breakpoint (F9)
    var item = _cache.Lookup(itemId);

    // Execution pauses; inspect item, itemId, locals, call stack
}
```

**Watch Windows:**
```
Watches:
- item.Name
- item.Index
- item.IsValid
- WorkQueue.Count
```

**Immediate Window:**
```csharp
// Execute expressions while paused
> item.Refresh()
> Console.WriteLine(item.Name)
```

**Exception Helper:**
```
Unhandled exception at line 245:
System.NullReferenceException: Object reference not set...

Stack trace:
  at MyBot.ActionModules.Targeting.ProcessItem(Int32 itemId) line 245
  at MyBot.BehaviorModules.MainLoop.Pulse() line 89
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
- No extra dependencies (ISXPantheon assumed present)
- Users can read/modify the source

**Cons:**
- Source is visible (if that matters)
- No source protection

### .NET Deployment

**Steps:**
1. Build solution in Release
2. Copy output DLLs
3. Copy dependencies (any NuGet deps)
4. Copy config files
5. Test on target machine

**Distribution:**
```
MyBot/
├── MyBot.exe
├── MyBot.Core.dll
├── Newtonsoft.Json.dll         ; Example NuGet dependency
├── LavishScriptAPI.dll         ; InnerSpace / LavishScript .NET API
└── Config/
    └── Settings.xml

Zip and share.
```

> **PLANNED — NOT YET IMPLEMENTED.** A mature InnerSpace extension typically ships a strongly-typed .NET wrapper DLL that a .NET bot references for direct, typed access to the game surface. ISXPantheon does not ship such a wrapper yet, so a .NET project can today only reach ISXPantheon through the generic `LavishScriptAPI.dll` object model (untyped `GetObject`/`GetMember`). The deployment bundle will gain a wrapper DLL once first-class .NET support is implemented.

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

### Calling Scripts from .NET (ISXPantheon interop model)

> **PLANNED — NOT YET IMPLEMENTED.** ISXPantheon does not yet provide a first-class, strongly-typed .NET wrapper (no typed TLO getters, no domain wrapper classes, no command-helper layer). A mature InnerSpace extension usually ships such a wrapper so that .NET code gets typed access to the game surface; that layer is on the ISXPantheon roadmap but is not available today. Until it lands, the only way to reach ISXPantheon from .NET is the generic InnerSpace `LavishScriptAPI.dll` object model described below — which is untyped and limited to the small real surface (`${ISXPantheon}`).

**Generic interop (works today, via `LavishScriptAPI.dll`).** Because TLO access in InnerSpace is engine-level rather than extension-level, you can read the real ISXPantheon surface from .NET without any ISXPantheon-specific wrapper:

```csharp
using LavishScriptAPI;

public class ExtensionState
{
    public bool IsReady()
    {
        // Untyped engine-level access to the ${ISXPantheon} TLO.
        return LavishScript.Objects.GetObject("ISXPantheon").GetMember<bool>("IsReady");
    }

    public string Version()
    {
        return LavishScript.Objects.GetObject("ISXPantheon").GetMember<string>("Version");
    }

    public void RunCommand()
    {
        // Fire any LavishScript command from .NET (engine-level escape hatch).
        LavishScript.ExecuteCommand("echo ISXPantheon ${ISXPantheon.Version}");
    }
}
```

That is the full extent of typed-by-hand interop available now: read the `${ISXPantheon}` members listed in `03_API_Reference.md`, and run commands via `LavishScript.ExecuteCommand(...)`. There are no game-data wrapper objects to call because those datatypes have no working members yet.

**Use Cases (today):**
- .NET app orchestrating LavishScript via `ExecuteCommand`
- .NET reading the real `${ISXPantheon}` state members

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
2. Reference LavishScriptAPI.dll (and the ISXPantheon .NET wrapper once it ships)
3. Implement core classes (StateCache, WorkQueue, etc.)
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
// supply its own base class. InnerSpace doesn't impose a bot framework;
// the .NET API just exposes engine-level LavishScript object access.)
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
- [ ] Learning ISXPantheon automation
- [ ] Need rapid iteration
- [ ] Sharing with the InnerSpace LavishScript community (by far the larger pool)
- [ ] Don't know C#/.NET
- [ ] Want simple deployment

**Examples:** simple utility scripts, log monitors, AFK alarm, custom-variable helpers.

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
Quick tweaks, config: .iss scripts
Heavy computation (pathfinding, analysis): .NET
UI: LGUI2 JSON + small .iss glue
```

---

## Planned .NET Wrapper

> **PLANNED — NOT YET IMPLEMENTED.** ISXPantheon does not currently ship a dedicated, strongly-typed .NET wrapper assembly (the typed `Extension` getters, domain wrapper classes, namespace layout, and command helpers that a mature extension provides). A wrapper is on the roadmap; its assembly name, namespaces, and surface are not finalized and will change. Do not write production .NET against an assumed wrapper API.

Until a wrapper ships, the only supported .NET interop with ISXPantheon is the generic, engine-level InnerSpace object model from `LavishScriptAPI.dll`:

- **Get a TLO handle:** `LavishScript.Objects.GetObject("ISXPantheon")` (the only datatype with working members today — see `03_API_Reference.md`).
- **Read a member:** `handle.GetMember<T>("MemberName")` (e.g. `GetMember<bool>("IsReady")`, `GetMember<string>("Version")`).
- **Run any LavishScript command:** `LavishScript.ExecuteCommand("cmd args...")` directly from `LavishScriptAPI.dll`.

There are no typed game-data wrapper classes yet because those datatypes have no working members.

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

1. **Scripts are simple.** Instant gratification, low barrier to entry, ideal for learning and for small-to-medium bots. For ISXPantheon today, LavishScript is the only supported automation language, so all current ISXPantheon automation is LavishScript-based.

2. **.NET is powerful.** Full debugger, LINQ, async, unit tests, and arbitrary third-party libraries. Better for complex, performance-critical, or commercially-maintained projects.

3. **ISXPantheon has no first-class .NET wrapper yet (planned).** Until one ships, .NET code can only reach ISXPantheon through the generic, untyped InnerSpace object model:
   - `LavishScript.Objects.GetObject("ISXPantheon")` for the one working TLO handle,
   - `handle.GetMember<T>(...)` to read its members,
   - `LavishScript.ExecuteCommand("...")` to run any LavishScript command.

4. **Choose based on need.** Simple utility or learning? Script. Complex/production/performance-critical? Plan for .NET once the wrapper lands. Can't pick? Hybrid — orchestrate in scripts, compute in .NET.

5. **Authoritative API reference:** the guide's `03_API_Reference.md` (real ISXPantheon surface). There is no separate .NET wrapper help file because no wrapper ships yet.
