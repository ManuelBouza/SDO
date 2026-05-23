> Research note — non-normative.
> This document is background material and may be superseded by current SDO core/reference documentation.
> Current normative docs start at [../start-here.md](../start-here.md).

## Propuesta: arquitectura documental final de SDO

La documentación de SDO debería dividirse en **5 capas**:

1. **Core metodológico**: define qué es SDO y qué reglas no deben romperse.
2. **Referencia**: describe fases, taxonomía, perfiles, artefactos, gates, evidencia, rollback, IA.
3. **Guías prácticas**: explican cómo usar SDO en operaciones reales.
4. **Plantillas y ejemplos**: permiten aplicar SDO sin reinterpretar la teoría.
5. **Integraciones opcionales**: Gentle AI, workflows estilo OpenSpec, PRs, tickets, pipelines, CLI, etc.

La separación clave sería esta:

| Capa         | Propósito                       | Naturaleza              |
| ------------ | ------------------------------- | ----------------------- |
| Core         | Definir la metodología          | Normativa               |
| Reference    | Explicar cada componente formal | Normativa + explicativa |
| Guides       | Enseñar uso práctico            | Práctica                |
| Templates    | Dar formatos reutilizables      | Operativa               |
| Examples     | Mostrar casos completos         | No normativa            |
| Integrations | Adaptar SDO a herramientas      | Opcional / no core      |

---

# 1. Árbol recomendado `docs/`

```text
sdo/
├── README.md
├── CHANGELOG.md
├── VERSIONING.md
├── docs/
│   ├── start-here.md
│   ├── glossary.md
│   │
│   ├── core/
│   │   ├── 00-sdo-core.md
│   │   ├── 01-principles.md
│   │   ├── 02-lifecycle.md
│   │   ├── 03-scope-and-non-goals.md
│   │   └── 04-conformance.md
│   │
│   ├── reference/
│   │   ├── phases.md
│   │   ├── taxonomy.md
│   │   ├── profiles.md
│   │   ├── artifacts.md
│   │   ├── gates-and-controls.md
│   │   ├── execution-model.md
│   │   ├── evidence-and-auditability.md
│   │   ├── rollback-and-reversibility.md
│   │   └── ai-governance.md
│   │
│   ├── guides/
│   │   ├── operator-guide.md
│   │   ├── spec-author-guide.md
│   │   ├── reviewer-approver-guide.md
│   │   ├── platform-engineer-guide.md
│   │   ├── adoption-guide.md
│   │   └── lightweight-sdo-guide.md
│   │
│   ├── templates/
│   │   ├── operational-spec.md
│   │   ├── operational-spec.yaml
│   │   ├── plan.md
│   │   ├── runbook.md
│   │   ├── execution-record.md
│   │   ├── validation-report.md
│   │   ├── evidence-package.yaml
│   │   ├── rollback-plan.md
│   │   ├── review-record.md
│   │   └── approval-record.md
│   │
│   ├── examples/
│   │   ├── README.md
│   │   ├── observe-linux-service/
│   │   ├── configure-windows-service/
│   │   ├── deploy-docker-service/
│   │   ├── database-maintenance/
│   │   ├── emergency-remediation/
│   │   └── pipeline-backed-change/
│   │
│   ├── integrations/
│   │   ├── README.md
│   │   ├── gentle-ai.md
│   │   ├── openspec-style-workflows.md
│   │   ├── git-and-pull-requests.md
│   │   ├── ticketing-systems.md
│   │   ├── cicd-pipelines.md
│   │   └── cli-agents.md
│   │
│   └── adoption/
│       ├── maturity-model.md
│       ├── organization-rollout.md
│       ├── governance-model.md
│       ├── roles-and-responsibilities.md
│       └── metrics.md
└── schemas/
    ├── operational-spec.schema.yaml
    ├── evidence-package.schema.yaml
    ├── execution-record.schema.yaml
    └── validation-report.schema.yaml
```

