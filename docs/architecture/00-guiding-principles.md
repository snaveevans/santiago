# Guiding Principles

## Purpose

These principles are the default decision framework for Santiago. They are written for both human developers and LLM agents so implementation choices stay consistent as the system evolves.

## How to Use This Document

- Apply these principles during design, implementation, and review.
- If a change intentionally violates a principle, document the trade-off and open an ADR.
- Treat this as normative guidance for architecture-impacting decisions.

## Principles

### 1) Boundary-first ownership

Keep Scarlet as control plane and Santiago as execution worker.

- Santiago should not take on polling, queue orchestration, branch lifecycle, push, or PR publishing.
- Scarlet should not absorb Santiago loop heuristics.
- When in doubt, preserve the current product boundary and clarify via ADR.

### 2) Contract is a product

Treat schemas and payloads as first-class product interfaces.

- Validate all external payloads at boundaries.
- Keep `contract_version` explicit and enforce compatibility rules.
- Prefer additive changes; use major version bumps for semantic breaks.

### 3) Determinism over cleverness

Favor reproducible behavior over dynamic heuristics that are hard to reason about.

- The same input should produce the same plan and ordering.
- Stable story IDs and deterministic tie-breakers are mandatory.
- Avoid hidden randomness in planning and orchestration.

### 4) Bounded autonomy

Agent execution is powerful but must be strictly constrained.

- Enforce attempt budgets and timeout budgets at story and run levels.
- Gate progress on explicit verification, not optimistic assumptions.
- Always return terminal status with clear reason taxonomy.

### 5) Evidence over inference

Operational truth should come from machine-readable artifacts.

- Emit append-only progress events for every meaningful transition.
- Persist terminal `result.json` on all terminal exits.
- Prefer structured context over free-form logs for orchestration decisions.

### 6) Change at the edges, stability at the core

Put volatile strategy behind stable interfaces.

- Loop heuristics and backend integrations belong in adapter modules.
- Core orchestration should depend on ports, not concrete tools.
- New backends should be enabled by configuration, not executor rewrites.

### 7) Secure by default

Assume tool and agent output is untrusted and constrain execution.

- Restrict writable paths to repository and artifact directory.
- Enforce command policy and bounded subprocess execution.
- Never log raw secrets; keep safe redaction behavior by default.

## Review Rubric

Before merge, confirm:

- Does the change preserve Scarlet/Santiago ownership boundaries?
- Are contracts validated, versioned, and backward-compatible as intended?
- Is behavior deterministic and testable?
- Are retries/timeouts bounded with explicit stop conditions?
- Are artifacts sufficient to reconstruct execution evidence?
- Are volatile concerns isolated behind interfaces?
- Are security restrictions still enforced end to end?
