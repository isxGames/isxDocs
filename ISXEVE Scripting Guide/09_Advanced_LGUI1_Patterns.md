# Advanced LavishGUI 1 Patterns and Techniques

**Purpose:** Real-world patterns from production EVE Online scripts
**Sources:**
- [EVEBot](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable) - Combat/mining/mission bot (2,974 line main UI)
- [WreckingBall2](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/WreckingBall2) - Salvaging bot (328 line UI)
- [Yamfa](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/Yamfa) - Multi-client automation (326 line UI)

**Target Audience:** Advanced script developers

---

## Table of Contents

1. [UI Element Navigation](#ui-element-navigation)
2. [Two-Way Data Binding](#two-way-data-binding)
3. [Dynamic UI Updates](#dynamic-ui-updates)
4. [Settings Integration](#settings-integration)
5. [Console and Logging](#console-and-logging)
6. [Advanced Button Patterns](#advanced-button-patterns)
7. [Alpha Transparency Effects](#alpha-transparency-effects)
8. [Multi-Client Relay Commands](#multi-client-relay-commands)
9. [Nested TabControls](#nested-tabcontrols)
10. [Slider with Synchronized Labels](#slider-with-synchronized-labels)
11. [Advanced List Management](#advanced-list-management)
12. [Advanced TextEntry Patterns](#advanced-textentry-patterns)
13. [ComboBox Advanced Patterns](#combobox-advanced-patterns)
14. [Production Patterns](#production-patterns)

---

## UI Element Navigation

### Deep Element Access with FindUsableChild

**Pattern:** Navigate complex UI hierarchies using `FindUsableChild`

**Example from EVEBot:**
```lavishscript
; Access nested element in tab control
UIElement[EVEBot].FindUsableChild[CurrentBehavior,combobox]:ClearItems
UIElement[EVEBot].FindUsableChild[CurrentBehavior,combobox]:AddItem["Idle"]
UIElement[EVEBot].FindUsableChild[CurrentBehavior,combobox].ItemByText[${Config.Common.CurrentBehavior}]:Select
```

**GitHub Reference:** [obj_EVEBotUI.iss](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable/core/obj_EVEBotUI.iss)

**Pattern:**
```lavishscript
; Format: UIElement[Window].FindUsableChild[ElementName,elementtype]
UIElement[<WindowName>].FindUsableChild[<ElementName>,<ElementType>]
```

**Why Use This:**
- Type-specific element searching
- Works across complex hierarchies
- More robust than @ notation for dynamic UIs
- Can skip intermediate parents

**When to Use @ Notation vs FindUsableChild:**

**@ Notation (Faster, but brittle):**
```lavishscript
UIElement[StatusText@Status@EVEBotOptionsTab@EVEBot]:SetText["Running"]
```

**FindUsableChild (Slower, but more robust):**
```lavishscript
UIElement[EVEBot].FindUsableChild[StatusText,text]:SetText["Running"]
```

Use `@` for simple, frequently accessed elements. Use `FindUsableChild` for:
- Elements nested deeply in tabs
- Dynamic element location
- When hierarchy might change
- Type-specific searches

---

## Two-Way Data Binding

### Textentry with Bidirectional Binding

**Pattern:** Synchronize text input with script variables

**Example from [WreckingBall2](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/WreckingBall2):**
```xml
<textentry name='HomeIdentifiers'>
    <X>5</X>
    <Y>20</Y>
    <Width>45%</Width>
    <Height>20</Height>
    <OnLoad>This:SetText[${Script[wreckingball2].VariableScope.HomeBookmarkSymbol}]</OnLoad>
    <OnChange>Script[wreckingball2].VariableScope.HomeBookmarkSymbol:Set[${This.Text.Escape}]</OnChange>
</textentry>
```

**GitHub Reference:** [BotUI.xml](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/WreckingBall2/UI/BotUI.xml)

**Key Techniques:**
- `OnLoad` - Initialize UI from script variable
- `OnChange` - Update script variable when user types
- `${This.Text.Escape}` - Safe text retrieval with escaping
- `Script[scriptname].VariableScope.VarName` - Direct variable access

**Pattern:**
```xml
<textentry name='MyInput'>
    <OnLoad>This:SetText[${Script[MyScript].VariableScope.MyVariable}]</OnLoad>
    <OnChange>Script[MyScript].VariableScope.MyVariable:Set[${This.Text}]</OnChange>
</textentry>
```

**Benefits:**
- No manual sync needed
- UI always reflects script state
- Script always has current UI value
- Real-time updates

---

## Dynamic UI Updates

### Stateful Toggle Buttons

**Pattern:** Buttons that remember and display their state

**Example from [WreckingBall2](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/WreckingBall2):**
```xml
<button name='DebugOn'>
    <X>70%</X>
    <Y>95%</Y>
    <Width>100</Width>
    <Height>20</height>
    <Text> </Text>
    <OnLeftClick>
        if ${Script[wreckingball2].VariableScope.ShowDebug}
        {
            This:SetText[DontShow]
            Script[wreckingball2].VariableScope.ShowDebug:Set[FALSE]
        }
        else
        {
            This:SetText[ShowDebug]
            Script[wreckingball2].VariableScope.ShowDebug:Set[TRUE]
        }
    </OnLeftClick>
    <OnLoad>
        if ${Script[wreckingball2].VariableScope.ShowDebug}
            This:SetText[ShowDebug]
        else
            This:SetText[DontShow]
    </OnLoad>
</button>
```

**GitHub Reference:** [BotUI.xml](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/WreckingBall2/UI/BotUI.xml)

**Key Techniques:**
- Button text shows current state
- OnLoad ensures UI matches script state
- OnLeftClick toggles both UI and variable
- Persistent state across sessions

**Pattern for Toggle Buttons:**
```xml
<button name='FeatureToggle'>
    <OnLoad>
        if ${Script[MyScript].VariableScope.FeatureEnabled}
            This:SetText[Enabled]
        else
            This:SetText[Disabled]
    </OnLoad>
    <OnLeftClick>
        if ${Script[MyScript].VariableScope.FeatureEnabled}
        {
            This:SetText[Disabled]
            Script[MyScript].VariableScope.FeatureEnabled:Set[FALSE]
        }
        else
        {
            This:SetText[Enabled]
            Script[MyScript].VariableScope.FeatureEnabled:Set[TRUE]
        }
    </OnLeftClick>
</button>
```

---

## Settings Integration

### LavishSettings with OnChange Auto-Save

**Pattern:** Automatically save settings when values change

**Example from EVEBot:**
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

**Key Techniques:**
- Update script variable in OnChange
- Call config object methods for validation
- No "Save" button needed
- Changes are immediate

**Pattern:**
```xml
<OnChange>
    ; 1. Update script variable
    Script[MyScript].VariableScope.Config.MySetting:Set[${This.Text}]

    ; 2. Call save method
    Script[MyScript].VariableScope.Config:Save
</OnChange>
```

**Why Use This:**
- User doesn't lose unsaved changes
- Simplified UI (no save button)
- Instant feedback
- Safer for production use

---

## Console and Logging

### Console Output from Script

**Pattern:** Real-time logging to UI console element

**Example from [WreckingBall2](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/WreckingBall2):**
```lavishscript
; Output to console in UI
UIElement[StatusConsole@Main@WreckingTabControl@WreckingBot]:Echo["${This.Runtime} - ${Message}"]
```

**Console Definition in XML:**
```xml
<console name='StatusConsole'>
    <X>5</X>
    <Y>20%</Y>
    <Width>r10</Width>
    <Height>75%</Height>
</console>
```

**GitHub Reference:** [BotUI.xml](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/WreckingBall2/UI/BotUI.xml), [theDebug.iss](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/WreckingBall2/Debug/theDebug.iss)

**Console Properties:**
- `Width="r10"` - Relative width (window width minus 10 pixels)
- `Height="75%"` - Percentage of parent height
- `:Echo[text]` - Add line to console
- Auto-scrolling output

**Use Cases:**
- Status updates
- Error messages
- Debug logging
- Operation history

---

## Advanced Button Patterns

### Dual-Action Buttons (Left/Right Click)

**Pattern:** Different actions for left and right click

**Example from Yamfa:**
```xml
<button name='Defense'>
    <OnLeftClick>
        relay all Script[yamfa].VariableScope.Config.Override:Set[${This.Name},TRUE]
        Script[yamfa].VariableScope.UI.Alpha:Set[${This.Name}, 0]
    </OnLeftClick>
    <OnRightClick>
        relay all Script[yamfa].VariableScope.Config.Override:Set[${This.Name},FALSE]
        Script[yamfa].VariableScope.UI.Alpha:Set[${This.Name}, 0]
    </OnRightClick>
</button>
```

**GitHub Reference:** [ui.xml](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/Yamfa/ui.xml)

**Common Patterns:**
- Left = Enable, Right = Disable
- Left = Primary action, Right = Secondary action
- Left = Approach, Right = Activate
- Left = Warp to safe, Right = Dock

**Why Use This:**
- Compact UI with rich functionality
- Intuitive enable/disable paradigm
- Reduces button count
- Natural user interaction

### Custom Close Button Actions

**Pattern:** Custom close behavior instead of default window close

**Example from EVEBot:**
```xml
<button Name='Close' template='EVESkin.Window.TitleBar.Close'>
    <OnLeftClick>
        EVEBot:EndBot[]
    </OnLeftClick>
</button>
```

**GitHub Reference:** [EVEBot.xml](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable/interface/EVEBot.xml)

**Why Use This:**
- Run cleanup before closing
- Proper script shutdown
- Save state before exit
- Different from just hiding window

---

## Alpha Transparency Effects

### Smooth Fade-In Animation

**Pattern:** Progressive alpha transparency for visual feedback

**Example from Yamfa:**

**XML Side:**
```xml
<button name='Defense'>
    <OnRender>
        if !${Script[yamfa].VariableScope.UI.Alpha.Element[${This.Name}](exists)}
            Script[yamfa].VariableScope.UI.Alpha:Set[${This.Name}, 0]
        This:SetAlpha[${Script[yamfa].VariableScope.UI.Alpha.Element[${This.Name}]}]
    </OnRender>
    <OnMouseEnter>
        Script[yamfa].VariableScope.UI.Alpha:Set[${This.Name}, 1]
    </OnMouseEnter>
    <OnMouseExit>
        Script[yamfa].VariableScope.UI.Alpha:Set[${This.Name}, .8]
    </OnMouseExit>
</button>
```

**Script Side (in Update method):**
```lavishscript
variable collection:float Alpha

method Update()
{
    if ${Alpha.FirstKey(exists)}
    do
    {
        if ${Alpha.CurrentValue.Equal[1]}
            continue
        if ${Alpha.CurrentValue} < 0.8
            Alpha.CurrentValue:Inc[.111]
        if ${Alpha.CurrentValue} > 0.8
            Alpha.CurrentValue:Set[0.8]
    }
    while ${Alpha.NextKey(exists)}
}
```

**GitHub Reference:** [ui.xml](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/Yamfa/ui.xml), [Yamfa.iss](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/Yamfa/Yamfa.iss)

**Key Techniques:**
- Collection stores alpha values per button
- OnRender applies alpha every frame
- Mouse enter/exit trigger alpha changes
- Update method creates smooth transitions

**Common Alpha Values:**
- `1.0` - Fully visible (active/hover)
- `0.8` - Semi-transparent (visible but not focused)
- `0.1` - Nearly invisible (disabled)
- `0.0` - Invisible (hidden)

**Use Cases:**
- Hover effects
- Disabled state indicators
- Smooth show/hide transitions
- "Discoverable" UI elements

---

## Multi-Client Relay Commands

### Broadcasting Commands to All Clients

**Pattern:** Synchronize multiple EVE clients with relay commands

**Example from Yamfa:**
```xml
<button name='Retreat'>
    <OnLeftClick>
        relay all Script[yamfa].VariableScope.Commands:Set[Retreat,Retreat,Safe]
        Script[yamfa].VariableScope.UI.Alpha:Set[${This.Name}, 0]
    </OnLeftClick>
    <OnRightClick>
        relay all Script[yamfa].VariableScope.Commands:Set[Retreat,Retreat,Dock]
        Script[yamfa].VariableScope.UI.Alpha:Set[${This.Name}, 0]
    </OnRightClick>
</button>
```

**GitHub Reference:** [ui.xml](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/Yamfa/ui.xml)

**Relay Command Syntax:**
```lavishscript
relay <target> <command>

; Examples:
relay all EVEBot:Resume["Clicked Run"]        ; All clients
relay other endscript yamfa                    ; All except current
relay "Character Name" call SomeFunction       ; Specific character
```

**Common Patterns:**
```xml
<!-- Enable feature on all clients -->
<OnLeftClick>
    relay all Script[MyScript].VariableScope.FeatureEnabled:Set[TRUE]
</OnLeftClick>

<!-- Queue command on all clients -->
<OnLeftClick>
    relay all Script[MyScript]:QueueCommand[call DoSomething]
</OnLeftClick>

<!-- Update config on all clients -->
<OnLeftClick>
    relay all Script[MyScript].VariableScope.Config.Setting:Set[${This.Text}]
</OnLeftClick>
```

**Use Cases:**
- Multi-boxing fleet commands
- Synchronized breaks/pauses
- Formation changes
- Emergency retreat

---

## Nested TabControls

### Tab Within Tab Pattern

**Pattern:** Organize complex options with nested tab structures

**Example from EVEBot:**
```xml
<Tab Name='Combat'>
  <TabControl Name='FleeingTabcontrol' template='EVESkin.TabControl'>
    <x>0</x>
    <y>0</y>
    <width>100%</width>
    <height>100%</height>
    <Tabs>
      <Tab name='Options'>
        <frame name='FleeingFrame'>
          <x>0</x>
          <y>0</y>
          <width>100%</width>
          <height>100%</height>
          <children>
            <!-- Fleeing options here -->
          </children>
        </frame>
      </Tab>
      <Tab name='Thresholds'>
        <!-- Threshold settings here -->
      </Tab>
    </Tabs>
  </TabControl>
</Tab>
```

**GitHub Reference:** [EVEBot.xml](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable/interface/EVEBot.xml)

**Key Techniques:**
- Outer tab contains inner TabControl
- Inner TabControl sized to 100% of parent
- Use frames within nested tabs for content
- Maintains clean visual hierarchy

**Access Pattern:**
```lavishscript
; Access element in nested tab
UIElement[EVEBot].FindUsableChild[FleeingTabcontrol,tabcontrol].Tab[Options].FindUsableChild[FleeingFrame,frame]
```

**Use Cases:**
- Complex settings screens
- Category-within-category organization
- Large number of options
- Logical grouping of related settings

---

## Slider with Synchronized Labels

### Real-Time Slider Feedback

**Pattern:** Update text labels dynamically as slider moves

**Example from EVEBot:**
```xml
<slider name='MinArmorPct'>
  <X>210</X>
  <Y>10</Y>
  <Width>40</Width>
  <Height>18</Height>
  <Range>100</Range>
  <OnLoad>
    This:SetValue[${Script[EVEBot].VariableScope.Config.Combat.MinimumArmorPct}]
    UIElement[EVEBot].FindUsableChild[MinArmorPctLabel,Text]:SetText["Minimum Armor: ${This.Value}"]
  </OnLoad>
  <OnChange>
    Script[EVEBot].VariableScope.Config.Combat:SetMinimumArmorPct[${Int[${This.Value}]}]
    UIElement[EVEBot].FindUsableChild[MinArmorPctLabel,Text]:SetText["Minimum Armor: ${This.Value}"]
  </OnChange>
</slider>
<Text name='MinArmorPctLabel'>
  <X>255</X>
  <Y>10</Y>
  <Width>150</Width>
  <Height>10</Height>
  <Text>Minimum Armor: 0</Text>
  <AutoTooltip>Lowest armor percent allowed before fleeing</AutoTooltip>
  <OnLoad>
    This:SetText["Minimum Armor: ${UIElement[EVEBot].FindUsableChild[MinArmorPct,slider].Value}"]
  </OnLoad>
</Text>
```

**GitHub Reference:** [EVEBot.xml](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable/interface/EVEBot.xml)

**Key Techniques:**
- Update label in both slider's OnLoad and OnChange
- Label also initializes from slider value
- Use FindUsableChild for robust element access
- Immediate visual feedback

**Advanced: Unit Conversion**
```xml
<OnChange>
  Script[EVEBot].VariableScope.Config.Miner:SetAvoidPlayerRange[${Int[${This.Value}]}]
  UIElement[AvoidPlayerRangeLabel@Miner@EVEBotOptionsTab@EVEBot]:SetText["Min. Distance: ${EVEBot.MetersToKM_Str[${This.Value}]}"]
</OnChange>
```

Call script methods for display formatting (meters to kilometers, etc.)

---

## Advanced List Management

### Listbox with Iterator Population

**Pattern:** Populate listboxes from settings or collections

**Example from EVEBot:**
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

**Pattern:**
```lavishscript
; In OnLoad event
variable iterator MyIterator
Collection:GetIterator[MyIterator]

if ${MyIterator:First(exists)}
{
    do
    {
        This:AddItem[${MyIterator.Value}]
    }
    while ${MyIterator:Next(exists)}
}

This:Sort
```

**Best Practices:**
- Always use `:First(exists)` check
- Clear list before repopulating
- Sort after adding all items
- Use iterators for collections/settings

---

## Advanced TextEntry Patterns

### OnKeyDown for Enter Key Detection

**Pattern:** Execute commands when user presses Enter in text field

**Example from Yamfa:**
```xml
<textentry name='SetDest'>
  <X>2</X>
  <Y>4</Y>
  <Width>96</Width>
  <Height>18</Height>
  <OnKeyDown>
    if ${Key.Equal["enter"]}
    {
      if ${EVE.Bookmark[${This.Text.Escape}](exists)}
        Relay all "EVE.Bookmark[${This.Text.Escape}]:SetDestination"
      else
        Relay all "Universe[${This.Text.Escape}]:SetDestination"
      This:SetText[""]
    }
  </OnKeyDown>
</textentry>
```

**GitHub Reference:** [ui.xml](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/Yamfa/ui.xml)

**Key Techniques:**
- `${Key.Equal["enter"]}` - Detect Enter key press
- Conditional command routing based on input
- `${This.Text.Escape}` - Safe text retrieval
- `This:SetText[""]` - Clear field after command
- Execute commands on Enter, not on every keystroke

**Common Key Checks:**
```lavishscript
${Key.Equal["enter"]}     ; Enter key
${Key.Equal["escape"]}    ; Escape key
${Key.Equal["tab"]}       ; Tab key
${Key.Equal["space"]}     ; Space bar
```

**Use Cases:**
- Command input fields
- Quick destination setters
- Search boxes
- Chat-like interfaces

---

## ComboBox Advanced Patterns

### Value-Based Selection

**Pattern:** Store both display text and numeric values

**Example from EVEBot:**
```xml
<combobox name='cbLowestStanding'>
  <X>210</X>
  <Y>130</Y>
  <Width>50</Width>
  <Height>15</Height>
  <FullHeight>200</FullHeight>
  <ButtonWidth>20</ButtonWidth>
  <Items>
    <Item Value='-11'>-11</Item>
    <Item Value='-10'>-10</Item>
    <Item Value='-5'>-5</Item>
    <Item Value='0'>0</Item>
    <Item Value='5'>5</Item>
    <Item Value='10'>10</Item>
  </Items>
  <OnSelect>
    Script[EVEBot].VariableScope.Config.Miner:SetLowestStanding[${This.SelectedItem.Value}]
  </OnSelect>
  <OnLoad>
    This:SetSelection[${This.ItemByText[${Script[EVEBot].VariableScope.Config.Miner.LowestStanding}].ID}]
  </OnLoad>
</combobox>
```

**GitHub Reference:** [EVEBot.xml](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable/interface/EVEBot.xml)

**Key Techniques:**
- `<Item Value='X'>DisplayText</Item>` - Separate value and display
- `${This.SelectedItem.Value}` - Get item value (not text)
- `${This.SelectedItem.Text}` - Get display text
- `This:SetSelection[ID]` - Select by item ID number
- `This.ItemByText[text].ID` - Get ID from text

**Selection Methods:**
```lavishscript
; Select by item ID (1-based index)
This:SetSelection[${This.ItemByText[SomeText].ID}]

; Get selected value
${This.SelectedItem.Value}

; Get selected text
${This.SelectedItem.Text}

; Get selected ID
${This.SelectedItem.ID}
```

**Use Cases:**
- Numeric ranges with labels
- Enum-like selections
- ID-based lookups
- When display text differs from stored value

---

## Production Patterns

### Dynamic Text Display with Conditionals

**Pattern:** Show different text based on game state

**Example from [WreckingBall2](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/WreckingBall2):**
```xml
<text name='ShipMode'>
    <X>10</X>
    <Y>95%</Y>
    <Width>300</Width>
    <Height>20</Height>
    <Text>${If[${Me.InSpace},${Me.ToEntity.Mode},DOCKED]}</Text>
</text>
```

**GitHub Reference:** [BotUI.xml](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/WreckingBall2/UI/BotUI.xml)

**Pattern:**
```xml
<text name='StatusDisplay'>
    <Text>${If[condition,TrueText,FalseText]}</Text>
</text>

<!-- Nested conditionals -->
<Text>${If[${Condition1},Text1,${If[${Condition2},Text2,Text3]}]}</Text>
```

**Common Use Cases:**
- Ship mode (Docked/Warping/Flying)
- Script state display
- Connection status
- Combat status

### ComboBox with Script Integration

**Pattern:** Dropdown selection updates script state

**Example from EVEBot:**
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

**Pattern:**
```xml
<OnSelect>
    if ${This.SelectedItem.Text.NotNULLOrEmpty}
    {
        Script[MyScript].VariableScope.Mode:Set[${This.SelectedItem.Text}]
        Script[MyScript]:QueueCommand[call ChangedMode]
    }
</OnSelect>
```

### Dynamic Text Input with Polymorphic Routing

**Pattern:** Text input that handles multiple command types

**Example from Yamfa:**
```xml
<textentry name='SetDest'>
    <X>2</X>
    <Y>4</Y>
    <Width>96</Width>
    <Height>18</Height>
    <OnKeyDown>
        if ${Key.Equal["enter"]}
        {
            if ${EVE.Bookmark[${This.Text.Escape}](exists)}
                Relay all "EVE.Bookmark[${This.Text.Escape}]:SetDestination"
            else
                Relay all "Universe[${This.Text.Escape}]:SetDestination"
            This:SetText[""]
        }
    </OnKeyDown>
</textentry>
```

**GitHub Reference:** [ui.xml](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/Yamfa/ui.xml)

**Key Techniques:**
- Check for Enter key press
- Conditional command routing (bookmark vs system)
- Self-clearing input field
- Text escaping for safety
- Multi-client relay

**Use Cases:**
- Universal destination setter
- Dynamic command input
- Flexible search boxes
- Multi-purpose entry fields

---

## Summary

These advanced patterns from EVE Online production scripts demonstrate:

1. **Deep UI Navigation** - `FindUsableChild` for robust element access
2. **Two-Way Data Binding** - Bidirectional sync between UI and script
3. **Stateful UI Components** - Toggle buttons that remember state
4. **Settings Persistence** - Auto-save on change patterns
5. **Console Logging** - Real-time output to UI console
6. **Visual Feedback** - Alpha transparency effects and animations
7. **Multi-Client Coordination** - Relay commands for fleet operations
8. **Nested TabControls** - Tab-within-tab organization
9. **Slider Synchronization** - Real-time label updates with unit conversion
10. **List Management** - Iterator-based population patterns
11. **Advanced TextEntry** - OnKeyDown for Enter key detection
12. **ComboBox Values** - Separate display text from stored values
13. **Production Robustness** - Dynamic displays and polymorphic input

**Key Takeaways:**
- Real production scripts require sophisticated UI patterns
- UI and script must stay synchronized
- Visual feedback improves user experience
- Multi-client support requires relay commands
- Sliders need synchronized labels for good UX
- Nested tabs organize complex option screens
- OnKeyDown enables command-line style input
- Use the right pattern for the job

---

<!-- CLAUDE_SKIP_START -->
## Additional Resources

**Script Sources:**
- **EVEBot:** https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable
- **WreckingBall2:** https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/WreckingBall2
- **Yamfa:** https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/Yamfa

**Related Guides:**
- [LavishGUI 1 UI Guide](08_LavishGUI1_UI_Guide.md)
- [Core Concepts](04_Core_Concepts.md)
- [Advanced Patterns](07_Advanced_Patterns_And_Examples.md)

---

*Part of ISXEVE Scripting Guide*
<!-- CLAUDE_SKIP_END -->
