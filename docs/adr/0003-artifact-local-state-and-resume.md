# ADR 0003: Artifact-Local State and Resume

- Status: Accepted
- Date: 2026-02-13

## Context

Santiago must support resume semantics without owning global orchestration state. Storing global execution state inside Santiago would blur boundaries with Scarlet and increase operational complexity.

## Decision

Santiago stores execution state only in the run artifact directory (`out-dir`) and reconstructs progress from:

- `plan.json`
- `progress.ndjson`
- `result.json`

Resume behavior is deterministic and must never restart stories that are already completed.

## Consequences

- Positive: clear separation of concerns with Scarlet.
- Positive: artifacts become portable execution evidence.
- Positive: simpler recovery model with no extra service dependency.
- Negative: resume startup cost grows with progress log size.

## Alternatives Considered

- External state database inside Santiago: rejected due to control-plane overlap.
- In-memory only state: rejected because it cannot survive process interruption.

## References

- `prds/001-santiago-loop-engine-architecture.md`
- `docs/architecture/03-execution-model.md`
