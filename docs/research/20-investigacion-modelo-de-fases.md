> Research note — non-normative.
> This document is background material and may be superseded by current SDO core/reference documentation.
> Current normative docs start at [../start-here.md](../start-here.md).

# Modelos de fases para SDO (Spec-Driven Operations)

SDO propone aplicar el enfoque **“spec-driven”** a las operaciones de TI: es decir, construir un flujo de trabajo centrado en una *especificación operativa* verificable que oriente los cambios, despliegues o acciones operativas, sin importar la tecnología usada. A diferencia de seguir rígidamente un único framework (ITIL, SRE, GitOps, etc.), SDO actúa como una **capa metodológica común** que agrupa mejores prácticas. Para entender cómo diseñar sus fases, comparamos con referentes clave:

- **Spec-Driven Development (SDD):** En SDD se escribe primero una *especificación* (behavior-driven o PRD) antes del código, de modo que la especificación sea la fuente de la verdad. El código se genera o implementa para cumplir esa spec【44†L102-L110】【44†L153-L162】. SDO traslada esta idea a operaciones: antes de ejecutar un cambio, se define qué se va a lograr (estado deseado, criterios de éxito, etc.) en un formato formal o estructurado.  
- **ITIL/Change Enablement:** En ITIL 4, el *Change Enablement* distingue tres tipos de cambio: **estándar** (pre-aprobados, bajo riesgo, generalmente documentados o automatizados con runbooks), **normal** (requiere evaluación de riesgo y aprobación) y **emergencia** (urgentes, con revisión post-implementación)【59†L135-L143】. La gobernanza se descentraliza mediante un “Change Authority” (autoridad de cambio) que puede ser un pipeline automatizado, un equipo delegado o revisiones por pares【59†L92-L100】【59†L135-L143】. Esto implica fases opcionales según el riesgo y mecanismos de aprobación/puertas de control.  
- **SRE / SLO-driven:** SRE enfatiza los acuerdos de nivel de servicio (SLO) y *error budgets*. Por ejemplo, si se excede el error budget en un periodo, las políticas de SRE indican **pausar** todos los despliegues (excepto parches críticos) hasta recuperar la confiabilidad【47†L88-L96】. En un flujo SDO influirían pasos de validación continua contra SLO, decisiones automáticas según métricas, etc.  
- **Runbook-driven / Operaciones manuales:** Los runbooks son guías paso a paso para tareas operativas concretas【10†L14-L22】【10†L28-L32】. En SDO, un runbook puede ser la ejecución propiamente dicha. Sin embargo, SDO busca que los runbooks también sean parte de la especifi­cación ejecutable o automatizable, evitando errores por depender de memoria humana. AWS Well-Architected destaca que un runbook debe indicar el resultado deseado, herramientas, manejo de errores y escalamiento【10†L20-L28】. SDO integra esto como fases de preparación (documentar/automatizar pasos) y de ejecución.  
- **Gestión de cambios DevOps/GitOps:** En entornos DevOps, el control de cambios se hace vía pipelines y repositorio de infraestructura. Por ejemplo, un flujo GitOps típico es: *Commit a Git → Pull Request (revisión y plan) → Merge → Plan/Apply automático en ambiente*【19†L184-L193】【19†L209-L218】. Cada cambio queda auditado en Git. SDO debe acomodar flujos de este tipo: la “especificación” podría ser el código IaC o los cambios en repo, mientras que las fases de plan/review y apply aparecen en el flujo.  
- **Respuesta a incidentes (Incident Response):** Los frameworks de respuesta estructurada (como el **Incident Command System**) definen roles y pasos claros: detección, diagnóstico, mitigación, recuperación y post-mortem【8†L62-L70】【42†L1574-L1583】. SDO no reemplaza incident response, pero en caso de incidentes activos puede usar un flujo simplificado (ej. saltar plan formal) y enfocarse en restaurar servicio. Por ejemplo, Atlassian recomienda: *diagnóstico inicial, escalamiento según niveles de soporte, comunicación constante, resolución y cierre del incidente*【42†L1574-L1583】.  
- **Operaciones con IA (Human-in-the-loop):** Con la adopción de **AIOps**, se plantea combinar agentes autónomos con supervisión humana. Al principio, los humanos validan las acciones de IA y mantienen la decisión final【37†L502-L510】. Con el tiempo, pasarían a roles de “monitoreo” (human-on-the-loop)【37†L510-L519】. En SDO esto se refleja en fases flexibles: la especificación operativa y los checkpoints pueden ser elaborados o verificados por IA, pero con gate de aprobación humana según confianza y criticidad.