---

# 2. Documentos finales recomendados

## Documentos Core

| Documento                             | Propósito                        | Audiencia                   | Contenido mínimo                                                               | Fuente base                          | Prioridad      |
| ------------------------------------- | -------------------------------- | --------------------------- | ------------------------------------------------------------------------------ | ------------------------------------ | -------------- |
| `README.md`                           | Presentar SDO rápidamente        | Todos                       | Qué es, para qué sirve, qué no es, enlace a Start Here                         | Estado del arte + decisiones finales | MVP            |
| `docs/start-here.md`                  | Ruta de lectura inicial          | Nuevos usuarios             | Qué leer en 15 min, flujo mínimo, ejemplo rápido                               | Síntesis general                     | MVP            |
| `docs/glossary.md`                    | Unificar vocabulario             | Todos                       | SDO ID, Operational Spec, Evidence, Gate, Profile, Runbook, Validation         | Todas                                | MVP            |
| `docs/core/00-sdo-core.md`            | Definir la metodología           | Todos                       | Definición, objetivo, unidad de trabajo, ciclo base, principios obligatorios   | Estado del arte + mapeo SDO↔SDD      | MVP            |
| `docs/core/01-principles.md`          | Declarar principios rectores     | Arquitectos, adopción       | Spec-first, evidence-first, risk-adaptive, reversible by design, tool-agnostic | Investigaciones generales            | MVP            |
| `docs/core/02-lifecycle.md`           | Definir el ciclo oficial         | Todos                       | Intent → Spec → Plan → Runbook → Execution → Validation → Review               | Modelo de fases                      | MVP            |
| `docs/core/03-scope-and-non-goals.md` | Evitar confusión metodológica    | Arquitectos, organizaciones | SDO no reemplaza ITIL, SRE, GitOps, IaC, runbooks ni incident response         | Estado del arte                      | MVP            |
| `docs/core/04-conformance.md`         | Definir qué significa “usar SDO” | Organizaciones, tooling     | Requisitos mínimos para SDO-Lightweight, Normal, Critical                      | Perfiles + gates                     | Futuro cercano |

---

## Documentos de referencia

| Documento                                 | Propósito                    | Audiencia                   | Contenido mínimo                                                                                      | Fuente base               | Prioridad |
| ----------------------------------------- | ---------------------------- | --------------------------- | ----------------------------------------------------------------------------------------------------- | ------------------------- | --------- |
| `reference/phases.md`                     | Describir cada fase          | Operadores, arquitectos     | Entrada, salida, responsables, gates por fase                                                         | Modelo de fases           | MVP       |
| `reference/taxonomy.md`                   | Clasificar operaciones       | Operadores, reviewers       | Base Class, overlays, criterios de clasificación                                                      | Taxonomía                 | MVP       |
| `reference/profiles.md`                   | Adaptar rigor por riesgo     | Operadores, adopción        | SDO-Lightweight, Normal, Critical, Emergency, Pipeline-Backed                                         | Taxonomía + gates         | MVP       |
| `reference/artifacts.md`                  | Definir artefactos oficiales | Todos                       | Operational Spec, Plan, Runbook, Execution Record, Validation Report, Evidence Package, Review Record | Artefactos                | MVP       |
| `reference/gates-and-controls.md`         | Definir controles            | Reviewers, organizaciones   | Spec Gate, Approval Gate, Pre-Execution Gate, Validation Gate, Closure Gate                           | Gates y controles         | MVP       |
| `reference/execution-model.md`            | Explicar ejecución agnóstica | Operadores, tooling         | Manual, CLI, API, script, pipeline, agente, ticket                                                    | Ejecución agnóstica       | MVP       |
| `reference/evidence-and-auditability.md`  | Formalizar evidencia         | Auditores, operadores       | Qué capturar, cuándo, cómo enlazar a SDO ID                                                           | Evidencia y auditabilidad | MVP       |
| `reference/rollback-and-reversibility.md` | Definir rollback/recovery    | Operadores, reviewers       | Rollback plan, reversibilidad, compensación, restore, fail-forward                                    | Rollback                  | MVP       |
| `reference/ai-governance.md`              | Gobernar uso de IA           | Arquitectos, organizaciones | IA como asistente, límites, HITL, trazabilidad, revisión humana                                       | Gobernanza IA             | MVP       |

