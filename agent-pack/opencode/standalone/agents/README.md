# Standalone OpenCode Agents

This directory contains the standalone SDO OpenCode agent prompt assets. The agent markdown file is the prompt source of truth; [`../opencode.overlay.json`](../opencode.overlay.json) is only a companion config overlay for registration fields such as mode and permissions.

## Current MVP Scope

- [`sdo-orchestrator.md`](sdo-orchestrator.md) is the source prompt for the standalone command-routed coordinator for `/sdo-init` and `/sdo-new`.
- The orchestrator defines routing, preflight checks, dependency gates, artifact-store decisions, and phase result envelopes.
- Phase agents do not exist yet.
- Execution support is intentionally not included. Do not install or advertise `/sdo-execute` from this MVP asset set.

## Installation Notes

Installers should install `sdo-orchestrator.md` as the OpenCode agent file, then merge or load the companion `opencode.json` overlay only for config fields that belong in OpenCode configuration. Do not copy the prompt body into `opencode.overlay.json`; that creates a second source that can drift.

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
