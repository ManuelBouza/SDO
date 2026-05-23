# Adopt SDO Without Creating a Bottleneck

Use this guide when a team, platform group, or organization wants to introduce Spec-Driven Operations gradually. The goal is safer operations, clearer ownership, useful evidence, and proportional controls without forcing one tool, one central queue, or one heavyweight template on every team.

## Adoption principles

- Start small: prove SDO on real operations before creating organization-wide rules.
- Apply proportional controls: increase rigor when risk, privilege, impact, data sensitivity, irreversibility, automation, or uncertainty increases.
- Do not force one tool: tickets, Markdown, PRs, ITSM, pipelines, runbook tools, object storage, and manual records can all carry SDO artifacts.
- Keep ownership with operating teams: platform and governance teams provide enablement and guardrails, not a central execution bottleneck.
- Measure usefulness, not paperwork: SDO is working when operations are easier to approve, execute, validate, recover, and audit.

## Phased rollout

| Phase | Goal | Entry criteria | Exit criteria |
|---|---|---|---|
| 1. Pilot | Apply SDO to a few real operations with minimal ceremony. | One willing service team, known operations, basic templates, named service owner. | Pilot operations have `SDO ID`, intent, profile, execution record, validation result, evidence, and review notes. Team can explain what helped and what was waste. |
| 2. Lightweight standard | Establish a common minimum for normal work. | Pilot lessons are reviewed, repeated operation types are known, teams agree on ID and evidence conventions. | Teams use a shared minimum package for Lightweight and Normal work, with local ownership and no central approval queue. |
| 3. Critical and emergency controls | Add stronger controls where failure matters most. | Production-critical, privileged, data-sensitive, broad-scope, destructive, or emergency operations are identified. | Critical and Emergency operations have explicit approval, stop conditions, rollback/recovery strategy, evidence retention, and mandatory review. |
| 4. Automation and tooling | Reduce manual effort after the process is understood. | Teams have stable SDO patterns, recurring operations, and clear evidence sources. | Existing tickets, PRs, pipelines, ITSM, or runbook tools create traceable records and preserve evidence references automatically where useful. |
| 5. Governance and improvement | Keep SDO useful as scale increases. | Multiple teams use SDO, metrics exist, and pain points are visible. | Governance reviews outcomes, tunes controls, removes waste, and updates templates based on real operational learning. |

Move to the next phase only when the previous phase produces safer execution or clearer review. Do not scale confusion.

## Roles and responsibilities

| Role | Responsibilities |
|---|---|
| Operator | Drafts or follows the spec/runbook, executes within approved scope, records actual actions, captures evidence, stops or escalates when conditions change. |
| Reviewer or approver | Confirms intent, scope, risk profile, plan, validation, rollback/recovery, and approval authority before execution. |
| Service owner | Owns operational risk for the service, defines acceptable validation, approves impact and residual risk, reviews outcomes. |
| Platform or enablement team | Provides templates, conventions, examples, tool mappings, evidence patterns, reusable automation, and training. Avoids becoming the mandatory path for every operation. |
| Security and compliance | Defines controls for sensitive data, privileged access, retention, auditability, and regulatory scope. Helps teams apply controls proportionally. |
| Incident commander | For Emergency operations, authorizes urgent action, controls scope, maintains decision flow, and ensures post-execution review. |

## Training and onboarding

Start with operators and service owners, not committees.

1. Teach the minimum lifecycle: intent, spec, plan/runbook, execution, validation, evidence, review.
2. Walk through one Lightweight example and one Emergency or Critical example.
3. Practice profile selection using real team operations.
4. Convert one existing runbook or ticket into a minimal SDO operation.
5. Review evidence quality: what proves the result, what should be referenced, and what should not be stored.
6. Add role-specific depth for approvers, platform teams, security/compliance, and incident commanders.

Use examples for shape, not as mandatory ceremony: [`../examples/`](../examples/).

## Success metrics

Track whether SDO improves operations:

- Percentage of relevant operations with an `SDO ID`, profile, validation result, and evidence reference.
- Reduction in operations blocked by unclear scope, missing approval, missing rollback, or missing validation.
- Time from intent to safe execution for Lightweight and Normal work.
- Percentage of Critical and Emergency operations with complete post-review and residual risk recorded.
- Number of repeated operations converted into safer templates, checklists, or automation.
- Evidence quality: reviewers can reconstruct what was approved, what happened, and how success was validated.

## Warning signals

- Teams write longer specs but still cannot validate success.
- Every operation waits for a central group even when risk is local and bounded.
- Templates grow because of organizational anxiety, not operational need.
- Approvals do not identify scope, authority, execution conditions, or risk acceptance.
- Evidence becomes raw log dumping without indexing, retention, or redaction.
- Automation runs faster than the organization can approve, stop, validate, or recover.
- AI tools draft or execute operational changes without approved scope, human accountability, and evidence.

## Migration from existing processes

Do not replace working systems first. Add SDO structure around them.

| Existing process | Practical migration |
|---|---|
| Runbooks | Add `SDO ID`, intent, profile, prechecks, stop conditions, validation, evidence references, and rollback/recovery notes. |
| Tickets | Add sections for spec, profile, approval, execution record, validation, evidence, and review. Keep the ticket as the system of record if it works. |
| Pull requests | Put the SDO ID in the title or body, link the spec/change request, preserve reviews, plans, policy checks, and pipeline runs as evidence. |
| Pipelines | Capture run ID, input version, plan/diff, approver, validation output, artifacts, rollback or roll-forward path, and evidence manifest. |
| ITSM/change records | Map SDO gates to existing change fields and approvals. Remove duplicate fields that do not improve safety or auditability. |
| Incident processes | Use Emergency controls during response, then complete the retrospective spec, evidence package, residual risk, and post-incident review after stabilization. |

For tool mapping details, see [`platform-tooling-guide.md`](platform-tooling-guide.md).

## Anti-patterns to avoid

- Big-bang rollout: forcing every team and every operation into SDO before learning from pilots.
- Mandatory heavy templates: using Critical-level artifacts for low-risk work.
- Central approval bottleneck: making one platform, CAB, or governance team approve local low-risk operations.
- Compliance-only adoption: optimizing for audit artifacts while execution safety, validation, rollback, and learning remain weak.
- AI-first without controls: letting AI generate, approve, or execute operations without profile selection, human accountability, tool boundaries, evidence, and stop conditions.

## Reference path

- Operators running an operation: [`operator-guide.md`](operator-guide.md)
- Platform and tooling implementation: [`platform-tooling-guide.md`](platform-tooling-guide.md)
- Core model: [`../core/00-sdo-core.md`](../core/00-sdo-core.md)
- Principles: [`../core/01-principles.md`](../core/01-principles.md)
- Profiles: [`../reference/profiles.md`](../reference/profiles.md)
- Gates and controls: [`../reference/gates-and-controls.md`](../reference/gates-and-controls.md)
- Templates: [`../templates/`](../templates/)
- Examples: [`../examples/`](../examples/)
