> Research note — non-normative.
> This document is background material and may be superseded by current SDO core/reference documentation.
> Current normative docs start at [../start-here.md](../start-here.md).

# Flujo operativo asistido por IA seguro para AWS, EC2 y Airflow

## Principios rectores

Para este PoC, el patrón más sólido no es “la IA ejecuta”, sino “la IA **propone en un formato revisable** y un humano **aprueba antes de cualquier ejecución**”. Ese enfoque encaja con varias guías oficiales de AWS: operar como código, hacer cambios pequeños y reversibles, usar **runbooks** para procedimientos conocidos y **playbooks** para investigación y respuesta, y asociar cada alerta o evento con un proceso claro y un responsable identificado. citeturn12search9turn12search0turn12search8turn12search19turn18search20

En AWS, ese patrón se puede implementar de forma especialmente limpia con **Systems Manager Automation** para cambios no triviales, porque Automation permite insertar pasos de aprobación manual con `aws:approve`, pausar hasta que los aprobadores designados acepten o rechacen, y luego continuar con el runbook aprobado. AWS también mantiene runbooks predefinidos y algunos incluyen aprobación explícita en versiones `WithApproval`. citeturn9search0turn9search3turn9search10turn9search17

Para nuevas adopciones, conviene no basar el diseño en **Change Manager** como dependencia central. AWS documenta que **Change Manager dejó de estar abierto a nuevos clientes a partir del 7 de noviembre de 2025**; los clientes existentes pueden seguir usándolo, pero para un diseño nuevo es más prudente apoyarse en **Automation + `aws:approve` + Change Calendar + Maintenance Windows**. Si ya sois clientes existentes, Change Manager aporta aprobaciones multinivel y plantillas; si no, no merece la pena introducirlo como pieza esencial del flujo. citeturn13search1turn13search2turn13search3turn13search12

La separación de canales también importa. Para **cambios** o acciones dentro de la instancia, es preferible **Run Command** o **Automation** frente a una shell interactiva libre, porque Run Command permite parametrizar el documento, capturar salida en CloudWatch Logs y S3, consultar estados e invocaciones, y aplicar controles de concurrencia y error. Session Manager sigue siendo útil, pero sobre todo para **investigación acotada** y siempre bajo restricciones adicionales. citeturn35search11turn20search0turn20search1turn16search2turn31search17

Hay una razón de seguridad concreta para no normalizar shells libres del agente en EC2. En Linux y macOS, Session Manager usa por defecto el usuario `ssm-user`, y AWS indica que en esos sistemas ese usuario se agrega a `/etc/sudoers`; además, AWS recomienda restringir `SendCommand` y `StartSession` a principals concretos y a nodos concretos. Si queréis sesiones interactivas, debéis activar **Run As** o usar **Session documents** que limiten comandos y parámetros. citeturn14search0turn14search6turn14search7turn14search1

La auditabilidad también condiciona el canal. Session Manager puede registrar comandos y salida en S3 o CloudWatch Logs, pero AWS documenta que **no hay logging de sesión** cuando la conexión se hace por **SSH o port forwarding** a través de Session Manager. Por tanto, cuando el objetivo es dejar evidencia completa, evitad SSH/port-forward y usad Run Command, Automation o Session Manager estándar con logging activado. citeturn9search2turn9search9turn9search12turn35search10

## Modelo de clasificación

La clasificación que mejor funciona aquí no es una sola etiqueta, sino **una clase base** de mutabilidad y **capas adicionales** de sensibilidad. Eso evita errores como tratar `get-secret-value` como “solo lectura” sin reconocer que devuelve **texto plano sensible**, o tratar `restore-db-instance-from-db-snapshot` como simple “restore” cuando en realidad crea una **nueva instancia de base de datos** y, por tanto, tiene impacto operativo y de coste. citeturn10search0turn10search2turn11search13turn15search8

