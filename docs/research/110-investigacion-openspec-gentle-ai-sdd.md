> Research note — non-normative.
> This document is background material and may be superseded by current SDO core/reference documentation.
> Current normative docs start at [../start-here.md](../start-here.md).

## Tesis principal

Para SDO, **OpenSpec debería ser la inspiración primaria del modelo de artefactos**, porque estructura muy bien el ciclo `proposal → specs → design → tasks → apply → verify → archive`, separa intención, comportamiento, diseño y tareas, y conserva historial auditable. **Gentle AI debería ser inspiración secundaria para el modelo de agentes, skills, memoria y perfiles de ejecución**, pero no para definir el núcleo metodológico de SDO.

La idea clave: **SDO no debería copiar SDD como “desarrollo de software”, sino adaptar su disciplina de especificación a operaciones TI**.

---

# 1. OpenSpec

## 1.1 Filosofía general

OpenSpec se define como una capa ligera de especificación para que humanos y agentes acuerden qué se va a construir antes de escribir código. Su README resume la filosofía como: fluido, iterativo, fácil, útil para brownfield y escalable de proyectos personales a empresas. También insiste en que cada cambio vive en una carpeta con `proposal`, `specs`, `design` y `tasks`, sin gates rígidos. ([GitHub][1])

**Lectura para SDO:**
Esto encaja muy bien con operaciones: una operación no siempre necesita un flujo pesado, pero sí necesita una unidad clara de cambio, intención, alcance, plan, ejecución, validación y cierre.

## 1.2 Workflow

OpenSpec tiene dos modos principales:

| Modo                | Flujo                                                      | Lectura para SDO                                         |                                                                     |
| ------------------- | ---------------------------------------------------------- | -------------------------------------------------------- | ------------------------------------------------------------------- |
| Core / rápido       | `/opsx:propose → /opsx:apply → /opsx:sync → /opsx:archive` | Útil como equivalente a `SDO-Lightweight` o `SDO-Normal` |                                                                     |
| Expanded / completo | `/opsx:new → /opsx:ff                                      | continue → /opsx:apply → /opsx:verify → /opsx:archive`   | Útil como base para `SDO-Critical` o cambios con revisión explícita |

OpenSpec habla de “acciones, no fases”: los comandos son cosas que puedes hacer, no etapas bloqueantes. Las dependencias habilitan el siguiente artefacto, pero no imponen un waterfall rígido. ([GitHub][2])

**Adaptación SDO:**
SDO puede mantener fases conceptuales, pero sus comandos deberían funcionar como acciones: `sdo-spec`, `sdo-plan`, `sdo-runbook`, `sdo-execute`, `sdo-validate`, etc.

## 1.3 Artefactos

OpenSpec usa este flujo de artefactos:

```text
proposal → specs → design → tasks → implement
why        what    how      steps
```

El `proposal.md` captura intención, alcance y aproximación; `specs/` captura requisitos y escenarios; `design.md` captura decisiones técnicas; `tasks.md` funciona como checklist de implementación. ([GitHub][3])

**Equivalente SDO recomendado:**

```text
intent → operational-spec → plan → runbook → execution-record → validation-report → review/archive
```

OpenSpec separa muy bien **qué debe ocurrir** de **cómo se hará**. Eso es esencial para SDO.

## 1.4 Proposal / spec / design / tasks / apply / verify / archive

| Elemento OpenSpec | Qué hace                                           | Equivalente SDO                                     |
| ----------------- | -------------------------------------------------- | --------------------------------------------------- |
| `proposal.md`     | Intención, alcance, enfoque                        | `intent.md` o sección `Intent` del `operation.yaml` |
| `specs/**/*.md`   | Requisitos y escenarios verificables               | `operational-spec.md`                               |
| `design.md`       | Decisiones técnicas y arquitectura                 | `plan.md` / `change-design.md`                      |
| `tasks.md`        | Checklist ejecutable                               | `runbook.md`                                        |
| `/opsx:apply`     | Ejecuta tareas y marca checkboxes                  | `sdo-execute`                                       |
| `/opsx:verify`    | Valida completitud, corrección y coherencia        | `sdo-validate`                                      |
| `/opsx:archive`   | Fusiona specs, mueve a archivo y conserva contexto | `sdo-archive`                                       |

