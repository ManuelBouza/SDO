# Agentic SDO

Agentic SDO makes SDO agentic-first and manual-compatible: the default design assumes AI agents may plan, execute, validate, and package evidence, while the same artifacts remain usable by humans.

## Definition

Agentic-first means SDO artifacts are explicit enough for agents to consume without guessing scope, authority, tools, targets, validation, or recovery.

Manual-compatible means the same spec, plan, runbook, validation contract, and evidence package can still be followed, reviewed, and closed by humans without AI.

## Specs as Agent Contracts

In Agentic SDO, the Operational Spec is a contract for agents and humans.

It defines:

- intended outcome;
- target environment and scope;
- constraints and exclusions;
- autonomy level;
- allowed tools and actions;
- approval boundary;
- validation criteria;
- evidence expectations;
- rollback or recovery path;
- stop conditions.

An agent may help draft or execute the plan, but it must not infer authority, expand scope, or replace missing approval.

## Agentic Lifecycle Overlay

The agentic overlay keeps the SDO lifecycle but makes agent boundaries explicit:

```text
Intent / Spec
→ Plan / Runbook
→ Approval Gate
→ Bounded Execution
→ Independent Validation
→ Evidence / Archive
→ Review / Learning
```

| Phase | Agentic focus |
|---|---|
| Intent / Spec | Convert intent into a precise contract for agents and humans. |
| Plan / Runbook | Produce executable steps, parameters, prechecks, rollback, and stop conditions. |
| Approval Gate | Approve scope, autonomy level, tools, identity, and execution window. |
| Bounded Execution | Execute only approved actions within declared limits. |
| Independent Validation | Verify results using checks not controlled only by the executor. |
| Evidence / Archive | Preserve tool calls, outputs, approvals, validation, deviations, and hashes or references. |
| Review / Learning | Improve specs, runbooks, policies, and autonomy eligibility. |

## Minimum Agentic Invariants

Every agentic operation must declare these before execution:

- explicit target and environment;
- allowed tools and actions;
- autonomy level;
- actor identity for human, agent, and executor;
- approval boundary;
- validation contract;
- evidence contract;
- rollback or recovery path;
- stop conditions.

If any invariant is missing, the operation must pause at a gate, run manually under tighter control, or be re-scoped.

## Manual Compatibility

Agentic SDO does not require AI to be present.

The same artifacts work manually when:

- a human reads the Operational Spec as the source of approved intent;
- the runbook is executable without hidden agent-only context;
- approvals record human authority and conditions;
- validation is objective and repeatable;
- evidence references prove what happened;
- rollback or recovery can be followed by a human operator.

Agentic-first changes the precision of the artifacts, not the accountability model.

## Related References

- [`../reference/agent-roles.md`](../reference/agent-roles.md)
- [`../reference/autonomy-levels.md`](../reference/autonomy-levels.md)
- [`../reference/tool-execution-policy.md`](../reference/tool-execution-policy.md)
- [`../reference/human-approval-model.md`](../reference/human-approval-model.md)
- [`../reference/ai-governance.md`](../reference/ai-governance.md)