| Clase | Qué significa en este diseño | Canal preferido | Aprobación mínima propuesta |
|---|---|---|---|
| **read-only** | No cambia estado; solo observa o lista | AWS CLI directo, Run Command read-only, o Session Manager restringido | 1 aprobador |
| **state-changing reversible** | Cambia estado, pero hay rollback directo y probado | Run Command o Automation | 1 aprobador |
| **state-changing hard-to-revert** | Cambia estado y el rollback no es inmediato ni idéntico | **Automation** con `aws:approve` | 2 aprobadores |
| **destructive** | Borra, destruye o puede causar pérdida permanente | Solo runbook “break-glass” | 2 aprobadores + justificación explícita |
| **secret-sensitive** | Puede exponer secretos o plaintext sensible | Canal restringido; salida redacted o sin stdout | Overlay obligatorio |
| **cost-impacting** | Puede crear recursos, ampliar uso facturable o habilitar logging caro | Igual que la clase base, pero con coste/timebox/cleanup | Overlay obligatorio |

Mi recomendación concreta es tratar **secret-sensitive** y **cost-impacting** como *overlays* que se suman a la clase base, no como clases mutuamente exclusivas. Así, por ejemplo, `aws secretsmanager get-secret-value` sería **read-only + secret-sensitive**, mientras que `aws rds restore-db-instance-from-db-snapshot` sería **state-changing hard-to-revert + cost-impacting**. Esa separación es más útil operativamente que una taxonomía plana. La necesidad del overlay secreto está respaldada por la documentación de AWS: `GetSecretValue` y `get-parameter --with-decryption` devuelven plaintext sensible y AWS indica explícitamente que hay que evitar registrar esos valores en logs propios; la necesidad del overlay coste nace de comandos que lanzan nuevas instancias o restauran nuevas bases de datos, y de funciones como los **CloudTrail data events**, que no se registran por defecto y tienen **coste adicional**. citeturn10search0turn10search2turn10search8turn11search13turn15search8turn11search16

Además de esa clasificación, yo exigiría dos campos más en cada propuesta: **`target_scope`** y **`executor`**. `target_scope` debe indicar si el cambio toca una sola instancia, una lista explícita o un conjunto resuelto por tags; `executor` debe fijar si la ejecución sería AWS CLI directo, Run Command, Session Manager restringido o Automation. Esto es importante porque AWS permite dirigir Automation o Run Command por tags y grupos, y eso puede multiplicar el alcance real de una acción. citeturn21search5turn17search1turn17search6

## Flujo de aprobación y ejecución

El flujo operativo que recomiendo para este repo es el siguiente: **clasificar → proponer → aprobar → ejecutar → capturar evidencia → validar → cerrar o revertir**. No interesa dejar a la IA “improvisar” en terminal; interesa que produzca un **paquete de cambio revisable** con el comando exacto, el porqué, los parámetros, el riesgo, el rollback y la evidencia esperada. Esto está muy alineado con la idea de runbooks/playbooks de AWS y con la práctica de cambios pequeños y reversibles. citeturn12search8turn18search6turn18search0turn12search0

En la fase de **propuesta**, la IA debería generar siempre un artefacto estructurado y, cuando el executor sea AWS CLI o SSM, acompañarlo de un **manifest** o input revisable. AWS CLI soporta esqueletos e inputs desde JSON/YAML, y muchos comandos soportan `--dry-run`; cuando un comando no soporta `dry-run`, `--generate-cli-skeleton output` sirve al menos para validar inputs localmente sin enviar la petición. Para el output, es mejor **JSON** que `text`, porque AWS advierte que `--query` combinado con `--output text` puede dar resultados sorprendentes por paginación. citeturn15search0turn15search4turn22search0turn22search1turn22search6

En la fase de **aprobación**, el humano no debería aprobar una idea difusa, sino un **objeto exacto**: comando, documento SSM, versión del documento, targets resueltos, precondiciones, rollback y validaciones. Para SSM, antes de aprobar una ejecución es útil que la IA incluya también el resultado de `describe-document` o `get-document`, porque AWS-RunShellScript y otros documentos tienen parámetros y timeouts que conviene inspeccionar explícitamente. AWS también permite fijar **versiones concretas** de documentos, lo que mejora la reproducibilidad frente a `$LATEST`. citeturn31search1turn31search3turn31search7turn31search19

En la fase de **ejecución**, mi recomendación es esta regla simple:  
**read-only** puede ir por AWS CLI directo o Session Manager restringido;  
**state-changing reversible** debería ir por Run Command o Automation;  
**hard-to-revert** y **destructive** deberían ir **solo** por Automation o un runbook de emergencia explícito, nunca por shell libre. El motivo es que Automation aporta aprobación integrada, salidas observables, rate controls y puntos de validación. citeturn9search0turn21search3turn17search6turn20search9

