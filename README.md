# santiago

Santiago is the large-PRD planning and execution engine used by Scarlet.

## Purpose

- Break large PRDs into deterministic story/task slices.
- Run bounded iterative agent loops per story.
- Gate progress with verification feedback.
- Emit machine-readable artifacts Scarlet can consume.

## Relationship to Scarlet

Santiago is the data plane. Scarlet is the control plane.

- Scarlet owns: polling, queueing, durable orchestration state, git branch lifecycle, push, PR creation, cleanup, and service operations.
- Santiago owns: PRD compilation, task decomposition, loop strategy, attempt control, and execution evidence.

Scarlet calls Santiago for each run and uses Santiago outputs to decide whether to open success or failure PRs.

## Core doc

- Loop architecture and integration contract: `prds/001-santiago-loop-engine-architecture.md`

## Architecture docs

- Guiding principles: `docs/architecture/00-guiding-principles.md`
- Architecture index: `docs/architecture/README.md`
- ADR index: `docs/adr/README.md`
