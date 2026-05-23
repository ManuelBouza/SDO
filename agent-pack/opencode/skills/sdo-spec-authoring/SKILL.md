---
name: sdo-spec-authoring
description: "Trigger: SDO Operational Spec, agentic contract, spec drafting. Draft specs with autonomy, approval, validation, and evidence controls."
license: Apache-2.0
metadata:
  author: gentleman-programming
  version: "0.1"
---

# SDO Spec Authoring

## Activation Contract

Use this skill when drafting, refining, reviewing, or repairing SDO Operational Specs as agentic contracts for humans, agents, pipelines, or platforms.

## Hard Rules

- Read the authoritative spec assets before authoring: `docs/templates/operational-spec.md`, `docs/core/04-agentic-sdo.md`, `docs/reference/autonomy-levels.md`, `docs/reference/tool-execution-policy.md`, and `docs/reference/human-approval-model.md`.
- The spec must declare intent, scope, target/environment, profile/risk, autonomy level, actor identity, executor identity, allowed tools/actions, forbidden tools/actions, approval boundary, validation contract, evidence contract, rollback/recovery, and stop conditions.
- Use the lowest autonomy level that safely achieves the operation.
- Do not infer authority, expand scope, approve execution, or invent missing policy from prose or chat history.
- If any invariant is missing, ambiguous, or contradictory, return `needs-human` or `blocked` instead of guessing.

## Decision Gates

| Condition | Action |
|---|---|
| Intent, target, environment, or scope is ambiguous | Return `needs-human` for clarification. |
| Autonomy, actor, executor, approval, tool, validation, evidence, or rollback invariant is missing | Return `blocked` until the spec is completed. |
| State-changing execution is requested without required approval/policy | Return `needs-human` or `blocked` according to the missing gate. |
| Spec is complete enough for the selected profile | Mark it draft or ready for approval according to manifest state. |

## Execution Steps

1. Load the spec template and agentic SDO references listed above.
2. Classify the operation profile, base class, risk tier, environment, reversibility, and autonomy ceiling from durable context.
3. Fill the Operational Spec as a contract: intended outcome, non-goals, included/excluded scope, current and desired state, constraints, risks, execution strategy, validation, rollback/recovery, evidence, and traceability.
4. Bind every proposed tool or action to allowed and forbidden boundaries, approval requirements, evidence expectations, and stop conditions.
5. Surface missing invariants as blockers or human questions; do not hide them in TODO prose if they gate execution.

## Output Contract

Return the spec path or draft content, status, missing invariants, approval needs, policy gaps, evidence requirements, validation contract, rollback/recovery status, and whether the next safe phase is `plan`, `approval-check`, or `blocked`.

## References

- `docs/templates/operational-spec.md`
- `docs/core/04-agentic-sdo.md`
- `docs/reference/autonomy-levels.md`
- `docs/reference/tool-execution-policy.md`
- `docs/reference/human-approval-model.md`
