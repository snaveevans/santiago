# Component and Module Model

## Design Goal

Separate stable contracts from fast-changing loop heuristics so the system is easy to maintain and evolve.

## Top-Level Modules

```text
src/
  cli/            # command parsing and wiring only
  contract/       # JSON schemas, types, reason codes, exit codes
  planner/        # PRD -> deterministic plan
  executor/       # story loop orchestration and resume
  agent/          # execution backend abstraction and implementations
  verification/   # story/run command verification
  artifacts/      # plan/progress/result persistence
  util/           # logger, clock, shared helpers
```

## Dependency Rule

Dependencies must point inward to stable abstractions:

1. `contract/` has no internal dependencies.
2. `planner/` and `executor/` depend on `contract/`.
3. `agent/`, `verification/`, and `artifacts/` implement ports used by `executor/`.
4. `cli/` composes concrete implementations but does not hold business logic.

## Port and Adapter Pattern

Core orchestration code should call interfaces, not concrete implementations.

### Key ports

- `AgentBackend`: execute one attempt with timeout and return normalized result.
- `Verifier`: execute story or run verification commands and return pass/fail details.
- `ArtifactStore`: read/write plan, progress events, attempts, critique, and result.
- `Clock`: provide current time for deterministic timeout and testing behavior.

## Component Responsibilities

### `planner/`

- Parse PRD inputs.
- Generate stable story IDs (`S1`, `S2`, ...).
- Build dependency graph.
- Produce deterministic ordering.
- Write `plan.json`.

### `executor/`

- Read plan and current artifact state.
- Skip already-completed stories during resume.
- Run per-story bounded attempt loops.
- Append progress events on each significant step.
- Trigger terminal result write.

### `agent/`

- Provide `ralph_fresh` default backend.
- Build prompt packet from story, acceptance criteria, context, and critique.
- Normalize backend failures into standard reason codes.

### `verification/`

- Execute story-level and run-level verification commands.
- Enforce timeout and output capture limits.
- Return structured failure metadata for critique.

### `artifacts/`

- Own all writes inside `out-dir`.
- Implement append-only `progress.ndjson` writing.
- Ensure atomic terminal `result.json` persistence.

## Composition Root

`cli/execute` is the composition root:

- validate `run-input.json` and `plan.json` against schemas,
- construct `ArtifactStore`, `AgentBackend`, `Verifier`, and `Budget` services,
- execute orchestration,
- map final status to exit code.

## Why This Model

- Fast-change areas (agent strategy, critique loop heuristics) stay isolated.
- Stable interfaces reduce accidental cross-module coupling.
- Unit tests can mock ports to validate loop behavior deterministically.