Para operaciones sobre varios targets, la propuesta debe incluir siempre una **expansión previa de targets** y, después, una ejecución con límites estrictos. En Run Command, AWS permite `--max-concurrency` y `--max-errors`; en Automation existen equivalentes. Si necesitáis un límite exacto de fallo para cambios sensibles, AWS recomienda ir incluso con concurrencia 1, porque cuando se alcanza `MaxErrors` todavía puede haber automatizaciones en vuelo que terminen después. En un PoC de una sola EC2 esto parece redundante, pero es una buena costumbre porque protege el día en que cambiéis de un `instance-id` explícito a un target por tags. citeturn17search3turn17search11turn17search16turn17search7turn21search5

La fase de **evidencia** debe apoyarse en varias capas. CloudTrail registra por defecto los **management events**, incluidos llamados hechos desde consola, SDK o AWS CLI; Session Manager puede dejar rastro de actividad y, si se configura, de contenido de sesión; Run Command puede enviar salida a CloudWatch Logs y S3; y Automation puede enviar output a CloudWatch Logs y expone un `automation:EXECUTION_ID` para correlación. Si no configuráis salida de Run Command, solo veréis los primeros **24.000 caracteres** en la respuesta, lo que es insuficiente para auditoría. citeturn0search2turn11search20turn9search12turn9search2turn20search0turn20search4turn20search9turn21search6

Por último, la fase de **validación** no debe limitarse a “salió exit code 0”. AWS y Airflow ofrecen mejores pruebas objetivas: waiters de AWS CLI para snapshots o bases de datos, `get-command-invocation` para estado y salida de SSM, `/api/v2/monitor/health` y `airflow jobs check` para salud de Airflow, `airflow db check` para base de datos y `airflow dags list-import-errors` para verificar que el cambio no rompió importación de DAGs. Además, AWS documenta que `get-command-invocation` sigue un modelo de **eventual consistency**, así que el polling debe tener pequeños reintentos y backoff. citeturn16search1turn16search0turn16search2turn31search11turn19search0turn24search0turn24search2turn19search5

## Plantilla de harness

La plantilla base que os recomiendo para cualquier propuesta es esta:

```md
id: CHG-YYYYMMDD-### 
objetivo:
clase_base: read-only | state-changing reversible | state-changing hard-to-revert | destructive
overlays: [secret-sensitive, cost-impacting]
target_scope:
executor: aws-cli | ssm-run-command | ssm-session-restricted | ssm-automation
change_window:
referencias:
  issue_or_ticket:
  approval_ref:
  runbook_ref:

precondiciones:
  - permisos IAM
  - targets resueltos explícitamente
  - backup/snapshot/AMI si aplica
  - calendar OPEN / maintenance window si aplica
  - documento SSM y versión fijados si aplica

comando_o_manifest:
explicacion_de_parametros:
riesgos:
rollback:
evidencia_esperada:
criterio_de_aprobacion:
criterio_de_exito:
criterio_de_fallo:
```

Esa forma refleja bastante bien lo que AWS pide que contengan playbooks y runbooks: overview, prerrequisitos, pasos técnicos y condiciones de verificación. Para comandos AWS CLI y SSM, además, os conviene acompañar el markdown con un **input JSON revisable** y, cuando sea posible, con un `dry-run`, `describe-document` o `get-document` previo. citeturn18search6turn12search8turn15search4turn31search1turn31search3

Las adaptaciones por clase deberían ser estas:

**Read-only.** Debe incluir finalidad, target exacto y evidencia esperada, pero el rollback puede declararse como “no aplica”. Si la lectura es interactiva, usad Session Manager con documento restringido o Run Command; si el objetivo es evidencia duradera, preferid Run Command porque deja mejor rastro técnico. Si la lectura expone secretos, deja de ser un read-only simple y pasa a overlay **secret-sensitive**. citeturn14search1turn35search5turn20search0turn10search0turn10search2

**State-changing reversible.** Debe traer rollback directo y probado en la misma propuesta: “qué hago para volver al estado anterior”. También debe incluir captura de **pre-state** y **post-state**. AWS Well-Architected recomienda precisamente cambios pequeños, frecuentes y reversibles para reducir el radio de impacto y acelerar la remediación. citeturn12search0turn12search13

