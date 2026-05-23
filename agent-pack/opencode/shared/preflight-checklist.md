# SDO OpenCode Preflight Checklist

The orchestrator runs this checklist before any SDO phase. It runs the execution section again immediately before any tool call or state-changing step.

Preflight fails closed: if a required answer is missing, ambiguous, stale, or only present in chat history, return a `blocked` or `needs-human` phase result envelope.

## Universal Phase Preflight

- [ ] Runtime mode is explicit: `gentle-integration` or `standalone`.
- [ ] Active orchestrator is explicit: Gentle-Orchestrator adapter, SDO-Orchestrator, or human/manual path.
- [ ] Artifact store mode is explicit: `file`, `engram`, or `hybrid`.
- [ ] File-based operation path is known or can be created under `sdo/operations/<SDO ID>/`.
- [ ] `SDO ID` is assigned or the requested phase is allowed to create one.
- [ ] Manifest exists for existing operations and matches the requested `SDO ID`.
- [ ] Requested phase is allowed by manifest state and dependency gates.
- [ ] Operation profile, base class, risk tier, and environment are declared.
- [ ] Autonomy level and approved autonomy ceiling are declared.
- [ ] Target scope and exclusions are explicit enough to prevent target drift.
- [ ] Actor identities are declared for requester, owner, agent, executor, validator, and approver where applicable.
- [ ] Approval state is known and linked to a file or approved external reference.
- [ ] Tool policy is known for any planned diagnostic, validation, or execution action.
- [ ] Evidence expectations, sensitivity, redaction, retention, and access classification are declared.
- [ ] Validation contract exists or the current phase is explicitly responsible for creating it.
- [ ] Rollback or recovery expectations are declared for state-changing work.
- [ ] Dependency state is read from artifacts or manifest fields, not from chat history.
- [ ] Review scope and closure expectations are known for the selected profile.

## Execution Preflight

- [ ] Requested action maps to an approved planned action or approved diagnostic permission.
- [ ] Tool, command, API, SQL, pipeline, GUI path, or AI tool call is allowlisted.
- [ ] Denied tools and denied parameters are known.
- [ ] Environment, account, cluster, namespace, workspace, database, region, or resource selector is unambiguous.
- [ ] Executor identity matches the approved boundary and least-privilege expectation.
- [ ] Execution window is currently valid or explicitly accepted as emergency-approved.
- [ ] Preview, dry-run, diff, or compensating control is complete where required.
- [ ] Stop conditions and monitoring signals are visible before execution begins.
- [ ] Evidence capture will record actor, executor, target, command or action summary, timestamps, result, and redaction status.
- [ ] Secrets and sensitive data are not included in prompts, command text, screenshots, logs, or evidence content.
- [ ] Rollback or recovery path is current, reachable, and validated enough for the profile.
- [ ] Point of no return is known and has an approval or risk acceptance path.

## Fail-Closed Rules

- Missing manifest for an existing operation blocks every phase except recovery-oriented `continue` that can safely locate or recreate state.
- Missing target, environment, identity, autonomy, approval, policy, evidence, validation, or rollback information blocks execution.
- Chat history, agent memory, or unstored conversation context cannot satisfy a gate by itself.
- Any scope, target, identity, privilege, timing, or tool boundary change returns to `approval-check`.
- Any unapproved state-changing action at A5 blocks execution.
- Any A6 action without a formal preapproved policy blocks execution.
- Any unexpected secret or sensitive output stops the phase until redaction and handling are resolved.
- Any inconclusive validation blocks archive unless explicitly accepted as residual risk by the required authority.
