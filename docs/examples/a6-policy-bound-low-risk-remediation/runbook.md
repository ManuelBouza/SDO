# SDO Runbook: A6 policy-bound low-risk worker remediation

## Metadata

- Runbook ID: RB-SDO-20260523-006
- Spec ID: SDO-20260523-006
- Version: 0.1.0
- Owner: platform-team
- Execution mode: autonomous policy-bound agent execution
- Executor type: agent
- Autonomy level: A6
- Environment: staging
- Flow profile: SDO-Pipeline-Backed
- Approved change/ticket: OPS-EXAMPLE-006

## Safety

- Approval required: no per-run approval for eligible executions; formal preapproved policy is mandatory
- Approval reference: `POL-A6-STAGING-WORKER-RECYCLE-001`
- Approval checkpoint: policy approval, active status, scope, expiry, identity, tool allowlist, action limits, monitoring, automatic stop, audit, and periodic review are checked before every run
- Backup required: no data backup required; recovery readiness and healthy capacity check required before recycle
- Backup evidence: EVI-005 recovery readiness record
- Stop conditions: policy missing, expired, suspended, or revoked; production target; broad selector; more than one candidate; missing telemetry; recovery readiness failed; sensitive data exposure; destructive or privileged request; novel symptom; action outside allowlist; recycle already attempted in cooldown; post-check failure
- Point of no return: recycle request accepted by orchestration API
- Rollback reference: `rollback-plan.md`
- Evidence location: `evidence-package.yaml`

## Parameters

| Name | Required | Sensitive | Description | Validation |
|---|---:|---:|---|---|
| policy_id | yes | no | Preapproved A6 policy | Must equal `POL-A6-STAGING-WORKER-RECYCLE-001` and be active |
| worker_pool | yes | no | Worker pool to remediate | Must equal `payments-worker-staging` |
| environment | yes | no | Target environment | Must equal `staging` |
| instance_id | yes | no | Exact worker instance selected after eligibility | Must be one immutable staging instance ID, never a wildcard or label selector |
| action_limit | yes | no | Maximum recycle actions per run | Must equal `1` |
| cooldown_minutes | yes | no | Minimum cooldown before another recycle for same pool | Must be policy-defined and active |
| observation_minutes | yes | no | Post-action observation window | Must be `10` or policy timeout, whichever is stricter |

## Pre-checks

| ID | Action | Expected result | Evidence |
|---|---|---|---|
| PRE-001 | Confirm formal A6 policy approval | policy active, unexpired, scoped to staging, and approved by authorized humans | EVI-001 |
| PRE-002 | Confirm trigger signal | known single-worker unhealthy signal matches policy threshold | EVI-002 |
| PRE-003 | Run eligibility and boundary checks | exactly one candidate; no production, sensitive, privileged, destructive, novel, or broad condition | EVI-003 |
| PRE-004 | Confirm exact target | immutable instance ID belongs to `payments-worker-staging` in staging | EVI-004 |
| PRE-005 | Confirm recovery readiness and telemetry | independent monitoring and safe-state recovery path available | EVI-005 |

## Execution Steps

