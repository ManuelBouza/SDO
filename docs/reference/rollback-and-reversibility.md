# Rollback and Reversibility

Rollback in SDO means safe-state recovery capability, not merely undo.

## Definition

Rollback is a planned, approved, executable, verifiable, and evidentiary capability to move a failed, degraded, or unsafe system into an accepted safe state.

That safe state may be:

- the exact previous state;
- an equivalent safe state;
- a previous stable version;
- a restored state;
- a failed-over state;
- a compensated state;
- a mitigated degraded state;
- a manually accepted no-rollback state.

Template: [`../templates/rollback-plan.md`](../templates/rollback-plan.md)

## Key concepts

| Concept | Meaning |
|---|---|
| Rollback | Return to a known previous state or version. |
| Roll-forward | Move to a corrected future state. |
| Restore | Recover from backup, snapshot, image, restore point, or PITR. |
| Failover | Move service to another healthy node, zone, region, or instance. |
| Compensating action | Neutralize an effect that cannot be directly undone. |
| Mitigation | Reduce impact without fully resolving root cause. |
| Recovery | The broader process of returning to accepted operation. |
| No-rollback | No exact rollback is technically viable. |
| Point of no return | A point where recovery strategy changes in cost, risk, or feasibility. |

## Reversibility levels

| Level | Meaning | Example |
|---|---|---|
| R0 | No-change / preview only | dry-run, diff, SELECT, plan. |
| R1 | Naturally reversible | toggle feature flag, restore config value. |
| R2 | Version reversible | Git revert, rollout undo, previous artifact. |
| R3 | Snapshot / backup reversible | VM snapshot, DB backup, volume snapshot. |
| R4 | Compensatable | cancel external transaction, revoke added access. |
| R5 | Roll-forward only | hotfix after irreversible migration. |
| R6 | Mitigation only | isolate compromised host, disable integration. |
| R7 | Irreversible / no-rollback | secure deletion, exposed secret, lost data without backup. |

General mapping:

- R0–R2 may fit Lightweight or Normal.
- R3–R5 require formal controls.
- R6–R7 require explicit risk acceptance, mitigation, recovery, or emergency handling.

## Strategy selection

| Strategy | Use when | Required evidence |
|---|---|---|
| rollback | Previous state/version is valid and reachable. | previous state reference, rollback steps, validation. |
| roll_forward | Going back is riskier than fixing forward. | hotfix plan, validation, residual risk. |
| restore | Previous state exists in backup/snapshot. | backup ID, restore test or risk acceptance. |
| failover | Alternate service path is prepared. | failover target, health proof, routing evidence. |
| compensating_action | Direct undo is not possible. | compensation steps, idempotency, validation. |
| mitigation | Need to reduce impact quickly. | containment action, residual risk, recovery plan. |
| recovery | Complex return to acceptable operation. | recovery plan, status, validation, review. |
| no_rollback | No viable rollback exists. | explicit acceptance, alternatives considered, mitigation. |

## Rollback Readiness Gate

The gate passes only if:

1. reversibility level is defined;
2. strategy is defined;
3. target safe state is defined;
4. rollback/recovery actions are documented;
5. triggers are defined;
6. stop conditions are defined;
7. point of no return is defined or marked absent;
8. prerequisites and permissions are verified;
9. backup/snapshot/previous version evidence exists if required;
10. validation after rollback is defined;
11. approval matches profile and risk.

The gate fails if:

- no rollback exists and no explicit acceptance exists;
- required backup was not verified;
- rollback depends on tribal knowledge;
- no owner exists;
- no success criteria exist;
- no stop conditions exist.

## Rollback Execution Gate

Before executing rollback or recovery:

1. confirm trigger;
2. freeze non-essential new actions;
3. classify current state;
4. verify rollback is still valid;
5. check whether point of no return was crossed;
6. select rollback, roll-forward, restore, failover, mitigation, compensation, or recovery;
7. approve execution if profile requires it;
8. execute step by step;
9. validate safe state;
10. record evidence.

