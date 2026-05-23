> Research note — non-normative.
> This document is background material and may be superseded by current SDO core/reference documentation.
> Current normative docs start at [../start-here.md](../start-here.md).

## 0. Base: qué tomo como práctica aceptada y qué propongo para SDO

**Prácticas aceptadas usadas como referencia:**

SDD moderno usa una cadena tipo **Spec → Plan → Tasks → Implement**, con artefactos Markdown/checklists que alimentan la siguiente fase y con validación de prerrequisitos antes de implementar. GitHub Spec Kit lo documenta así, y trata la spec como fuente de verdad para reducir ambigüedad y guiar a agentes de IA/código. ([github.github.com][1])

ITIL Change Enablement distingue cambios **standard/preaprobados**, **normal/con evaluación y aprobación**, y **emergency/con vía rápida y revisión posterior**. Esta clasificación es útil, pero SDO debe evitar convertirla en un CAB pesado por defecto. ([ITSM.tools][2])

SRE aporta tres controles muy fuertes: **SLO/error budget**, **rollouts graduales/canary**, y **postmortems blameless**. Google SRE recomienda supervisar rollouts y, ante comportamiento inesperado, “rollback first and diagnose afterward”. ([sre.google][3])

DevOps/DORA mide estabilidad con **change failure rate**, **time to restore service** y, en la evolución reciente de DORA, también **deployment rework rate**. Estos sirven como métricas de aprendizaje para decidir qué operaciones deben volverse más controladas o automatizables. ([dora.dev][4])

GitOps/IaC aportan **estado deseado versionado**, **diff/plan**, reconciliación y políticas automáticas. OpenGitOps define estado declarativo, versionado e inmutable, pull automático y reconciliación continua; Terraform `plan` permite previsualizar cambios antes de aplicarlos. ([opengitops.dev][5])

Runbook automation aporta pasos, salidas, logs y aprobaciones manuales insertables. AWS Systems Manager permite pausar una automatización hasta recibir aprobación; Rundeck define comportamiento de workflow cuando falla un paso. ([AWS Documentation][6])

Seguridad aporta **least privilege**, **separation of duties** y trazabilidad. NIST SP 800-53 es un catálogo de controles adaptable a organizaciones, y sus controles AC-5/AC-6 cubren separación de funciones y mínimo privilegio. ([csrc.nist.gov][7])

**Propuesta propia para SDO:** usar gates pequeños, verificables y proporcionales al riesgo. No todos son reuniones ni aprobaciones humanas. Un gate puede ser una checklist, una política automática, un diff revisado, un precheck, una pausa manual, una métrica SLO o una validación de evidencia.

---

# 1. Modelo propuesto de gates SDO

Propongo estos gates:

| Gate | Nombre                            | Momento                     |
| ---- | --------------------------------- | --------------------------- |
| G0   | Intake & Traceability Gate        | Antes de crear Spec         |
| G1   | Spec Readiness Gate               | Intent → Spec               |
| G2   | Risk/Profile Classification Gate  | Spec → Plan                 |
| G3   | Plan / Preview Gate               | Plan → Runbook              |
| G4   | Approval / Authority Gate         | Antes de ejecución          |
| G5   | Pre-Execution Readiness Gate      | Justo antes de ejecutar     |
| G6   | Step Execution Gate               | Durante cada paso relevante |
| G7   | Continue / Stop Gate              | Durante ejecución           |
| G8   | Rollback / Recovery Gate          | Cuando hay fallo o trigger  |
| G9   | Validation Gate                   | Después de ejecutar         |
| G10  | Evidence & Closure Gate           | Antes de cerrar             |
| G11  | Standardization / Automation Gate | Después de Review           |

---

# 2. Definición compacta de cada gate

