# ADR 0001: Modular Ports-and-Adapters Architecture

- Status: Accepted
- Date: 2026-02-13

## Context

Santiago must evolve loop heuristics quickly while keeping integration with Scarlet stable and predictable. Direct coupling between CLI, loop logic, subprocess details, and artifact I/O would increase change risk and reduce testability.

## Decision

Adopt a modular ports-and-adapters architecture with these core modules:

- `contract/` for schemas, enums, and shared types.
- `planner/` for deterministic PRD decomposition.
- `executor/` for story loop orchestration and resume.
- `agent/`, `verification/`, and `artifacts/` as adapter modules behind interfaces.
- `cli/` as a thin composition root.

Business logic must depend on interfaces, not concrete backends.

## Consequences

- Positive: isolates volatility to loop and backend modules.
- Positive: enables focused unit testing with mocks.
- Positive: keeps integration contracts explicit and stable.
- Negative: introduces more modules and interface definitions up front.

## Alternatives Considered

- Single-layer CLI service object: rejected due to high coupling and poor change isolation.
- Full framework-based dependency injection container: rejected as unnecessary complexity for current scale.

## References

- `prds/001-santiago-loop-engine-architecture.md`
- `docs/architecture/02-component-model.md`
