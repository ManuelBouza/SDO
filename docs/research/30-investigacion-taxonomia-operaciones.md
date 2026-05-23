> Research note — non-normative.
> This document is background material and may be superseded by current SDO core/reference documentation.
> Current normative docs start at [../start-here.md](../start-here.md).

# Informe: Taxonomía agnóstica de operaciones para SDO

## 0. Conclusión ejecutiva

La taxonomía de SDO no debería clasificar operaciones por herramienta, tecnología o comando, sino por **efecto operacional** y por **riesgo contextual**.

La propuesta queda así:

**Clase base:** qué tipo de operación es.
**Overlays:** qué factores elevan o reducen el rigor.
**Perfil SDO:** qué flujo aplicar: `lightweight`, `normal`, `critical`, `emergency` o `pipeline-backed`.

Esto encaja con prácticas aceptadas: ITIL distingue cambios standard, normal y emergency; SRE gestiona riesgo mediante SLO/error budgets; DORA mide inestabilidad con change failure rate; GitOps/IaC separa plan, revisión y apply; y seguridad exige tratar privilegios, datos y auditoría como factores de riesgo, no como detalles secundarios. ([ITSM.tools][1])

---

# 1. Prácticas aceptadas vs propuesta SDO

## 1.1 Prácticas aceptadas

En **ITIL Change Enablement**, los cambios se agrupan típicamente en `standard`, `normal` y `emergency`: los standard son repetibles, de bajo riesgo y preautorizados; los normal requieren evaluación y autorización; los emergency se ejecutan de forma acelerada para resolver o prevenir incidentes, con revisión posterior. ([ITSM.tools][1])

En **SRE**, el riesgo operacional no se elimina, se gestiona. Google SRE plantea que la fiabilidad tiene coste y que los error budgets sirven para equilibrar velocidad de cambio y estabilidad; además, el toil se identifica como trabajo manual, repetitivo y automatizable que conviene reducir. ([sre.google][2])

En **DORA**, el `change failure rate` mide la proporción de despliegues que requieren intervención inmediata, rollback, hotfix o remediación; por tanto, no basta con clasificar una operación por “tipo técnico”, también importa su historial de fallo y recuperación. ([dora.dev][3])

En **GitOps**, el estado deseado debe ser declarativo, versionado, inmutable y reconciliado automáticamente; en **IaC**, herramientas como Terraform separan `plan` de `apply`, permitiendo revisar cambios antes de ejecutarlos, y distinguen operaciones normales, refresh-only y destructivas. ([opengitops.dev][4])

En **runbook automation**, un runbook puede modelarse como pasos secuenciales con entradas, salidas y acciones; AWS Systems Manager, por ejemplo, permite incluir acciones de aprobación que detienen temporalmente la automatización hasta que un aprobador decide. ([docs.aws.amazon.com][5])

En **seguridad**, el principio de mínimo privilegio, la trazabilidad y el control de acceso granular son fundamentales; además, una operación aparentemente read-only puede ser crítica si expone secretos, PII, tokens, configuraciones sensibles o datos regulados. ([cisa.gov][6])

En **bases de datos**, la reversibilidad depende mucho del motor y del tipo de operación: una transacción puede revertirse con `ROLLBACK`, pero una migración aplicada, una restauración parcial o un `DROP` sin backup verificable pueden ser difíciles o imposibles de revertir. PostgreSQL documenta tanto `ROLLBACK` como `pg_dump`/`pg_restore`, incluyendo advertencias de seguridad al restaurar dumps. ([PostgreSQL][7])

En **operaciones con IA human-in-the-loop**, la supervisión humana debe ser proporcional al riesgo, autonomía y contexto de uso. Esta idea aparece claramente en el Artículo 14 del AI Act para sistemas de alto riesgo. ([ai-act-service-desk.ec.europa.eu][8])

## 1.2 Propuesta propia para SDO

Lo siguiente es propuesta SDO:

SDO debería usar una taxonomía en 3 capas:

```text
Operation Class
+ Risk / Context Overlays
= SDO Flow Profile
```

Ejemplo:

```text
OC-OBSERVE
+ Sensitive Data Overlay
+ Production Overlay
+ Multi-target Scope
= Normal o Critical, no Lightweight
```