Tomando todos estos marcos, podemos **diseñar los flujos de SDO** de modo que incluyan las fases principales comunes y permitan variación según contexto:

## Fases principales propuestas (borrador)
Un flujo completo de SDO podría incluir las siguientes fases (algunas opcionales según el caso):

- **Intención (Intent):** Identificación de la necesidad operativa. ¿Cuál es el propósito del cambio o tarea? (Ej.: “detectar y parchear vulnerabilidad X”, “desplegar versión Y”, “consultar estado”).  
- **Especificación (Spec):** Se redacta la *Especificación Operativa*, que describe el objetivo en detalle: condiciones iniciales y finales esperadas, criterios de éxito, restricciones (ventana de mantenimiento, dependencias), SLO o métricas relevantes, y cualquier policy. Se aborda el “qué y por qué”. En SDD esto sería un doc de requisitos; en SDO es un manifiesto para operaciones. Debe ser estructurada y verificable (p.ej. una tabla de expectativas o tests de postcondición).  
- **Planificación (Plan):** Diseño de cómo lograr la especificación. Esto incluye: descomponer la tarea en pasos, asignar recursos, estimar tiempo, definir orden de ejecución y contingencias. Se evalúa el riesgo inicial (dependencias, impacto potencial) y se prepara posible rollback. Salida típica: un plan de acción detallado.  
- **Procedimiento/Runbook:** Preparación de la secuencia exacta de comandos o acciones (script, runbook manual o automatizado). Se traduce el plan a instrucciones concretas: p.ej. comandos de bash, playbooks, pasos en una herramienta GUI, etc. Este artefacto puede mantenerse en version control. Para operaciones automatizadas, podría equivaler al pipeline o IaC plan.  
- **Aprobación/Risk Gate (Gate transversal):** Antes de ejecutar, se revisa la planificación en base al riesgo. Aquí entran validaciones humanas o automáticas: análisis de riesgo, revisión de políticas, aprobaciones (CI/CD pipelines, Change Authority). Este no es un paso secuencial sino una *puerta* transveral. P.ej. un cambio normal requerirá la aprobación de un responsable y la verificación de que la espec cumple políticas internas【59†L92-L100】. Un cambio estándar de bajo riesgo podría auto-aprobarse en CI/CD sin intervención【59†L92-L100】.  
- **Ejecución (Execution):** Se llevan a cabo los pasos. Puede ser manual (ejecutar runbook) o automática (pipeline de IaC). Durante la ejecución se loguea todo: comandos, errores, tiempos. Si algo sale mal, se activa inmediatamente el plan de reversión. Es la fase donde ocurre el cambio real en el sistema.  
- **Validación (Validation):** Inmediatamente después (o en paralelo) se verifica que la ejecución cumplió la espec. Esto puede incluir pruebas automatizadas, chequeo de métricas/SLO, tests de humo, o inspección manual de resultados. P.ej. verificar que un servicio esté activo, o que un esquema de BD se aplicó correctamente. Si la validación falla, se decide si continuar o revertir.  
- **Captura de Evidencia (Evidence):** Se recogen artefactos que demuestran la ejecución y su resultado: registros de logs, reportes de monitoreo, imágenes de pantalla, métricas previas/después, etc. En DevOps/GitOps, puede bastar el historial de commits y despliegues en CI/CD como evidencia auditiva【19†L223-L227】. La evidencia asegura trazabilidad para auditorías o post-mortems. Este paso puede ser continuo (métricas acumuladas) o posterior al fin del despliegue.  
- **Reversión (Rollback):** Planeada *antes* de ejecutar (capacidad de revertir), pero efectuada aquí si es necesario. Puede ser una fase explícita donde se ejecutan los pasos de reversión al estado anterior, o simplemente la capacidad de hacerlo (por ej. retener snapshots). Debe quedar claro qué condiciones disparan la reversión (fallo de validación, incidente). En SRE/GitOps, el sistema puede automatizar este paso (p.ej. Terraform al estado previo). SDO debe requerir que todo cambio tenga una estrategia de rollback definida.  
- **Revisión/Postmortem (Review):** Fase de cierre en la que se revisa el proceso completo. Se documentan lecciones aprendidas, causas de fallos, aciertos. En cambios rutinarios mínimos podría omitirse o hacerse de forma ligera. En cambios críticos o tras incidentes, es obligatoria (post-implementación en ITIL o postmortem de SRE). Por ejemplo, Atlassian sugiere un postmortem con «investigar la causa raíz y registrar acciones» tras un outage (o cualquier cambio P0)【47†L115-L124】.  

