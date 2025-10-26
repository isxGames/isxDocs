# ISXEQ2 API Reference

Complete API reference extracted from official ISXEQ2 documentation. This comprehensive guide documents all datatypes, members, methods, and their relationships for creating, debugging, and updating ISXEQ2 scripts.

**Version:** Based on official ISXEQ2 Reference
**Last Updated:** 2025-10-20

---

## Table of Contents

- [Introduction](#introduction)
  - [DataType Inheritance](#datatype-inheritance)
  - [Accessing DataTypes](#accessing-datatypes)
  - [NULL Checks](#null-checks)
  - [Asynchronous Data Loading](#asynchronous-data-loading)
- [Top-Level Objects (TLOs)](#top-level-objects-tlos)
  - [Extension TLOs](#extension-tlos)
  - [Game TLOs](#game-tlos)
  - [Window TLOs](#window-tlos)
- [Datatypes by Category](#datatypes-by-category)
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
- [Complete DataType Reference](#complete-datatype-reference)

---

## Introduction

This document provides comprehensive reference documentation for all datatypes and top-level objects available in the ISXEQ2 extension for EverQuest 2. ISXEQ2 extends LavishScript to provide access to game data and functionality through a structured type system.

### DataType Inheritance

Many datatypes inherit from other datatypes, gaining all the members and methods of their parent type. Common inheritance patterns:

- **char** inherits from **actor** - All actor members/methods are available on char objects
- **groupmember** inherits from **actor** - Group members have all actor capabilities
- **raidmember** inherits from **groupmember** (and **actor**) - Raid members have all group and actor capabilities
- **eq2widget** inherits from **eq2baseobject** - All widgets have base object properties
- **eq2window** provides base functionality for all window types
- **eq2clonewindow** inherits from **eq2window** - Clone windows have window functionality

**Inheritance Chain Examples:**
```
char → actor
groupmember → actor
raidmember → groupmember → actor
eq2button → eq2widget → eq2baseobject
lootwindow → eq2clonewindow → eq2window
```

### Accessing DataTypes

DataTypes are accessed through Top-Level Objects (TLOs) or through members of other datatypes. For example:

```lavishscript
${Me}                           ; TLO returning 'char' datatype
${Me.Target}                    ; Member returning 'actor' datatype
${Me.Inventory[5]}              ; Member returning 'item' datatype
${EQ2UIPage[MapWindow]}         ; TLO with parameter returning 'eq2uipage' datatype
${Me.Ability[1].ToAbilityInfo}  ; Method returning 'abilityinfo' datatype
```

### NULL Checks

Always check if a datatype exists before accessing its members:

```lavishscript
if ${Target(exists)}
{
    echo ${Target.Name}
    echo ${Target.Distance}
}

; Check for nested objects
if ${Me.Inventory[5](exists)} && ${Me.Inventory[5].IsItemInfoAvailable}
{
    echo ${Me.Inventory[5].ToItemInfo.Description}
}
```

### Asynchronous Data Loading

Many detailed info objects require asynchronous loading from the server. Always check availability before accessing detailed information:

**Item Information:**
```lavishscript
if !${MyItem.IsItemInfoAvailable}
{
    variable int Timeout = 0
    do { waitframe }
    while !${MyItem.IsItemInfoAvailable} && ${Timeout:Inc} < 1500
}
echo ${MyItem.ToItemInfo.Description}
```

**Effect Information:**
```lavishscript
Me:RequestEffectsInfo
wait 5
if ${MyEffect.IsEffectInfoAvailable}
{
    echo ${MyEffect.ToEffectInfo.Name}
}
```

**Ability Information:**
```lavishscript
if ${MyAbility.IsAbilityInfoAvailable}
{
    echo ${MyAbility.ToAbilityInfo.PowerCost}
}
```

**Recipe Information:**
```lavishscript
if ${MyRecipe.IsRecipeInfoAvailable}
{
    echo ${MyRecipe.ToRecipeInfo.Description}
}
```

---

## Top-Level Objects (TLOs)

Top-Level Objects (TLOs) are the entry points for accessing game data. They can be accessed directly in LavishScript without any prefix.

### Extension TLOs

| TLO | DataType | Description |
|-----|----------|-------------|
| **ISXEQ2** | [isxeq2](#isxeq2) | ISXEQ2 extension information and utilities |

### Game TLOs

| TLO | DataType | Parameters | Description |
|-----|----------|------------|-------------|
| **EQ2** | [eq2](#eq2) | - | Main game information and utilities |
| **Me** | [char](#char) | - | Player character |
| **Target** | [actor](#actor) | - | Current target |
| **Zone** | [zone](#zone) | - | Current zone information |
| **Actor** | [actor](#actor) | id / name | Access specific actor by ID or name |
| **CustomActor** | [actor](#actor) | index | Access custom actor array element |
| **Radar** | [radar](#radar) | - | Radar functionality |
| **EQ2Loc** | [eq2location](#eq2location) | x,y,z | Create location object |
| **Crafting** | [crafting](#crafting) | - | Crafting session information |
| **Achievement** | [achievement](#achievement) | id / name | Access achievement data |
| **StripTags** | string | text | Removes EQ2 formatting tags from text |
| **EQ2DataSourceContainer** | [eq2datasourcecontainer](#eq2datasourcecontainer) | name | Access UI data source (deprecated - use ${Me.GetGameData} instead) |

### Window TLOs

| TLO | DataType | Description |
|-----|----------|-------------|
| **EQ2UIPage** | [eq2uipage](#eq2uipage) | Access UI page by name |
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
| **TravelMapWindow** | [travelmapwindow](#travelmapwindow) | Travel map (fast travel) window |
| **RadialMenuWindow** | [radialmenuwindow](#radialmenuwindow) | Radial menu window |
| **BrokerWindow** | [brokerwindow](#brokerwindow) | Broker search window |
| **MerchantWindow** | [merchantwindow](#merchantwindow) | Merchant window |
| **ChannelerWindow** | [channelerwindow](#channelerwindow) | Channeler class window |
| **BeastlordWindow** | [beastlordwindow](#beastlordwindow) | Beastlord class window |

---

## Datatypes by Category

### Core Game DataTypes

Core datatypes that provide fundamental game information and utilities.

- [isxeq2](#isxeq2) - Extension information and utilities
- [eq2](#eq2) - Main game information and utilities
- [zone](#zone) - Zone information
- [eq2location](#eq2location) - 3D location coordinate
- [radar](#radar) - Radar functionality
- [class](#class) - Class information
- [achievement](#achievement) - Achievement information
- [reward](#reward) - Quest/loot reward

### Actor DataTypes

Datatypes representing actors (characters, NPCs, objects) in the game world.

- [actor](#actor) - Base datatype for all actors
- [char](#char) - Player character (inherits from actor)
- [groupmember](#groupmember) - Group member (inherits from actor)
- [raidmember](#raidmember) - Raid member (inherits from groupmember → actor)
- [moveableobject](#moveableobject) - Moveable house object
- [equipmentappearance](#equipmentappearance) - Equipment appearance information

### Item DataTypes

Datatypes for inventory, equipment, and item information.

- [item](#item) - Item in inventory, equipment, or containers
- [iteminfo](#iteminfo) - Detailed item examination data
- [itemmodifier](#itemmodifier) - Item stat modifier
- [itemeffectstring](#itemeffectstring) - Item effect description string
- [packageditem](#packageditem) - Packaged item in reward/examine
- [adornment](#adornment) - Adornment attached to item

### Ability DataTypes

Datatypes for character abilities and spells.

- [ability](#ability) - Character ability/spell
- [abilityinfo](#abilityinfo) - Detailed ability information

### Effect DataTypes

Datatypes for buffs, debuffs, and maintained spells.

- [effect](#effect) - Character effect/buff
- [maintained](#maintained) - Maintained buff/spell
- [actoreffect](#actoreffect) - Effect on another actor
- [effectinfo](#effectinfo) - Detailed effect information
- [effectstring](#effectstring) - Effect description string

### Quest DataTypes

Datatypes for quest journal and quest information.

- [quest](#quest) - Quest information
- [currentquest](#currentquest) - Currently selected quest in journal

### Commerce DataTypes

Datatypes for merchants, brokers, and trading.

- [merchandise](#merchandise) - Item for sale from merchant
- [consignment](#consignment) - Broker/consignment item
- [vendingcontainer](#vendingcontainer) - Player vendor container

### Mail DataTypes

Datatypes for the mail system.

- [inboxmailmessage](#inboxmailmessage) - Mail message in inbox
- [mailmessage](#mailmessage) - Opened mail message

### Crafting DataTypes

Datatypes for tradeskill crafting.

- [recipe](#recipe) - Crafting recipe
- [recipeinfo](#recipeinfo) - Detailed recipe information
- [primarycomponent](#primarycomponent) - Recipe primary component
- [buildcomponent](#buildcomponent) - Recipe build component
- [crafting](#crafting) - Crafting session information

### Window DataTypes

Datatypes for game windows and dialogs.

- [eq2window](#eq2window) - Base window type
- [eq2clonewindow](#eq2clonewindow) - Clone window type (inherits from eq2window)
- [journalquestwindow](#journalquestwindow) - Quest journal window
- [lootwindow](#lootwindow) - Loot window
- [choicewindow](#choicewindow) - Choice dialog window
- [containerwindow](#containerwindow) - Container window
- [containerwindowitem](#containerwindowitem) - Item in container window
- [rewardwindow](#rewardwindow) - Reward pack window
- [replydialog](#replydialog) - NPC conversation window
- [mapwindow](#mapwindow) - Map window
- [travelmapwindow](#travelmapwindow) - Travel map window
- [teleportlocation](#teleportlocation) - Teleport location on map
- [travelmapwindowlocation](#travelmapwindowlocation) - Travel map location
- [radialmenuwindow](#radialmenuwindow) - Radial menu window
- [radialmenuaction](#radialmenuaction) - Radial menu action
- [brokerwindow](#brokerwindow) - Broker search window
- [merchantwindow](#merchantwindow) - Merchant window
- [mailwindow](#mailwindow) - Mail inbox window
- [openedmailwindow](#openedmailwindow) - Opened mail window
- [reforgewindow](#reforgewindow) - Item reforge window
- [examineitemwindow](#examineitemwindow) - Item examination window
- [channelerwindow](#channelerwindow) - Channeler class window
- [beastlordwindow](#beastlordwindow) - Beastlord class window

### Widget/UI DataTypes

Datatypes for UI elements and widgets.

- [eq2baseobject](#eq2baseobject) - Base class for all EQ2 UI objects
- [eq2datasourcecontainer](#eq2datasourcecontainer) - UI data source container
- [eq2dynamicdata](#eq2dynamicdata) - UI dynamic data
- [eq2widget](#eq2widget) - Base UI widget (inherits from eq2baseobject)
- [eq2button](#eq2button) - Button widget
- [eq2text](#eq2text) - Text display widget
- [eq2icon](#eq2icon) - Icon widget
- [eq2textbox](#eq2textbox) - Text box widget
- [eq2scrollbar](#eq2scrollbar) - Scrollbar widget
- [eq2sliderbar](#eq2sliderbar) - Slider bar widget
- [eq2dropdownbox](#eq2dropdownbox) - Dropdown box widget
- [eq2list](#eq2list) - List widget
- [eq2listbox](#eq2listbox) - List box widget
- [eq2progressbar](#eq2progressbar) - Progress bar widget
- [eq2checkbox](#eq2checkbox) - Checkbox widget
- [eq2uipage](#eq2uipage) - UI page container
- [eq2composite](#eq2composite) - Composite UI element
- [eq2tabbedpane](#eq2tabbedpane) - Tabbed pane widget
- [eq2iconbank](#eq2iconbank) - Icon bank (ability bars, etc.)

---

## Complete DataType Reference

---

### isxeq2

Extension information and utilities.

**Access:** `${ISXEQ2}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Version | string | ISXEQ2 version string |
| APIVersion | string | API version string |
| IsReady | bool | TRUE if extension is ready for use |
| EQ2LocsCount[allzones] | int | Number of saved locations (pass TRUE for all zones) |
| IsValidEQ2PressKey[keyname] | bool | TRUE if key name is valid for EQ2 |
| AfflictionEventsOn | bool | TRUE if affliction events are enabled |
| GetCustomVariable[name,type] | variable | Gets custom variable value |
| GetCurrencyString[amount] | string | Formats currency amount as string |
| Round[type,value,multiple] | variable | Rounds value to nearest multiple |

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| ClearAbilitiesCache | - | Clears the abilities cache |
| ClearRecipesCache | - | Clears the recipes cache |
| Popup | text, title, status | Shows popup window with message |
| AddLoc | label, notes | Adds current location to saved locations |
| EnduringBreath / EB | - | Toggles enduring breath |
| NoFog | - | Toggles fog removal |
| EnableActorEvents | - | Enables actor spawn/despawn events |
| DisableActorEvents | - | Disables actor events |
| SetActorEventsRange | range | Sets range for actor events (in meters) |
| SetActorEventsTimeInterval | ms | Sets check interval for actor events (in milliseconds) |
| ResetInternalVendingSystem | - | Resets the vending system |
| EnableAfflictionEvents | - | Enables affliction events |
| DisableAfflictionEvents | - | Disables affliction events |
| SetAfflictionEventsTimeInterval | ms | Sets check interval for affliction events |
| EnableCustomZoningText | - | Enables custom zoning text |
| DisableCustomZoningText | - | Disables custom zoning text |
| SetCustomVariable | name, value | Sets custom variable |
| ClearAllCustomVariables | - | Clears all custom variables |
| InstallBeta | - | Switches to beta build |
| InstallTest | - | Switches to test build |
| InstallLive | - | Switches to live build |
| Reload | delay | Reloads extension after delay (in deciseconds) |
| Unload | - | Unloads the extension |

**Example Usage:**
```lavishscript
echo ${ISXEQ2.Version}
echo ${ISXEQ2.IsReady}
ISXEQ2:AddLoc["Bank","Main bank location"]
ISXEQ2:Popup["Message","Title",OK]
```

---

### eq2

Main game information and utilities.

**Access:** `${EQ2}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| ServerName | string | Server name |
| Zoning | int | Non-zero if currently zoning |
| CustomActorArraySize | int | **DEPRECATED** - Use EQ2:GetActors instead |
| PendingQuestName | string | Name of pending quest offer |
| PendingQuestDescription | string | Description of pending quest |
| MasterVolume | float | Master volume (0-100) |
| NumRadars | int | Number of radars |
| InboxMailCount | int | Number of inbox mail messages |
| HOWindowActive | bool | TRUE if Heroic Opportunity window is active |
| HOName | string | Current HO name |
| HODescription | string | Current HO description |
| HOWheelID | int | HO wheel ID |
| HOWheelState | int | HO wheel state |
| HOCurrentWheelSlot | int | Current HO wheel slot |
| HOWindowState | int | HO window state |
| HOTimeLimit | float | HO time limit in seconds |
| HOTimeElapsed | float | HO time elapsed in seconds |
| HOTimeRemaining | float | HO time remaining in seconds |
| HOLastManipulator | [actor](#actor) | Last actor to manipulate the HO |
| HOIconID1 through HOIconID6 | int | HO icon IDs for slots 1-6 |
| CheckCollision[x1,y1,z1,x2,y2,z2] | bool | TRUE if collision exists between two points |
| HeadingTo[x1,y1,z1,x2,y2,z2,asstring] | float/string | Calculates heading between two points (returns string if "asstring" passed) |
| PersistentZoneID[name] | uint | Gets zone ID by zone name |
| ReadyToRefineTransmuteOrSalvage | bool | TRUE if ready for refine/transmute/salvage |
| AbilityTierString[tier] | string | Gets tier string (Apprentice, Journeyman, etc.) from tier number |
| AtCharSelect | bool | TRUE if at character select screen |
| LoginState | string | Current login state |
| ObjectBeingMoved | [moveableobject](#moveableobject) | Object currently being moved (housing) |

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| CreateCustomActorArray | sortby, range, type | **DEPRECATED** - Use EQ2:GetActors instead |
| GetActors | index, searchparams | **MODERN METHOD** - Populates index with actor objects |
| QueryActors | index, query | Populates index with actors matching query |
| SetMasterVolume | volume | Sets master volume (0-100) |
| AcceptPendingQuest | - | Accepts pending quest offer |
| DeclinePendingQuest | - | Declines pending quest offer |
| ConfirmZoneTeleporterDestination | - | Confirms zone teleporter destination |
| ShowAllOnScreenAnnouncements | - | Toggles on-screen announcements |
| GetPersistentZones | index | Populates index with persistent zone names |
| GetAccountFeatures | index | Populates index with account feature strings |
| CreateSoundEffect | file | Creates sound effect from file |
| PlaceMoveableObject | - | Places moveable housing object |
| CancelMoveObject | - | Cancels moving housing object |
| OpenTravelMapWindow | - | Opens the fast travel map window |

**Example Usage:**
```lavishscript
echo ${EQ2.ServerName}
echo ${EQ2.Zoning}

; Modern method for getting actors
variable index:actor Actors
EQ2:GetActors[Actors,Range,50,NPC]

EQ2:AcceptPendingQuest
EQ2:OpenTravelMapWindow
```

---

### zone

Zone information.

**Access:** `${Zone}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Name | string | Zone full name |
| ShortName | string | Zone short name |
| ID | uint | Zone ID |
| Description | string | Zone description |

**Example Usage:**
```lavishscript
echo ${Zone.Name}
echo ${Zone.ID}
```

---

### eq2location

3D location coordinate.

**Access:** `${EQ2Loc[x,y,z]}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| X | float | X coordinate |
| Y | float | Y coordinate |
| Z | float | Z coordinate |
| Distance[x,y,z] | float | 3D distance to specified coordinates |
| Distance2D[x,z] | float | 2D distance to specified coordinates (ignores Y) |

**Example Usage:**
```lavishscript
variable eq2location MyLoc
MyLoc:Set[${EQ2Loc[100,50,-25]}]
echo ${MyLoc.X}
echo ${MyLoc.Distance[${Me.X},${Me.Y},${Me.Z}]}
```

---

### radar

Radar functionality.

**Access:** `${Radar}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| NumActors | int | Number of actors on radar |
| Actor[index] | [actor](#actor) | Actor by index (1 to NumActors) |

**Example Usage:**
```lavishscript
echo ${Radar.NumActors}
echo ${Radar.Actor[1].Name}
```

---

### class

Class information.

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Name | string | Class name |
| Level | int | Class level |

**Example Usage:**
```lavishscript
echo ${Me.Ability[1].ToAbilityInfo.Class[1].Name}
```

---

### achievement

Achievement information.

**Access:** `${Achievement[id/name]}` or via `${Me.Achievement[index/name]}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Name | string | Achievement name |
| ID | int | Achievement ID |
| Description | string | Achievement description |
| Points | int | Achievement points value |
| IsComplete | bool | TRUE if achievement is complete |

**Example Usage:**
```lavishscript
echo ${Achievement[123].Name}
echo ${Achievement["Explorer"].IsComplete}
```

---

### reward

Quest/loot reward.

**Access:** Via `${RewardWindow.Reward[index]}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Name | string | Reward item name |
| LinkID | int | Item link ID |
| ToLink | string | Creates clickable chat link |
| Type | string | Reward type |
| IsItemInfoAvailable | bool | TRUE if item info is available |
| ToItemInfo | [iteminfo](#iteminfo) | Converts to detailed iteminfo datatype |

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| Examine | - | Examines the reward |

**Example Usage:**
```lavishscript
echo ${RewardWindow.Reward[1].Name}
RewardWindow.Reward[1]:Examine
```

---

### actor

Base datatype for all actors (NPCs, PCs, objects) in the game world.

**Access:** `${Target}`, `${Actor[id/name]}`, `${CustomActor[index]}`, `${Me.Target}`, etc.

**Inherited By:** [char](#char), [groupmember](#groupmember), [raidmember](#raidmember)

#### Members - Identity

| Member | Type | Description |
|--------|------|-------------|
| Name | string | Actor's name |
| LastName | string | Actor's last name (surname) |
| ID | uint | Unique actor ID |
| Type | string | Actor type (PC, NPC, NamedNPC, Pet, MyPet, Chest, Door, Resource, etc.) |
| Gender | string | Gender (Male/Female) |
| Race | string | Race (returns "NPC" for non-player races) |
| Class | string | Class name |
| Level | int | Actor's level |
| EffectiveLevel | int | Effective/mentor level |
| Guild | string | Guild tag |
| SuffixTitle | string | Suffix title |
| Tooltip | string | Sign/tooltip text |

#### Members - Position & Movement

| Member | Type | Description |
|--------|------|-------------|
| X | float | X coordinate |
| Y | float | Y coordinate |
| Z | float | Z coordinate |
| Loc | point3f | 3D location (X,Y,Z) |
| Distance[x,y,z] | float | 3D distance from player (or from specified coordinates if provided) |
| Distance2D[x,z] | float | 2D distance from player (or from specified coordinates if provided) |
| Heading | float | Current heading (0-360) |
| HeadingTo[asstring] | float/string | Heading from player to actor (returns string if "asstring" passed) |
| Speed | float | Speed modifier |
| Velocity | point3f | Velocity vector |

#### Members - Stats & Combat

| Member | Type | Description |
|--------|------|-------------|
| Health | int | Current health percentage |
| Power | int | Current power/mana percentage |
| Difficulty | int | Difficulty arrows (up arrows) |
| EncounterSize | int | Number of mobs in encounter |
| Faction | int | Faction value (negative = hostile) |
| FactionStanding | string | Faction standing text |
| ConColor[raw] | string | Con color name (or RGB hex if "raw" passed) |
| ThreatToMe | int | Threat to player |
| ThreatToPet | int | Threat to player's pet |
| ThreatToNext | int | Secondary threat |

#### Members - Relationships

| Member | Type | Description |
|--------|------|-------------|
| Target | [actor](#actor) | Actor's current target |
| Pet | [actor](#actor) | Actor's pet |
| WhoFollowing | string | Name of actor being followed |
| WhoFollowingID | int | ID of actor being followed |
| InMyGroup | bool | TRUE if actor is in player's group |
| IsInSameEncounter[actorID] | bool | TRUE if actor is in same encounter as specified actor |

#### Members - Effects

| Member | Type | Description |
|--------|------|-------------|
| NumEffects | int | Number of visible effects |
| Effect[index/id/name/query] | [actoreffect](#actoreffect) | Get effect by index, "id",id / "name" / "Query","query" |

#### Members - Appearance

| Member | Type | Description |
|--------|------|-------------|
| CollisionRadius | float | Collision radius |
| CollisionScale | float | Collision scale |
| TargetRingRadius | float | Target ring radius |
| CurrentAnimation | string | Current animation state |
| Overlay | string | Overlay animation |
| Aura | string | Aura effect |
| Mood | string | Mood animation |
| VisualVariant | string | Visual variant |
| TintFlags | uint | Tint flags |
| EquipmentAppearance[slot] | [equipmentappearance](#equipmentappearance) | Equipment appearance at slot (1-34) |
| MountAppearance | [equipmentappearance](#equipmentappearance) | Mount appearance |

#### Members - Casting

| Member | Type | Description |
|--------|------|-------------|
| AbilityCastingID | int | ID of ability being cast |
| AbilityCastingTime | float | Time remaining for cast (in seconds) |

#### Members - Tags

| Member | Type | Description |
|--------|------|-------------|
| TagTargetNumber | string | Tag number (1-6 or empty) |
| TagTargetIcon | string | Tag icon (Skull, Shield, Star, Sword, Cross, Flame) |

#### Members - State Checks (Boolean)

| Member | Type | Description |
|--------|------|-------------|
| IsAggro | bool | TRUE if aggressive/KOS |
| IsSolo | bool | TRUE if solo encounter |
| IsHeroic | bool | TRUE if heroic encounter |
| IsEpic | int | Epic tier (0 if not epic) |
| IsEncounterBroken | bool | TRUE if encounter is broken |
| IsAFK | bool | TRUE if AFK flag set |
| IsLFG | bool | TRUE if looking for group |
| IsLFW | bool | TRUE if looking for work |
| IsCamping | bool | TRUE if camping out |
| IsLocked | bool | TRUE if locked |
| IsAPet | bool | TRUE if is a pet |
| IsMyPet | bool | TRUE if is player's pet |
| IsNamed | bool | TRUE if named NPC |
| IsSwimming | bool | TRUE if swimming |
| SwimmingSpeedMod | float | Swimming speed modifier |
| InCombatMode | bool | TRUE if in combat stance |
| IsCrouching | bool | TRUE if crouching |
| IsSitting | bool | TRUE if sitting |
| OnTransport | bool | TRUE if on mount/transport |
| IsRunning | bool | TRUE if running |
| IsWalking | bool | TRUE if walking |
| IsSprinting | bool | TRUE if sprinting |
| IsBackingUp | bool | TRUE if moving backward |
| IsStrafingLeft | bool | TRUE if strafing left |
| IsStrafingRight | bool | TRUE if strafing right |
| IsIdle | bool | TRUE if idle (not moving) |
| IsClimbing | bool | TRUE if climbing |
| IsJumping | bool | TRUE if jumping |
| IsFalling | bool | TRUE if falling |
| IsDead | bool | TRUE if dead |
| IsInvis | bool | TRUE if invisible |
| IsStealthed | bool | TRUE if stealthed |
| IsBanker | bool | TRUE if is banker NPC |
| IsMerchant | bool | TRUE if is merchant NPC |
| IsChest | bool | TRUE if is chest/container |
| IsRooted | bool | TRUE if rooted |
| CanTurn | bool | TRUE if can turn |
| IsFD | bool | TRUE if feign death |
| Interactable | bool | TRUE if highlights on mouse hover |
| HighlightOnMouseHover | bool | TRUE if highlights on mouse hover |
| FlyingUsingMount | bool | TRUE if flying using mount |
| OnFlyingMount | bool | TRUE if on flying mount |
| UpdatesMyQuest | bool | TRUE if updates player's quest |
| UpdatesGroupMemberQuest | bool | TRUE if updates group member quest |
| ActiveStateExists | bool | TRUE if has active states |
| OffersQuest | bool | TRUE if offers a quest |
| CheckCollision[x,y,z] | bool | TRUE if collision exists to specified point |

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| DoubleClick | - | Double-clicks the actor |
| RightClick | - | Right-clicks the actor |
| DoTarget | - | Targets the actor |
| DoFace | - | Faces toward the actor |
| WaypointTo / WT | - | Creates waypoint to actor |
| Location | - | Prints location info to console |
| RequestEffectsInfo | - | Requests effects information from server |
| GetActiveStates | - | Gets active states |
| Resize | scale | Resizes actor (visual only) |
| Move | x, y, z | Moves actor to coordinates (housing items) |
| Set | actorID | Sets actor reference by ID |

**Example Usage:**
```lavishscript
echo ${Target.Name}
echo ${Target.Level}
echo ${Target.Type}
echo ${Target.Distance}
echo ${Target.ConColor}
Target:DoFace
Target:WaypointTo

; Check for specific actor type
if ${Target.Type.Equal["NamedNPC"]}
    echo "Named NPC detected!"

; Distance check
if ${Target.Distance} < 5
    Target:DoubleClick
```

---

### char

Player character datatype. Inherits all members and methods from [actor](#actor).

**Access:** `${Me}`

**Inherits From:** [actor](#actor)

#### Members - Stats

| Member | Type | Description |
|--------|------|-------------|
| CurrentHealth / Health | int | Current health |
| MaxHealth | int | Maximum health |
| CurrentPower / Power | int | Current power |
| MaxPower | int | Maximum power |
| UsedConc | int | Used concentration |
| MaxConc | int | Maximum concentration |
| Strength | int | Strength stat (with buffs) |
| Stamina | int | Stamina stat (with buffs) |
| Agility | int | Agility stat (with buffs) |
| Wisdom | int | Wisdom stat (with buffs) |
| Intelligence | int | Intelligence stat (with buffs) |
| BaseStrength | int | Base strength (no buffs) |
| BaseStamina | int | Base stamina (no buffs) |
| BaseAgility | int | Base agility (no buffs) |
| BaseWisdom | int | Base wisdom (no buffs) |
| BaseIntelligence | int | Base intelligence (no buffs) |

#### Members - Resistances

| Member | Type | Description |
|--------|------|-------------|
| ElementalResist | int | Elemental resistance value |
| NoxiousResist | int | Noxious resistance value |
| ArcaneResist | int | Arcane resistance value |
| ElementalResistPct | float | Elemental resist percentage |
| NoxiousResistPct | float | Noxious resist percentage |
| ArcaneResistPct | float | Arcane resist percentage |

#### Members - Currency

| Member | Type | Description |
|--------|------|-------------|
| Copper | int | Copper coins |
| Silver | int | Silver coins |
| Gold | int | Gold coins |
| Platinum | int | Platinum coins |

#### Members - Class Info

| Member | Type | Description |
|--------|------|-------------|
| Archetype | string | Character archetype |
| SubClass | string | Character subclass |
| TSArchetype | string | Tradeskill archetype |
| TSClass | string | Tradeskill class |
| TSSubClass | string | Tradeskill subclass |
| TSLevel | int | Tradeskill level |

#### Members - Effects & Maintained

| Member | Type | Description |
|--------|------|-------------|
| CountEffects | int | Number of effects on character |
| CountMaintained | int | Number of maintained spells |
| Effect[index/id/name] | [effect](#effect) | Get effect by index, "id",id, or "name" |
| Maintained[index] | [maintained](#maintained) | Get maintained spell by index |
| NumEffects | int | Number of effects |

#### Members - Inventory

| Member | Type | Description |
|--------|------|-------------|
| Inv[slot] / Inventory[slot/query] | [item](#item) | Inventory item by slot or "Query","query" |
| Equip[slot] / Equipment[slot] | [item](#item) | Equipped item by slot |
| CustomInventory[index] | [item](#item) | Custom inventory array item |
| CustomInventoryArraySize | int | Size of custom inventory array |
| NextFreeInvContainer | int | Next free inventory container slot |
| InventorySlotsFree | int | Number of free inventory slots |
| BankSlotsFree | int | Number of free bank slots |
| SharedBankSlotsFree | int | Number of free shared bank slots |

**Valid Equipment Slots:** primary, secondary, ranged, ammo, head, chest, shoulders, forearms, hands, legs, feet, finger1, finger2, ears1, ears2, neck, waist, primarycharm, secondarycharm, wrist1, wrist2, cloak

**Valid Inventory Slots:** 0-15 (numbered slots), pack1-pack10 (inventory bags), bank1-bank16 (bank bags), sharedbank1-sharedbank12 (shared bank bags)

#### Members - Group & Raid

| Member | Type | Description |
|--------|------|-------------|
| Group[index] | [groupmember](#groupmember) | Group member by index (1 to GroupCount) |
| GroupCount | int | Number of group members |
| Grouped | bool | TRUE if in a group |
| IsGroupLeader | bool | TRUE if group leader |
| Raid[index] | [raidmember](#raidmember) | Raid member by index (1 to RaidCount) |
| RaidCount | int | Number of raid members |
| InRaid | bool | TRUE if in a raid |
| RaidGroupNum | int | Raid group number (1-4) |

#### Members - Abilities & Recipes

| Member | Type | Description |
|--------|------|-------------|
| NumAbilities | int | Number of abilities |
| Ability[index/name] | [ability](#ability) | Get ability by index or name |
| NumRecipes | int | Number of recipes |
| Recipe[index/name] | [recipe](#recipe) | Get recipe by index or name |

#### Members - State

| Member | Type | Description |
|--------|------|-------------|
| InGameWorld | bool | TRUE if in game world (not char select) |
| AtCharSelect | bool | TRUE if at character select |
| CastingSpell | bool | TRUE if currently casting |
| AutoAttackOn | bool | TRUE if auto-attack enabled |
| RangedAutoAttackOn | bool | TRUE if ranged auto-attack on |
| IsMoving | bool | TRUE if character is moving |
| In1stPersonView | bool | TRUE if in first-person view |
| In3rdPersonView | bool | TRUE if in third-person view |
| IsSitting | bool | TRUE if sitting |
| InWater | bool | TRUE if in water |
| InCombat | bool | TRUE if in combat |
| TargetLOS | bool | TRUE if target in line of sight |
| WaterDepth | float | Water depth |
| TimeToCampOut | float | Seconds remaining to camp out |

#### Members - Flags

| Member | Type | Description |
|--------|------|-------------|
| IsAnonymous | bool | TRUE if anonymous |
| IsRolePlaying | bool | TRUE if roleplaying flag |
| IsDecliningGroupInvites | bool | TRUE if declining groups |
| IsDecliningTradeInvites | bool | TRUE if declining trades |
| IsDecliningDuelInvites | bool | TRUE if declining duels |
| IsDecliningRaidInvites | bool | TRUE if declining raids |
| IsDecliningGuildInvites | bool | TRUE if declining guilds |
| IgnoringAll | bool | TRUE if ignoring all |
| GuildPrivacyOn | bool | TRUE if guild privacy on |
| CombatExpEnabled | bool | TRUE if combat XP enabled |

#### Members - Afflictions

| Member | Type | Description |
|--------|------|-------------|
| Arcane | int | Number of arcane afflictions |
| Elemental | int | Number of elemental afflictions |
| Trauma | int | Number of trauma afflictions |
| Noxious | int | Number of noxious afflictions |
| Cursed | int | Number of cursed afflictions |
| IsAfflicted | bool | TRUE if has any afflictions |

#### Members - Channeler Specific

| Member | Type | Description |
|--------|------|-------------|
| Dissonance | int | Current dissonance |
| DissonanceRemaining | int | Remaining dissonance capacity |
| MaxDissonance | int | Maximum dissonance |
| Dissipation | int | Dissipation value |

#### Members - Miscellaneous

| Member | Type | Description |
|--------|------|-------------|
| CursorActor | [actor](#actor) | Actor under mouse cursor |
| TotalEarnedAPs | int | Total earned achievement points |
| MaxAPs | int | Maximum achievement points |
| Achievement[index/name] | [achievement](#achievement) | Get achievement by index or name |
| IsInPVP | bool | TRUE if in PVP zone |
| IsHated | bool | TRUE if hated |
| PowerRegen | float | Power regeneration rate |
| HealthRegen | float | Health regeneration rate |
| InZone | bool | TRUE if fully loaded in zone |
| CameraPitch | float | Camera pitch angle |

#### Members - Collections

| Member | Type | Description |
|--------|------|-------------|
| GetInventory | collection | Returns LavishScript collection of item objects |
| GetInventoryAtHand | collection | Returns at-hand inventory collection |
| GetEquipment | collection | Returns equipment collection |
| GetAbilities | collection | Returns abilities collection |

#### Members - Dynamic Game Data

| Member | Type | Description |
|--------|------|-------------|
| GetGameData[string] | [eq2dynamicdata](#eq2dynamicdata) | Retrieves dynamic game data (experience, vitality, pet info, etc.) |

**Common GetGameData Values:**

*Experience:*
- `Self.Experience` - Adventure experience
- `Self.ExperienceNextLevel` - XP to next level
- `Self.ExperienceCurrent` - Current XP this level
- `Self.ExperienceDebtCurrent` - XP debt
- `Self.ExperienceBubble` - Experience bubble
- `Self.TradeskillExperience` - TS experience
- `Self.TradeskillExperienceNextLevel` - TS XP to next level
- `Self.TitheExperience` - Tithe experience
- `Self.AscensionExperience` - Ascension experience

*Vitality:*
- `Self.Vitality` - Vitality
- `Self.VitalityUpperMarker` - Vitality upper marker
- `Self.TSVitality` - Tradeskill vitality

*Pet Info:*
- `Pet.Name` - Pet name
- `Pet.ActualHealth` - Pet health
- `Pet.Health` - Pet health percentage
- `Pet.ActualPower` - Pet power
- `Pet.Power` - Pet power percentage

*Bank Coin:*
- `Coins.BankCoin_0` through `Coins.BankCoin_3` - Bank copper, silver, gold, platinum
- `Coins.SharedCoin_0` through `Coins.SharedCoin_3` - Shared bank coins

*Target Info:*
- `Target.Threat` - Threat to target

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| Face | heading/x,y,z/"actor",id | Face direction, coordinates, or actor |
| CreateCustomInventoryArray | - | Creates custom inventory array (access via CustomInventory[index]) |
| TakeAllVendingCoin | - | Takes all coin from vending containers |
| BankDeposit | amount | Deposits specified amount to bank |
| BankWithdraw | amount | Withdraws specified amount from bank |
| GuildBankDeposit | amount | Deposits to guild bank |
| GuildBankWithdraw | amount | Withdraws from guild bank |
| SharedBankDeposit | amount | Deposits to shared bank |
| SharedBankWithdraw | amount | Withdraws from shared bank |
| ResetZoneTimer | - | Resets zone timer |
| DepositIntoHouseEscrow | amount | Deposits to house escrow |
| QueryInventory | query | Queries inventory (populates index with item objects) |
| QueryEffects | query | Queries effects (populates index with effect objects) |
| RequestEffectsInfo | - | Requests effects information from server |
| QueryAbilities | query | Queries abilities (populates index with ability objects) |
| QueryRecipes | query | Queries recipes (populates index with recipe objects) |
| Set | characterID | Sets character reference by ID |
| SetCameraPitch | pitch | Sets camera pitch angle |

**Example Usage:**
```lavishscript
echo ${Me.Name}
echo ${Me.Level}
echo ${Me.CurrentHealth}
echo ${Me.MaxHealth}
echo ${Me.Platinum}

; Inventory access
echo ${Me.Inventory[5].Name}
echo ${Me.Equipment[chest].Name}

; Abilities
if ${Me.Ability["Holy Strike"].IsReady}
    Me.Ability["Holy Strike"]:Use

; Group
echo ${Me.GroupCount}
echo ${Me.Group[1].Name}

; GetGameData examples
echo ${Me.GetGameData[Self.Experience].Label}
echo ${Me.GetGameData[Pet.Name].Label}
```

---

### groupmember

Group member datatype. Inherits all members and methods from [actor](#actor).

**Access:** `${Me.Group[index]}`

**Inherits From:** [actor](#actor)

#### Additional Members

| Member | Type | Description |
|--------|------|-------------|
| ID | int | Group member ID |
| PetID | int | Pet ID |
| ToActor | [actor](#actor) | Returns as actor datatype |
| EffectiveLevel | int | Effective level |
| Noxious | int | Noxious afflictions |
| Cursed | int | Cursed afflictions |
| Arcane | int | Arcane afflictions |
| Elemental | int | Elemental afflictions |
| Trauma | int | Trauma afflictions |
| IsAfflicted | bool | TRUE if has afflictions |
| RaidRole | string | Raid role |
| RaidGroupNum | int | Raid group number |
| InZone | bool | TRUE if in zone with player |

**Example Usage:**
```lavishscript
echo ${Me.Group[1].Name}
echo ${Me.Group[1].EffectiveLevel}
echo ${Me.Group[1].InZone}
echo ${Me.Group[1].ToActor.Distance}
```

---

### raidmember

Raid member datatype. Same as [groupmember](#groupmember) - inherits all members and methods from [groupmember](#groupmember) and [actor](#actor).

**Access:** `${Me.Raid[index]}`

**Inherits From:** [groupmember](#groupmember) → [actor](#actor)

**Note:** When in a raid, both `${Me.Group[index]}` and `${Me.Raid[index]}` return this datatype.

**Example Usage:**
```lavishscript
echo ${Me.Raid[1].Name}
echo ${Me.Raid[1].RaidGroupNum}
```

---

### moveableobject

Moveable house object (for housing item placement).

**Access:** `${EQ2.ObjectBeingMoved}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| OldLocation | point3f | Location before object was moved |
| OldHeading | float | Heading before object was moved |
| OldScale | float | Scale before object was moved |
| NewHeading | float | New heading |
| NewScale | float | New scale |
| HeightOffset | float | Height difference between old and current locations |
| CurrentMouseX | int | Current mouse X position |
| CurrentMouseY | int | Current mouse Y position |
| ActorID | uint | ID of the actor being moved |

**Example Usage:**
```lavishscript
if ${EQ2.ObjectBeingMoved(exists)}
{
    echo "Moving object ID: ${EQ2.ObjectBeingMoved.ActorID}"
    echo "Old location: ${EQ2.ObjectBeingMoved.OldLocation}"
    EQ2:PlaceMoveableObject
}
```

**See Also:** `EQ2:PlaceMoveableObject`, `EQ2:CancelMoveObject`, `Actor:Move`

---

### equipmentappearance

Equipment appearance information for actor equipment slots.

**Access:** `${Actor.EquipmentAppearance[slot]}`, `${Actor.MountAppearance}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| ID | uint | Appearance ID |
| Texture1 | string | First texture |
| Texture2 | string | Second texture |
| Texture3 | string | Third texture |

**Example Usage:**
```lavishscript
echo ${Me.EquipmentAppearance[1].ID}
echo ${Target.MountAppearance.Texture1}
```

---

### item

Item in inventory, equipment, or containers.

**Access:** `${Me.Inventory[slot]}`, `${Me.Equipment[slot]}`, `${Me.CustomInventory[index]}`

#### Members - Identity

| Member | Type | Description |
|--------|------|-------------|
| Name | string | Item name |
| ID | uint | Item unique ID |
| LinkID | int | Link ID for chat links |
| ToLink | string | Creates clickable chat link |
| SerialNumber | int64 | Item serial number |
| IconID | int | Icon ID |

#### Members - Location

| Member | Type | Description |
|--------|------|-------------|
| Location | string | Location name (Inventory, Bank, Equipment, etc.) |
| LocationID | int | Numeric location ID |
| InContainerID | int | Container ID if item is inside a container |
| ContainerID | int | Container ID if this item IS a container |
| Slot | int | Slot number |
| Bag | int | Bag slot if in inventory container |
| Index | int | Index in inventory array |

#### Members - Properties

| Member | Type | Description |
|--------|------|-------------|
| Quantity | int | Stack quantity |
| EffectiveLevel | int | Effective level |

#### Members - Container Properties

| Member | Type | Description |
|--------|------|-------------|
| IsContainer | bool | TRUE if is a container |
| NumSlots | int | Number of slots (if container) |
| NumSlotsFree | int | Number of free slots (if container) |
| InContainer | bool | TRUE if inside a container |
| InInventorySlot | bool | TRUE if in inventory slot |
| IsInventoryContainer | bool | TRUE if is inventory container |
| IsBankContainer | bool | TRUE if is bank container |
| IsSharedBankContainer | bool | TRUE if is shared bank container |
| IsSlotOpen[slot] | bool | TRUE if slot is open (container) |
| ItemInSlot[slot] | [item](#item) | Item in specified slot (container) |
| NextSlotOpen | int | Next open slot index (container) |

#### Members - Item Type Checks

| Member | Type | Description |
|--------|------|-------------|
| IsAutoConsumeable | bool | TRUE if can auto-consume |
| AutoConsumeOn | bool | TRUE if auto-consume is active |
| CanBeRedeemed | bool | TRUE if can be redeemed |
| IsFoodOrDrink | bool | TRUE if food or drink |
| IsScribeable | bool | TRUE if can be scribed |
| IsUnpackable | bool | TRUE if can be unpacked |
| IsFamiliar | bool | TRUE if is familiar |
| IsAgent | bool | TRUE if is agent |
| IsUsable | bool | TRUE if is usable |

#### Members - Readiness

| Member | Type | Description |
|--------|------|-------------|
| IsReady | bool | TRUE if ready (not on cooldown) |
| TimeUntilReady | float | Seconds until ready (-1 if ready) |

#### Members - Item Info

| Member | Type | Description |
|--------|------|-------------|
| IsItemInfoAvailable | bool | TRUE if detailed ItemInfo data is available |
| ToItemInfo | [iteminfo](#iteminfo) | Returns detailed item information datatype |

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| Destroy | - | Destroys item (no confirmation) |
| DestroyWithConf | - | Destroys item with confirmation dialog |
| Move | location,slot / "container",containerID,slot | Moves item to location |
| Equip | - | Equips the item |
| UnEquip | - | Unequips the item |
| Consume | - | Consumes the item |
| Examine | - | Examines the item |
| Use | - | Uses the item |
| Activate | - | Activates the item |
| Open | - | Opens container |
| SendAsGift | characterName | Sends item as gift to character |
| InstallAsVendingContainer | - | Installs as vendor container |
| AddToConsignment | - | Adds to consignment |
| Transmute | "confirm" | Transmutes item (pass "confirm" to skip dialog) |
| Refine | "confirm" | Refines item |
| Salvage | "confirm" | Salvages item |
| Extract | "confirm" | Extracts mana from item |
| ReclaimAdornments | "confirm" | Reclaims adornments from item |
| Sacrifice | - | Sacrifices item at altar |
| Scribe | - | Scribes spell/art/recipe |
| ToggleAutoConsume | - | Toggles auto-consume |
| AddToDepot | - | Adds to depot |
| ApplyToItem | itemID | Applies item to specified item |
| EnchantItem | itemID | Enchants specified item |
| Read | - | Reads book |
| Unpack | - | Unpacks item |
| AddFamiliar | - | Adds familiar |
| EquipFamiliar | - | Equips familiar |
| UnequipFamiliar | - | Unequips familiar |
| SetAppearanceFamiliar | - | Sets familiar appearance |
| UnsetAppearanceFamiliar | - | Unsets familiar appearance |
| AddAgent | - | Adds agent |
| ConvertAgent | - | Converts agent |
| Set | itemID | Sets item reference by ID |

**Example Usage:**
```lavishscript
echo ${Me.Inventory[5].Name}
echo ${Me.Inventory[5].Quantity}
echo ${Me.Inventory[Pack1].NumSlots}

; Use item
Me.Inventory["Health Potion"]:Consume

; Check detailed info
if ${Me.Equipment[chest].IsItemInfoAvailable}
    echo ${Me.Equipment[chest].ToItemInfo.Type}

; Container operations
if ${Me.Inventory[Pack1].IsContainer}
    echo "Slots free: ${Me.Inventory[Pack1].NumSlotsFree}"
```

**See Also:** [iteminfo](#iteminfo)

---

### iteminfo

Detailed item examination data. Obtained via `Item.ToItemInfo`.

**Access:** `${Item.ToItemInfo}`

**Important:** Always check `Item.IsItemInfoAvailable` before accessing ToItemInfo.

#### Members - Identity

| Member | Type | Description |
|--------|------|-------------|
| Name | string | Item name |
| ID | uint | Item ID |
| LinkID | int | Link ID for chat |
| ToLink | string | Creates clickable chat link |
| SerialNumber | int64 | Item serial number |
| Description | string | Item description |
| Label | string | Item label |
| IconID | int | Icon ID |

#### Members - Type & Tier

| Member | Type | Description |
|--------|------|-------------|
| Type | string | Item type (Weapon, Armor, Food, etc.) |
| SubType | string | Item subtype |
| Tier | int | Item tier (Common=1, Handcrafted=2, Mastercrafted=3, etc.) |
| Level | int | Required level |

#### Members - Container

| Member | Type | Description |
|--------|------|-------------|
| NumSlots | int | Number of container slots |
| EmptySlots | int | Number of empty slots |
| ContentsForSale | bool | TRUE if contents are for sale |

#### Members - Equipment

| Member | Type | Description |
|--------|------|-------------|
| WieldStyle | string | Wield style (One-Hand, Two-Hand, etc.) |
| NumEquipSlots | int | Number of equip slots this item uses |
| EquipSlot[index] | string | Equipment slot name by index |

#### Members - Condition & Charges

| Member | Type | Description |
|--------|------|-------------|
| Condition | int | Condition percentage (0-100) |
| Charges | int | Current charges |
| MaxCharges | int | Maximum charges |
| Crafter | string | Crafter name |

#### Members - Class Restrictions

| Member | Type | Description |
|--------|------|-------------|
| NumClasses | int | Number of usable classes |
| Class[index] | string | Class name by index |

#### Members - Consumable

| Member | Type | Description |
|--------|------|-------------|
| Satiation | int | Satiation value (food/drink) |
| Duration | int | Effect duration |

#### Members - Combat Stats (Weapons)

| Member | Type | Description |
|--------|------|-------------|
| DamageRating | int | Damage rating |
| MyMinDamage | int | Player's min damage with this weapon |
| MyMaxDamage | int | Player's max damage with this weapon |
| BaseMinDamage | int | Base min damage |
| BaseMaxDamage | int | Base max damage |
| MasteryMinDamage | int | Mastery min damage |
| MasteryMaxDamage | int | Mastery max damage |
| Delay | float | Weapon delay in seconds |
| Range / MaxRange | float | Maximum weapon range |
| MinRange | float | Minimum weapon range |
| DamageType | string | Damage type (Slashing, Piercing, etc.) |
| DamageVerbType | string | Damage verb |

#### Members - Defense Stats (Armor/Shields)

| Member | Type | Description |
|--------|------|-------------|
| Mitigation | int | Mitigation value |
| MaxMitigation | int | Maximum mitigation |
| ShieldFactor | int | Shield factor |
| MaxShieldFactor | int | Maximum shield factor |
| Protection | int | Protection value |
| MaxProtection | int | Maximum protection |

#### Members - Modifiers

| Member | Type | Description |
|--------|------|-------------|
| NumModifiers | int | Number of stat modifiers |
| Modifier[index] | [itemmodifier](#itemmodifier) | Stat modifier by index |
| ModFlag | int | Modifier flags |

#### Members - Effects

| Member | Type | Description |
|--------|------|-------------|
| NumEffects | int | Number of effects |
| EffectName[index] | string | Effect name by index |
| EffectDescription[index] | string | Effect description by index |
| NumEffectStrings | int | Number of effect strings |
| EffectString[index] | [itemeffectstring](#itemeffectstring) | Effect string by index |
| CastingTime | float | Casting time in seconds |
| RecoveryTime | float | Recovery time in seconds |
| RecastTime | float | Recast time in seconds |

#### Members - Adornments

| Member | Type | Description |
|--------|------|-------------|
| NumAdornmentsAttached | int | Number of attached adornments |
| Adornment[index] | [adornment](#adornment) | Adornment data by index |

#### Members - Packaging

| Member | Type | Description |
|--------|------|-------------|
| NumItemsPackaged | int | Number of packaged items |
| PackagedItem[index] | [packageditem](#packageditem) | Packaged item by index |

#### Members - Recipe

| Member | Type | Description |
|--------|------|-------------|
| NumItemsCreated | int | Number of items created by recipe |
| CreatesItem[index] | string | Created item name by index |

#### Members - Restrictions & Flags

| Member | Type | Description |
|--------|------|-------------|
| RentStatusReduction | int | Rent status reduction |
| TradeRestrictedDate | string | Trade restriction date |
| Attuned | bool | TRUE if attuned |
| Attuneable | bool | TRUE if can be attuned |
| Lore | bool | TRUE if lore item |
| LoreOnEquip | bool | TRUE if lore on equip |
| Artifact | bool | TRUE if artifact |
| Ornate | bool | TRUE if ornate |
| Temporary | bool | TRUE if temporary |
| NoTrade | bool | TRUE if no-trade |
| NoValue | bool | TRUE if no value |
| NoZone | bool | TRUE if cannot zone with |
| NoDestroy | bool | TRUE if cannot destroy |
| NoRepair | bool | TRUE if cannot repair |
| NoSalvage | bool | TRUE if cannot salvage |
| NoTransmute | bool | TRUE if cannot transmute |
| NoExperiment | bool | TRUE if cannot experiment |
| NoBroker | bool | TRUE if cannot broker |
| NoMail | bool | TRUE if cannot mail |
| Indestructable | bool | TRUE if indestructable |
| Heirloom | bool | TRUE if heirloom |
| HouseLore | bool | TRUE if house lore |
| BuildingBlock | bool | TRUE if building block |
| FreeReforge | bool | TRUE if free reforge available |
| Infusable | bool | TRUE if infusable |
| MercOnly | bool | TRUE if mercenary only |
| MountOnly | bool | TRUE if mount only |
| RequiresEquip | bool | TRUE if requires being equipped |
| AppearanceOnly | bool | TRUE if appearance only |
| Good | bool | TRUE if good-aligned |
| Evil | bool | TRUE if evil-aligned |

#### Members - Quest & Collectible

| Member | Type | Description |
|--------|------|-------------|
| IsCollectible | bool | TRUE if collectible item |
| AlreadyCollected | bool | TRUE if already collected |
| OffersQuest | bool | TRUE if offers quest |
| RequiredByQuest | bool | TRUE if required by quest |
| IsQuestItemUsable | bool | TRUE if quest item is usable |

#### Members - State

| Member | Type | Description |
|--------|------|-------------|
| IsActivatable | bool | TRUE if can be activated |
| CanScribeNow | bool | TRUE if can scribe now |
| Unlocked | bool | TRUE if unlocked |
| Reforged | bool | TRUE if reforged |
| Refined | bool | TRUE if refined |

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| AttachAsAdornment | itemID | Attaches as adornment to specified item |
| PrepAdornmentForUse | - | Prepares adornment for use |

**Example Usage:**
```lavishscript
variable item MyItem
MyItem:Set[${Me.Equipment[chest]}]

if ${MyItem.IsItemInfoAvailable}
{
    echo ${MyItem.ToItemInfo.Type}
    echo ${MyItem.ToItemInfo.Mitigation}
    echo ${MyItem.ToItemInfo.NumModifiers}
}
```

**See Also:** [item](#item), [itemmodifier](#itemmodifier), [adornment](#adornment)

---

### itemmodifier

Item stat modifier.

**Access:** `${ItemInfo.Modifier[index]}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Type | string | Modifier type (Strength, Stamina, etc.) |
| Value | int | Modifier value |

**Example Usage:**
```lavishscript
echo ${MyItem.ToItemInfo.Modifier[1].Type}
echo ${MyItem.ToItemInfo.Modifier[1].Value}
```

---

### itemeffectstring

Item effect description string.

**Access:** `${ItemInfo.EffectString[index]}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Description | string | Effect description text |

**Example Usage:**
```lavishscript
echo ${MyItem.ToItemInfo.EffectString[1].Description}
```

---

### packageditem

Packaged item (in reward pack or examine window).

**Access:** `${ItemInfo.PackagedItem[index]}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Name | string | Item name |
| IconID | int | Icon ID |
| Quantity | int | Quantity |

**Example Usage:**
```lavishscript
echo ${MyItem.ToItemInfo.PackagedItem[1].Name}
echo ${MyItem.ToItemInfo.PackagedItem[1].Quantity}
```

---

### adornment

Adornment attached to item.

**Access:** `${ItemInfo.Adornment[index]}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Name | string | Adornment name |
| Slot | string | Adornment slot name |
| SlotIndex | int | Numeric slot index |
| Description | string | Adornment description |

**Example Usage:**
```lavishscript
echo ${MyItem.ToItemInfo.Adornment[1].Name}
echo ${MyItem.ToItemInfo.Adornment[1].Slot}
```

---

### ability

Character ability/spell.

**Access:** `${Me.Ability[index/name]}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| ID | uint | Ability ID |
| TimeUntilReady | float | Seconds until ready to use |
| IsReady | bool | TRUE if ready to cast |
| IsQueued | bool | TRUE if queued |
| IsAbilityInfoAvailable | bool | TRUE if detailed info available |
| ToAbilityInfo | [abilityinfo](#abilityinfo) | Returns detailed ability information |
| MainIconID | int | Main icon ID |
| BackDropIconID | int | Backdrop icon ID |
| HOIconID | int | Heroic Opportunity icon ID |
| Level | int | Ability level |
| IsConduit | bool | TRUE if is Channeler conduit |

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| Use | - | Uses/casts the ability |
| Examine | - | Examines the ability |
| AddToConduitBar | slot | Adds to Channeler conduit bar (1-8) |
| AddToChannelerBar | slot | Adds to Channeler bar (1-10) |
| AddToBeastlordBar | slot | Adds to Beastlord bar (1-10) |
| Set | ID | Sets ability reference by ID |

**Example Usage:**
```lavishscript
echo ${Me.Ability["Holy Strike"].IsReady}
echo ${Me.Ability["Holy Strike"].TimeUntilReady}

if ${Me.Ability["Holy Strike"].IsReady}
    Me.Ability["Holy Strike"]:Use

; Detailed info
if ${Me.Ability[1].IsAbilityInfoAvailable}
    echo ${Me.Ability[1].ToAbilityInfo.PowerCost}
```

**See Also:** [abilityinfo](#abilityinfo)

---

### abilityinfo

Detailed ability information.

**Access:** `${Ability.ToAbilityInfo}`

**Important:** Always check `Ability.IsAbilityInfoAvailable` before accessing ToAbilityInfo.

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Name | string | Ability name |
| Description | string | Ability description |
| Tier | string | Ability tier (Apprentice, Journeyman, etc.) |
| HealthCost | int64 | Health cost |
| PowerCost | int64 | Power cost |
| SavageryCost | int | Savagery cost |
| DissonanceCost | int | Dissonance cost |
| ConcentrationCost | int | Concentration cost |
| MainIconID | int | Main icon ID |
| HOIconID | int | HO icon ID |
| BackDropIconID | int | Backdrop icon ID |
| CastingTime | float | Casting time in seconds |
| RecoveryTime | float | Recovery time in seconds |
| RecastTime | float | Recast time in seconds |
| MaxDuration | float | Maximum duration (-1 if permanent) |
| ToLink[text] | string | Creates chat link with optional text |
| HealthCostPerTick | int64 | Health cost per tick |
| PowerCostPerTick | int64 | Power cost per tick |
| DissonanceCostPerTick | int | Dissonance cost per tick |
| SavageryCostPerTick | int | Savagery cost per tick |
| MaxAOETargets | int | Maximum AOE targets |
| DoesNotExpire | bool | TRUE if doesn't expire |
| GroupRestricted | bool | TRUE if group restricted |
| AllowRaid | bool | TRUE if allowed in raid |
| SpellBookType | int | Spell book type |
| TargetType | int | Target type ID |
| EffectRadius | float | Effect radius |
| NumEffects | int | Number of effects |
| Effect[index] | [effectstring](#effectstring) | Effect description by index |
| NumClasses | int | Number of classes that can use |
| Class[index/name] | [class](#class) | Class by index or name |
| MinRange | float | Minimum range |
| Range / MaxRange | float | Maximum range |
| IsBeneficial | bool | TRUE if beneficial |
| AscensionClass | string | Ascension class |
| AscensionLevel | int | Ascension level |

**Example Usage:**
```lavishscript
variable ability MyAbility
MyAbility:Set[${Me.Ability["Holy Strike"]}]

if ${MyAbility.IsAbilityInfoAvailable}
{
    echo ${MyAbility.ToAbilityInfo.Name}
    echo ${MyAbility.ToAbilityInfo.PowerCost}
    echo ${MyAbility.ToAbilityInfo.CastingTime}
    echo ${MyAbility.ToAbilityInfo.Range}
}
```

**See Also:** [ability](#ability), [effectstring](#effectstring)

---

### effect

Character effect/buff.

**Access:** `${Me.Effect[index/id/name]}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| CurrentIncrements | int | Stack count |
| ID | int | Effect ID |
| MainIconID | int | Main icon ID |
| BackDropIconID | int | Backdrop icon ID |
| MaxDuration | float | Maximum duration (-1 if permanent) |
| Duration | float | Time remaining in seconds |
| Type | string | Effect type |
| IsEffectInfoAvailable | bool | TRUE if detailed info available |
| ToEffectInfo | [effectinfo](#effectinfo) | Returns detailed effect information |

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| Cancel | - | Cancels the effect |
| Examine | - | Examines the effect |
| Hide | - | Hides the effect from display |
| Set | ID | Sets effect reference by ID |

**Example Usage:**
```lavishscript
echo ${Me.Effect[1].Duration}
echo ${Me.Effect[1].CurrentIncrements}

Me.Effect["Curse"]:Cancel

if ${Me.Effect[1].IsEffectInfoAvailable}
    echo ${Me.Effect[1].ToEffectInfo.Name}
```

**See Also:** [effectinfo](#effectinfo), [maintained](#maintained), [actoreffect](#actoreffect)

---

### maintained

Maintained buff/spell.

**Access:** `${Me.Maintained[index]}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Name | string | Effect name |
| TargetType | string | Target type (Self, SingleTarget, etc.) |
| CurrentIncrements | int | Stack count |
| IsBeneficial | bool | TRUE if beneficial |
| ConcentrationCost | int | Concentration cost |
| MaxDuration | float | Maximum duration (-1 if permanent) |
| Duration | float | Time remaining in seconds |
| Target | [actor](#actor) | Target of the maintained effect |
| UsesRemaining | int64 | Uses remaining |
| DamageRemaining | int64 | Damage remaining |

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| Cancel | - | Cancels the maintained effect |
| Examine | - | Examines the effect |

**Example Usage:**
```lavishscript
echo ${Me.Maintained[1].Name}
echo ${Me.Maintained[1].Duration}
echo ${Me.Maintained[1].ConcentrationCost}

Me.Maintained[1]:Cancel
```

**See Also:** [effect](#effect), [actoreffect](#actoreffect)

---

### actoreffect

Effect on another actor.

**Access:** `${Actor.Effect[index/id/name/query]}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| CurrentIncrements | int | Stack count |
| ID | int | Effect ID |
| MainIconID | int | Main icon ID |
| BackDropIconID | int | Backdrop icon ID |
| IsEffectInfoAvailable | bool | TRUE if detailed info available |
| ToEffectInfo | [effectinfo](#effectinfo) | Returns detailed effect information |

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| Examine | - | Examines the effect |

**Example Usage:**
```lavishscript
echo ${Target.Effect[1].ID}
Target.Effect[1]:Examine

if ${Target.Effect[1].IsEffectInfoAvailable}
    echo ${Target.Effect[1].ToEffectInfo.Name}
```

**See Also:** [effect](#effect), [effectinfo](#effectinfo)

---

### effectinfo

Detailed effect information.

**Access:** `${Effect.ToEffectInfo}` or `${ActorEffect.ToEffectInfo}`

**Important:** Always check `IsEffectInfoAvailable` before accessing ToEffectInfo.

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Name | string | Effect name |
| Description | string | Effect description |
| Type | string | Effect type |
| NumEffectStrings | int | Number of effect strings |
| EffectString[index] | [effectstring](#effectstring) | Effect string by index |
| UsesRemaining | int | Uses remaining |

**Example Usage:**
```lavishscript
if ${Me.Effect[1].IsEffectInfoAvailable}
{
    echo ${Me.Effect[1].ToEffectInfo.Name}
    echo ${Me.Effect[1].ToEffectInfo.Description}
}
```

**See Also:** [effect](#effect), [actoreffect](#actoreffect), [effectstring](#effectstring)

---

### effectstring

Effect description string.

**Access:** `${EffectInfo.EffectString[index]}` or `${AbilityInfo.Effect[index]}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| PercentSuccess | int | Percent success chance |
| Indentation | int | Indentation level |
| Description / Desc | string | Effect description text |

**Example Usage:**
```lavishscript
echo ${MyEffect.ToEffectInfo.EffectString[1].Description}
echo ${MyAbility.ToAbilityInfo.Effect[1].Desc}
```

---

### quest

Quest information.

**Access:** `${QuestJournalWindow.ActiveQuest[index/name]}` or `${QuestJournalWindow.CompletedQuest[index/name]}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| ID | uint | Quest ID |
| Level | int | Quest level |
| Name | string | Quest name |
| Category | string | Quest category |
| CurrentZone | string | Current zone |

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| Delete | - | Deletes/abandons the quest |
| Share | - | Shares quest with group |
| MakeCurrentActiveQuest | - | Makes this the current active quest |
| MakeCurrentCompletedQuest | - | Makes this the current completed quest |

**Example Usage:**
```lavishscript
echo ${QuestJournalWindow.ActiveQuest[1].Name}
echo ${QuestJournalWindow.ActiveQuest[1].Level}
QuestJournalWindow.ActiveQuest[1]:Share
```

---

### currentquest

Currently selected quest in journal.

**Access:** `${QuestJournalWindow.CurrentQuest}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Name | [eq2text](#eq2text) | Quest name |
| Level | [eq2text](#eq2text) | Quest level |
| Category | [eq2text](#eq2text) | Quest category |
| CurrentZone | [eq2text](#eq2text) | Current zone |
| TimeStamp | [eq2text](#eq2text) | Time stamp |
| MissionGroup | [eq2text](#eq2text) | Mission group |
| Status | [eq2text](#eq2text) | Quest status |
| ExpirationTime | [eq2text](#eq2text) | Expiration time |
| Body | [eq2text](#eq2text) | Quest body text |

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| GetDetails | index | Gets quest details in specified index |

**Example Usage:**
```lavishscript
echo ${QuestJournalWindow.CurrentQuest.Name}
echo ${QuestJournalWindow.CurrentQuest.Level}
```

---

### merchandise

Item for sale from merchant.

**Access:** `${MerchantWindow.Item[index/name]}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Name | string | Item name |
| Level | int | Item level |
| Price | float | Price in currency |
| PriceString | string | Formatted price string |
| Quantity | int | Quantity available |
| MaxQuantity | int | Maximum quantity |
| StatusCost | int | Status point cost |
| IsForSale | bool | TRUE if for sale |
| LinkID | int | Link ID |
| ToLink | string | Creates clickable chat link |
| IsItemInfoAvailable | bool | TRUE if item info available |
| ToItemInfo | [iteminfo](#iteminfo) | Returns detailed item information |

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| Buy | quantity | Buys specified quantity |
| Sell | quantity | Sells to merchant |
| Examine | - | Examines the item |
| SetPrice | plat, gold, silver, copper | Sets price (vendor) |
| ListForSale | - | Lists item for sale |
| UnListForSale | - | Unlists item from sale |

**Example Usage:**
```lavishscript
echo ${MerchantWindow.Item[1].Name}
echo ${MerchantWindow.Item[1].Price}
MerchantWindow.Item[1]:Buy[1]
```

---

### consignment

Broker/consignment item.

**Access:** Via `${VendingContainer.Consignment[index/name]}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Name | string | Item name |
| Level | int | Item level |
| Price | float | Price with commission |
| Value | float | Item value |
| BasePrice | float | Price without commission |
| BasePriceString | string | Formatted base price |
| Quantity | int | Quantity |
| LinkID | int | Link ID |
| ToLink | string | Creates clickable chat link |
| IsListed | bool | TRUE if listed for sale |
| Market | string | Market name |
| SerialNumber | int64 | Item serial number |
| IsItemInfoAvailable | bool | TRUE if item info available |
| ToItemInfo | [iteminfo](#iteminfo) | Returns detailed item information |

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| Buy | quantity | Purchases specified quantity |
| Examine | - | Examines the item |
| SetPrice | price | Sets sale price |
| List | - | Lists item for sale |
| Unlist | - | Unlists item from sale |
| Remove | quantity | Removes from consignment |

**Example Usage:**
```lavishscript
echo ${VendingContainer[1].Consignment[1].Name}
echo ${VendingContainer[1].Consignment[1].Price}
```

---

### vendingcontainer

Player vendor container.

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Name | string | Container name |
| UsedCapacity | int | Used capacity |
| TotalCapacity | int | Total capacity |
| CurrentCoin | float | Current coin collected |
| TotalCoin | float | Total coin capacity |
| Market | string | Market name |
| SerialNumber | int64 | Container serial number |
| CommissionReduction | int | Commission reduction percentage |
| NumItems | int | Number of items |
| Consignment[index/name] | [consignment](#consignment) | Get consignment item |

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| TakeCoin | amount | Takes coin from vendor |
| Remove | - | Removes vendor |
| ChangeTo | - | Changes to this vendor |

**Example Usage:**
```lavishscript
echo ${VendingContainer[1].Name}
echo ${VendingContainer[1].CurrentCoin}
echo ${VendingContainer[1].NumItems}
```

---

### inboxmailmessage

Mail message in inbox.

**Access:** `${MailWindow.Message[index]}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| ID | uint | Message ID |
| Author | string | Sender name |
| Subject | string | Message subject |
| Body | string | Message body |
| Platinum | int | Platinum attached |
| Gold | int | Gold attached |
| Silver | int | Silver attached |
| Copper | int | Copper attached |
| Attachment | [iteminfo](#iteminfo) | Attached item info |

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| Open | - | Opens the message |
| Delete | - | Deletes the message |
| ReceiveAttachment | - | Receives attachments |

**Example Usage:**
```lavishscript
echo ${MailWindow.Message[1].Author}
echo ${MailWindow.Message[1].Subject}
MailWindow.Message[1]:Open
```

---

### mailmessage

Opened mail message.

**Access:** `${OpenedMailWindow.Message}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| ID | uint | Message ID |
| Author | string | Sender name |
| Subject | string | Message subject |
| Body | string | Message body |
| Platinum | int | Platinum attached |
| Gold | int | Gold attached |
| Silver | int | Silver attached |
| Copper | int | Copper attached |
| Attachment | [iteminfo](#iteminfo) | Attached item info |

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| Delete | - | Deletes the message |
| ReceiveAttachment | - | Receives attachments |

**Example Usage:**
```lavishscript
echo ${OpenedMailWindow.Message.Author}
echo ${OpenedMailWindow.Message.Body}
```

---

### recipe

Crafting recipe.

**Access:** `${Me.Recipe[index/name]}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Name | string | Recipe name |
| Level | int | Recipe level |
| ID | uint | Recipe ID |
| Technique | string | Technique required |
| Knowledge | string | Knowledge required |
| RecipeBook | string | Recipe book name |
| IsRecipeInfoAvailable | bool | TRUE if detailed info available |
| ToRecipeInfo | [recipeinfo](#recipeinfo) | Returns detailed recipe information |

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| Create | - | Creates item from recipe |
| Examine | - | Examines the recipe |
| Set | ID | Sets recipe reference by ID |

**Example Usage:**
```lavishscript
echo ${Me.Recipe[1].Name}
echo ${Me.Recipe[1].Level}

if ${Me.Recipe[1].IsRecipeInfoAvailable}
    echo ${Me.Recipe[1].ToRecipeInfo.Device}
```

**See Also:** [recipeinfo](#recipeinfo)

---

### recipeinfo

Detailed recipe information.

**Access:** `${Recipe.ToRecipeInfo}`

**Important:** Always check `Recipe.IsRecipeInfoAvailable` before accessing ToRecipeInfo.

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Description | string | Recipe description |
| Device | string | Crafting device required |
| NumClasses | int | Number of classes that can use |
| Class[index/name] | [class](#class) | Class by index or name |
| PrimaryComponentQuantityOnHand | int | Primary component quantity available |
| PrimaryComponent | [primarycomponent](#primarycomponent) | Primary component |
| BuildComponent1 | [buildcomponent](#buildcomponent) | First build component |
| BuildComponent2 | [buildcomponent](#buildcomponent) | Second build component |
| BuildComponent3 | [buildcomponent](#buildcomponent) | Third build component |
| BuildComponent4 | [buildcomponent](#buildcomponent) | Fourth build component |
| Fuel | [buildcomponent](#buildcomponent) | Fuel component |
| Product | string | Product name |
| Byproduct | string | Byproduct name |

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| ExamineProduct | - | Examines the product |

**Example Usage:**
```lavishscript
if ${Me.Recipe[1].IsRecipeInfoAvailable}
{
    echo ${Me.Recipe[1].ToRecipeInfo.Description}
    echo ${Me.Recipe[1].ToRecipeInfo.Device}
    echo ${Me.Recipe[1].ToRecipeInfo.PrimaryComponent.Name}
}
```

**See Also:** [recipe](#recipe), [primarycomponent](#primarycomponent), [buildcomponent](#buildcomponent)

---

### primarycomponent

Recipe primary component.

**Access:** `${RecipeInfo.PrimaryComponent}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Name | string | Component name |
| Length | int | Name length |
| Quantity | uint | Quantity required |
| QuantityOnHand | uint | Quantity available |

**Example Usage:**
```lavishscript
echo ${MyRecipe.ToRecipeInfo.PrimaryComponent.Name}
echo ${MyRecipe.ToRecipeInfo.PrimaryComponent.QuantityOnHand}
```

---

### buildcomponent

Recipe build component.

**Access:** `${RecipeInfo.BuildComponent1}` through `${RecipeInfo.BuildComponent4}`, `${RecipeInfo.Fuel}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Name | string | Component name |
| Quantity | uint | Quantity required |
| QuantityOnHand | uint | Quantity available |

**Example Usage:**
```lavishscript
echo ${MyRecipe.ToRecipeInfo.BuildComponent1.Name}
echo ${MyRecipe.ToRecipeInfo.Fuel.Quantity}
```

---

### crafting

Crafting session information.

**Access:** `${Crafting}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Message / M | string | Crafting message text |
| Result / R | int | Crafting result code |
| Quality / Q | int | Quality level |
| Progress / P | int | Progress level |
| ProgressMod / PM | int | Progress modifier |
| Durability / D | int | Durability level |
| DurabilityMod / DM | int | Durability modifier |
| MainIconID / MII | int | Main icon ID |
| BackdropIconID / BDII | int | Backdrop icon ID |
| GV[variable,type] | variable | Gets custom crafting variable |

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| CA | - | Clears all custom variables |
| GK | variable, index | Generates key for variable |
| SV | variable, key, value | Sets variable with key |

**Example Usage:**
```lavishscript
echo ${Crafting.Progress}
echo ${Crafting.Durability}
echo ${Crafting.Quality}
```

---

### eq2window

Base window type. Many window datatypes inherit from this type.

**Inherited By:** [journalquestwindow](#journalquestwindow), [reforgewindow](#reforgewindow), [brokerwindow](#brokerwindow), [merchantwindow](#merchantwindow), [channelerwindow](#channelerwindow), [beastlordwindow](#beastlordwindow), [radialmenuwindow](#radialmenuwindow), [mailwindow](#mailwindow), [openedmailwindow](#openedmailwindow), [travelmapwindow](#travelmapwindow)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Child[type,name] | [eq2widget](#eq2widget) | Gets child widget by type and name |
| IsVisible | bool | TRUE if window is visible |
| RootPage | [eq2uipage](#eq2uipage) | Root UI page |
| BasePage | [eq2uipage](#eq2uipage) | Base UI page |

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| Close | - | Closes the window |

**Example Usage:**
```lavishscript
if ${QuestJournalWindow.IsVisible}
    QuestJournalWindow:Close
```

---

### eq2clonewindow

Clone window type. Inherits from [eq2window](#eq2window).

**Inherits From:** [eq2window](#eq2window)

**Inherited By:** [lootwindow](#lootwindow), [choicewindow](#choicewindow), [containerwindow](#containerwindow), [rewardwindow](#rewardwindow), [examineitemwindow](#examineitemwindow), [replydialog](#replydialog), [mapwindow](#mapwindow)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| ID | int | Window ID |

Plus all members from [eq2window](#eq2window)

**Example Usage:**
```lavishscript
echo ${LootWindow.ID}
```

---

### journalquestwindow

Quest journal window. Inherits from [eq2window](#eq2window).

**Access:** `${QuestJournalWindow}`

**Inherits From:** [eq2window](#eq2window)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| NumActiveQuests | int | Number of active quests |
| NumCompletedQuests | int | Number of completed quests |
| ActiveQuest[index/name] | [quest](#quest) | Get active quest by index or name |
| CompletedQuest[index/name] | [quest](#quest) | Get completed quest by index or name |
| CurrentQuest | [currentquest](#currentquest) | Currently selected quest |

Plus all members from [eq2window](#eq2window)

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| GetActiveQuests | index | Populates index with active quests |
| GetCompletedQuests | index | Populates index with completed quests |

Plus all methods from [eq2window](#eq2window)

**Example Usage:**
```lavishscript
echo ${QuestJournalWindow.NumActiveQuests}
echo ${QuestJournalWindow.ActiveQuest[1].Name}
echo ${QuestJournalWindow.CurrentQuest.Name}
```

---

### lootwindow

Loot window. Inherits from [eq2clonewindow](#eq2clonewindow) → [eq2window](#eq2window).

**Access:** `${LootWindow}`

**Inherits From:** [eq2clonewindow](#eq2clonewindow) → [eq2window](#eq2window)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| LootSourceID | uint | Loot source actor ID |
| NumItems | int | Number of items |
| Item[index/name] | [iteminfo](#iteminfo) | Get item by index or name |
| Type | string | Loot method type |
| ItemsPage | [eq2uipage](#eq2uipage) | Items page |
| LootAllButton | [eq2button](#eq2button) | Loot all button |
| LootSelected | [eq2button](#eq2button) | Loot selected button |
| RequestSelected | [eq2button](#eq2button) | Request selected button |
| LottoDecline | [eq2button](#eq2button) | Lotto decline button |
| LeaderAssign | [eq2button](#eq2button) | Leader assign button |
| NBG_Need | [eq2button](#eq2button) | Need button |
| NBG_Greed | [eq2button](#eq2button) | Greed button |
| NBG_Decline | [eq2button](#eq2button) | NBG decline button |
| GroupMembers | [eq2dropdownbox](#eq2dropdownbox) | Group members dropdown |

Plus all members from [eq2clonewindow](#eq2clonewindow) and [eq2window](#eq2window)

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| LootItem | index, noconfirm | Loots item by index (pass TRUE for noconfirm) |
| LootAll | - | Loots all items |
| RequestAll | - | Requests all items |
| DeclineLotto | - | Declines lotto |
| SelectNeed | - | Selects need |
| SelectGreed | - | Selects greed |
| DeclineNBG | - | Declines NBG |

Plus all methods from [eq2clonewindow](#eq2clonewindow) and [eq2window](#eq2window)

**Example Usage:**
```lavishscript
echo ${LootWindow.NumItems}
echo ${LootWindow.Item[1].Name}
LootWindow:LootAll
```

---

### choicewindow

Choice dialog window. Inherits from [eq2clonewindow](#eq2clonewindow) → [eq2window](#eq2window).

**Access:** `${ChoiceWindow}`

**Inherits From:** [eq2clonewindow](#eq2clonewindow) → [eq2window](#eq2window)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Text | [eq2text](#eq2text) | Dialog text widget |
| Choice1 | string | First choice text |
| Choice2 | string | Second choice text |

Plus all members from [eq2clonewindow](#eq2clonewindow) and [eq2window](#eq2window)

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| DoChoice1 | - | Selects first choice |
| DoChoice2 | - | Selects second choice |

Plus all methods from [eq2clonewindow](#eq2clonewindow) and [eq2window](#eq2window)

**Example Usage:**
```lavishscript
echo ${ChoiceWindow.Choice1}
echo ${ChoiceWindow.Choice2}
ChoiceWindow:DoChoice1
```

---

### containerwindow

Container window. Inherits from [eq2clonewindow](#eq2clonewindow) → [eq2window](#eq2window).

**Access:** `${ContainerWindow}`

**Inherits From:** [eq2clonewindow](#eq2clonewindow) → [eq2window](#eq2window)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| NumItems | int | Number of items in container |
| Item[index/name] | [containerwindowitem](#containerwindowitem) | Get item by index or name |

Plus all members from [eq2clonewindow](#eq2clonewindow) and [eq2window](#eq2window)

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| RemoveItem | id, quantity | Removes item from container |

Plus all methods from [eq2clonewindow](#eq2clonewindow) and [eq2window](#eq2window)

**Example Usage:**
```lavishscript
echo ${ContainerWindow.NumItems}
echo ${ContainerWindow.Item[1].Name}
```

---

### containerwindowitem

Item in container window.

**Access:** `${ContainerWindow.Item[index/name]}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| ID | int | Item ID |
| LinkID | int | Link ID |
| ToLink | string | Creates clickable chat link |
| IconID | int | Icon ID |
| Name | string | Item name |
| Quantity | int | Quantity |
| Level | int | Item level |
| IsItemInfoAvailable | bool | TRUE if item info available |
| ToItemInfo | [iteminfo](#iteminfo) | Returns detailed item information |

**Example Usage:**
```lavishscript
echo ${ContainerWindow.Item[1].Name}
echo ${ContainerWindow.Item[1].Quantity}
```

---

### rewardwindow

Reward pack window. Inherits from [eq2clonewindow](#eq2clonewindow) → [eq2window](#eq2window).

**Access:** `${RewardWindow}`

**Inherits From:** [eq2clonewindow](#eq2clonewindow) → [eq2window](#eq2window)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| NumRewards | int | Number of rewards |
| Reward[index] / Reward[LinkID,index] | [reward](#reward) | Get reward by index or by LinkID,index |

Plus all members from [eq2clonewindow](#eq2clonewindow) and [eq2window](#eq2window)

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| Cancel | - | Cancels reward selection |
| AcceptReward | linkid | Accepts reward by link ID |

Plus all methods from [eq2clonewindow](#eq2clonewindow) and [eq2window](#eq2window)

**Example Usage:**
```lavishscript
echo ${RewardWindow.NumRewards}
echo ${RewardWindow.Reward[1].Name}
RewardWindow:AcceptReward[${RewardWindow.Reward[1].LinkID}]
```

---

### replydialog

NPC conversation window. Inherits from [eq2clonewindow](#eq2clonewindow) → [eq2window](#eq2window).

**Access:** `${ReplyDialog}`

**Inherits From:** [eq2clonewindow](#eq2clonewindow) → [eq2window](#eq2window)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Text | [eq2text](#eq2text) | Dialog text widget |
| Replies | [eq2list](#eq2list) | List of reply options |

Plus all members from [eq2clonewindow](#eq2clonewindow) and [eq2window](#eq2window)

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| Choose / Select | index | Selects reply by index (0-based) |

Plus all methods from [eq2clonewindow](#eq2clonewindow) and [eq2window](#eq2window)

**Example Usage:**
```lavishscript
echo ${ReplyDialog.Text}
ReplyDialog:Choose[0]
```

---

### mapwindow

Map window. Inherits from [eq2clonewindow](#eq2clonewindow) → [eq2window](#eq2window).

**Access:** `${MapWindow}`

**Inherits From:** [eq2clonewindow](#eq2clonewindow) → [eq2window](#eq2window)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| CurrentZoneShortName | string | Current zone short name |
| CurrentRealm | string | Current realm |
| MapZoneShortName | string | Map zone short name |
| MapRealm | string | Map realm |
| ZoomLevel | float | Zoom level |
| TeleportLocations | int | Number of teleport locations |
| TeleportLocation[index] | [teleportlocation](#teleportlocation) | Get teleport location by index |

Plus all members from [eq2clonewindow](#eq2clonewindow) and [eq2window](#eq2window)

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| Teleport | index | Teleports to location by index |

Plus all methods from [eq2clonewindow](#eq2clonewindow) and [eq2window](#eq2window)

**Example Usage:**
```lavishscript
echo ${MapWindow.CurrentZoneShortName}
echo ${MapWindow.TeleportLocations}
```

---

### travelmapwindow

Travel map window (Fast Travel window). Inherits from [eq2window](#eq2window).

**Access:** `${TravelMapWindow}`

**Inherits From:** [eq2window](#eq2window)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| TeleportLocations | int | Total number of teleport locations available |
| TeleportLocation[index] | [travelmapwindowlocation](#travelmapwindowlocation) | Get teleport location by index (1 to TeleportLocations) |
| Buttons | [eq2uipage](#eq2uipage) | Buttons page |

Plus all members from [eq2window](#eq2window)

**Example Usage:**
```lavishscript
EQ2:OpenTravelMapWindow
wait 10 ${TravelMapWindow(exists)}
echo ${TravelMapWindow.TeleportLocations}
echo ${TravelMapWindow.TeleportLocation[1].ZoneName}
```

**See Also:** `EQ2:OpenTravelMapWindow`

---

### teleportlocation

Teleport location on the map window.

**Access:** `${MapWindow.TeleportLocation[index]}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| ID | uint | Location ID |
| Description | string | Location description |
| Location | point3f | Teleport location coordinates |

**Example Usage:**
```lavishscript
echo ${MapWindow.TeleportLocation[1].Description}
```

---

### travelmapwindowlocation

Travel map location (fast travel).

**Access:** `${TravelMapWindow.TeleportLocation[index]}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| ID | uint | Location ID |
| LowerLevel | int | Minimum level for location |
| UpperLevel | int | Maximum level for location |
| ZoneName | string | Zone name |
| ZoneShortName | string | Zone short name |
| ZoneDescription | string | Zone description |
| Destination | point3f | Destination coordinates |
| DestinationDescription | string | Destination description |
| MapPositionX | int | X position on map |
| MapPositionY | int | Y position on map |

**Example Usage:**
```lavishscript
echo ${TravelMapWindow.TeleportLocation[1].ZoneName}
echo ${TravelMapWindow.TeleportLocation[1].DestinationDescription}
```

---

### radialmenuwindow

Radial menu window. Inherits from [eq2window](#eq2window).

**Access:** `${RadialMenuWindow}`

**Inherits From:** [eq2window](#eq2window)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| NumActions | int | Number of actions |
| Action[index] | [radialmenuaction](#radialmenuaction) | Get action by index |
| Actor | [actor](#actor) | Target actor for radial menu |

Plus all members from [eq2window](#eq2window)

**Example Usage:**
```lavishscript
echo ${RadialMenuWindow.NumActions}
echo ${RadialMenuWindow.Action[1].Label}
```

---

### radialmenuaction

Radial menu action.

**Access:** `${RadialMenuWindow.Action[index]}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Label | string | Action label text |
| Command | string | Command line |
| Verb | string | Verb |
| UnavailableReason | string | Reason why unavailable |
| Unavailable | bool | TRUE if unavailable |
| MaxRange | float | Maximum range |
| Target | [actor](#actor) | Target actor |

**Example Usage:**
```lavishscript
echo ${RadialMenuWindow.Action[1].Label}
echo ${RadialMenuWindow.Action[1].Command}
```

---

### brokerwindow

Broker search window. Inherits from [eq2window](#eq2window).

**Access:** `${BrokerWindow}`

**Inherits From:** [eq2window](#eq2window)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| NumItems | int | Number of items in search results |

Plus all members from [eq2window](#eq2window)

**Example Usage:**
```lavishscript
echo ${BrokerWindow.NumItems}
```

---

### merchantwindow

Merchant window. Inherits from [eq2window](#eq2window).

**Access:** `${MerchantWindow}`

**Inherits From:** [eq2window](#eq2window)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| NumItems | int | Number of items |
| Item[index/name] | [merchandise](#merchandise) | Get merchandise by index or name |

Plus all members from [eq2window](#eq2window)

**Example Usage:**
```lavishscript
echo ${MerchantWindow.NumItems}
echo ${MerchantWindow.Item[1].Name}
```

---

### mailwindow

Mail inbox window. Inherits from [eq2window](#eq2window).

**Access:** `${MailWindow}`

**Inherits From:** [eq2window](#eq2window)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| NumMessages | int | Number of messages in inbox |
| Message[index] | [inboxmailmessage](#inboxmailmessage) | Get message by index |

Plus all members from [eq2window](#eq2window)

**Example Usage:**
```lavishscript
echo ${MailWindow.NumMessages}
echo ${MailWindow.Message[1].Author}
```

---

### openedmailwindow

Opened mail message window. Inherits from [eq2window](#eq2window).

**Access:** `${OpenedMailWindow}`

**Inherits From:** [eq2window](#eq2window)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Message | [mailmessage](#mailmessage) | Current opened message |

Plus all members from [eq2window](#eq2window)

**Example Usage:**
```lavishscript
echo ${OpenedMailWindow.Message.Subject}
echo ${OpenedMailWindow.Message.Body}
```

---

### reforgewindow

Item reforge window. Inherits from [eq2window](#eq2window).

**Access:** `${ReforgeWindow}`

**Inherits From:** [eq2window](#eq2window)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Item | [iteminfo](#iteminfo) | Item being reforged |

Plus all members from [eq2window](#eq2window)

**Example Usage:**
```lavishscript
echo ${ReforgeWindow.Item.Name}
```

---

### examineitemwindow

Item examination window. Inherits from [eq2clonewindow](#eq2clonewindow) → [eq2window](#eq2window).

**Access:** `${ExamineItemWindow}`

**Inherits From:** [eq2clonewindow](#eq2clonewindow) → [eq2window](#eq2window)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Item | [iteminfo](#iteminfo) | Item being examined |

Plus all members from [eq2clonewindow](#eq2clonewindow) and [eq2window](#eq2window)

**Example Usage:**
```lavishscript
echo ${ExamineItemWindow.Item.Name}
```

---

### channelerwindow

Channeler class window. Inherits from [eq2window](#eq2window).

**Access:** `${ChannelerWindow}`

**Inherits From:** [eq2window](#eq2window)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| NumAbilities | int | Number of abilities |

Plus all members from [eq2window](#eq2window)

**Example Usage:**
```lavishscript
echo ${ChannelerWindow.NumAbilities}
```

---

### beastlordwindow

Beastlord class window. Inherits from [eq2window](#eq2window).

**Access:** `${BeastlordWindow}`

**Inherits From:** [eq2window](#eq2window)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| NumAbilities | int | Number of abilities |

Plus all members from [eq2window](#eq2window)

**Example Usage:**
```lavishscript
echo ${BeastlordWindow.NumAbilities}
```

---

### eq2baseobject

Base class for all EQ2 UI objects. Many UI datatypes inherit from this type.

**Inherited By:** [eq2widget](#eq2widget), [eq2datasourcecontainer](#eq2datasourcecontainer), [eq2dynamicdata](#eq2dynamicdata)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| GetProperty[name,type] | variable | Gets UI property by name and type |
| Type | string | Object type name |
| Parent | [eq2baseobject](#eq2baseobject) | Parent object |

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| SetProperty | name, value | Sets UI property |
| GetProperties | map | Gets all properties in specified map |
| SpewProperties | - | Outputs all properties to console |

**Example Usage:**
```lavishscript
echo ${EQ2UIPage[MainHUD].Type}
EQ2UIPage[MainHUD]:SpewProperties
```

---

### eq2datasourcecontainer

UI data source container. Inherits from [eq2baseobject](#eq2baseobject).

**Access:** `${EQ2DataSourceContainer[name]}`

**Inherits From:** [eq2baseobject](#eq2baseobject)

**Note:** This TLO is deprecated. Use `${Me.GetGameData}` instead.

---

### eq2dynamicdata

UI dynamic data. Inherits from [eq2baseobject](#eq2baseobject).

**Access:** Via `${Me.GetGameData[string]}`

**Inherits From:** [eq2baseobject](#eq2baseobject)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Label | string | Full label text |
| ShortLabel | string | Short label text |
| Percent | float | Percentage value |

Plus all members from [eq2baseobject](#eq2baseobject)

**Example Usage:**
```lavishscript
echo ${Me.GetGameData[Self.Experience].Label}
echo ${Me.GetGameData[Self.Experience].Percent}
```

---

### eq2widget

Base UI widget. Inherits from [eq2baseobject](#eq2baseobject). Many UI widget types inherit from this type.

**Inherits From:** [eq2baseobject](#eq2baseobject)

**Inherited By:** [eq2button](#eq2button), [eq2text](#eq2text), [eq2icon](#eq2icon), [eq2textbox](#eq2textbox), [eq2scrollbar](#eq2scrollbar), [eq2sliderbar](#eq2sliderbar), [eq2dropdownbox](#eq2dropdownbox), [eq2list](#eq2list), [eq2listbox](#eq2listbox), [eq2progressbar](#eq2progressbar), [eq2checkbox](#eq2checkbox), [eq2uipage](#eq2uipage), [eq2iconbank](#eq2iconbank)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| IsEnabled | bool | TRUE if widget is enabled |

Plus all members from [eq2baseobject](#eq2baseobject)

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| LeftClick | - | Simulates left mouse click |
| RightClick | - | Simulates right mouse click |
| MiddleClick | - | Simulates middle mouse click |
| DoubleLeftClick | - | Simulates double left click |

Plus all methods from [eq2baseobject](#eq2baseobject)

**Example Usage:**
```lavishscript
EQ2UIPage[MainHUD].Child[button,MyButton]:LeftClick
```

---

### eq2button

Button widget. Inherits from [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject).

**Inherits From:** [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Text | string | Button text |

Plus all members from [eq2widget](#eq2widget) and [eq2baseobject](#eq2baseobject)

**Example Usage:**
```lavishscript
echo ${LootWindow.LootAllButton.Text}
LootWindow.LootAllButton:LeftClick
```

---

### eq2text

Text display widget. Inherits from [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject).

**Inherits From:** [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject)

Plus all members from [eq2widget](#eq2widget) and [eq2baseobject](#eq2baseobject)

**Example Usage:**
```lavishscript
echo ${ChoiceWindow.Text}
```

---

### eq2icon

Icon widget. Inherits from [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject).

**Inherits From:** [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| IconID | int | Icon ID |
| NodeID | uint | Node ID |
| ToAbility | [ability](#ability) | Converts to ability datatype |
| IsReady | bool | TRUE if ability is ready |
| PercentUndimmed | float | Percent undimmed (cooldown indicator) |

Plus all members from [eq2widget](#eq2widget) and [eq2baseobject](#eq2baseobject)

**Example Usage:**
```lavishscript
echo ${EQ2UIPage[MainHUD].Child[icon,Ability1].IconID}
```

---

### eq2textbox

Text box widget. Inherits from [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject).

**Inherits From:** [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject)

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| AppendText | text | Appends text to text box |

Plus all members and methods from [eq2widget](#eq2widget) and [eq2baseobject](#eq2baseobject)

---

### eq2scrollbar

Scrollbar widget. Inherits from [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject).

**Inherits From:** [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| AttachedControl | [eq2widget](#eq2widget) | Attached control widget |
| ThumbPosition | int | Thumb position |
| ThumbSize | int | Thumb size |
| CanScrollUp | bool | TRUE if can scroll up |
| CanScrollDown | bool | TRUE if can scroll down |

Plus all members from [eq2widget](#eq2widget) and [eq2baseobject](#eq2baseobject)

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| ScrollUp | - | Scrolls up |
| ScrollDown | - | Scrolls down |

Plus all methods from [eq2widget](#eq2widget) and [eq2baseobject](#eq2baseobject)

---

### eq2sliderbar

Slider bar widget. Inherits from [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject).

**Inherits From:** [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject)

Plus all members and methods from [eq2widget](#eq2widget) and [eq2baseobject](#eq2baseobject)

---

### eq2dropdownbox

Dropdown box widget. Inherits from [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject).

**Inherits From:** [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Label | string | Selected label text |

Plus all members from [eq2widget](#eq2widget) and [eq2baseobject](#eq2baseobject)

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| Set | index | Sets selection by index |
| GetOptions | index | Gets all options in specified index |

Plus all methods from [eq2widget](#eq2widget) and [eq2baseobject](#eq2baseobject)

---

### eq2list

List widget. Inherits from [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject).

**Inherits From:** [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject)

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| HighlightRow | index | Highlights specified row |
| GetOptions | index | Gets all options in specified index |

Plus all members and methods from [eq2widget](#eq2widget) and [eq2baseobject](#eq2baseobject)

---

### eq2listbox

List box widget. Inherits from [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject).

**Inherits From:** [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Label | string | Selected label text |

Plus all members from [eq2widget](#eq2widget) and [eq2baseobject](#eq2baseobject)

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| GetOptions | index | Gets all options in specified index |

Plus all methods from [eq2widget](#eq2widget) and [eq2baseobject](#eq2baseobject)

---

### eq2progressbar

Progress bar widget. Inherits from [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject).

**Inherits From:** [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject)

Plus all members and methods from [eq2widget](#eq2widget) and [eq2baseobject](#eq2baseobject)

---

### eq2checkbox

Checkbox widget. Inherits from [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject).

**Inherits From:** [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject)

Plus all members and methods from [eq2widget](#eq2widget) and [eq2baseobject](#eq2baseobject)

---

### eq2uipage

UI page container. Inherits from [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject).

**Access:** `${EQ2UIPage[name]}`

**Inherits From:** [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject)

**Inherited By:** [eq2composite](#eq2composite), [eq2tabbedpane](#eq2tabbedpane)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| NumChildren | int | Number of child widgets |
| ChildType[index] | string | Child type name by index |
| Child[index/name] / Child[instance,name] | [eq2widget](#eq2widget) | Get child widget by index/name or instance,name |

Plus all members from [eq2widget](#eq2widget) and [eq2baseobject](#eq2baseobject)

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| SpewChildren | - | Lists all children to console |

Plus all methods from [eq2widget](#eq2widget) and [eq2baseobject](#eq2baseobject)

**Example Usage:**
```lavishscript
echo ${EQ2UIPage[MainHUD].NumChildren}
EQ2UIPage[MainHUD]:SpewChildren
echo ${EQ2UIPage[MainHUD].Child[button,MyButton].Text}
```

---

### eq2composite

Composite UI element. Inherits from [eq2uipage](#eq2uipage) → [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject).

**Inherits From:** [eq2uipage](#eq2uipage) → [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject)

Plus all members and methods from [eq2uipage](#eq2uipage), [eq2widget](#eq2widget), and [eq2baseobject](#eq2baseobject)

---

### eq2tabbedpane

Tabbed pane widget. Inherits from [eq2uipage](#eq2uipage) → [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject).

**Inherits From:** [eq2uipage](#eq2uipage) → [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject)

Plus all members and methods from [eq2uipage](#eq2uipage), [eq2widget](#eq2widget), and [eq2baseobject](#eq2baseobject)

---

### eq2iconbank

Icon bank (ability bars, etc.). Inherits from [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject).

**Inherits From:** [eq2widget](#eq2widget) → [eq2baseobject](#eq2baseobject)

#### Members

| Member | Type | Description |
|--------|------|-------------|
| NumIcons | int | Number of icons |
| Icon[index] | [eq2icon](#eq2icon) | Get icon by index |

Plus all members from [eq2widget](#eq2widget) and [eq2baseobject](#eq2baseobject)

**Example Usage:**
```lavishscript
echo ${EQ2UIPage[MainHUD].Child[iconbank,Abilities].NumIcons}
```

---

## End of API Reference

This comprehensive API reference documents all datatypes, members, methods, and relationships available in the ISXEQ2 extension for EverQuest 2. For command reference, event documentation, and usage examples, please refer to the official ISXEQ2 Reference document.

**Key Points to Remember:**

1. Always check for NULL with `${Object(exists)}`
2. Check async availability before accessing detailed info (IsItemInfoAvailable, IsAbilityInfoAvailable, etc.)
3. Understand inheritance chains (char → actor, groupmember → actor, etc.)
4. Use GetGameData for experience, vitality, pet info, and bank coins
5. All TLOs are case-insensitive
6. Query syntax supports complex filtering with operators
7. Events require registration via LavishScript Event system

**For Additional Information:**
- Commands: See Commands section in official reference
- Events: See Events section in official reference
- Usage Examples: See Usage Examples section in official reference
- Deprecated Features: See Deprecated Features section in official reference