Esto evita el error de decir: “como es read-only, es bajo riesgo”.

---

# 2. Taxonomía principal SDO: clases base

## 2.1 Clases recomendadas

| ID             | Clase SDO                     | Definición                                                                            | Ejemplos agnósticos                                                                 |
| -------------- | ----------------------------- | ------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| `OC-OBSERVE`   | Observación                   | Lee estado, configuración, logs, métricas o datos sin cambiar el sistema.             | listar servicios, consultar pods, revisar logs, hacer `SELECT`, consultar API GET   |
| `OC-ANALYZE`   | Análisis / planificación      | Calcula impacto, genera diff, plan o diagnóstico sin aplicar cambios.                 | `terraform plan`, `kubectl diff`, dry-run, explain plan SQL                         |
| `OC-MAINTAIN`  | Mantenimiento estándar        | Acción repetible, conocida, documentada y normalmente preaprobada.                    | limpieza de temporales, rotación de logs, reinicio programado, renovación rutinaria |
| `OC-CONFIGURE` | Cambio de configuración       | Modifica configuración, parámetros, políticas o estado persistente.                   | editar config, cambiar GPO, modificar variable, actualizar parámetro cloud          |
| `OC-DEPLOY`    | Despliegue / release          | Introduce, reemplaza o revierte una versión de software, imagen, paquete o artefacto. | deploy app, actualizar contenedor, aplicar release, cambiar versión                 |
| `OC-CONTROL`   | Control runtime               | Cambia el comportamiento en ejecución sin modificar necesariamente el artefacto.      | start, stop, restart, scale, drain, failover, enable/disable                        |
| `OC-DATA`      | Operación sobre datos         | Lee, transforma, migra, restaura, corrige o elimina datos.                            | migración SQL, backup, restore, update masivo, reindex, import/export               |
| `OC-ACCESS`    | Acceso, identidad y seguridad | Cambia permisos, secretos, credenciales, roles, reglas o postura de seguridad.        | crear usuario, cambiar IAM, rotar secreto, abrir firewall, modificar RBAC           |
| `OC-REMEDIATE` | Remediación / recuperación    | Acción correctiva para restaurar servicio, seguridad o integridad.                    | rollback, restore, aislar nodo, limpiar cola, reparar índice                        |
| `OC-DESTROY`   | Destrucción / decommission    | Elimina recursos, datos, configuraciones o capacidades.                               | delete VM, drop table, borrar PVC, revocar tenant, purgar logs                      |

La clave es que `OC-OBSERVE` y `OC-ANALYZE` no son automáticamente “seguros”; solo indican que no modifican estado. El riesgo real se calcula con overlays.

---

# 3. Overlays transversales

Los overlays son modificadores que elevan o reducen el rigor.

| Overlay                | Valores recomendados                                         | Qué cambia                                                     |
| ---------------------- | ------------------------------------------------------------ | -------------------------------------------------------------- |
| `Environment`          | dev, test, staging, prod                                     | Prod eleva rigor.                                              |
| `Data Sensitivity`     | none, internal, confidential, regulated, secret-bearing      | Secretos o datos regulados elevan aprobación y evidencia.      |
| `Privilege`            | user, operator, admin, root, break-glass                     | Privilegio alto exige trazabilidad.                            |
| `Availability Impact`  | none, degraded, partial outage, full outage                  | Impacto en disponibilidad eleva flujo.                         |
| `Integrity Impact`     | none, reversible, corruption-risk, destructive               | Riesgo de corrupción exige backup/validación.                  |
| `Security Impact`      | none, posture-change, exposure, privilege-escalation         | Cambios IAM/firewall/secrets son sensibles.                    |
| `Cost Impact`          | none, bounded, variable, runaway-risk                        | Escalados, loops o recursos cloud pueden disparar coste.       |
| `Compliance Impact`    | none, audit-relevant, regulated, legal-hold                  | Aumenta evidencia y retención.                                 |
| `Scope`                | local, single-target, multi-target, fleet-wide, cross-system | Mayor blast radius, más rigor.                                 |
| `Reversibility`        | no-state, easy, backup-required, hard-to-revert, destructive | Define rollback obligatorio.                                   |
| `Execution Mode`       | manual, assisted, semi-auto, automated, autonomous           | Más autonomía requiere más guardrails.                         |
| `Urgency`              | routine, scheduled, urgent, emergency                        | Emergency acelera aprobación, no elimina revisión.             |
| `State Model`          | imperative, declarative, pipeline-backed, GitOps             | Puede cambiar el flujo recomendado.                            |
| `Error Budget State`   | healthy, warning, exhausted                                  | Si el presupuesto está agotado, reducir cambios no esenciales. |
| `Evidence Sensitivity` | normal, redacted, restricted                                 | Evita capturar secretos en evidencias.                         |

