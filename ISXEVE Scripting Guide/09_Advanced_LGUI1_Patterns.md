# Advanced LavishGUI 1 Patterns and Techniques

**Purpose:** Real-world patterns from production EVE Online scripts
**Sources:**
- [EVEBot](https://github.com/CyberTech/EVEBot/tree/master/Branches/Stable) - Combat/mining/mission bot 
- [WreckingBall2](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/WreckingBall2) - Salvaging bot
- [Yamfa](https://github.com/isxGames/isxScripts/tree/master/EVE-Online/Scripts/Yamfa) - Multi-client automation

**Target Audience:** Advanced script developers

---

## Table of Contents

1. [UI Element Navigation](#ui-element-navigation)
2. [Two-Way Data Binding](#two-way-data-binding)
3. [Cross-Element UI Synchronization](#cross-element-ui-synchronization)
4. [Dynamic UI Updates](#dynamic-ui-updates)
5. [Settings Integration](#settings-integration)
6. [Console and Logging](#console-and-logging)
7. [Advanced Button Patterns](#advanced-button-patterns)
8. [Alpha Transparency Effects](#alpha-transparency-effects)
9. [Multi-Client Relay Commands](#multi-client-relay-commands)
10. [Nested TabControls](#nested-tabcontrols)
11. [Slider with Synchronized Labels](#slider-with-synchronized-labels)
12. [Advanced List Management](#advanced-list-management)
13. [Advanced TextEntry Patterns](#advanced-textentry-patterns)
14. [ComboBox Advanced Patterns](#combobox-advanced-patterns)
15. [Production Patterns](#production-patterns)
16. [Production Skin Pattern](#production-skin-pattern)

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

## Cross-Element UI Synchronization

### Lookup-Driven Element Sync

**Pattern:** Use one element's value as a lookup key into a data collection, then drive the state of a different element based on the result. The same sync logic runs from multiple trigger points (TextEntry `OnChange` and Listbox `OnSelect`) so that "the current selection changed" always produces consistent UI state.

**Source:** EVEBot — Fleet member editor (tbAddFleetMember TextEntry syncs cbWing Checkbox)

**The scenario:** A text input lets the user type a fleet member name. A separate checkbox reflects whether that member is configured for "Wing 2". The checkbox must stay in sync with the text input as the user types, AND when the user clicks an existing entry in a listbox of fleet members.

**The TextEntry (triggers sync on every keystroke):**

```xml
<TextEntry name='tbAddFleetMember'>
  <X>300</X>
  <Y>40</Y>
  <Width>200</Width>
  <Height>18</Height>
  <MaxLength>50</MaxLength>
  <OnChange>
    if ${Script[EVEBot].VariableScope.Config.Fleet.IsWing["${This.Text.Escape}"]}
      UIElement[cbWing@Fleet@EVEBotOptionsTab@EVEBot]:SetChecked
    else
      UIElement[cbWing@Fleet@EVEBotOptionsTab@EVEBot]:UnsetChecked
  </OnChange>
</TextEntry>
```

**The Checkbox (target of the sync):**

```xml
<checkbox name='cbWing'>
  <X>450</X>
  <Y>65</Y>
  <Width>50</Width>
  <Height>20</Height>
  <Text>Wing 2</Text>
  <AutoTooltip>If checked, this character will be moved to Wing 2.</AutoTooltip>
  <OnLeftClick>
    Script[EVEBot].VariableScope.Config.Fleet:SetWingOne[${This.Checked}]
  </OnLeftClick>
</checkbox>
```

**The Listbox (triggers the same sync on selection):**

```xml
<listbox Name='FleetMembers'>
  <X>10</X>
  <Y>90</Y>
  <Width>530</Width>
  <Height>175</Height>
  <Sort>Text</Sort>

  <OnSelect>
    ; Copy selected item text into the TextEntry
    UIElement[tbAddFleetMember@Fleet@EVEBotOptionsTab@EVEBot]:SetText[${This.SelectedItem.Text}]

    ; Run the identical IsWing check to sync cbWing
    if ${Script[EVEBot].VariableScope.Config.Fleet.IsWing[${UIElement[tbAddFleetMember@Fleet@EVEBotOptionsTab@EVEBot].Text}]}
      UIElement[cbWing@Fleet@EVEBotOptionsTab@EVEBot]:SetChecked
    else
      UIElement[cbWing@Fleet@EVEBotOptionsTab@EVEBot]:UnsetChecked
  </OnSelect>
</listbox>
```

**The Script-Side Lookup Method:**

```lavishscript
member:bool IsWing(string value)
{
    This:RefreshFleetMembers
    variable iterator InfoFromSettings
    This.FleetMembers:GetIterator[InfoFromSettings]
    if ${InfoFromSettings:First(exists)}
        do
        {
            if ${InfoFromSettings.Value.FleetMemberName.Equal[${value}]} && ${InfoFromSettings.Value.Wing}
                return TRUE
        }
        while ${InfoFromSettings:Next(exists)}
    return FALSE
}
```

**Key techniques:**

1. **Fully-qualified element paths** — `UIElement[cbWing@Fleet@EVEBotOptionsTab@EVEBot]` reaches across the entire UI tree from the `@element@tab@tabcontrol@window` hierarchy. This is necessary when the trigger element and target element live in different containers.

2. **Lookup key is a UI value** — The TextEntry's current text is passed as a parameter to a script member function that queries persisted settings. The function returns a bool that directly drives the checkbox state.

3. **`${This.Text.Escape}`** — Escapes special characters in the current text before passing it as a parameter, preventing parse errors from quotes/brackets in typed names.

4. **Duplicated sync logic on multiple triggers** — Both the TextEntry's `OnChange` and the Listbox's `OnSelect` run the SAME lookup-and-sync block. Both events represent "the current selection changed" so both need to update the checkbox. (This duplication is acceptable when the logic is short — for longer logic, extract it to a script method.)

5. **Two-way data flow, not binding** — This is not traditional data binding. The UI elements drive each other through script-side lookups, not through a shared bound variable.

**When to use this pattern:**

- When one element's value is a key into a data collection and other elements must reflect properties of the matching record
- When multiple user actions (typing, selecting from a list, pasting) should all trigger the same UI state update
- When the relationship is too complex for simple one-to-one data binding
- When UI elements live in different tabs or containers

**Caveats:**

- Fully-qualified paths are brittle — if you rename a tab or window, every cross-element reference breaks. Consider using `FindUsableChild` variables for more resilient references (see [Deep Element Access with FindUsableChild](#deep-element-access-with-findusablechild)).
- Duplicated sync logic is a maintenance hazard. If the logic grows beyond a few lines, refactor it into a script atom/method that all trigger points call.
- `OnChange` fires on every keystroke, so the lookup method must be fast (avoid expensive iteration over large collections).

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

**XML Side:** (wrapper/geometry same as the Defense button shown in [Dual-Action Buttons (Left/Right Click)](#dual-action-buttons-leftright-click) above — only the event handlers differ)

```xml
<!-- Inside <button name='Defense'> ... </button> -->
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

### OnRightClick as Programmatic Refresh Trigger

**Pattern:** Decouple list population logic from multiple trigger points by placing it in `OnRightClick`, then calling `:RightClick` programmatically from `OnLoad` and from Add/Delete buttons.

**Source:** EVEBot — Whitelist management (Pilots, Corporations, Alliances lists)

**Why this pattern?** When a listbox needs to be refreshed from multiple places (initial load, after adding an item, after deleting an item, after external sync), duplicating the population logic in each trigger is error-prone. By centralizing the logic in `OnRightClick` and triggering it programmatically, you get a single source of truth for list population.

**The ListBox:**

```xml
<listbox name='lbWLPilots'>
  <X>r140</X>
  <Y>90</Y>
  <Width>130</Width>
  <Height>300</Height>
  <Sort>Text</Sort>

  <!-- Step 1: OnLoad triggers OnRightClick for initial population -->
  <OnLoad>
    This:RightClick
  </OnLoad>

  <!-- Step 2: OnLeftClick handles user interaction (show details) -->
  <OnLeftClick>
    variable string pilotinfo=${UIElement[MyWindow].FindUsableChild[txtPilotInfo,text].FullName}
    UIElement[${pilotinfo}]:SetText[Pilot: ${This.SelectedItem[1].Text} (${This.SelectedItem[1].Value})]
  </OnLeftClick>

  <!-- Step 3: OnRightClick IS the refresh logic (centralized) -->
  <OnRightClick>
    This:ClearItems
    variable iterator i
    Script[MyScript].VariableScope.Whitelist.PilotsRef:GetSettingIterator[i]
    if ${i:First(exists)}
    {
      do
      {
        This:AddItem[${i.Key},${i.Value}]
      }
      while ${i:Next(exists)}
    }
    This:Sort
  </OnRightClick>
</listbox>
```

**The Add Button (triggers refresh after adding):**

```xml
<button name='btnAddPilot'>
  <Text>Add</Text>
  <OnLeftClick>
    ; Get selected pilot from another list
    variable string pilotname=${UIElement[MyWindow].FindUsableChild[lbLocal,listbox].SelectedItem[1].Text}

    ; Add to whitelist via script method
    Script[MyScript].VariableScope.Social:AddWhiteList[Pilot,${pilotname}]

    ; Refresh the whitelist listbox by triggering OnRightClick
    UIElement[MyWindow].FindUsableChild[lbWLPilots,listbox]:RightClick
  </OnLeftClick>
</button>
```

**The Delete Button (triggers refresh after removing):**

```xml
<button name='btnDelPilot'>
  <Text>Del</Text>
  <OnLeftClick>
    ; Get selected item from whitelist listbox
    variable string pilotname=${UIElement[MyWindow].FindUsableChild[lbWLPilots,listbox].SelectedItem[1].Text}

    ; Remove from whitelist via script method
    Script[MyScript].VariableScope.Social:DelWhiteList[Pilot,${pilotname}]

    ; Refresh the whitelist listbox
    UIElement[MyWindow].FindUsableChild[lbWLPilots,listbox]:RightClick
  </OnLeftClick>
</button>
```

**The complete flow:**

1. **On UI load** → `OnLoad` calls `This:RightClick` → `OnRightClick` runs → list populated from settings
2. **User clicks Add** → button adds item to data → button calls `:RightClick` on listbox → `OnRightClick` runs → list refreshed
3. **User clicks Delete** → button removes item from data → button calls `:RightClick` on listbox → `OnRightClick` runs → list refreshed
4. **User right-clicks listbox manually** → `OnRightClick` runs → list refreshed (manual refresh)

**Key advantages:**
- Population logic exists in exactly one place (OnRightClick)
- Any trigger can refresh the list with a single `:RightClick` call
- Manual user refresh is available for free (right-click the list)
- Adding new trigger points (e.g., cross-session sync) only requires one `:RightClick` call

### Live Game Data + Info-Display Pattern

**Pattern:** Populate a listbox from a live ISXEVE query (not settings), then use `OnLeftClick` to drive a multi-field detail panel by re-querying the game TLO for the selected item.

**Source:** EVEBot — Local pilots listbox with pilot/corp/alliance info display

**Why this pattern?** For lists that reflect live game state (local pilots, nearby entities, fleet members), the listbox should only store a minimal identifier (usually the name) — and detail information should be re-queried from the game on selection to avoid showing stale data.

**The Refresh Button (populates the list from live game data):**

```xml
<button name='btnRefreshLocal' template='EveSkin.Window.ClickButton'>
  <X>20</X>
  <Y>235</Y>
  <Width>110</Width>
  <Height>22</Height>
  <Text>Refresh</Text>
  <OnLeftClick>
    variable string lbname = ${UIElement[EVEBot].FindUsableChild[lbLocal,listbox].FullName}
    variable index:pilot pilots
    variable iterator piter
    EVE:GetLocalPilots[pilots]
    pilots:GetIterator[piter]

    ; Clear stale detail text before repopulating the listbox
    variable string pilotinfo=${UIElement[EVEBot].FindUsableChild[txtPilotInfo,text].FullName}
    variable string corpinfo=${UIElement[EVEBot].FindUsableChild[txtCorpInfo,text].FullName}
    variable string allianceinfo=${UIElement[EVEBot].FindUsableChild[txtAllianceInfo,text].FullName}
    UIElement[${pilotinfo}]:SetText[""]
    UIElement[${corpinfo}]:SetText[""]
    UIElement[${allianceinfo}]:SetText[""]

    ; Populate the listbox, filtering out self via CharID comparison
    UIElement[${lbname}]:ClearItems
    if ${piter:First(exists)}
    {
      do
      {
        if !${piter.Value.CharID.Equal[${Me.CharID}]}
          UIElement[${lbname}]:AddItem[${piter.Value.Name}]
      }
      while ${piter:Next(exists)}
    }
    UIElement[${lbname}]:Sort
  </OnLeftClick>
</button>
```

**The Listbox (displays pilot info on selection via Local[name] re-query):**

```xml
<listbox name='lbLocal'>
  <X>10</X>
  <Y>25</Y>
  <Width>130</Width>
  <Height>200</Height>
  <Sort>Text</Sort>
  <OnLeftClick>
    variable string pilotinfo=${UIElement[EVEBot].FindUsableChild[txtPilotInfo,text].FullName}
    variable string corpinfo=${UIElement[EVEBot].FindUsableChild[txtCorpInfo,text].FullName}
    variable string allianceinfo=${UIElement[EVEBot].FindUsableChild[txtAllianceInfo,text].FullName}

    ; Re-query the Local TLO by pilot name to get fresh data
    variable pilot p = ${Local[${This.SelectedItem[1].Text}]}

    UIElement[${pilotinfo}]:SetText[Pilot: ${p.Name} (${p.CharID})]
    UIElement[${corpinfo}]:SetText[Corp ID: ${p.Corp.ID}]
    UIElement[${allianceinfo}]:SetText[Alliance: ${p.Alliance} (${p.AllianceID})]
  </OnLeftClick>
</listbox>
```

**The Info Display Fields:**

```xml
<Text Name='txtPilotInfo'>
  <X>150</X>
  <Y>25</Y>
  <Width>r355</Width>
  <Height>60</Height>
  <wrap />
  <text>Select a pilot from the list at left.</text>
</Text>

<Text Name='txtCorpInfo'>
  <X>150</X>
  <Y>105</Y>
  <Width>r355</Width>
  <Height>60</Height>
  <wrap />
</Text>

<Text Name='txtAllianceInfo'>
  <X>150</X>
  <Y>185</Y>
  <Width>r355</Width>
  <Height>60</Height>
  <wrap />
</Text>
```

**Key techniques:**

1. **Live query with iterator** — `EVE:GetLocalPilots[index:pilot]` populates an index with all pilots currently in the Local channel. Each iteration yields a `pilot` datatype with `.Name`, `.CharID`, `.Corp`, `.Alliance`, `.AllianceID`, `.Standing`, etc.

2. **Self-filtering** — `if !${piter.Value.CharID.Equal[${Me.CharID}]}` skips the player's own character by comparing CharIDs. This works because `character` inherits from `pilot`, so both have `CharID`.

3. **Name-only storage** — `AddItem[${piter.Value.Name}]` stores only the pilot name in the listbox item. No hidden value is needed because the `Local[name]` TLO will re-query on selection.

4. **Re-query on selection** — `variable pilot p = ${Local[${This.SelectedItem[1].Text}]}` uses the `Local[Name]` TLO (which accepts either an integer index or a pilot name) to get a fresh `pilot` object, ensuring the displayed data is current.

5. **Pre-clear detail fields on refresh** — The refresh button clears `txtPilotInfo`, `txtCorpInfo`, and `txtAllianceInfo` before repopulating, preventing stale text from referring to a pilot who may have left Local.

6. **FullName resolution pattern** — `FindUsableChild[elementName,type].FullName` stores the full element path in a variable for reuse, making long nested references cleaner and more efficient.

**Applicable live-query APIs:**

| API | Returns | Datatype | Re-query TLO |
|-----|---------|----------|--------------|
| `EVE:GetLocalPilots[index:pilot]` | Pilots in Local | `pilot` | `Local[name]` |
| `EVE:QueryEntities[index:entity,query]` | Filtered entities | `entity` | `Entity[name]` |
| `Me:GetFleetMembers[index:fleetmember]` | Fleet members | `fleetmember` | — |

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

### Dynamic Population from Game Data

**Pattern:** Populate a ComboBox at load time from an ISXEVE API (bookmarks, entities, etc.), with sorting and saved selection restoration.

**Source:** EVEBot — Freighter destination bookmark selector

**The ComboBox:**

```xml
<combobox name='DestinationBookmark'>
  <X>150</X>
  <Y>35</Y>
  <Width>375</Width>
  <Height>18</Height>
  <FullHeight>350</FullHeight>
  <ButtonWidth>20</ButtonWidth>
  <Sort>Text</Sort>
  <Items>
    <Item Value='1'>None</Item>
  </Items>

  <OnLoad>
    ; Configure sorting
    This:SetSortType[Text]
    This:SetAutoSort[TRUE]

    ; Populate from ISXEVE bookmark API
    variable index:bookmark bm_index
    EVE:GetBookmarks[bm_index]

    variable iterator bm_iterator
    bm_index:GetIterator[bm_iterator]
    This:ClearItems
    if ${bm_iterator:First(exists)}
    {
      do
      {
        This:AddItem[${bm_iterator.Value.Label},${bm_iterator.Value.ID}]
      }
      while ${bm_iterator:Next(exists)}
    }

    ; Restore saved selection (text → item ID lookup)
    This:SelectItem[${This.ItemByText[${Script[EVEBot].VariableScope.Config.Freighter.Destination}].ID}]
    This:Sort
  </OnLoad>

  <OnSelect>
    if ${This.SelectedItem.Text.NotNULLOrEmpty}
    {
      Script[EVEBot].VariableScope.Config.Freighter:SetDestination[${This.SelectedItem.Text}]
    }
  </OnSelect>
</combobox>
```

**Key techniques:**

1. **`EVE:GetBookmarks[index:bookmark]`** — Populates an index with all bookmarks. Returns count. Each bookmark has `.Label` (display name) and `.ID` (unique int64 identifier).

2. **`AddItem[text, value]`** — Dual-parameter form stores both the bookmark label (display) and ID (hidden value). This enables lookup by either text or value later.

3. **`SetSortType[Text]` + `SetAutoSort[TRUE]`** — Configures alphabetical sorting that applies automatically as items are added.

4. **Saved selection restoration** — Config stores the bookmark **label** (text). To restore the selection, `ItemByText[savedText]` finds the matching combobox item, `.ID` gets its internal item index, and `SelectItem[index]` selects it.

5. **`NotNULLOrEmpty` guard** — Prevents writing empty/null values to config when selection is cleared.

**Applicable ISXEVE APIs for dynamic population:**

| API | Returns | Use Case |
|-----|---------|----------|
| `EVE:GetBookmarks[index:bookmark]` | All bookmarks | Destination/location selectors |
| `EVE:GetLocalPilots[index:pilot]` | Local pilots | Fleet member selection |
| `Me:GetStations[index:station]` | Known stations | Station selectors |
| `Me:GetJournalEntries[index:journalentry]` | Journal entries | Entry browsers |

Each API follows the same pattern: populate an index, iterate with an iterator, add items via `AddItem[displayText, ID]`.

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

This section previously duplicated the Yamfa `SetDest` textentry example. It has been consolidated to avoid repetition.

The `SetDest` textentry block with `OnKeyDown` Enter-key detection and polymorphic routing between bookmarks and universe destinations is shown earlier in this guide under [OnKeyDown for Enter Key Detection](#onkeydown-for-enter-key-detection). See that section for the XML and Key Techniques.

---

## Production Skin Pattern

### Template Library Skin File

**Pattern:** Create a reusable visual skin by defining a collection of named templates in a separate XML file, then load that file before your main UI file. The templates become globally available and UI elements reference them via `template='Name.Path'` attributes.

**Source:** EVEBot — `interface/eveskin/eveskin.xml` (277 lines, 46 templates)

**Why this pattern?** Separating visual appearance from UI structure lets you theme an entire application by swapping one file. EVEBot keeps all fonts, colors, textures, and widget appearances in `eveskin.xml` while `EVEBot.xml` contains only layout and behavior.

**Important:** EVEBot does NOT use the `<Skin>` element or the `-skin` flag. It uses the simpler **"template library" pattern**: load the skin file first (registers templates globally), then load the main UI file (which references them by name). Both are loaded as plain UI files via `ui -load`.

**Skin File Structure (eveskin.xml):**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<ISUI>
  <!-- ======================================== -->
  <!-- 1. TEXTURE TEMPLATES                     -->
  <!-- ======================================== -->
  <template name='EVESkin.Window.TitleBar.Texture'
            Filename='eveskin/MainGUI/optionswindow_titlebar.png' />

  <!-- Texture atlas slicing (4 states from one PNG) -->
  <template name='checkbox.TextureUnchecked'
            Filename='CheckBox.png' Border='0'>
    <Left>0</Left><Right>16</Right><Top>0</Top><Bottom>16</Bottom>
  </template>
  <template name='checkbox.TextureUncheckedHover'
            Filename='CheckBox.png' Border='0'>
    <Left>16</Left><Right>32</Right><Top>0</Top><Bottom>16</Bottom>
  </template>
  <template name='checkbox.TextureUncheckedPressed'
            Filename='CheckBox.png' Border='0'>
    <Left>32</Left><Right>48</Right><Top>0</Top><Bottom>16</Bottom>
  </template>
  <template name='checkbox.TextureChecked'
            Filename='CheckBox.png' Border='0'>
    <Left>48</Left><Right>64</Right><Top>0</Top><Bottom>16</Bottom>
  </template>

  <!-- ======================================== -->
  <!-- 2. FONT TEMPLATES                        -->
  <!-- ======================================== -->
  <template name='EVESkin.Font.TitleBar'>
    <Name>Verdana</Name>
    <Size>12</Size>
    <Color>FFFF9900</Color>
  </template>
  <template name='Console.Font'>
    <Name>Terminal</Name>
    <Size>8</Size>
    <Color>FFFF9900</Color>
  </template>

  <!-- Font inherits from Default Font, overrides specific properties -->
  <template name='checkbox.Font' Template='Default Font'>
    <Name>Tahoma</Name>
    <Size>10</Size>
  </template>

  <!-- ======================================== -->
  <!-- 3. WIDGET TEMPLATES                      -->
  <!-- ======================================== -->
  <template name='checkbox'>
    <Font Template='checkbox.Font' />
    <Width>16</Width>
    <Height>16</Height>
    <Border>0</Border>
    <Texture template='checkbox.TextureUnchecked' />
    <TextureHover template='checkbox.TextureUncheckedHover' />
    <TexturePressed template='checkbox.TextureUncheckedPressed' />
    <TextureChecked template='checkbox.TextureChecked' />
    <TextureCheckedHover template='checkbox.TextureUncheckedPressed' />
    <TextureCheckedPressed template='checkbox.TextureUncheckedHover' />
  </template>

  <!-- commandcheckbox inherits from checkbox -->
  <template name='commandcheckbox' Template='checkbox'>
    <!-- Inherits all checkbox properties, can override if needed -->
  </template>

  <!-- ======================================== -->
  <!-- 4. COMPOSITE TEMPLATES                   -->
  <!-- ======================================== -->
  <template name='EveSkin.Window.ClickButton'>
    <Font template='Button.Font' />
    <Texture template='EVESkin.Window.ClickButton.Texture' />
    <TextureHover template='EVESkin.Window.ClickButton.HoverTexture' />
  </template>

  <template name='EVESkin.Console'>
    <Border>0</Border>
    <Font template='Console.Font' />
    <SelectionColor>FFD4D0C8</SelectionColor>
    <ScrollBar>console.ScrollBar</ScrollBar>
    <BackBufferSize>2000</BackBufferSize>
  </template>
</ISUI>
```

**Loading the Skin (obj_EVEBotUI.iss pattern):**

```lavishscript
objectdef obj_EVEBotUI
{
    variable string Skin = "EVESkin"
    variable string SkinFile = "eveskin/EVESkin.xml"

    method Initialize()
    {
        ; Load order is CRITICAL: skin file first, then main UI
        ui -load interface/${This.SkinFile}
        ui -load interface/EVEBot.xml
    }

    method Shutdown()
    {
        ; Unload in reverse order
        ui -unload interface/EVEBot.xml
        ui -unload interface/${This.SkinFile}
    }
}
```

**Consuming Templates from the Main UI File:**

```xml
<window name='EVEBot' template='EVESkin'>
  <TitleBar template='EVESkin.Window.Titlebar'>
    <!-- ... -->
  </TitleBar>
  <Children>
    <TabControl Name='MainTab' template='EVESkin.TabControl'>
      <!-- ... -->
    </TabControl>
    <console Name='StatusConsole' template='EVESkin.Console'>
      <!-- ... -->
    </console>
  </Children>
</window>
```

**Key techniques:**

1. **Naming convention** — Use dot-separated hierarchical names like `EVESkin.Window.TitleBar.Minimize.Texture`. This makes template purpose clear and prevents collisions with other skins loaded in the same session.

2. **Texture atlasing** — Multi-state widgets (checkboxes with 4 states, comboboxes with 3 regions) use a single PNG file with `<Left>/<Right>/<Top>/<Bottom>` slicing coordinates. This reduces file count and improves texture caching.

3. **Template inheritance for variants** — Use `Template='parentName'` on a template definition to create variants. EVEBot uses this for `commandcheckbox` (inherits `checkbox`), `combobox.ListBox` (inherits `listbox`), and font overrides inheriting from `Default Font`.

4. **Load order matters** — The skin file must be loaded BEFORE any UI file that references its templates. Templates are resolved at load time, so forward references fail. Unload in reverse order during shutdown.

5. **Self-contained file** — The skin file contains only templates — no windows, no events, no script bindings. This makes it a pure visual theme that can be swapped without touching UI logic.

6. **Runtime dynamic UI support** — When creating UI elements dynamically from script (via `UIElement:AddChild`), pass the skin name as the 4th argument: `UIElement[parent]:AddChild["type", name, xml, "eveskin"]`. This tells LavishGUI which template namespace to resolve references against.

**File organization:**

```
interface/
├── eveskin/
│   ├── eveskin.xml             ← Template definitions
│   └── MainGUI/
│       ├── CheckBox.png         ← Multi-state texture atlas
│       ├── Combobox.png
│       ├── button_close.png
│       ├── button_minimize.png
│       └── ...                  ← Individual widget textures
├── EVEBot.xml                    ← Main UI (references templates)
└── ...
```

**Benefits of this pattern:**

- **Theme swapping** — Create alternate skin files (e.g., `eveskin_dark.xml`, `eveskin_light.xml`) and select at load time
- **Consistent appearance** — All checkboxes, buttons, etc. look identical without repeating properties
- **Maintainability** — Change one template to update every element using it
- **Separation of concerns** — Visual designers work on the skin file, developers work on the main UI file
- **Reusability** — The skin file can be shared across multiple scripts

---

## Summary

These advanced patterns from EVE Online production scripts demonstrate:

1. **Deep UI Navigation** - `FindUsableChild` for robust element access
2. **Two-Way Data Binding** - Bidirectional sync between UI and script
3. **Cross-Element UI Sync** - Lookup-driven sync across elements via shared lookup key
4. **Stateful UI Components** - Toggle buttons that remember state
5. **Settings Persistence** - Auto-save on change patterns
6. **Console Logging** - Real-time output to UI console
7. **Visual Feedback** - Alpha transparency effects and animations
8. **Multi-Client Coordination** - Relay commands for fleet operations
9. **Nested TabControls** - Tab-within-tab organization
10. **Slider Synchronization** - Real-time label updates with unit conversion
11. **List Management** - Iterator-based population patterns
12. **Decoupled List Refresh** - OnRightClick as centralized refresh trigger
13. **Live Game Data Display** - Listbox population from live queries with re-query info display
14. **Advanced TextEntry** - OnKeyDown for Enter key detection
15. **ComboBox Values** - Separate display text from stored values
16. **Dynamic ComboBox Population** - Populate from ISXEVE APIs with saved selection restore
17. **Production Robustness** - Dynamic displays and polymorphic input
18. **Skin File Pattern** - Template library with texture atlasing for reusable visual themes

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
  <!-- CLAUDE_SKIP_END -->
