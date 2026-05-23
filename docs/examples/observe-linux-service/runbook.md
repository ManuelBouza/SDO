# SDO Runbook: Observe Linux service health

## Metadata

- Runbook ID: RB-SDO-20260523-001
- Spec ID: SDO-20260523-001
- Version: 0.1.0
- Owner: ops-team
- Execution mode: CLI
- Environment: staging
- Flow profile: SDO-Lightweight
- Approved change/ticket: N/A

## Safety

- Approval required: no formal approval
- Approval reference: N/A
- Backup required: no
- Backup evidence: N/A
- Stop conditions: target host mismatch; command differs from approved read-only commands; logs show secrets that cannot be safely redacted
- Point of no return: none
- Rollback reference: N/A
- Evidence location: `evidence-package.yaml`

## Parameters

| Name | Required | Sensitive | Description | Validation |
|---|---:|---:|---|---|
| target_host | yes | no | Linux host to observe | Must equal `staging-web-01` |
| service_name | yes | no | systemd service | Must equal `nginx.service` |
| log_lines | yes | no | Recent log lines to inspect | Max 100 |

## Pre-checks

| ID | Action | Expected result | Evidence |
|---|---|---|---|
| PRE-001 | Confirm target host identity | Host is `staging-web-01` | command summary |
| PRE-002 | Confirm approved command list | Commands are read-only | operator note |

## Execution Steps

| Step | Planned Action ID | Actor | Executor | Action/Command | Expected result | Validation | Evidence | Stop condition | Rollback hook |
|---|---|---|---|---|---|---|---|---|---|
| 1 | PA-001 | human | shell_command | `hostnamectl --static` | returns `staging-web-01` | target matches spec | redacted output ref | host mismatch | N/A |
| 2 | PA-002 | human | shell_command | `systemctl is-active nginx.service` | returns `active` | active state observed | output ref | command fails unexpectedly | N/A |
| 3 | PA-003 | human | shell_command | `journalctl -u nginx.service -n 100 --no-pager` | recent logs available | no obvious critical errors after redaction | non-sensitive or redacted excerpt ref | secrets cannot be redacted | N/A |

## Post-checks

| ID | Action | Expected result | Evidence |
|---|---|---|---|
| POST-001 | Confirm no state-changing action occurred | no mutation recorded | execution record |
| POST-002 | Confirm evidence package updated | evidence closure passes | evidence package |

## Rollback

### Trigger conditions

Not applicable; operation is read-only.

### Steps

Not applicable.

### Validation after rollback

Not applicable.

## Deviations

How to record deviations: add a row to `execution-record.md` and stop if deviation changes risk.

Who can approve deviations: platform-team owner.

## Evidence Collection

Required evidence:

- target host confirmation;
- active service result;
- non-sensitive or redacted log excerpt;
- validation result.

Storage location: `evidence-package.yaml` and local redacted evidence references.

Sensitive data handling: collect only non-sensitive staging logs; redact tokens, cookies, authorization headers, IPs if required, and secrets from minimal excerpts.

## Closeout

Success condition: service is active and evidence closure passes.

Failure condition: service inactive, command failure, unsafe log content, or target mismatch.

Review required: minimal closure note.
