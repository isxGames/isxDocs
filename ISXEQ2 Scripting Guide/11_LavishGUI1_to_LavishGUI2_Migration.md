# LavishGUI 1 to LavishGUI 2 Migration Guide

**Purpose:** Convert existing LavishGUI 1 XML scripts to LavishGUI 2 JSON
**Target Audience:** Script developers maintaining older LGUI1 scripts

---

## Table of Contents

1. [Migration Overview](#migration-overview)
2. [Key Differences Between LGUI1 and LGUI2](#key-differences-between-lgui1-and-lgui2)
3. [File Format Conversion](#file-format-conversion)
4. [Element Mapping](#element-mapping)
5. [Event Handler Migration](#event-handler-migration)
6. [Script Code Migration](#script-code-migration)
7. [Data Binding Migration](#data-binding-migration)
8. [Complete Migration Examples](#complete-migration-examples)
9. [Migration Checklist](#migration-checklist)
10. [Common Migration Issues](#common-migration-issues)

---

## Migration Overview

<!-- CLAUDE_SKIP_START -->
### Why Migrate?

**LavishGUI 2** is the modern, supported UI framework with:

- ✅ **Better tooling** - JSON schema provides IDE autocomplete
- ✅ **Automatic data binding** - Reduces manual UI updates
- ✅ **Cleaner syntax** - JSON is more readable than XML
- ✅ **Active development** - LGUI1 is legacy, LGUI2 is the future
- ✅ **More powerful** - Better event handling, templates, and features
<!-- CLAUDE_SKIP_END -->

### Migration Process

The migration requires:

1. **Convert XML files to JSON** - New file format
2. **Update element types and properties** - JSON structure differs from XML
3. **Migrate event handlers** - Inline JSON instead of XML attributes
4. **Update script code** - New loading methods and element access
5. **Implement data binding** - Replace manual UI updates

**Important:** This is **not** a simple find-replace. The paradigm shift from XML to JSON requires rethinking your UI structure.

**CRITICAL: Always use 1x (baseline) sizing during migration. Never copy 2x scaled values from existing examples. Scaling is applied SEPARATELY after migration is complete.**

---

## Key Differences Between LGUI1 and LGUI2

### Side-by-Side Comparison

| Aspect | LavishGUI 1 (Old) | LavishGUI 2 (New) |
|--------|-------------------|-------------------|
| **File Format** | XML (`.xml`) | JSON (`.json`) |
| **Schema** | None | `$schema` for validation |
| **Root Element** | `<LGUI>` or specific element | Package with `elements` array |
| **Elements** | `<Window>`, `<Button>`, etc. | `"type": "window"`, etc. |
| **Loading** | `ui -load filename.xml` | `LGUI2:LoadPackageFile[filename.json]` |
| **Unloading** | `ui -unload filename` | `LGUI2:UnloadPackageFile[filename.json]` |
| **Element Access** | `UIElement[name]` | `LGUI2.Element[name]` |
| **Events** | XML attributes (`OnLeftClick=""`) | JSON `eventHandlers` object |
| **Text Content** | `<Text>content</Text>` | `"text": "content"` |
| **Nested Elements** | XML children | `"children"` or `"content"` arrays |
| **Data Updates** | Manual (`SetText`, etc.) | Automatic data binding |
| **Templates** | XML-based | JSON `templates` with `jsonTemplate` |

### Conceptual Differences

**LGUI1:** Element-centric with XML attributes
```xml
<Button Name="mybutton" OnLeftClick="echo Clicked!">
    <Text>Click Me</Text>
</Button>
```

**LGUI2:** Property-based with JSON structure
```json
{
    "type": "button",
    "name": "mybutton",
    "content": "Click Me",
    "eventHandlers": {
        "onPress": {
            "type": "code",
            "code": "echo Clicked!"
        }
    }
}
```

---

## File Format Conversion

### LGUI1 XML Structure

```xml
<?xml version='1.0'?>
<LGUI>
    <Window Name="mywindow" Width="300" Height="200">
        <Title>My Window</Title>
        <Text>Hello World</Text>
    </Window>
</LGUI>
```

### LGUI2 JSON Structure

```json
{
    "$schema": "http://www.lavishsoft.com/schema/lgui2Package.json",
    "elements": [
        {
            "type": "window",
            "name": "mywindow",
            "title": "My Window",
            "x": 1480,
            "y": 960,
            "width": 800,
            "height": 800,
            "content": "Hello World"
        }
    ]
}
```

**Default Window Size and Location:**

When converting windows from LGUI1 to LGUI2, use these **standard defaults** for consistency:

- `"x": 1480` - Default horizontal position (secondary monitor placement)
- `"y": 960` - Default vertical position
- `"width": 800` - Default window width (adequate space for most UIs)
- `"height": 800` - Default window height

These defaults match the pattern used in EQ2BotCommander and other EQ2Bot UI files, placing windows on a secondary monitor in typical multi-monitor setups. Users can move/resize windows as needed, and positions can be persisted via settings.

### Conversion Steps

1. **Add JSON schema** - Always start with `$schema`

2. **Create `elements` array** - Root-level UI components

3. **Convert element tags to objects** - `<Window>` → `{"type": "window"}`

4. **Convert attributes to properties** - `Name="value"` → `"name": "value"`

5. **Convert text content** - `<Text>content</Text>` → `"text": "content"`

6. **Nest child elements** - Use `"content"` or `"children"` arrays

### Property Name Changes

| LGUI1 XML Attribute | LGUI2 JSON Property |
|---------------------|---------------------|
| `Name` | `name` |
| `Width` | `width` |
| `Height` | `height` |
| `Location` | Position via `x`, `y` or `SetLocation` |
| `OnLeftClick` | `eventHandlers.onPress` |
| `OnRightClick` | `eventHandlers.onRightPress` |
| `OnLoad` | `eventHandlers.onLoad` |
| `OnUnload` | `eventHandlers.onCloseButtonClick` (see note below) |

**Note on OnUnload:** LGUI2 does not have an `onUnload` event. Use `onCloseButtonClick` to handle window close button clicks. See the "OnLoad / OnUnload Events" section for details.

### Case Sensitivity

**LGUI1:** Case-insensitive XML tags and attributes
```xml
<Window NAME="test" WIDTH="100">
```

**LGUI2:** Case-sensitive JSON properties
```json
{
    "name": "test",    // Must be lowercase
    "width": 100       // Must be lowercase
}
```

---

## Element Mapping

### Window

**LGUI1:**

```xml
<Window Name="mywindow" Width="400" Height="300">
    <Title>My Window</Title>
    <Text>Window content</Text>
</Window>
```

**LGUI2 (Simple - No Scaling):**

```json
{
    "type": "window",
    "name": "mywindow",
    "title": "My Window",
    "x": 1480,
    "y": 960,
    "width": 800,
    "height": 800,
    "content": "Window content"
}
```

**LGUI2 (Scalable - Recommended for New Scripts):**

For scripts that use UI scaling (LGUI2Scaling.iss), the default title bar won't scale. Use a custom `titleBar` instead:

```json
{
    "type": "window",
    "name": "mywindow",
    "x": 1480,
    "y": 960,
    "width": 800,
    "height": 800,
    "eventHandlers": {
        "onCloseButtonClick": {
            "type": "code",
            "code": "Script[myscript]:QueueCommand[Shutdown]"
        }
    },
    "titleBar": {
        "type": "dockpanel",
        "children": [
            {
                "type": "button",
                "borderThickness": 2,
                "content": "🗙",
                "font": {"face": "Segoe UI", "height": 40},
                "padding": 12,
                "_dock": "right",
                "eventHandlers": {
                    "onMouseButtonMove": {
                        "type": "forward",
                        "elementType": "window",
                        "event": "onCloseButtonClick"
                    }
                }
            },
            {
                "type": "button",
                "borderThickness": 2,
                "content": "🗕",
                "font": {"face": "Segoe UI", "height": 40},
                "padding": 12,
                "margin": [0, 0, 8, 0],
                "_dock": "right",
                "eventHandlers": {
                    "onMouseButtonMove": {
                        "type": "forward",
                        "elementType": "window",
                        "event": "onShadeButtonClick"
                    }
                }
            },
            {
                "type": "textblock",
                "text": "My Window",
                "verticalAlignment": "center",
                "margin": [10, 0, 0, 0],
                "font": {"face": "Segoe UI", "height": 36, "bold": true},
                "color": "#FFFFFF",
                "_dock": "left"
            }
        ],
        "eventHandlers": {
            "onMouseMove": {
                "type": "forward",
                "elementType": "window",
                "event": "onHeaderMouseMove"
            },
            "lostMouseFocus": {
                "type": "forward",
                "elementType": "window",
                "event": "onHeaderLostMouseFocus"
            },
            "onMouseButtonMove": {
                "type": "forward",
                "elementType": "window",
                "event": "onHeaderMouseButtonMove"
            }
        }
    },
    "content": "Window content"
}
```

**Important:** When using custom `titleBar`, **remove** the `title` property to avoid duplication.

**Script-side implementation:**
```lavishscript
atom(script) Shutdown()
{
    call SaveSettings
    Script:End
}
```

See complete example: [EQ2BotCommander.json](https://github.com/isxGames/isxScripts/blob/master/EverQuest2/Scripts/EQ2Bot/UI/EQ2BotCommander.json) (lines 1-88)
For details on scalable title bars: [12_LGUI2_Scaling_System.md](12_LGUI2_Scaling_System.md#creating-scalable-title-bars)

### Button

**LGUI1:**

```xml
<Button Name="mybutton" OnLeftClick="echo Clicked!">
    <Text>Click Me</Text>
</Button>
```

**LGUI2:**

```json
{
    "type": "button",
    "name": "mybutton",
    "content": "Click Me",
    "eventHandlers": {
        "onPress": {
            "type": "code",
            "code": "echo Clicked!"
        }
    }
}
```

### Checkbox

**LGUI1:**

```xml
<Checkbox Name="mycheckbox" OnLeftClick="MyObject:OnCheckboxClicked">
    <Text>Enable Feature</Text>
</Checkbox>
```

**LGUI2:**

```json
{
    "type": "checkbox",
    "name": "mycheckbox",
    "content": "Enable Feature",
    "eventHandlers": {
        "onChecked": {
            "type": "method",
            "object": "MyObject",
            "method": "OnChecked"
        },
        "onUnchecked": {
            "type": "method",
            "object": "MyObject",
            "method": "OnUnchecked"
        }
    }
}
```

**Note:** LGUI2 has separate `onChecked` and `onUnchecked` events instead of a single click event.

### ComboBox

**LGUI1:**

```xml
<Combobox Name="mycombo" OnSelect="MyObject:OnSelection">
    <Items>
        <Text>Option 1</Text>
        <Text>Option 2</Text>
        <Text>Option 3</Text>
    </Items>
</Combobox>
```

**LGUI2:**

```json
{
    "type": "combobox",
    "name": "mycombo",
    "items": [
        {"type": "textblock", "text": "Option 1"},
        {"type": "textblock", "text": "Option 2"},
        {"type": "textblock", "text": "Option 3"}
    ],
    "eventHandlers": {
        "onSelectionChanged": {
            "type": "method",
            "object": "MyObject",
            "method": "OnSelection"
        }
    }
}
```

### TabControl

**CRITICAL:** LGUI2 has native tabcontrol support - use it!

**LGUI1:**

```xml
<Tabcontrol Name="mytabs">
    <Tabs>
        <Tab Name="Main">
            <Text>Main tab content</Text>
        </Tab>
        <Tab Name="Settings">
            <Text>Settings tab content</Text>
        </Tab>
    </Tabs>
</Tabcontrol>
```

**LGUI2:**

```json
{
    "type": "tabcontrol",
    "name": "mytabs",
    "tabs": [
        {
            "type": "tab",
            "header": "Main",
            "content": {
                "type": "textblock",
                "text": "Main tab content"
            }
        },
        {
            "type": "tab",
            "header": "Settings",
            "content": {
                "type": "textblock",
                "text": "Settings tab content"
            }
        }
    ]
}
```

**Key Differences:**

| LGUI1 | LGUI2 |
|-------|-------|
| `<Tabs>` container | `"tabs"` array property |
| `<Tab Name="...">` | `"type": "tab"` + `"header": "..."` |
| Child elements directly in `<Tab>` | `"content"` property (single element) |

**Important Notes:**

1. **Each tab MUST** have `"type": "tab"` (required)
2. **Use `"header"`** for the tab label (NOT "title" or "name")
3. **Wrap multiple elements** in tab content using a panel or stackpanel:

   ```json
   {
       "type": "tab",
       "header": "Complex Tab",
       "content": {
           "type": "stackpanel",
           "orientation": "vertical",
           "children": [
               {"type": "button", "content": "Button 1"},
               {"type": "button", "content": "Button 2"}
           ]
       }
   }
   ```

4. **No manual visibility management** - tabcontrol handles everything automatically

### Text / TextBlock

**LGUI1:**

```xml
<Text Name="mytext">Hello World</Text>
```

**LGUI2:**

```json
{
    "type": "textblock",
    "name": "mytext",
    "text": "Hello World"
}
```

**Or use string shorthand:**

```json
"content": "Hello World"
```

### Frame / Panel

**LGUI1:**

```xml
<Frame Name="myframe">
    <Text>Child 1</Text>
    <Text>Child 2</Text>
</Frame>
```

**LGUI2:**

```json
{
    "type": "panel",
    "name": "myframe",
    "children": [
        {"type": "textblock", "text": "Child 1"},
        {"type": "textblock", "text": "Child 2"}
    ]
}
```

### Listbox

**LGUI1:**

```xml
<Listbox Name="mylist">
    <Items>
        <Text>Item 1</Text>
        <Text>Item 2</Text>
    </Items>
</Listbox>
```

**LGUI2:**

```json
{
    "type": "listbox",
    "name": "mylist",
    "items": [
        {"type": "textblock", "text": "Item 1"},
        {"type": "textblock", "text": "Item 2"}
    ]
}
```

**Or with data binding:**

```json
{
    "type": "listbox",
    "name": "mylist",
    "itemsBinding": {
        "pullFormat": "${MyController.GetItems}"
    }
}
```

### Tab Control

**LGUI1:**

```xml
<TabControl>
    <TabPage Name="tab1">
        <Title>Page 1</Title>
        <Text>Content 1</Text>
    </TabPage>
    <TabPage Name="tab2">
        <Title>Page 2</Title>
        <Text>Content 2</Text>
    </TabPage>
</TabControl>
```

**LGUI2:**

Use a combination of buttons and panel visibility, or use advanced itemview with templates. Full tab control documentation coming soon.

---

## Event Handler Migration

### Click Events

**LGUI1:**

```xml
<Button OnLeftClick="echo Left clicked!" OnRightClick="echo Right clicked!">
    <Text>Click Me</Text>
</Button>
```

**LGUI2:**

```json
{
    "type": "button",
    "content": "Click Me",
    "eventHandlers": {
        "onPress": {
            "type": "code",
            "code": "echo Left clicked!"
        },
        "onRightPress": {
            "type": "code",
            "code": "echo Right clicked!"
        }
    }
}
```

### Calling Object Methods

**LGUI1:**

```xml
<Button OnLeftClick="MyController:HandleClick">
    <Text>Click</Text>
</Button>
```

**LGUI2:**

```json
{
    "type": "button",
    "content": "Click",
    "eventHandlers": {
        "onPress": {
            "type": "method",
            "object": "MyController",
            "method": "HandleClick"
        }
    }
}
```

**Controller method signature stays the same:**

```lavishscript
method HandleClick()
{
    echo Button was clicked!
}
```

### OnLoad / OnUnload Events

**LGUI1:**

```xml
<Window OnLoad="MyController:OnWindowLoad" OnUnload="MyController:OnWindowUnload">
    <Title>Window</Title>
</Window>
```

**LGUI2:**

```json
{
    "type": "window",
    "title": "Window",
    "name": "mywindow",
    "x": 1480,
    "y": 960,
    "width": 800,
    "height": 800,
    "eventHandlers": {
        "onLoad": {
            "type": "method",
            "object": "MyController",
            "method": "OnWindowLoad"
        },
        "onCloseButtonClick": {
            "type": "code",
            "code": "Script[myscript]:QueueCommand[Shutdown]"
        }
    }
}
```

**IMPORTANT:** LGUI2 does not have an `onUnload` event that fires when the window is closed. Instead, use `onCloseButtonClick` to handle the window close button.

**Script Implementation:**

```lavishscript
atom(script) Shutdown()
{
    call SaveSettings
    Script:End
}
```

**Why the change?**
- LGUI1's `OnUnload` fired when the window was destroyed
- LGUI2's `onCloseButtonClick` fires when the user clicks the X button
- This gives you explicit control over the shutdown sequence
- Use `QueueCommand` to safely schedule shutdown in the script thread

### Checkbox State Events

**LGUI1:**

```xml
<Checkbox Name="enableFeature" OnLeftClick="MyController:ToggleFeature">
    <Text>Enable</Text>
</Checkbox>
```

In the controller, you had to check the checkbox state manually.

**LGUI2:**

```json
{
    "type": "checkbox",
    "name": "enableFeature",
    "content": "Enable",
    "eventHandlers": {
        "onChecked": {
            "type": "method",
            "object": "MyController",
            "method": "OnFeatureEnabled"
        },
        "onUnchecked": {
            "type": "method",
            "object": "MyController",
            "method": "OnFeatureDisabled"
        }
    }
}
```

**Cleaner controller code:**

```lavishscript
method OnFeatureEnabled()
{
    echo Feature was enabled!
}

method OnFeatureDisabled()
{
    echo Feature was disabled!
}
```

---

## Script Code Migration

### Loading UI Files

**LGUI1:**

```lavishscript
ui -load myui.xml
```

**LGUI2:**

```lavishscript
LGUI2:LoadPackageFile[myui.json]
```

### Unloading UI Files

**LGUI1:**

```lavishscript
ui -unload myui
```

**LGUI2:**

```lavishscript
LGUI2:UnloadPackageFile[myui.json]
```

### Accessing Elements

**LGUI1:**

```lavishscript
UIElement[mywindow]
UIElement[mywindow,mybutton]    ; Nested access
```

**LGUI2:**

```lavishscript
LGUI2.Element[mywindow]
LGUI2.Element[mybutton]         ; Flat access by name
```

### Checking Element Existence

**LGUI1:**

```lavishscript
if ${UIElement[mywindow](exists)}
{
    echo Window exists
}
```

**LGUI2:**

```lavishscript
if ${LGUI2.Element[mywindow](exists)}
{
    echo Window exists
}
```

### Showing/Hiding Elements

**LGUI1:**

```lavishscript
UIElement[mywindow]:Show
UIElement[mywindow]:Hide
```

**LGUI2:**

```lavishscript
LGUI2.Element[mywindow]:Show
LGUI2.Element[mywindow]:Hide
```

### Setting Text

**LGUI1:**

```lavishscript
UIElement[mytext]:SetText["New text"]
```

**LGUI2:**

```lavishscript
LGUI2.Element[mytext]:SetText["New text"]
```

**Or use data binding (better):**

```lavishscript
; In controller
variable string StatusText = "New text"

; In JSON
"textBinding": {"pullFormat": "${MyController.StatusText}"}
```

### Setting Checkbox State

**LGUI1:**

```lavishscript
UIElement[mycheckbox]:SetChecked[TRUE]
```

**LGUI2:**

```lavishscript
LGUI2.Element[mycheckbox]:SetChecked[TRUE]
```

**Or use data binding (better):**

```lavishscript
; In controller
variable bool FeatureEnabled = TRUE

; In JSON
"checkedBinding": "MyController.FeatureEnabled"
```

### Moving Windows

**LGUI1:**

```lavishscript
UIElement[mywindow]:SetX[100]
UIElement[mywindow]:SetY[100]
```

**LGUI2:**

```lavishscript
LGUI2.Element[mywindow]:SetLocation[100, 100]
```

### Resizing Windows

**LGUI1:**

```lavishscript
UIElement[mywindow]:SetWidth[400]
UIElement[mywindow]:SetHeight[300]
```

**LGUI2:**

```lavishscript
LGUI2.Element[mywindow]:SetSize[400, 300]
```

### Controller Pattern

**LGUI1:**

```lavishscript
function main()
{
    ui -load myui.xml

    while 1
        waitframe
}

function OnExit()
{
    ui -unload myui
}
```

**LGUI2 (Recommended Pattern):**

```lavishscript
objectdef myui_controller
{
    method Initialize()
    {
        LGUI2:LoadPackageFile[myui.json]
    }

    method Shutdown()
    {
        LGUI2:UnloadPackageFile[myui.json]
    }
}

variable(global) myui_controller MyUIController

function main()
{
    while 1
        waitframe
}
```

### Textbox Operations

#### Appending Text to Textboxes

**LGUI1:**

```lavishscript
UIElement[Output]:AppendText["New line\n"]
```

**LGUI2:**

LGUI2 textboxes don't have an `AppendText` method. You must get the current text, append to it, and set it back:

```lavishscript
variable string currentText
variable string newLine = "New line\n"

currentText:Set["${LGUI2.Element[Output].Text}"]
currentText:Concat["${newLine}"]
LGUI2.Element[Output]:SetText["${currentText.Escape}"]
```

**Complete Console Echo Example:**

```lavishscript
function ConsoleEcho(string textString)
{
    variable string currentText
    variable string newLine = "[${Time.Time24}] ${textString.Escape}\n"

    currentText:Set["${LGUI2.Element[Output].Text}"]
    currentText:Concat["${newLine}"]
    LGUI2.Element[Output]:SetText["${currentText.Escape}"]
}
```

#### Getting Actual Element Dimensions

**LGUI1:**

```lavishscript
variable int width = ${UIElement[mywindow].Width}
variable int height = ${UIElement[mywindow].Height}
```

**LGUI2:**

When elements use percentage-based sizes (`"width": "100%"`), the `Width` property returns `-1`. Use `ActualWidth.Precision[0]` to get the rendered pixel dimensions:

```lavishscript
variable int width = ${LGUI2.Element[mywindow].ActualWidth.Precision[0]}
variable int height = ${LGUI2.Element[mywindow].ActualHeight.Precision[0]}
```

**Example - Calculate Max Characters Per Line:**

```lavishscript
function ConsoleEcho(string textString)
{
    variable string currentText
    variable string newLine = "[${Time.Time24}] ${textString.Escape}"

    ; Get actual rendered width
    variable int textboxWidth
    variable int fontSize
    variable int maxChars
    variable float charWidth

    textboxWidth:Set[${LGUI2.Element[EQ2AFKAlarm Console].ActualWidth.Precision[0]}]

    ; If that didn't work, use a safe default
    if ${textboxWidth} <= 0
        textboxWidth:Set[1800]

    fontSize:Set[${LGUI2.Element[Output].Font.Height}]
    charWidth:Set[${Math.Calc[${fontSize} * 0.5]}]
    ; Reduce usable width by 10% for padding/margins
    maxChars:Set[${Math.Calc[(${textboxWidth} * 0.9) / ${charWidth}].Int}]

    ; Trim the line if it's too long
    if ${newLine.Length} > ${maxChars}
    {
        newLine:Set["${newLine.Left[${Math.Calc[${maxChars} - 3]}]}..."]
    }

    newLine:Concat["\n"]

    currentText:Set["${LGUI2.Element[Output].Text}"]
    currentText:Concat["${newLine}"]
    LGUI2.Element[Output]:SetText["${currentText.Escape}"]
}
```

**Key Points:**
- Use `.ActualWidth.Precision[0]` and `.ActualHeight.Precision[0]` for rendered dimensions
- These work even when JSON specifies percentage values
- Always check if the value is valid (`> 0`) and provide a fallback
- Character width estimation: `fontSize * 0.5` works well for Segoe UI

---

## Data Binding Migration

### Manual Updates (LGUI1 Pattern)

**LGUI1 XML:**

```xml
<Text Name="healthText">Health: 0</Text>
```

**LGUI1 Script:**

```lavishscript
; Manual update every frame
function main()
{
    ui -load myui.xml

    while 1
    {
        UIElement[healthText]:SetText["Health: ${Me.Health}"]
        waitframe
    }
}
```

**Problems:**
- ❌ Manual updates required every frame
- ❌ More code to maintain
- ❌ Performance overhead from SetText calls

### Automatic Data Binding (LGUI2 Pattern)

**LGUI2 JSON:**

```json
{
    "type": "textblock",
    "name": "healthText",
    "textBinding": {
        "pullFormat": "Health: ${Me.Health}"
    }
}
```

**LGUI2 Script:**

```lavishscript
; No manual updates needed!
function main()
{
    LGUI2:LoadPackageFile[myui.json]

    while 1
        waitframe
}
```

**Benefits:**
- ✅ Automatic updates
- ✅ Less code
- ✅ Better performance

### Checkbox State Persistence

**LGUI1 Pattern:**

```lavishscript
; Save state manually
variable bool AutoLootEnabled = FALSE

function ToggleAutoLoot()
{
    This.AutoLootEnabled:Set[${UIElement[autoloot]:GetChecked}]
}

function UpdateCheckbox()
{
    UIElement[autoloot]:SetChecked[${This.AutoLootEnabled}]
}
```

**LGUI2 Pattern:**

```json
{
    "type": "checkbox",
    "checkedBinding": "MyController.AutoLootEnabled",
    "content": "Auto-Loot"
}
```

```lavishscript
; Automatic synchronization!
variable bool AutoLootEnabled = FALSE
```

The checkbox state and variable are **automatically synchronized** both ways.

### Dynamic List Updates

**LGUI1 Pattern:**

```lavishscript
; Manually rebuild list
method UpdateTargetList()
{
    UIElement[targetlist]:ClearItems

    ; Add items manually
    UIElement[targetlist]:AddItem["Target 1"]
    UIElement[targetlist]:AddItem["Target 2"]
}
```

**LGUI2 Pattern:**

```json
{
    "type": "listbox",
    "name": "targetlist",
    "itemsBinding": {
        "pullFormat": "${MyController.GetTargetList}"
    }
}
```

```lavishscript
member:string GetTargetList()
{
    ; Return JSON array
    return "$$>[
        {\"type\":\"textblock\",\"text\":\"Target 1\"},
        {\"type\":\"textblock\",\"text\":\"Target 2\"}
    ]<$$"
}
```

The list **automatically updates** whenever `GetTargetList` returns different data.

---

## Complete Migration Examples

### Example 1: Simple Status Window

#### LGUI1 Version

**status_lgui1.xml:**

```xml
<?xml version='1.0'?>
<LGUI>
    <Window Name="statuswindow" Width="250" Height="150">
        <Title>Character Status</Title>
        <Text Name="nametext">Name: </Text>
        <Text Name="healthtext">Health: 0</Text>
        <Text Name="powertext">Power: 0</Text>
        <Text Name="fpstext">FPS: 0</Text>
    </Window>
</LGUI>
```

**status_lgui1.iss:**

```lavishscript
function main()
{
    ui -load status_lgui1.xml

    while 1
    {
        ; Manual updates every frame
        UIElement[nametext]:SetText["Name: ${Me.Name}"]
        UIElement[healthtext]:SetText["Health: ${Me.Health}/${Me.MaxHealth}"]
        UIElement[powertext]:SetText["Power: ${Me.Power}/${Me.MaxPower}"]
        UIElement[fpstext]:SetText["FPS: ${Display.FPS.Centi}"]
        waitframe
    }
}

function OnExit()
{
    ui -unload status_lgui1
}
```

#### LGUI2 Version

**status_lgui2.json:**

```json
{
    "$schema": "http://www.lavishsoft.com/schema/lgui2Package.json",
    "elements": [
        {
            "type": "window",
            "title": "Character Status",
            "name": "status.window",
            "x": 1480,
            "y": 960,
            "width": 800,
            "height": 800,
            "content": {
                "type": "stackpanel",
                "orientation": "vertical",
                "children": [
                    {
                        "type": "textblock",
                        "textBinding": {
                            "pullFormat": "Name: ${Me.Name}"
                        }
                    },
                    {
                        "type": "textblock",
                        "textBinding": {
                            "pullFormat": "Health: ${Me.Health}/${Me.MaxHealth}"
                        }
                    },
                    {
                        "type": "textblock",
                        "textBinding": {
                            "pullFormat": "Power: ${Me.Power}/${Me.MaxPower}"
                        }
                    },
                    {
                        "type": "textblock",
                        "textBinding": {
                            "pullFormat": "FPS: ${Display.FPS.Centi}"
                        }
                    }
                ]
            }
        }
    ]
}
```

**status_lgui2.iss:**

```lavishscript
objectdef status_controller
{
    method Initialize()
    {
        LGUI2:LoadPackageFile[status_lgui2.json]
    }

    method Shutdown()
    {
        LGUI2:UnloadPackageFile[status_lgui2.json]
    }
}

variable(global) status_controller StatusController

function main()
{
    ; No manual updates needed!
    while 1
        waitframe
}
```

**Improvements:**
- ✅ No manual SetText calls
- ✅ Automatic data binding
- ✅ Cleaner code
- ✅ Better performance

---

### Example 2: Combat Control Panel

#### LGUI1 Version

**combat_lgui1.xml:**

```xml
<?xml version='1.0'?>
<LGUI>
    <Window Name="combatwindow" Width="300" Height="200">
        <Title>Combat Controls</Title>
        <Checkbox Name="autoattack" OnLeftClick="CombatController:ToggleAutoAttack">
            <Text>Auto-Attack</Text>
        </Checkbox>
        <Checkbox Name="autoloot" OnLeftClick="CombatController:ToggleAutoLoot">
            <Text>Auto-Loot</Text>
        </Checkbox>
        <Button OnLeftClick="CombatController:Attack">
            <Text>Attack Target</Text>
        </Button>
        <Button OnLeftClick="CombatController:Stop">
            <Text>Stop Combat</Text>
        </Button>
        <Text Name="statustext">Status: Ready</Text>
    </Window>
</LGUI>
```

**combat_lgui1.iss:**

```lavishscript
objectdef combat_controller
{
    variable bool AutoAttackEnabled = FALSE
    variable bool AutoLootEnabled = FALSE
    variable string Status = "Ready"

    method Initialize()
    {
        ui -load combat_lgui1.xml
    }

    method Shutdown()
    {
        ui -unload combat_lgui1
    }

    method ToggleAutoAttack()
    {
        This.AutoAttackEnabled:Set[${UIElement[autoattack]:GetChecked}]
        echo Auto-Attack: ${This.AutoAttackEnabled}
    }

    method ToggleAutoLoot()
    {
        This.AutoLootEnabled:Set[${UIElement[autoloot]:GetChecked}]
        echo Auto-Loot: ${This.AutoLootEnabled}
    }

    method Attack()
    {
        echo Attacking!
        This.Status:Set["Attacking..."]
        This:UpdateStatus
        Me.Ability[1]:Use
    }

    method Stop()
    {
        echo Stopping
        This.Status:Set["Stopped"]
        This:UpdateStatus
    }

    method UpdateStatus()
    {
        UIElement[statustext]:SetText["Status: ${This.Status}"]
    }

    method Pulse()
    {
        ; Update status text every frame
        This:UpdateStatus
    }
}

variable(global) combat_controller CombatController

function main()
{
    while 1
    {
        CombatController:Pulse
        waitframe
    }
}
```

#### LGUI2 Version

**combat_lgui2.json:**

```json
{
    "$schema": "http://www.lavishsoft.com/schema/lgui2Package.json",
    "elements": [
        {
            "type": "window",
            "title": "Combat Controls",
            "name": "combat.window",
            "x": 1480,
            "y": 960,
            "width": 800,
            "height": 800,
            "content": {
                "type": "stackpanel",
                "orientation": "vertical",
                "children": [
                    {
                        "type": "checkbox",
                        "checkedBinding": "CombatController.AutoAttackEnabled",
                        "content": "Auto-Attack"
                    },
                    {
                        "type": "checkbox",
                        "checkedBinding": "CombatController.AutoLootEnabled",
                        "content": "Auto-Loot"
                    },
                    {
                        "type": "panel",
                        "height": 10
                    },
                    {
                        "type": "button",
                        "content": "Attack Target",
                        "eventHandlers": {
                            "onPress": {
                                "type": "method",
                                "object": "CombatController",
                                "method": "Attack"
                            }
                        }
                    },
                    {
                        "type": "button",
                        "content": "Stop Combat",
                        "eventHandlers": {
                            "onPress": {
                                "type": "method",
                                "object": "CombatController",
                                "method": "Stop"
                            }
                        }
                    },
                    {
                        "type": "panel",
                        "height": 10
                    },
                    {
                        "type": "textblock",
                        "textBinding": {
                            "pullFormat": "Status: ${CombatController.Status}"
                        }
                    }
                ]
            }
        }
    ]
}
```

**combat_lgui2.iss:**

```lavishscript
objectdef combat_controller
{
    variable bool AutoAttackEnabled = FALSE
    variable bool AutoLootEnabled = FALSE
    variable string Status = "Ready"

    method Initialize()
    {
        LGUI2:LoadPackageFile[combat_lgui2.json]
    }

    method Shutdown()
    {
        LGUI2:UnloadPackageFile[combat_lgui2.json]
    }

    ; No toggle methods needed - data binding handles it!

    method Attack()
    {
        echo Attacking!
        This.Status:Set["Attacking..."]
        ; No UpdateStatus call needed - automatic!
        Me.Ability[1]:Use
    }

    method Stop()
    {
        echo Stopping
        This.Status:Set["Stopped"]
        ; No UpdateStatus call needed - automatic!
    }

    ; No Pulse method needed!
}

variable(global) combat_controller CombatController

function main()
{
    while 1
        waitframe
}
```

**Code Reduction:**
- ❌ Removed `ToggleAutoAttack` and `ToggleAutoLoot` methods
- ❌ Removed `UpdateStatus` method
- ❌ Removed `Pulse` method
- ✅ **50% less code** with same functionality
- ✅ Automatic checkbox synchronization
- ✅ Automatic status text updates

---

## Migration Checklist

### Before You Start

- [ ] Back up all existing `.xml` and `.iss` files
- [ ] Review the [LavishGUI 2 UI Guide](08_LavishGUI2_UI_Guide.md)
- [ ] Install a JSON editor with schema support (VS Code recommended)
- [ ] Test LGUI2 with a simple example first

### For Each UI File

- [ ] Create new `.json` file (don't delete `.xml` yet)
- [ ] Add JSON schema reference
- [ ] Convert XML structure to JSON `elements` array
- [ ] Map element types (`<Window>` → `{"type": "window"}`)
- [ ] Convert attributes to JSON properties
- [ ] Migrate event handlers to `eventHandlers` objects
- [ ] Test loading the JSON file
- [ ] Verify all elements appear correctly

### For Each Script File

- [ ] Update `ui -load` to `LGUI2:LoadPackageFile`
- [ ] Update `ui -unload` to `LGUI2:UnloadPackageFile`
- [ ] Replace `UIElement[]` with `LGUI2.Element[]`
- [ ] Implement controller objectdef pattern
- [ ] Replace manual SetText calls with data binding
- [ ] Replace manual checkbox state management with data binding
- [ ] Remove unnecessary Pulse/Update methods
- [ ] Test all functionality

### Testing

- [ ] Verify all windows load
- [ ] Test all buttons
- [ ] Test all checkboxes
- [ ] Verify data binding updates
- [ ] Test event handlers
- [ ] Check window positioning
- [ ] Verify settings persistence

### Cleanup

- [ ] Remove old `.xml` files (after confirming migration works)
- [ ] Remove obsolete update methods from controllers
- [ ] Update documentation/comments
- [ ] Commit changes to version control

---

## Common Migration Issues

### Issue 1: JSON Syntax Errors

**Problem:**

```
Parse error: Unexpected token
```

**Causes:**
- Missing commas between properties
- Trailing commas (JSON doesn't allow them)
- Missing quotes around strings
- Single quotes instead of double quotes

**Fix:**

Use a JSON validator or IDE with JSON schema support. VS Code will highlight errors automatically.

```json
// BAD
{
    "type": "window"
    "title": "Test",    // Missing comma above
}

// GOOD
{
    "type": "window",
    "title": "Test"
}
```

### Issue 2: Element Not Found

**Problem:**

```lavishscript
${LGUI2.Element[mywindow](exists)} returns FALSE
```

**Causes:**
- Element name doesn't match JSON
- Package not loaded yet
- Typo in element name

**Fix:**

```lavishscript
; Wait for load
LGUI2:LoadPackageFile[myui.json]
wait 5 ${LGUI2.Element[mywindow](exists)}

if !${LGUI2.Element[mywindow](exists)}
{
    echo ERROR: Window failed to load!
    return
}
```

### Issue 3: Event Handler Not Firing

**Problem:**

Clicking button does nothing.

**Causes:**
- Controller object not global
- Method name typo
- Object name typo in JSON

**Fix:**

```lavishscript
; Controller must be global!
variable(global) mycontroller MyController

; Not this:
variable mycontroller MyController
```

```json
// Names must match exactly
"eventHandlers": {
    "onPress": {
        "type": "method",
        "object": "MyController",    // Must match variable name
        "method": "OnPressed"        // Must match method name exactly
    }
}
```

### Issue 4: Data Binding Not Updating

**Problem:**

Text doesn't update even though data changes.

**Causes:**
- Incorrect pullFormat syntax
- Variable doesn't exist
- Object not global

**Fix:**

```lavishscript
; Controller must be global for data binding
variable(global) mycontroller MyController

objectdef mycontroller
{
    variable string StatusText = "Ready"
}
```

```json
{
    "type": "textblock",
    "textBinding": {
        "pullFormat": "Status: ${MyController.StatusText}"
    }
}
```

### Issue 5: Checkbox State Not Syncing

**Problem:**

Checkbox state and variable out of sync.

**Causes:**
- Using `onPress` instead of data binding
- Controller variable not updated

**Fix:**

**Don't do this:**

```json
// BAD - Manual toggle
"eventHandlers": {
    "onPress": {
        "type": "method",
        "object": "Controller",
        "method": "Toggle"
    }
}
```

**Do this:**

```json
// GOOD - Automatic sync
"checkedBinding": "Controller.Enabled"
```

```lavishscript
variable bool Enabled = FALSE
; Variable automatically syncs with checkbox!
```

### Issue 6: Window Position Lost

**Problem:**

Window appears at wrong position after reload.

**Fix:**

Implement position persistence:

```lavishscript
method SaveWindowPosition()
{
    variable jsonvalue pos
    pos:SetInteger[x, ${LGUI2.Element[mywindow].X}]
    pos:SetInteger[y, ${LGUI2.Element[mywindow].Y}]
    ; Save pos to file or LavishSettings
}

method LoadWindowPosition()
{
    ; Load pos from file or LavishSettings
    variable jsonvalue pos
    LGUI2.Element[mywindow]:SetLocation[${pos.Get[x]}, ${pos.Get[y]}]
}
```

### Issue 7: Dynamic Visual Property Changes Not Working

**Problem:**

Trying to change element appearance at runtime fails:

```lavishscript
; LGUI1 pattern - DOES NOT WORK in LGUI2
LGUI2.Element[mytext]:SetAlpha[0.5]        ; ERROR: Method doesn't exist
LGUI2.Element[mybutton]:Set[opacity,0.5]   ; ERROR: Only works for simple properties
```

**Root Cause:**

LGUI2 has different limitations on dynamic property changes compared to LGUI1:

1. **No `SetAlpha` method** - LGUI2 doesn't have direct methods for many visual properties
2. **`:Set` limitations** - The `:Set[property,value]` method ONLY works for simple/top-level properties like `text` and `isOpen`
3. **`:Set` does NOT work** for nested properties like `opacity`, `font.color`, `backgroundBrush.color`

**The Solution: Style-Based Approach**

LGUI2 uses **styles** for dynamic visual changes. This is a paradigm shift from LGUI1's direct property manipulation.

**Pattern Overview:**

1. Define **named styles** in JSON with desired visual states
2. Create **custom event handlers** that apply styles
3. Use **`FireEventHandler`** from script to trigger style changes

#### Example 1: Textbox Opacity Changes

**LGUI1 Pattern (Old):**

```xml
<Textbox Name="mytext">Content</Textbox>
```

```lavishscript
; Direct property manipulation
UIElement[mytext]:SetAlpha[1.0]    ; Bright
UIElement[mytext]:SetAlpha[0.1]    ; Dimmed
```

**LGUI2 Pattern (New):**

```json
{
    "type": "textbox",
    "name": "mytext",
    "text": "Content",
    "styles": {
        "enabled": {
            "opacity": 1.0
        },
        "disabled": {
            "opacity": 0.1
        }
    },
    "eventHandlers": {
        "setEnabled": {
            "type": "style",
            "styleName": "enabled"
        },
        "setDisabled": {
            "type": "style",
            "styleName": "disabled"
        }
    }
}
```

```lavishscript
; Style-based changes
LGUI2.Element[mytext]:FireEventHandler[setEnabled]   ; Bright
LGUI2.Element[mytext]:FireEventHandler[setDisabled]  ; Dimmed
```

**Key Concepts:**

- **`styles`** - Object containing named style definitions
- **`styleName`** - Reference to a named style
- **`FireEventHandler`** - Triggers event handlers (including style application)
- **Custom event names** - You choose the names (`setEnabled`, `setDisabled`, etc.)

#### Example 2: Button Color Changes (Toggle State)

**Problem:** Buttons need to change color to indicate active/inactive state.

**Additional Challenge:** Buttons ignore `font.color` - use `backgroundBrush.color` instead.

**LGUI1 Pattern (Old):**

```xml
<Button Name="mybutton">
    <Text>Toggle Feature</Text>
</Button>
```

```lavishscript
; Change font color (worked in LGUI1)
UIElement[mybutton].Font:SetColor[FF32CD32]  ; Green when active
UIElement[mybutton].Font:SetColor[FFFF0000]  ; Red when inactive
```

**LGUI2 Pattern (New):**

```json
{
    "type": "button",
    "name": "mybutton",
    "content": "Toggle Feature",
    "backgroundBrush": {
        "color": "#FF0000"
    },
    "styles": {
        "green": {
            "backgroundBrush": {"color": "#32CD32"}
        },
        "red": {
            "backgroundBrush": {"color": "#FF0000"}
        }
    },
    "eventHandlers": {
        "onPress": {
            "type": "code",
            "code": "Script[MyScript]:QueueCommand[call ToggleFeature]"
        },
        "setGreen": {
            "type": "style",
            "styleName": "green"
        },
        "setRed": {
            "type": "style",
            "styleName": "red"
        }
    }
}
```

```lavishscript
variable bool FeatureActive = FALSE

function ToggleFeature()
{
    if ${FeatureActive}
    {
        FeatureActive:Set[FALSE]
        ; Color changes from GREEN to RED (button text stays "Toggle Feature")
        LGUI2.Element[mybutton]:FireEventHandler[setRed]
    }
    else
    {
        FeatureActive:Set[TRUE]
        ; Color changes from RED to GREEN (button text stays "Toggle Feature")
        LGUI2.Element[mybutton]:FireEventHandler[setGreen]
    }
}
```

**Why `backgroundBrush.color` instead of `font.color`?**

Buttons in LGUI2 **ignore** `font.color` and `foregroundBrush.color` for their content text. Use `backgroundBrush.color` to change button background color instead.

#### Example 3: Persistent Button Colors (Hover State Issue)

**Problem:** Button colors revert to default when mouse hovers over them.

**Root Cause:** The skin's built-in hover state overrides custom styles.

**Solution:** Define all brush states and re-apply on mouse events.

**Complete Pattern:**

```json
{
    "type": "button",
    "name": "toggleButton",
    "content": "Toggle Feature",
    "styles": {
        "green": {
            "backgroundBrush": {"color": "#32CD32"},
            "hoverBackgroundBrush": {"color": "#32CD32"},
            "pressedBackgroundBrush": {"color": "#228B22"},
            "disabledBackgroundBrush": {"color": "#90EE90"}
        },
        "red": {
            "backgroundBrush": {"color": "#FF0000"},
            "hoverBackgroundBrush": {"color": "#FF0000"},
            "pressedBackgroundBrush": {"color": "#8B0000"},
            "disabledBackgroundBrush": {"color": "#FF6666"}
        }
    },
    "eventHandlers": {
        "onPress": {
            "type": "code",
            "code": "Script[MyScript]:QueueCommand[call ToggleFeature]"
        },
        "setGreen": {
            "type": "style",
            "styleName": "green"
        },
        "setRed": {
            "type": "style",
            "styleName": "red"
        },
        "gotMouseOver": {
            "type": "code",
            "code": "Script[MyScript]:QueueCommand[call ReapplyButtonColor]"
        },
        "lostMouseOver": {
            "type": "code",
            "code": "Script[MyScript]:QueueCommand[call ReapplyButtonColor]"
        }
    }
}
```

```lavishscript
variable bool FeatureActive = FALSE
variable string ButtonState = "red"

function ToggleFeature()
{
    if ${FeatureActive}
    {
        FeatureActive:Set[FALSE]
        ButtonState:Set["red"]
        LGUI2.Element[toggleButton]:FireEventHandler[setRed]
    }
    else
    {
        FeatureActive:Set[TRUE]
        ButtonState:Set["green"]
        LGUI2.Element[toggleButton]:FireEventHandler[setGreen]
    }
}

function ReapplyButtonColor()
{
    if ${ButtonState.Equal["green"]}
        LGUI2.Element[toggleButton]:FireEventHandler[setGreen]
    else
        LGUI2.Element[toggleButton]:FireEventHandler[setRed]
}
```

**Critical Requirements:**

1. **Define all brush states** - `backgroundBrush`, `hoverBackgroundBrush`, `pressedBackgroundBrush`, `disabledBackgroundBrush`
2. **Track state in variable** - Needed for reapply function
3. **Handle mouse events** - `gotMouseOver` and `lostMouseOver` to re-apply styles
4. **Reapply function** - Checks state variable and re-fires appropriate style event

**Why This Pattern Works:**

- Skin hover states try to override your custom styles
- By handling `gotMouseOver`/`lostMouseOver`, you "fight back" and re-apply your custom colors
- Defining all brush state variants prevents skin from having any defaults to fall back to
- State tracking variable ensures reapply function knows which color to use

#### Complete Working Example: Multi-Button UI

**Real-world example from EQ2BotCommander migration:**

```json
{
    "$schema": "http://www.lavishsoft.com/schema/lgui2Package.json",
    "elements": [
        {
            "type": "window",
            "title": "Bot Commander",
            "content": {
                "type": "stackpanel",
                "orientation": "vertical",
                "children": [
                    {
                        "type": "button",
                        "name": "Main.RunEQ2Bot",
                        "content": "Run EQ2BOT",
                        "styles": {
                            "green": {
                                "backgroundBrush": {"color": "#32CD32"},
                                "hoverBackgroundBrush": {"color": "#32CD32"},
                                "pressedBackgroundBrush": {"color": "#228B22"},
                                "disabledBackgroundBrush": {"color": "#90EE90"}
                            },
                            "red": {
                                "backgroundBrush": {"color": "#FF0000"},
                                "hoverBackgroundBrush": {"color": "#FF0000"},
                                "pressedBackgroundBrush": {"color": "#8B0000"},
                                "disabledBackgroundBrush": {"color": "#FF6666"}
                            }
                        },
                        "eventHandlers": {
                            "onPress": {"type": "code", "code": "Script[EQ2BotCommander]:QueueCommand[call ToggleRunEQ2Bot]"},
                            "setGreen": {"type": "style", "styleName": "green"},
                            "setRed": {"type": "style", "styleName": "red"},
                            "gotMouseOver": {"type": "code", "code": "Script[EQ2BotCommander]:QueueCommand[call ReapplyRunEQ2BotButtonColor]"},
                            "lostMouseOver": {"type": "code", "code": "Script[EQ2BotCommander]:QueueCommand[call ReapplyRunEQ2BotButtonColor]"}
                        }
                    },
                    {
                        "type": "button",
                        "name": "Main.Follow",
                        "content": "Follow",
                        "styles": {
                            "green": {
                                "backgroundBrush": {"color": "#32CD32"},
                                "hoverBackgroundBrush": {"color": "#32CD32"},
                                "pressedBackgroundBrush": {"color": "#228B22"},
                                "disabledBackgroundBrush": {"color": "#90EE90"}
                            },
                            "red": {
                                "backgroundBrush": {"color": "#FF0000"},
                                "hoverBackgroundBrush": {"color": "#FF0000"},
                                "pressedBackgroundBrush": {"color": "#8B0000"},
                                "disabledBackgroundBrush": {"color": "#FF6666"}
                            }
                        },
                        "eventHandlers": {
                            "onPress": {"type": "code", "code": "Script[EQ2BotCommander]:QueueCommand[call FollowToggle]"},
                            "setGreen": {"type": "style", "styleName": "green"},
                            "setRed": {"type": "style", "styleName": "red"},
                            "gotMouseOver": {"type": "code", "code": "Script[EQ2BotCommander]:QueueCommand[call ReapplyFollowButtonColor]"},
                            "lostMouseOver": {"type": "code", "code": "Script[EQ2BotCommander]:QueueCommand[call ReapplyFollowButtonColor]"}
                        }
                    }
                ]
            }
        }
    ]
}
```

```lavishscript
variable bool EQ2BotRunning = FALSE
variable bool Following = FALSE

function ToggleRunEQ2Bot()
{
    if ${EQ2BotRunning} == FALSE
    {
        EQ2BotRunning:Set[TRUE]
        LGUI2.Element[Main.RunEQ2Bot]:FireEventHandler[setGreen]
        Relay all RunScript EQ2Bot/EQ2Bot
    }
    else
    {
        EQ2BotRunning:Set[FALSE]
        LGUI2.Element[Main.RunEQ2Bot]:FireEventHandler[setRed]
        Relay all EndScript EQ2Bot
    }
}

function ReapplyRunEQ2BotButtonColor()
{
    if ${EQ2BotRunning}
        LGUI2.Element[Main.RunEQ2Bot]:FireEventHandler[setGreen]
    else
        LGUI2.Element[Main.RunEQ2Bot]:FireEventHandler[setRed]
}

function FollowToggle()
{
    if ${Following} == FALSE
    {
        Following:Set[TRUE]
        LGUI2.Element[Main.Follow]:FireEventHandler[setGreen]
        Relay "all other" Script[EQ2Bot]:ExecuteAtom[AutoFollowTank]
    }
    else
    {
        Following:Set[FALSE]
        LGUI2.Element[Main.Follow]:FireEventHandler[setRed]
        Relay "all other" Script[EQ2Bot]:ExecuteAtom[StopAutoFollowing]
    }
}

function ReapplyFollowButtonColor()
{
    if ${Following}
        LGUI2.Element[Main.Follow]:FireEventHandler[setGreen]
    else
        LGUI2.Element[Main.Follow]:FireEventHandler[setRed]
}
```

**Reference Implementation:** [EQ2BotCommander.json](https://github.com/isxGames/isxScripts/blob/master/EverQuest2/Scripts/EQ2Bot/UI/EQ2BotCommander.json) and [EQ2BotCommander.iss](https://github.com/isxGames/isxScripts/blob/master/EverQuest2/Scripts/EQ2BotCommander.iss)

#### Summary: Style-Based Visual Changes

**When to Use This Pattern:**

- ✅ Changing element opacity
- ✅ Changing button background colors
- ✅ Changing any visual property not supported by direct methods
- ✅ Toggle buttons that need visual state indication
- ✅ Any dynamic visual change that worked via `SetAlpha` or `Font:SetColor` in LGUI1

**Key Takeaways:**

1. **LGUI2 does NOT support** direct property manipulation like LGUI1's `SetAlpha` or `Font:SetColor`
2. **Use styles** for all dynamic visual changes
3. **Use `FireEventHandler`** to apply styles from script
4. **Buttons ignore font color** - use `backgroundBrush.color` instead
5. **Define all brush states** to prevent skin hover override
6. **Handle mouse events** to re-apply colors on hover
7. **Track state in variables** for reapply functions

**What Properties Need Styles?**

Properties requiring styles (can't use `:Set`):
- `opacity` - Element transparency
- `backgroundBrush.color` - Background color
- `foregroundBrush.color` - Foreground color (limited use)
- `borderBrush.color` - Border color
- `font.color` - Text color (limited use)
- Any nested property with dot notation

Properties that work with dedicated set methods:
- `text` - Text content for textblocks (use `:SetText`)
- `isOpen` - Open/closed state
- `checked` - Checkbox state (use `:SetChecked`)
- Other simple top-level boolean/string properties

**CRITICAL LIMITATION:** Button `content` (text) **cannot be changed dynamically** in LGUI2. The `SetContent` method exists but is for setting **element content** (child elements), not text strings.

**Workaround:** Use static button labels with color changes to indicate state:
```json
{
    "type": "button",
    "content": "Toggle Feature",
    "tooltip": "Toggle feature (RED=Off, GREEN=On)"
}
```

```lavishscript
; Change only the color, not the text
LGUI2.Element[myButton]:FireEventHandler[setGreen]  ; Active
LGUI2.Element[myButton]:FireEventHandler[setRed]     ; Inactive
```

**Common Pitfall:**

```lavishscript
; DON'T DO THIS - Will fail!
LGUI2.Element[myButton]:Set[opacity,0.5]
LGUI2.Element[myButton]:Set[backgroundBrush.color,"#FF0000"]

; DO THIS INSTEAD - Use styles!
LGUI2.Element[myButton]:FireEventHandler[setDimmed]
LGUI2.Element[myButton]:FireEventHandler[setRed]
```

### Issue 8: Window Position "Jumping" on Load

**Problem:**

Window briefly appears at default JSON position, then jumps to saved position.

**Cause:**

Window positions are applied AFTER the window is already visible:

```lavishscript
LGUI2:LoadPackageFile[myui.json]        ; Window loads and becomes visible
call config_load                         ; Position applied here - too late!
```

**Fix:**

Load window hidden, apply position, then show it:

**JSON - Set window to start hidden:**

```json
{
    "type": "window",
    "name": "mywindow",
    "visibility": "hidden",
    ...
}
```

**Script - Show window after positioning:**

```lavishscript
; Load UI (window starts hidden)
LGUI2:LoadPackageFile[myui.json]

; Wait for window to load
wait 10 ${LGUI2.Element[mywindow](exists)}

; Apply saved position
call config_load

; Now show the window
LGUI2.Element[mywindow]:SetVisibility[visible]
```

**For dynamically loaded windows (like config dialogs):**

Use an `onLoad` event handler to apply position and show:

```json
{
    "type": "window",
    "name": "ConfigWindow",
    "visibility": "hidden",
    "eventHandlers": {
        "onLoad": {
            "type": "code",
            "code": "Script[myscript]:QueueCommand[call ApplyConfigWindowPosition]"
        }
    }
}
```

```lavishscript
function ApplyConfigWindowPosition()
{
    ; Load and apply position
    variable int x = ${Settings.FindSetting[ConfigWindowX,-1]}
    variable int y = ${Settings.FindSetting[ConfigWindowY,-1]}

    if ${x} >= 0 && ${y} >= 0
        LGUI2.Element[ConfigWindow]:SetLocation[${x}, ${y}]

    ; Show window after positioning
    LGUI2.Element[ConfigWindow]:SetVisibility[visible]
}
```

### Issue 9: Textbox Content Doesn't Fill Window

**Problem:**

Textbox content area has gaps/doesn't fill the window completely.

**Cause:**

Textbox has explicit `x` and `y` positioning when it should fill the content area:

```json
{
    "content": {
        "type": "textbox",
        "x": 0,
        "y": 0,
        "width": "100%",
        "height": "100%"
    }
}
```

**Fix:**

Remove `x` and `y` properties - content elements don't need positioning:

```json
{
    "content": {
        "type": "textbox",
        "width": "100%",
        "height": "100%"
    }
}
```

**Key Point:**
When elements use `"width": "100%"` and `"height": "100%"` inside a `content` property, they should NOT have `x` or `y` positioning.

### Issue 10: AppendText Method Not Found

**Problem:**

```
Error: No such 'lgui2textbox' method 'AppendText'
```

**Cause:**

LGUI2 textboxes don't have an `AppendText` method like LGUI1.

**Fix:**

Get current text, concatenate, and set it back:

```lavishscript
; LGUI1 (Old)
UIElement[Output]:AppendText["New line\n"]

; LGUI2 (New)
variable string currentText
variable string newLine = "New line\n"

currentText:Set["${LGUI2.Element[Output].Text}"]
currentText:Concat["${newLine}"]
LGUI2.Element[Output]:SetText["${currentText.Escape}"]
```

See [Textbox Operations](#textbox-operations) section for complete examples.

### Issue 11: Checkbox State Not Persisting (QueueCommand Execution)

**Problem:**

Checkbox states don't save when closing configuration window. Checkboxes revert to their previous state when reopening.

**Example Symptoms:**
- Uncheck boxes, close window, reopen → boxes are checked again
- Configuration file shows old values even after toggling checkboxes
- Variables never change from their default values

**Root Cause:**

Inline variable assignments in `QueueCommand` are queued but **don't execute** before the config save runs:

```json
// BAD - Inline assignment doesn't execute immediately
{
    "type": "checkbox",
    "eventHandlers": {
        "onChecked": {
            "type": "code",
            "code": "Script[MyScript]:QueueCommand[MyVar:Set[TRUE]]"
        }
    }
}
```

**What happens:**
1. User clicks checkbox
2. Command `MyVar:Set[TRUE]` gets added to queue
3. Window close handler runs `config_save()` immediately
4. Variable still has old value (queued command hasn't executed yet)
5. Old value gets saved to file

**The Fix: Use Handler Functions**

Create dedicated handler functions that execute when `ExecuteQueued` runs:

**JSON:**

```json
{
    "type": "checkbox",
    "name": "Options.AutoLoot",
    "checked": "${Script[MyScript].Variable[AutoLootEnabled]}",
    "eventHandlers": {
        "onChecked": {
            "type": "code",
            "code": "Script[MyScript]:QueueCommand[call OnAutoLootChecked]"
        },
        "onUnchecked": {
            "type": "code",
            "code": "Script[MyScript]:QueueCommand[call OnAutoLootUnchecked]"
        }
    }
}
```

**Script:**

```lavishscript
variable bool AutoLootEnabled = FALSE

function OnAutoLootChecked()
{
    AutoLootEnabled:Set[TRUE]
}

function OnAutoLootUnchecked()
{
    AutoLootEnabled:Set[FALSE]
}

function CloseConfigWindow()
{
    ; Execute all queued checkbox commands BEFORE saving
    ExecuteQueued

    ; Now save config (variables have correct values)
    call config_save

    ; Unload UI
    LGUI2:UnloadPackageFile["${Script.CurrentDirectory}/Interface/ConfigUI.json"]
}

function config_save()
{
    Settings:AddSetting[AutoLootEnabled,${AutoLootEnabled}]
    LavishSettings[MyScript]:Export[${ConfigFile}]
}
```

**Why This Works:**

1. User clicks checkbox → `QueueCommand[call OnAutoLootChecked]` queues function call
2. `CloseConfigWindow` runs `ExecuteQueued` → function call executes immediately
3. `OnAutoLootChecked()` runs → `AutoLootEnabled:Set[TRUE]` executes
4. `config_save()` runs → saves correct value

**Complete Pattern for Persistent Checkboxes:**

```lavishscript
; 1. Declare variable
variable bool MyFeature = FALSE

; 2. Create handler functions
function OnMyFeatureChecked()
{
    MyFeature:Set[TRUE]
}

function OnMyFeatureUnchecked()
{
    MyFeature:Set[FALSE]
}

; 3. Load setting from config file
function config_load()
{
    MyFeature:Set[${Settings.FindSetting[MyFeature,FALSE]}]
}

; 4. Save setting to config file
function config_save()
{
    ExecuteQueued  ; CRITICAL - Execute queued commands first
    Settings:AddSetting[MyFeature,${MyFeature}]
    LavishSettings[MyScript]:Export[${ConfigFile}]
}

; 5. Sync checkbox state when opening config window
function SyncConfigCheckboxes()
{
    LGUI2.Element[Options.MyFeature]:SetChecked[${MyFeature}]
}
```

**JSON Pattern:**

```json
{
    "type": "checkbox",
    "name": "Options.MyFeature",
    "checked": "${Script[MyScript].Variable[MyFeature]}",
    "eventHandlers": {
        "onChecked": {
            "type": "code",
            "code": "Script[MyScript]:QueueCommand[call OnMyFeatureChecked]"
        },
        "onUnchecked": {
            "type": "code",
            "code": "Script[MyScript]:QueueCommand[call OnMyFeatureUnchecked]"
        }
    }
}
```

**Key Differences from LGUI1:**

| LGUI1 | LGUI2 |
|-------|-------|
| `OnLeftClick` fires immediately | Events queue via `QueueCommand` |
| Direct variable access in handler | Must use handler functions |
| No `ExecuteQueued` needed | **MUST** call `ExecuteQueued` before save |

**Common Mistakes:**

```lavishscript
// DON'T DO THIS - Inline assignment won't execute
"code": "Script[MyScript]:QueueCommand[MyVar:Set[TRUE]]"

// DON'T DO THIS - Saving before ExecuteQueued
function CloseWindow()
{
    call config_save    ; BAD - Queued commands haven't run yet
    ExecuteQueued       ; Too late!
}

// DO THIS - Function call + ExecuteQueued before save
"code": "Script[MyScript]:QueueCommand[call OnMyVarChecked]"

function CloseWindow()
{
    ExecuteQueued       ; Execute queued commands first
    call config_save    ; Now variables have correct values
}
```

**Real-World Example:**

See [EQ2AFKAlarm.iss](https://github.com/isxGames/isxScripts/blob/master/EverQuest2/Scripts/EQ2AFKAlarm/EQ2AFKAlarm.iss) for complete implementation with:
- 13 persistent checkboxes (6 trigger channels + 6 TTS options + 1 logging option)
- Handler function pairs for each checkbox
- Proper `ExecuteQueued` usage in save function
- Config window position persistence

**Debug Tip:**

Add debug output to verify execution order:

```lavishscript
function OnMyFeatureChecked()
{
    echo "OnMyFeatureChecked: Setting MyFeature to TRUE"
    MyFeature:Set[TRUE]
}

function config_save()
{
    echo "config_save: MyFeature=${MyFeature}"
    ExecuteQueued
    echo "config_save: After ExecuteQueued, MyFeature=${MyFeature}"
    Settings:AddSetting[MyFeature,${MyFeature}]
}
```

If you see the old value in both echoes, the queued command didn't execute - you forgot `ExecuteQueued` or used inline assignment instead of a function call.

### Issue 12: MessageBox Command Not Available

**Problem:**

LGUI1's `MessageBox` command creates an XML-based dialog window. This command does not exist in LGUI2.

```lavishscript
; LGUI1 (Old)
MessageBox "Configuration version mismatch, resetting to defaults..."
```

**Error:**
```
Error: No such command 'MessageBox'
```

**The Fix: Create a Custom LGUI2 MessageBox**

Create a reusable message box window that can be loaded dynamically when needed.

**Step 1: Create MessageBox JSON (`Interface/MessageBox.json`):**

```json
{
  "$schema":"http://www.lavishsoft.com/schema/lgui2Package.json",
  "elements":[
    {
      "type":"window",
      "name":"MyScript.MessageBox",
      "x":800,
      "y":500,
      "width":1200,
      "height":600,
      "titleBar":{
        "type":"dockpanel",
        "children":[
          {
            "type":"textblock",
            "name":"MessageBox.Title",
            "text":"Message",
            "verticalAlignment":"center",
            "margin":[30, 0, 0, 0],
            "font":{"face":"Segoe UI", "height":54, "bold":true},
            "color":"#FFFFFF",
            "_dock":"left"
          }
        ]
      },
      "content":{
        "type":"stackpanel",
        "orientation":"vertical",
        "children":[
          {
            "type":"textblock",
            "name":"MessageBox.Text",
            "text":"Message text goes here",
            "margin":60,
            "wrapText":true,
            "font":{"face":"Segoe UI", "height":42}
          },
          {
            "type":"panel",
            "height":30
          },
          {
            "type":"button",
            "name":"MessageBox.OKButton",
            "content":"OK",
            "width":300,
            "height":120,
            "horizontalAlignment":"center",
            "font":{"height":48},
            "eventHandlers":{
              "onPress":{
                "type":"code",
                "code":"Script[MyScript]:QueueCommand[call CloseMessageBox]"
              }
            }
          }
        ]
      }
    }
  ]
}
```

**Step 2: Create MessageBox Functions:**

```lavishscript
variable bool MessageBoxClosed = FALSE

function ShowMessageBox(string title, string message)
{
    ; Load the message box UI
    LGUI2:LoadPackageFile["${Script.CurrentDirectory}/Interface/MessageBox.json"]

    ; Wait for it to load
    wait 5 ${LGUI2.Element[MyScript.MessageBox](exists)}

    ; Set the title and message
    LGUI2.Element[MessageBox.Title]:SetText["${title.Escape}"]
    LGUI2.Element[MessageBox.Text]:SetText["${message.Escape}"]

    ; Reset the closed flag
    MessageBoxClosed:Set[FALSE]

    ; Wait for the user to close it (same pattern as checkbox persistence)
    while !${MessageBoxClosed}
    {
        ExecuteQueued
        waitframe
    }
}

function CloseMessageBox()
{
    ; Set the flag to exit the wait loop
    MessageBoxClosed:Set[TRUE]

    ; Unload the message box UI
    LGUI2:UnloadPackageFile["${Script.CurrentDirectory}/Interface/MessageBox.json"]
}
```

**Step 3: Add Cleanup on Script Shutdown:**

```lavishscript
function main()
{
    ; ... main loop ...

    echo MyScript shutting down...

    ; Close message box if it's open
    if ${LGUI2.Element[MyScript.MessageBox](exists)}
    {
        LGUI2:UnloadPackageFile["${Script.CurrentDirectory}/Interface/MessageBox.json"]
    }

    call config_save
}
```

**Step 4: Usage:**

```lavishscript
; LGUI2 (New)
call ShowMessageBox "Config Reset" "Configuration version mismatch.\n\nResetting to defaults..."
```

**Why the Wait Loop is Necessary:**

The message box uses the same `QueueCommand` + `ExecuteQueued` pattern as persistent checkboxes:

1. User clicks OK → `call CloseMessageBox` gets queued
2. `ShowMessageBox` enters wait loop calling `ExecuteQueued` each frame
3. `ExecuteQueued` executes the queued `CloseMessageBox` function
4. `CloseMessageBox` sets `MessageBoxClosed:Set[TRUE]` and unloads UI
5. Wait loop exits, allowing the calling code to continue

**Common Mistakes:**

```lavishscript
// DON'T DO THIS - No wait loop
function ShowMessageBox(string title, string message)
{
    LGUI2:LoadPackageFile["MessageBox.json"]
    LGUI2.Element[MessageBox.Title]:SetText["${title}"]
    LGUI2.Element[MessageBox.Text]:SetText["${message}"]
    ; BAD - Returns immediately, message box stays open forever
}

// DO THIS - Wait for user to close it
function ShowMessageBox(string title, string message)
{
    LGUI2:LoadPackageFile["MessageBox.json"]
    LGUI2.Element[MessageBox.Title]:SetText["${title}"]
    LGUI2.Element[MessageBox.Text]:SetText["${message}"]

    MessageBoxClosed:Set[FALSE]
    while !${MessageBoxClosed}
    {
        ExecuteQueued  ; Process the close command when OK is clicked
        waitframe
    }
}
```

**Scaling the MessageBox:**

The example above uses 3x scaling. For 1x (baseline), use these values:
- Window: width:400, height:200
- Title font: height:18
- Message font: height:14
- Button: width:100, height:40, font:16
- Margins: 10-20 instead of 30-60

**Real-World Example:**

See [EQ2AFKAlarm.iss](https://github.com/isxGames/isxScripts/blob/master/EverQuest2/Scripts/EQ2AFKAlarm/EQ2AFKAlarm.iss) and [EQ2AFKAlarm_MessageBox.json](https://github.com/isxGames/isxScripts/blob/master/EverQuest2/Scripts/EQ2AFKAlarm/Interface/EQ2AFKAlarm_MessageBox.json) for complete implementation.

**Key Differences from LGUI1:**

| LGUI1 | LGUI2 |
|-------|-------|
| `MessageBox "text"` command | `ShowMessageBox()` function |
| Automatic wait for user | Manual wait loop with `ExecuteQueued` |
| No cleanup needed | Must unload on shutdown |
| Fixed appearance | Fully customizable JSON |

---

## Advanced Migration Topics

### Migrating Templates

**LGUI1 Templates:**

```xml
<Template Name="MyButton">
    <Button>
        <BackgroundColor>#0000ff</BackgroundColor>
        <Text>Default Text</Text>
    </Button>
</Template>
```

**LGUI2 Templates:**

```json
{
    "templates": {
        "myButton": {
            "backgroundBrush": {"color": "#0000ff"},
            "padding": 5
        }
    },
    "elements": [
        {
            "jsonTemplate": "myButton",
            "type": "button",
            "content": "Custom Text"
        }
    ]
}
```

### Migrating Skins

**LGUI1:**

```lavishscript
ui -loadskin myskin
ui -load myui.xml
```

**LGUI2:**

```lavishscript
LGUI2:PushSkin[myskin]
LGUI2:LoadPackageFile[myui.json]
LGUI2:PopSkin[myskin]
```

### Migrating Complex Layouts

For complex nested layouts, break them into logical sections:

**LGUI2 Pattern:**

```json
{
    "type": "dockpanel",
    "children": [
        {
            "_dock": "top",
            "type": "panel",
            "height": 50,
            "children": [/* Header content */]
        },
        {
            "_dock": "left",
            "type": "panel",
            "width": 200,
            "children": [/* Sidebar content */]
        },
        {
            "type": "panel",
            "children": [/* Main content fills remaining space */]
        }
    ]
}
```

---

## Summary

### Key Takeaways

1. **File Format:** XML → JSON
2. **Loading:** `ui -load` → `LGUI2:LoadPackageFile`
3. **Access:** `UIElement[]` → `LGUI2.Element[]`
4. **Events:** XML attributes → JSON `eventHandlers` objects
5. **Data Binding:** Manual updates → Automatic `textBinding`/`checkedBinding`
6. **Controller Pattern:** Use objectdef with Initialize/Shutdown

### Benefits of Migration

- ✅ **Less code** - Data binding eliminates manual updates
- ✅ **Better tooling** - JSON schema provides IDE support
- ✅ **More powerful** - Advanced features like triggers, item view generators
- ✅ **Future-proof** - LGUI2 is actively developed, LGUI1 is legacy
- ✅ **Easier maintenance** - Cleaner, more readable JSON syntax

### UI Scaling Enhancement

After migrating to LGUI2, consider adding **dynamic UI scaling** to your script:

**Why Add Scaling?**
- Users have different screen resolutions and DPI settings
- 4K displays benefit from 2x scaling
- Compact mode (0.5x) useful for multi-boxing

**How to Add Scaling:**

See the comprehensive **[LGUI2 Scaling System Guide](12_LGUI2_Scaling_System.md)** for complete implementation details.

**Quick Implementation:**

```lavishscript
#include ${LavishScript.HomeDirectory}/Scripts/LGUI2Scaling.iss

function main()
{
    variable float uiScale = 1.0

    if (!${uiScale.Equal[1.0]})
    {
        call ScaleUIJson "MyUI.json" "MyUI_Scaled.json" ${uiScale}
        LGUI2:LoadPackageFile["MyUI_Scaled.json"]
    }
    else
        LGUI2:LoadPackageFile["MyUI.json"]
}
```

**Real Example:** `EQ2BotCommander_LGUI2.iss` implements full UI scaling.

### Next Steps

1. Start with a simple UI to practice migration
2. Review the [LavishGUI 2 UI Guide](10_LavishGUI2_UI_Guide.md) for complete reference
3. Migrate one UI at a time
4. Test thoroughly before removing old XML files
5. **Consider adding UI scaling** - See [LGUI2 Scaling System](12_LGUI2_Scaling_System.md)
6. Use version control to track changes

---

## Additional Resources

- **LavishGUI 2 UI Guide:** [10_LavishGUI2_UI_Guide.md](10_LavishGUI2_UI_Guide.md)
- **LGUI2 Scaling System:** [12_LGUI2_Scaling_System.md](12_LGUI2_Scaling_System.md)
- **LavishGUI 1 Reference:** [08_LavishGUI1_UI_Guide.md](08_LavishGUI1_UI_Guide.md)
- **Official LGUI2 Wiki:** https://www.lavishsoft.com/wiki/index.php/LavishGUI_2
- **LERN Examples:** https://github.com/LavishSoftware/LERN/tree/master/LGUI2
- **Example Implementation:** EQ2BotCommander LGUI1→LGUI2 migration with scaling

---

**This migration guide was created through analysis of both LavishGUI 1 and LavishGUI 2 systems, including:**
- Complete LGUI1 documentation and examples
- 60+ LGUI2 example files
- Production migration patterns
- Real-world conversion experiences

**Good luck with your migration!**

## Known Limitations

### Custom C++ Element Types Not Supported

**Critical Limitation:** LavishGUI 2 JSON packages **do not support custom C++ element types** registered via `LGUIFactory`.

#### What This Means

If your LGUI1 UI uses custom C++ elements (registered with `LGUIFactory<YourClass>`), you **cannot migrate** to LGUI2 JSON packages. These elements will produce errors like:

```
element type not found: yourelementtype
```

#### Examples of Affected Elements

- **ISXEQ2 Radar** (`eq2radar`) - Custom radar display with 3D-to-2D coordinate conversion, dynamic blips, and interactive tooltips
- **Custom HUD elements** - Game-specific overlays with custom rendering
- **Advanced interactive displays** - Elements with complex mouse interaction and real-time updates

#### Technical Details

**LGUI1 Pattern (Works):**
```cpp
// C++ Extension Code
LGUIFactory<LGUIEQ2Radar> RadarFactory("eq2radar");

class LGUIEQ2Radar : public LGUIElement
{
    void Render() { /* custom rendering */ }
    // ... custom behavior
};
```

```xml
<!-- LGUI1 XML (Works) -->
<eq2radar Name="radar1">
    <Width>100%</Width>
    <Height>100%</Height>
</eq2radar>
```

**LGUI2 Pattern (Does NOT Work):**
```json
{
    "$schema": "http://www.lavishsoft.com/schema/lgui2Package.json",
    "elements": [
        {
            "type": "eq2radar",
            "name": "radar1"
        }
    ]
}
```

**Error:**
```
element type not found: eq2radar
```

#### Why This Happens

LavishGUI 2's JSON package system only recognizes built-in element types:
- Layout: window, panel, dockpanel, stackpanel, etc.
- Display: textblock, imagebox, progressbar, etc.
- Interaction: button, checkbox, listbox, combobox, etc.

Custom C++ element types registered with `LGUIFactory` are **not accessible** from JSON packages.

#### Workarounds

There are no perfect workarounds, but options include:

1. **Keep Using LGUI1** - If your custom element is critical, continue using XML-based LGUI1
   ```lavishscript
   ui -reload -skin YourSkin x64\\extensions\\yourui
   ```

2. **Redesign Without Custom Elements** - If possible, rebuild functionality using standard LGUI2 elements
   - Use `imagebox` with dynamically generated textures
   - Use standard controls with data binding
   - Note: This may lose interactivity and features

3. **Wait for LGUI2 Support** - LavishSoft may add custom element support in the future
   - No timeline or confirmation this will happen
   - Monitor LavishSoft wiki and forums for updates

#### Affected ISXEQ2 Features

The following ISXEQ2 features **must remain on LGUI1**:

- **Radar Command** (`radar on`) - Uses custom `eq2radar` element type
  - File: `x64/extensions/ISXEQ2Radar.xml`
  - Cannot be converted to JSON
  - Custom C++ rendering for 3D radar display

#### Investigation Conducted

Exhaustive research was performed to find LGUI2 custom element support:

✅ **Searched:**
- LavishSoft wiki (lavishsoft.com)
- LavishGUI 2 documentation
- LGUI2:Elements reference
- InnerSpace extension documentation
- ISXEQ2 forums and documentation
- LERN example repository

❌ **Not Found:**
- Any mention of `LGUIFactory` in LGUI2
- Documentation for registering custom element types in LGUI2
- Migration path for custom LGUI1 elements
- C++ API for LGUI2 custom elements

**Conclusion:** As of October 2025, custom C++ element types are **not supported** in LavishGUI 2 JSON packages.

#### When to Use LGUI1 vs LGUI2

**Use LGUI1 (XML) when:**
- ✅ You have custom C++ element types
- ✅ You use `LGUIFactory<>` registered elements
- ✅ Migration would break critical functionality

**Use LGUI2 (JSON) when:**
- ✅ You only use standard element types
- ✅ You want data binding and modern UI features
- ✅ You can rebuild custom elements with standard controls

---

## LGUI2-Exclusive Features

These powerful features are **only available in LavishGUI 2** and have no LGUI1 equivalent:

> **Note:** For complete documentation with detailed examples, property tables, and advanced patterns, see **10_LavishGUI2_UI_Guide.md**. This section provides a migration-focused overview of new capabilities.

### Animations

LGUI2 provides a complete animation system for smooth transitions and effects:

```json
{
    "type": "button",
    "animations": {
        "fadeIn": {
            "type": "fade",
            "duration": 500,
            "from": 0.0,
            "to": 1.0
        }
    },
    "eventHandlers": {
        "gotMouseOver": {
            "type": "animation",
            "animation": "fadeIn"
        }
    }
}
```

**Available Animation Types:**
- `fade` - Opacity transitions
- `slide` - Position animations
- `value` - Numeric value animations
- `chain` - Sequential animations
- `composite` - Parallel animations
- `delay` - Timed pauses
- `repeat` - Looping animations

### Event Hooks

Attach event handlers to events from other elements:

```json
{
    "type": "textblock",
    "hooks": {
        "dataRefresh": {
            "elementName": "eventHub",
            "flags": "global",
            "event": "onDataUpdated",
            "eventHandler": {
                "type": "code",
                "code": "This:Pull"
            }
        }
    }
}
```

**Use Cases:**
- Coordinating behavior across multiple UI elements
- Creating event broadcasting systems
- Decoupling UI components

### Advanced Brushes

LGUI2 brushes support advanced features:

```json
{
    "backgroundBrush": {
        "color": [1.0, 0.5, 0.5, 1.0],
        "imageFile": "Interface/textures/button.png",
        "imageOrientation": "mirrorHorizontal",
        "canvas": { /* Canvas definition */ },
        "pixelShader": { /* Shader definition */ }
    }
}
```

**New Brush Features:**
- Canvas rendering
- Image orientation (mirroring)
- Pixel and vertex shaders
- Image transparency keys
- Advanced tinting/blending

### Box Model Control

LGUI2 provides explicit control over the box model:

```json
{
    "type": "panel",
    "width": 100,
    "height": 100,
    "margin": [5, 10, 5, 10],
    "padding": 10,
    "borderThickness": 2
}
```

**Key Difference:** Padding is INSIDE bounds (reduces content area), Margin is OUTSIDE bounds.

### Factor-Based Positioning

Proportional sizing relative to parent:

```json
{
    "widthFactor": 0.5,
    "heightFactor": 0.75,
    "xFactor": 0.25,
    "yFactor": 0.5
}
```

Combine with offsets: `final = offset + (factor × parentDimension)`

### Metadata System

Store custom data using underscore properties:

```json
{
    "type": "button",
    "_customData": "myValue",
    "_priority": 10,
    "_category": "combat"
}
```

Access from script:

```lavishscript
echo ${LGUI2.Element[myButton].Metadata.Get[customData]}
```

### Automatic Data Binding

LGUI2's most powerful feature - automatic UI updates:

```json
{
    "type": "textblock",
    "textBinding": {
        "pullFormat": "${Me.Health}",
        "pullHook": {
            "elementName": "events",
            "flags": "global",
            "event": "onHealthChanged"
        }
    }
}
```

**Benefits:**
- No manual `SetText` calls
- Event-driven updates via `pullHook`
- Two-way binding with `pushFormat`
- Context-aware bindings

### New UI Elements (No LGUI1 Equivalent)

LGUI2 introduces many new element types that have no counterpart in LGUI1:

#### Interactive Controls

**Slider** - Draggable numeric value selection
```json
{
    "type": "slider",
    "orientation": "horizontal",
    "minValue": 0,
    "maxValue": 100,
    "value": 50
}
```

**Knob** - Rotary control for fine-tuning values
```json
{
    "type": "knob",
    "minValue": 0,
    "maxValue": 10,
    "value": 5
}
```

**CommandBox** - Advanced text input with command history
```json
{
    "type": "commandbox",
    "placeholder": "Enter command..."
}
```

**InputPicker** - Combo input (free text + dropdown suggestions)
```json
{
    "type": "inputpicker",
    "comboMode": true,
    "items": []
}
```

#### Progress & Indicators

**ProgressBar** - Visual progress indicator
```json
{
    "type": "progressbar",
    "minValue": 0,
    "maxValue": 100,
    "value": 75
}
```

**RadialGauge** - Circular gauge (speedometer-style)
```json
{
    "type": "radialgauge",
    "startAngle": -135,
    "endAngle": 135,
    "value": 65
}
```

#### Layout & Organization

**ScrollViewer** - Scrollable container
```json
{
    "type": "scrollviewer",
    "horizontalScrollBarVisibility": "auto",
    "verticalScrollBarVisibility": "auto",
    "content": { /* scrollable content */ }
}
```

**Expander** - Collapsible sections
```json
{
    "type": "expander",
    "header": "Advanced Options",
    "isExpanded": false,
    "content": { /* collapsible content */ }
}
```

**Dragger** - Resizable panel dividers
```json
{
    "type": "dragger",
    "allowResize": true,
    "width": 5
}
```

#### Menus & Popups

**Popup** - Floating windows
```json
{
    "type": "popup",
    "placementTarget": "buttonName",
    "placement": "bottom",
    "isOpen": false
}
```

**ContextMenu** - Right-click menus
```json
{
    "contextMenu": {
        "type": "contextmenu",
        "children": [ /* menu items */ ]
    }
}
```

**Use Cases for New Elements:**

| Element | Replace LGUI1 Pattern | Primary Use |
|---------|----------------------|-------------|
| Slider | Custom drag logic | Volume, opacity, zoom controls |
| ProgressBar | Manual bar drawing | Health/mana bars, loading indicators |
| ScrollViewer | Manual scrolling | Long content, logs, lists |
| Expander | Show/hide button + panel | Settings panels, accordion UIs |
| Popup | Tooltip windows | Tooltips, dropdown menus |
| ContextMenu | Manual menu windows | Right-click actions |
| CommandBox | Basic textbox | Console input, command execution |
| Dragger | Fixed layouts | Resizable split-pane interfaces |

### Advanced Event Handler Types

LGUI2 adds new event handler types beyond LGUI1's simple callbacks:

**Forward Handler** - Delegate events to other elements
```json
{
    "eventHandlers": {
        "onPress": {
            "type": "forward",
            "elementName": "targetElement",
            "event": "onPress"
        }
    }
}
```

**Animation Handler** - Trigger animations
```json
{
    "eventHandlers": {
        "gotMouseOver": {
            "type": "animation",
            "animation": "fadeIn"
        }
    }
}
```

**Task Handler** - Execute background tasks
```json
{
    "eventHandlers": {
        "onPress": {
            "type": "task",
            "taskManager": "MyTaskManager",
            "task": "refreshData"
        }
    }
}
```

### Package-Level Features

LGUI2 packages support advanced features at the root level:

**Package Dependencies:**
```json
{
    "includes": ["required-library.json"],
    "optionalIncludes": ["optional-feature.json"]
}
```

**Audio System:**
```json
{
    "audioVoices": {
        "ui": {"volume": [1.0, 1.0]}
    },
    "audioStreams": {
        "click": {"filename": "click.wav"}
    }
}
```

**Shader Support:**
```json
{
    "pixelShaders": { /* ps_2_0 through ps_5_1 */ },
    "vertexShaders": { /* vs_2_0 through vs_5_1 */ }
}
```

**MetaScript Integration:**
```json
{
    "metaScripts": [
        {
            "filename": "helper.iss",
            "autoStart": true
        }
    ]
}
```

### Why These Features Matter

**LGUI1 Limitation:** Manual updates everywhere

```lavishscript
; LGUI1 - Manual updates required
UIElement[healthText]:SetText[${Me.Health}]
```

**LGUI2 Solution:** Automatic updates via data binding

```json
"textBinding": {"pullFormat": "${Me.Health}"}
```

The UI automatically updates when the bound data changes!

---

*Last Updated: 2025-10-25 (Added Issue 12: MessageBox Command Migration to LGUI2)*
*Covers: LavishGUI 1 → LavishGUI 2 migration*