| Step | Planned Action ID | Actor | Executor | Allowed tool/action | Action/Command | Approval checkpoint | Expected result | Validation | Evidence | Stop condition | Rollback hook |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | PA-001 | AI autonomous executor | `svc-sdo-staging-worker-remediator` | `policy_registry.get` exact policy | Confirm policy `POL-A6-STAGING-WORKER-RECYCLE-001` is active, approved, unexpired, and scoped to this action | policy preapproval only; no per-run human approval | policy allows this exact operation | policy fields match spec | EVI-001 | policy missing, stale, revoked, expired, too broad, or not A6-approved | stop and escalate |
| 2 | PA-002 | AI autonomous executor | `svc-sdo-staging-worker-remediator` | `monitoring.alert.get` known alert only | Read trigger signal for one unhealthy staging worker | policy preapproval | trigger matches known policy condition | no novel symptom or missing telemetry | EVI-002 | novel/unclassified symptom, missing telemetry, sensitive data, or production signal | stop and escalate |
| 3 | PA-003 | AI autonomous executor | `svc-sdo-staging-worker-remediator` | `policy_engine.evaluate` | Evaluate eligibility, action limits, cooldown, scope, and forbidden conditions | policy preapproval | eligible decision is `allow` | all policy predicates pass | EVI-003 | any predicate is `deny`, `unknown`, or `manual_review` | stop and escalate |
| 4 | PA-004 | AI autonomous executor | `svc-sdo-staging-worker-remediator` | `orchestrator.instance.get` exact ID | Confirm one immutable staging worker instance ID in `payments-worker-staging` | policy preapproval | exactly one target confirmed | no broad selector or second candidate | EVI-004 | target mismatch, wildcard, label-only selector, multiple candidates | stop and escalate |
| 5 | PA-005 | AI autonomous executor | `svc-sdo-staging-worker-remediator` | `recovery.check`, `monitoring.query` aggregate staging metrics | Confirm healthy capacity, recovery path, and independent monitoring availability | policy preapproval | recovery readiness passes | safe-state recovery is available | EVI-005 | recovery unavailable or monitoring inconclusive | stop and escalate |
| 6 | PA-006 | AI autonomous executor | `svc-sdo-staging-worker-remediator` | `orchestrator.worker.recycle` exact instance ID only | Recycle the selected staging worker once | policy preapproval | recycle accepted for one instance | action count equals 1; target exact | EVI-006 | action outside allowlist, second recycle, broad selector, privilege escalation | initiate safe-state recovery or escalate |
| 7 | PA-007 | independent verifier | `staging-monitoring-verifier` | `monitoring.query`, `orchestrator.pool.status` | Validate healthy capacity, worker health, and critical alerts from independent source | policy preapproval | pool returns healthy and no critical staging alert remains | independent validation passes | EVI-007 | health fails, capacity drops below threshold, alert remains | follow rollback plan and escalate |
| 8 | PA-008 | AI autonomous executor | `svc-sdo-staging-worker-remediator` | `audit.record`, `policy_engine.cooldown.set` | Record stop/escalation decision, cooldown, and audit trail | policy preapproval | run is closed or escalated with audit evidence | evidence refs complete | EVI-008, EVI-009 | missing evidence or cooldown failure | escalate to platform-team |

## Post-checks

| ID | Action | Expected result | Evidence |
|---|---|---|---|
| POST-001 | Confirm recycle count | exactly one recycle action executed | EVI-006 |
| POST-002 | Confirm worker pool health | `payments-worker-staging` healthy capacity at or above threshold | EVI-007 |
| POST-003 | Confirm no critical staging alerts | no critical alert remains for target pool | EVI-007 |
| POST-004 | Confirm cooldown and audit record | repeated recycle is blocked by policy cooldown and audit log is written | EVI-009 |
| POST-005 | Confirm no forbidden operation occurred | no production, sensitive, destructive, privileged, broad, repeated, or novel action occurred | EVI-003, EVI-009 |

## Rollback

### Trigger conditions

Trigger safe-state recovery if the worker pool remains below healthy capacity, replacement does not become healthy, a critical staging alert remains active, telemetry becomes inconclusive, or the recycle action affects any target outside the exact instance ID.

### Steps

Follow `rollback-plan.md`. Because worker recycle cannot restore exact prior process state, recover to the safe worker-pool state instead of claiming reversal.

### Validation after rollback

Independent verifier confirms healthy staging capacity, no critical alerts, cooldown/audit records, and service owner acceptance of residual risk if manual recovery was required.

## Deviations

How to record deviations: write deviation details to `execution-record.md`, stop autonomous execution, create EVI-008 stop/escalation evidence, and route to the platform-team or service owner.

Who can approve deviations: no autonomous deviation approval is allowed; service owner, platform change authority, or incident commander must approve any scope change.

## Evidence Collection

Required evidence:

- EVI-001 policy approval and periodic review basis;
- EVI-002 trigger signal;
- EVI-003 eligibility and boundary check;
- EVI-004 target confirmation;
- EVI-005 recovery readiness;
- EVI-006 recycle action;
- EVI-007 independent validation;
- EVI-008 stop/escalation decision;
- EVI-009 audit trail and cooldown record;
- EVI-010 periodic human review record.

Storage location: `evidence-package.yaml`.

Sensitive data handling: do not store tokens, credentials, customer data, queue payloads, raw request bodies, connection strings, or sensitive logs.

## Closeout

Success condition: one policy-eligible staging worker is recycled once, independent validation passes, cooldown is recorded, and evidence is complete.

Failure condition: any stop condition occurs, validation fails, safe-state recovery is required but unavailable, evidence is incomplete, or policy eligibility is not proven.

Review required: yes, sampled audit plus periodic human review under `POL-A6-STAGING-WORKER-RECYCLE-001`.
