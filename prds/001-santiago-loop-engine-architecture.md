# PRD 001: Santiago Loop Engine Architecture

## Document Intent

This PRD is the product requirement baseline for this repository.

It defines what Santiago must do as an execution worker, how it integrates with Scarlet,
and which contracts must remain stable as implementation evolves.

Normative architecture references:

- `docs/architecture/00-guiding-principles.md`
- `docs/architecture/README.md`
- `docs/architecture/01-system-context.md`
- `docs/architecture/02-component-model.md`
- `docs/architecture/03-execution-model.md`
- `docs/architecture/04-data-contracts.md`
- `docs/architecture/05-quality-and-operability.md`
- `docs/architecture/06-delivery-roadmap.md`

## Context

Scarlet and Santiago are intentionally split:

- **Scarlet** is the control plane (repo watching, orchestration state, git/GitHub I/O, service reliability).
- **Santiago** is the worker engine (deterministic PRD planning and bounded iterative execution loops).

This project implements Santiago only.

## Problem Statement

Large PRDs fail when sent to coding agents as one monolithic prompt. Reliable delivery needs:

- deterministic decomposition into smaller testable units,
- bounded attempts with verification feedback,
- explicit machine-readable progress and terminal outcomes.

These loop heuristics are expected to evolve quickly and must stay inside Santiago, not Scarlet.

## Goals

- **G1:** Convert one PRD into a deterministic execution plan of stories/tasks.
- **G2:** Execute bounded iterative loops per story with retries and critique.
- **G3:** Emit artifacts Scarlet can consume without parsing free-form logs.
- **G4:** Support artifact-driven resume without losing completed progress.
- **G5:** Keep execution backend strategy pluggable behind a stable interface.
- **G6:** Enforce explicit security and runtime boundaries.
- **G7:** Make architecture and contracts maintainable for both humans and LLM agents.

## Non-Goals

- Repository polling and global queue management.
- Branch creation, push, pull request creation, or merge orchestration.
- Scarlet service deployment and process supervision concerns.
- Long-term run state storage outside the run artifact directory.

## Guiding Principles (Project-Scoped)

Santiago implementation must follow:

1. Boundary-first ownership.
2. Contract is a product.
3. Determinism over cleverness.
4. Bounded autonomy.
5. Evidence over inference.
6. Change at the edges, stability at the core.
7. Secure by default.

Canonical definitions live in `docs/architecture/00-guiding-principles.md`.

## Product Boundary

### Scarlet owns

- PRD detection and run eligibility.
- Run identity and durable orchestration state.
- Git branch lifecycle and remote operations.
- Pull request success/failure workflows.
- Secret policy and operational guardrails.

### Santiago owns

- Plan generation (`PRD -> stories/tasks`).
- Story-level loop orchestration (`plan -> execute -> verify -> critique`).
- Per-story and run-level completion/failure signaling.
- Structured execution artifacts and terminal result payload.

## Integration Contract (v1)

Scarlet invokes Santiago with a run input file and output artifact directory.

### CLI surface

```bash
santiago plan --input <run-input.json> --out-dir <run-artifacts-dir>
santiago execute --input <run-input.json> --plan <plan.json> --out-dir <run-artifacts-dir>
```

### Input (`run-input.json`)

```json
{
  "contract_version": 1,
  "run_id": "run_20260213_abc123",
  "repo_path": "/abs/path/to/repo",
  "prd_path": "prds/003-example-feature.md",
  "base_branch": "main",
  "working_branch": "scarlet/example",
  "limits": {
    "story_max_attempts": 4,
    "run_max_attempts": 20,
    "story_timeout_minutes": 20,
    "run_timeout_minutes": 180
  },
  "verification": {
    "story_commands": ["npm test -- tests/auth.test.ts"],
    "run_commands": ["npm run lint", "npm test"]
  },
  "agent": {
    "mode": "ralph_fresh",
    "model_hint": "opencode"
  }
}
```

### Required outputs

- `plan.json`
- `progress.ndjson`
- `result.json`

Optional outputs:

- `critique.md`
- `attempts/<story-id>-attempt-<n>.md`
- `artifacts/<name>`

