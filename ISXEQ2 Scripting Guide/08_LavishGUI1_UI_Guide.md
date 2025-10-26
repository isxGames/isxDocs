# LavishGUI 1 - UI Creation Guide for ISXEQ2

Complete guide to creating custom user interfaces with LavishGUI 1 XML for ISXEQ2 scripts.

**Note:** This guide covers **LavishGUI 1**, the XML-based UI system used by older ISXEQ2 scripts like EQ2Bot. Newer scripts may use LavishGUI 2, which is a different system.

**Official Documentation:** https://www.lavishsoft.com/wiki/index.php/LavishGUI

---

## Table of Contents

1. [Introduction to LavishGUI 1](#introduction-to-lavishgui-1)
2. [Basic XML Structure](#basic-xml-structure)
3. [Core UI Elements](#core-ui-elements)
4. [Event Handling](#event-handling)
5. [Script-to-UI Interaction](#script-to-ui-interaction)
6. [Templates and Skinning](#templates-and-skinning)
7. [Complete Working Examples](#complete-working-examples)
8. [Best Practices](#best-practices)
9. [Troubleshooting](#troubleshooting)

---

## Introduction to LavishGUI 1

### What is LavishGUI 1?

**LavishGUI 1** is InnerSpace's original XML-based user interface system. It allows you to:
- Create custom windows and dialogs
- Build interactive controls (buttons, checkboxes, dropdowns, etc.)
- Interface between your LavishScript code and visual elements
- Save/restore window positions and settings
- Create themed UIs with skins and templates

### How LavishGUI 1 Works with Scripts

1. **XML Files** define the UI structure and appearance
2. **Events** in XML execute LavishScript code (OnClick, OnLoad, etc.)
3. **Script Code** can access and modify UI elements dynamically
4. **Two-way Communication** between UI and script:
   - UI → Script: Events trigger script functions
   - Script → UI: Code modifies UI element properties

### File Locations and Naming Conventions

**UI XML Files:**
```
Scripts\<ScriptName>\UI\<UIFile>.xml
```

**Example (EQ2Bot):**
```
Scripts\EQ2Bot\UI\EQ2Bot.xml         (main window)
Scripts\EQ2Bot\UI\EQ2BotExtras.xml   (additional UI)
Scripts\EQ2Bot\UI\EQ2Skin.xml        (skin/theme)
```

**Loading UI from Script:**
```lavishscript
ui -load "${LavishScript.HomeDirectory}/Scripts/EQ2Bot/UI/EQ2Bot.xml"
ui -reload "${LavishScript.HomeDirectory}/Scripts/EQ2Bot/UI/EQ2Bot.xml"
```

---

## Basic XML Structure

### Root <ISUI> Element

Every LavishGUI 1 XML file starts with:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<ISUI>
  <!-- Your UI elements go here -->
</ISUI>
```

### Basic Window Structure

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<ISUI>
  <Window name='My Window'>
    <X>100</X>
    <Y>100</Y>
    <Width>400</Width>
    <Height>300</Height>
    <Title>My Window Title</Title>
    <StorePosition>1</StorePosition>

    <TitleBar template='window.Titlebar'>
      <Children>
        <Text Name='Title' Template='window.Titlebar.Title' />
        <Button Name='Close' Template='window.Titlebar.Close' />
      </Children>
    </TitleBar>

    <Children>
      <!-- Your UI elements go here -->
    </Children>
  </Window>
</ISUI>
```

### Element Hierarchy

- **Window** - Top-level container
  - **TitleBar** - Window title bar
    - **Text** - Title text
    - **Button** - Close/minimize buttons
  - **Children** - Container for UI elements
    - **Tab Control, Buttons, Checkboxes, etc.**

---

## Core UI Elements

### Window

The top-level container for your UI.

#### Basic Window Properties

```xml
<Window name='MyWindow'>
  <!-- Position (pixels or percentages) -->
  <X>10</X>           <!-- Left position -->
  <Y>10</Y>           <!-- Top position -->

  <!-- Size -->
  <Width>500</Width>
  <Height>400</Height>

  <!-- Title -->
  <Title>My Window</Title>

  <!-- Save/restore window position automatically -->
  <StorePosition>1</StorePosition>

  <!-- Visibility -->
  <Visible>1</Visible>  <!-- 1 = visible, 0 = hidden -->
</Window>
```

#### Window Events

```xml
<Window name='MyWindow'>
  <!-- Called when window first loads -->
  <OnLoad>
    echo Window loaded
  </OnLoad>

  <!-- Called when window unloads -->
  <OnUnload>
    echo Window closing
  </OnUnload>

  <!-- Called when window is clicked -->
  <OnLeftClick>
    This:SetFocus
  </OnLeftClick>

  <!-- Called every frame while window is visible -->
  <OnRender>
    if ${UIElement[MyWindow].Minimized}
    {
      ; Do something when minimized
    }
  </OnRender>
</Window>
```

### Buttons and CommandButtons

#### Basic Button

```xml
<button Name='MyButton'>
  <X>10</X>
  <Y>10</Y>
  <Width>100</Width>
  <Height>25</Height>
  <Text>Click Me</Text>

  <OnLeftClick>
    echo Button clicked!
  </OnLeftClick>
</button>
```

#### CommandButton (Executes Script Commands)

```xml
<commandbutton name='StartBot'>
  <X>10</X>
  <Y>40</Y>
  <Width>100</Width>
  <Height>25</Height>
  <Text>Start Bot</Text>
  <AutoTooltip>Click to start the bot</AutoTooltip>

  <OnLeftClick>
    Script[EQ2Bot]:QueueCommand[call StartBot]
  </OnLeftClick>
</commandbutton>
```

**Note:** The difference:
- **button** - Generic button, can execute any code
- **commandbutton** - Specialized for executing script commands
- Use `Script[scriptname]:QueueCommand[command]` to call script functions

### Checkboxes

Checkboxes for boolean options.

#### Basic Checkbox

```xml
<checkbox Name='AutoLoot'>
  <X>10</X>
  <Y>70</Y>
  <Width>150</Width>
  <Height>20</Height>
  <Text>Enable Auto Loot</Text>

  <OnLoad>
    ; Set checked state from script variable
    if ${Script[MyScript].Variable[AutoLootEnabled]}
    {
      This:SetChecked
    }
  </OnLoad>

  <OnLeftClick>
    if ${This.Checked}
    {
      ; Checkbox was just checked
      Script[MyScript].Variable[AutoLootEnabled]:Set[TRUE]
      echo Auto loot enabled
    }
    else
    {
      ; Checkbox was just unchecked
      Script[MyScript].Variable[AutoLootEnabled]:Set[FALSE]
      echo Auto loot disabled
    }
  </OnLeftClick>
</checkbox>
```

#### CommandCheckbox (with LavishSettings Integration)

```xml
<commandcheckbox Name='MainTank'>
  <X>10</X>
  <Y>100</Y>
  <Width>150</Width>
  <Height>20</Height>
  <Text>I am Main Tank</Text>
  <AutoTooltip>Check if you are the main tank</AutoTooltip>

  <OnLeftClick>
    if ${This.Checked}
    {
      Script[EQ2Bot].Variable[MainTank]:Set[TRUE]
      LavishSettings[EQ2Bot].FindSet[Character].FindSet[Settings]:AddSetting[MainTank,TRUE]
      Script[EQ2Bot].VariableScope.EQ2Bot:Save_Settings
    }
    else
    {
      Script[EQ2Bot].Variable[MainTank]:Set[FALSE]
      LavishSettings[EQ2Bot].FindSet[Character].FindSet[Settings]:AddSetting[MainTank,FALSE]
      Script[EQ2Bot].VariableScope.EQ2Bot:Save_Settings
    }
  </OnLeftClick>

  <!-- Load checked state from settings -->
  <Data>${LavishSettings[EQ2Bot].FindSet[Character].FindSet[Settings].FindSetting[MainTank]}</Data>
</commandcheckbox>
```

### ComboBoxes (Dropdowns)

Dropdown selection lists.

#### Basic ComboBox

```xml
<combobox name='ClassSelection'>
  <X>10</X>
  <Y>130</Y>
  <Width>150</Width>
  <Height>20</Height>
  <AutoTooltip>Select your class</AutoTooltip>

  <!-- Initial items -->
  <Items>
    <Item>Warrior</Item>
    <Item>Mage</Item>
    <Item>Priest</Item>
  </Items>

  <!-- Sort items alphabetically -->
  <Sort>Text</Sort>

  <OnLoad>
    ; Select an item by text
    This.ItemByText[Warrior]:Select
  </OnLoad>

  <OnSelect>
    ; Called when user selects an item
    echo Selected: ${This.SelectedItem.Text}
    Script[MyScript].Variable[SelectedClass]:Set[${This.SelectedItem.Text}]
  </OnSelect>
</combobox>
```

#### Dynamic ComboBox (Populated from Game Data)

```xml
<combobox name='MainTankSelection'>
  <X>10</X>
  <Y>160</Y>
  <Width>150</Width>
  <Height>20</Height>
  <Items></Items>  <!-- Empty initially -->

  <OnLoad>
    ; Load saved selection
    if ${Actor[exactname,${Script[EQ2Bot].Variable[MainTankPC]}].Name(exists)}
    {
      This:AddItem[${Script[EQ2Bot].Variable[MainTankPC]}]
      This.ItemByText[${Script[EQ2Bot].Variable[MainTankPC]}]:Select
    }
  </OnLoad>

  <OnSelect>
    ; Save selection
    Script[EQ2Bot].Variable[MainTankPC]:Set[${This.SelectedItem.Text}]
    LavishSettings[EQ2Bot].FindSet[Character].FindSet[Settings]:AddSetting[MainTank,${This.SelectedItem.Text}]
    Script[EQ2Bot].VariableScope.EQ2Bot:Save_Settings
  </OnSelect>

  <OnLeftClick>
    ; Populate with group members when clicked
    declare tmpvar int
    This:ClearItems
    tmpvar:Set[1]

    if ${Me.Group} > 1
    {
      do
      {
        if ${Me.Group[${tmpvar}].InZone}
        {
          This:AddItem[${Me.Group[${tmpvar}].Name}]
        }
      }
      while ${tmpvar:Inc} < ${Me.Group}
    }

    This:Sort
  </OnLeftClick>
</combobox>
```

**Key ComboBox Methods:**
- `:AddItem[text]` - Add an item
- `:ClearItems` - Remove all items
- `:Sort` - Sort items alphabetically
- `.SelectedItem.Text` - Get selected item text
- `.ItemByText[text]:Select` - Select an item by text

### Text and Labels

Display static or dynamic text.

```xml
<text Name='StatusText'>
  <X>10</X>
  <Y>190</Y>
  <Width>300</Width>
  <Height>20</Height>
  <Text>Status: Idle</Text>
  <Alignment>Left</Alignment>
  <Color>FFFFFFFF</Color>  <!-- ARGB color -->
</text>
```

**Update text from script:**
```lavishscript
UIElement[StatusText]:SetText["Status: Running"]
```

### TextBoxes (Text Input)

Allow user text input.

```xml
<textentry Name='CharacterName'>
  <X>10</X>
  <Y>220</Y>
  <Width>200</Width>
  <Height>20</Height>
  <Text></Text>

  <OnLoad>
    ; Load saved value
    This:SetText[${LavishSettings[MyScript].FindSetting[CharName]}]
  </OnLoad>
</textentry>
```

**Read text from script:**
```lavishscript
variable string CharName
CharName:Set[${UIElement[CharacterName].Text}]
```

### Tabs and TabControls

Create tabbed interfaces.

```xml
<tabcontrol Name='MainTabs'>
  <X>5</X>
  <Y>5</Y>
  <Width>95%</Width>
  <Height>95%</Height>

  <Tabs>
    <!-- Tab 1 -->
    <Tab Name='General'>
      <checkbox Name='Option1'>
        <X>10</X>
        <Y>10</Y>
        <Width>150</Width>
        <Height>20</Height>
        <Text>Option 1</Text>
      </checkbox>

      <checkbox Name='Option2'>
        <X>10</X>
        <Y>35</Y>
        <Width>150</Width>
        <Height>20</Height>
        <Text>Option 2</Text>
      </checkbox>
    </Tab>

    <!-- Tab 2 -->
    <Tab Name='Advanced'>
      <text Name='AdvancedLabel'>
        <X>10</X>
        <Y>10</Y>
        <Text>Advanced Options</Text>
      </text>
    </Tab>
  </Tabs>
</tabcontrol>
```

**Switch tabs from script:**
```lavishscript
UIElement[MainTabs].Tab[Advanced]:Select
```

### Frames

Container elements for grouping UI elements.

```xml
<Frame Name='OptionsFrame'>
  <X>10</X>
  <Y>250</Y>
  <Height>100</Height>
  <Width>300</Width>

  <Children>
    <checkbox Name='Option1'>
      <X>5</X>
      <Y>5</Y>
      <Width>150</Width>
      <Height>20</Height>
      <Text>Option 1</Text>
    </checkbox>

    <checkbox Name='Option2'>
      <X>5</X>
      <Y>30</Y>
      <Width>150</Width>
      <Height>20</Height>
      <Text>Option 2</Text>
    </checkbox>
  </Children>

  <OnLoad>
    ; Show or hide frame based on condition
    if ${Script[MyScript].Variable[ShowOptions]}
    {
      This:Show
    }
    else
    {
      This:Hide
    }
  </OnLoad>
</Frame>
```

### Lists and ListBoxes

Display scrollable lists of items.

```xml
<listbox Name='TargetList'>
  <X>10</X>
  <Y>280</Y>
  <Width>200</Width>
  <Height>100</Height>
  <Items></Items>

  <OnLoad>
    ; Populate with targets
    declare i int
    i:Set[1]
    do
    {
      if ${Actor[${i}](exists)}
      {
        This:AddItem[${Actor[${i}].Name}]
      }
    }
    while ${i:Inc} <= 50
  </OnLoad>

  <OnSelect>
    echo Selected: ${This.SelectedItem.Text}
  </OnSelect>
</listbox>
```

---

## Event Handling

### Common Events

#### OnLoad
Called when the element is first created/loaded.

```xml
<checkbox Name='AutoAttack'>
  <OnLoad>
    ; Initialize from saved settings
    if ${LavishSettings[MyScript].FindSetting[AutoAttack]}
    {
      This:SetChecked
    }
  </OnLoad>
</checkbox>
```

#### OnLeftClick
Called when the element is left-clicked.

```xml
<button Name='AttackButton'>
  <OnLeftClick>
    Script[MyScript]:QueueCommand[call Attack]
  </OnLeftClick>
</button>
```

#### OnRightClick
Called when the element is right-clicked.

```xml
<button Name='ContextButton'>
  <OnRightClick>
    echo Right-clicked!
  </OnRightClick>
</button>
```

#### OnSelect
Called when an item is selected (combobox, listbox, tab).

```xml
<combobox name='TargetSelection'>
  <OnSelect>
    Script[MyScript].Variable[SelectedTarget]:Set[${This.SelectedItem.Text}]
  </OnSelect>
</combobox>
```

#### OnRender
Called every frame while the element is visible.

```xml
<Window name='MyWindow'>
  <OnRender>
    ; Update title dynamically
    UIElement[Title@TitleBar@MyWindow]:SetText[MyScript - ${Me.Name}]
  </OnRender>
</Window>
```

**Warning:** OnRender executes every frame (~60 times per second). Avoid heavy processing here!

#### OnUnload
Called when the element is being destroyed.

```xml
<Window name='MyWindow'>
  <OnUnload>
    echo Window closing - save state here
  </OnUnload>
</Window>
```

### Accessing Script Variables from UI

Access script variables using the `Script[]` TLO:

```xml
<OnLoad>
  if ${Script[EQ2Bot].Variable[MainTank]}
  {
    This:SetChecked
  }
</OnLoad>
```

**Syntax:**
- `${Script[scriptname].Variable[variablename]}` - Get variable value
- `Script[scriptname].Variable[variablename]:Set[value]` - Set variable value
- `${Script[scriptname].VariableScope.objectname}` - Access object/atom

### Calling Script Functions from UI

Use `Script:QueueCommand` to execute script code:

```xml
<commandbutton name='StartBot'>
  <OnLeftClick>
    Script[EQ2Bot]:QueueCommand[call StartBot]
  </OnLeftClick>
</commandbutton>
```

**Or execute directly in events:**
```xml
<checkbox Name='AutoLoot'>
  <OnLeftClick>
    if ${This.Checked}
    {
      Script[MyScript]:QueueCommand[call EnableAutoLoot]
    }
    else
    {
      Script[MyScript]:QueueCommand[call DisableAutoLoot]
    }
  </OnLeftClick>
</checkbox>
```

---

## Script-to-UI Interaction

### Finding UI Elements

From your script, access UI elements using `UIElement[]`:

```lavishscript
; Access by name
UIElement[MyWindow]

; Access nested elements (child@parent@grandparent)
UIElement[Title@TitleBar@MyWindow]

; Using FindChild
UIElement[MyWindow].FindChild[TitleBar].FindChild[Title]

; Using FindUsableChild (searches all descendants)
UIElement[MyWindow].FindUsableChild[SomeDeepElement,checkbox]
```

### Modifying UI Elements from Script

#### SetText - Update text

```lavishscript
UIElement[StatusLabel]:SetText["Status: Running"]
UIElement[Title@TitleBar@MyWindow]:SetText["MyScript - ${Me.Name}"]
```

#### SetChecked / UnCheck - Checkboxes

```lavishscript
UIElement[AutoLoot]:SetChecked
UIElement[AutoLoot]:UnCheck
```

#### Show / Hide - Visibility

```lavishscript
UIElement[AdvancedFrame]:Show
UIElement[AdvancedFrame]:Hide
```

#### SetHeight / SetWidth - Resize

```lavishscript
UIElement[MyWindow]:SetHeight[400]
UIElement[MyWindow]:SetWidth[600]
```

#### AddItem / ClearItems - ComboBoxes and Lists

```lavishscript
; Clear all items
UIElement[TargetList]:ClearItems

; Add items
UIElement[TargetList]:AddItem["Target 1"]
UIElement[TargetList]:AddItem["Target 2"]

; Sort items
UIElement[TargetList]:Sort
```

### Reading UI Element State

```lavishscript
; Check if checkbox is checked
if ${UIElement[AutoLoot].Checked}
{
    echo Auto loot is enabled
}

; Get selected item from combobox
variable string SelectedTarget
SelectedTarget:Set[${UIElement[TargetSelection].SelectedItem.Text}]

; Check if element is visible
if ${UIElement[MyWindow].Visible}
{
    echo Window is visible
}

; Check if window is minimized
if ${UIElement[MyWindow].Minimized}
{
    echo Window is minimized
}
```

---

## Templates and Skinning

### What are Templates?

Templates allow you to:
- Define reusable UI component styles
- Create consistent look and feel
- Separate appearance from structure
- Make themed UIs (skins)

### Using Templates

Reference a template in your element:

```xml
<button Name='MyButton' Template='EQ2.button'>
  <Text>Click Me</Text>
</button>
```

### Creating a Skin

**EQ2Skin.xml example:**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<ISUI>
  <Skin Name='eq2skin' Template='Default Skin'>
    <SkinTemplate Base='window' Skin='EQ2.window' />
    <SkinTemplate Base='button' Skin='EQ2.button' />
    <SkinTemplate Base='checkbox' Skin='EQ2.checkbox' />
    <SkinTemplate Base='combobox' Skin='EQ2.combobox' />
  </Skin>

  <!-- Window Template -->
  <template name='EQ2.window'>
    <Border>1</Border>
    <Resizable>1</Resizable>
    <StorePosition>1</StorePosition>
  </template>

  <!-- Button Template -->
  <template name='EQ2.button'>
    <Width>100</Width>
    <Height>25</Height>
  </template>
</ISUI>
```

### Using Textures in Templates

```xml
<template name='EQ2.window.Texture' Filename='windowelements.dds' ColorKey='00000000'>
  <Left>14</Left>
  <Right>414</Right>
  <Top>529</Top>
  <Bottom>893</Bottom>
</template>
```

---

## Complete Working Examples

### Example 1: Simple Status Window

**UI XML (MyScript_UI.xml):**
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<ISUI>
  <Window name='MyScript Window'>
    <X>100</X>
    <Y>100</Y>
    <Width>300</Width>
    <Height>150</Height>
    <Title>My Script</Title>
    <StorePosition>1</StorePosition>

    <TitleBar template='window.Titlebar'>
      <Children>
        <Text Name='Title' Template='window.Titlebar.Title' />
        <commandbutton name='Close' Template='window.Titlebar.Close'>
          <Command>Script[MyScript]:End</Command>
        </commandbutton>
      </Children>
    </TitleBar>

    <Children>
      <text Name='Status'>
        <X>10</X>
        <Y>10</Y>
        <Width>280</Width>
        <Height>20</Height>
        <Text>Status: Stopped</Text>
      </text>

      <commandbutton name='StartButton'>
        <X>10</X>
        <Y>40</Y>
        <Width>100</Width>
        <Height>25</Height>
        <Text>Start</Text>
        <OnLeftClick>
          Script[MyScript]:QueueCommand[call Start]
        </OnLeftClick>
      </commandbutton>

      <commandbutton name='StopButton'>
        <X>120</X>
        <Y>40</Y>
        <Width>100</Width>
        <Height>25</Height>
        <Text>Stop</Text>
        <OnLeftClick>
          Script[MyScript]:QueueCommand[call Stop]
        </OnLeftClick>
      </commandbutton>
    </Children>
  </Window>
</ISUI>
```

**Script Code (MyScript.iss):**
```lavishscript
variable bool Running = FALSE

function main()
{
    ; Load UI
    ui -load "${LavishScript.HomeDirectory}/Scripts/MyScript/UI/MyScript_UI.xml"

    ; Wait for script to end
    do
    {
        wait 10
    }
    while ${Script.Running}

    ; Cleanup
    ui -unload "${LavishScript.HomeDirectory}/Scripts/MyScript/UI/MyScript_UI.xml"
}

function Start()
{
    Running:Set[TRUE]
    UIElement[Status@MyScript Window]:SetText["Status: Running"]
    echo Script started
}

function Stop()
{
    Running:Set[FALSE]
    UIElement[Status@MyScript Window]:SetText["Status: Stopped"]
    echo Script stopped
}
```

### Example 2: Checkbox with Settings Persistence

**UI XML:**
```xml
<checkbox Name='AutoLoot'>
  <X>10</X>
  <Y>70</Y>
  <Width>200</Width>
  <Height>20</Height>
  <Text>Enable Auto Loot</Text>

  <OnLoad>
    if ${LavishSettings[MyScript].FindSetting[AutoLoot,FALSE]}
    {
      This:SetChecked
    }
  </OnLoad>

  <OnLeftClick>
    if ${This.Checked}
    {
      Script[MyScript].Variable[AutoLoot]:Set[TRUE]
      LavishSettings[MyScript]:AddSetting[AutoLoot,TRUE]
      LavishSettings[MyScript]:Export["${LavishScript.HomeDirectory}/Scripts/MyScript/MyScript.xml"]
    }
    else
    {
      Script[MyScript].Variable[AutoLoot]:Set[FALSE]
      LavishSettings[MyScript]:AddSetting[AutoLoot,FALSE]
      LavishSettings[MyScript]:Export["${LavishScript.HomeDirectory}/Scripts/MyScript/MyScript.xml"]
    }
  </OnLeftClick>
</checkbox>
```

**Script Code:**
```lavishscript
variable bool AutoLoot = FALSE

function main()
{
    ; Load settings
    LavishSettings:AddSet[MyScript]
    LavishSettings[MyScript]:Import["${LavishScript.HomeDirectory}/Scripts/MyScript/MyScript.xml"]

    ; Get saved value
    AutoLoot:Set[${LavishSettings[MyScript].FindSetting[AutoLoot,FALSE]}]

    ; Load UI
    ui -load "${LavishScript.HomeDirectory}/Scripts/MyScript/UI/MyScript_UI.xml"

    ; Main loop
    do
    {
        if ${AutoLoot}
        {
            call CheckForLoot
        }
        wait 10
    }
    while ${Script.Running}
}
```

### Example 3: Dynamic ComboBox (Group Members)

**UI XML:**
```xml
<combobox name='FollowTarget'>
  <X>10</X>
  <Y>100</Y>
  <Width>200</Width>
  <Height>20</Height>
  <AutoTooltip>Select who to follow</AutoTooltip>
  <Items></Items>

  <OnLoad>
    ; Load saved target
    variable string SavedTarget
    SavedTarget:Set[${LavishSettings[MyScript].FindSetting[FollowTarget]}]
    if ${SavedTarget.Length} > 0
    {
      This:AddItem[${SavedTarget}]
      This.ItemByText[${SavedTarget}]:Select
    }
  </OnLoad>

  <OnSelect>
    ; Save selection
    Script[MyScript].Variable[FollowTarget]:Set[${This.SelectedItem.Text}]
    LavishSettings[MyScript]:AddSetting[FollowTarget,${This.SelectedItem.Text}]
    LavishSettings[MyScript]:Export["${LavishScript.HomeDirectory}/Scripts/MyScript/MyScript.xml"]
  </OnSelect>

  <OnLeftClick>
    ; Populate with group members when clicked
    declare tmpvar int
    This:ClearItems
    tmpvar:Set[1]

    if ${Me.Group} > 1
    {
      do
      {
        if ${Me.Group[${tmpvar}].InZone} &amp;&amp; !${Me.Group[${tmpvar}].Name.Equal[${Me.Name}]}
        {
          This:AddItem[${Me.Group[${tmpvar}].Name}]
        }
      }
      while ${tmpvar:Inc} &lt; ${Me.Group}
    }

    This:Sort
  </OnLeftClick>
</combobox>
```

**Note:** In XML, use `&amp;` for `&&` and `&lt;` for `<`

**Script Code:**
```lavishscript
variable string FollowTarget

function main()
{
    ; Load settings and UI
    LavishSettings:AddSet[MyScript]
    LavishSettings[MyScript]:Import["${LavishScript.HomeDirectory}/Scripts/MyScript/MyScript.xml"]
    FollowTarget:Set[${LavishSettings[MyScript].FindSetting[FollowTarget]}]

    ui -load "${LavishScript.HomeDirectory}/Scripts/MyScript/UI/MyScript_UI.xml"

    ; Main loop
    do
    {
        if ${FollowTarget.Length} > 0
        {
            call FollowCharacter "${FollowTarget}"
        }
        wait 10
    }
    while ${Script.Running}
}
```

---

## Best Practices

### Organization

1. **Separate UI from Logic**
   - Put UI XML in separate `UI/` folder
   - Keep script logic in .iss files
   - Use events to bridge UI and logic

2. **Use Descriptive Names**
   ```xml
   <!-- Good -->
   <checkbox Name='EnableAutoLoot'>

   <!-- Bad -->
   <checkbox Name='cb1'>
   ```

3. **Group Related Elements**
   - Use Frames to group related controls
   - Use TabControls for multiple pages
   - Keep related settings together

### Performance

1. **Minimize OnRender Usage**
   - OnRender fires every frame (~60 FPS)
   - Avoid heavy calculations in OnRender
   - Update text only when needed

   ```xml
   <!-- Bad - updates every frame -->
   <OnRender>
     UIElement[HPText]:SetText["HP: ${Me.CurrentHealth}"]
   </OnRender>
   ```

2. **Use Efficient Element Access**
   ```lavishscript
   ; Cache element reference
   variable object StatusText
   StatusText:Set[${UIElement[StatusText]}]
   StatusText:SetText["New status"]
   ```

3. **Don't Populate ComboBoxes on OnLoad**
   - Populate on OnLeftClick instead
   - Reduces initial load time

### Persistence

1. **Always Save Important Settings**
   ```xml
   <OnLeftClick>
     ; Update script variable
     Script[MyScript].Variable[Setting]:Set[${This.Checked}]

     ; Save to LavishSettings
     LavishSettings[MyScript]:AddSetting[Setting,${This.Checked}]
     LavishSettings[MyScript]:Export[file.xml]
   </OnLeftClick>
   ```

2. **Use StorePosition for Windows**
   ```xml
   <Window name='MyWindow'>
     <StorePosition>1</StorePosition>
   </Window>
   ```

### Error Handling

1. **Check Existence Before Access**
   ```lavishscript
   if ${UIElement[MyWindow](exists)}
   {
       UIElement[MyWindow]:SetText["text"]
   }
   ```

2. **Validate User Input**
   ```xml
   <OnSelect>
     if ${This.SelectedItem.Text.Length} > 0
     {
       Script[MyScript].Variable[Target]:Set[${This.SelectedItem.Text}]
     }
   </OnSelect>
   ```

---

## Troubleshooting

### Common XML Errors

**Problem:** UI doesn't load
```
Solution: Check XML syntax
- Ensure <?xml version="1.0" encoding="UTF-8" ?> at top
- All tags must be closed: <tag></tag> or <tag />
- Root must be <ISUI></ISUI>
```

**Problem:** Special characters causing errors
```
Solution: Use XML entities:
&amp;  for &&
&lt;   for <
&gt;   for >
&quot; for "
&apos; for '
```

**Problem:** Element not found
```
Solution:
- Check element name spelling (case-sensitive)
- Use correct path: child@parent@window
- Verify element actually exists in XML
```

### Event Issues

**Problem:** OnClick not firing
```
Solution:
- Ensure element is visible (not hidden)
- Check if element has size (Width/Height > 0)
- Verify script is running
```

**Problem:** Can't access script variable
```
Solution:
- Use ${Script[scriptname].Variable[varname]}
- Ensure script is loaded and running
- Check variable scope (script vs local)
```

### Script-to-UI Communication

**Problem:** SetText not working
```
Solution:
- Check element name and path
- Ensure element is a text element
- Use ${UIElement[name](exists)} to verify
```

**Problem:** UI changes not visible
```
Solution:
- Check if element is hidden (Visible=0)
- Verify window is not minimized
- Use :Show if needed
```

---

## Advanced Topics

### Dynamic Window Creation

You can create UI elements dynamically from script:

```lavishscript
variable string UICode

UICode:Set["<Window name='DynamicWindow'><X>100</X><Y>100</Y><Width>300</Width><Height>200</Height></Window>"]

ui -load -skin eq2skin code ${UICode}
```

### Multiple Windows

Load multiple UI files:

```lavishscript
ui -load "${LavishScript.HomeDirectory}/Scripts/MyScript/UI/Main.xml"
ui -load "${LavishScript.HomeDirectory}/Scripts/MyScript/UI/Options.xml"
ui -load "${LavishScript.HomeDirectory}/Scripts/MyScript/UI/Advanced.xml"
```

### Conditional UI Elements

Show/hide elements based on class, level, etc.:

```xml
<checkbox Name='ClassSpecificOption'>
  <OnLoad>
    if ${Me.SubClass.Equal[Guardian]}
    {
      This:Show
    }
    else
    {
      This:Hide
    }
  </OnLoad>
</checkbox>
```

---

## Summary

**LavishGUI 1** provides a complete XML-based UI system for ISXEQ2 scripts. Key concepts:

1. **XML Structure** - `<ISUI>` root, Windows, Children, elements
2. **Elements** - Windows, Buttons, Checkboxes, ComboBoxes, Text, Tabs, etc.
3. **Events** - OnLoad, OnClick, OnSelect, OnRender, etc.
4. **Script Integration** - `Script[].Variable[]`, `UIElement[]`, `:SetText`, etc.
5. **Persistence** - LavishSettings, StorePosition, Data attribute
6. **Templates** - Reusable styles and skins

**Next Steps:**
- Study EQ2Bot UI files for real-world examples
- Start with a simple window and build up
- Test frequently as you add elements
- Refer to official LavishGUI documentation for advanced features

---

## Cross-References

- **Core Concepts:** [03_Core_Concepts.md](03_Core_Concepts.md)
- **Advanced Patterns:** [06_Advanced_Patterns_And_Examples.md](06_Advanced_Patterns_And_Examples.md)
- **API Reference:** [01_API_Reference.md](01_API_Reference.md)

**Official Documentation:** https://www.lavishsoft.com/wiki/index.php/LavishGUI

---

*Part of ISXEQ2 Scripting Guide*
