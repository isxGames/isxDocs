# LavishGUI 1 - UI Creation Guide

Complete guide to creating custom user interfaces with LavishGUI 1 XML for InnerSpace scripts.

**Note:** This guide covers **LavishGUI 1**, the XML-based UI system. This is a game-agnostic guide applicable to any InnerSpace extension. For newer scripts, consider LavishGUI 2, which is a different system.

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
Scripts\<ScriptName>\interface\<UIFile>.xml
```

**Example (EVEBot):**
```
Scripts\EVEBot\interface\EVEBot.xml           (main window)
Scripts\EVEBot\interface\eveskin\eveskin.xml  (skin/theme)
```

**GitHub Reference:** https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable/interface

**Loading UI from Script:**
```lavishscript
ui -load "${LavishScript.HomeDirectory}/Scripts/EVEBot/interface/EVEBot.xml"
ui -reload "${LavishScript.HomeDirectory}/Scripts/EVEBot/interface/EVEBot.xml"
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

#### CommandButton (Executes Script Commands)

```xml
<commandbutton name='Run EVEBot' template='EVESkin.Window.TitleBar.RunButton'>
  <OnLeftClick>
    EVEBot:Resume["Clicked Run"]
  </OnLeftClick>
</commandbutton>
```

**EVEBot Example - Save Config Button:**
```xml
<button Name='SaveConfig' template='EveSkin.Window.ClickButton'>
  <text>Save Config</text>
  <x>r320</x>
  <y>10</y>
  <width>150</width>
  <height>20</height>
  <OnLeftClick>
    Script[EVEBot].VariableScope.Config:Save
  </OnLeftClick>
</button>
```

**GitHub Reference:** [EVEBot.xml](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable/interface/EVEBot.xml)

**Note:** The difference:
- **button** - Generic button, can execute any code
- **commandbutton** - Specialized for executing script commands
- Use `Script[scriptname]:QueueCommand[command]` to call script functions

### Checkboxes

Checkboxes for boolean options.

#### Basic Checkbox

**EVEBot Example - Enable Sound:**
```xml
<checkbox Name='cbUseSound'>
  <X>10</X>
  <Y>240</Y>
  <Height>20</Height>
  <Width>100</Width>
  <Text>Enable sound</Text>
  <AutoTooltip>Enable audio events</AutoTooltip>
  <OnLoad>
    if ${Script[EVEBot].VariableScope.Config.Common.UseSound}
    {
      This:SetChecked
    }
  </OnLoad>
  <OnLeftClick>
    Script[EVEBot].VariableScope.Config.Common:SetUseSound[${This.Checked}]
  </OnLeftClick>
</checkbox>
```

**GitHub Reference:** [EVEBot.xml](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable/interface/EVEBot.xml)


### ComboBoxes (Dropdowns)

Dropdown selection lists.

#### Basic ComboBox

**EVEBot Example - Behavior Selection:**
```xml
<combobox name='CurrentBehavior'>
  <X>80</X>
  <Y>10</Y>
  <Width>250</Width>
  <Height>15</Height>
  <FullHeight>200</FullHeight>
  <ButtonWidth>20</ButtonWidth>
  <Items>
    <Item Value='1'>Idle</Item>
  </Items>
  <OnSelect>
    if ${This.SelectedItem.Text.NotNULLOrEmpty}
    {
      Logger:Log["Current behavior switched to ${This.SelectedItem.Text}"]
      Script[EVEBot].VariableScope.Config.Common:CurrentBehavior[${This.SelectedItem.Text}]
    }
  </OnSelect>
</combobox>
```

**GitHub Reference:** [EVEBot.xml](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable/interface/EVEBot.xml)

**Key ComboBox Methods:**
- `:AddItem[text]` - Add an item
- `:ClearItems` - Remove all items
- `:Sort` - Sort items alphabetically
- `.SelectedItem.Text` - Get selected item text
- `.ItemByText[text]:Select` - Select an item by text

### Text and Labels

Display static or dynamic text.

**EVEBot Example - Title Bar Text:**
```xml
<Text name='EVEBot_TitleBar_Title' template='EVESkin.Font.TitleBar'>
  <X>200</X>
  <Y>3</Y>
  <Width>130</Width>
  <Height>20</Height>
  <Text>${Script[EVEBot].VariableScope.AppVersion}</Text>
  <OnLoad>
    This:SetText[${Script[EVEBot].VariableScope.AppVersion}]
  </OnLoad>
</Text>
```

