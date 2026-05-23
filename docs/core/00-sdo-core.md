# SDO Core

Spec-Driven Operations is a technology-agnostic methodology for governing IT operations through verifiable Operational Specs, risk-adapted controls, controlled execution, validation, evidence, and safe-state recovery.

SDO is agentic-first and manual-compatible: it expects artifacts to be precise enough for AI agents to use safely, while preserving the same lifecycle for human-only execution.

## Definition

**Spec-Driven Operations** transforms operational intent into a safe, approved, traceable, validated, and recoverable operation.

SDO does this by requiring each relevant operation to define what is intended, where it applies, what risk it carries, who may authorize it, how it will be executed, how success will be validated, what evidence will prove the result, and how the system can return to an accepted safe state if something fails.

## The core distinction from SDD

```text
SDD converts intent into verifiable code.
SDO converts operational intent into controlled execution, evidence, validation, and safe recovery.
```

SDO inherits the spec-first discipline from SDD, but it does not copy the development lifecycle literally. In software development, the main output is code. In operations, the main output is a controlled change or observation on a live system.

## Problem statement

Operational work often starts too late in the lifecycle: with a command, a script, a ticket comment, or an urgent instruction. That creates avoidable risk:

- unclear scope;
- unreviewed blast radius;
- missing rollback;
- weak validation;
- evidence collected after the fact;
- approvals that do not correspond to risk;
- automation that amplifies mistakes;
- AI assistance without operational accountability.

SDO moves the starting point from commands to an Operational Spec.

## Core lifecycle

The compact lifecycle is:

```text
Intent → Spec → Plan → Runbook → Execution → Validation → Review
```

The full lifecycle is:

```text
Intent
→ Operational Spec
→ Risk & Profile
→ Execution Plan
→ Planned Actions
→ Runbook / Executor Binding
→ Approval
→ Pre-Execution Check
→ Execution
→ Validation
→ Evidence Closure
→ Review / Recovery / Archive
```

The compact lifecycle is useful for communication. The full lifecycle is useful for implementation, tooling, training, and audit.

## Transversal controls

SDO has controls that are not merely sequential phases:

| Control | Purpose |
|---|---|
| Risk / Approval | Decide the required rigor and authority. |
| Evidence | Prove what was approved, executed, validated, and closed. |
| Rollback / Recovery | Define how to reach an accepted safe state. |
| AI Governance | Control AI roles, autonomy, tool use, and accountability. |

## Source-of-truth model

SDO separates two truths:

```text
Operational Spec = source of truth for approved intent.
Evidence = source of truth for what actually happened.
```

This distinction matters. A spec can define what should happen, but live systems may drift, fail partially, behave unexpectedly, or be changed by other actors. Evidence is what proves the real outcome.

## Core rules

These rules define the minimum discipline of SDO:

1. No meaningful operation starts with commands.
2. Every relevant operation starts with intent and a spec.
3. Every operation has an SDO ID.
4. Every operation is classified by base class and overlays.
5. Every operation selects a risk-adapted profile.
6. Every state-changing action maps to an approved Planned Action.
7. Every executed action is recorded.
8. Every deviation is classified.
9. Every state-changing action has evidence.
10. Every operation has validation criteria before execution.
11. Every operation declares rollback, recovery, or explicit no-rollback acceptance.
12. Every operation closes with enough evidence to be reviewed by someone other than the operator.

## What counts as an SDO operation

An SDO operation may be:

- observing state;
- analyzing a plan or diff;
- maintaining a system;
- changing configuration;
- deploying a version;
- controlling runtime behavior;
- operating on data;
- changing access or security posture;
- remediating an incident;
- destroying or decommissioning resources.

The operation may be executed through Linux, Windows, SQL, APIs, Kubernetes, cloud CLIs, IaC, GitOps, CI/CD, runbook platforms, manual GUI steps, or AI-assisted tools.

## Profiles

SDO adapts rigor by profile:

| Profile | Use |
|---|---|
| SDO-Lightweight | Low-risk, bounded, repeatable operations. |
| SDO-Normal | Usual operational changes. |
| SDO-Critical | High impact, production-critical, data/security-sensitive, broad-scope, destructive, or hard-to-revert operations. |
| SDO-Emergency | Active incident or imminent risk. |
| SDO-Pipeline-Backed | GitOps, IaC, CI/CD, or automation-backed operations. |

## Standalone and integrable

SDO Core must work with humans, Markdown, YAML, tickets, PRs, pipelines, or manual processes.

Agent systems such as Gentle AI are optional integrations, not dependencies. They may assist with drafting, analysis, orchestration, validation, and evidence packaging, but they do not define SDO and do not replace human accountability.

For the agentic operating model, see [`04-agentic-sdo.md`](04-agentic-sdo.md).

## Minimum conformance

A process is using SDO only if it can show, at minimum:

- an SDO ID;
- an Operational Spec or lightweight equivalent;
- operation classification and profile;
- planned action or runbook;
- execution record;
- validation result;
- evidence;
- rollback/recovery/no-rollback declaration;
- closure or review note.

Higher-risk profiles require stronger forms of each artifact and control.

## Anti-patterns

SDO explicitly rejects:

- “SDD with commands”;
- spec theater;
- approval theater;
- treating all read-only work as low risk;
- treating automation as inherently safe;
- closing because a command returned exit code 0;
- rollback text without rollback capability;
- evidence collected only after something goes wrong;
- AI running production without approved scope, least privilege, evidence, and human accountability.
