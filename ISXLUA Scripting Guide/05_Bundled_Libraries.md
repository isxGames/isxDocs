# Bundled Libraries

ISXLUA ships several popular Lua libraries **built in**. There is nothing to
install and no files to place on disk -- just `require` them. Every bundled module
is require-able only; none is installed as a global, so they never shadow your own
variables or the top-level-object bridge.

All of these are MIT-licensed.

## The modules

| `require` | What it is |
|---|---|
| `require("cjson")` | Fast JSON encode/decode. |
| `require("cjson.safe")` | The never-throwing variant of cjson (returns `nil` + error instead of raising). |
| `require("lfs")` | LuaFileSystem -- directory listing, file attributes, `mkdir`, `chdir`. |
| `require("lpeg")` | LPeg -- Parsing Expression Grammars, for robust parsing/lexing. |
| `require("serpent")` | Table serializer and pretty-printer. |
| `require("inspect")` | Human-readable dump of nested tables (great for debugging). |
| `require("json")` | A pure-Lua JSON implementation (a lightweight alternative to cjson). |
| `require("re")` | LPeg's regex-like front-end (uses `lpeg` under the hood). |

### JSON with `cjson`

```lua
local cjson = require("cjson")

local s = cjson.encode({ name = "Fippy", level = 95, alive = true })
echo(s)                              -- {"name":"Fippy","level":95,"alive":true}

local t = cjson.decode(s)
echo(t.name .. " is level " .. t.level)
```

Use `cjson.safe` when you are decoding data you do not control and do not want an
error to stop your script:

```lua
local cjson = require("cjson.safe")
local t, err = cjson.decode(maybeBadInput)
if not t then
    echo("bad JSON: " .. tostring(err))
end
```

`require("json")` is a pure-Lua fallback with a similar `encode` / `decode` API if
you prefer it.

### Listing files with `lfs`

Lua's own `io` / `os` cannot enumerate a directory; `lfs` fills that gap:

```lua
local lfs = require("lfs")
for entry in lfs.dir(".") do
    echo(entry)
end

lfs.mkdir("logs")                    -- create a directory
local attr = lfs.attributes("hello.lua")
echo("size: " .. tostring(attr and attr.size))
```

### Serializing tables with `serpent`

`serpent` turns a table into Lua source text you can save and load back -- ideal
for persisting state between runs:

```lua
local serpent = require("serpent")

local state = { runs = 3, best = 128, names = { "a", "b" } }
local text = serpent.dump(state)     -- compact, loadable
-- ...or serpent.block(state) for a pretty, human-readable form

-- read it back:
local ok, restored = serpent.load(text)
if ok then echo("runs so far: " .. restored.runs) end
```

### Debugging with `inspect`

```lua
local inspect = require("inspect")
echo(inspect(Me.Name))               -- see exactly what you have
echo(inspect({ 1, 2, nested = { x = true } }))
```

### Parsing with `lpeg` / `re`

For parsing structured text -- chat logs, combat logs, small DSLs -- `lpeg` (and
its friendlier `re` front-end) is far more robust than string patterns:

```lua
local re = require("re")
local number = re.compile("%d+")
echo(tostring(number:match("abc123") == nil))   -- pattern basics
```

See the upstream LPeg / re documentation for the full grammar syntax.

## Loading your own modules

`package.path` and `package.cpath` are set to your InnerSpace **Scripts**
directory (and a `lua_modules` subdirectory inside it), so you can split a script
into your own reusable modules and `require` them:

```lua
-- Scripts\mymodule.lua  or  Scripts\lua_modules\mymodule.lua
local mymodule = require("mymodule")
```

The search covers `mymodule.lua`, `mymodule/init.lua`, and the same under
`lua_modules`. The bundled modules above resolve first (they are already loaded
internally), so they never depend on the path.

Next: `06_Migration_Gotchas.md`.
