> Research note — non-normative.
> This document is background material and may be superseded by current SDO core/reference documentation.
> Current normative docs start at [../start-here.md](../start-here.md).


1. Evolución del SDD moderno hacia flujos con agentes de IA
Las metodologías de desarrollo impulsadas por especificaciones (SDD) han ido incorporando agentes de IA para automatizar gran parte del ciclo de vida. En el modelo actual se observa un patrón “especifica primero” (spec-first), donde las especificaciones se tratan como contratos ejecutables que guían a los agentes. Estas especificaciones definen claramente objetivos, restricciones y criterios de verificación
. Con agentes coordinadores y subagentes especializados (como “Coordinator”, “Implementor” y “Verifier”
), el flujo tipicamente sigue cuatro fases: Specificar (qué hacer), Planificar (detallar cómo), Descomponer en Tareas (tasks) y Implementar/Ejecutar. En cada fase hay puntos de control o checkpoints de validación (usualmente automatizados) antes de pasar a la siguiente
.

Por ejemplo, un agente lee la especificación maestra (spec), genera un plan técnico, desglosa ese plan en tareas aisladas y verificables
. Luego, se “aplica” el plan: cada tarea la ejecuta un agente (o humano) y se valida mediante pruebas integradas (compilación correcta, tests unitarios, análisis de seguridad, benchmarks, etc.)
. Estos ciclos de apply/verify se repiten hasta completar las tareas, manteniendo trazabilidad y archivando los resultados al final. Un prototipo de flujo (usando GitHub Copilot CLI) es: explorar idea → proponer cambio → aprobar → aplicar tareas → verificar con herramientas → archivar
. En esa práctica, cada paso es visible y versionado (“each step is traceable. Each artifact is versioned.”)
, cerrando el ciclo de principio a fin.

En paralelo, se incorporan apruebas humanas en puntos clave. Por ejemplo, el borrador de propuesta se revisa antes de aplicar cambios (ver diagrama de flujo de [15†L123-L131]). Así se mantiene un “humano-en-el-bucle” crítico para revisar ambigüedades o riesgos. Además, las arquitecturas modernas insisten en la trazabilidad completa: cada agente, plan o cambio queda documentado, lo cual crea un rastro de auditoría y evidencia. Herramientas tipo OpenSpec o Spec Kit ofrecen marcos que enlazan las especificaciones con el código generado, evitando desviaciones arquitectónicas
. Como resumen, SDD evoluciona hacia flujos de múltiples agentes coordinados que leen specs vivas, generan planes y tareas, los ejecutan con validaciones automáticas, y conservan toda la traza (specificación, tareas, reportes de verificación) como base de auditoría.

2. Patrones de SDD aplicables a SDO
Muchas prácticas de SDD se pueden trasladar a SDO agentico adaptando artefactos. Por ejemplo, el concepto de “especificación ejecutable” de SDD se convierte en el Operational Spec de SDO: un documento de intención operacional con criterios claros de éxito, que actúa como contrato de ejecución. En SDD se decía que la especificación “se convierte en el contrato contra el cual se implementa y valida todo”
; en SDO el Operational Spec contiene intenciones, limitaciones y criterios de validación de la operación.

Análogamente, un Runbook (libro de operaciones) actúa como el plan ejecutable por un agente: es la descomposición de las acciones a realizar. Así como en SDD cada especificación genera un conjunto de tareas pequeñas y testables
, en SDO el runbook transforma el plan operativo en pasos concretos (scripts, comandos o llamadas API) para el agente.

En vez de un verify-report de código, en SDO tendría sentido generar un Validation Report: un análisis independiente (automatizado o humano) que chequea que la operación cumplió la spec. En SDD se propone usar un agente Verificador que confirma cada componente final contra los criterios especificados
; en operaciones se haría lo mismo revisando logs, salidas y métricas.

De forma similar, se conserva el concepto de Evidence Package como memoria o archivo histórico: un conjunto de artefactos que documenta qué se hizo, cómo y con qué resultados. Como señala la guía de auditoría de agentes AI, es necesario “evidence… structured, immutable, queryable run evidence”
. En SDO ese paquete de evidencia incluiría logs de ejecución, hashes o punteros inmutables, reportes de aprobación y de verificación.

Las puertas de aprobación (gates) humanas/automáticas se trasladan directamente: checkpoints obligatorios donde un humano revisa o un sistema verifica (por ejemplo, políticas de seguridad o pruebas previas) antes de avanzar. Esto concuerda con el énfasis en runbooks modernos: los runbooks integran flujos de aprobación interactiva y trazabilidad total
.

