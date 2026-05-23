# SDO Principles

These principles define how SDO should be applied. They are normative: if a process contradicts them, it may still be an operational process, but it is not following SDO.

## 1. Spec-first, not command-first

Operational work starts from a clear intent, scope, target state, limits, and validation criteria.

Commands, scripts, clicks, API calls, SQL statements, or pipeline jobs are implementation details. They should appear after the operation is specified, classified, and bounded.

Minimum requirement:

```text
Before execution, the operator can answer:
- What are we trying to achieve?
- Where does it apply?
- What is out of scope?
- What state is expected after the operation?
- How will we know it worked?
```

Anti-pattern: starting with “run this command” without intent, scope, validation, and recovery context.

## 2. Evidence-first closure

An operation is not complete until its result is evidenced and traceable.

Evidence does not need to mean saving every log. It means preserving enough protected, verifiable evidence for someone other than the operator to understand what was approved, what was executed, what changed, what was validated, and how the operation closed.

Minimum requirement:

```text
Every operation closes with:
- execution record;
- validation result;
- evidence references;
- deviations, if any;
- final decision.
```

Anti-pattern: closing because the command returned exit code 0.

## 3. Risk-adaptive rigor

Control increases with impact, irreversibility, privilege, sensitive data, production scope, automation, and uncertainty — not with organizational habit.

Low-risk work should not be buried under unnecessary process. Critical work should not be treated as routine just because it is familiar.

Minimum requirement:

```text
Every operation has:
- base class;
- overlays;
- selected SDO profile;
- approval level proportional to risk.
```

Anti-pattern: a universal CAB for everything, or auto-approval for destructive production work.

## 4. Tool-agnostic execution

Commands, APIs, pipelines, GUIs, SQL scripts, GitOps controllers, ticket workflows, and AI agents are execution backends, not the methodology.

SDO defines operational intent, controls, validation, evidence, and recovery. Execution tools implement those decisions.

Minimum requirement:

```text
Every action identifies:
- planned action;
- executor;
- target;
- expected result;
- evidence;
- rollback or recovery mapping.
```

Anti-pattern: designing SDO as “YAML for shell commands”.

## 5. Human accountability

AI can assist, propose, summarize, classify, draft, review, and sometimes execute under policy, but it is not accountable for operational risk.

SDO separates proposing, approving, executing, and validating.

```text
AI proposes ≠ Human approves ≠ Executor performs ≠ SDO validates
```

Minimum requirement:

```text
For AI-assisted operations, record:
- AI role;
- autonomy level;
- approved scope;
- tool calls;
- human approval where required;
- evidence of outcome.
```

Anti-pattern: “the model decided” as an operational justification.

## 6. Reversible by design

Every state-changing operation declares a reversibility level and a safe-state strategy.

Rollback in SDO means safe-state recovery capability, not merely undo. Some operations restore a previous version, some roll forward, some fail over, some compensate, some mitigate, and some require explicit no-rollback acceptance.

Minimum requirement:

```text
Every state-changing operation declares:
- reversibility level;
- rollback/recovery strategy;
- trigger conditions;
- point of no return, if any;
- validation after recovery;
- risk acceptance if rollback is unavailable.
```

Anti-pattern: writing “restore backup” without proving the backup is restorable.

## 7. Bounded agentic execution

Agents may execute only inside an explicit operational contract.

The contract must define target, environment, allowed tools, autonomy level, identity, approval boundary, validation, evidence, rollback or recovery, and stop conditions.

Minimum requirement:

```text
An agent can answer before any tool call:
- What am I allowed to do?
- Where am I allowed to do it?
- Who approved it?
- When must I stop?
- What evidence must I leave?
```

Anti-pattern: letting an agent infer scope, tools, credentials, or approval from a prompt.

## 8. Planned vs actual separation

SDO always distinguishes what was approved from what actually happened.

The plan may be correct and the execution may still deviate. The execution may succeed while validation fails. The evidence may reveal impact not anticipated in the spec.

Minimum requirement:

```text
Every executed action maps to:
- an approved planned action; or
- a classified deviation.
```

Anti-pattern: editing the plan after execution so it looks like nothing deviated.

## 9. Smallest useful artifact

SDO should produce the smallest set of artifacts that safely governs the operation.

The methodology is not an excuse for documentation theater. A low-risk read-only action may need a short spec and minimal evidence. A critical data migration needs full controls.

Minimum requirement:

```text
Artifact detail matches the selected SDO profile.
```

Anti-pattern: using the same heavyweight template for every operation.

## 10. Integrate, do not duplicate

SDO should reference existing systems of record instead of duplicating them.

Tickets, PRs, pipelines, monitoring systems, audit logs, cloud logs, SIEMs, and ITSM tools can all be evidence or workflow carriers.

Minimum requirement:

```text
Use stable references:
- ticket IDs;
- PR URLs;
- pipeline run IDs;
- audit event IDs;
- evidence storage references;
- hashes.
```

Anti-pattern: copying large logs, tickets, or pipeline output into SDO documents without need.

## 11. Standalone core, optional integrations

SDO must work without any specific tool or agent ecosystem.

Gentle AI, OpenSpec-style workflows, Git, ITSM, CI/CD, and CLI agents can improve SDO adoption, but they are integrations. They do not define SDO Core.

Minimum requirement:

```text
An SDO operation can be represented with plain documents and references.
```

Anti-pattern: making SDO unusable without one specific platform.