**State-changing hard-to-revert.** Aquí el rollback “en un comando” ya no existe. Por eso el harness debe exigir **backup ID previo** y un plan de restore real, no solo “si falla restauramos”. En vuestro contexto, eso aplica clarísimamente a **migraciones de base de datos de Airflow**: Airflow indica que `airflow db migrate` debe ejecutarse con los componentes parados, y el restore de RDS desde snapshot crea una **nueva** instancia, no un undo instantáneo sobre la actual. citeturn19search6turn19search14turn11search16turn11search13

**Destructive.** Aquí el harness debería volverse excepcional. Mi recomendación es que el agente **nunca** ejecute esta clase desde shell general, aunque exista aprobación; debe limitarse a proponer un runbook “break-glass” con target explícito, backup validado o aceptación explícita de no-rollback, y una segunda revisión humana antes de pulsar ejecutar. Aunque algunos comandos como `terminate-instances` soportan `--dry-run`, AWS aclara que ese flag solo comprueba permisos, no simula consecuencias reales. citeturn15search5turn15search0

**Overlay secret-sensitive.** Si un comando puede devolver plaintext de un secreto —por ejemplo `get-secret-value` o `get-parameter --with-decryption`— el harness debe **prohibir stdout persistente**, prohibir copiar el valor a evidencias, y preferir comprobaciones por metadata (`describe-secret`, ARN, stage, version ID) o por validación funcional indirecta. Esto es importante porque Session Manager y Run Command pueden registrar comandos y salidas, y AWS pide explícitamente no registrar esos valores sensibles en logs propios. citeturn10search0turn10search2turn10search8turn9search2turn20search0

**Overlay cost-impacting.** El harness debe añadir un mini-bloque de coste con “qué recurso nuevo se crea o qué metrado adicional se habilita”, “TTL o fecha de revisión” y “owner de cleanup”. Esa capa debe activarse, por ejemplo, al lanzar instancias, restaurar RDS a una nueva instancia o habilitar data events de CloudTrail. citeturn15search8turn11search13turn11search16

## Ejemplos prácticos

Los comandos de esta sección son ejemplos de diseño para harnesses. No son artefactos aprobados para ejecución directa; antes de usarlos deben convertirse en una propuesta concreta con targets reales, clasificación, riesgos, rollback, evidencia esperada y aprobación humana explícita.

**Ejemplo read-only para snapshot de salud de Airflow.** Para un PoC como el vuestro, un buen primer harness es un snapshot de salud que combine comprobaciones de scheduler, DB e import errors. Airflow documenta `/api/v2/monitor/health`, `airflow jobs check`, `airflow db check` y `airflow dags list-import-errors`; Run Command permite ejecutarlo sin login y enviar toda la salida a CloudWatch Logs y S3. citeturn19search0turn24search0turn24search2turn19search5turn20search0turn20search1

```bash
aws ssm send-command \
  --instance-ids "<instance-id>" \
  --document-name "AWS-RunShellScript" \
  --document-version "$DEFAULT" \
  --parameters 'commands=[
    "airflow jobs check --job-type SchedulerJob --local",
    "airflow db check",
    "airflow dags list-import-errors --output json",
    "curl -fsS http://127.0.0.1:8080/api/v2/monitor/health"
  ],executionTimeout=["120"],workingDirectory=["/"]' \
  --comment "readonly: airflow health snapshot CHG-001" \
  --max-concurrency "1" \
  --max-errors "0" \
  --cloud-watch-output-config "CloudWatchOutputEnabled=true,CloudWatchLogGroupName=/aws/ssm/airflow-readonly" \
  --output-s3-bucket-name "<evidence-bucket>" \
  --output-s3-key-prefix "ssm/readonly/CHG-001/"
```

La evidencia esperada aquí es: `CommandId`, salida completa en CloudWatch/S3, `StatusDetails` de `get-command-invocation`, y un resumen breve en markdown con “scheduler OK / DB OK / import errors = [] / health endpoint OK”. Como `get-command-invocation` es eventualmente consistente, el polling debe reintentar unos segundos antes de concluir fallo de evidencia. citeturn16search2turn31search11turn20search8

