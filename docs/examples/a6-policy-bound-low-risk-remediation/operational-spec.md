# SDO Operational Spec: A6 policy-bound low-risk worker remediation

## Metadata

- SDO ID: SDO-20260523-006
- Version: 0.1.0
- Status: approved-by-policy
- Owner: platform-team
- Requested by: monitoring-policy
- Created at: 2026-05-23T00:00:00Z
- Environment: staging
- Base Class: Remediate
- Flow Profile: SDO-Pipeline-Backed
- Risk Tier: low
- Related ticket / request: OPS-EXAMPLE-006

## Agentic Controls

- Agentic mode: autonomous-policy
- Autonomy level: A6 - policy-bound autonomous operations
- Actor roles: policy owner, AI autonomous executor, independent verifier, evidence collector, periodic human reviewer
- Actor identity ref: `autonomous-remediation-agent`
- Executor identity ref: `svc-sdo-staging-worker-remediator`
- Target / environment: one unhealthy staging worker instance for `payments-worker-staging` only
- Allowed tools / actions: policy eligibility check, target confirmation, health and monitoring checks, recycle exactly one eligible staging worker instance, post-checks, evidence collection, auto-escalation
- Approval boundary: no per-run human approval is required only while `POL-A6-STAGING-WORKER-RECYCLE-001` remains approved, in scope, unexpired, and all eligibility checks pass
- Validation contract ref: `validation-report.md`
- Evidence contract ref: `evidence-package.yaml`
- Rollback / recovery ref: `rollback-plan.md`
- Stop conditions: policy missing, expired, or revoked; target mismatch; production target; broad selector; more than one candidate; novel or unclassified symptom; missing telemetry; recovery readiness failed; sensitive data exposure; privilege escalation; action limit exceeded; post-check failure; any tool or parameter outside policy

## 1. Intent

What should be achieved: automatically recycle one unhealthy staging worker instance for `payments-worker-staging` when a known low-risk health signal matches the preapproved A6 policy.

Why it is necessary: a single staging worker can become stuck after test workload spikes, and recycling one eligible worker is a standard, recoverable remediation that should not wait for per-run approval when policy controls are satisfied.

Expected outcome: exactly one eligible staging worker is recycled, the worker pool returns to the expected healthy capacity, independent monitoring confirms recovery, and an audit trail is preserved for periodic human review.

Non-goals: production remediation, customer data access, sensitive data handling, queue purging, deployments, configuration changes, database operations, destructive actions, broad worker selectors, privilege escalation, repeated actions beyond policy limits, novel symptom handling, or recovery without telemetry.

## 2. Scope

Included: one recycle action for one unhealthy staging worker instance in the `payments-worker-staging` pool, prechecks, eligibility decision, post-checks, independent validation, evidence capture, and periodic review record.

Excluded: production targets, all customer-facing environments, workers outside `payments-worker-staging`, broad labels or wildcards, multiple workers in one run, data mutation, data deletion, queue clearing, secret retrieval, IAM/RBAC changes, network changes, package installation, deployment, and manual break-glass actions.

Affected targets: exactly one instance selected by immutable instance ID after policy eligibility confirms there is one and only one unhealthy staging worker candidate.

## 3. Current State

Known current state: monitoring reports one `payments-worker-staging` worker as unhealthy for the policy-defined duration while minimum healthy capacity remains available.

Current-state evidence: policy approval, trigger signal, eligibility decision, target confirmation, telemetry snapshot, and recovery readiness references in `evidence-package.yaml`.

## 4. Desired State

Desired state: the unhealthy staging worker has been recycled once, replacement or restarted worker becomes healthy, healthy capacity returns to the expected threshold, and no critical staging alert remains active.

Acceptance criteria:

- policy `POL-A6-STAGING-WORKER-RECYCLE-001` is approved, current, scoped to staging, and recorded as EVI-001;
- trigger signal matches the known policy condition and is recorded as EVI-002;
- eligibility check confirms exactly one staging worker target, no sensitive data, no production resource, and recovery readiness before mutation;
- exactly one recycle action is executed by the approved executor identity;
- no repeated action is attempted during the cooldown window;
- independent monitoring, not the executor alone, validates healthy capacity and absence of critical alerts;
- audit evidence is sufficient for periodic human review.

## 5. Constraints

Allowed window: continuous policy window while the policy is active; execution is blocked during declared freeze windows or policy suspension.

Allowed downtime: no expected service downtime; one staging worker may be unavailable while healthy staging capacity remains above policy threshold.

Security constraints: use least-privilege staging remediation identity; no secret access, privilege escalation, IAM/RBAC modification, or production credential use.

Data constraints: no customer data, sensitive data, raw payloads, queue contents, database records, or secret-bearing logs may be read or stored.

