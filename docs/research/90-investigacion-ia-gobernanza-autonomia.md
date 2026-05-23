> Research note — non-normative.
> This document is background material and may be superseded by current SDO core/reference documentation.
> Current normative docs start at [../start-here.md](../start-here.md).

## Modelo propuesto: **SDO AI Governance & Autonomy**

La regla central debería ser:

> **La IA en SDO puede asistir, proponer, resumir, simular y ejecutar bajo límites explícitos; pero no debe convertirse por defecto en autoridad operacional sobre producción.**

Esto encaja con NIST AI RMF, que organiza la gestión de riesgo de IA en funciones de **Govern, Map, Measure, Manage**, y con su perfil para IA generativa, que extiende esos controles a riesgos propios de GenAI. ([NIST][1])
También encaja con el EU AI Act, cuyo enfoque de supervisión humana exige que los sistemas de alto riesgo permitan intervención, supervisión y reducción de riesgos razonablemente previsibles. ([eur-lex.europa.eu][2])
OWASP refuerza que los riesgos clave en LLMs incluyen **prompt injection**, **sensitive information disclosure**, **excessive agency**, **insecure output handling** y **overreliance**, todos directamente relevantes para operaciones TI con herramientas. ([OWASP Gen AI Security Project][3])

---

# 1. Principio base

## Propuesta SDO

SDO debería tratar la IA como un **actor operacional gobernado**, no como un operador implícitamente confiable.

La IA nunca debería tener autoridad por sí sola sobre:

* cambios no aprobados;
* acciones irreversibles;
* secretos;
* sistemas productivos críticos;
* decisiones de contención de seguridad de alto impacto;
* destrucción, rotación, revocación o modificación masiva;
* bypass de gates SDO.

La IA debe operar dentro de una relación explícita:

```text
AI proposes ≠ Human approves ≠ Executor performs ≠ SDO validates
```

Incluso cuando la IA ejecuta, la ejecución debe estar subordinada a:

```text
Approved Spec + Approved Plan + Allowed Tools + Scope + Identity + Evidence
```

---

# 2. Roles de IA en SDO

| Rol IA                | Descripción                                                | Riesgo base | Recomendación                                                  |
| --------------------- | ---------------------------------------------------------- | ----------: | -------------------------------------------------------------- |
| `researcher`          | Investiga documentación, patrones, restricciones, riesgos. |  Bajo/medio | Permitido ampliamente.                                         |
| `spec_drafter`        | Redacta Operational Spec inicial.                          |       Medio | Permitido, requiere revisión humana.                           |
| `risk_classifier`     | Sugiere clase, perfil, riesgo, reversibilidad.             |  Medio/alto | Permitido como recomendación, no como decisión final.          |
| `runbook_drafter`     | Propone pasos, validaciones y rollback.                    |       Medio | Permitido con revisión.                                        |
| `reviewer`            | Revisa Spec, Plan, Runbook, evidencias.                    |  Bajo/medio | Permitido; no sustituye aprobación.                            |
| `simulator`           | Simula impactos o dry-run lógico.                          |       Medio | Permitido si no toca producción.                               |
| `execution_assistant` | Guía al humano durante ejecución.                          |  Medio/alto | Permitido con límites.                                         |
| `guarded_executor`    | Ejecuta tool calls aprobados y acotados.                   |        Alto | Solo con controles fuertes.                                    |
| `autonomous_executor` | Ejecuta sin aprobación humana por acción.                  |    Muy alto | Solo para operaciones estándar, reversibles y de bajo impacto. |
| `evidence_summarizer` | Resume logs, resultados, evidencias.                       |       Medio | Permitido con trazabilidad y redacción de secretos.            |
| `incident_copilot`    | Asiste en diagnóstico, triage y coordinación.              |        Alto | Permitido; contención destructiva requiere humano.             |

---

# 3. Niveles de autonomía SDO-AI

## Propuesta propia

