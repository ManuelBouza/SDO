# SDO Profiles

Profiles adapt SDO rigor to operational risk. They prevent two opposite failures: bureaucratic overload for simple work and dangerous under-control for critical work.

## Profile selection

Select a profile from the operation taxonomy:

```text
Base Class + Overlays + Risk Tier = SDO Profile
```

The profile determines required artifacts, gates, approval, evidence, validation, and rollback rigor.

## Summary matrix

| Profile | Best for | Approval | Evidence | Review |
|---|---|---|---|---|
| SDO-Lightweight | Low-risk, bounded, repeatable operations | Auto or peer if needed | Minimal | Minimal closure note |
| SDO-Normal | Usual operational changes | Peer or owner | Standard | Brief review |
| SDO-Critical | High-impact, sensitive, privileged, broad, destructive, or hard-to-revert work | Formal, often multi-party | Complete | Formal review |
| SDO-Emergency | Active incident or imminent risk | Accelerated authority | Timeline + critical evidence | Mandatory post-review |
| SDO-Pipeline-Backed | GitOps, IaC, CI/CD, automation-backed operations | PR/policy/environment gates | Pipeline artifacts + refs | Required if failure or risk warrants |

## SDO-Lightweight

For low-risk, bounded, repeatable operations.

```text
Intent → Mini-Spec → Execution → Validation → Minimal Evidence
```

Use when:

- no sensitive data is exposed;
- scope is local or single-target;
- reversibility is R0–R2;
- validation is simple;
- failure has low impact;
- operation is known or preapproved.

Minimum artifacts:

- SDO ID;
- lightweight spec or ticket section;
- short runbook/checklist;
- execution note;
- validation result;
- minimal evidence.

Required controls:

- basic classification;
- basic precheck;
- basic postcheck;
- rollback declaration if state changes.

Should not be used when:

- secrets, PII, regulated data, privileged access, production-critical systems, or broad scope are involved.

## SDO-Normal

For usual operational changes with controlled risk.

```text
Intent → Spec → Risk/Profile → Plan → Runbook → Approval → Execution → Validation → Evidence Closure → Review
```

Use when:

- operation changes state;
- production may be involved but blast radius is limited;
- rollback is defined;
- approval can be handled by owner or peer;
- validation is objective.

Minimum artifacts:

- Operational Spec;
- risk/profile decision;
- plan;
- runbook;
- approval record if applicable;
- execution record;
- validation report;
- evidence package;
- review note.

Required controls:

- prechecks;
- postchecks;
- approval proportional to risk;
- rollback or recovery plan;
- evidence closure.

## SDO-Critical

For high impact, sensitive data, security, production-critical, broad scope, destructive, or hard-to-revert operations.

```text
Intent → Full Spec → Formal Risk/Profile → Detailed Plan → Tested Runbook → Approval → Prechecks → Controlled Execution → Formal Validation → Full Evidence Closure → Formal Review
```

Use when one or more are true:

- production-critical system;
- sensitive or regulated data;
- privileged access;
- IAM/RBAC/security posture change;
- fleet-wide or cross-system scope;
- destructive or hard-to-revert action;
- no reliable preview;
- point of no return;
- significant compliance, cost, or reputational impact.

Minimum artifacts:

- complete Operational Spec;
- detailed risk assessment;
- formal approval record;
- detailed plan;
- tested or reviewed runbook;
- rollback/recovery plan;
- backup/restore evidence if applicable;
- execution record;
- validation report;
- complete evidence package;
- decision log where relevant;
- formal review.

Required controls:

- independent review or owner approval;
- pre-execution readiness gate;
- explicit stop conditions;
- rollback readiness gate;
- strong evidence retention;
- no autoapproval for destructive or privileged changes.

## SDO-Emergency

For active incidents or imminent risk.

```text
Emergency Intent → Minimal Spec → Risk Snapshot → Emergency Approval → Execution → Immediate Validation → Evidence Capture → Post-Execution Review → Retrospective Spec Update
```

Emergency means faster control, not no control.

Use when:

- service is down or degraded;
- security incident requires containment;
- data integrity or availability is at imminent risk;
- delaying action creates more harm than acting.

Minimum before execution:

- emergency intent;
- impact summary;
- proposed action;
- emergency authority or incident commander;
- known rollback/recovery option if available;
- immediate validation criteria.

Minimum after execution:

- timeline;
- decision log;
- executed actions;
- validation result;
- evidence;
- deviations;
- post-incident review;
- retrospective spec update if the change remains.

Required controls:

- emergency approval or break-glass record;
- evidence capture during execution;
- post-review mandatory;
- residual risk documented.

## SDO-Pipeline-Backed

For GitOps, IaC, CI/CD, or automation-backed operations.

```text
Intent / Change Request → Operational Spec → Plan / Diff / Preview → Policy Checks → Approval → Pipeline Execution → Automated Validation → Evidence Package → Review / Archive
```

Use when:

- desired state is versioned;
- execution occurs through CI/CD, IaC, GitOps, or automation;
- plans, diffs, artifacts, logs, and approvals are produced by platform tooling.

Minimum artifacts:

- change request or Operational Spec;
- PR/commit reference;
- plan/diff/preview;
- policy check results;
- environment approval if needed;
- pipeline run ID;
- artifact IDs/hashes;
- validation output;
- rollback or roll-forward strategy;
- evidence package references.

Required controls:

- protected branch or environment where appropriate;
- policy checks;
- automated validation;
- artifact retention;
- rollback or roll-forward job or procedure;
- evidence closure with external references.

Warning:

```text
Pipeline-backed does not mean low risk.
```

A pipeline-backed destructive infrastructure change still requires Critical-level controls.

## Profile escalation rules

Escalate to SDO-Critical when any of these are present:

- secret-bearing or regulated data;
- admin/root/break-glass privilege;
- production-critical target;
- fleet-wide or cross-system scope;
- destructive action;
- R5–R7 reversibility;
- no reliable rollback;
- unbounded cost risk;
- compliance or legal impact.

Use SDO-Emergency only for actual urgency or active incident conditions, not to bypass normal planning.

## Minimum conformance by profile

| Requirement | Lightweight | Normal | Critical | Emergency | Pipeline-Backed |
|---|---|---|---|---|---|
| SDO ID | Required | Required | Required | Required | Required |
| Spec | Minimal | Complete | Complete | Minimal before, completed after | Complete or embedded in change vehicle |
| Risk classification | Simple | Formal | Formal | Snapshot | Formal or policy-backed |
| Approval | Auto/peer | Peer/owner | Formal | Emergency authority | PR/policy/environment |
| Runbook | Short | Required | Detailed/tested | Urgent procedure | Pipeline/job definition |
| Validation | Basic | Formal | Formal + owner/independent where needed | Immediate | Automated checks |
| Evidence | Minimal | Standard | Complete | Timeline + critical evidence | Artifacts/logs/refs |
| Review | Closure note | Brief | Formal | Mandatory | If failure/risk warrants |

## Anti-patterns

- Using Lightweight because the team is in a hurry.
- Treating every pipeline-backed change as safe.
- Calling Emergency to avoid approval.
- Using Critical for everything and creating process fatigue.
- Failing to downgrade rigor for genuinely standard, low-risk work.
- Not escalating when a sensitive overlay appears.