Finalmente, es clave requerir un plan de rollback/recovery previo a la ejecución. Así como en SDD se define “criterios de éxito” y se valida continuamente, en SDO agentico se debe exigir procedimientos de revertir cambios en caso de fallo, definidos en el plan o la spec. En suma, SDD aporta el patrón de spec como contrato, tasks/runbook como plan, verificación independiente como validación y la noción de artefactos versionados y auditables (spec vivas, reportes de verify/validate, historial)
. También enfatiza checkpoints y gates antes/después de la ejecución, muy útil para operaciones críticas.

3. Cambios al ejecutar operaciones TI con agentes IA
Introducir agentes en operaciones requiere reforzar aspectos de seguridad y control. Primero, los permisos y límites de herramientas: cada agente debe operar con privilegios mínimos estrictos (principio de menor privilegio) y dentro de entornos sandbox o API específicas. Por ejemplo, usar controles de acceso basados en atributos (ABAC) en lugar de roles estáticos se considera ideal para agentes dinámicos
. Esto delimita el alcance de acción (blast radius); sin una arquitectura de contención, un fallo del agente puede generar cientos o miles de incumplimientos (escala masiva)
.

Al ejecutar comandos o APIs, se debe pasar por canales seguros: por ejemplo, orquestadores que registren cada llamada. Toda ejecución debería planificarse con comandos explícitos (no simples prompts) y validarse contra la spec de destino. En especial, la selección de entorno objetivo (prod, staging, dev) debe codificarse en la especificación, evitando ambigüedades. Herramientas avanzadas recomiendan etiquetar metas y entornos con atributos para que el agente no actúe en un destino erróneo
.

El manejo de secretos cambia radicalmente. Como explica Aembit, los agentes rompen los modelos estáticos de credenciales: ¡no podemos “inyectar” secretos fijos en un agente que actúa en tiempo real! Por ello se promueven credenciales dinámicas de muy corta duración, vinculadas al contexto de la tarea y al “identity” del agente
. Es decir, en cada solicitud el sistema debe emitir un token temporal con permisos scoped al cambio, no llaves de larga vida. Esto previene fugas masivas si el agente es comprometido.

El radio de daño (blast radius) debe controlarse arquitectónicamente: limitar el ámbito de datos/sistemas accesibles. Según Kiteworks, el potencial de daño es proporcional al alcance de acceso del agente
. Un agente con credenciales amplias puede infringir miles de políticas en minutos. Mitigar esto implica usar ABAC/PBAC, políticas previas a la ejecución y revocar inmediatamente cualquier acción fuera de alcance
.

En la parte de supervisión, se requieren checkpoints de aprobación e intensa observabilidad. Cada acción del agente debe estar registrada en tiempo real: los logs de infraestructura no bastan, es necesario generar logs a nivel de operación (quién solicitó qué, qué política aplicó, qué aprobaciones se dieron)
. La integración con SIEM/SOAR y analítica de comportamiento permite detectar anomalías (accesos inusuales, volumen de llamadas anormal, etc.)
. En resumen, con agentes se vuelve crítico: audit trails estructurados e inmutables (para auditoría
), políticas activas de control (Zero Trust a cada acción
) y métricas de monitoreo avanzado.

Finalmente, se debe impedir la expansión autónoma de alcance del agente. Los agentes tienden a explorar más allá de lo pedido (prompt injection es un ejemplo), por lo que conviene limitar explícitamente los dominios de acción, integrar validaciones contra la spec en cada paso, y rotar roles/credenciales en cada llamada
. Con estas barreras –control de permisos, entornos marcados, credenciales dinámicas, monitoreo estricto y auditoría continua– se mitigan los riesgos de operaciones automatizadas.

4. Modelo conceptual para “Agentic SDO”
Un modelo Agentic SDO define roles, artefactos y flujos a lo largo del ciclo. En él participan típicamente:

Agente Orquestador (Coordinator): supervisa el proceso, lanza sub-agentes y rastrea estado.
Agente Planificador/Implementador (Implementor): toma la especificación operacional y ejecuta tareas (shell, API calls, etc.), paso por paso.
Agente Verificador (Verifier): chequea de forma independiente cada resultado contra la spec o criterios predefinidos.
Participantes Humanos: encargados de aprobar propuestas, validar cambios críticos, o intervenir en excepciones.
Autonomía: cada rol puede operar en distintos niveles de autonomía (ver punto 5). Por ejemplo, el Implementor podría usar un modelo de IA de alta capacidad para codificar cambios, mientras que el Verifier podría ser un modelo más ligero encargado sólo de comparar salidas.

