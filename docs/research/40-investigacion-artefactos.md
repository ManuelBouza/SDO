> Research note — non-normative.
> This document is background material and may be superseded by current SDO core/reference documentation.
> Current normative docs start at [../start-here.md](../start-here.md).

A continuación separo **prácticas aceptadas** de **propuesta SDO**.

## 0. Base comparada

**Prácticas aceptadas:**

SDD moderno suele trabajar con un flujo tipo **Spec → Plan → Tasks → Implement**, donde cada fase produce artefactos Markdown reutilizables por humanos y agentes. GitHub Spec Kit lo documenta explícitamente así, con gates de revisión entre fases. ([GitHub Documentation][1])

ITSM/Change Management suele registrar justificación, plan de implementación, análisis de riesgo/impacto, plan de backout, plan de pruebas, ventana planificada, aprobación y cierre. ServiceNow lo refleja en sus modelos de cambio. ([ServiceNow][2])

SRE aporta artefactos de **SLO**, **error budget policy**, **playbooks**, **incident docs** y **postmortems**. Google SRE recomienda postmortems con impacto, causa raíz, timeline, datos, acciones y owners; también vincula error budgets con decisiones de cambio. ([Google SRE][3])

GitOps/IaC aporta manifiestos declarativos, estado deseado versionado, PRs, reconciliación, diffs/planes y logs de pipeline. OpenGitOps resume sus principios como declarativo, versionado/inmutable, pull automático y reconciliación continua. ([Open GitOps][4]) Kubernetes documenta el uso de configuración declarativa en Git y `diff/apply` para anticipar cambios. ([Kubernetes][5])

Automatización operativa aporta runbooks ejecutables, parámetros, aprobaciones y logs. AWS Systems Manager Automation permite acciones de aprobación dentro de un runbook mediante `aws:approve`. ([AWS Documentation][6]) Ansible aporta `check mode` y `diff mode` para simular y mostrar cambios, aunque advierte que el diff puede revelar información sensible. ([Ansible Documentation][7])

En seguridad/gobernanza, NIST SP 800-128 recomienda analizar impacto de seguridad antes, durante y después de cambios, enlazando control de configuración con gestión de riesgo. ([NIST Publications][8])

En bases de datos, la evidencia de backup/restore y rollback no debe asumirse. PostgreSQL documenta `pg_dump` como backup lógico consistente y `pg_restore/psql` para restauración. ([PostgreSQL][9]) Liquibase advierte que rollback no equivale a backup de datos y recomienda definir rollback para changesets. ([Liquibase Support][10])

---

# 1. Modelo propuesto de artefactos SDO

## 1.1 Artefactos core

**Propuesta SDO:**

Todo flujo SDO debería poder producir, como mínimo:

1. **Intent Record**
   Captura qué se quiere lograr y por qué.

2. **Operational Spec**
   Fuente de verdad operacional: alcance, estado esperado, restricciones, criterios de validación y límites.

3. **Execution Plan**
   Estrategia de ejecución: orden lógico, dependencias, ventana, actores, sistemas afectados.

4. **SDO Runbook**
   Procedimiento ejecutable o manual, con pasos, comandos, parámetros, checkpoints, rollback hooks y validaciones.

5. **Execution Record**
   Registro de lo que realmente ocurrió: quién ejecutó, cuándo, dónde, versión usada, resultado de pasos.

6. **Validation Report**
   Evidencia de que el estado final cumple la spec.

7. **Review Record**
   Cierre, aprendizaje, desviaciones, acciones posteriores.

## 1.2 Artefactos transversales

**Propuesta SDO:**

Estos no son fases, sino controles enlazados a varias fases:

1. **Risk Assessment**
2. **Approval Record**
3. **Evidence Package**
4. **Rollback Plan / Recovery Plan**
5. **Decision Log**
6. **Traceability Index**
7. **Sensitive Data Handling Record**

---

# 2. Core vs opcionales

