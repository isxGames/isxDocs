# Getting Started

This chapter covers loading ISXLUA, writing your first script, running and
stopping scripts, passing arguments, and what the Lua environment gives you.

## Loading ISXLUA

From the InnerSpace console:

```
ext ISXLUA
```

You will see a load banner such as `ISXLUA v20260906.143107 (Lua 5.4.9)`. The
version carries a build-time suffix so you can tell same-day rebuilds apart;
`${ISXLUA.Version}` reports the same string.

ISXLUA attaches to no particular game -- it is a Lua runtime for InnerSpace. To
automate a game you also load that game's extension (for example `ext isxeq2`).

## Your first script

Create a file called `hello.lua` in your InnerSpace **Scripts** directory:

```lua
echo("Hello from Lua!")
```

Run it from the console:

```
lua hello
```

The `.lua` extension is optional -- `lua hello` and `lua hello.lua` are the same.

Note that you use `echo(...)` (or `print(...)`), **not** Lua's raw output. Lua's
built-in `print` writes to standard output, which is invisible in-game; ISXLUA
replaces `print` and adds `echo` so both go to the InnerSpace console. See
`02_The_IS_Bridge.md`.

## Running and stopping scripts

| Command | Effect |
|---|---|
| `lua <name> [args...]` | Run a `.lua` script. |
| `endlua <name>` | Stop one running script. |
| `endlua all` (or `endlua *`) | Stop every running Lua script. |
| `luas` | List the running Lua scripts and how long each has been running. |

**Name resolution** works like the native `run` command: the name is looked for
(1) as given (an absolute path, or relative to the current game directory), then
(2) in the InnerSpace **Scripts** directory, then (3) in the InnerSpace base
directory. The first file found wins.

**Names are matched loosely** -- without the directory, without the `.lua`
extension, case-insensitively. So `endlua Hello` stops the script started by
`lua hello.lua`.

**One instance per name.** Starting a script whose name is already running is
refused (just like `run`). Different scripts run **simultaneously**, each fully
isolated in its own Lua state -- one script's globals never leak into another's.

### The native `run` / `endscript` commands also work

ISXLUA registers a Lua script engine, so InnerSpace's built-in commands drive
`.lua` files too:

```
run hello.lua
endscript hello
```

A script started with `run` and one started with `lua` share the **same**
running-script set, so `endlua`, `endscript`, and `luas` all see both.

> **One limitation to know:** the native `scripts` / `scripts -running` listing
> does **not** show `.lua` scripts (its table is LavishScript-scoped). This is an
> InnerSpace-side constraint, not something ISXLUA can change. **Use `luas` to
> list running Lua scripts.**

## Script arguments

Anything after the script name is passed to your script two ways: as the chunk's
varargs (`...`) and as a global 1-indexed table named `args`.

```lua
-- args.lua   ->   lua args hello 42
echo("first arg:  " .. tostring(args[1]))   -- hello
echo("second arg: " .. tostring(args[2]))   -- 42

-- ... works too:
local a, b = ...
echo(a .. " / " .. b)
```

Because the `lua` command parses LavishScript first, a `${...}` in your arguments
is evaluated before the script sees it. For example `lua greet ${Me.Name}` passes
your character's name as `args[1]`.

## The `${ISXLUA}` object

ISXLUA exposes its own status through a top-level object, reachable from Lua as
the bare global `ISXLUA`:

```lua
echo("ISXLUA version: " .. ISXLUA.Version)
if ISXLUA.IsReady then echo("ISXLUA is ready") end
```

| Member | Type | Meaning |
|---|---|---|
| `ISXLUA.Version` | string | Version string (`<YYYYMMDD>.<HHMMSS>`), same as the banner. |
| `ISXLUA.IsReady` | bool | True once the extension has finished loading. |
| `ISXLUA.IsLoading` | bool | True while the extension is still loading. |
| `ISXLUA.InQuietMode` | bool | True when quiet mode is on. |

| Method | Effect |
|---|---|
| `ISXLUA:QuietMode()` | Toggles quiet mode on/off. |

## What the Lua environment gives you

- **Lua 5.4.9**, the full standard library (`string`, `table`, `math`, `os`,
  `io`, `coroutine`, `utf8`, `package`, ...). ISXLUA does not currently sandbox
  the standard library.
- **`echo` / `print`** -- console output (see `02_The_IS_Bridge.md`).
- **`wait(seconds)` / `waitframe()`** -- cooperative timing (see
  `04_Timing_And_Events.md`).
- **The `IS` bridge** -- `IS.Execute`, `IS.Parse`, the event functions
  (`02_The_IS_Bridge.md`, `04_Timing_And_Events.md`).
- **The object model** -- bare-global TLOs and generic member/method access
  (`03_Object_Model.md`).
- **Bundled libraries** -- `require("cjson")`, `require("serpent")`, and more
  (`05_Bundled_Libraries.md`).

Next: `02_The_IS_Bridge.md`.