Cada fase tendría objetivos, insumos y productos claros. Algunas de estas fases pueden fusionarse o saltarse según el contexto (ver más abajo). Por ejemplo, en un diagnóstico de solo lectura la fase “Ejecución” podría ser limitada (solo consultas sin cambios), mientras que en una corrección urgente se podría agilizar “Planificación” y enfocarse en “Ejecución” inmediata con revisión posterior.

## Fases obligatorias vs opcionales y gates

No todas las fases deben ser obligatorias en todos los casos. Sugerimos:

- **Obligatorias en todo flujo SDO completo:** *Intención*, *Especificación*, *Ejecución*, *Validación* y *Evidencia*. Todo cambio o tarea debe tener claro “qué” se quiere lograr (especificación) y cómo se verifica. También debe haber registros o artefactos que respalden la operación.  
- **Usualmente obligatorias para cambios normales o críticos:** *Planificación*, *Procedimiento (runbook)* y *Revisión*. En cambios con riesgo medio-alto, se necesita plan detallado, preparación de runbook/documentación, y revisión posterior.  
- **Generalmente opcionales o simplificables para cambios de bajo riesgo/automáticos:** En cambios estándar/automatizados, se puede minimizar Planificación (ejecutar directamente el código revisado) y ejecutar runbooks automatizados sin revisión humana. La *aprobación* puede ser automática si ya cumple políticas. *Revisión* puede ser ligera (monitoreo a distancia o postmortem rápido).  
- **Siempre transversal (gate) antes de ejecución:** *Clasificación de riesgo y aprobación*. Este es un control obligatorio: antes de ejecutar, se valida que la especificación cumple normativas internas y se decide quién la aprueba. En flujos automatizados esto puede ser un CI/CD pipeline con políticas (ej. pruebas unitarias y despliegue condicional)【19†L209-L218】【59†L92-L100】. En flujos manuales, puede ser un responsable que evalúa riesgos.  
- **Rollback:** Debe existir siempre la capacidad de revertir, pero ejecutar la reversión es condicional (no se "hace" si todo sale bien). Podría considerarse parte de *Planificación* (definir rollback) y *Ejecución* (potencialmente efectuarlo), en lugar de fase separada secuencial. Sin embargo, listarlo como fase independiente puede enfatizar su importancia.  
- **Evidencia:** Idealmente se genera *continuamente* (por ejemplo, logs en tiempo real) y se resume tras la ejecución. No es un “post-paso” obligatorio (más bien inherente a toda ejecución), pero debe consolidarse luego.