---

# 4. Criterios para distinguir bajo riesgo vs crítico

## 4.1 Bajo riesgo

Una operación puede considerarse de bajo riesgo si cumple todas estas condiciones:

```text
- No toca producción crítica.
- No modifica datos persistentes sensibles.
- No usa privilegios elevados.
- No expone secretos, PII, credenciales ni datos regulados.
- Tiene alcance local o single-target.
- Es reversible o no cambia estado.
- Tiene procedimiento conocido.
- Tiene validación simple.
- Su fallo no provoca caída, pérdida de datos, brecha de seguridad ni coste relevante.
```

Ejemplos:

```text
systemctl status nginx
Get-Service Spooler
kubectl get pods -n dev
SELECT count(*) FROM orders;
aws ec2 describe-instances --filters ...
GET /health
```

## 4.2 Crítica

Una operación debería clasificarse como crítica si cumple una o más condiciones:

```text
- Afecta producción crítica.
- Puede causar indisponibilidad amplia.
- Modifica datos de negocio o datos regulados.
- Usa root/admin/Domain Admin/IAM admin/break-glass.
- Cambia permisos, firewall, secretos, cifrado o identidad.
- Es fleet-wide o cross-system.
- Es destructiva o difícil de revertir.
- No tiene rollback probado.
- Puede generar coste no acotado.
- Puede afectar cumplimiento, auditoría o reputación.
```

Ejemplos:

```text
DROP TABLE customers;
kubectl delete pvc --all -n prod;
aws iam attach-role-policy con AdministratorAccess;
Restart-Service sobre muchos servidores de producción;
terraform apply con destrucción de recursos;
rotación de secretos usados por múltiples servicios;
restore de base de datos productiva;
```

---

# 5. Read-only no siempre es bajo riesgo

SDO debería distinguir:

| Tipo read-only                                      |       Riesgo | Ejemplo                                                             |
| --------------------------------------------------- | -----------: | ------------------------------------------------------------------- |
| Read-only técnico sin datos sensibles               |         Bajo | `systemctl status nginx`                                            |
| Read-only operacional con posible metadata sensible |        Medio | `kubectl describe pod`, logs de app                                 |
| Read-only de datos de negocio                       |   Medio/alto | `SELECT email, phone FROM customers`                                |
| Read-only de secretos                               | Alto/crítico | `kubectl get secret -o yaml`, `aws secretsmanager get-secret-value` |
| Read-only masivo/exportable                         |         Alto | dump de tabla, export CSV, descarga de logs completos               |

Regla SDO:

```text
Read-only clasifica por exposición, no por ausencia de escritura.
```

Una consulta que no modifica nada puede ser crítica si permite exfiltrar credenciales, tokens, datos personales, información financiera, logs con secretos o configuraciones internas. OWASP incluye riesgos relacionados con autorización a nivel de objeto/propiedad y exposición de datos en APIs, lo que refuerza esta distinción. ([owasp.org][9])

---

# 6. Reversibilidad

## 6.1 Niveles propuestos

| Nivel  | Nombre                         | Definición                                                                    | Ejemplo                                    |
| ------ | ------------------------------ | ----------------------------------------------------------------------------- | ------------------------------------------ |
| `RV-0` | Sin cambio de estado           | No requiere rollback.                                                         | status, diff, dry-run                      |
| `RV-1` | Reversible simple              | Puede revertirse con comando directo.                                         | restart, enable/disable flag               |
| `RV-2` | Reversible con versión         | Requiere versión previa, snapshot, backup o manifiesto anterior.              | deploy, config change                      |
| `RV-3` | Difícil de revertir            | Requiere migración inversa, ventana, reconstrucción o intervención.           | schema migration compleja                  |
| `RV-4` | Destructiva                    | Borra, purga o invalida estado.                                               | drop, delete, purge                        |
| `RV-5` | Irreversible regulatoria/legal | Aunque técnicamente posible, no debe revertirse sin control legal/compliance. | borrado por retención legal, anonimización |

