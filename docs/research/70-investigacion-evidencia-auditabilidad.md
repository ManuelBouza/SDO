> Research note — non-normative.
> This document is background material and may be superseded by current SDO core/reference documentation.
> Current normative docs start at [../start-here.md](../start-here.md).

# Modelo SDO Evidence & Auditability

**Idea central:** en SDO, la evidencia no es “guardar todo”. Es construir un **paquete verificable, mínimo y trazable** que permita demostrar que una operación fue especificada, aprobada, ejecutada, validada y cerrada correctamente, sin copiar logs masivos ni exponer secretos.

Leyenda:

* **[PA]** Práctica aceptada en marcos/herramientas existentes.
* **[SDO]** Propuesta propia para SDO.

---

## 1. Principio base: evidencia suficiente

**[SDO] Evidencia suficiente** significa que una persona auditora, distinta del operador, puede responder:

1. Qué se pretendía hacer.
2. Quién lo aprobó.
3. Qué acciones estaban planificadas.
4. Qué acciones se ejecutaron realmente.
5. Qué cambió.
6. Qué se validó.
7. Qué desviaciones ocurrieron.
8. Qué rollback/recovery existía o se ejecutó.
9. Qué evidencia respalda cada afirmación.
10. Dónde vive la evidencia original y cómo comprobar que no fue alterada.

**No significa evidencia exhaustiva.** SDO debe preferir referencias verificables, hashes, IDs de eventos, ventanas temporales, extractos mínimos y enlaces a sistemas fuente, siguiendo el enfoque de gestión de logs como ciclo de generación, transmisión, almacenamiento, acceso y disposición, no como copia indiscriminada de todo dato operativo. NIST SP 800-92 Rev. 1 define log management precisamente como gestión de logs para investigación, operación y retención requerida. ([csrc.nist.gov][1])

---

## 2. Comparación de prácticas existentes

### ITIL / ITSM

**[PA]** ITIL se centra en gobernanza flexible, valor, resultados y gestión del ciclo de vida de servicios digitales. ([peoplecert.org][2]) En Change Enablement, las prácticas aceptadas distinguen cambios estándar, normales y de emergencia; los normales requieren evaluación de riesgo y aprobación, mientras que los de emergencia se aceleran y deben revisarse después. ([ITSM.tools][3])

**[SDO]** SDO toma de ITIL la trazabilidad de cambio, aprobación, revisión y PIR, pero evita convertir cada operación en un ticket pesado. La unidad de trazabilidad no es “el ticket ITSM”, sino el **SDO ID**, que puede referenciar un ticket, PR, incidente o pipeline sin duplicarlo.

### SRE / Incident Response

**[PA]** SRE aporta líneas de tiempo, postmortems, revisiones sin culpa y criterios previos para decidir cuándo revisar un evento. Google SRE recomienda postmortems para eventos significativos y enfatiza que el objetivo es aprender y mejorar sistemas, no culpar personas. ([Google SRE][4])

**[SDO]** SDO reutiliza la idea de timeline y review, pero la aplica a operaciones normales, cambios planificados, remediaciones y emergencias. El `Review Record` de SDO puede ser ligero o formal según perfil.

### GitOps / IaC

**[PA]** GitOps exige estado deseado declarativo, versionado e inmutable, extracción automática y reconciliación continua. ([Open GitOps][5]) Terraform `plan` crea un plan de ejecución para previsualizar cambios, compara estado actual con configuración y propone acciones antes de aplicar. ([HashiCorp Developer][6])

**[SDO]** SDO generaliza esto: no todo será GitOps o Terraform, pero toda operación debe intentar producir un equivalente de **planned actions + actual execution + validation evidence**.

### CI/CD

**[PA]** Las plataformas CI/CD ya producen logs, artefactos, aprobaciones de entornos y registros de ejecución. GitHub Actions permite protección de entornos con aprobadores requeridos y reglas antes de desplegar. ([GitHub Docs][7]) También genera logs por workflow/job/step y permite descargar logs y artefactos. ([GitHub Docs][8]) Sus artefactos y logs tienen retención configurable; por defecto, GitHub indica 90 días. ([GitHub Docs][9])

**[SDO]** SDO no debe copiar todos los logs del pipeline. Debe guardar `pipeline_run_id`, commit SHA, environment approval, artifact IDs, hashes, outcome y enlaces permanentes.

### Seguridad, compliance y cloud audit