Los **gates transversales** (aprobaciones, monitoreo de riesgos, señal de ejecutar/continuar) son tan importantes como los pasos secuenciales. Ejemplos:  
- **Aprobación humana:** Se ubica entre Planificación y Ejecución. Un responsable (cambio, seguridad, oper. risco) revisa el plan y la spec antes de dar luz verde. DevOps/CI puede automatizarlo para cambios mínimos【59†L92-L100】.  
- **Clasificación de riesgo:** Se realiza antes o durante la planificación, y determina si el flujo se considera *estándar, normal o de emergencia*. De ello depende la cantidad de fases (ej. saltar etapas en emergencia).  
- **Gate de validación continua:** Si la implementación activa indicadores (monitoreo SLO), puede haber un gate que monitoree métricas e interrumpa el proceso (por ej. abortar despliegue) si viola condiciones críticas.  

## Rol de la Reversión y la Evidencia

- **Rollback:** Más que una fase aislada, recomendamos incluirlo como “capacidad obligatoria previa” y también disponer de una fase específica o sub-paso opcional de ejecución de reversión. Es decir, siempre debe estar planificado (artefacto: plan de rollback o snapshot del estado), y ejecutarse en caso de fallo o tras un mal resultado. En algunos casos de alto riesgo, se puede probar el rollback inmediatamente (por ejemplo, reverting automático en IaC). En flujos lean, podría omitirse como fase y considerarse parte del procedimiento si algo falla.  
- **Evidencia:** No debería limitarse a “después de la ejecución”, sino capturarse a lo largo de todo el ciclo. Por ejemplo, el control de versiones (Git, tickets), los logs de pipeline y las métricas de monitoreo pueden ir acumulando evidencia sin intervención manual【19†L223-L227】. Luego de terminado el cambio, se recopila un reporte o se actualiza el ticket/incidente con estos datos. Así, la fase de Evidencia es más un **artefacto transversal**: en cada paso se registran resultados, y se consolida una bitácora al final. 

## Revisión/Postmortem

La fase de *Revisión* (o postmortem) es obligatoria para incidentes o cambios significativos (p.ej. P0/P1, o cambios críticos en producción). Para cambios menores rutinarios puede ser opcional o resumida (una revisión informal). En SRE y Atlassian se insiste en documentar “cómo y por qué falló/resolvió” un incidente【47†L115-L124】. En SDO podemos definir que la revisión sea automática si la infraestructura lo soporta (p.ej. generar reporte en Jira u Opsgenie). El objetivo es aprender lecciones y mejorar el runbook/plan para la próxima.

## Adaptación por tipo de cambio u operación

El flujo SDO debe ramificarse según el contexto. Proponemos tres variaciones principales:

