# ADR 0006: Deterministic Planning and Ordering Rules

- Status: Accepted
- Date: 2026-02-13

## Context

Non-deterministic plan generation creates noisy diffs, unstable retries, and inconsistent outcomes between equivalent runs.

## Decision

Planning output must be deterministic for the same PRD input and configuration.

Story ordering is resolved by:

1. dependency order,
2. requirement ID order,
3. lexicographic tie-breakers.

Story IDs are stable (`S1`, `S2`, ...) and preserved across resumed execution for the same plan.

## Consequences

- Positive: reproducible plans simplify debugging and test assertions.
- Positive: reduces orchestration uncertainty and diff churn.
- Positive: makes resume semantics more predictable.
- Negative: planner implementation requires stronger normalization and ordering discipline.

## Alternatives Considered

- Heuristic score-based dynamic ordering: rejected due to poor reproducibility.
- Randomized tie-breaking: rejected due to non-repeatable outcomes.

## References

- `prds/001-santiago-loop-engine-architecture.md`
- `docs/architecture/03-execution-model.md`
