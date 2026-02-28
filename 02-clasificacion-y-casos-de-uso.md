# Clasificación y Casos de Uso de los System Prompts de Claude Code

> **Versión de Claude Code**: v2.1.62  
> **Total de prompts**: 223  
> **Fecha**: 28 de febrero de 2026

---

## Índice

1. [Visión General de la Arquitectura](#1-visión-general-de-la-arquitectura)
2. [Categoría 1: Agent Prompts](#2-categoría-1-agent-prompts-28-archivos)
3. [Categoría 2: System Prompts](#3-categoría-2-system-prompts-42-archivos)
4. [Categoría 3: Data](#4-categoría-3-data-25-archivos)
5. [Categoría 4: Skills](#5-categoría-4-skills-7-archivos)
6. [Categoría 5: System Reminders](#6-categoría-5-system-reminders-40-archivos)
7. [Categoría 6: Tool Descriptions](#7-categoría-6-tool-descriptions-80-archivos)
8. [Categoría 7: Tool Parameters](#8-categoría-7-tool-parameters-1-archivo)
9. [Guía de Invocación](#9-guía-de-invocación)

---

## 1. Visión General de la Arquitectura

Claude Code **no tiene un único system prompt monolítico**. En su lugar, utiliza un sistema modular compuesto por más de 110 fragmentos de texto que se combinan dinámicamente según:

- El entorno de ejecución (OS, shell, IDE)
- La configuración del usuario (permisos, modos)
- El contexto de la conversación (plan mode, team mode)
- Las herramientas disponibles (MCP servers, browser, etc.)

### Flujo de Composición del System Prompt

```
┌──────────────────┐    ┌─────────────────┐    ┌──────────────────┐
│  System Prompts  │ +  │ Tool Descriptions│ +  │ System Reminders │
│  (base behavior) │    │ (tool guidance)  │    │ (context alerts)  │
└────────┬─────────┘    └────────┬────────┘    └────────┬─────────┘
         │                       │                       │
         └───────────┬───────────┘                       │
                     │                                   │
              ┌──────▼──────┐                            │
              │  Prompt     │◄───────────────────────────┘
              │  Compuesto  │
              │  (runtime)  │
              └──────┬──────┘
                     │
          ┌──────────▼──────────┐
          │ Variables Inyectadas│
          │ ${TOOL_NAME}, etc.  │
          └─────────────────────┘
```

### Distribución por Categoría

| Categoría | Cantidad | Tokens totales aprox. | % del total |
|---|---|---|---|
| Tool Descriptions | 80 | ~5.500 | 35% |
| System Prompts | 42 | ~8.500 | 19% |
| System Reminders | 40 | ~4.200 | 18% |
| Agent Prompts | 28 | ~12.000 | 13% |
| Data | 25 | ~52.000 | 11% |
| Skills | 7 | ~10.500 | 3% |
| Tool Parameters | 1 | ~250 | <1% |

---

## 2. Categoría 1: Agent Prompts (28 archivos)

**Propósito**: System prompts completos para subagentes y utilidades internas que Claude Code invoca como agentes especializados.

### 2.1 Sub-agentes (4 prompts)

Agentes que Claude puede lanzar como procesos independientes para tareas específicas.

| Prompt | Tokens | Descripción | Cuándo se usa |
|---|---|---|---|
| **Explore** | 516 | Agente de exploración de código | Cuando se está en plan mode y se necesita investigar la base de código |
| **Plan mode (enhanced)** | 633 | Agente de planificación avanzada | Al diseñar una implementación compleja en plan mode |
| **Task tool** | 294 | System prompt del subagente Task | Cada vez que se lanza un subagente via `Task` tool |
| **Task tool (extra notes)** | 127 | Notas adicionales para Task | Complementa al anterior con reglas sobre rutas absolutas y formato |

**Caso de uso**: Se invocan automáticamente cuando Claude determina que una tarea es suficientemente compleja para delegar a un agente especializado, o cuando el usuario entra en "plan mode".

### 2.2 Asistentes de Creación (3 prompts)

Agentes que ayudan a crear artefactos de configuración.

| Prompt | Tokens | Descripción | Cuándo se usa |
|---|---|---|---|
| **Agent creation architect** | 1.110 | Diseña agentes personalizados | Cuando el usuario pide crear un agente personalizado |
| **CLAUDE.md creation** | 384 | Genera archivos CLAUDE.md | Al inicializar documentación del proyecto |
| **Status line setup** | 1.502 | Configura la línea de estado | Al configurar la barra de estado del terminal |

**Caso de uso**: Se invocan con los slash commands `/create-agent`, `/init`, `/setup-statusline` respectivamente.

### 2.3 Slash Commands (3 prompts)

Agentes activados por comandos `/` del usuario.

| Prompt | Tokens | Descripción | Cuándo se usa |
|---|---|---|---|
| **/pr-comments** | 396 | Obtiene comentarios de PR en GitHub | `/pr-comments` — Para revisar feedback de un pull request |
| **/review-pr** | 211 | Revisión de código de PRs | `/review-pr` — Para análisis de calidad de código |
| **/security-review** | 2.610 | Revisión de seguridad exhaustiva | `/security-review` — Auditoría de seguridad con categorías OWASP |

**Caso de uso**: Invocación explícita del usuario via comandos slash. Son los prompts más estructurados con metodologías faseadas y formatos de salida estrictos.

### 2.4 Utilidades (18 prompts)

Funciones AI-powered que Claude usa internamente.

| Prompt | Tokens | Descripción | Cuándo se usa |
|---|---|---|---|
| **Agent Hook** | 133 | Prompt para hooks de agente | Al ejecutar hooks configurados |
| **Bash command description writer** | 207 | Describe comandos bash | Internamente, al generar descripciones de comandos |
| **Bash command file path extraction** | 286 | Extrae rutas de archivos | Tras ejecutar comandos bash, para indexar archivos afectados |
| **Bash command prefix detection** | 823 | Detecta prefijos e inyección | Antes de ejecutar cualquier comando bash |
| **Claude guide agent** | 761 | Guía sobre Claude Code/API/SDK | Cuando el usuario pregunta cómo usar Claude |
| **Conversation summarization** | 1.121 | Resumen detallado de conversaciones | Al compactar contexto (context window llena) |
| **Hook condition evaluator** | 78 | Evalúa condiciones de hooks | Al determinar si un hook debe ejecutarse |
| **Memory selection** | 156 | Selecciona memorias relevantes | Al cargar memorias de sesión previas |
| **Prompt Suggestion Generator v2** | 296 | Genera sugerencias de prompts | Al proponer próximos pasos al usuario |
| **Quick git commit** | 507 | Commit rápido con contexto | Al ejecutar commits automatizados |
| **Quick PR creation** | 945 | Creación de PR con contexto | Al crear pull requests |
| **Recent Message Summarization** | 720 | Resume mensajes recientes | Para mantener contexto optimizado |
| **Session Search Assistant** | 439 | Busca sesiones anteriores | Al buscar en historial de sesiones |
| **Session memory update** | 756 | Actualiza memoria de sesión | Al guardar estado de la sesión actual |
| **Session title and branch generation** | 307 | Genera títulos y ramas | Al inicio de cada sesión |
| **Update Magic Docs** | 718 | Actualiza documentación automática | Al detectar cambios en documentación |
| **User sentiment analysis** | 205 | Analiza frustración del usuario | Internamente, para adaptar el tono de respuesta |
| **WebFetch summarizer** | 189 | Resume contenido web descargado | Tras usar WebFetch, para reducir tokens |

**Caso de uso**: Son funciones internas automatizadas. El usuario no las invoca directamente — se activan por eventos del sistema (ejecución de comandos, compactación, inicio de sesión, etc.).

---

## 3. Categoría 2: System Prompts (42 archivos)

**Propósito**: Fragmentos que se inyectan en el system prompt principal de Claude Code según el contexto. Son las "reglas de comportamiento" del agente.

### 3.1 Reglas de Ejecución de Tareas (13 prompts "Doing tasks")

| Prompt | Tokens | Directiva principal |
|---|---|---|
| **Ambitious tasks** | 47 | Permitir tareas ambiciosas, respetar el juicio del usuario |
| **Avoid over-engineering** | 30 | Solo hacer cambios directamente solicitados |
| **Blocked approach** | 90 | Considerar alternativas cuando se está bloqueado |
| **Help and feedback** | 24 | Informar sobre canales de ayuda |
| **Minimize file creation** | 47 | Preferir editar archivos existentes en vez de crear nuevos |
| **No compatibility hacks** | 52 | Eliminar código no usado, no añadir shims |
| **No premature abstractions** | 60 | No crear abstracciones para casos hipotéticos |
| **No time estimates** | 47 | No dar estimaciones de tiempo |
| **No unnecessary additions** | 78 | No añadir features, refactorizar ni mejorar más allá de lo pedido |
| **No unnecessary error handling** | 64 | No manejar errores imposibles |
| **Read before modifying** | 46 | Leer y entender el código antes de modificar |
| **Security** | 67 | Evitar vulnerabilidades OWASP top 10 |
| **Software engineering focus** | 104 | Interpretar instrucciones en contexto de ingeniería de software |

**Cuándo se activan**: Siempre activos en el system prompt principal. Son las reglas base de comportamiento.

### 3.2 Seguridad y Permisos (5 prompts)

| Prompt | Tokens | Función |
|---|---|---|
| **Executing actions with care** | 541 | Clasificación de riesgo de acciones y cuándo pedir confirmación |
| **Censoring assistance** | 98 | Política de uso dual para seguridad ofensiva/defensiva |
| **Tool permission mode** | 155 | Gestión de permisos de herramientas seleccionados por el usuario |
| **Tool execution denied** | 144 | Comportamiento cuando se deniega una herramienta |
| **Git status** | 97 | Mostrar estado de git al inicio de la conversación |

### 3.3 Tono y Estilo (3 prompts)

| Prompt | Tokens | Directiva |
|---|---|---|
| **Code references** | 39 | Incluir file_path:line_number al referenciar código |
| **Concise output — detailed** | 89 | Salida concisa y pulida sin monólogo interior |
| **Concise output — short** | 16 | Respuestas cortas y concisas |

**Cuándo se activan**: Según la configuración de verbosidad del usuario.

### 3.4 Uso de Herramientas (11 prompts "Tool usage")

| Prompt | Tokens | Directiva |
|---|---|---|
| **Create files** | 30 | Usar Write tool en vez de echo/cat |
| **Delegate exploration** | 114 | Usar Task para exploración amplia del código |
| **Direct search** | 52 | Usar Glob/Grep directamente para búsquedas simples |
| **Edit files** | 26 | Usar Edit tool en vez de sed/awk |
| **Read files** | 29 | Usar Read tool en vez de cat/head/tail |
| **Reserve Bash** | 75 | Bash solo para comandos del sistema, no para manipulación de archivos |
| **Search content** | 30 | Usar Grep tool en vez de grep/rg |
| **Search files** | 26 | Usar Glob tool en vez de find/ls |
| **Skill invocation** | 102 | Los slash commands invocan skills via Skill tool |
| **Subagent guidance** | 103 | Cuándo y cómo usar subagentes |
| **Task management** | 73 | Usar TodoWrite para desglosar y rastrear progreso |

### 3.5 Otros System Prompts (10 prompts)

| Prompt | Tokens | Función |
|---|---|---|
| **Agent memory instructions** | 337 | Instrucciones de actualización de memoria |
| **Agent Summary Generation** | 178 | Generación de resúmenes de agente |
| **Context compaction summary** | 278 | Compactación de contexto para el SDK |
| **Hooks Configuration** | 1.461 | Configuración de hooks de Claude Code |
| **Learning mode** | 1.042 | Modo de aprendizaje con instrucciones de colaboración humana |
| **Option previewer** | 129 | Previsualización de opciones en layout side-by-side |
| **Scratchpad directory** | 170 | Directorio temporal para archivos de trabajo |
| **Skillify Current Session** | 1.882 | Convertir la sesión actual en un skill reutilizable |
| **Teammate Communication** | 127 | Comunicación en modo swarm entre agentes |
| **Parallel tool call note** | 102 | Instrucción de usar tool calls en paralelo |

### 3.6 Insights y Análisis (6 prompts)

| Prompt | Tokens | Función |
|---|---|---|
| **Insights at a glance** | 569 | Resumen de 4 partes para informe de insights |
| **Insights friction analysis** | 139 | Análisis de patrones de fricción |
| **Insights on the horizon** | 148 | Identificación de flujos de trabajo futuros |
| **Insights session facets** | 310 | Extracción de facetas de cada sesión |
| **Insights suggestions** | 748 | Generación de sugerencias accionables |
| **Learning mode (insights)** | 142 | Insights educativos cuando learning mode está activo |

### 3.7 Herramientas de Navegador (2 prompts)

| Prompt | Tokens | Función |
|---|---|---|
| **Chrome browser MCP tools** | 156 | Instrucciones para cargar herramientas MCP de Chrome |
| **Claude in Chrome browser automation** | 759 | Uso de automatización de navegador Chrome |

### 3.8 Otros (2 prompts)

| Prompt | Tokens | Función |
|---|---|---|
| **Tool Use Summary Generation** | 171 | Generación de resúmenes de uso de herramientas |
| **Parallel tool call note** | 102 | Nota sobre tool calls en paralelo |

---

## 4. Categoría 3: Data (25 archivos)

**Propósito**: Documentación técnica de referencia embebida en Claude Code. Son como "libros de consulta" que el agente utiliza cuando necesita información sobre APIs, SDKs y modelos.

### 4.1 Referencias de API por Lenguaje (7 prompts)

| Prompt | Tokens | Lenguaje |
|---|---|---|
| **Claude API reference — Python** | 3.248 | Python |
| **Claude API reference — TypeScript** | 2.388 | TypeScript |
| **Claude API reference — Java** | 1.226 | Java |
| **Claude API reference — Go** | 621 | Go |
| **Claude API reference — Ruby** | 622 | Ruby |
| **Claude API reference — C#** | 550 | C# |
| **Claude API reference — PHP** | 394 | PHP |

**Cuándo se usan**: Cuando el usuario pide ayuda para construir con la API de Claude en un lenguaje específico.

### 4.2 Agent SDK (4 prompts)

| Prompt | Tokens | Contenido |
|---|---|---|
| **Agent SDK reference — Python** | 2.750 | Instalación, quick start, tools, hooks |
| **Agent SDK reference — TypeScript** | 2.287 | Instalación, quick start, tools, hooks |
| **Agent SDK patterns — Python** | 2.350 | Patrones: tools, hooks, subagentes, MCP |
| **Agent SDK patterns — TypeScript** | 1.069 | Patrones: agentes básicos, hooks, MCP |

### 4.3 Tool Use y Streaming (6 prompts)

| Prompt | Tokens | Contenido |
|---|---|---|
| **Tool use concepts** | 3.640 | Fundamentos conceptuales de tool use |
| **Tool use reference — Python** | 4.180 | Tool runner, loop agéntico manual |
| **Tool use reference — TypeScript** | 3.228 | Tool runner, ejecución de código |
| **Streaming reference — Python** | 1.534 | Streaming sync/async |
| **Streaming reference — TypeScript** | 1.553 | Streaming básico |
| **Message Batches API — Python** | 1.505 | Procesamiento por lotes al 50% del coste |

### 4.4 Otros Data (8 prompts)

| Prompt | Tokens | Contenido |
|---|---|---|
| **Claude model catalog** | 1.510 | Catálogo de modelos con IDs, context windows, precios |
| **Files API — Python** | 1.303 | API de archivos |
| **Files API — TypeScript** | 798 | API de archivos |
| **GitHub Actions workflow** | 527 | Workflow template para @claude mentions |
| **GitHub App installation** | 424 | Template de PR para instalación del GitHub App |
| **HTTP error codes** | 1.387 | Referencia de errores HTTP de la API |
| **Live documentation sources** | 2.337 | URLs de WebFetch para docs actualizados |
| **Session memory template** | 292 | Estructura de archivos `summary.md` de sesión |

---

## 5. Categoría 4: Skills (7 archivos)

**Propósito**: Habilidades especializadas que Claude Code puede ejecutar. Son los prompts más largos y complejos, con instrucciones multi-fase detalladas.

| Prompt | Tokens | Descripción | Invocación |
|---|---|---|---|
| **Build with Claude API** | 17.356 | Guía completa para construir apps con la API de Claude | Cuando el usuario trabaja con la API de Claude |
| **Build with Claude API (reference guide)** | 1.513 | Guía de referencia compacta | Complementa el skill principal |
| **Build with Claude API (trigger)** | 811 | Condiciones de activación | Define cuándo activar el skill |
| **Create verifier skills** | 9.771 | Crea skills de verificación automática | `/init-verifiers` |
| **Debugging** | 1.417 | Asistente de debugging | Al depurar problemas |
| **Update Claude Code config** | 4.513 | Actualizar configuración de Claude Code | Al modificar configuración |
| **Verification specialist** | 9.143 | Agente especializado en verificación | Tras completar implementaciones |

**Caso de uso**: Son los "modos expertos" de Claude Code. Se activan por slash commands o automáticamente cuando el contexto lo requiere. Son los prompts con mayor nivel de detalle y estructura faseada.

---

## 6. Categoría 5: System Reminders (40 archivos)

**Propósito**: Mensajes contextuales inyectados durante la conversación como respuesta a eventos del sistema. Son notificaciones y recordatorios en tiempo real.

### 6.1 Recordatorios de Plan Mode (5 prompts)

| Prompt | Tokens | Cuándo se inyecta |
|---|---|---|
| **Plan mode — 5-phase** | 1.511 | Al activar plan mode con exploración paralela y multi-agente |
| **Plan mode — iterative** | 797 | Al activar plan mode iterativo con entrevista al usuario |
| **Plan mode — subagent** | 307 | Al activar plan mode en un subagente |
| **Plan mode re-entry** | 236 | Al re-entrar en plan mode tras haberlo dejado |
| **Exited plan mode** | 73 | Al salir de plan mode |

### 6.2 Recordatorios de Archivos (6 prompts)

| Prompt | Tokens | Cuándo se inyecta |
|---|---|---|
| **File exists but empty** | 27 | Al leer un archivo vacío |
| **File modified by user/linter** | 97 | Al detectar modificación externa |
| **File opened in IDE** | 37 | Al detectar que el usuario abrió un archivo |
| **File shorter than offset** | 59 | Al intentar leer más allá del final |
| **File truncated** | 74 | Al truncar un archivo por tamaño |
| **Compact file reference** | 57 | Referencia a archivo leído antes de compactación |

### 6.3 Recordatorios de Hooks (5 prompts)

| Prompt | Tokens | Cuándo se inyecta |
|---|---|---|
| **Hook additional context** | 35 | Contexto adicional de un hook |
| **Hook blocking error** | 52 | Error de un hook bloqueante |
| **Hook stopped continuation prefix** | 12 | Prefijo de hook que detuvo la ejecución |
| **Hook stopped continuation** | 30 | Detalle del hook que detuvo la ejecución |
| **Hook success** | 29 | Éxito de un hook |

### 6.4 Recordatorios de Equipo (2 prompts)

| Prompt | Tokens | Cuándo se inyecta |
|---|---|---|
| **Team Coordination** | 247 | Durante coordinación de equipo swarm |
| **Team Shutdown** | 136 | Al apagar un equipo de agentes |

### 6.5 Recordatorios de Tareas (5 prompts)

| Prompt | Tokens | Cuándo se inyecta |
|---|---|---|
| **Task status** | 18 | Estado de una tarea |
| **Task tools reminder** | 123 | Recordatorio de usar herramientas de tareas |
| **Todo list changed** | 61 | Al cambiar la lista de tareas |
| **Todo list empty** | 83 | Cuando la lista está vacía |
| **TodoWrite reminder** | 98 | Recordatorio de usar TodoWrite |

### 6.6 Otros Reminders (17 prompts)

| Prompt | Tokens | Cuándo se inyecta |
|---|---|---|
| **Agent mention** | 45 | Al detectar @mención de un agente |
| **/btw side question** | 172 | Al usar /btw para preguntas laterales |
| **Malware analysis** | 87 | Tras leer archivos que podrían ser malware |
| **Memory file contents** | 38 | Al cargar contenidos de memoria |
| **Nested memory contents** | 33 | Al cargar memorias anidadas |
| **New diagnostics detected** | 35 | Al detectar errores de diagnóstico (lint, etc.) |
| **Output style active** | 32 | Al activar un estilo de salida |
| **Output token limit exceeded** | 35 | Al exceder el límite de tokens de salida |
| **Plan file reference** | 62 | Al referenciar un archivo de plan |
| **Session continuation** | 37 | Al continuar sesión desde otra máquina |
| **Token usage** | 39 | Estadísticas de uso de tokens |
| **USD budget** | 42 | Estadísticas de presupuesto en USD |
| **Verify plan reminder** | 47 | Recordatorio de verificar el plan |
| **MCP resource no content** | 41 | Cuando un recurso MCP no tiene contenido |
| **MCP resource no displayable content** | 43 | Cuando no es visualizable |
| **Invoked skills** | 33 | Lista de skills invocados |
| **Lines selected in IDE** | 66 | Al seleccionar líneas en el IDE |

---

## 7. Categoría 6: Tool Descriptions (80 archivos)

**Propósito**: Descripciones de las herramientas integradas (built-in tools) disponibles para Claude Code. Cada descripción incluye cuándo usar la herramienta y directivas de uso.

### 7.1 Herramientas Principales (18 prompts)

| Herramienta | Tokens | Función |
|---|---|---|
| **TodoWrite** | 2.167 | Crear y gestionar listas de tareas |
| **Task** | 1.331 | Lanzar subagentes especializados |
| **TeammateTool** | 1.642 | Gestionar equipos y coordinar agentes swarm |
| **SendMessageTool** | 1.241 | Enviar mensajes entre agentes del equipo |
| **EnterPlanMode** | 878 | Entrar en modo de planificación |
| **ExitPlanMode** | 417 | Salir del modo de planificación |
| **Edit** | 246 | Reemplazar texto exacto en archivos |
| **Write** | 129 | Escribir archivos al filesystem |
| **ReadFile** | 469 | Leer archivos |
| **Glob** | 122 | Búsqueda de archivos por patrón |
| **Grep** | 300 | Búsqueda de contenido con ripgrep |
| **AskUserQuestion** | 287 | Preguntar al usuario |
| **WebFetch** | 297 | Descargar contenido web |
| **WebSearch** | 319 | Búsqueda web |
| **Skill** | 326 | Ejecutar skills |
| **Computer** | 161 | Automatización de navegador Chrome |
| **NotebookEdit** | 121 | Editar celdas de Jupyter |
| **Sleep** | 154 | Esperar/pausar ejecución |

### 7.2 Herramientas de Equipo (4 prompts)

| Herramienta | Tokens | Función |
|---|---|---|
| **TaskCreate** | 558 | Crear tareas para el equipo |
| **TaskList (teammate workflow)** | 133 | Flujo de trabajo de tareas para equipos |
| **TeamDelete** | 154 | Eliminar un equipo |
| **EnterWorktree** | 334 | Trabajar en un worktree git aislado |

### 7.3 Herramientas de Búsqueda (3 prompts)

| Herramienta | Tokens | Función |
|---|---|---|
| **ToolSearch** | 144 | Buscar herramientas deferred |
| **ToolSearch extended** | 690 | Instrucciones extendidas de búsqueda |
| **LSP** | 255 | Language Server Protocol |

### 7.4 Notas Adicionales de Bash (55 prompts)

La herramienta Bash tiene la descripción más extensa, dividida en 55 fragmentos modulares que cubren:

| Subcategoría | Prompts | Temas |
|---|---|---|
| **Overview y basics** | 5 | Descripción, newlines, timeout, directorio de trabajo, directorio padre |
| **Comandos** | 4 | Secuenciales (&&), paralelos, semicolons, sleep |
| **Alternativas a Bash** | 6 | Usar herramientas dedicadas en vez de cat/grep/find/echo/sed/awk |
| **Git** | 4 | Evitar operaciones destructivas, no skip hooks, preferir nuevos commits, PRs |
| **Sleep** | 6 | Mantener cortos, sin polling, sin retry loops, uso inmediato |
| **Sandbox** | 16 | Modo default, mandatory, evidencias, restricciones, tmpdir |
| **Commit y PR** | 1 | Instrucciones detalladas (1.558 tokens) para crear commits y PRs |

---

## 8. Categoría 7: Tool Parameters (1 archivo)

| Prompt | Tokens | Función |
|---|---|---|
| **computer-action** | ~250 | Parámetros del tool Computer para acciones del navegador |

---

## 9. Guía de Invocación

### ¿Cuándo se activa cada tipo de prompt?

```
INICIO DE SESIÓN
├── System Prompts (todos los "doing tasks", seguridad, tono)
├── Tool Descriptions (todos los disponibles)
├── System Reminder: Git status
└── System Reminder: Memory file contents

DURANTE LA CONVERSACIÓN
├── System Reminders (inyección contextual)
│   ├── File opened in IDE → cuando el usuario abre un archivo
│   ├── New diagnostics → cuando hay errores de lint
│   ├── Token usage → cuando se acerca al límite
│   └── ...
└── Agent Prompts (bajo demanda)
    ├── Conversation summarization → al compactar
    ├── User sentiment analysis → al detectar frustración
    └── ...

SLASH COMMANDS
├── /security-review → Agent: Security Review
├── /review-pr → Agent: Review PR
├── /pr-comments → Agent: PR Comments
├── /init-verifiers → Skill: Create Verifier Skills
└── /skillify → System Prompt: Skillify Current Session

PLAN MODE
├── System Reminder: Plan mode is active (5-phase/iterative)
├── Agent Prompt: Explore
└── Agent Prompt: Plan mode (enhanced)

TEAM/SWARM MODE
├── Tool: TeammateTool
├── Tool: SendMessageTool
├── System Prompt: Teammate Communication
└── System Reminders: Team Coordination/Shutdown

AL EJECUTAR CÓDIGO
├── Agent: Bash command prefix detection
├── Tool: Bash (sandbox rules)
└── System Reminder: Malware analysis (si se lee código sospechoso)
```

### Tabla de Referencia Rápida por Necesidad

| Si necesitas... | Mira estos prompts |
|---|---|
| Entender cómo Claude planifica | `plan-mode-is-active-5-phase`, `agent-prompt-explore`, `agent-prompt-plan-mode-enhanced` |
| Entender el sistema de seguridad | `bash-command-prefix-detection`, `bash-sandbox-*`, `executing-actions-with-care`, `censoring-assistance` |
| Entender la orquestación multi-agente | `tool-description-task`, `tool-description-teammatetool`, `tool-description-sendmessagetool` |
| Diseñar un agente personalizado | `agent-prompt-agent-creation-architect`, `system-prompt-skillify-current-session` |
| Construir con la API de Claude | `skill-build-with-claude-api`, `data-claude-api-reference-*` |
| Gestión de tareas | `tool-description-todowrite`, `system-reminder-todo-*` |
| Compactación de contexto | `agent-prompt-conversation-summarization`, `system-prompt-context-compaction-summary` |
