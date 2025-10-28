# Advanced LavishGUI 1 Patterns and Techniques

**Purpose:** Real-world patterns from production scripts
**Sources:**
- [MyPrices](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/MyPrices) broker script (3,937 lines + 3,288 line UI)
- [EQ2Craft](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Craft) automation script (1,802 line UI)
- [EQ2Bot](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/EQ2Bot) combat bot (7,509 lines + 2,498 line main UI + 39 class UIs)
**Target Audience:** Advanced script developers

---

## Table of Contents

1. [UI Element Navigation](#ui-element-navigation)
2. [Dynamic UI Updates](#dynamic-ui-updates)
3. [Script Communication](#script-communication)
4. [Settings Integration](#settings-integration)
5. [Advanced List Management](#advanced-list-management)
6. [UI State Management](#ui-state-management)
7. [Advanced Patterns from EQ2Craft](#advanced-patterns-from-eq2craft)
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
8. [Advanced Patterns from EQ2Bot](#advanced-patterns-from-eq2bot)
   - [Checkbox-Controlled Frame Visibility](#checkbox-controlled-frame-visibility)
   - [Dynamic Combobox Population OnLeftClick](#dynamic-combobox-population-onleftclick)
   - [OnRender for Dynamic Window Sizing](#onrender-for-dynamic-window-sizing)
   - [Commandcheckbox with Data Binding](#commandcheckbox-with-data-binding)
   - [LavishSettings Character-Specific Paths](#lavishsettings-character-specific-paths)
   - [Percentage-Based Textentry](#percentage-based-textentry)
   - [Conditional Frame Visibility Based on Script Variable](#conditional-frame-visibility-based-on-script-variable)
   - [Commandbutton with Visible Property](#commandbutton-with-visible-property)
9. [Production Patterns](#production-patterns)

---

## UI Element Navigation

### Deep Element Access with FindChild

**Pattern:** Navigate complex UI hierarchies using chained `FindChild` calls

**Example from MyPrices:**
```lavishscript
; Access nested text element
ItemName:Set["${UIElement[MyPrices].FindChild[GUITabs].FindChild[Sell].FindChild[ItemList].Item[${currentpos}]}"]

; Check nested element state
if !${UIElement[MyPrices].FindChild[GUITabs].FindChild[Sell].FindChild[Errortext].Text.Equal["** Waiting **"]}
{
    ; Do something
}

; Get text from deeply nested input
Platina:Set[${UIElement[MyPrices].FindChild[GUITabs].FindChild[Sell].FindChild[MinPlatPrice].Text}]
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
UIElement[ErrorText@Buy@GUITabs@MyPrices]:SetText[Saved]
```

**FindChild (Slower, but more robust):**
```lavishscript
UIElement[MyPrices].FindChild[GUITabs].FindChild[Buy].FindChild[ErrorText]:SetText[Saved]
```

Use `@` for simple, frequently accessed elements. Use `FindChild` for:
- Dynamic tab navigation
- Complex nested structures
- When element hierarchy might change

---

## Dynamic UI Updates

### Alpha-Based Show/Hide

**Pattern:** Use `SetAlpha` to disable elements visually without hiding them

**Example from MyPrices:**
```lavishscript
; Disable price inputs when MinPrice is unchecked
if !${MinPriceSet}
{
    UIElement[MinPlatPrice@Sell@GUITabs@MyPrices]:SetAlpha[0.1]
    UIElement[MinGoldPrice@Sell@GUITabs@MyPrices]:SetAlpha[0.1]
    UIElement[MinSilverPrice@Sell@GUITabs@MyPrices]:SetAlpha[0.1]
    UIElement[MinCopperPrice@Sell@GUITabs@MyPrices]:SetAlpha[0.1]
    UIElement[MinPlatPriceText@Sell@GUITabs@MyPrices]:SetAlpha[0.1]
    UIElement[MinPrice@Sell@GUITabs@MyPrices]:UnsetChecked
}
else
{
    UIElement[MinPlatPrice@Sell@GUITabs@MyPrices]:SetAlpha[1]
    UIElement[MinGoldPrice@Sell@GUITabs@MyPrices]:SetAlpha[1]
    UIElement[MinSilverPrice@Sell@GUITabs@MyPrices]:SetAlpha[1]
    UIElement[MinCopperPrice@Sell@GUITabs@MyPrices]:SetAlpha[1]
    UIElement[MinPlatPriceText@Sell@GUITabs@MyPrices]:SetAlpha[1]
    UIElement[MinPrice@Sell@GUITabs@MyPrices]:SetChecked
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
        if ${UIElement[MatchLowPrice@Sell@GUITabs@MyPrices].Checked}
        {
            Script[myprices]:QueueCommand[call SaveSetting MatchLowPrice TRUE]
            Script[myprices].VariableScope.MatchLowPrice:Set[TRUE]
        }
        else
        {
            Script[myprices]:QueueCommand[call SaveSetting MatchLowPrice FALSE]
            Script[myprices].VariableScope.MatchLowPrice:Set[FALSE]
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
    Script[myprices].VariableScope.MatchLowPrice:Set[${This.Checked}]
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
        if ${Script[myprices].VariableScope.MerchantMatch}
        {
            UIElement[MerchantMatch@Sell@GUITabs@MyPrices]:SetChecked
        }
        else
        {
            UIElement[MerchantMatch@Sell@GUITabs@MyPrices]:UnsetChecked
        }
    </OnLoad>
</checkbox>
```

**Saving Settings from UI (OnLeftClick):**
```xml
<OnLeftClick>
    if ${UIElement[MerchantMatch@Sell@GUITabs@MyPrices].Checked}
    {
        Script[myprices]:QueueCommand[call SaveSetting MerchantMatch TRUE]
        Script[myprices].VariableScope.MerchantMatch:Set[TRUE]
    }
    else
    {
        Script[myprices]:QueueCommand[call SaveSetting MerchantMatch FALSE]
        Script[myprices].VariableScope.MerchantMatch:Set[FALSE]
    }
</OnLeftClick>
```

**Script-side SaveSetting Function:**
```lavishscript
function SaveSetting(string setting, string value)
{
    General:Set[${LavishSettings[myprices].FindSet[General]}]
    General:AddSetting["${setting}","${value}"]
    LavishSettings[myprices]:Export["${Script.CurrentDirectory}/MyPrices.xml"]
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

**Example from MyPrices:**
```lavishscript
function ColourItem(int position, string UITab, string colour)
{
    UIElement[MyPrices].FindChild[GUITabs].FindChild[${UITab}].FindChild[ItemList].Item[${position}]:SetTextColor[${colour}]
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

**Example from MyPrices:**
```lavishscript
; Show extra options for advanced item types
UIElement[BuyNameOnly@Buy@GUITabs@MyPrices]:Show
UIElement[Harvest@Buy@GUITabs@MyPrices]:Show
UIElement[Transmute@Buy@GUITabs@MyPrices]:Show
UIElement[BuyAttuneOnly@Buy@GUITabs@MyPrices]:Show
UIElement[StartLevelText@Buy@GUITabs@MyPrices]:Show
UIElement[EndLevelText@Buy@GUITabs@MyPrices]:Show
UIElement[StartLevel@Buy@GUITabs@MyPrices]:Show
UIElement[EndLevel@Buy@GUITabs@MyPrices]:Show
UIElement[Tier@Buy@GUITabs@MyPrices]:Show

; Hide them for simple items
UIElement[BuyNameOnly@Buy@GUITabs@MyPrices]:Hide
UIElement[Harvest@Buy@GUITabs@MyPrices]:Hide
UIElement[Transmute@Buy@GUITabs@MyPrices]:Hide
UIElement[BuyAttuneOnly@Buy@GUITabs@MyPrices]:Hide
UIElement[StartLevelText@Buy@GUITabs@MyPrices]:Hide
UIElement[StartLevel@Buy@GUITabs@MyPrices]:Hide
UIElement[EndLevelText@Buy@GUITabs@MyPrices]:Hide
UIElement[EndLevel@Buy@GUITabs@MyPrices]:Hide
UIElement[Tier@Buy@GUITabs@MyPrices]:Hide
```

**Best Practice:**
- Group related Show/Hide calls
- Use functions to encapsulate visibility logic
- Consider Alpha instead of Hide for layout stability

**Example Function:**
```lavishscript
function ShowAdvancedOptions()
{
    UIElement[BuyNameOnly@Buy@GUITabs@MyPrices]:Show
    UIElement[Harvest@Buy@GUITabs@MyPrices]:Show
    UIElement[Transmute@Buy@GUITabs@MyPrices]:Show
}

function HideAdvancedOptions()
{
    UIElement[BuyNameOnly@Buy@GUITabs@MyPrices]:Hide
    UIElement[Harvest@Buy@GUITabs@MyPrices]:Hide
    UIElement[Transmute@Buy@GUITabs@MyPrices]:Hide
}
```

---

## Advanced Patterns from EQ2Craft

### OnRender for Conditional Visibility

**Pattern:** Use `OnRender` to dynamically show/hide elements based on conditions

**Source:** EQ2Craft UI - Zone-based element visibility

```xml
<Text Name='Writ Count Label'>
    <OnLoad>
        declarevariable CG_Frame_Counter int global 0
    </OnLoad>
    <OnUnload>
        deletevariable CG_Frame_Counter
    </OnUnload>
    <OnRender>
        if ${CG_Frame_Counter} == 0
        {
            if ${Script[EQ2Craft].Variable[CraftVersion]} > 8.93
            {
                if ${Zone.ShortName.Find[guildhall]}
                {
                    if !${This.Parent.FindChild[NumWrits].Visible}
                        This.Parent.FindChild[NumWrits]:Show
                    if ${This.Parent.FindChild[NumWritsOOG].Visible}
                        This.Parent.FindChild[NumWritsOOG]:Hide
                }
                else
                {
                    if !${This.Parent.FindChild[NumWritsOOG].Visible}
                        This.Parent.FindChild[NumWritsOOG]:Show
                    if ${This.Parent.FindChild[NumWrits].Visible}
                        This.Parent.FindChild[NumWrits]:Hide
                }
            }
        }
        CG_Frame_Counter:Set[${Math.Calc[(${CG_Frame_Counter}+1)%10]}]
    </OnRender>
</Text>
```

**Key Techniques:**
- **Frame counter** - Only check every N frames to reduce CPU usage
- **Modulo operation** - `(${CG_Frame_Counter}+1)%10` resets counter to 0 every 10 frames
- **Zone detection** - Different UI based on location
- **Parent navigation** - `This.Parent.FindChild[...]` to access siblings

**Why Use This:**
- Avoid constant CPU-intensive checks
- Dynamic UI based on game state
- Cleaner than polling in script

---

### Commandcheckbox with noop Commands

**Pattern:** Execute multiple commands in a single `Command` or `CommandChecked`

```xml
<Commandcheckbox Name='Quality 1'>
    <Text>1</Text>
    <Command>noop ${Script[EQ2Craft].Variable[Quality1]:Set[TRUE]} noop ${Script[EQ2Craft].Variable[Quality2]:Set[FALSE]} noop ${Script[EQ2Craft].Variable[Quality3]:Set[FALSE]} noop ${Script[EQ2Craft].Variable[Quality4]:Set[FALSE]}</Command>
    <CommandChecked />
    <Data>${Script[EQ2Craft].Variable[Quality1]}</Data>
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
<Slider Name='WritSlider'>
    <X>30</X>
    <Y>35</Y>
    <Width>120</Width>
    <Height>20</Height>
    <Range>24</Range>
    <OnLoad>
        This:SetValue[${Math.Calc[${This.Parent.FindChild[NumWrits].Text} - 1].Int}]
        This.Parent.FindChild[NumWritsOOG]:SetText[${Math.Calc[(${This.Value}/2)+1].Int}]
    </OnLoad>
    <OnChange>
        variable int ThisValue=${Math.Calc[${This.Value}+1].Int}
        This.Parent.FindChild[NumWrits]:SetText[${ThisValue}]
        This.Parent.FindChild[NumWritsOOG]:SetText[${Math.Calc[(${This.Value}/2)+1].Int}]
        LavishSettings[Craft Config File].FindSet[General Options].FindSetting[How many Writs to create per craft session?]:Set[${ThisValue}]
        Script[EQ2Craft].Variable[WritCount]:Set[${ThisValue}]
        LavishSettings[Craft Config File]:Export[${Script[EQ2Craft].Variable[configfile]}]
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
    <OnLoad>This:SetText[${Script[EQ2Craft].Variable[CampTimer]}]</OnLoad>
    <OnChange>
        LavishSettings[Craft Config File].FindSet[General Options].FindSetting[Camp out after a specified time has elapsed for a crafting session?]:Set[${This.Text}]
        Script[EQ2Craft].Variable[CampTimer]:Set[${This.Text}]
        LavishSettings[Craft Config File]:Export[${Script[EQ2Craft].Variable[configfile]}]
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
    <Data>${Script[EQ2Craft].Variable[gaugelevel]}</Data>
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

**Pattern:** Execute broker search when selecting from list

```xml
<Listbox Name='Harvest Resource List'>
    <X>0</X>
    <Y>16</Y>
    <Width>100%</Width>
    <Height>88</Height>
    <SelectMultiple>1</SelectMultiple>
    <OnSelect>
        broker Name "${This.Item[${ID}].Text.Token[1,|]}" Sort ByPriceAsc MinTier Handcrafted MaxTier Treasured
    </OnSelect>
    <Sort>Text</Sort>
</Listbox>
```

**Key Techniques:**
- `${This.Item[${ID}].Text}` - Get selected item text
- `.Token[1,|]` - Parse item text (format: "ItemName|Quantity")
- Execute game command directly from OnSelect
- `SelectMultiple` allows multiple selections

**Common List Patterns:**
```lavishscript
; Get selected item text
${This.Item[${ID}].Text}

; Get specific token from item text (format: "Name|Value|Color")
${This.Item[${ID}].Text.Token[1,|]}  ; Name
${This.Item[${ID}].Text.Token[2,|]}  ; Value

; Execute command with item data
broker Name "${This.Item[${ID}].Text.Token[1,|]}" Sort ByPriceAsc
```

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
            Script[EQ2Craft]:QueueCommand[Craft:AddRecipe]
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
    <AutoTooltip>Conversation choice for writ selection, starting with 1 at the top.</AutoTooltip>
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
- `<AutoTooltip>` for single-line tooltips
- `<Autotooltip>` (lowercase 't') for multi-line tooltips
- No script code needed
- Great for user help

---

### Combobox with OnLoad Selection

**Pattern:** Initialize combobox to specific value on load

```xml
<Combobox Name='Queue Class'>
    <X>100</X>
    <Y>10</Y>
    <Width>80</Width>
    <Height>23</Height>
    <Fullheight>185</Fullheight>
    <ButtonWidth>20</ButtonWidth>
    <OnLoad>
        This.ItemByText[ALL]:Select
        This.ItemByText[${Me.TSSubClass}]:Select
    </OnLoad>
    <OnSelect>
        Script[EQ2Craft].VariableScope.Craft:PopulateFavorites[${This.SelectedItem.Text}]
    </OnSelect>
    <Items>
        <Item Value='1'>ALL</Item>
        <Item Value='2'>Alchemist</Item>
        <Item Value='3'>Armorer</Item>
        <Item Value='4'>Carpenter</Item>
        <!-- ... more items -->
    </Items>
</Combobox>
```

**Key Techniques:**
- `This.ItemByText[...]` - Select by text (not value)
- Multiple `:Select` calls (last one wins)
- `${Me.TSSubClass}` - Select based on character class
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
            <Command>Script[EQ2Craft]:End</Command>
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

## Advanced Patterns from EQ2Bot

### Checkbox-Controlled Frame Visibility

**Pattern:** Use checkbox to show/hide associated input frame

**Source:** EQ2Bot - MainTank and MainAssist selection

```xml
<checkbox Name='MainTank'>
    <X>5%</X>
    <Y>10</Y>
    <Text>Main Tank:</Text>
    <OnLoad>
        if ${Script[EQ2Bot].VariableScope.MainTank}
        {
            This:SetChecked
        }
    </OnLoad>
    <OnLeftClick>
        if ${This.Checked}
        {
            UIElement[EQ2 Bot].FindUsableChild[MainTank Frame,Frame]:Hide
            Script[EQ2Bot].Variable[MainTank]:Set[TRUE]
            LavishSettings[EQ2Bot].FindSet[Character].FindSet[General Settings]:AddSetting[I am the Main Tank?,TRUE]
            Script[EQ2Bot].VariableScope.EQ2Bot:Save_Settings
        }
        else
        {
            UIElement[EQ2 Bot].FindUsableChild[MainTank Frame,Frame]:Show
            Script[EQ2Bot].Variable[MainTank]:Set[FALSE]
            LavishSettings[EQ2Bot].FindSet[Character].FindSet[General Settings]:AddSetting[I am the Main Tank?,FALSE]
            Script[EQ2Bot].VariableScope.EQ2Bot:Save_Settings
        }
    </OnLeftClick>
</checkbox>

<Frame Name='MainTank Frame'>
    <OnLoad>
        if ${Script[EQ2Bot].VariableScope.MainTank}
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

**Pattern:** Populate combobox with current group/raid members when clicked

```xml
<combobox name='MainAssist'>
    <OnLoad>
        if ${Actor[exactname,${Script[EQ2Bot].Variable[MainAssist]}].Name(exists)}
        {
            This:AddItem[${Script[EQ2Bot].Variable[MainAssist]}]
            This.ItemByText[${Script[EQ2Bot].Variable[MainAssist]}]:Select
        }
    </OnLoad>
    <OnSelect>
        Script[EQ2Bot].Variable[MainAssist]:Set[${This.SelectedItem.Text}]
        Script[EQ2Bot].Variable[MainAssistID]:Set[${Actor[exactname,${Script[EQ2Bot].Variable[MainAssist]}].ID}]
        LavishSettings[EQ2Bot].FindSet[Character].FindSet[General Settings]:AddSetting[Who is the Main Assist?,${This.SelectedItem.Text}]
        Script[EQ2Bot].VariableScope.EQ2Bot:Save_Settings
    </OnSelect>
    <OnLeftClick>
        declare tmpvar int
        This:ClearItems
        tmpvar:Set[1]

        ; Populate from raid
        if ${Me.InRaid}
        {
            do
            {
                if ${Me.Raid[${tmpvar}].InZone}
                {
                    This:AddItem[${Me.Raid[${tmpvar}].Name}]
                }
            }
            while ${tmpvar:Inc} &lt;= ${Me.Raid}
        }
        ; Populate from group
        elseif ${Me.Group} > 1
        {
            do
            {
                if ${Me.Group[${tmpvar}].InZone}
                {
                    This:AddItem[${Me.Group[${tmpvar}].Name}]
                }
                ; Include pets
                if ${Me.Group[${tmpvar}].PetID(exists)}
                {
                    if ${Actor[${Me.Group[${tmpvar}].PetID}].Name(exists)}
                    {
                        This:AddItem[${Me.Group[${tmpvar}].Pet.Name}]
                    }
                }
            }
            while ${tmpvar:Inc} &lt; ${Me.Group}
        }

        ; Include own pet
        if ${Me.Pet(exists)}
        {
            This:AddItem[${Me.Pet.Name}]
        }

        This:Sort
    </OnLeftClick>
</combobox>
```

**Key Techniques:**
- Clear items on click
- Check raid first, then group
- Include pets as options
- Sort alphabetically
- Declare local variable in OnLeftClick

**Why Use This:**
- Always show current group/raid members
- Handles group/raid changes
- Includes pets dynamically
- Cleaner than pre-populating

---

###OnRender for Dynamic Window Sizing

**Pattern:** Adjust window height based on active tab

```xml
<Window name='EQ2 Bot'>
    <Width>500</Width>
    <Height>500</Height>
    <OnRender>
        if ${UIElement[EQ2 Bot].Minimized}
        {
            UIElement[Title@TitleBar@EQ2 Bot]:SetText[EQ2Bot: ${Script[EQ2Bot].Variable[Me_Name]}]
        }
        else
        {
            UIElement[Title@TitleBar@EQ2 Bot]:SetText[EQ2Bot - ${Script[EQ2Bot].Variable[Me_Name]}]

            if ${UIElement[EQ2 Bot].FindChild[EQ2Bot Tabs].FindChild[Main].Visible}
            {
                if ${UIElement[EQ2 Bot].Height} != 350
                {
                    UIElement[EQ2 Bot]:SetHeight[350]
                }
            }
            elseif ${UIElement[EQ2 Bot].FindChild[EQ2Bot Tabs].FindChild[EQ2Bot Options].Visible}
            {
                if ${UIElement[EQ2 Bot].Height} != 350
                {
                    UIElement[EQ2 Bot]:SetHeight[350]
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
    <Text>EQ2's Auto Follow:</Text>
    <AutoTooltip>Select to use EQ2 AutoFollow on the Main Assist instead of EQ2bot Follow</AutoTooltip>
    <OnLeftClick>
        if ${This.Checked}
        {
            Script[EQ2Bot].Variable[AutoFollowMode]:Set[TRUE]
            LavishSettings[EQ2Bot].FindSet[Character].FindSet[EQ2BotExtras]:AddSetting["Auto Follow Mode",TRUE]
            Script[EQ2Bot].VariableScope.EQ2Bot:Save_Settings
        }
        else
        {
            Script[EQ2Bot].Variable[AutoFollowMode]:Set[FALSE]
            LavishSettings[EQ2Bot].FindSet[Character].FindSet[EQ2BotExtras]:AddSetting["Auto Follow Mode",FALSE]
            Script[EQ2Bot].VariableScope.EQ2Bot:Save_Settings
        }
    </OnLeftClick>
    <Data>${LavishSettings[EQ2Bot].FindSet[Character].FindSet[EQ2BotExtras].FindSetting[Auto Follow Mode]}</Data>
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
    LavishSettings[EQ2Bot].FindSet[Character].FindSet[General Settings]:AddSetting[Who is the Main Assist?,${This.SelectedItem.Text}]

    ; Call script method to save
    Script[EQ2Bot].VariableScope.EQ2Bot:Save_Settings
</OnSelect>
```

**Settings Structure:**
```
LavishSettings[EQ2Bot]
└── Character (Character Name)
    ├── General Settings
    │   ├── I am the Main Tank?
    │   ├── Who is the Main Tank?
    │   ├── I am the Main Assist?
    │   └── Who is the Main Assist?
    └── EQ2BotExtras
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
    <OnLoad>This:SetText[${Script[EQ2Bot].Variable[AssistHP]}]</OnLoad>
    <OnChange>
        LavishSettings[EQ2Bot].FindSet[Character].FindSet[General Settings]:AddSetting[Assist and Engage in combat at what Health?,${This.Text}]
        Script[EQ2Bot].VariableScope.EQ2Bot:Save_Settings
        Script[EQ2Bot].Variable[AssistHP]:Set[${This.Text}]
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
        if ${Script[EQ2Bot].Variable[PathType]}
        {
            UIElement[EQ2 Bot].FindUsableChild[Following Frame,Frame]:Hide
        }
        else
        {
            UIElement[EQ2 Bot].FindUsableChild[Following Frame,Frame]:Show
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
<Commandbutton name='Feign Death'>
    <X>65%</X>
    <Y>120</Y>
    <Width>115</Width>
    <Height>21</Height>
    <Visible>0</Visible>
    <Text>Feign Death</Text>
    <OnLeftClick>
        Script[EQ2Bot]:QueueCommand[call FeignDeath]
    </OnLeftClick>
</Commandbutton>
```

**Script-Side Control:**
```lavishscript
; Show button for Monk class
if ${Me.SubClass.Equal[Monk]}
{
    UIElement[EQ2 Bot].FindUsableChild[Feign Death,commandbutton]:Show
}

; Hide button for other classes
else
{
    UIElement[EQ2 Bot].FindUsableChild[Feign Death,commandbutton]:Hide
}
```

**Key Techniques:**
- Start hidden with `<Visible>0</Visible>`
- Script shows based on class/conditions
- Class-specific abilities
- Dynamic UI based on character

---

## Production Patterns

### Status Text Updates

**Pattern:** Centralized status message display

**Example from MyPrices:**
```lavishscript
; Different status messages throughout execution
UIElement[Errortext@Sell@GUITabs@MyPrices]:SetText["** Waiting **"]
UIElement[Errortext@Sell@GUITabs@MyPrices]:SetText[" **Processing**"]
UIElement[Errortext@Sell@GUITabs@MyPrices]:SetText[" ** Finished **"]
UIElement[Errortext@Sell@GUITabs@MyPrices]:SetText["Placing Items"]
UIElement[Errortext@Sell@GUITabs@MyPrices]:SetText[" **Paused**"]
UIElement[Errortext@Sell@GUITabs@MyPrices]:SetText["Pause ${Math.Calc[${WaitTimer}/600]} Mins"]
```

**Better Pattern with Function:**
```lavishscript
function UpdateStatus(string message, string colour="FFFFFFFF")
{
    UIElement[Errortext@Sell@GUITabs@MyPrices]:SetText["${message}"]
    UIElement[Errortext@Sell@GUITabs@MyPrices]:SetTextColor[${colour}]
}

; Usage
call UpdateStatus "** Waiting **" FFFFFF00
call UpdateStatus "Processing" FF00FF00
call UpdateStatus "Error occurred" FFFF0000
```

---

### List Population Pattern

**Pattern:** Clear and rebuild lists efficiently

**Example from MyPrices:**
```lavishscript
function UpdateItemList(string tabname)
{
    ; Clear existing list
    UIElement[ItemList@${tabname}@GUITabs@MyPrices]:ClearItems

    ; Iterate settings and populate
    LavishSettings[myprices].FindSet[Item]:GetSetIterator[ItemIterator]

    if ${ItemIterator:First(exists)}
    {
        do
        {
            ItemName:Set["${ItemIterator.Key}"]
            UIElement[ItemList@${tabname}@GUITabs@MyPrices]:AddItem["${ItemName}"]
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

**Example from MyPrices:**
```lavishscript
function main()
{
    ; Load settings
    MyPrices:loadsettings

    ; Backup immediately after load
    LavishSettings[myprices]:Export[${BackupPath}${EQ2.ServerName}_${Me.Name}_MyPrices.XML]

    ; Continue with script
    MyPrices:LoadUI
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

    LavishSettings[myprices]:Export[${BackupPath}${Me.Name}_${timestamp}.XML]
}
```

---

## Summary

These advanced patterns from the [MyPrices](https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/MyPrices) script demonstrate:

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

---

## Additional Resources

**Script Source:**
- https://github.com/isxGames/isxScripts/tree/master/EverQuest2/Scripts/MyPrices

**Related Guides:**
- [LavishGUI 1 UI Guide](08_LavishGUI1_UI_Guide.md)
- [Advanced Scripting Patterns](13_Advanced_Scripting_Patterns.md)
- [LavishSettings Guide](01_LavishScript_Fundamentals.md)

---

*Last Updated: 2025-10-23*
*Source: MyPrices v0.15d analysis*
