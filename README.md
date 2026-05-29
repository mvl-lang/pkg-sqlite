# pkg-sqlite

SQLite embedded database driver for [MVL](https://github.com/LAB271/mvl_language).

Wraps [rusqlite](https://github.com/rusqlite/rusqlite) (with bundled SQLite) behind a fully-verified MVL API. All 11 compiler requirements are enforced on the public API surface.

## Install

```bash
mvl add github.com/mvl-lang/pkg-sqlite v0.1.0
mvl install
```

## Usage

```mvl
use pkg.sqlite.{SqliteDb, SqliteError, open, execute, query, query_scalar, close}
use std.db.{DbValue}

partial fn main() -> Unit ! DB + FileRead + FileWrite + Console {
    let db: SqliteDb = match open("app.db") {
        Ok(db) => db,
        Err(e) => { println("open failed"); return },
    };

    let _: Result[Int, SqliteError] = execute(db,
        "CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, name TEXT NOT NULL)",
        []);

    let _: Result[Int, SqliteError] = execute(db,
        "INSERT INTO users (name) VALUES (?)",
        [DbValue::Text("Alice")]);

    let rows: Result[List[Map[String, DbValue]], SqliteError] = query(db,
        "SELECT * FROM users", []);

    close(db)
}
```

## API

### Types

| Type | Description |
|------|-------------|
| `SqliteDb` | Opaque handle to an open database connection |
| `SqliteError` | Enum: `NotFound`, `ConstraintViolation`, `InvalidSql`, `Busy`, `Other(String)` |

### Functions

| Function | Signature | Effect |
|----------|-----------|--------|
| `open` | `(path: String) -> Result[SqliteDb, SqliteError]` | `! DB + FileRead + FileWrite` |
| `execute` | `(val db, sql: String, params: List[DbValue]) -> Result[Int, SqliteError]` | `! DB` |
| `query` | `(val db, sql: String, params: List[DbValue]) -> Result[List[Map[String, DbValue]], SqliteError]` | `! DB` |
| `query_scalar` | `(val db, sql: String, params: List[DbValue]) -> Result[DbValue, SqliteError]` | `! DB` |
| `close` | `(db: SqliteDb) -> Unit` | pure |

### Parameters

All query functions accept `List[DbValue]` for parameterized queries (prevents SQL injection):

```mvl
use std.db.{DbValue}

// Parameterized — safe
execute(db, "INSERT INTO users (name, age) VALUES (?, ?)",
    [DbValue::Text("Bob"), DbValue::Int(30)])

// DbValue variants: Int(Int), Float(Float), Text(String), Blob(List[Byte]), Null
```

## Architecture

```
pkg-sqlite/
├── mvl.toml         # package manifest ([native] declares rusqlite)
├── bridge.rs        # Rust FFI — rusqlite bindings
├── llvm.rs          # LLVM backend C-ABI bindings
└── src/
    ├── sqlite.mvl        # public API
    ├── sqlite_test.mvl   # tests
    └── internal/
        └── ffi.mvl       # extern "rust" declarations
```

The package follows the MVL extended package model ([ADR-0012](https://github.com/LAB271/mvl_language/blob/main/.openspec/adr/0012-extended-package-model.md)): `extern` at the bottom, verified API at the top.

## Effects

All database operations require the `DB` effect. `open` additionally requires `FileRead + FileWrite` for creating/opening the database file. Functions that don't perform I/O (like `close`) are pure.

## License

Apache License 2.0 — see [LICENSE](LICENSE).
