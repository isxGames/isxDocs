# ISXIM Reference

## Table of Contents

1. [Introduction](#introduction)
2. [Top-Level Objects](#top-level-objects)
   - [Extension TLOs](#extension-tlos)
   - [IRC TLOs](#irc-tlos)
3. [DataTypes](#datatypes)
   - [Extension DataTypes](#extension-datatypes)
   - [IRC DataTypes](#irc-datatypes)
4. [Events](#events)
   - [Event Registration](#event-registration)
   - [IRC Events](#irc-events)
5. [Usage Examples](#usage-examples)
   - [IRC Connection and Authentication](#irc-connection-and-authentication)
   - [Joining and Leaving Channels](#joining-and-leaving-channels)
   - [Sending Messages](#sending-messages)
   - [Working with Channel Members](#working-with-channel-members)
   - [Event Handling](#event-handling)
6. [Notes](#notes)
   - [Case Sensitivity](#case-sensitivity)
   - [NULL Checks](#null-checks)
   - [Parameter Notation](#parameter-notation)
   - [IRC Connection Sequence](#irc-connection-sequence)
   - [Deprecated Features](#deprecated-features)

---

## Introduction

This document provides comprehensive reference documentation for all datatypes, top-level objects, and events available in the ISXIM extension. ISXIM extends LavishScript to provide IRC (Internet Relay Chat) communication functionality through a structured type system.

### DataType Inheritance

ISXIM uses a simple type hierarchy where all IRC datatypes are standalone with no inheritance relationships.

### Accessing DataTypes

DataTypes are accessed through Top-Level Objects (TLOs) or through members of other datatypes. For example:

```lavishscript
${IM}                           // TLO returning 'im' datatype
${IRC}                          // TLO returning 'irc' datatype
${IRCUser[MyNick]}              // TLO with parameter returning 'ircuser' datatype
${IRCUser[MyNick].Channel[#general]}  // Member returning 'channel' datatype
${IRCUser[MyNick].Channel[#general].Nick[SomeUser]}  // Member returning 'nick' datatype
```

---

## Top-Level Objects

Top-Level Objects (TLOs) are the entry points for accessing extension functionality. They can be accessed directly in LavishScript.

### Extension TLOs

| TLO | DataType | Description |
|-----|----------|-------------|
| **IM** | [im](#im) | ISXIM extension information and utilities |

### IRC TLOs

| TLO | DataType | Description |
|-----|----------|-------------|
| **IRC** | [irc](#irc) | Main IRC functionality and utilities |
| **IRCUser[#]** | [ircuser](#ircuser) | Access IRC user by index (1 to IRC.NumUsers) |
| **IRCUser[nickname]** | [ircuser](#ircuser) | Access IRC user by exact nickname match |

---

## DataTypes

### Extension DataTypes

#### **im**

Extension information and utilities. Accessed via IM TLO.

**Members:**
- `InQuietMode` - bool: Whether extension is in quiet mode (suppresses output)
- `Version` - string: ISXIM version number
- `IsReady` - bool: Whether extension is ready for use

**Methods:**
- `QuietMode[on]` - Enable quiet mode (suppresses extension output)
- `QuietMode[off]` - Disable quiet mode

**Example:**
```lavishscript
echo "ISXIM Version: ${IM.Version}"
echo "Is Ready: ${IM.IsReady}"

; Enable quiet mode to suppress output
IM:QuietMode[on]

; Disable quiet mode
IM:QuietMode[off]
```

---

### IRC DataTypes

#### **irc**

Main IRC datatype providing core IRC functionality. Accessed via IRC TLO.

**Members:**
- `NumUsers` - int: Number of currently connected IRC users
- `CWD` - string: Current Working Directory

**Methods:**
- `Connect[server,nickname]` - Connect to IRC server with specified nickname
- `Connect[server,nickname,port]` - Connect to IRC server on specific port
- `Connect[server,nickname,port,password]` - Connect to IRC server with server password

**Example:**
```lavishscript
; Connect to IRC server
IRC:Connect[irc.lavishsoft.com,MyNickname]

; Connect with custom port
IRC:Connect[irc.example.com,MyNickname,6667]

; Connect with server password
IRC:Connect[irc.example.com,MyNickname,6667,serverpass]

echo "Connected users: ${IRC.NumUsers}"
```

**See Also:**
- [ircuser](#ircuser) - Connected IRC user instances

---

#### **ircuser**

Represents a connected IRC user session. Accessed via IRCUser TLO.

**Members:**

*Identity:*
- `MyNick` - string: Your nickname for this connection
- `ID` - int: Unique identifier for this IRC user session
- `Server` - string: IRC server address
- `Port` - int: IRC server port

*Status:*
- `IsConnecting` - bool: Whether connection is in progress (TRUE while connecting, FALSE when complete)
- `IsConnected` - bool: Whether connection is established (FALSE while connecting, TRUE when complete)

*Channels:*
- `NumChannelsIn` - int: Number of channels you're currently in
- `Channel[#]` - [channel](#channel): Access channel by index (1 to NumChannelsIn)
- `Channel[name]` - [channel](#channel): Access channel by exact name match

*Users:*
- `Nick[name]` - [nick](#nick): Access nick by exact name match

**Methods:**
- `Disconnect` - Disconnect with default quit reason
- `Disconnect[reason]` - Disconnect with custom quit reason
- `Join[channelname]` - Join IRC channel (requires wait after call - see notes)
- `Join[channelname,key]` - Join password-protected channel
- `PM[to,message]` - Send private message to user or channel
- `ChangeNickTo[newnick]` - Change your nickname
- `Notice[to,message]` - Send notice to user or channel (no auto-response)
- `Emote[to,message]` - Send emote/action message
- `SendRaw[string]` - Send raw IRC command to server (no alterations)

**Example:**
```lavishscript
; Wait for connection to complete
do
{
    wait 3
}
while (${IRCUser[MyNick](exists)} && ${IRCUser[MyNick].IsConnecting})

; Join a channel (requires wait time)
IRCUser[MyNick]:Join[#general]
wait 25

; Send messages
IRCUser[MyNick]:PM[#general,Hello everyone!]
IRCUser[MyNick]:PM[SomeUser,Private message]

; Send notice (no auto-response capability)
IRCUser[MyNick]:Notice[SomeUser,This is a notice]

; Send emote
IRCUser[MyNick]:Emote[#general,waves hello]

; Change nickname
IRCUser[MyNick]:ChangeNickTo[NewNickname]

; Disconnect
IRCUser[MyNick]:Disconnect[Goodbye!]
```

**Important Notes:**
- The Join method requires communication between client and server. Use `wait 25` (or higher for high-latency connections) after each Join call
- When IsConnecting is TRUE and IsConnected is FALSE, the user is still connecting - do not issue commands yet
- When IsConnecting is FALSE and IsConnected is TRUE, connection is complete and commands can be issued
- Private messages (PM) can trigger auto-responses; Notices cannot (per IRC protocol)

**See Also:**
- [channel](#channel) - IRC channel information
- [nick](#nick) - IRC nickname/user information

---

#### **channel**

Represents an IRC channel. Accessed via IRCUser.Channel member.

**Members:**

*Identity:*
- `Name` - string: Channel name
- `Topic` - string: Current channel topic
- `TopicBy` - string: Who set the current topic

*Members:*
- `NumNicks` - int: Number of users in channel
- `Nick[#]` - [nick](#nick): Access nick by index (1 to NumNicks)
- `Nick[name]` - [nick](#nick): Access nick by exact name match

*Modes:*
- `IsSet[mode]` - bool: Check if channel mode is set
- `Limit` - int: User limit for channel (-1 if no limit set)
- `Password` - string: Channel password (if set)

**Valid Mode Values for IsSet:**
- `PASSWORD` - Channel requires password
- `LIMIT` - Channel has user limit
- `SECRET` - Channel is secret
- `PRIVATE` - Channel is private
- `INVITEONLY` - Invite-only channel
- `MODERATED` - Channel is moderated
- `NOEXTERNALMSGS` - No external messages allowed
- `ONLYOPSCHANGETOPIC` - Only operators can change topic
- `REGISTERED` - Channel is registered
- `REGISTRATIONREQ` - Registration required
- `NOCOLORSALLOWED` - No color codes allowed

**Methods:**
- `Leave` - Leave the channel
- `Say[message]` - Send message to channel
- `SetMode[arguments]` - Set channel mode (see IRC protocol documentation)
- `GetBans[variable]` - Populate index:string variable with channel ban list

**Example:**
```lavishscript
; Get channel information
variable channel MyChannel
MyChannel:Set[${IRCUser[MyNick].Channel[#general]}]

echo "Channel: ${MyChannel.Name}"
echo "Topic: ${MyChannel.Topic}"
echo "Set by: ${MyChannel.TopicBy}"
echo "Users: ${MyChannel.NumNicks}"

; Check channel modes
if ${MyChannel.IsSet[INVITEONLY]}
{
    echo "This is an invite-only channel"
}

; Send message to channel
MyChannel:Say[Hello everyone!]

; Get ban list
variable index:string BanList
MyChannel:GetBans[BanList]

; Leave channel
MyChannel:Leave
```

**See Also:**
- [ircuser](#ircuser) - IRC user connection
- [nick](#nick) - Channel member information

---

#### **nick**

Represents a nickname/user in an IRC channel. Accessed via Channel.Nick or IRCUser.Nick members.

**Members:**
- `Name` - string: Nickname
- `Type` - string: User type/privilege level in channel

**Valid Type Values:**
- `Owner` - Channel owner
- `SOP` - Super operator
- `OP` - Operator
- `HOP` - Half operator
- `Voice` - Voiced user
- `Normal` - Normal user (no special privileges)

**Methods:**
- `PM[message]` - Send private message to this user
- `SetMode[arguments]` - Set mode for this user (see IRC protocol documentation)

**Example:**
```lavishscript
; Get user information
variable nick ChannelUser
ChannelUser:Set[${IRCUser[MyNick].Channel[#general].Nick[SomeUser]}]

echo "Nickname: ${ChannelUser.Name}"
echo "Type: ${ChannelUser.Type}"

; Send private message
ChannelUser:PM[Hello there!]

; Check privilege level
if ${ChannelUser.Type.Equal[OP]} || ${ChannelUser.Type.Equal[SOP]} || ${ChannelUser.Type.Equal[Owner]}
{
    echo "This user is an operator or higher"
}
```

**See Also:**
- [channel](#channel) - IRC channel containing this nick
- [ircuser](#ircuser) - IRC user connection

---

## Events

### Event Registration

ISXIM events follow standard LavishScript event registration patterns. Events are registered using the Event system and attached with atoms.

**Example Event Registration:**
```lavishscript
function OnIRCMessage(string User, string Channel, string From, string Message)
{
    echo "Message in ${Channel} from ${From}: ${Message}"
}

; Register the event handler
Event[IRC_ReceivedChannelMsg]:AttachAtom[OnIRCMessage]

; Later: Detach when no longer needed
Event[IRC_ReceivedChannelMsg]:DetachAtom[OnIRCMessage]
```

### IRC Events

#### **IRC_ReceivedNotice**

Fires when a notice is received.

**Parameters:**
- `User` - string: Your IRC user nickname
- `From` - string: Who sent the notice
- `To` - string: Notice recipient (you or channel)
- `Message` - string: Notice text

**Example:**
```lavishscript
function OnIRCNotice(string User, string From, string To, string Message)
{
    echo "Notice from ${From} to ${To}: ${Message}"
}

Event[IRC_ReceivedNotice]:AttachAtom[OnIRCNotice]
```

---

#### **IRC_ReceivedChannelMsg**

Fires when a channel message is received.

**Parameters:**
- `User` - string: Your IRC user nickname
- `Channel` - string: Channel where message was sent
- `From` - string: Who sent the message
- `Message` - string: Message text

**Example:**
```lavishscript
function OnChannelMessage(string User, string Channel, string From, string Message)
{
    echo "[${Channel}] ${From}: ${Message}"
}

Event[IRC_ReceivedChannelMsg]:AttachAtom[OnChannelMessage]
```

---

#### **IRC_ReceivedPrivateMsg**

Fires when a private message is received.

**Parameters:**
- `User` - string: Your IRC user nickname
- `From` - string: Who sent the message
- `To` - string: Message recipient (your nickname)
- `Message` - string: Message text

**Example:**
```lavishscript
function OnPrivateMessage(string User, string From, string To, string Message)
{
    echo "PM from ${From}: ${Message}"
}

Event[IRC_ReceivedPrivateMsg]:AttachAtom[OnPrivateMessage]
```

---

#### **IRC_ReceivedCTCP**

Fires when a CTCP (Client-To-Client Protocol) request is received. ISXIM automatically responds to: Version, Finger, Ping, Time, Userinfo, and Clientinfo.

**Parameters:**
- `User` - string: Your IRC user nickname
- `From` - string: Who sent the CTCP
- `To` - string: CTCP recipient
- `Message` - string: CTCP command/message

**Example:**
```lavishscript
function OnCTCP(string User, string From, string To, string Message)
{
    echo "CTCP from ${From}: ${Message}"
}

Event[IRC_ReceivedCTCP]:AttachAtom[OnCTCP]
```

---

#### **IRC_ReceivedEmote**

Fires when an emote/action message is received.

**Parameters:**
- `User` - string: Your IRC user nickname
- `From` - string: Who sent the emote
- `To` - string: Emote recipient (you or channel)
- `Message` - string: Emote text

**Example:**
```lavishscript
function OnEmote(string User, string From, string To, string Message)
{
    echo "* ${From} ${Message}"
}

Event[IRC_ReceivedEmote]:AttachAtom[OnEmote]
```

---

#### **IRC_NickJoinedChannel**

Fires when a user joins a channel you're in.

**Parameters:**
- `User` - string: Your IRC user nickname
- `Channel` - string: Channel that was joined
- `WhoJoined` - string: Nickname of user who joined

**Example:**
```lavishscript
function OnUserJoined(string User, string Channel, string WhoJoined)
{
    echo "${WhoJoined} joined ${Channel}"
}

Event[IRC_NickJoinedChannel]:AttachAtom[OnUserJoined]
```

---

#### **IRC_NickLeftChannel**

Fires when a user leaves a channel you're in (via PART command).

**Parameters:**
- `User` - string: Your IRC user nickname
- `Channel` - string: Channel that was left
- `WhoLeft` - string: Nickname of user who left

**Example:**
```lavishscript
function OnUserLeft(string User, string Channel, string WhoLeft)
{
    echo "${WhoLeft} left ${Channel}"
}

Event[IRC_NickLeftChannel]:AttachAtom[OnUserLeft]
```

---

#### **IRC_NickQuit**

Fires when a user quits IRC entirely.

**Parameters:**
- `User` - string: Your IRC user nickname
- `Channel` - string: Channel where you saw the quit
- `Nick` - string: Nickname of user who quit
- `Reason` - string: Quit message

**Example:**
```lavishscript
function OnUserQuit(string User, string Channel, string Nick, string Reason)
{
    echo "${Nick} quit (${Reason})"
}

Event[IRC_NickQuit]:AttachAtom[OnUserQuit]
```

---

#### **IRC_TopicSet**

Fires when a channel topic is set or changed.

**Parameters:**
- `User` - string: Your IRC user nickname
- `Channel` - string: Channel whose topic changed
- `NewTopic` - string: New topic text
- `TopicSetBy` - string: Who set the topic

**Example:**
```lavishscript
function OnTopicChanged(string User, string Channel, string NewTopic, string TopicSetBy)
{
    echo "${Channel} topic set by ${TopicSetBy}: ${NewTopic}"
}

Event[IRC_TopicSet]:AttachAtom[OnTopicChanged]
```

---

#### **IRC_NickChanged**

Fires when a user (including yourself) changes nickname.

**Parameters:**
- `User` - string: Your IRC user nickname (may be old or new depending on who changed)
- `OldNick` - string: Previous nickname
- `NewNick` - string: New nickname

**Example:**
```lavishscript
function OnNickChange(string User, string OldNick, string NewNick)
{
    echo "${OldNick} is now known as ${NewNick}"
}

Event[IRC_NickChanged]:AttachAtom[OnNickChange]
```

---

#### **IRC_KickedFromChannel**

Fires when someone is kicked from a channel.

**Parameters:**
- `User` - string: Your IRC user nickname
- `Channel` - string: Channel where kick occurred
- `WhoKicked` - string: Nickname of user who was kicked
- `KickedBy` - string: Who performed the kick
- `Reason` - string: Kick reason/message

**Example:**
```lavishscript
function OnUserKicked(string User, string Channel, string WhoKicked, string KickedBy, string Reason)
{
    echo "${WhoKicked} was kicked from ${Channel} by ${KickedBy} (${Reason})"
}

Event[IRC_KickedFromChannel]:AttachAtom[OnUserKicked]
```

---

#### **IRC_PRIVMSGErrorResponse**

Fires when a PRIVMSG command fails (e.g., user doesn't exist, external messages not allowed).

**Parameters:**
- `User` - string: Your IRC user nickname
- `ErrorType` - string: Type of error (see below)
- `To` - string: Who/where you tried to message
- `Response` - string: Server error response text

**Valid ErrorType Values:**
- `NO_SUCH_NICKORCHANNEL` - Recipient doesn't exist
- `NO_EXTERNAL_MSGS_ALLOWED` - Channel doesn't allow external messages

**Example:**
```lavishscript
function OnPMError(string User, string ErrorType, string To, string Response)
{
    echo "Failed to message ${To}: ${ErrorType}"
}

Event[IRC_PRIVMSGErrorResponse]:AttachAtom[OnPMError]
```

---

#### **IRC_JOINErrorResponse**

Fires when a JOIN command fails.

**Parameters:**
- `User` - string: Your IRC user nickname
- `ErrorType` - string: Type of error (see below)
- `Channel` - string: Channel you tried to join
- `Response` - string: Server error response text

**Valid ErrorType Values:**
- `BANNED` - You are banned from the channel
- `MUST_BE_REGISTERED` - Channel requires registration
- `REQUIRES_KEY` - Channel requires password

**Example:**
```lavishscript
function OnJoinError(string User, string ErrorType, string Channel, string Response)
{
    echo "Failed to join ${Channel}: ${ErrorType}"
}

Event[IRC_JOINErrorResponse]:AttachAtom[OnJoinError]
```

---

#### **IRC_NickTypeChange**

Fires when a user's privilege level changes in a channel (e.g., opped, deopped, voiced).

**Parameters:**
- `User` - string: Your IRC user nickname
- `Channel` - string: Channel where change occurred
- `NickName` - string: User whose type changed
- `NickType` - string: Type of privilege (see below)
- `Toggle` - string: "TRUE" if granted, "FALSE" if removed
- `WhoSet` - string: Who made the change

**Valid NickType Values:**
- `OWNER` - Channel owner
- `SOP` - Super operator
- `OP` - Operator
- `HOP` - Half operator
- `Voice` - Voice privilege
- `Normal` - Normal user

**Example:**
```lavishscript
function OnNickTypeChange(string User, string Channel, string NickName, string NickType, string Toggle, string WhoSet)
{
    if ${Toggle.Equal[TRUE]}
    {
        echo "${NickName} was granted ${NickType} by ${WhoSet} in ${Channel}"
    }
    else
    {
        echo "${NickName} had ${NickType} removed by ${WhoSet} in ${Channel}"
    }
}

Event[IRC_NickTypeChange]:AttachAtom[OnNickTypeChange]
```

---

#### **IRC_ChannelModeChange**

Fires when a channel mode is changed.

**Parameters:**
- `User` - string: Your IRC user nickname
- `Channel` - string: Channel whose mode changed
- `ModeType` - string: Type of mode (see below)
- `Toggle` - string: "TRUE" if enabled, "FALSE" if disabled
- `WhoSet` - string: Who made the change
- `Extra` - string: Additional info (password for PASSWORD mode, limit for LIMIT mode)

**Valid ModeType Values:**
- `PASSWORD` - Channel password
- `LIMIT` - User limit
- `SECRET` - Secret channel
- `PRIVATE` - Private channel
- `INVITEONLY` - Invite-only
- `MODERATED` - Moderated channel
- `NOEXTERNALMSGS` - No external messages
- `ONLYOPSCHANGETOPIC` - Only ops change topic
- `REGISTERED` - Registered channel
- `REGISTRATIONREQ` - Registration required
- `NOCOLORSALLOWED` - No color codes

**Example:**
```lavishscript
function OnChannelModeChange(string User, string Channel, string ModeType, string Toggle, string WhoSet, string Extra)
{
    if ${Toggle.Equal[TRUE]}
    {
        echo "${Channel} mode ${ModeType} enabled by ${WhoSet}"
        if ${ModeType.Equal[LIMIT]}
        {
            echo "Limit set to: ${Extra}"
        }
    }
    else
    {
        echo "${Channel} mode ${ModeType} disabled by ${WhoSet}"
    }
}

Event[IRC_ChannelModeChange]:AttachAtom[OnChannelModeChange]
```

---

#### **IRC_AddChannelBan**

Fires when a ban is added to a channel.

**Parameters:**
- `User` - string: Your IRC user nickname
- `Channel` - string: Channel where ban was added
- `WhoSet` - string: Who set the ban
- `Ban` - string: Ban mask

**Example:**
```lavishscript
function OnBanAdded(string User, string Channel, string WhoSet, string Ban)
{
    echo "Ban added to ${Channel} by ${WhoSet}: ${Ban}"
}

Event[IRC_AddChannelBan]:AttachAtom[OnBanAdded]
```

---

#### **IRC_RemoveChannelBan**

Fires when a ban is removed from a channel.

**Parameters:**
- `User` - string: Your IRC user nickname
- `Channel` - string: Channel where ban was removed
- `WhoSet` - string: Who removed the ban
- `Ban` - string: Ban mask that was removed

**Example:**
```lavishscript
function OnBanRemoved(string User, string Channel, string WhoSet, string Ban)
{
    echo "Ban removed from ${Channel} by ${WhoSet}: ${Ban}"
}

Event[IRC_RemoveChannelBan]:AttachAtom[OnBanRemoved]
```

---

#### **IRC_UnhandledEvent**

Fires for IRC protocol events not specifically handled by other ISXIM events.

**Parameters:**
- `User` - string: Your IRC user nickname
- `Command` - string: IRC command received
- `Param` - string: Command parameters
- `Rest` - string: Remaining data

**Example:**
```lavishscript
function OnUnhandledEvent(string User, string Command, string Param, string Rest)
{
    echo "Unhandled IRC event - Command: ${Command}, Param: ${Param}"
}

Event[IRC_UnhandledEvent]:AttachAtom[OnUnhandledEvent]
```

---

#### **IRC_UserDisconnected**

Fires when you are disconnected from IRC server.

**Parameters:**
- `User` - string: Your IRC user nickname
- `Reason` - string: Disconnect reason
- `OptionalServerResponse` - string: Optional server response message

**Example:**
```lavishscript
function OnDisconnected(string User, string Reason, string OptionalServerResponse)
{
    echo "Disconnected: ${Reason}"
    if ${OptionalServerResponse.Length}
    {
        echo "Server said: ${OptionalServerResponse}"
    }
}

Event[IRC_UserDisconnected]:AttachAtom[OnDisconnected]
```

---

## Usage Examples

### IRC Connection and Authentication

```lavishscript
; Basic IRC connection
function main()
{
    ; Check if extension is ready
    if !${IM.IsReady}
    {
        echo "ISXIM is not ready yet"
        return
    }

    echo "ISXIM Version: ${IM.Version}"

    ; Connect to IRC server
    echo "Connecting to IRC..."
    IRC:Connect[irc.lavishsoft.com,MyNickname]

    ; Wait for connection to complete
    do
    {
        wait 3
    }
    while (${IRCUser[MyNickname](exists)} && ${IRCUser[MyNickname].IsConnecting})

    ; Check if connected
    if !${IRCUser[MyNickname](exists)}
    {
        echo "Failed to connect"
        return
    }

    if ${IRCUser[MyNickname].IsConnected}
    {
        echo "Successfully connected to ${IRCUser[MyNickname].Server}:${IRCUser[MyNickname].Port}"
        echo "Your nickname: ${IRCUser[MyNickname].MyNick}"
    }
}
```

### Joining and Leaving Channels

```lavishscript
; Join multiple channels
function JoinChannels()
{
    variable string MyNick = "MyNickname"

    ; Join first channel
    echo "Joining #general..."
    IRCUser[${MyNick}]:Join[#general]
    wait 25

    ; Join password-protected channel
    echo "Joining #private..."
    IRCUser[${MyNick}]:Join[#private,secretkey]
    wait 25

    ; Join another channel
    echo "Joining #bots..."
    IRCUser[${MyNick}]:Join[#bots]
    wait 25

    echo "Joined ${IRCUser[${MyNick}].NumChannelsIn} channels"

    ; List all channels
    variable int i
    for (i:Set[1]; ${i} <= ${IRCUser[${MyNick}].NumChannelsIn}; i:Inc)
    {
        echo "Channel ${i}: ${IRCUser[${MyNick}].Channel[${i}].Name}"
    }
}

; Leave a specific channel
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
; Send various types of messages
function SendMessages()
{
    variable string MyNick = "MyNickname"

    ; Send channel message
    IRCUser[${MyNick}]:PM[#general,Hello everyone!]

    ; Send private message to user
    IRCUser[${MyNick}]:PM[SomeUser,Hi there!]

    ; Send notice (no auto-response capability)
    IRCUser[${MyNick}]:Notice[#general,This is a notice]

    ; Send emote/action
    IRCUser[${MyNick}]:Emote[#general,waves hello]

    ; Send message using channel object
    variable channel MyChannel
    MyChannel:Set[${IRCUser[${MyNick}].Channel[#general]}]
    MyChannel:Say[Hello via channel object]
}

; Respond to commands in channel
function OnChannelMsg(string User, string Channel, string From, string Message)
{
    ; Check for !hello command
    if ${Message.Find[!hello]}
    {
        IRCUser[${User}]:PM[${Channel},Hello ${From}!]
    }

    ; Check for !time command
    if ${Message.Find[!time]}
    {
        IRCUser[${User}]:PM[${Channel},The time is ${Time}]
    }
}

Event[IRC_ReceivedChannelMsg]:AttachAtom[OnChannelMsg]
```

### Working with Channel Members

```lavishscript
; List all members of a channel
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

    for (i:Set[1]; ${i} <= ${MyChannel.NumNicks}; i:Inc)
    {
        CurrentNick:Set[${MyChannel.Nick[${i}]}]
        echo "[${CurrentNick.Type}] ${CurrentNick.Name}"
    }
}

; Check if user has operator privileges
function IsUserOp(string ChannelName, string UserName)
{
    variable string MyNick = "MyNickname"
    variable nick User

    User:Set[${IRCUser[${MyNick}].Channel[${ChannelName}].Nick[${UserName}]}]

    if !${User(exists)}
    {
        return FALSE
    }

    if ${User.Type.Equal[OP]} || ${User.Type.Equal[SOP]} || ${User.Type.Equal[Owner]}
    {
        return TRUE
    }

    return FALSE
}

; Get channel information
function GetChannelInfo(string ChannelName)
{
    variable string MyNick = "MyNickname"
    variable channel Chan

    Chan:Set[${IRCUser[${MyNick}].Channel[${ChannelName}]}]

    if !${Chan(exists)}
    {
        echo "Not in channel ${ChannelName}"
        return
    }

    echo "Channel: ${Chan.Name}"
    echo "Topic: ${Chan.Topic} (set by ${Chan.TopicBy})"
    echo "Members: ${Chan.NumNicks}"

    ; Check modes
    if ${Chan.IsSet[INVITEONLY]}
        echo "Mode: Invite Only"
    if ${Chan.IsSet[MODERATED]}
        echo "Mode: Moderated"
    if ${Chan.IsSet[SECRET]}
        echo "Mode: Secret"
    if ${Chan.IsSet[PRIVATE]}
        echo "Mode: Private"

    ; Check limit
    if ${Chan.Limit} > 0
        echo "User Limit: ${Chan.Limit}"

    ; Check password
    if ${Chan.IsSet[PASSWORD]}
        echo "Password Protected: Yes"
}
```

### Event Handling

```lavishscript
; Complete IRC bot example with event handling
variable string gMyNick = "MyBot"
variable bool gRunning = TRUE

function main()
{
    ; Register all event handlers
    RegisterEvents

    ; Connect to IRC
    echo "Connecting..."
    IRC:Connect[irc.lavishsoft.com,${gMyNick}]

    ; Wait for connection
    do
    {
        wait 3
    }
    while (${IRCUser[${gMyNick}](exists)} && ${IRCUser[${gMyNick}].IsConnecting})

    if !${IRCUser[${gMyNick}].IsConnected}
    {
        echo "Connection failed"
        return
    }

    echo "Connected! Joining channels..."

    ; Join channels
    IRCUser[${gMyNick}]:Join[#general]
    wait 25

    ; Main loop
    while ${gRunning}
    {
        wait 10
    }

    ; Cleanup
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

function OnChannelMsg(string User, string Channel, string From, string Message)
{
    echo "[${Channel}] <${From}> ${Message}"

    ; Respond to !quit command from ops only
    if ${Message.Equal[!quit]}
    {
        if ${IsUserOp[${Channel},${From}]}
        {
            IRCUser[${User}]:PM[${Channel},Shutting down as requested by ${From}]
            gRunning:Set[FALSE]
        }
    }
}

function OnPrivateMsg(string User, string From, string To, string Message)
{
    echo "[PM] <${From}> ${Message}"

    ; Auto-respond to PMs
    IRCUser[${User}]:PM[${From},Thanks for your message!]
}

function OnUserJoined(string User, string Channel, string WhoJoined)
{
    echo "*** ${WhoJoined} joined ${Channel}"

    ; Greet new users
    IRCUser[${User}]:PM[${Channel},Welcome ${WhoJoined}!]
}

function OnUserLeft(string User, string Channel, string WhoLeft)
{
    echo "*** ${WhoLeft} left ${Channel}"
}

function OnDisconnected(string User, string Reason, string Response)
{
    echo "*** Disconnected: ${Reason}"
    gRunning:Set[FALSE]
}
```

---

## Notes

### Case Sensitivity

LavishScript is generally case-insensitive for datatypes and members. However, IRC nicknames and channel names are case-sensitive on the IRC protocol level and should be matched exactly.

### NULL Checks

Always check if objects exist before accessing their members:

```lavishscript
if ${IRCUser[MyNick](exists)}
{
    ; Safe to access
    echo ${IRCUser[MyNick].Server}
}
```

### Parameter Notation

In this documentation:
- `[parameter]` - Required parameter
- `[parameter,optional]` - Parameter with optional additional parameter
- Multiple syntax variations are shown as separate lines

### IRC Connection Sequence

As of April 13, 2012, the IRC connection logic works as follows:

**Previous Behavior (Before 20120413.0024):**
- `IRC:Connect[]` initiated connection and waited until fully connected
- User was only added and valid after connection completed

**Current Behavior (20120413.0024 and later):**
- `IRC:Connect[]` initiates connection and adds user to list immediately
- User has `IsConnecting = TRUE` and `IsConnected = FALSE` while connecting
- User becomes fully usable when `IsConnecting = FALSE` and `IsConnected = TRUE`
- User only becomes invalid if connection fails

**Important:** Scripts must check `IsConnecting` and wait for connection to complete before issuing commands. The extension ignores commands issued while still connecting.

**Recommended Connection Pattern:**
```lavishscript
IRC:Connect[irc.lavishsoft.com,MyNick]
do
{
    wait 3
}
while (${IRCUser[MyNick](exists)} && ${IRCUser[MyNick].IsConnecting})
```

### Deprecated Features

This section documents features that have been removed from ISXIM. These are no longer available and scripts using them must be updated.

#### Removed TLOs and DataTypes

##### Yahoo/Y TLO and yahoo/y datatype (March 21, 2022)

**Removed:** March 21, 2022 (ISXIM-20220321.0001)

The Yahoo instant messaging functionality was completely removed, including:
- **Yahoo** TLO (formerly **Y** TLO)
- **yahoo** datatype (formerly **y** datatype)
- **buddy** datatype
- All Yahoo-related events (see below)

**Reason:** Yahoo Messenger was discontinued by Yahoo.

---

#### Removed Members

##### irc.IsConnecting (April 13, 2012)

**Removed:** April 13, 2012 (ISXIM-20120413.0024)

**Removed Member:**
- `IsConnecting` - Previously on irc datatype

**Replacement:** Use `IRCUser[nickname].IsConnecting` and `IRCUser[nickname].IsConnected` instead.

The connection logic was changed so individual IRC users track their own connection state rather than the global IRC object.

---

#### Removed Events

##### Yahoo Events (March 21, 2022)

**Removed:** March 21, 2022 (ISXIM-20220321.0001)

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

---