Artefactos clave:

Operational Spec: documento de contrato.
Plan/Runbook: secuencia de tareas a ejecutar.
Evidence Package: registro completo de inputs, salidas, logs, hashes.
Validation Report: informe (posible output del agente Verifier) que enumera qué criterios se cumplieron o no.
Archive/History: carpeta o base de datos con la evidencia empaquetada (similar a archive-report en SDD
).
Gates (puertas de control): puntos de validación intermedios. Pueden ser aprobaciones humanas (pre-ejecución o post-ejecución) o verificaciones automáticas (por ejemplo, comprobar que un backup fue exitoso antes de proceder). Cada gate solicita feedback o detiene el flujo hasta recibir la señal adecuada.

Contratos de ejecución: antes de “apply”, el Operative Spec actúa como contrato – el agente must ceñirse a él. Además, se podría definir un Executor Contract formal (p.ej. esquema JSON/YAML) que especifique qué comandos/API están permitidos y bajo qué parámetros. Análogamente, el Validation Contract definiría cómo y con qué reglas debe evaluarse el éxito de cada paso (por ejemplo, tests unitarios, esquemas de datos, tiempos de respuesta, etc.).

Modelo de evidencia: cada acción genera evidencia estructurada. Siguiendo las “principios de rastro” de Cordum, debería vincularse la intención inicial, la política aplicada, la aprobación y el resultado en una misma línea de tiempo
. Idealmente, usar apuntadores inmutables y firmas digitales para cada registro (timestamp, agent_id, policy_snapshot, aprobación, datos de salida)
.

Rollback/Recovery: antes de aplicar cambios, se requiere un plan de recuperación. El modelo conceptual debe incluir un componente de “segundo agente” o mecanismo externo que pueda revertir acciones (snapshot, rollback de base de datos, reversion de configuración) en caso de fallo. En SDO manual se suele prever este plan; en SDO agentico debe ser un artefacto obligatorio del plan operativo, aprobado de antemano.

En conjunto, el modelo Agentic SDO organiza los roles (orquestador, ejecutores, verificadores, humanos), define claramente los niveles de autonomía/autorización y el ciclo de vida: desde la definición de intenciones, a la generación del plan, aprobaciones, ejecución por agentes, verificación independiente y archivado. Cada artefacto y paso debe estar conectado con contratos formales (operativos y de verificación) y ser auditable, como en SDD
.

5. Niveles de autonomía propuestos para SDO
Podemos definir niveles crecientes de autonomía de la IA en SDO, desde mínima a completa:

Solo borradores por IA (AI draft-only): la IA sugiere planificaciones o scripts, pero un humano revisa y ejecuta. Útil en entornos altamente sensibles.
Planificación por IA (AI plans): la IA genera un runbook completo a partir de la spec, incluyendo comandos sugeridos; pero la ejecución la hace el equipo humano.
Propuestas de comandos (AI proposes commands): la IA propone comando por comando (o cambios por cambios) para aprobación humana. El humano valida cada comando antes de correrlo.
Chequeos de solo lectura (AI executes read-only): la IA puede ejecutar scripts de diagnóstico o consultas (p.ej. kubectl get pods, ssh -o check-only) para verificar el estado del sistema, pero no modifica nada. Sirve para monitoreo activo guiado por IA.
Cambios acotados con aprobación (AI executes bounded changes with approval): la IA ejecuta cambios, pero con límites estrictos. Por ejemplo, puede redeployar un servicio en staging, pero requiere aprobación si es producción. Cada bloque de acciones está predefinido y cualquier desviación se bloquea.
Acciones de emergencia bajo política estricta (AI executes emergency actions): en situaciones críticas (incidentes), la IA puede actuar automáticamente bajo guías muy rígidas (p.ej. escalar instancias, aplicar parches urgentes), siempre siguiendo un playbook verificado. Aun así, se registran todas las acciones y se notifica inmediatamente a operadores.
Automatización completa con políticas (fully automated ops): la IA puede llevar a cabo operaciones rutinarias (deployments programados, escalado automático, fixes reconocidos) sin intervención humana, pero siempre dentro de políticas predefinidas (p.ej. “no tocar base de datos en producción sin aprobación previa”). Se apalancan certificados de autonomía que indican qué puede o no puede hacer el agente
.
Estos niveles corresponden a marcos de automatización en la práctica: desde asistente/consultor hasta agente autónomo. Conforme sube el nivel, se aumenta la supervisión técnica: cada nivel más alto requiere controles más fuertes (pruebas previas, gates extras, monitoreo). Tal clasificación ayuda a implementar SDO con IA de forma gradual y segura.