**[PA]** NIST SP 800-53 Rev. 5 organiza controles de seguridad y privacidad, incluyendo la familia **Audit and Accountability**, con enfoque adaptable al riesgo. ([csrc.nist.gov][10]) AWS CloudTrail muestra una práctica fuerte de integridad: digest files, SHA-256 y firmas digitales para detectar modificación o borrado de logs. ([AWS Documentation][11]) Google Cloud Audit Logs clasifica logs en Admin Activity, Data Access, System Event y Policy Denied; además declara que sus entradas de audit log son inmutables. ([Google Cloud Documentation][12]) Azure Activity Log registra eventos a nivel de suscripción y otros scopes administrativos. ([Microsoft Learn][13])

**[SDO]** SDO debe adoptar los conceptos de inmutabilidad, hash, firma, timestamp, actor identity y least privilege, pero sin depender de AWS/Azure/GCP. CloudTrail, Activity Log y GCP Audit Logs son ejemplos, no requisitos.

### Linux, Windows, DB, API y AI-assisted

**[PA]** En Linux, `auditd` escribe registros de auditoría al disco y se consulta con herramientas como `ausearch`/`aureport`. ([Man7][14]) En Windows/PowerShell, Microsoft documenta Script Block Logging, Transcription y advierte que el logging elevado puede capturar credenciales u otros datos sensibles. ([Microsoft Learn][15]) En bases de datos, Flyway usa una schema history table como audit trail de migraciones, con checksums y estado de éxito/fallo. ([Redgate Documentation][16]) En APIs distribuidas, W3C Trace Context estandariza propagación de identificadores de traza entre componentes. ([W3C][17]) En AI-assisted operations, OWASP LLM Top 10 destaca riesgos como sensitive information disclosure, excessive agency y overreliance. ([OWASP Foundation][18])

**[SDO]** SDO debe exigir evidencia operacional por canal de ejecución, pero con redacción fuerte. En AI-assisted ops, el modelo no es actor aprobador; el actor aprobador debe ser humano, sistema autorizado o política explícita.

---

# 3. Tipos de evidencia en SDO

## 3.1 Evidencia técnica

Prueba qué se ejecutó y qué cambió.

Ejemplos:

* comando ejecutado, pero redactado;
* script hash;
* pipeline run ID;
* Cloud/API event ID;
* `journalctl` / syslog / auditd reference;
* Windows Event ID;
* DB migration ID;
* checksum antes/después;
* diff de configuración;
* request ID / correlation ID;
* backup ID;
* restore test ID.

## 3.2 Evidencia de aprobación

Prueba que la operación estaba autorizada.

Ejemplos:

* approval record;
* PR approval;
* CAB/change authority decision;
* protected environment approval;
* emergency break-glass approval;
* peer review;
* business owner sign-off.

## 3.3 Evidencia de planificación

Prueba qué estaba previsto.

Ejemplos:

* Operational Spec snapshot;
* Execution Plan hash;
* Planned Actions list;
* Terraform plan / dry-run / preview;
* SQL migration preview;
* API dry-run;
* maintenance window;
* rollback plan.

## 3.4 Evidencia de ejecución

Prueba lo que ocurrió.

Ejemplos:

* executed action record;
* command transcript redactado;
* pipeline logs;
* audit event IDs;
* return code;
* response status;
* executor identity;
* timestamps;
* deviations.

## 3.5 Evidencia de validación

Prueba que el resultado fue aceptable.

Ejemplos:

* health check;
* smoke test;
* service status;
* metric snapshot;
* row count;
* checksum;
* endpoint response;
* business KPI;
* user acceptance;
* monitoring status.

## 3.6 Evidencia de rollback / recovery

Prueba que existía o se aplicó una salida segura.

Ejemplos:

* backup ID;
* restore proof;
* rollback command/script hash;
* previous version tag;
* restore point;
* compensating control;
* rollback unavailable justification.

## 3.7 Evidencia de negocio

Prueba que la operación cumplió una intención de negocio.

Ejemplos:

* incidencia resuelta;
* servicio restaurado;
* usuario desbloqueado;
* coste reducido;
* ventana cumplida;
* riesgo mitigado;
* aprobación del área solicitante.

---

# 4. Niveles de evidencia por perfil SDO

