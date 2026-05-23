# Agent Roles

Agentic SDO separates planning, execution, validation, approval, ownership, and incident command. One person or system may hold multiple roles only when the selected profile allows it.

## Role Summary

| Role | Responsibilities | Boundaries |
|---|---|---|
| Coordinator | Tracks lifecycle state, routes work, requests gates, keeps artifacts connected. | Does not approve its own scope expansion or hide deviations. |
| Planner | Converts the spec into a plan, runbook, parameters, checks, and rollback path. | Does not execute state-changing actions unless also approved as Executor. |
| Executor | Performs approved actions through CLI, API, SQL, Kubernetes, cloud, IaC, GUI, pipeline, or agent tools. | Executes only approved actions within target, identity, tool, and time boundaries. |
| Verifier | Independently checks observed state against the validation contract. | Must not rely only on the Executor's self-report for closure. |
| Evidence Collector / Auditor | Captures evidence references, hashes, approvals, outputs, deviations, and retention metadata. | Does not invent evidence or store secrets unnecessarily. |
| Human Approver | Authorizes scope, autonomy, tools, timing, risk acceptance, and emergency exceptions. | Approval must be explicit, recorded, and proportional to risk. |
| Service Owner | Owns service risk, operational acceptance, residual risk, and follow-up work. | Does not bypass SDO controls for convenience. |
| Incident Commander | Directs emergency response, approves emergency actions, and ensures post-review. | Emergency authority accelerates control; it does not remove evidence or review. |

## Separation Rules

- Approval and execution should be separated for risky, sensitive, privileged, destructive, or production-critical operations.
- Verification should be independent for Critical, Emergency, and high-autonomy operations.
- AI may perform Coordinator, Planner, Executor, Verifier, or Evidence roles only within the declared autonomy level and tool policy.
- Human accountability remains with the approver, service owner, or incident commander as applicable.

## Minimum Role Record

Record these fields when roles affect approval, execution, or evidence:

- role name;
- actor type: human, agent, pipeline, platform, or service account;
- actor identity;
- authority source;
- scope and environment;
- timestamp or execution window;
- related SDO ID.
