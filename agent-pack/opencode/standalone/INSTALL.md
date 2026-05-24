# Standalone OpenCode Installation

This guide explains how to manually test the SDO OpenCode pack in standalone mode. Standalone mode uses `sdo-orchestrator` directly and currently supports only `/sdo-init` and `/sdo-new`.

The current MVP is easiest to test from this repository checkout because several skill and command assets reference authoritative files by repo-relative paths such as `agent-pack/opencode/shared/`. Copying shared assets into another project is possible, but those references need path adaptation until an installer exists.

## Before You Start

- Use this guide only for OpenCode. No Codex or Claude Code adapter exists yet.
- Do not overwrite an existing project `AGENTS.md`. Merge the SDO block from `agent-pack/opencode/AGENTS.md` into the existing file instead.
- Restart OpenCode after installing or changing agents, commands, skills, or instruction files. OpenCode loads these assets at startup.

## Copy Map

Install the standalone assets into the target project using these canonical OpenCode paths:

| Source | Target |
|---|---|
| `agent-pack/opencode/standalone/agents/sdo-orchestrator.md` | `.opencode/agent/sdo-orchestrator.md` |
| `agent-pack/opencode/standalone/commands/sdo-init.md` | `.opencode/command/sdo-init.md` |
| `agent-pack/opencode/standalone/commands/sdo-new.md` | `.opencode/command/sdo-new.md` |
| `agent-pack/opencode/skills/*` | `.opencode/skills/` |
| `agent-pack/opencode/AGENTS.md` | Merge into project `AGENTS.md`, or place at root as `AGENTS.md` if none exists. |

### Shared Assets

The shared protocol assets live under `agent-pack/opencode/shared/`:

- `phase-result-envelope.yaml`
- `artifact-contract.yaml`
- `operation-layout.md`
- `manifest-template.yaml`
- `artifact-state-machine.md`
- `preflight-checklist.md`
- `dependency-gates.md`

For the current MVP, prefer testing from a checkout that preserves the repo-relative path `agent-pack/opencode/shared/`. The skills and command prompts currently reference that path directly.

If you copy shared assets into a target project, use `.opencode/sdo/shared/` as the intended local destination, then adapt every installed SDO command, skill, and agent reference from `agent-pack/opencode/shared/...` to `.opencode/sdo/shared/...`. Do not leave mixed paths unless both locations exist and contain the same version.

## Manual Installation

1. Create the OpenCode directories if they do not already exist: `.opencode/agent/`, `.opencode/command/`, and `.opencode/skills/`.
2. Copy `standalone/agents/sdo-orchestrator.md` to `.opencode/agent/sdo-orchestrator.md`.
3. Copy `standalone/commands/sdo-init.md` and `standalone/commands/sdo-new.md` to `.opencode/command/`.
4. Copy each directory under `agent-pack/opencode/skills/` into `.opencode/skills/`.
5. Merge `agent-pack/opencode/AGENTS.md` into the project's root `AGENTS.md`, or add it as the root `AGENTS.md` when the project does not already have one.
6. Keep `agent-pack/opencode/shared/` available from the project root, or copy it to `.opencode/sdo/shared/` and update installed references consistently.
7. Quit and restart OpenCode.

## Verification Checklist

- [ ] OpenCode was restarted after copying assets.
- [ ] The `sdo-orchestrator` agent is available.
- [ ] `/sdo-init` routes to `sdo-orchestrator` and returns a success, blocked, or needs-human envelope.
- [ ] `/sdo-new "test standalone operation"` routes to `sdo-orchestrator` and creates or blocks operation artifacts based on durable state.
- [ ] Created artifacts, blockers, and stop reasons are visible in file artifacts or the returned envelope.
- [ ] No `/sdo-execute` command is installed or advertised.
- [ ] Existing `AGENTS.md`, Gentle, SDD, and OpenCode configuration were merged safely, not overwritten unexpectedly.

## Uninstall

1. Remove `.opencode/agent/sdo-orchestrator.md`.
2. Remove `.opencode/command/sdo-init.md` and `.opencode/command/sdo-new.md` if they came from this standalone pack.
3. Remove `.opencode/skills/sdo-shared-protocol/`, `.opencode/skills/sdo-artifact-store/`, and `.opencode/skills/sdo-spec-authoring/` if they came from this standalone pack.
4. Remove `.opencode/sdo/shared/` only if it was created solely for this pack and no operation artifacts depend on it.
5. Remove the SDO block from the project `AGENTS.md`. Do not delete unrelated project guidance.
6. Restart OpenCode.

## Known Limitations

- OpenCode is the only supported implementation target today.
- The standalone MVP supports `/sdo-init` and `/sdo-new` only.
- `/sdo-execute` is intentionally absent.
- Standalone phase agents do not exist yet.
- Manual copying can create stale or inconsistent paths. Until an installer exists, repo-checkout testing is safer than copying shared assets into arbitrary projects.
- File artifacts remain the source of truth. Engram or chat history may help with continuity, but they are not audit-grade SDO operation state.
