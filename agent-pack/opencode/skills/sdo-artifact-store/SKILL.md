---
name: sdo-artifact-store
description: "Trigger: SDO operation files, manifest, artifact refs, evidence refs. Maintain file-based SDO operation state safely."
license: Apache-2.0
metadata:
  author: gentleman-programming
  version: "0.1"
---

# SDO Artifact Store

## Activation Contract

Use this skill when creating or updating SDO operation files, manifests, artifact refs, evidence refs, external refs, hashes, or file-based operation state.

## Hard Rules

- Read the authoritative artifact assets before changing operation state: `agent-pack/opencode/shared/artifact-contract.yaml`, `agent-pack/opencode/shared/operation-layout.md`, `agent-pack/opencode/shared/manifest-template.yaml`, and `agent-pack/opencode/shared/artifact-state-machine.md`.
- Files are the source of truth for MVP routing, audit, approval, execution, validation, review, and archive.
- Create each operation under `sdo/operations/<SDO ID>/`; the directory name must match `operation.sdo_id` in `manifest.yaml`.
- Never rewrite execution history silently. Corrections must be appended with timestamp, actor, reason, and replacement reference.
- Preserve approved versions of specs, runbooks, approvals, and policies. New versions must reference the approved version they replace.
- Append or merge evidence. Do not delete failed, partial, superseded, redaction-blocked, or inconvenient evidence entries.
- Engram is optional continuity memory only; it must not be the authoritative store for operation state.

## Decision Gates

| Condition | Action |
|---|---|
| New operation has no directory | Create `sdo/operations/<SDO ID>/` and initialize `manifest.yaml`. |
| Existing artifact changes approved state | Preserve the prior version and record the replacement reference. |
| Evidence is new or corrected | Append or merge evidence refs and update manifest timestamps/hashes. |
| Operation is archived | Do not mutate normal artifacts; append an explicit post-archive correction record if needed. |

## Execution Steps

1. Load the artifact contract, layout, manifest template, and state machine.
2. Resolve or create the operation directory and confirm the manifest `SDO ID` matches the path.
3. Update manifest lifecycle, dependency state, artifact refs, external refs, approval refs, evidence refs, blockers, and deviations from durable sources only.
4. Keep rejected, blocked, failed, rolled-back, superseded, and accepted-risk states visible.
5. Recompute or clear hashes according to the artifact contract after intentional edits.

## Output Contract

Return changed files, manifest fields updated, evidence refs added or merged, blockers/deviations changed, and any unsafe condition that requires `blocked` or `needs-human`.

## References

- `agent-pack/opencode/shared/artifact-contract.yaml`
- `agent-pack/opencode/shared/operation-layout.md`
- `agent-pack/opencode/shared/manifest-template.yaml`
- `agent-pack/opencode/shared/artifact-state-machine.md`
