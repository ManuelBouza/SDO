# Human Approval Model

Human approval in SDO authorizes risk, scope, autonomy, and timing. It is not a ceremonial checkbox.

## Where Humans Approve

Human approval is required when policy, risk, or autonomy demands it:

- before state-changing production actions;
- before sensitive data access;
- before privileged, destructive, or hard-to-revert actions;
- before AI executes bounded changes at A5;
- before exceptions to tool policy, validation, rollback, or evidence requirements;
- during emergencies through incident commander or break-glass authority;
- after execution when residual risk is accepted.

A6 exception: for eligible standard operations, approval is granted through formal preapproved policy rather than per-run approval. The policy must define scope, allowed actions, identity boundaries, monitoring, stop conditions, audit evidence, and periodic review.

## Approval Must Contain

An approval must be specific enough to enforce and audit:

- SDO ID;
- approver identity and role;
- approved target and environment;
- approved actions or action group;
- autonomy level;
- tool and identity boundary;
- execution window;
- validation expectations;
- rollback or recovery expectation;
- conditions, limits, or exclusions;
- timestamp and approval source.

## Emergency Approval

Emergency approval accelerates decision-making but does not remove accountability.

Minimum emergency approval:

- incident or emergency reference;
- approving authority;
- immediate objective;
- approved action boundary;
- known risk and rollback or mitigation option;
- evidence capture requirement;
- mandatory post-execution review.

## Approval Theater Anti-Patterns

Avoid approvals that cannot control execution:

- approving “the change” without target, scope, or commands;
- approving after execution and pretending it was prior approval;
- self-approving high-risk work;
- approving AI autonomy without tool boundaries;
- using a permanent generic approval for broad production access;
- ignoring preview, validation, rollback, or evidence gaps;
- approving because the agent or pipeline “looked confident”.

## Approval Record Fields

Use these fields as the minimum practical approval record:

```yaml
approval_record:
  approval_id: "APR-001"
  sdo_id: "SDO-YYYYMMDD-001"
  approver:
    name: ""
    role: "service_owner | human_approver | incident_commander | change_authority"
    identity_ref: ""
  approved_scope:
    environment: ""
    targets: []
    actions: []
    autonomy_level: "A3"
    tool_boundary_ref: ""
  conditions: []
  execution_window_utc:
    from: ""
    to: ""
  rollback_or_recovery_expectation: ""
  validation_expectation: ""
  decision: "approved | rejected | hold | emergency-approved | accepted-risk"
  decided_at_utc: ""
  evidence_ref: ""
```
