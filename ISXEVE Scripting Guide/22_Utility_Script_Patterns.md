# Utility Script Patterns

**Purpose:** Common utility patterns for robust ISXEVE script development
**Audience:** Intermediate scripters building production-quality automation
**Prerequisites:** Understanding of LavishScript fundamentals and basic ISXEVE scripting

---

## Table of Contents

1. [Overview](#overview)
2. [Custom Timer Objects](#custom-timer-objects)
3. [Script RunningTime Calculations](#script-runningtime-calculations)
4. [ISK and Wallet Management](#isk-and-wallet-management)
5. [Position Tracking and Bookmarks](#position-tracking-and-bookmarks)
6. [Character-Specific Configuration](#character-specific-configuration)
7. [Dynamic File Loading](#dynamic-file-loading)
8. [Prioritized Target Lists](#prioritized-target-lists)
9. [Event-Based Chat Parsing](#event-based-chat-parsing)
10. [Randomized Movement](#randomized-movement)
11. [Module Usage Verification](#module-usage-verification)
12. [Audio Notifications](#audio-notifications)
13. [Input Validation Patterns](#input-validation-patterns)
14. [Progressive Tracking (ISK/XP/Loot)](#progressive-tracking-isk-xp-loot)
15. [Script Existence Checking](#script-existence-checking)
16. [ExecuteQueued Loop Patterns](#executequeued-loop-patterns)
17. [Complete Working Examples](#complete-working-examples)

---

## Overview

These utility patterns are essential for building robust, user-friendly ISXEVE scripts. They demonstrate best practices for:

- Persistent configuration with LavishSettings
- Extensive input validation and user feedback
- Timer-based operations with precise tracking
- Character-specific settings support
- Error handling and graceful degradation
- UI integration and user notifications

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

	member:bool Expired()
	{
		return ${This.TimeLeft} == 0
	}
}
```

### Usage Example - Module Activation Cooldown

```lavishscript
variable(global) TimerObject NextModuleTimer
variable int RandomDelay

function main()
{
	; Set a random delay between 30-45 seconds (30000-45000 ms)
	RandomDelay:Set[${Math.Rand[15000]:Inc[30000]}]
	NextModuleTimer:Set[${RandomDelay}]

	echo Next module activation in ${Math.Calc[${NextModuleTimer.TimeLeft}/1000]} seconds

	while 1
	{
		if ${NextModuleTimer.Expired}
		{
			echo Timer expired! Activating module...
			call ActivateModule

			; Reset timer for next use
			RandomDelay:Set[${Math.Rand[15000]:Inc[30000]}]
			NextModuleTimer:Set[${RandomDelay}]
		}

		; Display countdown
		if ${Math.Calc[${NextModuleTimer.TimeLeft}/1000]} > 0
			echo Time until next module: ${Math.Calc[${NextModuleTimer.TimeLeft}/1000]} seconds

		wait 1000
	}
}
```

### Benefits

- **Non-blocking**: Check timer status without waiting
- **Resettable**: Restart timer dynamically
- **Multiple timers**: Track different operations independently
- **Precise**: Based on `${Script.RunningTime}` for accuracy

---

## Script RunningTime Calculations

Converting `${Script.RunningTime}` (milliseconds) into human-readable time formats is essential for displaying elapsed time, countdowns, and statistics.

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
	DisplayHours:Set[${Math.Calc64[${StartTime}/60/60]}]

	; Display formatted time
	echo Script running for: ${DisplayHours.LeadingZeroes[2]}:${DisplayMinutes.LeadingZeroes[2]}:${DisplaySeconds.LeadingZeroes[2]}
}
```

### ISK Per Hour Calculator

```lavishscript
variable int64 StartingISK
variable int64 CurrentISK
variable int64 ISKGained

function CalculateISKPerHour()
{
	variable int RunHours
	variable int RunMinutes
	variable int64 ISKPerHour

	; Get current ISK
	CurrentISK:Set[${Me.Wallet}]
	ISKGained:Set[${Math.Calc64[${CurrentISK}-${StartingISK}]}]

	; Calculate runtime in hours
	RunHours:Set[${Math.Calc64[${Script.RunningTime}/1000/60/60]}]
	RunMinutes:Set[${Math.Calc64[${Script.RunningTime}/1000/60%60]}]

	; Calculate ISK/hour
	if ${RunHours} > 0
	{
		ISKPerHour:Set[${Math.Calc64[${ISKGained}/${RunHours}]}]
		echo ISK/Hour: ${ISKPerHour.Comma} ISK
	}

	echo Total ISK Gained: ${ISKGained.Comma} ISK
	echo Runtime: ${RunHours}h ${RunMinutes}m
}
```

---

## ISK and Wallet Management

### Tracking ISK Gains

```lavishscript
variable int64 SessionStartISK
variable int64 TotalISKGained

function InitializeISKTracking()
{
	SessionStartISK:Set[${Me.Wallet}]
	TotalISKGained:Set[0]
	echo Session started with ${SessionStartISK.Comma} ISK
}

function UpdateISKGained()
{
	TotalISKGained:Set[${Math.Calc64[${Me.Wallet}-${SessionStartISK}]}]
	echo ISK Gained This Session: ${TotalISKGained.Comma} ISK
}
```

### ISK Threshold Alerts

```lavishscript
variable int64 ISKThreshold = 100000000

function CheckISKThreshold()
{
	if ${Me.Wallet} >= ${ISKThreshold}
	{
		echo ALERT: Wallet has reached ${Me.Wallet.Comma} ISK!
		call AudioAlert
	}
}
```

---

## Position Tracking and Bookmarks

### Saving Current Position

```lavishscript
variable float SavedX
variable float SavedY
variable float SavedZ

function SaveCurrentPosition()
{
	SavedX:Set[${Me.ToEntity.X}]
	SavedY:Set[${Me.ToEntity.Y}]
	SavedZ:Set[${Me.ToEntity.Z}]

	echo Position saved: ${SavedX}, ${SavedY}, ${SavedZ}
}

function ReturnToSavedPosition()
{
	if !${SavedX(exists)}
	{
		echo ERROR: No saved position
		return
	}

	echo Returning to saved position...
	EVE:Execute[CmdWarpToLocation, ${SavedX}, ${SavedY}, ${SavedZ}]
}
```

### Distance Calculations

```lavishscript
function CalculateDistance(float X, float Y, float Z)
{
	variable float DeltaX
	variable float DeltaY
	variable float DeltaZ
	variable float Distance

	DeltaX:Set[${Math.Calc[${Me.ToEntity.X}-${X}]}]
	DeltaY:Set[${Math.Calc[${Me.ToEntity.Y}-${Y}]}]
	DeltaZ:Set[${Math.Calc[${Me.ToEntity.Z}-${Z}]}]

	Distance:Set[${Math.Sqrt[${Math.Calc[${DeltaX}*${DeltaX}+${DeltaY}*${DeltaY}+${DeltaZ}*${DeltaZ}]}]}]

	return ${Distance}
}
```

---

## Character-Specific Configuration

### Pattern Using LavishSettings

```lavishscript
variable string SettingsFile = "${Me.Name}_Settings.xml"

function LoadCharacterSettings()
{
	if !${LavishSettings[MyBot](exists)}
	{
		LavishSettings:AddSet[MyBot]
		LavishSettings[MyBot]:Import[${SettingsFile}]
	}

	; Load character-specific values
	if ${LavishSettings[MyBot].FindSet[${Me.Name}](exists)}
	{
		echo Loading settings for ${Me.Name}
		; Load settings here
	}
	else
	{
		echo No settings found for ${Me.Name}, using defaults
		call CreateDefaultSettings
	}
}

function CreateDefaultSettings()
{
	LavishSettings[MyBot]:AddSet[${Me.Name}]
	LavishSettings[MyBot].FindSet[${Me.Name}]:AddSetting[AutoTarget, TRUE]
	LavishSettings[MyBot].FindSet[${Me.Name}]:AddSetting[PreferredRange, 15000]

	; Save to file
	LavishSettings[MyBot]:Export[${SettingsFile}]
	echo Default settings created for ${Me.Name}
}
```

---

## Dynamic File Loading

### Loading Include Files by Character

```lavishscript
function LoadCharacterProfile()
{
	variable string ProfileFile = "${Me.Name}_Profile.iss"

	if !${System.FileExists[${ProfileFile}]}
	{
		echo WARNING: Profile file not found: ${ProfileFile}
		echo Using default profile
		ProfileFile:Set["Default_Profile.iss"]
	}

	echo Loading profile: ${ProfileFile}
	#include ${ProfileFile}
}
```

---

## Prioritized Target Lists

### Priority-Based Target Selection

```lavishscript
objectdef TargetPriority
{
	variable string Name
	variable int Priority
	variable int GroupID

	method Initialize(string _Name, int _Priority, int _GroupID)
	{
		Name:Set[${_Name}]
		Priority:Set[${_Priority}]
		GroupID:Set[${_GroupID}]
	}
}

variable collection:TargetPriority PriorityTargets

function InitializePriorityTargets()
{
	; Priority 1 (highest)
	PriorityTargets:Set["Serpentis Commander", TargetPriority["Serpentis Commander", 1, 100]]

	; Priority 2
	PriorityTargets:Set["Serpentis Battleship", TargetPriority["Serpentis Battleship", 2, 100]]

	; Priority 3
	PriorityTargets:Set["Serpentis Cruiser", TargetPriority["Serpentis Cruiser", 3, 100]]
}

function FindHighestPriorityTarget()
{
	variable int HighestPriority = 999
	variable int64 BestTargetID = 0
	variable iterator Target

	EVE:QueryEntities[Target, "GroupID = 100"]

	if ${Target:First(exists)}
	{
		do
		{
			if ${PriorityTargets.Element[${Target.Value.Name}](exists)}
			{
				if ${PriorityTargets.Element[${Target.Value.Name}].Priority} < ${HighestPriority}
				{
					HighestPriority:Set[${PriorityTargets.Element[${Target.Value.Name}].Priority}]
					BestTargetID:Set[${Target.Value.ID}]
				}
			}
		}
		while ${Target:Next(exists)}
	}

	if ${BestTargetID} > 0
	{
		echo Targeting priority target: ${Entity[${BestTargetID}].Name}
		Entity[${BestTargetID}]:LockTarget
	}
}
```

---

## Event-Based Chat Parsing

### Monitoring Local Chat for Threats

```lavishscript
variable bool HostileDetected = FALSE

function main()
{
	Event[OnIncomingText]:AttachAtom[OnChatMessage]

	while 1
	{
		if ${HostileDetected}
		{
			echo HOSTILE DETECTED! Taking evasive action!
			call EmergencyWarp
		}

		wait 10
	}
}

atom OnChatMessage(string Message, string Channel)
{
	if ${Channel.Equal["local"]}
	{
		; Check for hostile keywords
		if ${Message.Find["gf"](exists)} || ${Message.Find["o/"](exists)}
		{
			echo Possible hostile communication detected in local
			HostileDetected:Set[TRUE]
		}
	}
}
```

---

## Randomized Movement

### Random Orbit Distance

```lavishscript
function RandomOrbit(int64 TargetID)
{
	variable int MinDistance = 10000
	variable int MaxDistance = 20000
	variable int RandomDistance

	; Generate random distance
	RandomDistance:Set[${Math.Rand[${Math.Calc[${MaxDistance}-${MinDistance}]}]:Inc[${MinDistance}]}]

	echo Orbiting at ${RandomDistance}m
	Entity[${TargetID}]:Orbit[${RandomDistance}]
}
```

### Random Wait Times

```lavishscript
function RandomWait(int MinMS, int MaxMS)
{
	variable int WaitTime

	WaitTime:Set[${Math.Rand[${Math.Calc[${MaxMS}-${MinMS}]}]:Inc[${MinMS}]}]

	echo Waiting for ${WaitTime}ms
	wait ${WaitTime}
}
```

---

## Module Usage Verification

### Verifying Module Activation

```lavishscript
function ActivateModuleWithVerification(int Slot)
{
	variable int Attempts = 0
	variable int MaxAttempts = 3

	while ${Attempts} < ${MaxAttempts}
	{
		if !${MyShip.Module[${Slot}].IsActive}
		{
			echo Activating module in slot ${Slot}
			MyShip.Module[${Slot}]:Click

			wait 20

			; Verify activation
			if ${MyShip.Module[${Slot}].IsActive}
			{
				echo Module activated successfully
				return TRUE
			}
			else
			{
				Attempts:Inc
				echo Activation failed, attempt ${Attempts}/${MaxAttempts}
			}
		}
		else
		{
			echo Module already active
			return TRUE
		}
	}

	echo ERROR: Failed to activate module after ${MaxAttempts} attempts
	return FALSE
}
```

---

## Audio Notifications

### Playing Alert Sounds

```lavishscript
function AudioAlert()
{
	PlaySound "${LavishScript.HomeDirectory}/Sounds/alert.wav"
}

function VoiceAlert(string Message)
{
	speak "${Message}"
}

function AlertHostileDetected()
{
	call AudioAlert
	call VoiceAlert "Hostile detected in local"
}
```

---

## Input Validation Patterns

### Validating User Input

```lavishscript
function ValidatePositiveInteger(string Input)
{
	variable int Value

	if !${Input.IsNumber}
	{
		echo ERROR: Input must be a number
		return FALSE
	}

	Value:Set[${Input}]

	if ${Value} <= 0
	{
		echo ERROR: Value must be positive
		return FALSE
	}

	return TRUE
}

function ValidateISKAmount(string Input)
{
	variable int64 Amount

	if !${Input.IsNumber}
	{
		echo ERROR: ISK amount must be numeric
		return FALSE
	}

	Amount:Set[${Input}]

	if ${Amount} < 0
	{
		echo ERROR: ISK amount cannot be negative
		return FALSE
	}

	if ${Amount} > ${Me.Wallet}
	{
		echo ERROR: Insufficient ISK (${Amount.Comma} requested, ${Me.Wallet.Comma} available)
		return FALSE
	}

	return TRUE
}
```

---

## Progressive Tracking (ISK/XP/Loot)

### Loot Value Tracking

```lavishscript
variable int64 TotalLootValue = 0
variable int ItemsLooted = 0

function TrackLootedItem(string ItemName, int64 EstimatedValue)
{
	TotalLootValue:Set[${Math.Calc64[${TotalLootValue}+${EstimatedValue}]}]
	ItemsLooted:Inc

	echo Looted: ${ItemName} (${EstimatedValue.Comma} ISK)
	echo Session Total: ${ItemsLooted} items, ${TotalLootValue.Comma} ISK
}

function DisplayLootSummary()
{
	variable int RunHours = ${Math.Calc[${Script.RunningTime}/1000/60/60]}
	variable int64 ISKPerHour

	if ${RunHours} > 0
		ISKPerHour:Set[${Math.Calc64[${TotalLootValue}/${RunHours}]}]

	echo ===== LOOT SUMMARY =====
	echo Items Looted: ${ItemsLooted}
	echo Total Value: ${TotalLootValue.Comma} ISK
	echo Runtime: ${RunHours} hours
	echo ISK/Hour: ${ISKPerHour.Comma} ISK
}
```

---

## Script Existence Checking

### Checking if Another Script is Running

```lavishscript
function IsScriptRunning(string ScriptName)
{
	if ${Script[${ScriptName}](exists)}
	{
		echo Script ${ScriptName} is already running
		return TRUE
	}

	return FALSE
}

function LaunchScriptIfNotRunning(string ScriptPath, string ScriptName)
{
	if !${Script[${ScriptName}](exists)}
	{
		echo Launching ${ScriptName}
		runscript "${ScriptPath}"
		wait 10
		return TRUE
	}
	else
	{
		echo ${ScriptName} is already running
		return FALSE
	}
}
```

---

## ExecuteQueued Loop Patterns

### UI-Safe Operations with ExecuteQueued

```lavishscript
function main()
{
	while 1
	{
		; Queue UI operations
		EVE:Execute[OpenCargoHold]
		wait 5

		; Process cargo with queued execution
		call ProcessCargoQueued

		wait 10
	}
}

function ProcessCargoQueued()
{
	variable iterator CargoItem

	MyShip:GetCargo[CargoItem]

	if ${CargoItem:First(exists)}
	{
		do
		{
			echo Processing: ${CargoItem.Value.Name}
			; Queued operations here
			wait 2
		}
		while ${CargoItem:Next(exists)}
	}
}
```

---

## Complete Working Examples

### Example 1: Mining Session Tracker

```lavishscript
variable int64 SessionStartISK
variable int OreUnits = 0
variable TimerObject MiningTimer

function main()
{
	; Initialize
	SessionStartISK:Set[${Me.Wallet}]
	MiningTimer:Set[3600000]  ; 1 hour

	echo Mining session started
	echo Starting ISK: ${SessionStartISK.Comma}

	while !${MiningTimer.Expired}
	{
		; Mining logic here
		call MineCycle

		; Display stats every 60 seconds
		if ${Math.Calc[${Script.RunningTime}%60000]} < 100
			call DisplayStats

		wait 10
	}

	echo Session complete!
	call DisplayFinalStats
}

function MineCycle()
{
	; Simulate mining
	OreUnits:Inc[100]
}

function DisplayStats()
{
	variable int64 ISKGained = ${Math.Calc64[${Me.Wallet}-${SessionStartISK}]}
	variable int MinutesLeft = ${Math.Calc[${MiningTimer.TimeLeft}/1000/60]}

	echo Ore Mined: ${OreUnits} units
	echo ISK Gained: ${ISKGained.Comma}
	echo Time Remaining: ${MinutesLeft} minutes
}

function DisplayFinalStats()
{
	variable int64 ISKGained = ${Math.Calc64[${Me.Wallet}-${SessionStartISK}]}
	variable int RunMinutes = ${Math.Calc[${Script.RunningTime}/1000/60]}

	echo ===== MINING SESSION COMPLETE =====
	echo Total Ore Mined: ${OreUnits} units
	echo Total ISK Gained: ${ISKGained.Comma}
	echo Total Runtime: ${RunMinutes} minutes
}
```

---

## Summary

These utility patterns form the foundation of robust ISXEVE scripts:

1. **Custom Timers** - Flexible, non-blocking time tracking
2. **RunningTime Calculations** - Display runtime and calculate rates
3. **ISK Tracking** - Monitor wallet and calculate profit
4. **Position Tracking** - Save and return to locations
5. **Character-Specific Config** - Per-character settings with LavishSettings
6. **Dynamic Loading** - Load files based on character/context
7. **Priority Lists** - Prioritize targets, items, or actions
8. **Event Parsing** - React to chat and game events
9. **Randomization** - Vary timing and movement
10. **Verification** - Confirm actions completed successfully
11. **Audio Alerts** - Notify user of important events
12. **Input Validation** - Validate user input and settings
13. **Progressive Tracking** - Track cumulative stats over time
14. **Script Management** - Check and launch other scripts
15. **ExecuteQueued** - UI-safe queued operations

Use these patterns as building blocks for your ISXEVE automation projects.

---

*Last Updated: 2025-10-26*
*Part of ISXEVE Scripting Guide*
