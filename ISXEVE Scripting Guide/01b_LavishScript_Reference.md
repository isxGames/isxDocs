# LavishScript and Inner Space Reference

**Purpose:** Exhaustive command, datatype, and Top-Level Object inventory for LavishScript core, Inner Space core, and bundled first-party subsystem packages. One canonical entry per feature, with a clickable link to the authoritative Lavish Software wiki page.

**Audience:** Scripters who already know the basics and need lookup tables. For tutorial-style introduction to the language, see [01_LavishScript_Fundamentals.md](01_LavishScript_Fundamentals.md).

**Scope:** This file covers ONLY platform features:

- LavishScript core (`Command:`, `ObjectType:`, `TLO:` namespaces)
- Inner Space core (`ISKernel:`, `ISSession:`, `ISUplink:` namespaces)
- First-party subsystem packages bundled with Inner Space (LavishGUI, LavishGUI 2, LavishNav, LavishSettings, LavishMachine)
- Optional first-party Lavish Software modules (`LSModule:*`)

This file does NOT cover any extension-provided content (game-specific TLOs, datatypes, methods, members, events, or commands belong in extension-specific guide files, not here).

**Authoritative source:** [Lavish Software wiki](https://www.lavishsoft.com/wiki/index.php/Main_Page). Every entry below links to its canonical wiki page. Where the wiki is silent on a detail, this file says so explicitly rather than inferring.

---

## Table of Contents

1. [How to Use This Reference](#1-how-to-use-this-reference)
2. [Commands](#2-commands)
   - 2.1 [LavishScript-Core Commands](#21-lavishscript-core-commands)
   - 2.2 [Inner Space Kernel Commands](#22-inner-space-kernel-commands)
   - 2.3 [Inner Space Session Commands](#23-inner-space-session-commands)
   - 2.4 [Inner Space Uplink Commands](#24-inner-space-uplink-commands)
   - 2.5 [Inner Space .NET Commands](#25-inner-space-net-commands)
   - 2.6 [LSModule Commands](#26-lsmodule-commands)
   - 2.7 [Deprecated Commands](#27-deprecated-commands)
3. [Object Types](#3-object-types)
   - 3.1 [Text](#31-text)
   - 3.2 [Numbers](#32-numbers)
   - 3.3 [Boolean](#33-boolean)
   - 3.4 [Pointer Types](#34-pointer-types)
   - 3.5 [Containers](#35-containers)
   - 3.6 [JSON](#36-json)
   - 3.7 [Iteration](#37-iteration)
   - 3.8 [Date and Time](#38-date-and-time)
   - 3.9 [Events (Object Types)](#39-events-object-types)
   - 3.10 [Other Core Types](#310-other-core-types)
   - 3.11 [File Handling](#311-file-handling)
   - 3.12 [Utilities](#312-utilities)
   - 3.13 [Tasks and LavishMachine](#313-tasks-and-lavishmachine)
   - 3.14 [XML](#314-xml)
   - 3.15 [Inner Space Audio](#315-inner-space-audio)
   - 3.16 [Inner Space Display](#316-inner-space-display)
   - 3.17 [Inner Space Distributed Scope](#317-inner-space-distributed-scope)
   - 3.18 [Inner Space Input](#318-inner-space-input)
   - 3.19 [Inner Space MIDI](#319-inner-space-midi)
   - 3.20 [Inner Space Misc](#320-inner-space-misc)
   - 3.21 [Inner Space Navigation (deprecated direct types)](#321-inner-space-navigation-deprecated-direct-types)
   - 3.22 [LavishSettings](#322-lavishsettings)
   - 3.23 [LavishNav](#323-lavishnav)
   - 3.24 [LavishGUI 1 (pointer)](#324-lavishgui-1-pointer)
   - 3.25 [LavishGUI 2 (pointer)](#325-lavishgui-2-pointer)
4. [Top-Level Objects](#4-top-level-objects)
   - 4.1 [LavishScript-Core TLOs](#41-lavishscript-core-tlos)
   - 4.2 [Inner Space Kernel TLOs](#42-inner-space-kernel-tlos)
   - 4.3 [Inner Space Session TLOs](#43-inner-space-session-tlos)
   - 4.4 [Subsystem TLOs (LGUI, LGUI2, LMAC)](#44-subsystem-tlos-lgui-lgui2-lmac)
5. [Events](#5-events)
6. [Triggers](#6-triggers)
7. [Modules (LSModule)](#7-modules-lsmodule)
8. [.NET Integration](#8-net-integration)
9. [Wiki Source Map](#9-wiki-source-map)

---

## 1. How to Use This Reference

- **Looking up a command, type, or TLO?** Use the section index above. Commands are grouped by namespace; types and TLOs are grouped by topic.
- **Need tutorial-style introduction?** See [01_LavishScript_Fundamentals.md](01_LavishScript_Fundamentals.md).
- **Cross-reference convention:** Every entry below links to its canonical wiki page. When the wiki is the right place to read more, the link goes there. When deeper coverage exists in a sibling guide file in this knowledge base, the link goes there too.
- **Provenance label convention:** Each section header notes the namespace (LavishScript core vs. Inner Space Kernel vs. subsystem package). Mixing these layers is what makes scripts non-portable to bare-LavishScript hosts (no Inner Space) or to other Inner Space sessions running different extensions.
- **Wiki-verification gaps:** Some wiki pages are stubs, missing, or contain only inline-template content. Where surface detail is not on the wiki, this file says so and provides the canonical wiki link for future reference.

---

## 2. Commands

Commands are first-class statements you type at a console or call from a script. Each entry below has the form: command name, one-line purpose, link to canonical wiki page.

### 2.1 LavishScript-Core Commands

These are documented under the `Command:` namespace. They work in any LavishScript host (Inner Space sessions, the Inner Space Uplink, and any other host that loads LavishScript). Source: [Category:LavishScript Commands](https://www.lavishsoft.com/wiki/index.php/Category:LavishScript_Commands).

#### Information

| Command | Purpose |
|---|---|
| [Commands](https://www.lavishsoft.com/wiki/index.php/Command:Commands) | List all registered commands. |
| [LSType](https://www.lavishsoft.com/wiki/index.php/Command:LSType) | Inspect a registered object type, listing its members and methods. Run with no argument to list all registered types. |
| [LSVersion](https://www.lavishsoft.com/wiki/index.php/Command:LSVersion) | Print the LavishScript engine version. |
| [Scripts](https://www.lavishsoft.com/wiki/index.php/Command:Scripts) | List currently running scripts. |
| [TopLevelObject](https://www.lavishsoft.com/wiki/index.php/Command:TopLevelObject) | List, define, or remove Top-Level Objects from the running session. Run with no argument to enumerate every registered TLO including those added by extensions. |

#### Aliases

| Command | Purpose |
|---|---|
| [Alias](https://www.lavishsoft.com/wiki/index.php/Command:Alias) | Add, list, or remove a user-defined command shortcut. See [LavishScript:Aliases](https://www.lavishsoft.com/wiki/index.php/LavishScript:Aliases). |

#### Atom Registration

| Command | Purpose |
|---|---|
| [AddAtom](https://www.lavishsoft.com/wiki/index.php/Command:AddAtom) | Register an atom by name in global scope so other scripts (and `Relay`) can invoke it. |
| [DeleteAtom](https://www.lavishsoft.com/wiki/index.php/Command:DeleteAtom) | Unregister a previously added atom. |
| [ExecuteAtom](https://www.lavishsoft.com/wiki/index.php/Command:ExecuteAtom) | Invoke a previously registered atom by name. The mechanism `Relay` uses under the hood for cross-session atom invocation. |

#### Command Execution

| Command | Purpose |
|---|---|
| [Execute](https://www.lavishsoft.com/wiki/index.php/Command:Execute) | Execute an arbitrary command string (often constructed dynamically from data). |
| [ExecuteFile](https://www.lavishsoft.com/wiki/index.php/Command:ExecuteFile) | Execute a file of commands as a batch script. |
| [NoOp](https://www.lavishsoft.com/wiki/index.php/Command:NoOp) | Do nothing. The canonical placeholder for branches that should take no action. |
| [NoParse](https://www.lavishsoft.com/wiki/index.php/Command:NoParse) | Run a command without parsing data sequences. Useful for echoing literal `${...}` text. |
| [Redirect](https://www.lavishsoft.com/wiki/index.php/Command:Redirect) | Redirect console output of a command to a file. |
| [Squelch](https://www.lavishsoft.com/wiki/index.php/Command:Squelch) | Run a command silently with no console output. |
| [TimedCommand](https://www.lavishsoft.com/wiki/index.php/Command:TimedCommand) | Schedule a command for execution after a delay (deciseconds). Non-blocking; the engine fires it later. Usable inside atoms (which cannot `wait`). |

#### Script Lifecycle

| Command | Purpose |
|---|---|
| [RunScript](https://www.lavishsoft.com/wiki/index.php/Command:RunScript) | Start a script. The bare `run myscript` shortcut form invokes this. |
| [EndScript](https://www.lavishsoft.com/wiki/index.php/Command:EndScript) | Stop a script by name. Inside a script, `endscript` with no argument ends the calling script. |
| [WaitScript](https://www.lavishsoft.com/wiki/index.php/Command:WaitScript) | Pause the calling script until a named script finishes. |

#### Variable Management

| Command | Purpose |
|---|---|
| [DeclareVariable](https://www.lavishsoft.com/wiki/index.php/Command:DeclareVariable) | Declare a variable at runtime (the runtime form of the `variable` keyword). Used from console or for dynamic variable creation. |
| [DeleteVariable](https://www.lavishsoft.com/wiki/index.php/Command:DeleteVariable) | Remove a previously declared variable. |

#### Function Calls

| Command | Purpose |
|---|---|
| [Call](https://www.lavishsoft.com/wiki/index.php/Command:Call) | Invoke a function in the current script. |
| [Return](https://www.lavishsoft.com/wiki/index.php/Command:Return) | Return from the current function, optionally with a value (accessible to the caller via the [Return TLO](https://www.lavishsoft.com/wiki/index.php/TLO:Return)). |

#### Command Queue

| Command | Purpose |
|---|---|
| [QueueCommand](https://www.lavishsoft.com/wiki/index.php/Command:QueueCommand) | Append a command to the script's command queue for later execution by the main loop. The canonical pattern for deferring work out of an atom. |
| [ExecuteQueued](https://www.lavishsoft.com/wiki/index.php/Command:ExecuteQueued) | Run all queued commands now. |
| [FlushQueued](https://www.lavishsoft.com/wiki/index.php/Command:FlushQueued) | Discard all queued commands without running them. |

The current queue is exposed via the [QueuedCommands TLO](https://www.lavishsoft.com/wiki/index.php/TLO:QueuedCommands).

#### Triggers

Triggers fire when a registered text pattern is seen in console output (see also [LavishScript:Triggers](https://www.lavishsoft.com/wiki/index.php/LavishScript:Triggers)).

| Command | Purpose |
|---|---|
| [AddTrigger](https://www.lavishsoft.com/wiki/index.php/Command:AddTrigger) | Register a text-pattern trigger that fires a command when matched. |
| [RemoveTrigger](https://www.lavishsoft.com/wiki/index.php/Command:RemoveTrigger) | Unregister a previously added trigger. |
| [WaitFor](https://www.lavishsoft.com/wiki/index.php/Command:WaitFor) | Pause the current script until a text pattern is seen or a timeout elapses. |

#### Timing

| Command | Purpose |
|---|---|
| [Wait](https://www.lavishsoft.com/wiki/index.php/Command:Wait) | Pause execution. Forms: `Wait <deciseconds>`, `Wait <deciseconds> <condition>` (early-continue), `Wait -s <seconds>`. Frame-quantized; sub-frame waits round up to one frame. |
| [WaitFrame](https://www.lavishsoft.com/wiki/index.php/Command:WaitFrame) | Pause until the next frame. The canonical yield in long-running loops. |
| [Turbo](https://www.lavishsoft.com/wiki/index.php/Command:Turbo) | Set the maximum number of commands the engine will execute per frame before yielding. Default 80. Raising it lets a script do more per frame at the cost of frame time. |

#### File System

Documented at [LavishScript:File System](https://www.lavishsoft.com/wiki/index.php/LavishScript:File_System). All take Unix-style paths.

| Command | Purpose |
|---|---|
| [Cat](https://www.lavishsoft.com/wiki/index.php/Command:Cat) | Print file contents to console. |
| [cd](https://www.lavishsoft.com/wiki/index.php/Command:cd) | Change current directory. |
| [cp](https://www.lavishsoft.com/wiki/index.php/Command:cp) | Copy a file. |
| [Head](https://www.lavishsoft.com/wiki/index.php/Command:Head) | Print first N lines of a file. |
| [Line](https://www.lavishsoft.com/wiki/index.php/Command:Line) | Print a specific line from a file. |
| [mkdir](https://www.lavishsoft.com/wiki/index.php/Command:mkdir) | Create a directory. |
| [rename](https://www.lavishsoft.com/wiki/index.php/Command:rename) | Rename a file. |
| [rm](https://www.lavishsoft.com/wiki/index.php/Command:rm) | Delete a file. |
| [rmdir](https://www.lavishsoft.com/wiki/index.php/Command:rmdir) | Remove an empty directory. |
| [Tail](https://www.lavishsoft.com/wiki/index.php/Command:Tail) | Print last N lines of a file. |

#### Operating System

| Command | Purpose |
|---|---|
| [OSExecute](https://www.lavishsoft.com/wiki/index.php/Command:OSExecute) | Spawn an OS-level process from inside a script. Avoid passing untrusted input. |
| [Processor](https://www.lavishsoft.com/wiki/index.php/Command:Processor) | Print CPU information. |

#### Output

| Command | Purpose |
|---|---|
| [Echo](https://www.lavishsoft.com/wiki/index.php/Command:Echo) | Write text to the console. |

(`Squelch` is listed under Command Execution above.)

#### Modules

| Command | Purpose |
|---|---|
| [Module](https://www.lavishsoft.com/wiki/index.php/Command:Module) | Load, unload, or list LavishScript modules. See [LavishScript:Modules](https://www.lavishsoft.com/wiki/index.php/LavishScript:Modules) and section [7. Modules](#7-modules-lsmodule) below. |

#### Testing

| Command | Purpose |
|---|---|
| [Arg](https://www.lavishsoft.com/wiki/index.php/Command:Arg) | Access script command-line arguments by index (also available as the [Arg TLO](https://www.lavishsoft.com/wiki/index.php/TLO:Arg)). |
| [Test](https://www.lavishsoft.com/wiki/index.php/Command:Test) | Development helper for testing expressions and data sequences from the console. |

### 2.2 Inner Space Kernel Commands

Documented under the `ISKernel:` namespace. Available in any Inner Space session and the Uplink. Source: [ISKernel:Commands](https://www.lavishsoft.com/wiki/index.php/ISKernel:Commands).

#### Console

| Command | Purpose |
|---|---|
| [Console](https://www.lavishsoft.com/wiki/index.php/ISKernel:Console_(Command)) | Open, close, or toggle the console window. |
| [ConsoleClear](https://www.lavishsoft.com/wiki/index.php/ISKernel:ConsoleClear_(Command)) | Clear the console buffer. |
| [Echo](https://www.lavishsoft.com/wiki/index.php/ISKernel:Echo_(Command)) | Write text to the console. Forms: `Echo <text>`, `Echo on`/`Echo off` to enable/disable console output globally. |
| [Log](https://www.lavishsoft.com/wiki/index.php/ISKernel:Log_(Command)) | Log all console output to a file. |
| [Squelch](https://www.lavishsoft.com/wiki/index.php/ISKernel:Squelch_(Command)) | Run a command silently. |

#### Display

| Command | Purpose |
|---|---|
| [DisplayInfo](https://www.lavishsoft.com/wiki/index.php/ISKernel:DisplayInfo_(Command)) | Print adapter resolution, display mode, viewable size, scale, distortion. |
| [FPS](https://www.lavishsoft.com/wiki/index.php/ISKernel:FPS_(Command)) | Display the current framerate. |
| [Gamma](https://www.lavishsoft.com/wiki/index.php/ISKernel:Gamma_(Command)) | Set, store, restore, reset, or display the gamma level. |
| [MaxFPS](https://www.lavishsoft.com/wiki/index.php/ISKernel:MaxFPS_(Command)) | Set or display the framerate / CPU-limiter cap. |
| [Wireframe](https://www.lavishsoft.com/wiki/index.php/ISKernel:Wireframe_(Command)) | Toggle wireframe rendering (does not improve performance). |

`RestoreGamma` is the `-restore` form of [Gamma](https://www.lavishsoft.com/wiki/index.php/ISKernel:Gamma_(Command)), not a separate command.

#### Window

| Command | Purpose |
|---|---|
| [ClipMouse](https://www.lavishsoft.com/wiki/index.php/ISKernel:ClipMouse_(Command)) | Prevent the mouse from leaving the window (not used by DirectInput-based games). |
| [WindowFrame](https://www.lavishsoft.com/wiki/index.php/ISKernel:WindowFrame_(Command)) | Set window frame style. |
| [WindowPos](https://www.lavishsoft.com/wiki/index.php/ISKernel:WindowPos_(Command)) | Set or display window position. |
| [WindowScale](https://www.lavishsoft.com/wiki/index.php/ISKernel:WindowScale_(Command)) | Set window size relative to the host's resolution. |
| [WindowSize](https://www.lavishsoft.com/wiki/index.php/ISKernel:WindowSize_(Command)) | Set or display window size. |
| [WindowTaskbar](https://www.lavishsoft.com/wiki/index.php/ISKernel:WindowTaskbar_(Command)) | Add or remove a system-tray icon for this session. |
| [WindowText](https://www.lavishsoft.com/wiki/index.php/ISKernel:WindowText_(Command)) | Set the window title. |
| [WindowVisibility](https://www.lavishsoft.com/wiki/index.php/ISKernel:WindowVisibility_(Command)) | Move the window above or below others, or toggle "always on top". |

#### Input Emulation

| Command | Purpose |
|---|---|
| [Bind](https://www.lavishsoft.com/wiki/index.php/ISKernel:Bind_(Command)) | Add, list, or remove a session-level hotkey. Forms: `Bind -keylist`, `Bind -list`, `Bind -clear`, `Bind -delete <name>`, `Bind [-press\|-release] <name> <combo> <command>`. |
| [GlobalBind](https://www.lavishsoft.com/wiki/index.php/ISKernel:GlobalBind_(Command)) | Add a Windows-global hotkey usable from any window/desktop. Forms mirror `Bind`. |
| [Press](https://www.lavishsoft.com/wiki/index.php/ISKernel:Press_(Command)) | Emulate a key press / hold / release. Forms: `Press -keylist`, `Press <combo>`, `Press -hold <combo>`, `Press -release <combo>`, `Press -nomodifiers <combo>`. |
| [Type](https://www.lavishsoft.com/wiki/index.php/ISKernel:Type_(Command)) | Emulate typing text into the session (does not send Enter). |
| [MouseTo](https://www.lavishsoft.com/wiki/index.php/ISKernel:MouseTo_(Command)) | Move the mouse cursor to a coordinate. |
| [MouseClick](https://www.lavishsoft.com/wiki/index.php/ISKernel:MouseClick_(Command)) | Emulate a mouse-button press or release. Forms: `MouseClick -hold <left\|right>`, `MouseClick -release <left\|right>`. |
| [MouseCursor](https://www.lavishsoft.com/wiki/index.php/ISKernel:MouseCursor_(Command)) | Show or hide the mouse cursor. |
| [DIMouse](https://www.lavishsoft.com/wiki/index.php/ISKernel:DIMouse_(Command)) | Capture or release the mouse for DirectInput-based games. |
| [Macro](https://www.lavishsoft.com/wiki/index.php/ISKernel:Macro_(Command)) | Record or play back keyboard and mouse macros. |

`MouseWheel` (in the user inventory) is documented on the wiki as the `MouseWheelUp` / `MouseWheelDown` key names usable with `Press` and `Bind`, not a separate command. Wiki has no standalone `MouseWheel` command page (verified 2026-04-27).

#### Extensions and Services

| Command | Purpose |
|---|---|
| [Extension](https://www.lavishsoft.com/wiki/index.php/ISKernel:Extension_(Command)) | Load, unload, or list Inner Space extensions. |
| [Services](https://www.lavishsoft.com/wiki/index.php/ISKernel:Services_(Command)) | List registered extension services and their client counts. |
| [HTTPGet](https://www.lavishsoft.com/wiki/index.php/ISKernel:HTTPGet_(Command)) | One-shot HTTP GET. Lower-level than the [webrequest object type](#320-inner-space-misc). |

#### HUD

| Command | Purpose |
|---|---|
| [HUD](https://www.lavishsoft.com/wiki/index.php/ISKernel:HUD_(Command)) | Add, remove, or list elements in the heads-up display. |
| [HUDGroup](https://www.lavishsoft.com/wiki/index.php/ISKernel:HUDGroup_(Command)) | Hide or show a group of HUD elements. |
| [HUDSet](https://www.lavishsoft.com/wiki/index.php/ISKernel:HUDSet_(Command)) | Modify a HUD element's properties. |

#### Settings

| Command | Purpose |
|---|---|
| [XMLSetting](https://www.lavishsoft.com/wiki/index.php/ISKernel:XMLSetting_(Command)) | Add, list, or remove XML setting nodes. The runtime command form of LavishSettings (see also section [3.22](#322-lavishsettings)). |

#### Misc

| Command | Purpose |
|---|---|
| [Version](https://www.lavishsoft.com/wiki/index.php/ISKernel:Version_(Command)) | Print the Inner Space version. |
| [Exit](https://www.lavishsoft.com/wiki/index.php/ISKernel:Exit_(Command)) | Close this session immediately. |
| [Diagnostics](https://www.lavishsoft.com/wiki/index.php/Special:Search?search=Diagnostics&go=Go) | Print diagnostic information about the running Inner Space instance. (Wiki dedicated page not found by direct lookup; the command is documented in the Inner Space command set. Pending wiki-link verification.) |

### 2.3 Inner Space Session Commands

Documented under the `ISSession:` namespace. Available inside a launched session. Source: [Category:Inner Space Session Commands](https://www.lavishsoft.com/wiki/index.php/Category:Inner_Space_Session_Commands).

| Command | Purpose |
|---|---|
| [ConsoleVisibleLines](https://www.lavishsoft.com/wiki/index.php/ISSession:ConsoleVisibleLines_(Command)) | Set the number of console lines visible. |
| [EndRecord](https://www.lavishsoft.com/wiki/index.php/ISSession:EndRecord_(Command)) | Stop a previously started recording. |
| [FileRedirect](https://www.lavishsoft.com/wiki/index.php/ISSession:FileRedirect_(Command)) | Redirect file access for the host process. |
| [Game](https://www.lavishsoft.com/wiki/index.php/ISSession:Game_(Command)) | Configured-game info for this session. |
| [Games](https://www.lavishsoft.com/wiki/index.php/ISSession:Games_(Command)) | List configured games visible to this session. |
| [HUDAdd](https://www.lavishsoft.com/wiki/index.php/ISSession:HUDAdd_(Command)) | Add an HUD element. (Session-level analog of `HUD -add`.) |
| [HUDList](https://www.lavishsoft.com/wiki/index.php/ISSession:HUDList_(Command)) | List session HUD elements. |
| [HUDRemove](https://www.lavishsoft.com/wiki/index.php/ISSession:HUDRemove_(Command)) | Remove a session HUD element. |
| [IniRedirect](https://www.lavishsoft.com/wiki/index.php/ISSession:IniRedirect_(Command)) | Redirect INI file access for the host process. |
| [Profile](https://www.lavishsoft.com/wiki/index.php/ISSession:Profile_(Command)) | Print or set the launch profile of this session. |
| [Profiles](https://www.lavishsoft.com/wiki/index.php/ISSession:Profiles_(Command)) | List configured profiles. |
| [Record](https://www.lavishsoft.com/wiki/index.php/ISSession:Record_(Command)) | Start recording. |
| [Uplink](https://www.lavishsoft.com/wiki/index.php/ISSession:Uplink_(Command)) | Send a command to the local Uplink (counterpart to `Relay`). |

### 2.4 Inner Space Uplink Commands

Documented under the `ISUplink:` namespace. Available in the Inner Space Uplink (the multi-boxing coordinator process). Source: [Category:Inner Space Uplink Commands](https://www.lavishsoft.com/wiki/index.php/Category:Inner_Space_Uplink_Commands).

| Command | Purpose |
|---|---|
| [Console](https://www.lavishsoft.com/wiki/index.php/ISUplink:Console_(Command)) | Open / close / toggle the Uplink console. |
| [ConsoleClear](https://www.lavishsoft.com/wiki/index.php/ISUplink:ConsoleClear_(Command)) | Clear the Uplink console. |
| [Echo](https://www.lavishsoft.com/wiki/index.php/ISUplink:Echo_(Command)) | Echo to the Uplink console. |
| [Exit](https://www.lavishsoft.com/wiki/index.php/ISUplink:Exit_(Command)) | Close the Uplink. |
| [Focus](https://www.lavishsoft.com/wiki/index.php/ISUplink:Focus_(Command)) | Bring a session to foreground. Forms include `-next`, `-previous`, named session, indexed session. |
| [Game](https://www.lavishsoft.com/wiki/index.php/ISUplink:Game_(Command)) | Configured game info from the Uplink side. |
| [Games](https://www.lavishsoft.com/wiki/index.php/ISUplink:Games_(Command)) | List configured games. |
| [Log](https://www.lavishsoft.com/wiki/index.php/ISUplink:Log_(Command)) | Log Uplink console output to a file. |
| [MakeShortcut](https://www.lavishsoft.com/wiki/index.php/ISUplink:MakeShortcut_(COmmand)) | Create a Windows shortcut that launches a specific game/profile via Inner Space. |
| [Name](https://www.lavishsoft.com/wiki/index.php/ISUplink:Name_(Command)) | Get or set the Uplink's name. |
| [Open](https://www.lavishsoft.com/wiki/index.php/ISUplink:Open_(Command)) | Open a session by name/profile. |
| [Profile](https://www.lavishsoft.com/wiki/index.php/ISUplink:Profile_(Command)) | Profile info from the Uplink. |
| [Profiles](https://www.lavishsoft.com/wiki/index.php/ISUplink:Profiles_(Command)) | List profiles. |
| [Relay](https://www.lavishsoft.com/wiki/index.php/ISUplink:Relay_(Command)) | Send a command asynchronously to a session, group, or all sessions. Forms: `Relay <session\|all\|all local> [-echo\|-event <name>\|-noredirect] <command>`. The canonical IPC mechanism. |
| [RemoteUplink](https://www.lavishsoft.com/wiki/index.php/ISUplink:RemoteUplink_(Command)) | Connect to or manage remote Inner Space Uplinks. |
| [Sessions](https://www.lavishsoft.com/wiki/index.php/ISUplink:Sessions_(Command)) | List sessions. |
| [Squelch](https://www.lavishsoft.com/wiki/index.php/ISUplink:Squelch_(Command)) | Squelch Uplink-side. |
| [Version](https://www.lavishsoft.com/wiki/index.php/ISUplink:Version_(Command)) | Uplink version. |

`RelayTargets` (in the user inventory) is not present as a standalone wiki page; it is functionally the addressing dimension of `Relay` (`<session>` / `all` / `all local` / named group). Pending wiki-page verification if it is actually a separate command in newer builds.

### 2.5 Inner Space .NET Commands

| Command | Purpose |
|---|---|
| [DotNet](https://www.lavishsoft.com/wiki/index.php/ISKernel:DotNet_(Command)) | Run .NET 2.0/4.0 code from a script. See also the platform-side [IS:.NET wiki](https://www.lavishsoft.com/wiki/index.php/IS:.NET). |

`DotScript` (in the user inventory) does not have a dedicated `Command:DotScript` page on the wiki and is not in any canonical category listing as of 2026-04-27. It may be a build-specific or obsolete name; pending wiki verification. Game-extension-specific .NET wrappers belong in extension-specific guide files, not here.

### 2.6 LSModule Commands

The first-party optional Lavish Software modules each register their own commands when loaded. The most commonly cited:

| Command | Source module | Purpose |
|---|---|---|
| [tar](https://www.lavishsoft.com/wiki/index.php/LSModule:Targz:_tar_(Command)) | [LSModule:Targz](https://www.lavishsoft.com/wiki/index.php/LSModule:Targz) | Read or extract `.tar.gz` archives. |

Additional modules add commands of their own; see section [7. Modules](#7-modules-lsmodule) for the full module list and their wiki entry points.

### 2.7 Deprecated Commands

The wiki explicitly flags these as deprecated. Do not use in new scripts.

| Command | Replacement / note |
|---|---|
| [APICall](https://www.lavishsoft.com/wiki/index.php/Command:APICall) | Deprecated. Not available in 64-bit builds. Use modern equivalents (DotNet for arbitrary code, dedicated commands for specific tasks). |
| [Navigation](https://www.lavishsoft.com/wiki/index.php/ISKernel:Navigation_(Command)) | Deprecated direct command form. Use [LavishNav](https://www.lavishsoft.com/wiki/index.php/LavishNav). |
| [NavPath](https://www.lavishsoft.com/wiki/index.php/ISKernel:NavPath_(Command)) | Deprecated. See LavishNav. |
| [NavPoint](https://www.lavishsoft.com/wiki/index.php/ISKernel:NavPoint_(Command)) | Deprecated. See LavishNav. |

---

## 3. Object Types

LavishScript object types describe a class of objects. Variables of a given type expose that type's **members** (`.dot` access, retrieve a value) and **methods** (`:colon` access, perform an action). A type may **inherit** another, in which case its variables expose the inherited type's members and methods too.

Each entry below has the form: type name, key surface (members and methods), link to canonical wiki page. Where the wiki page provides the authoritative full surface, this section reproduces only the most-used members/methods and points readers at the wiki for the rest.

### 3.1 Text

[`string`](https://www.lavishsoft.com/wiki/index.php/ObjectType:string) -- a series of characters. Reduces to the stored text. Variable declaration auto-routes to [`mutablestring`](https://www.lavishsoft.com/wiki/index.php/ObjectType:mutablestring).

Members:

| Member | Returns | Description |
|---|---|---|
| `Length` | int | Number of characters. |
| `Upper` | string | All upper case. |
| `Lower` | string | All lower case. |
| `Mid[pos,len]` | string | Substring of `len` chars starting at 1-based `pos`. |
| `Left[#]` | string | Leftmost `#` chars. Negative `#` = leftmost (Length-#). |
| `Right[#]` | string | Rightmost `#` chars. Negative `#` = rightmost (Length-#). |
| `Find[text]` | int | 1-based position of substring, or NULL. |
| `Count[char]` | int | Number of occurrences of a single character. |
| `Token[#,sep]` | string | The #th token after splitting on `sep`. |
| `Compare[text]` | int | Case-insensitive dictionary compare. |
| `CompareCS[text]` | int | Case-sensitive compare. |
| `Equal[text]` | bool | Case-insensitive equality. |
| `EqualCS[text]` | bool | Case-sensitive equality. |
| `NotEqual[text]` | bool | Case-insensitive inequality. |
| `NotEqualCS[text]` | bool | Case-sensitive inequality. |
| `Escape` | string | Backslash-escape `\`, `"`, CR/LF/tab, and LavishScript data sequences. |
| `Escape[bool]` | string | If `FALSE`, do not escape data sequences. |
| `EscapeQuotes` | string | Escape only `"`. |
| `Replace[char,with,...]` | string | Single-character pair replacement. |
| `ReplaceSubstring[needle,replacement]` | string | Substring replacement. |
| `GetAt[#]` | byte | Character byte at 1-based position. |
| `UniString` | unistring | Convert to UTF-16. |
| `URLEncode` | string | Percent-encoded form, suitable for URLs. |
| `NotNULLOrEmpty` | bool | TRUE if non-empty and not the literal `NULL`. |
| `Trim` | string | Strip leading/trailing whitespace. |
| `AsJSON` | string | JSON-encoded form including surrounding quotes. |

Methods: none on `string` (immutable). For mutation use [`mutablestring`](https://www.lavishsoft.com/wiki/index.php/ObjectType:mutablestring), which adds `Set[text]`, `Concat[text]`, etc.

[`unistring`](https://www.lavishsoft.com/wiki/index.php/ObjectType:unistring) -- UTF-16 Unicode counterpart to `string`. Inherits from `string`, so all string members and methods work. Adds:

| Member | Returns | Description |
|---|---|---|
| `String` | string | Convert to UTF-8 string. |

### 3.2 Numbers

[`int`](https://www.lavishsoft.com/wiki/index.php/ObjectType:int) -- 32-bit signed integer (-2,147,483,648 to 2,147,483,647).

| Member | Returns | Description |
|---|---|---|
| `Float` | float | Convert to float. |
| `Hex` | string | Hexadecimal representation. |
| `Reverse` | int | Bytes reversed. |
| `LeadingZeroes[#]` | string | Decimal string padded to at least `#` digits. |
| `Unsigned` | uint | As 32-bit unsigned. |
| `Between[a,b]` | bool | TRUE if a <= value <= b. |
| `Equal[formula]` | bool | TRUE if value matches the formula's result. |
| `AsJSON` | string | JSON value. |

Methods: `Inc`, `Inc[formula]`, `Dec`, `Dec[formula]`, `Set[formula]`.

[`uint`](https://www.lavishsoft.com/wiki/index.php/ObjectType:uint) -- 32-bit unsigned integer (0 to 4,294,967,295).
[`int64`](https://www.lavishsoft.com/wiki/index.php/ObjectType:int64) -- 64-bit signed integer.
[`byte`](https://www.lavishsoft.com/wiki/index.php/ObjectType:byte) -- 8-bit unsigned.

[`float`](https://www.lavishsoft.com/wiki/index.php/ObjectType:float) -- single-precision floating point. Reduces to the value rounded to the nearest hundredth.

| Member | Returns | Description |
|---|---|---|
| `Deci` | string | To the nearest tenth. |
| `Centi` | string | To the nearest hundredth. |
| `Milli` | string | To the nearest thousandth. |
| `Int` | int | Largest whole number not exceeding the value (floor). |
| `Precision[#]` | string | Rounded to `#` decimal places. |
| `Ceil` | int | Smallest whole number not less than the value. |
| `Round` | int | Rounded to nearest whole number. |
| `Between[a,b]` | bool | TRUE if a <= value <= b. |
| `Equal[formula]` | bool | TRUE if value matches the formula's result. |
| `AsJSON` | string | JSON value. |

Methods: `Inc`, `Inc[formula]`, `Dec`, `Dec[formula]`, `Set[formula]`.

[`float64`](https://www.lavishsoft.com/wiki/index.php/ObjectType:float64) -- double-precision floating point.

### 3.3 Boolean

[`bool`](https://www.lavishsoft.com/wiki/index.php/ObjectType:bool) -- groups all numbers into TRUE (non-zero) and FALSE (zero).

| Member | Returns | Description |
|---|---|---|
| `Not` | bool | Opposite value. |
| `Equal[calc]` | bool | TRUE if matches calculation's boolean result. |
| `AsJSON` | string | `true` or `false`. |

Methods: `Toggle`, `Set[formula]`.

### 3.4 Pointer Types

Pointer types are typed memory addresses, used by extensions and rarely directly from scripts. Listed here for completeness. Surface is generally a single `Set[value]` method and member-style dereference.

| Type | Description |
|---|---|
| [`boolptr`](https://www.lavishsoft.com/wiki/index.php/ObjectType:boolptr) | Pointer to bool. |
| [`byteptr`](https://www.lavishsoft.com/wiki/index.php/ObjectType:byteptr) | Pointer to byte. |
| [`floatptr`](https://www.lavishsoft.com/wiki/index.php/ObjectType:floatptr) | Pointer to float. |
| [`float64ptr`](https://www.lavishsoft.com/wiki/index.php/ObjectType:float64ptr) | Pointer to float64. |
| [`intptr`](https://www.lavishsoft.com/wiki/index.php/ObjectType:intptr) | Pointer to int. |
| [`uintptr`](https://www.lavishsoft.com/wiki/index.php/ObjectType:uintptr) | Pointer to uint. |
| [`int64ptr`](https://www.lavishsoft.com/wiki/index.php/ObjectType:int64ptr) | Pointer to int64. |
| [`rgbptr`](https://www.lavishsoft.com/wiki/index.php/ObjectType:rgbptr) | Pointer to rgb. |
| [`stringptr`](https://www.lavishsoft.com/wiki/index.php/ObjectType:stringptr) | Pointer to string. |

### 3.5 Containers

All container types inherit from [`objectcontainer`](https://www.lavishsoft.com/wiki/index.php/ObjectType:objectcontainer) and require a sub-type when declared (`variable index:string`, `variable collection:int`, etc.).

[`index`](https://www.lavishsoft.com/wiki/index.php/ObjectType:index) -- dynamically sized 1-based list.

| Member | Returns | Description |
|---|---|---|
| `[#]`, `Get[#]` | sub-type | Element at 1-based position. |
| `Used` | int | Number of currently-existing elements. |
| `Size` | int | Allocated capacity. |
| `Next[#]` | uint | ID of next valid element after `#`. |
| `Insert[...]` | uint | Append element (auto-resize). Returns new ID. Args passed to sub-type's Initialize. |
| `Expand[start,len]` | mutablestring | Quoted-space-separated text representation. |
| `ExpandComma[start,len]` | mutablestring | Quoted-comma-separated text representation. |
| `AsJSON` | unistring | JSON array. |

Methods: `Insert[...]`, `Set[#,...]`, `Remove[#]`, `RemoveByQuery[query_id[,remove_MATCHES]]`, `Collapse` (close gaps), `Move[#,#]`, `Swap[#,#]`, `Shift[pos,places]`, `Resize[#]`, `Clear`, `ForEach[code]`, `FromJSON[json]`.

[`collection`](https://www.lavishsoft.com/wiki/index.php/ObjectType:collection) -- sorted key-value map. Keys are strings, case-insensitive.

| Member | Returns | Description |
|---|---|---|
| `Element[key]` | sub-type | Value for key. |
| `FirstKey` / `NextKey` / `CurrentKey` | string | Built-in iterator over keys. |
| `FirstValue` / `NextValue` / `CurrentValue` | sub-type | Built-in iterator over values. |
| `Type` | type | The contained sub-type. |
| `AsJSON` | unistring | JSON object (default) or array (`AsJSON[array]`). |

Methods: `Set[key,value]`, `Erase[key]`, `EraseByQuery[query_id[,remove_MATCHES]]`, `ForEach[code]`, `GetIterator[iterator_var]`.

[`array`](https://www.lavishsoft.com/wiki/index.php/ObjectType:array) -- fixed-size, multi-dimensional array. Elements exist for the entire array's lifetime (vs. `index` where elements come into being on Insert).

[`queue`](https://www.lavishsoft.com/wiki/index.php/ObjectType:queue) -- FIFO container. `Push`/`Pop`/`Peek`.
[`stack`](https://www.lavishsoft.com/wiki/index.php/ObjectType:stack) -- LIFO container. `Push`/`Pop`/`Peek`.
[`set`](https://www.lavishsoft.com/wiki/index.php/ObjectType:set) -- unordered set of unique keys.
[`variablescope`](https://www.lavishsoft.com/wiki/index.php/ObjectType:variablescope) -- a runtime container of variables. Used to expose script-scope or global-scope from outside.
[`objectcontainer`](https://www.lavishsoft.com/wiki/index.php/ObjectType:objectcontainer) -- abstract base of all containers.

### 3.6 JSON

JSON support is built into the engine via these types. See also [13_JSON_Guide.md](13_JSON_Guide.md) for usage patterns.

[`jsonvalue`](https://www.lavishsoft.com/wiki/index.php/ObjectType:jsonvalue) -- an immutable JSON value.

| Member | Returns | Description |
|---|---|---|
| `AsString` | string | Value as string. |
| `AsJSON` | string | Single-line JSON text. |
| `AsJSON[multiline]` | string | Multi-line JSON text. |
| `Type` | string | One of `null`, `object`, `string`, `number`, `array`, `true`, `false`, `integer`. |
| `Value` | varies | The contained value (typed by `Type`). |

Methods: `WriteFile[filename]`, `WriteFile[filename,multiline[,line_splitter]]`, `ParseFile[filename]`. The variable form is actually [`jsonvaluecontainer`](https://www.lavishsoft.com/wiki/index.php/ObjectType:jsonvaluecontainer) which adds `Set[key,value]`, `Add[value]`, `[key]` indexing, `Get[#]`, `Erase[key]`, etc.

[`jsonobject`](https://www.lavishsoft.com/wiki/index.php/ObjectType:jsonobject) -- a JSON object node specifically.
[`jsonarray`](https://www.lavishsoft.com/wiki/index.php/ObjectType:jsonarray) -- a JSON array node specifically.
[`jsonvaluecontainer`](https://www.lavishsoft.com/wiki/index.php/ObjectType:jsonvaluecontainer) -- the mutable container behind a `jsonvalue` variable.

### 3.7 Iteration

[`iterator`](https://www.lavishsoft.com/wiki/index.php/ObjectType:iterator) -- traverses a container's contents. Surface depends on the container being iterated.

| Member | Returns | Description |
|---|---|---|
| `Target` | varies | The container being iterated. |
| `Key` | varies | Current key (type depends on container). |
| `Value` | varies | Current value (type depends on container). |
| `IsValid` | bool | Iterator points to a valid position. |
| `Reversible` | bool | Container supports backward iteration. |
| `Constant` | bool | Container disallows `SetValue`. |
| `RandomAccess` | bool | Container supports `Jump`. |

Methods: `First`, `Last`, `Next`, `Previous`, `SetValue[...]`, `Jump[...]`.

[`jsoniterator`](https://www.lavishsoft.com/wiki/index.php/Special:Search?search=jsoniterator&go=Go) -- iterator specialization for JSON traversal. (Wiki page fetch returned no dedicated `ObjectType:jsoniterator`; surface is documented inline on JSON-related pages. Pending wiki-link verification.)

The [`ForEach`](https://www.lavishsoft.com/wiki/index.php/TLO:ForEach) TLO is the canonical compact alternative to manual iterator setup; see section [4.1](#41-lavishscript-core-tlos).

### 3.8 Date and Time

[`time`](https://www.lavishsoft.com/wiki/index.php/ObjectType:time) -- date and time of day. Reduces to `Time24`.

| Member | Returns | Description |
|---|---|---|
| `Hour` / `Minute` / `Second` | intptr | Time-of-day components (0-23, 0-59, 0-59). |
| `Day` / `Month` / `Year` | intptr / int | Date components. |
| `MonthPtr` / `YearPtr` / `DayOfWeekPtr` | intptr | "Set"-friendly forms (Month-1, Year-1900, DayOfWeek-1). |
| `DayOfWeek` | int | Day of week (1-7). |
| `Time12` / `Time24` | string | Formatted time. |
| `Date` | string | mm/dd/yyyy. |
| `Night` | bool | TRUE between 7pm and 7am. |
| `SecondsSinceMidnight` | int | Seconds since 00:00. |
| `Timestamp` | uint | UNIX timestamp (seconds since epoch). |

Methods: `Set[timestamp]`, `Update` (recompute after setting individual components).

### 3.9 Events (Object Types)

[`event`](https://www.lavishsoft.com/wiki/index.php/ObjectType:event) -- a registered LavishScript event.

| Member | Returns | Description |
|---|---|---|
| `ID` | int | Internal event ID (used by modules / C API). |
| `Name` | string | Event name. |

Methods: `AttachAtom[name]`, `DetachAtom[name]`, `Execute[...]`, `ThisExecute[object,...]`, `Clear`, `Unregister`.

[`anonevent`](https://www.lavishsoft.com/wiki/index.php/Special:Search?search=anonevent&go=Go) -- anonymous event variant. (Wiki dedicated page not located; pending verification.)

See section [5. Events](#5-events) for the lifecycle, and [LavishScript:Events](https://www.lavishsoft.com/wiki/index.php/LavishScript:Events) for the canonical reference.

### 3.10 Other Core Types

| Type | Description |
|---|---|
| [`point3f`](https://www.lavishsoft.com/wiki/index.php/ObjectType:point3f) | 3D float vector (X, Y, Z). Used for spatial coordinates. |
| [`rgb`](https://www.lavishsoft.com/wiki/index.php/ObjectType:rgb) | 3-component color (R, G, B). |
| [`script`](https://www.lavishsoft.com/wiki/index.php/ObjectType:script) | A running script. See [TLO:Script](https://www.lavishsoft.com/wiki/index.php/TLO:Script). Members: `Filename`, `RunningTime`, `CurrentDirectory`, `Paused`, `Profiling`, `AllowDebug`, `Variable[name]`, `VariableScope`, `ExecuteAtom[name,...]`. Methods: `End`, `QueueCommand[cmd]`, `Squelch`, `Unsquelch`, `Pause`, `Resume`, `ExecuteAtom[...]`, `EnableProfiling`, `DisableProfiling`, `DisableDebugging`, `DumpStack`, `DumpProfiling`, `EnableDebugLogging[file]`, `DisableDebugLogging`. |
| [`binary`](https://www.lavishsoft.com/wiki/index.php/ObjectType:binary) | Binary blob, typically a webrequest response or file content. |
| [`enumtype`](https://www.lavishsoft.com/wiki/index.php/ObjectType:enumtype) | A registered enumeration type. See [Enum TLO](https://www.lavishsoft.com/wiki/index.php/TLO:Enum) and `LavishScript:RegisterEnum[...]`. |

### 3.11 File Handling

[`filepath`](https://www.lavishsoft.com/wiki/index.php/ObjectType:filepath) -- filesystem path. Inherits from `unistring`. Reduces to `Path`.

| Member | Returns | Description |
|---|---|---|
| `Path` | string | Path text. |
| `AbsolutePath` | string | Resolved absolute path. |
| `PathExists` | bool | TRUE if path exists. |
| `FileExists[subpath]` | bool | TRUE if a file/dir at `subpath` (relative or absolute) exists. |

Methods: `MakeSubdirectory[name]`.

[`file`](https://www.lavishsoft.com/wiki/index.php/ObjectType:file) -- an open file handle.

[`filelist`](https://www.lavishsoft.com/wiki/index.php/ObjectType:filelist) -- enumerated set of files / directories.

| Member | Returns | Description |
|---|---|---|
| `Files` | int | Total file count. |
| `File[#]` | filelistentry | The #th entry. |

Methods: `GetFiles[pattern]`, `GetDirectories[pattern]`, `Reset` (clear the list; note `GetFiles`/`GetDirectories` do NOT auto-clear).

[`filelistentry`](https://www.lavishsoft.com/wiki/index.php/ObjectType:filelistentry) -- one entry. Members include `Filename`, `FullPath`. Wiki page provides full surface.

### 3.12 Utilities

[`lavishscript`](https://www.lavishsoft.com/wiki/index.php/ObjectType:lavishscript) -- the engine itself. Accessed via the [LavishScript TLO](https://www.lavishsoft.com/wiki/index.php/TLO:LavishScript).

| Member | Returns | Description |
|---|---|---|
| `Version` | string | Engine version. |
| `CurrentDirectory` | filepath | Engine's current directory. |
| `Executable` | filepath | Host process executable. |
| `HomeDirectory` | filepath | Engine's home directory. |
| `RunningTime` | int | Milliseconds since the host started (wraps after about 23 days). |
| `LSModule` | string | LavishScript version. |
| `VariableScope` | variablescope | The global variable scope. |
| `ExecuteAtom[name,...]` | string | Execute an atom and capture its return value. |
| `CreateQuery[expr]` | uint | Compile an [Object Query](https://www.lavishsoft.com/wiki/index.php/LavishScript:Object_Queries) expression. Returns a query ID. |
| `RetrieveQueryExpression[#]` | string | Retrieve the source expression for a query. |
| `QueryEvaluate[#,obj]` | bool | Test if `obj` matches a previously created query. |

Methods: `ExecuteAtom[name,...]`, `RegisterEvent[name]`, `Eval[command,index_var]` (run a command and capture output lines), `FreeQuery[#]`, `RegisterEnum[typeName,is_flags]`.

[`math`](https://www.lavishsoft.com/wiki/index.php/ObjectType:math) -- math library. Accessed via the [Math TLO](https://www.lavishsoft.com/wiki/index.php/TLO:Math).

| Member | Returns | Description |
|---|---|---|
| `Calc[formula]` | float64ptr | Evaluate a [formula](https://www.lavishsoft.com/wiki/index.php/LavishScript:Mathematical_Formulae). |
| `Calc64[formula]` | int64ptr | 64-bit integer-precision form. |
| `Abs[formula]` | float | Absolute value. |
| `Sin/Cos/Tan[formula]` | float | Trig (degrees). |
| `Asin/Acos/Atan[formula]` | float | Inverse trig (degrees). |
| `Atan[a,b]` | float | atan2 (degrees). |
| `Sqrt[formula]` | float | Square root. |
| `Distance[x1,x2]` / `[x1,y1,x2,y2]` / `[x1,y1,z1,x2,y2,z2]` | float | 1D / 2D / 3D distance. |
| `DistancePointLine[x,y,ax,ay,bx,by]` | float | Distance from point to line segment. |
| `Hex[#]` | string | Hex of a number. |
| `Dec[hex]` | int | Decimal of a hex string. |
| `Rand[formula]` | int | Random number 0 to (formula-1). Max value 32767. |
| `Not[#]` | int | Bitwise NOT. |

Methods: none.

[`system`](https://www.lavishsoft.com/wiki/index.php/ObjectType:system) -- host machine info. Accessed via the [System TLO](https://www.lavishsoft.com/wiki/index.php/TLO:System).

| Member | Returns | Description |
|---|---|---|
| `OS` | string | OS name. |
| `OSBuild` | string | OS build. |
| `MemoryUsage` | uint | This process's memory use (bytes). |
| `MemFree` | uint | Free physical RAM (MB). |
| `MemTotal` | uint | Total physical RAM (MB). |
| `TickCount` | uint | Millisecond timestamp (wraps every ~46 days). |
| `CurrentDirectory` | filepath | OS-level current directory. |
| `RegistryValue[hklm/hkcu,key,value]` | string | Read a REG_DWORD / REG_SZ / REG_EXPAND_SZ. |
| `GetProcAddress[module,function]` | uint | Address of an exported function (advanced). |

Methods: none documented on the wiki page. (Older docs reference `APICall` which is deprecated and 32-bit-only.)

[`type`](https://www.lavishsoft.com/wiki/index.php/ObjectType:type) -- represents an object type. Accessed via `${SomeObject(type)}` or the [Type TLO](https://www.lavishsoft.com/wiki/index.php/TLO:Type).

[`variable`](https://www.lavishsoft.com/wiki/index.php/ObjectType:variable) -- a runtime variable handle. Accessed via the [Variable TLO](https://www.lavishsoft.com/wiki/index.php/TLO:Variable).

[`exists`](https://www.lavishsoft.com/wiki/index.php/ObjectType:exists) -- the special `(exists)` cast that yields `bool` based on object validity.

### 3.13 Tasks and LavishMachine

The Task system (built into LavishScript as of Inner Space build 6374) lets scripts run instant or time-spanning Tasks managed by Task Managers. Tasks may be JSON-defined or script-defined. See [LavishScript:Tasks](https://www.lavishsoft.com/wiki/index.php/LavishScript:Tasks) and [14_LavishMachine_Guide.md](14_LavishMachine_Guide.md) for full coverage.

[`lavishmachine`](https://www.lavishsoft.com/wiki/index.php/ObjectType:lavishmachine) -- root of the Task system. Accessed via the [LMAC TLO](https://www.lavishsoft.com/wiki/index.php/Special:Search?search=TLO:LMAC&go=Go).

| Member | Returns | Description |
|---|---|---|
| `NewTaskManager[name]` | taskmanager | Create or retrieve a Task Manager. |
| `NewTaskTypeSet[name]` | tasktypeset | Create a new Task Type Set. |
| `NewTaskLibrary[name]` | tasklibrary | Create or retrieve a Task Library. |
| `NewTaskType[json]` | tasktype | Create a Task Type from JSON. |
| `TaskManager[#\|name]` | taskmanager | Retrieve. |
| `TaskTypeSet[#\|name]` | tasktypeset | Retrieve. |
| `TaskType[#\|name]` | tasktype | Retrieve. |
| `TaskLibrary[#\|name]` | tasklibrary | Retrieve. |
| `Task[#\|name]` | task | Retrieve a running Task. |

Methods: `LoadTaskTypesFile[file]`, `LoadTaskTypesJSON[json]`, `LoadPackageFile[file]`, `LoadPackageJSON[json]`.

[`taskmanager`](https://www.lavishsoft.com/wiki/index.php/ObjectType:taskmanager) -- runs Tasks.

| Member | Returns | Description |
|---|---|---|
| `Name` | string | Manager name. |
| `ID` | uint | Manager ID. |
| `BeginTask[json]` | task | Start a Task from a JSON object. |
| `BeginTaskLibrary[name]` | task | Start every Task in a Library, wrapped as a parallel Task. |

Methods: `BeginTask[json]`, `BeginTaskLibrary[name]`, `BeginTasks[jsonarray]`, `Clear` (stop all running Tasks), `Destroy`.

[`task`](https://www.lavishsoft.com/wiki/index.php/ObjectType:task) -- a running Task instance.
[`tasktype`](https://www.lavishsoft.com/wiki/index.php/ObjectType:tasktype) -- a Task Type definition.
[`tasklibrary`](https://www.lavishsoft.com/wiki/index.php/ObjectType:tasklibrary) -- a named library of Task Types.
[`taskpulseargs`](https://www.lavishsoft.com/wiki/index.php/Special:Search?search=ObjectType:taskpulseargs&go=Go) -- arguments passed to a Task's pulse callback. (Listed on [LavishScript:Tasks](https://www.lavishsoft.com/wiki/index.php/LavishScript:Tasks) but no dedicated `ObjectType:` page found; surface documented inline only. Pending wiki verification.)
[`elmactaskstate`](https://www.lavishsoft.com/wiki/index.php/Special:Search?search=ObjectType:elmactaskstate&go=Go) -- Task state enum. (Same status as `taskpulseargs` -- referenced from the Tasks page but no dedicated wiki entry. Pending verification.)

### 3.14 XML

[`xmlnode`](https://www.lavishsoft.com/wiki/index.php/Special:Search?search=ObjectType:xmlnode&go=Go) -- XML DOM node. (Listed in [Template:LavishScript:Object Types](https://www.lavishsoft.com/wiki/index.php/Template:LavishScript:Object_Types) but the dedicated page was not located; surface is mostly used through [LavishSettings](https://www.lavishsoft.com/wiki/index.php/LavishSettings). Pending dedicated wiki-page verification.)

[`xmlreader`](https://www.lavishsoft.com/wiki/index.php/Special:Search?search=ObjectType:xmlreader&go=Go) -- streaming XML reader. (Same status. Pending verification.)

### 3.15 Inner Space Audio

Inner-Space-Kernel-side audio. See [ISKernel:Object Types](https://www.lavishsoft.com/wiki/index.php/ISKernel:Object_Types) and the [Audio TLO](https://www.lavishsoft.com/wiki/index.php/ISKernel:Audio_(Top-Level_Object)).

| Type | Description |
|---|---|
| [`audio`](https://www.lavishsoft.com/wiki/index.php/ISKernel:audio_(Object_Type)) | Root audio object. Accessed via `Audio` TLO. Provides `AddVoice[name]`, `AddStream[name,path]`, `RemoveVoice[name]`, `RemoveStream[name]`, `Voice[name]` member, `Stream[name]` member. |
| [`audiovoice`](https://www.lavishsoft.com/wiki/index.php/ISKernel:audiovoice_(Object_Type)) | A playback channel. Methods: `PlayStream[name]`, `EnqueueStream[name]`, `Stop`, `ClearQueue`, `SetVolume[v]` / `SetVolume[left,right]`. |
| [`audiostream`](https://www.lavishsoft.com/wiki/index.php/ISKernel:audiostream_(Object_Type)) | An audio file (MP3, WAV, OGG, etc.). |

### 3.16 Inner Space Display

| Type | Description |
|---|---|
| [`display`](https://www.lavishsoft.com/wiki/index.php/ISKernel:display_(Data_Type)) | Display configuration. Accessed via `Display` TLO. |
| [`monitor`](https://www.lavishsoft.com/wiki/index.php/ISKernel:monitor_(Data_Type)) | A physical monitor. |
| [`gdiwindow`](https://www.lavishsoft.com/wiki/index.php/ISKernel:gdiwindow_(Data_Type)) | GDI-side window object. |

### 3.17 Inner Space Distributed Scope

A shared key-value store visible across sessions, used heavily for multi-boxing state synchronization.

| Type | Description |
|---|---|
| [`distributedscope`](https://www.lavishsoft.com/wiki/index.php/Special:Search?search=ISKernel:distributedscope&go=Go) | The container. (Listed in [ISKernel:Object Types](https://www.lavishsoft.com/wiki/index.php/ISKernel:Object_Types); pending dedicated-page verification.) |
| [`distributedvalue`](https://www.lavishsoft.com/wiki/index.php/Special:Search?search=ISKernel:distributedvalue&go=Go) | One entry. (Same status. Pending verification.) |

### 3.18 Inner Space Input

| Type | Description |
|---|---|
| [`input`](https://www.lavishsoft.com/wiki/index.php/ISKernel:input_(Object_Type)) | Top-level input subsystem. Accessed via `Input` TLO. |
| [`keyboard`](https://www.lavishsoft.com/wiki/index.php/ISKernel:keyboard_(Object_Type)) | Keyboard state. Accessed via `Keyboard` TLO. |
| [`mouse`](https://www.lavishsoft.com/wiki/index.php/ISKernel:mouse_(Object_Type)) | Mouse state. Accessed via `Mouse` TLO. |
| [`bind`](https://www.lavishsoft.com/wiki/index.php/ISKernel:bind_(Object_Type)) | A registered Bind. |
| [`button`](https://www.lavishsoft.com/wiki/index.php/ISKernel:button_(Object_Type)) | A button (gamepad/joystick). |
| [`dpad`](https://www.lavishsoft.com/wiki/index.php/ISKernel:dpad_(Object_Type)) | A D-pad. |
| [`axis`](https://www.lavishsoft.com/wiki/index.php/ISKernel:axis_(Object_Type)) | An analog axis. |
| [`g15`](https://www.lavishsoft.com/wiki/index.php/ISKernel:g15_(Object_Type)) | Logitech G15-series keyboard hardware. |

### 3.19 Inner Space MIDI

See [ISKernel:MIDI](https://www.lavishsoft.com/wiki/index.php/Special:Search?search=ISKernel:MIDI&go=Go) and the [MIDI TLO](https://www.lavishsoft.com/wiki/index.php/ISKernel:MIDI_(Top-Level_Object)).

| Type | Description |
|---|---|
| [`midi`](https://www.lavishsoft.com/wiki/index.php/ISKernel:midi_(Object_Type)) | MIDI subsystem root. |
| [`midiindevice`](https://www.lavishsoft.com/wiki/index.php/ISKernel:midiindevice_(Object_Type)) | A MIDI input device. |
| [`midiinevent`](https://www.lavishsoft.com/wiki/index.php/ISKernel:midiinevent_(Object_Type)) | A MIDI input event. |
| [`midioutdevice`](https://www.lavishsoft.com/wiki/index.php/Special:Search?search=ISKernel:midioutdevice&go=Go) | A MIDI output device. (Listed in [ISKernel:Object Types](https://www.lavishsoft.com/wiki/index.php/ISKernel:Object_Types); pending dedicated-page verification.) |

### 3.20 Inner Space Misc

| Type | Description |
|---|---|
| [`agent`](https://www.lavishsoft.com/wiki/index.php/ISKernel:agent_(Object_Type)) | An Inner Space "Agent" (app loader). See [ISKernel:Agents](https://www.lavishsoft.com/wiki/index.php/ISKernel:Agents). |
| [`console`](https://www.lavishsoft.com/wiki/index.php/ISKernel:console_(Data_Type)) | The console. Accessed via `Console` TLO. |
| [`extension`](https://www.lavishsoft.com/wiki/index.php/ISKernel:extension_(Data_Type)) | A loaded Inner Space extension. Accessed via `Extension` TLO. |
| [`innerspace`](https://www.lavishsoft.com/wiki/index.php/ISKernel:innerspace_(Data_Type)) | Inner Space root. Accessed via `InnerSpace` TLO. |
| [`localization`](https://www.lavishsoft.com/wiki/index.php/ISKernel:localization_(Object_Type)) | Localization subsystem. Accessed via `Localization` TLO. |
| [`webrequest`](https://www.lavishsoft.com/wiki/index.php/ISKernel:webrequest_(Object_Type)) | An HTTP/HTTPS request. Members: `State`, `InterpretAs`, `URL`, `POST`, `Filename`, `Result` (jsonobject), `Binary`. Methods: `SetURL[url]`, `InterpretAs[string\|json\|binary]`, `InterpretAs[file,filename]`, `Begin`, `Abort`, `Reset`, `FromJSON[json]`. |

### 3.21 Inner Space Navigation (deprecated direct types)

These types are wiki-flagged as deprecated. Use [LavishNav](#323-lavishnav) instead.

| Type | Description |
|---|---|
| [`navigation`](https://www.lavishsoft.com/wiki/index.php/ISKernel:navigation_(Data_Type)) | Deprecated. |
| [`navpath`](https://www.lavishsoft.com/wiki/index.php/ISKernel:navpath_(Data_Type)) | Deprecated. |
| [`navpoint`](https://www.lavishsoft.com/wiki/index.php/ISKernel:navpoint_(Data_Type)) | Deprecated. |
| [`navworld`](https://www.lavishsoft.com/wiki/index.php/ISKernel:navworld_(Data_Type)) | Deprecated. |

### 3.22 LavishSettings

XML-backed hierarchical key-value config system. See [LavishSettings](https://www.lavishsoft.com/wiki/index.php/LavishSettings) and [LavishSettings:Object Types](https://www.lavishsoft.com/wiki/index.php/LavishSettings:Object_Types).

| Type | Description |
|---|---|
| [`lavishsettings`](https://www.lavishsoft.com/wiki/index.php/LavishSettings:lavishsettings_(Object_Type)) | Root of the Settings system. |
| [`settingset`](https://www.lavishsoft.com/wiki/index.php/LavishSettings:settingset_(Object_Type)) | A named hierarchy node containing settings and child sets. |
| [`setting`](https://www.lavishsoft.com/wiki/index.php/LavishSettings:setting_(Object_Type)) | A named scalar setting. |
| [`settingnode`](https://www.lavishsoft.com/wiki/index.php/LavishSettings:settingnode_(Object_Type)) | Generic node base type. |
| [`settingcomment`](https://www.lavishsoft.com/wiki/index.php/LavishSettings:settingcomment_(Object_Type)) | An XML comment node. |
| [`settingsetref`](https://www.lavishsoft.com/wiki/index.php/LavishSettings:settingsetref_(Object_Type)) | A reference to a settingset. |
| [`settingattribute`](https://www.lavishsoft.com/wiki/index.php/LavishSettings:settingattribute_(Object_Type)) | An XML attribute on a setting. |
| [`dataset`](https://www.lavishsoft.com/wiki/index.php/Special:Search?search=ISKernel:dataset&go=Go) | A LavishSettings dataset. (Listed in [ISKernel:Object Types](https://www.lavishsoft.com/wiki/index.php/ISKernel:Object_Types) under Settings; pending dedicated-page verification.) |

### 3.23 LavishNav

3D navigation system. See [LavishNav](https://www.lavishsoft.com/wiki/index.php/LavishNav) and [LavishNav:Object Types](https://www.lavishsoft.com/wiki/index.php/LavishNav:Object_Types).

| Type | Description |
|---|---|
| [`lnavregion`](https://www.lavishsoft.com/wiki/index.php/LavishNav:lnavregion_(Object_Type)) | A navigation region (point, radius, sphere, prism, etc.). |
| [`lnavregionref`](https://www.lavishsoft.com/wiki/index.php/LavishNav:lnavregionref_(Object_Type)) | Reference to a region. |
| [`lnavregiongroup`](https://www.lavishsoft.com/wiki/index.php/LavishNav:lnavregiongroup_(Object_Type)) | A group of regions. |
| [`lnavconnectionref`](https://www.lavishsoft.com/wiki/index.php/LavishNav:lnavconnectionref_(Object_Type)) | Reference to a connection between regions. |
| [`lnavpathfinder`](https://www.lavishsoft.com/wiki/index.php/LavishNav:lnavpathfinder_(Object_Type)) | A pathfinder (A*, Dijkstra). |

### 3.24 LavishGUI 1 (pointer)

LavishGUI 1 is the legacy XML-based UI system. It is feature-frozen and superseded by LavishGUI 2. The full type table (lguitree, lguitreenode, plus the LGUI1 element types) is not duplicated here. See the LavishGUI 1 to LavishGUI 2 migration guide ([11_LavishGUI1_to_LavishGUI2_Migration.md](11_LavishGUI1_to_LavishGUI2_Migration.md)) for the LGUI 1 surface and the migration path. The canonical wiki entry is [LavishGUI](https://www.lavishsoft.com/wiki/index.php/LavishGUI).

### 3.25 LavishGUI 2 (pointer)

LavishGUI 2 is the modern JSON-based UI system. The type family is large (approximately 50 types: `lgui2element`, `lgui2layer`, `lgui2window`, `lgui2button`, `lgui2textbox`, `lgui2listbox`, `lgui2itemview`, `lgui2skin`, etc.). The full table is not duplicated here. See the dedicated LavishGUI 2 guide ([10_LavishGUI2_UI_Guide.md](10_LavishGUI2_UI_Guide.md)) for full surface coverage. The category index of LGUI 2 LS1 types lives at [Category:Object Types](https://www.lavishsoft.com/wiki/index.php/Category:Object_Types) (search for the `lgui2` prefix).

---

## 4. Top-Level Objects

A Top-Level Object (TLO) is the entry point of a data sequence. `${Math.Calc[1+1]}` reads the `Math` TLO, calls its `Calc` member, and emplaces the result. The form is `${TLO[indexargs].Member.Member.Method:...}`. See [LavishScript:Top-Level Objects](https://www.lavishsoft.com/wiki/index.php/LavishScript:Top-Level_Objects).

To enumerate every TLO registered in the current session (including those added by extensions), run [`TopLevelObject`](https://www.lavishsoft.com/wiki/index.php/Command:TopLevelObject) at the console with no argument.

### 4.1 LavishScript-Core TLOs

Source: [Template:LavishScript:Top-Level Objects](https://www.lavishsoft.com/wiki/index.php/Template:LavishScript:Top-Level_Objects). 22 entries.

#### Data-Storage Conversion

| TLO | Returns | Purpose |
|---|---|---|
| [`Bool`](https://www.lavishsoft.com/wiki/index.php/TLO:Bool) | bool | `${Bool[expr]}` evaluates `expr` as a boolean. |
| [`Float`](https://www.lavishsoft.com/wiki/index.php/TLO:Float) | float | Convert a value to float. |
| [`Int`](https://www.lavishsoft.com/wiki/index.php/TLO:Int) | int | Convert a value to int. |
| [`String`](https://www.lavishsoft.com/wiki/index.php/TLO:String) | string | Wrap a value as a string for further `.string` member access. |

#### Enums

| TLO | Returns | Purpose |
|---|---|---|
| [`Enum`](https://www.lavishsoft.com/wiki/index.php/TLO:Enum) | enumtype | Access a registered enum (registered via `LavishScript:RegisterEnum`). |

#### Date and Time

| TLO | Returns | Purpose |
|---|---|---|
| [`Time`](https://www.lavishsoft.com/wiki/index.php/TLO:Time) | time | Current date and time. |

#### Events

| TLO | Returns | Purpose |
|---|---|---|
| [`Event`](https://www.lavishsoft.com/wiki/index.php/TLO:Event) | event | Access a registered event by name: `${Event[Alias Added]}`. |

#### Inline Branching

| TLO | Returns | Purpose |
|---|---|---|
| [`If`](https://www.lavishsoft.com/wiki/index.php/TLO:If) | varies | Inline conditional: `${If[condition,trueValue,falseValue]}`. |

#### Math

| TLO | Returns | Purpose |
|---|---|---|
| [`Math`](https://www.lavishsoft.com/wiki/index.php/TLO:Math) | math | The math object. See [`math` type](#312-utilities). |

#### Misc

| TLO | Returns | Purpose |
|---|---|---|
| [`Arg`](https://www.lavishsoft.com/wiki/index.php/TLO:Arg) | varies | `${Arg[#]}` reads the #th command-line argument passed to the script. |
| [`Execute`](https://www.lavishsoft.com/wiki/index.php/TLO:Execute) | varies | `${Execute[command]}` runs a command and emplaces the result. |
| [`LavishScript`](https://www.lavishsoft.com/wiki/index.php/TLO:LavishScript) | lavishscript | The engine object. |
| [`Script`](https://www.lavishsoft.com/wiki/index.php/TLO:Script) | script | `${Script}` is the current script; `${Script[name]}` retrieves another by name. |
| [`Select`](https://www.lavishsoft.com/wiki/index.php/TLO:Select) | varies | Multi-way switch in a data sequence. |
| [`Type`](https://www.lavishsoft.com/wiki/index.php/TLO:Type) | type | `${Type[typename]}` resolves a registered type. |

#### Operating System

| TLO | Returns | Purpose |
|---|---|---|
| [`System`](https://www.lavishsoft.com/wiki/index.php/TLO:System) | system | Host machine info. |

#### Scripting

| TLO | Returns | Purpose |
|---|---|---|
| [`QueuedCommands`](https://www.lavishsoft.com/wiki/index.php/TLO:QueuedCommands) | varies | The current command queue. |
| [`Return`](https://www.lavishsoft.com/wiki/index.php/TLO:Return) | varies | The most recent function `return` value. Lifespan: until the next `call`. |
| [`Variable`](https://www.lavishsoft.com/wiki/index.php/TLO:Variable) | variable | Access a variable by name dynamically. |
| [`This`](https://www.lavishsoft.com/wiki/index.php/TLO:This) | varies | Inside an object method/member, the current instance. |
| [`VariableScope`](https://www.lavishsoft.com/wiki/index.php/TLO:VariableScope) | variablescope | Access the global variable scope. |
| [`ForEach`](https://www.lavishsoft.com/wiki/index.php/TLO:ForEach) | varies | Inside a `ForEach[code]` block on a container, exposes `Key` and `Value` of the current iteration. |
| [`Context`](https://www.lavishsoft.com/wiki/index.php/Special:Search?search=TLO:Context&go=Go) | varies | Provides per-frame / per-event context (used by Tasks, LGUI 2 event handlers, etc.). Wiki dedicated page not located via direct fetch; pending verification. |

### 4.2 Inner Space Kernel TLOs

Source: [ISKernel:Top-Level Objects](https://www.lavishsoft.com/wiki/index.php/ISKernel:Top-Level_Objects).

#### Audio

| TLO | Returns | Purpose |
|---|---|---|
| [`Audio`](https://www.lavishsoft.com/wiki/index.php/ISKernel:Audio_(Top-Level_Object)) | audio | Audio system root. |

#### Display

| TLO | Returns | Purpose |
|---|---|---|
| [`Display`](https://www.lavishsoft.com/wiki/index.php/ISKernel:Display_(Top-Level_Object)) | display | Display configuration. |

#### Input

| TLO | Returns | Purpose |
|---|---|---|
| [`Input`](https://www.lavishsoft.com/wiki/index.php/ISKernel:Input_(Top-Level_Object)) | input | Input subsystem root. |
| [`Keyboard`](https://www.lavishsoft.com/wiki/index.php/ISKernel:Keyboard_(Top-Level_Object)) | keyboard | Keyboard state. |
| [`Mouse`](https://www.lavishsoft.com/wiki/index.php/ISKernel:Mouse_(Top-Level_Object)) | mouse | Mouse state. |

#### Misc

| TLO | Returns | Purpose |
|---|---|---|
| [`InnerSpace`](https://www.lavishsoft.com/wiki/index.php/ISKernel:InnerSpace_(Top-Level_Object)) | innerspace | Inner Space root. |
| [`Console`](https://www.lavishsoft.com/wiki/index.php/ISKernel:Console_(Top-Level_Object)) | console | The console. |
| [`Extension`](https://www.lavishsoft.com/wiki/index.php/ISKernel:Extension_(Top-Level_Object)) | extension | A loaded Inner Space extension. |
| [`Localization`](https://www.lavishsoft.com/wiki/index.php/ISKernel:Localization_(Top-Level_Object)) | localization | Localization subsystem. |
| [`MIDI`](https://www.lavishsoft.com/wiki/index.php/ISKernel:MIDI_(Top-Level_Object)) | midi | MIDI subsystem. |

#### Settings

| TLO | Returns | Purpose |
|---|---|---|
| [`Game`](https://www.lavishsoft.com/wiki/index.php/ISKernel:Game_(Top-Level_Object)) | varies | Configured game profile. |
| [`Profile`](https://www.lavishsoft.com/wiki/index.php/ISKernel:Profile_(Top-Level_Object)) | varies | Launch profile. |

#### Navigation (deprecated direct TLOs)

| TLO | Returns | Purpose |
|---|---|---|
| [`Navigation`](https://www.lavishsoft.com/wiki/index.php/ISKernel:Navigation_(Top-Level_Object)) | navigation | Deprecated. Use [LavishNav](#323-lavishnav). |
| [`NavPath`](https://www.lavishsoft.com/wiki/index.php/ISKernel:NavPath_(Top-Level_Object)) | navpath | Deprecated. |

### 4.3 Inner Space Session TLOs

| TLO | Returns | Purpose |
|---|---|---|
| [`Session`](https://www.lavishsoft.com/wiki/index.php/ISSession:Session_(Top-Level_Object)) | varies | This session. |
| [`Sessions`](https://www.lavishsoft.com/wiki/index.php/ISSession:Sessions_(Top-Level_Object)) | varies | All sessions (used to address `Relay` targets). |

### 4.4 Subsystem TLOs (LGUI, LGUI2, LMAC)

| TLO | Returns | Purpose |
|---|---|---|
| [`LGUI`](https://www.lavishsoft.com/wiki/index.php/LavishGUI#Top-Level_Objects) | lgui | LavishGUI 1 root. See [LavishGUI](https://www.lavishsoft.com/wiki/index.php/LavishGUI) and the migration guide [11_LavishGUI1_to_LavishGUI2_Migration.md](11_LavishGUI1_to_LavishGUI2_Migration.md). |
| [`LGUI2`](https://www.lavishsoft.com/wiki/index.php/Special:Search?search=TLO:LGUI2&go=Go) | lgui2 | LavishGUI 2 root. See [10_LavishGUI2_UI_Guide.md](10_LavishGUI2_UI_Guide.md). Wiki dedicated TLO page not located via direct fetch; pending verification. |
| [`LMAC`](https://www.lavishsoft.com/wiki/index.php/Special:Search?search=TLO:LMAC&go=Go) | lavishmachine | LavishMachine / Tasks system root. Documented inline on [LavishScript:Tasks](https://www.lavishsoft.com/wiki/index.php/LavishScript:Tasks); dedicated TLO page not located. Pending verification. |

LavishSettings exposes its root via the [LavishSettings:Top-Level Objects](https://www.lavishsoft.com/wiki/index.php/LavishSettings:Top-Level_Objects) page (a `LavishSettings` TLO, plus a `Settings` TLO in some contexts).

LavishNav exposes its TLOs via [LavishNav:Top-Level Objects](https://www.lavishsoft.com/wiki/index.php/LavishNav:Top-Level_Objects).

(Per the Scope Boundary rule, this file does NOT cover extension-provided TLOs. Game-specific extensions register their own TLOs which belong in extension-specific guide files.)

---

## 5. Events

LavishScript events provide a way to hook functionality. Each event has a name and any number of attached **atoms** (or C functions, in extensions). When an event executes, every attached target runs in arbitrary order, receiving the event's parameters. See [LavishScript:Events](https://www.lavishsoft.com/wiki/index.php/LavishScript:Events).

### 5.1 Lifecycle

#### Register

```
LavishScript:RegisterEvent[<name>]
```

Registering an event twice is a no-op. The recommended pattern is to register once at script start. There is also a variable-declaration form that auto-registers on declaration and unregisters on scope exit:

```
declare MyEvent event "My Event"
```

#### Attach

Atoms attach via the [`event` object type's](https://www.lavishsoft.com/wiki/index.php/ObjectType:event) `AttachAtom` method, accessed through the [Event TLO](https://www.lavishsoft.com/wiki/index.php/TLO:Event):

```
Event[<name>]:AttachAtom[<atomName>]
```

There is no limit to the number of atoms attached to an event. Atoms execute in arbitrary order; do not rely on attach-order.

#### Execute

```
Event[<name>]:Execute[<param1>,<param2>,...]
Event[<name>]:ThisExecute[<contextObject>,<param1>,...]
```

`ThisExecute` makes the first argument available as `This` inside attached atoms.

#### Detach

```
Event[<name>]:DetachAtom[<atomName>]
```

#### Unregister

```
Event[<name>]:Unregister
```

Unregistering an event is rarely needed. Doing so forcefully detaches every attached target without notifying them. Prefer `DetachAtom` for individual cleanup.

### 5.2 Built-in Events

Events the engine fires automatically. These are always available; no `RegisterEvent` call is needed in your script.

| Event | When it fires | Parameters |
|---|---|---|
| `Alias Added` | An alias is created via [`Alias`](https://www.lavishsoft.com/wiki/index.php/Command:Alias). | `<alias name>` |

Additional engine and Inner Space events are listed in [Category:LavishScript Events](https://www.lavishsoft.com/wiki/index.php/Category:LavishScript_Events). Inner Space subsystems (input, audio, display, sessions) and the LavishMachine Tasks system fire their own events; consult the appropriate wiki section.

---

## 6. Triggers

A **trigger** fires a command when a registered text pattern is seen in console output. Used for chat-driven automation, log monitoring, and event-driven scripting from text streams. See [LavishScript:Triggers](https://www.lavishsoft.com/wiki/index.php/LavishScript:Triggers).

| Command | Purpose |
|---|---|
| [`AddTrigger`](https://www.lavishsoft.com/wiki/index.php/Command:AddTrigger) | Register a text-pattern trigger that fires a command when matched. |
| [`RemoveTrigger`](https://www.lavishsoft.com/wiki/index.php/Command:RemoveTrigger) | Unregister a previously added trigger. |
| [`WaitFor`](https://www.lavishsoft.com/wiki/index.php/Command:WaitFor) | Pause the calling script until a text pattern is seen, or a timeout elapses. |

Triggers are global to the session. A trigger persists until removed or until the session ends.

---

## 7. Modules (LSModule)

Optional Lavish Software modules add additional features when loaded via the [`Module`](https://www.lavishsoft.com/wiki/index.php/Command:Module) command. Each is an independent package; load only what you need. See [Category:LavishScript Modules](https://www.lavishsoft.com/wiki/index.php/Category:LavishScript_Modules).

| Module | Purpose |
|---|---|
| [LSModule:Sound](https://www.lavishsoft.com/wiki/index.php/LSModule:Sound) | Sound effect playback. Separate from the Inner Space audio object type. |
| [LSModule:MySQL](https://www.lavishsoft.com/wiki/index.php/LSModule:MySQL) | MySQL client. Adds [`mysql`](https://www.lavishsoft.com/wiki/index.php/LSModule:MySQL:mysql_(Object_Type)) and [`mysqlresult`](https://www.lavishsoft.com/wiki/index.php/LSModule:MySQL:mysqlresult_(Object_Type)) types. |
| [LSModule:Regex](https://www.lavishsoft.com/wiki/index.php/LSModule:Regex) | Regular-expression matching. |
| [LSModule:Targz](https://www.lavishsoft.com/wiki/index.php/LSModule:Targz) | `.tar.gz` archive support. Adds the [`tar`](https://www.lavishsoft.com/wiki/index.php/LSModule:Targz:_tar_(Command)) command. |
| [LSModule:Ventrilo](https://www.lavishsoft.com/wiki/index.php/LSModule:Ventrilo) | Ventrilo voice client control. Adds `ventrilo`, `ventrilochannel`, `ventriloinstance`, `ventriloserver`, `ventrilouser`, `ventriloelement` types. |
| [LSModule:Scheduler](https://www.lavishsoft.com/wiki/index.php/LSModule:Scheduler) | Job scheduler. |

To use a module:

```
Module -load <module-name>
```

Then its commands, types, and TLOs become available. Unload with `Module -unload <name>`.

---

## 8. .NET Integration

Inner Space supports running .NET 2.0/3.5 and 4.0 code from inside scripts. See [IS:.NET](https://www.lavishsoft.com/wiki/index.php/IS:.NET) for the canonical reference.

Platform-side commands:

| Command | Purpose |
|---|---|
| [`DotNet`](https://www.lavishsoft.com/wiki/index.php/ISKernel:DotNet_(Command)) | Run .NET code from a script. |

Platform-side guidance only -- game-extension .NET wrappers belong in their respective extension-specific guide files. The corresponding KB-tree guide for .NET-side development patterns is [19_DotNet_Development.md](19_DotNet_Development.md).

`DotScript` (mentioned in some inventories) does not have a dedicated `Command:DotScript` page on the wiki and is not in any canonical category listing. Pending wiki verification; treat as undocumented for now.

---

## 9. Wiki Source Map

The complete set of canonical wiki entry points used in this reference. Every link is `https:` and uses the canonical `index.php` form for stability.

### LavishScript Core

- [LavishScript main](https://www.lavishsoft.com/wiki/index.php/LavishScript)
- [LavishScript:Commands](https://www.lavishsoft.com/wiki/index.php/LavishScript:Commands)
- [LavishScript:Object Types](https://www.lavishsoft.com/wiki/index.php/LavishScript:Object_Types)
- [LavishScript:Top-Level Objects](https://www.lavishsoft.com/wiki/index.php/LavishScript:Top-Level_Objects)
- [LavishScript:Syntax](https://www.lavishsoft.com/wiki/index.php/LavishScript:Syntax)
- [LavishScript:Variables](https://www.lavishsoft.com/wiki/index.php/LavishScript:Variables)
- [LavishScript:Data Sequences](https://www.lavishsoft.com/wiki/index.php/LavishScript:Data_Sequences)
- [LavishScript:Mathematical Formulae](https://www.lavishsoft.com/wiki/index.php/LavishScript:Mathematical_Formulae)
- [LavishScript:Events](https://www.lavishsoft.com/wiki/index.php/LavishScript:Events)
- [LavishScript:Triggers](https://www.lavishsoft.com/wiki/index.php/LavishScript:Triggers)
- [LavishScript:Aliases](https://www.lavishsoft.com/wiki/index.php/LavishScript:Aliases)
- [LavishScript:Modules](https://www.lavishsoft.com/wiki/index.php/LavishScript:Modules)
- [LavishScript:Tasks](https://www.lavishsoft.com/wiki/index.php/LavishScript:Tasks)
- [LavishScript:Object Queries](https://www.lavishsoft.com/wiki/index.php/LavishScript:Object_Queries)
- [LavishScript:File System](https://www.lavishsoft.com/wiki/index.php/LavishScript:File_System)
- [LavishScript:Taking Actions](https://www.lavishsoft.com/wiki/index.php/LavishScript:Taking_Actions)

### Inner Space

- [Inner Space main](https://www.lavishsoft.com/wiki/index.php/Inner_Space)
- [IS:Kernel](https://www.lavishsoft.com/wiki/index.php/IS:Kernel)
- [IS:Session](https://www.lavishsoft.com/wiki/index.php/IS:Session)
- [IS:Uplink](https://www.lavishsoft.com/wiki/index.php/IS:Uplink)
- [IS:.NET](https://www.lavishsoft.com/wiki/index.php/IS:.NET)
- [ISKernel:Commands](https://www.lavishsoft.com/wiki/index.php/ISKernel:Commands)
- [ISKernel:Object Types](https://www.lavishsoft.com/wiki/index.php/ISKernel:Object_Types)
- [ISKernel:Top-Level Objects](https://www.lavishsoft.com/wiki/index.php/ISKernel:Top-Level_Objects)
- [ISKernel:Agents](https://www.lavishsoft.com/wiki/index.php/ISKernel:Agents)

### Subsystems

- [LavishGUI](https://www.lavishsoft.com/wiki/index.php/LavishGUI)
- [LavishGUI:Object Types](https://www.lavishsoft.com/wiki/index.php/LavishGUI:Object_Types)
- [LavishNav](https://www.lavishsoft.com/wiki/index.php/LavishNav)
- [LavishNav:Object Types](https://www.lavishsoft.com/wiki/index.php/LavishNav:Object_Types)
- [LavishNav:Top-Level Objects](https://www.lavishsoft.com/wiki/index.php/LavishNav:Top-Level_Objects)
- [LavishSettings](https://www.lavishsoft.com/wiki/index.php/LavishSettings)
- [LavishSettings:Object Types](https://www.lavishsoft.com/wiki/index.php/LavishSettings:Object_Types)
- [LavishSettings:Top-Level Objects](https://www.lavishsoft.com/wiki/index.php/LavishSettings:Top-Level_Objects)
- [ISXDK](https://www.lavishsoft.com/wiki/index.php/ISXDK)

### Category Indexes

- [Category:LavishScript](https://www.lavishsoft.com/wiki/index.php/Category:LavishScript)
- [Category:LavishScript Commands](https://www.lavishsoft.com/wiki/index.php/Category:LavishScript_Commands)
- [Category:LavishScript Top-Level Objects](https://www.lavishsoft.com/wiki/index.php/Category:LavishScript_Top-Level_Objects)
- [Category:LavishScript Events](https://www.lavishsoft.com/wiki/index.php/Category:LavishScript_Events)
- [Category:LavishScript Modules](https://www.lavishsoft.com/wiki/index.php/Category:LavishScript_Modules)
- [Category:Object Types](https://www.lavishsoft.com/wiki/index.php/Category:Object_Types)
- [Category:Inner Space](https://www.lavishsoft.com/wiki/index.php/Category:Inner_Space)
- [Category:Inner Space Kernel](https://www.lavishsoft.com/wiki/index.php/Category:Inner_Space_Kernel)
- [Category:Inner Space Session](https://www.lavishsoft.com/wiki/index.php/Category:Inner_Space_Session)
- [Category:Inner Space Uplink](https://www.lavishsoft.com/wiki/index.php/Category:Inner_Space_Uplink)

### Templates (where the actual feature lists live)

The wiki's `LavishScript:Object Types` and `LavishScript:Top-Level Objects` pages include their authoritative feature lists from sub-templates rather than inline. Fetch these directly to enumerate:

- [Template:LavishScript:Object Types](https://www.lavishsoft.com/wiki/index.php/Template:LavishScript:Object_Types)
- [Template:LavishScript:Top-Level Objects](https://www.lavishsoft.com/wiki/index.php/Template:LavishScript:Top-Level_Objects)

### Wiki-Verification Gaps Flagged in This File

For traceability, these are the entries that were referenced but where a dedicated wiki page could not be located via direct fetch as of 2026-04-27. They are marked inline as "pending verification". None block usage; they are flagged so future updates can fill in dedicated wiki links if/when those pages exist:

- `Diagnostics` command (section 2.2)
- `RelayTargets` (mentioned at end of section 2.4 -- functionally part of `Relay` addressing)
- `DotScript` command (section 2.5 / section 8)
- `jsoniterator` object type (section 3.7)
- `anonevent` object type (section 3.9)
- `taskpulseargs`, `elmactaskstate` object types (section 3.13)
- `xmlnode`, `xmlreader` object types (section 3.14)
- `distributedscope`, `distributedvalue` object types (section 3.17)
- `midioutdevice` object type (section 3.19)
- `dataset` object type (section 3.22)
- `Context` TLO (section 4.1)
- `LGUI2` and `LMAC` TLOs (section 4.4 -- documented inline on parent pages but no dedicated `TLO:` page found)
