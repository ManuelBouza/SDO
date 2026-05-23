# OpenCode Agent Pack Design

This document defines the proposed OpenCode agent pack for SDO. The pack should make SDO usable inside OpenCode in two modes: as an adapter for an existing Gentle-Orchestrator, or as a native standalone SDO-Orchestrator runtime. In both modes, the SDO protocol remains the shared core.

## Purpose

The OpenCode agent pack should turn SDO from documentation into an executable agent workflow while preserving the methodology's controls: explicit scope, approval, bounded execution, validation, evidence, and review.

The design optimizes for:

- a thin orchestration layer;
- phase-specific agents with narrow responsibilities;
- commands that route intent instead of containing complex logic;
- shared artifact contracts that work with or without AI;
- dependency gates that prevent unsafe phase skipping;
- local-first documentation and artifact storage.

## Non-goals

The MVP should not copy Gentle AI's full installer or TUI complexity.

Out of scope for the first implementation:

- a full interactive installer;
- a cross-platform package manager;
- a remote service or hosted control plane;
- replacing SDO templates, docs, or methodology references;
- replacing Gentle-Orchestrator when the user selected it intentionally;
- storing raw secrets, full terminal transcripts, or unredacted logs as evidence;
- automating production changes without the SDO approval and autonomy gates.

## Architecture Principles

| Principle | Decision |
|---|---|
| SDO protocol is shared core | Commands, agents, skills, and storage modes must implement the same SDO lifecycle and artifact contracts. |
| Gentle integration is an adapter | When Gentle-Orchestrator is present and desired, SDO adds commands, skills, and subagents that route through Gentle without redefining its runtime. |
| SDO-Orchestrator is native runtime | When Gentle is absent or not desired, SDO installs an independent orchestrator that owns SDO phase routing and gates. |
| Commands stay thin | Slash commands collect intent and call the orchestrator or phase agent; they should not duplicate lifecycle logic. |
| Phase agents stay narrow | Each agent should produce or verify one artifact class or gate decision. |
| Gates fail closed | Missing spec, approval, policy, evidence, or dependency state should stop progress, not be inferred. |
| Evidence is referenced, not dumped | Store evidence indexes, hashes, source references, and summaries; avoid raw logs unless needed and redacted. |
| Manual compatibility remains mandatory | Every artifact should remain understandable and executable by humans without OpenCode. |

## Runtime Modes

### Gentle Integration Mode

Use this mode when Gentle-Orchestrator is installed and the user wants SDO to run inside that runtime.

The SDO pack should install:

- `/sdo-*` commands as thin wrappers routed to Gentle-Orchestrator;
- SDO-specific phase subagents;
- SDO skills packaged as `SKILL.md` directories;
- shared SDO protocol references and artifact contract helpers;
- a registry entry or resolver metadata so Gentle can discover the SDO skills.

Rules:

- do not overwrite existing Gentle or SDD config;
- do not replace Gentle-Orchestrator;
- do not assume SDD and SDO share artifacts;
- preserve Gentle's command and skill conventions where the adapter runs inside Gentle;
- surface SDO gates as first-class checks before execution, validation, review, and archive.

In this mode, Gentle-Orchestrator coordinates agent dispatch, while SDO assets define the domain protocol, phase boundaries, and artifacts.

### Standalone SDO-Orchestrator Mode

Use this mode when Gentle-Orchestrator is absent or the user chooses an independent SDO runtime.

The SDO pack should install:

- `sdo-orchestrator` as the root OpenCode agent;
- `/sdo-*` commands routed directly to `sdo-orchestrator`;
- SDO phase agents;
- SDO skills and shared protocol files;
- local artifact storage defaults;
- optional Engram or hybrid storage configuration.

In this mode, SDO-Orchestrator owns lifecycle state, dependency gates, preflight checks, command routing, phase result interpretation, and archive readiness.

## Shared SDO Protocol

The protocol must be identical in both runtime modes.

### Phase Lifecycle

```text
Intent / Init
→ Classification / Spec
→ Plan / Runbook
→ Approval Gate
→ Bounded Execution
→ Evidence Capture
→ Independent Validation
→ Review
→ Archive
```

