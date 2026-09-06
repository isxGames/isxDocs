# LavishScript Migration Gotchas

If you already write LavishScript, these are the differences that will bite you
when you move to Lua under ISXLUA. Read this chapter before porting a script.

## `wait()` takes SECONDS, not tenths of a second

LavishScript's `wait` counts tenths of a second; ISXLUA's `wait` counts seconds
(a float).

```
LavishScript:  wait 10        ; waits 1 second (tenths)
ISXLUA:        wait(1)         -- waits 1 second (seconds)
               wait(1.5)       -- waits 1.5 seconds
```

So an LS `wait 10` becomes `wait(1)` here. Use `waitframe()` to yield exactly one
frame.

## Member arguments use parentheses `()`, not brackets `[]`

```
LavishScript:  ${Me.Ability[Fireball].Name}
ISXLUA:        Me.Ability("Fireball").Name
```

Index / TLO arguments too: `${Actor[5]}` becomes `Actor(5)`. Methods use a colon:
`Actor(5):DoubleClick()`.

## An unknown bare global returns `nil` (a typo does not error at the global lookup)

A name that is neither a Lua value nor a LavishScript top-level object returns
`nil`, with a one-time console warning (disable it via
`IS.WarnUnknownGlobals(false)`). **But** a misspelled `.Member` or `:Method` on an
object that *did* resolve **does** raise a Lua error, with a traceback.

## Do not hold an object across a `wait()`

Object wrappers point at frame-scoped LavishScript data that can go stale after a
yield. Re-fetch from the TLO after a `wait()` (for example, read `Actor(id)`
again) rather than reusing a wrapper you captured before the wait. If you only
need a value, copy it out as a native scalar before waiting.

## A real game member/method always wins over a value-helper

The value-helpers are `:Int()`, `:Number()`, `:Bool()`, `:Str()`, `:LSType()`,
`:IsNull()`, and `:Exists()`. If a datatype has a member or method of the same
name, **you get the game one**. That is why the type-name helper is `:LSType()`
(not `:Type`) -- many datatypes have a real `.Type` member (for example
`Actor(5).Type` is `PC` / `NPC` / ...). The helpers answer only when the type has
no such member or method.

## Scalar values are NATIVE Lua values (numbers / strings / booleans)

A member or TLO that resolves to a scalar comes back as a real Lua value, so `==`,
all operators, and Lua's own string/number methods work directly:

```lua
Me.Level == 95
Me.Name == "Fippy"
Actor(5).Distance < 10
Me.Name:upper()
```

Only **object** results stay as wrappers (they chain, coerce via `tostring()` /
`..`, and offer `:Int()` / `:Number()` / `:Bool()` / `:Str()` / `:LSType()` /
`:IsNull()` / `:Exists()`).

**Rare edge:** a member that returns a scalar with no arguments but *also* takes
arguments for a different result comes back as the native scalar (so you cannot
then call it with arguments). For that case, use `IS.Parse("${...}")` to build the
full expression by hand.

## Testing whether an object EXISTS -- `if Me.Pet then` is ALWAYS true

An absent object (no pet, no target) comes back as a NULL object wrapper, and a
Lua userdata can never be falsy -- so a bare `if Me.Pet then` is always truthy.
Test presence with the `Exists()` global or the `:Exists()` method:

```lua
if Exists(Me.Pet) then ... end        -- true only if there is a real pet
if Me.Pet:Exists() then ... end        -- same
if Me.Pet:IsNull() then ... end        -- the negative
```

(This is a hard Lua rule; a null object cannot be turned into `nil` generically,
because an absent `Pet` is indistinguishable from an argument-needing member like
`Ability` at a zero-argument probe.)

## No `wait()` / `waitframe()` inside an event handler

An event handler (`IS.AttachEvent`) runs **atomically**, like a LavishScript atom
-- it must run to completion in one go. Calling `wait()` / `waitframe()` inside a
handler is an error (it is reported to the console; it does not crash). Do
time-based work in your main script loop, not in the handler. Handlers receive the
event's arguments as varargs: `function(a, b, ...)`.

## Quick reference: LavishScript to Lua

| LavishScript | ISXLUA (Lua) |
|---|---|
| `${Me.Name}` | `Me.Name` |
| `${Me.Ability[Fireball].Name}` | `Me.Ability("Fireball").Name` |
| `${Actor[5].Distance}` | `Actor(5).Distance` |
| `Actor[5]:DoubleClick` | `Actor(5):DoubleClick()` |
| `${Target(exists)}` | `Exists(Target)` |
| `wait 10` (1 second) | `wait(1)` |
| `wait 5` (0.5 second) | `wait(0.5)` |
| `echo Hello` | `echo("Hello")` |
| `${Math.Calc[${a}+${b}]}` | `a + b` (native Lua math) |

Next: `07_Examples.md`.