| Perfil                  | Evidencia mínima requerida                                                                                                                                                                                                                               |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **SDO-Lightweight**     | SDO ID, intención, spec breve, actor, planned action mínima, resultado, evidencia para acciones con cambio de estado, validación básica y cierre.                                                                                                        |
| **SDO-Normal**          | Todo lo anterior + approval record, risk/profile decision, plan/preview si existe, pre-checks, post-checks, evidencia por cada executed action, desviaciones clasificadas, rollback definido, referencias externas y hash del paquete.                   |
| **SDO-Critical**        | Todo SDO-Normal + doble control o aprobación reforzada, evidencia independiente del executor, hash manifest firmado, referencias inmutables, validación independiente, rollback probado o justificación formal, retención extendida, cadena de custodia. |
| **SDO-Emergency**       | Antes: intención, riesgo, autorización rápida o break-glass, acciones mínimas y evidencia crítica. Después: reconstrucción completa, timeline, desviaciones, validación, rollback/recovery, review/PIR obligatorio.                                      |
| **SDO-Pipeline-Backed** | Commit/PR, plan/diff, pipeline run, environment approval, artifact ID/hash, deployment logs, change result, validation job, rollback job o release previous version, links a logs y retención del pipeline.                                              |

---

# 5. Relación entre evidencia y objetos SDO

```yaml
sdo_traceability_model:
  sdo_id:
    links_to:
      - operational_spec
      - execution_plan
      - approval_record
      - evidence_package
      - validation_report
      - review_record

  operational_spec:
    evidence_required:
      - spec_snapshot_hash
      - author_identity
      - created_at
      - approved_version

  approval_record:
    evidence_required:
      - approver_identity
      - approval_decision
      - approval_scope
      - timestamp
      - approval_source_reference

  planned_action:
    evidence_required:
      - action_id
      - intended_state_change
      - expected_outcome
      - risk_level
      - rollback_mapping

  executed_action:
    evidence_required:
      - executed_action_id
      - mapped_planned_action_id
      - executor_identity
      - command_or_operation_summary
      - result_status
      - source_event_references
      - deviation_classification

  validation_report:
    evidence_required:
      - validation_check_id
      - success_criteria
      - observed_result
      - evidence_items
      - validator_identity

  rollback_or_recovery:
    evidence_required:
      - rollback_status
      - rollback_action_ids
      - backup_or_restore_reference
      - recovery_validation
      - unavailable_reason_if_any

  review_record:
    evidence_required:
      - closure_decision
      - lessons_learned
      - follow_up_actions
      - reviewer_identity
```

---

# 6. Estructura genérica de `sdo_evidence_package`

```yaml
sdo_evidence_package:
  schema_version: "1.0"
  sdo_id: "SDO-YYYYMMDD-0001"
  package_id: "EVP-SDO-YYYYMMDD-0001"
  package_type: "evidence_package"

  operation:
    title: "Short operation title"
    profile: "SDO-Normal"
    base_class: "Configure"
    overlays:
      - "Production"
      - "Customer-impacting"
    environment: "prod"
    service_or_asset: "service-name-or-asset-id"

  traceability:
    operational_spec_ref:
      uri: "sdo://specs/SDO-YYYYMMDD-0001"
      version: "v1.2"
      sha256: "..."
    execution_plan_ref:
      uri: "sdo://plans/SDO-YYYYMMDD-0001"
      version: "v1.0"
      sha256: "..."
    approval_record_ref:
      uri: "itsm://change/CHG000123"
      decision: "approved"
      approved_at_utc: "2026-05-22T14:00:00Z"
    validation_report_ref:
      uri: "sdo://validation/SDO-YYYYMMDD-0001"
      sha256: "..."
    review_record_ref:
      uri: "sdo://review/SDO-YYYYMMDD-0001"
      sha256: "..."

  actors:
    requester: "user-or-service-principal"
    approvers:
      - "approver-id"
    operators:
      - "operator-id"
    validators:
      - "validator-id"
    evidence_collector: "collector-id"

  timeline:
    created_at_utc: "2026-05-22T13:30:00Z"
    approved_at_utc: "2026-05-22T14:00:00Z"
    execution_started_at_utc: "2026-05-22T14:15:00Z"
    execution_finished_at_utc: "2026-05-22T14:25:00Z"
    validation_finished_at_utc: "2026-05-22T14:30:00Z"
    closed_at_utc: "2026-05-22T14:45:00Z"

  evidence_index:
    - evidence_item_id: "EVI-001"
      type: "technical_execution"
      planned_action_id: "PA-001"
      executed_action_id: "EA-001"
      sensitivity: "internal"
      sha256: "..."
      source_ref: "journal://host/unit/service?boot_id=...&from=...&to=..."
    - evidence_item_id: "EVI-002"
      type: "validation"
      validation_check_id: "VAL-001"
      sensitivity: "internal"
      sha256: "..."
      source_ref: "monitoring://check/abc123"

  integrity:
    package_manifest_sha256: "..."
    signed_by: "sdo-evidence-service-or-human"
    signature_ref: "sig://..."
    timestamp_authority_ref: "tsa://optional"
    immutable_storage_ref: "object-lock://bucket/key/version"

  retention:
    policy: "SDO-Normal-365d"
    retain_until_utc: "2027-05-22T00:00:00Z"
    legal_hold: false
    owner: "ops-governance"

  access_control:
    classification: "confidential"
    allowed_roles:
      - "sdo-auditor"
      - "sdo-operator"
      - "security-reviewer"

  closure:
    evidence_closure_gate: "passed"
    closed_by: "reviewer-id"
    closure_notes: "All planned actions mapped; validation passed; no unresolved deviations."
```

