# ISXEVE Extension Architecture
## The Bridge Between Scripts and EVE Online

**Purpose**: Deep understanding of ISXEVE - the extension that makes EVE automation possible
**Audience**: AI systems developing EVE bots
**Wiki Reference**: IsxeveWiki/ folder for complete API documentation

---

## Table of Contents

1. [What Is ISXEVE](#what-is-isxeve)
2. [How ISXEVE Works](#how-isxeve-works)
3. [Core Object Hierarchy](#core-object-hierarchy)
4. [Object Types and TLOs](#object-types-and-tlos)
5. [Members vs Methods](#members-vs-methods)
6. [Entity System](#entity-system)
7. [UI Interaction](#ui-interaction)
8. [Event System](#event-system)
9. [Common Patterns](#common-patterns)
10. [Limitations and Constraints](#limitations-and-constraints)

---

## What Is ISXEVE

### Definition

**ISXEVE** = **I**nner**S**pace E**X**tension for **EVE** Online

**Purpose**:
- Reads EVE client memory
- Exposes game data as LavishScript objects
- Provides methods to interact with game
- Handles EVE-specific logic

**NOT ISXEVE**:
- Does NOT inject fake data into game
- Does NOT modify game memory (read-only)
- Does NOT send network packets
- Does NOT bypass server authority

**What ISXEVE Actually Does**:
```
EVE Client Memory (Game State)
    ↓ ISXEVE Reads ↓
LavishScript Objects (Bot-Accessible Data)
    ↓ Bot Script Logic ↓
EVE UI Commands (Mouse/Keyboard Simulation)
    ↓ EVE Client Processes ↓
Server Commands (Normal Game Protocol)
```

### Why ISXEVE Is Necessary

**Without ISXEVE**:
- Would need to OCR (read pixels) to see game state
- Would need complex image recognition
- Would be extremely slow and fragile
- Would be human-level reaction times only

**With ISXEVE**:
- Instant access to all game data
- Precise coordinates, IDs, values
- Millisecond reaction times
- Structured, reliable data

**Example Comparison**:
```lavishscript
; WITHOUT ISXEVE (impossible in LavishScript):
; 1. Take screenshot
; 2. Run image recognition on shield bar
; 3. Estimate shield % from pixel colors
; 4. Hope it's accurate
;
; WITH ISXEVE (actual code):
variable float ShieldPct = ${MyShip.ShieldPct}
; Instant, precise, reliable
```

---

## How ISXEVE Works

### Memory Reading

**The Process**:
1. EVE Client stores game state in memory (C++ objects)
2. ISXEVE knows memory layouts (reverse-engineered)
3. ISXEVE reads memory addresses
4. ISXEVE creates LavishScript objects from memory data
5. Your script accesses these objects

**Memory Offsets**:
```
EVE Memory Address 0x12345678:
    ShipID: 1000000001234
    ShipName: "Dominix"
    ShieldHP: 8542.5
    ArmorHP: 12000.0
    ...

ISXEVE Reads This:
    ${MyShip.ID} = 1000000001234
    ${MyShip.ToEntity.Name} = "Dominix"
    ${MyShip.Shield} = 8542.5
    ${MyShip.Armor} = 12000.0
```

**Patch Resistance**:
- CCP patches EVE regularly
- Memory layouts change
- ISXEVE must be updated for each patch
- **Your scripts don't need updates** (ISXEVE abstracts the changes)

### Object Creation

**ISXEVE Registers Object Types**:
```cpp
// In ISXEVE C++ code (simplified):
RegisterTopLevelObject("Me", &GetMeObject);
RegisterTopLevelObject("MyShip", &GetMyShipObject);
RegisterTopLevelObject("EVE", &GetEVEObject);
```

**LavishScript Can Then Use**:
```lavishscript
; These "magically" exist because ISXEVE registered them
${Me}        ; Character object
${MyShip}    ; Ship object
${EVE}       ; EVE client object
```

### Update Cycle

**ISXEVE Updates**:
- ISXEVE reads memory every frame (60+ times per second)
- Caches data to reduce memory reads
- Invalidates cache when things change

**Script Access**:
```lavishscript
; When you access ${MyShip.Shield}:
; 1. ISXEVE checks cache (is it still valid?)
; 2. If cache invalid, read memory
; 3. If cache valid, return cached value
; 4. Return value to script

; This happens in microseconds
```

---

## Core Object Hierarchy

### Top-Level Objects (TLOs)

**What Are TLOs**:
- **Top-Level Objects** = objects accessible directly (not through another object)
- Start with `${}` syntax
- Provided by ISXEVE extension

**Most Important TLOs**:

```lavishscript
${Me}                   ; Your character
${MyShip}               ; Your ship
${EVE}                  ; EVE client/game state
${Entity[ID]}           ; Entity by ID
${Entity[Name]}         ; Entity by name
${Station}              ; Current station (if docked)
${Local}                ; Local chat
${Config}               ; ISXEVE configuration
${ISXEVE}               ; ISXEVE extension object
```

**Usage Examples**:
```lavishscript
; Character name
echo "${Me.Name}"

; Ship name
echo "${MyShip.ToEntity.Name}"

; Current solar system
echo "${Me.SolarSystemID}"

; Capacitor percentage
echo "${MyShip.CapacitorPct}%"
```

### Object Chaining

**Members Return Objects**:
```lavishscript
; ${Me} is an object
; ${Me.ToEntity} returns an entity object
; ${Me.ToEntity.Name} returns a string object

variable entity MyEntity = ${Me.ToEntity}
variable string MyName = ${MyEntity.Name}

; Or chain:
variable string MyName = ${Me.ToEntity.Name}
```

**Deep Chains**:
```lavishscript
; Get active target's owner's alliance name
${MyShip.ActiveTarget.ToEntity.Owner.Alliance.Name}

; Breakdown:
; MyShip                   -> ship object
; .ActiveTarget            -> module object
; .ToEntity                -> entity object
; .Owner                   -> owner object
; .Alliance                -> alliance object
; .Name                    -> string
```

### Type System

**Every Object Has A Type**:
```lavishscript
; Check type
echo "${Me.Type}"           ; Outputs: "me"
echo "${MyShip.Type}"       ; Outputs: "ship"
echo "${Entity[123].Type}"  ; Outputs: "entity"

; Type checking
if ${MyShip.Type.Equal["ship"]}
{
    echo "Yes, it's a ship"
}
```

**Common Types**:
- `me` - Character
- `ship` - Ship
- `entity` - Entity in space
- `item` - Inventory item
- `module` - Ship module
- `bookmark` - Bookmark
- `agent` - Agent (mission)
- `window` - UI window

---

## Object Types and TLOs

### The "me" Object

**${Me}** = Your character

**Key Members**:
```lavishscript
${Me.Name}              ; Character name (string)
${Me.CharID}            ; Character ID (int64)
${Me.CorporationID}     ; Corp ID (int)
${Me.AllianceID}        ; Alliance ID (int)
${Me.InSpace}           ; In space? (bool)
${Me.InStation}         ; In station? (bool)
${Me.SolarSystemID}     ; Current system ID (int)
${Me.ToEntity}          ; Character as entity (entity)
${Me.Wallet}            ; ISK balance (int64)
```

**Methods**:
```lavishscript
Me.GetTargets           ; Get index of locked targets
Me.GetFleetMembers      ; Get index of fleet members
```

**Example Usage**:
```lavishscript
if ${Me.InSpace}
{
    echo "${Me.Name} is in space in system ${Me.SolarSystemID}"
}
else if ${Me.InStation}
{
    echo "${Me.Name} is docked"
}
```

### The "ship" Object

**${MyShip}** = Your current ship

**Critical Members**:
```lavishscript
${MyShip.ID}                    ; Ship entity ID (int64)
${MyShip.ToEntity}              ; Ship as entity (entity)
${MyShip.Shield}                ; Current shield HP (float)
${MyShip.ShieldPct}             ; Shield % (float)
${MyShip.Armor}                 ; Current armor HP (float)
${MyShip.ArmorPct}              ; Armor % (float)
${MyShip.Structure}             ; Current structure HP (float)
${MyShip.StructurePct}          ; Structure % (float)
${MyShip.Capacitor}             ; Current cap (float)
${MyShip.CapacitorPct}          ; Cap % (float)
${MyShip.MaxTargetRange}        ; Max targeting range (float)
${MyShip.MaxLockedTargets}      ; Max lockable targets (int)
```

**Cargo Access** (⚠️ Modern API - use EVEWindow[Inventory]):
```lavishscript
; MODERN (July 2020+): Use inventory windows
${EVEWindow[Inventory].Child[ShipCargo].FreeSpace}   ; Free cargo space
${EVEWindow[Inventory].Child[ShipCargo].Capacity}    ; Total cargo capacity
${EVEWindow[Inventory].Child[ShipCargo].UsedSpace}   ; Used cargo space

; DEPRECATED (legacy reference only):
; ${MyShip.CargoFreeSpace}, ${MyShip.CargoCapacity} removed July 2020
; See File 13 for modern inventory API patterns
```

**Module Access**:
```lavishscript
${MyShip.Module[ID]}            ; Module by ID
${MyShip.Module[Index]}         ; Module by slot index
${MyShip.Module[TypeID,Group]}  ; Module by type and group
```

**Methods**:
```lavishscript
MyShip:Open[]                   ; Open cargo
MyShip:GetModules[]             ; Get index of modules
MyShip:GetCargo[]               ; Get index of cargo items
MyShip:SetActiveTarget[]        ; Set active target
MyShip:LaunchAllDrones[]        ; Launch all drones
```

### The "EVE" Object

**${EVE}** = EVE client state

**Critical Members**:
```lavishscript
${EVE.Bookmark[ID]}             ; Bookmark by ID
${EVE.Bookmark[Name]}           ; Bookmark by name
${EVE.Station[ID]}              ; Station by ID
```

**Critical Methods**:
```lavishscript
EVE:QueryEntities[]             ; Query entities (MOST USED)
EVE:Execute[]                   ; Execute EVE command
EVE:GetBookmarks[]              ; Get index of bookmarks
```

**EVE:Execute Examples**:
```lavishscript
; Dock at station
EVE:Execute[CmdExitStation]     ; Undock

; UI commands
EVE:Execute[OpenShipHangar]     ; Open ship hangar
EVE:Execute[OpenCargoHold]      ; Open cargo
```

### The "entity" Object

**Most Important Object Type** (covered in detail later)

**${Entity[]}** = Access entity by ID or name

```lavishscript
variable entity Target = ${Entity[1234567890]}
variable entity Station = ${Entity["Jita IV - Moon 4 - Caldari Navy Assembly Plant"]}

; Entity members
${Target.ID}                    ; Unique ID (int64)
${Target.Name}                  ; Name (string)
${Target.Distance}              ; Distance in meters (float)
${Target.IsLockedTarget}        ; Locked by me? (bool)
${Target.IsNPC}                 ; Is this an NPC? (bool)
${Target.GroupID}               ; Group ID (int)
${Target.CategoryID}            ; Category ID (int)
```

---

## Members vs Methods

### Syntax Distinction

**Get Value (Member)**:
```lavishscript
; Use ${} to GET a value
variable float Shield = ${MyShip.Shield}
echo "Shield: ${MyShip.Shield}"
```

**Call Method (Action)**:
```lavishscript
; Use : to CALL a method
Entity:LockTarget[]
MyShip:Open[]
Module:Activate[]
```

### Members (Properties)

**Characteristics**:
- Read-only (usually)
- Return data from game state
- No side effects
- Fast (memory read)

**Examples**:
```lavishscript
${Me.Name}                  ; Read character name
${MyShip.Shield}            ; Read shield HP
${Entity.Distance}          ; Read distance
${Module.IsActive}          ; Read module state
```

**Some Members Take Parameters**:
```lavishscript
${MyShip.Module[1]}         ; Get module in slot 1
${Entity["Frigate"]}        ; Get entity by name
${Math.Calc[5 + 3]}         ; Calculate expression
```

### Methods (Actions)

**Characteristics**:
- Perform action in game
- Can modify game state
- May have side effects
- Take time to execute (server delay)

**Examples**:
```lavishscript
Entity:LockTarget[]                 ; Lock entity
Entity:Approach[]                   ; Approach entity
Entity:Orbit[5000]                  ; Orbit at 5km
Module:Activate[]                   ; Activate module
Module:Deactivate[]                 ; Deactivate module
MyShip:SetActiveTarget[ID]          ; Set active target
```

### Return Values

**Methods Can Return Values**:
```lavishscript
; Some methods return bool (success/failure)
if ${Entity:LockTarget[]}
{
    echo "Lock command sent successfully"
}
else
{
    echo "Lock command failed"
}
```

**But Most Don't**:
```lavishscript
; Most methods return nothing, just execute
Entity:Approach[]
; No return value to check

; Must verify action worked via members:
wait 10
if ${Entity.Distance} < ${OldDistance}
{
    echo "Approach working"
}
```

---

## Entity System

### What Are Entities

**Entity** = Anything in space with coordinates

**Entity Types**:
- Player ships
- NPC ships
- Asteroids
- Wrecks
- Stations
- Stargates
- Planets
- Containers
- Drones
- Missiles (projectiles)

**ALL are entity objects**

### Entity Identification

**Unique ID**:
```lavishscript
; Every entity has unique int64 ID
variable int64 EntityID = ${SomeEntity.ID}

; Can retrieve entity by ID later
variable entity Same = ${Entity[${EntityID}]}
```

**GroupID and CategoryID**:
```lavishscript
; GroupID = specific type (e.g., "Asteroid Belt", "Battleship", "Stargate")
; CategoryID = broad category (e.g., CATEGORY_ENTITY, CATEGORY_CELESTIAL)

; Example IDs (from EVE static data):
; GroupID 15 = Station
; GroupID 10 = Stargate
; GroupID 25 = Frigate
; CategoryID 11 = Ship
; CategoryID 2 = Celestial
```

**Filtering Entities**:
```lavishscript
; Query all NPCs
variable index:entity NPCs
EVE:QueryEntities[NPCs, "CategoryID = CATEGORY_ENTITY && IsNPC"]

; Query all asteroids
variable index:entity Roids
EVE:QueryEntities[Roids, "GroupID = GROUP_ASTEROID"]

; Query all wrecks
variable index:entity Wrecks
EVE:QueryEntities[Wrecks, "GroupID = GROUP_WRECK"]
```

### QueryEntities (Critical Method)

**Syntax**:
```lavishscript
EVE:QueryEntities[OutputIndex, "FilterExpression"]
```

**Parameters**:
- `OutputIndex`: index:entity variable to fill with results
- `FilterExpression`: string expression to filter entities

**Filter Syntax**:
```lavishscript
; Simple filter
"IsNPC"                     ; All NPCs

; Comparison
"Distance < 50000"          ; Within 50km

; Logical operators
"IsNPC && Distance < 50000" ; NPCs within 50km
"IsNPC || IsPC"             ; NPCs or players

; Grouping
"(IsNPC || IsPC) && Distance < 100000"

; Constants
"CategoryID = CATEGORY_ENTITY"
"GroupID = GROUP_WRECK"
```

**Common Filters**:
```lavishscript
; Hostiles (NPCs or unfriendly players)
"IsNPC || (IsPC && !IsFleetMember)"

; Lockable targets
"!IsLockedTarget && Distance < ${MyShip.MaxTargetRange}"

; Lootable wrecks
"GroupID = GROUP_WRECK && Distance < 100000"

; Minable asteroids
"GroupID = GROUP_ASTEROID && Distance < 50000"
```

**Example**:
```lavishscript
; Get all NPCs within 100km
variable index:entity NPCs
EVE:QueryEntities[NPCs, "IsNPC && Distance < 100000"]

; Iterate through results
variable iterator NPC
NPCs:GetIterator[NPC]

if ${NPC:First(exists)}
{
    do
    {
        echo "NPC: ${NPC.Value.Name} at ${NPC.Value.Distance}m"
    }
    while ${NPC:Next(exists)}
}
```

### Entity Members (Comprehensive)

**Identification**:
```lavishscript
${Entity.ID}                ; Unique ID (int64)
${Entity.Name}              ; Name (string)
${Entity.TypeID}            ; Type ID (int)
${Entity.GroupID}           ; Group ID (int)
${Entity.CategoryID}        ; Category ID (int)
```

**Position**:
```lavishscript
${Entity.X}                 ; X coordinate (float)
${Entity.Y}                 ; Y coordinate (float)
${Entity.Z}                 ; Z coordinate (float)
${Entity.Distance}          ; Distance from me (float)
```

**State**:
```lavishscript
${Entity.IsLockedTarget}    ; Am I locking this? (bool)
${Entity.IsTargetingMe}     ; Is it targeting me? (bool)
${Entity.IsNPC}             ; Is this an NPC? (bool)
${Entity.IsPC}              ; Is this a player? (bool)
${Entity.IsMoribund}        ; Is it exploding/dead? (bool)
${Entity.IsWarpingAway}     ; Is it warping away? (bool)
```

**Ownership**:
```lavishscript
${Entity.Owner}             ; Owner object
${Entity.OwnerID}           ; Owner ID (int)
${Entity.Owner.Name}        ; Owner name (string)
${Entity.Owner.CorpID}      ; Owner corp ID (int)
${Entity.Owner.AllianceID}  ; Owner alliance ID (int)
```

**Combat**:
```lavishscript
${Entity.Shield}            ; Current shield (float)
${Entity.ShieldPct}         ; Shield % (float)
${Entity.Armor}             ; Current armor (float)
${Entity.ArmorPct}          ; Armor % (float)
```

### Entity Methods (Critical Actions)

**Targeting**:
```lavishscript
Entity:LockTarget[]         ; Lock this entity
Entity:UnlockTarget[]       ; Unlock this entity
Entity:MakeActiveTarget[]   ; Set as active target
```

**Movement**:
```lavishscript
Entity:Approach[]           ; Approach entity
Entity:Orbit[distance]      ; Orbit at distance (meters)
Entity:KeepAtRange[distance]; Keep at distance
Entity:WarpTo[distance]     ; Warp to entity at distance
Entity:Dock[]               ; Dock (if station/structure)
```

**Interaction**:
```lavishscript
Entity:Open[]               ; Open (wreck, container, etc.)
Entity:LootAll[]            ; Loot everything
Entity:Activate[]           ; Activate (gate, etc.)
```

---

## UI Interaction

### EVE:Execute Command

**Purpose**: Execute EVE UI commands

**Common Commands**:
```lavishscript
EVE:Execute[CmdExitStation]             ; Undock
EVE:Execute[OpenShipHangar]             ; Open ship hangar
EVE:Execute[OpenCargoHold]              ; Open cargo hold
EVE:Execute[OpenDroneBay]               ; Open drone bay
EVE:Execute[CmdDronesReturnToBay]       ; Return drones
EVE:Execute[CmdDronesEngage]            ; Engage target with drones
EVE:Execute[CmdStopShip]                ; Stop ship
```

**Full List**:
- See IsxeveWiki for complete EVE:Execute command list
- Hundreds of commands available

### Windows

**Window Object**:
```lavishscript
; Access UI windows
variable index:window Windows
EVE:GetWindows[Windows]

; Check if specific window open
if ${Window["Local"](exists)}
{
    echo "Local chat is open"
}
```

**Common Window Names**:
- "Local" - Local chat
- "Inventory" - Inventory window
- "Ship Hangar" - Ship hangar
- "Market" - Market window

---

## Event System

### ISXEVE Events

**What Are Events**:
- Notifications when things happen in game
- Your script can subscribe (listen) to events
- Event fires → your function/atom called

**Common Events**:
```lavishscript
; On entity appeared
Event[OnFrame]:AttachAtom[OnFrameHandler]

; On target changed
Event[OnActiveTargetChanged]:AttachAtom[OnTargetChanged]
```

**Example**:
```lavishscript
; Define atom to handle event
atom OnTargetChanged()
{
    echo "Active target changed to: ${MyShip.ActiveTarget.ToEntity.Name}"
}

; Attach atom to event
Event[OnActiveTargetChanged]:AttachAtom[OnTargetChanged]

; Later: Detach
Event[OnActiveTargetChanged]:DetachAtom[OnTargetChanged]
```

**Available Events**:
- OnFrame - Every frame update
- OnActiveTargetChanged - Active target changed
- OnMyShipShieldsUpdate - Shield HP changed
- Many more (see IsxeveWiki)

---

## Common Patterns

### Existence Checking

**CRITICAL Pattern**:
```lavishscript
; ALWAYS check if object exists before using
if ${Entity[${TargetID}](exists)}
{
    ; Safe to use
    echo "${Entity[${TargetID}].Name}"
}
else
{
    echo "Entity doesn't exist (dead/left grid)"
}
```

**Why Necessary**:
- Entities can despawn (NPCs die, ships warp, etc.)
- Accessing non-existent object = crash or error
- Must ALWAYS check (exists) for dynamic objects

### Iterator Pattern

**Iterate Through Index**:
```lavishscript
variable index:entity Entities
EVE:QueryEntities[Entities, "IsNPC"]

variable iterator Entity
Entities:GetIterator[Entity]

if ${Entity:First(exists)}
{
    do
    {
        ; Process each entity
        echo "NPC: ${Entity.Value.Name}"

        ; Check if still exists (could despawn during iteration)
        if !${Entity.Value(exists)}
        {
            echo "Entity despawned during iteration!"
            continue
        }

        ; Do something with entity
        Entity.Value:LockTarget[]
    }
    while ${Entity:Next(exists)}
}
```

### Distance Checking

**Before Approaching**:
```lavishscript
; Check distance before approaching
if ${Entity.Distance} > 2500
{
    Entity:Approach[]
    wait 10

    ; Wait until in range
    while ${Entity.Distance} > 2500 && ${Entity(exists)}
    {
        wait 10
    }
}

if ${Entity.Distance} <= 2500
{
    ; In range, do action
    Entity:Open[]
}
```

---

## Limitations and Constraints

### What ISXEVE Cannot Do

**Server Authority**:
- Cannot fake actions to server
- Cannot teleport ship
- Cannot spawn items
- Cannot change standings
- Cannot bypass game rules

**UI Limitations**:
- Cannot read text from UI labels directly (in most cases)
- Cannot easily detect all UI states
- Some windows hard to interact with

**Timing**:
- No control over server response times
- Cannot force instant actions
- Must wait for server confirmation

### What ISXEVE Can Do

**Reading**:
- All entity data (positions, IDs, states)
- All ship data (HP, cap, modules)
- All inventory data
- Character data (wallet, skills, etc.)
- UI state (some windows)

**Actions**:
- All normal player actions (via UI commands)
- Lock, shoot, move, loot, dock, etc.
- Anything a player can do

**The Rule**:
> If a player can do it by clicking, ISXEVE can do it by scripting

---

## Next Steps

**Now you understand ISXEVE. Next:**
- Read `04_LavishScript_Language_Complete_Reference.md` for language mastery
- Read `09_ISXEVE_Core_Objects_Reference.md` for detailed object documentation
- Study example scripts to see ISXEVE in action

**You should now understand**:
- ISXEVE reads memory, provides objects
- Objects have members (data) and methods (actions)
- Entity system is core to EVE automation
- QueryEntities is critical for finding targets
- Must always check (exists) for dynamic objects
- EVE:Execute runs UI commands
- Everything follows server authority (no cheating)

---

**END OF FILE**
**Next File**: 04_LavishScript_Language_Complete_Reference.md