Dependencies: monitoring trigger, independent monitoring source, worker orchestration API, recovery readiness check, audit log storage, policy registry, and periodic review owner.

## 6. Risk Summary

Risk level: low only while the policy and eligibility criteria hold; otherwise stop and escalate.

Base class: Remediate.

Overlays: staging, A6 autonomous-policy execution, policy preapproval, automatic stop, independent validation, periodic human review.

Blast radius: one non-production worker instance.

Reversibility: R4 recoverable; recycling a worker is not an exact undo, but the worker pool can recover to a safe healthy state through replacement, restart, or manual staging recovery.

Main risks:

- policy is stale, revoked, or too broad;
- target selection expands beyond one staging worker;
- telemetry is incomplete or inconsistent;
- the symptom is novel and not covered by policy;
- recycling worsens capacity or reveals a dependency failure;
- the agent repeats actions instead of stopping.

Mitigations:

- formal preapproved policy with expiry, owner, and quarterly review;
- exact environment and instance ID matching;
- one-action maximum per run and cooldown limit;
- independent monitoring source for validation;
- automatic stop and human escalation for any boundary crossing;
- audit trail with policy approval, trigger, eligibility, action, validation, and review evidence.

Approval required: no per-run human approval for eligible executions; formal preapproval is required through `POL-A6-STAGING-WORKER-RECYCLE-001`.

Approval owner / authority: platform change authority and service owner recorded in the policy approval evidence.

## 7. Execution Strategy

Plan summary: autonomous executor checks active policy, verifies the trigger and eligibility criteria, confirms exactly one staging worker instance, recycles it once, validates recovery through independent monitoring, stores evidence, and escalates to a human if any condition falls outside policy.

Execution method: policy-bound agent tool calls through approved monitoring, orchestration, policy registry, and evidence tools.

Planned actions reference: see `runbook.md`.

Preview / dry-run / diff available: no recycle dry-run is available; compensating controls are policy eligibility, exact target match, one-action limit, cooldown, recovery readiness, independent validation, and automatic stop.

## 8. Validation

Pre-checks:

- confirm policy `POL-A6-STAGING-WORKER-RECYCLE-001` is approved, active, unexpired, and scoped to staging;
- confirm trigger signal matches a known unhealthy single-worker condition;
- confirm exactly one target instance in `payments-worker-staging`;
- confirm minimum healthy capacity remains available;
- confirm telemetry and recovery readiness are available;
- confirm no production, sensitive, privileged, destructive, or novel condition is present.

Post-checks:

- confirm recycle action executed once against the exact instance ID;
- confirm replacement or restarted worker reaches healthy state;
- confirm healthy capacity is at or above policy threshold;
- confirm no critical staging alert remains active;
- confirm cooldown is recorded and repeated execution is blocked;
- confirm no commands outside the allowlist were executed.

Success criteria:

- all executed actions map to planned actions and policy controls;
- independent monitoring validates recovery;
- no stop condition or escalation boundary is crossed;
- evidence package contains policy approval, trigger signal, eligibility check, action, validation, stop/escalation decision, audit trail, and periodic review references.

Observation period: 10 minutes after recycle action or until independent monitoring reaches the policy-defined healthy state, whichever is longer within the policy timeout.

## 9. Rollback / Recovery

Rollback possible: partially; worker recycle is not exactly reversible.

Strategy: recover to the safe state defined by policy: maintain or restore healthy staging worker capacity by allowing orchestration replacement, restarting the selected worker if supported, or escalating to human/manual recovery if the pool does not recover. Do not claim an exact undo of the recycled process state.

Reversibility level: R4.

Point of no return: after the orchestration API accepts the recycle request; exact previous process state is gone, but safe-state recovery remains available.

Rollback plan reference: `rollback-plan.md`.

## 10. Evidence

Required evidence:

- policy approval and periodic review basis: EVI-001;
- trigger signal: EVI-002;
- eligibility and boundary check: EVI-003;
- target confirmation: EVI-004;
- recovery readiness: EVI-005;
- recycle action: EVI-006;
- independent validation: EVI-007;
- stop/escalation decision: EVI-008;
- audit trail and cooldown record: EVI-009;
- periodic human review record: EVI-010.

Evidence location: `evidence-package.yaml`.

Retention: 90 days or operational audit policy, whichever is longer.

Sensitive data handling: redact unexpected sensitive output and stop if evidence cannot be stored without secrets, customer data, raw payloads, or credentials.

## 11. Traceability

Ticket: OPS-EXAMPLE-006

PR: N/A

Pipeline: N/A

Runbook: `runbook.md`

Evidence package: `evidence-package.yaml`

Review: `review-record.md`
