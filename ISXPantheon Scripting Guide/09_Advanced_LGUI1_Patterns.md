# Advanced LavishGUI 1 Patterns and Techniques

**Purpose:** Real-world LavishGUI 1 patterns for building script UIs

The patterns and XML in this document are LavishGUI 1 mechanics and are platform-generic (they apply to any InnerSpace extension). The examples use neutral placeholder windows and data sources; where an example would otherwise read live game data (player roster, entities, currency), that input is marked **(planned — not yet implemented)** for ISXPantheon and replaced with a generic placeholder.

**Target Audience:** Advanced script developers

---

## Table of Contents

1. [UI Element Navigation](#ui-element-navigation)
2. [Dynamic UI Updates](#dynamic-ui-updates)
3. [Script Communication](#script-communication)
4. [Settings Integration](#settings-integration)
5. [Advanced List Management](#advanced-list-management)
6. [UI State Management](#ui-state-management)
7. [Advanced Single-Window Patterns](#advanced-single-window-patterns)
   - [OnRender for Conditional Visibility](#onrender-for-conditional-visibility)
   - [Commandcheckbox with noop Commands](#commandcheckbox-with-noop-commands)
   - [Slider with OnChange Updates](#slider-with-onchange-updates)
   - [TextEntry with OnChange Auto-Save](#textentry-with-onchange-auto-save)
   - [Nested Tabcontrols](#nested-tabcontrols)
   - [VariableGauge with Progress Tracking](#variablegauge-with-progress-tracking)
   - [OnLeftClick Tab Switching with Z-Order](#onleftclick-tab-switching-with-z-order)
   - [Listbox with OnSelect Integration](#listbox-with-onselect-integration)
   - [Commandbutton with Global Variable Gates](#commandbutton-with-global-variable-gates)
   - [AutoTooltip for Help Text](#autotooltip-for-help-text)
   - [Combobox with OnLoad Selection](#combobox-with-onload-selection)
   - [Custom Close Button Pattern](#custom-close-button-pattern)
   - [Relative Sizing and Positioning](#relative-sizing-and-positioning)
   - [Frame Visibility Toggle Pattern](#frame-visibility-toggle-pattern)
8. [Advanced Multi-Tab Window Patterns](#advanced-multi-tab-window-patterns)
   - [Checkbox-Controlled Frame Visibility](#checkbox-controlled-frame-visibility)
   - [Dynamic Combobox Population OnLeftClick](#dynamic-combobox-population-onleftclick)
   - [OnRender for Dynamic Window Sizing](#onrender-for-dynamic-window-sizing)
   - [Commandcheckbox with Data Binding](#commandcheckbox-with-data-binding)
   - [LavishSettings Character-Specific Paths](#lavishsettings-character-specific-paths)
   - [Percentage-Based Textentry](#percentage-based-textentry)
   - [Conditional Frame Visibility Based on Script Variable](#conditional-frame-visibility-based-on-script-variable)
   - [Commandbutton with Visible Property](#commandbutton-with-visible-property)
9. [OnRightClick as Programmatic Refresh Trigger](#onrightclick-as-programmatic-refresh-trigger)
10. [Cross-Element UI Synchronization](#cross-element-ui-synchronization)
11. [Production Skin Pattern](#production-skin-pattern)
12. [Production Patterns](#production-patterns)

---

## UI Element Navigation

### Deep Element Access with FindChild

**Pattern:** Navigate complex UI hierarchies using chained `FindChild` calls

**Example:**
```lavishscript
; Access nested text element
ItemName:Set["${UIElement[MyApp].FindChild[GUITabs].FindChild[Sell].FindChild[ItemList].Item[${currentpos}]}"]

; Check nested element state
if !${UIElement[MyApp].FindChild[GUITabs].FindChild[Sell].FindChild[Errortext].Text.Equal["** Waiting **"]}
{
    ; Do something
}

; Get text from deeply nested input
Platina:Set[${UIElement[MyApp].FindChild[GUITabs].FindChild[Sell].FindChild[MinPlatPrice].Text}]
```

**Pattern:**
```lavishscript
; Format: UIElement[Window].FindChild[Tab].FindChild[Tab Name].FindChild[Element]
UIElement[<WindowName>].FindChild[<TabControl>].FindChild[<TabName>].FindChild[<ElementName>]
```

**Why Use This:**
- Precise element targeting in complex UIs
- Works with tab controls
- Survives UI structure changes better than @ notation
- More readable than long @ paths

**When to Use @ Notation vs FindChild:**

**@ Notation (Faster, but brittle):**
```lavishscript
UIElement[ErrorText@Buy@GUITabs@MyApp]:SetText[Saved]
```

**FindChild (Slower, but more robust):**
```lavishscript
UIElement[MyApp].FindChild[GUITabs].FindChild[Buy].FindChild[ErrorText]:SetText[Saved]
```

Use `@` for simple, frequently accessed elements. Use `FindChild` for:
- Dynamic tab navigation
- Complex nested structures
- When element hierarchy might change

---

## Dynamic UI Updates

### Alpha-Based Show/Hide

**Pattern:** Use `SetAlpha` to disable elements visually without hiding them

**Example:**
```lavishscript
; Disable price inputs when MinPrice is unchecked
if !${MinPriceSet}
{
    UIElement[MinPlatPrice@Sell@GUITabs@MyApp]:SetAlpha[0.1]
    UIElement[MinGoldPrice@Sell@GUITabs@MyApp]:SetAlpha[0.1]
    UIElement[MinSilverPrice@Sell@GUITabs@MyApp]:SetAlpha[0.1]
    UIElement[MinCopperPrice@Sell@GUITabs@MyApp]:SetAlpha[0.1]
    UIElement[MinPlatPriceText@Sell@GUITabs@MyApp]:SetAlpha[0.1]
    UIElement[MinPrice@Sell@GUITabs@MyApp]:UnsetChecked
}
else
{
    UIElement[MinPlatPrice@Sell@GUITabs@MyApp]:SetAlpha[1]
    UIElement[MinGoldPrice@Sell@GUITabs@MyApp]:SetAlpha[1]
    UIElement[MinSilverPrice@Sell@GUITabs@MyApp]:SetAlpha[1]
    UIElement[MinCopperPrice@Sell@GUITabs@MyApp]:SetAlpha[1]
    UIElement[MinPlatPriceText@Sell@GUITabs@MyApp]:SetAlpha[1]
    UIElement[MinPrice@Sell@GUITabs@MyApp]:SetChecked
}
```

**Benefits:**
- Elements stay in place (no layout shift)
- Values are preserved when re-enabled
- Visually indicates disabled state
- Better UX than Hide/Show

**Common Alpha Values:**
- `1.0` - Fully visible (enabled)
- `0.5` - Semi-transparent (partially disabled)
- `0.1` - Nearly invisible (disabled)
- `0.0` - Invisible (but still exists)

---

## Script Communication

### QueueCommand Pattern

**Pattern:** UI elements communicate with script using `Script:QueueCommand`

**XML Event Handler:**
```xml
<checkbox Name='MatchLowPrice'>
    <OnLeftClick>
        if ${UIElement[MatchLowPrice@Sell@GUITabs@MyApp].Checked}
        {
            Script[myapp]:QueueCommand[call SaveSetting MatchLowPrice TRUE]
            Script[myapp].VariableScope.MatchLowPrice:Set[TRUE]
        }
        else
        {
            Script[myapp]:QueueCommand[call SaveSetting MatchLowPrice FALSE]
            Script[myapp].VariableScope.MatchLowPrice:Set[FALSE]
        }
    </OnLeftClick>
</checkbox>
```

**Why Use QueueCommand:**
- ✅ Thread-safe script communication
- ✅ Commands execute in script context
- ✅ Prevents UI blocking
- ✅ Can queue multiple commands

**Alternative Pattern (Direct Variable Access):**
```xml
<OnLeftClick>
    Script[myapp].VariableScope.MatchLowPrice:Set[${This.Checked}]
</OnLeftClick>
```

**When to Use Each:**

| Pattern | Use When |
|---------|----------|
| `QueueCommand` | Need to call functions, save settings, complex logic |
| `VariableScope` | Simple variable updates, immediate changes |

---

## Settings Integration

### LavishSettings with UI Sync

**Pattern:** Synchronize UI state with LavishSettings on load and save

**Loading Settings to UI (OnLoad):**
```xml
<checkbox Name='MerchantMatch'>
    <OnLoad>
        if ${Script[myapp].VariableScope.MerchantMatch}
        {
            UIElement[MerchantMatch@Sell@GUITabs@MyApp]:SetChecked
        }
        else
        {
            UIElement[MerchantMatch@Sell@GUITabs@MyApp]:UnsetChecked
        }
    </OnLoad>
</checkbox>
```

**Saving Settings from UI (OnLeftClick):**
```xml
<OnLeftClick>
    if ${UIElement[MerchantMatch@Sell@GUITabs@MyApp].Checked}
    {
        Script[myapp]:QueueCommand[call SaveSetting MerchantMatch TRUE]
        Script[myapp].VariableScope.MerchantMatch:Set[TRUE]
    }
    else
    {
        Script[myapp]:QueueCommand[call SaveSetting MerchantMatch FALSE]
        Script[myapp].VariableScope.MerchantMatch:Set[FALSE]
    }
</OnLeftClick>
```

**Script-side SaveSetting Function:**
```lavishscript
function SaveSetting(string setting, string value)
{
    General:Set[${LavishSettings[myapp].FindSet[General]}]
    General:AddSetting["${setting}","${value}"]
    LavishSettings[myapp]:Export["${Script.CurrentDirectory}/MyApp.xml"]
}
```

**Best Practice Pattern:**
1. UI element changes → Call script function
2. Script function updates variable
3. Script function saves to LavishSettings
4. Export settings file

This ensures:
- Settings persistence
- Variable and UI stay in sync
- One source of truth (LavishSettings)

---

## Advanced List Management

### Color-Coded List Items

**Pattern:** Use `SetTextColor` to provide visual feedback in lists

**Example:**
```lavishscript
function ColourItem(int position, string UITab, string colour)
{
    UIElement[MyApp].FindChild[GUITabs].FindChild[${UITab}].FindChild[ItemList].Item[${position}]:SetTextColor[${colour}]
}

; Usage examples
call ColourItem ${currentpos} Sell FFFF0000   ; Red = Error
call ColourItem ${currentpos} Sell FF00FF00   ; Green = Success
call ColourItem ${currentpos} Sell FFFFFF00   ; Yellow = Warning
call ColourItem ${currentpos} Sell FF0000FF   ; Blue = Info
```

**Common Color Codes:**
```lavishscript
; Format: AARRGGBB (Alpha, Red, Green, Blue)
FFFF0000  ; Red - Errors, stopped items
FF00FF00  ; Green - Success, active items
FFFFFF00  ; Yellow - Warnings, pending items
FF0000FF  ; Blue - Information
FFFFFFFF  ; White - Normal/default
FF808080  ; Gray - Disabled items
```

**Use Cases:**
- Highlight errors in lists
- Show item status (selling, sold, failed)
- Indicate selection state
- Provide visual feedback without tooltips

---

## UI State Management

### Dynamic Element Visibility

**Pattern:** Show/hide UI sections based on context

**Example:**
```lavishscript
; Show extra options for advanced item types
UIElement[BuyNameOnly@Buy@GUITabs@MyApp]:Show
UIElement[Harvest@Buy@GUITabs@MyApp]:Show
UIElement[Transmute@Buy@GUITabs@MyApp]:Show
UIElement[BuyAttuneOnly@Buy@GUITabs@MyApp]:Show
UIElement[StartLevelText@Buy@GUITabs@MyApp]:Show
UIElement[EndLevelText@Buy@GUITabs@MyApp]:Show
UIElement[StartLevel@Buy@GUITabs@MyApp]:Show
UIElement[EndLevel@Buy@GUITabs@MyApp]:Show
UIElement[Tier@Buy@GUITabs@MyApp]:Show

; Hide them for simple items
UIElement[BuyNameOnly@Buy@GUITabs@MyApp]:Hide
UIElement[Harvest@Buy@GUITabs@MyApp]:Hide
UIElement[Transmute@Buy@GUITabs@MyApp]:Hide
UIElement[BuyAttuneOnly@Buy@GUITabs@MyApp]:Hide
UIElement[StartLevelText@Buy@GUITabs@MyApp]:Hide
UIElement[StartLevel@Buy@GUITabs@MyApp]:Hide
UIElement[EndLevelText@Buy@GUITabs@MyApp]:Hide
UIElement[EndLevel@Buy@GUITabs@MyApp]:Hide
UIElement[Tier@Buy@GUITabs@MyApp]:Hide
```

**Best Practice:**
- Group related Show/Hide calls
- Use functions to encapsulate visibility logic
- Consider Alpha instead of Hide for layout stability

**Example Function:**
```lavishscript
function ShowAdvancedOptions()
{
    UIElement[BuyNameOnly@Buy@GUITabs@MyApp]:Show
    UIElement[Harvest@Buy@GUITabs@MyApp]:Show
    UIElement[Transmute@Buy@GUITabs@MyApp]:Show
}

function HideAdvancedOptions()
{
    UIElement[BuyNameOnly@Buy@GUITabs@MyApp]:Hide
    UIElement[Harvest@Buy@GUITabs@MyApp]:Hide
    UIElement[Transmute@Buy@GUITabs@MyApp]:Hide
}
```

---

## Advanced Single-Window Patterns

### OnRender for Conditional Visibility

**Pattern:** Use `OnRender` with a frame counter to throttle conditional show/hide of sibling elements

```xml
<Text Name='Status Label'>
    <OnLoad>
        declarevariable CG_Frame_Counter int global 0
    </OnLoad>
    <OnUnload>
        deletevariable CG_Frame_Counter
    </OnUnload>
    <OnRender>
        if ${CG_Frame_Counter} == 0
        {
            ; Drive sibling visibility from a script-side condition
            if ${Script[myapp].Variable[ModeA]}
            {
                if !${This.Parent.FindChild[PanelA].Visible}
                    This.Parent.FindChild[PanelA]:Show
                if ${This.Parent.FindChild[PanelB].Visible}
                    This.Parent.FindChild[PanelB]:Hide
            }
            else
            {
                if !${This.Parent.FindChild[PanelB].Visible}
                    This.Parent.FindChild[PanelB]:Show
                if ${This.Parent.FindChild[PanelA].Visible}
                    This.Parent.FindChild[PanelA]:Hide
            }
        }
        CG_Frame_Counter:Set[${Math.Calc[(${CG_Frame_Counter}+1)%10]}]
    </OnRender>
</Text>
```

**Key Techniques:**
- **Frame counter** - Only check every N frames to reduce CPU usage
- **Modulo operation** - `(${CG_Frame_Counter}+1)%10` resets counter to 0 every 10 frames
- **Condition-driven UI** - Different panels shown based on a script variable
- **Parent navigation** - `This.Parent.FindChild[...]` to access siblings

**Why Use This:**
- Avoid constant CPU-intensive checks
- Dynamic UI based on runtime state
- Cleaner than polling in script

> **PLANNED — NOT YET IMPLEMENTED.** A common driver for this pattern is game state such as the current zone or location; those game-state objects are not yet surfaced for ISXPantheon, so the condition above reads a script variable instead. The OnRender throttle and show/hide mechanics work today.

---

### Commandcheckbox with noop Commands

**Pattern:** Execute multiple commands in a single `Command` or `CommandChecked`

```xml
<Commandcheckbox Name='Quality 1'>
    <Text>1</Text>
    <Command>noop ${Script[myapp].Variable[Quality1]:Set[TRUE]} noop ${Script[myapp].Variable[Quality2]:Set[FALSE]} noop ${Script[myapp].Variable[Quality3]:Set[FALSE]} noop ${Script[myapp].Variable[Quality4]:Set[FALSE]}</Command>
    <CommandChecked />
    <Data>${Script[myapp].Variable[Quality1]}</Data>
</Commandcheckbox>
```

**Key Techniques:**
- `noop` prefix before each command
- Multiple variable sets in single line
- `<CommandChecked />` empty tag (no action when unchecked)

**Pattern for Radio Button Groups:**
```xml
<!-- Radio button effect - only one checked at a time -->
<Commandcheckbox Name='Option1'>
    <Command>noop ${Var1:Set[TRUE]} noop ${Var2:Set[FALSE]} noop ${Var3:Set[FALSE]}</Command>
    <Data>${Var1}</Data>
</Commandcheckbox>
<Commandcheckbox Name='Option2'>
    <Command>noop ${Var1:Set[FALSE]} noop ${Var2:Set[TRUE]} noop ${Var3:Set[FALSE]}</Command>
    <Data>${Var2}</Data>
</Commandcheckbox>
```

---

### Slider with OnChange Updates

**Pattern:** Use sliders for numeric input with real-time feedback

```xml
<Slider Name='CountSlider'>
    <X>30</X>
    <Y>35</Y>
    <Width>120</Width>
    <Height>20</Height>
    <Range>24</Range>
    <OnLoad>
        This:SetValue[${Math.Calc[${This.Parent.FindChild[NumCount].Text} - 1].Int}]
        This.Parent.FindChild[NumCountHalf]:SetText[${Math.Calc[(${This.Value}/2)+1].Int}]
    </OnLoad>
    <OnChange>
        variable int ThisValue=${Math.Calc[${This.Value}+1].Int}
        This.Parent.FindChild[NumCount]:SetText[${ThisValue}]
        This.Parent.FindChild[NumCountHalf]:SetText[${Math.Calc[(${This.Value}/2)+1].Int}]
        LavishSettings[Config File].FindSet[General Options].FindSetting[How many items per session?]:Set[${ThisValue}]
        Script[myapp].Variable[ItemCount]:Set[${ThisValue}]
        LavishSettings[Config File]:Export[${Script[myapp].Variable[configfile]}]
    </OnChange>
</Slider>
```

**Key Techniques:**
- `<Range>24</Range>` - Sets max slider value (0-24)
- `ThisValue` local variable in OnChange
- Update multiple text elements simultaneously
- Save to LavishSettings on every change
- Math calculations for display values

**Common Slider Patterns:**
```lavishscript
; Value 0-24 maps to display 1-25
DisplayValue:Set[${Math.Calc[${This.Value}+1].Int}]

; Value 0-24 maps to display 1-12 (every 2 values)
DisplayValue:Set[${Math.Calc[(${This.Value}/2)+1].Int}]

; Initialize slider from variable
This:SetValue[${Math.Calc[${SomeVariable} - 1].Int}]
```

---

### TextEntry with OnChange Auto-Save

**Pattern:** Automatically save settings when text changes

```xml
<Textentry Name='Camp Timer'>
    <X>240</X>
    <Y>40</Y>
    <Width>45</Width>
    <Height>20</Height>
    <MaxLength>4</MaxLength>
    <Color>FFDDBB00</Color>
    <OnLoad>This:SetText[${Script[myapp].Variable[CampTimer]}]</OnLoad>
    <OnChange>
        LavishSettings[Config File].FindSet[General Options].FindSetting[Camp out after a specified time has elapsed for a crafting session?]:Set[${This.Text}]
        Script[myapp].Variable[CampTimer]:Set[${This.Text}]
        LavishSettings[Config File]:Export[${Script[myapp].Variable[configfile]}]
    </OnChange>
</Textentry>
```

**Key Techniques:**
- Update script variable
- Update LavishSettings
- Export to file immediately
- No "Save" button needed

**Pattern:**
```xml
<OnChange>
    ; 1. Update script variable
    Script[MyScript].Variable[MyVar]:Set[${This.Text}]

    ; 2. Update LavishSettings
    LavishSettings[Config].FindSet[Settings].FindSetting[MySetting]:Set[${This.Text}]

    ; 3. Export to file
    LavishSettings[Config]:Export[${ConfigFile}]
</OnChange>
```

---

### Nested Tabcontrols

**Pattern:** Tabs within tabs for complex option screens

```xml
<Tab Name='Options'>
    <Tabcontrol Name='SubOptions'>
        <X>2%</X>
        <Y>2%</Y>
        <Height>96%</Height>
        <Width>96%</Width>
        <Tabs>
            <Tab Name='General Options'>
                <!-- General options here -->
            </Tab>
            <Tab Name='Shopping'>
                <!-- Shopping options here -->
            </Tab>
            <Tab Name='Movement'>
                <!-- Movement options here -->
            </Tab>
            <Tab Name='Thresholds'>
                <!-- Threshold options here -->
            </Tab>
        </Tabs>
    </Tabcontrol>
</Tab>
```

**Access Pattern:**
```lavishscript
; Access element in nested tab
UIElement[Window].FindUsableChild[Main Tabs,tabcontrol].Tab[Options].FindUsableChild[SubOptions,tabcontrol].Tab[General Options].FindUsableChild[Camp Timer,textentry]
```

---

### VariableGauge with Progress Tracking

**Pattern:** Use VariableGauge for visual progress feedback

```xml
<VariableGauge Name='Process Recipe'>
    <Visible>0</Visible>
    <Data>${Script[myapp].Variable[gaugelevel]}</Data>
    <X>106</X>
    <Y>152</Y>
    <Border>3</Border>
    <Width>200</Width>
    <Height>15</Height>
</VariableGauge>

<Text Name='Gauge Label'>
    <Visible>0</Visible>
    <Text>Currently Processing Recipe..</Text>
</Text>
```

**Script Side:**
```lavishscript
; Show progress UI
UIElement[Craft Selection].FindUsableChild[Process Recipe,variablegauge]:Show
UIElement[Craft Selection].FindUsableChild[Gauge Label,text]:Show

; Update progress (0.0 to 1.0)
gaugelevel:Set[0.1]
UIElement[Craft Selection].FindUsableChild[Gauge Label,text]:SetText[Loading Recipe Info...]

gaugelevel:Set[0.5]
UIElement[Craft Selection].FindUsableChild[Gauge Label,text]:SetText[Calculating Components...]

gaugelevel:Set[1.0]

; Hide when complete
UIElement[Craft Selection].FindUsableChild[Process Recipe,variablegauge]:Hide
UIElement[Craft Selection].FindUsableChild[Gauge Label,text]:Hide
```

**Key Techniques:**
- Start hidden (`<Visible>0</Visible>`)
- Data-bind to script variable
- Update text label to show current step
- Show/hide as needed

---

### OnLeftClick Tab Switching with Z-Order

**Pattern:** Custom tab switching logic with frame management

```xml
<Tabcontrol Name='Craft Main'>
    <OnLeftClick>
        if ${UIElement[Craft Selection].FindUsableChild[Craft Main,tabcontrol].SelectedTab.Name.Equal[Queue]}
        {
            UIElement[Craft Selection].FindUsableChild[Harvest Frame,frame]:Hide:SetZOrder[notalwaysontop]:SetZOrder[movedown]
            UIElement[Craft Selection].FindUsableChild[Crafting List Frame,frame]:Show:SetZOrder[notalwaysontop]:SetZOrder[movetop]
        }
        elseif ${UIElement[Craft Selection].FindUsableChild[Craft Main,tabcontrol].SelectedTab.Name.Equal[Options]}
        {
            UIElement[Craft Selection].FindUsableChild[Harvest Frame,frame]:Hide:SetZOrder[notalwaysontop]:SetZOrder[movedown]
            UIElement[Craft Selection].FindUsableChild[Crafting List Frame,frame]:Hide:SetZOrder[notalwaysontop]:SetZOrder[movedown]
        }
        else
        {
            UIElement[Craft Selection].FindUsableChild[Crafting List Frame,frame]:Hide:SetZOrder[notalwaysontop]:SetZOrder[movedown]
            if ${thisharvestframe}
            {
                UIElement[Craft Selection].FindUsableChild[Harvest Frame,frame]:Show:SetZOrder[alwaysontop]:SetZOrder[movetop]
            }
        }
    </OnLeftClick>
</Tabcontrol>
```

**Key Techniques:**
- `SelectedTab.Name` to detect which tab
- Chain commands: `:Hide:SetZOrder[...]`
- Z-order management for overlapping frames
- Conditional visibility based on script variables

**Z-Order Commands:**
```lavishscript
:SetZOrder[alwaysontop]     ; Always on top of other windows
:SetZOrder[notalwaysontop]  ; Not always on top
:SetZOrder[movetop]         ; Move to top of current layer
:SetZOrder[movedown]        ; Move to bottom of current layer
```

---

### Listbox with OnSelect Integration

**Pattern:** Run a command (or queue work) when an item is selected from a list

```xml
<Listbox Name='Resource List'>
    <X>0</X>
    <Y>16</Y>
    <Width>100%</Width>
    <Height>88</Height>
    <SelectMultiple>1</SelectMultiple>
    <OnSelect>
        echo "Selected: ${This.Item[${ID}].Text.Token[1,|]}"
    </OnSelect>
    <Sort>Text</Sort>
</Listbox>
```

**Key Techniques:**
- `${This.Item[${ID}].Text}` - Get selected item text
- `.Token[1,|]` - Parse item text (format: "Name|Quantity")
- Execute a command directly from OnSelect (here `echo`; could queue a script command)
- `SelectMultiple` allows multiple selections

**Common List Patterns:**
```lavishscript
; Get selected item text
${This.Item[${ID}].Text}

; Get specific token from item text (format: "Name|Value|Color")
${This.Item[${ID}].Text.Token[1,|]}  ; Name
${This.Item[${ID}].Text.Token[2,|]}  ; Value

; Run a command with the item data (a future game command would go here)
echo "${This.Item[${ID}].Text.Token[1,|]}"
```

> **PLANNED — NOT YET IMPLEMENTED.** Driving a game action (such as a marketplace/broker search) from the OnSelect handler requires game commands that are not yet surfaced. The listbox + OnSelect + token-parsing mechanics shown here work today against any data source.

---

### Commandbutton with Global Variable Gates

**Pattern:** Prevent double-clicks with global flags

```xml
<OnLoad>
    declare thisbuttondisabled bool global 0
    declare thisbuttonrefresh bool global 0
    declare thisbuttonpaused bool global 0
</OnLoad>
<OnUnload>
    deletevar thisbuttondisabled
    deletevar thisbuttonrefresh
    deletevar thisbuttonpaused
</OnUnload>

<!-- In button -->
<Commandbutton name='Add Recipe'>
    <OnLeftClick>
        if !${thisbuttondisabled}
        {
            thisbuttondisabled:Set[1]
            UIElement[Craft Selection].FindUsableChild[Add Recipe,commandbutton]:Hide
            Script[myapp]:QueueCommand[Craft:AddRecipe]
        }
    </OnLeftClick>
</Commandbutton>
```

**Key Techniques:**
- Declare flags in window's `OnLoad`
- Check flag before executing
- Set flag and hide button immediately
- Script re-enables when done

**Pattern:**
```lavishscript
; In Window OnLoad
declare thisbutton_<action> bool global 0

; In Button OnLeftClick
if !${thisbutton_<action>}
{
    thisbutton_<action>:Set[1]
    UIElement[...]:Hide  ; Hide button
    Script:QueueCommand[DoAction]
}

; In script when action completes
thisbutton_<action>:Set[0]
UIElement[...]:Show  ; Re-show button
```

---

### AutoTooltip for Help Text

**Pattern:** Add hover tooltips without script code

```xml
<Textentry Name='Tier'>
    <X>280</X>
    <Y>10</Y>
    <Width>35</Width>
    <Height>20</Height>
    <Text>1</Text>
    <AutoTooltip>Choice index, starting with 1 at the top.</AutoTooltip>
</Textentry>

<Commandcheckbox Name='Craft TTS'>
    <Text>Enable TTS</Text>
    <Autotooltip>
        Enables Text to Speech support.
        Requires LSMTTS.dll, available separately in the Modules folder of SVN.
    </Autotooltip>
</Commandcheckbox>
```

**Key Techniques:**
- `<AutoTooltip>` works for both single-line and multi-line tooltips (LavishGUI 1 XML element names are case-insensitive)
- No script code needed
- Great for user help

---

### Combobox with OnLoad Selection

**Pattern:** Initialize combobox to specific value on load

```xml
<Combobox Name='Category Select'>
    <X>100</X>
    <Y>10</Y>
    <Width>80</Width>
    <Height>23</Height>
    <Fullheight>185</Fullheight>
    <ButtonWidth>20</ButtonWidth>
    <OnLoad>
        This.ItemByText[ALL]:Select
        This.ItemByText[${Script[myapp].Variable[DefaultCategory]}]:Select
    </OnLoad>
    <OnSelect>
        Script[myapp]:QueueCommand[call OnCategorySelected "${This.SelectedItem.Text}"]
    </OnSelect>
    <Items>
        <Item Value='1'>ALL</Item>
        <Item Value='2'>Category A</Item>
        <Item Value='3'>Category B</Item>
        <Item Value='4'>Category C</Item>
        <!-- ... more items -->
    </Items>
</Combobox>
```

**Key Techniques:**
- `This.ItemByText[...]` - Select by text (not value)
- Multiple `:Select` calls (last one wins)
- Seed the default from a script variable in `OnLoad`
- `OnSelect` fires when selection changes

**Selection Methods:**
```lavishscript
This.ItemByText[ItemName]:Select       ; Select by display text
This.ItemByValue[1]:Select             ; Select by value
This:SelectItem[3]                     ; Select by index (1-based)
```

---

### Custom Close Button Pattern

**Pattern:** Custom close action instead of default window close

```xml
<TitleBar Template='window.Titlebar'>
    <Children>
        <Text Name='Title' Template='window.Titlebar.Title' />
        <Button Name='Minimize' Template='window.Titlebar.Minimize' />
        <commandbutton name='Custom Close Button' Template='window.Titlebar.Close'>
            <Command>Script[myapp]:End</Command>
        </commandbutton>
    </Children>
</TitleBar>
```

**Why Use This:**
- Run cleanup before closing
- Prompt for confirmation
- Save state before exit
- Different from just hiding window

---

### Relative Sizing and Positioning

**Pattern:** Use `r` prefix for relative sizing

```xml
<Frame Name='Main Frame'>
    <X>10</X>
    <Y>28</Y>
    <Height>r-13</Height>  <!-- Window height minus 13 pixels -->
    <Width>r20</Width>      <!-- Window width minus 20 pixels -->
</Frame>

<Listbox Name='Process List'>
    <X>20</X>
    <Y>78</Y>
    <Width>r40</Width>      <!-- Window width minus 40 pixels -->
    <Height>r120</Height>   <!-- Window height minus 120 pixels -->
</Listbox>
```

**Relative Positioning:**
```xml
<!-- Position from right edge -->
<X>r85</X>  <!-- 85 pixels from right edge -->

<!-- Size relative to parent -->
<Width>r20</Width>   <!-- Parent width minus 20 -->
<Height>r50</Height> <!-- Parent height minus 50 -->

<!-- Percentage sizing -->
<Width>50%</Width>   <!-- 50% of parent width -->
<X>25%</X>           <!-- 25% from left edge -->
```

---

### Frame Visibility Toggle Pattern

**Pattern:** Show/hide sections of UI

```xml
<!-- Script variable controls visibility -->
<Frame Name='Harvest Frame'>
    <Visible>0</Visible>  <!-- Start hidden -->
    <Children>
        <!-- Complex content here -->
    </Children>
</Frame>

<!-- In script or other element OnClick -->
if ${thisharvestframe}
{
    UIElement[Craft Selection].FindUsableChild[Harvest Frame,frame]:Show
}
else
{
    UIElement[Craft Selection].FindUsableChild[Harvest Frame,frame]:Hide
}
```

**Common Frame Patterns:**
```lavishscript
; Show frame and bring to front
Frame:Show:SetZOrder[alwaysontop]:SetZOrder[movetop]

; Hide frame and send to back
Frame:Hide:SetZOrder[notalwaysontop]:SetZOrder[movedown]

; Toggle frame visibility
if ${Frame.Visible}
    Frame:Hide
else
    Frame:Show
```

---

## Advanced Multi-Tab Window Patterns

### Checkbox-Controlled Frame Visibility

**Pattern:** Use a checkbox to show/hide an associated input frame (primary/secondary selection toggles)

```xml
<checkbox Name='MainTank'>
    <X>5%</X>
    <Y>10</Y>
    <Text>Main Tank:</Text>
    <OnLoad>
        if ${Script[mybot].VariableScope.MainTank}
        {
            This:SetChecked
        }
    </OnLoad>
    <OnLeftClick>
        if ${This.Checked}
        {
            UIElement[MyBot].FindUsableChild[MainTank Frame,Frame]:Hide
            Script[mybot].Variable[MainTank]:Set[TRUE]
            LavishSettings[MyBot].FindSet[Character].FindSet[General Settings]:AddSetting[I am the Main Tank?,TRUE]
            Script[mybot].VariableScope.MyBot:Save_Settings
        }
        else
        {
            UIElement[MyBot].FindUsableChild[MainTank Frame,Frame]:Show
            Script[mybot].Variable[MainTank]:Set[FALSE]
            LavishSettings[MyBot].FindSet[Character].FindSet[General Settings]:AddSetting[I am the Main Tank?,FALSE]
            Script[mybot].VariableScope.MyBot:Save_Settings
        }
    </OnLeftClick>
</checkbox>

<Frame Name='MainTank Frame'>
    <OnLoad>
        if ${Script[mybot].VariableScope.MainTank}
        {
            This:Hide
        }
        else
        {
            This:Show
        }
    </OnLoad>
    <Children>
        <combobox name='MainTank'>
            <!-- Combobox for selecting someone else as MT -->
        </combobox>
    </Children>
</Frame>
```

**Key Techniques:**
- Checkbox controls frame visibility
- Checked = "I am X" (hide selection frame)
- Unchecked = "Someone else is X" (show selection frame)
- Sync state on OnLoad

**Use Cases:**
- "I am the Main Tank" vs "Select Main Tank"
- "I am the Main Assist" vs "Select Main Assist"
- "Use this setting" vs "Configure setting"

---

### Dynamic Combobox Population OnLeftClick

**Pattern:** Rebuild a combobox's items from a runtime data source each time it is clicked

The teaching point is the LGUI1 mechanic: clear the combobox in `OnLeftClick`, re-add items from whatever data you have at that moment, then sort. The data source below is a generic script-side list so the example runs today.

```xml
<combobox name='Selection'>
    <OnLoad>
        This:AddItem[${Script[myapp].Variable[CurrentSelection]}]
        This.ItemByText[${Script[myapp].Variable[CurrentSelection]}]:Select
    </OnLoad>
    <OnSelect>
        Script[myapp].Variable[CurrentSelection]:Set[${This.SelectedItem.Text}]
        LavishSettings[MyApp].FindSet[Profile].FindSet[General Settings]:AddSetting[Selection,${This.SelectedItem.Text}]
        Script[myapp].VariableScope.MyApp:Save_Settings
    </OnSelect>
    <OnLeftClick>
        declare tmpIt iterator
        This:ClearItems

        ; Rebuild from a generic collection the script maintains
        Script[myapp].VariableScope.Choices:GetIterator[tmpIt]
        if ${tmpIt:First(exists)}
        {
            do
            {
                This:AddItem[${tmpIt.Value}]
            }
            while ${tmpIt:Next(exists)}
        }

        This:Sort
    </OnLeftClick>
</combobox>
```

**Key Techniques:**
- Clear items on click, then rebuild from the current data
- Iterate a data source (collection/settings) to add items
- Sort alphabetically after populating
- Declare a local iterator inside `OnLeftClick`

> **PLANNED — NOT YET IMPLEMENTED.** A common use is populating this combobox with the current group/raid members (and their pets) read live from the game. Those roster/entity objects are not yet surfaced for ISXPantheon, so the example above populates from a generic script-side collection instead. When entity enumeration is available, swap the iterator source for the live roster. (Note for that future case: rosters can have gaps — iterate the full slot range, not just up to the member count.)

**Why Use This:**
- Always reflects the current data at click time
- Handles data changes without pre-populating
- Cleaner than maintaining the list eagerly

---

### OnRender for Dynamic Window Sizing

**Pattern:** Adjust window height based on active tab

```xml
<Window name='MyBot'>
    <Width>500</Width>
    <Height>500</Height>
    <OnRender>
        if ${UIElement[MyBot].Minimized}
        {
            UIElement[Title@TitleBar@MyBot]:SetText[MyBot: ${Script[mybot].Variable[Me_Name]}]
        }
        else
        {
            UIElement[Title@TitleBar@MyBot]:SetText[MyBot - ${Script[mybot].Variable[Me_Name]}]

            if ${UIElement[MyBot].FindChild[MyBot Tabs].FindChild[Main].Visible}
            {
                if ${UIElement[MyBot].Height} != 350
                {
                    UIElement[MyBot]:SetHeight[350]
                }
            }
            elseif ${UIElement[MyBot].FindChild[MyBot Tabs].FindChild[MyBot Options].Visible}
            {
                if ${UIElement[MyBot].Height} != 350
                {
                    UIElement[MyBot]:SetHeight[350]
                }
            }
        }
    </OnRender>
</Window>
```

**Key Techniques:**
- Check window state (minimized)
- Update title text based on state
- Resize window based on active tab
- Prevent unnecessary resize with height check

**Use Cases:**
- Different tab requires different height
- Minimize state changes title format
- Dynamic content needs resize

---

### Commandcheckbox with Data Binding

**Pattern:** Two-way data binding between checkbox and LavishSettings

```xml
<Commandcheckbox Name='AutoFollow'>
    <Text>Use built-in Auto Follow:</Text>
    <AutoTooltip>Select to use the game-client auto-follow on the target instead of the script follow</AutoTooltip>
    <OnLeftClick>
        if ${This.Checked}
        {
            Script[mybot].Variable[AutoFollowMode]:Set[TRUE]
            LavishSettings[MyBot].FindSet[Character].FindSet[MyBotExtras]:AddSetting["Auto Follow Mode",TRUE]
            Script[mybot].VariableScope.MyBot:Save_Settings
        }
        else
        {
            Script[mybot].Variable[AutoFollowMode]:Set[FALSE]
            LavishSettings[MyBot].FindSet[Character].FindSet[MyBotExtras]:AddSetting["Auto Follow Mode",FALSE]
            Script[mybot].VariableScope.MyBot:Save_Settings
        }
    </OnLeftClick>
    <Data>${LavishSettings[MyBot].FindSet[Character].FindSet[MyBotExtras].FindSetting[Auto Follow Mode]}</Data>
</Commandcheckbox>
```

**Key Techniques:**
- `<Data>` binds to LavishSettings value
- `OnLeftClick` updates both script variable and settings
- Call script method to save settings
- Auto-updates when settings change externally

**Difference from Regular Checkbox:**
```xml
<!-- Regular checkbox - manual state management -->
<checkbox Name='MyCheck'>
    <OnLoad>
        if ${SomeCondition}
            This:SetChecked
    </OnLoad>
</checkbox>

<!-- Commandcheckbox - data-bound -->
<Commandcheckbox Name='MyCheck'>
    <Data>${SomeSetting}</Data>
    <OnLeftClick>
        ; Update setting
    </OnLeftClick>
</Commandcheckbox>
```

---

### LavishSettings Character-Specific Paths

**Pattern:** Store settings per character

```xml
<OnSelect>
    ; Store in character-specific setting
    LavishSettings[MyBot].FindSet[Character].FindSet[General Settings]:AddSetting[Who is the Main Assist?,${This.SelectedItem.Text}]

    ; Call script method to save
    Script[mybot].VariableScope.MyBot:Save_Settings
</OnSelect>
```

**Settings Structure:**
```
LavishSettings[MyBot]
└── Character (Character Name)
    ├── General Settings
    │   ├── I am the Main Tank?
    │   ├── Who is the Main Tank?
    │   ├── I am the Main Assist?
    │   └── Who is the Main Assist?
    └── MyBotExtras
        ├── Auto Follow Mode
        └── AutoFollowee
```

**Key Techniques:**
- `FindSet[Character]` - Character-specific root
- `FindSet[General Settings]` - Category
- `AddSetting[Name,Value]` - Add or update
- `Save_Settings` method - Export to file

---

### Percentage-Based Textentry

**Pattern:** Numeric input for percentage values

```xml
<Textentry Name='AssistHP'>
    <X>42%</X>
    <Y>1</Y>
    <Width>5%</Width>
    <Height>15</Height>
    <Color>FFDDBB00</Color>
    <MaxLength>3</MaxLength>
    <OnLoad>This:SetText[${Script[mybot].Variable[AssistHP]}]</OnLoad>
    <OnChange>
        LavishSettings[MyBot].FindSet[Character].FindSet[General Settings]:AddSetting[Assist and Engage in combat at what Health?,${This.Text}]
        Script[mybot].VariableScope.MyBot:Save_Settings
        Script[mybot].Variable[AssistHP]:Set[${This.Text}]
    </OnChange>
</Textentry>
<Text Name='Health Text'>
    <X>48%</X>
    <Y>1</Y>
    <Text>%  Health</Text>
</Text>
```

**Key Techniques:**
- MaxLength=3 for percentage (0-100)
- Label after textentry for context
- Golden color (FFDDBB00) for input fields
- Small width (5%) for short numbers

---

### Conditional Frame Visibility Based on Script Variable

**Pattern:** Show/hide frames based on path/mode

```xml
<Frame Name='Following Frame'>
    <OnLoad>
        if ${Script[mybot].Variable[PathType]}
        {
            UIElement[MyBot].FindUsableChild[Following Frame,Frame]:Hide
        }
        else
        {
            UIElement[MyBot].FindUsableChild[Following Frame,Frame]:Show
        }
    </OnLoad>
    <Children>
        <!-- Following options -->
    </Children>
</Frame>
```

**Use Cases:**
- Different modes require different UI sections
- Pathing enabled = hide manual follow options
- Feature disabled = hide related UI

---

### Commandbutton with Visible Property

**Pattern:** Toggle button visibility from script

```xml
<Commandbutton name='Special Action'>
    <X>65%</X>
    <Y>120</Y>
    <Width>115</Width>
    <Height>21</Height>
    <Visible>0</Visible>
    <Text>Special Action</Text>
    <OnLeftClick>
        Script[mybot]:QueueCommand[call SpecialAction]
    </OnLeftClick>
</Commandbutton>
```

**Script-Side Control:**
```lavishscript
; Show the button only when the feature is enabled
if ${Script[mybot].Variable[SpecialActionEnabled]}
{
    UIElement[MyBot].FindUsableChild[Special Action,commandbutton]:Show
}

; Hide it otherwise
else
{
    UIElement[MyBot].FindUsableChild[Special Action,commandbutton]:Hide
}
```

**Key Techniques:**
- Start hidden with `<Visible>0</Visible>`
- Script shows based on a runtime condition
- Conditionally-available actions
- Dynamic UI based on character

---

## OnRightClick as Programmatic Refresh Trigger

### Centralized Handler Dispatch Pattern

**Pattern:** Place state-change logic in `OnRightClick`, then use `OnLeftClick` (physical click) and external buttons to forward to it via `:RightClick`. This gives you a single handler that can be invoked from multiple places without duplicating code. Useful for a controller UI with many checkboxes sharing one dispatch handler.

**Why this pattern?** When the same logic must run from multiple triggers (user click, external "update all" button, startup re-apply, cross-session sync), duplicating the code in each trigger is fragile. By centralizing it in `OnRightClick` and using `:RightClick` as a programmatic dispatcher, you get one source of truth.

**The Checkbox (dual handler pattern):**

```xml
<checkbox name='ChkBoxPriorityCurePotionArcaneID' template='chkbox'>
  <X>15</X>
  <Y>30</Y>
  <Font>
    <Color>FF00FF00</Color>
  </Font>
  <Text>Arcane Potion</Text>

  <!-- Physical click just forwards to OnRightClick -->
  <OnLeftClick>
    This:RightClick
  </OnLeftClick>

  <!-- Actual state-change logic lives here (single source of truth) -->
  <OnRightClick>
    relay ${OgreRelayGroup} "Script[\${OgreBotScriptName}]:QueueCommand[call ChangeCastStackListBoxItem ArcanePotionName ${This.Checked}]"
  </OnRightClick>
</checkbox>
```

**The "Update All" Button (programmatic dispatch to multiple checkboxes):**

```xml
<CommandButton Name="UpdateAllCurePotions" template='Button'>
  <X>15</X>
  <Y>10</Y>
  <Width>110</Width>
  <Height>20</Height>
  <Text>Update All Cures</Text>
  <OnLeftClick>
    ; Fire each checkbox's OnRightClick handler programmatically
    This.Parent.FindChild[ChkBoxPriorityCurePotionArcaneID]:RightClick
    This.Parent.FindChild[ChkBoxPriorityCurePotionNoxiousID]:RightClick
    This.Parent.FindChild[ChkBoxPriorityCurePotionElementalID]:RightClick
    This.Parent.FindChild[ChkBoxPriorityCurePotionTraumaID]:RightClick
  </OnLeftClick>
</CommandButton>
```

**Batch "Save Settings" Button** (fires every checkbox's handler to force a save-all):

```xml
<CommandButton Name="SaveCombatSettings">
  <Text>Save All</Text>
  <OnLeftClick>
    This.Parent.FindChild[checkbox_settings_rangedattack]:RightClick
    This.Parent.FindChild[checkbox_settings_meleeattack]:RightClick
    This.Parent.FindChild[checkbox_settings_movemelee]:RightClick
    This.Parent.FindChild[checkbox_settings_movebehind]:RightClick
    This.Parent.FindChild[checkbox_settings_combatarts]:RightClick
    This.Parent.FindChild[checkbox_settings_namedcombatarts]:RightClick
  </OnLeftClick>
</CommandButton>
```

**Key techniques:**

1. **OnLeftClick → OnRightClick forwarding** — The `OnLeftClick` handler contains only `This:RightClick`. The physical click just dispatches to the real handler, making every trigger go through the same code path.

2. **`:RightClick` as a programmatic event** — External code can call `UIElement[name]:RightClick` to fire the OnRightClick handler without requiring an actual mouse click. The user can still right-click the element manually for the same effect.

3. **Parent-relative lookup** — `This.Parent.FindChild[name]` is often cleaner than fully-qualified paths when the dispatcher button is in the same container as the target elements.

4. **Batch operations** — A single "Update All" button can dispatch to dozens of checkboxes, each running its own persistence/broadcast logic, without the button needing to know what that logic is.

**When to use this pattern:**

- Checkboxes that must save state to LavishSettings AND broadcast via relay commands
- "Save All" / "Update All" / "Refresh All" buttons that trigger multiple individual element handlers
- Cross-session synchronization where a relay event needs to re-fire all UI element handlers to reapply state
- Any scenario where the same logic runs from user action + automated trigger

**Note:** Applied to listboxes, this same pattern centralizes list population logic — see the ISXEVE version of this guide for the EVEBot whitelist example where OnRightClick holds the iterate-and-populate code.

---

## Cross-Element UI Synchronization

### One-Way Sync with Value Transformation

**Pattern:** A TextEntry's `OnChange` handler reads its own value, clamps/transforms it into the valid range of a sibling element (like a Slider), and programmatically updates that sibling. Deliberately one-way to avoid infinite feedback loops. A TextEntry and a Slider are kept in sync.

**Why this pattern?** Users often want two ways to set the same value — typing a precise number into a TextEntry or dragging a Slider. Both elements need to stay visually in sync, but naive bidirectional update handlers will cause infinite loops (textentry changes slider → slider's OnChange fires → tries to change textentry → textentry's OnChange fires → ...). This pattern solves it by having both elements independently write to a shared backing store, while only ONE direction of UI sync is implemented.

**The TextEntry (drives the Slider, clamps out-of-range input):**

```xml
<Textentry Name='ScanRange'>
  <X>90%</X>
  <Y>216</Y>
  <Width>7%</Width>
  <Height>15</Height>
  <MaxLength>4</MaxLength>

  <OnLoad>
    This:SetText[${Script[mybot].Variable[ScanRange]}]
  </OnLoad>

  <OnChange>
    variable int Value
    Value:Set[${This.Text}]

    ; Clamp value into slider's valid range (0-45) with transformation (Value - 5)
    if ${Value} >= 50
      UIElement[MyBot].FindUsableChild[ScanRange Slider,slider]:SetValue[45]
    elseif ${Value} <= 5
      UIElement[MyBot].FindUsableChild[ScanRange Slider,slider]:SetValue[1]
    else
      UIElement[MyBot].FindUsableChild[ScanRange Slider,slider]:SetValue[${Math.Calc[${Value} - 5]}]

    ; Both elements write to the shared LavishSetting + script variable
    LavishSettings[MyBot].FindSet[Character].FindSet[General Settings]:AddSetting[What RANGE to SCAN for Mobs?,${Value}]
    Script[mybot].VariableScope.MyBot:Save_Settings
    Script[mybot].Variable[ScanRange]:Set[${Value}]
  </OnChange>
</Textentry>
```

**The Slider (does NOT call back to TextEntry — avoids loops):**

```xml
<Slider Name='ScanRange Slider'>
  <Range>45</Range>
  <OnLoad>
    This:SetValue[${Math.Calc[${Script[mybot].Variable[ScanRange]} - 5]}]
  </OnLoad>

  <OnChange>
    variable int Value
    Value:Set[${This.Value} + 5]

    ; Writes to shared backing store but does NOT touch the TextEntry
    LavishSettings[MyBot].FindSet[Character].FindSet[General Settings]:AddSetting[What RANGE to SCAN for Mobs?,${Value}]
    Script[mybot].VariableScope.MyBot:Save_Settings
    Script[mybot].Variable[ScanRange]:Set[${Value}]
  </OnChange>
</Slider>
```

**Key techniques:**

1. **Value transformation** — The TextEntry accepts 5-50 but the Slider displays 0-45, so values are transformed with `${Math.Calc[${Value} - 5]}` and `${This.Value} + 5`. One element can have a different display range than the other.

2. **Input clamping** — The TextEntry's OnChange handles out-of-range input by clamping rather than rejecting, giving better UX than silently ignoring bad values.

3. **One-way UI sync** — The TextEntry's OnChange updates the Slider, but the Slider's OnChange does NOT update the TextEntry. This breaks the feedback loop. If you need the other direction (slider drag updates TextEntry display), use a separate mechanism like OnRender polling or a shared textBinding.

4. **Shared backing store** — Both handlers independently write to the same LavishSetting and script variable (`ScanRange`). This gives you implicit two-way data sync: the underlying value always reflects the most recent user action, regardless of which element was used to change it.

5. **`FindUsableChild` for robust cross-element lookup** — `UIElement[MyBot].FindUsableChild[ScanRange Slider,slider]:SetValue[...]` works even if the slider is nested inside multiple containers.

**When to use this pattern:**

- Paired input controls (TextEntry + Slider, Slider + Label, Combobox + Visibility toggle)
- Dropdown selections that show/hide dependent panels (e.g. a mode dropdown that reveals mode-specific frames)
- Master/detail UIs where selecting an item in one list updates fields elsewhere
- Any UI state that must stay visually consistent across multiple elements

**Caveats:**

- **Never implement full bidirectional sync with direct element updates** — you will create infinite loops. Use a shared backing variable or one-way sync instead.
- **Value transformations must be symmetric** — if the TextEntry writes `value - 5` to the Slider, the Slider's OnChange must apply `value + 5` to get back to the "real" value.
- **FindUsableChild performance** — each call walks the element tree. For frequently-fired handlers (OnChange fires on every keystroke), consider caching the result in a script variable.

---

## Production Skin Pattern

### Skin Definition File with SkinTemplate Mappings

**Pattern:** Create a complete visual skin in a separate XML file that defines texture templates, font templates, widget templates, AND a `<Skin>` element with `<SkinTemplate>` mappings. Load the skin file first, then load the main UI with `-skin <SkinName>` to apply the skin's templates. A production skin file typically has many templates and multiple DDS texture atlases mapping base element types to skinned templates.

**Why this pattern?** For a complete themed application, defining every template and explicitly mapping them to widget types gives maximum control. The `<Skin>` element tells LavishGUI "when a script requests this skin, use my templates for these widget types by default." UI files can then be loaded with `-skin SkinName` to automatically apply the theme without having to specify `template='...'` on every single element.

**Skin File Structure:**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<ISUI>
  <!-- ============================================== -->
  <!-- 1. SKIN DECLARATION + TEMPLATE MAPPINGS        -->
  <!-- ============================================== -->
  <Skin Name='appskin' Template='Default Skin'>
    <SkinTemplate Base='window' Skin='App.window' />
    <SkinTemplate Base='tabcontrol' Skin='App.tabcontrol' />
    <SkinTemplate Base='combobox' Skin='App.combobox' />
    <SkinTemplate Base='listbox' Skin='App.listbox' />
    <SkinTemplate Base='checkbox' Skin='App.checkbox' />
    <SkinTemplate Base='button' Skin='App.button' />
    <SkinTemplate Base='slider' Skin='App.slider' />
    <SkinTemplate Base='gauge' Skin='App.gauge' />
    <SkinTemplate Base='commandbutton' Skin='App.commandbutton' />
    <SkinTemplate Base='commandcheckbox' Skin='App.commandcheckbox' />
    <SkinTemplate Base='textentry' Skin='App.textentry' />
    <SkinTemplate Base='console' Skin='App.console' />
    <SkinTemplate Base='text' Skin='App.text' />
  </Skin>

  <!-- ============================================== -->
  <!-- 2. TEXTURE TEMPLATES (atlas slicing)           -->
  <!-- ============================================== -->
  <template name='App.window.Texture' Filename='windowelements.dds'>
    <Left>14</Left>
    <Right>414</Right>
    <Top>529</Top>
    <Bottom>893</Bottom>
  </template>

  <template name='App.checkbox.Texture' Filename='commonelements.dds'>
    <Left>0</Left>
    <Right>16</Right>
    <Top>0</Top>
    <Bottom>16</Bottom>
  </template>
  <template name='App.checkbox.TextureHover' Filename='commonelements.dds'>
    <Left>16</Left>
    <Right>32</Right>
    <Top>0</Top>
    <Bottom>16</Bottom>
  </template>
  <template name='App.checkbox.TextureChecked' Filename='commonelements.dds'>
    <Left>48</Left>
    <Right>64</Right>
    <Top>0</Top>
    <Bottom>16</Bottom>
  </template>

  <!-- ============================================== -->
  <!-- 3. FONT TEMPLATES                              -->
  <!-- ============================================== -->
  <template name='App.Font'>
    <Name>Tahoma</Name>
    <Size>10</Size>
    <Color>FFDDBB00</Color>
  </template>

  <!-- Console requires a fixed-width font -->
  <template name='App.console.Font' Template='Default Fixed Font'>
    <Color>FFDDBB00</Color>
  </template>

  <!-- ============================================== -->
  <!-- 4. WIDGET COMPOSITE TEMPLATES                  -->
  <!-- ============================================== -->
  <template name='App.checkbox'>
    <Font template='App.checkbox.Font' />
    <Texture template='App.checkbox.Texture' />
    <TextureHover template='App.checkbox.TextureHover' />
    <TextureChecked template='App.checkbox.TextureChecked' />
  </template>

  <!-- ============================================== -->
  <!-- 5. ORIENTATION-FLIPPED VARIANTS                -->
  <!-- ============================================== -->
  <template name='App.verticalslider' Template='App.slider'>
    <Orientation>vertical</Orientation>
  </template>
</ISUI>
```

**Loading the Skin:**

```lavishscript
method Init_UI()
{
    ; Load order: skin file first, then UI file with -skin flag
    ui -reload "${LavishScript.HomeDirectory}/Interface/skins/AppTheme/AppTheme.xml"
    ui -reload -skin AppTheme "${PATH_UI}/mybot.xml"
}
```

The `-skin` flag tells LavishGUI to apply the named skin's `<SkinTemplate>` mappings to all widgets in the loaded UI file. Every `<checkbox>`, `<button>`, etc. automatically uses the skin's templates without needing `template='...'` on each element.

**Loading child UIs into the same skin:**

```lavishscript
; Class-specific UIs loaded as children must specify the same -skin flag
ui -load -parent "Profile@MyBot Tabs@MyBot" -skin AppTheme "${PATH_UI}/Profile.xml"
ui -load -parent "Extras@MyBot Tabs@MyBot" -skin AppTheme "${PATH_UI}/MyBotExtras.xml"
```

Each child XML loaded via `-parent` also needs `-skin AppTheme` so the templates resolve consistently.

**Key techniques:**

1. **`<Skin>` + `<SkinTemplate>` vs. template library pattern** — The `<Skin>` element explicitly maps base widget types to skinned templates. This is more structured than just defining templates (the "template library" pattern). UI files loaded with `-skin SkinName` automatically use the mapped templates.

2. **Template inheritance from `Default Skin`** — `<Skin Name='appskin' Template='Default Skin'>` means any widget type NOT explicitly mapped falls back to the Default Skin, so you only need to override what you want to change.

3. **Texture atlasing** — Multiple states or widgets packed into a single `.dds` file. Each template uses `<Left>/<Right>/<Top>/<Bottom>` to slice out its region. A typical skin uses several atlases: `windowelements.dds` (window chrome), `commonelements.dds` (widgets), `window_elements_generic.dds` (gauges).

4. **Hierarchical naming convention** — `App.widget.SubComponent` (e.g., `App.checkbox.TextureHover`, `App.window.TitleBar.Title.Font`). This makes template purpose clear and prevents collisions with other loaded skins.

5. **Orientation variants** — Templates like `App.verticalslider` inherit from `App.slider` via `Template='App.slider'` attribute and override with `<Orientation>vertical</Orientation>`.

6. **Widget templates use empty child tags** — `<Font />`, `<Texture />` self-closing inside a widget template act as placeholders that LavishGUI resolves to sibling templates by naming convention. Combined with the skin mapping, this gives you a chain of resolution: widget → texture template → file + slicing.

**File organization:**

```
Interface/
└── skins/
    └── AppTheme/
        ├── AppTheme.xml          ← Skin definition with <Skin> mappings
        ├── windowelements.dds     ← Window chrome atlas
        ├── commonelements.dds     ← Widget atlas
        └── window_elements_generic.dds  ← Gauge atlas

Scripts/
└── MyBot/
    └── UI/
        └── mybot.xml              ← Main UI (loaded with -skin AppTheme)
```

**Benefits of this pattern:**

- **Implicit template application** — UI files don't need `template='...'` on every element; the `-skin` flag applies mappings automatically
- **Theme swapping** — Load a different skin file (`AppTheme` vs `AltTheme`) to retheme without touching UI XML
- **Partial overrides** — `Template='Default Skin'` means you only define what you want to change; everything else uses defaults
- **Child UI consistency** — Class-specific UIs loaded via `-parent -skin` automatically match the parent window's appearance
- **Separation of concerns** — Artists work on the skin file, developers work on the UI files

**Two approaches, one file format:**

LavishGUI supports two approaches to reusable visual templates:

1. **Skin mapping pattern** — Explicit `<Skin>` element with `<SkinTemplate>` mappings; UI loaded with `-skin SkinName` flag
2. **Template library pattern (EVEBot)** — No `<Skin>` element; templates are defined globally and UI elements reference them by name via `template='...'` attribute

Both are valid. The skin mapping pattern is more structured and better for complete themes. The template library pattern is simpler and better for partial styling or when UI elements need explicit template control.

---

## Production Patterns

### Status Text Updates

**Pattern:** Centralized status message display

**Example:**
```lavishscript
; Different status messages throughout execution
UIElement[Errortext@Sell@GUITabs@MyApp]:SetText["** Waiting **"]
UIElement[Errortext@Sell@GUITabs@MyApp]:SetText[" **Processing**"]
UIElement[Errortext@Sell@GUITabs@MyApp]:SetText[" ** Finished **"]
UIElement[Errortext@Sell@GUITabs@MyApp]:SetText["Placing Items"]
UIElement[Errortext@Sell@GUITabs@MyApp]:SetText[" **Paused**"]
UIElement[Errortext@Sell@GUITabs@MyApp]:SetText["Pause ${Math.Calc[${WaitTimer}/600]} Mins"]
```

**Better Pattern with Function:**
```lavishscript
function UpdateStatus(string message, string colour="FFFFFFFF")
{
    UIElement[Errortext@Sell@GUITabs@MyApp]:SetText["${message}"]
    UIElement[Errortext@Sell@GUITabs@MyApp]:SetTextColor[${colour}]
}

; Usage
call UpdateStatus "** Waiting **" FFFFFF00
call UpdateStatus "Processing" FF00FF00
call UpdateStatus "Error occurred" FFFF0000
```

---

### List Population Pattern

**Pattern:** Clear and rebuild lists efficiently

**Example:**
```lavishscript
function UpdateItemList(string tabname)
{
    ; Clear existing list
    UIElement[ItemList@${tabname}@GUITabs@MyApp]:ClearItems

    ; Iterate settings and populate
    LavishSettings[myapp].FindSet[Item]:GetSetIterator[ItemIterator]

    if ${ItemIterator:First(exists)}
    {
        do
        {
            ItemName:Set["${ItemIterator.Key}"]
            UIElement[ItemList@${tabname}@GUITabs@MyApp]:AddItem["${ItemName}"]
        }
        while ${ItemIterator:Next(exists)}
    }
}
```

**Best Practices:**
- Always clear before repopulating
- Use iterators for settings/collections
- Check `(exists)` before accessing
- Consider sorting before adding items

---

### Backup Settings on Load

**Pattern:** Auto-backup settings when script starts

**Example:**
```lavishscript
function main()
{
    ; Load settings
    MyApp:loadsettings

    ; Backup immediately after load
    LavishSettings[myapp]:Export[${BackupPath}Profile_MyApp.XML]

    ; Continue with script
    MyApp:LoadUI
    ; ...
}
```

**Why This Matters:**
- Protects against script crashes
- User can recover from mistakes
- Timestamped backups show history
- Essential for production scripts

**Enhanced Backup Pattern:**
```lavishscript
function BackupSettings()
{
    variable string timestamp
    timestamp:Set["${Time.Date}-${Time.Time}"]
    timestamp:Set["${timestamp.Replace["/","-"]}"]
    timestamp:Set["${timestamp.Replace[":","-"]}"]

    LavishSettings[myapp]:Export[${BackupPath}Profile_${timestamp}.XML]
}
```

---

<!-- CLAUDE_SKIP_START -->
## Summary

These advanced LavishGUI 1 patterns demonstrate:

1. **Deep UI Navigation** - `FindChild` chains for complex hierarchies
2. **Visual State Management** - Alpha-based enable/disable
3. **Script Communication** - `QueueCommand` for thread-safe calls
4. **Settings Persistence** - Sync UI ↔ Variables ↔ LavishSettings
5. **List Management** - Color coding and dynamic population
6. **Production Robustness** - Status updates and backups

**Key Takeaways:**
- Real production scripts are much more complex than examples
- UI and script must stay synchronized
- Visual feedback improves user experience
- Settings backups are essential
- Use the right pattern for the job (@ vs FindChild, Show vs Alpha)
<!-- CLAUDE_SKIP_END -->
