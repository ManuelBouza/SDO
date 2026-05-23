# SDO Execution Record: SDO-20260523-004

## Metadata

- Execution ID: EXE-SDO-20260523-004
- SDO ID: SDO-20260523-004
- Runbook ID: RB-SDO-20260523-004
- Approved plan: `runbook.md`
- Approval reference: APR-SDO-20260523-004
- Flow profile: SDO-Normal
- Environment: staging
- Started at: `<execution-start-utc>`
- Finished at: `<execution-finish-utc>`
- Executed by: guarded-agent-runtime
- Executor type: agent
- Actor identity: agent
- Effective identity: `svc-sdo-staging-restart`
- Policy / approval reference: `tool-execution-policy.md`, APR-SDO-20260523-004

## Planned vs actual

| Planned Action ID | Planned Action Hash/Ref | Tool call / action ID | Executed Action Summary | Actual parameters | Executed Action Hash/Ref | Policy / approval ref | Status | Deviation | Output / evidence ref |
|---|---|---|---|---|---|---|---|---|---|
| PA-001 | `runbook.md#execution-steps` | TOOL-001 | Confirm exact staging service target | `service=checkout-api-staging.service`, `env=staging` | `sha256:<fill>` | APR-SDO-20260523-004 | pending | none | EVI-002 |
| PA-002 | `runbook.md#execution-steps` | TOOL-002 | Capture pre-restart health baseline | `service=checkout-api-staging.service`, `window=10m` | `sha256:<fill>` | APR-SDO-20260523-004 | pending | none | EVI-003 |
| PA-003 | `runbook.md#execution-steps` | TOOL-003 | Confirm recovery readiness | `service=checkout-api-staging.service` | `sha256:<fill>` | APR-SDO-20260523-004 | pending | none | EVI-004 |
| PA-004 | `runbook.md#execution-steps` | TOOL-004 | Restart one staging service once | `service=checkout-api-staging.service`, `restart_count=1` | `sha256:<fill>` | APR-SDO-20260523-004 | pending | none | EVI-005 |
| PA-005 | `runbook.md#execution-steps` | TOOL-005 | Capture post-restart health and monitoring | `service=checkout-api-staging.service`, `window=10m` | `sha256:<fill>` | APR-SDO-20260523-004 | pending | none | EVI-006 |

## Step results

| Step | Planned Action ID | Actor identity | Tool call / action ID | Started | Finished | Result | Exit/status code | Actual parameters | Output reference | Evidence | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | PA-001 | guarded-agent-runtime | TOOL-001 | `<utc>` | `<utc>` | pending | `<status>` | exact staging target | `<output-ref>` | EVI-002 | stop if target differs |
| 2 | PA-002 | guarded-agent-runtime | TOOL-002 | `<utc>` | `<utc>` | pending | `<status>` | approved health and metrics | `<output-ref>` | EVI-003 | no secrets or raw logs |
| 3 | PA-003 | guarded-agent-runtime | TOOL-003 | `<utc>` | `<utc>` | pending | `<status>` | recovery path check | `<output-ref>` | EVI-004 | must pass before restart |
| 4 | PA-004 | guarded-agent-runtime | TOOL-004 | `<utc>` | `<utc>` | pending | `<status>` | exact service restart once | `<output-ref>` | EVI-005 | state-changing A5 action |
| 5 | PA-005 | guarded-agent-runtime | TOOL-005 | `<utc>` | `<utc>` | pending | `<status>` | approved post-checks | `<output-ref>` | EVI-006 | triggers recovery if worse |

## Deviations

| ID | Classification | Description | Impact | Approved by | Evidence |
|---|---|---|---|---|---|
| N/A | none | No deviation recorded in example state. | none | N/A | N/A |

## Rollback execution

- Triggered: no
- Reason: pending example execution; trigger if post-check health worsens
- Rollback record reference: `rollback-plan.md`
- Final safe state: pending validation

## Evidence summary

- Evidence package: `evidence-package.yaml`
- Logs / artifacts: EVI-001 through EVI-008
- Sensitive data redaction applied: pending confirmation

## Final status

- Status: pending
- Validation report: `validation-report.md`
- Closed by: pending
- Closed at: pending
