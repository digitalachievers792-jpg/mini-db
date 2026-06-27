# Mini-DB

A minimal relational database engine built entirely from scratch in Python — no external database libraries, no SQLite, no ORM. Just pure Python standard library.

## Features

- **SQL Support** — `CREATE TABLE`, `INSERT`, `SELECT` (with `WHERE`, column selection), `UPDATE`, `DELETE`
- **Persistent Storage** — Custom 4 KB page-based file format; data survives restarts
- **B+ Tree Indexing** — O(log n) primary-key lookups, range scans via linked leaf nodes
- **Query Planner** — Automatically chooses index scan vs. full table scan based on query
- **Transactions** — `BEGIN` / `COMMIT` / `ROLLBACK` with file-snapshot isolation
- **Data Types** — `INT`, `TEXT`, `VARCHAR`, `BOOL`
- **Interactive CLI** — REPL shell with formatted table output (similar to `sqlite3`)
- **LIKE Operator** — Pattern matching with `%` and `_` wildcards
- **Zero Dependencies** — Only Python 3.10+ standard library required

## Quick Start

```bash
# Clone the repository
git clone https://github.com/digitalachievers792-jpg/mini-db.git
cd mini-db

# Start the interactive shell
python cli.py

# Start fresh (deletes all existing data)
python cli.py --init
```

## Example Session

```
Mini-DB v0.1

mini-db> CREATE TABLE students (
  ...>     id INT PRIMARY KEY,
  ...>     name TEXT,
  ...>     age INT
  ...> );
  Table 'students' created

mini-db> INSERT INTO students VALUES (1, 'Alice', 20);
  1 row inserted into 'students'

mini-db> INSERT INTO students VALUES (2, 'Bob', 22);
  1 row inserted into 'students'

mini-db> INSERT INTO students VALUES (3, 'Charlie', 21);
  1 row inserted into 'students'

mini-db> SELECT * FROM students;

id | name    | age
---+---------+----
1  | Alice   | 20
2  | Bob     | 22
3  | Charlie | 21

(3 rows)

mini-db> SELECT name, age FROM students WHERE age >= 21;

name    | age
--------+----
Bob     | 22
Charlie | 21

(2 rows)

mini-db> UPDATE students SET age = 23 WHERE name = 'Bob';
  1 row(s) updated

mini-db> DELETE FROM students WHERE id = 2;
  1 row(s) deleted

mini-db> BEGIN;
  BEGIN
mini-db> INSERT INTO students VALUES (4, 'Diana', 19);
  1 row inserted into 'students'
mini-db> ROLLBACK;
  ROLLBACK

mini-db> SELECT * FROM students;

id | name    | age
---+---------+----
1  | Alice   | 20
3  | Charlie | 21

(2 rows)

mini-db> EXIT
Bye.
```

## Architecture

```
┌──────────────────────────────────────────────────────┐
│                      CLI (cli.py)                     │
├──────────────────────────────────────────────────────┤
│   SQL Layer                                           │
│   ┌──────────┐  ┌──────────┐  ┌──────────────┐      │
│   │  Lexer   │→ │  Parser  │→ │   AST Nodes  │      │
│   └──────────┘  └──────────┘  └──────────────┘      │
├──────────────────────────────────────────────────────┤
│   Engine Layer                                        │
│   ┌──────────┐  ┌──────────┐                         │
│   │ Catalog  │→ │ Planner  │→ Execution Plan         │
│   └──────────┘  └──────────┘                         │
│   ┌──────────┐                                        │
│   │ Executor │ ← runs the plan                       │
│   └──────────┘                                        │
├──────────────────────────────────────────────────────┤
│   Storage Layer                                       │
│   ┌──────────────┐  ┌──────────────┐  ┌───────────┐  │
│   │ DiskManager  │  │   Page (4KB) │  │ B+ Tree   │  │
│   └──────────────┘  └──────────────┘  └───────────┘  │
├──────────────────────────────────────────────────────┤
│   Transaction Layer                                   │
│   ┌──────────────┐                                    │
│   │ TxnManager   │ ← BEGIN / COMMIT / ROLLBACK       │
│   └──────────────┘                                    │
└──────────────────────────────────────────────────────┘
```

