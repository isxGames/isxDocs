# LavishScript Fundamentals

**Purpose:** Tutorial-style introduction to LavishScript programming. Concepts, intuition, short examples.
**Audience:** Beginners learning LavishScript. Already-fluent scripters who want exhaustive command/datatype/TLO lookup tables should use [01b_LavishScript_Reference.md](01b_LavishScript_Reference.md) instead.

> **For exhaustive reference content, see [01b_LavishScript_Reference.md](01b_LavishScript_Reference.md).** This file teaches concepts; `01b` lists every command, object type, and Top-Level Object with links to canonical wiki pages.

---

## Table of Contents

1. [Introduction](#introduction)
2. [Your First Script](#your-first-script)
3. [Variables and Data Types](#variables-and-data-types)
4. [Top-Level Objects](#top-level-objects)
5. [Data Sequences](#data-sequences)
6. [Parameters](#parameters)
7. [Text Escaping](#text-escaping)
8. [Functions](#functions)
9. [Return Values](#return-values)
10. [Object-Oriented Programming](#object-oriented-programming)
11. [Members](#members)
12. [Methods](#methods)
13. [Initialize and Shutdown](#initialize-and-shutdown)
14. [Inheritance and Type Casting](#inheritance-and-type-casting)
15. [Atoms and Atomicity](#atoms-and-atomicity)
16. [Wait Commands](#wait-commands)
17. [Conditional Branching](#conditional-branching)
18. [Loops](#loops)
19. [Switch Statements](#switch-statements)
20. [Collections and Lists](#collections-and-lists)
21. [Events](#events)
22. [Triggers](#triggers)
23. [Aliases](#aliases)
24. [Modules](#modules)
25. [Input Emulation](#input-emulation)
26. [Web Requests](#web-requests)
27. [Audio System](#audio-system)
28. [Best Practices Summary](#best-practices-summary)
29. [Type Inspection and Debugging](#type-inspection-and-debugging)
30. [Additional Resources](#additional-resources)

---

## Introduction

**LavishScript** is the scripting language used by Inner Space and the extensions that load into it. Before diving into extension-specific scripting, it's essential to understand the fundamentals of LavishScript itself.

### What is LavishScript?

LavishScript is a custom scripting language designed for Inner-Space-hosted automation. It provides:

- Object-oriented programming with single inheritance
- Static type declarations with runtime coercion at data-sequence resolution
- Atomic execution for callbacks and accessors that must resolve immediately
- Direct access to Inner Space subsystems and any registered extensions
- Event-driven and trigger-driven programming for reactive scripts

LavishScript has no block-comment syntax; only line comments via `;`. There is no `/* */` form.

For the canonical language reference, see [LavishScript on the Lavish Software wiki](https://www.lavishsoft.com/wiki/index.php/LavishScript).

---

## Your First Script

### Basic Script Structure

Every LavishScript file has a `.iss` extension. The most basic script contains a `main` function:

```lavishscript
; This is a comment
function main()
{
    echo "Hello World!"
}
```

### Key Elements

**1. Comments**
- Single-line comments begin with `;` (semicolon).
- Must appear at the start of a line (after any whitespace). Trailing same-line comments are not supported.

```lavishscript
; This is a valid comment
    ; This is also a valid comment (indented)

echo "Hello"  ; NOT a comment -- this is part of the echo argument!
```

**2. The main Function**
- Entry point for script execution.
- Defined with `function main()`.
- Code block enclosed in `{` and `}`.

**3. Code Block Braces for Definitions and Control Flow**
- For `function`, `atom`, `objectdef`, `member`, `method`, `if`/`elseif`/`else`, `while`/`do-while`/`for`, and `switch`/`case`: opening `{` and closing `}` must each be on their own line.
- This rule does NOT apply to JSON object/array literals inside data sequences (e.g., `{}` and `[1,2,3]` inside `${...}` are part of JSON syntax, not LavishScript blocks).

```lavishscript
; CORRECT - control-flow braces on their own lines
function main()
{
    echo "Hello"
}

; INCORRECT - opening brace on same line as function
function main() {
    echo "Hello"
}

; FINE - JSON literal inside data sequence is exempt
variable jsonvalue jo={"key":"value"}
```

For the canonical syntax reference, see [LavishScript:Syntax](https://www.lavishsoft.com/wiki/index.php/LavishScript:Syntax).

**4. The echo Command**

`echo` writes text to the console. It is the platform's `echo` command -- documented at [ISKernel:Echo (Command)](https://www.lavishsoft.com/wiki/index.php/ISKernel:Echo_(Command)) -- and is one of the commands you will use most often in tutorial code.

<!-- CLAUDE_SKIP_START -->
### Running Your First Script

1. Save the file as `hello.iss` in your `Scripts` directory.
2. In the Inner Space console, type: `run hello`.
3. You should see: `Hello World!`.
<!-- CLAUDE_SKIP_END -->

---

## Variables and Data Types

### Declaring Variables

Variables are declared with the `variable` keyword:

```lavishscript
variable string TextToDisplay="Hello World!"
```

**Syntax:**

```
variable[(<scope>)] <type> <name>[=<initial_value>]
```

**Scope qualifier (optional):**

| Scope | Lifetime |
|---|---|
| (omitted) | Function-local; destroyed when the enclosing function returns. |
| `script` | Script-scope; lives for the duration of the script. |
| `global` | Global; lives until Inner Space exits. |
| `globalkeep` | Global plus persisted across script restarts. |

```lavishscript
variable string Name="Alice"           ; function-local
variable(script) int Counter=0         ; script-scope
variable(global) bool Initialized      ; global
```

For the canonical reference, see [LavishScript:Variables](https://www.lavishsoft.com/wiki/index.php/LavishScript:Variables). The runtime forms `DeclareVariable` and `DeleteVariable` are documented in [01b §2.1 Variable Management](01b_LavishScript_Reference.md#21-lavishscript-core-commands).

### Common Data Types

| Type | Description | Example Values |
|---|---|---|
| `string` | Text data | `"Hello"`, `"Bob"` |
| `int` | 32-bit signed integer | `-1`, `0`, `2147483647` |
| `uint` | 32-bit unsigned integer | `0`, `4294967295` |
| `int64` | 64-bit signed integer | Used for IDs/values exceeding 32-bit range |
| `byte` | 8-bit unsigned integer | `0`-`255` |
| `float` | Single-precision float | `1.5`, `-3.14`, `0.001` |
| `float64` | Double-precision float | Higher precision than `float` |
| `bool` | TRUE or FALSE | `TRUE`, `FALSE` |

When a `variable` is `string`-typed, the engine actually creates a [`mutablestring`](https://www.lavishsoft.com/wiki/index.php/ObjectType:mutablestring) (the mutable form of `string`). The two are interoperable; you generally don't need to think about the distinction.

For the complete object-types catalogue, see [01b §3](01b_LavishScript_Reference.md#3-object-types).

### Integer Range

- **Signed (`int`):** -2,147,483,648 to 2,147,483,647.
- **Unsigned (`uint`):** 0 to 4,294,967,295.
- **64-bit signed (`int64`):** -2^63 to 2^63 - 1. Use when values exceed 32-bit range.

### Type Conversion

LavishScript exposes conversion Top-Level Objects for converting between primitive types in data sequences:

| TLO | Use |
|---|---|
| [`Bool`](https://www.lavishsoft.com/wiki/index.php/TLO:Bool) | `${Bool[expr]}` evaluates `expr` as a boolean. |
| [`Float`](https://www.lavishsoft.com/wiki/index.php/TLO:Float) | `${Float[stringValue]}` parses to a float. |
| [`Int`](https://www.lavishsoft.com/wiki/index.php/TLO:Int) | `${Int[stringValue]}` parses to an int. |
| [`String`](https://www.lavishsoft.com/wiki/index.php/TLO:String) | `${String[value]}` wraps for further `.string` member access. |

```lavishscript
variable string CountText="42"
variable int Count=${Int[${CountText}]}
echo "Parsed: ${Count}"
```

### Using Variables

```lavishscript
function main()
{
    variable string Name="Alice"
    variable int Age=25

    echo "Name: ${Name}"
    echo "Age: ${Age}"
}
```

---

## Top-Level Objects

A **Top-Level Object** (TLO) is the entry point of a data sequence. When you write `${Math.Calc[1+1]}` you are reading the `Math` TLO, calling its `Calc` member, and emplacing the result. The general form is:

```
${TLO[indexargs].Member.Member.Method:colonAccessor}
```

Anything to the left of the first `.` is a TLO. Everything to the right walks members and methods.

### Why TLOs Matter

Code samples elsewhere in this file reference `${Math.Calc[...]}`, `${Script.RunningTime}`, `${Time.Timestamp}`, and similar. Each of those names is a TLO documented on the wiki. Knowing the TLO concept lets you find the canonical reference for any of them.

### LavishScript-Core TLOs

The engine ships 22 LavishScript-core TLOs (see [01b §4.1](01b_LavishScript_Reference.md#41-lavishscript-core-tlos) for the full table):

- **Type conversion:** `Bool`, `Float`, `Int`, `String`, `Enum`
- **Date/time:** `Time`
- **Events:** `Event`
- **Inline branching:** `If` (form: `${If[condition,trueValue,falseValue]}`)
- **Math:** `Math`
- **Misc:** `Arg`, `Execute`, `LavishScript`, `Script`, `Select`, `Type`
- **Operating system:** `System`
- **Scripting:** `QueuedCommands`, `Return`, `Variable`, `This`, `VariableScope`, `ForEach`, `Context`

### Inner Space TLOs

Inner Space adds further TLOs when loaded as the host. Highlights:

- **Audio / Display / Input:** `Audio`, `Display`, `Input`, `Keyboard`, `Mouse`
- **Multi-boxing:** `Session`, `Sessions`
- **Subsystems:** `InnerSpace`, `Console`, `Extension`, `Localization`, `MIDI`, `Game`, `Profile`
- **UI / Tasks:** `LGUI`, `LGUI2`, `LMAC`

The full Inner Space TLO list is in [01b §4.2 -- §4.4](01b_LavishScript_Reference.md#42-inner-space-kernel-tlos).

### Enumerating TLOs at Runtime

Run [`TopLevelObject`](https://www.lavishsoft.com/wiki/index.php/Command:TopLevelObject) at the console with no argument to list every TLO registered in the current session, including those added by extensions.

### Per-Extension TLOs

Game-specific extensions register their own TLOs (e.g., character/world/inventory entry points). Those are documented in their respective extension guide files; they are out of scope for this tutorial.

---

## Data Sequences

### What is a Data Sequence?

A **data sequence** retrieves a value and **emplaces** it in a command before execution.

**Syntax:** `${ }`

```lavishscript
variable string Name="Alice"
echo "Hello ${Name}!"
```

**Output:** `Hello Alice!`

### How Data Sequences Work

1. The parser encounters `${Name}`.
2. It looks up `Name` (which is `"Alice"`).
3. It replaces `${Name}` with `Alice` in the surrounding text.
4. The resulting `echo "Hello Alice!"` is executed.

For the canonical reference, see [LavishScript:Data Sequences](https://www.lavishsoft.com/wiki/index.php/LavishScript:Data_Sequences) and the "Data" section of [LavishScript:Syntax](https://www.lavishsoft.com/wiki/index.php/LavishScript:Syntax).

### Multiple Data Sequences

You can use multiple data sequences in a single command:

```lavishscript
variable string First="John"
variable string Last="Smith"

echo "${First} ${Last} ${First} ${Last}"
```

**Output:** `John Smith John Smith`

### Accessing Nested Data

Data sequences walk nested properties using `.` (dot) notation:

```lavishscript
variable string Name="Alice"
echo "Name length: ${Name.Length}"
```

**Output:** `Name length: 5`

Here, `Length` is a member of the `string` type. The `string` type has many more members (`Upper`, `Lower`, `Find`, `Token`, `Replace`, `Trim`, etc.) -- see [01b §3.1 Text](01b_LavishScript_Reference.md#31-text) for the full list, or the canonical [`ObjectType:string`](https://www.lavishsoft.com/wiki/index.php/ObjectType:string) wiki page.

### The Tilde Escape -- REQUIRED for Re-Parsed Output

When a data sequence's value will be **re-parsed** by another command, you MUST use the tilde (`~`) suffix to escape parser-significant characters in the value:

```lavishscript
variable string Text="He said \"Hello\""
echo "${Text~}"
```

**The rule:** use `~` whenever the data sequence output is consumed by a second command parse pass. Specifically:

1. Inside a quoted string passed to another command (e.g., `call MyFunction "${Text~}"`).
2. Inside method index brackets (e.g., `Object:Method["${Value~}"]`).
3. When constructing JSON or other quoted-text content from variables.

Without `~`, code that LOOKS correct will mis-parse the moment the variable contains a `"`, `\`, `}`, or other parser-significant character.

```lavishscript
; BAD - breaks if Text contains quotes or special characters
call SomeFunction ${Text}

; CORRECT - the ~ escapes any parser-significant characters in Text
call SomeFunction "${Text~}"
```

This is not a "best practice" -- it is the rule. Quoted-string parameters and method indices both trigger a second parse pass.

---

## Parameters

### Function Parameters

Functions can accept parameters:

```lavishscript
function main(string TextToDisplay)
{
    echo "${TextToDisplay}"
}
```

**Syntax:**

```
function name(type param1, type param2)
```

### Default Parameter Values

Parameters can have default values:

```lavishscript
function main(string Text="Hello World!", int Count=3)
{
    echo "${Text}"
    echo "Count: ${Count}"
}
```

If you run the script without parameters:

```
run myscript
```

It uses the defaults. Override them by passing values:

```
run myscript "Custom text" 5
```

### Multiple Parameters

Separate parameters with commas:

```lavishscript
function main(string First, string Last, int Age)
{
    echo "Name: ${First} ${Last}"
    echo "Age: ${Age}"
}
```

**Usage:** `run myscript "John" "Smith" 30`

### Accessing Arguments by Position

The [`Arg` TLO](https://www.lavishsoft.com/wiki/index.php/TLO:Arg) reads command-line arguments by position when you don't want to declare named parameters:

```lavishscript
echo "First arg: ${Arg[1]}"
echo "Second arg: ${Arg[2]}"
```

Useful for variadic / generic scripts.

### Command Parameter Splitting

Parameters are split by spaces:

```lavishscript
echo Multiple Parameters
; This passes TWO parameters to echo: "Multiple" and "Parameters"

echo "One Parameter"
; This passes ONE parameter to echo: "One Parameter"
```

If a parameter contains spaces, it MUST be quoted:

```lavishscript
run myscript "Hello World" 42
; Parameter 1: "Hello World"
; Parameter 2: "42"

run myscript Hello World 42
; Parameter 1: "Hello"
; Parameter 2: "World"
; Parameter 3: "42"
```

---

## Text Escaping

### Why Escape Text?

Certain characters have special meaning in LavishScript:

- `"` (quote) -- splits parameters
- `\` (backslash) -- escape character inside quoted strings
- `~` (tilde) -- data-sequence escape (see [Data Sequences](#data-sequences))

### Escaping Quotes in Quoted Strings

Use `\` (backslash) to escape quotes:

```lavishscript
echo "He said \"Hello World!\""
```

**Output:** `He said "Hello World!"`

### Escaping in Data Sequences

Use `~` (tilde) to escape data sequences -- see the [Tilde Escape rule](#the-tilde-escape----required-for-re-parsed-output) above. The short form: any time the data sequence's value will be re-parsed (inside a quoted string parameter, inside method index brackets, etc.), use `~`.

```lavishscript
; BAD - May break if Text contains quotes
call SomeFunction ${Text}

; CORRECT - Safely escapes any special characters
call SomeFunction "${Text~}"
```

### Escaping Example

```lavishscript
function main(string Phrase)
{
    variable string SafePhrase="${Phrase~}"
    echo "You said: ${SafePhrase}"
}
```

**Usage:**

```
run myscript "He said \"Hi there\""
```

**Output:** `You said: He said "Hi there"`

---

## Functions

### Defining Functions

Functions are code blocks invoked by name:

```lavishscript
function SayHello()
{
    echo "Hello!"
}

function main()
{
    call SayHello
}
```

### Calling Functions

Use the `call` command to execute a function:

```lavishscript
call FunctionName parameters
```

**Example:**

```lavishscript
function Greet(string Name)
{
    echo "Hello ${Name}!"
}

function main()
{
    call Greet "Alice"
    call Greet "Bob"
}
```

**Output:**

```
Hello Alice!
Hello Bob!
```

### Function Parameters

Functions accept parameters just like `main`:

```lavishscript
function Add(int A, int B)
{
    variable int Sum=${Math.Calc[${A}+${B}]}
    echo "Sum: ${Sum}"
}

function main()
{
    call Add 5 10
}
```

### Parameter Escaping with Functions

Just like with commands, quote and escape string parameters:

```lavishscript
function DisplayMessage(string Message)
{
    echo "Message: ${Message}"
}

function main()
{
    variable string Text="Important \"Info\""

    ; CORRECT - Quote and escape
    call DisplayMessage "${Text~}"
}
```

### Script Lifecycle Commands

The bare `run myscript` form is shorthand for [`RunScript`](https://www.lavishsoft.com/wiki/index.php/Command:RunScript). Related script-lifecycle commands -- [`EndScript`](https://www.lavishsoft.com/wiki/index.php/Command:EndScript), [`WaitScript`](https://www.lavishsoft.com/wiki/index.php/Command:WaitScript), [`Scripts`](https://www.lavishsoft.com/wiki/index.php/Command:Scripts) -- are documented in [01b §2.1 Script Lifecycle](01b_LavishScript_Reference.md#21-lavishscript-core-commands).

---

## Return Values

### The return Command

The [`return`](https://www.lavishsoft.com/wiki/index.php/Command:Return) command has two purposes:

1. Immediately exits the current function.
2. Provides a value to the caller.

```lavishscript
function GetAnswer()
{
    return 42
}
```

### Return Without a Value

Use `return` with no value to exit early:

```lavishscript
function CheckAge(int Age)
{
    if ${Age}<18
    {
        echo "Too young!"
        return
    }

    echo "Age verified"
}
```

### Accessing Return Values

After `call`-ing a function, access the return value via the [`Return` Top-Level Object](https://www.lavishsoft.com/wiki/index.php/TLO:Return) -- written as `${Return}`:

```lavishscript
function GetAnswer()
{
    return 42
}

function main()
{
    call GetAnswer
    echo "The answer is: ${Return}"
}
```

**Output:** `The answer is: 42`

### Return Value Types

Return values can be any type:

```lavishscript
function GetName()
{
    return "Alice"
}

function GetAge()
{
    return 25
}

function main()
{
    call GetName
    variable string Name="${Return}"

    call GetAge
    variable int Age=${Return}

    echo "${Name} is ${Age} years old"
}
```

### Return Value Lifespan

`${Return}` is only valid until the next `call`:

```lavishscript
function main()
{
    call GetName
    variable string Name="${Return}"  ; Save it immediately!

    call GetAge  ; This REPLACES ${Return}
    variable int Age=${Return}

    ; Now ${Return} contains the age, not the name
}
```

`Return` is one of the LavishScript-core TLOs (see [Top-Level Objects](#top-level-objects)). The "lifespan until next `call`" behavior is the TLO's defining contract -- save the value into your own variable immediately if you need it later.

---

## Object-Oriented Programming

### What is Object-Oriented Programming?

Object-Oriented Programming (OOP) ties available functionality to a type of data. In LavishScript, this is done with `objectdef` (object definition) -- equivalent to a `class` in many other languages.

### Defining an Object

```lavishscript
objectdef fruit
{
    variable string Name="Apple"
    variable string Color="Red"
}
```

**Key points:**
- `objectdef` keyword followed by the type name.
- Code block with `{` and `}` on separate lines.
- Can contain variables, members, and methods.

### Creating Object Variables

Once defined, create variables of your custom type:

```lavishscript
variable fruit MyFruit
```

This creates a `fruit` object called `MyFruit` with:
- `Name` = "Apple"
- `Color` = "Red"

### Accessing Object Variables

Use `.` (dot) notation in data sequences:

```lavishscript
objectdef fruit
{
    variable string Name="Apple"
    variable string Color="Red"
}

function main()
{
    variable fruit MyFruit

    echo "Name: ${MyFruit.Name}"
    echo "Color: ${MyFruit.Color}"
}
```

**Output:**

```
Name: Apple
Color: Red
```

### Data Sequence as Address

Think of `${MyFruit.Name}` as an address:

- Start at `MyFruit`.
- Navigate to `Name`.
- Retrieve the value.

Just like a postal address leads to a house, a data sequence leads to data.

### Nested Objects

Objects can contain other objects:

```lavishscript
objectdef person
{
    variable string Name="Alice"
    variable int Age=25
}

objectdef team
{
    variable person Leader
    variable person Member
}

function main()
{
    variable team MyTeam

    echo "Leader: ${MyTeam.Leader.Name}"
    echo "Member: ${MyTeam.Member.Name}"
}
```

---

## Members

### What is a Member?

A **member** is a specialized function that provides a value when accessed in a data sequence. Where a `variable` stores a value, a member calculates and returns one.

### Defining a Member

Use the `member` keyword:

```lavishscript
objectdef fruit
{
    variable string Name="Apple"
    variable string Color="Red"

    member ToText()
    {
        return "${Name} ${Color}"
    }
}
```

### The ToText Member

`ToText` is special: it is called when an object's value is accessed directly in a data sequence (the object's "reduces to" value).

```lavishscript
variable fruit MyFruit

echo "Fruit: ${MyFruit}"
; This calls MyFruit.ToText()
```

**Output:** `Fruit: Apple Red`

### Accessing Members

Members are accessed with `.` (dot) notation:

```lavishscript
objectdef fruit
{
    variable string Name="Apple"

    member NameLength()
    {
        return ${Name.Length}
    }
}

function main()
{
    variable fruit MyFruit

    echo "Length: ${MyFruit.NameLength}"
}
```

**Output:** `Length: 5`

### Members vs Variables

| Feature | Variable | Member |
|---|---|---|
| Stores value | Yes | No |
| Calculates value | No | Yes |
| Access syntax | `${Object.Variable}` | `${Object.Member}` |
| Can have parameters | No | Yes |

### Built-in Type Members

Built-in types like `string` ship with many members. The basics:

```lavishscript
variable string Name="Alice"

echo "Length: ${Name.Length}"
echo "Upper: ${Name.Upper}"
echo "Lower: ${Name.Lower}"
```

**Output:**

```
Length: 5
Upper: ALICE
Lower: alice
```

`string` has many more members beyond `Length`/`Upper`/`Lower` (`Find`, `Token`, `Replace`, `Trim`, `Equal`, `URLEncode`, etc.). For the full member list, see the canonical [`ObjectType:string`](https://www.lavishsoft.com/wiki/index.php/ObjectType:string) wiki page or [01b §3.1 Text](01b_LavishScript_Reference.md#31-text).

### Member Parameters

Members can accept parameters using index syntax `[ ]`:

```lavishscript
objectdef calculator
{
    member Add(int A, int B)
    {
        return ${Math.Calc[${A}+${B}]}
    }
}

function main()
{
    variable calculator Calc

    echo "Sum: ${Calc.Add[5,10]}"
}
```

**Output:** `Sum: 15`

Parameters use `[` `]` brackets and are separated by `,` commas.

---

## Methods

### What is a Method?

A **method** is a specialized function that performs an action on an object. Where a member retrieves a value, a method does something.

### Defining a Method

Use the `method` keyword:

```lavishscript
objectdef fruit
{
    variable string Name="Apple"
    variable string Color="Red"

    method SetName(string NewName)
    {
        Name:Set["${NewName~}"]
    }

    method SetColor(string NewColor)
    {
        Color:Set["${NewColor~}"]
    }
}
```

### Calling Methods

Methods use `:` (colon) notation:

```lavishscript
function main()
{
    variable fruit MyFruit

    echo "Before: ${MyFruit.Name}"

    MyFruit:SetName["Banana"]

    echo "After: ${MyFruit.Name}"
}
```

**Output:**

```
Before: Apple
After: Banana
```

### Member vs Method Syntax

| Type | Syntax | Purpose | Example |
|---|---|---|---|
| Member | `.` (dot) | Retrieve value | `${Object.Member}` |
| Method | `:` (colon) | Perform action | `Object:Method` |

### Methods in Data Sequences vs As Commands

Methods can be invoked two ways, with subtly different behavior:

**As a command (recommended for actions):**

```lavishscript
MyFruit:SetName["Banana"]
```

The method runs. The result is discarded.

**Inside a data sequence:**

```lavishscript
echo "${MyFruit:SetName["Banana"]}"
```

The method runs, then the resulting object's reduced-text representation is emplaced into the data sequence.

There are two further wrinkles when calling methods inside data sequences:

1. The method call retains the original object (which is what allows method chaining -- see below).
2. If the method `return FALSE`s to indicate failure or destruction, the entire data sequence resolves to `NULL` instead of the object's reduced text.

```lavishscript
method Destroy()
{
    ; Cleanup code here
    return FALSE
}
```

**Best practice:** invoke methods as commands when you intend an action. Use the data-sequence form only when you specifically want to chain or to detect failure via NULL.

### Method Chaining

Methods can be chained:

```lavishscript
function main()
{
    variable fruit MyFruit

    MyFruit:SetName["Banana"]:SetColor["Yellow"]

    echo "Fruit: ${MyFruit.Name} ${MyFruit.Color}"
}
```

**Output:** `Fruit: Banana Yellow`

### String Set Method

The built-in `string` type has a `Set` method to change its value:

```lavishscript
variable string Name="Alice"

Name:Set["Bob"]

echo "${Name}"  ; Output: Bob
```

### Parameter Escaping in Methods

When passing string parameters to methods, quote and escape (the same rule as the [Tilde Escape](#the-tilde-escape----required-for-re-parsed-output)):

```lavishscript
objectdef person
{
    variable string Name

    method SetName(string NewName)
    {
        Name:Set["${NewName~}"]
    }
}

function main()
{
    variable person P
    variable string NewName="John \"Johnny\" Smith"

    P:SetName["${NewName~}"]
}
```

---

## Initialize and Shutdown

### The Initialize Method

`Initialize` is a special method called when an object is created (instantiated).

**Purpose:**
1. Event handler for object creation.
2. Accepts initial values.
3. Sets up default state.

```lavishscript
objectdef fruit
{
    variable string Name
    variable string Color

    method Initialize(string InitialName="Apple")
    {
        This:SetName["${InitialName~}"]
        This:SetColor["Red"]
    }

    method SetName(string NewName)
    {
        Name:Set["${NewName~}"]
    }

    method SetColor(string NewColor)
    {
        Color:Set["${NewColor~}"]
    }
}
```

### Using Initialize

```lavishscript
function main()
{
    variable fruit Fruit1                ; Uses default "Apple"
    variable fruit Fruit2="Banana"       ; Passes "Banana" to Initialize

    echo "Fruit1: ${Fruit1.Name}"
    echo "Fruit2: ${Fruit2.Name}"
}
```

**Output:**

```
Fruit1: Apple
Fruit2: Banana
```

### The Shutdown Method

`Shutdown` is called when an object is destroyed.

**Purpose:**
1. Event handler for object destruction.
2. Cleanup resources (destroy GUI, close files, etc.).
3. No parameters accepted.

```lavishscript
objectdef logger
{
    method Initialize()
    {
        echo "Logger created"
    }

    method Shutdown()
    {
        echo "Logger destroyed"
    }
}

function main()
{
    variable logger L
    echo "Logger active"
}
```

**Output:**

```
Logger created
Logger active
Logger destroyed
```

### The This Reference

`This` is a special reference to the current object being operated on -- the object whose method or member is currently executing.

Inside a member or method, you don't have access to the variable name (like `MyFruit`). You only know it's "some fruit object". `This` refers to that object.

```lavishscript
objectdef fruit
{
    variable string Name

    method Initialize(string InitialName)
    {
        ; Use This to call another method on the same object
        This:SetName["${InitialName~}"]
    }

    method SetName(string NewName)
    {
        Name:Set["${NewName~}"]
    }
}
```

### Accessing Members and Methods with This

```lavishscript
objectdef person
{
    variable string FirstName
    variable string LastName

    member FullName()
    {
        return "${This.FirstName} ${This.LastName}"
    }

    method SetName(string First, string Last)
    {
        This.FirstName:Set["${First~}"]
        This.LastName:Set["${Last~}"]
    }
}
```

**Why `This.FirstName` instead of just `FirstName`?**

Inside an `objectdef`, variables defined in the objectdef CAN be accessed directly by name (e.g., `FirstName:Set[...]`). Members and methods MUST be accessed through an object reference like `This` (e.g., `This:SetName[...]`). Using `This.VariableName` for variables is optional but is recommended for clarity and forward-compatibility.

### atexit and Object Lifecycle Globals

A common pattern: declare a script-scope or global object whose `Initialize`/`Shutdown` runs the entire script's lifecycle, with an `atexit` atom forwarding the exit signal.

```lavishscript
objectdef MyApp
{
    variable bool IsRunning=TRUE

    method Initialize()
    {
        echo "App started"
    }

    method Shutdown()
    {
        echo "Shutting down..."
        IsRunning:Set[FALSE]
    }
}

variable(global) MyApp App

atom atexit()
{
    App:Shutdown
}

function main()
{
    App:Initialize

    while ${App.IsRunning}
    {
        ; Main loop
        waitframe
    }
}
```

**Avoid shadowing built-in TLOs.** Don't name a custom global `Script`, `Math`, `Time`, `System`, `Event`, etc. -- those are built-in TLOs (see [Top-Level Objects](#top-level-objects)) and your global will shadow them, breaking later `${Script.RunningTime}`-style references in the same script. Use a clearly distinct name like `App`, `MyApp`, `Bot`, `Controller`, or your project name.

---

## Inheritance and Type Casting

### Object Inheritance

Inheritance allows one object type to inherit properties and behavior from another.

**Example:** Apples, Bananas, and Cherries are all fruit.

```lavishscript
objectdef fruit
{
    variable string Name
    variable string Color

    method Initialize(string InitialName="Unknown")
    {
        This:SetName["${InitialName~}"]
    }

    method SetName(string NewName)
    {
        Name:Set["${NewName~}"]
    }
}
```

### Creating Inherited Objects

Use the `inherits` keyword:

```lavishscript
objectdef apple inherits fruit
{
    method Initialize()
    {
        This(fruit):Initialize["Apple"]
        This:SetColor["Red"]
    }

    method SetColor(string NewColor)
    {
        Color:Set["${NewColor~}"]
    }
}
```

### How Inheritance Works

An `apple` object has:

- All variables from `fruit` (Name, Color).
- All methods from `fruit` (SetName).
- Plus any new variables and methods defined in `apple` (SetColor).

```lavishscript
function main()
{
    variable apple MyApple

    ; Inherited from fruit
    echo "Name: ${MyApple.Name}"

    ; Defined in apple
    MyApple:SetColor["Green"]

    echo "Color: ${MyApple.Color}"
}
```

### Overriding Members and Methods

Child objects can override (replace) members and methods from parent objects:

```lavishscript
objectdef fruit
{
    variable string Name

    member ToText()
    {
        return "Fruit: ${Name}"
    }
}

objectdef cherry inherits fruit
{
    ; This OVERRIDES fruit.ToText
    member ToText()
    {
        return "Cherry: ${Name}"
    }
}

function main()
{
    variable fruit F
    variable cherry C

    echo "${F}"  ; Output: Fruit: Unknown
    echo "${C}"  ; Output: Cherry: Unknown
}
```

### Type Casting -- Two Forms

Type casting lets you access a parent type's members or methods even when they have been overridden. There are two cast forms with distinct purposes:

**Named-type cast: `Object(<typeName>)`**

Treats the object as the named type. Used most often inside an overriding method to call the named parent's version:

```lavishscript
objectdef banana inherits fruit
{
    method Initialize()
    {
        ; Call fruit's Initialize method specifically
        This(fruit):Initialize["Banana"]

        ; Then do banana-specific stuff
        This:SetColor["Yellow"]
    }
}
```

**Parent-keyword cast: `Object(parent)`**

`(parent)` is a special keyword cast that resolves to the immediate parent of whatever the object's type is, regardless of name. Use it when you want "call the inherited type's method" without naming the parent type explicitly:

```lavishscript
method Initialize()
{
    This(parent):Initialize          ; Call inherited type's Initialize
    ; Child-specific initialization
}
```

The `(parent)` form keeps working if you later refactor the inheritance chain, because it is resolved against the actual parent type at parse time. Prefer it over a named-type cast when you're calling "the inherited version" of the same-named method.

### Type Casting Example

```lavishscript
objectdef fruit
{
    variable string Name="Unknown"

    method Initialize(string InitialName)
    {
        Name:Set["${InitialName~}"]
    }
}

objectdef apple inherits fruit
{
    method Initialize()
    {
        ; Call the parent's Initialize using the (parent) keyword cast
        This(parent):Initialize["Apple"]

        echo "Apple initialized!"
    }
}

function main()
{
    variable apple A
}
```

**Output:**

```
Apple initialized!
```

### Inheritance Best Practices

1. **Call parent Initialize when overriding:**

```lavishscript
method Initialize()
{
    This(parent):Initialize
    ; Child-specific initialization
}
```

2. **Use inheritance for "is-a" relationships.**
   - An Apple **is a** Fruit -- inherit.
   - An Apple **has a** Color -- use a variable instead.

3. **Override carefully.** Make sure you understand what the parent method does.

---

## Atoms and Atomicity

### What is Atomicity?

"Atomic" means **unbroken, as a whole**. In programming, atomic functions must complete immediately without waiting.

### Atomic vs Non-Atomic

| Function Type | Can `wait` / `waitframe`? | Blocks Frame? |
|---|---|---|
| `function` | Yes | No |
| `atom` | No | Yes |
| `member` | No | Yes |
| `method` | No | Yes |

Atomic functions (`atom`, `member`, `method`) block the host frame from advancing until they complete.

### Why Atomicity Matters

Members and methods MUST be atomic because they're used in data sequences, which must resolve immediately:

```lavishscript
echo "${Object.Member}"
```

If `Member` could wait for frames, the `echo` command would be stuck waiting, freezing the entire host until the wait elapsed.

### Defining an Atom

Use the `atom` keyword:

```lavishscript
atom MyCommand()
{
    echo "Command executed!"
}
```

### Using Atoms as Commands

Atoms are executed as commands by name (no `call` needed, unlike functions):

```lavishscript
atom greet(string Name)
{
    echo "Hello ${Name}!"
}

function main()
{
    greet "Alice"
    greet "Bob"
}
```

**Output:**

```
Hello Alice!
Hello Bob!
```

### Atoms vs Functions

| Feature | function | atom |
|---|---|---|
| Can use `waitframe` | Yes | No |
| Can use `wait` | Yes | No |
| Can run for minutes | Yes | No |
| Invoked as named command (no `call`) | No | Yes |
| Invoked with `call` | Yes | No |
| Blocks frame | No | Yes |

### Registered Atoms (cross-script invocation)

The `atom` keyword defines an atom in the local script. To make an atom callable from outside the defining script (from another script, from the console, or from a `Relay`-broadcast command), use the [`AddAtom`](https://www.lavishsoft.com/wiki/index.php/Command:AddAtom) / [`DeleteAtom`](https://www.lavishsoft.com/wiki/index.php/Command:DeleteAtom) / [`ExecuteAtom`](https://www.lavishsoft.com/wiki/index.php/Command:ExecuteAtom) commands:

- `AddAtom <name>` -- register the atom under a global name.
- `ExecuteAtom <name> [args]` -- invoke a registered atom by name.
- `DeleteAtom <name>` -- unregister.

This is the mechanism `Relay` uses under the hood to invoke atoms in other sessions. See [01b §2.1 Atom Registration](01b_LavishScript_Reference.md#21-lavishscript-core-commands) for the table of commands and [Input Emulation](#input-emulation) below for the multi-boxing context.

### The atexit Atom

`atexit` is a special atom called when a script is ending. Use it for cleanup before script shutdown (destroy GUI, save data, close files, etc.):

```lavishscript
atom atexit()
{
    echo "Script ending - cleanup complete"
}

function main()
{
    echo "Script running"
}
```

**Output:**

```
Script running
Script ending - cleanup complete
```

### Atomic Restrictions

Cannot use in atomic code:

- `waitframe`
- `wait`
- Anything that depends on frame updates

Attempting these will produce a script error:

```lavishscript
; BAD - This will error!
atom BadAtom()
{
    waitframe  ; ERROR: waitframe not allowed in atomic code
}
```

### Deferring Work From an Atom

Atoms can't `wait`, but they can defer work for the main loop to pick up. Two canonical mechanisms:

**[`QueueCommand`](https://www.lavishsoft.com/wiki/index.php/Command:QueueCommand)** -- append a command to the script's command queue:

```lavishscript
atom OnSomeEvent()
{
    ; Can't wait inside an atom, so defer the work
    QueueCommand "call ProcessEvent"
}
```

The main loop pulls and runs queued commands between frames. Related: [`ExecuteQueued`](https://www.lavishsoft.com/wiki/index.php/Command:ExecuteQueued) (run queued commands now), [`FlushQueued`](https://www.lavishsoft.com/wiki/index.php/Command:FlushQueued) (discard the queue), and the [`QueuedCommands` TLO](https://www.lavishsoft.com/wiki/index.php/TLO:QueuedCommands).

**[`TimedCommand`](https://www.lavishsoft.com/wiki/index.php/Command:TimedCommand)** -- schedule a command to run after a delay (deciseconds), without blocking the calling code:

```lavishscript
atom OnTrigger()
{
    ; Run a follow-up after half a second, non-blocking
    TimedCommand 5 "call FollowUp"
}
```

Use `TimedCommand` when you need a one-shot delayed action; use `QueueCommand` when you want the next main-loop tick to pick it up immediately.

For both commands' full surface, see [01b §2.1 Command Execution and Command Queue](01b_LavishScript_Reference.md#21-lavishscript-core-commands).

---

## Wait Commands

### Script Running Time

The current script's running time (in milliseconds) is exposed via the [`Script` TLO](https://www.lavishsoft.com/wiki/index.php/TLO:Script):

```lavishscript
function main()
{
    echo "Start: ${Script.RunningTime} ms"
    wait 10
    echo "After wait: ${Script.RunningTime} ms"
}
```

**Output:**

```
Start: 0 ms
After wait: 1000 ms
```

### The waitframe Command

`waitframe` waits until the host prepares and renders the next frame.

```lavishscript
function main()
{
    echo "Before frame"
    waitframe
    echo "After frame"
}
```

**When to use:**

- In long-running loops (to avoid maximum CPU usage).
- When waiting for host state changes that update frame-to-frame.
- To yield control back to the host.

### The wait Command

The [`wait`](https://www.lavishsoft.com/wiki/index.php/Command:Wait) command pauses execution for at least a specified duration. It has three forms:

**1. Deciseconds (tenths of a second):**

```lavishscript
wait 10  ; Wait 1 second  (10 deciseconds)
wait 50  ; Wait 5 seconds (50 deciseconds)
```

**2. Seconds (with `-s` flag):**

```lavishscript
wait -s 1.0    ; Wait 1 second
wait -s 2.5    ; Wait 2.5 seconds
```

**3. With early-continue condition:**

```lavishscript
wait <max_deciseconds> <condition>
```

The script waits up to `max_deciseconds` OR until `condition` becomes non-zero, whichever happens first. Both decisecond and `-s` seconds forms support the condition:

```lavishscript
; Wait up to 10 seconds for MyObject to exist
wait 100 ${MyObject(exists)}

; Wait up to 5 seconds (in seconds form) for IsLoaded
wait -s 5.0 ${MyObject.IsLoaded}
```

### Wait Precision is Frame-Quantized

Wait precision depends on host framerate (FPS):

- At **60 FPS**, each frame takes ~16.7ms.
- At **10 FPS**, each frame takes ~100ms.

`wait` cannot resolve below one frame. A request like `wait -s 0.001` (1 millisecond) actually waits one full frame (~16.7ms at 60fps). The rule: waits are quantized to frame intervals, so do NOT rely on sub-frame precision.

### Wait Example

```lavishscript
function main()
{
    echo "Starting: ${Script.RunningTime}ms"

    wait -s 1.0
    echo "After 1 second: ${Script.RunningTime}ms"

    wait 50
    echo "After 5 more seconds: ${Script.RunningTime}ms"
}
```

**Output:**

```
Starting: 0ms
After 1 second: 1000ms
After 5 more seconds: 6000ms
```

### Infinite Loops Need waitframe

Always use `waitframe` (or `wait`) in infinite loops:

```lavishscript
function main()
{
    variable bool Running=TRUE

    while ${Running}
    {
        ; Do work here

        waitframe  ; IMPORTANT - Yields the frame
    }
}
```

Without `waitframe`, the loop consumes the entire frame budget and the host appears frozen.

### Wait Restrictions

`wait` and `waitframe` are forbidden in atomic code:

```lavishscript
; ERROR - members are atomic
member BadMember()
{
    waitframe
    return "value"
}

; ERROR - atoms are atomic
atom BadAtom()
{
    wait 10
}
```

Only `function`s can wait. From inside an atom or method, defer via [`QueueCommand`](#deferring-work-from-an-atom) or [`TimedCommand`](#deferring-work-from-an-atom) instead.

---

## Conditional Branching

### The if Statement

Execute code only if a condition is met:

```lavishscript
if condition
    code
```

**With code block:**

```lavishscript
if condition
{
    multiple
    lines
    of code
}
```

### Conditions

Conditions evaluate to **zero or non-zero**:

- `0`, `NULL`, `FALSE` → condition NOT met.
- non-zero, `TRUE` → condition met.

### Comparison Operators

Numeric:

| Operator | Meaning | Example |
|---|---|---|
| `<` | Less than | `${Value}<10` |
| `>` | Greater than | `${Value}>100` |
| `<=` | Less than or equal | `${Value}<=10` |
| `>=` | Greater than or equal | `${Value}>=100` |
| `==` | Equal to | `${Value}==5` |
| `!=` | Not equal to | `${Value}!=0` |

String members:

| Member | Use |
|---|---|
| `.Equal[text]` | Case-insensitive equality. |
| `.NotEqual[text]` | Case-insensitive inequality. |
| `.EqualCS[text]` | Case-sensitive equality. |
| `.NotEqualCS[text]` | Case-sensitive inequality. |
| `.Find[substring]` | Returns 1-based position of substring or NULL. |
| `.Compare[text]` | <0, 0, or >0 for case-insensitive lexicographic compare. |

For complete operator and formula syntax, see [LavishScript:Mathematical Formulae](https://www.lavishsoft.com/wiki/index.php/LavishScript:Mathematical_Formulae).

### Simple if Example

```lavishscript
function main(int Value=5)
{
    if ${Value}<10
        echo "Value is less than 10"
}
```

### if with Code Block

```lavishscript
function main(int Health=50)
{
    if ${Health}<20
    {
        echo "Warning: Low health!"
        echo "Health: ${Health}"
    }
}
```

### The elseif Statement

Chain multiple conditions:

```lavishscript
function main(int Value=15)
{
    if ${Value}<10
        echo "Less than 10"
    elseif ${Value}<50
        echo "Between 10 and 49"
    elseif ${Value}<100
        echo "Between 50 and 99"
}
```

**How it works:**
1. Check the first `if`. If true, execute and skip the rest.
2. If false, check the first `elseif`.
3. Continue down the chain until a match is found.

### The else Statement

Execute code when no conditions match:

```lavishscript
function main(int Value=150)
{
    if ${Value}<10
        echo "Less than 10"
    elseif ${Value}<50
        echo "Between 10 and 49"
    else
        echo "50 or higher"
}
```

### Complete if-elseif-else Example

```lavishscript
function main(int Health=100)
{
    if ${Health}<=0
    {
        echo "Dead"
    }
    elseif ${Health}<20
    {
        echo "Critical health!"
    }
    elseif ${Health}<50
    {
        echo "Low health"
    }
    elseif ${Health}<80
    {
        echo "Moderate health"
    }
    else
    {
        echo "Good health"
    }
}
```

### Logical Operators

Combine conditions with `&&` (AND), `||` (OR), and `!` (NOT):

```lavishscript
; AND - both must be true
if ${Health}<50 && ${Power}<20
    echo "Low on both!"

; OR - either can be true
if ${Health}<20 || ${Power}<10
    echo "Emergency!"

; NOT - prefix to negate
if !${MyObject(exists)}
    echo "MyObject does not exist"
```

### Checking Existence

Check if an object exists with `(exists)`:

```lavishscript
if ${MyObject(exists)}
    echo "Object exists"
else
    echo "No object"
```

`(exists)` is the canonical NULL-check pattern. Use it before reaching into a member chain that might dereference NULL.

---

## Loops

### The while Loop

Repeat code while a condition is met:

```lavishscript
while condition
{
    code
}
```

### Basic while Example

```lavishscript
function main()
{
    variable int Count=1

    while ${Count}<=10
    {
        echo "${Count}"
        Count:Inc
    }
}
```

**Output:** `1 2 3 4 5 6 7 8 9 10`

### int:Inc Method

Increment an integer:

```lavishscript
Count:Inc        ; Increment by 1
Count:Inc[5]     ; Increment by 5
Count:Inc[1+1]   ; Increment by result of math (2)
```

### Infinite Loop

```lavishscript
function main()
{
    while TRUE
    {
        echo "Running..."
        waitframe  ; IMPORTANT - prevents CPU overload
    }
}
```

Always use `waitframe` in infinite loops.

### The do-while Loop

Execute code at least once, then check condition:

```lavishscript
do
{
    code
}
while condition
```

### do-while Example

```lavishscript
function main()
{
    variable int Count=1

    do
    {
        echo "${Count}"
        Count:Inc
    }
    while ${Count}<=10
}
```

**Difference from `while`:**
- `while` checks condition before executing.
- `do-while` checks condition after executing (guarantees at least one execution).

### The break Statement

Exit a loop immediately:

```lavishscript
function main()
{
    variable int Count=1

    while TRUE
    {
        echo "${Count}"

        if ${Count}>=5
            break  ; Exit loop when Count reaches 5

        Count:Inc
    }

    echo "Loop exited"
}
```

**Output:**

```
1
2
3
4
5
Loop exited
```

### The continue Statement

Skip the rest of the iteration and advance to the next:

```lavishscript
function main()
{
    variable int Count=0

    while ${Count}<10
    {
        Count:Inc

        ; Modulo-2 is non-zero for ODD numbers, so continue triggers on odd
        if ${Math.Calc[${Count}%2]}
            continue

        echo "${Count}"
    }
}
```

**Output:** `2 4 6 8 10` (only even numbers, because the `continue` skips odd ones).

### The for Loop

Compact loop syntax combining initialization, condition, and increment:

```lavishscript
for ( start ; condition ; advance )
{
    code
}
```

### for Loop Example

```lavishscript
function main()
{
    variable int Count

    for ( Count:Set[1] ; ${Count}<=10 ; Count:Inc )
    {
        echo "${Count}"
    }
}
```

**Equivalent while loop:**

```lavishscript
variable int Count
Count:Set[1]
while ${Count}<=10
{
    echo "${Count}"
    Count:Inc
}
```

### int:Set Method

Set an integer value (accepts a math formula):

```lavishscript
Count:Set[1]            ; Set to 1
Count:Set[5+5]          ; Set to 10
Count:Set[${Count}*2]   ; Double the current value
```

### for Loop with Step

```lavishscript
function main()
{
    variable int Count

    ; Count by 5s: 5, 10, 15, ..., 100
    for ( Count:Set[5] ; ${Count}<=100 ; Count:Inc[5] )
    {
        echo "${Count}"
    }
}
```

### Nested Loops

```lavishscript
function main()
{
    variable int I
    variable int J

    for ( I:Set[1] ; ${I}<=3 ; I:Inc )
    {
        for ( J:Set[1] ; ${J}<=3 ; J:Inc )
        {
            echo "I=${I} J=${J}"
        }
    }
}
```

---

## Switch Statements

### The switch Statement

Select one of several code branches based on a value:

```lavishscript
switch value
{
    case branch1
        code
        break
    case branch2
        code
        break
    default
        code
        break
}
```

### Basic switch Example

```lavishscript
function main(string Color="Red")
{
    switch ${Color}
    {
        case Red
            echo "Color is red"
            break
        case Blue
            echo "Color is blue"
            break
        case Green
            echo "Color is green"
            break
        default
            echo "Unknown color"
            break
    }
}
```

### How switch Works

1. Evaluate the `switch` value.
2. Case-insensitive string compare against each `case`.
3. If match found, execute code from that point downward.
4. `break` exits the switch block.
5. If no match, execute `default` (if provided).

### Fall-through Behavior

Without `break`, execution continues to the next case:

```lavishscript
switch ${Value}
{
    case 1
    case 2
    case 3
        echo "Value is 1, 2, or 3"
        break
    case 4
        echo "Value is 4"
        break
}
```

This matches multiple values to the same code block.

### The default Branch

Executes when no `case` matches:

```lavishscript
function main(string Input="Unknown")
{
    switch ${Input}
    {
        case Yes
            echo "Confirmed"
            break
        case No
            echo "Declined"
            break
        default
            echo "Invalid input: ${Input}"
            break
    }
}
```

### The variablecase Branch

Use when branch values cannot be hard-coded (contain data sequences):

```lavishscript
function main()
{
    variable string ExpectedValue="Apple"
    variable string Input="Apple"

    switch ${Input}
    {
        case Orange
            echo "Input is Orange"
            break
        variablecase ${ExpectedValue}
            echo "Input matches expected: ${ExpectedValue}"
            break
        default
            echo "No match"
            break
    }
}
```

**How variablecase works:**
1. Check all `case` branches first.
2. If no `case` matches, check `variablecase` branches in order.
3. Resolve data sequences in `variablecase` only when needed.
4. Execute the first matching `variablecase`.

### switch Best Practices

1. Always use `break` (unless intentional fall-through).
2. Provide `default` for unexpected values.
3. Use `case` for constants, `variablecase` for variables.
4. Case values are verbatim text -- don't add `:` like in C/C++.
5. **CRITICAL:** Never use quotes around case values.

### Important: No Quotes in Case Values

**Case values MUST NOT be quoted, even for strings with spaces:**

```lavishscript
; WRONG - this will NEVER match!
switch ${textValue}
{
    case "Hello World"        ; Will NOT work
        echo "Matched"
        break
}

; CORRECT - no quotes!
switch ${textValue}
{
    case Hello World          ; Works
        echo "Matched"
        break
}
```

This applies to all case values:
- Strings with spaces: `case Some Phrase`
- Negative numbers: `case -10`
- Numbers: `case 42`
- Any text: `case CategoryName`

**Why this matters:** adding quotes makes the switch look for the literal text including the quote characters, which will never match your actual values.

### Complete switch Example

```lavishscript
function ProcessCommand(string Command)
{
    switch ${Command}
    {
        case start
        case begin
        case go
            echo "Starting..."
            break

        case stop
        case end
        case quit
            echo "Stopping..."
            break

        case help
            echo "Available commands: start, stop, help"
            break

        default
            echo "Unknown command: ${Command}"
            echo "Type 'help' for available commands"
            break
    }
}
```

---

## Collections and Lists

LavishScript provides several container types. The two used most often are `index` (ordered list, 1-based) and `collection` (key-value map). Other containers -- `array`, `queue`, `stack`, `set`, `variablescope`, `objectcontainer` -- are documented in [01b §3.5 Containers](01b_LavishScript_Reference.md#35-containers).

### What is an Index?

An [`index`](https://www.lavishsoft.com/wiki/index.php/ObjectType:index) is a dynamically sized list of objects.

**Key features:**
- Dynamic size -- grows automatically as you add items.
- Type-safe -- specify element type like `index:int` or `index:string`.
- 1-based indexing -- first element is at position 1, not 0.
- JSON serialization built in.

### Creating an Index

Use `variable index:type Name` syntax:

```lavishscript
variable index:int Numbers
variable index:string Names
variable index:person People
```

You must specify the element type after the colon.

### Adding Items

Use the `Insert` method:

```lavishscript
variable index:int Numbers

Numbers:Insert[17]
Numbers:Insert[42]
Numbers:Insert[35]
Numbers:Insert[9000]
```

**Result:** `[17, 42, 35, 9000]`

### Accessing Items

Use bracket `[]` syntax (1-based indexing):

```lavishscript
echo "First number: ${Numbers[1]}"   ; Output: 17
echo "Third number: ${Numbers[3]}"   ; Output: 35
```

Indices start at 1, not 0.

### Removing Items

Use the `Remove` method with the position number:

```lavishscript
variable index:int Numbers

Numbers:Insert[17]
Numbers:Insert[42]
Numbers:Insert[35]

Numbers:Remove[2]  ; Remove 42 (second item)

echo "${Numbers[2]}"  ; Output: NULL (position 2 is now empty)
```

Removing creates a gap -- the position becomes NULL.

### Collapsing Gaps

Use `Collapse` to remove gaps by shifting items:

```lavishscript
Numbers:Remove[2]    ; Remove 42 - creates gap
Numbers:Collapse     ; Shift items to fill gap

echo "${Numbers[2]}"  ; Output: 35 (was at position 3)
```

### Common Index Operations

| Operation | Method or Member | Example |
|---|---|---|
| Add item | `Insert[value]` | `List:Insert[42]` |
| Remove item | `Remove[position]` | `List:Remove[3]` |
| Fill gaps | `Collapse` | `List:Collapse` |
| Clear all | `Clear` | `List:Clear` |
| Get count | `.Used` | `${List.Used}` |
| To JSON | `.AsJSON` | `${List.AsJSON}` |
| From JSON | `FromJSON[json]` | `List:FromJSON[...]` |

For the full member/method surface (`Next`, `Get`, `Move`, `Swap`, `Shift`, `Resize`, `RemoveByQuery`, `ForEach`, etc.), see [01b §3.5 Containers](01b_LavishScript_Reference.md#35-containers) or the canonical [`ObjectType:index`](https://www.lavishsoft.com/wiki/index.php/ObjectType:index) wiki page.

### Index with Custom Objects

Indices can store custom objectdefs:

```lavishscript
objectdef person
{
    variable string FirstName
    variable string LastName

    method Initialize(string first, string last)
    {
        FirstName:Set["${first~}"]
        LastName:Set["${last~}"]
    }

    member:string AsJSON()
    {
        variable jsonvalue jo={}
        jo:Set[first_name,"${FirstName.AsJSON~}"]
        jo:Set[last_name,"${LastName.AsJSON~}"]
        return "${jo.AsJSON~}"
    }
}

function main()
{
    variable index:person People

    ; Insert passes parameters to person:Initialize
    People:Insert["John","Doe"]
    People:Insert["Jane","Doe"]
    People:Insert["John","Public"]

    echo "${People.AsJSON}"
}
```

### Iteration

Walking a container without index-assumption fragility uses an [`iterator`](https://www.lavishsoft.com/wiki/index.php/ObjectType:iterator). Most containers expose a `GetIterator` method that initializes one:

```lavishscript
variable index:string Names
variable iterator It

Names:Insert["Alice"]
Names:Insert["Bob"]
Names:Insert["Charlie"]

Names:GetIterator[It]

if ${It:First(exists)}
{
    do
    {
        echo "Name: ${It.Value}"
    }
    while ${It:Next(exists)}
}
```

The compact alternative is the [`ForEach` Top-Level Object](https://www.lavishsoft.com/wiki/index.php/TLO:ForEach), which exposes `Key` and `Value` for the current iteration when you use a container's `ForEach` method:

```lavishscript
Names:ForEach[ "echo Name: \${ForEach.Value}" ]
```

For full iteration surface (`Last`, `Previous`, `Jump`, `SetValue`, `IsValid`, `Reversible`, `Constant`, `RandomAccess`), see [01b §3.7 Iteration](01b_LavishScript_Reference.md#37-iteration).

### JSON Serialization

**`AsJSON` -- convert to JSON array:**

```lavishscript
variable index:int Numbers

Numbers:Insert[17]
Numbers:Insert[42]
Numbers:Insert[35]

echo "${Numbers.AsJSON}"  ; Output: [17,42,35]
```

**`FromJSON` -- load from JSON array:**

```lavishscript
objectdef person
{
    variable string FirstName
    variable string LastName

    ; Initialize accepts a jsonvalue
    method Initialize(jsonvalue jo)
    {
        FirstName:Set["${jo.Get[first_name]~}"]
        LastName:Set["${jo.Get[last_name]~}"]
    }
}

variable index:person People

People:FromJSON["$$>[
    {\"first_name\":\"John\",\"last_name\":\"Doe\"},
    {\"first_name\":\"Jane\",\"last_name\":\"Doe\"}
]<$$"]

; FromJSON calls person:Initialize for each JSON object
```

The `$$> ... <$$` heredoc syntax delimits a multi-line string literal that allows quotes and braces inline without backslash-escaping.

### Index vs Collection

LavishScript also provides [`collection`](https://www.lavishsoft.com/wiki/index.php/ObjectType:collection) (case-insensitive key-value map). Use:

- **`index`** when you need ordered lists.
- **`collection`** when you need key-value lookups.

For JSON-heavy work, see [13_JSON_Guide.md](13_JSON_Guide.md).

---

## Events

LavishScript **events** are named hooks that any number of atoms can attach to. When an event executes, every attached atom runs (in arbitrary order) with the event's parameters. Events are how reactive code is wired up: instead of polling for a state change in a loop, you attach an atom to the relevant event and let the engine call you when the change happens.

### Lifecycle

1. **Register** the event by name (only needed for events you create -- built-in and extension-fired events are pre-registered).
2. **Attach** one or more atoms to the event.
3. **Execute** the event (yourself, or let the engine fire it).
4. **Detach** atoms when you no longer want them to fire.

### Register an Event

The [`lavishscript` object type](https://www.lavishsoft.com/wiki/index.php/ObjectType:lavishscript) provides a `RegisterEvent` method on the `LavishScript` TLO:

```lavishscript
LavishScript:RegisterEvent["My Event"]
```

Registering an event twice is a no-op. There is also a variable-declaration form that auto-registers on declaration and unregisters on scope exit:

```lavishscript
declare MyEvent event "My Event"
```

### Attach an Atom

The [`event` object type's](https://www.lavishsoft.com/wiki/index.php/ObjectType:event) `AttachAtom` method binds an atom to the event. The event is looked up via the [`Event` Top-Level Object](https://www.lavishsoft.com/wiki/index.php/TLO:Event):

```lavishscript
atom OnMyEvent(string Message)
{
    echo "Got event: ${Message}"
}

function main()
{
    Event[My Event]:AttachAtom[OnMyEvent]

    ; ... later ...
    Event[My Event]:Execute["hello"]
}
```

Any number of atoms can attach to the same event. Atoms execute in arbitrary order; do not rely on attach order.

### Execute an Event

Two methods on the `event` object type:

- `Execute[<arg1>,<arg2>,...]` -- fire the event with arguments.
- `ThisExecute[<contextObject>,<arg1>,...]` -- fire the event with the first argument bound as `This` inside attached atoms.

### Detach an Atom

```lavishscript
Event[My Event]:DetachAtom[OnMyEvent]
```

Detach in `atexit` or `Shutdown` to keep the event registry tidy when the script ends.

### Built-in Events

Some events are fired by the engine automatically. The simplest example is `Alias Added`, which fires whenever an alias is created:

```lavishscript
atom OnAliasAdded(string Name)
{
    echo "New alias registered: ${Name}"
}

function main()
{
    Event[Alias Added]:AttachAtom[OnAliasAdded]
    while TRUE
        waitframe
}
```

Inner Space subsystems (input, audio, display, sessions) and the LavishMachine Tasks system fire their own events; consult the relevant wiki page or [Category:LavishScript Events](https://www.lavishsoft.com/wiki/index.php/Category:LavishScript_Events).

For the canonical reference and the full lifecycle including the C-side API, see [LavishScript:Events](https://www.lavishsoft.com/wiki/index.php/LavishScript:Events) and [01b §5](01b_LavishScript_Reference.md#5-events).

---

## Triggers

A **trigger** fires a command when a registered text pattern is seen in console output. Triggers are how scripts react to log lines, chat messages, or any other text stream that lands on the console.

### Lifecycle

- [`AddTrigger <pattern> <command>`](https://www.lavishsoft.com/wiki/index.php/Command:AddTrigger) -- register a pattern and the command to run when matched.
- [`RemoveTrigger <pattern>`](https://www.lavishsoft.com/wiki/index.php/Command:RemoveTrigger) -- unregister.
- [`WaitFor <pattern> [timeout]`](https://www.lavishsoft.com/wiki/index.php/Command:WaitFor) -- pause the calling script until the pattern is seen, or the optional timeout elapses.

Triggers persist for the session until removed or until the session ends. For the canonical reference see [LavishScript:Triggers](https://www.lavishsoft.com/wiki/index.php/LavishScript:Triggers) and [01b §6](01b_LavishScript_Reference.md#6-triggers).

### Trigger Example

```lavishscript
function main()
{
    AddTrigger "Hello*" "echo Triggered on greeting"

    ; ... script keeps running and console output is monitored ...
    while TRUE
        waitframe
}
```

When any console line matches `Hello*`, the registered command runs.

### WaitFor Pattern

```lavishscript
function main()
{
    echo "Waiting for ready..."
    WaitFor "Ready" 100   ; up to 10 seconds
    echo "Got ready signal"
}
```

---

## Aliases

An **alias** is a user-defined shortcut for one or more commands. Once registered, typing the alias name behaves like typing the underlying command(s). Aliases are session-wide and persist until removed.

### Alias Command

The [`Alias`](https://www.lavishsoft.com/wiki/index.php/Command:Alias) command has a few forms:

- `Alias <name> <command>` -- register an alias.
- `Alias -list` -- list current aliases.
- `Alias -delete <name>` -- remove an alias.

### Example

```lavishscript
Alias greet "echo Hello there"

; Now typing
greet
; behaves like
echo Hello there
```

### Hooking Alias Creation

The engine fires the [built-in `Alias Added` event](#built-in-events) when a new alias is registered. Attach an atom if you need to react to alias creation.

For the canonical reference see [LavishScript:Aliases](https://www.lavishsoft.com/wiki/index.php/LavishScript:Aliases).

---

## Modules

A **module** is a compiled LavishScript library loaded with the [`Module`](https://www.lavishsoft.com/wiki/index.php/Command:Module) command. Modules can add commands, object types, TLOs, and atoms to the running session. They are session-wide and stay loaded until unloaded.

### Module Command

- `Module -load <name>` -- load a module by name.
- `Module -unload <name>` -- unload.
- `Module -list` -- list loaded modules.

### First-Party Modules

Lavish Software ships several optional modules:

| Module | Purpose |
|---|---|
| [LSModule:Sound](https://www.lavishsoft.com/wiki/index.php/LSModule:Sound) | Sound effect playback (separate from the Inner Space audio object type). |
| [LSModule:MySQL](https://www.lavishsoft.com/wiki/index.php/LSModule:MySQL) | MySQL client. |
| [LSModule:Regex](https://www.lavishsoft.com/wiki/index.php/LSModule:Regex) | Regular-expression matching. |
| [LSModule:Targz](https://www.lavishsoft.com/wiki/index.php/LSModule:Targz) | `.tar.gz` archive support (adds the `tar` command). |
| [LSModule:Ventrilo](https://www.lavishsoft.com/wiki/index.php/LSModule:Ventrilo) | Ventrilo voice client control. |
| [LSModule:Scheduler](https://www.lavishsoft.com/wiki/index.php/LSModule:Scheduler) | Job scheduler. |

Load only what you need. For the canonical module reference and per-module surface (commands, object types), see [01b §7 Modules (LSModule)](01b_LavishScript_Reference.md#7-modules-lsmodule) and the [Category:LavishScript Modules](https://www.lavishsoft.com/wiki/index.php/Category:LavishScript_Modules) wiki index.

---

## Input Emulation

Inner Space provides commands and object types for emulating keyboard, mouse, and hotkey input. These are Inner-Space-Kernel-side features (provided by the host platform), NOT LavishScript-core -- they are unavailable in any LavishScript host that lacks Inner Space.

This section gives the conceptual onboarding. For the full per-command syntax and the input object types (`bind`, `keyboard`, `mouse`, `input`, `button`, `dpad`, `axis`), see [01b §2.2 Input Emulation](01b_LavishScript_Reference.md#22-inner-space-kernel-commands) and [01b §3.18](01b_LavishScript_Reference.md#318-inner-space-input).

### Hotkeys (Bind / GlobalBind)

[`Bind`](https://www.lavishsoft.com/wiki/index.php/ISKernel:Bind_(Command)) registers a session-level hotkey -- a key combination that runs a command when pressed (or released) while the session is in focus.

```
Bind <name> <combo> <command>
Bind -press <name> <combo> <command>
Bind -release <name> <combo> <command>
Bind -list
Bind -delete <name>
Bind -clear
```

[`GlobalBind`](https://www.lavishsoft.com/wiki/index.php/ISKernel:GlobalBind_(Command)) is identical in form but the hotkey is Windows-global -- it fires no matter which window has focus.

### Key Press Emulation (Press / Type)

[`Press`](https://www.lavishsoft.com/wiki/index.php/ISKernel:Press_(Command)) emulates pressing keys:

```
Press <combo>              ; press, then release
Press -hold <combo>        ; press and hold
Press -release <combo>     ; release after a -hold
Press -nomodifiers <combo> ; release modifiers for the duration
Press -keylist             ; list valid key names
```

[`Type`](https://www.lavishsoft.com/wiki/index.php/ISKernel:Type_(Command)) emulates typing literal text (does not include Enter):

```
Type "Hello, how are you"
```

### Mouse Emulation (MouseTo / MouseClick)

- [`MouseTo <x> <y>`](https://www.lavishsoft.com/wiki/index.php/ISKernel:MouseTo_(Command)) -- move the mouse cursor.
- [`MouseClick -hold <left|right>`](https://www.lavishsoft.com/wiki/index.php/ISKernel:MouseClick_(Command)) -- press a mouse button.
- `MouseClick -release <left|right>` -- release a mouse button.

A complete mouse click typically wants a frame between press and release:

```lavishscript
MouseClick -hold left
waitframe
MouseClick -release left
```

### Macros

[`Macro`](https://www.lavishsoft.com/wiki/index.php/ISKernel:Macro_(Command)) records and plays back keyboard and mouse sequences. Useful for recording a UI-interaction sequence once and replaying it programmatically.

### Input Broadcasting Caveat

Some input-emulation patterns (broadcasting one keystroke to multiple sessions for multi-boxing) may be restricted by host-game terms of service. Verify policy before designing scripts around input broadcasting.

---

## Web Requests

Inner Space provides a [`webrequest`](https://www.lavishsoft.com/wiki/index.php/ISKernel:webrequest_(Object_Type)) object type for HTTP and HTTPS requests. This is an Inner-Space-Kernel-side feature, not LavishScript-core.

> **Choose the right API.** This section covers the imperative `webrequest` object type -- you create the variable, configure it, call `Begin`, and poll `${WR.State}` (or `wait` on a state condition). LavishMachine also exposes a declarative **`webrequest` task type** with an event-style callback when the request finishes; that variant is documented in [14_LavishMachine_Guide.md](14_LavishMachine_Guide.md). Use the imperative form for simple script-driven fetches; use the LMAC task form when you are already structured around tasks.

### What is a Web Request?

A web request fetches data from URLs over HTTP/HTTPS.

**Common uses:**

- Fetch JSON from APIs.
- Download web pages.
- Submit form data (POST).
- Check online resources.

### Creating a Web Request

```lavishscript
variable webrequest WR
```

### Setting the URL

```lavishscript
WR:SetURL["https://example.com/api/data"]
```

### Interpreting Results

Tell the engine how to interpret the response:

```lavishscript
WR:InterpretAs[json]     ; Parse as JSON
WR:InterpretAs[string]   ; Return as text
WR:InterpretAs[binary]   ; Return as binary data
```

### Beginning the Request

```lavishscript
WR:Begin
```

`Begin` is asynchronous -- the request runs in the background.

### Checking Request State

```lavishscript
echo "State: ${WR.State}"
```

Possible states:

- `Idle` -- not started.
- `Queued` -- waiting to start.
- `Working` -- in progress.
- `Completed` -- finished successfully.
- `Aborted` -- failed or cancelled.

### Accessing the Result

Once completed:

```lavishscript
if ${WR.State.Equal[Completed]}
{
    echo "Result: ${WR.Result}"
}
```

The `Result` shape depends on `InterpretAs`:

- `json` -- a `jsonobject` with response codes and data.
- `string` -- a string.
- `binary` -- a `binary` blob (also exposed via `WR.Binary`).

### Web Request With Timeout

The cleanest pattern uses `wait` with a condition:

```lavishscript
WR:SetURL["https://example.com/data"]
WR:InterpretAs[string]
WR:Begin

; Wait up to 10 seconds for completion
wait 100 ${WR.State.Equal[Completed]}

if ${WR.State.Equal[Completed]}
{
    echo "Success: ${WR.Result}"
}
else
{
    echo "Request timed out or failed"
}
```

### Polling Pattern

If you want to react to every state change instead of just completion:

```lavishscript
variable webrequest WR

function main()
{
    variable uint lastState

    WR:SetURL["https://example.com/api/data.json"]
    WR:InterpretAs[json]
    WR:Begin

    while TRUE
    {
        if ${WR.State.Value}!=${lastState}
        {
            lastState:Set[${WR.State.Value}]
            echo "State: ${WR.State}"

            if ${WR.Result(exists)}
                echo "Result: ${WR.Result}"

            if ${WR.State.Equal[Completed]}
                break
        }

        waitframe
    }
}
```

### POST Requests

Set POST data before calling `Begin`. The full method/member surface (including `FromJSON` initialization, `POST` jsonarray, file-output mode, abort/reset semantics) is in [01b §3.20 Inner Space Misc](01b_LavishScript_Reference.md#320-inner-space-misc) under `webrequest`.

### Best Practices

1. Always check state before accessing `Result`.
2. Use `InterpretAs` to specify the expected format.
3. Handle timeouts -- never wait forever.
4. Check for `Aborted` -- not just `Completed`.
5. Escape URLs that contain query parameters or user-supplied values.

### Reusable Web Request Helper

```lavishscript
function FetchJSON(string url)
{
    variable webrequest WR
    WR:SetURL["${url~}"]
    WR:InterpretAs[json]
    WR:Begin

    wait 100 ${WR.State.Equal[Completed]}

    if ${WR.State.Equal[Completed]}
    {
        return "${WR.Result}"
    }

    return ""
}
```

For more advanced async patterns, see the LavishMachine Guide's `webrequest` task type.

---

## Audio System

Inner Space provides an audio system for scripts to play sounds and music. This is an Inner-Space-Kernel-side feature, NOT LavishScript-core. The relevant object types are [`audio`](https://www.lavishsoft.com/wiki/index.php/ISKernel:audio_(Object_Type)), [`audiovoice`](https://www.lavishsoft.com/wiki/index.php/ISKernel:audiovoice_(Object_Type)), and [`audiostream`](https://www.lavishsoft.com/wiki/index.php/ISKernel:audiostream_(Object_Type)), accessed via the [`Audio` Top-Level Object](https://www.lavishsoft.com/wiki/index.php/ISKernel:Audio_(Top-Level_Object)).

### Voices and Streams

Two key concepts:

- **Voice** (object type: `audiovoice`) -- a channel that plays sound. Each voice plays one sound at a time, but multiple voices can play simultaneously.
- **Stream** (object type: `audiostream`) -- an audio file. Many formats supported (MP3, WAV, OGG, etc.).

### Creating a Voice

```lavishscript
Audio:AddVoice[music]
```

Use distinct voices for distinct purposes (background music, sound effects, alerts) so they don't interrupt each other.

### Adding a Stream

```lavishscript
Audio:AddStream[stream_name,"path/to/file.mp3"]
```

Path rules:

- Relative to script's current directory unless absolute.
- Use forward slashes `/` or escaped backslashes `\\`.
- Can use `../` to go up directories.

### Playing Audio

`PlayStream` plays immediately:

```lavishscript
function main()
{
    Audio:AddVoice[music]
    Audio:AddStream[tune,"../Assets/Audio/song.mp3"]

    Audio.Voice[music]:PlayStream[tune]

    wait 50  ; 5 seconds

    Audio.Voice[music]:Stop:ClearQueue
}
```

### Queueing Audio

`EnqueueStream` queues a stream behind the current sound:

```lavishscript
Audio.Voice[music]:PlayStream[song1]
Audio.Voice[music]:EnqueueStream[song2]   ; plays after song1 finishes
```

`PlayStream` interrupts and plays now. `EnqueueStream` waits for the current sound.

### Setting Volume

Per-channel volume (stereo = 2 channels):

```lavishscript
; Same volume both channels
Audio.Voice[music]:SetVolume[1.0]

; Different volumes (left, right)
Audio.Voice[music]:SetVolume[1.0,0.2]
```

Volume range: `0.0` is silence, `1.0` is normal/full, `>1.0` is amplified.

### Stopping and Clearing

```lavishscript
Audio.Voice[music]:Stop          ; stop current sound (queue intact)
Audio.Voice[music]:ClearQueue    ; clear queued sounds
Audio.Voice[music]:Stop:ClearQueue  ; common flush pattern
```

### Audio Alerts Pattern

```lavishscript
objectdef AlertSystem
{
    method Initialize()
    {
        Audio:AddVoice[alerts]
        Audio:AddStream[error_sound,"sounds/error.wav"]
        Audio:AddStream[success_sound,"sounds/success.wav"]
    }

    method PlayError()
    {
        Audio.Voice[alerts]:PlayStream[error_sound]
    }

    method PlaySuccess()
    {
        Audio.Voice[alerts]:PlayStream[success_sound]
    }

    method Shutdown()
    {
        Audio.Voice[alerts]:Stop:ClearQueue
        Audio:RemoveVoice[alerts]
    }
}

variable(global) AlertSystem Alerts

function main()
{
    Alerts:Initialize

    if ${SomeCondition}
        Alerts:PlaySuccess
    else
        Alerts:PlayError

    Alerts:Shutdown
}
```

### Audio Best Practices

1. Always create voices before using them.
2. Use separate voices for distinct purposes (music, effects, alerts).
3. Clean up on shutdown -- remove voices when done.
4. Verify file paths -- audio files must exist.
5. Use moderate volumes; clipping/distortion happens above `1.0`.

### Advanced Audio

For time-based volume changes (smooth fades) and async audio control, see [14_LavishMachine_Guide.md](14_LavishMachine_Guide.md), which documents the `audio.playstream` and `audio.setvolume` task types with duration support.

---

## Best Practices Summary

### 1. Variable and Parameter Naming

Use descriptive names:

```lavishscript
; GOOD
variable int CurrentHealth
variable string TargetName

; BAD
variable int ph
variable string tn
```

### 2. String Escaping

Always quote and escape strings that will be re-parsed (the [Tilde Escape rule](#the-tilde-escape----required-for-re-parsed-output)):

```lavishscript
; CORRECT
call MyFunction "${StringVar~}"

; BAD - breaks on quotes / special chars in StringVar
call MyFunction ${StringVar}
```

### 3. Code Block Braces

Always on separate lines for control-flow and definition blocks:

```lavishscript
; CORRECT
function main()
{
    echo "Hello"
}

; INCORRECT
function main() {
    echo "Hello"
}
```

(JSON literals inside data sequences are exempt -- see [Your First Script](#your-first-script).)

### 4. Comments

Use comments to explain why, not what:

```lavishscript
; GOOD - explains reasoning
; Wait for data to load completely before proceeding
wait 10 ${MyObject.IsLoaded}

; BAD - states the obvious
; Wait for data
wait 10 ${MyObject.IsLoaded}
```

### 5. NULL Checks

Always check object existence:

```lavishscript
; GOOD
if ${MyObject(exists)}
    echo "${MyObject.Name}"

; BAD - may error if object is NULL
echo "${MyObject.Name}"
```

### 6. Loop Safety

Always use `waitframe` (or `wait`) in long-running or infinite loops:

```lavishscript
; GOOD
while TRUE
{
    ; Work here
    waitframe
}

; BAD - freezes the host
while TRUE
{
    ; Work here (no waitframe!)
}
```

### 7. Function Organization

One function, one purpose:

```lavishscript
; GOOD - focused function
function GetCurrentValue()
{
    if !${SomeObject(exists)}
        return 0
    return ${SomeObject.Value}
}

; BAD - does too many things
function DoEverything()
{
    ; Checks state, performs action, cleans up, etc.
}
```

### 8. Error Handling

Check conditions before acting:

```lavishscript
; GOOD
if ${MyObject(exists)} && ${MyObject.Distance}<10
{
    MyObject:Activate
}

; BAD - may fail if object is NULL
MyObject:Activate
```

### 9. Variable Scope

Use the narrowest scope that works -- function-local first, then `script`-scope, then `global` only when needed:

```lavishscript
function main()
{
    ; Function-local (preferred)
    variable int Count

    ; Script-scope (when persistence across function calls is needed)
    variable(script) int TotalProcessed

    ; Global (use sparingly)
    variable(global) bool SystemInitialized
}
```

### 10. Initialization

Initialize objects properly using the `Initialize` method:

```lavishscript
objectdef MyObject
{
    variable string Name

    method Initialize(string InitName)
    {
        ; Properly initialize with escaping
        This:SetName["${InitName~}"]
    }

    method SetName(string NewName)
    {
        Name:Set["${NewName~}"]
    }
}
```

### 11. Avoid Shadowing Built-in TLOs

Don't name custom globals after built-in TLOs (`Script`, `Math`, `Time`, `System`, `Event`, `Audio`, `Display`, `Input`, `Mouse`, `Keyboard`, `InnerSpace`, etc.). A global named the same as a built-in TLO shadows the built-in for the rest of the script. See [Top-Level Objects](#top-level-objects) for the full list to avoid.

---

## Type Inspection and Debugging

When working with LavishScript, you'll often need to inspect what members and methods an object has. The platform exposes five "information" commands for this. All five are LavishScript-core, documented under the `Command:` namespace and listed in [01b §2.1 Information](01b_LavishScript_Reference.md#21-lavishscript-core-commands).

### The Five Information Commands

| Command | Use |
|---|---|
| [`Commands`](https://www.lavishsoft.com/wiki/index.php/Command:Commands) | List every registered command in the current session. |
| [`LSType`](https://www.lavishsoft.com/wiki/index.php/Command:LSType) | Inspect a registered object type, listing its members and methods. |
| [`LSVersion`](https://www.lavishsoft.com/wiki/index.php/Command:LSVersion) | Print the LavishScript engine version. |
| [`Scripts`](https://www.lavishsoft.com/wiki/index.php/Command:Scripts) | List currently running scripts. |
| [`TopLevelObject`](https://www.lavishsoft.com/wiki/index.php/Command:TopLevelObject) | List, define, or remove Top-Level Objects. With no argument, enumerates every TLO in the session. |

### Get the Type of Any Object

The `(type)` cast yields the object's type name as a string:

```lavishscript
echo ${SomeObject(type)}
```

### Inspect a Type's Surface

```
lstype <typename>
```

Run with no `<typename>` to list every registered type in the session:

```
lstype
```

`LSType` output shows:

- **Members** -- properties accessible with `.` (dot) notation.
- **Methods** -- actions accessible with `:` (colon) notation.
- **Static Members** -- type-level members accessed on the type name.
- **Static Methods** -- type-level methods called on the type name.

### Common Inspection Patterns

**Unknown object -- find its type:**

```lavishscript
echo ${UnknownObject(type)}
lstype <result>
```

**Check whether a member exists:**

```lavishscript
if ${Object.SomeMember(exists)}
    echo "${Object.SomeMember}"
```

**List every TLO registered in this session:**

```
TopLevelObject
```

The `TopLevelObject` output is the canonical answer to "is `${MyTLO}` registered, and what type does it return?". Useful for confirming an extension's TLOs are loaded.

### When to Use Type Inspection

- **Learning the API** -- discover what's available on an object.
- **Debugging** -- verify object types and available members.
- **Documentation** -- understand undocumented or new types.
- **Development** -- find the right method or member for your task.

---

## Additional Resources

- [LavishScript main page (Lavish Software wiki)](https://www.lavishsoft.com/wiki/index.php/LavishScript)
- [LavishScript Mathematical Formulae](https://www.lavishsoft.com/wiki/index.php/LavishScript:Mathematical_Formulae)
- [Inner Space main page](https://www.lavishsoft.com/wiki/index.php/Inner_Space)
- [Lavish Software wiki main page](https://www.lavishsoft.com/wiki/index.php/Main_Page)
- [01b_LavishScript_Reference.md](01b_LavishScript_Reference.md) -- exhaustive command/datatype/TLO inventory for this knowledge base, with one canonical entry per feature.

<!-- CLAUDE_SKIP_START -->
LERN is the official Lavish Software example/tutorial repository on GitHub:

- [LERN repository (general)](https://github.com/LavishSoftware/LERN/tree/master)
- [LERN LavishScript tutorials (19 lessons)](https://github.com/LavishSoftware/LERN/tree/master/LS)
<!-- CLAUDE_SKIP_END -->
