> Research note — non-normative.
> This document is background material and may be superseded by current SDO core/reference documentation.
> Current normative docs start at [../start-here.md](../start-here.md).

## 1. Conclusión de diseño

Mi propuesta es que SDO tenga un **modelo de ejecución por contrato**, no por tecnología.

Es decir:

> **Operational Spec → Execution Plan → Planned Actions → Executor Binding → Actual Execution → Evidence → Validation → Review/Rollback**

El punto clave: **SDO no ejecuta “bash”, “PowerShell”, “SQL” o “kubectl” directamente como conceptos metodológicos**. SDO define una acción operacional abstracta y la vincula a un **executor** que declara sus capacidades: preview, dry-run, diff, rollback, logs, exit code, output, identidad, permisos, timeout, retry, concurrencia y evidencia.

Esto encaja con prácticas ya existentes: Terraform separa `plan` y `apply`; Ansible tiene check/diff mode; Kubernetes soporta `kubectl diff`, dry-run y rollout undo; Azure ARM tiene `what-if`; CloudFormation tiene change sets; PowerShell usa `-WhatIf`/`-Confirm`; SQL tiene transacciones, rollback, backups/restores; GitOps usa desired state versionado y reconciliación automática. ([HashiCorp Developer][1])

---

## 2. Prácticas aceptadas vs propuesta SDO

**Prácticas aceptadas observadas:**

* Separar preview/plan de apply/execute cuando la tecnología lo permite: Terraform, Pulumi, Azure ARM, CloudFormation, Kubernetes, Ansible. ([HashiCorp Developer][1])
* Usar idempotencia, desired state o guard conditions para reducir riesgo de repetición: HTTP define métodos idempotentes; AWS EC2 usa client tokens; Puppet, Chef, Salt y Ansible se apoyan en idempotencia o condiciones de estado. ([RFC Editor][2])
* Capturar logs, outputs, códigos de salida, eventos y artefactos: `journalctl`, pipelines, runbook platforms y APIs ya producen registros de ejecución o respuestas estructuradas. ([Man7][3])
* Incluir aprobaciones humanas en operaciones sensibles: GitHub Actions environments, GitLab deployment approvals, Jenkins input step, AWS SSM `aws:approve` y flujos HITL de agentes. ([GitHub Docs][4])

**Propuesta propia para SDO:**

* Normalizar todo como `sdo_execution_step`.
* Separar siempre `planned_action` de `executed_action`.
* Exigir `executor_capabilities`.
* Convertir preview/dry-run/diff en una capacidad opcional, no obligatoria.
* Tratar acciones manuales, GUI e IA como ejecutores de primera clase.
* Registrar desviaciones como evento formal de control, no como simple comentario.
* Hacer que rollback sea un **plan de recuperación ejecutable o verificable**, no solo una frase.

---

## 3. Tipos de executor que debería reconocer SDO

| Executor SDO                | Uso típico                                                    |
| --------------------------- | ------------------------------------------------------------- |
| `shell_command`             | Bash, sh, zsh, comandos Linux, systemctl, package managers    |
| `powershell_command`        | PowerShell, servicios Windows, registry, GPO, scheduled tasks |
| `sql_executor`              | DDL, DML, migraciones, backups, restores                      |
| `api_executor`              | REST, GraphQL, gRPC, webhooks                                 |
| `kubernetes_executor`       | kubectl, Helm, controllers, apply/diff/rollout                |
| `cloud_cli_executor`        | AWS CLI, Azure CLI, gcloud                                    |
| `iac_executor`              | Terraform/OpenTofu, Pulumi, Ansible, Chef, Puppet, Salt       |
| `gitops_executor`           | Argo CD, Flux, pull-based reconciliation                      |
| `pipeline_executor`         | GitHub Actions, GitLab CI/CD, Jenkins, Azure DevOps           |
| `runbook_platform_executor` | Rundeck, StackStorm, AWX, SSM Automation, Azure Automation    |
| `manual_executor`           | Checklist, GUI, consola web, validación visual                |
| `ai_assisted_executor`      | IA redacta/propone/ejecuta bajo límites y aprobación          |

