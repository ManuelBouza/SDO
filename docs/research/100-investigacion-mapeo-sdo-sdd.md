> Research note — non-normative.
> This document is background material and may be superseded by current SDO core/reference documentation.
> Current normative docs start at [../start-here.md](../start-here.md).

## Síntesis

SDO puede heredar de SDD la idea de **trabajar desde una especificación revisable antes de ejecutar**, pero no debe copiar su ciclo literalmente. En SDD, la salida principal es código; en SDO, la salida principal es **un cambio operacional controlado sobre un sistema vivo**.

La diferencia central:

> **SDD convierte intención en código verificable.
> SDO convierte intención operacional en ejecución controlada, evidencia, validación y recuperación segura.**

SDD actual no es un estándar único; hay variantes como *spec-first*, *spec-anchored* y *spec-as-source*. Esa distinción es útil para SDO, pero debe adaptarse porque en operaciones la realidad del sistema siempre debe verificarse contra evidencia, no asumirse desde la spec. ([martinfowler.com][1])

---

## 1. Mapa conceptual SDD → SDO

| SDD concept                 | SDO equivalent                           | Diferencia clave                                                                                          |
| --------------------------- | ---------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| Proposal / intent           | **Operational Intent**                   | En SDO incluye urgencia, impacto, sistema afectado, ventana, actor y motivo operativo.                    |
| Spec                        | **Operational Spec**                     | No describe una feature; describe estado esperado, alcance, límites, riesgos, validación y recuperación.  |
| Design / plan               | **Execution Plan**                       | No diseña arquitectura de software; define estrategia operacional segura.                                 |
| Tasks                       | **Planned Actions**                      | No son tareas de programación; son acciones operativas ordenadas, verificables e interrumpibles.          |
| Implementation              | **Actual Execution**                     | No es escribir código; es actuar sobre infraestructura, servicios, datos, accesos o configuración real.   |
| Tests                       | **Validation Checks**                    | No solo prueban comportamiento; verifican salud, estado, evidencias, drift, impacto y criterios de éxito. |
| Archive                     | **Closure / Evidence Archive**           | No solo archiva una feature; cierra trazabilidad, evidencias, aprobación, resultado y lecciones.          |
| Source of truth             | **Spec as source of operational intent** | La spec gobierna intención y autorización; la realidad se confirma con evidencia.                         |
| Coding agent                | **Executor / Assistant / Operator**      | En SDO se separan AI, humano aprobador, executor y validador.                                             |
| TDD/BDD acceptance criteria | **Operational Acceptance Criteria**      | Se traducen a checks: precondición, acción esperada, señal de éxito, señal de fallo y rollback trigger.   |

---

## 2. Qué se puede reutilizar directamente de SDD

De SDD conviene reutilizar:

1. **Spec-first**: no ejecutar sin intención y alcance claros.
2. **Spec como contrato**: la ejecución debe estar anclada a lo aprobado.
3. **Plan antes de apply**: equivalente a preview, dry-run, diff o execution plan.
4. **Tasks pequeñas y trazables**: en SDO serían acciones operativas atómicas.
5. **Acceptance criteria**: convertidos en validaciones operacionales.
6. **Iteración controlada**: si cambia el alcance, se vuelve a la spec.
7. **Archivo de decisiones**: útil para auditoría y aprendizaje.

GitHub Spec Kit usa comandos como `specify`, `plan`, `tasks` e `implement`, y OpenSpec organiza cambios con `proposal.md`, `design.md`, `tasks.md` y specs delta; ambos refuerzan la idea de separar intención, diseño, tareas e implementación. ([GitHub][2])

---

## 3. Qué NO debe copiar SDO literalmente

| Concepto SDD                       | Por qué no copiarlo igual en SDO                                                                                                          |
| ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `implementation = escribir código` | En SDO la ejecución puede ser CLI, GUI, API, pipeline, SQL, script, ticket, runbook o intervención humana.                                |
| `tests = unit/integration tests`   | En operaciones se necesitan health checks, queries, métricas, logs, evidencias y validación de impacto.                                   |
| `archive = mover spec a histórico` | En SDO archive debe incluir evidencia, resultado real, desviaciones, aprobación y estado final.                                           |
| `spec-as-source` absoluto          | En operaciones la spec no sustituye al estado real; puede haber drift, fallos parciales y efectos externos.                               |
| Ciclo lineal rígido                | Emergencias, cambios estándar y pipelines requieren variantes de rigor.                                                                   |
| “AI implementa”                    | En SDO AI no debe ser automáticamente executor; puede proponer, analizar o generar runbooks, pero la responsabilidad sigue siendo humana. |