OpenSpec verifica tres dimensiones: completitud, corrección y coherencia. La verificación no bloquea necesariamente el archivo, pero reporta problemas como críticos, advertencias o sugerencias. ([GitHub][4])

Para SDO, eso debería endurecerse según riesgo: en `SDO-Lightweight` puede advertir; en `SDO-Critical` debería bloquear si hay fallos críticos.

## 1.5 Archivo y trazabilidad

OpenSpec archiva moviendo el cambio a `openspec/changes/archive/YYYY-MM-DD-name/`, conserva `proposal.md`, `design.md`, `tasks.md` y los delta specs, y actualiza los specs principales como fuente de verdad. ([GitHub][3])

**Lección directa para SDO:**
SDO necesita un archivo que no sea solo “cerrado”, sino un paquete auditable:

```text
archive/
└── 2026-05-23-SDO-000123-restart-rds-maintenance/
    ├── intent.md
    ├── operational-spec.md
    ├── plan.md
    ├── runbook.md
    ├── execution-record.md
    ├── validation-report.md
    ├── evidence/
    └── review.md
```

## 1.6 Lo específico de desarrollo de software

OpenSpec está pensado para código: specs de comportamiento de software, archivos fuente, pruebas, refactors, APIs, diseño técnico y tareas de implementación. Sus ejemplos y comandos hablan de codebase, implementación, tests y cambios en repositorios. ([GitHub][4])

**No trasladar literalmente a SDO:**

* “Implementación” no siempre equivale a escribir código.
* “Spec merge” no siempre equivale a cambiar una especificación funcional.
* “Tests” no siempre equivalen a validación operacional.
* “Codebase” debe convertirse en “operational target”: servidor, servicio, tenant, clúster, base de datos, API, red, pipeline, etc.

---

# 2. Gentle AI / Gentle-IA SDD

## 2.1 Filosofía general

Gentle AI no es solo una implementación SDD; es un configurador de ecosistema para agentes de IA. Añade memoria persistente, workflows SDD, skills, MCP, proveedor/model switcher, persona y asignación de modelos por fase. ([GitHub][5])

**Lectura para SDO:**
Gentle AI no es la mejor fuente para definir el núcleo metodológico de SDO, pero sí es muy valioso para pensar cómo una futura herramienta SDO podría usar agentes, memoria, skills y perfiles.

## 2.2 Workflow SDD

Gentle AI describe SDD como un flujo estructurado para features sustanciales con fases: `explore`, `propose`, `spec`, `design`, `implement`, `verify`. Pero recalca que el usuario no necesita aprenderlas: el agente las activa de forma orgánica cuando la tarea lo requiere. ([GitHub][6])

**Adaptación SDO:**
SDO puede tener dos niveles:

1. **SDO como metodología explícita:** usada por humanos.
2. **SDO asistido por agentes:** donde un orquestador decide cuándo pasar de un flujo ligero a uno completo.

## 2.3 Skills / fases

Gentle AI instala skills SDD como:

| Skill         | Función                                  |
| ------------- | ---------------------------------------- |
| `sdd-init`    | Inicializa contexto del proyecto         |
| `sdd-explore` | Investiga antes de comprometer un cambio |
| `sdd-propose` | Crea propuesta                           |
| `sdd-spec`    | Escribe especificaciones                 |
| `sdd-design`  | Diseña solución técnica                  |
| `sdd-tasks`   | Divide en tareas                         |
| `sdd-apply`   | Implementa según spec y diseño           |
| `sdd-verify`  | Valida contra specs                      |
| `sdd-archive` | Sincroniza delta specs y archiva         |

La documentación de componentes enumera además `sdd-onboard` y `judgment-day`, este último como revisión adversarial paralela. ([GitHub][7])

**Equivalente SDO:**

```text
sdo-init
sdo-observe
sdo-intent
sdo-spec
sdo-plan
sdo-runbook
sdo-execute
sdo-validate
sdo-review
sdo-archive
```

## 2.4 Agentes, subagentes y modelos por fase

Gentle AI soporta distintos modelos de delegación. En modo full, cada fase SDD puede ejecutarse en una ventana de contexto aislada mediante subagentes; el orquestador coordina y los subagentes ejecutan. En modo solo-agent, todas las fases corren en la misma conversación. ([GitHub][8])

