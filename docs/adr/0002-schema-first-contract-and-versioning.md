# ADR 0002: Schema-First Contract and Versioning

- Status: Accepted
- Date: 2026-02-13

## Context

Scarlet and Santiago communicate through file-based JSON contracts. Without strict validation and versioning rules, contract drift can cause silent orchestration failures.

## Decision

Use JSON Schema as the source of truth for external contracts:

- validate `run-input.json` on command entry,
- validate generated `plan.json` before persistence,
- validate `result.json` before terminal exit.

Require `contract_version` in all top-level contract files and apply explicit compatibility rules.

## Consequences

- Positive: early detection of invalid payloads.
- Positive: safe evolution path for additive changes.
- Positive: clear major-version boundary for breaking changes.
- Negative: schema updates add process overhead for quick experiments.

## Alternatives Considered

- Type-only validation in code: rejected because runtime payloads can be malformed.
- No explicit versioning field: rejected because compatibility cannot be reasoned safely.

## References

- `prds/001-santiago-loop-engine-architecture.md`
- `docs/architecture/04-data-contracts.md`