---

## 4. Equivalencias clave

### “Tasks” en SDO

El equivalente oficial debería ser:

> **Planned Actions**

Una **Planned Action** es una acción operacional concreta, ordenada, reversible o contenible, con precondición, comando/procedimiento, expected outcome, evidencia esperada y criterio de parada.

Ejemplo conceptual:

```text
PA-03 Restart service X on node Y
Precondition: node Y drained or no active sessions
Executor: systemctl / API / pipeline / human
Expected outcome: service active within 30s
Evidence: command output + log timestamp + health endpoint
Stop condition: service fails twice or dependent check degrades
Rollback/recovery: restore previous service state or fail over
```

### “Implementation” en SDO

El equivalente debe ser:

> **Actual Execution**

No es “implementar”, sino **ejecutar el plan aprobado contra un sistema real**.

Terraform separa claramente `plan` como vista previa de cambios y `apply` como ejecución de las operaciones propuestas; esa separación es útil como analogía parcial para SDO. ([HashiCorp Developer][3])

### “Tests” en SDO

El equivalente debe ser:

> **Validation Checks**

Incluyen:

* pre-checks;
* post-checks;
* health checks;
* métricas;
* logs;
* queries;
* comparación estado esperado vs estado real;
* verificación de no-regresión operacional;
* confirmación de evidencia.

BDD/Cucumber aporta una idea reutilizable: especificaciones ejecutables en lenguaje estructurado que validan que el sistema cumple lo especificado. En SDO esto se transforma en checks operacionales, no necesariamente tests de software. ([Cucumber][4])

### “Archive” en SDO

El equivalente debe ser:

> **Evidence Closure / Operational Archive**

Debe archivar:

* Operational Spec final;
* Execution Plan;
* Runbook usado;
* Execution Record;
* Evidence Package;
* Validation Report;
* aprobaciones;
* desviaciones;
* rollback/recovery aplicado o disponible;
* Review Record.

---

## 5. Tabla SDD artifact → SDO artifact

| SDD artifact           | SDO artifact                                 |
| ---------------------- | -------------------------------------------- |
| PRD / feature request  | Operational Intent                           |
| Spec / requirements    | Operational Spec                             |
| User stories           | Operational scenarios / operational outcomes |
| Acceptance criteria    | Validation criteria                          |
| Design doc             | Execution Plan                               |
| Architecture decisions | Operational constraints / risk decisions     |
| `tasks.md`             | Planned Actions                              |
| Test suite             | Validation Checks                            |
| Implementation diff    | Execution Record                             |
| Build logs             | Execution evidence                           |
| Pull request           | Change request / approval record             |
| Archive folder         | Evidence Package + Review Record             |
| Project constitution   | SDO Operating Principles / Governance Policy |

---

## 6. Tabla SDD phase → SDO phase

| SDD phase              | SDO phase oficial propuesta | Comentario                                                            |
| ---------------------- | --------------------------- | --------------------------------------------------------------------- |
| Proposal / request     | **Intent**                  | Captura necesidad operacional.                                        |
| Specify                | **Operational Spec**        | Define qué debe lograrse y bajo qué límites.                          |
| Design / plan          | **Execution Plan**          | Define estrategia, dependencias, riesgos y validación.                |
| Tasks                  | **Planned Actions**         | Descompone el plan en acciones ejecutables.                           |
| Implementation / apply | **Execution**               | Ejecuta mediante humano, script, CLI, API, pipeline o automatización. |
| Tests / verify         | **Validation**              | Verifica resultado y ausencia de impacto no aceptado.                 |
| Archive                | **Review & Closure**        | Cierra evidencia, aprendizaje, rollback y estado final.               |

---

## 7. Adaptación de “spec as source of truth”

En SDD puede decirse que la spec es fuente de verdad para lo que el código debe hacer. GitHub describe SDD como un proceso donde la spec actúa como contrato y fuente de verdad para generar, probar y validar código. ([The GitHub Blog][5])

En SDO debe formularse distinto:

> **La Operational Spec es la fuente de verdad de la intención aprobada, el alcance, los límites, los criterios de éxito, el riesgo aceptado y la estrategia de recuperación.
> La evidencia es la fuente de verdad de lo que realmente ocurrió.**

