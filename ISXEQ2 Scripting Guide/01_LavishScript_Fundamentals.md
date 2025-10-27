# LavishScript Fundamentals

**Purpose:** Complete introduction to LavishScript programming for ISXEQ2 scripters
**Audience:** Beginners with little to no LavishScript experience
**Prerequisite:** None - start here if new to LavishScript

---

## Table of Contents

1. [Introduction](#introduction)
2. [Your First Script - "Hello World"](#your-first-script---hello-world)
3. [Variables and Data Types](#variables-and-data-types)
4. [Data Sequences](#data-sequences)
5. [Parameters](#parameters)
6. [Text Escaping](#text-escaping)
7. [Functions](#functions)
8. [Return Values](#return-values)
9. [Object-Oriented Programming](#object-oriented-programming)
10. [Members](#members)
11. [Methods](#methods)
12. [Initialize and Shutdown](#initialize-and-shutdown)
13. [Inheritance and Type Casting](#inheritance-and-type-casting)
14. [Atoms and Atomicity](#atoms-and-atomicity)
15. [Wait Commands](#wait-commands)
16. [Conditional Branching](#conditional-branching)
17. [Loops](#loops)
18. [Switch Statements](#switch-statements)
19. [Collections and Lists (index)](#collections-and-lists-index)
20. [Web Requests](#web-requests)
21. [Audio System](#audio-system)
22. [Best Practices Summary](#best-practices-summary)
23. [Next Steps](#next-steps)

---

## Introduction

**LavishScript** is the scripting language used by InnerSpace and all InnerSpace extensions, including ISXEQ2. Before diving into ISXEQ2-specific scripting, it's essential to understand the fundamentals of LavishScript itself.

### What is LavishScript?

LavishScript is a **custom scripting language** designed specifically for game automation and extension. It provides:

- **Object-oriented programming** with inheritance
- **Strong typing** with built-in and custom types
- **Atomic execution** for performance-critical code
- **Direct game memory access** through extensions like ISXEQ2
- **Event-driven programming** for reactive scripts

### Why Learn LavishScript First?

ISXEQ2 scripts are written in LavishScript. Understanding LavishScript fundamentals will help you:

- Understand ISXEQ2 API documentation
- Write correct and efficient scripts
- Debug errors more effectively
- Create custom objects and patterns
- Build complex automation systems

---

## Your First Script - "Hello World"

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
- Single-line comments begin with `;` (semicolon)
- Must appear at the start of a line (after any whitespace)

```lavishscript
; This is a valid comment
    ; This is also a valid comment (indented)
```

**2. The main Function**
- Entry point for script execution
- Defined with `function main()`
- Code block enclosed in `{` and `}`

**3. Code Block Braces**
- Opening `{` and closing `}` **must each be on their own line**
- This is a strict requirement in LavishScript

```lavishscript
; CORRECT
function main()
{
    echo "Hello"
}

; INCORRECT - Will cause errors
function main() {
    echo "Hello"
}
```

**4. Commands**
- `echo` outputs text to the console
- Commands are executed line by line

### Running Your First Script

1. Save the file as `hello.iss` in your `Scripts` directory
2. In the InnerSpace console, type: `run hello`
3. You should see: `Hello World!`

---

## Variables and Data Types

### Declaring Variables

Variables store data and are declared with the `variable` keyword:

```lavishscript
variable string TextToDisplay="Hello World!"
```

**Syntax:**
```
variable <type> <name> = <optional_initial_value>
```

**Examples:**
```lavishscript
variable string Name="Alice"
variable int Counter=0
variable string Message    ; No initial value
```

### Common Data Types

| Type | Description | Example Values |
|------|-------------|----------------|
| `string` | Text data | `"Hello"`, `"Bob"` |
| `int` | 32-bit signed integer | `-1`, `0`, `100`, `2147483647` |
| `uint` | 32-bit unsigned integer | `0`, `100`, `4294967296` |
| `float` | Floating-point number | `1.5`, `-3.14`, `0.001` |
| `bool` | Boolean (TRUE/FALSE) | `TRUE`, `FALSE` |

### Integer Types

**Signed Integer (`int`):**
- Range: -2,147,483,648 to 2,147,483,647
- Can be positive or negative

**Unsigned Integer (`uint`):**
- Range: 0 to 4,294,967,296
- Cannot be negative (more room for positive values)

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

## Data Sequences

### What is a Data Sequence?

A **Data Sequence** retrieves a value and **emplaces** it in a command before execution.

**Syntax:** `${ }`

```lavishscript
variable string Name="Alice"
echo "Hello ${Name}!"
```

**Output:** `Hello Alice!`

### How Data Sequences Work

1. LavishScript encounters `${Name}`
2. Looks up the value of `Name` (which is `"Alice"`)
3. Replaces `${Name}` with `Alice` in the command
4. Executes: `echo "Hello Alice!"`

### Multiple Data Sequences

You can use multiple Data Sequences in a single command:

```lavishscript
variable string First="John"
variable string Last="Smith"

echo "${First} ${Last} ${First} ${Last}"
```

**Output:** `John Smith John Smith`

### Accessing Nested Data

Data Sequences can access nested properties using `.` (dot) notation:

```lavishscript
variable string Name="Alice"
echo "Name length: ${Name.Length}"
```

**Output:** `Name length: 5`

Here, `Length` is a **member** of the `string` type (more on members later).

### Data Sequence Best Practice

When using Data Sequences that may contain special characters (like quotes), use the tilde `~` for escaping:

```lavishscript
variable string Text="He said \"Hello\""
echo "${Text~}"
```

The `~` ensures the text is properly escaped (more on escaping later).

---

## Parameters

### Function Parameters

Functions can accept parameters (input values):

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

It will use the defaults. But you can override them:
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

**Usage:**
```
run myscript "John" "Smith" 30
```

### Command Parameter Splitting

**Important:** Parameters are split by **spaces**.

```lavishscript
echo Multiple Parameters
; This passes TWO parameters to echo: "Multiple" and "Parameters"

echo "One Parameter"
; This passes ONE parameter to echo: "One Parameter"
```

**Rule:** If a parameter contains spaces, it **must be quoted** with `"`.

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
- `"` (quote) - Splits parameters
- `\` (backslash) - Escape character
- `~` (tilde) - Escape in Data Sequences

### Escaping Quotes in Commands

Use `\` (backslash) to escape quotes:

```lavishscript
echo "He said \"Hello World!\""
```

**Output:** `He said "Hello World!"`

### Escaping in Data Sequences

Use `~` (tilde) to escape Data Sequences:

```lavishscript
variable string Text="He said \"Hello\""
echo "${Text~}"
```

**Why use `~`?**

Without `~`, if the variable contains quotes or other special characters, they might interfere with parameter parsing:

```lavishscript
; BAD - May break if Text contains quotes
call SomeFunction ${Text}

; GOOD - Safely escapes any special characters
call SomeFunction "${Text~}"
```

### Best Practice for Escaping

**Always quote and escape `string` parameters:**

```lavishscript
variable string Message="Hello \"World\""

; BEST PRACTICE
call MyFunction "${Message~}"

; RISKY - May fail if Message has spaces or quotes
call MyFunction ${Message}
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

Functions are code blocks that can be called (executed) by name:

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

Functions can accept parameters just like `main`:

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

Just like with commands, **quote and escape** string parameters:

```lavishscript
function DisplayMessage(string Message)
{
    echo "Message: ${Message}"
}

function main()
{
    variable string Text="Important \"Info\""

    ; BEST PRACTICE - Quote and escape
    call DisplayMessage "${Text~}"
}
```

---

## Return Values

### The return Command

The `return` command has two purposes:

1. **Immediately exits** the current function
2. **Provides a value** to the caller

```lavishscript
function GetAnswer()
{
    return 42
}
```

### Return Without a Value

You can use `return` without a value to exit early:

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

After calling a function, access the return value via `${Return}`:

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

---

## Object-Oriented Programming

### What is Object-Oriented Programming?

Object-Oriented Programming (OOP) allows you to **tie available functionality to a type of data**.

In LavishScript, this is done with `objectdef` (object definition), equivalent to a `class` in many other languages.

### Defining an Object

```lavishscript
objectdef fruit
{
    variable string Name="Apple"
    variable string Color="Red"
}
```

**Key points:**
- `objectdef` keyword followed by the type name
- Code block with `{` and `}` on separate lines
- Can contain variables, members, and methods

### Creating Object Variables

Once defined, you can create variables of your custom type:

```lavishscript
variable fruit MyFruit
```

This creates a `fruit` object called `MyFruit` with:
- `Name` = "Apple"
- `Color` = "Red"

### Accessing Object Variables

Use `.` (dot) notation in Data Sequences:

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

Think of `${MyFruit.Name}` as an **address**:

- Start at `MyFruit`
- Navigate to `Name`
- Retrieve the value

Just like a postal address leads to a house, a Data Sequence leads to data.

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

A **member** is a specialized function that **provides a value** when accessed in a Data Sequence.

Unlike a variable (which stores a value), a member **calculates and returns** a value.

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

`ToText` is special - it's called when you access an object's value directly:

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
|---------|----------|--------|
| Stores value | Yes | No |
| Calculates value | No | Yes |
| Access syntax | `${Object.Variable}` | `${Object.Member}` |
| Can have parameters | No | Yes |

### Built-in Type Members

Built-in types like `string` have members too:

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

### Member Parameters

Members can accept parameters (using index syntax `[ ]`):

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

**Note:** Parameters use `[` `]` brackets and are separated by `,` commas.

---

## Methods

### What is a Method?

A **method** is a specialized function that **performs an action** using an object.

Where a **member** retrieves a value, a **method** does something (and may or may not return the same object).

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
|------|--------|---------|---------|
| **Member** | `.` (dot) | Retrieve value | `${Object.Member}` |
| **Method** | `:` (colon) | Perform action | `Object:Method` |

### Methods as Commands

Methods can be used **as commands** (without `${ }`):

```lavishscript
function main()
{
    variable fruit MyFruit

    ; As a command (recommended for actions)
    MyFruit:SetName["Banana"]

    ; In a Data Sequence (unnecessary)
    echo "${MyFruit:SetName["Banana"]}"
}
```

**Best practice:** Use methods as commands when you're performing an action, not retrieving a value.

### Method Chaining

Methods can be chained together:

```lavishscript
function main()
{
    variable fruit MyFruit

    MyFruit:SetName["Banana"]:SetColor["Yellow"]

    echo "Fruit: ${MyFruit.Name} ${MyFruit.Color}"
}
```

**Output:** `Fruit: Banana Yellow`

### Method Return Values

Methods typically return the same object (for chaining). To indicate failure or destruction, return `FALSE`:

```lavishscript
method Destroy()
{
    ; Cleanup code here
    return FALSE
}
```

When a method returns `FALSE`, the Data Sequence results in `NULL` (no object).

### String Set Method

The built-in `string` type has a `Set` method to change its value:

```lavishscript
variable string Name="Alice"

Name:Set["Bob"]

echo "${Name}"  ; Output: Bob
```

### Parameter Escaping in Methods

When passing string parameters to methods, **quote and escape**:

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

    ; BEST PRACTICE
    P:SetName["${NewName~}"]
}
```

---

## Initialize and Shutdown

### The Initialize Method

`Initialize` is a special method called when an object is **created** (instantiated).

**Purpose:**
1. Event handler for object creation
2. Accepts initial values
3. Sets up default state

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
    variable fruit Fruit1  ; Uses default "Apple"
    variable fruit Fruit2="Banana"  ; Passes "Banana" to Initialize

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

`Shutdown` is called when an object is **destroyed**.

**Purpose:**
1. Event handler for object destruction
2. Cleanup resources (destroy GUI, close files, etc.)
3. No parameters accepted

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

**`This`** is a special reference to the **current object** being operated on.

**Why use `This`?**

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

### Accessing Members/Methods with This

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

Inside a member, you **must** use `This` to access variables/members/methods of the same object. Variables without `This` refer to local or global variables.

---

## Inheritance and Type Casting

### Object Inheritance

Inheritance allows one object type to **inherit** properties and behavior from another.

**Example:** Apples, Bananas, and Cherries are all **fruit**.

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
- All variables from `fruit` (Name, Color)
- All methods from `fruit` (SetName)
- Plus any new variables/methods defined in `apple` (SetColor)

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

Child objects can **override** (replace) members/methods from parent objects:

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

### Type Casting

**Type casting** allows you to access a parent type's members/methods even when they're overridden.

**Syntax:** `Object(type)`

```lavishscript
objectdef banana inherits fruit
{
    method Initialize()
    {
        ; Call fruit's Initialize method
        This(fruit):Initialize["Banana"]

        ; Then do banana-specific stuff
        This:SetColor["Yellow"]
    }
}
```

**Why is this useful?**

The `banana` object overrides `Initialize`, but we still want to call `fruit`'s `Initialize` first. Type casting lets us do that.

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
        ; Call the parent's Initialize
        This(fruit):Initialize["Apple"]

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

1. **Call parent Initialize** when overriding:
```lavishscript
method Initialize()
{
    This(parent):Initialize
    ; Child-specific initialization
}
```

2. **Use inheritance for "is-a" relationships**:
   - An Apple **is a** Fruit ✓
   - An Apple **has a** Color (use a variable instead) ✓

3. **Override carefully** - Make sure you understand what the parent method does

---

## Atoms and Atomicity

### What is Atomicity?

"Atomic" means **unbroken, as a whole**. In programming, atomic functions **must complete immediately** without waiting.

### Atomic vs Non-Atomic

| Function Type | Can Wait? | Can Use waitframe? | Blocks Game? |
|---------------|-----------|-------------------|--------------|
| `function` | Yes | Yes | No |
| `atom` | No | No | Yes |
| `member` | No | No | Yes |
| `method` | No | No | Yes |

**Atomic functions** (`atom`, `member`, `method`) **block** the host game from advancing until they complete.

### Why Atomicity Matters

**Members and methods must be atomic** because they're used in Data Sequences, which must resolve immediately:

```lavishscript
echo "${Object.Member}"
```

If `Member` could wait for frames, the `echo` command would be stuck waiting, freezing the entire script.

### Defining an Atom

Use the `atom` keyword:

```lavishscript
atom MyCommand()
{
    echo "Command executed!"
}
```

### Using Atoms as Commands

Atoms can be executed **as commands**:

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
|---------|----------|------|
| Can use waitframe | Yes | No |
| Can use wait | Yes | No |
| Can run for minutes | Yes | No |
| Can be used as command | No | Yes |
| Blocks game | No | Yes |

### The atexit Atom

`atexit` is a special atom called **when a script is ending**.

**Purpose:** Cleanup before script shutdown (destroy GUI, save data, etc.)

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

### Common atexit Usage

```lavishscript
objectdef MyScript
{
    variable bool IsRunning=TRUE

    method Initialize()
    {
        echo "Script started"
    }

    method Shutdown()
    {
        echo "Shutting down..."
        IsRunning:Set[FALSE]
    }
}

variable(global) MyScript Script

atom atexit()
{
    Script:Shutdown
}

function main()
{
    Script:Initialize

    while ${Script.IsRunning}
    {
        ; Main script loop
        waitframe
    }
}
```

### Atomic Restrictions

**Cannot use in atomic code:**
- `waitframe`
- `wait`
- Anything that depends on frame updates

**Attempting to use these will cause a script error:**

```lavishscript
; BAD - This will error!
atom BadAtom()
{
    waitframe  ; ERROR: waitframe not allowed in atomic code
}
```

---

## Wait Commands

### Script Running Time

Access the current script's running time via `${Script.RunningTime}`:

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

`waitframe` waits until the game prepares and renders the next frame.

```lavishscript
function main()
{
    echo "Before frame"
    waitframe
    echo "After frame"
}
```

**When to use:**
- In loops (to avoid max CPU usage)
- When waiting for game state changes
- To yield control back to the game

### The wait Command

`wait` waits for **at least** a specified amount of time.

**Two syntax options:**

1. **Deciseconds (tenths of a second):**
```lavishscript
wait 10  ; Wait 1 second (10 deciseconds)
wait 50  ; Wait 5 seconds
```

2. **Seconds (with -s parameter):**
```lavishscript
wait -s 1.0    ; Wait 1 second
wait -s 2.5    ; Wait 2.5 seconds
wait -s 0.001  ; Wait 0.001 seconds (1 millisecond)
```

### Wait Precision

Wait precision depends on **framerate (FPS)**:

- At **60 FPS**, each frame takes ~16.7ms
- At **10 FPS**, each frame takes ~100ms

```lavishscript
wait -s 0.001  ; Request 1ms wait
```

At **10 FPS**, this actually waits ~100ms (one frame).

**Rule:** Waits are "quantized" to framerate intervals.

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

### Conditional Wait

Wait for a condition to become true:

```lavishscript
wait 100 ${Target(exists)}
```

**Syntax:** `wait <max_time> <condition>`

This waits up to 10 seconds (100 deciseconds) for `${Target(exists)}` to become TRUE.

### Infinite Loops with waitframe

**Always use `waitframe` in infinite loops:**

```lavishscript
function main()
{
    variable bool Running=TRUE

    while ${Running}
    {
        ; Do work here

        waitframe  ; IMPORTANT - Yields CPU
    }
}
```

**Why?** Without `waitframe`, the loop would consume maximum CPU and freeze the game.

### Wait Restrictions

**Cannot use in atomic code:**

```lavishscript
; BAD - Will cause error
member BadMember()
{
    waitframe  ; ERROR!
    return "value"
}

; BAD - Will cause error
atom BadAtom()
{
    wait 10  ; ERROR!
}
```

Only use `wait` and `waitframe` in regular `function`s.

---

## Conditional Branching

### The if Statement

Execute code **only if a condition is met**:

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

- **0**, **NULL**, **FALSE** → Condition NOT met (false)
- **Non-zero**, **TRUE** → Condition met (true)

### Comparison Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `<` | Less than | `${Value}<10` |
| `>` | Greater than | `${Value}>100` |
| `<=` | Less than or equal | `${Value}<=10` |
| `>=` | Greater than or equal | `${Value}>=100` |
| `==` | Equal to | `${Value}==5` |
| `!=` | Not equal to | `${Value}!=0` |

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
1. Check first `if` - if true, execute and skip rest
2. If false, check first `elseif`
3. Continue down the chain until a match is found

### The else Statement

Execute code when **no conditions match**:

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

Combine conditions with `&&` (AND) and `||` (OR):

```lavishscript
; AND - Both must be true
if ${Health}<50 && ${Power}<20
    echo "Low on both!"

; OR - Either can be true
if ${Health}<20 || ${Power}<10
    echo "Emergency!"
```

### Checking Existence

Check if an object exists with `(exists)`:

```lavishscript
if ${Target(exists)}
    echo "Target exists"
else
    echo "No target"
```

---

## Loops

### The while Loop

Repeat code **while a condition is met**:

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
        waitframe  ; IMPORTANT - Prevents CPU overload
    }
}
```

**Remember:** Always use `waitframe` in infinite loops!

### The do-while Loop

Execute code **at least once**, then check condition:

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
- `while` checks condition **before** executing
- `do-while` checks condition **after** executing (guarantees at least one execution)

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

Skip the rest of the loop and advance to the next iteration:

```lavishscript
function main()
{
    variable int Count=0

    while ${Count}<10
    {
        Count:Inc

        ; Skip odd numbers
        if ${Math.Calc[${Count}%2]}
            continue

        echo "${Count}"
    }
}
```

**Output:** `2 4 6 8 10` (only even numbers)

### The for Loop

Compact loop syntax combining initialization, condition, and increment:

```lavishscript
for (start ; condition ; advance)
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

Set an integer value (accepts math):

```lavishscript
Count:Set[1]        ; Set to 1
Count:Set[5+5]      ; Set to 10
Count:Set[${Count}*2]  ; Double the current value
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

**Output:**
```
I=1 J=1
I=1 J=2
I=1 J=3
I=2 J=1
...
```

---

## Switch Statements

### The switch Statement

Selectively execute one of several code branches based on a value:

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

1. Evaluate the `switch` value
2. **Case-insensitive string compare** with each `case`
3. If match found, execute code from that point **downward**
4. `break` exits the switch block
5. If no match, execute `default` (if provided)

### Fall-through Behavior

**Without `break`, execution continues to the next case:**

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

This matches multiple values to the same code.

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

Use when branch values **cannot be hard-coded** (contain Data Sequences):

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
1. Check all `case` branches first
2. If no `case` matches, check `variablecase` branches in order
3. Resolve Data Sequences in `variablecase` only when needed
4. Execute first matching `variablecase`

### switch Best Practices

1. **Always use `break`** (unless intentional fall-through)
2. **Provide `default`** for unexpected values
3. **Use `case` for constants**, `variablecase` for variables
4. **Case values are verbatim text** (don't add `:` like in C/C++)

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

## Collections and Lists (index)

### What is an Index?

An **index** is a dynamic array (list) that can grow and shrink as needed. Unlike fixed arrays, indices automatically manage their size.

**Key features:**
- **Dynamic size** - Grows automatically as you add items
- **Type-safe** - Specify element type like `index:int` or `index:string`
- **1-based indexing** - First element is at position 1 (not 0!)
- **JSON serialization** - Easy conversion to/from JSON arrays

### Creating an Index

Use `variable index:type Name` syntax:

```lavishscript
variable index:int Numbers
variable index:string Names
variable index:person People
```

**Important:** You must specify the element type after the colon.

### Adding Items

Use the `Insert` method to add items:

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

**Remember:** Indices start at 1, not 0!

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

**Important:** Removing creates a gap - the position becomes NULL.

### Collapsing Gaps

Use `Collapse` to remove gaps by shifting items:

```lavishscript
variable index:int Numbers

Numbers:Insert[17]
Numbers:Insert[42]
Numbers:Insert[35]

Numbers:Remove[2]    ; Remove 42 - creates gap
Numbers:Collapse     ; Shift items to fill gap

echo "${Numbers[2]}"  ; Output: 35 (was at position 3)
```

**After Collapse:** `[17, 35]` (no gaps)

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

**Output:**
```json
[{"first_name":"John","last_name":"Doe"},{"first_name":"Jane","last_name":"Doe"},{"first_name":"John","last_name":"Public"}]
```

### JSON Serialization

**AsJSON - Convert to JSON array:**

```lavishscript
variable index:int Numbers

Numbers:Insert[17]
Numbers:Insert[42]
Numbers:Insert[35]

echo "${Numbers.AsJSON}"  ; Output: [17,42,35]
```

**FromJSON - Load from JSON array:**

```lavishscript
objectdef person
{
    variable string FirstName
    variable string LastName

    ; Initialize accepts jsonvalue
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

### Common Index Operations

| Operation | Method | Example |
|-----------|--------|---------|
| Add item | `Insert[value]` | `List:Insert[42]` |
| Remove item | `Remove[position]` | `List:Remove[3]` |
| Fill gaps | `Collapse` | `List:Collapse` |
| Clear all | `Clear` | `List:Clear` |
| Get count | `.Used` | `${List.Used}` |
| To JSON | `.AsJSON` | `${List.AsJSON}` |
| From JSON | `FromJSON[json]` | `List:FromJSON[...]` |

### Index Example - Complete

```lavishscript
function main()
{
    variable index:string Names

    ; Add some names
    Names:Insert["Alice"]
    Names:Insert["Bob"]
    Names:Insert["Charlie"]
    Names:Insert["Diana"]

    echo "Count: ${Names.Used}"  ; Output: 4

    ; Access by position
    echo "First: ${Names[1]}"    ; Output: Alice
    echo "Third: ${Names[3]}"    ; Output: Charlie

    ; Remove Bob (position 2)
    Names:Remove[2]

    echo "After remove: ${Names.AsJSON}"  ; Output: ["Alice",null,"Charlie","Diana"]

    ; Collapse to remove gap
    Names:Collapse

    echo "After collapse: ${Names.AsJSON}"  ; Output: ["Alice","Charlie","Diana"]
    echo "New count: ${Names.Used}"         ; Output: 3
}
```

### Index vs Collection

LavishScript also has `collection` (key-value pairs, like a dictionary/map). Use:
- **index** - When you need ordered lists
- **collection** - When you need key-value lookups

See the JSON Guide for more on collections.

---

## Web Requests

### What is a Web Request?

A **web request** allows scripts to fetch data from URLs using HTTP/HTTPS protocols.

**Common uses:**
- Fetch JSON data from APIs
- Download web pages
- Submit form data (POST)
- Check online resources

### The webrequest Type

Create with `variable webrequest Name`:

```lavishscript
variable webrequest WR
```

### Setting the URL

Use `SetURL` method:

```lavishscript
WR:SetURL["https://example.com/api/data"]
```

**Supports:**
- HTTP and HTTPS
- GET and POST requests
- Query parameters in URL

### Interpreting Results

Tell LavishScript how to interpret the response:

```lavishscript
WR:InterpretAs[json]     ; Parse as JSON
WR:InterpretAs[string]   ; Return as text
WR:InterpretAs[binary]   ; Return as binary data
```

**Default:** If not specified, result is returned as string.

### Beginning the Request

Use `Begin` to start the request:

```lavishscript
WR:Begin
```

**Note:** This is asynchronous - the request happens in the background.

### Checking Request State

Monitor the request with `.State`:

```lavishscript
echo "State: ${WR.State}"
```

**Possible states:**
- `Idle` - Not started
- `Queued` - Waiting to start
- `Working` - In progress
- `Completed` - Finished successfully
- `Aborted` - Failed or cancelled

### Accessing the Result

Once completed, access via `.Result`:

```lavishscript
if ${WR.State.Equal[Completed]}
{
    echo "Result: ${WR.Result}"
}
```

**Result type depends on `InterpretAs`:**
- `json` → Returns jsonvalue object
- `string` → Returns string
- `binary` → Returns binary data

### Complete Web Request Example

```lavishscript
variable webrequest WR

function main()
{
    variable uint lastState

    ; Set up the request
    WR:SetURL["https://raw.githubusercontent.com/LavishSoftware/LERN/master/LGUI2/1.json"]
    WR:InterpretAs[json]

    ; Start the request
    WR:Begin

    echo "Request started..."

    ; Wait for completion
    while 1
    {
        ; Check if state changed
        if ${WR.State.Value}!=${lastState}
        {
            lastState:Set[${WR.State.Value}]
            echo "State: ${WR.State}"

            ; Check for result
            if ${WR.Result(exists)}
                echo "Result: ${WR.Result}"

            ; Exit when done
            if ${WR.State.Equal[Completed]}
                break
        }

        waitframe
    }

    echo "Request complete"
}
```

### Web Request with Timeout

Wait with a timeout:

```lavishscript
WR:SetURL["https://example.com/data"]
WR:InterpretAs[string]
WR:Begin

; Wait up to 10 seconds
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

### POST Requests

Set POST data before calling Begin:

```lavishscript
WR:SetURL["https://api.example.com/submit"]
WR:SetPOSTData["key1=value1&key2=value2"]
WR:InterpretAs[json]
WR:Begin
```

### Best Practices

1. **Always check state** before accessing Result
2. **Use InterpretAs** to specify expected format
3. **Handle timeouts** - Don't wait forever
4. **Check for errors** - State could be Aborted
5. **Escape URLs** if they contain parameters

### Web Request Pattern

```lavishscript
function FetchJSON(string url)
{
    variable webrequest WR
    WR:SetURL["${url~}"]
    WR:InterpretAs[json]
    WR:Begin

    ; Wait up to 10 seconds
    wait 100 ${WR.State.Equal[Completed]}

    if ${WR.State.Equal[Completed]}
    {
        return "${WR.Result}"
    }

    return ""
}
```

**Note:** For more advanced async web requests, see the LavishMachine Guide's webrequest task type.

---

## Audio System

### What is the Audio System?

LavishScript's **audio system** allows scripts to play sounds and music through voices and streams.

**Key concepts:**
- **Voice** - A channel that plays sound (like a speaker)
- **Stream** - An audio file that can be played

### Audio Voices

A **voice** is something that makes sound. Each voice can play one sound at a time, but multiple voices can play simultaneously.

**Creating a voice:**

```lavishscript
Audio:AddVoice[music]
```

**Use cases:**
- Background music voice
- Sound effects voice
- Alert voice
- Separate voice for each character in multi-boxing

### Audio Streams

A **stream** is an audio file. Many formats are supported (MP3, WAV, OGG, etc.).

**Adding a stream:**

```lavishscript
Audio:AddStream[stream_name,"path/to/file.mp3"]
```

**Path rules:**
- Relative to script directory
- Use forward slashes `/` or escaped backslashes `\\`
- Can use `../` to go up directories

### Playing Audio

Use `PlayStream` to play immediately:

```lavishscript
function main()
{
    ; Add a voice
    Audio:AddVoice[music]

    ; Add a stream
    Audio:AddStream[tune,"../Assets/Audio/song.mp3"]

    ; Play the sound
    Audio.Voice[music]:PlayStream[tune]

    ; Wait 5 seconds
    wait 50

    ; Stop the music
    Audio.Voice[music]:Stop:ClearQueue
}
```

### Queueing Audio

Use `EnqueueStream` to queue after current sound:

```lavishscript
; Play immediately
Audio.Voice[music]:PlayStream[song1]

; Queue next (plays after song1 finishes)
Audio.Voice[music]:EnqueueStream[song2]
```

**Difference:**
- `PlayStream` - Stops current sound, plays immediately
- `EnqueueStream` - Waits for current sound to finish

### Setting Volume

Control volume per channel (stereo = 2 channels):

```lavishscript
; Same volume both channels
Audio.Voice[music]:SetVolume[1.0]

; Different volumes (left, right)
Audio.Voice[music]:SetVolume[1.0,0.2]
```

**Volume values:**
- `0.0` - Silence
- `1.0` - Normal/full volume
- `>1.0` - Amplified (louder)

**Channels:**
- Stereo: 2 channels (left, right)
- 5.1 Surround: 6 channels
- 7.1 Surround: 8 channels

### Stopping Audio

Stop current sound:

```lavishscript
Audio.Voice[music]:Stop
```

**Note:** This pauses the stream but keeps it queued.

### Clearing the Queue

Remove all queued sounds:

```lavishscript
Audio.Voice[music]:ClearQueue
```

**Common pattern (flush):**

```lavishscript
Audio.Voice[music]:Stop:ClearQueue
```

This stops current sound AND clears the queue.

### Complete Audio Example

```lavishscript
function main()
{
    ; Setup
    Audio:AddVoice[music]
    Audio:AddVoice[sfx]

    Audio:AddStream[song,"../Assets/Audio/music.mp3"]
    Audio:AddStream[beep,"../Assets/Audio/beep.wav"]

    ; Set volumes
    Audio.Voice[music]:SetVolume[0.5]    ; Quiet music
    Audio.Voice[sfx]:SetVolume[1.0]      ; Full volume effects

    ; Play music
    Audio.Voice[music]:PlayStream[song]

    ; Play sound effect (different voice, plays simultaneously)
    Audio.Voice[sfx]:PlayStream[beep]

    ; Wait 10 seconds
    wait 100

    ; Stop everything
    Audio.Voice[music]:Stop:ClearQueue
    Audio.Voice[sfx]:Stop:ClearQueue
}
```

### Audio for Alerts

Common pattern for script alerts:

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

    ; Do some work...
    if ${Success}
        Alerts:PlaySuccess
    else
        Alerts:PlayError

    Alerts:Shutdown
}
```

### Panning Example

Create stereo effects by adjusting left/right volume:

```lavishscript
function main()
{
    Audio:AddVoice[music]
    Audio:AddStream[tune,"music.mp3"]

    ; Start mostly on left
    Audio.Voice[music]:SetVolume[1.0,0.2]
    Audio.Voice[music]:PlayStream[tune]

    wait 30  ; 3 seconds

    ; Pan to right
    Audio.Voice[music]:SetVolume[0.2,1.0]

    wait 30

    ; Back to center
    Audio.Voice[music]:SetVolume[1.0,1.0]

    wait 30

    Audio.Voice[music]:Stop:ClearQueue
}
```

### Audio Best Practices

1. **Always create voices before using them**
2. **Use separate voices for different purposes** (music, effects, alerts)
3. **Clean up on shutdown** - Remove voices when done
4. **Check file paths** - Audio files must exist
5. **Use appropriate volumes** - Don't deafen users!

### Advanced Audio

For **time-based volume changes** and **async audio control**, see the **LavishMachine Guide** which covers:
- `audio.playstream` task type
- `audio.setvolume` task type with duration
- Smooth volume transitions
- Looping audio

---

## Best Practices Summary

### 1. Variable and Parameter Naming

**Use descriptive names:**
```lavishscript
; GOOD
variable int PlayerHealth
variable string TargetName

; BAD
variable int ph
variable string tn
```

### 2. String Escaping

**Always quote and escape strings:**
```lavishscript
; BEST PRACTICE
call MyFunction "${StringVar~}"

; RISKY
call MyFunction ${StringVar}
```

### 3. Code Block Braces

**Always on separate lines:**
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

### 4. Comments

**Use comments to explain why, not what:**
```lavishscript
; GOOD - Explains reasoning
; Wait for target to load completely before proceeding
wait 10 ${Target.IsLoaded}

; BAD - States the obvious
; Wait for target
wait 10 ${Target.IsLoaded}
```

### 5. NULL Checks

**Always check object existence:**
```lavishscript
; GOOD
if ${Target(exists)}
    echo "${Target.Name}"

; BAD - May error if no target
echo "${Target.Name}"
```

### 6. Loop Safety

**Always use waitframe in infinite loops:**
```lavishscript
; GOOD
while TRUE
{
    ; Work here
    waitframe
}

; BAD - Freezes game
while TRUE
{
    ; Work here (no waitframe!)
}
```

### 7. Function Organization

**One function, one purpose:**
```lavishscript
; GOOD - Focused function
function GetTargetHealth()
{
    if !${Target(exists)}
        return 0
    return ${Target.Health}
}

; BAD - Does too many things
function DoEverything()
{
    ; Checks target, casts spell, loots, etc.
}
```

### 8. Error Handling

**Check conditions before acting:**
```lavishscript
; GOOD
if ${Target(exists)} && ${Target.Distance}<10
{
    Target:DoTarget
}

; BAD - May fail if no target
Target:DoTarget
```

### 9. Variable Scope

**Use local variables when possible:**
```lavishscript
function main()
{
    ; Local variable (preferred)
    variable int Count

    ; Global variable (use sparingly)
    variable(global) int GlobalCount
}
```

### 10. Initialization

**Initialize objects properly:**
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

---

## Next Steps

Congratulations! You now understand the fundamentals of LavishScript.

### Where to Go from Here

Now that you know LavishScript basics, you're ready to learn **ISXEQ2-specific scripting**:

1. **[QUICK_START_GUIDE.md](QUICK_START_GUIDE.md)** - Your first ISXEQ2 script
   - Access character information
   - Basic combat checks
   - Inventory access

2. **[04_Core_Concepts.md](04_Core_Concepts.md)** - ISXEQ2 concepts
   - Top-Level Objects (TLOs) like `${Me}`, `${Target}`, `${Zone}`
   - ISXEQ2-specific datatypes
   - Query syntax for finding actors and items
   - Asynchronous data loading
   - Event system

3. **[06_Working_Examples.md](06_Working_Examples.md)** - Practical ISXEQ2 code
   - Character stats and abilities
   - Inventory management
   - Combat automation
   - Quest information
   - UI interaction

4. **[03_API_Reference.md](03_API_Reference.md)** - Complete ISXEQ2 API
   - Detailed reference for all ISXEQ2 datatypes
   - All members and methods
   - Commands and events

### Key Differences Between LavishScript and ISXEQ2

**LavishScript provides:**
- Language fundamentals (variables, functions, objects, loops)
- Core datatypes (string, int, float)
- Script control (wait, waitframe, call, return)

**ISXEQ2 adds:**
- Game-specific TLOs (`${Me}`, `${Target}`, `${Actor[...]}`)
- Game datatypes (char, actor, item, ability, quest)
- Game commands (target, cast, useitem)
- Game events (OnIncomingChatText, OnIncomingText, etc.)

### Practice Exercises

Before moving to ISXEQ2, try these exercises to solidify your LavishScript knowledge:

**Exercise 1: Character Manager**
```
Create an objectdef called 'character' with:
- Variables: Name (string), Level (int), Class (string)
- Initialize method accepting Name, Level, Class
- ToText member returning formatted character info
- LevelUp method to increment Level
```

**Exercise 2: Inventory Simulator**
```
Create a script with:
- Function to add items to an array/collection
- Function to remove items
- Function to list all items
- while loop in main that accepts user commands
```

**Exercise 3: Combat Simulator**
```
Create objectdefs for:
- Enemy (Health, Name)
- Attack result calculator
- Use a while loop to simulate combat
- Use if-elseif-else for different attack results
```

### Additional Resources

- **LavishScript Wiki:** http://www.lavishsoft.com/wiki/LavishScript
- **LavishScript Math Operators:** http://www.lavishsoft.com/wiki/index.php/LavishScript:Mathematical_Formulae
- **InnerSpace Documentation:** http://www.lavishsoft.com/wiki/
- **LERN Examples:** https://github.com/LavishSoftware/LERN/tree/master

---

**You're now ready to begin ISXEQ2 scripting! Proceed to [QUICK_START_GUIDE.md](QUICK_START_GUIDE.md) to get started.**

---

*Last Updated: 2025-10-21*
*Based on LERN/LS Tutorial Series: https://github.com/LavishSoftware/LERN/tree/master/LS (19 lessons)*
*Part of ISXEQ2 Scripting Guide*
