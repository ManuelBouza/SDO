> Research note — non-normative.
> This document is background material and may be superseded by current SDO core/reference documentation.
> Current normative docs start at [../start-here.md](../start-here.md).

Perfecto. Entonces no hablamos de herramientas, sino de una **metodología**.

Yo la llamaría:

# SDO: Spec-Driven Operations

O en español:

# Operaciones Guiadas por Especificación

La idea es adaptar SDD a operaciones TI:

```text
Intención → Especificación → Plan operativo → Procedimiento → Ejecución controlada → Validación → Evidencia → Mejora
```

## Principio central

Ningún cambio operativo debería empezar por comandos.

Debe empezar por una **especificación operativa** que defina:

```text
Qué se quiere cambiar
Por qué
Dónde aplica
Qué riesgos hay
Cómo se valida
Cómo se revierte
Qué evidencia queda
```

---

# Flujo metodológico propuesto

## 1. Intent

Define la intención del cambio.

Ejemplo:

```text
Actualizar Python en una VM Windows sin afectar scripts productivos.
```

Aquí todavía no se escriben comandos.

---

## 2. Spec

Convierte la intención en una especificación clara.

Debe incluir:

```text
Objetivo
Contexto
Alcance
Fuera de alcance
Estado actual
Estado deseado
Riesgos
Suposiciones
Dependencias
Restricciones
Criterios de éxito
Criterios de fallo
Rollback esperado
```

---

## 3. Plan

Define la estrategia operativa.

Debe responder:

```text
Qué se hará primero
Qué se comprobará antes
Qué se modificará
Qué se validará después
Qué se hará si falla
Qué no debe tocarse
```

Todavía puede no tener comandos exactos.

---

## 4. Runbook

Convierte el plan en pasos ejecutables.

Cada paso debe tener:

```text
Nombre del paso
Propósito
Comando o acción
Resultado esperado
Validación
Acción si falla
```

---

## 5. Execution

Ejecuta el runbook de forma controlada.

Reglas:

```text
Un paso a la vez
No avanzar si la validación falla
Guardar salidas importantes
No improvisar cambios fuera del plan
Registrar desviaciones
```

---

## 6. Validation

Comprueba que el sistema quedó en el estado deseado.

Debe validar:

```text
Servicio activo
Versión correcta
Logs sin errores críticos
Conectividad
Permisos
Persistencia tras reinicio si aplica
Scripts o procesos dependientes
```

---

## 7. Evidence

Documenta la evidencia del cambio.

Puede incluir:

```text
Comandos ejecutados
Salidas relevantes
Capturas si aplica
Logs
Antes/después
Fecha
Responsable
Resultado final
Incidencias
```

---

## 8. Rollback

Debe existir antes de ejecutar.

No después.

Debe definir:

```text
Condiciones para revertir
Pasos de reversión
Validación posterior al rollback
Riesgos del rollback
Punto de no retorno
```

---

## 9. Review

Después del cambio, se revisa:

```text
Qué funcionó
Qué falló
Qué se puede automatizar
Qué se debe documentar mejor
Qué validaciones faltaron
```

---

# Estructura resumida

```text
SDO/
├── 01-intent
├── 02-spec
├── 03-plan
├── 04-runbook
├── 05-execution
├── 06-validation
├── 07-evidence
├── 08-rollback
└── 09-review
```

---

# Diferencia con SDD clásico

```text
SDD:
Spec → Plan → Tasks → Code

SDO:
Spec → Plan → Runbook → Change
```

En SDD el producto final es código.

En SDO el producto final es un **cambio operativo seguro, validado y documentado**.

---

# Regla de oro

```text
Todo comando debe estar justificado por una especificación.
Toda ejecución debe tener validación.
Todo cambio debe tener rollback o decisión explícita de no rollback.
```

Esta sería la base metodológica.
