> Research note — non-normative.
> This document is background material and may be superseded by current SDO core/reference documentation.
> Current normative docs start at [../start-here.md](../start-here.md).

A continuación separo **prácticas aceptadas** de **propuesta SDO**.

## 0. Base investigada

**Prácticas aceptadas:**

SRE recomienda supervisar los rollouts y, si aparece comportamiento inesperado, **hacer rollback primero y diagnosticar después** para reducir el tiempo de recuperación. También usa canaries para exponer cambios a una fracción pequeña antes de completar el despliegue. ([Google SRE][1])

DORA mide la inestabilidad mediante indicadores como **change failure rate**, es decir, cambios que requieren intervención inmediata, normalmente rollback, hotfix o remediación rápida. ([dora.dev][2])

GitOps se basa en estado deseado declarativo, versionado, inmutable y reconciliado automáticamente. Por eso, en GitOps el rollback correcto suele ser **revertir el estado deseado en Git**, no tocar el runtime manualmente y dejar drift. ([Open GitOps][3])

Kubernetes soporta rollback de Deployments con `kubectl rollout undo`, incluso a revisiones concretas, pero eso no revierte automáticamente datos persistentes, PVCs, dependencias externas ni cambios fuera del objeto controlado. ([Kubernetes][4])

En IaC, Terraform/OpenTofu dependen de un state que mapea recursos reales con la configuración. HCP Terraform permite restaurar una versión anterior del state, y OpenTofu recomienda versionar, cifrar y compartir de forma segura el state; aun así, restaurar state no equivale siempre a restaurar infraestructura real. ([HashiCorp Developer][5])

En bases de datos, PITR depende de backups base y secuencia continua de logs/WAL; PostgreSQL advierte que para recuperar correctamente se necesita una secuencia continua de WAL archivados, y SQL Server recomienda probar la estrategia restaurando backups. ([PostgreSQL][6])

En APIs distribuidas, las idempotency keys permiten reintentar sin duplicar efectos, y las transacciones compensatorias se usan para deshacer efectos en sistemas eventualmente consistentes, aunque la compensación puede fallar y debe ser idempotente. ([Stripe Docs][7])

En incident response, NIST SP 800-61 Rev. 3 integra la respuesta a incidentes dentro de la gestión de riesgo y busca mejorar detección, respuesta y recuperación. NIST CSF define Recover como restaurar capacidades o servicios afectados. ([csrc.nist.gov][8])

---

# 1. Qué significa rollback en SDO

## Práctica aceptada

Rollback suele entenderse como volver a una versión anterior conocida como estable. Pero en operaciones reales eso no siempre es posible: puede haber datos modificados, credenciales filtradas, cambios externos, estado parcial o acciones humanas irrepetibles.

## Propuesta SDO

En SDO, **rollback no significa obligatoriamente “volver exactamente al estado anterior”**.

Significa:

> Capacidad planificada, aprobada, ejecutable, verificable y evidenciable para llevar el sistema desde un estado fallido, degradado o inseguro hacia un **estado seguro aceptado**.

Ese estado seguro puede ser:

1. el estado anterior;
2. una versión anterior estable;
3. un estado corregido nuevo;
4. un estado degradado pero controlado;
5. un estado restaurado desde backup/snapshot;
6. un estado aislado/mitigado;
7. un estado manualmente aceptado como no reversible.

---

# 2. Diferencias conceptuales

| Concepto                | Significado en SDO                                                           | Ejemplo                                                        |
| ----------------------- | ---------------------------------------------------------------------------- | -------------------------------------------------------------- |
| **Rollback**            | Volver a un estado anterior conocido o versión previa.                       | Revertir versión de app, restaurar config anterior.            |
| **Roll-forward**        | Avanzar a un estado corregido nuevo, sin volver atrás.                       | Aplicar hotfix después de un deploy fallido.                   |
| **Restore**             | Recuperar desde backup, snapshot, restore point o imagen.                    | Restaurar BD a las 10:05.                                      |
| **Failover**            | Mover servicio a otro nodo, zona, región o instancia sana.                   | Cambiar tráfico a región secundaria.                           |
| **Compensating action** | Ejecutar una acción lógica que compensa efectos no reversibles directamente. | Cancelar una orden creada por API.                             |
| **Mitigation**          | Reducir impacto sin resolver causa raíz.                                     | Desactivar feature flag, limitar tráfico.                      |
| **Recovery**            | Proceso completo de volver a operación aceptable.                            | Mitigar, restaurar datos, validar servicio y cerrar incidente. |
| **No-rollback**         | Operación sin retorno técnico viable.                                        | Rotar clave ya filtrada; no se “desfiltra”.                    |
| **Point of no return**  | Momento a partir del cual la reversión cambia de estrategia, coste o riesgo. | Migración de datos ya escrita y usada por usuarios.            |

