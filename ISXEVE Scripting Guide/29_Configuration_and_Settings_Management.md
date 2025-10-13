# File 29: Configuration and Settings Management

**Layer 7: Advanced Topics - Part 2 of 4**

## Table of Contents
1. [LavishSettings Foundation](#lavishsettings)
2. [EVEBot Configuration Architecture](#evebot-config)
3. [Tehbot Configuration Architecture](#tehbot-config)
4. [Configuration Patterns](#config-patterns)
5. [XML File Structure](#xml-structure)
6. [Config Macros and Helpers](#macros)
7. [UI Integration](#ui-integration)
8. [Per-Character Settings](#per-character)
9. [Config Migration](#migration)
10. [Fleet Config Synchronization](#fleet-sync)
11. [isxSQLite Configuration Storage](#isxsqlite-config)
12. [Best Practices](#best-practices)
13. [Complete Examples](#examples)

---

## LavishSettings Foundation <a name="lavishsettings"></a>

### What is LavishSettings?

**LavishSettings** is LavishScript's hierarchical configuration API:
- Stores settings in XML files
- Supports nested Sets and Settings
- Import/Export to/from XML
- Runtime modification with auto-save

### Basic LavishSettings API

```lavish
; Create a settings object
LavishSettings:AddSet[MySettings]

; Add a nested set
LavishSettings[MySettings]:AddSet[Character1]

; Add individual settings
LavishSettings[MySettings].FindSet[Character1]:AddSetting[Name, "Jovehn"]
LavishSettings[MySettings].FindSet[Character1]:AddSetting[HomeStation, "Jita 4-4"]
LavishSettings[MySettings].FindSet[Character1]:AddSetting[IsActive, TRUE]

; Read settings
variable string name = ${LavishSettings[MySettings].FindSet[Character1].FindSetting[Name]}
variable bool active = ${LavishSettings[MySettings].FindSet[Character1].FindSetting[IsActive]}

; Export to XML
LavishSettings[MySettings]:Export["config/MyConfig.xml"]

; Import from XML
LavishSettings[MySettings]:Import["config/MyConfig.xml"]

; Remove settings
LavishSettings[MySettings]:Clear
LavishSettings[MySettings]:Remove
```

### Settings Hierarchy

```
LavishSettings[MySettings]                    ; Root settings object
    └── FindSet[Character1]                   ; Character set
        ├── FindSetting[Name]                 ; Individual setting
        ├── FindSetting[HomeStation]          ; Individual setting
        └── FindSet[Behavior]                 ; Nested set
            ├── FindSetting[Mode]             ; Nested setting
            └── FindSetting[Target]           ; Nested setting
```

### settingsetref Type

**settingsetref** is a reference to a settings set, used for efficient access:

```lavish
objectdef obj_MyConfig
{
    variable settingsetref CharacterRef

    method Initialize()
    {
        ; Set reference to a deeply nested set
        CharacterRef:Set[${LavishSettings[MySettings].FindSet[Character1]}]

        ; Now access is simpler
        variable string name = ${This.CharacterRef.FindSetting[Name]}
    }
}
```

---

## EVEBot Configuration Architecture <a name="evebot-config"></a>

### Overview

EVEBot uses a sophisticated multi-layer configuration system:

**Architecture:**
```
obj_Configuration_BaseConfig                  ; File I/O, core functionality
    ├── Manages ${Me.Name} Config.xml
    ├── Import/Export
    └── Save on shutdown

obj_Configuration                             ; Main config container
    ├── Common       (obj_Configuration_Common)
    ├── Combat       (obj_Configuration_Combat)
    ├── Miner        (obj_Configuration_Miner)
    ├── Hauler       (obj_Configuration_Hauler)
    ├── Missioneer   (obj_Configuration_Missioneer)
    ├── Fleet        (obj_Configuration_Fleet)
    └── [etc...]
```

### BaseConfig Implementation

```lavish
objectdef obj_Configuration_BaseConfig
{
    variable float ConfigVersion = 1.0
    variable filepath CONFIG_PATH = "${Script.CurrentDirectory}/Config"
    variable string CONFIG_FILE = "${Me.Name} Config.xml"
    variable settingsetref BaseRef

    method Initialize()
    {
        ; Remove old settings
        LavishSettings[EVEBotSettings]:Remove

        ; Create new settings
        LavishSettings:AddSet[EVEBotSettings]
        LavishSettings[EVEBotSettings]:AddSet[${Me.Name}]

        ; Check if config file exists
        if !${CONFIG_PATH.FileExists[${CONFIG_FILE}]}
        {
            Logger:Log["${CONFIG_FILE} not found - creating new"]
            ; Config modules will set defaults
        }
        else
        {
            Logger:Log["Configuration file is ${CONFIG_FILE}"]
            LavishSettings[EVEBotSettings]:Import[${CONFIG_PATH}/${CONFIG_FILE}]
        }

        ; Set reference to character's settings
        BaseRef:Set[${LavishSettings[EVEBotSettings].FindSet[${Me.Name}]}]
    }

    method Shutdown()
    {
        This:Save
        LavishSettings[EVEBotSettings]:Remove
    }

    method Save()
    {
        Logger:Log["Saving configuration to ${CONFIG_FILE}"]
        LavishSettings[EVEBotSettings]:Export[${CONFIG_PATH}/${CONFIG_FILE}]
    }
}
```

### Main Config Object

```lavish
objectdef obj_Configuration
{
    ; Sub-configuration objects
    variable obj_Configuration_Common Common
    variable obj_Configuration_Combat Combat
    variable obj_Configuration_Miner Miner
    variable obj_Configuration_Hauler Hauler
    variable obj_Configuration_Missioneer Missioneer
    variable obj_Configuration_Fleet Fleet

    method Shutdown()
    {
        ; Sub-configs don't need individual shutdown
        ; BaseConfig handles save
    }

    method Save()
    {
        ; Delegate to BaseConfig
        BaseConfig:Save
    }
}
```

**Usage:**

```lavish
; Access is clean and organized
variable string behavior = ${Config.Common.CurrentBehavior}
variable int minShield = ${Config.Combat.MinimumShieldPct}
variable bool useDrones = ${Config.Miner.UseMiningDrones}

; Set values
Config.Common:SetCurrentBehavior["Miner"]
Config.Combat:SetMinimumShieldPct[50]
Config.Miner:SetUseMiningDrones[TRUE]

; Save to disk
Config:Save
```

### Sub-Configuration Pattern

```lavish
objectdef obj_Configuration_Common
{
    variable string SetName = "Common"
    variable settingsetref Ref

    method Initialize()
    {
        ; Check if set exists in config file
        if !${BaseConfig.BaseRef.FindSet[${This.SetName}](exists)}
        {
            Logger:Log["Warning: ${This.SetName} settings missing - initializing"]
            This:Set_Default_Values
        }
        else
        {
            ; Set exists - get reference
            Ref:Set[${BaseConfig.BaseRef.FindSet[${This.SetName}]}]
        }
    }

    method Set_Default_Values()
    {
        ; Create the set
        BaseConfig.BaseRef:AddSet[${This.SetName}]
        Ref:Set[${BaseConfig.BaseRef.FindSet[${This.SetName}]}]

        ; Add default settings
        This.Ref:AddSetting[Home Station, 1]
        This.Ref:AddSetting[CurrentBehavior, "Idle"]
        This.Ref:AddSetting[AutoLogin, TRUE]
        This.Ref:AddSetting[Maximum Runtime, 0]
        This.Ref:AddSetting[Use Sound, FALSE]
        This.Ref:AddSetting[Disable 3D, FALSE]
    }

    ; Member to read
    member:string CurrentBehavior()
    {
        return ${This.Ref.FindSetting[CurrentBehavior, "Idle"]}
    }

    ; Method to write
    method SetCurrentBehavior(string value)
    {
        This.Ref:AddSetting[CurrentBehavior, ${value}]
    }

    ; More settings...
    member:int HomeStation()
    {
        return ${This.Ref.FindSetting[Home Station, 1]}
    }

    method SetHomeStation(int value)
    {
        This.Ref:AddSetting[Home Station, ${value}]
    }

    member:bool AutoLogin()
    {
        return ${This.Ref.FindSetting[AutoLogin, TRUE]}
    }

    method SetAutoLogin(bool value)
    {
        This.Ref:AddSetting[AutoLogin, ${value}]
    }
}
```

---

## Tehbot Configuration Architecture <a name="tehbot-config"></a>

### Overview

Tehbot uses a cleaner inheritance-based pattern:

**Architecture:**
```
obj_Configuration_Manager                     ; Singleton manager
    ├── CONFIG_FILE
    ├── ConfigRoot (settingsetref)
    └── Initialize/Shutdown/Save

obj_Configuration_Base                        ; Base class
    ├── SetName
    ├── ConfigRef (member)
    └── Set_Default_Values (method)

obj_Configuration_Common : obj_Configuration_Base
obj_Configuration_Behavior : obj_Configuration_Base
obj_Configuration_Combat : obj_Configuration_Base
[etc...]
```

### ConfigManager Singleton

```lavish
objectdef obj_Configuration_Manager
{
    variable string CONFIG_FILE = "${Me.Name} Config.xml"
    variable filepath CONFIG_PATH = "${Script.CurrentDirectory}/config"
    variable settingsetref ConfigRoot

    method Initialize()
    {
        ; Handle alternative character name
        if ${EVEExtension.Character.Length}
        {
            CONFIG_FILE:Set["${EVEExtension.Character} Config.xml"]
        }

        ; Clear and recreate settings
        LavishSettings[TehbotSettings]:Clear
        LavishSettings:AddSet[TehbotSettings]

        ; Add character set
        if ${EVEExtension.Character.Length}
        {
            LavishSettings[TehbotSettings]:AddSet[${EVEExtension.Character}]
        }
        else
        {
            LavishSettings[TehbotSettings]:AddSet[${Me.Name}]
        }

        ; Import if exists
        if ${CONFIG_PATH.FileExists["${CONFIG_PATH}/${CONFIG_FILE}"]}
        {
            LavishSettings[TehbotSettings]:Import["${CONFIG_PATH}/${CONFIG_FILE}"]
        }

        ; Set root reference
        if ${EVEExtension.Character.Length}
        {
            ConfigRoot:Set[${LavishSettings[TehbotSettings].FindSet[${EVEExtension.Character}]}]
        }
        else
        {
            ConfigRoot:Set[${LavishSettings[TehbotSettings].FindSet[${Me.Name}]}]
        }
    }

    method Shutdown()
    {
        This:Save
        LavishSettings[TehbotSettings]:Clear
    }

    method Save()
    {
        LavishSettings[TehbotSettings]:Export["${CONFIG_PATH}/${CONFIG_FILE}"]
    }
}

; Global instantiation
variable(global) obj_Configuration_Manager ConfigManager
```

### Base Configuration Class

```lavish
objectdef obj_Configuration_Base
{
    variable string SetName = ""

    method Initialize(string name)
    {
        SetName:Set[${name}]

        ; Check if set exists
        if !${ConfigManager.ConfigRoot.FindSet[${This.SetName}](exists)}
        {
            Logger:Log["${This.SetName} settings missing - initializing"]
            ConfigManager.ConfigRoot:AddSet[${This.SetName}]
            This:Set_Default_Values
        }
    }

    ; Get reference to this config's set
    member:settingsetref ConfigRef()
    {
        return ${ConfigManager.ConfigRoot.FindSet[${This.SetName}]}
    }

    ; Override in derived classes
    method Set_Default_Values()
    {
        ; Nothing - override in derived classes
    }
}
```

### Derived Configuration Classes

```lavish
objectdef obj_Configuration_Common inherits obj_Configuration_Base
{
    method Initialize()
    {
        ; Call parent with set name
        This[parent]:Initialize["Common"]
    }

    method Set_Default_Values()
    {
        ; This.ConfigRef is provided by base class
        This.ConfigRef:AddSetting[Tehbot_Mode, "MiniMode"]
        This.ConfigRef:AddSetting[ActiveTab, "Status"]
        This.ConfigRef:AddSetting[LogLevelBar, LOG_INFO]
        This.ConfigRef:AddSetting[AutoStart, FALSE]
        This.ConfigRef:AddSetting[Disable3D, FALSE]
        This.ConfigRef:AddSetting[DisableUI, FALSE]
    }

    ; Settings using Setting() macro (defined elsewhere)
    Setting(string, Tehbot_Mode, SetTehbot_Mode)
    Setting(bool, AutoStart, SetAutoStart)
    Setting(bool, Disable3D, SetDisable3D)
    Setting(bool, DisableUI, SetDisableUI)
    Setting(string, ActiveTab, SetActiveTab)
    Setting(int, LogLevelBar, SetLogLevelBar)
}
```

**Setting() Macro:**

```lavish
#define Setting(type, name, setMethod) \
    member:type name() \
    { \
        return ${This.ConfigRef.FindSetting[#name]} \
    } \
    method setMethod(type value) \
    { \
        This.ConfigRef:AddSetting[#name, ${value}] \
    }
```

---

## Configuration Patterns <a name="config-patterns"></a>

### Pattern 1: Member/Method Pair

**Every config value needs:**
- **Member:** Read-only accessor
- **Method:** Write accessor that updates config

```lavish
; MEMBER - Read value
member:int MinimumShieldPct()
{
    return ${This.ConfigRef.FindSetting[MinimumShieldPct, 50]}
                                                          ; ^^^ Default value
}

; METHOD - Write value
method SetMinimumShieldPct(int value)
{
    This.ConfigRef:AddSetting[MinimumShieldPct, ${value}]
}

; Usage
variable int shield = ${Config.Combat.MinimumShieldPct}    ; Read
Config.Combat:SetMinimumShieldPct[75]                      ; Write
```

### Pattern 2: Default Values

**Always provide defaults** in case setting doesn't exist:

```lavish
; Inline default
member:string CurrentBehavior()
{
    return ${This.ConfigRef.FindSetting[CurrentBehavior, "Idle"]}
}

; Constant default
member:int MaxTargets()
{
    return ${This.ConfigRef.FindSetting[MaxTargets, ${EVE.MaxLockedTargets}]}
}

; Calculated default
member:int OrbitDistance()
{
    return ${This.ConfigRef.FindSetting[OrbitDistance, ${Math.Calc[${MyShip.MaxTargetRange} * 0.8]}]}
}
```

### Pattern 3: Nested Sets

For complex configuration with sub-categories:

```lavish
objectdef obj_Configuration_Miner
{
    variable settingsetref Ref
    variable settingsetref OreTypesRef

    method Initialize()
    {
        Ref:Set[${BaseConfig.BaseRef.FindSet["Miner"]}]

        ; Check if nested set exists
        if !${This.Ref.FindSet["ORE_Types"](exists)}
        {
            This.Ref:AddSet["ORE_Types"]
        }

        OreTypesRef:Set[${This.Ref.FindSet["ORE_Types"]}]
    }

    method Set_Default_Values()
    {
        ; Main miner settings
        This.Ref:AddSetting[Strip Mine, TRUE]
        This.Ref:AddSetting[Ice Mining, FALSE]
        This.Ref:AddSetting[Cargo Threshold, 11500]

        ; Nested ore types
        This.Ref:AddSet["ORE_Types"]
        OreTypesRef:Set[${This.Ref.FindSet["ORE_Types"]}]

        ; Add each ore type
        This.OreTypesRef:AddSetting["Veldspar", 1]
        This.OreTypesRef:AddSetting["Scordite", 1]
        This.OreTypesRef:AddSetting["Pyroxeres", 1]
        This.OreTypesRef:AddSetting["Plagioclase", 1]
        ; ... etc
    }

    ; Check if ore type enabled
    member:bool OreEnabled(string oreName)
    {
        return ${This.OreTypesRef.FindSetting[${oreName}, 0]}
    }

    ; Enable/disable ore type
    method SetOreEnabled(string oreName, bool enabled)
    {
        This.OreTypesRef:AddSetting[${oreName}, ${If[${enabled}, 1, 0]}]
    }
}
```

### Pattern 4: Config Lists (Collections)

Store lists of items:

```lavish
objectdef obj_Configuration_Targets
{
    method Initialize()
    {
        if !${This.ConfigRef.FindSet["PriorityTargets"](exists)}
        {
            This.ConfigRef:AddSet["PriorityTargets"]
            This:Set_Default_Priority_Targets
        }
    }

    method Set_Default_Priority_Targets()
    {
        variable settingsetref PriorityRef
        PriorityRef:Set[${This.ConfigRef.FindSet["PriorityTargets"]}]

        ; Each setting is a priority target (name, priority)
        PriorityRef:AddSetting["Dire Pithi Arrogator", 10]
        PriorityRef:AddSetting["Guardian Agent", 10]
        PriorityRef:AddSetting["Pithi Saboteur", 8]
        PriorityRef:AddSetting["Frigate", 5]
    }

    ; Get priority for target name
    member:int GetTargetPriority(string targetName)
    {
        return ${This.ConfigRef.FindSet["PriorityTargets"].FindSetting[${targetName}, 0]}
    }

    ; Get all priority targets
    method GetPriorityTargets(index:string targetList)
    {
        variable iterator Setting
        This.ConfigRef.FindSet["PriorityTargets"]:GetSettingIterator[Setting]

        if ${Setting:First(exists)}
        {
            do
            {
                targetList:Insert["${Setting.Key}"]
            }
            while ${Setting:Next(exists)}
        }
    }
}
```

---

## XML File Structure <a name="xml-structure"></a>

### EVEBot XML Structure

```xml
<?xml version='1.0' encoding='UTF-8'?>
<!-- Generated by LavishSettings v2 -->
<InnerSpaceSettings>
    <Set Name="Jovehn">                          <!-- Character Name -->
        <Set Name="Common">                      <!-- Config Category -->
            <Setting Name="Home Station">Jita 4-4</Setting>
            <Setting Name="CurrentBehavior">Miner</Setting>
            <Setting Name="AutoLogin">TRUE</Setting>
            <Setting Name="Maximum Runtime">8</Setting>
            <Setting Name="Disable 3D">FALSE</Setting>
        </Set>

        <Set Name="Combat">                      <!-- Combat Category -->
            <Setting Name="MinimumShieldPct">50</Setting>
            <Setting Name="MinimumArmorPct">70</Setting>
            <Setting Name="MinimumCapPct">25</Setting>
            <Setting Name="Launch Combat Drones">TRUE</Setting>
            <Setting Name="OrbitDistance">30000</Setting>
        </Set>

        <Set Name="Miner">                       <!-- Miner Category -->
            <Setting Name="Strip Mine">TRUE</Setting>
            <Setting Name="Cargo Threshold">11500</Setting>
            <Setting Name="Ice Mining">FALSE</Setting>

            <Set Name="ORE_Types">               <!-- Nested Set -->
                <Setting Name="Veldspar">1</Setting>
                <Setting Name="Scordite">1</Setting>
                <Setting Name="Pyroxeres">1</Setting>
                <Setting Name="Plagioclase">1</Setting>
                <Setting Name="Omber">1</Setting>
                <Setting Name="Kernite">1</Setting>
                <!-- ... more ore types ... -->
            </Set>

            <Set Name="ICE_Types">               <!-- Another Nested Set -->
                <Setting Name="Blue Ice">1</Setting>
                <Setting Name="Clear Icicle">1</Setting>
                <Setting Name="Dark Glitter">1</Setting>
                <!-- ... more ice types ... -->
            </Set>
        </Set>

        <Set Name="Fleet">                       <!-- Fleet Category -->
            <Setting Name="FleetMode">TRUE</Setting>
            <Setting Name="MasterName">Hauler Main</Setting>
            <Setting Name="IsMaster">FALSE</Setting>
        </Set>
    </Set>
</InnerSpaceSettings>
```

### File Naming Convention

```
EVEBot:
    Jovehn Config.xml
    Jovehn2 Config.xml
    Miner Alt Config.xml
    Hauler Main Config.xml

Tehbot:
    MyCharacter Config.xml
    Combat Pilot Config.xml
```

**Pattern:** `${Me.Name} Config.xml`

### Multiple Config Files

```
Config/
    ├── Jovehn Config.xml                        ; Character config
    ├── Jovehn Blacklist.xml                     ; Character blacklist
    ├── Jovehn Whitelist.xml                     ; Character whitelist
    ├── Jovehn Mission Cache.xml                 ; Mission data
    └── Launcher_example.xml                     ; Fleet launcher config
```

---

## Config Macros and Helpers <a name="macros"></a>

### EVEBot Define_ConfigItem Macro

```lavish
#macro Define_ConfigItem(_Type, _Key, _DefaultValue)
    ; Member to read
    member:_Type _Key()
    {
        return ${This.Ref.FindSetting[_Key, _DefaultValue]}
    }

    ; Method to write
    method _Key(_Type Value)
    {
        This.Ref:AddSetting[_Key, ${Value}]
    }
#endmac
```

**Usage:**

```lavish
objectdef obj_Configuration_Combat
{
    variable settingsetref Ref

    ; Instead of writing member/method pairs manually...
    Define_ConfigItem(int, MinimumShieldPct, 50)
    Define_ConfigItem(int, MinimumArmorPct, 70)
    Define_ConfigItem(int, MinimumCapPct, 25)
    Define_ConfigItem(bool, LaunchCombatDrones, TRUE)
    Define_ConfigItem(int, OrbitDistance, 30000)
}

; Creates:
; member:int MinimumShieldPct() { return ${This.Ref.FindSetting[MinimumShieldPct, 50]} }
; method MinimumShieldPct(int Value) { This.Ref:AddSetting[MinimumShieldPct, ${Value}] }
; ... etc for each config item
```

### Tehbot Setting Macro

```lavish
#define Setting(type, name, setMethod) \
    member:type name() \
    { \
        return ${This.ConfigRef.FindSetting[#name]} \
    } \
    method setMethod(type value) \
    { \
        This.ConfigRef:AddSetting[#name, ${value}] \
    }
```

**Usage:**

```lavish
objectdef obj_Configuration_Common inherits obj_Configuration_Base
{
    Setting(string, Tehbot_Mode, SetTehbot_Mode)
    Setting(bool, AutoStart, SetAutoStart)
    Setting(bool, Disable3D, SetDisable3D)
    Setting(int, LogLevelBar, SetLogLevelBar)
}

; Creates:
; member:string Tehbot_Mode() { return ${This.ConfigRef.FindSetting[Tehbot_Mode]} }
; method SetTehbot_Mode(string value) { This.ConfigRef:AddSetting[Tehbot_Mode, ${value}] }
; ... etc
```

### Config Helper Functions

```lavish
; Check if setting exists
member:bool SettingExists(string settingName)
{
    return ${This.ConfigRef.FindSetting[${settingName}](exists)}
}

; Get setting with fallback
member:string GetSetting(string settingName, string defaultValue)
{
    if ${This:SettingExists[${settingName}]}
    {
        return ${This.ConfigRef.FindSetting[${settingName}]}
    }
    return ${defaultValue}
}

; Ensure setting has value
method EnsureSetting(string settingName, string defaultValue)
{
    if !${This:SettingExists[${settingName}]}
    {
        This.ConfigRef:AddSetting[${settingName}, ${defaultValue}]
    }
}
```

---

## UI Integration <a name="ui-integration"></a>

### LavishGUI Config Binding

Config values can be bound to UI elements for real-time editing:

```xml
<!-- EVEBot UI XML -->
<Tab Name="Common" Template='EveTabTemplate'>
    <ComboBox Name='CurrentBehavior' Template='EveComboTemplate'>
        <X>10</X>
        <Y>50</Y>
        <Width>200</Width>
        <Height>25</Height>
        <Items>
            <Item Value='Miner' Text='Miner'/>
            <Item Value='Hauler' Text='Hauler'/>
            <Item Value='Combat' Text='Combat'/>
            <Item Value='Idle' Text='Idle'/>
        </Items>
        <SelectedItem>${Config.Common.CurrentBehavior}</SelectedItem>
        <OnSelect>
            Config.Common:SetCurrentBehavior["${This.SelectedItem.Value}"]
        </OnSelect>
    </ComboBox>

    <CheckBox Name='AutoLogin' Template='EveCheckBoxTemplate'>
        <X>10</X>
        <Y>100</Y>
        <Text>Auto Login</Text>
        <Checked>${Config.Common.AutoLogin}</Checked>
        <OnLeftClick>
            Config.Common:SetAutoLogin[${This.Checked}]
        </OnLeftClick>
    </CheckBox>

    <TextEntry Name='MaxRuntime' Template='EveTextEntryTemplate'>
        <X>10</X>
        <Y>150</Y>
        <Width>100</Width>
        <Height>25</Height>
        <Text>${Config.Common.MaxRuntime}</Text>
        <OnChange>
            Config.Common:SetMaxRuntime[${This.Text}]
        </OnChange>
    </TextEntry>
</Tab>
```

### UI Update Pattern

```lavish
objectdef obj_ConfigUI
{
    method Initialize()
    {
        Event[EVENT_ONFRAME]:AttachAtom[This:UpdateUI]
    }

    method UpdateUI()
    {
        ; Update UI elements with config values
        UIElement[CurrentBehavior@Common]:SetText[${Config.Common.CurrentBehavior}]
        UIElement[MinShield@Combat]:SetText[${Config.Combat.MinimumShieldPct}]
        UIElement[AutoLogin@Common]:SetChecked[${Config.Common.AutoLogin}]
    }

    method OnBehaviorChanged()
    {
        ; Get value from UI
        variable string newBehavior = ${UIElement[CurrentBehavior@Common].SelectedItem.Value}

        ; Update config
        Config.Common:SetCurrentBehavior[${newBehavior}]

        ; Save to disk
        Config:Save

        echo "Behavior changed to: ${newBehavior}"
    }
}
```

### Config Tabs Pattern

Organize config UI into tabs by category:

```xml
<TabControl Name='ConfigTabs'>
    <Tab Name='Common'>
        <!-- General settings: behavior, home station, runtime -->
    </Tab>
    <Tab Name='Combat'>
        <!-- Combat settings: tank thresholds, orbit, drones -->
    </Tab>
    <Tab Name='Mining'>
        <!-- Mining settings: ore types, ice, cargo threshold -->
    </Tab>
    <Tab Name='Hauling'>
        <!-- Hauling settings: routes, autopilot, delivery -->
    </Tab>
    <Tab Name='Fleet'>
        <!-- Fleet settings: master/slave, coordination -->
    </Tab>
</TabControl>
```

---

## Per-Character Settings <a name="per-character"></a>

### Character-Specific Config Files

**Pattern:** One config file per character

```lavish
objectdef obj_Configuration_BaseConfig
{
    variable string CONFIG_FILE = "${Me.Name} Config.xml"

    method Initialize()
    {
        ; Each character has their own file
        LavishSettings:AddSet[EVEBotSettings]
        LavishSettings[EVEBotSettings]:AddSet[${Me.Name}]

        ; Import character's config
        if ${CONFIG_PATH.FileExists[${CONFIG_FILE}]}
        {
            LavishSettings[EVEBotSettings]:Import[${CONFIG_PATH}/${CONFIG_FILE}]
        }
    }
}
```

**Result:**
```
Config/
    Miner Main Config.xml
    Hauler Alt Config.xml
    Combat Pilot Config.xml
```

### Shared Config Pattern

For settings that should be shared across all characters:

```lavish
objectdef obj_Configuration_Shared
{
    variable string SHARED_CONFIG = "SharedConfig.xml"
    variable settingsetref SharedRef

    method Initialize()
    {
        ; Create separate settings object for shared config
        LavishSettings:AddSet[SharedSettings]

        if ${CONFIG_PATH.FileExists[${SHARED_CONFIG}]}
        {
            LavishSettings[SharedSettings]:Import[${CONFIG_PATH}/${SHARED_CONFIG}]
        }

        SharedRef:Set[${LavishSettings[SharedSettings]}]
    }

    ; Shared settings accessible to all characters
    member:string FleetChannel()
    {
        return ${This.SharedRef.FindSetting[Fleet Channel, "FleetChat"]}
    }

    member:string IRCServer()
    {
        return ${This.SharedRef.FindSetting[IRC Server, "irc.example.com"]}
    }

    member:string MasterName()
    {
        return ${This.SharedRef.FindSetting[Master Name, "FleetCommander"]}
    }
}
```

### Character Roles Pattern

Define character roles in config:

```lavish
objectdef obj_Configuration_Character
{
    method Set_Default_Values()
    {
        This.ConfigRef:AddSetting[CharacterRole, "DPS"]
        This.ConfigRef:AddSetting[CharacterPriority, 1]
        This.ConfigRef:AddSetting[FleetPosition, "DPS"]
    }

    member:string CharacterRole()
    {
        return ${This.ConfigRef.FindSetting[CharacterRole, "DPS"]}
    }

    method SetCharacterRole(string role)
    {
        This.ConfigRef:AddSetting[CharacterRole, ${role}]
    }
}

; Usage in fleet coordination
method AssignFleetRoles()
{
    switch ${Config.Character.CharacterRole}
    {
        case FC
            This:SetupFleetCommander
            break
        case DPS
            This:SetupDPSShip
            break
        case Logi
            This:SetupLogisticsShip
            break
        case Scout
            This:SetupScoutShip
            break
    }
}
```

---

## Config Migration <a name="migration"></a>

### Version Tracking

```lavish
objectdef obj_Configuration_BaseConfig
{
    variable float ConfigVersion = 2.1

    method Initialize()
    {
        ; Import existing config
        LavishSettings[EVEBotSettings]:Import[${CONFIG_FILE}]

        ; Check version
        variable float fileVersion = ${BaseRef.FindSetting[ConfigVersion, 1.0]}

        if ${fileVersion} < ${This.ConfigVersion}
        {
            Logger:Log["Config version ${fileVersion} -> ${This.ConfigVersion}, migrating"]
            This:MigrateConfig[${fileVersion}]
        }
    }

    method MigrateConfig(float fromVersion)
    {
        ; Migrate from 1.0 to 2.0
        if ${fromVersion} < 2.0
        {
            This:Migrate_1_0_to_2_0
        }

        ; Migrate from 2.0 to 2.1
        if ${fromVersion} < 2.1
        {
            This:Migrate_2_0_to_2_1
        }

        ; Update version number
        BaseRef:AddSetting[ConfigVersion, ${This.ConfigVersion}]
        This:Save
    }

    method Migrate_1_0_to_2_0()
    {
        Logger:Log["Migrating config 1.0 -> 2.0"]

        ; Rename settings
        variable settingsetref CommonRef = ${BaseRef.FindSet["Common"]}

        if ${CommonRef.FindSetting["Bot Mode Name"](exists)}
        {
            ; Old key: "Bot Mode Name"
            ; New key: "CurrentBehavior"
            variable string oldValue = ${CommonRef.FindSetting["Bot Mode Name"]}
            CommonRef:AddSetting[CurrentBehavior, ${oldValue}]
            CommonRef.FindSetting["Bot Mode Name"]:Remove
            CommonRef.FindSetting["Bot Mode"]:Remove
        }
    }

    method Migrate_2_0_to_2_1()
    {
        Logger:Log["Migrating config 2.0 -> 2.1"]

        ; Add new settings with defaults
        variable settingsetref CombatRef = ${BaseRef.FindSet["Combat"]}

        if !${CombatRef.FindSetting["EnableDroneDefense"](exists)}
        {
            CombatRef:AddSetting[EnableDroneDefense, TRUE]
        }
    }
}
```

### Real Example from EVEBot

```lavish
; From obj_Configuration_Common:
method Initialize()
{
    if ${This.Ref.FindSetting[Bot Mode Name](exists)}
    {
        ; The previous key was present, migrate it to the new one and delete it
        This.Ref:AddSetting[CurrentBehavior, ${This.Ref.FindSetting[Bot Mode Name]}]
        This.Ref.FindSetting[Bot Mode]:Remove
        This.Ref.FindSetting[Bot Mode Name]:Remove
        Logger:Log["Configuration: Migrating config value: Bot Mode Name -> Behavior (${This.Ref.FindSetting[CurrentBehavior]})", LOG_ECHOTOO]
    }
}
```

### Backup Before Migration

```lavish
method MigrateConfig(float fromVersion)
{
    ; Backup config before migration
    variable string backupFile = "${CONFIG_FILE}.v${fromVersion}.backup"
    LavishSettings[EVEBotSettings]:Export[${CONFIG_PATH}/${backupFile}]
    Logger:Log["Config backed up to ${backupFile}"]

    ; Perform migration
    This:Migrate_${fromVersion}_to_${This.ConfigVersion}

    ; Save migrated config
    This:Save
}
```

---

## Fleet Config Synchronization <a name="fleet-sync"></a>

### Broadcasting Config Changes

Use relay to synchronize config across fleet:

```lavish
objectdef obj_ConfigSync
{
    method Initialize()
    {
        LavishScript:RegisterEvent[Fleet_Config_Update]
        Event[Fleet_Config_Update]:AttachAtom[This:OnConfigUpdate]
    }

    ; Master broadcasts config change
    method BroadcastConfigChange(string category, string setting, string value)
    {
        relay "all other" -event Fleet_Config_Update "${category}" "${setting}" "${value}"
    }

    ; Slaves receive and apply
    atom OnConfigUpdate(string category, string setting, string value)
    {
        echo "Config update: ${category}.${setting} = ${value}"

        switch ${category}
        {
            case Common
                This:UpdateCommonConfig["${setting}", "${value}"]
                break
            case Combat
                This:UpdateCombatConfig["${setting}", "${value}"]
                break
            case Fleet
                This:UpdateFleetConfig["${setting}", "${value}"]
                break
        }
    }

    method UpdateCommonConfig(string setting, string value)
    {
        switch ${setting}
        {
            case CurrentBehavior
                Config.Common:SetCurrentBehavior["${value}"]
                break
            case MaxRuntime
                Config.Common:SetMaxRuntime[${value}]
                break
        }

        Config:Save
    }
}
```

### Master Config Pattern

Fleet master sets config for all slaves:

```lavish
objectdef obj_FleetMaster
{
    ; Set fleet-wide behavior
    method SetFleetBehavior(string behavior)
    {
        ; Set locally
        Config.Common:SetCurrentBehavior[${behavior}]

        ; Broadcast to fleet
        relay all -event Fleet_Config_Update "Common" "CurrentBehavior" "${behavior}"

        echo "Fleet behavior set to: ${behavior}"
    }

    ; Set fleet-wide combat threshold
    method SetFleetMinShield(int shieldPct)
    {
        Config.Combat:SetMinimumShieldPct[${shieldPct}]
        relay all -event Fleet_Config_Update "Combat" "MinimumShieldPct" "${shieldPct}"

        echo "Fleet min shield set to: ${shieldPct}%"
    }

    ; Synchronize all config to fleet
    method SyncConfigToFleet()
    {
        echo "Syncing config to entire fleet..."

        ; Common settings
        relay all -event Fleet_Config_Update "Common" "CurrentBehavior" "${Config.Common.CurrentBehavior}"
        relay all -event Fleet_Config_Update "Common" "MaxRuntime" "${Config.Common.MaxRuntime}"

        ; Combat settings
        relay all -event Fleet_Config_Update "Combat" "MinimumShieldPct" "${Config.Combat.MinimumShieldPct}"
        relay all -event Fleet_Config_Update "Combat" "MinimumArmorPct" "${Config.Combat.MinimumArmorPct}"

        ; Fleet settings
        relay all -event Fleet_Config_Update "Fleet" "MasterName" "${Me.Name}"

        echo "Config sync complete"
    }
}
```

---

## isxSQLite Configuration Storage <a name="isxsqlite-config"></a>

### Overview

**isxSQLite** provides an alternative to XML-based configuration using SQLite databases:

**Advantages over XML/LavishSettings:**
- ✅ Better performance for large configs
- ✅ Easier querying with SQL
- ✅ Built-in data validation (column types)
- ✅ Transaction support for atomic updates
- ✅ Can store historical data (versioning)
- ✅ Better for complex data structures

**Disadvantages:**
- ❌ Requires isxSQLite extension
- ❌ More complex setup
- ❌ Not human-editable (binary format)
- ❌ Requires manual database schema design

**See Also:** `__CRITICAL_NEWEST_isxSQLite_Reference.md` for complete API documentation.

### Basic Database Config Pattern

```lavish
objectdef obj_ConfigDatabase
{
    variable sqlitedb ConfigDB
    variable string DBPath = "${Script.CurrentDirectory}/config/botconfig.db"

    method Initialize()
    {
        ; Check if isxSQLite loaded
        if !${ISXSQLite(exists)}
        {
            echo "ERROR: isxSQLite not loaded - run: extension isxsqlite"
            return FALSE
        }

        ; Wait for ready
        while !${ISXSQLite.IsReady}
            waitframe

        ; Open/create database
        ConfigDB:Set[${SQLite.OpenDB["BotConfig", "${This.DBPath}"]}]

        if !${ConfigDB.ID(exists)}
        {
            echo "ERROR: Failed to open config database"
            return FALSE
        }

        ; Initialize schema
        This:InitializeSchema

        echo "Config database ready: ${This.DBPath}"
        return TRUE
    }

    method InitializeSchema()
    {
        ; Create settings table if doesn't exist
        if !${ConfigDB.TableExists["settings"]}
        {
            ConfigDB:ExecDML["CREATE TABLE settings (
                category TEXT NOT NULL,
                key TEXT NOT NULL,
                value TEXT,
                type TEXT,
                PRIMARY KEY (category, key)
            );"]

            ; Insert defaults
            This:LoadDefaults
        }
    }

    method LoadDefaults()
    {
        ; Use transaction for bulk insert
        variable index:string DML

        DML:Insert["INSERT OR REPLACE INTO settings VALUES ('General', 'BotMode', 'Idle', 'string');"]
        DML:Insert["INSERT OR REPLACE INTO settings VALUES ('General', 'HomeStation', 'Jita 4-4', 'string');"]
        DML:Insert["INSERT OR REPLACE INTO settings VALUES ('General', 'AutoStart', 'FALSE', 'bool');"]

        DML:Insert["INSERT OR REPLACE INTO settings VALUES ('Combat', 'MinShield', '50', 'int');"]
        DML:Insert["INSERT OR REPLACE INTO settings VALUES ('Combat', 'MinArmor', '70', 'int');"]
        DML:Insert["INSERT OR REPLACE INTO settings VALUES ('Combat', 'MinCap', '25', 'int');"]

        ConfigDB:ExecDMLTransaction[DML]
    }

    method Close()
    {
        if ${ConfigDB.ID(exists)}
            ConfigDB:Close
    }

    ; Get setting value
    member:string Get(string category, string key)
    {
        ; Escape inputs to prevent SQL injection
        variable string SafeCategory = "${SQLite.Escape_String[${category}]}"
        variable string SafeKey = "${SQLite.Escape_String[${key}]}"

        variable sqlitequery Query
        Query:Set[${ConfigDB.ExecQuery["SELECT value FROM settings WHERE category='${SafeCategory}' AND key='${SafeKey}'"]}]

        variable string Value = ""
        if ${Query.NumRows} > 0
            Value:Set["${Query.GetFieldValue[1]}"]

        Query:Finalize
        return "${Value}"
    }

    member:int GetInt(string category, string key)
    {
        variable string val = "${This.Get[${category}, ${key}]}"
        if ${val.Length} > 0
            return ${val}
        return 0
    }

    member:bool GetBool(string category, string key)
    {
        variable string val = "${This.Get[${category}, ${key}]}"
        return ${val.Equal["TRUE"]}
    }

    ; Set setting value
    method Set(string category, string key, string value, string type)
    {
        variable string SafeCategory = "${SQLite.Escape_String[${category}]}"
        variable string SafeKey = "${SQLite.Escape_String[${key}]}"
        variable string SafeValue = "${SQLite.Escape_String[${value}]}"

        ConfigDB:ExecDML["INSERT OR REPLACE INTO settings VALUES ('${SafeCategory}', '${SafeKey}', '${SafeValue}', '${type}');"]
    }
}
```

### Config Object with Database Backend

```lavish
objectdef obj_Config_General
{
    ; Get values
    member:string BotMode()
    {
        variable string val = "${DBConfig.Get["General", "BotMode"]}"
        if ${val.Length} == 0
            return "Idle"
        return "${val}"
    }

    member:string HomeStation()
    {
        variable string val = "${DBConfig.Get["General", "HomeStation"]}"
        if ${val.Length} == 0
            return "Jita 4-4"
        return "${val}"
    }

    member:bool AutoStart()
    {
        return ${DBConfig.GetBool["General", "AutoStart"]}
    }

    ; Set values
    method SetBotMode(string mode)
    {
        DBConfig:Set["General", "BotMode", "${mode}", "string"]
    }

    method SetHomeStation(string station)
    {
        DBConfig:Set["General", "HomeStation", "${station}", "string"]
    }

    method SetAutoStart(bool value)
    {
        DBConfig:Set["General", "AutoStart", "${If[${value}, TRUE, FALSE]}", "bool"]
    }
}

objectdef obj_Config_Combat
{
    member:int MinShield()
    {
        variable int val = ${DBConfig.GetInt["Combat", "MinShield"]}
        if ${val} == 0
            return 50
        return ${val}
    }

    method SetMinShield(int value)
    {
        DBConfig:Set["Combat", "MinShield", "${value}", "int"]
    }

    ; ... similar for other combat settings
}

; Main config object
objectdef obj_Config
{
    variable obj_Config_General General
    variable obj_Config_Combat Combat
}

; Global instances
variable(global) obj_ConfigDatabase DBConfig
variable(global) obj_Config Config

; Usage
DBConfig:Initialize
echo "Bot Mode: ${Config.General.BotMode}"
Config.General:SetBotMode["Mining"]
DBConfig:Close
```

### Hybrid Approach: XML + Database

Use XML for user-editable settings, database for runtime/computed data:

```lavish
objectdef obj_HybridConfig
{
    ; User settings (XML)
    variable obj_ConfigManager XMLConfig

    ; Runtime data (Database)
    variable obj_ConfigDatabase DBConfig

    method Initialize()
    {
        ; Initialize both
        XMLConfig:Initialize
        DBConfig:Initialize

        ; Store runtime info in database
        This:UpdateRuntimeInfo
    }

    method UpdateRuntimeInfo()
    {
        ; Track session statistics
        DBConfig:Set["Runtime", "LastStartTime", "${Time.Timestamp}", "int64"]
        DBConfig:Set["Runtime", "CharacterName", "${Me.Name}", "string"]
        DBConfig:Set["Runtime", "Version", "4.5.2", "string"]
    }

    ; User preferences from XML
    member:string BotMode()
    {
        return ${XMLConfig.General.BotMode}
    }

    ; Runtime stats from database
    member:int64 LastStartTime()
    {
        return ${DBConfig.GetInt["Runtime", "LastStartTime"]}
    }
}
```

### Config with History/Versioning

Track config changes over time:

```lavish
objectdef obj_ConfigWithHistory
{
    variable sqlitedb ConfigDB

    method InitializeSchema()
    {
        ; Current config table
        ConfigDB:ExecDML["CREATE TABLE IF NOT EXISTS settings (
            category TEXT NOT NULL,
            key TEXT NOT NULL,
            value TEXT,
            updated_at INTEGER,
            PRIMARY KEY (category, key)
        );"]

        ; History table
        ConfigDB:ExecDML["CREATE TABLE IF NOT EXISTS settings_history (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            category TEXT,
            key TEXT,
            old_value TEXT,
            new_value TEXT,
            changed_at INTEGER,
            changed_by TEXT
        );"]
    }

    method Set(string category, string key, string newValue)
    {
        ; Get old value
        variable string oldValue = "${This.Get[${category}, ${key}]}"

        ; Update current value
        variable string SafeCat = "${SQLite.Escape_String[${category}]}"
        variable string SafeKey = "${SQLite.Escape_String[${key}]}"
        variable string SafeVal = "${SQLite.Escape_String[${newValue}]}"

        ConfigDB:ExecDML["INSERT OR REPLACE INTO settings VALUES (
            '${SafeCat}',
            '${SafeKey}',
            '${SafeVal}',
            ${Time.Timestamp}
        );"]

        ; Record history
        variable string SafeOld = "${SQLite.Escape_String[${oldValue}]}"
        variable string SafeName = "${SQLite.Escape_String[${Me.Name}]}"

        ConfigDB:ExecDML["INSERT INTO settings_history (category, key, old_value, new_value, changed_at, changed_by) VALUES (
            '${SafeCat}',
            '${SafeKey}',
            '${SafeOld}',
            '${SafeVal}',
            ${Time.Timestamp},
            '${SafeName}'
        );"]
    }

    ; Get config history for a setting
    method GetHistory(string category, string key)
    {
        variable sqlitequery Query
        Query:Set[${ConfigDB.ExecQuery["SELECT old_value, new_value, changed_at, changed_by
            FROM settings_history
            WHERE category='${SQLite.Escape_String[${category}]}' AND key='${SQLite.Escape_String[${key}]}'
            ORDER BY changed_at DESC
            LIMIT 10"]}]

        echo "History for ${category}.${key}:"
        if ${Query.NumRows} > 0
        {
            do
            {
                echo "  ${Query.GetFieldValue[4]} changed '${Query.GetFieldValue[1]}' -> '${Query.GetFieldValue[2]}' at ${Query.GetFieldValue[3]}"
                Query:NextRow
            }
            while !${Query.LastRow}
        }

        Query:Finalize
    }
}
```

### Performance Comparison

**XML (LavishSettings):**
```lavish
; Read: ~100 settings in ~5ms
; Write: ~100 settings in ~50ms (export entire file)
; Good for: < 1000 settings, human-editable
```

**Database (isxSQLite):**
```lavish
; Read: ~100 settings in ~2ms (query)
; Write: ~100 settings in ~1ms (transaction)
; Good for: > 1000 settings, complex queries, history
```

### When to Use isxSQLite Config

**Use isxSQLite when:**
- Storing large amounts of configuration data (> 500 settings)
- Need complex queries (e.g., "all ore types with priority > 5")
- Want configuration history/audit trail
- Sharing config data between multiple scripts
- Need transactional updates
- Storing runtime statistics alongside config

**Use XML/LavishSettings when:**
- Simple, small configuration files
- Need human-editability
- Following existing bot patterns (EVEBot, Tehbot)
- Don't want extra dependency on isxSQLite

---

## Best Practices <a name="best-practices"></a>

### 1. Always Provide Defaults

```lavish
; GOOD - has default
member:int MinShield()
{
    return ${This.ConfigRef.FindSetting[MinShield, 50]}
}

; BAD - no default, crashes if missing
member:int MinShield()
{
    return ${This.ConfigRef.FindSetting[MinShield]}
}
```

### 2. Validate Config Values

```lavish
method SetMinimumShieldPct(int value)
{
    ; Validate range
    if ${value} < 0 || ${value} > 100
    {
        Logger:Log["ERROR: Invalid shield pct ${value}, must be 0-100"]
        return
    }

    This.ConfigRef:AddSetting[MinimumShieldPct, ${value}]
}

method SetOrbitDistance(int distance)
{
    ; Validate against ship capabilities
    if ${distance} > ${MyShip.MaxTargetRange}
    {
        Logger:Log["WARNING: Orbit distance ${distance}m exceeds targeting range ${MyShip.MaxTargetRange}m"]
        distance:Set[${MyShip.MaxTargetRange}]
    }

    This.ConfigRef:AddSetting[OrbitDistance, ${distance}]
}
```

### 3. Save After Critical Changes

```lavish
method SetCurrentBehavior(string behavior)
{
    ; Update config
    This.ConfigRef:AddSetting[CurrentBehavior, ${behavior}]

    ; Save immediately for critical setting
    BaseConfig:Save

    echo "Behavior changed to ${behavior} and saved"
}
```

### 4. Use Inheritance for Related Configs

```lavish
; Base behavior config
objectdef obj_Configuration_BehaviorBase inherits obj_Configuration_Base
{
    method Set_Default_Values()
    {
        ; Common to all behaviors
        This.ConfigRef:AddSetting[Enabled, TRUE]
        This.ConfigRef:AddSetting[Priority, 1]
    }

    Setting(bool, Enabled, SetEnabled)
    Setting(int, Priority, SetPriority)
}

; Specific behavior configs inherit common settings
objectdef obj_Configuration_Miner inherits obj_Configuration_BehaviorBase
{
    method Initialize()
    {
        This[parent]:Initialize["Miner"]
        ; Miner-specific init
    }

    method Set_Default_Values()
    {
        ; Call parent for common settings
        This[parent]:Set_Default_Values

        ; Add miner-specific settings
        This.ConfigRef:AddSetting[StripMine, TRUE]
        This.ConfigRef:AddSetting[CargoThreshold, 11500]
    }
}
```

### 5. Encrypt Sensitive Data

```lavish
; WARNING: LavishScript has limited encryption

member:string LoginPassword()
{
    ; Stored in plain text in XML - not secure!
    return ${This.ConfigRef.FindSetting[Login Password, ""]}
}

; Better: Use InnerSpace's secure storage
member:string LoginPassword()
{
    ; Store in InnerSpace settings, not script config
    return ${LavishScript.Settings.FindSetting[SecurePassword, ""]}
}

; Or base64 encode (obscurity, not security)
method SetLoginPassword(string password)
{
    variable string encoded = ${password.Base64Encode}
    This.ConfigRef:AddSetting[Login Password, ${encoded}]
}

member:string LoginPassword()
{
    variable string encoded = ${This.ConfigRef.FindSetting[Login Password, ""]}
    return ${encoded.Base64Decode}
}
```

### 6. Group Related Settings

```lavish
; GOOD - organized into sets
<Set Name="Combat">
    <Set Name="Tank">
        <Setting Name="MinShield">50</Setting>
        <Setting Name="MinArmor">70</Setting>
        <Setting Name="MinCap">25</Setting>
    </Set>
    <Set Name="Movement">
        <Setting Name="OrbitDistance">30000</Setting>
        <Setting Name="KeepAtRange">FALSE</Setting>
    </Set>
</Set>

; Access
variable int minShield = ${Config.Combat.Tank.MinShield}
variable int orbitDist = ${Config.Combat.Movement.OrbitDistance}
```

### 7. Document Config Files

```xml
<?xml version='1.0' encoding='UTF-8'?>
<!--
    EVEBot Configuration for: Jovehn
    Last Modified: 2024-01-15
    Bot Version: 4.0
-->
<InnerSpaceSettings>
    <Set Name="Jovehn">
        <!-- Common Settings -->
        <Set Name="Common">
            <!-- Current active behavior: Miner, Hauler, Combat, Idle -->
            <Setting Name="CurrentBehavior">Miner</Setting>

            <!-- Maximum runtime in hours (0 = unlimited) -->
            <Setting Name="Maximum Runtime">8</Setting>
        </Set>

        <!-- Combat Settings -->
        <Set Name="Combat">
            <!-- Shield threshold to flee (0-100) -->
            <Setting Name="MinimumShieldPct">50</Setting>
        </Set>
    </Set>
</InnerSpaceSettings>
```

---

## Complete Examples <a name="examples"></a>

### Example 1: Complete Config System

```lavish
/* ========== CONFIG MANAGER ========== */
objectdef obj_MyBot_ConfigManager
{
    variable string CONFIG_FILE = "${Me.Name} Config.xml"
    variable filepath CONFIG_PATH = "${Script.CurrentDirectory}/config"
    variable settingsetref ConfigRoot
    variable float ConfigVersion = 1.5

    method Initialize()
    {
        ; Setup LavishSettings
        LavishSettings[MyBotSettings]:Remove
        LavishSettings:AddSet[MyBotSettings]
        LavishSettings[MyBotSettings]:AddSet[${Me.Name}]

        ; Import existing config
        if ${CONFIG_PATH.FileExists[${CONFIG_FILE}]}
        {
            echo "Loading config: ${CONFIG_FILE}"
            LavishSettings[MyBotSettings]:Import[${CONFIG_PATH}/${CONFIG_FILE}]
        }
        else
        {
            echo "Creating new config: ${CONFIG_FILE}"
        }

        ; Set root reference
        ConfigRoot:Set[${LavishSettings[MyBotSettings].FindSet[${Me.Name}]}]

        ; Check version and migrate if needed
        variable float fileVersion = ${ConfigRoot.FindSetting[ConfigVersion, 1.0]}
        if ${fileVersion} < ${This.ConfigVersion}
        {
            This:MigrateConfig[${fileVersion}]
        }

        ; Update version
        ConfigRoot:AddSetting[ConfigVersion, ${This.ConfigVersion}]
    }

    method Shutdown()
    {
        This:Save
        LavishSettings[MyBotSettings]:Remove
    }

    method Save()
    {
        echo "Saving config to ${CONFIG_FILE}"
        LavishSettings[MyBotSettings]:Export[${CONFIG_PATH}/${CONFIG_FILE}]
    }

    method MigrateConfig(float fromVersion)
    {
        echo "Migrating config from v${fromVersion} to v${This.ConfigVersion}"

        ; Backup
        variable string backup = "${CONFIG_FILE}.v${fromVersion}.backup"
        LavishSettings[MyBotSettings]:Export[${CONFIG_PATH}/${backup}]

        ; Migration logic here
        if ${fromVersion} < 1.5
        {
            ; Add new settings, rename old ones, etc.
        }
    }
}

/* ========== BASE CONFIG CLASS ========== */
objectdef obj_MyBot_ConfigBase
{
    variable string SetName = ""

    method Initialize(string name)
    {
        SetName:Set[${name}]

        if !${ConfigManager.ConfigRoot.FindSet[${This.SetName}](exists)}
        {
            echo "Creating config set: ${This.SetName}"
            ConfigManager.ConfigRoot:AddSet[${This.SetName}]
            This:Set_Default_Values
        }
    }

    member:settingsetref ConfigRef()
    {
        return ${ConfigManager.ConfigRoot.FindSet[${This.SetName}]}
    }

    method Set_Default_Values()
    {
        ; Override in derived classes
    }
}

/* ========== GENERAL CONFIG ========== */
objectdef obj_MyBot_Config_General inherits obj_MyBot_ConfigBase
{
    method Initialize()
    {
        This[parent]:Initialize["General"]
    }

    method Set_Default_Values()
    {
        This.ConfigRef:AddSetting[BotMode, "Idle"]
        This.ConfigRef:AddSetting[HomeStation, "Jita 4-4"]
        This.ConfigRef:AddSetting[AutoStart, FALSE]
        This.ConfigRef:AddSetting[MaxRuntime, 0]
    }

    member:string BotMode()
    {
        return ${This.ConfigRef.FindSetting[BotMode, "Idle"]}
    }

    method SetBotMode(string mode)
    {
        This.ConfigRef:AddSetting[BotMode, ${mode}]
        ConfigManager:Save
    }

    member:string HomeStation()
    {
        return ${This.ConfigRef.FindSetting[HomeStation, "Jita 4-4"]}
    }

    method SetHomeStation(string station)
    {
        This.ConfigRef:AddSetting[HomeStation, ${station}]
    }

    member:bool AutoStart()
    {
        return ${This.ConfigRef.FindSetting[AutoStart, FALSE]}
    }

    method SetAutoStart(bool value)
    {
        This.ConfigRef:AddSetting[AutoStart, ${value}]
    }

    member:int MaxRuntime()
    {
        return ${This.ConfigRef.FindSetting[MaxRuntime, 0]}
    }

    method SetMaxRuntime(int hours)
    {
        if ${hours} < 0
        {
            echo "ERROR: Invalid runtime ${hours}"
            return
        }
        This.ConfigRef:AddSetting[MaxRuntime, ${hours}]
    }
}

/* ========== COMBAT CONFIG ========== */
objectdef obj_MyBot_Config_Combat inherits obj_MyBot_ConfigBase
{
    method Initialize()
    {
        This[parent]:Initialize["Combat"]
    }

    method Set_Default_Values()
    {
        This.ConfigRef:AddSetting[MinShield, 50]
        This.ConfigRef:AddSetting[MinArmor, 70]
        This.ConfigRef:AddSetting[MinCap, 25]
        This.ConfigRef:AddSetting[OrbitDistance, 30000]
        This.ConfigRef:AddSetting[UseDrones, TRUE]
    }

    member:int MinShield()
    {
        return ${This.ConfigRef.FindSetting[MinShield, 50]}
    }

    method SetMinShield(int value)
    {
        if ${value} < 0 || ${value} > 100
        {
            echo "ERROR: Shield pct must be 0-100"
            return
        }
        This.ConfigRef:AddSetting[MinShield, ${value}]
    }

    member:int MinArmor()
    {
        return ${This.ConfigRef.FindSetting[MinArmor, 70]}
    }

    method SetMinArmor(int value)
    {
        if ${value} < 0 || ${value} > 100
        {
            echo "ERROR: Armor pct must be 0-100"
            return
        }
        This.ConfigRef:AddSetting[MinArmor, ${value}]
    }

    member:int MinCap()
    {
        return ${This.ConfigRef.FindSetting[MinCap, 25]}
    }

    method SetMinCap(int value)
    {
        if ${value} < 0 || ${value} > 100
        {
            echo "ERROR: Cap pct must be 0-100"
            return
        }
        This.ConfigRef:AddSetting[MinCap, ${value}]
    }

    member:int OrbitDistance()
    {
        return ${This.ConfigRef.FindSetting[OrbitDistance, 30000]}
    }

    method SetOrbitDistance(int distance)
    {
        This.ConfigRef:AddSetting[OrbitDistance, ${distance}]
    }

    member:bool UseDrones()
    {
        return ${This.ConfigRef.FindSetting[UseDrones, TRUE]}
    }

    method SetUseDrones(bool value)
    {
        This.ConfigRef:AddSetting[UseDrones, ${value}]
    }
}

/* ========== MAIN CONFIG OBJECT ========== */
objectdef obj_MyBot_Config
{
    variable obj_MyBot_Config_General General
    variable obj_MyBot_Config_Combat Combat

    method Save()
    {
        ConfigManager:Save
    }
}

/* ========== USAGE ========== */
variable(global) obj_MyBot_ConfigManager ConfigManager
variable(global) obj_MyBot_Config Config

function main()
{
    ; Initialize config system
    ConfigManager:Initialize

    ; Access config
    echo "Bot Mode: ${Config.General.BotMode}"
    echo "Home Station: ${Config.General.HomeStation}"
    echo "Min Shield: ${Config.Combat.MinShield}%"

    ; Change config
    Config.General:SetBotMode["Combat"]
    Config.Combat:SetMinShield[60]

    ; Save
    Config:Save

    ; Shutdown
    ConfigManager:Shutdown
}
```

### Example 2: UI-Driven Config

```lavish
/* ========== CONFIG UI ========== */
objectdef obj_ConfigUI
{
    method Initialize()
    {
        This:CreateUI
    }

    method CreateUI()
    {
        ; Create config window
        ui -load config_ui.xml
    }

    ; Called when behavior dropdown changes
    method OnBehaviorChanged()
    {
        variable string newBehavior = ${UIElement[BehaviorCombo].SelectedItem.Value}
        Config.General:SetBotMode[${newBehavior}]
        echo "Behavior changed to: ${newBehavior}"
    }

    ; Called when min shield slider changes
    method OnMinShieldChanged()
    {
        variable int newValue = ${UIElement[MinShieldSlider].Value}
        Config.Combat:SetMinShield[${newValue}]
        UIElement[MinShieldText]:SetText["${newValue}%"]
    }

    ; Called when Save button clicked
    method OnSaveClicked()
    {
        Config:Save
        echo "Configuration saved!"
        UIElement[SaveButton]:SetText["Saved!"]
        wait 20
        UIElement[SaveButton]:SetText["Save Config"]
    }

    ; Update UI from config (called periodically)
    method UpdateUIFromConfig()
    {
        UIElement[BehaviorCombo]:SelectItem[${Config.General.BotMode}]
        UIElement[MinShieldSlider]:SetValue[${Config.Combat.MinShield}]
        UIElement[MinShieldText]:SetText["${Config.Combat.MinShield}%"]
        UIElement[AutoStartCheckbox]:SetChecked[${Config.General.AutoStart}]
    }
}
```

**config_ui.xml:**

```xml
<?xml version='1.0' encoding='UTF-8'?>
<LGUI2>
    <window name='ConfigWindow' width='400' height='600'>
        <label>Bot Configuration</label>

        <!-- Behavior Selection -->
        <label y='50'>Behavior:</label>
        <combobox name='BehaviorCombo' x='100' y='50' width='200'>
            <item value='Idle'>Idle</item>
            <item value='Mining'>Mining</item>
            <item value='Combat'>Combat</item>
            <item value='Hauling'>Hauling</item>
            <onselect>ConfigUI:OnBehaviorChanged</onselect>
        </combobox>

        <!-- Min Shield Slider -->
        <label y='100'>Min Shield %:</label>
        <slider name='MinShieldSlider' x='100' y='100' width='200' min='0' max='100'>
            <onchange>ConfigUI:OnMinShieldChanged</onchange>
        </slider>
        <label name='MinShieldText' x='310' y='100'>50%</label>

        <!-- Auto Start Checkbox -->
        <checkbox name='AutoStartCheckbox' x='10' y='150'>
            <label>Auto Start</label>
            <onclick>Config.General:SetAutoStart[${This.Checked}]</onclick>
        </checkbox>

        <!-- Save Button -->
        <button name='SaveButton' x='150' y='550' width='100'>
            <label>Save Config</label>
            <onclick>ConfigUI:OnSaveClicked</onclick>
        </button>
    </window>
</LGUI2>
```

### Example 3: Fleet Config Sync

```lavish
/* ========== FLEET CONFIG SYNC ========== */
objectdef obj_FleetConfigSync
{
    variable bool IsMaster = FALSE
    variable string MasterName

    method Initialize()
    {
        This.IsMaster:Set[${Config.Fleet.IsMaster}]
        This.MasterName:Set["${Config.Fleet.MasterName}"]

        ; Register events
        LavishScript:RegisterEvent[Fleet_Config_Sync]
        Event[Fleet_Config_Sync]:AttachAtom[This:OnConfigSync]
    }

    ; Master: Sync config to all slaves
    method SyncToFleet()
    {
        if !${This.IsMaster}
        {
            echo "ERROR: Only master can sync config"
            return
        }

        echo "Syncing config to fleet..."

        ; Build config string: category|setting|value
        variable string configData = ""

        ; General
        configData:Concat["General|BotMode|${Config.General.BotMode};"]
        configData:Concat["General|MaxRuntime|${Config.General.MaxRuntime};"]

        ; Combat
        configData:Concat["Combat|MinShield|${Config.Combat.MinShield};"]
        configData:Concat["Combat|MinArmor|${Config.Combat.MinArmor};"]
        configData:Concat["Combat|OrbitDistance|${Config.Combat.OrbitDistance};"]

        ; Broadcast
        relay "all other" -event Fleet_Config_Sync "${configData}"

        echo "Config sync complete"
    }

    ; Slave: Receive and apply config
    atom OnConfigSync(string configData)
    {
        if ${This.IsMaster}
            return

        echo "Receiving config from master..."

        ; Parse config string
        variable int i = 1
        variable string configItem

        while ${configData.Token[${i}, ";"](exists)}
        {
            configItem:Set["${configData.Token[${i}, ";"]}"]

            variable string category = ${configItem.Token[1, "|"]}
            variable string setting = ${configItem.Token[2, "|"]}
            variable string value = ${configItem.Token[3, "|"]}

            This:ApplyConfigItem["${category}", "${setting}", "${value}"]

            i:Inc
        }

        ; Save applied config
        Config:Save

        echo "Config sync applied"
    }

    method ApplyConfigItem(string category, string setting, string value)
    {
        switch ${category}
        {
            case General
                switch ${setting}
                {
                    case BotMode
                        Config.General:SetBotMode["${value}"]
                        break
                    case MaxRuntime
                        Config.General:SetMaxRuntime[${value}]
                        break
                }
                break

            case Combat
                switch ${setting}
                {
                    case MinShield
                        Config.Combat:SetMinShield[${value}]
                        break
                    case MinArmor
                        Config.Combat:SetMinArmor[${value}]
                        break
                    case OrbitDistance
                        Config.Combat:SetOrbitDistance[${value}]
                        break
                }
                break
        }

        echo "Applied: ${category}.${setting} = ${value}"
    }
}
```

---

## Summary

### Key Takeaways

1. **LavishSettings Foundation**
   - Hierarchical Sets and Settings
   - Import/Export XML
   - settingsetref for efficient access

2. **Config Architecture Patterns**
   - BaseConfig for file I/O
   - Category-based sub-configs
   - Member/method pairs for access

3. **Configuration Best Practices**
   - Always provide defaults
   - Validate values
   - Version and migrate
   - Per-character files

4. **UI Integration**
   - Bind config to UI elements
   - Real-time updates
   - Save on critical changes

5. **Fleet Coordination**
   - Broadcast config changes via relay
   - Master syncs to slaves
   - Validate before applying

### What's Next?

**File 30:** Debugging Techniques
- Logging frameworks
- Debug output levels
- Error handling patterns
- Performance profiling
- Common debugging tools
- Troubleshooting guide

### Navigation

[← Previous: File 28 - Relay System and IPC](28_Relay_System_and_IPC.md) | [Next: File 30 - Debugging Techniques →](30_Debugging_Techniques.md)

---

*Layer 7 Progress: 2/4 Complete (50%)*
*Total Documentation Progress: 28/38 Files (74%)*
