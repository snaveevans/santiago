# Data Contracts and Artifact Model

## Contract-First Policy

All external payloads must be schema-validated at boundaries. Contract changes are versioned and explicit.

## Required Schema Artifacts

- `run-input.schema.json`
- `plan.schema.json`
- `result.schema.json`

The runtime must validate:

- input JSON before command execution,
- generated `plan.json` before write,
- terminal `result.json` before exit.

## Versioning Rules

- `contract_version` is mandatory in all top-level payloads.
- New optional fields are additive and backward compatible.
- Removing or changing meaning of existing fields requires a major version bump.

## Artifact Layout

Expected output directory structure:

```text
<out-dir>/
  plan.json
  progress.ndjson
  result.json
  critique.md                      # optional
  attempts/<story-id>-attempt-<n>.md  # optional
  artifacts/<name>                 # optional
```

## Progress Event Contract

Each `progress.ndjson` event includes:

- `timestamp`
- `run_id`
- `story_id` (nullable for run-level events)
- `phase`
- `attempt`
- `status`
- `context`

Event log is append-only.

## Exit Code Contract

- `0`: terminal completion with `result.json`.
- `10`: blocked, requires manual/system input.
- `20`: transient runtime failure, safe to retry.
- `30`: deterministic validation/spec failure.

## Reason Code Contract

`result.reason` is one of:

- `plan_generation_failed`
- `agent_exit_nonzero`
- `agent_timeout`
- `story_verification_failed`
- `run_verification_failed`
- `attempt_budget_exhausted`
- `run_timeout`
- `blocked_dependency`

## Write Guarantees

- `result.json` must exist on every terminal exit path.
- `progress.ndjson` writes must be append-only and never rewrite previous lines.
- Artifact writes should use atomic replace where feasible (`*.tmp` then rename).
