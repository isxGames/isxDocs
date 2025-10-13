# Decision Making and Logic Patterns - Bot Intelligence Reference

**Purpose**: This file documents how bots make decisions - target selection, priority systems, condition checking, and intelligent behavior. This is where your bot becomes "smart" rather than just mechanical.

**Target Reader**: AI learning to build intelligent EVE bots

**Prerequisites**:
- Files 01-17 (Foundation, API, Main Loops)
- Understanding of boolean logic
- Knowledge of entity queries and filtering

**Wiki References**:
- Entity queries: `IsxeveWiki/DataType/eve.html#QueryEntities`
- Boolean logic: `LavishScriptWiki/LavishScript/Data_Sequences.html`
- Math operations: `LavishScriptWiki/TopLevelObjects/Math.html`

**⚠️ API Note:** Some examples use simplified cargo checks (`MyShip.CargoFreeSpace`) to illustrate decision logic. For production code, use modern inventory API: `EVEWindow[Inventory].Child[ShipCargo].FreeSpace`. See Files 13 and 15 for modern cargo handling.

---

## Table of Contents

1. [Decision-Making Overview](#decision-making-overview)
2. [Target Selection Patterns](#target-selection-patterns)
3. [Priority Systems](#priority-systems)
4. [Condition Checking Patterns](#condition-checking-patterns)
5. [Decision Trees](#decision-trees)
6. [Weighted Scoring Systems](#weighted-scoring-systems)
7. [Real Examples from Bots](#real-examples-from-bots)
8. [Mining Decision Logic](#mining-decision-logic)
9. [Combat Decision Logic](#combat-decision-logic)
10. [Movement Decision Logic](#movement-decision-logic)
11. [Common Patterns](#common-patterns)
12. [Performance Considerations](#performance-considerations)

---

## Decision-Making Overview

Bot intelligence comes from **making good decisions** based on **available information**. The decision-making process follows this flow:

```
┌──────────────┐
│ Gather Data  │ Query entities, check ship status, read environment
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Filter Data  │ Remove invalid/irrelevant options
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Prioritize   │ Rank remaining options by importance
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Choose Best  │ Select highest priority option
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Execute      │ Perform chosen action
└──────────────┘
```

### Key Principles

1. **Safety First**: Always check hostile/damage conditions before normal operations
2. **Priority Order**: Process high-priority options before low-priority
3. **Validity Checking**: Ensure chosen option still exists/is valid before acting
4. **State Awareness**: Different decisions in different states (idle vs active vs fleeing)
5. **Performance**: Don't re-query/re-calculate unnecessarily

---

## Target Selection Patterns

Target selection is the most common decision-making task for bots. The pattern is:

1. **Query** all potential targets
2. **Filter** to valid targets only
3. **Prioritize** by importance/threat/value
4. **Select** highest priority target
5. **Validate** before locking

### Pattern 1: Simple Closest Target

```lavish
function GetClosestNPC()
{
    variable index:entity NPCs
    variable iterator NPC

    ; Query all NPCs within range
    EVE:QueryEntities[NPCs, "CategoryID = CATEGORYID_ENTITY && IsNPC && !IsMoribund && Distance < ${MyShip.MaxTargetRange}"]

    NPCs:GetIterator[NPC]

    ; QueryEntities automatically sorts by distance
    ; First result is closest
    if ${NPC:First(exists)}
    {
        return ${NPC.Value.ID}
    }

    return 0  ; No target found
}
```

**Use Case**: Simple mining bots, basic combat bots

### Pattern 2: Priority List Matching

```lavish
; ===== PRIORITY TARGET PATTERN =====
; Based on EVEBot obj_Targets.iss

variable(global) index:string PriorityTargets
variable(global) index:string NormalTargets

function LoadTargetPriorities()
{
    ; Priority targets (scramblers, webifiers, jammers) - kill FIRST
    PriorityTargets:Insert["Dire Pithi Arrogator"]  ; web/scram
    PriorityTargets:Insert["Dire Pithi Imputor"]    ; web/scram
    PriorityTargets:Insert["Dire Pithi Infiltrator"]; web/scram
    PriorityTargets:Insert["Dire Pithi Saboteur"]   ; jamming
    PriorityTargets:Insert["Guardian Agent"]        ; web/scram
    PriorityTargets:Insert["Guardian Scout"]        ; web/scram

    ; Normal targets - kill after priority
    NormalTargets:Insert["Pithi Arrogator"]
    NormalTargets:Insert["Pithi Infiltrator"]
    NormalTargets:Insert["Guristas Imputor"]
}

function GetBestCombatTarget()
{
    variable index:entity Entities
    variable iterator Entity

    ; Query all hostile NPCs
    EVE:QueryEntities[Entities, "CategoryID = CATEGORYID_ENTITY && IsNPC && !IsMoribund && Distance < ${MyShip.MaxTargetRange}"]
    Entities:GetIterator[Entity]

    if !${Entity:First(exists)}
        return 0

    ; PASS 1: Look for priority targets
    do
    {
        variable int i
        for (i:Set[1]; ${i} <= ${PriorityTargets.Used}; i:Inc)
        {
            if ${Entity.Value.Name.Find["${PriorityTargets[${i}]}"]}
            {
                echo "PRIORITY TARGET: ${Entity.Value.Name}"
                return ${Entity.Value.ID}
            }
        }
    }
    while ${Entity:Next(exists)}

    ; PASS 2: No priority targets, take closest normal target
    Entity:First

    return ${Entity.Value.ID}
}
```

**EVEBot Real Example** (obj_Targets.iss, lines 92-115):

```lavish
PriorityTargets:Insert["Dire Pithi Destructor"]
PriorityTargets:Insert["Dire Pithi Wrecker"]
PriorityTargets:Insert["Dire Pithi Plunderer"]
PriorityTargets:Insert["Factory Defense Battery"] 		/* web/scram */
PriorityTargets:Insert["Dire Pithi Arrogator"] 		/* web/scram */
PriorityTargets:Insert["Dire Pithi Despoiler"] 		/* Jamming */
PriorityTargets:Insert["Dire Pithi Imputor"] 		/* web/scram */
PriorityTargets:Insert["Dire Pithi Infiltrator"] 	/* web/scram */
PriorityTargets:Insert["Dire Pithi Invader"] 		/* web/scram */
PriorityTargets:Insert["Dire Pithi Saboteur"] 		/* Jamming */
```

### Pattern 3: Weighted Scoring

```lavish
; ===== WEIGHTED SCORING PATTERN =====

function GetBestTargetScored()
{
    variable index:entity Entities
    variable iterator Entity

    ; Query all NPCs
    EVE:QueryEntities[Entities, "CategoryID = CATEGORYID_ENTITY && IsNPC && !IsMoribund && Distance < ${MyShip.MaxTargetRange}"]
    Entities:GetIterator[Entity]

    if !${Entity:First(exists)}
        return 0

    variable int64 BestTarget = 0
    variable float BestScore = 0

    ; Score each entity
    do
    {
        variable float score = 0

        ; Distance (closer = higher score)
        score:Set[${Math.Calc[10000 / ${Entity.Value.Distance}]}]

        ; Shield %  (lower shields = higher score, prioritize damaged)
        if ${Entity.Value.ShieldPct} < 50
        {
            score:Inc[${Math.Calc[100 - ${Entity.Value.ShieldPct}]}]
        }

        ; Ship size (frigates = 100, cruisers = 50, battleships = 10)
        switch ${Entity.Value.GroupID}
        {
            case ${GROUPID_FRIGATES}
                score:Inc[100]
                break
            case ${GROUPID_CRUISERS}
                score:Inc[50]
                break
            case ${GROUPID_BATTLESHIPS}
                score:Inc[10]
                break
        }

        ; Priority target bonus
        if ${IsPriorityTarget["${Entity.Value.Name}"]}
        {
            score:Inc[1000]  ; Huge bonus!
        }

        ; Already locked penalty (don't re-pick same target)
        if ${Entity.Value.IsLockedTarget} || ${Entity.Value.BeingTargeted}
        {
            score:Dec[500]
        }

        ; Update best
        if ${score} > ${BestScore}
        {
            BestScore:Set[${score}]
            BestTarget:Set[${Entity.Value.ID}]
        }
    }
    while ${Entity:Next(exists)}

    if ${BestTarget} > 0
    {
        echo "Best target: ${Entity[${BestTarget}].Name} (score: ${BestScore})"
    }

    return ${BestTarget}
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

### Pattern 4: Query String Building

**Tehbot Style** - Build complex query strings dynamically

```lavish
; ===== DYNAMIC QUERY STRING BUILDING =====
; Based on Tehbot obj_TargetList.iss

objectdef obj_TargetSelector
{
    variable index:string QueryStringList
    variable index:entity ResultEntities
    variable set ExcludeTargetID
    variable set ExcludeNameParts
    variable int MaxRange = 20000
    variable int MinRange = 0

    method ClearQueryStrings()
    {
        QueryStringList:Clear
    }

    method AddQueryString(string QueryString)
    {
        QueryStringList:Insert["${QueryString.Escape}"]
    }

    method AddAllNPCs()
    {
        variable string QueryString = "CategoryID = CATEGORYID_ENTITY && IsNPC && !IsMoribund && !("

        ; Exclude specific groups (concord, convoy, structures, etc.)
        QueryString:Concat["GroupID = GROUP_CONCORDDRONE ||"]
        QueryString:Concat["GroupID = GROUP_CONVOYDRONE ||"]
        QueryString:Concat["GroupID = GROUP_CONVOY ||"]
        QueryString:Concat["GroupID = GROUP_LARGECOLLIDABLEOBJECT ||"]
        QueryString:Concat["GroupID = GROUP_LARGECOLLIDABLESHIP ||"]
        QueryString:Concat["GroupID = GROUP_SPAWNCONTAINER ||"]
        QueryString:Concat["GroupID = GROUP_DEADSPACEOVERSEERSSTRUCTURE ||"]
        QueryString:Concat["GroupID = GROUP_LARGECOLLIDABLESTRUCTURE"]
        QueryString:Concat[")"]

        This:AddQueryString["${QueryString.Escape}"]
    }

    method AddTargetingMe()
    {
        This:AddQueryString["Distance < 150000 && IsTargetingMe && IsNPC && !IsMoribund"]
    }

    method AddPCTargetingMe()
    {
        This:AddQueryString["Distance < 150000 && IsTargetingMe && !IsFleetMember && IsPC && !IsMoribund"]
    }

    method AddExcludeByID(int64 EntityID)
    {
        ExcludeTargetID:Add[${EntityID}]
    }

    method AddExcludeByNamePart(string NamePart)
    {
        ExcludeNameParts:Add["${NamePart}"]
    }

    method UpdateTargetList()
    {
        ResultEntities:Clear

        variable iterator QueryIterator
        QueryStringList:GetIterator[QueryIterator]

        ; Execute each query
        if ${QueryIterator:First(exists)}
            do
            {
                variable index:entity TempEntities
                variable iterator Entity

                EVE:QueryEntities[TempEntities, "${QueryIterator.Value.Escape}"]
                TempEntities:GetIterator[Entity]

                ; Add to result list with filtering
                if ${Entity:First(exists)}
                    do
                    {
                        ; Filter by range
                        if ${Entity.Value.Distance} < ${MinRange} || ${Entity.Value.Distance} > ${MaxRange}
                            continue

                        ; Filter by excluded IDs
                        if ${ExcludeTargetID.Contains[${Entity.Value.ID}]}
                            continue

                        ; Filter by excluded name parts
                        variable bool excludeByName = FALSE
                        variable iterator NamePart
                        ExcludeNameParts:GetIterator[NamePart]

                        if ${NamePart:First(exists)}
                            do
                            {
                                if ${Entity.Value.Name.Find["${NamePart.Value}"]}
                                {
                                    excludeByName:Set[TRUE]
                                    break
                                }
                            }
                            while ${NamePart:Next(exists)}

                        if ${excludeByName}
                            continue

                        ; Add to result
                        ResultEntities:Insert[${Entity.Value}]
                    }
                    while ${Entity:Next(exists)}
            }
            while ${QueryIterator:Next(exists)}
    }

    member:int64 GetBestTarget()
    {
        call This.UpdateTargetList

        ; Results are sorted by distance by default
        if ${ResultEntities.Used} > 0
        {
            return ${ResultEntities.Get[1].ID}
        }

        return 0
    }
}
```

**Tehbot Real Example** (obj_TargetList.iss, lines 130-177):

```lavish
method AddAllNPCs()
{
    variable string QueryString="CategoryID = CATEGORYID_ENTITY && IsNPC && !IsMoribund && !("

    ; Abyssal switch for ignoring the really distant cans if we aren't using an MTU
    if !${Abyssal.Config.UseMTU}
    {
        QueryString:Concat["TypeID = 49663 ||"]
        QueryString:Concat["TypeID = 49662 ||"]
        QueryString:Concat["TypeID = 49661 ||"]
    }
    ;Exclude Groups here
    QueryString:Concat["GroupID = GROUP_CONCORDDRONE ||"]
    QueryString:Concat["GroupID = GROUP_CONVOYDRONE ||"]
    QueryString:Concat["GroupID = GROUP_CONVOY ||"]
    QueryString:Concat["GroupID = GROUP_LARGECOLLIDABLEOBJECT ||"]
    QueryString:Concat["GroupID = GROUP_LARGECOLLIDABLESHIP ||"]
    QueryString:Concat["GroupID = GROUP_SPAWNCONTAINER ||"]
    QueryString:Concat["GroupID = CATEGORYID_ORE ||"]
    QueryString:Concat["GroupID = GROUP_DEADSPACEOVERSEERSSTRUCTURE ||"]
    QueryString:Concat["GroupID = GROUP_LARGECOLLIDABLESTRUCTURE ||"]
    ; ... more exclusions ...

    This:AddQueryString["${QueryString.Escape}"]
}
```

---

## Priority Systems

Priority systems allow bots to handle multiple concerns simultaneously.

### Pattern 1: Layered Priority

```lavish
; ===== LAYERED PRIORITY PATTERN =====

function GetNextAction()
{
    ; LAYER 1: SAFETY (highest priority)
    if ${CheckForHostiles}
    {
        echo "ACTION: Flee (hostiles detected)"
        return "FLEE"
    }

    if ${MyShip.ShieldPct} < 30
    {
        echo "ACTION: Dock (low shields)"
        return "DOCK"
    }

    ; LAYER 2: CRITICAL OPERATIONS
    if ${MyShip.CargoFreeSpace} < 50
    {
        echo "ACTION: Haul (cargo full)"
        return "HAUL"
    }

    ; LAYER 3: NORMAL OPERATIONS
    if ${GetNearbyNPCCount} > 0 && ${BotMode.Equal["Combat"]}
    {
        echo "ACTION: Engage (NPCs found)"
        return "COMBAT"
    }

    if ${GetNearbyAsteroidCount} > 0 && ${BotMode.Equal["Mining"]}
    {
        echo "ACTION: Mine (asteroids found)"
        return "MINE"
    }

    ; LAYER 4: IDLE
    echo "ACTION: Idle (nothing to do)"
    return "IDLE"
}
```

### Pattern 2: Numeric Priority

```lavish
; ===== NUMERIC PRIORITY PATTERN =====

objectdef obj_Task
{
    variable string Name
    variable int Priority  ; Higher = more important
    variable string Action

    method Initialize(string name, int priority, string action)
    {
        This.Name:Set["${name}"]
        This.Priority:Set[${priority}]
        This.Action:Set["${action}"]
    }
}

variable(global) index:obj_Task TaskList

function AddTask(string name, int priority, string action)
{
    TaskList:Insert[${name},${priority},${action}]
}

function GetHighestPriorityTask()
{
    if ${TaskList.Used} == 0
        return ""

    variable iterator Task
    TaskList:GetIterator[Task]

    variable string BestTask = ""
    variable int BestPriority = 0

    if ${Task:First(exists)}
        do
        {
            if ${Task.Value.Priority} > ${BestPriority}
            {
                BestPriority:Set[${Task.Value.Priority}]
                BestTask:Set["${Task.Value.Action}"]
            }
        }
        while ${Task:Next(exists)}

    return "${BestTask}"
}

; Usage example
function BotPulse()
{
    TaskList:Clear

    ; Add tasks with priorities
    if ${CheckForHostiles}
    {
        call AddTask "Flee" 1000 "FLEE"
    }

    if ${MyShip.ShieldPct} < 30
    {
        call AddTask "Dock" 900 "DOCK"
    }

    if ${MyShip.CargoFreeSpace} < 50
    {
        call AddTask "Haul" 500 "HAUL"
    }

    if ${GetNearbyNPCCount} > 0
    {
        call AddTask "Combat" 300 "COMBAT"
    }

    ; Get highest priority task
    variable string nextAction = ${GetHighestPriorityTask}

    switch ${nextAction}
    {
        case FLEE
            call EmergencyFlee
            break
        case DOCK
            call DockForRepairs
            break
        case HAUL
            call ReturnToStation
            break
        case COMBAT
            call EngageCombat
            break
    }
}
```

### Pattern 3: Priority Queues

```lavish
; ===== PRIORITY QUEUE PATTERN =====

variable(global) index:int64 PriorityTargets
variable(global) index:int64 NormalTargets
variable(global) index:int64 LowPriorityTargets

function CategorizeTargets()
{
    PriorityTargets:Clear
    NormalTargets:Clear
    LowPriorityTargets:Clear

    variable index:entity Entities
    variable iterator Entity

    EVE:QueryEntities[Entities, "CategoryID = CATEGORYID_ENTITY && IsNPC && !IsMoribund"]
    Entities:GetIterator[Entity]

    if ${Entity:First(exists)}
        do
        {
            ; Categorize by name matching
            if ${Entity.Value.Name.Find["Dire"]} || ${Entity.Value.Name.Find["Guardian"]}
            {
                PriorityTargets:Insert[${Entity.Value.ID}]
            }
            elseif ${Entity.Value.Name.Find["Hauler"]} || ${Entity.Value.Name.Find["Transport"]}
            {
                LowPriorityTargets:Insert[${Entity.Value.ID}]
            }
            else
            {
                NormalTargets:Insert[${Entity.Value.ID}]
            }
        }
        while ${Entity:Next(exists)}

    echo "Targets - Priority: ${PriorityTargets.Used}, Normal: ${NormalTargets.Used}, Low: ${LowPriorityTargets.Used}"
}

function GetNextTarget()
{
    call CategorizeTargets

    ; Take from priority queue first
    if ${PriorityTargets.Used} > 0
    {
        return ${PriorityTargets.Get[1]}
    }

    ; Then normal queue
    if ${NormalTargets.Used} > 0
    {
        return ${NormalTargets.Get[1]}
    }

    ; Finally low priority queue
    if ${LowPriorityTargets.Used} > 0
    {
        return ${LowPriorityTargets.Get[1]}
    }

    return 0
}
```

---

## Condition Checking Patterns

### Pattern 1: Guard Clauses

```lavish
; ===== GUARD CLAUSE PATTERN =====

function ProcessCombat()
{
    ; Guard clauses - early returns for invalid states
    if !${ISXEVE.IsReady}
    {
        echo "ISXEVE not ready"
        return
    }

    if !${Me.InSpace}
    {
        echo "Not in space"
        return
    }

    if ${Me.InWarp}
    {
        echo "In warp, waiting"
        return
    }

    if ${MyShip.ToEntity.IsCloaked}
    {
        echo "Cloaked, cannot engage"
        return
    }

    ; All checks passed, do combat
    call FindAndEngageTargets
}
```

### Pattern 2: Compound Conditions

```lavish
; ===== COMPOUND CONDITION PATTERN =====

; BAD: Nested ifs (hard to read)
if ${Me.InSpace}
{
    if ${ISXEVE.IsReady}
    {
        if !${Me.InWarp}
        {
            if ${MyShip.ShieldPct} > 50
            {
                call ProcessCombat
            }
        }
    }
}

; GOOD: Compound condition with AND operator
if ${Me.InSpace} && \
   ${ISXEVE.IsReady} && \
   !${Me.InWarp} && \
   ${MyShip.ShieldPct} > 50
{
    call ProcessCombat
}

; GOOD: Stored in boolean variable
variable bool CanEngage = ${Me.InSpace} && ${ISXEVE.IsReady} && !${Me.InWarp} && ${MyShip.ShieldPct} > 50

if ${CanEngage}
{
    call ProcessCombat
}
```

### Pattern 3: Condition Functions

```lavish
; ===== CONDITION FUNCTION PATTERN =====

function IsSafeToOperate()
{
    ; Check all safety conditions
    if !${ISXEVE.IsReady}
        return FALSE

    if !${Me(exists)} || !${MyShip(exists)}
        return FALSE

    if ${CheckForHostiles}
        return FALSE

    if ${MyShip.ShieldPct} < 30
        return FALSE

    if ${Me.InWarp}
        return FALSE

    return TRUE
}

function CanMine()
{
    if !${IsSafeToOperate}
        return FALSE

    if !${Me.InSpace}
        return FALSE

    if ${MyShip.CargoFreeSpace} < 100
        return FALSE

    if ${GetNearbyAsteroidCount} == 0
        return FALSE

    return TRUE
}

function CanEngage()
{
    if !${IsSafeToOperate}
        return FALSE

    if !${Me.InSpace}
        return FALSE

    if ${Me.TargetedByCount} > 0 && ${MyShip.ToEntity.GroupID} != GROUPID_BATTLESHIPS
        return FALSE  ; Being targeted and not in battleship

    if ${GetNearbyNPCCount} == 0
        return FALSE

    return TRUE
}

; Usage
function BotPulse()
{
    if !${IsSafeToOperate}
    {
        echo "Not safe, idling"
        return
    }

    if ${CanMine}
    {
        call ProcessMining
    }
    elseif ${CanEngage}
    {
        call ProcessCombat
    }
    else
    {
        call ProcessIdle
    }
}
```

---

## Decision Trees

Decision trees help visualize complex decision-making logic.

### Example: Mining Bot Decision Tree

```
                        ┌──────────────┐
                        │ Bot Pulse    │
                        └──────┬───────┘
                               │
                ┌──────────────┴──────────────┐
                │ Hostiles detected?          │
                └──────┬──────────────┬───────┘
                  YES  │              │ NO
                       ▼              ▼
                ┌──────────┐   ┌─────────────┐
                │ FLEE     │   │ Low shields?│
                └──────────┘   └──────┬──────┘
                                 YES  │  NO
                                      ▼      ▼
                               ┌──────────┐ ┌──────────────┐
                               │ DOCK     │ │ Cargo full?  │
                               └──────────┘ └──────┬───────┘
                                              YES │ NO
                                                  ▼  ▼
                                           ┌──────────┐ ┌────────────┐
                                           │ HAUL     │ │ In space?  │
                                           └──────────┘ └──────┬─────┘
                                                          YES │ NO
                                                              ▼  ▼
                                                       ┌─────────┐ ┌────────┐
                                                       │ MINE    │ │ UNDOCK │
                                                       └─────────┘ └────────┘
```

### Implementation

```lavish
function MiningBotDecisionTree()
{
    ; Level 1: Safety check
    if ${CheckForHostiles}
    {
        echo "Decision: FLEE (hostiles detected)"
        return "FLEE"
    }

    ; Level 2: Ship integrity
    if ${MyShip.ShieldPct} < 30
    {
        echo "Decision: DOCK (low shields)"
        return "DOCK"
    }

    ; Level 3: Cargo status
    if ${MyShip.CargoFreeSpace} < 50
    {
        echo "Decision: HAUL (cargo full)"
        return "HAUL"
    }

    ; Level 4: Location
    if !${Me.InSpace}
    {
        echo "Decision: UNDOCK (in station)"
        return "UNDOCK"
    }

    ; Level 5: Normal operation
    echo "Decision: MINE (all checks passed)"
    return "MINE"
}
```

### Example: Combat Bot Decision Tree

```
                          ┌─────────────┐
                          │ Bot Pulse   │
                          └──────┬──────┘
                                 │
                    ┌────────────┴────────────┐
                    │ Hostiles on grid?       │
                    └────────┬────────────┬───┘
                        NO   │            │ YES
                             ▼            ▼
                      ┌──────────┐  ┌─────────────────┐
                      │ Wrecks?  │  │ Low HP or outnumbered? │
                      └────┬─────┘  └────────┬────────────┬───┘
                       YES │ NO        YES   │            │ NO
                           ▼  ▼              ▼            ▼
                     ┌─────────┐ ┌────┐ ┌──────┐  ┌────────────┐
                     │ LOOT    │ │IDLE│ │RETREAT│  │ Priority?  │
                     └─────────┘ └────┘ └──────┘  └─────┬──────┘
                                                     YES │ NO
                                                         ▼  ▼
                                                  ┌──────────┐ ┌────────┐
                                                  │ KILL PRI │ │ KILL   │
                                                  └──────────┘ └────────┘
```

### Implementation

```lavish
function CombatBotDecisionTree()
{
    variable int npcCount = ${GetNearbyNPCCount}

    ; No NPCs - check for loot or idle
    if ${npcCount} == 0
    {
        if ${GetNearbyWreckCount} > 0
        {
            echo "Decision: LOOT (no enemies, wrecks present)"
            return "LOOT"
        }

        echo "Decision: IDLE (no enemies, no wrecks)"
        return "IDLE"
    }

    ; NPCs present - assess threat
    variable int hpPercent = ${MyShip.ShieldPct}
    if ${MyShip.ArmorPct} < ${hpPercent}
    {
        hpPercent:Set[${MyShip.ArmorPct}]
    }

    ; Check if outnumbered or low HP
    if ${hpPercent} < 40 || ${npcCount} > 5
    {
        echo "Decision: RETREAT (${hpPercent}% HP, ${npcCount} enemies)"
        return "RETREAT"
    }

    ; Check for priority targets
    if ${GetPriorityTargetCount} > 0
    {
        echo "Decision: KILL_PRIORITY (${GetPriorityTargetCount} priority targets)"
        return "KILL_PRIORITY"
    }

    echo "Decision: KILL (${npcCount} enemies, no priorities)"
    return "KILL"
}
```

---

## Weighted Scoring Systems

For complex decisions with multiple factors, weighted scoring provides nuanced selection.

### Example: Best Asteroid Selection

```lavish
; ===== WEIGHTED ASTEROID SCORING =====

function GetBestAsteroid()
{
    variable index:entity Asteroids
    variable iterator Asteroid

    EVE:QueryEntities[Asteroids, "CategoryID = CATEGORYID_ASTEROID"]
    Asteroids:GetIterator[Asteroid]

    if !${Asteroid:First(exists)}
        return 0

    variable int64 BestAsteroid = 0
    variable float BestScore = 0

    do
    {
        variable float score = 0

        ; Distance factor (closer = better)
        ; Score: 0-100 based on distance (0m = 100, 50km = 0)
        variable float distanceScore = ${Math.Calc[100 - (${Asteroid.Value.Distance} / 500)]}
        if ${distanceScore} < 0
        {
            distanceScore:Set[0]
        }
        score:Inc[${distanceScore}]

        ; Ore type factor
        variable string oreName = "${Asteroid.Value.Name}"

        if ${oreName.Find["Veldspar"]}
        {
            score:Inc[10]  ; Low value
        }
        elseif ${oreName.Find["Scordite"]}
        {
            score:Inc[20]
        }
        elseif ${oreName.Find["Pyroxeres"]}
        {
            score:Inc[30]
        }
        elseif ${oreName.Find["Plagioclase"]}
        {
            score:Inc[40]
        }
        elseif ${oreName.Find["Kernite"]}
        {
            score:Inc[80]  ; High value
        }
        elseif ${oreName.Find["Jaspet"]}
        {
            score:Inc[100]  ; Highest value
        }

        ; Quantity factor (larger = better)
        variable int quantity = ${Asteroid.Value.Quantity}
        variable float quantityScore = ${Math.Calc[${quantity} / 1000]}
        if ${quantityScore} > 50
        {
            quantityScore:Set[50]  ; Cap at 50
        }
        score:Inc[${quantityScore}]

        ; Already locked penalty
        if ${Asteroid.Value.IsLockedTarget}
        {
            score:Dec[500]  ; Heavy penalty
        }

        ; Update best
        if ${score} > ${BestScore}
        {
            BestScore:Set[${score}]
            BestAsteroid:Set[${Asteroid.Value.ID}]
        }
    }
    while ${Asteroid:Next(exists)}

    if ${BestAsteroid} > 0
    {
        echo "Best asteroid: ${Entity[${BestAsteroid}].Name} (score: ${BestScore.Precision[2]})"
    }

    return ${BestAsteroid}
}
```

### Example: Yamfa Master Target Selection

**Real Yamfa Pattern** (Yamfa.iss, lines 210-310):

```lavish
function MasterPulse()
{
    variable int CurrentTime = ${Math.Calc[${LavishScript.RunningTime} / 100]}
    variable index:entity Targets
    variable iterator Target
    variable index:int64 CurrentTargets

    ; Get all currently locked/locking targets
    EVE:QueryEntities[Targets]
    Targets:GetIterator[Target]

    if ${Target:First(exists)}
        do
        {
            if ${Target.Value.IsLockedTarget} || ${Target.Value.BeingTargeted}
            {
                CurrentTargets:Insert[${Target.Value.ID}]
                ; Update last seen time for this target
                MasterTargetTimers:Set[${Target.Value.ID}, ${CurrentTime}]
            }
        }
        while ${Target:Next(exists)}

    ; Apply hysteresis - keep targets for MASTER_HOLD_TIME after they disappear
    variable iterator ExistingTarget
    MasterTargetSet:GetIterator[ExistingTarget]

    if ${ExistingTarget:First(exists)}
        do
        {
            variable int64 TargetID = ${MasterTargetSet.Get[${ExistingTarget.Key}]}
            variable int LastSeen = ${MasterTargetTimers.Get[${TargetID}]}

            ; If target still exists in current set, keep it
            variable bool StillExists = FALSE
            variable int i
            for (i:Set[1] ; ${i} <= ${CurrentTargets.Used} ; i:Inc)
            {
                if ${CurrentTargets.Get[${i}]} == ${TargetID}
                {
                    StillExists:Set[TRUE]
                    break
                }
            }

            ; If not in current set but within hold time AND still locked, keep it
            if !${StillExists}
            {
                if ${Math.Calc[${CurrentTime} - ${LastSeen}]} < ${MASTER_HOLD_TIME} && \
                   ${Entity[${TargetID}](exists)} && \
                   (${Entity[${TargetID}].IsLockedTarget} || ${Entity[${TargetID}].BeingTargeted})
                {
                    CurrentTargets:Insert[${TargetID}]
                }
            }
        }
        while ${ExistingTarget:Next(exists)}

    ; Update master target set
    MasterTargetSet:Clear
    variable int j
    for (j:Set[1] ; ${j} <= ${CurrentTargets.Used} ; j:Inc)
    {
        MasterTargetSet:Insert[${CurrentTargets.Get[${j}]}]
    }

    ; Get active target
    MasterActiveTarget:Set[0]
    if ${Me.ActiveTarget(exists)}
        MasterActiveTarget:Set[${Me.ActiveTarget.ID}]

    ; Relay to fleet...
}
```

**Key Pattern**: Yamfa uses **hysteresis with timers** to prevent rapid target changes. Targets are kept for 700ms after unlocking to allow slaves to catch up.

---

## Real Examples from Bots

### EVEBot: Priority Target System

**File**: `obj_Targets.iss`

**Pattern**: Explicit priority lists with name matching

```lavish
; Priority targets will be targeted (and killed)
; before other targets, they often do special things
; which we cant use (scramble / web / damp / etc)

PriorityTargets:Insert["Dire Pithi Destructor"]
PriorityTargets:Insert["Dire Pithi Wrecker"]
PriorityTargets:Insert["Dire Pithi Plunderer"]
PriorityTargets:Insert["Factory Defense Battery"]  /* web/scram */
PriorityTargets:Insert["Dire Pithi Arrogator"]    /* web/scram */
PriorityTargets:Insert["Dire Pithi Despoiler"]    /* Jamming */

; Special targets will trigger an alert
; This should include haulers / faction / officers

SpecialTargets:Insert["Estamel Tharchon"]  ; Guristas officer
SpecialTargets:Insert["Kaikka Peunato"]
SpecialTargets:Insert["Thon Eney"]
SpecialTargets:Insert["Vepas Minimala"]

SpecialTargets:Insert["Dread Guristas"]   ; Faction spawns
SpecialTargets:Insert["Shadow Serpentis"]
SpecialTargets:Insert["True Sansha"]
SpecialTargets:Insert["Dark Blood"]

SpecialTargets:Insert["Courier"]  ; Haulers (low threat, valuable loot)
SpecialTargets:Insert["Ferrier"]
SpecialTargets:Insert["Gatherer"]
SpecialTargets:Insert["Harvester"]
```

**Usage Pattern**:

```lavish
function GetNextCombatTarget()
{
    variable index:entity Entities
    variable iterator Entity

    EVE:QueryEntities[Entities, "IsNPC && !IsMoribund"]
    Entities:GetIterator[Entity]

    ; Pass 1: Check for priority targets
    if ${Entity:First(exists)}
        do
        {
            if ${IsPriorityTarget["${Entity.Value.Name}"]}
            {
                return ${Entity.Value.ID}
            }
        }
        while ${Entity:Next(exists)}

    ; Pass 2: Check for special targets (alerts)
    Entity:First
    do
    {
        if ${IsSpecialTarget["${Entity.Value.Name}"]}
        {
            call AlertSpecialTarget "${Entity.Value.Name}"
            return ${Entity.Value.ID}
        }
    }
    while ${Entity:Next(exists)}

    ; Pass 3: Take closest normal target
    Entity:First
    return ${Entity.Value.ID}
}
```

### Tehbot: Query String Exclusion System

**File**: `obj_TargetList.iss`

**Pattern**: Build complex exclusion query string, then filter results

```lavish
method AddAllNPCs()
{
    variable string QueryString="CategoryID = CATEGORYID_ENTITY && IsNPC && !IsMoribund && !("

    ; Exclude specific groups
    QueryString:Concat["GroupID = GROUP_CONCORDDRONE ||"]
    QueryString:Concat["GroupID = GROUP_CONVOYDRONE ||"]
    QueryString:Concat["GroupID = GROUP_CONVOY ||"]
    QueryString:Concat["GroupID = GROUP_LARGECOLLIDABLEOBJECT ||"]
    QueryString:Concat["GroupID = GROUP_SPAWNCONTAINER ||"]
    QueryString:Concat["GroupID = GROUP_DEADSPACEOVERSEERSSTRUCTURE"]
    QueryString:Concat[")"]

    This:AddQueryString["${QueryString.Escape}"]
}

method AddTargetExceptionByID(int64 ID)
{
    ExcludeTargetID:Add[${ID}]

    ; Remove from existing lists
    variable iterator RemoveIterator
    TargetList:GetIterator[RemoveIterator]
    if ${RemoveIterator:First(exists)}
    {
        do
        {
            if ${RemoveIterator.Value.ID.Equal[${ID}]}
            {
                TargetList:Remove[${RemoveIterator.Key}]
            }
        }
        while ${RemoveIterator:Next(exists)}
    }

    ; Unlock if locked
    if ${Entity[${ID}].IsLockedTarget}
    {
        Entity[${ID}]:UnlockTarget
    }
}
```

---

## Mining Decision Logic

### Decision Flow

```
1. Is safe to mine? (No hostiles, sufficient HP)
   NO → Flee/Dock
   YES ↓

2. Have cargo space?
   NO → Return to station
   YES ↓

3. Have locked asteroid?
   NO → Find and lock best asteroid
   YES ↓

4. Is asteroid depleted?
   YES → Unlock, go to step 3
   NO ↓

5. Are miners active?
   NO → Activate miners
   YES → Continue mining
```

### Implementation

```lavish
function ProcessMining()
{
    ; Safety check
    if !${IsSafeToMine}
    {
        call EmergencyDock
        return
    }

    ; Cargo check
    if ${MyShip.CargoFreeSpace} < 100
    {
        call ReturnToStation
        return
    }

    ; Asteroid lock check
    if ${Me.TargetCount} == 0
    {
        variable int64 bestAsteroid = ${GetBestAsteroid}
        if ${bestAsteroid} > 0
        {
            Entity[${bestAsteroid}]:LockTarget
        }
        return
    }

    ; Get current locked asteroid
    variable int64 currentAsteroid = ${Me.GetTargets.Get[1].ID}

    ; Check if depleted
    if !${Entity[${currentAsteroid}](exists)} || ${Entity[${currentAsteroid}].Distance} > 20000
    {
        echo "Asteroid depleted or out of range, unlocking"
        Entity[${currentAsteroid}]:UnlockTarget
        return
    }

    ; Activate miners
    call ActivateMinersOn ${currentAsteroid}
}

function ActivateMinersOn(int64 asteroidID)
{
    variable int i
    for (i:Set[1]; ${i} <= ${MyShip.ModuleCount}; i:Inc)
    {
        variable item module = ${MyShip.Module[${i}]}

        if ${module.ToItem.Group.Equal["Mining Laser"]} || \
           ${module.ToItem.Group.Equal["Strip Miner"]}
        {
            if !${module.IsActive} && ${module.IsOnline}
            {
                echo "Activating ${module.ToItem.Name}"
                module:Activate[${asteroidID}]
                wait 10
            }
        }
    }
}
```

---

## Combat Decision Logic

### Decision Flow

```
1. Is safe to engage? (No overwhelming force, sufficient HP)
   NO → Retreat
   YES ↓

2. Are priority targets present?
   YES → Lock and kill priority targets first
   NO ↓

3. Are targets within range?
   NO → Approach closer target
   YES ↓

4. Lock targets (up to max)

5. Activate weapons on best target

6. All targets dead?
   YES → Look for loot
   NO → Return to step 2
```

### Implementation

```lavish
function ProcessCombat()
{
    ; Safety check
    if !${IsSafeToCombat}
    {
        call EmergencyRetreat
        return
    }

    ; Get priority targets
    variable index:int64 priorityTargets
    call GetPriorityTargets priorityTargets

    if ${priorityTargets.Used} > 0
    {
        echo "Engaging ${priorityTargets.Used} priority targets"
        call EngageTargets priorityTargets
        return
    }

    ; Get normal targets
    variable index:int64 normalTargets
    call GetNormalTargets normalTargets

    if ${normalTargets.Used} > 0
    {
        echo "Engaging ${normalTargets.Used} normal targets"
        call EngageTargets normalTargets
        return
    }

    ; No targets, check for loot
    if ${GetNearbyWreckCount} > 0
    {
        call ProcessLooting
    }
}

function GetPriorityTargets(index:int64 outTargets)
{
    outTargets:Clear

    variable index:entity Entities
    variable iterator Entity

    EVE:QueryEntities[Entities, "IsNPC && !IsMoribund && Distance < ${MyShip.MaxTargetRange}"]
    Entities:GetIterator[Entity]

    if ${Entity:First(exists)}
        do
        {
            if ${IsPriorityTarget["${Entity.Value.Name}"]}
            {
                outTargets:Insert[${Entity.Value.ID}]
            }
        }
        while ${Entity:Next(exists)}
}

function EngageTargets(index:int64 targetIDs)
{
    ; Lock targets
    variable int i
    for (i:Set[1]; ${i} <= ${targetIDs.Used} && ${Me.TargetCount} < ${Me.MaxLockedTargets}; i:Inc)
    {
        variable int64 targetID = ${targetIDs.Get[${i}]}

        if !${Entity[${targetID}].IsLockedTarget} && !${Entity[${targetID}].BeingTargeted}
        {
            Entity[${targetID}]:LockTarget
            wait 5
        }
    }

    ; Activate weapons on first locked target
    variable int64 activeTarget = 0
    for (i:Set[1]; ${i} <= ${targetIDs.Used}; i:Inc)
    {
        if ${Entity[${targetIDs.Get[${i}]}].IsLockedTarget}
        {
            activeTarget:Set[${targetIDs.Get[${i}]}]
            break
        }
    }

    if ${activeTarget} > 0
    {
        call ActivateWeaponsOn ${activeTarget}
    }
}
```

---

## Movement Decision Logic

### Decision: Approach, Orbit, or Keep Range?

```lavish
function GetMovementCommand(int64 targetID)
{
    if ${targetID} == 0 || !${Entity[${targetID}](exists)}
        return "NONE"

    variable entity target = ${Entity[${targetID}]}
    variable float distance = ${target.Distance}
    variable float optimalRange = ${GetWeaponOptimalRange}

    ; Too far - approach
    if ${distance} > ${Math.Calc[${optimalRange} * 1.5]}
    {
        echo "Movement: APPROACH (${distance}m > ${optimalRange}m optimal)"
        return "APPROACH"
    }

    ; Too close - keep range
    if ${distance} < ${Math.Calc[${optimalRange} * 0.5]}
    {
        echo "Movement: KEEPRANGE (${distance}m < ${optimalRange}m optimal)"
        return "KEEPRANGE"
    }

    ; In range - orbit
    echo "Movement: ORBIT (${distance}m ≈ ${optimalRange}m optimal)"
    return "ORBIT"
}

function ExecuteMovementCommand(string command, int64 targetID)
{
    if !${Entity[${targetID}](exists)}
        return

    switch ${command}
    {
        case APPROACH
            Entity[${targetID}]:Approach
            break
        case ORBIT
            Entity[${targetID}]:Orbit[${GetWeaponOptimalRange}]
            break
        case KEEPRANGE
            Entity[${targetID}]:KeepAtRange[${GetWeaponOptimalRange}]
            break
    }
}
```

---

## Common Patterns

### Pattern 1: Fallback Chain

```lavish
; Try preferred option, fall back to alternatives

function GetTarget()
{
    variable int64 target

    ; Try: Priority targets
    target:Set[${GetPriorityTarget}]
    if ${target} > 0
        return ${target}

    ; Try: Active target (don't switch unnecessarily)
    if ${Me.ActiveTarget(exists)}
        return ${Me.ActiveTarget.ID}

    ; Try: Already locked target
    if ${Me.TargetCount} > 0
        return ${Me.GetTargets.Get[1].ID}

    ; Try: Closest NPC
    target:Set[${GetClosestNPC}]
    if ${target} > 0
        return ${target}

    ; No target found
    return 0
}
```

### Pattern 2: Time-Based Decisions

```lavish
; Make different decisions based on time elapsed

variable int MiningStartTime = 0
variable int MIN_MINING_DURATION = 120000  ; 2 minutes

function ShouldReturnToStation()
{
    ; Don't return if mining less than 2 minutes
    if ${MiningStartTime} > 0
    {
        variable int miningDuration = ${Math.Calc[${LavishScript.RunningTime} - ${MiningStartTime}]}

        if ${miningDuration} < ${MIN_MINING_DURATION}
        {
            echo "Mining for ${miningDuration}ms, minimum ${MIN_MINING_DURATION}ms"
            return FALSE
        }
    }

    ; Cargo full
    if ${MyShip.CargoFreeSpace} < 50
    {
        return TRUE
    }

    return FALSE
}
```

### Pattern 3: Threshold with Hysteresis

```lavish
; Different thresholds for entering/exiting state

variable string ShieldState = "NORMAL"

function CheckShieldState()
{
    ; Enter CRITICAL when shields drop below 30%
    if ${ShieldState.Equal["NORMAL"]} && ${MyShip.ShieldPct} < 30
    {
        echo "Shield state: NORMAL -> CRITICAL"
        ShieldState:Set["CRITICAL"]
        call DockForRepairs
        return
    }

    ; Exit CRITICAL when shields recover above 80%
    if ${ShieldState.Equal["CRITICAL"]} && ${MyShip.ShieldPct} > 80
    {
        echo "Shield state: CRITICAL -> NORMAL"
        ShieldState:Set["NORMAL"]
        return
    }
}
```

---

## Performance Considerations

### 1. Cache Expensive Queries

```lavish
; BAD: Query every pulse
function BotPulse()
{
    variable int npcCount = ${GetNPCCount}  ; Queries entities every time
    if ${npcCount} > 0
    {
        call ProcessCombat
    }
}

; GOOD: Cache and refresh periodically
variable int CachedNPCCount = 0
variable int LastNPCQuery = 0

function BotPulse()
{
    ; Refresh every 1 second
    if ${Math.Calc[${LavishScript.RunningTime} - ${LastNPCQuery}]} > 1000
    {
        CachedNPCCount:Set[${GetNPCCount}]
        LastNPCQuery:Set[${LavishScript.RunningTime}]
    }

    if ${CachedNPCCount} > 0
    {
        call ProcessCombat
    }
}
```

### 2. Short-Circuit Evaluation

```lavish
; BAD: All conditions evaluated
if ${ExpensiveCheck1} && ${ExpensiveCheck2} && ${ExpensiveCheck3}
{
    ; ...
}

; GOOD: Cheap checks first, short-circuit on failure
if ${Me.InSpace} && ${ISXEVE.IsReady} && ${ExpensiveCheck1} && ${ExpensiveCheck2}
{
    ; Me.InSpace and ISXEVE.IsReady are cheap, checked first
    ; If they fail, expensive checks never run
}
```

### 3. Limit Search Scope

```lavish
; BAD: Query all entities
EVE:QueryEntities[Entities, "IsNPC"]

; GOOD: Query only nearby entities
EVE:QueryEntities[Entities, "IsNPC && Distance < ${MyShip.MaxTargetRange}"]

; BETTER: Query specific category/group
EVE:QueryEntities[Entities, "CategoryID = CATEGORYID_ENTITY && IsNPC && Distance < ${MyShip.MaxTargetRange}"]
```

---

## Summary

### Key Takeaways

1. **Decision-making is layered**:
   - Safety checks FIRST (hostiles, damage)
   - Critical operations SECOND (cargo full, repairs needed)
   - Normal operations THIRD (combat, mining)
   - Idle LAST

2. **Target selection patterns**:
   - Closest (simple, fast)
   - Priority lists (explicit threat ranking)
   - Weighted scoring (nuanced, multi-factor)
   - Query string building (flexible, filterable)

3. **Priority systems**:
   - Layered priority (safety > critical > normal > idle)
   - Numeric priority (explicit scoring)
   - Priority queues (separate high/normal/low)

4. **Condition checking**:
   - Guard clauses (early return on invalid state)
   - Compound conditions (AND/OR logic)
   - Condition functions (reusable checks)

5. **Performance**:
   - Cache expensive queries
   - Short-circuit evaluation (cheap checks first)
   - Limit search scope (distance filters)

### Next Files

- **File 19**: Error Handling and Recovery (What to do when things go wrong)
- **File 20**: Performance and Timing (Optimization, profiling, efficiency)

---

**File Complete**: Decision-making and logic patterns fully documented with real examples from EVEBot, Tehbot, and Yamfa.
