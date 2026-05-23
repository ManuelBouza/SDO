# Run an SDO Operation Safely

Use this guide when you are about to run an operation and need the shortest practical path through SDO. It helps you choose the right level of control, prepare only the artifacts you need, execute within approved boundaries, and close with validation and evidence.

## Quick path

1. Assign an `SDO ID` and write the operational intent.
2. Choose the profile using the quick profile table below.
3. Fill the minimum artifacts for that profile.
4. Confirm approval, execution window, stop conditions, and rollback or recovery path.
5. Run pre-checks and capture current-state evidence.
6. Execute only the approved actions.
7. Stop, escalate, or switch profile if conditions change.
8. Run validation and capture post-execution evidence.
9. Record deviations, rollback actions, or residual risk.
10. Close with the required review for the selected profile.

## Choose the profile quickly

| Situation | Use this profile |
|---|---|
| Low-risk, bounded, repeatable work with simple validation | `SDO-Lightweight` |
| Usual operational change with limited blast radius and clear rollback | `SDO-Normal` |
| Production-critical, privileged, destructive, sensitive-data, broad-scope, or hard-to-revert work | `SDO-Critical` |
| Active incident or imminent harm where delay increases impact | `SDO-Emergency` |
| GitOps, IaC, CI/CD, or automation-backed execution | `SDO-Pipeline-Backed` |

If any Critical condition appears, use `SDO-Critical` controls even when the execution is automated. Pipeline-backed does not mean low risk.

Use `SDO-Emergency` only for real urgency. Emergency means faster control, not no control.

## Minimum artifacts by profile

| Profile | Minimum practical package |
|---|---|
| `SDO-Lightweight` | `SDO ID`, short intent, short checklist or runbook, execution note, validation result, minimal evidence. |
| `SDO-Normal` | Operational Spec, profile/risk decision, runbook, approval if applicable, execution record, validation report, evidence package, brief review note. |
| `SDO-Critical` | Complete Operational Spec, detailed risk assessment, formal approval, reviewed or tested runbook, rollback/recovery plan, execution record, validation report, complete evidence package, formal review. |
| `SDO-Emergency` | Before execution: emergency intent, impact, proposed action, emergency authority, known rollback/recovery option, immediate validation criteria. After execution: timeline, decision log, executed actions, validation, evidence, deviations, post-review. |
| `SDO-Pipeline-Backed` | Change request or Operational Spec, PR/commit or change reference, plan/diff/preview, policy checks, approval gate, pipeline run ID, validation output, rollback or roll-forward strategy, evidence references. |

For full rules, see [`../reference/profiles.md`](../reference/profiles.md).

## Operator decision points

Stop before execution when:

- the intent, target, or expected outcome is unclear;
- approval is missing or does not match the selected profile;
- rollback or recovery is undefined for a state-changing operation;
- pre-checks fail or current-state evidence contradicts the spec;
- the execution window, scope, or boundaries are not explicit.

Ask for approval when:

- the operation changes state in a shared or production environment;
- the action uses elevated privilege;
- the blast radius reaches another team, service, customer, or data domain;
- the plan deviates from an approved spec or runbook.

Escalate when:

- validation cannot prove success;
- rollback fails or introduces new risk;
- evidence is missing for a material step;
- an unexpected dependency, outage, security issue, or data risk appears.

Switch to `SDO-Critical` when privileged, destructive, sensitive-data, production-critical, broad-scope, compliance, or hard-to-revert conditions appear.

Switch to `SDO-Emergency` only when there is an active incident or imminent harm and waiting for the normal path would increase impact.

## Evidence and rollback reminders

- Capture evidence before, during, and after execution; do not wait until memory fades.
- Keep evidence tied to the `SDO ID`, action IDs, timestamps, actors, and validation criteria.
- Redact or restrict sensitive evidence instead of omitting it.
- Know the rollback trigger before executing each material step.
- Record whether rollback was available, used, skipped, or impossible.
- If rollback is impossible or risky, document the recovery path and residual risk before approval.

## AI and automation

AI or automation can help draft specs, generate runbook steps, summarize logs, run checks, or assemble evidence. The operator remains responsible for approval, execution boundaries, validation, evidence quality, deviations, and escalation decisions.

Do not let an AI tool or automated job expand scope, bypass approval, hide uncertainty, or execute outside the approved runbook.

## Templates and examples

Start from the smallest artifact set that fits the chosen profile:

- [`../templates/operational-spec.md`](../templates/operational-spec.md)
- [`../templates/runbook.md`](../templates/runbook.md)
- [`../templates/execution-record.md`](../templates/execution-record.md)
- [`../templates/validation-report.md`](../templates/validation-report.md)
- [`../templates/evidence-package.yaml`](../templates/evidence-package.yaml)
- [`../templates/rollback-plan.md`](../templates/rollback-plan.md)
- [`../templates/review-record.md`](../templates/review-record.md)

Use examples for shape, not as mandatory ceremony:

- [`../examples/`](../examples/)
- [`../examples/observe-linux-service/`](../examples/observe-linux-service/)
- [`../examples/emergency-remediation/`](../examples/emergency-remediation/)

SDO is proportional control. Add rigor when risk justifies it; do not add paperwork that does not improve approval, execution safety, validation, rollback readiness, or auditability.
