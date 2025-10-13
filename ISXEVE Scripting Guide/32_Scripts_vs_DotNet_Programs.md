# File 32: Scripts vs DotNet Programs

**Layer 8: DotNet Bridge - Part 1 of 3**

## Table of Contents
1. [Fundamental Differences](#fundamental-differences)
2. [LavishScript Scripts (.iss files)](#lavishscript-scripts)
3. [DotNet Programs (.exe/.dll files)](#dotnet-programs)
4. [Language Comparison](#language-comparison)
5. [Development Workflow](#development-workflow)
6. [Performance Comparison](#performance-comparison)
7. [Debugging Capabilities](#debugging-capabilities)
8. [Deployment and Distribution](#deployment)
9. [Maintenance and Updates](#maintenance)
10. [Hybrid Approaches](#hybrid-approaches)
11. [Migration Strategies](#migration)
12. [Decision Matrix](#decision-matrix)

---

## Fundamental Differences <a name="fundamental-differences"></a>

### Scripts (.iss files)

**What They Are:**
- Plain text files containing LavishScript code
- Interpreted at runtime by LavishScript engine
- Executed directly in InnerSpace via `run scriptname`

**Technology:**
- **Language:** LavishScript
- **Execution:** Interpreted
- **Compilation:** None
- **Runtime:** LavishScript engine (C++)

**Examples:**
- Yamfa (fleet assist) - single .iss file
- EVEBot (mining/combat) - multiple .iss files
- Tehbot (combat) - multiple .iss files

### DotNet Programs (.exe/.dll)

**What They Are:**
- Compiled .NET assemblies (executable or library)
- Run via `dotnet programname` command in InnerSpace
- Written in C# or VB.NET

**Technology:**
- **Language:** C#, VB.NET, F#
- **Execution:** Compiled to IL, JIT compiled
- **Compilation:** Required (Visual Studio, MSBuild)
- **Runtime:** .NET Framework CLR

**Examples:**
- Metatron (combat/mining) - C# .NET application
- Custom utilities and tools

---

## LavishScript Scripts (.iss files) <a name="lavishscript-scripts"></a>

### Characteristics

**Advantages:**
1. **No Compilation** - Edit and run immediately
2. **Simple Deployment** - Just copy .iss files
3. **Easy to Share** - Text files, readable source
4. **Quick Iteration** - Change code, restart script
5. **Low Barrier to Entry** - Simpler language
6. **ISXEVE Native** - Designed for ISXEVE

**Disadvantages:**
1. **Slower Execution** - Interpreted, not compiled
2. **Limited IDE Support** - Basic text editors only
3. **No Type Safety** - Runtime type errors
4. **Limited Debugging** - Echo statements mainly
5. **No Intellisense** - Must memorize API
6. **Limited Libraries** - Fewer third-party options

### Structure Example (Yamfa)

```
Yamfa.iss                          # Single file script

Contents:
- Variable declarations
- Objectdef definitions
- Function definitions
- Main execution loop
- Event handlers

Size: ~845 lines
Execution: run Yamfa
```

### Structure Example (EVEBot)

```
EVEBot/
├── EVEBot.iss                     # Main entry point
├── Core/
│   ├── obj_Configuration.iss      # Config system
│   ├── obj_Logger.iss             # Logging
│   ├── obj_Ship.iss               # Ship control
│   └── obj_Asteroids.iss          # Mining logic
├── Behaviors/
│   ├── obj_Miner.iss              # Mining behavior
│   ├── obj_Hauler.iss             # Hauling behavior
│   └── obj_Combat.iss             # Combat behavior
└── Config/
    └── CharacterName Config.xml   # Per-character settings

Size: ~50+ files, 20,000+ lines
Execution: run EVEBot
```

### Development Cycle

```
1. Edit .iss file in text editor (Notepad++, VS Code)
2. Save file
3. In InnerSpace console: run ScriptName
4. Test in game
5. If error: echo statements for debugging
6. Make changes
7. endscript ScriptName
8. Repeat from step 3
```

**Typical Iteration Time:** 10-30 seconds

---

## DotNet Programs (.exe/.dll files) <a name="dotnet-programs"></a>

### Characteristics

**Advantages:**
1. **Better Performance** - Compiled to native code
2. **Full IDE Support** - Visual Studio, ReSharper
3. **Type Safety** - Compile-time error detection
4. **Advanced Debugging** - Breakpoints, watches, profiling
5. **Intellisense** - Auto-complete, parameter hints
6. **Rich Ecosystem** - NuGet packages, libraries
7. **Better Architecture** - OOP, LINQ, async/await
8. **Professional Tooling** - Git integration, refactoring

**Disadvantages:**
1. **Compilation Required** - Must build before running
2. **Complex Deployment** - Multiple DLLs, dependencies
3. **Higher Skill Requirement** - C# knowledge needed
4. **Longer Iteration** - Edit → Build → Run cycle
5. **Not Open Source** - Compiled binaries are opaque

### Structure Example (Metatron)

```
Metatron_spacekoala/
├── Metatron/                      # Main application
│   ├── ActionModules/
│   │   ├── Movement.cs            # Movement logic
│   │   ├── Offensive.cs           # Weapon control
│   │   ├── Defense.cs             # Tank management
│   │   └── Targeting.cs           # Target management
│   ├── BehaviorModules/
│   │   ├── Mining.cs              # Mining behavior
│   │   ├── Salvaging.cs           # Salvaging behavior
│   │   ├── CombatAssist.cs        # Combat behavior
│   │   └── MissionRunner.cs       # Mission automation
│   ├── Core/
│   │   ├── Ship.cs                # Ship state
│   │   ├── EntityCache.cs         # Entity caching
│   │   ├── MeCache.cs             # Player state
│   │   └── EntityProvider.cs      # Entity registry
│   ├── Program.cs                 # Entry point
│   └── Metatron.csproj            # Project file
├── Metatron.Core/
│   ├── Loader.cs                  # App initialization
│   └── Interfaces/
│       ├── IEntityProvider.cs     # Entity interface
│       └── ITargetQueue.cs        # Target queue interface
└── Metatron.sln                   # Solution file

Size: 3000+ files, 100,000+ lines
Execution: dotnet Metatron
```

### Development Cycle

```
1. Edit .cs files in Visual Studio
2. Save changes (auto-saves)
3. Build solution (Ctrl+Shift+B)
   - Compilation errors shown inline
   - Fix errors before running
4. Copy built DLL to game directory
5. In InnerSpace console: dotnet Metatron
6. Test in game
7. If error: View stack trace, use breakpoints
8. Make changes
9. Repeat from step 3
```

**Typical Iteration Time:** 1-3 minutes (compilation time)

---

## Language Comparison <a name="language-comparison"></a>

### Type Systems

**LavishScript:**
```lavish
; Weakly typed - runtime type checking
variable string myString = "hello"
variable int myInt = 123
variable float myFloat = 45.67

; Type errors caught at runtime
myInt:Set["not a number"]    ; Runtime error!
```

**C# (.NET):**
```csharp
// Strongly typed - compile-time checking
string myString = "hello";
int myInt = 123;
float myFloat = 45.67f;

// Type errors caught at compile time
myInt = "not a number";    // Compiler error!
```

### Object-Oriented Programming

**LavishScript:**
```lavish
; Basic OOP - objectdef
objectdef obj_Ship
{
    variable string Name
    variable float ShieldPct

    method Initialize(string name)
    {
        This.Name:Set[${name}]
    }

    member:float GetShield()
    {
        return ${MyShip.Shield.Pct}
    }

    method ActivateShield()
    {
        Ship:Activate_Shield_Boosters
    }
}

; Usage
variable obj_Ship MyShipController
MyShipController:Initialize["Vexor"]
```

**C# (.NET):**
```csharp
// Full OOP - class, interface, inheritance
public class Ship
{
    public string Name { get; set; }
    public float ShieldPct { get; private set; }

    public Ship(string name)
    {
        Name = name;
    }

    public float GetShield()
    {
        return MyShip.Shield.Pct;
    }

    public void ActivateShield()
    {
        // Activate shield boosters
    }
}

// Usage
var myShipController = new Ship("Vexor");
```

### Collections

**LavishScript:**
```lavish
; Limited collection types
variable index:entity Targets
variable collection:int TargetIDs
variable queue:string Messages

; Usage
Targets:Insert[${Entity[${ID}]}]
TargetIDs:Set[${ID}, 1]
Messages:Queue["Hello"]
```

**C# (.NET):**
```csharp
// Rich collection library
List<Entity> targets = new List<Entity>();
Dictionary<int, int> targetIDs = new Dictionary<int, int>();
Queue<string> messages = new Queue<string>();

// LINQ support
var hostiles = targets.Where(t => t.IsHostile).OrderBy(t => t.Distance);
```

### Error Handling

**LavishScript:**
```lavish
; Limited error handling
if !${Entity[${targetID}](exists)}
{
    echo "Error: Entity doesn't exist"
    return
}

; No try-catch
```

**C# (.NET):**
```csharp
// Full exception handling
try
{
    var entity = GetEntity(targetID);
    entity.LockTarget();
}
catch (EntityNotFoundException ex)
{
    LogError($"Entity not found: {ex.Message}");
}
catch (Exception ex)
{
    LogException("Unexpected error", ex);
}
finally
{
    // Cleanup code
}
```

### Async Programming

**LavishScript:**
```lavish
; No async/await - must use waits
Entity[${targetID}]:LockTarget
wait 50 ${Entity[${targetID}].IsLockedTarget}

; Blocking waits only
```

**C# (.NET):**
```csharp
// Async/await support
public async Task<bool> LockTargetAsync(long targetID)
{
    entity.LockTarget();

    // Non-blocking wait
    await Task.Delay(50);

    return entity.IsLockedTarget;
}

// Usage
var result = await LockTargetAsync(targetID);
```

---

## Development Workflow <a name="development-workflow"></a>

### Script Development (LavishScript)

**Tools:**
- **Editor:** Notepad++, VS Code, Sublime Text
- **Testing:** In-game only
- **Debugging:** Echo statements, log files
- **Version Control:** Manual copies or Git

**Workflow:**
```
Edit → Save → Run → Test → Debug (echo) → Repeat
```

**Pros:**
- Immediate feedback
- Simple tools
- No build step
- Easy sharing (copy .iss files)

**Cons:**
- No syntax highlighting (basic editors)
- No code completion
- Manual debugging
- No refactoring tools

### DotNet Development (.NET)

**Tools:**
- **IDE:** Visual Studio (Community, Professional, Enterprise)
- **Testing:** In-game + Unit tests
- **Debugging:** Full debugger with breakpoints
- **Version Control:** Integrated Git

**Workflow:**
```
Edit → Save → Build → Copy DLL → Run → Test → Debug (breakpoints) → Repeat
```

**Pros:**
- Syntax highlighting
- Intellisense/auto-complete
- Refactoring tools
- Advanced debugging
- Unit testing

**Cons:**
- Build time (10-60 seconds)
- Complex setup
- Deployment overhead
- Learning curve

---

## Performance Comparison <a name="performance-comparison"></a>

### Execution Speed

**LavishScript:**
- Interpreted - ~10-100x slower than compiled
- No JIT optimization
- String operations especially slow
- Entity queries reasonable (native ISXEVE)

**Example Timing:**
```lavish
; Loop 10,000 times
variable int i
variable float startTime = ${Time.Timestamp}

for (i:Set[1]; ${i} <= 10000; i:Inc)
{
    ; Some calculation
    variable int result = ${Math.Calc[${i} * 2 + 5]}
}

variable float duration = ${Math.Calc[${Time.Timestamp} - ${startTime}]}
echo "LavishScript: ${duration}s"
; Typical: 2-5 seconds
```

**.NET:**
```csharp
var startTime = DateTime.Now;

for (int i = 1; i <= 10000; i++)
{
    int result = i * 2 + 5;
}

var duration = (DateTime.Now - startTime).TotalSeconds;
Console.WriteLine($".NET: {duration}s");
// Typical: 0.001-0.01 seconds (100-1000x faster!)
```

### Memory Usage

**LavishScript:**
- ~20-50 MB for simple scripts
- ~50-100 MB for complex scripts (EVEBot)
- Grows over time (limited GC)

**.NET:**
- ~30-80 MB base (CLR overhead)
- ~100-300 MB for complex apps (Metatron)
- Better garbage collection
- Can grow large with caching

### Startup Time

**LavishScript:**
- Nearly instant (< 1 second)
- Just parse text and run

**.NET:**
- 2-5 seconds startup
- CLR initialization
- Assembly loading
- JIT compilation

---

## Debugging Capabilities <a name="debugging-capabilities"></a>

### Script Debugging

**Available Tools:**
```lavish
; Echo statements
echo "Current state: ${This.CurrentState}"
echo "Target: ${Entity[${targetID}].Name}"

; Log files
Logger:Log["Debug: Processing target ${targetID}"]

; Variable inspection (manual)
method DebugDump()
{
    echo "=== Debug ==="
    echo "State: ${This.CurrentState}"
    echo "InSpace: ${Me.InSpace}"
    echo "Shield: ${MyShip.Shield.Pct}%"
    echo "============="
}
```

**Limitations:**
- No breakpoints
- No watch windows
- No stack traces
- No step-through debugging
- Manual variable dumping

### .NET Debugging

**Available Tools:**

**Breakpoints:**
```csharp
public void ProcessTarget(long targetID)
{
    // Set breakpoint here (F9)
    var entity = EntityProvider.GetEntity(targetID);

    // Execution pauses, inspect variables:
    // - entity
    // - targetID
    // - All local variables
    // - Call stack
}
```

**Watch Windows:**
```
Watches:
- entity.Name
- entity.Distance
- entity.IsLockedTarget
- TargetQueue.Count
```

**Immediate Window:**
```csharp
// Execute code while paused
> entity.LockTarget()
> Console.WriteLine(entity.Name)
```

**Exception Helper:**
```
Unhandled exception at line 245:
System.NullReferenceException: Object reference not set...

Stack trace:
  at Metatron.ActionModules.Targeting.ProcessTarget(Int64 targetID) line 245
  at Metatron.BehaviorModules.CombatAssist.Pulse() line 89
  at Metatron.Core.ModuleBase.Execute() line 34

Shows exact line, full stack, all variables
```

---

## Deployment and Distribution <a name="deployment"></a>

### Script Deployment

**Steps:**
1. Copy .iss files to InnerSpace/Scripts directory
2. Copy any config/data files
3. Run script via `run ScriptName`

**Distribution:**
```
YourBot/
├── YourBot.iss
├── Core/
│   ├── obj_Config.iss
│   └── obj_Logger.iss
└── README.txt

Zip and share - that's it!
```

**Pros:**
- Dead simple
- No dependencies (ISXEVE assumed)
- Works immediately
- Easy to modify

**Cons:**
- Source code visible (if you care)
- No protection

### .NET Deployment

**Steps:**
1. Build solution in Release mode
2. Copy output DLLs (10-50 files)
3. Copy dependencies (ISXEVE.dll, etc.)
4. Copy config files
5. Test on target machine

**Distribution:**
```
Metatron/
├── Metatron.exe
├── Metatron.Core.dll
├── Newtonsoft.Json.dll         # NuGet dependency
├── System.Net.Http.dll         # Framework library
├── ISXEVE.dll                  # ISXEVE wrapper
├── LavishScript.dll            # LavishScript wrapper
└── Config/
    └── Settings.xml

Zip and share
```

**Pros:**
- Compiled (source protected)
- Professional packaging

**Cons:**
- Complex - many DLLs
- Dependency management
- Version conflicts possible
- Harder for users to modify

---

## Maintenance and Updates <a name="maintenance"></a>

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
```lavish
; In script
variable string VERSION = "2.5"

method Initialize()
{
    echo "YourBot v${VERSION} starting"
}
```

**Pros:**
- Instant updates
- Users can see changes (text diff)
- Easy to patch

**Cons:**
- No automatic updates
- Users must manually download

### .NET Maintenance

**Updating:**
```
1. Edit .cs files
2. Save and build
3. Test build
4. Create release package
5. Users download new DLLs
6. Replace all DLLs
7. Restart program
```

**Version Management:**
```csharp
// In AssemblyInfo.cs
[assembly: AssemblyVersion("2.5.0.0")]
[assembly: AssemblyFileVersion("2.5.0.0")]

// In code
var version = Assembly.GetExecutingAssembly().GetName().Version;
Console.WriteLine($"Metatron v{version}");
```

**Auto-Update:**
```csharp
// Can implement auto-updater
public class AutoUpdater
{
    public async Task<bool> CheckForUpdates()
    {
        // Download version manifest
        // Compare versions
        // Download new DLLs
        // Replace on restart
    }
}
```

**Pros:**
- Professional versioning
- Auto-update possible
- Release management

**Cons:**
- Complex update process
- Must replace all DLLs
- Potential for breakage

---

## Hybrid Approaches <a name="hybrid-approaches"></a>

### Calling .NET from Scripts

**Pattern:** Use .NET library from LavishScript

```csharp
// .NET DLL: MyUtilities.dll
namespace MyUtilities
{
    public class Helper
    {
        public static int Calculate(int a, int b)
        {
            return a * 2 + b;
        }
    }
}
```

```lavish
; LavishScript calls .NET
dotnet MyUtilities

variable int result = ${DotNet.Invoke[MyUtilities.Helper, "Calculate", 5, 10]}
echo "Result: ${result}"
; Output: Result: 20
```

**Use Cases:**
- Complex calculations in .NET
- Use .NET libraries (JSON parsing, HTTP requests)
- Leverage C# performance for bottlenecks

### Calling Scripts from .NET

**Pattern:** Use LavishScript commands from .NET

```csharp
// .NET code
using LavishScriptAPI;

public class ScriptCaller
{
    public void CallScript()
    {
        // Execute LavishScript command
        LavishScript.ExecuteCommand("run MyScript");

        // Wait for script to finish
        Thread.Sleep(5000);

        // End script
        LavishScript.ExecuteCommand("endscript MyScript");
    }

    public string GetVariable(string varName)
    {
        // Get LavishScript variable
        return LavishScript.DataParse($"${{${varName}}}").ToString();
    }
}
```

**Use Cases:**
- .NET app orchestrates multiple scripts
- .NET reads script variables
- Mixed architecture

---

## Migration Strategies <a name="migration"></a>

### Script to .NET Migration

**When to Migrate:**
- Performance becomes critical
- Need advanced features (async, LINQ)
- Want professional tooling
- Planning commercial distribution

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
1. Create .NET project
2. Reference ISXEVE.dll
3. Implement core classes (Ship, Entity, etc.)
4. Port configuration system
```

**Phase 3: Incremental Port**
```
1. Port one behavior at a time
2. Test each behavior in-game
3. Keep script running alongside .NET during transition
4. Gradually shift functionality to .NET
```

**Phase 4: Completion**
```
1. Port all behaviors
2. Remove script
3. Polish .NET app
4. Package for distribution
```

**Example Migration:**

```lavish
; Original Script: obj_Miner
objectdef obj_Miner
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
// .NET Port: Mining.cs
namespace Metatron.BehaviorModules
{
    public class Mining : ModuleBase
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

## Decision Matrix <a name="decision-matrix"></a>

### When to Use Scripts

**Use LavishScript Scripts When:**

✅ Prototyping or proof-of-concept
✅ Simple automation (< 1000 lines)
✅ Learning EVE botting
✅ Need rapid iteration
✅ Sharing with beginner community
✅ Don't know C#/.NET
✅ Want simple deployment

**Examples:**
- Fleet assist bot (Yamfa)
- Simple hauler
- Salvage bot
- Mining bot (small scale)

### When to Use .NET

**Use .NET Programs When:**

✅ Complex logic (> 5000 lines)
✅ Performance critical
✅ Need advanced features (async, LINQ, ESI API)
✅ Professional/commercial project
✅ Team development
✅ Long-term maintenance
✅ Want best debugging tools
✅ Already know C#

**Examples:**
- Full-featured combat bot (Metatron)
- Mission runner with ESI integration
- Multi-character fleet coordinator
- Market trading bot
- Large-scale mining operation

### Hybrid Approach

**Use Hybrid (Scripts + .NET) When:**

✅ Want best of both worlds
✅ Performance-critical algorithms need .NET
✅ Rapid prototyping in scripts, production in .NET
✅ Incremental migration strategy

**Example:**
```
Main bot: .NET (Metatron)
Simple tasks: Scripts
Fleet coordination: Script relay, .NET processing
```

---

## Comparison Table

| Feature | LavishScript Scripts | .NET Programs |
|---------|---------------------|---------------|
| **Language** | LavishScript | C#, VB.NET, F# |
| **Execution** | Interpreted | Compiled (JIT) |
| **Performance** | Slow (~100x slower) | Fast (native code) |
| **Startup** | Instant (< 1s) | Slow (2-5s) |
| **IDE Support** | Basic editors | Visual Studio (full) |
| **Debugging** | Echo/logs only | Breakpoints, watches, stack traces |
| **Type Safety** | Runtime | Compile-time |
| **Intellisense** | None | Full |
| **Unit Testing** | None | Full (NUnit, xUnit) |
| **Async/Await** | No | Yes |
| **LINQ** | No | Yes |
| **Error Handling** | Limited | Full try-catch-finally |
| **Memory Usage** | Low (20-100 MB) | Medium (100-300 MB) |
| **Build Time** | None | 10-60 seconds |
| **Deployment** | Copy .iss files | Copy many DLLs |
| **Updates** | Replace .iss | Replace all DLLs |
| **Learning Curve** | Easy | Moderate-Hard |
| **Code Sharing** | Easy (text files) | Harder (binaries) |
| **Source Protection** | None | Compiled |
| **Best For** | Simple bots, prototypes | Complex bots, production |

---

## Real-World Examples

### Script Example: Yamfa (Fleet Assist)

**Stats:**
- **Lines:** 845
- **Files:** 1
- **Complexity:** Medium
- **Performance:** Good (relay focused)
- **Maintenance:** Easy (one file)

**Why Script:**
- Simple relay pattern
- Rapid iteration needed
- Easy to share with beginners
- Performance sufficient

### .NET Example: Metatron (Combat/Mining)

**Stats:**
- **Lines:** 100,000+
- **Files:** 3000+
- **Complexity:** Very High
- **Performance:** Excellent
- **Maintenance:** Complex but manageable

**Why .NET:**
- Complex ESI API integration
- Performance critical (combat)
- Professional tooling needed
- Long-term project
- Team development

---

## Summary

### Key Takeaways

1. **Scripts are Simple**
   - Instant gratification
   - Low barrier to entry
   - Perfect for learning
   - Good for simple bots

2. **.NET is Powerful**
   - Professional tools
   - Better performance
   - Advanced features
   - Better for complex bots

3. **Choose Based on Need**
   - Simple bot? Script
   - Complex bot? .NET
   - Learning? Script
   - Production? .NET

4. **Hybrid is Valid**
   - Scripts for simple tasks
   - .NET for heavy lifting
   - Best of both worlds

### What's Next?

**File 33:** When to Use DotNet Instead
- Detailed decision criteria
- Performance thresholds
- Feature requirements
- Migration triggers

**File 34:** Metatron DotNet Architecture Overview
- Real .NET bot analysis
- Module architecture
- Best practices from production code

---

*Layer 8 Progress: 1/3 Complete (33%)*
*Total Documentation Progress: 31/37 Files (83.8%)*
*ONLY 6 FILES REMAINING!* 🚀💎✨
