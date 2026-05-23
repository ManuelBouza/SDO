# Execution Model

SDO uses execution by contract, not execution by technology.

```text
Operational Spec
→ Execution Plan
→ Planned Actions
→ Executor Binding
→ Actual Execution
→ Evidence
→ Validation
→ Review / Rollback
```

The methodology does not execute “Bash”, “PowerShell”, “SQL”, “kubectl”, or “Terraform” as first-class concepts. It defines approved operational actions and binds them to executors with declared capabilities.

## Core concepts

### Planned Action

An approved operational action with target, preconditions, expected result, validation, evidence, stop condition, and rollback or recovery mapping.

### Executor Binding

The mapping between a Planned Action and a concrete executor.

### Actual Execution

The real execution performed by a human, system, pipeline, or agent.

### Execution Record

The record that proves what actually happened and whether it matched the approved plan.

## Executor types

| Executor | Typical use |
|---|---|
| shell_command | Bash, sh, zsh, Linux commands, systemd, package managers. |
| powershell_command | PowerShell, Windows services, registry, GPO, scheduled tasks. |
| sql_executor | DDL, DML, migrations, backups, restores. |
| api_executor | REST, GraphQL, gRPC, webhooks. |
| kubernetes_executor | kubectl, Helm, controllers, rollout, dry-run, diff. |
| cloud_cli_executor | AWS CLI, Azure CLI, gcloud, cloud APIs. |
| iac_executor | Terraform/OpenTofu, Pulumi, Ansible, Chef, Puppet, Salt. |
| gitops_executor | Argo CD, Flux, pull-based reconciliation. |
| pipeline_executor | GitHub Actions, GitLab CI/CD, Jenkins, Azure DevOps. |
| runbook_platform_executor | Rundeck, StackStorm, AWX, SSM Automation, Azure Automation. |
| manual_executor | Checklist, GUI, console, physical action, operator sign-off. |
| ai_assisted_executor | AI-assisted tool use under SDO AI governance. |

## Executor capability contract

Every executor should declare capabilities. If a capability is unsupported, say so explicitly and compensate with controls.

```yaml
executor_capabilities:
  supports_preview: true | false
  preview_modes: [dry_run, diff, plan, validate, simulate]
  supports_execute: true
  supports_rollback: true | false
  supports_native_idempotency: true | false
  supports_retries: true | false
  supports_timeout: true | false
  supports_concurrency_control: true | false
  supports_structured_output: true | false
  supports_artifact_capture: true | false
  supports_interactive_steps: true | false
  supports_human_approval_pause: true | false
```

## Generic execution step

```yaml
sdo_execution_step:
  step_id: "STEP-001"
  sdo_id: "SDO-YYYYMMDD-001"
  planned_action_id: "PA-001"
  title: ""
  base_class: "Control"
  flow_profile: "SDO-Normal"

  executor:
    type: "shell_command"
    name: "bash"
    version: ""
    capability_profile:
      preview: "limited"
      rollback: "custom"

  target:
    scope: "single"
    target_id: ""
    environment: "prod"

  planned_action:
    action_type: "command | script | sql | api_call | pipeline_job | manual_step | gitops_change"
    action_ref: ""
    immutable_hash: "sha256:"

  parameters:
    inputs: {}
    secrets:
      - name: ""
        source: "external_secret_store"
        exposure_policy: "never_log"

  identity_and_permissions:
    run_as: ""
    privilege_level: ""
    required_permissions: []
    approval_required: true

  controls:
    timeout_seconds: 0
    retry_enabled: false
    max_attempts: 1
    concurrency_mode: "serial"
    max_parallel: 1

  preview:
    required: true
    supported: true
    mode: "diff"
    unsupported_reason: ""

  validation:
    success_criteria: []
    validation_actions: []

  rollback:
    rollback_type: "rollback | roll_forward | restore | failover | compensating_action | mitigation | recovery | no_rollback"
    rollback_ref: ""

  evidence_requirements:
    capture_stdout: true
    capture_stderr: true
    capture_exit_code: true
    capture_artifacts: []
```

## Planned vs actual execution

Approved actions and executed actions must be recorded separately.

```yaml
approved_action:
  planned_action_id: "PA-001"
  normalized_representation: ""
  immutable_hash: "sha256:"
  approval_id: "APR-001"

executed_action:
  executed_action_id: "EA-001"
  planned_action_id: "PA-001"
  raw_action_summary: ""
  normalized_representation: ""
  immutable_hash: "sha256:"
  deviation_status: "none | minor | major | emergency | unauthorized"
```

Requires re-approval if any of these change:

- target;
- command/script/API body/SQL;
- effective identity;
- permission level;
- secret source;
- region/account/cluster/database;
- rollback plan;
- risk level;
- concurrency or batch size.

## Preview support

If an executor supports preview, dry-run, diff, plan, or simulation, use it before mutating production unless an emergency exception is documented.

If preview is unsupported:

```yaml
preview:
  supported: false
  compensation_controls:
    - peer review
    - backup or snapshot
    - smaller batch
    - manual checkpoint
    - pre-state capture
    - explicit rollback/recovery plan
```

## Manual and GUI execution

Manual operations are first-class executions, not informal notes.

Manual/GUI steps must include:

- system or console;
- navigation path;
- exact instruction;
- expected screen or state;
- before/after evidence;
- operator;
- timestamp;
- deviation notes;
- sign-off if required.

## Multi-target execution

Fleet execution must not be treated as single-target execution.

Required controls:

- explicit target selector;
- target expansion evidence;
- batch strategy;
- max parallelism;
- rate limits;
- per-target result;
- stop condition;
- rollback strategy per target, batch, or global scope.

Example:

```yaml
multi_target_execution:
  target_scope: "fleet"
  batch_strategy:
    mode: "rolling"
    batch_size: 5
    max_parallel: 2
  stop_conditions:
    max_failed_targets: 1
    validation_failure_action: "stop_and_review"
  per_target_record_required: true
```

## AI-assisted execution

AI-assisted execution follows AI governance.

Rules:

- AI tool calls must map to approved Planned Actions or approved diagnostic permissions.
- AI must not expand scope by inference.
- AI must not elevate privilege without approval.
- AI-generated commands must be reviewed for R2+ or any sensitive/privileged operation.
- AI execution evidence must include role, autonomy level, prompt summary/hash, tool calls, approvals, and outputs.

## Invariants

- Every executed action maps to an approved planned action.
- Every deviation is classified.
- Every state-changing action has evidence.
- Every non-previewable action has compensating controls.
- Every rollback is executable, verifiable, or explicitly unavailable.
- Every manual action has enough evidence to be reviewed.
- Every multi-target operation is batch-aware or explicitly risk-accepted.

## Anti-patterns

- Tying SDO to one command language.
- Confusing runbook with execution record.
- Accepting fake preview.
- Not separating planned and actual execution.
- Assuming idempotency without declaring or testing it.
- Running fleet-wide changes without batch controls.
- Treating GUI operations as unauditable.
- Giving AI broad production permissions.
- Logging secrets as execution evidence.
