# AGENTS.md

## Mission
- This file guides autonomous coding agents working in `santiago`.
- Optimize for deterministic behavior, bounded execution, and machine-readable artifacts.
- Treat architecture docs as normative, not optional commentary.

## Primary References (Read First)
- `README.md`
- `docs/architecture/README.md`
- `docs/architecture/00-guiding-principles.md`
- `docs/architecture/01-system-context.md`
- `docs/architecture/02-component-model.md`
- `docs/architecture/03-execution-model.md`
- `docs/architecture/04-data-contracts.md`
- `docs/architecture/05-quality-and-operability.md`
- `docs/adr/README.md`

## Architecture Principles (Required)
These come from `docs/architecture/README.md` and `docs/architecture/00-guiding-principles.md`.

- Boundary-first ownership: keep Scarlet as control plane and Santiago as worker.
- Contract is a product: schemas/versioning are first-class interfaces.
- Determinism over cleverness: same inputs produce same plan and ordering.
- Bounded autonomy: enforce attempt/time budgets and explicit stop conditions.
- Evidence over inference: rely on `plan.json`, `progress.ndjson`, `result.json`.
- Change at the edges: isolate volatile heuristics behind stable ports.
- Secure by default: treat tool output as untrusted and constrain execution.

## Repository Reality (Current)
- The repo is architecture-first; implementation code is not scaffolded yet.
- There is no committed package/runtime manifest at the repository root.
- Do not assume hidden scripts exist.
- If you introduce tooling, update this file in the same change.

## Build, Lint, Test Commands

### Current runnable commands
- There are no project build/lint/test commands committed yet.
- Current work is document and contract driven.

### Required command contract when scaffolding
When implementation is added, expose these stable commands for agents:

```bash
# install deps
npm ci

# compile/build
npm run build

# static checks
npm run lint

# full test suite
npm test

# run one test file (critical)
npm test -- tests/path/to/file.test.ts

# run one test by name (critical)
npm test -- -t "name substring"
```

Notes:
- The single-test file pattern aligns with the PRD example (`npm test -- tests/auth.test.ts`).
- If a non-Node stack is adopted, provide equivalent commands and keep single-test support.

### Runtime CLI contract (must remain stable)
From architecture docs and PRD:

```bash
santiago plan --input <run-input.json> --out-dir <run-artifacts-dir>
santiago execute --input <run-input.json> --plan <plan.json> --out-dir <run-artifacts-dir>
```

## Code Style Guidelines

### Module boundaries
- Keep business logic out of CLI wiring.
- Use ports/adapters so executor core depends on interfaces, not concrete backends.
- Preserve module roles described in `docs/architecture/02-component-model.md`.

### Imports
- Group imports: standard library, third-party, internal.
- Prefer absolute/internal aliases over deep relative chains when available.
- No wildcard imports.
- Remove unused imports in the same patch.

### Formatting
- Use one formatter and one linter for the chosen stack.
- Do not hand-format against formatter output.
- Keep lines readable (target 100 chars unless toolchain enforces otherwise).
- Keep markdown edits stable and low-noise.

### Types and schemas
- Prefer strict types at all boundaries.
- Validate all external JSON payloads against versioned schemas.
- Keep `contract_version` explicit in top-level payloads.
- Prefer additive schema changes; breaking semantics require version bump.
- Avoid untyped escape hatches unless isolated and documented.

### Naming conventions
- Types/interfaces/classes: `PascalCase`.
- Functions/variables: `camelCase`.
- Constants/env keys: `UPPER_SNAKE_CASE`.
- File names: `kebab-case` unless language conventions require another pattern.
- Story IDs are stable and sequential (`S1`, `S2`, ...).
- Keep reason codes aligned with architecture docs.

### Error handling and exit behavior
- Never swallow errors.
- Return structured, machine-readable failures.
- Preserve root cause when wrapping errors.
- Distinguish deterministic validation failures from transient runtime failures.
- Respect exit code contract: `0`, `10`, `20`, `30`.
- Always produce terminal `result.json` on terminal exits.

### Observability and artifacts
- `progress.ndjson` is append-only execution evidence.
- Emit normalized phase/status events with run/story/attempt context.
- Use atomic write patterns for terminal artifacts when feasible.
- Do not log secrets or raw sensitive payloads.

### Verification and testing behavior
- Gate progress on explicit verification commands, never assumptions.
- Keep story-level checks fast and specific.
- Keep run-level checks comprehensive but bounded.
- Add determinism tests for plan ordering and tie-breakers.
- Add retry/budget/timeout tests for execute loop.
- Add resume tests proving completed stories are not rerun.

### Security and command execution
- Treat agent/tool output as untrusted input.
- Restrict writes to repo workspace and artifact directory.
- Enforce timeout/output bounds on subprocesses.
- Follow an explicit command allowlist policy when implemented.

## ADR Expectations
- Any architecture-impacting change should include a new or updated ADR.
- Follow `docs/adr/README.md` naming and required sections.
- Do not renumber existing ADRs.

## Cursor and Copilot Rules
- No `.cursor/rules/` directory is currently present.
- No `.cursorrules` file is currently present.
- No `.github/copilot-instructions.md` file is currently present.
- If these files appear, treat them as authoritative and merge constraints into this file.

## Agent Checklist Before Finishing
- Re-read relevant architecture docs for the area touched.
- Confirm boundary ownership (Scarlet vs Santiago) is preserved.
- Validate contracts and version semantics for any payload changes.
- Run lint/tests (or document why not runnable in current scaffold state).
- Verify single-test command works once tests exist.
- Update `AGENTS.md` when commands, structure, or policy changes.