---

## Guías prácticas

| Documento                           | Propósito                         | Audiencia                  | Contenido mínimo                                           | Fuente base               | Prioridad |
| ----------------------------------- | --------------------------------- | -------------------------- | ---------------------------------------------------------- | ------------------------- | --------- |
| `guides/operator-guide.md`          | Usar SDO en una operación real    | Operadores                 | Cómo crear spec, elegir perfil, ejecutar, validar y cerrar | Todas                     | MVP       |
| `guides/spec-author-guide.md`       | Escribir buenas Operational Specs | Técnicos, responsables     | Intención, alcance, riesgo, criterios de éxito, rollback   | Artefactos + gates        | MVP       |
| `guides/reviewer-approver-guide.md` | Revisar y aprobar specs           | Responsables técnicos      | Qué revisar, señales de bloqueo, criterios de aprobación   | Gates                     | MVP       |
| `guides/platform-engineer-guide.md` | Implementar tooling SDO           | Platform engineers         | Repos, schemas, PRs, pipelines, validaciones automáticas   | Ejecución + integraciones | MVP       |
| `guides/adoption-guide.md`          | Adoptar SDO en una organización   | Managers, líderes técnicos | Piloto, roles, niveles de madurez, métricas                | Adopción + gobernanza     | MVP       |
| `guides/lightweight-sdo-guide.md`   | Usar SDO sin burocracia           | Equipos pequeños           | Flujo mínimo para operaciones de bajo riesgo               | Perfiles                  | MVP       |

---

## Plantillas iniciales

| Plantilla                         | Propósito                                 | Prioridad      |
| --------------------------------- | ----------------------------------------- | -------------- |
| `templates/operational-spec.md`   | Documento principal de intención aprobada | MVP            |
| `templates/operational-spec.yaml` | Versión estructurada para tooling         | MVP            |
| `templates/plan.md`               | Plan de implementación o cambio           | MVP            |
| `templates/runbook.md`            | Pasos ejecutables                         | MVP            |
| `templates/execution-record.md`   | Registro de lo ocurrido                   | MVP            |
| `templates/validation-report.md`  | Resultado de validación                   | MVP            |
| `templates/evidence-package.yaml` | Índice de evidencias                      | MVP            |
| `templates/rollback-plan.md`      | Plan de reversión o recuperación          | MVP            |
| `templates/review-record.md`      | Cierre y aprendizaje                      | MVP            |
| `templates/approval-record.md`    | Registro explícito de aprobación          | Futuro cercano |

---

## Integraciones opcionales

| Documento                                  | Propósito                                             | Prioridad      |
| ------------------------------------------ | ----------------------------------------------------- | -------------- |
| `integrations/README.md`                   | Explicar que las integraciones no son SDO Core        | MVP            |
| `integrations/gentle-ai.md`                | Usar Gentle AI como ejecutor/asistente opcional       | MVP            |
| `integrations/openspec-style-workflows.md` | Adaptar ideas de lifecycle tipo propose/apply/archive | MVP            |
| `integrations/git-and-pull-requests.md`    | Representar SDO con ramas, PRs y revisiones           | MVP            |
| `integrations/ticketing-systems.md`        | Mapear SDO a Jira, GitHub Issues, ServiceNow, etc.    | Futuro cercano |
| `integrations/cicd-pipelines.md`           | Automatizar gates y validaciones                      | Futuro cercano |
| `integrations/cli-agents.md`               | Integrar agentes CLI sin hacerlos obligatorios        | Futuro         |

