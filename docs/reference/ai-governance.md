# AI Governance

AI Governance is central to Agentic SDO. AI in SDO is a governed operational actor, not an implicit production authority.

For the agentic operating model and canonical references, read:

- [`../core/04-agentic-sdo.md`](../core/04-agentic-sdo.md)
- [`agent-roles.md`](agent-roles.md)
- [`autonomy-levels.md`](autonomy-levels.md)
- [`tool-execution-policy.md`](tool-execution-policy.md)
- [`human-approval-model.md`](human-approval-model.md)

```text
AI proposes ≠ Human approves ≠ Executor performs ≠ SDO validates
```

## Formula

```text
AI role + autonomy level + approved scope + least privilege + evidence + human accountability
```

AI may assist SDO, but SDO must remain valid without AI.

## Core rule

AI autonomous execution is only acceptable for standard, reversible, low-risk, bounded, monitored, and preapproved operations.

AI must not become the default authority for production operations.

## AI roles

| Role | Description | Recommendation |
|---|---|---|
| researcher | Finds documentation, constraints, and risks. | Broadly allowed. |
| spec_drafter | Drafts Operational Specs. | Allowed with review. |
| risk_classifier | Suggests base class, overlays, profile, and reversibility. | Advisory only. |
| runbook_drafter | Drafts steps, checks, and rollback. | Allowed with review. |
| reviewer | Reviews specs, plans, runbooks, and evidence. | Allowed; does not replace approval. |
| simulator | Performs logical simulation or dry-run interpretation. | Allowed if non-mutating. |
| execution_assistant | Guides a human during execution. | Allowed with boundaries. |
| guarded_executor | Executes approved tool calls. | Restricted and policy-bound. |
| autonomous_executor | Executes without per-action approval. | Only for standard low-risk operations. |
| evidence_summarizer | Summarizes logs, outputs, and evidence. | Allowed with redaction. |
| incident_copilot | Assists triage, timeline, hypotheses, communication. | Allowed; commander remains human. |

## Autonomy levels

[`autonomy-levels.md`](autonomy-levels.md) is the canonical Agentic SDO autonomy scale. `A6` means policy-bound autonomous operations.

When AI must not be used, state `AI prohibited for this operation/context` or `AI use forbidden by policy` instead of assigning an autonomy level.

| Level | Meaning | Allows | Does not allow |
|---|---|---|---|
| A0 | No AI / manual | Humans perform all work. | Any AI use. |
| A1 | AI drafts only | Drafting and summarization. | AI decisions, approvals, commands, or live queries. |
| A2 | AI plans | Recommendations, classification, plans, checks, and rollback ideas. | Running commands or calling APIs. |
| A3 | AI proposes commands | Exact commands, SQL, API calls, or GUI steps for human approval and execution. | AI executing proposed commands. |
| A4 | AI executes read-only checks | Bounded non-mutating diagnostics and validation checks. | Mutating state, sensitive data access without approval, or scope expansion. |
| A5 | AI executes bounded changes with approval | Approved state-changing actions inside a narrow contract. | Unapproved production changes, privilege elevation, or destructive action outside policy. |
| A6 | Policy-bound autonomous operations | Standard operations without per-run human approval under preapproved policy. | Critical, destructive, sensitive, privileged, or novel operations without human approval. |

## Profile to autonomy mapping

| SDO profile | Maximum recommended autonomy | Notes |
|---|---|---|
| SDO-Lightweight | A4 | A5 only for standard, monitored, reversible, preapproved changes with explicit approval. |
| SDO-Normal | A4/A5 | A5 requires bounded approval, rollback or recovery, validation, and evidence. |
| SDO-Critical | A2/A3 | A4 only for narrowly approved read-only checks with strong evidence. |
| SDO-Emergency | A2/A3 | AI is incident copilot; emergency authority remains human. |
| SDO-Pipeline-Backed | A5/A6 | A6 only under formal policy, monitoring, automatic stop, audit, and periodic review. |

## Risk to oversight mapping

| Condition | Minimum oversight |
|---|---|
| Low risk, R0–R2, no sensitive data, low privilege | Human-on-the-loop or bounded autonomy. |
| Medium risk or R3–R4 | Human-in-the-loop; approval by plan, batch, or checkpoint. |
| R5–R7 | Human approval plus explicit recovery/no-rollback handling. |
| Sensitive data | Human review, redaction, least privilege. |
| Admin/root/cloud-owner privilege | Human approval required. |
| Production-critical target | Approval checkpoint required. |
| Destructive action | Human approval required; often two-person rule. |
| Read-only sensitive operation | Allowed only with purpose, redaction, logging, and least privilege. |
| Tool call outside plan | Block and return to Approval Gate. |

## Integration with SDO gates

| Gate | AI control |
|---|---|
| Spec Gate | Declare AI use, role, and limits. |
| Risk/Profile Gate | Determine maximum autonomy. |
| Preview/Plan Gate | Verify AI-proposed actions map to Planned Actions. |
| Approval Gate | Approve AI scope, tools, identity, and autonomy. |
| Pre-Execution Gate | Verify tool policy, secrets handling, permissions, and environment. |
| Continue/Stop Gate | AI may recommend; high-risk decisions remain human. |
| Rollback Gate | AI may suggest; execution follows autonomy and risk rules. |
| Validation Gate | AI may summarize; primary checks must exist. |
| Evidence/Closure Gate | Record AI decision trace and tool calls. |
| Automation Eligibility Gate | Decide if A3 can become A4/A5. |

