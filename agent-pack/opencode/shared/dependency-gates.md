# SDO OpenCode Dependency Gates

Dependency gates prevent unsafe phase skipping. The orchestrator evaluates gates from files, manifest state, and stable external references. It must not trust chat history as proof that a gate passed.

## Gate Rules

- Gates fail closed when required state is missing, stale, contradictory, or only present in chat history.
- The operation manifest is the routing index for `sdo/operations/<SDO ID>/`.
- A phase result envelope may propose `next_allowed_phases`, but the orchestrator must still evaluate these gates.
- External systems of record are allowed only when the manifest stores stable references and enough state to audit the decision.
- Any boundary change in target, environment, identity, tool, scope, timing, autonomy, approval, rollback, or evidence returns the operation to `approval-check` or a blocked state.

## Phase Prerequisites

| Requested phase | Required state before start | Blocked when |
|---|---|---|
| `init` | Repository or workspace is accessible; OpenCode runtime can read/write the selected install path. | Asset path is not writable, runtime mode cannot be determined, or existing config would be overwritten without approval. |
| `new` | Intent exists; artifact store is selected; `SDO ID` can be created or confirmed. | Intent is absent, target/environment is unknowable, or operation duplicates an active manifest without an explicit continue decision. |
| `plan` | Manifest exists; operation identity is confirmed; intent and target scope are recorded. | Missing `SDO ID`, missing spec draft, unknown profile/risk/autonomy, or ambiguous target scope. |
| `approval-check` | Spec/runbook or equivalent lightweight artifacts exist; requested autonomy, tool boundaries, target, environment, actor identities, validation, evidence, and rollback expectations are declared. | Approval requirement cannot be determined, approver authority is missing, tool policy is absent, or A6 lacks formal policy reference. |
| `execute` | Spec and runbook are approved or policy-eligible; approval/policy gate passed; execution policy exists; rollback or recovery is ready when required; evidence capture is ready. | Approval is missing for A5, A6 policy is missing, requested action is outside policy, target or identity drifted, preview failed, stop condition is active, or secrets appear. |
| `validate` | Execution record exists or read-only validation is explicitly planned; evidence refs exist for executed or observed actions; validation contract exists. | No execution/observation record, validation criteria are absent, validator is not independent enough, or evidence needed for validation is missing. |
| `review` | Validation report exists; evidence package is updated; deviations and residual risks are listed. | Validation is failed or inconclusive without disposition, deviations are hidden or unresolved, evidence redaction is pending, or rollback/recovery status is unknown. |
| `archive` | Review record exists; final evidence package is complete or accepted-risk; blockers and deviations have dispositions; manifest is ready to freeze. | Review is missing, evidence is incomplete without acceptance, blockers remain open, or final artifact refs/hashes are missing. |
| `continue` | Manifest exists; last phase result or lifecycle state identifies candidate next phases. | No manifest exists, manifest conflicts with artifacts, next phase cannot be determined from file state, or previous stop reason remains unresolved. |

## A4/A5/A6 Execution Distinctions

| Autonomy | Execution gate | Required proof | Never allowed |
|---|---|---|---|
| `A4` | Read-only diagnostics and validation only. | Read-only identity, allowlisted non-mutating tools, explicit target/environment, logging/evidence, stop conditions. | Mutating state, privilege escalation, broad sensitive-data reads, or treating diagnostics as approval for change. |
| `A5` | Bounded state-changing execution with per-operation human approval. | Approval record, approved planned actions, allowlisted tools/parameters, executor identity, rollback/recovery, validation, evidence. | Unapproved production changes, destructive actions outside policy, target expansion, or self-approval for high-risk work. |
| `A6` | Policy-bound autonomous execution for standard repeatable operations. | Formal preapproved policy, narrow scope, allowlisted actions, monitoring, automatic stop, audit trail, periodic review. | Novel, critical, destructive, sensitive, privileged, or weakly recoverable actions without human approval. |

## Gate Outputs

When a gate blocks progress, return a phase result envelope with:

- `status: blocked` when required artifact, dependency, policy, or runtime state is missing.
- `status: needs-human` when a human approval, clarification, risk acceptance, or authority decision is required.
- `stop_reason` that names the missing or unsafe condition.
- `approvals_required`, `policy_decisions`, `deviations`, or `evidence_refs` populated when relevant.
- `next_allowed_phases` limited to safe recovery or prerequisite phases.
