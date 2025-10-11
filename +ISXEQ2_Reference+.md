# ISXEQ2 Reference

## Table of Contents

1. [Introduction](#introduction)
2. [Top-Level Objects](#top-level-objects) 
3. [DataType Categories](#datatype-categories)
   - [Core Game DataTypes](#core-game-datatypes)
   - [Actor DataTypes](#actor-datatypes)
   - [Item DataTypes](#item-datatypes)
   - [Ability DataTypes](#ability-datatypes)
   - [Effect DataTypes](#effect-datatypes)
   - [Quest DataTypes](#quest-datatypes)
   - [Commerce DataTypes](#commerce-datatypes)
   - [Mail DataTypes](#mail-datatypes)
   - [Crafting DataTypes](#crafting-datatypes)
   - [Window DataTypes](#window-datatypes)
   - [Widget/UI DataTypes](#widgetui-datatypes)
4. [Commands](#commands)
   - [Core Commands](#core-commands)
   - [Utility Commands](#utility-commands)
   - [Web Commands](#web-commands)
5. [Events](#events)
   - [Event Registration](#event-registration)
   - [Actor Events](#actor-events)
   - [Casting Events](#casting-events)
   - [Character Events](#character-events)
   - [Group and Raid Events](#group-and-raid-events)
   - [Quest Events](#quest-events)
   - [Inventory and Item Events](#inventory-and-item-events)
   - [Crafting Events](#crafting-events)
   - [Window and UI Events](#window-and-ui-events)
   - [Communication Events](#communication-events)
   - [Zone Events](#zone-events)
   - [Miscellaneous Events](#miscellaneous-events)
   - [ISXEQ2 System Events](#isxeq2-system-events)
6. [Deprecated Features](#deprecated-features)
   - [Removed Character Members](#removed-character-members-november-2018)
   - [Removed Pet Members](#removed-pet-members-march-2012)
   - [Removed Actor Members](#removed-actor-members-july-2020)
   - [Removed TLO](#removed-tlo-december-2016)
7. [Usage Examples](#usage-examples)
   - [Basic Information](#basic-information)
   - [Inventory and Items](#inventory-and-items)
   - [Abilities and Casting](#abilities-and-casting)
   - [Effects and Maintained Spells](#effects-and-maintained-spells)
   - [Quests](#quests)
   - [Commerce](#commerce)
   - [UI Interaction](#ui-interaction)
   - [Advanced Queries](#advanced-queries)
   - [Crafting](#crafting)
   - [Movement and Position](#movement-and-position)
   - [Extension Utilities](#extension-utilities)
   - [Adornment Attachment](#adornment-attachment)
   - [Fast Travel](#fast-travel)
   - [House Item Movement](#house-item-movement)
   - [Asynchronous Data Loading](#asynchronous-data-loading)
   - [Advanced Query Patterns](#advanced-query-patterns)
   - [Event-Driven Scripting](#event-driven-scripting)
   - [UI Widget Inspection](#ui-widget-inspection)
   - [Window Interaction Examples](#window-interaction-examples)
   - [Quest Journal Iteration](#quest-journal-iteration)
   - [Specialized Item Methods](#specialized-item-methods)
   - [Account and Character Data](#account-and-character-data)
   - [Group Leader Detection](#group-leader-detection)
   - [Commands](#commands-1)
8. [Notes](#notes)
   - [Deprecated Members](#deprecated-members)
   - [Case Sensitivity](#case-sensitivity)
   - [NULL Checks](#null-checks)
   - [Parameter Notation](#parameter-notation)

---

## Introduction

This document provides comprehensive reference documentation for all datatypes and top-level objects available in the ISXEQ2 extension for EverQuest 2. ISXEQ2 extends LavishScript to provide access to game data and functionality through a structured type system.

### DataType Inheritance

Many datatypes inherit from other datatypes, gaining all the members and methods of their parent type. Common inheritance patterns:

- **char** inherits from **actor**
- **groupmember** inherits from **actor**
- **eq2widget** inherits from **eq2baseobject**
- All window types inherit from **eq2window**

### Accessing DataTypes

DataTypes are accessed through Top-Level Objects (TLOs) or through members of other datatypes. For example:

```
${Me}                           // TLO returning 'char' datatype
${Me.Target}                    // Member returning 'actor' datatype
${Me.Inventory[5]}              // Member returning 'item' datatype
${EQ2UIPage[MapWindow]}         // TLO with parameter returning 'eq2uipage' datatype
```

---

## Top-Level Objects

Top-Level Objects (TLOs) are the entry points for accessing game data. They can be accessed directly in LavishScript.

### Extension TLOs

| TLO | DataType | Description |
|-----|----------|-------------|
| **ISXEQ2** | [isxeq2](#isxeq2) | ISXEQ2 extension information and utilities |



### Game TLOs

| TLO | DataType | Description |
|-----|----------|-------------|
| **EQ2** | [eq2](#eq2) | Main game information and utilities |
| **Me** | [char](#char) | Player character |
| **Target** | [actor](#actor) | Current target |
| **Zone** | [zone](#zone) | Current zone information |
| **Actor[id/name]** | [actor](#actor) | Access specific actor by ID or name |
| **CustomActor[index]** | [actor](#actor) | Access custom actor array |
| **Radar** | [radar](#radar) | Radar functionality |
| **EQ2Loc[x,y,z]** | [eq2location](#eq2location) | Create location object |
| **Crafting** | [crafting](#crafting) | Crafting session information |
| **Achievement[id/name]** | [achievement](#achievement) | Access achievement data |
| **StripTags[text]** | string | Removes EQ2 formatting tags from text |
| **EQ2DataSourceContainer[name]** | [eq2datasourcecontainer](#eq2datasourcecontainer) | Access UI data source |



### Window TLOs

| TLO | DataType | Description |
|-----|----------|-------------|
| **EQ2UIPage[name]** | [eq2uipage](#eq2uipage) | Access UI page by name |
| **ContainerWindow** | [containerwindow](#containerwindow) | Container window |
| **RewardWindow** | [rewardwindow](#rewardwindow) | Reward selection window |
| **ChoiceWindow** | [choicewindow](#choicewindow) | Choice dialog window |
| **LootWindow** | [lootwindow](#lootwindow) | Loot window |
| **ReplyDialog** | [replydialog](#replydialog) | NPC conversation window |
| **ExamineItemWindow** | [examineitemwindow](#examineitemwindow) | Item examination window |
| **ReforgeWindow** | [reforgewindow](#reforgewindow) | Item reforge window |
| **QuestJournalWindow** | [journalquestwindow](#journalquestwindow) | Quest journal window |
| **MailWindow** | [mailwindow](#mailwindow) | Mail inbox window |
| **OpenedMailWindow** | [openedmailwindow](#openedmailwindow) | Opened mail message window |
| **MapWindow** | [mapwindow](#mapwindow) | Zone map window |
| **TravelMapWindow** | [travelmapwindow](#travelmapwindow) | Travel map window |
| **RadialMenuWindow** | [radialmenuwindow](#radialmenuwindow) | Radial menu window |
| **BrokerWindow** | [brokerwindow](#brokerwindow) | Broker search window |
| **MerchantWindow** | [merchantwindow](#merchantwindow) | Merchant window |
| **ChannelerWindow** | [channelerwindow](#channelerwindow) | Channeler class window |
| **BeastlordWindow** | [beastlordwindow](#beastlordwindow) | Beastlord class window |

---

## DataTypes

### Core Game DataTypes

#### **isxeq2**

Extension information and utilities.

**Members:**
- `Version` - string: ISXEQ2 version
- `APIVersion` - string: API version
- `IsReady` - bool: Whether extension is ready
- `EQ2LocsCount[allzones]` - int: Number of saved locations
- `IsValidEQ2PressKey[keyname]` - bool: Whether key name is valid
- `AfflictionEventsOn` - bool: Whether affliction events enabled
- `GetCustomVariable[name,type]` - variable: Gets custom variable
- `GetCurrencyString[amount]` - string: Formats currency string
- `Round[type,value,multiple]` - variable: Rounds to nearest multiple

**Methods:**
- `ClearAbilitiesCache` - Clears abilities cache
- `ClearRecipesCache` - Clears recipes cache
- `Popup[text,title,status]` - Shows popup window
- `AddLoc[label,notes]` - Adds location
- `EnduringBreath` / `EB` - Toggles enduring breath
- `NoFog` - Toggles fog removal
- `EnableActorEvents` - Enables actor events
- `DisableActorEvents` - Disables actor events
- `SetActorEventsRange[range]` - Sets actor events range
- `SetActorEventsTimeInterval[ms]` - Sets check interval
- `ResetInternalVendingSystem` - Resets vending system
- `EnableAfflictionEvents` - Enables affliction events
- `DisableAfflictionEvents` - Disables affliction events
- `SetAfflictionEventsTimeInterval[ms]` - Sets check interval
- `EnableCustomZoningText` - Enables custom zoning text
- `DisableCustomZoningText` - Disables custom zoning text
- `SetCustomVariable[name,value]` - Sets custom variable
- `ClearAllCustomVariables` - Clears all custom variables
- `InstallBeta` - Switches to beta build
- `InstallTest` - Switches to test build
- `InstallLive` - Switches to live build
- `Reload[delay]` - Reloads extension
- `Unload` - Unloads extension

---

#### **eq2**

Main game information and utilities.

**Members:**
- `ServerName` - string: Server name
- `Zoning` - int: Whether currently zoning
- `CustomActorArraySize` - int: Custom actor array size
- `PendingQuestName` - string: Pending quest name
- `PendingQuestDescription` - string: Pending quest description
- `MasterVolume` - float: Master volume (0-100)
- `NumRadars` - int: Number of radars
- `InboxMailCount` - int: Inbox mail count
- `HOWindowActive` - bool: Whether Heroic Opportunity window is active
- `HOName` - string: Current HO name
- `HODescription` - string: Current HO description
- `HOWheelID` - int: HO wheel ID
- `HOWheelState` - int: HO wheel state
- `HOCurrentWheelSlot` - int: Current HO wheel slot
- `HOWindowState` - int: HO window state
- `HOTimeLimit` - float: HO time limit
- `HOTimeElapsed` - float: HO time elapsed
- `HOTimeRemaining` - float: HO time remaining
- `HOLastManipulator` - [actor](#actor): Last HO manipulator
- `HOIconID1` through `HOIconID6` - int: HO icon IDs
- `CheckCollision[x1,y1,z1,x2,y2,z2]` - bool: Checks collision between two points
- `HeadingTo[x1,y1,z1,x2,y2,z2,asstring]` - float/string: Calculates heading
- `PersistentZoneID[name]` - uint: Gets zone ID by name
- `ReadyToRefineTransmuteOrSalvage` - bool: Whether ready for refine/transmute/salvage
- `AbilityTierString[tier]` - string: Gets tier string (Apprentice, Journeyman, etc.)
- `AtCharSelect` - bool: Whether at character select
- `LoginState` - string: Current login state
- `ObjectBeingMoved` - [moveableobject](#moveableobject): Object being moved

**Methods:**
- `CreateCustomActorArray[sortby,range,type]` - Creates custom actor array (access via `CustomActor[index]` which returns [actor](#actor))
- `GetActors[index,searchparams]` - Populates index with [actor](#actor) objects
- `QueryActors[index,query]` - Populates index with [actor](#actor) objects matching query
- `SetMasterVolume[volume]` - Sets master volume
- `AcceptPendingQuest` - Accepts pending quest
- `DeclinePendingQuest` - Declines pending quest
- `ConfirmZoneTeleporterDestination` - Confirms zone teleporter
- `ShowAllOnScreenAnnouncements` - Toggles announcements
- `GetPersistentZones[index]` - Populates index with persistent zones
- `GetAccountFeatures[index]` - Populates index with account features
- `CreateSoundEffect[file]` - Creates sound effect
- `PlaceMoveableObject` - Places moveable object
- `CancelMoveObject` - Cancels move object
- `OpenTravelMapWindow` - Opens travel map window

---

#### **zone**

Zone information.

**Members:**
- `Name` - string: Zone name
- `ShortName` - string: Zone short name
- `ID` - uint: Zone ID
- `Description` - string: Zone description

---

#### **eq2location**

3D location coordinate.

**Members:**
- `X` - float: X coordinate
- `Y` - float: Y coordinate
- `Z` - float: Z coordinate
- `Distance[x,y,z]` - float: Distance to specified coordinates
- `Distance2D[x,z]` - float: 2D distance to coordinates

---

#### **radar**

Radar functionality.

**Members:**
- `NumActors` - int: Number of actors on radar
- `Actor[index]` - [actor](#actor): Actor by index

---

#### **class**

Class information.

**Members:**
- `Name` - string: Class name
- `Level` - int: Class level

---

#### **achievement**

Achievement information.

**Members:**
- `Name` - string: Achievement name
- `ID` - int: Achievement ID
- `Description` - string: Achievement description
- `Points` - int: Achievement points
- `IsComplete` - bool: Whether achievement is complete

---

#### **reward**

Quest/loot reward.

**Members:**
- `Name` - string: Reward name
- `LinkID` - int: Link ID
- `ToLink` - string: Creates clickable chat link
- `Type` - string: Reward type
- `IsItemInfoAvailable` - bool: Whether item info available
- `ToItemInfo` - [iteminfo](#iteminfo): Converts to iteminfo

**Methods:**
- `Examine` - Examines the reward

---

### Actor DataTypes

#### **actor**

Base datatype for all actors (NPCs, PCs, objects) in the game world.

**Members:**

*Identity:*
- `Name` - string: Actor's name
- `LastName` - string: Actor's last name
- `ID` - uint: Unique actor ID
- `Type` - string: Actor type (PC, NPC, NamedNPC, Pet, MyPet, Chest, etc.)
- `Gender` - string: Gender (Male/Female)
- `Race` - string: Race (returns "NPC" for non-player races)
- `Class` - string: Class name
- `Level` - int: Actor's level
- `EffectiveLevel` - int: Effective/mentor level
- `Guild` - string: Guild tag
- `SuffixTitle` - string: Suffix title
- `Tooltip` - string: Sign/tooltip text

*Position & Movement:*
- `X` - float: X coordinate
- `Y` - float: Y coordinate
- `Z` - float: Z coordinate
- `Loc` - point3f: 3D location (X,Y,Z)
- `Distance[x,y,z]` - float: 3D distance from player or specified point
- `Distance2D[x,z]` - float: 2D distance from player or specified point
- `Heading` - float: Current heading (0-360)
- `HeadingTo[asstring]` - float/string: Heading from player to actor
- `Speed` - float: Speed modifier
- `Velocity` - point3f: Velocity vector

*Stats & Combat:*
- `Health` - int: Current health
- `Power` - int: Current power/mana
- `Difficulty` - int: Difficulty arrows (up arrows)
- `EncounterSize` - int: Number in encounter
- `Faction` - int: Faction value (negative = hostile)
- `FactionStanding` - string: Faction standing text
- `ConColor[raw]` - string: Con color (or RGB if "raw")
- `ThreatToMe` - int: Threat to player
- `ThreatToPet` - int: Threat to player's pet
- `ThreatToNext` - int: Secondary threat

*Relationships:*
- `Target` - [actor](#actor): Actor's target
- `Pet` - [actor](#actor): Actor's pet
- `WhoFollowing` - string: Name of followed actor
- `WhoFollowingID` - int: ID of followed actor
- `InMyGroup` - bool: TRUE if in player's group
- `IsInSameEncounter[actorID]` - bool: TRUE if in same encounter

*Effects:*
- `NumEffects` - int: Number of effects
- `Effect[index|"id",id|"name"|"Query","query"]` - [actoreffect](#actoreffect): Get effect

*Appearance:*
- `CollisionRadius` - float: Collision radius
- `CollisionScale` - float: Collision scale
- `TargetRingRadius` - float: Target ring radius
- `CurrentAnimation` - string: Current animation
- `Overlay` - string: Overlay animation
- `Aura` - string: Aura effect
- `Mood` - string: Mood animation
- `VisualVariant` - string: Visual variant
- `TintFlags` - uint: Tint flags
- `EquipmentAppearance[slot]` - [equipmentappearance](#equipmentappearance): Equipment appearance at slot (1-34)
- `MountAppearance` - [equipmentappearance](#equipmentappearance): Mount appearance

*Casting:*
- `AbilityCastingID` - int: ID of ability being cast
- `AbilityCastingTime` - float: Time remaining for cast

*Tags:*
- `TagTargetNumber` - string: Tag number (1-6 or empty)
- `TagTargetIcon` - string: Tag icon (Skull, Shield, Star, Sword, Cross, Flame)

*State Checks:*
- `IsAggro` - bool: Aggressive/KOS
- `IsSolo` - bool: Solo encounter
- `IsHeroic` - bool: Heroic encounter
- `IsEpic` - int: Epic tier (0 if not epic)
- `IsEncounterBroken` - bool: Encounter broken
- `IsAFK` - bool: AFK flag
- `IsLFG` - bool: Looking for group
- `IsLFW` - bool: Looking for work
- `IsCamping` - bool: Camping out
- `IsLocked` - bool: Locked
- `IsAPet` - bool: Is a pet
- `IsMyPet` - bool: Is player's pet
- `IsNamed` - bool: Is named NPC
- `IsSwimming` - bool: Swimming
- `SwimmingSpeedMod` - float: Swimming speed mod
- `InCombatMode` - bool: Combat stance
- `IsCrouching` - bool: Crouching
- `IsSitting` - bool: Sitting
- `OnTransport` - bool: On mount/transport
- `IsRunning` - bool: Running
- `IsWalking` - bool: Walking
- `IsSprinting` - bool: Sprinting
- `IsBackingUp` - bool: Moving backward
- `IsStrafingLeft` - bool: Strafing left
- `IsStrafingRight` - bool: Strafing right
- `IsIdle` - bool: Idle (not moving)
- `IsClimbing` - bool: Climbing
- `IsJumping` - bool: Jumping
- `IsFalling` - bool: Falling
- `IsDead` - bool: Dead
- `IsInvis` - bool: Invisible
- `IsStealthed` - bool: Stealthed
- `IsBanker` - bool: Is banker
- `IsMerchant` - bool: Is merchant
- `IsChest` - bool: Is chest
- `IsRooted` - bool: Rooted
- `CanTurn` - bool: Can turn
- `IsFD` - bool: Feign death
- `Interactable` - bool: Highlights on hover
- `HighlightOnMouseHover` - bool: Highlights on hover
- `FlyingUsingMount` - bool: Flying with mount
- `OnFlyingMount` - bool: On flying mount
- `UpdatesMyQuest` - bool: Updates player's quest
- `UpdatesGroupMemberQuest` - bool: Updates group quest
- `ActiveStateExists` - bool: Has active states
- `OffersQuest` - bool: Offers quest
- `CheckCollision[x,y,z]` - bool: Checks collision to point

**Methods:**
- `DoubleClick` - Double-clicks actor
- `RightClick` - Right-clicks actor
- `DoTarget` - Targets actor
- `DoFace` - Faces toward actor
- `WaypointTo` / `WT` - Creates waypoint to actor
- `Location` - Prints location info
- `RequestEffectsInfo` - Requests effects from server
- `GetActiveStates` - Gets active states
- `Resize[scale]` - Resizes actor (visual)
- `Move[x,y,z]` - Moves actor to coordinates
- `Set[actorID]` - Sets actor reference

---

#### **char**

Player character. Inherits from [**actor**](#actor).

**Inheritance:**
- All members and methods from [**actor**](#actor) are available

**Additional Members:**

*Stats:*
- `CurrentHealth` / `Health` - int: Current health
- `MaxHealth` - int: Maximum health
- `CurrentPower` / `Power` - int: Current power
- `MaxPower` - int: Maximum power
- `UsedConc` - int: Used concentration
- `MaxConc` - int: Maximum concentration
- `Strength` - int: Strength stat
- `Stamina` - int: Stamina stat
- `Agility` - int: Agility stat
- `Wisdom` - int: Wisdom stat
- `Intelligence` - int: Intelligence stat
- `BaseStrength` - int: Base strength (no buffs)
- `BaseStamina` - int: Base stamina
- `BaseAgility` - int: Base agility
- `BaseWisdom` - int: Base wisdom
- `BaseIntelligence` - int: Base intelligence

*Resistances:*
- `ElementalResist` - int: Elemental resistance
- `NoxiousResist` - int: Noxious resistance
- `ArcaneResist` - int: Arcane resistance
- `ElementalResistPct` - float: Elemental resist %
- `NoxiousResistPct` - float: Noxious resist %
- `ArcaneResistPct` - float: Arcane resist %

*Currency:*
- `Copper` - int: Copper coins
- `Silver` - int: Silver coins
- `Gold` - int: Gold coins
- `Platinum` - int: Platinum coins

*Class Info:*
- `Archetype` - string: Character archetype
- `SubClass` - string: Character subclass
- `TSArchetype` - string: Tradeskill archetype
- `TSClass` - string: Tradeskill class
- `TSSubClass` - string: Tradeskill subclass
- `TSLevel` - int: Tradeskill level

*Effects & Maintained:*
- `CountEffects` - int: Number of effects
- `CountMaintained` - int: Number of maintained
- `Effect[index|"id",id|"name"]` - [effect](#effect): Get effect
- `Maintained[index]` - [maintained](#maintained): Get maintained spell
- `NumEffects` - int: Number of effects

*Inventory:*
- `Inv[slot]` / `Inventory[slot|"Query","query"]` - [item](#item): Inventory item
- `Equip[slot]` / `Equipment[slot]` - [item](#item): Equipped item
- `CustomInventory[index]` - [item](#item): Custom inventory item
- `CustomInventoryArraySize` - int: Custom inventory size
- `NextFreeInvContainer` - int: Next free container slot
- `InventorySlotsFree` - int: Free inventory slots
- `BankSlotsFree` - int: Free bank slots
- `SharedBankSlotsFree` - int: Free shared bank slots

*Group & Raid:*
- `Group[index]` - [groupmember](#groupmember): Group member
- `GroupCount` - int: Group size
- `Grouped` - bool: In a group
- `IsGroupLeader` - bool: Is group leader
- `Raid[index]` - [raidmember](#raidmember): Raid member
- `RaidCount` - int: Raid size
- `InRaid` - bool: In a raid
- `RaidGroupNum` - int: Raid group number

*Abilities & Recipes:*
- `NumAbilities` - int: Number of abilities
- `Ability[index|"name"]` - [ability](#ability): Get ability
- `NumRecipes` - int: Number of recipes
- `Recipe[index|"name"]` - [recipe](#recipe): Get recipe

*State:*
- `InGameWorld` - bool: In game world
- `AtCharSelect` - bool: At character select
- `CastingSpell` - bool: Currently casting
- `AutoAttackOn` - bool: Auto-attack enabled
- `RangedAutoAttackOn` - bool: Ranged auto-attack on
- `IsMoving` - bool: Character moving
- `In1stPersonView` - bool: First-person view
- `In3rdPersonView` - bool: Third-person view
- `IsSitting` - bool: Sitting
- `InWater` - bool: In water
- `InCombat` - bool: In combat
- `TargetLOS` - bool: Target in line of sight
- `WaterDepth` - float: Water depth
- `TimeToCampOut` - float: Seconds to camp out

*Flags:*
- `IsAnonymous` - bool: Anonymous
- `IsRolePlaying` - bool: Roleplaying
- `IsDecliningGroupInvites` - bool: Declining groups
- `IsDecliningTradeInvites` - bool: Declining trades
- `IsDecliningDuelInvites` - bool: Declining duels
- `IsDecliningRaidInvites` - bool: Declining raids
- `IsDecliningGuildInvites` - bool: Declining guilds
- `IgnoringAll` - bool: Ignoring all
- `GuildPrivacyOn` - bool: Guild privacy
- `CombatExpEnabled` - bool: Combat XP enabled

*Afflictions:*
- `Arcane` - int: Arcane afflictions
- `Elemental` - int: Elemental afflictions
- `Trauma` - int: Trauma afflictions
- `Noxious` - int: Noxious afflictions
- `Cursed` - int: Cursed afflictions
- `IsAfflicted` - bool: Has afflictions

*Channeler Specific:*
- `Dissonance` - int: Current dissonance
- `DissonanceRemaining` - int: Remaining dissonance
- `MaxDissonance` - int: Max dissonance
- `Dissipation` - int: Dissipation

*Misc:*
- `CursorActor` - [actor](#actor): Actor under cursor
- `TotalEarnedAPs` - int: Total achievement points
- `MaxAPs` - int: Max achievement points
- `Achievement[index|"name"]` - [achievement](#achievement): Get achievement
- `IsInPVP` - bool: In PVP zone
- `IsHated` - bool: Hated
- `PowerRegen` - float: Power regen rate
- `HealthRegen` - float: Health regen rate
- `InZone` - bool: Fully in zone
- `CameraPitch` - float: Camera pitch angle

*Collections:*
- `GetInventory` - collection: Returns inventory collection (LavishScript collection of [item](#item) objects)
- `GetInventoryAtHand` - collection: Returns at-hand inventory (LavishScript collection of [item](#item) objects)
- `GetEquipment` - collection: Returns equipment collection (LavishScript collection of [item](#item) objects)
- `GetAbilities` - collection: Returns abilities collection (LavishScript collection of [ability](#ability) objects)

*Dynamic Game Data:*
- `GetGameData["string"]` - [eq2dynamicdata](#eq2dynamicdata): Retrieves dynamic game data values
  - Returns an eq2dynamicdata object (also known as eq2uielement) with `.Label`, `.ShortLabel`, and `.Percent` members
  - Used to access many character values that were previously direct members
  - Example: `${Me.GetGameData[Self.ExperienceNextLevel].Label}`.  See "GetGameData Examples" below for more.

**Methods:**

- `Face[heading|x,y,z|"actor",id]` - Face direction/coordinates/actor
- `CreateCustomInventoryArray` - Creates custom inventory array (access via `CustomInventory[index]` which returns [item](#item))
- `TakeAllVendingCoin` - Takes all vending coin
- `BankDeposit[amount]` - Deposits to bank
- `BankWithdraw[amount]` - Withdraws from bank
- `GuildBankDeposit[amount]` - Deposits to guild bank
- `GuildBankWithdraw[amount]` - Withdraws from guild bank
- `SharedBankDeposit[amount]` - Deposits to shared bank
- `SharedBankWithdraw[amount]` - Withdraws from shared bank
- `ResetZoneTimer` - Resets zone timer
- `DepositIntoHouseEscrow[amount]` - Deposits to house escrow
- `QueryInventory[query]` - Queries inventory (populates index with [item](#item) objects)
- `QueryEffects[query]` - Queries effects (populates index with [effect](#effect) objects)
- `RequestEffectsInfo` - Requests effects info
- `QueryAbilities[query]` - Queries abilities (populates index with [ability](#ability) objects)
- `QueryRecipes[query]` - Queries recipes (populates index with [recipe](#recipe) objects)
- `Set[characterID]` - Sets character reference
- `SetCameraPitch[pitch]` - Sets camera pitch

**GetGameData Examples:**

The `GetGameData` member provides access to dynamic game data values. It returns an eq2dynamicdata object with `.Label`, `.ShortLabel`, and `.Percent` members.

*Experience Values:*
```lavishscript
; Adventure Experience
${Me.GetGameData[Self.Experience].Label}
${Me.GetGameData[Self.ExperienceNextLevel].Label}
${Me.GetGameData[Self.ExperienceCurrent].Label}
${Me.GetGameData[Self.ExperienceDebtCurrent].Label}
${Me.GetGameData[Self.ExperienceBubble].Label}

; Vitality
${Me.GetGameData[Self.Vitality].Label}
${Me.GetGameData[Self.VitalityUpperMarker].Label}
${Me.GetGameData[Self.VitalityLowerMarker].Label}
${Me.GetGameData[Self.VitalityOverflowMarker].Label}

; Tradeskill Experience
${Me.GetGameData[Self.TradeskillExperience].Label}
${Me.GetGameData[Self.TradeskillExperienceNextLevel].Label}
${Me.GetGameData[Self.TradeskillExperienceCurrent].Label}
${Me.GetGameData[Self.TSExperienceDebtCurrent].Label}
${Me.GetGameData[Self.TradeskillExperienceBubble].Label}

; Tradeskill Vitality
${Me.GetGameData[Self.TSVitality].Label}
${Me.GetGameData[Self.TSVitalityUpperMarker].Label}
${Me.GetGameData[Self.TSVitalityLowerMarker].Label}
${Me.GetGameData[Self.TSVitalityOverflowMarker].Label}

; Tithe Experience
${Me.GetGameData[Self.TitheExperience].Label}
${Me.GetGameData[Self.TitheExperienceNextLevel].Label}
${Me.GetGameData[Self.TitheExperienceCurrent].Label}
${Me.GetGameData[Self.TitheExperienceBubble].Label}
${Me.GetGameData[Self.TitheVitalityOverflowMarker].Label}

; Ascension Experience
${Me.GetGameData[Self.AscensionLevel].Label}
${Me.GetGameData[Self.AscensionExperience].Label}
${Me.GetGameData[Self.AscensionExperienceNextLevel].Label}
${Me.GetGameData[Self.AscensionExperienceCurrent].Label}
${Me.GetGameData[Self.AscensionExperienceBubble].Label}
```

*Pet Information:*
```lavishscript
; Pet data (replaces deprecated PetName, PetHealth, PetPower members)
${Me.GetGameData[Pet.Name].Label}
${Me.GetGameData[Pet.ActualHealth].ShortLabel.Left[-1]}
${Me.GetGameData[Pet.Health].ShortLabel.Left[-1]}
${Me.GetGameData[Pet.ActualPower].ShortLabel.Left[-1]}
${Me.GetGameData[Pet.Power].ShortLabel.Left[-1]}
```

*Bank Coin Information:*
```lavishscript
; Bank coin values
${Me.GetGameData[Coins.BankCoin_0].ShortLabel}     ; Bank copper
${Me.GetGameData[Coins.BankCoin_1].ShortLabel}     ; Bank silver
${Me.GetGameData[Coins.BankCoin_2].ShortLabel}     ; Bank gold
${Me.GetGameData[Coins.BankCoin_3].ShortLabel}     ; Bank platinum

; Shared bank coin values
${Me.GetGameData[Coins.SharedCoin_0].ShortLabel}   ; Shared Bank copper
${Me.GetGameData[Coins.SharedCoin_1].ShortLabel}   ; Shared Bank silver
${Me.GetGameData[Coins.SharedCoin_2].ShortLabel}   ; Shared Bank gold
${Me.GetGameData[Coins.SharedCoin_3].ShortLabel}   ; Shared Bank platinum
```

*Target Information:*
```lavishscript
; Target threat and other data
${Me.GetGameData[Target.Threat].Label}
```

---

#### **groupmember**

Group member. Inherits from [**actor**](#actor).

**Inheritance:**
- All members and methods from [**actor**](#actor) are available

**Additional Members:**
- `ID` - int: Group member ID
- `PetID` - int: Pet ID
- `ToActor` - [actor](#actor): Returns actor datatype
- `EffectiveLevel` - int: Effective level
- `Noxious` - int: Noxious afflictions
- `Cursed` - int: Cursed afflictions
- `Arcane` - int: Arcane afflictions
- `Elemental` - int: Elemental afflictions
- `Trauma` - int: Trauma afflictions
- `IsAfflicted` - bool: Has afflictions
- `RaidRole` - string: Raid role
- `RaidGroupNum` - int: Raid group number
- `InZone` - bool: In zone

---

#### **raidmember**

Raid member. This is the same as [**groupmember**](#groupmember) - see that datatype for all members and methods.

**Inheritance:**
- All members and methods from [**groupmember**](#groupmember) (and [**actor**](#actor)) are available

**Note:** When you're in a raid, both `${Me.Group[index]}` and `${Me.Raid[index]}` return this datatype.

---

#### **moveableobject**

Moveable house object (for housing).

**Members:**
- `OldLocation` - point3f: Location before object was moved
- `OldHeading` - float: Heading before object was moved
- `OldScale` - float: Scale before object was moved
- `NewHeading` - float: New heading
- `NewScale` - float: New scale
- `HeightOffset` - float: Height difference between old and current locations
- `CurrentMouseX` - int: Current mouse X position
- `CurrentMouseY` - int: Current mouse Y position
- `ActorID` - uint: ID of the actor being moved

**Notes:**
- Available via `${EQ2.ObjectBeingMoved}` when an object is being moved
- Returns NULL if no object is currently being moved
- Used in conjunction with `EQ2:PlaceMoveableObject` and `EQ2:CancelMoveObject` methods

---

#### **equipmentappearance**

Equipment appearance information for actor equipment slots.

**Members:**
- `ID` - uint: Appearance ID
- `Texture1` - string: First texture
- `Texture2` - string: Second texture
- `Texture3` - string: Third texture

---

### Item DataTypes

#### **item**

Item in inventory, equipment, or containers.

**Members:**

*Identity:*
- `Name` - string: Item name
- `ID` - uint: Item unique ID
- `LinkID` - int: Link ID for chat links
- `ToLink` - string: Creates chat link
- `SerialNumber` - int64: Item serial number
- `IconID` - int: Icon ID

*Location:*
- `Location` - string: Location name (Inventory, Bank, etc.)
- `LocationID` - int: Numeric location ID
- `InContainerID` - int: Container ID if in container
- `ContainerID` - int: Container ID if this IS a container
- `Slot` - int: Slot number
- `Bag` - int: Bag slot if in inventory container
- `Index` - int: Index in inventory array

*Properties:*
- `Quantity` - int: Stack quantity
- `EffectiveLevel` - int: Effective level

*Container Properties:*
- `IsContainer` - bool: Is a container
- `NumSlots` - int: Number of slots (container)
- `NumSlotsFree` - int: Free slots (container)
- `InContainer` - bool: Is inside a container
- `InInventorySlot` - bool: In inventory slot
- `IsInventoryContainer` - bool: Is inventory container
- `IsBankContainer` - bool: Is bank container
- `IsSharedBankContainer` - bool: Is shared bank container
- `IsSlotOpen[slot]` - bool: Slot is open (container)
- `ItemInSlot[slot]` - [item](#item): Item in slot (container)
- `NextSlotOpen` - int: Next open slot (container)

*Item Type Checks:*
- `IsAutoConsumeable` - bool: Can auto-consume
- `AutoConsumeOn` - bool: Auto-consume active
- `CanBeRedeemed` - bool: Can be redeemed
- `IsFoodOrDrink` - bool: Is food/drink
- `IsScribeable` - bool: Can be scribed
- `IsUnpackable` - bool: Can be unpacked
- `IsFamiliar` - bool: Is familiar
- `IsAgent` - bool: Is agent
- `IsUsable` - bool: Is usable

*Readiness:*
- `IsReady` - bool: Ready (not on cooldown)
- `TimeUntilReady` - float: Seconds until ready (-1 if ready)

*Item Info:*
- `IsItemInfoAvailable` - bool: ItemInfo data available
- `ToItemInfo` - [iteminfo](#iteminfo): Detailed item information

**Methods:**
- `Destroy` - Destroys item (no confirmation)
- `DestroyWithConf` - Destroys with confirmation
- `Move[location,slot|"container",containerID,slot]` - Moves item
- `Equip` - Equips item
- `UnEquip` - Unequips item
- `Consume` - Consumes item
- `Examine` - Examines item
- `Use` - Uses item
- `Activate` - Activates item
- `Open` - Opens container
- `SendAsGift[characterName]` - Sends as gift
- `InstallAsVendingContainer` - Installs as vendor
- `AddToConsignment` - Adds to consignment
- `Transmute["confirm"]` - Transmutes item
- `Refine["confirm"]` - Refines item
- `Salvage["confirm"]` - Salvages item
- `Extract["confirm"]` - Extracts mana
- `ReclaimAdornments["confirm"]` - Reclaims adornments
- `Sacrifice` - Sacrifices item
- `Scribe` - Scribes spell/art/recipe
- `ToggleAutoConsume` - Toggles auto-consume
- `AddToDepot` - Adds to depot
- `ApplyToItem[itemID]` - Applies to item
- `EnchantItem[itemID]` - Enchants item
- `Read` - Reads book
- `Unpack` - Unpacks item
- `AddFamiliar` - Adds familiar
- `EquipFamiliar` - Equips familiar
- `UnequipFamiliar` - Unequips familiar
- `SetAppearanceFamiliar` - Sets familiar appearance
- `UnsetAppearanceFamiliar` - Unsets familiar appearance
- `AddAgent` - Adds agent
- `ConvertAgent` - Converts agent
- `Set[itemID]` - Sets item reference

**See Also:**
- [iteminfo](#iteminfo) - Detailed item information (accessed via `Item.ToItemInfo`)
- [char](#char) - Character inventory access via `Me.Inventory[slot]` or `Me.Equip[slot]`

---

#### **iteminfo**

Detailed item examination data. Obtained via `Item.ToItemInfo`.

**Members:**

*Identity:*
- `Name` - string: Item name
- `ID` - uint: Item ID
- `LinkID` - int: Link ID
- `ToLink` - string: Chat link
- `SerialNumber` - int64: Serial number
- `Description` - string: Item description
- `Label` - string: Item label
- `IconID` - int: Icon ID

*Type & Tier:*
- `Type` - string: Item type (Weapon, Armor, etc.)
- `SubType` - string: Item subtype
- `Tier` - int: Item tier
- `Level` - int: Required level

*Container:*
- `NumSlots` - int: Container slots
- `EmptySlots` - int: Empty slots
- `ContentsForSale` - bool: Contents for sale

*Equipment:*
- `WieldStyle` - string: Wield style (One-Hand, Two-Hand, etc.)
- `NumEquipSlots` - int: Number of equip slots
- `EquipSlot[index]` - string: Equipment slot name

*Condition & Charges:*
- `Condition` - int: Condition percentage
- `Charges` - int: Current charges
- `MaxCharges` - int: Max charges
- `Crafter` - string: Crafter name

*Class Restrictions:*
- `NumClasses` - int: Number of usable classes
- `Class[index]` - string: Class name

*Consumable:*
- `Satiation` - int: Satiation value (food/drink)
- `Duration` - int: Effect duration

*Combat Stats (Weapons):*
- `DamageRating` - int: Damage rating
- `MyMinDamage` - int: Player's min damage
- `MyMaxDamage` - int: Player's max damage
- `BaseMinDamage` - int: Base min damage
- `BaseMaxDamage` - int: Base max damage
- `MasteryMinDamage` - int: Mastery min damage
- `MasteryMaxDamage` - int: Mastery max damage
- `Delay` - float: Weapon delay
- `Range` / `MaxRange` - float: Weapon range
- `MinRange` - float: Minimum range
- `DamageType` - string: Damage type
- `DamageVerbType` - string: Damage verb

*Defense Stats (Armor/Shields):*
- `Mitigation` - int: Mitigation value
- `MaxMitigation` - int: Max mitigation
- `ShieldFactor` - int: Shield factor
- `MaxShieldFactor` - int: Max shield factor
- `Protection` - int: Protection value
- `MaxProtection` - int: Max protection

*Modifiers:*
- `NumModifiers` - int: Number of stat modifiers
- `Modifier[index]` - [itemmodifier](#itemmodifier): Stat modifier
- `ModFlag` - int: Modifier flags

*Effects:*
- `NumEffects` - int: Number of effects
- `EffectName[index]` - string: Effect name
- `EffectDescription[index]` - string: Effect description
- `NumEffectStrings` - int: Number of effect strings
- `EffectString[index]` - [itemeffectstring](#itemeffectstring): Effect string
- `CastingTime` - float: Casting time
- `RecoveryTime` - float: Recovery time
- `RecastTime` - float: Recast time

*Adornments:*
- `NumAdornmentsAttached` - int: Attached adornments
- `Adornment[index]` - [adornment](#adornment): Adornment data

*Packaging:*
- `NumItemsPackaged` - int: Packaged items count
- `PackagedItem[index]` - [packageditem](#packageditem): Packaged item

*Recipe:*
- `NumItemsCreated` - int: Items created by recipe
- `CreatesItem[index]` - string: Created item name

*Restrictions & Flags:*
- `RentStatusReduction` - int: Rent status reduction
- `TradeRestrictedDate` - string: Trade restriction date
- `Attuned` - bool: Is attuned
- `Attuneable` - bool: Can be attuned
- `Lore` - bool: Is lore
- `LoreOnEquip` - bool: Lore on equip
- `Artifact` - bool: Is artifact
- `Ornate` - bool: Is ornate
- `Temporary` - bool: Is temporary
- `NoTrade` - bool: No-trade
- `NoValue` - bool: No value
- `NoZone` - bool: Cannot zone with
- `NoDestroy` - bool: Cannot destroy
- `NoRepair` - bool: Cannot repair
- `NoSalvage` - bool: Cannot salvage
- `NoTransmute` - bool: Cannot transmute
- `NoExperiment` - bool: Cannot experiment
- `NoBroker` - bool: Cannot broker
- `NoMail` - bool: Cannot mail
- `Indestructable` - bool: Indestructable
- `Heirloom` - bool: Is heirloom
- `HouseLore` - bool: House lore
- `BuildingBlock` - bool: Is building block
- `FreeReforge` - bool: Free reforge available
- `Infusable` - bool: Is infusable
- `MercOnly` - bool: Mercenary only
- `MountOnly` - bool: Mount only
- `RequiresEquip` - bool: Requires being equipped
- `AppearanceOnly` - bool: Appearance only
- `Good` - bool: Good-aligned
- `Evil` - bool: Evil-aligned

*Quest & Collectible:*
- `IsCollectible` - bool: Is collectible
- `AlreadyCollected` - bool: Already collected
- `OffersQuest` - bool: Offers quest
- `RequiredByQuest` - bool: Required by quest
- `IsQuestItemUsable` - bool: Quest item usable

*State:*
- `IsActivatable` - bool: Can be activated
- `CanScribeNow` - bool: Can scribe now
- `Unlocked` - bool: Is unlocked
- `Reforged` - bool: Is reforged
- `Refined` - bool: Is refined

**Methods:**
- `AttachAsAdornment[itemID]` - Attaches as adornment
- `PrepAdornmentForUse` - Prepares adornment

**See Also:**
- [item](#item) - Basic item object (use `Item.ToItemInfo` to get detailed info)
- [itemmodifier](#itemmodifier) - Item stat modifiers (accessed via `ItemInfo.Modifier[index]`)
- [adornment](#adornment) - Adornment information (accessed via `ItemInfo.Adornment[index]`)

---

#### **itemmodifier**

Item stat modifier.

**Members:**
- `Type` - string: Modifier type
- `Value` - int: Modifier value

---

#### **itemeffectstring**

Item effect description string.

**Members:**
- `Description` - string: Effect description

---

#### **packageditem**

Packaged item (in reward pack or examine window).

**Members:**
- `Name` - string: Item name
- `IconID` - int: Icon ID
- `Quantity` - int: Quantity

---

#### **adornment**

Adornment attached to item.

**Members:**
- `Name` - string: Adornment name
- `Slot` - string: Adornment slot
- `SlotIndex` - int: Numeric slot index
- `Description` - string: Description

---

### Ability DataTypes

#### **ability**

Character ability/spell.

**Members:**
- `ID` - uint: Ability ID
- `TimeUntilReady` - float: Seconds until ready
- `IsReady` - bool: Ready to cast
- `IsQueued` - bool: Is queued
- `IsAbilityInfoAvailable` - bool: Info available
- `ToAbilityInfo` - [abilityinfo](#abilityinfo): Detailed info
- `MainIconID` - int: Main icon ID
- `BackDropIconID` - int: Backdrop icon ID
- `HOIconID` - int: HO icon ID
- `Level` - int: Ability level
- `IsConduit` - bool: Is Channeler conduit

**Methods:**
- `Use` - Uses/casts ability
- `Examine` - Examines ability
- `AddToConduitBar[slot]` - Adds to Channeler conduit bar (1-8)
- `AddToChannelerBar[slot]` - Adds to Channeler bar (1-10)
- `AddToBeastlordBar[slot]` - Adds to Beastlord bar (1-10)
- `Set[ID]` - Sets ability by ID

**See Also:**
- [abilityinfo](#abilityinfo) - Detailed ability information (accessed via `Ability.ToAbilityInfo`)
- [char](#char) - Character abilities access via `Me.Ability[index]`

---

#### **abilityinfo**

Detailed ability information.

**Members:**
- `Name` - string: Ability name
- `Description` - string: Description
- `Tier` - string: Ability tier
- `HealthCost` - int64: Health cost
- `PowerCost` - int64: Power cost
- `SavageryCost` - int: Savagery cost
- `DissonanceCost` - int: Dissonance cost
- `ConcentrationCost` - int: Concentration cost
- `MainIconID` - int: Main icon ID
- `HOIconID` - int: HO icon ID
- `BackDropIconID` - int: Backdrop icon ID
- `CastingTime` - float: Casting time (seconds)
- `RecoveryTime` - float: Recovery time
- `RecastTime` - float: Recast time
- `MaxDuration` - float: Max duration (-1 if permanent)
- `ToLink[text]` - string: Creates chat link
- `HealthCostPerTick` - int64: Health cost per tick
- `PowerCostPerTick` - int64: Power cost per tick
- `DissonanceCostPerTick` - int: Dissonance cost per tick
- `SavageryCostPerTick` - int: Savagery cost per tick
- `MaxAOETargets` - int: Max AOE targets
- `DoesNotExpire` - bool: Doesn't expire
- `GroupRestricted` - bool: Group restricted
- `AllowRaid` - bool: Allowed in raid
- `SpellBookType` - int: Spell book type
- `TargetType` - int: Target type ID
- `EffectRadius` - float: Effect radius
- `NumEffects` - int: Number of effects
- `Effect[index]` - [effectstring](#effectstring): Effect by index
- `NumClasses` - int: Number of classes
- `Class[index|name]` - [class](#class): Class by index/name
- `MinRange` - float: Minimum range
- `Range` / `MaxRange` - float: Maximum range
- `IsBeneficial` - bool: Is beneficial
- `AscensionClass` - string: Ascension class
- `AscensionLevel` - int: Ascension level

**See Also:**
- [ability](#ability) - Basic ability object (use `Ability.ToAbilityInfo` to get detailed info)
- [effectstring](#effectstring) - Effect descriptions (accessed via `AbilityInfo.Effect[index]`)

---

### Effect DataTypes

#### **effect**

Character effect/buff.

**Members:**
- `CurrentIncrements` - int: Stack count
- `ID` - int: Effect ID
- `MainIconID` - int: Main icon ID
- `BackDropIconID` - int: Backdrop icon ID
- `MaxDuration` - float: Max duration (-1 if permanent)
- `Duration` - float: Time remaining (seconds)
- `Type` - string: Effect type
- `IsEffectInfoAvailable` - bool: Info available
- `ToEffectInfo` - [effectinfo](#effectinfo): Detailed info

**Methods:**
- `Cancel` - Cancels effect
- `Examine` - Examines effect
- `Hide` - Hides effect
- `Set[ID]` - Sets effect by ID

**See Also:**
- [effectinfo](#effectinfo) - Detailed effect information (accessed via `Effect.ToEffectInfo`)
- [maintained](#maintained) - Maintained spell effects on player
- [actoreffect](#actoreffect) - Effects on other actors
- [char](#char) - Character effects access via `Me.Effect[index]`

---

#### **maintained**

Maintained buff/spell.

**Members:**
- `Name` - string: Effect name
- `TargetType` - string: Target type (Self, SingleTarget, etc.)
- `CurrentIncrements` - int: Stack count
- `IsBeneficial` - bool: Is beneficial
- `ConcentrationCost` - int: Concentration cost
- `MaxDuration` - float: Max duration (-1 if permanent)
- `Duration` - float: Time remaining
- `Target` - [actor](#actor): Target of effect
- `UsesRemaining` - int64: Uses remaining
- `DamageRemaining` - int64: Damage remaining

**Methods:**
- `Cancel` - Cancels maintained effect
- `Examine` - Examines effect

**See Also:**
- [effect](#effect) - Regular character effects
- [actoreffect](#actoreffect) - Effects on other actors
- [char](#char) - Maintained effects access via `Me.Maintained[index]`

---

#### **actoreffect**

Effect on another actor.

**Members:**
- `CurrentIncrements` - int: Stack count
- `ID` - int: Effect ID
- `MainIconID` - int: Main icon ID
- `BackDropIconID` - int: Backdrop icon ID
- `IsEffectInfoAvailable` - bool: Info available
- `ToEffectInfo` - [effectinfo](#effectinfo): Detailed info

**Methods:**
- `Examine` - Examines effect

**See Also:**
- [effect](#effect) - Regular character effects
- [effectinfo](#effectinfo) - Detailed effect information (accessed via `ActorEffect.ToEffectInfo`)
- [actor](#actor) - Actor effects access via `Actor.Effect[index]`

---

#### **effectinfo**

Detailed effect information.

**Members:**
- `Name` - string: Effect name
- `Description` - string: Effect description
- `Type` - string: Effect type
- `NumEffectStrings` - int: Number of effect strings
- `EffectString[index]` - [effectstring](#effectstring): Effect string
- `UsesRemaining` - int: Uses remaining

**See Also:**
- [effect](#effect) - Basic effect object (use `Effect.ToEffectInfo` to get detailed info)
- [actoreffect](#actoreffect) - Effects on actors (use `ActorEffect.ToEffectInfo` to get detailed info)
- [effectstring](#effectstring) - Effect descriptions (accessed via `EffectInfo.EffectString[index]`)

---

#### **effectstring**

Effect description string.

**Members:**
- `PercentSuccess` - int: Percent success
- `Indentation` - int: Indentation level
- `Description` / `Desc` - string: Effect description

---

### Quest DataTypes

#### **quest**

Quest information.

**Members:**
- `ID` - uint: Quest ID
- `Level` - int: Quest level
- `Name` - string: Quest name
- `Category` - string: Quest category
- `CurrentZone` - string: Current zone

**Methods:**
- `Delete` - Deletes quest
- `Share` - Shares quest with group
- `MakeCurrentActiveQuest` - Makes current active quest
- `MakeCurrentCompletedQuest` - Makes current completed quest

---

#### **currentquest**

Currently selected quest in journal.

**Members:**
- `Name` - [eq2text](#eq2text): Quest name
- `Level` - [eq2text](#eq2text): Quest level
- `Category` - [eq2text](#eq2text): Quest category
- `CurrentZone` - [eq2text](#eq2text): Current zone
- `TimeStamp` - [eq2text](#eq2text): Time stamp
- `MissionGroup` - [eq2text](#eq2text): Mission group
- `Status` - [eq2text](#eq2text): Quest status
- `ExpirationTime` - [eq2text](#eq2text): Expiration time
- `Body` - [eq2text](#eq2text): Quest body text

**Methods:**
- `GetDetails[index]` - Gets quest details in index

---

### Commerce DataTypes

#### **merchandise**

Item for sale from merchant.

**Members:**
- `Name` - string: Item name
- `Level` - int: Item level
- `Price` - float: Price in currency
- `PriceString` - string: Formatted price
- `Quantity` - int: Quantity available
- `MaxQuantity` - int: Max quantity
- `StatusCost` - int: Status point cost
- `IsForSale` - bool: Is for sale
- `LinkID` - int: Link ID
- `ToLink` - string: Chat link
- `IsItemInfoAvailable` - bool: Info available
- `ToItemInfo` - [iteminfo](#iteminfo): Detailed info

**Methods:**
- `Buy[quantity]` - Buys item
- `Sell[quantity]` - Sells to merchant
- `Examine` - Examines item
- `SetPrice[plat,gold,silver,copper]` - Sets price (vendor)
- `ListForSale` - Lists for sale
- `UnListForSale` - Unlists item

---

#### **consignment**

Broker/consignment item.

**Members:**
- `Name` - string: Item name
- `Level` - int: Item level
- `Price` - float: Price with commission
- `Value` - float: Item value
- `BasePrice` - float: Price without commission
- `BasePriceString` - string: Formatted base price
- `Quantity` - int: Quantity
- `LinkID` - int: Link ID
- `ToLink` - string: Chat link
- `IsListed` - bool: Is listed
- `Market` - string: Market name
- `SerialNumber` - int64: Serial number
- `IsItemInfoAvailable` - bool: Info available
- `ToItemInfo` - [iteminfo](#iteminfo): Detailed info

**Methods:**
- `Buy[quantity]` - Purchases item
- `Examine` - Examines item
- `SetPrice[price]` - Sets price
- `List` - Lists for sale
- `Unlist` - Unlists item
- `Remove[quantity]` - Removes from consignment

---

#### **vendingcontainer**

Player vendor container.

**Members:**
- `Name` - string: Container name
- `UsedCapacity` - int: Used capacity
- `TotalCapacity` - int: Total capacity
- `CurrentCoin` - float: Current coin collected
- `TotalCoin` - float: Total coin capacity
- `Market` - string: Market name
- `SerialNumber` - int64: Serial number
- `CommissionReduction` - int: Commission reduction %
- `NumItems` - int: Number of items
- `Consignment[index|name]` - [consignment](#consignment): Get item

**Methods:**
- `TakeCoin[amount]` - Takes coin
- `Remove` - Removes vendor
- `ChangeTo` - Changes to this vendor

---

### Mail DataTypes

#### **inboxmailmessage**

Mail message in inbox.

**Members:**
- `ID` - uint: Message ID
- `Author` - string: Sender name
- `Subject` - string: Message subject
- `Body` - string: Message body
- `Platinum` - int: Platinum attached
- `Gold` - int: Gold attached
- `Silver` - int: Silver attached
- `Copper` - int: Copper attached
- `Attachment` - [iteminfo](#iteminfo): Attached item

**Methods:**
- `Open` - Opens message
- `Delete` - Deletes message
- `ReceiveAttachment` - Receives attachments

---

#### **mailmessage**

Opened mail message.

**Members:**
- `ID` - uint: Message ID
- `Author` - string: Sender name
- `Subject` - string: Message subject
- `Body` - string: Message body
- `Platinum` - int: Platinum attached
- `Gold` - int: Gold attached
- `Silver` - int: Silver attached
- `Copper` - int: Copper attached
- `Attachment` - [iteminfo](#iteminfo): Attached item

**Methods:**
- `Delete` - Deletes message
- `ReceiveAttachment` - Receives attachments

---

### Crafting DataTypes

#### **recipe**

Crafting recipe.

**Members:**
- `Name` - string: Recipe name
- `Level` - int: Recipe level
- `ID` - uint: Recipe ID
- `Technique` - string: Technique required
- `Knowledge` - string: Knowledge required
- `RecipeBook` - string: Recipe book name
- `IsRecipeInfoAvailable` - bool: Info available
- `ToRecipeInfo` - [recipeinfo](#recipeinfo): Detailed info

**Methods:**
- `Create` - Creates item from recipe
- `Examine` - Examines recipe
- `Set[ID]` - Sets recipe by ID

**See Also:**
- [recipeinfo](#recipeinfo) - Detailed recipe information (accessed via `Recipe.ToRecipeInfo`)
- [char](#char) - Character recipes access via `Me.Recipe[index]`

---

#### **recipeinfo**

Detailed recipe information.

**Members:**
- `Description` - string: Recipe description
- `Device` - string: Crafting device required
- `NumClasses` - int: Number of classes
- `Class[index|name]` - [class](#class): Class by index/name
- `PrimaryComponentQuantityOnHand` - int: Primary component quantity
- `PrimaryComponent` - [primarycomponent](#primarycomponent): Primary component
- `BuildComponent1` - [buildcomponent](#buildcomponent): Build component 1
- `BuildComponent2` - [buildcomponent](#buildcomponent): Build component 2
- `BuildComponent3` - [buildcomponent](#buildcomponent): Build component 3
- `BuildComponent4` - [buildcomponent](#buildcomponent): Build component 4
- `Fuel` - [buildcomponent](#buildcomponent): Fuel component
- `Product` - string: Product name
- `Byproduct` - string: Byproduct name

**Methods:**
- `ExamineProduct` - Examines product

**See Also:**
- [recipe](#recipe) - Basic recipe object (use `Recipe.ToRecipeInfo` to get detailed info)
- [primarycomponent](#primarycomponent) - Primary component (accessed via `RecipeInfo.PrimaryComponent`)
- [buildcomponent](#buildcomponent) - Build components (accessed via `RecipeInfo.BuildComponent1-4`, `RecipeInfo.Fuel`)

---

#### **primarycomponent**

Recipe primary component.

**Members:**
- `Name` - string: Component name
- `Length` - int: Name length
- `Quantity` - uint: Quantity required
- `QuantityOnHand` - uint: Quantity available

---

#### **buildcomponent**

Recipe build component.

**Members:**
- `Name` - string: Component name
- `Quantity` - uint: Quantity required
- `QuantityOnHand` - uint: Quantity available

---

#### **crafting**

Crafting session information.

**Members:**
- `Message` / `M` - string: Crafting message
- `Result` / `R` - int: Crafting result
- `Quality` / `Q` - int: Quality level
- `Progress` / `P` - int: Progress level
- `ProgressMod` / `PM` - int: Progress modifier
- `Durability` / `D` - int: Durability level
- `DurabilityMod` / `DM` - int: Durability modifier
- `MainIconID` / `MII` - int: Main icon ID
- `BackdropIconID` / `BDII` - int: Backdrop icon ID
- `GV[variable,type]` - variable: Gets custom variable

**Methods:**
- `CA` - Clears all custom variables
- `GK[variable,index]` - Generates key for variable
- `SV[variable,key,value]` - Sets variable with key

---

### Window DataTypes

#### **eq2window**

Base window type. Many window datatypes inherit from this type.

**Members:**
- `Child[type,name]` - [eq2widget](#eq2widget): Gets child widget
- `IsVisible` - bool: Window is visible
- `RootPage` - [eq2uipage](#eq2uipage): Root page
- `BasePage` - [eq2uipage](#eq2uipage): Base page

**Methods:**
- `Close` - Closes window

**Inherited By:**
- [journalquestwindow](#journalquestwindow)
- [reforgewindow](#reforgewindow)
- [brokerwindow](#brokerwindow)
- [merchantwindow](#merchantwindow)
- [channelerwindow](#channelerwindow)
- [beastlordwindow](#beastlordwindow)
- [radialmenuwindow](#radialmenuwindow)
- [mailwindow](#mailwindow)
- [openedmailwindow](#openedmailwindow)
- [travelmapwindow](#travelmapwindow)

---

#### **eq2clonewindow**

Clone window type. Inherits from [**eq2window**](#eq2window).

**Inheritance:**
- All members and methods from [**eq2window**](#eq2window) are available

**Members:**
- `ID` - int: Window ID

**Inherited By:**
- [lootwindow](#lootwindow)
- [choicewindow](#choicewindow)
- [containerwindow](#containerwindow)
- [rewardwindow](#rewardwindow)
- [examineitemwindow](#examineitemwindow)
- [replydialog](#replydialog)
- [mapwindow](#mapwindow)

---

#### **journalquestwindow**

Quest journal window. Inherits from [**eq2window**](#eq2window).

**Inheritance:**
- All members and methods from [**eq2window**](#eq2window) are available

**Additional Members:**
- `NumActiveQuests` - int: Number of active quests
- `NumCompletedQuests` - int: Number of completed quests
- `ActiveQuest[index|name]` - [quest](#quest): Get active quest
- `CompletedQuest[index|name]` - [quest](#quest): Get completed quest
- `CurrentQuest` - [currentquest](#currentquest): Currently selected quest

**Methods:**
- `GetActiveQuests[index]` - Populates index with active quests
- `GetCompletedQuests[index]` - Populates index with completed quests

---

#### **lootwindow**

Loot window. Inherits from [**eq2clonewindow**](#eq2clonewindow).

**Inheritance:**
- All members and methods from [**eq2clonewindow**](#eq2clonewindow) and [**eq2window**](#eq2window) are available

**Additional Members:**
- `LootSourceID` - uint: Loot source ID
- `NumItems` - int: Number of items
- `Item[index|name]` - [iteminfo](#iteminfo): Get item
- `Type` - string: Loot method type
- `ItemsPage` - [eq2uipage](#eq2uipage): Items page
- `LootAllButton` - [eq2button](#eq2button): Loot all button
- `LootSelected` - [eq2button](#eq2button): Loot selected button
- `RequestSelected` - [eq2button](#eq2button): Request selected button
- `LottoDecline` - [eq2button](#eq2button): Lotto decline button
- `LeaderAssign` - [eq2button](#eq2button): Leader assign button
- `NBG_Need` - [eq2button](#eq2button): Need button
- `NBG_Greed` - [eq2button](#eq2button): Greed button
- `NBG_Decline` - [eq2button](#eq2button): NBG decline button
- `GroupMembers` - [eq2dropdownbox](#eq2dropdownbox): Group members dropdown

**Methods:**
- `LootItem[index,noconfirm]` - Loots item by index
- `LootAll` - Loots all items
- `RequestAll` - Requests all items
- `DeclineLotto` - Declines lotto
- `SelectNeed` - Selects need
- `SelectGreed` - Selects greed
- `DeclineNBG` - Declines NBG

---

#### **choicewindow**

Choice dialog window. Inherits from [**eq2clonewindow**](#eq2clonewindow).

**Inheritance:**
- All members and methods from [**eq2clonewindow**](#eq2clonewindow) and [**eq2window**](#eq2window) are available

**Additional Members:**
- `Text` - [eq2text](#eq2text): Dialog text
- `Choice1` - string: First choice text
- `Choice2` - string: Second choice text

**Methods:**
- `DoChoice1` - Selects first choice
- `DoChoice2` - Selects second choice

---

#### **containerwindow**

Container window. Inherits from [**eq2clonewindow**](#eq2clonewindow).

**Inheritance:**
- All members and methods from [**eq2clonewindow**](#eq2clonewindow) and [**eq2window**](#eq2window) are available

**Additional Members:**
- `NumItems` - int: Number of items
- `Item[index|name]` - [containerwindowitem](#containerwindowitem): Get item

**Methods:**
- `RemoveItem[id,quantity]` - Removes item

---

#### **containerwindowitem**

Item in container window.

**Members:**
- `ID` - int: Item ID
- `LinkID` - int: Link ID
- `ToLink` - string: Chat link
- `IconID` - int: Icon ID
- `Name` - string: Item name
- `Quantity` - int: Quantity
- `Level` - int: Item level
- `IsItemInfoAvailable` - bool: Info available
- `ToItemInfo` - [iteminfo](#iteminfo): Detailed info

---

#### **rewardwindow**

Reward pack window. Inherits from [**eq2clonewindow**](#eq2clonewindow).

**Inheritance:**
- All members and methods from [**eq2clonewindow**](#eq2clonewindow) and [**eq2window**](#eq2window) are available

**Additional Members:**
- `NumRewards` - int: Number of rewards
- `Reward[index]` / `Reward[LinkID,index]` - [reward](#reward): Get reward

**Methods:**
- `Cancel` - Cancels reward selection
- `AcceptReward[linkid]` - Accepts reward

---

#### **replydialog**

NPC conversation window. Inherits from [**eq2clonewindow**](#eq2clonewindow).

**Inheritance:**
- All members and methods from [**eq2clonewindow**](#eq2clonewindow) and [**eq2window**](#eq2window) are available

**Additional Members:**
- `Text` - [eq2text](#eq2text): Dialog text
- `Replies` - [eq2list](#eq2list): List of replies

**Methods:**
- `Choose[index]` / `Select[index]` - Selects reply by index

---

#### **mapwindow**

Map window. Inherits from [**eq2clonewindow**](#eq2clonewindow).

**Inheritance:**
- All members and methods from [**eq2clonewindow**](#eq2clonewindow) and [**eq2window**](#eq2window) are available

**Additional Members:**
- `CurrentZoneShortName` - string: Current zone short name
- `CurrentRealm` - string: Current realm
- `MapZoneShortName` - string: Map zone short name
- `MapRealm` - string: Map realm
- `ZoomLevel` - float: Zoom level
- `TeleportLocations` - int: Number of teleport locations
- `TeleportLocation[index]` - [teleportlocation](#teleportlocation): Get teleport location

**Methods:**
- `Teleport[index]` - Teleports to location

---

#### **travelmapwindow**

Travel map window (Fast Travel window). Inherits from [**eq2window**](#eq2window).

**Members:**
- `TeleportLocations` - int: Total number of teleport locations available
- `TeleportLocation[index]` - [travelmapwindowlocation](#travelmapwindowlocation): Get teleport location by index (1 to TeleportLocations)
- `Buttons` - [eq2uipage](#eq2uipage): Buttons page

**Notes:**
- This is the "Fast Travel" window that opens with the EQ2.OpenTravelMapWindow method
- TeleportLocation provides informational data about all fast travel locations
- Inherits all members and methods from eq2window

---

#### **teleportlocation**

Teleport location on the map window.

**Members:**
- `ID` - uint: Location ID
- `Description` - string: Location description
- `Location` - point3f: Teleport location coordinates

---

#### **travelmapwindowlocation**

Travel map location.

**Members:**
- `ID` - uint: Location ID
- `LowerLevel` - int: Minimum level for location
- `UpperLevel` - int: Maximum level for location
- `ZoneName` - string: Zone name
- `ZoneShortName` - string: Zone short name
- `ZoneDescription` - string: Zone description
- `Destination` - point3f: Destination coordinates
- `DestinationDescription` - string: Destination description
- `MapPositionX` - int: X position on map
- `MapPositionY` - int: Y position on map

---

#### **radialmenuwindow**

Radial menu window. Inherits from [**eq2window**](#eq2window).

**Inheritance:**
- All members and methods from [**eq2window**](#eq2window) are available

**Additional Members:**
- `NumActions` - int: Number of actions
- `Action[index]` - [radialmenuaction](#radialmenuaction): Get action
- `Actor` - [actor](#actor): Target actor

---

#### **radialmenuaction**

Radial menu action.

**Members:**
- `Label` - string: Action label
- `Command` - string: Command line
- `Verb` - string: Verb
- `UnavailableReason` - string: Why unavailable
- `Unavailable` - bool: Is unavailable
- `MaxRange` - float: Maximum range
- `Target` - [actor](#actor): Target actor

---

#### **brokerwindow**

Broker search window. Inherits from [**eq2window**](#eq2window).

**Inheritance:**
- All members and methods from [**eq2window**](#eq2window) are available

**Additional Members:**
- `NumItems` - int: Number of items in results

---

#### **merchantwindow**

Merchant window. Inherits from [**eq2window**](#eq2window).

**Inheritance:**
- All members and methods from [**eq2window**](#eq2window) are available

**Additional Members:**
- `NumItems` - int: Number of items
- `Item[index|name]` - [merchandise](#merchandise): Get merchandise

---

#### **mailwindow**

Mail inbox window. Inherits from [**eq2window**](#eq2window).

**Inheritance:**
- All members and methods from [**eq2window**](#eq2window) are available

**Additional Members:**
- `NumMessages` - int: Number of messages
- `Message[index]` - [inboxmailmessage](#inboxmailmessage): Get message

---

#### **openedmailwindow**

Opened mail message window. Inherits from [**eq2window**](#eq2window).

**Inheritance:**
- All members and methods from [**eq2window**](#eq2window) are available

**Additional Members:**
- `Message` - [mailmessage](#mailmessage): Current message

---

#### **reforgewindow**

Item reforge window. Inherits from [**eq2window**](#eq2window).

**Inheritance:**
- All members and methods from [**eq2window**](#eq2window) are available

**Additional Members:**
- `Item` - [iteminfo](#iteminfo): Item being reforged

---

#### **examineitemwindow**

Item examination window. Inherits from [**eq2clonewindow**](#eq2clonewindow).

**Inheritance:**
- All members and methods from [**eq2clonewindow**](#eq2clonewindow) and [**eq2window**](#eq2window) are available

**Additional Members:**
- `Item` - [iteminfo](#iteminfo): Item being examined

---

#### **channelerwindow**

Channeler class window. Inherits from [**eq2window**](#eq2window).

**Inheritance:**
- All members and methods from [**eq2window**](#eq2window) are available

**Additional Members:**
- `NumAbilities` - int: Number of abilities

---

#### **beastlordwindow**

Beastlord class window. Inherits from [**eq2window**](#eq2window).

**Inheritance:**
- All members and methods from [**eq2window**](#eq2window) are available

**Additional Members:**
- `NumAbilities` - int: Number of abilities

---

### Widget/UI DataTypes

#### **eq2baseobject**

Base class for all EQ2 UI objects. Many UI datatypes inherit from this type.

**Members:**
- `GetProperty[name,type]` - variable: Gets UI property
- `Type` - string: Object type name
- `Parent` - [eq2baseobject](#eq2baseobject): Parent object

**Methods:**
- `SetProperty[name,value]` - Sets UI property
- `GetProperties[map]` - Gets all properties in map
- `SpewProperties` - Outputs all properties to console

**Inherited By:**
- [eq2widget](#eq2widget)
- [eq2datasourcecontainer](#eq2datasourcecontainer)
- [eq2dynamicdata](#eq2dynamicdata)

---

#### **eq2datasourcecontainer**

UI data source container. Inherits from [**eq2baseobject**](#eq2baseobject).

**Inheritance:**
- All members and methods from [**eq2baseobject**](#eq2baseobject) are available

---

#### **eq2dynamicdata**

UI dynamic data. Inherits from [**eq2baseobject**](#eq2baseobject).

**Inheritance:**
- All members and methods from [**eq2baseobject**](#eq2baseobject) are available

---

#### **eq2widget**

Base UI widget. Inherits from [**eq2baseobject**](#eq2baseobject). Many UI widget types inherit from this type.

**Inheritance:**
- All members and methods from [**eq2baseobject**](#eq2baseobject) are available

**Members:**
- `IsEnabled` - bool: Widget is enabled

**Methods:**
- `LeftClick` - Simulates left click
- `RightClick` - Simulates right click
- `MiddleClick` - Simulates middle click
- `DoubleLeftClick` - Simulates double left click

**Inherited By:**
- [eq2button](#eq2button)
- [eq2text](#eq2text)
- [eq2icon](#eq2icon)
- [eq2textbox](#eq2textbox)
- [eq2scrollbar](#eq2scrollbar)
- [eq2sliderbar](#eq2sliderbar)
- [eq2dropdownbox](#eq2dropdownbox)
- [eq2list](#eq2list)
- [eq2listbox](#eq2listbox)
- [eq2progressbar](#eq2progressbar)
- [eq2checkbox](#eq2checkbox)
- [eq2uipage](#eq2uipage)
- [eq2iconbank](#eq2iconbank)

---

#### **eq2button**

Button widget. Inherits from [**eq2widget**](#eq2widget).

**Inheritance:**
- All members and methods from [**eq2widget**](#eq2widget) and [**eq2baseobject**](#eq2baseobject) are available

**Members:**
- `Text` - string: Button text

---

#### **eq2text**

Text display widget. Inherits from [**eq2widget**](#eq2widget).

**Inheritance:**
- All members and methods from [**eq2widget**](#eq2widget) and [**eq2baseobject**](#eq2baseobject) are available

---

#### **eq2icon**

Icon widget. Inherits from [**eq2widget**](#eq2widget).

**Inheritance:**
- All members and methods from [**eq2widget**](#eq2widget) and [**eq2baseobject**](#eq2baseobject) are available

**Additional Members:**
- `IconID` - int: Icon ID
- `NodeID` - uint: Node ID
- `ToAbility` - [ability](#ability): Converts to ability
- `IsReady` - bool: Ability is ready
- `PercentUndimmed` - float: Percent undimmed

---

#### **eq2textbox**

Text box widget. Inherits from [**eq2widget**](#eq2widget).

**Inheritance:**
- All members and methods from [**eq2widget**](#eq2widget) and [**eq2baseobject**](#eq2baseobject) are available

**Methods:**
- `AppendText[text]` - Appends text

---

#### **eq2scrollbar**

Scrollbar widget. Inherits from [**eq2widget**](#eq2widget).

**Inheritance:**
- All members and methods from [**eq2widget**](#eq2widget) and [**eq2baseobject**](#eq2baseobject) are available

**Additional Members:**
- `AttachedControl` - [eq2widget](#eq2widget): Attached control
- `ThumbPosition` - int: Thumb position
- `ThumbSize` - int: Thumb size
- `CanScrollUp` - bool: Can scroll up
- `CanScrollDown` - bool: Can scroll down

**Methods:**
- `ScrollUp` - Scrolls up
- `ScrollDown` - Scrolls down

---

#### **eq2sliderbar**

Slider bar widget. Inherits from [**eq2widget**](#eq2widget).

**Inheritance:**
- All members and methods from [**eq2widget**](#eq2widget) and [**eq2baseobject**](#eq2baseobject) are available

---

#### **eq2dropdownbox**

Dropdown box widget. Inherits from [**eq2widget**](#eq2widget).

**Inheritance:**
- All members and methods from [**eq2widget**](#eq2widget) and [**eq2baseobject**](#eq2baseobject) are available

**Additional Members:**
- `Label` - string: Selected label

**Methods:**
- `Set[index]` - Sets selection by index
- `GetOptions[index]` - Gets all options in index

---

#### **eq2list**

List widget. Inherits from [**eq2widget**](#eq2widget).

**Inheritance:**
- All members and methods from [**eq2widget**](#eq2widget) and [**eq2baseobject**](#eq2baseobject) are available

**Methods:**
- `HighlightRow[index]` - Highlights row
- `GetOptions[index]` - Gets all options in index

---

#### **eq2listbox**

List box widget. Inherits from [**eq2widget**](#eq2widget).

**Inheritance:**
- All members and methods from [**eq2widget**](#eq2widget) and [**eq2baseobject**](#eq2baseobject) are available

**Additional Members:**
- `Label` - string: Selected label

**Methods:**
- `GetOptions[index]` - Gets all options in index

---

#### **eq2progressbar**

Progress bar widget. Inherits from [**eq2widget**](#eq2widget).

**Inheritance:**
- All members and methods from [**eq2widget**](#eq2widget) and [**eq2baseobject**](#eq2baseobject) are available

---

#### **eq2checkbox**

Checkbox widget. Inherits from [**eq2widget**](#eq2widget).

**Inheritance:**
- All members and methods from [**eq2widget**](#eq2widget) and [**eq2baseobject**](#eq2baseobject) are available

---

#### **eq2uipage**

UI page container. Inherits from [**eq2widget**](#eq2widget).

**Inheritance:**
- All members and methods from [**eq2widget**](#eq2widget) and [**eq2baseobject**](#eq2baseobject) are available

**Additional Members:**
- `NumChildren` - int: Number of child widgets
- `ChildType[index]` - string: Child type by index
- `Child[index|name]` / `Child[instance,name]` - [eq2widget](#eq2widget): Get child

**Methods:**
- `SpewChildren` - Lists all children

---

#### **eq2composite**

Composite UI element. Inherits from [**eq2uipage**](#eq2uipage).

**Inheritance:**
- All members and methods from [**eq2uipage**](#eq2uipage), [**eq2widget**](#eq2widget), and [**eq2baseobject**](#eq2baseobject) are available

---

#### **eq2tabbedpane**

Tabbed pane widget. Inherits from [**eq2uipage**](#eq2uipage).

**Inheritance:**
- All members and methods from [**eq2uipage**](#eq2uipage), [**eq2widget**](#eq2widget), and [**eq2baseobject**](#eq2baseobject) are available

---

#### **eq2iconbank**

Icon bank (ability bars, etc.). Inherits from [**eq2widget**](#eq2widget).

**Inheritance:**
- All members and methods from [**eq2widget**](#eq2widget) and [**eq2baseobject**](#eq2baseobject) are available

**Additional Members:**
- `NumIcons` - int: Number of icons
- `Icon[index]` - [eq2icon](#eq2icon): Get icon by index

---

## Commands

ISXEQ2 provides several console commands that can be executed directly from the InnerSpace console or through LavishScript.

### Core Commands

#### **EQ2Execute**

Executes an EQ2 game command directly.

**Syntax:**
```
EQ2Execute <command>
```

**Examples:**
```
EQ2Execute /say Hello World
EQ2Execute /target Gnoll
EQ2Execute /useability Holy Strike
```

**Notes:**
- Allows execution of any in-game slash command
- Can be used to execute commands that are not directly accessible through ISXEQ2
- Works at login screen and in-game

---

#### **Face**

Turns your character to face a target, direction, or coordinates.

**Syntax:**
```
Face
Face <name>
Face <ActorType>
Face <heading>
Face <X> <Z>
Face <X> <Y> <Z>
Face nocamera
```

**Parameters:**
- No parameters - Faces current target
- `<name>` - Partial or full actor name
- `<ActorType>` - PC, NPC, NamedNPC, AggroNPC, Pet, MyPet, Chest, Door, Resource, etc.
- `<heading>` - Heading value (0-360)
- `<X> <Z>` - 2D coordinates (Y ignored)
- `<X> <Y> <Z>` - 3D coordinates (Y still ignored)
- `nocamera` - Toggle whether to adjust camera in 3rd person view

**Examples:**
```
Face                          ; Face current target
Face gnoll                    ; Face nearest actor matching "gnoll"
Face NPC                      ; Face nearest NPC
Face 180                      ; Face heading 180
Face 100.5 50.2 -25.8         ; Face specific coordinates
Face nocamera                 ; Toggle camera adjustment
```

**Notes:**
- Sorted by distance when using name/type searches
- Camera adjustment can be toggled for 3rd person view
- Heading 0 = North, 90 = East, 180 = South, 270 = West

---

#### **Target**

Targets an actor by type, name, or ID.

**Syntax:**
```
Target <type>
Target <ID#>
Target <name>
```

**Parameters:**
- `<type>` - PC, NPC, NamedNPC, AggroNPC, Pet, MyPet, Me, Chest, Door, Resource, etc.
- `<ID#>` - Actor ID number
- `<name>` - Partial or full actor name

**Examples:**
```
Target NPC                    ; Target nearest NPC
Target 12345                  ; Target actor with ID 12345
Target gnoll scout            ; Target nearest "gnoll scout"
Target Me                     ; Target yourself
```

**Notes:**
- Searches are sorted by distance (closest first)
- Partial name matching is case-insensitive
- Falls back to EQ2 native targeting if actor not found

---

#### **Activate**

Activates an equipped item in a specific equipment slot.

**Syntax:**
```
Activate <equip slot>
```

**Parameters:**
- `<equip slot>` - Equipment slot name (primary, secondary, chest, head, etc.)

**Examples:**
```
Activate primary
Activate chest
Activate ranged
```

**Notes:**
- Only works on items with activate/use effects
- See the [ISXEQ2 Wiki](http://forge.isxgames.com/projects/isxeq2/wiki/Activate_(command)) for complete slot list

---

#### **Radar**

Displays a list of actors within radar range.

**Syntax:**
```
Radar
```

**Notes:**
- Shows all actors currently on your radar
- Useful for debugging and actor enumeration

---

#### **Where**

Displays information about actors matching search criteria.

**Syntax:**
```
Where
Where <ActorType>
Where <ActorType> <level>
Where <ActorType> <lowlevel> <highlevel>
Where byLevel
Where byType
Where byName
Where byDist
```

**Parameters:**
- No parameters - Lists all actors
- `<ActorType>` - PC, NPC, NamedNPC, AggroNPC, Pet, MyPet, Chest, Door, Resource, etc.
- `<level>` - Specific level
- `<lowlevel> <highlevel>` - Level range
- `byLevel`, `byType`, `byName`, `byDist` - Sorting options

**Examples:**
```
Where                         ; List all actors
Where NPC                     ; List all NPCs
Where NPC 50                  ; List level 50 NPCs
Where NPC 45 55               ; List NPCs level 45-55
Where byDist                  ; List all actors sorted by distance
```

**Notes:**
- Provides detailed information including location, heading, distance
- Results can be sorted by various criteria
- Useful for finding specific mobs or resources

---

### Utility Commands

#### **eq2echo**

Outputs text to console, chat, or file with formatting options.

**Syntax:**
```
eq2echo [options] <text>
```

**Options:**
- `-chattype <type>` - Send to specific chat channel
- `> <location>` - Pipe output to console
- `>& <location>` - Pipe output to debugger

**Examples:**
```
eq2echo Hello World
eq2echo -chattype say Hello everyone
eq2echo Current health: ${Me.Health}
```

**Notes:**
- Supports variable expansion
- Can output to various destinations
- Handles EQ2 chat link formatting

---

#### **eq2loc**

Manages saved locations for waypoints and navigation.

**Syntax:**
```
eq2loc Add <label> <X> <Y> <Z> <zone> <roomID> [notes]
eq2loc List [filter]
eq2loc Delete <label>
eq2loc ListAll [filter]
```

**Parameters:**
- `<label>` - Unique name for the location
- `<X> <Y> <Z>` - Coordinates
- `<zone>` - Zone name
- `<roomID>` - Room/instance ID
- `[notes]` - Optional description
- `[filter]` - Filter results by label or notes

**Examples:**
```
eq2loc Add Bank -123.45 67.89 -45.67 "Freeport" 0 "Main bank location"
eq2loc List
eq2loc List bank
eq2loc Delete Bank
eq2loc ListAll
```

**Notes:**
- Locations are saved to XML file
- Supports filtering when listing
- Locations are zone-specific

---

#### **announce**

Displays an on-screen announcement message.  (This command is currently non-functional.  Currently, it does the same thing as eq2echo.)

**Syntax:**
```
announce <text> [timer] [sound]
```

**Parameters:**
- `<text>` - Message to display
- `[timer]` - Display duration in seconds (0.0-10.0, default 4.5)
- `[sound]` - Sound effect type (0-max announcement types)

**Examples:**
```
announce "Quest Complete!"
announce "Warning!" 5.0
announce "Level Up!" 3.0 1
```

**Notes:**
- Displays as on-screen notification similar to quest completion
- Text color defaults to white
- Sound types correspond to different announcement categories

---

#### **eq2ignore**

Manages chat message filtering.

**Syntax:**
```
eq2ignore ALL
eq2ignore List
eq2ignore Add <phrase> [chat type]
eq2ignore Delete <phrase>
```

**Parameters:**
- `ALL` - Toggle ignoring all chat messages
- `<phrase>` - Text phrase to ignore
- `[chat type]` - Specific chat channel ID (optional)

**Examples:**
```
eq2ignore ALL                 ; Toggle ignore all
eq2ignore List                ; List all ignored phrases
eq2ignore Add "spam message"  ; Ignore phrase in all channels
eq2ignore Add "trade" 5       ; Ignore phrase in chat type 5
eq2ignore Delete "spam message"
```

**Notes:**
- Filters chat messages before display
- Can ignore specific phrases or all chat
- Settings saved to XML file

---

#### **broker**

Performs broker/marketplace searches.

**Syntax:**
```
broker [parameters]
broker Cancel
```

**Parameters:**
- `Name <item name>` - Search by item name
- `Sort <method>` - Sort results
- `MinPrice <amount>` - Minimum price
- `MaxPrice <amount>` - Maximum price
- `MinLevel <level>` - Minimum level
- `MaxLevel <level>` - Maximum level
- `MinTier <tier>` - Minimum tier
- `MaxTier <tier>` - Maximum tier
- `Tier <tier>` - Specific tier
- `MinSkill <skill>` - Minimum skill
- `MaxSkill <skill>` - Maximum skill
- `Seller <name>` - Seller name
- `Effect <effect>` - Item effect/adornment
- `Class <class>` - Class restriction
- `SimpleSearch` - Use simple search mode
- `Cancel` - Cancel current broker search

**Examples:**
```
broker Name "rough opal"
broker Name sword MinLevel 50 MaxLevel 60
broker Tier Legendary Class Guardian
broker MinPrice 1000 MaxPrice 50000
broker Cancel
```

**Notes:**
- Supports complex search criteria
- Simple search mode when using Name parameter only
- Results displayed in broker window

---

#### **InitCommands**

Manages commands that run automatically when ISXEQ2 initializes.

**Syntax:**
```
InitCommands List
InitCommands Add <command>
InitCommands Delete <number>
```

**Parameters:**
- `List` - Display all initialization commands
- `Add <command>` - Add a new init command
- `Delete <number>` - Remove init command by number

**Examples:**
```
InitCommands List
InitCommands Add "echo ISXEQ2 Loaded"
InitCommands Delete 1
```

**Notes:**
- Commands execute when extension loads
- Useful for auto-configuration
- Saved to XML file

---

### Web Commands

#### **GetURL**

Fetches content from a URL using HTTP GET method.

**Syntax:**
```
GetURL <URL> [content_type]
```

**Parameters:**
- `<URL>` - HTTP or HTTPS URL to fetch
- `[content_type]` - Optional content type (e.g., "application/json", "text/plain")

**Examples:**
```
GetURL http://example.com/data.txt
GetURL https://example.com/api/data "application/json"
```

**Notes:**
- Retrieves web content asynchronously (does not block script execution)
- Response is accessible via the `isxGames_onHTTPResponse` event
- Can be used for external data integration
- Supports both HTTP and HTTPS

---

#### **PostURL**

Posts data to a URL using HTTP POST method.

**Syntax:**
```
PostURL <URL> <post_data> [content_type]
```

**Parameters:**
- `<URL>` - HTTP or HTTPS URL to post to
- `<post_data>` - Data to send in POST request
- `[content_type]` - Optional content type (default: "application/x-www-form-urlencoded")

**Examples:**
```
PostURL http://example.com/api "key=value&data=test"
PostURL https://discord.com/api/webhooks/ID/TOKEN "{\"content\":\"This is a test\"}" "application/json"
PostURL https://example.com/submit "name=John&age=30" "application/x-www-form-urlencoded"
```

**Notes:**
- Sends HTTP POST request asynchronously (does not block script execution)
- Response is accessible via the `isxGames_onHTTPResponse` event
- Useful for web API integration, Discord webhooks, and external logging
- Supports both HTTP and HTTPS
- For JSON data, ensure proper escaping of quotes

**Event Example:**
```lavishscript
; Event handler for HTTP responses
function isxGames_onHTTPResponse(string ResponseCode, string ResponseBody)
{
    echo "HTTP Response Code: ${ResponseCode}"
    echo "Response Body: ${ResponseBody}"
}

; Make a POST request
PostURL "https://example.com/api" "{\"message\":\"Hello World\"}" "application/json"
```

---

## Events

ISXEQ2 provides a comprehensive event system that allows scripts to react to various in-game occurrences. Events can be registered using the `Event` command in LavishScript and trigger custom atom callbacks when specific game actions occur.

### Event Registration

Events are registered using the standard LavishScript event syntax:

```lavishscript
Event[EventName]:AttachAtom[AtomName]
```

To unregister an event:

```lavishscript
Event[EventName]:DetachAtom[AtomName]
```

### Actor Events

Events related to actor (character/NPC) lifecycle and state changes.

#### **EQ2_ActorSpawned**

Fired when an actor spawns in the game world.

**Arguments:**
- `ID` (string): Actor ID
- `Name` (string): Actor name
- `Level` (string): Actor level
- `Type` (string): Actor type

**Usage:**
```lavishscript
Event[EQ2_ActorSpawned]:AttachAtom[OnActorSpawned]

atom OnActorSpawned(string ID, string Name, string Level, string Type)
{
    echo "Actor spawned: ${Name} (ID: ${ID}, Level: ${Level}, Type: ${Type})"
}
```

---

#### **EQ2_ActorDespawned**

Fired when an actor despawns from the game world.

**Arguments:**
- `ID` (string): Actor ID
- `Name` (string): Actor name

**Usage:**
```lavishscript
Event[EQ2_ActorDespawned]:AttachAtom[OnActorDespawned]

atom OnActorDespawned(string ID, string Name)
{
    echo "Actor despawned: ${Name} (ID: ${ID})"
}
```

---

#### **EQ2_ActorTargetChange**

Fired when an actor changes their target.

**Arguments:**
- `ActorID` (string): ID of the actor changing target
- `ActorName` (string): Name of the actor
- `ActorType` (string): Type of the actor
- `OldTargetID` (string): Previous target ID
- `NewTargetID` (string): New target ID
- `Distance` (string): Distance to the actor
- `IsInGroup` (string): Whether actor is in your group
- `IsInRaid` (string): Whether actor is in your raid

**Usage:**
```lavishscript
Event[EQ2_ActorTargetChange]:AttachAtom[OnTargetChange]

atom OnTargetChange(string ActorID, string ActorName, string ActorType, string OldTargetID, string NewTargetID, string Distance, string IsInGroup, string IsInRaid)
{
    echo "${ActorName} changed target from ${OldTargetID} to ${NewTargetID}"
}
```

---

#### **EQ2_ActorAnimationChanged**

Fired when an actor's animation state changes.

**Arguments:**
- `ActorID` (string): ID of the actor
- `ActorName` (string): Name of the actor
- `ActorType` (string): Type of the actor
- `OldAnimation` (string): Previous animation state
- `NewAnimation` (string): New animation state
- `Distance` (string): Distance to the actor
- `IsInGroup` (string): Whether actor is in your group
- `IsInRaid` (string): Whether actor is in your raid

**Usage:**
```lavishscript
Event[EQ2_ActorAnimationChanged]:AttachAtom[OnAnimationChanged]

atom OnAnimationChanged(string ActorID, string ActorName, string ActorType, string OldAnimation, string NewAnimation, string Distance, string IsInGroup, string IsInRaid)
{
    echo "${ActorName} animation changed from ${OldAnimation} to ${NewAnimation}"
}
```

---

#### **EQ2_ActorStanceChange**

Fired when an actor changes their stance (combat, peace, etc.).

**Arguments:**
- `ActorID` (string): ID of the actor
- `ActorName` (string): Name of the actor
- `ActorType` (string): Type of the actor
- `OldStance` (string): Previous stance
- `NewStance` (string): New stance
- `TargetID` (string): Actor's target ID
- `Distance` (string): Distance to the actor
- `IsInGroup` (string): Whether actor is in your group
- `IsInRaid` (string): Whether actor is in your raid

**Usage:**
```lavishscript
Event[EQ2_ActorStanceChange]:AttachAtom[OnStanceChange]

atom OnStanceChange(string ActorID, string ActorName, string ActorType, string OldStance, string NewStance, string TargetID, string Distance, string IsInGroup, string IsInRaid)
{
    echo "${ActorName} stance changed from ${OldStance} to ${NewStance}"
}
```

---

#### **EQ2_ActorHealthChange**

Fired when an actor's health changes.

**Arguments:**
- `ActorID` (string): ID of the actor
- `ActorName` (string): Name of the actor
- `ActorType` (string): Type of the actor
- `OldHealth` (string): Previous health value
- `NewHealth` (string): New health value
- `Distance` (string): Distance to the actor
- `IsInGroup` (string): Whether actor is in your group
- `IsInRaid` (string): Whether actor is in your raid

**Usage:**
```lavishscript
Event[EQ2_ActorHealthChange]:AttachAtom[OnHealthChange]

atom OnHealthChange(string ActorID, string ActorName, string ActorType, string OldHealth, string NewHealth, string Distance, string IsInGroup, string IsInRaid)
{
    echo "${ActorName} health changed from ${OldHealth} to ${NewHealth}"
}
```

---

#### **EQ2_ActorPowerChange**

Fired when an actor's power changes.

**Arguments:**
- `ActorID` (string): ID of the actor
- `ActorName` (string): Name of the actor
- `ActorType` (string): Type of the actor
- `OldPower` (string): Previous power value
- `NewPower` (string): New power value
- `Distance` (string): Distance to the actor
- `IsInGroup` (string): Whether actor is in your group
- `IsInRaid` (string): Whether actor is in your raid

**Usage:**
```lavishscript
Event[EQ2_ActorPowerChange]:AttachAtom[OnPowerChange]

atom OnPowerChange(string ActorID, string ActorName, string ActorType, string OldPower, string NewPower, string Distance, string IsInGroup, string IsInRaid)
{
    echo "${ActorName} power changed from ${OldPower} to ${NewPower}"
}
```

---

### Casting Events

Events related to spell and ability casting.

#### **EQ2_CastingStarted**

Fired when casting begins.

**Arguments:** None

**Usage:**
```lavishscript
Event[EQ2_CastingStarted]:AttachAtom[OnCastingStarted]

atom OnCastingStarted()
{
    ; Casting started
}
```

**Notes:**
- This event is currently disabled in the code as it was causing client lockups

---

#### **EQ2_CastingEnded**

Fired when casting ends.

**Arguments:** None

**Usage:**
```lavishscript
Event[EQ2_CastingEnded]:AttachAtom[OnCastingEnded]

atom OnCastingEnded()
{
    ; Casting ended
}
```

**Notes:**
- This event is currently disabled in the code as it was causing client lockups

---

### Character Events

Events related to character progression and state changes.

#### **EQ2_onLevelChange**

Fired when the character's level changes.

**Arguments:**
- `OldLevel` (int): Previous level
- `NewLevel` (int): New level

**Usage:**
```lavishscript
Event[EQ2_onLevelChange]:AttachAtom[OnLevelChange]

atom OnLevelChange(int OldLevel, int NewLevel)
{
    echo "Congratulations! You advanced from level ${OldLevel} to ${NewLevel}"
}
```

---

#### **EQ2_onAbilityGained**

Fired when the character gains a new ability.

**Arguments:**
- `Name` (string): Name of the ability
- `Tier` (int): Tier of the ability

**Usage:**
```lavishscript
Event[EQ2_onAbilityGained]:AttachAtom[OnAbilityGained]

atom OnAbilityGained(string Name, int Tier)
{
    echo "New ability gained: ${Name} (Tier ${Tier})"
}
```

---

#### **EQ2_onCharacterSheetUpdate**

Fired when the character sheet data is updated.

**Arguments:** None

**Usage:**
```lavishscript
Event[EQ2_onCharacterSheetUpdate]:AttachAtom[OnCharacterSheetUpdate]

atom OnCharacterSheetUpdate()
{
    ; Character sheet updated - refresh stats
}
```

---

#### **EQ2_onMeAfflicted**

Fired when the player character is afflicted with a detrimental effect.

**Arguments:**
- `TraumaCounter` (int): Number of trauma detriments
- `ArcaneCounter` (int): Number of arcane detriments
- `NoxiousCounter` (int): Number of noxious detriments
- `ElementalCounter` (int): Number of elemental detriments
- `CursedCounter` (int): Number of cursed detriments

**Usage:**
```lavishscript
Event[EQ2_onMeAfflicted]:AttachAtom[OnMeAfflicted]

atom OnMeAfflicted(int TraumaCounter, int ArcaneCounter, int NoxiousCounter, int ElementalCounter, int CursedCounter)
{
    echo "Afflicted - Trauma:${TraumaCounter} Arcane:${ArcaneCounter} Noxious:${NoxiousCounter} Elemental:${ElementalCounter} Cursed:${CursedCounter}"
}
```

---

### Group and Raid Events

Events related to group and raid membership and states.

#### **EQ2_onGroupMembershipChange**

Fired when group membership changes (member joins or leaves).

**Arguments:**
- `PreviousGroupCount` (int): Previous number of group members
- `NewGroupCount` (int): New number of group members

**Usage:**
```lavishscript
Event[EQ2_onGroupMembershipChange]:AttachAtom[OnGroupChange]

atom OnGroupChange(int PreviousGroupCount, int NewGroupCount)
{
    echo "Group changed from ${PreviousGroupCount} to ${NewGroupCount} members"
}
```

---

#### **EQ2_onGroupMemberAfflicted**

Fired when a group member is afflicted with a detrimental effect.

**Arguments:**
- `ActorID` (int): ID of the afflicted group member
- `TraumaCounter` (int): Number of trauma detriments
- `ArcaneCounter` (int): Number of arcane detriments
- `NoxiousCounter` (int): Number of noxious detriments
- `ElementalCounter` (int): Number of elemental detriments
- `CursedCounter` (int): Number of cursed detriments

**Usage:**
```lavishscript
Event[EQ2_onGroupMemberAfflicted]:AttachAtom[OnGroupMemberAfflicted]

atom OnGroupMemberAfflicted(int ActorID, int TraumaCounter, int ArcaneCounter, int NoxiousCounter, int ElementalCounter, int CursedCounter)
{
    echo "Group member ${ActorID} afflicted with detriments"
}
```

---

#### **EQ2_onRaidMembershipChange**

Fired when raid membership changes (member joins or leaves).

**Arguments:**
- `PreviousRaidCount` (int): Previous number of raid members
- `NewRaidCount` (int): New number of raid members

**Usage:**
```lavishscript
Event[EQ2_onRaidMembershipChange]:AttachAtom[OnRaidChange]

atom OnRaidChange(int PreviousRaidCount, int NewRaidCount)
{
    echo "Raid changed from ${PreviousRaidCount} to ${NewRaidCount} members"
}
```

---

#### **EQ2_onRaidMemberAfflicted**

Fired when a raid member is afflicted with a detrimental effect.

**Arguments:**
- `ActorID` (int): ID of the afflicted raid member
- `TraumaCounter` (int): Number of trauma detriments
- `ArcaneCounter` (int): Number of arcane detriments
- `NoxiousCounter` (int): Number of noxious detriments
- `ElementalCounter` (int): Number of elemental detriments
- `CursedCounter` (int): Number of cursed detriments

**Usage:**
```lavishscript
Event[EQ2_onRaidMemberAfflicted]:AttachAtom[OnRaidMemberAfflicted]

atom OnRaidMemberAfflicted(int ActorID, int TraumaCounter, int ArcaneCounter, int NoxiousCounter, int ElementalCounter, int CursedCounter)
{
    echo "Raid member ${ActorID} afflicted with detriments"
}
```

---

### Quest Events

Events related to quests and achievements.

#### **EQ2_onQuestOffered**

Fired when a quest is offered to the player.

**Arguments:**
- `Name` (string): Quest name
- `Description` (string): Quest description
- `Level` (int): Quest level
- `StatusReward` (int): Status reward amount

**Usage:**
```lavishscript
Event[EQ2_onQuestOffered]:AttachAtom[OnQuestOffered]

atom OnQuestOffered(string Name, string Description, int Level, int StatusReward)
{
    echo "Quest offered: ${Name} (Level ${Level})"
}
```

---

#### **EQ2_onQuestUpdate**

Fired when quest progress is updated.

**Arguments:**
- `ID` (string): Quest ID
- `Name` (string): Quest name
- `CurrentZone` (string): Current zone
- `Category` (string): Quest category
- `Description` (string): Quest description
- `ProgressText...` (string): Variable number of progress text strings

**Usage:**
```lavishscript
Event[EQ2_onQuestUpdate]:AttachAtom[OnQuestUpdate]

atom OnQuestUpdate(string ID, string Name, string CurrentZone, string Category, string Description, ...)
{
    echo "Quest updated: ${Name}"
}
```

**Notes:**
- This event has a variable number of arguments - ProgressText strings are appended after the first 5 fixed arguments

---

#### **EQ2_onDeleteQuest**

Fired when a quest is deleted/abandoned.

**Arguments:**
- `Param1` (string): First parameter (quest-related)
- `Param2` (string): Second parameter (quest-related)

**Usage:**
```lavishscript
Event[EQ2_onDeleteQuest]:AttachAtom[OnDeleteQuest]

atom OnDeleteQuest(string Param1, string Param2)
{
    echo "Quest deleted"
}
```

---

#### **EQ2_ExamineAchievement**

Fired when an achievement is examined.

**Arguments:**
- `Type` (string): Achievement type
- `ID` (string): Achievement ID

**Usage:**
```lavishscript
Event[EQ2_ExamineAchievement]:AttachAtom[OnExamineAchievement]

atom OnExamineAchievement(string Type, string ID)
{
    echo "Examining achievement: Type=${Type}, ID=${ID}"
}
```

---

### Inventory and Item Events

Events related to inventory management and item interactions.

#### **EQ2_onInventoryUpdate**

Fired when inventory is updated (items added, removed, or changed).

**Arguments:** None

**Usage:**
```lavishscript
Event[EQ2_onInventoryUpdate]:AttachAtom[OnInventoryUpdate]

atom OnInventoryUpdate()
{
    ; Inventory updated - refresh inventory data
}
```

---

#### **EQ2_onLootWindowAppeared**

Fired when a loot window appears.

**Arguments:**
- `ID` (uint): Loot window ID

**Usage:**
```lavishscript
Event[EQ2_onLootWindowAppeared]:AttachAtom[OnLootWindowAppeared]

atom OnLootWindowAppeared(uint ID)
{
    echo "Loot window ${ID} appeared"
    ; Auto-loot logic here
}
```

---

#### **EQ2_onDestroyItem**

Fired when an item is destroyed.

**Arguments:**
- `ItemID` (int): Item ID
- `ItemName` (string): Item name

**Usage:**
```lavishscript
Event[EQ2_onDestroyItem]:AttachAtom[OnDestroyItem]

atom OnDestroyItem(int ItemID, string ItemName)
{
    echo "Item destroyed: ${ItemName} (ID: ${ItemID})"
}
```

---

#### **EQ2_onSellItem**

Fired when an item is sold to a merchant.

**Arguments:**
- `ItemName` (string): Name of the item sold
- `Quantity` (int): Quantity sold
- `LinkID` (int): Item link ID
- `ItemLinkString` (string): Item link string

**Usage:**
```lavishscript
Event[EQ2_onSellItem]:AttachAtom[OnSellItem]

atom OnSellItem(string ItemName, int Quantity, int LinkID, string ItemLinkString)
{
    echo "Sold ${Quantity}x ${ItemName}"
}
```

---

#### **EQ2_ExamineItemWindowAppeared**

Fired when an item examine window appears.

**Arguments:**
- `ItemName` (string): Name of the item
- `WindowID` (uint): Window ID

**Usage:**
```lavishscript
Event[EQ2_ExamineItemWindowAppeared]:AttachAtom[OnExamineItem]

atom OnExamineItem(string ItemName, uint WindowID)
{
    echo "Examining item: ${ItemName} (Window ID: ${WindowID})"
}
```

---

#### **EQ2_ItemAddedToAltarForSacrifice**

Fired when an item is added to an altar for sacrifice.

**Arguments:**
- `ItemIndex` (int): Index of the item in inventory

**Usage:**
```lavishscript
Event[EQ2_ItemAddedToAltarForSacrifice]:AttachAtom[OnItemSacrifice]

atom OnItemSacrifice(int ItemIndex)
{
    echo "Item at index ${ItemIndex} added to altar"
}
```

---

#### **EQ2_onContainerWindowAppeared**

Fired when a container window (chest, crate, etc.) appears.

**Arguments:**
- `ID` (uint): Container window ID

**Usage:**
```lavishscript
Event[EQ2_onContainerWindowAppeared]:AttachAtom[OnContainerAppeared]

atom OnContainerAppeared(uint ID)
{
    echo "Container window ${ID} appeared"
}
```

---

### Crafting Events

Events related to crafting activities.

#### **EQ2_onCraftRoundResult**

Fired when a crafting round completes with results.

**Arguments:**
- `Message` (string): Result message
- `Result` (int): Result code
- `Quality` (int): Quality value
- `Progress` (int): Progress value
- `ProgressMod` (int): Progress modifier
- `Durability` (int): Durability value
- `DurabilityMod` (int): Durability modifier
- `MainIconID` (int): Main icon ID
- `BackdropIconID` (int): Backdrop icon ID

**Usage:**
```lavishscript
Event[EQ2_onCraftRoundResult]:AttachAtom[OnCraftRoundResult]

atom OnCraftRoundResult(string Message, int Result, int Quality, int Progress, int ProgressMod, int Durability, int DurabilityMod, int MainIconID, int BackdropIconID)
{
    echo "Craft round: ${Message} - Quality:${Quality} Progress:${Progress} Durability:${Durability}"
}
```

---

### Window and UI Events

Events related to UI windows and dialogs.

#### **EQ2_ReplyDialogAppeared**

Fired when a reply dialog appears (conversation with NPC).

**Arguments:**
- `ID` (uint): Reply dialog ID

**Usage:**
```lavishscript
Event[EQ2_ReplyDialogAppeared]:AttachAtom[OnReplyDialog]

atom OnReplyDialog(uint ID)
{
    echo "Reply dialog ${ID} appeared"
}
```

---

#### **EQ2_onChoiceWindowAppeared**

Fired when a choice window appears.

**Arguments:**
- `ID` (uint): Choice window ID

**Usage:**
```lavishscript
Event[EQ2_onChoiceWindowAppeared]:AttachAtom[OnChoiceWindow]

atom OnChoiceWindow(uint ID)
{
    echo "Choice window ${ID} appeared"
}
```

---

#### **EQ2_onRewardWindowAppeared**

Fired when a reward selection window appears (quest rewards, etc.).

**Arguments:**
- `ID` (uint): Reward window ID

**Usage:**
```lavishscript
Event[EQ2_onRewardWindowAppeared]:AttachAtom[OnRewardWindow]

atom OnRewardWindow(uint ID)
{
    echo "Reward window ${ID} appeared"
}
```

---

#### **EQ2_OnHOWindowStateChange**

Fired when the Heroic Opportunity window state changes.

**Arguments:**
- `HOName` (string): HO name
- `HODescription` (string): HO description
- `HOWindowState` (string): Window state
- `HOTimeLimit` (string): Time limit
- `HOTimeElapsed` (string): Time elapsed
- `HOTimeRemaining` (string): Time remaining
- `HOCurrentWheelSlot` (string): Current wheel slot
- `HOWheelState` (string): Wheel state
- `HOIconID1` (string): Icon ID for slot 1
- `HOIconID2` (string): Icon ID for slot 2
- `HOIconID3` (string): Icon ID for slot 3
- `HOIconID4` (string): Icon ID for slot 4
- `HOIconID5` (string): Icon ID for slot 5
- `HOIconID6` (string): Icon ID for slot 6

**Usage:**
```lavishscript
Event[EQ2_OnHOWindowStateChange]:AttachAtom[OnHOStateChange]

atom OnHOStateChange(string HOName, string HODescription, string HOWindowState, string HOTimeLimit, string HOTimeElapsed, string HOTimeRemaining, string HOCurrentWheelSlot, string HOWheelState, string HOIconID1, string HOIconID2, string HOIconID3, string HOIconID4, string HOIconID5, string HOIconID6)
{
    echo "HO: ${HOName} - State:${HOWindowState} Time Remaining:${HOTimeRemaining}"
}
```

---

### Communication Events

Events related to chat and text messages.

#### **EQ2_onIncomingChatText**

Fired when chat text is received.

**Arguments:**
- `ChatType` (int): Type of chat
- `Message` (string): Chat message
- `Speaker` (string): Name of speaker
- `Target` (string): Target of the message
- `SpeakerIsNPC` (string): Whether speaker is an NPC
- `ChannelName` (string): Channel name
- `SpeakerID` (int): Speaker actor ID
- `TargetID` (int): Target actor ID
- `UnkString1` (string): Unknown parameter

**Usage:**
```lavishscript
Event[EQ2_onIncomingChatText]:AttachAtom[OnChatText]

atom OnChatText(int ChatType, string Message, string Speaker, string Target, string SpeakerIsNPC, string ChannelName, int SpeakerID, int TargetID, string UnkString1)
{
    echo "[${ChannelName}] ${Speaker}: ${Message}"
}
```

---

#### **EQ2_onIncomingText**

Fired when any text message is received (broader than chat).

**Arguments:**
- `Text` (string): The incoming text
- `Type` (int): Text type

**Usage:**
```lavishscript
Event[EQ2_onIncomingText]:AttachAtom[OnIncomingText]

atom OnIncomingText(string Text, int Type)
{
    echo "Incoming text (Type ${Type}): ${Text}"
}
```

**Notes:**
- This event fires for ALL incoming text, including non-chat messages like "Too far away"
- Different from EQ2_onIncomingChatText which only handles chat messages

---

#### **EQ2_onTellIgnored**

Fired when a tell is ignored (from ignored player).

**Arguments:**
- `Message` (string): The tell message
- `Speaker` (string): Name of the ignored speaker

**Usage:**
```lavishscript
Event[EQ2_onTellIgnored]:AttachAtom[OnTellIgnored]

atom OnTellIgnored(string Message, string Speaker)
{
    echo "Ignored tell from ${Speaker}: ${Message}"
}
```

**Notes:**
- This event implementation is currently commented out in the source code

---

#### **EQ2_onAnnouncement**

Fired when a server announcement is made.

**Arguments:**
- `Text` (string): Announcement text
- `SoundType` (string): Sound type to play
- `Timer` (float): Display timer

**Usage:**
```lavishscript
Event[EQ2_onAnnouncement]:AttachAtom[OnAnnouncement]

atom OnAnnouncement(string Text, string SoundType, float Timer)
{
    echo "Announcement (${Timer}s): ${Text}"
}
```

---

### Zone Events

Events related to zoning and world changes.

#### **EQ2_StartedZoning**

Fired when the zoning process begins.

**Arguments:** None

**Usage:**
```lavishscript
Event[EQ2_StartedZoning]:AttachAtom[OnStartedZoning]

atom OnStartedZoning()
{
    echo "Zoning started..."
}
```

---

#### **EQ2_FinishedZoning**

Fired when the zoning process completes and the character is fully loaded in the new zone.

**Arguments:**
- `TimeInSeconds` (string): Time taken to zone in seconds

**Usage:**
```lavishscript
Event[EQ2_FinishedZoning]:AttachAtom[OnFinishedZoning]

atom OnFinishedZoning(string TimeInSeconds)
{
    echo "Arrived in ${Zone.Name} (zoning took ${TimeInSeconds}s)"
}
```

---

### Miscellaneous Events

Other utility events.

#### **EQ2_onMenderRepairAll**

Fired when repairing all items at a mender.

**Arguments:**
- `TargetID` (string): ID of the mender NPC

**Usage:**
```lavishscript
Event[EQ2_onMenderRepairAll]:AttachAtom[OnMenderRepair]

atom OnMenderRepair(string TargetID)
{
    echo "Repaired all items at mender ${TargetID}"
}
```

---

#### **EQ2_onSoundEffect**

Fired when a sound effect plays.

**Arguments:**
- `EffectName` (string): Name of the sound effect

**Usage:**
```lavishscript
Event[EQ2_onSoundEffect]:AttachAtom[OnSoundEffect]

atom OnSoundEffect(string EffectName)
{
    echo "Sound effect played: ${EffectName}"
}
```

**Notes:**
- Not every sound effect in the game is guaranteed to trigger this event
- Can be used with the `EQ2:CreateSoundEffect` method to replay sounds

---

### ISXEQ2 System Events

Events specific to the ISXEQ2 extension itself.

#### **ISXEQ2_onInstanceReloadingAfterUpdate**

Fired when ISXEQ2 is reloading after an update in another session.

**Arguments:**
- `SessionPatched` (string): Session name where the update occurred

**Usage:**
```lavishscript
Event[ISXEQ2_onInstanceReloadingAfterUpdate]:AttachAtom[OnReloading]

atom OnReloading(string SessionPatched)
{
    echo "ISXEQ2 is reloading after update in session: ${SessionPatched}"
}
```

**Notes:**
- This event helps coordinate extension updates across multiple game sessions
- Scripts can use this to clean up before the extension reloads
- The automatic cross-session reload feature is currently disabled in the code

---

## Deprecated Features

This section documents features that have been removed from ISXEQ2 and provides migration paths to current alternatives.

### Removed Character Members (November 2018)

The following experience and vitality members were removed from the `character` datatype. Use `GetGameData` member instead:

**Experience Members:**
- ~~`ExpPoints`~~ → Use `${Me.GetGameData[Self.Experience].Label}`
- ~~`ExpPointsToLevel`~~ → Use `${Me.GetGameData[Self.ExperienceNextLevel].Label}`
- ~~`TSExpPoints`~~ → Use `${Me.GetGameData[Self.TradeskillExperience].Label}`
- ~~`TSExpPointsToLevel`~~ → Use `${Me.GetGameData[Self.TradeskillExperienceNextLevel].Label}`
- ~~`Exp`~~ → Use `${Me.GetGameData[Self.ExperienceCurrent].Label}`
- ~~`ExpDebt`~~ → Use `${Me.GetGameData[Self.ExperienceDebtCurrent].Label}`
- ~~`TSExp`~~ → Use `${Me.GetGameData[Self.TradeskillExperienceCurrent].Label}`
- ~~`TSExpDebt`~~ → Use `${Me.GetGameData[Self.TSExperienceDebtCurrent].Label}`

**Vitality Members:**
- ~~`Vitality`~~ → Use `${Me.GetGameData[Self.Vitality].Label}`
- ~~`TSVitality`~~ → Use `${Me.GetGameData[Self.TSVitality].Label}`

**Other Members:**
- ~~`MentoringXPAdj`~~ → Use `${Me.GetGameData[Self.MentoringXPAdj].Label}`

### Removed Pet Members (March 2012)

**Pet Information:**
- ~~`PetName`~~ → Use `${Me.GetGameData[Pet.Name].Label}`
- ~~`PetHealth`~~ → Use `${Me.GetGameData[Pet.ActualHealth].ShortLabel.Left[-1]}`
- ~~`PetPower`~~ → Use `${Me.GetGameData[Pet.ActualPower].ShortLabel.Left[-1]}`

### Removed Actor Members (July 2020)

**Mount/Transport Members:**
- ~~`OnHorse`~~ → Use `${Actor.OnTransport}`
- ~~`OnCarpet`~~ → Use `${Actor.OnTransport}`
- ~~`OnGriffin`~~ → Use `${Actor.OnTransport}`

**Note:** All mount types are now consolidated into the `OnTransport` boolean member.

### Removed Character Member (November 2022)

- ~~`Breath`~~ → This member was removed entirely with no replacement

### Removed TLO (December 2016)

- ~~`EQ2DataSourceContainer`~~ → Use `${Me.GetGameData}` instead

**Migration Example:**
```lavishscript
; Old way (no longer works):
echo ${EQ2DataSourceContainer.GetGameData[Self.Experience].Label}

; New way:
echo ${Me.GetGameData[Self.Experience].Label}
```

### Complete GetGameData Migration Examples

See the [GetGameData Examples](#getgamedata-examples) section under the **char** datatype for comprehensive examples of all experience, vitality, pet, and bank coin values accessible through GetGameData.

---

## Usage Examples

### Basic Information

```lavishscript
; Character information
echo ${Me.Name}                    ; "Mycharacter"
echo ${Me.Level}                   ; 120
echo ${Me.Class}                   ; "Guardian"
echo ${Me.CurrentHealth}           ; 85000
echo ${Me.MaxHealth}               ; 100000
echo ${Me.Platinum}                ; 523

; Target information
echo ${Target.Name}                ; "a gnoll scout"
echo ${Target.Level}               ; 50
echo ${Target.Distance}            ; 15.3
echo ${Target.Type}                ; "NPC"
echo ${Target.IsAggro}             ; TRUE
echo ${Target.ConColor}            ; "grey"
```

### Inventory and Items

```lavishscript
; Access inventory items
echo ${Me.Inventory[5].Name}                            ; "Glowing Black Stone"
echo ${Me.Inventory[5].Quantity}                        ; 10
echo ${Me.Inventory[Pack1].IsContainer}                 ; TRUE
echo ${Me.Inventory[Pack1].NumSlots}                    ; 8

; Item information
echo ${Me.Equipment[chest].Name}                        ; "Breastplate of Valor"
echo ${Me.Equipment[chest].ToItemInfo.Type}             ; "Heavy Armor"
echo ${Me.Equipment[Primary].ToItemInfo.DamageRating}   ; 523
echo ${Me.Equipment[Primary].ToItemInfo.Delay}          ; 2.5

; Use items
Me.Inventory[5]:Use
Me.Inventory["Health Potion"]:Consume
```

### Abilities and Casting

```lavishscript
; Check abilities
echo ${Me.Ability["Holy Strike"].IsReady}               ; TRUE
echo ${Me.Ability["Holy Strike"].TimeUntilReady}        ; 0
echo ${Me.NumAbilities}                                 ; 156

; Cast abilities
Me.Ability["Holy Strike"]:Use
Me.Ability[1]:Use

; Ability information
echo ${Me.Ability["Holy Strike"].ToAbilityInfo.PowerCost}      ; 500
echo ${Me.Ability["Holy Strike"].ToAbilityInfo.CastingTime}    ; 2.5
echo ${Me.Ability["Holy Strike"].ToAbilityInfo.Range}          ; 15.0
```

### Effects and Maintained Spells

```lavishscript
; Check effects
echo ${Me.CountEffects}                         ; 12
echo ${Me.Effect[1].Duration}                   ; 45.5
echo ${Target.Effect[1].ToEffectInfo.Name}      ; "Curse of Weakness"

; Maintained spells
echo ${Me.CountMaintained}                      ; 3
echo ${Me.Maintained[1].Name}                   ; "Protection of the Wild"
echo ${Me.Maintained[1].Duration}               ; 120.0
Me.Maintained[1]:Cancel
```

### Quests

```lavishscript
; Quest journal
echo ${QuestJournalWindow.NumActiveQuests}      ; 15
echo ${QuestJournalWindow.ActiveQuest[1].Name}  ; "The Hunt Begins"
echo ${QuestJournalWindow.ActiveQuest[1].Level} ; 50

; Current quest
echo ${QuestJournalWindow.CurrentQuest.Name}    ; "The Hunt Begins"
```

### Commerce

```lavishscript
; Merchant
echo ${MerchantWindow.NumItems}                 ; 25
echo ${MerchantWindow.Item[1].Name}             ; "Iron Sword"
echo ${MerchantWindow.Item[1].Price}            ; 5.25
MerchantWindow.Item[1]:Buy[1]

; Broker
echo ${BrokerWindow.NumItems}                   ; 100
```

### UI Interaction

```lavishscript
; Access UI elements
echo ${EQ2UIPage[MapWindow].IsVisible}          ; TRUE
EQ2UIPage[MapWindow]:Close

; Button clicks
EQ2UIPage[LootWindow].Child[LootAllButton,eq2button]:LeftClick

; Window information
echo ${LootWindow.NumItems}                     ; 5
echo ${LootWindow.Item[1].Name}                 ; "Gold Coin"
LootWindow:LootAll
```

### Advanced Queries

```lavishscript
; Find specific actors
EQ2:CreateCustomActorArray[Distance,50,All]
echo ${EQ2.CustomActorArraySize}                ; 35
echo ${CustomActor[1].Name}                     ; "a gnoll scout"

; Query inventory
variable index:item MyItems
Me:QueryInventory[MyItems,"Name=~Potion"]
echo ${MyItems.Used}                            ; 5

; Query abilities
variable index:ability MyAbilities
Me:QueryAbilities[MyAbilities,"IsReady"]
echo ${MyAbilities.Used}                        ; 42
```

### Crafting

```lavishscript
; Recipe information
echo ${Me.Recipe[1].Name}                       ; "Iron Sword"
echo ${Me.Recipe[1].Level}                      ; 20
Me.Recipe[1]:Create

; Crafting session
echo ${Crafting.Progress}                       ; 35
echo ${Crafting.Durability}                     ; 80
echo ${Crafting.Quality}                        ; 45
```

### Movement and Position

```lavishscript
; Character position
echo ${Me.X}                                    ; 123.45
echo ${Me.Y}                                    ; 67.89
echo ${Me.Z}                                    ; -45.67
echo ${Me.Heading}                              ; 180.5

; Distance calculations
echo ${Me.Distance[100,50,-25]}                 ; 45.3
echo ${Me.HeadingTo[100,50,-25,AsString]}       ; "180.5h"

; Movement
Me:DoFace
Me:WaypointTo
```

### Extension Utilities

```lavishscript
; ISXEQ2 utilities
echo ${ISXEQ2.Version}                          ; "2023.09.01"
echo ${ISXEQ2.IsReady}                          ; TRUE
ISXEQ2:AddLoc["MyLocation","Notes here"]
ISXEQ2:Popup["Message","Title",OK]

; EQ2 utilities
echo ${EQ2.ServerName}                          ; "Maj'Dul"
echo ${EQ2.Zoning}                              ; FALSE
echo ${StripTags["<c 00FF00>Green Text</c>"]}   ; "Green Text"
```

### Adornment Attachment

```lavishscript
; Attaching an adornment to an item
function AttachAdornment(string adornmentName, string itemName)
{
    variable item AdornmentItem
    variable item TargetItem

    ; Find the adornment and target item
    AdornmentItem:Set[${Me.Inventory[${adornmentName}]}]
    TargetItem:Set[${Me.Inventory[${itemName}]}]

    if !${AdornmentItem.IsItemInfoAvailable}
    {
        echo "Adornment not found: ${adornmentName}"
        return
    }

    if !${TargetItem(exists)}
    {
        echo "Target item not found: ${itemName}"
        return
    }

    ; Prepare the adornment for use
    AdornmentItem.ToItemInfo:PrepAdornmentForUse
    wait 10

    ; Attach the adornment to the target item in slot 0
    AdornmentItem.ToItemInfo:AttachAsAdornment[${TargetItem.ID}, 0]

    ; Wait for the casting to complete
    wait 30 ${Me.CastingSpell} == FALSE
    wait 5

    ; Cancel the attach cursor
    press esc

    echo "Adornment ${adornmentName} attached to ${itemName}"
}

; Example usage
call AttachAdornment "Flickering Adornment of Blasting (Greater)" "Warlord's Blade"
```

### Fast Travel

```lavishscript
; Fast travel to a specific zone
function FastTravelTo(string zoneName)
{
    variable int i

    ; Open the fast travel window
    EQ2:OpenTravelMapWindow
    wait 10 ${TravelMapWindow(exists)}

    if !${TravelMapWindow(exists)}
    {
        echo "Failed to open travel map window"
        return
    }

    ; Search through available teleport locations
    for (i:Set[1]; ${i} <= ${TravelMapWindow.TeleportLocations}; i:Inc)
    {
        if ${TravelMapWindow.TeleportLocation[${i}].ZoneName.Find[${zoneName}]}
        {
            echo "Found travel location: ${TravelMapWindow.TeleportLocation[${i}].ZoneName}"
            echo "Destination: ${TravelMapWindow.TeleportLocation[${i}].DestinationDescription}"

            ; Select the location and travel
            ; (Note: Additional UI interaction may be needed to actually initiate travel)
            TravelMapWindow:Close
            return
        }
    }

    echo "Could not find travel location matching: ${zoneName}"
    TravelMapWindow:Close
}

; List all available fast travel locations
function ListFastTravelLocations()
{
    variable int i

    EQ2:OpenTravelMapWindow
    wait 10 ${TravelMapWindow(exists)}

    if !${TravelMapWindow(exists)}
    {
        echo "Failed to open travel map window"
        return
    }

    echo "Available Fast Travel Locations:"
    echo "================================"

    for (i:Set[1]; ${i} <= ${TravelMapWindow.TeleportLocations}; i:Inc)
    {
        echo "${i}. ${TravelMapWindow.TeleportLocation[${i}].ZoneName}"
        echo "   ${TravelMapWindow.TeleportLocation[${i}].DestinationDescription}"
        echo "   Level Range: ${TravelMapWindow.TeleportLocation[${i}].LowerLevel}-${TravelMapWindow.TeleportLocation[${i}].UpperLevel}"
    }

    TravelMapWindow:Close
}

; Example usage
call ListFastTravelLocations
call FastTravelTo "Commonlands"
```

### House Item Movement

```lavishscript
; Moving house items programmatically
function MoveHouseItem(uint actorID, float newX, float newY, float newZ)
{
    variable actor HouseItem
    HouseItem:Set[${Actor[id,${actorID}]}]

    if !${HouseItem(exists)}
    {
        echo "Actor ${actorID} not found"
        return
    }

    ; Pick up the item
    HouseItem:Move
    wait 5 ${EQ2.ObjectBeingMoved(exists)}

    if !${EQ2.ObjectBeingMoved(exists)}
    {
        echo "Failed to pick up object"
        return
    }

    echo "Moving object from ${EQ2.ObjectBeingMoved.OldLocation}"
    echo "Actor ID: ${EQ2.ObjectBeingMoved.ActorID}"

    ; Place the object at new coordinates
    ; (Note: You would need to position the mouse at the appropriate screen location)
    ; For demonstration - this shows the concept
    EQ2:PlaceMoveableObject

    echo "Object placed at new location"
}

; Cancel moving an object
if ${EQ2.ObjectBeingMoved(exists)}
{
    echo "Canceling object move"
    EQ2:CancelMoveObject
}
```

### Asynchronous Data Loading

Many detailed info objects require asynchronous loading. Always check availability before accessing:

```lavishscript
; Item information with async check
variable item MyItem
MyItem:Set[${Me.Inventory["Glowing Stone"]}]

if !${MyItem.IsItemInfoAvailable}
{
    ; Wait for item info to load
    variable int Timeout = 0
    do
    {
        waitframe
    }
    while !${MyItem.IsItemInfoAvailable} && ${Timeout:Inc} < 1500
}

; Now safe to access detailed info
echo ${MyItem.ToItemInfo.Description}
echo ${MyItem.ToItemInfo.Type}

; Effect information with pre-request
Me:RequestEffectsInfo
wait 5

variable index:effect MyEffects
Me:QueryEffect[MyEffects,"IsBeneficial"]
variable iterator EffectIterator
MyEffects:GetIterator[EffectIterator]

if ${EffectIterator:First(exists)}
{
    do
    {
        if !${EffectIterator.Value.IsEffectInfoAvailable}
        {
            variable int Timer = 0
            do { waitframe }
            while !${EffectIterator.Value.IsEffectInfoAvailable} && ${Timer:Inc} < 1500
        }

        echo ${EffectIterator.Value.ToEffectInfo.Name}
        echo ${EffectIterator.Value.ToEffectInfo.Description}
    }
    while ${EffectIterator:Next(exists)}
}

; Ability information with timeout
variable index:ability MyAbilities
Me:QueryAbilities[MyAbilities,"Name =- Strike"]
variable iterator AbilityIterator
MyAbilities:GetIterator[AbilityIterator]

if ${AbilityIterator:First(exists)}
{
    do
    {
        if !${AbilityIterator.Value.IsAbilityInfoAvailable}
        {
            variable int Timer = 0
            do { wait 2 }
            while !${AbilityIterator.Value.IsAbilityInfoAvailable} && ${Timer:Inc} < 1500
        }

        echo ${AbilityIterator.Value.ToAbilityInfo.PowerCost}
        echo ${AbilityIterator.Value.ToAbilityInfo.Description}
    }
    while ${AbilityIterator:Next(exists)}
}

; Recipe information
variable index:recipe MyRecipes
Me:QueryRecipes[MyRecipes,"Level == 20"]
variable iterator RecipeIterator
MyRecipes:GetIterator[RecipeIterator]

if ${RecipeIterator:First(exists)}
{
    do
    {
        if !${RecipeIterator.Value.IsRecipeInfoAvailable}
        {
            variable int Timeout = 0
            do { waitframe }
            while !${RecipeIterator.Value.IsRecipeInfoAvailable} && ${Timeout:Inc} < 1500
        }

        echo ${RecipeIterator.Value.Name}: ${RecipeIterator.Value.ToRecipeInfo.Description}
    }
    while ${RecipeIterator:Next(exists)}
}
```

### Advanced Query Patterns

Query strings support complex filtering with special operators:

```lavishscript
; Query operators:
; == (equals), != (not equals), > (greater), < (less), >= (greater or equal), <= (less or equal)
; =- (contains, case-insensitive), !- (does not contain)
; =~ (regex match), !~ (regex does not match)
; && (and), || (or)

; Find actors within range and specific level
Me:CreateCustomActorArray[NearbyMobs,"Distance < 50 && Level == 120"]
echo Found ${EQ2.CustomActorArraySize} actors

; Find inventory items with complex criteria
variable index:item Items
Me:QueryInventory[Items,"Location == \"Inventory\" && Name =- \"Fang\" && Quantity > 1"]
Items:GetIterator[ItemIterator]

; Find detrimental effects lasting longer than 30 seconds
variable index:effect Detriments
Me:QueryEffect[Detriments,"IsDetrimental && Duration > 30"]

; Find ready combat arts
variable index:ability ReadyAbilities
Me:QueryAbilities[ReadyAbilities,"IsReady && Type == \"Combat Art\""]

; Complex actor search
Me:CreateCustomActorArray[Targets,"Type == \"NPC\" && Level >= 110 && Distance < 100 && !IsDead"]

; Find specific item types
Me:QueryInventory[Weapons,"Type =- \"Weapon\" && ToItemInfo.MinLevel <= ${Me.Level}"]

; Find maintained spells by name pattern
variable index:effect Wards
Me:QueryEffect[Wards,"IsMaintained && Name =- \"Ward\""]
```

### Event-Driven Scripting

Use events and atoms for reactive scripting:

```lavishscript
; Auto-sacrifice items added to altar
function main()
{
    while !${ISXEQ2.IsReady}
    {
        waitframe
    }

    ; Register and attach event handler
    LavishScript:RegisterEvent[EQ2_ItemAddedToAltarForSacrifice]
    Event[EQ2_ItemAddedToAltarForSacrifice]:AttachAtom[OnItemAddedToAltar]

    echo "Auto-sacrifice enabled"
    wait 99999999 ${StopScript}
}

atom OnItemAddedToAltar()
{
    ; Add randomized delay to appear more human
    wait ${Math.Rand[10]:Inc[5]}

    ; Execute sacrifice command
    EQ2Execute "/deity_offer"
    wait ${Math.Rand[10]:Inc[5]}

    ; Confirm if dialog appears
    if ${ReplyDialog(exists)}
    {
        ReplyDialog:ChooseReply[0]
    }
}

; Combat event handler example
function main()
{
    Event[EQ2_onIncomingChatText]:AttachAtom[OnChat]
    wait 99999999
}

atom OnChat(string ChatType, string ChatMessage, string ChatTarget, string ChatSender)
{
    if ${ChatMessage.Find["You are being attacked"]}
    {
        echo "Under attack! Responding..."
        ; Defensive response code here
    }
}
```

### UI Widget Inspection

Explore and interact with UI widget hierarchies:

```lavishscript
; Recursive widget dumping utility
function DumpWidgets(string WidgetPath, int Depth)
{
    variable int Counter = 1
    variable string DumpText
    variable string Padding

    ; Build padding for depth visualization
    while ${Padding.Length} < ${Depth}
        Padding:Concat[" "]

    ; Get widget properties
    DumpText:Set["${Padding}Label: ${EQ2UIPage[${WidgetPath}].Label}"]
    DumpText:Concat["\n${Padding}ShortLabel: ${EQ2UIPage[${WidgetPath}].ShortLabel}"]
    DumpText:Concat["\n${Padding}NumChildren: ${EQ2UIPage[${WidgetPath}].NumChildren}"]

    echo "${DumpText}"

    ; Recursively dump children
    while ${Counter} <= ${EQ2UIPage[${WidgetPath}].NumChildren}
    {
        call DumpWidgets "${WidgetPath}.Child[${Counter}]" ${Math.Calc[${Depth}+2]}
        Counter:Inc
    }
}

; Usage - explore a specific UI page
call DumpWidgets "MainHUD" 0

; Check widget properties
variable string TextColor
TextColor:Set[${EQ2UIPage[MainHUD,GroupMembers].Child[Page,GroupMember1.MemberInfoPage.MemberInfo].Child[Text,2].GetProperty[TextColor]}]
echo "Text Color: ${TextColor}"

; Yellow color (ffff00 or 7f7f00) indicates group leader
if ${TextColor.Right[6].Equal["ffff00"]} || ${TextColor.Right[6].Equal["7f7f00"]}
{
    echo "This member is the group leader"
}
```

### Window Interaction Examples

```lavishscript
; Iterate and accept rewards
function main()
{
    if !${RewardWindow(exists)}
    {
        echo "No RewardWindow exists"
        return
    }

    variable int Counter = 1
    echo "RewardWindow has ${RewardWindow.NumRewards} rewards available"

    ; List all rewards
    do
    {
        echo "- [${RewardWindow.Reward[${Counter}].LinkID}] ${RewardWindow.Reward[${Counter}].Name}"
    }
    while ${Counter:Inc} <= ${RewardWindow.NumRewards}

    ; Accept specific reward by name or ID
    RewardWindow:AcceptReward["Glowing Sword"]
    ; or
    RewardWindow:AcceptReward[12345]
}

; Reforge window interaction
function UseReforgeWindow()
{
    if !${ReforgeWindow.IsVisible}
    {
        echo "ReforgeWindow is not visible"
        return
    }

    echo "Item: ${ReforgeWindow.ItemName}"
    echo "Stats: ${ReforgeWindow.StatsText}"
    echo "Currency: ${ReforgeWindow.CurrencyAmount}"

    ; Set attribute slider to position 5
    ReforgeWindow:SetAttributeSlider[5]

    ; Get dropdown options
    variable iterator OptionsIt
    ReforgeWindow:GetDropdownOptions[OptionsIt]
    if ${OptionsIt:First(exists)}
    {
        do
        {
            echo "Option: ${OptionsIt.Value}"
        }
        while ${OptionsIt:Next(exists)}
    }
}

; Get dropdown box options
function GetDropdownOptions()
{
    variable index:collection:string Options
    variable iterator OptionsIterator
    variable int OptionCounter = 0

    ; Get options from a specific dropdown
    EQ2UIPage[mainhud,guild].Child[DropDownBox,Guild.MainTabPage.MembersPage.ShowMemberSelection]:GetOptions[Options]
    Options:GetIterator[OptionsIterator]

    echo "The dropdown has ${Options.Used} options"

    if ${OptionsIterator:First(exists)}
    {
        do
        {
            if ${OptionsIterator.Value.FirstKey(exists)}
            {
                do
                {
                    echo "Option #${OptionCounter}: '${OptionsIterator.Value.CurrentKey}' => '${OptionsIterator.Value.CurrentValue}'"
                }
                while ${OptionsIterator.Value.NextKey(exists)}
            }
            OptionCounter:Inc
        }
        while ${OptionsIterator:Next(exists)}
    }
}

; Reply dialog interaction
function HandleReplyDialog()
{
    if !${ReplyDialog(exists)}
    {
        echo "No reply dialog available"
        return
    }

    echo "Dialog Text: ${ReplyDialog.Text}"

    variable index:collection:string Options
    variable iterator OptionsIterator

    ReplyDialog:GetOptions[Options]
    Options:GetIterator[OptionsIterator]

    if ${OptionsIterator:First(exists)}
    {
        do
        {
            if ${OptionsIterator.Value.FirstKey(exists)}
            {
                do
                {
                    echo "Reply Option: '${OptionsIterator.Value.CurrentKey}' => '${OptionsIterator.Value.CurrentValue}'"
                }
                while ${OptionsIterator.Value.NextKey(exists)}
            }
        }
        while ${OptionsIterator:Next(exists)}
    }

    ; Choose a specific reply
    ReplyDialog:ChooseReply[0]
}

; NPC conversation interaction
function InteractWithNPC()
{
    ; Get last NPC statement
    variable string NPCText
    NPCText:Set[${EQ2UIPage[ProxyActor,Conversation].Child[Text,ChatPage.MessageText].Label}]
    echo "NPC said: ${NPCText}"

    ; Count available responses
    variable int NumResponses
    NumResponses:Set[${EQ2UIPage[ProxyActor,Conversation].Child[composite,replies].NumChildren}]
    echo "You have ${NumResponses} response options"

    ; Get specific response text
    variable int i
    for (i:Set[1]; ${i} <= ${NumResponses}; i:Inc)
    {
        echo "${i}. ${EQ2UIPage[ProxyActor,Conversation].Child[composite,replies].Child[button,${i}].Label}"
    }

    ; Click a specific response (button 1)
    EQ2UIPage[ProxyActor,Conversation].Child[composite,replies].Child[button,1]:LeftClick
}
```

### Quest Journal Iteration

```lavishscript
; Iterate through active and completed quests
function ListAllQuests()
{
    variable index:quest Quests
    variable iterator QuestIt
    variable int NumQuests
    variable int Counter = 1

    ; Get active quests
    NumQuests:Set[${QuestJournalWindow.NumActiveQuests}]
    QuestJournalWindow:GetActiveQuests[Quests]
    Quests:GetIterator[QuestIt]

    echo "=== Active Quests (${NumQuests}) ==="

    if ${QuestIt:First(exists)}
    {
        do
        {
            echo "- [${Counter}] ${QuestIt.Value.Name}"
            echo "-- ID: ${QuestIt.Value.ID}"
            echo "-- Level: ${QuestIt.Value.Level}"
            echo "-- Category: ${QuestIt.Value.Category}"
            echo "-- Zone: ${QuestIt.Value.Zone}"
            Counter:Inc
        }
        while ${QuestIt:Next(exists)}
    }

    ; Get completed quests
    Counter:Set[1]
    NumQuests:Set[${QuestJournalWindow.NumCompletedQuests}]
    QuestJournalWindow:GetCompletedQuests[Quests]
    Quests:GetIterator[QuestIt]

    echo "=== Completed Quests (${NumQuests}) ==="

    if ${QuestIt:First(exists)}
    {
        do
        {
            echo "- [${Counter}] ${QuestIt.Value.Name}"
            Counter:Inc
        }
        while ${QuestIt:Next(exists)}
    }
}
```

### Specialized Item Methods

```lavishscript
; Enchant an item with a scroll
function EnchantWeapon()
{
    variable int WeaponID = ${Me.Equipment[primary].ID}

    ; Use enchantment scroll
    Me.Inventory["scroll of combat"]:Use
    wait 5

    ; Apply to weapon
    Me.Inventory["scroll of combat"]:EnchantItem[${WeaponID}]
    wait 5

    ; Wait for casting to complete
    do
    {
        waitframe
    }
    while ${Me.CastingSpell}

    press esc
    echo "Weapon enchanted"
}
```

### Account and Character Data

```lavishscript
; Get account features
function ListAccountFeatures()
{
    variable int Counter = 1
    variable index:string AccountFeatures
    variable iterator Iterator

    EQ2:GetAccountFeatures[AccountFeatures]
    AccountFeatures:GetIterator[Iterator]

    echo "This EQ2 account has ${AccountFeatures.Used} features:"

    if ${Iterator:First(exists)}
    {
        do
        {
            echo "${Counter}: ${Iterator.Value}"
            Counter:Inc
        }
        while ${Iterator:Next(exists)}
    }
}

; Manipulate AA experience slider
function SetAAConversion(int Percentage)
{
    ; Get current conversion
    echo "Current AA conversion: ${EQ2DataSourceContainer[GameData].GetDynamicData[Achievement.ExperienceConversion].Percent}%"

    ; Set new conversion percentage
    EQ2Execute "achievement_conversion ${Percentage}"
    wait 5

    echo "AA conversion set to ${Percentage}%"
}

; Access GameData values
echo ${Me.GetGameData[Self.Experience].Label}
echo ${Me.GetGameData[Self.ExperienceNextLevel].Label}
echo ${Me.GetGameData[Self.TradeskillExperience].Label}
echo ${Me.GetGameData[Group.Group_1.Health].Label}
echo ${Me.GetGameData[Raid.Member_1.Effect1].Label}
```

### Group Leader Detection

```lavishscript
; Detect group leader using UI widget inspection
function DetectGroupLeader()
{
    variable int GroupCounter = 1
    variable string GMName
    variable string GMType
    variable string RGBAColor
    variable int NumChildren

    ; Check if player is leader
    if ${Me.IsGroupLeader}
        echo "0. ${Me.Name} (GROUP LEADER)"
    else
        echo "0. ${Me.Name}"

    ; Check each group member
    do
    {
        NumChildren:Set[${EQ2UIPage[MainHUD,GroupMembers].Child[Page,GroupMember${GroupCounter}.MemberInfoPage.MemberInfo].NumChildren}]
        if ${NumChildren} < 8
            continue

        GMType:Set[${Me.Group[${GroupCounter}].ToActor.Type}]
        if ${GMType.Equal["Mercenary"]}
            continue

        GMName:Set[${EQ2UIPage[MainHUD,GroupMembers].Child[Page,GroupMember${GroupCounter}.MemberInfoPage.MemberInfo].Child[Text,2].GetProperty[Text]}]
        RGBAColor:Set[${EQ2UIPage[MainHUD,GroupMembers].Child[Page,GroupMember${GroupCounter}.MemberInfoPage.MemberInfo].Child[Text,2].GetProperty[TextColor].Right[6]}]

        ; Yellow color indicates group leader
        if ${RGBAColor.Equal["ffff00"]} || ${RGBAColor.Equal["7f7f00"]}
            echo "${GroupCounter}. ${GMName} (GROUP LEADER)"
        else
            echo "${GroupCounter}. ${GMName}"
    }
    while ${GroupCounter:Inc} <= 5
}
```

### Commands

```lavishscript
; Targeting and facing
Target NPC                                      ; Target nearest NPC
Target gnoll                                    ; Target by partial name
Face                                            ; Face current target
Face NPC                                        ; Face nearest NPC
Face 180                                        ; Face south

; Location management
eq2loc Add Home ${Me.X} ${Me.Y} ${Me.Z} "${Zone.Name}" 0 "My home location"
eq2loc List                                     ; List all locations in current zone
eq2loc List bank                                ; Filter locations containing "bank"

; Finding actors
Where NPC 50                                    ; Find all level 50 NPCs
Where NamedNPC                                  ; Find all named NPCs
Where Chest byDist                              ; Find all chests sorted by distance

; Broker searches
broker Name "rough opal"                        ; Simple search by name
broker Name sword MinLevel 50 MaxLevel 60      ; Search with level range
broker Tier Legendary Class Guardian MinPrice 1000  ; Complex search

; Chat and output
eq2echo Player: ${Me.Name} Level: ${Me.Level}  ; Echo to console
eq2echo -chattype say Hello everyone!          ; Send to say chat
announce "Quest Complete!" 5.0                  ; On-screen announcement

; Filtering chat
eq2ignore Add "WTS"                             ; Ignore phrase in all channels
eq2ignore Add "spam" 5                          ; Ignore phrase in specific channel
eq2ignore List                                  ; List all ignored phrases

; Executing EQ2 commands
EQ2Execute /say Hello world
EQ2Execute /useability "Holy Strike"
EQ2Execute /camp

; Activating equipped items
Activate chest                                  ; Activate chest item
Activate primary                                ; Activate primary weapon

; Practical targeting script example
function TargetAndAttack(string mobName)
{
    Target ${mobName}
    wait 5
    if ${Target.Name.Equal[${mobName}]}
    {
        Face
        wait 2
        EQ2Execute /auto_attack
        Me.Ability[1]:Use
    }
}

; Practical location saving example
function SaveCurrentLocation(string label, string notes)
{
    eq2loc Add "${label}" ${Me.X} ${Me.Y} ${Me.Z} "${Zone.Name}" 0 "${notes}"
    echo Saved location: ${label}
}

; Practical broker search script
function SearchBrokerForUpgrades()
{
    broker Name weapon MinLevel ${Me.Level} Tier Legendary Class ${Me.Class}
    wait 10
}

; Combat helper with announcements
function CombatAlert()
{
    if ${Me.Health} < ${Math.Calc[${Me.MaxHealth}*0.3]}
    {
        announce "Low Health!" 3.0 1
        ; Use healing ability
        Me.Ability["Heal"]:Use
    }
}
```

---

## Notes

### Deprecated Members

Some members are marked as deprecated and should not be used in new scripts:
- `Item.IsInitialized` - Use `IsItemInfoAvailable` instead
- `Ability.TimeRemaining` - Use `TimeUntilReady` instead
- `Actor.ToActor` - No longer necessary

### Case Sensitivity

LavishScript is case-insensitive for member and method names:
- `${Me.Name}` and `${Me.name}` are equivalent
- `Me:DoTarget` and `Me:dotarget` are equivalent

### NULL Checks

Always check if a datatype exists before accessing its members:

```lavishscript
if ${Target(exists)}
{
    echo ${Target.Name}
}
```

### Parameter Notation

- `[parameter]` - Optional parameter
- `parameter` - Required parameter
- `param1|param2` - Alternative parameters (use one or the other)

---
