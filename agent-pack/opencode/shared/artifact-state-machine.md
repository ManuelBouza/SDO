# SDO Artifact State Machine

This state machine defines how file-based SDO operation artifacts move from draft to archive. The orchestrator must evaluate transitions from `manifest.yaml`, referenced files, and stable external references, not from chat history or memory.

## Lifecycle Statuses

| Manifest status | Meaning | Allowed next statuses |
|---|---|---|
| `draft` | Operation identity exists; spec may still be incomplete. | `planned`, `blocked`, `failed`, `superseded` |
| `planned` | Spec/runbook are ready enough for approval evaluation. | `approval-required`, `approved`, `blocked`, `failed`, `superseded` |
| `approval-required` | Human approval, accepted risk, or policy decision is needed. | `approved`, `blocked`, `failed`, `superseded` |
| `approved` | Approval or policy gate passed for the recorded scope. | `executing`, `blocked`, `superseded` |
| `executing` | Bounded execution is active or being recorded. | `validating`, `blocked`, `failed` |
| `validating` | Outcome is being checked against the spec. | `reviewing`, `blocked`, `failed` |
| `reviewing` | Validation, deviations, evidence, and residual risk are being closed. | `archived`, `blocked`, `failed`, `superseded` |
| `archived` | Final artifacts are closed and immutable references are recorded. | None except explicit post-archive correction record |
| `blocked` | Required artifact, policy, approval, evidence, or decision is missing or unsafe. | Previous safe phase, `approval-required`, `failed`, `superseded` |
| `failed` | Operation cannot meet its current objective or safe completion criteria. | `reviewing`, `superseded`, explicit recovery operation |
| `superseded` | Another operation replaces this one. | None except archive/reference updates |

## Artifact Statuses

| Artifact class | Valid statuses |
|---|---|
| `spec` | `missing`, `draft`, `approved`, `superseded` |
| `runbook` | `missing`, `draft`, `approved`, `superseded` |
| `approval` | `missing`, `pending`, `approved`, `rejected`, `hold`, `emergency-approved`, `accepted-risk`, `superseded` |
| `execution` | `missing`, `pending`, `in-progress`, `completed`, `failed`, `rolled-back`, `superseded` |
| `validation` | `missing`, `pending`, `passed`, `failed`, `partial`, `inconclusive`, `superseded` |
| `evidence` | `missing`, `open`, `complete`, `incomplete`, `redaction-required`, `closed`, `superseded` |
| `review` | `missing`, `draft`, `completed`, `accepted-risk`, `superseded` |
| `archive` | `missing`, `ready`, `archived` |

## Normal Phase Transitions

| Phase result | Manifest transition | Required artifact transitions |
|---|---|---|
| `/sdo-new` success | `draft` | spec `missing` to `draft`, evidence `missing` to `open` |
| `/sdo-plan` success | `draft` to `planned` | spec `draft` to `approved` when ready; runbook `missing` to `draft` or `approved` |
| `/sdo-approval-check` needs approval | `planned` to `approval-required` | approval `missing` to `pending` |
| `/sdo-approval-check` approved | `planned` or `approval-required` to `approved` | approval `pending` to `approved`, `emergency-approved`, or `accepted-risk` |
| `/sdo-execute` started | `approved` to `executing` | execution `missing` to `pending` or `in-progress` |
| `/sdo-execute` completed | `executing` to `validating` | execution to `completed`; evidence remains `open` or becomes `complete` |
| `/sdo-validate` passed | `validating` to `reviewing` | validation to `passed`; evidence updated |
| `/sdo-review` completed | `reviewing` | review to `completed` or `accepted-risk`; archive to `ready` when closure gates pass |
| `/sdo-archive` completed | `reviewing` to `archived` | archive to `archived`; evidence to `closed`; final hashes recorded |

## Blocked, Failed, And Superseded Handling

Blocked:

- Use `blocked` when progress is unsafe or impossible because required state is missing, contradictory, stale, or only present in chat history.
- Add or update a manifest blocker with phase, reason, required action, owner, and status.
- Keep the previous artifact statuses unless the blocking phase produced a durable partial artifact.
- `/sdo-continue` may resume only after the blocker is resolved or accepted as risk by an authorized record.

Failed:

- Use `failed` when execution, validation, or closure cannot meet the current spec or safe-state criteria.
- Preserve execution and evidence history, including failed and rolled-back steps.
- Create or update review content before archive whenever possible, even for failed operations.
- Recovery work that changes scope should use a new SDO ID and reference the failed operation.

Superseded:

- Use `superseded` when a new operation replaces the current operation before normal closure.
- Set `operation.superseded_by` in the old manifest and `operation.supersedes` in the new manifest.
- Do not delete or rewrite the superseded operation directory.
- Approved versions and execution history remain preserved in the superseded operation.

## `/sdo-continue` Rules

`/sdo-continue` must choose the next phase from manifest state and dependency gates.

Selection rules:

- If `lifecycle.status` is `blocked`, resume only a safe prerequisite phase or return blocked with the unresolved blocker.
- If `lifecycle.status` is `failed`, route to `review` when review is possible; otherwise return blocked for human recovery decision.
- If `lifecycle.status` is `superseded` or `archived`, do not continue execution phases.
- Prefer `lifecycle.next_allowed_phases` only after verifying dependency gates.
- If `next_allowed_phases` conflicts with artifact statuses, return blocked and require manifest repair.
- If a boundary changed in target, environment, scope, actor identity, tool policy, autonomy, approval, rollback, timing, or evidence, route to `approval-check` or blocked.
- If no safe next phase can be derived from files, return blocked rather than guessing.

Default next phase mapping:

| Manifest status | Candidate next phase |
|---|---|
| `draft` | `plan` |
| `planned` | `approval-check` |
| `approval-required` | `approval-check` |
| `approved` | `execute` |
| `executing` | `execute` or `validate`, depending on execution record status |
| `validating` | `validate` |
| `reviewing` | `review` or `archive`, depending on review/archive readiness |
| `blocked` | prerequisite phase or blocker resolution |
| `failed` | `review` or recovery decision |

## Archive And Closure Requirements

An operation may be archived only when:

- `review-record.md` is completed or accepted-risk is explicitly recorded.
- `evidence-package.yaml` is complete, closed, or incomplete with accepted risk.
- Open blockers are resolved or accepted-risk.
- Deviations have dispositions.
- Final artifact refs and hashes are recorded in `manifest.yaml`.
- External references needed for audit include stable IDs, versions or timestamps, and summaries.
- `archive-record.md` records final status, immutable storage reference when available, final manifest hash, archive timestamp, and archiver identity.

After archive, normal artifact mutation stops. Any correction must preserve the original archive hash and append a correction record rather than silently rewriting history.
