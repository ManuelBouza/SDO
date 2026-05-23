# Operation Taxonomy

SDO classifies operations by operational effect and contextual risk, not by tool.

```text
Base Class + Risk / Context Overlays = SDO Flow Profile
```

The same tool can produce very different risk depending on what it does. A `kubectl get pods` operation and a `kubectl delete pvc` operation are not equivalent because both use Kubernetes. The effect and context matter.

## Classification flow

Use this sequence:

1. Identify the base class.
2. Apply overlays.
3. Determine the dominant risk tier.
4. Select the SDO profile.
5. Derive approval, evidence, validation, and rollback requirements.

```text
Operation → Base Class → Overlays → Risk Tier → SDO Profile → Controls
```

## Base classes

| Class | Definition | Examples |
|---|---|---|
| Observe | Reads state, configuration, logs, metrics, or data without intended mutation. | service status, log review, API GET, SQL SELECT |
| Analyze | Produces a plan, diff, impact assessment, diagnosis, or preview without applying changes. | terraform plan, dry-run, kubectl diff, SQL explain |
| Maintain | Performs known, repeatable operational maintenance. | rotate logs, renew certificate, scheduled restart, cleanup |
| Configure | Changes configuration, parameters, policy, or persistent settings. | edit config, modify GPO, update cloud parameter, change feature flag |
| Deploy | Introduces, replaces, or reverts software, package, image, or artifact version. | app deploy, container update, release rollback |
| Control | Changes runtime behavior without necessarily changing the artifact. | start, stop, restart, scale, drain, failover, enable/disable |
| Data | Reads, transforms, migrates, restores, corrects, or deletes data. | migration, backup, restore, bulk update, export |
| Access | Changes identity, permissions, secrets, credentials, roles, or security posture. | create user, change IAM, rotate secret, open firewall, modify RBAC |
| Remediate | Corrective action to restore service, security, or integrity. | rollback, isolate node, clear queue, repair index, restore backup |
| Destroy | Removes resources, data, configurations, or capabilities. | delete VM, drop table, delete PVC, purge logs, decommission service |

## Overlay dimensions

Overlays modify rigor. They can elevate or, rarely, reduce the required profile.

| Overlay | Typical values | Effect |
|---|---|---|
| Environment | dev, test, staging, prod | Production raises rigor. |
| Sensitivity | none, internal, confidential, regulated, secret-bearing | Sensitive data increases approval and evidence control. |
| Privilege | user, operator, admin, root, break-glass | High privilege requires stronger traceability. |
| Scope | local, single-target, multi-target, fleet-wide, cross-system, external | Larger blast radius increases rigor. |
| Reversibility | R0–R7 | Harder rollback increases rigor. |
| Availability Impact | none, degraded, partial outage, full outage | Availability risk raises profile. |
| Data Impact | none, read-only, mutation, corruption-risk, loss-risk | Data mutation or exposure raises profile. |
| Security Impact | none, posture-change, exposure, privilege-escalation | Security changes require stronger approval. |
| Cost Impact | none, bounded, variable, runaway-risk | Unbounded cost raises rigor. |
| Compliance Impact | none, audit-relevant, regulated, legal-hold | Compliance increases evidence and retention. |
| Execution Mode | manual, assisted, semi-auto, automated, autonomous | More autonomy requires stronger guardrails. |
| Urgency | routine, scheduled, urgent, emergency | Emergency changes flow, not accountability. |
| State Model | imperative, declarative, pipeline-backed, GitOps | Changes preview/evidence mechanics. |
| Evidence Sensitivity | normal, redacted, restricted | Controls evidence storage and access. |

## Reversibility overlay

Use the R0–R7 model:

| Level | Meaning |
|---|---|
| R0 | No-change / preview only |
| R1 | Naturally reversible |
| R2 | Version reversible |
| R3 | Snapshot / backup reversible |
| R4 | Compensatable |
| R5 | Roll-forward only |
| R6 | Mitigation only |
| R7 | Irreversible / no-rollback |

General mapping:

- R0–R2 can often fit Lightweight or Normal.
- R3–R5 require formal controls.
- R6–R7 require explicit risk acceptance, mitigation, recovery, or emergency handling.

## Risk tier rule

SDO does not average risk away.

```text
Risk Tier = max(
  availability,
  data,
  security,
  cost,
  compliance,
  reversibility,
  scope,
  privilege,
  urgency
)
```

A technically simple operation can still be Critical if it touches secrets, production data, IAM, fleet-wide targets, or irreversible actions.

## Read-only classification

Read-only means no intended mutation. It does not mean low risk.

| Read-only type | Typical risk | Example |
|---|---:|---|
| Technical status without sensitive data | Low | `systemctl status nginx` |
| Operational metadata | Low/Medium | `kubectl describe pod` |
| Business data | Medium/High | `SELECT email FROM customers` |
| Secrets or credentials | High/Critical | `kubectl get secret -o yaml` |
| Bulk export | High/Critical | table dump, log export, CSV export |

Rule:

```text
Read-only classifies by exposure, not only by mutation.
```

## Profile selection guide

| Conditions | Likely profile |
|---|---|
| Local, no sensitive data, no mutation, R0–R1 | SDO-Lightweight |
| Single target, reversible, limited production impact | SDO-Normal |
| Production-critical, sensitive, privileged, data-changing, multi-target, hard rollback | SDO-Critical |
| Active incident or imminent harm | SDO-Emergency |
| Mature CI/CD, GitOps, IaC, or automation with plans/artifacts | SDO-Pipeline-Backed |

## Examples

| Operation | Base Class | Overlays | Profile |
|---|---|---|---|
| `systemctl status nginx` | Observe | single-target, no-sensitive | SDO-Lightweight |
| `journalctl -u app` with possible tokens | Observe | sensitive logs | SDO-Normal |
| Restart one production service | Control | prod, availability, R1/R2 | SDO-Normal |
| Patch 50 production servers | Maintain / Configure | fleet-wide, prod | SDO-Critical |
| `SELECT count(*) FROM orders` | Observe / Data | aggregate, no-sensitive | SDO-Lightweight |
| Export customer table | Data | PII, bulk export | SDO-Critical |
| Add user to Domain Admins | Access | privileged, security | SDO-Critical |
| `terraform apply` with resource destruction | Configure / Destroy | pipeline-backed, destructive | SDO-Critical or Pipeline-Backed with Critical controls |
| Emergency host isolation | Remediate | emergency, security | SDO-Emergency |

## Core rules

- Read-only does not always mean low risk.
- The most severe overlay dominates.
- Emergency accelerates approval; it does not reduce accountability.
- Pipeline-backed does not mean low risk.
- Rollback must be a verifiable capability, not decorative text.
- Automation reduces variance but can amplify blast radius.
- Scope multiplies risk.
- No-rollback is a risk decision, not a missing field.

## Anti-patterns

| Anti-pattern | Why it is wrong |
|---|---|
| Classifying by tool | Bash, SQL, kubectl, and Terraform can all be low or critical depending on effect. |
| Treating all read-only as safe | Read-only can expose secrets or regulated data. |
| Treating automation as safety | Automation can execute mistakes faster and wider. |
| Using emergency to skip responsibility | Emergency requires more post-evidence, not less. |
| Ignoring blast radius | A safe single-target action can be critical fleet-wide. |
| Hiding data changes inside deploys | Data operations need explicit classification. |
| Calling backup a rollback | Restore capability must be verified or risk-accepted. |
