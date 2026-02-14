# ADR 0004: Pluggable Agent Backend Interface

- Status: Accepted
- Date: 2026-02-13

## Context

Execution strategy and backend tooling are expected to change over time. Hard-coding one backend into orchestration logic would make backend changes high risk and expensive.

## Decision

Define a stable `AgentBackend` interface used by `executor/`. Implement `ralph_fresh` as the initial backend and select implementation via `run-input.agent.mode`.

The executor must only consume normalized attempt results and reason codes.

## Consequences

- Positive: adding alternate backends does not require executor rewrites.
- Positive: easier testing through mock backend implementation.
- Positive: aligns with PRD goal for pluggable strategy.
- Negative: adapter implementation adds normalization code.

## Alternatives Considered

- Direct subprocess calls from executor: rejected due to tight coupling.
- Dynamic plugin loading system from day one: rejected as premature complexity.

## References

- `prds/001-santiago-loop-engine-architecture.md`
- `docs/architecture/02-component-model.md`
