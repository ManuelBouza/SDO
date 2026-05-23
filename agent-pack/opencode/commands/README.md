# SDO OpenCode Commands

SDO command assets are split by runtime mode so installers do not have to infer which orchestrator should receive a slash command.

## Runtime Sets

| Runtime mode | Source directory | Command agent |
|---|---|---|
| Gentle integration | `agent-pack/opencode/gentle-integration/commands/` | `gentle-orchestrator` |
| Standalone | `agent-pack/opencode/standalone/commands/` | `sdo-orchestrator` |

## Installer Rule

Installers should copy exactly one runtime set into the target OpenCode command location. Do not merge both sets into the same command namespace because the file names are intentionally identical and differ by orchestrator route.

The initial command assets are:

- `sdo-init.md`
- `sdo-new.md`

`/sdo-execute` is intentionally not included yet.

## Command Responsibilities

Commands are thin wrappers. They identify the workflow, provide runtime context, name the authoritative shared contracts, and instruct the active orchestrator what to coordinate.

Commands must not perform SDO phase work inline. Phase work, dependency gates, artifact updates, and routing decisions belong to the active orchestrator and phase agents.

## Authoritative Contracts

Shared contracts under `agent-pack/opencode/shared/` are authoritative for both runtime modes:

- `phase-result-envelope.yaml`
- `artifact-contract.yaml`
- `operation-layout.md`
- `manifest-template.yaml`
- `artifact-state-machine.md`
- `preflight-checklist.md`
- `dependency-gates.md`
