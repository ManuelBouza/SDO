# SDO Runbook: AI-assisted production diagnostic

## Metadata

- Runbook ID: RB-SDO-20260523-003
- Spec ID: SDO-20260523-003
- Version: 0.1.0
- Owner: platform-team
- Execution mode: AI tool calls
- Executor type: agent
- Autonomy level: A4
- Environment: production
- Flow profile: SDO-Normal
- Approved change/ticket: OPS-EXAMPLE-003

## Safety

- Approval required: yes, before execution
- Approval reference: APR-SDO-20260523-003
- Approval checkpoint: approve target, time window, read-only identity, tools, and parameters before Step 1
- Backup required: no
- Backup evidence: N/A
- Stop conditions: target mismatch; unapproved tool or parameter; mutating action requested; sensitive data appears; output is inconclusive; monitoring API returns broader data than approved
- Point of no return: none
- Rollback reference: N/A; read-only diagnostic
- Evidence location: `evidence-package.yaml`

## Parameters

| Name | Required | Sensitive | Description | Validation |
|---|---:|---:|---|---|
| service_name | yes | no | Service to diagnose | Must equal `checkout-api` |
| environment | yes | no | Target environment | Must equal `production` |
| window_minutes | yes | no | Metric lookback window | Must be `15` or less |
| health_url | yes | no | Health endpoint | Must match approved production health endpoint |

## Pre-checks

| ID | Action | Expected result | Evidence |
|---|---|---|---|
| PRE-001 | Confirm approval record | `APR-SDO-20260523-003` approved | approval ref |
| PRE-002 | Confirm executor identity | read-only monitoring identity active | identity check ref |
| PRE-003 | Confirm tool allowlist | only `monitoring.query` and `https.get` are enabled | policy check ref |

## Execution Steps

| Step | Planned Action ID | Actor | Executor | Allowed tool/action | Action/Command | Approval checkpoint | Expected result | Validation | Evidence | Stop condition | Rollback hook |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | PA-001 | AI guarded executor | read-only agent identity | `https.get` health endpoint | GET approved `/health` endpoint | APR-SDO-20260523-003 | current service health returned | target and URL match spec | EVI-002 | unexpected sensitive output or wrong target | N/A |
| 2 | PA-002 | AI guarded executor | read-only agent identity | `monitoring.query` aggregate latency | query p95 latency for `checkout-api` over 15 minutes | APR-SDO-20260523-003 | aggregate latency series returned | service and window match spec | EVI-003 | query expands service/window | N/A |
| 3 | PA-003 | AI guarded executor | read-only agent identity | `monitoring.query` aggregate errors | query error rate for `checkout-api` over 15 minutes | APR-SDO-20260523-003 | aggregate error-rate series returned | service and window match spec | EVI-004 | query expands service/window | N/A |

## Post-checks

| ID | Action | Expected result | Evidence |
|---|---|---|---|
| POST-001 | Compare actual tool calls to allowlist | all calls match planned actions | execution record |
| POST-002 | Confirm no mutation occurred | no state-changing call recorded | execution record |
| POST-003 | Confirm evidence package updated | evidence items linked | evidence package |

## Rollback

### Trigger conditions

Not applicable; operation is read-only. If remediation is needed, stop and create a separate approved action.

### Steps

Not applicable.

### Validation after rollback

Not applicable.

## Deviations

How to record deviations: add a row to `execution-record.md`, stop execution, and return to the Approval Gate.

Who can approve deviations: service-owner or incident commander if this becomes an incident.

## Evidence Collection

Required evidence:

- approval reference;
- identity and policy check;
- health check output reference;
- aggregate metric output references;
- validation report.

Storage location: `evidence-package.yaml`.

Sensitive data handling: do not store raw sensitive payloads; redact unexpected sensitive output and stop if unsafe.

## Closeout

Success condition: approved read-only checks complete and verifier confirms A4 boundary compliance.

Failure condition: any stop condition occurs or evidence is insufficient.

Review required: yes, lightweight review record.