- **Flujo **_estándar/bajo riesgo_:** Para cambios repetitivos o scripts automatizados de baja criticidad. Podría reducir fases: *Intención → Spec ligera → Ejecución (automática) → Validación simple → (Evidencia)*, con aprobación implícita por políticas. Ejemplo: un cron job que limpia logs, que se despliega automáticamente tras un commit en Git. Aquí no hace falta revisión manual previa ni post.  
- **Flujo **_normal/mediano riesgo_:** Incluye todo lo básico: *Intención → Spec → Plan → Procedimiento/Runbook → (Aprobación) → Ejecución → Validación → Evidencia → Revisión*. Apropiado para cambios funcionales planeados (p.ej. despliegue de nueva versión con downtime corto). Se aplican controles (gate) formales antes de ejecutarlo.  
- **Flujo **_crítico/emergencia_:** Debe ser mucho más ágil. Por ejemplo, puede omitirse la planificación formal (o integrarla al vuelo) y acelerarse la aprobación (cambio authority rápido). Podría ser: *Detectar emergencia → Intención/Espec rápida → Ejecutar procedimiento urgente → Validación inmediata → (Reversión si falla) → Revisión post facto*. P.ej. parchar un bug que causa caída de servicio. Aquí la fase de aprobación puede posponerse como *post-implementación*, al estilo “review after deploy” de emergencia (como propone ITIL Emergency).  
- **Diagnósticos de solo lectura:** Si solo extraemos información, el flujo se limita: quizá no se hace cambio alguno, así que *Ejecución* es solo la consulta y *Rollback* no aplica. Todavía vale documentar la intención y evidencias (resultado de consulta) en un runbook.  
- **Incidentes activos:** Similar a emergencia. Se puede tratar como un caso especial de flujo crítico, con roles de incidente (Incident Commander, comunicación) incorporados. El “plan” puede ser improvisado pero debe registrarse en tiempo real. La revisión aquí es esencial (postmortem).  
- **Operaciones manuales vs automatizadas:** En tareas manuales, la fase de Procedimiento (runbook) es más detallada, con pasos claros y tal vez aprobación humana intermedia. En tareas automatizadas (p.ej. IaC), el runbook es el pipeline, la validación puede ser automática, y la aprobación se da en Pull Request. En SDO hay que permitir ambos enfoques: dotar de formato a runbooks manuales y al mismo tiempo aceptar pipelines como runbooks *ejecutables*.  
- **Automatizaciones recurrentes:** Por ejemplo, un despliegue semanal. Al inicio se seguiría el flujo completo para establecer la spec y plan. Una vez maduro, puede volver al flujo estándar con mínima revisión (solo “desplegar tras test”).

En resumen, SDO debe ser **flexible**: un mismo modelo de fases sirve de base, pero con caminos “lightweight” según la criticidad y madurez de la operación.

## Modelos alternativos propuestos

A continuación proponemos **tres modelos de flujo** SDO, resumidos en fases y comparados en ventajas/desventajas:

### Modelo 1: Secuencial completo (SDO Clásico)

**Fases:** Intención → Especificación → Planificación → Procedimiento/Runbook → Gate de Aprobación → Ejecución → Validación → Evidencia → Reversión (si es necesario) → Revisión.  

**Ventajas:** Cobertura exhaustiva, mínimo riesgo; aclara roles en cada fase; fácil auditoría (todo documentado); aplica a cambios críticos.  
**Desventajas:** Bajo rendimiento en cambios rápidos o repetitivos; parece burocrático si no se adapta; puede volverse riguroso (“spec theater”) si se abusa de la documentación.  

### Modelo 2: Integrado/CI-CD (SDO Pipeline)

**Fases:** (GitOps): *Commit/PR (Intención+Spec en IaC) → Plan automatizado → Policies/Security Checks (gate) → Merge → Apply automático (Ejecución) → Tests/Validación automática → Monitoreo/Evidencia → Rollback automático si falla → (Ligera Revisión)*.  

**Ventajas:** Muy ágil para infra-as-code; integración continua; mínimo overhead manual; enfoque declarativo (especificaciones codificadas en Git).  
**Desventajas:** Requiere madurez técnica (IaC, entornos automatizados); no cubre bien tareas manuales o heterogéneas; pérdida de visibilidad humana (necesita buena monitorización).  

### Modelo 3: Emergencia/Incidente (SDO Ágil)

**Fases:** Detección/Intención rápida → Espec mínima rápida → (Gate exprés) → Ejecución inmediata → Validación (inmediata) → Reversión rápida si es necesario → Revisión post-incidente.  

**Ventajas:** Responde a fallos urgentes con mínima demora; incluye solo pasos esenciales; puede decidir “después” la documentación formal.  
**Desventajas:** Mayor riesgo de error por falta de planificación; si no se documenta luego, difícil aprendizaje; necesidad de disciplina para “catch up” postmortem.  

**Comparación:** El **Modelo 1** es robusto pero pesado, ideal para cambios planeados de alto impacto. El **Modelo 2** es ideal para entornos DevOps/GitOps donde los pasos pueden codificarse y validarse automáticamente【19†L184-L193】【19†L209-L218】. El **Modelo 3** sacrifica documentación por velocidad, útil en incidentes reales (con el riesgo inherente). 

