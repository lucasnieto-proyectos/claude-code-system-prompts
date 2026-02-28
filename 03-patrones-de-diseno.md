# Patrones de Diseño en los System Prompts de Claude Code

> **Versión de Claude Code**: v2.1.62  
> **Total de prompts analizados**: 223  
> **Fecha**: 28 de febrero de 2026

---

## Índice

1. [Patrón 1: Metadatos Estructurados](#1-patrón-1-metadatos-estructurados)
2. [Patrón 2: Modularidad Atómica](#2-patrón-2-modularidad-atómica)
3. [Patrón 3: Composición por Plantillas](#3-patrón-3-composición-por-plantillas)
4. [Patrón 4: Ejemplificación Few-Shot](#4-patrón-4-ejemplificación-few-shot)
5. [Patrón 5: Directivas con Prioridad](#5-patrón-5-directivas-con-prioridad)
6. [Patrón 6: Seguridad en Capas](#6-patrón-6-seguridad-en-capas)
7. [Patrón 7: Orquestación Multi-Agente](#7-patrón-7-orquestación-multi-agente)
8. [Patrón 8: Fases de Ejecución](#8-patrón-8-fases-de-ejecución)
9. [Patrón 9: Inyección Contextual Dinámica](#9-patrón-9-inyección-contextual-dinámica)
10. [Patrón 10: Restricciones Negativas Explícitas](#10-patrón-10-restricciones-negativas-explícitas)
11. [Base para un Diseñador de System Prompts](#11-base-para-un-diseñador-de-system-prompts)

---

## 1. Patrón 1: Metadatos Estructurados

### Descripción

Todos los prompts incluyen un bloque de metadatos en formato de comentario HTML al inicio del archivo con información estandarizada.

### Estructura

```html
<!--
name: 'Categoría: Nombre descriptivo'
description: Descripción concisa de una línea
ccVersion: 2.1.62
variables:
  - NOMBRE_VARIABLE_1
  - NOMBRE_VARIABLE_2
-->
```

### Elementos

| Campo | Obligatorio | Descripción |
|---|---|---|
| `name` | ✅ | Nombre con prefijo de categoría (`Agent Prompt:`, `System Prompt:`, etc.) |
| `description` | ✅ | Descripción de una línea sobre la función del prompt |
| `ccVersion` | ✅ | Versión de Claude Code en la que se introdujo/modificó |
| `variables` | ❌ | Lista de variables runtime inyectadas por Claude Code |

### Convención de Nombres

El nombre sigue un esquema jerárquico de `Categoría: Subcategoría (detalle)`:

```
Agent Prompt: Bash command prefix detection
System Prompt: Doing tasks (no premature abstractions)
Tool Description: Bash (sandbox — mandatory mode)
System Reminder: Plan mode is active (5-phase)
Data: Claude API reference — Python
Skill: Build with Claude API
```

### Uso en un Diseñador de Prompts

Todo system prompt que diseñemos debería comenzar con un bloque de metadatos que identifique:
- **Nombre canónico** — para referencia programática
- **Versión** — para trazabilidad de cambios
- **Variables** — para documentar dependencias externas
- **Categoría** — para organización automática

---

## 2. Patrón 2: Modularidad Atómica

### Descripción

Los prompts siguen el principio de **responsabilidad única**: cada archivo contiene una directiva específica y autocontenida. En lugar de un system prompt monolítico, se componen docenas de fragmentos pequeños y especializados.

### Evidencias

```
system-prompt-doing-tasks-avoid-over-engineering.md      → 30 tokens
system-prompt-doing-tasks-no-premature-abstractions.md   → 60 tokens
system-prompt-doing-tasks-no-unnecessary-additions.md    → 78 tokens
system-prompt-doing-tasks-read-before-modifying.md       → 46 tokens
```

Cada uno de estos archivos contiene **una única regla clara**, por ejemplo:

> "Only make changes that are requested or clearly necessary. Do not add additional code, create extra files, or implement features beyond what is explicitly asked for."

### Beneficios

1. **Mantenimiento**: Cada regla se modifica independientemente sin afectar a las demás
2. **Composición selectiva**: Se pueden activar/desactivar reglas según el contexto
3. **Testabilidad**: Cada regla se puede validar por separado
4. **Versionado**: Los cambios en el changelog son granulares y trazables

### Uso en un Diseñador de Prompts

El diseñador debería:
- Descomponer cualquier instrucción compleja en **directivas atómicas** (máx. 30-100 tokens cada una)
- Mantener **una idea por archivo/bloque**
- Permitir la **activación condicional** de cada directiva

---

## 3. Patrón 3: Composición por Plantillas

### Descripción

Los prompts utilizan un sistema de **plantillas con variables** que se interpolan en runtime. Esto permite que un mismo prompt se adapte a diferentes configuraciones.

### Arquitectura Interna

Según `updatePrompts.js`, internamente cada prompt está compuesto por:

```javascript
{
  pieces: ["Texto antes de la variable ", "texto después de la variable"],
  identifiers: [42],
  identifierMap: { "42": "TASK_TOOL_NAME" }
}
```

Que se reconstruye como:

```
Texto antes de la variable ${TASK_TOOL_NAME} texto después de la variable
```

### Tipos de Variables Encontradas

| Variable | Propósito | Ejemplo de uso |
|---|---|---|
| `${TASK_TOOL_NAME}` | Nombre de la herramienta Task | "Use the ${TASK_TOOL_NAME} tool to launch..." |
| `${EDIT_TOOL_NAME}` | Nombre de la herramienta Edit | "Use the ${EDIT_TOOL_NAME} tool instead of sed" |
| `${WRITE_TOOL}` | Objeto de la herramienta Write | "Create your plan using the ${WRITE_TOOL.name} tool" |
| `${BASH_TOOL_NAME}` | Nombre de la herramienta Bash | "Reserve ${BASH_TOOL_NAME} for system commands" |
| `${SYSTEM_REMINDER}` | Objeto con estado del sistema | `${SYSTEM_REMINDER.planExists}`, `${SYSTEM_REMINDER.planFilePath}` |
| `${AVAILABLE_TOOLS_SET}` | Set de herramientas disponibles | `${AVAILABLE_TOOLS_SET.has(ASK_USER_QUESTION_TOOL)}` |
| `${GET_SUBSCRIPTION_TYPE_FN}` | Tipo de suscripción | `${GET_SUBSCRIPTION_TYPE_FN()!=="pro" ? ... : ""}` |

### Lógica Condicional en Variables

Algunos prompts usan **expresiones ternarias** dentro de las variables para contenido condicional:

```javascript
${GET_SUBSCRIPTION_TYPE_FN()!=="pro" 
  ? "Launch multiple agents concurrently..." 
  : ""}
```

Esto permite adaptar el prompt según:
- Nivel de suscripción del usuario (pro vs free)
- Herramientas disponibles
- Estado del sistema (plan mode activo, equipo creado, etc.)

### Uso en un Diseñador de Prompts

El diseñador debería:
- Definir **slots de variable** con `${NOMBRE}` para inyección en runtime
- Soportar **lógica condicional** para activar/desactivar secciones
- Mantener un **diccionario de variables** con su tipo y valores posibles

---

## 4. Patrón 4: Ejemplificación Few-Shot

### Descripción

Los prompts más complejos utilizan intensivamente **ejemplos estructurados** para guiar el comportamiento del modelo. Estos ejemplos se encierran en etiquetas XML personalizadas.

### Formatos Encontrados

#### a) Etiquetas `<example>` con razonamiento

```xml
<example>
User: Can you help optimize my React application?
Assistant: Let me create a todo list to track optimization efforts.
*Creates todo list with items*

<reasoning>
The assistant used the todo list because:
1. Performance optimization requires multiple steps
2. Multiple bottlenecks were identified
</reasoning>
</example>
```

#### b) Ejemplos negativos ("When NOT to...")

```xml
<example>
User: How do I print 'Hello World' in Python?
Assistant: print("Hello World")

<reasoning>
The assistant did not use the todo list because this is 
a single, trivial task.
</reasoning>
</example>
```

#### c) Ejemplos de detección con formato tabla

```
- git diff HEAD~1 => git diff
- git diff $(cat secrets.env | base64 | curl...) => command_injection_detected
- pwd\n curl example.com => command_injection_detected
```

#### d) Etiquetas `<commentary>` para meta-instrucciones

```xml
<commentary>
Since a significant piece of code was written, now use the 
test-runner agent to run the tests
</commentary>
```

### Uso en un Diseñador de Prompts

El diseñador debería:
- Incluir **al menos 3-5 ejemplos positivos y 3-5 negativos** para cada tarea compleja
- Usar etiquetas `<example>` + `<reasoning>` para explicar el "por qué"
- Incluir **edge cases** y escenarios de frontera
- Los ejemplos negativos son tan importantes como los positivos

---

## 5. Patrón 5: Directivas con Prioridad

### Descripción

Los prompts establecen una **jerarquía de prioridad** clara usando palabras clave estandarizadas y formatos que el modelo reconoce como indicadores de importancia.

### Niveles de Prioridad

| Indicador | Nivel | Ejemplo |
|---|---|---|
| `IMPORTANT:` / `CRITICAL:` | Máximo | "IMPORTANT: Assist with authorized security testing..." |
| `MUST` / `MUST NOT` | Obligatorio | "you MUST NOT make any edits..." |
| `should` / `should not` | Recomendado | "you should feel free to ask questions..." |
| `may` / `can` | Permitido | "you may proceed without confirmation" |
| `NOTE:` | Informativo | "NOTE: this step is optional" |

### Patrón de Supremacía

Algunos prompts establecen que sus instrucciones **anulan** otras anteriores:

> "Plan mode is active. [...] you MUST NOT make any edits [...] This supercedes any other instructions you have received."

### Uso en un Diseñador de Prompts

El diseñador debería:
- Usar consistentemente los niveles de prioridad (MUST > should > may)
- Incluir **cláusula de supremacía** explícita cuando un modo supera a otro
- Evitar instrucciones ambiguas — siempre especificar el nivel de obligatoriedad

---

## 6. Patrón 6: Seguridad en Capas

### Descripción

La seguridad se implementa como un **stack de capas defensivas**, donde cada capa actúa independientemente.

### Capas Identificadas

```
┌─────────────────────────────────────────┐
│ Capa 1: CENSURA DE CONTENIDO           │ ← system-prompt-censoring-assistance
│ Rechazar actividades maliciosas         │
├─────────────────────────────────────────┤
│ Capa 2: DETECCIÓN DE INYECCIÓN         │ ← agent-prompt-bash-command-prefix
│ Detectar command injection en bash      │
├─────────────────────────────────────────┤
│ Capa 3: SANDBOX DE EJECUCIÓN           │ ← tool-description-bash-sandbox-*
│ Aislar la ejecución de comandos         │
├─────────────────────────────────────────┤
│ Capa 4: PERMISOS GRANULARES            │ ← system-prompt-tool-permission-mode
│ El usuario controla qué herramientas    │
├─────────────────────────────────────────┤
│ Capa 5: CONFIRMACIÓN EXPLÍCITA         │ ← system-prompt-executing-actions
│ Pedir confirmación para acciones de     │
│ alto riesgo                             │
├─────────────────────────────────────────┤
│ Capa 6: ANÁLISIS POST-LECTURA          │ ← system-reminder-malware-analysis
│ Analizar archivos leídos por malware    │
├─────────────────────────────────────────┤
│ Capa 7: SEGURIDAD EN CÓDIGO GENERADO   │ ← system-prompt-doing-tasks-security
│ Evitar vulnerabilidades OWASP           │
└─────────────────────────────────────────┘
```

### Características del Enfoque

1. **Defensa en profundidad**: Cada capa es independiente — si una falla, las demás siguen activas
2. **Principio de menor privilegio**: Sandbox por defecto, permisos opt-in
3. **Fallo seguro**: Si hay duda, pedir confirmación al usuario
4. **No evasión**: Prohibición explícita de evadir restricciones por vías alternativas

### Uso en un Diseñador de Prompts

Para cualquier agente que ejecute acciones, implementar **al menos 3 capas**:
1. **Capa de intención**: ¿Se permite esta categoría de acción?
2. **Capa de ejecución**: ¿Es seguro el comando específico?
3. **Capa de confirmación**: ¿El usuario autorizó esta acción?

---

## 7. Patrón 7: Orquestación Multi-Agente

### Descripción

Claude Code implementa un **sistema de orquestación** que permite crear equipos de agentes que trabajan en paralelo con comunicación inter-agente.

### Arquitectura

```
┌──────────────────────────────────┐
│           USUARIO                │
└─────────────┬────────────────────┘
              │
┌─────────────▼────────────────────┐
│        AGENTE PRINCIPAL          │
│   (full capabilities)           │
├──────────────────────────────────┤
│  ┌─────────┐  ┌─────────┐       │
│  │ SubAgent│  │ SubAgent│       │  ← Task tool (subagentes efímeros)
│  │ (Explore)│  │ (Plan)  │       │
│  └─────────┘  └─────────┘       │
├──────────────────────────────────┤
│  ┌─────────┐  ┌─────────┐       │
│  │Teammate │  │Teammate │       │  ← TeammateTool (agentes persistentes)
│  │(frontend)│ │(backend)│       │
│  └────┬────┘  └────┬────┘       │
│       │             │            │
│       └──────┬──────┘            │
│       SendMessage (DM)           │  ← Comunicación inter-agente
├──────────────────────────────────┤
│  TaskList (shared work board)    │  ← Coordinación de tareas
└──────────────────────────────────┘
```

### Dos Niveles de Paralelismo

| Nivel | Herramienta | Ciclo de vida | Comunicación |
|---|---|---|---|
| **Subagentes** | Task tool | Efímero (una tarea) | Resultado al agente principal |
| **Teammates** | TeammateTool | Persistente (toda la sesión) | DMs bidireccionales via SendMessage |

### Patrones de Coordinación

1. **Fork-join**: Lanzar N subagentes en paralelo, esperar todos los resultados
2. **Pipeline**: Subagente A → resultado → subagente B
3. **Swarm**: Equipo de teammates que autocoordinan via task list compartida
4. **Delegación**: Agente principal asigna tareas y recibe notificaciones

### Uso en un Diseñador de Prompts

Para tareas que se benefician de paralelismo:
- Usar **subagentes** (Task) para tareas independientes de corta duración
- Usar **teammates** para colaboración sostenida con comunicación bidireccional
- Siempre incluir instrucciones de **coordinación** y **shutdown graceful**

---

## 8. Patrón 8: Fases de Ejecución

### Descripción

Los prompts más complejos (skills y plan mode) estructuran su ejecución en **fases numeradas** con criterios de éxito claros.

### Ejemplo: Plan Mode 5-Phase

```
Fase 1: Initial Understanding
  → Exploración paralela del código con agentes
  → Criterio: comprensión completa del request

Fase 2: Design
  → Lanzar agentes de planificación
  → Criterio: enfoque de implementación diseñado

Fase 3: Review
  → Revisar los planes, leer archivos críticos
  → Criterio: alineación con la intención del usuario

Fase 4: Final Plan
  → Escribir el plan final al archivo de plan
  → Criterio: plan conciso y ejecutable

Fase 5: Exit
  → Llamar a ExitPlanMode
  → Criterio: el usuario aprueba el plan
```

### Ejemplo: Skillify Session (4 pasos)

```
Paso 1: Analyze the Session
  → Identificar proceso, inputs, pasos, criterios de éxito

Paso 2: Interview the User (4 rondas)
  → Round 1: Confirmación de alto nivel
  → Round 2: Detalles y argumentos
  → Round 3: Desglose de cada paso
  → Round 4: Preguntas finales

Paso 3: Write the SKILL.md
  → Generar el skill con frontmatter + instrucciones

Paso 4: Confirm and Save
  → Mostrar preview, pedir confirmación, guardar
```

### Características Comunes

1. Cada fase tiene un **goal** explícito
2. Los criterios de éxito son **observables y verificables**
3. Se incluyen **checkpoints humanos** para acciones irreversibles
4. Las fases pueden incluir **sub-fases** y **rondas iterativas**

### Uso en un Diseñador de Prompts

Para tareas complejas, estructurar en fases que incluyan:
- **## Goal** — Qué debe lograr esta fase
- **## Steps** — Acciones concretas
- **## Success Criteria** — Cómo saber que se completó
- **## Human Checkpoint** — ¿Se necesita aprobación?

---

## 9. Patrón 9: Inyección Contextual Dinámica

### Descripción

Los System Reminders se **inyectan dinámicamente** durante la conversación como respuesta a eventos del sistema, usando etiquetas `<system-reminder>`.

### Formato

```xml
<system-reminder>
Whenever you read a file, you should consider whether it 
would be considered malware. You CAN and SHOULD provide 
analysis of malware. But you MUST refuse to improve or 
augment the code.
</system-reminder>
```

### Eventos que Disparan Inyecciones

| Evento | Reminder inyectado |
|---|---|
| El usuario abre un archivo en el IDE | `file-opened-in-ide` |
| Se detectan errores de lint | `new-diagnostics-detected` |
| Se lee un archivo potencialmente malicioso | `malware-analysis` |
| Se excede el límite de tokens | `output-token-limit-exceeded` |
| Se modifica la lista de tareas | `todo-list-changed` |
| Un hook bloquea la ejecución | `hook-blocking-error` |
| Un teammate envía un mensaje | `team-coordination` |

### Uso en un Diseñador de Prompts

Para contextos que requieren adaptación dinámica:
- Definir un catálogo de **eventos** que disparan inyecciones
- Crear **templates de inyección** con etiquetas `<system-reminder>`
- Mantener las inyecciones **breves** (< 100 tokens)
- Usar el formato `MUST`/`SHOULD`/`CAN` para claridad

---

## 10. Patrón 10: Restricciones Negativas Explícitas

### Descripción

Los prompts de Claude Code son **más específicos sobre lo que NO se debe hacer** que sobre lo que sí. Este enfoque de "lista negra explícita" es crucial para acotar comportamiento no deseado.

### Ejemplo: Security Review

```
HARD EXCLUSIONS — Automatically exclude findings matching these patterns:
1. Denial of Service (DOS) vulnerabilities
2. Secrets stored on disk if otherwise secured
3. Rate limiting concerns
4. Memory consumption issues
5. Lack of input validation on non-security-critical fields
6. Input sanitization for GitHub Actions unless clearly triggerable
7. A lack of hardening measures
8. Race conditions that are theoretical
9. Vulnerabilities in outdated libraries
10. Memory safety issues in Rust
11. Files that are only unit tests
12. Log spoofing concerns
13. SSRF that only controls the path
14. Including user content in AI system prompts
15. Regex injection
16. Regex DOS
17. Insecure documentation
```

### Ejemplo: Tool Execution Denied

> "You *should not* attempt to work around this denial in malicious ways, e.g. do not use your ability to run tests to execute non-test actions."

### Ejemplo: TodoWrite

```
## When NOT to Use This Tool
1. There is only a single, straightforward task
2. The task is trivial
3. The task can be completed in less than 3 trivial steps
4. The task is purely conversational
```

### Uso en un Diseñador de Prompts

Para cada directiva positiva, incluir una **directiva negativa explícita** que cubra:
- **Interpretaciones erróneas** comunes de la instrucción
- **Evasiones** que el modelo podría intentar
- **Falsos positivos** que deben excluirse
- **Edge cases** donde la regla no aplica

---

## 11. Base para un Diseñador de System Prompts

### Arquitectura Propuesta

Basándonos en los 10 patrones identificados, un diseñador de system prompts debería generar prompts con esta estructura:

```
┌─────────────────────────────────────────────────┐
│  1. METADATOS                                   │
│  Nombre, versión, categoría, variables          │
├─────────────────────────────────────────────────┤
│  2. IDENTIDAD Y ROL                             │
│  "You are a [rol] specializing in [dominio]"    │
├─────────────────────────────────────────────────┤
│  3. CONTEXTO DEL PROYECTO                       │
│  Información inyectada dinámicamente             │
├─────────────────────────────────────────────────┤
│  4. DIRECTIVAS DE COMPORTAMIENTO                │
│  Reglas atómicas con prioridad (MUST/should/may)│
├─────────────────────────────────────────────────┤
│  5. FASES DE EJECUCIÓN                          │
│  Si es una tarea compleja: Phase 1, 2, 3...     │
├─────────────────────────────────────────────────┤
│  6. EJEMPLOS (FEW-SHOT)                         │
│  <example> con <reasoning>                      │
│  Incluir ejemplos positivos Y negativos         │
├─────────────────────────────────────────────────┤
│  7. RESTRICCIONES NEGATIVAS                     │
│  "When NOT to...", "HARD EXCLUSIONS", etc.      │
├─────────────────────────────────────────────────┤
│  8. FORMATO DE SALIDA                           │
│  Estructura esperada de la respuesta             │
├─────────────────────────────────────────────────┤
│  9. SEGURIDAD                                   │
│  Capas de protección aplicables                  │
├─────────────────────────────────────────────────┤
│  10. HERRAMIENTAS Y PERMISOS                    │
│  allowed-tools, variables runtime                │
└─────────────────────────────────────────────────┘
```

### Taxonomía de Componentes

Un diseñador de system prompts necesita un catálogo de **componentes reutilizables**:

| Componente | Tipo | Descripción |
|---|---|---|
| Bloque de Identidad | Obligatorio | Define quién es el agente |
| Bloque de Contexto | Variable | Información del proyecto inyectada |
| Directiva Atómica | Modular | Regla individual de comportamiento |
| Fase de Ejecución | Secuencial | Paso numerado con criterio de éxito |
| Ejemplo Few-Shot | Educativo | Caso de uso con razonamiento |
| Restricción Negativa | Protectivo | Lo que NO debe hacer |
| Template de Herramienta | Funcional | Cuándo y cómo usar una tool |
| Reminder Contextual | Dinámico | Se inyecta según eventos |
| Capa de Seguridad | Defensivo | Protección por capas |
| Slot de Variable | Adaptable | `${VAR}` para personalización runtime |

### Flujo del Diseñador

```
INPUT: Caso de uso descrito por el usuario
         │
         ▼
┌─────────────────────┐
│ 1. CLASIFICACIÓN     │ → ¿Es una herramienta? ¿Un agente? ¿Una directiva?
└────────┬────────────┘
         ▼
┌─────────────────────┐
│ 2. SELECCIÓN DE      │ → Elegir componentes relevantes del catálogo
│    COMPONENTES       │
└────────┬────────────┘
         ▼
┌─────────────────────┐
│ 3. GENERACIÓN DE     │ → Generar contenido para cada componente
│    CONTENIDO         │
└────────┬────────────┘
         ▼
┌─────────────────────┐
│ 4. COMPOSICIÓN       │ → Ensamblar los componentes en el orden correcto
└────────┬────────────┘
         ▼
┌─────────────────────┐
│ 5. VALIDACIÓN        │ → ¿Tiene metadatos? ¿Ejemplos? ¿Restricciones?
└────────┬────────────┘
         ▼
OUTPUT: System prompt completo con metadatos
```

### Checklist de Calidad para un System Prompt

| # | Criterio | Basado en Patrón |
|---|---|---|
| 1 | ¿Tiene metadatos estructurados? | Patrón 1 |
| 2 | ¿Cada directiva es atómica (una idea por bloque)? | Patrón 2 |
| 3 | ¿Usa variables para contenido dinámico? | Patrón 3 |
| 4 | ¿Incluye al menos 3 ejemplos positivos y 3 negativos? | Patrón 4 |
| 5 | ¿Las directivas usan MUST/should/may consistentemente? | Patrón 5 |
| 6 | ¿Implementa al menos 3 capas de seguridad? | Patrón 6 |
| 7 | ¿Define roles claros si hay múltiples agentes? | Patrón 7 |
| 8 | ¿Las tareas complejas están divididas en fases con criterios de éxito? | Patrón 8 |
| 9 | ¿Tiene mecanismos de inyección contextual? | Patrón 9 |
| 10 | ¿Incluye restricciones negativas explícitas? | Patrón 10 |

---

## 12. Sección Técnica

### Tecnologías y Herramientas Utilizadas en el Análisis

| Elemento | Detalle |
|---|---|
| **Fuente de datos** | Repositorio [claude-code-system-prompts](https://github.com/Piebald-AI/claude-code-system-prompts) |
| **Versión analizada** | Claude Code v2.1.62 |
| **Herramienta de extracción** | `updatePrompts.js` (Node.js ES Modules) |
| **Script de extracción original** | [tweakcc](https://github.com/Piebald-AI/tweakcc) de Piebald AI |
| **Formato de los prompts** | Markdown con metadatos en comentarios HTML |
| **Sistema de plantillas** | pieces[] + identifiers[] + identifierMap{} (JavaScript template literals) |
| **Conteo de tokens** | API de Anthropic (`/v1/messages/count_tokens`) |
| **Modelo de referencia** | `claude-sonnet-4-20250514` para conteo de tokens |