| Gate                  | Objetivo                                            | Entradas requeridas                                                               | Criterios de aprobación/fallo                                | Salidas posibles                              | Evidencia                         |
| --------------------- | --------------------------------------------------- | --------------------------------------------------------------------------------- | ------------------------------------------------------------ | --------------------------------------------- | --------------------------------- |
| **G0 Intake**         | Dar trazabilidad mínima.                            | Intent, solicitante, sistema, objetivo, SDO ID.                                   | Hay objetivo claro y scope inicial.                          | proceed / hold.                               | Registro SDO ID.                  |
| **G1 Spec Readiness** | Evitar ejecutar con intención ambigua.              | Operational Spec, objetivo, no-objetivos, alcance, estado esperado.               | Spec verificable, sin contradicciones críticas.              | proceed / hold.                               | Spec versionada.                  |
| **G2 Risk/Profile**   | Elegir rigor proporcional.                          | Clase base, overlays, impacto, reversibilidad, criticidad, datos afectados.       | Perfil SDO asignado y justificado.                           | proceed / escalate / hold.                    | Risk record.                      |
| **G3 Plan/Preview**   | Saber qué cambiará antes de cambiar.                | Plan, runbook draft, dry-run/diff/simulation si existe.                           | Impacto entendido; dependencias y orden definidos.           | proceed / hold / abort / accept-risk.         | Plan, diff, preview, análisis.    |
| **G4 Approval**       | Autorizar según riesgo, no por burocracia.          | G2 + G3, owner, aprobador, ventana, rollback.                                     | Autoridad correcta; sin autoaprobación en alto riesgo.       | proceed / hold / reject / accept-risk.        | Approval record.                  |
| **G5 Pre-Execution**  | Confirmar que se puede ejecutar ahora.              | Prechecks, backup/snapshot si aplica, acceso, ventana, monitoreo, rollback listo. | Prechecks OK o riesgo aceptado explícitamente.               | proceed / hold / abort.                       | Precheck report.                  |
| **G6 Step Execution** | Controlar cada paso con logs.                       | Runbook, comandos/acciones, operador/agente.                                      | Cada paso produce salida esperada o estado conocido.         | continue / hold / abort.                      | Execution log.                    |
| **G7 Continue/Stop**  | Decidir si seguir, parar o escalar.                 | Métricas, logs, errores, SLO/SLA, health checks.                                  | No se cruzan stop conditions.                                | proceed / hold / abort / rollback / escalate. | Decision log.                     |
| **G8 Rollback**       | Recuperar de forma controlada.                      | Rollback plan, trigger, estado actual, backups.                                   | Trigger confirmado y rollback viable o recovery alternativo. | rollback / hold / escalate / accept-risk.     | Rollback record.                  |
| **G9 Validation**     | Verificar resultado real.                           | Postchecks, pruebas, métricas, estado esperado.                                   | Resultado cumple Spec y no rompe condiciones críticas.       | proceed / rollback / hold / accept-risk.      | Validation report.                |
| **G10 Closure**       | Cerrar solo con evidencia mínima.                   | Execution Record, Validation Report, Evidence Package, desviaciones.              | Evidencia suficiente y decisión documentada.                 | close / reopen / escalate.                    | Evidence Package + Review Record. |
| **G11 Automation**    | Convertir operaciones repetibles en flujo estándar. | Historial, tasa de éxito, fallos, duración, evidencia.                            | Operación repetible, bajo riesgo, controles automatizables.  | standardize / automate / keep-manual.         | Automation candidate record.      |

---

# 3. Gates universales obligatorios

Estos deberían existir en **todos** los perfiles, incluso SDO-Lightweight:

1. **SDO ID** y trazabilidad mínima.
2. **Spec mínima**: qué se quiere lograr, dónde, límites y resultado esperado.
3. **Clasificación de riesgo/perfil**.
4. **Prechecks mínimos**.
5. **Plan de rollback o declaración explícita de “rollback no disponible”**.
6. **Registro de ejecución**.
7. **Postchecks/validación**.
8. **Evidencia mínima para cierre**.
9. **Review ligero**, aunque sea una nota breve.

La propuesta sigue la lógica de SDD: no pasar a implementación sin spec/plan/tareas suficientes; pero en SDO la “implementación” es una operación real sobre sistemas, por lo que el gate de ejecución debe ser más estricto que en desarrollo puro. ([github.github.com][1])