Each phase should declare:

- input artifacts required;
- output artifacts produced or updated;
- actor role and autonomy level;
- allowed tools and denied tools;
- approval or policy requirement;
- evidence behavior;
- stop conditions;
- next eligible phases.

### Artifact Contract

Every operation should be tracked by an `SDO ID` and a manifest that links phase outputs.

Minimum file-based artifact graph:

```text
sdo/
  operations/
    SDO-YYYYMMDD-NNN/
      manifest.yaml
      operational-spec.md
      runbook.md
      approval-record.yaml
      execution-record.md
      validation-report.md
      evidence-package.yaml
      review-record.md
      archive-record.md
```

The manifest should include:

- `sdo_id`;
- title and current phase;
- profile and risk classification;
- autonomy level;
- target environment and scope;
- artifact paths or external references;
- approval record reference;
- execution policy reference;
- evidence package reference;
- lifecycle state and timestamps;
- deviations and unresolved blockers.

### Phase Result Envelope

Each command, skill, or agent should return a structured envelope so the orchestrator can make deterministic routing decisions.

```yaml
phase_result:
  sdo_id: "SDO-YYYYMMDD-NNN"
  phase: "spec | runbook | approval | execute | validate | evidence | review | archive"
  status: "success | blocked | needs-human | failed"
  summary: ""
  artifacts_created: []
  artifacts_updated: []
  evidence_refs: []
  approvals_required: []
  policy_decisions: []
  deviations: []
  next_allowed_phases: []
  stop_reason: ""
```

The envelope is the handoff contract. Agents may write rich artifacts, but routing should depend on this compact result.

### Autonomy and Policy Gates

The pack must enforce the existing SDO autonomy model.

Execution must be blocked when:

- autonomy level is missing;
- requested tool use exceeds the approved policy;
- approval is required and absent;
- target, environment, identity, or action boundary is ambiguous;
- stop conditions were reached;
- secrets or sensitive data appear unexpectedly;
- rollback or recovery is missing where the selected profile requires it.

### Evidence Behavior

Evidence collection should default to references and summaries.

Store:

- source system and immutable reference when available;
- command, API, SQL, pipeline, or AI tool-call summary;
- actor and executor identity;
- UTC timestamps;
- result, exit code, request ID, event ID, run ID, or object version;
- redaction status;
- hash or checksum for material artifacts.

Do not store secrets, unnecessary personal data, raw `.env` values, or full transcripts unless explicitly approved and redacted.

## Proposed Commands

Commands are user-facing OpenCode entry points. They should collect intent, resolve the target `SDO ID`, call the active orchestrator, and return the phase result envelope.

| Command | Purpose | Primary output |
|---|---|---|
| `/sdo-init` | Initialize SDO agent-pack assets, storage mode, and local configuration. | Pack config and preflight report. |
| `/sdo-new` | Create a new operation from intent and assign or confirm an `SDO ID`. | Manifest and draft spec. |
| `/sdo-plan` | Classify the operation and produce or update spec/runbook artifacts. | Operational Spec and runbook. |
| `/sdo-approval-check` | Verify approval requirements, autonomy level, tool policy, and blockers. | Approval gate result. |
| `/sdo-execute` | Run or guide approved bounded execution. | Execution record and evidence references. |
| `/sdo-validate` | Validate observed state against the validation contract. | Validation report. |
| `/sdo-review` | Produce post-operation review and learning notes. | Review record. |
| `/sdo-archive` | Finalize immutable references and close the operation. | Archive record. |
| `/sdo-continue` | Resume the next eligible phase from manifest state. | Phase result envelope. |

## Proposed Agents and Subagents

