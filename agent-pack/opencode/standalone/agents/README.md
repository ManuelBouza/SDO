# Standalone OpenCode Agents

This directory contains OpenCode agent assets for standalone SDO mode.

## Current MVP Scope

- [`sdo-orchestrator.md`](sdo-orchestrator.md) is the standalone command-routed coordinator for `/sdo-init` and `/sdo-new`.
- The orchestrator defines routing, preflight checks, dependency gates, artifact-store decisions, and phase result envelopes.
- Phase agents do not exist yet.
- Execution support is intentionally not included. Do not install or advertise `/sdo-execute` from this MVP asset set.

## Installation Notes

Installers should copy this directory's agent files into the target OpenCode agent location together with the standalone command assets in [`../commands/`](../commands/).

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
