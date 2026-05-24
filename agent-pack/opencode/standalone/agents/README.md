# Standalone OpenCode Agents

This directory contains source prompt assets for standalone SDO mode. Runtime registration belongs in `opencode.json` or a generated overlay such as [`../opencode.overlay.json`](../opencode.overlay.json).

## Current MVP Scope

- [`sdo-orchestrator.md`](sdo-orchestrator.md) is the source prompt for the standalone command-routed coordinator for `/sdo-init` and `/sdo-new`.
- The orchestrator defines routing, preflight checks, dependency gates, artifact-store decisions, and phase result envelopes.
- Phase agents do not exist yet.
- Execution support is intentionally not included. Do not install or advertise `/sdo-execute` from this MVP asset set.

## Installation Notes

Installers should register agents through OpenCode configuration. For the current MVP, merge or generate an `opencode.json` overlay with `agent.sdo-orchestrator`, then install the standalone command assets in [`../commands/`](../commands/).

Do not use `AGENTS.md` as an agent registry, skill index, or routing table. `AGENTS.md` is only for mergeable global instruction and protocol reminders.

The current support target is OpenCode only, but the SDO protocol remains portable. Shared protocol, artifact, and gate rules live in [`../../shared/`](../../shared/) and should remain the source of truth for future adapters.

## Required Companion Assets

- [`../../skills/sdo-shared-protocol/SKILL.md`](../../skills/sdo-shared-protocol/SKILL.md)
- [`../../skills/sdo-artifact-store/SKILL.md`](../../skills/sdo-artifact-store/SKILL.md)
- [`../../skills/sdo-spec-authoring/SKILL.md`](../../skills/sdo-spec-authoring/SKILL.md)
- [`../../shared/phase-result-envelope.yaml`](../../shared/phase-result-envelope.yaml)
- [`../../shared/artifact-contract.yaml`](../../shared/artifact-contract.yaml)
- [`../../shared/preflight-checklist.md`](../../shared/preflight-checklist.md)
- [`../../shared/dependency-gates.md`](../../shared/dependency-gates.md)
- [`../../shared/operation-layout.md`](../../shared/operation-layout.md)
- [`../../shared/manifest-template.yaml`](../../shared/manifest-template.yaml)
- [`../../shared/artifact-state-machine.md`](../../shared/artifact-state-machine.md)
