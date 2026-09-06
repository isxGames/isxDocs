# Examples

Complete, runnable scripts. Save any of them into your InnerSpace **Scripts**
directory and run with `lua <name>`.

> These examples are kept in sync as ISXLUA features are added. If something here
> disagrees with the reference chapters, the reference chapters are current.

---

## Simple

### 1. Hello, world

```lua
-- hello.lua
echo("Hello from Lua!")
```

Run it: `lua hello`. Remember to use `echo` / `print`, not Lua's raw output --
see `02_The_IS_Bridge.md`.

### 2. Read your character (with the ISXEQ2 load-and-ready gate)

Before you touch `Me`, `Actor`, or any game data, make sure the game extension is
loaded **and** finished initializing. This is the idiomatic gate: load ISXEQ2 if
it is not present, then use `waituntil` -- first for the extension to appear, then
for it to report ready -- giving each step up to about 30 seconds. `waituntil`
replaces the hand-rolled poll loops this used to need.

```lua
-- whoami.lua
-- Ensure ISXEQ2 is loaded and ready, then print some character info.

local function ensureISXEQ2()
    -- We deliberately probe possibly-absent globals below; don't spam the
    -- one-time unknown-global warning while we do.
    IS.WarnUnknownGlobals(false)

    -- Load the extension if it isn't already present.
    if not Exists(Extension("ISXEQ2")) then
        echo("ISXEQ2 not loaded -- loading it now...")
        IS.Execute("ext isxeq2")
    end

    -- Wait (up to ~30s) for the extension to appear.
    if not waituntil(function() return Exists(Extension("ISXEQ2")) end, 30) then
        echo("ERROR: ISXEQ2 failed to load within 30 seconds.")
        return false
    end

    -- Wait (up to ~30s) for it to finish initializing. ISXEQ2 becomes a
    -- resolvable global only once it registers its TLO, so guard against
    -- indexing it before then.
    if not waituntil(function() local e = ISXEQ2; return e ~= nil and e.IsReady end, 30) then
        echo("ERROR: ISXEQ2 did not become ready within 30 seconds.")
        return false
    end

    return true
end

if not ensureISXEQ2() then
    return
end

echo("Name:  " .. Me.Name)
echo("Level: " .. Me.Level)
echo("Class: " .. Me.Class)

if Exists(Me.Pet) then
    echo("Pet:   " .. Me.Pet.Name)
else
    echo("Pet:   (none)")
end
```

Note `Me.Level` is used in arithmetic/concatenation directly -- scalars are native
Lua values. And `Me.Pet` is tested with `Exists()`, never a bare `if Me.Pet then`
(which is always true). See `03_Object_Model.md`.

### 3. A wait loop

```lua
-- healthwatch.lua
-- Print health every 2 seconds until it drops below 30%.

while true do
    local hp = Me.Health          -- a native number
    echo("Health: " .. hp .. "%")
    if hp < 30 then
        echo("Low health!")
        break
    end
    wait(2)
end
```

`wait(2)` is two seconds. (In LavishScript that would be `wait 20`.) Because we
re-read `Me.Health` at the top of each loop, we never hold a stale wrapper across
the wait.

### 4. A basic event handler

```lua
-- chatecho.lua
-- Echo every incoming chat line. Runs until you `endlua chatecho`.

IS.AttachEvent("EQ2_onIncomingChat", function(text)
    echo("[chat] " .. text)
end)

-- Keep the script alive so the handler stays attached.
while true do
    wait(1)
end
```

The handler auto-detaches when the script ends, so `endlua chatecho` cleans up for
you. Handlers run atomically -- never call `wait()` inside one (see
`04_Timing_And_Events.md`).

### 5. Remembering state between runs

`IS.SaveTable` / `IS.LoadTable` persist a table to disk, so a script can pick up
where it left off. This one counts how many times it has been run.

```lua
-- runcount.lua
-- Load a saved counter (or start fresh), bump it, save it back.

local state = IS.LoadTable("runcount") or { runs = 0 }

state.runs = state.runs + 1
IS.SaveTable("runcount", state)

echo("This script has now been run " .. state.runs .. " time(s).")
```

The first run prints `1` (there is no saved file yet, so `IS.LoadTable` returns
`nil` and the `or { runs = 0 }` default kicks in); each later run increments and
persists the count. The data lives in an `ISXLUAData` folder inside your Scripts
directory -- see `02_The_IS_Bridge.md`.