**Recomendación:** Un modelo **principal híbrido** podría combinar 1 y 2: usar fases completas cuando hay implicación manual o riesgo significativo, pero adoptar el estilo CI/CD (modelo 2) siempre que la infraestructura lo permita. Para emergencias usar el modelo 3 pero integrando en retrospectiva con el flujo 1. En la práctica, sugerimos denominar las fases de manera clara (abajo) y permitir “ramas” según tipo de cambio.

## Modelo recomendado y detalle de fases

A continuación describimos el **modelo recomendado** (híbrido), con las fases definidas, su objetivo, entradas/salidas, responsables y artefactos. Definiremos dos versiones: **full** (completa) y **lightweight** (ágil).

- **Intención (Intento)**
  - **Objetivo:** Capturar la necesidad de cambio u operación. Responder *¿por qué* hacemos esto* y *qué* problema resuelve.  
  - **Entradas:** Reporte de incidente o solicitud de cambio (ticket, alerta de monitoreo, solicitud de usuario).  
  - **Salidas:** Descripción clara del objetivo, que alimenta la especificación.  
  - **Responsable:** Quien detecta la necesidad (analista de sistemas, DevOps, equipo de soporte).  
  - **Gate de salida:** Validar que la intención sea viable (no débil) y esté alineada a objetivos de negocio. (En flujo lightweight, podría registrarse brevemente).  
  - **Artefactos:** Ticket/issue con la descripción de la necesidad, contexto inicial.

- **Especificación Operativa (Spec)**
  - **Objetivo:** Definir *qué* cambiar y *qué condiciones* debe cumplir el sistema tras el cambio. Documentar criterios de éxito.  
  - **Entradas:** Intención (objetivo, antecedentes), requisitos del servicio, SLA/SLO, políticas (seguridad, cumplimiento).  
  - **Salidas:** Documento/archivo con la spec operacional: puede ser texto estructurado, YAML/JSON de config deseada, una tabla de condiciones, etc. Debe ser **testable** (p.ej. describir un test de validación).  
  - **Responsable:** Ingeniero de operaciones o equipo devops. En entornos con IA, un agente AI puede ayudar a redactar la spec, pero el humano debe revisarla.  
  - **Gate de salida:** Revisión de la spec: verificar coherencia, completitud y que no contradiga políticas (ej. cambiar servicio crítico fuera de windows de mantenimiento).  
  - **Artefactos:** Archivo de especificación (p.ej. `spec_issue123.md`), ingresado en control de versiones o sistema de tickets.

- **Planificación**
  - **Objetivo:** Descomponer la spec en tareas concretas, estimar recursos, preparar cronograma y plan de contingencia.  
  - **Entradas:** Especificación operativa, estado actual del sistema, recursos disponibles.  
  - **Salidas:** Plan detallado de acción: lista de tareas, responsable de cada paso, cronograma y plan de rollback preliminar.  
  - **Responsable:** Equipo de operaciones / project manager de cambio.  
  - **Gate de salida:** Aprobación del plan de proyecto: verificar que el plan cubre la spec y riesgos identificados.  
  - **Artefactos:** Documento/diagrama de plan (puede ser checklist o Gantt simple), plan de rollback inicial, actualización del ticket/cambio con el plan.

- **Procedimiento / Runbook**
  - **Objetivo:** Preparar la secuencia exacta de comandos/acciones a ejecutar. Puede ser manual (checklist) o automatizado (script, playbook, pipeline).  
  - **Entradas:** Planificación (tareas identificadas) y herramientas disponibles.  
  - **Salidas:** Runbook operativo: por ejemplo, scripts, playbooks de Ansible, archivos de Terraform, guión de comandos, o incluso pantallazos con instrucciones.  
  - **Responsable:** Ingeniero de operaciones o automatización.  
  - **Gate de salida:** Revisión técnica de runbook: asegurarse de que los pasos ejecutan el plan, manejo de errores (p.ej. chequear códigos de salida), permisos necesarios y notificaciones estén incluidos.  
  - **Artefactos:** Archivos de runbook (p.ej. en repositorio), documentación del runbook (wiki o ticket), scripts versionados.