---

## 4. Interfaz mínima común de todo executor

Todo executor debería declarar este contrato mínimo:

```yaml
sdo_executor_interface:
  identity:
    executor_type: string
    executor_name: string
    executor_version: string
    runtime_environment: string

  capabilities:
    supports_preview: boolean
    preview_modes: [dry_run, diff, plan, validate, simulate]
    supports_execute: boolean
    supports_rollback: boolean
    supports_native_idempotency: boolean
    supports_retries: boolean
    supports_timeout: boolean
    supports_concurrency_control: boolean
    supports_structured_output: boolean
    supports_artifact_capture: boolean
    supports_interactive_steps: boolean
    supports_human_approval_pause: boolean

  methods:
    prepare: required
    validate_inputs: required
    render_planned_action: required
    preview: optional
    execute: required
    monitor: optional
    collect_evidence: required
    validate_result: required
    rollback_or_recover: optional
```

**Regla SDO:** si el executor no soporta una capacidad, no se simula que existe. Se marca explícitamente como `unsupported`, y el perfil SDO debe compensarlo con controles adicionales: aprobación, backup, snapshot, ventana de mantenimiento, ejecución manual supervisada o rollback externo.

---

## 5. Estructura genérica `sdo_execution_step`

```yaml
sdo_execution_step:
  step_id: "STEP-001"
  sdo_id: "SDO-2026-0001"
  title: "Restart application service"
  description: "Restart service and validate health endpoint"

  phase: "Execution"
  base_class: "Maintain"
  flow_profile: "SDO-Normal"

  executor:
    type: "shell_command"
    name: "bash"
    version: "5.x"
    host_runtime: "linux"
    capability_profile:
      preview: "limited"
      rollback: "custom"
      structured_output: false

  target:
    scope: "single"
    target_id: "app-server-01"
    environment: "production"
    selector: null

  planned_action:
    action_type: "command"
    command: "sudo systemctl restart app.service"
    working_directory: null
    script_ref: null
    api_ref: null
    sql_ref: null
    pipeline_ref: null
    manual_instruction: null
    immutable_hash: "sha256:<hash-of-approved-action>"

  parameters:
    inputs:
      service_name: "app.service"
    secrets:
      - name: "sudo_credential"
        source: "external_secret_store"
        exposure_policy: "never_log"
    variables:
      maintenance_window: "2026-05-22T22:00:00Z"

  identity_and_permissions:
    run_as: "operator"
    privilege_level: "sudo-limited"
    required_permissions:
      - "restart app.service"
      - "read journal logs"
    approval_required: true

  execution_controls:
    idempotency:
      expected: "safe_if_repeated"
      strategy: "state_check_before_execute"
      idempotency_key: null
    retry:
      enabled: false
      max_attempts: 1
      backoff: null
    timeout_seconds: 120
    concurrency:
      mode: "serial"
      max_parallel: 1
    rate_limit: null

  preview:
    required: true
    mode: "validate"
    command: "systemctl status app.service"
    expected_preview_output: "service exists and is manageable"
    preview_unsupported_reason: null

  validation:
    success_criteria:
      - "service active"
      - "health endpoint returns 200"
    validation_commands:
      - "systemctl is-active app.service"
      - "curl -fsS http://localhost:8080/health"

  rollback:
    rollback_type: "compensating_action"
    rollback_action:
      command: "sudo systemctl restart app.service && sudo journalctl -u app.service -n 100"
    recovery_notes: "If restart fails, restore previous package/config snapshot if available."

  evidence_requirements:
    capture_stdout: true
    capture_stderr: true
    capture_exit_code: true
    capture_logs:
      - "journalctl -u app.service --since '<execution_start>'"
    capture_artifacts: []
    screenshots_required: false
    operator_notes_required: true
```

