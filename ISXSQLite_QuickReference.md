# isxSQLite Reference

## Table of Contents

1. [Introduction](#introduction)
2. [Top-Level Objects](#top-level-objects)
   - [Extension TLOs](#extension-tlos)
   - [Core TLOs](#core-tlos)
3. [DataTypes](#datatypes)
   - [Core/Extension DataTypes](#coreextension-datatypes)
   - [Database DataTypes](#database-datatypes)
   - [Query DataTypes](#query-datatypes)
   - [Table DataTypes](#table-datatypes)
4. [Commands](#commands)
   - [Test Commands](#test-commands)
5. [Events](#events)
   - [Error Events](#error-events)
   - [Status Events](#status-events)
6. [Usage Examples](#usage-examples)
   - [Checking Extension Status and Setting Up Events](#checking-extension-status-and-setting-up-events)
   - [Opening and Closing Databases](#opening-and-closing-databases)
   - [Creating Tables and Checking Schema](#creating-tables-and-checking-schema)
   - [Inserting Data with Transactions](#inserting-data-with-transactions)
   - [Inserting Data Individually](#inserting-data-individually)
   - [Querying Data with Do/While Loop](#querying-data-with-dowhile-loop)
   - [Querying Data with For Loop](#querying-data-with-for-loop)
   - [Exporting an Entire Table](#exporting-an-entire-table)
   - [Working with Memory Databases](#working-with-memory-databases)
   - [Using Escape_String for SQL Injection Prevention](#using-escape_string-for-sql-injection-prevention)
   - [Complete Production Script Example](#complete-production-script-example)
7. [Notes](#notes)
   - [Case Sensitivity](#case-sensitivity)
   - [NULL Checks](#null-checks)
   - [Parameter Notation](#parameter-notation)
   - [Resource Management](#resource-management)
   - [Database Defaults](#database-defaults)
   - [Events and Quiet Mode](#events-and-quiet-mode)
   - [Query vs Table Usage](#query-vs-table-usage)
   - [LastRow Behavior](#lastrow-behavior)
   - [ExecDMLTransaction Details](#execdmltransaction-details)
   - [File Paths](#file-paths)
   - [Database Name Uniqueness](#database-name-uniqueness)
   - [sqlitetable.GetFieldValue Typed-Read Caveat](#sqlitetablegetfieldvalue-typed-read-caveat)

---

## Introduction

This document provides comprehensive reference documentation for all datatypes, top-level objects, commands, and events available in the isxSQLite extension for InnerSpace. isxSQLite extends LavishScript to provide access to SQLite database functionality through a structured type system.

isxSQLite allows you to create, access, and manipulate SQLite databases from LavishScript. You can create databases in memory or on disk, execute SQL statements, query data, and manage results through an easy-to-use object-oriented interface.

### Key Features

- Create and manage SQLite databases (disk or memory-based)
- Execute SQL Data Manipulation Language (DML) statements
- Query databases with full result navigation
- Retrieve entire tables or execute custom queries
- Transaction support for efficient bulk operations
- String escaping utilities for SQL injection prevention
- Event-driven error and status reporting

### Source and Versioning

- **Open-source repository:** [github.com/isxGames/isxSQLite](https://github.com/isxGames/isxSQLite)
- Bundled SQLite engine version: **3.32.3** (as of the [isxSQLite-20200812.0001](#) update).
- Versioning scheme: `YYYYMMDD.####` (e.g., `20200812.0001`).

### Accessing DataTypes

DataTypes are accessed through Top-Level Objects (TLOs) or through members of other datatypes. For example:

```lavishscript
${ISXSQLite}                                    // TLO returning 'isxsqlite' datatype
${SQLite}                                       // TLO returning 'sqlite' datatype
${SQLite.OpenDB["MyDB","./myfile.db"]}          // Member returning 'sqlitedb' datatype
${MyDB.ExecQuery["SELECT * FROM users"]}        // Member returning 'sqlitequery' datatype
```

---

## Top-Level Objects

Top-Level Objects (TLOs) are the entry points for accessing isxSQLite functionality. They can be accessed directly in LavishScript.

### Extension TLOs

| TLO | DataType | Description |
|-----|----------|-------------|
| **ISXSQLite** | [isxsqlite](#isxsqlite) | Extension status and quiet-mode control. Available immediately on load. |

### Core TLOs

| TLO | DataType | Description |
|-----|----------|-------------|
| **SQLite** | [sqlite](#sqlite) | Main SQLite interface for opening databases and utility helpers. Available after extension finishes loading. |

---

## DataTypes

### Core/Extension DataTypes

#### **isxsqlite**

Extension information and utilities. Accessed via the [ISXSQLite](#extension-tlos) TLO.

**Members:**

| Member | Return Type | Description |
|--------|-------------|-------------|
| `Version` | string | isxSQLite version number |
| `IsReady` | bool | Whether the extension has completed post-initialization and is ready for use |
| `IsLoading` | bool | Whether the extension is currently in the middle of loading |
| `InQuietMode` | bool | Whether quiet mode is active (console messages suppressed) |

**Methods:**

| Method | Description |
|--------|-------------|
| `QuietMode` | Toggles quiet mode on/off. When active, no error messages or method results are printed to the console; however, all messages still fire via applicable events. |

**See Also:**
- [sqlite](#sqlite) — Main SQLite interface

---

#### **sqlite**

Main SQLite datatype providing database management functionality. Accessed via the [SQLite](#core-tlos) TLO.

**Members:**

| Member | Return Type | Description |
|--------|-------------|-------------|
| `OpenDB[name,path]` | [sqlitedb](#sqlitedb) | Opens or creates a database. `name` is a session-unique identifier; `path` is the file path. Use `":Memory:"` as the path to create an in-memory database. The base directory is `innerspace/Extensions`. If the file does not exist it will be created with the default PRAGMA settings (see [Database Defaults](#database-defaults)). |
| `GetQueryByID[id]` | [sqlitequery](#sqlitequery) | Retrieves a live query object by its numeric ID |
| `Escape_String[text]` | string | Returns an escaped string suitable for SQL use (equivalent to `sqlite3_mprintf("%q", text)` or PHP's `sqlite_escape_string()`). Example: `test` becomes `''test''`. |

**Methods:** *(none)*

**See Also:**
- [sqlitedb](#sqlitedb) — Database object
- [sqlitequery](#sqlitequery) — Query result object
- [sqlitetable](#sqlitetable) — Table result object

---

### Database DataTypes

#### **sqlitedb**

Represents an open SQLite database connection. Obtained from [sqlite](#sqlite)'s `OpenDB` member, or by declaring a variable and setting it to an existing database ID.

**Members:**

| Member | Return Type | Description |
|--------|-------------|-------------|
| `ID` | string | The unique session identifier for this database |
| `ExecQuery[statement]` | [sqlitequery](#sqlitequery) | Executes a SQL statement (typically SELECT) and returns a query object for stepping through results. Queries that return no results, or that encounter an error, finalize automatically. |
| `TableExists[tablename]` | bool | Returns TRUE if the named table exists in the database |
| `GetTable[tablename]` | [sqlitetable](#sqlitetable) | Retrieves an entire table. If `tablename` exactly matches an existing table, the hardcoded statement `SELECT * from <TableName> order by 1;` is used. Otherwise `GetTable` treats the argument as a custom DML statement. |
| `GetTable[statement]` | [sqlitetable](#sqlitetable) | Same member as above, used with a custom DML statement when the argument does not match an existing table name. Note: `GetTable` allocates space for the entire result set — memory-intensive for large queries. Always `Finalize` the result. |

**Methods:**

| Method | Description |
|--------|-------------|
| `Set[sqlitedb or id]` | Rebinds this variable to reference another database (by object or ID) |
| `Close` | Closes the database connection |
| `ExecDML[statement]` | Executes a Data Manipulation Language statement that does not return data (INSERT, UPDATE, DELETE, CREATE TABLE, PRAGMA, etc.). Succeeds silently unless an error occurs, which fires [isxSQLite_onErrorMsg](#isxsqlite_onerrormsg) and prints to console unless `QuietMode` is active. |
| `ExecDMLTransaction[index:string]` | Executes multiple DML statements as a single transaction. The method automatically wraps the statements with `BEGIN TRANSACTION;` and `END TRANSACTION;` — **do NOT include these directives** in your statement index. Much faster than individual `ExecDML` calls for bulk operations. |

**See Also:**
- [sqlite](#sqlite) — Main interface for opening databases
- [sqlitequery](#sqlitequery) — Query result navigation
- [sqlitetable](#sqlitetable) — Table result access

---

### Query DataTypes

#### **sqlitequery**

Represents the results of a SQL query, obtained from `sqlitedb.ExecQuery`. Allows stepping through result rows one at a time (callback-style) — more memory-efficient than loading the entire result set as a [sqlitetable](#sqlitetable).

**Members:**

| Member | Return Type | Description |
|--------|-------------|-------------|
| `ID` | int | Numeric identifier for this query |
| `NumRows` | int | Number of rows in the result set |
| `NumFields` | int | Number of fields (columns) in the result set |
| `GetFieldName[fieldnum]` | string | Name of the specified field (0-based index from SQLite) |
| `GetFieldIndex[fieldname]` | int | Numeric index of the named field (`-1` on error) |
| `GetFieldDeclType[fieldnum]` | string | Declared SQL type of the specified field |
| `GetFieldType[fieldnum]` | int | SQLite type code of the specified field (`-1` on error) |
| `GetFieldValue[field]` | string | Current-row value as a string (default) |
| `GetFieldValue[field, type]` | varies | Current-row value cast to `type`. `field` may be an index or name. See type list below. |
| `FieldIsNULL[field]` | bool | TRUE if the specified field in the current row is NULL |
| `LastRow` | bool | TRUE once the query has been stepped past the last row (see [LastRow Behavior](#lastrow-behavior)) |

Valid `type` values for `GetFieldValue`:

| Type | Result |
|------|--------|
| `string` | string (default if omitted) |
| `int` | int |
| `double` | double |
| `float` | double (identical to `double`) |
| `int64` | int64 |

**Methods:**

| Method | Description |
|--------|-------------|
| `NextRow` | Advances to the next row. When called past the last row, [LastRow](#lastrow-behavior) becomes TRUE. |
| `Reset` | Returns to the first row in the result set |
| `Finalize` | Finalizes and releases the query. Not strictly required, but failing to finalize when no longer needed causes memory leaks. |
| `Set[sqlitequery or id]` | Rebinds this variable to reference another query (by object or ID) |

**See Also:**
- [sqlitedb](#sqlitedb) — Database object that creates queries
- [sqlitetable](#sqlitetable) — Alternative for retrieving entire result sets at once

---

### Table DataTypes

#### **sqlitetable**

Represents an entire table (or query result) loaded into memory, obtained from `sqlitedb.GetTable`. Unlike [sqlitequery](#sqlitequery), all rows are allocated up-front for random access by row number.

**Members:**

| Member | Return Type | Description |
|--------|-------------|-------------|
| `ID` | int | Numeric identifier for this table |
| `NumRows` | int | Number of rows in the table |
| `NumFields` | int | Number of fields (columns) in the table |
| `GetFieldName[fieldnum]` | string | Name of the specified field |
| `GetFieldValue[field]` | string | Value of the specified field in the **current row** (see `SetRow`) as a string. See [sqlitetable.GetFieldValue Typed-Read Caveat](#sqlitetablegetfieldvalue-typed-read-caveat) before passing a `type` argument. |
| `GetFieldValue[field, type]` | varies | Same as above with a typed return. *See caveat linked above.* |
| `FieldIsNULL[field]` | bool | TRUE if the specified field in the current row is NULL |

Valid `type` values for `GetFieldValue`:

| Type | Result |
|------|--------|
| `string` | string (default if omitted) |
| `int` | int |
| `double` | double |
| `float` | double (identical to `double`) |
| `int64` | int64 |

**Methods:**

| Method | Description |
|--------|-------------|
| `SetRow[rownum]` | Sets the current row to `rownum` (0-based). All subsequent `GetFieldValue` / `FieldIsNULL` calls operate on this row. |
| `Finalize` | Finalizes and releases the table. Not strictly required, but failing to finalize causes memory leaks. |
| `Set[sqlitetable or id]` | Rebinds this variable to reference another table (by object or ID) |

**See Also:**
- [sqlitedb](#sqlitedb) — Database object that creates tables
- [sqlitequery](#sqlitequery) — Alternative for stepping through results one row at a time

---

## Commands

Commands extend the InnerSpace console with additional functionality. isxSQLite contributes the commands listed below.

### Test Commands

#### **sqlite**

**Syntax:** `sqlite`

**Description:** A test command used by the developers for runtime testing while the extension is loaded. The default implementation is a no-op. Do not rely on this command for any script behavior.

---

## Events

Events allow you to respond to isxSQLite operations and conditions. Register atoms with `Event[EventName]:AttachAtom[AtomName]`.

### Error Events

#### **isxSQLite_onErrorMsg**

**Description:** Fires when an error occurs during any isxSQLite operation (database open, DML execution, query field access, etc.).

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `ErrorNum` | int | Error number (`-1` if no unique numeric code is available) |
| `ErrorMsg` | string | Error message describing the failure |

**Notes:**
- Not all errors carry a unique `ErrorNum` — it may be `-1`.
- **Every** error has a unique `ErrorMsg`.
- This event fires regardless of `QuietMode`.

**Example:**
```lavishscript
atom(script) OnSQLiteError(int ErrorNum, string ErrorMsg)
{
    echo "SQLite Error ${ErrorNum}: ${ErrorMsg}"
}

Event[isxSQLite_onErrorMsg]:AttachAtom[OnSQLiteError]
```

---

### Status Events

#### **isxSQLite_onStatusMsg**

**Description:** Fires for non-error status messages (e.g., confirmations from `sqlitedb:Close`, `sqlitequery:Finalize`, `sqlitetable:Finalize`).

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `StatusMsg` | string | Status message text |

**Example:**
```lavishscript
atom(script) OnSQLiteStatus(string StatusMsg)
{
    echo "SQLite Status: ${StatusMsg}"
}

Event[isxSQLite_onStatusMsg]:AttachAtom[OnSQLiteStatus]
```

---

## Usage Examples

This section demonstrates common isxSQLite operations. Examples are adapted from the bundled `sqlite.iss` tutorial script.

### Checking Extension Status and Setting Up Events

Before using isxSQLite, verify it's loaded and ready, and set up event handlers.

```lavishscript
function main()
{
    ; Check if isxSQLite is loaded
    if (!${ISXSQLite(exists)})
    {
        echo "isxSQLite is not loaded!"
        return
    }

    ; Wait for extension to be ready
    if (!${ISXSQLite.IsReady})
    {
        echo "Waiting for isxSQLite to load..."
        do
        {
            waitframe
        }
        while !${ISXSQLite.IsReady}
    }

    ; Attach event handlers
    Event[isxSQLite_onErrorMsg]:AttachAtom[isxSQLite_onErrorMsg]
    Event[isxSQLite_onStatusMsg]:AttachAtom[isxSQLite_onStatusMsg]

    ; Enable quiet mode if you're handling all events programmatically
    if (!${ISXSQLite.InQuietMode})
        ISXSQLite:QuietMode

    echo "isxSQLite version ${ISXSQLite.Version} is ready!"

    ; ... your database operations here ...

    ; Detach events when done
    Event[isxSQLite_onErrorMsg]:DetachAtom[isxSQLite_onErrorMsg]
    Event[isxSQLite_onStatusMsg]:DetachAtom[isxSQLite_onStatusMsg]
}

; Event handlers
atom(script) isxSQLite_onErrorMsg(int ErrorNum, string ErrorMsg)
{
    if (${ErrorNum} > 0)
        echo "[sqlite] ERROR (${ErrorNum}): ${ErrorMsg}"
    else
        echo "[sqlite] ERROR: ${ErrorMsg}"
}

atom(script) isxSQLite_onStatusMsg(string StatusMsg)
{
    echo "[sqlite] STATUS: ${StatusMsg}"
}
```

### Opening and Closing Databases

```lavishscript
variable sqlitedb PlayerInfoDB

; Open or create a disk-based database
; File will be created in innerspace/Extensions directory
PlayerInfoDB:Set[${SQLite.OpenDB["PlayerInfoDB","PlayerInfoDB.sqlite3"]}]

; Always verify the database opened successfully
if (${PlayerInfoDB.ID(exists)})
{
    echo "Database '${PlayerInfoDB.ID}' opened successfully"
}
else
{
    echo "Failed to open database!"
    return
}

; You can create a copy of the database object using the ID
variable sqlitedb DBCopy = "PlayerInfoDB"
if (${DBCopy.ID(exists)})
    echo "Database copy created"

; Close the database when completely done
PlayerInfoDB:Close

; Note: When using subdirectories, the directory must already exist
; isxSQLite will NOT create it for you.
; MyDB:Set[${SQLite.OpenDB["MyDB","sqlite/PlayerInfoDB.sqlite3"]}]
```

### Creating Tables and Checking Schema

```lavishscript
variable sqlitedb PlayerInfoDB
PlayerInfoDB:Set[${SQLite.OpenDB["PlayerInfoDB","PlayerInfoDB.sqlite3"]}]

; Check if a table exists before creating it
if (!${PlayerInfoDB.TableExists["Amadeus_Friends"]})
{
    ; Create tables with proper schema
    PlayerInfoDB:ExecDML["create table Amadeus_Friends (key INTEGER PRIMARY KEY, name TEXT, level INTEGER, age REAL, notes TEXT);"]
    PlayerInfoDB:ExecDML["create table Amadeus_Inventory (key INTEGER PRIMARY KEY, name TEXT, mass REAL, value REAL);"]

    echo "Tables created"

    ; Use table name prefixes to keep names unique when sharing databases
    ; Example: "PlayerName_TableName" helps organize multiple users' data
}
else
{
    echo "Tables already exist, skipping creation"
}

; For a large number of table creation statements, consider ExecDMLTransaction
```

### Inserting Data with Transactions

Transactions are dramatically faster for bulk inserts than individual `ExecDML` calls.

```lavishscript
variable sqlitedb PlayerInfoDB
PlayerInfoDB:Set[${SQLite.OpenDB["PlayerInfoDB","PlayerInfoDB.sqlite3"]}]

; Build an index of INSERT statements
variable index:string DML
DML:Insert["insert into Amadeus_Friends (name,level,age,notes) values ('Cybertech', 100, 123543.25, 'CyberTech works on isxeve');"]
DML:Insert["insert into Amadeus_Friends (name,level,age,notes) values ('Lax', 13, 123.65, 'Lax owns Lavishsoft');"]
DML:Insert["insert into Amadeus_Friends (name,level,age,notes) values ('Doctor Who', 13, 995.1, 'The Doctor');"]

DML:Insert["insert into Amadeus_Inventory (name,mass,value) values ('iPad', 1.35, 0.053);"]
DML:Insert["insert into Amadeus_Inventory (name,mass,value) values ('Titan(EVE)',2278125000.0,70000000000.0);"]
DML:Insert["insert into Amadeus_Inventory (name,mass,value) values ('Public Opinion', 234234653245.254, 0.0);"]

; Execute all statements as a single transaction.
; isxSQLite automatically wraps with BEGIN TRANSACTION / END TRANSACTION.
PlayerInfoDB:ExecDMLTransaction[DML]

echo "Bulk insert completed via transaction"
```

### Inserting Data Individually

For small numbers of inserts (1-5 statements), `ExecDML` is fine.

```lavishscript
variable sqlitedb PlayerInfoDB
PlayerInfoDB:Set[${SQLite.OpenDB["PlayerInfoDB","PlayerInfoDB.sqlite3"]}]

PlayerInfoDB:ExecDML["insert into Amadeus_Friends (name,level,age,notes) values ('Cybertech', 100, 123543.25, 'CyberTech works on isxeve');"]
PlayerInfoDB:ExecDML["insert into Amadeus_Friends (name,level,age,notes) values ('Lax', 13, 123.65, 'Lax owns Lavishsoft');"]
PlayerInfoDB:ExecDML["insert into Amadeus_Friends (name,level,age,notes) values ('Doctor Who', 13, 995.1, 'The Doctor');"]

echo "Individual inserts completed"
```

### Querying Data with Do/While Loop

The most efficient way to iterate through query results.

```lavishscript
variable sqlitedb PlayerInfoDB
variable sqlitequery Level13Friends

PlayerInfoDB:Set[${SQLite.OpenDB["PlayerInfoDB","PlayerInfoDB.sqlite3"]}]

Level13Friends:Set[${PlayerInfoDB.ExecQuery["SELECT * FROM Amadeus_Friends where level=13;"]}]

if (${Level13Friends.NumRows} > 0)
{
    echo "Amadeus has ${Level13Friends.NumRows} friends who are level 13."

    do
    {
        echo "- ${Level13Friends.GetFieldValue["name",string]}"
        echo "  Age:   ${Level13Friends.GetFieldValue["age",double].Precision[2]}"
        echo "  Notes: '${Level13Friends.GetFieldValue["notes",string]}'"
        Level13Friends:NextRow
    }
    while (!${Level13Friends.LastRow})

    ; You can reset back to the first row if needed
    ; Level13Friends:Reset
}

; IMPORTANT: Always finalize queries when done to prevent memory leaks
Level13Friends:Finalize
```

### Querying Data with For Loop

Alternative iteration using a counter.

```lavishscript
variable sqlitedb PlayerInfoDB
variable sqlitequery Friends
variable int i

PlayerInfoDB:Set[${SQLite.OpenDB["PlayerInfoDB","PlayerInfoDB.sqlite3"]}]

Friends:Set[${PlayerInfoDB.ExecQuery["SELECT * FROM Amadeus_Friends ORDER BY level DESC"]}]

if (${Friends.NumRows} > 0)
{
    echo "Found ${Friends.NumRows} friends:"

    for (i:Set[1]; ${i} <= ${Friends.NumRows}; i:Inc)
    {
        echo "${i}. ${Friends.GetFieldValue["name"]} - Level ${Friends.GetFieldValue["level",int]}"

        ; Advance to the next row (except after the final row)
        if (${i} < ${Friends.NumRows})
            Friends:NextRow
    }
}

Friends:Finalize
```

### Exporting an Entire Table

Retrieve and display all rows and fields from a table.

```lavishscript
function SpewTable(string DatabaseID, string TableName)
{
    variable int fCount = 0
    variable int rCount = 0
    variable sqlitedb DB = ${DatabaseID}

    if (!${DB.ID(exists)})
    {
        echo "ERROR: Invalid Database ID (${DatabaseID})"
        return
    }

    if (!${DB.TableExists[${TableName}]})
    {
        echo "ERROR: Table '${TableName}' does not exist"
        return
    }

    declare Table sqlitetable ${DB.GetTable[${TableName}]}
    if (!${Table.ID(exists)})
    {
        echo "ERROR: Failed to retrieve table"
        return
    }

    echo "Exporting table '${TableName}' (${Table.NumRows} rows, ${Table.NumFields} fields):"

    if (${Table.NumRows} > 0)
    {
        for (rCount:Set[0]; ${rCount} < ${Table.NumRows}; rCount:Inc)
        {
            Table:SetRow[${rCount}]

            for (fCount:Set[0]; ${fCount} < ${Table.NumFields}; fCount:Inc)
            {
                if (${Table.FieldIsNULL[${fCount}]})
                    continue

                ; Displaying as string — no type argument required
                ; (see sqlitetable.GetFieldValue typed-read caveat in Notes)
                echo "  ${rCount}. ${Table.GetFieldName[${fCount}]}: ${Table.GetFieldValue[${fCount}]}"
            }
        }
    }

    Table:Finalize
}

; Usage:
; call SpewTable ${PlayerInfoDB.ID} "Amadeus_Inventory"
```

### Working with Memory Databases

```lavishscript
variable sqlitedb MemDB

; Create an in-memory database (no disk I/O)
; Data is lost when database is closed or extension is unloaded
MemDB:Set[${SQLite.OpenDB["TempDatabase",":Memory:"]}]

; Use exactly like a disk database
MemDB:ExecDML["CREATE TABLE temp_data (id INTEGER, value TEXT)"]
MemDB:ExecDML["INSERT INTO temp_data VALUES (1, 'temporary')"]
MemDB:ExecDML["INSERT INTO temp_data VALUES (2, 'session-only')"]

variable sqlitequery Query
Query:Set[${MemDB.ExecQuery["SELECT * FROM temp_data"]}]

echo "Memory database contents:"
do
{
    echo "ID: ${Query.GetFieldValue[1,int]} - Value: ${Query.GetFieldValue[2]}"
    Query:NextRow
}
while !${Query.LastRow}

Query:Finalize

MemDB:Close
echo "Memory database closed - all data is now gone"
```

### Using Escape_String for SQL Injection Prevention

```lavishscript
variable sqlitedb MyDB
variable string UserInput = "O'Brien"
variable string SafeInput

MyDB:Set[${SQLite.OpenDB["MyDB","MyDB.sqlite3"]}]

; Escape user input to defeat SQL injection
SafeInput:Set[${SQLite.Escape_String[${UserInput}]}]

; SafeInput now contains ''O''Brien'' — safe to embed in SQL
MyDB:ExecDML["INSERT INTO users (name) VALUES (${SafeInput})"]

; Example: "test" becomes ''test''
echo "Escaped 'test': ${SQLite.Escape_String["test"]}"
```

### Complete Production Script Example

A production-ready script demonstrating best practices.

```lavishscript
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;; PlayerDatabase.iss
;;; Demonstrates complete isxSQLite usage with error handling
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

variable sqlitedb PlayerInfoDB

atom(script) isxSQLite_onErrorMsg(int ErrorNum, string ErrorMsg)
{
    if (${ErrorNum} > 0)
        echo "[PlayerDB] ERROR (${ErrorNum}): ${ErrorMsg}"
    else
        echo "[PlayerDB] ERROR: ${ErrorMsg}"
}

atom(script) isxSQLite_onStatusMsg(string StatusMsg)
{
    echo "[PlayerDB] STATUS: ${StatusMsg}"
}

function main()
{
    if (!${ISXSQLite(exists)})
    {
        echo "[PlayerDB] isxSQLite is not loaded!"
        return
    }

    if (!${ISXSQLite.IsReady})
    {
        do
        {
            waitframe
        }
        while !${ISXSQLite.IsReady}
    }

    Event[isxSQLite_onErrorMsg]:AttachAtom[isxSQLite_onErrorMsg]
    Event[isxSQLite_onStatusMsg]:AttachAtom[isxSQLite_onStatusMsg]

    if (!${ISXSQLite.InQuietMode})
        ISXSQLite:QuietMode

    echo "[PlayerDB] Using isxSQLite version ${ISXSQLite.Version}"

    PlayerInfoDB:Set[${SQLite.OpenDB["PlayerInfoDB","PlayerInfoDB.sqlite3"]}]

    if (!${PlayerInfoDB.ID(exists)})
    {
        echo "[PlayerDB] Failed to open database!"
        return
    }

    call InitializeDatabase ${PlayerInfoDB.ID}
    call AddRecords ${PlayerInfoDB.ID}
    call DisplayFriends ${PlayerInfoDB.ID}
    call SpewTable ${PlayerInfoDB.ID} "Amadeus_Inventory"

    PlayerInfoDB:Close
    echo "[PlayerDB] Database closed"

    Event[isxSQLite_onErrorMsg]:DetachAtom[isxSQLite_onErrorMsg]
    Event[isxSQLite_onStatusMsg]:DetachAtom[isxSQLite_onStatusMsg]

    echo "[PlayerDB] Script complete"
}

function InitializeDatabase(string DatabaseID)
{
    variable sqlitedb DB = ${DatabaseID}

    if (!${DB.ID(exists)})
        return

    if (!${DB.TableExists["Amadeus_Friends"]})
    {
        DB:ExecDML["create table Amadeus_Friends (key INTEGER PRIMARY KEY, name TEXT, level INTEGER, age REAL, notes TEXT);"]
        DB:ExecDML["create table Amadeus_Inventory (key INTEGER PRIMARY KEY, name TEXT, mass REAL, value REAL);"]
        echo "[PlayerDB] Database schema created"
    }
}

function AddRecords(string DatabaseID)
{
    variable sqlitedb DB = ${DatabaseID}
    variable index:string DML

    if (!${DB.ID(exists)})
        return

    DML:Insert["insert into Amadeus_Friends (name,level,age,notes) values ('Cybertech', 100, 123543.25, 'CyberTech works on isxeve');"]
    DML:Insert["insert into Amadeus_Friends (name,level,age,notes) values ('Lax', 13, 123.65, 'Lax owns Lavishsoft');"]
    DML:Insert["insert into Amadeus_Friends (name,level,age,notes) values ('Doctor Who', 13, 995.1, 'The Doctor');"]

    DML:Insert["insert into Amadeus_Inventory (name,mass,value) values ('iPad', 1.35, 0.053);"]
    DML:Insert["insert into Amadeus_Inventory (name,mass,value) values ('Titan(EVE)',2278125000.0,70000000000.0);"]
    DML:Insert["insert into Amadeus_Inventory (name,mass,value) values ('Public Opinion', 234234653245.254, 0.0);"]

    DB:ExecDMLTransaction[DML]
    echo "[PlayerDB] Records added via transaction"
}

function DisplayFriends(string DatabaseID)
{
    variable sqlitedb DB = ${DatabaseID}
    variable sqlitequery Query

    if (!${DB.ID(exists)})
        return

    Query:Set[${DB.ExecQuery["SELECT * FROM Amadeus_Friends where level=13;"]}]

    if (${Query.NumRows} > 0)
    {
        echo "[PlayerDB] Level 13 Friends (${Query.NumRows} found):"
        do
        {
            echo "  - ${Query.GetFieldValue["name",string]}"
            echo "    Age:   ${Query.GetFieldValue["age",double].Precision[2]}"
            echo "    Notes: ${Query.GetFieldValue["notes",string]}"
            Query:NextRow
        }
        while (!${Query.LastRow})
    }

    Query:Finalize
}

function SpewTable(string DatabaseID, string TableName)
{
    variable int fCount = 0
    variable int rCount = 0
    variable sqlitedb DB = ${DatabaseID}

    if (!${DB.ID(exists)} || !${DB.TableExists[${TableName}]})
        return

    declare Table sqlitetable ${DB.GetTable[${TableName}]}
    if (!${Table.ID(exists)})
        return

    echo "[PlayerDB] Table '${TableName}' contents:"

    if (${Table.NumRows} > 0)
    {
        for (rCount:Set[0]; ${rCount} < ${Table.NumRows}; rCount:Inc)
        {
            Table:SetRow[${rCount}]
            for (fCount:Set[0]; ${fCount} < ${Table.NumFields}; fCount:Inc)
            {
                if (!${Table.FieldIsNULL[${fCount}]})
                    echo "  ${Table.GetFieldName[${fCount}]}: ${Table.GetFieldValue[${fCount}]}"
            }
        }
    }

    Table:Finalize
}
```

---

## Notes

### Case Sensitivity

LavishScript (and therefore isxSQLite) member and method names are **NOT** case-sensitive. SQL statements and table/column names may or may not be case-sensitive depending on the SQL syntax and SQLite pragmas used.

```lavishscript
${SQLite.OpenDB["db","file.db"]}  ; Works
${sqlite.opendb["db","file.db"]}  ; Also works — identical result
```

### NULL Checks

Always check that database objects, queries, and tables exist before using them:

```lavishscript
if ${MyDB.ID(exists)}
{
    ; Safe to use MyDB
}

if ${MyQuery.ID}
{
    ; Query returned results
}

; Check for NULL field values
if ${MyQuery.FieldIsNULL["optional_field"]}
{
    ; Handle NULL value
}
```

### Parameter Notation

In this documentation:
- `[parameter]` indicates a required parameter
- `[param1,param2]` indicates multiple required parameters
- `field` in `GetFieldValue[field]` accepts either a numeric index or a field name

### Resource Management

Always finalize query and table objects when done to avoid memory leaks:

```lavishscript
; Queries
MyQuery:Set[${DB.ExecQuery["SELECT * FROM table"]}]
; ... use query ...
MyQuery:Finalize

; Tables
MyTable:Set[${DB.GetTable["table"]}]
; ... use table ...
MyTable:Finalize
```

Queries and tables that return no results or that encounter errors finalize automatically — manual `Finalize` is only needed for successful non-empty results.

### Database Defaults

When creating a new database file, isxSQLite applies these PRAGMA defaults (note: `synchronous` is set twice — second value wins):

```sql
PRAGMA encoding      = 'UTF-8';
PRAGMA auto_vacuum   = 1;
PRAGMA cache_size    = 2048;
PRAGMA page_size     = 4096;
PRAGMA synchronous   = NORMAL;
PRAGMA journal_mode  = OFF;
PRAGMA temp_store    = MEMORY;
PRAGMA synchronous   = OFF;
```

You can override any of these with `ExecDML` after opening:

```lavishscript
MyDB:ExecDML["PRAGMA synchronous = FULL"]
```

### Events and Quiet Mode

When `QuietMode` is enabled, console messages are suppressed but events still fire. This is useful for production scripts that handle errors programmatically:

```lavishscript
ISXSQLite:QuietMode

atom(script) OnError(int ErrorNum, string ErrorMsg)
{
    ; Custom error handling
    Logger:Log["SQLite Error: ${ErrorMsg}"]
}
Event[isxSQLite_onErrorMsg]:AttachAtom[OnError]
```

### Query vs Table Usage

**Use `ExecQuery` (returns [sqlitequery](#sqlitequery)) when:**
- Result sets may be large
- You can process rows sequentially
- Memory efficiency matters

**Use `GetTable` (returns [sqlitetable](#sqlitetable)) when:**
- Result set is small
- You need random access to any row by number
- Convenience outweighs memory cost

**Key difference:** `ExecQuery` steps through rows one at a time via SQLite callbacks. `GetTable` allocates space for the entire result set up front.

### LastRow Behavior

The changelog refers to `LastRow` as belonging to `sqlitedb`, but it is actually a member of [sqlitequery](#sqlitequery). The flag semantics:

1. After you step past the last row (`NextRow` when already on the final row), SQLite reports no more rows.
2. isxSQLite sets `LastRow` to TRUE and returns a blank row.

This is why the canonical do/while pattern works:

```lavishscript
do
{
    ; Process current row
    Query:NextRow
}
while !${Query.LastRow}
```

The loop processes each row, advances, and exits when `LastRow` becomes TRUE after the final `NextRow`.

### ExecDMLTransaction Details

`ExecDMLTransaction` automatically wraps your statement list:

```lavishscript
; Your code
MyDB:ExecDMLTransaction[Statements]

; What isxSQLite actually executes
; BEGIN TRANSACTION;
; ... your statements ...
; END TRANSACTION;
```

**Do NOT** include `BEGIN TRANSACTION` / `END TRANSACTION` in your statement index — the method handles them.

The method also requires the argument be an `index:string` (or `index:mutablestring`). Passing any other type fires [isxSQLite_onErrorMsg](#isxsqlite_onerrormsg).

### File Paths

The base path for database files is your `innerspace/Extensions` directory. Relative and absolute paths both work:

```lavishscript
; Relative to innerspace/Extensions
${SQLite.OpenDB["db","./mydata.db"]}
${SQLite.OpenDB["db","../databases/mydata.db"]}

; Absolute path
${SQLite.OpenDB["db","C:/MyDatabases/mydata.db"]}
```

Subdirectories must already exist — isxSQLite does not create them.

### Database Name Uniqueness

Database names (first parameter to `OpenDB`) must be unique for the active isxSQLite session. Opening a second database with a name that is already in use fires [isxSQLite_onErrorMsg](#isxsqlite_onerrormsg) and returns a NULL `sqlitedb`.

```lavishscript
; First database
DB1:Set[${SQLite.OpenDB["MyDB","./file1.db"]}]

; Second database must use a different name
DB2:Set[${SQLite.OpenDB["MyDB2","./file2.db"]}]   ; OK

; This will fail — name already in use
; DB3:Set[${SQLite.OpenDB["MyDB","./file3.db"]}]
```

### sqlitetable.GetFieldValue Typed-Read Caveat

The typed form `GetFieldValue[field, type]` is reliable on [sqlitequery](#sqlitequery) but has a known implementation quirk on [sqlitetable](#sqlitetable):

- On `sqlitequery`, the second argument (`argv[1]`) is correctly parsed as the type selector.
- On `sqlitetable`, the type-selector parsing reads `argv[0]` rather than `argv[1]`, which means the field-selector and the type-selector compete for the same argument slot. In addition, the cast branches in the table implementation fall through without `break` statements.

**Practical guidance:**
- Prefer `sqlitetable.GetFieldValue[field]` (no type) and cast on the LavishScript side: `${Table.GetFieldValue[amount].Int}`, `${Table.GetFieldValue[price].Float}`, etc.
- When you need typed numeric access to a large result set, use [sqlitequery](#sqlitequery) rather than [sqlitetable](#sqlitetable) — the typed form is well-behaved there.
- If you do need typed table reads, test thoroughly against your specific isxSQLite build before relying on the result.

This caveat is based on direct source review; the behavior has not been formally documented in [isxSQLiteChanges.txt](https://github.com/isxGames/isxSQLite/blob/master/isxSQLiteChanges.txt).
