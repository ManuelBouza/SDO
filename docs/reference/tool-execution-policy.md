# Tool Execution Policy

Tool execution policy defines what an agent, human, pipeline, or platform may run for a specific SDO operation.

## Core Rule

No tool call may execute unless it maps to an approved Planned Action or an approved diagnostic permission.

## Policy Scope

Define these before execution:

- SDO ID;
- target environment and resource scope;
- actor identity and executor identity;
- autonomy level;
- allowed tools and actions;
- denied tools and actions;
- parameter constraints;
- approval boundary;
- dry-run or preview requirement;
- stop conditions;
- evidence and logging requirements.

## Tool Boundaries

| Tool class | Required boundary |
|---|---|
| CLI | Allow commands, subcommands, flags, target selectors, timeout, and working directory. |
| API | Allow methods, endpoints, request bodies, rate limits, and idempotency expectations. |
| SQL | Allow databases, schemas, query type, row limits, transaction behavior, and rollback plan. |
| Kubernetes | Allow cluster, namespace, resource type, verbs, selectors, dry-run, and rollout limits. |
| Cloud | Allow account, project, region, service, action, tags, and cost or quota limits. |
| IaC | Require plan/diff, policy checks, approved workspace, state lock, and apply boundary. |
| GUI | Define console, navigation path, fields, screenshots or audit logs, and operator sign-off. |
| AI tool calls | Bind prompts, context, tools, arguments, autonomy level, and evidence trace to the SDO ID. |

## Allowlist and Denylist

Allowlists should be specific enough to prevent scope drift:

- exact tool or API name;
- allowed verbs or commands;
- allowed parameters and patterns;
- allowed targets and environments;
- maximum batch size or concurrency;
- approved identity and permissions.

Deny by default when:

- the tool is not listed;
- parameters do not match the approved contract;
- environment targeting is missing or ambiguous;
- the action is destructive, privileged, or sensitive without approval;
- the tool may expose secrets to prompts, logs, or evidence.

## Secrets

Secrets must be referenced, not exposed.

Rules:

- never place secrets in prompts, specs, runbooks, screenshots, or evidence text;
- use external secret stores and short-lived credentials where possible;
- redact secret-bearing output before evidence storage;
- stop if a tool returns tokens, keys, passwords, cookies, or connection strings.

## Dry-Run and Preview

Use dry-run, diff, plan, validate, or simulation before mutating production when supported.

If preview is unavailable, record compensating controls such as smaller batch, backup, peer review, manual checkpoint, staging trial, or explicit risk acceptance.

## Stop Conditions

Stop execution when:

- target expansion differs from approval;
- permissions or identity differ from the policy;
- preview shows unexpected changes;
- validation fails or becomes inconclusive;
- secrets appear in output;
- error rate, latency, cost, or failure count crosses limits;
- the agent needs a tool, parameter, or action outside policy.

## Logging and Evidence

Record enough to reconstruct the operation without exposing secrets:

- actor and executor identity;
- command, API, SQL, GUI, pipeline, or AI tool call summary;
- normalized parameters or hash;
- start and end time in UTC;
- target environment;
- result, exit code, request ID, event ID, or run ID;
- approval and planned action reference;
- redaction status;
- evidence reference or hash.