| Artefacto              |             Tipo |                       Obligatorio general | Cuándo se amplía                          |
| ---------------------- | ---------------: | ----------------------------------------: | ----------------------------------------- |
| Intent Record          |             Core |                                        Sí | Siempre                                   |
| Operational Spec       |             Core |                                        Sí | Siempre                                   |
| Execution Plan         |             Core | Sí, excepto micro-operaciones Lightweight | Normal, Critical, Emergency, Pipeline     |
| SDO Runbook            |             Core |                                        Sí | Siempre, aunque sea mínimo                |
| Execution Record       |             Core |                                        Sí | Siempre                                   |
| Validation Report      |             Core |                                        Sí | Siempre                                   |
| Review Record          |             Core |                                Sí, mínimo | Más amplio en Critical/Emergency          |
| Risk Assessment        |      Transversal |                                Sí, mínimo | Normal/Critical/Emergency                 |
| Approval Record        |      Transversal |                        Depende del riesgo | Normal/Critical/Emergency                 |
| Evidence Package       |      Transversal |                                Sí, mínimo | Critical, auditables, DB, seguridad       |
| Rollback/Recovery Plan |      Transversal |                  Sí, al menos declaración | Más detallado si hay cambio destructivo   |
| Decision Log           |      Transversal |                                  Opcional | Critical, Emergency, incidentes           |
| Communication Log      |      Transversal |                                  Opcional | Emergency, producción, impacto usuario    |
| Change Schedule        | Externo/SDO-link |                                  Opcional | ITSM, cambios productivos                 |
| PR/Pipeline Record     | Externo/SDO-link |                                  Opcional | GitOps/IaC/Pipeline-backed                |
| Postmortem/PIR         |         Opcional |                                        No | Fallo, incidente, emergency, alto impacto |

---

# 3. Relación tipo grafo / lifecycle

**Propuesta SDO:**

```text
Intent
  └── produces → Operational Spec
        ├── constrained_by → Risk Assessment
        ├── references → Existing Ticket / Change / PR / Incident
        ├── produces → Execution Plan
        │      ├── requires → Approval Record
        │      ├── schedules → Change Window
        │      └── produces → SDO Runbook
        │              ├── executes_as → Manual / CLI / Script / API / Pipeline / GitOps
        │              ├── emits → Execution Record
        │              ├── emits → Logs
        │              └── may_trigger → Rollback Plan
        ├── validated_by → Validation Report
        ├── evidenced_by → Evidence Package
        └── closed_by → Review Record
```

Clave: **SDO no debe duplicar sistemas externos**. Debe guardar referencias estables:

```yaml
external_refs:
  ticket: CHG003421
  pr: https://github.com/org/repo/pull/123
  pipeline_run: gitlab/project/-/pipelines/7788
  wiki_page: RUNBOOK-DB-014
  incident: INC001882
```

---

# 4. Tabla fase → artefacto → campos mínimos

| Fase SDO    | Artefacto         | Campos mínimos                                                                                                                     | Obligatorio   | Formatos posibles                            |
| ----------- | ----------------- | ---------------------------------------------------------------------------------------------------------------------------------- | ------------- | -------------------------------------------- |
| Intent      | Intent Record     | id, título, solicitante, objetivo, motivo, urgencia, resultado esperado                                                            | Sí            | Ticket, Markdown, YAML, formulario ITSM      |
| Spec        | Operational Spec  | id, clase base, perfil, scope, targets, precondiciones, estado esperado, criterios de aceptación, restricciones, riesgos iniciales | Sí            | Markdown, YAML/JSON, ticket, PR description  |
| Plan        | Execution Plan    | estrategia, secuencia, dependencias, ventana, responsables, impacto, validaciones, punto de no retorno                             | Sí en Normal+ | Markdown, change record, pipeline plan       |
| Plan        | Risk Assessment   | impacto, probabilidad, blast radius, criticidad, reversibilidad, datos afectados, seguridad, score                                 | Sí, mínimo    | YAML, ITSM risk form, ticket fields          |
| Plan        | Approval Record   | aprobador, criterio, fecha, decisión, condiciones                                                                                  | Según riesgo  | ITSM approval, PR review, pipeline gate      |
| Runbook     | SDO Runbook       | pasos, comandos/acciones, parámetros, checks, esperado por paso, rollback hooks, owner                                             | Sí            | Markdown, YAML, wiki, job template           |
| Execution   | Execution Record  | operador, inicio/fin, versión, entorno, steps ejecutados, outputs relevantes, desviaciones                                         | Sí            | Log, ticket comment, pipeline log, JSON      |
| Execution   | Decision Log      | timestamp, decisión, contexto, alternativas, responsable                                                                           | Opcional      | Markdown, incident timeline, ticket comments |
| Validation  | Validation Report | pruebas realizadas, resultado esperado/obtenido, evidencia, estado final                                                           | Sí            | Markdown, test report, pipeline artifacts    |
| Review      | Review Record     | resultado, desviaciones, lecciones, acciones, cierre                                                                               | Sí, mínimo    | PIR, postmortem, ticket close notes          |
| Transversal | Evidence Package  | índice, evidencias, hash/enlace, clasificación, retención, owner                                                                   | Sí, mínimo    | Carpeta, ZIP, S3, Drive, ITSM attachments    |
| Transversal | Rollback Plan     | estrategia, gatillos, pasos, límite temporal, validación post-rollback                                                             | Sí, mínimo    | Markdown, runbook section, pipeline job      |

