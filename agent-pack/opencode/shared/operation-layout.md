# SDO Operation Layout

File-based artifacts are the MVP source of truth for SDO operations. Agents may use Engram or other memory systems for indexing and continuity, but routing, audit, approval, execution, validation, review, and archive decisions must be derived from the manifest and files in the operation directory.

## Default Root

```text
sdo/operations/<SDO ID>/
```

Each operation has exactly one operation directory. The directory name must match the `operation.sdo_id` value in `manifest.yaml`.

## SDO ID Convention

Use this default format for new operations:

```text
SDO-YYYYMMDD-NNN
```

Rules:

- `YYYYMMDD` is the UTC creation date.
- `NNN` is a zero-padded sequence for that date, starting at `001`.
- IDs are uppercase, ASCII-only, and path-safe.
- IDs must not include environment names, customer names, usernames, ticket summaries, or secrets.
- If an external change ID already exists, store it as an external reference; do not replace the SDO ID with it.
- A superseding operation receives a new SDO ID and references the superseded operation.

## Required Layout

```text
sdo/operations/SDO-YYYYMMDD-NNN/
  manifest.yaml
  operational-spec.md
  runbook.md
  approval-record.yaml
  execution-record.md
  validation-report.md
  evidence-package.yaml
  review-record.md
  archive-record.md
  evidence/
```

`manifest.yaml` is required for every operation. Other files may begin as missing, draft, or pending depending on phase and profile, but their expected paths should be listed in the manifest from operation creation.

## Required Files By Phase

| Phase | Required before phase starts | Produced or updated |
|---|---|---|
| `new` | Intent and selected artifact root. | `manifest.yaml`, draft `operational-spec.md`. |
| `plan` | Manifest with confirmed SDO ID, scope, target, and profile placeholders. | `operational-spec.md`, `runbook.md`, manifest dependency state. |
| `approval-check` | Spec and runbook or profile-approved lightweight equivalents. | `approval-record.yaml`, blocker/deviation updates. |
| `execute` | Approved or policy-eligible spec/runbook, approval or policy refs, rollback or recovery readiness, evidence package open. | `execution-record.md`, `evidence-package.yaml`, evidence refs. |
| `validate` | Execution record or planned read-only observation record, evidence refs. | `validation-report.md`, evidence refs. |
| `review` | Validation report, evidence package, deviations and blockers. | `review-record.md`, final deviation disposition. |
| `archive` | Review record, complete or accepted-risk evidence package, closed blockers. | `archive-record.md`, final manifest hashes and archive state. |

## Required Files By Profile

| File | Lightweight | Normal | Critical | Emergency | Pipeline-Backed |
|---|---|---|---|---|---|
| `manifest.yaml` | Required | Required | Required | Required | Required |
| `operational-spec.md` | Minimal | Complete | Complete | Minimal before, complete after | Spec, PR, or change description reference plus manifest |
| `runbook.md` | Short | Required | Tested or reviewed | Urgent procedure | Pipeline/job definition reference plus safety steps |
| `approval-record.yaml` | Optional or policy | Required when risk/tool policy requires | Formal approval required | Emergency authority required | PR/environment/policy approval reference |
| `execution-record.md` | Required | Required | Required | Required | Pipeline run reference plus record |
| `validation-report.md` | Basic | Required | Formal | Immediate | Automated check references |
| `evidence-package.yaml` | Minimal | Standard | Complete | Timeline plus critical evidence | Artifact/log references |
| `review-record.md` | Closure note | Brief | Formal | Mandatory | Required for failure, deviation, or accepted risk |
| `archive-record.md` | Required at closure | Required at closure | Required at closure | Required at closure | Required at closure |

## Optional Files

Optional files should be referenced from `manifest.yaml` when they affect routing, approval, execution, validation, evidence, or closure.

Common optional files:

- `execution-plan.md` for Normal+ operations when the plan is separate from the spec or runbook.
- `rollback-plan.md` for state-changing work with non-trivial recovery.
- `decision-log.md` for Critical, Emergency, or ambiguous operations.
- `traceability-index.yaml` for complex external reference graphs.
- `sensitive-data-handling.md` when evidence or target data is sensitive.
- `evidence/<evidence-id>.*` for redacted evidence extracts or exported artifacts.

## Source Of Truth

The source of truth is:

```text
manifest.yaml + referenced operation files + stable external system references
```

The source of truth is not:

- chat history;
- agent memory;
- unstored command output;
- unreferenced local files;
- informal statements that are not captured in an artifact or external system of record.

Engram or hybrid memory may index summaries, decisions, next actions, and discoveries. It must not be the only place where operation state, approvals, execution history, validation results, or evidence closure are stored.

## Safe Update Rules

- Append or merge new evidence; do not delete evidence entries to hide failed, partial, or superseded results.
- Never rewrite execution history silently. Corrections must be appended with timestamp, actor, reason, and replacement reference.
- Preserve approved versions of specs, runbooks, approvals, and policies. New versions must reference the approved version they replace.
- Use `superseded_by` and `supersedes` references instead of overwriting old operation directories.
- Keep rejected, failed, blocked, and rolled-back states visible in the manifest and relevant records.
- Recompute hashes after intentional artifact updates and record the update in the manifest timestamp or archive record.
- Do not edit archived operations except to add an explicit post-archive correction record that preserves the original archive hash.

## External References

External systems of record are allowed when the manifest stores stable references and enough state to audit the decision.

Store references in manifest sections such as `external_refs`, `approval_refs`, `policy_refs`, and `evidence_refs`.

Recommended fields for each external reference:

```yaml
external_ref_id: "EXT-001"
type: "ticket | pr | pipeline | itsm | monitoring | audit-log | storage | other"
system: ""
uri: ""
stable_id: ""
version: ""
status: ""
captured_at_utc: ""
summary: ""
```

Rules:

- Prefer immutable run IDs, event IDs, object versions, commit SHAs, PR numbers, ITSM change IDs, or audit event IDs.
- Do not paste entire tickets, PRs, logs, or pipeline output into SDO files when a stable reference is sufficient.
- If the external system is mutable, record the version, timestamp, status, and summary used for the SDO decision.
- If the external system may become unavailable, export a redacted evidence extract under `evidence/` and reference its hash.

## Redaction And Large Evidence

Evidence packages are indexes, not raw log dumps.

Rules:

- Store summaries, stable source refs, hashes, actors, timestamps, request IDs, run IDs, event IDs, and redaction status by default.
- Do not store secrets, tokens, private keys, raw `.env` values, unnecessary personal data, or unredacted sensitive logs.
- Put redacted extracts under `evidence/` only when they materially support execution, validation, approval, or audit.
- For large artifacts, store an external storage reference, size, checksum, sensitivity classification, retention policy, and access-control owner.
- Mark evidence as `redaction-required` or `blocked` when safe evidence cannot yet be produced.
- Closure may proceed with incomplete evidence only when the review record and manifest capture explicit accepted risk.
