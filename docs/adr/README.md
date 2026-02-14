# Architecture Decision Records

This directory stores Architecture Decision Records (ADRs) for Santiago.

## Why ADRs

ADRs make architectural intent explicit, reviewable, and durable over time.

## Status Values

- `Proposed`: drafted, pending acceptance.
- `Accepted`: approved and active.
- `Deprecated`: no longer preferred for new work.
- `Superseded`: replaced by a newer ADR.

## ADR Naming

- Use zero-padded numeric prefixes: `0001-...md`, `0002-...md`.
- Keep titles short and decision-focused.
- Never renumber existing ADRs.

## Required Sections

- Status
- Date
- Context
- Decision
- Consequences
- Alternatives Considered

## How to Add a New ADR

1. Copy `docs/adr/0000-template.md`.
2. Assign next sequential number.
3. Fill all required sections.
4. Link related ADRs when superseding or extending prior decisions.

## ADR Index

- `docs/adr/0001-modular-ports-and-adapters-architecture.md`
- `docs/adr/0002-schema-first-contract-and-versioning.md`
- `docs/adr/0003-artifact-local-state-and-resume.md`
- `docs/adr/0004-pluggable-agent-backend-interface.md`
- `docs/adr/0005-append-only-progress-event-log.md`
- `docs/adr/0006-deterministic-planning-ordering.md`