---

## 6. Estructura genérica `sdo_execution_record`

```yaml
sdo_execution_record:
  execution_id: "EXEC-2026-0001"
  sdo_id: "SDO-2026-0001"
  run_id: "RUN-001"

  approved_plan:
    plan_id: "PLAN-001"
    approval_id: "APR-001"
    approved_by: "change-owner"
    approved_at: "2026-05-22T20:30:00Z"
    approved_action_hash: "sha256:<hash>"

  actual_execution:
    started_at: "2026-05-22T22:01:00Z"
    finished_at: "2026-05-22T22:02:10Z"
    executed_by:
      actor_type: "human"
      actor_id: "operator-01"
      effective_identity: "svc-sdo-runner"
    executor:
      type: "shell_command"
      name: "bash"
      version: "5.x"
      runtime_host: "app-server-01"

  step_results:
    - step_id: "STEP-001"
      planned_action_hash: "sha256:<approved>"
      executed_action_hash: "sha256:<actual>"
      deviation_status: "none"
      status: "succeeded"
      exit_code: 0
      stdout_ref: "evidence/stdout-step-001.log"
      stderr_ref: "evidence/stderr-step-001.log"
      structured_response_ref: null
      artifacts:
        - "evidence/journal-app-service.log"
      started_at: "2026-05-22T22:01:10Z"
      finished_at: "2026-05-22T22:01:50Z"

  validation_results:
    status: "passed"
    checks:
      - name: "service_active"
        result: "passed"
      - name: "health_endpoint"
        result: "passed"

  rollback_execution:
    triggered: false
    reason: null
    rollback_record_ref: null

  evidence_package:
    package_id: "EVD-2026-0001"
    location: "sdo/evidence/SDO-2026-0001/"
    integrity_hash: "sha256:<evidence-package-hash>"

  closure:
    final_status: "completed"
    closed_by: "operator-01"
    closed_at: "2026-05-22T22:10:00Z"
    review_required: true
```

---

## 7. Matriz executor → preview → evidencia → rollback → riesgos

| Executor           |                                  Preview support | Evidencia típica                                              | Rollback support                                    | Riesgos principales                                                  |
| ------------------ | -----------------------------------------------: | ------------------------------------------------------------- | --------------------------------------------------- | -------------------------------------------------------------------- |
| Shell/Linux        |                                         Limitado | stdout, stderr, exit code, `journalctl`, archivos modificados | Manual/custom                                       | Comandos destructivos, diferencias entre distros, paquetes, permisos |
| PowerShell/Windows |            Medio si soporta `-WhatIf`/`-Confirm` | transcript, event logs, service state, registry export        | Manual/custom, restore point, backup                | Registry, GPO, servicios críticos, ejecución remota                  |
| SQL                | Medio/alto si hay transacciones o migration tool | script, affected rows, query output, backup ID                | Transaction rollback, restore, snapshot             | DDL no siempre reversible, locks, pérdida de datos                   |
| REST/API           |                                         Variable | request/response, status code, correlation ID                 | Compensating API call                               | Reintentos no idempotentes, rate limits, efectos parciales           |
| GraphQL/gRPC       |                                         Variable | response body, errors, status, deadline, trace ID             | Compensating call                                   | Partial success, deadline exceeded, retry mal diseñado               |
| Kubernetes         |                                             Alto | diff, dry-run, rollout status, events, manifests              | rollout undo, re-apply previous manifest            | drift, admission webhooks, RBAC, namespace equivocado                |
| Cloud CLI          |                        Medio/alto según servicio | CLI output, request ID, audit logs, change set                | Depende del servicio                                | IAM excesivo, región/cuenta equivocada, costes                       |
| IaC                |                 Alto en Terraform/Pulumi/Ansible | plan, diff, state, logs                                       | re-apply previous state/config, destroy con cuidado | state drift, destroy accidental, provider bugs                       |
| GitOps             |                         Alto a nivel declarativo | commit, PR, sync status, drift report                         | revert commit                                       | reconciliación automática de cambios no deseados                     |
| CI/CD              |                 Alto si hay stages/protected env | build logs, artifacts, approvals                              | redeploy previous artifact                          | secretos, permisos, job mal parametrizado                            |
| Runbook platform   |                                       Medio/alto | job log, node log, parameters, approvals                      | job rollback o compensating job                     | jobs reutilizados fuera de contexto                                  |
| Manual/GUI         |                                             Bajo | screenshots, checklist, notas, sign-off                       | Manual                                              | error humano, evidencia débil, pasos ambiguos                        |
| AI-assisted        |                                         Variable | prompt, plan, approval, tool calls, trace                     | Humano o compensating action                        | excessive agency, prompt injection, ejecución fuera de intención     |

