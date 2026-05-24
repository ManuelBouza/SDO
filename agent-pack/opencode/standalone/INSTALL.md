# Standalone OpenCode Installation

This guide explains how to manually test the SDO OpenCode pack in standalone mode. Standalone mode uses `sdo-orchestrator` directly and currently supports only `/sdo-init` and `/sdo-new`.

The current MVP is easiest to test from this repository checkout because several skill and command assets reference authoritative files by repo-relative paths such as `agent-pack/opencode/shared/`. Copying shared assets into another project is possible, but those references need path adaptation until an installer exists.

## Before You Start

- Use this guide only for OpenCode. No Codex or Claude Code adapter exists yet.
- Do not overwrite an existing `AGENTS.md`. Merge only the marker-delimited SDO protocol block from `agent-pack/opencode/AGENTS.md` when those global reminders are desired.
- Register `sdo-orchestrator` by installing the agent markdown file, with `opencode.json` or a generated overlay used only for companion config fields. Do not rely on `AGENTS.md` for command routing or skill discovery.
- Restart OpenCode after installing or changing agents, commands, skills, or instruction files. OpenCode loads these assets at startup.

## Copy Map

Install the standalone assets into the target project using these canonical OpenCode paths:

| Source | Target |
|---|---|
| `agent-pack/opencode/standalone/agents/sdo-orchestrator.md` | `.opencode/agent/sdo-orchestrator.md` |
| `agent-pack/opencode/standalone/opencode.overlay.json` | optional companion config: merge into `opencode.json`, or load as an explicit/generated OpenCode config overlay |
| `agent-pack/opencode/standalone/commands/sdo-init.md` | `.opencode/command/sdo-init.md` |
| `agent-pack/opencode/standalone/commands/sdo-new.md` | `.opencode/command/sdo-new.md` |
| `agent-pack/opencode/skills/*` | `.opencode/skills/` |
| `agent-pack/opencode/AGENTS.md` | Optional merge into `~/.config/opencode/AGENTS.md` or project `AGENTS.md` as protocol reminders only. |

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
3. Optionally merge `standalone/opencode.overlay.json` into the target `opencode.json`, or load it as an explicit/generated OpenCode overlay. Do not add a `prompt` field there.
4. Copy `standalone/commands/sdo-init.md` and `standalone/commands/sdo-new.md` to `.opencode/command/`.
5. Copy each directory under `agent-pack/opencode/skills/` into `.opencode/skills/`.
6. Optionally merge the marker-delimited protocol block from `agent-pack/opencode/AGENTS.md` into `~/.config/opencode/AGENTS.md` or a project `AGENTS.md`. Do not add skill indexes or routing tables there.
7. Keep `agent-pack/opencode/shared/` available from the project root, or copy it to `.opencode/sdo/shared/` and update installed references consistently.
8. Quit and restart OpenCode.

## Verification Checklist

- [ ] OpenCode was restarted after copying assets.
- [ ] The `sdo-orchestrator` agent is available from `.opencode/agent/sdo-orchestrator.md`, not from `AGENTS.md` routing text or a copied inline prompt.
- [ ] `/sdo-init` routes to `sdo-orchestrator` and returns a success, blocked, or needs-human envelope.
- [ ] `/sdo-new "test standalone operation"` routes to `sdo-orchestrator` and creates or blocks operation artifacts based on durable state.
- [ ] Created artifacts, blockers, and stop reasons are visible in file artifacts or the returned envelope.
- [ ] No `/sdo-execute` command is installed or advertised.
- [ ] Existing `AGENTS.md`, Gentle, SDD, and OpenCode configuration were merged safely, not overwritten unexpectedly.

## Uninstall

1. Remove `.opencode/agent/sdo-orchestrator.md` if it came from this standalone pack.
2. Remove the `agent.sdo-orchestrator` entry from the installed `opencode.json` or stop loading the generated overlay if either was used.
3. Remove `.opencode/command/sdo-init.md` and `.opencode/command/sdo-new.md` if they came from this standalone pack.
4. Remove `.opencode/skills/sdo-shared-protocol/`, `.opencode/skills/sdo-artifact-store/`, and `.opencode/skills/sdo-spec-authoring/` if they came from this standalone pack.
5. Remove `.opencode/sdo/shared/` only if it was created solely for this pack and no operation artifacts depend on it.
6. Remove the marker-delimited SDO protocol block from `AGENTS.md` if it was installed. Do not delete unrelated project guidance.
7. Restart OpenCode.

## Known Limitations

- OpenCode is the only supported implementation target today.
- The standalone MVP supports `/sdo-init` and `/sdo-new` only.
- `/sdo-execute` is intentionally absent.
- Standalone phase agents do not exist yet.
- Manual copying can create stale or inconsistent paths. Until an installer exists, repo-checkout testing is safer than copying shared assets into arbitrary projects.
- File artifacts remain the source of truth. Engram or chat history may help with continuity, but they are not audit-grade SDO operation state.
