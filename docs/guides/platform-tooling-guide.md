# Implement SDO With Your Existing Tools

Use this guide when a platform, operations, or enablement team needs to make SDO usable across teams without mandating a specific product. SDO can run on Markdown, YAML, tickets, pull requests, pipelines, ITSM records, object storage, GitOps/IaC workflows, manual process, or any combination that preserves traceability, approvals, evidence, validation, and review.

## Quick path

1. Choose where each SDO artifact will live.
2. Define an `SDO ID` format and require it in every related record.
3. Map SDO gates to controls your teams already use.
4. Store evidence as indexed references, not raw dumps.
5. Start lightweight, then add controls by risk and profile.

## Reference layout, not a requirement

This folder layout is a practical reference for file-based or Git-backed teams. Use it only when it helps; equivalent tickets, ITSM records, PRs, pipeline artifacts, or object storage paths are valid.

```text
sdo/
  operations/
    SDO-20260523-001/
      operational-spec.md
      runbook.md
      execution-record.md
      validation-report.md
      rollback-plan.md
      evidence-package.yaml
      review-record.md
  templates/
  references/
```

For small teams, a single ticket with sections for spec, runbook, execution, validation, and evidence references may be enough. For regulated or Critical work, separate versioned artifacts and stronger storage controls are usually worth it.

## SDO IDs and naming

The `SDO ID` is the unit of traceability. It should appear in specs, tickets, PRs, commits, pipeline runs, change records, evidence indexes, approvals, rollback records, and review notes.

Recommended format:

```text
SDO-YYYYMMDD-NNN
```

Examples:

```text
SDO-20260523-001
SDO-20260523-001-PA-001
SDO-20260523-001-EVI-003
```

Practical conventions:

- keep IDs stable after assignment;
- use UTC dates if teams span time zones;
- add suffixes for planned actions, executed actions, evidence items, approvals, and rollback steps;
- avoid putting environment, team, or system names in the ID unless your organization needs routing from the ID itself;
- use titles for human meaning and IDs for traceability.

## Artifact storage and versioning

Store SDO artifacts where the owning team already works, but make the artifact graph explicit.

| Artifact need | Existing tool mapping |
|---|---|
| Operational Spec | Markdown file, ticket section, change request, PR description, ITSM record. |
| Runbook | Markdown, runbook platform, pipeline job definition, checklist, automation document. |
| Approval Record | PR review, ticket approval, ITSM approval, protected environment approval, signed change record. |
| Execution Record | Pipeline run, CLI/API output summary, operator log, change task, runbook platform job. |
| Validation Report | Test output, monitoring check, smoke test, query result, owner confirmation. |
| Evidence Package | YAML/JSON index, ticket attachment index, object storage manifest, audit package. |
| Review Record | Ticket closure, incident review, post-change review, PR comment, Markdown note. |

Versioning rules:

- version the approved spec, runbook, rollback plan, and evidence package manifest;
- preserve the approved version separately from later corrections or post-review notes;
- record hashes for material files or generated artifacts when integrity matters;
- reference immutable IDs from source systems when available, such as commit SHA, pipeline run ID, request ID, event ID, object version, or ticket history ID;
- never overwrite evidence to fix history; create a new version and explain the correction.

## Evidence handling

Evidence should prove the operation, not become a second logging platform.

Store:

- source system and source reference;
- UTC time window;
- actor or service identity;
- action/result summary;
- validation result;
- hash, object version, or immutable event ID when available;
- sensitivity level, retention owner, and access rule.

Redact or avoid:

- passwords, tokens, cookies, private keys, session IDs, connection strings, and `.env` values;
- unnecessary PII, regulated data, or full logs;
- raw AI prompts or transcripts that contain secrets;
- screenshots when audit logs or structured exports are available.

Retention and integrity should match risk. Lightweight work may only need a ticket and source references. Critical or audit-relevant work should use restricted storage, append-only or object-versioned evidence, checksums, access logs, and a hashed or signed manifest.

For the detailed model, see [`../reference/evidence-and-auditability.md`](../reference/evidence-and-auditability.md) and [`../templates/evidence-package.yaml`](../templates/evidence-package.yaml).

## Map gates to existing controls

SDO gates do not require a new platform. Map each gate to the control your teams already trust.

