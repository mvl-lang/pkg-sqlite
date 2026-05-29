# Changelog

All notable changes to pkg-sqlite will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [0.1.1] - 2026-05-29

### Added
- Apache License 2.0
- CONTRIBUTING.md with setup, code style, and testing instructions
- README.md with install, usage example, API reference, and architecture

## [0.1.0] - 2026-05-29

### Added
- Initial release — SQLite embedded database driver for MVL
- Types: `SqliteDb` (opaque handle), `SqliteError` enum (`NotFound`, `ConstraintViolation`, `InvalidSql`, `Busy`, `Other`)
- `open(path)` — open/create database with `! DB + FileRead + FileWrite` effect
- `execute(db, sql, params)` — run INSERT/UPDATE/DELETE/DDL with `! DB` effect
- `query(db, sql, params)` — run SELECT, returns `List[Map[String, DbValue]]` with `! DB` effect
- `query_scalar(db, sql, params)` — single-value queries with `! DB` effect
- `close(db)` — pure function, no effect required
- Parameterized queries via `List[DbValue]` (prevents SQL injection)
- Wraps [rusqlite](https://github.com/rusqlite/rusqlite) with bundled SQLite via `bridge.rs` / `llvm.rs`
- Follows MVL extended package model (ADR-0012)