Además, para OpenCode permite asignar modelos distintos a fases distintas, por ejemplo un modelo potente para diseño, uno rápido para implementación y uno barato para exploración. ([GitHub][5])

**Lección SDO:**
Esto es muy útil para operaciones:

| Fase SDO          | Tipo de agente posible                              |
| ----------------- | --------------------------------------------------- |
| Observe / Analyze | Agente lector, read-only                            |
| Plan              | Agente de planificación                             |
| Risk Gate         | Agente revisor/adversarial                          |
| Runbook           | Agente procedural                                   |
| Execute           | Agente con permisos limitados                       |
| Validate          | Agente independiente, preferiblemente fresh context |
| Review            | Agente auditor                                      |

## 2.5 Memoria

Gentle AI usa Engram como memoria persistente para guardar decisiones, descubrimientos, bugs y contexto entre sesiones. También permite exportar memorias a `.engram/` para compartirlas por Git. ([GitHub][6])

**Adaptación SDO:**
SDO puede tener un `sdo-memory` o `registry` que recuerde:

* decisiones operacionales;
* patrones aprobados;
* runbooks reutilizables;
* riesgos conocidos por sistema;
* validaciones históricas;
* incidentes previos;
* excepciones aceptadas.

Pero cuidado: en operaciones, la memoria no puede sustituir a evidencia formal. Debe apoyar, no certificar.

## 2.6 Skill registry

Gentle AI tiene un registro de skills que indexa nombres, descripciones, scopes y rutas exactas a `SKILL.md`. El orquestador no resume los skills; pasa rutas exactas para que el subagente lea el contrato original. ([GitHub][9])

**Lección SDO muy importante:**
SDO debería tener un **Operational Skill Registry**:

```text
sdo/registry/skills/
├── linux-service-restart.skill.yaml
├── windows-patching.skill.yaml
├── postgres-maintenance.skill.yaml
├── kubernetes-rollout.skill.yaml
├── aws-rds-change.skill.yaml
└── entra-id-access-change.skill.yaml
```

Cada skill no sería “código”, sino una capacidad operacional controlada.

## 2.7 Backup / rollback

Gentle AI hace snapshots de configuración antes de install/sync/upgrade, con compresión, deduplicación, retención y restore. Pero aclara que su rollback cubre archivos de configuración, no desinstala paquetes del sistema. ([GitHub][10])

**Lección SDO:**
Buen patrón para el **pre-execution snapshot**, pero SDO necesita rollback operacional real: backup, restore point, reversión de config, plan de vuelta atrás, compensación o fail-forward.

## 2.8 Límites y riesgos

Un issue de Gentle AI muestra problemas de interoperabilidad con OpenSpec: `sdd-init` generaba `openspec/project-context.md` en vez del `openspec/config.yaml` canónico, lo que rompía compatibilidad y perdía reglas por artefacto. ([GitHub][11])

**Lectura para SDO:**
No conviene que SDO dependa de convenciones demasiado ligadas a una herramienta concreta. La metodología debe definir formatos mínimos estables, no asumir que un agente específico será la fuente de verdad.

---

# 3. OpenSpec vs Gentle AI SDD

| Criterio                   | OpenSpec                                               | Gentle AI SDD                                    | Mejor inspiración para SDO                 |
| -------------------------- | ------------------------------------------------------ | ------------------------------------------------ | ------------------------------------------ |
| Núcleo metodológico        | Muy claro: artefactos, cambios, specs, archive         | Más orientado a orquestar agentes                | OpenSpec                                   |
| Artefactos                 | `proposal`, `specs`, `design`, `tasks`                 | Skills y fases SDD; menos foco documental propio | OpenSpec                                   |
| Workflow                   | Acciones claras `/opsx:*`                              | Fases orgánicas gestionadas por agente           | Combinado                                  |
| Verificación               | `/opsx:verify` con completitud, corrección, coherencia | `sdd-verify`, fresh review, subagentes           | Combinado                                  |
| Archivo                    | Muy fuerte: merge + archive + audit trail              | `sdd-archive`, memoria, Engram                   | OpenSpec para archivo; Gentle para memoria |
| Agentes                    | Compatible con muchos asistentes                       | Modelo mucho más desarrollado de subagentes      | Gentle AI                                  |
| Skills                     | Menos central                                          | Muy central, con registry                        | Gentle AI                                  |
| Modos ligero/completo      | Core vs expanded                                       | Small request vs substantial feature             | Ambos                                      |
| Riesgo de copiar demasiado | Convertir SDO en “software change framework”           | Convertir SDO en “agent framework”               | Evitar ambos extremos                      |

