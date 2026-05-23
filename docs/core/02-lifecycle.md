# SDO Lifecycle

The SDO lifecycle describes how an operational need moves from intent to controlled execution, validation, evidence, and review.

## Compact lifecycle

Use the compact lifecycle for communication and training:

```text
Intent → Spec → Plan → Runbook → Execution → Validation → Review
```

## Full lifecycle

Use the full lifecycle for implementation, tooling, governance, and audit:

```text
Intent
→ Operational Spec
→ Risk & Profile
→ Execution Plan
→ Planned Actions
→ Runbook / Executor Binding
→ Approval
→ Pre-Execution Check
→ Execution
→ Validation
→ Evidence Closure
→ Review / Recovery / Archive
```

The full lifecycle does not mean every operation needs heavyweight documentation. It means every operation has a place where each concern is handled, even if the artifact is lightweight.

For AI or agent-driven operations, use the agentic overlay in [`04-agentic-sdo.md`](04-agentic-sdo.md): Intent/Spec → Plan/Runbook → Approval Gate → Bounded Execution → Independent Validation → Evidence/Archive → Review/Learning.

## Phase details

### 1. Intent

Captures the operational need.

Inputs:

- request;
- alert;
- incident;
- maintenance need;
- compliance requirement;
- operator observation.

Outputs:

- SDO ID;
- objective;
- reason;
- urgency;
- initial scope;
- expected outcome.

Exit gate:

```text
The operation has a clear objective and a traceable owner or requester.
```

### 2. Operational Spec

Turns intent into a verifiable operational contract.

Outputs:

- target scope;
- current state summary;
- desired state;
- constraints;
- success criteria;
- failure criteria;
- validation expectations;
- recovery expectations.

Exit gate:

```text
The spec is specific enough to plan, review, validate, and close.
```

### 3. Risk & Profile

Classifies the operation and selects the required rigor.

Outputs:

- base class;
- overlays;
- risk tier;
- SDO profile;
- approval requirements;
- evidence expectations;
- rollback rigor.

Exit gate:

```text
The selected profile is justified by risk, not habit.
```

### 4. Execution Plan

Defines the operational strategy.

Outputs:

- sequence;
- dependencies;
- affected systems;
- execution window;
- roles;
- target expansion;
- preview or dry-run plan;
- rollback or recovery approach.

Exit gate:

```text
The plan explains how the spec will be achieved safely.
```

### 5. Planned Actions

Breaks the plan into approved operational actions.

Each Planned Action should define:

- precondition;
- executor;
- target;
- action;
- expected result;
- validation;
- evidence;
- stop condition;
- rollback or recovery mapping.

Exit gate:

```text
Every action is clear, bounded, and reviewable.
```

### 6. Runbook / Executor Binding

Converts Planned Actions into executable or manual procedure.

Outputs:

- runbook;
- scripts or command references;
- API calls;
- SQL scripts;
- pipeline jobs;
- GUI steps;
- parameter list;
- executor capability notes.

Exit gate:

```text
The procedure can be followed without improvising beyond the approved plan.
```

### 7. Approval

Authorizes execution according to risk.

Approval may be:

- auto-approved by policy;
- peer review;
- service owner approval;
- security or data owner approval;
- incident commander approval;
- protected environment approval;
- change authority approval.

Exit gate:

```text
The approver, scope, conditions, and timestamp are recorded.
```

### 8. Pre-Execution Check

Confirms execution can safely start now.

Checks may include:

- identity and permissions;
- current state;
- health;
- backup or snapshot;
- maintenance window;
- monitoring availability;
- target list;
- dry-run or diff;
- rollback readiness.

Exit gate:

```text
The operation can proceed, hold, abort, or require updated approval.
```

### 9. Execution

Performs approved actions against real targets.

Rules:

- execute step by step;
- record actual actions;
- compare planned vs actual;
- classify deviations;
- respect stop conditions;
- do not expand scope implicitly.

Exit gate:

```text
Every executed action has a result and evidence reference.
```

### 10. Validation

Verifies that the observed state satisfies the Operational Spec.

Validation may include:

- health checks;
- service status;
- logs;
- metrics;
- SQL queries;
- API checks;
- data integrity checks;
- security checks;
- owner confirmation;
- monitoring period.

Exit gate:

```text
The operation is passed, failed, partial, rolled back, or accepted with residual risk.
```

### 11. Evidence Closure

Confirms enough evidence exists to close.

The closure gate verifies:

- SDO ID exists;
- approved spec is referenced;
- planned actions are accounted for;
- executed actions map to planned actions;
- deviations are classified;
- validation has expected and observed results;
- rollback or recovery status is documented;
- evidence contains no secrets;
- retention and access rules are defined.

Exit gate:

```text
The operation can be audited without reconstructing it from memory.
```

### 12. Review / Recovery / Archive

Closes the operational learning loop.

Review may be lightweight or formal depending on profile and outcome.

Outputs:

- final status;
- lessons learned;
- follow-up actions;
- runbook improvements;
- risk updates;
- archive references.

Exit gate:

```text
The operation is closed, archived, and future improvement is captured.
```

## Transversal controls

The lifecycle includes controls that apply across phases.

| Control | Applies from | Applies until | Purpose |
|---|---|---|---|
| Risk / Approval | Spec | Closure | Select profile and authority. |
| Evidence | Intent | Archive | Prove lifecycle and outcome. |
| Rollback / Recovery | Spec | Review | Preserve safe-state capability. |
| AI Governance | Intent | Closure | Bound AI role, autonomy, and evidence. |

## Profile variants

### SDO-Lightweight

```text
Intent → Lightweight Spec → Pre-approved Runbook → Execution → Validation → Evidence Closure
```

Used for bounded, low-risk operations.

### SDO-Normal

```text
Intent → Spec → Risk/Profile → Plan → Runbook → Approval → Execution → Validation → Evidence Closure → Review
```

Used for usual operational changes.

### SDO-Critical

```text
Intent → Full Spec → Formal Risk/Profile → Detailed Plan → Tested Runbook → Approval → Prechecks → Controlled Execution → Formal Validation → Full Evidence Closure → Formal Review
```

Used for production-critical, data, security, broad-scope, destructive, or hard-to-revert operations.

### SDO-Emergency

```text
Emergency Intent → Minimal Spec → Risk Snapshot → Emergency Approval → Execution → Immediate Validation → Evidence Capture → Post-Execution Review → Retrospective Spec Update
```

Used to restore service, contain impact, or respond to imminent risk. Emergency accelerates approval; it does not remove evidence or review.

### SDO-Pipeline-Backed

```text
Intent / Change Request → Operational Spec → Plan / Diff / Preview → Policy Checks → Approval → Pipeline Execution → Automated Validation → Evidence Package → Review / Archive
```

Used for GitOps, IaC, CI/CD, or mature automation flows.

## Source-of-truth rule

The Operational Spec is the source of truth for approved intent, scope, limits, success criteria, risk acceptance, and recovery strategy.

Evidence is the source of truth for what actually happened.