---

# 7. Estructura genérica de `sdo_evidence_item`

```yaml
sdo_evidence_item:
  schema_version: "1.0"
  evidence_item_id: "EVI-001"
  sdo_id: "SDO-YYYYMMDD-0001"

  classification:
    evidence_type: "technical_execution"
    sensitivity: "confidential"
    contains_secret: false
    contains_pii: false
    regulated_data: false
    redaction_status: "redacted"

  traceability:
    planned_action_id: "PA-001"
    executed_action_id: "EA-001"
    validation_check_id: null
    rollback_action_id: null
    deviation_id: null

  source:
    executor_type: "linux_cli"
    system: "hostname-or-system-id"
    source_kind: "journalctl"
    source_reference: "journal://host/unit/service?from=...&to=..."
    source_event_ids:
      - "event-or-line-id"
    source_time_window_utc:
      from: "2026-05-22T14:15:00Z"
      to: "2026-05-22T14:16:00Z"

  content:
    summary: "Service restarted and returned active state."
    excerpt_redacted: "systemctl restart service-x -> exit_code=0"
    artifact_ref: "object://evidence/redacted/EVI-001.txt"
    artifact_sha256: "..."

  integrity:
    collected_at_utc: "2026-05-22T14:20:00Z"
    collected_by: "collector-id"
    canonicalization: "utf8-normalized-json-v1"
    sha256: "..."
    previous_item_sha256: "optional-chain-link"
    signature_ref: "sig://optional"

  retention:
    retain_until_utc: "2027-05-22T00:00:00Z"
    storage_tier: "warm"
    deletion_policy: "after-retention-unless-legal-hold"
```

---

# 8. Cómo indexar evidencia sin duplicar logs pesados

**[SDO] Regla:** el Evidence Package debe ser un **índice verificable**, no un SIEM paralelo.

Guardar:

* sistema fuente;
* tipo de fuente;
* event IDs;
* request IDs;
* trace IDs;
* pipeline run IDs;
* commit SHA;
* artifact IDs;
* ventana temporal UTC;
* hash de extractos relevantes;
* query usada para recuperar el evento;
* retención del sistema fuente;
* propietario del sistema fuente.

No guardar por defecto:

* logs completos;
* dumps completos;
* tokens;
* secretos;
* `.env`;
* outputs con PII;
* capturas de pantalla con datos sensibles;
* transcripciones completas si contienen secretos.

Esto encaja con el enfoque de NIST de planificar generación, almacenamiento, acceso y disposición de logs, y con el hecho de que sistemas CI/CD como GitHub ya tienen sus propias políticas de retención de logs y artefactos. ([csrc.nist.gov][1])

---

# 9. Hashes, timestamps, actor identity e immutable references

**[SDO] Reglas mínimas:**

1. Todo `sdo_evidence_item` debe tener `sha256`.
2. Todo paquete debe tener `package_manifest_sha256`.
3. Todo timestamp debe guardarse en UTC.
4. Debe distinguirse:

   * `source_created_at`;
   * `collected_at`;
   * `approved_at`;
   * `executed_at`;
   * `validated_at`;
   * `closed_at`.
