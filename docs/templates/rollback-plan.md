# SDO Rollback / Recovery Plan: <SDO-ID>

## Metadata

- SDO ID:
- Operation ID:
- Owner:
- Approved by:
- Version:
- Flow profile:
- Related spec:

## Agent Execution Constraints

- Agent execution allowed: yes | no
- Autonomy level for rollback / recovery:
- Allowed rollback tools / actions:
- Authorization boundary:
- Human approval required before execution:

## Reversibility

- Level: R0 | R1 | R2 | R3 | R4 | R5 | R6 | R7
- Exact previous state possible: yes | no
- Target safe state:

## Strategy

- Type: rollback | roll_forward | restore | failover | compensating_action | mitigation | recovery | no_rollback
- Rationale:
- Preferred order:
- No-rollback accepted: yes | no | n/a
- No-rollback approver:

## Point of no return

- Exists: yes | no
- Description:
- Last safe abort step:
- Approval required before crossing:

## Prerequisites

- Required permissions:
- Required artifacts:
- Required evidence:
- Backup / snapshot reference:
- Restore test evidence:
- Previous version / state reference:
- Required approval:

## Triggers

| Trigger ID | Signal | Threshold | Window | Action | Responsible |
|---|---|---|---|---|---|

## Stop conditions

| Condition | Action | Escalation |
|---|---|---|

## Rollback actions

| ID | Maps to Planned Action | Executor | Action | Expected result | Evidence required |
|---|---|---|---|---|---|

## Validation after rollback

Success criteria:

Timeout:

Owner:

## Evidence requirements

- Readiness evidence:
- Execution evidence:
- Validation evidence:
- Residual risk record:
