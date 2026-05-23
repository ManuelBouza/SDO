# SDO Operational Spec: Observe Linux service health

## Metadata

- SDO ID: SDO-20260523-001
- Version: 0.1.0
- Status: draft
- Owner: ops-team
- Requested by: platform-team
- Created at: 2026-05-23T00:00:00Z
- Environment: staging
- Base Class: Observe
- Flow Profile: SDO-Lightweight
- Risk Tier: low
- Related ticket / request: N/A

## 1. Intent

What should be achieved: collect a read-only health snapshot for `nginx.service` on one Linux host.

Why it is necessary: verify the service is running and producing no recent critical errors.

Expected outcome: service is active, a recent log excerpt is reviewed, and evidence is recorded without changing system state.

Non-goals: restart the service, modify configuration, rotate logs, install packages, or change firewall/network settings.

## 2. Scope

Included: one staging Linux host and `nginx.service`.

Excluded: production hosts, service restarts, configuration changes, package changes, and log exports beyond the requested excerpt.

Affected targets: `staging-web-01`.

## 3. Current State

Known current state: unknown before observation.

Current-state evidence: captured during execution.

## 4. Desired State

Desired state: no state change; service health is observed and documented.

Acceptance criteria:

- `systemctl is-active nginx.service` returns `active`.
- Recent non-sensitive staging logs contain no obvious critical errors in the selected time window.
- Evidence contains command summaries and redacted output references.

## 5. Constraints

Allowed window: anytime in staging.

Allowed downtime: none; operation is read-only.

Security constraints: do not capture secrets, tokens, or full log dumps.

Data constraints: collect only non-sensitive staging logs or redacted minimal excerpts.

Dependencies: shell access with read-only service/log permissions.

## 6. Risk Summary

Risk level: low.

Base class: Observe.

Overlays: staging, single-target, no intended mutation, possible internal log metadata.

Blast radius: one staging host.

Reversibility: R0 No-change / preview only.

Main risks:

- logs may contain sensitive values;
- operator may accidentally run mutating command.

Mitigations:

- use only approved read-only commands;
- collect only non-sensitive staging logs or redact minimal excerpts before evidence storage.

Approval required: no formal approval; peer awareness is sufficient.

Approval owner / authority: platform-team.

## 7. Execution Strategy

Plan summary: run service status and recent log checks, capture minimal evidence, validate result.

Execution method: CLI.

Planned actions reference: see `runbook.md`.

Preview / dry-run / diff available: not applicable; operation is read-only.

## 8. Validation

Pre-checks:

- confirm target host is `staging-web-01`;
- confirm commands are read-only.

Post-checks:

- service active state observed;
- non-sensitive or redacted log excerpt reviewed;
- evidence package updated.

Success criteria:

- service active;
- no critical errors found in sampled logs;
- evidence closure passes.

Observation period: immediate.

## 9. Rollback / Recovery

Rollback possible: not applicable.

Strategy: no state change.

Reversibility level: R0.

Point of no return: none.

Rollback plan reference: N/A.

## 10. Evidence

Required evidence:

- executed command summaries;
- status result;
- non-sensitive or redacted log excerpt reference;
- validation result.

Evidence location: `evidence-package.yaml`.

Retention: 30 days.

Sensitive data handling: collect only non-sensitive staging logs; if logs contain sensitive values, store only redacted minimal excerpts.

## 11. Traceability

Ticket: N/A

PR: N/A

Pipeline: N/A

Runbook: `runbook.md`

Evidence package: `evidence-package.yaml`

Review: `review-record.md`
