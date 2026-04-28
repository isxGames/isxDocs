# ISXEVE Quick Reference

<!-- CLAUDE_SKIP_START -->
**Purpose:** Single-file quick reference for every TLO, datatype, command, and event in ISXEVE, plus common patterns and cross-references to the in-depth [ISXEVE Scripting Guide](ISXEVE%20Scripting%20Guide/README.md).
**Audience:** Script authors who need fast lookup of API signatures, valid member/method names, and canonical usage idioms.
**Authoritative source:** `ISXEVEChanges.txt` (the ISXEVE changelog) is the definitive source for the API surface documented here. Where this document and a guide file disagree, the changelog wins. Every member/method/command/event below has been verified against the changelog as of the rewrite date on the commit that introduced this file.
<!-- CLAUDE_SKIP_END -->

---

## Table of Contents

1. [Bootstrap and Safety Checks](#bootstrap-and-safety-checks)
2. [Top-Level Objects](#top-level-objects)
3. [Datatype Inheritance Map](#datatype-inheritance-map)
4. [Character and Being Datatypes](#character-and-being-datatypes)
5. [Ship and Module Datatypes](#ship-and-module-datatypes)
6. [Item and Inventory Datatypes](#item-and-inventory-datatypes)
7. [Entity and Space Datatypes](#entity-and-space-datatypes)
8. [Station and Structure Datatypes](#station-and-structure-datatypes)
9. [Interstellar and Bookmark Datatypes](#interstellar-and-bookmark-datatypes)
10. [Window Datatypes](#window-datatypes)
11. [UI Element Datatypes](#ui-element-datatypes)
12. [Market Datatypes](#market-datatypes)
13. [Scanner Datatypes](#scanner-datatypes)
14. [Agent and Mission Datatypes](#agent-and-mission-datatypes)
15. [Skill Datatypes](#skill-datatypes)
16. [Chat Datatypes](#chat-datatypes)
17. [Fleet Datatypes](#fleet-datatypes)
18. [Drone Datatypes](#drone-datatypes)
19. [Misc Datatypes](#misc-datatypes)
20. [Commands](#commands)
21. [Events](#events)
22. [EVE:Execute Command Constants](#eveexecute-command-constants)
23. [Common Patterns and Idioms](#common-patterns-and-idioms)
24. [Canonical Slot Names and Destinations](#canonical-slot-names-and-destinations)
25. [Deprecated and Removed](#deprecated-and-removed)
26. [Notes](#notes)

---

## Bootstrap and Safety Checks

**Every ISXEVE script must gate its first API access behind `${ISXEVE.IsReady}`**. Without this the extension may not be initialized and members will silently return NULL.

```lavishscript
function main()
{
    while !${ISXEVE.IsReady}
        wait 10

    ; Safe to access ISXEVE APIs from this point on
    while TRUE
    {
        if ${Me.ToEntity(exists)}
            echo "Ship: ${MyShip.ToItem.Name} at ${Me.ToEntity.X},${Me.ToEntity.Y},${Me.ToEntity.Z}"
        wait 50
    }
}
```

**Existence checks — always validate before dereferencing.** Members and TLOs returning datatype objects can be NULL when the queried object does not exist or is out of context (e.g. station methods in space, entity lookups for IDs that have despawned). Use `(exists)` guards:

```lavishscript
if ${Entity[${TargetID}](exists)}
    echo "Target: ${Entity[${TargetID}].Name}"

if ${Me.Fleet.ID(exists)}
    echo "Fleet of size ${Me.Fleet.Size}"

if ${Me.ActiveTarget(exists)}
    echo "Active target: ${Me.ActiveTarget.Name}"
```

**Async data prerequisites.** Several APIs require a one-time UI interaction before returning valid data in the current session:

| API | Prerequisite |
|---|---|
| `entity.CargoCapacity` / `entity.UsedCargoCapacity` / `entity.GetCargo` | The entity's cargo hold must have been opened at least once in-session (`IsCargoAccessible` becomes TRUE). |
| `MyShip.CargoCapacity` / `MyShip.UsedCargoCapacity` | The ship's cargo window must have been opened at least once, or use the `EVEWindow[Inventory]` child-window pattern which triggers the open implicitly. |
| `EVEWindow[Inventory].ChildWindow[...].Capacity` / `.UsedCapacity` | Returns `-1` on error and `-2` when the child has not yet been made active. |
| `Me.Wallet.Balance` / `Me.Wallet.BalanceAUR` / `Me.Corp.Wallet.Balance` | Returns `-1.0` until the wallet window has been opened at least once. |
| `Universe[name]` (string lookup) | First string lookup per session caches the entire name database (~100k+ names) and lags the client. Prefer ID lookups. |

See [03_API_Reference.md](ISXEVE%20Scripting%20Guide/03_API_Reference.md) for the full async-data discussion.

---

## Top-Level Objects

Top-Level Objects (TLOs) are the entry points for accessing game data. Syntax: `${TLO}` for parameter-less, `${TLO[argument]}` for parameterized.

### Extension TLO

| TLO | Datatype | Description |
|-----|----------|-------------|
| `ISXEVE` | [isxeve](#isxeve) | Extension status, debug hooks, utility members. |

### Core Game TLOs

| TLO | Datatype | Description |
|-----|----------|-------------|
| `EVE` | [eve](#eve) | Main game universe object. Entities, bookmarks, agents, market, drones. |
| `Me` | [character](#character) | Your character. Inherits from [pilot](#pilot) which inherits from [being](#being). |
| `MyShip` | [ship](#ship) | Your active ship. Same entity as `Me.Ship`. |
| `EVETime` | [evetime](#evetime) | Current EVE server time. |
| `Local[id]` / `Local[name]` | [pilot](#pilot) | Access a pilot from local chat by CharID (int64) or name. |
| `Chat` | [evechat](#evechat) | Chat system root. |
| `Chat[id]` / `Chat[name]` | [chatchannel](#chatchannel) | Specific chat channel by ID or name. |
| `Universe[id]` / `Universe[name]` | [interstellar](#interstellar) | Universe object — solar system, constellation, region, planet, moon, etc. Returned type depends on what the ID/name resolves to. |
| `Entity[id]` | [entity](#entity) | Entity on the current grid by ID. Only works in space. |
| `Entity[name]` | [entity](#entity) | Entity by name. Only works in space. |
| `Being[id]` | [being](#being) | Being (character/NPC) by CharID. |
| `CharSelect` | [charselect](#charselect) | Character selection screen. Only valid at login before character selection. |
| `Overview` | [overview](#overview) | Overview selected-item access. |

### EVEWindow TLO

All variants return an [evewindow](#evewindow) (or a more specialized subtype when the window implies one, e.g. `evefittingwindow`, `eveinvwindow`, `evecustomsofficewindow`).

| Syntax | Description |
|---|---|
| `EVEWindow[name]` | Window by one-word name (e.g. `Inventory`, `Fitting`, `RepairShop`, `SellItems`, `MessageBox`). |
| `EVEWindow[byName,name]` | Window by exact internal name (e.g. `byName,modal`, `byName,PlanetaryImportExportUI`). |
| `EVEWindow[byCaption,text]` | Window by visible caption text (e.g. `byCaption,Agent Conversation`). |
| `EVEWindow[byItemID,id]` | Window associated with an item ID. |
| `EVEWindow[active]` | Currently focused window. |
| `EVEWindow[local]` | Local chat window. |
| `EVEWindow[#]` | Window by sequential index. |

Specialized subtypes returned automatically by `EVEWindow`:

- `Inventory` → [eveinvwindow](#eveinvwindow)
- `Fitting` → [evefittingwindow](#evefittingwindow)
- `SellItems` → [evesellitemswindow](#evesellitemswindow)
- `MarketAction` / `Market Action` → [evemarketactionwindow](#evemarketactionwindow)
- `RepairShop` / `Repair Shop` → [everepairshopwindow](#everepairshopwindow)
- `MessageBox` / `Message Box` → [evemessageboxwindow](#evemessageboxwindow)
- `directionalScannerWindow` → [evedirectionalscannerwindow](#evedirectionalscannerwindow)
- Customs-office windows (e.g. `byName,PlanetaryImportExportUI`) → [evecustomsofficewindow](#evecustomsofficewindow)
- Agent dialog (`byCaption,Agent Conversation`) → [eveagentdialogwindow](#eveagentdialogwindow)

---

## Datatype Inheritance Map

```
being
  └─ pilot
       ├─ character                (Me TLO)
       └─ fleetmember

iteminfo
  └─ item
       └─ module

iteminfo
  └─ iteminfolist

interstellar
  ├─ solarsystem
  ├─ constellation
  ├─ region
  └─ planet

station
  └─ structure                     (citadels)

entity
  ├─ entitywormhole
  ├─ entityplayerstructure
  ├─ attacker
  │    └─ jammer
  └─ activedrone                   (via ToActiveDrone)

evewindow
  ├─ eveinvwindow
  ├─ evefittingwindow
  ├─ evesellitemswindow
  ├─ evemarketactionwindow
  ├─ everepairshopwindow
  ├─ eveagentdialogwindow
  ├─ evemessageboxwindow
  ├─ evedirectionalscannerwindow
  └─ evecustomsofficewindow
```

Inheritance means child types expose ALL members and methods of their parent(s). Casting between types uses explicit `To<Type>` members (e.g. `entity.ToFleetMember`, `pilot.ToEntity`, `module.ToItem`, `character.ToPilot`).

---

## Character and Being Datatypes

### isxeve

Access: `ISXEVE` TLO.

**Members**

| Member | Type | Description |
|---|---|---|
| `Version` | string | Extension version string. |
| `IsReady` | bool | TRUE when ISXEVE is fully loaded and safe to use. |
| `IsLoading` | bool | TRUE while ISXEVE is initializing. |
| `IsSafe` | bool | TRUE if the loaded build is the "safe" (release) build. |
| `IsBeta` | bool | TRUE if the loaded build is the beta build. |
| `Debug1` | bool | Internal debug flag. |
| `SecsToString[int]` | string | Formats a seconds count as `X seconds` / `X minutes and Y seconds` / `X hours, Y minutes, and Z seconds` / `X days, Y hours, Z minutes, and W seconds`. |
| `IsNumeric[string]` | bool | TRUE if the string parses as a number (decimal point and leading minus allowed). |

**Methods**

- `Flush` — Release dangling ISXEVE-managed objects. Recommended every 20-30 minutes for long-running scripts (especially .NET apps).
- `Unload` — Cleanly unload the extension.
- `InstallBeta`, `InstallTest`, `InstallLive` — Switch channel.
- `Debug_SetTypeValidation[bool]`, `Debug_SetEntityCacheEnabled`, `Debug_SetEntityCacheDisabled`, `Debug_SetHighPerfLogging[bool]`, `Debug_LogMsg[text]`, `Debug_PrintCacheInfo`, `Debug_DumpEntityCache` — Debug hooks.

### eve

Access: `EVE` TLO.

**Members**

| Member | Type | Description |
|---|---|---|
| `EntitiesCount` | int | Number of entities on the current grid. |
| `DistanceBetween[id1,id2]` | double | Distance between two entities (meters). |
| `Bookmark[label]` | [bookmark](#bookmark) | Bookmark by LABEL. Source only accepts a label string; despite changelog wording, the current implementation has no ID-lookup branch (there is a TODO in source to add one). To look up by ID, iterate `EVE:GetBookmarks[index:bookmark]` and filter by `bookmark.ID`. |
| `JumpsToStation[stationID]` | int | Jumps to a station via current autopilot route logic. |
| `JumpsBetween[fromID,toID]` | int | Jumps between two solar systems. |
| `JumpsTo[solarSystemID]` | int | Jumps to a solar system from current location. |
| `Time` | string | Current EVE time. |
| `Date` | string | Current EVE date. |
| `NumAssetsAtStation[stationID]` | int | Asset count at a station. |
| `GetLocationNameByID[int64]` | string | Resolves a location ID to a name. |
| `Station[id]` | [station](#station) | Station by ID. |
| `Structure[id]` | [structure](#structure) | Structure/citadel by ID. |
| `Agent` | int | Number of agents known to the character (0-arg form). |
| `Agent[#]` / `Agent[id, #]` / `Agent[name]` | [eveagent](#eveagent) | Agent by 1-based index (single int), by explicit agent ID (2-arg form with literal token `id`), or by name. |
| `ItemInfo[typeID]` | [iteminfo](#iteminfo) | Static type info by TypeID. |
| `Is3DDisplayOn` | bool | 3D display state. |
| `IsUIDisplayOn` | bool | UI display state. |
| `IsTextureLoadingOn` | bool | Texture loading state. |
| `NextSessionChange` | [evetime](#evetime) | Scheduled next session-change time. |
| `InCriticalSection` | bool | Whether a critical-section lock is active. |
| `IsProgressWindowOpen` | bool | Progress-window state. |
| `ProgressWindowTitle` | string | Progress-window title text. |
| `AbandonedDronesExist` | bool | Whether reclaimable abandoned drones exist nearby. |
| `MinWarpDistance` | double | Minimum warp distance in meters. |
| `QueryEvaluate[query,entity]` | bool | Evaluates a query expression against an entity (deprecated wrapper for `LavishScript:QueryEvaluate`; may be removed in future). |

**Methods**

| Method | Description |
|---|---|
| `Execute[Cmd...]` | Execute a named UI command (see [EVE:Execute Command Constants](#eveexecute-command-constants)). |
| `DoCommand[text]` | Run an EVE chat/slash command. |
| `InfoWindow[text]` | Open an informational popup window. |
| `SetInSpaceStatus[text]` | Set the in-space status text. |
| `Toggle3DDisplay`, `ToggleUIDisplay`, `ToggleTextureLoading` | Toggle client display states. |
| `CloseAllMessageBoxes`, `CloseAllChatInvites` | Mass-close helpers. |
| `GetEVEWindows[index:evewindow]` | Populate index with all open windows. |
| `QueryEntities[index:entity]` | Populate with all visible entities. |
| `QueryEntities[index:entity, "query"]` | Populate with entities matching a query string (e.g. `"CategoryID = 11 && Distance < 50000"`). |
| `QueryEntities[index:entity, QueryID]` | Populate with entities matching a precompiled QueryID. |
| `PopulateEntities[categoryID]` | Populate the client's entity cache for a category. |
| `GetViewedWrecks[index:int64]` | Wreck IDs that have been viewed. |
| `GetBookmarks[index:bookmark]` / `GetBookmarks[index:bookmark, "folder"]` | Bookmarks, optionally scoped to a folder. |
| `RefreshBookmarks` | Refresh bookmark cache. Call once after login before using bookmark APIs. |
| `CreateBookmark[label, notes, folder, expiry]` | Create a bookmark at current location. All params after label are optional. `expiry` is a discrete enum: `0` = no expiry (default), `1` = 3 hours, `2` = 2 days. Label capped at 100 chars, notes at 3900 chars. |
| `AddWaypoint[solarSystemID]`, `ClearWaypoint[solarSystemID]`, `ClearAllWaypoints` | Waypoint management. |
| `GetToDestinationPath[index:int]`, `GetWaypoints[index:int]` | Current route info. |
| `OptimizeAutopilotRoute` | Optimize the current waypoint route. |
| `GetLocalPilots[index:pilot]` | Populate with pilots currently in local. |
| `GetOnlineCorpMembers[index:being]` | Online corp members. |
| `GetContacts[index:being]` | Contacts list. |
| `GetAgents[index:eveagent]` | All agents known to the character. |
| `GetAgentMissions[index:agentmission, agentID]` | Active missions for an agent. |
| `FetchMarketOrders[typeID, regionID]` | Request market orders (async — wait for cache). |
| `ClearMarketOrderCache` | Clear cached market data. |
| `GetMarketOrders[index:marketorder, typeID]` | Pull cached market orders for a type. |
| `CreateMarketBuyOrder[typeID]` | Open the market buy-order window for a type. |
| `MoveItemsTo[index:int64, destinationID, destination, folder]` | Batch move items. The index must be `index:int64` (not `index:item`) — source validates this strictly. See [Canonical Slot Names and Destinations](#canonical-slot-names-and-destinations). |
| `StackItems[locationID, destination]` / `StackItems[locationName, destination]` / `StackItems[locationName, destination, folder]` | Stack items at a location. First arg is a location (an item/container ID, or one of the shorthands `MyShip`, `MyStationHangar`, `MyStationCorporateHangar`), second arg is a destination name (see [Canonical Slot Names and Destinations](#canonical-slot-names-and-destinations)). This is NOT an index-based batch method. |
| `LaunchDrones[index:int64 or index:item or index:activedrone]` | Launch drones. Source accepts any of the three index types. |
| `DronesMine[index:int64]`, `DronesMineRepeatedly[index:int64]` | Commanded mining. |
| `DronesScoopToDroneBay[index:int64]`, `DronesReturnToDroneBay[index:int64]`, `DronesReturnAndOrbit[index:int64]` | Return commands. |
| `DronesEngageMyTarget[index:int64]` | Engage active target. |
| `DronesAssist[index:int64, charID]`, `DronesGuard[index:int64, charID]` | Fleet-support drones. |
| `AbandonDrones[index:int64]` | Abandon specific drones. |
| `ReclaimDrones` | Reclaim abandoned drones in range. |
| `ReturnFighterControl[index:int64]`, `DelegateFighterControl[index:int64, charID]` | Fighter handoff. |
| `RefreshStandings` | Refresh the standings cache. Call at least once after login; data is not immediately available. |
| `EnterCriticalSection`, `LeaveCriticalSection` | Critical-section control. |
| `ViewPlanetaryIndustry[planetID]` | Open the Planetary Industry window. |

### evetime

**Members**

| Member | Type | Description |
|---|---|---|
| `DateAndTime` | string | Full formatted timestamp. |
| `Date` | string | Date portion. |
| `Time` | string | Time portion. |
| `AsInt64` | int64 | Time as FILETIME-style int64. |

### character

Access: `Me` TLO. Inherits from [pilot](#pilot) which inherits from [being](#being).

**Identity and Location**

| Member | Type | Notes |
|---|---|---|
| `Corp` | [corporation](#corporation) | Corporation object. Use `Me.Corp.ID` for the corp ID (there is no `Me.CorpID`). |
| `Alliance` / `AllianceID` / `AllianceTicker` | string / int / string | Alliance info (`AllianceTicker` can fail since EVE's 2015 optimization). |
| `Wallet` | [wallet](#wallet) | Personal wallet. |
| `RegionID` / `ConstellationID` / `SolarSystemID` | int | Current location IDs. |
| `ShipID` | int64 | Current ship ID. |
| `StationID` | int64 | Current station/structure ID, or 0 in space. |
| `InStation` / `InSpace` | bool | Location predicates. Both become true inside a citadel too. |
| `Station` | [station](#station) or [structure](#structure) | Returns structure when docked at a citadel, station otherwise. |
| `AutoPilotOn` | bool | Autopilot state. |

**Ship, Targets, and Drones**

| Member | Type | Notes |
|---|---|---|
| `Ship` | [ship](#ship) | Current ship (same as `MyShip` TLO). |
| `ActiveTarget` | [entity](#entity) | Currently targeted entity. Check with `Me.ActiveTarget(exists)`. |
| `TargetCount` / `TargetingCount` / `TargetedByCount` | int | Locked, locking-in-progress, and being-targeted counts. |
| `MaxLockedTargets` | int | Character-side max targets (compare with `MyShip.MaxLockedTargets`; effective max is the lower of the two). |
| `MaxActiveDrones` | int | Max simultaneous active drones. |
| `MiningDroneAmountBonus` | double | Skill-derived mining-drone bonus (%). |
| `DroneControlDistance` | double | Drone control range (meters). |

**Attributes and Skills**

| Member | Type | Notes |
|---|---|---|
| `Intelligence` / `Perception` / `Charisma` / `Willpower` / `Memory` | int | Base attributes. |
| `Skill[id]` / `Skill[name]` | [skill](#skill) | Specific skill. |
| `SkillPoints` | double | Total SP. (Returns double — 2013 change from int.) |
| `SkillCurrentlyTraining` | [skill](#skill) | Active training. |
| `SkillQueueLength` | int | Queue length. |

**Standings and Contacts**

| Member | Type | Notes |
|---|---|---|
| `StandingTo[id]` | float | Your standing toward a character/corp/faction. |
| `MaxJumpClones` | int | Max jump clones. |
| `Contact[#]` / `Contact[id, charID]` / `Contact[CharID, charID]` / `Contact[name]` | [being](#being) | Contact by 1-based index, by explicit CharID (2-arg form with literal token `id` or `CharID`), or by name. |
| `ToPilot` | [pilot](#pilot) | Cast to pilot. |
| `ToEntity` | [entity](#entity) | Cast to entity (only valid in space). |
| `ToFleetMember` | [fleetmember](#fleetmember) | Cast to fleet member (only valid when in a fleet). |
| `Fleet` | [fleet](#fleet) | Fleet object (guard with `Me.Fleet.ID(exists)`). |

**Methods**

- Inventory: `GetHangarItems[index:item]`, `GetHangarShips[index:item]`, `GetCorpHangarItems[index:item, folder]`, `GetCorpHangarShips[index:item, folder]`, `GetStationsWithAssets[index:int64]`, `GetAssets[index:item, stationID]`.
- UI: `OpenCorpHangar` — open the corp hangar window (must be docked in a station or structure). The 2012 changelog claims this method was removed from the `character` datatype, but source still registers and implements it (DataTypes.h:771, DT-Beings.cpp:928-939) by calling the EveCmd service's `OpenCorpHangar`. It works. `EVE:Execute[OpenHangarFloor]` is an equivalent alternative.
- Targeting: `GetTargets[index:entity]`, `GetTargeting[index:entity]`, `GetTargetedBy[index:entity]`.
- Combat intel: `GetAttackers[index:attacker]`, `GetJammers[index:jammer]`.
- Drones: `GetActiveDrones[index:activedrone]`, `GetActiveDroneIDs[index:int64]`.
- Skills: `GetSkills[index:skill]`, `GetSkillQueue[index:queuedskill]`.
- Standings: `SetCorpStanding[id, value]`, `SetPilotStanding[id, value]`.
- Market: `GetMyOrders[index:myorder]`, `UpdateMyOrders`.
- Movement: `SetVelocity[0-100]` (percent of max).

### pilot

Inherits from [being](#being). Returned by `Local[...]`, `Entity.ToPilot` (NPCs/PCs in space), and many methods.

| Member | Type | Notes |
|---|---|---|
| `Type` / `TypeID` | string / int | Current ship type (visible ship type). |
| `Corp` | string | Corp name (NOT corporation object; `character.Corp` is the object). |
| `Alliance` / `AllianceID` / `AllianceTicker` | string / int / string | Alliance info. |
| `Standing` | float | Your standing toward this pilot. |
| `WarFactionID` | int | Faction warfare ID. |
| `IsLimitedEngagement` / `IsSuspect` / `IsCriminal` | bool | Flagging predicates. |
| `ToEntity` | [entity](#entity) | Cast to entity (in-space only). |
| `ToFleetMember` | [fleetmember](#fleetmember) | Cast to fleet member (fleet-only). |

**Methods:** `SetStanding[value]`, `InviteToFleet`, `OpenShowInfo`.

### being

Base type for characters/NPCs.

| Member | Type | Notes |
|---|---|---|
| `ID` / `CharID` | int64 | Being/character ID. |
| `Name` | string | Character name. |
| `IsOnline` | bool | Online state. |
| `IsNPC` / `IsPC` | bool | NPC/PC flags. |

**Methods:** `InviteToFleet`, `GiveMoney[amount, reason]`.

### corporation

| Member | Type | Notes |
|---|---|---|
| `ID` | int64 | Corp ID. |
| `Name` | string | Corp name. |
| `Ticker` | string | Corp ticker. |
| `Wallet` | [corporationwallet](#corporationwallet) | Corp wallet (your corp only). |

### corporationwallet

| Member | Type | Notes |
|---|---|---|
| `Balance` | double | ISK balance. Returns `-1.0` until the wallet window has been opened. |

### wallet

| Member | Type | Notes |
|---|---|---|
| `Balance` | double | ISK balance. Returns `-1.0` until the wallet has been opened this session. |
| `BalanceAUR` | double | Aurum balance. Returns 0 until the wallet has been opened. |

### standing

Standings lookup. Usually obtained via chained access on character/corp/alliance IDs.

| Member | Type | Notes |
|---|---|---|
| `MeToAlliance[id]` / `MeToCorp[id]` / `MeToPilot[id]` | double | Your standings. |
| `CorpToAlliance[id]` / `CorpToCorp[id]` / `CorpToPilot[id]` | double | Your corp's standings. |
| `AllianceToAlliance[id]` / `AllianceToCorp[id]` / `AllianceToPilot[id]` | double | Your alliance's standings. |

Deep-dive: [03_API_Reference.md — Me Object](ISXEVE%20Scripting%20Guide/03_API_Reference.md#me-object).

---

## Ship and Module Datatypes

### ship

Access: `MyShip` TLO or `Me.Ship`. Not inherited.

**Identity**

| Member | Type | Notes |
|---|---|---|
| `ID` | int64 | Ship ID. |
| `Name` | string | Ship name (custom or default). |
| `ToItem` | [item](#item) | Ship as item (for Name, TypeID, etc.). |
| `ToEntity` | [entity](#entity) | Ship as entity (in-space only). |

**Cargo and Bay Access**

Note: `CargoCapacity` / `UsedCargoCapacity` require the cargo to have been opened this session (see [Bootstrap and Safety Checks](#bootstrap-and-safety-checks)). For reliable totals, prefer `EVEWindow[Inventory].ChildWindow[ShipCargo].Capacity` / `.UsedSpace`.

| Member | Type | Notes |
|---|---|---|
| `CargoCapacity` | double | Total cargo capacity (m³). |
| `UsedCargoCapacity` | double | Used cargo (m³). |
| `HasOreHold` | bool | TRUE if the ship hull has a specialized ore hold. |
| `Cargo[#]` / `Cargo[name]` | [item](#item) | Cargo item by 1-based index or name. Use `:GetCargo` / inventory window for iteration. |
| `Module[slotName]` | [module](#module) | Module in slot. `slotName` is `HiSlot0..HiSlot7`, `MedSlot0..MedSlot7`, `LoSlot0..LoSlot7`, or `RigSlot0..RigSlot7`. Also accepts int64 module item ID. |
| `Drone[#]` / `Drone[name]` | [item](#item) | Drone in drone bay by 1-based index or name. |

**Drone Bay**

| Member | Type | Notes |
|---|---|---|
| `DroneBandwidth` | double | Bandwidth remaining. |
| `DronebayCapacity` | double | Total drone bay capacity. |
| `UsedDronebayCapacity` | double | Used drone bay. |

**Capacitor**

| Member | Type | Notes |
|---|---|---|
| `Capacitor` | double | Current capacitor energy. |
| `MaxCapacitor` | double | Max capacitor. |
| `CapacitorPct` | double | Percent (0-100). |
| `CapacitorRechargeRate` | double | Recharge rate (ms). |

**Defense**

| Member | Type | Notes |
|---|---|---|
| `Shield` / `MaxShield` / `ShieldPct` | double | Shield HP (current, max, percent). |
| `ShieldRechargeRate` | double | Ms. |
| `Armor` / `MaxArmor` / `ArmorPct` | double | Armor HP. |
| `Structure` / `MaxStructure` / `StructurePct` | double | Structure HP. |

**IMPORTANT**: `Shield`, `Armor`, `Structure`, `Capacitor` are FLAT HP/EP values. Use the `Pct` variants (`ShieldPct`, `ArmorPct`, `StructurePct`, `CapacitorPct`) for percentages. Chained forms like `MyShip.Shield.Pct` do NOT exist.

**Resources (CPU/PG)**

| Member | Type |
|---|---|
| `CPULoad` / `CPUOutput` / `PowerLoad` / `PowerOutput` | double |

**Slots**

| Member | Type | Notes |
|---|---|---|
| `HighSlots` / `MediumSlots` / `LowSlots` | double | Slot counts. Note: registered as doubles in the changelog; safe to use as integers. |
| `RigSlots` / `RigSlotsLeft` | double | Rig slot totals and remaining empty. |
| `TurretSlotsLeft` | double | Empty turret slots. |
| `LauncherSlotsLeft` | double | Empty launcher slots. |

Canonical module iteration:

```lavishscript
variable index:module Mods
MyShip:GetModules[Mods]
variable iterator it
Mods:GetIterator[it]
if ${it:First(exists)}
    do
    {
        echo "${it.Value.ToItem.Name}  active=${it.Value.IsActive}"
    }
    while ${it:Next(exists)}
```

**Heat**

| Member | Type |
|---|---|
| `HeatHigh` / `HeatMedium` / `HeatLow` | int |
| `HeatCapacityHigh` / `HeatCapacityMedium` / `HeatCapacityLow` | double |

**Attributes and Targeting**

| Member | Type | Notes |
|---|---|---|
| `MaxVelocity` | double | Max sublight velocity (m/s). |
| `Agility` | double | Hull agility. |
| `Radius` | double | Hull radius (m). |
| `TechLevel` | double | Tech level. |
| `SignatureRadius` | double | Signature radius. |
| `WarpSpeedMultiplier` | double | Warp-speed multiplier (AU/s scaling). The 2007 changelog documented this as `WarpFactor` but the member is registered as `WarpSpeedMultiplier` in source. |
| `MaxLockedTargets` | double | Ship-side max targets (effective max is `min(ship, character)`). |
| `MaxTargetRange` | double | Max lock range (m). |
| `ScanSpeed` / `ScanResolution` / `ScanRadarStrength` | double | Sensor stats. |

**Scanners**

| Member | Type |
|---|---|
| `Scanners` | [scanners](#scanners) |

**Methods**

- Module/equipment iteration: `GetModules[index:module]`, `GetRigs[index:item]`, `GetDrones[index:item]`.
- Drone control: `LaunchAllDrones`.
- Fitting: `StripFitting`.
- Window: `Open` (opens unified inventory centered on ship).
- Movement: `Approach[x, y, z]` and `Align[x, y, z]` — these accept a three-component direction VECTOR (in space coordinates), not an entity ID or distance. To align toward or approach an entity, use the entity's own `AlignTo` / `Approach` methods instead of `MyShip:Align` / `MyShip:Approach`.
- Cargo: `Jettison[index:item or index:int64]`.
- POS: `SetStarbaseForcefieldPassword[password]`.
- Danger: `SelfDestruct`.

Deep-dive: [03_API_Reference.md — MyShip Object](ISXEVE%20Scripting%20Guide/03_API_Reference.md#myship-object) and [03 — Module Management](ISXEVE%20Scripting%20Guide/03_API_Reference.md#module-management-and-ship-control).

### module

Inherits from [item](#item).

**State**

| Member | Type | Notes |
|---|---|---|
| `IsActive` | bool | Currently cycling. |
| `IsActivatable` | bool | Can be activated (requires a valid target if offensive). |
| `IsOnline` | bool | Powered on. |
| `IsOffensive` / `IsAssistance` | bool | Effect category. |
| `IsGoingOnline` | bool | Bringing online. |
| `IsReloading` / `IsReloadingAmmo` | bool | Reload in progress. |
| `IsWaitingForActiveTarget` | bool | Waiting for a locked target before it can fire. |
| `IsDeactivating` | bool | Shutting off. |
| `IsBeingRepaired` | bool | Module repair in progress. |
| `IsBlinking` | bool | UI-blinking (usually deactivation warning). |
| `IsAutoReloadOn` | bool | Auto-reload setting. |
| `AutoRepeat` | bool | Auto-repeat setting. |
| `IsBankSlave` / `IsBankMaster` | bool | Capacitor/cap-booster banking. |

**Charge and Ammo**

| Member | Type | Notes |
|---|---|---|
| `EffectCategory` | string | Category label. |
| `Charge` | [modulecharge](#modulecharge) | Current charge object (if any). |
| `CurrentCharges` / `MaxCharges` | int | Charge counts. |
| `ChargeSize` | int | Size class. |

**Targeting**

| Member | Type | Notes |
|---|---|---|
| `Target` | [entity](#entity) | Current target of this module. |
| `TargetID` | int64 | Current target ID. |
| `LastTarget` / `LastTargeted` | [entity](#entity) | Historical targets. |
| `LastTargetedID` | int64 | Last target ID. |

**Requirements and Stats**

| Member | Type |
|---|---|
| `ToItem` | [item](#item) |
| `PowergridUsage` / `CPUUsage` | double |
| `ActivationCost` | double |
| `ActivationTime` / `Duration` / `RateOfFire` / `ReactivationDelay` | double |
| `OptimalRange` / `AccuracyFalloff` / `EffectivenessFalloff` | double |
| `TrackingSpeed` | double |
| `DamageModifier` | double |
| `EMDamage` / `KineticDamage` / `ThermalDamage` / `ExplosiveDamage` | double |
| `SignatureResolution` | double |
| `HP` / `Damage` | double (module hit points and current damage) |
| `TechLevel` | int |
| `Volume` / `Capacity` / `Mass` | double |

**Mining**

| Member | Type |
|---|---|
| `MiningAmount` / `MiningAmountPerSecond` / `MiningAmountBonus` | double |
| `SpecialtyCrystalMiningAmount` / `CrystalsDamage` | double |
| `TargetGroup` | int |
| `SurveyScanRange` | double |
| `UsesFrequencyCrystals` | bool |

**Tank/Resist Bonuses**

| Member | Type |
|---|---|
| `ArmorHPRepaired` / `ArmorHPBonus` | double |
| `ShieldBonus` / `ShieldHPBonus` | double |
| `EMDmgResistanceBonus` / `KineticDmgResistanceBonus` / `ThermalDmgResistanceBonus` / `ExplosiveDmgResistanceBonus` | double |

**Overload Bonuses**

| Member | Type |
|---|---|
| `OverloadRateOfFireBonus` / `OverloadOptimalRangeBonus` / `OverloadRepairBonus` / `OverloadDurationBonus` / `OverloadSpeedFactorBonus` | double |

**Propulsion**

| Member | Type |
|---|---|
| `MaxVelocityBonus` / `MaxVelocityPenalty` | double |
| `MassAddition` / `Thrust` / `VelocityModifier` | double |

**Other**

| Member | Type |
|---|---|
| `HeatDamage` / `ChargeRate` | double |
| `DefaultEffectName` / `DefaultEffectDescription` | string |
| `MaxNeutralizationRange` / `EnergyDestabilizationRange` / `EnergyNeutralized` / `EnergyDestabilizationAmount` | double |
| `EnergyTransferAmount` / `PowerTransferAmount` / `TransferRange` / `PowerTransferRange` / `ShieldTransferRange` | double |
| `MaxTractorVelocity` | double |
| `WarpScrambleStrength` | int |
| `AccessDifficultyBonus` | double |
| `ScanResolutionBonus` / `SensorRecalibrationTime` | double |

**Ship/Fitting Bonuses**

| Member | Type |
|---|---|
| `StructureHPBonus` / `CargoCapacityBonus` / `CapacitorBonus` / `PowergridBonus` / `CPUOutputBonus` / `CPUPenaltyPercent` | double |
| `CapacitorRechargeRateBonus` / `ShieldRechargeRateBonus` | double |

**Methods**

| Method | Description |
|---|---|
| `Activate` / `Activate[targetID]` | Activate the module. With no arg, activates on the module's implicit target (required for offensive modules — active target must be set). With a targetID, activates against that specific entity. Only works in space; does not work for grouped or passive+hidden modules. |
| `Deactivate` | Deactivate. Only works in space. If the module is waiting for an active target, a Deactivate call triggers an OnClick fallback. |
| `Click` | Click — same as Activate for target-requiring modules. |
| `Reload` | Reload active charge. **Note:** source marks this as "not implemented" and it returns false with a printf. Use `ReloadAll` or `ChangeAmmo` instead. |
| `ReloadAll` | Reload all modules of the same type. |
| `ChangeAmmo[itemID]` | Load ammo. Takes the itemID of a charge stack to load. A second `quantity` argument was removed in 2018 and is no longer honored — the stack's full quantity is used. **Do NOT use the zero-arg form.** The 2011 changelog documented `ChangeAmmo` with no arguments as a "reload current ammo type" shortcut, but source now rejects this with `"You must specify the item id to load"` and `return false;`. Use `Reload` (NOTE: also stubbed non-functional, see above) or `ReloadAll` for same-ammo reload. |
| `SetAutoReloadOn` / `SetAutoReloadOff` | Auto-reload toggle. |
| `PutOnline` / `PutOffline` | Online/offline control. |
| `GetAvailableAmmo[index:item]` | List loadable ammo. |
| `ToggleOverload` | Toggle overload on the module. |
| `SetManualOn` / `SetManualOff` | Manual override (for ganged module control). |
| `UnloadToCargo` | Unload current charge to cargo. |
| `Repair` / `CancelRepair` | Module repair control (nanite paste). |

Note: there is NO `Toggle` method on module — use `Activate`/`Deactivate` or inspect `IsActive` first.

### modulecharge

| Member | Type |
|---|---|
| `ID` | int64 |
| `Type` / `TypeID` | string / int |
| `Group` / `GroupID` | string / int |
| `Category` / `CategoryID` | string / int |
| `Location` / `LocationID` / `Slot` / `SlotID` | string / int64 / string / int |
| `Quantity` | int |
| `MaxFlightTime` / `MaxVelocity` | double |
| `Volume` | double |
| `ChargeSize` | int |

Deep-dive: [15_Combat_Automation.md](ISXEVE%20Scripting%20Guide/15_Combat_Automation.md), [16_Mining_And_Hauling.md](ISXEVE%20Scripting%20Guide/16_Mining_And_Hauling.md).

---

## Item and Inventory Datatypes

### item

Inherits from [iteminfo](#iteminfo).

**Identity and Location**

| Member | Type | Notes |
|---|---|---|
| `ID` | int64 | Unique item ID. |
| `Name` | string | Custom name or type name. |
| `Category` / `CategoryID` | string / int | Inventory category. |
| `OwnerID` | int64 | Owner CharID. |
| `Location` / `LocationID` | string / int64 | Container location. |
| `Quantity` | int | Stack quantity. |
| `Slot` / `SlotID` | string / int | Slot name/ID (for fitted items). |
| `IsRepackable` | bool | Can be repackaged. |
| `CargoCapacity` / `UsedCargoCapacity` | double | For containers/ships, requires cargo opened at least once. |

**Missile/Charge Stats**

| Member | Type |
|---|---|
| `MaxFlightTime` / `MaxVelocity` | double |
| `ExplosionRadius` / `ExplosionVelocity` | double |
| `SignatureRadiusBonus` | double |
| `EMDamage` / `KineticDamage` / `ThermalDamage` / `ExplosiveDamage` | double |
| `FallofMultiplier` / `TrackingSpeedMultiplier` | double |
| `MetaLevel` | int |

**Ship-as-Item Stats**

| Member | Type | Notes |
|---|---|---|
| `IsInsured` | bool | Ship insured. |
| `InsuranceLevel` | string | Insurance tier. |
| `BookmarkID` | int64 | Bookmark ID (for bookmark items). |

**Methods**

| Method | Description |
|---|---|
| `MoveTo[toID, toDestination, quantity, folder]` | Move item. Destination constants listed in [Canonical Slot Names and Destinations](#canonical-slot-names-and-destinations). |
| `Jettison` | Jettison from cargo. |
| `LaunchDrones` | Launch drones (from cargo/drone bay). |
| `LaunchForSelf` | Launch deployable for self. |
| `LaunchForCorp[ignoreWarnings]` | Launch deployable for corp. |
| `MakeActive` | Board ship (when item is a ship in hangar). |
| `LeaveShip` | Leave current ship (board this one). |
| `Repackage` | Repackage the item. |
| `Compress` | Open compression window (ore). |
| `AssembleShip` / `AssembleContainer` | Assemble packaged items. |
| `Refine` / `Reprocess` | Refining/reprocessing flow. |
| `TrainSkill` | Start training a skill book. |
| `InjectSkill` | Inject a skill injector. |
| `AddToSellOrder` | Open the sell-order flow (replaces the old `PlaceSellOrder`). |
| `Open` | Open as container/ship. |
| `ConsumeBooster` | Consume a booster item. |
| `PluginImplant` | Insert an implant. |
| `ApplyPilotLicense` | Apply a Pilot's License Extension. |
| `GetInsuranceQuotes[index:iteminfolist]` | Get insurance quotes. |
| `Insure[level, cost]` | Purchase insurance. |
| `GetContrabandFactions[index:iteminfolist]` | List factions that consider the item contraband. |
| `GetRepairQuote` | Open the repair-shop window for this item. |
| `UseAbyssalFilament` | Activate an abyssal filament. |

Note: `FitToActiveShip` was REMOVED in Jan 2025. Use `EVEWindow[Fitting].Slot[slotName]:FitItem[itemID]`.

### iteminfo

Static type info (looked up via `EVE.ItemInfo[typeID]` or inherited by `item`).

| Member | Type |
|---|---|
| `Name` / `Type` | string (alias) |
| `TypeID` | int |
| `Group` / `GroupID` | string / int |
| `Capacity` / `Volume` / `Radius` | double |
| `BasePrice` | double |
| `PortionSize` | int |
| `MarketGroupID` | int |
| `RaceID` | int |
| `Description` | string |
| `IsContraband` | bool |
| `GraphicID` | int |
| `ChargeSize` | int |
| `RangeBonus` | float |
| `ShieldRadius` | int |

### iteminfolist

Inherits from [iteminfo](#iteminfo). Adds `ID` (int — same as TypeID), `TypeID` (int), `Quantity` (int64). Used by several `Get...Results` methods for scanned/listed items with stack counts.

Deep-dive: [03_API_Reference.md — Inventory and Cargo](ISXEVE%20Scripting%20Guide/03_API_Reference.md#inventory-and-cargo-systems).

---

## Entity and Space Datatypes

### entity

Represents anything on the current grid: ships, NPCs, stations, gates, asteroids, wrecks, containers, drones, structures. Returned by `Entity[...]` TLO, `Me.ActiveTarget`, `EVE:QueryEntities`, many chained members.

**Identity**

| Member | Type |
|---|---|
| `ID` | int64 |
| `Name` | string |
| `Type` / `TypeID` | string / int |
| `Group` / `GroupID` | string / int |
| `Category` / `CategoryID` | string / int |

**Ownership**

| Member | Type |
|---|---|
| `Owner` / `OwnerID` | string / int64 |
| `CharID` | int64 |
| `Corp` / `Alliance` / `AllianceID` / `AllianceTicker` | string / string / int / string |
| `Security` | float |

**Position and Motion**

| Member | Type |
|---|---|
| `X` / `Y` / `Z` | double |
| `vX` / `vY` / `vZ` | double |
| `Distance` | double |
| `Distance2` | double (squared distance — use for comparisons; 3-5x faster) |
| `Velocity` / `MaxVelocity` | double |
| `AngularVelocity` / `RadialVelocity` / `TransversalVelocity` | double |
| `Pitch` / `Roll` / `Yaw` | double |
| `Mass` / `Radius` | double |

**Combat State**

| Member | Type | Notes |
|---|---|---|
| `ShieldPct` / `ArmorPct` / `StructurePct` | double | Percent damage values (0-100). |
| `Bounty` | double | NPC bounty. |
| `IsActiveTarget` | bool | Is YOUR active target. |
| `IsLockedTarget` | bool | Is one of your locked targets. |
| `IsTargetingMe` / `BeingTargeted` | bool | Being-targeted predicates. |
| `IsWarpScrambled` | bool | This entity is scrambled. |
| `IsWarpScramblingMe` | bool | This entity is scrambling you. |
| `IsJammingMe` / `IsTargetJammingMe` | bool | ECM predicates. |

**Type Predicates**

| Member | Type |
|---|---|
| `IsNPC` / `IsPC` | bool |
| `IsCelestial` / `IsGlobal` / `IsMassive` / `IsInteractive` | bool |
| `IsMoribund` | bool (dying) |
| `IsCloaked` | bool |
| `IsFleetMember` | bool |
| `IsOwnedByCorpMember` / `IsOwnedByAllianceMember` | bool |
| `IsPOS` | bool |
| `POSState` | string |
| `IsDockable` | bool (citadels/structures) |

**Cargo**

| Member | Type | Notes |
|---|---|---|
| `CargoCapacity` / `UsedCargoCapacity` | double | Requires the cargo to have been opened this session. |
| `HasOreHold` | bool | |
| `CargoWindow` | [evewindow](#evewindow) | Container cargo window, if open. |
| `LootWindow` | [evewindow](#evewindow) | Wreck loot window. |

**Wrecks**

| Member | Type |
|---|---|
| `IsAbandoned` / `HaveLootRights` / `IsWreckEmpty` / `IsWreckViewed` | bool |
| `WreckID` | int64 |

**Scanner Results**

| Member | Type |
|---|---|
| `HasCargoScannerResults` / `HasShipScannerResults` | bool |
| `ShipScannerCapacitorCapacity` / `ShipScannerCapacitorCharge` | double |
| `SurveyScannerOreQuantity` | int64 |

**Fleet / Formation**

| Member | Type |
|---|---|
| `FleetTag` | string |
| `FormationID` | int |
| `Following` | [entity](#entity) (entity this one is following) |
| `FollowRange` | double |
| `Approaching` | [entity](#entity) |
| `Mode` | int (movement mode flags) |

**Wormhole-Specific**

Available on entities that are wormholes. See [entitywormhole](#entitywormhole) for the specialized subtype.

| Member | Type |
|---|---|
| `WormHoleAge` | int (-1 if not a wormhole) |
| `WormHoleSize` | float (-10.0 on error) |
| `WormHoleClass` | int (-10 on error) |

**Type Casts**

| Member | Type |
|---|---|
| `ToActiveDrone` | [activedrone](#activedrone) |
| `ToAttacker` | [attacker](#attacker) |
| `ToFleetMember` | [fleetmember](#fleetmember) |
| `ToJammer` | [jammer](#jammer) |
| `ToWormhole` | [entitywormhole](#entitywormhole) |

**Methods — Basic Interaction**

| Method | Description |
|---|---|
| `Set[entityID]` | Reassign a variable reference to another entity. |
| `Activate` | Activate (acceleration gates, etc.). |
| `Dock` | Dock at station/citadel. Since Crucible, will auto-warp/approach as needed. |
| `Jump` | Jump through a stargate. |
| `Open` | Open as container/wreck/etc. |
| `AccessCustomsOffice` | Open the customs office UI (orbital only). |

**Methods — Navigation**

| Method | Description |
|---|---|
| `Approach` | Approach entity. With no args, defaults to 50m hold distance. Source has an `argc > 1` guard on the distance parse, so a single-arg `Approach[1000]` actually ignores the 1000 and still uses the 50m default — prefer `KeepAtRange[distance]` when you need a specific hold distance. |
| `AlignTo` | Align to entity. |
| `Orbit` / `Orbit[distance]` | Orbit at distance in meters. Defaults to 5000m if distance omitted. |
| `KeepAtRange` / `KeepAtRange[distance]` | Keep at range in meters. Defaults to 1000m if distance omitted. |
| `WarpTo` / `WarpTo[distance]` | Warp to entity at distance in meters. Defaults to 0m if distance omitted. Source only accepts one optional arg — there is NO second "fleet warp" argument on `WarpTo`; use `WarpFleetTo` for fleet warps. |
| `WarpFleetTo` / `WarpFleetTo[distance]` | Fleet warp to entity (FC/WC/SC only). Defaults to 0m if distance omitted. |

Note: method name is `WarpFleetTo`, not `FleetWarpTo`.

**Methods — Targeting**

| Method | Description |
|---|---|
| `LockTarget` | Lock as a target. |
| `UnlockTarget` | Unlock. |
| `MakeActiveTarget` | Make the active target (must be locked first). |
| `SetAsSelectedItem` | Set as the overview-selected item. |

**Methods — Scanning**

| Method | Description |
|---|---|
| `GetCargoScannerResults[index:iteminfolist]` | Populate with cargo-scan results. |
| `GetShipScannerResults[index:iteminfolist]` | Populate with ship-scan results. |

**Methods — Drones (when the entity is one of your drones)**

| Method | Description |
|---|---|
| `GetActiveDrones[index:activedrone]` | Populate with drones controlled by this entity. |
| `Mine` / `MineRepeatedly` | Command drones to mine this entity. |
| `EngageMyTarget` | Order drones to engage. |
| `ReturnAndOrbit` / `ReturnToDroneBay` | Return commands. |
| `ReturnFighterControl` | Return fighter control. |
| `ScoopToCargoBay` / `ScoopToCargoHold` (alias) | Scoop to cargo. |
| `ScoopToDroneBay` | Scoop to drone bay. |
| `ScoopToShipMaintenanceBay` | Scoop to ship maintenance bay. |
| `Abandon` / `AbandonAll` / `AbandonDrone` | Abandon drone(s). |
| `DroneAssist[charID]` / `DroneGuard[charID]` | Drone support commands. |
| `DelegateFighterControl[charID]` | Delegate fighter. |

**Methods — Misc**

| Method | Description |
|---|---|
| `CreateBookmark[label, notes, folder, expiry]` | Bookmark at this entity's location. |
| `SetFleetTag[tag]` | Set a fleet tag on the entity. |
| `SetName[name]` | Rename (only for entities that support it, e.g. containers). |
| `MarkWreckViewed` | Mark wreck as viewed. |

Deep-dive: [03_API_Reference.md — Entity System](ISXEVE%20Scripting%20Guide/03_API_Reference.md#entity-system-and-targeting).

### entitywormhole

Inherits from [entity](#entity). Obtained via `entity.ToWormhole`.

| Member | Type | Notes |
|---|---|---|
| `Age` | int | Wormhole age. -10 on error. |
| `Size` | float | Wormhole size. -10.0 on error. |
| `Class` | int | Wormhole class. -10 on error. |

**Methods:** `EnterWormhole`.

### entityplayerstructure

Inherits from [entity](#entity). Represents POS modules and player-structures.

| Member | Type |
|---|---|
| `Anchored` / `CanAnchorAt` / `CanUnanchor` | bool |
| `CanAssumeControl` | bool |
| `CanOffline` / `CanOnline` | bool |
| `ControllerID` / `ControllerName` | int64 / string |
| `CurrentTargetID` | int64 |
| `Online` / `Orphaned` | bool |
| `State` | string |
| `ToTower` | [entity](#entity) |

**Methods:** `Anchor`, `AssumeControl`, `ReleaseControl`, `UnlockTarget`.

### attacker

Inherits from [entity](#entity). Returned by `character:GetAttackers`.

| Member | Type |
|---|---|
| `ID` | int64 |
| `IsCurrentlyAttacking` | bool |
| `ToJammer` | [jammer](#jammer) |

**Methods:** `GetAttacks[index:attack]`.

### attack

| Member | Type |
|---|---|
| `ID` | int64 |
| `Name` | string |
| `TimeStarted` | [evetime](#evetime) |

### jammer

Inherits from [attacker](#attacker). Returned by `character:GetJammers`.

| Member | Type |
|---|---|
| `ID` | int64 |

**Methods:** `GetJams[index:???]` — populate index with jam events from this jammer.

### overview

Access: `Overview` TLO.

| Member | Type |
|---|---|
| `SelectedItem` | [entity](#entity) |

**Methods:** `ClearSelectedItem`.

---

## Station and Structure Datatypes

### station

Access: `Me.Station` (when docked at an NPC station), `EVE.Station[id]`.

| Member | Type |
|---|---|
| `ID` | int64 |
| `Name` | string |
| `TypeID` / `Type` | int / string |
| `OwnerID` / `Owner` | int64 / string |
| `OwnerTypeID` / `OwnerType` | int / string |
| `SolarSystem` | [solarsystem](#solarsystem) |
| `Cargo[#]` / `Cargo[name]` | [item](#item) (station-item access, like ship.Cargo) |

**Methods**

| Method | Description |
|---|---|
| `GetHangarItems[index:item]` | Personal hangar items (must be docked). |
| `GetHangarShips[index:item]` | Personal hangar ships (must be docked). |
| `GetCorpHangarItems[index:item, division]` | Corp hangar items (must be docked). |
| `GetCorpHangarShips[index:item, division]` | Corp hangar ships (must be docked). |
| `GetRepairableItems[index:item]` | Items that can be repaired here. |
| `SetDestination` / `AddWaypoint` / `ClearWaypoint` | Route management via station ID. |
| `OpenFitting` | Open the fitting window. |

Note: there is NO `Undock` method on station. Use `EVE:Execute[CmdExitStation]`.

### structure

Citadels and upwell structures. Inherits from [station](#station).

| Member | Type |
|---|---|
| `CanBoard` | bool |
| `MyCorpHasOffice` | bool |

**Methods:** `Board`.

Deep-dive: [03_API_Reference.md — Station Object](ISXEVE%20Scripting%20Guide/03_API_Reference.md#station-object).

---

## Interstellar and Bookmark Datatypes

### interstellar

Base for universe objects. Obtained via the `Universe[id]` / `Universe[name]` TLO.

| Member | Type |
|---|---|
| `ID` | int64 |
| `Name` | string |

### solarsystem

Inherits from [interstellar](#interstellar).

| Member | Type |
|---|---|
| `Security` | float |
| `Faction` / `FactionID` | string / int |
| `Constellation` | [constellation](#constellation) |
| `Region` | [region](#region) |
| `JumpsTo[solarSystemID]` | int |
| `JumpsTo[stationID]` | int |

**Methods**

| Method | Description |
|---|---|
| `SetDestination` / `AddWaypoint` / `ClearWaypoint` | Route management. |
| `GetNumPlanetsByType[collection:int]` | Populate collection where key=PlanetType name/ID, value=count. |
| `GetPlanetIDs[index:int]` | List planet IDs in the system. |

### constellation

Inherits from [interstellar](#interstellar). Adds `Region` ([region](#region)).

### region

Inherits from [interstellar](#interstellar).

### planet

Inherits from [interstellar](#interstellar).

| Member | Type |
|---|---|
| `Radius` | int |
| `Type` / `TypeID` | string / int |
| `SolarSystem` | [solarsystem](#solarsystem) |

**Methods:** `GetOrbitalCustomsOffices[index:int64]` (populate with customs-office entity IDs; only works in space).

### bookmark

Obtained via `EVE.Bookmark[id]`, `EVE.Bookmark[label]`, or `EVE:GetBookmarks`.

**Identity**

| Member | Type |
|---|---|
| `ID` | int64 |
| `Label` | string |
| `Note` | string |
| `Type` / `TypeID` | string / int |

**Ownership**

| Member | Type |
|---|---|
| `OwnerID` / `CreatorID` | int64 |
| `FolderID` | int64 |
| `AgentID` | int64 (for agent-provided bookmarks) |

**Location**

| Member | Type |
|---|---|
| `SolarSystemID` | int64 |
| `LocationID` / `LocationType` / `LocationNumber` | int64 / string / int |
| `ItemID` | int64 |
| `X` / `Y` / `Z` | double |
| `ToEntity` | [entity](#entity) (if on current grid) |
| `DeadSpace` | bool |

**Timestamps**

| Member | Type |
|---|---|
| `Created` | int64 |
| `DateCreated` / `TimeCreated` | string |

**Navigation**

| Member | Type |
|---|---|
| `JumpsTo` | int |
| `Distance` | double |

**Methods**

| Method | Description |
|---|---|
| `WarpTo` / `WarpTo[distance]` | Warp to bookmark. Defaults to 0m if distance omitted. |
| `WarpFleetTo` / `WarpFleetTo[distance]` | Fleet warp (FC/WC/SC only). Defaults to 0m. |
| `SetDestination` / `AddWaypoint` / `ClearWaypoint` | Route management. |
| `AlignTo` | Align. Takes no arguments. |
| `Approach` | Approach. Takes no arguments. |
| `Remove` | Delete the bookmark. |

Deep-dive: [03_API_Reference.md — Movement and Navigation](ISXEVE%20Scripting%20Guide/03_API_Reference.md#movement-navigation-and-autopilot).

---

## Window Datatypes

### evewindow

Base type for all EVE windows.

| Member | Type |
|---|---|
| `Name` | string |
| `Caption` | string |
| `Text` / `HTML` | string |
| `Minimized` | bool |
| `X` / `Y` / `Width` / `Height` | int |
| `NumButtons` | int |
| `Button[#]` | [eveuibutton](#eveuibutton) (1-based up to NumButtons) |
| `Button[name]` | [eveuibutton](#eveuibutton) — matches against the button widget's internal `name` attribute, which for many buttons IS the visible text but not always. The match is case-insensitive. |

**Methods**

| Method | Description |
|---|---|
| `Close` / `Minimize` / `Maximize` | Window actions. |
| `StackAll` | Stack items (loot/inventory windows). |
| `LootAll` | Loot all (loot windows). |
| `ClickButtonOK` / `ClickButtonCancel` / `ClickButtonYes` / `ClickButtonNo` / `ClickButtonClose` | Modal-button helpers. |

### eveinvwindow

Inventory window. Inherits from [evewindow](#evewindow).

| Member | Type | Notes |
|---|---|---|
| `ActiveChild` | [eveinvchildwindow](#eveinvchildwindow) | Currently active tab. |
| `ChildWindow[name]` | [eveinvchildwindow](#eveinvchildwindow) | By name (e.g. `ShipCargo`, `StationItems`, `StationCorpHangars`, `Folder6`). |
| `ChildWindow[id]` | [eveinvchildwindow](#eveinvchildwindow) | By ID (e.g. `${Me.StationID}`). |
| `ChildWindow[id, name]` / `ChildWindow[name, location]` / `ChildWindow[id, name, location]` | [eveinvchildwindow](#eveinvchildwindow) | Disambiguated lookup. |
| `IsInRange` | bool | |
| `HasCapacity` | bool | |
| `ItemID` | int64 | Container item ID. |
| `LocationFlag` / `LocationFlagID` | string / int | |
| `Capacity` / `UsedCapacity` | double | -1 on error, -2 if not ready. |

**Methods:** `GetChildren[index:eveinvchildwindow]`.

Note: the correct member is `ChildWindow[...]`, NOT `Child[...]`. Older examples using `.Child[ShipCargo]` will silently fail.

### eveinvchildwindow

A single tab/child in the inventory window.

| Member | Type | Notes |
|---|---|---|
| `Name` | string | |
| `Capacity` / `UsedCapacity` | double | -1 on error, -2 if not made active yet. |
| `LocationFlag` / `LocationFlagID` | string / int | |
| `IsInRange` / `HasCapacity` | bool | |
| `ItemID` | int64 | |

**Methods**

| Method | Description |
|---|---|
| `MakeActive` | Switch to this child tab. |
| `OpenAsNewWindow` | Break out into separate window. |
| `GetItems[index:item]` | List items in this child. |
| `StackAll` | Stack items in this child. |
| `MoveTo[destID, destFlagID]` | Batch move all items. |

### evefittingwindow

Inherits from [evewindow](#evewindow).

| Member | Type | Notes |
|---|---|---|
| `CPU` | string | Now returns the displayed text, e.g. `"78.4/80.4"` — parse it. |
| `Power` | string | Same. |
| `Calibration` | string | Same. |
| `IsShipSimulated` | bool | |
| `Slot[slotName]` | [fittingslot](#fittingslot) | E.g. `HiSlot0`, `MedSlot3`, `LoSlot1`, `RigSlot0..RigSlot7`. |
| `Slot[id]` | [fittingslot](#fittingslot) | By numeric slot ID. |

**Methods:** `GetSlots[index:fittingslot]`, `StripFitting`.

### evesellitemswindow

Inherits from [evewindow](#evewindow).

| Member | Type |
|---|---|
| `Duration` | int (days) |
| `RemainingOrders` | int |
| `BrokersFee` / `SalesTax` / `TotalAmount` | string |
| `NumItems` | int |
| `Item[#]` | [sellitem](#sellitem) (1-based) |

**Methods:** `SetDuration[days]` (0/1/3/7/14/30/90), `Sell`.

### evemarketactionwindow

Market buy-order window. Inherits from [evewindow](#evewindow).

| Member | Type |
|---|---|
| `BidPrice` | [eveuisinglelineedit](#eveuisinglelineedit) |
| `BidPricePercentageComparison` | [eveuilabel](#eveuilabel) |
| `RegionalAverage` | double |
| `BestRegional` / `BestMatchable` | [eveuilabel](#eveuilabel) |
| `Quantity` / `QuantityMin` | [eveuisinglelineedit](#eveuisinglelineedit) |
| `Duration` / `Range` | [eveuicombo](#eveuicombo) |
| `Fee` | [eveuisinglelineedit](#eveuisinglelineedit) |
| `Total` | [eveuilabel](#eveuilabel) |
| `IsReady` | bool |

**Methods:** `Buy`, `Close`.

### everepairshopwindow

Inherits from [evewindow](#evewindow).

| Member | Type |
|---|---|
| `AverageDamage` | string |
| `TotalCost` | string |

**Methods:** `RepairAll`.

### eveagentdialogwindow

Inherits from [evewindow](#evewindow).

| Member | Type |
|---|---|
| `BriefingHTML` / `ObjectivesHTML` | string |

### evemessageboxwindow

Inherits from [evewindow](#evewindow). Returned by `EVEWindow[MessageBox]` when a message box is open.

### evedirectionalscannerwindow

Inherits from [evewindow](#evewindow).

| Member | Type | Notes |
|---|---|---|
| `Range` | float | Scanner range (always in AU). |
| `Angle` | double | Scanner angle. |
| `IsScanning` | bool | Scan-in-progress. |

**Methods:** `GetScanResults[index:directionalscannerresult]`.

### evecustomsofficewindow

Inherits from [evewindow](#evewindow). Returned by `EVEWindow[byName,PlanetaryImportExportUI]`.

| Member | Type |
|---|---|
| `TaxRate` | float |
| `HeaderTitle` | string |

---

## UI Element Datatypes

### eveuibutton

| Member | Type |
|---|---|
| `Name` | string |
| `Text` | string |

**Methods:** `Press`.

### eveuilabel

| Member | Type |
|---|---|
| `Text` | string |

### eveuisinglelineedit

| Member | Type |
|---|---|
| `Value` | string |

**Methods:** `SetValue[value]`.

### eveuicombo

| Member | Type |
|---|---|
| `Index` / `Key` / `Value` | string |

**Methods:** `SelectByIndex[#]`, `SetectByValue[#]`, `SelectByLabel[label]`.

Note: the second method name is `SetectByValue` in source — a longstanding typo in the registered method name. The 2009 changelog documents it as `SelectByValue`, but calling `SelectByValue` will not resolve. Scripts must use the misspelled form `SetectByValue` to actually select by value.

### fittingslot

| Member | Type |
|---|---|
| `Name` | string (e.g. `HiSlot0`) |
| `ID` | int |
| `IsEmpty` / `IsOnline` / `ContainsCharge` | bool |
| `Module` | [module](#module) |

**Methods:** `FitItem[itemID]`, `Unfit`, `UnfitCharge`, `PutOnline`, `PutOffline`.

---

## Market Datatypes

### marketorder

Market order visible to the player.

| Member | Type |
|---|---|
| `ID` | int64 |
| `Name` | string |
| `TypeID` | int |
| `IsSellOrder` / `IsBuyOrder` | bool |
| `Price` | double |
| `InitialQuantity` / `QuantityRemaining` / `MinQuantityToBuy` | int |
| `TimeStampWhenIssued` | int64 |
| `DateWhenIssued` / `TimeWhenIssued` | string |
| `Duration` | int |
| `StationID` / `Station` | int64 / string |
| `RegionID` / `Region` | int64 / string |
| `SolarSystemID` | int64 |
| `SolarSystem` | [solarsystem](#solarsystem) (returns solarsystem object since 2023) |
| `Range` | int |
| `Jumps` | int |

### myorder

Your own market order.

Same members as [marketorder](#marketorder), plus:

| Member | Type |
|---|---|
| `IsContraband` | bool |
| `IsCorp` | bool |

**Methods:** `Cancel`, `Modify[price, quantity, duration]`.

### sellitem

Item in the sell-items window.

| Member | Type |
|---|---|
| `Name` | string |
| `ItemID` | int64 |
| `Quantity` | string |
| `AskPrice` | string |
| `AveragePrice` / `BestPrice` / `BestVolumeRemaining` | double |
| `BestJumps` | int |
| `SalesTax` / `TotalAmount` | double |

**Methods:** `SetAskPrice[price]`, `SetQuantity[quantity]`.

---

## Scanner Datatypes

### scanners

Container. Access via `MyShip.Scanners`.

| Member | Type | Notes |
|---|---|---|
| `System` | [systemscanner](#systemscanner) | Probe scanner / sensor overlay. |
| `Survey[moduleSlotOrID]` | [surveyscanner](#surveyscanner) | Requires a fitted survey scanner; argument is a slot name or module ID, same form as `MyShip.Module[]`. |
| `Ship[moduleSlotOrID]` | [shipscanner](#shipscanner) | Requires a fitted ship scanner. |
| `Cargo[moduleSlotOrID]` | [cargoscanner](#cargoscanner) | Requires a fitted cargo scanner. |
| `Directional` | [directionalscanner](#directionalscanner) | Directional scan (D-scan). |

### directionalscanner

**Methods:** `StartScan[angle, range]`, `GetScanResults[index:directionalscannerresult]`.

### systemscanner

| Member | Type |
|---|---|
| `IsSensorOverlayActive` | bool |

**Methods**

- `EnableSensorOverlay` / `DisableSensorOverlay`
- `GetAnomalies[index:systemanomaly]`
- `GetSignatures[index:systemsignature]`

### surveyscanner

**Methods:** `StartScan`, `ClearSurveyResults`. Data arrives via `EVE_OnSurveyScanData` event.

### shipscanner

**Methods:** `StartScan[entityID]`. Results via `entity.GetShipScannerResults`.

### cargoscanner

**Methods:** `StartScan[entityID]`. Results via `entity.GetCargoScannerResults`.

### directionalscannerresult

| Member | Type |
|---|---|
| `ID` | int64 |
| `Name` | string |
| `Group` / `GroupID` | string / int |
| `Type` / `TypeID` | string / int |
| `ToEntity` | [entity](#entity) (only if on grid) |

### systemsignature

Wormhole, data/relic site, gas site, etc.

| Member | Type |
|---|---|
| `ID` | string (e.g. `"ABC-123"`) |
| `Name` | string |
| `Group` / `GroupID` | string / int |
| `SignalStrength` / `Deviation` | double |
| `Difficulty` | string |
| `IsWarpable` | bool |
| `X` / `Y` / `Z` | double |
| `ToEntity` | [entity](#entity) |
| `ToItem` | [item](#item) |

**Methods:** `WarpTo[distance]`, `AlignTo`, `Approach[distance]`.

### systemanomaly

Combat/ore/ice site.

| Member | Type |
|---|---|
| `ID` | int64 |
| `Name` | string |
| `Group` / `GroupID` | string / int |
| `DungeonID` / `DungeonName` | int / string |
| `Faction` / `FactionID` | string / int |
| `Difficulty` | string |
| `ScanStrength` / `SignalStrength` | double |
| `IsWarpable` | bool |
| `X` / `Y` / `Z` | double |

**Methods:** `WarpTo[distance]`, `AlignTo`, `Approach[distance]`.

---

## Agent and Mission Datatypes

### eveagent

| Member | Type |
|---|---|
| `ID` | int64 |
| `Name` | string |
| `TypeID` / `AgentTypeID` / `AgentTypeName` | int / int / string |
| `Index` | int |
| `Gender` | string |
| `Division` / `DivisionID` | string / int |
| `CorporationID` / `FactionID` | int / int |
| `Level` | int |
| `StandingTo` / `StandingToCorp` / `StandingToFaction` | double |
| `EffectiveStanding` | double |
| `Station` / `StationID` | string / int64 |
| `Solarsystem` | [solarsystem](#solarsystem) |
| `IsLocatorAgent` | bool |

**Methods:** `StartConversation`.

### agentmission

| Member | Type |
|---|---|
| `ID` | int |
| `Name` / `Type` | string |
| `AgentID` | int64 |
| `State` | string |
| `Expires` | int64 (seconds) |
| `ExpirationTime` | [evetime](#evetime) |
| `ImportantMission` | bool |
| `RemoteOfferable` / `RemoteCompletable` | bool |

**Methods:** `GetBookmarks[index:bookmark]`.

---

## Skill Datatypes

### skill

| Member | Type |
|---|---|
| `Name` | string |
| `TrainedLevel` | int (renamed from `SkillLevel` in 2023) |
| `EffectiveLevel` | int (2023) |
| `Points` | int (renamed from `SkillPoints` in 2023) |
| `TimeToTrain` | int64 (seconds to next level) |
| `TrainingTimeMultiplier` | double |

**Methods:** `StartTraining`, `AddToQueue[level]`.

### queuedskill

| Member | Type |
|---|---|
| `Name` | string |
| `TrainingTo` | int |
| `ToSkill` | [skill](#skill) |
| `StartTime` / `EndTime` | [evetime](#evetime) |
| `QueuePosition` | int (0-based) |
| `StartSkillPoints` / `DestinationSkillPoints` | int |

---

## Chat Datatypes

### evechat

Access: `Chat` TLO.

| Member | Type |
|---|---|
| `ChannelCount` | int |

**Methods:** `GetChannels[index:chatchannel]`.

### chatchannel

Access: `Chat[id]` or `Chat[name]`.

| Member | Type |
|---|---|
| `Name` | string |
| `ID` | int64 |
| `PilotCount` | int |
| `Category` | string |
| `MOTD` | string |
| `LastActivityTime` | [evetime](#evetime) |

**Methods**

- `GetMembers[index:pilot]` — recent speakers for some channel types.
- `GetMessages[index:chatchannelmessage]` — up to 100 cached messages.

Note: `MarkAsRead`, `Echo`, `Send`, `NewMessageReceived`, `CanSpeak` were REMOVED in 2018 when EVE changed the chat system. Use LavishScript's own chat send/echo where applicable.

### chatchannelmessage

| Member | Type |
|---|---|
| `Author` | [pilot](#pilot) |
| `Message` | string |
| `Timestamp` | [evetime](#evetime) |

---

## Fleet Datatypes

### fleet

Access: `Me.Fleet`. Guard with `${Me.Fleet.ID(exists)}`.

| Member | Type | Notes |
|---|---|---|
| `ID` | int64 | Fleet ID. |
| `IsFleetCommander` | bool | You are the FC. There is NO `IsBoss` or `IsLeader` — this is the only FC predicate. |
| `Size` | int | Number of members. There is NO `MemberCount` — use `Size`. |
| `Invited` | bool | You have a pending invitation. |
| `InvitationText` | string | Invitation text (HTML). |
| `Member[charID]` | [fleetmember](#fleetmember) | Member by character ID (int64). Source parses argv[0] as int64 only — does NOT accept numeric 1-based index or character name strings despite the changelog's suggestion otherwise. To look up by name, use `Me.Fleet:GetMembers[index:fleetmember]` and filter by `fleetmember.Name`. |
| `IsMember[charID]` | bool | Check membership by CharID. Requires a CharID argument — there is NO zero-arg `IsMember`. To check if you are in a fleet, use `${Me.Fleet.ID(exists)}`. |
| `SquadName[squadID]` | string | Squad name (returns `"Squad <id>"` unless customized). |
| `SquadNameToID[name]` | int64 | Squad ID by name. |
| `WingName[wingID]` | string | Wing name. |
| `WingNameToID[name]` | int64 | Wing ID. |

**Methods — Invitation**

| Method | Description |
|---|---|
| `Invite[charID]` | Send invitation. Source only accepts a numeric charID via atoi — passing a name string will not work despite older changelog wording. |
| `AcceptInvite` / `RejectInvite` | Respond to pending invite. |
| `LeaveFleet` | Leave the fleet. |

**Methods — Structure**

| Method | Description |
|---|---|
| `CreateWing[name]` | Create a wing. |
| `DeleteWing[wingID]` | Delete a wing. |
| `ChangeWingName[wingID, name]` | Rename a wing (max 10 chars). |
| `CreateSquad[wingID]` | Create a squad in a wing. |
| `DeleteSquad[squadID]` | Delete a squad. |
| `ChangeSquadName[squadID, name]` | Rename a squad (max 10 chars). |

**Methods — Iteration**

| Method | Description |
|---|---|
| `GetMembers[index:fleetmember]` | All fleet members. |
| `GetWings[index:int64]` | Wing IDs. |
| `GetSquads[index:int64]` / `GetSquads[index:int64, wingID]` | All squad IDs, or squads in a specific wing. |

**Methods — Commands**

| Method | Description |
|---|---|
| `WarpFleetToMember[charID, optionalDistance]` | Warp the fleet to a member. |
| `Regroup` | Issue fleet regroup. |
| `SetIgnoreFleetWarp` / `SetTakesFleetWarp` | Configure your ship's fleet-warp behavior. |

**Methods — Broadcasts**

| Method | Description |
|---|---|
| `Broadcast_Target[entityID]` | Broadcast a target. Works. |
| `Broadcast_TravelTo[solarSystemID]` | Travel-to broadcast. Works. |
| `Broadcast_Location` | Broadcast your current location. Takes no arguments despite older changelog wording that hints at `[itemID]`. |
| `Broadcast_HoldPosition` / `Broadcast_InPosition` | Position broadcasts. No arguments. |
| `Broadcast_NeedBackup` / `Broadcast_EnemySpotted` | Combat broadcasts. No arguments. |
| `Broadcast_HealArmor` / `Broadcast_HealShield` / `Broadcast_HealCapacitor` | Heal broadcasts. No arguments. |
| `Broadcast_JumpBeacon` | Jump-to-beacon broadcast. Takes no arguments (the changelog's `[charID]` signature is not accepted; source calls the Python method with no parameters). |
| `Broadcast_AlignTo[entityID]` | BROKEN. Source has been stubbed out with an unconditional `return false;` since 2012 ("requires typeid as second param as of 4/24/12" — never fixed). The method is registered but non-functional. |
| `Broadcast_WarpTo[entityID]` | BROKEN. Same stub pattern as `Broadcast_AlignTo`. Non-functional since 2012. |
| `Broadcast_JumpTo[solarSystemID]` | BROKEN. Same stub pattern. Non-functional since 2012. |

For the three broken broadcasts, use `EVE:Execute[CmdSendBroadcast_*]` equivalents where they exist, or post directly to fleet chat as a workaround.

### fleetmember

Inherits from [pilot](#pilot) (and therefore [being](#being)).

| Member | Type |
|---|---|
| `Boosting` | bool |
| `HasActiveBeacon` | bool |
| `Job` / `JobID` | string / int |
| `Role` / `RoleID` | string / int |
| `SquadID` / `WingID` | int64 / int64 |
| `IsFleetCommander` / `IsWingCommander` / `IsSquadCommander` | bool |
| `ToEntity` | [entity](#entity) (on-grid only) |
| `ToPilot` | [pilot](#pilot) |

**Methods**

| Method | Description |
|---|---|
| `AddToWatchList` / `RemoveFromWatchList` | Watchlist management. |
| `Kick` | Kick from fleet. |
| `MakeLeader` | Promote to fleet commander. |
| `Move[wingID, squadID]` | Move within fleet. |
| `MoveToFleetCommander` / `MoveToWingCommander[wingID]` / `MoveToSquadCommander[wingID, squadID]` | Promotion helpers. |
| `SetBooster` | Set as booster. |
| `WarpFleetTo[entity, distance]` | Fleet warp to specified entity. |
| `WarpTo[entity, distance]` | Warp solo to entity. |

Deep-dive: [17_Fleet_Operations.md](ISXEVE%20Scripting%20Guide/17_Fleet_Operations.md).

---

## Drone Datatypes

### activedrone

Drone currently in space. Returned by `character:GetActiveDrones` or `entity:GetActiveDrones`, or cast from entity via `entity.ToActiveDrone`.

| Member | Type |
|---|---|
| `ID` | int64 |
| `Owner` | string |
| `Controller` | string |
| `Type` / `TypeID` | string / int |
| `State` | string |
| `Target` | [entity](#entity) |
| `ToEntity` | [entity](#entity) |

Note: drone-control methods (Mine/EngageMyTarget/etc.) live on the `entity` datatype, not here. Pass lists of int64 IDs to `EVE:LaunchDrones` / `EVE:DronesMine` / etc.

---

## Misc Datatypes

### charselect

Access: `CharSelect` TLO. Only valid at the character-selection screen (before character enters the game).

| Member | Type |
|---|---|
| `SelectedChar` | string |
| `SelectedCharID` | int |
| `CharExists[name]` | bool |

**Methods:** `ClickCharacter[name]`.

---

## Commands

Commands are first-class ISXEVE commands invoked directly from scripts (as opposed to `EVE:Execute[...]` which drives EVE's internal UI commands).

### Core Commands

#### UnloadISXEVE

**Syntax:** `UnloadISXEVE`

Safely unload ISXEVE. Logs to console when the unload completes.

#### DebugSpew

**Syntax:** `DebugSpew <message>`

Write a debug message to ISXEVE's debug log file.

```lavishscript
DebugSpew "Starting mining routine"
```

### Web Commands

All web commands are multi-threaded. The script does NOT block waiting for a response. Handle responses via the [isxGames_onHTTPResponse](#isxgames_onhttpresponse) event.

#### GetURL

**Syntax:** `GetURL "<url>" [content_type]`

HTTP GET. `https://` supported.

```lavishscript
GetURL "https://example.com/api/data.json" "application/json"
```

#### PostURL

**Syntax:** `PostURL "<url>" "<post_data>" [content_type]`

HTTP POST.

```lavishscript
PostURL "https://discord.com/api/webhooks/..." "{\"content\":\"hello\"}" "application/json"
```

#### PostURLFiles

**Syntax:** `PostURLFiles "<url>" "<file1>" ["<file2>" ...] [content_type]`

HTTP POST with file uploads (multiple files supported).

```lavishscript
PostURLFiles "https://example.com/upload" "C:/logs/combat.log"
```

---

## Events

Events fire when specific game actions occur. Attach handlers via LavishScript's event system:

```lavishscript
Event[EventName]:AttachAtom[HandlerFunction]
```

### EVE Events

#### EVE_OnChannelMessage

Fires when a chat message is received in any channel.

| Parameter | Type | Description |
|---|---|---|
| `ChannelID` | int | Channel ID (use with `Chat[id]`). |
| `SenderID` | int | Sender CharID. |
| `Message` | string | Message text. |

```lavishscript
atom(script) OnChannelMessage(int channelID, int senderID, string message)
{
    echo "[${Chat[${channelID}].Name}] ${Being[${senderID}].Name}: ${message}"
}

function main()
{
    while !${ISXEVE.IsReady}
        wait 10
    Event[EVE_OnChannelMessage]:AttachAtom[OnChannelMessage]
    ; ...
}
```

#### EVE_OnPilotJoinedChannel

DISABLED since March 21, 2018 — EVE's chat-system rewrite removed the underlying events. Kept for legacy detection only.

#### EVE_OnPilotLeftChannel

DISABLED since March 21, 2018 — see above.

#### EVE_OnSurveyScanData

Fires when survey-scanner results are received (added May 10, 2020).

| Parameter | Type | Description |
|---|---|---|
| `ResultCount` | int | Number of asteroids returned. |

After this event fires, ore-quantity data is available on nearby asteroid entities via `entity.SurveyScannerOreQuantity`.

### System Events

#### ISXEVE_onFrame

Fires each ISXEVE pulse. This is the canonical pulse event for ISXEVE scripts — use it instead of LavishScript's generic `onFrame` event so your handlers only run when ISXEVE has fully pulsed and refreshed its state. Takes no parameters.

```lavishscript
atom(script) OnISXEVEFrame()
{
    ; This fires every ISXEVE pulse; keep the body cheap.
    if ${Me.ToEntity(exists)} && ${MyShip.ShieldPct} < 30
        echo "Shield low: ${MyShip.ShieldPct.Int}%"
}

function main()
{
    while !${ISXEVE.IsReady}
        wait 10
    Event[ISXEVE_onFrame]:AttachAtom[OnISXEVEFrame]
    while TRUE
        waitframe
}
```

#### isxGames_onHTTPResponse

Fires when an HTTP response is received from `GetURL` / `PostURL` / `PostURLFiles`.

| Parameter | Type | Description |
|---|---|---|
| `Size` | int | Response size (bytes). |
| `URL` | string | Requested URL. |
| `IPAddress` | string | Remote IP. |
| `ResponseCode` | int | HTTP status code. |
| `TransferTime` | float | Seconds. |
| `ResponseText` | string | Full response body. |
| `ParsedBody` | string | Parsed body (server-dependent). |

```lavishscript
atom(script) OnHTTPResponse(int size, string url, string ip, int code, float time, string body, string parsed)
{
    if ${code.Equal[200]}
        echo "OK from ${url}: ${size} bytes in ${time}s"
    else
        echo "FAIL ${code} from ${url}"
}

Event[isxGames_onHTTPResponse]:AttachAtom[OnHTTPResponse]
```

---

## EVE:Execute Command Constants

Use `EVE:Execute[<Cmd>]` to drive EVE's built-in UI commands. The full list from the changelog:

**Movement:** `CmdAccelerate`, `CmdDecelerate`, `CmdStopShip`
**Modules:** `CmdActivateHighPowerSlot1..CmdActivateHighPowerSlot8`, `CmdActivateMediumPowerSlot1..CmdActivateMediumPowerSlot8`, `CmdActivateLowPowerSlot1..CmdActivateLowPowerSlot8`, `CmdReloadAmmo`
**Station:** `CmdExitStation`
**Windows:** `CmdCloseActiveWindow`, `CmdCloseAllWindows`, `CmdMinimizeActiveWindow`, `CmdMinimizeAllWindows`
**Autopilot/UI:** `CmdToggleAutopilot`, `CmdToggleMap`, `CmdToggleAudio`, `CmdToggleWindowed`
**Monitor/Dev:** `CmdResetMonitor`, `CmdQuitGame`
**Broadcasts:** `CmdSendBroadcast_EnemySpotted`, `CmdSendBroadcast_HealArmor`, `CmdSendBroadcast_HealCapacitor`, `CmdSendBroadcast_HealShield`, `CmdSendBroadcast_HoldPosition`, `CmdSendBroadcast_InPosition`, `CmdSendBroadcast_JumpBeacon`, `CmdSendBroadcast_Location`, `CmdSendBroadcast_NeedBackup`
**Window Openers (no Cmd prefix):** `OpenInventory`, `OpenAssets`, `OpenBrowser`, `OpenCalculator`, `OpenCargoHoldOfActiveShip`, `OpenChannels`, `OpenCharactersheet`, `OpenConfigMenu`, `OpenContracts`, `OpenCorporationPanel`, `OpenDroneBayOfActiveShip`, `OpenFactory`, `OpenFitting`, `OpenFpsMonitor`, `OpenHangarFloor`, `OpenHelp`, `OpenInbox`, `OpenJournal`, `OpenJukebox`, `OpenLog`, `OpenMapBrowser`, `OpenMarket`, `OpenMonitor`, `OpenMoonMining`, `OpenNotepad`, `OpenOverviewSettings`, `OpenPeopleAndPlaces`, `OpenScanner`, `OpenShipConfig`, `OpenShipHangar`, `OpenStationManagement`, `OpenTutorials`, `OpenWallet`
**Other:** `PrintScreen`

Note: window-opener commands do NOT have a `Cmd` prefix. `OpenInventory` is the idiomatic way to open the unified inventory window.

---

## Common Patterns and Idioms

### Main-Loop Skeleton

```lavishscript
function main()
{
    while !${ISXEVE.IsReady}
        wait 10

    while TRUE
    {
        call Pulse
        wait 10
    }
}

function Pulse()
{
    if !${Me.ToEntity(exists)}
        return
    ; ... bot logic
}
```

### Targeting an Entity by Query

```lavishscript
variable index:entity Targets
EVE:QueryEntities[Targets, "CategoryID = 11 && Distance < 50000"]

variable iterator it
Targets:GetIterator[it]
if ${it:First(exists)}
    do
    {
        if !${it.Value.IsLockedTarget}
            it.Value:LockTarget
    }
    while ${it:Next(exists)}
```

Categories: `6` = Ship, `11` = Entity (NPCs), `25` = Asteroid, `7` = Module, `18` = Drone. See [03_API_Reference.md — Entity Categories](ISXEVE%20Scripting%20Guide/03_API_Reference.md#entity-categories-and-groups) for the canonical table.

### Iterating Locked Targets

```lavishscript
variable index:entity Locked
Me:GetTargets[Locked]
echo "Locked: ${Me.TargetCount} / ${MyShip.MaxLockedTargets}"
```

### Iterating Modules by Slot Name

```lavishscript
variable int i
i:Set[0]
while ${i} < ${MyShip.HighSlots}
{
    if ${MyShip.Module[HiSlot${i}](exists)} && !${MyShip.Module[HiSlot${i}].IsActive}
        MyShip.Module[HiSlot${i}]:Activate
    i:Inc
}
```

### Iterating Modules via GetModules (Canonical)

```lavishscript
variable index:module Mods
MyShip:GetModules[Mods]
variable iterator it
Mods:GetIterator[it]
if ${it:First(exists)}
    do
    {
        if ${it.Value.IsActivatable} && !${it.Value.IsActive}
            it.Value:Activate
    }
    while ${it:Next(exists)}
```

### Reading Ship and Capacitor Health

```lavishscript
echo "Shield ${MyShip.ShieldPct.Precision[2]}%  Armor ${MyShip.ArmorPct.Precision[2]}%  Structure ${MyShip.StructurePct.Precision[2]}%  Cap ${MyShip.CapacitorPct.Precision[2]}%"
```

Remember: `Shield`, `Armor`, `Structure`, `Capacitor` are absolute HP/EP. Use the `Pct` variants for percentages.

### Unified Inventory Cargo Access

The modern, reliable pattern — avoids the "cargo not opened yet" caveat:

```lavishscript
EVE:Execute[OpenInventory]
wait 15 ${EVEWindow[Inventory](exists)}

if ${EVEWindow[Inventory](exists)}
{
    if ${EVEWindow[Inventory].ChildWindow[ShipCargo](exists)}
    {
        EVEWindow[Inventory].ChildWindow[ShipCargo]:MakeActive
        ; Source enforces a 0.5-second cooldown between MakeActive and GetItems;
        ; wait >= 10 deciseconds to avoid the "too soon after MakeActive" rejection.
        wait 10

        variable index:item Items
        EVEWindow[Inventory].ChildWindow[ShipCargo]:GetItems[Items]
        echo "Cargo: ${EVEWindow[Inventory].ChildWindow[ShipCargo].UsedCapacity} / ${EVEWindow[Inventory].ChildWindow[ShipCargo].Capacity} m³  (${Items.Used} stacks)"
    }
}
```

### Fleet Member Iteration

```lavishscript
if ${Me.Fleet.ID(exists)}
{
    variable index:fleetmember Members
    Me.Fleet:GetMembers[Members]
    echo "Fleet size: ${Me.Fleet.Size}"

    variable iterator it
    Members:GetIterator[it]
    if ${it:First(exists)}
        do
        {
            echo "${it.Value.Name} (${it.Value.Role})  FC=${it.Value.IsFleetCommander}"
        }
        while ${it:Next(exists)}
}
```

### Warping to a Bookmark

```lavishscript
if ${EVE.Bookmark["Safe Spot"](exists)}
    EVE.Bookmark["Safe Spot"]:WarpTo[0]
```

### Docking a Station by ID

```lavishscript
variable int64 StationID = 60003760
if ${Entity[${StationID}](exists)}
    Entity[${StationID}]:Dock
else
{
    ; Station not on grid — set it as autopilot destination and travel
    EVE.Station[${StationID}]:SetDestination
    EVE:Execute[CmdToggleAutopilot]
}
```

### Undocking

```lavishscript
if ${Me.InStation}
{
    EVE:Execute[CmdExitStation]
    wait 150 ${Me.InSpace}
}
```

### Drone Engage-and-Return

```lavishscript
if ${Me.ActiveTarget(exists)}
{
    variable index:int64 DroneIDs
    Me:GetActiveDroneIDs[DroneIDs]
    if ${DroneIDs.Used} == 0
        MyShip:LaunchAllDrones
    else
        EVE:DronesEngageMyTarget[DroneIDs]
}
```

### Reading the Wallet

```lavishscript
; Balance returns -1.0 until the wallet window has been opened at least once.
if ${Me.Wallet.Balance} >= 0
    echo "ISK: ${Me.Wallet.Balance.Int}"
else
{
    EVE:Execute[OpenWallet]
    wait 30
    echo "ISK: ${Me.Wallet.Balance.Int}"
}
```

### D-Scan

```lavishscript
function DoDScan()
{
    if !${EVEWindow[directionalScannerWindow](exists)}
    {
        EVE:Execute[OpenScanner]
        wait 15 ${EVEWindow[directionalScannerWindow](exists)}
    }
    if !${EVEWindow[directionalScannerWindow](exists)}
        return

    ; Start the scan via the scan button, then poll IsScanning
    EVEWindow[directionalScannerWindow].Button[1]:Press
    wait 50 !${EVEWindow[directionalScannerWindow].IsScanning}

    variable index:directionalscannerresult Results
    EVEWindow[directionalScannerWindow]:GetScanResults[Results]

    variable iterator it
    Results:GetIterator[it]
    if ${it:First(exists)}
        do
        {
            echo "${it.Value.Name}  (${it.Value.Group})"
        }
        while ${it:Next(exists)}
}
```

### Bookmark Creation

```lavishscript
; Simple
EVE:CreateBookmark["Safe Spot"]

; With notes
EVE:CreateBookmark["Safe Spot", "Deep safe"]

; Corp folder
EVE:CreateBookmark["Corp Safe", "Fleet rally", "corp"]

; Temporary (3-hour expiry) — the 4th argument is a discrete enum: 0=no expiry, 1=3 hours, 2=2 days
EVE:CreateBookmark["Temp", "Short-lived", "Personal", 1]

; At an entity's location
${Entity[${ID}]}:CreateBookmark["Mission Spot", "Where the pocket starts"]
```

For deeper patterns see [05_Patterns_And_Best_Practices.md](ISXEVE%20Scripting%20Guide/05_Patterns_And_Best_Practices.md), [15_Combat_Automation.md](ISXEVE%20Scripting%20Guide/15_Combat_Automation.md), [16_Mining_And_Hauling.md](ISXEVE%20Scripting%20Guide/16_Mining_And_Hauling.md), [17_Fleet_Operations.md](ISXEVE%20Scripting%20Guide/17_Fleet_Operations.md).

---

## Canonical Slot Names and Destinations

### Module Slot Names

`MyShip.Module[...]` and `EVEWindow[Fitting].Slot[...]` both accept these string names (case-insensitive):

| Slot Type | Valid names |
|---|---|
| High | `HiSlot0`, `HiSlot1`, `HiSlot2`, `HiSlot3`, `HiSlot4`, `HiSlot5`, `HiSlot6`, `HiSlot7` |
| Medium | `MedSlot0`, `MedSlot1`, `MedSlot2`, `MedSlot3`, `MedSlot4`, `MedSlot5`, `MedSlot6`, `MedSlot7` |
| Low | `LoSlot0`, `LoSlot1`, `LoSlot2`, `LoSlot3`, `LoSlot4`, `LoSlot5`, `LoSlot6`, `LoSlot7` |
| Rig | `RigSlot0`, `RigSlot1`, `RigSlot2`, `RigSlot3`, `RigSlot4`, `RigSlot5`, `RigSlot6`, `RigSlot7` |

Also accepted: the module's int64 item ID.

Note: numeric-index forms like `MyShip.Module[0]` and two-arg forms like `MyShip.Module[HiSlot, 0]` are NOT supported. Use the concatenated token (`HiSlot0`) or a module ID.

### MoveTo Destinations

Used by `item:MoveTo[toID, destination, ...]` and `EVE:MoveItemsTo[items, toID, destination, ...]`:

`CargoHold`, `DroneBay`, `CorpHangars`, `MaintenanceBay`, `OreHold`, `FuelBay`, `GasHold`, `MineralHold`, `SalvageHold`, `IndustrialShipHold`, `AmmoHold`, `StationCorporateHangar`, `Hangar`, `None`, `N/A`, `FleetHangar`, `CommandCenterHold`, `PlanetaryCommoditiesHold`.

### ToLocation Short-hands

For the `toID` / `toLocation` first argument, these string aliases are also accepted in place of a numeric location ID:

`MyShip`, `MyStationHangar`, `MyStationCorporateHangar`.

### Corp Folder Names

Used as the trailing `folder` argument when moving to/from `CorpHangars` or `StationCorporateHangar`:

`Corporation Folder 1` through `Corporation Folder 7`.

---

## Deprecated and Removed

This section tracks APIs removed from ISXEVE over time. Scripts using them must be updated. Sourced from `ISXEVEChanges.txt`; dates are the official changelog-recorded removals.

### High-Impact Migrations

| From (removed) | To (current) | Notes |
|---|---|---|
| `item:FitToActiveShip` | `EVEWindow[Fitting].Slot[slotName]:FitItem[itemID]` | Jan 2025 |
| Ship/Entity/Item cargo methods (`GetCargo`, `OpenCargo`, `CloseCargo`, `StackAllCargo`, `OpenStorage`, `OpenMaintenanceBay`, `OpenCorpHangars`, `OpenDroneBay`, `OpenOreHold`, `StorageWindow`, and the specialized `Get*HoldCargo` family) | `EVEWindow[Inventory].ChildWindow[name]:GetItems[index:item]` and related child-window methods | July 2020 |
| `eve:PlaceBuyOrder[typeID]` | `eve:CreateMarketBuyOrder[typeID]` | April 2018 |
| `eve:PlaceSellOrder` | `item:AddToSellOrder` | January 2015 |
| `character:Undock` | `EVE:Execute[CmdExitStation]` | July 2007 |
| `station:OpenCorpHangar` | `EVE:Execute[OpenHangarFloor]` | 2012. NOTE: the changelog also claims `character:OpenCorpHangar` was removed at this time, but source still implements that character-datatype method. See [character methods](#character) — `OpenCorpHangar` is documented there as a working API. Only the `station` version was actually removed. |
| `Agent` TLO | `EVE.Agent[#]` (index), `EVE.Agent[id, #]` (explicit ID), or `EVE.Agent[name]` | May 2020 |
| `Login` TLO / `login` datatype | Removed — in-game login no longer supported by EVE | June 2020 |

### Removed Members

| Datatype | Member | Date | Notes |
|---|---|---|---|
| `module` | `Accuracy` | May 2020 | Had been disabled for years. |
| `module` | `TimeLastClicked` | 2019 | |
| `module` | `IsChangingAmmo` | 2019 | Replaced by `IsReloadingAmmo`. |
| `skill` | `ID`, `Group`, `GroupID`, `IsTraining` | Nov 2015 | |
| `evewindow` | `ItemID`, `Capacity`, `UsedCapacity` | Aug 2018 | Moved to `eveinvwindow`. |
| `chatchannel` | `NewMessageReceived`, `CanSpeak` | Apr 2018 | EVE chat rewrite. |
| `agent` / `eveagent` | `Dialog` | Feb 2021 | |
| `skill` | Renamed `SkillLevel` → `TrainedLevel`, `SkillPoints` → `Points` | 2023 | `EffectiveLevel` added. |
| `marketorder` / `myorder` | `SolarSystem` (return type changed) | June 2023 | Now returns `solarsystem` object, not string. |

### Removed Methods

| Datatype | Method | Date | Replacement |
|---|---|---|---|
| `agentmission` | `GetDetails` | June 2023 | — |
| `chatchannel` | `MarkAsRead`, `Echo`, `Send` | April 2018 | LavishScript chat send (may return in a future build). |
| `skill` | `AbortTraining` | Nov 2015 | — |
| `isxeve` | `Debug_SetFullMiniDumps` | July 2020 | — |
| `evewindow` | `MoveBookmarkHere[bookmarkID]` | July 2020 | — |
| `eveinvwindow` | Various older `ChildCapacity` / `ChildWindowExists` member flavors | December 2012 | Use `ChildWindow[name].HasCapacity`, `ChildWindow[name].Capacity`, etc. |
| `entity` | `StackAllCargo` | March 2012 | `evewindow:StackAll`. |
| `ship` | `StackAllCargo` | March 2012 | `evewindow:StackAll`. |
| `item` | `Launch` | ~2014 | `LaunchDrones`. |

### Removed Datatypes

- `dialogstring` (Feb 2021) — removed when agent dialog system changed.
- `login` (June 2020) — removed.

### Removed Events

- `EVE_OnPilotJoinedChannel` (disabled March 2018) — EVE chat-system change.
- `EVE_OnPilotLeftChannel` (disabled March 2018) — EVE chat-system change.

ISXEVE maintains backward compatibility for a transition period: deprecated APIs log a console warning identifying the replacement. Update scripts promptly when a warning appears.

---

## Notes

### Case Sensitivity

Case-INSENSITIVE:
- TLO names, datatype names, member names, method names
- `EVE:Execute` command constants (though idiomatic form is `CmdExitStation` etc.)

Case-SENSITIVE:
- Entity/item/character names (string comparisons within queries)
- Query expressions in `EVE:QueryEntities[]` — strings inside the query are compared case-sensitively
- File-system paths when writing portable code

### Existence and NULL

Every datatype-returning member may be NULL. Always guard:

```lavishscript
; WRONG — crashes / prints NULL
echo ${Me.ActiveTarget.Name}

; RIGHT
if ${Me.ActiveTarget(exists)}
    echo ${Me.ActiveTarget.Name}
```

**Sentinel return values:** many numeric members return sentinels on error or unready data:
- `-1` — error.
- `-2` — data not ready yet (inventory child windows in particular).
- `-10` / `-10.0` — wormhole members when the entity isn't a wormhole.

Always validate before arithmetic.

### Parameter Notation in This Document

- `[param]` — required.
- `[optional]` — optional.
- `#` — any integer/numeric.
- `"text"` — string literal.
- `id` — int64 entity/character/item/etc. ID.
- `name` — string name lookup.

### EVE Client Updates

ISXEVE is updated regularly to track EVE client patches. When EVE patches:
1. Wait for the ISXEVE update announcement.
2. Update ISXEVE via `ext isxeve` reload or InnerSpace's built-in updater.
3. Re-test scripts against the changelog for any breaking changes.

### Performance Notes

**Entity queries**: Cache results when possible; don't re-query every frame. Prefer `Distance2` to `Distance` for comparisons (skips the square-root).

**Universe TLO**: The first string-lookup against `Universe[name]` each session caches the entire name database and causes a visible lag spike. Prefer ID lookups, or do your first name lookup during startup before the main loop.

**Inventory**: `EVEWindow[Inventory].ChildWindow[...]` methods are far more reliable than the deprecated cargo methods. They also trigger the "cargo has been opened" side-effect automatically so you don't need to pre-open anything.

**Fleet/standings**: `EVE:RefreshStandings` and `EVE:RefreshBookmarks` must be called at least once after login. Data is not immediately available.

### Authentication

ISXEVE requires valid isxGames credentials. Set via either:

1. `ISXEVE.xml` configuration file (default method), or
2. Global variables (for shared machines or temporary sessions):
   ```lavishscript
   declarevariable isxGames_Login string globalkeep YourUsername
   declarevariable isxGames_Password string globalkeep YourPassword
   ext isxeve
   ```

Global variables persist across ISXEVE reloads within a session but not across InnerSpace sessions. Change password at [https://auth.isxgames.com/login](https://auth.isxgames.com/login).

Best practices:
- Use a unique isxGames password (not shared with other services).
- Don't commit credentials into scripts.
- Keep credentials in a separate private file that isn't tracked or shared.

### Async Data Gotchas (Quick Checklist)

- Cargo members (`ship.CargoCapacity`, `entity.CargoCapacity`, etc.) require the corresponding inventory/cargo window to have been opened at least once this session.
- `Me.Wallet.Balance` returns `-1.0` until the wallet window has been opened.
- `Universe[name]` first-call lag — see above.
- `ChildWindow[...].Capacity` / `.UsedCapacity` return `-2` when the child hasn't been made active yet.
- Standings/bookmark caches need `EVE:RefreshStandings` / `EVE:RefreshBookmarks` after login.
- Market data needs `EVE:FetchMarketOrders[typeID, regionID]` before `EVE:GetMarketOrders`.

### Cross-References

This QuickReference is the source-verified single-file ISXEVE API reference. For broader topical coverage -- platform fundamentals, ISXEVE-specific patterns and architecture, UI, multiboxing, debugging, etc. -- see the [ISXEVE Scripting Guide](ISXEVE%20Scripting%20Guide/README.md). Every guide in the KB tree is listed below with a one-line "best for" routing hint. Order: README first, then numerical guide order, then meta/agent files.

- [README.md](ISXEVE%20Scripting%20Guide/README.md) -- KB tree overview, learning paths, version history. Best for first-time orientation; skip for targeted technical lookups.
- [00_MASTER_GUIDE.md](ISXEVE%20Scripting%20Guide/00_MASTER_GUIDE.md) -- master cross-reference hub linking ISXEVE datatypes, commands, events, and learning levels to the relevant numbered guides. Best for "where do I find X" navigation when you already know the topic.
- [01_LavishScript_Fundamentals.md](ISXEVE%20Scripting%20Guide/01_LavishScript_Fundamentals.md) -- tutorial-style introduction to LavishScript core, Inner Space core, and bundled subsystem packages (LGUI / LavishNav / LavishSettings / LavishMachine / LSModule). Game-agnostic. Best for "how do I learn", conceptual onboarding, atom/method/wait semantics, OOP basics, control flow, OS-level features (file system / web requests / audio / input emulation).
- [01b_LavishScript_Reference.md](ISXEVE%20Scripting%20Guide/01b_LavishScript_Reference.md) -- exhaustive LavishScript / Inner Space command, datatype, and Top-Level-Object inventory with one canonical entry per feature linked to the Lavish Software wiki. Game-agnostic. Best for "what's the exact signature of X", "does Y exist", "what wiki page documents Z" -- pure lookup.
- [02_Quick_Start_Guide.md](ISXEVE%20Scripting%20Guide/02_Quick_Start_Guide.md) -- shortest path from zero to a running ISXEVE script. Best for "I just installed ISXEVE, where's the smoke-test script".
- [03_API_Reference.md](ISXEVE%20Scripting%20Guide/03_API_Reference.md) -- exhaustive ISXEVE API reference (~11,500 lines): every datatype, member, method, event, command, and TLO that ISXEVE registers. Best for full ISXEVE-specific API surface lookups when this QuickReference is too compact and you need every detail. Defer to `ISXEVEChanges.txt` (~5,600 lines) if the two disagree -- the changelog is the definitive source.
- [04_Core_Concepts.md](ISXEVE%20Scripting%20Guide/04_Core_Concepts.md) -- ISXEVE-specific architectural concepts: ship/entity/character relationships, query syntax, async data loading, the Me/MyShip/EVE TLO interaction model. Best for "why does my entity query return NULL" / "how do bookmarks/agents/locations interrelate" / "what does the inventory window flow look like".
- [05_Patterns_And_Best_Practices.md](ISXEVE%20Scripting%20Guide/05_Patterns_And_Best_Practices.md) -- production scripting patterns and anti-patterns. Best for "what's the canonical way to structure a bot's main loop / state machine / pulse architecture" and to avoid common pitfalls.
- [06_Working_Examples.md](ISXEVE%20Scripting%20Guide/06_Working_Examples.md) -- complete runnable scripts for common ISXEVE tasks. Best for "show me a copy-paste starting point" -- combat rotation, mining cycle, dock/undock, character info, etc.
- [07_Advanced_Patterns_And_Examples.md](ISXEVE%20Scripting%20Guide/07_Advanced_Patterns_And_Examples.md) -- advanced multi-script / multi-box / IPC patterns built on top of guide 05. Best for "how do I coordinate multiple sessions" / "how does relay-based broadcasting work in practice".
- [08_LavishGUI1_UI_Guide.md](ISXEVE%20Scripting%20Guide/08_LavishGUI1_UI_Guide.md) -- LavishGUI 1 (legacy XML-based) UI basics. Best for maintaining or reading existing LGUI 1 scripts. Use LGUI 2 (guide 10) for new UIs.
- [09_Advanced_LGUI1_Patterns.md](ISXEVE%20Scripting%20Guide/09_Advanced_LGUI1_Patterns.md) -- advanced LGUI 1 patterns and idioms. Best for deep work on existing legacy UIs.
- [10_LavishGUI2_UI_Guide.md](ISXEVE%20Scripting%20Guide/10_LavishGUI2_UI_Guide.md) -- LavishGUI 2 (modern JSON-based) UI guide. Best for ALL new UI work in ISXEVE scripts -- elements, layouts, skins, event handlers, item views.
- [11_LavishGUI1_to_LavishGUI2_Migration.md](ISXEVE%20Scripting%20Guide/11_LavishGUI1_to_LavishGUI2_Migration.md) -- LGUI 1 to LGUI 2 migration cookbook plus the canonical LGUI 1 type/element surface. Best for "I have an existing LGUI 1 script and want to port it" or "what's the LGUI 1 surface" (since 01b only points here for LGUI 1).
- [12_LGUI2_Scaling_System.md](ISXEVE%20Scripting%20Guide/12_LGUI2_Scaling_System.md) -- LGUI 2 dynamic scaling subsystem. Best for "how do I make my UI respect the user's scale setting" / DPI-aware UI work.
- [13_JSON_Guide.md](ISXEVE%20Scripting%20Guide/13_JSON_Guide.md) -- JSON-specific patterns: index/collection serialization, custom-objectdef AsJSON / Initialize(jsonvalueref), jsonvalueref usage. Best for "how do I (de)serialize my custom objects to JSON" and the LMAC-task-style JSON-driven config patterns.
- [14_LavishMachine_Guide.md](ISXEVE%20Scripting%20Guide/14_LavishMachine_Guide.md) -- LavishMachine / Tasks system: Task Managers, Task Types, Task Libraries, declarative `audio.playstream` / `audio.setvolume` / `webrequest` task types. Best for "I want declarative timed/async automation managed by a task manager" -- pairs with the imperative APIs in 01.
- [15_Combat_Automation.md](ISXEVE%20Scripting%20Guide/15_Combat_Automation.md) -- combat-specific patterns: target selection, weapon control, tank management, the Module Control Layer. Best for "how do I write a combat bot" / module-cycling / drone-management / aggro-handling questions.
- [16_Mining_And_Hauling.md](ISXEVE%20Scripting%20Guide/16_Mining_And_Hauling.md) -- mining + hauling patterns: ore scanning, strip-miner control, hauler coordination, Orca patterns. Best for "I want to write a mining bot" / "how does Orca-fleet interaction work".
- [17_Fleet_Operations.md](ISXEVE%20Scripting%20Guide/17_Fleet_Operations.md) -- fleet coordination, multi-box patterns, leader-follower architecture, fleet-warp/broadcast primitives. Best for "how do I coordinate N sessions doing fleet ops together" / canonical Yamfa-style patterns.
- [18_Bot_Architecture_Analysis.md](ISXEVE%20Scripting%20Guide/18_Bot_Architecture_Analysis.md) -- analysis of EVEBot / Yamfa / Tehbot / Navigator architectures, with grounded references to the real source. Best for "how do production bots structure themselves" / "where in EVEBot does the X pattern live" / "what does Tehbot's mission-data registry look like".
- [19_DotNet_Development.md](ISXEVE%20Scripting%20Guide/19_DotNet_Development.md) -- .NET 2.0/3.5 and 4.0 integration with Inner Space. Best for "I want to write part of my ISXEVE script in C#" / `DotNet` command + assembly loading questions.
- [20_Debugging_And_Troubleshooting.md](ISXEVE%20Scripting%20Guide/20_Debugging_And_Troubleshooting.md) -- debugging capstone: logging, profiling, atom-leak-across-reload, common runtime errors, diagnostic patterns. Best for "my script is broken / silent / leaking, where do I look".
- [21_Advanced_Scripting_Patterns.md](ISXEVE%20Scripting%20Guide/21_Advanced_Scripting_Patterns.md) -- production-grade architectural patterns: state machines, multi-threading, LavishSettings persistence, the Unified Movement Facade, timer objects, relay-based IPC. Best for "I'm building a polished production bot, what's the canonical scaffolding".
- [22_Utility_Script_Patterns.md](ISXEVE%20Scripting%20Guide/22_Utility_Script_Patterns.md) -- short-utility / one-shot / market-tool / character-info script patterns. Best for "I want a small utility, not a full bot" -- clipboard helpers, info dumps, ad-hoc automation.
- [+How To Use This Guide with Claude Code+.md](ISXEVE%20Scripting%20Guide/%2BHow%20To%20Use%20This%20Guide%20with%20Claude%20Code%2B.md) -- human-facing setup guide for Claude Code integration. Mostly out of agent scope (skip-blocked); useful only if a user asks about installing or configuring the coordinator/agent.
- [Claude AI Commands (optional)/isxeve.md](ISXEVE%20Scripting%20Guide/Claude%20AI%20Commands%20%28optional%29/isxeve.md) -- master copy of the `/isxeve` Claude Code coordinator command (slash-command definition that spawns the ISXEVE-Expert agent). Best for the user's reference when re-installing the integration.
- [Claude AI Commands (optional)/ISXEVE-Expert.md](ISXEVE%20Scripting%20Guide/Claude%20AI%20Commands%20%28optional%29/ISXEVE-Expert.md) -- master copy of the ISXEVE-Expert worker-agent definition. Best for the user's reference when re-installing the integration.
- [Claude AI Commands (optional)/README.md](ISXEVE%20Scripting%20Guide/Claude%20AI%20Commands%20%28optional%29/README.md) -- quick-start summary for installing the Claude Code coordinator + agent files into `~/.claude/`. Best for the user when first setting up the integration.
