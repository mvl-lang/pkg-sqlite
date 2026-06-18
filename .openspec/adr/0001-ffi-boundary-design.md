# ADR-0001: FFI Boundary Design — Primitive Types Only

**Status:** Accepted
**Date:** 2026-06-18
**Context:** pkg-sqlite wraps rusqlite via an `extern "rust"` block. The design question is: what types should cross the FFI boundary?

## Decision

**Only primitive types cross the FFI boundary.** The `extern "rust"` block in `src/internal/ffi.mvl` uses only `Int`, `Float`, `Bool`, and `String`. The MVL-level types (`SqliteDb`, `SqliteError`, `DbValue`) are composed entirely in `sqlite.mvl` from these primitives.

Specifically:
- Database and result handles are `Int` (positive = valid, -1 = error)
- Error codes are `Int` with a documented enum mapping in the FFI comment
- Blob values are `String` in comma-separated decimal format (`"65,66,67"`)
- `blob_to_wire` / `wire_to_blob` in `sqlite.mvl` handle blob encoding/decoding

## Rationale

MVL's IFC and effect system cannot enforce properties inside `extern "rust"` code. By keeping the boundary primitive, the trust surface is minimal:
- The MVL layer can verify all logic that composes `DbValue`, `SqliteError`, and `Row`
- Only the Rust bridge (`bridge.rs`) deals with rusqlite directly
- LLVM backend uses the same primitive ABI (`llvm.rs`)
- No MVL struct or enum leaks into unsafe Rust; no Rust type leaks into MVL

## Blob encoding

Blobs cross as comma-separated decimal strings because MVL has no `List[Byte]` FFI primitive. `blob_to_wire([65,66,67])` = `"65,66,67"`. This is intentionally simple and auditable — the encoding is visible in plain text in `sqlite.mvl` and is covered by `blob_to_wire` / `wire_to_blob` unit tests.

## Consequences

- All complex type construction is verifiable by `mvl check`
- The Rust bridge is small and auditable (~100 lines)
- Blob round-trips are tested; blob encoding is explicit, not opaque
- Future: if MVL gains a native `Bytes` type, blob encoding can be removed

## Connected to

- MVL ADR-0006: FFI extern "rust" bridge (trust boundary model)
- MVL ADR-0007: stdlib import model
- `src/internal/ffi.mvl` — the extern block
- `bridge.rs` / `llvm.rs` — Rust implementation
