# ADR 0005: Append-Only Progress Event Log

- Status: Accepted
- Date: 2026-02-13

## Context

Scarlet needs precise, machine-readable execution evidence for orchestration and PR narrative generation. Mutable status files are difficult to audit and can lose historical context.

## Decision

Use an append-only NDJSON event log (`progress.ndjson`) as the canonical progress stream. Each event must include timestamp, run ID, story ID (if applicable), phase, attempt, status, and context.

No event line may be rewritten or deleted.

## Consequences

- Positive: complete and auditable history of the execution loop.
- Positive: easy streaming and parsing in control-plane consumers.
- Positive: supports deterministic resume reconstruction.
- Negative: file size grows over long runs and requires linear replay.

## Alternatives Considered

- Single mutable status document: rejected due to loss of history.
- Binary event format: rejected due to reduced operability and debugging ergonomics.

## References

- `prds/001-santiago-loop-engine-architecture.md`
- `docs/architecture/04-data-contracts.md`