| Nivel | Nombre                              | Qué permite                                                                                      | Qué no permite                                                                  |
| ----- | ----------------------------------- | ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------- |
| `A0`  | No AI / manual                      | Humanos realizan redacción, planificación, ejecución, validación y evidencia.                    | Cualquier intervención IA.                                                      |
| `A1`  | AI drafts only                      | Redacción, resumen y documentación bajo revisión humana.                                         | Decisiones, aprobaciones, comandos o consultas en vivo.                         |
| `A2`  | AI plans                            | Recomendaciones, clasificación, planes, checks y propuestas de rollback.                         | Ejecutar comandos o llamar APIs.                                                |
| `A3`  | AI proposes commands                | La IA propone comandos, SQL, API calls o pasos GUI para aprobación y ejecución humana.           | Ejecutar los comandos propuestos.                                               |
| `A4`  | AI executes read-only checks        | La IA ejecuta diagnósticos y validaciones no mutativas, acotadas y registradas.                  | Mutar estado, leer datos sensibles sin aprobación o ampliar alcance.            |
| `A5`  | AI executes bounded changes with approval | La IA ejecuta cambios aprobados dentro de un contrato estrecho, con rollback, validación y evidencia. | Cambios productivos no aprobados, elevación de privilegios o acciones destructivas fuera de política. |
| `A6`  | Policy-bound autonomous operations  | La IA ejecuta operaciones estándar sin aprobación humana por corrida bajo política formal preaprobada. | Operaciones críticas, destructivas, sensibles, privilegiadas o novedosas sin aprobación humana. |

Cuando la IA no debe usarse en una operación o contexto, debe indicarse explícitamente como `AI prohibited` o `AI use forbidden by policy`, no como nivel de autonomía.

---

# 4. Matriz: SDO Profile → autonomía IA permitida

| SDO Profile           | Autonomía máxima recomendada | Roles permitidos                                                                                       |
| --------------------- | ---------------------------: | ------------------------------------------------------------------------------------------------------ |
| `SDO-Lightweight`     |                         `A4` | researcher, drafter, reviewer, simulator, execution_assistant, guarded_executor de solo lectura o bajo riesgo |
| `SDO-Normal`          |                  `A4` / `A5` | A5 solo con aprobación humana, contrato acotado, rollback/recovery, validación y evidencia                  |
| `SDO-Critical`        |                  `A2` / `A3` | researcher, drafter, reviewer, simulator; tool use solo aprobado                                            |
| `SDO-Emergency`       |                  `A2` / `A3` | incident_copilot, evidence_summarizer, execution_assistant; autoridad de emergencia sigue siendo humana     |
| `SDO-Pipeline-Backed` |                  `A5` / `A6` | A6 solo bajo política formal, monitoreo, stop automático, auditoría y revisión periódica                    |

Regla práctica:

```text
A mayor criticidad, menor autonomía IA directa.
A mayor automatización madura y prevalidada, mayor autonomía posible.
```

---

# 5. Matriz: riesgo / reversibilidad / sensibilidad / privilegio → supervisión humana

| Condición                                                                | Supervisión mínima                                                     |
| ------------------------------------------------------------------------ | ---------------------------------------------------------------------- |
| Bajo riesgo + reversible `R5-R7` + sin datos sensibles + bajo privilegio | `human-on-the-loop` o `A4/A5`                                          |
| Medio riesgo + reversible `R3-R4`                                        | `human-in-the-loop`, aprobación por plan o lote                        |
| Alto riesgo + reversibilidad parcial `R1-R2`                             | aprobación humana obligatoria por acción crítica                       |
| Sin rollback `R0`                                                        | aprobación explícita + aceptación de riesgo + mitigation/recovery plan |
| Datos sensibles                                                          | revisión humana + redacción + mínimo acceso                            |
| Acceso privilegiado admin/root/cloud-owner                               | aprobación humana obligatoria                                          |
| Producción crítica                                                       | approval checkpoint obligatorio                                        |
| Acción destructiva                                                       | humano obligatorio                                                     |
| Seguridad: aislamiento, bloqueo, revocación, rotación                    | humano obligatorio salvo playbook preaprobado de emergencia            |
| Operación read-only sensible                                             | permitido con redacción, logging y least privilege                     |
| Tool call no previsto en plan                                            | bloquear o volver a Approval Gate                                      |

