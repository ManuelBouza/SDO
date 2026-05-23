# SDO: Spec-Driven Operations

SDO is a technology-agnostic methodology for designing, approving, executing, validating, and auditing IT operations through operational specifications.

SDO is agentic-first and manual-compatible: its artifacts are precise enough for AI agents to consume under policy, while remaining usable by human operators without AI.

## Repository status

This repository is a draft public baseline for an evolving methodology. Core documentation is intended to be usable and reviewable, while some examples and integrations may continue to mature.

Research notes are non-normative and live in [`docs/research/`](docs/research/). They provide context and background, but do not define binding SDO methodology.

## What SDO solves

SDO reduces improvised operations by making operational intent explicit, adapting control to risk, preserving traceability, defining rollback or recovery before execution, and closing every relevant operation with validation and evidence.

## What SDO is not

SDO does not replace ITIL, SRE, GitOps, Infrastructure as Code, runbooks, CI/CD pipelines, incident response, ticketing systems, or AI agents. It orchestrates them through a common operational lifecycle.

## Base lifecycle

```text
Intent → Spec → Plan → Runbook → Execution → Validation → Review
```

With transversal controls:

```text
Risk / Approval
Evidence
Rollback / Recovery
AI Governance
```

## Core artifacts

- Operational Spec
- Execution Plan
- Runbook
- Execution Record
- Validation Report
- Evidence Package
- Review Record

## Start here

Read [`docs/start-here.md`](docs/start-here.md).

## Documentation map

| Area | Documents |
|---|---|
| Core | [`docs/core/00-sdo-core.md`](docs/core/00-sdo-core.md), [`docs/core/01-principles.md`](docs/core/01-principles.md), [`docs/core/02-lifecycle.md`](docs/core/02-lifecycle.md), [`docs/core/03-scope-and-non-goals.md`](docs/core/03-scope-and-non-goals.md), [`docs/core/04-agentic-sdo.md`](docs/core/04-agentic-sdo.md) |
| Classification | [`docs/reference/taxonomy.md`](docs/reference/taxonomy.md), [`docs/reference/profiles.md`](docs/reference/profiles.md) |
| Artifacts and execution | [`docs/reference/artifacts.md`](docs/reference/artifacts.md), [`docs/reference/gates-and-controls.md`](docs/reference/gates-and-controls.md), [`docs/reference/execution-model.md`](docs/reference/execution-model.md) |
| Guides | [`docs/guides/operator-guide.md`](docs/guides/operator-guide.md), [`docs/guides/platform-tooling-guide.md`](docs/guides/platform-tooling-guide.md), [`docs/guides/adoption-guide.md`](docs/guides/adoption-guide.md) |
| Guarantees | [`docs/reference/evidence-and-auditability.md`](docs/reference/evidence-and-auditability.md), [`docs/reference/rollback-and-reversibility.md`](docs/reference/rollback-and-reversibility.md) |
| AI and agentic operations | [`docs/reference/ai-governance.md`](docs/reference/ai-governance.md), [`docs/reference/agent-roles.md`](docs/reference/agent-roles.md), [`docs/reference/autonomy-levels.md`](docs/reference/autonomy-levels.md), [`docs/reference/tool-execution-policy.md`](docs/reference/tool-execution-policy.md), [`docs/reference/human-approval-model.md`](docs/reference/human-approval-model.md) |
| Future agent packs | [`docs/agent-pack/opencode-agent-pack-design.md`](docs/agent-pack/opencode-agent-pack-design.md) |
| Templates | [`docs/templates/`](docs/templates/) |

## Optional integrations

- Gentle AI
- OpenSpec-style file workflows
- Git and pull requests
- Ticketing systems
- CI/CD pipelines

See [`docs/integrations/`](docs/integrations/).

## License and contributing

SDO is licensed under the [Apache License 2.0](LICENSE). See [CONTRIBUTING.md](CONTRIBUTING.md) for contribution guidelines.
