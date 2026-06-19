# ISXPantheon Master Guide - Quick Reference

Complete quick reference for ISXPantheon scripting. For detailed information, see the individual documentation files.

> **IMPORTANT.** ISXPantheon is in early development. Two areas are live today: `${ISXPantheon}` (extension control + helpers) and `${Login}` (login state, realm enumeration, and the login screen's buttons/input fields). The in-world game-data surface (local player, entities, abilities, quests, crafting, navigation, radar, events, etc.) is **planned but not yet implemented** — see [03_API_Reference.md - Planned API](03_API_Reference.md#planned-api). Sections below marked **(planned — not yet implemented)** describe the intended future API and WILL change.

---

## Table of Contents

- [Quick Links](#quick-links)
- [Essential TLOs](#essential-tlos-top-level-objects)
- [ISXPantheon (Extension) - Most Used Members](#isxpantheon-extension---most-used-members)
- [Login / Realm / UI Surface](#login--realm--ui-surface)
- [Commands](#commands)
- [Essential Patterns](#essential-patterns)
- [Events](#events)
- [Control Flow](#control-flow)
- [Variable Declarations](#variable-declarations)
- [String Manipulation](#string-manipulation)
- [Math Operations](#math-operations)
- [Wait Commands](#wait-commands)
- [Datatype Inheritance](#datatype-inheritance)
- [Function Syntax](#function-syntax)
- [Debugging Tips](#debugging-tips)
- [Best Practices Checklist](#best-practices-checklist)
- [Quick Troubleshooting](#quick-troubleshooting)
- [Planned Surface](#planned-surface)

---

## Quick Links

- **[README.md](README.md)** - Start here for navigation
- **[02_Quick_Start_Guide.md](02_Quick_Start_Guide.md)** - Get started in 5-10 minutes
- **[03_API_Reference.md](03_API_Reference.md)** - Full API reference (real surface + planned roadmap)
- **[04_Core_Concepts.md](04_Core_Concepts.md)** - Fundamental concepts
- **[05_Patterns_And_Best_Practices.md](05_Patterns_And_Best_Practices.md)** - Best practices
- **[06_Working_Examples.md](06_Working_Examples.md)** - Code examples
- **[20_Debugging_And_Troubleshooting.md](20_Debugging_And_Troubleshooting.md)** - Debugging and troubleshooting capstone

---

## Essential TLOs (Top-Level Objects)

| TLO | Type | Status | Common Usage |
|-----|------|--------|--------------|
| `${ISXPantheon}` | isxpantheon | REAL | `${ISXPantheon.IsReady}`, `${ISXPantheon.Version}` |
| `${Login}` | login | REAL | `${Login.State}`, `${Login.NumRealms}`, `${Login.Realm[1].Name}`, `${Login.AccountName}` |
| `${CharSelect}` | charselect | REAL | `${CharSelect.NumCharacters}`, `${CharSelect.Character[1].Name}`, `:ChooseCharacter[1]`, `:EnterWorld` (NULL unless at char-select scene) |
| `${CharCreate}` | charcreate | REAL | `${CharCreate.Race}`, `:SetName[...]`, `:SetClass[...]`, `:Create` (NULL unless at char-create scene) |
| `${Pantheon}` | pantheon | REAL | `${Pantheon.ResolutionWidth}`, `${Pantheon.FPSLimit}`, `${Pantheon.NumCameras}`, `:SetFPSLimit[60]` |

`${Me}` and `${Radar}` exist in the source only as reserved (commented-out) top-level objects — they are not registered and return nothing today. The game-data surface is planned; see [03_API_Reference.md](03_API_Reference.md#top-level-objects-tlos).

---

## ISXPantheon (Extension) - Most Used Members

### Status / Version
```lavishscript
${ISXPantheon.Version}          ; Extension version string
${ISXPantheon.APIVersion}       ; API version string
${ISXPantheon.IsReady}          ; TRUE once the extension has finished loading
```

### Custom Variables
```lavishscript
ISXPantheon:SetCustomVariable[MyFlag,1]            ; Set a custom variable
${ISXPantheon.GetCustomVariable[MyFlag,int]}       ; Read it back (typed)
ISXPantheon:ClearAllCustomVariables                ; Clear all custom variables
```

### Helpers
```lavishscript
${ISXPantheon.GetCurrencyString[12345]}            ; Format a currency amount
${ISXPantheon.Round[int,47,5]}                     ; Round to nearest multiple
```

### Extension Control
```lavishscript
ISXPantheon:Reload[10]                             ; Reload the extension after a delay
ISXPantheon:Unload                                 ; Unload the extension
```

---

## Login / Realm / UI Surface

Live at the login / realm-selection screen. The `login`, `realm`, `uibutton`, `uitext`, and `uiinputfield` datatypes all inherit from the base `object` type. Full reference: [03_API_Reference.md - login](03_API_Reference.md#login).

```lavishscript
${Login.State}                  ; None, EULA, Connecting, LogIn, WaitingForPlayerToSelectRealm, EnteringWorld, ...
${Login.AccountName}            ; account name currently entered
${Login.AccountPassword}        ; account password currently entered
${Login.SelectedRealmName}      ; realm selected by the player
${Login.CurrentRealmName}       ; realm currently connected to
${Login.NumRealms}              ; realm count (only while State == WaitingForPlayerToSelectRealm)
${Login.Realm[1].Name}          ; realm fields: Name, Address, Location, Version, Load, IsFull
```

```lavishscript
; Buttons (uibutton): IsInteractable, IsActive, Label (uitext)  +  :Press
${Login.LoginButton.IsInteractable}
${Login.QuitButton.Label.Text}
; Login.LoginButton:Press

; Input fields (uiinputfield): IsInteractable, IsActive, IsFocused, IsReadOnly, CharacterLimit, Text, ContentType  +  :SetText[...]
${Login.AccountNameInputField.Text}
${Login.AccountPasswordInputField.ContentType}
; Login.AccountNameInputField:SetText["myaccount"]
```

```lavishscript
; uitext members: Text, HorizontalAlignment, VerticalAlignment, OverflowMode, FontColor (uicolor), FontSize, IsRichText  +  :Set[...] / GetFontStyle
; uicolor members: R, G, B, A

; Login action methods (drive the flow — call deliberately):
; Login:SetAccountName["acct"] ; Login:SetAccountPassword["pw"] ; Login:Login
; Login:SetRealm["Realm Name"] ; Login:EnterWorld
```

---

## Character Select / Create Surface

`${CharSelect}` and `${CharCreate}` are only valid at the matching scene (NULL otherwise). Datatypes inherit from `object`. Full reference: [03_API_Reference.md - charselect](03_API_Reference.md#charselect).

```lavishscript
; charselect: NumCharacters, Character[#] (charselect-character), CurrentSelectedCharacter,
;             EnterWorldButton/DeleteCharacterButton/ChangeModelButton (uibutton), DisplayTutorials (uitoggle)
${CharSelect.NumCharacters}
${CharSelect.Character[1].Name}            ; charselect-character: Name, CharacterID (int64), Level, Race, Class, Gender, Zone
${CharSelect.CurrentSelectedCharacter.Name}
; CharSelect:ChooseCharacter[1] ; :EnterWorld ; :DeleteSelectedCharacter ; :EnterCharacterCreation ; :ReturnToLogin

; charcreate: Name, Race, RaceDescription, Class, ClassDescription, Gender, *Button (uibutton),
;             Male/FemaleToggle (uitoggle), HairStyle/HairColor/FacialHair/FacialHairColor (uislider), Attributes
${CharCreate.Race}
${CharCreate.HairStyle.Value}              ; uislider: Value, Max, Min (Min is ALWAYS -1)  +  :SetValue[#]
${CharCreate.Attributes.PointsLeft}        ; uiattributeselection: PointsLeft, NumSelectors, Selector[#], *Button  +  :Reset
${CharCreate.Attributes.Selector[1].Name}  ; uiattributeselector: Name, Value, BaseValue, Type, Minus/PlusButton  +  :SetValue[#]
; CharCreate:SetName["X"] ; :SetRace["Human"] ; :SetClass["Warrior"] ; :SetGender["Male"] ; :RandomizeAppearance ; :Create ; :Cancel
```

---

## Render / Camera Surface (${Pantheon})

```lavishscript
${Pantheon.ResolutionWidth}     ; ResolutionWidth, ResolutionHeight, RenderQuality, FrameCount
${Pantheon.FPSLimit}            ; FPSLimit, VSyncCount (0 = off), FullScreenMode (string)
${Pantheon.NumCameras}          ; Camera[#] returns uicamera: Enabled  +  :Enable / :Disable
; Pantheon:SetFPSLimit[60]      ; NOTE: also disables VSync (Unity ignores the FPS limit while VSync is on)
; Pantheon:RestoreCameras
```

---

## Commands

```lavishscript
GetURL "https://example.com/data"                  ; HTTP GET (async; result via isxGames_onHTTPResponse)
PostURL "https://example.com/api" "field=value"    ; HTTP POST (async)
```

No game-control commands (movement, targeting, radar, where) exist yet — those are **(planned — not yet implemented)**.

---

## Essential Patterns

### NULL Check Pattern
```lavishscript
if ${ISXPantheon(exists)}
{
    echo ${ISXPantheon.Version}
}
```

### Wait for ISXPantheon Pattern
```lavishscript
function main()
{
    while !${ISXPantheon.IsReady}
        wait 10

    ; Script logic here
}
```

### HTTP Request Pattern
```lavishscript
function main()
{
    Event[isxGames_onHTTPResponse]:AttachAtom[OnHTTP]
    GetURL "https://example.com/data"

    while 1
        waitframe
}

atom OnHTTP(int Size, string URL, string IPAddress, int ResponseCode, float TransferTime, string ResponseText, string ParsedBody)
{
    echo HTTP ${ResponseCode} from ${URL} (${Size} bytes)
}
```

---

## Events

| Event | When It Fires | Arguments |
|-------|---------------|-----------|
| `isxGames_onHTTPResponse` | Response received from GetURL/PostURL | Size, URL, IPAddress, ResponseCode, TransferTime, ResponseText, ParsedBody |

> **PLANNED — NOT YET IMPLEMENTED.** Game events (entity spawn/despawn, incoming chat text, zoning, level change, etc.) are planned but not wired up in the current build. They will never fire today.

---

## Control Flow

### If Statement
```lavishscript
if ${Condition}
{
    ; Code
}
elseif ${OtherCondition}
{
    ; Code
}
else
{
    ; Code
}
```

### While Loop
```lavishscript
while ${Condition}
{
    ; Code
    wait 10
}
```

### For Loop
```lavishscript
variable int i
for (i:Set[1]; ${i} <= 10; i:Inc)
{
    ; Code
}
```

### Switch Statement
```lavishscript
switch ${Variable}
{
    case Value1
        ; Code
        break
    case Value2
        ; Code
        break
    default
        ; Code
        break
}
```

---

## Variable Declarations

### Basic Types
```lavishscript
declare MyInt int script 100
declare MyFloat float script 3.14
declare MyString string script "Hello"
declare MyBool bool script TRUE
```

### Collections
```lavishscript
variable index:string Names
variable collection:string Lookup
variable iterator MyIterator
```

### Scopes
```lavishscript
declare ScriptVar int script 0      ; Persists across functions
variable LocalVar int local 0        ; Only exists in function
```

---

## String Manipulation

```lavishscript
${String.Length}                ; Length
${String.Left[5]}              ; First 5 characters
${String.Right[3]}             ; Last 3 characters
${String.Upper}                ; UPPERCASE
${String.Lower}                ; lowercase
${String.Find["text"]}         ; Position of "text" (0 if not found)
${String.Equal["text"]}        ; TRUE if equal (case-sensitive)
${String.Replace["old","new"]} ; Replace text
${String.Token[2,:]}           ; Second token separated by ':'
```

---

## Math Operations

```lavishscript
${Math.Calc[10+5]}             ; 15
${Math.Calc[10-5]}             ; 5
${Math.Calc[10*5]}             ; 50
${Math.Calc[10/5]}             ; 2
${Math.Calc[(10+5)*2]}         ; 30
${Math.Rand[100]}              ; Random 0-99
${Math.Rand[100]:Inc}          ; Random 1-100
```

---

## Wait Commands

```lavishscript
wait 10                        ; Wait 1 second (10 deciseconds)
wait 50 ${Condition}           ; Wait up to 5 seconds for condition
waitframe                      ; Wait one frame (~16ms)
```

---

## Datatype Inheritance

Understanding inheritance helps you know what members are available. In LavishScript a derived datatype exposes all of its parent's members in addition to its own. The planned game-data datatypes are intended to use inheritance (for example, the local-player datatype is expected to inherit from the generic entity datatype), but those types are not implemented yet — see [03_API_Reference.md - Planned API](03_API_Reference.md#planned-api). For the full explanation of how inheritance works, see [04_Core_Concepts.md](04_Core_Concepts.md#datatype-inheritance).

---

## Function Syntax

### Basic Function
```lavishscript
function MyFunction()
{
    echo "Function called"
}

call MyFunction
```

### Function with Parameters
```lavishscript
function Greet(string name, int level)
{
    echo "Hello ${name}, level ${level}!"
}

call Greet "Bob" 120
```

### Function with Return
```lavishscript
function GetValue()
{
    return 42
}

call GetValue
variable int V
V:Set[${Return}]
```

---

## Debugging Tips

### Echo Debugging
```lavishscript
echo "Debug: Variable = ${MyVariable}"
echo "Debug: ISXPantheon ready? ${ISXPantheon.IsReady}"
```

### Conditional Debug
```lavishscript
declare DEBUG_MODE bool script TRUE

if ${DEBUG_MODE}
    echo "Debug: Reached checkpoint 1"
```

---

## Best Practices Checklist

- Always check `(exists)` before accessing objects
- Use `wait 10` in loops to prevent CPU spikes
- Cache frequently accessed values in variables
- Use early returns to reduce nesting
- Name variables descriptively
- Wait for `${ISXPantheon.IsReady}` at script start
- Add comments for complex logic
- Use timeouts when waiting for async data
- Handle errors gracefully

---

## Quick Troubleshooting

| Problem | Solution |
|---------|----------|
| "Invalid type" error | Add `(exists)` check before access |
| Script hangs | Add `wait` in loops |
| Variable always empty | Check variable scope (script vs local) |
| `${ISXPantheon}` not found | Make sure the extension is loaded (`extension isxpantheon`) |
| Game-data member returns nothing | That surface is likely planned/not yet implemented — see [03_API_Reference.md](03_API_Reference.md#planned-api) |

For the full debugging-and-troubleshooting treatment, see **[20_Debugging_And_Troubleshooting.md](20_Debugging_And_Troubleshooting.md)**.

---

## Planned Surface

> **PLANNED — NOT YET IMPLEMENTED.** The following are on the ISXPantheon roadmap but are not available today. Names and signatures are provisional and WILL change. Do not write production scripts against them.

- `${Me}` and `${Radar}` — reserved (commented-out) top-level objects in the source; not registered yet.
- `pantheon`, `entity`, `ability`, `quest` datatypes — registered today but have no working members yet.
- A `Where` command, a `Radar` command, and a `Crafting` TLO/datatype — named on the roadmap.

Additional game-data datatypes and commands are planned. See [03_API_Reference.md - Planned API](03_API_Reference.md#planned-api).