---

# 3. Modelo SDO Rollback & Reversibility

## Propuesta SDO

Añadir a SDO un submodelo transversal:

```text
Rollback Intent
  → Reversibility Assessment
  → Rollback Strategy Selection
  → Rollback Readiness Gate
  → Execution Monitoring
  → Rollback Trigger / Stop Condition
  → Rollback Execution Gate
  → Safe-State Validation
  → Rollback Evidence
  → Review / Lessons Learned
```

No es una fase adicional después de Execution. Es un **control transversal** que debe existir desde Spec y Plan.

---

# 4. Niveles de reversibilidad recomendados

## Propuesta SDO

| Nivel  | Nombre                       | Descripción                                        | Ejemplo                                                        |
| ------ | ---------------------------- | -------------------------------------------------- | -------------------------------------------------------------- |
| **R0** | No-change / Preview only     | No cambia estado. Reversible por no ejecución.     | `terraform plan`, `SELECT`, `--dry-run`.                       |
| **R1** | Naturally reversible         | La propia operación tiene undo directo y seguro.   | Revertir config versionada.                                    |
| **R2** | Version reversible           | Se puede volver a una versión anterior conocida.   | Git revert, Kubernetes rollout undo.                           |
| **R3** | Snapshot / backup reversible | Requiere snapshot, backup, restore point o imagen. | Restaurar VM, BD, volumen.                                     |
| **R4** | Compensatable                | No vuelve idéntico, pero compensa efectos.         | Cancelar transacción API, revocar permiso.                     |
| **R5** | Roll-forward only            | La vía segura es corregir hacia adelante.          | Fix de migración ya aplicada.                                  |
| **R6** | Mitigation only              | Solo se puede contener o reducir impacto.          | Aislar host comprometido.                                      |
| **R7** | Irreversible / no-rollback   | No hay reversión técnica aceptable.                | Borrado seguro, secreto expuesto, pérdida de datos sin backup. |

Regla SDO: **R0–R2 pueden ser SDO-Lightweight o Normal; R3–R5 requieren controles formales; R6–R7 requieren aceptación explícita de riesgo.**

---

# 5. Tipos de estrategia

## Propuesta SDO

| Estrategia              | Cuándo usarla                                            | Resultado esperado                         |
| ----------------------- | -------------------------------------------------------- | ------------------------------------------ |
| **rollback**            | Existe versión/config anterior válida.                   | Estado anterior restaurado.                |
| **roll_forward**        | Volver atrás es más riesgoso que corregir.               | Nuevo estado estable corregido.            |
| **restore**             | El estado previo solo existe en backup/snapshot.         | Estado recuperado desde punto temporal.    |
| **failover**            | Hay alternativa preparada.                               | Servicio operativo en destino alternativo. |
| **compensating_action** | Sistemas distribuidos o APIs sin rollback transaccional. | Efecto lógico neutralizado.                |
| **mitigation**          | Prioridad es detener degradación.                        | Impacto reducido, causa pendiente.         |
| **recovery**            | Proceso amplio post-fallo o incidente.                   | Servicio vuelve a condición aceptada.      |
| **no_rollback**         | No existe reversión viable.                              | Riesgo aceptado antes de ejecutar.         |

---

# 6. Cuándo exigir rollback probado

## Propuesta SDO

Una operación requiere **rollback probado** si cumple una o más condiciones:

1. modifica producción;
2. afecta datos persistentes;
3. afecta identidad, permisos, claves o secretos;
4. impacta disponibilidad, red, routing, DNS, firewall o balanceadores;
5. se ejecuta sobre múltiples nodos;
6. no tiene preview fiable;
7. tiene punto de no retorno;
8. requiere ventana de mantenimiento;
9. pertenece a SDO-Critical, SDO-Emergency o SDO-Pipeline-Backed;
10. el coste de fallo supera el coste de probar rollback.

