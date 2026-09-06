# ISXLUA Scripting Guide

A guide for people who **write Lua scripts** for InnerSpace using **ISXLUA**. It
covers the Lua-facing API only -- how to run scripts, how the `IS` bridge works,
how to reach LavishScript top-level objects and datatypes from Lua, how timing
and events behave, which libraries are bundled, and the gotchas you will hit if
you are coming from LavishScript. You do not need any source access or C/C++
knowledge to use this guide.

## What ISXLUA is

ISXLUA is an InnerSpace extension that embeds **Lua 5.4.9** directly. It lets you
automate InnerSpace and any loaded game extension (ISXEQ2, ISXEVE, ...) from Lua
instead of LavishScript. The whole Lua core and standard library are built in --
there is no separate Lua install, no DLL, nothing extra to ship. Load it from the
InnerSpace console with:

```
ext ISXLUA
```

When it loads you will see a banner like `ISXLUA v20260906.143107 (Lua 5.4.9)`,
and `${ISXLUA.Version}` will report the same version.

## The files in this guide

Read them in order the first time; after that use them as a reference.

| File | What it covers |
|---|---|
| **`README.md`** (this file) | Orientation and the file map. |
| `01_Getting_Started.md` | Running scripts (`lua` / `endlua` / `luas`, and native `run` / `endscript`), script arguments, the `${ISXLUA}` object, what the Lua environment gives you. |
| `02_The_IS_Bridge.md` | The `IS` global -- `IS.Execute`, `IS.Parse`, `print` / `echo` -- your two escape hatches into LavishScript. |
| `03_Object_Model.md` | The heart of ISXLUA: bare-global TLOs (`Me`, `Actor`, `EQ2`, ...), `.Member` / `.Member(args)`, `:Method(args)`, native scalar values, the typed getters, `Exists()`, and `obj[i]` indexing. |
| `04_Timing_And_Events.md` | `wait(seconds)` / `waitframe()`, and the events layer (`IS.AttachEvent` / `IS.DetachEvent` / `IS.FireEvent`) with atomic handlers. |
| `05_Bundled_Libraries.md` | The `require`-able modules that ship inside ISXLUA (`cjson`, `lfs`, `lpeg`, `serpent`, `inspect`, `json`, `re`) and how to load your own loose modules. |
| `06_Migration_Gotchas.md` | The differences that will bite a LavishScript scripter moving to Lua. **Read this if you know LavishScript.** |
| `07_Examples.md` | Complete, runnable scripts -- simple (hello world, reading your character, a wait loop, an event handler) and a larger realistic one that combines them. |

## The five-minute version

- Save a `.lua` file in your InnerSpace **Scripts** directory and run it with
  `lua myscript` (the `.lua` is optional). `endlua myscript` stops it; `luas`
  lists what is running.
- **Top-level objects are bare Lua globals.** `Me.Name`, `Me.Level`,
  `Actor(5).Distance`, `Target.Name` all just work.
- **Member arguments use parentheses, methods use a colon:**
  `Me.Ability("Fireball").Name`, `Actor(5):DoubleClick()`.
- **Scalar results come back as real Lua values** (numbers, strings, booleans),
  so `Me.Level == 95` and `Me.Name:upper()` work directly.
- **`wait()` takes SECONDS**, not tenths of a second like LavishScript.
- Coming from LavishScript? Read `06_Migration_Gotchas.md` before anything else.

## A note on scope

This guide documents the **Lua-facing behavior** of ISXLUA. The list of members
and methods available on `Me`, `Actor`, `EQ2`, etc. is not defined by ISXLUA --
it comes from whichever game extension you have loaded (ISXEQ2, ISXEVE, ...).
For those, consult that extension's own scripting guide or quick reference; every
member/method it documents is reachable from Lua exactly as described here.

The examples are kept in sync as ISXLUA features are added.