| SDO gate | Possible implementation |
|---|---|
| Spec Gate | Required ticket fields, spec template, PR checklist, change request completeness rule. |
| Risk/Profile Gate | Risk fields, labels, policy engine, triage checklist, ITSM classification. |
| Preview/Plan Gate | Dry-run, diff, IaC plan, SQL explain, impact analysis, peer-reviewed runbook. |
| Approval Gate | PR review, ticket approval, environment protection, CAB approval, emergency authority. |
| Pre-Execution Gate | Prechecks, backup confirmation, target list, maintenance window, permission check. |
| Continue/Stop Gate | Manual checkpoint, pipeline pause, alert threshold, incident commander decision. |
| Rollback Gate | Rollback job, restore procedure, failover runbook, compensating action approval. |
| Validation Gate | Automated tests, monitoring checks, smoke tests, queries, owner sign-off. |
| Evidence/Closure Gate | Evidence manifest review, ticket closure checklist, audit package approval. |

Use stronger gates for `SDO-Critical` and emergency post-review, but do not add ceremonies that do not improve safety, validation, or auditability. See [`../reference/gates-and-controls.md`](../reference/gates-and-controls.md) and [`../reference/profiles.md`](../reference/profiles.md).

## Execution adapters

An execution adapter is the bridge between approved SDO actions and the tool or person that performs them.

| Adapter | How to make it SDO-compatible |
|---|---|
| Human runbook | Use exact steps, expected result, stop condition, operator, timestamp, before/after evidence. |
| CLI/API | Record command or request summary, parameters, identity, exit/status code, output evidence, rollback mapping. |
| Pipeline | Link commit/PR, plan/diff, approval, run ID, artifacts, validation job, rollback or roll-forward path. |
| GitOps/IaC | Link desired-state change, plan/diff, policy checks, reconciliation result, drift/health validation. |
| GUI/manual | Record system path, exact action, before/after state, audit event if available, limitation note. |
| AI-assisted | Bind tool calls to approved actions, record human oversight, prompt summary/hash, tool trace, and final accepted action. |

For executor contracts and invariants, see [`../reference/execution-model.md`](../reference/execution-model.md). For AI-specific controls, see [`../reference/ai-governance.md`](../reference/ai-governance.md).

## Rollout path

Start with the smallest useful implementation:

1. Require an `SDO ID`, intent, owner, profile, validation, and evidence reference for every operation.
2. Add templates for specs, runbooks, validation, rollback, and evidence only where the profile needs them.
3. Map approvals to existing PR, ticket, ITSM, or pipeline gates.
4. Teach teams to link source evidence instead of dumping logs.
5. Add automation for repeated operations after the manual path is understood.
6. Add central policy only for shared risks: production, privileged access, sensitive data, broad blast radius, compliance, or hard rollback.

Avoid a central platform bottleneck. Platform teams should publish templates, conventions, guardrails, and reusable adapters, while service teams keep ownership of their operations and evidence quality.

## Anti-patterns

- Tool lock-in: making one vendor or repository layout part of the methodology.
- Evidence dumping: attaching raw logs, screenshots, or transcripts without an index, retention rule, or redaction.
- Approval theater: collecting approvals that do not identify scope, risk, authority, or execution conditions.
- Spec theater: writing long specs that do not improve execution, validation, rollback, or auditability.
- AI bypassing gates: allowing an AI assistant or agent to expand scope, execute unapproved actions, hide uncertainty, or skip human approval where required.
- Central bottleneck: forcing every team through a platform queue when proportional local controls would be safer and faster.

## Templates and examples

Use these as starting points, not mandatory bureaucracy:

- [`../templates/operational-spec.md`](../templates/operational-spec.md)
- [`../templates/runbook.md`](../templates/runbook.md)
- [`../templates/execution-record.md`](../templates/execution-record.md)
- [`../templates/validation-report.md`](../templates/validation-report.md)
- [`../templates/evidence-package.yaml`](../templates/evidence-package.yaml)
- [`../templates/rollback-plan.md`](../templates/rollback-plan.md)
- [`../templates/review-record.md`](../templates/review-record.md)
- [`../examples/`](../examples/)
- [`../integrations/`](../integrations/)
