# File 39: Extension Reference Index

## Overview

This file provides a comprehensive index and navigation guide for the three critical InnerSpace/LavishScript extensions used in EVE Online bot development. Use this as a quick reference to understand which extension provides specific functionality and where to find detailed documentation.

## Available Extensions Summary

### 1. ISXEVE - EVE Online Game Extension

**Purpose:** Core EVE Online game interaction - the foundation of all EVE bot development

**Key Capabilities:**
- Character and ship control
- Entity detection and targeting
- Module management and activation
- Inventory and cargo operations
- UI window interaction
- Fleet and social systems
- Market operations
- Scanning and exploration
- Bookmarks and navigation

**TLOs (20+ total):**
- `Me` - Player character (character → pilot → being)
- `MyShip` - Active ship
- `EVE` - Main game interface
- `EVEWindow[...]` - UI window access
- `Entity[id]`, `Being[id]` - Entity access
- `Chat`, `Local` - Chat system
- `Universe[id]` - Universe objects
- `Overview` - Overview table
- `CharSelect` - Character selection
- `EVETime` - EVE time

**Datatypes (50+ total):**
- Character/Pilot/Being (inheritance chain)
- Entity (ships, NPCs, gates, asteroids, wrecks, drones)
- Module → Item → ItemInfo (ship equipment)
- Ship (player ship with cargo, modules, stats)
- EVEWindow hierarchy (inventory, fitting, sell, market, etc.)
- Bookmark, MarketOrder, Skill, Fleet, etc.

**Reference File:** `__CRITICAL_NEWEST_ISXEVE_Reference.md` (3,847 lines)
**Last Updated:** January 2025
**Covered in Bible Files:** 09-15, 21-23, 35, 37

---

### 2. isxSQLite - Database Extension

**Purpose:** Persistent data storage for configuration, session tracking, market data, and cross-session state management

**Key Capabilities:**
- Disk and memory databases (`:Memory:` for temporary)
- SQL INSERT, UPDATE, DELETE, SELECT operations
- Transaction support (10-100x faster for bulk operations)
- SQL injection prevention (`Escape_String`)
- Event-driven error/status reporting
- HTTP/HTTPS requests (optional libisxGames integration)

**TLOs:**
- `ISXSQLite` - Extension version and status
- `SQLite` - Main database interface

**Datatypes:**
- `sqlitedb` - Database connection
- `sqlitequery` - Query results (step-through)
- `sqlitetable` - In-memory table data

**Common Use Cases:**
- Bot configuration storage (alternative to XML files)
- Session statistics tracking (kills, loot, profit, ore mined)
- Market data caching and analysis
- Player/corp/alliance tracking databases
- Cross-session state persistence
- Audit trails and history logging

**Reference File:** `__CRITICAL_NEWEST_isxSQLite_Reference.md` (1,075 lines)
**Version:** 20200812 (SQLite 3.32.3)
**Covered in Bible Files:** 29 (389 lines dedicated section), 37 (code examples)

---

### 3. ISXIM - IRC Communication Extension

**Purpose:** Internet Relay Chat (IRC) integration for remote monitoring, bot command & control, and multi-bot coordination

**Key Capabilities:**
- IRC server connection and authentication
- Channel joining/leaving
- Private messages and channel messages
- Event-driven message handling (17 IRC events)
- Multi-bot coordination via IRC channels
- Remote bot monitoring and alerting
- Cross-computer communication

**TLOs:**
- `IM` - Extension version and status
- `IRC` - Connection manager
- `IRCUser[nickname]` - Specific IRC connection

**Datatypes:**
- `im` - Extension info
- `irc` - IRC system
- `ircuser` - IRC connection
- `channel` - IRC channel
- `nick` - Channel member

**IRC Events (17 total):**
- Message events: ReceivedChannelMsg, ReceivedPrivateMsg, ReceivedNotice, ReceivedEmote, ReceivedCTCP
- User events: NickJoinedChannel, NickLeftChannel, NickQuit, NickChanged, KickedFromChannel
- Channel events: TopicSet, NickTypeChange, ChannelModeChange
- Error/system events: PRIVMSGErrorResponse, JOINErrorResponse, UnhandledEvent, UserDisconnected