## 6.2 Regla SDO

```text
Rollback no debe ser un campo de texto; debe ser una capacidad verificable.
```

Para `RV-2` o superior, SDO debería exigir:

```text
- rollback plan;
- pre-checks;
- backup/snapshot o versión anterior;
- criterio claro de activación;
- validación post-rollback;
- responsable autorizado.
```

En `RV-4`, no basta con decir “rollback: restaurar backup”. Hay que verificar que el backup existe, es reciente, restaurable y que el RTO/RPO es aceptable.

---

# 7. Impacto operacional

SDO debería evaluar impacto en seis dimensiones:

| Dimensión      | Pregunta                                                          |
| -------------- | ----------------------------------------------------------------- |
| Disponibilidad | ¿Puede degradar o tumbar un servicio?                             |
| Datos          | ¿Puede exponer, corromper, perder o modificar datos?              |
| Seguridad      | ¿Cambia privilegios, secretos, superficie de ataque o controles?  |
| Coste          | ¿Puede crear consumo variable o no acotado?                       |
| Compliance     | ¿Afecta auditoría, retención, privacidad, licencias o regulación? |
| Reputación     | ¿Puede afectar usuarios, clientes, SLA o confianza pública?       |

Matriz simple:

| Nivel | Criterio                                               |
| ----- | ------------------------------------------------------ |
| `I0`  | Sin impacto observable                                 |
| `I1`  | Impacto local, reversible                              |
| `I2`  | Impacto limitado, requiere validación                  |
| `I3`  | Impacto productivo, datos, seguridad o coste relevante |
| `I4`  | Impacto amplio, crítico, regulado o reputacional       |

---

# 8. Alcance

| Alcance       | Definición                                   | Ejemplo                                    |
| ------------- | -------------------------------------------- | ------------------------------------------ |
| `SC-LOCAL`    | Solo entorno local o laboratorio             | script en máquina personal                 |
| `SC-SINGLE`   | Un host, servicio, pod, DB o recurso         | reiniciar una VM                           |
| `SC-MULTI`    | Varios objetivos acotados                    | actualizar 5 servidores                    |
| `SC-FLEET`    | Flota completa o selector amplio             | `kubectl rollout restart deployment --all` |
| `SC-CROSS`    | Cruza sistemas, regiones, tenants o dominios | IAM + Kubernetes + DB                      |
| `SC-EXTERNAL` | Afecta clientes, terceros o proveedores      | cambio DNS público, API pública            |

Regla:

```text
El alcance multiplica el riesgo.
```

Una operación segura en un host puede ser crítica si se ejecuta sobre una flota.

---

# 9. Manual, semiautomática y automática

| Modo            | Definición                                     | Riesgo típico                                     |
| --------------- | ---------------------------------------------- | ------------------------------------------------- |
| `EX-MANUAL`     | Humano ejecuta pasos directamente.             | Error humano, baja repetibilidad.                 |
| `EX-ASSISTED`   | IA o herramienta sugiere, humano ejecuta.      | Riesgo de confianza excesiva.                     |
| `EX-SEMI-AUTO`  | Automatización ejecuta con aprobación humana.  | Buen equilibrio para riesgo medio/alto.           |
| `EX-AUTO`       | Automatización ejecuta por trigger o pipeline. | Requiere guardrails, logs, idempotencia.          |
| `EX-AUTONOMOUS` | Agente decide y ejecuta.                       | Solo aceptable con límites fuertes y bajo riesgo. |

Regla SDO:

```text
Más automatización no reduce el riesgo por sí sola.
Reduce variabilidad, pero aumenta blast radius si el control es malo.
```

Para operaciones con IA, la autonomía debe limitarse por riesgo, alcance, sensibilidad y reversibilidad. El criterio de supervisión proporcional al riesgo y autonomía coincide con el enfoque del AI Act para sistemas de alto riesgo. ([ai-act-service-desk.ec.europa.eu][8])

---

# 10. Operaciones bajo incidente o emergencia

SDO debería tratar `Emergency` como overlay, no como clase base.

