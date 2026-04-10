# LavishGUI 1 - UI Creation Guide

Complete guide to creating custom user interfaces with LavishGUI 1 XML for InnerSpace scripts.

**Note:** This guide covers **LavishGUI 1**, the XML-based UI system used by older InnerSpace scripts. Newer scripts may use LavishGUI 2, which is a different system.

**Official Documentation:** https://www.lavishsoft.com/wiki/index.php/LavishGUI

---

## Table of Contents

1. [Introduction to LavishGUI 1](#introduction-to-lavishgui-1)
2. [Basic XML Structure](#basic-xml-structure)
3. [Positioning and Sizing](#positioning-and-sizing)
4. [Core UI Elements](#core-ui-elements)
5. [Event Handling](#event-handling)
6. [Script-to-UI Interaction](#script-to-ui-interaction)
7. [Templates and Skinning](#templates-and-skinning)
8. [Complete Working Examples](#complete-working-examples)
9. [Best Practices](#best-practices)
10. [Troubleshooting](#troubleshooting)
11. [Advanced Topics](#advanced-topics)

---

## Introduction to LavishGUI 1

<!-- CLAUDE_SKIP_START -->
### What is LavishGUI 1?

**LavishGUI 1** is InnerSpace's original XML-based user interface system. It allows you to:
- Create custom windows and dialogs
- Build interactive controls (buttons, checkboxes, dropdowns, etc.)
- Interface between your LavishScript code and visual elements
- Save/restore window positions and settings
- Create themed UIs with skins and templates
<!-- CLAUDE_SKIP_END -->

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

**Example:**
```
Scripts\MyScript\UI\MyScript.xml      (main window)
Scripts\MyScript\UI\Options.xml       (additional UI)
Scripts\MyScript\UI\MySkin.xml        (skin/theme)
```

**Loading UI from Script:**
```lavishscript
ui -load "${LavishScript.HomeDirectory}/Scripts/MyScript/UI/MyScript.xml"
ui -reload "${LavishScript.HomeDirectory}/Scripts/MyScript/UI/MyScript.xml"
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

## Positioning and Sizing

LavishGUI 1 supports three value types for element position (`X`, `Y`) and size (`Width`, `Height`) properties. These can be freely mixed on the same element.

### Absolute Values (Pixels)

Fixed pixel values measured from the top-left corner of the parent:

```xml
<X>100</X>       <!-- 100px from left edge -->
<Y>50</Y>        <!-- 50px from top edge -->
<Width>200</Width>  <!-- 200px wide -->
<Height>30</Height> <!-- 30px tall -->
```

### Relative Values (`r` Prefix)

The `r` prefix creates coordinates measured from the **opposite edge** of the parent. This is essential for elements that should stay anchored to the right or bottom edge, or stretch to fill available space.

**For position (X, Y):** `rN` places the element N pixels from the right/bottom edge:

```xml
<X>r160</X>      <!-- 160px from the RIGHT edge of parent -->
<Y>r120</Y>      <!-- 120px from the BOTTOM edge of parent -->
```

**For size (Width, Height):** `rN` means "parent dimension minus N pixels":

```xml
<Width>r10</Width>    <!-- parent width minus 10px -->
<Height>r32</Height>  <!-- parent height minus 32px -->
```

**Common patterns:**

```xml
<!-- Console that fills window with a 14px input bar at the bottom -->
<console name='output'>
  <X>0</X>
  <Y>0</Y>
  <Height>r14</Height>      <!-- fills parent height minus 14px -->
  <Width>100%</Width>
</console>
<commandentry name='input'>
  <X>0</X>
  <Y>r14</Y>                <!-- positioned 14px from bottom -->
  <Height>14</Height>
  <Width>100%</Width>
</commandentry>

<!-- Scrollbar slider between two 16px buttons -->
<X>16</X>                    <!-- after left button -->
<Width>r32</Width>           <!-- parent minus 32px (16px each side for buttons) -->

<!-- Right-anchored buttons -->
<X>r200</X>                  <!-- 200px from right edge -->
<Width>80</Width>
```

### Percentage Values

Percentage values are relative to the parent element's corresponding dimension. Fractional percentages are supported.

```xml
<X>10%</X>             <!-- 10% of parent width from left -->
<Y>50%</Y>             <!-- 50% of parent height from top -->
<Width>80%</Width>     <!-- 80% of parent width -->
<Height>47%</Height>   <!-- 47% of parent height -->
<Width>97.5%</Width>   <!-- fractional percentages work -->
```

**Common pattern — fill parent with small margin:**

```xml
<X>2%</X>
<Y>2%</Y>
<Width>96%</Width>
<Height>96%</Height>
```

### Mixing Value Types

All three types can be freely combined on the same element:

```xml
<!-- Console: absolute X, relative Y, percentage width, absolute height -->
<X>5</X>
<Y>r120</Y>
<Width>97.5%</Width>
<Height>115</Height>

<!-- Tab control: absolute position, relative fill -->
<X>5</X>
<Y>5</Y>
<Width>r10</Width>
<Height>r40</Height>
```

### Summary Table

| Property | Absolute | `r` Prefix | Percentage |
|----------|----------|------------|------------|
| **X** | Pixels from left | Pixels from right | % of parent width |
| **Y** | Pixels from top | Pixels from bottom | % of parent height |
| **Width** | Fixed pixels | Parent width minus N | % of parent width |
| **Height** | Fixed pixels | Parent height minus N | % of parent height |

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
    Script[MyScript]:QueueCommand[call StartProcess]
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
<commandcheckbox Name='VerboseMode'>
  <X>10</X>
  <Y>100</Y>
  <Width>150</Width>
  <Height>20</Height>
  <Text>Verbose Logging</Text>
  <AutoTooltip>Enable detailed logging output</AutoTooltip>

  <OnLeftClick>
    if ${This.Checked}
    {
      Script[MyScript].Variable[VerboseMode]:Set[TRUE]
      LavishSettings[MyScript].FindSet[Profile].FindSet[Settings]:AddSetting[VerboseMode,TRUE]
      Script[MyScript].VariableScope.MyScript:Save_Settings
    }
    else
    {
      Script[MyScript].Variable[VerboseMode]:Set[FALSE]
      LavishSettings[MyScript].FindSet[Profile].FindSet[Settings]:AddSetting[VerboseMode,FALSE]
      Script[MyScript].VariableScope.MyScript:Save_Settings
    }
  </OnLeftClick>

  <!-- Load checked state from settings -->
  <Data>${LavishSettings[MyScript].FindSet[Profile].FindSet[Settings].FindSetting[VerboseMode]}</Data>
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

#### Dynamic ComboBox (Populated from Script Data)

```xml
<combobox name='ProfileSelection'>
  <X>10</X>
  <Y>160</Y>
  <Width>150</Width>
  <Height>20</Height>
  <Items></Items>  <!-- Empty initially -->

  <OnLoad>
    ; Load saved selection
    if ${Script[MyScript].Variable[SelectedProfile].Length}
    {
      This:AddItem[${Script[MyScript].Variable[SelectedProfile]}]
      This.ItemByText[${Script[MyScript].Variable[SelectedProfile]}]:Select
    }
  </OnLoad>

  <OnSelect>
    ; Save selection
    Script[MyScript].Variable[SelectedProfile]:Set[${This.SelectedItem.Text}]
    LavishSettings[MyScript].FindSet[Config]:AddSetting[SelectedProfile,${This.SelectedItem.Text}]
    LavishSettings[MyScript]:Export["${LavishScript.HomeDirectory}/Scripts/MyScript/Config.xml"]
  </OnSelect>

  <OnLeftClick>
    ; Populate with available profiles when clicked
    declare tmpvar int
    This:ClearItems
    tmpvar:Set[1]

    while ${tmpvar} <= ${Script[MyScript].Variable[NumProfiles]}
    {
      This:AddItem[${Script[MyScript].Variable[Profile${tmpvar}]}]
      tmpvar:Inc
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
- `:SetSortType[type]` - Set sort type (`text`, `valuereverse`)
- `:SetAutoSort[TRUE]` - Auto-sort when items are added

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

**Multi-line wrapping text:**

```xml
<text Name='DescriptionText'>
  <X>10</X>
  <Y>190</Y>
  <Width>300</Width>
  <Height>60</Height>
  <Text>This is a long description that will wrap to multiple lines within the element bounds.</Text>
  <Wrap>1</Wrap>
</text>
```

The `<Wrap>` element controls word-wrapping: `1` enables wrapping, `0` disables (default). Can also be written as self-closing `<Wrap />` to enable.

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

**TextEntry with input constraints:**

```xml
<textentry Name='PasswordInput'>
  <X>10</X>
  <Y>250</Y>
  <Width>200</Width>
  <Height>20</Height>
  <MaxLength>50</MaxLength>
  <PasswordCharacter>*</PasswordCharacter>
  <OnLoad>
    This:SetText[${LavishSettings[MyScript].FindSetting[Password]}]
  </OnLoad>
</textentry>
```

**Key TextEntry properties:**
- `<MaxLength>N</MaxLength>` — Maximum character count (e.g., `2`, `6`, `25`, `50`, `64`)
- `<PasswordCharacter>*</PasswordCharacter>` — Masks typed characters for password fields

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
    ; Populate with items from a collection
    declare i int
    i:Set[1]
    while ${i} <= ${Script[MyScript].Variable[ItemCount]}
    {
      This:AddItem[${Script[MyScript].Variable[Item${i}]}]
      i:Inc
    }
  </OnLoad>

  <OnSelect>
    echo Selected: ${This.SelectedItem.Text}
  </OnSelect>
</listbox>
```

**ListBox properties:**

| Property | Values | Description |
|----------|--------|-------------|
| `<Sort>` | `None`, `Text`, `Value`, `User` | Sort mode (default: `None`) |
| `<AutoSort>` | `TRUE` / `FALSE` | Re-sort automatically when items are added |
| `<SelectMultiple>` | `0` / `1` | Enable multi-select (default: `0` = single) |

**AddItem with hidden value:**

Items can store both display text and a hidden value (useful for IDs):

```lavishscript
; Single parameter — display text only
This:AddItem[Item Name]

; Dual parameter — display text + hidden value
This:AddItem[Item Name,42]
```

The hidden value enables lookup by value (`ItemByValue`) vs by text (`ItemByText`), and is used as the sort key when `<Sort>Value</Sort>` is set.

**Sort from script:**

```lavishscript
; Set sort type and apply
UIElement[MyList]:SetSortType[text]
UIElement[MyList]:Sort

; Sort by value in reverse (e.g., highest level first)
UIElement[MyList]:SetSortType[valuereverse]
UIElement[MyList]:Sort

; Enable auto-sort for future additions
UIElement[MyList]:SetAutoSort[TRUE]
```

### Sliders

Sliders for numeric input with visual feedback.

```xml
<slider name='SpeedSlider'>
  <X>10</X>
  <Y>200</Y>
  <Width>150</Width>
  <Height>18</Height>
  <Range>100</Range>
  <OnLoad>
    This:SetValue[${Script[MyScript].Variable[SpeedValue]}]
  </OnLoad>
  <OnChange>
    Script[MyScript].Variable[SpeedValue]:Set[${Int[${This.Value}]}]
    UIElement[SpeedLabel]:SetText["Speed: ${This.Value}%"]
  </OnChange>
</slider>
<Text name='SpeedLabel'>
  <X>170</X>
  <Y>200</Y>
  <Width>200</Width>
  <Height>18</Height>
  <Text>Speed: 0%</Text>
  <OnLoad>
    This:SetText["Speed: ${Script[MyScript].Variable[SpeedValue]}%"]
  </OnLoad>
</Text>
```

**Key Slider Properties:**
- `<Range>N</Range>` - Maximum value (slider range is 0 to N)
- `${This.Value}` - Current slider value
- `:SetValue[n]` - Set slider position programmatically
- `OnChange` - Fires when the slider value changes

**Pattern — Slider with Synchronized Label:**
- Update label text in both `OnLoad` (initial state) and `OnChange` (live updates)
- Use `${Int[${This.Value}]}` to ensure integer when storing values
- Place the label element adjacent to the slider for a clean layout

### Console

A dedicated scrollable log output element with a built-in scrollback buffer. Unlike Text or TextBox elements, console elements receive text via the `:Echo[]` method and manage their own line buffer automatically.

**Note:** Console elements require a fixed-width font. LavishGUI 1 will automatically substitute a fixed font even if you specify a variable-width font.

**Basic Console:**

```xml
<console Name='StatusConsole'>
  <X>5</X>
  <Y>r120</Y>
  <Width>97.5%</Width>
  <Height>115</Height>
  <BackBufferSize>1000</BackBufferSize>
  <BackgroundColor>FF000000</BackgroundColor>
  <BorderColor>FFFFFFFF</BorderColor>
  <SelectionColor>FF006666</SelectionColor>
  <Border>1</Border>
  <Font template='console.Font' />
  <ScrollBar template='console.ScrollBar' />
</console>
```

**Console Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `BackBufferSize` | int | Number of lines in the scrollback buffer (default varies; common values: 1000, 2000) |
| `BackgroundColor` | ARGB hex | Background color (e.g., `FF000000` for black) |
| `ScrollbackColor` | ARGB hex | Background color for scrollback area (older lines) |
| `BorderColor` | ARGB hex | Border color |
| `SelectionColor` | ARGB hex | Text selection highlight color |
| `Border` | int | Border width in pixels |
| `Font` | template ref | Font template (must be fixed-width) |
| `ScrollBar` | template ref | Scrollbar template reference |

**Writing to Console from Script:**

Use the `:Echo[]` method on the element — this is the only way to write to a console element:

```lavishscript
; Address by full element path (Name@Parent@Grandparent@...)
UIElement[StatusConsole@Status@OptionsTab@MyWindow]:Echo["Process started"]

; Color codes are supported via \a escape sequences
UIElement[StatusConsole@Status@OptionsTab@MyWindow]:Echo["\agSuccess: task complete"]
UIElement[StatusConsole@Status@OptionsTab@MyWindow]:Echo["\arError: connection failed"]

; Use .Escape on dynamic text to handle special characters
UIElement[StatusConsole@Status@OptionsTab@MyWindow]:Echo["${msg.Escape}"]
```

**Common color codes:** `\ag` (green), `\ar` (red), `\at` (teal), `\ao` (orange), `\ax` (reset to default).

**Important distinctions:**
- The standalone `echo` command writes to the InnerSpace system console, NOT to your custom console element
- Custom console elements only receive text when explicitly addressed via `UIElement[...]:Echo[]`
- Console elements manage their own line buffer and scrollback automatically — no script-side trimming needed (unlike LGUI2 textbox consoles)

**Console with Template:**

Define a reusable console template in your skin file:

```xml
<!-- In your skin file -->
<template name='MyConsole.Font'>
  <Name>Terminal</Name>
  <Size>8</Size>
  <Color>FFFFFFFF</Color>
</template>

<template name='MyConsole'>
  <Border>0</Border>
  <Font template='MyConsole.Font' />
  <SelectionColor>FFD4D0C8</SelectionColor>
  <ScrollBar>console.ScrollBar</ScrollBar>
  <BackBufferSize>2000</BackBufferSize>
</template>
```

```xml
<!-- In your UI file -->
<console Name='StatusConsole' template='MyConsole'>
  <X>5</X>
  <Y>r120</Y>
  <Width>97.5%</Width>
  <Height>115</Height>
</console>
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
    UIElement[Title@TitleBar@MyWindow]:SetText[MyScript - ${Script[MyScript].Variable[StatusText]}]
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
  if ${Script[MyScript].Variable[VerboseMode]}
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
<commandbutton name='StartProcess'>
  <OnLeftClick>
    Script[MyScript]:QueueCommand[call StartProcess]
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
UIElement[Title@TitleBar@MyWindow]:SetText["MyScript - ${Script[MyScript].Variable[StatusText]}"]
```

#### SetChecked / UnsetChecked - Checkboxes

```lavishscript
UIElement[AutoLoot]:SetChecked
UIElement[AutoLoot]:UnsetChecked
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
<button Name='MyButton' Template='Custom.button'>
  <Text>Click Me</Text>
</button>
```

### Creating a Skin

**MySkin.xml example:**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<ISUI>
  <Skin Name='customskin' Template='Default Skin'>
    <SkinTemplate Base='window' Skin='Custom.window' />
    <SkinTemplate Base='button' Skin='Custom.button' />
    <SkinTemplate Base='checkbox' Skin='Custom.checkbox' />
    <SkinTemplate Base='combobox' Skin='Custom.combobox' />
  </Skin>

  <!-- Window Template -->
  <template name='Custom.window'>
    <Border>1</Border>
    <Resizable>1</Resizable>
    <StorePosition>1</StorePosition>
  </template>

  <!-- Button Template -->
  <template name='Custom.button'>
    <Width>100</Width>
    <Height>25</Height>
  </template>
</ISUI>
```

### Using Textures in Templates

```xml
<template name='Custom.window.Texture' Filename='windowelements.dds' ColorKey='00000000'>
  <Left>14</Left>
  <Right>414</Right>
  <Top>529</Top>
  <Bottom>893</Bottom>
</template>
```

### Template Inheritance

Templates can inherit from other templates using the `Template` attribute on the `<template>` definition itself. The child template starts with all parent properties and can override specific ones.

**Basic inheritance (exact copy):**

```xml
<!-- Child is an exact copy of parent (self-closing, no overrides) -->
<template name='button.TextureHover' Template='button.Texture' />
```

**Inheritance with property overrides:**

```xml
<!-- Parent defines base font -->
<template name='Default Font'>
  <Name>Arial</Name>
  <Size>12</Size>
  <Color>FFFFFFFF</Color>
</template>

<!-- Child inherits all properties, overrides specific ones -->
<template name='MyScript.Font' Template='Default Font'>
  <Color>FFFF9900</Color>
  <Size>14</Size>
</template>
```

The child `MyScript.Font` inherits `Name=Arial` from the parent but overrides `Color` and `Size`.

**Multi-level chaining:**

Inheritance can be chained through multiple levels:

```xml
<!-- Level 1: Base widget template -->
<template name='checkbox'>
  <Width>20</Width>
  <Height>20</Height>
  <Border>1</Border>
  <!-- ... textures, colors, etc. -->
</template>

<!-- Level 2: Inherits everything from checkbox -->
<template name='commandcheckbox' Template='checkbox'>
  <!-- Inherits all checkbox properties -->
  <!-- Can override specific ones here -->
</template>
```

The InnerSpace DefaultSkin uses chains up to 3 levels deep (e.g., `slider` → `variableslider` → `verticalvariableslider`).

**Common pattern — skin-specific overrides:**

```xml
<!-- Base template from DefaultSkin -->
<template name='scrollbar'>
  <Border>0</Border>
  <Height>16</Height>
  <Width>16</Width>
</template>

<!-- Skin override inherits base, changes specific properties -->
<template name='MySkin.scrollbar' Template='scrollbar'>
  <Border>1</Border>
  <Height>12</Height>
  <Width>12</Width>
</template>
```

**Note:** The `Template` attribute is case-insensitive (`Template` and `template` both work). Template inheritance on a `<template>` definition creates a new reusable template, while `template` on an element (e.g., `<button template='...'>`) applies a template to that specific element instance.

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
    ; Populate with available options when clicked
    declare tmpvar int
    This:ClearItems
    tmpvar:Set[1]

    while ${tmpvar} &lt;= ${Script[MyScript].Variable[NumOptions]}
    {
      This:AddItem[${Script[MyScript].Variable[Option${tmpvar}]}]
      tmpvar:Inc
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
     UIElement[StatusText]:SetText["Status: ${Script[MyScript].Variable[CurrentStatus]}"]
   </OnRender>
   ```

2. **Only Update UI When Values Change**
   ```lavishscript
   ; Cache the value and only update UI when it changes
   variable string LastStatus
   if !${LastStatus.Equal["${NewStatus}"]}
   {
       UIElement[StatusText]:SetText["${NewStatus}"]
       LastStatus:Set["${NewStatus}"]
   }
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

ui -load -skin customskin code ${UICode}
```

### Multiple Windows

Load multiple UI files:

```lavishscript
ui -load "${LavishScript.HomeDirectory}/Scripts/MyScript/UI/Main.xml"
ui -load "${LavishScript.HomeDirectory}/Scripts/MyScript/UI/Options.xml"
ui -load "${LavishScript.HomeDirectory}/Scripts/MyScript/UI/Advanced.xml"
```

### Conditional UI Elements

Show/hide elements based on script variables or conditions:

```xml
<checkbox Name='ConditionalOption'>
  <OnLoad>
    if ${Script[MyScript].Variable[AdvancedMode]}
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

**LavishGUI 1** provides a complete XML-based UI system for InnerSpace scripts. Key concepts:

1. **XML Structure** - `<ISUI>` root, Windows, Children, elements
2. **Elements** - Windows, Buttons, Checkboxes, ComboBoxes, Text, Tabs, etc.
3. **Events** - OnLoad, OnClick, OnSelect, OnRender, etc.
4. **Script Integration** - `Script[].Variable[]`, `UIElement[]`, `:SetText`, etc.
5. **Persistence** - LavishSettings, StorePosition, Data attribute
6. **Templates** - Reusable styles and skins

<!-- CLAUDE_SKIP_START -->
**Next Steps:**
- Study existing script UI files for real-world examples
- Start with a simple window and build up
- Test frequently as you add elements
- Refer to official LavishGUI documentation for advanced features
<!-- CLAUDE_SKIP_END -->

---

## Reference

**Official Documentation:** https://www.lavishsoft.com/wiki/index.php/LavishGUI