| Agent | Responsibility | Notes |
|---|---|---|
| `sdo-orchestrator` | Route commands, enforce dependencies, run preflight, interpret phase results. | Installed only in standalone mode. |
| `sdo-init` | Inspect OpenCode environment, install assets, verify storage and policy settings. | Shared by both modes. |
| `sdo-classify` | Classify operation type, risk, profile, autonomy ceiling, and required gates. | Should not execute tools beyond read-only context gathering. |
| `sdo-spec` | Draft or refine the Operational Spec as an agent contract. | Must surface missing invariants instead of guessing. |
| `sdo-runbook` | Produce execution steps, prechecks, rollback or recovery, and stop conditions. | Should bind each action to policy and evidence needs. |
| `sdo-approval` | Check approval records and policy eligibility. | Does not approve on behalf of humans. |
| `sdo-execute` | Execute or guide approved actions within the tool policy. | Must fail closed on boundary changes. |
| `sdo-validate` | Verify outcomes independently from executor self-report. | Should prefer objective checks and source references. |
| `sdo-evidence` | Build and update evidence package references. | Must redact and avoid raw secret-bearing output. |
| `sdo-review` | Summarize deviations, residual risk, learning, and follow-up work. | Should not rewrite execution history. |
| `sdo-archive` | Freeze final manifest state and archive references. | Requires completed review gate. |

## Skill Packaging

SDO skills should follow OpenCode/Gentle-style skill directories with `SKILL.md` as the entry point.

Initial skill categories:

- `sdo-shared-protocol` for lifecycle, gates, envelope, artifact contracts, and terminology;
- `sdo-spec-authoring` for Operational Spec drafting rules;
- `sdo-runbook-authoring` for plan, precheck, stop condition, and rollback patterns;
- `sdo-approval-gates` for human approval and autonomy policy checks;
- `sdo-execution-policy` for bounded tool execution;
- `sdo-evidence-packaging` for evidence references, hashes, redaction, and retention;
- `sdo-validation` for independent validation behavior;
- `sdo-review-archive` for review and archive closure.

The pack should include a shared resolver or registry pattern inspired by Gentle AI:

- keep a generated or maintained skill index;
- map command and phase names to required skills;
- expose dependency and preflight requirements in metadata;
- allow Gentle integration mode to register SDO skills without overwriting existing SDD skills;
- allow standalone mode to resolve SDO skills directly from the SDO pack.

## Artifact Store Modes

| Mode | Use case | Behavior |
|---|---|---|
| File-based default | Local docs, Git workflows, public repo examples, manual review. | Store operation artifacts under a configured `sdo/operations/<SDO ID>/` path. |
| Optional Engram | Agent memory, cross-session continuity, searchable decisions and discoveries. | Store summaries, decisions, discoveries, and session context; do not rely on Engram as the only audit artifact. |
| Optional hybrid | Teams that want durable files plus agent memory. | Files remain source of truth; Engram indexes important state, decisions, and next actions. |

The MVP should implement file-based storage first and treat Engram as an optional companion, not a required dependency.

## Preflight Checks

Before an operation can execute, the orchestrator should confirm:

- active runtime mode: `gentle-integration` or `standalone`;
- artifact store mode: `file`, `engram`, or `hybrid`;
- writable artifact path or configured external references;
- autonomy level and ceiling;
- approval policy and current approval state;
- execution policy and allowed tools;
- evidence retention mode and redaction expectations;
- review budget and review scope;
- target environment and identity boundaries;
- dependency state for the requested phase.

Preflight should return a phase result envelope with `blocked` or `needs-human` when required information is missing.

## Dependency Gates

The orchestrator must enforce phase order by artifact state, not by trust in chat history.

| Requested phase | Required before it can start |
|---|---|
| Execute | Spec, runbook, approval or policy eligibility, execution policy, rollback or recovery if required. |
| Validate | Execution record and evidence references. |
| Review | Validation report, evidence package, deviations list, residual risk notes. |
| Archive | Review record, final evidence package, unresolved blocker decision. |

`/sdo-continue` should inspect the manifest and choose only from `next_allowed_phases` emitted by the prior phase result.

## Gentle Integration Behavior

Installation in Gentle integration mode should:

1. Detect or confirm Gentle-Orchestrator availability.
2. Add SDO commands under the `/sdo-*` namespace.
3. Register SDO skills and subagents as additional assets.
4. Route command execution to Gentle-Orchestrator with SDO phase metadata.
5. Preserve existing SDD and Gentle configuration.
6. Write an SDO pack config that records adapter mode and asset paths.

The adapter should make SDO feel native inside Gentle while keeping the conceptual boundary clear: Gentle coordinates, SDO defines the operational protocol.

