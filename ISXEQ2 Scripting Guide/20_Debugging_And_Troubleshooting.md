# Debugging and Troubleshooting

**Purpose:** Diagnostic techniques, logging strategy, performance profiling, and EQ2-specific problem catalog for ISXEQ2 scripts.
**Audience:** Script authors troubleshooting misbehavior, tuning performance, or hardening long-running automation.

This guide is a **capstone** that consolidates debugging and troubleshooting. It leans on the EQ2-native `Debug:` object (from `EQ2Common/Debug.iss`), LavishScript-native `redirect`, and `${Script.*}` introspection rather than reinventing a logger from scratch. Foundational existence checks, async-data patterns, anti-patterns, and stuck-detection are covered canonically elsewhere and cross-linked rather than duplicated.

---

## Table of Contents

### Core Diagnostic Toolkit
1. [The ISXEQ2 Debug-Tool Landscape](#the-isxeq2-debug-tool-landscape)
2. [The `Debug:` Built-in Object](#the-debug-built-in-object)
3. [Echo, eq2echo, and Console Output](#echo-eq2echo-and-console-output)
4. [Verbosity Tiers and Conditional Logging](#verbosity-tiers-and-conditional-logging)
5. [File-Based Logging with `redirect`](#file-based-logging-with-redirect)
6. [Script Introspection via `${Script.*}`](#script-introspection-via-script)
7. [Type Inspection and Variable Dumping](#type-inspection-and-variable-dumping)
8. [Performance Profiling](#performance-profiling)
9. [Session Validation Checklist](#session-validation-checklist)

### EQ2-Specific Common Problems
10. [Zoning: the `${EQ2.Zoning}` Pattern](#zoning-the-eq2zoning-pattern)
11. [Ability Casting Problems](#ability-casting-problems)
12. [Async Data Loading Failures](#async-data-loading-failures)
13. [Actor Queries and Deprecated CustomActorArray](#actor-queries-and-deprecated-customactorarray)
14. [EQ2Bot Class-Routine `#include` Failures](#eq2bot-class-routine-include-failures)
15. [Maintained Effects vs Buff Checks](#maintained-effects-vs-buff-checks)
16. [Inventory Queries After Bag Rearrangement](#inventory-queries-after-bag-rearrangement)
17. [ExactName Case Sensitivity and String Comparison](#exactname-case-sensitivity-and-string-comparison)
18. [LGUI1 `_executeOnSelect` Recursion and UI Timing](#lgui1-_executeonselect-recursion-and-ui-timing)
19. [LGUI2 Element-Not-Found Diagnosis](#lgui2-element-not-found-diagnosis)

### Reliability and Resilience
20. [Systematic Issue Diagnosis Workflow](#systematic-issue-diagnosis-workflow)
21. [Performance Investigation Workflow](#performance-investigation-workflow)
22. [Stuck Detection and Recovery](#stuck-detection-and-recovery)
23. [Dead Man's Switch and Periodic Restart](#dead-mans-switch-and-periodic-restart)
24. [Config Backup and Restore](#config-backup-and-restore)
25. [Relay Event Debugging](#relay-event-debugging)
26. [Top-10 Checklist](#top-10-checklist)

---

## The ISXEQ2 Debug-Tool Landscape

Unlike ISXEVE (which exposes `ISXEVE.Debug_LogMsg`, `ISXEVE.IsSafe`, `ISXEVE.LastError`, and `ISXEVE.Lag` as TLO members), **ISXEQ2 does not expose any TLO-level debug helpers.** The only ISXEQ2-TLO debug-relevant members are:

- `${ISXEQ2.IsReady}` — gate before first API access
- `${ISXEQ2.Version}` — version string for bug reports

If you came here looking for an `ISXEQ2.Debug_LogMsg`, `ISXEQ2.IsSafe`, or `ISXEQ2.LastError`, they do not exist. Use these instead:

- **`Debug:` built-in object** (from `EQ2Common/Debug.iss`) — the idiomatic ISXEQ2 logger. See [section 2](#the-debug-built-in-object).
- **LavishScript-native `redirect`** for raw file output. See [section 5](#file-based-logging-with-redirect).
- **`${Script.*}` introspection** (`RunningTime`, `MemUsage`, `CurrentDirectory`, `Filename`) for runtime state. See [section 6](#script-introspection-via-script).
- **Type inspection** (`${Obj.Type}`, `Script:Dump`) for variable introspection. See [section 7](#type-inspection-and-variable-dumping).

For **foundational patterns** — existence checks, async data loading, NULL safety, iterator patterns — see:
- [04_Core_Concepts.md → NULL Checks and Existence Validation](04_Core_Concepts.md#null-checks-and-existence-validation)
- [04_Core_Concepts.md → Asynchronous Data Loading](04_Core_Concepts.md#asynchronous-data-loading)
- [05_Patterns_And_Best_Practices.md → Error Handling Strategies](05_Patterns_And_Best_Practices.md#error-handling-strategies)
- [05_Patterns_And_Best_Practices.md → Common Anti-Patterns](05_Patterns_And_Best_Practices.md#common-anti-patterns)

---

## The `Debug:` Built-in Object

`EQ2Common/Debug.iss` defines a `debug` datatype with a global `Debug` instance. This is the idiomatic logging tool across EQ2Craft, EQ2RaidAttendance, EQ2AFKAlarm, EQ2Bot (main script and utilities), and many others. Prefer it over rolling your own logger.

### API

**Members:**

- `Enabled` (bool) — whether logging is active
- `TimestampEcho` (bool) — whether `Debug:Echo` prepends a timestamp (default TRUE)
- `TimestampLog` (bool) — whether `Debug:Log` prepends a timestamp (default TRUE)

**Methods:**

- `Enable` — turn logging on (default is off)
- `Disable` — turn logging off
- `Echo[...]` — echo a timestamped line to the console (respects `Enabled`)
- `Log[...]` — append a timestamped line to the log file (respects `Enabled`)
- `SetFilename[string]` — set the log file path (default: `${Script.CurrentDirectory}/${Script.Filename}.txt`)
- `SetPrefix[string]` — set a prefix for all output (default: `"DEBUG: "`; pass empty brackets `[]` to clear)
- `TimestampEcho[bool]` — toggle echo timestamps
- `TimestampLog[bool]` — toggle log timestamps
- `SetEchoAlsoLogs[bool]` — if TRUE, every `Debug:Echo` also writes to the log file
- `ClearLog` — truncate the log file

### Canonical setup

```lavishscript
#include EQ2Common/Debug.iss

function main()
{
    Debug:Enable
    Debug:SetPrefix[]                                              ; no prefix
    Debug:SetEchoAlsoLogs[TRUE]                                    ; echo mirrors to file
    Debug:SetFilename["${LavishScript.HomeDirectory}/Scripts/MyScript/MyScript_Debug.txt"]

    Debug:Echo["-------- Session start --------"]
    Debug:Echo["(${Time.Date} at ${Time.Time24})"]

    Debug:Echo["Script initialized"]
}
```

### Conditional debug through a script-scoped flag

```lavishscript
variable(script) bool EnableDebug = FALSE

function main()
{
    Script:DisableDebugging                                        ; drop the LavishScript-level debugger
    Debug:SetFilename["${LavishScript.HomeDirectory}/Scripts/MyScript/MyScript_Debug.txt"]

    if ${EnableDebug}
        Debug:Enable

    Debug:Echo["Starting..."]                                      ; only appears if enabled
}
```

When `Debug:` is disabled, `Debug:Echo` and `Debug:Log` are no-ops — safe to leave sprinkled throughout production code. This is the EQ2-native equivalent of an "`if DEBUG` guard" without the explicit guards.

### EQ2RaidAttendance-style logfile session header

```lavishscript
#include EQ2Common/Debug.iss

function main()
{
    Debug:Enable
    Debug:SetPrefix[]
    Debug:SetEchoAlsoLogs[TRUE]
    Debug:SetFilename["EQ2RaidAttendance.txt"]

    Debug:Echo["-------- EQ2 Raid Attendance --------"]
    Debug:Echo["(Created ${Time.Date} at ${Time})"]
    Debug:Echo["-"]

    Debug:Echo["- Initializing..."]
    ; ... work ...
    Debug:Echo["-----------------------------------"]
    Debug:Log["\n\n\n"]                                            ; blank lines between sessions
}
```

### Layering severity on top of `Debug:`

`Debug:` itself has no concept of levels. For severity, wrap it in atoms:

```lavishscript
variable(script) bool EnableDebug = FALSE
variable(script) bool Quiet = FALSE

atom(script) LogInfo(... Params)
{
    if !${Params.Size}
        return
    Debug:Echo["[INFO] ${Params.Expand}"]
}

atom(script) LogWarn(... Params)
{
    if !${Params.Size}
        return
    Debug:Echo["[WARN] ${Params.Expand}"]
}

atom(script) LogError(... Params)
{
    if !${Params.Size} || ${Quiet}
        return
    Debug:Echo["[ERROR] ${Params.Expand}"]                         ; errors ignore EnableDebug via own path
    eq2echo "\#FF4444[ERROR]\#FFFFFF ${Params.Expand}"             ; also surface in-game
}

atom(script) LogDebug(... Params)
{
    if !${EnableDebug} || !${Params.Size}
        return
    Debug:Echo["[DEBUG] ${Params.Expand}"]
}
```

Call these from anywhere in the script — `LogInfo "Bot starting"`, `LogError "Failed to cast ${SpellName}"`, etc. The `... Params` plus `${Params.Expand}` pattern is the same one used by EQ2Craft's `ChatEcho`/`ErrorEcho`. See [07_Advanced_Patterns_And_Examples.md → Debugging and Logging](07_Advanced_Patterns_And_Examples.md#debugging-and-logging) for the full EQ2Craft and EQ2RaidAttendance patterns.

### When to NOT use `Debug:`

- When you want output even when debugging is disabled — use `echo` or `eq2echo` directly.
- When you need per-subsystem log files — roll your own with `redirect -append` (see [section 5](#file-based-logging-with-redirect)).
- When you need sub-second-precision timing — `Debug:` timestamps are `${Time.Time24}` (HH:MM:SS); use `${Script.RunningTime}` (ms since script start) or `${Time.Timestamp}` (seconds since epoch) directly.

---

## Echo, eq2echo, and Console Output

Two output streams exist:

- **`echo`** — writes to the InnerSpace console only. Developer-visible, invisible to the player.
- **`eq2echo`** — writes to the EverQuest 2 chat window. Visible in-game.

```lavishscript
echo "Debug: Health = ${Me.CurrentHealth}"                         ; InnerSpace console
eq2echo "Heal incoming!"                                           ; EQ2 chat window
```

### Color codes

InnerSpace console color codes (use with `echo`, `eq2echo`, and `Debug:Echo`):

```lavishscript
echo "\agGreen success\ax"
echo "\arRed error\ax"
echo "\ayYellow warning\ax"
echo "\aoOrange alert\ax"
echo "\awWhite (default)\ax"
```

For `eq2echo`, EQ2 hex color codes also work:

```lavishscript
eq2echo "\#00FF00Green success\#FFFFFF"
eq2echo "\#FF4444Red error\#FFFFFF"
eq2echo "\#FFFF00Yellow warning\#FFFFFF"
```

### Deciding which stream

| Purpose | Stream | Visibility |
|---|---|---|
| Developer diagnostics | `echo` or `Debug:Echo` | InnerSpace console only |
| User-facing status | `eq2echo` | EQ2 chat window |
| Persistent record | `Debug:Log` or `redirect -append` | File on disk |
| Critical alert | `eq2echo` + audio (`play`) | Chat + sound |

---

## Verbosity Tiers and Conditional Logging

For scripts with noisy diagnostic output, tier your verbosity. Pattern from EQ2Craft:

```lavishscript
variable(script) bool V1 = FALSE                                   ; verbose
variable(script) bool V2 = FALSE                                   ; very verbose
variable(script) bool EchoInChat = FALSE                           ; also route to EQ2 chat

atom(script) ChatEcho(... Params)
{
    if !${Params.Size}
        return

    if ${EchoInChat} && ${V1}
        eq2echo ${Params.Expand}
    elseif ${V1}
        echo ${Params.Expand}
}
```

Flip `V1` / `V2` at runtime from the command line or a UI checkbox to elevate verbosity without restarting the script.

---

## File-Based Logging with `redirect`

`Debug:` writes to a single file. For multiple log streams (main, errors, combat, metrics) or rotation, use `redirect` directly.

### Basic usage

```lavishscript
; Append one line
redirect -append "mybot.log" echo "[${Time.Time24}] Started"

; Truncate and start fresh
redirect "mybot.log" echo "=== New session ==="

; Write a longer block
redirect -append "mybot.log" echo "Character: ${Me.Name} (${Me.SubClass} ${Me.Level})"
redirect -append "mybot.log" echo "Zone: ${Zone.Name}"
```

### Per-purpose log files

```lavishscript
objectdef obj_MultiLogger
{
    variable string MainLog = "./logs/${Me.Name}_main.log"
    variable string ErrorLog = "./logs/${Me.Name}_errors.log"
    variable string CombatLog = "./logs/${Me.Name}_combat.log"

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

    method LogCombat(int ActorID, string action)
    {
        variable string line = "${action} -> ${Actor[${ActorID}].Name} (${ActorID})"
        redirect -append "${This.CombatLog}" echo "[${Time.Time24}] ${line}"
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

        This.Current:Set["./logs/${Me.Name}_${Time.Year}${Time.Month.LeadingZeroes[2]}${Time.Day.LeadingZeroes[2]}_${Time.Hour.LeadingZeroes[2]}${Time.Minute.LeadingZeroes[2]}.log"]

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
echo "Type of Me:      ${Me.Type}"                                 ; character
echo "Type of Target:  ${Target.Type}"                             ; actor
echo "Type of item:    ${Me.Inventory[5].Type}"                    ; item
```

### Dumping an object's state

`Script:Dump` and member walks are useful when an object looks wrong:

```lavishscript
function DumpTarget()
{
    echo "=== Target Dump ==="

    if !${Target(exists)}
    {
        echo "(no target)"
        return
    }

    echo "Name:         ${Target.Name}"
    echo "ID:           ${Target.ID}"
    echo "Type:         ${Target.Type}"
    echo "Level:        ${Target.Level}"
    echo "Distance:     ${Target.Distance}"
    echo "Health:       ${Target.Health}%"
    echo "IsDead:       ${Target.IsDead}"
    echo "IsAggro:      ${Target.IsAggro}"
    echo "IsNamed:      ${Target.IsNamed}"
    echo "IsEpic:       ${Target.IsEpic}"
    echo "==================="
}
```

Place a single call to a function like this wherever behavior diverges from expectations. For actor-info dependent fields, wait on `${Target.IsActorInfoAvailable}` first — see [section 12](#async-data-loading-failures) and [04_Core_Concepts.md → Asynchronous Data Loading](04_Core_Concepts.md#asynchronous-data-loading).

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
Profiler:Start["ActorQuery"]
EQ2:GetActors[Actors, Range, 50, NPC]
Profiler:End["ActorQuery"]

; ...

Profiler:PrintReport
```

### Pulse-budget tracking

EQ2Bot uses tiered pulse timers (1s, 2s, 5s, 10s). When a pulse runs long, the next pulse is late. Detect this:

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

Before any first API access, verify the session is in a usable state:

```lavishscript
function bool ValidateSession()
{
    echo "=== Session Validation ==="

    if !${ISXEQ2.IsReady}
    {
        echo "FAIL: ISXEQ2 not ready"
        return FALSE
    }
    echo "OK:   ISXEQ2 ready (v${ISXEQ2.Version})"

    if !${Me(exists)}
    {
        echo "FAIL: Character not loaded"
        return FALSE
    }
    echo "OK:   Character: ${Me.Name} (${Me.SubClass} ${Me.Level})"

    if !${Zone(exists)}
    {
        echo "FAIL: Zone not loaded"
        return FALSE
    }
    echo "OK:   Zone: ${Zone.Name}"

    if ${EQ2.Zoning} != 0
    {
        echo "FAIL: Zoning in progress (EQ2.Zoning=${EQ2.Zoning})"
        return FALSE
    }
    echo "OK:   Not zoning"

    if ${Me.IsDead}
    {
        echo "FAIL: Character is dead"
        return FALSE
    }
    echo "OK:   Alive"

    echo "=========================="
    return TRUE
}
```

Call from the top of `main`:

```lavishscript
function main()
{
    while !${ISXEQ2.IsReady}
        wait 10

    while !${ValidateSession}
    {
        echo "Waiting for session..."
        wait 20
    }

    ; Safe to proceed
}
```

---

## Zoning: the `${EQ2.Zoning}` Pattern

`${EQ2.Zoning}` is the single most important EQ2-specific guard. Miss it and your script will fail unpredictably every time the character zones.

### The member

`${EQ2.Zoning}` returns an `int`:

- `-1` — unsure (rare; treat as "maybe zoning")
- `0`  — not zoning
- `1`  — zoning in progress

Idiomatic check (EQ2Bot uses this 13+ times in its main loop):

```lavishscript
while ${EQ2.Zoning} != 0
    wait 10
```

### The two zoning events

| Event | Signature |
|---|---|
| `EQ2_StartedZoning` | no parameters |
| `EQ2_FinishedZoning` | `string TimeInSeconds` |

Note: `TimeInSeconds` is a **string**, not a numeric type — coerce with `.Float` if you need to compare:

```lavishscript
atom(script) OnFinishedZoning(string TimeInSeconds)
{
    Debug:Echo["Finished zoning in ${TimeInSeconds}s"]

    if ${TimeInSeconds.Float} > 10
        Debug:Echo["Slow zone (>10s) — server may be laggy"]

    ; Refresh cached state that may be stale after zone
    This.CachedActorIDs:Clear
    This.LastQueryTime:Set[0]
}

function main()
{
    LavishScript:RegisterEvent[EQ2_FinishedZoning]
    Event[EQ2_FinishedZoning]:AttachAtom[OnFinishedZoning]

    ; ... main loop ...
}
```

### What breaks if you skip the zoning check

- **Actor queries return empty** during zoning. `EQ2:GetActors` returns an empty index; iterators look valid but have zero elements.
- **Ability casts silently fail.** `Me.Ability[...]:Use` during zoning does nothing.
- **`${Target(exists)}` can return stale TRUE** briefly as the old target references linger.
- **UI windows disappear and reappear.** Any `UIElement[...]` reference from before the zone is invalid until the UI reloads.

### Canonical pattern: pre-action zoning guard

```lavishscript
function bool SafeToAct()
{
    if !${ISXEQ2.IsReady}
        return FALSE

    if ${EQ2.Zoning} != 0
        return FALSE

    if ${Me.IsDead}
        return FALSE

    return TRUE
}

while 1
{
    if !${SafeToAct}
    {
        wait 10
        continue
    }

    call DoPulseWork
    wait 10
}
```

---

## Ability Casting Problems

### Symptom: `Me.Ability[...]:Use` silently does nothing

Root causes, in order of frequency:

1. **Ability name case/spelling mismatch.** EQ2 ability names are exact, case-sensitive strings.
2. **Ability not ready** — `IsReady` is FALSE (on cooldown or already casting).
3. **Global cooldown active** — another ability was just cast.
4. **Character zoning** — see [section 10](#zoning-the-eq2zoning-pattern).
5. **No target for a targeted ability.**
6. **Out of power.**

### Diagnostic helper

```lavishscript
function DiagnoseCast(string AbilityName)
{
    echo "=== Cast Diagnostic: ${AbilityName} ==="

    if !${Me.Ability[${AbilityName}](exists)}
    {
        echo "FAIL: Ability '${AbilityName}' not found — check spelling and case"
        echo "HINT: iterate ${Me.NumAbilities} and log names to confirm"
        return
    }

    echo "OK:   Ability exists"
    echo "      IsReady:        ${Me.Ability[${AbilityName}].IsReady}"
    echo "      IsQueued:       ${Me.Ability[${AbilityName}].IsQueued}"
    echo "      TimeUntilReady: ${Me.Ability[${AbilityName}].TimeUntilReady}ms"
    echo "      PowerCost:      ${Me.Ability[${AbilityName}].PowerCost}"
    echo "      Me.CurrentPower:${Me.CurrentPower}"
    echo "      Me.CastingSpell:${Me.CastingSpell}"
    echo "      EQ2.Zoning:     ${EQ2.Zoning}"
    echo "      Target:         ${If[${Target(exists)}, ${Target.Name}, (none)]}"

    if ${Me.Ability[${AbilityName}].PowerCost} > ${Me.CurrentPower}
        echo "FAIL: insufficient power"

    if !${Me.Ability[${AbilityName}].IsReady} && !${Me.Ability[${AbilityName}].IsQueued}
        echo "FAIL: not ready (cooldown ${Me.Ability[${AbilityName}].TimeUntilReady}ms remaining)"

    echo "========================================"
}
```

### `IsReady` history note

The `IsReady` member of `ability` and `item` has been fixed multiple times in ISXEQ2 history (see `ISXEQ2Changes.txt` entries dated 2006-2024). If you see `IsReady` returning wrong values, verify against the latest changelog before working around it — it may be a recent fix or a known edge case.

Per the changelog: `IsReady` returns FALSE if the ability is currently queued (`IsQueued`), casting, or on cooldown. Always check `IsQueued` separately if you need to distinguish "on cooldown" from "already queued to cast next."

### Safe cast wrapper

```lavishscript
function bool SafeCast(string AbilityName)
{
    if !${Me.Ability[${AbilityName}](exists)}
    {
        Debug:Echo["SafeCast: ability '${AbilityName}' not found"]
        return FALSE
    }

    if ${EQ2.Zoning} != 0
    {
        Debug:Echo["SafeCast: blocked by zoning"]
        return FALSE
    }

    if !${Me.Ability[${AbilityName}].IsReady}
    {
        Debug:Echo["SafeCast: ${AbilityName} not ready (${Me.Ability[${AbilityName}].TimeUntilReady}ms)"]
        return FALSE
    }

    if ${Me.Ability[${AbilityName}].PowerCost} > ${Me.CurrentPower}
    {
        Debug:Echo["SafeCast: ${AbilityName} needs ${Me.Ability[${AbilityName}].PowerCost} power, have ${Me.CurrentPower}"]
        return FALSE
    }

    Me.Ability[${AbilityName}]:Use
    return TRUE
}
```

---

## Async Data Loading Failures

Many ISXEQ2 members return data that loads asynchronously — most commonly item detail (`ToItemInfo`) and actor detail. The pattern is already covered canonically in [04_Core_Concepts.md → Asynchronous Data Loading](04_Core_Concepts.md#asynchronous-data-loading) and [05_Patterns_And_Best_Practices.md → Async Data Timeouts](05_Patterns_And_Best_Practices.md#async-data-timeouts). This section covers **diagnosing** failures.

### Symptom: fields are blank or zero

If `${Item.ToItemInfo.Description}` returns empty but the item itself is clearly in your inventory, the async load hasn't completed yet.

```lavishscript
variable item TheItem
TheItem:Set[${Me.Inventory[5]}]

if !${TheItem.IsItemInfoAvailable}
{
    Debug:Echo["Item info not loaded for ${TheItem.Name} — waiting"]

    variable int waited = 0
    while !${TheItem.IsItemInfoAvailable} && ${waited} < 1500
    {
        waitframe
        waited:Inc
    }

    if !${TheItem.IsItemInfoAvailable}
    {
        Debug:Echo["TIMEOUT: ${TheItem.Name} info never loaded after 1500 frames"]
        return
    }

    Debug:Echo["Loaded after ${waited} frames"]
}

echo ${TheItem.ToItemInfo.Description}
```

### When info never loads

- The item was moved/removed during the wait — `${TheItem.IsItemInfoAvailable}` stays FALSE forever.
- The character zoned — cached item references invalidate.
- The reference was stored via `item Variable` and the inventory index shifted.

**Mitigation:** always time-bound your waits; on timeout, re-query rather than spinning.

### Actor info

`${Actor[...].IsActorInfoAvailable}` works the same way for detailed actor stats. Requesting any detail member (`IsHeroic`, `IsEpic`, `IsNamed`, etc.) on an actor that hasn't loaded detail yet returns FALSE/0 misleadingly.

```lavishscript
if ${Actor[${id}](exists)} && !${Actor[${id}].IsActorInfoAvailable}
{
    variable int waited = 0
    while !${Actor[${id}].IsActorInfoAvailable} && ${waited} < 100
    {
        waitframe
        waited:Inc
    }
}
```

---

## Actor Queries and Deprecated CustomActorArray

### `CreateCustomActorArray` is deprecated

If you inherited a script with `CreateCustomActorArray` + `${CustomActor[...]}`, it may still work but is deprecated. Use modern equivalents:

```lavishscript
; DEPRECATED
EQ2:CreateCustomActorArray[Range,50]
i:Set[1]
while ${i} <= ${CustomActor.Count}
{
    echo ${CustomActor[${i}].Name}
    i:Inc
}

; MODERN — keyword style
variable index:actor Actors
EQ2:GetActors[Actors, Range, 50, NPC]
variable iterator it
Actors:GetIterator[it]
if ${it:First(exists)}
{
    do
        echo ${it.Value.Name}
    while ${it:Next(exists)}
}

; MODERN — query style
EQ2:QueryActors[Actors, "Type =- \"NPC\" && Distance <= 50"]
```

### Symptom: migration from CustomActorArray appears to work, then returns empty

Common mistake — passing a parameter that `GetActors` doesn't accept. `GetActors` uses **keyword parameters**, not a query string:

```lavishscript
; WRONG — query syntax passed to GetActors
EQ2:GetActors[Actors, "Type == NPC && Distance < 50"]              ; fails silently

; RIGHT — keyword style
EQ2:GetActors[Actors, Range, 50, NPC]

; For complex filters, use QueryActors instead
EQ2:QueryActors[Actors, "Type =- \"NPC\" && Distance < 50"]
```

See [15_Advanced_Scripting_Patterns.md → Modern EQ2:GetActors Usage](15_Advanced_Scripting_Patterns.md#modern-eq2getactors-usage) for the canonical patterns.

### Symptom: query returns entries but members are empty/zero

The actor index was populated, but individual actors haven't loaded detail. Wait on `IsActorInfoAvailable` before reading detail fields — see [section 12](#async-data-loading-failures).

---

## EQ2Bot Class-Routine `#include` Failures

EQ2Bot's main script uses this include pattern at the top:

```lavishscript
#include ${LavishScript.HomeDirectory}/Scripts/${Script.Filename}/Class Routines/${Me.SubClass}.iss
#includeoptional ${LavishScript.HomeDirectory}/Scripts/${Script.Filename}/Character Config/${Me.Name}.iss
#includeoptional ${LavishScript.HomeDirectory}/Scripts/${Script.Filename}/Class Routines/${Me.SubClass}_StrRes.iss
```

### Symptom: script fails to start, or fails on some characters but not others

**Root cause:** the `#include` directive runs at **parse time**, before `function main()` executes. `${Me.SubClass}` must evaluate to a valid string at the moment the script is loaded. If `${Me}` is not yet available (e.g., at character-select screen or during an early zone), the include path evaluates to an empty subpath and parsing fails.

### Diagnostic

Run the script from a fully-loaded character in a zoned-in state, not from character select or during zoning. If it works then, the include-time evaluation is the issue.

### Mitigations

1. **Load EQ2Bot only after character is in-zone and ISXEQ2 is ready.** Add a wrapper script:

   ```lavishscript
   ; Launcher.iss
   function main()
   {
       while !${ISXEQ2.IsReady}
           wait 10
       while !${Me(exists)} || ${EQ2.Zoning} != 0
           wait 10
       wait 20                                                      ; extra settle

       run EQ2Bot
   }
   ```

2. **Use `#includeoptional` for the class file** when developing a new class-routine variant, so missing files don't kill script load. Note the main EQ2Bot line is NOT optional — deliberately — because a bot without a class routine has nothing to do.

3. **For customized forks:** if you renamed a class file, match the subclass string exactly. `${Me.SubClass}` returns strings like `Guardian`, `Berserker`, `Illusionist` — no spaces, PascalCase. Check the filename case on disk matches exactly (Windows is case-insensitive but cross-machine sync to Linux boxes is not).

---

## Maintained Effects vs Buff Checks

### Symptom: buff check returns the wrong state

EQ2 surfaces effects through several distinct collections. A common mistake is checking the wrong one.

- **Maintained** — effects your character is actively maintaining (your own buffs, your concentration spells). Access via `${Me.Maintained[...]}`.
- **Effect** — effects ON your character (from any source, including yours). Access via `${Me.Effect[...]}`.
- **`Beneficial` / `Detrimental`** — filtered subsets of effects.

The key distinction: if you cast a self-buff, it appears in BOTH `Me.Maintained` and `Me.Effect`. If someone else casts a buff on you, it's only in `Me.Effect`. If you cast a buff on a group member, it's in `Me.Maintained` (for you) and in their `Me.Effect` list.

### Diagnostic: iterate both to see the difference

```lavishscript
function DumpEffects()
{
    echo "=== Maintained (${Me.CountMaintained}) ==="
    variable int i = 1
    while ${i} <= ${Me.CountMaintained}
    {
        echo "  [${i}] ${Me.Maintained[${i}].Name} on ${Me.Maintained[${i}].Target.Name}"
        i:Inc
    }

    echo "=== Effects (${Me.CountEffects}) ==="
    i:Set[1]
    while ${i} <= ${Me.CountEffects}
    {
        echo "  [${i}] ${Me.Effect[${i}].Name}"
        i:Inc
    }
    echo "=================================="
}
```

### Correct "am I buffed?" check

```lavishscript
; Is my group member buffed with my spell?
if ${Me.Maintained[${BuffName}](exists)}
{
    ; I am maintaining this on someone
}

; Am I personally under this effect (from any source)?
if ${Me.Effect[${BuffName}](exists)}
{
    ; The effect is on me
}
```

See [03_API_Reference.md → effect](03_API_Reference.md#effect) for the full datatype members.

---

## Inventory Queries After Bag Rearrangement

### Symptom: item reference becomes stale mid-script

```lavishscript
variable item MyPotion
MyPotion:Set[${Me.Inventory[ExactName,"Minor Healing Potion"]}]

Me:MoveItem[...]                                                   ; or drag-drop, or vendor interaction

echo ${MyPotion.Name}                                              ; may be wrong item now
```

**Root cause:** `item` variables store a **reference to an inventory slot index**, not to an item identity. When bags reshuffle, the slot refers to whatever is there now.

### Mitigation

- Re-query with `ExactName` immediately before each use, not once at the top of the script.
- For repeated use in a tight loop where you know the inventory is stable, cache briefly — but invalidate after any `Me:MoveItem`, `Item:Use`, merchant transaction, or loot event.

```lavishscript
function bool UsePotion(string PotionName)
{
    if !${Me.Inventory[ExactName,${PotionName}](exists)}
    {
        Debug:Echo["${PotionName} not in inventory"]
        return FALSE
    }

    if !${Me.Inventory[ExactName,${PotionName}].IsReady}
    {
        Debug:Echo["${PotionName} not ready"]
        return FALSE
    }

    Me.Inventory[ExactName,${PotionName}]:Use
    return TRUE
}
```

### Iterator invalidation

If you're iterating an inventory index and the inventory changes mid-iteration (loot drops in, item is consumed), the iterator may yield stale results. Rebuild the index after any action that mutates inventory:

```lavishscript
variable index:item Items
Me:QueryInventory[Items, "Name =- \"Potion\""]

variable iterator it
Items:GetIterator[it]

if ${it:First(exists)}
{
    do
    {
        if ${it.Value.IsReady}
            it.Value:Use

        ; After :Use, the inventory may have changed.
        ; If we continue iterating, we may skip/repeat items.
        ; Safer: break and re-query.
        break
    }
    while ${it:Next(exists)}
}
```

---

## ExactName Case Sensitivity and String Comparison

LavishScript **member names** are case-insensitive (`${Me.Name}` = `${Me.NAME}`). But **string comparisons and string-parameter lookups** are case-sensitive:

```lavishscript
; Member names: case-insensitive
${Me.Name}           ; works
${Me.NAME}           ; works
${me.name}           ; works

; String comparisons: case-sensitive
${Me.Name.Equal["Bob"]}    ; only matches "Bob"
${Me.Name.Equal["bob"]}    ; only matches "bob"

; Inventory lookups with ExactName: case-sensitive
${Me.Inventory[ExactName,"Health Potion"]}    ; exact match
${Me.Inventory[ExactName,"health potion"]}    ; WILL NOT FIND IT
```

### Mitigation

For case-insensitive string comparison, use `.EqualCS` variants sparingly (they exist for comparison), or normalize to a canonical case first:

```lavishscript
; Normalize before compare
variable string lname = "${Me.Name.Lower}"
if ${lname.Equal["bob"]}
    echo "It's Bob"

; For inventory, use fuzzy-match operators in a query
variable index:item results
Me:QueryInventory[results, "Name =- \"potion\""]                   ; case-sensitive substring
Me:QueryInventory[results, "Name =~ \"(?i)potion\""]               ; regex with case-insensitive flag (if supported)
```

For query operator details, see [04_Core_Concepts.md → Query Syntax](04_Core_Concepts.md#query-syntax).

---

## LGUI1 `_executeOnSelect` Recursion and UI Timing

LGUI1 combo boxes, listboxes, and checkboxes fire their `onSelect`/`onChange`/`onCheck` handlers when set programmatically during initialization — not just when the user interacts. Without guards, initialization causes a cascade of handler fires that can corrupt settings or crash.

### Per-element guard pattern

```lavishscript
variable(script) bool FilterCombo_executeOnSelect = FALSE
variable(script) bool SortCombo_executeOnSelect  = FALSE

; During UI init, SET without firing handlers
FilterCombo_executeOnSelect:Set[FALSE]
UIElement[Filter@Tab@MyBot]:SelectItem[3]
FilterCombo_executeOnSelect:Set[TRUE]

; The onSelect handler itself:
atom FilterCombo_OnSelect()
{
    if !${FilterCombo_executeOnSelect}
        return
    ; ... real work ...
}
```

### Why per-element, not one global flag

```lavishscript
; WRONG — one global flag for multiple combos
variable(script) bool UIInitialized = FALSE

; Bug: any handler firing during init sees UIInitialized=FALSE for
; EVERY element, even ones already initialized. A handler that should
; have fired (because the user changed ComboA after ComboB is still
; initializing) gets suppressed.

; RIGHT — per-element flags
variable(script) bool Combo1_executeOnSelect = FALSE
variable(script) bool Combo2_executeOnSelect = FALSE
```

See [15_Advanced_Scripting_Patterns.md → UI Initialization Guards](15_Advanced_Scripting_Patterns.md) for the full treatment with real-world examples.

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
    Debug:Echo["UI package failed to load within 100 frames"]
    return
}
```

See [10_LavishGUI2_UI_Guide.md → Troubleshooting](10_LavishGUI2_UI_Guide.md#troubleshooting) for more LGUI2-specific issues.

---

## Systematic Issue Diagnosis Workflow

Follow this ordered workflow when a script misbehaves:

- [ ] **Reproduce** — can you make it happen on demand? What are the exact steps?
- [ ] **Isolate** — does it happen only in certain states (combat / zoning / solo / group)? Certain classes? Certain zones?
- [ ] **Log** — add `Debug:Echo` around the misbehavior site; log inputs, outputs, and state transitions
- [ ] **Validate** — confirm `${ISXEQ2.IsReady}`, `${Me(exists)}`, `${EQ2.Zoning} == 0`, and any async `Is*InfoAvailable` precondition
- [ ] **Dump state** — call a full variable/target/inventory dump at the point of failure
- [ ] **Simplify** — comment out peripheral logic; can you reproduce with a 10-line test script?
- [ ] **Hypothesize** — which invariant is violated? Write it down before testing.
- [ ] **Fix and verify** — re-run the test script, then the full script, then under extended runtime

---

## Performance Investigation Workflow

- [ ] **Measure first** — add `${Script.RunningTime}` bookends to suspect regions; never optimize without measurement
- [ ] **Profile** — use `obj_Profiler` (see [section 8](#performance-profiling)) to identify the top-3 time consumers
- [ ] **Sample memory** — track `${Script.MemUsage}` over time; flat is fine, growing is a leak
- [ ] **Throttle hot loops** — reduce query frequency; use tiered pulse timers (1s / 2s / 5s / 10s)
- [ ] **Cache aggressively** — for data that changes slowly (actor list in stable zone), cache and invalidate on `EQ2_FinishedZoning`
- [ ] **Re-measure** — confirm the optimization moved the needle
- [ ] **Monitor long-term** — run for hours; watch for drift

---

## Stuck Detection and Recovery

Stuck detection has two flavors in ISXEQ2: **state-machine stuck** (script hasn't changed state in X minutes) and **navigation stuck** (character isn't moving when it should be).

For **navigation stuck**, [18_Navigation_Library_Patterns.md → Stuck Detection and Recovery](18_Navigation_Library_Patterns.md#stuck-detection-and-recovery) is the canonical treatment — multi-level detection, jump-unstuck, door handling, obstacle avoidance. Use that.

For **state-machine stuck**, the pattern is:

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
            Debug:Echo["STATE STUCK: in '${currentState}' for ${Math.Calc[${heldMs}/60000].Int} min"]
            This:Recover["${currentState}"]
            This.LastStateChange:Set[${Script.RunningTime}]
        }
    }

    method Recover(string state)
    {
        Debug:Echo["Attempting recovery from state: ${state}"]

        ; Script-specific recovery logic
        switch ${state}
        {
            case Looting
                ; Unstick loot window
                press esc
                break
            case Crafting
                ; Close all windows
                press esc
                wait 5
                press esc
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

### Dead man's switch — abort after no user input

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
            Debug:Echo["Dead-man's switch: ${idleMin.Precision[0]} min with no input — halting"]
            eq2echo "\#FF0000AFK timeout — script halting\#FFFFFF"
            Script:Pause
        }
    }
}
```

### Periodic restart — bound total runtime

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
            Debug:Echo["Max runtime ${maxHours}h reached — exiting for restart"]
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

        Debug:Echo["Config saved (backup: ${This.BackupFile})"]
    }

    method Load(string settingsSetName)
    {
        declare FP filepath "${Script.CurrentDirectory}"

        if !${FP.FileExists["config.xml"]}
        {
            Debug:Echo["Config missing — checking backup"]

            if ${FP.FileExists["config.backup.xml"]}
            {
                redirect "${This.ConfigFile}" cat "${This.BackupFile}"
                Debug:Echo["Restored from backup"]
            }
            else
            {
                Debug:Echo["No backup — using defaults"]
                return
            }
        }

        LavishSettings[${settingsSetName}]:Import["${This.ConfigFile}"]
        Debug:Echo["Config loaded"]
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

### Diagnostic — list registered events

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
; WRONG — name with space becomes first word only
relay all -event TargetNamed ${Target.Name}

; RIGHT — quote string parameters
relay all -event TargetNamed "${Target.Name}"

; Multiple params
relay all -event TargetInfo "${Target.Name}" ${Target.Distance} ${Target.ID}
```

### Receiver

```lavishscript
atom(script) OnTargetInfo(string Name, float Distance, int64 ID)
{
    Debug:Echo["Received: ${Name} at ${Distance}m (id=${ID})"]
}

function main()
{
    LavishScript:RegisterEvent[TargetInfo]
    Event[TargetInfo]:AttachAtom[OnTargetInfo]

    while 1
        waitframe
}
```

---

## Top-10 Checklist

Quick checklist to run through when any script misbehaves:

- [ ] Did I `wait` for `${ISXEQ2.IsReady}` before the first API access?
- [ ] Did I check `${EQ2.Zoning} == 0` before acting on actors/abilities?
- [ ] Am I using `(exists)` before every object member access?
- [ ] Am I waiting on `${Item.IsItemInfoAvailable}` / `${Actor.IsActorInfoAvailable}` with a timeout before reading detail?
- [ ] Does every `while` / `do ... while` loop contain a `wait` of at least 1?
- [ ] Are events `RegisterEvent`'d before any `relay -event` fires them?
- [ ] Are long-lived atoms declared `atom(script)`?
- [ ] Do I `Clear` collections periodically (or `EQ2_FinishedZoning`) to avoid memory growth?
- [ ] Am I using keyword-style `EQ2:GetActors` (or `EQ2:QueryActors` for query strings), not the deprecated `CreateCustomActorArray`?
- [ ] Have I saved config via `:Export` / `:Save` after changes, with a backup?

For full anti-pattern treatment, see [05_Patterns_And_Best_Practices.md → Common Anti-Patterns](05_Patterns_And_Best_Practices.md#common-anti-patterns). For beginner troubleshooting, see [02_Quick_Start_Guide.md → Troubleshooting](02_Quick_Start_Guide.md#troubleshooting) and the [Quick Troubleshooting table](00_MASTER_GUIDE.md#quick-troubleshooting) in the master guide.
