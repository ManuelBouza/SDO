# Start here

This guide gives you the shortest useful path into SDO.

## What you need to know

SDO works with a traceable unit of work: an operation identified by an `SDO ID`.

Every SDO operation should have, at minimum:

```text
Operational Spec
Runbook or execution steps
Execution Record
Validation Result
Evidence
```

## Minimum flow

1. Declare the operational intent.
2. Write an Operational Spec.
3. Classify the operation.
4. Choose an SDO profile.
5. Prepare the plan or runbook.
6. Review required gates.
7. Execute.
8. Capture evidence.
9. Validate the result.
10. Close with review.

## Which profile should I use?

| Situation | Profile |
|---|---|
| Low-risk, repeatable, bounded operation | SDO-Lightweight |
| Usual operational change | SDO-Normal |
| High impact, data, security, production-critical, or hard rollback | SDO-Critical |
| Active incident or imminent risk | SDO-Emergency |
| GitOps, IaC, CI/CD, or automation-backed operation | SDO-Pipeline-Backed |

## Next reading

- [`core/00-sdo-core.md`](core/00-sdo-core.md)
- [`core/01-principles.md`](core/01-principles.md)
- [`core/02-lifecycle.md`](core/02-lifecycle.md)
- [`reference/taxonomy.md`](reference/taxonomy.md)
- [`reference/profiles.md`](reference/profiles.md)
- [`reference/artifacts.md`](reference/artifacts.md)
- [`reference/gates-and-controls.md`](reference/gates-and-controls.md)
- [`templates/operational-spec.md`](templates/operational-spec.md)

## If you are about to run an operation

Read the practical operator guide before filling templates:

1. [`guides/operator-guide.md`](guides/operator-guide.md)

Then use the templates that match the selected profile:

1. [`templates/operational-spec.md`](templates/operational-spec.md)
2. [`templates/runbook.md`](templates/runbook.md)
3. [`templates/execution-record.md`](templates/execution-record.md)
4. [`templates/validation-report.md`](templates/validation-report.md)
5. [`templates/evidence-package.yaml`](templates/evidence-package.yaml)
6. [`templates/rollback-plan.md`](templates/rollback-plan.md)
7. [`templates/review-record.md`](templates/review-record.md)

## If you are implementing SDO for a team or platform

Read [`guides/platform-tooling-guide.md`](guides/platform-tooling-guide.md) to map SDO to existing tickets, PRs, pipelines, ITSM, GitOps/IaC, evidence storage, or manual processes without making any one tool mandatory.

If you are introducing SDO across a team or organization, read [`guides/adoption-guide.md`](guides/adoption-guide.md) to roll it out gradually without bureaucracy or central bottlenecks.

## If AI is involved

Start with [`core/04-agentic-sdo.md`](core/04-agentic-sdo.md) to understand agentic-first, manual-compatible SDO.

Then read the controls that match the operation:

- [`reference/autonomy-levels.md`](reference/autonomy-levels.md)
- [`reference/agent-roles.md`](reference/agent-roles.md)
- [`reference/tool-execution-policy.md`](reference/tool-execution-policy.md)
- [`reference/human-approval-model.md`](reference/human-approval-model.md)
- [`reference/ai-governance.md`](reference/ai-governance.md)