---

# 5. Campos mínimos de una Operational Spec

**Propuesta SDO:**

```yaml
sdo_spec:
  id: SDO-YYYY-NNN
  title: ""
  version: "1.0.0"
  status: draft | approved | executing | completed | failed | superseded
  owner: ""
  requested_by: ""
  created_at: ""
  updated_at: ""

  classification:
    base_class: Observe | Analyze | Maintain | Configure | Deploy | Control | Data | Access | Remediate | Destroy
    profile: SDO-Lightweight | SDO-Normal | SDO-Critical | SDO-Emergency | SDO-Pipeline-Backed
    risk_level: low | medium | high | critical
    environment: dev | test | staging | prod
    operation_type: manual | cli | script | api | sql | pipeline | gitops | mixed

  intent:
    objective: ""
    business_or_operational_reason: ""
    expected_outcome: ""
    non_goals: []

  scope:
    targets:
      - type: server | service | database | cluster | repository | account | user | network | application
        id: ""
        environment: ""
    included: []
    excluded: []

  current_state:
    summary: ""
    evidence_refs: []

  desired_state:
    summary: ""
    measurable_acceptance_criteria:
      - ""

  constraints:
    allowed_window: ""
    downtime_allowed: true
    max_expected_downtime: ""
    compliance_constraints: []
    security_constraints: []
    data_constraints: []

  prerequisites:
    access_required: []
    dependencies: []
    backups_required: false
    backup_refs: []
    dry_run_required: false

  risk:
    assessment_ref: ""
    known_risks: []
    blast_radius: ""
    reversibility: reversible | partially_reversible | irreversible
    approval_required: true

  validation:
    pre_checks: []
    post_checks: []
    success_criteria: []
    monitoring_window: ""

  rollback:
    rollback_ref: ""
    rollback_possible: true
    rollback_limit: ""
    fallback_strategy: ""

  traceability:
    ticket_refs: []
    pr_refs: []
    pipeline_refs: []
    runbook_ref: ""
    evidence_package_ref: ""
```

## Campos que no deben faltar

Para no caer en documentación decorativa, la spec mínima debe responder:

```text
Qué se quiere cambiar/operar.
Por qué.
Dónde aplica.
Qué queda fuera.
Qué estado final se espera.
Cómo se valida.
Qué riesgo tiene.
Quién aprueba si aplica.
Cómo se revierte o recupera.
Dónde queda la evidencia.
```

---

# 6. Plantilla base de Runbook SDO

**Propuesta SDO:**

