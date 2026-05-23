# SDO Rollback / Recovery Plan: SDO-20260523-002

## Metadata

- SDO ID: SDO-20260523-002
- Operation ID: OP-EMERGENCY-REMEDIATION
- Owner: incident-commander
- Approved by: incident-commander
- Version: 0.1.0
- Flow profile: SDO-Emergency
- Related spec: `operational-spec.md`

## Reversibility

- Level: selected at execution based on the remediation action
- Exact previous state possible: yes for restart; no for broader mitigation
- Target safe state: checkout API healthy or impact mitigated with documented residual risk

| Executed action | Reversibility |
|---|---|
| Restart | R2 |
| Failover with temporary routing | R2 |
| Failover with persistent routing or degraded dependency | R6 |
| Mitigation without exact previous state | R6 |

## Strategy

- Type: rollback | failover | mitigation | recovery
- Rationale: restore or mitigate service health as quickly as possible.
- Preferred order: stop worsening action; restore previous stable route/version if available; fail over if supported; escalate if health does not recover.
- No-rollback accepted: no
- No-rollback approver: N/A

## Point of no return

- Exists: no for bounded restart/failover
- Description: Any data mutation, IAM change, secret rotation, or global routing change requires a new explicit point of no return.
- Last safe abort step: before PA-002
- Approval required before crossing: incident commander plus relevant owner

## Prerequisites

- Required permissions: service restart/failover permissions
- Required artifacts: current service state, previous stable state if available, monitoring dashboard
- Required evidence: approval, action record, validation result
- Backup / snapshot reference: N/A for bounded restart
- Restore test evidence: N/A
- Previous version / state reference: platform-specific previous stable route/version if applicable
- Required approval: incident commander

## Triggers

| Trigger ID | Signal | Threshold | Window | Action | Responsible |
|---|---|---|---|---|---|
| TRG-001 | health check | failing after remediation | 5 minutes | rollback/failover/escalate | incident commander |
| TRG-002 | error rate | above incident threshold | 10 minutes | escalate recovery | incident commander |
| TRG-003 | unexpected scope | any non-approved target/action | immediate | stop and reapprove | operator |

## Stop conditions

| Condition | Action | Escalation |
|---|---|---|
| action requires data mutation | stop | data owner + incident commander |
| action requires IAM/security change | stop | security owner + incident commander |
| monitoring unavailable | hold unless impact requires break-glass | incident commander |

## Rollback actions

| ID | Maps to Planned Action | Executor | Action | Expected result | Evidence required |
|---|---|---|---|---|---|
| RB-001 | PA-002 | service_orchestrator | revert to previous stable route/version if available | service health improves | action log + monitoring |
| RB-002 | PA-002 | service_orchestrator | fail over to healthy instance/region if supported | traffic served by healthy target | routing/failover event |
| RB-003 | PA-003 | monitoring | validate safe state | health and error thresholds acceptable | validation report |

## Validation after rollback

Success criteria:

- health checks pass or impact is demonstrably mitigated;
- error rate below threshold or decreasing;
- no new critical alerts from remediation.

Timeout: 10 minutes unless incident policy differs.

Owner: incident commander.

## Evidence requirements

- Readiness evidence: previous stable route/version if available.
- Execution evidence: rollback/failover action logs.
- Validation evidence: monitoring checks.
- Residual risk record: required if service remains degraded.
