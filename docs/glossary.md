# Glossary

This glossary defines the vocabulary used across SDO. These terms are normative unless a specific document states otherwise.

## Core concepts

### SDO

Spec-Driven Operations: a technology-agnostic methodology for governing IT operations through verifiable Operational Specs, risk-adapted controls, controlled execution, validation, evidence, and safe-state recovery.

### SDO ID

Unique identifier for an SDO operation, for example `SDO-20260523-001`. The SDO ID links specs, plans, runbooks, approvals, execution records, validation reports, evidence, reviews, tickets, PRs, and pipeline runs.

### SDO Operation

A traceable unit of operational work governed by SDO. It may be a diagnostic action, configuration change, deployment, data operation, access change, remediation, manual procedure, pipeline-backed change, or emergency response.

### Operational Intent

The reason, objective, urgency, expected outcome, and operational context behind an operation.

### Operational Spec

The source of truth for approved operational intent, scope, limits, expected state, success criteria, accepted risk, and recovery strategy.

The Operational Spec defines what is approved. It does not prove what happened.

### Evidence

The source of truth for what actually happened. Evidence may include logs, event IDs, command outputs, request IDs, screenshots, metrics, pipeline artifacts, hashes, approvals, validation results, and review records.

### Evidence Package

An indexed, protected, and traceable package of evidence references, hashes, timestamps, actors, sensitivity classifications, retention rules, and closure status. It should be sufficient, not exhaustive.

## Lifecycle terms

### Intent

Lifecycle phase where the operational need is captured before designing or executing anything.

### Spec

Lifecycle phase where the intent is turned into a verifiable Operational Spec.

### Plan

Lifecycle phase where the strategy, order, dependencies, target scope, risk controls, validation, and recovery approach are defined.

### Runbook

The executable or manual procedure used to perform the operation. A runbook may be Markdown, YAML, a checklist, a script, an API workflow, a SQL script, a pipeline, a GitOps action, or manual GUI instructions.

### Execution

The actual application of approved Planned Actions against real operational targets.

### Validation

Evidence-backed confirmation that the observed final state satisfies the Operational Spec and does not violate critical constraints.

### Review

The closure phase where result, deviations, lessons, follow-ups, and archival decisions are recorded.

## Action and execution terms

### Planned Action

An approved operational action with preconditions, expected result, validation, evidence, stop condition, and rollback or recovery mapping.

### Executed Action

The action that actually ran. It must map to an approved Planned Action or be classified as a deviation.

### Executor

The mechanism that performs an action. Examples: shell, PowerShell, SQL engine, API client, Kubernetes, cloud CLI, IaC tool, GitOps controller, CI/CD pipeline, runbook platform, human operator, GUI, or AI-assisted tool.

### Executor Binding

The mapping between an abstract Planned Action and the concrete executor that will perform it.

### Execution Record

The record of what was actually executed, by whom, when, where, with which executor, and with what result.

### Deviation

A difference between approved plan and actual execution, validation result, scope, target, identity, timing, risk, or recovery path. Deviations must be classified and evidenced.

## Classification terms

### Base Class

The operational effect of an action. Recommended classes are Observe, Analyze, Maintain, Configure, Deploy, Control, Data, Access, Remediate, and Destroy.

### Overlay

A contextual modifier that changes rigor or risk. Examples: Environment, Sensitivity, Privilege, Scope, Reversibility, Availability Impact, Data Impact, Security Impact, Cost Impact, Compliance Impact, Execution Mode, Urgency, State Model, and Evidence Sensitivity.

### Profile

The SDO flow variant selected by risk and context: SDO-Lightweight, SDO-Normal, SDO-Critical, SDO-Emergency, or SDO-Pipeline-Backed.

### Risk Tier

The effective risk level derived from base class and overlays. SDO uses the most severe relevant factor, not an average that hides critical risk.

## Gate and control terms

### Gate

A control point that decides whether an operation can proceed, hold, abort, rollback, escalate, accept risk, or close.

### Approval

An explicit authorization to proceed, proportional to risk. Approval may be manual, peer-based, owner-based, security/data-owner-based, emergency authority, or policy/pipeline-based.

### Precheck

A verification performed before execution, such as identity, permissions, current state, backup, maintenance window, monitoring, dependencies, or preview.

### Postcheck

A verification performed after execution, such as service health, metrics, logs, data integrity, effective configuration, or expected business outcome.

### Stop Condition

An observable condition that requires pausing, aborting, escalating, or switching to rollback or recovery.

## Recovery terms

### Rollback

In SDO, rollback means safe-state recovery capability, not merely undo.

### Recovery

The broader process of returning a system to an accepted safe or operational state. Recovery may include rollback, restore, roll-forward, failover, compensation, mitigation, or manual repair.

### Roll-forward

Moving to a corrected future state instead of returning to the previous state.

### Restore

Recovering state from backup, snapshot, image, restore point, or point-in-time recovery.

### Failover

Moving service to another healthy node, region, cluster, instance, or provider path.

### Compensating Action

An action that neutralizes or offsets an effect that cannot be directly undone.

### Mitigation

An action that reduces impact without necessarily solving root cause or restoring the original state.

### Point of No Return

A declared point where the recovery strategy changes in cost, risk, or feasibility.

### No-rollback

A state where exact rollback is not technically viable. No-rollback requires explicit risk acceptance and a mitigation or recovery plan.

## AI terms

### AI-assisted Operation

An operation where AI contributes to research, drafting, analysis, review, execution assistance, evidence summarization, or guarded tool use.

### AI Autonomy Level

The maximum permitted AI autonomy for an operation, from A0 (No AI/manual) to A6 (policy-bound autonomous operations).

If AI must not be used, state that AI is prohibited for the operation or context instead of assigning an autonomy level.

### AI Decision Trace

An evidence record of AI role, model/tool, prompt summary or hash, context, recommendations, tool calls, approvals, policy decisions, and human modifications.

### Human Accountability

The rule that AI may be responsible for drafting or assistance, but humans, systems of record, or approved policies remain accountable for operational risk.