```yaml
sdo_runbook:
  id: RB-SDO-YYYY-NNN
  spec_id: SDO-YYYY-NNN
  title: ""
  version: "1.0.0"
  owner: ""
  execution_mode: manual | cli | script | api | sql | pipeline | gitops | mixed
  profile: SDO-Lightweight | SDO-Normal | SDO-Critical | SDO-Emergency | SDO-Pipeline-Backed

  safety:
    requires_approval: true
    approval_ref: ""
    requires_backup: false
    backup_validation_ref: ""
    stop_conditions:
      - ""
    point_of_no_return: ""
    rollback_ref: ""

  parameters:
    - name: ""
      description: ""
      required: true
      default: ""
      sensitive: false
      validation: ""

  pre_checks:
    - id: PRE-001
      description: ""
      command_or_action: ""
      expected_result: ""
      evidence_required: true

  steps:
    - id: STEP-001
      description: ""
      actor: human | automation | pipeline
      command_or_action: ""
      expected_result: ""
      timeout: ""
      retry_policy: ""
      validation: ""
      evidence_required: true
      rollback_hook: ""

  post_checks:
    - id: POST-001
      description: ""
      command_or_action: ""
      expected_result: ""
      evidence_required: true

  rollback:
    trigger_conditions:
      - ""
    steps:
      - id: RBK-001
        description: ""
        command_or_action: ""
        expected_result: ""
    validation_after_rollback:
      - ""

  evidence:
    required_items:
      - pre_state
      - execution_log
      - post_state
      - validation_result
    storage_ref: ""

  closeout:
    success_condition: ""
    failure_condition: ""
    review_required: true
```

---

# 7. Evidence Package

**Práctica aceptada:** logs, timelines, diffs, test outputs, backups y aprobaciones se usan habitualmente como evidencia en ITSM, SRE, IaC y auditoría. Atlassian describe la timeline de incidente como registro central de qué pasó, cuándo y con qué contexto. ([Atlassian][11])

**Propuesta SDO:**

```yaml
sdo_evidence_package:
  id: EVD-SDO-YYYY-NNN
  spec_id: SDO-YYYY-NNN
  runbook_id: RB-SDO-YYYY-NNN
  classification: public | internal | confidential | restricted
  retention_policy: ""
  owner: ""
  created_at: ""

  evidence_items:
    - id: EVD-001
      type: screenshot | log | command_output | diff | backup_proof | approval | pipeline_artifact | monitoring_graph | ticket_comment
      description: ""
      source: ""
      storage_ref: ""
      captured_at: ""
      captured_by: human | system
      hash: ""
      sensitive: false
      redaction_applied: false

  validation_summary:
    result: passed | failed | partial
    validation_report_ref: ""

  audit:
    immutable_storage: true
    access_control_ref: ""
    chain_of_custody: []
```

## Evidencia mínima por perfil

| Perfil              | Evidencia mínima                                                                    |
| ------------------- | ----------------------------------------------------------------------------------- |
| SDO-Lightweight     | Resultado final + nota de ejecución                                                 |
| SDO-Normal          | Pre-check, ejecución, post-check, validación                                        |
| SDO-Critical        | Todo lo anterior + aprobación, backup, rollback readiness, logs completos, decisión |
| SDO-Emergency       | Timeline, decisiones, mitigación, validación rápida, post-review obligatorio        |
| SDO-Pipeline-Backed | PR, diff/plan, pipeline run, artifacts, logs, aprobación/gate si aplica             |

---

# 8. Risk / Approval / Rollback como artefactos transversales

## 8.1 Risk Assessment

```yaml
risk_assessment:
  id: RISK-SDO-YYYY-NNN
  spec_id: SDO-YYYY-NNN
  impact: low | medium | high | critical
  likelihood: low | medium | high
  blast_radius:
    users_affected: ""
    services_affected: []
    environments_affected: []
  reversibility: reversible | partially_reversible | irreversible
  data_risk: none | low | medium | high
  security_risk: none | low | medium | high
  compliance_risk: none | low | medium | high
  operational_risk: none | low | medium | high
  risk_score: 0
  mitigations: []
  residual_risk: low | medium | high | critical
  approval_required: true
```

## 8.2 Approval Record

```yaml
approval_record:
  id: APR-SDO-YYYY-NNN
  spec_id: SDO-YYYY-NNN
  approval_type: peer | owner | change_manager | security | data_owner | emergency
  approver: ""
  decision: approved | rejected | conditionally_approved
  conditions: []
  approved_at: ""
  evidence_ref: ""
```