Práctica relacionada: AWS recomienda probar regularmente failover/DR para validar que se cumplen RTO y RPO. ([AWS Documentation][9])

---

# 7. Cómo modelar rollback cuando no se puede volver exactamente atrás

## Propuesta SDO

SDO debe abandonar la idea binaria “hay rollback / no hay rollback”.

Debe registrar:

```text
original_state_target: exact | equivalent | safe_degraded | corrected_forward | manually_accepted
```

Ejemplos:

| Caso                                          | No se puede volver exacto porque…                                  | Estrategia SDO                      |
| --------------------------------------------- | ------------------------------------------------------------------ | ----------------------------------- |
| Migración SQL con datos escritos por usuarios | Ya hay datos nuevos dependientes del esquema.                      | roll-forward o compensación.        |
| Secreto expuesto                              | El secreto no puede “desexponerse”.                                | revocar, rotar, auditar uso.        |
| Cambio IAM aplicado                           | Los accesos usados durante la ventana pueden haber tenido efectos. | revocar + revisar logs.             |
| API externa llamada                           | El tercero ya procesó la acción.                                   | compensating action.                |
| Borrado sin backup                            | No hay fuente de verdad previa.                                    | no-rollback + aceptación explícita. |

---

# 8. Validación de backups, snapshots, restore points y versiones previas

## Práctica aceptada

Un backup no validado no debe considerarse rollback. SQL Server recomienda probar la estrategia restaurando backups y recuperando la base de datos; PostgreSQL requiere continuidad de WAL para PITR; Windows System Restore usa restore points como snapshots de configuración y ajustes del sistema. ([Microsoft Learn][10])

## Propuesta SDO

Un recurso de recuperación es válido solo si tiene evidencia de:

| Recurso        | Evidencia mínima                                                      |
| -------------- | --------------------------------------------------------------------- |
| Backup         | timestamp, scope, ubicación, retención, cifrado, último restore test. |
| Snapshot       | ID, recurso origen, consistencia, dependencia de volúmenes, owner.    |
| Restore point  | fecha, tipo, cobertura, limitaciones.                                 |
| Versión previa | commit/tag/build/image digest/config hash.                            |
| State IaC      | backend, versión, lock, checksum, relación con config.                |
| DB PITR        | backup base + logs continuos + punto temporal recuperable.            |

Regla SDO:

> “Backup exists” no equivale a “rollback ready”.
> “Restore tested” sí puede considerarse evidencia de rollback readiness.

---

# 9. Rollback triggers

## Propuesta SDO

Un rollback trigger es una condición observable que activa rollback, stop, mitigación o revisión humana.

| Tipo        | Ejemplo                                                          |
| ----------- | ---------------------------------------------------------------- |
| Health      | servicio no responde, pods no ready, servicio Windows detenido.  |
| SLO/SLA     | error rate, latency, disponibilidad fuera de umbral.             |
| Functional  | validación post-cambio falla.                                    |
| Security    | acceso no autorizado, política IAM incorrecta, secreto expuesto. |
| Data        | conteos inconsistentes, constraints fallidas, pérdida detectada. |
| Deployment  | canary falla, rollout excede deadline.                           |
| Human       | operador detecta desviación o inconsistencia.                    |
| AI-assisted | acción propuesta no coincide con spec aprobada.                  |

Regla SDO:

```text
Cada rollback trigger debe tener:
- métrica o señal observable
- umbral
- ventana temporal
- severidad
- acción asociada
- responsable
```

---

# 10. Stop conditions relacionadas con rollback

## Propuesta SDO

Una stop condition detiene la ejecución antes de empeorar la reversibilidad.

| Stop condition                                    | Acción                           |
| ------------------------------------------------- | -------------------------------- |
| No existe evidencia del backup requerido.         | Abort.                           |
| Snapshot falló o está incompleto.                 | Abort.                           |
| Drift detectado entre plan y estado real.         | Pause + replan.                  |
| Acción ejecutada no corresponde al plan aprobado. | Stop + deviation classification. |
| Se alcanza point of no return sin aprobación.     | Stop.                            |
| Rollback test falla.                              | Escalar aprobación o rediseñar.  |
| Métricas canary degradan.                         | Rollback o pause.                |
| IA propone comando no aprobado.                   | Rechazar ejecución.              |