**Common Use Cases:**
- Bot command & control from IRC client
- Multi-bot fleet coordination (commands broadcast to IRC channel)
- Status reporting (bots report to IRC channel)
- Cross-computer IPC (IRC as relay between machines)
- Real-time debugging and monitoring
- User interaction (accept commands from human operators)
- Critical alerts and notifications

**Reference File:** `__CRITICAL_NEWEST_ISXIM_Reference.md` (1,272 lines)
**Last Updated:** March 2022
**Covered in Bible Files:** 28 (229 lines IRC Bridge Integration), 24, 30

---

## Quick Feature Lookup Table

| Need to... | Use Extension | Primary TLO/Datatype | See Bible File |
|------------|---------------|----------------------|----------------|
| **Character & Ship** |
| Access character info | ISXEVE | `Me` (character) | File 09 |
| Get ship stats | ISXEVE | `MyShip` (ship) | File 09 |
| Control ship modules | ISXEVE | `MyShip.Module[#]` | File 12 |
| Access cargo/inventory | ISXEVE | `EVEWindow[Inventory]` | File 13, 15 |
| Check skill levels | ISXEVE | `Me.Skill[name]` | File 09 |
| **Entity System** |
| Find nearby ships | ISXEVE | `Entity[...]` | File 10 |
| Target NPCs | ISXEVE | `Me:ActiveTarget` | File 10 |
| Query entities | ISXEVE | `EVE:QueryEntities` | File 10 |
| Get distance to object | ISXEVE | `Entity.Distance2` | File 10 |
| **Combat** |
| Activate weapons | ISXEVE | `Module:Click` | File 12, 21 |
| Target management | ISXEVE | `Entity:LockTarget` | File 10, 21 |
| Tank monitoring | ISXEVE | `MyShip.Shield/Armor/Hull` | File 09, 21 |
| **Mining** |
| Survey scan | ISXEVE | `MyShip.Scanners.Survey` | File 22 |
| Target asteroids | ISXEVE | `Entity[CategoryID=25]` | File 22 |
| Check ore hold | ISXEVE | `EVEWindow[Inventory]` | File 22 |
| **Navigation** |
| Autopilot to system | ISXEVE | `EVE:Execute[CmdSetDestination]` | File 11 |
| Dock at station | ISXEVE | `Entity:Dock` | File 11 |
| Use stargate | ISXEVE | `Entity:Jump` | File 11 |
| **Data Storage** |
| Store configuration | isxSQLite | `SQLite:OpenDB[name,path]` | File 29 |
| Track session stats | isxSQLite | `DB:ExecDML[INSERT...]` | File 37 |
| Query saved data | isxSQLite | `DB:ExecQuery[SELECT...]` | File 29 |
| Bulk data inserts | isxSQLite | `DB:ExecDMLTransaction` | File 29 |
| **Communication** |
| Connect to IRC | ISXIM | `IRC:Connect[server,nick]` | File 28 |
| Join IRC channel | ISXIM | `IRCUser:Join[#channel]` | File 28 |
| Send messages | ISXIM | `Channel:Say[message]` | File 28 |
| Remote monitoring | ISXIM | IRC events | File 28, 30 |
| **UI Interaction** |
| Open windows | ISXEVE | `EVE:Execute[OpenXXX]` | File 15 |
| Click buttons | ISXEVE | `EVEWindow:ClickButton` | File 15 |
| Access inventory | ISXEVE | `EVEWindow[Inventory]` | File 13, 15 |
| Market operations | ISXEVE | `EVE:FetchMarketOrders` | File 09 |

---

## Integration Patterns

