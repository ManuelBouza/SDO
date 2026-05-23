# Contributing to SDO

SDO is a technology-agnostic methodology for designing, approving, executing, validating, and auditing IT operations through operational specifications.

Documentation artifacts in this repository are written in English. SDO is agentic-first and manual-compatible: AI agents should be able to consume the artifacts under explicit policy, while human operators can still use them without AI.

## Contribution Guidelines

- Keep changes proportional to the concept being changed.
- Update examples, templates, and references when changing core SDO concepts.
- Avoid tool lock-in; integrations should remain optional unless explicitly scoped as normative.
- Keep AI governance explicit, including approval, autonomy, evidence, and rollback expectations.
- Treat research notes in [`docs/research/`](docs/research/) as non-normative context, not binding methodology.

## Review Expectations

Contributions should be easy to review, trace to the affected SDO concept, and avoid mixing unrelated methodology, examples, and tooling changes. Reviewers should check that changes preserve technology neutrality, manual compatibility, and explicit governance boundaries.
