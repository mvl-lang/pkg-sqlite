# ADR-0002: `read_cell` Refinement — Row and Column Indices Are Non-Negative

**Status:** Accepted
**Date:** 2026-06-18
**Context:** `read_cell(rh, row, col)` is the internal helper that decodes one cell from an FFI result handle. Row and column indices are non-negative integers. How do we document and enforce this invariant?

## Decision

`read_cell` carries explicit `where` constraints on both index parameters:

```mvl
total fn read_cell(rh: Int, row: Int where row >= 0, col: Int where col >= 0) -> DbValue
```

This creates a refinement proof obligation at every call site. The only current caller is `query_scalar`, which calls `read_cell(rh, 0, 0)` — both literal indices `0` trivially satisfy `>= 0` (L1:trivial), giving two proven call sites in the assurance report.

## Why not `collect_rows`?

`collect_rows` iterates over rows and columns using `ref Int` loop counters. If it called `read_cell(rh, ri, cj)`, the prover would need to prove `ri >= 0` and `cj >= 0` across loop iterations for every cell. While the prover *can* prove this (both counters start at 0 and only increment), it would add O(rows × cols) runtime-checked call sites to the assurance report — or require complex loop invariant annotations.

Instead, `collect_rows` calls the FFI cell accessors directly, avoiding the refinement overhead. This is a deliberate trade-off: `query_scalar` gets clean L1:trivial proofs; `collect_rows` stays direct and auditable.

## Proof result

```
make prove:
  01:[216]  query_scalar → read_cell(row) — `self >= 0`  (1:trivial)
  02:[216]  query_scalar → read_cell(col) — `self >= 0`  (1:trivial)
  Summary: 2 proven (L1:2), 0 runtime, 0 failed
```

## Consequences

- `read_cell` is the single verified path for safe cell decoding
- `query_scalar` is fully proven at Req 10
- `collect_rows` remains free of unproven constraint call sites
- Any future caller passing computed indices must prove non-negativity statically or accept a runtime check

## Connected to

- ADR-0003: Totality policy — `read_cell` and `collect_rows` are both `total fn`
- pkg-http ADR-0003: Refinement proof approach (same philosophy: proofs over computed values, not constant wrappers)