Esto aplica el principio de supervisión humana proporcional al riesgo, coherente con el enfoque del EU AI Act y con los riesgos OWASP de agencia excesiva y exposición de información sensible. ([artificialintelligenceact.eu][4])

---

# 6. AI Approval Checkpoints dentro de SDO

## Integración con gates existentes

| Gate SDO                      | Control IA añadido                                                          |
| ----------------------------- | --------------------------------------------------------------------------- |
| `Spec Gate`                   | Declarar si se usó IA, modelo, rol y límites.                               |
| `Risk/Profile Gate`           | Determinar autonomía máxima permitida.                                      |
| `Preview/Plan Gate`           | Validar que las acciones IA propuestas mapean a Planned Actions.            |
| `Approval Gate`               | Aprobar alcance, herramientas, identidad y autonomía.                       |
| `Pre-Execution Gate`          | Verificar permisos, secretos, entorno, dry-run y tool policy.               |
| `Continue/Stop Gate`          | La IA puede recomendar continuar/parar; humano decide si hay riesgo alto.   |
| `Rollback Gate`               | IA puede sugerir rollback; ejecución depende del nivel A/R.                 |
| `Validation Gate`             | IA puede resumir evidencias, pero no autocertificar sin checks.             |
| `Evidence/Closure Gate`       | Registrar prompts, decisiones, tool calls, aprobaciones y outputs saneados. |
| `Automation Eligibility Gate` | Evaluar si puede pasar de `A3` a `A4/A5`.                                   |

---

# 7. Acciones que requieren revisión humana obligatoria

## Regla SDO propuesta

La IA no debe ejecutar sin aprobación humana previa cuando la acción:

1. modifica producción;
2. afecta disponibilidad;
3. toca datos personales, secretos, credenciales o claves;
4. cambia IAM/RBAC/permisos;
5. elimina, destruye, purga o rota recursos;
6. modifica red, firewall, rutas, DNS, certificados o balanceadores;
7. cambia backups, retención o cifrado;
8. ejecuta comandos con privilegios elevados;
9. contiene SQL `UPDATE`, `DELETE`, `DROP`, `TRUNCATE`, migraciones o scripts masivos;
10. contiene acciones de seguridad como aislar hosts, bloquear usuarios, revocar tokens o eliminar artefactos;
11. no tiene rollback verificable;
12. se basa en información no confiable o input externo susceptible a prompt injection.

---

# 8. Cómo evitar prompt injection, tool misuse y excessive agency

OWASP clasifica prompt injection y excessive agency como riesgos principales en aplicaciones LLM; excessive agency ocurre cuando el LLM puede causar acciones dañinas por permisos, herramientas o autonomía excesiva. ([OWASP Gen AI Security Project][3])

## Reglas SDO

```text
Untrusted content must never become instruction.
```

Controles:

* separar instrucciones del sistema, Spec aprobada y datos externos;
* marcar logs, tickets, emails, páginas web y outputs CLI como **untrusted input**;
* no permitir que la IA modifique su propia policy;
* no permitir tool calls derivados solo de contenido externo;
* usar allowlist de herramientas;
* limitar argumentos permitidos;
* exigir preview antes de ejecución;
* validar output antes de pasarlo a shells, APIs o SQL;
* bloquear comandos ambiguos o expansivos;
* aplicar timeouts, cuotas y rate limits;
* registrar tool calls y decisiones;
* usar sandbox para pruebas;
* usar credenciales efímeras y de mínimo privilegio.

---

# 9. Least privilege para agentes IA

La IA no debería “heredar” el acceso completo del operador humano.

## Modelo recomendado

```text
Human identity ≠ AI runtime identity ≠ Executor identity
```

Controles:

| Área         | Regla                                                               |
| ------------ | ------------------------------------------------------------------- |
| Identidad    | La IA usa una identidad técnica propia, no credenciales personales. |
| Permisos     | Solo permisos necesarios para el rol y operación.                   |
| Alcance      | Scope por SDO ID, entorno, recurso, ventana temporal.               |
| Duración     | Credenciales temporales.                                            |
| Herramientas | Allowlist por operación.                                            |
| Red          | Acceso segmentado.                                                  |
| Secretos     | Nunca visibles al modelo salvo necesidad justificada y redacción.   |
| Logs         | Guardar hashes/referencias cuando haya secretos.                    |
| Elevación    | Prohibida sin Approval Gate.                                        |

---

# 10. Estructura genérica `sdo_ai_policy`

```yaml
sdo_ai_policy:
  policy_id: "sdo-ai-policy-standard"
  version: "1.0"

  ai_allowed: true
  max_autonomy_level: "A3"

  allowed_roles:
    - researcher
    - spec_drafter
    - risk_classifier
    - runbook_drafter
    - reviewer
    - simulator
    - execution_assistant
    - evidence_summarizer

  prohibited_roles:
    - autonomous_executor

  model_constraints:
    approved_models:
      - "approved-enterprise-llm"
    external_models_allowed: false
    training_on_operational_data_allowed: false

  data_handling:
    secrets_to_model: "forbidden"
    sensitive_data_to_model: "redacted_only"
    pii_to_model: "redacted_or_approved"
    log_raw_prompts: true
    log_redacted_prompts: true
    store_full_outputs: "only_if_no_sensitive_data"
    retention_days: 90

  tool_use:
    tool_calls_allowed: true
    approval_before_tool_call: true
    allowlisted_tools:
      - "read_logs"
      - "query_metrics"
      - "dry_run_plan"
    denied_tools:
      - "delete_resource"
      - "modify_iam"
      - "rotate_secret"
      - "drop_database"

  execution_boundaries:
    environments_allowed:
      - dev
      - staging
    production_allowed: false
    max_privilege: "read_only"
    network_scope:
      - "approved_resources_only"

  approval_rules:
    require_human_for:
      - production_change
      - privileged_action
      - destructive_action
      - non_previewable_action
      - no_rollback
      - sensitive_data_access
      - security_containment
      - deviation_from_plan

  evidence_rules:
    record_ai_role: true
    record_model_id: true
    record_prompt_hash: true
    record_redacted_prompt: true
    record_tool_calls: true
    record_human_approvals: true
    record_policy_decision: true
    record_output_hash: true

  emergency_rules:
    emergency_max_autonomy: "A3"
    allow_fast_track: true
    require_post_incident_review: true
    require_retrospective_approval_record: true
```

---

# 11. Estructura genérica `sdo_ai_decision_trace`

```yaml
sdo_ai_decision_trace:
  trace_id: "aitrace-2026-000123"
  sdo_id: "SDO-2026-0017"

  timestamp_utc: "2026-05-22T14:30:00Z"

  ai_actor:
    role: "runbook_drafter"
    autonomy_level: "A2"
    model_id: "approved-enterprise-llm"
    session_id: "session-abc123"

  input_context:
    operational_spec_ref: "spec.yaml#sha256:..."
    execution_plan_ref: "plan.yaml#sha256:..."
    evidence_refs:
      - "metrics_snapshot.json#sha256:..."
    prompt_redacted: "Generate a rollback-aware runbook for..."
    prompt_hash: "sha256:..."

  decision:
    type: "recommendation"
    summary: "Suggested restarting service after config validation."
    confidence: "medium"
    assumptions:
      - "Service restart is supported by maintenance window."
    risks_identified:
      - "Temporary connection interruption."
    alternatives:
      - "Reload config without restart."
      - "Rollback config file."

  tool_calls:
    - tool: "read_logs"
      approved: true
      approved_by: "operator@example.com"
      arguments_redacted:
        host: "srv-app-01"
        path: "/var/log/app/app.log"
      result_ref: "tool-result-001#sha256:..."

  human_oversight:
    proposed_by: "ai"
    reviewed_by: "operator@example.com"
    approved_by: "change-owner@example.com"
    executed_by: "human"
    approval_timestamp_utc: "2026-05-22T14:35:00Z"

  policy_evaluation:
    policy_id: "sdo-ai-policy-standard"
    allowed: true
    reason: "A2 advisory role allowed for SDO-Normal."
    blocked_items:
      - "Direct production restart by AI not allowed."

  outcome:
    accepted: true
    modifications_by_human:
      - "Added pre-check for active sessions."
    final_action_refs:
      - "planned_action:PA-003"
```