Ejemplo:

```text
OC-CONTROL + Emergency + Production + Availability Impact
```

Una emergencia puede ser:

```text
- reiniciar servicio crítico;
- aplicar parche urgente;
- bloquear IP;
- revocar token comprometido;
- aislar host;
- restaurar backup;
- deshabilitar integración externa.
```

Reglas:

```text
- La aprobación puede ser acelerada.
- La evidencia mínima no desaparece.
- El rollback puede ser más simple, pero debe existir si aplica.
- La revisión posterior es obligatoria.
- Debe quedar claro quién autorizó, quién ejecutó, por qué y con qué resultado.
```

Esto está alineado con ITIL emergency changes y con guías modernas de respuesta a incidentes: NIST SP 800-61r3 integra la respuesta a incidentes dentro de la gestión de riesgo de ciberseguridad, y Google SRE enfatiza roles, coordinación, comunicación y postmortems sin culpa. ([ITSM.tools][1])

---

# 11. Perfiles de flujo SDO

## 11.1 `SDO-Lightweight`

Para operaciones simples, acotadas y de bajo riesgo.

```text
Intent → Mini-Spec → Execution → Validation → Evidence mínima
```

Uso:

```text
- read-only no sensible;
- mantenimiento estándar;
- operación local;
- procedimiento preaprobado;
- reversibilidad simple.
```

## 11.2 `SDO-Normal`

Para cambios operacionales habituales.

```text
Intent → Spec → Plan → Runbook → Execution → Validation → Review
```

Uso:

```text
- cambios de configuración;
- reinicios productivos acotados;
- despliegues menores;
- cambios de permisos no críticos;
- operaciones con backup claro.
```

## 11.3 `SDO-Critical`

Para alto impacto, datos, seguridad, alcance amplio o difícil rollback.

```text
Intent
→ Spec completa
→ Risk/Approval
→ Plan detallado
→ Runbook probado
→ Pre-checks
→ Execution controlada
→ Validation
→ Evidence
→ Rollback readiness
→ Review formal
```

Uso:

```text
- IAM/admin;
- producción crítica;
- bases de datos productivas;
- operaciones destructivas;
- fleet-wide;
- cross-system;
- datos regulados.
```

## 11.4 `SDO-Emergency`

Para incidentes o riesgo inminente.

```text
Intent urgente
→ Emergency Spec mínima
→ Approval acelerada
→ Execution
→ Validation inmediata
→ Evidence
→ Stabilization
→ Post-review obligatorio
```

Uso:

```text
- mitigar incidente;
- parche urgente;
- aislar sistema comprometido;
- rollback de producción;
- rotar secreto comprometido.
```

## 11.5 `SDO-Pipeline-Backed`

Para GitOps, IaC, CI/CD o automatización versionada.

```text
Intent
→ Spec como código
→ Plan/Diff
→ Review/Gates
→ Apply
→ Automated Validation
→ Evidence automática
→ Rollback/Roll-forward
→ Review si falla
```

Uso:

```text
- Terraform;
- Kubernetes manifests;
- Helm;
- GitOps;
- pipelines CI/CD;
- políticas versionadas.
```

Terraform y Kubernetes ya encajan parcialmente con este flujo: Terraform permite revisar planes antes de aplicar, y Kubernetes permite aplicar configuración declarativa y previsualizar diferencias con `kubectl diff`; pero SDO debe añadir clasificación de riesgo, evidencias y decisión de rollback. ([HashiCorp Developer][10])

---

# 12. Matriz de decisión base