---

# 4. Gate obligatorio antes de ejecución

El gate clave antes de ejecución debe ser:

## **Execution Authorization Gate = G3 + G4 + G5**

No basta con “alguien aprobó”. Deben cumplirse estas condiciones:

| Condición                  | Regla SDO                                                                      |
| -------------------------- | ------------------------------------------------------------------------------ |
| Spec clara                 | Qué cambiará y qué no cambiará.                                                |
| Plan/runbook listo         | Pasos ordenados, dependencias, operador/agente definido.                       |
| Preview/diff/dry-run       | Obligatorio si la tecnología lo permite.                                       |
| Risk profile asignado      | Lightweight, Normal, Critical, Emergency o Pipeline-Backed.                    |
| Rollback/recovery definido | Incluso si es “restore backup”, “failover”, “manual repair” o “no reversible”. |
| Prechecks OK               | Salud actual, permisos, backups, ventana, monitoreo.                           |
| Aprobación proporcional    | Según riesgo, no por costumbre.                                                |

Ejemplos de preview aceptado: `terraform plan`, Ansible check/diff mode, Flyway dry-run, Liquibase rollback SQL, pull request diff, GitOps diff o policy-as-code. Terraform documenta `plan` como mecanismo para previsualizar los cambios antes de aplicar; Ansible soporta check/diff mode; Flyway permite dry-run SQL en ediciones compatibles. ([HashiCorp Developer][8])

---

# 5. Gate durante ejecución: continuar, parar o revertir

Durante ejecución debe existir un **Continue/Stop Gate**.

Regla propuesta:

> Cada paso significativo debe terminar en uno de estos estados: `continue`, `hold`, `abort`, `rollback`, `escalate`, `accept-risk`.

El gate se evalúa cuando ocurre cualquiera de estos eventos:

| Evento                                 | Acción                           |
| -------------------------------------- | -------------------------------- |
| Paso falla                             | hold o rollback según severidad. |
| Métrica crítica empeora                | stop y evaluar rollback.         |
| SLO/error budget afectado              | stop si cruza umbral definido.   |
| Cambio no coincide con diff esperado   | abort.                           |
| Se detecta alcance no previsto         | hold + revalidar Spec.           |
| Operador/agente necesita permiso extra | hold + approval.                 |
| Operación tarda más de lo previsto     | hold/escalate.                   |
| Validación intermedia falla            | rollback o recovery.             |

SRE recomienda rollouts supervisados y rollback primero cuando aparece comportamiento inesperado; por eso SDO debe tratar el stop/rollback como una decisión normal, no como fracaso. ([sre.google][3])

---

# 6. Gate antes de cerrar

Antes de cerrar, SDO debe exigir:

## **Closure Gate = G9 + G10**

Criterios mínimos:

| Criterio           | Debe existir                                            |
| ------------------ | ------------------------------------------------------- |
| Resultado esperado | Confirmado con postchecks.                              |
| Estado final       | Documentado.                                            |
| Desviaciones       | Registradas.                                            |
| Evidencia          | Adjunta o enlazada.                                     |
| Impacto            | Confirmado o declarado no aplicable.                    |
| Rollback           | No requerido, ejecutado o descartado con justificación. |
| Review             | Lecciones, follow-ups o “sin acciones”.                 |

Para operaciones críticas, el cierre no debería depender solo de “el comando salió 0”. Debe depender de evidencia externa: métricas, logs, health checks, consultas, estado del servicio o confirmación del owner.

---

# 7. Matriz: perfil SDO → gates requeridos

