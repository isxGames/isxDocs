# LavishGUI 2 UI Creation Guide

**System:** LavishGUI 2 (LGUI2)
**Format:** JSON-based package files
**Target:** InnerSpace / ISXEQ2 Scripts

---

## Table of Contents

1. [Introduction to LavishGUI 2](#introduction-to-lavishgui-2)
2. [Getting Started](#getting-started)
3. [Package Structure](#package-structure)
4. [Core UI Elements](#core-ui-elements)
5. [Layout Containers](#layout-containers)
6. [Event Handling](#event-handling)
7. [Data Binding](#data-binding)
8. [Styling and Visual Customization](#styling-and-visual-customization)
9. [Templates](#templates)
10. [Advanced Features](#advanced-features)
    - Canvas Drawing API
    - Audio System Integration
    - Keyboard Input Handling
    - Game Controller Pattern
    - Input Hooks
    - Event Hooks
    - Visibility Control
    - Triggers
    - FilePicker
    - Animations
    - Item View Generators
11. [Element Lifecycle and Events](#element-lifecycle-and-events)
12. [Script-to-UI Interaction](#script-to-ui-interaction)
13. [Complete Working Examples](#complete-working-examples)
14. [Best Practices](#best-practices)
15. [Troubleshooting](#troubleshooting)

---

## Introduction to LavishGUI 2

<!-- CLAUDE_SKIP_START -->
### What is LavishGUI 2?

**LavishGUI 2** (LGUI2) is the modern UI framework for InnerSpace, replacing the older XML-based LavishGUI 1 system. It provides a powerful, flexible way to create custom user interfaces for your scripts using **JSON package files** instead of XML.

### Key Advantages Over LavishGUI 1

| Feature | LavishGUI 1 | LavishGUI 2 |
|---------|-------------|-------------|
| **File Format** | XML | JSON |
| **Schema Validation** | None | Full JSON schema support |
| **Data Binding** | Manual updates | Automatic data binding |
| **Event Handlers** | XML attributes | Inline JSON event definitions |
| **Loading Method** | `ui -load` command | `LGUI2:LoadPackageFile` |
| **Element Access** | `UIElement[]` | `LGUI2.Element[]` |
| **Templates** | XML-based | JSON-based with jsonTemplate |
| **Audio Support** | Limited | Full audio integration |
| **Modern Tooling** | No IDE support | Full schema autocomplete |

### When to Use LavishGUI 2

- **All new scripts** should use LavishGUI 2
- Provides better maintainability with JSON
- Enables IDE autocomplete and validation via JSON schema
- More powerful data binding reduces manual UI updates
- Cleaner event handling with inline definitions
<!-- CLAUDE_SKIP_END -->

---

## Getting Started

### Basic Workflow

Creating a LavishGUI 2 interface involves:

1. **Create a JSON package file** (e.g., `myui.json`) with UI definition
2. **Create a LavishScript controller** (e.g., `myui.iss`) to manage the UI
3. **Load the package** using `LGUI2:LoadPackageFile`
4. **Interact with elements** using `LGUI2.Element[]`

### Your First LavishGUI 2 Interface

#### Step 1: Create the JSON Package (hello.json)

```json
{
    "$schema": "http://www.lavishsoft.com/schema/lgui2Package.json",
    "elements": [
        {
            "type": "window",
            "title": "My First LGUI2 Window",
            "name": "hello.window",
            "width": 300,
            "height": 100,
            "content": "Hello World!"
        }
    ]
}
```

#### Step 2: Create the Controller Script (hello.iss)

```lavishscript
objectdef hello_controller
{
    method Initialize()
    {
        LGUI2:LoadPackageFile[hello.json]
    }

    method Shutdown()
    {
        LGUI2:UnloadPackageFile[hello.json]
    }
}

variable(global) hello_controller HelloController

function main()
{
    while 1
        waitframe
}
```

#### Step 3: Run Your Script

```
run hello
```

A window will appear with "Hello World!" displayed. End the script with:

```
end hello
```

---

## Package Structure

### The JSON Schema

Every LavishGUI 2 package should start with the schema reference:

```json
{
    "$schema": "http://www.lavishsoft.com/schema/lgui2Package.json",
    ...
}
```

This enables:
- IDE autocomplete
- JSON validation
- Property suggestions
- Error detection

### Root-Level Properties

A complete package can include:

```json
{
    "$schema": "http://www.lavishsoft.com/schema/lgui2Package.json",
    "includes": ["required-package.json"],
    "optionalIncludes": ["optional-package.json"],
    "audioVoices": { /* Audio voice definitions */ },
    "audioStreams": { /* Audio stream definitions */ },
    "brushes": { /* Global brush definitions */ },
    "fonts": { /* Global font definitions */ },
    "pixelShaders": { /* Pixel shader definitions */ },
    "vertexShaders": { /* Vertex shader definitions */ },
    "skin": "skinName",
    "templates": { /* Reusable templates */ },
    "animationTypes": [ /* Custom animation types */ ],
    "metaScripts": [ /* MetaScript integration */ ],
    "elements": [ /* Root-level UI elements */ ]
}
```

**Package Dependencies:**

- **`includes`** - Array of required package files to load before this one (paths relative to current file)
- **`optionalIncludes`** - Array of optional package files loaded if available

**Audio System:**

- **`audioVoices`** - Define audio voices with volume settings (supports up to 32-channel audio)
- **`audioStreams`** - Define audio streams with file paths or URLs (supports any Windows Media Foundation format)

**Visual Resources:**

- **`brushes`** - Global brush definitions for reuse across elements
- **`fonts`** - Global font definitions for consistent typography
- **`skin`** - Apply a named skin or embed a custom skin definition
- **`templates`** - Reusable element templates

**Shaders:**

- **`pixelShaders`** - Pixel shader definitions (supports ps_5_1, ps_5_0, ps_4_1, ps_4_0, ps_3_0, ps_2_0)
- **`vertexShaders`** - Vertex shader definitions (supports vs_5_1, vs_5_0, vs_4_1, vs_4_0, vs_3_0, vs_2_0)

**Advanced Features:**

- **`animationTypes`** - Custom animation type definitions
- **`metaScripts`** - MetaScript objects with `filename`, `autoStart`, and `autoStartArgs`

### The `elements` Array

The `elements` array contains **root-level UI components**, typically windows:

```json
"elements": [
    {
        "type": "window",
        "title": "Window 1",
        "name": "window1",
        "content": "Content here"
    },
    {
        "type": "window",
        "title": "Window 2",
        "name": "window2",
        "content": "More content"
    }
]
```

### String Shorthand

**Strings are automatically converted to textblock elements:**

```json
"content": "Hello World!"
```

Is equivalent to:

```json
"content": {
    "type": "textblock",
    "text": "Hello World!"
}
```

This shorthand makes simple UIs very concise.

---

## Core UI Elements

### Window

Windows are the top-level containers for your UI.

**Basic Window:**

```json
{
    "type": "window",
    "name": "mywindow",
    "title": "My Window",
    "width": 400,
    "height": 300,
    "minSize": [200, 150],
    "content": {
        "type": "textblock",
        "text": "Window content goes here"
    }
}
```

**Window Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `type` | string | Must be "window" |
| `name` | string | Unique identifier for accessing the window |
| `title` | string | Window title bar text (use OR custom titleBar, not both) |
| `titleBar` | object | Custom title bar definition (for scalable UIs) |
| `width` | number | Window width in pixels |
| `height` | number | Window height in pixels |
| `minSize` | array | `[width, height]` minimum size |
| `content` | element/string | Window content (single element) |
| `backgroundBrush` | object | Background brush definition |

**Custom Title Bar for Scalable UIs:**

When using UI scaling (see [12_LGUI2_Scaling_System.md](12_LGUI2_Scaling_System.md)), the default title bar from the skin doesn't scale. To create a scalable title bar, define a custom `titleBar` and **remove the `title` property**:

```json
{
    "type": "window",
    "name": "mywindow",
    "width": 800,
    "height": 600,
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
                "text": "My Window Title",
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
    "content": {
        "type": "panel"
    }
}
```

**When to use custom titleBar:**
- ✅ When using UI scaling (LGUI2Scaling.iss)
- ✅ When you need large, accessible title bar buttons
- ✅ When you want full control over title bar appearance

**When to use simple title:**
- ✅ When not scaling the UI
- ✅ When default skin appearance is acceptable
- ✅ For quick prototypes or simple tools

See complete example: [EQ2BotCommander.json](https://github.com/isxGames/isxScripts/blob/master/EverQuest2/Scripts/EQ2Bot/UI/EQ2BotCommander.json) (lines 1-88)

### TextBlock

Displays text on the screen.

**Simple TextBlock:**

```json
{
    "type": "textblock",
    "text": "This is some text"
}
```

**Dynamic Text (with variables):**

```json
{
    "type": "textblock",
    "text": "FPS: ${Display.FPS.Centi}"
}
```

The text will automatically update every frame with the current FPS value!

**TextBlock with Data Binding:**

```json
{
    "type": "textblock",
    "textBinding": {
        "pullFormat": "Health: ${Me.Health}"
    }
}
```

**TextBlock Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `type` | string | "textblock" |
| `text` | string | Static or variable-embedded text |
| `textBinding` | object | Data binding for dynamic updates |
| `color` | string/array | Text color (hex or RGB array) |
| `font` | object | Font definition (family, size, bold, etc.) |
| `horizontalAlignment` | string | "left", "center", "right", "stretch" |
| `verticalAlignment` | string | "top", "center", "bottom", "stretch" |

### Button

Clickable buttons that trigger actions.

**Simple Button:**

```json
{
    "type": "button",
    "content": "Click Me!",
    "eventHandlers": {
        "onPress": {
            "type": "code",
            "code": "echo Button clicked!"
        }
    }
}
```

**Button Calling Controller Method:**

```json
{
    "type": "button",
    "content": "Attack",
    "eventHandlers": {
        "onPress": {
            "type": "method",
            "object": "MyController",
            "method": "OnAttackPressed"
        }
    }
}
```

**Button Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `type` | string | "button" |
| `content` | element/string | Button content (text or other elements) |
| `eventHandlers` | object | Event handler definitions |
| `verticalAlignment` | string | Vertical alignment |
| `horizontalAlignment` | string | Horizontal alignment |

### Checkbox

Toggleable checkbox control.

**Checkbox with Event Handlers:**

```json
{
    "type": "checkbox",
    "content": "Enable Auto-Attack",
    "eventHandlers": {
        "onChecked": {
            "type": "code",
            "code": "echo Checkbox checked!"
        },
        "onUnchecked": {
            "type": "code",
            "code": "echo Checkbox unchecked!"
        }
    }
}
```

**Checkbox with Data Binding:**

```json
{
    "type": "checkbox",
    "checkedBinding": "MyController.AutoAttackEnabled",
    "content": "Auto-Attack: ${MyController.AutoAttackEnabled}"
}
```

The checkbox state is **automatically synchronized** with `MyController.AutoAttackEnabled`.

**Persistent Checkbox State (Configuration Pattern):**

When checkboxes need to persist their state across sessions (like in configuration windows), use handler functions with `ExecuteQueued`:

```json
{
    "type": "checkbox",
    "name": "Options.AutoLoot",
    "content": "Enable Auto-Loot",
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

**Controller Implementation:**

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
    ; CRITICAL: Execute queued commands BEFORE saving
    ExecuteQueued

    ; Now save configuration
    call config_save

    LGUI2:UnloadPackageFile["ConfigUI.json"]
}

function config_save()
{
    Settings:AddSetting[AutoLootEnabled,${AutoLootEnabled}]
    LavishSettings[MyScript]:Export[${ConfigFile}]
}
```

**Why This Pattern Is Necessary:**

Event handlers using `QueueCommand` don't execute immediately - commands are queued and execute later. If you save configuration before calling `ExecuteQueued`, variables will still have their old values:

```lavishscript
// DON'T DO THIS - Variables won't update before save
function CloseConfigWindow()
{
    call config_save    ; BAD - Runs before queued commands execute
    ExecuteQueued       ; Too late!
}

// DO THIS - Execute queued commands first
function CloseConfigWindow()
{
    ExecuteQueued       ; Execute queued checkbox handlers
    call config_save    ; Now variables have correct values
}
```

See **11_LavishGUI1_to_LavishGUI2_Migration.md** Issue 11 for complete details and troubleshooting.

**Checkbox Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `type` | string | "checkbox" |
| `content` | element/string | Checkbox label |
| `checkedBinding` | string | Data binding for checked state |
| `checked` | string | Initial checked state (supports variable references) |
| `eventHandlers` | object | Event handlers (onChecked, onUnchecked) |

### ComboBox (Dropdown)

Drop-down selection control.

**ComboBox with Static Items:**

```json
{
    "type": "combobox",
    "name": "classSelector",
    "items": [
        {
            "type": "textblock",
            "text": "Warrior"
        },
        {
            "type": "textblock",
            "text": "Mage"
        },
        {
            "type": "textblock",
            "text": "Priest"
        }
    ],
    "eventHandlers": {
        "onSelectionChanged": {
            "type": "method",
            "object": "MyController",
            "method": "OnClassSelected"
        }
    }
}
```

**ComboBox with Data Binding:**

```json
{
    "type": "combobox",
    "name": "memberSelector",
    "itemsBinding": {
        "pullFormat": "${MyController.GetGroupMembers}"
    },
    "eventHandlers": {
        "onSelectionChanged": {
            "type": "method",
            "object": "MyController",
            "method": "OnMemberSelected"
        }
    }
}
```

In the controller:

```lavishscript
method OnSelectionChanged()
{
    echo Selected: Index=${Context.Source.SelectedItem.Index} Data=${Context.Source.SelectedItem.Data}
}

member:string GetGroupMembers()
{
    return "[{\"type\":\"textblock\",\"text\":\"Tank\"},{\"type\":\"textblock\",\"text\":\"Healer\"}]"
}
```

**ComboBox Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `type` | string | "combobox" |
| `name` | string | Element identifier |
| `items` | array | Static item list |
| `itemsBinding` | object | Data binding for dynamic items |
| `eventHandlers` | object | Event handlers (onSelectionChanged) |
| `horizontalAlignment` | string | Horizontal alignment |

### InputPicker

Input field with optional dropdown suggestions, combining text entry with selection capabilities.

**Basic InputPicker (Combo Mode):**

```json
{
    "type": "inputpicker",
    "name": "hotkeyInput",
    "comboMode": true,
    "multipleControlMode": false,
    "minSize": [200, 0],
    "valueBinding": {
        "pullFormat": "${Settings.Get[hotkey]}",
        "pullReplaceNull": "",
        "pushFormat": ["Settings:Set[hotkey,\"", "\"]"]
    }
}
```

**InputPicker with Context Binding:**

```json
{
    "type": "inputpicker",
    "comboMode": true,
    "multipleControlMode": false,
    "minSize": [200, 0],
    "valueBinding": {
        "pullFormat": "${BasicCore.Settings.Hotkeys.Get[slotHotkeys,${_CONTEXTITEM_.Index}]}",
        "pullReplaceNull": "",
        "pushFormat": [
            "BasicCore.Settings.Hotkeys.Get[slotHotkeys]:SetString[${_CONTEXTITEM_.Index},\"",
            "\"]"
        ]
    }
}
```

**InputPicker Properties:**

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `type` | string | Yes | "inputpicker" |
| `name` | string | No | Element identifier |
| `comboMode` | boolean | No | If true, shows dropdown for suggestions |
| `multipleControlMode` | boolean | No | If true, allows multiple selections |
| `minSize` | array | No | `[width, height]` minimum dimensions |
| `valueBinding` | object | No | Bidirectional value binding |
| `items` | array | No | Static list of suggestion items |
| `itemsBinding` | object | No | Dynamic suggestion items via binding |

**Key Differences from ComboBox:**

- **InputPicker** allows free-form text entry AND dropdown selection
- **ComboBox** restricts input to dropdown items only
- InputPicker is ideal for hotkey assignment, custom values with suggestions
- ComboBox is ideal for restricted choice scenarios

**Use Cases:**

- **Hotkey assignment** - User can type or pick from common hotkeys
- **Search with suggestions** - Free text with common search hints
- **Custom values** - Allow custom input with helpful presets
- **Configuration fields** - Editable values with template options

### ImageBox

Displays images.

**Simple ImageBox:**

```json
{
    "type": "imagebox",
    "imageBrush": {
        "color": "#ffffff",
        "imageFile": "banner.png"
    }
}
```

**ImageBox with Image Orientation:**

```json
{
    "type": "imagebox",
    "margin": [1, 1, 1, 1],
    "imageBrush": {
        "color": [1.0, 1.0, 1.0],
        "imageFile": "../Assets/Images/logo.png",
        "imageOrientation": "mirrorVertical"
    }
}
```

**ImageBox Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `type` | string | "imagebox" |
| `imageBrush` | object | Image brush definition |
| `margin` | array | `[left, top, right, bottom]` margins |

**imageBrush Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `color` | string/array | Image tint color (hex or RGB) |
| `imageFile` | string | Path to image file (relative to JSON) |
| `imageOrientation` | string | "normal", "mirrorHorizontal", "mirrorVertical" |

### ListBox

Scrollable list of items.

**ListBox with Data Binding:**

```json
{
    "type": "listbox",
    "name": "targetList",
    "height": 200,
    "horizontalAlignment": "stretch",
    "itemsBinding": {
        "pullFormat": "${MyController.GetTargetList}"
    }
}
```

In the controller:

```lavishscript
member:string GetTargetList()
{
    return "$$>[
        {
            \"type\":\"textblock\",
            \"text\":\"Orc Warrior\",
            \"color\":\"#ff0000\"
        },
        {
            \"type\":\"textblock\",
            \"text\":\"Goblin Mage\",
            \"color\":\"#00ff00\"
        }
    ]<$$"
}
```

**ListBox Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `type` | string | "listbox" |
| `name` | string | Element identifier |
| `height` | number | List height in pixels |
| `itemsBinding` | object | Data binding for items |
| `horizontalAlignment` | string | Horizontal alignment |
| `verticalAlignment` | string | Vertical alignment |

### ScrollViewer

Container that provides scrolling functionality for content that exceeds its bounds.

**Basic ScrollViewer:**

```json
{
    "type": "scrollviewer",
    "width": 400,
    "height": 300,
    "horizontalScrollBarVisibility": "auto",
    "verticalScrollBarVisibility": "auto",
    "content": {
        "type": "stackpanel",
        "children": [
            {
                "type": "textblock",
                "text": "Content line 1"
            },
            {
                "type": "textblock",
                "text": "Content line 2"
            }
        ]
    }
}
```

**ScrollViewer Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `type` | string | "scrollviewer" |
| `horizontalScrollBarVisibility` | string | "auto", "visible", "hidden", "disabled" |
| `verticalScrollBarVisibility` | string | "auto", "visible", "hidden", "disabled" |
| `content` | object | Content element to scroll |

**Use Cases:**
- Long lists or text content
- Configuration panels with many settings
- Log viewers
- Any content that may exceed viewport size

### ProgressBar

Visual indicator showing progress or completion percentage.

**Basic ProgressBar:**

```json
{
    "type": "progressbar",
    "name": "healthBar",
    "width": 200,
    "height": 20,
    "valueBinding": {
        "pullFormat": "${Me.Health}"
    },
    "minValue": 0,
    "maxValue": 100,
    "backgroundBrush": {
        "color": "#333333"
    },
    "foregroundBrush": {
        "color": "#00ff00"
    }
}
```

**ProgressBar with Text Overlay:**

```json
{
    "type": "progressbar",
    "value": 75,
    "minValue": 0,
    "maxValue": 100,
    "content": {
        "type": "textblock",
        "text": "75%",
        "horizontalAlignment": "center",
        "verticalAlignment": "center"
    }
}
```

**ProgressBar Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `type` | string | "progressbar" |
| `value` | number | Current progress value |
| `minValue` | number | Minimum value (default: 0) |
| `maxValue` | number | Maximum value (default: 100) |
| `valueBinding` | object | Binding for dynamic updates |
| `backgroundBrush` | object | Bar background |
| `foregroundBrush` | object | Fill color |
| `content` | object | Optional overlay (e.g., text) |

### Slider

Interactive control for selecting numeric values within a range.

**Horizontal Slider:**

```json
{
    "type": "slider",
    "name": "volumeSlider",
    "width": 200,
    "height": 20,
    "minValue": 0,
    "maxValue": 100,
    "value": 50,
    "orientation": "horizontal",
    "valueBinding": {
        "pullFormat": "${Settings.Volume}",
        "pushFormat": ["Settings:Set[Volume,", "]"]
    },
    "eventHandlers": {
        "onValueChanged": {
            "type": "method",
            "object": "MyController",
            "method": "OnVolumeChanged"
        }
    }
}
```

**Vertical Slider:**

```json
{
    "type": "slider",
    "orientation": "vertical",
    "height": 150,
    "width": 30,
    "minValue": 0,
    "maxValue": 10,
    "value": 5
}
```

**Slider Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `type` | string | "slider" |
| `orientation` | string | "horizontal" (default) or "vertical" |
| `minValue` | number | Minimum slider value |
| `maxValue` | number | Maximum slider value |
| `value` | number | Current value |
| `valueBinding` | object | Bidirectional value binding |
| `tickFrequency` | number | Spacing between tick marks |
| `isSnapToTickEnabled` | boolean | Snap to tick marks |

**Use Cases:**
- Volume controls
- Opacity adjustments
- Numeric settings (zoom level, speed, etc.)
- Any continuous value selection

### ScrollBar

Scrollbar control for manual scrolling (typically used internally by ScrollViewer).

**Basic ScrollBar:**

```json
{
    "type": "scrollbar",
    "orientation": "vertical",
    "height": 300,
    "minValue": 0,
    "maxValue": 1000,
    "viewportSize": 100
}
```

**ScrollBar Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `type` | string | "scrollbar" |
| `orientation` | string | "horizontal" or "vertical" |
| `minValue` | number | Minimum scroll value |
| `maxValue` | number | Maximum scroll value |
| `value` | number | Current scroll position |
| `viewportSize` | number | Visible area size |

---

## Box Model and Composition

### The LGUI2 Box Model

LavishGUI 2 uses a box model system that differs from CSS in important ways:

**Box Model Structure:**

```
+------------------------------------------+
|            Margin (outside)              |
|  +------------------------------------+  |
|  |      Border                        |  |
|  |  +------------------------------+  |  |
|  |  |   Padding (inside)           |  |  |
|  |  |  +------------------------+  |  |  |
|  |  |  |  Content Area          |  |  |  |
|  |  |  |                        |  |  |  |
|  |  |  +------------------------+  |  |  |
|  |  +------------------------------+  |  |
|  +------------------------------------+  |
+------------------------------------------+
```

**Critical Difference from CSS:**

> **Margins are OUTSIDE an element's bounds, while Padding is INSIDE an element's bounds**

This means:
- **Margin**: Creates space around the element (outside its borders)
- **Padding**: Creates space inside the element (reduces content area)

**Padding Reduces Content Space:**

An element with `width: 100` and `padding: 10` (left + right = 20px total):
- Total element width: 100px
- Content area: **80px** (100 - 20)
- The padding "eats into" the available content space

**Example:**

```json
{
    "type": "panel",
    "width": 100,
    "height": 50,
    "padding": 5,
    "children": [
        {
            "type": "textblock",
            "text": "Content only has 90x40 px available"
        }
    ]
}
```

### Element Sizing

**Automatic Sizing:**

Elements default to their content dimensions when width/height aren't specified.

**Explicit Sizing:**

```json
{
    "width": 200,
    "height": 100
}
```

**Factor-Based Sizing:**

Use proportional sizing relative to parent:

```json
{
    "widthFactor": 0.5,    // 50% of parent width
    "heightFactor": 0.75   // 75% of parent height
}
```

**Combined Sizing with Offset:**

When both a dimension and factor are specified, they're added together:

```json
{
    "x": 10,
    "xFactor": 0.25
}
```

Result: `finalX = 10 + (0.25 × parentWidth)`

This makes the factor component a **base position** and the dimension an **offset**.

**Size Constraints:**

```json
{
    "minWidth": 100,
    "maxWidth": 500,
    "minHeight": 50,
    "maxHeight": 300
}
```

### Alignment

Control element positioning within containers:

**Horizontal Alignment:**

```json
"horizontalAlignment": "left"      // Align to left
"horizontalAlignment": "center"    // Center horizontally
"horizontalAlignment": "right"     // Align to right
"horizontalAlignment": "stretch"   // Fill width
```

**Vertical Alignment:**

```json
"verticalAlignment": "top"        // Align to top
"verticalAlignment": "center"     // Center vertically
"verticalAlignment": "bottom"     // Align to bottom
"verticalAlignment": "stretch"    // Fill height
```

**Example:**

```json
{
    "type": "panel",
    "width": 200,
    "height": 200,
    "children": [
        {
            "type": "button",
            "width": 100,
            "height": 40,
            "content": "Centered Button",
            "horizontalAlignment": "center",
            "verticalAlignment": "center"
        }
    ]
}
```

### Metadata System

**Underscore Properties:**

All properties beginning with an underscore (`_`) are placed in the element's **metadata table** with the underscore removed.

**Purpose:** Metadata provides a way to store custom data or configuration without conflicting with standard element properties.

**Common Metadata Usage - Docking:**

The `_dock` metadata property is used by `dockpanel` containers:

```json
{
    "type": "dockpanel",
    "children": [
        {
            "type": "button",
            "_dock": "top",
            "content": "Top Button"
        },
        {
            "type": "button",
            "_dock": "left",
            "content": "Left Button"
        },
        {
            "type": "button",
            "_dock": "right",
            "content": "Right Button"
        },
        {
            "type": "panel",
            "content": "Center (fills remaining space)"
        }
    ]
}
```

**Custom Metadata:**

Use metadata to store custom configuration:

```json
{
    "type": "button",
    "name": "actionButton",
    "content": "Click Me",
    "_actionType": "combat",
    "_priority": 10,
    "_category": "offensive"
}
```

Access metadata from script:

```lavishscript
variable string actionType
actionType:Set[${LGUI2.Element[actionButton].Metadata.Get[actionType]}]
echo Action Type: ${actionType}
```

**Metadata Best Practices:**

- Use metadata for custom data that doesn't fit standard properties
- Prefix custom metadata with your script name to avoid conflicts: `_myScript_customData`
- Document metadata properties for maintainability

## Layout Containers

Layout containers organize child elements.

### Panel

Simple container with absolute or aligned positioning.

**Basic Panel:**

```json
{
    "type": "panel",
    "width": 100,
    "height": 100,
    "children": [
        {
            "type": "textblock",
            "text": "Centered Text",
            "verticalAlignment": "center",
            "horizontalAlignment": "center"
        }
    ]
}
```

**Panel with Border:**

```json
{
    "type": "panel",
    "width": 200,
    "height": 150,
    "margin": 5,
    "borderThickness": 2,
    "borderBrush": {
        "color": "#ffff00"
    },
    "children": [
        {
            "type": "textblock",
            "text": "Panel with yellow border"
        }
    ]
}
```

### StackPanel

Stacks children vertically or horizontally.

**Vertical StackPanel:**

```json
{
    "type": "stackpanel",
    "orientation": "vertical",
    "children": [
        "Item 1",
        "Item 2",
        "Item 3"
    ]
}
```

**Horizontal StackPanel:**

```json
{
    "type": "stackpanel",
    "orientation": "horizontal",
    "horizontalContentAlignment": "left",
    "children": [
        {
            "type": "button",
            "content": "Button 1"
        },
        {
            "type": "button",
            "content": "Button 2"
        }
    ]
}
```

### DockPanel

Docks children to edges (left, right, top, bottom) with last child filling remaining space.

**DockPanel Example:**

```json
{
    "type": "dockpanel",
    "horizontalAlignment": "stretch",
    "verticalAlignment": "stretch",
    "children": [
        {
            "_dock": "top",
            "type": "textblock",
            "text": "Header at top"
        },
        {
            "_dock": "left",
            "type": "textblock",
            "text": "Sidebar",
            "width": 100
        },
        {
            "type": "textblock",
            "text": "Main content fills remaining space"
        }
    ]
}
```

### Table

Grid layout with rows and columns.

**Table with 3x3 Grid:**

```json
{
    "type": "table",
    "rows": [
        {"heightFactor": 0.333},
        {"heightFactor": 0.333},
        {"heightFactor": 0.334}
    ],
    "columns": [
        {"widthFactor": 0.333},
        {"widthFactor": 0.333},
        {"widthFactor": 0.334}
    ],
    "children": [
        {
            "type": "button",
            "_row": 1,
            "_column": 1,
            "content": "Row 1, Col 1",
            "verticalAlignment": "stretch",
            "horizontalAlignment": "stretch"
        },
        {
            "type": "button",
            "_row": 1,
            "_column": 2,
            "content": "Row 1, Col 2",
            "verticalAlignment": "stretch",
            "horizontalAlignment": "stretch"
        }
    ]
}
```

**Table Properties:**

- `rows`: Array of row definitions with `heightFactor` (relative height)
- `columns`: Array of column definitions with `widthFactor` (relative width)
- `_row` and `_column` on children specify cell position (1-indexed)

### TabControl

**CRITICAL:** LGUI2 has a **native tabcontrol** element type that handles tab switching automatically.

**Native TabControl Example:**

```json
{
    "type": "tabcontrol",
    "name": "myTabControl",
    "tabs": [
        {
            "type": "tab",
            "header": "Main",
            "content": {
                "type": "panel",
                "children": [
                    {
                        "type": "button",
                        "content": "Button on Main tab"
                    }
                ]
            }
        },
        {
            "type": "tab",
            "header": "Settings",
            "content": {
                "type": "panel",
                "children": [
                    {
                        "type": "checkbox",
                        "content": "Enable feature"
                    }
                ]
            }
        }
    ]
}
```

**TabControl Properties:**

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `type` | string | **YES** | Must be "tabcontrol" |
| `name` | string | No | Element identifier |
| `tabs` | array | **YES** | Array of tab definitions |

**Tab Properties:**

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `type` | string | **YES** | Must be "tab" |
| `header` | string | **YES** | Tab label displayed to user |
| `content` | element | **YES** | Content shown when tab is active |

**Key Points:**

1. **Each tab is a separate object** with `type: "tab"`
2. **`header` property** is the visible tab label
3. **`content` property** contains the tab's UI (typically a panel with children)
4. **Tab switching is automatic** - no manual visibility code needed
5. **Each tab's content is independent** - they don't share children

**Complete TabControl Example:**

```json
{
    "type": "window",
    "title": "Settings Window",
    "name": "settings.window",
    "width": 400,
    "height": 300,
    "content": {
        "type": "tabcontrol",
        "name": "settings.tabcontrol",
        "tabs": [
            {
                "type": "tab",
                "header": "Combat",
                "content": {
                    "type": "stackpanel",
                    "orientation": "vertical",
                    "children": [
                        {
                            "type": "checkbox",
                            "content": "Auto-Attack",
                            "checkedBinding": "Settings.AutoAttack"
                        },
                        {
                            "type": "checkbox",
                            "content": "Auto-Loot",
                            "checkedBinding": "Settings.AutoLoot"
                        }
                    ]
                }
            },
            {
                "type": "tab",
                "header": "Display",
                "content": {
                    "type": "stackpanel",
                    "orientation": "vertical",
                    "children": [
                        {
                            "type": "checkbox",
                            "content": "Show FPS",
                            "checkedBinding": "Settings.ShowFPS"
                        },
                        {
                            "type": "checkbox",
                            "content": "Show Minimap",
                            "checkedBinding": "Settings.ShowMinimap"
                        }
                    ]
                }
            },
            {
                "type": "tab",
                "header": "About",
                "content": {
                    "type": "textblock",
                    "text": "Version 1.0.0"
                }
            }
        ]
    }
}
```

**Important Notes:**

- **DO NOT** manually toggle panel visibility for tabs - the tabcontrol handles this automatically
- **DO NOT** try to use buttons with SetVisibility to simulate tabs - use the native control
- Each tab's content is only rendered when that tab is active (performance benefit)
- Tab selection state is managed automatically by the tabcontrol

---

### Dragger

**Dragger** elements create resizable dividers between panels, allowing users to adjust layout proportions at runtime.

**Basic Dragger Example:**

```json
{
    "type": "dockpanel",
    "children": [
        {
            "_dock": "left",
            "type": "panel",
            "width": 200,
            "children": [ /* Left panel content */ ]
        },
        {
            "_dock": "left",
            "type": "dragger",
            "width": 5,
            "allowResize": true,
            "backgroundBrush": {
                "color": "#333333"
            }
        },
        {
            "type": "panel",
            "children": [ /* Right panel content fills remaining space */ ]
        }
    ]
}
```

**Dragger Properties:**

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `type` | string | **YES** | Must be "dragger" |
| `allowResize` | bool | No | Enable drag-to-resize (default: false) |
| `width` | number | No | Width in pixels (for vertical dividers) |
| `height` | number | No | Height in pixels (for horizontal dividers) |
| `backgroundBrush` | object | No | Visual styling for the divider |

**Horizontal Divider Example:**

```json
{
    "type": "dockpanel",
    "children": [
        {
            "_dock": "top",
            "type": "panel",
            "height": 150,
            "children": [ /* Top panel */ ]
        },
        {
            "_dock": "top",
            "type": "dragger",
            "height": 3,
            "allowResize": true,
            "backgroundBrush": {
                "color": "#555555"
            }
        },
        {
            "type": "panel",
            "children": [ /* Bottom panel fills remaining */ ]
        }
    ]
}
```

**Key Points:**

1. **Position matters** - Place dragger between the panels you want to resize
2. **Use with dockpanel** - Draggers work best in dockpanel layouts
3. **Set `allowResize: true`** - Required for interactive resizing
4. **Small width/height** - Typically 3-5 pixels for visual clarity
5. **Style with backgroundBrush** - Make the dragger visible to users

**Complete Resizable Layout Example:**

```json
{
    "type": "window",
    "title": "Resizable Panels",
    "width": 800,
    "height": 600,
    "content": {
        "type": "dockpanel",
        "children": [
            {
                "_dock": "left",
                "type": "panel",
                "name": "sidebar",
                "width": 200,
                "backgroundBrush": {
                    "color": "#1e1e1e"
                },
                "children": [
                    {
                        "type": "textblock",
                        "text": "Sidebar",
                        "horizontalAlignment": "center"
                    }
                ]
            },
            {
                "_dock": "left",
                "type": "dragger",
                "width": 5,
                "allowResize": true,
                "backgroundBrush": {
                    "color": "#444444"
                }
            },
            {
                "type": "panel",
                "name": "mainContent",
                "backgroundBrush": {
                    "color": "#2d2d2d"
                },
                "children": [
                    {
                        "type": "textblock",
                        "text": "Main Content Area",
                        "horizontalAlignment": "center"
                    }
                ]
            }
        ]
    }
}
```

**Use Cases:**

- **Split-pane editors** - Sidebar + main content area
- **Multi-panel dashboards** - Adjustable widget sizes
- **Property inspector layouts** - Resize property panel vs preview
- **Chat + game view** - User-adjustable chat width

### Expander

Collapsible section that can show/hide content.

**Basic Expander:**

```json
{
    "type": "expander",
    "header": "Advanced Options",
    "isExpanded": false,
    "content": {
        "type": "stackpanel",
        "orientation": "vertical",
        "children": [
            {
                "type": "checkbox",
                "content": "Enable Debug Mode"
            },
            {
                "type": "checkbox",
                "content": "Show Console"
            }
        ]
    }
}
```

**Expander with Custom Header:**

```json
{
    "type": "expander",
    "isExpanded": true,
    "header": {
        "type": "stackpanel",
        "orientation": "horizontal",
        "children": [
            {
                "type": "imagebox",
                "width": 16,
                "height": 16,
                "imageBrush": {
                    "imageFile": "settings.png"
                }
            },
            {
                "type": "textblock",
                "text": "Settings",
                "margin": [5, 0, 0, 0]
            }
        ]
    },
    "content": {
        "type": "panel",
        "children": []
    }
}
```

**Expander Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `type` | string | "expander" |
| `header` | string/object | Header text or element |
| `isExpanded` | boolean | Initially expanded state (default: false) |
| `content` | object | Content to show/hide |

**Use Cases:**
- Collapsible settings sections
- Advanced options panels
- Accordion-style interfaces
- Space-saving layouts

### Popup

Floating window that appears on demand, useful for tooltips and menus.

**Basic Popup:**

```json
{
    "type": "popup",
    "name": "myPopup",
    "isOpen": false,
    "placementTarget": "triggerButton",
    "placement": "bottom",
    "content": {
        "type": "panel",
        "backgroundBrush": {
            "color": "#2d2d2d"
        },
        "borderBrush": {
            "color": "#444444"
        },
        "borderThickness": 1,
        "padding": 10,
        "children": [
            {
                "type": "textblock",
                "text": "Popup content here"
            }
        ]
    }
}
```

**Popup with Button Trigger:**

```json
{
    "type": "stackpanel",
    "children": [
        {
            "type": "button",
            "name": "menuButton",
            "content": "Show Menu",
            "eventHandlers": {
                "onPress": {
                    "type": "code",
                    "code": "LGUI2.Element[menuPopup]:Set[isOpen,TRUE]"
                }
            }
        },
        {
            "type": "popup",
            "name": "menuPopup",
            "placementTarget": "menuButton",
            "placement": "bottom",
            "content": {
                "type": "stackpanel",
                "children": []
            }
        }
    ]
}
```

**Popup Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `type` | string | "popup" |
| `isOpen` | boolean | Visibility state |
| `placementTarget` | string | Element name to position relative to |
| `placement` | string | "top", "bottom", "left", "right", "center" |
| `content` | object | Popup content |
| `staysOpen` | boolean | Remains open when clicking outside (default: true) |

### ContextMenu

Right-click menu for elements.

**Basic ContextMenu:**

```json
{
    "type": "panel",
    "name": "targetPanel",
    "width": 200,
    "height": 200,
    "backgroundBrush": {
        "color": "#2d2d2d"
    },
    "contextMenu": {
        "type": "contextmenu",
        "children": [
            {
                "type": "button",
                "content": "Copy",
                "eventHandlers": {
                    "onPress": {
                        "type": "method",
                        "object": "MyController",
                        "method": "OnCopy"
                    }
                }
            },
            {
                "type": "button",
                "content": "Paste",
                "eventHandlers": {
                    "onPress": {
                        "type": "method",
                        "object": "MyController",
                        "method": "OnPaste"
                    }
                }
            },
            {
                "type": "button",
                "content": "Delete",
                "eventHandlers": {
                    "onPress": {
                        "type": "method",
                        "object": "MyController",
                        "method": "OnDelete"
                    }
                }
            }
        ]
    }
}
```

**ContextMenu Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `type` | string | "contextmenu" |
| `children` | array | Menu item elements (typically buttons) |

**Pattern:** Attach to parent element via `contextMenu` property. Right-clicking the parent element opens the menu.

### CommandBox

Advanced text input with command history and auto-completion.

**Basic CommandBox:**

```json
{
    "type": "commandbox",
    "name": "consoleInput",
    "width": 400,
    "placeholder": "Enter command...",
    "eventHandlers": {
        "onCommand": {
            "type": "method",
            "object": "ConsoleController",
            "method": "OnCommand"
        }
    }
}
```

**CommandBox with History:**

```lavishscript
method OnCommand()
{
    variable string command = "${Context.Args[1]~}"
    echo Executing: ${command}

    ; Execute the command
    execute "${command}"

    ; Clear the input
    LGUI2.Element[consoleInput]:Set[text,""]
}
```

**CommandBox Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `type` | string | "commandbox" |
| `placeholder` | string | Placeholder text when empty |
| `text` | string | Current text value |
| `eventHandlers.onCommand` | object | Fired when Enter is pressed |

**Use Cases:**
- Console commands
- Chat input
- Command-line interfaces
- Script execution input

### RadialGauge

Circular gauge for displaying values (speedometer-style).

**Basic RadialGauge:**

```json
{
    "type": "radialgauge",
    "name": "speedGauge",
    "width": 150,
    "height": 150,
    "minValue": 0,
    "maxValue": 100,
    "value": 65,
    "valueBinding": {
        "pullFormat": "${Me.Speed}"
    },
    "foregroundBrush": {
        "color": "#00ff00"
    },
    "backgroundBrush": {
        "color": "#333333"
    }
}
```

**RadialGauge Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `type` | string | "radialgauge" |
| `minValue` | number | Minimum gauge value |
| `maxValue` | number | Maximum gauge value |
| `value` | number | Current value |
| `valueBinding` | object | Binding for dynamic updates |
| `startAngle` | number | Start angle in degrees (default: -135) |
| `endAngle` | number | End angle in degrees (default: 135) |

**Use Cases:**
- Speed indicators
- Progress indicators (circular)
- Health/mana displays (circular)
- Performance metrics

### Knob

Rotary control for adjusting values by dragging.

**Basic Knob:**

```json
{
    "type": "knob",
    "name": "volumeKnob",
    "width": 60,
    "height": 60,
    "minValue": 0,
    "maxValue": 100,
    "value": 50,
    "valueBinding": {
        "pullFormat": "${Settings.Volume}",
        "pushFormat": ["Settings:Set[Volume,", "]"]
    },
    "eventHandlers": {
        "onValueChanged": {
            "type": "method",
            "object": "AudioController",
            "method": "OnVolumeChanged"
        }
    }
}
```

**Knob Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `type` | string | "knob" |
| `minValue` | number | Minimum knob value |
| `maxValue` | number | Maximum knob value |
| `value` | number | Current value |
| `valueBinding` | object | Bidirectional value binding |
| `sensitivity` | number | Drag sensitivity (higher = less sensitive) |

**Use Cases:**
- Volume controls
- Pan/balance controls
- Fine-tuning numeric values
- Audio mixing interfaces

---

## Event Handling

### Event Handler Types

LavishGUI 2 supports several event handler types:

| Type | Description | Use Case |
|------|-------------|----------|
| `code` | Inline LavishScript code | Simple actions |
| `method` | Call controller method | Complex logic |
| `audio` | Play audio | Sound effects |
| `style` | Apply style | Visual state changes |
| `forward` | Forward event to another element | Event delegation |
| `animation` | Trigger animation | Visual transitions |
| `task` | Execute task manager task | Background operations |

### Code Event Handlers

Execute inline LavishScript code.

```json
"eventHandlers": {
    "onPress": {
        "type": "code",
        "code": "echo Button pressed!"
    }
}
```

Multiple statements:

```json
"eventHandlers": {
    "onPress": {
        "type": "code",
        "code": "echo Starting attack; Me:Target[${Actor[nearest npc]}]; Me.Ability[1]:Use"
    }
}
```

### Method Event Handlers

Call a method on your controller object.

**JSON:**

```json
"eventHandlers": {
    "onPress": {
        "type": "method",
        "object": "CombatController",
        "method": "OnAttackButtonPressed"
    }
}
```

**LavishScript Controller:**

```lavishscript
objectdef combat_controller
{
    method OnAttackButtonPressed()
    {
        echo Attack button was pressed!
        ; Access event context
        echo Event type: ${Context(type)}
        echo Source element: ${Context.Source}
    }
}

variable(global) combat_controller CombatController
```

### Audio Event Handlers

Play audio when event fires.

**JSON Package:**

```json
{
    "$schema": "http://www.lavishsoft.com/schema/lgui2Package.json",
    "audioVoices": {
        "ui sound": {
            "volume": [1.0, 1.0]
        }
    },
    "audioStreams": {
        "click": {
            "filename": "sounds/click.wav"
        }
    },
    "elements": [
        {
            "type": "window",
            "title": "Audio Example",
            "name": "audio.window",
            "content": {
                "type": "button",
                "content": "Play Sound",
                "eventHandlers": {
                    "onPress": {
                        "type": "audio",
                        "streamName": "click",
                        "voiceName": "ui sound"
                    }
                }
            }
        }
    ]
}
```

**Audio Event Handler Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `type` | string | "audio" |
| `voiceName` | string | Name from `audioVoices` |
| `streamName` | string | Name from `audioStreams` (omit to stop) |
| `flush` | boolean | Stop voice and clear queue (default: true) |

### Style Event Handlers

Apply a style when event fires.

```json
{
    "type": "imagebox",
    "imageBrush": {
        "color": "#ffffff",
        "imageFile": "icon.png"
    },
    "styles": {
        "mouseover gained": {
            "imageBrush": {
                "color": "#ff0000",
                "imageFile": "icon.png"
            }
        },
        "mouseover lost": {
            "imageBrush": {
                "color": "#ffffff",
                "imageFile": "icon.png"
            }
        }
    },
    "eventHandlers": {
        "gotMouseOver": {
            "type": "style",
            "styleName": "mouseover gained"
        },
        "lostMouseOver": {
            "type": "style",
            "styleName": "mouseover lost"
        }
    }
}
```

### Forward Event Handlers

Forward an event to another element, useful for delegating event handling.

```json
{
    "type": "button",
    "content": "Forward Click",
    "eventHandlers": {
        "onPress": {
            "type": "forward",
            "elementName": "targetButton",
            "event": "onPress"
        }
    }
}
```

**Forward Handler Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `type` | string | "forward" |
| `elementName` | string | Target element name |
| `event` | string | Event to fire on target |

### Animation Event Handlers

Trigger an animation when an event fires.

```json
{
    "type": "panel",
    "name": "animatedPanel",
    "eventHandlers": {
        "gotMouseOver": {
            "type": "animation",
            "elementName": "animatedPanel",
            "animation": "fadeIn"
        },
        "lostMouseOver": {
            "type": "animation",
            "elementName": "animatedPanel",
            "animation": "fadeOut"
        }
    }
}
```

**Animation Handler Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `type` | string | "animation" |
| `elementName` | string | Element to animate (optional, defaults to current element) |
| `animation` | string | Name of animation to trigger |

### Task Event Handlers

Execute a task manager task when event fires.

```json
{
    "type": "button",
    "content": "Run Task",
    "eventHandlers": {
        "onPress": {
            "type": "task",
            "taskManager": "MyTaskManager",
            "task": "refreshData",
            "stacking": false
        }
    }
}
```

**Task Handler Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `type` | string | "task" |
| `taskManager` | string | Task manager object name |
| `task` | string | Task name to execute |
| `stacking` | boolean | Allow task stacking (default: false) |

### Common Events

| Event | Fired When |
|-------|------------|
| `onPress` | Button or checkbox pressed |
| `onChecked` | Checkbox checked |
| `onUnchecked` | Checkbox unchecked |
| `onSelectionChanged` | ComboBox or ListBox selection changes |
| `gotMouseOver` | Mouse enters element |
| `lostMouseOver` | Mouse leaves element |
| `onLoad` | Element is loaded |
| `onUnload` | Element is unloaded |
| `onFinalized` | FilePicker finalized |

### The Context Object

Inside event handlers, `${Context}` provides event information:

```lavishscript
method OnEvent()
{
    echo Event type: ${Context(type)}
    echo Source element: ${Context.Source}
    echo Arguments: ${Context.Args}
}
```

**For ComboBox onSelectionChanged:**

```lavishscript
method OnSelectionChanged()
{
    echo Selected Index: ${Context.Source.SelectedItem.Index}
    echo Selected Data: ${Context.Source.SelectedItem.Data}
}
```

---

## Data Binding

Data binding automatically synchronizes UI with script data, eliminating manual updates.

### Pull Binding (Read-Only)

The UI **pulls** data from script.

**textBinding with pullFormat:**

```json
{
    "type": "textblock",
    "textBinding": {
        "pullFormat": "FPS: ${Display.FPS.Centi}"
    }
}
```

This textblock **automatically updates** every frame with the current FPS.

**itemsBinding for Lists:**

```json
{
    "type": "listbox",
    "itemsBinding": {
        "pullFormat": "${MyController.GetPlayerList}"
    }
}
```

In the controller:

```lavishscript
member:string GetPlayerList()
{
    ; Return JSON array of elements
    return "$$>[
        {\"type\":\"textblock\",\"text\":\"Player 1\"},
        {\"type\":\"textblock\",\"text\":\"Player 2\"}
    ]<$$"
}
```

### Two-Way Binding (Read-Write)

The UI can both pull and push data.

**Checkbox with Bidirectional Binding:**

**JSON:**

```json
{
    "type": "checkbox",
    "checkedBinding": "MyController.AutoLootEnabled",
    "content": "Auto-Loot: ${MyController.AutoLootEnabled}"
}
```

**Controller:**

```lavishscript
objectdef mycontroller
{
    variable bool AutoLootEnabled = TRUE

    method Initialize()
    {
        LGUI2:LoadPackageFile[myui.json]
    }
}
```

When the user clicks the checkbox, `MyController.AutoLootEnabled` is automatically updated!

### pullFormat vs Direct Variable Reference

**For text content, use pullFormat:**

```json
"textBinding": {
    "pullFormat": "Health: ${Me.Health}"
}
```

**For boolean checkbox, use direct reference:**

```json
"checkedBinding": "MyController.SettingEnabled"
```

### Pull Once

Use `pullOnce: true` to bind data only once at initialization:

```json
"itemsBinding": {
    "pullFormat": "${MyController.GetInitialData}",
    "pullOnce": true
}
```

### Advanced Data Binding with ObjectView

ObjectView provides sophisticated property editing with bidirectional binding:

```json
{
    "type": "objectview",
    "objectBinding": {
        "pullFormat": "${LGUI2.Element[mylist].SelectedItem}"
    },
    "properties": [
        {
            "name": "First Name",
            "dataBinding": {
                "pullFormat": "${This.Object.Data.Get[firstName]}",
                "pullReplaceNull": "",
                "pushFormat": [
                    "This.Object.Data:SetString[firstName,\"",
                    "\"]"
                ]
            },
            "editTemplate": "propertyview.textbox"
        }
    ]
}
```

###pull Hooks - Event-Driven Updates

**pullHook** triggers data refresh when specific events fire, enabling efficient data synchronization:

```json
{
    "type": "listbox",
    "name": "agentList",
    "itemsBinding": {
        "pullFormat": "${agent.List}",
        "pullHook": {
            "elementName": "innerspace.events",
            "flags": "global",
            "event": "onAgentsUpdated"
        }
    }
}
```

**pullHook Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `elementName` | string | Element that fires the event |
| `flags` | string | "global" to listen to global events |
| `event` | string | Event name to listen for |

**Pattern:** Create a hidden panel as event hub:

```json
{
    "$schema": "http://www.lavishsoft.com/schema/lgui2Package.json",
    "elements": [
        {
            "type": "panel",
            "visibility": "hidden",
            "name": "events"
        },
        {
            "type": "window",
            "title": "Data Window",
            "name": "datawindow",
            "content": {
                "type": "listbox",
                "itemsBinding": {
                    "pullFormat": "${MyController.GetData}",
                    "pullHook": {
                        "elementName": "events",
                        "flags": "global",
                        "event": "onDataUpdated"
                    }
                }
            }
        }
    ]
}
```

In your controller:

```lavishscript
method RefreshData()
{
    ; Update your data...

    ; Fire event to trigger UI refresh
    LGUI2.Element[events]:FireEventHandler[onDataUpdated]
}
```

### pushFormat - Two-Way Binding

**pushFormat** enables UI to write data back to your controller:

```json
{
    "type": "checkbox",
    "checkedBinding": {
        "pullFormat": "${This.Context.Started}",
        "pushFormat": [
            "noop ${This.Context:${If[", ",Reload:Start,Stop]}}"
        ]
    }
}
```

**pushFormat with Arrays:**

```json
{
    "type": "textbox",
    "textBinding": {
        "pullFormat": "${SettingXML[Config.XML].Set[General].Get[Username]}",
        "pushFormat": [
            "SettingXML[Config.XML].Set[General]:Set[Username,\"",
            "\"]:Save"
        ]
    }
}
```

The array elements are concatenated with the user input in between.

### contextBinding - Item Context

**contextBinding** provides context data for item templates:

```json
{
    "type": "listboxitem",
    "contextBinding": {
        "pullFormat": "${agent.Get[${_CONTEXTITEMDATA_.Get[id]}]}"
    },
    "content": {
        "type": "textblock",
        "textBinding": {
            "pullFormat": "${This.Context.Name}"
        }
    }
}
```

**Special Variables:**

- `${This.Context}` - Accesses bound context object
- `${_CONTEXTITEMDATA_.Get[key]}` - Accesses item's data
- `${Context.Source}` - Event source element
- `${Context.Args[index]}` - Event arguments

### Context Variables for ObjectView

When using **ObjectView** for property editing, special context variables provide access to the bound object:

**_CONTEXT_ - ObjectView Object Access:**

```json
{
    "type": "objectview",
    "objectBinding": {
        "pullFormat": "${MyController.SelectedProfile}"
    },
    "properties": [
        {
            "name": "Profile Name",
            "dataBinding": {
                "pullFormat": "${_CONTEXT_.Object.Get[name]}",
                "pushFormat": ["_CONTEXT_.Object:SetString[name,\"", "\"]"]
            },
            "editTemplate": "propertyview.textbox"
        },
        {
            "name": "Game Path",
            "dataBinding": {
                "pullFormat": "${_CONTEXT_.Object.Get[path]}",
                "pushFormat": ["_CONTEXT_.Object:SetString[path,\"", "\"]"]
            },
            "editTemplate": "propertyview.textbox"
        },
        {
            "name": "Lock CPU Affinity",
            "dataBinding": {
                "pullFormat": "${_CONTEXT_.Object.Performance.Get[lockAffinity]}",
                "pushFormat": ["_CONTEXT_.Object.Performance:SetBool[lockAffinity,\"", "\"]"]
            },
            "editTemplate": "propertyview.checkbox"
        }
    ]
}
```

**Context Variable Reference:**

| Variable | Scope | Description |
|----------|-------|-------------|
| `${_CONTEXT_.Object}` | ObjectView properties | Accesses the bound object from objectBinding |
| `${_CONTEXTITEM_}` | List item templates | Accesses the current list item |
| `${_CONTEXTITEMDATA_}` | List item templates | Accesses the current item's data property |
| `${This.Context}` | Elements with contextBinding | Accesses custom context object |

**Pattern: Nested Object Editing**

```json
{
    "type": "objectview",
    "objectBinding": {
        "pullFormat": "${LGUI2.Element[profileList].SelectedItem}"
    },
    "properties": [
        {
            "name": "Profile Name",
            "dataBinding": {
                "pullFormat": "${_CONTEXT_.Object.Get[name]}",
                "pushFormat": ["_CONTEXT_.Object:SetString[name,\"", "\"]"]
            },
            "editTemplate": "propertyview.textbox"
        },
        {
            "name": "Show Border",
            "dataBinding": {
                "pullFormat": "${_CONTEXT_.Object.Highlighter.Get[showBorder]}",
                "pushFormat": ["_CONTEXT_.Object.Highlighter:SetBool[showBorder,\"", "\"]"]
            },
            "editTemplate": "propertyview.checkbox"
        }
    ]
}
```

This allows direct editing of JSON object properties with automatic bidirectional sync.

---

## Styling and Visual Customization

### Colors

Colors can be specified as:

**Hex strings:**

```json
"color": "#ff0000"        // Red
"color": "#00ff00"        // Green
"color": "#0000ff"        // Blue
"color": "#ffffff"        // White
```

**RGB/RGBA arrays:**

```json
"color": [1.0, 0.0, 0.0]           // Red (RGB)
"color": [1.0, 1.0, 1.0, 0.5]      // Semi-transparent white (RGBA)
```

### Brushes

Brushes are used to "paint" visual elements in LavishGUI 2. A Brush consists of a color, optional image/canvas, and optional pixel/vertex shaders.

**Brush Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `color` | Color | RGBA color (default: [1.0, 0, 0, 0]) |
| `imageFile` | string | Optional image filename for texture |
| `canvas` | Canvas | Optional Canvas definition for rendering |
| `imageFileTransparencyKey` | Color | Color to treat as transparent (default: [0, 0, 0, 0]) |
| `imageBrush` | Brush | Alternative Brush for rendering the image |
| `imageOrientation` | string | "normal", "mirrorHorizontal", "mirrorVertical" |
| `pixelShader` | Pixel Shader | Provides final color for each pixel |
| `vertexShader` | Vertex Shader | Optional vertex shader definition |

**Solid Color Brush:**

```json
"backgroundBrush": {
    "color": "#ffffff"
}
```

Or using RGBA array format:

```json
"backgroundBrush": {
    "color": [1.0, 1.0, 1.0, 1.0]
}
```

**Image Brush:**

```json
"backgroundBrush": {
    "color": "#ffffff",
    "imageFile": "Interface/textures/background.png"
}
```

**Image Brush with Tinting:**

Blend a color with an image to create tinted effects:

```json
"backgroundBrush": {
    "color": [1.0, 0.5, 0.5, 1.0],
    "imageFile": "Interface/textures/button.png"
}
```

**Image Brush with Transparency:**

```json
"backgroundBrush": {
    "imageFile": "Interface/icons/icon.png",
    "imageFileTransparencyKey": [0, 0, 0, 0]
}
```

**Image Orientation:**

Mirror images horizontally or vertically:

```json
"backgroundBrush": {
    "imageFile": "Interface/textures/arrow.png",
    "imageOrientation": "mirrorHorizontal"
}
```

Possible values:
- `"normal"` - No transformation
- `"mirrorHorizontal"` - Flip horizontally
- `"mirrorVertical"` - Flip vertically

**Named Brushes (Global Brushes):**

Define brushes in a Skin for reuse:

```json
{
    "brushes": {
        "white": {
            "color": "#ffffff"
        },
        "darkBlue": {
            "color": "#000080"
        }
    }
}
```

Then reference by name:

```json
"backgroundBrush": "white",
"borderBrush": "darkBlue"
```

**Canvas Brush:**

Use a Canvas for dynamic rendering:

```json
"backgroundBrush": {
    "canvas": {
        "type": "canvas",
        "width": 100,
        "height": 100
    }
}
```

### Borders and Margins

**Border:**

```json
{
    "type": "panel",
    "borderThickness": 2,
    "borderBrush": {
        "color": "#ff0000"
    }
}
```

**Margin:**

```json
{
    "type": "textblock",
    "margin": [5, 10, 5, 10],    // [left, top, right, bottom]
    "text": "Text with margins"
}
```

### Padding

```json
{
    "type": "button",
    "padding": 5,
    "content": "Padded button"
}
```

### Fonts

Fonts in LavishGUI 2 specify text appearance through a JSON object configuration.

**Font Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `face` | string | Font name (e.g., "Arial", "Comic Sans MS") |
| `height` | integer | **Font height in POINTS** (e.g., 12 for 12pt font, NOT pixels) |
| `heightOffset` | integer | Height adjustment in points relative to parent Font |
| `heightFactor` | float | Multiplier: heightOffset + (heightFactor × parentHeight) |
| `bold` | boolean | Applies bold styling |
| `fixed` | boolean | Enables fixed-width (monospace) rendering |

**Important:** The `height` property uses **point-based sizing**, not pixel-based sizing!

**Basic Font Example:**

```json
{
    "type": "textblock",
    "text": "Fancy Text",
    "font": {
        "face": "Arial",
        "height": 12,
        "bold": true
    }
}
```

**12-point Arial:**

```json
"font": {
    "face": "Arial",
    "height": 12
}
```

**14-point Bold Comic Sans MS:**

```json
"font": {
    "face": "Comic Sans MS",
    "height": 14,
    "bold": true
}
```

**Fixed-Width (Monospace) Font:**

```json
"font": {
    "face": "Courier New",
    "height": 10,
    "fixed": true
}
```

**Relative Font Sizing with heightOffset:**

```json
"font": {
    "face": "Arial",
    "heightOffset": 2
}
```

This creates a font 2 points larger than the parent element's font.

**Relative Font Sizing with heightFactor:**

```json
"font": {
    "face": "Arial",
    "heightFactor": 1.5
}
```

This creates a font 1.5× the size of the parent element's font.

**Combined Relative Sizing:**

```json
"font": {
    "face": "Arial",
    "heightOffset": 2,
    "heightFactor": 1.2
}
```

Calculates as: `heightOffset + (heightFactor × parentHeight) = 2 + (1.2 × parentHeight)`

### Dynamic Styles

**Dynamic styles** are LGUI2's primary mechanism for changing element appearance at runtime. This section covers how to use styles for dynamic visual changes.

#### Key Concept: Why Styles?

Unlike LGUI1 which supported direct property manipulation (e.g., `SetAlpha`, `Font:SetColor`), LGUI2 uses a **style-based approach** for dynamic visual changes.

**LGUI1 Approach (No Longer Works):**
```lavishscript
; These methods DON'T exist in LGUI2
UIElement[mytext]:SetAlpha[0.5]
UIElement[mybutton].Font:SetColor[FF00FF00]
```

**LGUI2 Approach (Correct):**
```lavishscript
; Use FireEventHandler to apply predefined styles
LGUI2.Element[mytext]:FireEventHandler[setDimmed]
LGUI2.Element[mybutton]:FireEventHandler[setGreen]
```

**Important Limitation:** The `:Set[property,value]` method in LGUI2 **only works for simple top-level properties** like `text` and `isOpen`. It does **NOT work** for nested properties like `opacity`, `backgroundBrush.color`, or `font.color`.

#### Basic Style Pattern

**1. Define Named Styles in JSON:**

```json
{
    "type": "textblock",
    "text": "Example",
    "styles": {
        "highlighted": {
            "color": "#ffff00",
            "backgroundBrush": {"color": "#000000"}
        },
        "normal": {
            "color": "#ffffff",
            "backgroundBrush": {"color": "#333333"}
        }
    }
}
```

**2. Create Event Handlers to Apply Styles:**

```json
{
    "type": "textblock",
    "text": "Example",
    "styles": {
        "highlighted": {
            "color": "#ffff00"
        },
        "normal": {
            "color": "#ffffff"
        }
    },
    "eventHandlers": {
        "setHighlighted": {
            "type": "style",
            "styleName": "highlighted"
        },
        "setNormal": {
            "type": "style",
            "styleName": "normal"
        }
    }
}
```

**3. Trigger from Script Using FireEventHandler:**

```lavishscript
; Apply the "highlighted" style
LGUI2.Element[mytext]:FireEventHandler[setHighlighted]

; Apply the "normal" style
LGUI2.Element[mytext]:FireEventHandler[setNormal]
```

#### Style Properties

Styles can contain any visual property that an element supports:

```json
"styles": {
    "customStyle": {
        "opacity": 0.8,
        "backgroundBrush": {"color": "#336699"},
        "foregroundBrush": {"color": "#ffffff"},
        "borderBrush": {"color": "#000000"},
        "borderThickness": 2,
        "padding": 5,
        "margin": [10, 5, 10, 5],
        "color": "#ffff00",
        "font": {"face": "Arial", "height": 14, "bold": true}
    }
}
```

#### Example 1: Textbox Opacity (Enable/Disable Visual State)

**Use Case:** Dim textboxes when their associated checkbox is unchecked.

```json
{
    "type": "textbox",
    "name": "userInput",
    "text": "Enter value...",
    "opacity": 1.0,
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
function OnCheckboxToggled(bool isChecked)
{
    if ${isChecked}
        LGUI2.Element[userInput]:FireEventHandler[setEnabled]
    else
        LGUI2.Element[userInput]:FireEventHandler[setDisabled]
}
```

**Why This Works:**
- `opacity` is a nested property that can't be set via `:Set[opacity,value]`
- Styles provide the only way to change opacity dynamically
- `FireEventHandler` triggers the style application

#### Example 2: Button State Colors (Toggle Buttons)

**Use Case:** Toggle buttons that change color to indicate active/inactive state.

**Important:** Buttons in LGUI2 **ignore** `font.color` and `foregroundBrush.color` for their content text. Use `backgroundBrush.color` to change button background color instead.

```json
{
    "type": "button",
    "name": "toggleButton",
    "content": "Feature: OFF",
    "backgroundBrush": {"color": "#FF0000"},
    "styles": {
        "active": {
            "backgroundBrush": {"color": "#32CD32"}
        },
        "inactive": {
            "backgroundBrush": {"color": "#FF0000"}
        }
    },
    "eventHandlers": {
        "onPress": {
            "type": "code",
            "code": "Script[MyScript]:QueueCommand[call ToggleFeature]"
        },
        "setActive": {
            "type": "style",
            "styleName": "active"
        },
        "setInactive": {
            "type": "style",
            "styleName": "inactive"
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
        LGUI2.Element[toggleButton]:SetContent["Feature: OFF"]
        LGUI2.Element[toggleButton]:FireEventHandler[setInactive]
    }
    else
    {
        FeatureActive:Set[TRUE]
        LGUI2.Element[toggleButton]:SetContent["Feature: ON"]
        LGUI2.Element[toggleButton]:FireEventHandler[setActive]
    }
}
```

#### Example 3: Persistent Button Colors (Solving Hover Override)

**Problem:** Button colors revert to default when mouse hovers over them.

**Root Cause:** The skin's built-in hover state overrides custom styles.

**Solution:** Define all button brush states and re-apply styles on mouse events.

```json
{
    "type": "button",
    "name": "persistentButton",
    "content": "Toggle",
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
            "code": "Script[MyScript]:QueueCommand[call Toggle]"
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
variable bool IsActive = FALSE
variable string ButtonState = "red"

function Toggle()
{
    if ${IsActive}
    {
        IsActive:Set[FALSE]
        ButtonState:Set["red"]
        LGUI2.Element[persistentButton]:SetContent["OFF"]
        LGUI2.Element[persistentButton]:FireEventHandler[setRed]
    }
    else
    {
        IsActive:Set[TRUE]
        ButtonState:Set["green"]
        LGUI2.Element[persistentButton]:SetContent["ON"]
        LGUI2.Element[persistentButton]:FireEventHandler[setGreen]
    }
}

function ReapplyButtonColor()
{
    if ${ButtonState.Equal["green"]}
        LGUI2.Element[persistentButton]:FireEventHandler[setGreen]
    else
        LGUI2.Element[persistentButton]:FireEventHandler[setRed]
}
```

**Critical Requirements for Persistent Colors:**

1. **Define all brush states** - `backgroundBrush`, `hoverBackgroundBrush`, `pressedBackgroundBrush`, `disabledBackgroundBrush`
2. **Track state in a variable** - Needed for the reapply function to know which style to use
3. **Handle mouse events** - `gotMouseOver` and `lostMouseOver` events re-apply the current style
4. **Create a reapply function** - Checks the state variable and re-fires the appropriate style event

**Why This Pattern Works:**

- Skin hover states attempt to override your custom styles
- By handling `gotMouseOver`/`lostMouseOver`, you "fight back" and re-apply your custom colors
- Defining all brush state variants prevents the skin from having default colors to fall back to
- The state tracking variable ensures the reapply function knows which color to apply

#### Example 4: Automatic Hover Styles

For simple hover effects (no persistence needed), let events handle it automatically:

```json
{
    "type": "button",
    "content": "Hover Me",
    "styles": {
        "hover": {
            "backgroundBrush": {"color": "#ff0000"}
        },
        "normal": {
            "backgroundBrush": {"color": "#0000ff"}
        }
    },
    "eventHandlers": {
        "gotMouseOver": {
            "type": "style",
            "styleName": "hover"
        },
        "lostMouseOver": {
            "type": "style",
            "styleName": "normal"
        }
    }
}
```

This works well for temporary hover effects, but **not** for toggle buttons that need to maintain their color state.

#### Style Application from Script

**Using FireEventHandler:**

```lavishscript
; Apply a named style by triggering its event handler
LGUI2.Element[myButton]:FireEventHandler[setGreen]
LGUI2.Element[myText]:FireEventHandler[setHighlighted]
```

**Common Pattern: State-Based Styling:**

```lavishscript
variable bool IsEnabled = FALSE

function UpdateVisualState()
{
    if ${IsEnabled}
        LGUI2.Element[myElement]:FireEventHandler[setEnabled]
    else
        LGUI2.Element[myElement]:FireEventHandler[setDisabled]
}
```

#### Properties That Require Styles

Use styles for these properties (`:Set` won't work):

- `opacity` - Element transparency (0.0 to 1.0)
- `backgroundBrush.color` - Background color
- `foregroundBrush.color` - Foreground/content color
- `borderBrush.color` - Border color
- `font.color` - Text color (limited - buttons ignore this)
- `hoverBackgroundBrush.color` - Hover state background
- `pressedBackgroundBrush.color` - Pressed state background
- `disabledBackgroundBrush.color` - Disabled state background
- **Any property with dot notation** - All nested properties

Properties that work with `:Set`:

- `text` - Text content (use `:SetText`)
- `isOpen` - Open/closed state
- `checked` - Checkbox state (use `:SetChecked`)
- **Simple top-level boolean/string properties**

#### Complete Working Example

**Real-world example from EQ2BotCommander:**

```json
{
    "type": "button",
    "name": "Main.RunEQ2Bot",
    "content": "Run EQ2BOT",
    "font": {"height": 36},
    "backgroundBrush": {"color": "#FF0000"},
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
            "code": "Script[EQ2BotCommander]:QueueCommand[call ToggleRunEQ2Bot]"
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
            "code": "Script[EQ2BotCommander]:QueueCommand[call ReapplyRunEQ2BotButtonColor]"
        },
        "lostMouseOver": {
            "type": "code",
            "code": "Script[EQ2BotCommander]:QueueCommand[call ReapplyRunEQ2BotButtonColor]"
        }
    }
}
```

```lavishscript
variable bool EQ2BotRunning = FALSE

function ToggleRunEQ2Bot()
{
    if ${EQ2BotRunning} == FALSE
    {
        EQ2BotRunning:Set[TRUE]
        LGUI2.Element[Main.RunEQ2Bot]:SetContent["End EQ2BOT"]
        LGUI2.Element[Main.RunEQ2Bot]:FireEventHandler[setGreen]
        ; Start EQ2Bot...
    }
    else
    {
        EQ2BotRunning:Set[FALSE]
        LGUI2.Element[Main.RunEQ2Bot]:SetContent["Run EQ2BOT"]
        LGUI2.Element[Main.RunEQ2Bot]:FireEventHandler[setRed]
        ; Stop EQ2Bot...
    }
}

function ReapplyRunEQ2BotButtonColor()
{
    if ${EQ2BotRunning}
        LGUI2.Element[Main.RunEQ2Bot]:FireEventHandler[setGreen]
    else
        LGUI2.Element[Main.RunEQ2Bot]:FireEventHandler[setRed]
}
```

**Reference Implementation:** See [EQ2BotCommander.json](https://github.com/isxGames/isxScripts/blob/master/EverQuest2/Scripts/EQ2Bot/UI/EQ2BotCommander.json) and [EQ2BotCommander.iss](https://github.com/isxGames/isxScripts/blob/master/EverQuest2/Scripts/EQ2BotCommander.iss) for a complete production example with 11 toggle buttons using this pattern.

#### Summary: Dynamic Styles Best Practices

**When to Use Styles:**

- ✅ Changing element opacity
- ✅ Changing button background colors
- ✅ Changing any nested visual property
- ✅ Toggle buttons with visual state indication
- ✅ Any property that can't be set via `:Set` method

**Key Patterns:**

1. **Define named styles** in JSON with all visual states
2. **Create custom event handlers** of type "style" to apply those styles
3. **Use `FireEventHandler`** from script to trigger style changes
4. **For buttons:** Use `backgroundBrush.color` (NOT `font.color`)
5. **For persistent colors:** Define all brush states and handle mouse events
6. **Track state in variables** for reapply functions

**Common Pitfall:**

```lavishscript
; DON'T - These won't work in LGUI2
LGUI2.Element[myButton]:Set[opacity,0.5]
LGUI2.Element[myButton]:Set[backgroundBrush.color,"#FF0000"]
LGUI2.Element[myText]:SetAlpha[0.5]  ; Method doesn't exist

; DO - Use styles instead
LGUI2.Element[myButton]:FireEventHandler[setDimmed]
LGUI2.Element[myButton]:FireEventHandler[setRed]
LGUI2.Element[myText]:FireEventHandler[setEnabled]
```

---

## Templates

Templates define reusable UI patterns.

### Defining Templates

Templates are defined in the `templates` section of the package:

```json
{
    "$schema": "http://www.lavishsoft.com/schema/lgui2Package.json",
    "templates": {
        "fancyText": {
            "font": {
                "bold": true,
                "size": 14
            },
            "color": "#ff0000"
        },
        "primaryButton": {
            "backgroundBrush": {
                "color": "#0000ff"
            },
            "padding": 10
        }
    },
    "elements": [
        ...
    ]
}
```

### Using Templates

Apply templates with `jsonTemplate`:

```json
{
    "jsonTemplate": "fancyText",
    "type": "textblock",
    "text": "This text uses the fancyText template"
}
```

**Multiple elements using same template:**

```json
"children": [
    {
        "jsonTemplate": "primaryButton",
        "type": "button",
        "content": "Save"
    },
    {
        "jsonTemplate": "primaryButton",
        "type": "button",
        "content": "Cancel"
    }
]
```

### Data Templates

Templates can store data:

```json
"templates": {
    "editor.data": {
        "people": [
            {
                "firstName": "John",
                "lastName": "Doe",
                "birthYear": 1980
            },
            {
                "firstName": "Jane",
                "lastName": "Smith",
                "birthYear": 1985
            }
        ]
    }
}
```

Access template data:

```json
"itemsBinding": {
    "pullFormat": "${LGUI2.Skin[default].Template[editor.data].Get[people]}",
    "pullOnce": true
}
```

### Item View Templates

Define how list items are displayed:

```json
"templates": {
    "personView": {
        "jsonTemplate": "listboxitem",
        "padding": 2,
        "content": {
            "type": "stackpanel",
            "orientation": "vertical",
            "children": [
                {
                    "type": "textblock",
                    "textBinding": {
                        "pullFormat": "${_CONTEXTITEMDATA_.Get[firstName]} ${_CONTEXTITEMDATA_.Get[lastName]}"
                    }
                }
            ]
        }
    }
}
```

Use in ListBox:

```json
{
    "type": "listbox",
    "itemsBinding": {
        "pullFormat": "${MyController.GetPeople}"
    },
    "itemViewGenerators": {
        "default": {
            "type": "template",
            "template": "personView"
        }
    }
}
```

---

## Advanced Features

### Input Hooks

Capture keyboard and mouse input:

```json
{
    "type": "textblock",
    "text": "Press any key!",
    "inputHooks": {
        "keyboard hook": {
            "control": {
                "deviceType": "keyboard",
                "controlType": "button",
                "value": 1
            },
            "eventHandler": {
                "type": "code",
                "code": "echo Key pressed: ${Context.Args[controlName]}"
            }
        }
    }
}
```

### Event Hooks

Hooks provide a way to attach Event Handlers to events from other related elements. This is useful for coordinating behavior across multiple UI elements.

**Hook Definition Structure:**

Hooks are defined as JSON objects with these properties:

| Property | Type | Description |
|----------|------|-------------|
| `elementType` | string | (Optional) The element type to attach to |
| `elementName` | string | (Optional) The name of element to attach to |
| `flags` | string | (Optional) Relationship flags (default: "ancestor") |
| `event` | string | **Required** - Event name to listen for |
| `eventHandler` | Event Handler | **Required** - Handler to execute when event fires |

**Default Behavior:** When `elementType` and `elementName` are omitted, the parent element is used.

**Relationship Flags:**
- `"ancestor"` (default) - Search up the parent chain
- `"global"` - Search globally across all elements

**Basic Hook Example:**

Attach a style to a parent checkbox event:

```json
{
    "type": "panel",
    "hooks": {
        "onUnchecked": {
            "elementType": "checkbox",
            "event": "onUnchecked",
            "eventHandler": {
                "type": "style",
                "styleName": "onUnchecked"
            }
        }
    }
}
```

**Global Event Hook:**

Listen to global events from a named element:

```json
{
    "type": "textblock",
    "textBinding": {
        "pullFormat": "${MyController.Status}"
    },
    "hooks": {
        "statusUpdate": {
            "elementName": "events",
            "flags": "global",
            "event": "onStatusChanged",
            "eventHandler": {
                "type": "code",
                "code": "This:Pull"
            }
        }
    }
}
```

**Hook with Element Locator:**

Attach to a specific named element anywhere in the UI:

```json
{
    "hooks": {
        "settingsChanged": {
            "elementName": "settings.panel",
            "flags": "global",
            "event": "onSettingsUpdated",
            "eventHandler": {
                "type": "method",
                "object": "MyController",
                "method": "RefreshSettings"
            }
        }
    }
}
```

**Multiple Hooks:**

Elements can have multiple hooks:

```json
{
    "type": "panel",
    "hooks": {
        "hook1": {
            "elementName": "dataSource",
            "flags": "global",
            "event": "onDataChanged",
            "eventHandler": {
                "type": "code",
                "code": "This:RefreshDisplay"
            }
        },
        "hook2": {
            "elementName": "config",
            "flags": "global",
            "event": "onConfigChanged",
            "eventHandler": {
                "type": "code",
                "code": "This:ApplyNewConfig"
            }
        }
    }
}
```

**Best Practice - Event Hub Pattern:**

Create a hidden panel as a centralized event broadcaster:

```json
{
    "$schema": "http://www.lavishsoft.com/schema/lgui2Package.json",
    "elements": [
        {
            "type": "panel",
            "name": "eventHub",
            "visibility": "hidden"
        },
        {
            "type": "window",
            "content": {
                "type": "textblock",
                "textBinding": {
                    "pullFormat": "${MyController.Data}"
                },
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
        }
    ]
}
```

In your controller script:

```lavishscript
method UpdateData()
{
    ; Update your data...

    ; Broadcast event to all hooked elements
    LGUI2.Element[eventHub]:FireEventHandler[onDataUpdated]
}
```

### Visibility Control

**visibility Property (JSON):**

The `visibility` property in JSON uses lowercase string values:

```json
{
    "type": "panel",
    "name": "myPanel",
    "visibility": "visible"
}
```

**Visibility Values:**

| Value | Behavior |
|-------|----------|
| `"visible"` | Element is shown (default) |
| `"hidden"` | Element is hidden but takes up layout space |
| `"collapsed"` | Element is hidden and doesn't take up layout space |

**SetVisibility Method (Script):**

Use capitalized enum values when calling from script:

```lavishscript
; CORRECT - Capitalized
LGUI2.Element[myPanel]:SetVisibility[Visible]
LGUI2.Element[myPanel]:SetVisibility[Hidden]
LGUI2.Element[myPanel]:SetVisibility[Collapsed]

; WRONG - Lowercase doesn't work
LGUI2.Element[myPanel]:SetVisibility[visible]   ; Won't work!
```

**Important Distinction:**

- **JSON property:** Use lowercase strings (`"visible"`, `"hidden"`, `"collapsed"`)
- **Script method:** Use capitalized enum values (`Visible`, `Hidden`, `Collapsed`)

**Example:**

```json
{
    "type": "window",
    "name": "mywindow",
    "visibility": "hidden",
    "content": {
        "type": "button",
        "content": "Show Me",
        "eventHandlers": {
            "onPress": {
                "type": "code",
                "code": "LGUI2.Element[mywindow]:SetVisibility[Visible]"
            }
        }
    }
}
```

### Triggers

Triggers fire event handlers based on conditions:

```json
{
    "type": "imagebox",
    "imageBrush": {
        "imageFile": "icon.png"
    },
    "triggers": {
        "health-warning": {
            "condition": "${Me.Health} < 50",
            "matched": {
                "type": "audio",
                "voiceName": "ui sound",
                "streamName": "warning"
            },
            "unmatched": {
                "type": "audio",
                "voiceName": "ui sound"
            }
        }
    }
}
```

### FilePicker

Built-in file selection dialog:

```json
{
    "type": "filepicker",
    "name": "myFilePicker",
    "horizontalAlignment": "stretch",
    "verticalAlignment": "stretch",
    "requireExisting": true,
    "multiselect": false,
    "path": ".",
    "wildcard": "*.iss",
    "eventHandlers": {
        "onFinalized": {
            "type": "method",
            "object": "MyController",
            "method": "OnFileSelected"
        }
    }
}
```

In controller:

```lavishscript
method OnFileSelected()
{
    if ${Context.Source.IsMultiselect}
        echo Selected files: ${Context.Source.Values~}
    else
        echo Selected file: ${Context.Source.Value~}
}
```

### Animations

Animations in LavishGUI 2 modify elements over time, unlike Styles which apply changes instantly. Each animation is an instance of a specific Animation Type with customized parameters.

**Basic Animation Structure:**

```json
{
    "type": "slide",
    "name": "slide-animation",
    "duration": 1000,
    "destination": [800, 600]
}
```

**Core Animation Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `type` | string | Animation Type (e.g., "slide", "fade", "value") **[Required]** |
| `name` | string | Unique identifier within element (replaces existing with same name) |
| `duration` | number | Time in milliseconds (integers) or seconds (floats) |
| `instant` | boolean | If true, zero-duration animations execute instantly vs. running indefinitely |

**Available Animation Types:**

**Basic Animations:**
- **fade** - Opacity transitions
- **slide** - Position animations
- **value** - Numeric value animations

**Utility Animations:**
- **chain** - Sequential animation sequences
- **composite** - Parallel animation combinations
- **delay** - Timed pauses between animations
- **random** - Randomized animation selection
- **repeat** - Looping animation patterns

**Fade Animation Example:**

```json
{
    "type": "button",
    "name": "fadeButton",
    "content": "Fade Me",
    "animations": {
        "fadeOut": {
            "type": "fade",
            "name": "fadeOut",
            "duration": 500,
            "from": 1.0,
            "to": 0.0
        },
        "fadeIn": {
            "type": "fade",
            "name": "fadeIn",
            "duration": 500,
            "from": 0.0,
            "to": 1.0
        }
    },
    "eventHandlers": {
        "gotMouseOver": {
            "type": "animation",
            "animation": "fadeIn"
        },
        "lostMouseOver": {
            "type": "animation",
            "animation": "fadeOut"
        }
    }
}
```

**Slide Animation Example:**

```json
{
    "animations": {
        "slideToCorner": {
            "type": "slide",
            "name": "slideToCorner",
            "duration": 1000,
            "destination": [800, 600]
        }
    }
}
```

**Chain Animation Example:**

```json
{
    "animations": {
        "sequencedMove": {
            "type": "chain",
            "name": "sequencedMove",
            "animations": [
                {
                    "type": "slide",
                    "duration": 500,
                    "destination": [100, 100]
                },
                {
                    "type": "delay",
                    "duration": 200
                },
                {
                    "type": "slide",
                    "duration": 500,
                    "destination": [200, 200]
                }
            ]
        }
    }
}
```

**Triggering Animations from Script:**

```lavishscript
; Start an animation
LGUI2.Element[fadeButton]:Animate[fadeOut]

; Stop an animation
LGUI2.Element[fadeButton]:StopAnimation[fadeOut]
```

**Triggering Animations from Event Handlers:**

```json
{
    "eventHandlers": {
        "onPress": {
            "type": "animation",
            "animation": "slideToCorner"
        }
    }
}
```

### Item View Generators

Dynamically generate custom item views:

```json
{
    "type": "listbox",
    "itemsBinding": {
        "pullFormat": "${Audio.Streams}"
    },
    "itemViewGenerators": {
        "default": {
            "type": "method",
            "object": "MyController",
            "method": "GetItemView"
        }
    }
}
```

In controller:

```lavishscript
method GetItemView()
{
    Context:SetView["$$>
    {
        \"type\":\"itemview\",
        \"content\":{
            \"type\":\"dockpanel\",
            \"children\":[
                {
                    \"_dock\":\"left\",
                    \"type\":\"textblock\",
                    \"text\":${Context.Args[name].AsJSON~}
                },
                {
                    \"_dock\":\"right\",
                    \"type\":\"button\",
                    \"content\":\"Play\"
                }
            ]
        }
    }
    <$$"]
}
```

### Canvas Drawing API

The `canvas` element provides a powerful 2D drawing surface for creating custom graphics, game elements, and visual effects.

**Basic Canvas Structure:**

```json
{
    "type": "canvas",
    "name": "myCanvas",
    "width": 640,
    "height": 480,
    "brushes": {
        "myBrush": {"color": "#ffcccc66"}
    },
    "initialize": [
        {
            "op": "clear",
            "color": "#ff000000"
        }
    ]
}
```

**Canvas Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `brushes` | object | Defines reusable brushes with colors or gradients |
| `initialize` | array | Drawing operations to execute on creation |
| `strata` | number | Z-order priority (default 0.5) |

**Drawing Operations:**

**Clear Canvas:**

```json
{
    "op": "clear",
    "color": "#ff000000"
}
```

**Push Brush (Activate for Drawing):**

```json
{
    "op": "pushBrush",
    "brush": "myBrush"
}
```

**Draw Circle:**

```json
{
    "op": "drawCircle",
    "x": 0.5,              // X position (0-1 = factor-based, >1 = pixel-based)
    "y": 0.5,              // Y position
    "radius": 0.1,         // Radius
    "resolution": 32,      // Number of segments (optional, default: 32)
    "transform": true,     // Apply transformations (optional, default: true)
    "opacity": 0.9         // Opacity 0.0-1.0 (optional)
}
```

**Note on Coordinates:**
- Values between 0 and 1 are factor-based (relative to canvas size)
- Values greater than 1 are pixel-based (absolute coordinates)
- `"transform": false` uses pixel coordinates regardless

**Draw Primitives (Lines, Triangles):**

```json
{
    "op": "drawPrimitives",
    "type": "lineList",
    "vertices": [
        [x1, y1, u1, v1, "#ffRRGGBB"],
        [x2, y2, u2, v2, "#ffRRGGBB"]
    ]
}
```

**Primitive Types:**
- `"lineList"` - Each pair of vertices draws a line
- `"triangleList"` - Each triplet of vertices draws a triangle

**Vertex Format:** `[x, y, u, v, color]`
- `x, y` - Position (factor or pixel-based)
- `u, v` - Texture coordinates (usually 0)
- `color` - ARGB hex color string (e.g., `"#ffff0000"` for red)

**Complete Canvas Example - Draw a Stick Figure:**

```json
{
    "type": "canvas",
    "name": "player",
    "width": 48,
    "height": 48,
    "brushes": {
        "skin": {"color": "#ffcccc66"}
    },
    "initialize": [
        {
            "op": "pushBrush",
            "brush": "skin"
        },
        {
            "op": "drawCircle",
            "x": 0.5,
            "y": 0.2,
            "radius": 0.1
        },
        {
            "op": "drawPrimitives",
            "type": "lineList",
            "vertices": [
                [0.5, 0.3, 0, 0, "#ffff0000"],
                [0.5, 0.6, 0, 0, "#ffff0000"],

                [0.5, 0.35, 0, 0, "#ffff0000"],
                [0.35, 0.55, 0, 0, "#ff00ff00"],

                [0.5, 0.35, 0, 0, "#ffff0000"],
                [0.65, 0.55, 0, 0, "#ff00ff00"],

                [0.5, 0.6, 0, 0, "#ffff0000"],
                [0.35, 0.8, 0, 0, "#ff00ff00"],

                [0.5, 0.6, 0, 0, "#ffff0000"],
                [0.65, 0.8, 0, 0, "#ff00ff00"]
            ]
        }
    ]
}
```

**Reference:** See https://github.com/LavishSoftware/LERN/tree/master/Game for complete game examples using canvas drawing.

### Audio System Integration

LavishGUI 2 provides seamless audio integration for background music, sound effects, and spatial audio.

**Audio Package Structure:**

```json
{
    "$schema": "http://www.lavishsoft.com/schema/lgui2Package.json",
    "audioVoices": {
        "game.player": {},
        "game.music": {"volume": [0.4, 0.4]}
    },
    "audioStreams": {
        "background-music": {
            "filename": "../Assets/Audio/music.mp3"
        },
        "jump-sound": {
            "filename": "../Assets/Audio/jump.mp3"
        }
    },
    "elements": [...]
}
```

**Audio Voices:**

Audio voices are output channels that can play streams. Define them in the `audioVoices` object:

```json
"audioVoices": {
    "voiceName": {},                         // Simple voice
    "music": {"volume": [0.5, 0.5]},        // With initial volume [left, right]
    "effects": {"volume": [1.0, 1.0]}
}
```

**Audio Streams:**

Audio streams are sound files that can be played on voices:

```json
"audioStreams": {
    "streamName": {
        "filename": "path/to/audio.mp3"
    }
}
```

**Supported Formats:**
- MP3
- WAV
- OGG
- Other formats supported by DirectSound

**Playing Audio from Script:**

```lavishscript
; Play a stream once
Audio.Voice[game.player]:PlayStream[jump-sound]

; Play a stream with loop count (-1 = infinite loop)
Audio.Voice[game.music]:PlayStream[background-music, -1]

; Stop playback
Audio.Voice[game.music]:Stop

; Stop and clear queue
Audio.Voice[game.music]:Stop:ClearQueue

; Set volume (left, right) - values 0.0 to 1.0
Audio.Voice[game.player]:SetVolume[0.8, 0.8]

; Stereo panning example (left=1.0, right=0.0)
Audio.Voice[game.player]:SetVolume[1.0, 0.0]
```

**Playing Audio from Event Handler:**

```json
{
    "type": "button",
    "content": "Play Sound",
    "eventHandlers": {
        "onPress": {
            "type": "audio",
            "voiceName": "ui-sound",
            "streamName": "button-click"
        }
    }
}
```

**Stereo Panning Pattern:**

Calculate stereo positioning based on element location:

```lavishscript
method UpdatePlayerAudio()
{
    ; Calculate percentage position (0.0 = left, 1.0 = right)
    variable float pctX=0.5
    if ${LGUI2.Element[game.board].ActualWidth}
    {
        pctX:Set[ ( ${LGUI2.Element[game.player].X} + ( ${LGUI2.Element[game.player].ActualWidth}/2 ) ) / ${LGUI2.Element[game.board].ActualWidth} ]
    }

    ; Clamp to 0-1 range
    if ${pctX}<0
        pctX:Set[0]
    elseif ${pctX}>1
        pctX:Set[1]

    ; Calculate stereo volumes
    variable float useLeft
    variable float useRight

    useLeft:Set[(1-${pctX})*2]
    useRight:Set[(${pctX})*2]

    ; Clamp volumes to 1.0 max
    if ${useLeft}>1
        useLeft:Set[1]
    if ${useRight}>1
        useRight:Set[1]

    Audio.Voice[game.player]:SetVolume[${useLeft}, ${useRight}]
}
```

**Cleanup Pattern:**

Always stop audio in your Shutdown method:

```lavishscript
method Shutdown()
{
    LGUI2:UnloadPackageFile[myui.json]

    ; Stop all audio voices
    Audio.Voice[game.music]:Stop:ClearQueue
    Audio.Voice[game.player]:Stop:ClearQueue
}
```

**Reference:** See https://github.com/LavishSoftware/LERN/tree/master/Game/platform-2.iss for complete audio example.

### Keyboard Input Handling

The `onButtonMove` event provides low-level keyboard and gamepad input handling, perfect for games or applications requiring direct input control.

**Basic Keyboard Handler:**

```json
{
    "type": "panel",
    "name": "game.board",
    "acceptsKeyboardFocus": true,
    "acceptsMouseFocus": true,
    "eventHandlers": {
        "onButtonMove": {
            "type": "method",
            "object": "GameController",
            "method": "OnButtonMove"
        }
    }
}
```

**Taking Keyboard Focus:**

```lavishscript
method Initialize()
{
    LGUI2:LoadPackageFile[game.json]

    ; Take keyboard focus so we receive input
    LGUI2.Element[game.board]:KeyboardFocus
}
```

**Reading Input in Handler:**

```lavishscript
method OnButtonMove()
{
    ; Context.Args provides:
    ; - controlName: Key name (e.g., "Esc", "Left", "Space", "A")
    ; - position: 1 = key pressed, 0 = key released

    switch ${Context.Args[controlName]}
    {
        case Esc
            Script:End
            break
        case Left
            if ${Context.Args[position]}
                echo Left key pressed
            else
                echo Left key released
            break
        case Space
            if ${Context.Args[position]}
                echo Jump!
            break
    }
}
```

**Common Key Names:**

| Key | Name | Key | Name |
|-----|------|-----|------|
| Escape | `Esc` | Space | `Space` |
| Arrow Keys | `Left`, `Right`, `Up`, `Down` | Enter | `Return` |
| Letters | `A`, `B`, `C`, ... `Z` | Numbers | `0`, `1`, ... `9` |
| Shift | `LShift`, `RShift` | Control | `LControl`, `RControl` |
| Alt | `LAlt`, `RAlt` | Tab | `Tab` |

**Movement Pattern with Press/Release:**

```lavishscript
variable float SpeedX
variable float SpeedY
variable float BaseSpeed=100

method OnButtonMove()
{
    switch ${Context.Args[controlName]}
    {
        case A
        case Left
            if ${Context.Args[position]}
                SpeedX:Set[-${BaseSpeed}]    ; Moving left
            else
                SpeedX:Set[0]                 ; Stopped
            break
        case D
        case Right
            if ${Context.Args[position]}
                SpeedX:Set[${BaseSpeed}]      ; Moving right
            else
                SpeedX:Set[0]                 ; Stopped
            break
        case Space
            if ${Context.Args[position]}
                This:Jump                     ; Only on press, not release
            break
    }
}
```

**Gamepad Support:**

The same event handler works for gamepad input:
- D-pad triggers `onButtonMove` and `onDpadMove`
- Analog sticks trigger `onAxisMove`

**Best Practices:**

1. **Always take keyboard focus** in Initialize
2. **Set acceptsKeyboardFocus to true** on the element
3. **Handle both press and release** for movement (use position check)
4. **Handle only press for actions** like jumping
5. **Provide Esc key handler** to end the script gracefully

**Reference:** See https://github.com/LavishSoftware/LERN/tree/master/Game for complete keyboard input examples.

### Game Controller Pattern

The Game Controller pattern provides a robust architecture for game-like applications with frame timing, input handling, and lifecycle management.

**Pattern Structure:**

```lavishscript
objectdef gameController
{
    ; Frame timing
    variable float CurrentFrame
    variable float LastFrame
    variable float FrameTime

    ; Game state variables
    variable float SpeedX
    variable float SpeedY
    variable float BaseSpeed=100

    ; Constructor
    method Initialize()
    {
        ; Initialize frame timing
        CurrentFrame:Set[${Script.RunningTime}/1000.0]
        LastFrame:Set[${CurrentFrame}]

        ; Load UI
        LGUI2:LoadPackageFile[game.json]

        ; Take keyboard focus
        LGUI2.Element[game.board]:KeyboardFocus
    }

    ; Destructor
    method Shutdown()
    {
        LGUI2:UnloadPackageFile[game.json]
    }

    ; Called every frame by LGUI2 onRefresh event
    method OnRefresh()
    {
        ; Update timestamp
        CurrentFrame:Set[${Script.RunningTime}/1000.0]

        ; Skip if no time passed
        if ${CurrentFrame.Equal[${LastFrame}]}
            return

        ; Calculate frame time
        FrameTime:Set[${CurrentFrame}-${LastFrame}]

        ; Update game logic
        This:UpdateGame

        ; Update last frame timestamp
        LastFrame:Set[${CurrentFrame}]
    }

    ; Game logic update
    method UpdateGame()
    {
        ; Frame-independent movement: Distance = Speed × Time
        variable float newX=${LGUI2.Element[game.player].X}
        variable float newY=${LGUI2.Element[game.player].Y}

        newX:Inc[ ${SpeedX} * ${This.FrameTime} ]
        newY:Inc[ ${SpeedY} * ${This.FrameTime} ]

        LGUI2.Element[game.player]:SetLocation[${newX}, ${newY}]
    }

    ; Keyboard input handler
    method OnButtonMove()
    {
        switch ${Context.Args[controlName]}
        {
            case Esc
                Script:End
                break
            case Left
                if ${Context.Args[position]}
                    SpeedX:Set[-${BaseSpeed}]
                else
                    SpeedX:Set[0]
                break
            case Right
                if ${Context.Args[position]}
                    SpeedX:Set[${BaseSpeed}]
                else
                    SpeedX:Set[0]
                break
        }
    }
}

variable(global) gameController GameController

function main()
{
    while 1
        waitframe
}
```

**Corresponding JSON:**

```json
{
    "$schema": "http://www.lavishsoft.com/schema/lgui2Package.json",
    "elements": [
        {
            "type": "window",
            "name": "game.window",
            "title": "My Game",
            "content": {
                "type": "panel",
                "name": "game.board",
                "acceptsKeyboardFocus": true,
                "acceptsMouseFocus": true,
                "width": 640,
                "height": 480,
                "children": [
                    {
                        "type": "canvas",
                        "name": "game.player",
                        "width": 48,
                        "height": 48
                    }
                ],
                "eventHandlers": {
                    "onButtonMove": {
                        "type": "method",
                        "object": "GameController",
                        "method": "OnButtonMove"
                    }
                }
            },
            "eventHandlers": {
                "onRefresh": {
                    "type": "method",
                    "object": "GameController",
                    "method": "OnRefresh"
                }
            }
        }
    ]
}
```

**Key Concepts:**

**Frame Timing:**
- `CurrentFrame` - Current time in seconds since script start
- `LastFrame` - Previous frame time
- `FrameTime` - Delta time between frames (usually 0.016 for 60fps)

**Frame-Independent Movement:**

```lavishscript
; WRONG - Frame-dependent (moves faster on high FPS)
newX:Inc[5]

; CORRECT - Frame-independent (consistent speed regardless of FPS)
newX:Inc[ ${Speed} * ${FrameTime} ]
```

**Why Frame-Independent Movement Matters:**
- Consistent behavior across different frame rates
- Professional game development practice
- Prevents speed variations due to system performance

**Lifecycle:**
1. `Initialize()` - Load UI, set up initial state, take keyboard focus
2. `OnRefresh()` - Called every frame, updates game state
3. `UpdateGame()` - Apply game logic, move objects, check collisions
4. `OnButtonMove()` - Handle input events
5. `Shutdown()` - Clean up UI and resources

**Pattern Benefits:**
- Clean separation of concerns
- Reusable template structure
- Professional game loop architecture
- Easy to extend with additional features

**Common Extensions:**

```lavishscript
; Collision detection
member:bool IsOnGround()
{
    variable float Y=${LGUI2.Element[game.player].Y}
    Y:Inc[${LGUI2.Element[game.player].ActualHeight}]

    if ${Y}>=${LGUI2.Element[game.board].ActualHeight}
        return TRUE
    return FALSE
}

; Gravity simulation
method UpdateGame()
{
    ; ... movement code ...

    ; Apply gravity if not on ground
    if !${This.IsOnGround}
        SpeedY:Inc[${BaseSpeed}*5*${This.FrameTime}]
}
```

**Reference:** See https://github.com/LavishSoftware/LERN/tree/master/Game for complete examples including gravity, jumping, and collision detection.

---

## Element Lifecycle and Events

### Element Lifecycle Events

LGUI2 elements go through a well-defined lifecycle from creation to destruction. Understanding these events is crucial for proper initialization and cleanup.

**Lifecycle Event Order:**

1. **onLoad** - Fired when element is first created from JSON
2. **onVisualAttached** - Fired when element is attached to visual tree
3. **onLogicalAttached** - Fired when element is attached to logical tree
4. **Interaction Events** - Mouse/keyboard focus and interaction events
5. **onRendered** - Fired each frame the element is rendered
6. **onRenderedChildren** - Fired after children are rendered
7. **onRefresh** - Fired when element needs to refresh its display
8. **onVisualDetached** - Fired when element is removed from visual tree
9. **onLogicalDetached** - Fired when element is removed from logical tree

**Example: Using Lifecycle Events**

```json
{
    "type": "window",
    "name": "myWindow",
    "eventHandlers": {
        "onLoad": {
            "type": "code",
            "code": "echo Window loaded and ready"
        },
        "onVisualAttached": {
            "type": "method",
            "object": "MyController",
            "method": "OnWindowAttached"
        },
        "onLogicalDetached": {
            "type": "method",
            "object": "MyController",
            "method": "OnWindowClosed"
        }
    }
}
```

**Important Notes:**

- **onLoad** is the earliest reliable event for initialization
- **onVisualAttached** indicates the element is part of the visual hierarchy
- **onRendered** fires every frame - avoid heavy processing in this event
- **Use `onCloseButtonClick`** - This is the correct event for window close button
  - Events like `onClose`, `onClosing`, and `onUnload` do not exist
  - See "Troubleshooting: Window Close Button Event" section for examples

### Complete Event Reference

**Mouse Events:**

- **onMouseMove** - Mouse cursor moves over element
- **onMouseWheel** - Mouse wheel scrolled over element
- **onMouseButtonMove** - Mouse button state changed over element
- **onDragDrop** - Drag and drop operation completed
- **gotMouseOver** - Mouse cursor entered element bounds
- **lostMouseOver** - Mouse cursor left element bounds
- **gotMouseCapture** - Element captured mouse input
- **lostMouseCapture** - Element lost mouse capture
- **gotMouseFocus** - Element received mouse focus
- **lostMouseFocus** - Element lost mouse focus

**Keyboard Events:**

- **onButtonMove** - Keyboard button state changed
- **onAxisMove** - Input axis changed (gamepad)
- **onDpadMove** - D-pad state changed (gamepad)
- **gotKeyboardFocus** - Element received keyboard focus
- **lostKeyboardFocus** - Element lost keyboard focus

**Lifecycle Events:**

- **onLoad** - Element created and initialized
- **onVisualAttached** - Element attached to visual tree
- **onLogicalAttached** - Element attached to logical tree
- **onVisualDetached** - Element removed from visual tree
- **onLogicalDetached** - Element removed from logical tree
- **onRendered** - Element rendered (fires every frame)
- **onRenderedChildren** - Children finished rendering
- **onRefresh** - Element needs display refresh

**Element-Specific Events:**

- **onPress** / **onRelease** - Button pressed/released (buttons)
- **onVisualPress** / **onVisualRelease** - Visual feedback for button press
- **onChecked** / **onUnchecked** / **onIndeterminate** - Checkbox state changed
- **onSelected** / **onDeselected** - Item selection changed
- **onSelectionChanged** - Selection changed (listbox, combobox)
- **onTextChanged** - Text modified (textbox)
- **onValueChanged** - Value changed (slider, scrollbar, inputpicker)
- **onExpanded** / **onCollapsed** - Expander state changed
- **onShowExpander** / **onHideExpander** - Expander visibility changed

**Window-Specific Events:**

- **onCloseButtonClick** - Close button (X) clicked
- **onShadeButtonClick** - Minimize/shade button clicked
- **onHeaderMouseMove** - Mouse moved over title bar
- **onHeaderMouseButtonMove** - Mouse button on title bar
- **onHeaderLostMouseFocus** - Title bar lost mouse focus
- **onHeaderButtonMove** - Keyboard button on title bar

**Tab Control Events:**

- **onTabHeaderMouseButtonMove** - Mouse button on tab header

**List/Item Events:**

- **onItemSelected** / **onItemDeselected** - Item selection state
- **onItemMouseButtonMove** - Mouse button on item
- **onItemMouse1DoubleClick** - Double-click on item
- **onItemExpanderPress** - Tree item expander clicked

**Scrollbar Events:**

- **onDecMouseButtonMove** - Decrease button mouse event
- **onIncMouseButtonMove** - Increase button mouse event
- **onHandleMouseMove** - Slider handle mouse move
- **onHandleMouseButtonMove** - Slider handle mouse button
- **onHorizontalValueChanged** / **onVerticalValueChanged** - Scrollviewer scroll

**Popup/Combobox Events:**

- **onPopupButtonMouseButtonMove** - Popup dropdown button clicked
- **onListBoxItemSelected** / **onListBoxItemDeselected** - Combobox list selection
- **onListBoxSelectionChanged** - Combobox selection changed
- **onSuggestionSelected** - Autocomplete suggestion selected
- **autoComplete** - Tab pressed in autocomplete
- **refreshSuggestions** / **hideSuggestions** - Autocomplete popup management

**FilePicker Events:**

- **setPathToParent** - Navigate to parent directory
- **tryFinalize** - Attempt to finalize file selection
- **onFileListUpdated** - File list refreshed

**PageControl Events:**

- **onNextPage** / **onPreviousPage** - Navigate pages
- **onFinish** - Finish button clicked
- **onShowNext** / **onHideNext** - Show/hide next button
- **onShowPrevious** / **onHidePrevious** - Show/hide previous button
- **onShowFinish** / **onHideFinish** - Show/hide finish button

**Context Menu Events:**

- **onContextMenuItemSelected** - Context menu item selected

**Special Events:**

- **onScreenRenderedChildren** - Screen finished rendering children
- **onOK** / **onCancel** - Confirmation window response
- **onTabPress** - Tab key pressed (textbox/autocomplete)

### Element Properties Reference

All LGUI2 elements inherit common base properties:

**Positioning:**

```json
{
    "x": 100,              // Pixel-based X coordinate
    "y": 50,               // Pixel-based Y coordinate
    "xFactor": 0.5,        // Factor-based X (relative to parent width)
    "yFactor": 0.25        // Factor-based Y (relative to parent height)
}
```

**Sizing:**

```json
{
    "width": 400,          // Pixel-based width
    "height": 300,         // Pixel-based height
    "widthFactor": 0.5,    // Factor-based width (50% of parent)
    "heightFactor": 0.75,  // Factor-based height (75% of parent)
    "minSize": [200, 150], // Minimum [width, height]
    "maxSize": [800, 600]  // Maximum [width, height]
}
```

**Alignment:**

```json
{
    "horizontalAlignment": "left",    // left, right, center, stretch
    "verticalAlignment": "top"        // top, bottom, center, stretch
}
```

**Spacing:**

```json
{
    "margin": [10, 10, 10, 10],  // [left, top, right, bottom]
    "padding": [5, 5, 5, 5]      // Internal padding
}
```

**Visual:**

```json
{
    "opacity": 0.8,           // 0.0 (transparent) to 1.0 (opaque)
    "strata": 0.5,            // Z-order priority (default 0.5)
    "visibility": "visible"   // visible, hidden, collapsed
}
```

**Focus:**

```json
{
    "acceptsKeyboardFocus": true,
    "acceptsMouseFocus": true
}
```

**Other Common Properties:**

```json
{
    "name": "uniqueName",        // Required for script access
    "tooltip": "Help text",
    "contextMenu": "menuName",
    "font": {
        "height": 16,
        "color": "#FFFFFF",
        "family": "Arial"
    }
}
```

### Metadata Properties

Properties prefixed with underscore (`_`) are stored as metadata and used by parent layout containers.

**Common Metadata Properties:**

```json
{
    "_dock": "left",           // For dockpanel: left, right, top, bottom, fill
    "_row": 0,                 // For table: row number
    "_column": 1,              // For table: column number
    "_rowSpan": 2,             // For table: spans multiple rows
    "_columnSpan": 3           // For table: spans multiple columns
}
```

**Example: Using Metadata in Dockpanel**

```json
{
    "type": "dockpanel",
    "children": [
        {
            "_dock": "top",
            "type": "textblock",
            "text": "Header",
            "height": 40
        },
        {
            "_dock": "left",
            "type": "panel",
            "width": 200
        },
        {
            "_dock": "fill",
            "type": "panel"
        }
    ]
}
```

**Important:** Metadata properties are NOT directly accessible via `Element.Property` syntax. They are consumed by the parent container during layout.

### Responsive Layout with Factors

Use factor-based sizing for responsive layouts that adapt to parent size changes:

```json
{
    "type": "window",
    "width": 800,
    "height": 600,
    "content": {
        "type": "panel",
        "children": [
            {
                "type": "textblock",
                "text": "Takes 50% of parent width",
                "widthFactor": 0.5,
                "heightFactor": 0.1,
                "xFactor": 0.25,     // Centered horizontally
                "yFactor": 0.05
            }
        ]
    }
}
```

**Benefits of Factor-based Sizing:**

- Automatically adapts to parent resizing
- Maintains proportions across different screen sizes
- Simplifies responsive UI design
- Can be combined with pixel-based constraints (minSize, maxSize)

### Element Inheritance Hierarchy

Understanding the inheritance hierarchy helps determine which properties and methods are available:

```
lgui2element (base)
├── lgui2contentbase
│   ├── lgui2headeredcontentbase
│   │   └── window, expander, tab
│   └── button, panel, scrollviewer
└── lgui2itemlist
    └── listbox, combobox, itemview
```

**Special Base Types:**

- **lgui2bordered** - Elements with border support
- **lgui2interactive** - Elements that accept user input
- **lgui2textbase** - Elements that display text

Each element inherits all properties, methods, and events from its parent types.

---

## Script-to-UI Interaction

### Loading and Unloading Packages

**Load a package:**

```lavishscript
LGUI2:LoadPackageFile[myui.json]
```

**Unload a package:**

```lavishscript
LGUI2:UnloadPackageFile[myui.json]
```

### Accessing Elements

**Find an element by name:**

```lavishscript
LGUI2.Element[mywindow]
```

**Check if element exists:**

```lavishscript
if ${LGUI2.Element[mywindow](exists)}
{
    echo Window exists!
}
```

### Manipulating Elements

**Show/Hide:**

```lavishscript
LGUI2.Element[mywindow]:Show
LGUI2.Element[mywindow]:Hide
```

**Move window:**

```lavishscript
LGUI2.Element[mywindow]:SetLocation[100, 100]
```

**Resize window:**

```lavishscript
LGUI2.Element[mywindow]:SetSize[400, 300]
```

**Set text:**

```lavishscript
LGUI2.Element[statusText]:SetText["New status message"]
```

**Check checkbox state:**

```lavishscript
if ${LGUI2.Element[myCheckbox].Checked}
{
    echo Checkbox is checked
}
```

**Set checkbox state:**

```lavishscript
LGUI2.Element[myCheckbox]:SetChecked[TRUE]
LGUI2.Element[myCheckbox]:SetChecked[FALSE]
```

**Get selected item from combobox:**

```lavishscript
echo Selected: ${LGUI2.Element[myComboBox].SelectedItem.Index}
echo Data: ${LGUI2.Element[myComboBox].SelectedItem.Data}
```

**Programmatically set event handler:**

```lavishscript
; Set a simple code event handler
LGUI2.Element[myButton]:SetEventHandler[onPress,"{\"type\":\"code\",\"code\":\"echo Button pressed!\"}"]

; Set a method event handler
LGUI2.Element[myButton]:SetEventHandler[onPress,"{\"type\":\"method\",\"object\":\"MyController\",\"method\":\"OnButtonPress\"}"]
```

**Fire event handler manually:**

```lavishscript
; Trigger an event handler on an element
LGUI2.Element[myButton]:FireEventHandler[onPress]

; Common pattern: Global event broadcasting
LGUI2.Element[events]:FireEventHandler[onDataUpdated]
```

**SetEventHandler Usage Notes:**

- **Syntax:** `Element:SetEventHandler[eventName, jsonString]`
- The second parameter must be a valid JSON event handler definition (as a string)
- Useful for dynamic event binding at runtime
- Can override event handlers defined in the JSON package
- **Limitation:** Does not work for window close events (`onClose`, `onUnload`)

**FireEventHandler Usage Notes:**

- **Syntax:** `Element:FireEventHandler[eventName]`
- Manually triggers the specified event handler on the element
- Commonly used with global event panels for broadcasting state changes
- See "Global Event Broadcasting Pattern" in Best Practices section

### Skins

**Push a skin:**

```lavishscript
LGUI2:PushSkin[myskin]
```

**Pop a skin:**

```lavishscript
LGUI2:PopSkin[myskin]
```

Skins provide consistent styling across packages.

### Persisting Window Position

From the production `Object_LGUI2Helper.iss`:

```lavishscript
method Save_WindowInformation(string _WindowName)
{
    variable jsonvalue MyRef

    ; Get window position
    MyRef:SetInteger[x, ${LGUI2.Element["${_WindowName}"].X}]
    MyRef:SetInteger[y, ${LGUI2.Element["${_WindowName}"].Y}]

    ; Save to settings
    ; (implementation depends on your settings system)
}

method Load_WindowInformation(string _WindowName)
{
    variable jsonvalue MyRef

    ; Load from settings
    ; (implementation depends on your settings system)

    ; Set window position
    LGUI2.Element["${_WindowName}"]:SetLocation[${MyRef.Get["x"]}, ${MyRef.Get["y"]}]
}
```

---

## Complete Working Examples

### Example 1: Simple Status Window

**status.json:**

```json
{
    "$schema": "http://www.lavishsoft.com/schema/lgui2Package.json",
    "elements": [
        {
            "type": "window",
            "title": "Character Status",
            "name": "status.window",
            "width": 250,
            "height": 150,
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
                            "pullFormat": "Level: ${Me.Level}"
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

**status.iss:**

```lavishscript
objectdef status_controller
{
    method Initialize()
    {
        LGUI2:LoadPackageFile[status.json]
    }

    method Shutdown()
    {
        LGUI2:UnloadPackageFile[status.json]
    }
}

variable(global) status_controller StatusController

function main()
{
    while 1
        waitframe
}
```

### Example 2: Combat Control Panel

**combat.json:**

```json
{
    "$schema": "http://www.lavishsoft.com/schema/lgui2Package.json",
    "elements": [
        {
            "type": "window",
            "title": "Combat Controls",
            "name": "combat.window",
            "width": 300,
            "height": 200,
            "content": {
                "type": "stackpanel",
                "orientation": "vertical",
                "children": [
                    {
                        "type": "checkbox",
                        "name": "autoAttack.checkbox",
                        "checkedBinding": "CombatController.AutoAttackEnabled",
                        "content": "Auto-Attack Enabled"
                    },
                    {
                        "type": "checkbox",
                        "name": "autoLoot.checkbox",
                        "checkedBinding": "CombatController.AutoLootEnabled",
                        "content": "Auto-Loot Enabled"
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
                                "method": "OnAttackPressed"
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
                                "method": "OnStopPressed"
                            }
                        }
                    },
                    {
                        "type": "panel",
                        "height": 10
                    },
                    {
                        "type": "textblock",
                        "name": "status.text",
                        "textBinding": {
                            "pullFormat": "Status: ${CombatController.StatusText}"
                        }
                    }
                ]
            }
        }
    ]
}
```

**combat.iss:**

```lavishscript
objectdef combat_controller
{
    variable bool AutoAttackEnabled = FALSE
    variable bool AutoLootEnabled = FALSE
    variable string StatusText = "Ready"

    method Initialize()
    {
        LGUI2:LoadPackageFile[combat.json]
    }

    method Shutdown()
    {
        LGUI2:UnloadPackageFile[combat.json]
    }

    method OnAttackPressed()
    {
        echo Attacking!
        This.StatusText:Set["Attacking..."]

        if !${Target(exists)}
        {
            This.StatusText:Set["No target!"]
            return
        }

        ; Start combat
        Me.Ability[1]:Use
        This.StatusText:Set["In combat"]
    }

    method OnStopPressed()
    {
        echo Stopping combat
        This.StatusText:Set["Stopped"]
        ; Stop combat logic here
    }
}

variable(global) combat_controller CombatController

function main()
{
    while 1
        waitframe
}
```

### Example 3: Dynamic Target List

**targets.json:**

```json
{
    "$schema": "http://www.lavishsoft.com/schema/lgui2Package.json",
    "elements": [
        {
            "type": "window",
            "title": "Target Selector",
            "name": "targets.window",
            "width": 350,
            "height": 400,
            "content": {
                "type": "dockpanel",
                "children": [
                    {
                        "_dock": "top",
                        "type": "textblock",
                        "text": "Select a target:",
                        "padding": 5
                    },
                    {
                        "_dock": "top",
                        "type": "listbox",
                        "name": "targetList",
                        "height": 300,
                        "horizontalAlignment": "stretch",
                        "itemsBinding": {
                            "pullFormat": "${TargetsController.GetNearbyActors}"
                        },
                        "eventHandlers": {
                            "onSelectionChanged": {
                                "type": "method",
                                "object": "TargetsController",
                                "method": "OnTargetSelected"
                            }
                        }
                    },
                    {
                        "_dock": "bottom",
                        "type": "button",
                        "content": "Refresh List",
                        "eventHandlers": {
                            "onPress": {
                                "type": "method",
                                "object": "TargetsController",
                                "method": "OnRefreshPressed"
                            }
                        }
                    }
                ]
            }
        }
    ]
}
```

**targets.iss:**

```lavishscript
objectdef targets_controller
{
    method Initialize()
    {
        LGUI2:LoadPackageFile[targets.json]
    }

    method Shutdown()
    {
        LGUI2:UnloadPackageFile[targets.json]
    }

    member:string GetNearbyActors()
    {
        variable string result = "["
        variable int count = 0
        variable index:actor actorList
        variable iterator iter

        ; Get nearby NPCs
        EQ2:QueryActors[actorList, "Type = NPC && Distance < 20"]

        actorList:GetIterator[iter]
        if ${iter:First(exists)}
        {
            do
            {
                if ${count} > 0
                    result:Concat[","]

                result:Concat["{\"type\":\"textblock\",\"text\":\"${iter.Value.Name.Escape}\"}"]
                count:Inc
            }
            while ${iter:Next(exists)}
        }

        result:Concat["]"]
        return "${result}"
    }

    method OnTargetSelected()
    {
        echo Selected target index: ${Context.Source.SelectedItem.Index}
    }

    method OnRefreshPressed()
    {
        echo Refreshing target list...
        ; The data binding will automatically refresh
    }
}

variable(global) targets_controller TargetsController

function main()
{
    while 1
        waitframe
}
```

---

## Best Practices

### Organization

1. **One package per major UI component**
   - Don't put all UIs in one giant JSON file
   - Separate: `combat.json`, `inventory.json`, `settings.json`

2. **Use consistent naming conventions**
   - Element names: `combat.window`, `attack.button`, `status.text`
   - Controller names: `CombatController`, `InventoryController`

3. **Group related elements in containers**
   - Use StackPanels for vertical/horizontal groups
   - Use DockPanels for structured layouts
   - Use Tables for grid layouts

### Performance

1. **Use data binding instead of manual updates**
   ```json
   // GOOD - Automatic updates
   "textBinding": {"pullFormat": "Health: ${Me.Health}"}
   
   // BAD - Manual updates required
   "text": "Health: 100"
   ```

2. **Use `pullOnce: true` for static data**
   ```json
   "itemsBinding": {
       "pullFormat": "${MyController.GetStaticData}",
       "pullOnce": true
   }
   ```

3. **Avoid complex computations in pullFormat**
   ```lavishscript
   // GOOD - Compute in controller member
   member:string GetFormattedHealth()
   {
       return "${Me.Health}/${Me.MaxHealth} (${Math.Calc[${Me.Health}*100/${Me.MaxHealth}]}%)"
   }
   ```

### Event Handlers

1. **Use method event handlers for complex logic**
   ```json
   // GOOD - Complex logic in controller
   "eventHandlers": {
       "onPress": {
           "type": "method",
           "object": "Controller",
           "method": "HandleComplexAction"
       }
   }
   
   // BAD - Complex inline code
   "eventHandlers": {
       "onPress": {
           "type": "code",
           "code": "if ${condition} { ... long code ... }"
       }
   }
   ```

2. **Use inline code for simple actions**
   ```json
   // GOOD - Simple echo
   "eventHandlers": {
       "onPress": {
           "type": "code",
           "code": "echo Button clicked!"
       }
   }
   ```

### Templates

1. **Create templates for repeated UI patterns**
   ```json
   "templates": {
       "primaryButton": {
           "backgroundBrush": {"color": "#0000ff"},
           "padding": 10,
           "font": {"bold": true}
       }
   }
   ```

2. **Use templates for consistent styling**

### Settings Persistence

**CRITICAL BEST PRACTICE:** Any script with a GUI window **must persist the window position** so users don't have to reposition it every time they run the script.

#### Complete Window Position Pattern

This pattern uses LavishSettings for configuration persistence.

**Setup (script variables):**
```lavishscript
variable settingsetref Config
variable filepath ConfigFile="${LavishScript.HomeDirectory}/Scripts/MyScript/Settings.xml"
```

**LoadSettings() - Restore window position:**
```lavishscript
function LoadSettings()
{
    ; Initialize LavishSettings
    LavishSettings:AddSet[MyScript]
    LavishSettings[MyScript]:Clear
    LavishSettings[MyScript]:AddSet[Config]
    Config:Set[${LavishSettings[MyScript].FindSet[Config]}]
    LavishSettings[MyScript]:Import["${ConfigFile}"]

    ; Load other settings...
    variable bool SomeSetting
    SomeSetting:Set[${Config.FindSetting[SomeSetting,FALSE]}]

    ; Load window position (use -1 as default for "not set")
    variable int WindowX
    variable int WindowY
    WindowX:Set[${Config.FindSetting[WindowX,-1]}]
    WindowY:Set[${Config.FindSetting[WindowY,-1]}]

    ; Apply saved window position (only if previously saved)
    if ${WindowX} >= 0 && ${WindowY} >= 0
    {
        LGUI2.Element[MyScript.Window]:SetLocation[${WindowX}, ${WindowY}]
    }
    ; Otherwise window uses default position from JSON

    return
}
```

**SaveSettings() - Save window position:**
```lavishscript
function SaveSettings()
{
    ; Save other settings...
    Config.FindSetting[SomeSetting]:Set[${SomeSetting}]

    ; Save current window position
    Config.FindSetting[WindowX]:Set[${LGUI2.Element[MyScript.Window].X}]
    Config.FindSetting[WindowY]:Set[${LGUI2.Element[MyScript.Window].Y}]

    ; Export to file
    LavishSettings[MyScript]:Export["${ConfigFile}"]
    return
}
```

**Shutdown atom - Save on close:**
```lavishscript
atom(script) Shutdown()
{
    call SaveSettings
    Script:End
}
```

**JSON - Window with close handler:**
```json
{
    "type": "window",
    "name": "MyScript.Window",
    "title": "My Script",
    "x": 1480,
    "y": 960,
    "width": 800,
    "height": 600,
    "eventHandlers": {
        "onCloseButtonClick": {
            "type": "code",
            "code": "Script[MyScript]:QueueCommand[call Shutdown]"
        }
    }
}
```

**atexit() - Save on script end:**
```lavishscript
function atexit()
{
    echo Ending MyScript...
    call SaveSettings
}
```

**Why This Pattern?**

1. **User Experience**: Users expect windows to "remember" where they placed them
2. **Multi-monitor**: Critical for users with multiple displays
3. **First run**: Uses JSON defaults (-1 check ensures this)
4. **Shutdown paths**: Saves on close button AND script end
5. **Production tested**: Based on EQ2BotCommander implementation

**Reference Implementation:**
- [EQ2BotCommander.iss](https://github.com/isxGames/isxScripts/blob/master/EverQuest2/Scripts/EQ2BotCommander.iss) (lines 83-133, 548-573)
- Complete working example with window position persistence

#### Other Settings Persistence

**Checkbox states:**
```lavishscript
function SaveSettings()
{
    Config.FindSetting[AutoAttack]:Set[${AutoAttackEnabled}]
}

function LoadSettings()
{
    AutoAttackEnabled:Set[${Config.FindSetting[AutoAttack,FALSE]}]
    LGUI2.Element[AutoAttackCheckbox]:SetChecked[${AutoAttackEnabled}]
}
```

**Textbox values:**
```lavishscript
function SaveSettings()
{
    variable string TextValue
    TextValue:Set[${LGUI2.Element[MyTextbox].Text}]
    Config.FindSetting[SavedText]:Set[${TextValue}]
}

function LoadSettings()
{
    variable string TextValue
    TextValue:Set[${Config.FindSetting[SavedText,"Default Value"]}]
    LGUI2.Element[MyTextbox]:SetText[${TextValue}]
}
```

### Controller Pattern

1. **Use objectdef for controllers**
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
   ```

2. **Store UI state in controller variables**
   ```lavishscript
   variable bool SettingEnabled = FALSE
   variable string StatusText = "Ready"
   ```

3. **Use descriptive method names**
   ```lavishscript
   method OnAttackButtonPressed()
   method OnTargetSelected()
   method OnRefreshRequested()
   ```

### Advanced Controller Patterns

#### Weakref for Settings Management

Use **weakref** to maintain safe references to JSON settings objects without ownership:

**Controller with weakref:**

```lavishscript
objectdef myui_controller
{
    variable weakref SelectedProfile
    variable weakref EditingLayout

    method SetSelectedProfile(uint id)
    {
        ; Set reference to settings object
        SelectedProfile:SetReference["Settings.Profiles.Get[profiles,${id}]"]

        ; Fire event to update UI
        LGUI2.Element[events]:FireEventHandler[onSelectedProfileUpdated]
    }

    method OnDeleteProfile()
    {
        ; Validate reference exists before using
        if !${SelectedProfile.Reference(exists)}
        {
            echo No profile selected
            return
        }

        ; Access weakref data
        variable string profileName = "${SelectedProfile.Get[name]~}"
        Settings:EraseProfile[${profileName}]

        ; Clear reference
        SelectedProfile:SetReference[NULL]

        ; Notify UI
        LGUI2.Element[events]:FireEventHandler[onProfilesUpdated]
    }

    method OnSaveProfile()
    {
        if !${SelectedProfile.Reference(exists)}
            return

        ; Modify via weakref
        SelectedProfile:SetString[lastModified,"${Time.Time24}"]
        Settings:Save

        LGUI2.Element[events]:FireEventHandler[onSelectedProfileUpdated]
    }
}
```

**Weakref Benefits:**

- **Safe references** - Automatically detects if object no longer exists
- **No ownership** - Object can be deleted without affecting weakref
- **JSON object access** - Direct access to Get[], Set[], etc. methods
- **Null handling** - Can check `.Reference(exists)` before use

#### Global Event Broadcasting Pattern

Create a **global event hub** for reactive UI updates:

**JSON Package with Event Element:**

```json
{
    "$schema": "http://www.lavishsoft.com/schema/lgui2Package.json",
    "elements": [
        {
            "type": "panel",
            "visibility": "hidden",
            "name": "events"
        },
        {
            "type": "window",
            "title": "Profile Manager",
            "name": "profileWindow",
            "content": {
                "type": "listbox",
                "name": "profileList",
                "itemsBinding": {
                    "pullFormat": "${MyController.GetProfiles}",
                    "pullHook": {
                        "elementName": "events",
                        "flags": "global",
                        "event": "onProfilesUpdated"
                    }
                }
            }
        },
        {
            "type": "window",
            "title": "Profile Editor",
            "name": "editorWindow",
            "content": {
                "type": "objectview",
                "objectBinding": {
                    "pullFormat": "${MyController.SelectedProfile}",
                    "pullHook": {
                        "elementName": "events",
                        "flags": "global",
                        "event": "onSelectedProfileUpdated"
                    }
                },
                "properties": []
            }
        }
    ]
}
```

**Controller Broadcasting Events:**

```lavishscript
objectdef myui_controller
{
    ; Weakref to selected profile
    variable weakref SelectedProfile

    method OnNewProfile()
    {
        ; Create new profile in settings
        Settings:AddProfile["New Profile"]

        ; Broadcast event - ALL UI elements with pullHook listening to
        ; onProfilesUpdated will refresh automatically
        LGUI2.Element[events]:FireEventHandler[onProfilesUpdated]
    }

    method OnCopyProfile()
    {
        if !${SelectedProfile.Reference(exists)}
            return

        Settings:CopyProfile[${SelectedProfile.Get[name]~}]

        ; Refresh profile list
        LGUI2.Element[events]:FireEventHandler[onProfilesUpdated]
    }

    method OnDeleteProfile()
    {
        if !${SelectedProfile.Reference(exists)}
            return

        Settings:EraseProfile[${SelectedProfile.Get[name]~}]
        SelectedProfile:SetReference[NULL]

        ; Multiple events can be fired
        LGUI2.Element[events]:FireEventHandler[onProfilesUpdated]
        LGUI2.Element[events]:FireEventHandler[onSelectedProfileUpdated]
    }

    method SetSelectedProfile(uint id)
    {
        SelectedProfile:SetReference["Settings.Profiles.Get[profiles,${id}]"]

        ; Any UI bound to onSelectedProfileUpdated refreshes
        LGUI2.Element[events]:FireEventHandler[onSelectedProfileUpdated]
    }

    member:string GetProfiles()
    {
        ; Return JSON array of profiles
        return "${Settings.Profiles.AsJSON~}"
    }
}
```

**Event Broadcasting Pattern Summary:**

1. **Create hidden event panel** - `"type": "panel", "visibility": "hidden", "name": "events"`
2. **Use pullHook in bindings** - Elements listen for specific events
3. **Fire events from controller** - `LGUI2.Element[events]:FireEventHandler[eventName]`
4. **All listeners refresh** - Any UI with matching pullHook updates automatically

**Common Event Names:**

| Event | When to Fire | Listening UI |
|-------|-------------|--------------|
| `onProfilesUpdated` | Profile added/deleted/modified | Profile list, profile dropdown |
| `onSelectedProfileUpdated` | Selection changes | Property editor, detail view |
| `onSettingsUpdated` | Settings saved | All settings-bound UI |
| `onDataRefreshed` | Background data updated | Charts, statistics displays |

This pattern provides **reactive UI** without tight coupling between controller and UI elements.

---

## Troubleshooting

### Common JSON Errors

**Error: Invalid JSON syntax**

```
Check for:
- Missing commas between properties
- Missing quotes around strings
- Trailing commas (not allowed in JSON)
```

**Fix:**

```json
// BAD
{
    "type": "window"
    "title": "My Window",  // Missing comma above
}

// GOOD
{
    "type": "window",
    "title": "My Window"
}
```

### Package Not Loading

**Symptom:** `LGUI2:LoadPackageFile[myui.json]` doesn't show window

**Checks:**

1. **Verify file path** - Relative to script location
   ```lavishscript
   echo ${Script.CurrentDirectory}
   LGUI2:LoadPackageFile[${Script.CurrentDirectory}/myui.json]
   ```

2. **Check JSON syntax** - Use a JSON validator

3. **Verify schema** - Ensure schema URL is correct
   ```json
   "$schema": "http://www.lavishsoft.com/schema/lgui2Package.json"
   ```

### Element Not Found

**Symptom:** `${LGUI2.Element[mywindow](exists)}` returns FALSE

**Checks:**

1. **Verify element name matches JSON**
   ```json
   "name": "mywindow"  // Must match exactly
   ```

2. **Check if package is loaded**
   ```lavishscript
   echo ${LGUI2.Package[myui.json](exists)}
   ```

3. **Wait for load to complete**
   ```lavishscript
   LGUI2:LoadPackageFile[myui.json]
   wait 5 ${LGUI2.Element[mywindow](exists)}
   ```

### Data Binding Not Updating

**Symptom:** UI shows stale data

**Checks:**

1. **Verify pullFormat syntax**
   ```json
   "textBinding": {
       "pullFormat": "Health: ${Me.Health}"  // Correct variable reference
   }
   ```

2. **Check controller member exists**
   ```lavishscript
   member:string GetHealth()
   {
       return "${Me.Health}"
   }
   ```

3. **Ensure data is actually changing**
   ```lavishscript
   echo ${Me.Health}  // Verify the value changes
   ```

### Event Handler Not Firing

**Symptom:** Clicking button does nothing

**Checks:**

1. **Verify event handler syntax**
   ```json
   "eventHandlers": {
       "onPress": {  // Must be valid event name
           "type": "method",
           "object": "MyController",
           "method": "OnPressed"
       }
   }
   ```

2. **Check controller object exists**
   ```lavishscript
   variable(global) mycontroller MyController  // Must be global
   ```

3. **Verify method exists**
   ```lavishscript
   method OnPressed()
   {
       echo Button was pressed!
   }
   ```

4. **Check for typos in object/method names**

### Window Not Visible

**Symptom:** Package loads but window doesn't appear

**Checks:**

1. **Check if window is minimized**
   ```lavishscript
   echo ${LGUI2.Element[mywindow].Minimized}
   ```

2. **Check window visibility**
   ```lavishscript
   echo ${LGUI2.Element[mywindow].Visible}
   LGUI2.Element[mywindow]:Show
   ```

3. **Check window position** (may be off-screen)
   ```lavishscript
   LGUI2.Element[mywindow]:SetLocation[100, 100]
   ```

### Window Close Button Event

**Correct Event:** Use `onCloseButtonClick` to handle window close button clicks

The window close button (X) triggers the `onCloseButtonClick` event. This is the proper way to handle window closing in LGUI2.

**Solution:**

```json
{
    "type": "window",
    "name": "mywindow",
    "title": "My Window",
    "eventHandlers": {
        "onCloseButtonClick": {
            "type": "code",
            "code": "Script[myscript]:QueueCommand[Shutdown]"
        }
    }
}
```

**Script Implementation:**

```lavishscript
atom(script) Shutdown()
{
    call SaveSettings
    Script:End
}

function main()
{
    LGUI2:LoadPackageFile[myui.json]
    wait 10 ${LGUI2.Element[mywindow](exists)}

    while 1
        waitframe
}
```

**Why `QueueCommand`?**
- The event handler runs in the UI thread
- `QueueCommand` safely schedules the shutdown for the script thread
- Prevents potential threading issues

**Related Window Events:**
- `onCloseButtonClick` - Close button (X) clicked
- `onShadeButtonClick` - Minimize/shade button clicked
- `onHeaderMouseMove` - Mouse moved over title bar
- `onHeaderMouseButtonMove` - Mouse button pressed on title bar
- `onHeaderLostMouseFocus` - Title bar lost mouse focus

**Note:** Events like `onClose`, `onClosing`, and `onUnload` do not exist in LGUI2. Always use `onCloseButtonClick` for the close button.

---

## Quick Reference

### Loading/Unloading

```lavishscript
; Load package
LGUI2:LoadPackageFile[myui.json]

; Unload package
LGUI2:UnloadPackageFile[myui.json]

; Load with skin
LGUI2:PushSkin[myskin]
LGUI2:LoadPackageFile[myui.json]
LGUI2:PopSkin[myskin]
```

### Element Access

```lavishscript
; Get element
${LGUI2.Element[elementName]}

; Check existence
${LGUI2.Element[elementName](exists)}

; Common operations
LGUI2.Element[mywindow]:Show
LGUI2.Element[mywindow]:Hide
LGUI2.Element[mywindow]:SetLocation[x, y]
LGUI2.Element[mywindow]:SetSize[width, height]
LGUI2.Element[mytext]:SetText["New text"]
LGUI2.Element[mycheckbox]:SetChecked[TRUE]
```

### Data Binding Shortcuts

```json
// Text binding
"textBinding": {"pullFormat": "${Me.Health}"}

// Checkbox binding (bidirectional)
"checkedBinding": "MyController.Enabled"

// Items binding
"itemsBinding": {"pullFormat": "${MyController.GetItems}"}
```

### Event Handler Shortcuts

```json
// Inline code
"eventHandlers": {
    "onPress": {"type": "code", "code": "echo Clicked!"}
}

// Controller method
"eventHandlers": {
    "onPress": {"type": "method", "object": "Ctrl", "method": "OnClick"}
}

// Audio
"eventHandlers": {
    "onPress": {"type": "audio", "voiceName": "ui", "streamName": "click"}
}
```

---

## Next Steps

1. **Read the migration guide** - [11_LavishGUI1_to_LavishGUI2_Migration.md](11_LavishGUI1_to_LavishGUI2_Migration.md) if you have existing LGUI1 scripts

2. **Add UI scaling** - See [12_LGUI2_Scaling_System.md](12_LGUI2_Scaling_System.md) to make your UI scale to different screen resolutions

3. **Study complete examples** - See the example scripts in the documentation (e.g., EQ2BotCommander_LGUI2.iss)

4. **Experiment with the schema** - Use an IDE with JSON schema support for autocomplete

5. **Build incrementally** - Start with a simple window and add features one at a time

6. **Review production code** - Study `Object_LGUI2Helper.iss` and other production scripts

---

## Additional Resources

- **LGUI2 Scaling System:** [12_LGUI2_Scaling_System.md](12_LGUI2_Scaling_System.md) - Add dynamic UI scaling
- **Migration Guide:** [11_LavishGUI1_to_LavishGUI2_Migration.md](11_LavishGUI1_to_LavishGUI2_Migration.md) - Migrate from LGUI1
- **LavishGUI 2 Official Wiki:** https://www.lavishsoft.com/wiki/index.php/LavishGUI_2
- **LERN Examples:** https://github.com/LavishSoftware/LERN/tree/master/LGUI2
- **JSON Schema:** http://www.lavishsoft.com/schema/lgui2Package.json

---

**This guide was created through comprehensive analysis of:**
- 60+ LERN/LGUI2 example files: https://github.com/LavishSoftware/LERN/tree/master/LGUI2 (.json, .iss, .md)
- 12 LERN/Game example files: https://github.com/LavishSoftware/LERN/tree/master/Game (canvas, audio, game controller patterns)
- Official LavishGUI 2 documentation and wiki
- Production scripts (Object_LGUI2Helper.iss, EQ2BotCommander_LGUI2.iss)
- Complete coverage of all element types, events, and patterns
- Real-world migration experience (EQ2BotCommander LGUI1→LGUI2)

**Ready to migrate from LavishGUI 1? See:** [11_LavishGUI1_to_LavishGUI2_Migration.md](11_LavishGUI1_to_LavishGUI2_Migration.md)

**Want to add UI scaling? See:** [12_LGUI2_Scaling_System.md](12_LGUI2_Scaling_System.md)

---

*Last Updated: 2025-10-25*
*LavishGUI 2 Version: 2023+*