---

# 3. Documentos normativos vs explicativos

## Normativos

Estos documentos definen SDO. Cambiarlos implica cambiar la metodología.

```text
docs/core/00-sdo-core.md
docs/core/01-principles.md
docs/core/02-lifecycle.md
docs/core/03-scope-and-non-goals.md
docs/core/04-conformance.md
docs/reference/taxonomy.md
docs/reference/profiles.md
docs/reference/artifacts.md
docs/reference/gates-and-controls.md
docs/reference/execution-model.md
docs/reference/evidence-and-auditability.md
docs/reference/rollback-and-reversibility.md
docs/reference/ai-governance.md
```

## Explicativos

Ayudan a entender, pero no redefinen la metodología.

```text
README.md
docs/start-here.md
docs/glossary.md
docs/examples/*
docs/integrations/*
```

## Guías prácticas

Explican cómo aplicar SDO.

```text
docs/guides/operator-guide.md
docs/guides/spec-author-guide.md
docs/guides/reviewer-approver-guide.md
docs/guides/platform-engineer-guide.md
docs/guides/adoption-guide.md
docs/guides/lightweight-sdo-guide.md
```

## Plantillas

Son el puente entre teoría y operación.

```text
docs/templates/*
schemas/*
```

---

# 4. Orden de lectura recomendado

## Para entender SDO en 15 minutos

1. `README.md`
2. `docs/start-here.md`
3. `docs/core/00-sdo-core.md`
4. `docs/core/02-lifecycle.md`
5. `docs/reference/profiles.md`
6. `docs/templates/operational-spec.md`
7. Un ejemplo de `docs/examples/`

## Para un operador

1. `docs/start-here.md`
2. `docs/guides/operator-guide.md`
3. `docs/reference/profiles.md`
4. `docs/templates/operational-spec.md`
5. `docs/templates/runbook.md`
6. `docs/templates/execution-record.md`
7. `docs/templates/validation-report.md`
8. `docs/templates/evidence-package.yaml`
9. `docs/templates/rollback-plan.md`

## Para un arquitecto o platform engineer

1. `docs/core/00-sdo-core.md`
2. `docs/reference/artifacts.md`
3. `docs/reference/gates-and-controls.md`
4. `docs/reference/execution-model.md`
5. `schemas/*`
6. `docs/guides/platform-engineer-guide.md`
7. `docs/integrations/git-and-pull-requests.md`
8. `docs/integrations/cicd-pipelines.md`
9. `docs/integrations/gentle-ai.md`

## Para una organización

1. `README.md`
2. `docs/core/03-scope-and-non-goals.md`
3. `docs/guides/adoption-guide.md`
4. `docs/adoption/organization-rollout.md`
5. `docs/adoption/roles-and-responsibilities.md`
6. `docs/adoption/governance-model.md`
7. `docs/adoption/maturity-model.md`
8. `docs/adoption/metrics.md`

---

# 5. README principal propuesto

El `README.md` debería ser breve y orientado a decisión.

Contenido recomendado:

```text
# SDO: Spec-Driven Operations

## Qué es SDO

SDO es una metodología agnóstica para diseñar, aprobar, ejecutar, validar y auditar operaciones TI mediante especificaciones operacionales.

## Problema que resuelve

Reduce operaciones improvisadas, mejora trazabilidad, facilita rollback, captura evidencia y adapta el nivel de control según el riesgo.

## Qué no es

SDO no reemplaza ITIL, SRE, GitOps, IaC, runbooks, pipelines ni herramientas de tickets. Los orquesta.

## Ciclo base

Intent → Spec → Plan → Runbook → Execution → Validation → Review

## Principios

- Spec-first
- Evidence-first
- Risk-adaptive
- Tool-agnostic
- Human accountable
- Reversible by design

## Empieza aquí

Leer `docs/start-here.md`.

## Artefactos principales

- Operational Spec
- Plan
- Runbook
- Execution Record
- Validation Report
- Evidence Package
- Review Record

## Integraciones opcionales

- Gentle AI
- OpenSpec-style workflows
- Git/PRs
- Tickets
- Pipelines
```