## Rollback triggers

Each trigger must define signal, threshold, time window, severity, action, and responsible actor.

Examples:

| Trigger | Action |
|---|---|
| Health check fails | rollback or hold. |
| Error rate exceeds threshold | pause, rollback, or escalate. |
| Canary fails | stop promotion and revert. |
| Data count mismatch | stop, preserve evidence, recover. |
| Security exposure detected | mitigate, revoke, rotate, or isolate. |
| Approved plan deviated | abort and reapprove. |
| Point of no return reached without approval | stop. |

## Point of no return

Declare a point of no return when an operation changes recovery strategy.

Common cases:

- persistent data mutation;
- destructive deletion;
- incompatible schema change;
- secret rotation or exposure;
- DNS/routing/global traffic change;
- fleet-wide operation;
- manual action that cannot be repeated exactly.

Recommended structure:

```yaml
point_of_no_return:
  exists: true
  point: "before destructive migration step"
  consequence: "rollback changes from config revert to PITR restore"
  required_approval: "explicit_human_approval"
  last_safe_abort_step: "PA-004"
  evidence_before_crossing:
    - backup_verified
    - stakeholder_approval
    - maintenance_window_active
```

## No-rollback acceptance

No-rollback is not absence of planning. It is explicit acceptance of irreversibility plus mitigation or recovery.

Required fields:

```yaml
no_rollback_acceptance:
  accepted: true
  reason: ""
  alternatives_considered: []
  chosen_strategy: "mitigation | recovery | compensating_action | roll_forward"
  residual_risk: ""
  approver: ""
  accepted_at: ""
  required_follow_up: []
```

## Data operations

Data operations require special care.

Rules:

- separate schema migration from data migration when possible;
- prefer expand/contract migrations;
- define whether rollback is transactional, restore-based, compensating, or roll-forward;
- validate backups before destructive data work;
- record row counts, checksums, constraints, and functional validation;
- treat `DROP`, `TRUNCATE`, bulk `DELETE`, and irreversible migration as Critical unless proven otherwise.

## Multi-target rollback

Fleet rollback must be batch-aware.

Define:

- one-by-one, batch, canary, or ring strategy;
- max failed targets;
- max failed percentage;
- target-specific rollback;
- batch-level rollback;
- global rollback;
- human checkpoint between batches where needed.

## AI-assisted rollback

AI may propose rollback or recovery, but irreversible or privileged rollback actions require human approval.

Rules:

- AI cannot invent previous state;
- AI must reference evidence;
- AI-generated rollback commands must map to approved rollback actions;
- R4–R7 actions require approval-required mode;
- prompts and outputs must not contain secrets.

## Final rules

- Every state-changing operation declares a reversibility level.
- Every rollback strategy defines target safe state.
- Backup existence is not rollback readiness unless restore capability is verified or explicitly risk-accepted.
- Every point of no return is declared before execution.
- Crossing a point of no return requires explicit approval for Normal, Critical, and Emergency profiles.
- No-rollback requires explicit risk acceptance and a mitigation or recovery plan.
- Rollback actions are mapped, evidenced, and validated like normal execution actions.
- If exact rollback is impossible, choose roll-forward, restore, failover, compensation, mitigation, or recovery.
- In fleet execution, rollback is batch-aware and blast-radius bounded.
- AI-assisted rollback cannot execute irreversible or privileged actions without human approval and audit trail.

## Anti-patterns

- “We have backup” without restore test.
- Rollback documented after the change.
- Rollback that reverts code but not data.
- Manual rollback not rehearsed for a critical operation.
- Confusing mitigation with full recovery.
- Treating Terraform state rollback as infrastructure restoration.
- Running `kubectl rollout undo` in GitOps without reverting Git.
- Not declaring point of no return.
- No record of who approved crossing the irreversible point.
- Rollback of a fleet without canary or batch control.
- Closing because the rollback command exited 0 without validating safe state.
