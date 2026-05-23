# SDO Runbook: Emergency service remediation

## Metadata

- Runbook ID: RB-SDO-20260523-002
- Spec ID: SDO-20260523-002
- Version: 0.1.0
- Owner: incident-commander
- Execution mode: mixed
- Environment: production
- Flow profile: SDO-Emergency
- Approved change/ticket: INC-EXAMPLE-001

## Safety

- Approval required: yes
- Approval reference: incident commander approval in execution record
- Backup required: no for restart/failover; required before data mutation, which is out of scope
- Backup evidence: N/A
- Stop conditions: action worsens health; target mismatch; action requires IAM/data mutation; monitoring unavailable
- Point of no return: none for bounded restart/failover; must be declared for any expanded action
- Rollback reference: `rollback-plan.md`
- Evidence location: `evidence-package.yaml`

## Parameters

| Name | Required | Sensitive | Description | Validation |
|---|---:|---:|---|---|
| service_name | yes | no | impacted service | must equal `checkout-api` |
| environment | yes | no | target environment | must equal `production` |
| incident_id | yes | no | incident reference | must equal `INC-EXAMPLE-001` |

## Pre-checks

| ID | Action | Expected result | Evidence |
|---|---|---|---|
| PRE-001 | Confirm incident commander approval | approval recorded | execution record |
| PRE-002 | Confirm target service | `checkout-api` in production | command/dashboard reference |
| PRE-003 | Confirm monitoring available | health and error-rate visible | monitoring reference |

## Execution Steps

| Step | Planned Action ID | Actor | Executor | Action/Command | Expected result | Validation | Evidence | Stop condition | Rollback hook |
|---|---|---|---|---|---|---|---|---|---|
| 1 | PA-001 | human | monitoring | confirm current incident signal | incident state confirmed | alert still active | EVI-001 | signal not matching spec | reassess incident |
| 2 | PA-002 | human | service_orchestrator | perform bounded service restart or failover according to local platform | service begins recovery | health improves | EVI-002 | target mismatch or action expands scope | stop and escalate |
| 3 | PA-003 | human | monitoring | observe health and error rate | metrics improve | thresholds pass | EVI-003 | metrics worsen | rollback/escalate |

## Post-checks

| ID | Action | Expected result | Evidence |
|---|---|---|---|
| POST-001 | Health check | passing | validation report |
| POST-002 | Error rate | below threshold | monitoring reference |
| POST-003 | Incident timeline updated | decision/action recorded | evidence package |

## Rollback

### Trigger conditions

- remediation worsens health;
- service fails to recover within observation window;
- unexpected scope expansion;
- new critical alert.

### Steps

Use `rollback-plan.md`.

### Validation after rollback

Safe-state validation must confirm either restored health or documented escalation path.

## Deviations

How to record deviations: write a decision log entry with timestamp, actor, reason, approval, action, and evidence.

Who can approve deviations: incident commander.

## Evidence Collection

Required evidence:

- approval;
- alert/monitoring snapshot;
- action record;
- validation result;
- incident timeline;
- review record.

Storage location: `evidence-package.yaml`.

Sensitive data handling: redact customer data, tokens, credentials, and security-sensitive log details.

## Closeout

Success condition: service healthy and incident commander agrees impact is mitigated.

Failure condition: service remains unhealthy or remediation worsens incident.

Review required: mandatory post-incident review.