---

# 6. Guía “Start here” propuesta

`docs/start-here.md` debería responder tres preguntas:

## 1. ¿Qué necesito saber?

SDO trabaja con una unidad trazable: una operación identificada por un `SDO ID`.

Cada operación debe tener, como mínimo:

```text
Operational Spec
Runbook o pasos de ejecución
Execution Record
Validation Result
Evidence
```

## 2. ¿Cuál es el flujo mínimo?

```text
1. Declarar intención.
2. Escribir Operational Spec.
3. Clasificar la operación.
4. Elegir perfil SDO.
5. Preparar plan/runbook.
6. Revisar gates aplicables.
7. Ejecutar.
8. Capturar evidencia.
9. Validar resultado.
10. Cerrar con review.
```

## 3. ¿Qué perfil uso?

```text
Bajo riesgo        → SDO-Lightweight
Riesgo normal      → SDO-Normal
Alto impacto       → SDO-Critical
Incidente urgente  → SDO-Emergency
Automatizado       → SDO-Pipeline-Backed
```

---

# 7. Plantillas iniciales mínimas

## `operational-spec.md`

Campos mínimos:

```text
SDO ID
Título
Estado
Autor
Fecha
Clase de operación
Perfil SDO
Contexto
Intención
Alcance
Fuera de alcance
Sistemas afectados
Riesgos
Criterios de éxito
Criterios de no éxito
Plan resumido
Validación esperada
Rollback esperado
Aprobaciones requeridas
Referencias
```

## `runbook.md`

```text
SDO ID
Precondiciones
Ventana de ejecución
Responsable
Herramientas necesarias
Pasos
Comandos
Resultados esperados
Puntos de control
Condiciones de stop
Condiciones de rollback
Validaciones
```

## `execution-record.md`

```text
SDO ID
Ejecutor
Inicio
Fin
Pasos ejecutados
Desviaciones
Errores
Decisiones tomadas
Evidencias capturadas
Estado final
```

## `validation-report.md`

```text
SDO ID
Criterios validados
Método de validación
Resultado esperado
Resultado observado
Evidencia asociada
Conclusión
```

## `evidence-package.yaml`

```yaml
sdo_id:
operation_title:
evidence:
  - id:
    type:
    description:
    source:
    path_or_url:
    captured_at:
    captured_by:
    related_phase:
    related_gate:
integrity:
  checksum:
  storage_location:
```

## `rollback-plan.md`

```text
SDO ID
Tipo de reversibilidad
Condiciones de activación
Pasos de rollback
Responsable
Tiempo estimado
Riesgos del rollback
Validación post-rollback
Plan alternativo
```

---

# 8. Tabla de trazabilidad: investigación → documentos finales

| Investigación existente   | Documento final principal                  | Documentos derivados                                      |
| ------------------------- | ------------------------------------------ | --------------------------------------------------------- |
| Estado del arte           | `core/03-scope-and-non-goals.md`           | `README.md`, `adoption-guide.md`                          |
| Modelo de fases           | `core/02-lifecycle.md`                     | `reference/phases.md`, `operator-guide.md`                |
| Taxonomía de operaciones  | `reference/taxonomy.md`                    | `reference/profiles.md`, `lightweight-sdo-guide.md`       |
| Artefactos                | `reference/artifacts.md`                   | `templates/*`, `schemas/*`                                |
| Gates y controles         | `reference/gates-and-controls.md`          | `reviewer-approver-guide.md`                              |
| Ejecución agnóstica       | `reference/execution-model.md`             | `platform-engineer-guide.md`, `integrations/*`            |
| Evidencia y auditabilidad | `reference/evidence-and-auditability.md`   | `evidence-package.yaml`, `validation-report.md`           |
| Rollback y reversibilidad | `reference/rollback-and-reversibility.md`  | `rollback-plan.md`, ejemplos críticos                     |
| Gobernanza IA             | `reference/ai-governance.md`               | `integrations/gentle-ai.md`, `integrations/cli-agents.md` |
| Mapeo SDO ↔ SDD           | `core/00-sdo-core.md`                      | `integrations/openspec-style-workflows.md`                |
| OpenSpec                  | `integrations/openspec-style-workflows.md` | lifecycle de artifacts                                    |
| Gentle AI                 | `integrations/gentle-ai.md`                | guía de agentes opcionales                                |