| Gate                  |        Lightweight |                Normal |                  Critical |                      Emergency |        Pipeline-Backed |
| --------------------- | -----------------: | --------------------: | ------------------------: | -----------------------------: | ---------------------: |
| G0 Intake/SDO ID      |                 Sí |                    Sí |                        Sí |                             Sí |                     Sí |
| G1 Spec               |             Mínima |              Completa |                  Completa |                 Mínima urgente |    Completa/versionada |
| G2 Risk/Profile       |             Simple |                Formal |                    Formal |                         Rápida |    Automática + formal |
| G3 Plan/Preview       |          Checklist | Obligatorio si existe |               Obligatorio |        Si el tiempo lo permite |            Obligatorio |
| G4 Approval           | Auto/peer opcional |            Peer/owner | Owner + técnico + control | Incident Commander/ECAB ligero |        PR/env approval |
| G5 Prechecks          |            Básicos |          Obligatorios |     Obligatorios + backup |               Críticos mínimos |          Automatizados |
| G6 Step Gate          |             Simple |    Por paso relevante |                  Por paso |                      Por hitos |        Pipeline stages |
| G7 Continue/Stop      |             Básico |              Definido |                  Estricto |         Estricto por seguridad |        Métricas/policy |
| G8 Rollback           |          Declarado |              Definido |        Probado o validado |           Recovery prioritario | Automatizado o runbook |
| G9 Validation         |             Básica |                Formal |            Formal + owner |     Confirmación de mitigación |    Tests/health checks |
| G10 Evidence/Closure  |             Mínima |              Completa |         Completa + Review |        Post-review obligatorio |       Artefactos CI/CD |
| G11 Automation Review |           Opcional |       Sí si repetible |           Sí, con cautela |          Después del incidente |                     Sí |

---

# 8. Matriz: clase/overlay → controles adicionales

| Clase / overlay         | Controles adicionales                                                                                 |
| ----------------------- | ----------------------------------------------------------------------------------------------------- |
| **Observe**             | No mutación; permisos read-only; evidencia de consulta; cuidado con datos sensibles.                  |
| **Analyze**             | Separar hipótesis de hechos; preservar evidencia; no ejecutar cambios correctivos sin nuevo gate.     |
| **Maintain**            | Ventana, backup/snapshot, health checks, rollback operativo.                                          |
| **Configure**           | Diff obligatorio si existe; control de drift; validación de configuración efectiva.                   |
| **Deploy**              | Canary/rolling/blue-green si aplica; métricas de salud; rollback rápido.                              |
| **Control**             | Aprobación fuerte; least privilege; separación de funciones; audit trail.                             |
| **Data**                | Backup verificado; dry-run/migration preview; transacción/rollback; validación de conteos/integridad. |
| **Access**              | Justificación, caducidad, mínimo privilegio, doble aprobación si privilegio alto.                     |
| **Remediate**           | Prioridad a restaurar servicio; decision log; postmortem si impacto alto.                             |
| **Destroy**             | Two-person rule; confirmación explícita del recurso; backup/export si aplica; irreversible flag.      |
| **High-risk overlay**   | Aprobación independiente, rollback probado, ventana y monitoreo.                                      |
| **Security overlay**    | SoD, least privilege, evidencia de autorización, logs inmutables.                                     |
| **Production overlay**  | Protected environment, owner approval, SLO guardrails.                                                |
| **No-rollback overlay** | Accept-risk obligatorio antes de ejecución.                                                           |
| **AI-assisted overlay** | Revisión humana antes de acciones mutativas.                                                          |

Para despliegues y entornos protegidos, GitHub, GitLab y Azure DevOps ya soportan conceptos equivalentes: required reviews, deployment protection rules, protected environments y approvals/checks. ([GitHub Docs][9])

---

# 9. Política de aprobación proporcional al riesgo

Propuesta SDO:

| Riesgo                                     | Aprobación mínima                                | Ejemplos                                                                |
| ------------------------------------------ | ------------------------------------------------ | ----------------------------------------------------------------------- |
| **R0 trivial/read-only**                   | Autoaprobado                                     | Consultar logs, revisar estado.                                         |
| **R1 bajo/repetible**                      | Auto o peer async                                | Reiniciar servicio no crítico, rotar logs.                              |
| **R2 medio**                               | Peer técnico u owner                             | Cambiar config, actualizar paquete.                                     |
| **R3 alto/producción**                     | Owner + ejecutor técnico distinto                | Deploy productivo, cambios de red.                                      |
| **R4 crítico/datos/seguridad/destructivo** | Dos personas + owner + seguridad/datos si aplica | Borrar datos, modificar IAM admin, migración DB.                        |
| **Emergency**                              | Incident Commander o autoridad de emergencia     | Restaurar servicio, contener incidente. Revisión posterior obligatoria. |
| **Pipeline-backed**                        | PR review + policy checks + environment approval | IaC, GitOps, CI/CD.                                                     |

Reglas importantes:

* **No autoaprobación** para cambios R3/R4.
* **La aprobación debe mirar evidencia**, no solo el título del cambio.
* **La autoridad debe corresponder al riesgo**: técnico, owner de servicio, seguridad, datos o incidente.
* **Toda aceptación de riesgo debe quedar explícita**: quién, por qué, hasta cuándo y qué compensación existe.
* **Los cambios estándar pueden ser preaprobados**, pero solo si el runbook, controles y límites están congelados.

---

# 10. Prechecks y postchecks

## Prechecks mínimos

| Tipo             | Ejemplo                                                         |
| ---------------- | --------------------------------------------------------------- |
| Identidad/acceso | Usuario correcto, privilegio mínimo, sesión trazable.           |
| Estado inicial   | Servicio activo, versión actual, capacidad, errores existentes. |
| Dependencias     | Red, DNS, DB, storage, APIs, permisos.                          |
| Backup/recovery  | Snapshot, backup reciente, restore probado si aplica.           |
| Ventana          | Hora válida, usuarios informados si aplica.                     |
| Observabilidad   | Métricas/logs disponibles antes de tocar.                       |
| Dry-run/diff     | Generado y revisado cuando exista.                              |

## Postchecks mínimos

| Tipo         | Ejemplo                                       |
| ------------ | --------------------------------------------- |
| Estado final | Config efectiva, versión, servicio activo.    |
| Funcional    | Health check, login, consulta, endpoint, job. |
| Métricas     | Latencia, errores, saturación, tráfico.       |
| Seguridad    | Permisos esperados, no exposición accidental. |
| Datos        | Conteos, checksums, constraints, integridad.  |
| Drift        | Estado real coincide con Spec/desired state.  |

Google SRE usa las “four golden signals” —latency, traffic, errors, saturation— como señales base de monitoreo en sistemas de usuario final. Para SDO, esas señales pueden convertirse en postchecks y stop conditions cuando la operación afecta servicios. ([sre.google][10])

---

# 11. Dry-run, diff, simulation o preview

Regla SDO:

> Si la tecnología ofrece preview razonable, el preview es obligatorio antes de mutar producción, salvo emergencia documentada.

Ejemplos:

| Contexto      | Preview                                                        |
| ------------- | -------------------------------------------------------------- |
| Terraform/IaC | `terraform plan`.                                              |
| Ansible       | check mode + diff mode.                                        |
| GitOps        | diff entre desired/current state.                              |
| Kubernetes    | server-side dry-run, admission policy, diff.                   |
| DB migrations | Flyway dry-run, Liquibase SQL/rollback SQL, staging migration. |
| Código/deploy | PR diff, tests, canary.                                        |
| Config manual | captura antes/después, export config, checklist.               |

Policy-as-code puede automatizar parte del gate. OPA se define como motor general para especificar políticas como código y descargar decisiones de autorización/política desde aplicaciones, APIs, CI/CD o Kubernetes. ([Open Policy Agent][11])

---

# 12. Operaciones sin dry-run posible

No se bloquean automáticamente, pero suben de rigor.

Reglas propuestas:

1. Reducir blast radius: un host, un tenant, un nodo, un porcentaje pequeño.
2. Crear backup/snapshot/export antes.
3. Ejecutar en staging/lab si existe.
4. Hacer checkpoint manual antes del paso irreversible.
5. Definir stop conditions más conservadoras.
6. Requerir accept-risk explícito si el cambio es irreversible.
7. Registrar por qué no existe preview.
8. Tener recovery alternativo, aunque no sea rollback exacto.

