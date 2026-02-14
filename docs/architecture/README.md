# Santiago Architecture

This document set defines the target architecture for Santiago as a reliable worker process called by Scarlet.

## Goals

- Keep the Scarlet and Santiago boundary stable through explicit contracts.
- Make loop heuristics easy to evolve without changing stable integration surfaces.
- Guarantee deterministic planning, bounded execution, and resumable progress.
- Keep artifacts machine-readable so Scarlet can orchestrate confidently.

## Architecture Principles

Canonical principles for developers and LLM agents:

- `docs/architecture/00-guiding-principles.md`

## Document Map

- Guiding principles and review rubric: `docs/architecture/00-guiding-principles.md`
- System context and boundaries: `docs/architecture/01-system-context.md`
- Component and module model: `docs/architecture/02-component-model.md`
- Planning and execution loop model: `docs/architecture/03-execution-model.md`
- Contract and artifact data model: `docs/architecture/04-data-contracts.md`
- Quality, security, and operability: `docs/architecture/05-quality-and-operability.md`
- Delivery roadmap and milestones: `docs/architecture/06-delivery-roadmap.md`

## ADRs

All major architecture decisions are captured under `docs/adr/`.

- ADR process and index: `docs/adr/README.md`

## Scope

This architecture covers Santiago only. Scarlet remains the control plane and source of truth for global orchestration state.