Esto evita un error grave: creer que porque algo está especificado, el sistema real ya coincide.

GitOps sí usa estado deseado declarativo, versionado y reconciliado automáticamente, pero SDO debe ser más amplio porque también cubre operaciones manuales, diagnósticos, accesos, datos, incidentes y cambios no declarativos. ([Open GitOps][6])

---

## 8. Cómo evitar que SDO sea “SDD con comandos”

SDO no debe partir de “qué comando ejecutar”, sino de:

1. qué estado operacional se quiere lograr;
2. qué riesgo introduce;
3. qué evidencia probará el resultado;
4. qué límites no deben cruzarse;
5. cómo detenerse;
6. cómo recuperar un estado seguro;
7. quién aprueba;
8. quién ejecuta;
9. quién valida.

Los comandos son solo una posible forma de **Executor Binding**.

---

## 9. Integración de risk, approval, evidence y rollback

Aquí SDO se separa claramente de SDD.

| Control  | En SDD típico               | En SDO                                                                                  |
| -------- | --------------------------- | --------------------------------------------------------------------------------------- |
| Risk     | Técnico o de producto       | Operacional: disponibilidad, datos, seguridad, continuidad, cumplimiento.               |
| Approval | Revisión de PR/spec         | Aprobación proporcional al perfil de riesgo.                                            |
| Evidence | Tests, logs CI, diff        | Evidencia mínima verificable de ejecución y resultado.                                  |
| Rollback | Revert commit / fix forward | Recovery seguro: restore, failover, compensación, contención, rebuild, manual recovery. |

ITIL distingue cambios estándar, normales y de emergencia, y recomienda adaptar flujo y aprobación al riesgo; esa idea encaja directamente con los perfiles SDO. ([atlassian.com][7])

---

## 10. Lifecycle SDO oficial propuesto

Mi propuesta final de fases oficiales:

```text
Intent
→ Operational Spec
→ Risk & Profile
→ Execution Plan
→ Planned Actions
→ Runbook / Executor Binding
→ Approval
→ Pre-Execution Check
→ Execution
→ Validation
→ Evidence Closure
→ Review / Recovery / Archive
```

Versión compacta compatible con tu modelo base:

```text
Intent
→ Spec
→ Plan
→ Runbook
→ Execution
→ Validation
→ Review
```

Pero internamente, los controles transversales deben existir siempre:

```text
Risk / Approval
Evidence
Rollback / Recovery
AI Governance
```

---

## 11. Variantes del lifecycle

### SDO-Full

Para cambios normales o críticos.

```text
Intent
→ Operational Spec
→ Risk/Profile Gate
→ Execution Plan
→ Planned Actions
→ Runbook
→ Approval
→ Pre-Execution Check
→ Execution
→ Validation
→ Evidence Closure
→ Review
```

### SDO-Lightweight

Para cambios estándar, repetibles y de bajo riesgo.

```text
Intent
→ Lightweight Spec
→ Pre-approved Runbook
→ Execution
→ Validation
→ Evidence Closure
```

### SDO-Emergency

Para restaurar servicio o contener impacto.

```text
Emergency Intent
→ Minimal Spec
→ Risk Snapshot
→ Emergency Approval / Break-glass
→ Execution
→ Validation
→ Evidence Capture
→ Post-Execution Review
→ Full Retrospective Spec Update
```

### SDO-Pipeline-Backed

Para GitOps, IaC, CI/CD o automatización.

```text
Intent / Change Request
→ Operational Spec
→ Plan / Diff / Preview
→ Policy Checks
→ Approval
→ Pipeline Execution
→ Automated Validation
→ Evidence Package
→ Review / Archive
```

Terraform ofrece una analogía fuerte para esta variante: `write → plan → apply`, donde `plan` permite previsualizar y revisar cambios antes de aplicar. ([HashiCorp Developer][8])

---

## 12. Qué parte es herencia de SDD y qué parte es propia de operaciones

