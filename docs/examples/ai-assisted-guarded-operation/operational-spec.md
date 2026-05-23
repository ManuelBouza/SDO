# SDO Operational Spec: AI-assisted production diagnostic

## Metadata

- SDO ID: SDO-20260523-003
- Version: 0.1.0
- Status: approved
- Owner: platform-team
- Requested by: service-owner
- Created at: 2026-05-23T00:00:00Z
- Environment: production
- Base Class: Observe
- Flow Profile: SDO-Normal
- Risk Tier: medium
- Related ticket / request: OPS-EXAMPLE-003

## Agentic Controls

- Agentic mode: guarded-agent
- Autonomy level: A4 - AI executes read-only checks
- Actor roles: human approver, AI guarded executor, independent human verifier, evidence collector
- Target / environment: production `checkout-api` service metrics and health endpoints only
- Allowed tools / actions: approved read-only monitoring queries and HTTPS health check listed in `runbook.md`
- Approval boundary: human approval is required before the agent starts; any unlisted query, parameter, target, sensitive data access, or mutation stops execution

## 1. Intent

What should be achieved: collect a bounded production diagnostic snapshot for `checkout-api` without changing system state.

Why it is necessary: verify whether a latency alert reflects active service degradation or a transient monitoring signal.

Expected outcome: approved read-only checks complete, evidence is captured, and an independent verifier confirms the finding.

Non-goals: restart services, deploy code, change routing, query customer records, inspect secrets, or perform remediation.

## 2. Scope

Included: production `checkout-api` health endpoint and aggregate non-sensitive service metrics.

Excluded: logs containing user data, database queries, Kubernetes mutations, scaling, restarts, and configuration changes.

Affected targets: `checkout-api` production service.

## 3. Current State

Known current state: latency alert fired for `checkout-api`.

Current-state evidence: monitoring alert reference in `evidence-package.yaml`.

## 4. Desired State

Desired state: no state change; diagnostic result is documented with evidence.

Acceptance criteria:

- health endpoint check returns the current status;
- aggregate latency and error-rate metrics are captured for the approved window;
- no mutating command or sensitive data access occurs;
- independent validation confirms the execution stayed inside A4 boundaries.

## 5. Constraints

Allowed window: approved 15-minute diagnostic window.

Allowed downtime: none; operation is read-only.

Security constraints: use read-only monitoring identity; do not expose secrets or customer data to the agent.

Data constraints: aggregate metrics only; no raw request payloads, customer records, tokens, or full logs.

Dependencies: monitoring API availability, approved read-only identity, human approval record.

## 6. Risk Summary

Risk level: medium due to production observation and AI tool execution.

Base class: Observe.

Overlays: production, AI-assisted, read-only, bounded target.

Blast radius: none intended; diagnostic only.

Reversibility: R0 No-change / preview only.

Main risks:

- query scope expands beyond approved service or time window;
- tool output contains sensitive data;
- the agent infers remediation steps instead of stopping at diagnostics.

Mitigations:

- allowlisted tools and parameters;
- read-only identity;
- stop conditions in `runbook.md`;
- independent validation.

Approval required: yes, before A4 execution starts.

Approval owner / authority: service-owner or delegated human approver.

## 7. Execution Strategy

Plan summary: human approves the diagnostic boundary; AI guarded executor runs approved read-only checks; human verifier validates evidence and boundaries.

Execution method: AI tool calls with read-only monitoring and HTTPS checks.

Planned actions reference: see `runbook.md`.

Preview / dry-run / diff available: not applicable; checks are non-mutating.

## 8. Validation

Pre-checks:

- confirm approval reference;
- confirm read-only identity;
- confirm target service and time window;
- confirm allowed tool list.

Post-checks:

- compare executed tool calls to runbook allowlist;
- verify evidence references exist;
- confirm no mutation or sensitive data access occurred.

Success criteria:

- all executed checks match approved planned actions;
- diagnostic result is supported by evidence;
- verifier confidence is medium or high.

Observation period: approved 15-minute diagnostic window.

## 9. Rollback / Recovery

Rollback possible: not applicable.

Strategy: read-only diagnostic; no recovery action authorized in this SDO.

Reversibility level: R0.

Point of no return: none.

Rollback plan reference: N/A; if remediation is needed, open a separate SDO or incident action.

## 10. Evidence

Required evidence:

- approval record;
- tool execution records;
- health check output reference;
- aggregate metric query output references;
- validation report;
- review record.

Evidence location: `evidence-package.yaml`.

Retention: 30 days or operational policy.

Sensitive data handling: redact unexpected sensitive output and stop if redaction cannot make evidence safe.

## 11. Traceability

Ticket: OPS-EXAMPLE-003

PR: N/A

Pipeline: N/A

Runbook: `runbook.md`

Evidence package: `evidence-package.yaml`

Review: `review-record.md`