5. Actor identity debe ser estable:

   * humano: usuario corporativo;
   * sistema: service principal;
   * pipeline: workflow identity;
   * AI-assisted: humano aprobador + herramienta/modelo como auxiliar.
6. En SDO-Critical, el manifiesto debe ser firmado o almacenado en medio inmutable/WORM.
7. Si la fuente soporta integridad nativa, referenciarla. Ejemplo: digest files y firmas de CloudTrail. ([AWS Documentation][11])

---

# 10. Clasificación de sensibilidad

| Nivel | Nombre       | Ejemplos                                                     | Regla SDO                                                                            |
| ----- | ------------ | ------------------------------------------------------------ | ------------------------------------------------------------------------------------ |
| E0    | Public       | release note pública, status público                         | Puede incluirse si no revela arquitectura sensible.                                  |
| E1    | Internal     | IDs internos, nombres de servicios, timestamps               | Permitido en Evidence Package.                                                       |
| E2    | Confidential | hostnames, rutas internas, configs parciales                 | Permitido con control de acceso.                                                     |
| E3    | Restricted   | seguridad, IAM, firewall, audit logs sensibles               | Requiere acceso limitado y retención controlada.                                     |
| E4    | Regulated    | PII, datos financieros, salud, datos sujetos a regulación    | Evitar; si es imprescindible, minimizar y justificar.                                |
| E5    | Secret       | passwords, tokens, private keys, cookies, connection strings | Prohibido como evidencia. Guardar solo referencia al secret manager, nunca el valor. |

**[PA]** El principio de minimización de datos exige limitar datos personales a lo necesario; EDPS lo vincula al artículo 5 del GDPR. ([European Data Protection Supervisor][19]) **[SDO]** Por tanto, el paquete debe demostrar la operación, no maximizar la recolección.

---

# 11. Redaction / masking rules

**[SDO] Reglas obligatorias:**

1. Redactar siempre:

   * `Authorization`;
   * `Cookie`;
   * passwords;
   * tokens;
   * API keys;
   * private keys;
   * connection strings;
   * session IDs;
   * JWTs;
   * `.env`;
   * secrets en CLI output;
   * valores de variables sensibles.
2. Mantener contexto útil:

   * nombre de parámetro;
   * tipo de operación;
   * status code;
   * timestamp;
   * recurso lógico;
   * hash;
   * últimos 4 caracteres solo si ayuda a correlacionar.
3. Usar placeholders estables:

   * `[REDACTED_SECRET:sha256-prefix]`;
   * `[REDACTED_EMAIL:user-001]`;
   * `[REDACTED_TOKEN:token-003]`.
4. No guardar raw output si no se puede sanear.
5. Preferir evidencia estructurada a pantallazos.
6. En PowerShell, tener especial cuidado: Microsoft advierte que logging detallado puede capturar credenciales u otros datos sensibles. ([Microsoft Learn][15])

Ejemplo:

```text
Original:
curl -H "Authorization: Bearer eyJhbGciOi..." https://api.example.com/users/123

SDO evidence:
curl -H "Authorization: [REDACTED_SECRET:tok-001]" https://api.example.com/users/[REDACTED_ID:user-123]
response_status=200
request_id=req-abc123
```

---

# 12. Matriz executor type → evidence sources → risks → retention

| Executor type        | Evidence sources                                                                        | Riesgos de sensibilidad                                  | Retention notes                                     |
| -------------------- | --------------------------------------------------------------------------------------- | -------------------------------------------------------- | --------------------------------------------------- |
| Linux CLI            | shell history controlada, syslog, journalctl, auditd, script hash, exit code            | comandos con secretos, rutas internas, usuarios          | Guardar extractos; referenciar logs fuente.         |
| Windows / PowerShell | Event Viewer, PowerShell transcript, script block logging, module logging               | transcripts con credenciales, rutas, nombres de usuarios | Redacción obligatoria; acceso restringido.          |
| IaC                  | plan/diff, state reference, apply log, resource IDs, drift report                       | state con secretos o outputs sensibles                   | No copiar state completo; guardar hash/ref.         |
| GitOps               | commit SHA, PR, merge approval, controller sync status, drift/reconcile event           | repos privados, manifests con secretos                   | Referenciar commit y controller event.              |
| CI/CD                | workflow run, job logs, artifacts, environment approval, artifact attestation           | logs con secrets, artefactos pesados                     | Usar IDs, hashes y retención del pipeline.          |
| Cloud API / Console  | audit event ID, principal, API action, request ID, resource ID                          | IAM, tenant/account IDs, metadata sensible               | Preferir audit log nativo e immutable refs.         |
| Kubernetes           | audit logs, admission logs, deployment revision, rollout status                         | secrets, service account tokens, namespace sensitive     | No guardar manifests secretos.                      |
| Database             | migration ID, schema history, transaction ID, row count, checksum, backup ID            | datos de negocio, PII, SQL con valores reales            | Guardar metadata, no dumps.                         |
| API                  | request ID, correlation/trace ID, status, idempotency key, response hash                | payload con PII/secrets                                  | Guardar payload hash o extracto redactado.          |
| Manual / GUI         | screenshot redactado, screen recording restringido, operator notes, system audit log    | pantallas con PII, tokens, datos clientes                | Screenshot solo si no hay fuente mejor.             |
| AI-assisted          | prompt summary, decision trace, tool-call summary, human approvals, redacted transcript | prompt con secretos, overreliance, excessive agency      | Guardar resumen redactado + approvals, no secretos. |