## 8.3 Rollback / Recovery Plan

```yaml
rollback_plan:
  id: RBK-SDO-YYYY-NNN
  spec_id: SDO-YYYY-NNN
  strategy: rollback | roll_forward | restore | failover | manual_recovery | not_possible
  rollback_possible: true
  last_safe_point: ""
  point_of_no_return: ""
  triggers:
    - ""
  steps:
    - id: RBK-001
      action: ""
      expected_result: ""
  validation:
    - ""
  estimated_time: ""
  data_loss_risk: none | possible | expected
  backup_refs: []
```

Importante: en operaciones de datos, **rollback no sustituye backup**. Liquibase lo deja claro: no hace backup de datos automáticamente antes de desplegar changesets. ([Liquibase Support][12])

---

# 9. Cómo evitar duplicar tickets, PRs, runbooks, pipelines o change records

**Propuesta SDO:**

Regla principal:

```text
SDO debe ser la capa semántica, no otro repositorio duplicado de información.
```

## Patrón recomendado

| Sistema existente | Qué debe contener                              | Qué NO debe duplicarse                                |
| ----------------- | ---------------------------------------------- | ----------------------------------------------------- |
| Jira/ServiceNow   | Workflow, aprobaciones, SLA, ownership, cierre | No copiar logs completos si están en pipeline/storage |
| PR GitHub/GitLab  | Diff, revisión técnica, plan IaC/GitOps        | No repetir todo el change record                      |
| Pipeline          | Ejecución, logs, artifacts, resultado          | No explicar todo el contexto operacional              |
| Wiki/runbook      | Procedimiento reusable                         | No registrar cada ejecución histórica completa        |
| Markdown repo SDO | Spec versionada, plantillas, trazabilidad      | No sustituir ITSM si la empresa exige ITSM            |
| Evidence storage  | Evidencias pesadas, logs, screenshots          | No incrustar secretos ni adjuntos enormes en spec     |

## Técnica concreta

Usar **referencias canónicas**:

```yaml
source_of_truth:
  intent: jira_ticket
  operational_spec: repo_markdown
  approval: servicenow_change
  implementation_diff: github_pr
  execution: pipeline_run
  evidence: s3_or_drive_folder
  review: ticket_closure_or_postmortem
```

---

# 10. Mapeo a sistemas existentes

## 10.1 Markdown repo

```text
/sdo/
  /specs/SDO-2026-001.md
  /runbooks/RB-SDO-2026-001.md
  /risk/RISK-SDO-2026-001.md
  /evidence/EVD-SDO-2026-001.index.md
  /reviews/REV-SDO-2026-001.md
```

Bueno para: versionado, PR review, trazabilidad, operaciones repetibles.

## 10.2 YAML/JSON manifest

Bueno para: automatización, validación con schema, pipelines, generación de runbooks.

```yaml
apiVersion: sdo.io/v1
kind: OperationalSpec
metadata:
  id: SDO-2026-001
  version: 1.0.0
spec:
  baseClass: Configure
  profile: SDO-Normal
  target:
    environment: prod
  desiredState: ""
  validation: []
```

## 10.3 Jira / ServiceNow ticket

Mapeo recomendado:

| SDO             | Ticket/Change                                  |
| --------------- | ---------------------------------------------- |
| Intent          | Summary / Description / Business justification |
| Risk Assessment | Risk / Impact / Urgency / Risk questionnaire   |
| Execution Plan  | Implementation plan                            |
| Rollback Plan   | Backout plan                                   |
| Validation      | Test plan                                      |
| Approval Record | Approval tab / workflow                        |
| Review Record   | Close notes / PIR                              |

ServiceNow ya maneja campos similares: justification, implementation plan, risk and impact analysis, backout plan, test plan y planned dates. ([ServiceNow][2])

## 10.4 GitHub/GitLab PR

Mapeo recomendado:

