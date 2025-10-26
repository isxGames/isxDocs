# Utility Script Patterns

**Source:** EQ2BJCommon Scripts Analysis
**Repository:** https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2BJCommon
**Scripts Analyzed:** bjauction, bjlooter, bjmagic, bjshuffle, bjxpbot
**Total Lines Analyzed:** ~2,500 lines

---

## Table of Contents

1. [Overview](#overview)
2. [Custom Timer Objects](#custom-timer-objects)
3. [Script RunningTime Calculations](#script-runningtime-calculations)
4. [Currency Management](#currency-management)
5. [Position Tracking and Return-to-Home](#position-tracking-and-return-to-home)
6. [Character-Specific Configuration](#character-specific-configuration)
7. [Dynamic File Loading](#dynamic-file-loading)
8. [Prioritized Item Lists](#prioritized-item-lists)
9. [Event-Based Text Parsing](#event-based-text-parsing)
10. [EQ2DataSourceContainer Usage](#eq2datasourcecontainer-usage)
11. [Randomized Movement](#randomized-movement)
12. [Item Usage Verification](#item-usage-verification)
13. [ApplyVerb for Object Interaction](#applyverb-for-object-interaction)
14. [ReplyDialog for NPC Conversations](#replydialog-for-npc-conversations)
15. [Audio Notifications](#audio-notifications)
16. [Input Validation Patterns](#input-validation-patterns)
17. [Progressive XP Tracking](#progressive-xp-tracking)
18. [Script Existence Checking](#script-existence-checking)
19. [ExecuteQueued Loop Patterns](#executequeued-loop-patterns)
20. [Complete Working Examples](#complete-working-examples)

---

## Overview

The EQ2BJCommon script collection provides utility scripts for auction management, looting, movement, and XP tracking. These scripts demonstrate practical patterns for building robust, user-friendly tools that handle real-world scenarios.

**Key Characteristics:**
- Heavy use of LavishSettings for persistent configuration
- Extensive input validation and user feedback
- Timer-based operations with precise time tracking
- UI integration with queued commands
- Character-specific settings support
- Error handling and graceful degradation

---

## Custom Timer Objects

Custom timer objects provide more flexible time tracking than simple `wait` commands. They allow you to check remaining time, restart timers, and coordinate multiple time-based operations.

### Pattern

```lavishscript
objectdef TimerObject
{
	variable uint EndTime

	method Set(uint Milliseconds)
	{
		EndTime:Set[${Milliseconds}+${Script.RunningTime}]
	}

	member:uint TimeLeft()
	{
		if ${Script.RunningTime}>=${EndTime}
			return 0
		return ${Math.Calc[${EndTime}-${Script.RunningTime}]}
	}
}
```

### Usage Example

```lavishscript
variable(global) TimerObject NextPotionTimer
variable int RandomDelay

function main()
{
	; Set a random delay between 10-15 minutes (600000-900000 ms)
	RandomDelay:Set[${Math.Rand[300000]:Inc[600000]}]
	NextPotionTimer:Set[${RandomDelay}]

	echo Next potion use in ${Math.Calc[${NextPotionTimer.TimeLeft}/1000]} seconds

	while 1
	{
		if ${NextPotionTimer.TimeLeft} == 0
		{
			echo Timer expired! Using potion...
			call UsePotion

			; Reset timer for next use
			RandomDelay:Set[${Math.Rand[300000]:Inc[600000]}]
			NextPotionTimer:Set[${RandomDelay}]
		}

		; Display countdown
		echo Time until next potion: ${Math.Calc[${NextPotionTimer.TimeLeft}/1000]} seconds
		wait 10
	}
}
```

### Benefits

- **Non-blocking**: Check timer status without waiting
- **Resettable**: Restart timer dynamically
- **Multiple timers**: Track different operations independently
- **Precise**: Based on `${Script.RunningTime}` for accuracy

**Source**: bjmagic.iss:94-109, bjxpbot.iss:1115-1130

---

## Script RunningTime Calculations

Converting `${Script.RunningTime}` (milliseconds) into human-readable time formats is essential for displaying elapsed time, countdowns, and time-per-hour statistics.

### Time Conversion Pattern

```lavishscript
variable int StartTime
variable int DisplaySeconds
variable int DisplayMinutes
variable int DisplayHours

function CalculateTime()
{
	; Convert milliseconds to seconds
	StartTime:Set[${Math.Calc64[${Script.RunningTime}/1000]}]

	; Calculate individual components
	DisplaySeconds:Set[${Math.Calc64[${StartTime}%60]}]
	DisplayMinutes:Set[${Math.Calc64[${StartTime}/60%60]}]
	DisplayHours:Set[${Math.Calc64[${StartTime}/60\\60]}]

	; Display formatted time
	echo Script running for: ${DisplayHours.LeadingZeroes[2]}:${DisplayMinutes.LeadingZeroes[2]}:${DisplaySeconds.LeadingZeroes[2]}
}
```

### Complete Timer Display Example

```lavishscript
variable int StartTime
variable int DisplaySeconds
variable int DisplayMinutes
variable int DisplayHours

function main()
{
	echo Starting timer display...

	while 1
	{
		call UpdateTimerDisplay
		wait 10  ; Update every second
	}
}

function UpdateTimerDisplay()
{
	StartTime:Set[${Math.Calc64[${Script.RunningTime}/1000]}]
	DisplaySeconds:Set[${Math.Calc64[${StartTime}%60]}]
	DisplayMinutes:Set[${Math.Calc64[${StartTime}/60%60]}]
	DisplayHours:Set[${Math.Calc64[${StartTime}/60\\60]}]

	; Update UI element or display in console
	echo Elapsed: ${DisplayHours.LeadingZeroes[2]}:${DisplayMinutes.LeadingZeroes[2]}:${DisplaySeconds.LeadingZeroes[2]}
}
```

### Countdown Timer Example

```lavishscript
variable(global) TimerObject CountdownTimer
variable int TimeRemaining
variable int DisplaySeconds
variable int DisplayMinutes
variable int DisplayHours

function ShowCountdown()
{
	while ${CountdownTimer.TimeLeft} > 0
	{
		; Convert remaining milliseconds to seconds
		TimeRemaining:Set[${Math.Calc64[${CountdownTimer.TimeLeft}/1000]}]

		; Calculate display components
		DisplaySeconds:Set[${Math.Calc64[${TimeRemaining}%60]}]
		DisplayMinutes:Set[${Math.Calc64[${TimeRemaining}/60%60]}]
		DisplayHours:Set[${Math.Calc64[${TimeRemaining}/60\\60]}]

		echo Time remaining: ${DisplayHours.LeadingZeroes[2]}:${DisplayMinutes.LeadingZeroes[2]}:${DisplaySeconds.LeadingZeroes[2]}
		wait 10
	}

	echo Countdown complete!
}
```

### Per-Hour Rate Calculation

```lavishscript
variable float TimeRunningHours
variable float ItemsGained
variable float ItemsPerHour

function CalculateRatePerHour()
{
	; Calculate hours script has been running
	TimeRunningHours:Set[${Math.Calc[(((${Script.RunningTime}/1000)/60/60))].Milli}]

	if ${TimeRunningHours} > 0
	{
		ItemsPerHour:Set[${Math.Calc[${ItemsGained}/${TimeRunningHours}]}]
		echo Gaining ${ItemsPerHour.Centi} items per hour
	}
}
```

**Source**: bjauctiontimer.iss:1-12, bjxpbot.iss:1091-1101, xpcalclevel.iss:231-238

---

## Currency Management

Converting all currency denominations to a single base unit (copper) simplifies tracking and calculations.

### Currency Conversion Pattern

```lavishscript
variable int StartingCoins
variable int CurrentCoins
variable int GainedCoins

variable int DisplayCopper
variable int DisplaySilver
variable int DisplayGold
variable int DisplayPlatinum

function InitializeCoinTracking()
{
	; Convert all currency to copper (1p = 1,000,000c, 1g = 10,000c, 1s = 100c)
	StartingCoins:Set[${Math.Calc[(${Me.Platinum}*1000000)+(${Me.Gold}*10000)+(${Me.Silver}*100)+${Me.Copper}]}]
	echo Starting with ${StartingCoins} total copper
}

function CalculateGainedCoin()
{
	CurrentCoins:Set[${Math.Calc[(${Me.Platinum}*1000000)+(${Me.Gold}*10000)+(${Me.Silver}*100)+${Me.Copper}]}]
	GainedCoins:Set[${Math.Calc[${CurrentCoins}-${StartingCoins}]}]

	; Break down into individual denominations for display
	DisplayCopper:Set[${Math.Calc[${GainedCoins}%100]}]
	DisplaySilver:Set[${Math.Calc[${GainedCoins}/100%100]}]
	DisplayGold:Set[${Math.Calc[${GainedCoins}/10000%100]}]
	DisplayPlatinum:Set[${Math.Calc[${GainedCoins}/10000\\100]}]

	echo Gained: ${DisplayPlatinum.LeadingZeroes[2]}p ${DisplayGold.LeadingZeroes[2]}g ${DisplaySilver.LeadingZeroes[2]}s ${DisplayCopper.LeadingZeroes[2]}c
}
```

### Complete Coin Tracking Example

```lavishscript
variable int StartingCoins
variable int CurrentCoins
variable int GainedCoins
variable int DisplayCopper
variable int DisplaySilver
variable int DisplayGold
variable int DisplayPlatinum

function main()
{
	call InitializeCoinTracking

	echo Looting for 1 minute...
	wait 600  ; 60 seconds

	call DisplayGainedCoin
}

function InitializeCoinTracking()
{
	StartingCoins:Set[${Math.Calc[(${Me.Platinum}*1000000)+(${Me.Gold}*10000)+(${Me.Silver}*100)+${Me.Copper}]}]
	echo Starting coins (in copper): ${StartingCoins}
}

function DisplayGainedCoin()
{
	CurrentCoins:Set[${Math.Calc[(${Me.Platinum}*1000000)+(${Me.Gold}*10000)+(${Me.Silver}*100)+${Me.Copper}]}]
	GainedCoins:Set[${Math.Calc[${CurrentCoins}-${StartingCoins}]}]

	; Convert back to individual denominations
	DisplayCopper:Set[${Math.Calc[${GainedCoins}%100]}]
	DisplaySilver:Set[${Math.Calc[${GainedCoins}/100%100]}]
	DisplayGold:Set[${Math.Calc[${GainedCoins}/10000%100]}]
	DisplayPlatinum:Set[${Math.Calc[${GainedCoins}/10000\\100]}]

	echo ===============================
	echo Total Gained: ${DisplayPlatinum}p ${DisplayGold}g ${DisplaySilver}s ${DisplayCopper}c
	echo ===============================
}
```

### Benefits

- **Simplifies math**: Single number to track
- **Accurate**: No rounding errors from multiple variables
- **Display-friendly**: Easy conversion back to p/g/s/c format

**Source**: bjlooter.iss:9, 222-238

---

## Position Tracking and Return-to-Home

Save character position and heading, then automatically return after completing actions elsewhere.

### Position Saving Pattern

```lavishscript
variable float InitialHeading
variable point3f InitialLocation
variable int MaxDistance=4

function main()
{
	; Save current position
	InitialHeading:Set[${Me.Heading}]
	InitialLocation:Set[${Me.Loc}]

	echo Saved position: ${InitialLocation.X}, ${InitialLocation.Y}, ${InitialLocation.Z}
	echo Saved heading: ${InitialHeading}

	; Do some movement
	call GoDoSomething

	; Return to saved position
	call ReturnToHome
}

function ReturnToHome()
{
	; Check if we're too far from home
	if ${Math.Distance[${Me.Loc},${InitialLocation}]} > ${MaxDistance}
	{
		echo Returning to home position...

		; Move back to saved location
		while ${Math.Distance[${Me.Loc},${InitialLocation}]} > 3
		{
			face ${InitialLocation.X} ${InitialLocation.Z}
			eq2press -hold w
		}
		eq2press -release w

		echo Arrived at home position
	}

	; Restore original heading
	face ${InitialHeading}
}
```

### Complete Auto-Return Example

```lavishscript
variable float InitialHeading
variable point3f InitialLocation
variable int MaxDistance=5

function main()
{
	echo Starting auto-looter with return-to-home...

	; Save starting position
	InitialHeading:Set[${Me.Heading}]
	InitialLocation:Set[${Me.Loc}]

	while 1
	{
		if ${Actor[chest,"Treasure Chest"].Name(exists)} && ${Actor[chest,"Treasure Chest"].Distance} <= 20
		{
			call LootChest
			call ReturnToHome
		}

		wait 10
	}
}

function LootChest()
{
	echo Chest found! Moving to loot...

	face "Treasure Chest"
	wait 5

	; Move to chest
	while ${Actor[chest,"Treasure Chest"].Distance} > 2
	{
		eq2press -hold w
		face "Treasure Chest"
	}
	eq2press -release w

	; Loot it
	EQ2execute "/apply_verb ${Actor[Treasure Chest].ID} Open"
	wait 20

	echo Chest looted
}

function ReturnToHome()
{
	; Check distance from home
	if ${Math.Distance[${Me.Loc},${InitialLocation}]} > ${MaxDistance}
	{
		echo Too far from home (${Math.Distance[${Me.Loc},${InitialLocation}].Centi}m). Returning...

		while ${Math.Distance[${Me.Loc},${InitialLocation}]} > 3
		{
			face ${InitialLocation.X} ${InitialLocation.Z}
			eq2press -hold w
		}
		eq2press -release w

		echo Back at home position
	}

	; Face original direction
	face ${InitialHeading}
}
```

### Benefits

- **Automation**: Return automatically after tasks
- **Safety**: Stay near designated area
- **Precision**: `Math.Distance` for exact range checking
- **Flexible**: Configurable max distance threshold

**Source**: bjshuffle.iss:15-17,28-29,59-67, bjlooter.iss:203-216

---

## Character-Specific Configuration

Allow the same script to maintain different settings for each character on each server.

### Pattern

```lavishscript
variable settingsetref Settings
variable settingsetref _ref
variable filepath ConfigFile="${LavishScript.HomeDirectory}/Scripts/MyScript/Character Config/${EQ2.ServerName}_${Me.Name}_Settings.xml"

function LoadSettings()
{
	LavishSettings:AddSet[MyScript]
	LavishSettings[MyScript]:Clear
	LavishSettings[MyScript]:AddSet[Settings]
	LavishSettings[MyScript]:Import["${LavishScript.HomeDirectory}/Scripts/MyScript/Character Config/${EQ2.ServerName}_${Me.Name}_Settings.xml"]
	_ref:Set[${LavishSettings.FindSet[MyScript]}]

	echo Loaded settings for ${Me.Name} on ${EQ2.ServerName}
}

function SaveSettings()
{
	; Save settings with character-specific filename
	LavishSettings[MyScript]:Export["${LavishScript.HomeDirectory}/Scripts/MyScript/Character Config/${EQ2.ServerName}_${Me.Name}_Settings.xml"]
	echo Saved settings for ${Me.Name} on ${EQ2.ServerName}
}
```

### Complete Example

```lavishscript
variable settingsetref Settings
variable settingsetref _ref
variable filepath ConfigFile="${LavishScript.HomeDirectory}/Scripts/AutoBuff/Character Config/${EQ2.ServerName}_${Me.Name}_BuffSettings.xml"
variable bool AutoBuffEnabled
variable string PreferredBuff

function main()
{
	call LoadSettings

	echo Auto-buff enabled: ${AutoBuffEnabled}
	echo Preferred buff: ${PreferredBuff}

	; Use settings...
	if ${AutoBuffEnabled}
	{
		while 1
		{
			call CheckBuffs
			wait 50
		}
	}
}

function LoadSettings()
{
	LavishSettings:AddSet[AutoBuff]
	LavishSettings[AutoBuff]:Clear
	LavishSettings[AutoBuff]:AddSet[Settings]
	LavishSettings[AutoBuff]:Import["${ConfigFile}"]
	_ref:Set[${LavishSettings.FindSet[AutoBuff]}]

	; Load individual settings
	if ${_ref.FindSetting[AutoBuffEnabled](exists)}
		AutoBuffEnabled:Set[${_ref.FindSetting[AutoBuffEnabled]}]
	else
		AutoBuffEnabled:Set[TRUE]

	if ${_ref.FindSetting[PreferredBuff](exists)}
		PreferredBuff:Set[${_ref.FindSetting[PreferredBuff]}]
	else
		PreferredBuff:Set["Brell's Stalwart Shield"]
}

function SaveSettings()
{
	_ref:AddSetting[AutoBuffEnabled,${AutoBuffEnabled}]
	_ref:AddSetting[PreferredBuff,${PreferredBuff}]

	LavishSettings[AutoBuff]:Export["${ConfigFile}"]
	echo Settings saved for ${Me.Name} on ${EQ2.ServerName}
}

function atexit()
{
	call SaveSettings
}
```

### Benefits

- **Character-specific**: Different settings per character
- **Server-aware**: Handles same character names on different servers
- **Automatic**: No manual file selection needed
- **Organized**: Character Config folder keeps files together

**Source**: bjxpbot.iss:17,25,166

---

## Dynamic File Loading

Enumerate files in a directory to create dynamic preset lists, allowing users to save and load custom configurations.

### Pattern

```lavishscript
variable filelist ConfigFiles
variable int Counter

function LoadAvailablePresets()
{
	variable int Count=0

	; Clear existing list
	UIElement[PresetComboBox@Settings]:ClearItems

	; Get all XML files in presets directory
	ConfigFiles:GetFiles[${LavishScript.HomeDirectory}/Scripts/MyScript/Presets/\*.xml]

	; Add each file to combo box (without .xml extension)
	while ${Count:Inc}<=${ConfigFiles.Files}
	{
		UIElement[PresetComboBox@Settings]:AddItem[${ConfigFiles.File[${Count}].Filename.Left[-4]}]
	}

	; Sort and select first item
	if ${UIElement[PresetComboBox@Settings].Items}
	{
		UIElement[PresetComboBox@Settings]:Sort:SelectItem[1]
	}

	echo Loaded ${ConfigFiles.Files} preset(s)
}
```

### Complete Save/Load Preset Example

```lavishscript
variable filelist PresetFiles
variable string PresetName
variable settingsetref PresetSettings

function SavePreset()
{
	; Get preset name from text entry
	PresetName:Set[${UIElement[PresetNameTextEntry@Settings].Text}]

	if ${PresetName.Length}
	{
		echo Saving preset: ${PresetName}

		LavishSettings:AddSet[Presets]
		LavishSettings[Presets]:AddSet[${PresetName}]

		; Save settings to preset
		LavishSettings[Presets].FindSet[${PresetName}]:AddSetting[Setting1,${MySetting1}]
		LavishSettings[Presets].FindSet[${PresetName}]:AddSetting[Setting2,${MySetting2}]
		LavishSettings[Presets].FindSet[${PresetName}]:AddSetting[Setting3,${MySetting3}]

		; Export to file
		LavishSettings[Presets].FindSet[${PresetName}]:Export[${LavishScript.HomeDirectory}/Scripts/MyScript/Presets/${PresetName}.xml]
		LavishSettings[Presets]:Remove

		echo Preset saved: ${PresetName}

		; Refresh preset list
		call RefreshPresetList
	}
	else
	{
		echo Please enter a preset name
	}
}

function LoadPreset()
{
	variable iterator SettingIter

	; Get selected preset name
	PresetName:Set[${UIElement[PresetComboBox@Settings].SelectedItem.Text}]

	if ${PresetName.Length}
	{
		echo Loading preset: ${PresetName}

		LavishSettings:AddSet[Presets]
		LavishSettings[Presets]:AddSet[${PresetName}]
		LavishSettings[Presets].FindSet[${PresetName}]:Import[${LavishScript.HomeDirectory}/Scripts/MyScript/Presets/${PresetName}.xml]

		; Load settings from preset
		if ${LavishSettings[Presets].FindSet[${PresetName}].FindSetting[Setting1](exists)}
			MySetting1:Set[${LavishSettings[Presets].FindSet[${PresetName}].FindSetting[Setting1]}]

		if ${LavishSettings[Presets].FindSet[${PresetName}].FindSetting[Setting2](exists)}
			MySetting2:Set[${LavishSettings[Presets].FindSet[${PresetName}].FindSetting[Setting2]}]

		if ${LavishSettings[Presets].FindSet[${PresetName}].FindSetting[Setting3](exists)}
			MySetting3:Set[${LavishSettings[Presets].FindSet[${PresetName}].FindSetting[Setting3]}]

		LavishSettings[Presets]:Remove

		echo Preset loaded: ${PresetName}
	}
}

function DeletePreset()
{
	PresetName:Set[${UIElement[PresetComboBox@Settings].SelectedItem.Text}]

	if ${PresetName.Length}
	{
		echo Deleting preset: ${PresetName}
		rm "${LavishScript.HomeDirectory}/Scripts/MyScript/Presets/${PresetName}.xml"
		call RefreshPresetList
	}
}

function RefreshPresetList()
{
	variable int Count=0

	UIElement[PresetComboBox@Settings]:ClearItems
	PresetFiles:GetFiles[${LavishScript.HomeDirectory}/Scripts/MyScript/Presets/\*.xml]

	while ${Count:Inc}<=${PresetFiles.Files}
	{
		UIElement[PresetComboBox@Settings]:AddItem[${PresetFiles.File[${Count}].Filename.Left[-4]}]
	}

	if ${UIElement[PresetComboBox@Settings].Items}
	{
		UIElement[PresetComboBox@Settings]:Sort:SelectItem[1]
	}

	echo Found ${PresetFiles.Files} preset(s)
}
```

### Benefits

- **User-friendly**: Users can create their own presets
- **Dynamic**: No hardcoded preset names
- **Flexible**: Add/remove presets without script changes
- **Organized**: All presets in one directory

**Source**: bjxpbotinventory.iss:74-88

---

## Prioritized Item Lists

Process items in order of priority, automatically falling through to the next available item if the preferred one isn't available.

### Pattern

```lavishscript
variable int ItemPriority=1

function UseNextAvailableItem()
{
	; Try items in priority order
	if ${UIElement[PriorityListBox@Settings].Items} > 0 && ${ItemPriority} <= ${UIElement[PriorityListBox@Settings].Items}
	{
		; Check if current priority item exists
		if ${Me.Inventory[${UIElement[PriorityListBox@Settings].OrderedItem[${ItemPriority}]}](exists)}
		{
			echo Using: ${UIElement[PriorityListBox@Settings].OrderedItem[${ItemPriority}]}
			Me.Inventory[exactitem,${UIElement[PriorityListBox@Settings].OrderedItem[${ItemPriority}]}]:Use
		}
		else
		{
			; Item not found, try next priority
			ItemPriority:Inc
			call UseNextAvailableItem
		}
	}
	else
	{
		echo No items available from priority list
		ItemPriority:Set[1]  ; Reset for next attempt
	}
}
```

### Complete XP Potion Priority Example

```lavishscript
variable int PotionPriority=1
variable string CurrentPotion

function main()
{
	echo Starting XP potion rotation...

	; Assume priority list is:
	; 1. Distillate of Alacrity XIII
	; 2. Distillate of Alacrity XII
	; 3. Potion of Alacrity

	while 1
	{
		wait 10

		; Try to use next potion in priority order
		call UseNextPotion

		; Wait for potion to expire (check effect duration)
		wait 36000  ; 10 minutes
	}
}

function UseNextPotion()
{
	echo Checking priority list for available potions...

	PotionPriority:Set[1]  ; Start at highest priority

	while ${PotionPriority} <= ${UIElement[PotionPriorityListBox@Settings].Items}
	{
		CurrentPotion:Set[${UIElement[PotionPriorityListBox@Settings].OrderedItem[${PotionPriority}]}]

		if ${Me.Inventory[${CurrentPotion}](exists)}
		{
			echo Found priority ${PotionPriority} potion: ${CurrentPotion}
			echo Using potion...

			Me.Inventory[exactitem,${CurrentPotion}]:Use
			wait 10

			; Verify it was used
			if ${Me.CastingSpell}
			{
				echo Successfully used: ${CurrentPotion}
				return
			}
			else
			{
				echo Failed to use ${CurrentPotion}, trying next...
			}
		}

		; Try next priority
		PotionPriority:Inc
	}

	echo No potions available from priority list
}
```

### Recursive Fall-Through Example

```lavishscript
variable int ItemCounter=1

function TryUseItem()
{
	if ${ItemCounter} > ${UIElement[ItemListBox@UI].Items}
	{
		echo Exhausted all items in priority list
		ItemCounter:Set[1]
		return
	}

	variable string ItemName
	ItemName:Set[${UIElement[ItemListBox@UI].OrderedItem[${ItemCounter}]}]

	if ${Me.Inventory[${ItemName}](exists)} && ${Me.Inventory[${ItemName}].IsReady}
	{
		echo Attempting to use: ${ItemName}
		Me.Inventory[exactitem,${ItemName}]:Use
		wait 5

		; Check if successful
		if ${Me.CastingSpell}
		{
			echo Successfully using: ${ItemName}
			ItemCounter:Set[1]  ; Reset for next cycle
		}
		else
		{
			echo Failed to use ${ItemName}, trying next item
			ItemCounter:Inc
			call TryUseItem  ; Recursively try next item
		}
	}
	else
	{
		echo ${ItemName} not available, trying next item
		ItemCounter:Inc
		call TryUseItem  ; Recursively try next item
	}
}
```

### Benefits

- **Automatic fallback**: Uses best available item
- **User control**: Users set their own priority order
- **Resilient**: Doesn't fail if preferred item is missing
- **Efficient**: Stops at first successful item

**Source**: bjxpbot.iss:617-743

---

## Event-Based Text Parsing

Parse incoming game text to detect events and automatically adjust script behavior.

### Pattern

```lavishscript
atom(script) EQ2_onIncomingText(string Message)
{
	; ChatType values:
	; 15=group, 16=raid, 28=tell, 8=say

	if ${Message.Find["specific text pattern"](exists)}
	{
		; React to detected message
		call HandleDetectedMessage
	}
}

function main()
{
	; Attach event handler
	Event[EQ2_onIncomingText]:AttachAtom[EQ2_onIncomingText]

	; Main loop
	while 1
	{
		wait 10
	}
}
```

### Complete Auto-Adjust Timer Example

```lavishscript
variable(global) TimerObject PotionTimer
variable int PotionCooldown=600000  ; 10 minutes default

atom(script) EQ2_onIncomingText(string Message)
{
	; Detect potion failure message
	if ${Message.Find["You can only have one experience potion active at a time."](exists)}
	{
		echo Potion already active! Adjusting timer...

		; Reset timer to 10 minutes from now
		PotionTimer:Set[${PotionCooldown}]

		echo Next potion attempt in ${Math.Calc[${PotionTimer.TimeLeft}/1000/60]} minutes
	}

	; Detect potion success
	if ${Message.Find["You drink"](exists)} && ${Message.Find["experience"](exists)}
	{
		echo Potion consumed successfully
	}
}

function main()
{
	Event[EQ2_onIncomingText]:AttachAtom[EQ2_onIncomingText]

	echo Starting auto-potion script with smart detection...

	while 1
	{
		if ${PotionTimer.TimeLeft} == 0
		{
			call TryUsePotion
		}

		wait 10
	}
}

function TryUsePotion()
{
	if ${Me.Inventory[Potion of Adventure](exists)}
	{
		echo Attempting to use potion...
		Me.Inventory[exactitem,Potion of Adventure]:Use

		; Set timer for next attempt
		PotionTimer:Set[${PotionCooldown}]
	}
}
```

### Chat Command Detection Example

```lavishscript
variable bool ScriptPaused=FALSE

atom(script) EQ2_onIncomingText(string Message)
{
	; Detect group member commands
	if ${ChatType} == 15  ; Group chat
	{
		if ${Message.Find["!pause"](exists)}
		{
			echo Pause command received from ${Speaker}
			ScriptPaused:Set[TRUE]
		}

		if ${Message.Find["!resume"](exists)}
		{
			echo Resume command received from ${Speaker}
			ScriptPaused:Set[FALSE]
		}

		if ${Message.Find["!status"](exists)}
		{
			if ${ScriptPaused}
				eq2execute /group Script is PAUSED
			else
				eq2execute /group Script is RUNNING
		}
	}
}

function main()
{
	Event[EQ2_onIncomingText]:AttachAtom[EQ2_onIncomingText]

	echo Listening for group commands: !pause, !resume, !status

	while 1
	{
		if !${ScriptPaused}
		{
			call DoWork
		}

		wait 10
	}
}
```

### Benefits

- **Smart detection**: React to game feedback automatically
- **Error recovery**: Adjust behavior when things fail
- **User commands**: Accept commands via chat
- **Event-driven**: No polling needed

**Source**: bjxpbot.iss:1103-1113

---

## EQ2DataSourceContainer Usage

Access dynamic game data that isn't available through standard TLOs, especially useful for getting currently casting spell information.

### Pattern

```lavishscript
; Get currently casting spell name
variable string CastingSpellName
CastingSpellName:Set[${EQ2DataSourceContainer[GameData].GetDynamicData[Spells.Casting].ShortLabel}]

; Get buff duration immediately after casting
variable float BuffDuration
BuffDuration:Set[${Me.Effect[${CastingSpellName}].Duration}]
```

### Complete Potion Timer Example

```lavishscript
variable(global) TimerObject NextPotionTimer
variable string PotionSpellName
variable float PotionDuration
variable int NextUseTime

function UseXPPotion()
{
	if ${Me.Inventory[Distillate of Alacrity XIII](exists)}
	{
		echo Using XP potion...

		; Use the potion
		Me.Inventory[exactitem,Distillate of Alacrity XIII]:Use
		wait 10

		; Check if casting started
		if ${Me.CastingSpell}
		{
			; Get the spell name from what's being cast
			PotionSpellName:Set[${EQ2DataSourceContainer[GameData].GetDynamicData[Spells.Casting].ShortLabel}]
			echo Casting: ${PotionSpellName}

			; Wait for cast to complete
			wait 110

			; Get buff duration (read multiple times for accuracy)
			PotionDuration:Set[${Me.Effect[${PotionSpellName}].Duration}]
			wait 30
			PotionDuration:Set[${Me.Effect[${PotionSpellName}].Duration}]
			wait 30
			PotionDuration:Set[${Me.Effect[${PotionSpellName}].Duration}]
			wait 30
			PotionDuration:Set[${Me.Effect[${PotionSpellName}].Duration}]

			; Calculate next use time (duration + 90 second safety buffer)
			NextUseTime:Set[${Math.Calc[(${PotionDuration}*1000)+90000]}]
			NextPotionTimer:Set[${NextUseTime}]

			echo Potion will last ${PotionDuration} seconds
			echo Next potion in ${Math.Calc[${NextUseTime}/1000/60].Centi} minutes
		}
		else
		{
			echo Failed to cast potion
		}
	}
}
```

### Dynamic Buff Tracking Example

```lavishscript
variable string LastBuffCast
variable float BuffRemaining

function CastAndTrackBuff(string BuffName)
{
	echo Casting ${BuffName}...

	; Cast the buff ability
	Me.Ability[${BuffName}]:Use
	wait 5

	; Verify it's casting
	if ${Me.CastingSpell}
	{
		; Get exact spell name being cast (may differ from ability name)
		LastBuffCast:Set[${EQ2DataSourceContainer[GameData].GetDynamicData[Spells.Casting].ShortLabel}]
		echo Actually casting: ${LastBuffCast}

		; Wait for cast to complete
		wait ${Me.CastingSpell.CastingTime}
		wait 20

		; Get buff duration
		if ${Me.Effect[${LastBuffCast}](exists)}
		{
			BuffRemaining:Set[${Me.Effect[${LastBuffCast}].Duration}]
			echo ${LastBuffCast} will last for ${BuffRemaining} seconds
		}
	}
}
```

### Benefits

- **Real-time data**: Get info about ongoing casts
- **Precise tracking**: Know exact buff durations
- **Spell name mapping**: Handle ability vs spell name differences
- **Dynamic**: Works with any potion/buff

**Source**: bjxpbot.iss:563-664

---

## Randomized Movement

Create natural-looking movement patterns with randomized delays and directions to avoid detection patterns.

### Pattern

```lavishscript
variable int RandomHeading
variable int RandomWaitTime
variable point3f InitialLocation
variable float InitialHeading
variable int MaxDistance=4

function RandomShuffle()
{
	; Save current position
	InitialLocation:Set[${Me.Loc}]
	InitialHeading:Set[${Me.Heading}]

	; Generate random heading (0-360 degrees)
	RandomHeading:Set[${Math.Rand[360]}]

	; Face random direction
	face ${RandomHeading}

	; Move for random duration (2-5 seconds)
	RandomWaitTime:Set[${Math.Rand[30]:Inc[20]}]
	eq2press -hold w
	wait ${RandomWaitTime}
	eq2press -release w

	; Return to original position if too far
	if ${Math.Distance[${Me.Loc},${InitialLocation}]} > ${MaxDistance}
	{
		call ReturnToHome
	}

	; Face original direction
	face ${InitialHeading}
}
```

### Class-Based Positioning Example

```lavishscript
variable int FacingScout
variable int FacingMage
variable int FacingPriest
variable int ClassHeading
variable float InitialHeading
variable point3f InitialLocation
variable int MaxDistance=4

function PositionByArchetype()
{
	; Save starting position
	InitialHeading:Set[${Me.Heading}]
	InitialLocation:Set[${Me.Loc}]

	; Generate base headings for each archetype (spread out around raid)
	FacingScout:Set[${Math.Rand[65]:Inc[45]}]      ; 45-110 degrees
	FacingMage:Set[${Math.Rand[155]:Inc[45]}]      ; 45-200 degrees
	FacingPriest:Set[${Math.Rand[290]:Inc[45]}]    ; 45-335 degrees

	; Adjust heading based on class
	if ${Me.Archetype.Equal[Scout]}
	{
		if ${Me.Class.Equal[rogue]}
			ClassHeading:Set[${Math.Calc[${FacingScout}+10]}]
		elseif ${Me.Class.Equal[predator]}
			ClassHeading:Set[${Math.Calc[${FacingScout}+20]}]
		elseif ${Me.Class.Equal[bard]}
			ClassHeading:Set[${Math.Calc[${FacingScout}-20]}]
		elseif ${Me.Class.Equal[animalist]}
			ClassHeading:Set[${Math.Calc[${FacingScout}-10]}]

		face ${ClassHeading}
		eq2press -hold w
		wait 4
		eq2press -release w
	}
	elseif ${Me.Archetype.Equal[Mage]}
	{
		if ${Me.Class.Equal[sorcerer]}
			ClassHeading:Set[${Math.Calc[${FacingMage}+10]}]
		elseif ${Me.Class.Equal[summoner]}
			ClassHeading:Set[${Math.Calc[${FacingMage}-20]}]
		elseif ${Me.Class.Equal[enchanter]}
			ClassHeading:Set[${Math.Calc[${FacingMage}+20]}]

		face ${ClassHeading}
		eq2press -hold w
		wait 3
		eq2press -release w
	}
	elseif ${Me.Archetype.Equal[Priest]}
	{
		if ${Me.Class.Equal[druid]}
			ClassHeading:Set[${Math.Calc[${FacingPriest}+10]}]
		elseif ${Me.Class.Equal[shaman]}
			ClassHeading:Set[${Math.Calc[${FacingPriest}+20]}]
		elseif ${Me.Class.Equal[cleric]}
			ClassHeading:Set[${Math.Calc[${FacingPriest}-20]}]

		face ${ClassHeading}
		eq2press -hold w
		wait 3
		eq2press -release w
	}

	; Return to home if moved too far
	if ${Math.Distance[${Me.Loc},${InitialLocation}]} > ${MaxDistance}
	{
		while ${Math.Distance[${Me.Loc},${InitialLocation}]} > 3
		{
			face ${InitialLocation.X} ${InitialLocation.Z}
			eq2press -hold w
		}
		eq2press -release w
	}

	; Face original heading
	face ${InitialHeading}
}
```

### Benefits

- **Anti-detection**: Randomized patterns look more human
- **Organized**: Class-based positioning for raids
- **Safe**: Auto-return prevents wandering too far
- **Realistic**: Variable timing and directions

**Source**: bjshuffle.iss:24-26,37-53

---

## Item Usage Verification

Verify that item usage actually succeeded by checking cast state and effects.

### Pattern

```lavishscript
variable string ItemToUse

function UseItemWithVerification(string ItemName)
{
	if ${Me.Inventory[${ItemName}](exists)}
	{
		echo Attempting to use: ${ItemName}

		; Use the item
		ItemToUse:Set[${ItemName}]
		Me.Inventory[exactitem,${ItemToUse}]:Use
		wait 10

		; Verify casting started
		if ${Me.CastingSpell}
		{
			echo Successfully using: ${ItemName}
			return TRUE
		}
		else
		{
			echo Failed to use ${ItemName}
			return FALSE
		}
	}
	else
	{
		echo ${ItemName} not found in inventory
		return FALSE
	}
}
```

### Complete Potion Usage Example

```lavishscript
variable string ItemToUse
variable string ExpectedEffect

function UseVitalityPotion()
{
	wait !${Me.InCombat} && !${Me.CastingSpell}

	if ${Me.Inventory[Orb of Concentrated Memories](exists)} && ${Me.Inventory[Orb of Concentrated Memories].IsReady} && ${Me.Vitality} == 0
	{
		echo Orb of Concentrated Memories detected

		ItemToUse:Set[Orb of Concentrated Memories]
		ExpectedEffect:Set[Orb of Concentrated Memories]

		; Use the orb
		Me.Inventory[exactitem,${ItemToUse}]:Use
		wait 10

		; Verify it's casting and check for expected effect name
		if ${Me.CastingSpell} && ${EQ2DataSourceContainer[GameData].GetDynamicData[Spells.Casting].ShortLabel.Find[${ExpectedEffect}](exists)}
		{
			echo Successfully used: ${ItemToUse}
			return TRUE
		}
		else
		{
			echo Failed to use ${ItemToUse}
			return FALSE
		}
	}
	elseif ${Me.Inventory[Potion of Vitality](exists)} && ${Me.Vitality} == 0
	{
		echo Potion of Vitality detected

		ItemToUse:Set[Potion of Vitality]
		ExpectedEffect:Set[Rested Mind and Body]

		; Use the potion
		Me.Inventory[exactitem,${ItemToUse}]:Use
		wait 10

		; Verify casting and check for "Rested Mind and Body" effect
		if ${Me.CastingSpell} && ${EQ2DataSourceContainer[GameData].GetDynamicData[Spells.Casting].ShortLabel.Find[${ExpectedEffect}](exists)}
		{
			echo Successfully used: ${ItemToUse}
			return TRUE
		}
		else
		{
			echo Failed to use ${ItemToUse}
			return FALSE
		}
	}

	return FALSE
}
```

### Multiple Verification Checks Example

```lavishscript
function UseItemSafely(string ItemName, string ExpectedBuff)
{
	; Pre-checks
	if !${Me.Inventory[${ItemName}](exists)}
	{
		echo ${ItemName} not in inventory
		return FALSE
	}

	if !${Me.Inventory[${ItemName}].IsReady}
	{
		echo ${ItemName} is on cooldown
		return FALSE
	}

	if ${Me.InCombat}
	{
		echo Cannot use ${ItemName} while in combat
		return FALSE
	}

	; Use item
	echo Using ${ItemName}...
	Me.Inventory[exactitem,${ItemName}]:Use
	wait 10

	; Verify cast started
	if !${Me.CastingSpell}
	{
		echo Cast did not start for ${ItemName}
		return FALSE
	}

	; Wait for cast to complete
	wait ${Me.CastingSpell.CastingTime}
	wait 20

	; Verify buff applied
	if ${Me.Effect[${ExpectedBuff}](exists)}
	{
		echo ${ItemName} successfully applied ${ExpectedBuff}
		return TRUE
	}
	else
	{
		echo ${ItemName} did not apply expected buff
		return FALSE
	}
}
```

### Benefits

- **Reliability**: Know if item actually worked
- **Error detection**: Catch failures immediately
- **Exact matching**: `exactitem` prevents partial matches
- **Ready check**: Verify item is off cooldown

**Source**: bjxpbot.iss:560-572,627

---

## ApplyVerb for Object Interaction

Interact with game objects (chests, corpses, NPCs) using the ApplyVerb command.

### Pattern

```lavishscript
; Open a chest
EQ2execute "/apply_verb ${Actor[Chest Name].ID} Open"

; Loot a corpse
EQ2execute "/apply_verb ${Actor[Corpse].ID} Loot"

; Inspect an NPC/object
EQ2execute "/apply_verb ${Actor[Object Name].ID} Inspect"
```

### Complete Auto-Looter Example

```lavishscript
variable int ScanRange=20
variable point3f HomeLocation

function main()
{
	HomeLocation:Set[${Me.Loc}]

	echo Starting auto-looter (${ScanRange}m range)...

	while 1
	{
		; Check for treasure chest
		if ${Actor[chest,"Treasure Chest"].Name(exists)} && ${Actor[chest,"Treasure Chest"].Distance} <= ${ScanRange}
		{
			call LootChest "Treasure Chest"
		}
		; Check for ornate chest
		elseif ${Actor[chest,"Ornate Chest"].Name(exists)} && ${Actor[chest,"Ornate Chest"].Distance} <= ${ScanRange}
		{
			call LootChest "Ornate Chest"
		}
		; Check for corpses
		elseif ${Actor[npc,"corpse"].Name(exists)} && ${Actor[npc,"corpse"].Distance} <= ${ScanRange}
		{
			call LootCorpse
		}

		wait 10
	}
}

function LootChest(string ChestType)
{
	echo ${ChestType} found at ${Actor[chest,${ChestType}].Distance.Centi}m

	; Face the chest
	face "${ChestType}"
	wait 10

	; Move to chest if needed
	if ${Actor[chest,${ChestType}].Distance} > 2
	{
		echo Moving to ${ChestType}...

		while ${Actor[chest,${ChestType}].Distance} > 2
		{
			eq2press -hold w
			face "${ChestType}"
		}
		eq2press -release w

		echo Arrived at ${ChestType}
	}

	; Open the chest
	echo Opening ${ChestType}...
	EQ2execute "/apply_verb ${Actor[${ChestType}].ID} Open"
	wait 20

	echo ${ChestType} looted
}

function LootCorpse()
{
	echo Corpse found at ${Actor[npc,"corpse"].Distance.Centi}m

	; Face corpse
	face corpse
	wait 10

	; Move to corpse if needed
	if ${Actor[npc,"corpse"].Distance} > 2
	{
		while ${Actor[npc,"corpse"].Distance} > 2
		{
			eq2press -hold w
			face corpse
		}
		eq2press -release w
	}

	; Loot corpse
	EQ2execute "/apply_verb ${Actor[Corpse].ID} Loot"
	wait 20
}
```

### Distance Check and Movement Pattern

```lavishscript
function InteractWithObject(string ObjectType, string ObjectName, string Verb)
{
	if !${Actor[${ObjectType},${ObjectName}].Name(exists)}
	{
		echo ${ObjectName} not found
		return
	}

	echo Found ${ObjectName} at ${Actor[${ObjectType},${ObjectName}].Distance.Centi}m

	; Face the object
	face "${ObjectName}"
	wait 10

	; Move closer if needed (within 2 meters)
	if ${Actor[${ObjectType},${ObjectName}].Distance} > 2
	{
		echo Moving to ${ObjectName}...

		while ${Actor[${ObjectType},${ObjectName}].Distance} > 2 && ${Actor[${ObjectType},${ObjectName}].Name(exists)}
		{
			eq2press -hold w
			face "${ObjectName}"
			wait 1
		}
		eq2press -release w

		echo Arrived at ${ObjectName}
		wait 5
	}

	; Apply the verb
	echo Using ${Verb} on ${ObjectName}...
	EQ2execute "/apply_verb ${Actor[${ObjectName}].ID} ${Verb}"
	wait 10
}

; Usage examples:
call InteractWithObject "chest" "Treasure Chest" "Open"
call InteractWithObject "special" "Druzaic Shrine" "Inspect"
call InteractWithObject "npc" "corpse" "Loot"
```

### Benefits

- **Reliable interaction**: Works with any game object
- **ID-based**: Uses actor ID for precision
- **Flexible**: Any verb (Open, Loot, Inspect, etc.)
- **Distance aware**: Move closer before interacting

**Source**: bjlooter.iss:43,79,115,151,187,195

---

## ReplyDialog for NPC Conversations

Interact with NPC dialog windows by choosing numbered responses.

### Pattern

```lavishscript
; Choose first dialog option
ReplyDialog:Choose[1]

; Choose second dialog option
ReplyDialog:Choose[2]

; Multi-step conversation
ReplyDialog:Choose[1]
wait 30
ReplyDialog:Choose[1]
wait 30
ReplyDialog:Choose[2]
```

### Complete Shrine Clicker Example

```lavishscript
variable int AttemptCount=0
variable int NextAttemptDelay

function main()
{
	echo Starting shrine interaction script...

	while 1
	{
		; Random delay between attempts (10-15 minutes)
		NextAttemptDelay:Set[${Math.Rand[300000]:Inc[600000]}]

		AttemptCount:Inc
		echo Attempt ${AttemptCount} to interact with shrine

		; Check for shrine nearby
		if ${Actor[special,"Druzaic Shrine"].Name(exists)} && ${Actor[special,"Druzaic Shrine"].Distance} <= 10
		{
			echo Shrine detected! Interacting...

			; Inspect the shrine to open dialog
			EQ2execute "/apply_verb ${Actor[druzaic shrine].ID} inspect"
			wait 30

			; Choose first dialog option
			ReplyDialog:Choose[1]
			wait 30

			; Choose first option again (confirmation)
			ReplyDialog:Choose[1]
			wait 30

			echo Shrine interaction complete!

			; End script after successful interaction
			return
		}
		else
		{
			echo Shrine not detected. Waiting ${Math.Calc[${NextAttemptDelay}/1000/60]} minutes...
			wait ${Math.Calc[${NextAttemptDelay}/100]}
		}
	}
}
```

### Quest Dialog Example

```lavishscript
function AcceptQuest(string NPCName)
{
	if !${Actor[npc,${NPCName}].Name(exists)}
	{
		echo ${NPCName} not found
		return FALSE
	}

	echo Talking to ${NPCName}...

	; Target and hail NPC
	Actor[npc,${NPCName}]:DoTarget
	wait 5
	eq2execute /hail
	wait 20

	; Wait for dialog window
	if ${EQ2UIPage[Conversation](exists)}
	{
		echo Dialog opened. Accepting quest...

		; Navigate through dialog
		ReplyDialog:Choose[1]  ; "Tell me more"
		wait 20

		ReplyDialog:Choose[1]  ; "I'll help you"
		wait 20

		ReplyDialog:Choose[1]  ; "Accept quest"
		wait 20

		echo Quest accepted from ${NPCName}
		return TRUE
	}
	else
	{
		echo Dialog did not open
		return FALSE
	}
}
```

### Multi-Branch Dialog Example

```lavishscript
function NavigateDialog(int Option1, int Option2, int Option3)
{
	echo Starting dialog navigation...

	; First choice
	if ${Option1} > 0
	{
		echo Choosing option ${Option1}
		ReplyDialog:Choose[${Option1}]
		wait 30
	}

	; Second choice
	if ${Option2} > 0
	{
		echo Choosing option ${Option2}
		ReplyDialog:Choose[${Option2}]
		wait 30
	}

	; Third choice
	if ${Option3} > 0
	{
		echo Choosing option ${Option3}
		ReplyDialog:Choose[${Option3}]
		wait 30
	}

	echo Dialog navigation complete
}

; Usage:
call NavigateDialog 1 1 2  ; Choose option 1, then 1, then 2
```

### Benefits

- **Quest automation**: Accept quests without clicking
- **Repeatable interactions**: Shrine clicks, turn-ins
- **Multi-step dialogs**: Navigate complex conversations
- **Simple syntax**: Just specify the option number

**Source**: bjmagic.iss:51-55

---

## Audio Notifications

Play sound files to alert the user when important events occur.

### Pattern

```lavishscript
; Play a WAV file
play "${LavishScript.HomeDirectory}/Scripts/MyScript/Sounds/alert.wav"

; With error handling
if ${System.FileExists["${LavishScript.HomeDirectory}/Scripts/MyScript/Sounds/alert.wav"]}
	play "${LavishScript.HomeDirectory}/Scripts/MyScript/Sounds/alert.wav"
else
	echo Sound file not found
```

### Complete Alert System Example

```lavishscript
variable bool AlertSoundPlayed=FALSE
variable int AlertCooldown=300  ; 5 minutes between alerts

function main()
{
	echo Starting monitoring with audio alerts...

	while 1
	{
		; Check for alert condition
		if ${Me.Health} <= 20
		{
			call PlayLowHealthAlert
		}

		; Check if level limit reached
		if ${Me.Level} >= ${TargetLevel}
		{
			call PlayLevelReachedAlert
		}

		wait 10
	}
}

function PlayLowHealthAlert()
{
	if !${AlertSoundPlayed}
	{
		echo LOW HEALTH! Playing alert sound...
		play "${LavishScript.HomeDirectory}/Scripts/AutoBot/Sounds/chatalarm.wav"
		AlertSoundPlayed:Set[TRUE]

		; Reset flag after cooldown
		wait ${AlertCooldown}
		AlertSoundPlayed:Set[FALSE]
	}
}

function PlayLevelReachedAlert()
{
	echo Target level reached! Playing notification...
	play "${LavishScript.HomeDirectory}/Scripts/AutoBot/Sounds/ping.wav"
}
```

### Multiple Sound Types Example

```lavishscript
function PlayAlertSound(string AlertType)
{
	variable string SoundFile

	switch ${AlertType}
	{
		case "limit_reached"
			SoundFile:Set["${LavishScript.HomeDirectory}/Scripts/XPBot/Sounds/ping.wav"]
			break
		case "player_dead"
			SoundFile:Set["${LavishScript.HomeDirectory}/Scripts/XPBot/Sounds/chatalarm.wav"]
			break
		case "error"
			SoundFile:Set["${LavishScript.HomeDirectory}/Scripts/XPBot/Sounds/error.wav"]
			break
		case "success"
			SoundFile:Set["${LavishScript.HomeDirectory}/Scripts/XPBot/Sounds/success.wav"]
			break
		default
			echo Unknown alert type: ${AlertType}
			return
	}

	; Verify file exists before playing
	if ${System.FileExists["${SoundFile}"]}
	{
		echo Playing ${AlertType} alert
		play "${SoundFile}"
	}
	else
	{
		echo Sound file not found: ${SoundFile}
	}
}

; Usage:
call PlayAlertSound "limit_reached"
call PlayAlertSound "player_dead"
```

### Configurable Alert System

```lavishscript
variable bool EnableSoundAlerts=TRUE
variable bool SoundPlayed=FALSE
variable string AlertSoundFile="${LavishScript.HomeDirectory}/Scripts/MyScript/Sounds/ping.wav"

function TriggerAlert(string Message)
{
	echo ALERT: ${Message}

	if ${EnableSoundAlerts} && !${SoundPlayed}
	{
		if ${System.FileExists["${AlertSoundFile}"]}
		{
			play "${AlertSoundFile}"
			SoundPlayed:Set[TRUE]
			echo Alert sound played
		}
		else
		{
			echo Alert sound file not found: ${AlertSoundFile}
		}
	}
}

function ResetAlertSound()
{
	SoundPlayed:Set[FALSE]
}
```

### Benefits

- **User attention**: Alert when script needs intervention
- **Background operation**: Work while script runs
- **Configurable**: Enable/disable via settings
- **Prevents spam**: Boolean flags prevent repeated sounds

**Source**: bjxpbot.iss:841,1014

---

## Input Validation Patterns

Extensive validation ensures user input is correct before script execution, preventing errors and providing helpful feedback.

### Pattern

```lavishscript
function ValidateSettings()
{
	; Check for required text input
	if ${SettingValue.Equal[ ]} || ${SettingValue.Equal[NULL]}
	{
		echo ERROR: Please enter a value for SettingName
		StatusDisplay:Set["Please enter a value for SettingName"]
		call CleanupAndExit
		return FALSE
	}

	; Check numeric range
	if ${NumericSetting} > ${MaxValue}
	{
		echo ERROR: Value must be ${MaxValue} or less
		StatusDisplay:Set["Value must be ${MaxValue} or less"]
		call CleanupAndExit
		return FALSE
	}

	; Check for dropdown selection
	if ${ComboBoxValue.Equal["Please choose..."]}
	{
		echo ERROR: Please select an option
		StatusDisplay:Set["Please select an option"]
		call CleanupAndExit
		return FALSE
	}

	return TRUE
}
```

### Complete Validation Example

```lavishscript
variable int CurrentMaxLevel=125
variable int CurrentMaxAA=400
variable int TargetLevel
variable int TargetAA
variable string LimitAction

function main()
{
	echo Validating settings before starting...

	if !${call ValidateAllSettings}
	{
		echo Validation failed. Please correct settings and try again.
		return
	}

	echo All settings validated. Starting script...

	; Continue with script
	while 1
	{
		call DoWork
		wait 10
	}
}

function ValidateAllSettings()
{
	; Validate max level entry
	if ${CurrentMaxLevel} == NULL
	{
		echo ERROR: Please enter the current EQ2 max level
		StatusDisplay:Set["Please enter the current EQ2 max level"]
		return FALSE
	}

	; Validate max AA entry
	if ${CurrentMaxAA} == NULL
	{
		echo ERROR: Please enter the current EQ2 max AA level
		StatusDisplay:Set["Please enter the current EQ2 max AA level"]
		return FALSE
	}

	; Check if already maxed
	if ${Me.Level} == ${CurrentMaxLevel} && ${Me.TotalEarnedAPs} == ${CurrentMaxAA}
	{
		echo ERROR: You're already max level and max AA
		StatusDisplay:Set["Already maxed - don't waste resources"]
		return FALSE
	}

	; Validate target level
	if ${TargetLevel} > ${CurrentMaxLevel}
	{
		echo ERROR: Target level cannot exceed max level (${CurrentMaxLevel})
		StatusDisplay:Set["Target level too high"]
		return FALSE
	}

	if ${TargetLevel} == ${Me.Level}
	{
		echo ERROR: Target level cannot equal current level
		StatusDisplay:Set["Target level cannot equal current level"]
		return FALSE
	}

	if ${TargetLevel} < ${Me.Level}
	{
		echo ERROR: Target level cannot be less than current level
		StatusDisplay:Set["Target level cannot be less than current level"]
		return FALSE
	}

	; Validate limit action selection
	if ${LimitAction.Equal["Please choose..."]}
	{
		echo ERROR: Please select what to do when limit is reached
		StatusDisplay:Set["Please select a limit action"]
		return FALSE
	}

	; All validations passed
	echo All validations passed!
	return TRUE
}
```

### Progressive Validation Example

```lavishscript
function ValidateWithFeedback()
{
	variable int ErrorCount=0

	echo ========================================
	echo Running Settings Validation...
	echo ========================================

	; Check each setting and count errors
	if ${Setting1.Equal[ ]}
	{
		echo [ERROR] Setting 1 is empty
		ErrorCount:Inc
	}
	else
	{
		echo [OK] Setting 1: ${Setting1}
	}

	if ${Setting2} > 100
	{
		echo [ERROR] Setting 2 must be 100 or less (currently ${Setting2})
		ErrorCount:Inc
	}
	else
	{
		echo [OK] Setting 2: ${Setting2}
	}

	if ${Setting3.Equal["Please choose..."]}
	{
		echo [ERROR] Setting 3 not selected
		ErrorCount:Inc
	}
	else
	{
		echo [OK] Setting 3: ${Setting3}
	}

	echo ========================================

	if ${ErrorCount} > 0
	{
		echo Validation FAILED: ${ErrorCount} error(s) found
		echo Please correct the errors above and try again
		return FALSE
	}
	else
	{
		echo Validation PASSED: All settings are valid
		return TRUE
	}
}
```

### Benefits

- **Prevents errors**: Catch bad input before execution
- **User-friendly**: Clear error messages
- **Status updates**: Visual feedback in UI
- **Graceful exit**: Clean shutdown on validation failure

**Source**: bjxpbot.iss:209-468

---

## Progressive XP Tracking

Track experience gained across multiple levels, including per-hour rates for ADV, AA, and TS experience.

### Pattern

```lavishscript
; Basic tracking variables
variable int StartLevel
variable float StartXP
variable int CurrentLevel
variable float CurrentXP
variable int GainedLevels
variable float TotalXP

function InitializeTracking()
{
	StartLevel:Set[${Me.Level}]
	StartXP:Set[${Me.Exp}]
}

function CalculateGainedXP()
{
	CurrentLevel:Set[${Me.Level}]
	CurrentXP:Set[${Me.Exp}]

	if ${StartLevel} < ${CurrentLevel}
	{
		; Leveled up - calculate total including full levels
		GainedLevels:Set[${Math.Calc[${CurrentLevel}-${StartLevel}]}]
		TotalXP:Set[${Math.Calc[(${GainedLevels}*100)-100+${CurrentXP}+(100-${StartXP})]}]
	}
	else
	{
		; Same level - simple subtraction
		TotalXP:Set[${Math.Calc[${CurrentXP}-${StartXP}]}]
	}

	echo Total XP gained: ${TotalXP.Centi}%
}
```

### Complete Multi-Level XP Tracker

```lavishscript
variable int StartLevel
variable float StartXP
variable int CurrentLevel
variable float CurrentXP
variable int GainedLevels
variable float TotalXPGained
variable float TotalXPSinceLastLevel

variable int DisplayLevels
variable float DisplayPercent
variable float DisplayPercentSinceLastLevel

variable float TimeRunningHours
variable float LevelsPerHour
variable float PercentPerHour

function main()
{
	call InitializeTracking

	echo XP Tracker started at level ${StartLevel} (${StartXP}%)

	while 1
	{
		call UpdateXPCalculations
		call DisplayProgress
		wait 100  ; Update every 10 seconds
	}
}

function InitializeTracking()
{
	StartLevel:Set[${Me.Level}]
	StartXP:Set[${Me.Exp}]
	CurrentLevel:Set[${Me.Level}]
	CurrentXP:Set[${Me.Exp}]
}

function UpdateXPCalculations()
{
	CurrentLevel:Set[${Me.Level}]
	CurrentXP:Set[${Me.Exp}]

	; Calculate hours running
	TimeRunningHours:Set[${Math.Calc[(((${Script.RunningTime}/1000)/60/60))].Milli}]

	if ${StartLevel} < ${CurrentLevel}
	{
		; Gained at least one level
		GainedLevels:Set[${Math.Calc[${CurrentLevel}-${StartLevel}]}]

		; Total XP = (full levels * 100) - 100 + current% + (100 - starting%)
		; The -100 accounts for the starting level not being a "full" gain
		TotalXPGained:Set[${Math.Calc[(${GainedLevels}*100)-100+${CurrentXP}+(100-${StartXP})]}]

		; XP since last level-up
		TotalXPSinceLastLevel:Set[${CurrentXP}]
	}
	else
	{
		; Same level - simple calculation
		TotalXPGained:Set[${Math.Calc[${CurrentXP}-${StartXP}]}]
		TotalXPSinceLastLevel:Set[${TotalXPGained}]
	}

	; Format for display (levels.percent)
	DisplayLevels:Set[${Math.Calc[${TotalXPGained}/100].Int}]
	DisplayPercent:Set[${Math.Calc[${TotalXPGained}%100]}]
	DisplayPercentSinceLastLevel:Set[${Math.Calc[${TotalXPSinceLastLevel}%100]}]

	; Calculate per-hour rate
	if ${TimeRunningHours} > 0
	{
		LevelsPerHour:Set[${Math.Calc[${TotalXPGained}/${TimeRunningHours}/100].Int}]
		PercentPerHour:Set[${Math.Calc[(${TotalXPGained}/${TimeRunningHours})%100]}]
	}
}

function DisplayProgress()
{
	echo ========================================
	echo XP Progress Report
	echo ========================================
	echo Total Gained: ${DisplayLevels} levels, ${DisplayPercent.Centi}%
	echo Current Level: ${CurrentLevel} (${DisplayPercentSinceLastLevel.Centi}%)

	if ${TimeRunningHours} > 0
	{
		echo Rate: ${LevelsPerHour}.${PercentPerHour.Centi}% per hour
	}

	echo ========================================
}
```

### AA and TS Tracking Example

```lavishscript
variable int StartAA
variable float StartAAXP
variable int CurrentAA
variable float CurrentAAXP
variable float TotalAAXPGained

variable int StartTS
variable float StartTSXP
variable int CurrentTS
variable float CurrentTSXP
variable float TotalTSXPGained

function InitializeAllTracking()
{
	; Adventure XP
	StartLevel:Set[${Me.Level}]
	StartXP:Set[${Me.Exp}]

	; AA XP
	StartAA:Set[${Me.TotalEarnedAPs}]
	StartAAXP:Set[${Me.APExp}]

	; Tradeskill XP
	StartTS:Set[${Me.TSLevel}]
	StartTSXP:Set[${Me.TSExp}]

	echo Tracking initialized:
	echo   ADV: Level ${StartLevel} (${StartXP}%)
	echo   AA: ${StartAA} points (${StartAAXP}%)
	echo   TS: Level ${StartTS} (${StartTSXP}%)
}

function UpdateAATracking()
{
	CurrentAA:Set[${Me.TotalEarnedAPs}]
	CurrentAAXP:Set[${Me.APExp}]

	if ${StartAA} < ${CurrentAA}
	{
		; Gained AA points
		variable int GainedAA
		GainedAA:Set[${Math.Calc[${CurrentAA}-${StartAA}]}]
		TotalAAXPGained:Set[${Math.Calc[(${GainedAA}*100)-100+${CurrentAAXP}+(100-${StartAAXP})]}]
	}
	else
	{
		; Same AA level
		TotalAAXPGained:Set[${Math.Calc[${CurrentAAXP}-${StartAAXP}]}]
	}
}

function UpdateTSTracking()
{
	CurrentTS:Set[${Me.TSLevel}]
	CurrentTSXP:Set[${Me.TSExp}]

	if ${StartTS} < ${CurrentTS}
	{
		; Gained TS levels
		variable int GainedTS
		GainedTS:Set[${Math.Calc[${CurrentTS}-${StartTS}]}]
		TotalTSXPGained:Set[${Math.Calc[(${GainedTS}*100)-100+${CurrentTSXP}+(100-${StartTSXP})]}]
	}
	else
	{
		; Same TS level
		TotalTSXPGained:Set[${Math.Calc[${CurrentTSXP}-${StartTSXP}]}]
	}
}
```

### Benefits

- **Accurate across levels**: Handles level-ups correctly
- **Multiple XP types**: Track ADV, AA, TS separately
- **Rate calculation**: Shows progress per hour
- **Long-term tracking**: Works for extended sessions

**Source**: xpcalclevel.iss:105-229

---

## Script Existence Checking

Prevent duplicate script instances and manage script dependencies.

### Pattern

```lavishscript
; Check if script is already running
if ${Script[scriptname](exists)}
{
	echo Script is already running
}
else
{
	echo Starting script
	runscript scriptname
}

; End a script if it exists
if ${Script[scriptname](exists)}
{
	endscript scriptname
}
```

### Shell Script Pattern

```lavishscript
function main()
{
	; Check if main script is already running
	if ${Script[MainScript](exists)}
	{
		echo MainScript is already running
		return
	}
	else
	{
		echo Starting MainScript...
		runscript MainScript
	}
}
```

### Complete Multi-Script Manager

```lavishscript
variable bool HelperScriptRequired=TRUE
variable bool UIScriptRequired=TRUE

function main()
{
	echo Starting script manager...

	; Ensure helper script is running
	if ${HelperScriptRequired}
	{
		if !${Script[HelperScript](exists)}
		{
			echo Starting HelperScript...
			runscript HelperScript
			wait 10
		}
		else
		{
			echo HelperScript already running
		}
	}

	; Ensure UI script is running
	if ${UIScriptRequired}
	{
		if !${Script[UIScript](exists)}
		{
			echo Starting UIScript...
			runscript UIScript
			wait 10
		}
		else
		{
			echo UIScript already running
		}
	}

	echo All required scripts are running

	; Main loop
	while 1
	{
		call DoWork
		wait 10
	}
}

function atexit()
{
	echo Shutting down and cleaning up helper scripts...

	; Stop helper scripts if they're running
	if ${Script[HelperScript](exists)}
	{
		echo Stopping HelperScript
		endscript HelperScript
	}

	if ${Script[UIScript](exists)}
	{
		echo Stopping UIScript
		endscript UIScript
	}

	echo Cleanup complete
}
```

### Safe Script Restart Pattern

```lavishscript
function RestartScript(string ScriptName)
{
	echo Restarting ${ScriptName}...

	; Stop it if running
	if ${Script[${ScriptName}](exists)}
	{
		echo Stopping ${ScriptName}
		endscript ${ScriptName}
		wait 5
	}

	; Verify it stopped
	if ${Script[${ScriptName}](exists)}
	{
		echo WARNING: ${ScriptName} did not stop cleanly
		return FALSE
	}

	; Start it
	echo Starting ${ScriptName}
	runscript ${ScriptName}
	wait 10

	; Verify it started
	if ${Script[${ScriptName}](exists)}
	{
		echo ${ScriptName} restarted successfully
		return TRUE
	}
	else
	{
		echo ERROR: ${ScriptName} failed to start
		return FALSE
	}
}
```

### Benefits

- **Prevents duplicates**: Won't run multiple instances
- **Clean shutdown**: Stop dependent scripts on exit
- **Dependency management**: Ensure required scripts are running
- **Safe restarts**: Verify stop before starting

**Source**: bjshuffleSHELL.iss:3-10

---

## ExecuteQueued Loop Patterns

Handle queued commands from UI interactions or other scripts.

### Pattern 1: Simple Queue Check

```lavishscript
function main()
{
	while 1
	{
		; Check if there are queued commands
		if ${QueuedCommands}
			ExecuteQueued

		; Do other work
		call DoMainWork

		waitframe
	}
}
```

### Pattern 2: Queue-Only Script

```lavishscript
function main()
{
	; Script that only processes queued commands
	while 1
	{
		ExecuteQueued
		waitframe
	}
}
```

### Complete UI Command Handler

```lavishscript
variable bool ScriptRunning=TRUE

function main()
{
	echo Main script started with queue support

	while ${ScriptRunning}
	{
		; Process any queued commands first
		if ${QueuedCommands}
		{
			ExecuteQueued
		}

		; Do regular work
		call MainLoop

		waitframe
	}

	echo Main script ending
}

function MainLoop()
{
	; Your main script logic here
	wait 10
}

; This function can be called from UI via queue
function PauseScript()
{
	echo Pause requested via queue
	ScriptRunning:Set[FALSE]
}

; This function can be called from UI via queue
function RefreshData()
{
	echo Refresh requested via queue
	call LoadData
}
```

### Inventory Refresh Pattern

```lavishscript
function main()
{
	; Dedicated script for handling inventory refresh requests
	while 1
	{
		; Wait for and execute queued commands
		ExecuteQueued
		waitframe
	}
}

; Called from UI button via Script:QueueCommand
function RefreshInventoryQueue()
{
	echo Refreshing inventory list...
	call RefreshInventory
	wait 10
	call RefreshInventory
	wait 10
	call RefreshInventory
	echo Inventory refresh complete
}

function RefreshInventory()
{
	variable index:item Items
	variable iterator ItemIterator

	; Clear existing list
	UIElement[InventoryListBox@UI]:ClearItems

	; Query inventory
	Me:QueryInventory[Items, Location == "Inventory"]
	Items:GetIterator[ItemIterator]

	; Populate list
	if ${ItemIterator:First(exists)}
	{
		do
		{
			if !${ItemIterator.Value.IsContainer}
			{
				UIElement[InventoryListBox@UI]:AddItem[${ItemIterator.Value.Name}]
			}
		}
		while ${ItemIterator:Next(exists)}
	}
}
```

### Multi-Function Queue Handler

```lavishscript
function main()
{
	echo Starting multi-function queue handler

	while 1
	{
		; Execute any queued commands from UI
		if ${QueuedCommands}
		{
			ExecuteQueued
		}

		; Continue normal operations
		call NormalOperations

		waitframe
	}
}

function NormalOperations()
{
	; Regular script work
	wait 10
}

; UI-callable functions via Script:QueueCommand

function SaveCurrentSettings()
{
	echo Saving settings (called from UI)...
	call SaveSettings
}

function LoadPreset(string PresetName)
{
	echo Loading preset: ${PresetName} (called from UI)
	call LoadSettings ${PresetName}
}

function RefreshList(string ListType)
{
	echo Refreshing ${ListType} list (called from UI)

	switch ${ListType}
	{
		case "inventory"
			call RefreshInventoryList
			break
		case "presets"
			call RefreshPresetList
			break
		default
			echo Unknown list type: ${ListType}
	}
}
```

### Benefits

- **UI integration**: Handle button clicks and UI events
- **Async communication**: Scripts can queue commands to each other
- **Non-blocking**: Doesn't interrupt main loop
- **Flexible**: Any function can be queued

**Source**: bjauction.iss:58-60, bjxpbotinventory.iss:3-9

---

## Complete Working Examples

### Example 1: Smart Auto-Potion Script

```lavishscript
; Complete auto-potion script with all patterns combined

variable(global) TimerObject PotionTimer
variable int PotionPriority=1
variable string CurrentPotion
variable string PotionSpellName
variable float PotionDuration
variable int NextUseTime
variable int PotionCooldown=600000  ; 10 minutes default
variable settingsetref Settings
variable settingsetref _ref

atom(script) EQ2_onIncomingText(string Message)
{
	if ${Message.Find["You can only have one experience potion active at a time."](exists)}
	{
		echo Potion already active! Adjusting timer...
		PotionTimer:Set[${PotionCooldown}]
	}
}

objectdef TimerObject
{
	variable uint EndTime

	method Set(uint Milliseconds)
	{
		EndTime:Set[${Milliseconds}+${Script.RunningTime}]
	}

	member:uint TimeLeft()
	{
		if ${Script.RunningTime}>=${EndTime}
			return 0
		return ${Math.Calc[${EndTime}-${Script.RunningTime}]}
	}
}

function main()
{
	Event[EQ2_onIncomingText]:AttachAtom[EQ2_onIncomingText]

	call LoadSettings

	echo Smart Auto-Potion script started
	echo Using priority list from settings

	; Initial potion use
	call UseNextPotion

	while 1
	{
		if ${PotionTimer.TimeLeft} == 0
		{
			call UseNextPotion
		}

		; Display countdown
		if ${Math.Calc[${PotionTimer.TimeLeft}/1000]%60} == 0
		{
			echo Next potion in ${Math.Calc[${PotionTimer.TimeLeft}/1000/60]} minutes
		}

		wait 10
	}
}

function UseNextPotion()
{
	wait !${Me.InCombat} && !${Me.CastingSpell}

	echo Checking for available potions...
	PotionPriority:Set[1]

	while ${PotionPriority} <= ${UIElement[PotionListBox@Settings].Items}
	{
		CurrentPotion:Set[${UIElement[PotionListBox@Settings].OrderedItem[${PotionPriority}]}]

		if ${Me.Inventory[${CurrentPotion}](exists)} && ${Me.Inventory[${CurrentPotion}].IsReady}
		{
			echo Using priority ${PotionPriority}: ${CurrentPotion}

			; Use the potion
			Me.Inventory[exactitem,${CurrentPotion}]:Use
			wait 10

			; Verify casting started
			if ${Me.CastingSpell}
			{
				; Get spell name being cast
				PotionSpellName:Set[${EQ2DataSourceContainer[GameData].GetDynamicData[Spells.Casting].ShortLabel}]
				echo Casting: ${PotionSpellName}

				; Wait for cast
				wait 110

				; Get buff duration
				PotionDuration:Set[${Me.Effect[${PotionSpellName}].Duration}]
				wait 30
				PotionDuration:Set[${Me.Effect[${PotionSpellName}].Duration}]

				; Set timer (duration + 90 second buffer)
				NextUseTime:Set[${Math.Calc[(${PotionDuration}*1000)+90000]}]
				PotionTimer:Set[${NextUseTime}]

				echo Potion active for ${PotionDuration} seconds
				echo Next use in ${Math.Calc[${NextUseTime}/1000/60].Centi} minutes

				return
			}
			else
			{
				echo Failed to use ${CurrentPotion}
			}
		}

		PotionPriority:Inc
	}

	echo No potions available. Waiting 10 minutes...
	PotionTimer:Set[${PotionCooldown}]
}

function LoadSettings()
{
	LavishSettings:AddSet[AutoPotion]
	LavishSettings[AutoPotion]:Clear
	LavishSettings[AutoPotion]:Import["${LavishScript.HomeDirectory}/Scripts/AutoPotion/${EQ2.ServerName}_${Me.Name}_Settings.xml"]
	_ref:Set[${LavishSettings.FindSet[AutoPotion]}]

	echo Loaded settings for ${Me.Name} on ${EQ2.ServerName}
}

function SaveSettings()
{
	LavishSettings[AutoPotion]:Export["${LavishScript.HomeDirectory}/Scripts/AutoPotion/${EQ2.ServerName}_${Me.Name}_Settings.xml"]
	echo Settings saved
}

function atexit()
{
	call SaveSettings
	echo Auto-Potion script stopped
}
```

### Example 2: Treasure Chest Hunter with Return-to-Home

```lavishscript
; Complete chest hunter with position tracking and coin counting

variable point3f HomeLocation
variable float HomeHeading
variable int MaxRoamDistance=20

variable int StartingCoins
variable int CurrentCoins
variable int GainedCoins
variable int DisplayCopper
variable int DisplaySilver
variable int DisplayGold
variable int DisplayPlatinum

variable int ChestsLooted=0
variable int StartTime
variable int DisplaySeconds
variable int DisplayMinutes
variable int DisplayHours

function main()
{
	; Save starting position
	HomeLocation:Set[${Me.Loc}]
	HomeHeading:Set[${Me.Heading}]

	; Initialize coin tracking
	StartingCoins:Set[${Math.Calc[(${Me.Platinum}*1000000)+(${Me.Gold}*10000)+(${Me.Silver}*100)+${Me.Copper}]}]

	echo ========================================
	echo Treasure Chest Hunter Started
	echo Home Position: ${HomeLocation.X.Centi}, ${HomeLocation.Y.Centi}, ${HomeLocation.Z.Centi}
	echo Max Roam: ${MaxRoamDistance}m
	echo ========================================

	while 1
	{
		call UpdateRunTime

		; Check for chests
		if ${Actor[chest,"Exquisite Chest"].Name(exists)} && ${Actor[chest,"Exquisite Chest"].Distance} <= ${MaxRoamDistance}
		{
			call LootChest "Exquisite Chest"
			call ReturnHome
			call DisplayProgress
		}
		elseif ${Actor[chest,"Ornate Chest"].Name(exists)} && ${Actor[chest,"Ornate Chest"].Distance} <= ${MaxRoamDistance}
		{
			call LootChest "Ornate Chest"
			call ReturnHome
			call DisplayProgress
		}
		elseif ${Actor[chest,"Treasure Chest"].Name(exists)} && ${Actor[chest,"Treasure Chest"].Distance} <= ${MaxRoamDistance}
		{
			call LootChest "Treasure Chest"
			call ReturnHome
			call DisplayProgress
		}

		wait 10
	}
}

function LootChest(string ChestType)
{
	echo ${ChestType} found at ${Actor[chest,${ChestType}].Distance.Centi}m

	face "${ChestType}"
	wait 10

	; Move to chest
	if ${Actor[chest,${ChestType}].Distance} > 2
	{
		echo Moving to ${ChestType}...

		while ${Actor[chest,${ChestType}].Distance} > 2 && ${Actor[chest,${ChestType}].Name(exists)}
		{
			eq2press -hold w
			face "${ChestType}"
		}
		eq2press -release w
	}

	; Open it
	echo Opening ${ChestType}...
	EQ2execute "/apply_verb ${Actor[${ChestType}].ID} Open"
	wait 20

	ChestsLooted:Inc
	echo ${ChestType} looted! (Total: ${ChestsLooted})
}

function ReturnHome()
{
	if ${Math.Distance[${Me.Loc},${HomeLocation}]} > 3
	{
		echo Returning to home position...

		while ${Math.Distance[${Me.Loc},${HomeLocation}]} > 3
		{
			face ${HomeLocation.X} ${HomeLocation.Z}
			eq2press -hold w
		}
		eq2press -release w
	}

	face ${HomeHeading}
}

function DisplayProgress()
{
	; Calculate coin gain
	CurrentCoins:Set[${Math.Calc[(${Me.Platinum}*1000000)+(${Me.Gold}*10000)+(${Me.Silver}*100)+${Me.Copper}]}]
	GainedCoins:Set[${Math.Calc[${CurrentCoins}-${StartingCoins}]}]

	DisplayCopper:Set[${Math.Calc[${GainedCoins}%100]}]
	DisplaySilver:Set[${Math.Calc[${GainedCoins}/100%100]}]
	DisplayGold:Set[${Math.Calc[${GainedCoins}/10000%100]}]
	DisplayPlatinum:Set[${Math.Calc[${GainedCoins}/10000\\100]}]

	echo ========================================
	echo Chests Looted: ${ChestsLooted}
	echo Coins Gained: ${DisplayPlatinum}p ${DisplayGold}g ${DisplaySilver}s ${DisplayCopper}c
	echo Running Time: ${DisplayHours.LeadingZeroes[2]}:${DisplayMinutes.LeadingZeroes[2]}:${DisplaySeconds.LeadingZeroes[2]}
	echo ========================================
}

function UpdateRunTime()
{
	StartTime:Set[${Math.Calc64[${Script.RunningTime}/1000]}]
	DisplaySeconds:Set[${Math.Calc64[${StartTime}%60]}]
	DisplayMinutes:Set[${Math.Calc64[${StartTime}/60%60]}]
	DisplayHours:Set[${Math.Calc64[${StartTime}/60\\60]}]
}

function atexit()
{
	echo ========================================
	echo Treasure Chest Hunter Stopped
	echo ========================================
	call DisplayProgress
}
```

### Example 3: Multi-Type XP Tracker

```lavishscript
; Comprehensive XP tracker for ADV, AA, and TS

variable int StartADVLevel
variable float StartADVXP
variable int StartAA
variable float StartAAXP
variable int StartTS
variable float StartTSXP

variable int CurrentADVLevel
variable float CurrentADVXP
variable int CurrentAA
variable float CurrentAAXP
variable int CurrentTS
variable float CurrentTSXP

variable float TotalADVXP
variable float TotalAAXP
variable float TotalTSXP

variable float TimeRunningHours

variable int DisplayADVLevels
variable float DisplayADVPercent
variable int DisplayAAPoints
variable float DisplayAAPercent
variable int DisplayTSLevels
variable float DisplayTSPercent

variable float ADVPerHour
variable float AAPerHour
variable float TSPerHour

function main()
{
	call InitializeTracking

	echo ========================================
	echo Multi-Type XP Tracker Started
	echo ========================================
	echo ADV: Level ${StartADVLevel} (${StartADVXP.Centi}%)
	echo AA: ${StartAA} points (${StartAAXP.Centi}%)
	echo TS: Level ${StartTS} (${StartTSXP.Centi}%)
	echo ========================================

	while 1
	{
		call UpdateADVTracking
		call UpdateAATracking
		call UpdateTSTracking
		call DisplayProgress

		wait 100  ; Update every 10 seconds
	}
}

function InitializeTracking()
{
	StartADVLevel:Set[${Me.Level}]
	StartADVXP:Set[${Me.Exp}]
	StartAA:Set[${Me.TotalEarnedAPs}]
	StartAAXP:Set[${Me.APExp}]
	StartTS:Set[${Me.TSLevel}]
	StartTSXP:Set[${Me.TSExp}]
}

function UpdateADVTracking()
{
	CurrentADVLevel:Set[${Me.Level}]
	CurrentADVXP:Set[${Me.Exp}]

	TimeRunningHours:Set[${Math.Calc[(((${Script.RunningTime}/1000)/60/60))].Milli}]

	if ${StartADVLevel} < ${CurrentADVLevel}
	{
		variable int GainedLevels
		GainedLevels:Set[${Math.Calc[${CurrentADVLevel}-${StartADVLevel}]}]
		TotalADVXP:Set[${Math.Calc[(${GainedLevels}*100)-100+${CurrentADVXP}+(100-${StartADVXP})]}]
	}
	else
	{
		TotalADVXP:Set[${Math.Calc[${CurrentADVXP}-${StartADVXP}]}]
	}

	DisplayADVLevels:Set[${Math.Calc[${TotalADVXP}/100].Int}]
	DisplayADVPercent:Set[${Math.Calc[${TotalADVXP}%100]}]

	if ${TimeRunningHours} > 0
		ADVPerHour:Set[${Math.Calc[${TotalADVXP}/${TimeRunningHours}]}]
}

function UpdateAATracking()
{
	CurrentAA:Set[${Me.TotalEarnedAPs}]
	CurrentAAXP:Set[${Me.APExp}]

	if ${StartAA} < ${CurrentAA}
	{
		variable int GainedAA
		GainedAA:Set[${Math.Calc[${CurrentAA}-${StartAA}]}]
		TotalAAXP:Set[${Math.Calc[(${GainedAA}*100)-100+${CurrentAAXP}+(100-${StartAAXP})]}]
	}
	else
	{
		TotalAAXP:Set[${Math.Calc[${CurrentAAXP}-${StartAAXP}]}]
	}

	DisplayAAPoints:Set[${Math.Calc[${TotalAAXP}/100].Int}]
	DisplayAAPercent:Set[${Math.Calc[${TotalAAXP}%100]}]

	if ${TimeRunningHours} > 0
		AAPerHour:Set[${Math.Calc[${TotalAAXP}/${TimeRunningHours}]}]
}

function UpdateTSTracking()
{
	CurrentTS:Set[${Me.TSLevel}]
	CurrentTSXP:Set[${Me.TSExp}]

	if ${StartTS} < ${CurrentTS}
	{
		variable int GainedTS
		GainedTS:Set[${Math.Calc[${CurrentTS}-${StartTS}]}]
		TotalTSXP:Set[${Math.Calc[(${GainedTS}*100)-100+${CurrentTSXP}+(100-${StartTSXP})]}]
	}
	else
	{
		TotalTSXP:Set[${Math.Calc[${CurrentTSXP}-${StartTSXP}]}]
	}

	DisplayTSLevels:Set[${Math.Calc[${TotalTSXP}/100].Int}]
	DisplayTSPercent:Set[${Math.Calc[${TotalTSXP}%100]}]

	if ${TimeRunningHours} > 0
		TSPerHour:Set[${Math.Calc[${TotalTSXP}/${TimeRunningHours}]}]
}

function DisplayProgress()
{
	variable int StartTime
	variable int DisplaySeconds
	variable int DisplayMinutes
	variable int DisplayHours

	StartTime:Set[${Math.Calc64[${Script.RunningTime}/1000]}]
	DisplaySeconds:Set[${Math.Calc64[${StartTime}%60]}]
	DisplayMinutes:Set[${Math.Calc64[${StartTime}/60%60]}]
	DisplayHours:Set[${Math.Calc64[${StartTime}/60\\60]}]

	echo ========================================
	echo XP Progress Report
	echo Time: ${DisplayHours.LeadingZeroes[2]}:${DisplayMinutes.LeadingZeroes[2]}:${DisplaySeconds.LeadingZeroes[2]}
	echo ========================================
	echo ADV: ${DisplayADVLevels} levels, ${DisplayADVPercent.Centi}%
	if ${TimeRunningHours} > 0
		echo     Rate: ${ADVPerHour.Centi}% per hour
	echo AA:  ${DisplayAAPoints} points, ${DisplayAAPercent.Centi}%
	if ${TimeRunningHours} > 0
		echo     Rate: ${AAPerHour.Centi}% per hour
	echo TS:  ${DisplayTSLevels} levels, ${DisplayTSPercent.Centi}%
	if ${TimeRunningHours} > 0
		echo     Rate: ${TSPerHour.Centi}% per hour
	echo ========================================
}

function atexit()
{
	echo XP Tracker stopped
	call DisplayProgress
}
```

---

## Summary

The EQ2BJCommon scripts demonstrate 18 essential utility patterns:

1. **Custom Timer Objects** - Flexible time tracking with `TimerObject`
2. **RunningTime Calculations** - Convert milliseconds to HH:MM:SS
3. **Currency Management** - Track platinum/gold/silver/copper as single value
4. **Position Tracking** - Save and return to home location
5. **Character-Specific Config** - Per-character, per-server settings
6. **Dynamic File Loading** - Enumerate and load user-created presets
7. **Prioritized Item Lists** - Auto-fallback to next available item
8. **Event Text Parsing** - React to game messages automatically
9. **EQ2DataSourceContainer** - Access dynamic game data for buffs/casts
10. **Randomized Movement** - Natural-looking, class-based positioning
11. **Item Usage Verification** - Confirm items actually worked
12. **ApplyVerb Interaction** - Open chests, loot corpses, inspect objects
13. **ReplyDialog Usage** - Navigate NPC conversations programmatically
14. **Audio Notifications** - Alert users with sound files
15. **Input Validation** - Extensive checks with user-friendly errors
16. **Progressive XP Tracking** - Handle level-ups across ADV/AA/TS
17. **Script Existence Checking** - Prevent duplicates, manage dependencies
18. **ExecuteQueued Loops** - Handle UI and inter-script commands

These patterns provide the foundation for building robust, user-friendly utility scripts for EverQuest 2.

---

**Next Steps:**
- Study [05_Patterns_And_Best_Practices.md](05_Patterns_And_Best_Practices.md) for EQ2Bot patterns
- Review [14_Advanced_Scripting_Patterns.md](14_Advanced_Scripting_Patterns.md) for production patterns
- Check [06_Working_Examples.md](06_Working_Examples.md) for more code samples
- Reference [03_API_Reference.md](03_API_Reference.md) for API details

---

*ISXEQ2 Scripting Guide - Utility Patterns from EQ2BJCommon*
*Version: 2.6*
*Last Updated: 2025-10-23*
*Source: https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2BJCommon*