---

# 13. Reglas de chain of custody

**[SDO] Cadena de custodia mínima:**

1. Cada item tiene `collected_by`, `collected_at_utc`, `source_ref`, `sha256`.
2. Cada modificación crea nueva versión; no se sobrescribe evidencia previa.
3. El paquete final se firma o hashea.
4. SDO-Critical usa almacenamiento inmutable o append-only.
5. Se registra quién accedió, exportó o eliminó evidencia.
6. Se separan roles:

   * operador;
   * aprobador;
   * recolector de evidencia;
   * validador;
   * auditor.
7. Se documentan brechas:

   * evidencia faltante;
   * fuente no disponible;
   * log rotado;
   * captura manual;
   * timestamp inconsistente.
8. Si se mueve evidencia entre sistemas, se registra:

   * origen;
   * destino;
   * hash antes/después;
   * actor;
   * timestamp;
   * motivo.

---

# 14. Manual / GUI evidence rules

**[SDO] Regla principal:** la evidencia manual es válida, pero es más débil que evidencia generada por sistema.

Orden de preferencia:

1. Audit log del sistema.
2. Event ID / request ID / transaction ID.
3. Export estructurado.
4. Screenshot redactado.
5. Nota del operador.
6. Testimonio o sign-off manual.

Reglas:

* Captura antes/después cuando aplique.
* Incluir timestamp visible o metadata externa.
* No capturar secretos, tokens, datos personales o pantallas completas innecesarias.
* Asociar screenshot a `planned_action_id` y `executed_action_id`.
* Añadir nota del operador: qué se hizo, dónde, resultado, limitaciones.
* Para SDO-Critical, una captura manual debe ser corroborada por otra fuente.

Ejemplo:

```yaml
manual_gui_evidence:
  evidence_item_id: "EVI-GUI-001"
  type: "manual_gui"
  action: "Changed service setting from disabled to enabled"
  screenshot_ref: "object://evidence/redacted/gui-before-after.png"
  screenshot_sha256: "..."
  operator_note: "Setting changed in admin console; no API/export available."
  corroborating_source: "audit://admin-console/event/987654"
  limitations: "Screenshot is supporting evidence, not primary audit source."
```

---

# 15. AI-assisted evidence rules

**[SDO] Reglas específicas:**

1. La IA no aprueba cambios por sí misma.
2. Toda acción de cambio debe mapear a una Planned Action aprobada.
3. Guardar:

   * resumen del prompt;
   * objetivo;
   * modelo/herramienta usada;
   * tool-call summary;
   * comandos propuestos;
   * comandos aprobados;
   * aprobador humano;
   * outputs redactados;
   * checkpoints de aprobación.
4. No guardar:

   * prompt completo si contiene secretos;
   * credenciales;
   * datos personales innecesarios;
   * dumps pegados por el operador.
5. Clasificar riesgo:

   * suggestion only;
   * assisted drafting;
   * tool-call capable;
   * autonomous execution prohibited unless policy-approved.
6. En SDO-Critical:

   * aprobación humana explícita;
   * revisión independiente;
   * logs de tool calls;
   * redacción previa al almacenamiento.

Esto responde a riesgos OWASP como sensitive information disclosure, excessive agency y overreliance. ([OWASP Foundation][18])

Ejemplo:

```yaml
ai_assisted_evidence:
  evidence_item_id: "EVI-AI-001"
  type: "ai_assisted_decision_trace"
  ai_role: "assistant_drafted_runbook"
  model_or_tool: "ai-assistant-name/version"
  prompt_summary: "Generate safe rollback steps for service config change."
  raw_prompt_stored: false
  redacted_transcript_ref: "object://evidence/redacted/ai-trace.txt"
  proposed_actions:
    - "PA-001"
    - "PA-002"
  human_approval_required: true
  approved_by: "operator-or-approver-id"
  tool_calls_executed:
    - executed_action_id: "EA-001"
      mapped_planned_action_id: "PA-001"
```

---

# 16. Evidence Closure Gate

**[SDO] El Evidence/Closure Gate pasa solo si:**

1. El `sdo_id` existe y es único.
2. La Operational Spec aprobada está referenciada y hasheada.
3. Cada Planned Action tiene estado:

   * executed;
   * skipped;
   * failed;
   * rolled_back;
   * superseded.
4. Cada Executed Action mapea a una Planned Action aprobada.
5. Toda desviación está clasificada:

   * harmless;
   * operational adjustment;
   * risk-impacting;
   * emergency deviation;
   * unauthorized.
6. Toda acción con cambio de estado tiene evidencia.
7. Toda validación tiene criterio esperado y resultado observado.
8. Rollback/recovery está:

   * probado;
   * ejecutado;
   * disponible;
   * o explícitamente marcado como no disponible con compensating controls.
9. No hay secretos en el paquete.
10. Sensibilidad y retención están definidas.
11. Hashes y referencias son resolubles.
12. El cierre tiene actor, timestamp y decisión.
13. En Emergency, existe Review/PIR post-ejecución.
14. En Pipeline-Backed, commit, run ID, artifact ID y approval están enlazados.

Resultado posible:

```yaml
evidence_closure_gate:
  status: "passed"
  blocking_findings: []
  warnings:
    - "Manual screenshot used as secondary evidence."
  closed_by: "reviewer-id"
  closed_at_utc: "2026-05-22T14:45:00Z"
```

---

# 17. Retention y access control

**[SDO] Retención sugerida, ajustable por regulación:**

| Perfil              | Retención base                                                                                             |
| ------------------- | ---------------------------------------------------------------------------------------------------------- |
| SDO-Lightweight     | 30–90 días                                                                                                 |
| SDO-Normal          | 180–365 días                                                                                               |
| SDO-Critical        | 1–7 años, según compliance                                                                                 |
| SDO-Emergency       | igual que Critical si afecta seguridad, datos o disponibilidad crítica                                     |
| SDO-Pipeline-Backed | al menos durante vida útil del release; logs según plataforma; artefactos críticos con retención extendida |

Reglas:

* Retention del paquete no debe asumir que el sistema fuente retendrá logs igual de tiempo.
* Si el sistema fuente rota logs antes, exportar extracto mínimo hasheado.
* Si hay legal hold, bloquear eliminación.
* Acceso por least privilege:

  * operadores ven evidencia operativa;
  * auditores ven paquete;
  * seguridad ve evidencia restringida;
  * negocio ve resumen no técnico.
* Separar evidencia técnica sensible del resumen ejecutivo.

---

# 18. Ejemplo agnóstico: cambio de configuración

```yaml
operation_summary:
  sdo_id: "SDO-20260522-0012"
  profile: "SDO-Normal"
  base_class: "Configure"
  intent: "Enable feature flag for service X in production."

planned_actions:
  - id: "PA-001"
    description: "Update feature flag from false to true."
    preview_available: true
    rollback: "Set feature flag back to false."

executed_actions:
  - id: "EA-001"
    planned_action_id: "PA-001"
    executor_type: "api"
    request_id: "req-123"
    status: "success"

evidence:
  - id: "EVI-001"
    type: "approval"
    source_ref: "itsm://change/CHG000456"
  - id: "EVI-002"
    type: "technical_execution"
    source_ref: "api-gateway://logs?request_id=req-123"
  - id: "EVI-003"
    type: "validation"
    source_ref: "monitoring://synthetic-check/check-789"
  - id: "EVI-004"
    type: "rollback"
    summary: "Rollback command tested in staging; production rollback available."

closure:
  validation_status: "passed"
  deviations: []
  evidence_closure_gate: "passed"
```

---

# 19. Anti-patrones a evitar