**Ejemplo de Session Manager acotado para lectura interactiva.** Si el humano quiere mirar un log en directo, no dejéis que el agente proponga una shell libre; que proponga un **Session document** acotado. AWS documenta expresamente este patrón con `sessionType: InteractiveCommands`, parámetros validados con `allowedPattern` y una orden fija, y añade que este tipo de documentos solo funciona en sesiones iniciadas desde AWS CLI. citeturn14search1turn35search1turn35search4

```yaml
---
schemaVersion: '1.0'
description: Read-only tail for Airflow logs
sessionType: InteractiveCommands
parameters:
  logpath:
    type: String
    default: "/var/log/airflow/scheduler.log"
    allowedPattern: "^[a-zA-Z0-9._/\\-]+$"
  lines:
    type: String
    default: "200"
    allowedPattern: "^[0-9]{1,4}$"
properties:
  linux:
    commands: "tail -n {{ lines }} {{ logpath }}"
    runAsElevated: false
```

```bash
aws ssm start-session \
  --target "<instance-id>" \
  --document-name "Airflow-ReadOnly-Tail" \
  --parameters 'logpath=["/var/log/airflow/scheduler.log"],lines=["200"]'
```

Este harness es útil para investigación, pero no debéis sustituir con él a Run Command cuando la prioridad sea evidencia durable. Y, de nuevo, si necesitáis auditoría de contenido, evitad SSH y port forwarding porque AWS no registra esas sesiones. citeturn9search9turn35search10turn9search2

**Ejemplo reversible para reinicio controlado de un servicio Airflow.** Aquí conviene exigir pre-state, cambio y validación posterior. En un repo interno yo lo modelaría como Run Command o Automation, con nombre de servicio parametrizado para no asumir units concretos. La reversibilidad está en que el cambio es acotado, la validación es inmediata y el rollback es volver al estado previo del servicio o ejecutar el restart inverso tras restaurar la configuración conocida si el cambio iba acompañado de edición de config. El patrón de pequeños cambios reversibles está alineado con Well-Architected. citeturn12search0turn35search11turn20search0

```bash
aws ssm send-command \
  --instance-ids "<instance-id>" \
  --document-name "AWS-RunShellScript" \
  --document-version "$DEFAULT" \
  --parameters 'commands=[
    "systemctl is-active <airflow-service>",
    "sudo systemctl restart <airflow-service>",
    "sleep 10",
    "systemctl is-active <airflow-service>",
    "airflow jobs check --job-type SchedulerJob --local"
  ],executionTimeout=["180"]' \
  --comment "reversible: restart airflow service CHG-002" \
  --max-concurrency "1" \
  --max-errors "0" \
  --cloud-watch-output-config "CloudWatchOutputEnabled=true,CloudWatchLogGroupName=/aws/ssm/airflow-change" \
  --output-s3-bucket-name "<evidence-bucket>" \
  --output-s3-key-prefix "ssm/change/CHG-002/"
```

**Ejemplo hard-to-revert para migración de base de datos de Airflow.** Este es el caso que yo elevaría sin dudar a **Automation con aprobación humana**. Airflow indica que `airflow db migrate` puede aplicar migraciones de esquema y que los componentes no deben estar corriendo durante la migración; RDS exige que el snapshot manual se tome cuando la instancia está `available` o en `storage-optimization`, y el restore desde snapshot crea una nueva instancia. Eso encaja de lleno en la clase hard-to-revert. citeturn19search6turn19search14turn11search9turn11search16turn11search13turn9search0

Pre-backup propuesto:

```bash
aws rds create-db-snapshot \
  --db-instance-identifier "<rds-id>" \
  --db-snapshot-identifier "chg-003-pre-airflow-db-migrate"

aws rds wait db-snapshot-available \
  --db-snapshot-identifier "chg-003-pre-airflow-db-migrate"
```

Cambio en EC2 propuesto vía SSM:

```bash
aws ssm send-command \
  --instance-ids "<instance-id>" \
  --document-name "AWS-RunShellScript" \
  --document-version "$DEFAULT" \
  --parameters 'commands=[
    "sudo systemctl stop <airflow-scheduler.service>",
    "sudo systemctl stop <airflow-webserver.service>",
    "airflow db migrate",
    "airflow db check",
    "sudo systemctl start <airflow-webserver.service>",
    "sudo systemctl start <airflow-scheduler.service>",
    "airflow jobs check --job-type SchedulerJob --local"
  ],executionTimeout=["1800"]' \
  --comment "hard-to-revert: airflow db migrate CHG-003" \
  --max-concurrency "1" \
  --max-errors "0" \
  --cloud-watch-output-config "CloudWatchOutputEnabled=true,CloudWatchLogGroupName=/aws/ssm/airflow-db-migrate" \
  --output-s3-bucket-name "<evidence-bucket>" \
  --output-s3-key-prefix "ssm/hard-to-revert/CHG-003/"
```

