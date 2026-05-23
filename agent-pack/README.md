# SDO OpenCode Agent Pack

This directory contains experimental MVP assets for running SDO workflows in OpenCode.

The first assets define shared protocol contracts before commands, agents, or installers are added. They are intended to work in both runtime modes described by the design:

- Gentle integration mode, where SDO assets plug into an existing Gentle-Orchestrator runtime.
- Standalone SDO-Orchestrator mode, where SDO owns phase routing, preflight checks, and dependency gates.

Files remain the source of truth for the MVP. Optional Engram or hybrid indexing may help agents resume context, but audit and routing decisions must be based on operation artifacts under `sdo/operations/<SDO ID>/`.

## Shared Assets

- [`opencode/shared/phase-result-envelope.yaml`](opencode/shared/phase-result-envelope.yaml) defines the compact handoff contract returned by phase agents, skills, and commands.
- [`opencode/shared/artifact-contract.yaml`](opencode/shared/artifact-contract.yaml) defines the file-based operation manifest and artifact graph.
- [`opencode/shared/preflight-checklist.md`](opencode/shared/preflight-checklist.md) defines checks the orchestrator must run before phases and execution.
- [`opencode/shared/dependency-gates.md`](opencode/shared/dependency-gates.md) defines phase prerequisites, blocked conditions, and autonomy distinctions.

## Design Reference

- [`../docs/agent-pack/opencode-agent-pack-design.md`](../docs/agent-pack/opencode-agent-pack-design.md)
