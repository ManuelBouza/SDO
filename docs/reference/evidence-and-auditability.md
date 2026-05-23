# Evidence and Auditability

Evidence in SDO does not mean saving everything. It means building a minimal, verifiable, protected, and traceable package proving the operation lifecycle.

## Central rule

An SDO operation is auditable when every executed state-changing action can be traced to an approved Planned Action, supported by sufficient technical evidence, validated against explicit success criteria, protected against tampering, and closed without exposing secrets or unnecessary data.

## Sufficient evidence answers

- What was intended?
- Who approved it?
- What was planned?
- What was executed?
- What changed?
- What was validated?
- What deviations occurred?
- What rollback or recovery existed or ran?
- Where is the original evidence?
- How can integrity be checked?

## Evidence package rule

The Evidence Package is an index, not a parallel SIEM.

Store references, IDs, hashes, UTC time windows, actor identities, sensitivity, retention, and access rules. Do not store secrets or unnecessary raw logs.

Template: [`../templates/evidence-package.yaml`](../templates/evidence-package.yaml)

## Types of evidence

| Type | Purpose | Examples |
|---|---|---|
| Planning evidence | Proves what was intended and approved. | spec hash, plan, dry-run, diff, approval. |
| Technical execution evidence | Proves what ran and what changed. | command output, exit code, event ID, request ID, pipeline run. |
| Validation evidence | Proves the outcome satisfies the spec. | health check, metric, query, smoke test, owner confirmation. |
| Rollback / recovery evidence | Proves recovery readiness or execution. | backup ID, restore test, rollback log, failover event. |
| Approval evidence | Proves authority and conditions. | ticket approval, PR review, environment approval, emergency authority. |
| Business evidence | Proves operational or business outcome. | incident resolved, customer impact cleared, risk mitigated. |
| AI-assisted evidence | Proves AI contribution and oversight. | prompt summary/hash, tool call trace, human approval, output hash. |

## Evidence levels by profile

| Profile | Required evidence |
|---|---|
| SDO-Lightweight | SDO ID, intent, actor, action summary, result, minimal validation, evidence for state change. |
| SDO-Normal | Lightweight evidence plus approval, risk/profile decision, prechecks, postchecks, execution record, deviations, rollback declaration. |
| SDO-Critical | Normal evidence plus independent/strong validation, complete evidence package, backup/rollback readiness, decision log, stronger retention, chain of custody. |
| SDO-Emergency | Before: intent, authority, action, critical evidence. After: timeline, decisions, validation, deviations, recovery status, post-review. |
| SDO-Pipeline-Backed | Commit/PR, plan/diff, pipeline run, environment approval, artifact IDs/hashes, deployment logs, validation job, rollback/roll-forward reference. |

## Evidence item structure

Each evidence item should include:

```yaml
evidence_item:
  id: "EVI-001"
  sdo_id: "SDO-YYYYMMDD-001"
  type: "technical_execution | approval | validation | rollback | business | ai_assisted"
  sensitivity: "public | internal | confidential | restricted | regulated | secret"
  related_planned_action: "PA-001"
  related_executed_action: "EA-001"
  source_ref: ""
  source_time_window_utc:
    from: ""
    to: ""
  summary: ""
  artifact_ref: ""
  sha256: ""
  collected_by: ""
  collected_at_utc: ""
  redaction_applied: true
```

## What to store

Prefer storing:

- source system;
- event IDs;
- request IDs;
- trace IDs;
- pipeline run IDs;
- commit SHA;
- artifact IDs;
- UTC time windows;
- hashes;
- short redacted excerpts;
- query used to recover source evidence;
- retention and owner of the source system.

Avoid storing:

- full logs by default;
- full database dumps;
- secrets;
- tokens;
- `.env` files;
- unredacted PII;
- screenshots with sensitive data;
- full AI prompts containing secrets.

## Sensitivity levels

| Level | Name | Examples | Rule |
|---|---|---|---|
| E0 | Public | Public release note. | Allowed if it does not reveal sensitive architecture. |
| E1 | Internal | Internal IDs, service names, timestamps. | Allowed in normal evidence package. |
| E2 | Confidential | Hostnames, paths, partial configs. | Access control required. |
| E3 | Restricted | IAM, firewall, audit logs, security events. | Restricted access and retention. |
| E4 | Regulated | PII, financial, health, legal data. | Avoid unless necessary; minimize and justify. |
| E5 | Secret | Passwords, tokens, private keys, cookies. | Prohibited as evidence; store only external references. |

## Redaction rules

Always redact:

- Authorization headers;
- cookies;
- passwords;
- tokens;
- API keys;
- private keys;
- connection strings;
- session IDs;
- JWTs;
- `.env` values;
- secret-bearing CLI output.

Keep useful context:

- parameter name;
- operation type;
- status code;
- timestamp;
- logical resource;
- hash;
- last four characters only when useful and safe.

Example:

```text
Authorization: [REDACTED_SECRET:tok-001]
request_id=req-abc123
response_status=200
```

## Manual / GUI evidence

Manual evidence is valid but weaker than system-generated evidence.

Preference order:

1. audit log;
2. event/request/transaction ID;
3. structured export;
4. screenshot with redaction;
5. operator note;
6. manual sign-off.

Manual/GUI evidence should include:

- before/after state;
- operator;
- timestamp;
- screen or path used;
- limitation note;
- corroborating source when available.

## AI-assisted evidence

AI output is derived evidence, not primary evidence.

Record:

- AI role;
- autonomy level;
- model/tool identifier;
- prompt summary or hash;
- redacted output;
- tool calls;
- human approvals;
- policy decision;
- final accepted action.

Do not store raw prompts or transcripts containing secrets or unnecessary sensitive data.

## Chain of custody

For Critical or audit-relevant operations:

- every evidence item has `collected_by`, `collected_at_utc`, `source_ref`, and `sha256`;
- evidence is append-only or immutable where possible;
- every change creates a new version;
- package manifest is hashed or signed;
- access and export are logged;
- missing evidence is documented as a finding.

## Evidence Closure Gate

The gate passes only if:

- SDO ID exists;
- approved spec is referenced;
- each Planned Action has a final state;
- each Executed Action maps to a Planned Action;
- deviations are classified;
- every state-changing action has evidence;
- validation checks have expected and observed results;
- rollback/recovery status is documented;
- no secrets are present;
- sensitivity and retention are defined;
- closure has actor, timestamp, and decision.

## Anti-patterns

- Saving all logs “just in case”.
- Copying secrets into evidence.
- Using screenshots as the only proof when audit logs exist.
- Approving by chat without actor, timestamp, and scope.
- Closing with “works” and no validation criteria.
- Not recording timezone or time window.
- Treating AI summaries as primary evidence.
- Trusting pipeline logs without checking retention.
- Overwriting evidence instead of versioning it.
- Duplicating SIEM, ITSM, Git, or pipeline logs instead of referencing them.