| SDO              | PR                                        |
| ---------------- | ----------------------------------------- |
| Operational Spec | PR description o archivo `/sdo/specs/...` |
| Plan             | Checklist del PR                          |
| Approval         | Review approval                           |
| Evidence         | CI artifacts                              |
| Validation       | Checks obligatorios                       |
| Rollback         | Revert plan / rollback job                |

## 10.5 CI/CD pipeline

Mapeo recomendado:

| SDO        | Pipeline                                |
| ---------- | --------------------------------------- |
| Pre-checks | validate/lint/plan                      |
| Risk gate  | manual approval / protected environment |
| Execution  | deploy/apply/migrate                    |
| Validation | smoke tests / health checks             |
| Evidence   | artifacts/logs                          |
| Rollback   | rollback job                            |

## 10.6 Wiki/runbook platform

Bueno para runbooks estables. Pero cada ejecución debe tener su propio **Execution Record**, no editar el runbook cada vez.

---

# 11. Versionado y trazabilidad

**Propuesta SDO:**

Cada artefacto debe tener:

```yaml
id: SDO-2026-001
version: 1.2.0
status: draft | approved | active | completed | failed | deprecated
created_at: ""
updated_at: ""
owner: ""
related_refs: []
```

## Reglas

1. **Spec versionada**: cada cambio significativo incrementa versión.
2. **Runbook versionado**: una ejecución apunta a una versión exacta.
3. **Evidence inmutable**: no editar evidencia; añadir nuevas evidencias.
4. **Execution Record no reescribible**: corregir mediante anexos.
5. **Traceability ID único**: aparece en ticket, PR, pipeline, logs y evidence package.

Ejemplo:

```yaml
traceability:
  sdo_id: SDO-2026-001
  spec_version: 1.1.0
  runbook_version: 2.0.0
  git_commit: a1b2c3d
  ticket: CHG003421
  pipeline_run: 7788
```

---

# 12. Evidencia sensible y secretos

**Prácticas aceptadas:** Ansible advierte que `diff mode` puede revelar información sensible y permite desactivarlo por tarea. ([Ansible Documentation][7]) NIST exige considerar impacto de seguridad durante cambios de configuración. ([NIST Publications][8])

**Propuesta SDO:**

Reglas mínimas:

```text
Nunca guardar secretos en Spec, Runbook, Evidence o logs.
Guardar referencias a secretos, no valores.
Redactar outputs antes de adjuntarlos.
Clasificar evidencia por sensibilidad.
Aplicar retención y control de acceso.
Guardar hashes para integridad.
Separar evidencia técnica de evidencia sensible.
```

Ejemplo:

```yaml
secrets:
  handling: external_reference_only
  allowed_refs:
    - aws_secrets_manager:prod/db/admin
    - vault:kv/prod/app
  forbidden:
    - plain_text_passwords
    - tokens_in_logs
    - private_keys
```

---

# 13. Adaptación por perfil

## SDO-Lightweight

Para cambios de bajo riesgo, observaciones, análisis o mantenimiento rutinario.

**Artefactos mínimos:**

```text
Intent breve
Operational Spec mínima
Runbook/checklist corta
Execution Record
Validation mínima
```

No debe requerir aprobación pesada salvo política externa.

## SDO-Normal

Para cambios productivos controlados.

**Artefactos:**

```text
Spec completa
Plan
Risk Assessment
Approval si aplica
Runbook
Execution Record
Validation Report
Evidence Package básico
Review breve
```

## SDO-Critical

Para alto impacto, datos, seguridad, producción crítica o baja reversibilidad.

**Artefactos:**

```text
Spec completa
Risk Assessment detallado
Approval formal
Plan con ventana
Runbook paso a paso
Rollback/Recovery probado o justificado
Backup evidence
Evidence Package completo
Decision Log
Review/PIR obligatorio
```

## SDO-Emergency

Para restaurar servicio o contener impacto.

**Idea clave:** menos documentación previa, más trazabilidad durante y después.

**Artefactos:**

```text
Emergency Intent
Risk rápido
Approval de emergencia o autorización verbal registrada
Execution timeline
Decision Log
Validation inmediata
Evidence mínima
Post-incident Review obligatorio
Spec retroactiva si el cambio queda permanente
```