### Pattern 1: Basic Combat Bot (ISXEVE only)
```lavishscript
; Minimal bot - just ISXEVE extension
function main()
{
    while TRUE
    {
        ; Find and target NPCs
        if !${Me.ActiveTarget(exists)}
        {
            EVE:QueryEntities[NPCs,"CategoryID = 11 && Distance < 50000"]
            NPCs:GetIterator[NPC]
            if ${NPC:First(exists)}
                NPC.Value:LockTarget
        }

        ; Activate weapons
        MyShip.Module[XXXXX]:Click

        wait 10
    }
}
```
**Uses:** ISXEVE only
**Files:** 10 (entities), 12 (modules), 21 (combat patterns)

---

### Pattern 2: Mining Bot with Database Stats (ISXEVE + isxSQLite)
```lavishscript
; Mining bot with session tracking
variable(global) sqlitedb DB

function main()
{
    ; Open session database
    SQLite:OpenDB["BotStats","./BotStats.db"]
    DB:Set[${SQLite.DB["BotStats"]}]

    ; Create session record
    variable string SessionID = "${Time.Timestamp}"
    DB:ExecDML["INSERT INTO sessions (id, start_time) VALUES ('${SessionID.Escape}', ${Time.Timestamp})"]

    ; Mining loop
    while TRUE
    {
        ; Mining logic using ISXEVE...
        ; ...

        ; Track ore mined
        DB:ExecDML["UPDATE sessions SET ore_mined = ore_mined + ${OreAmount} WHERE id = '${SessionID.Escape}'"]

        wait 10
    }
}
```
**Uses:** ISXEVE + isxSQLite
**Files:** 22 (mining), 29 (database config), 37 (session tracking)

---

### Pattern 3: Fleet Bot with IRC Coordination (All 3 Extensions)
```lavishscript
; Fleet mining coordinator with database and IRC
variable(global) sqlitedb DB
variable(global) string MyIRCNick = "FleetBot_${Me.Name}"

function main()
{
    ; Initialize database for fleet stats
    SQLite:OpenDB["FleetStats","./FleetStats.db"]
    DB:Set[${SQLite.DB["FleetStats"]}]

    ; Connect to IRC for coordination
    IRC:Connect["irc.myserver.com","${MyIRCNick}"]
    wait 30 ${IRCUser[${MyIRCNick}].IsConnected}
    IRCUser[${MyIRCNick}]:Join["#fleet"]

    ; Main loop with all extensions
    while TRUE
    {
        ; ISXEVE: Mining logic
        ; ...

        ; isxSQLite: Track ore mined
        DB:ExecDML["INSERT INTO fleet_ore (pilot, ore_type, amount) VALUES (...)"]

        ; ISXIM: Report status to IRC
        if ${Math.Calc[${Script.RunningTime} % 300]} == 0
        {
            variable int TotalOre = ${DB.ExecScalar["SELECT SUM(amount) FROM fleet_ore WHERE pilot='${Me.Name}'"]}
            IRCUser[${MyIRCNick}].Channel["#fleet"]:Say["${Me.Name}: ${TotalOre} m³ ore mined"]
        }

        wait 10
    }
}

function OnIRC_ReceivedChannelMsg(string ircUser, string channel, string nickname, string message)
{
    ; Handle IRC commands from fleet coordinator
    if ${message.Equal["!dock"]}
    {
        ; ISXEVE: Dock at nearest station
        ; ...

        ; ISXIM: Confirm via IRC
        IRCUser[${MyIRCNick}].Channel["${channel}"]:Say["${Me.Name}: Docking now"]
    }
}
```
**Uses:** ISXEVE + isxSQLite + ISXIM
**Files:** 22 (mining), 24 (fleet), 28 (IRC), 29 (database)

---