---

# 9. MVP documental

Para una primera versión usable, yo limitaría el MVP a esto:

```text
README.md
CHANGELOG.md
VERSIONING.md

docs/start-here.md
docs/glossary.md

docs/core/00-sdo-core.md
docs/core/01-principles.md
docs/core/02-lifecycle.md
docs/core/03-scope-and-non-goals.md

docs/reference/taxonomy.md
docs/reference/profiles.md
docs/reference/artifacts.md
docs/reference/gates-and-controls.md
docs/reference/execution-model.md
docs/reference/evidence-and-auditability.md
docs/reference/rollback-and-reversibility.md
docs/reference/ai-governance.md

docs/guides/operator-guide.md
docs/guides/platform-engineer-guide.md
docs/guides/adoption-guide.md

docs/templates/operational-spec.md
docs/templates/runbook.md
docs/templates/execution-record.md
docs/templates/validation-report.md
docs/templates/evidence-package.yaml
docs/templates/rollback-plan.md

docs/examples/README.md
docs/examples/observe-linux-service/
docs/examples/emergency-remediation/

docs/integrations/README.md
docs/integrations/gentle-ai.md
docs/integrations/openspec-style-workflows.md
```

Todo lo demás puede quedar para fases posteriores.

---

# 10. Orden de escritura recomendado

1. `glossary.md`
2. `core/00-sdo-core.md`
3. `core/01-principles.md`
4. `core/02-lifecycle.md`
5. `reference/taxonomy.md`
6. `reference/profiles.md`
7. `reference/artifacts.md`
8. `templates/operational-spec.md`
9. `templates/runbook.md`
10. `templates/execution-record.md`
11. `templates/validation-report.md`
12. `templates/evidence-package.yaml`
13. `templates/rollback-plan.md`
14. `reference/gates-and-controls.md`
15. `reference/execution-model.md`
16. `reference/evidence-and-auditability.md`
17. `reference/rollback-and-reversibility.md`
18. `reference/ai-governance.md`
19. `start-here.md`
20. `operator-guide.md`
21. `platform-engineer-guide.md`
22. `adoption-guide.md`
23. `integrations/gentle-ai.md`
24. `integrations/openspec-style-workflows.md`
25. Ejemplos completos

---

# 11. Convenciones de nombres

Recomiendo:

```text
kebab-case.md
```

Ejemplos:

```text
operational-spec.md
execution-record.md
evidence-package.yaml
rollback-and-reversibility.md
openspec-style-workflows.md
```

Para ejemplos:

```text
examples/<base-class>-<context>/
```

Ejemplos:

```text
examples/observe-linux-service/
examples/configure-windows-service/
examples/deploy-docker-service/
examples/remediate-database-incident/
```

Para identificadores SDO:

```text
SDO-YYYYMMDD-NNN
```

Ejemplo:

```text
SDO-20260523-001
```

---

# 12. Cómo evitar duplicación

Regla principal:

> Cada concepto debe tener un único documento dueño.

Ejemplo:

| Concepto           | Documento dueño                           |
| ------------------ | ----------------------------------------- |
| Ciclo SDO          | `core/02-lifecycle.md`                    |
| Fases detalladas   | `reference/phases.md`                     |
| Tipos de operación | `reference/taxonomy.md`                   |
| Perfiles           | `reference/profiles.md`                   |
| Artefactos         | `reference/artifacts.md`                  |
| Gates              | `reference/gates-and-controls.md`         |
| Evidencia          | `reference/evidence-and-auditability.md`  |
| Rollback           | `reference/rollback-and-reversibility.md` |
| IA                 | `reference/ai-governance.md`              |
| Uso con Gentle AI  | `integrations/gentle-ai.md`               |

Los demás documentos deben enlazar, no redefinir.

---

# 13. Cómo hacer la documentación accionable

Cada documento importante debería incluir:

```text
Propósito
Cuándo usarlo
Entradas
Salidas
Responsables
Checklist mínima
Errores comunes
Ejemplo breve
Enlaces a plantillas
```

Y cada referencia normativa debería cerrar con:

```text
Minimum requirements
Recommended practices
Anti-patterns
```

Ejemplo:

En `reference/gates-and-controls.md`, no basta con explicar gates. Debe tener tablas como:

```text
Gate
Cuándo aplica
Quién lo revisa
Qué evidencia requiere
Criterio de aprobación
Criterio de bloqueo
```

---

# 14. Versionado de la metodología

Usaría SemVer:

```text
MAJOR.MINOR.PATCH
```

Ejemplo:

```text
SDO 0.1.0
```

Criterios:

| Cambio                                         | Versión |
| ---------------------------------------------- | ------- |
| Cambia el ciclo base o artefactos obligatorios | MAJOR   |
| Añade perfiles, gates o plantillas compatibles | MINOR   |
| Corrige redacción, ejemplos o ambigüedades     | PATCH   |

Además:

```text
CHANGELOG.md
VERSIONING.md
docs/core/04-conformance.md
```

Y frontmatter opcional en cada documento:

```yaml
---
title: Operational Spec
sdo_version: 0.1.0
doc_type: template
status: draft
owner: sdo-core
---
```

---

# 15. Gentle AI y OpenSpec-style workflows

## Gentle AI

Debe documentarse como integración opcional.

`integrations/gentle-ai.md` debería dejar claro:

```text
Gentle AI puede asistir en:
- generación de specs;
- análisis de riesgo;
- preparación de runbooks;
- ejecución supervisada;
- validación;
- empaquetado de evidencia.

Gentle AI no sustituye:
- aprobación humana;
- responsabilidad operacional;
- criterios de validación;
- evidencia real;
- decisiones de rollback.
```

Regla importante:

> SDO Core no depende de Gentle AI. Gentle AI implementa o asiste SDO, pero no define SDO.

## OpenSpec-style workflows

`integrations/openspec-style-workflows.md` debería adaptar ideas como:

```text
propose → review → approve → apply → verify → archive
```

A SDO:

```text
Intent → Spec → Approval → Runbook → Execution → Validation → Evidence → Review/Archive
```

La inspiración útil de OpenSpec sería:

```text
- lifecycle claro de artefactos;
- separación entre propuesta y aplicación;
- archivo histórico;
- trazabilidad entre intención, cambio y resultado;
- workflows reproducibles.
```

Pero evitando copiar su foco exclusivo en desarrollo de software.

---

# Conclusión

La documentación final de SDO debería empezar pequeña pero estricta:

```text
Core claro
Referencia estable
Plantillas accionables
Guías prácticas
Ejemplos reales
Integraciones opcionales separadas
```

Mi recomendación para el MVP es priorizar:

```text
1. Core
2. Lifecycle
3. Profiles
4. Artifacts
5. Gates
6. Templates
7. Operator Guide
8. Evidence
9. Rollback
10. Integraciones Gentle AI / OpenSpec-style
```

Eso permitiría que SDO sea entendible, aplicable y extensible sin quedar atado a una herramienta concreta.