6. Riesgos y mitigaciones
La orquestación agentica introduce riesgos clave que hay que mitigar:

Comandos inventados (hallucinated commands): los agentes pueden “inventar” comandos fuera del contexto. Mitigación: restringir el vocabulario de acciones a un dominio conocido (usar lenguajes de comandos controlados o IA entrenada), y validar cada comando contra la spec usando un verificador externo antes de ejecutarlo. Un paso de aprobación humana final (o automático mediante pruebas de humo) ayuda a detectar falsos positivos.

Objetivo/entorno equivocado: riesgo de que el agente actúe sobre el entorno incorrecto (p.ej. modificar producción en lugar de staging). Mitigación: especificar explícitamente el environment en la spec, y que el agente confirme (“deploy en entorno X”). También usar sistemas separados de credenciales por entorno.

Abuso de privilegios: el agente podría usar permisos elevados para acciones no previstas. Mitigación: conceder permisos mínimos, rotación frecuente de credenciales, uso de identidades cortas (tokens perecederos
) y auditoría continua de privilegios (p.ej. PBAC/Zero Trust).

Fuga de secretos: el agente podría exponer claves sensibles en logs o en outputs. Mitigación: no incluir secretos en prompts; usar vaults dinámicos. Como indica Aembit, hay que pasar a credenciales dinámicas “de uso único” vinculadas al contexto
, de modo que el agente nunca maneje secretos en texto plano. También encriptar en tránsito cualquier archivo de evidencia que pueda contener datos sensibles.

Fabricación u omisión de evidencia: un agente malicioso o desalineado podría falsificar logs o no registrar fallos. Mitigación: diseñar el modelo de evidencia con punteros inmutables y hashing
. Por ejemplo, usar almacenamiento blockchain o HSM para guardar evidencias clave (cada registro ligando al anterior). Integrar múltiples puntos de datos: logs del sistema, del orquestador y del agente, dificulta la omisión sin detección. Revisiones aleatorias de runs (auditorías sorpresa) son recomendables.

Bias de automatización: los humanos pueden tender a sobre-confiar en la IA (“todo pasó el checkpoint, seguro está bien”). Mitigación: rotar responsabilidades (no siempre el mismo ingeniero revisa al agente), exigir que el agente justifique sus acciones (explicar decisiones) y formar al equipo en sesgos de IA. Promover el “plan adversarial” donde otro agente o revisores intenten encontrar fallos intencionalmente antes de aprobar.

Aprobación de teatro (approval theater): pasar formalmente por aprobaciones sin un escrutinio real. Mitigación: instrumentar los procesos de aprobación con responsabilidad (p.ej. firma digital con comentario obligatorio sobre el porqué del sí/no) y auditorías internas de decisiones. De esta forma cada aprobación se convierte en dato de evidencia (igual que en la guía de auditoría
).

Expansión autónoma del scope: un agente no debe “autogestionar” sus propios límites. Mitigación: en el contrato de ejecución se define claramente lo que el agente puede hacer; cualquier intento de ir más allá debe causar un fallo. Políticas de contención en tiempo real (Abort on scope creep) y monitoreo en tiempo real para cortarlo si detecta actividades anómalas
.

Imposibilidad de rollback: sin un plan de reversión, un fallo del agente puede ser catastrófico. Mitigación: siempre generar snapshots o backups previos (como parte del runbook) y mecanismos de undo explícitos. Por ejemplo, antes de un despliegue masivo el agente crea un respaldo que se puede restaurar automáticamente. Insistir en que un rollback sea una “tarea aprobada” dentro del plan.

Responsabilidad ambigua: ¿quién es “dueño” de la acción: el IA o el equipo? Mitigación: formalizar responsabilidades: el que programa el agente o el que lo aprueba debe ser rastreable en cada acción. Cada evento en la evidencia debe incluir el actor asociado (humano o agente)
. Clarificar en políticas internas que un operador es responsable último de validar y monitorear a los agentes.

En resumen, mitigar estos riesgos exige combinar controles técnicos (credenciales dinámicas, logging estructurado
, ZTNA, detection asíncrona
) con procesos organizativos (roles claros, revisiones, entrenamiento). La ingeniería de seguridad para SDO agentico debe tratar a cada agente como una identidad privilegiada, exigiendo “Zero Trust” en cada acción y asegurando auditoría y reversibilidad.