## Standalone Behavior

Installation in standalone mode should:

1. Install `sdo-orchestrator` as the root agent for SDO commands.
2. Install SDO phase agents and skills.
3. Configure the selected artifact store.
4. Generate or refresh the SDO skill registry.
5. Run preflight to confirm commands can resolve the orchestrator and artifact path.
6. Leave Gentle and SDD configuration untouched if present.

The standalone runtime should be minimal. It should route commands, enforce gates, and delegate phase work rather than becoming a large framework.

## Initial `agent-pack/` Asset Layout

Future implementation assets can start with this layout:

```text
agent-pack/
  README.md
  opencode/
    commands/
      sdo-init.md
      sdo-new.md
      sdo-plan.md
      sdo-approval-check.md
      sdo-execute.md
      sdo-validate.md
      sdo-review.md
      sdo-archive.md
      sdo-continue.md
    agents/
      sdo-orchestrator.md
      sdo-init.md
      sdo-classify.md
      sdo-spec.md
      sdo-runbook.md
      sdo-approval.md
      sdo-execute.md
      sdo-validate.md
      sdo-evidence.md
      sdo-review.md
      sdo-archive.md
    skills/
      sdo-shared-protocol/
        SKILL.md
      sdo-spec-authoring/
        SKILL.md
      sdo-runbook-authoring/
        SKILL.md
      sdo-approval-gates/
        SKILL.md
      sdo-execution-policy/
        SKILL.md
      sdo-evidence-packaging/
        SKILL.md
      sdo-validation/
        SKILL.md
      sdo-review-archive/
        SKILL.md
    shared/
      phase-result-envelope.yaml
      artifact-contract.yaml
      registry.yaml
      preflight-checklist.md
  installers/
    install-opencode-pack.md
```

The first implementation can be file-copy based. A richer installer can come later only if repeated local installation proves painful.

## Risks and Tradeoffs

| Risk or tradeoff | Mitigation |
|---|---|
| Two runtime modes can drift | Keep protocol files shared and test command behavior against the same phase envelope. |
| Gentle adapter may accidentally depend on Gentle internals | Treat Gentle as a routing host; keep SDO protocol, commands, and artifacts independent. |
| Standalone orchestrator could become too large | Keep it thin: preflight, dependency gates, routing, and envelope interpretation only. |
| Agent execution may bypass SDO controls | Fail closed when approval, autonomy, tool policy, or artifact dependencies are missing. |
| Evidence can become noisy or sensitive | Store references and summaries by default; require redaction and retention metadata. |
| File-based state can be edited manually | Use manifests, hashes where useful, and archive records for closure integrity. |
| Engram memory is useful but not audit-grade by itself | Treat Engram as optional continuity memory; keep durable audit artifacts in files or approved systems. |

## MVP Sequence

1. Define shared protocol assets: phase result envelope, artifact contract, preflight checklist, and dependency gates.
2. Create file-based artifact store conventions and manifest template.
3. Implement standalone `sdo-orchestrator` and the minimal `/sdo-init`, `/sdo-new`, `/sdo-plan`, `/sdo-approval-check`, and `/sdo-continue` commands.
4. Add read-only validation and evidence packaging commands before state-changing execution.
5. Add `/sdo-execute` only after approval, autonomy, and tool policy gates are enforceable.
6. Add Gentle integration adapter registration for `/sdo-*` commands, skills, and phase agents.
7. Add `/sdo-review` and `/sdo-archive` closure behavior.
8. Add optional Engram or hybrid indexing after file-based artifacts are stable.

## Related References

- [`../core/04-agentic-sdo.md`](../core/04-agentic-sdo.md)
- [`../reference/agent-roles.md`](../reference/agent-roles.md)
- [`../reference/autonomy-levels.md`](../reference/autonomy-levels.md)
- [`../reference/tool-execution-policy.md`](../reference/tool-execution-policy.md)
- [`../reference/human-approval-model.md`](../reference/human-approval-model.md)
- [`../guides/platform-tooling-guide.md`](../guides/platform-tooling-guide.md)
- [`../integrations/gentle-ai.md`](../integrations/gentle-ai.md)