---

# 4. Tabla: OpenSpec concept → posible equivalente SDO

| OpenSpec concept            | Posible equivalente SDO                                         |
| --------------------------- | --------------------------------------------------------------- |
| `openspec/`                 | `sdo/` artifact store                                           |
| `openspec/specs/`           | `sdo/specs/` o catálogo de especificaciones operacionales vivas |
| `openspec/changes/`         | `sdo/operations/active/`                                        |
| `openspec/changes/archive/` | `sdo/archive/`                                                  |
| Change folder               | Carpeta por operación `SDO-ID`                                  |
| `proposal.md`               | `intent.md`                                                     |
| Delta specs                 | Operational delta: qué cambia en estado/config/proceso          |
| `design.md`                 | `plan.md` o `technical-plan.md`                                 |
| `tasks.md`                  | `runbook.md`                                                    |
| `/opsx:propose`             | `sdo-new` + `sdo-spec`                                          |
| `/opsx:apply`               | `sdo-execute`                                                   |
| `/opsx:verify`              | `sdo-validate`                                                  |
| `/opsx:sync`                | `sdo-evidence` o `sdo-sync-spec`                                |
| `/opsx:archive`             | `sdo-archive`                                                   |
| Core profile                | `SDO-Lightweight`                                               |
| Expanded workflow           | `SDO-Normal` / `SDO-Critical`                                   |
| Custom schemas              | SDO Flow Profiles por clase/riesgo                              |

---

# 5. Tabla: Gentle AI SDD concept → posible equivalente SDO

| Gentle AI concept          | Posible equivalente SDO                     |
| -------------------------- | ------------------------------------------- |
| `sdd-init`                 | `sdo-init`                                  |
| `sdd-explore`              | `sdo-observe` / `sdo-analyze`               |
| `sdd-propose`              | `sdo-intent`                                |
| `sdd-spec`                 | `sdo-spec`                                  |
| `sdd-design`               | `sdo-plan`                                  |
| `sdd-tasks`                | `sdo-runbook`                               |
| `sdd-apply`                | `sdo-execute`                               |
| `sdd-verify`               | `sdo-validate`                              |
| `sdd-archive`              | `sdo-archive`                               |
| Orchestrator               | SDO Orchestrator                            |
| Sub-agents por fase        | Operational agents por capacidad            |
| Skill registry             | Operational skill registry                  |
| Engram memory              | SDO memory / operational knowledge base     |
| Per-phase model assignment | Per-risk/per-phase execution policy         |
| Fresh review               | Independent validation / adversarial review |
| Backup snapshots           | Pre-execution state snapshot                |

---

# 6. Tabla: feature SDD → adopt / adapt / avoid para SDO

| Feature                      | Decisión          | Motivo                                                                                      |
| ---------------------------- | ----------------- | ------------------------------------------------------------------------------------------- |
| Carpeta por cambio           | Adopt             | Encaja con trazabilidad operacional                                                         |
| `proposal/spec/design/tasks` | Adapt             | Cambiar a `intent/spec/plan/runbook`                                                        |
| Delta specs                  | Adapt             | Útil para cambios de configuración, acceso, despliegue, datos                               |
| Apply                        | Adapt             | En SDO debe llamarse execute y soportar manual, CLI, API, IaC, pipeline                     |
| Verify                       | Adopt             | Pero con validaciones operacionales, no solo tests                                          |
| Archive                      | Adopt             | Fundamental para auditoría                                                                  |
| Core vs expanded             | Adopt             | Base natural para SDO-Lightweight vs SDO-Critical                                           |
| Custom schemas               | Adapt             | Convertir en SDO Flow Profiles                                                              |
| Subagentes por fase          | Adapt             | Útil, pero con permisos y HITL estrictos                                                    |
| Skill registry               | Adopt             | Muy útil para capacidades operacionales reutilizables                                       |
| Persistent memory            | Adapt             | Memoria ayuda, pero evidencia formal manda                                                  |
| Model per phase              | Adapt             | Bueno para coste/riesgo, pero no debe afectar responsabilidad humana                        |
| Code review gate             | Avoid como núcleo | SDO no debe depender de Git/code review                                                     |
| TDD estricto                 | Avoid como núcleo | Operaciones requieren validación, no siempre tests automatizados                            |
| Git repo como única fuente   | Avoid             | SDO debe funcionar también con tickets, CMDB, ITSM, carpetas, pipelines o sistemas manuales |

