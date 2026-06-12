# Go JSON to SQLite Importer

### A lightweight, server-side Go application that reads nested JSON data (e.g., RFID reader bundles

### and associated item reads), parses it, and safely stores it into a relational SQLite database.

This project uses a pure Go SQLite driver (modernc.org/sqlite), meaning it compiles and runs flawlessly
on any operating system (Windows, macOS, Linux) without requiring CGO, GCC, or any external C compilers.

## Features

- Zero-Dependency SQLite: Uses modernc.org/sqlite for a seamless, cross-platform database
    experience.
- Relational Data Mapping: Automatically splits nested JSON arrays into normalized, relational database
    tables (proto_reader_bundles and proto_reads) linked via Foreign Keys.
- High Performance: Uses SQL transactions and prepared statements for rapid, bulk data insertion.
- Dual Logging: Implements io.MultiWriter to stream application logs to both the standard console
    output and an app.log file simultaneously.

## Prerequisites

- Go (version 1.18 or higher recommended)

## Project Structure

.nnn main.go # The main Go application code
nnnnnn data.json # The source JSON file containing the nested read data go.mod # Go module file defining project properties
nnnnnn go.sum # Auto-generated lock file for dependencies app.log # (Generated) Log file capturing all import activities
nnn reader_data.db # (Generated) The SQLite database containing the parsed data


## Setup & Installation

1. Initialize the Go module
If you haven't already initialized your module, run:
go mod init go-sqlite-demo
2. Download the required dependencies
This command will fetch the pure Go SQLite driver and clean up your go.mod file.
go mod tidy

## Usage

1. Ensure your data.json file is in the same directory as main.go.
2. Run the application:
go run main.go

### Expected Output

The application will process the JSON file and output its progress to the console (and save it to app.log).
--- Starting Go JSON-to-SQLite Import Process ---INFO: Loaded 3 ProtoReaderBundles from JSON.
INFO: Database schema established.INFO: Inserting Bundle ID: 1...
INFO: -> Stored Read ID: 101 for Bundle ID: 1INFO: -> Stored Read ID: 102 for Bundle ID: 1
...SUCCESS: All relational data has been cleanly written to SQLite.


## Database Schema

The application automatically creates the following relational structure in reader_data.db. Foreign keys are
strictly enforced (ON DELETE CASCADE).

### Table: proto_reader_bundles (Parent)

```
Column Type Description
id INTEGER (PK) Unique Bundle ID
reader_id TEXT ID of the physical reader
site_id TEXT ID of the location/site
sent_timestamp_ms INTEGER Timestamp in milliseconds
```
### Table: proto_reads (Child)

```
Column Type Description
id INTEGER (PK) Unique Read ID
proto_reader_bundle_id INTEGER (FK) Links back to proto_reader_bundles.id
timestamp_ns INTEGER Precise read timestamp in nanoseconds
confidence REAL Read accuracy/confidence score
antenna_id TEXT ID of the antenna used
antenna_type TEXT Type of antenna (e.g., RFID)
x REAL X coordinate
y REAL Y coordinate
item_id TEXT Unique identifier of the scanned item
floor_id TEXT Floor location identifier
```

## Querying the Data

You can inspect the generated reader_data.db file using any standard SQLite tool (like DB Browser for
SQLite or the sqlite3 CLI).
To view reads alongside their parent bundle data, use a standard JOIN query:
SELECT b.reader_id,
b.site_id, r.item_id,
r.confidence, r.timestamp_ns
FROM proto_reads rJOIN proto_reader_bundles b ON r.proto_reader_bundle_id = b.id;