Rollback propuesto:

```bash
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier "<restore-rds-id>" \
  --db-snapshot-identifier "chg-003-pre-airflow-db-migrate"

aws rds wait db-instance-available \
  --db-instance-identifier "<restore-rds-id>"
```

Aquí el cierre correcto no es “migró bien”, sino “hay snapshot previo, evidence completa, Airflow volvió sano, y existe plan documentado para repuntar al restore si el comportamiento posterior degrada”. Para cambios destructivos como `terminate-instances`, mi recomendación es más dura: el agente puede **proponer** incluso un `--dry-run`, pero la ejecución real debería quedar reservada a un runbook de emergencia y doble aprobación humana. citeturn16search0turn24search0turn15search5turn15search0

## Encaje con Gentle AI

La buena noticia es que no necesitáis reinventar varias piezas que Gentle AI ya trae. En el repositorio público `gentle-ai`, el directorio de skills embebidas incluye, entre otras, `branch-pr`, `judgment-day`, las fases `sdd-*`, `skill-creator`, `skill-registry` y `work-unit-commits`. La documentación del proyecto también explica que Gentle-AI se concibe como un configurador con workflow SDD, skills y permisos orientados a seguridad, y que el registro de skills se refresca en `.atl/skill-registry.md`. citeturn7view0turn30view0turn30view1turn28view0turn28view1turn34search0turn33search1turn27search11

También es importante distinguir entre **skills embebidas** y **skills de catálogo externo**. La documentación pública de Gentle AI indica que las skills SDD y de base se instalan directamente, mientras que muchas skills de framework o comunidad viven en el repositorio separado `Gentleman-Skills`. En el catálogo público que revisé, `Gentleman-Skills` lista skills de frontend/backend/workflow y testing, pero no vi una skill pública ya catalogada para **AWS/SSM/Airflow operations** ni en `curated/` ni en `community/`. citeturn3search7turn1search15turn25search0turn32view0turn32view1

Por tanto, mi recomendación es esta:

| Necesidad | Ya cubierto por Gentle AI | Recomendación |
|---|---|---|
| Registro y resolución de skills | `skill-registry` | **Reutilizar** |
| Creación de nuevas skills | `skill-creator` | **Reutilizar** |
| Dividir cambios en unidades revisables y reversibles | `work-unit-commits` | **Reutilizar** |
| Flujo issue/PR y shellcheck en scripts | `branch-pr` | **Reutilizar** |
| Revisión adversarial antes de mergear harnesses | `judgment-day` | **Reutilizar** |
| Diseño e implementación del flujo | `sdd-propose`, `sdd-design`, `sdd-verify`, etc. | **Reutilizar** |
| Operación AWS/EC2/Airflow con aprobación humana | No encontré skill pública equivalente en los catálogos revisados | **Crear project-local skills nuevas** |

La clave del punto anterior es **no solapar**. No propongo nuevas skills para registro, PRs, commits, shellcheck o revisión dual porque ya existen en el ecosistema público revisado. Sí propongo añadir **skills de dominio**, con nombres nuevos y estrechos, por ejemplo:

- `aws-hitl-ops`: clasifica el comando, elige el canal de ejecución, genera el paquete de aprobación y exige rollback/evidencia.
- `aws-airflow-runbooks`: contiene harnesses concretos para salud, restart, migraciones, logs e incidentes del PoC Airflow.
- `aws-secret-safe-ops`: solo si de verdad vais a trabajar con rotaciones, validaciones o troubleshooting de Secrets Manager/Parameter Store frecuentemente.