| Clase / Overlay                              | Aprobación requerida                       | Evidencia mínima                              | Rollback requerido                             | Flujo recomendado           |
| -------------------------------------------- | ------------------------------------------ | --------------------------------------------- | ---------------------------------------------- | --------------------------- |
| `OC-OBSERVE` sin sensibilidad                | No o autoaprobación                        | comando/consulta + resultado resumido         | No                                             | Lightweight                 |
| `OC-OBSERVE` con secretos/PII/logs sensibles | Owner o seguridad/datos                    | evidencia redactada + quién accedió + motivo  | No, pero sí control de exposición              | Normal/Critical             |
| `OC-ANALYZE` plan/diff                       | Peer review si prod                        | plan/diff guardado                            | No aplica                                      | Lightweight/Pipeline-backed |
| `OC-MAINTAIN` estándar                       | Preaprobada                                | ejecución + resultado                         | Simple                                         | Lightweight                 |
| `OC-CONFIGURE` single-target                 | Peer/owner                                 | antes/después + validación                    | Sí                                             | Normal                      |
| `OC-CONFIGURE` prod crítica o multi-target   | Change authority + owner                   | plan, diff, ventana, validación               | Sí, probado                                    | Critical                    |
| `OC-DEPLOY` con pipeline                     | PR/review/gates                            | commit, build, plan, logs, health checks      | Rollback o roll-forward                        | Pipeline-backed             |
| `OC-CONTROL` restart/scale single-target     | Owner o preaprobada                        | timestamp, target, estado antes/después       | Sí, simple                                     | Normal                      |
| `OC-CONTROL` fleet-wide                      | Change authority                           | lista targets, ventana, métricas              | Sí, por fases                                  | Critical                    |
| `OC-DATA` read-only no sensible              | Auto/peer                                  | consulta + resultado agregado                 | No                                             | Lightweight                 |
| `OC-DATA` lectura sensible/export            | Data owner/security                        | justificación, dataset, redacción, destino    | No, pero control de distribución               | Critical                    |
| `OC-DATA` schema/data migration              | DBA/app owner                              | backup, script, conteos, validación           | Backup o migración inversa                     | Critical                    |
| `OC-ACCESS` permisos bajos                   | Owner                                      | cambio antes/después                          | Sí                                             | Normal                      |
| `OC-ACCESS` admin/root/IAM/secrets           | Security owner + change authority          | actor, permiso, motivo, expiración, logs      | Sí, revocación/rotación                        | Critical                    |
| `OC-REMEDIATE` rutinaria                     | Owner                                      | síntoma, acción, resultado                    | Según clase subyacente                         | Normal                      |
| `OC-REMEDIATE` incidente activo              | Incident commander o autoridad emergencia  | timeline, decisión, impacto, resultado        | Si aplica                                      | Emergency                   |
| `OC-DESTROY` no productivo                   | Owner                                      | target, confirmación, resultado               | Si existe snapshot                             | Normal                      |
| `OC-DESTROY` prod/datos/seguridad            | Change authority + owner + seguridad/datos | backup verificado, plan, aprobación explícita | Obligatorio o aceptación formal de no rollback | Critical                    |

---

# 13. Ejemplos agnósticos por tecnología

| Tecnología | Operación                                  | Clase                          | Overlays                    | Flujo              |
| ---------- | ------------------------------------------ | ------------------------------ | --------------------------- | ------------------ |
| Linux      | `systemctl status nginx`                   | `OC-OBSERVE`                   | single-target, no-sensitive | Lightweight        |
| Linux      | `journalctl -u app` con tokens posibles    | `OC-OBSERVE`                   | sensitive logs              | Normal             |
| Linux      | `systemctl restart nginx` en prod          | `OC-CONTROL`                   | prod, availability          | Normal             |
| Linux      | actualización de paquetes en 50 servidores | `OC-MAINTAIN` / `OC-CONFIGURE` | fleet-wide                  | Critical           |
| Windows    | `Get-Service`                              | `OC-OBSERVE`                   | no-sensitive                | Lightweight        |
| Windows    | modificar GPO de seguridad                 | `OC-CONFIGURE` / `OC-ACCESS`   | security, multi-target      | Critical           |
| Windows    | añadir usuario a Domain Admins             | `OC-ACCESS`                    | privileged, security        | Critical           |
| SQL        | `SELECT count(*)`                          | `OC-OBSERVE`                   | no-sensitive                | Lightweight        |
| SQL        | `SELECT * FROM customers`                  | `OC-DATA`                      | PII/data exposure           | Critical           |
| SQL        | `ALTER TABLE` productivo                   | `OC-DATA`                      | integrity, availability     | Critical           |
| SQL        | `DROP TABLE`                               | `OC-DESTROY`                   | destructive                 | Critical           |
| Kubernetes | `kubectl get pods`                         | `OC-OBSERVE`                   | namespace scope             | Lightweight        |
| Kubernetes | `kubectl get secret -o yaml`               | `OC-OBSERVE`                   | secrets                     | Critical           |
| Kubernetes | `kubectl rollout restart deployment/api`   | `OC-CONTROL`                   | prod, availability          | Normal             |
| Kubernetes | `kubectl delete pvc`                       | `OC-DESTROY`                   | data loss                   | Critical           |
| Cloud CLI  | `aws ec2 describe-instances`               | `OC-OBSERVE`                   | metadata                    | Lightweight/Normal |
| Cloud CLI  | cambiar Security Group público             | `OC-ACCESS` / `OC-CONFIGURE`   | security exposure           | Critical           |
| Cloud CLI  | escalar ASG de 2 a 20                      | `OC-CONTROL`                   | cost, availability          | Normal/Critical    |
| Cloud CLI  | borrar RDS                                 | `OC-DESTROY`                   | destructive, data           | Critical           |
| API        | `GET /health`                              | `OC-OBSERVE`                   | no-sensitive                | Lightweight        |
| API        | `POST /users/{id}/disable`                 | `OC-ACCESS` / `OC-CONFIGURE`   | identity impact             | Normal             |
| API        | `DELETE /tenant/{id}`                      | `OC-DESTROY`                   | cross-system, destructive   | Critical           |
| Manual     | reiniciar appliance desde GUI              | `OC-CONTROL`                   | manual, prod                | Normal             |
| Manual     | cambiar cable en switch core               | `OC-CONTROL`                   | physical, availability      | Critical           |
| Manual     | llamar proveedor para failover             | `OC-REMEDIATE`                 | emergency/cross-system      | Emergency          |

