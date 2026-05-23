# Gates and Controls

SDO gates are small, verifiable, and proportional to risk.

A gate is not necessarily a meeting or a human approval. A gate can be a checklist, diff, dry-run, policy check, approval, metric threshold, stop condition, validation result, or evidence closure check.

## Central rule

An SDO operation can advance only when it has enough specification, classified risk, verifiable plan, proportional authorization, approved prechecks, stop/rollback conditions, post-validation, and closure evidence.

## Anti-bureaucracy rule

Control increases with impact, irreversibility, privilege, data sensitivity, production scope, automation, and uncertainty — not with organizational habit.

## Gate outcomes

Every gate should produce one of these outcomes:

| Outcome | Meaning |
|---|---|
| proceed | Continue to the next phase or action. |
| hold | Pause until missing information or condition is resolved. |
| abort | Stop the operation before execution or before further impact. |
| rollback | Start rollback or recovery flow. |
| escalate | Require higher authority or broader review. |
| accept-risk | Continue with explicit risk acceptance. |
| close | Close the operation. |
| reopen | Reopen because evidence, validation, or closure is insufficient. |

## Core gates

| Gate | Purpose | Evaluated when |
|---|---|---|
| Spec Gate | Ensure intent and spec are clear enough to plan. | Before planning. |
| Risk/Profile Gate | Select profile and required rigor. | Before plan/runbook approval. |
| Preview/Plan Gate | Understand what will change before changing it. | Before approval/execution. |
| Approval Gate | Authorize according to risk. | Before execution. |
| Pre-Execution Gate | Confirm execution can safely start now. | Immediately before execution. |
| Continue/Stop Gate | Decide whether to continue, pause, abort, rollback, or escalate. | During execution. |
| Rollback Gate | Decide and control rollback or recovery. | When triggers or failure occur. |
| Validation Gate | Confirm observed state satisfies the spec. | After execution or rollback. |
| Evidence/Closure Gate | Confirm sufficient evidence exists to close. | Before closure. |
| Automation Eligibility Gate | Decide if a repeated operation can become standard or automated. | After review. |

## Gate definitions

### Spec Gate

Objective: prevent execution from vague intent.

Inputs:

- Operational Intent;
- Operational Spec draft;
- scope and exclusions;
- expected state;
- validation criteria.

Passes when:

- objective is clear;
- scope is bounded;
- desired state is testable;
- non-goals are known;
- owner/requester is traceable.

Fails when:

- the operation is described only as a command;
- targets are ambiguous;
- success cannot be validated.

Evidence:

- spec version or ticket section;
- SDO ID.

### Risk/Profile Gate

Objective: choose the right level of rigor.

Inputs:

- base class;
- overlays;
- reversibility;
- target scope;
- data/security/availability impact.

Passes when:

- profile is selected and justified;
- approval and evidence requirements are known;
- escalation conditions are clear.

Evidence:

- risk/profile decision;
- risk assessment reference.

### Preview/Plan Gate

Objective: know what will change before changing it.

Inputs:

- execution plan;
- runbook draft;
- preview/dry-run/diff/simulation where available;
- rollback plan.

Passes when:

- planned impact is understood;
- dependencies and sequence are defined;
- no unreviewed expansion of scope exists;
- preview is reviewed or unsupported preview is compensated.

Fails when:

- preview shows unexpected destructive changes;
- plan does not match spec;
- no compensating controls exist for non-previewable work.

Evidence:

- plan/diff/dry-run output;
- preview unsupported reason and compensating controls.

### Approval Gate

Objective: authorize execution according to risk.

Inputs:

- spec;
- risk/profile decision;
- plan/runbook;
- preview;
- rollback readiness;
- execution window.

Passes when:

- correct authority approves;
- scope and conditions are explicit;
- approval is recorded.

Fails when:

- approval is generic;
- approver does not match risk;
- high-risk operation is self-approved.

Evidence:

- approval record;
- PR approval;
- ticket approval;
- protected environment approval;
- emergency authority record.

### Pre-Execution Gate

Objective: confirm execution can safely start now.

Inputs:

- prechecks;
- permissions;
- current state;
- backup/snapshot if required;
- maintenance window;
- monitoring availability;
- target list.

Passes when:

- prechecks pass;
- identity and permissions are correct;
- rollback readiness is adequate;
- targets match approved scope.

Fails when:

- backup evidence is missing;
- target scope changed;
- monitoring is unavailable for a risky change;
- window is invalid.

