# SDO Execution Record: SDO-20260523-003

## Metadata

- Execution ID: EXE-SDO-20260523-003
- SDO ID: SDO-20260523-003
- Runbook ID: RB-SDO-20260523-003
- Approved plan: `runbook.md`
- Approval reference: APR-SDO-20260523-003
- Flow profile: SDO-Normal
- Environment: production
- Started at: `<execution-start-utc>`
- Finished at: `<execution-finish-utc>`
- Executed by: guarded-agent-runtime
- Executor type: agent
- Actor identity: agent
- Effective identity: `svc-sdo-readonly-monitoring`
- Policy / approval reference: `tool-execution-policy.md`, APR-SDO-20260523-003

## Planned vs actual

| Planned Action ID | Planned Action Hash/Ref | Tool call / action ID | Executed Action Summary | Actual parameters | Executed Action Hash/Ref | Policy / approval ref | Status | Deviation | Output / evidence ref |
|---|---|---|---|---|---|---|---|---|---|
| PA-001 | `runbook.md#execution-steps` | TOOL-001 | Health endpoint read | `service=checkout-api`, `env=production` | `sha256:<fill>` | APR-SDO-20260523-003 | pending | none | EVI-002 |
| PA-002 | `runbook.md#execution-steps` | TOOL-002 | Aggregate latency query | `service=checkout-api`, `window=15m`, `metric=p95_latency` | `sha256:<fill>` | APR-SDO-20260523-003 | pending | none | EVI-003 |
| PA-003 | `runbook.md#execution-steps` | TOOL-003 | Aggregate error-rate query | `service=checkout-api`, `window=15m`, `metric=error_rate` | `sha256:<fill>` | APR-SDO-20260523-003 | pending | none | EVI-004 |

## Step results

| Step | Planned Action ID | Actor identity | Tool call / action ID | Started | Finished | Result | Exit/status code | Actual parameters | Output reference | Evidence | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | PA-001 | guarded-agent-runtime | TOOL-001 | `<utc>` | `<utc>` | pending | `<status>` | approved health URL | `<output-ref>` | EVI-002 | read-only |
| 2 | PA-002 | guarded-agent-runtime | TOOL-002 | `<utc>` | `<utc>` | pending | `<status>` | approved metric query | `<output-ref>` | EVI-003 | aggregate only |
| 3 | PA-003 | guarded-agent-runtime | TOOL-003 | `<utc>` | `<utc>` | pending | `<status>` | approved metric query | `<output-ref>` | EVI-004 | aggregate only |

## Deviations

| ID | Classification | Description | Impact | Approved by | Evidence |
|---|---|---|---|---|---|
| N/A | none | No deviation recorded in example state. | none | N/A | N/A |

## Rollback execution

- Triggered: no
- Reason: read-only diagnostic; no state changed
- Rollback record reference: N/A
- Final safe state: unchanged production state

## Evidence summary

- Evidence package: `evidence-package.yaml`
- Logs / artifacts: EVI-001 through EVI-005
- Sensitive data redaction applied: pending confirmation

## Final status

- Status: pending
- Validation report: `validation-report.md`
- Closed by: pending
- Closed at: pending