**GitHub Reference:** [EVEBot.xml](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable/interface/EVEBot.xml)

**Update text from script:**
```lavishscript
UIElement[EVEBot_TitleBar_Title]:SetText["New Version"]
```

### TextBoxes (Text Input)

Allow user text input.

**EVEBot Example - Minimum Drones Input:**
```xml
<Textentry name='MinimumDronesInBay'>
  <X>10</X>
  <Y>30</Y>
  <Width>32</Width>
  <Height>18</Height>
  <MaxLength>2</MaxLength>
  <OnLoad>
    This:SetText[${Script[EVEBot].VariableScope.Config.Common.DronesInBay}]
  </OnLoad>
  <OnChange>
    if ${This.Text.Length} > 0
    {
      Script[EVEBot].VariableScope.Config.Common:SetDronesInBay[${Int[${This.Text}]}]
    }
  </OnChange>
</Textentry>
```

**GitHub Reference:** [EVEBot.xml](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable/interface/EVEBot.xml)

**Read text from script:**
```lavishscript
variable string InputValue
InputValue:Set[${UIElement[MinimumDronesInBay].Text}]
```

### Sliders

Sliders for numeric input with visual feedback.

**EVEBot Example - Avoid Player Range Slider:**
```xml
<slider name='AvoidPlayerRange'>
  <X>10</X>
  <Y>110</Y>
  <Width>40</Width>
  <Height>18</Height>
  <Range>100000</Range>
  <OnLoad>
    This:SetValue[${Script[EVEBot].VariableScope.Config.Miner.AvoidPlayerRange}]
  </OnLoad>
  <OnChange>
    Script[EVEBot].VariableScope.Config.Miner:SetAvoidPlayerRange[${Int[${This.Value}]}]
    UIElement[AvoidPlayerRangeLabel@Miner@EVEBotOptionsTab@EVEBot]:SetText["Min. Distance to Players: ${EVEBot.MetersToKM_Str[${This.Value}]}"]
  </OnChange>
</slider>
<Text name='AvoidPlayerRangeLabel'>
  <X>55</X>
  <Y>110</Y>
  <Width>300</Width>
  <Height>10</Height>
  <Text>Min. Distance to Players: 0km</Text>
  <AutoTooltip>Minimum distance we will keep other players at</AutoTooltip>
  <OnLoad>
    This:SetText["Min. Distance to Players: ${EVEBot.MetersToKM_Str[${UIElement[AvoidPlayerRange@Miner@EVEBotOptionsTab@EVEBot].Value}]}"]
  </OnLoad>
</Text>
```

**GitHub Reference:** [EVEBot.xml](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable/interface/EVEBot.xml)

**Key Slider Properties:**
- `<Range>100000</Range>` - Maximum value (0 to Range)
- `${This.Value}` - Current slider value
- `:SetValue[n]` - Set slider position
- `OnChange` - Fires when value changes

**Pattern - Slider with Synchronized Label:**
- Update label text in both OnLoad and OnChange
- Use script methods for unit conversion
- Store value with `${Int[${This.Value}]}` to ensure integer

### Tabs and TabControls

Create tabbed interfaces.

**EVEBot Example - Main Options TabControl:**
```xml
<TabControl Name='EVEBotOptionsTab' template='EVESkin.TabControl'>
  <X>0</X>
  <Y>4</Y>
  <Width>100%</Width>
  <Height>100%</Height>
  <Tabs>
    <Tab Name='Status'>
      <button Name='SaveConfig' template='EveSkin.Window.ClickButton'>
        <text>Save Config</text>
        <x>r320</x>
        <y>10</y>
        <width>150</width>
        <height>20</height>
        <OnLeftClick>
          Script[EVEBot].VariableScope.Config:Save
        </OnLeftClick>
      </button>
    </Tab>
    <Tab Name='Main'>
      <!-- Main tab content -->
    </Tab>
  </Tabs>
</TabControl>
```

**GitHub Reference:** [EVEBot.xml](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable/interface/EVEBot.xml)

