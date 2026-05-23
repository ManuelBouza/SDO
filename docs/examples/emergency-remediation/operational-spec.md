# SDO Operational Spec: Emergency service remediation

## Metadata

- SDO ID: SDO-20260523-002
- Version: 0.1.0
- Status: draft
- Owner: incident-commander
- Requested by: monitoring-alert
- Created at: 2026-05-23T00:00:00Z
- Environment: production
- Base Class: Remediate
- Flow Profile: SDO-Emergency
- Risk Tier: high
- Related ticket / request: INC-EXAMPLE-001

## 1. Intent

What should be achieved: restore availability of `checkout-api` after health checks started failing.

Why it is necessary: active production incident affecting user checkout.

Expected outcome: service returns healthy or traffic is routed to a known healthy fallback.

Non-goals: perform root-cause analysis before mitigation, deploy unreviewed code, change database schema, or modify IAM.

## 2. Scope

Included: `checkout-api` production deployment and immediate remediation steps.

Excluded: database migrations, infrastructure destruction, IAM/security policy changes, long-term fix.

Affected targets: production `checkout-api` service.

## 3. Current State

Known current state: health checks failing; error rate above threshold.

Current-state evidence: monitoring alert and service health dashboard references.

## 4. Desired State

Desired state: checkout API health check passing and error rate below incident threshold.

Acceptance criteria:

- health endpoint returns success;
- error rate returns below defined threshold for observation window;
- no broader service degradation introduced;
- all emergency actions are recorded.

## 5. Constraints

Allowed window: immediate incident response.

Allowed downtime: active incident; minimize further impact.

Security constraints: no IAM, firewall, certificate, or secret changes without separate approval.

Data constraints: no schema or data mutation.

Dependencies: incident commander approval, production access, monitoring available.

## 6. Risk Summary

Risk level: high.

Base class: Remediate.

Overlays: production, emergency, availability impact, potential customer impact.

Blast radius: checkout service users.

Reversibility: selected at execution based on the approved remediation action.

| Selected action | Reversibility |
|---|---|
| Restart | R2 |
| Failover with temporary routing | R2 |
| Failover with persistent routing or degraded dependency | R6 |
| Mitigation without exact previous state | R6 |

Main risks:

- remediation worsens incident;
- target scope expands accidentally;
- evidence is incomplete under pressure.

Mitigations:

- incident commander approves action;
- run only bounded remediation steps;
- capture timeline and decision log;
- validate immediately.

Approval required: yes, emergency authority.

Approval owner / authority: incident commander.

## 7. Execution Strategy

Plan summary: confirm incident state, choose bounded remediation, execute one action, validate immediately, then continue/rollback/escalate.

Execution method: mixed.

Planned actions reference: see `runbook.md`.

Preview / dry-run / diff available: limited due to emergency; compensate with narrow scope and approval.

## 8. Validation

Pre-checks:

- confirm service and environment;
- confirm incident commander approval;
- confirm monitoring is available;
- confirm action does not mutate data or IAM.

Post-checks:

- health endpoint;
- error rate;
- service logs;
- customer-impact signal if available.

Success criteria:

- service health passes;
- error rate below threshold;
- no new critical alerts.

Observation period: 10 minutes minimum or incident policy.

## 9. Rollback / Recovery

Rollback possible: depends on remediation action selected at execution.

Strategy: rollback, failover, mitigation, or recovery.

Reversibility level: R2 or R6 using the decision table in Risk Summary.

Point of no return: none for restart/failover; must be declared if action changes data, secrets, IAM, or routing globally.

Rollback plan reference: `rollback-plan.md`.

## 10. Evidence

Required evidence:

- incident timeline;
- approval by incident commander;
- selected action;
- execution record;
- validation result;
- residual risk;
- post-incident review.

Evidence location: `evidence-package.yaml`.

Retention: incident policy.

Sensitive data handling: redact logs, tokens, customer data, and internal security details.

## 11. Traceability

Ticket: INC-EXAMPLE-001

PR: N/A

Pipeline: N/A

Runbook: `runbook.md`

Evidence package: `evidence-package.yaml`

Review: `review-record.md`