---

# 11. Evidencia de rollback readiness

## Propuesta SDO

Antes de ejecutar, el Evidence Package debe contener:

```text
rollback_readiness_evidence:
  - rollback_strategy_selected
  - reversibility_level
  - previous_state_reference
  - backup_or_snapshot_reference
  - rollback_steps_defined
  - validation_steps_defined
  - trigger_conditions_defined
  - point_of_no_return_defined
  - required_permissions_verified
  - restore_test_or_reason_not_tested
  - approver_acceptance
```

Para SDO-Critical, “reason_not_tested” no debería bastar salvo emergencia documentada.

---

# 12. Evidencia de rollback execution

## Propuesta SDO

Después de ejecutar rollback o recuperación:

```text
rollback_execution_evidence:
  - trigger_that_started_rollback
  - timestamp_start
  - timestamp_end
  - operator_or_executor
  - actions_executed
  - mapping_to_rollback_plan
  - deviations
  - validation_results
  - final_safe_state
  - residual_risk
  - user_or_service_impact
  - follow_up_actions
```

Regla:

> Toda acción de rollback también debe mapearse a una acción planificada o a una desviación aprobada de emergencia.

---

# 13. Multi-target / fleet execution

## Propuesta SDO

Para ejecución sobre flotas:

| Patrón               | Regla SDO                                    |
| -------------------- | -------------------------------------------- |
| One-by-one           | Validar cada target antes de continuar.      |
| Batch                | Definir tamaño de lote y umbral de fallo.    |
| Canary               | Primer grupo pequeño con métricas estrictas. |
| Ring deployment      | Dev → staging → subset prod → full prod.     |
| Quorum-safe          | No degradar mayoría o réplica crítica.       |
| Blast-radius bounded | Nunca afectar más del límite aprobado.       |

Ejemplo de stop condition:

```yaml
fleet_stop_conditions:
  max_failed_targets: 2
  max_failed_percentage: 10
  stop_on_critical_target_failure: true
  require_human_continue_after_batch: true
```

---

# 14. Operaciones de datos

## Propuesta SDO

Las operaciones de datos deben tratarse como reversibilidad especial.

| Operación         | Riesgo     | Estrategia                              |
| ----------------- | ---------- | --------------------------------------- |
| `SELECT`          | Bajo       | R0, evidencia opcional.                 |
| `UPDATE/DELETE`   | Alto       | transacción, backup, export previo.     |
| Schema migration  | Medio/alto | backward-compatible migration.          |
| Data migration    | Alto       | shadow table, checksum, reconciliation. |
| Drop table/column | Crítico    | backup probado + point of no return.    |
| PITR              | Crítico    | restore probado en entorno alterno.     |

Reglas SDO para datos:

1. No ejecutar cambios destructivos sin snapshot/export/backup válido.
2. Separar schema migration de data migration cuando sea posible.
3. Preferir expand/contract: añadir primero, migrar después, borrar al final.
4. Definir si el rollback es transaccional, PITR, compensatorio o roll-forward.
5. Registrar checksums, conteos, constraints y validaciones funcionales.

---

# 15. Operaciones manuales/GUI

## Propuesta SDO

Para GUI/manual, rollback debe ser más explícito porque no hay trazabilidad automática completa.

Evidencia mínima:

```text
- captura antes
- captura después
- operador
- timestamp
- pantalla o ruta usada
- valor anterior
- valor nuevo
- validación independiente
- sign-off
```

Regla:

> Si una operación GUI no permite exportar configuración previa, debe exigirse captura o transcripción verificable del estado anterior.

---

# 16. IA asistida

## Práctica aceptada

OWASP identifica **Excessive Agency** como riesgo cuando un LLM puede ejecutar acciones dañinas por salidas inesperadas, ambiguas o manipuladas. Google describe patrones de uso con human-in-the-loop y audit trail: la IA propone, el humano aprueba y las acciones quedan registradas. ([OWASP Gen AI Security Project][11])

