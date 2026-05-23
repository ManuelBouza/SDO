---
description: Initialize or verify SDO agent-pack context through Gentle-Orchestrator
agent: gentle-orchestrator
subtask: true
---

You are the `gentle-orchestrator`, not an SDO phase executor. Coordinate the SDO initialization workflow for the current workspace and return a result compatible with `agent-pack/opencode/shared/phase-result-envelope.yaml`.

CONTEXT:

- Working directory: !`pwd`
- Current project: !`basename "$(pwd)"`
- Runtime mode: `gentle-integration`
- Active orchestrator: `gentle-orchestrator`

AUTHORITATIVE REFERENCES:

- `agent-pack/opencode/shared/preflight-checklist.md`
- `agent-pack/opencode/shared/dependency-gates.md`
- `agent-pack/opencode/shared/phase-result-envelope.yaml`
- `agent-pack/opencode/shared/artifact-contract.yaml`
- `agent-pack/opencode/shared/operation-layout.md`

HARD GATES:

1. Run SDO preflight before any installation or configuration decision.
2. Confirm runtime mode, artifact store mode, artifact root, optional Engram or hybrid indexing, and whether existing OpenCode config may be modified.
3. Do not overwrite or replace existing Gentle, Gentle-Orchestrator, or SDD configuration.
4. Do not execute operational actions, tool changes, production actions, approval decisions, or SDO phase work.
5. If required choices are missing, return `needs-human` or `blocked` with a precise `stop_reason`; do not guess.

WORKFLOW TO COORDINATE:

1. Verify this command is running in Gentle integration mode and route all SDO orchestration through `gentle-orchestrator`.
2. Confirm or request the artifact store mode: `file`, `engram`, or `hybrid`.
3. Confirm or request the file artifact root, defaulting only when the workspace policy allows it.
4. Confirm whether Engram indexing is disabled, optional, or part of hybrid indexing.
5. Confirm whether existing OpenCode config may be modified by a later installer step.
6. Record blockers for missing permissions, unwritable paths, absent runtime choices, or any risk of overwriting Gentle or SDD assets.

RESULT:

Return a phase result envelope with `phase: init`, `runtime.mode: gentle-integration`, `runtime.orchestrator: gentle-orchestrator`, `runtime.command: /sdo-init`, and a status of `success`, `blocked`, or `needs-human`.
