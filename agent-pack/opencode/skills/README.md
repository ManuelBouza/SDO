# SDO OpenCode Skills

This directory packages reusable OpenCode skills for the SDO agent-pack MVP.

Each skill is a directory containing a `SKILL.md` file with YAML frontmatter. The required `name` value must match the folder name and use lowercase hyphen-separated words. The `description` should include the trigger terms that help OpenCode load the skill for the right task.

Skills are runtime instruction contracts for agents. They should point to authoritative shared assets instead of copying full methodology content into every command or future phase agent.

## Current Skills

| Skill | Use when |
|---|---|
| `sdo-shared-protocol` | Coordinating SDO phases, validating gates, returning phase result envelopes, or operating under the SDO agent-pack. |
| `sdo-artifact-store` | Creating or updating operation directories, manifests, artifact refs, evidence refs, or file-based operation state. |
| `sdo-spec-authoring` | Drafting or refining Operational Specs as agentic contracts with autonomy, tool, approval, validation, evidence, and recovery controls. |

## Packaging Rules

- Keep one skill per folder under `agent-pack/opencode/skills/<skill-name>/SKILL.md`.
- Keep frontmatter valid YAML with at least `name` and `description`.
- Keep skill names lowercase, hyphen-separated, and identical to the folder name.
- Keep skill bodies concise and instruction-oriented.
- Reference local shared files and docs by path.
- Do not rely on Engram or chat history as authoritative SDO operation state; operation files remain the source of truth.

Installers or runtime-specific adapters should copy or register this directory as part of the SDO OpenCode pack without rewriting existing user skills.