## Actions requiring human approval

AI must not execute without prior human approval when an action:

- modifies production;
- affects availability;
- touches personal data, secrets, credentials, or keys;
- changes IAM, RBAC, permissions, or firewall rules;
- deletes, destroys, purges, rotates, or revokes resources;
- modifies network, DNS, certificates, routes, or load balancers;
- changes backups, retention, or encryption;
- executes privileged commands;
- runs SQL `UPDATE`, `DELETE`, `DROP`, `TRUNCATE`, migrations, or bulk scripts;
- performs security containment;
- has no verified rollback;
- is based on untrusted input that may contain prompt injection.

## Tool-call approval rule

```text
No AI tool call may execute unless it maps to an approved Planned Action or an approved diagnostic permission.
```

| Tool call type | Examples | Approval |
|---|---|---|
| Low-risk read-only | non-sensitive status or metrics | Policy preapproval allowed. |
| Sensitive read-only | logs with PII, IAM, audit logs | Human approval or redaction required. |
| Preview/dry-run | plan, diff, explain | Allowed if non-mutating. |
| Reversible state change | bounded restart or scale | A5 requires human approval. A6 may use formal preapproved policy for eligible standard operations. |
| Irreversible state change | delete, purge, drop | Human approval required. |
| Privileged security action | isolate host, revoke token | Human approval unless preapproved emergency playbook. |
| Unknown or unplanned | anything outside plan | Block and return to gate. |

## Least privilege for AI

```text
Human identity ≠ AI runtime identity ≠ Executor identity
```

Rules:

- AI uses a bounded technical identity, not a human's full credentials.
- Tool access is allowlisted per operation.
- Scope is bound to SDO ID, target, environment, and window.
- Credentials are temporary where possible.
- Secrets are referenced, not exposed to the model.
- Privilege elevation requires Approval Gate.
- Tool calls and policy decisions are evidenced.

## AI decision trace

Every AI-assisted operation should record, proportional to risk:

```yaml
sdo_ai_decision_trace:
  trace_id: ""
  sdo_id: ""
  ai_role: "runbook_drafter | reviewer | execution_assistant | guarded_executor"
  autonomy_level: "A2"
  model_or_tool: ""
  prompt_summary: ""
  prompt_hash: "sha256:"
  context_refs: []
  recommendation_summary: ""
  tool_calls: []
  human_reviewed_by: ""
  human_approved_by: ""
  policy_decision: "allowed | blocked | modified"
  output_hash: "sha256:"
  final_action_refs: []
```

## Prompt injection and untrusted input

Rule:

```text
Untrusted content must never become instruction.
```

Treat logs, tickets, emails, webpages, command outputs, user-provided text, and incident notes as untrusted input unless explicitly validated.

Controls:

- separate system instructions, approved spec, and external data;
- do not allow AI to modify its own policy;
- block tool calls derived only from untrusted content;
- validate generated shell/API/SQL before execution;
- sandbox when possible;
- enforce allowlisted tools and argument constraints.

## AI in incidents

In emergencies, AI is a speed copilot, not incident commander.

Allowed:

- summarize timeline;
- correlate alerts;
- suggest hypotheses;
- search runbooks;
- draft communication;
- generate candidate commands;
- collect evidence;
- recommend rollback or containment.

Restricted:

- destructive containment;
- credential rotation;
- IAM changes;
- host/network isolation at broad scope;
- production shutdown;
- data deletion;
- incident closure.

## AI-assisted evidence

Distinguish:

```text
Primary Evidence: logs, outputs, snapshots, tickets, commits.
AI-Derived Evidence: summaries, classifications, recommendations.
Human Validation: approvals, acceptance, overrides.
```

AI may summarize evidence, but it must not invent evidence or validate its own execution without external checks.

## RACI guidance

| Activity | AI | Operator | Approver | System/Pipeline |
|---|---|---|---|---|
| Draft spec | Responsible | Accountable | Consulted | — |
| Classify risk | Consulted | Responsible | Accountable | — |
| Draft runbook | Responsible | Accountable | Consulted | — |
| Approve execution | No | Consulted | Accountable | — |
| Execute action | Optional | Responsible/Accountable | Consulted | Responsible if pipeline |
| Validate result | Consulted | Responsible | Accountable | Provides evidence |
| Close operation | Consulted | Responsible | Accountable | Archives evidence |

Short rule:

```text
AI may be responsible for drafting.
AI should not be accountable for operational risk.
```

## Anti-patterns

- “AI runs prod” as a default operating model.
- “The model decided” as accountability.
- Prompt as a security boundary.
- Broad tool access.
- Permanent generic approval.
- Feeding raw logs or secrets to the model.
- AI validating its own result without external checks.
- Emergency bypass without post-review.
- No record of prompts, summaries, tool calls, or approvals.
- Copying generated commands without preview or understanding.