---

# 14. Reglas de clasificación recomendadas

## 14.1 Regla principal

```text
Primero clasifica la clase base.
Después aplica overlays.
El overlay más severo domina.
```

Ejemplo:

```text
kubectl get secret -o yaml
```

Clase base:

```text
OC-OBSERVE
```

Overlays:

```text
Data Sensitivity = secret-bearing
Privilege = elevated
Environment = prod
Evidence Sensitivity = redacted
```

Resultado:

```text
SDO-Critical
```

No debe ser `Lightweight`.

## 14.2 Regla de severidad por máximo

```text
Risk Tier = max(
  Availability,
  Data,
  Security,
  Cost,
  Compliance,
  Reversibility,
  Scope,
  Privilege,
  Urgency
)
```

Esto evita promedios engañosos. Una operación puede ser “simple” técnicamente, pero crítica por seguridad.

## 14.3 Regla de emergencia

```text
Emergency reduce tiempo de aprobación, no reduce responsabilidad.
```

## 14.4 Regla de pipeline

```text
Pipeline-backed no significa bajo riesgo.
```

Un `terraform apply` con destrucción de recursos sigue siendo crítico aunque esté automatizado. Terraform documenta explícitamente el modo destroy y advierte que destruir infraestructura no tiene undo directo. ([HashiCorp Developer][11])

---

# 15. Anti-patrones de clasificación

1. **Clasificar por herramienta.**
   “Es Kubernetes, entonces es crítico” o “es Bash, entonces es simple” es incorrecto. Importa el efecto.

2. **Tratar todo read-only como bajo riesgo.**
   Leer secretos, PII o logs sensibles puede ser crítico.

3. **Confundir automatización con seguridad.**
   Automatizar reduce variabilidad, pero puede amplificar errores.

4. **Usar emergency para saltarse controles.**
   Emergency debe dejar más evidencia posterior, no menos.

5. **Rollback teatral.**
   Escribir “se restaura backup” sin verificar backup, RTO y procedimiento no es rollback real.

6. **Ignorar blast radius.**
   Un comando seguro en un host puede ser peligroso con `--all`, wildcard, selector o loop.

7. **Meter cambios de datos dentro de despliegues normales.**
   Una migración de datos no debería quedar oculta como “deploy”.

8. **No separar evidencia de secretos.**
   Capturar pantallazos, logs o outputs con tokens puede crear una segunda fuga.

9. **Aprobar por burocracia, no por riesgo.**
   Demasiada aprobación para bajo riesgo genera spec theater.

10. **No distinguir hard-to-revert de destructive.**
    No todo lo difícil de revertir es destructivo, pero ambos requieren control fuerte.

