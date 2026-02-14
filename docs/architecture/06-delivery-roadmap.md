# Delivery Roadmap and Milestones

## Purpose

Translate architecture into phased delivery with clear done criteria.

## Phase 1: Contract and Planner Foundation

### Scope

- Define `run-input`, `plan`, and `result` schemas.
- Implement schema validation boundary checks.
- Implement deterministic `santiago plan`.

### Done criteria

- AC-01 satisfied.
- Golden tests prove deterministic plan output.
- Invalid input paths return deterministic validation failure.

## Phase 2: Execution Loop and Verification

### Scope

- Implement `santiago execute` story loop.
- Integrate story and run verification commands.
- Implement reason code and exit code mapping.

### Done criteria

- AC-02, AC-03, AC-04 satisfied.
- Event stream records every attempt transition.
- Failure paths generate terminal `result.json`.

## Phase 3: Resume and Critique Enrichment

### Scope

- Artifact-based resume reconstruction.
- Skip already-completed stories.
- Improve critique packet generation for retries.

### Done criteria

- AC-05 and AC-06 satisfied.
- Resume tests validate no duplicate story execution.
- Critique artifacts provide actionable retry context.

## Phase 4: Backend Expansion and Evaluation

### Scope

- Add alternate execution backends via `AgentBackend`.
- Add quality metrics and evaluation harness.
- Add comparative reliability and throughput reports.

### Done criteria

- New backend can be enabled by config without executor changes.
- Evaluation suite produces repeatable benchmark output.
- Architecture docs and ADRs updated for major changes.

## Cross-Phase Delivery Rules

- Keep contracts backward compatible unless version is bumped.
- Add or update ADRs for architecture-impacting changes.
- Require passing contract, unit, and integration tests before merge.
