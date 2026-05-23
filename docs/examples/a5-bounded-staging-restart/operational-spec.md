# SDO Operational Spec: A5 bounded staging service restart

## Metadata

- SDO ID: SDO-20260523-004
- Version: 0.1.0
- Status: approved
- Owner: platform-team
- Requested by: service-owner
- Created at: 2026-05-23T00:00:00Z
- Environment: staging
- Base Class: Change
- Flow Profile: SDO-Normal
- Risk Tier: medium
- Related ticket / request: OPS-EXAMPLE-004

## Agentic Controls

- Agentic mode: guarded-agent
- Autonomy level: A5 - AI executes bounded changes with approval
- Actor roles: human approver, AI bounded executor, independent verifier, evidence collector
- Actor identity ref: `guarded-agent-runtime`
- Executor identity ref: `svc-sdo-staging-restart`
- Target / environment: staging service `checkout-api-staging.service` only
- Allowed tools / actions: target confirmation, current health check, restart exactly one approved staging service, post-checks, evidence collection
- Approval boundary: explicit human approval is required before any state-changing action; the agent must not execute the restart without `APR-SDO-20260523-004`
- Validation contract ref: `validation-report.md`
- Evidence contract ref: `evidence-package.yaml`
- Rollback / recovery ref: `rollback-plan.md`
- Stop conditions: target mismatch; missing approval; command outside allowlist; health worsens; unexpected dependency errors; rollback or recovery unavailable

## 1. Intent

What should be achieved: restart one staging service after approval to clear a suspected stale process state.

Why it is necessary: `checkout-api-staging.service` is responding slowly in staging after a test deployment, and the service owner approved a bounded restart before deeper remediation.

Expected outcome: the approved staging service is restarted once, health returns OK, evidence is captured, and an independent verifier confirms the result.

Non-goals: production changes, broad service restarts, deployments, package installs, configuration changes, data deletion, secrets access, privilege escalation, or repeated restarts beyond the approved limit.

## 2. Scope

Included: one restart of `checkout-api-staging.service` in staging, pre-health check, post-health check, validation, and evidence capture.

Excluded: production targets, wildcard service patterns, other staging services, package or OS changes, application config changes, database access, secret retrieval, and any destructive command.

Affected targets: `checkout-api-staging.service` on the approved staging host or service manager target.

## 3. Current State

Known current state: staging service is active but the health endpoint shows elevated latency.

Current-state evidence: pre-health and target confirmation references in `evidence-package.yaml`.

## 4. Desired State

Desired state: `checkout-api-staging.service` is active after one approved restart; health endpoint returns OK; error rate remains stable; no critical staging alerts are active.

Acceptance criteria:

- approval record exists before restart execution;
- target equals `checkout-api-staging.service` in staging;
- exactly one restart action is executed by the approved agent identity;
- post-health endpoint returns OK within the observation window;
- independent verifier confirms service active, health OK, error rate stable, and no critical alerts.

## 5. Constraints

Allowed window: approved 20-minute staging maintenance window.

Allowed downtime: up to 2 minutes for the single staging service.

Security constraints: use the least-privilege staging restart identity; do not access secrets or elevate privileges.

Data constraints: no database mutations, data deletion, customer data inspection, or raw sensitive log capture.

Dependencies: service manager access, staging health endpoint, monitoring API, rollback/recovery owner availability.

## 6. Risk Summary

Risk level: medium due to AI executing a bounded state-changing action.

Base class: Change.

Overlays: staging, A5 guarded-agent execution, human approval gate, independent validation.

Blast radius: one staging service.

Reversibility: R4 recoverable; a restart is not an exact undo, but the service can be recovered through previous running instance restoration where supported or manual recovery.

Main risks:

- target is broader than approved;
- restart worsens health or exposes dependency failures;
- agent attempts an unapproved command or repeated restart;
- rollback/recovery path is not available when needed.

Mitigations:

- explicit target and command allowlist;
- one-restart maximum;
- pre-check of rollback/recovery availability;
- stop conditions before and after the restart;
- independent validation by a verifier different from the executor.

Approval required: yes, before A5 execution starts.

Approval owner / authority: service-owner or delegated human approver.

## 7. Execution Strategy

Plan summary: human approves the bounded A5 restart; AI executor confirms target and health; AI executor restarts exactly one staging service; verifier independently validates health and evidence.

Execution method: guarded agent tool calls through approved service-management and monitoring tools.

Planned actions reference: see `runbook.md`.

Preview / dry-run / diff available: no restart dry-run is available; compensating controls are exact target match, single-action limit, rollback readiness check, and explicit approval.

## 8. Validation

Pre-checks:

- confirm approval reference `APR-SDO-20260523-004`;
- confirm target is exactly `checkout-api-staging.service` in staging;
- confirm current service state and health endpoint;
- confirm rollback/recovery owner and path are available.

Post-checks:

- confirm service is active;
- confirm health endpoint returns OK;
- confirm staging error rate is stable against pre-check baseline;
- confirm no critical staging alerts;
- confirm no commands outside the allowlist were executed.

Success criteria:

- all executed actions match approved planned actions;
- post-health is not worse than pre-health;
- independent validation passes;
- evidence package contains approval, pre-check, restart, post-check, validation, rollback availability, and review references.

Observation period: 10 minutes after restart.

## 9. Rollback / Recovery

Rollback possible: partially; restart is not exactly reversible.

Strategy: if restart worsens service health, recover to a safe staging state by restoring the previous running instance when supported by the platform, starting the last known-good unit instance, or escalating to human/manual recovery. Do not claim a fake undo of process restart state.

Reversibility level: R4.

Point of no return: after the restart command is accepted; recovery remains available but exact prior process state is gone.

Rollback plan reference: `rollback-plan.md`.

## 10. Evidence

Required evidence:

- approval record: EVI-001;
- target confirmation: EVI-002;
- pre-health baseline: EVI-003;
- rollback/recovery readiness: EVI-004;
- restart action: EVI-005;
- post-health and monitoring checks: EVI-006;
- independent validation: EVI-007;
- review record: EVI-008.

Evidence location: `evidence-package.yaml`.

Retention: 30 days or operational policy.

Sensitive data handling: redact unexpected sensitive output and stop if evidence cannot be safely stored.

## 11. Traceability

Ticket: OPS-EXAMPLE-004

PR: N/A

Pipeline: N/A

Runbook: `runbook.md`

Evidence package: `evidence-package.yaml`

Review: `review-record.md`