Sobre la base documental: PowerShell `SupportsShouldProcess` añade `WhatIf`/`Confirm`; Kubernetes soporta diff y dry-run en flujos declarativos; Terraform ejecuta acciones propuestas por un plan; AWS EC2 y muchas APIs usan tokens de idempotencia; gRPC documenta deadlines, retries y status codes; OpenAI documenta flujos HITL para pausar tool calls sensibles. ([Microsoft Learn][5])

---

## 8. Reglas SDO para planned vs actual execution

### Regla 1: lo aprobado debe ser inmutable

Toda acción aprobada debe tener:

```yaml
approved_action:
  normalized_representation: "<canonical action>"
  immutable_hash: "sha256:<hash>"
  approval_id: "APR-001"
```

### Regla 2: lo ejecutado se registra aparte

Nunca se sobrescribe la acción aprobada. Se registra:

```yaml
executed_action:
  raw_command_or_call: "<actual action>"
  normalized_representation: "<canonical actual action>"
  immutable_hash: "sha256:<actual-hash>"
```

### Regla 3: desviaciones

```yaml
deviation:
  status: "none | minor | major | emergency"
  detected_by: "executor | operator | reviewer"
  reason: "Actual command differs from approved command"
  requires_reapproval: true
```

### Regla 4: qué exige re-aprobación

Requiere re-aprobación si cambia cualquiera de estos elementos:

* target
* comando/script/API body/SQL
* identidad efectiva
* permisos
* secreto usado
* región/cuenta/cluster/base de datos
* rollback plan
* nivel de riesgo
* concurrencia o batch size

### Regla 5: excepción de emergencia

En `SDO-Emergency`, puede ejecutarse una desviación antes de re-aprobar, pero debe quedar marcada como:

```yaml
deviation:
  status: "emergency"
  post_approval_required: true
  review_required: true
```

---

## 9. Reglas para executors sin preview

Cuando `supports_preview: false`, SDO no debe inventar un dry-run. Debe usar controles compensatorios:

```yaml
preview:
  supported: false
  compensation_controls:
    - "peer review"
    - "backup or snapshot before execution"
    - "smaller batch"
    - "manual checkpoint"
    - "pre-state capture"
    - "explicit rollback/recovery plan"
```

Ejemplos:

* GUI administrativa sin modo preview.
* Script legacy no idempotente.
* SQL DDL no transaccional.
* API POST sin idempotency key.
* Operación manual sobre consola cloud.

---

## 10. Reglas para manual/GUI execution

Las operaciones GUI deben modelarse como pasos ejecutables, no como notas sueltas.

```yaml
manual_gui_step:
  step_id: "STEP-GUI-001"
  executor:
    type: "manual_executor"
  instruction:
    system: "Cloud console"
    path: "EC2 > Instances > Select instance > Reboot"
    expected_screen_state: "Instance state changes to rebooting"
  required_evidence:
    - "screenshot_before"
    - "screenshot_after"
    - "operator_note"
    - "timestamp"
    - "sign_off"
  controls:
    two_person_rule: true
    copy_paste_allowed: false
    deviation_requires_note: true
```