### Pattern 4: Database-Backed Configuration Manager
```lavishscript
; Using isxSQLite for bot configuration instead of XML
objectdef obj_Configuration
{
    variable sqlitedb ConfigDB

    method Initialize()
    {
        SQLite:OpenDB["BotConfig","./config.db"]
        ConfigDB:Set[${SQLite.DB["BotConfig"]}]

        ; Create config table if not exists
        ConfigDB:ExecDML["CREATE TABLE IF NOT EXISTS config (key TEXT PRIMARY KEY, value TEXT)"]
    }

    member:string Get(string key, string default = "")
    {
        variable string result = "${ConfigDB.ExecScalar["SELECT value FROM config WHERE key='${key.Escape}'"]}"
        if ${result.Length} > 0
            return "${result}"
        return "${default}"
    }

    method Set(string key, string value)
    {
        ConfigDB:ExecDML["INSERT OR REPLACE INTO config (key, value) VALUES ('${key.Escape}', '${value.Escape}')"]
    }
}
```
**Uses:** isxSQLite only
**File:** 29 (389 lines of database config patterns)

---

### Pattern 5: IRC Remote Monitoring
```lavishscript
; Send critical alerts to IRC channel
function ReportCritical(string message)
{
    if ${IRCUser[BotMonitor](exists)} && ${IRCUser[BotMonitor].IsConnected}
    {
        IRCUser[BotMonitor].Channel["#alerts"]:Say["[CRITICAL] ${Me.Name}: ${message}"]
    }
}

; Usage in bot:
if ${Me.Attacked(exists)}
{
    ReportCritical("Under attack by ${Me.Attacked.Name}!")
    ; Emergency procedures...
}

if ${MyShip.ArmorPct} < 50
{
    ReportCritical("Armor at ${MyShip.ArmorPct.Precision[1]}% - emergency warp!")
    ; Warp out...
}
```
**Uses:** ISXEVE + ISXIM
**Files:** 28 (IRC bridge), 30 (debugging/monitoring)

---

## Extension Version Information

| Extension | Last Updated | Version Details | Stability |
|-----------|--------------|-----------------|-----------|
| ISXEVE | January 2025 | Item.FitToActiveShip removed (latest change) | Active Development |
| isxSQLite | August 2020 | Version 20200812, SQLite 3.32.3 | Stable |
| ISXIM | March 2022 | Yahoo Messenger removed (latest change) | Stable |

**Note:** ISXEVE has ongoing updates as CCP changes EVE Online. The July 2020 unified inventory system change was the most significant recent update, deprecating all old cargo API methods.

---

## When to Use Each Extension

### Use ISXEVE When:
- ✅ Interacting with EVE Online game directly
- ✅ Reading game state (character, ship, entities, market)
- ✅ Controlling character/ship actions
- ✅ Accessing UI windows and menus
- ✅ Every bot needs ISXEVE - it's the foundation

### Use isxSQLite When:
- ✅ Storing configuration (alternative to XML files)
- ✅ Tracking session statistics (kills, loot, ore mined, profit)
- ✅ Caching market data for analysis
- ✅ Building player/corp/alliance databases
- ✅ Need persistent state across bot restarts
- ✅ Complex data relationships (relational database)
- ✅ Audit trails and history logging

### Use ISXIM When:
- ✅ Remote monitoring of bot status
- ✅ Multi-bot coordination (commands via IRC)
- ✅ Cross-computer communication (bots on different PCs)
- ✅ Human operator interaction (accept commands from IRC client)
- ✅ Critical alerts and notifications
- ✅ Persistent chat logs for debugging
- ✅ Fleet coordination across multiple sessions

---

## Authoritative Reference Files

All three extension reference files are located in the BIBLE directory and should be considered the **authoritative source** for complete API coverage:

1. **`__CRITICAL_NEWEST_ISXEVE_Reference.md`**
   - 3,847 lines
   - 50+ datatypes with full member/method listings
   - Usage examples for all major operations
   - Complete deprecation history (2015-2025)
   - Inheritance hierarchies explained

2. **`__CRITICAL_NEWEST_isxSQLite_Reference.md`**
   - 1,075 lines
   - Complete SQL operation coverage
   - Transaction patterns and best practices
   - Event handling documentation
   - Performance optimization guidance

3. **`__CRITICAL_NEWEST_ISXIM_Reference.md`**
   - 1,272 lines
   - All 17 IRC events documented
   - Complete connection/authentication examples
   - Channel management patterns
   - Nickserv integration