Esta recomendación no entra en conflicto con Gentle AI por dos motivos. Primero, `skill-creator` dice explícitamente que hay que crear skills cuando un patrón es repetido, específico del proyecto o necesita decisión guiada paso a paso. Segundo, `gentle-ai` documenta que **las skills project-local ganan frente a las globales del mismo nombre**, así que para evitar overrides accidentales conviene usar nombres nuevos como los anteriores en lugar de reaprovechar un nombre genérico. Tras añadirlas, deberíais registrarlas en `AGENTS.md` y refrescar el skill registry. citeturn29view0turn27search11turn28view1

Un matiz importante: si vuestra instalación local ya tiene skills privadas de “runbooks”, “secrets” o similares que **no están en el catálogo público**, entonces antes de crear nada nuevo deberíais refrescar `skill-registry` y comprobar qué existe realmente en vuestro workspace, porque el propio `skill-registry` declara que `SKILL.md` es la fuente de verdad y que su función es precisamente indexar rutas y triggers disponibles. citeturn28view1turn34search0

## Diseño recomendado para este PoC

Si tuviera que dejar esto operativo en vuestro repo sin inflarlo demasiado, haría esto:

Primero, definiría una **política única** en el repo: el agente puede **proponer** AWS CLI, SSM y comandos in-node, pero no puede ejecutarlos hasta que haya una aprobación humana referenciando un `change_id` concreto. Esa política viviría en `AGENTS.md` y sería consumida por una skill project-local `aws-hitl-ops`, creada con `skill-creator` y registrada después con `skill-registry`. citeturn29view0turn28view1turn27search11

Segundo, crearía un árbol de artefactos simple y estable, por ejemplo `docs/ops/change-classes.md`, `docs/ops/harness-template.md`, `ops/proposals/`, `ops/manifests/`, `ops/evidence/` y `ops/runbooks/`. No hace falta una plataforma nueva; hace falta que cada cambio tenga propuesta, evidencia y rollback en rutas predecibles. Ese enfoque casa bien con el modelo index-first de skills y con el patrón de work-unit commits de Gentle AI. citeturn27search2turn28view0

Tercero, empezaría por **tres harnesses** y no más:  
un harness **read-only** de salud de Airflow;  
un harness **reversible** de restart controlado de servicios;  
y un harness **hard-to-revert** para `airflow db migrate` con snapshot previo de RDS. Con solo esos tres cubrís observabilidad, cambio operativo frecuente y cambio delicado de base de datos. Airflow y AWS ya os dan los mecanismos de validación y restore necesarios para que esos runbooks sean realmente auditables. citeturn19search0turn19search5turn19search14turn11search16turn11search13

Cuarto, dejaría **destructive** como clase casi prohibida para el agente generalista. La IA puede redactar el plan, enumerar targets, adjuntar `dry-run` y preparar el runbook de emergencia; la ejecución real debería quedar fuera del circuito normal y bajo doble validación humana. Ese sesgo conservador está más alineado con incident-safe automation que con “full autonomy”, y en este contexto es lo correcto. citeturn18search3turn18search9turn15search5turn15search0

Quinto, activaría evidencia en capas: **CloudTrail** para API management events, **Session Manager logging** para investigación interactiva acotada, **Run Command output a CloudWatch Logs y S3** para comandos node-side, y **Automation output** para runbooks de cambio. Con eso tendréis una cadena razonable de quién aprobó, qué se ejecutó, dónde se ejecutó, qué devolvió y cómo se validó. citeturn0search2turn9search12turn9search2turn20search0turn20search9

Sexto, usaría las skills que ya existen para la calidad del propio sistema: `branch-pr` para issue-first + shellcheck + PR checks, `work-unit-commits` para que cada harness o runbook caiga en commits revisables y con rollback claro, y `judgment-day` para revisión dual antes de mergear cambios delicados en estas automatizaciones. Eso sí evita duplicidades y aprovecha exactamente lo que Gentle AI ya os da. citeturn30view0turn28view0turn30view1

La conclusión práctica es esta: para vuestro PoC, el mejor diseño no es un “agente operador”, sino un **agente preparador de cambios**. Debe producir comandos y manifests explícitos, explicar parámetros y riesgos, esperar aprobación, ejecutar solo por canales auditables, capturar evidencia estructurada y dejar rollback documentado. Con AWS Systems Manager y las skills públicas de Gentle AI que ya existen, eso es perfectamente viable sin pelearos contra el ecosistema actual ni introducir una capa nueva innecesaria. citeturn9search0turn21search3turn34search0turn28view1
