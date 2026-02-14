# Quality, Security, and Operability

## Engineering Quality Model

Quality is enforced through deterministic outputs, bounded runtime behavior, and explicit evidence.

## Test Strategy

### Unit tests

- Planner determinism and ordering rules.
- Budget behavior under edge conditions.
- Resume reconstruction from progress events.
- Reason and exit code mapping.

### Integration tests

- `plan` command writes valid deterministic `plan.json`.
- `execute` writes progress events per attempt.
- Retry behavior and critique persistence on verification failure.
- Terminal `result.json` write on all exit scenarios.

### Contract tests

- Schema validation of input, plan, and result payloads.
- Compatibility checks between fixture payload versions.

## Observability Standards

- `progress.ndjson` is the canonical execution evidence stream.
- Event phases are normalized for reliable upstream parsing.
- Logs avoid secrets and avoid storing raw untrusted output by default.

## Security Controls

- Treat agent and tool output as untrusted.
- Restrict command execution to configured allowlist policy.
- Enforce subprocess timeout and output capture bounds.
- Restrict write paths to repository and artifact directory only.

## Reliability Controls

- Every command path returns a terminal result or deterministic error code.
- Runtime budget enforcement prevents runaway loops.
- Retry logic is bounded and reasoned by explicit taxonomy.

## Performance Targets (Initial)

- Plan generation should complete quickly for typical PRDs (target under 10 seconds for moderate input size).
- Event logging must be O(1) append per event.
- Resume startup should scale with linearly parsed event history.

## Release Readiness Checklist

- Contract schemas updated and versioned.
- ADR updated for architecture-impacting changes.
- Unit, integration, and contract test suites green.
- Manual dry run confirms artifact and exit code behavior.