**When in doubt, consult the reference files** - the Bible files (01-38) provide curated examples and patterns, but the reference files are comprehensive.

---

## Cross-Reference Map

### ISXEVE Coverage in Bible Files:
- **File 09:** Core objects (Me, MyShip, EVE) and TLO reference
- **File 10:** Entity system and targeting (Distance2 optimization)
- **File 11:** Navigation and autopilot
- **File 12:** Module management and ship control
- **File 13:** Inventory/cargo (DEPRECATED API - legacy reference only)
- **File 15:** UI windows and menus (modern EVEWindow API)
- **File 21:** Combat bot patterns
- **File 22:** Mining bot patterns
- **File 23:** Hauling and logistics
- **File 35:** Common objects and methods reference

### isxSQLite Coverage in Bible Files:
- **File 29:** Configuration and Settings Management (389 lines dedicated section)
  - Basic database config pattern
  - Config object with database backend
  - Hybrid XML + database approach
  - Config with history/versioning
  - Performance comparisons
- **File 37:** Common Code Snippets
  - Database-backed config manager
  - Session statistics tracker

### ISXIM Coverage in Bible Files:
- **File 28:** Relay System and IPC (229 lines IRC Bridge Integration)
  - Tehbot ChatRelay pattern
  - Multi-computer coordination
  - Remote control via IRC
  - Event handling examples
- **File 24:** Multi-Boxing and Fleet Coordination (cross-reference)
- **File 30:** Debugging Techniques (IRC remote monitoring)

---

## Migration Notes

### ISXEVE: Deprecated Cargo API (July 2020)
**⚠️ CRITICAL:** The old cargo API was completely deprecated in July 2020 and replaced with unified inventory windows.

**DEPRECATED (Do NOT use):**
```lavishscript
MyShip:GetCargo[cargoItems]
MyShip.Cargo[#]
MyShip.CargoCapacity
MyShip.UsedCargoCapacity
MyShip.CargoFreeSpace
MyShip.OreHoldCapacity
```

**MODERN (Use instead):**
```lavishscript
EVEWindow[Inventory].Child[ShipCargo]:GetItems[items]
EVEWindow[Inventory].Child[ShipCargo].Capacity
EVEWindow[Inventory].Child[ShipCargo].UsedCapacity
EVEWindow[Inventory].Child[ShipCargo].FreeSpace
EVEWindow[Inventory].Child[ShipOreHold].Capacity
```

**See:** Files 09, 13, 15 for complete migration guidance

### ISXEVE: Distance2 Performance Optimization
**⚠️ PERFORMANCE:** Always use `Distance2` (squared distance) for range comparisons - it's 3-5x faster than `Distance`.

```lavishscript
; SLOW - calculates sqrt(x²+y²+z²)
if ${Entity.Distance} < 50000

; FAST - compares x²+y²+z² directly (no square root)
if ${Entity.Distance2} < ${Math.Calc[50000*50000]}
```

**See:** File 10 for comprehensive Distance2 optimization patterns

---

## Summary

This index provides navigation to the three essential extensions for EVE Online bot development:

- **ISXEVE** - Foundation for all EVE interaction (every bot uses this)
- **isxSQLite** - Optional but powerful for data persistence and configuration
- **ISXIM** - Optional but valuable for remote monitoring and multi-bot coordination

Use the Quick Feature Lookup Table to find specific functionality, consult Integration Patterns for multi-extension examples, and refer to the authoritative reference files for complete API coverage.

**New to bot development?** Start with:
1. File 09 (ISXEVE Core Objects) to understand Me, MyShip, EVE
2. File 10 (Entity System) to learn entity queries and targeting
3. File 12 (Module Management) for ship control
4. Then explore isxSQLite (File 29) and ISXIM (File 28) as needed

---

**File 39 - Extension Reference Index**
**Last Updated:** 2025-10-10
**Related Files:** All reference files (__CRITICAL_NEWEST_*.md), Files 09-15, 21-24, 28-29, 35, 37