**Switch tabs from script:**
```lavishscript
UIElement[EVEBotOptionsTab].Tab[Main]:Select
```

### Frames

Container elements for grouping UI elements.

**EVEBot Example - Fleeing Options Frame:**
```xml
<frame name='FleeingFrame'>
  <x>0</x>
  <y>0</y>
  <width>100%</width>
  <height>100%</height>
  <children>
    <checkbox name='cbRunOnLowAmmo'>
      <X>10</X>
      <Y>10</Y>
      <Height>20</Height>
      <Width>100</Width>
      <Text>Run On Low Ammo</Text>
      <AutoTooltip>If checked, run to safe spot when low on ammo.</AutoTooltip>
      <OnLoad>
        if ${Script[EVEBot].VariableScope.Config.Combat.RunOnLowAmmo}
        {
          This:SetChecked
        }
      </OnLoad>
      <OnLeftClick>
        Script[EVEBot].VariableScope.Config.Combat:SetRunOnLowAmmo[${This.Checked}]
      </OnLeftClick>
    </checkbox>
  </children>
</frame>
```

**GitHub Reference:** [EVEBot.xml](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable/interface/EVEBot.xml)

### Lists and ListBoxes

Display scrollable lists of items.

**EVEBot Example - Fleet Members Listbox:**
```xml
<listbox Name='FleetMembers'>
  <X>10</X>
  <Y>90</Y>
  <Width>530</Width>
  <Height>175</Height>
  <SelectMultiple>0</SelectMultiple>
  <Sort>Text</Sort>
  <OnLoad>
    Script[EVEBot].VariableScope.Config.Fleet:RefreshFleetMembers
    variable iterator InfoFromSettings
    Script[EVEBot].VariableScope.Config.Fleet.FleetMembers:GetIterator[InfoFromSettings]
    if ${InfoFromSettings:First(exists)}
    {
      do
      {
        This:AddItem[${InfoFromSettings.Value.FleetMemberName}]
      }
      while ${InfoFromSettings:Next(exists)}
    }
  </OnLoad>
  <OnSelect>
    UIElement[tbAddFleetMember@Fleet@EVEBotOptionsTab@EVEBot]:SetText[${This.SelectedItem.Text}]
  </OnSelect>
</listbox>
```

**GitHub Reference:** [EVEBot.xml](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable/interface/EVEBot.xml)

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

**EVEBot Example:**
```xml
<OnLoad>
  if ${Script[EVEBot].VariableScope.Config.Common.UseSound}
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

Use direct atom method calls or `Script:QueueCommand` to execute script code:

**EVEBot Example - Direct method call:**
```xml
<commandbutton name='Run EVEBot'>
  <OnLeftClick>
    EVEBot:Resume["Clicked Run"]
  </OnLeftClick>
</commandbutton>
```

**EVEBot Example - Config save:**
```xml
<button Name='SaveConfig'>
  <OnLeftClick>
    Script[EVEBot].VariableScope.Config:Save
  </OnLeftClick>
</button>
```

**GitHub Reference:** [EVEBot.xml](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable/interface/EVEBot.xml)

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

**EVEBot Example:**
```xml
<button Name='SaveConfig' Template='EveSkin.Window.ClickButton'>
  <text>Save Config</text>
  <width>150</width>
  <height>20</height>
</button>
```

### Creating a Skin

**EVEBot uses eveskin.xml for skin/template definitions:**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<ISUI>
  <Skin Name='eveskin' Template='Default Skin'>
    <SkinTemplate Base='window' Skin='EVESkin.window' />
    <SkinTemplate Base='button' Skin='EVESkin.button' />
    <SkinTemplate Base='tabcontrol' Skin='EVESkin.TabControl' />
  </Skin>

  <!-- Example: Button Template -->
  <template name='EVESkin.Window.TitleBar.RunButton'>
    <X>50</X>
    <Y>3</Y>
    <Width>50</Width>
    <Height>14</Height>
    <Border>0</Border>
    <AutoTooltip>Run the bot</AutoTooltip>
  </template>
</ISUI>
```

**GitHub Reference:** [eveskin.xml](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable/interface/eveskin/eveskin.xml)

### Using Textures in Templates

EVEBot uses PNG texture files stored in `interface/eveskin/MainGUI/` for UI elements like buttons, checkboxes, and comboboxes. These are referenced in template definitions.