---

## Complex

### Chat session tracker (gate + event + wait + a bundled library)

A realistic script that combines everything: it gates ISXEQ2, attaches a chat
event handler that records activity into a shared table, and runs a main loop that
periodically snapshots your state and persists it to disk with the bundled
`serpent` library. The handler stays fast and atomic (it only records); the main
loop does all the waiting and file I/O.

```lua
-- sessiontracker.lua
-- Tracks incoming chat volume and periodically saves a session summary to disk.
-- Usage:  lua sessiontracker [saveIntervalSeconds]

local serpent = require("serpent")

--------------------------------------------------------------------------------
-- Ensure ISXEQ2 is loaded and ready (same gate as whoami.lua).
--------------------------------------------------------------------------------
local function ensureISXEQ2()
    IS.WarnUnknownGlobals(false)

    if not Exists(Extension("ISXEQ2")) then
        echo("ISXEQ2 not loaded -- loading it now...")
        IS.Execute("ext isxeq2")
    end

    if not waituntil(function() return Exists(Extension("ISXEQ2")) end, 30) then
        echo("ERROR: ISXEQ2 failed to load within 30 seconds.")
        return false
    end

    if not waituntil(function() local e = ISXEQ2; return e ~= nil and e.IsReady end, 30) then
        echo("ERROR: ISXEQ2 did not become ready within 30 seconds.")
        return false
    end

    return true
end

if not ensureISXEQ2() then
    return
end

--------------------------------------------------------------------------------
-- Session state. The event handler only *records* into this table; the main
-- loop reads it. Both run on the same thread, so no locking is needed.
--------------------------------------------------------------------------------
local session = {
    character  = Me.Name,          -- native string, safe to capture once
    startedAt  = os.time(),
    chatLines  = 0,
    lastChat   = "",
}

local saveInterval = tonumber(args[1]) or 30      -- seconds between saves
local saveFile     = "session_" .. session.character .. ".lua"

--------------------------------------------------------------------------------
-- Chat handler: fast and atomic -- just count and stash. No wait() in here.
--------------------------------------------------------------------------------
local function onChat(text)
    session.chatLines = session.chatLines + 1
    session.lastChat  = text
end

IS.AttachEvent("EQ2_onIncomingChat", onChat)

--------------------------------------------------------------------------------
-- Persist the current session to disk as loadable Lua source.
--------------------------------------------------------------------------------
local function save()
    local snapshot = {
        character = session.character,
        startedAt = session.startedAt,
        savedAt   = os.time(),
        chatLines = session.chatLines,
        lastChat  = session.lastChat,
        -- Re-fetch live values here rather than trusting anything captured
        -- across an earlier wait().
        level     = Me.Level,
        health    = Me.Health,
        zone      = tostring(Zone.Name),
    }

    local text = "return " .. serpent.block(snapshot, { comment = false })
    local f, err = io.open(saveFile, "w")
    if not f then
        echo("Could not write " .. saveFile .. ": " .. tostring(err))
        return
    end
    f:write(text)
    f:close()

    echo(string.format("[session] saved: %d chat lines, level %s, %s%% hp",
        snapshot.chatLines, tostring(snapshot.level), tostring(snapshot.health)))
end

--------------------------------------------------------------------------------
-- Main loop: do the timed work here, where waiting is allowed.
--------------------------------------------------------------------------------
echo(string.format("Session tracker started for %s (saving every %ds).",
    session.character, saveInterval))
echo("Stop it with: endlua sessiontracker")

while true do
    wait(saveInterval)
    save()
end
```

Why it is shaped this way:

- **The gate runs first.** Nothing reads `Me` until ISXEQ2 is ready.
- **The handler is trivial.** It records into `session` and returns immediately --
  no `wait()`, no file I/O -- because event handlers are atomic.
- **The main loop owns all timing and I/O.** `wait(saveInterval)` paces the saves,
  and every live value written in `save()` is re-fetched at that moment rather
  than reused across a wait.
- **`serpent`** turns the snapshot into loadable Lua text. To read a saved session
  back elsewhere: `local ok, data = serpent.load(io.open(saveFile):read("a"))`.

Swap `serpent` for `cjson` if you would rather persist JSON -- see
`05_Bundled_Libraries.md`.
