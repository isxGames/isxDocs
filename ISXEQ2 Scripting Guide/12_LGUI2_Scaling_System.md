# LavishGUI 2 UI Scaling System

**Purpose:** Add dynamic UI scaling to LGUI2-based scripts
**Created:** 2025-10-25
**Pattern:** Reusable library for consistent UI scaling across scripts

---

## Table of Contents

1. [Overview](#overview)
2. [Quick Start](#quick-start)
3. [How Scaling Works](#how-scaling-works)
4. [Implementation Details](#implementation-details)
5. [Real-World Example: EQ2BotCommander](#real-world-example-eq2botcommander)
6. [Troubleshooting](#troubleshooting)
7. [Creating Scalable Title Bars](#creating-scalable-title-bars)
8. [Advanced Topics](#advanced-topics)

---

## Overview

### What is LGUI2Scaling?

**LGUI2Scaling.iss** is a reusable LavishScript library that provides **runtime UI scaling** for LGUI2 JSON interfaces. It allows users to scale entire UIs (windows, buttons, fonts, positions) by a single scale factor without manually editing JSON files.

### Why Use UI Scaling?

**Problem:** Players have different screen resolutions and DPI settings. A UI designed for 1920x1080 may be too small on 4K displays or too large on 1366x768 screens.

**Solution:** The scaling system allows users to easily adjust UI size:
- **2.0x scale** = Double size (4K displays)
- **1.0x scale** = Original size (default)
- **0.5x scale** = Half size (compact mode)

### Key Features

- ✅ **Single-file inclusion** - Just `#include` and use
- ✅ **One-line configuration** - Set `uiScale` variable
- ✅ **Preserves layouts** - Maintains proportions and spacing
- ✅ **Font scaling** - Automatically scales all text
- ✅ **Window position preservation** - Screen position stays constant
- ✅ **Percentage conversion** - Handles percentage-based positioning
- ✅ **Debug mode** - Optional logging for troubleshooting
- ✅ **Reusable** - Works with any LGUI2 JSON UI

---

## Quick Start

### Step 1: Copy the Scaling Library

Place `LGUI2Scaling.iss` in your scripts directory:

**GitHub:** [LGUI2Scaling.iss](https://github.com/isxGames/isxScripts/blob/master/EverQuest2/Scripts/LGUI2Scaling.iss)

### Step 2: Include in Your Script

At the top of your script:

```lavishscript
#include ${LavishScript.HomeDirectory}/Scripts/LGUI2Scaling.iss
```

### Step 3: Add Scaling Logic to main()

```lavishscript
function main()
{
    ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
    ;;;; GUI Scaling
    ;; Set your desired scale factor (1.0 = no scaling, 2.0 = 2x larger, 0.5 = half size)
    variable float uiScale = 1.0

    ;; Do not edit below this line
    if (!${uiScale.Equal[1.0]})
    {
        ; DebugScaling variable is set in LGUI2Scaling.iss
        if ${DebugScaling}
            echo Preprocessing UI with ${uiScale}x scale factor...

        ; Call scaling function: ScaleUIJson(inputFile, outputFile, scaleFactor)
        call ScaleUIJson "${LavishScript.HomeDirectory}/Scripts/YourScript/YourUI.json" "${LavishScript.HomeDirectory}/Scripts/YourScript/YourUI_Scaled.json" ${uiScale}

        ; Load the scaled version
        LGUI2:LoadPackageFile["YourScript/YourUI_Scaled.json"]
    }
    else
    {
        ; Load original unscaled version
        LGUI2:LoadPackageFile["YourScript/YourUI.json"]
    }
    ;;;;
    ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

    ; Continue with rest of your script...
    while 1
        waitframe
}
```

### Step 4: Test

1. **Run with default size:**
   ```
   run yourscript
   ```
   Should load original UI.

2. **Change scale factor:**
   ```lavishscript
   variable float uiScale = 2.0
   ```

3. **Run with 2x scaling:**
   ```
   run yourscript
   ```
   Should load UI at double size.

---

## How Scaling Works

### The Scaling Pipeline

```
Original JSON → Parse → Scale Dimensions → Scale Positions → Scale Fonts → Write Scaled JSON → Load UI
```

### What Gets Scaled

| Element Property | Scaling Behavior |
|------------------|------------------|
| **width** | Scaled by factor (buttons use 85% to prevent overlap) |
| **height** | Scaled by factor |
| **x** (non-window) | Scaled (absolute) or converted (percentage) |
| **y** (non-window) | Scaled (absolute) or converted (percentage) |
| **x** (window) | **NOT scaled** (preserves screen position) |
| **y** (window) | **NOT scaled** (preserves screen position) |
| **font.height** | Scaled by factor (added if missing) |
| **textblock alignment** | Fixed to "left" for labels |

### Percentage Position Conversion

**Problem:** Scaling percentage positions directly causes overflow:
```
Original: x = "5%" at 1x scale
Wrong:    x = "10%" at 2x scale  (20% at 4x scale = OVERFLOW!)
```

**Solution:** Convert percentages to absolute pixels based on scaled window dimensions:

```lavishscript
; Original: x = "5%" in 330px window
; Scaled 2x:
windowWidth = 330 * 2 = 660px
absoluteX = 660 * 5% = 33px
```

### Button Width Adjustment

**Problem:** Buttons with full width scaling overlap in multi-column layouts.

**Solution:** Scale button width to **85%** of the scale factor:

```lavishscript
widthFactor = scaleFactor * 0.85
buttonWidth = originalWidth * widthFactor
```

This leaves spacing between columns while still fitting scaled text.

### Font Injection

**Problem:** Not all elements have explicit fonts defined.

**Solution:** Add default scaled fonts to text-bearing elements:

```lavishscript
; If element has no font but should have text
if ${elemType.Equal[button]} || ${elemType.Equal[textblock]}
{
    variable jsonvalue defaultFont={}
    defaultFont:SetInteger[height,${Math.Calc[12 * ${scaleFactor}].Int}]
    elem:Set[font,"${defaultFont~}"]
}
```

---

## Implementation Details

### File Structure

**LGUI2Scaling.iss** contains three main functions:

```lavishscript
function ScaleUIJson(string inputFile, string outputFile, float scaleFactor)
{
    ; 1. Load JSON file
    ; 2. Parse to jsonvalue
    ; 3. Call ScaleElementsArray on root elements
    ; 4. Write scaled JSON to output file
}

function ScaleElementsArray(jsonvalueref rootObj, float scaleFactor)
{
    ; 1. Iterate elements array using ForEach
    ; 2. Call ScaleElement on each element
}

function ScaleElement(jsonvalueref elem, float scaleFactor)
{
    ; 1. Scale width/height
    ; 2. Check if element is window (preserve position)
    ; 3. Scale or convert x/y positions
    ; 4. Scale or add fonts
    ; 5. Fix textblock alignment
    ; 6. Recursively process nested elements (content, children, tabs)
}
```

### Critical Code Patterns

#### Window Detection

```lavishscript
; Check if this is a window type element (to preserve window position)
variable bool isWindow=FALSE
if ${elem.Has[type]}
{
    elemType:Set["${elem.Get[type]}"]
    if ${elemType.Equal[window]}
    {
        isWindow:Set[TRUE]
    }
}

; Skip position scaling for windows
if ${elem.Has[x]} && !${isWindow}
{
    ; Scale x position
}
```

**Why:** Window x/y positions are screen coordinates and should not scale (user expects window to stay in same place on screen).

#### Percentage Conversion

```lavishscript
; Handle x position - convert percentages to absolute pixels
if ${elem.Has[x]} && !${isWindow}
{
    variable string xValue
    xValue:Set["${elem.Get[x]}"]

    ; Check if it's a percentage
    if ${xValue.Find[%](exists)}
    {
        ; Convert percentage to absolute pixels based on scaled window width
        ; Original window is 330px, scaled window is 330 * scaleFactor
        variable float percent
        variable float windowWidth
        percent:Set[${xValue.Left[${Math.Calc[${xValue.Length}-1]}]}]
        windowWidth:Set[${Math.Calc[330 * ${scaleFactor}]}]
        newVal:Set[${Math.Calc[${windowWidth} * ${percent} / 100]}]
        elem:SetInteger[x,${newVal.Int}]
    }
    else
    {
        ; It's already an absolute pixel value, scale it
        newVal:Set[${elem.Get[x]} * ${scaleFactor}]
        elem:SetInteger[x,${newVal.Int}]
    }
}
```

**Why:** Prevents position values from exceeding 100% and going off-screen.

#### Type-Safe Recursion

```lavishscript
; Handle nested content object (tabcontrol, panel, etc.)
if ${elem.Has[content]}
{
    ; Only recurse if content is a jsonobject (not a string like button text)
    if ${elem.Get[content](type).Name.Equal[jsonobject]}
    {
        call ScaleElement elem.Get[content] ${scaleFactor}
    }
}
```

**Why:** Button `content` can be a string (text) or an object (nested element). Attempting to recurse into a string causes errors.

#### Button Width Adjustment

```lavishscript
; Scale width - but use smaller factor for buttons to prevent overlap
if ${elem.Has[width]}
{
    widthFactor:Set[${scaleFactor}]

    ; For buttons and textblocks, use larger width to fit scaled text
    if ${elem.Has[type]}
    {
        elemType:Set["${elem.Get[type]}"]
        if ${elemType.Equal[button]}
        {
            ; Buttons use 85% to fit text while leaving spacing between columns
            widthFactor:Set[${scaleFactor} * 0.85]
        }
        elseif ${elemType.Equal[textblock]}
        {
            ; Textblocks use full scale factor (they're just labels)
            widthFactor:Set[${scaleFactor}]
        }
    }

    newVal:Set[${elem.Get[width]} * ${widthFactor}]
    elem:SetInteger[width,${newVal.Int}]
}
```

**Why:** Full-width scaled buttons overlap in multi-column layouts. 85% width leaves space while fitting scaled text.

### Debug Mode

Enable debug output by editing `LGUI2Scaling.iss`:

```lavishscript
; Debug flag for UI scaling messages (set to TRUE to see scaling debug output)
variable bool DebugScaling=FALSE   ; Change to TRUE
```

**Debug Output:**
```
ScaleJSON: Loading C:\InnerSpace\Scripts\MyUI.json
ScaleJSON: JSON loaded, size 12345 chars
ScaleJSON: Scaling by factor 2.0
ScaleJSON: Writing to C:\InnerSpace\Scripts\MyUI_Scaled.json
ScaleJSON: Complete!
```

---

## Real-World Example: EQ2BotCommander

### Original UI Specifications

**EQ2BotCommander.json** (unscaled):
- Window: 800x800px
- Buttons: 229x60px each
- Font: 36pt
- Layout: 2-column button grid with tabs

### Scaled Version (2x)

**EQ2BotCommander_Scaled.json** (2x scale):
- Window: 1600x1600px
- Buttons: 389x120px each (85% width scaling)
- Font: 72pt
- Layout: Same 2-column grid, wider spacing

### Implementation in EQ2BotCommander_LGUI2.iss

```lavishscript
function main()
{
    ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
    ;;;; GUI Scaling
    ;; EQ2BotCommander comes with a GUI window. If you need to increase or decrease
    ;; the size of the window (and the elements within) simply change
    ;; the uiScale variable below. For example, to make everything twice as large,
    ;; set it to 2.0. To make everything half as large, set to 0.5
    variable float uiScale = 1.0

    ;; Do not edit below this line
    if (!${uiScale.Equal[1.0]})
    {
        ; Preprocess JSON to scale all dimensions
        if ${DebugScaling}
            echo Preprocessing UI with ${uiScale}x scale factor...

        ; Call the scaling function directly with absolute paths
        call ScaleUIJson "${LavishScript.HomeDirectory}/Scripts/EQ2Bot/UI/EQ2BotCommander.json" "${LavishScript.HomeDirectory}/Scripts/EQ2Bot/UI/EQ2BotCommander_Scaled.json" ${uiScale}

        ; Load the scaled LGUI2 package
        LGUI2:LoadPackageFile["EQ2Bot/UI/EQ2BotCommander_Scaled.json"]
    }
    else
        LGUI2:LoadPackageFile["EQ2Bot/UI/EQ2BotCommander.json"]
    ;;;;
    ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

    ; Wait for window to initialize
    wait 10 ${LGUI2.Element[EQ2BotCommander.Window](exists)}

    ; Rest of script...
}
```

### Comparison

#### Button Positions (Main Tab, Left Column)

| Element | Original | Scaled 2x | Calculation |
|---------|----------|-----------|-------------|
| RunEQ2Bot.x | 49 | 98 | 49 × 2 |
| RunEQ2Bot.y | 15 | 30 | 15 × 2 |
| Follow.x | 49 | 98 | 49 × 2 |
| Follow.y | 195 | 390 | 195 × 2 |

#### Button Dimensions

| Element | Original | Scaled 2x | Calculation |
|---------|----------|-----------|-------------|
| RunEQ2Bot.width | 229 | 389 | 229 × 2 × 0.85 |
| RunEQ2Bot.height | 60 | 120 | 60 × 2 |

#### Fonts

| Element | Original | Scaled 2x | Calculation |
|---------|----------|-----------|-------------|
| Button font | 36pt | 72pt | 36 × 2 |
| Tab font | 36pt | 72pt | 36 × 2 |

---

## Troubleshooting

### Issue 1: Buttons Overlapping

**Symptom:** After scaling, buttons in multi-column layouts overlap each other.

**Diagnosis:**
```lavishscript
; Original button positions:
; Column 1: x=49, width=229
; Column 2: x=544, width=229

; Scaled 2x (wrong):
; Column 1: x=98, width=458
; Column 2: x=1088, width=458
; Result: 98+458 = 556 overlaps with 1088!
```

**Fix:** Button width scaling uses 85% factor (already implemented):
```lavishscript
if ${elemType.Equal[button]}
{
    widthFactor:Set[${scaleFactor} * 0.85]
}
```

**Correct Result:**
```
Column 1: x=98, width=389 (229×2×0.85)
Column 2: x=1088, width=389
Result: 98+389 = 487, no overlap with 1088
```

### Issue 2: Labels Misaligned

**Symptom:** "PC Name:" and "Port Number:" labels appear too far right or centered.

**Diagnosis:** Textblocks with `horizontalAlignment: "center"` don't align with left-edge textboxes.

**Fix:** Force textblock labels to left alignment (already implemented):
```lavishscript
; Fix textblock horizontal alignment - labels should be left-aligned
if ${elem.Has[type]}
{
    elemType:Set["${elem.Get[type]}"]
    if ${elemType.Equal[textblock]} && ${elem.Has[horizontalAlignment]}
    {
        ; Change center alignment to left for proper label positioning
        variable string alignment
        alignment:Set["${elem.Get[horizontalAlignment]}"]
        if ${alignment.Equal[center]}
        {
            elem:SetString[horizontalAlignment,"left"]
        }
    }
}
```

### Issue 3: Window Moves After Scaling

**Symptom:** Scaled window appears at different screen position.

**Diagnosis:** Window x/y coordinates are being scaled.

**Fix:** Skip position scaling for window elements (already implemented):
```lavishscript
variable bool isWindow=FALSE
if ${elem.Has[type]}
{
    elemType:Set["${elem.Get[type]}"]
    if ${elemType.Equal[window]}
    {
        isWindow:Set[TRUE]
    }
}

if ${elem.Has[x]} && !${isWindow}
{
    ; Scale x position (skips windows)
}
```

### Issue 4: Fonts Too Small

**Symptom:** Text doesn't scale with buttons.

**Diagnosis:** Elements missing font definitions.

**Fix:** Auto-inject scaled fonts (already implemented):
```lavishscript
; If element has no font at all but is a type that should have text, add default font
elseif ${elem.Has[type]}
{
    elemType:Set["${elem.Get[type]}"]

    ; Add default font to text-bearing elements (LGUI2 uses "height" property)
    if ${elemType.Equal[button]} || ${elemType.Equal[textblock]} || ${elemType.Equal[textbox]}
    {
        variable jsonvalue defaultFont={}
        defaultFont:SetInteger[height,${Math.Calc[12 * ${scaleFactor}].Int}]
        elem:Set[font,"${defaultFont~}"]
    }
}
```

### Issue 5: Percentages Overflow Screen

**Symptom:** Elements with percentage positions go off-screen at high scale factors.

**Wrong Approach:**
```lavishscript
; Original: x = "5%"
; Scaled 5x: x = "25%"  (OK)
; Scaled 20x: x = "100%"  (At edge)
; Scaled 25x: x = "125%"  (OFF SCREEN!)
```

**Fix:** Convert percentages to absolute pixels (already implemented):
```lavishscript
if ${xValue.Find[%](exists)}
{
    ; Convert percentage to absolute pixels based on scaled window width
    variable float percent
    variable float windowWidth
    percent:Set[${xValue.Left[${Math.Calc[${xValue.Length}-1]}]}]
    windowWidth:Set[${Math.Calc[330 * ${scaleFactor}]}]
    newVal:Set[${Math.Calc[${windowWidth} * ${percent} / 100]}]
    elem:SetInteger[x,${newVal.Int}]
}
```

**Correct Result:**
```
Original: x = "5%" in 330px window
Scaled 5x: x = 82px (5% of 1650px)
Scaled 20x: x = 330px (5% of 6600px)
Scaled 25x: x = 412px (5% of 8250px)
All stay at 5% of scaled window width!
```

---

## Creating Scalable Title Bars

### The Title Bar Scaling Problem

**Issue:** Window title bars (title text, close button, shade button) are defined in the **DefaultSkin.json** template, not in your UI JSON file. The LGUI2Scaling.iss function **only scales elements in your JSON file**, so the title bar remains at its original small size even when the rest of your UI is scaled.

**Evidence from DefaultSkin.json:**
```json
"window.closeButton": {
    "font": {"height": 10},  // Hardcoded in skin
    "padding": 3
},
"window.shadeButton": {
    "font": {"height": 10},  // Hardcoded in skin
    "padding": 3
}
```

**Result:** If you scale your UI 2x or 3x, the window content is huge but the title bar buttons remain tiny.

### Solution: Custom Title Bar Override

To make the title bar scalable, you must **override** the default title bar by defining a custom `titleBar` in your JSON file. Once it's in your JSON, the scaling function will process it.

### Complete Example

Here's the pattern used in EQ2BotCommander.json:

```json
{
    "type": "window",
    "name": "MyWindow",
    "x": 100,
    "y": 100,
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
                "font": {
                    "face": "Segoe UI",
                    "height": 40
                },
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
                "font": {
                    "face": "Segoe UI",
                    "height": 40
                },
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
                "font": {
                    "face": "Segoe UI",
                    "height": 36,
                    "bold": true
                },
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

### Important: Remove the `title` Property

When you define a custom `titleBar`, you must **remove** the `title` property from the window definition to avoid duplication:

**Wrong (has both):**
```json
{
    "type": "window",
    "name": "MyWindow",
    "title": "My Window Title",  // ❌ Remove this!
    "titleBar": {
        // Custom title bar
    }
}
```

**Correct (custom titleBar only):**
```json
{
    "type": "window",
    "name": "MyWindow",
    "titleBar": {
        // Custom title bar with title text as textblock
    }
}
```

### Title Bar Component Breakdown

**1. Close Button (🗙):**
```json
{
    "type": "button",
    "borderThickness": 2,      // Will scale (1→2 at 2x, 1→3 at 3x)
    "content": "🗙",           // Unicode close symbol
    "font": {
        "face": "Segoe UI",
        "height": 40            // Will scale (10→40 at 2x from original 20)
    },
    "padding": 12,              // Will scale (6→12 at 2x)
    "_dock": "right",           // Dock to right side of title bar
    "eventHandlers": {
        "onMouseButtonMove": {
            "type": "forward",  // Forward event to window
            "elementType": "window",
            "event": "onCloseButtonClick"  // Triggers window close
        }
    }
}
```

**2. Shade Button (🗕):**
```json
{
    "type": "button",
    "borderThickness": 2,
    "content": "🗕",           // Unicode minimize symbol
    "font": {
        "face": "Segoe UI",
        "height": 40
    },
    "padding": 12,
    "margin": [0, 0, 8, 0],     // Right margin for spacing from close button
    "_dock": "right",
    "eventHandlers": {
        "onMouseButtonMove": {
            "type": "forward",
            "elementType": "window",
            "event": "onShadeButtonClick"  // Triggers minimize/restore
        }
    }
}
```

**3. Title Text:**
```json
{
    "type": "textblock",
    "text": "My Window Title",   // Your window title
    "verticalAlignment": "center",  // Center vertically in title bar
    "margin": [10, 0, 0, 0],  // Left margin for spacing
    "font": {
        "face": "Segoe UI",
        "height": 36,           // Will scale (36→72 at 2x)
        "bold": true            // Make title bold
    },
    "color": "#FFFFFF",         // White text
    "_dock": "left"             // Takes remaining space after buttons
}
```

**4. Title Bar Event Handlers:**
```json
"eventHandlers": {
    "onMouseMove": {
        "type": "forward",
        "elementType": "window",
        "event": "onHeaderMouseMove"  // Enables window dragging
    },
    "lostMouseFocus": {
        "type": "forward",
        "elementType": "window",
        "event": "onHeaderLostMouseFocus"  // Ends window dragging
    },
    "onMouseButtonMove": {
        "type": "forward",
        "elementType": "window",
        "event": "onHeaderMouseButtonMove"  // Mouse button on title bar
    }
}
```

### Sizing Guidelines

**For 2x Scale Factor (recommended starting point):**
- Close button font: 40 (original 20)
- Close button padding: 12 (original 6)
- Shade button font: 40
- Shade button padding: 12
- Shade button margin: [0, 0, 8, 0]
- Title text font: 36
- Border thickness: 2

**For 3x Scale Factor:**
- Close button font: 60
- Close button padding: 18
- Shade button font: 60
- Shade button padding: 18
- Shade button margin: [0, 0, 12, 0]
- Title text font: 54
- Border thickness: 3

**Formula:**
```
Scaled Value = Original Value × Scale Factor

Example:
Original padding: 6
Scale factor: 2x
Result: 6 × 2 = 12
```

### Why Event Forwarding is Required

The title bar buttons use the `"type": "forward"` event handler to pass events to the parent window element:

```json
"eventHandlers": {
    "onMouseButtonMove": {
        "type": "forward",         // Forward this event
        "elementType": "window",   // To the parent window element
        "event": "onCloseButtonClick"  // As this event name
    }
}
```

**Without forwarding:**
- Close button click does nothing
- Window can't be dragged by title bar
- Events stay trapped in button element

**With forwarding:**
- Button click → triggers window's `onCloseButtonClick` event
- Title bar drag → triggers window dragging behavior
- Events properly bubble up to window

### Script-Side Implementation

When using custom title bars, you need a shutdown atom in your script:

```lavishscript
atom(script) Shutdown()
{
    call SaveSettings
    Script:End
}
```

The `onCloseButtonClick` event handler calls this atom:

```json
"onCloseButtonClick": {
    "type": "code",
    "code": "Script[myscript]:QueueCommand[Shutdown]"
}
```

**Why `QueueCommand`?**
- Event handlers run in UI thread
- `QueueCommand` safely schedules execution in script thread
- Prevents threading issues during shutdown

### Complete Working Example: EQ2BotCommander

See [EQ2BotCommander.json](https://github.com/isxGames/isxScripts/blob/master/EverQuest2/Scripts/EQ2Bot/UI/EQ2BotCommander.json) lines 1-88 for a complete, production-ready implementation of a scalable custom title bar.

**Key features:**
- Custom title bar with 2x scaled buttons (font 40, padding 12)
- Bold white title text (font 36)
- Proper event forwarding for all window operations
- Works with LGUI2Scaling.iss automatic scaling

### Benefits of Custom Title Bars

✅ **Scalable** - Title bar scales with rest of UI
✅ **Customizable** - Full control over appearance
✅ **Consistent** - Same style across all scale factors
✅ **Accessible** - Large buttons easier to click
✅ **Professional** - Matches your UI design

### Limitations

❌ **More code** - ~70 lines of JSON vs 1-line `title` property
❌ **Manual maintenance** - Must update if changing button styles
❌ **No skin inheritance** - Doesn't inherit future skin updates

---

## Advanced Topics

### Hard-Coded Window Dimensions

**Problem:** The scaling function has hard-coded window dimensions:

```lavishscript
; Original window is 330px, scaled window is 330 * scaleFactor
windowWidth:Set[${Math.Calc[330 * ${scaleFactor}]}]

; Original window is 465px, scaled window is 465 * scaleFactor
windowHeight:Set[${Math.Calc[465 * ${scaleFactor}]}]
```

**Limitation:** These values are specific to a 330×465px window. Different window sizes won't calculate correctly.

**Solution:** For UIs with different window dimensions, update these values in `LGUI2Scaling.iss`:

```lavishscript
; For your 800x800 window:
windowWidth:Set[${Math.Calc[800 * ${scaleFactor}]}]
windowHeight:Set[${Math.Calc[800 * ${scaleFactor}]}]
```

**Better Solution (Future Enhancement):** Dynamically detect window dimensions from the root window element:

```lavishscript
; In ScaleElement function, detect window dimensions
if ${isWindow}
{
    ; Store window dimensions for percentage calculations
    if ${elem.Has[width]}
        windowWidth:Set[${elem.Get[width]} * ${scaleFactor}]
    if ${elem.Has[height]}
        windowHeight:Set[${elem.Get[height]} * ${scaleFactor}]
}
```

### Scaling Multiple Windows

**Pattern:** Scale each window separately if they have different dimensions:

```lavishscript
function main()
{
    variable float uiScale = 2.0

    if (!${uiScale.Equal[1.0]})
    {
        ; Scale window 1 (800x800)
        call ScaleUIJson "Window1.json" "Window1_Scaled.json" ${uiScale}

        ; Scale window 2 (400x300) - may need adjusted hard-coded values
        call ScaleUIJson "Window2.json" "Window2_Scaled.json" ${uiScale}

        LGUI2:LoadPackageFile["Window1_Scaled.json"]
        LGUI2:LoadPackageFile["Window2_Scaled.json"]
    }
}
```

### Custom Scale Factors per Element

**Use Case:** Scale buttons 2x but fonts 1.5x.

**Approach:** Modify `ScaleElement` to accept element-specific scale factors:

```lavishscript
function ScaleElement(jsonvalueref elem, float scaleFactor, float fontScaleFactor=0)
{
    ; Use fontScaleFactor if provided, otherwise use scaleFactor
    if ${fontScaleFactor} == 0
        fontScaleFactor:Set[${scaleFactor}]

    ; Scale fonts using fontScaleFactor
    if ${elem.Get[font].Has[height]}
    {
        variable float fontHeight
        fontHeight:Set[${elem.Get[font].Get[height]} * ${fontScaleFactor}]
        elem.Get[font]:SetInteger[height,${fontHeight.Int}]
    }
}
```

### User-Configurable Scaling

**Pattern:** Load scale factor from settings file:

```lavishscript
function main()
{
    variable float uiScale

    ; Load from LavishSettings
    uiScale:Set[${LavishSettings[MyScript].Set[UI].GetSetting[ScaleFactor,1.0]}]

    ; Or load from JSON config
    variable jsonvalue config
    config:ParseFile["config.json"]
    uiScale:Set[${config.Get[uiScale]}]

    ; Continue with scaling logic...
}
```

**User Experience:** Add a UI control to adjust scale factor and save to settings.

---

## Summary

### Key Takeaways

1. **Single Include** - `#include LGUI2Scaling.iss` in your script
2. **One Variable** - Set `uiScale` to desired scaling factor
3. **Conditional Loading** - Load original or scaled JSON based on scale factor
4. **Automatic Processing** - Scaling handles all dimensions, positions, and fonts
5. **Preserved Layouts** - Proportions and spacing maintained at any scale
6. **Window Position Fixed** - Screen position doesn't change with scaling
7. **Debug Mode** - Enable `DebugScaling=TRUE` for troubleshooting

### Benefits

- ✅ **Accessibility** - Users can adjust UI to their screen/DPI
- ✅ **Reusable** - Same library works for all LGUI2 scripts
- ✅ **Maintainable** - Update one file to improve all UIs
- ✅ **No Manual Editing** - Users don't touch JSON files
- ✅ **Automatic** - Handles all scaling complexity

### Common Patterns

**Basic Implementation:**
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

**With Debug Mode:**
```lavishscript
if (!${uiScale.Equal[1.0]})
{
    if ${DebugScaling}
        echo Preprocessing UI with ${uiScale}x scale factor...
    call ScaleUIJson "MyUI.json" "MyUI_Scaled.json" ${uiScale}
    LGUI2:LoadPackageFile["MyUI_Scaled.json"]
}
```

**With User Settings:**
```lavishscript
uiScale:Set[${LavishSettings[MyScript].Set[UI].GetSetting[ScaleFactor,1.0]}]
```

---

## Next Steps

1. **Copy** `LGUI2Scaling.iss` to your scripts directory
2. **Include** it in your LGUI2-based script
3. **Add** scaling logic to your `main()` function
4. **Test** with different scale factors (0.5, 1.0, 1.5, 2.0, etc.)
5. **Adjust** hard-coded window dimensions if needed
6. **Enable** debug mode if troubleshooting issues

---

## Additional Resources

- **LavishGUI 2 UI Guide:** [10_LavishGUI2_UI_Guide.md](10_LavishGUI2_UI_Guide.md)
- **Migration Guide:** [11_LavishGUI1_to_LavishGUI2_Migration.md](11_LavishGUI1_to_LavishGUI2_Migration.md)
- **Example Implementation:** `EQ2BotCommander_LGUI2.iss`
- **Scaling Library Source:** `LGUI2Scaling.iss`

---

*Last Updated: 2025-10-25*
*Created during EQ2BotCommander LGUI1→LGUI2 migration*