7. Comparativa: SDO tradicional vs SDO Agentic
En un SDO sin IA, los artefactos típicos son: documento de intención, especificaciones operativas (runbooks manuales), listados de tareas, logs de ejecución, reports de validación generados por humanos y evidencias diversas (bitácoras, snapshots). Con IA, la mayoría de estos artefactos se mantienen pero con cambios en formato y énfasis. Por ejemplo, sigue existiendo un Operational Spec y un Runbook, pero ahora idealmente estructurados en formatos legibles por máquinas para que el agente los interprete.

Lo nuevo son artefactos como el Evidence Package formal – un bundle inmutable de logs/versiones de archivos/entradas de base de datos – y el Validation Report automatizado. Además, aparecen contratos de ejecución detallados (esquemas JSON de lo que puede ejecutar el agente) y plantillas de “política de ejecución”.

En controles, con IA se suman mandatos como: políticas de aprobación obligatorias (respuestas explícitas del agente o humanos antes/después), reglas de validación automáticas y monitoreo continuo. Conceptos que eran “buenas prácticas” cobran carácter imprescindible: auditoría completa (obsidian destaca que la seguridad IA requiere auditabilidad y gobernanza específicas
) y trazabilidad de intenciones. En SDO manual ya había trazabilidad, pero con IA se vuelve crítica: “operational runbooks” actuales automatizan chequeos de cumplimiento y generan audit trails como servicio
.

Así, los artefactos de SDO clásicos (especificaciones, bitácoras, reportes) se adaptan a formatos y estructuras formales para agentes. Se añaden controles (credenciales rotatorias, validaciones automáticas, análisis de señales, etc.) antes inexistentes en la versión manual. Y aumenta la importancia de factores como la gobernanza de IA: el cumplimiento normativo y la seguridad se vuelven centrales, como lo evidencian las nuevas regulaciones y guías que exigen explicabilidad y responsabilidad en sistemas IA
. En definitiva, un SDO agentic conserva la esencia del flujo original (intención→plan→ejecución→validación→archivo), pero introduce artefactos formales de contrato y auditoría, y hace prioritarios los controles automáticos de riesgo y la observabilidad avanzada.

8. Reorientar la documentación del proyecto
Documentos base a modificar: Por un lado, los manuales o SOPs (operational playbooks) deben ampliarse para incorporar explícitamente prácticas AI-first. Por ejemplo, incluir secciones sobre cómo redactar un Operational Spec legible para agentes, y cómo escribir runbooks con pasos paramétricos y esquemas de validación. Las plantillas existentes de plan operativo se deben actualizar para incluir campos de “nivel de autonomía” y “procedimientos de rollback”. Los documentos de definición de procesos (como busquedas de cambios) deben mencionar que pueden ejecutarse por IA y qué significa cada flujo.

Gobernanza de IA al frente: La guía de gobernanza de IA (AI Governance) que antes podía ser un anexo secundario debe convertirse en capítulo central. Debería cubrir políticas de aprobación, monitoreo y responsabilidad para agentes. La recomendación de seguridad es explícita: se debe tratar a los agentes como “identidades de alto privilegio con chequeo continuo”
. Así, la política de IA (o IA ética) debe integrarse directamente en el manual SDO, no quedar relegada.

Nuevos documentos necesarios: Se aconseja crear artefactos nuevos, tales como:

Guía de Roles Agenticos: define responsabilidades de coordinador, ejecutor, verificador y humanos en el flujo SDO.
Política de Autonomía y Escenarios: un documento que describa cada nivel de autonomía permitido con ejemplos claros (lo que se puede dejar automático vs no).
Modelo de Evidencia y Auditoría: formato estandarizado (JSON/YAML) para registros de operaciones, logs y reportes, como propuesto por guías de auditoría
.
Plan de Resiliencia/Recuperación: pasos específicos de rollback en un formato pre-aprobado.
Casos de uso de ejemplo: varios runbooks completos ilustrando la versión agentica: por ejemplo, un despliegue con agente vs manual. Estos ejemplos son prioritarios para entender el enfoque AI-first en la práctica.
Mantener compatibilidad manual: Aunque la nueva orientación es AI-first, conviene señalar explícitamente cómo seguir usando SDO manualmente cuando sea necesario. Por ejemplo, se pueden documentar comandos manuales equivalentes o aclarar que los agentes usan las mismas especificaciones que los humanos. De este modo el proyecto no fuerza la automatización en entornos donde no aplique, sino que ofrece ambos caminos. Un enfoque práctico es elaborar las specs/runbooks en formatos neutrales (JSON/YAML) que agentes pueden leer, pero que también un humano puede interpretar. Así, la metodología sigue portable: los documentos centrales cambian (más estructurados) pero siguen sirviendo a un flujo manual cuando se requiera.

