# Changelog

All notable changes to pkg-sqlite will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [0.2.3] - 2026-07-09

### Changed
- Bumped `rusqlite` from 0.31 to 0.40
- Updated bundled SQLite from 3.44 to 3.47 (embedded via rusqlite's `bundled` feature)

## [0.2.2] - 2026-07-05

### Fixed
- Declare SPDX licenses on external dependencies so `mvl package check` (mvl-lang/mvl#1698) passes:
  - `rusqlite 0.31` — MIT
  - Bundled SQLite C library (`sqlite3 3.44`, embedded via rusqlite's `bundled` feature) — `blessing` (SQLite's public-domain-style license). Added as a new `[c-native]` entry.

## [0.2.0] - 2026-06-18

### Changed
- `collect_rows` upgraded from `partial fn` to `total fn` — three while loops
  now carry `decreases` clauses, making termination statically provable
- `query`, `query_scalar`, `query_by_min_age` upgraded to `total fn ! DB` — were
  only `partial fn` because `collect_rows` was partial; totality now proven end-to-end

### Added
- `make coverage` — behavioral branch coverage report (`mvl test --coverage`)
- `make prove` — per-call-site refinement proof breakdown (`mvl prove --verbose`)
- `make version` — print current package version from `mvl.toml`
- Makefile `MVL` guard now falls back to `mvl` on PATH (no debug build required)
- `.openspec/adr/` — three ADRs documenting FFI boundary design, refinement proof
  approach, and totality policy

## [0.1.4] - 2026-06-18

### Fixed
- Removed stale `Clean[String]` from FFI declarations (`sqlite_open`, `sqlite_execute`, `sqlite_query`) — `Clean` is not a declared label in MVL's IFC system, causing REQ1 type mismatches in all consumers running `mvl check` (closes #4)

## [0.1.3] - 2026-06-18

### Changed
- Removed stale `#96` workaround re-declarations from `sqlite_test.mvl` — test file
  now imports directly from the package without local type mirrors

## [0.1.2] - 2026-06-04

### Fixed
- `bind_params`: handle `DbValue::Timestamp` variant added in MVL 0.206.0 — missing
  arm caused non-exhaustive match error in all callers (#1)

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