En bases de datos, por ejemplo, algunas operaciones pueden depender de transacciones o soporte del motor. PostgreSQL documenta `ROLLBACK` para descartar cambios dentro de una transacción, pero no todas las operaciones de todos los motores son reversibles de la misma forma. ([PostgreSQL][12])

---

# 13. Stop conditions y rollback triggers

## Stop conditions

| Tipo      | Ejemplo                                                           |
| --------- | ----------------------------------------------------------------- |
| Técnico   | Error no esperado, timeout, dependencia caída.                    |
| Métrico   | Error rate sube, latencia sube, saturación crítica.               |
| Seguridad | Permiso más amplio de lo previsto, exposición pública accidental. |
| Datos     | Conteo inesperado, constraint rota, pérdida de registros.         |
| Alcance   | Recurso objetivo no coincide con Spec.                            |
| Humano    | Operador no entiende el siguiente paso o detecta ambigüedad.      |
| IA        | El agente propone acción fuera de spec o pide credenciales extra. |

## Rollback triggers

| Trigger                           | Acción                               |
| --------------------------------- | ------------------------------------ |
| Cambio rompe health check crítico | rollback inmediato.                  |
| Canary falla                      | detener promoción y revertir.        |
| Error afecta SLO/error budget     | rollback o escalado.                 |
| Se detecta dato inconsistente     | parar, congelar evidencia, recovery. |
| Desviación de plan aprobado       | abortar y reaprobar.                 |
| Acción irreversible no confirmada | abortar.                             |

---

# 14. Reglas para SDO-Emergency

SDO-Emergency no significa “sin control”. Significa **controles mínimos, rápidos y post-review obligatorio**.

Reglas:

1. El objetivo primario es **contener, restaurar o reducir impacto**.
2. La Spec puede ser mínima: problema, impacto, hipótesis, acción propuesta.
3. Aprobación por **Incident Commander**, service owner o autoridad definida.
4. Se permite saltar dry-run si el tiempo lo impide, pero debe quedar registrado.
5. Cada decisión importante va al **decision log**.
6. Stop condition principal: la acción empeora el incidente o amplía el blast radius.
7. Closure requiere evidencia de mitigación.
8. Review/postmortem obligatorio si hubo impacto relevante.

NIST SP 800-61 Rev. 3, publicado como guía actual de respuesta a incidentes, alinea la respuesta con funciones de gestión de riesgo tipo Govern, Identify, Protect, Detect, Respond y Recover. ([csrc.nist.gov][13])

---

# 15. Reglas para IA / Human-in-the-loop

Propuesta SDO:

| Nivel de autonomía    | Permitido                                                                 |
| --------------------- | ------------------------------------------------------------------------- |
| **AI-Observe**        | Leer logs, resumir, detectar anomalías.                                   |
| **AI-Analyze**        | Proponer causa, plan, comandos, riesgos.                                  |
| **AI-Draft**          | Generar Spec, Plan, Runbook, checks.                                      |
| **AI-Assist Execute** | Sugerir siguiente paso; humano ejecuta.                                   |
| **AI-Semi-Auto**      | Ejecuta pasos no críticos con aprobación por checkpoint.                  |
| **AI-Auto**           | Solo operaciones standard, reversibles, acotadas y con policy guardrails. |

Reglas duras:

* La IA no debe ejecutar mutaciones productivas sin gate aprobado.
* La IA no debe ampliar scope por inferencia.
* Todo comando generado por IA debe ser revisable antes de ejecución en R2+.
* Para R3/R4: humano aprueba plan, diff y comando final.
* Para acciones destructivas: two-person rule.
* El log debe distinguir **quién decidió** y **quién ejecutó**: humano, agente, pipeline.

NIST AI RMF está orientado a gestionar riesgos de IA y mejorar la incorporación de consideraciones de confiabilidad en diseño, desarrollo, uso y evaluación de sistemas de IA. ([NIST][14])