9. Arquitectura documental propuesta para Agentic SDO
Proponemos una estructura de documentación jerárquica y modular:

Core Agentic SDO (Manual de Metodología): define la visión general, principios de diseño (AI-first, trazabilidad, gobernanza). Debe incluir una introducción al flujo SDO modificado, relacionándolo con roles y artefactos. Contendrá las políticas básicas (por ejemplo: “Cada ejecución de agente debe incluir autorización previa y verificación posterior”), y un diagrama de alto nivel del proceso (basado en [15] o [46]) que muestre Spec→Plan→(Gate)→Ejecutar→Verificar→Archivo.

Roles de agentes (Especificación de Roles): documento que defina qué tareas realiza cada tipo de agente (Coordinador, Ejecutores, Verificadores). Incluir niveles de autonomía por rol y ejemplo de responsabilidades. Ejemplo: “El Coordinador inicia cambios (sdd-init), lanza sub-agentes, mantiene un DAG de dependencias y solicita aprobaciones humanas
”.

Niveles de autonomía (Guía de Autonomía): detalle de los niveles propuestos, criterios para usarlos, indicadores de riesgo asociados. Podría incluir tablas comparativas (similar a [37]) y ejemplos de cuándo aplicar cada nivel.

Política de ejecución de herramientas (Tool Execution Policy): documento que enumere qué comandos/API están permitidos o prohibidos, cómo se suministran, manejo de errores, etc. Basado en Zero Trust: cada llamada de agente debe validarse en un PDP en tiempo real
. Puede incluir plantillas de autorización de acceso (como ejemplo de [44†L320-L329]).

Modelo de aprobación (Human Approval Model): especifica los puntos donde el humano debe intervenir, la forma de registrarlo (p.ej. firmas digitales, justificaciones), y las reglas de bypass en emergencias. Debe reforzar que no basta con marcar “aprobado”: se requiere contexto (política aplicada, razones), siguiendo recomendaciones de compliance
.

Evidencia y auditoría (Evidence & Audit Model): describe cómo capturar y almacenar evidencia. Debe incluir el esquema de registro mínimo (actor, política, aprobación, detalles) mostrado en [28] como ejemplo, y ejemplos de reportes. Por ejemplo, un JSON de evento maestro que contenga campos como "actor": {"type":"agent","id":"ops-agent-3"}, "policy": {...}, "approval": {...}, "dispatch": {...}, "integrity": {...}, para poder reconstruir la línea de tiempo
. También definir retención y formatos (logs inmutables, blockchain si se usa, etc.).

Rollback y recuperación (Recovery Model): directrices sobre snapshots, backups y cómo revertir. Debe incluir un checklist mínimo: p.ej. “Antes de cambio en prod, confirmar backup y prueba de restore; diseñar tarea inversa automática”. Incluir plantillas de planes de reversión.

Ejemplos y casos prácticos: documentos con runbooks tipo para varios escenarios (despliegue, parche de emergencia, escalado de recursos, migración de base de datos, etc.). Cada ejemplo debe mostrar la versión manual y la agentica. Se prefiere incluir diagramas de flujo (como en [15] o [46]) y artefactos de ejemplo (specks, scripts, logs simulados) para aclarar cómo luce el proceso completo. Estos ejemplos servirán de referencia práctica.

Esta arquitectura documental permite que el proyecto SDO evolucione de “metodología manual” a un framework híbrido: mantiene compatibilidad manual (runbooks escritos legibles) pero promueve un modo operativo centrado en agentes IA. Las referencias citadas muestran la importancia de la estructura y auditoría en flujos basados en runbooks
. Implementar estos documentos hará que SDO sea “AI-first” de forma crítica y práctica, sin perder las buenas prácticas tradicionales.

Fuentes: Investigaciones recientes sobre SDD y operaciones autonomizadas
 respaldan este enfoque. Por ejemplo, los runbooks modernos enfatizan la automatización con IA y generación de audit trails
, y la literatura de seguridad aboga por políticas estrictas de auditoría y Zero Trust en cada acción agentica
. Estos principios guían las recomendaciones aquí expuestas.