---

# 7. Propuesta inicial de comandos SDO

| Comando          | Propósito                                                          |
| ---------------- | ------------------------------------------------------------------ |
| `sdo-init`       | Inicializa el artifact store y perfiles SDO                        |
| `sdo-new`        | Crea una operación con `SDO-ID`                                    |
| `sdo-intent`     | Captura intención, alcance, no-alcance, solicitante y contexto     |
| `sdo-spec`       | Define estado esperado, restricciones, criterios de aceptación     |
| `sdo-risk`       | Clasifica clase base, overlays, perfil y approvals                 |
| `sdo-plan`       | Define estrategia, impacto, dependencias y rollback                |
| `sdo-runbook`    | Genera pasos ejecutables, prechecks, checkpoints y stop conditions |
| `sdo-preview`    | Muestra plan/diff/comandos sin ejecutar                            |
| `sdo-approve`    | Registra aprobación humana o política                              |
| `sdo-execute`    | Ejecuta o guía la ejecución                                        |
| `sdo-checkpoint` | Registra avance, outputs, evidencias parciales                     |
| `sdo-rollback`   | Ejecuta o documenta reversión                                      |
| `sdo-validate`   | Comprueba estado final contra la spec                              |
| `sdo-evidence`   | Empaqueta logs, capturas, outputs, hashes y referencias            |
| `sdo-review`     | Cierra lecciones aprendidas y desviaciones                         |
| `sdo-archive`    | Mueve la operación a archivo inmutable o controlado                |

---

# 8. Propuesta de artifact store SDO

```text
sdo/
├── config.yaml
├── registry/
│   ├── flow-profiles.yaml
│   ├── operation-classes.yaml
│   ├── risk-overlays.yaml
│   ├── skills/
│   ├── validators/
│   └── evidence-types.yaml
├── specs/
│   ├── services/
│   ├── systems/
│   ├── environments/
│   └── controls/
├── operations/
│   ├── active/
│   │   └── SDO-2026-000123-restart-rds/
│   │       ├── operation.yaml
│   │       ├── intent.md
│   │       ├── operational-spec.md
│   │       ├── risk-assessment.md
│   │       ├── plan.md
│   │       ├── runbook.md
│   │       ├── approvals.md
│   │       ├── execution-record.md
│   │       ├── validation-report.md
│   │       ├── review.md
│   │       └── evidence/
│   └── paused/
├── archive/
│   └── 2026/
│       └── 2026-05-23-SDO-2026-000123-restart-rds/
└── templates/
    ├── lightweight/
    ├── normal/
    ├── critical/
    ├── emergency/
    └── pipeline-backed/
```

---

# 9. Recomendaciones concretas para SDO

1. **Usar OpenSpec como base conceptual del artifact lifecycle.**
   Su modelo de cambio autocontenido, specs delta, verificación y archivo es directamente adaptable a operaciones.

2. **Usar Gentle AI como inspiración para orquestación avanzada.**
   Especialmente subagentes, skills, memoria, registry y perfiles por fase.

3. **Separar SDO metodología de SDO tooling.**
   La metodología debe poder vivir en Markdown/YAML sin IA, sin CLI y sin Git obligatorio.

4. **Convertir `apply` en `execute`.**
   En operaciones, ejecutar puede significar correr comandos, aplicar Terraform, hacer cambios manuales, modificar permisos, reiniciar servicios, ejecutar SQL o coordinar con terceros.

5. **Convertir `verify` en `validate`.**
   Validar no es solo probar: incluye health checks, métricas, logs, estado esperado, ausencia de regresión, evidencia y aprobación de cierre.

