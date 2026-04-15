# Combat Automation

**Purpose:** Combat bot patterns for EVE Online automation
**Audience:** Developers building combat bots and mission runners

---

## Table of Contents

1. [Combat Bot Overview](#combat-bot-overview)
2. [Combat Modes](#combat-modes)
3. [Simple Combat Bot](#simple-combat-bot)
4. [Target Management](#target-management)
5. [Weapon Systems](#weapon-systems)
6. [Movement in Combat](#movement-in-combat)
7. [Tank Management](#tank-management)
8. [Drone Combat](#drone-combat)
9. [Advanced Combat Computer](#advanced-combat-computer)
10. [Complete Working Examples](#complete-working-examples)
11. [Combat Patterns by Ship Type](#combat-patterns-by-ship-type)
12. [Common Combat Problems](#common-combat-problems)

**See also:** For architectural analysis of Tehbot's StateQueue and MiniMode system, see [18_Bot_Architecture_Analysis.md](18_Bot_Architecture_Analysis.md#tehbot-combat-analysis).

---

## Combat Bot Overview

Combat bots automate ship-to-ship combat against NPCs (and occasionally other players in defensive scenarios).

### Combat Bot Types

| Type | Purpose | Example Use Case |
|------|---------|------------------|
| Defensive Combat | Fight back when attacked | Miners with combat capability |
| Aggressive Combat | Seek and destroy NPCs | Belt ratting, anomaly running |
| Mission Runner | Complete PvE missions | Level 4 mission grinding |
| Fleet Assist | Coordinate fleet combat | Multi-ship operations |
| PvP Defensive | Defend against players | Hostile detection and escape |

### Combat Loop Architecture

```
┌─────────────┐
│ Bot Pulse   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Safety      │ Check hostiles, tank status
│ Checks      │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Target      │ Find/prioritize/lock targets
│ Management  │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Position    │ Orbit/keep range/kite
│ Management  │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Weapon      │ Activate weapons/drones
│ Systems     │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Tank        │ Manage shield/armor/hardeners
│ Management  │
└─────────────┘
```

---

## Combat Modes

**EVEBot Pattern**: Three combat modes for different situations

### Mode 1: DEFENSIVE

Fight back only when attacked and taking damage above threshold.

The implementation is integrated into `obj_SimpleCombat` below -- `State_Fight` checks `This.CombatMode` to select the appropriate behavior. See [Simple Combat Bot](#simple-combat-bot).

**Use Case**: Mining bots that can fight back if attacked

### Mode 2: AGGRESSIVE

Seek out and destroy all NPCs.

The implementation is integrated into `obj_SimpleCombat` below -- `State_Fight` checks `This.CombatMode` to select the appropriate behavior. See [Simple Combat Bot](#simple-combat-bot).

**Use Case**: Anomaly runners, belt ratters, mission bots

### Mode 3: TANK

Maintain defenses but attack nothing (or only what's already locked).

The implementation is integrated into `obj_SimpleCombat` below -- `State_Fight` checks `This.CombatMode` to select the appropriate behavior. See [Simple Combat Bot](#simple-combat-bot).

**Use Case**: During fleet operations, or when conserving ammo/cap

---

## Simple Combat Bot

**EVEBot-style simple combat implementation**

### Complete Simple Combat Object

```lavish
; ===== SIMPLE COMBAT OBJECT =====
; Based on EVEBot obj_Combat.iss

objectdef obj_SimpleCombat
{
    variable string CombatMode = "AGGRESSIVE"
    variable string CurrentState = "IDLE"
    variable bool Fled = FALSE
    variable int NextPulse = 0
    variable int PulseInterval = 1000  ; 1 second

    method Initialize()
    {
        echo "Combat system initialized (Mode: ${This.CombatMode})"
    }

    method Pulse()
    {
        if ${LavishScript.RunningTime} < ${This.NextPulse}
            return

        call This.SetState
        call This.ProcessState

        This.NextPulse:Set[${Math.Calc[${LavishScript.RunningTime} + ${This.PulseInterval}]}]
    }

    method SetState()
    {
        ; In station
        if ${Me.InStation}
        {
            This.CurrentState:Set["INSTATION"]
            return
        }

        ; In pod (emergency!)
        if ${MyShip.ToEntity.GroupID} == 29  ; Capsule (pod)
        {
            This.CurrentState:Set["FLEE"]
            return
        }

        ; Should flee
        if ${CheckShouldFlee}
        {
            This.CurrentState:Set["FLEE"]
            return
        }

        ; Have targets and not in tank mode
        if ${This.CombatMode.NotEqual["TANK"]} && ${Me.TargetCount} > 0
        {
            This.CurrentState:Set["FIGHT"]
        }
        else
        {
            This.CurrentState:Set["IDLE"]
        }
    }

    method ProcessState()
    {
        switch ${This.CurrentState}
        {
            case INSTATION
                call This.State_InStation
                break
            case IDLE
                call This.State_Idle
                break
            case FLEE
                call This.State_Flee
                break
            case FIGHT
                call This.State_Fight
                break
        }
    }

    method State_InStation()
    {
        ; Wait in station
        This.Fled:Set[FALSE]
    }

    method State_Idle()
    {
        ; Maintain tank
        call MaintainTank

        ; If aggressive mode, look for targets
        if ${This.CombatMode.Equal["AGGRESSIVE"]}
        {
            call FindAndLockTargets
        }
    }

    method State_Fight()
    {
        ; Maintain tank
        call MaintainTank

        ; Manage position
        call ManagePosition

        ; Activate weapons
        call ActivateWeapons

        ; Launch drones
        call ManageDrones
    }

    method State_Flee()
    {
        if !${This.Fled}
        {
            echo "EMERGENCY: Fleeing to safety!"
            This.Fled:Set[TRUE]
        }

        ; Recall drones
        if ${MyShip.UsedDroneBayCapacity} > 0
        {
            EVE:Execute[DroneReturnAndOrbit]
        }

        ; Warp to safe
        call FleeToSafe
    }

    method SetMode(string mode)
    {
        This.CombatMode:Set["${mode}"]
        echo "Combat mode: ${This.CombatMode}"
    }
}
```

### Support Functions

```lavish
; ===== COMBAT SUPPORT FUNCTIONS =====

function CheckShouldFlee()
{
    ; Hostile players in local
    if ${CheckForHostilesInLocal}
    {
        return TRUE
    }

    ; Low tank
    if ${MyShip.ShieldPct} < 25
    {
        echo "WARNING: Low shields!"
        return TRUE
    }

    ; Low armor (if armor tank)
    if ${MyShip.ArmorPct} < 50
    {
        echo "WARNING: Low armor!"
        return TRUE
    }

    ; Overwhelmed (too many targets)
    if ${Me.TargetedByCount} > 10
    {
        echo "WARNING: Overwhelmed by ${Me.TargetedByCount} enemies!"
        return TRUE
    }

    return FALSE
}

function MaintainTank()
{
    ; Combined tank management — delegates to specialized functions
    ; in the Tank Management section below.
    call ManageShieldTank
    call ManageArmorTank
}

function FindAndLockTargets()
{
    ; Thin wrapper — see ManageTargetLocks() in the Target Management
    ; section below for the priority-aware canonical implementation.
    call ManageTargetLocks
}

function ManagePosition()
{
    ; Get active target
    if !${Me.ActiveTarget(exists)}
    {
        if ${Me.TargetCount} > 0
        {
            ; Set first locked target as active
            variable index:entity MyTargets
            Me:GetTargets[MyTargets]
            MyTargets.Get[1]:MakeActiveTarget
        }
        return
    }

    ; Orbit active target
    variable float orbitRange = 10000  ; 10km orbit

    if ${Me.ActiveTarget.Distance} > ${Math.Calc[${orbitRange} * 1.5]}
    {
        ; Too far, approach
        Me.ActiveTarget:Approach
    }
    elseif ${Me.ActiveTarget.Distance} < ${Math.Calc[${orbitRange} * 0.5]}
    {
        ; Too close, keep range
        Me.ActiveTarget:KeepAtRange[${orbitRange}]
    }
    else
    {
        ; In range, orbit
        Me.ActiveTarget:Orbit[${orbitRange}]
    }
}

function ActivateWeapons()
{
    if !${Me.ActiveTarget(exists)}
        return

    ; Activate all weapons on active target
    variable index:module modules
    MyShip:GetModules[modules]
    variable iterator m
    modules:GetIterator[m]
    if ${m:First(exists)}
    {
        do
        {
            ; Check if weapon module
            if ${m.Value.ToItem.Group.Find["Projectile Weapon"]} || \
               ${m.Value.ToItem.Group.Find["Energy Weapon"]} || \
               ${m.Value.ToItem.Group.Find["Hybrid Weapon"]} || \
               ${m.Value.ToItem.Group.Find["Missile Launcher"]}
            {
                if !${m.Value.IsActive} && ${m.Value.IsOnline}
                {
                    m.Value:Activate[${Me.ActiveTarget.ID}]
                    wait 5
                }
            }
        }
        while ${m:Next(exists)}
    }
}

function LaunchDrones()
{
    ; Thin wrapper — see ManageDrones() in the Drone Combat section
    ; below for the full implementation with target assignment and recall.
    call ManageDrones
}

function FleeToSafe()
{
    ; Find nearest station. QueryEntities returns results sorted by distance
    ; (ISXEVEChanges.txt line 5321), so Stations.Get[1] is the closest.
    variable index:entity Stations
    EVE:QueryEntities[Stations, "GroupID = 15"]  ; Station

    if ${Stations.Used} == 0
        return

    variable entity station = ${Stations.Get[1]}

    if ${station(exists)}
    {
        if ${station.Distance} > 150000
        {
            echo "Warping to ${station.Name}"
            station:WarpTo[0]
            wait 100  ; Wait for warp
        }

        if !${Me.InStation}
        {
            echo "Docking at ${station.Name}"
            station:Dock
            wait 50  ; Wait for dock
        }
    }
}
```

---

## Target Management

### Priority Targeting Pattern

```lavish
; ===== PRIORITY TARGETING =====

variable(global) index:string PriorityTargets

function LoadPriorityTargets()
{
    ; Web/scram rats (highest priority)
    PriorityTargets:Insert["Dire Pithi Arrogator"]    ; web/scram
    PriorityTargets:Insert["Dire Pithi Infiltrator"]  ; web/scram
    PriorityTargets:Insert["Guardian Agent"]          ; web/scram
    PriorityTargets:Insert["Guardian Scout"]          ; web/scram

    ; Jamming rats
    PriorityTargets:Insert["Dire Pithi Saboteur"]     ; jamming
    PriorityTargets:Insert["Dire Guristas Nullifier"] ; jamming

    ; Neut rats
    PriorityTargets:Insert["Elder Blood Upholder"]    ; neut
    PriorityTargets:Insert["Blood Wraith"]            ; neut
}

function GetBestTarget()
{
    variable index:entity AllTargets
    EVE:QueryEntities[AllTargets, "CategoryID = CATEGORYID_ENTITY && IsNPC && !IsMoribund && Distance < ${MyShip.MaxTargetRange}"]

    ; Pass 1: Priority targets
    variable iterator Target
    AllTargets:GetIterator[Target]

    if ${Target:First(exists)}
        do
        {
            if ${IsPriorityTarget["${Target.Value.Name}"]}
            {
                return ${Target.Value.ID}
            }
        }
        while ${Target:Next(exists)}

    ; Pass 2: Anything targeting me
    Target:First

    if ${Target:First(exists)}
        do
        {
            if ${Target.Value.IsTargetingMe}
            {
                return ${Target.Value.ID}
            }
        }
        while ${Target:Next(exists)}

    ; Pass 3: Closest
    Target:First

    if ${Target:First(exists)}
    {
        return ${Target.Value.ID}
    }

    return 0
}

function IsPriorityTarget(string name)
{
    variable int i
    for (i:Set[1]; ${i} <= ${PriorityTargets.Used}; i:Inc)
    {
        if ${name.Find["${PriorityTargets[${i}]}"]}
        {
            return TRUE
        }
    }

    return FALSE
}
```

### Smart Target Switching

```lavish
; ===== SMART TARGET SWITCHING =====

function ManageTargetLocks()
{
    ; Fill to max targets
    while ${Me.TargetCount} < ${Me.MaxLockedTargets}
    {
        variable int64 bestTarget = ${GetBestTarget}

        if ${bestTarget} == 0
            break

        ; Check if already locked
        if ${Entity[${bestTarget}].IsLockedTarget} || ${Entity[${bestTarget}].BeingTargeted}
            continue

        ; Lock
        Entity[${bestTarget}]:LockTarget
        wait 10
    }

    ; Switch active target if needed
    call CheckActiveTargetSwitch
}

function CheckActiveTargetSwitch()
{
    variable index:entity MyTargets
    Me:GetTargets[MyTargets]

    ; No active target
    if !${Me.ActiveTarget(exists)}
    {
        if ${MyTargets.Used} > 0
        {
            MyTargets.Get[1]:MakeActiveTarget
        }
        return
    }

    ; Active target dead
    if ${Me.ActiveTarget.IsMoribund}
    {
        echo "Active target dead, switching"
        call SwitchToNextTarget
        return
    }

    ; Active target not priority, but we have a priority locked
    if !${IsPriorityTarget["${Me.ActiveTarget.Name}"]}
    {
        variable int i
        for (i:Set[1]; ${i} <= ${MyTargets.Used}; i:Inc)
        {
            variable entity target = ${MyTargets.Get[${i}]}

            if ${IsPriorityTarget["${target.Name}"]} && !${target.IsMoribund}
            {
                echo "Switching to priority target: ${target.Name}"
                target:MakeActiveTarget
                return
            }
        }
    }
}

function SwitchToNextTarget()
{
    variable index:entity MyTargets
    Me:GetTargets[MyTargets]

    ; Find first non-dead locked target
    variable int i
    for (i:Set[1]; ${i} <= ${MyTargets.Used}; i:Inc)
    {
        variable entity target = ${MyTargets.Get[${i}]}

        if !${target.IsMoribund}
        {
            target:MakeActiveTarget
            return
        }
    }
}
```

---

## Weapon Systems

### Turret Weapon Management

```lavish
; ===== TURRET WEAPON MANAGEMENT =====

function ActivateTurrets(int64 targetID)
{
    if !${Entity[${targetID}](exists)}
        return

    variable index:module modules
    MyShip:GetModules[modules]
    variable iterator m
    modules:GetIterator[m]
    if ${m:First(exists)}
    {
        do
        {
            ; Check if turret
            if ${m.Value.ToItem.Group.Find["Projectile Weapon"]} || \
               ${m.Value.ToItem.Group.Find["Energy Weapon"]} || \
               ${m.Value.ToItem.Group.Find["Hybrid Weapon"]}
            {
                if !${m.Value.IsActive} && ${m.Value.IsOnline}
                {
                    m.Value:Activate[${targetID}]
                }
            }
        }
        while ${m:Next(exists)}
    }
}

function CheckTurretTracking(int64 targetID)
{
    ; Check if target is too close to track
    variable entity target = ${Entity[${targetID}]}

    if !${target(exists)}
        return FALSE

    ; Get ship tracking (simplified - real implementation would check module stats)
    variable float trackingSpeed = 0.05  ; rad/s (example)
    variable float targetAngularVelocity = ${Math.Calc[${target.Velocity} / ${target.Distance}]}

    if ${targetAngularVelocity} > ${trackingSpeed}
    {
        echo "WARNING: Target moving too fast to track (${targetAngularVelocity} > ${trackingSpeed})"
        return FALSE
    }

    return TRUE
}

function CheckAmmo()
{
    ; Check if need to reload
    variable index:module modules
    MyShip:GetModules[modules]
    variable iterator m
    modules:GetIterator[m]
    if ${m:First(exists)}
    {
        do
        {
            ; Check if weapon with charges
            if ${m.Value.ToItem.Group.Find["Weapon"]} && ${m.Value.MaxCharges} > 0
            {
                ; Below 30% ammo and not currently active
                if ${m.Value.Charge} < ${Math.Calc[${m.Value.MaxCharges} * 0.3]} && !${m.Value.IsActive}
                {
                    echo "Reloading ${m.Value.ToItem.Name}"
                    m.Value:ReloadCycle
                    wait 50  ; Wait for reload (varies by weapon)
                }
            }
        }
        while ${m:Next(exists)}
    }
}
```

### Missile Weapon Management

```lavish
; ===== MISSILE WEAPON MANAGEMENT =====

function ActivateMissiles(int64 targetID)
{
    if !${Entity[${targetID}](exists)}
        return

    variable index:module modules
    MyShip:GetModules[modules]
    variable iterator m
    modules:GetIterator[m]
    if ${m:First(exists)}
    {
        do
        {
            ; Check if missile launcher
            if ${m.Value.ToItem.Group.Find["Missile Launcher"]}
            {
                if !${m.Value.IsActive} && ${m.Value.IsOnline}
                {
                    m.Value:Activate[${targetID}]
                    wait 5  ; Missiles activate slower than turrets
                }
            }
        }
        while ${m:Next(exists)}
    }
}

function CheckMissileRange(int64 targetID)
{
    variable entity target = ${Entity[${targetID}]}

    if !${target(exists)}
        return FALSE

    ; Missiles have max range (example: light missiles ~50km)
    variable float missileRange = 50000

    if ${target.Distance} > ${missileRange}
    {
        echo "WARNING: Target out of missile range (${target.Distance}m > ${missileRange}m)"
        return FALSE
    }

    return TRUE
}
```

---

## Movement in Combat

### Orbit Pattern

```lavish
; ===== ORBIT PATTERN =====

variable float OptimalOrbitRange = 10000
variable int64 CurrentOrbitTarget = 0

function OrbitActiveTarget()
{
    if !${Me.ActiveTarget(exists)}
        return

    variable float distance = ${Me.ActiveTarget.Distance}

    ; Already orbiting this target
    if ${CurrentOrbitTarget} == ${Me.ActiveTarget.ID}
    {
        ; Check if orbit broke
        if ${distance} > ${Math.Calc[${OptimalOrbitRange} * 2]} || \
           ${distance} < ${Math.Calc[${OptimalOrbitRange} * 0.5]}
        {
            echo "Orbit broke, re-establishing"
            Me.ActiveTarget:Orbit[${OptimalOrbitRange}]
            wait 10
        }

        return
    }

    ; Not orbiting, start orbit
    echo "Orbiting ${Me.ActiveTarget.Name} at ${OptimalOrbitRange}m"
    Me.ActiveTarget:Orbit[${OptimalOrbitRange}]
    CurrentOrbitTarget:Set[${Me.ActiveTarget.ID}]
    wait 10
}
```

### Keep Range Pattern

```lavish
; ===== KEEP RANGE PATTERN =====

variable float OptimalKeepRange = 20000

function KeepRangeFromActiveTarget()
{
    if !${Me.ActiveTarget(exists)}
        return

    variable float distance = ${Me.ActiveTarget.Distance}

    ; Too far
    if ${distance} > ${Math.Calc[${OptimalKeepRange} * 1.2]}
    {
        echo "Too far (${distance}m), approaching to ${OptimalKeepRange}m"
        Me.ActiveTarget:Approach
        wait 10
        return
    }

    ; Too close
    if ${distance} < ${Math.Calc[${OptimalKeepRange} * 0.8]}
    {
        echo "Too close (${distance}m), keeping range at ${OptimalKeepRange}m"
        Me.ActiveTarget:KeepAtRange[${OptimalKeepRange}]
        wait 10
        return
    }

    ; In good range -- issue KeepAtRange if we're not already moving toward
    ; or orbiting the target. Entity Mode values: 0=Aligned, 1=Approaching,
    ; 2=Stopped, 3=Warping, 4=Orbiting. The KeepAtRange command produces
    ; Mode 4 (Orbiting), so skipping Mode 1 and 4 avoids re-issuing the
    ; command every pulse while we're already maintaining range.
    if ${Me.ToEntity.Mode} != 1 && ${Me.ToEntity.Mode} != 4
    {
        Me.ActiveTarget:KeepAtRange[${OptimalKeepRange}]
        wait 10
    }
}
```

### Kiting Pattern (Advanced)

```lavish
; ===== KITING PATTERN =====
; Move away while shooting (for long-range ships)

variable float KiteRange = 50000  ; Stay at 50km

function KiteActiveTarget()
{
    if !${Me.ActiveTarget(exists)}
        return

    variable float distance = ${Me.ActiveTarget.Distance}
    variable float velocity = ${Me.ActiveTarget.Velocity}

    ; Target approaching fast
    if ${velocity} > 500 && ${distance} < ${Math.Calc[${KiteRange} * 1.5]}
    {
        echo "Target approaching fast, increasing distance"

        ; Find opposite direction and warp there
        call MoveAwayFromTarget ${Me.ActiveTarget.ID}
        return
    }

    ; Normal kiting - keep at max range
    if ${distance} > ${Math.Calc[${KiteRange} * 1.1]}
    {
        Me.ActiveTarget:Approach
        wait 10
    }
    elseif ${distance} < ${Math.Calc[${KiteRange} * 0.9]}
    {
        Me.ActiveTarget:KeepAtRange[${KiteRange}]
        wait 10
    }
}

function MoveAwayFromTarget(int64 targetID)
{
    ; Find celestial opposite to target
    variable index:entity Celestials
    EVE:QueryEntities[Celestials, "GroupID = 7 || GroupID = 8"]  ; Planet, Moon

    ; Find furthest from target
    variable float maxDistance = 0
    variable int64 bestCelestial = 0

    variable iterator Celestial
    Celestials:GetIterator[Celestial]

    if ${Celestial:First(exists)}
        do
        {
            variable float dist = ${Entity[${targetID}].DistanceTo[${Celestial.Value.ID}]}

            if ${dist} > ${maxDistance}
            {
                maxDistance:Set[${dist}]
                bestCelestial:Set[${Celestial.Value.ID}]
            }
        }
        while ${Celestial:Next(exists)}

    if ${bestCelestial} > 0
    {
        echo "Warping away from target to ${Entity[${bestCelestial}].Name}"
        Entity[${bestCelestial}]:WarpTo[0]
    }
}
```

---

## Tank Management

### Shield Tank

```lavish
; ===== SHIELD TANK MANAGEMENT =====

function ManageShieldTank()
{
    variable float shieldPct = ${MyShip.ShieldPct}

    ; Activate passive hardeners (always on)
    call ActivatePassiveHardeners

    ; Activate shield boosters when needed
    if ${shieldPct} < 80
    {
        call ActivateShieldBoosters
    }

    ; Check if need to flee
    if ${shieldPct} < 30
    {
        echo "CRITICAL: Shields at ${shieldPct}%!"
        return TRUE  ; Should flee
    }

    return FALSE
}

function ActivatePassiveHardeners()
{
    variable index:module modules
    MyShip:GetModules[modules]
    variable iterator m
    modules:GetIterator[m]
    if ${m:First(exists)}
    {
        do
        {
            if ${m.Value.ToItem.Group.Find["Shield Hardener"]}
            {
                if !${m.Value.IsActive} && ${m.Value.IsOnline}
                {
                    m.Value:Activate
                }
            }
        }
        while ${m:Next(exists)}
    }
}

function ActivateShieldBoosters()
{
    variable index:module modules
    MyShip:GetModules[modules]
    variable iterator m
    modules:GetIterator[m]
    if ${m:First(exists)}
    {
        do
        {
            if ${m.Value.ToItem.Group.Find["Shield Booster"]}
            {
                ; Check capacitor before activating
                if ${MyShip.CapacitorPct} > 30
                {
                    if !${m.Value.IsActive} && ${m.Value.IsOnline}
                    {
                        m.Value:Activate
                    }
                }
            }
        }
        while ${m:Next(exists)}
    }
}
```

### Armor Tank

```lavish
; ===== ARMOR TANK MANAGEMENT =====

function ManageArmorTank()
{
    variable float armorPct = ${MyShip.ArmorPct}
    variable float hullPct = ${MyShip.StructurePct}

    ; Activate hardeners (always on)
    call ActivateArmorHardeners

    ; Activate repairers when damaged
    if ${armorPct} < 90
    {
        call ActivateArmorRepairers
    }

    ; Critical: Taking hull damage
    if ${hullPct} < 99
    {
        echo "CRITICAL: Hull damage!"
        return TRUE  ; Should flee
    }

    ; Low armor
    if ${armorPct} < 40
    {
        echo "WARNING: Low armor (${armorPct}%)"
        return TRUE  ; Should flee
    }

    return FALSE
}

function ActivateArmorHardeners()
{
    variable index:module modules
    MyShip:GetModules[modules]
    variable iterator m
    modules:GetIterator[m]
    if ${m:First(exists)}
    {
        do
        {
            if ${m.Value.ToItem.Group.Find["Armor Hardener"]}
            {
                if !${m.Value.IsActive} && ${m.Value.IsOnline}
                {
                    m.Value:Activate
                }
            }
        }
        while ${m:Next(exists)}
    }
}

function ActivateArmorRepairers()
{
    variable index:module modules
    MyShip:GetModules[modules]
    variable iterator m
    modules:GetIterator[m]
    if ${m:First(exists)}
    {
        do
        {
            if ${m.Value.ToItem.Group.Find["Armor Repairer"]}
            {
                ; Check capacitor
                if ${MyShip.CapacitorPct} > 25
                {
                    if !${m.Value.IsActive} && ${m.Value.IsOnline}
                    {
                        m.Value:Activate
                    }
                }
            }
        }
        while ${m:Next(exists)}
    }
}
```

---

## Drone Combat

### Basic Drone Management

```lavish
; ===== BASIC DRONE MANAGEMENT =====

function ManageDrones()
{
    ; No drone bay
    if ${MyShip.DroneBayCapacity} == 0
        return

    variable index:activedrone MyDrones
    Me:GetActiveDrones[MyDrones]

    ; Drones not launched yet
    if ${MyDrones.Used} == 0
    {
        if ${Me.TargetCount} > 0
        {
            call LaunchDrones
        }

        return
    }

    ; Drones launched, manage them
    call UpdateDroneTargets
}

function LaunchDrones()
{
    echo "Launching drones"
    MyShip:LaunchAllDrones
    wait 30  ; Wait for launch
}

function UpdateDroneTargets()
{
    ; Check if drones are engaging current target
    if !${Me.ActiveTarget(exists)}
        return

    ; Command drones to attack active target
    EVE:Execute[CmdDronesEngage]
}

function RecallDrones()
{
    echo "Recalling drones"
    EVE:Execute[CmdDronesReturnToBay]

    ; Wait for drones to return
    variable index:activedrone MyDrones
    variable int waitTime = 0
    Me:GetActiveDrones[MyDrones]
    while ${MyDrones.Used} > 0 && ${waitTime} < 30000
    {
        wait 10
        waitTime:Inc[100]
        Me:GetActiveDrones[MyDrones]
    }

    if ${MyDrones.Used} > 0
    {
        echo "WARNING: Drones did not return in time!"
    }
}
```

---

## Advanced Combat Computer

**Tehbot Pattern**: SQL database for damage optimization

### Damage Calculation System

```lavish
; ===== ADVANCED DAMAGE OPTIMIZATION =====
; Based on Tehbot obj_CombatComputer.iss

objectdef obj_AdvancedCombat
{
    variable sqlitedb CombatData
    variable collection:int AmmoTypes

    method Initialize()
    {
        ; Create in-memory database
        CombatData:Set[${SQLite.OpenDB["CombatData",":memory:"]}]

        ; Create tables
        CombatData:ExecDML["CREATE TABLE CurrentData (EntityID INTEGER PRIMARY KEY, NPCName TEXT, Distance REAL, ShieldPct REAL, ArmorPct REAL, Velocity REAL)"]

        CombatData:ExecDML["CREATE TABLE AmmoEffectiveness (EntityID INTEGER, AmmoTypeID INTEGER, ExpectedDamage REAL, ShotsToKill REAL, TimeToKill REAL, PRIMARY KEY(EntityID, AmmoTypeID))"]

        ; Load available ammo
        call This.LoadAmmoTypes
    }

    method LoadAmmoTypes()
    {
        ; ⚠️ Modern Inventory API (July 2020+)
        ; Open inventory window if not already open
        if !${EVEWindow[Inventory](exists)}
        {
            EVE:Execute[CmdOpenInventory]
            wait 20
        }

        ; Scan cargo for ammo
        variable index:item CargoItems
        EVEWindow[Inventory].Child[ShipCargo]:GetItems[CargoItems]

        variable iterator Item
        CargoItems:GetIterator[Item]

        if ${Item:First(exists)}
            do
            {
                ; Check if ammo
                if ${Item.Value.CategoryID} == 8  ; Charge category
                {
                    AmmoTypes:Set[${Item.Value.TypeID}, ${Item.Value.Quantity}]
                    echo "Found ammo: ${Item.Value.Name} (${Item.Value.Quantity})"
                }
            }
            while ${Item:Next(exists)}
    }

    method AnalyzeTarget(int64 entityID)
    {
        ; Get target info
        variable entity target = ${Entity[${entityID}]}

        if !${target(exists)}
            return

        ; Store basic target data
        CombatData:ExecDML["INSERT OR REPLACE INTO CurrentData VALUES (${entityID}, '${target.Name}', ${target.Distance}, ${target.ShieldPct}, ${target.ArmorPct}, ${target.Velocity})"]

        ; Calculate effectiveness for each ammo type
        call This.CalculateAmmoEffectiveness ${entityID}
    }

    method CalculateAmmoEffectiveness(int64 entityID)
    {
        variable entity target = ${Entity[${entityID}]}

        ; For each ammo type
        variable iterator Ammo
        AmmoTypes:GetIterator[Ammo]

        if ${Ammo:First(exists)}
            do
            {
                ; Calculate damage (simplified - real version uses resist profiles)
                variable float baseDamage = 100  ; Example
                variable float damageAfterResists = ${Math.Calc[${baseDamage} * 0.75]}  ; 75% effectiveness

                ; Calculate shots to kill
                variable float targetHP = ${Math.Calc[${target.ShieldHP} + ${target.ArmorHP}]}
                variable float shotsToKill = ${Math.Calc[${targetHP} / ${damageAfterResists}]}

                ; Calculate time to kill (assume 2s cycle time)
                variable float timeToKill = ${Math.Calc[${shotsToKill} * 2]}

                ; Store in database
                CombatData:ExecDML["INSERT OR REPLACE INTO AmmoEffectiveness VALUES (${entityID}, ${Ammo.Key}, ${damageAfterResists}, ${shotsToKill}, ${timeToKill})"]
            }
            while ${Ammo:Next(exists)}
    }

    member:int GetBestAmmoForTarget(int64 entityID)
    {
        ; Query for best ammo (shortest time to kill)
        variable sqlitequery result
        result:Set[${CombatData.ExecQuery["SELECT AmmoTypeID FROM AmmoEffectiveness WHERE EntityID=${entityID} ORDER BY TimeToKill ASC LIMIT 1"]}]

        if ${result.NumRows} > 0
        {
            variable int ammoTypeID = ${result.GetFieldValue["AmmoTypeID"]}
            result:Finalize
            return ${ammoTypeID}
        }

        result:Finalize
        return 0
    }
}
```

---

## Complete Working Examples

### Example 1: Simple Anomaly Runner

```lavish
; ===== SIMPLE ANOMALY RUNNER =====

variable bool BotRunning = TRUE
variable obj_SimpleCombat Combat

function main()
{
    ; Startup
    while !${ISXEVE.IsReady}
    {
        wait 10
    }

    while !${Me(exists)} || !${MyShip(exists)}
    {
        wait 10
    }

    ; Initialize
    echo "Anomaly Runner starting"
    Combat:Initialize
    Combat:SetMode["AGGRESSIVE"]

    ; Main loop
    while ${BotRunning}
    {
        if ${Me.InSpace}
        {
            Combat:Pulse
        }

        wait ${Math.Calc[${Math.Rand[8]} + 8]}  ; 8..15 inclusive
        waitframe
    }
}

function atexit()
{
    echo "Anomaly Runner shutting down"
    call RecallDrones
}
```

### Example 2: Defensive Miner

```lavish
; ===== DEFENSIVE MINER WITH COMBAT =====

variable bool BotRunning = TRUE
variable obj_SimpleCombat Combat
variable string CurrentState = "MINING"

function main()
{
    ; (Startup boilerplate — ISXEVE.IsReady + Me/MyShip wait — identical to Example 1 above)

    ; Initialize
    echo "Defensive Miner starting"
    Combat:Initialize
    Combat:SetMode["DEFENSIVE"]

    ; Main loop
    while ${BotRunning}
    {
        if ${Me.InSpace}
        {
            ; Check if under attack
            if ${Me.TargetedByCount} > 0
            {
                if !${CurrentState.Equal["COMBAT"]}
                {
                    echo "Under attack! Switching to combat mode"
                    CurrentState:Set["COMBAT"]
                }
            }
            else
            {
                if !${CurrentState.Equal["MINING"]}
                {
                    echo "Threats cleared, resuming mining"
                    CurrentState:Set["MINING"]
                }
            }

            ; Process current state
            switch ${CurrentState}
            {
                case MINING
                    call ProcessMining
                    break
                case COMBAT
                    Combat:Pulse
                    break
            }
        }

        wait ${Math.Calc[${Math.Rand[6]} + 5]}  ; 5..10 inclusive
        waitframe
    }
}

function ProcessMining()
{
    ; Mine asteroids
    ; (See File 22 for mining implementation)
}
```

---

## Combat Patterns by Ship Type

### Frigate Combat

| Setting | Value | Rationale |
|---------|-------|-----------|
| Movement | Orbit at 2,000m | Close orbit for speed tanking |
| Tank | Afterburner (speed tank) | Low HP, rely on velocity |

Use `OrbitActiveTarget()` from the [Movement in Combat](#movement-in-combat) section with `FrigateOrbitRange = 2000`. Activate afterburner as the defensive measure.

### Cruiser Combat

| Setting | Value | Rationale |
|---------|-------|-----------|
| Movement | KeepAtRange 15,000m | Medium standoff distance |
| Tank | Shield tank | Moderate buffer, active hardeners |

Use `KeepRangeFromActiveTarget()` from the [Movement in Combat](#movement-in-combat) section with `CruiserKeepRange = 15000`. Run shield tank modules for defense.

### Battleship Combat

| Setting | Value | Rationale |
|---------|-------|-----------|
| Movement | KeepAtRange 50,000m | Kite at long range |
| Tank | Armor tank + drones | Heavy buffer, drone DPS supplement |

Use `KeepRangeFromActiveTarget()` from the [Movement in Combat](#movement-in-combat) section with `BattleshipKeepRange = 50000`. Manage armor tank and deploy drones for additional damage.

---

## Common Combat Problems

### Problem 1: Weapons Won't Activate

**Symptoms**: Module:Activate does nothing

**Solutions**:

```lavish
; moduleSlot is a slot-name token, e.g. "HiSlot0".."HiSlot7" (NOT numeric).
function DiagnoseWeaponFailure(string moduleSlot, int64 targetID)
{
    variable module module = ${MyShip.Module[${moduleSlot}]}

    ; Not online
    if !${module.IsOnline}
    {
        echo "ERROR: Module offline"
        return "OFFLINE"
    }

    ; Already active
    if ${module.IsActive}
    {
        return "ALREADY_ACTIVE"
    }

    ; No target
    if !${Entity[${targetID}](exists)}
    {
        echo "ERROR: Target doesn't exist"
        return "NO_TARGET"
    }

    ; Target not locked
    if !${Entity[${targetID}].IsLockedTarget}
    {
        echo "ERROR: Target not locked"
        return "NOT_LOCKED"
    }

    ; Out of range
    if ${Entity[${targetID}].Distance} > ${MyShip.MaxTargetRange}
    {
        echo "ERROR: Out of range"
        return "OUT_OF_RANGE"
    }

    ; No capacitor
    if ${MyShip.CapacitorPct} < 5
    {
        echo "ERROR: No capacitor"
        return "NO_CAP"
    }

    return "UNKNOWN"
}
```

### Problem 2: Drones Won't Launch

```lavish
function DiagnoseDroneProblem()
{
    ; No drone bay
    if ${MyShip.DroneBayCapacity} == 0
    {
        echo "ERROR: No drone bay"
        return
    }

    ; No drones
    if ${MyShip.UsedDroneBayCapacity} == 0
    {
        echo "ERROR: No drones in bay"
        return
    }

    ; Already launched
    variable index:activedrone MyDrones
    Me:GetActiveDrones[MyDrones]
    if ${MyDrones.Used} > 0
    {
        echo "Drones already launched"
        return
    }

    ; Drone bandwidth issue
    if ${MyShip.DroneBandwidth} < 5
    {
        echo "ERROR: Insufficient drone bandwidth"
        return
    }

    echo "Drones should be able to launch, trying again"
    MyShip:LaunchAllDrones
}
```


---