## Propuesta SDO

Reglas SDO para IA:

1. La IA puede proponer rollback, pero no ejecutar acciones irreversibles sin aprobación.
2. Cada comando generado por IA debe mapearse a Planned Action o Rollback Action.
3. La IA no puede inventar estado anterior: debe referenciar evidencia.
4. Toda acción IA debe tener `proposed_by`, `approved_by`, `executed_by`.
5. Para R4–R7, la IA solo puede operar en modo asistido o approval-required.
6. Comandos destructivos requieren preview, dry-run o confirmación humana.
7. La evidencia debe guardar prompt/resumen de intención, comando final aprobado y resultado, sin secretos.

---

# 17. Matriz: reversibility level → controles

| Nivel | Controles requeridos             | Approval    | Evidencia                 | Perfil SDO         |
| ----- | -------------------------------- | ----------- | ------------------------- | ------------------ |
| R0    | Preview/dry-run                  | No o ligera | output preview            | Lightweight        |
| R1    | Estado anterior conocido         | Ligera      | diff/config anterior      | Lightweight/Normal |
| R2    | Versión previa identificada      | Sí si prod  | commit/tag/revision       | Normal             |
| R3    | Backup/snapshot probado          | Sí          | backup ID + restore test  | Normal/Critical    |
| R4    | Compensación definida            | Sí          | compensating steps + logs | Normal/Critical    |
| R5    | Roll-forward plan                | Sí          | hotfix plan + validation  | Critical           |
| R6    | Mitigación + recuperación        | Sí, urgente | incident record           | Emergency/Critical |
| R7    | Aceptación explícita no-rollback | Obligatoria | risk acceptance           | Critical/Emergency |

---

# 18. Matriz: executor type → rollback mechanisms → caveats

| Executor           | Mecanismos                                                                | Caveats                                                                                                                                                  |
| ------------------ | ------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Linux CLI          | config backup, package downgrade, service restart, snapshot               | Red Hat advierte que rollback de paquetes como kernel/glibc/SELinux no siempre está soportado. ([Red Hat Customer Portal][12])                           |
| Windows            | restore point, export registry, service rollback, MSI uninstall, snapshot | Restore points cubren configuración/sistema, no sustituyen backup completo de datos. ([Soporte de Microsoft][13])                                        |
| SQL                | transaction rollback, backup restore, PITR, migration rollback            | PITR depende de logs completos y backups válidos. ([PostgreSQL][6])                                                                                      |
| API                | idempotency key, retry, compensating call                                 | No siempre se puede deshacer efecto externo. ([Stripe Docs][7])                                                                                          |
| Kubernetes         | rollout undo, previous ReplicaSet, Helm rollback, Git revert              | No revierte PVCs, datos externos ni cambios manuales fuera de control. ([Kubernetes][4])                                                                 |
| Terraform/OpenTofu | revert config, re-apply, restore state version                            | State rollback no garantiza que recursos reales vuelvan solos. ([HashiCorp Developer][5])                                                                |
| Ansible            | idempotency, previous vars/templates, re-run desired state                | Idempotente no significa automáticamente reversible.                                                                                                     |
| GitOps             | revert commit, reconcile desired state                                    | Rollback manual en cluster puede generar drift. ([Open GitOps][3])                                                                                       |
| CI/CD              | blue-green, canary, progressive delivery                                  | Requiere métricas y routing controlado. AWS define blue/green como dos entornos con cambio de tráfico y capacidad de rollback. ([AWS Documentation][14]) |
| IAM/Security       | revoke, rotate, break-glass, policy revert                                | Secretos expuestos no son reversibles; solo revocables/rotables.                                                                                         |
| GUI/manual         | checklist, screenshot, export config, sign-off                            | Evidencia depende del operador.                                                                                                                          |
| AI-assisted        | approval gate, command binding, audit trail                               | Riesgo de excessive agency; limitar autonomía. ([OWASP Gen AI Security Project][11])                                                                     |

---

# 19. `sdo_rollback_plan` genérico