Reglas:

1. Cada paso GUI debe tener ruta, objetivo, resultado esperado y evidencia.
2. Capturar estado antes/después.
3. Prohibir “hacer lo necesario” como instrucción.
4. Registrar operador real, hora y pantalla o evidencia equivalente.
5. Si el operador cambia el procedimiento, se registra como desviación.
6. En perfiles Critical/Emergency, exigir segundo operador o aprobador.

---

## 11. Reglas para AI-assisted execution

SDO debería permitir IA, pero con límites fuertes. La IA puede actuar como:

| Rol IA                  | Permitido                                          |
| ----------------------- | -------------------------------------------------- |
| `draft_only`            | Redacta spec, plan, comandos, SQL, API body        |
| `advisor`               | Sugiere riesgos, validaciones, rollback            |
| `operator_assist`       | Guía al humano paso a paso                         |
| `tool_executor_guarded` | Ejecuta herramientas con aprobación humana         |
| `autonomous_executor`   | Solo permitido en bajo riesgo y entornos limitados |

Reglas mínimas:

```yaml
ai_execution_policy:
  autonomy_level: "draft_only | approve_each_action | bounded_autonomy | autonomous"
  allowed_tools:
    - "read_logs"
    - "generate_plan"
    - "execute_command"
  forbidden_tools:
    - "delete_resource"
    - "modify_iam"
    - "rotate_secrets"
  approval_checkpoints:
    before_tool_call: true
    before_state_change: true
    before_privileged_action: true
  guardrails:
    prompt_injection_detection: true
    output_validation: true
    secret_redaction: true
    max_actions_per_run: 3
```

Esto está alineado con patrones actuales de HITL: OpenAI documenta aprobaciones para pausar llamadas a herramientas sensibles, OWASP advierte del riesgo de “excessive agency” en LLMs, y NIST/Microsoft recomiendan gobernanza, accountability y supervisión humana en sistemas de IA. ([OpenAI Developers][6])

---

## 12. Reglas para multi-target, fleet execution y concurrencia

```yaml
multi_target_execution:
  target_scope: "fleet"
  selector:
    type: "label_query"
    expression: "role=web AND environment=prod"
  batch_strategy:
    mode: "rolling"
    batch_size: 5
    max_parallel: 2
    pause_between_batches_seconds: 60
  stop_conditions:
    max_failed_targets: 1
    error_rate_threshold_percent: 10
    validation_failure_action: "stop_and_review"
  rate_limits:
    requests_per_second: 5
    burst: 10
  per_target_record_required: true
```

Reglas:

1. Nunca ejecutar fleet como si fuera single target.
2. Todo batch debe tener criterio de parada.
3. Cada target produce su propio `step_result`.
4. Fallos parciales no son éxito global.
5. El rollback puede ser por target, por batch o global.
6. En SDO-Critical: serial o rolling obligatorio.
7. En SDO-Emergency: se permite paralelismo mayor, pero con evidencia posterior obligatoria.

---

## 13. Mapeo a perfiles SDO

| Perfil                | Modelo de ejecución recomendado                                                          |
| --------------------- | ---------------------------------------------------------------------------------------- |
| `SDO-Lightweight`     | 1 executor, pocos pasos, preview opcional, evidencia mínima                              |
| `SDO-Normal`          | preview si existe, aprobación simple, validation obligatoria                             |
| `SDO-Critical`        | plan congelado, aprobación formal, rollback probado o recovery claro, evidencia completa |
| `SDO-Emergency`       | ejecución rápida, controles mínimos antes, evidencia y review obligatorios después       |
| `SDO-Pipeline-Backed` | ejecución mediante pipeline, artifacts, protected environments, approvals y logs         |

