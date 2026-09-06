# Timing and Events

Two things make a script do work over time: **cooperative waiting** (yielding the
script for a while and resuming later) and **events** (running a function when
something happens in the game).

## `wait(seconds)` and `waitframe()`

`wait(seconds)` suspends your script for that many **seconds** (a float), then
resumes it. `waitframe()` suspends until the next frame.

```lua
wait(1)       -- one second
wait(0.5)     -- half a second
wait(2.5)     -- two and a half seconds
waitframe()   -- resume next frame
```

> **Coming from LavishScript?** LavishScript's `wait` counts **tenths of a
> second**; ISXLUA's `wait` counts **seconds**. An LS `wait 10` (one second)
> becomes `wait(1)` here. This is the single most common porting mistake -- see
> `06_Migration_Gotchas.md`.

While a script is waiting, other scripts keep running and the game keeps ticking;
your script simply resumes when its time is up. A typical polling loop looks like:

```lua
local running = true
while running do
    -- do a little work
    echo("Health: " .. Me.Health)
    wait(1)
end
```

Because ISXLUA resumes waiting scripts each frame, a loop like this costs almost
nothing between iterations.

**Remember the object-lifetime rule:** do not keep an object wrapper across a
`wait()` / `waitframe()`. Re-fetch it from its TLO afterward, or copy out scalar
values (which are native and safe to keep) before you wait. See
`03_Object_Model.md`.

## `waituntil(condfn, timeoutSeconds)` -- wait for a condition

`waituntil` is the clean replacement for a hand-rolled polling loop. You give it a
**condition function** and (optionally) a **timeout in seconds**. It yields your
script and re-checks the condition every frame; it resumes the moment the condition
returns a truthy value, or when the timeout elapses -- whichever comes first.

It **returns `true` if the condition was met**, or **`false` if it timed out**, so
you can branch on the outcome.

```lua
-- Wait up to 30 seconds for some condition to become true.
if waituntil(function() return someReadyCheck() end, 30) then
    echo("ready")
else
    echo("timed out")
end
```

Compare that with the equivalent by hand -- `waituntil` replaces all of this:

```lua
local waited = 0
while not someReadyCheck() do
    if waited >= 30 then break end
    wait(0.5)
    waited = waited + 0.5
end
```

The condition function takes no arguments and returns a value that is checked for
truthiness (in Lua, everything except `false` and `nil` is truthy).

**Omit the timeout to wait indefinitely:**

```lua
waituntil(function() return done end)   -- no timeout: waits until done is truthy
```

The condition is re-evaluated once per frame, so keep it cheap -- read a value and
compare it, do not do heavy work inside it. If the condition function raises an
error, it is reported to the console (with a traceback) and `waituntil` returns
`false`.

Because `waituntil` yields, the same rule as `wait()` applies: **you cannot call it
inside an event handler** (handlers run atomically -- see below), and do not hold an
object wrapper across it.

### `wait(seconds, condfn)` -- a timed wait that can finish early

`wait` also accepts an optional condition function as a second argument. This waits
**up to** `seconds`, but resumes early the moment the condition becomes truthy. It
is the same behavior as `waituntil` with the arguments in the order a LavishScript
scripter tends to expect (time first). Like `waituntil`, it returns `true` if the
condition was met and `false` if the full time elapsed:

```lua
-- Give it at most 5 seconds, but continue as soon as ready() is true.
local met = wait(5, function() return ready() end)
```

Plain `wait(seconds)` with no condition returns nothing, exactly as before.

## Events

An event lets you run a Lua function when something happens -- a chat line
arrives, a zone changes, and so on. The set of available events comes from the
game extension you have loaded (for example ISXEQ2's `EQ2_onIncomingChat`); the
mechanism below is the same for all of them.

**A Lua function *is* the event target.** There is no separate "atom" concept to
declare as in LavishScript -- you just hand ISXLUA a function.

### `IS.AttachEvent(name, fn)`

Registers `fn` to be called whenever the named event fires. Your function
receives the event's arguments as **varargs**, each a string:

```lua
IS.AttachEvent("EQ2_onIncomingChat", function(text, ...)
    echo("chat: " .. text)
end)
```

- Multiple scripts (and multiple functions) may attach to the same event.
- `IS.AttachEvent` returns the function you passed, so you can keep a reference to
  it for later detaching.
- Handlers **auto-detach when their script ends** -- you do not have to clean up
  on exit.

### `IS.DetachEvent(name, fn)`

Removes a handler you previously attached (matched by identity). Returns `true` if
one was removed.

```lua
local function onChat(text) echo("chat: " .. text) end
IS.AttachEvent("EQ2_onIncomingChat", onChat)
-- ...later...
IS.DetachEvent("EQ2_onIncomingChat", onChat)
```

Keep the function in a variable if you intend to detach it -- you need the same
value you attached.

### `IS.FireEvent(name, ...)`

Fires an event yourself, passing extra arguments (converted to strings) to every
handler:

```lua
IS.FireEvent("MyScript_onSomething", "payload", 42)
```

This is handy for your own custom events, or for testing a handler.

## Handlers run atomically -- no `wait()` inside them

An event handler runs **atomically**, exactly like a LavishScript atom: it must
run to completion in a single step. **You cannot call `wait()` or `waitframe()`
inside a handler** -- doing so is reported to the console as an error (it does not
crash anything). If a handler needs to do timed work, have it set a flag or push
some data into a table, and let your main script loop (which *can* wait) act on it.

```lua
local pending = {}

IS.AttachEvent("EQ2_onIncomingChat", function(text)
    -- fast, no waiting -- just record it
    pending[#pending + 1] = text
end)

-- main loop handles the queued work, and here waiting is fine
while true do
    while #pending > 0 do
        local line = table.remove(pending, 1)
        echo("saw: " .. line)
    end
    wait(0.5)
end
```

Any error thrown inside a handler is printed to the console -- it will not take
down the game or your script.

## Limits

You can have up to **64 distinct events** attached at once (across all scripts).
Attaching handlers to the *same* event does not count against this -- only the
number of different event names does.

Next: `05_Bundled_Libraries.md`.
