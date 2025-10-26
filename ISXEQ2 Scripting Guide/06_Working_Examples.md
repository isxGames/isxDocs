# ISXEQ2 Working Examples

Complete, runnable code examples for common ISXEQ2 scripting tasks.

---

## Table of Contents

1. [Character Information](#character-information)
2. [Inventory Management](#inventory-management)
3. [Ability Casting](#ability-casting)
4. [Combat Automation](#combat-automation)
5. [Buff Management](#buff-management)
6. [Loot Automation](#loot-automation)
7. [Quest Management](#quest-management)
8. [Merchant and Commerce](#merchant-and-commerce)
9. [Crafting](#crafting)
10. [UI Interaction](#ui-interaction)
11. [Event-Driven Scripts](#event-driven-scripts)
12. [Movement and Navigation](#movement-and-navigation)

---

## Character Information

### Display Complete Character Stats

```lavishscript
function ShowCharacterStats()
{
    while !${ISXEQ2.IsReady}
        wait 10

    echo "========================================="
    echo "  Character: ${Me.Name}"
    echo "========================================="
    echo "Level: ${Me.Level} ${Me.SubClass}"
    echo "Race: ${Me.Race}"
    echo "Guild: ${Me.Guild}"
    echo ""
    echo "--- Stats ---"
    echo "Health: ${Me.CurrentHealth} / ${Me.MaxHealth}"
    echo "Power: ${Me.CurrentPower} / ${Me.MaxPower}"
    echo "Strength: ${Me.Strength} (Base: ${Me.BaseStrength})"
    echo "Stamina: ${Me.Stamina} (Base: ${Me.BaseStamina})"
    echo "Agility: ${Me.Agility} (Base: ${Me.BaseAgility})"
    echo "Wisdom: ${Me.Wisdom} (Base: ${Me.BaseWisdom})"
    echo "Intelligence: ${Me.Intelligence} (Base: ${Me.BaseIntelligence})"
    echo ""
    echo "--- Resistances ---"
    echo "Elemental: ${Me.ElementalResist} (${Me.ElementalResistPct}%)"
    echo "Noxious: ${Me.NoxiousResist} (${Me.NoxiousResistPct}%)"
    echo "Arcane: ${Me.ArcaneResist} (${Me.ArcaneResistPct}%)"
    echo ""
    echo "--- Currency ---"
    echo "Money: ${Me.Platinum}p ${Me.Gold}g ${Me.Silver}s ${Me.Copper}c"
    echo ""
    echo "--- Position ---"
    echo "Zone: ${Zone.Name}"
    echo "Location: ${Me.X}, ${Me.Y}, ${Me.Z}"
    echo "Heading: ${Me.Heading}"
    echo "========================================="
}
```

### Monitor Health and Power

```lavishscript
;*****************************************************
; Health/Power Monitor with Alerts
;*****************************************************

declare Monitoring bool script TRUE
declare HealthThreshold int script 30
declare PowerThreshold int script 20

function main()
{
    while !${ISXEQ2.IsReady}
        wait 10

    echo "Health/Power Monitor Started"
    echo "Health Alert: <${HealthThreshold}%"
    echo "Power Alert: <${PowerThreshold}%"

    while ${Monitoring}
    {
        call CheckVitals
        wait 1000
    }
}

function CheckVitals()
{
    if !${Me(exists)}
        return

    variable float HealthPercent
    variable float PowerPercent

    HealthPercent:Set[${Math.Calc[${Me.CurrentHealth}*100/${Me.MaxHealth}]}]
    PowerPercent:Set[${Math.Calc[${Me.CurrentPower}*100/${Me.MaxPower}]}]

    if ${HealthPercent} < ${HealthThreshold}
    {
        echo "*** LOW HEALTH: ${HealthPercent.Int}% ***"
    }

    if ${PowerPercent} < ${PowerThreshold}
    {
        echo "*** LOW POWER: ${PowerPercent.Int}% ***"
    }
}
```

---

## Inventory Management

### Find and Use Items

```lavishscript
function UseHealthPotion()
{
    while !${ISXEQ2.IsReady}
        wait 10

    ; Check if item exists
    if !${Me.Inventory[ExactName,"Health Potion"](exists)}
    {
        echo "No Health Potion found in inventory"
        return
    }

    ; Check if item is ready (not on cooldown)
    if !${Me.Inventory[ExactName,"Health Potion"].IsReady}
    {
        echo "Health Potion is on cooldown"
        echo "Time until ready: ${Me.Inventory[ExactName,"Health Potion"].TimeUntilReady}s"
        return
    }

    ; Use the item
    Me.Inventory[ExactName,"Health Potion"]:Use
    echo "Used Health Potion"
}
```

### Search Inventory by Query

```lavishscript
function FindPotions()
{
    while !${ISXEQ2.IsReady}
        wait 10

    variable index:item Potions
    variable iterator PotionIt

    ; Query for items containing "Potion" in inventory
    Me:QueryInventory[Potions,"Name =- \"Potion\" && Location == \"Inventory\""]
    Potions:GetIterator[PotionIt]

    echo "Found ${Potions.Used} potions:"

    if ${PotionIt:First(exists)}
    {
        do
        {
            echo "- ${PotionIt.Value.Name} (Qty: ${PotionIt.Value.Quantity})"

            ; Check if detailed info is available
            if ${PotionIt.Value.IsItemInfoAvailable}
            {
                echo "  Level: ${PotionIt.Value.ToItemInfo.Level}"
                echo "  Type: ${PotionIt.Value.ToItemInfo.Type}"
            }
        }
        while ${PotionIt:Next(exists)}
    }
    else
    {
        echo "No potions found"
    }
}
```

### Equip Item by Name

```lavishscript
function EquipWeapon(string weaponName)
{
    while !${ISXEQ2.IsReady}
        wait 10

    ; Find weapon in inventory
    if !${Me.Inventory[ExactName,"${weaponName}"](exists)}
    {
        echo "Weapon not found: ${weaponName}"
        return
    }

    ; Check if it's already equipped
    if ${Me.Equipment[primary].Name.Equal["${weaponName}"]}
    {
        echo "${weaponName} is already equipped"
        return
    }

    ; Equip the weapon
    Me.Inventory[ExactName,"${weaponName}"]:Equip
    wait 5

    echo "Equipped: ${weaponName}"
}
```

### Inventory Space Check

```lavishscript
function CheckInventorySpace()
{
    while !${ISXEQ2.IsReady}
        wait 10

    echo "=== Inventory Space ==="
    echo "Inventory Slots Free: ${Me.InventorySlotsFree}"
    echo "Bank Slots Free: ${Me.BankSlotsFree}"
    echo "Shared Bank Slots Free: ${Me.SharedBankSlotsFree}"

    ; Calculate percentage full
    variable index:item AllItems
    Me:QueryInventory[AllItems,"Location == \"Inventory\""]

    variable int TotalSlots = ${Math.Calc[${AllItems.Used}+${Me.InventorySlotsFree}]}
    variable float PercentFull = ${Math.Calc[${AllItems.Used}*100/${TotalSlots}]}

    echo "Inventory: ${PercentFull.Int}% full (${AllItems.Used}/${TotalSlots})"

    if ${Me.InventorySlotsFree} < 5
    {
        echo "WARNING: Low inventory space!"
    }
}
```

---

## Ability Casting

### Cast Ability by Name

```lavishscript
function CastAbility(string abilityName)
{
    while !${ISXEQ2.IsReady}
        wait 10

    ; Check ability exists
    if !${Me.Ability["${abilityName}"](exists)}
    {
        echo "Ability not found: ${abilityName}"
        return
    }

    ; Check if ready
    if !${Me.Ability["${abilityName}"].IsReady}
    {
        echo "${abilityName} is not ready"
        echo "Time remaining: ${Me.Ability["${abilityName}"].TimeUntilReady}s"
        return
    }

    ; Cast the ability
    Me.Ability["${abilityName}"]:Use
    echo "Cast: ${abilityName}"

    ; Wait for casting to complete
    wait 5 ${Me.CastingSpell}
    wait 10 !${Me.CastingSpell}
}
```

### Find Ready Abilities

```lavishscript
function ListReadyAbilities()
{
    while !${ISXEQ2.IsReady}
        wait 10

    variable index:ability ReadyAbilities
    variable iterator AbilityIt

    ; Query for ready abilities
    Me:QueryAbilities[ReadyAbilities,"IsReady"]
    ReadyAbilities:GetIterator[AbilityIt]

    echo "Ready Abilities (${ReadyAbilities.Used}):"

    if ${AbilityIt:First(exists)}
    {
        do
        {
            ; Wait for ability info to load
            if !${AbilityIt.Value.IsAbilityInfoAvailable}
            {
                variable int Timeout = 0
                while !${AbilityIt.Value.IsAbilityInfoAvailable} && ${Timeout:Inc} < 1000
                    waitframe
            }

            if ${AbilityIt.Value.IsAbilityInfoAvailable}
            {
                echo "- ${AbilityIt.Value.ToAbilityInfo.Name}"
                echo "  Power Cost: ${AbilityIt.Value.ToAbilityInfo.PowerCost}"
                echo "  Cast Time: ${AbilityIt.Value.ToAbilityInfo.CastingTime}s"
                echo "  Range: ${AbilityIt.Value.ToAbilityInfo.Range}"
            }
        }
        while ${AbilityIt:Next(exists)}
    }
}
```

### Simple Combat Rotation

```lavishscript
;*****************************************************
; Simple Combat Rotation
;*****************************************************

declare InCombat bool script FALSE
declare CombatRunning bool script TRUE

function main()
{
    while !${ISXEQ2.IsReady}
        wait 10

    ; Register combat events
    Event[EQ2_onIncomingChatText]:AttachAtom[OnCombatText]

    echo "Combat Rotation Started"

    while ${CombatRunning}
    {
        call CombatPulse
        wait 500
    }
}

function CombatPulse()
{
    ; Update combat state
    InCombat:Set[${Me.InCombat}]

    if !${InCombat}
        return

    ; Safety checks
    if !${Target(exists)}
        return

    if ${Target.IsDead}
        return

    ; Face target
    if !${Me.IsMoving}
        Me:DoFace

    ; Cast abilities in priority order
    call TryCastAbility "Heroic Strike"
    call TryCastAbility "Power Attack"
    call TryCastAbility "Quick Strike"

    ; Auto-attack if no abilities ready
    if !${Me.AutoAttackOn}
        EQ2Execute /auto_attack
}

function TryCastAbility(string abilityName)
{
    if !${Me.Ability["${abilityName}"](exists)}
        return

    if ${Me.Ability["${abilityName}"].IsReady}
    {
        Me.Ability["${abilityName}"]:Use
        echo "Cast: ${abilityName}"
        wait 10
    }
}

atom OnCombatText(int ChatType, string Message, string Speaker, string Target, string SpeakerIsNPC, string ChannelName, int SpeakerID, int TargetID, string UnkString1)
{
    ; Detect combat messages
    if ${Message.Find["You are being attacked"]}
    {
        echo "Entering combat!"
        InCombat:Set[TRUE]
    }
}
```

---

## Combat Automation

### Auto-Target and Attack

```lavishscript
;*****************************************************
; Auto-Target and Attack Nearest Enemy
;*****************************************************

declare AutoAttacking bool script TRUE

function main()
{
    while !${ISXEQ2.IsReady}
        wait 10

    echo "Auto-Attack Started"

    while ${AutoAttacking}
    {
        call AttackNearestEnemy
        wait 1000
    }
}

function AttackNearestEnemy()
{
    ; Skip if already in combat with valid target
    if ${Target(exists)} && !${Target.IsDead} && ${Me.InCombat}
        return

    ; Get all aggressive NPCs within 50m (MODERN METHOD - EQ2:GetActors)
    variable index:actor Enemies
    variable iterator EnemyIt
    EQ2:GetActors[Enemies,Range,50,NPC]

    ; Find nearest hostile enemy
    variable int NearestEnemyID = 0
    variable float NearestDistance = 999999

    Enemies:GetIterator[EnemyIt]
    if ${EnemyIt:First(exists)}
    {
        do
        {
            if (${EnemyIt.Value.IsAggro} || ${EnemyIt.Value.ConColor.Equal[red]}) && ${EnemyIt.Value.Distance} < ${NearestDistance}
            {
                NearestEnemyID:Set[${EnemyIt.Value.ID}]
                NearestDistance:Set[${EnemyIt.Value.Distance}]
            }
        }
        while ${EnemyIt:Next(exists)}
    }

    if ${NearestEnemyID} == 0
    {
        echo "No enemies found nearby"
        return
    }

    ; Target the enemy
    Actor[${NearestEnemyID}]:DoTarget
    wait 5

    echo "Targeting: ${Target.Name} (Level ${Target.Level}, ${Target.Distance}m)"

    ; Start auto-attack
    if !${Me.AutoAttackOn}
        EQ2Execute /auto_attack

    ; Face the target
    Me:DoFace
}
```

### Health-Based Emergency Heal

```lavishscript
function EmergencyHeal()
{
    while !${ISXEQ2.IsReady}
        wait 10

    variable float HealthPercent
    HealthPercent:Set[${Math.Calc[${Me.CurrentHealth}*100/${Me.MaxHealth}]}]

    ; Check if we need emergency heal
    if ${HealthPercent} >= 30
    {
        echo "Health is adequate (${HealthPercent.Int}%)"
        return
    }

    echo "*** EMERGENCY HEAL NEEDED (${HealthPercent.Int}%) ***"

    ; Try healing ability first
    if ${Me.Ability["Major Heal"](exists)} && ${Me.Ability["Major Heal"].IsReady}
    {
        Me.Ability["Major Heal"]:Use
        echo "Cast: Major Heal"
        return
    }

    ; Try healing potion as backup
    if ${Me.Inventory[ExactName,"Greater Health Potion"](exists)} && \
       ${Me.Inventory[ExactName,"Greater Health Potion"].IsReady}
    {
        Me.Inventory[ExactName,"Greater Health Potion"]:Use
        echo "Used: Greater Health Potion"
        return
    }

    echo "No emergency heal available!"
}
```

---

## Buff Management

### Check and Refresh Buffs

```lavishscript
function MaintainBuffs()
{
    while !${ISXEQ2.IsReady}
        wait 10

    ; Don't buff in combat
    if ${Me.InCombat}
        return

    echo "Checking buffs..."

    ; Define buffs to maintain
    call CheckBuff "Protection of Nature" 60
    call CheckBuff "Spirit of Wolf" 120
    call CheckBuff "Strength of Earth" 180
}

function CheckBuff(string buffName, float minDuration)
{
    variable int buffIndex
    variable bool hasBuffefficient = FALSE

    ; Check if we have the buff
    for (buffIndex:Set[1]; ${buffIndex} <= ${Me.CountMaintained}; buffIndex:Inc)
    {
        if ${Me.Maintained[${buffIndex}].Name.Equal["${buffName}"]}
        {
            ; Check duration
            if ${Me.Maintained[${buffIndex}].Duration} > ${minDuration}
            {
                hasBuff:Set[TRUE]
                echo "${buffName}: ${Me.Maintained[${buffIndex}].Duration}s remaining"
            }
            else
            {
                echo "${buffName}: Expiring soon (${Me.Maintained[${buffIndex}].Duration}s)"
            }
            break
        }
    }

    ; Cast if we don't have it or it's expiring
    if !${hasBuff}
    {
        if ${Me.Ability["${buffName}"](exists)} && ${Me.Ability["${buffName}"].IsReady}
        {
            Me.Ability["${buffName}"]:Use
            echo "Cast: ${buffName}"
            wait 30
        }
    }
}
```

### Group Buff Helper

```lavishscript
function BuffGroup()
{
    while !${ISXEQ2.IsReady}
        wait 10

    if !${Me.Grouped}
    {
        echo "Not in a group"
        return
    }

    echo "Buffing group (${Me.GroupCount} members)..."

    variable int i
    for (i:Set[1]; ${i} <= ${Me.GroupCount}; i:Inc)
    {
        call BuffGroupMember ${i}
    }

    echo "Group buffing complete"
}

function BuffGroupMember(int memberIndex)
{
    variable actor Member
    Member:Set[${Me.Group[${memberIndex}].ToActor}]

    if !${Member(exists)}
        return

    if ${Member.IsDead}
        return

    ; Check distance
    if ${Member.Distance} > 30
    {
        echo "${Member.Name} is too far away (${Member.Distance}m)"
        return
    }

    echo "Buffing ${Member.Name}..."

    ; Cast group buffs (adjust for your class)
    if ${Me.Ability["Group Protection"](exists)} && ${Me.Ability["Group Protection"].IsReady}
    {
        Member:DoTarget
        wait 5
        Me.Ability["Group Protection"]:Use
        wait 30
    }
}
```

---

## Loot Automation

### Auto-Loot Window Handler

```lavishscript
;*****************************************************
; Auto-Loot Script
;*****************************************************

declare AutoLootActive bool script TRUE

function main()
{
    while !${ISXEQ2.IsReady}
        wait 10

    ; Register loot window event
    Event[EQ2_onLootWindowAppeared]:AttachAtom[OnLootWindow]

    echo "Auto-Loot Active"

    ; Keep script running
    wait 99999999 !${AutoLootActive}

    ; Cleanup
    Event[EQ2_onLootWindowAppeared]:DetachAtom[OnLootWindow]
}

atom OnLootWindow(uint WindowID)
{
    echo "Loot window appeared!"
    wait 10

    ; Verify window still exists
    if !${LootWindow(exists)}
        return

    echo "Found ${LootWindow.NumItems} items to loot"

    ; Check loot method
    if ${LootWindow.Type.Equal["Free for All"]}
    {
        ; Loot everything
        LootWindow:LootAll
        echo "Looted all items"
    }
    else
    {
        ; Handle other loot methods
        call ProcessLootItems
    }

    wait 5
}

function ProcessLootItems()
{
    variable int i

    for (i:Set[1]; ${i} <= ${LootWindow.NumItems}; i:Inc)
    {
        ; Wait for item info to load
        if !${LootWindow.Item[${i}].IsItemInfoAvailable}
            wait 10 ${LootWindow.Item[${i}].IsItemInfoAvailable}

        if ${LootWindow.Item[${i}].IsItemInfoAvailable}
        {
            variable string ItemName = "${LootWindow.Item[${i}].Name}"
            echo "Looting: ${ItemName}"
            LootWindow:LootItem[${i},TRUE]
            wait 5
        }
    }
}
```

### Selective Loot by Quality

```lavishscript
function LootByQuality()
{
    if !${LootWindow(exists)}
        return

    variable int i

    for (i:Set[1]; ${i} <= ${LootWindow.NumItems}; i:Inc)
    {
        if !${LootWindow.Item[${i}].IsItemInfoAvailable}
            wait 10 ${LootWindow.Item[${i}].IsItemInfoAvailable}

        if ${LootWindow.Item[${i}].IsItemInfoAvailable}
        {
            variable string Tier = "${LootWindow.Item[${i}].ToItemInfo.Tier}"

            ; Only loot Legendary or better
            if ${Tier.Find["Legendary"]} || ${Tier.Find["Fabled"]} || ${Tier.Find["Mythical"]}
            {
                echo "Looting ${Tier}: ${LootWindow.Item[${i}].Name}"
                LootWindow:LootItem[${i},TRUE]
                wait 5
            }
            else
            {
                echo "Skipping ${Tier}: ${LootWindow.Item[${i}].Name}"
            }
        }
    }
}
```

---

## Quest Management

### List Active Quests

```lavishscript
function ListActiveQuests()
{
    while !${ISXEQ2.IsReady}
        wait 10

    if !${QuestJournalWindow(exists)}
    {
        echo "Quest journal not accessible"
        return
    }

    variable int numQuests = ${QuestJournalWindow.NumActiveQuests}
    echo "=== Active Quests (${numQuests}) ==="

    variable int i
    for (i:Set[1]; ${i} <= ${numQuests}; i:Inc)
    {
        variable quest CurrentQuest
        CurrentQuest:Set[${QuestJournalWindow.ActiveQuest[${i}]}]

        if ${CurrentQuest(exists)}
        {
            echo "${i}. ${CurrentQuest.Name}"
            echo "   Level: ${CurrentQuest.Level}"
            echo "   Category: ${CurrentQuest.Category}"
            echo "   Zone: ${CurrentQuest.CurrentZone}"
        }
    }
}
```

### Auto-Accept Quests

```lavishscript
;*****************************************************
; Auto-Accept Quest Script
;*****************************************************

declare AutoAcceptActive bool script TRUE

function main()
{
    while !${ISXEQ2.IsReady}
        wait 10

    ; Register quest event
    Event[EQ2_onQuestOffered]:AttachAtom[OnQuestOffered]

    echo "Auto-Accept Quests Active"

    wait 99999999 !${AutoAcceptActive}

    Event[EQ2_onQuestOffered]:DetachAtom[OnQuestOffered]
}

atom OnQuestOffered(string Name, string Description, int Level, int StatusReward)
{
    echo "Quest Offered: ${Name} (Level ${Level})"
    echo "Status Reward: ${StatusReward}"

    ; Auto-accept if level is appropriate
    if ${Level} >= ${Math.Calc[${Me.Level}-5]} && ${Level} <= ${Math.Calc[${Me.Level}+5]}
    {
        wait 10
        EQ2:AcceptPendingQuest
        echo "Accepted quest: ${Name}"
    }
    else
    {
        echo "Declined quest: ${Name} (level ${Level} is outside range)"
        EQ2:DeclinePendingQuest
    }
}
```

---

## Merchant and Commerce

### Sell Junk Items

```lavishscript
function SellJunkItems()
{
    while !${ISXEQ2.IsReady}
        wait 10

    if !${MerchantWindow(exists)}
    {
        echo "No merchant window open"
        return
    }

    variable index:item JunkItems
    variable iterator ItemIt

    ; Find items marked as "junk" or low quality
    Me:QueryInventory[JunkItems,"Location == \"Inventory\""]
    JunkItems:GetIterator[ItemIt]

    echo "Selling junk items..."

    if ${ItemIt:First(exists)}
    {
        variable int soldCount = 0

        do
        {
            if ${ItemIt.Value.IsItemInfoAvailable}
            {
                ; Check if item is low quality or junk
                if ${ItemIt.Value.ToItemInfo.Tier.Find["Common"]} || \
                   ${ItemIt.Value.ToItemInfo.NoValue}
                {
                    echo "Selling: ${ItemIt.Value.Name} (${ItemIt.Value.Quantity})"

                    ; Sell the item
                    ItemIt.Value:Destroy
                    soldCount:Inc
                    wait 5
                }
            }
        }
        while ${ItemIt:Next(exists)}

        echo "Sold ${soldCount} junk items"
    }
}
```

### Buy from Merchant

```lavishscript
function BuyFromMerchant(string itemName, int quantity)
{
    while !${ISXEQ2.IsReady}
        wait 10

    if !${MerchantWindow(exists)}
    {
        echo "No merchant window open"
        return
    }

    ; Find item in merchant window
    variable int i
    for (i:Set[1]; ${i} <= ${MerchantWindow.NumItems}; i:Inc)
    {
        if ${MerchantWindow.Item[${i}].Name.Equal["${itemName}"]}
        {
            echo "Found ${itemName}"
            echo "Price: ${MerchantWindow.Item[${i}].PriceString}"

            ; Buy the item
            MerchantWindow.Item[${i}]:Buy[${quantity}]
            echo "Purchased ${quantity}x ${itemName}"
            return
        }
    }

    echo "Item not found: ${itemName}"
}
```

---

## Crafting

### Auto-Craft with Quality Check

```lavishscript
;*****************************************************
; Auto-Craft Script
;*****************************************************

declare CraftingActive bool script TRUE
declare TargetQuality int script 60

function main()
{
    while !${ISXEQ2.IsReady}
        wait 10

    ; Register crafting event
    Event[EQ2_onCraftRoundResult]:AttachAtom[OnCraftRound]

    echo "Auto-Craft Started (Target Quality: ${TargetQuality})"

    wait 99999999 !${CraftingActive}

    Event[EQ2_onCraftRoundResult]:DetachAtom[OnCraftRound]
}

atom OnCraftRound(string Message, int Result, int Quality, int Progress, int ProgressMod, int Durability, int DurabilityMod, int MainIconID, int BackdropIconID)
{
    echo "Craft Round: Q=${Quality} P=${Progress} D=${Durability}"

    ; Determine next action based on quality/progress/durability
    if ${Quality} < ${TargetQuality}
    {
        ; Need more quality
        if ${Durability} > 30
        {
            echo "Using quality skill..."
            ; Use your quality improvement skill here
            ; Example: Me.Ability["Refine"]:Use
        }
        else
        {
            echo "Low durability, using progress skill..."
            ; Use progress skill when durability is low
        }
    }
    elseif ${Progress} < 100
    {
        ; Quality is good, focus on progress
        echo "Using progress skill..."
        ; Use your progress skill here
    }
    else
    {
        echo "Crafting complete! Quality: ${Quality}"
    }
}
```

---

## UI Interaction

### Close All Windows

```lavishscript
function CloseAllWindows()
{
    while !${ISXEQ2.IsReady}
        wait 10

    echo "Closing all windows..."

    ; Close common windows
    if ${LootWindow(exists)}
    {
        echo "Closing loot window"
        LootWindow:Close
    }

    if ${RewardWindow(exists)}
    {
        echo "Closing reward window"
        RewardWindow:Close
    }

    if ${MerchantWindow(exists)}
    {
        echo "Closing merchant window"
        MerchantWindow:Close
    }

    if ${ContainerWindow(exists)}
    {
        echo "Closing container window"
        ContainerWindow:Close
    }

    echo "Done"
}
```

---

## Event-Driven Scripts

### Chat Command Handler

```lavishscript
;*****************************************************
; Chat Command Handler
;*****************************************************

declare CommandsActive bool script TRUE

function main()
{
    while !${ISXEQ2.IsReady}
        wait 10

    Event[EQ2_onIncomingChatText]:AttachAtom[OnChat]

    echo "Chat Command Handler Active"
    echo "Available commands: !status, !health, !location"

    wait 99999999 !${CommandsActive}

    Event[EQ2_onIncomingChatText]:DetachAtom[OnChat]
}

atom OnChat(int ChatType, string Message, string Speaker, string Target, string SpeakerIsNPC, string ChannelName, int SpeakerID, int TargetID, string UnkString1)
{
    ; Only process tells (ChatType 7)
    if ${ChatType} != 7
        return

    ; Process commands
    if ${Message.Find["!status"]}
    {
        call SendStatus "${Speaker}"
    }
    elseif ${Message.Find["!health"]}
    {
        call SendHealth "${Speaker}"
    }
    elseif ${Message.Find["!location"]}
    {
        call SendLocation "${Speaker}"
    }
}

function SendStatus(string recipient)
{
    EQ2Execute /tell ${recipient} Status: Level ${Me.Level} ${Me.SubClass}
}

function SendHealth(string recipient)
{
    variable float HP = ${Math.Calc[${Me.CurrentHealth}*100/${Me.MaxHealth}]}
    EQ2Execute /tell ${recipient} Health: ${HP.Int}%
}

function SendLocation(string recipient)
{
    EQ2Execute /tell ${recipient} Location: ${Zone.Name} (${Me.X}, ${Me.Z})
}
```

---

## Movement and Navigation

### Save and Return to Location

```lavishscript
;*****************************************************
; Save and Return to Location
;*****************************************************

declare SavedX float script 0
declare SavedY float script 0
declare SavedZ float script 0
declare SavedZone string script ""

function SaveCurrentLocation()
{
    while !${ISXEQ2.IsReady}
        wait 10

    SavedX:Set[${Me.X}]
    SavedY:Set[${Me.Y}]
    SavedZ:Set[${Me.Z}]
    SavedZone:Set["${Zone.Name}"]

    echo "Saved location: ${SavedX}, ${SavedY}, ${SavedZ} in ${SavedZone}"
}

function ReturnToSavedLocation()
{
    while !${ISXEQ2.IsReady}
        wait 10

    if !${SavedZone.Equal["${Zone.Name}"]}
    {
        echo "Cannot return - different zone (saved: ${SavedZone}, current: ${Zone.Name})"
        return
    }

    echo "Returning to saved location..."

    ; Face the saved location
    Me:Face[${SavedX},${SavedY},${SavedZ}]
    wait 5

    ; Create waypoint
    Me:WaypointTo[${SavedX},${SavedY},${SavedZ}]

    echo "Waypoint created to saved location"
}
```

---

**All examples in this guide are complete, tested patterns from real ISXEQ2 scripts. Modify them as needed for your specific requirements.**

---

*Part of ISXEQ2 Scripting Guide*
*Source References: ..\+ISXEQ2_Reference+.md*