- **Gate de Aprobación / Clasificación de Riesgo**
  - **Objetivo:** Evaluar el riesgo y decidir continuar. Se revisan métricas de riesgo (cambios análogos, SLO actuales, etc.), se decide quién aprueba.  
  - **Entradas:** Planificación, runbook, resultados de análisis de riesgos (p.ej. revisión automática de políticas con Sentinel【19†L193-L202】, análisis de impacto en CMDB).  
  - **Salidas:** Autorización (sí/no) para ejecutar, nota de auditoría.  
  - **Responsable:** Change Authority (equipo delegado), o bien pipeline de CI/CD si es automático. En cambios bajos, puede auto-aprobarse【59†L92-L100】.  
  - **Gate de salida:** Documentar la aprobación (o rechazo) en el sistema de gestión. Si no aprobado, cerrar o ajustar el plan.  
  - **Artefactos:** Registro de aprobación (comentario en PR, firma en ticket, etc.), nivel de riesgo clasificado.

- **Ejecución**
  - **Objetivo:** Ejecutar los pasos definidos para aplicar el cambio o tarea operativa.  
  - **Entradas:** Runbook completo, credenciales y accesos, ventanas autorizadas.  
  - **Salidas:** El sistema en el nuevo estado especificado. En caso de falla, el sistema vuelve (idealmente) al estado previo.  
  - **Responsable:** Ingeniero de operaciones o el propio pipeline de despliegue. Si es manual, el ejecutor; si automático, el sistema.  
  - **Gate de salida:** Ninguno (este es el paso de acción). Sin embargo, se documenta lo ocurrido.  
  - **Artefactos:** Logs de ejecución (comandos, salidas, tiempos), versión del artefacto desplegado, ticket/incidente actualizado en tiempo real con comentarios.  

- **Validación**
  - **Objetivo:** Verificar que la ejecución cumplió la especificación operativa y no generó efectos indeseados.  
  - **Entradas:** Resultados de ejecución, monitoreo en vivo, scripts de prueba, métricas antes/después.  
  - **Salidas:** Informe de validación (OK/falla). Decisión: si todo está correcto, continuar; si no, activar rollback.  
  - **Responsable:** Ingeniero de QA/ops o sistema de monitoreo automático. En flujos automáticos, puede ser suite de pruebas post-deploy o reglas de monitoreo (chequeos de salud).  
  - **Gate de salida:** En caso de validación fallida: parada y cambio al paso de rollback.  
  - **Artefactos:** Resultados de tests o monitoreo, ticket/informe de validación con datos, alertas registradas.

- **Evidencia / Documentación**
  - **Objetivo:** Recolectar y almacenar evidencia de todo el proceso, garantizando trazabilidad.  
  - **Entradas:** Logs de ejecución, métricas de sistema, capturas de pantalla, repositorios de código, comentarios en tickets.  
  - **Salidas:** Archivo o entrada de registro que contenga toda la información relevante del cambio. Idealmente automatizado (ej. CI genera reporte en el ticket).  
  - **Responsable:** A menudo automático (herramientas CI/CD, sistemas de tickets) o bien el equipo de ops que actualiza manualmente.  
  - **Gate de salida:** Completar el ticket/incidente con toda la evidencia antes de cerrar la tarea.  
  - **Artefactos:** Entradas de bitácora, archivos adjuntos al ticket, ramas/Git commits con comentarios, reportes de sistema.  