---

# 16. Anti-patrones de control

| Anti-patrón                               | Por qué es malo                                       |
| ----------------------------------------- | ----------------------------------------------------- |
| **Approval theater**                      | Se aprueba sin revisar evidencia real.                |
| **CAB universal**                         | Todo pasa por comité aunque sea bajo riesgo.          |
| **Checklist sin decisión**                | Marca casillas pero no cambia el comportamiento.      |
| **Rollback ficticio**                     | Dice “rollback disponible” pero nadie lo probó.       |
| **Dry-run ignorado**                      | Existe preview, pero se ejecuta directo.              |
| **Spec vaga**                             | “Actualizar servidor” sin estado esperado ni límites. |
| **Over-control de cambios triviales**     | Lentitud, bypass informal y pérdida de confianza.     |
| **Under-control de cambios destructivos** | Riesgo real sin autoridad ni recuperación.            |
| **IA con permisos amplios**               | Agente puede hacer más de lo que la Spec permite.     |
| **Cierre por comando exitoso**            | Exit code 0 no equivale a validación operacional.     |
| **Postmortem culpabilizador**             | Reduce aprendizaje y oculta información.              |

---

# 17. Síntesis final del modelo SDO Gates

La regla central sería:

> **Una operación SDO puede avanzar solo si tiene suficiente especificación, riesgo clasificado, plan verificable, autorización proporcional, prechecks aprobados, stop/rollback definidos, validación posterior y evidencia de cierre.**

Y la regla anti-burocracia:

> **El control debe aumentar con impacto, irreversibilidad, privilegio, datos, producción, automatización e incertidumbre; no con costumbre organizativa.**

Mi recomendación metodológica sería dejar estos gates como núcleo oficial de SDO:

1. **Spec Gate**
2. **Risk/Profile Gate**
3. **Preview/Plan Gate**
4. **Approval Gate**
5. **Pre-Execution Gate**
6. **Continue/Stop Gate**
7. **Rollback Gate**
8. **Validation Gate**
9. **Evidence/Closure Gate**
10. **Automation Eligibility Gate**

[1]: https://github.github.com/spec-kit/?utm_source=chatgpt.com "Spec Kit Documentation"
[2]: https://itsm.tools/change-enablement/?utm_source=chatgpt.com "Change Enablement – Change Management in ITIL 4"
[3]: https://sre.google/sre-book/service-best-practices/?utm_source=chatgpt.com "Google SRE: Production Services Best Practices"
[4]: https://dora.dev/guides/dora-metrics/?utm_source=chatgpt.com "DORA's software delivery performance metrics"
[5]: https://opengitops.dev/?utm_source=chatgpt.com "OpenGitOps: Home"
[6]: https://docs.aws.amazon.com/systems-manager/latest/userguide/automation-action-approve.html?utm_source=chatgpt.com "aws:approve – Pause an automation for manual approval"
[7]: https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final?utm_source=chatgpt.com "SP 800-53 Rev. 5, Security and Privacy Controls for ..."
[8]: https://developer.hashicorp.com/terraform/cli/commands/plan?utm_source=chatgpt.com "terraform plan command reference"
[9]: https://docs.github.com/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches?utm_source=chatgpt.com "About protected branches"
[10]: https://sre.google/sre-book/monitoring-distributed-systems/?utm_source=chatgpt.com "Chapter 6 - Monitoring Distributed Systems"
[11]: https://openpolicyagent.org/docs?utm_source=chatgpt.com "Open Policy Agent (OPA)"
[12]: https://www.postgresql.org/docs/current/sql-rollback.html?utm_source=chatgpt.com "Documentation: 18: ROLLBACK"
[13]: https://csrc.nist.gov/pubs/sp/800/61/r3/final?utm_source=chatgpt.com "SP 800-61 Rev. 3, Incident Response Recommendations and ..."
[14]: https://www.nist.gov/itl/ai-risk-management-framework?utm_source=chatgpt.com "AI Risk Management Framework | NIST"