1. Guardar todos los logs “por si acaso”.
2. Copiar secretos dentro del Evidence Package.
3. Usar screenshots como única prueba cuando existe audit log.
4. Aprobar por chat sin actor, timestamp ni alcance.
5. Ejecutar acciones no planificadas y no clasificarlas como desviación.
6. Cerrar con “funciona” sin criterio de validación.
7. No guardar ventana temporal ni timezone.
8. No diferenciar evidencia técnica de evidencia de negocio.
9. Usar AI transcripts completos con datos sensibles.
10. Confiar en logs de pipeline sin revisar retención.
11. Sobrescribir evidencia en vez de versionarla.
12. No indicar quién recolectó la evidencia.
13. No registrar por qué rollback no existe.
14. Duplicar ITSM, SIEM, Git, pipeline o cloud logs en vez de referenciarlos.
15. Usar hashes de archivos ya redactados sin indicar que no son el original.

---

# 20. Definición final propuesta

**SDO Evidence & Auditability** es el submodelo de SDO que define cómo una operación deja una prueba mínima, verificable, protegida y trazable de su ciclo completo:

```text
Intent
  → Spec evidence
  → Approval evidence
  → Plan / Preview evidence
  → Execution evidence
  → Validation evidence
  → Rollback / Recovery evidence
  → Review / Closure evidence
```

Su regla central:

```text
An SDO operation is auditable when every executed state-changing action
can be traced to an approved planned action, supported by sufficient
technical evidence, validated against explicit success criteria, protected
against tampering, and closed without exposing secrets or unnecessary data.
```

Esta formulación es propuesta SDO; se apoya en prácticas aceptadas de ITIL/ITSM, SRE, GitOps/IaC, CI/CD, cloud audit, logging, privacidad y seguridad, pero no depende de una herramienta concreta.

[1]: https://csrc.nist.gov/pubs/sp/800/92/r1/ipd "SP 800-92 Rev. 1, Cybersecurity Log Management Planning Guide | CSRC"
[2]: https://www.peoplecert.org/Frameworks-Professionals/ITIL-framework "www.peoplecert.org"
[3]: https://itsm.tools/change-enablement/ "Change Enablement – Change Management in ITIL 4"
[4]: https://sre.google/sre-book/postmortem-culture/ "Google SRE - Blameless Postmortem for System Resilience"
[5]: https://opengitops.dev/ "Home | OpenGitOps"
[6]: https://developer.hashicorp.com/terraform/cli/commands/plan "terraform plan command reference | Terraform | HashiCorp Developer"
[7]: https://docs.github.com/en/actions/reference/workflows-and-actions/deployments-and-environments "Deployments and environments - GitHub Docs"
[8]: https://docs.github.com/actions/managing-workflow-runs/using-workflow-run-logs "Using workflow run logs - GitHub Docs"
[9]: https://docs.github.com/en/organizations/managing-organization-settings/configuring-the-retention-period-for-github-actions-artifacts-and-logs-in-your-organization "Configuring the retention period for GitHub Actions artifacts and logs in your organization - GitHub Docs"
[10]: https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final "SP 800-53 Rev. 5, Security and Privacy Controls for Information Systems and Organizations | CSRC"
[11]: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-log-file-validation-intro.html "Validating CloudTrail log file integrity - AWS CloudTrail"
[12]: https://docs.cloud.google.com/logging/docs/audit "Cloud Audit Logs overview  |  Cloud Logging  |  Google Cloud Documentation"
[13]: https://learn.microsoft.com/en-us/azure/azure-monitor/platform/activity-log "Activity Log in Azure Monitor - Azure Monitor | Microsoft Learn"
[14]: https://man7.org/linux/man-pages/man8/auditd.8.html "auditd(8) - Linux manual page"
[15]: https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging_windows?view=powershell-7.6 "about_Logging_Windows - PowerShell | Microsoft Learn"
[16]: https://documentation.red-gate.com/fd/flyway-schema-history-table-273973417.html "Flyway schema history table - Redgate Flyway - Product Documentation"
[17]: https://www.w3.org/TR/trace-context/ "Trace Context"
[18]: https://owasp.org/www-project-top-10-for-large-language-model-applications/ "OWASP Top 10 for Large Language Model Applications | OWASP Foundation"
[19]: https://www.edps.europa.eu/data-protection/data-protection/glossary/d_en?utm_source=chatgpt.com "D | European Data Protection Supervisor"
