# SDO OpenCode Global Instructions

This file is a mergeable global-instructions block for OpenCode. It is intended for `~/.config/opencode/AGENTS.md` or a project `AGENTS.md` when a project explicitly wants the same SDO protocol reminders.

It is not a skill index, command registry, or orchestrator router. OpenCode runtime assets are installed separately:

- orchestrator and future phase agents are registered through `opencode.json` or a generated `opencode.json` overlay;
- slash commands live under the SDO OpenCode `commands/` asset directories;
- reusable skills live under `skills/<skill-name>/SKILL.md`.

<!-- sdo:opencode-protocol:start -->

## SDO Protocol Reminders

- Files and stable external references are the source of truth for SDO MVP routing, audit, approval, execution, validation, review, and archive state.
- Fail closed. If required state is missing, stale, contradictory, unsafe, or only present in chat history, return `blocked` or `needs-human`.
- Do not trust chat history, agent memory, or unstored output as proof that a gate passed.
- Do not substitute execution for approval, approval for execution, validation for execution, or chat confirmation for durable artifacts.
- Do not perform operational execution unless the current phase, autonomy level, approval boundary, tool policy, and durable artifacts explicitly allow it. The current OpenCode MVP has no `/sdo-execute` support.
- Do not overwrite existing SDO, Gentle, SDD, or OpenCode configuration without explicit human approval and a safe merge path.

## Current OpenCode MVP Scope

- SDO is OpenCode-only today; no Codex, Claude Code, or other runtime adapter exists yet.
- Supported commands are `/sdo-init` and `/sdo-new` only.
- Standalone mode routes those commands to `sdo-orchestrator` through OpenCode agent configuration.
- Phase agents are not part of the current standalone MVP.
- `/sdo-execute` is deferred. Do not create, advertise, route, or simulate execution support from this MVP pack.

<!-- sdo:opencode-protocol:end -->
