# Planning and Execution Model

## Planning Flow

`santiago plan` transforms one PRD into a deterministic plan with stable story IDs, dependencies, acceptance criteria mapping, file-focus hints, and story-level verification commands.

Ordering is deterministic by rule:

1. dependency order,
2. requirement ID order,
3. lexicographic tie-breaker.

## Execution Flow

`santiago execute` consumes `run-input.json` and `plan.json`, then processes stories in plan order.

Per story loop:

1. Build prompt packet from story context and prior critique.
2. Execute one agent attempt.
3. Run story verification commands.
4. If verification fails, capture critique and retry while budget allows.
5. If verification passes, mark story done and continue.

## State Machines

### Story state

`pending -> in_progress -> done | failed | blocked`

### Run state

`initialized -> planning | executing -> run_verification -> success | failed | blocked`

## Budget Model

Budgets are hard limits and must be checked before each attempt:

- `story_max_attempts`
- `run_max_attempts`
- `story_timeout_minutes`
- `run_timeout_minutes`

On exhaustion, execution exits terminally with explicit reason (`attempt_budget_exhausted` or `run_timeout`).

## Resume Model

Resume is artifact-driven and local to `out-dir`:

- Load `plan.json` as immutable run plan.
- Reconstruct story status from `progress.ndjson` and `result.json` if present.
- Skip stories already marked done.
- Continue from first non-terminal story.

Resume never rewinds completed work.

## Stop Conditions

- `success`: all stories done and run-level verification passed.
- `failed`: timeout, budget exhaustion, or deterministic failure reason.
- `blocked`: external precondition or dependency issue requiring manual intervention.

## Concurrency Policy

Initial mode is serial story execution to minimize merge conflict risk and simplify deterministic behavior.

Future optimization can parallelize independent stories if and only if:

- dependency DAG allows parallel branches,
- file-focus hints do not overlap,
- deterministic ordering of completion events is preserved.
