# SDO Rollback / Recovery Plan: SDO-20260523-006

## Metadata

- SDO ID: SDO-20260523-006
- Operation ID: EXE-SDO-20260523-006
- Owner: platform-team
- Approved by: platform-change-authority through `POL-A6-STAGING-WORKER-RECYCLE-001`
- Version: 0.1.0
- Flow profile: SDO-Pipeline-Backed
- Related spec: `operational-spec.md`

## Agent Execution Constraints

- Agent execution allowed: yes, only for the policy-bound safe-state recovery checks and actions listed here
- Autonomy level for rollback / recovery: A6 only while `POL-A6-STAGING-WORKER-RECYCLE-001` explicitly covers the recovery action; otherwise stop and require human approval
- Allowed rollback tools / actions: recovery readiness check, worker pool status check, allow orchestrator replacement to complete, restart the same staging instance if supported and policy-covered, collect post-recovery health evidence, record escalation
- Authorization boundary: staging `payments-worker-staging` only; no production target, broad selector, data deletion, queue purge, package install, deployment, config change, secret access, privilege escalation, or repeated recycle beyond policy limit
- Human approval required before execution: no for policy-covered safe-state recovery; yes for any scope change, repeated action, privileged action, production action, or inconclusive telemetry

## Reversibility

- Level: R4
- Exact previous state possible: no
- Target safe state: `payments-worker-staging` healthy capacity at or above policy threshold, no critical staging alerts, cooldown recorded, and autonomous retries stopped

## Strategy

- Type: recovery
- Rationale: recycling a worker cannot restore exact prior process memory or in-flight state; recovery returns the staging worker pool to a safe healthy capacity.
- Preferred order: confirm failed validation signal, wait for orchestrator replacement within policy timeout, restart the same staging instance only if policy-covered and not a second recycle, escalate to human/manual recovery if healthy capacity does not recover.
- No-rollback accepted: no
- No-rollback approver: N/A

## Point of no return

- Exists: yes
- Description: once the recycle request is accepted, the prior worker process state is gone.
- Last safe abort step: before PA-006 recycle execution.
- Approval required before crossing: formal policy approval `POL-A6-STAGING-WORKER-RECYCLE-001`; no per-run human approval if eligible.

## Prerequisites

- Required permissions: least-privilege staging worker remediation identity
- Required artifacts: `runbook.md`, execution record, monitoring access, policy record, service owner contact
- Required evidence: EVI-005 recovery readiness before recycle
- Backup / snapshot reference: N/A for worker process recycle; use orchestrator replacement or last healthy worker-pool state reference
- Restore test evidence: `<safe-state-recovery-test-ref>`
- Previous version / state reference: `<last-healthy-worker-pool-state-ref>`
- Required approval: `POL-A6-STAGING-WORKER-RECYCLE-001` or explicit human approval if recovery exceeds policy

## Triggers

| Trigger ID | Signal | Threshold | Window | Action | Responsible |
|---|---|---|---|---|---|
| TRG-001 | healthy capacity low | below policy threshold after recycle | 10 minutes | start safe-state recovery and escalate | autonomous executor, then verifier |
| TRG-002 | replacement unhealthy | replacement or restarted worker not healthy | 10 minutes | stop autonomous retries and escalate | autonomous executor |
| TRG-003 | critical staging alert | critical alert for worker pool | any time in observation window | stop and escalate to service owner | verifier |
| TRG-004 | telemetry inconclusive | missing or conflicting monitoring data | any time before closure | stop and escalate; no further mutation | autonomous executor |
| TRG-005 | boundary crossed | production, sensitive, privileged, broad, destructive, repeated, or novel condition | any time | stop immediately and escalate | autonomous executor |

## Stop conditions

| Condition | Action | Escalation |
|---|---|---|
| policy missing, expired, suspended, or revoked | stop before action | platform-change-authority |
| target differs from one `payments-worker-staging` instance | stop before action | service-owner |
| production target appears | stop execution | incident commander or change authority |
| broad selector or multiple candidates appear | stop execution | platform-team |
| telemetry or independent monitoring unavailable | stop execution | platform-team |
| recovery path unavailable | stop before recycle | service-owner |
| second recycle or repeated action requested | stop execution | service-owner |
| secrets, customer data, or sensitive payloads appear | stop and redact evidence | security owner |
| requested command outside allowlist | stop execution | human approver |

## Rollback actions

| ID | Maps to Planned Action | Executor | Action | Expected result | Evidence required |
|---|---|---|---|---|---|
| REC-001 | PA-007 | autonomous executor | Confirm failed validation signal and exact staging target | failure is real and scoped to the one worker pool | EVI-007 |
| REC-002 | PA-007 | platform orchestrator | Allow policy-covered orchestrator replacement to complete within timeout | worker pool returns to healthy capacity | recovery evidence ref |
| REC-003 | PA-007 | autonomous executor | Restart same staging instance only if policy-covered and not a second recycle | worker becomes healthy without expanding scope | recovery evidence ref |
| REC-004 | PA-008 | autonomous executor | Record escalation and block further autonomous retries through cooldown or policy stop | no repeated autonomous action | EVI-008, EVI-009 |
| REC-005 | manual service owner | human operator | Manual recovery if policy-covered recovery fails or scope changes | safe staging state or incident escalation | recovery evidence ref |

## Validation after rollback

Success criteria: worker pool healthy capacity is at or above threshold, no critical staging alerts remain, cooldown blocks repeated autonomous action, audit evidence is complete, and service owner accepts residual risk if manual recovery was required.

Timeout: 10 minutes after recovery action or policy-defined timeout, whichever is stricter.

Owner: staging-monitoring-verifier

## Evidence requirements

- Readiness evidence: EVI-005
- Execution evidence: EVI-006 plus recovery action output reference if triggered
- Validation evidence: EVI-007 plus recovery validation output if triggered
- Stop/escalation evidence: EVI-008
- Audit and cooldown evidence: EVI-009
- Residual risk record: `review-record.md`