11. **Clasificar solo por entorno.**
    Dev con datos reales puede ser más sensible que prod sin datos personales.

12. **Olvidar coste.**
    Cloud permite errores caros sin afectar disponibilidad.

---

# 16. Nombres recomendados

Evitar nombres ambiguos como:

```text
minor / medium / major
simple / complex
safe / unsafe
manual / automatic
```

Usar nombres consistentes:

## Clases

```text
Observe
Analyze
Maintain
Configure
Deploy
Control
Data
Access
Remediate
Destroy
```

## Overlays

```text
Sensitivity
Privilege
Scope
Reversibility
Availability Impact
Data Impact
Security Impact
Cost Impact
Compliance Impact
Execution Mode
Urgency
State Model
Evidence Sensitivity
```

## Flujos

```text
SDO-Lightweight
SDO-Normal
SDO-Critical
SDO-Emergency
SDO-Pipeline-Backed
```

---

# 17. Modelo final recomendado

```text
SDO Operation Classification
├── Base Class
│   ├── Observe
│   ├── Analyze
│   ├── Maintain
│   ├── Configure
│   ├── Deploy
│   ├── Control
│   ├── Data
│   ├── Access
│   ├── Remediate
│   └── Destroy
│
├── Overlays
│   ├── Environment
│   ├── Sensitivity
│   ├── Privilege
│   ├── Scope
│   ├── Reversibility
│   ├── Availability Impact
│   ├── Data Impact
│   ├── Security Impact
│   ├── Cost Impact
│   ├── Compliance Impact
│   ├── Execution Mode
│   ├── Urgency
│   ├── State Model
│   └── Evidence Sensitivity
│
└── Flow Profile
    ├── Lightweight
    ├── Normal
    ├── Critical
    ├── Emergency
    └── Pipeline-Backed
```

---

# 18. Definición compacta para SDO

**SDO clasifica una operación por su efecto operacional base y por overlays de riesgo, sensibilidad, alcance, reversibilidad, privilegio, urgencia y modo de ejecución. Esa clasificación determina el flujo, aprobación, evidencia, validación y rollback requeridos.**

Esta sería una buena formulación:

```text
An SDO operation is not classified by the tool used to perform it,
but by the operational state it observes or changes, the sensitivity
of what it can expose, the blast radius it can affect, the privilege it
requires, and the reversibility of its outcome.
```

Mi recomendación: esta taxonomía debería convertirse en el **núcleo del modelo SDO**, antes de definir plantillas de Spec o formatos de Runbook.

[1]: https://itsm.tools/why-what-change-management/?utm_source=chatgpt.com "Why & What Change Management Is in ITSM | ITIL Change Guide"
[2]: https://sre.google/sre-book/embracing-risk/?utm_source=chatgpt.com "Google SRE - Embracing risk and reliability engineering book"
[3]: https://dora.dev/guides/dora-metrics/?utm_source=chatgpt.com "DORA | DORA’s software delivery performance metrics"
[4]: https://opengitops.dev/?utm_source=chatgpt.com "Home | OpenGitOps"
[5]: https://docs.aws.amazon.com/en_us/systems-manager/latest/userguide/automation-documents.html?utm_source=chatgpt.com "Creating your own runbooks - AWS Systems Manager"
[6]: https://www.cisa.gov/resources-tools/resources/zero-trust-maturity-model?utm_source=chatgpt.com "Zero Trust Maturity Model | CISA"
[7]: https://www.postgresql.org/docs/14/sql-rollback.html?utm_source=chatgpt.com "PostgreSQL: Documentation: 14: ROLLBACK"
[8]: https://ai-act-service-desk.ec.europa.eu/en/ai-act/article-14?utm_source=chatgpt.com "Article 14: Human oversight | AI Act Service Desk"
[9]: https://owasp.org/API-Security/editions/2023/en/0x11-t10/?utm_source=chatgpt.com "OWASP Top 10 API Security Risks – 2023 - OWASP API Security Top 10"
[10]: https://developer.hashicorp.com/terraform/cli/commands/plan?utm_source=chatgpt.com "terraform plan command reference | Terraform | HashiCorp Developer"
[11]: https://developer.hashicorp.com/terraform/tutorials/cli/plan?utm_source=chatgpt.com "Create a Terraform plan | Terraform | HashiCorp Developer"
