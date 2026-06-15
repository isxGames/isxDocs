# ISXPantheon API Reference

Complete API reference for the **ISXPantheon** extension (InnerSpace / ISXDK35) for **Pantheon: Rise of the Fallen**. This guide documents the datatypes, members, methods, and commands that the extension currently exposes to LavishScript, plus a clearly labeled roadmap of planned game-data APIs.

> **IMPORTANT — READ THIS FIRST.** ISXPantheon is in early development, but its live surface has grown beyond extension control. Two areas work today:
>
> - **Extension control and utility helpers** via `${ISXPantheon}`.
> - **Login / realm-selection / login-UI automation** via the `${Login}` TLO and its supporting datatypes (`login`, `realm`, `uibutton`, `uitext`, `uiinputfield`, `uicolor`, and the base `object` type they derive from). These let a script inspect login state, enumerate realms, read and drive the login screen's buttons and input fields, and enter the world.
>
> The **in-world game-data surface** (the local player, entities, abilities, quests, crafting, navigation, radar, and game events) is still **planned but not yet implemented**. Anything below marked **(planned — not yet implemented)** describes the intended future API; its names and signatures are provisional and WILL change. Do not write production scripts against planned features.

---

## Table of Contents

- [Introduction](#introduction)
  - [DataType Inheritance](#datatype-inheritance)
  - [Accessing DataTypes](#accessing-datatypes)
  - [NULL Checks](#null-checks)
  - [Asynchronous Data Loading](#asynchronous-data-loading)
- [Top-Level Objects (TLOs)](#top-level-objects-tlos)
- [Commands](#commands)
- [Implemented DataType Reference](#implemented-datatype-reference)
  - [isxpantheon](#isxpantheon)
  - [object](#object)
  - [uicolor](#uicolor)
  - [login](#login)
  - [realm](#realm)
  - [uibutton](#uibutton)
  - [uitext](#uitext)
  - [uiinputfield](#uiinputfield)
  - [pantheon](#pantheon)
- [Registered but Stubbed DataTypes](#registered-but-stubbed-datatypes)
  - [entity](#entity)
  - [ability](#ability)
  - [quest](#quest)
- [Events](#events)
- [Planned API](#planned-api)

---

## Introduction

This document provides reference documentation for the datatypes and top-level objects available in the ISXPantheon extension for Pantheon: Rise of the Fallen. ISXPantheon extends LavishScript to provide access to extension control, utility helpers, and (over time) game data through a structured type system.

### DataType Inheritance

LavishScript datatypes can inherit from parent types, so a derived datatype exposes all of its parent's members in addition to its own. The planned game-data datatypes (see [Planned API](#planned-api)) are intended to use inheritance — for example, the local-player datatype is expected to inherit from the generic entity datatype — but those types are not implemented yet.

See [04_Core_Concepts.md - Datatype Inheritance](04_Core_Concepts.md#datatype-inheritance) for the full explanation of how inheritance works in LavishScript.

### Accessing DataTypes

DataTypes are accessed through Top-Level Objects (TLOs) or through members of other datatypes. For example:

```lavishscript
${ISXPantheon}                  ; TLO returning the 'isxpantheon' datatype
${ISXPantheon.Version}          ; Member returning a string
${ISXPantheon.IsReady}          ; Member returning a bool
```

### NULL Checks

Always check that a TLO or datatype exists before accessing its members:

```lavishscript
if ${ISXPantheon(exists)}
{
    echo ISXPantheon version: ${ISXPantheon.Version}
}
```

For the planned game-data TLOs, the same `(exists)` pattern will apply once they are implemented. Until then, use it only against the real `${ISXPantheon}` object as shown above.

> **PLANNED — NOT YET IMPLEMENTED.** Game-data top-level objects are on the ISXPantheon roadmap but are not available in the current build. The API is provisional and WILL change. Do not write production scripts against it yet.

### Asynchronous Data Loading

LavishScript extensions commonly load detailed information from the game server asynchronously, so scripts must wait for data to become available before reading it. ISXPantheon does not yet expose any game-data objects, so there is no asynchronous game data to wait on today. The only readiness gate that exists now is the extension-load gate:

```lavishscript
; Wait until the extension has finished loading before doing anything
do { waitframe }
while !${ISXPantheon.IsReady}
```

The one asynchronous mechanism ISXPantheon provides today is HTTP: `GetURL` / `PostURL` return immediately and their results arrive later through the `isxGames_onHTTPResponse` event (see [Events](#events)). This is described in more detail in [04_Core_Concepts.md](04_Core_Concepts.md#asynchronous-data-loading). The loading semantics of any future game-data APIs are not defined yet.

---

## Top-Level Objects (TLOs)

Top-Level Objects (TLOs) are the entry points for accessing extension and game data. They can be accessed directly in LavishScript without any prefix.

| TLO | DataType | Status | Description |
|-----|----------|--------|-------------|
| **ISXPantheon** | [isxpantheon](#isxpantheon) | REAL | Extension information, control, and utility helpers. Available regardless of game/login state. |
| **Login** | [login](#login) | REAL | The login / realm-selection screen. Inspect login state, enumerate realms, read and drive the login UI, and enter the world. Most members are only meaningful while you are sitting at the login/realm-selection screen. |
| **Pantheon** | [pantheon](#pantheon) | Registered (empty shell) | Reserved game TLO. Registered, but the `pantheon` datatype currently exposes no members or methods. |

`${Me}` and `${Radar}` exist in the source only as reserved (commented-out) top-level objects and return nothing today. Additional game-data TLOs are planned — see [Planned API](#planned-api).

---

## Commands

Console/script commands compiled into the current public build:

| Command | Status | Description |
|---------|--------|-------------|
| **GetURL** | REAL | Performs an HTTP GET request. The response is delivered asynchronously via the `isxGames_onHTTPResponse` event (see [Events](#events)). |
| **PostURL** | REAL | Performs an HTTP POST request. The response is delivered asynchronously via the `isxGames_onHTTPResponse` event. |

No game-control commands (movement, targeting, radar, etc.) exist yet. Those are **planned but not yet implemented** — see [Planned API](#planned-api).

---

## Implemented DataType Reference

---

### isxpantheon

Extension information, control, and utility helpers. This is the always-available control datatype; the login / realm / UI datatypes below are also live while you are at the login screen.

**Access:** `${ISXPantheon}`

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Version | string | Extension product version string |
| APIVersion | string | Extension API version string |
| IsReady | bool | TRUE once the extension has finished loading. Wait on this before doing any work in a script. |
| GetCustomVariable[name] | string | Reads a user-defined custom variable by name (returns it as a string). |
| GetCustomVariable[name,type] | typed | Reads a user-defined custom variable as a specific type. `type` is one of `str`, `int`, `uint`, `bool`, `float`. |
| GetCurrencyString[amount] | string | Formats a currency amount (given in the game's smallest unit) into a display string. |
| Round[type,value,multiple] | numeric | Rounds `value` to the nearest `multiple`. `type` is one of `float`, `int`, `uint`, `double`. |

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| SetCustomVariable | name, value | Sets a user-defined custom variable. |
| ClearAllCustomVariables | - | Clears all user-defined custom variables. |
| Reload | delay | Reloads the extension after an optional delay. |
| Unload | - | Unloads the extension. |
| ToggleOptionalAutoReloads | - | Toggles the optional automatic-reload behavior. |
| InstallLive | - | Switches to the live build channel. |
| InstallTest | - | Switches to the test build channel. |
| InstallBeta | - | Switches to the beta build channel. |
| InstallTest \| InstallLive \| InstallBeta | - | (Build-channel management. Dev-oriented; most scripts will not need these.) |

**Example Usage:**
```lavishscript
echo ISXPantheon version: ${ISXPantheon.Version}
echo API version:         ${ISXPantheon.APIVersion}

; Wait for the extension to be ready before doing anything
do { waitframe }
while !${ISXPantheon.IsReady}

; Custom variables
ISXPantheon:SetCustomVariable[MyFlag,1]
echo ${ISXPantheon.GetCustomVariable[MyFlag,int]}

; Helpers
echo ${ISXPantheon.GetCurrencyString[12345]}
echo ${ISXPantheon.Round[int,47,5]}
```

---

### object

The base datatype for the login / UI object family. `login`, `realm`, `uibutton`, `uitext`, and `uiinputfield` all **inherit from `object`**, so any member or method defined on `object` is also available on those derived types.

`object` is the generic handle for an underlying game/UI object. It exposes no members or methods of its own today — it exists as the common base of the inheritance chain. You will rarely reference `object` directly; you work with the derived types below and gain `object`'s (currently empty) surface automatically through inheritance.

> **Inheritance note.** Throughout this section, a datatype marked *(inherits `object`)* exposes everything listed in its own table **plus** every member and method of `object`. Because `object` is currently empty, the derived type's own table is its full surface today — but new `object` members added later will automatically appear on all derived types.

---

### uicolor

An RGBA color, as returned by a UI element's color member (for example `${Login.AccountNameInputField}`'s text color is exposed through `uitext`'s `FontColor` member, which returns a `uicolor`).

**Access:** returned by other datatypes' color members (e.g. `${SomeUIText.FontColor}`).

#### Members

| Member | Type | Description |
|--------|------|-------------|
| R | int | Red channel (0-255). |
| G | int | Green channel (0-255). |
| B | int | Blue channel (0-255). |
| A | int | Alpha channel (0-255). |

This datatype has no methods.

**Example Usage:**
```lavishscript
echo "Font color: R=${SomeText.FontColor.R} G=${SomeText.FontColor.G} B=${SomeText.FontColor.B} A=${SomeText.FontColor.A}"
```

---

### login

The login / realm-selection screen. *(inherits `object`)*

**Access:** `${Login}`

Most members below are only meaningful while you are at the login screen. In particular, **`NumRealms` and `Realm[#]` only return data while `${Login.State}` is `WaitingForPlayerToSelectRealm`** — they return false/empty at any other state. Always check `State` before iterating realms.

#### Members

| Member | Type | Description |
|--------|------|-------------|
| State | string | Current login state. One of: `None`, `EULA`, `BrightnessCalibration`, `Connecting`, `FailedToConnect`, `Disconnected`, `LogIn`, `LoginWithSteam`, `WaitingForPlayerToSelectRealm`, `InvalidVersion`, `EnteringWorld`. |
| NumRealms | int | Number of realms available for selection. Only valid while `State` is `WaitingForPlayerToSelectRealm`; returns false otherwise. |
| Realm[#] | [realm](#realm) | The realm at index `#` (1-based). Only valid while `State` is `WaitingForPlayerToSelectRealm`; returns NULL otherwise. |
| SelectedRealmName | string | Name of the realm currently selected by the player. |
| CurrentRealmName | string | Name of the realm currently connected to. |
| AccountName | string | The account name currently entered. |
| AccountPassword | string | The account password currently entered. |
| LoginButton | [uibutton](#uibutton) | The "Login" button on the login screen. |
| QuitButton | [uibutton](#uibutton) | The "Quit" button on the login screen. |
| CalibrateButton | [uibutton](#uibutton) | The brightness-calibration button. |
| EulaAcceptButton | [uibutton](#uibutton) | The "Accept" button on the EULA screen. |
| RealmBackButton | [uibutton](#uibutton) | The "Back" button on the realm-selection screen. |
| RealmQuitButton | [uibutton](#uibutton) | The "Quit" button on the realm-selection screen. |
| AccountNameInputField | [uiinputfield](#uiinputfield) | The account-name input field. |
| AccountPasswordInputField | [uiinputfield](#uiinputfield) | The account-password input field. |

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| SetAccountName | name | Sets the text of the account-name field. |
| SetAccountPassword | password | Sets the text of the account-password field. |
| Login | - | Submits the current account name/password (presses the login flow). |
| SetRealm | name | Selects the named realm during realm selection. |
| EnterWorld | - | Enters the world with the currently selected realm/character. |

**Example Usage:**
```lavishscript
echo "Login state: ${Login.State}"
echo "Account:     ${Login.AccountName}"

; Enumerate realms (only while at realm selection)
if ${Login.State.Equal[WaitingForPlayerToSelectRealm]}
{
    variable int i
    for (i:Set[1] ; ${i} <= ${Login.NumRealms} ; i:Inc)
    {
        echo "Realm ${i}: ${Login.Realm[${i}].Name} (load ${Login.Realm[${i}].Load})"
    }
}

; Driving the login (action methods — only call deliberately):
; Login:SetAccountName["myaccount"]
; Login:SetAccountPassword["mypassword"]
; Login:Login
```

---

### realm

A single realm (server) entry in the realm-selection list. *(inherits `object`)*

**Access:** `${Login.Realm[#]}` (only while `${Login.State}` is `WaitingForPlayerToSelectRealm`).

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Name | string | Realm display name. |
| Address | string | Realm network address. |
| Location | string | Realm geographic location/label. |
| Version | string | Realm version string. |
| Load | string | Current population/load indicator. |
| IsFull | bool | TRUE if the realm is full. |

This datatype has no methods.

**Example Usage:**
```lavishscript
if ${Login.State.Equal[WaitingForPlayerToSelectRealm]} && ${Login.NumRealms} > 0
{
    echo "First realm: ${Login.Realm[1].Name} @ ${Login.Realm[1].Address}"
    echo "  Location: ${Login.Realm[1].Location}  Version: ${Login.Realm[1].Version}"
    echo "  Load: ${Login.Realm[1].Load}  Full: ${Login.Realm[1].IsFull}"
}
```

---

### uibutton

A clickable UI button. *(inherits `object`)*

**Access:** returned by `login` button members (e.g. `${Login.LoginButton}`).

#### Members

| Member | Type | Description |
|--------|------|-------------|
| IsInteractable | bool | TRUE if the button can currently be pressed. |
| IsActive | bool | TRUE if the button is active/visible. |
| Label | string | The button's text label. |

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| Press | - | Presses (clicks) the button. |

**Example Usage:**
```lavishscript
echo "Login button: interactable=${Login.LoginButton.IsInteractable} label=${Login.LoginButton.Label}"

; Press it (action method — only call deliberately):
; Login.LoginButton:Press
```

---

### uitext

A read/write UI text label. *(inherits `object`)*

**Access:** returned by UI members that expose a text label.

#### Members

| Member | Type | Description |
|--------|------|-------------|
| Text | string | The current text content. |
| HorizontalAlignment | string | Horizontal text alignment. |
| VerticalAlignment | string | Vertical text alignment. |
| OverflowMode | string | How overflowing text is handled. |
| FontColor | [uicolor](#uicolor) | The text color (RGBA). |
| FontSize | float | The font size. |
| IsRichText | bool | TRUE if the text supports rich-text markup. |

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| Set | text | Sets the text content. |
| GetFontStyle | - | Returns the font style as a string. |

**Example Usage:**
```lavishscript
echo "Text: ${SomeText.Text}"
echo "Align: ${SomeText.HorizontalAlignment} / ${SomeText.VerticalAlignment}"
echo "Color: ${SomeText.FontColor.R},${SomeText.FontColor.G},${SomeText.FontColor.B},${SomeText.FontColor.A}"
echo "Size: ${SomeText.FontSize}  Rich: ${SomeText.IsRichText}  Style: ${SomeText.GetFontStyle}"

; Set the text (action method — only call deliberately):
; SomeText:Set["Hello"]
```

---

### uiinputfield

A text input field. *(inherits `object`)*

**Access:** returned by `login` input-field members (e.g. `${Login.AccountNameInputField}`).

#### Members

| Member | Type | Description |
|--------|------|-------------|
| IsInteractable | bool | TRUE if the field can currently be edited. |
| IsActive | bool | TRUE if the field is active/visible. |
| IsFocused | bool | TRUE if the field currently has input focus. |
| IsReadOnly | bool | TRUE if the field is read-only. |
| CharacterLimit | int | Maximum number of characters allowed. |
| Text | string | The current text content. |
| ContentType | string | The field's content type. One of: `Standard`, `Autocorrected`, `IntegerNumber`, `DecimalNumber`, `Alphanumeric`, `Name`, `EmailAddress`, `Password`, `Pin`, `Custom`. |

#### Methods

| Method | Parameters | Description |
|--------|-----------|-------------|
| SetText | text | Sets the field's text content. |

**Example Usage:**
```lavishscript
echo "Account field: focused=${Login.AccountNameInputField.IsFocused} readonly=${Login.AccountNameInputField.IsReadOnly}"
echo "  limit=${Login.AccountNameInputField.CharacterLimit} type=${Login.AccountNameInputField.ContentType}"

; Set the text (action method — only call deliberately):
; Login.AccountNameInputField:SetText["myaccount"]
```

---

### pantheon

Reserved game datatype, returned by the `${Pantheon}` TLO.

**Access:** `${Pantheon}`

This datatype is **registered but empty** — it currently exposes no members or methods. It exists as a placeholder for future game-wide information and utilities. Its `ToText` value is `Pantheon`.

> **PLANNED — NOT YET IMPLEMENTED.** Members and methods for `${Pantheon}` are on the roadmap but are not available in the current build. Do not write scripts that depend on this datatype having any members yet.

---

## Registered but Stubbed DataTypes

The following datatypes are **registered** (their type names exist and resolve), but every member and method is currently inactive — they return nothing. They are listed here so scripters know the type names exist, but they must be treated as **planned** until the members are implemented.

### entity

Generic world entity (player, NPC, or object). Backed by an entity id.

> **PLANNED — NOT YET IMPLEMENTED.** The `entity` datatype is registered but exposes no working members yet. Members such as the entity's name, class, race, gender, level, position, health, and so on are planned but not implemented. Names and signatures are provisional. Do not write production scripts against this datatype.

### ability

A character ability/skill.

> **PLANNED — NOT YET IMPLEMENTED.** The `ability` datatype is registered but exposes no working members yet. Do not write production scripts against this datatype.

### quest

A quest entry.

> **PLANNED — NOT YET IMPLEMENTED.** The `quest` datatype is registered but exposes no working members yet. Do not write production scripts against this datatype.

---

## Events

ISXPantheon does not yet fire any game events (spawn/despawn, chat, zoning, combat, etc.). The only event available today is the framework HTTP-response event:

| Event | Parameters | Description |
|-------|-----------|-------------|
| **isxGames_onHTTPResponse** | int Size, string URL, string IPAddress, int ResponseCode, float TransferTime, string ResponseText, string ParsedBody | Fired when a response is received from a `GetURL` or `PostURL` request. |

> **PLANNED — NOT YET IMPLEMENTED.** Game events (entity spawn/despawn, incoming chat text, zoning, and similar) are planned but not wired up in the current build. Do not register handlers for game events yet — they will never fire.

---

## Planned API

> **PLANNED — NOT YET IMPLEMENTED.** Everything in this section describes the intended future ISXPantheon surface. None of it is available in the current build. The names, members, and signatures shown are provisional and WILL change. Do not write production scripts against any of it.

The following are on the ISXPantheon roadmap. These are high-level feature names only — no provisional members, signatures, or return values are shown because none are finalized, and several of these types are not yet registered at all.

- **Reserved top-level objects.** `${Me}` and `${Radar}` exist in the source today only as reserved (commented-out) top-level objects. They are not registered and return nothing.
- **Registered-but-empty datatypes.** `pantheon`, `entity`, `ability`, and `quest` are registered datatypes today but expose no working members yet. As development continues, members are expected to be added to them.
- **Reserved datatype names.** Additional datatype names — including `radar`, `crafting`, and (among others) `char`, `item`, and `effect` — exist in the source as reserved type names without working members.
- **Named roadmap features.** A `Where` command, a `Radar` command, and a `Crafting` TLO/datatype are named on the roadmap.

Additional game-data datatypes and commands are planned. Names and signatures will be added here once they are implemented.

---

## End of API Reference

This reference documents the datatypes, members, methods, commands, and events currently available in the ISXPantheon extension for Pantheon: Rise of the Fallen, plus the planned roadmap surface. As the extension matures, planned sections will be promoted to the implemented reference above.
