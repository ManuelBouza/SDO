---
name: sdo-shared-protocol
description: "Trigger: SDO phase, gates, phase result envelope, agent-pack. Coordinate SDO phases and fail-closed protocol checks."
license: Apache-2.0
metadata:
  author: gentleman-programming
  version: "0.1"
---

# SDO Shared Protocol

## Activation Contract

Use this skill when coordinating any SDO phase, validating phase gates, returning phase result envelopes, resuming operations, or operating under the SDO OpenCode agent-pack.

## Hard Rules

- Read the authoritative shared assets before phase work: `agent-pack/opencode/shared/phase-result-envelope.yaml`, `agent-pack/opencode/shared/preflight-checklist.md`, `agent-pack/opencode/shared/dependency-gates.md`, and `agent-pack/opencode/shared/artifact-state-machine.md`.
- Fail closed. If required state is missing, stale, contradictory, or only present in chat history, return `blocked` or `needs-human`.
- Do not trust chat history, agent memory, or unstored output as proof that a gate passed.
- Return envelope-compatible results. Include `status`, `stop_reason`, artifact refs, evidence refs, policy decisions, approvals required, deviations, autonomy, risk, validation, and next allowed phases when applicable.
- Do not perform operational execution unless the current phase, autonomy level, approval boundary, and tool policy explicitly allow it.
- Treat the manifest and referenced operation files as routing inputs; the envelope can propose next phases, but gates still decide.

## Decision Gates

| Condition | Action |
|---|---|
| Gate is satisfied from files or stable external refs | Continue the requested phase. |
| Human approval, clarification, or risk acceptance is required | Return `needs-human`. |
| Required artifact, policy, dependency, or boundary is missing | Return `blocked`. |
| Work was attempted and cannot meet the contract | Return `failed`. |

## Execution Steps

1. Load the shared protocol files listed above.
2. Identify runtime mode, orchestrator, `SDO ID`, current phase, manifest state, autonomy ceiling, risk profile, target, actors, and tool policy.
3. Run universal preflight before any phase and execution preflight before any tool call or state-changing step.
4. Evaluate dependency gates from durable artifacts and stable references.
5. Produce a phase result matching `phase-result-envelope.yaml`.

## Output Contract

Return a compact phase result envelope or an envelope-compatible summary. Never hide blockers, deviations, missing approvals, failed validation, or unsafe state in prose only.

## References

- `agent-pack/opencode/shared/phase-result-envelope.yaml`
- `agent-pack/opencode/shared/preflight-checklist.md`
- `agent-pack/opencode/shared/dependency-gates.md`
- `agent-pack/opencode/shared/artifact-state-machine.md`