Atlassian recomienda registrar observaciones y decisiones durante incidentes para facilitar postmortems y detectar relación con cambios recientes. ([Atlassian][13])

## SDO-Pipeline-Backed

Para GitOps, IaC, CI/CD, migraciones automatizadas.

**Artefactos:**

```text
Spec en repo
PR como change vehicle
Plan/diff generado
Pipeline logs
Artifacts
Approval gate si aplica
Validation automatizada
Rollback job o roll-forward strategy
Evidence Package generado por pipeline
```

Esto encaja con GitOps: estado deseado declarativo, versionado, pull automático y reconciliación continua. ([Open GitOps][4])

---

# 14. Plantilla Markdown de Operational Spec

```markdown
# SDO Operational Spec: <título>

## Metadata
- SDO ID:
- Version:
- Status:
- Owner:
- Requested by:
- Created at:
- Environment:
- Base Class:
- Flow Profile:

## 1. Intent
Qué se quiere lograr:

Por qué es necesario:

Resultado esperado:

No objetivos:

## 2. Scope
Incluye:

Excluye:

Targets afectados:

## 3. Current State
Estado actual conocido:

Evidencia del estado actual:

## 4. Desired State
Estado deseado:

Criterios de aceptación:

## 5. Constraints
Ventana permitida:

Downtime permitido:

Restricciones de seguridad:

Restricciones de datos:

Dependencias:

## 6. Risk Summary
Nivel de riesgo:

Blast radius:

Reversibilidad:

Riesgos principales:

Mitigaciones:

Approval requerido:

## 7. Execution Strategy
Resumen del plan:

Método de ejecución:
- Manual
- CLI
- Script
- API
- SQL
- Pipeline
- GitOps
- Mixto

## 8. Validation
Pre-checks:

Post-checks:

Criterios de éxito:

Periodo de observación:

## 9. Rollback / Recovery
Rollback posible:

Estrategia:

Punto de no retorno:

Referencia al rollback plan:

## 10. Evidence
Evidencia requerida:

Ubicación de evidencia:

Retención:

## 11. Traceability
Ticket:

PR:

Pipeline:

Runbook:

Evidence package:

Review:
```

---

# 15. Plantilla Markdown de Runbook

```markdown
# SDO Runbook: <título>

## Metadata
- Runbook ID:
- Spec ID:
- Version:
- Owner:
- Execution mode:
- Environment:
- Approved change/ticket:

## Safety
- Approval required:
- Approval reference:
- Backup required:
- Backup evidence:
- Stop conditions:
- Point of no return:
- Rollback reference:

## Parameters
| Name | Required | Sensitive | Description | Validation |
|---|---:|---:|---|---|

## Pre-checks
| ID | Action | Expected result | Evidence |
|---|---|---|---|

## Execution Steps
| Step | Actor | Action/Command | Expected result | Validation | Evidence | Rollback hook |
|---|---|---|---|---|---|---|

## Post-checks
| ID | Action | Expected result | Evidence |
|---|---|---|---|

## Rollback
### Trigger conditions

### Steps

### Validation after rollback

## Evidence Collection
Required evidence:

Storage location:

Sensitive data handling:

## Closeout
Success condition:

Failure condition:

Review required:
```

---

# 16. Plantilla Markdown de Evidence Package

```markdown
# SDO Evidence Package: <SDO-ID>

## Metadata
- Evidence Package ID:
- Spec ID:
- Runbook ID:
- Execution ID:
- Owner:
- Classification:
- Retention:
- Created at:

## Summary
Resultado general:

Estado final:

Validación:

## Evidence Index
| ID | Type | Description | Source | Captured at | Storage ref | Sensitive | Hash |
|---|---|---|---|---|---|---:|---|

## Required Evidence Checklist
- [ ] Pre-state evidence
- [ ] Approval evidence
- [ ] Backup evidence, if required
- [ ] Execution logs
- [ ] Post-state evidence
- [ ] Validation results
- [ ] Rollback evidence, if executed
- [ ] Review record

## Sensitive Data Handling
Redaction applied:

Secrets excluded:

Access control:

## Validation Summary
Expected:

Observed:

Result:

## Audit Notes
Immutability:

Chain of custody:

Exceptions:
```

