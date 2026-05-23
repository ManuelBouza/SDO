# Autonomy Levels

Autonomy levels define what AI may do in an SDO operation. Higher autonomy requires stronger controls, narrower scope, better evidence, and clearer stop conditions.

## Levels A0-A6

| Level | Meaning | Allowed behavior | Required controls | Examples | Forbidden behavior |
|---|---|---|---|---|---|
| A0 | No AI / manual | Humans perform all drafting, planning, execution, validation, and evidence work. | Normal SDO controls for the selected profile. | Manual database maintenance, regulated change with no AI exposure. | Any AI use for the operation. |
| A1 | AI drafts only | AI drafts summaries, specs, notes, or evidence descriptions. | Human review before use; no tool execution. | Draft an Operational Spec from a ticket. | AI decisions, approvals, commands, or live queries. |
| A2 | AI plans | AI proposes classifications, plans, runbooks, checks, and rollback ideas. | Human review and approval before execution; no tool execution. | Generate a candidate Kubernetes rollout runbook. | Running commands or calling APIs. |
| A3 | AI proposes commands | AI proposes exact commands, SQL, API calls, or GUI steps for human approval and execution. | Command review, target confirmation, secret redaction, approval record. | Suggest `kubectl rollout status` and rollback commands. | AI executing the proposed commands. |
| A4 | AI executes read-only checks | AI runs bounded non-mutating diagnostics and validation checks. | Allowlisted tools, read-only identity, environment targeting, logging, stop conditions. | Query service health, fetch non-sensitive metrics, run dry-run or diff. | Mutating state, reading sensitive data without approval, expanding target scope. |
| A5 | AI executes bounded changes with approval | AI executes approved state-changing actions inside a narrow contract. | Explicit human approval, allowlisted actions, least privilege, rollback path, validation, evidence. | Restart one service in staging, apply approved feature flag change. | Unapproved production changes, privilege elevation, destructive action outside policy. |
| A6 | Policy-bound autonomous operations | AI executes standard operations without per-run human approval under preapproved policy. | Formal policy, narrow scope, monitoring, automatic stop, audit trail, periodic human review. | Routine reversible remediation for known low-risk alerts. | Critical, destructive, sensitive, privileged, or novel operations without human approval. |

## Selection Rules

- Use the lowest autonomy level that safely achieves the operation.
- Increase autonomy only when the operation is repeatable, bounded, observable, reversible or recoverable, and policy-covered.
- Treat production, sensitive data, privileged access, destructive action, and weak rollback as autonomy limiters.
- Emergency does not automatically increase autonomy; it changes approval timing and requires post-review.

## Minimum Controls by Level

| Level | Minimum control |
|---|---|
| A0 | Standard SDO artifacts and evidence. |
| A1 | Human review of AI output before it becomes an artifact. |
| A2 | Human approval before any execution path uses the plan. |
| A3 | Human approval of each command or approved command batch. |
| A4 | Read-only identity, allowlisted tools, logged tool calls. |
| A5 | Approval Gate, bounded executor identity, rollback/recovery, validation, evidence. |
| A6 | Preapproved policy, monitoring, automatic stop, sampled audit, periodic recertification. |

## Escalation Triggers

Pause and return to an Approval Gate when:

- target, environment, identity, or privilege changes;
- the agent proposes an unapproved tool or parameter;
- preview, dry-run, or validation output differs from expectation;
- a stop condition is reached;
- sensitive data or secrets appear unexpectedly;
- rollback or recovery is no longer valid.