### Result (`result.json`)

```json
{
  "contract_version": 1,
  "run_id": "run_20260213_abc123",
  "status": "success",
  "reason": null,
  "stories": [
    {
      "id": "S1",
      "status": "done",
      "attempts": 2,
      "verification": "passed"
    }
  ],
  "summary": {
    "completed": 3,
    "failed": 0,
    "skipped": 0
  },
  "next_action": "open_pr"
}
```

### Exit codes

- `0`: terminal completion with `result.json`.
- `10`: blocked (requires human/system input).
- `20`: transient runtime failure (safe to retry).
- `30`: deterministic spec/validation failure.

## Planning Model

`santiago plan` compiles PRD content into a deterministic plan with:

- stable story IDs (`S1`, `S2`, ...),
- dependency edges,
- acceptance criteria mapping,
- file-focus hints,
- story-level verification command mapping.

Ordering rules:

1. dependency order first,
2. requirement ID order second,
3. lexicographic tie-breakers third.

Determinism requirement: same PRD input and config must produce byte-stable ordering and IDs.

## Execution Loop Model

Default mode: `ralph_fresh` (fresh process per attempt).

Per story algorithm:

1. Build prompt packet from story data, acceptance criteria, repo context, and prior critique.
2. Execute one agent attempt.
3. Run story verification commands.
4. If verification fails, persist critique and retry while budgets permit.
5. If verification passes, mark story done and continue.

Run stop conditions:

- success: all stories done and run-level verification passed,
- failed: attempt budget or timeout exhausted,
- blocked: non-recoverable dependency or external precondition.

## State and Resume

Santiago uses artifact-local state only inside `out-dir`.

- No global queue state in Santiago.
- Resume is driven by existing `plan.json`, `progress.ndjson`, and `result.json`.
- Completed stories must not be restarted on resume.
- Scarlet remains source of truth for cross-run orchestration state.

## Error Taxonomy

`result.reason` is one of:

- `plan_generation_failed`
- `agent_exit_nonzero`
- `agent_timeout`
- `story_verification_failed`
- `run_verification_failed`
- `attempt_budget_exhausted`
- `run_timeout`
- `blocked_dependency`

## Security and Safety

- Treat all tool and agent output as untrusted.
- Never log raw secrets.
- Restrict writes to repository path and run artifact directory.
- Enforce bounded subprocess execution via timeout and command policy.

## Observability

`progress.ndjson` is append-only and event-based.

Each event includes:

- `timestamp`
- `run_id`
- `story_id` (if applicable)
- `phase`
- `attempt`
- `status`
- `context`

This event stream is canonical execution evidence for Scarlet orchestration and PR reporting.

## Acceptance Criteria

- **AC-01:** Given a valid PRD input, `santiago plan` writes deterministic `plan.json` and exits `0`.
- **AC-02:** Given an existing plan, `santiago execute` writes `progress.ndjson` events per attempt.
- **AC-03:** Given story verification failures within budget, Santiago retries and records critique.
- **AC-04:** Given attempt budget exhaustion, Santiago exits non-success with reason `attempt_budget_exhausted`.
- **AC-05:** Given resumed execution, Santiago does not restart completed stories.
- **AC-06:** `result.json` always exists on terminal exit, including failure paths.
- **AC-07:** Invalid contract payloads exit with deterministic validation failure (`30`).
- **AC-08:** Agent backend can be swapped by mode without executor rewrite.

## Delivery Phases

### Phase 1: Contract and Planner Foundation

- Define and validate contract schemas.
- Implement deterministic `plan` command.

### Phase 2: Execute Loop and Verification

- Implement `execute` loop with retries and progress events.
- Integrate story/run verification hooks.

### Phase 3: Resume and Critique

- Implement artifact-driven resume.
- Add richer critique packet generation.

### Phase 4: Backend Expansion and Evaluation

- Add optional agent backends.
- Add reliability and quality evaluation harness.

## Bottom Line

Santiago exists to make large PRDs tractable by converting them into deterministic, verifiable,
bounded iterative work units with auditable artifacts.

Scarlet remains the control plane. Santiago remains the execution worker.