```yaml
sdo_rollback_plan:
  sdo_id: "SDO-YYYY-NNNN"
  operation_id: "OP-001"
  reversibility:
    level: "R3"
    exact_previous_state_possible: false
    target_safe_state: "equivalent"
    point_of_no_return:
      exists: true
      description: "After data migration starts, exact rollback requires PITR."
      approval_required_before_crossing: true

  strategy:
    type: "restore" # rollback | roll_forward | restore | failover | compensating_action | mitigation | recovery | no_rollback
    rationale: "Schema/data change cannot be safely reversed by code-only rollback."
    preferred_order:
      - "pause_execution"
      - "restore_database_snapshot"
      - "validate_application_health"

  prerequisites:
    required_permissions:
      - "db_restore"
      - "service_restart"
    required_artifacts:
      - "backup_id"
      - "pre_change_schema_hash"
      - "validation_queries"
    required_evidence:
      - "backup_exists"
      - "restore_test_completed"
      - "rollback_steps_reviewed"

  previous_state:
    source: "backup"
    reference: "backup-2026-05-22-1000"
    timestamp: "2026-05-22T10:00:00Z"
    integrity_check: "checksum-or-provider-validation"
    retention_until: "2026-06-22T00:00:00Z"

  triggers:
    - id: "TRG-001"
      signal: "post_validation_failed"
      threshold: "critical query returns mismatch"
      action: "start_rollback"
    - id: "TRG-002"
      signal: "error_rate"
      threshold: ">5% for 5 minutes"
      action: "pause_and_evaluate"

  stop_conditions:
    - id: "STOP-001"
      condition: "backup_not_verified"
      action: "abort_before_execution"
    - id: "STOP-002"
      condition: "unexpected_drift_detected"
      action: "pause_and_replan"

  rollback_actions:
    - id: "RB-001"
      maps_to_planned_action: "PA-003"
      description: "Stop application writes"
      executor_type: "service_orchestrator"
      command_or_procedure: "disable_write_traffic"
      expected_result: "No new writes accepted"
      evidence_required: true

    - id: "RB-002"
      maps_to_planned_action: "PA-004"
      description: "Restore database to pre-change snapshot"
      executor_type: "database"
      command_or_procedure: "restore_snapshot"
      expected_result: "Database restored to snapshot timestamp"
      evidence_required: true

    - id: "RB-003"
      description: "Run validation checks"
      executor_type: "sql"
      command_or_procedure: "run_validation_queries"
      expected_result: "All checks pass"
      evidence_required: true

  validation:
    success_criteria:
      - "service_health_ok"
      - "critical_queries_match_expected"
      - "no_new_errors_in_logs"
    timeout: "30m"
    owner: "ops-lead"

  approvals:
    rollback_plan_approved_by: "change-owner"
    no_rollback_acceptance_required: false
    emergency_override_allowed: true
```

---

# 20. `sdo_rollback_record` genérico

```yaml
sdo_rollback_record:
  sdo_id: "SDO-YYYY-NNNN"
  rollback_record_id: "RBREC-001"
  related_execution_record: "EXE-001"

  trigger:
    trigger_id: "TRG-001"
    detected_at: "2026-05-22T11:14:00Z"
    detected_by: "monitoring"
    evidence_ref: "evidence://metrics/error-rate-1114"

  decision:
    decision: "execute_rollback"
    decided_by: "ops-lead"
    decided_at: "2026-05-22T11:16:00Z"
    rationale: "Validation failed after deployment."

  execution:
    started_at: "2026-05-22T11:17:00Z"
    ended_at: "2026-05-22T11:31:00Z"
    executor_type: "database"
    operator: "manuel"
    actions:
      - rollback_action_id: "RB-001"
        status: "success"
        evidence_ref: "evidence://logs/disable-write-traffic"
      - rollback_action_id: "RB-002"
        status: "success"
        evidence_ref: "evidence://provider/snapshot-restore"
      - rollback_action_id: "RB-003"
        status: "success"
        evidence_ref: "evidence://sql/validation-results"

  deviations:
    - id: "DEV-001"
      classification: "timing_deviation"
      description: "Restore took 4 minutes longer than estimated."
      approved_by: "ops-lead"

  validation:
    final_state: "safe_equivalent"
    validation_status: "passed"
    residual_risk: "low"
    service_impact: "write downtime for 14 minutes"

  closure:
    closed_at: "2026-05-22T11:45:00Z"
    closed_by: "change-owner"
    review_required: true
```