**GitHub Reference:** [MainGUI Folder](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable/interface/eveskin/MainGUI)

---

## Complete Working Examples

### Example: EVEBot Main Window Structure

**EVEBot provides a comprehensive real-world example of LavishGUI 1 implementation.**

**Window with TitleBar and TabControl:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<ISUI>
  <window name='EVEBot'>
    <Visible>1</Visible>
    <X>200</X>
    <Y>300</Y>
    <Width>550</Width>
    <Height>320</Height>
    <Title>EVEBot Options</Title>
    <TitleBar template='EVESkin.Window.TitleBar'>
      <Width>100%</Width>
      <Height>20</Height>
      <Children>
        <commandbutton name='Run EVEBot' template='EVESkin.Window.TitleBar.RunButton'>
          <OnLeftClick>
            EVEBot:Resume["Clicked Run"]
          </OnLeftClick>
        </commandbutton>
        <button Name='Close' template='EVESkin.Window.TitleBar.Close'>
          <OnLeftClick>
            EVEBot:EndBot[]
          </OnLeftClick>
        </button>
      </Children>
    </TitleBar>
    <Children>
      <TabControl Name='EVEBotOptionsTab' template='EVESkin.TabControl'>
        <X>0</X>
        <Y>4</Y>
        <Width>100%</Width>
        <Height>100%</Height>
        <Tabs>
          <Tab Name='Main'>
            <!-- Tab content with checkboxes, comboboxes, etc. -->
          </Tab>
        </Tabs>
      </TabControl>
    </Children>
  </window>
</ISUI>
```

**Key Patterns from EVEBot:**

1. **Checkbox with Config Integration:**
```xml
<checkbox Name='cbUseSound'>
  <OnLoad>
    if ${Script[EVEBot].VariableScope.Config.Common.UseSound}
    {
      This:SetChecked
    }
  </OnLoad>
  <OnLeftClick>
    Script[EVEBot].VariableScope.Config.Common:SetUseSound[${This.Checked}]
  </OnLeftClick>
</checkbox>
```

2. **Textentry with Validation:**
```xml
<Textentry name='MinimumDronesInBay'>
  <OnLoad>
    This:SetText[${Script[EVEBot].VariableScope.Config.Common.DronesInBay}]
  </OnLoad>
  <OnChange>
    if ${This.Text.Length} > 0
    {
      Script[EVEBot].VariableScope.Config.Common:SetDronesInBay[${Int[${This.Text}]}]
    }
  </OnChange>
</Textentry>
```

3. **Combobox with Dynamic Selection:**
```xml
<combobox name='CurrentBehavior'>
  <Items>
    <Item Value='1'>Idle</Item>
  </Items>
  <OnSelect>
    if ${This.SelectedItem.Text.NotNULLOrEmpty}
    {
      Logger:Log["Current behavior switched to ${This.SelectedItem.Text}"]
      Script[EVEBot].VariableScope.Config.Common:CurrentBehavior[${This.SelectedItem.Text}]
    }
  </OnSelect>
</combobox>
```

**Full Source:** [EVEBot.xml on GitHub](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable/interface/EVEBot.xml)

**Note:** In XML, use `&amp;` for `&&` and `&lt;` for `<` when needed in event code.

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

ui -load -skin eveskin code ${UICode}
```

### Multiple Windows

Load multiple UI files:

**EVEBot Example:**
```lavishscript
ui -load "${LavishScript.HomeDirectory}/Scripts/EVEBot/interface/EVEBot.xml"
ui -load "${LavishScript.HomeDirectory}/Scripts/EVEBot/interface/eveskin/eveskin.xml"
```

### Conditional UI Elements

Show/hide elements based on script state or configuration:

```xml
<checkbox Name='AdvancedOption'>
  <OnLoad>
    if ${Script[MyScript].Variable[ShowAdvanced]}
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

**Real-World Example:**
- Study **EVEBot** for comprehensive LGUI1 implementation
- GitHub: [EVEBot Interface Files](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable/interface)
- Main UI: [EVEBot.xml](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable/interface/EVEBot.xml)
- Skin: [eveskin.xml](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable/interface/eveskin/eveskin.xml)

---

*Part of ISXEVE Scripting Guide*
