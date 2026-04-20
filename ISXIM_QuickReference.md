# ISXIM Reference

## Table of Contents

1. [Introduction](#introduction)
2. [Top-Level Objects](#top-level-objects)
   - [Extension TLOs](#extension-tlos)
   - [IRC TLOs](#irc-tlos)
3. [DataTypes](#datatypes)
   - [Extension DataTypes](#extension-datatypes)
     - [im](#im)
   - [IRC DataTypes](#irc-datatypes)
     - [irc](#irc)
     - [ircuser](#ircuser)
     - [channel](#channel)
     - [nick](#nick)
4. [Commands](#commands)
5. [Events](#events)
   - [Event Registration](#event-registration)
   - [IRC Events](#irc-events)
   - [Extension Events](#extension-events)
6. [Usage Examples](#usage-examples)
   - [IRC Connection and Authentication](#irc-connection-and-authentication)
   - [Joining and Leaving Channels](#joining-and-leaving-channels)
   - [Sending Messages](#sending-messages)
   - [Working with Channel Members](#working-with-channel-members)
   - [Event Handling](#event-handling)
7. [Notes](#notes)
   - [Case Sensitivity](#case-sensitivity)
   - [NULL Checks](#null-checks)
   - [Parameter Notation](#parameter-notation)
   - [IRC Connection Sequence](#irc-connection-sequence)
   - [Known Issues and Gotchas](#known-issues-and-gotchas)
   - [CTCP Auto-Responses](#ctcp-auto-responses)
   - [Message Sanitization](#message-sanitization)
8. [Deprecated Features](#deprecated-features)
   - [Removed TLOs and DataTypes](#removed-tlos-and-datatypes)
   - [Removed Members](#removed-members)
   - [Removed Events](#removed-events)

---

## Introduction

This document provides reference documentation for all datatypes, top-level objects, commands, and events exposed by the ISXIM extension. ISXIM extends LavishScript to provide IRC (Internet Relay Chat) client functionality.

### DataType Inheritance

ISXIM uses a flat type hierarchy — IRC datatypes do not inherit from each other.

### Accessing DataTypes

DataTypes are accessed through Top-Level Objects (TLOs) or through members of other datatypes:

```lavishscript
${IM}                                                 // TLO returning 'im' datatype
${IRC}                                                // TLO returning 'irc' datatype
${IRCUser[MyNick]}                                    // TLO with parameter returning 'ircuser' datatype
${IRCUser[MyNick].Channel[#general]}                  // Member returning 'channel' datatype
${IRCUser[MyNick].Channel[#general].Nick[SomeUser]}   // Member returning 'nick' datatype
```

---

## Top-Level Objects

Top-Level Objects (TLOs) are the entry points for accessing extension functionality. They can be accessed directly in LavishScript.

### Extension TLOs

| TLO | DataType | Description |
|-----|----------|-------------|
| IM | [im](#im) | ISXIM extension information and utilities |

### IRC TLOs

| TLO | DataType | Description |
|-----|----------|-------------|
| IRC | [irc](#irc) | Main IRC functionality |
| IRCUser[#] | [ircuser](#ircuser) | IRC user by 1-based index (1 to IRC.NumUsers) |
| IRCUser[nickname] | [ircuser](#ircuser) | IRC user by exact nickname match |

---

## DataTypes

### Extension DataTypes

#### **im**

Extension information and utilities. Accessed via the `IM` TLO.

**Members:**

| Member | Type | Description |
|--------|------|-------------|
| IsReady | bool | TRUE when extension has completed loading |
| Version | string | ISXIM product version string |
| InQuietMode | bool | TRUE when extension is suppressing informational output |

**Methods:**

| Method | Description |
|--------|-------------|
| QuietMode[on] | Enable quiet mode (suppresses most extension printf output) |
| QuietMode[off] | Disable quiet mode |

**Example:**
```lavishscript
if !${IM.IsReady}
{
    echo "ISXIM is not ready yet"
    return
}

echo "ISXIM Version: ${IM.Version}"

; Suppress extension output
IM:QuietMode[on]
```

---

### IRC DataTypes

#### **irc**

Global IRC datatype providing the entry point for creating IRC connections. Accessed via the `IRC` TLO.

**Members:**

| Member | Type | Description |
|--------|------|-------------|
| NumUsers | int | Number of IRCUser sessions currently tracked (connected or connecting) |
| CWD | string | Current working directory used by the extension (trailing backslash included) |

**Methods:**

| Method | Description |
|--------|-------------|
| Connect[server,nickname] | Connect to IRC server (default port 6667) |
| Connect[server,nickname,port] | Connect on specified port |
| Connect[server,nickname,port,password] | Connect with server password (PASS) |

**Notes:**
- `Connect` adds the `ircuser` to the user list **immediately** and returns. The user starts with `IsConnecting = TRUE` and `IsConnected = FALSE`. See [IRC Connection Sequence](#irc-connection-sequence).
- Attempting to `Connect` with a nickname that is already in use by another `ircuser` session in this extension instance will be rejected.
- There is no `IRC:Disconnect`. Use `IRCUser[name]:Disconnect` on a specific connection.

**Example:**
```lavishscript
IRC:Connect[irc.example.com,MyNickname]
IRC:Connect[irc.example.com,MyNickname,6667]
IRC:Connect[irc.example.com,MyNickname,6667,serverpass]

echo "Tracked IRC users: ${IRC.NumUsers}"
echo "ISXIM CWD: ${IRC.CWD}"
```

**See Also:** [ircuser](#ircuser)

---

#### **ircuser**

Represents a single IRC client session (one connection to one server). Accessed via the `IRCUser` TLO or as a return from `IRC:Connect`.

**Members:**

*Identity:*

| Member | Type | Description |
|--------|------|-------------|
| MyNick | string | Your nickname on this connection |
| Server | string | Server address (as passed to Connect) |
| Port | int | Server port |
| ID | int | Internal thread ID for this connection (not a persistent/stable identifier) |

*Status:*

| Member | Type | Description |
|--------|------|-------------|
| IsConnecting | bool | TRUE while the socket/handshake is in progress |
| IsConnected | bool | TRUE once the connection is established and usable |

*Channels:*

| Member | Returns | Description |
|--------|---------|-------------|
| NumChannelsIn | int | Number of channels this user is currently in |
| Channel[#] | [channel](#channel) | Channel by 1-based index |
| Channel[name] | [channel](#channel) | Channel by exact (case-insensitive) name match |

*Users (searches across all joined channels):*

| Member | Returns | Description |
|--------|---------|-------------|
| Nick[name] | [nick](#nick) | Nick by exact name match, searched across all joined channels |

**Methods:**

| Method | Description |
|--------|-------------|
| Disconnect | QUIT with a default reason ("Disconnecting...") |
| Disconnect[reason] | QUIT with a custom reason string |
| Join[channel] | JOIN a channel. Channel name must begin with `#` or `&` |
| Join[channel,key] | JOIN a password-protected channel |
| PM[to,message] | PRIVMSG to a nick or channel |
| ChangeNickTo[newnick] | NICK change |
| Emote[to,message] | Send a CTCP ACTION to a nick or channel |
| SendRaw[string] | Send a raw line to the server (no alterations; `\n` is appended automatically) |
| Notice[to,message] | See [Known Issues and Gotchas](#known-issues-and-gotchas) — this method is registered but not functional in the current source. Use `SendRaw` to send raw `NOTICE` lines. |

**Connection Guard:**
All methods on `ircuser` (except accessing members) silently no-op if the user is still connecting or is not yet connected — i.e., commands are ignored when `IsConnecting` is TRUE or `IsConnected` is FALSE. Always wait for connection to complete before issuing methods.

**Important: Join Latency:**
The `Join` method requires server round-trips. After calling it, wait at least `wait 25` (or longer on high-latency links) before assuming the channel is populated. The channel object is created immediately when `Join` is called; it populates with nicks, topic, etc. as the server responds.

**Example:**
```lavishscript
; Wait for connection to complete
IRC:Connect[irc.example.com,MyNick]
do
{
    wait 3
}
while (${IRCUser[MyNick](exists)} && ${IRCUser[MyNick].IsConnecting})

if !${IRCUser[MyNick].IsConnected}
{
    echo "Connection failed"
    return
}

; Join a channel (requires wait time)
IRCUser[MyNick]:Join[#general]
wait 25

; Send messages
IRCUser[MyNick]:PM[#general,Hello everyone!]
IRCUser[MyNick]:PM[SomeUser,Private message]
IRCUser[MyNick]:Emote[#general,waves hello]
IRCUser[MyNick]:ChangeNickTo[NewName]

; Send a raw IRC command
IRCUser[MyNick]:SendRaw[WHOIS SomeUser]

; Disconnect with reason
IRCUser[MyNick]:Disconnect[Goodbye!]
```

**See Also:** [channel](#channel), [nick](#nick)

---

#### **channel**

Represents a joined IRC channel. Accessed via `ircuser.Channel`.

**Members:**

| Member | Type | Description |
|--------|------|-------------|
| Name | string | Channel name (including the `#` or `&` prefix) |
| Topic | string | Current channel topic |
| TopicBy | string | Who set the current topic |
| NumNicks | int | Number of users in the channel |
| Nick[#] | [nick](#nick) | Nick by 1-based index |
| Nick[name] | [nick](#nick) | Nick by exact name match |

**Methods:**

| Method | Description |
|--------|-------------|
| Leave | PART the channel and clean up internal state |
| Say[message] | PRIVMSG to this channel |
| SetMode[arguments] | MODE change on this channel — arguments are appended to `MODE <channel>` |
| GetBans[variable] | Populate an `index:string` variable with the cached ban list for this channel |

**Example:**
```lavishscript
variable channel MyChannel
MyChannel:Set[${IRCUser[MyNick].Channel[#general]}]

echo "Channel: ${MyChannel.Name}"
echo "Topic: ${MyChannel.Topic}"
echo "Set by: ${MyChannel.TopicBy}"
echo "Users: ${MyChannel.NumNicks}"

; Send message to channel
MyChannel:Say[Hello everyone!]

; Get ban list
variable index:string BanList
MyChannel:GetBans[BanList]

; Set a channel mode
MyChannel:SetMode[+i]

; Leave
MyChannel:Leave
```

**Note on Channel Mode Queries:**
The C++ source additionally defines `IsSet[mode]`, `Limit`, and `Password` members in its internal enum, and the corresponding `GetMember` switch handles them. However, these members are **not registered** with LavishScript via `TypeMember` in the current source build and therefore **are not accessible from scripts**. To observe channel mode changes, subscribe to the [IRC_ChannelModeChange](#irc_channelmodechange) event. See [Known Issues and Gotchas](#known-issues-and-gotchas).

**See Also:** [ircuser](#ircuser), [nick](#nick)

---

#### **nick**

Represents a user in an IRC channel. Accessed via `channel.Nick` or `ircuser.Nick`.

**Members:**

| Member | Type | Description |
|--------|------|-------------|
| Name | string | Nickname |
| Type | string | Privilege level in the channel (see below) |

**Valid Type values:** `Owner`, `SOP`, `OP`, `HOP`, `Voice`, `Normal`

**Methods:**

| Method | Description |
|--------|-------------|
| PM[message] | PRIVMSG directly to this user |
| SetMode[arguments] | MODE change targeting this user in the parent channel; arguments are in the form e.g. `+o`, `-v`, etc. (ISXIM constructs `MODE <channel> <args> <nick>`) |

**Example:**
```lavishscript
variable nick ChannelUser
ChannelUser:Set[${IRCUser[MyNick].Channel[#general].Nick[SomeUser]}]

echo "Nickname: ${ChannelUser.Name}"
echo "Type: ${ChannelUser.Type}"

; Send a private message
ChannelUser:PM[Hello there!]

; Check privilege level
if ${ChannelUser.Type.Equal[OP]} || ${ChannelUser.Type.Equal[SOP]} || ${ChannelUser.Type.Equal[Owner]}
{
    echo "This user is an operator or higher"
}

; Op someone (you must have the required privilege)
ChannelUser:SetMode[+o]
```

**See Also:** [channel](#channel), [ircuser](#ircuser)

---

## Commands

ISXIM does not register any custom LavishScript commands. All functionality is exposed through the TLOs and datatypes above.

---

## Events

### Event Registration

ISXIM events use standard LavishScript event registration. Register an atom with `Event[name]:AttachAtom[atomname]` and remove it with `DetachAtom` when no longer needed.

**Example Event Registration:**
```lavishscript
atom OnIRCMessage(string User, string Channel, string From, string Message)
{
    echo "Message in ${Channel} from ${From}: ${Message}"
}

Event[IRC_ReceivedChannelMsg]:AttachAtom[OnIRCMessage]

; Later: detach when no longer needed
Event[IRC_ReceivedChannelMsg]:DetachAtom[OnIRCMessage]
```

**Event names are case-sensitive.** All events below are written using their exact registered name. Pay particular attention to [IRC_ReceivedNOTICE](#irc_receivednotice), which uses an uppercase suffix.

### IRC Events

In every event below, the `User` parameter is your own nickname (i.e., the nickname of the `ircuser` that observed the event). This lets a single atom dispatch correctly when your script holds multiple simultaneous IRC sessions.

---

#### **IRC_ReceivedNOTICE**

Fires when a NOTICE message is received. Note the all-caps `NOTICE` — this is the exact registered event name.

**Parameters:**
- `User` — string: Your nickname
- `From` — string: Who sent the notice
- `To` — string: Notice recipient (you or a channel)
- `Message` — string: Notice text

**Example:**
```lavishscript
atom OnIRCNotice(string User, string From, string To, string Message)
{
    echo "Notice from ${From} to ${To}: ${Message}"
}

Event[IRC_ReceivedNOTICE]:AttachAtom[OnIRCNotice]
```

---

#### **IRC_ReceivedChannelMsg**

Fires when a channel message is received.

**Parameters:**
- `User` — string: Your nickname
- `Channel` — string: Channel where the message was sent
- `From` — string: Who sent the message
- `Message` — string: Message text

---

#### **IRC_ReceivedPrivateMsg**

Fires when a private message (PM) is received.

**Parameters:**
- `User` — string: Your nickname
- `From` — string: Who sent the message
- `To` — string: Message recipient (you)
- `Message` — string: Message text

---

#### **IRC_ReceivedCTCP**

Fires when a CTCP (Client-To-Client Protocol) request is received. ISXIM automatically responds to: `VERSION`, `FINGER`, `PING`, `TIME`, `USERINFO`, and `CLIENTINFO`. This event fires **in addition** to the auto-response so your script can observe the incoming CTCP.

**Parameters:**
- `User` — string: Your nickname
- `From` — string: Who sent the CTCP
- `To` — string: CTCP recipient
- `Message` — string: CTCP command/payload

---

#### **IRC_ReceivedEmote**

Fires when an emote/ACTION message is received.

**Parameters:**
- `User` — string: Your nickname
- `From` — string: Who sent the emote
- `To` — string: Emote recipient (you or a channel)
- `Message` — string: Emote text

---

#### **IRC_NickJoinedChannel**

Fires when a user joins a channel you are in.

**Parameters:**
- `User` — string: Your nickname
- `Channel` — string: Channel joined
- `WhoJoined` — string: Nickname of the user who joined

---

#### **IRC_NickLeftChannel**

Fires when a user PARTs a channel you are in.

**Parameters:**
- `User` — string: Your nickname
- `Channel` — string: Channel left
- `WhoLeft` — string: Nickname of the user who left

---

#### **IRC_NickQuit**

Fires when a user quits IRC entirely (QUIT).

**Parameters:**
- `User` — string: Your nickname
- `Channel` — string: Channel where the quit was observed
- `Nick` — string: Nickname of the user who quit
- `Reason` — string: Quit message

**Note:** If the quitting user shared multiple channels with you, this event may fire multiple times (once per channel).

---

#### **IRC_TopicSet**

Fires when a channel topic is set or changed.

**Parameters:**
- `User` — string: Your nickname
- `Channel` — string: Channel whose topic changed
- `NewTopic` — string: New topic text
- `TopicSetBy` — string: Who set the topic

---

#### **IRC_NickChanged**

Fires when a user (including possibly yourself) changes nickname.

**Parameters:**
- `User` — string: Your nickname
- `OldNick` — string: Previous nickname
- `NewNick` — string: New nickname

---

#### **IRC_KickedFromChannel**

Fires when someone is KICKed from a channel (yourself or anyone else).

**Parameters:**
- `User` — string: Your nickname
- `Channel` — string: Channel where the kick occurred
- `WhoKicked` — string: Nickname of the kicked user
- `KickedBy` — string: Who performed the kick
- `Reason` — string: Kick reason

---

#### **IRC_PRIVMSGErrorResponse**

Fires when a PRIVMSG you sent fails.

**Parameters:**
- `User` — string: Your nickname
- `ErrorType` — string: See values below
- `To` — string: Intended recipient
- `Response` — string: Raw server error response

**Valid ErrorType values:**
- `NO_SUCH_NICKORCHANNEL` — Recipient does not exist
- `NO_EXTERNAL_MSGS_ALLOWED` — Channel does not allow external (non-member) messages

---

#### **IRC_JOINErrorResponse**

Fires when a JOIN you issued fails.

**Parameters:**
- `User` — string: Your nickname
- `ErrorType` — string: See values below
- `Channel` — string: Channel you tried to join
- `Response` — string: Raw server error response

**Valid ErrorType values:**
- `BANNED` — You are banned from the channel
- `MUST_BE_REGISTERED` — Channel requires a registered nick
- `REQUIRES_KEY` — Channel requires a password (key)

---

#### **IRC_NickTypeChange**

Fires when a user's privilege level changes in a channel (opped, deopped, voiced, etc.).

**Parameters:**
- `User` — string: Your nickname
- `Channel` — string: Channel where the change occurred
- `NickName` — string: User whose type changed
- `NickType` — string: See values below
- `Toggle` — string: `"TRUE"` if granted, `"FALSE"` if removed
- `WhoSet` — string: Who made the change

**Valid NickType values:** `OWNER`, `SOP`, `OP`, `HOP`, `Voice`, `Normal`

---

#### **IRC_ChannelModeChange**

Fires when a channel mode changes.

**Parameters:**
- `User` — string: Your nickname
- `Channel` — string: Channel whose mode changed
- `ModeType` — string: See values below
- `Toggle` — string: `"TRUE"` if enabled, `"FALSE"` if disabled
- `WhoSet` — string: Who made the change
- `Extra` — string: Additional context — password for `PASSWORD`, limit for `LIMIT`

**Valid ModeType values:**
- `PASSWORD` — Channel password set/cleared
- `LIMIT` — User limit set/cleared
- `SECRET` — +s
- `PRIVATE` — +p
- `INVITEONLY` — +i
- `MODERATED` — +m
- `NOEXTERNALMSGS` — +n
- `ONLYOPSCHANGETOPIC` — +t
- `REGISTERED` — Channel is registered
- `REGISTRATIONREQ` — Registration required
- `NOCOLORSALLOWED` — +c

---

#### **IRC_AddChannelBan**

Fires when a ban mask is added to a channel.

**Parameters:**
- `User` — string: Your nickname
- `Channel` — string: Channel where the ban was added
- `WhoSet` — string: Who set the ban
- `Ban` — string: Ban mask

---

#### **IRC_RemoveChannelBan**

Fires when a ban mask is removed from a channel.

**Parameters:**
- `User` — string: Your nickname
- `Channel` — string: Channel where the ban was removed
- `WhoSet` — string: Who removed the ban
- `Ban` — string: Ban mask that was removed

---

#### **IRC_UnhandledEvent**

Fires for IRC protocol messages that ISXIM does not specifically handle. Useful as a fallback for raw IRC numerics and other server messages.

**Parameters:**
- `User` — string: Your nickname
- `Command` — string: IRC command or numeric received
- `Param` — string: Command parameters
- `Rest` — string: Remaining data (trailing parameter)

---

#### **IRC_UserDisconnected**

Fires when your IRC user is disconnected from the server (whether voluntarily or not).

**Parameters:**
- `User` — string: Your nickname
- `Reason` — string: Disconnect reason (set by `Disconnect[reason]` or by ISXIM internally)
- `OptionalServerResponse` — string: Optional server-supplied message

---

### Extension Events

#### **ISXIM_onInstanceReloadingAfterUpdate**

Fires when an ISX extension is updated and another InnerSpace session is reloading after patch. If auto-reload is enabled, ISXIM uses this to reload itself 3 seconds later when the update occurred in a different session.

**Parameters:**
- `SessionPatched` — string: Name of the InnerSpace session that received the patch

**Note:** Scripts generally do not need to attach to this event — it is handled internally. It is documented here for completeness.

---

## Usage Examples

### IRC Connection and Authentication

```lavishscript
; Basic IRC connection
function main()
{
    if !${IM.IsReady}
    {
        echo "ISXIM is not ready yet"
        return
    }

    echo "ISXIM Version: ${IM.Version}"

    echo "Connecting to IRC..."
    IRC:Connect[irc.example.com,MyNickname]

    ; Wait for connection to complete
    do
    {
        wait 3
    }
    while (${IRCUser[MyNickname](exists)} && ${IRCUser[MyNickname].IsConnecting})

    if !${IRCUser[MyNickname](exists)}
    {
        echo "Failed to connect — user no longer exists"
        return
    }

    if ${IRCUser[MyNickname].IsConnected}
    {
        echo "Connected to ${IRCUser[MyNickname].Server}:${IRCUser[MyNickname].Port}"
        echo "Your nickname: ${IRCUser[MyNickname].MyNick}"
    }
}
```

### Joining and Leaving Channels

```lavishscript
function JoinChannels()
{
    variable string MyNick = "MyNickname"

    echo "Joining #general..."
    IRCUser[${MyNick}]:Join[#general]
    wait 25

    echo "Joining #private..."
    IRCUser[${MyNick}]:Join[#private,secretkey]
    wait 25

    echo "Joined ${IRCUser[${MyNick}].NumChannelsIn} channels"

    variable int i
    for (i:Set[1] ; ${i} <= ${IRCUser[${MyNick}].NumChannelsIn} ; i:Inc)
    {
        echo "Channel ${i}: ${IRCUser[${MyNick}].Channel[${i}].Name}"
    }
}

function LeaveChannel(string ChannelName)
{
    variable string MyNick = "MyNickname"

    if ${IRCUser[${MyNick}].Channel[${ChannelName}](exists)}
    {
        echo "Leaving ${ChannelName}..."
        IRCUser[${MyNick}].Channel[${ChannelName}]:Leave
    }
    else
    {
        echo "Not in channel ${ChannelName}"
    }
}
```

### Sending Messages

```lavishscript
function SendMessages()
{
    variable string MyNick = "MyNickname"

    ; Channel message
    IRCUser[${MyNick}]:PM[#general,Hello everyone!]

    ; Private message
    IRCUser[${MyNick}]:PM[SomeUser,Hi there!]

    ; Emote/action
    IRCUser[${MyNick}]:Emote[#general,waves hello]

    ; Raw IRC command (e.g., NOTICE, since ircuser:Notice is non-functional)
    IRCUser[${MyNick}]:SendRaw[NOTICE SomeUser :This is a notice]

    ; Use the channel object directly
    variable channel MyChannel
    MyChannel:Set[${IRCUser[${MyNick}].Channel[#general]}]
    MyChannel:Say[Hello via channel object]
}

; Respond to chat commands
atom OnChannelMsg(string User, string Channel, string From, string Message)
{
    if ${Message.Equal[!hello]}
    {
        IRCUser[${User}]:PM[${Channel},Hello ${From}!]
    }

    if ${Message.Equal[!time]}
    {
        IRCUser[${User}]:PM[${Channel},The time is ${Time}]
    }
}

Event[IRC_ReceivedChannelMsg]:AttachAtom[OnChannelMsg]
```

### Working with Channel Members

```lavishscript
function ListChannelMembers(string ChannelName)
{
    variable string MyNick = "MyNickname"
    variable channel MyChannel

    MyChannel:Set[${IRCUser[${MyNick}].Channel[${ChannelName}]}]

    if !${MyChannel(exists)}
    {
        echo "Not in channel ${ChannelName}"
        return
    }

    echo "Members of ${MyChannel.Name} (${MyChannel.NumNicks} total):"
    echo "Topic: ${MyChannel.Topic}"
    echo "Topic set by: ${MyChannel.TopicBy}"
    echo ""

    variable int i
    variable nick CurrentNick

    for (i:Set[1] ; ${i} <= ${MyChannel.NumNicks} ; i:Inc)
    {
        CurrentNick:Set[${MyChannel.Nick[${i}]}]
        echo "[${CurrentNick.Type}] ${CurrentNick.Name}"
    }
}

function IsUserOp(string ChannelName, string UserName)
{
    variable string MyNick = "MyNickname"
    variable nick U

    U:Set[${IRCUser[${MyNick}].Channel[${ChannelName}].Nick[${UserName}]}]

    if !${U(exists)}
        return FALSE

    if ${U.Type.Equal[OP]} || ${U.Type.Equal[SOP]} || ${U.Type.Equal[Owner]}
        return TRUE

    return FALSE
}
```

### Event Handling

```lavishscript
; Complete IRC bot example
variable string gMyNick = "MyBot"
variable bool gRunning = TRUE

function main()
{
    RegisterEvents

    echo "Connecting..."
    IRC:Connect[irc.example.com,${gMyNick}]

    do
    {
        wait 3
    }
    while (${IRCUser[${gMyNick}](exists)} && ${IRCUser[${gMyNick}].IsConnecting})

    if !${IRCUser[${gMyNick}].IsConnected}
    {
        echo "Connection failed"
        UnregisterEvents
        return
    }

    echo "Connected! Joining channel..."
    IRCUser[${gMyNick}]:Join[#general]
    wait 25

    while ${gRunning}
    {
        wait 10
    }

    UnregisterEvents
    IRCUser[${gMyNick}]:Disconnect[Bot shutting down]
}

function RegisterEvents()
{
    Event[IRC_ReceivedChannelMsg]:AttachAtom[OnChannelMsg]
    Event[IRC_ReceivedPrivateMsg]:AttachAtom[OnPrivateMsg]
    Event[IRC_NickJoinedChannel]:AttachAtom[OnUserJoined]
    Event[IRC_NickLeftChannel]:AttachAtom[OnUserLeft]
    Event[IRC_UserDisconnected]:AttachAtom[OnDisconnected]
}

function UnregisterEvents()
{
    Event[IRC_ReceivedChannelMsg]:DetachAtom[OnChannelMsg]
    Event[IRC_ReceivedPrivateMsg]:DetachAtom[OnPrivateMsg]
    Event[IRC_NickJoinedChannel]:DetachAtom[OnUserJoined]
    Event[IRC_NickLeftChannel]:DetachAtom[OnUserLeft]
    Event[IRC_UserDisconnected]:DetachAtom[OnDisconnected]
}

atom OnChannelMsg(string User, string Channel, string From, string Message)
{
    echo "[${Channel}] <${From}> ${Message}"

    if ${Message.Equal[!quit]}
    {
        IRCUser[${User}]:PM[${Channel},Shutting down as requested by ${From}]
        gRunning:Set[FALSE]
    }
}

atom OnPrivateMsg(string User, string From, string To, string Message)
{
    echo "[PM] <${From}> ${Message}"
    IRCUser[${User}]:PM[${From},Thanks for your message!]
}

atom OnUserJoined(string User, string Channel, string WhoJoined)
{
    echo "*** ${WhoJoined} joined ${Channel}"
    IRCUser[${User}]:PM[${Channel},Welcome ${WhoJoined}!]
}

atom OnUserLeft(string User, string Channel, string WhoLeft)
{
    echo "*** ${WhoLeft} left ${Channel}"
}

atom OnDisconnected(string User, string Reason, string Response)
{
    echo "*** Disconnected: ${Reason}"
    gRunning:Set[FALSE]
}
```

---

## Notes

### Case Sensitivity

- **LavishScript datatypes and members** are case-insensitive.
- **Event names are case-sensitive** on registration/attachment. Use the exact names shown in [IRC Events](#irc-events) — in particular, [IRC_ReceivedNOTICE](#irc_receivednotice) uses an uppercase `NOTICE` suffix.
- **IRC nicknames and channel names** are protocol-level strings and should be matched exactly as the server presents them; however, ISXIM's internal lookups use case-insensitive comparison (via `stricmp`) when looking up a channel or nick by name.

### NULL Checks

Always check existence before accessing members:

```lavishscript
if ${IRCUser[MyNick](exists)}
{
    echo ${IRCUser[MyNick].Server}
}
```

### Parameter Notation

In this documentation:
- `[parameter]` — parameter list passed to a method/member
- `[param1,param2]` — multiple parameters separated by commas
- Optional parameters are shown as separate method overloads on their own lines

### IRC Connection Sequence

Since ISXIM-20120413.0024, `IRC:Connect[...]` behaves asynchronously:

**Behavior:**
- `IRC:Connect[]` initiates a connection and adds the `ircuser` to the user list immediately.
- While connecting: `IsConnecting = TRUE`, `IsConnected = FALSE`.
- Once established: `IsConnecting = FALSE`, `IsConnected = TRUE`.
- The `ircuser` only goes invalid if the connection fails.

**Method Guard:**
All `ircuser` methods silently no-op while the user is still connecting or not yet connected. Your script must wait for connection to complete before issuing commands.

**Recommended Pattern:**
```lavishscript
IRC:Connect[irc.example.com,MyNick]
do
{
    wait 3
}
while (${IRCUser[MyNick](exists)} && ${IRCUser[MyNick].IsConnecting})

if !${IRCUser[MyNick](exists)} || !${IRCUser[MyNick].IsConnected}
{
    echo "Connection failed"
    return
}
```

### Known Issues and Gotchas

These items are documented in `ISXIMChanges.txt` as available features, but inspection of the current source in `ISXIM\src` shows they are not fully wired up in the build. Scripts should work around them as noted.

1. **`ircuser:Notice[to,message]` is non-functional.**
   The `Notice` method is declared in the type's method table, but the current `IRCUserType::GetMethod` switch in `IRCDataTypes.cpp` does not include a `case Notice:` handler, so invoking it has no effect. Use `SendRaw` instead:
   ```lavishscript
   IRCUser[MyNick]:SendRaw[NOTICE SomeUser :Your notice text]
   ```

2. **`channel.IsSet[mode]`, `channel.Limit`, `channel.Password` are not accessible.**
   These members are defined in the enum and implemented in `ChannelType::GetMember`, but the `ChannelType` constructor does not register them via `TypeMember(...)`, so LavishScript does not expose them. To track channel modes, subscribe to [IRC_ChannelModeChange](#irc_channelmodechange) and maintain state in your script.

3. **`IRC:QuietMode[...]` is not functional.**
   The global `IRC` datatype declares a `QuietMode` method in its enum but does not register it via `TypeMethod` and has no case in `GetMethod`. Use [`IM:QuietMode[on|off]`](#im) instead, which controls the same global `gQuietMode` flag.

4. **`ircuser:Join` channel-name prefix.**
   ISXIM enforces that channel names begin with `#` or `&` (standard IRC prefixes). Attempting to join a name without one of these prefixes is rejected with an error message.

### CTCP Auto-Responses

ISXIM automatically responds to these incoming CTCP requests without any script intervention: `VERSION`, `FINGER`, `PING`, `TIME`, `USERINFO`, `CLIENTINFO`. The [IRC_ReceivedCTCP](#irc_receivedctcp) event still fires, so your script can observe them.

### Message Sanitization

ISXIM strips mIRC-style formatting from incoming messages: color codes, bold, and underline are removed before event dispatch and display. Outgoing messages are not modified.

---

## Deprecated Features

This section documents features that have been removed from ISXIM. Scripts using any of these must be updated.

### Removed TLOs and DataTypes

#### Yahoo/Y TLO and yahoo/y datatype — March 21, 2022

**Removed in:** ISXIM-20220321.0001

The Yahoo Messenger functionality was removed in its entirety, including:
- The `Yahoo` TLO (previously `Y`)
- The `yahoo` datatype (previously `y`)
- The `buddy` datatype
- All Yahoo-related events (see [Removed Events](#removed-events))

**Reason:** Yahoo Messenger was discontinued by Yahoo.

### Removed Members

#### irc.IsConnecting — April 13, 2012

**Removed in:** ISXIM-20120413.0024

**Removed:** `IsConnecting` member on the `irc` datatype.

**Replacement:** Use per-session state on the user itself: `${IRCUser[nickname].IsConnecting}` and `${IRCUser[nickname].IsConnected}`. See [IRC Connection Sequence](#irc-connection-sequence).

### Removed Events

#### Yahoo Events — March 21, 2022

**Removed in:** ISXIM-20220321.0001

All Yahoo-related events were removed:
- `Yahoo_onSystemMessage`
- `Yahoo_onLoginResponse`
- `Yahoo_onLogout`
- `Yahoo_onIMReceived`
- `Yahoo_onOfflineIMReceived`
- `Yahoo_onTypingNotice`
- `Yahoo_onPing`
- `Yahoo_onStatusChanged`
- `Yahoo_onErrorMessage`
- `Yahoo_onBuzz`

**Reason:** Yahoo Messenger was discontinued.
