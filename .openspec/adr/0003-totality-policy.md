# ADR-0003: Totality Policy — All Terminating Functions Must Be Explicit `total fn`

**Status:** Accepted
**Date:** 2026-06-18
**Context:** MVL infers totality (`total*`) for functions with no unbounded loops and no `partial fn` callees. The question is whether pkg-sqlite should rely on inference or annotate explicitly.

## Decision

Same policy as pkg-http ADR-0002: **all terminating functions carry explicit `total fn`. No implicit totality (`total*`) permitted.**

## Application to pkg-sqlite

`collect_rows` was `partial fn` because its three while loops lacked `decreases` clauses — the termination checker couldn't verify them. Since `collect_rows` was partial, its callers `query`, `query_scalar`, and `query_by_min_age` were also partial.

With `decreases` clauses added:

```mvl
while ci < n_cols decreases n_cols - ci { ... }
while ri < n_rows decreases n_rows - ri {
    while cj < n_cols decreases n_cols - cj { ... }
}
```

The termination checker can verify all three loops, `collect_rows` becomes `total fn`, and the upgrade propagates up:

| Function | Before | After |
|---|---|---|
| `collect_rows` | `partial fn` | `total fn` |
| `query` | `partial fn ! DB` | `total fn ! DB` |
| `query_scalar` | `partial fn ! DB` | `total fn ! DB` |
| `query_by_min_age` | `partial fn ! DB` | `total fn ! DB` |

`make assurance` reports `total fn: 10 (10 explicit, 0 implicit)` — the target state.

## Remaining `partial fn`

No functions are `partial fn` after this upgrade — all functions in the package are proven total, including those with effects. `total fn` and effects are orthogonal in MVL.

## Consequences

- `make assurance` will flag any new function without an explicit totality keyword as `total*`
- Reviewers should reject PRs that introduce implicit totality
- The `decreases` clause is the mechanism; the `total fn` keyword is the contract

## Connected to

- MVL Req 3 (Totality) and Req 8 (Termination): verified by `mvl assurance`
- ADR-0002: `read_cell` refinement — `read_cell` is also `total fn`
- pkg-http ADR-0002: same policy, established first