| Herencia de SDD                     | Propio de SDO                                     |
| ----------------------------------- | ------------------------------------------------- |
| Spec-first                          | Risk/Profile Gate                                 |
| Spec como contrato                  | Approval proporcional                             |
| Design/plan antes de ejecutar       | Evidence Package                                  |
| Tasks trazables                     | Rollback/recovery levels R0–R7                    |
| Acceptance criteria                 | Operational Validation                            |
| Archive histórico                   | Auditability                                      |
| AI como asistente de especificación | AI autonomy governance                            |
| Iteración sobre spec                | Stop/continue gates                               |
| Separar qué / cómo                  | Separar intent / approval / executor / validation |

---

## 13. Cómo describir SDO a alguien que ya entiende SDD

> **SDO es a operaciones lo que SDD es al desarrollo, pero adaptado a sistemas vivos.
> En vez de convertir una spec en código, SDO convierte una spec operacional en un plan ejecutable, validado, auditable y recuperable.
> La diferencia es que en operaciones no basta con generar el resultado: hay que controlar riesgo, aprobar, ejecutar con límites, capturar evidencia, validar el estado real y mantener capacidad de recuperación.**

---

## 14. Diferencia fundamental: construir software vs operar sistemas reales

| Construir software                           | Operar sistemas reales                                                            |
| -------------------------------------------- | --------------------------------------------------------------------------------- |
| El artefacto principal es código.            | El artefacto principal es el estado operacional.                                  |
| El fallo suele detectarse con tests.         | El fallo puede afectar usuarios, datos, disponibilidad o seguridad.               |
| Revertir puede ser un commit.                | Recuperar puede requerir restore, failover, compensación o intervención manual.   |
| El entorno puede ser reproducible.           | El entorno puede tener drift, carga real, dependencias externas y estado mutable. |
| La implementación crea algo.                 | La ejecución cambia algo que ya existe.                                           |
| El riesgo principal es defectos de software. | El riesgo principal es impacto operacional.                                       |

---

## 15. Anti-patrones a evitar

| Anti-patrón                 | Riesgo                                                                      |
| --------------------------- | --------------------------------------------------------------------------- |
| **SDD con comandos**        | Reduce SDO a una lista de shell scripts.                                    |
| **Spec theater**            | Mucha documentación, poca seguridad real.                                   |
| **Approval washing**        | Aprobaciones formales sin análisis de riesgo.                               |
| **Evidence afterthought**   | Buscar evidencias después de ejecutar.                                      |
| **Rollback naïve**          | Creer que rollback es solo “deshacer”.                                      |
| **AI executor sin límites** | Mezcla propuesta, aprobación y ejecución.                                   |
| **One lifecycle fits all**  | Burocracia para cambios simples y ligereza peligrosa para cambios críticos. |
| **Plan sin pre-checks**     | Ejecutar sobre supuestos no verificados.                                    |
| **Validation superficial**  | Confirmar que el comando terminó, no que el sistema quedó bien.             |
| **Archive muerto**          | Guardar documentos sin aprendizaje ni trazabilidad útil.                    |

---

## 16. Definición final propuesta de SDO

> **Spec-Driven Operations es una metodología agnóstica para gobernar operaciones TI mediante una Operational Spec verificable que transforma una intención operacional en un plan de ejecución seguro, aprobado, trazable, validado y recuperable.
> SDO no reemplaza ITIL, SRE, GitOps, IaC, runbooks ni incident response; los orquesta mediante un ciclo común de Intent, Spec, Plan, Runbook, Execution, Validation y Review, reforzado por controles transversales de riesgo, aprobación, evidencia, rollback y gobernanza de IA.**

Esa definición mantiene compatibilidad mental con SDD, pero deja claro que SDO pertenece al mundo operacional, no al desarrollo de software.

[1]: https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html "Understanding Spec-Driven-Development: Kiro, spec-kit, and Tessl"
[2]: https://github.com/github/spec-kit "GitHub - github/spec-kit:  Toolkit to help you get started with Spec-Driven Development · GitHub"
[3]: https://developer.hashicorp.com/terraform/cli/commands/plan "terraform plan command reference | Terraform | HashiCorp Developer"
[4]: https://cucumber.io/docs/ "Introduction | Cucumber"
[5]: https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/ "Spec-driven development with AI: Get started with a new open source toolkit - The GitHub Blog"
[6]: https://opengitops.dev/ "Home | OpenGitOps"
[7]: https://www.atlassian.com/itsm/change-management "IT Change Management: ITIL Framework & Best Practices | Atlassian"
[8]: https://developer.hashicorp.com/terraform/intro/core-workflow "Overview of the core Terraform workflow | Terraform | HashiCorp Developer"
