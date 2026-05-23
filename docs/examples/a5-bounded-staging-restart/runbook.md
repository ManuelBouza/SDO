# SDO Runbook: A5 bounded staging service restart

## Metadata

- Runbook ID: RB-SDO-20260523-004
- Spec ID: SDO-20260523-004
- Version: 0.1.0
- Owner: platform-team
- Execution mode: AI tool calls after human approval
- Executor type: agent
- Autonomy level: A5
- Environment: staging
- Flow profile: SDO-Normal
- Approved change/ticket: OPS-EXAMPLE-004

## Safety

- Approval required: yes, before restart execution
- Approval reference: APR-SDO-20260523-004
- Approval checkpoint: approve service name, staging environment, execution window, executor identity, one-restart limit, tool allowlist, and recovery path before Step 1
- Backup required: no data backup required; recovery readiness required before restart
- Backup evidence: EVI-004 recovery readiness record
- Stop conditions: target mismatch; missing approval; health worsens before restart; unexpected dependency errors; unapproved command or parameter; production target; broad service pattern; rollback/recovery unavailable; restart already attempted once
- Point of no return: restart command accepted by service manager
- Rollback reference: `rollback-plan.md`
- Evidence location: `evidence-package.yaml`

## Parameters

| Name | Required | Sensitive | Description | Validation |
|---|---:|---:|---|---|
| service_name | yes | no | Service to restart | Must equal `checkout-api-staging.service` |
| environment | yes | no | Target environment | Must equal `staging` |
| restart_limit | yes | no | Maximum restart attempts | Must equal `1` |
| health_url | yes | no | Staging health endpoint | Must match approved staging endpoint; do not include secrets |
| observation_minutes | yes | no | Post-restart observation window | Must be `10` or less |

## Pre-checks

| ID | Action | Expected result | Evidence |
|---|---|---|---|
| PRE-001 | Confirm explicit human approval | `APR-SDO-20260523-004` approved before execution | EVI-001 |
| PRE-002 | Confirm target and environment | target is exactly `checkout-api-staging.service` in staging | EVI-002 |
| PRE-003 | Check current service and health | service state and health baseline captured | EVI-003 |
| PRE-004 | Confirm recovery readiness | previous instance or manual recovery owner/path available | EVI-004 |

## Execution Steps

| Step | Planned Action ID | Actor | Executor | Allowed tool/action | Action/Command | Approval checkpoint | Expected result | Validation | Evidence | Stop condition | Rollback hook |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | PA-001 | AI bounded executor | `svc-sdo-staging-restart` | `service_manager.status` exact target | Confirm service manager target and active state for `checkout-api-staging.service` | APR-SDO-20260523-004 | exact staging target confirmed | target and environment match spec | EVI-002 | wrong target, production target, wildcard, or broad selector | stop before mutation |
| 2 | PA-002 | AI bounded executor | `svc-sdo-staging-restart` | `https.get` staging health endpoint; `monitoring.query` aggregate staging metrics | Capture pre-restart health, error rate, and critical alerts | APR-SDO-20260523-004 | baseline captured | no critical dependency errors; health not already worse than threshold | EVI-003 | health already critical or dependency errors unexpected | stop and escalate |
| 3 | PA-003 | AI bounded executor | `svc-sdo-staging-restart` | `recovery.check` | Confirm recovery path and owner availability | APR-SDO-20260523-004 | recovery is available before restart | readiness evidence exists | EVI-004 | recovery unavailable | stop before mutation |
| 4 | PA-004 | AI bounded executor | `svc-sdo-staging-restart` | `service_manager.restart` exact target only | Restart `checkout-api-staging.service` once | APR-SDO-20260523-004 | restart accepted for one service | action count is 1; target exact | EVI-005 | command outside allowlist or restart already attempted | initiate recovery/escalate |
| 5 | PA-005 | AI bounded executor | `svc-sdo-staging-restart` | `service_manager.status`, `https.get`, `monitoring.query` | Check active state, health endpoint, error rate, and critical alerts | APR-SDO-20260523-004 | service active, health OK, error rate stable, no critical alerts | compare to validation contract | EVI-006 | health worsens, service inactive, critical alert | follow rollback plan |

## Post-checks

| ID | Action | Expected result | Evidence |
|---|---|---|---|
| POST-001 | Confirm service active | `checkout-api-staging.service` active | EVI-006 |
| POST-002 | Confirm health endpoint | health endpoint returns OK | EVI-006 |
| POST-003 | Confirm error rate stable | error rate not worse than pre-check baseline | EVI-006 |
| POST-004 | Confirm no critical alerts | no critical staging alerts for target service | EVI-006 |
| POST-005 | Compare executed actions to allowlist | no production, broad pattern, repeated restart, package, config, data, secret, or privilege action occurred | execution record |

## Rollback

### Trigger conditions

Trigger recovery if the service is inactive after restart, health endpoint fails, error rate worsens beyond the approved threshold, a critical staging alert fires, or a dependency error appears unexpectedly.

### Steps

Follow `rollback-plan.md`. Because a process restart cannot be exactly undone, recover to a safe staging state instead of claiming reversal.

### Validation after rollback

Verifier confirms service active, health OK, error rate stable, no critical alerts, and residual risk accepted by the service owner.

## Deviations

How to record deviations: add a row to `execution-record.md`, stop execution, and return to the Approval Gate or service owner.

Who can approve deviations: service-owner or incident commander if the staging issue becomes an incident.

## Evidence Collection

Required evidence:

- EVI-001 approval record;
- EVI-002 target confirmation;
- EVI-003 pre-health baseline;
- EVI-004 rollback/recovery availability;
- EVI-005 restart action;
- EVI-006 post-health and monitoring checks;
- EVI-007 independent validation;
- EVI-008 review record.

Storage location: `evidence-package.yaml`.

Sensitive data handling: do not store tokens, credentials, customer data, connection strings, or raw sensitive logs.

## Closeout

Success condition: exactly one approved staging restart completes and independent validation passes.

Failure condition: any stop condition occurs, health worsens, recovery is required but unavailable, or evidence is insufficient.

Review required: yes, lightweight review record.