---

# 21. Rollback Readiness Gate

## Propuesta SDO

El gate pasa solo si:

```text
1. reversibility_level definido
2. strategy.type definido
3. rollback_actions documentadas
4. triggers definidos
5. stop_conditions definidos
6. point_of_no_return definido o marcado como inexistente
7. evidencia previa disponible
8. permisos verificados
9. validación post-rollback definida
10. aprobación acorde al perfil SDO
```

Falla si:

```text
- no hay rollback y no hay aceptación explícita
- backup requerido no fue validado
- rollback depende de conocimiento tribal
- no hay owner
- no hay criterio de éxito
- no hay criterio de parada
```

---

# 22. Rollback Execution Gate

## Propuesta SDO

Antes de ejecutar rollback:

```text
1. confirmar trigger
2. congelar nuevas acciones no esenciales
3. clasificar estado actual
4. verificar que rollback sigue siendo válido
5. verificar si se cruzó point of no return
6. seleccionar rollback / roll-forward / restore / failover / mitigation
7. aprobar ejecución si el perfil lo exige
8. ejecutar paso a paso
9. validar safe state
10. registrar evidencia
```

---

# 23. Point of no return

## Propuesta SDO

Un point of no return debe declararse cuando:

1. se modifican datos persistentes;
2. se elimina información;
3. se cambia esquema incompatible;
4. se rota o expone secreto;
5. se revoca acceso crítico;
6. se destruye infraestructura;
7. se cambia DNS/routing global;
8. se afecta una flota completa;
9. se ejecuta una acción manual no repetible.

Estructura recomendada:

```yaml
point_of_no_return:
  exists: true
  point: "before destructive migration step"
  consequence: "rollback changes from config rollback to PITR restore"
  required_approval: "explicit_human_approval"
  last_safe_abort_step: "PA-004"
  evidence_before_crossing:
    - "backup_verified"
    - "stakeholder_approval"
    - "maintenance_window_active"
```

---

# 24. No-rollback explicit acceptance

## Propuesta SDO

Una operación `no_rollback` solo debe aceptarse si contiene:

```yaml
no_rollback_acceptance:
  accepted: true
  reason: "Cryptographic key exposure cannot be reversed."
  alternatives_considered:
    - "rollback"
    - "restore"
    - "compensating_action"
  chosen_strategy: "rotate_and_revoke"
  residual_risk: "old key may have been used before revocation"
  approver: "security-owner"
  accepted_at: "2026-05-22T10:00:00Z"
  required_follow_up:
    - "audit key usage"
    - "rotate dependent secrets"
    - "review access logs"
```

Regla:

> No-rollback no es ausencia de plan. Es aceptación explícita de irreversibilidad más plan de mitigación, recovery o compensación.

---

# 25. Ejemplos agnósticos

| Caso                                | Reversibilidad | Estrategia SDO                                                              |
| ----------------------------------- | -------------: | --------------------------------------------------------------------------- |
| Linux: cambiar config de Nginx      |          R1/R2 | guardar config previa, validar `nginx -t`, reload, restore config si falla. |
| Windows: modificar servicio crítico |          R1/R3 | exportar config, restore point/snapshot, revertir servicio.                 |
| SQL: `UPDATE` masivo                |          R3/R4 | transacción si cabe; backup/export; compensación si ya confirmó.            |
| API: crear objeto externo           |             R4 | idempotency key + delete/cancel compensatorio.                              |
| Kubernetes: deploy app              |             R2 | rollout undo o revert Git si GitOps.                                        |
| Terraform: cambio infra             |          R2/R5 | revert config y apply; state restore solo con análisis.                     |
| GUI: cambiar política en consola    |          R1/R4 | screenshot/export previo, checklist, sign-off.                              |
| IA asistida: comando generado       |        depende | ejecutar solo comandos aprobados y trazados contra plan.                    |

---

# 26. Anti-patrones a evitar

## Propuesta SDO

