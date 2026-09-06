# The IS Bridge

Every script gets a global table named **`IS`** -- your direct line into
InnerSpace and LavishScript. Most of the time you will use the object model
(`03_Object_Model.md`) instead, because it is cleaner and returns real Lua
values. But `IS.Execute` and `IS.Parse` are the escape hatches: anything the
object model cannot express, you can still do through the string API.

`IS` also holds the console output helpers and the event functions (events are
covered separately in `04_Timing_And_Events.md`).

## `IS.Execute(command)` -- run a command

Runs an InnerSpace / LavishScript command, exactly as if you had typed it at the
console. Returns the command's integer result.

```lua
IS.Execute("echo hello from a command")
IS.Execute("ext isxeq2")            -- load another extension
IS.Execute("target Fippy")          -- run a game command
```

Use this to trigger commands that have no object-model equivalent (loading
extensions, running other scripts, invoking console commands).

## `IS.Parse(dataSequence)` -- evaluate a `${...}` expression

Evaluates a LavishScript data sequence and returns the result as a string, or
`nil` if it could not be evaluated. **Pass the full `${...}` form.**

```lua
local name = IS.Parse("${Me.Name}")
echo("You are " .. tostring(name))

local n = tonumber(IS.Parse("${Me.Level}"))
```

`IS.Parse` always returns a **string** (or nil), so convert it yourself with
`tonumber` when you need a number. Compare this with the object model, where
`Me.Level` comes back as a real Lua number already.

> **8 KB limit.** `IS.Parse` returns at most 8 KB of text; a longer result comes
> back as `nil` rather than truncated. The object model does not have this limit
> (it reads values directly), so prefer it for large data.

**When to use `IS.Parse` instead of the object model:** the rare case of a member
that returns a scalar with no arguments but *also* takes arguments for a
different result. The object model gives you the native scalar and you cannot
then pass arguments to it; `IS.Parse("${...}")` lets you build the full
expression by hand. This is uncommon -- see `03_Object_Model.md` and
`06_Migration_Gotchas.md`.

## `print(...)` and `echo(...)` -- console output

Both write to the InnerSpace console. Arguments are converted with `tostring`
(honoring a value's `__tostring`, so object wrappers print sensibly) and
separated by tabs.

```lua
print("x", 1, true)          -- x    1    true
echo("Level is " .. Me.Level)
echo(Me)                     -- object wrappers coerce to text
```

They are interchangeable; `echo` exists because it reads naturally to
LavishScript scripters, and `print` is replaced so the standard Lua idiom still
produces visible output (raw Lua `print` goes to stdout, which you cannot see
in-game).

Your script text is never treated as a format string, so `echo(userText)` is safe
even if `userText` contains `%` characters.

## `IS.WarnUnknownGlobals(enabled)` -- control the unknown-global warning

When you read a global that is neither a Lua value nor a LavishScript top-level
object, ISXLUA returns `nil` and prints a one-time warning for that name (to help
you catch typos). Turn the warning off if you intentionally probe possibly-absent
globals:

```lua
IS.WarnUnknownGlobals(false)   -- silence the warning
```

See `03_Object_Model.md` for how bare-global lookup works.

## The event functions

`IS.AttachEvent`, `IS.DetachEvent`, and `IS.FireEvent` also live on the `IS`
table; they are documented in `04_Timing_And_Events.md`.

Next: `03_Object_Model.md`.