---

# 17. Anti-patrones de documentación

1. **Spec theater**
   Documentos largos que no guían ejecución ni validación.

2. **Duplicar el ticket entero en Markdown**
   SDO debe enlazar, no copiar sin necesidad.

3. **Runbook sin criterios de parada**
   Peligroso en producción.

4. **Rollback genérico tipo “restore backup”**
   Insuficiente si no dice qué backup, cómo restaurar, cuánto tarda y cómo validar.

5. **Evidencia como screenshots sueltos**
   Sirven, pero sin índice, timestamp, contexto y owner pierden valor.

6. **Aprobaciones fuera de trazabilidad**
   “Me lo aprobaron por chat” solo vale si queda registrado.

7. **Logs con secretos**
   Especialmente peligroso en pipelines, Ansible diff, SQL outputs y capturas.

8. **Postmortem sin acciones trazables**
   Si no hay owner, fecha y seguimiento, queda como documento decorativo.

9. **Una plantilla única para todo**
   Rompe el modelo adaptable por riesgo.

10. **Confundir rollback con reversibilidad real**
    Algunas operaciones son irreversibles o solo recuperables por restore/roll-forward.

---

# 18. Síntesis final del modelo SDO

**Propuesta propia:**

SDO debería tener un sistema de artefactos con esta jerarquía:

```text
Operational Spec = fuente de verdad
Runbook = procedimiento ejecutable
Execution Record = realidad observada
Validation Report = prueba de cumplimiento
Evidence Package = paquete auditable
Review Record = aprendizaje/cierre
Risk, Approval y Rollback = controles transversales
```

La idea central:

```text
SDO no crea más burocracia.
SDO crea trazabilidad operacional mínima, adaptable al riesgo y portable entre herramientas.
```

La unidad principal no debería ser el documento, sino el **SDO ID** que enlaza spec, ticket, PR, pipeline, evidencia y revisión.

[1]: https://github.github.com/spec-kit/index.html?utm_source=chatgpt.com "GitHub Spec Kit | Spec Kit Documentation"
[2]: https://www.servicenow.com/docs/r/operational-technology/operational-technology-change-management/basic-ot-change-model.html?utm_source=chatgpt.com "Basic OT Change Model playbook"
[3]: https://sre.google/workbook/table-of-contents/?utm_source=chatgpt.com "Google SRE - SRE workbook table of content"
[4]: https://opengitops.dev/?utm_source=chatgpt.com "Home | OpenGitOps"
[5]: https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config?utm_source=chatgpt.com "Declarative Management of Kubernetes Objects Using Configuration Files | Kubernetes"
[6]: https://docs.aws.amazon.com/systems-manager/latest/userguide/running-automations-require-approvals.html?utm_source=chatgpt.com "Run an automation that requires approvals - AWS Systems Manager"
[7]: https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_checkmode.html?utm_source=chatgpt.com "Validating tasks: check mode and diff mode — Ansible Community Documentation"
[8]: https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-128.pdf?utm_source=chatgpt.com "NIST Special Publication 800-128"
[9]: https://www.postgresql.org/docs/17/app-pgdump.html?utm_source=chatgpt.com "PostgreSQL: Documentation: 17: pg_dump"
[10]: https://support.liquibase.com/hc/en-us/articles/29383086010523-How-to-Define-Rollbacks?utm_source=chatgpt.com "How to Define Rollbacks – Liquibase"
[11]: https://www.atlassian.com/incident-management/postmortem/timelines?utm_source=chatgpt.com "Incident timelines (and why they matter) | Atlassian"
[12]: https://support.liquibase.com/hc/en-us/articles/29383069283739-Does-Liquibase-Backup-Any-Data?utm_source=chatgpt.com "Does Liquibase Backup Any Data? – Liquibase"
[13]: https://www.atlassian.com/incident-management/handbook/incident-response?utm_source=chatgpt.com "How we respond to an incident | Atlassian"
