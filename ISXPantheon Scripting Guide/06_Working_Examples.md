# ISXPantheon Working Examples

Complete, runnable code examples for common ISXPantheon scripting tasks.

> **NOTE — surface maturity.** The live ISXPantheon surface today is `${ISXPantheon}` (version / `IsReady` / custom variables / currency formatting / rounding), the `GetURL` / `PostURL` commands, and the **login / realm / login-UI** surface via `${Login}` (login state, realm enumeration, and the login screen's buttons and input fields). In-world game-data objects (local player, targets, abilities, inventory, quests, merchants, crafting, loot windows) are **(planned — not yet implemented)**. Every example in the [Working Examples (Live Surface)](#working-examples-live-surface) section runs today; everything in [Planned Examples](#planned-examples) is provided only to show the eventual shape and must not be run yet.

---

## Table of Contents

1. [Working Examples (Live Surface)](#working-examples-live-surface)
   - [Verify the Extension Is Loaded](#verify-the-extension-is-loaded)
   - [Report Extension Version and API Version](#report-extension-version-and-api-version)
   - [Custom Variables as Cross-Script State](#custom-variables-as-cross-script-state)
   - [Format Currency](#format-currency)
   - [Rounding Helper](#rounding-helper)
   - [Fetch a Web Resource with GetURL](#fetch-a-web-resource-with-geturl)
   - [Post Data with PostURL](#post-data-with-posturl)
   - [A Robust Script Skeleton](#a-robust-script-skeleton)
   - [Login Screen (State, Realms, Buttons, Fields)](#login-screen-state-realms-buttons-fields)
2. [Planned Examples](#planned-examples)
   - [Character Information (Planned)](#character-information-planned)
   - [Inventory, Abilities, Combat (Planned)](#inventory-abilities-combat-planned)
   - [Event-Driven Scripts (Planned)](#event-driven-scripts-planned)

---

## Working Examples (Live Surface)

These examples use only the surface that exists in the current build. They are runnable as-is.

### Verify the Extension Is Loaded

```lavishscript
function main()
{
    ; Confirm the extension is present and finished loading
    if !${ISXPantheon(exists)}
    {
        echo "ISXPantheon is not loaded. Run: extension isxpantheon"
        return
    }

    while !${ISXPantheon.IsReady}
        wait 10

    echo "ISXPantheon is ready."
}
```

### Report Extension Version and API Version

```lavishscript
function main()
{
    while !${ISXPantheon.IsReady}
        wait 10

    echo "========================================="
    echo "  ISXPantheon"
    echo "========================================="
    echo "Version:     ${ISXPantheon.Version}"
    echo "API Version: ${ISXPantheon.APIVersion}"
    echo "========================================="
}
```

### Custom Variables as Cross-Script State

Custom variables are a real, built-in key/value store on `${ISXPantheon}`. They are useful for passing state between scripts or persisting flags during a session.

```lavishscript
function main()
{
    while !${ISXPantheon.IsReady}
        wait 10

    ; Write some typed values
    ISXPantheon:SetCustomVariable[Mode,"Aggressive"]
    ISXPantheon:SetCustomVariable[MaxRange,40]
    ISXPantheon:SetCustomVariable[Enabled,TRUE]

    ; Read them back with the appropriate type
    echo "Mode:     ${ISXPantheon.GetCustomVariable[Mode]}"
    echo "MaxRange: ${ISXPantheon.GetCustomVariable[MaxRange,int]}"
    echo "Enabled:  ${ISXPantheon.GetCustomVariable[Enabled,bool]}"

    ; Clear everything when done
    ISXPantheon:ClearAllCustomVariables
}
```

### Format Currency

`GetCurrencyString` formats an amount (expressed in the smallest unit) into a human-readable currency string.

```lavishscript
function main()
{
    while !${ISXPantheon.IsReady}
        wait 10

    variable int Amount = 123456

    echo "Formatted: ${ISXPantheon.GetCurrencyString[${Amount}]}"
}
```

### Rounding Helper

`Round` rounds a value to the nearest multiple, with a selectable numeric type.

```lavishscript
function main()
{
    while !${ISXPantheon.IsReady}
        wait 10

    ; Round 47 to the nearest multiple of 10 (-> 50)
    echo "Rounded int:   ${ISXPantheon.Round[int,47,10]}"

    ; Round a float to the nearest 0.25
    echo "Rounded float: ${ISXPantheon.Round[float,3.30,0.25]}"
}
```

### Fetch a Web Resource with GetURL

`GetURL` performs an HTTP GET. This is fully functional today and is handy for pulling configuration, version checks, or remote data.

```lavishscript
function main()
{
    while !${ISXPantheon.IsReady}
        wait 10

    ; Issue the GET request
    GetURL "https://example.com/config.txt"

    echo "GetURL request issued."
}
```

### Post Data with PostURL

`PostURL` performs an HTTP POST. Use it to report status to a web endpoint, submit telemetry, or drive a remote service.

```lavishscript
function main()
{
    while !${ISXPantheon.IsReady}
        wait 10

    ; Issue the POST request with a body
    PostURL "https://example.com/report" "status=running&session=1"

    echo "PostURL request issued."
}
```

### A Robust Script Skeleton

A minimal but defensive entry point: confirm the extension exists, wait for it to be ready, run a throttled main loop, and clean up. This is the recommended starting template for any ISXPantheon script today.

```lavishscript
;*****************************************************
; SkeletonScript.iss
;*****************************************************

declare Running bool script TRUE
declare LastPulse int64 script 0
declare PULSE_INTERVAL int script 500

function main()
{
    ; Confirm the extension is present
    if !${ISXPantheon(exists)}
    {
        echo "ISXPantheon not loaded. Run: extension isxpantheon"
        return
    }

    ; Wait until it has finished loading
    while !${ISXPantheon.IsReady}
        wait 10

    echo "Script started (ISXPantheon ${ISXPantheon.Version})"

    ; Throttled main loop
    while ${Running}
    {
        if ${Script.RunningTime} >= ${Math.Calc64[${LastPulse}+${PULSE_INTERVAL}]}
        {
            LastPulse:Set[${Script.RunningTime}]
            call Pulse
        }
        wait 10
    }

    echo "Script stopped"
}

function Pulse()
{
    ; Periodic work goes here
}
```

### Login Screen (State, Realms, Buttons, Fields)

Read-only dump of the live `${Login}` surface — login state, account fields, the realm list, every login button (with its label's full `uitext` styling), and both input fields. Safe to run at the login / realm-selection screen; it won't log you in. The login-flow and UI-driving methods at the bottom are commented out — uncomment only the ones you want to fire.

`NumRealms` / `Realm[#]` only return data once you've reached the realm-selection screen (`State` == `WaitingForPlayerToSelectRealm`), so the loop is skipped otherwise.

```lavishscript
function main()
{
    ; ---- Guards -------------------------------------------------
    if !${ISXPantheon(exists)}
    {
        echo "ISXPantheon not loaded."
        return
    }

    while !${ISXPantheon.IsReady}
        wait 10

    if !${Login(exists)}
    {
        echo "The Login object is not available right now."
        echo "(This script only works at the login / realm-selection screen.)"
        return
    }

    ; ---- Login state and account fields -------------------------
    echo "========================================="
    echo "  Login Screen"
    echo "========================================="
    echo "State:             ${Login.State}"
    echo "AccountName:       ${Login.AccountName}"
    variable string AccountPassword = ${Login.AccountPassword}
    if (${AccountPassword.Length} == 0)
    {
        echo "AccountPassword:   (nothing is typed / write-only state)"
    }
    else
    {
        echo "AccountPassword:   ${AccountPassword}"
    }
    echo "SelectedRealmName: ${Login.SelectedRealmName}"
    echo "CurrentRealmName:  ${Login.CurrentRealmName}"

    variable int RealmCount = ${Login.NumRealms}
    if (${RealmCount} > 0)
    {
        echo "NumRealms:         ${RealmCount}"
        variable int i
        for (i:Set[1] ; ${i} <= ${RealmCount} ; i:Inc)
        {
            echo "-- Realm ${i} ---------------------------------"
            echo "  Name:     ${Login.Realm[${i}].Name}"
            echo "  Address:  ${Login.Realm[${i}].Address}"
            echo "  Location: ${Login.Realm[${i}].Location}"
            echo "  Version:  ${Login.Realm[${i}].Version}"
            echo "  Load:     ${Login.Realm[${i}].Load}"
            echo "  IsFull:   ${Login.Realm[${i}].IsFull}"
        }
    }
    else
    {
        echo "(Realm list is only available after viewing the list of realms at least once (i.e., when Login.State.Equal[WaitingForPlayerToSelectRealm]))"
    }

    ; ---- Buttons ------------------------------------------------
    call DumpButton "LoginButton"
    call DumpButton "QuitButton"
    call DumpButton "CalibrateButton"
    call DumpButton "EulaAcceptButton"
    call DumpButton "RealmBackButton"
    call DumpButton "RealmQuitButton"

    ; ---- Input fields -------------------------------------------
    call DumpInputField "AccountNameInputField"
    call DumpInputField "AccountPasswordInputField"

    echo "========================================="
    echo "  Login Screen complete (read-only)."
    echo "========================================="

    ; Uncomment ONLY if you actually want to test login flow.
    ; Running them will type credentials and/or log you in.
    ;
    ;   Login:SetAccountName["myaccount"]
    ;   Login:SetAccountPassword["mypassword"]
    ;   Login:Login
    ;
    ;   Login:SetRealm["My Realm Name"]
    ;   Login:EnterWorld
    ;
    ;   And here are some methods that can be used with buttons/inputfields if you want to manipulate UI elements directly
    ;   Login.LoginButton:Press
    ;   Login.AccountNameInputField:SetText["myaccount"]
}

function DumpButton(string Which)
{
    variable index:string FontStyles
    variable iterator It
    variable string FontStyle

    Login.${Which}.Label:GetFontStyle[FontStyles]
    FontStyles:GetIterator[It]
    if ${It:First(exists)}
    {
        do
        {
            FontStyle:Concat["${It.Value} "]
        }
        while ${It:Next(exists)}
    }

    echo "-- Button: ${Which}"
    echo "  IsInteractable: ${Login.${Which}.IsInteractable}"
    echo "  IsActive:       ${Login.${Which}.IsActive}"
    echo "  Label: \"${Login.${Which}.Label.Text}\""
    echo "     - HAlign:         ${Login.${Which}.Label.HorizontalAlignment}"
    echo "     - VAlign:         ${Login.${Which}.Label.VerticalAlignment}"
    echo "     - Overflow:       ${Login.${Which}.Label.OverflowMode}"
    echo "     - FontColor:      R=${Login.${Which}.Label.FontColor.R} G=${Login.${Which}.Label.FontColor.G} B=${Login.${Which}.Label.FontColor.B} A=${Login.${Which}.Label.FontColor.A}"
    echo "     - FontSize:       ${Login.${Which}.Label.FontSize}"
    echo "     - IsRichText:     ${Login.${Which}.Label.IsRichText}"
    echo "     - FontStyle:      ${FontStyle}"
}

function DumpInputField(string Which)
{
    echo "-- InputField: ${Which}"
    echo "  IsInteractable: ${Login.${Which}.IsInteractable}"
    echo "  IsActive:       ${Login.${Which}.IsActive}"
    echo "  IsFocused:      ${Login.${Which}.IsFocused}"
    echo "  IsReadOnly:     ${Login.${Which}.IsReadOnly}"
    echo "  CharacterLimit: ${Login.${Which}.CharacterLimit}"
    echo "  Text:           ${Login.${Which}.Text}"
    echo "  ContentType:    ${Login.${Which}.ContentType}"
}
```

---

## Planned Examples

> **PLANNED — NOT YET IMPLEMENTED.** Everything in this section depends on game-data objects that are on the ISXPantheon roadmap but are **not available in the current build**. The code is provisional and the member names, signatures, and event names shown are illustrative only — they WILL change. Do not write production scripts against them yet. The patterns are included so you can see the intended shape of future scripts and structure your code accordingly.

### Character Information (Planned)

Once the local player object is surfaced, a stats dump will follow roughly this shape. The exact member names are not finalized.

```lavishscript
; PLANNED - illustrative only, not runnable today
function ShowCharacterStats()
{
    while !${ISXPantheon.IsReady}
        wait 10

    ; A future local-player object is planned, but its members are not
    ; defined yet. Whatever it exposes would be read here. The names,
    ; member set, and structure are all TBD and intentionally not shown.
    echo "<local-player details, once the object is surfaced>"
}
```

### Inventory, Abilities, Combat (Planned)

Inventory, ability, and entity objects are planned but not surfaced yet, and their TLOs, members, and methods are not defined. When they exist, scripts that act on game data will follow the standard ISX guard pattern — check `(exists)`, check a readiness flag, then act — using whatever objects the extension surfaces. The skeleton below shows that general guard shape without naming any specific (and unconfirmed) object, member, or method.

```lavishscript
; PLANNED - illustrative only, not runnable today
function ActOnGameData()
{
    while !${ISXPantheon.IsReady}
        wait 10

    ; General guard shape once game-data objects are surfaced:
    ;   if the object exists and is ready, act on it.
    ; The actual object names, members, and methods are TBD.
}

; PLANNED - a combat pulse is expected to resolve a target entity
; (exists / not dead / in range) and dispatch abilities in priority
; order. Entity and ability objects are not surfaced yet.
function CombatPulse()
{
    ; ... resolve target, then act on it ...
}
```

### Event-Driven Scripts (Planned)

ISXPantheon does not yet fire game events (incoming chat, spawn/despawn, window-appeared, zoning, quest-offered, craft-round). When events are surfaced, the attach/handle/detach pattern below is what you will use. General LavishScript event mechanics (`Event[...]:AttachAtom`, `atom` handlers) are real and usable today with platform/script events.

```lavishscript
; PLANNED - illustrative only, not runnable today
function main()
{
    while !${ISXPantheon.IsReady}
        wait 10

    ; Expected shape once game events exist:
    Event[SomeGameEvent]:AttachAtom[OnSomeEvent]

    wait 99999999

    Event[SomeGameEvent]:DetachAtom[OnSomeEvent]
}

atom OnSomeEvent(string Arg)
{
    ; React to the event
}
```

---

**The Live Surface examples above run today. The Planned examples illustrate intended future shapes only -- always re-check the API Reference for the current, real surface before writing a script.**