Evidence:

- precheck report;
- target list;
- backup/restore readiness evidence.

### Continue/Stop Gate

Objective: decide whether to continue during execution.

Evaluated when:

- a step fails;
- a metric crosses a threshold;
- validation fails;
- an unexpected target appears;
- a command differs from the approved action;
- an operator or agent needs extra permission;
- execution takes longer than expected.

Possible outcomes:

- proceed;
- hold;
- abort;
- rollback;
- escalate;
- accept-risk.

Evidence:

- decision log;
- metric/log references;
- operator notes;
- deviation record.

### Rollback Gate

Objective: switch from execution to safe-state recovery.

Inputs:

- rollback trigger;
- current state;
- rollback plan;
- point of no return status;
- backup/snapshot/version references.

Passes when:

- trigger is confirmed;
- strategy is still valid;
- required approval exists;
- rollback or recovery can be validated.

Evidence:

- rollback decision;
- rollback execution record;
- safe-state validation.

### Validation Gate

Objective: determine whether the operation achieved the spec.

Inputs:

- postchecks;
- metrics;
- logs;
- tests;
- queries;
- owner confirmation where needed.

Passes when:

- expected state matches observed state;
- critical constraints are not violated;
- residual risk is documented.

Evidence:

- validation report;
- evidence items per check.

### Evidence/Closure Gate

Objective: close only when evidence is sufficient.

Passes when:

- SDO ID exists;
- approved spec is referenced;
- planned actions are accounted for;
- executed actions map to planned actions;
- deviations are classified;
- validation results are recorded;
- rollback/recovery status is documented;
- evidence contains no secrets;
- retention and access rules are set.

Evidence:

- evidence package;
- review/closure record.

### Automation Eligibility Gate

Objective: decide whether a repeated operation can become standard or automated.

Inputs:

- execution history;
- failure rate;
- duration;
- deviations;
- evidence quality;
- rollback success;
- risk trend.

Outcomes:

- keep manual;
- standardize;
- automate with approval;
- automate with bounded autonomy;
- reject automation.

## Approval policy

| Risk | Minimum approval |
|---|---|
| R0 trivial/read-only non-sensitive | Auto-approved or none. |
| R1 low/repeatable | Auto or peer async. |
| R2 medium | Peer or service owner. |
| R3 high/production | Owner plus separate technical approver. |
| R4 critical/data/security/destructive | Multi-party approval; owner plus security/data/change authority as relevant. |
| Emergency | Incident commander or emergency authority; post-review mandatory. |
| Pipeline-backed | PR review, policy checks, environment approval where applicable. |

## Gates by profile

| Gate | Lightweight | Normal | Critical | Emergency | Pipeline-Backed |
|---|---|---|---|---|---|
| Spec Gate | Minimal | Complete | Complete | Minimal urgent | Complete/versioned |
| Risk/Profile Gate | Simple | Formal | Formal | Fast snapshot | Policy/formal |
| Preview/Plan Gate | Checklist | Required if available | Required | If time allows | Required |
| Approval Gate | Auto/peer | Peer/owner | Formal | Emergency authority | PR/env/policy |
| Pre-Execution Gate | Basic | Required | Strict | Critical minimum | Automated |
| Continue/Stop Gate | Basic | Defined | Strict | Strict | Metric/policy-based |
| Rollback Gate | Declared | Defined | Tested/validated | Recovery-focused | Rollback/roll-forward job |
| Validation Gate | Basic | Formal | Formal | Immediate | Automated checks |
| Evidence/Closure Gate | Minimal | Standard | Complete | Mandatory review | Artifacts/refs |
| Automation Eligibility | Optional | Recommended if repeated | Cautious | After incident | Recommended |

## Operations without preview

If preview, dry-run, diff, or simulation is unsupported, do not pretend it exists.

Use compensating controls:

- smaller batch;
- backup/snapshot/export;
- staging/lab trial;
- peer review;
- manual checkpoint;
- stricter stop conditions;
- explicit risk acceptance;
- alternative recovery plan.

## Anti-patterns

- Approval theater: approval without reviewing evidence.
- CAB universal: committee for everything, regardless of risk.
- Checklist without decision authority.
- Dry-run ignored when available.
- Rollback gate without rollback readiness.
- Closing with “looks good” and no evidence.
- Emergency used to bypass accountability.
- AI tool call outside approved plan.
