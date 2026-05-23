# SDO OpenCode Agent Pack

This directory contains experimental MVP assets for running SDO workflows in OpenCode.

The first assets define shared protocol contracts and initial command wrappers. They are intended to work in both runtime modes described by the design:

- Gentle integration mode, where SDO assets plug into an existing Gentle-Orchestrator runtime.
- Standalone SDO-Orchestrator mode, where SDO owns phase routing, preflight checks, and dependency gates.

Files remain the source of truth for the MVP. Optional Engram or hybrid indexing may help agents resume context, but audit and routing decisions must be based on operation artifacts under `sdo/operations/<SDO ID>/`.

## Shared Assets

- [`opencode/shared/phase-result-envelope.yaml`](opencode/shared/phase-result-envelope.yaml) defines the compact handoff contract returned by phase agents, skills, and commands.
- [`opencode/shared/artifact-contract.yaml`](opencode/shared/artifact-contract.yaml) defines the file-based operation manifest and artifact graph.
- [`opencode/shared/operation-layout.md`](opencode/shared/operation-layout.md) defines the default operation directory layout, SDO ID naming, source-of-truth rules, safe updates, external references, and evidence handling.
- [`opencode/shared/manifest-template.yaml`](opencode/shared/manifest-template.yaml) provides the starter manifest that `/sdo-new` can copy for a draft operation.
- [`opencode/shared/artifact-state-machine.md`](opencode/shared/artifact-state-machine.md) defines lifecycle and artifact status transitions for routing, continuation, failure, supersession, and archive.
- [`opencode/shared/preflight-checklist.md`](opencode/shared/preflight-checklist.md) defines checks the orchestrator must run before phases and execution.
- [`opencode/shared/dependency-gates.md`](opencode/shared/dependency-gates.md) defines phase prerequisites, blocked conditions, and autonomy distinctions.

## Command Assets

Initial `/sdo-init` and `/sdo-new` OpenCode command assets now exist in runtime-specific directories:

- [`opencode/gentle-integration/commands/`](opencode/gentle-integration/commands/) routes commands to `gentle-orchestrator`.
- [`opencode/standalone/commands/`](opencode/standalone/commands/) routes commands to `sdo-orchestrator`.

Installers should copy one command set into the target OpenCode command location. The commands are thin wrappers around orchestrator workflows; shared contracts under [`opencode/shared/`](opencode/shared/) remain authoritative. `/sdo-execute` is intentionally not included yet.

See [`opencode/commands/README.md`](opencode/commands/README.md) for command packaging rules.

## Skill Assets

Initial reusable OpenCode skills now exist under [`opencode/skills/`](opencode/skills/):

- [`sdo-shared-protocol`](opencode/skills/sdo-shared-protocol/SKILL.md) for phase coordination, gates, and envelope-compatible results.
- [`sdo-artifact-store`](opencode/skills/sdo-artifact-store/SKILL.md) for file-based operation state, manifests, artifact refs, and evidence refs.
- [`sdo-spec-authoring`](opencode/skills/sdo-spec-authoring/SKILL.md) for Operational Spec drafting as an agentic contract.

See [`opencode/skills/README.md`](opencode/skills/README.md) for skill packaging rules.

## Design Reference

- [`../docs/agent-pack/opencode-agent-pack-design.md`](../docs/agent-pack/opencode-agent-pack-design.md)