## Project Structure

```
mini-db/
├── cli.py                  # Interactive REPL shell
├── sql/
│   ├── lexer.py            # Tokenizer (regex-based)
│   ├── parser.py           # Recursive-descent SQL parser
│   └── ast_nodes.py        # AST node classes
├── engine/
│   ├── catalog.py          # Schema persistence (JSON)
│   ├── planner.py          # Query planning & validation
│   └── executor.py         # Plan execution & row I/O
├── storage/
│   ├── page.py             # 4 KB fixed-size pages with slot-based records
│   ├── disk_manager.py     # Binary file I/O for page storage
│   └── bptree.py           # B+ Tree index (primary key lookups)
├── transaction/
│   └── txn_manager.py      # Snapshot-based BEGIN/COMMIT/ROLLBACK
├── data/                   # Runtime data files (*.db, *.idx, catalog.json)
├── visual.html             # Visual project overview
├── test_step1.py           # Tests: Page + DiskManager
├── test_step2.py           # Tests: B+ Tree
├── test_step3.py           # Tests: Lexer
├── test_step4.py           # Tests: Parser + AST
├── test_step5.py           # Tests: Catalog
├── test_step6.py           # Tests: Planner
├── test_step7.py           # Tests: Executor (end-to-end)
├── test_step8.py           # Tests: Transactions
└── README.md
```

## How It Works

### SQL Pipeline

1. **Lexer** (`sql/lexer.py`) — Converts raw SQL text into tokens using regex
2. **Parser** (`sql/parser.py`) — Builds an AST from tokens using recursive-descent parsing
3. **Planner** (`engine/planner.py`) — Validates the AST against the catalog and creates an execution plan
4. **Executor** (`engine/executor.py`) — Executes the plan against the storage engine

### Storage Engine

- **Pages** — Fixed 4 KB blocks with a header (record count + free pointer) and length-prefixed records
- **DiskManager** — Maps logical record IDs (page_id << 16 | slot_id) to physical offsets
- **B+ Tree** — Maps primary key values to record IDs; supports point lookups and range scans via linked leaf nodes

### Transactions

- `BEGIN` — Creates `.bak` snapshots of all `.db` and `.idx` files
- `COMMIT` — Deletes the `.bak` files (changes are permanent)
- `ROLLBACK` — Restores `.bak` files over originals (undoes all changes)

## Running Tests

```bash
python test_step1.py   # Page + DiskManager
python test_step2.py   # B+ Tree
python test_step3.py   # Lexer
python test_step4.py   # Parser + AST
python test_step5.py   # Catalog
python test_step6.py   # Planner
python test_step7.py   # Executor (end-to-end)
python test_step8.py   # Transactions
```

## Supported SQL Commands

| Command | Description |
|---------|-------------|
| `CREATE TABLE` | Create a new table with column definitions |
| `INSERT INTO ... VALUES` | Insert rows into a table |
| `SELECT ... FROM` | Query data with optional `WHERE` clause |
| `UPDATE ... SET` | Modify existing rows |
| `DELETE FROM` | Remove rows from a table |
| `BEGIN` | Start a transaction |
| `COMMIT` | Commit current transaction |
| `ROLLBACK` | Rollback current transaction |
| `EXIT` / `QUIT` | Exit the shell |

## Data Types

| Type | Description | Storage |
|------|-------------|---------|
| `INT` | 64-bit integer | 8 bytes |
| `TEXT` | Variable-length string | 4 + N bytes |
| `VARCHAR` | Variable-length string | 4 + N bytes |
| `BOOL` | Boolean value | 1 byte |

## License

MIT
