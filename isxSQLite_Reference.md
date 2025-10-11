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
   - [Web Commands](#web-commands)
5. [Events](#events)
   - [Error Events](#error-events)
   - [Status Events](#status-events)
   - [HTTP Events](#http-events)
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
- HTTP/HTTPS request capabilities (optional)
- Event-driven error and status reporting

### Accessing DataTypes

DataTypes are accessed through Top-Level Objects (TLOs) or through members of other datatypes. For example:

```lavishscript
${ISXSQLite}                           // TLO returning 'isxsqlite' datatype
${SQLite}                              // TLO returning 'sqlite' datatype
${SQLite.OpenDB["MyDB","./myfile.db"]} // Member returning 'sqlitedb' datatype
${MyDB.ExecQuery["SELECT * FROM users"]} // Member returning 'sqlitequery' datatype
```

---

## Top-Level Objects

Top-Level Objects (TLOs) are the entry points for accessing isxSQLite functionality. They can be accessed directly in LavishScript.

### Extension TLOs

| TLO | DataType | Description |
|-----|----------|-------------|
| **ISXSQLite** | [isxsqlite](#isxsqlite) | isxSQLite extension information and utilities |

### Core TLOs

| TLO | DataType | Description |
|-----|----------|-------------|
| **SQLite** | [sqlite](#sqlite) | Main SQLite interface for database operations |

---

## DataTypes

### Core/Extension DataTypes

#### **isxsqlite**

Extension information and utilities. Accessed via ISXSQLite TLO.

**Members:**
- `Version` - string: isxSQLite version number
- `IsReady` - bool: Whether extension is ready for use
- `IsLoading` - bool: Whether extension is currently loading
- `InQuietMode` - bool: Whether quiet mode is active (no console messages)

**Methods:**
- `QuietMode` - Toggles quiet mode on/off. When quiet mode is active, no error messages or method results are displayed in the console. However, all messages will still be available via applicable events.

**See Also:**
- [sqlite](#sqlite) - Main SQLite interface

---

#### **sqlite**

Main SQLite datatype providing database management functionality. Accessed via SQLite TLO.

**Members:**
- `OpenDB[name,path]` - [sqlitedb](#sqlitedb): Opens or creates a database. The first parameter is a unique name for this session, and the second parameter is the file path. Use ":Memory:" as the path to create an in-memory database. The base path is your "innerspace/Extensions" directory. If the database file does not exist, it will be created with default PRAGMA settings.
- `GetQueryByID[id]` - [sqlitequery](#sqlitequery): Retrieves a query object by its numeric ID
- `Escape_String[text]` - string: Returns an escaped string suitable for SQL queries (similar to sqlite3_mprintf("%q",text) or PHP's sqlite_escape_string()). For example, 'test' becomes ''test''

**See Also:**
- [sqlitedb](#sqlitedb) - Database object
- [sqlitequery](#sqlitequery) - Query result object
- [sqlitetable](#sqlitetable) - Table result object

---

### Database DataTypes

#### **sqlitedb**

Represents an open SQLite database connection. Obtained from the [sqlite](#sqlite) datatype's OpenDB member.

**Members:**
- `ExecQuery[statement]` - [sqlitequery](#sqlitequery): Executes a SQL query statement and returns a query object for navigating results. Use this for SELECT statements and other queries that return data. Queries that return no results or result in an error finalize automatically.
- `TableExists[tablename]` - bool: Returns true if the specified table exists in the database
- `GetTable[tablename]` - [sqlitetable](#sqlitetable): Retrieves an entire table as a table object. Uses the hardcoded statement "SELECT * from TableName order by 1;". For partial table retrieval, use GetTable with a custom DML statement.
- `GetTable[statement]` - [sqlitetable](#sqlitetable): Executes a custom SQL statement and returns the entire result set as a table object. isxSQLite assumes you're using a custom DML statement if the argument doesn't exactly match an existing table name. Note: GetTable mallocs space for entire results, which can use significant memory for large queries. Be sure to Finalize table objects when done.
- `ID` - string: The unique identifier name for this database

**Methods:**
- `Set[sqlitedb or id]` - Sets this variable to reference another database object (by object or ID)
- `Close` - Closes the database connection
- `ExecDML[statement]` - Executes a Data Manipulation Language (DML) statement that doesn't require a return value (e.g., INSERT, UPDATE, DELETE, CREATE TABLE). The method succeeds silently unless an error occurs, which fires an isxSQLite_onErrorMsg event (and shows a console message unless QuietMode is active).
- `ExecDMLTransaction[index:string]` - Executes multiple DML statements as a single transaction for improved performance. Automatically wraps the statements with "BEGIN TRANSACTION;" and "END TRANSACTION;" directives. Do NOT include these directives in your statement index. This is much more efficient than executing statements individually for bulk operations.

**See Also:**
- [sqlite](#sqlite) - Main interface for opening databases
- [sqlitequery](#sqlitequery) - Query result navigation
- [sqlitetable](#sqlitetable) - Table result access

---

### Query DataTypes

#### **sqlitequery**

Represents the results of a SQL query. Obtained from the [sqlitedb](#sqlitedb) datatype's ExecQuery member. Allows stepping through result rows one at a time.

**Members:**
- `ID` - int: Numeric identifier for this query
- `NumRows` - int: Number of rows in the result set
- `NumFields` - int: Number of fields (columns) in the result set
- `GetFieldName[fieldnum]` - string: Returns the name of the specified field (1-based index)
- `GetFieldIndex[fieldname]` - int: Returns the numeric index of the named field
- `GetFieldDeclType[fieldnum]` - string: Returns the declared type of the specified field
- `GetFieldType[fieldnum]` - int: Returns the type code of the specified field
- `GetFieldValue[fieldnum or fieldname]` - string: Returns the value of the specified field as a string (default)
- `GetFieldValue[fieldnum or fieldname, type]` - varies: Returns the value of the specified field as the specified type. Valid types: 'string', 'int', 'double', 'float' (same as double), 'int64'
- `FieldIsNULL[fieldnum or fieldname]` - bool: Returns true if the specified field contains NULL
- `LastRow` - bool: Returns true when at the last row. Note: NextRow must be called to move past the last row, at which point LastRow becomes true and a blank row is returned.

**Methods:**
- `NextRow` - Advances to the next row in the result set
- `Reset` - Returns to the first row in the result set
- `Finalize` - Finalizes and releases the query object. While not strictly required, failing to finalize queries when no longer needed will cause memory leaks.
- `Set[sqlitequery or id]` - Sets this variable to reference another query object (by object or ID)

**See Also:**
- [sqlitedb](#sqlitedb) - Database object that creates queries
- [sqlitetable](#sqlitetable) - Alternative for retrieving entire result sets at once

---

### Table DataTypes

#### **sqlitetable**

Represents an entire table or query result set loaded into memory. Obtained from the [sqlitedb](#sqlitedb) datatype's GetTable member. Unlike queries, tables load all results at once.

**Members:**
- `ID` - int: Numeric identifier for this table
- `NumRows` - int: Number of rows in the table
- `NumFields` - int: Number of fields (columns) in the table
- `GetFieldName[fieldnum]` - string: Returns the name of the specified field (1-based index)
- `GetFieldValue[fieldnum or fieldname]` - string: Returns the value of the specified field in the current row as a string (default)
- `GetFieldValue[fieldnum or fieldname, type]` - varies: Returns the value of the specified field in the current row as the specified type. Valid types: 'string', 'int', 'double', 'float' (same as double), 'int64'
- `FieldIsNULL[fieldnum or fieldname]` - bool: Returns true if the specified field in the current row contains NULL

**Methods:**
- `SetRow[rownum]` - Sets the current row to the specified row number (1-based index)
- `Finalize` - Finalizes and releases the table object. While not strictly required, failing to finalize tables when no longer needed will cause memory leaks.
- `Set[sqlitetable or id]` - Sets this variable to reference another table object (by object or ID)

**See Also:**
- [sqlitedb](#sqlitedb) - Database object that creates tables
- [sqlitequery](#sqlitequery) - Alternative for navigating results one row at a time

---

## Commands

Commands extend the InnerSpace console with additional functionality.

### Test Commands

#### **sqlite**

**Syntax:** `sqlite`

**Description:** A test command for development and debugging purposes. This command is used for testing isxSQLite functionality during development.

---

### Web Commands

These commands are only available when isxSQLite is compiled with the USE_LIBISXGAMES flag (libisxGames integration).

#### **GetURL**

**Syntax:** `GetURL "http://address/to/file"`
**Syntax:** `GetURL "https://address/to/file"`

**Description:** Retrieves content from a URL using HTTP or HTTPS. This command executes asynchronously using multiple threads, so your script will not wait or freeze after issuing the command. Multiple GetURL commands can be issued at once and will queue in the order received.

**Parameters:**
- `url` - The HTTP or HTTPS URL to retrieve. Any normalized URL should work regardless of file type.

**Notes:**
- HTTPS requests typically return one response
- HTTP requests (when response is larger than ~3000 bytes) may be split into multiple responses
- HTTPS requests are slower than HTTP requests
- Only HTTP and HTTPS protocols are supported
- Responses are accessible via the isxGames_onHTTPResponse event

**Examples:**
```lavishscript
GetURL "https://www.isxgames.com/libcurltest.html"
GetURL "http://www.isxgames.com/libcurltest.html"
```

**See Also:**
- [isxGames_onHTTPResponse](#isxgames_onhttpresponse) - Event for handling URL responses

---

#### **DebugSpew**

**Syntax:** `DebugSpew`

**Description:** A debug command for development purposes. Only available when compiled with USE_LIBISXGAMES flag.

---

## Events

Events allow you to respond to various isxSQLite operations and conditions.

### Error Events

#### **isxSQLite_onErrorMsg**

**Description:** Fires when an error occurs during database operations.

**Parameters:**
- `ErrorNum` - int: Error number (may be -1 if no unique number is available)
- `ErrorMsg` - string: Error message describing what went wrong

**Notes:**
- Not all errors have a unique ErrorNum. In these cases, it will be -1
- ALL errors have a unique ErrorMsg
- This event fires even when QuietMode is active

**Example:**
```lavishscript
function OnSQLiteError(int ErrorNum, string ErrorMsg)
{
    echo "SQLite Error ${ErrorNum}: ${ErrorMsg}"
}

Event[isxSQLite_onErrorMsg]:AttachAtom[OnSQLiteError]
```

---

### Status Events

#### **isxSQLite_onStatusMsg**

**Description:** Fires when isxSQLite generates status messages that are not errors (e.g., results from database operations like Close).

**Parameters:**
- `StatusMsg` - string: Status message text

**Example:**
```lavishscript
function OnSQLiteStatus(string StatusMsg)
{
    echo "SQLite Status: ${StatusMsg}"
}

Event[isxSQLite_onStatusMsg]:AttachAtom[OnSQLiteStatus]
```

---

### HTTP Events

These events are only available when isxSQLite is compiled with the USE_LIBISXGAMES flag (libisxGames integration).

#### **isxGames_onHTTPResponse**

**Description:** Fires when a response is received from a GetURL command.

**Parameters:**
- `Size` - int: Size of the response in bytes
- `URL` - string: The URL that was requested (may be modified by server redirects)
- `IPAddress` - string: IP address of the responding server
- `ResponseCode` - int: HTTP response code (e.g., 200 for success)
- `TransferTime` - float: Time taken for the transfer in seconds
- `ResponseText` - string: The entire response text unparsed (including all HTML/XML tags)
- `ParsedBody` - string: Plain text extracted from the &lt;body&gt; section of HTML documents (only works for simple documents with plain text between &lt;body&gt;&lt;/body&gt; tags)

**Notes:**
- Only fires for responses from GetURL commands
- ParsedBody parsing is currently limited to simple HTML documents

**Example:**
```lavishscript
function OnHTTPResponse(int Size, string URL, string IPAddress, int ResponseCode, float TransferTime, string ResponseText, string ParsedBody)
{
    echo "Received ${Size} bytes from ${URL}"
    echo "Response Code: ${ResponseCode}"
    echo "Transfer Time: ${TransferTime}s"
    echo "Body: ${ParsedBody}"
}

Event[isxGames_onHTTPResponse]:AttachAtom[OnHTTPResponse]
```

**See Also:**
- [GetURL](#geturl) - Command to initiate HTTP requests

---

## Usage Examples

This section provides comprehensive examples demonstrating common isxSQLite operations. These examples are based on the official isxSQLite tutorial script.

### Checking Extension Status and Setting Up Events

Before using isxSQLite, verify it's loaded and ready, and set up event handlers for error and status messages.

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
    ; Note: Not all errors have unique ErrorNum (may be -1)
    ; But all errors have unique ErrorMsg
    if (${ErrorNum} > 0)
        echo "[sqlite] ERROR (${ErrorNum}): ${ErrorMsg}"
    else
        echo "[sqlite] ERROR: ${ErrorMsg}"
}

atom(script) isxSQLite_onStatusMsg(string StatusMsg)
{
    ; Status messages are typically for debugging
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

; Note: Databases using subdirectories
; MyDB:Set[${SQLite.OpenDB["MyDB","sqlite/PlayerInfoDB.sqlite3"]}]
; Make sure the directory exists - isxSQLite won't create it for you
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

    ; Note: Use table name prefixes to keep names unique when sharing databases
    ; Example: "PlayerName_TableName" helps organize multiple users' data
}
else
{
    echo "Tables already exist, skipping creation"
}

; For a large number of table creation statements, consider using ExecDMLTransaction
```

### Inserting Data with Transactions

Transactions are much faster for bulk inserts than individual ExecDML calls.

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

; Execute all statements as a single transaction
; isxSQLite automatically wraps with BEGIN TRANSACTION and END TRANSACTION
PlayerInfoDB:ExecDMLTransaction[DML]

echo "Bulk insert completed via transaction"
```

### Inserting Data Individually

For smaller numbers of inserts (1-5 statements), ExecDML is fine.

```lavishscript
variable sqlitedb PlayerInfoDB
PlayerInfoDB:Set[${SQLite.OpenDB["PlayerInfoDB","PlayerInfoDB.sqlite3"]}]

; Single INSERT statements
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

; Execute a SELECT query with WHERE clause
Level13Friends:Set[${PlayerInfoDB.ExecQuery["SELECT * FROM Amadeus_Friends where level=13;"]}]

; Check if query returned any rows
if (${Level13Friends.NumRows} > 0)
{
    echo "Amadeus has ${Level13Friends.NumRows} friends who are level 13."

    ; Iterate through results using do/while pattern
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

Alternative iteration method using a counter.

```lavishscript
variable sqlitedb PlayerInfoDB
variable sqlitequery Friends
variable int i

PlayerInfoDB:Set[${SQLite.OpenDB["PlayerInfoDB","PlayerInfoDB.sqlite3"]}]

Friends:Set[${PlayerInfoDB.ExecQuery["SELECT * FROM Amadeus_Friends ORDER BY level DESC"]}]

if (${Friends.NumRows} > 0)
{
    echo "Found ${Friends.NumRows} friends:"

    ; Loop using counter (less efficient than do/while)
    for (i:Set[1]; ${i} <= ${Friends.NumRows}; i:Inc)
    {
        echo "${i}. ${Friends.GetFieldValue["name"]} - Level ${Friends.GetFieldValue["level",int]}"

        ; Must call NextRow to advance (except on first row)
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

    ; Verify database is valid
    if (!${DB.ID(exists)})
    {
        echo "ERROR: Invalid Database ID (${DatabaseID})"
        return
    }

    ; Verify table exists
    if (!${DB.TableExists[${TableName}]})
    {
        echo "ERROR: Table '${TableName}' does not exist"
        return
    }

    ; Get the entire table
    declare Table sqlitetable ${DB.GetTable[${TableName}]}
    if (!${Table.ID(exists)})
    {
        echo "ERROR: Failed to retrieve table"
        return
    }

    echo "Exporting table '${TableName}' (${Table.NumRows} rows, ${Table.NumFields} fields):"

    ; Loop through all rows
    if (${Table.NumRows} > 0)
    {
        for (rCount:Set[0]; ${rCount} < ${Table.NumRows}; rCount:Inc)
        {
            Table:SetRow[${rCount}]

            ; Loop through all fields in current row
            for (fCount:Set[0]; ${fCount} < ${Table.NumFields}; fCount:Inc)
            {
                ; Skip NULL fields
                if (${Table.FieldIsNULL[${fCount}]})
                    continue

                ; Display field name and value
                ; No type argument needed since we're just displaying as strings
                echo "  ${rCount}. ${Table.GetFieldName[${fCount}]}: ${Table.GetFieldValue[${fCount}]}"
            }
        }
    }

    ; IMPORTANT: Finalize to prevent memory leaks
    Table:Finalize
}

; Usage example:
; call SpewTable ${PlayerInfoDB.ID} "Amadeus_Inventory"
```

### Working with Memory Databases

```lavishscript
variable sqlitedb MemDB

; Create an in-memory database (no disk I/O)
; Data is lost when database is closed or extension is unloaded
MemDB:Set[${SQLite.OpenDB["TempDatabase",":Memory:"]}]

; Use just like a disk database
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

; Database is lost when closed
MemDB:Close
echo "Memory database closed - all data is now gone"
```

### Using Escape_String for SQL Injection Prevention

```lavishscript
variable sqlitedb MyDB
variable string UserInput = "O'Brien"
variable string SafeInput

MyDB:Set[${SQLite.OpenDB["MyDB","MyDB.sqlite3"]}]

; Escape user input to prevent SQL injection
SafeInput:Set[${SQLite.Escape_String[${UserInput}]}]

; SafeInput now contains ''O''Brien'' which is safe for SQL
MyDB:ExecDML["INSERT INTO users (name) VALUES (${SafeInput})"]

; Example: "test" becomes ''test''
echo "Escaped 'test': ${SQLite.Escape_String["test"]}"
```

### Complete Production Script Example

A complete, production-ready script demonstrating best practices.

```lavishscript
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;; PlayerDatabase.iss
;;; Demonstrates complete isxSQLite usage with error handling
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

variable sqlitedb PlayerInfoDB

; Event handlers
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
    ; Verify extension is loaded
    if (!${ISXSQLite(exists)})
    {
        echo "[PlayerDB] isxSQLite is not loaded!"
        return
    }

    ; Wait for extension to be ready
    if (!${ISXSQLite.IsReady})
    {
        do
        {
            waitframe
        }
        while !${ISXSQLite.IsReady}
    }

    ; Attach events
    Event[isxSQLite_onErrorMsg]:AttachAtom[isxSQLite_onErrorMsg]
    Event[isxSQLite_onStatusMsg]:AttachAtom[isxSQLite_onStatusMsg]

    ; Enable quiet mode (handle all output via events)
    if (!${ISXSQLite.InQuietMode})
        ISXSQLite:QuietMode

    echo "[PlayerDB] Using isxSQLite version ${ISXSQLite.Version}"

    ; Open database
    PlayerInfoDB:Set[${SQLite.OpenDB["PlayerInfoDB","PlayerInfoDB.sqlite3"]}]

    if (!${PlayerInfoDB.ID(exists)})
    {
        echo "[PlayerDB] Failed to open database!"
        return
    }

    ; Initialize schema if needed
    call InitializeDatabase ${PlayerInfoDB.ID}

    ; Add sample data
    call AddRecords ${PlayerInfoDB.ID}

    ; Query and display data
    call DisplayFriends ${PlayerInfoDB.ID}

    ; Export inventory table
    call SpewTable ${PlayerInfoDB.ID} "Amadeus_Inventory"

    ; Close database
    PlayerInfoDB:Close
    echo "[PlayerDB] Database closed"

    ; Detach events
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

LavishScript and isxSQLite member/method names are **NOT** case-sensitive. However, SQL statements and table/column names may be case-sensitive depending on your SQL syntax and SQLite configuration.

**Examples:**
```lavishscript
${SQLite.OpenDB["db","file.db"]}  ; Works
${sqlite.opendb["db","file.db"]}  ; Also works (same result)
```

### NULL Checks

Always check if database objects, queries, and tables exist before using them:

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
- Text in italics describes the parameter type or purpose

**Examples:**
- `OpenDB[name,path]` requires two string parameters
- `GetFieldValue[fieldnum or fieldname]` accepts either a number or string
- `GetFieldValue[field, type]` optional second parameter specifies return type

### Resource Management

**IMPORTANT:** Always finalize query and table objects when done to prevent memory leaks:

```lavishscript
; Queries
MyQuery:Set[${DB.ExecQuery["SELECT * FROM table"]}]
; ... use query ...
MyQuery:Finalize  ; Release memory

; Tables
MyTable:Set[${DB.GetTable["table"]}]
; ... use table ...
MyTable:Finalize  ; Release memory
```

**Note:** Queries and tables that return no results or encounter errors finalize automatically.

### Database Defaults

When creating a new database file, isxSQLite automatically sets these PRAGMA defaults:

```sql
PRAGMA encoding = 'UTF-8';
PRAGMA auto_vacuum = 1;
PRAGMA cache_size = 2048;
PRAGMA page_size = 4096;
PRAGMA synchronous = NORMAL;
PRAGMA journal_mode = OFF;
PRAGMA temp_store = MEMORY;
PRAGMA synchronous = OFF;
```

You can override these settings using the ExecDML method after opening the database:

```lavishscript
MyDB:ExecDML["PRAGMA synchronous = FULL"]
```

### Events and Quiet Mode

When QuietMode is enabled, console messages are suppressed, but events still fire. This is useful for production scripts that handle errors programmatically:

```lavishscript
; Enable quiet mode
ISXSQLite:QuietMode

; Attach error handler
function OnError(int ErrorNum, string ErrorMsg)
{
    ; Custom error handling
    Logger:Log["SQLite Error: ${ErrorMsg}"]
}
Event[isxSQLite_onErrorMsg]:AttachAtom[OnError]

; Now errors won't spam console but will be logged
```

### Query vs Table Usage

**Use ExecQuery when:**
- Working with large result sets
- You need to process rows sequentially
- Memory efficiency is important
- You want to navigate forward/backward through results

**Use GetTable when:**
- Result set is small
- You need random access to any row
- You want all data loaded at once
- Convenience outweighs memory concerns

**Key Difference:** ExecQuery uses callbacks for each row (step-through), while GetTable mallocs space for the entire result set.

### LastRow Behavior

The `LastRow` member of [sqlitequery](#sqlitequery) can be confusing. It works as follows:

1. When you reach the last row and call `NextRow`, SQLite returns "no more rows"
2. isxSQLite sets the `LastRow` flag to true
3. A blank row is returned

This is why the do/while loop pattern works:
```lavishscript
do
{
    ; Process current row
    Query:NextRow
}
while !${Query.LastRow}
```

The loop processes each row, calls NextRow, and exits when LastRow becomes true.

### ExecDMLTransaction Details

When using `ExecDMLTransaction`, isxSQLite automatically wraps your statements:

```lavishscript
; Your code:
MyDB:ExecDMLTransaction[Statements]

; What isxSQLite executes:
; BEGIN TRANSACTION;
; ... your statements ...
; END TRANSACTION;
```

**Do NOT** include BEGIN/END TRANSACTION in your statement index. The method handles this automatically.

### File Paths

The base path for database files is your `innerspace/Extensions` directory. You can use relative or absolute paths:

```lavishscript
; Relative to innerspace/Extensions
${SQLite.OpenDB["db","./mydata.db"]}
${SQLite.OpenDB["db","../databases/mydata.db"]}

; Absolute path
${SQLite.OpenDB["db","C:/MyDatabases/mydata.db"]}
```

### Database Name Uniqueness

Database names (first parameter to OpenDB) must be unique for each isxSQLite session. You cannot have two databases with the same name open simultaneously.

```lavishscript
; First database
DB1:Set[${SQLite.OpenDB["MyDB","./file1.db"]}]

; Second database - must use different name
DB2:Set[${SQLite.OpenDB["MyDB2","./file2.db"]}]  ; ✓ Correct

; This would fail or cause conflicts:
; DB3:Set[${SQLite.OpenDB["MyDB","./file3.db"]}]  ; ✗ Name already used
```

### Version Information

This reference is based on isxSQLite version 20200812 (August 12, 2020), which includes:
- SQLite 3.32.3
- Visual Studio 2019 with v140 toolchain
- Fixed x64 build support
