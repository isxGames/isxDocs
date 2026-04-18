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
9. [Panic State Machine (Emergency Patterns)](#panic-state-machine-emergency-patterns)
10. [Advanced Combat Computer](#advanced-combat-computer)
11. [Module Control Layer (Production Patterns)](#module-control-layer-production-patterns)
    - [Self-Maintaining Locked-Target Lists (`obj_TargetList`)](#self-maintaining-locked-target-lists-obj_targetlist)
    - [Per-Module Instruction Queue (`obj_Module`)](#per-module-instruction-queue-obj_module)
    - [Group Control over Module Wrappers (`obj_ModuleList`)](#group-control-over-module-wrappers-obj_modulelist)
    - [Declarative Module Registration (`AddModuleList`)](#declarative-module-registration-addmodulelist)
    - [Module Overload by Module HP](#module-overload-by-module-hp)
    - [Static Module Classification on Init (EVEBot Stable)](#static-module-classification-on-init-evebot-stable)
12. [Complete Working Examples](#complete-working-examples)
13. [Combat Patterns by Ship Type](#combat-patterns-by-ship-type)
14. [Common Combat Problems](#common-combat-problems)

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
            ; Check if weapon module (GroupID compare — numeric, robust, no substring false positives)
            ; 55 = Projectile Weapon, 53 = Energy Weapon, 74 = Hybrid Weapon, 56 = Missile Launcher
            if ${m.Value.ToItem.GroupID} == 55 || \
               ${m.Value.ToItem.GroupID} == 53 || \
               ${m.Value.ToItem.GroupID} == 74 || \
               ${m.Value.ToItem.GroupID} == 56
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
            ; Check if turret (GroupID compare — numeric, robust, no substring false positives)
            ; 55 = Projectile Weapon, 53 = Energy Weapon, 74 = Hybrid Weapon
            if ${m.Value.ToItem.GroupID} == 55 || \
               ${m.Value.ToItem.GroupID} == 53 || \
               ${m.Value.ToItem.GroupID} == 74
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
    ; Returns TRUE if our turret tracking speed can keep up with the target's
    ; angular velocity, FALSE if the target is angularly too fast to hit reliably.
    variable entity target = ${Entity[${targetID}]}

    if !${target(exists)}
        return FALSE

    ; Example turret tracking speed (rad/s). A real implementation would read
    ; the turret module's TrackingSpeed attribute and scale by signature radius
    ; / scan resolution for the actual hit-probability computation.
    variable float trackingSpeed = 0.05  ; rad/s

    ; Use ISXEVE's entity.AngularVelocity member directly (rad/s, computed by
    ; the game client). Do NOT approximate as (Velocity / Distance): Velocity
    ; is the target's total speed magnitude, not its tangential component, so a
    ; target flying straight toward you (pure radial motion -- true angular
    ; velocity = 0) would spuriously appear to have nonzero angular velocity.
    ; If you need the tangential component explicitly, use entity.TransversalVelocity
    ; (m/s) -- angular velocity is then TransversalVelocity / Distance.
    if ${target.AngularVelocity} > ${trackingSpeed}
    {
        echo "WARNING: Target angular velocity ${target.AngularVelocity} rad/s exceeds tracking ${trackingSpeed} rad/s"
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
            ; Check if weapon with charges (GroupID compare — numeric, robust)
            ; 55 = Projectile Weapon, 53 = Energy Weapon, 74 = Hybrid Weapon, 56 = Missile Launcher
            ; (Note: prior code used Group.Find["Weapon"] which also matches "Shield Boost Amplifier" via substring.)
            if (${m.Value.ToItem.GroupID} == 55 || \
                ${m.Value.ToItem.GroupID} == 53 || \
                ${m.Value.ToItem.GroupID} == 74 || \
                ${m.Value.ToItem.GroupID} == 56) && ${m.Value.MaxCharges} > 0
            {
                ; Below 30% ammo and not currently active.
                ; Use module.CurrentCharges (int, the loaded-charge count) — NOT
                ; module.Charge, which returns an item object (the charge type
                ; reference, e.g. for Charge.Name / Charge.TypeID lookups) and
                ; would not be meaningful in a numeric comparison.
                if ${m.Value.CurrentCharges} < ${Math.Calc[${m.Value.MaxCharges} * 0.3]} && !${m.Value.IsActive}
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
            ; Check if missile launcher (GroupID compare — numeric, robust)
            if ${m.Value.ToItem.GroupID} == 56  ; 56 = Missile Launcher
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

    ; Find the first active missile launcher and compute its max range from the
    ; loaded charge. Missile range = Charge.MaxVelocity (m/s) * Charge.MaxFlightTime (s).
    ; Charge.MaxVelocity and Charge.MaxFlightTime are only populated when the
    ; charge is currently loaded in a high-slot launcher on your active ship —
    ; see ISXEVE changelog (ISXEVE-20090310.0242, item datatype notes).
    ; This is the canonical production pattern — see EVEBot obj_Ship.iss and
    ; combot obj_ModuleBase.iss for the same formula.
    variable index:module modules
    MyShip:GetModules[modules]
    variable iterator m
    modules:GetIterator[m]
    variable float missileRange = 0

    if ${m:First(exists)}
    {
        do
        {
            ; GroupID 56 = Missile Launcher
            if ${m.Value.ToItem.GroupID} == 56 && ${m.Value.Charge(exists)}
            {
                missileRange:Set[${Math.Calc[${m.Value.Charge.MaxVelocity} * ${m.Value.Charge.MaxFlightTime}]}]
                break
            }
        }
        while ${m:Next(exists)}
    }

    if ${missileRange} <= 0
    {
        echo "CheckMissileRange: no missile launcher with a loaded charge found"
        return FALSE
    }

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
            if ${m.Value.ToItem.GroupID} == 77  ; 77 = Shield Hardener
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
            if ${m.Value.ToItem.GroupID} == 40  ; 40 = Shield Booster
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
            if ${m.Value.ToItem.GroupID} == 328  ; 328 = Armor Hardener
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
            if ${m.Value.ToItem.GroupID} == 62  ; 62 = Armor Repairer
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

## Panic State Machine (Emergency Patterns)

**Reference implementation:** see `EVEBot/Branches/Stable/Behaviors/obj_Miner.iss` and `core/obj_Social.iss`.

A combat bot needs more than one "stop" button. Production bots layer three distinct emergency tiers, each with its own trigger conditions and exit criteria. Conflating them -- e.g. always hard-stopping on any threat -- turns every fleet warp into a 20-minute recovery ordeal. Conversely, trying to finish the current activity when a Dreadnought lands in the belt is how you lose a ship.

### The Three Tiers

| Tier | Intent | Trigger Examples | Recovery |
|---|---|---|---|
| **SOFTSTOP** | Finish current activity cleanly, then stop. | Planned break timer; user-configured quota reached; lag above threshold. | Automatic once activity completes; user resumes manually. |
| **FLEE** | Abandon activity, warp to panic bookmark, idle. | Low-standings pilot in system; NPC Dreadnought / Titan / Invasion NPC visible; unsafe local. | Automatic once threat clears (e.g. pilot leaves local). |
| **HARDSTOP** | Immediate emergency: dock / log out / panic measures. No self-recovery. | Hostiles on grid; ship destroyed, now in pod; critical config error; user pressed break button. | **User intervention required.** |

The key discipline is that tier escalation is one-way during a single threat episode -- if you enter HARDSTOP, `FLEE` conditions that subsequently clear do not pull you back down to `SOFTSTOP`; the user must manually resume.

### Trigger Pattern

The simple form looks like this (derived from the EVEBot Miner behavior):

```lavishscript
; Checked every combat pulse, early in the state machine.
method EvaluatePanicState()
{
    ; --- HARDSTOP: immediate, non-recoverable ---
    if ${This.PossibleHostiles}
    {
        This.CurrentState:Set["HARDSTOP"]
        Logger:Log["HARD STOP: Possible hostiles, notifying fleet"]
        relay all -event MyBot_HARDSTOP "${Me.Name} (Hostiles)"
        This.ReturnToStation:Set[TRUE]
        return
    }

    if ${MyShip.ToEntity.GroupID} == 29  ; Capsule (pod)
    {
        This.CurrentState:Set["HARDSTOP"]
        Logger:Log["HARD STOP: Ship lost, I am in a pod"]
        relay all -event MyBot_HARDSTOP "${Me.Name} (InPod)"
        This.ReturnToStation:Set[TRUE]
        return
    }

    ; If HARDSTOP already latched, only transition out when in station or at panic BM.
    if ${This.ReturnToStation}
    {
        if ${Me.InStation} || ${This.AtPanicBookmark}
        {
            This.CurrentState:Set["IDLE"]
            return
        }
        This.CurrentState:Set["HARDSTOP"]
        return
    }

    ; --- FLEE: abandon activity, warp to panic bookmark ---
    if !${This.LocalSafe}
    {
        if ${This.AtPanicBookmark}
        {
            This.CurrentState:Set["IDLE"]
            return
        }
        This.CurrentState:Set["FLEE"]
        Logger:Log["FLEE: Local unsafe, warping to panic bookmark"]
        return
    }

    ; Big NPCs on grid
    if ${Entity["GroupID = GROUP_DREADNOUGHT && CategoryID = CATEGORYID_ENTITY"](exists)} || \
       ${Entity["GroupID = GROUP_TITAN && CategoryID = CATEGORYID_ENTITY"](exists)}
    {
        This.CurrentState:Set["FLEE"]
        Logger:Log["FLEE: Dreadnought/Titan detected"]
        return
    }

    ; --- SOFTSTOP: planned / quota-based, no escalation needed ---
    if ${This.QuotaReached} || ${This.BreakTimerElapsed}
    {
        This.CurrentState:Set["SOFTSTOP"]
        return
    }

    ; Normal operation
    This.CurrentState:Set["HUNTING"]
}
```

Note the ordering: HARDSTOP checks come first because they are non-recoverable, then the HARDSTOP latch guard, then FLEE, then SOFTSTOP. A single pulse can only transition in one direction along this severity axis.

### The Panic Bookmark

Both FLEE and HARDSTOP reference a **panic bookmark** -- a pre-configured safe bookmark the bot warps to and idles at until further instruction. Structurally it is just a string stored in the bot's config (`Config.Miner.PanicLocation` in EVEBot); the bot reads it at panic time and dispatches through its normal navigation layer:

```lavishscript
; Called from FLEE / HARDSTOP states
method WarpToPanicBookmark()
{
    variable string bmName = ${Config.PanicLocation}
    if !${EVE.Bookmark[${bmName}](exists)}
    {
        Logger:Log["CRITICAL: No panic bookmark configured!"]
        return
    }

    ; Same-system: warp to it (or dock if it's a station bookmark)
    if ${EVE.Bookmark[${bmName}].SolarSystemID} == ${Me.SolarSystemID}
    {
        if ${EVE.Bookmark[${bmName}].TypeID} == 5   ; station bookmark
        {
            call Station.DockAtStation ${EVE.Bookmark[${bmName}].ItemID}
        }
        else
        {
            EVE.Bookmark[${bmName}]:WarpTo[0]
        }
        return
    }

    ; Cross-system: autopilot to the bookmark's system first
    call Ship.TravelToSystem ${EVE.Bookmark[${bmName}].SolarSystemID}
}

member:bool AtPanicBookmark()
{
    variable string bmName = ${Config.PanicLocation}
    if !${EVE.Bookmark[${bmName}](exists)}
        return FALSE
    ; Distance threshold -- within 5 km of the bookmark counts as "there"
    return ${Math.Distance[${Me.ToEntity.X}, ${Me.ToEntity.Y}, ${Me.ToEntity.Z}, ${EVE.Bookmark[${bmName}].X}, ${EVE.Bookmark[${bmName}].Y}, ${EVE.Bookmark[${bmName}].Z}]} < 5000
}
```

Two conventions from EVEBot worth adopting:

- **Bookmark name is user-configurable**, not a hardcoded label. Different ships operate in different regions and need different safes.
- **Station bookmarks dock; celestial bookmarks just warp.** EVE bookmark `TypeID = 5` identifies station types; warping to a station bookmark and stopping at 0 km does not auto-dock.

### Fleet-Wide HARDSTOP via Relay Event

The line `relay all -event MyBot_HARDSTOP "..."` in the trigger pattern above broadcasts a HARDSTOP across every InnerSpace session on the relay network. Receiving sessions latch their own `ReturnToStation` flag, so a single character spotting a hostile pulls the whole fleet to safety automatically.

Setup on the receiving side:

```lavishscript
method Initialize()
{
    LavishScript:RegisterEvent[MyBot_HARDSTOP]
    Event[MyBot_HARDSTOP]:AttachAtom[This:TriggerHARDSTOP]

    LavishScript:RegisterEvent[MyBot_ABORTHARDSTOP]
    Event[MyBot_ABORTHARDSTOP]:AttachAtom[This:AbortHARDSTOP]
}

method TriggerHARDSTOP(string SourceInfo)
{
    Logger:Log["TriggerHARDSTOP called by ${SourceInfo}", LOG_CRITICAL]
    This.ReturnToStation:Set[TRUE]
}

method AbortHARDSTOP()
{
    ; Manual override from the user -- clear the latch.
    This.ReturnToStation:Set[FALSE]
}
```

A paired `MyBot_ABORTHARDSTOP` event lets one character (typically the fleet master) clear the flag on every bot in one command -- useful once the threat has cleared and you want to resume operations without walking to each session.

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
            EVE:Execute[OpenInventory]
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

## Module Control Layer (Production Patterns)

The patterns earlier in this guide call `Module:Activate` and friends directly each pulse. Production bots (Tehbot, combot, EVEBot) instead build a thin abstraction layer between the bot's combat logic and raw module commands. This chapter documents six of the most useful pieces of that layer, all grounded in real production code. Use these when the simple patterns earlier in the guide stop scaling -- typically once you have more than a handful of modules, more than one ship fit, or any need for priority/retry behavior.

### Self-Maintaining Locked-Target Lists (`obj_TargetList`)

Reference implementation: see `Tehbot/core/obj_TargetList.iss`.

A pulse-driven combat bot needs a continuously updated list of "things I am locked onto and may want to shoot," filtered to the current situation (NPCs only, not fleet members, in range, not pod, not CONCORD, not on the bot's exception list, etc.). Tehbot's `obj_TargetList` does this by holding a list of composable query strings, re-running them on a timer, and exposing three ready-to-iterate result indexes.

**Public surface (verified):**

| Member / Method | Purpose |
|---|---|
| `LockedTargetList` (`index:entity`) | Entities currently locked that match the composed query. |
| `LockedAndLockingTargetList` (`index:entity`) | Same, plus entities the lock is being acquired on. |
| `TargetList` (`index:entity`) | All matching candidates (locked or not). |
| `MinLockCount`, `MaxLockCount` (`int`) | Bounds for `AutoLock`. |
| `AutoLock` (`bool`) | When TRUE, the object will issue `:LockTarget` on candidates to reach `MinLockCount`. |
| `MaxRange`, `MinRange` (`int`) | Distance window. Targets outside `MaxRange` get a lower priority but still appear if `ListOutOfRange` is TRUE. |
| `LockTop` (`bool`) | When TRUE, only acquires locks on the top entries by priority order. |
| `ClearQueryString` | Empty the composed query. |
| `AddQueryString[QueryString]` | Append a raw filter string (combined with logical AND across separate calls is not done -- each string is a separate independent query that contributes its matches). |
| `AddTargetingMe` | Convenience: add `Distance < 150000 && IsTargetingMe && IsNPC && !IsMoribund`. |
| `AddPCTargetingMe` | Convenience: same but PCs only, excluding fleet members. |
| `AddAllPC` | Convenience: all PCs in 150 km, excluding fleet members. |
| `AddAllNPCs` | Convenience: all NPCs except common non-combat groups (CONCORD, large collidables, etc.). |
| `AddTargetExceptionByID[int64]` | Permanently exclude a specific entity ID from results, and unlock it if currently locked. |
| `AddTargetExceptionByPartOfName[string]` | Permanently exclude any entity whose name contains the given substring. |
| `ClearExcludeTarget` | Reset both exception sets. |
| `RequestUpdate` | Force the next pulse to re-run all queries. |

`obj_TargetList` inherits from `obj_StateQueue` and pulses on its own (~150 ms). The bot does not poll it -- it just reads `LockedTargetList` whenever it needs to dispatch modules.

**Abstract example -- registering a target list and using its locked output:**

```lavishscript
; Defined as part of an objectdef somewhere in the bot
variable obj_TargetList MyHostileNPCs

method InitializeBot()
{
    ; Compose the filter at startup
    MyHostileNPCs:ClearQueryString
    MyHostileNPCs:AddAllNPCs
    MyHostileNPCs:AddTargetingMe

    ; Auto-lock between 2 and 4 hostiles within 50 km
    MyHostileNPCs.MinLockCount:Set[2]
    MyHostileNPCs.MaxLockCount:Set[4]
    MyHostileNPCs.MaxRange:Set[50000]
    MyHostileNPCs.AutoLock:Set[TRUE]
}

; Called every combat pulse -- iterate the up-to-date locked list
method PulseDispatchModules()
{
    variable iterator t
    MyHostileNPCs.LockedTargetList:GetIterator[t]
    if ${t:First(exists)}
    {
        do
        {
            ; Fire weapons at this locked target. Module-control layer below
            ; (obj_ModuleList) handles the actual dispatch and avoids
            ; double-activation.
            MyOffensiveModules:ActivateOne[${t.Value.ID}]
        }
        while ${t:Next(exists)}
    }
}
```

**Why this beats hand-rolled `Me:GetTargets[...]` per pulse:**

- The list is **debounced and deduped** behind a `RequestUpdate`/`Updated` handshake, so combat code reads stable data instead of racing the lock-acquisition cycle.
- Targets that are dead (`IsMoribund`) or destroyed are removed via a 5-second dead-delay, eliminating the "fired at a corpse" failure mode.
- Multiple separately-tagged query strings can co-exist: e.g. one filter for NPCs targeting me, another for priority frigates, a third for healing buddies in the fleet -- each contributes its own matches with its own `Priority` weight, and `ManageLocks` resolves contention automatically.
- Adding/removing exceptions at runtime (`AddTargetExceptionByID`, `AddTargetExceptionByPartOfName`) does not require rebuilding the query string -- the object honors the exception sets on every refresh and unlocks newly-excluded entities.

If your bot has more than one combat role (mission runner, anomaly clearer, defender) you typically declare one `obj_TargetList` per role, populate each with the appropriate convenience methods, and let them all run in parallel.

---

### Per-Module Instruction Queue (`obj_Module`)

Reference implementation: see `Tehbot/core/obj_Module.iss`.

Calling `MyShip.Module[HiSlot0]:Activate[${targetID}]` directly each pulse has well-known failure modes: double-activation racing the cycle timer, retry-spam when the server lags, ammo swaps mid-cycle that get ignored, overload toggles that flip back and forth. Tehbot wraps each module in an `obj_Module` object that owns a single pending **instruction**, drains it on a state-queue pulse (~100 ms with random delta), and uses retry timers to avoid command spam.

**Instruction enum (verified, from `Tehbot/core/Defines.iss`):**

| Constant | Value | Meaning |
|---|---|---|
| `INSTRUCTION_NONE` | 0 | No pending action; module is idle (but may still process overload toggling). |
| `INSTRUCTION_ACTIVATE_ON` | 1 | Activate against a specific target; deactivate-and-reactivate if target changes. |
| `INSTRUCTION_DEACTIVATE` | 2 | Deactivate the module. |
| `INSTRUCTION_RELOAD_AMMO` | 3 | Reload current charge type. |
| `INSTRUCTION_ACTIVATE_FOR` | 4 | Activate self-targeted (shield boosters, armor reps, etc.). `targetID` is `MyShip.ID` semantically, passed as `TARGET_NA = 0`. |
| `INSTRUCTION_ACTIVATE_ONCE` | 5 | One-shot activation; clears instruction after a single fire. |

`TARGET_NA = 0` and `TARGET_ANY = 0` are sentinel values used for self-targeted or "any target" instructions.

**Public surface (verified):**

| Member / Method | Purpose |
|---|---|
| `GiveInstruction[instruction, targetID]` | Submit a new instruction. No-op if the same instruction+target is already pending and the module is mid-cycle. |
| `IsInstructionMatch[instruction, targetID]` | TRUE iff this is the pending instruction. |
| `IsModuleActiveOn[targetID]` | TRUE iff `IsActive` AND the module's current target matches. |
| `OverloadIfHPAbovePercent` (`int`, default 100) | When the module's own HP is above this percent, the instruction loop will toggle overload on; when at/below, toggle it off. (See "Module Overload by Module HP" below.) |
| `ConfigureAmmo[shortRange, longRange]` | Set the two ammo types the module should pick from. The `Operate` state picks the optimal one per target via `_pickOptimalAmmo`. |
| `Ammo`, `LongRangeAmmo` (`string`) | Direct access to the configured ammo names. |
| `ModuleID` (`int64`) | The underlying module item ID (resolved via `MyShip.Module[${ModuleID}]` through `GetFallthroughObject`). |

The state-queue pulse calls a `member:bool Operate()` that:

1. Bails out when the ship is in station, the module is offline, being repaired, reloading, or `_tooSoon` (within the per-instruction retry interval, typically 2 seconds).
2. Switches on the pending `Instruction` and dispatches to the appropriate `OperateActivateOn` / `OperateDeactivate` / `OperateReloadAmmo` / `OperateActivateFor` handler.
3. Even at `INSTRUCTION_NONE`, runs `_toggleOverload` so overload state tracks `OverloadIfHPAbovePercent`.

**Why this beats raw `Module:Activate`:**

- The retry interval (`_activationRetryInterval = 2000`, `_deactivationRetryInterval = 2000`, `_changeAmmoRetryInterval = 2000`) prevents the bot from spamming `:Activate` every pulse while the server has not yet acknowledged the previous attempt. Without this, modules look "stuck off" for several pulses then suddenly all activate at once.
- `IsInstructionMatch` is the no-op guard: `GiveInstruction` checks "did you already ask for this?" before mutating state, so combat code can call `module:GiveInstruction[INSTRUCTION_ACTIVATE_ON, ${activeTarget}]` every pulse without consequence.
- Target switches are handled gracefully: `OperateActivateOn` first deactivates if the current target does not match, then re-activates on the new target -- preserving cycle progress when possible.
- Overload management piggybacks on the same pulse loop, so combat code never has to know about it.

**Abstract example -- registering a module and keeping it on the active target:**

```lavishscript
variable obj_Module MyHardener

method InitializeBot()
{
    ; Bind the wrapper to a specific module ID at undock time.
    MyHardener:Initialize[${MyShip.Module[MedSlot0].ID}]
    MyHardener.OverloadIfHPAbovePercent:Set[80]  ; only overload if heat HP > 80%
}

method PulseDefenseLayer()
{
    ; Self-targeted activation -- pass TARGET_NA.
    if ${MyShip.ShieldPct} < 50
    {
        MyHardener:GiveInstruction[INSTRUCTION_ACTIVATE_FOR, 0]  ; TARGET_NA
    }
    else
    {
        MyHardener:GiveInstruction[INSTRUCTION_DEACTIVATE]
    }
}
```

In production you almost never instantiate `obj_Module` directly -- you let `obj_Ship` build them as a side effect of `AddModuleList` (see "Declarative Module Registration" below) and dispatch through `obj_ModuleList` group methods.

---

### Group Control over Module Wrappers (`obj_ModuleList`)

Reference implementation: see `Tehbot/core/obj_ModuleList.iss`.

`obj_ModuleList` is a thin index of module IDs that fans `GiveInstruction` calls out to the underlying `obj_Module` wrappers. It exists so combat code can talk to "all my offensive modules" or "all my repair modules" in one call instead of iterating manually.

**Public surface (verified):**

| Member / Method | Purpose |
|---|---|
| `Insert[int64 ID]` | Add a module ID to the group. |
| `ActivateAll[targetID = 0]` | `INSTRUCTION_ACTIVATE_ON` to every member. |
| `ActivateOne[targetID = 0]` | `INSTRUCTION_ACTIVATE_ON` to the first idle member only. |
| `ForceActivateOne[targetID = 0]` | Re-target the first member to `targetID` even if currently busy on a different target. |
| `ActivateFor[targetID = 0]` | `INSTRUCTION_ACTIVATE_FOR` (self-targeted) to every member. |
| `DeactivateAll` | `INSTRUCTION_DEACTIVATE` to every member that is currently active. |
| `DeactivateOn[targetID]` | Deactivate every member currently active on `targetID`. |
| `DeactivateOneNotOn[targetID]` | Deactivate one member that is active on something other than `targetID` (target switch helper). |
| `ReloadDefaultAmmo` | `INSTRUCTION_RELOAD_AMMO` to every member. |
| `ConfigureAmmo[shortRange, longRange]` | Push the same ammo configuration to every member. |
| `SetOverloadHPThreshold[int threshold]` | Set `OverloadIfHPAbovePercent` on every member. |
| `IsActiveOn[targetID]` (`bool`) | TRUE iff any member has a pending or active `INSTRUCTION_ACTIVATE_ON` for `targetID`. |
| `Count` (`int`) | Number of registered modules. |
| `ActiveCount` (`int`) | Number of members where the underlying module reports `IsActive`. |
| `ActiveCountOn[targetID]` (`int`) | Number of members currently active on `targetID`. |
| `InactiveCount` (`int`) | Number of members at `INSTRUCTION_NONE`. |
| `Allowed` (`bool`, default TRUE) | When FALSE, all `Activate*` calls are no-ops. Use to soft-disable a group from external code. |

Several "fallthrough" members (`Range`, `OptimalRange`, `TrackingSpeed`, `AccuracyFalloff`, `DamageEfficiency[targetID]`, `FallbackAmmo`, `IsUsingLongRangeAmmo`) read off the **first** registered module, on the assumption that grouped modules in EVE share fit attributes.

**Abstract example -- combat dispatch via grouped wrappers:**

```lavishscript
; Registered elsewhere (see "Declarative Module Registration" next).
; Ship.ModuleList_Turret holds the bot's primary turret group.
; Ship.ModuleList_Repair_Shield holds shield boosters.

method PulseCombatDispatch(int64 primaryTargetID)
{
    ; Fire turrets at the primary target -- safe to call every pulse.
    Ship.ModuleList_Turret:ActivateAll[${primaryTargetID}]

    ; Self-repair if shields below threshold; otherwise stop.
    if ${MyShip.ShieldPct} < 60
    {
        Ship.ModuleList_Repair_Shield:ActivateFor[0]  ; TARGET_NA
    }
    else
    {
        Ship.ModuleList_Repair_Shield:DeactivateAll
    }
}
```

---

### Declarative Module Registration (`AddModuleList`)

Reference implementation: see `combot/core/obj_Ship.iss` (also inherited/used in `Tehbot/core/obj_Ship.iss`).

`obj_Ship:AddModuleList[Name, QueryString]` is a small piece of LavishScript metaprogramming: it pre-compiles a query against the ship's module list and creates an `obj_ModuleList` member named `ModuleList_<Name>`, populated automatically every time the bot redocks and re-fits.

The implementation is three lines:

```lavishscript
method AddModuleList(string Name, string QueryString)
{
    This.ModuleLists:Insert[${Name}]
    This.ModuleQueries:Set[${This.ModuleLists.Used}, ${LavishScript.CreateQuery[${QueryString.Escape}]}]
    declarevariable ModuleList_${Name} obj_ModuleList object
    This:Clear
    This:QueueState["WaitForSpace"]
    This:QueueState["UpdateModules"]
}
```

Two LavishScript features make this work:

- **`LavishScript.CreateQuery[expr]`** -- compiles a query expression into a query handle, then `LavishScript.QueryEvaluate[handle, candidate]` evaluates it against any object. Cheaper than re-parsing the string per module per pulse.
- **`declarevariable Name Type Scope`** -- declares a new variable at runtime, using a name composed via `${...}` substitution. After this call, `This.ModuleList_Turret` exists as an `obj_ModuleList` object and can be referenced by name from anywhere.

The companion `UpdateModules` state runs `MyShip:GetModules[ModuleList]`, then for each module evaluates every registered query against it via `LavishScript.QueryEvaluate` and inserts matches into the corresponding `ModuleList_<Name>`. Result: dynamic per-fit module classification driven entirely by query strings declared at startup.

**Abstract example -- one-time registration at undock:**

```lavishscript
; Inside obj_Ship:Initialize or an undock handler
method RegisterModuleGroups()
{
    This:AddModuleList[Turret, "ToItem.GroupID = GROUP_ENERGYWEAPON || ToItem.GroupID = GROUP_PROJECTILEWEAPON || ToItem.GroupID = GROUP_HYBRIDWEAPON"]
    This:AddModuleList[Repair_Shield, "ToItem.GroupID = GROUP_SHIELD_BOOSTER"]
    This:AddModuleList[ActiveResists, "ToItem.GroupID = GROUP_SHIELD_HARDENER || ToItem.GroupID = GROUP_ARMOR_HARDENERS"]
    This:AddModuleList[TractorBeams, "ToItem.GroupID = GROUP_TRACTOR_BEAM"]
    ; ...etc
}

; Anywhere afterward
Ship.ModuleList_Turret:ActivateAll[${primary}]
Ship.ModuleList_Repair_Shield:ActivateFor[0]
```

The `declarevariable + ${LavishScript.CreateQuery}` idiom is reusable beyond modules: any time a script needs a named, query-filtered, dynamically-registered collection (entity pools, drone pools, item-cache buckets), the same three-line pattern applies. Substitute `index:entity`, `index:item`, etc. for the underlying type, and use `EVE:QueryEntities` or whatever populator fits.

---

### Module Overload by Module HP

Reference implementation: see `Tehbot/core/obj_Module.iss` (`OverloadIfHPAbovePercent` and `_toggleOverload`).

Overload management in production scripts is driven by the **module's own HP**, not the ship's shield/armor. The reasoning: a module with full overload HP can be safely flipped on for the burst cycle and let cool when HP drops, but coupling it to ship HP confuses the burn-in/burn-out economy.

The relevant state on each `obj_Module`:

- `OverloadIfHPAbovePercent` (`int`, default `100`) -- when the module's HP percentage is **strictly greater than** this threshold, the per-module pulse will issue `ToggleOverload` to enable overheat. When HP is at or below the threshold, it issues `ToggleOverload` again to disable.
- `_overloadToggledOn` (`bool`) -- internal latch tracking whether the script believes overload is currently on. Prevents redundant toggles.

The default of `100` means "never overload" -- you must explicitly lower the threshold to opt in. A value of, say, `80` means "stay overloaded as long as the module's HP is above 80%; cool off below."

`obj_ModuleList:SetOverloadHPThreshold[int]` is the convenience setter that pushes a single threshold to every module in a group.

**Abstract example -- overload guns above 75% module HP:**

```lavishscript
method InitializeOverload()
{
    ; Aggressive: keep guns overloaded as long as racks are above 75% HP
    Ship.ModuleList_Turret:SetOverloadHPThreshold[75]

    ; Cautious: only overload prop mod when HP is essentially full
    Ship.ModuleList_AB_MWD:SetOverloadHPThreshold[95]
}
```

Once configured, the per-module pulse loop handles toggling on its own; combat code never needs to call overload on/off explicitly.

---

### Static Module Classification on Init (EVEBot Stable)

Reference implementation: see `EVEBot/Branches/Stable/core/obj_Ship.iss` (`UpdateModuleList`).

EVEBot Stable predates the `AddModuleList` pattern and uses a more conservative but equally valid approach: declare every `index:module` field statically, then populate them once per fit with a single iteration over `MyShip:GetModules`, dispatching by `GroupID` and `TypeID`.

The pattern is three pieces:

1. **Static field declarations** at object scope -- one `variable index:module ModuleList_<X>` for each pool the bot recognizes (Weapon, MiningLaser, ActiveResists, Repair_Armor, AB_MWD, Salvagers, TractorBeams, Cloaks, etc.).
2. **A `Clear` block** that empties every pool before re-populating, called on undock or when modules change.
3. **A single `GetModules` iteration** with a `switch ${GroupID}` (and special-case checks like `MiningAmount(exists)`) that inserts each module into the correct pool. Dual-pool insertion is allowed if the module logically belongs in two groups.

**Abstract example with two representative pools:**

```lavishscript
objectdef obj_Ship
{
    variable index:module ModuleList
    variable index:module ModuleList_Weapon
    variable index:module ModuleList_Repair_Shield

    method UpdateModuleList()
    {
        ; Clear
        This.ModuleList:Clear
        This.ModuleList_Weapon:Clear
        This.ModuleList_Repair_Shield:Clear

        ; Populate the master list
        if !${MyShip:GetModules[This.ModuleList]}
        {
            return
        }

        ; Single pass classification
        variable iterator m
        This.ModuleList:GetIterator[m]
        if ${m:First(exists)}
        {
            do
            {
                if !${m.Value(exists)}
                    continue

                variable int gid = ${m.Value.ToItem.GroupID}
                switch ${gid}
                {
                    case 53   ; Energy Weapon
                    case 55   ; Projectile Weapon
                    case 74   ; Hybrid Weapon
                        This.ModuleList_Weapon:Insert[${m.Value.ID}]
                        break

                    case 40   ; Shield Booster
                        This.ModuleList_Repair_Shield:Insert[${m.Value.ID}]
                        break
                }
            }
            while ${m:Next(exists)}
        }
    }
}
```

**When to prefer this over `AddModuleList`:**

- You want compile-time visibility of every module group your bot understands (no metaprogramming, no `declarevariable`).
- You only support a fixed set of fits and the GroupID/TypeID dispatch is well-defined.
- You need to encode multi-attribute classification (e.g. "shield booster but not capital-class") that does not fit naturally into a single query string.

**When to prefer `AddModuleList`:**

- The bot must support an open-ended set of user-configured module roles.
- You want one-line registration of new groups without touching the iterator.
- Your filters fit naturally as query strings against the `module` datatype's accessible members.

Both approaches end with the same shape: typed `index:module` (or `obj_ModuleList`) pools that combat code iterates per pulse, never calling `MyShip:GetModules` in the hot path.

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
