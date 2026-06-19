# Debugging and Troubleshooting

**Purpose:** Diagnostic techniques, logging strategy, performance profiling, and the current ISXPantheon problem set for InnerSpace scripts.
**Audience:** Script authors troubleshooting misbehavior, tuning performance, or hardening long-running automation.

This guide is a **capstone** that consolidates debugging and troubleshooting. It leans on LavishScript-native `redirect`, `echo`, and `${Script.*}` introspection rather than reinventing a logger from scratch. Foundational existence checks, NULL safety, anti-patterns, and the `${ISXPantheon.IsReady}` wait-gate are covered canonically elsewhere and cross-linked rather than duplicated.

> **Note on the current ISXPantheon surface.** ISXPantheon today exposes a deliberately minimal surface: the `${ISXPantheon}` TLO (version / API / readiness / custom variables / currency / rounding helpers), the `GetURL` / `PostURL` commands, and registered-but-empty `${Pantheon}` / `entity` / `ability` / `quest` datatypes. There are no game-state members (no character, target, entity, item, ability, or quest data) and no game events yet. Because of that, the bulk of real-world troubleshooting today is **"this feature is not yet implemented"** rather than game-mechanics diagnosis. The generic debugging toolkit below (logging, `${Script.*}`, profiling, the readiness gate) is fully usable now; sections that depend on planned game surface are labeled as such.

---

## Table of Contents