---

# 12. Reglas para approvals before tool calls

## Regla general

```text
No AI tool call may execute unless it maps to an approved Planned Action or an approved diagnostic permission.
```

## Clasificación de tool calls

| Tipo de tool call           | Ejemplos                                            | Aprobación                                                  |
| --------------------------- | --------------------------------------------------- | ----------------------------------------------------------- |
| Read-only bajo riesgo       | consultar métricas públicas internas, listar estado | preaprobación por policy                                    |
| Read-only sensible          | logs con PII, IAM, auditoría, seguridad             | aprobación humana o redacción previa                        |
| Preview/dry-run             | terraform plan, kubectl diff, SQL explain           | permitido si no cambia estado                               |
| State-changing reversible   | restart controlado, scale up/down limitado          | aprobación humana o `A4` si bounded                         |
| State-changing irreversible | delete, purge, drop, revoke masivo                  | humano obligatorio                                          |
| Privileged security action  | aislar host, bloquear cuenta, rotar secretos        | humano obligatorio salvo playbook de emergencia preaprobado |
| Unknown/unplanned           | cualquier acción fuera del plan                     | bloquear y volver a gate                                    |

---

# 13. Reglas para AI-assisted evidence

Microsoft, por ejemplo, documenta audit logs para interacciones y actividades de Copilot, lo que refuerza que las intervenciones de IA deben ser trazables como parte del sistema de auditoría. ([Microsoft Learn][5])

## Reglas SDO

La IA puede:

* resumir logs;
* clasificar evidencias;
* detectar inconsistencias;
* generar Validation Report preliminar;
* comparar resultado esperado vs resultado real.

La IA no debe:

* inventar evidencias;
* reemplazar evidencias primarias;
* ocultar desviaciones;
* validar su propia ejecución sin checks externos;
* guardar secretos en claro.

El Evidence Package debería distinguir:

```text
Primary Evidence: logs, outputs, snapshots, tickets, commits.
AI-Derived Evidence: summaries, classifications, recommendations.
Human Validation: approvals, acceptance, overrides.
```

---

# 14. IA en emergencia/incidente

En incidentes, la IA es útil como **copiloto de velocidad**, no como comandante.

AIOps suele usarse para correlación de alertas, diagnóstico, predicción y respuestas automatizadas; la práctica recomendada es empezar con human-in-the-loop y graduar flujos específicos a automatización cuando existan guardrails y confianza operacional. ([resolve.io][6])

## Reglas SDO-Emergency

Permitido:

* resumir timeline;
* correlacionar alertas;
* sugerir hipótesis;
* consultar runbooks;
* preparar comunicación;
* generar comandos candidatos;
* recolectar evidencia;
* recomendar rollback o contención.

Restringido:

* ejecutar contención destructiva;
* borrar indicadores;
* rotar credenciales sin aprobación;
* aislar segmentos completos;
* modificar IAM;
* apagar producción;
* eliminar datos;
* cerrar incidente sin revisión humana.

Excepción posible:

```text
A4 emergency bounded autonomy
```

Solo si existe:

* playbook preaprobado;
* blast radius limitado;
* acción reversible o compensable;
* monitoreo activo;
* stop control;
* post-incident review obligatorio.

---

# 15. Operaciones read-only sensibles

No todas las operaciones read-only son seguras.

Leer puede ser riesgoso cuando expone:

* secretos;
* PII;
* datos médicos/financieros;
* credenciales;
* configuración de seguridad;
* vulnerabilidades;
* logs de autenticación;
* información contractual o legal.

## Regla SDO

```text
Read-only does not mean low-risk.
```

Para `Observe` y `Analyze` sensibles:

* usar redacción automática;
* limitar query;
* limitar columnas/campos;
* evitar dumps completos;
* preferir agregados;
* registrar propósito;
* registrar quién solicitó;
* registrar qué datos vio la IA;
* no usar datos sensibles para entrenamiento;
* aplicar TTL a evidencias.

---

# 16. Auditoría de resultados generados por IA

Un resultado IA en SDO debe auditarse como **propuesta derivada**, no como hecho.

## Criterios mínimos

| Elemento                        | Debe registrarse                            |
| ------------------------------- | ------------------------------------------- |
| Quién propuso                   | IA, humano o sistema externo                |
| Quién aprobó                    | humano, grupo, policy automática            |
| Quién ejecutó                   | humano, pipeline, agente, script            |
| Qué contexto usó                | Spec, plan, logs, docs, tickets             |
| Qué modelo                      | proveedor/modelo/version si aplica          |
| Qué prompt                      | redacted prompt + hash                      |
| Qué salida                      | output redacted + hash                      |
| Qué tool calls                  | herramienta, argumentos saneados, resultado |
| Qué policy aplicó               | allow/deny, razón                           |
| Qué desviaciones hubo           | clasificación y aprobación                  |
| Qué evidencia primaria respalda | referencias verificables                    |

---

# 17. Anti-patrones a evitar

| Anti-patrón                        | Por qué es peligroso                               |
| ---------------------------------- | -------------------------------------------------- |
| “AI runs prod”                     | Convierte asistencia en autoridad operacional.     |
| “El modelo decidió”                | Borra responsabilidad humana.                      |
| “Prompt como control de seguridad” | El prompt no sustituye RBAC, sandbox ni approvals. |
| “Read-only es seguro”              | Puede filtrar secretos o datos sensibles.          |
| “Aprobación genérica permanente”   | Deriva en agencia excesiva.                        |
| “Tool access amplio”               | Aumenta blast radius.                              |
| “Logs completos al modelo”         | Riesgo de exposición y retención indebida.         |
| “AI valida su propio resultado”    | Conflicto de interés operacional.                  |
| “Emergency bypass sin review”      | Normaliza atajos peligrosos.                       |
| “No registrar prompts”             | Rompe auditoría y explicabilidad.                  |
| “Copiar comandos IA sin preview”   | Riesgo directo de ejecución no entendida.          |

---

# 18. Ejemplos agnósticos

## Ejemplo 1: reinicio de servicio no crítico

```yaml
operation:
  base_class: Maintain
  profile: SDO-Lightweight
  reversibility: R6
  sensitivity: low
  privilege: service_admin
  ai_autonomy: A4
```

Permitido:

* IA revisa logs;
* propone reinicio;
* ejecuta restart si está en ventana aprobada;
* valida healthcheck;
* adjunta evidencia.

No permitido:

* cambiar configuración;
* reiniciar otros servicios;
* escalar privilegios.

---

## Ejemplo 2: cambio IAM en producción

```yaml
operation:
  base_class: Access
  profile: SDO-Critical
  reversibility: R2
  sensitivity: high
  privilege: iam_admin
  ai_autonomy: A2
```

Permitido:

* IA redacta plan;
* detecta riesgos;
* sugiere comandos;
* revisa policy.

No permitido:

* aplicar cambios;
* generar credenciales reales;
* aprobar permisos;
* modificar roles.

---

## Ejemplo 3: incidente de seguridad

```yaml
operation:
  base_class: Remediate
  profile: SDO-Emergency
  reversibility: R3
  sensitivity: high
  privilege: security_admin
  ai_autonomy: A3
```

Permitido:

* IA correlaciona alertas;
* sugiere contención;
* prepara comunicación;
* resume timeline.

Requiere humano:

* aislar host;
* bloquear usuario;
* revocar tokens;
* borrar artefactos;
* cerrar incidente.

---

# 19. Separación de responsabilidades

## Modelo RACI mínimo

| Acción            | IA          | Operador                | Aprobador   | Sistema/Pipeline        |
| ----------------- | ----------- | ----------------------- | ----------- | ----------------------- |
| Proponer Spec     | Responsible | Accountable             | Consulted   | —                       |
| Clasificar riesgo | Consulted   | Responsible             | Accountable | —                       |
| Redactar Runbook  | Responsible | Accountable             | Consulted   | —                       |
| Aprobar ejecución | No          | Consulted               | Accountable | —                       |
| Ejecutar acción   | Optional    | Responsible/Accountable | Consulted   | Responsible si pipeline |
| Validar resultado | Consulted   | Responsible             | Accountable | Provides evidence       |
| Cerrar operación  | Consulted   | Responsible             | Accountable | Archives evidence       |

Regla corta:

```text
AI may be Responsible for drafting.
AI should not be Accountable for operational risk.
```

---

# 20. Prácticas aceptadas vs propuesta SDO

## Respaldado por marcos existentes

* Gestión de riesgo de IA con gobierno, medición y gestión continua: NIST AI RMF. ([NIST][1])
* Supervisión humana para reducir riesgos, especialmente en sistemas de alto impacto: EU AI Act. ([eur-lex.europa.eu][2])
* Riesgos específicos de LLMs: prompt injection, sensitive information disclosure, excessive agency, misinformation, insecure output handling: OWASP. ([OWASP Gen AI Security Project][3])
* AI governance como sistema de gestión organizacional: ISO/IEC 42001. ([ISO][7])
* Uso de IA en operaciones como triage, correlación, diagnóstico, resumen y automatización gradual con guardrails: prácticas AIOps actuales. ([resolve.io][6])

## Propuesta propia para SDO

* Niveles `A0-A6`.
* Matriz SDO Profile → AI autonomy.
* `sdo_ai_policy`.
* `sdo_ai_decision_trace`.
* Regla “AI proposes ≠ Human approves ≠ Executor performs”.
* Integración directa de autonomía IA con gates SDO.
* Relación autonomía/riesgo/reversibilidad/sensibilidad/privilegio.
* Prohibición explícita de convertir SDO en “AI runs prod”.

---

# Conclusión

El modelo recomendado para SDO no debería ser “IA sí” o “IA no”, sino:

```text
AI role + autonomy level + approved scope + least privilege + evidence + human accountability
```

La fórmula más segura sería:

```text
IA como copiloto por defecto.
IA como ejecutor solo bajo policy.
IA autónoma solo para operaciones estándar, reversibles, de bajo riesgo y preaprobadas.
IA prohibida cuando no puede garantizarse control, trazabilidad, rollback o supervisión humana efectiva.
```

[1]: https://www.nist.gov/itl/ai-risk-management-framework?utm_source=chatgpt.com "AI Risk Management Framework | NIST"
[2]: https://eur-lex.europa.eu/eli/reg/2024/1689/oj/eng?utm_source=chatgpt.com "Regulation - EU - 2024/1689 - EN - EUR-Lex - European Union"
[3]: https://genai.owasp.org/llm-top-10/?utm_source=chatgpt.com "LLMRisks Archive - OWASP Gen AI Security Project"
[4]: https://artificialintelligenceact.eu/article/14/?utm_source=chatgpt.com "Article 14: Human Oversight | EU Artificial Intelligence Act"
[5]: https://learn.microsoft.com/en-us/purview/audit-copilot?utm_source=chatgpt.com "Audit logs for Copilot and AI applications"
[6]: https://resolve.io/blog/practical-guide-automating-incident-response-with-aiops?utm_source=chatgpt.com "Automate IT Incident Response with AIOps: A Practical Guide"
[7]: https://www.iso.org/artificial-intelligence/ai-management-systems?utm_source=chatgpt.com "AI management systems: What businesses need to know"