---

## 14. Ejemplos agnósticos

### Bash / Linux

```yaml
executor:
  type: "shell_command"

planned_action:
  action_type: "command"
  command: "sudo systemctl restart nginx"

preview:
  mode: "validate"
  command: "systemctl status nginx"

validation:
  commands:
    - "systemctl is-active nginx"
    - "curl -fsS http://localhost/health"

evidence:
  logs:
    - "journalctl -u nginx --since '<execution_start>'"
```

`systemd` modela servicios con acciones como `ExecStart`/`ExecStop`, y `journalctl` permite consultar logs del journal, por eso son fuentes naturales de ejecución y evidencia en Linux. ([Freedesktop][7])

### PowerShell / Windows

```yaml
executor:
  type: "powershell_command"

planned_action:
  action_type: "command"
  command: "Restart-Service -Name Spooler"

preview:
  mode: "whatif"
  command: "Restart-Service -Name Spooler -WhatIf"

validation:
  commands:
    - "Get-Service -Name Spooler"
```

PowerShell administra servicios con cmdlets como `Get-Service`, `Start-Service`, `Stop-Service` y `Set-Service`; para cambios seguros, los cmdlets o funciones avanzadas pueden exponer `-WhatIf` y `-Confirm`. ([Microsoft Learn][8])

### SQL

```yaml
executor:
  type: "sql_executor"

planned_action:
  action_type: "sql_script"
  sql_ref: "migrations/20260522_add_index.sql"

execution_controls:
  transaction:
    enabled: true
    isolation_level: "read_committed"

rollback:
  rollback_type: "transaction_or_restore"
  rollback_sql_ref: "rollback/20260522_drop_index.sql"
  backup_required: true
```

PostgreSQL soporta transacciones con `BEGIN`/`COMMIT` y `ROLLBACK`; SQL Server documenta backups/restores y rollback de transacciones, pero SDO debe asumir que no todo DDL en todos los motores es igual de reversible. ([PostgreSQL][9])

### API REST

```yaml
executor:
  type: "api_executor"

planned_action:
  action_type: "http_request"
  method: "POST"
  url: "https://api.example.com/v1/resources"
  headers:
    Idempotency-Key: "SDO-2026-0001-STEP-001"
  body_ref: "payloads/create-resource.json"

execution_controls:
  retry:
    enabled: true
    max_attempts: 3
    retry_on: [429, 500, 502, 503, 504]
  timeout_seconds: 30
```

HTTP define la idempotencia por efecto sobre el servidor, no por igualdad de respuesta; para métodos no idempotentes como POST/PATCH, el header `Idempotency-Key` se usa precisamente para hacer reintentos más seguros cuando la API lo soporta. ([RFC Editor][2])

### Kubernetes

```yaml
executor:
  type: "kubernetes_executor"

planned_action:
  action_type: "declarative_apply"
  manifest_ref: "k8s/deployment.yaml"

preview:
  mode: "diff"
  command: "kubectl diff -f k8s/deployment.yaml"

execute:
  command: "kubectl apply -f k8s/deployment.yaml"

validation:
  commands:
    - "kubectl rollout status deployment/app"

rollback:
  command: "kubectl rollout undo deployment/app"
```

Kubernetes recomienda gestión declarativa con `kubectl apply`; `kubectl diff` previsualiza cambios, y `kubectl rollout undo` permite revertir rollouts soportados. ([Kubernetes][10])

### Terraform

```yaml
executor:
  type: "iac_executor"
  tool: "terraform"

preview:
  mode: "plan"
  command: "terraform plan -out=tfplan"

execute:
  command: "terraform apply tfplan"

rollback:
  rollback_type: "reapply_previous_config_or_compensating_change"
```

Terraform separa `plan` de `apply`: `plan` calcula acciones propuestas y `apply` ejecuta las operaciones del plan. ([HashiCorp Developer][1])

