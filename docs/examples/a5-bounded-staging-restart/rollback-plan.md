# SDO Rollback / Recovery Plan: SDO-20260523-004

## Metadata

- SDO ID: SDO-20260523-004
- Operation ID: EXE-SDO-20260523-004
- Owner: platform-team
- Approved by: service-owner
- Version: 0.1.0
- Flow profile: SDO-Normal
- Related spec: `operational-spec.md`

## Agent Execution Constraints

- Agent execution allowed: yes, only for the approved recovery checks and bounded recovery actions listed here
- Autonomy level for rollback / recovery: A5 only with the original approval boundary or renewed explicit human approval if scope changes
- Allowed rollback tools / actions: recovery readiness check, service status check, start last known-good staging instance where supported, collect post-recovery health evidence
- Authorization boundary: staging `checkout-api-staging.service` only; no production target, broad selector, data deletion, package install, config change, secret access, or privilege escalation
- Human approval required before execution: yes if recovery action differs from the approved plan or requires manual intervention

## Reversibility

- Level: R4
- Exact previous state possible: no
- Target safe state: staging service active, health endpoint OK, error rate stable, no critical staging alerts

## Strategy

- Type: recovery
- Rationale: restarting a process cannot restore the exact prior in-memory process state; recovery aims to return the staging service to a safe operating state.
- Preferred order: confirm failure signal, attempt platform-supported previous instance restoration if available, start last known-good service instance, escalate to human/manual recovery if health remains degraded.
- No-rollback accepted: no
- No-rollback approver: N/A

## Point of no return

- Exists: yes
- Description: once the restart command is accepted, the previous process state is gone.
- Last safe abort step: before PA-004 restart execution.
- Approval required before crossing: APR-SDO-20260523-004

## Prerequisites

- Required permissions: least-privilege staging service recovery identity
- Required artifacts: `runbook.md`, execution record, monitoring access, service owner contact
- Required evidence: EVI-004 recovery readiness before restart
- Backup / snapshot reference: N/A for process restart; use platform previous-instance reference if available
- Restore test evidence: `<restore-test-ref>`
- Previous version / state reference: `<last-known-good-staging-instance-ref>`
- Required approval: APR-SDO-20260523-004 or renewed approval if recovery scope changes

## Triggers

| Trigger ID | Signal | Threshold | Window | Action | Responsible |
|---|---|---|---|---|---|
| TRG-001 | service inactive | service not active after restart | 2 minutes | start recovery path | AI executor, then verifier |
| TRG-002 | health endpoint failed | non-OK health response | 2 minutes | start recovery path | AI executor, then verifier |
| TRG-003 | error rate worsened | above approved baseline threshold | 10 minutes | escalate to service owner | verifier |
| TRG-004 | critical alert | critical staging alert for target service | any time in observation window | escalate to service owner | verifier |
| TRG-005 | dependency error | unexpected dependency failure appears | any time in observation window | stop agent action and escalate | AI executor |

## Stop conditions

| Condition | Action | Escalation |
|---|---|---|
| target differs from `checkout-api-staging.service` | stop before action | service-owner |
| recovery path unavailable | stop before restart | service-owner |
| requested command outside allowlist | stop execution | human approver |
| second restart requested | stop execution | service-owner |
| production target appears | stop execution | incident commander or change authority |
| secrets appear in output | stop and redact evidence | security owner |

## Rollback actions

| ID | Maps to Planned Action | Executor | Action | Expected result | Evidence required |
|---|---|---|---|---|---|
| REC-001 | PA-005 | AI bounded executor | Confirm failed post-check signal and exact target | failure is real and scoped to staging target | EVI-006 |
| REC-002 | PA-005 | AI bounded executor | Restore previous running instance only if platform supports it and target remains exact | service active on previous known-good instance | recovery evidence ref |
| REC-003 | PA-005 | AI bounded executor | If previous instance restore is unavailable, start last known-good staging service instance | service active | recovery evidence ref |
| REC-004 | PA-005 | human service owner | Manual recovery if automated bounded recovery fails or scope changes | safe staging state or explicit incident escalation | recovery evidence ref |

## Validation after rollback

Success criteria: service active, health endpoint OK, error rate stable, no critical staging alerts, and service owner accepts residual risk.

Timeout: 10 minutes after recovery action.

Owner: staging-verifier

## Evidence requirements

- Readiness evidence: EVI-004
- Execution evidence: recovery action output reference if triggered
- Validation evidence: EVI-007 plus recovery validation output if triggered
- Residual risk record: `review-record.md`