1. **“Tenemos backup” sin restore test.**
2. **Rollback documentado después del cambio.**
3. **Rollback que solo revierte código, pero no datos.**
4. **Rollback manual no ensayado para operación crítica.**
5. **Confundir mitigación con recuperación completa.**
6. **Confundir Terraform state rollback con infraestructura restaurada.**
7. **Hacer `kubectl rollout undo` en entorno GitOps sin revertir Git.**
8. **No declarar point of no return.**
9. **No registrar quién aprobó cruzar el punto irreversible.**
10. **Permitir a IA ejecutar acciones destructivas sin approval gate.**
11. **Rollback de flota completa sin canary/batch.**
12. **No clasificar desviaciones durante rollback.**
13. **Guardar secretos en evidencia.**
14. **No medir impacto real ni residual risk.**
15. **Cerrar operación solo porque el comando terminó con exit code 0.**

---

# 27. Reglas finales del modelo SDO

```text
SDO-RB-01: Every state-changing operation must declare a reversibility level.

SDO-RB-02: Every rollback strategy must define the target safe state.

SDO-RB-03: Backup existence is not rollback readiness unless restore capability is verified or explicitly risk-accepted.

SDO-RB-04: Every point of no return must be declared before execution.

SDO-RB-05: Crossing a point of no return requires explicit approval for Normal/Critical/Emergency profiles.

SDO-RB-06: No-rollback requires explicit risk acceptance and a mitigation/recovery plan.

SDO-RB-07: Rollback actions must be mapped, evidenced and validated like normal execution actions.

SDO-RB-08: If exact rollback is impossible, SDO must choose roll-forward, restore, failover, compensation, mitigation or recovery.

SDO-RB-09: In fleet execution, rollback must be batch-aware and blast-radius bounded.

SDO-RB-10: AI-assisted rollback cannot execute irreversible or privileged actions without human approval and audit trail.
```

## Síntesis

El modelo recomendado es:

```text
Rollback in SDO = safe-state recovery capability,
not merely “undo”.
```

La clave metodológica es clasificar cada operación por **nivel de reversibilidad**, declarar la **estrategia de retorno o avance seguro**, validar la preparación antes de ejecutar y evidenciar tanto la readiness como la ejecución real.

[1]: https://sre.google/sre-book/service-best-practices/?utm_source=chatgpt.com "Google SRE: Production Services Best Practices"
[2]: https://dora.dev/guides/dora-metrics/?utm_source=chatgpt.com "DORA's software delivery performance metrics"
[3]: https://opengitops.dev/?utm_source=chatgpt.com "OpenGitOps: Home"
[4]: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/?utm_source=chatgpt.com "Deployments"
[5]: https://developer.hashicorp.com/terraform/cloud-docs/workspaces/state?utm_source=chatgpt.com "Manage workspace state in HCP Terraform"
[6]: https://www.postgresql.org/docs/current/continuous-archiving.html?utm_source=chatgpt.com "25.3. Continuous Archiving and Point-in-Time Recovery ..."
[7]: https://docs.stripe.com/api/idempotent_requests?utm_source=chatgpt.com "Idempotent requests | Stripe API Reference"
[8]: https://csrc.nist.gov/pubs/sp/800/61/r3/final?utm_source=chatgpt.com "SP 800-61 Rev. 3, Incident Response Recommendations and ..."
[9]: https://docs.aws.amazon.com/wellarchitected/latest/framework/rel_planning_for_recovery_dr_tested.html?utm_source=chatgpt.com "REL13-BP03 Test disaster recovery implementation to ..."
[10]: https://learn.microsoft.com/en-us/sql/relational-databases/backup-restore/back-up-and-restore-of-sql-server-databases?view=sql-server-ver17&utm_source=chatgpt.com "Back up and Restore of SQL Server Databases"
[11]: https://genai.owasp.org/llmrisk/llm06-sensitive-information-disclosure/?utm_source=chatgpt.com "LLM06:2025 Excessive Agency - OWASP Gen AI Security ..."
[12]: https://access.redhat.com/solutions/64069?utm_source=chatgpt.com "How to use yum history to roll back an update in Red Hat ..."
[13]: https://support.microsoft.com/en-us/windows/system-restore-a5ae3ed9-07c4-fd56-45ee-096777ecd14e?utm_source=chatgpt.com "System Restore"
[14]: https://docs.aws.amazon.com/whitepapers/latest/blue-green-deployments/introduction.html?utm_source=chatgpt.com "Introduction - Blue/Green Deployments on AWS"