### GUI manual

```yaml
executor:
  type: "manual_executor"

manual_instruction:
  system: "Cloud console"
  steps:
    - "Open instance details"
    - "Confirm instance ID matches approved target"
    - "Click Reboot"
    - "Capture screenshot after state changes"

evidence:
  screenshots_required: true
  operator_notes_required: true
  sign_off_required: true
```

---

## 15. Anti-patrones a evitar

1. **Atar SDO a comandos**: SDO no debe ser “YAML para bash”.
2. **Confundir runbook con ejecución real**: el runbook dice qué hacer; el execution record demuestra qué ocurrió.
3. **Aceptar preview falso**: si no hay dry-run real, debe decir `unsupported`.
4. **No separar planned vs actual**: sin esa separación no hay auditoría seria.
5. **Rollback decorativo**: “volver atrás” no basta; debe haber acción, backup, restore, snapshot o compensación.
6. **Idempotencia asumida**: debe declararse y validarse.
7. **Fleet sin batch control**: operar 100 nodos como si fueran 1 es un fallo metodológico.
8. **GUI sin evidencia**: las acciones manuales necesitan screenshots/notas/sign-off.
9. **IA con permisos amplios**: riesgo de excessive agency; debe tener límites, approvals y trazabilidad.
10. **Secrets en logs**: los secretos se referencian, no se imprimen.

---

## 16. Definición corta del modelo

> **SDO Execution Model** es el contrato metodológico que transforma una Operational Spec aprobada en una secuencia de acciones planificadas, vinculadas a ejecutores concretos, ejecutadas bajo controles declarados, comparadas contra lo aprobado, validadas con criterios objetivos y cerradas con evidencia auditable.

La pieza central sería:

```yaml
sdo_execution_model:
  input: "Operational Spec"
  produces:
    - "Execution Plan"
    - "Planned Actions"
    - "Execution Records"
    - "Validation Results"
    - "Evidence Package"
    - "Rollback/Recovery Records"
  invariant:
    - "Every executed action must map to an approved planned action"
    - "Every deviation must be classified"
    - "Every state-changing action must have evidence"
    - "Every non-previewable action must have compensating controls"
    - "Every rollback must be executable, verifiable, or explicitly marked unavailable"
```

Mi recomendación: para SDO, este modelo de ejecución debería convertirse en un capítulo propio llamado **Execution Abstraction Layer** o **Executor Contract Model**.

[1]: https://developer.hashicorp.com/terraform/cli/commands/plan?utm_source=chatgpt.com "terraform plan command reference"
[2]: https://www.rfc-editor.org/rfc/rfc9110.html?utm_source=chatgpt.com "RFC 9110: HTTP Semantics"
[3]: https://man7.org/linux/man-pages/man1/journalctl.1.html?utm_source=chatgpt.com "journalctl(1) - Linux manual page"
[4]: https://docs.github.com/actions/deployment/targeting-different-environments/using-environments-for-deployment?utm_source=chatgpt.com "Managing environments for deployment"
[5]: https://learn.microsoft.com/en-us/powershell/scripting/learn/ps101/09-functions?view=powershell-7.6&utm_source=chatgpt.com "Functions - PowerShell"
[6]: https://developers.openai.com/api/docs/guides/agents/guardrails-approvals?utm_source=chatgpt.com "Guardrails and human review | OpenAI API"
[7]: https://www.freedesktop.org/software/systemd/man/systemd.service.html?utm_source=chatgpt.com "systemd.service"
[8]: https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-service?view=powershell-7.6&utm_source=chatgpt.com "Get-Service (Microsoft.PowerShell.Management)"
[9]: https://www.postgresql.org/docs/current/tutorial-transactions.html?utm_source=chatgpt.com "Documentation: 18: 3.4. Transactions"
[10]: https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/?utm_source=chatgpt.com "Declarative Management of Kubernetes Objects Using ..."
