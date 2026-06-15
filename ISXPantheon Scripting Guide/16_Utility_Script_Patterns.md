# Utility Script Patterns

**Reusable, platform-agnostic LavishScript utility patterns for building robust, user-friendly tools.**

The patterns in this guide are general-purpose: custom timers, time/rate calculations, dynamic file loading, prioritized lists, text parsing, audio notifications, input validation, script management, and queued-command handling. Patterns that depend on live game data (currency, position, item/object interaction, NPC dialog, experience tracking) are marked as planned, since that surface is not yet implemented in ISXPantheon.

---

## Table of Contents

1. [Overview](#overview)
2. [Custom Timer Objects](#custom-timer-objects)
3. [Script RunningTime Calculations](#script-runningtime-calculations)
4. [Currency Management (planned)](#currency-management-planned)
5. [Position Tracking and Return-to-Home (planned)](#position-tracking-and-return-to-home-planned)
6. [Session-Specific Configuration](#session-specific-configuration)
7. [Dynamic File Loading](#dynamic-file-loading)
8. [Prioritized Lists](#prioritized-lists)
9. [Event-Based Text Parsing](#event-based-text-parsing)
10. [Shared Data Containers (planned)](#shared-data-containers-planned)
11. [Randomized Movement](#randomized-movement)
12. [Item Usage Verification (planned)](#item-usage-verification-planned)
13. [Object Interaction (planned)](#object-interaction-planned)
14. [NPC Conversations (planned)](#npc-conversations-planned)
15. [Audio Notifications](#audio-notifications)
16. [Input Validation Patterns](#input-validation-patterns)
17. [Progressive XP Tracking (planned)](#progressive-xp-tracking-planned)
18. [Script Existence Checking](#script-existence-checking)
19. [ExecuteQueued Loop Patterns](#executequeued-loop-patterns)
20. [Complete Working Examples](#complete-working-examples)

---

## Overview

This guide collects practical utility patterns for building robust, user-friendly LavishScript tools. The infrastructure patterns demonstrated here work today against any InnerSpace extension; the game-data-dependent patterns are flagged as planned.

**Key Characteristics:**
- Heavy use of LavishSettings for persistent configuration
- Extensive input validation and user feedback
- Timer-based operations with precise time tracking
- UI integration with queued commands
- Character-specific settings support
- Error handling and graceful degradation

---

## Custom Timer Objects

Custom timer objects (a `TimerObject` objectdef with `Set`/`TimeLeft` members) provide non-blocking, resettable timing — more flexible than inline `wait` commands. This pattern is ideal for randomized cooldowns and similar delayed-action logic.

**This pattern is documented canonically in [15_Advanced_Scripting_Patterns.md](15_Advanced_Scripting_Patterns.md#reusable-timer-objects)**, which covers the basic `TimerObject`, an `AdvancedTimer` variant with pause/resume, and a full multi-timer pulse system. Refer there for the complete objectdef and usage examples.

**Minimal signature (for recognition):**

```lavishscript
objectdef TimerObject
{
    variable uint EndTime
    method Set(uint Milliseconds)
    member:uint TimeLeft()
}
```

**Typical usage** — a randomized action delay:

```lavishscript
variable(global) TimerObject NextActionTimer
RandomDelay:Set[${Math.Rand[300000]:Inc[600000]}]  ; 10-15 min
NextActionTimer:Set[${RandomDelay}]

if ${NextActionTimer.TimeLeft} == 0
{
    call DoAction
    NextActionTimer:Set[${Math.Rand[300000]:Inc[600000]}]
}
```

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


---

## Currency Management (planned)

> **PLANNED — NOT YET IMPLEMENTED.** Reading the player's on-hand currency requires a local-player datatype that is not yet available, and Pantheon: Rise of the Fallen's currency denomination model is not exposed yet. The currency-tracking pattern below is provisional and WILL change. Do not write production scripts against it.

The general technique — convert all denominations to a single base unit so tracking is a single integer, then format back to a human-readable string for display — is sound and platform-agnostic. The `${ISXPantheon}` top-level object already provides a formatting helper for this once you have an amount:

```lavishscript
; REAL today: format an amount (expressed in the game's base sub-unit) for display
echo ${ISXPantheon.GetCurrencyString[${AmountInBaseUnits}]}
```

A future session-gain tracker would snapshot the player's currency at start, snapshot it again later, and report the delta through `GetCurrencyString`. The snapshot source (the local player's on-hand currency) is the planned piece:

```lavishscript
; PLANNED — the on-hand-currency source is not yet surfaced
variable int StartingAmount
variable int CurrentAmount

; StartingAmount:Set[${...player on-hand currency...}]   ; planned source
; CurrentAmount:Set[${...player on-hand currency...}]     ; planned source
; echo Gained: ${ISXPantheon.GetCurrencyString[${Math.Calc[${CurrentAmount}-${StartingAmount}]}]}
```

---

## Position Tracking and Return-to-Home (planned)

> **PLANNED - NOT YET IMPLEMENTED.** This pattern reads the player's position and heading and drives movement to return to a saved spot. It depends on a local-player datatype (position, heading) and movement/facing surface that are on the ISXPantheon roadmap but are not yet available. The shape below is provisional and WILL change.

The technique is straightforward once the surface exists: snapshot the player's location and heading at start, perform other work, then walk back toward the saved location until within a distance threshold and restore the original heading. The distance math (`Math.Distance`) and the `point3f` storage type are platform-agnostic; the player position/heading reads and the movement inputs are the planned pieces.

```lavishscript
; PLANNED - position/heading source and movement inputs are not yet surfaced
variable float InitialHeading
variable point3f InitialLocation
variable int MaxDistance=4

; InitialHeading:Set[${...player heading...}]     ; planned source
; InitialLocation:Set[${...player location...}]   ; planned source
;
; To return home: while still farther than the threshold, face the saved
; location and hold the move-forward input, then release it on arrival and
; restore the saved heading.
```

## Session-Specific Configuration

Allow the same script to maintain different settings per InnerSpace session by naming the config file with the session name. This keeps each running instance isolated without manual file selection.

> **Note:** For the full LavishSettings API (nested sets, iterators, shared caches, `FindSetting` with defaults, and complete Load/Save bodies), see [07_Advanced_Patterns_And_Examples.md](07_Advanced_Patterns_And_Examples.md#lavishsettings-xml-configuration). This section only covers the per-session filename convention.

### Filename Convention

```lavishscript
variable filepath ConfigFile="${LavishScript.HomeDirectory}/Scripts/MyScript/Session Config/${Session}_Settings.xml"
```

Two conventions worth noting:

- **`${Session}` key** — the InnerSpace session name is unique per running instance, so each session gets its own config file with no collisions.
- **Dedicated `Session Config/` subfolder** — keeps per-session files grouped separately from shared config, presets, or logs.

### Auto-Save on Exit

Pair the per-session filename with an `atexit` handler so settings are always persisted when the script stops — no manual save step required:

```lavishscript
function atexit()
{
	call SaveSettings
}
```

### Benefits

- **Per-session**: Different settings per running instance
- **Collision-free**: Unique session name avoids overwrites
- **Automatic**: No manual file selection needed
- **Organized**: `Session Config/` folder keeps files together

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


---

## Prioritized Lists

Process entries in order of priority, automatically falling through to the next available entry if the preferred one isn't usable. The priority-list traversal (a UI listbox plus `OrderedItem` indexing and recursive fall-through) is platform-agnostic; the "is this entry available / use it" check is abstracted behind two helper functions so you can wire it to whatever availability source you have.

> **NOTE.** The example helpers `IsEntryAvailable` and `UseEntry` below stand in for game-data actions (checking and using an inventory item, etc.). Item/inventory surface is planned but not yet implemented — see Item Usage Verification (planned). The list-traversal control flow itself works today.

### Pattern

```lavishscript
variable int EntryPriority=1

function UseNextAvailableEntry()
{
	; Try entries in priority order
	if ${UIElement[PriorityListBox@Settings].Items} > 0 && ${EntryPriority} <= ${UIElement[PriorityListBox@Settings].Items}
	{
		variable string Entry = "${UIElement[PriorityListBox@Settings].OrderedItem[${EntryPriority}]}"

		; Check if the current priority entry is usable (abstracted helper)
		if ${IsEntryAvailable["${Entry}"]}
		{
			echo Using: ${Entry}
			call UseEntry "${Entry}"
		}
		else
		{
			; Not available, try next priority
			EntryPriority:Inc
			call UseNextAvailableEntry
		}
	}
	else
	{
		echo No entries available from priority list
		EntryPriority:Set[1]  ; Reset for next attempt
	}
}
```

### Recursive Fall-Through Example

```lavishscript
variable int EntryCounter=1

function TryUseEntry()
{
	if ${EntryCounter} > ${UIElement[ItemListBox@UI].Items}
	{
		echo Exhausted all entries in priority list
		EntryCounter:Set[1]
		return
	}

	variable string Entry
	Entry:Set[${UIElement[ItemListBox@UI].OrderedItem[${EntryCounter}]}]

	if ${IsEntryAvailable["${Entry}"]}
	{
		echo Attempting to use: ${Entry}
		call UseEntry "${Entry}"
		wait 5

		; Check if successful (replace with a real success condition)
		if ${LastUseSucceeded}
		{
			echo Successfully used: ${Entry}
			EntryCounter:Set[1]  ; Reset for next cycle
		}
		else
		{
			echo Failed to use ${Entry}, trying next entry
			EntryCounter:Inc
			call TryUseEntry  ; Recursively try next entry
		}
	}
	else
	{
		echo ${Entry} not available, trying next entry
		EntryCounter:Inc
		call TryUseEntry  ; Recursively try next entry
	}
}
```

### Benefits

- **Automatic fallback**: Uses best available entry
- **User control**: Users set their own priority order
- **Resilient**: Doesn't fail if the preferred entry is missing
- **Efficient**: Stops at the first successful entry

---

## Event-Based Text Parsing

Parse incoming text to detect events and automatically adjust script behavior. The event-handler structure (attach an atom, pattern-match the message, react) is platform-agnostic and works against any text event the platform exposes.

> **NOTE.** ISXPantheon does not yet fire game chat/text events; incoming-chat and incoming-text events are planned but not wired. The event names below (`Game_onIncomingText`, `Game_onIncomingChatText`) are provisional placeholders for that future surface. The atom/pattern-match/react technique itself is valid; only the event source is planned. Likewise, sending text back into the game (group/tell replies) is planned game-action surface.

### Pattern

```lavishscript
; PLANNED event source - provisional event name
atom(script) Game_onIncomingText(string Message, int Type)
{
	if ${Message.Find["specific text pattern"](exists)}
	{
		; React to detected message
		call HandleDetectedMessage
	}
}

function main()
{
	; Attach event handler (once the event is wired)
	Event[Game_onIncomingText]:AttachAtom[Game_onIncomingText]

	; Main loop
	while 1
	{
		wait 10
	}
}
```

### Auto-Adjust Timer Example

This shows the general shape: a detected message resets a cooldown timer so the script backs off automatically. The timer logic is fully agnostic; only the event that drives it is planned.

```lavishscript
variable(global) TimerObject CooldownTimer
variable int Cooldown=600000  ; 10 minutes default

atom(script) Game_onIncomingText(string Message, int Type)
{
	; Detect a "cannot act yet" style message and back off
	if ${Message.Find["already active"](exists)}
	{
		echo Action already active - adjusting timer...
		CooldownTimer:Set[${Cooldown}]
		echo Next attempt in ${Math.Calc[${CooldownTimer.TimeLeft}/1000/60]} minutes
	}
}

function main()
{
	Event[Game_onIncomingText]:AttachAtom[Game_onIncomingText]

	while 1
	{
		if ${CooldownTimer.TimeLeft} == 0
		{
			; ... perform the gated action here ...
			CooldownTimer:Set[${Cooldown}]
		}

		wait 10
	}
}
```

### Chat Command Detection Example

A common use is letting other players drive your script through chat commands. The pattern-match and state-toggle logic works today; receiving the chat event and replying into chat are the planned pieces.

```lavishscript
variable bool ScriptPaused=FALSE

; PLANNED event source - provisional signature
atom(script) Game_onIncomingChatText(int ChatType, string Message, string Speaker)
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
		; PLANNED: replying into game chat is not yet supported
		echo Status requested by ${Speaker}: ${ScriptPaused}
	}
}

function main()
{
	Event[Game_onIncomingChatText]:AttachAtom[Game_onIncomingChatText]

	echo Listening for commands: !pause, !resume, !status

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

- **Smart detection**: React to feedback automatically
- **Error recovery**: Adjust behavior when things fail
- **User commands**: Accept commands via chat (once the chat event is wired)
- **Event-driven**: No polling needed

---

## Shared Data Containers (planned)

> **PLANNED — NOT YET IMPLEMENTED.** Accessing dynamic game data that is not exposed through standard top-level objects (for example, the currently-casting action, or a live effect's remaining duration) requires data-container/effect surface that is on the ISXPantheon roadmap but is not available today. There is no equivalent container or effect datatype in the current build. Any API for this is provisional and WILL change; do not write production scripts against it yet.

When this surface ships it is expected to provide a way to read named dynamic-data fields and live effect durations, enabling timers that key off real in-game state instead of fixed delays.

## Randomized Movement

Create natural-looking movement by randomizing headings and durations rather than moving in fixed patterns. The randomization itself is platform-agnostic — `Math.Rand` produces the random heading and the random hold duration:

```lavishscript
; Agnostic randomization helpers
variable int RandomHeading = ${Math.Rand[360]}        ; 0-359 degrees
variable int RandomWaitTime = ${Math.Rand[30]:Inc[20]} ; 20-49 deciseconds
```

> **PLANNED — NOT YET IMPLEMENTED.** Reading the player's position/heading and issuing movement/facing inputs depends on a local-player datatype and movement surface that are on the ISXPantheon roadmap but are not yet available. A full randomized-shuffle routine (face a random heading, hold the move input for a random duration, then return toward a saved location if it drifted too far) cannot be implemented until that surface ships. Do NOT key movement on invented class/archetype lists — Pantheon's class and race sets are not enumerated here.

## Item Usage Verification (planned)

> **PLANNED — NOT YET IMPLEMENTED.** Verifying that using an item actually succeeded — by checking inventory presence, a resulting cast/cooldown state, or a resulting effect — requires item, inventory, and effect surface that is on the ISXPantheon roadmap but is not available in the current build. The verification pattern (act, wait, then confirm a resulting state changed) is sound in general, but every state it would read here is planned. Do not write production scripts against it yet.

## Object Interaction (planned)

> **PLANNED — NOT YET IMPLEMENTED.** Interacting with in-world objects (the EQ2 "ApplyVerb" mechanic — opening chests, clicking doors, using world objects) requires an entity/object datatype with an interaction method that is on the ISXPantheon roadmap but is not available today. There is no object-interaction command or entity verb surface in the current build. Any API for this is provisional and WILL change; do not write production scripts against it yet.

## NPC Conversations (planned)

> **PLANNED — NOT YET IMPLEMENTED.** Automating NPC conversation/dialog (the EQ2 "ReplyDialog" mechanic — selecting conversation options, advancing quest dialog) requires a dialog/conversation datatype that is on the ISXPantheon roadmap but is not available in the current build. Any API for this is provisional and WILL change; do not write production scripts against it yet.

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
		; Check for a critical alert condition (wire to your own state)
		if ${CriticalCondition}
		{
			call PlayCriticalAlert
		}

		; Check for a goal-reached condition (wire to your own state)
		if ${GoalReached}
		{
			call PlayGoalReachedAlert
		}

		wait 10
	}
}

function PlayCriticalAlert()
{
	if !${AlertSoundPlayed}
	{
		echo CRITICAL CONDITION! Playing alert sound...
		play "${LavishScript.HomeDirectory}/Scripts/MyScript/Sounds/chatalarm.wav"
		AlertSoundPlayed:Set[TRUE]

		; Reset flag after cooldown
		wait ${AlertCooldown}
		AlertSoundPlayed:Set[FALSE]
	}
}

function PlayGoalReachedAlert()
{
	echo Goal reached! Playing notification...
	play "${LavishScript.HomeDirectory}/Scripts/MyScript/Sounds/ping.wav"
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
variable int MaxThreshold=1000
variable int TargetValue
variable int CurrentValue=0
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
	; Validate the max-threshold entry
	if ${MaxThreshold} == NULL
	{
		echo ERROR: Please enter the maximum threshold
		StatusDisplay:Set["Please enter the maximum threshold"]
		return FALSE
	}

	; Check if already at the goal
	if ${CurrentValue} == ${MaxThreshold}
	{
		echo ERROR: Already at the maximum - nothing to do
		StatusDisplay:Set["Already at maximum"]
		return FALSE
	}

	; Validate the target value against the max
	if ${TargetValue} > ${MaxThreshold}
	{
		echo ERROR: Target cannot exceed the maximum (${MaxThreshold})
		StatusDisplay:Set["Target too high"]
		return FALSE
	}

	if ${TargetValue} == ${CurrentValue}
	{
		echo ERROR: Target cannot equal the current value
		StatusDisplay:Set["Target cannot equal current value"]
		return FALSE
	}

	if ${TargetValue} < ${CurrentValue}
	{
		echo ERROR: Target cannot be less than the current value
		StatusDisplay:Set["Target cannot be less than current value"]
		return FALSE
	}

	; Validate limit action selection
	if ${LimitAction.Equal["Please choose..."]}
	{
		echo ERROR: Please select what to do when the limit is reached
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


---

## Progressive XP Tracking (planned)

> **PLANNED — NOT YET IMPLEMENTED.** Tracking experience gain over time (current XP, percent-to-level, XP-per-hour, and the EQ2 "AA"/tradeskill variants) requires a local-player datatype exposing experience values that is on the ISXPantheon roadmap but is not available today. The session-rate math (snapshot, delta, per-hour projection) is itself agnostic and reusable — see Script RunningTime Calculations for the per-hour rate technique — but the XP source it would read is planned. Do not write production scripts against it yet.

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

### List Refresh Pattern

The ExecuteQueued mechanic below is fully agnostic: a dedicated script drains queued commands so a UI button can request a list refresh on the script's own thread. The list's data source here is abstracted into `PopulateList`; once a game-data query (e.g. inventory enumeration) ships, that is the only piece to wire in.

> **NOTE.** `PopulateList` stands in for whatever data source fills the list. Inventory/item enumeration is planned, not yet implemented — see Item Usage Verification (planned). The queue/refresh control flow works today.

```lavishscript
function main()
{
	; Dedicated script for handling list-refresh requests
	while 1
	{
		; Wait for and execute queued commands
		ExecuteQueued
		waitframe
	}
}

; Called from UI button via Script:QueueCommand
function RefreshListQueue()
{
	echo Refreshing list...
	call RefreshList
	wait 10
	call RefreshList
	echo List refresh complete
}

function RefreshList()
{
	; Clear existing list
	UIElement[DataListBox@UI]:ClearItems

	; Populate from the data source (abstracted; wire to a real query later)
	call PopulateList
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


---

## Complete Working Examples

> **PLANNED — NOT YET IMPLEMENTED.** The original worked examples here (auto-potion rotation, treasure-chest hunter with return-to-home, multi-type XP tracker) are each built end-to-end on game-data surface — inventory/item use, player position and movement, in-world object interaction, effect durations, and experience values — none of which is available in the current ISXPantheon build. They have been removed rather than shown against an API that does not exist.

Examples that DO work today against the real surface — verifying the extension is ready, reading version/API-version, custom variables, currency formatting, rounding, and `GetURL`/`PostURL` — live in [06_Working_Examples.md](06_Working_Examples.md). The infrastructure patterns earlier in THIS guide (custom timers, time/rate calculations, dynamic file loading, prioritized-list traversal, audio notifications, input validation, script management, and ExecuteQueued handling) are all runnable now and compose into complete tools as the game-data surface comes online.

## Summary

This guide covered 20 utility patterns. The infrastructure patterns work today; the game-data-dependent ones are flagged as planned.

**Runnable today (platform-agnostic):**

1. **Custom Timer Objects** - Flexible time tracking with `TimerObject`
2. **RunningTime Calculations** - Convert milliseconds to HH:MM:SS and per-hour rates
3. **Session-Specific Config** - Per-session settings files
4. **Dynamic File Loading** - Enumerate and load user-created presets
5. **Prioritized Lists** - Auto-fallback to the next available entry
6. **Audio Notifications** - Alert users with sound files
7. **Input Validation** - Extensive checks with user-friendly errors
8. **Script Existence Checking** - Prevent duplicates, manage dependencies
9. **ExecuteQueued Loops** - Handle UI and inter-script commands

**Planned (depend on game-data surface not yet implemented):**

10. **Currency Management** - Session currency tracking (currency model not yet exposed)
11. **Position Tracking** - Save and return to a home location (player position/movement)
12. **Event-Based Text Parsing** - React to game messages (chat/text events not yet wired)
13. **Shared Data Containers** - Access dynamic game data (casts/effects)
14. **Randomized Movement** - Natural-looking movement (player position/movement)
15. **Item Usage Verification** - Confirm an item action succeeded (item/inventory)
16. **Object Interaction** - Interact with in-world objects (entity verbs)
17. **NPC Conversations** - Automate dialog (conversation datatype)
18. **Progressive XP Tracking** - Track experience over time (player XP values)
