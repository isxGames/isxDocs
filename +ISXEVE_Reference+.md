# ISXEVE Reference

## Table of Contents

1. [Introduction](#introduction)
2. [Top-Level Objects](#top-level-objects)
   - [Extension TLOs](#extension-tlos)
   - [Core TLOs](#core-tlos)
   - [Window TLOs](#window-tlos)
3. [DataType Categories](#datatype-categories)
   - [Core/Extension DataTypes](#coreextension-datatypes)
   - [Character & Being DataTypes](#character--being-datatypes)
   - [Entity & Space DataTypes](#entity--space-datatypes)
   - [Item & Equipment DataTypes](#item--equipment-datatypes)
   - [Ship DataTypes](#ship-datatypes)
   - [Station & Structure DataTypes](#station--structure-datatypes)
   - [Window DataTypes](#window-datatypes)
   - [UI Element DataTypes](#ui-element-datatypes)
   - [Interstellar DataTypes](#interstellar-datatypes)
   - [Market DataTypes](#market-datatypes)
   - [Scanner DataTypes](#scanner-datatypes)
   - [Agent & Mission DataTypes](#agent--mission-datatypes)
   - [Skill DataTypes](#skill-datatypes)
   - [Chat DataTypes](#chat-datatypes)
   - [Drone DataTypes](#drone-datatypes)
   - [Fleet DataTypes](#fleet-datatypes)
   - [Miscellaneous DataTypes](#miscellaneous-datatypes)
4. [Commands](#commands)
   - [Core Commands](#core-commands)
   - [Web Commands](#web-commands)
5. [Events](#events)
   - [Event Registration](#event-registration)
   - [EVE Events](#eve-events)
   - [System Events](#system-events)
6. [Usage Examples](#usage-examples)
   - [Basic Information](#basic-information)
   - [Autopilot and Navigation](#autopilot-and-navigation)
   - [Bookmarks](#bookmarks)
   - [Items and Inventory](#items-and-inventory)
   - [Ship and Modules](#ship-and-modules)
   - [Targets and Combat](#targets-and-combat)
   - [Market Operations](#market-operations)
   - [Scanning](#scanning)
   - [Universe and Planetary Information](#universe-and-planetary-information)
   - [Station and Structure Operations](#station-and-structure-operations)
   - [UI Interaction](#ui-interaction)
   - [Fleet Operations](#fleet-operations)
   - [Chat Operations](#chat-operations)
7. [Notes](#notes)
   - [Case Sensitivity](#case-sensitivity)
   - [NULL Checks](#null-checks)
   - [Parameter Notation](#parameter-notation)
   - [EVE Client Updates](#eve-client-updates)
   - [Performance Considerations](#performance-considerations)
   - [Authentication](#authentication)
   - [Deprecated Features](#deprecated-features)

---

## Introduction

This document provides comprehensive reference documentation for all datatypes, top-level objects, commands, and events available in the ISXEVE extension for EVE Online. ISXEVE extends LavishScript to provide access to game data and functionality through a structured type system.

### DataType Inheritance

Many datatypes inherit from other datatypes, gaining all the members and methods of their parent type. Common inheritance patterns in ISXEVE:

- [**character**](#character) inherits from [**pilot**](#pilot)
- [**pilot**](#pilot) inherits from [**being**](#being)
- [**fleetmember**](#fleetmember) inherits from [**pilot**](#pilot)
- [**module**](#module) inherits from [**item**](#item)
- [**structure**](#structure) inherits from [**station**](#station)
- [**planet**](#planet) inherits from [**interstellar**](#interstellar)
- [**solarsystem**](#solarsystem) inherits from [**interstellar**](#interstellar)
- [**constellation**](#constellation) inherits from [**interstellar**](#interstellar)
- [**region**](#region) inherits from [**interstellar**](#interstellar)
- [**entitywormhole**](#entitywormhole) inherits from [**entity**](#entity)
- All specialized window types inherit from [**evewindow**](#evewindow)

### Accessing DataTypes

DataTypes are accessed through Top-Level Objects (TLOs) or through members of other datatypes. For example:

```lavishscript
${Me}                           // TLO returning 'character' datatype
${Me.Ship}                      // Member returning 'ship' datatype
${Me.GetTargets}                // Method populating index with 'entity' objects
${EVEWindow[Inventory]}         // TLO with parameter returning 'eveinvwindow' datatype
${MyShip.Module[1]}             // Member returning 'module' datatype
```

---

## Top-Level Objects

Top-Level Objects (TLOs) are the entry points for accessing game data. They can be accessed directly in LavishScript.

### Extension TLOs

| TLO | DataType | Description |
|-----|----------|-------------|
| **ISXEVE** | [isxeve](#isxeve) | ISXEVE extension information and utilities |


### Core TLOs

| TLO | DataType | Description |
|-----|----------|-------------|
| **EVE** | [eve](#eve) | Main game information and utilities |
| **Me** | [character](#character) | Player character |
| **MyShip** | [ship](#ship) | Player's active ship |
| **EVETime** | [evetime](#evetime) | Current EVE time |
| **Local[id/name]** | [pilot](#pilot) | Access pilot from local channel by ID or name |
| **Chat** | [evechat](#evechat) | Access chat channels |
| **Chat[id/name]** | [chatchannel](#chatchannel) | Access specific chat channel by ID or name |
| **Overview** | [overview](#overview) | Overview functionality |
| **Universe[id/name]** | [interstellar](#interstellar) | Access universe objects by ID or name |
| **Entity[id/name]** | [entity](#entity) | Access entity by ID or name (space only) |
| **Being[id]** | [being](#being) | Access being by ID |
| **CharSelect** | [charselect](#charselect) | Character selection screen (login only) |


### Window TLOs

| TLO | DataType | Description |
|-----|----------|-------------|
| **EVEWindow[name]** | [evewindow](#evewindow) | Access EVE window by name |
| **EVEWindow[byName,name]** | [evewindow](#evewindow) | Access EVE window by exact name |
| **EVEWindow[byCaption,text]** | [evewindow](#evewindow) | Access EVE window by caption text |
| **EVEWindow[byItemID,id]** | [evewindow](#evewindow) | Access EVE window by item ID |
| **EVEWindow[local]** | [evewindow](#evewindow) | Access local chat window |
| **EVEWindow[active]** | [evewindow](#evewindow) | Access currently active window |
| **EVEWindow[#]** | [evewindow](#evewindow) | Access window by index number |

---

## DataTypes

### Core/Extension DataTypes

#### **isxeve**

Extension information and utilities. Accessed via ISXEVE TLO.

**Members:**
- `Version` - string: ISXEVE version number
- `IsReady` - bool: Whether extension is ready for use
- `IsLoading` - bool: Whether extension is currently loading
- `SecsToString[seconds]` - string: Converts seconds to formatted time string
- `IsSafe` - bool: Whether running safe build
- `IsBeta` - bool: Whether running beta build
- `Debug1` - bool: Debug flag

**Methods:**

*Debug Methods:*
- `Debug_SetTypeValidation[enabled]` - Enable/disable type validation
- `Debug_SetEntityCacheEnabled` - Enable entity caching
- `Debug_SetEntityCacheDisabled` - Disable entity caching
- `Debug_SetHighPerfLogging[enabled]` - Enable/disable high performance logging
- `Debug_LogMsg[message]` - Log debug message
- `Debug_PrintCacheInfo` - Print entity cache information
- `Debug_DumpEntityCache` - Dump entity cache contents

*Installation Methods:*
- `InstallBeta` - Install beta version
- `InstallTest` - Install test version
- `InstallLive` - Install live version

---

#### **eve**

Main EVE datatype providing core game functionality. Accessed via EVE TLO.

**Members:**
- `EntitiesCount` - int: Number of entities in space
- `DistanceBetween[id1,id2]` - double: Distance between two entities
- `Bookmark[id or label]` - [bookmark](#bookmark): Bookmark by ID or label
- `JumpsToStation[stationID]` - int: Jumps to station
- `Time` - string: Current EVE time
- `Date` - string: Current EVE date
- `NumAssetsAtStation[stationID]` - int: Number of assets at station
- `GetLocationNameByID[id]` - string: Location name by ID
- `Station[id]` - [station](#station): Station by ID
- `JumpsBetween[fromID,toID]` - int: Jumps between locations
- `JumpsTo[solarSystemID]` - int: Jumps to solar system
- `Is3DDisplayOn` - bool: Whether 3D display is on
- `IsUIDisplayOn` - bool: Whether UI display is on
- `IsTextureLoadingOn` - bool: Whether texture loading is on
- `NextSessionChange` - [evetime](#evetime): Next session change time
- `InCriticalSection` - bool: Whether in critical section
- `QueryEvaluate[query,entity]` - bool: Evaluates query against entity
- `IsProgressWindowOpen` - bool: Whether progress window is open
- `ProgressWindowTitle` - string: Progress window title
- `AbandonedDronesExist` - bool: Whether abandoned drones exist
- `MinWarpDistance` - double: Minimum warp distance
- `ItemInfo[typeID]` - [iteminfo](#iteminfo): Item info by type ID
- `Structure[id]` - [structure](#structure): Structure by ID
- `Agent[id or name]` - [agent](#agent): Agent by ID or name

**Methods:**

*UI & Display:*
- `InfoWindow[text]` - Opens info window
- `SetInSpaceStatus[text]` - Sets in-space status text
- `DoCommand[command]` - Executes command
- `Execute[command]` - Executes EVE UI command
- `Toggle3DDisplay` - Toggles 3D display
- `ToggleUIDisplay` - Toggles UI display
- `ToggleTextureLoading` - Toggles texture loading
- `CloseAllMessageBoxes` - Closes all message boxes
- `CloseAllChatInvites` - Closes all chat invites
- `GetEVEWindows[index:evewindow]` - Populates index with EVE windows

*Drones:*
- `LaunchDrones[index:item or int64]` - Launches drones
- `DronesMine[index:int64]` - Commands drones to mine
- `DronesMineRepeatedly[index:int64]` - Commands drones to mine repeatedly
- `DronesScoopToDroneBay[index:int64]` - Scoops drones to bay
- `DronesEngageMyTarget[index:int64]` - Engages target with drones
- `AbandonDrones[index:int64]` - Abandons drones
- `ReturnFighterControl[index:int64]` - Returns fighter control
- `DelegateFighterControl[index:int64,charID]` - Delegates fighter control
- `DronesAssist[index:int64,charID]` - Drones assist character
- `DronesGuard[index:int64,charID]` - Drones guard character
- `DronesReturnAndOrbit[index:int64]` - Drones return and orbit
- `DronesReturnToDroneBay[index:int64]` - Returns drones to bay
- `ReclaimDrones` - Reclaims abandoned drones

*Entities:*
- `PopulateEntities[categoryID]` - Populates entity list
- `QueryEntities[index:entity,query]` - Populates index with entities matching query
- `GetViewedWrecks[index:int64]` - Populates index with viewed wreck IDs

*Bookmarks & Navigation:*
- `GetBookmarks[index:bookmark,folder]` - Populates index with bookmarks
- `RefreshBookmarks` - Refreshes bookmark cache
- `CreateBookmark[label,notes,folder,expiry]` - Creates bookmark
- `AddWaypoint[solarSystemID]` - Adds waypoint
- `ClearWaypoint[solarSystemID]` - Clears waypoint
- `ClearAllWaypoints` - Clears all waypoints
- `GetToDestinationPath[index:int]` - Gets path to destination
- `GetWaypoints[index:int]` - Populates index with waypoint IDs
- `OptimizeAutopilotRoute` - Optimizes autopilot route

*Contacts & Agents:*
- `GetLocalPilots[index:pilot]` - Populates index with local pilots
- `GetOnlineCorpMembers[index:being]` - Populates index with online corp members
- `GetContacts[index:being]` - Populates index with contacts
- `GetBuddies[index:being]` - Populates index with buddies (deprecated)
- `GetAgents[index:agent]` - Populates index with agents
- `GetAgentMissions[index:agentmission,agentID]` - Populates index with agent missions

*Market:*
- `FetchMarketOrders[typeID,regionID]` - Fetches market orders
- `ClearMarketOrderCache` - Clears market order cache
- `GetMarketOrders[index:marketorder,typeID]` - Populates index with market orders
- `CreateMarketBuyOrder[typeID]` - Opens market buy order window

*Items:*
- `MoveItemsTo[location,destination,index:item or int64,folder]` - Moves items
- `StackItems[location,destination,index:item or int64,folder]` - Stacks items

*Other:*
- `RefreshStandings` - Refreshes standings cache
- `EnterCriticalSection` - Enters critical section
- `LeaveCriticalSection` - Leaves critical section
- `ViewPlanetaryIndustry[planetID]` - Opens planetary industry window

**See Also:**
- [character](#character) - Player character (Me TLO)
- [ship](#ship) - Player ship (MyShip TLO)
- [bookmark](#bookmark) - Bookmarks
- [agent](#agent) - Agents
- [entity](#entity) - Entities

---

#### **evetime**

EVE time representation.

**Members:**
- `DateAndTime` - string: Formatted date and time string
- `Date` - string: Date portion
- `Time` - string: Time portion
- `AsInt64` - int64: Time as int64 (FILETIME format)

---

### Character & Being DataTypes

#### **character**

Player character. Inherits from [**pilot**](#pilot) which inherits from [**being**](#being).

**Inheritance:**
- All members and methods from [**pilot**](#pilot) (and [**being**](#being)) are available

**Additional Members:**

*Fleet & Corporation:*
- `Fleet` - [fleet](#fleet): Current fleet (if in fleet)
- `ToFleetMember` - [fleetmember](#fleetmember): Converts to fleetmember type if in fleet
- `ToEntity` - [entity](#entity): Converts to entity type
- `Corp` - [corporation](#corporation): Corporation information
- `Wallet` - [wallet](#wallet): Wallet information
- `Alliance` - string: Alliance name
- `AllianceID` - int: Alliance ID
- `AllianceTicker` - string: Alliance ticker

*Location:*
- `RegionID` - int: Current region ID
- `ConstellationID` - int: Current constellation ID
- `SolarSystemID` - int: Current solar system ID
- `ShipID` - int64: Current ship ID
- `StationID` - int64: Current station/structure ID (0 if in space)
- `InStation` - bool: Whether in station or structure
- `InSpace` - bool: Whether in space
- `Station` - [station](#station) or [structure](#structure): Current station/structure
- `AutoPilotOn` - bool: Whether autopilot is on

*Ship & Target:*
- `Ship` - [ship](#ship): Current ship
- `ActiveTarget` - [entity](#entity): Current active target

*Skills:*
- `Skill[id or name]` - [skill](#skill): Skill by ID or name
- `SkillPoints` - int: Total skill points
- `SkillCurrentlyTraining` - [skill](#skill): Currently training skill
- `SkillQueueLength` - int: Number of skills in queue

*Attributes:*
- `Intelligence` - int: Intelligence attribute
- `Perception` - int: Perception attribute
- `Charisma` - int: Charisma attribute
- `Willpower` - int: Willpower attribute
- `Memory` - int: Memory attribute

*Combat & Drones:*
- `MaxLockedTargets` - int: Maximum locked targets
- `MiningDroneAmountBonus` - float: Mining drone amount bonus
- `MaxActiveDrones` - int: Maximum active drones
- `DroneControlDistance` - double: Drone control distance
- `TargetCount` - int: Number of locked targets
- `TargetingCount` - int: Number of targets being locked
- `TargetedByCount` - int: Number of entities targeting you

*Other:*
- `MaxJumpClones` - int: Maximum jump clones
- `StandingTo[id]` - float: Standing to specified character/corp/faction
- `ToPilot` - [pilot](#pilot): Converts to pilot type
- `Contact[# or id or name]` - [being](#being): Contact by index, ID, or name

**Additional Methods:**

*Inventory & Assets:*
- `OpenCorpHangar` - Opens corp hangar
- `GetCorpHangarItems[index:item,folder]` - Populates index with corp hangar [item](#item) objects
- `GetCorpHangarShips[index:item,folder]` - Populates index with corp hangar ships
- `GetHangarItems[index:item]` - Populates index with personal hangar [item](#item) objects
- `GetHangarShips[index:item]` - Populates index with personal hangar ships
- `GetStationsWithAssets[index:int64]` - Populates index with station IDs that have assets
- `GetAssets[index:item,stationID]` - Populates index with [item](#item) objects at station

*Combat:*
- `SetVelocity[speed]` - Sets ship velocity
- `GetTargets[index:entity]` - Populates index with locked [entity](#entity) objects
- `GetTargeting[index:entity]` - Populates index with [entity](#entity) objects being targeted
- `GetTargetedBy[index:entity]` - Populates index with [entity](#entity) objects targeting you
- `GetAttackers[index:attacker]` - Populates index with [attacker](#attacker) objects
- `GetJammers[index:jammer]` - Populates index with [jammer](#jammer) objects

*Drones:*
- `GetActiveDrones[index:activedrone]` - Populates index with [activedrone](#activedrone) objects
- `GetActiveDroneIDs[index:int64]` - Populates index with active drone IDs

*Skills:*
- `GetSkills[index:skill]` - Populates index with [skill](#skill) objects
- `GetSkillQueue[index:queuedskill]` - Populates index with [queuedskill](#queuedskill) objects

*Standing:*
- `SetCorpStanding[id,value]` - Sets standing to corporation
- `SetPilotStanding[id,value]` - Sets standing to pilot

*Market:*
- `GetMyOrders[index:myorder]` - Populates index with [myorder](#myorder) objects
- `UpdateMyOrders` - Updates market orders cache

**See Also:**
- [pilot](#pilot) - Parent type
- [being](#being) - Base type
- [ship](#ship) - Current ship
- [fleet](#fleet) - Fleet information
- [skill](#skill) - Skills
- [corporation](#corporation) - Corporation
- [wallet](#wallet) - Wallet

---

#### **pilot**

Pilot in space or local. Inherits from [**being**](#being).

**Inheritance:**
- All members and methods from [**being**](#being) are available

**Additional Members:**
- `Type` - string: Ship type name
- `TypeID` - int: Ship type ID
- `Corp` - string: Corporation name
- `Alliance` - string: Alliance name
- `AllianceTicker` - string: Alliance ticker
- `AllianceID` - int: Alliance ID
- `Standing` - float: Standing to pilot
- `WarFactionID` - int: War faction ID
- `IsLimitedEngagement` - bool: Whether in limited engagement
- `IsSuspect` - bool: Whether is suspect
- `IsCriminal` - bool: Whether is criminal
- `ToEntity` - [entity](#entity): Converts to entity type
- `ToFleetMember` - [fleetmember](#fleetmember): Converts to fleetmember type if in fleet

**Additional Methods:**
- `SetStanding[value]` - Sets standing to pilot
- `InviteToFleet` - Invites pilot to fleet
- `OpenShowInfo` - Opens show info window

**See Also:**
- [character](#character) - Extended pilot type for player
- [being](#being) - Parent type
- [fleetmember](#fleetmember) - Pilot in fleet
- [entity](#entity) - Entity conversion

---

#### **being**

Base type for any character/NPC representation.

**Members:**
- `ID` - int64: Being ID
- `CharID` - int64: Character ID
- `Name` - string: Character name
- `IsOnline` - bool: Whether character is online
- `IsNPC` - bool: Whether is NPC
- `IsPC` - bool: Whether is player character

**Methods:**
- `InviteToFleet` - Invites to fleet
- `GiveMoney[amount,reason]` - Gives money to this character

**See Also:**
- [pilot](#pilot) - Inherits from being
- [character](#character) - Player character type

---

#### **corporation**

Corporation information.

**Members:**
- `ID` - int64: Corporation ID
- `Name` - string: Corporation name
- `Ticker` - string: Corporation ticker
- `Wallet` - [corporationwallet](#corporationwallet): Corporation wallet

**See Also:**
- [character](#character) - Corp member returns corporation
- [corporationwallet](#corporationwallet) - Corporation wallet datatype

---

#### **wallet**

Player wallet information.

**Members:**
- `Balance` - double: ISK balance
- `BalanceAUR` - int: AURUM balance (legacy)

**See Also:**
- [character](#character) - Wallet member returns wallet

---

#### **corporationwallet**

Corporation wallet information.

**Members:**
- `Balance` - double: Corporation ISK balance

**See Also:**
- [corporation](#corporation) - Parent corporation datatype

---

#### **standing**

Standing information between entities (player, corp, alliance).

**Members:**

*Player to Others:*
- `MeToAlliance[id]` - double: Your standing to alliance
- `MeToCorp[id]` - double: Your standing to corporation
- `MeToPilot[id]` - double: Your standing to pilot

*Corporation to Others:*
- `CorpToAlliance[id]` - double: Your corp's standing to alliance
- `CorpToCorp[id]` - double: Your corp's standing to corporation
- `CorpToPilot[id]` - double: Your corp's standing to pilot

*Alliance to Others:*
- `AllianceToAlliance[id]` - double: Your alliance's standing to alliance
- `AllianceToCorp[id]` - double: Your alliance's standing to corporation
- `AllianceToPilot[id]` - double: Your alliance's standing to pilot

**See Also:**
- [character](#character) - Character standings
- [corporation](#corporation) - Corporation information

---

### Entity & Space DataTypes

#### **entity**

Represents objects in space (ships, NPCs, gates, stations, asteroids, wrecks, drones, etc.).

**Members:**

*Identity:*
- `ID` - int64: Entity ID
- `Name` - string: Entity name
- `Type` - string: Entity type name
- `TypeID` - int: Entity type ID
- `Group` - string: Entity group name
- `GroupID` - int: Entity group ID
- `Category` - string: Entity category name
- `CategoryID` - int: Entity category ID

*Ownership:*
- `Owner` - string: Owner name
- `OwnerID` - int64: Owner character ID
- `Corp` - string: Corporation name
- `Alliance` - string: Alliance name
- `AllianceID` - int: Alliance ID
- `AllianceTicker` - string: Alliance ticker
- `CharID` - int64: Character ID (for player ships)
- `Security` - float: Security status

*Position:*
- `X` - double: X coordinate
- `Y` - double: Y coordinate
- `Z` - double: Z coordinate
- `vX` - double: X velocity component
- `vY` - double: Y velocity component
- `vZ` - double: Z velocity component

*Distance & Movement:*
- `Distance` - double: Distance from player
- `Distance2` - double: Distance squared (faster for comparisons)
- `DistanceTo` - double: Distance to specific coordinates (requires parameters)
- `Velocity` - double: Current velocity
- `MaxVelocity` - double: Maximum velocity
- `AngularVelocity` - double: Angular velocity
- `RadialVelocity` - double: Radial velocity
- `TransversalVelocity` - double: Transversal velocity

*Physical Properties:*
- `Mass` - double: Entity mass
- `Radius` - double: Entity radius
- `Pitch` - double: Pitch angle
- `Roll` - double: Roll angle
- `Yaw` - double: Yaw angle

*Combat & Status:*
- `ShieldPct` - double: Shield percentage
- `ArmorPct` - double: Armor percentage
- `StructurePct` - double: Structure percentage
- `Bounty` - double: NPC bounty value

*Targeting:*
- `IsActiveTarget` - bool: Whether is active target
- `IsLockedTarget` - bool: Whether is locked target
- `IsTargetingMe` - bool: Whether targeting player
- `BeingTargeted` - bool: Whether being targeted
- `IsWarpScrambled` - bool: Whether entity is warp scrambled
- `IsWarpScramblingMe` - bool: Whether warp scrambling player
- `IsJammingMe` - bool: Whether jamming player (any type)
- `IsTargetJammingMe` - bool: Whether target jamming player

*Type Checks:*
- `IsNPC` - bool: Whether is NPC
- `IsPC` - bool: Whether is player character
- `IsCelestial` - bool: Whether is celestial object
- `IsGlobal` - bool: Whether is global entity
- `IsMassive` - bool: Whether is massive object
- `IsInteractive` - bool: Whether is interactive
- `IsMoribund` - bool: Whether is moribund (dying)
- `IsCloaked` - bool: Whether is cloaked

*Relationships:*
- `IsFleetMember` - bool: Whether is fleet member
- `IsOwnedByCorpMember` - bool: Whether owned by corp member
- `IsOwnedByAllianceMember` - bool: Whether owned by alliance member

*Wrecks & Loot:*
- `IsAbandoned` - bool: Whether is abandoned
- `HaveLootRights` - bool: Whether player has loot rights
- `IsWreckEmpty` - bool: Whether wreck is empty
- `IsWreckViewed` - bool: Whether wreck has been viewed
- `WreckID` - int64: Wreck ID

*Cargo:*
- `CargoCapacity` - double: Cargo capacity
- `UsedCargoCapacity` - double: Used cargo capacity
- `HasOreHold` - bool: Whether has ore hold
- `CargoWindow` - [evewindow](#evewindow): Cargo window
- `LootWindow` - [evewindow](#evewindow): Loot window

*Scanning:*
- `HasCargoScannerResults` - bool: Whether cargo scan results available
- `HasShipScannerResults` - bool: Whether ship scan results available
- `ShipScannerCapacitorCapacity` - double: Scanned ship capacitor capacity
- `ShipScannerCapacitorCharge` - double: Scanned ship capacitor charge
- `SurveyScannerOreQuantity` - int64: Survey scan ore quantity

*Fleet & Formation:*
- `FleetTag` - string: Fleet tag
- `FormationID` - int: Formation ID
- `Following` - [entity](#entity): Entity being followed
- `FollowRange` - double: Follow range
- `Approaching` - [entity](#entity): Entity being approached
- `Mode` - int: Entity mode

*POS & Structures:*
- `IsPOS` - bool: Whether is player-owned starbase
- `POSState` - string: POS state (anchored, online, etc.)
- `IsDockable` - bool: Whether dockable (citadels/structures)

*Wormholes:*
- `WormHoleAge` - int: Wormhole age
- `WormHoleSize` - float: Wormhole size
- `WormHoleClass` - int: Wormhole class

*Type Conversions:*
- `ToActiveDrone` - [activedrone](#activedrone): Converts to activedrone type
- `ToAttacker` - [attacker](#attacker): Converts to attacker type
- `ToFleetMember` - [fleetmember](#fleetmember): Converts to fleetmember type
- `ToJammer` - [jammer](#jammer): Converts to jammer type
- `ToWormhole` - [entitywormhole](#entitywormhole): Converts to wormhole type

**Methods:**

*Basic Interaction:*
- `Set[entityID]` - Sets entity reference by ID
- `Activate` - Activates entity (gates, etc.)
- `Dock` - Docks with entity (station/structure)
- `Jump` - Jumps through entity (stargate)
- `Open` - Opens entity (cargo container, wreck, etc.)
- `AccessCustomsOffice` - Opens customs office

*Navigation:*
- `Approach` - Approaches entity
- `AlignTo` - Aligns to entity
- `Orbit[distance]` - Orbits entity at distance
- `KeepAtRange[distance]` - Keeps at range from entity
- `WarpTo[distance]` - Warps to entity at distance
- `WarpTo[distance,fleet]` - Warps to entity (fleet warp if fleet=TRUE)
- `WarpFleetTo[distance]` - Fleet warps to entity

*Targeting:*
- `LockTarget` - Locks entity as target
- `UnlockTarget` - Unlocks entity
- `MakeActiveTarget` - Makes entity active target
- `SetAsSelectedItem` - Sets as overview selected item

*Scanning:*
- `GetCargoScannerResults[index:item]` - Populates index with cargo scan [item](#item) results
- `GetShipScannerResults[index:item]` - Populates index with ship scan [item](#item) results

*Drones:*
- `GetActiveDrones[index:activedrone]` - Populates index with active [activedrone](#activedrone) objects
- `Mine` - Commands drones to mine entity
- `MineRepeatedly` - Commands drones to mine repeatedly
- `EngageMyTarget` - Commands drones to engage target
- `ReturnAndOrbit` - Commands drones to return and orbit
- `ReturnToDroneBay` - Returns drones to bay
- `ReturnFighterControl` - Returns fighter control
- `ScoopToCargoBay` - Scoops to cargo bay
- `ScoopToDroneBay` - Scoops to drone bay
- `ScoopToCargoHold` - Scoops to cargo hold
- `ScoopToShipMaintenanceBay` - Scoops to ship maintenance bay
- `Abandon` - Abandons drone
- `AbandonAll` - Abandons all drones
- `AbandonDrone` - Abandons specific drone
- `DroneAssist[charID]` - Assists character with drones
- `DroneGuard[charID]` - Guards character with drones
- `DelegateFighterControl[charID]` - Delegates fighter control

*Other:*
- `CreateBookmark[label,notes,folder,expiry]` - Creates bookmark
- `SetFleetTag[tag]` - Sets fleet tag
- `SetName[name]` - Sets entity name
- `MarkWreckViewed` - Marks wreck as viewed

**See Also:**
- [entitywormhole](#entitywormhole) - Wormhole-specific entity
- [entityplayerstructure](#entityplayerstructure) - Player structure entity
- [attacker](#attacker) - Attacker entity
- [jammer](#jammer) - Jammer entity
- [activedrone](#activedrone) - Active drone
- [character](#character) - Player character
- [overview](#overview) - Overview selection

---

#### **entitywormhole**

Wormhole entity. Inherits from [**entity**](#entity).

**Inheritance:**
- All members and methods from [**entity**](#entity) are available

**Members:**
- `Age` - int: Wormhole age (-10 on error)
- `Size` - float: Wormhole size (-10.0 on error)
- `Class` - int: Wormhole class (-10 on error)

**Methods:**
- `EnterWormhole` - Enters the wormhole

**See Also:**
- [entity](#entity) - Parent type

---

#### **entityplayerstructure**

Player-owned structure entity (POS modules, etc.). Inherits from [**entity**](#entity).

**Inheritance:**
- All members and methods from [**entity**](#entity) are available

**Members:**
- `Anchored` - bool: Whether structure is anchored
- `CanAnchorAt` - bool: Whether can anchor at current location
- `CanAssumeControl` - bool: Whether can assume control
- `CanOffline` - bool: Whether can offline
- `CanOnline` - bool: Whether can online
- `CanUnanchor` - bool: Whether can unanchor
- `ControllerID` - int64: ID of controlling character
- `ControllerName` - string: Name of controlling character
- `CurrentTargetID` - int64: ID of current target
- `Online` - bool: Whether structure is online
- `Orphaned` - bool: Whether structure is orphaned
- `State` - string: Structure state
- `ToTower` - [entity](#entity): Converts to tower entity

**Methods:**
- `Anchor` - Anchors the structure
- `AssumeControl` - Assumes control of structure
- `ReleaseControl` - Releases control of structure
- `UnlockTarget` - Unlocks current target

**See Also:**
- [entity](#entity) - Parent type

---

#### **attacker**

Entity that is attacking you. Inherits from [**entity**](#entity).

**Inheritance:**
- All members and methods from [**entity**](#entity) are available

**Additional Members:**
- `ID` - int64: Attacker ID
- `IsCurrentlyAttacking` - bool: Whether currently attacking
- `ToJammer` - [jammer](#jammer): Converts to jammer type if applicable

**Additional Methods:**
- `GetAttacks[index:attack]` - Populates index with [attack](#attack) objects

**See Also:**
- [entity](#entity) - Parent type
- [jammer](#jammer) - Jammer entity
- [attack](#attack) - Attack instance

---

#### **attack**

Represents an attack instance.

**Members:**
- `ID` - int64: Attack ID
- `Name` - string: Attack name
- `TimeStarted` - [evetime](#evetime): Time attack started

---

#### **jammer**

Entity that is jamming you. Inherits from [**attacker**](#attacker) which inherits from [**entity**](#entity).

**Inheritance:**
- All members and methods from [**attacker**](#attacker) (and [**entity**](#entity)) are available

**See Also:**
- [attacker](#attacker) - Parent type
- [entity](#entity) - Grandparent type

---

#### **overview**

Overview functionality.

**Members:**
- `SelectedItem` - [entity](#entity): Currently selected entity in overview

**Methods:**
- `ClearSelectedItem` - Clears overview selection

**See Also:**
- [entity](#entity) - Entities shown in overview

---

### Item & Equipment DataTypes

#### **item**

Item instance in game. Inherits from [**iteminfo**](#iteminfo).

**Inheritance:**
- All members from [**iteminfo**](#iteminfo) are available

**Additional Members:**
- `Category` - string: Item category name
- `CategoryID` - int: Item category ID
- `OwnerID` - int64: Owner character ID
- `Location` - string: Location name
- `LocationID` - int64: Location ID
- `Quantity` - int: Item quantity
- `ID` - int64: Unique item ID
- `Slot` - string: Slot name
- `SlotID` - int: Slot ID
- `IsRepackable` - bool: Whether can be repackaged
- `GivenName` - string: Custom given name (deprecated - use Name)
- `Name` - string: Item name (custom or type name)
- `CargoCapacity` - double: Cargo capacity
- `UsedCargoCapacity` - double: Used cargo capacity
- `MaxFlightTime` - double: Max flight time (missiles/charges)
- `MaxVelocity` - double: Max velocity
- `ExplosionRadius` - double: Explosion radius
- `ExplosionVelocity` - double: Explosion velocity
- `SignatureRadiusBonus` - double: Signature radius bonus
- `EMDamage` - double: EM damage
- `KineticDamage` - double: Kinetic damage
- `ThermalDamage` - double: Thermal damage
- `ExplosiveDamage` - double: Explosive damage
- `FallofMultiplier` - double: Falloff multiplier
- `TrackingSpeedMultiplier` - double: Tracking speed multiplier
- `MetaLevel` - int: Meta level
- `IsInsured` - bool: Whether ship is insured
- `InsuranceLevel` - string: Insurance level
- `BookmarkID` - int64: Bookmark ID (for bookmark items)

**Methods:**
- `Jettison` - Jettisons item
- `MoveTo[location,destination,quantity,folder]` - Moves item
- `LaunchDrones` - Launches drones
- `LaunchForSelf` - Launches deployable for self
- `LaunchForCorp[ignoreWarnings]` - Launches deployable for corp
- `MakeActive` - Makes ship active
- `LeaveShip` - Leaves current ship (board this one)
- `Repackage` - Repackages item
- `Compress` - Compresses ore
- `AssembleShip` - Assembles packaged ship
- `AssembleContainer` - Assembles packaged container
- `Refine` - Refines ore
- `Reprocess` - Reprocesses item
- `TrainSkill` - Trains skill book
- `InjectSkill` - Injects skill injector
- `AddToSellOrder` - Opens sell order window
- `Open` - Opens item (container/ship)
- `ConsumeBooster` - Consumes booster
- `PluginImplant` - Plugs in implant
- `ApplyPilotLicense` - Applies pilot license
- `GetInsuranceQuotes[index:???]` - Gets insurance quotes
- `Insure[level,cost]` - Insures ship
- `GetContrabandFactions[index:???]` - Gets contraband factions
- `GetRepairQuote` - Gets repair quote
- `UseAbyssalFilament` - Uses abyssal filament

**See Also:**
- [iteminfo](#iteminfo) - Parent type with static info
- [module](#module) - Inherits from item
- [ship](#ship) - Ship cargo and items

---

#### **iteminfo**

Static information about item types (from EVE database).

**Members:**
- `Name` - string: Item type name
- `Type` - string: Item type name (alias for Name)
- `TypeID` - int: Item type ID
- `Group` - string: Group name
- `GroupID` - int: Group ID
- `IsContraband` - bool: Whether item is contraband
- `GraphicID` - int: Graphic ID
- `Capacity` - double: Item capacity
- `Radius` - double: Item radius
- `RaceID` - int: Race ID
- `Volume` - double: Item volume
- `BasePrice` - double: Base price
- `PortionSize` - int: Portion size
- `MarketGroupID` - int: Market group ID
- `Description` - string: Item description
- `ChargeSize` - int: Charge size
- `RangeBonus` - float: Range bonus
- `ShieldRadius` - int: Shield radius

**See Also:**
- [item](#item) - Item instances (inherit from iteminfo)
- [iteminfolist](#iteminfolist) - List item with quantity

---

#### **iteminfolist**

Item info with quantity (used in various lists). Inherits from [**iteminfo**](#iteminfo).

**Inheritance:**
- All members from [**iteminfo**](#iteminfo) are available

**Additional Members:**
- `ID` - int: Type ID (same as TypeID for convenience)
- `TypeID` - int: Item type ID
- `Quantity` - int64: Quantity

**See Also:**
- [iteminfo](#iteminfo) - Parent type

---

#### **module**

Ship module. Inherits from [**item**](#item).

**Inheritance:**
- All members and methods from [**item**](#item) (and [**iteminfo**](#iteminfo)) are available

**Additional Members:**

*State:*
- `IsGoingOnline` - bool: Whether going online
- `IsWaitingForActiveTarget` - bool: Whether waiting for active target
- `IsReloadingAmmo` - bool: Whether reloading ammo
- `IsReloading` - bool: Whether reloading
- `IsOnline` - bool: Whether online
- `IsActivatable` - bool: Whether can be activated
- `IsAutoReloadOn` - bool: Whether auto-reload enabled
- `IsActive` - bool: Whether active
- `IsOffensive` - bool: Whether offensive module
- `IsAssistance` - bool: Whether assistance module
- `IsDeactivating` - bool: Whether deactivating
- `IsBeingRepaired` - bool: Whether being repaired
- `AutoRepeat` - bool: Whether auto-repeat enabled
- `IsBlinking` - bool: Whether blinking
- `IsBankSlave` - bool: Whether bank slave
- `IsBankMaster` - bool: Whether bank master

*Charge & Ammo:*
- `EffectCategory` - string: Effect category
- `Charge` - [modulecharge](#modulecharge): Current charge
- `CurrentCharges` - int: Current charges
- `MaxCharges` - int: Max charges
- `ChargeSize` - int: Charge size

*Targeting:*
- `Target` - [entity](#entity): Current target
- `TargetID` - int64: Target ID
- `LastTarget` - [entity](#entity): Last target
- `LastTargeted` - [entity](#entity): Last targeted entity
- `LastTargetedID` - int64: Last targeted ID

*Requirements & Usage:*
- `ToItem` - [item](#item): Converts to item type
- `PowergridUsage` - double: Powergrid usage
- `OptimalRange` - double: Optimal range
- `TechLevel` - int: Tech level
- `ActivationCost` - double: Activation cost (capacitor)
- `HP` - double: Hit points
- `Damage` - double: Damage taken
- `ActivationTime` - double: Activation time (ms)
- `Duration` - double: Duration (ms)
- `Volume` - double: Volume
- `CPUUsage` - double: CPU usage
- `Capacity` - double: Capacity
- `Mass` - double: Mass

*Weapon Stats:*
- `AccuracyFalloff` - double: Accuracy falloff
- `EffectivenessFalloff` - double: Effectiveness falloff
- `TrackingSpeed` - double: Tracking speed
- `DamageModifier` - double: Damage modifier
- `EMDamage` - double: EM damage
- `KineticDamage` - double: Kinetic damage
- `ThermalDamage` - double: Thermal damage
- `ExplosiveDamage` - double: Explosive damage
- `SignatureResolution` - double: Signature resolution
- `RateOfFire` - double: Rate of fire
- `OverloadRateOfFireBonus` - double: Overload rate of fire bonus
- `OverloadOptimalRangeBonus` - double: Overload optimal range bonus
- `RateOfFireBonus` - double: Rate of fire bonus

*Mining:*
- `MiningAmount` - double: Mining amount
- `MiningAmountPerSecond` - double: Mining amount per second
- `MiningAmountBonus` - double: Mining amount bonus
- `CrystalsDamage` - double: Crystal damage
- `SpecialtyCrystalMiningAmount` - double: Specialty crystal mining amount
- `TargetGroup` - int: Target group
- `SurveyScanRange` - double: Survey scan range
- `UsesFrequencyCrystals` - bool: Uses frequency crystals

*Repair & Tank:*
- `OverloadRepairBonus` - double: Overload repair bonus
- `OverloadDurationBonus` - double: Overload duration bonus
- `ArmorHPRepaired` - double: Armor HP repaired
- `ExplosiveDmgResistanceBonus` - double: Explosive damage resistance bonus
- `KineticDmgResistanceBonus` - double: Kinetic damage resistance bonus
- `ThermalDmgResistanceBonus` - double: Thermal damage resistance bonus
- `EMDmgResistanceBonus` - double: EM damage resistance bonus
- `ArmorHPBonus` - double: Armor HP bonus
- `ShieldBonus` - double: Shield bonus
- `ShieldHPBonus` - double: Shield HP bonus

*Propulsion:*
- `MaxVelocityBonus` - double: Max velocity bonus
- `MaxVelocityPenalty` - double: Max velocity penalty
- `OverloadSpeedFactorBonus` - double: Overload speed factor bonus
- `MassAddition` - double: Mass addition
- `Thrust` - double: Thrust
- `VelocityModifier` - double: Velocity modifier

*Ship Bonuses:*
- `StructureHPBonus` - double: Structure HP bonus
- `CargoCapacityBonus` - double: Cargo capacity bonus
- `CapacitorRechargeRateBonus` - double: Capacitor recharge rate bonus
- `ShieldRechargeRateBonus` - double: Shield recharge rate bonus
- `CapacitorBonus` - double: Capacitor bonus
- `PowergridBonus` - double: Powergrid bonus
- `CPUOutputBonus` - double: CPU output bonus
- `CPUPenaltyPercent` - double: CPU penalty percent

*Scan & ECM:*
- `ScanResolutionBonus` - double: Scan resolution bonus
- `SensorRecalibrationTime` - double: Sensor recalibration time
- `WarpScrambleStrength` - int: Warp scramble strength

*Energy/Shield Transfer:*
- `EnergyTransferAmount` - double: Energy transfer amount
- `PowerTransferAmount` - double: Power transfer amount
- `TransferRange` - double: Transfer range
- `PowerTransferRange` - double: Power transfer range
- `ShieldTransferRange` - double: Shield transfer range

*Neuts/Nos:*
- `MaxNeutralizationRange` - double: Max neutralization range
- `EnergyDestabilizationRange` - double: Energy destabilization range
- `EnergyNeutralized` - double: Energy neutralized
- `EnergyDestabilizationAmount` - double: Energy destabilization amount

*Other:*
- `AccessDifficultyBonus` - double: Access difficulty bonus
- `MaxTractorVelocity` - double: Max tractor velocity
- `HeatDamage` - double: Heat damage
- `ChargeRate` - double: Charge rate
- `DefaultEffectName` - string: Default effect name
- `DefaultEffectDescription` - string: Default effect description
- `ReactivationDelay` - double: Reactivation delay

**Additional Methods:**
- `Click` - Clicks module (for targeted modules)
- `Activate` - Activates module
- `Deactivate` - Deactivates module
- `Reload` - Reloads module
- `ChangeAmmo[itemID,quantity]` - Changes ammo
- `SetAutoReloadOn` - Enables auto-reload
- `SetAutoReloadOff` - Disables auto-reload
- `PutOnline` - Puts module online
- `PutOffline` - Puts module offline
- `GetAvailableAmmo[index:item]` - Populates index with available ammo
- `ToggleOverload` - Toggles overload
- `SetManualOn` - Sets manual mode on
- `SetManualOff` - Sets manual mode off
- `UnloadToCargo` - Unloads charge to cargo
- `ReloadAll` - Reloads all modules
- `Repair` - Repairs module
- `CancelRepair` - Cancels repair

**See Also:**
- [item](#item) - Parent type
- [modulecharge](#modulecharge) - Module charge
- [ship](#ship) - Ship modules
- [fittingslot](#fittingslot) - Fitting slot

---

#### **modulecharge**

Charge/ammunition loaded in a module.

**Members:**
- `ID` - int64: Charge ID
- `Type` - string: Charge type name
- `TypeID` - int: Charge type ID
- `Group` - string: Group name
- `GroupID` - int: Group ID
- `Category` - string: Category name
- `CategoryID` - int: Category ID
- `Location` - string: Location name
- `LocationID` - int64: Location ID
- `Slot` - string: Slot name
- `SlotID` - int: Slot ID
- `Quantity` - int: Quantity
- `MaxFlightTime` - double: Max flight time
- `MaxVelocity` - double: Max velocity
- `Volume` - double: Volume
- `ChargeSize` - int: Charge size

**See Also:**
- [module](#module) - Parent module

---

### Ship DataTypes

#### **ship**

Player's active ship.

**Members:**

*Identity:*
- `ID` - int64: Ship ID
- `Name` - string: Ship name
- `ToItem` - [item](#item): Converts to item type
- `ToEntity` - [entity](#entity): Converts to entity type

*Cargo & Inventory:*
- `CargoCapacity` - double: Cargo capacity
- `UsedCargoCapacity` - double: Used cargo capacity
- `HasOreHold` - bool: Whether has ore hold
- `Cargo[# or name]` - [item](#item): Cargo item by index or name
- `Module[# or name]` - [module](#module): Module by slot or name
- `Drone[#]` - [item](#item): Drone in bay by index

*Drones:*
- `DroneBandwidth` - double: Drone bandwidth
- `DronebayCapacity` - double: Drone bay capacity
- `UsedDronebayCapacity` - double: Used drone bay capacity

*Capacitor:*
- `Capacitor` - double: Current capacitor
- `MaxCapacitor` - double: Maximum capacitor
- `CapacitorPct` - double: Capacitor percentage
- `CapacitorRechargeRate` - double: Capacitor recharge rate (ms)

*Defense:*
- `Shield` - double: Current shield HP
- `MaxShield` - double: Maximum shield HP
- `ShieldPct` - double: Shield percentage
- `ShieldRechargeRate` - double: Shield recharge rate (ms)
- `Armor` - double: Current armor HP
- `MaxArmor` - double: Maximum armor HP
- `ArmorPct` - double: Armor percentage
- `Structure` - double: Current structure HP
- `MaxStructure` - double: Maximum structure HP
- `StructurePct` - double: Structure percentage

*Resources:*
- `CPULoad` - double: CPU used
- `CPUOutput` - double: CPU available
- `PowerLoad` - double: Powergrid used
- `PowerOutput` - double: Powergrid available

*Slots:*
- `HighSlots` - int: Number of high slots
- `MediumSlots` - int: Number of medium slots
- `LowSlots` - int: Number of low slots
- `RigSlots` - int: Number of rig slots
- `RigSlotsLeft` - int: Number of empty rig slots
- `TurretSlotsLeft` - int: Number of empty turret slots
- `LauncherSlotsLeft` - int: Number of empty launcher slots

*Heat:*
- `HeatHigh` - double: High slot heat
- `HeatMedium` - double: Medium slot heat
- `HeatLow` - double: Low slot heat
- `HeatCapacityHigh` - double: High slot heat capacity
- `HeatCapacityMedium` - double: Medium slot heat capacity
- `HeatCapacityLow` - double: Low slot heat capacity

*Attributes:*
- `MaxVelocity` - double: Max velocity
- `Agility` - double: Agility
- `Radius` - double: Radius
- `TechLevel` - int: Tech level
- `SignatureRadius` - double: Signature radius
- `WarpSpeedMultiplier` - double: Warp speed multiplier

*Targeting & Scanning:*
- `MaxLockedTargets` - int: Max locked targets
- `MaxTargetRange` - double: Max target range
- `ScanSpeed` - double: Scan speed
- `ScanResolution` - double: Scan resolution
- `ScanRadarStrength` - double: Scan radar strength

*Scanners:*
- `Scanners` - [scanners](#scanners): Ship scanners

**Methods:**
- `StripFitting` - Strips all modules from ship
- `LaunchAllDrones` - Launches all drones
- `GetModules[index:module]` - Populates index with modules
- `GetRigs[index:item]` - Populates index with rigs
- `GetDrones[index:item]` - Populates index with drones in bay
- `Open` - Opens ship
- `Approach[target,distance]` - Approaches target
- `Align[target]` - Aligns to target
- `SelfDestruct` - Initiates self-destruct
- `SetStarbaseForcefieldPassword[password]` - Sets POS forcefield password
- `Jettison[index:item or int64]` - Jettisons items

**See Also:**
- [module](#module) - Ship modules
- [item](#item) - Ship as item
- [entity](#entity) - Ship as entity
- [scanners](#scanners) - Ship scanners
- [item](#item) - Ship cargo items
- [entity](#entity) - Drones in space
- [character](#character) - Character.Ship member

---

### Station & Structure DataTypes

#### **station**

Station information and methods.

**Members:**

*Identity:*
- `Name` - string: Station name
- `ID` - int64: Station ID
- `TypeID` - int: Station type ID
- `Type` - string: Station type name

*Ownership:*
- `OwnerID` - int64: Owner ID
- `Owner` - string: Owner name
- `OwnerTypeID` - int: Owner type ID
- `OwnerType` - string: Owner type

*Location:*
- `SolarSystem` - [solarsystem](#solarsystem): Station's solar system

*Cargo:*
- `Cargo` - [cargo](#cargo): Station cargo hold

**Methods:**

*Hangar:*
- `GetHangarItems[index:item]` - Populates index with [item](#item) objects in station hangar
- `GetHangarShips[index:item]` - Populates index with ships in station hangar
- `GetCorpHangarItems[index:item,division]` - Populates index with corp hangar items
- `GetCorpHangarShips[index:item,division]` - Populates index with corp hangar ships
- `GetRepairableItems[index:item]` - Populates index with repairable [item](#item) objects

*Navigation:*
- `SetDestination` - Sets station as autopilot destination
- `AddWaypoint` - Adds station as waypoint
- `ClearWaypoint` - Clears station waypoint

*Windows:*
- `OpenFitting` - Opens fitting window

**See Also:**
- [structure](#structure) - Inherits from station
- [character](#character) - Character.Station member
- [solarsystem](#solarsystem) - Solar system location

---

#### **structure**

Structure/citadel. Inherits from [**station**](#station).

**Inheritance:**
- All members and methods from [**station**](#station) are available

**Members:**
- `CanBoard` - bool: Whether player can board structure
- `MyCorpHasOffice` - bool: Whether player's corp has office

**Methods:**
- `Board` - Boards the structure

**See Also:**
- [station](#station) - Parent type
- [entity](#entity) - Structure entities in space

---

### Window DataTypes

#### **evewindow**

Base type for all EVE windows.

**Members:**

*Identity:*
- `Name` - string: Window name
- `Caption` - string: Window caption
- `Text` - string: Window text
- `HTML` - string: Window HTML content

*State:*
- `Minimized` - bool: Whether window is minimized

*Position & Size:*
- `X` - int: Window X position
- `Y` - int: Window Y position
- `Width` - int: Window width
- `Height` - int: Window height

*Buttons:*
- `NumButtons` - int: Number of visible, enabled buttons
- `Button[#]` - [eveuibutton](#eveuibutton): Button by number (1 to NumButtons)
- `Button[text]` - [eveuibutton](#eveuibutton): Button by label text

**Methods:**

*Window Actions:*
- `Close` - Closes window
- `Minimize` - Minimizes window
- `Maximize` - Maximizes window
- `StackAll` - Stacks all items (for loot windows)
- `LootAll` - Loots all items (for loot windows)

*Button Clicks:*
- `ClickButtonOK` - Clicks OK button
- `ClickButtonCancel` - Clicks Cancel button
- `ClickButtonYes` - Clicks Yes button
- `ClickButtonNo` - Clicks No button
- `ClickButtonClose` - Clicks Close button

**See Also:**
- All specialized window types inherit from evewindow

---

#### **eveinvwindow**

Inventory window. Inherits from [**evewindow**](#evewindow).

**Inheritance:**
- All members and methods from [**evewindow**](#evewindow) are available

**Additional Members:**
- `ActiveChild` - [eveinvchildwindow](#eveinvchildwindow): Currently active child window
- `ChildWindow[index or name]` - [eveinvchildwindow](#eveinvchildwindow): Child window by index or name
- `IsInRange` - bool: Whether inventory is in range
- `HasCapacity` - bool: Whether inventory has capacity
- `ItemID` - int64: Inventory container item ID
- `LocationFlagID` - int: Location flag ID
- `Capacity` - double: Total capacity (-1 if error, -2 if not ready)
- `UsedCapacity` - double: Used capacity (-1 if error, -2 if not ready)
- `LocationFlag` - string: Location flag name

**Methods:**
- `GetChildren[index:eveinvchildwindow]` - Populates index with [eveinvchildwindow](#eveinvchildwindow) objects

**See Also:**
- [eveinvchildwindow](#eveinvchildwindow) - Child inventory windows
- [evewindow](#evewindow) - Parent type

---

#### **eveinvchildwindow**

Child inventory window (tab within inventory window).

**Members:**
- `Name` - string: Child window name
- `Capacity` - double: Total capacity (-1 if error, -2 if not ready)
- `UsedCapacity` - double: Used capacity (-1 if error, -2 if not ready)
- `LocationFlag` - string: Location flag name
- `LocationFlagID` - int: Location flag ID
- `IsInRange` - bool: Whether inventory is in range
- `ItemID` - int64: Inventory container item ID
- `HasCapacity` - bool: Whether inventory has capacity

**Methods:**
- `MakeActive` - Makes child window active (switches tab)
- `OpenAsNewWindow` - Opens child as new window
- `GetItems[index:item]` - Populates index with [item](#item) objects
- `StackAll` - Stacks all items in inventory
- `MoveTo[destID,destFlagID]` - Moves all items to destination

**See Also:**
- [eveinvwindow](#eveinvwindow) - Parent inventory window
- [item](#item) - Items in inventory

---

#### **evefittingwindow**

Fitting window. Inherits from [**evewindow**](#evewindow).

**Inheritance:**
- All members and methods from [**evewindow**](#evewindow) are available

**Members:**

*Ship Stats:*
- `CPU` - string: CPU usage/total (e.g., "78.4/80.4")
- `Power` - string: Power usage/total
- `Calibration` - string: Calibration usage/total

*Status:*
- `IsShipSimulated` - bool: Whether showing simulated ship

*Slots:*
- `Slot[id]` - [fittingslot](#fittingslot): Slot by ID number
- `Slot[name]` - [fittingslot](#fittingslot): Slot by name (e.g., "HiSlot0")

**Methods:**
- `GetSlots[index:fittingslot]` - Populates index with available [fittingslot](#fittingslot) objects
- `StripFitting` - Strips all fittings from ship

**See Also:**
- [fittingslot](#fittingslot) - Fitting slots
- [module](#module) - Fitted modules
- [evewindow](#evewindow) - Parent type

---

#### **evesellitemswindow**

Sell items window. Inherits from [**evewindow**](#evewindow).

**Inheritance:**
- All members and methods from [**evewindow**](#evewindow) are available

**Members:**

*Order Info:*
- `Duration` - int: Order duration in days
- `RemainingOrders` - int: Remaining market orders
- `BrokersFee` - string: Broker's fee
- `SalesTax` - string: Sales tax
- `TotalAmount` - string: Total amount

*Items:*
- `NumItems` - int: Number of items to sell
- `Item[#]` - [sellitem](#sellitem): Item by index (1 to NumItems)

**Methods:**
- `SetDuration[days]` - Sets order duration (0,1,3,7,14,30,90)
- `Sell` - Sells items (requires confirmation checkbox)

**See Also:**
- [sellitem](#sellitem) - Items being sold
- [evewindow](#evewindow) - Parent type

---

#### **evemarketactionwindow**

Market buy order window. Inherits from [**evewindow**](#evewindow).

**Inheritance:**
- All members and methods from [**evewindow**](#evewindow) are available

**Members:**

*Price Info:*
- `BidPrice` - [eveuisinglelineedit](#eveuisinglelineedit): Bid price field
- `BidPricePercentageComparison` - [eveuilabel](#eveuilabel): Price comparison label
- `RegionalAverage` - double: Regional average price
- `BestRegional` - [eveuilabel](#eveuilabel): Best regional price
- `BestMatchable` - [eveuilabel](#eveuilabel): Best matchable price

*Order Details:*
- `Quantity` - [eveuisinglelineedit](#eveuisinglelineedit): Quantity field
- `QuantityMin` - [eveuisinglelineedit](#eveuisinglelineedit): Minimum quantity field
- `Duration` - [eveuicombo](#eveuicombo): Duration combo box
- `Range` - [eveuicombo](#eveuicombo): Range combo box

*Costs:*
- `Fee` - [eveuisinglelineedit](#eveuisinglelineedit): Fee field
- `Total` - [eveuilabel](#eveuilabel): Total cost

*Status:*
- `IsReady` - bool: Whether window is ready

**Methods:**
- `Buy` - Places buy order
- `Close` - Closes the window

**See Also:**
- [evewindow](#evewindow) - Parent type
- UI element types for field access

---

#### **everepairshopwindow**

Repair shop window. Inherits from [**evewindow**](#evewindow).

**Inheritance:**
- All members and methods from [**evewindow**](#evewindow) are available

**Members:**
- `AverageDamage` - string: Average damage percentage
- `TotalCost` - string: Total repair cost

**Methods:**
- `RepairAll` - Repairs all items

**See Also:**
- [evewindow](#evewindow) - Parent type
- [item](#item) - GetRepairQuote method

---

#### **eveagentdialogwindow**

Agent dialog window. Inherits from [**evewindow**](#evewindow).

**Inheritance:**
- All members and methods from [**evewindow**](#evewindow) are available

**Members:**
- `BriefingHTML` - string: Mission briefing HTML
- `ObjectivesHTML` - string: Mission objectives HTML

**See Also:**
- [evewindow](#evewindow) - Parent type
- [eveagent](#eveagent) - Agent information

---

#### **evemessageboxwindow**

Message box window. Inherits from [**evewindow**](#evewindow).

**Inheritance:**
- All members and methods from [**evewindow**](#evewindow) are available

**See Also:**
- [evewindow](#evewindow) - Parent type

---

#### **evedirectionalscannerwindow**

Directional scanner window. Inherits from [**evewindow**](#evewindow).

**Inheritance:**
- All members and methods from [**evewindow**](#evewindow) are available

**Members:**
- `Range` - float: Scanner range (always in AU)
- `Angle` - double: Scanner angle
- `IsScanning` - bool: Whether currently scanning

**Methods:**
- `GetScanResults[index:directionalscannerresult]` - Populates index with [directionalscannerresult](#directionalscannerresult) objects from last scan

**See Also:**
- [directionalscannerresult](#directionalscannerresult) - Scan results
- [evewindow](#evewindow) - Parent type

---

#### **evecustomsofficewindow**

Customs office window. Inherits from [**evewindow**](#evewindow).

**Inheritance:**
- All members and methods from [**evewindow**](#evewindow) are available

**Members:**
- `TaxRate` - float: Customs office tax rate
- `HeaderTitle` - string: Window header title

**See Also:**
- [evewindow](#evewindow) - Parent type
- [entity](#entity) - AccessCustomsOffice method

---

### UI Element DataTypes

#### **eveuibutton**

UI button element.

**Members:**
- `Name` - string: Button name
- `Text` - string: Button text/label

**Methods:**
- `Press` - Presses the button

**See Also:**
- [evewindow](#evewindow) - Window buttons

---

#### **eveuilabel**

UI label element.

**Members:**
- `Text` - string: Label text

---

#### **eveuisinglelineedit**

UI single-line edit field.

**Members:**
- `Value` - string: Field value

**Methods:**
- `SetValue[value]` - Sets field value

---

#### **eveuicombo**

UI combo box element.

**Members:**
- `Index` - string: Selected index
- `Key` - string: Selected key
- `Value` - string: Selected value

**Methods:**
- `SelectByIndex[#]` - Selects by index
- `SelectByValue[#]` - Selects by value
- `SelectByLabel[label]` - Selects by label

---

### Interstellar DataTypes

#### **interstellar**

Base type for universe objects (systems, constellations, regions, planets, etc.).

**Members:**
- `Name` - string: Object name
- `ID` - int64: Object ID

**See Also:**
- [solarsystem](#solarsystem) - Inherits from interstellar
- [constellation](#constellation) - Inherits from interstellar
- [region](#region) - Inherits from interstellar
- [planet](#planet) - Inherits from interstellar

---

#### **solarsystem**

Solar system. Inherits from [**interstellar**](#interstellar).

**Inheritance:**
- All members and methods from [**interstellar**](#interstellar) are available

**Additional Members:**
- `Security` - float: Security status
- `Faction` - string: Faction name
- `FactionID` - int: Faction ID
- `Constellation` - [constellation](#constellation): Parent constellation
- `Region` - [region](#region): Parent region
- `JumpsTo[solarSystemID]` - int: Jumps to specified solar system
- `JumpsTo[stationID]` - int: Jumps to specified station

**Methods:**
- `SetDestination` - Sets as autopilot destination
- `AddWaypoint` - Adds as waypoint
- `ClearWaypoint` - Clears waypoint
- `GetNumPlanetsByType[collection:int]` - Populates collection with planet type counts (key=TypeID, value=count)
- `GetPlanetIDs[index:int]` - Populates index with planet IDs

**See Also:**
- [interstellar](#interstellar) - Parent type
- [constellation](#constellation) - Parent constellation
- [region](#region) - Parent region
- [planet](#planet) - System planets

---

#### **constellation**

Constellation. Inherits from [**interstellar**](#interstellar).

**Inheritance:**
- All members and methods from [**interstellar**](#interstellar) are available

**Additional Members:**
- `Region` - [region](#region): Parent region

**See Also:**
- [interstellar](#interstellar) - Parent type
- [region](#region) - Parent region
- [solarsystem](#solarsystem) - Child solar systems

---

#### **region**

Region. Inherits from [**interstellar**](#interstellar).

**Inheritance:**
- All members and methods from [**interstellar**](#interstellar) are available

**Additional Members:**
- None (only inherited members)

**See Also:**
- [interstellar](#interstellar) - Parent type
- [constellation](#constellation) - Child constellations

---

#### **planet**

Planet. Inherits from [**interstellar**](#interstellar).

**Inheritance:**
- All members and methods from [**interstellar**](#interstellar) are available

**Additional Members:**
- `Radius` - int: Planet radius
- `Type` - string: Planet type name
- `TypeID` - int: Planet type ID
- `SolarSystem` - [solarsystem](#solarsystem): Parent solar system

**Methods:**
- `GetOrbitalCustomsOffices[index:int64]` - Populates index with orbital customs office entity IDs (only works in space)

**See Also:**
- [interstellar](#interstellar) - Parent type
- [solarsystem](#solarsystem) - Parent solar system
- [evecustomsofficewindow](#evecustomsofficewindow) - Customs office window

---

#### **bookmark**

Bookmark information and navigation.

**Members:**

*Identity:*
- `ID` - int64: Bookmark ID
- `Label` - string: Bookmark label
- `Note` - string: Bookmark notes
- `Type` - string: Bookmark type
- `TypeID` - int: Bookmark type ID

*Ownership:*
- `OwnerID` - int64: Owner ID
- `CreatorID` - int64: Creator ID
- `FolderID` - int64: Folder ID
- `AgentID` - int64: Agent ID (for agent bookmarks)

*Location:*
- `SolarSystemID` - int64: Solar system ID
- `LocationID` - int64: Location ID
- `LocationType` - string: Location type
- `LocationNumber` - int: Location number
- `ItemID` - int64: Item ID
- `X` - double: X coordinate
- `Y` - double: Y coordinate
- `Z` - double: Z coordinate
- `ToEntity` - [entity](#entity): Entity if bookmark is on grid
- `DeadSpace` - bool: Whether bookmark is in deadspace

*Timestamps:*
- `Created` - int64: Creation timestamp
- `DateCreated` - string: Date created
- `TimeCreated` - string: Time created

*Navigation:*
- `JumpsTo` - int: Jumps to bookmark
- `Distance` - double: Distance to bookmark

**Methods:**
- `WarpTo[distance]` - Warps to bookmark at distance
- `WarpFleetTo[distance]` - Warps fleet to bookmark
- `SetDestination` - Sets bookmark as autopilot destination
- `AddWaypoint` - Adds bookmark as waypoint
- `ClearWaypoint` - Clears bookmark waypoint
- `AlignTo` - Aligns to bookmark
- `Approach[distance]` - Approaches bookmark
- `Remove` - Removes bookmark

**See Also:**
- [eve](#eve) - EVE.Bookmark member
- [solarsystem](#solarsystem) - Bookmark location

---

### Market DataTypes

#### **marketorder**

Market order (buy or sell) visible in the market.

**Members:**

*Order Info:*
- `ID` - int64: Order ID
- `Name` - string: Item name
- `TypeID` - int: Item type ID
- `IsSellOrder` - bool: Whether this is a sell order
- `IsBuyOrder` - bool: Whether this is a buy order

*Pricing & Quantity:*
- `Price` - double: Price per unit
- `InitialQuantity` - int: Initial order quantity
- `QuantityRemaining` - int: Quantity remaining
- `MinQuantityToBuy` - int: Minimum quantity to buy

*Timing:*
- `TimeStampWhenIssued` - int64: Timestamp when issued
- `DateWhenIssued` - string: Date when issued
- `TimeWhenIssued` - string: Time when issued
- `Duration` - int: Order duration

*Location:*
- `StationID` - int64: Station ID
- `Station` - string: Station name
- `RegionID` - int64: Region ID
- `Region` - string: Region name
- `SolarSystemID` - int64: Solar system ID
- `SolarSystem` - string: Solar system name
- `Range` - int: Order range
- `Jumps` - int: Jumps from current location

**See Also:**
- [eve](#eve) - GetMarketOrders method
- [myorder](#myorder) - Your own market orders

---

#### **myorder**

Your own market order (buy or sell).

**Members:**

*Order Info:*
- `ID` - int64: Order ID
- `Name` - string: Item name
- `TypeID` - int: Item type ID
- `IsSellOrder` - bool: Whether this is a sell order
- `IsBuyOrder` - bool: Whether this is a buy order
- `IsContraband` - bool: Whether item is contraband
- `IsCorp` - bool: Whether this is a corporation order

*Pricing & Quantity:*
- `Price` - double: Price per unit
- `InitialQuantity` - int: Initial order quantity
- `QuantityRemaining` - int: Quantity remaining
- `MinQuantityToBuy` - int: Minimum quantity to buy

*Timing:*
- `TimeStampWhenIssued` - int64: Timestamp when issued
- `DateWhenIssued` - string: Date when issued
- `TimeWhenIssued` - string: Time when issued
- `Duration` - int: Order duration

*Location:*
- `StationID` - int64: Station ID
- `Station` - string: Station name
- `RegionID` - int64: Region ID
- `Region` - string: Region name
- `SolarSystemID` - int64: Solar system ID
- `SolarSystem` - string: Solar system name
- `Range` - int: Order range

**Methods:**
- `Cancel` - Cancels this order
- `Modify[price,quantity,duration]` - Modifies order price, quantity, and/or duration

**See Also:**
- [character](#character) - GetMyOrders method
- [marketorder](#marketorder) - General market orders

---

#### **sellitem**

Item being sold in sell window.

**Members:**

*Item Info:*
- `Name` - string: Item name
- `ItemID` - int64: Item ID
- `Quantity` - string: Quantity to sell

*Pricing:*
- `AskPrice` - string: Ask price
- `AveragePrice` - double: Average price
- `BestPrice` - double: Best price
- `BestVolumeRemaining` - double: Best volume remaining
- `BestJumps` - int: Jumps to best price

*Costs:*
- `SalesTax` - double: Sales tax
- `TotalAmount` - double: Total amount

**Methods:**
- `SetAskPrice[price]` - Sets ask price
- `SetQuantity[quantity]` - Sets quantity

**See Also:**
- [evesellitemswindow](#evesellitemswindow) - Sell items window
- [item](#item) - Item information

---

### Scanner DataTypes

#### **scanners**

Container datatype providing access to all scanner types.

**Members:**
- `Directional` - [directionalscanner](#directionalscanner): Directional scanner
- `System` - [systemscanner](#systemscanner): System/probe scanner
- `Survey` - [surveyscanner](#surveyscanner): Survey scanner
- `Ship` - [shipscanner](#shipscanner): Ship scanner
- `Cargo` - [cargoscanner](#cargoscanner): Cargo scanner

**See Also:**
- [character](#character) - Access scanners via Me TLO

---

#### **directionalscanner**

Directional scanner (D-scan).

**Methods:**
- `StartScan[angle,range]` - Starts directional scan with angle and range
- `GetScanResults[index:directionalscannerresult]` - Populates index with scan results

**See Also:**
- [directionalscannerresult](#directionalscannerresult) - Scan results

---

#### **systemscanner**

System scanner (probe scanner).

**Members:**
- `IsSensorOverlayActive` - bool: Whether sensor overlay is active

**Methods:**
- `EnableSensorOverlay` - Enables sensor overlay
- `DisableSensorOverlay` - Disables sensor overlay
- `GetAnomalies[index:systemanomaly]` - Populates index with [systemanomaly](#systemanomaly) objects
- `GetSignatures[index:systemsignature]` - Populates index with [systemsignature](#systemsignature) objects

**See Also:**
- [systemanomaly](#systemanomaly) - Combat/data/relic sites
- [systemsignature](#systemsignature) - Wormholes and other signatures
- [systemscannerresult](#systemscannerresult) - Scanner result container

---

#### **systemscannerresult**

System scanner result container (returned from scan operations).

**Methods:**
- `StartScan` - Starts system scan
- `GetScanResults[index:result]` - Retrieves scan results

**See Also:**
- [systemscanner](#systemscanner) - System scanner

---

#### **surveyscanner**

Survey scanner for mining.

**Methods:**
- `StartScan` - Starts survey scan
- `ClearSurveyResults` - Clears survey scan results

**See Also:**
- EVE_OnSurveyScanData event provides survey data

---

#### **shipscanner**

Ship scanner.

**Methods:**
- `StartScan[entityID]` - Starts ship scan on entity

---

#### **cargoscanner**

Cargo scanner.

**Methods:**
- `StartScan[entityID]` - Starts cargo scan on entity

---

#### **directionalscannerresult**

Directional scanner result.

**Members:**
- `ID` - int64: Entity ID
- `Name` - string: Entity name
- `Group` - string: Entity group name
- `GroupID` - int: Entity group ID
- `Type` - string: Entity type name
- `TypeID` - int: Entity type ID
- `ToEntity` - [entity](#entity): Entity object (if on grid)

**See Also:**
- [evedirectionalscannerwindow](#evedirectionalscannerwindow) - D-scan window
- [entity](#entity) - Entity objects

---

#### **systemsignature**

System signature (wormhole, data/relic site, gas site, etc.).

**Members:**

*Identity:*
- `ID` - string: Signature ID (e.g., "ABC-123")
- `Name` - string: Signature name
- `Group` - string: Signature group
- `GroupID` - int: Signature group ID

*Scanning:*
- `SignalStrength` - double: Signal strength percentage
- `Deviation` - double: Deviation value
- `Difficulty` - string: Scan difficulty

*Navigation:*
- `IsWarpable` - bool: Whether signature can be warped to
- `X` - double: X coordinate
- `Y` - double: Y coordinate
- `Z` - double: Z coordinate
- `ToEntity` - [entity](#entity): Entity if signature is on grid
- `ToItem` - [item](#item): Item representation

**Methods:**
- `WarpTo[distance]` - Warps to signature at distance
- `AlignTo` - Aligns to signature
- `Approach[distance]` - Approaches signature

**See Also:**
- [systemscanner](#systemscanner) - GetSignatures method
- [entity](#entity) - Entity representation

---

#### **systemanomaly**

System anomaly (combat site, ore site, etc.).

**Members:**

*Identity:*
- `ID` - int64: Anomaly ID
- `Name` - string: Anomaly name
- `Group` - string: Anomaly group
- `GroupID` - int: Anomaly group ID
- `DungeonID` - int: Dungeon ID
- `DungeonName` - string: Dungeon name

*Faction:*
- `Faction` - string: Faction name
- `FactionID` - int: Faction ID

*Scanning:*
- `Difficulty` - string: Site difficulty
- `ScanStrength` - double: Scan strength
- `SignalStrength` - double: Signal strength

*Navigation:*
- `IsWarpable` - bool: Whether anomaly can be warped to
- `X` - double: X coordinate
- `Y` - double: Y coordinate
- `Z` - double: Z coordinate

**Methods:**
- `WarpTo[distance]` - Warps to anomaly at distance
- `AlignTo` - Aligns to anomaly
- `Approach[distance]` - Approaches anomaly

**See Also:**
- [systemscanner](#systemscanner) - GetAnomalies method

---

### Agent & Mission DataTypes

#### **eveagent**

EVE agent information and interaction.

**Members:**

*Identity:*
- `ID` - int64: Agent ID
- `Name` - string: Agent name
- `TypeID` - int: Agent type ID
- `AgentTypeID` - int: Agent type ID
- `AgentTypeName` - string: Agent type name
- `Index` - int: Agent index
- `Gender` - string: Agent gender

*Organization:*
- `Division` - string: Division name
- `DivisionID` - int: Division ID
- `CorporationID` - int: Corporation ID
- `FactionID` - int: Faction ID
- `Level` - int: Agent level

*Standing:*
- `StandingTo` - double: Your standing to agent
- `StandingToCorp` - double: Your standing to agent's corporation
- `StandingToFaction` - double: Your standing to agent's faction
- `EffectiveStanding` - double: Effective standing

*Location:*
- `Station` - string: Station name
- `StationID` - int64: Station ID
- `Solarsystem` - [solarsystem](#solarsystem): Solar system location

*Special:*
- `IsLocatorAgent` - bool: Whether this is a locator agent

**Methods:**
- `StartConversation` - Starts conversation with agent

**See Also:**
- [eve](#eve) - Agent method returns eveagent
- [character](#character) - GetAgents method
- [agentmission](#agentmission) - Agent missions

---

#### **agentmission**

Agent mission information.

**Members:**

*Identity:*
- `ID` - int: Mission ID
- `Name` - string: Mission name
- `Type` - string: Mission type
- `AgentID` - int64: Agent ID

*Status:*
- `State` - string: Mission state
- `Expires` - int64: Expiration time (seconds)
- `ExpirationTime` - [evetime](#evetime): Expiration time
- `ImportantMission` - bool: Whether mission is important

*Capabilities:*
- `RemoteOfferable` - bool: Whether mission can be offered remotely
- `RemoteCompletable` - bool: Whether mission can be completed remotely

**Methods:**
- `GetBookmarks[index:bookmark]` - Populates index with mission [bookmark](#bookmark) objects

**See Also:**
- [eveagent](#eveagent) - Agent information
- [character](#character) - GetAgentMissions method

---

### Skill DataTypes

#### **skill**

Character skill.

**Members:**
- `Name` - string: Skill name
- `TrainedLevel` - int: Trained skill level
- `EffectiveLevel` - int: Effective skill level (with bonuses)
- `Points` - int: Current skill points
- `TimeToTrain` - int64: Time to train next level (seconds)
- `TrainingTimeMultiplier` - double: Training time multiplier

**Methods:**
- `StartTraining` - Starts training this skill
- `AddToQueue[level]` - Adds skill to queue at specified level

**See Also:**
- [character](#character) - Character.Skill member
- [queuedskill](#queuedskill) - Queued skills

---

#### **queuedskill**

Skill in training queue.

**Members:**
- `Name` - string: Skill name
- `TrainingTo` - int: Target skill level
- `ToSkill` - [skill](#skill): Skill being trained
- `StartTime` - [evetime](#evetime): Training start time
- `EndTime` - [evetime](#evetime): Training end time
- `QueuePosition` - int: Position in queue (0-based)
- `StartSkillPoints` - int: Skill points at start
- `DestinationSkillPoints` - int: Skill points at completion

**See Also:**
- [skill](#skill) - Skill information
- [character](#character) - GetSkillQueue method

---

### Chat DataTypes

#### **evechat**

Chat system.

**Members:**
- `ChannelCount` - int: Number of chat channels

**Methods:**
- `GetChannels[index:chatchannel]` - Populates index with [chatchannel](#chatchannel) objects

**See Also:**
- [chatchannel](#chatchannel) - Chat channels

---

#### **chatchannel**

Chat channel.

**Members:**

*Identity:*
- `Name` - string: Channel name
- `ID` - int64: Channel ID
- `Category` - string: Channel category

*Status:*
- `MOTD` - string: Message of the day
- `LastActivityTime` - [evetime](#evetime): Last activity time

**Methods:**

*Members:*
- `GetMembers[index:pilot]` - Populates index with [pilot](#pilot) objects (recent speakers for some channel types)

*Messages:*
- `GetMessages[index:chatchannelmessage]` - Populates index with cached [chatchannelmessage](#chatchannelmessage) objects (up to 100)

**See Also:**
- [evechat](#evechat) - Chat system
- [chatchannelmessage](#chatchannelmessage) - Channel messages
- [pilot](#pilot) - Channel members

---

#### **chatchannelmessage**

Chat channel message.

**Members:**
- `Author` - [pilot](#pilot): Message author
- `Message` - string: Message text
- `Timestamp` - [evetime](#evetime): Message timestamp

**See Also:**
- [chatchannel](#chatchannel) - Chat channels
- [pilot](#pilot) - Message author

---

### Drone DataTypes

#### **activedrone**

Represents a drone currently active in space.

**Members:**

*Identity:*
- `ID` - int64: Drone ID
- `Owner` - string: Owner name
- `Controller` - string: Controller name
- `Type` - string: Drone type name
- `TypeID` - int: Drone type ID

*Status & Control:*
- `State` - string: Current drone state
- `Target` - [entity](#entity): Drone's current target
- `ToEntity` - [entity](#entity): Drone as entity object

**See Also:**
- [character](#character) - GetActiveDrones method returns activedrone collection
- [entity](#entity) - Drone entity representation

---

### Fleet DataTypes

#### **fleet**

Fleet information and management.

**Members:**

*Identity & Status:*
- `ID` - int64: Fleet ID
- `IsMember` - bool: Whether you are a fleet member
- `IsFleetCommander` - bool: Whether you are fleet commander
- `Size` - int: Number of members in fleet

*Invitations:*
- `Invited` - bool: Whether you have a pending fleet invitation
- `InvitationText` - string: Fleet invitation text

*Organization:*
- `Member[charID or name]` - [fleetmember](#fleetmember): Gets fleet member by ID or name
- `SquadName[squadID]` - string: Squad name by ID
- `SquadNameToID[name]` - int64: Squad ID by name
- `WingName[wingID]` - string: Wing name by ID
- `WingNameToID[name]` - int64: Wing ID by name

**Methods:**

*Invitation Management:*
- `Invite[charID or name]` - Invites character to fleet
- `AcceptInvite` - Accepts fleet invitation
- `RejectInvite` - Rejects fleet invitation
- `LeaveFleet` - Leaves the fleet

*Fleet Structure:*
- `CreateWing[name]` - Creates new wing
- `DeleteWing[wingID]` - Deletes wing
- `ChangeWingName[wingID,newName]` - Renames wing
- `CreateSquad[wingID,name]` - Creates new squad in wing
- `DeleteSquad[squadID]` - Deletes squad
- `ChangeSquadName[squadID,newName]` - Renames squad

*Fleet Information:*
- `GetMembers[index:fleetmember]` - Populates index with all [fleetmember](#fleetmember) objects
- `GetWings[index:int64]` - Populates index with wing IDs
- `GetSquads[index:int64]` - Populates index with squad IDs

*Fleet Commands:*
- `WarpFleetToMember[charID]` - Warps fleet to member
- `Regroup` - Issues fleet regroup command
- `SetIgnoreFleetWarp` - Sets your ship to ignore fleet warps
- `SetTakesFleetWarp` - Sets your ship to take fleet warps

*Fleet Broadcasts:*
- `Broadcast_Target[targetID]` - Broadcasts target
- `Broadcast_Location[itemID]` - Broadcasts location
- `Broadcast_AlignTo[celestialID]` - Broadcasts align to
- `Broadcast_WarpTo[celestialID]` - Broadcasts warp to
- `Broadcast_TravelTo[solarSystemID]` - Broadcasts travel to
- `Broadcast_JumpTo[solarSystemID]` - Broadcasts jump to
- `Broadcast_JumpBeacon[charID]` - Broadcasts jump to beacon
- `Broadcast_HoldPosition` - Broadcasts hold position
- `Broadcast_InPosition` - Broadcasts in position
- `Broadcast_NeedBackup` - Broadcasts need backup
- `Broadcast_EnemySpotted` - Broadcasts enemy spotted
- `Broadcast_HealArmor` - Broadcasts need armor repair
- `Broadcast_HealShield` - Broadcasts need shield repair
- `Broadcast_HealCapacitor` - Broadcasts need capacitor

**See Also:**
- [fleetmember](#fleetmember) - Fleet members
- [character](#character) - Character.Fleet member

---

#### **fleetmember**

Fleet member. Inherits from [**pilot**](#pilot) which inherits from [**being**](#being).

**Inheritance:**
- All members and methods from [**pilot**](#pilot) (and [**being**](#being)) are available

**Additional Members:**
- `Boosting` - bool: Whether providing fleet boosts
- `HasActiveBeacon` - bool: Whether has active fleet beacon
- `Job` - string: Fleet job name
- `JobID` - int: Fleet job ID
- `Role` - string: Fleet role name
- `RoleID` - int: Fleet role ID
- `SquadID` - int64: Squad ID
- `WingID` - int64: Wing ID
- `IsFleetCommander` - bool: Whether is fleet commander
- `IsWingCommander` - bool: Whether is wing commander
- `IsSquadCommander` - bool: Whether is squad commander
- `ToEntity` - [entity](#entity): Converts to entity type
- `ToPilot` - [pilot](#pilot): Converts to pilot type

**Additional Methods:**
- `AddToWatchList` - Adds to watch list
- `RemoveFromWatchList` - Removes from watch list
- `Kick` - Kicks from fleet
- `MakeLeader` - Makes fleet leader
- `Move[wingID,squadID]` - Moves to wing/squad
- `MoveToFleetCommander` - Moves to fleet commander level
- `MoveToWingCommander` - Moves to wing commander level
- `MoveToSquadCommander` - Moves to squad commander level
- `SetBooster` - Sets as booster
- `WarpFleetTo[entity,distance]` - Warps fleet to entity
- `WarpTo[entity,distance]` - Warps to entity

**See Also:**
- [pilot](#pilot) - Parent type
- [being](#being) - Base type
- [fleet](#fleet) - Fleet information
- [entity](#entity) - Entity conversion

---

### Miscellaneous DataTypes

#### **charselect**

Character selection interface. Used to select characters at login.

**Members:**
- `SelectedChar` - string: Currently selected character name
- `SelectedCharID` - int: Currently selected character ID
- `CharExists[name]` - bool: Whether character with name exists

**Methods:**
- `ClickCharacter[name]` - Clicks character by name in character selection screen

**See Also:**
- [character](#character) - Active character datatype

---

#### **fittingslot**

Ship fitting slot.

**Members:**

*Identity:*
- `Name` - string: Slot name (e.g., "HiSlot0")
- `ID` - int: Slot ID

*Status:*
- `IsEmpty` - bool: Whether slot is empty
- `IsOnline` - bool: Whether slot is online
- `ContainsCharge` - bool: Whether slot contains charge

*Module:*
- `Module` - [module](#module): Module in slot (if not empty)

**Methods:**

*Fitting:*
- `FitItem[itemID]` - Fits item to slot (checks if slot is empty)
- `Unfit` - Unfits module from slot
- `UnfitCharge` - Unfits charge from slot

*Status:*
- `PutOnline` - Puts module online
- `PutOffline` - Puts module offline

**See Also:**
- [evefittingwindow](#evefittingwindow) - Fitting window
- [module](#module) - Fitted modules

---

## Commands

### Core Commands

#### **UnloadISXEVE**

**Syntax:** `UnloadISXEVE`

**Description:** Safely unloads ISXEVE extension. Logs to console when unload completes.

**Examples:**
```lavishscript
UnloadISXEVE
```

---

#### **DebugSpew**

**Syntax:** `DebugSpew <message>`

**Description:** Writes debug messages to the ISXEVE debug log file.

**Parameters:**
- `message` - Debug message to write to log

**Examples:**
```lavishscript
DebugSpew "Starting mining script"
```

---

### Web Commands

#### **GetURL**

**Syntax:** `GetURL "http://address" [content_type]`

**Description:** Sends HTTP GET request to specified URL. Response is received via `isxGames_onHTTPResponse` event. This command uses multiple threads, so script will not freeze.

**Parameters:**
- `url` - URL to fetch (http or https)
- `content_type` - Optional content type (default: auto-detected)

**Examples:**
```lavishscript
GetURL "https://api.example.com/data.json" "application/json"
```

**See Also:**
- [PostURL](#posturl) - HTTP POST requests
- [isxGames_onHTTPResponse](#isxgames_onhttpresponse) - HTTP response event

---

#### **PostURL**

**Syntax:** `PostURL "http://address" "post_data" [content_type]`

**Description:** Sends HTTP POST request to specified URL. Response is received via `isxGames_onHTTPResponse` event. This command uses multiple threads, so script will not freeze.

**Parameters:**
- `url` - URL to post to (http or https)
- `post_data` - Data to post
- `content_type` - Optional content type (default: auto-detected)

**Examples:**
```lavishscript
PostURL "https://discord.com/api/webhooks/123456789/abcdefg" "{\"content\":\"This is a test\"}" "application/json"
```

**See Also:**
- [GetURL](#geturl) - HTTP GET requests
- [PostURLFiles](#posturlfiles) - HTTP POST with file uploads
- [isxGames_onHTTPResponse](#isxgames_onhttpresponse) - HTTP response event

---

#### **PostURLFiles**

**Syntax:** `PostURLFiles "http://address" "file_path1" ["file_path2" ...] [content_type]`

**Description:** Sends HTTP POST request with file uploads to specified URL. Response is received via `isxGames_onHTTPResponse` event. This command uses multiple threads, so script will not freeze.

**Parameters:**
- `url` - URL to post to (http or https)
- `file_path1` - Path to first file to upload
- `file_path2` - Optional additional file paths
- `content_type` - Optional content type (default: auto-detected)

**Examples:**
```lavishscript
PostURLFiles "https://api.example.com/upload" "C:\\logs\\combat.log"
PostURLFiles "https://api.example.com/upload" "C:\\logs\\mining.log" "C:\\logs\\trading.log"
```

**See Also:**
- [GetURL](#geturl) - HTTP GET requests
- [PostURL](#posturl) - HTTP POST requests
- [isxGames_onHTTPResponse](#isxgames_onhttpresponse) - HTTP response event

---

## Events

ISXEVE provides events that fire when specific game actions occur. Events are registered using LavishScript's event system.

### Event Registration

Events are registered using LavishScript's event system:

```lavishscript
Event[EventName]:AttachAtom[FunctionName]
```

Example:
```lavishscript
function OnChannelMessage(int channelID, int senderID, string message)
{
    echo "Channel: ${channelID}, Sender: ${senderID}, Message: ${message}"
}

Event[EVE_OnChannelMessage]:AttachAtom[OnChannelMessage]
```

### EVE Events

#### **EVE_OnChannelMessage**

**Description:** Fires when a chat message is received in any channel.

**Parameters:**
- `ChannelID` - int: Channel ID
- `SenderID` - int: Sender character ID
- `Message` - string: Message text

**Example:**
```lavishscript
function OnChannelMessage(int channelID, int senderID, string message)
{
    echo "Channel ${channelID}: ${message}"
}

Event[EVE_OnChannelMessage]:AttachAtom[OnChannelMessage]
```

**Notes:**
- Use the Chat TLO to get channel and sender details from IDs

---

#### **EVE_OnPilotJoinedChannel**

**Description:** Fires when a pilot joins a channel. **NOTE:** This event has been disabled since March 2018 due to EVE client chat system changes.

**Status:** DISABLED (March 21, 2018)

---

#### **EVE_OnPilotLeftChannel**

**Description:** Fires when a pilot leaves a channel. **NOTE:** This event has been disabled since March 2018 due to EVE client chat system changes.

**Status:** DISABLED (March 21, 2018)

---

#### **EVE_OnSurveyScanData**

**Description:** Fires when survey scanner results are received.

**Parameters:**
- `ResultCount` - int: Number of asteroids/objects returned

**Example:**
```lavishscript
function OnSurveyScan(int resultCount)
{
    echo "Survey scan returned ${resultCount} asteroids"
    ; Scan results available in entity list
}

Event[EVE_OnSurveyScanData]:AttachAtom[OnSurveyScan]
```

**Notes:**
- Added May 10, 2020
- Survey data is available in the entity list after this event fires
- Use entity queries to access the survey results

---

### System Events

#### **isxGames_onHTTPResponse**

**Description:** Fires when HTTP response is received from GetURL or PostURL command.

**Parameters:**
- `URL` - string: Requested URL
- `ResponseCode` - int: HTTP response code
- `ResponseText` - string: Response content

**Example:**
```lavishscript
function OnHTTPResponse(string url, int code, string response)
{
    echo "Response from ${url}: Code ${code}"
    if ${code.Equal[200]}
    {
        echo "Success: ${response}"
    }
}

Event[isxGames_onHTTPResponse]:AttachAtom[OnHTTPResponse]
```

**See Also:**
- [GetURL](#geturl) - HTTP GET command
- [PostURL](#posturl) - HTTP POST command

**Notes:**
- Both GetURL and PostURL use this same event
- Commands are non-blocking (script continues while request is processed)

---

## Usage Examples

### Basic Information

#### Getting Character Information
```lavishscript
echo "Character: ${Me.Name}"
echo "Character ID: ${Me.ID}"
echo "Corporation ID: ${Me.CorpID}"
echo "Alliance ID: ${Me.AllianceID}"
echo "Current Ship: ${MyShip.ToItem.Name}"
echo "In Station: ${Me.InStation}"
echo "In Space: ${Me.InSpace}"
```

#### Getting Location Information
```lavishscript
echo "Solar System ID: ${Me.SolarSystemID}"
if ${Me.InStation}
{
    echo "Station ID: ${Me.StationID}"
    echo "Station Name: ${Me.Station.Name}"
}
```

---

### Autopilot and Navigation

#### Setting Destination
```lavishscript
; Set destination by solar system ID
EVE:SetDestination[30000142]

; Add waypoint
EVE:AddWaypoint[30000143]

; Clear all waypoints
EVE:ClearAllWaypoints

; Optimize route
EVE:OptimizeAutopilotRoute
```

#### Using Bookmarks for Navigation
```lavishscript
; Warp to bookmark
EVE.Bookmark[Safe Spot]:WarpTo

; Warp to bookmark within 50km
EVE.Bookmark[Safe Spot]:WarpTo[50000]

; Set bookmark as destination
EVE.Bookmark[Mission System]:SetDestination
```

#### Jumps Calculation
```lavishscript
; Get jumps to solar system
echo "Jumps to Jita: ${EVE.JumpsTo[30000142]}"

; Get jumps between systems
echo "Jumps between systems: ${EVE.JumpsBetween[30000142,30000143]}"

; Get jumps from bookmark
echo "Jumps to bookmark: ${EVE.Bookmark[Home]:JumpsTo[30000142]}"
```

---

### Bookmarks

#### Creating Bookmarks
```lavishscript
; Create bookmark in current location (personal folder)
EVE:CreateBookmark[My Safe Spot]

; Create bookmark with notes
EVE:CreateBookmark[My Safe Spot,"This is a safe location"]

; Create corp bookmark
EVE:CreateBookmark[Corp Safe,"Safe for all members","corp"]

; Create bookmark in specific folder
EVE:CreateBookmark[Mission Spot,"Mission location","My Missions"]

; Create bookmark with expiry (3 hours)
EVE:CreateBookmark[Temp Spot,"Temporary location","Personal",1]

; Create bookmark at entity location
Target:CreateBookmark[Target Location,"Where I found the target"]
```

---

### Items and Inventory

#### Accessing Inventory
```lavishscript
; Open inventory window
EVEWindow[Inventory]:MakeActive

; Access inventory child windows
echo "Cargo capacity: ${EVEWindow[Inventory].Child[ShipCargo].Capacity}"
echo "Cargo used: ${EVEWindow[Inventory].Child[ShipCargo].UsedCapacity}"

; Get items from cargo
variable index:item CargoItems
EVEWindow[Inventory].Child[ShipCargo]:GetItems[CargoItems]
echo "Items in cargo: ${CargoItems.Used}"
```

#### Moving Items
```lavishscript
; Move single item to ship cargo
MyItem:MoveTo[CargoHold]

; Move specific quantity
MyItem:MoveTo[CargoHold,100]

; Move items from index to hangar
variable index:item ItemsToMove
; ... populate ItemsToMove ...
EVE:MoveItemsTo[MyStationHangar,ItemsToMove]

; Stack all items in cargo
EVEWindow[Inventory].Child[ShipCargo]:StackAll
```

#### Item Compression
```lavishscript
; Open compress window for ore
MyOre:Compress
wait 10 ${EVEWindow[Compress](exists)}
EVEWindow[Compress]:ClickButtonOK
```

---

### Ship and Modules

#### Accessing Modules
```lavishscript
; Get module by slot index
variable module MyModule
MyModule:Set[${MyShip.Module[1]}]
echo "Module: ${MyModule.ToItem.Name}"

; Get module by name
MyModule:Set[${MyShip.Module[Medium Shield Booster II]}]

; Get all modules
variable index:module AllModules
MyShip:GetModules[AllModules]
echo "Total modules: ${AllModules.Used}"
```

#### Activating Modules
```lavishscript
; Activate specific module
MyShip.Module[1]:Activate

; Deactivate module
MyShip.Module[1]:Deactivate

; Toggle module
MyShip.Module[1]:Toggle

; Activate module on target
MyShip.Module[1]:Click
```

#### Module Status
```lavishscript
; Check module status
if ${MyShip.Module[1].IsOnline}
{
    echo "Module is online"
}

if ${MyShip.Module[1].IsActive}
{
    echo "Module is active"
}

if ${MyShip.Module[1].IsReloading}
{
    echo "Module is reloading"
}
```

#### Reloading and Changing Ammo
```lavishscript
; Change ammo
MyShip.Module[1]:ChangeAmmo[24493]

; Change ammo with quantity
MyShip.Module[1]:ChangeAmmo[24493,1000]

; Enable auto-reload
MyShip.Module[1]:SetAutoReloadOn

; Disable auto-reload
MyShip.Module[1]:SetAutoReloadOff
```

#### Drones
```lavishscript
; Launch all drones
MyShip:LaunchAllDrones

; Recall all drones
MyShip:RecallAllDrones

; Get drones in bay
variable index:item DronesInBay
MyShip:GetDrones[DronesInBay]
echo "Drones in bay: ${DronesInBay.Used}"

; Get active drones
variable index:entity ActiveDrones
MyShip:GetActiveDrones[ActiveDrones]
echo "Active drones: ${ActiveDrones.Used}"
```

---

### Targets and Combat

#### Targeting Entities
```lavishscript
; Lock target
Target:LockTarget

; Unlock target
Target:UnlockTarget

; Make active target
Target:MakeActiveTarget

; Set as overview selected item
Target:SetAsSelectedItem
```

#### Querying Entities
```lavishscript
; Get all entities
variable index:entity AllEntities
EVE:GetEntities[AllEntities]
echo "Total entities: ${AllEntities.Used}"

; Query specific entities (NPCs)
variable index:entity NPCs
EVE:QueryEntities[NPCs,"CategoryID = 11"]
echo "NPCs found: ${NPCs.Used}"

; Query entities in range
EVE:QueryEntities[NPCs,"Distance < 50000 && CategoryID = 11"]
```

#### Combat Status
```lavishscript
; Check if being targeted
if ${Target.IsTargetingMe}
{
    echo "Entity is targeting me"
}

; Check if being jammed
if ${Target.IsJammingMe}
{
    echo "Being jammed!"
}

; Check if being scrambled
if ${Target.IsWarpScramblingMe}
{
    echo "Being scrambled!"
}
```

---

### Market Operations

#### Creating Buy Orders
```lavishscript
; Open market buy order window
EVE:CreateMarketBuyOrder[34]

; Wait for window
wait 20 ${EVEWindow[MarketActionWindow](exists)}

; Set price
EVEWindow[MarketActionWindow].BidPrice:SetValue[1000000]

; Set quantity
EVEWindow[MarketActionWindow].Quantity:SetValue[100]

; Place order
EVEWindow[MarketActionWindow]:Buy
```

#### Selling Items
```lavishscript
; Add item to sell order
MyItem:AddToSellOrder

; Wait for sell window
wait 20 ${EVEWindow[SellItems](exists)}

; Set duration (90 days)
EVEWindow[SellItems]:SetDuration[90]

; Set price for first item
EVEWindow[SellItems].Item[1]:SetAskPrice[5000000]

; Sell items
EVEWindow[SellItems]:Sell
```

---

### Scanning

#### Directional Scanner - Basic
```lavishscript
; Access d-scan window
echo "D-scan range: ${EVEWindow[directionalScannerWindow].Range} AU"
echo "D-scan angle: ${EVEWindow[directionalScannerWindow].Angle}"

; Wait for scan to complete
wait 50 !${EVEWindow[directionalScannerWindow].IsScanning}

; Get scan results
variable index:directionalscannerresult DScanResults
EVEWindow[directionalScannerWindow]:GetScanResults[DScanResults]

; Process results
variable iterator ResultIterator
DScanResults:GetIterator[ResultIterator]
if ${ResultIterator:First(exists)}
{
    do
    {
        echo "ID: ${ResultIterator.Value.ID}"
        echo "Name: ${ResultIterator.Value.Name}"
        echo "Type: ${ResultIterator.Value.Type}"
        echo "Group: ${ResultIterator.Value.Group}"
    }
    while ${ResultIterator:Next(exists)}
}
```

#### Directional Scanner - Complete Example with Error Handling
```lavishscript
function PerformDScan()
{
    variable index:directionalscannerresult ScanResults
    variable iterator ScanResultsIterator
    variable int Counter = 1

    ; Open directional scanner window if not already open
    if (!${EVEWindow[directionalScannerWindow](exists)})
    {
        EVE:Execute[OpenDirectionalScanner]
        do
        {
            waitframe
        }
        while !${EVEWindow[directionalScannerWindow](exists)} && ${Counter:Inc} < 1000
        wait 15
        if (!${EVEWindow[directionalScannerWindow](exists)})
        {
            echo "[DScanWindow] ERROR - Unable to Open and/or Access the DirectionalScannerWindow."
            return
        }
        Counter:Set[1]
    }

    echo "[DScanWindow] Range set to ${EVEWindow[directionalScannerWindow].Range}"
    echo "[DScanWindow] Angle set to ${EVEWindow[directionalScannerWindow].Angle}"

    ; Perform scan
    echo "[DScanWindow] Scanning..."
    EVEWindow[directionalScannerWindow].Button[1]:Press
    do
    {
        waitframe
    }
    while ${EVEWindow[directionalScannerWindow].IsScanning} && ${Counter:Inc} < 1000
    if (${EVEWindow[directionalScannerWindow].IsScanning})
    {
        echo "[DScanWindow] ERROR - IsScanning TRUE too long"
        return
    }
    else
        Counter:Set[1]

    ; Get and process scan results
    EVEWindow[directionalScannerWindow]:GetScanResults[ScanResults]
    ScanResults:GetIterator[ScanResultsIterator]
    echo "[DScanWindow] Scan Finished - ${ScanResults.Used} entries:"

    if ${ScanResultsIterator:First(exists)}
    do
    {
        echo "[DScanWindow] ${Counter}. ${ScanResultsIterator.Value.Name} (ID: ${ScanResultsIterator.Value.ID})"
        echo "[DScanWindow] ${Counter}. -- Type: ${ScanResultsIterator.Value.Type} (${ScanResultsIterator.Value.TypeID})"
        echo "[DScanWindow] ${Counter}. -- Group: ${ScanResultsIterator.Value.Group} (${ScanResultsIterator.Value.GroupID})"
        if (${ScanResultsIterator.Value.ToEntity.ID} > 0)
        {
            echo "[DScanWindow] ${Counter}. -- Distance: ${ScanResultsIterator.Value.ToEntity.Distance}"
        }
        else
        {
            echo "[DScanWindow] ${Counter}. -- Distance: Unknown"
        }
        echo "==="
        Counter:Inc
    }
    while ${ScanResultsIterator:Next(exists)}
}
```

---

### Universe and Planetary Information

#### Accessing Planet Information
```lavishscript
function GetPlanetInfo()
{
    variable index:int Planets
    variable iterator PlanetsIterator
    variable iterator PlanetsByTypeIterator
    variable collection:int PlanetsByType
    variable int Counter = 1

    ; Using system Gulfonodi as example (Gulfonodi's ID is 30002384)
    Universe[30002384]:GetPlanetIDs[Planets]
    Planets:GetIterator[PlanetsIterator]
    Universe[30002384]:GetNumPlanetsByType[PlanetsByType]
    PlanetsByType:GetIterator[PlanetsByTypeIterator]

    ; Display planet details
    if ${PlanetsIterator:First(exists)}
    {
        echo "[PI] The Gulfonodi System has ${Planets.Used} planets:"
        do
        {
            echo "[PI] - ${Counter}. ${PlanetsIterator.Value}: ${Universe[${PlanetsIterator.Value}].Name}"
            Counter:Inc
        }
        while ${PlanetsIterator:Next(exists)}
    }
    Counter:Set[1]

    ; Display planet type breakdown
    if ${PlanetsByTypeIterator:First(exists)}
    {
        echo "[PI] The breakdown of planet types in the Gulfondi System are as follows:"
        do
        {
            echo "[PI] - ${Counter}. ${PlanetsByTypeIterator.Key}: ${PlanetsByTypeIterator.Value}"
            Counter:Inc
        }
        while ${PlanetsByTypeIterator:Next(exists)}

        ; Note: To get quantities by type, use ${PlanetsByType.Element["Barren"]}, for example.
    }
    Counter:Set[1]

    return
}
```

#### Accessing Solar System Information
```lavishscript
; Access solar system by ID
echo "System: ${Universe[30000142].Name}"
echo "Security: ${Universe[30000142].Security}"

; Access solar system by name
echo "System ID: ${Universe[Jita].ID}"

; Get constellation and region
echo "Constellation: ${Universe[30000142].Constellation.Name}"
echo "Region: ${Universe[30000142].Region.Name}"

; Get celestial objects
variable index:int Celestials
Universe[30000142]:GetCelestials[Celestials]
echo "Celestial objects: ${Celestials.Used}"
```

#### Using Collections and Iterators
```lavishscript
; Collections store key-value pairs
variable collection:int MyCollection
variable iterator MyIterator

; Populate collection (example with planet types)
Universe[30002384]:GetNumPlanetsByType[MyCollection]
MyCollection:GetIterator[MyIterator]

; Iterate through collection
if ${MyIterator:First(exists)}
{
    do
    {
        echo "Key: ${MyIterator.Key}, Value: ${MyIterator.Value}"
        ; Access specific element: ${MyCollection.Element[${MyIterator.Key}]}
    }
    while ${MyIterator:Next(exists)}
}
```

---

### Station and Structure Operations

#### Docking
```lavishscript
; Dock with station entity
Target:Dock

; Wait for docking
wait 50 ${Me.InStation}
```

#### Undocking
```lavishscript
; Undock from station
Me.Station:Undock

; Wait for undocking
wait 50 ${Me.InSpace}
```

#### Accessing Station Services
```lavishscript
; Open fitting window
Me.Station:OpenFitting

; Get repairable items
variable index:item RepairItems
Me.Station:GetRepairableItems[RepairItems]
echo "Repairable items: ${RepairItems.Used}"
```

#### Repairing Ship
```lavishscript
; Get repair quote
MyShip.ToItem:GetRepairQuote

; Wait for repair window
wait 20 ${EVEWindow[RepairShop](exists)}

; Check repair cost
echo "Total cost: ${EVEWindow[RepairShop].TotalCost}"
echo "Average damage: ${EVEWindow[RepairShop].AverageDamage}"

; Repair all
EVEWindow[RepairShop]:RepairAll
```

#### Citadel/Structure Operations
```lavishscript
; Check if structure is dockable
if ${Target.IsDockable}
{
    Target:Dock
}

; Check if can board structure
if ${Me.Station.CanBoard}
{
    Me.Station:Board
}

; Check if corp has office
if ${Me.Station.MyCorpHasOffice}
{
    echo "Corp has office here"
}
```

---

### UI Interaction

#### Window Management
```lavishscript
; Check if window exists
if ${EVEWindow[Inventory](exists)}
{
    echo "Inventory window is open"
}

; Open window by name
EVEWindow[Inventory]:MakeActive

; Close window
EVEWindow[Inventory]:Close

; Get window position and size
echo "X: ${EVEWindow[Inventory].X}"
echo "Y: ${EVEWindow[Inventory].Y}"
echo "Width: ${EVEWindow[Inventory].Width}"
echo "Height: ${EVEWindow[Inventory].Height}"
```

#### Button Interaction
```lavishscript
; Click OK button
EVEWindow[Modal]:ClickButtonOK

; Click Yes button
EVEWindow[Modal]:ClickButtonYes

; Access specific button
EVEWindow[Modal].Button[1]:Press

; Access button by text
EVEWindow[Modal].Button[Accept]:Press
```

#### Agent Dialog
```lavishscript
; Access agent dialog
variable eveagentdialogwindow AgentDialog
AgentDialog:Set[${EVEWindow[ByCaption,Agent Conversation]}]

; Get briefing
echo "${AgentDialog.BriefingHTML}"

; Get objectives
echo "${AgentDialog.ObjectivesHTML}"

; Press dialog button
AgentDialog.Button[Accept]:Press
```

---

### Fleet Operations

#### Fleet Information
```lavishscript
; Check if in fleet
if ${Me.Fleet(exists)}
{
    echo "In fleet with ${Me.Fleet.MemberCount} members"
    echo "Fleet MotD: ${Me.Fleet.MotD}"
}

; Check fleet roles
if ${Me.Fleet.IsBoss}
{
    echo "I am fleet boss"
}

if ${Me.Fleet.IsCommander}
{
    echo "I am wing commander"
}
```

#### Fleet Commands
```lavishscript
; Warp fleet to target
Me.Fleet:WarpFleet[${Target.ID}]

; Issue regroup command
Me.Fleet:Regroup

; Leave fleet
Me.Fleet:LeaveFleet
```

#### Fleet Settings
```lavishscript
; Set to ignore fleet warps
Me.Fleet:SetIgnoreFleetWarp

; Set to take fleet warps
Me.Fleet:SetTakesFleetWarp
```

#### Fleet Members
```lavishscript
; Get all fleet members
variable index:fleetmember FleetMembers
Me.Fleet:GetMembers[FleetMembers]

; Iterate through members
variable iterator MemberIterator
FleetMembers:GetIterator[MemberIterator]
if ${MemberIterator:First(exists)}
{
    do
    {
        echo "Member: ${MemberIterator.Value.Name}"
        echo "System: ${MemberIterator.Value.SolarSystemID}"
        echo "Ship: ${MemberIterator.Value.ShipTypeID}"
        echo "Role: ${MemberIterator.Value.Role}"
    }
    while ${MemberIterator:Next(exists)}
}
```

---

### Chat Operations

#### Accessing Channels
```lavishscript
; Get channel count
echo "Channels: ${Chat.ChannelCount}"

; Access specific channel
variable chatchannel LocalChannel
LocalChannel:Set[${Chat[Local]}]

; Access channel by ID
LocalChannel:Set[${Chat[123456]}]

; Get all channels
variable index:chatchannel AllChannels
Chat:GetChannels[AllChannels]
```

#### Reading Messages
```lavishscript
; Get messages from channel
variable index:chatchannelmessage Messages
Chat[Local]:GetMessages[Messages]

; Iterate through messages
variable iterator MsgIterator
Messages:GetIterator[MsgIterator]
if ${MsgIterator:First(exists)}
{
    do
    {
        echo "[${MsgIterator.Value.Timestamp.DateAndTime}] ${MsgIterator.Value.Author.Name}: ${MsgIterator.Value.Message}"
    }
    while ${MsgIterator:Next(exists)}
}
```

#### Channel Members
```lavishscript
; Get channel members
variable index:pilot ChannelMembers
Chat[Corp]:GetMembers[ChannelMembers]

; Iterate through members
variable iterator MemberIterator
ChannelMembers:GetIterator[MemberIterator]
if ${MemberIterator:First(exists)}
{
    do
    {
        echo "Member: ${MemberIterator.Value.Name}"
    }
    while ${MemberIterator:Next(exists)}
}
```

---

## Notes

### Case Sensitivity

LavishScript and ISXEVE are **case-insensitive** for:
- Member names
- Method names
- Datatype names
- TLO names

However, **case-sensitive** for:
- String comparisons in queries
- Character names, item names, etc.

Best practice is to maintain consistent casing for readability.

### NULL Checks

Always check if an object exists before accessing its members:

```lavishscript
; Bad - will crash if Target doesn't exist
echo ${Target.Name}

; Good - checks existence first
if ${Target(exists)}
{
    echo ${Target.Name}
}
```

For datatypes that can return NULL or invalid data:
- Check with `(exists)` before using
- Some members return -1, -2, or -10 to indicate errors (documented in member descriptions)
- Always validate return values before using them in calculations

### Parameter Notation

In this documentation:
- `[parameter]` - Required parameter
- `[optional_parameter]` - Optional parameter (may be omitted)
- `#` - Numeric value
- `"text"` - String value
- `id` - ID number (usually int64)
- `name` - Name string

Method syntax examples:
- `Method` - No parameters
- `Method[param]` - One required parameter
- `Method[param1,param2]` - Multiple required parameters
- `Method[param1,optional]` - Required and optional parameters

### EVE Client Updates

ISXEVE is updated regularly to maintain compatibility with EVE Online client patches. After major EVE expansions or patches:
1. Wait for ISXEVE update announcement
2. Update ISXEVE to latest version
3. Test your scripts for any breaking changes
4. Review changelog for new features and changes

### Performance Considerations

**Entity Queries:**
- Use specific queries rather than getting all entities
- Cache entity lists when possible rather than querying every frame
- Use Distance2 instead of Distance when you only need comparison (faster)

**Inventory Access:**
- Unified inventory methods are more reliable than old deprecated methods
- Wait for inventory windows to be ready before accessing (check IsReady or wait for capacities != -2)
- Batch item moves when possible using index:item collections

**Universe TLO:**
- First use each session with string parameter will lag (caches hundreds of thousands of names)
- Use ID numbers instead of names when possible for better performance
- Cache frequently accessed universe objects

### Authentication

ISXEVE requires valid isxGames credentials. Credentials can be set in two ways:

1. **ISXEVE.xml configuration file** (default method)
2. **Global variables** (for shared computers or temporary sessions):
   ```lavishscript
   declarevariable isxGames_Login string globalkeep YourUsername
   declarevariable isxGames_Password string globalkeep YourPassword
   ext isxeve
   ```

Variables persist through ISXEVE reloads but not between sessions. Change password at https://auth.isxgames.com/login

**Security best practices:**
- Use unique isxGames password (different from other sites)
- Don't share scripts containing credentials
- Store credentials in separate, private script files

### Deprecated Features

This section documents features that have been removed from ISXEVE over time. These are no longer available and scripts using them must be updated to use the replacements.

**Deprecation Policy:**

ISXEVE maintains backward compatibility for deprecated members and methods for a transition period. When a member or method is deprecated, it will log a warning to the console indicating the deprecated feature and the recommended replacement. Script writers should update their scripts to use the new methods to ensure future compatibility.

**Major deprecation changes:**
- **2015-2018**: Gradual migration to new UI element and window datatypes
- **July 2020**: Many cargo and inventory methods deprecated in favor of unified inventory window child methods
- **2020**: Migration from 32-bit to 64-bit EVE client

---

#### DataType Members/Methods

##### Item DataType (January 2025)

**Removed Method:**

- `FitToActiveShip` - Use `EVEWindow[fitting].Slot[SlotName]:FitItem[itemID]` instead

**Removed:** January 7, 2025 [20240603.0012]

---

##### Agent DataType (June 2023)

**Removed Method:**
- `GetDetails` - Method removed

**Removed:** June 13, 2023 [20230613.0001]

---

##### Module DataType (May 2020)

**Removed Member:**
- `Accuracy` - Had been disabled for years

**Removed:** May 10, 2020 [20200430.0003]

---

##### Entity DataType (July 2020)

**Deprecated Methods:**
- `StackAllCargo`
- `OpenCargo`
- `CloseCargo`
- `GetCargo`
- `GetCorpHangarsCargo`
- `OpenCorpHangars`
- `OpenMaintenanceBay`
- `OpenStorage`
- `StorageWindow`

**Replacement:** Use `EVEWindow[Inventory].Child[name]:GetItems[index:item]` and related methods

**Removed:** July 12, 2020 [20200625.0002]

---

##### Item DataType (July 2020)

**Deprecated Methods:**
- `Close`
- `GetCargo`

**Replacement:** Use inventory window child methods

**Removed:** July 12, 2020 [20200625.0002]

---

##### Ship DataType (July 2020)

**Deprecated Methods:**
- `Cargo`
- `GetCargo`
- `GetCorpHangarsCargo`
- `GetFleetHangarCargo`
- `GetOreHoldCargo`
- `GetFuelHoldCargo`
- `GetGasHoldCargo`
- `GetMineralHoldCargo`
- `GetSalvageHoldCargo`
- `GetIndustrialShipHoldCargo`
- `GetAmmoHoldCargo`
- `GetMaintenanceHoldCargo`

**Replacement:** Use inventory window child methods

**Removed:** July 12, 2020 [20200625.0002]

---

##### EVEWindow DataType (August 2018)

**Removed Members:**
- `ItemID` - Moved to eveinvwindow
- `Capacity` - Moved to eveinvwindow
- `UsedCapacity` - Moved to eveinvwindow

**Removed:** August 23, 2018 [20180823.0003]

---

##### EVEWindow DataType (July 2020)

**Removed Method:**
- `MoveBookmarkHere[BookmarkID]`

**Removed:** July 12, 2020 [20200625.0002]

---

##### EVE DataType (April 2018)

**Removed Method:**
- `PlaceBuyOrder` - Use `CreateMarketBuyOrder[typeID]` instead

**Removed:** April 21, 2018 [20180417.0003]

---

##### EVE DataType (January 2015)

**Removed Method:**
- `PlaceSellOrder` - Use `Item:AddToSellOrder` instead

**Removed:** January 22, 2015 [ISXEVE-20150113.0002]

---

##### ISXEVE DataType (July 2020)

**Removed Method:**
- `Debug_SetFullMiniDumps`

**Removed:** July 12, 2020 [20200625.0002]

---

##### Agent DataType (February 2021)

**Removed Members:**
- `Dialog`

**Removed Methods:**
- `GetDialogResponses`

**Removed:** February 15, 2021 [20210209.0004]

---

##### Skill DataType (November 2015)

**Removed Members:**
- `ID`
- `Group`
- `GroupID`
- `IsTraining`

**Removed Methods:**
- `AbortTraining`

**Removed:** November 12, 2015 [ISXEVE-20151110.0002]

---

##### ChatChannel DataType (March 2018)

**Removed Members:**
- `NewMessageReceived`
- `CanSpeak`

**Removed Methods:**
- `MarkAsRead`
- `Echo`
- `Send`

**Removed:** April 1, 2018 [20180327.0004] (temporarily removed, may return)

---

#### Removed DataTypes

##### **dialogstring**

**Description:** Dialog string datatype removed when agent dialog system was updated.

**Removed:** February 15, 2021 [20210209.0004]

---

##### **login**

**Description:** Login datatype removed as in-game login hasn't been supported by EVE in years.

**Removed:** June 4, 2020 [20200604.0001]

---

#### Removed TLOs

##### **Agent**

**Description:** Agent TLO removed. Use `EVE.Agent[id]` or `EVE.Agent[name]` instead.

**Removed:** May 19, 2020 [20200519.0001]

---

##### **Login**

**Description:** Login TLO removed as in-game login hasn't been supported by EVE in years.

**Removed:** June 4, 2020 [20200604.0001]

---

*This reference documentation is maintained for ISXEVE. For the latest updates and changes, refer to ISXEVEChanges.txt.*