6. **Hacer que archive produzca evidencia, no solo historial.**
   El cierre SDO debe contener outputs, logs, capturas, timestamps, responsable, desviaciones, rollback no usado/usado y validación final.

7. **Definir perfiles de rigor desde el inicio.**
   OpenSpec tiene core/expanded; SDO debería formalizar `Lightweight`, `Normal`, `Critical`, `Emergency` y `Pipeline-Backed`.

8. **Incluir skill registry, pero con permisos.**
   Una skill operacional debe declarar alcance, riesgo, permisos requeridos, comandos permitidos, modo read-only/read-write y validaciones esperadas.

9. **No asumir que memoria equivale a trazabilidad.**
   Memoria ayuda al agente; evidencia y artefactos son los que auditan.

10. **No copiar el sesgo a software.**
    SDO debe hablar de servicios, hosts, tenants, pipelines, redes, identidades, bases de datos, configuraciones, datos y usuarios; no de features, PRs y codebase como centro único.

---

# 10. Conclusión

**Inspiración primaria:** OpenSpec.
**Qué tomar:** artifact store, change folder, delta specs, separación `why/what/how/steps`, verify y archive.

**Inspiración secundaria:** Gentle AI.
**Qué tomar:** orquestador, subagentes, skills, registry, memoria, perfiles por fase y revisión con contexto fresco.

**No copiar:** TDD estricto, code review como gate universal, Git como fuente obligatoria, “implementation tasks” centradas en archivos de código, ni memoria como sustituto de evidencia.

La versión SDO debería quedar así:

```text
Intent → Operational Spec → Risk/Profile → Plan → Runbook → Execution → Validation → Evidence → Review → Archive
```

Con comandos:

```text
sdo-new
sdo-intent
sdo-spec
sdo-risk
sdo-plan
sdo-runbook
sdo-preview
sdo-approve
sdo-execute
sdo-validate
sdo-evidence
sdo-review
sdo-archive
```

Mi recomendación final: **diseñar SDO como “OpenSpec para operaciones”, enriquecido con “Gentle-style agents/skills/memory”, pero gobernado por controles operacionales, evidencia y rollback propios de TI.**

[1]: https://github.com/Fission-AI/OpenSpec "GitHub - Fission-AI/OpenSpec: Spec-driven development (SDD) for AI coding assistants. · GitHub"
[2]: https://github.com/Fission-AI/OpenSpec/blob/main/docs/workflows.md "OpenSpec/docs/workflows.md at main · Fission-AI/OpenSpec · GitHub"
[3]: https://github.com/Fission-AI/OpenSpec/blob/main/docs/concepts.md "OpenSpec/docs/concepts.md at main · Fission-AI/OpenSpec · GitHub"
[4]: https://github.com/Fission-AI/OpenSpec/blob/main/docs/commands.md "OpenSpec/docs/commands.md at main · Fission-AI/OpenSpec · GitHub"
[5]: https://github.com/Gentleman-Programming/gentle-ai "GitHub - Gentleman-Programming/gentle-ai · GitHub"
[6]: https://github.com/Gentleman-Programming/gentle-ai/blob/main/docs/intended-usage.md "gentle-ai/docs/intended-usage.md at main · Gentleman-Programming/gentle-ai · GitHub"
[7]: https://github.com/Gentleman-Programming/gentle-ai/blob/main/docs/components.md "gentle-ai/docs/components.md at main · Gentleman-Programming/gentle-ai · GitHub"
[8]: https://github.com/Gentleman-Programming/gentle-ai/blob/main/docs/agents.md "gentle-ai/docs/agents.md at main · Gentleman-Programming/gentle-ai · GitHub"
[9]: https://github.com/Gentleman-Programming/gentle-ai/blob/main/docs/skill-registry.md "gentle-ai/docs/skill-registry.md at main · Gentleman-Programming/gentle-ai · GitHub"
[10]: https://github.com/Gentleman-Programming/gentle-ai/blob/main/docs/rollback.md "gentle-ai/docs/rollback.md at main · Gentleman-Programming/gentle-ai · GitHub"
[11]: https://github.com/Gentleman-Programming/gentle-ai/issues/466 "fix(sdd-init): generates openspec/project-context.md instead of canonical openspec/config.yaml · Issue #466 · Gentleman-Programming/gentle-ai · GitHub"