### Core Diagnostic Toolkit
1. [The ISXPantheon Debug-Tool Landscape](#the-isxpantheon-debug-tool-landscape)
2. [A Lightweight Logging Object](#a-lightweight-logging-object)
3. [Echo and Console Output](#echo-and-console-output)
4. [Verbosity Tiers and Conditional Logging](#verbosity-tiers-and-conditional-logging)
5. [File-Based Logging with `redirect`](#file-based-logging-with-redirect)
6. [Script Introspection via `${Script.*}`](#script-introspection-via-script)
7. [Type Inspection and Variable Dumping](#type-inspection-and-variable-dumping)
8. [Performance Profiling](#performance-profiling)
9. [Session Validation Checklist](#session-validation-checklist)

### Common Problems
10. [Feature Not Yet Implemented (the #1 issue today)](#feature-not-yet-implemented-the-1-issue-today)
11. [Waiting on `${ISXPantheon.IsReady}`](#waiting-on-isxpantheonisready)
12. [Custom Variables Returning Blank](#custom-variables-returning-blank)
13. [GetURL / PostURL Troubleshooting](#geturl--posturl-troubleshooting)
14. [LGUI2 Element-Not-Found Diagnosis](#lgui2-element-not-found-diagnosis)

### Reliability and Resilience
15. [Systematic Issue Diagnosis Workflow](#systematic-issue-diagnosis-workflow)
16. [Performance Investigation Workflow](#performance-investigation-workflow)
17. [State-Machine Stuck Detection](#state-machine-stuck-detection)
18. [Dead Man's Switch and Periodic Restart](#dead-mans-switch-and-periodic-restart)
19. [Config Backup and Restore](#config-backup-and-restore)
20. [Relay Event Debugging](#relay-event-debugging)
21. [Top-10 Checklist](#top-10-checklist)

---

## The ISXPantheon Debug-Tool Landscape

**ISXPantheon does not expose any TLO-level debug helpers** (no `Debug_LogMsg`, `IsSafe`, `LastError`, or `Lag` member). The only `${ISXPantheon}` members relevant to diagnostics are:

- `${ISXPantheon.IsReady}` — gate before first API access
- `${ISXPantheon.Version}` — version string for bug reports
- `${ISXPantheon.APIVersion}` — API version string for bug reports

For everything else, lean on LavishScript-native tooling, which is platform-level and always available:

- **A lightweight logging object** you define yourself (see [section 2](#a-lightweight-logging-object)).
- **LavishScript-native `redirect`** for raw file output. See [section 5](#file-based-logging-with-redirect).
- **`${Script.*}` introspection** (`RunningTime`, `MemUsage`, `CurrentDirectory`, `Filename`) for runtime state. See [section 6](#script-introspection-via-script).
- **Type inspection** (`${Obj.Type}`, `Script:Dump`) for variable introspection. See [section 7](#type-inspection-and-variable-dumping).

For **foundational patterns** — existence checks, NULL safety, iterator patterns — see:
- [04_Core_Concepts.md → NULL Checks and Existence Validation](04_Core_Concepts.md#null-checks-and-existence-validation)
- [05_Patterns_And_Best_Practices.md → Error Handling Strategies](05_Patterns_And_Best_Practices.md#error-handling-strategies)
- [05_Patterns_And_Best_Practices.md → Common Anti-Patterns](05_Patterns_And_Best_Practices.md#common-anti-patterns)

---

## A Lightweight Logging Object

ISXPantheon does not ship a logging facility, so define a small `objectdef` logger yourself. The one below is self-contained pure LavishScript — drop it into any script. It gates output behind an `Enabled` flag (so calls can be left in production code as no-ops) and can mirror console output to a file.

### A reusable logger objectdef

```lavishscript
objectdef obj_Log
{
    variable bool   Enabled = FALSE
    variable bool   EchoAlsoLogs = FALSE
    variable string Prefix = "DEBUG: "
    variable string Filename = "${Script.CurrentDirectory}/${Script.Filename}.txt"

    method Enable()
    {
        This.Enabled:Set[TRUE]
    }

    method Disable()
    {
        This.Enabled:Set[FALSE]
    }

    method SetFilename(string path)
    {
        This.Filename:Set["${path}"]
    }

    method SetPrefix(string p)
    {
        This.Prefix:Set["${p}"]
    }

    method Echo(string msg)
    {
        if !${This.Enabled}
            return

        echo "${This.Prefix}[${Time.Time24}] ${msg}"

        if ${This.EchoAlsoLogs}
            redirect -append "${This.Filename}" echo "${This.Prefix}[${Time.Time24}] ${msg}"
    }

    method Log(string msg)
    {
        if !${This.Enabled}
            return

        redirect -append "${This.Filename}" echo "${This.Prefix}[${Time.Time24}] ${msg}"
    }
}

variable(script) obj_Log Log
```

### Canonical setup

```lavishscript
function main()
{
    Log:Enable
    Log:SetPrefix[""]                                             ; no prefix
    Log.EchoAlsoLogs:Set[TRUE]                                    ; echo mirrors to file
    Log:SetFilename["${LavishScript.HomeDirectory}/Scripts/MyScript/MyScript_Debug.txt"]

    Log:Echo["-------- Session start --------"]
    Log:Echo["(${Time.Date} at ${Time.Time24})"]

    Log:Echo["Script initialized"]
}
```

### Conditional debug through a script-scoped flag

```lavishscript
variable(script) bool EnableDebug = FALSE

function main()
{
    Script:DisableDebugging                                        ; drop the LavishScript-level debugger
    Log:SetFilename["${LavishScript.HomeDirectory}/Scripts/MyScript/MyScript_Debug.txt"]

    if ${EnableDebug}
        Log:Enable

    Log:Echo["Starting..."]                                        ; only appears if enabled
}
```

When the logger is disabled, `Log:Echo` and `Log:Log` are no-ops — safe to leave sprinkled throughout production code. This is the equivalent of an "`if DEBUG` guard" without the explicit guards.

### Layering severity on top of the logger

The logger has no concept of levels. For severity, wrap it in atoms:

```lavishscript
variable(script) bool EnableDebug = FALSE
variable(script) bool Quiet = FALSE

atom(script) LogInfo(... Params)
{
    if !${Params.Size}
        return
    Log:Echo["[INFO] ${Params.Expand}"]
}

atom(script) LogWarn(... Params)
{
    if !${Params.Size}
        return
    Log:Echo["[WARN] ${Params.Expand}"]
}

atom(script) LogError(... Params)
{
    if !${Params.Size} || ${Quiet}
        return
    Log:Echo["[ERROR] ${Params.Expand}"]
}

atom(script) LogDebug(... Params)
{
    if !${EnableDebug} || !${Params.Size}
        return
    Log:Echo["[DEBUG] ${Params.Expand}"]
}
```

Call these from anywhere in the script — `LogInfo "Worker starting"`, `LogError "Custom variable missing"`, etc. The `... Params` plus `${Params.Expand}` pattern lets a logging atom accept any number of arguments.

### When to NOT use the logger

- When you want output even when debugging is disabled — use `echo` directly.
- When you need per-subsystem log files — use `redirect -append` directly (see [section 5](#file-based-logging-with-redirect)).
- When you need sub-second-precision timing — the logger timestamps with `${Time.Time24}` (HH:MM:SS); use `${Script.RunningTime}` (ms since script start) or `${Time.Timestamp}` (seconds since epoch) directly.

---

## Echo and Console Output

The primary output stream for diagnostics is the InnerSpace console:

- **`echo`** — writes to the InnerSpace console. Developer-visible.

```lavishscript
echo "Debug: ISXPantheon ready = ${ISXPantheon.IsReady}"           ; InnerSpace console
```

### Color codes

InnerSpace console color codes (use with `echo`):

```lavishscript
echo "\agGreen success\ax"
echo "\arRed error\ax"
echo "\ayYellow warning\ax"
echo "\aoOrange alert\ax"
echo "\awWhite (default)\ax"
```

### Deciding which stream

| Purpose | Stream | Visibility |
|---|---|---|
| Developer diagnostics | `echo` or `Log:Echo` | InnerSpace console |
| Persistent record | `Log:Log` or `redirect -append` | File on disk |
| Critical alert | `echo` + audio (`play`) | Console + sound |

---

## Verbosity Tiers and Conditional Logging

For scripts with noisy diagnostic output, tier your verbosity:

```lavishscript
variable(script) bool V1 = FALSE                                   ; verbose
variable(script) bool V2 = FALSE                                   ; very verbose

atom(script) VerboseEcho(... Params)
{
    if !${Params.Size}
        return

    if ${V1}
        echo ${Params.Expand}
}
```

Flip `V1` / `V2` at runtime from the command line or a UI checkbox to elevate verbosity without restarting the script.

---

## File-Based Logging with `redirect`

The lightweight logger writes to a single file. For multiple log streams (main, errors, metrics) or rotation, use `redirect` directly.

### Basic usage

```lavishscript
; Append one line
redirect -append "mybot.log" echo "[${Time.Time24}] Started"

; Truncate and start fresh
redirect "mybot.log" echo "=== New session ==="

; Write a longer block
redirect -append "mybot.log" echo "Extension: ISXPantheon ${ISXPantheon.Version}"
redirect -append "mybot.log" echo "API: ${ISXPantheon.APIVersion}"
```

### Per-purpose log files

```lavishscript
objectdef obj_MultiLogger
{
    variable string MainLog = "./logs/main.log"
    variable string ErrorLog = "./logs/errors.log"
    variable string MetricsLog = "./logs/metrics.log"

    method Initialize()
    {
        declare FP filepath "${Script.CurrentDirectory}"
        if !${FP.FileExists["logs"]}
            FP:MakeSubdirectory["logs"]
    }

    method LogMain(string msg)
    {
        redirect -append "${This.MainLog}" echo "[${Time.Time24}] ${msg}"
    }

    method LogError(string msg)
    {
        redirect -append "${This.MainLog}" echo "[${Time.Time24}] ERROR: ${msg}"
        redirect -append "${This.ErrorLog}" echo "[${Time.Time24}] ${msg}"
        echo "\ar${msg}\ax"
    }

    method LogMetric(string name, string value)
    {
        variable string line = "${name} = ${value}"
        redirect -append "${This.MetricsLog}" echo "[${Time.Time24}] ${line}"
    }
}

variable(script) obj_MultiLogger MultiLog
```

### Rotating log files

```lavishscript
objectdef obj_RotatingLogger
{
    variable string Current

    method Initialize()
    {
        declare FP filepath "${Script.CurrentDirectory}"
        if !${FP.FileExists["logs"]}
            FP:MakeSubdirectory["logs"]

        This.Current:Set["./logs/${Time.Year}${Time.Month.LeadingZeroes[2]}${Time.Day.LeadingZeroes[2]}_${Time.Hour.LeadingZeroes[2]}${Time.Minute.LeadingZeroes[2]}.log"]

        redirect "${This.Current}" echo "=== ${Time.Date} ${Time.Time24} ==="
    }
}
```

---

## Script Introspection via `${Script.*}`

LavishScript exposes runtime information through the `Script` object. Useful for diagnostics:

| Expression | Meaning |
|---|---|
| `${Script.RunningTime}` | milliseconds since the script started |
| `${Script.MemUsage}` | approximate memory usage (bytes) |
| `${Script.CurrentDirectory}` | directory the script is running from |
| `${Script.Filename}` | the script's filename (no extension) |
| `${Script.Paused}` | TRUE if paused |

### Example: uptime and memory snapshot

```lavishscript
function DumpScriptStats()
{
    variable float uptimeSec = ${Math.Calc[${Script.RunningTime} / 1000]}
    variable float memMB = ${Math.Calc[${Script.MemUsage} / 1048576]}

    echo "=== Script Stats ==="
    echo "Uptime:  ${uptimeSec.Precision[1]}s"
    echo "Memory:  ${memMB.Precision[1]} MB"
    echo "Paused:  ${Script.Paused}"
    echo "Cwd:     ${Script.CurrentDirectory}"
    echo "===================="
}
```

### Memory-growth monitoring

Periodic memory growth in long-running scripts almost always means a collection is growing unbounded. Track delta between samples:

```lavishscript
objectdef obj_MemoryWatch
{
    variable int64 Baseline = 0

    method Sample(string label)
    {
        variable int64 now = ${Script.MemUsage}

        if ${This.Baseline} == 0
        {
            This.Baseline:Set[${now}]
            echo "[MEM] ${label}: ${Math.Calc[${now}/1048576].Precision[1]} MB (baseline)"
            return
        }

        variable int64 delta = ${Math.Calc64[${now} - ${This.Baseline}]}
        variable float deltaMB = ${Math.Calc[${delta}/1048576]}

        echo "[MEM] ${label}: ${Math.Calc[${now}/1048576].Precision[1]} MB (Δ ${deltaMB.Precision[2]} MB)"
    }
}
```

Call `MemWatch:Sample["after pulse"]` periodically. Sustained positive delta after the first minute is a leak — look for a collection being populated but never cleared.

---

## Type Inspection and Variable Dumping

When a member returns something unexpected, check its type before assuming:

```lavishscript
echo "Type of ISXPantheon: ${ISXPantheon.Type}"                    ; isxpantheon
echo "Type of Version:     ${ISXPantheon.Version.Type}"            ; string
echo "Type of IsReady:     ${ISXPantheon.IsReady.Type}"            ; bool
```

### Dumping an object's state

`Script:Dump` and member walks are useful when an object looks wrong. Today the only datatype with working members is `${ISXPantheon}`:

```lavishscript
function DumpExtension()
{
    echo "=== ISXPantheon Dump ==="

    if !${ISXPantheon(exists)}
    {
        echo "(ISXPantheon TLO missing — extension not loaded)"
        return
    }

    echo "Version:      ${ISXPantheon.Version}"
    echo "APIVersion:   ${ISXPantheon.APIVersion}"
    echo "IsReady:      ${ISXPantheon.IsReady}"
    echo "======================="
}
```

Place a single call to a function like this wherever behavior diverges from expectations.

> **PLANNED — NOT YET IMPLEMENTED.** Game-object dumps are not possible yet because the game-data datatypes (`pantheon`, `entity`, `ability`, `quest`) are registered but expose no members, and `${Me}` / `${Radar}` exist only as reserved (commented-out) top-level objects. Once the game surface is implemented, the same dump pattern will apply to those objects; until then, dumping is limited to `${ISXPantheon}`.

---

## Performance Profiling

### Timestamp profiler

Wrap suspect operations in Start/End markers; accumulate totals and average call times.

```lavishscript
objectdef obj_Profiler
{
    variable collection:float StartTimes
    variable collection:float TotalTimes
    variable collection:int  CallCounts

    method Start(string op)
    {
        This.StartTimes:Set[${op}, ${Script.RunningTime}]
    }

    method End(string op)
    {
        if !${This.StartTimes.Element[${op}](exists)}
        {
            echo "Profiler: End without Start for ${op}"
            return
        }

        variable float durMs = ${Math.Calc[${Script.RunningTime} - ${This.StartTimes.Get[${op}]}]}

        if !${This.TotalTimes.Element[${op}](exists)}
        {
            This.TotalTimes:Set[${op}, 0]
            This.CallCounts:Set[${op}, 0]
        }

        This.TotalTimes:Set[${op}, ${Math.Calc[${This.TotalTimes.Get[${op}]} + ${durMs}]}]
        This.CallCounts:Set[${op}, ${Math.Calc[${This.CallCounts.Get[${op}]} + 1]}]

        if ${durMs} > 500
            echo "\ayPROFILER: slow ${op} = ${durMs.Precision[1]} ms\ax"
    }

    method PrintReport()
    {
        echo "=== Profiler Report ==="

        variable iterator it
        This.TotalTimes:GetIterator[it]

        if ${it:First(exists)}
        {
            do
            {
                variable string op = "${it.Key}"
                variable float total = ${it.Value}
                variable int   calls = ${This.CallCounts.Get[${op}]}
                variable float avg   = ${Math.Calc[${total} / ${calls}]}

                echo "${op}: calls=${calls} total=${total.Precision[1]}ms avg=${avg.Precision[2]}ms"
            }
            while ${it:Next(exists)}
        }

        echo "======================="
    }
}

variable(script) obj_Profiler Profiler
```

**Usage:**

```lavishscript
Profiler:Start["WorkBatch"]
call DoExpensiveWork
Profiler:End["WorkBatch"]

; ...

Profiler:PrintReport
```

### Pulse-budget tracking

A common pattern is tiered pulse timers (1s, 2s, 5s, 10s). When a pulse runs long, the next pulse is late. Detect this:

```lavishscript
variable(script) uint LastPulseTime = 0

function main()
{
    LastPulseTime:Set[${Script.RunningTime}]

    while 1
    {
        variable uint gap = ${Math.Calc[${Script.RunningTime} - ${LastPulseTime}]}

        if ${gap} > 2000 && ${LastPulseTime} > 0
            echo "\ayPulse gap ${gap} ms — main thread stalled\ax"

        call DoPulseWork
        LastPulseTime:Set[${Script.RunningTime}]
        wait 10                                                     ; 1 second
    }
}
```

For the tiered pulse pattern itself, see [05_Patterns_And_Best_Practices.md → Performance Patterns](05_Patterns_And_Best_Practices.md#performance-patterns).

---

## Session Validation Checklist

Before any first API access, verify the extension is in a usable state:

```lavishscript
function bool ValidateSession()
{
    echo "=== Session Validation ==="

    if !${ISXPantheon(exists)}
    {
        echo "FAIL: ISXPantheon TLO missing — extension not loaded"
        return FALSE
    }
    echo "OK:   ISXPantheon loaded"

    if !${ISXPantheon.IsReady}
    {
        echo "FAIL: ISXPantheon not ready"
        return FALSE
    }
    echo "OK:   ISXPantheon ready (v${ISXPantheon.Version})"

    echo "=========================="
    return TRUE
}
```

Call from the top of `main`:

```lavishscript
function main()
{
    while !${ISXPantheon.IsReady}
        wait 10

    while !${ValidateSession}
    {
        echo "Waiting for extension..."
        wait 20
    }

    ; Safe to proceed
}
```

> **PLANNED — NOT YET IMPLEMENTED.** A richer validation (character loaded, in-world, not transitioning between areas, alive) requires game-state surface that does not exist yet. When the local-player and world datatypes are implemented, this checklist will extend to those checks; today the only meaningful gate is `${ISXPantheon.IsReady}`.

---

## Feature Not Yet Implemented (the #1 issue today)

The single most common "bug" report against ISXPantheon today is **trying to use surface that is not yet implemented.** Because the guide files (and habits carried over from other InnerSpace extensions) reference a far larger API than ISXPantheon currently exposes, scripts frequently reach for members/TLOs/commands/events that simply return nothing.

### Symptom: a TLO or member silently returns blank / FALSE / 0

Most likely you referenced surface that exists only as a planned stub. Today the real surface is:

- **TLO `${ISXPantheon}`** — fully working (`Version`, `APIVersion`, `IsReady`, `GetCustomVariable[...]`, `GetCurrencyString[...]`, `Round[...]`, plus the custom-variable / install / reload methods).
- **TLO `${Login}`** — working at the login / realm-selection screen (login state, realm enumeration, login-screen buttons/input fields). Most members are blank away from the login screen.
- **TLOs `${CharSelect}` / `${CharCreate}`** — working, but **only at the matching scene**. They return **NULL** anywhere else, so guard with `${CharSelect(exists)}` / `${CharCreate(exists)}` before reading members. A blank read here usually means you are simply not at that scene yet.
- **TLO `${Pantheon}`** — working render/camera surface (`ResolutionWidth`/`ResolutionHeight`, `RenderQuality`, `FPSLimit`, `VSyncCount`, `FullScreenMode`, `FrameCount`, `NumCameras`, `Camera[#]`, plus `RestoreCameras` / `SetFPSLimit`). Its **in-world game-data** members are still planned.
- **Datatypes `entity`, `ability`, `quest`** — registered but **expose no members**. A variable of these types holds an id, but every member you try to read is unimplemented.
- **Commands `GetURL`, `PostURL`** — working.
- **No game events fire yet.**

### Diagnostic: confirm the TLO/member actually exists

```lavishscript
; Confirm the working TLO is present and ready first
if !${ISXPantheon(exists)} || !${ISXPantheon.IsReady}
{
    echo "Extension not loaded/ready — nothing will work yet"
    return
}

; If a member returns blank, check whether it is part of the real surface.
; Only ${ISXPantheon.*} members return data today.
echo "Version:    ${ISXPantheon.Version}"
echo "APIVersion: ${ISXPantheon.APIVersion}"
```

> **PLANNED — NOT YET IMPLEMENTED.** The game-data surface is on the ISXPantheon roadmap but unavailable today: the `pantheon`, `entity`, `ability`, and `quest` datatypes are registered with no working members; `${Me}` and `${Radar}` exist only as reserved (commented-out) top-level objects; and a `Where` command, a `Radar` command, and a `Crafting` TLO/datatype are named on the roadmap. If a script needs any game-data feature, it cannot run against the current build — the surface simply is not wired yet. Track this via the planned-feature notes in `03_API_Reference.md` rather than assuming it exists.

### Rule of thumb

If a feature is not documented as **real** in `03_API_Reference.md`, assume it is **not implemented** and design around its absence — do not write code that depends on it "probably working."

---

## Waiting on `${ISXPantheon.IsReady}`

`${ISXPantheon.IsReady}` is the single most important gate in any ISXPantheon script. The `${ISXPantheon}` TLO is registered the moment the extension DLL loads, but its data (and any future game surface) is not usable until the extension finishes initializing. `IsReady` flips TRUE at that point.

### The member

`${ISXPantheon.IsReady}` returns a `bool`:

- `FALSE` — extension still initializing; do not access other members yet
- `TRUE`  — extension fully loaded; safe to proceed

### Symptom: first API access returns blank right after loading the extension

You accessed a member before `IsReady` was TRUE. Always gate the start of `main`:

```lavishscript
function main()
{
    ; Wait for the extension to finish loading
    while !${ISXPantheon.IsReady}
        wait 10

    echo "ISXPantheon ${ISXPantheon.Version} ready"

    ; ... safe to proceed ...
}
```

### Bounded wait (don't spin forever)

```lavishscript
variable int waited = 0
while !${ISXPantheon.IsReady} && ${waited} < 300
{
    wait 10                                                        ; 1 second
    waited:Inc
}

if !${ISXPantheon.IsReady}
{
    echo "ISXPantheon never became ready — is the extension loaded? (extension isxpantheon)"
    return
}
```

If `IsReady` never flips TRUE, the extension probably is not loaded at all — confirm with `${ISXPantheon(exists)}` and that you ran `extension isxpantheon`.

---

## Custom Variables Returning Blank

The `${ISXPantheon}` TLO provides a small key/value custom-variable store, useful for sharing state across scripts or persisting simple values. A blank read is almost always one of a few mistakes.

### The members and methods

- `${ISXPantheon.GetCustomVariable[name]}` — read a custom variable (string)
- `${ISXPantheon.GetCustomVariable[name,str|int|uint|bool|float]}` — read it as a typed value
- `ISXPantheon:SetCustomVariable[name,value]` — set one
- `ISXPantheon:ClearAllCustomVariables` — clear them all

### Symptom: `GetCustomVariable` returns blank

Root causes, in order of frequency:

1. **Never set.** You read a name that was never written with `SetCustomVariable`.
2. **Name case/spelling mismatch** between the `Set` and the `Get`.
3. **Cleared.** Something called `ClearAllCustomVariables` (or the extension reloaded).
4. **Type mismatch.** You requested a typed read (`,int`) on a value that is not parseable as that type.

### Diagnostic

```lavishscript
ISXPantheon:SetCustomVariable[myKey, 42]

echo "raw:   ${ISXPantheon.GetCustomVariable[myKey]}"              ; 42
echo "asInt: ${ISXPantheon.GetCustomVariable[myKey,int]}"          ; 42
echo "asBad: ${ISXPantheon.GetCustomVariable[myKey,bool]}"         ; blank/false — not a bool
```

If the raw read works but the typed read is blank, the stored value is not parseable as the requested type — store it in the type you intend to read.

---

## GetURL / PostURL Troubleshooting

`GetURL` and `PostURL` are the two general-purpose commands ISXPantheon compiles into the public build. They perform HTTP requests from a script.

### Symptom: request appears to do nothing

Root causes, in order of frequency:

1. **No network / blocked host.** The target host is unreachable or blocked by a firewall.
2. **Malformed URL.** Missing scheme (`http://` / `https://`) or unescaped characters.
3. **Reading the result too early.** If the command delivers its result asynchronously (e.g. via a registered event or a written file), reading before completion yields nothing — wait for completion first.

### Diagnostic

```lavishscript
function main()
{
    while !${ISXPantheon.IsReady}
        wait 10

    echo "Issuing GetURL..."
    GetURL "https://example.com/"

    ; Give the request time to complete before inspecting any result
    wait 50
}
```

If the request never completes, test basic connectivity outside InnerSpace first (a browser or `curl` to the same URL), then re-check the URL string for typos and proper escaping.

---

## LGUI2 Element-Not-Found Diagnosis

LGUI2 uses a JSON element tree accessed via `LGUI2.Element[...]`. Unlike LGUI1, element paths are more structured, and typos fail more silently.

### Symptom: `${LGUI2.Element["MyButton"](exists)}` returns FALSE

Root causes:

1. **Package not loaded yet.** LGUI2 packages load asynchronously. Check immediately after `LGUI2:LoadPackageFile` and you may see FALSE.
2. **Element name mismatch.** LGUI2 names are case-sensitive and exact.
3. **Element is a child, not top-level.** `MyButton` inside `MyWindow` is `MyButton@MyWindow` in LGUI1 syntax, or requires `.FindChild` navigation in LGUI2.
4. **Package was unloaded** (e.g., by `LGUI2:UnloadPackageFile`).

### Diagnostic

```lavishscript
function DiagnoseLGUI2Element(string name)
{
    echo "=== LGUI2 Diagnostic: ${name} ==="

    if ${LGUI2.Element[${name}](exists)}
    {
        echo "OK:   element exists at top level"
        echo "      Type: ${LGUI2.Element[${name}].Type}"
        echo "      Visible: ${LGUI2.Element[${name}].Visible}"
        return
    }

    echo "FAIL: element '${name}' not found at top level"
    echo "HINT: check package loaded, case matches, and element is top-level (not a descendant)"
}
```

### Waiting for package load

```lavishscript
LGUI2:LoadPackageFile["MyUI.json"]

variable int waited = 0
while !${LGUI2.Element[MainWindow](exists)} && ${waited} < 100
{
    waitframe
    waited:Inc
}

if !${LGUI2.Element[MainWindow](exists)}
{
    Log:Echo["UI package failed to load within 100 frames"]
    return
}
```

See [10_LavishGUI2_UI_Guide.md → Troubleshooting](10_LavishGUI2_UI_Guide.md#troubleshooting) for more LGUI2-specific issues.

---

## Systematic Issue Diagnosis Workflow

Follow this ordered workflow when a script misbehaves:

- [ ] **Check the surface first** — is the feature you're using actually implemented? See [Feature Not Yet Implemented](#feature-not-yet-implemented-the-1-issue-today). This is the most common cause today.
- [ ] **Reproduce** — can you make it happen on demand? What are the exact steps?
- [ ] **Isolate** — does it happen only in certain states (before vs after `IsReady`, certain inputs)?
- [ ] **Log** — add `Log:Echo` around the misbehavior site; log inputs, outputs, and state transitions
- [ ] **Validate** — confirm `${ISXPantheon(exists)}` and `${ISXPantheon.IsReady}` before any API access
- [ ] **Dump state** — call a full variable/`${ISXPantheon}` dump at the point of failure
- [ ] **Simplify** — comment out peripheral logic; can you reproduce with a 10-line test script?
- [ ] **Hypothesize** — which invariant is violated? Write it down before testing.
- [ ] **Fix and verify** — re-run the test script, then the full script, then under extended runtime

---

## Performance Investigation Workflow

- [ ] **Measure first** — add `${Script.RunningTime}` bookends to suspect regions; never optimize without measurement
- [ ] **Profile** — use `obj_Profiler` (see [section 8](#performance-profiling)) to identify the top-3 time consumers
- [ ] **Sample memory** — track `${Script.MemUsage}` over time; flat is fine, growing is a leak
- [ ] **Throttle hot loops** — reduce work frequency; use tiered pulse timers (1s / 2s / 5s / 10s)
- [ ] **Cache aggressively** — for data that changes slowly, cache and invalidate on a clear trigger
- [ ] **Re-measure** — confirm the optimization moved the needle
- [ ] **Monitor long-term** — run for hours; watch for drift

---

## State-Machine Stuck Detection

A long-running script can wedge in a single state (it hasn't changed state in X minutes). Detect and recover from that with a small state-watch object. This is pure LavishScript and does not depend on any game surface.

```lavishscript
objectdef obj_StateStuckDetector
{
    variable string LastState
    variable uint   LastStateChange = 0
    variable uint   StuckThresholdMs = 300000                      ; 5 minutes

    method Note(string currentState)
    {
        if !${currentState.Equal[${This.LastState}]}
        {
            This.LastState:Set[${currentState}]
            This.LastStateChange:Set[${Script.RunningTime}]
            return
        }

        variable uint heldMs = ${Math.Calc[${Script.RunningTime} - ${This.LastStateChange}]}

        if ${heldMs} > ${This.StuckThresholdMs}
        {
            Log:Echo["STATE STUCK: in '${currentState}' for ${Math.Calc[${heldMs}/60000].Int} min"]
            This:Recover["${currentState}"]
            This.LastStateChange:Set[${Script.RunningTime}]
        }
    }

    method Recover(string state)
    {
        Log:Echo["Attempting recovery from state: ${state}"]

        ; Script-specific recovery logic
        switch ${state}
        {
            case Working
                ; Reset the work subsystem
                This.LastState:Set["Idle"]
                break
            case Waiting
                ; Re-issue the pending request
                This.LastState:Set["Idle"]
                break
            default
                ; Fall back to idle
                This.LastState:Set["Idle"]
                break
        }
    }
}

variable(script) obj_StateStuckDetector StuckDetector
```

Call `StuckDetector:Note["${CurrentState}"]` every main loop.

---

## Dead Man's Switch and Periodic Restart

### Dead man's switch - abort after no user input

```lavishscript
objectdef obj_DeadManSwitch
{
    variable int   LastMouseX = 0
    variable int   LastMouseY = 0
    variable uint  LastInputTime = 0
    variable int   TimeoutMinutes = 30

    method Check()
    {
        if ${Mouse.X} != ${This.LastMouseX} || ${Mouse.Y} != ${This.LastMouseY}
        {
            This.LastMouseX:Set[${Mouse.X}]
            This.LastMouseY:Set[${Mouse.Y}]
            This.LastInputTime:Set[${Script.RunningTime}]
        }

        variable float idleMin = ${Math.Calc[(${Script.RunningTime} - ${This.LastInputTime}) / 60000]}

        if ${idleMin} > ${This.TimeoutMinutes}
        {
            Log:Echo["Dead-man's switch: ${idleMin.Precision[0]} min with no input — halting"]
            echo "\arAFK timeout — script halting\ax"
            Script:Pause
        }
    }
}
```

### Periodic restart - bound total runtime

```lavishscript
function main()
{
    variable int maxHours = 8

    while 1
    {
        call DoWork

        variable float hoursRunning = ${Math.Calc[${Script.RunningTime} / 1000 / 60 / 60]}

        if ${hoursRunning} >= ${maxHours}
        {
            Log:Echo["Max runtime ${maxHours}h reached — exiting for restart"]
            Script:End
        }

        wait 10
    }
}
```

---

## Config Backup and Restore

Scripts that write config (LavishSettings XML, JSON files) risk corruption on crash. Back up before writing.

```lavishscript
objectdef obj_SafeConfig
{
    variable string ConfigFile   = "${Script.CurrentDirectory}/config.xml"
    variable string BackupFile   = "${Script.CurrentDirectory}/config.backup.xml"

    method Save(string settingsSetName)
    {
        declare FP filepath "${Script.CurrentDirectory}"

        ; Back up current good config
        if ${FP.FileExists["config.xml"]}
        {
            redirect "${This.BackupFile}" cat "${This.ConfigFile}"
        }

        LavishSettings[${settingsSetName}]:Export["${This.ConfigFile}"]

        Log:Echo["Config saved (backup: ${This.BackupFile})"]
    }

    method Load(string settingsSetName)
    {
        declare FP filepath "${Script.CurrentDirectory}"

        if !${FP.FileExists["config.xml"]}
        {
            Log:Echo["Config missing — checking backup"]

            if ${FP.FileExists["config.backup.xml"]}
            {
                redirect "${This.ConfigFile}" cat "${This.BackupFile}"
                Log:Echo["Restored from backup"]
            }
            else
            {
                Log:Echo["No backup — using defaults"]
                return
            }
        }

        LavishSettings[${settingsSetName}]:Import["${This.ConfigFile}"]
        Log:Echo["Config loaded"]
    }
}
```

For a canonical LavishSettings treatment see [15_Advanced_Scripting_Patterns.md → LavishSettings Configuration Management](15_Advanced_Scripting_Patterns.md#lavishsettings-configuration-management).

---

## Relay Event Debugging

Cross-session relay events fail most often because of name mismatches, missing `RegisterEvent`, or wrong parameter quoting.

### Checklist

- [ ] Event name is **identical** in `relay`, `RegisterEvent`, and `AttachAtom` (case-sensitive)
- [ ] Receiver called `LavishScript:RegisterEvent[EventName]` before the relay fires
- [ ] Receiver called `Event[EventName]:AttachAtom[HandlerAtom]` after registering
- [ ] String parameters are **quoted** in the `relay` command, and atom signature matches types
- [ ] Handler atom is declared `atom(script)` if it should survive past the function that attached it

### Diagnostic - list registered events

```lavishscript
function ListRegisteredEvents()
{
    echo "=== Registered Events ==="
    variable iterator it
    LavishScript:GetRegisteredEvents[it]
    if ${it:First(exists)}
    {
        do
            echo "  ${it.Value}"
        while ${it:Next(exists)}
    }
    echo "========================="
}
```

### Parameter quoting

```lavishscript
; WRONG — value with space becomes first word only
relay all -event StatusUpdate ${This.StatusText}

; RIGHT — quote string parameters
relay all -event StatusUpdate "${This.StatusText}"

; Multiple params
relay all -event WorkerInfo "${This.WorkerName}" ${This.Progress} ${This.Iteration}
```

### Receiver

```lavishscript
atom(script) OnWorkerInfo(string Name, float Progress, int64 Iteration)
{
    Log:Echo["Received: ${Name} at ${Progress}% (iter=${Iteration})"]
}

function main()
{
    LavishScript:RegisterEvent[WorkerInfo]
    Event[WorkerInfo]:AttachAtom[OnWorkerInfo]

    while 1
        waitframe
}
```

---

## Top-10 Checklist

Quick checklist to run through when any script misbehaves:

- [ ] Is the feature I'm using actually implemented? (Most common cause today — see [Feature Not Yet Implemented](#feature-not-yet-implemented-the-1-issue-today).)
- [ ] Did I `wait` for `${ISXPantheon.IsReady}` before the first API access?
- [ ] Did I confirm `${ISXPantheon(exists)}` (extension actually loaded via `extension isxpantheon`)?
- [ ] Am I using `(exists)` before every object member access?
- [ ] Did I set a custom variable before reading it, with matching name and type?
- [ ] Does every `while` / `do ... while` loop contain a `wait` of at least 1?
- [ ] Are events `RegisterEvent`'d before any `relay -event` fires them?
- [ ] Are long-lived atoms declared `atom(script)`?
- [ ] Do I `Clear` collections periodically to avoid memory growth?
- [ ] Did I bound `GetURL` / `PostURL` waits and verify connectivity outside InnerSpace?
- [ ] Have I saved config via `:Export` / `:Save` after changes, with a backup?

For full anti-pattern treatment, see [05_Patterns_And_Best_Practices.md → Common Anti-Patterns](05_Patterns_And_Best_Practices.md#common-anti-patterns). For beginner troubleshooting, see [02_Quick_Start_Guide.md → Troubleshooting](02_Quick_Start_Guide.md#troubleshooting) and the [Quick Troubleshooting table](00_MASTER_GUIDE.md#quick-troubleshooting) in the master guide.