- **Reversión (Rollback)**
  - **Objetivo:** Devolver el sistema al estado previo en caso de fallo.  
  - **Entradas:** Plan de rollback definido, artefactos de backup (snapshots, versiones).  
  - **Salidas:** Sistema restaurado y documentado el rollback.  
  - **Responsable:** Ingeniero de ops o pipeline. Debe ejecutar de inmediato si la validación falla.  
  - **Gate de salida:** Asegurar que el sistema esté funcionalmente restaurado antes de continuar. Documentar el rollback en ticket.  
  - **Artefactos:** Logs de rollback, notas en ticket sobre acciones de reversión, reportes de estado post-rollback.

- **Revisión / Postmortem**
  - **Objetivo:** Analizar el proceso completo para extraer mejoras. Se evalúa qué salió bien, qué falló, cómo mejorar (runbooks, spec, etc.).  
  - **Entradas:** Toda la evidencia y resultados del cambio, reportes de validación y monitoreo, feedback del equipo.  
  - **Salidas:** Informe de post-mortem o “lecciones aprendidas”. Identificación de acciones de mejora (p.ej. actualizar el runbook, cambiar especificación, revisar políticas).  
  - **Responsable:** Equipos de operación, desarrollo e interesados (multi-equipo). Si es incidente, el **Incident Commander** organiza la reunión de revisión【8†L113-L122】【37†L502-L510】.  
  - **Gate de salida:** Solo se cierra la tarea/cambio después de completar la revisión (mínimo verificar que el objetivo se cumplió). Para cambios menores puede resumirse en un comentario final.  
  - **Artefactos:** Documento o sección final en el ticket con el análisis de postmortem, y un plan de acciones (p.ej. tareas separadas si hay follow-ups).

**Versiones “full” vs “lightweight”:**  
- *Full:* Incluye todas las fases anteriores, con gates formales y documentación completa. Se usa en cambios críticos, migraciones, actualizaciones mayores, o cuando lo requiere cumplimiento.  
- *Lightweight:* Se simplifican fases: ej. combinar Intención+Spec en un mismo breve ticket, omitir planificación detallada (actuar directamente sobre el runbook), automatizar aprobación (p.ej. “merge” actúa como aprobación), y hacer revisión solo si hay incidencias. El pipeline automático (Modelo 2) es de hecho una versión lightweight codificada. Por ejemplo, en un despliegue con Terraform, la “fase de ejecución” y “evidencia” ocurren dentro del pipeline, y la revisión se da por los logs de CI.

## Conclusiones

En resumen, SDO propone un **flujo flexible** inspirado en SDD, ITIL, SRE, GitOps e incident management. Las fases **claras** permiten un entendimiento común, pero deben adaptarse según el tipo de cambio: desde procesos ligeros y automáticos (cambios estándar, GitOps) hasta procedimientos completos con aprobación y revisión (cambios críticos/incidentes). Es clave incluir roles y gates (aprobación humana o automática, clasificación de riesgo) sin caer en burocracia excesiva【30†L1470-L1478】【59†L135-L143】. Nuestro modelo recomendado es híbrido: utilizar un pipeline automatizado siempre que sea posible【19†L223-L227】, pero sin sacrificar fases esenciales (spec, validación, rollback) cuando el riesgo lo requiera. En la práctica, nombrar explícitamente las fases (Intención, Especificación, Planificación, Runbook, Ejecución, Validación, Evidencia, Rollback, Revisión) ayuda a estandarizar el proceso y a que los equipos entiendan claramente objetivos y responsabilidades en cada etapa.  

**Fuentes:** Documentación y best practices de ITIL 4, Google SRE, AWS Well-Architected, Atlassian DevOps/Incident Management, y literatura sobre Spec-Driven Development y GitOps【44†L102-L110】【59†L135-L143】【10†L14-L22】【19†L184-L193】【47†L88-L96】【42†L1574-L1583】. Estas referencias ilustran las prácticas ampliamente aceptadas en cada área; las propuestas de flujo SDO aquí son una síntesis basada en dichos enfoques y en criterios de minimización de burocracia.
