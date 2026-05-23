# SDO Examples

Examples are non-normative. They show how SDO can be applied in concrete operational contexts.

## Available examples

| Example | Profile | Purpose |
|---|---|---|
| [`observe-linux-service/`](observe-linux-service/) | SDO-Lightweight | Read-only Linux service health observation. |
| [`ai-assisted-guarded-operation/`](ai-assisted-guarded-operation/) | SDO-Normal | A4 AI-assisted read-only production diagnostic with bounded tool execution. |
| [`a5-bounded-staging-restart/`](a5-bounded-staging-restart/) | SDO-Normal | A5 agent-executed restart of one staging service after explicit human approval. |
| [`a6-policy-bound-low-risk-remediation/`](a6-policy-bound-low-risk-remediation/) | SDO-Pipeline-Backed | A6 policy-bound autonomous recycle of one unhealthy staging worker under formal preapproved policy. |
| [`emergency-remediation/`](emergency-remediation/) | SDO-Emergency | Emergency service remediation under incident pressure. |

## How to read examples

Each example may include:

- `operational-spec.md`
- `runbook.md`
- `execution-record.md`
- `validation-report.md`
- `evidence-package.yaml`
- `rollback-plan.md` when applicable
- `review-record.md`

Examples intentionally contain placeholders such as `<fill>` where real execution evidence would be captured.
