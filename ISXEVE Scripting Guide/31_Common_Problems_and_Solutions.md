# File 31: Common Problems and Solutions

**Layer 7: Advanced Topics - Part 4 of 4 (FINAL LAYER 7 FILE!)**

## Table of Contents
1. [Entity and Targeting Problems](#entity-problems)
2. [Module and Ship Control Issues](#module-issues)
3. [Navigation and Movement Problems](#navigation-problems)
4. [Inventory and Cargo Issues](#inventory-issues)
5. [Fleet and Relay Problems](#fleet-problems)
6. [Performance and Memory Issues](#performance-issues)
7. [Configuration Problems](#config-problems)
8. [ISXEVE Bugs and Limitations](#isxeve-bugs)
9. [Common Scripting Mistakes](#scripting-mistakes)
10. [Recovery and Failsafe Strategies](#recovery-strategies)
11. [Debugging Workflows](#debugging-workflows)
12. [Production Deployment Issues](#deployment-issues)

---

## Entity and Targeting Problems <a name="entity-problems"></a>

### Problem 1: Entity Doesn't Exist After Query

**Symptom:**
```lavish
EVE:QueryEntities[Targets, "CategoryID = CATEGORYID_ENTITY"]
Entity[${Targets.Get[1]}]:LockTarget    ; ERROR: Entity doesn't exist!
```

**Cause:** Entity despawned between query and action (asteroid mined out, NPC killed, warp away)

**Solution:**

```lavish
; ALWAYS validate before acting
method LockEntity(int64 entityID)
{
    ; Validate exists
    if !${Entity[${entityID}](exists)}
    {
        Logger:Log["Entity ${entityID} no longer exists"]
        return FALSE
    }

    ; Validate ID is still valid
    if ${Entity[${entityID}].ID} == 0
    {
        Logger:Log["Entity ${entityID} has ID=0 (despawned)"]
        return FALSE
    }

    ; Now safe to lock
    Entity[${entityID}]:LockTarget
    return TRUE
}
```

### Problem 2: Target Lock Fails Silently

**Symptom:** Call `LockTarget` but target never locks

**Causes:**
- Already at max locked targets
- Target out of range
- Target not targetable (structure, etc.)
- Ship ECM jammed

**Solution:**

```lavish
method SafeLockTarget(int64 targetID)
{
    ; Check target count
    if ${Me.TargetCount} >= ${Me.MaxLockedTargets}
    {
        Logger:Log["At max targets (${Me.TargetCount}/${Me.MaxLockedTargets})"]
        return FALSE
    }

    ; Validate entity
    if !${Entity[${targetID}](exists)}
    {
        Logger:Log["Entity doesn't exist"]
        return FALSE
    }

    ; Check already locked/targeting
    if ${Entity[${targetID}].IsLockedTarget}
    {
        return TRUE    ; Already locked
    }

    if ${Entity[${targetID}].BeingTargeted}
    {
        Logger:Log["Already targeting, waiting..."]
        return FALSE    ; Wait for lock to complete
    }

    ; Check range
    if ${Entity[${targetID}].Distance} > ${MyShip.MaxTargetRange}
    {
        Logger:Log["Out of range: ${Entity[${targetID}].Distance}m > ${MyShip.MaxTargetRange}m"]
        return FALSE
    }

    ; Attempt lock
    Entity[${targetID}]:LockTarget

    ; Wait briefly and verify
    wait 5

    if ${Entity[${targetID}].BeingTargeted}
    {
        return TRUE    ; Lock initiated successfully
    }

    Logger:Log["Lock failed - possible jam or targeting error"]
    return FALSE
}
```

### Problem 3: QueryEntities Returns Too Many Results

**Symptom:** Script lags when querying entities in busy systems

**Cause:** Inefficient query - returns thousands of entities

**Solution:**

```lavish
; BAD - returns ALL entities
EVE:QueryEntities[AllStuff]    ; Thousands of results!

; GOOD - specific query
EVE:QueryEntities[Rats, "CategoryID = CATEGORYID_ENTITY && IsNPC"]

; BETTER - even more specific
EVE:QueryEntities[Rats, "CategoryID = CATEGORYID_ENTITY && IsNPC && Distance < 150000"]

; BEST - cache and reuse
variable index:entity CachedRats
variable float LastQueryTime = 0

function GetRats()
{
    ; Only re-query every 5 seconds
    if ${Math.Calc[${Time.Timestamp} - ${LastQueryTime}]} > 5
    {
        EVE:QueryEntities[CachedRats, "CategoryID = CATEGORYID_ENTITY && IsNPC && Distance < 150000"]
        LastQueryTime:Set[${Time.Timestamp}]
    }

    return ${CachedRats}
}
```

### Problem 4: Entity.Distance Suddenly Changes Massively

**Symptom:** Entity shows 10km away, next frame shows 250,000km

**Cause:** Grid change or warp - entity moved to different location

**Solution:**

```lavish
method ValidateEntityDistance(int64 entityID, float maxDistance)
{
    if !${Entity[${entityID}](exists)}
    {
        return FALSE
    }

    ; Check reasonable distance
    if ${Entity[${entityID}].Distance} > ${maxDistance}
    {
        Logger:Log["Entity ${Entity[${entityID}].Name} moved out of range (${Entity[${entityID}].Distance}m)"]
        return FALSE
    }

    ; Check for grid change (sudden massive distance jump)
    variable static float lastDistance = 0

    if ${lastDistance} > 0
    {
        variable float distanceChange = ${Math.Calc[${Entity[${entityID}].Distance} - ${lastDistance}]}

        if ${Math.Calc[${distanceChange}].Abs} > 100000    ; 100km sudden change
        {
            Logger:Log["Grid change detected - entity moved ${distanceChange}m instantly"]
            lastDistance:Set[0]
            return FALSE
        }
    }

    lastDistance:Set[${Entity[${entityID}].Distance}]
    return TRUE
}
```

---

## Module and Ship Control Issues <a name="module-issues"></a>

### Problem 5: Module Won't Activate

**Symptom:** `Module:Click` does nothing

**Common Causes:**
1. Module offline
2. No capacitor
3. No target (for targeted modules)
4. No charges (for guns)
5. Module still cycling

**Complete Diagnostic:**

```lavish
function DiagnoseModuleActivation(int64 moduleID)
{
    if !${Me.GetModule[${moduleID}](exists)}
    {
        echo "✗ Module doesn't exist"
        return
    }

    variable module M = ${Me.GetModule[${moduleID}]}

    echo "=== Module Diagnostic ==="
    echo "Name: ${M.ToItem.Name}"
    echo "IsOnline: ${M.IsOnline}"
    echo "IsActive: ${M.IsActive}"
    echo "IsActivatable: ${M.IsActivatable}"
    echo "IsDeactivating: ${M.IsDeactivating}"
    echo "IsReloadingAmmo: ${M.IsReloadingAmmo}"
    echo "IsGoingOnline: ${M.IsGoingOnline}"
    echo "CurrentCharges: ${M.Charge.Quantity}"
    echo "Ship Cap: ${MyShip.Capacitor.Pct.Precision[0]}%"
    echo "========================="

    ; Specific fixes
    if !${M.IsOnline}
    {
        echo "FIX: Module OFFLINE - put online manually or script:Module:SetOnline"
    }

    if ${M.IsActive}
    {
        echo "INFO: Module already active"
    }

    if !${M.IsActivatable}
    {
        echo "PROBLEM: Module not activatable"

        ; Check if needs target
        if ${M.ToItem.Group.Find["Remote"](exists)} || ${M.ToItem.Group.Find["Weapon"](exists)}
        {
            if ${Me.TargetCount} == 0
            {
                echo "FIX: Module needs target, none locked"
            }
        }

        ; Check capacitor
        if ${MyShip.Capacitor.Pct} < 5
        {
            echo "FIX: Insufficient capacitor"
        }

        ; Check charges
        if ${M.ToItem.Group.Find["Projectile"](exists)} || ${M.ToItem.Group.Find["Hybrid"](exists)}
        {
            if ${M.Charge.Quantity} == 0
            {
                echo "FIX: Out of ammo - reload"
            }
        }
    }

    if ${M.IsReloadingAmmo}
    {
        echo "INFO: Module reloading - wait ${Math.Calc[10 - ${M.ReloadTimeLeft}].Int}s"
    }
}
```

### Problem 6: Weapons Don't Shoot at Target

**Symptom:** Modules activate but don't apply to target

**Cause:** Target not locked, or wrong target active

**Solution:**

```lavish
method EngageTarget(int64 targetID)
{
    ; Lock target first
    if !${Entity[${targetID}].IsLockedTarget}
    {
        Logger:Log["Target not locked - locking first"]
        Entity[${targetID}]:LockTarget
        wait 50 ${Entity[${targetID}].IsLockedTarget}

        if !${Entity[${targetID}].IsLockedTarget}
        {
            Logger:Log["Failed to lock target"]
            return FALSE
        }
    }

    ; Make active target
    Entity[${targetID}]:MakeActiveTarget
    wait 5

    ; Verify active
    if ${Me.ActiveTarget.ID} != ${targetID}
    {
        Logger:Log["Failed to make target active"]
        return FALSE
    }

    ; Now activate weapons
    Ship:Activate_Weapons

    return TRUE
}
```

### Problem 7: Ship.Activate_* Methods Miss Some Modules

**Symptom:** Only some weapons activate when calling `Ship:Activate_Weapons`

**Cause:** EVEBot Ship object methods are helper methods that may not find all modules

**Solution:**

```lavish
; Manual activation - more reliable
method ActivateAllWeapons()
{
    variable iterator Module
    MyShip.Modules:GetIterator[Module]

    if ${Module:First(exists)}
    {
        do
        {
            ; Check if weapon module
            if ${Module.Value.ToItem.Group.Find["Projectile Weapon"](exists)} ||
               ${Module.Value.ToItem.Group.Find["Energy Weapon"](exists)} ||
               ${Module.Value.ToItem.Group.Find["Hybrid Weapon"](exists)} ||
               ${Module.Value.ToItem.Group.Find["Missile Launcher"](exists)}
            {
                ; Activate if not active
                if ${Module.Value.IsActivatable} && !${Module.Value.IsActive}
                {
                    Module.Value:Click
                    wait 1
                }
            }
        }
        while ${Module:Next(exists)}
    }
}
```

---

## Navigation and Movement Problems <a name="navigation-problems"></a>

### Problem 8: WarpTo Fails Silently

**Symptom:** Call `WarpTo` but ship doesn't warp

**Causes:**
- Warp scrambled
- Warp disrupted
- Not aligned
- Insufficient capacitor
- Docking request active

**Solution:**

```lavish
method SafeWarpTo(int64 destID, int distance)
{
    ; Check if can warp
    if !${Me.ToEntity.CanWarp}
    {
        Logger:Log["Cannot warp - likely scrambled/disrupted"]

        ; Check for scrams
        variable index:entity Entities
        EVE:QueryEntities[Entities, "IsTargetingMe && Distance < 50000"]

        if ${Entities.Used} > 0
        {
            Logger:Log["Hostile within 50km - likely scrambled"]
        }

        return FALSE
    }

    ; Check capacitor
    if ${MyShip.Capacitor.Pct} < 10
    {
        Logger:Log["Insufficient capacitor for warp"]
        return FALSE
    }

    ; Cancel any active approach/orbit
    EVE:Execute[CmdStopShip]
    wait 5

    ; Initiate warp
    Entity[${destID}]:WarpTo[${distance}]

    ; Wait for warp to start
    wait 50 ${Me.ToEntity.Mode} == 3

    if ${Me.ToEntity.Mode} == 3
    {
        Logger:Log["Warp initiated successfully"]
        return TRUE
    }

    Logger:Log["Warp failed to initiate"]
    return FALSE
}
```

### Problem 9: Autopilot Gets Stuck

**Symptom:** EVE:Execute[CmdSetDestination] sets destination but ship doesn't autopilot

**Cause:** EVE's autopilot must be manually activated, or use Navigator for scripted autopilot

**Solution:**

```lavish
; Option 1: Activate EVE's built-in autopilot (not recommended - slow)
EVE:Execute[CmdSetDestination, ${destinationID}]
EVE:Execute[CmdToggleAutopilot]    ; Toggle on

; Option 2: Use ISXEVE Navigator (recommended - faster, more control)
Navigator:Destination:Set[${destinationID}]
Navigator:FlyToDestination

; Option 3: Manual autopilot with control (best for bots)
function AutopilotTo(int destinationSystemID)
{
    variable index:int Path
    EVE:GetToDestinationPath[Path]

    variable iterator System
    Path:GetIterator[System]

    if ${System:First(exists)}
    {
        do
        {
            ; Warp to stargate
            call WarpToStargate ${System.Value}

            ; Jump through
            call JumpThroughGate

            ; Wait in new system
            wait 50 ${Me.InSpace}
        }
        while ${System:Next(exists)}
    }
}
```

### Problem 10: Approach Never Reaches Target

**Symptom:** Ship approaches target forever, never stops

**Cause:** Default approach range is 0m (unreachable for large objects)

**Solution:**

```lavish
; BAD - approaches to 0m (impossible)
Entity[${targetID}]:Approach

; GOOD - specify reasonable range
Entity[${targetID}]:Approach[2500]    ; Stop at 2500m

; BETTER - approach with timeout
function ApproachWithTimeout(int64 targetID, int range, int timeoutSeconds)
{
    Entity[${targetID}]:Approach[${range}]

    variable float startTime = ${Time.Timestamp}

    ; Wait until in range or timeout
    while ${Entity[${targetID}].Distance} > ${range}
    {
        ; Check timeout
        if ${Math.Calc[${Time.Timestamp} - ${startTime}]} > ${timeoutSeconds}
        {
            Logger:Log["Approach timeout after ${timeoutSeconds}s"]
            EVE:Execute[CmdStopShip]
            return FALSE
        }

        ; Check still exists
        if !${Entity[${targetID}](exists)}
        {
            Logger:Log["Target disappeared during approach"]
            EVE:Execute[CmdStopShip]
            return FALSE
        }

        wait 10
    }

    Logger:Log["Reached target"]
    EVE:Execute[CmdStopShip]
    return TRUE
}
```

---

## Inventory and Cargo Issues <a name="inventory-issues"></a>

### Problem 11: Item:MoveTo Fails

**Symptom:** `Item:MoveTo` returns but item doesn't move

**Causes:**
- Target cargo full
- Item already in destination
- Cargo not open
- Item is fitted module
- Insufficient permissions

**Solution:**

```lavish
method SafeMoveTo(int64 itemID, string destination, int quantity)
{
    ; Validate item exists
    if !${MyShip.Cargo.Item[${itemID}](exists)}
    {
        Logger:Log["Item ${itemID} not found in cargo"]
        return FALSE
    }

    variable item TheItem = ${MyShip.Cargo.Item[${itemID}]}

    ; Check destination capacity
    switch ${destination}
    {
        case MyHangar
            variable float hangarFree = ${Math.Calc[${Me.Station.ItemHangar.Capacity} - ${Me.Station.ItemHangar.UsedCapacity}]}

            if ${TheItem.Volume} > ${hangarFree}
            {
                Logger:Log["Hangar full - need ${TheItem.Volume}m3, have ${hangarFree}m3"]
                return FALSE
            }
            break

        case OtherCargo
            ; Assumes other cargo (Orca fleet hangar, etc.) is open
            if !${EVEWindow[ByName, "Fleet Hangar"](exists)}
            {
                Logger:Log["Other cargo window not open"]
                return FALSE
            }
            break
    }

    ; Attempt move
    Logger:Log["Moving ${TheItem.Name} x${quantity} to ${destination}"]
    TheItem:MoveTo[${destination}, ${quantity}]

    ; Wait for move
    wait 20

    ; Verify moved (check if still in source)
    if ${MyShip.Cargo.Item[${itemID}].Quantity} < ${quantity}
    {
        Logger:Log["Move successful"]
        return TRUE
    }

    Logger:Log["Move may have failed - item still in cargo"]
    return FALSE
}
```

### Problem 12: Can't Access Station Hangar

**Symptom:** `Me.Station.ItemHangar` returns NULL

**Cause:** Not docked, or hangar not loaded yet

**Solution:**

```lavish
function WaitForHangar(int timeoutSeconds)
{
    if !${Me.InStation}
    {
        Logger:Log["Not in station"]
        return FALSE
    }

    variable float startTime = ${Time.Timestamp}

    ; Wait for hangar to load
    while !${Me.Station.ItemHangar(exists)}
    {
        if ${Math.Calc[${Time.Timestamp} - ${startTime}]} > ${timeoutSeconds}
        {
            Logger:Log["Timeout waiting for hangar"]
            return FALSE
        }

        wait 10
    }

    ; Hangar loaded
    wait 5    ; Extra safety wait
    return TRUE
}

; Usage
function UnloadCargo()
{
    ; Dock first
    call Dock

    ; Wait for hangar
    if !${call WaitForHangar 30}
    {
        Logger:Log["Failed to access hangar"]
        return
    }

    ; Now safe to unload using modern inventory API
    if ${EVEWindow[Inventory](exists)}
    {
        ; Modern API: Transfer cargo items to hangar
        variable index:item cargoItems
        EVEWindow[Inventory].Child[ShipCargo]:GetItems[cargoItems]

        variable iterator item
        cargoItems:GetIterator[item]

        if ${item:First(exists)}
        {
            do
            {
                item.Value:MoveTo[MyItemHangar]
                wait 5
            }
            while ${item:Next(exists)}
        }
    }
}
```

### Problem 13: Cargo Operations Fail After Docking/Undocking

**Symptom:** Cargo operations work in space, fail after docking

**Cause:** Inventory window state may change when docking/undocking

**Solution:**

```lavish
; Modern API: Always ensure inventory window is open and valid
function GetCargoSpace()
{
    ; Ensure inventory window is open
    if !${EVEWindow[Inventory](exists)}
    {
        EVE:Execute[OpenInventory]
        wait 20
    }

    if !${EVEWindow[Inventory](exists)}
    {
        Logger:Log["Cannot access inventory"]
        return 0
    }

    ; Check if we're in station or space
    if ${Me.InStation}
    {
        ; In station - cargo still accessed via ShipCargo
        return ${EVEWindow[Inventory].Child[ShipCargo].FreeSpace}
    }
    else
    {
        ; In space - use ShipCargo
        return ${EVEWindow[Inventory].Child[ShipCargo].FreeSpace}
    }
}

; Usage
variable float cargoFree = ${call GetCargoSpace}
Logger:Log["Cargo free: ${cargoFree}m3"]
```

---

## Fleet and Relay Problems <a name="fleet-problems"></a>

### Problem 14: Relay Events Not Received

**Symptom:** Relay broadcasts but other sessions don't respond

**Causes:**
- Event not registered
- Atom not attached
- Wrong event name
- Typo in relay command

**Solution:**

```lavish
; Sender (Master)
method BroadcastPrimary(int64 targetID)
{
    ; Make sure using correct event name
    relay all -event FC_Primary_Target ${targetID}
    ;                 ^^^^^^^^^^^^^^^^ Must match exactly
}

; Receiver (Slave) - MUST match exactly
method Initialize()
{
    ; Register event
    LavishScript:RegisterEvent[FC_Primary_Target]
    ;                            ^^^^^^^^^^^^^^^^ Must match exactly

    ; Attach handler
    Event[FC_Primary_Target]:AttachAtom[This:OnPrimaryTarget]
    ;     ^^^^^^^^^^^^^^^^                    ^^^^^^^^^^^^^^
}

atom OnPrimaryTarget(int64 targetID)
{
    echo "Received primary: ${targetID}"
    Entity[${targetID}]:LockTarget
}

; DEBUG: Verify event registration
function CheckEvents()
{
    echo "Registered events:"
    variable iterator Ev
    LavishScript:GetRegisteredEvents[Ev]

    if ${Ev:First(exists)}
    {
        do
        {
            echo "  ${Ev.Value}"
        }
        while ${Ev:Next(exists)}
    }
}
```

### Problem 15: Relay Sends But Parameter Missing

**Symptom:** Event fires but parameters are NULL or empty

**Cause:** Incorrect parameter types or escaping

**Solution:**

```lavish
; WRONG - string parameter not quoted
relay all -event MyEvent ${someString}
; Receiver gets: empty or just first word

; RIGHT - quote string parameters
relay all -event MyEvent "$ {someString}"

; Example: Target name with spaces
relay all -event TargetInfo "${Entity[${targetID}].Name}" ${Entity[${targetID}].Distance}

; Receiver
atom OnTargetInfo(string targetName, float distance)
{
    ; Now targetName is complete, even with spaces
    echo "Target: ${targetName} at ${distance}m"
}
```

### Problem 16: Master-Slave Desync

**Symptom:** Slaves get out of sync with master

**Cause:** Missed heartbeats or relay messages

**Solution:**

```lavish
objectdef obj_FleetSync
{
    variable time LastMasterHeartbeat
    variable int HeartbeatTimeout = 10    ; Seconds

    method Initialize()
    {
        LavishScript:RegisterEvent[Master_Heartbeat]
        Event[Master_Heartbeat]:AttachAtom[This:OnHeartbeat]
    }

    ; Master: Broadcast heartbeat
    method MasterPulse()
    {
        if ${Config.Fleet.IsMaster}
        {
            ; Send state every 2 seconds
            if ${Math.Calc[${Time.Timestamp} % 2]} < 0.1
            {
                relay all -event Master_Heartbeat "${This.CurrentState}" ${Time.Timestamp}
            }
        }
    }

    ; Slave: Receive heartbeat
    atom OnHeartbeat(string masterState, float timestamp)
    {
        This.LastMasterHeartbeat:Set[${timestamp}]
        This.MasterState:Set[${masterState}]
    }

    ; Slave: Check for timeout
    method CheckMasterAlive()
    {
        if ${Config.Fleet.IsMaster}
        {
            return TRUE    ; We are master
        }

        variable float timeSinceHeartbeat = ${Math.Calc[${Time.Timestamp} - ${This.LastMasterHeartbeat.Timestamp}]}

        if ${timeSinceHeartbeat} > ${This.HeartbeatTimeout}
        {
            Logger:Log["Master timeout - ${timeSinceHeartbeat}s since last heartbeat", LOG_CRITICAL]
            Logger:Log["Entering autonomous mode"]

            ; Go autonomous
            This:EnterAutonomousMode
            return FALSE
        }

        return TRUE
    }
}
```

---

## Performance and Memory Issues <a name="performance-issues"></a>

### Problem 17: Script Slows Down Over Time

**Symptom:** Script runs fast initially, becomes sluggish after hours

**Cause:** Memory leak - collections growing unbounded

**Solution:**

```lavish
; BAD - collection never cleared
objectdef obj_LeakyBot
{
    variable collection:entity AllTargetsEverSeen

    method Pulse()
    {
        variable index:entity CurrentTargets
        EVE:QueryEntities[CurrentTargets, "CategoryID = CATEGORYID_ENTITY"]

        variable iterator Target
        CurrentTargets:GetIterator[Target]

        if ${Target:First(exists)}
        {
            do
            {
                ; Leak! Adds forever, never removes
                This.AllTargetsEverSeen:Set[${Target.Value.ID}, ${Target.Value}]
            }
            while ${Target:Next(exists)}
        }
    }
}

; GOOD - periodic cleanup
objectdef obj_CleanBot
{
    variable collection:entity RecentTargets
    variable time LastCleanup

    method Pulse()
    {
        ; Periodic cleanup
        if ${Math.Calc[${Time.Timestamp} - ${This.LastCleanup.Timestamp}]} > 300    ; Every 5 minutes
        {
            This.RecentTargets:Clear
            This.LastCleanup:Set[${Time.Timestamp}]
            Logger:Log["Cleaned target cache"]
        }

        ; ... normal processing ...
    }
}
```

### Problem 18: Entity Queries Lag

**Symptom:** `QueryEntities` takes seconds to complete

**Cause:** Too many entities on grid (1000+), or query too broad

**Solution:**

```lavish
; BAD - queries all entities every frame
method Pulse()
{
    variable index:entity AllStuff
    EVE:QueryEntities[AllStuff]    ; Can be thousands!

    ; ... process ...
}

; GOOD - cache and throttle
objectdef obj_CachedQueries
{
    variable index:entity CachedRats
    variable float LastRatQuery = 0
    variable float RatQueryInterval = 5.0    ; Seconds

    method GetRats()
    {
        ; Only query every 5 seconds
        if ${Math.Calc[${Time.Timestamp} - ${This.LastRatQuery}]} > ${This.RatQueryInterval}
        {
            ; Specific query
            EVE:QueryEntities[This.CachedRats, "CategoryID = CATEGORYID_ENTITY && IsNPC && Distance < 150000"]
            This.LastRatQuery:Set[${Time.Timestamp}]
            Logger:Log["Queried ${This.CachedRats.Used} rats"]
        }

        return ${This.CachedRats}
    }
}
```

### Problem 19: High Memory Usage

**Symptom:** Script uses 200MB+ memory

**Causes:**
- Too many cached objects
- Large index/collection variables
- Storing full entity objects instead of IDs

**Solution:**

```lavish
; BAD - stores full entity objects
variable collection:entity AllRats    ; Each entity is large object

; GOOD - stores only IDs
variable collection:int64 AllRatIDs   ; Just 64-bit integers

; Access pattern
method ProcessRat(int64 ratID)
{
    ; Retrieve entity when needed
    if ${Entity[${ratID}](exists)}
    {
        ; Process
    }
    else
    {
        ; Remove from collection
        AllRatIDs:Remove[${ratID}]
    }
}
```

---

## Configuration Problems <a name="config-problems"></a>

### Problem 20: Config Changes Don't Save

**Symptom:** Change config values but they reset on reload

**Cause:** Config not saved to file

**Solution:**

```lavish
method SetConfigValue(string value)
{
    ; Update in memory
    Config.MyValue:Set[${value}]

    ; MUST save to persist!
    Config:Save

    ; Or use BaseConfig
    BaseConfig:Save
}
```

### Problem 21: Config File Corrupt After Crash

**Symptom:** XML parse errors when loading config

**Cause:** Script crashed while writing config file

**Solution:**

```lavish
; Backup before saving
method SafeSave()
{
    ; Create backup
    variable string backupFile = "${CONFIG_FILE}.backup"

    ; Copy current to backup
    if ${CONFIG_PATH.FileExists[${CONFIG_FILE}]}
    {
        System:Copy["${CONFIG_PATH}/${CONFIG_FILE}", "${CONFIG_PATH}/${backupFile}"]
    }

    ; Save new config
    LavishSettings[MyBotSettings]:Export["${CONFIG_PATH}/${CONFIG_FILE}"]

    Logger:Log["Config saved (backup created)"]
}

; Recovery
method LoadConfig()
{
    if !${CONFIG_PATH.FileExists[${CONFIG_FILE}]}
    {
        Logger:Log["Config file missing - checking for backup"]

        variable string backupFile = "${CONFIG_FILE}.backup"

        if ${CONFIG_PATH.FileExists[${backupFile}]}
        {
            Logger:Log["Restoring from backup"]
            System:Copy["${CONFIG_PATH}/${backupFile}", "${CONFIG_PATH}/${CONFIG_FILE}"]
        }
    }

    LavishSettings[MyBotSettings]:Import["${CONFIG_PATH}/${CONFIG_FILE}"]
}
```

---

## ISXEVE Bugs and Limitations <a name="isxeve-bugs"></a>

### Known Issue 1: Me.GetTargets Returns Stale Targets

**Symptom:** `Me.GetTargets` includes targets that are no longer locked

**Workaround:**

```lavish
; Validate each target from GetTargets
variable index:entity Targets
Me:GetTargets[Targets]

variable iterator Target
Targets:GetIterator[Target]

if ${Target:First(exists)}
{
    do
    {
        ; ALWAYS validate IsLockedTarget
        if ${Target.Value.IsLockedTarget}
        {
            ; Safe to use
            This:ProcessTarget[${Target.Value.ID}]
        }
        else
        {
            Logger:Log["Target ${Target.Value.ID} in GetTargets but not locked (stale)"]
        }
    }
    while ${Target:Next(exists)}
}
```

### Known Issue 2: Module.IsActivatable Unreliable

**Symptom:** `Module.IsActivatable` returns TRUE but Click() fails

**Workaround:**

```lavish
; Don't trust IsActivatable alone
method ActivateModule(int64 moduleID)
{
    variable module M = ${Me.GetModule[${moduleID}]}

    ; Check multiple conditions
    if !${M.IsOnline}
    {
        return FALSE
    }

    if ${M.IsActive}
    {
        return TRUE    ; Already active
    }

    if ${M.IsDeactivating}
    {
        return FALSE    ; Wait for deactivation
    }

    if ${M.IsReloadingAmmo}
    {
        return FALSE    ; Wait for reload
    }

    ; Try to activate
    M:Click

    ; Verify activation started
    wait 10

    if ${M.IsActive} || ${M.IsGoingActive}
    {
        return TRUE
    }

    return FALSE
}
```

### Known Issue 3: Entity.Distance Can Be Wrong After Warp

**Symptom:** After warping, entity distances show very large or 0

**Cause:** Grid change - entities not updated yet

**Workaround:**

```lavish
function WaitForGridLoad()
{
    ; After warp, wait for grid to stabilize
    wait 50    ; Basic wait

    ; Verify entities loaded
    variable int retries = 0

    while ${retries} < 10
    {
        variable index:entity NearbyStuff
        EVE:QueryEntities[NearbyStuff, "Distance < 150000"]

        if ${NearbyStuff.Used} > 0
        {
            ; Found entities - grid loaded
            return TRUE
        }

        wait 10
        retries:Inc
    }

    return FALSE
}

; Use after warping
function WarpAndSettle(int64 destID)
{
    Entity[${destID}]:WarpTo[0]
    wait 50 ${Me.ToEntity.Mode} == 3

    ; Wait for warp to complete
    wait 50 ${Me.ToEntity.Mode} != 3

    ; CRITICAL: Wait for grid to stabilize
    call WaitForGridLoad

    ; Now entity distances are reliable
}
```

---

## Common Scripting Mistakes <a name="scripting-mistakes"></a>

### Mistake 1: Not Checking Existence

```lavish
; WRONG - assumes entity exists
Entity[${targetID}]:LockTarget    ; CRASH if doesn't exist!

; RIGHT - always validate
if ${Entity[${targetID}](exists)}
{
    Entity[${targetID}]:LockTarget
}
```

### Mistake 2: Infinite Loops Without Wait

```lavish
; WRONG - freezes InnerSpace
while TRUE
{
    echo "Looping"
}

; RIGHT - include wait
while TRUE
{
    echo "Looping"
    wait 1    ; CRITICAL!
}
```

### Mistake 3: Variable Scope Confusion

```lavish
; WRONG - local variable, lost after function
function GetTarget()
{
    variable int64 targetID = ${Entity["CategoryID = CATEGORYID_ENTITY"].ID}
    return ${targetID}
}

variable int64 myTarget = ${call GetTarget}    ; Works

echo ${targetID}    ; ERROR: targetID doesn't exist here!

; RIGHT - use return value
variable int64 myTarget = ${call GetTarget}
echo ${myTarget}    ; Works
```

### Mistake 4: Forgetting to Detach Events

```lavish
; WRONG - memory leak
objectdef obj_LeakyObject
{
    method Initialize()
    {
        Event[MyEvent]:AttachAtom[This:OnMyEvent]
        ; No shutdown method!
    }
}

; RIGHT - always detach
objectdef obj_CleanObject
{
    method Initialize()
    {
        Event[MyEvent]:AttachAtom[This:OnMyEvent]
    }

    method Shutdown()
    {
        Event[MyEvent]:DetachAtom[This:OnMyEvent]
    }
}
```

### Mistake 5: Race Conditions with Async Operations

```lavish
; WRONG - doesn't wait for lock
Entity[${targetID}]:LockTarget
Ship:Activate_Weapons    ; Target not locked yet!

; RIGHT - wait for lock
Entity[${targetID}]:LockTarget
wait 50 ${Entity[${targetID}].IsLockedTarget}

if ${Entity[${targetID}].IsLockedTarget}
{
    Ship:Activate_Weapons
}
```

---

## Recovery and Failsafe Strategies <a name="recovery-strategies"></a>

### Strategy 1: Automatic Recovery from Stuck States

```lavish
objectdef obj_StuckDetector
{
    variable string LastState = ""
    variable time StateChangeTime
    variable int StuckThresholdSeconds = 300    ; 5 minutes

    method CheckStuck()
    {
        ; Detect state change
        if !${This.CurrentState.Equal[${This.LastState}]}
        {
            This.LastState:Set["${This.CurrentState}"]
            This.StateChangeTime:Set[${Time.Timestamp}]
            return FALSE
        }

        ; Same state - check how long
        variable float timeInState = ${Math.Calc[${Time.Timestamp} - ${This.StateChangeTime.Timestamp}]}

        if ${timeInState} > ${This.StuckThresholdSeconds}
        {
            Logger:Log["STUCK DETECTED: In ${This.CurrentState} for ${timeInState}s", LOG_CRITICAL]
            This:RecoverFromStuck
            return TRUE
        }

        return FALSE
    }

    method RecoverFromStuck()
    {
        Logger:Log["Attempting automatic recovery"]

        ; Generic recovery actions
        switch ${This.CurrentState}
        {
            case WARPING
                ; Stuck warping - might be autopilot issue
                EVE:Execute[CmdStopShip]
                This.CurrentState:Set["IDLE"]
                break

            case MINING
                ; Stuck mining - might be targeting issue
                Me:UnlockAll
                This.CurrentState:Set["IDLE"]
                break

            case HAULING
                ; Stuck hauling - go to station
                This.CurrentState:Set["DOCKING"]
                break

            default
                ; Unknown state - return to base
                This.CurrentState:Set["RETURN_HOME"]
                break
        }

        ; Reset state timer
        This.StateChangeTime:Set[${Time.Timestamp}]
    }
}
```

### Strategy 2: Dead Man's Switch

```lavish
objectdef obj_DeadMansSwitch
{
    variable time LastUserInput
    variable int TimeoutMinutes = 30

    method Pulse()
    {
        ; Check for user activity
        if ${Mouse.X} != ${This.LastMouseX} || ${Mouse.Y} != ${This.LastMouseY}
        {
            This.LastUserInput:Set[${Time.Timestamp}]
            This.LastMouseX:Set[${Mouse.X}]
            This.LastMouseY:Set[${Mouse.Y}]
        }

        ; Check timeout
        variable float minutesSinceInput = ${Math.Calc[(${Time.Timestamp} - ${This.LastUserInput.Timestamp}) / 60]}

        if ${minutesSinceInput} > ${This.TimeoutMinutes}
        {
            Logger:Log["Dead man's switch: No user input for ${minutesSinceInput} minutes", LOG_CRITICAL]
            This:EmergencyShutdown
        }
    }

    method EmergencyShutdown()
    {
        Logger:Log["Emergency shutdown initiated"]

        ; Dock if in space
        if ${Me.InSpace}
        {
            This:EmergencyDock
        }

        ; Pause script
        Script:Pause
    }
}
```

### Strategy 3: Periodic Restarts

```lavish
function main()
{
    ; Check runtime
    variable int maxRuntimeHours = 8

    while TRUE
    {
        ; Run bot
        call ProcessBotLogic

        ; Check runtime
        variable float hoursRunning = ${Math.Calc[${Script.RunningTime} / 1000 / 60 / 60]}

        if ${hoursRunning} >= ${maxRuntimeHours}
        {
            Logger:Log["Max runtime ${maxRuntimeHours}h reached - restarting"]

            ; Dock safely
            call SafeDock

            ; End script (launcher will restart)
            Script:End
        }

        wait 1
    }
}
```

---

## Debugging Workflows <a name="debugging-workflows"></a>

### Workflow 1: Systematic Issue Diagnosis

```
1. REPRODUCE
   - Can you make it happen again?
   - What are exact steps?

2. ISOLATE
   - Does it happen in specific state?
   - Specific ship/fit?
   - Specific system/situation?

3. LOG
   - Add logging around problem area
   - Log all relevant variables
   - Log state transitions

4. VALIDATE
   - Are entities valid?
   - Is ISXEVE.IsSafe?
   - Are modules online?

5. SIMPLIFY
   - Comment out complex logic
   - Test minimal case
   - Add back complexity gradually

6. FIX
   - Add validation
   - Add error handling
   - Add fallback logic

7. TEST
   - Run for extended period
   - Test edge cases
   - Monitor logs
```

### Workflow 2: Performance Investigation

```
1. PROFILE
   - Add performance profiler
   - Identify slow methods

2. MEASURE
   - Time critical operations
   - Count iterations
   - Track memory usage

3. OPTIMIZE
   - Cache frequent queries
   - Reduce iteration count
   - Clear unused collections

4. VERIFY
   - Re-measure
   - Compare before/after
   - Monitor long-term
```

---

## Production Deployment Issues <a name="deployment-issues"></a>

### Issue: Script Works in Testing, Fails in Production

**Common Causes:**
- Different system/environment
- Different ship/fit
- Network lag
- Higher entity count

**Solution:**

```lavish
; Add environment detection
function DetectEnvironment()
{
    echo "=== Environment Detection ==="
    echo "System: ${Me.SolarSystem.Name}"
    echo "Security: ${Me.SolarSystem.Security}"
    echo "Ship: ${MyShip.ToEntity.Name}"
    echo "Entities on Grid: ${call CountEntities}"
    echo "FPS: ${Display.FPS}"
    echo "Network Lag: ${ISXEVE.Lag}"
    echo "============================="

    ; Adjust settings based on environment
    if ${call CountEntities} > 500
    {
        Logger:Log["High entity count - increasing query intervals"]
        This.QueryInterval:Set[10.0]    ; Slower queries
    }
}

function CountEntities()
{
    variable index:entity All
    EVE:QueryEntities[All]
    return ${All.Used}
}
```

### Issue: Fleet Coordination Breaks Down

**Symptom:** Master-slave coordination works 1-on-1, fails with 10+ slaves

**Cause:** Relay message flooding, race conditions

**Solution:**

```lavish
; Throttle broadcasts
objectdef obj_ThrottledBroadcaster
{
    variable float LastBroadcast = 0
    variable float BroadcastInterval = 0.5    ; Max 2 broadcasts/second

    method BroadcastTargets()
    {
        ; Check if too soon
        if ${Math.Calc[${Time.Timestamp} - ${This.LastBroadcast}]} < ${This.BroadcastInterval}
        {
            return    ; Skip this broadcast
        }

        ; Broadcast
        relay "all other" -event TargetUpdate "${targetData}"
        This.LastBroadcast:Set[${Time.Timestamp}]
    }
}

; Add sequence numbers to detect missed messages
method BroadcastWithSequence()
{
    This.SequenceNumber:Inc

    relay "all other" -event TargetUpdate ${This.SequenceNumber} "${targetData}"
}

atom OnTargetUpdate(int sequence, string targetData)
{
    ; Check for missed messages
    if ${sequence} != ${Math.Calc[${This.LastSequence} + 1]}
    {
        Logger:Log["Missed ${Math.Calc[${sequence} - ${This.LastSequence} - 1]} relay messages"]

        ; Request full state
        relay "${MasterName}" -event RequestFullState
    }

    This.LastSequence:Set[${sequence}]
    This:ProcessTargetData["${targetData}"]
}
```

---

## Summary

### Top 10 Most Common Issues

1. **Entity validation** - Always check `(exists)` before using
2. **Module activation** - Check IsOnline, cap, target, ammo
3. **Infinite loops** - Always include `wait` in loops
4. **Event handling** - Register events AND attach atoms
5. **Memory leaks** - Clear collections periodically
6. **Query performance** - Cache and throttle entity queries
7. **Race conditions** - Wait for async operations to complete
8. **Config persistence** - Must call Save() to persist changes
9. **Relay parameters** - Quote string parameters properly
10. **Grid changes** - Validate after warps/jumps

### Quick Diagnostic Checklist

```
□ Is ISXEVE loaded? (${ISXEVE(exists)})
□ Is game safe? (${ISXEVE.IsSafe})
□ Is character loaded? (${Me(exists)})
□ Are entities validated? (${Entity[ID](exists)})
□ Are events registered? (LavishScript:RegisterEvent)
□ Are atoms attached? (Event:AttachAtom)
□ Are atoms detached in shutdown? (Event:DetachAtom)
□ Do loops have waits? (wait 1 minimum)
□ Is config saved? (Config:Save or BaseConfig:Save)
□ Are collections cleared? (Collection:Clear periodically)
```

### What's Next?

**Layer 8: DotNet Bridge**
- File 32: Scripts vs DotNet Programs
- File 33: When to Use DotNet Instead
- File 34: Metatron DotNet Architecture Overview

---

*Layer 7 Progress: 4/4 Complete (100%) ✅✅✅ LAYER 7 COMPLETE!*
*Total Documentation Progress: 30/37 Files (81.1%)*
*ONLY 7 FILES REMAINING!* 🚀💎✨
