# JSON in LavishScript - Complete Guide

**System:** LavishScript JSON API
**Target:** InnerSpace Scripts (Game-Agnostic)

---

## Table of Contents

1. [Introduction to JSON](#introduction-to-json)
2. [JSON Data Types](#json-data-types)
3. [Creating JSON Values](#creating-json-values)
4. [Accessing JSON Data](#accessing-json-data)
5. [Modifying JSON Values](#modifying-json-values)
6. [Working with JSON Objects](#working-with-json-objects)
7. [Working with JSON Arrays](#working-with-json-arrays)
8. [Reading and Writing JSON Files](#reading-and-writing-json-files)
9. [Object Serialization (AsJSON)](#object-serialization-asjson)
10. [Object Deserialization (FromJSON)](#object-deserialization-fromjson)
11. [Collections and Indexes with JSON](#collections-and-indexes-with-json)
12. [JSON References (jsonvalueref)](#json-references-jsonvalueref)
13. [Best Practices](#best-practices)
14. [Complete Examples](#complete-examples)
15. [Troubleshooting](#troubleshooting)

---

## Introduction to JSON

### What is JSON?

**JSON (JavaScript Object Notation)** is a lightweight data interchange format that's easy for humans to read and write, and easy for machines to parse and generate. It's the standard format for:

- Configuration files
- Data storage
- API communication
- Inter-script data exchange
- **LavishGUI 2 UI packages** (covered in [08_LavishGUI2_UI_Guide.md](08_LavishGUI2_UI_Guide.md))

### Why Use JSON in InnerSpace Scripts?

1. ✅ **Configuration Management** - Store script settings in readable files
2. ✅ **Data Persistence** - Save and load complex data structures
3. ✅ **Inter-Script Communication** - Share data between scripts
4. ✅ **UI Creation** - LavishGUI 2 uses JSON packages
5. ✅ **API Integration** - Modern web APIs use JSON
6. ✅ **Structured Data** - Organize complex information clearly

### JSON in LavishScript

LavishScript provides native JSON support through:

- `jsonvalue` - Variable type for JSON data
- `jsonvalueref` - Reference to JSON data (efficient)
- `jsonobject` - JSON object type
- `jsonarray` - JSON array type
- Built-in methods for parsing, creating, and manipulating JSON

---

## JSON Data Types

### Type Overview

| JSON Type | Example | LavishScript Type | Description |
|-----------|---------|-------------------|-------------|
| **integer** | `1234` | `int64` | Whole numbers |
| **number** | `1.5` | `float64` | Decimal numbers |
| **string** | `"Hello"` | `string` | Text in quotes |
| **true** | `true` | `bool` | Boolean true |
| **false** | `false` | `bool` | Boolean false |
| **null** | `null` | NULL | No value |
| **array** | `[1,2,3]` | `jsonarray` | Ordered list |
| **object** | `{"key":"value"}` | `jsonobject` | Key-value pairs |

### Type Detection

Every `jsonvalue` has a `.Type` member that returns its type as a string:

```lavishscript
variable jsonvalue myValue=42
echo ${myValue.Type}  ; Output: integer

myValue:SetValue["\"Hello\""]
echo ${myValue.Type}  ; Output: string

myValue:SetValue[null]
echo ${myValue.Type}  ; Output: null
```

---

## Creating JSON Values

### Variable Declaration with Initial Values

**Integer:**

```lavishscript
variable jsonvalue myInt=1234
echo ${myInt.Type}  ; Output: integer
echo ${myInt}       ; Output: 1234
```

**Number (decimal):**

```lavishscript
variable jsonvalue myNumber=1.5
echo ${myNumber.Type}  ; Output: number
echo ${myNumber}       ; Output: 1.5
```

**String:**

```lavishscript
; Strings require ESCAPED quotes in JSON
variable jsonvalue myString="\"Hello World\""
echo ${myString.Type}  ; Output: string
echo ${myString~}      ; Output: "Hello World" (with quotes)
```

**Note:** The `~` modifier prevents variable expansion and preserves quotes.

**Boolean:**

```lavishscript
variable jsonvalue myTrue=true
variable jsonvalue myFalse=false
echo ${myTrue.Type}   ; Output: true
echo ${myFalse.Type}  ; Output: false
```

**Null:**

```lavishscript
variable jsonvalue myNull=null
echo ${myNull.Type}  ; Output: null
```

**Object:**

```lavishscript
variable jsonvalue myObject="{\"name\":\"apple\",\"color\":\"red\"}"
echo ${myObject.Type}  ; Output: object
echo ${myObject~}      ; Output: {"name":"apple","color":"red"}
```

**Array:**

```lavishscript
variable jsonvalue myArray="[1,2,3,4]"
echo ${myArray.Type}  ; Output: array
echo ${myArray~}       ; Output: [1,2,3,4]
```

**Empty Object/Array:**

```lavishscript
variable jsonvalue emptyObject={}
variable jsonvalue emptyArray=[]
```

### Complete Type Example

```lavishscript
function main()
{
    variable jsonvalue integerValue=1234
    variable jsonvalue numberValue=1.0
    variable jsonvalue stringValue="\"Smooth\""
    variable jsonvalue booleanValue0=false
    variable jsonvalue booleanValue1=true
    variable jsonvalue objectValue="{\"name\":\"apple\",\"color\":\"red\"}"
    variable jsonvalue arrayValue="[1,2,3,4]"
    variable jsonvalue nullValue=null

    echo integerValue[${integerValue.Type}] = ${integerValue}
    echo numberValue[${numberValue.Type}] = ${numberValue}
    echo stringValue[${stringValue.Type}] = "${stringValue~}"
    echo booleanValue0[${booleanValue0.Type}] = ${booleanValue0}
    echo booleanValue1[${booleanValue1.Type}] = ${booleanValue1}
    echo objectValue[${objectValue.Type}] = "${objectValue~}"
    echo arrayValue[${arrayValue.Type}] = "${arrayValue~}"
    echo nullValue[${nullValue.Type}] = ${nullValue}
}
```

**Output:**

```
integerValue[integer] = 1234
numberValue[number] = 1.0
stringValue[string] = "Smooth"
booleanValue0[false] = FALSE
booleanValue1[true] = TRUE
objectValue[object] = {"name":"apple","color":"red"}
arrayValue[array] = [1,2,3,4]
nullValue[null] = NULL
```

---

## Accessing JSON Data

### String Representation

Get JSON as a string:

```lavishscript
variable jsonvalue data="{\"health\":100,\"power\":50}"
echo ${data~}  ; Output: {"health":100,"power":50}
```

**Always use `~` when echoing JSON** to prevent variable expansion.

### Type Checking

```lavishscript
variable jsonvalue myValue=42

if ${myValue.Type.Equal[integer]}
{
    echo It's an integer!
}

; Check for null
if ${myValue.Type.Equal[null]}
{
    echo Value is null
}
```

### Object Property Access

Use `.Get[property]` to access object properties:

```lavishscript
variable jsonvalue player="{\"name\":\"Warrior\",\"level\":50,\"health\":1500}"

echo Name: ${player.Get[name]}        ; Output: Warrior
echo Level: ${player.Get[level]}      ; Output: 50
echo Health: ${player.Get[health]}    ; Output: 1500
```

**With escaping:**

```lavishscript
echo Name: ${player.Get[name]~}  ; Output: "Warrior" (preserves quotes)
```

### Array Element Access

Use `.Get[index]` to access array elements (0-indexed):

```lavishscript
variable jsonvalue numbers="[10,20,30,40]"

echo First: ${numbers.Get[0]}   ; Output: 10
echo Second: ${numbers.Get[1]}  ; Output: 20
echo Third: ${numbers.Get[2]}   ; Output: 30
```

### Nested Access

Access nested objects and arrays:

```lavishscript
variable jsonvalue data="{\"player\":{\"name\":\"Hero\",\"stats\":{\"health\":100,\"power\":50}}}"

echo Name: ${data.Get[player].Get[name]}                ; Output: Hero
echo Health: ${data.Get[player].Get[stats].Get[health]} ; Output: 100
```

---

## Modifying JSON Values

### SetValue Method

Change a `jsonvalue` to a different value (can change type):

```lavishscript
variable jsonvalue myValue="\"Hello\""
echo ${myValue.Type}: ${myValue~}  ; Output: string: "Hello"

; Change to integer
myValue:SetValue[42]
echo ${myValue.Type}: ${myValue}   ; Output: integer: 42

; Change to object
myValue:SetValue["{\"key\":\"value\"}"]
echo ${myValue.Type}: ${myValue~}  ; Output: object: {"key":"value"}

; Change to null
myValue:SetValue[null]
echo ${myValue.Type}: ${myValue}   ; Output: null: NULL
```

### Dynamic Value Example

```lavishscript
function main(string parameter="\"Hello World!\"")
{
    variable jsonvalue userValue
    userValue:SetValue["${parameter~}"]

    echo userValue[${userValue.Type}] = "${userValue~}"
}
```

**Usage:**

```
run myscript 42                   ; integer
run myscript "\"Test\""           ; string
run myscript "{\"a\":1}"          ; object
run myscript null                 ; null
```

### Multi-Line JSON with $$> <$$

For large JSON, use `$$> <$$` notation:

```lavishscript
variable jsonvalue config
config:SetValue["$$>
{
    "window": {
        "width": 800,
        "height": 600,
        "title": "My Window"
    },
    "features": [
        "auto-loot",
        "auto-attack",
        "auto-heal"
    ]
}
<$$"]

echo ${config~}
```

This is **much cleaner** than escaping everything manually!

---

## Working with JSON Objects

### Creating Objects

**Empty object:**

```lavishscript
variable jsonvalue player={}
```

**Initialized object:**

```lavishscript
variable jsonvalue player="{\"name\":\"Warrior\",\"level\":50}"
```

### Setting Object Properties

Use `:Set[key, value]` or type-specific methods:

**Generic Set:**

```lavishscript
variable jsonvalue player={}
player:Set[name,"\"Warrior\""]
player:Set[level,50]
player:Set[health,1500]

echo ${player~}  ; Output: {"name":"Warrior","level":50,"health":1500}
```

**Type-Specific Set Methods:**

```lavishscript
variable jsonvalue player={}

; SetString - Automatically adds quotes and escapes
player:SetString[name,"Warrior"]

; SetInteger
player:SetInteger[level,50]

; SetNumber
player:SetNumber[healthPercent,85.5]

; SetBool
player:SetBool[alive,TRUE]

echo ${player~}
; Output: {"name":"Warrior","level":50,"healthPercent":85.5,"alive":true}
```

**SetString is safer** - it handles escaping automatically!

### Getting Object Properties

```lavishscript
variable jsonvalue player="{\"name\":\"Warrior\",\"level\":50,\"health\":1500}"

variable string playerName
playerName:Set["${player.Get[name]~}"]

variable int playerLevel=${player.Get[level]}
variable int playerHealth=${player.Get[health]}

echo Name: ${playerName}
echo Level: ${playerLevel}
echo Health: ${playerHealth}
```

### Checking if Property Exists

```lavishscript
variable jsonvalue player="{\"name\":\"Warrior\",\"level\":50}"

if ${player.Get[name](exists)}
{
    echo Name exists: ${player.Get[name]~}
}

if !${player.Get[guild](exists)}
{
    echo No guild property
}
```

### Nested Objects

```lavishscript
variable jsonvalue character={}

; Create nested stats object
variable jsonvalue stats={}
stats:SetInteger[health,1500]
stats:SetInteger[power,500]
stats:SetInteger[strength,100]

; Add stats to character
character:SetString[name,"Warrior"]
character:SetInteger[level,50]
character:Set[stats,"${stats~}"]

echo ${character~}
; Output: {"name":"Warrior","level":50,"stats":{"health":1500,"power":500,"strength":100}}
```

---

## Working with JSON Arrays

### Creating Arrays

**Empty array:**

```lavishscript
variable jsonvalue items=[]
```

**Initialized array:**

```lavishscript
variable jsonvalue numbers="[1,2,3,4,5]"
```

### Accessing Array Elements

```lavishscript
variable jsonvalue fruits="[\"apple\",\"banana\",\"cherry\"]"

echo First: ${fruits.Get[0]~}   ; Output: "apple"
echo Second: ${fruits.Get[1]~}  ; Output: "banana"
echo Third: ${fruits.Get[2]~}   ; Output: "cherry"
```

### Array Size

```lavishscript
variable jsonvalue numbers="[10,20,30,40,50]"
echo Size: ${numbers.Size}  ; Output: 5
```

### Adding to Arrays

Use `:Add[value]`:

```lavishscript
variable jsonvalue items=[]

items:Add["\"sword\""]
items:Add["\"shield\""]
items:Add["\"potion\""]

echo ${items~}  ; Output: ["sword","shield","potion"]
```

**Type-specific Add methods:**

```lavishscript
variable jsonvalue numbers=[]

numbers:AddInteger[10]
numbers:AddInteger[20]
numbers:AddInteger[30]

echo ${numbers~}  ; Output: [10,20,30]
```

```lavishscript
variable jsonvalue names=[]

names:AddString["John"]
names:AddString["Jane"]
names:AddString["Bob"]

echo ${names~}  ; Output: ["John","Jane","Bob"]
```

### Iterating Arrays with ForEach

```lavishscript
variable jsonvalue fruits="[\"apple\",\"banana\",\"cherry\"]"

fruits:ForEach["echo Fruit: \${ForEach.Value~}"]
```

**Output:**

```
Fruit: "apple"
Fruit: "banana"
Fruit: "cherry"
```

**With index:**

```lavishscript
fruits:ForEach["echo [\${ForEach.Key}] = \${ForEach.Value~}"]
```

**Output:**

```
[0] = "apple"
[1] = "banana"
[2] = "cherry"
```

### Array of Objects

```lavishscript
variable jsonvalue players="$$>
[
    {\"name\":\"Warrior\",\"level\":50},
    {\"name\":\"Mage\",\"level\":48},
    {\"name\":\"Priest\",\"level\":52}
]
<$$"

; Access specific player
echo First player: ${players.Get[0].Get[name]~}  ; Output: "Warrior"

; Iterate all players
players:ForEach["echo Player: \${ForEach.Value.Get[name]~} Level: \${ForEach.Value.Get[level]}"]
```

**Output:**

```
Player: "Warrior" Level: 50
Player: "Mage" Level: 48
Player: "Priest" Level: 52
```

---

## Reading and Writing JSON Files

### Reading JSON from File

Use `:ParseFile[filename]`:

**example.json:**

```json
{
    "fruits": [
        {"name": "apple", "color": "red"},
        {"name": "blueberry", "color": "blue"}
    ],
    "vegetables": [
        {"name": "carrot", "color": "orange"}
    ]
}
```

**Script:**

```lavishscript
function main()
{
    variable jsonvalue data
    data:ParseFile["example.json"]

    echo ${data~}

    ; Access specific values
    echo First fruit: ${data.Get[fruits].Get[0].Get[name]~}
    ; Output: "apple"
}
```

**Relative paths** are relative to the script location.

### Writing JSON to File

Use `:WriteFile[filename]`:

```lavishscript
function main()
{
    variable jsonvalue config={}
    config:SetString[version,"1.0"]
    config:SetBool[autoLoot,TRUE]
    config:SetInteger[maxTargets,5]

    config:WriteFile["config.json"]
    echo Config saved to config.json
}
```

**config.json output:**

```json
{"version":"1.0","autoLoot":true,"maxTargets":5}
```

**Pretty printing** - Use `:WriteFile[filename, TRUE]` for readable formatting:

```lavishscript
config:WriteFile["config.json", TRUE]
```

**Output (pretty):**

```json
{
    "version": "1.0",
    "autoLoot": true,
    "maxTargets": 5
}
```

### Complete File I/O Example

```lavishscript
function SaveSettings()
{
    variable jsonvalue settings={}
    settings:SetBool[autoAttack,${AutoAttackEnabled}]
    settings:SetBool[autoLoot,${AutoLootEnabled}]
    settings:SetInteger[combatRange,${CombatRange}]

    settings:WriteFile["settings.json", TRUE]
    echo Settings saved
}

function LoadSettings()
{
    variable jsonvalue settings

    if !${settings:ParseFile["settings.json"](exists)}
    {
        echo Settings file not found, using defaults
        return
    }

    if ${settings.Get[autoAttack](exists)}
        AutoAttackEnabled:Set[${settings.Get[autoAttack]}]

    if ${settings.Get[autoLoot](exists)}
        AutoLootEnabled:Set[${settings.Get[autoLoot]}]

    if ${settings.Get[combatRange](exists)}
        CombatRange:Set[${settings.Get[combatRange]}]

    echo Settings loaded
}
```

---

## Object Serialization (AsJSON)

### The AsJSON Pattern

Custom objects can define an `AsJSON` member to convert themselves to JSON:

```lavishscript
objectdef person
{
    variable string FirstName="John"
    variable string LastName="Doe"

    member:jsonvalueref AsJSON()
    {
        variable jsonvalue jo={}
        jo:SetString[first_name,"${FirstName~}"]
        jo:SetString[last_name,"${LastName~}"]
        return jo
    }
}
```

**Usage:**

```lavishscript
variable person myPerson
myPerson.FirstName:Set[Jane]
myPerson.LastName:Set[Smith]

echo ${myPerson.AsJSON~}
; Output: {"first_name":"Jane","last_name":"Smith"}
```

### Why AsJSON is Powerful

1. ✅ **Automatic serialization** - Objects convert to JSON when needed
2. ✅ **Consistency** - All objects follow same pattern
3. ✅ **Integration** - Works with collections, indexes, and files
4. ✅ **Clean code** - Encapsulates serialization logic

### AsJSON with Nested Objects

```lavishscript
objectdef stats
{
    variable int Health=100
    variable int Power=50

    member:jsonvalueref AsJSON()
    {
        variable jsonvalue jo={}
        jo:SetInteger[health,${Health}]
        jo:SetInteger[power,${Power}]
        return jo
    }
}

objectdef character
{
    variable string Name="Hero"
    variable int Level=50
    variable stats Stats

    member:jsonvalueref AsJSON()
    {
        variable jsonvalue jo={}
        jo:SetString[name,"${Name~}"]
        jo:SetInteger[level,${Level}]
        jo:Set[stats,"${Stats.AsJSON~}"]  ; Nested object
        return jo
    }
}
```

**Usage:**

```lavishscript
variable character myChar
myChar.Name:Set[Warrior]
myChar.Level:Set[60]
myChar.Stats.Health:Set[1500]
myChar.Stats.Power:Set[500]

echo ${myChar.AsJSON~}
; Output: {"name":"Warrior","level":60,"stats":{"health":1500,"power":500}}
```

### Saving Objects to Files

```lavishscript
variable character myChar

; Configure character
myChar.Name:Set[Paladin]
myChar.Level:Set[55]

; Save to file
variable jsonvalueref charJSON
charJSON:SetReference["myChar.AsJSON"]
charJSON:WriteFile["character.json", TRUE]
```

---

## Object Deserialization (FromJSON)

### The FromJSON Pattern

Custom objects can define `FromJSON` or `SetFromJSON` methods to initialize from JSON:

```lavishscript
objectdef person
{
    variable string FirstName
    variable string LastName

    ; Initialize from JSON
    method Initialize(jsonvalueref jo)
    {
        This:FromJSON[jo]
    }

    ; Set from JSON (can be called anytime)
    method SetFromJSON(jsonvalueref jo)
    {
        This:FromJSON[jo]
    }

    ; Core deserialization logic
    method FromJSON(jsonvalueref jo)
    {
        ; Validate it's an object
        if !${jo.Type.Equal[object]}
            return

        ; Extract properties
        FirstName:Set["${jo.Get[first_name]~}"]
        LastName:Set["${jo.Get[last_name]~}"]
    }

    member:jsonvalueref AsJSON()
    {
        variable jsonvalue jo={}
        jo:SetString[first_name,"${FirstName~}"]
        jo:SetString[last_name,"${LastName~}"]
        return jo
    }
}
```

**Usage:**

```lavishscript
variable person myPerson

; Initialize from JSON
myPerson:SetFromJSON["{\"first_name\":\"John\",\"last_name\":\"Doe\"}"]

echo Name: ${myPerson.FirstName} ${myPerson.LastName}
; Output: Name: John Doe

echo As JSON: ${myPerson.AsJSON~}
; Output: {"first_name":"John","last_name":"Doe"}
```

### Loading Objects from Files

```lavishscript
variable person loadedPerson
variable jsonvalue personData

personData:ParseFile["person.json"]
loadedPerson:SetFromJSON[personData]

echo Loaded: ${loadedPerson.FirstName} ${loadedPerson.LastName}
```

### Why FromJSON is Powerful

1. ✅ **Automatic deserialization** - Objects populate from JSON
2. ✅ **Validation** - Can validate JSON structure before loading
3. ✅ **Flexibility** - Can handle missing or extra properties
4. ✅ **Persistence** - Easy save/load from files

---

## Collections and Indexes with JSON

### Index with AsJSON/FromJSON

An `index` automatically serializes/deserializes if elements have `AsJSON` and `Initialize(jsonvalueref)`:

```lavishscript
objectdef person
{
    variable string FirstName
    variable string LastName

    method Initialize(jsonvalueref jo)
    {
        This:FromJSON[jo]
    }

    method FromJSON(jsonvalueref jo)
    {
        if !${jo.Type.Equal[object]}
            return
        FirstName:Set["${jo.Get[first_name]~}"]
        LastName:Set["${jo.Get[last_name]~}"]
    }

    member:jsonvalueref AsJSON()
    {
        variable jsonvalue jo={}
        jo:SetString[first_name,"${FirstName~}"]
        jo:SetString[last_name,"${LastName~}"]
        return jo
    }
}

function main()
{
    variable index:person Persons

    ; JSON array input
    variable jsonvalue jaPersons="$$>
    [
        {\"first_name\":\"John\",\"last_name\":\"Doe\"},
        {\"first_name\":\"Jane\",\"last_name\":\"Doe\"},
        {\"first_name\":\"John\",\"last_name\":\"Public\"}
    ]
    <$$"

    ; Deserialize from JSON array
    Persons:FromJSON[jaPersons]

    ; Serialize back to JSON
    echo Output: ${Persons.AsJSON~}
    ; Output: [{"first_name":"John","last_name":"Doe"},{"first_name":"Jane","last_name":"Doe"},{"first_name":"John","last_name":"Public"}]
}
```

### Collection with AsJSON/FromJSON

A `collection` can serialize as object or array:

```lavishscript
function main()
{
    variable collection:person Persons

    variable jsonvalue jaPersons="$$>
    [
        {\"first_name\":\"John\",\"last_name\":\"Doe\"},
        {\"first_name\":\"Jane\",\"last_name\":\"Doe\"}
    ]
    <$$"

    ; Build collection from array
    ; Key = full name, Value = person object
    jaPersons:ForEach["Persons:Set[\"\${ForEach.Value.Get[first_name]~} \${ForEach.Value.Get[last_name]~}\",ForEach.Value]"]

    ; Serialize as object (keyed)
    echo As object: ${Persons.AsJSON~}
    ; Output: {"John Doe":{"first_name":"John","last_name":"Doe"},"Jane Doe":{"first_name":"Jane","last_name":"Doe"}}

    ; Serialize as array (values only)
    echo As array: ${Persons.AsJSON[array]~}
    ; Output: [{"first_name":"John","last_name":"Doe"},{"first_name":"Jane","last_name":"Doe"}]

    ; Deserialize from object
    Persons:Clear
    variable jsonvalue joPersons="{\"John Doe\":{\"first_name\":\"John\",\"last_name\":\"Doe\"}}"
    Persons:FromJSON[joPersons]

    echo Restored: ${Persons.AsJSON~}
}
```

**collection:AsJSON** formats:
- `${collection.AsJSON~}` - Object format (with keys)
- `${collection.AsJSON[array]~}` - Array format (values only)

---

## JSON References (jsonvalueref)

### What is jsonvalueref?

`jsonvalueref` is a **reference** to a JSON value, not a copy. This is **more efficient** than copying JSON data.

### When to Use jsonvalueref

Use `jsonvalueref` for:
- ✅ **Method parameters** - Avoid copying large JSON objects
- ✅ **Return values** - Return AsJSON as reference, not string
- ✅ **Performance** - When working with large JSON data

### jsonvalueref vs jsonvalue

| Feature | jsonvalue | jsonvalueref |
|---------|-----------|--------------|
| **Storage** | Stores actual JSON data | References existing data |
| **Performance** | Copies data | No copy (faster) |
| **Lifetime** | Independent | Depends on source |
| **Use Case** | Variables, storage | Parameters, returns |

### Using jsonvalueref

**As parameter:**

```lavishscript
method FromJSON(jsonvalueref jo)
{
    ; jo is a reference, not a copy
    echo ${jo~}
}
```

**As return value:**

```lavishscript
member:jsonvalueref AsJSON()
{
    variable jsonvalue jo={}
    jo:SetString[name,"Value"]
    return jo  ; Returns reference
}
```

**Setting a reference:**

```lavishscript
variable jsonvalue source="{\"a\":1,\"b\":2}"
variable jsonvalueref ref

ref:SetReference["source"]
echo ${ref~}  ; Output: {"a":1,"b":2}

; Modifying source affects ref
source:Set[c,3]
echo ${ref~}  ; Output: {"a":1,"b":2,"c":3}
```

### Performance Benefit Example

**Inefficient (copying):**

```lavishscript
method ProcessData(jsonvalue data)  ; Copies entire JSON!
{
    echo ${data~}
}

variable jsonvalue bigData  ; 10MB of JSON
bigData:ParseFile["huge.json"]
This:ProcessData[bigData]  ; Copies 10MB!
```

**Efficient (reference):**

```lavishscript
method ProcessData(jsonvalueref data)  ; Just a reference!
{
    echo ${data~}
}

variable jsonvalue bigData  ; 10MB of JSON
bigData:ParseFile["huge.json"]
This:ProcessData[bigData]  ; No copy, instant!
```

---

## Best Practices

### 1. Always Use ~ for JSON Output

```lavishscript
; BAD - Variable expansion can break JSON
echo ${jsonData}

; GOOD - Preserves JSON exactly
echo ${jsonData~}
```

### 2. Use Type-Specific Set Methods

```lavishscript
; BAD - Manual escaping, error-prone
obj:Set[name,"\"John\""]

; GOOD - Automatic escaping
obj:SetString[name,"John"]
```

### 3. Validate JSON Type Before Access

```lavishscript
; BAD - May crash if wrong type
echo ${data.Get[property]}

; GOOD - Check type first
if ${data.Type.Equal[object]}
{
    if ${data.Get[property](exists)}
        echo ${data.Get[property]~}
}
```

### 4. Use $$> <$$ for Large JSON

```lavishscript
; BAD - Hard to read and maintain
variable jsonvalue data="{\"a\":1,\"b\":{\"c\":2,\"d\":3}}"

; GOOD - Clean and readable
variable jsonvalue data="$$>
{
    \"a\": 1,
    \"b\": {
        \"c\": 2,
        \"d\": 3
    }
}
<$$"
```

### 5. Use jsonvalueref for Parameters

```lavishscript
; BAD - Copies JSON data
method ProcessConfig(jsonvalue config)
{
    ; ...
}

; GOOD - No copy, more efficient
method ProcessConfig(jsonvalueref config)
{
    ; ...
}
```

### 6. Implement AsJSON and FromJSON for Custom Types

```lavishscript
objectdef mytype
{
    ; Always implement both for persistence
    member:jsonvalueref AsJSON()
    {
        ; Serialization
    }

    method FromJSON(jsonvalueref jo)
    {
        ; Deserialization
    }
}
```

### 7. Pretty Print When Writing Config Files

```lavishscript
; BAD - Hard to read/edit manually
config:WriteFile["config.json"]

; GOOD - Readable, editable
config:WriteFile["config.json", TRUE]
```

### 8. Use Relative Paths for Portability

```lavishscript
; BAD - Absolute path, not portable
data:ParseFile["C:\\Scripts\\MyScript\\config.json"]

; GOOD - Relative to script
data:ParseFile["config.json"]
```

---

## Complete Examples

### Example 1: Configuration Management

**config.iss:**

```lavishscript
objectdef script_config
{
    variable bool AutoAttack=TRUE
    variable bool AutoLoot=TRUE
    variable int CombatRange=30
    variable string TargetMode="Nearest"

    method Initialize()
    {
        This:Load
    }

    method Load()
    {
        variable jsonvalue data

        if !${data:ParseFile["config.json"](exists)}
        {
            echo Config not found, using defaults
            return
        }

        This:FromJSON[data]
        echo Config loaded
    }

    method Save()
    {
        variable jsonvalueref joConfig
        joConfig:SetReference["This.AsJSON"]
        joConfig:WriteFile["config.json", TRUE]
        echo Config saved
    }

    method FromJSON(jsonvalueref jo)
    {
        if !${jo.Type.Equal[object]}
            return

        if ${jo.Get[autoAttack](exists)}
            AutoAttack:Set[${jo.Get[autoAttack]}]
        if ${jo.Get[autoLoot](exists)}
            AutoLoot:Set[${jo.Get[autoLoot]}]
        if ${jo.Get[combatRange](exists)}
            CombatRange:Set[${jo.Get[combatRange]}]
        if ${jo.Get[targetMode](exists)}
            TargetMode:Set["${jo.Get[targetMode]~}"]
    }

    member:jsonvalueref AsJSON()
    {
        variable jsonvalue jo={}
        jo:SetBool[autoAttack,${AutoAttack}]
        jo:SetBool[autoLoot,${AutoLoot}]
        jo:SetInteger[combatRange,${CombatRange}]
        jo:SetString[targetMode,"${TargetMode~}"]
        return jo
    }
}

variable(global) script_config Config

function main()
{
    echo Current config: ${Config.AsJSON~}

    ; Modify settings
    Config.AutoAttack:Set[FALSE]
    Config.CombatRange:Set[40]

    ; Save
    Config:Save

    while 1
        waitframe
}

function OnExit()
{
    Config:Save
}
```

### Example 2: Player Stat Tracker

**stats.iss:**

```lavishscript
objectdef player_stats
{
    variable string Name
    variable int Level
    variable int TotalKills=0
    variable int TotalDeaths=0
    variable int64 TotalDamage=0
    variable int64 SessionStart

    method Initialize()
    {
        This.SessionStart:Set[${Script.RunningTime}]
    }

    method RecordKill()
    {
        This.TotalKills:Inc
    }

    method RecordDeath()
    {
        This.TotalDeaths:Inc
    }

    method RecordDamage(int64 amount)
    {
        This.TotalDamage:Inc[${amount}]
    }

    method SaveStats()
    {
        variable jsonvalueref joStats
        joStats:SetReference["This.AsJSON"]
        joStats:WriteFile["stats_${Name}.json", TRUE]
        echo Stats saved for ${Name}
    }

    method LoadStats()
    {
        variable jsonvalue data
        if ${data:ParseFile["stats_${Name}.json"](exists)}
        {
            This:FromJSON[data]
            echo Stats loaded for ${Name}
        }
    }

    method FromJSON(jsonvalueref jo)
    {
        if !${jo.Type.Equal[object]}
            return

        if ${jo.Get[level](exists)}
            Level:Set[${jo.Get[level]}]
        if ${jo.Get[totalKills](exists)}
            TotalKills:Set[${jo.Get[totalKills]}]
        if ${jo.Get[totalDeaths](exists)}
            TotalDeaths:Set[${jo.Get[totalDeaths]}]
        if ${jo.Get[totalDamage](exists)}
            TotalDamage:Set[${jo.Get[totalDamage]}]
    }

    member:jsonvalueref AsJSON()
    {
        variable jsonvalue jo={}
        jo:SetString[name,"${Name~}"]
        jo:SetInteger[level,${Level}]
        jo:SetInteger[totalKills,${TotalKills}]
        jo:SetInteger[totalDeaths,${TotalDeaths}]
        jo:SetInteger[totalDamage,${TotalDamage}]

        variable int64 sessionTime=${Math.Calc[${Script.RunningTime}-${SessionStart}]}
        jo:SetInteger[sessionTime,${sessionTime}]

        return jo
    }

    method DisplayStats()
    {
        echo ===== Stats for ${Name} =====
        echo Level: ${Level}
        echo Kills: ${TotalKills}
        echo Deaths: ${TotalDeaths}
        echo Damage: ${TotalDamage}
        echo K/D Ratio: ${Math.Calc[${TotalKills}/${TotalDeaths}]}
    }
}

variable(global) player_stats Stats

function main()
{
    Stats.Name:Set["${Me.Name}"]
    Stats.Level:Set[${Me.Level}]
    Stats:LoadStats

    ; Simulate some combat
    Stats:RecordKill
    Stats:RecordDamage[1523]
    Stats:RecordKill
    Stats:RecordDamage[2341]

    Stats:DisplayStats
    Stats:SaveStats

    while 1
        waitframe
}
```

### Example 3: Target List Manager

**targets.iss:**

```lavishscript
objectdef target_entry
{
    variable string Name
    variable int Level
    variable string Type
    variable float Distance

    method Initialize(jsonvalueref jo)
    {
        This:FromJSON[jo]
    }

    method FromJSON(jsonvalueref jo)
    {
        if !${jo.Type.Equal[object]}
            return

        Name:Set["${jo.Get[name]~}"]
        Level:Set[${jo.Get[level]}]
        Type:Set["${jo.Get[type]~}"]
        Distance:Set[${jo.Get[distance]}]
    }

    member:jsonvalueref AsJSON()
    {
        variable jsonvalue jo={}
        jo:SetString[name,"${Name~}"]
        jo:SetInteger[level,${Level}]
        jo:SetString[type,"${Type~}"]
        jo:SetNumber[distance,${Distance}]
        return jo
    }
}

objectdef target_manager
{
    variable collection:target_entry Targets

    method ScanNearbyTargets()
    {
        Targets:Clear

        variable index:actor nearbyActors
        EQ2:QueryActors[nearbyActors, "Type = NPC && Distance < 50"]

        variable iterator iter
        nearbyActors:GetIterator[iter]

        if ${iter:First(exists)}
        {
            do
            {
                variable jsonvalue joTarget={}
                joTarget:SetString[name,"${iter.Value.Name~}"]
                joTarget:SetInteger[level,${iter.Value.Level}]
                joTarget:SetString[type,"${iter.Value.Type~}"]
                joTarget:SetNumber[distance,${iter.Value.Distance}]

                Targets:Set["${iter.Value.Name~}",joTarget]
            }
            while ${iter:Next(exists)}
        }

        echo Scanned ${Targets.Used} targets
    }

    method SaveTargets()
    {
        variable jsonvalueref joTargets
        joTargets:SetReference["Targets.AsJSON[array]"]
        joTargets:WriteFile["targets.json", TRUE]
        echo Targets saved
    }

    method LoadTargets()
    {
        variable jsonvalue data
        if ${data:ParseFile["targets.json"](exists)}
        {
            Targets:Clear
            data:ForEach["Targets:Set[\"\${ForEach.Value.Get[name]~}\",ForEach.Value]"]
            echo Loaded ${Targets.Used} targets
        }
    }

    method DisplayTargets()
    {
        echo ===== Nearby Targets =====

        variable iterator iter
        Targets:GetIterator[iter]

        if ${iter:First(exists)}
        {
            do
            {
                echo ${iter.Value.Name} (Lvl ${iter.Value.Level}) - ${iter.Value.Distance}m
            }
            while ${iter:Next(exists)}
        }
    }
}

variable(global) target_manager TargetMgr

function main()
{
    TargetMgr:ScanNearbyTargets
    TargetMgr:DisplayTargets
    TargetMgr:SaveTargets

    while 1
    {
        wait 50
        TargetMgr:ScanNearbyTargets
    }
}
```

---

## Troubleshooting

### Issue 1: JSON Parse Error

**Problem:**

```
Error parsing JSON
```

**Causes:**
- Invalid JSON syntax
- Missing quotes
- Trailing commas
- Unescaped special characters

**Fix:**

```lavishscript
; BAD - Missing quotes on key
variable jsonvalue data="{name:\"value\"}"

; GOOD
variable jsonvalue data="{\"name\":\"value\"}"

; BAD - Trailing comma
variable jsonvalue data="{\"a\":1,\"b\":2,}"

; GOOD
variable jsonvalue data="{\"a\":1,\"b\":2}"
```

Use a JSON validator (jsonlint.com) to check syntax.

### Issue 2: String Not Escaped

**Problem:**

```lavishscript
variable jsonvalue obj={}
obj:Set[name,"John"]  ; Missing quotes!
echo ${obj~}  ; ERROR or wrong output
```

**Fix:**

```lavishscript
; Option 1: Manual escaping
obj:Set[name,"\"John\""]

; Option 2: Use SetString (RECOMMENDED)
obj:SetString[name,"John"]
```

### Issue 3: Variable Expansion in JSON

**Problem:**

```lavishscript
variable jsonvalue data="{\"value\":${SomeVar}}"
echo ${data}  ; Variable gets expanded!
```

**Fix:**

```lavishscript
; Always use ~ to prevent expansion
echo ${data~}
```

### Issue 4: Type Mismatch

**Problem:**

```lavishscript
variable jsonvalue data="[1,2,3]"
echo ${data.Get[name]}  ; ERROR - array doesn't have "Get[name]"
```

**Fix:**

```lavishscript
; Check type first
if ${data.Type.Equal[array]}
{
    echo ${data.Get[0]}
}
else if ${data.Type.Equal[object]}
{
    echo ${data.Get[name]~}
}
```

### Issue 5: File Not Found

**Problem:**

```lavishscript
variable jsonvalue data
data:ParseFile["config.json"]  ; File not found
```

**Fix:**

```lavishscript
if !${data:ParseFile["config.json"](exists)}
{
    echo File not found, using defaults
    ; Initialize with defaults
    data:SetValue["{}"]
}
```

### Issue 6: Reference Lifetime

**Problem:**

```lavishscript
member:jsonvalueref GetData()
{
    variable jsonvalue temp="{\"a\":1}"
    return temp  ; temp is destroyed when function returns!
}

variable jsonvalueref myRef
myRef:SetReference["This:GetData"]
echo ${myRef~}  ; ERROR - reference is invalid
```

**Fix:**

```lavishscript
; Use a persistent variable, not local
variable jsonvalue persistentData="{\"a\":1}"

member:jsonvalueref GetData()
{
    return persistentData  ; Safe - persistentData exists
}
```

---

## Quick Reference

### Creating JSON

```lavishscript
variable jsonvalue myInt=42
variable jsonvalue myStr="\"text\""
variable jsonvalue myObj={}
variable jsonvalue myArr=[]
```

### Setting Values

```lavishscript
myValue:SetValue[42]
myObj:SetString[key,"value"]
myObj:SetInteger[num,100]
myObj:SetBool[flag,TRUE]
myArr:AddString["item"]
```

### Accessing Data

```lavishscript
echo ${myObj.Get[key]~}
echo ${myArr.Get[0]~}
echo ${myValue.Type}
```

### Files

```lavishscript
data:ParseFile["file.json"]
data:WriteFile["file.json", TRUE]
```

### Serialization

```lavishscript
member:jsonvalueref AsJSON()
{
    variable jsonvalue jo={}
    jo:SetString[prop,"${PropValue~}"]
    return jo
}
```

### Deserialization

```lavishscript
method FromJSON(jsonvalueref jo)
{
    if ${jo.Type.Equal[object]}
        PropValue:Set["${jo.Get[prop]~}"]
}
```

---

<!-- CLAUDE_SKIP_START -->
## Additional Resources

- **JSON Standard:** https://www.json.org
- **LavishScript Reference:** http://www.lavishsoft.com/wiki/LavishScript
- **LERN JSON Examples:** https://github.com/LavishSoftware/LERN/tree/master/JSON

---

**This guide was created through comprehensive analysis of:**
- LERN/JSON tutorial files: https://github.com/LavishSoftware/LERN/tree/master/JSON (1.md, 2.md, 3.md)
- LERN/JSON example scripts (1.iss, 2.iss, 3.iss)
- LERN/JSON serialization examples (serialize-1 through serialize-4)
- Complete coverage of jsonvalue, jsonvalueref, jsonobject, jsonarray
- Real-world use cases and patterns

---

*Last Updated: 2025-10-21*
*LavishScript JSON API - Complete Coverage*

*Part of ISXEVE Scripting Guide*
<!-- CLAUDE_SKIP_END -->
