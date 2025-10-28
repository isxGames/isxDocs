# LavishMachine (LMAC) - Complete Guide

**System:** LavishMachine Virtual Machine
**Target:** InnerSpace Scripts (Game-Agnostic)

---

## Table of Contents

1. [Introduction to LavishMachine](#introduction-to-lavish machine)
2. [Core Concepts](#core-concepts)
3. [Task Managers](#task-managers)
4. [Built-in Task Types](#built-in-task-types)
5. [Creating Custom Task Types](#creating-custom-task-types)
6. [Task States and Lifecycle](#task-states-and-lifecycle)
7. [Task Context](#task-context)
8. [Audio Tasks](#audio-tasks)
9. [Web Request Tasks](#web-request-tasks)
10. [Chain Tasks](#chain-tasks)
11. [Error Handling](#error-handling)
12. [Controller Pattern](#controller-pattern)
13. [Complete Examples](#complete-examples)
14. [Best Practices](#best-practices)
15. [Troubleshooting](#troubleshooting)

---

## Introduction to LavishMachine

### What is LavishMachine?

**LavishMachine (LMAC)** is a **Virtual Machine** integrated into LavishScript that handles running **Tasks**. It provides a powerful framework for:

- ✅ **Asynchronous operations** - Run tasks in the background
- ✅ **Time-based animations** - Smooth value transitions over time
- ✅ **Audio control** - Play sounds and music with volume/panning
- ✅ **Web requests** - HTTP GET/POST operations
- ✅ **Task sequencing** - Chain multiple tasks together
- ✅ **Custom behaviors** - Create your own task types

### Why Use LavishMachine?

**Without LMAC:**
```lavishscript
; Manual volume fade - complex and error-prone
variable float volume=1.0
variable int frames=0
while ${frames} < 100
{
    volume:Set[${Math.Calc[${volume}-0.01]}]
    Audio.Voice[music]:SetVolume[${volume},${volume}]
    waitframe
    frames:Inc
}
```

**With LMAC:**
```lavishscript
; Automatic volume fade - clean and simple
TaskManager:BeginTask["$$>
{
    \"type\":\"audio.setvolume\",
    \"duration\":5.0,
    \"voiceName\":\"music\",
    \"volume\":[0.1,0.1]
}
<$$"]
```

### Key Benefits

1. ✅ **Cleaner code** - Declare what you want, not how to do it
2. ✅ **Non-blocking** - Tasks run in background while script continues
3. ✅ **Automatic timing** - No manual frame counting
4. ✅ **Error handling** - Built-in error management
5. ✅ **Reusable** - Custom task types can be shared across scripts

---

## Core Concepts

### Tasks

A **Task** is a unit of work that executes over time. Tasks are defined as **JSON objects** and managed by a **Task Manager**.

**Simple task:**
```lavishscript
{
    "type": "ls1.echo",
    "output": "Hello World!"
}
```

**Complex task:**
```lavishscript
{
    "type": "audio.setvolume",
    "duration": 5.0,
    "voiceName": "music",
    "volume": [0.1, 0.1]
}
```

### Task Anatomy

| Property | Required | Description |
|----------|----------|-------------|
| `type` | Yes | Task type identifier (e.g., "ls1.echo", "audio.playstream") |
| `duration` | No | How long the task should run (seconds or milliseconds) |
| Other properties | No | Task-specific arguments |

**Duration formats:**
- **Floating point = seconds**: `5.0` = 5 seconds
- **Integer = milliseconds**: `5000` = 5000 milliseconds (5 seconds)

### Task Managers

A **Task Manager** manages a collection of tasks. It:
- Starts and stops tasks
- Tracks running tasks
- Cleans up completed tasks
- Provides task lifecycle management

**Creating a Task Manager:**
```lavishscript
variable taskmanager TaskManager=${LMAC.NewTaskManager["my task manager"]}
```

---

## Task Managers

### Creating a Task Manager

```lavishscript
variable taskmanager MyTM=${LMAC.NewTaskManager["descriptive name"]}
```

The name is for debugging/identification purposes.

### Beginning a Task

Use `:BeginTask[jsonDefinition]`:

```lavishscript
TaskManager:BeginTask["$$>
{
    \"type\":\"ls1.echo\",
    \"output\":\"Hello World!\"
}
<$$"]
```

**With JSON variable:**
```lavishscript
variable jsonvalue taskDef="{\"type\":\"ls1.echo\",\"output\":\"Test\"}"
TaskManager:BeginTask["${taskDef~}"]
```

### Destroying a Task Manager

**Always destroy task managers** when done to stop running tasks:

```lavishscript
TaskManager:Destroy
```

This stops all tasks managed by that task manager.

### Multiple Task Managers

You can have multiple task managers for organization:

```lavishscript
variable taskmanager AudioTM=${LMAC.NewTaskManager["audio"]}
variable taskmanager WebTM=${LMAC.NewTaskManager["web requests"]}
variable taskmanager AnimationTM=${LMAC.NewTaskManager["animations"]}
```

---

## Built-in Task Types

### ls1.echo

Echoes output to console (instant task).

**Definition:**
```json
{
    "type": "ls1.echo",
    "output": "Hello World!"
}
```

**Example:**
```lavishscript
TaskManager:BeginTask["{\"type\":\"ls1.echo\",\"output\":\"Task started!\"}"]
```

### audio.playstream

Plays an audio stream on a voice.

**Definition:**
```json
{
    "type": "audio.playstream",
    "voiceName": "music",
    "streamName": "tune",
    "playCount": 1,
    "flush": true
}
```

**Properties:**
- `voiceName` (required) - Audio voice to play on
- `streamName` (optional) - Stream to play (omit to stop voice)
- `playCount` (optional) - Number of times to play (default: 1, -1 = loop forever)
- `flush` (optional) - Stop voice and clear queue before playing (default: true)

**Example:**
```lavishscript
; Add voice and stream first
Audio:AddVoice[music]
Audio:AddStream[tune,"music.mp3"]

; Play the sound 3 times
TaskManager:BeginTask["$$>
{
    \"type\":\"audio.playstream\",
    \"voiceName\":\"music\",
    \"streamName\":\"tune\",
    \"playCount\":3
}
<$$"]
```

**Stop a voice:**
```lavishscript
TaskManager:BeginTask["$$>
{
    \"type\":\"audio.playstream\",
    \"voiceName\":\"music\"
}
<$$"]
```

### audio.setvolume

Adjusts volume on a voice, optionally over time.

**Definition:**
```json
{
    "type": "audio.setvolume",
    "voiceName": "music",
    "volume": [0.5, 0.5],
    "duration": 5.0
}
```

**Properties:**
- `voiceName` (required) - Audio voice to adjust
- `volume` (required) - Single number or array of per-channel volumes (up to 32 channels)
- `duration` (optional) - Time to adjust over (seconds/milliseconds)

**Instant volume change:**
```lavishscript
TaskManager:BeginTask["$$>
{
    \"type\":\"audio.setvolume\",
    \"voiceName\":\"music\",
    \"volume\":[0.5,0.5]
}
<$$"]
```

**Fade to 10% over 5 seconds:**
```lavishscript
TaskManager:BeginTask["$$>
{
    \"type\":\"audio.setvolume\",
    \"duration\":5.0,
    \"voiceName\":\"music\",
    \"volume\":[0.1,0.1]
}
<$$"]
```

**Pan left to right over 3 seconds:**
```lavishscript
; Pan to left (left=1.0, right=0.1)
TaskManager:BeginTask["{\"type\":\"audio.setvolume\",\"duration\":3.0,\"voiceName\":\"music\",\"volume\":[1.0,0.1]}"]

; Wait for completion, then pan to right
wait 30
TaskManager:BeginTask["{\"type\":\"audio.setvolume\",\"duration\":3.0,\"voiceName\":\"music\",\"volume\":[0.1,1.0]}"]
```

### webrequest

Performs HTTP requests (GET, POST, etc.).

**Definition:**
```json
{
    "type": "webrequest",
    "url": "https://example.com/data.json",
    "as": "json",
    "object": "MyController",
    "method": "OnRequestComplete"
}
```

**Properties:**
- `url` (required) - URL to request
- `as` (required) - Result format: "json", "string", "binary", or "file"
- `file` (required if as="file") - Output filename
- `object` (optional) - Controller object for callback
- `method` (optional) - Method to call on completion

**Result formats:**
- `"as":"json"` - Parse response as JSON
- `"as":"string"` - Return as string
- `"as":"binary"` - Return as binary data
- `"as":"file"` - Save to file

**Example:**
```lavishscript
TaskManager:BeginTask["$$>
{
    \"type\":\"webrequest\",
    \"url\":\"https://api.example.com/data.json\",
    \"as\":\"json\",
    \"object\":\"MyController\",
    \"method\":\"OnDataReceived\"
}
<$$"]
```

**In controller:**
```lavishscript
method OnDataReceived()
{
    echo State: ${Context.State}
    echo Result: ${Context.Result~}
}
```

### chain

Executes multiple tasks in sequence.

**Definition:**
```json
{
    "type": "chain",
    "tasks": [
        { "type": "task1", ... },
        { "type": "task2", ... },
        { "type": "task3", ... }
    ]
}
```

**Example:**
```lavishscript
TaskManager:BeginTask["$$>
{
    \"type\":\"chain\",
    \"tasks\":[
        {
            \"type\":\"ls1.echo\",
            \"output\":\"First task\"
        },
        {
            \"type\":\"audio.setvolume\",
            \"duration\":2.0,
            \"voiceName\":\"music\",
            \"volume\":[0.5,0.5]
        },
        {
            \"type\":\"ls1.echo\",
            \"output\":\"Last task\"
        }
    ]
}
<$$"]
```

Each task runs **after** the previous one completes.

---

## Creating Custom Task Types

### Why Create Custom Task Types?

Custom task types let you:
- ✅ Encapsulate complex behaviors
- ✅ Create reusable animations
- ✅ Build domain-specific operations
- ✅ Maintain clean, declarative code

### Basic Custom Task Type

**Step 1: Register the task type**

```lavishscript
noop ${LMAC.NewTaskType["$$>
{
    \"name\":\"custom.echo\",
    \"object\":\"MyController\",
    \"method\":\"Task_Echo\"
}
<$$"]}
```

**Breakdown:**
- `name` - Unique task type identifier
- `object` - Controller object name (must be global)
- `method` - Method to call for task execution

**Step 2: Implement the task method**

```lavishscript
method Task_Echo()
{
    switch ${Context.TaskState}
    {
    case Start
        echo ${Context.Task.Args[output]~}
        Context.Task:Stop
    break
    case Continue
        ; Not reached for instant tasks
    break
    case Stop
        ; Cleanup if needed
    break
    }
}
```

**Step 3: Use the custom task**

```lavishscript
TaskManager:BeginTask["$$>
{
    \"type\":\"custom.echo\",
    \"output\":\"Hello from custom task!\"
}
<$$"]
```

### Advanced Custom Task Type - Slide Value

This example smoothly transitions a value over time.

**Registration:**
```lavishscript
noop ${LMAC.NewTaskType["$$>
{
    \"name\":\"custom.slideX\",
    \"object\":\"MyController\",
    \"method\":\"Task_SlideX\"
}
<$$"]}
```

**Implementation:**
```lavishscript
variable float X=100.0

method Task_SlideX()
{
    switch ${Context.TaskState}
    {
    case Start
        ; Validate required argument
        if !${Context.Task.Args.Has[to]}
        {
            Context:SetError["custom.slideX requires 'to' argument"]
            Context.Task:Stop
            return
        }

        ; Store original value
        Context.Task.Args:Set["original_x",${X}]

        ; Calculate speed: (target - current) / duration
        if ${Context.Task.Duration}
        {
            variable float speed=${Math.Calc[(${Context.Task.Args[to]}-${X})/${Context.Task.Duration}]}
            Context.Task.Args:Set["speed_x",${speed}]
        }
        else
        {
            ; No duration = instant
            X:Set[${Context.Task.Args[to]}]
            Context.Task:Stop
        }
    break

    case Continue
        ; Update value: original + (speed * time)
        X:Set[${Math.Calc[${Context.Task.Args[original_x]}+(${Context.Task.Args[speed_x]}*${Context.Task.RunningTime})]}]
        echo X=${X}
    break

    case Stop
        ; Set final value (if no error)
        if !${Context.Error.NotNULLOrEmpty}
        {
            X:Set[${Context.Task.Args[to]}]
        }
        echo Final X=${X}
    break
    }
}
```

**Usage:**
```lavishscript
; Slide X from 100.0 to 250.0 over 3 seconds
TaskManager:BeginTask["$$>
{
    \"type\":\"custom.slideX\",
    \"duration\":3.0,
    \"to\":250.0
}
<$$"]
```

---

## Task States and Lifecycle

### Task States

Every task goes through three states:

| State | When | Purpose |
|-------|------|---------|
| **Start** | Task begins | Initialize, validate arguments, prepare |
| **Continue** | Each frame while running | Update values, check conditions |
| **Stop** | Task ends | Cleanup, finalize values |

### State Diagram

```
    BeginTask
        ↓
    [Start] ←───────────────┐
        ↓                   │
    [Continue] ─────────────┘
        ↓
    [Stop]
        ↓
   Task Complete
```

### When Tasks Stop

Tasks stop when:
1. ✅ **Duration expires** - Task ran for specified time
2. ✅ **Manually stopped** - `Context.Task:Stop` called
3. ✅ **Error occurs** - `Context:SetError` + `Context.Task:Stop`
4. ✅ **Task Manager destroyed** - All tasks stopped

### Instant Tasks

For tasks that complete immediately:

```lavishscript
case Start
    ; Do work
    echo Instant task executed!
    Context.Task:Stop  ; Stop immediately
break
```

The `Continue` state is never reached.

### Timed Tasks

For tasks that run over time:

```lavishscript
case Start
    ; Initialize
    ; DON'T call Stop - let it run
break

case Continue
    ; Update each frame
    ; Check if Context.Task.RunningTime >= some condition
    if ${Context.Task.RunningTime} >= ${SomeLimit}
        Context.Task:Stop
break

case Stop
    ; Finalize
break
```

---

## Task Context

### What is Context?

Inside task methods, `${Context}` provides access to task information via `taskpulseargs`.

### taskpulseargs Members

| Member | Type | Description |
|--------|------|-------------|
| `Timestamp` | int64 | Current timestamp |
| `ElapsedMS` | int | Milliseconds since last pulse |
| `TaskState` | string | "Start", "Continue", or "Stop" |
| `Task` | task | The running task object |
| `Error` | string | Error description (if any) |

### task Members

| Member | Type | Description |
|--------|------|-------------|
| `Name` | string | Task type name |
| `ID` | int64 | Unique task ID |
| `Type` | tasktype | Task type object |
| `TaskManager` | taskmanager | Managing task manager |
| `Args` | jsonobject | Task arguments (from definition) |
| `Result` | jsonvalue | Task result |
| `IsRunning` | bool | TRUE if currently running |
| `RunningTime` | float | Seconds since start |
| `RunningTimeMS` | int | Milliseconds since start |
| `FrameElapsed` | float | Seconds since last frame |
| `FrameElapsedMS` | int | Milliseconds since last frame |
| `StartTimestamp` | int64 | When task started |
| `LastFrameTimestamp` | int64 | Last frame timestamp |
| `Duration` | float | Specified duration (seconds) |
| `DurationMS` | int | Specified duration (milliseconds) |
| `IsInstant` | bool | TRUE if instant (no duration) |

### task Methods

| Method | Description |
|--------|-------------|
| `Start` | Start the task |
| `Stop` | Stop the task |
| `Toggle` | Toggle running state |

### Accessing Task Arguments

Arguments from the task definition are in `Context.Task.Args`:

**Task definition:**
```json
{
    "type": "custom.slide",
    "from": 0,
    "to": 100,
    "duration": 5.0
}
```

**Accessing in task method:**
```lavishscript
variable float fromValue=${Context.Task.Args[from]}
variable float toValue=${Context.Task.Args[to]}
variable float duration=${Context.Task.Duration}
```

### Storing Temporary Data

Use `Context.Task.Args:Set` to store per-task data:

```lavishscript
case Start
    ; Store calculated values
    Context.Task.Args:Set["speed",${calculatedSpeed}]
    Context.Task.Args:Set["originalValue",${CurrentValue}]
break

case Continue
    ; Retrieve stored values
    variable float speed=${Context.Task.Args[speed]}
    variable float original=${Context.Task.Args[originalValue]}
break
```

---

## Audio Tasks

### Setup

**Create audio voice:**
```lavishscript
Audio:AddVoice[music]
```

**Add audio stream:**
```lavishscript
Audio:AddStream[tune,"path/to/music.mp3"]
```

### Play Sound

**Play once:**
```lavishscript
TaskManager:BeginTask["$$>
{
    \"type\":\"audio.playstream\",
    \"voiceName\":\"music\",
    \"streamName\":\"tune\"
}
<$$"]
```

**Play multiple times:**
```lavishscript
TaskManager:BeginTask["$$>
{
    \"type\":\"audio.playstream\",
    \"voiceName\":\"music\",
    \"streamName\":\"tune\",
    \"playCount\":3
}
<$$"]
```

**Loop forever:**
```lavishscript
TaskManager:BeginTask["$$>
{
    \"type\":\"audio.playstream\",
    \"voiceName\":\"music\",
    \"streamName\":\"tune\",
    \"playCount\":-1
}
<$$"]
```

### Volume Control

**Instant volume change:**
```lavishscript
TaskManager:BeginTask["$$>
{
    \"type\":\"audio.setvolume\",
    \"voiceName\":\"music\",
    \"volume\":[1.0,1.0]
}
<$$"]
```

**Fade out over 5 seconds:**
```lavishscript
TaskManager:BeginTask["$$>
{
    \"type\":\"audio.setvolume\",
    \"duration\":5.0,
    \"voiceName\":\"music\",
    \"volume\":[0.0,0.0]
}
<$$"]
```

**Pan left to right:**
```lavishscript
; Start: both channels at 50%
TaskManager:BeginTask["{\"type\":\"audio.setvolume\",\"voiceName\":\"music\",\"volume\":[0.5,0.5]}"]

; Pan to left over 2 seconds
TaskManager:BeginTask["{\"type\":\"audio.setvolume\",\"duration\":2.0,\"voiceName\":\"music\",\"volume\":[1.0,0.1]}"]

; Pan to right over 2 seconds
TaskManager:BeginTask["{\"type\":\"audio.setvolume\",\"duration\":2.0,\"voiceName\":\"music\",\"volume\":[0.1,1.0]}"]
```

### Complete Audio Example

```lavishscript
objectdef audio_controller
{
    variable taskmanager TM=${LMAC.NewTaskManager["audio"]}

    method Initialize()
    {
        Audio:AddVoice[bgmusic]
        Audio:AddStream[theme,"music/theme.mp3"]

        ; Play music looping
        TM:BeginTask["$$>
        {
            \"type\":\"audio.playstream\",
            \"voiceName\":\"bgmusic\",
            \"streamName\":\"theme\",
            \"playCount\":-1
        }
        <$$"]

        ; Fade in over 3 seconds
        TM:BeginTask["$$>
        {
            \"type\":\"audio.setvolume\",
            \"duration\":3.0,
            \"voiceName\":\"bgmusic\",
            \"volume\":[1.0,1.0]
        }
        <$$"]
    }

    method FadeOut()
    {
        ; Fade to silence over 2 seconds
        TM:BeginTask["$$>
        {
            \"type\":\"audio.setvolume\",
            \"duration\":2.0,
            \"voiceName\":\"bgmusic\",
            \"volume\":[0.0,0.0]
        }
        <$$"]
    }

    method Shutdown()
    {
        This:FadeOut
        wait 20  ; Wait for fade
        Audio.Voice[bgmusic]:Stop:ClearQueue
        TM:Destroy
    }
}
```

---

## Web Request Tasks

### Basic Web Request

```lavishscript
TaskManager:BeginTask["$$>
{
    \"type\":\"webrequest\",
    \"url\":\"https://api.example.com/data\",
    \"as\":\"json\",
    \"object\":\"MyController\",
    \"method\":\"OnDataReceived\"
}
<$$"]
```

### Result Formats

**JSON:**
```lavishscript
{
    "type": "webrequest",
    "url": "https://api.example.com/data.json",
    "as": "json",
    "object": "Controller",
    "method": "OnJSON"
}
```

```lavishscript
method OnJSON()
{
    echo Data: ${Context.Result~}
    ; Context.Result is a jsonvalue
}
```

**String:**
```lavishscript
{
    "type": "webrequest",
    "url": "https://example.com/page.html",
    "as": "string",
    "object": "Controller",
    "method": "OnHTML"
}
```

```lavishscript
method OnHTML()
{
    echo HTML: ${Context.Result~}
    ; Context.Result is a string
}
```

**File:**
```lavishscript
{
    "type": "webrequest",
    "url": "https://example.com/image.png",
    "as": "file",
    "file": "downloaded_image.png",
    "object": "Controller",
    "method": "OnFileDownloaded"
}
```

```lavishscript
method OnFileDownloaded()
{
    echo File saved to: downloaded_image.png
    echo State: ${Context.State}
}
```

### Complete Web Request Example

```lavishscript
objectdef api_client
{
    variable taskmanager TM=${LMAC.NewTaskManager["api"]}

    method RequestPlayerData(string playerName)
    {
        variable jsonvalue taskDef
        taskDef:SetValue["$$>
        {
            \"type\":\"webrequest\",
            \"as\":\"json\",
            \"object\":\"APIClient\",
            \"method\":\"OnPlayerData\",
            \"url\":\"https://api.example.com/player/${playerName.Escape}\"
        }
        <$$"]

        TM:BeginTask["${taskDef~}"]
    }

    method OnPlayerData()
    {
        if ${Context.State.Equal[complete]}
        {
            echo Player Data: ${Context.Result~}

            ; Extract specific values
            if ${Context.Result.Type.Equal[object]}
            {
                echo Name: ${Context.Result.Get[name]~}
                echo Level: ${Context.Result.Get[level]}
                echo Guild: ${Context.Result.Get[guild]~}
            }
        }
        else
        {
            echo Request failed: ${Context.State}
        }
    }

    method Shutdown()
    {
        TM:Destroy
    }
}

variable(global) api_client APIClient
```

---

## Chain Tasks

### Basic Chain

Execute tasks in sequence:

```lavishscript
TaskManager:BeginTask["$$>
{
    \"type\":\"chain\",
    \"tasks\":[
        {
            \"type\":\"ls1.echo\",
            \"output\":\"Step 1\"
        },
        {
            \"type\":\"ls1.echo\",
            \"output\":\"Step 2\"
        },
        {
            \"type\":\"ls1.echo\",
            \"output\":\"Step 3\"
        }
    ]
}
<$$"]
```

### Chain with Delays

Use timed custom tasks or audio.setvolume with 0 volume change as delays:

```lavishscript
{
    "type": "chain",
    "tasks": [
        { "type": "ls1.echo", "output": "Starting..." },
        { "type": "custom.wait", "duration": 2.0 },
        { "type": "ls1.echo", "output": "Done waiting!" }
    ]
}
```

### Chain with Audio Sequence

```lavishscript
TaskManager:BeginTask["$$>
{
    \"type\":\"chain\",
    \"tasks\":[
        {
            \"type\":\"audio.playstream\",
            \"voiceName\":\"effects\",
            \"streamName\":\"beep\"
        },
        {
            \"type\":\"audio.setvolume\",
            \"duration\":1.0,
            \"voiceName\":\"music\",
            \"volume\":[0.5,0.5]
        },
        {
            \"type\":\"ls1.echo\",
            \"output\":\"Sequence complete!\"
        }
    ]
}
<$$"]
```

### Complex Panning Sequence

From the audio-1.iss example:

```lavishscript
TaskManager:BeginTask["$$>
{
    \"type\":\"chain\",
    \"tasks\":[
        {
            \"type\":\"audio.setvolume\",
            \"duration\":5.0,
            \"voiceName\":\"music\",
            \"volume\":[0.1,0.1]
        },
        {
            \"type\":\"audio.setvolume\",
            \"duration\":5.0,
            \"voiceName\":\"music\",
            \"volume\":[1.0,0.1]
        },
        {
            \"type\":\"audio.setvolume\",
            \"duration\":5.0,
            \"voiceName\":\"music\",
            \"volume\":[0.1,1.0]
        },
        {
            \"type\":\"audio.setvolume\",
            \"duration\":5.0,
            \"voiceName\":\"music\",
            \"volume\":[1.0,0.1]
        },
        {
            \"type\":\"audio.setvolume\",
            \"duration\":5.0,
            \"voiceName\":\"music\",
            \"volume\":[0.1,1.0]
        }
    ]
}
<$$"]
```

This creates a smooth pan effect from center → left → right → left → right.

---

## Error Handling

### Setting Errors

Use `Context:SetError[description]` to indicate errors:

```lavishscript
case Start
    if !${Context.Task.Args.Has[required_arg]}
    {
        Context:SetError["Missing required argument: required_arg"]
        Context.Task:Stop
        return
    }
break
```

### Checking for Errors

In the `Stop` state:

```lavishscript
case Stop
    if ${Context.Error.NotNULLOrEmpty}
    {
        echo Task failed: ${Context.Error~}
        ; Don't apply final values
    }
    else
    {
        ; Success - apply final values
        FinalValue:Set[${Context.Task.Args[to]}]
    }
break
```

### Validation Pattern

```lavishscript
method Task_MyTask()
{
    switch ${Context.TaskState}
    {
    case Start
        ; Validate all required arguments
        if !${Context.Task.Args.Has[target]}
        {
            Context:SetError["'target' is required"]
            Context.Task:Stop
            return
        }

        if !${Context.Task.Args.Has[duration]}
        {
            Context:SetError["'duration' is required"]
            Context.Task:Stop
            return
        }

        ; Validate argument types
        if ${Context.Task.Args[duration]} <= 0
        {
            Context:SetError["'duration' must be positive"]
            Context.Task:Stop
            return
        }

        ; All valid - proceed
    break

    case Continue
        ; Normal operation
    break

    case Stop
        if ${Context.Error.NotNULLOrEmpty}
        {
            echo ERROR: ${Context.Error~}
        }
        else
        {
            echo Task completed successfully
        }
    break
    }
}
```

---

## Controller Pattern

### Basic LMAC Controller

This is the recommended pattern for using LMAC:

```lavishscript
objectdef lmac_controller
{
    variable taskmanager TaskManager=${LMAC.NewTaskManager["my tasks"]}

    method Initialize()
    {
        ; Register custom task types
        This:AddTaskType["$$>
        {
            \"name\":\"custom.task1\",
            \"object\":\"MyController\",
            \"method\":\"Task_Task1\"
        }
        <$$"]

        ; Begin initial tasks
        This:StartInitialTasks
    }

    method AddTaskType(string jsonData)
    {
        variable int64 id=${LMAC.NewTaskType["${jsonData.Escape}"].ID}
        if ${id}
        {
            echo Task Type ${id} added: ${LMAC.TaskType[${id}].Name.Escape}
        }
    }

    method StartInitialTasks()
    {
        TaskManager:BeginTask["{ ... }"]
    }

    method Task_Task1()
    {
        switch ${Context.TaskState}
        {
        case Start
            ; Initialize
        break
        case Continue
            ; Update
        break
        case Stop
            ; Cleanup
        break
        }
    }

    method Shutdown()
    {
        TaskManager:Destroy
    }
}

variable(global) lmac_controller MyController

function main()
{
    while 1
        waitframe
}
```

### Why Use the Controller Pattern?

1. ✅ **Organization** - All LMAC logic in one place
2. ✅ **Global variable** - Required for task type callbacks
3. ✅ **Lifecycle management** - Initialize/Shutdown methods
4. ✅ **Reusable** - Easy to copy to other scripts
5. ✅ **Maintainable** - Clear structure

---

## Complete Examples

### Example 1: Health Bar Animation

Smoothly animate a health bar value.

```lavishscript
objectdef healthbar_controller
{
    variable taskmanager TM=${LMAC.NewTaskManager["healthbar"]}
    variable float DisplayedHealth=100.0
    variable float ActualHealth=100.0

    method Initialize()
    {
        This:AddTaskType["$$>
        {
            \"name\":\"custom.slidehealth\",
            \"object\":\"HealthBarController\",
            \"method\":\"Task_SlideHealth\"
        }
        <$$"]
    }

    method AddTaskType(string jsonData)
    {
        variable int64 id=${LMAC.NewTaskType["${jsonData.Escape}"].ID}
        if ${id}
        {
            echo Task Type added: ${LMAC.TaskType[${id}].Name.Escape}
        }
    }

    method UpdateHealth(float newHealth)
    {
        ActualHealth:Set[${newHealth}]

        ; Smoothly transition displayed health to actual
        TM:BeginTask["$$>
        {
            \"type\":\"custom.slidehealth\",
            \"duration\":0.5,
            \"to\":${newHealth}
        }
        <$$"]
    }

    method Task_SlideHealth()
    {
        switch ${Context.TaskState}
        {
        case Start
            if !${Context.Task.Args.Has[to]}
            {
                Context:SetError["'to' is required"]
                Context.Task:Stop
                return
            }

            Context.Task.Args:Set["original",${DisplayedHealth}]

            if ${Context.Task.Duration}
            {
                variable float speed=${Math.Calc[(${Context.Task.Args[to]}-${DisplayedHealth})/${Context.Task.Duration}]}
                Context.Task.Args:Set["speed",${speed}]
            }
            else
            {
                DisplayedHealth:Set[${Context.Task.Args[to]}]
                Context.Task:Stop
            }
        break

        case Continue
            DisplayedHealth:Set[${Math.Calc[${Context.Task.Args[original]}+(${Context.Task.Args[speed]}*${Context.Task.RunningTime})]}]

            ; Update UI element
            LGUI2.Element[healthbar.text]:SetText["Health: ${Math.Calc[${DisplayedHealth}.Int]}"]
        break

        case Stop
            if !${Context.Error.NotNULLOrEmpty}
            {
                DisplayedHealth:Set[${Context.Task.Args[to]}]
                LGUI2.Element[healthbar.text]:SetText["Health: ${Math.Calc[${DisplayedHealth}.Int]}"]
            }
        break
        }
    }

    method Shutdown()
    {
        TM:Destroy
    }
}

variable(global) healthbar_controller HealthBarController
```

**Usage:**
```lavishscript
HealthBarController:UpdateHealth[75.0]  ; Smoothly animate to 75%
wait 10
HealthBarController:UpdateHealth[50.0]  ; Then to 50%
```

### Example 2: Combat Music System

Dynamic music that responds to combat state.

```lavishscript
objectdef combat_music
{
    variable taskmanager TM=${LMAC.NewTaskManager["music"]}
    variable bool InCombat=FALSE

    method Initialize()
    {
        Audio:AddVoice[bgmusic]
        Audio:AddStream[explore,"music/exploration.mp3"]
        Audio:AddStream[combat,"music/combat.mp3"]

        This:PlayExploreMusic
    }

    method PlayExploreMusic()
    {
        TM:BeginTask["$$>
        {
            \"type\":\"audio.playstream\",
            \"voiceName\":\"bgmusic\",
            \"streamName\":\"explore\",
            \"playCount\":-1
        }
        <$$"]

        TM:BeginTask["$$>
        {
            \"type\":\"audio.setvolume\",
            \"duration\":2.0,
            \"voiceName\":\"bgmusic\",
            \"volume\":[0.7,0.7]
        }
        <$$"]
    }

    method PlayCombatMusic()
    {
        TM:BeginTask["$$>
        {
            \"type\":\"audio.playstream\",
            \"voiceName\":\"bgmusic\",
            \"streamName\":\"combat\",
            \"playCount\":-1
        }
        <$$"]

        TM:BeginTask["$$>
        {
            \"type\":\"audio.setvolume\",
            \"duration\":1.0,
            \"voiceName\":\"bgmusic\",
            \"volume\":[1.0,1.0]
        }
        <$$"]
    }

    method OnCombatStart()
    {
        if !${InCombat}
        {
            InCombat:Set[TRUE]
            This:PlayCombatMusic
        }
    }

    method OnCombatEnd()
    {
        if ${InCombat}
        {
            InCombat:Set[FALSE]

            ; Fade out combat music
            TM:BeginTask["$$>
            {
                \"type\":\"audio.setvolume\",
                \"duration\":2.0,
                \"voiceName\":\"bgmusic\",
                \"volume\":[0.0,0.0]
            }
            <$$"]

            ; Wait, then play explore music
            wait 20
            This:PlayExploreMusic
        }
    }

    method Shutdown()
    {
        Audio.Voice[bgmusic]:Stop:ClearQueue
        TM:Destroy
    }
}

variable(global) combat_music CombatMusic
```

### Example 3: API Data Fetcher

Fetch and cache player data from a web API.

```lavishscript
objectdef player_data_cache
{
    variable taskmanager TM=${LMAC.NewTaskManager["api"]}
    variable collection:jsonvalue CachedPlayers

    method FetchPlayerData(string playerName)
    {
        ; Check cache first
        if ${CachedPlayers.Element["${playerName}"](exists)}
        {
            echo Cached data for ${playerName}: ${CachedPlayers.Element["${playerName}"]~}
            return
        }

        ; Fetch from API
        variable jsonvalue taskDef
        taskDef:SetValue["$$>
        {
            \"type\":\"webrequest\",
            \"url\":\"https://api.example.com/player/${playerName.Escape}\",
            \"as\":\"json\",
            \"object\":\"PlayerDataCache\",
            \"method\":\"OnPlayerDataReceived\"
        }
        <$$"]

        ; Store player name for callback
        taskDef:Set["playerName","${playerName.Escape}"]

        TM:BeginTask["${taskDef~}"]
    }

    method OnPlayerDataReceived()
    {
        if ${Context.State.Equal[complete]}
        {
            variable string playerName="${Context.Task.Args[playerName]~}"
            echo Received data for ${playerName}

            ; Cache the result
            CachedPlayers:Set["${playerName}","${Context.Result~}"]

            ; Display data
            echo Name: ${Context.Result.Get[name]~}
            echo Level: ${Context.Result.Get[level]}
            echo Class: ${Context.Result.Get[class]~}
        }
        else
        {
            echo Request failed: ${Context.State}
        }
    }

    method Shutdown()
    {
        TM:Destroy
    }
}

variable(global) player_data_cache PlayerDataCache
```

**Usage:**
```lavishscript
PlayerDataCache:FetchPlayerData["Warrior"]
PlayerDataCache:FetchPlayerData["Mage"]
```

---

## Best Practices

### 1. Always Use Task Managers

```lavishscript
; GOOD
variable taskmanager TM=${LMAC.NewTaskManager["my tasks"]}
TM:BeginTask[...]
TM:Destroy  ; Clean shutdown

; BAD - No task manager (can't manage tasks)
```

### 2. Use Controller Pattern

```lavishscript
; GOOD - Organized controller
objectdef mycontroller
{
    variable taskmanager TM=${LMAC.NewTaskManager["tasks"]}

    method Initialize()
    {
        ; Setup
    }

    method Shutdown()
    {
        TM:Destroy
    }
}

; BAD - Global variables scattered everywhere
```

### 3. Validate Task Arguments

```lavishscript
; GOOD
case Start
    if !${Context.Task.Args.Has[required_param]}
    {
        Context:SetError["Missing required_param"]
        Context.Task:Stop
        return
    }
break

; BAD - No validation, will crash
case Start
    variable float value=${Context.Task.Args[required_param]}
break
```

### 4. Use $$> <$$ for Readability

```lavishscript
; GOOD - Readable
TaskManager:BeginTask["$$>
{
    \"type\":\"audio.playstream\",
    \"voiceName\":\"music\",
    \"streamName\":\"tune\"
}
<$$"]

; BAD - Hard to read
TaskManager:BeginTask["{\"type\":\"audio.playstream\",\"voiceName\":\"music\",\"streamName\":\"tune\"}"]
```

### 5. Destroy Task Managers on Shutdown

```lavishscript
; GOOD
method Shutdown()
{
    TaskManager:Destroy  ; Stops all tasks
}

function OnExit()
{
    MyController:Shutdown
}

; BAD - Tasks keep running after script ends
```

### 6. Use Error Handling

```lavishscript
; GOOD
case Stop
    if ${Context.Error.NotNULLOrEmpty}
    {
        echo Task failed: ${Context.Error~}
    }
    else
    {
        ; Apply results
    }
break

; BAD - No error checking
case Stop
    ; Always apply, even if error
break
```

### 7. Store Temporary Data in Args

```lavishscript
; GOOD
case Start
    Context.Task.Args:Set["original",${CurrentValue}]
break

case Continue
    variable float orig=${Context.Task.Args[original]}
break

; BAD - Using global/member variables (affects all instances)
variable float TempOriginal

case Start
    TempOriginal:Set[${CurrentValue}]
break
```

---

## Troubleshooting

### Issue 1: Task Type Not Found

**Problem:**
```
Error: Task type 'custom.mytask' not found
```

**Causes:**
- Task type not registered
- Typo in task type name
- Registration failed

**Fix:**
```lavishscript
; Verify registration
variable int64 id=${LMAC.NewTaskType["$$>
{
    \"name\":\"custom.mytask\",
    \"object\":\"MyController\",
    \"method\":\"Task_MyTask\"
}
<$$"].ID}

if !${id}
{
    echo ERROR: Failed to register task type
}
else
{
    echo Task type registered with ID ${id}
}
```

### Issue 2: Callback Method Not Called

**Problem:**
Task runs but callback method never executes.

**Causes:**
- Object name doesn't match global variable
- Method name typo
- Object not global

**Fix:**
```lavishscript
; BAD - Not global
variable mycontroller MyController

; GOOD - Global
variable(global) mycontroller MyController

; Task type must reference global name exactly
{
    "name": "custom.task",
    "object": "MyController",  ; Must match global variable name exactly
    "method": "Task_Handler"
}
```

### Issue 3: Task Runs Forever

**Problem:**
Task never stops.

**Causes:**
- No `Context.Task:Stop` in Start state (for instant tasks)
- No duration specified (for timed tasks)
- Logic error preventing Stop condition

**Fix:**
```lavishscript
; For instant tasks - MUST call Stop in Start
case Start
    ; Do work
    Context.Task:Stop  ; Don't forget this!
break

; For timed tasks - ensure duration is set
{
    "type": "custom.slide",
    "duration": 5.0,  ; Required for timed tasks
    "to": 100
}
```

### Issue 4: JSON Syntax Error

**Problem:**
```
Error parsing task definition
```

**Causes:**
- Invalid JSON syntax
- Missing quotes
- Unescaped quotes

**Fix:**
```lavishscript
; BAD - Missing escaped quotes
TaskManager:BeginTask["{\"type\":\"ls1.echo\",\"output\":\"Hello\"}"]
                                                        ; ^ Missing escape

; GOOD
TaskManager:BeginTask["{\"type\":\"ls1.echo\",\"output\":\"Hello\"}"]

; BETTER - Use $$><$$
TaskManager:BeginTask["$$>
{
    \"type\":\"ls1.echo\",
    \"output\":\"Hello\"
}
<$$"]
```

### Issue 5: Audio Task Not Working

**Problem:**
Audio task runs but no sound plays.

**Causes:**
- Voice not created
- Stream not added
- Volume at 0

**Fix:**
```lavishscript
; Create voice BEFORE using it
Audio:AddVoice[music]

; Add stream BEFORE playing it
Audio:AddStream[tune,"music.mp3"]

; Then play
TaskManager:BeginTask["$$>
{
    \"type\":\"audio.playstream\",
    \"voiceName\":\"music\",
    \"streamName\":\"tune\"
}
<$$"]
```

### Issue 6: Web Request Never Completes

**Problem:**
Web request task doesn't call callback.

**Causes:**
- Invalid URL
- Network error
- No callback specified

**Fix:**
```lavishscript
; Ensure callback is specified
{
    "type": "webrequest",
    "url": "https://example.com/data.json",
    "as": "json",
    "object": "MyController",  ; Required for callback
    "method": "OnComplete"     ; Required for callback
}

; Check state in callback
method OnComplete()
{
    if ${Context.State.Equal[complete]}
    {
        echo Success: ${Context.Result~}
    }
    else
    {
        echo Failed: ${Context.State}
    }
}
```

---

## Quick Reference

### Create Task Manager

```lavishscript
variable taskmanager TM=${LMAC.NewTaskManager["name"]}
```

### Begin Task

```lavishscript
TM:BeginTask["{ JSON definition }"]
```

### Register Custom Task Type

```lavishscript
noop ${LMAC.NewTaskType["$$>
{
    \"name\":\"custom.taskname\",
    \"object\":\"ControllerName\",
    \"method\":\"Task_MethodName\"
}
<$$"]}
```

### Task Method Template

```lavishscript
method Task_MethodName()
{
    switch ${Context.TaskState}
    {
    case Start
        ; Initialize, validate
    break
    case Continue
        ; Update each frame
    break
    case Stop
        ; Cleanup, finalize
    break
    }
}
```

### Destroy Task Manager

```lavishscript
TM:Destroy
```

### Common Task Types

```lavishscript
; Echo
{"type":"ls1.echo","output":"text"}

; Play audio
{"type":"audio.playstream","voiceName":"v","streamName":"s"}

; Set volume
{"type":"audio.setvolume","voiceName":"v","volume":[0.5,0.5],"duration":5.0}

; Web request
{"type":"webrequest","url":"...","as":"json","object":"Obj","method":"Method"}

; Chain
{"type":"chain","tasks":[{...},{...}]}
```

---

<!-- CLAUDE_SKIP_START -->
## Additional Resources

- **LavishMachine Documentation:** https://www.lavishsoft.com/wiki/index.php/LavishMachine
- **LERN Examples:** https://github.com/LavishSoftware/LERN/tree/master/LMAC
- **Audio Documentation:** https://github.com/LavishSoftware/LERN/tree/master/Audio
- **Web Request Documentation:** https://github.com/LavishSoftware/LERN/tree/master/Web

---

**This guide was created through comprehensive analysis of:**
- LERN/LMAC example files: https://github.com/LavishSoftware/LERN/tree/master/LMAC (1.iss, 2.iss, 3.iss, 4.iss)
- LERN/LMAC audio documentation and examples (audio-1.md, audio-1.iss)
- LERN/LMAC web request documentation and examples (webrequest-1.md, webrequest-1.iss)
- Complete coverage of task types, task managers, custom tasks, and patterns

---

*Last Updated: 2025-10-21*
*LavishMachine (LMAC) - Complete Coverage*

*Part of ISXEVE Scripting Guide*
<!-- CLAUDE_SKIP_END -->
