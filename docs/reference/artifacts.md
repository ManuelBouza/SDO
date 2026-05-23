# SDO Artifacts

Artifacts make SDO usable. They turn the methodology into something that can be reviewed, executed, validated, audited, and improved.

## Artifact principle

The unit of traceability is the `SDO ID`, not a single document.

```text
SDO ID → Spec → Plan → Runbook → Execution Record → Validation Report → Evidence Package → Review
```

SDO should reference existing systems of record instead of copying them. A ticket, PR, pipeline run, audit event, or monitoring check can be part of the artifact graph.

## Core artifacts

| Artifact | Purpose | Required? |
|---|---|---|
| Intent Record | Captures what is needed and why. | Always, can be embedded in spec/ticket. |
| Operational Spec | Source of truth for approved intent, scope, success criteria, risk, and recovery. | Always, lightweight allowed. |
| Execution Plan | Defines strategy, order, dependencies, window, and responsibilities. | Normal+. |
| Runbook | Defines executable or manual steps. | Always, can be short. |
| Execution Record | Records what actually happened. | Always. |
| Validation Report | Proves the final state satisfies the spec. | Always. |
| Review Record | Captures closure, deviations, lessons, and follow-ups. | Always, can be minimal. |

## Transversal artifacts

| Artifact | Purpose | Required? |
|---|---|---|
| Risk Assessment | Records base class, overlays, risk tier, and profile. | Always, detail varies. |
| Approval Record | Records who approved what, when, and under what conditions. | Risk-dependent. |
| Evidence Package | Indexes evidence references, hashes, timestamps, actors, and closure status. | Always, detail varies. |
| Rollback / Recovery Plan | Defines safe-state strategy. | Always for state-changing work. |
| Decision Log | Records important decisions, especially under uncertainty. | Critical/Emergency. |
| Traceability Index | Links all internal and external references. | Recommended; required for complex work. |
| Sensitive Data Handling Record | Describes redaction, access, and retention decisions. | Required when evidence/data is sensitive. |

## Artifact lifecycle

```text
Intent Record
  └── produces → Operational Spec
        ├── constrained_by → Risk Assessment
        ├── produces → Execution Plan
        │      ├── requires → Approval Record
        │      └── produces → Runbook
        │              ├── executes_as → Executor Binding
        │              └── emits → Execution Record
        ├── validated_by → Validation Report
        ├── evidenced_by → Evidence Package
        └── closed_by → Review Record
```

## Operational Spec

The Operational Spec defines the approved operational intent.

Minimum fields:

- SDO ID;
- title;
- owner;
- status;
- environment;
- base class;
- profile;
- intent;
- scope and exclusions;
- current state;
- desired state;
- acceptance criteria;
- constraints;
- risk summary;
- validation criteria;
- rollback or recovery strategy;
- evidence requirements;
- traceability references.

Template: [`../templates/operational-spec.md`](../templates/operational-spec.md)

## Execution Plan

The Execution Plan defines how the spec will be achieved safely.

Minimum fields:

- strategy;
- sequence;
- dependencies;
- target expansion;
- execution window;
- responsible actors;
- preview/dry-run/diff approach;
- prechecks;
- postchecks;
- rollback/recovery approach;
- point of no return;
- known risks and mitigations.

For SDO-Lightweight, the plan may be embedded in the Operational Spec or Runbook.

## Runbook

The Runbook turns the plan into executable or manual steps.

Minimum fields:

- runbook ID;
- SDO ID or spec ID;
- owner;
- execution mode;
- safety controls;
- parameters;
- prechecks;
- execution steps;
- expected results;
- evidence per step;
- stop conditions;
- rollback hooks;
- postchecks;
- closeout conditions.

Template: [`../templates/runbook.md`](../templates/runbook.md)

## Execution Record

The Execution Record proves what actually happened.

Minimum fields:

- execution ID;
- SDO ID;
- approved plan reference;
- approval reference;
- executor identity;
- start and end timestamps;
- planned vs actual action mapping;
- step results;
- exit codes or response status where applicable;
- deviations;
- evidence references;
- rollback execution status;
- final status.

Template: [`../templates/execution-record.md`](../templates/execution-record.md)

## Validation Report

The Validation Report proves whether the observed final state satisfies the spec.

Minimum fields:

- SDO ID;
- validation criteria;
- expected results;
- observed results;
- validation method;
- evidence references;
- impact assessment;
- pass/fail/partial decision;
- residual risk;
- follow-up actions.

Template: [`../templates/validation-report.md`](../templates/validation-report.md)

## Evidence Package

The Evidence Package is an index, not a log dump.

Minimum fields:

- SDO ID;
- package ID;
- operation summary;
- traceability references;
- actor identities;
- timeline;
- evidence index;
- hashes;
- sensitivity classification;
- retention;
- access control;
- closure status.

Template: [`../templates/evidence-package.yaml`](../templates/evidence-package.yaml)

## Rollback / Recovery Plan

The Rollback / Recovery Plan defines how the system reaches an accepted safe state if execution fails or creates unacceptable impact.

Minimum fields:

- SDO ID;
- reversibility level;
- recovery strategy;
- target safe state;
- triggers;
- stop conditions;
- point of no return;
- prerequisites;
- backup/snapshot/previous version references;
- rollback actions;
- validation after rollback;
- approval or risk acceptance.

Template: [`../templates/rollback-plan.md`](../templates/rollback-plan.md)

## Review Record

The Review Record closes the learning loop.

Minimum fields:

- final status;
- what changed;
- what worked;
- what failed;
- deviations;
- rollback/recovery status;
- evidence closure status;
- lessons learned;
- follow-up actions;
- owner and due dates.

Template: [`../templates/review-record.md`](../templates/review-record.md)

## Artifact requirements by profile

| Artifact | Lightweight | Normal | Critical | Emergency | Pipeline-Backed |
|---|---|---|---|---|---|
| Intent | Brief | Required | Required | Emergency intent | Change request |
| Spec | Minimal | Complete | Complete | Minimal before, complete after | Spec or PR/change description |
| Risk | Simple | Formal | Detailed | Snapshot | Policy/formal |
| Plan | Embedded | Required | Detailed | Minimal | Plan/diff/preview |
| Runbook | Short | Required | Tested/reviewed | Urgent procedure | Pipeline/job definition |
| Approval | Optional/auto | Peer/owner | Formal | Emergency authority | PR/environment/policy |
| Execution Record | Required | Required | Required | Required | Pipeline run + record |
| Validation | Basic | Formal | Formal | Immediate | Automated checks |
| Evidence | Minimal | Standard | Complete | Timeline + critical evidence | Artifacts/log references |
| Review | Closure note | Brief | Formal | Mandatory | If failure/risk warrants |

## Traceability rule

The `SDO ID` links all artifacts, external tickets, PRs, pipeline runs, evidence, approvals, and review records.

Use stable references:

```yaml
external_refs:
  ticket: ""
  pr: ""
  pipeline_run: ""
  monitoring_check: ""
  audit_event: ""
  evidence_storage: ""
```

## Anti-patterns

- Duplicating an entire ticket in Markdown instead of linking it.
- Treating screenshots as sufficient evidence when audit logs exist.
- Keeping runbooks stable but overwriting execution history.
- Writing rollback as free text without executable or verifiable recovery.
- Using templates as bureaucracy instead of adapting detail to profile.
- Storing secrets in specs, runbooks, logs, or evidence packages.
- Closing an operation without mapping planned actions to executed actions.
