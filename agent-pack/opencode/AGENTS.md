# SDO OpenCode Agent Skills Index

When working on SDO operations in a project using this OpenCode pack, load the relevant SDO skill(s) before doing SDO work.

This file is an installable or mergeable agent-facing instruction layer for OpenCode projects. It follows the same pattern as Gentle AI's root `AGENTS.md`: use it as a compact skill index and runtime reminder, not as a replacement for the authoritative SDO artifacts.

The paths below are relative to this SDO OpenCode pack directory. If you merge this block into a project root `AGENTS.md` after copying skills into `.opencode/skills/`, adapt the paths to the installed location.

## How to Use

1. Check the trigger column to find skills that match the current SDO task.
2. Load the skill by reading the `SKILL.md` file at the listed path, or use OpenCode's skill loader when available.
3. Follow all hard rules, gates, and output contracts from the loaded skill.
4. Read the referenced shared assets before making routing, artifact, approval, or execution decisions.
5. If more than one skill applies, load all relevant skills before continuing.

## Skills

| Skill | Trigger | Path |
|---|---|---|
| `sdo-shared-protocol` | Coordinating SDO phases, validating gates, returning phase result envelopes, resuming operations, or operating under the SDO OpenCode agent-pack. | [`skills/sdo-shared-protocol/SKILL.md`](skills/sdo-shared-protocol/SKILL.md) |
| `sdo-artifact-store` | Creating or updating SDO operation directories, manifests, artifact refs, evidence refs, external refs, hashes, or file-based operation state. | [`skills/sdo-artifact-store/SKILL.md`](skills/sdo-artifact-store/SKILL.md) |
| `sdo-spec-authoring` | Drafting, refining, reviewing, or repairing Operational Specs as agentic contracts with autonomy, approval, validation, evidence, and recovery controls. | [`skills/sdo-spec-authoring/SKILL.md`](skills/sdo-spec-authoring/SKILL.md) |

## Command and Runtime Notes

- Current OpenCode command assets are `/sdo-init` and `/sdo-new` only.
- Standalone mode routes those commands to `sdo-orchestrator`.
- Gentle integration mode routes those commands to `gentle-orchestrator`.
- `/sdo-execute` is not supported yet. Do not create, advertise, route, or simulate execution support from this MVP pack.
- Phase agents are not part of the current standalone MVP. The standalone orchestrator coordinates init and new-operation workflows only.

## Hard Rules

- File artifacts are the source of truth for SDO MVP routing, audit, approval, execution, validation, review, and archive state.
- Fail closed. If required state is missing, stale, contradictory, unsafe, or only present in chat history, return `blocked` or `needs-human`.
- Do not trust chat history, agent memory, or unstored output as proof that a gate passed.
- Do not substitute execution for approval, approval for execution, validation for execution, or chat confirmation for durable artifacts.
- Do not perform operational execution unless the current phase, autonomy level, approval boundary, tool policy, and durable artifacts explicitly allow it. The current MVP has no `/sdo-execute` support.
- Do not overwrite existing SDO, Gentle, SDD, or OpenCode configuration without explicit human approval and a safe merge path.
