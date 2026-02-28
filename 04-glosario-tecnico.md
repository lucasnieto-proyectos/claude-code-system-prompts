# Glosario de Términos Técnicos en Claude Code y Desarrollo de Software

> Guía orientada a usuarios que quieren entender el vocabulario técnico que aparece en los system prompts de Claude Code y en el desarrollo de software en general.

---

## Índice Rápido

| Término | Una frase | Ir a |
|---|---|---|
| **Hook** | Código que se ejecuta automáticamente antes o después de una acción | [→](#1-hook-gancho) |
| **Command / Comando** | Instrucción que se escribe en la terminal | [→](#2-command--comando) |
| **Lint / Linter** | Herramienta que detecta errores de estilo y calidad en el código | [→](#3-lint--linter) |
| **Workflow** | Secuencia de pasos automatizados | [→](#4-workflow-flujo-de-trabajo) |
| **Rules** | Reglas de configuración que definen comportamiento | [→](#5-rules-reglas) |
| **Sandbox** | Entorno aislado y seguro para ejecutar código | [→](#6-sandbox-caja-de-arena) |
| **Prompt** | Texto que le das a una IA como instrucción | [→](#7-prompt--system-prompt) |
| **Token** | Unidad mínima de texto que procesa una IA | [→](#8-token) |
| **API** | Interfaz para que programas hablen entre sí | [→](#9-api) |
| **SDK** | Kit de herramientas para construir con una plataforma | [→](#10-sdk) |
| **MCP** | Protocolo estándar para conectar IAs con herramientas | [→](#11-mcp-model-context-protocol) |
| **Agent / Agente** | IA autónoma que puede tomar decisiones y ejecutar acciones | [→](#12-agent--agente) |
| **Subagent** | Agente que trabaja dentro de otro agente | [→](#13-subagent--subagente) |
| **Swarm** | Grupo de agentes que trabajan en equipo | [→](#14-swarm-enjambre) |
| **Slash Command** | Comando que empieza por `/` | [→](#15-slash-command) |
| **PR (Pull Request)** | Petición para integrar cambios en un proyecto | [→](#16-pull-request-pr) |
| **Git** | Sistema de control de versiones | [→](#17-git) |
| **Commit** | Guardado de cambios con mensaje descriptivo | [→](#18-commit) |
| **Branch / Rama** | Línea paralela de desarrollo | [→](#19-branch--rama) |
| **Push / Pull** | Enviar / descargar cambios a/desde un servidor | [→](#20-push--pull) |
| **Worktree** | Copia de trabajo aislada del repositorio | [→](#21-worktree) |
| **Context Window** | Cantidad máxima de texto que una IA puede "recordar" | [→](#22-context-window-ventana-de-contexto) |
| **Compaction** | Resumir el historial para no llenar la ventana de contexto | [→](#23-compaction--compactación) |
| **Streaming** | Recibir la respuesta de la IA palabra por palabra | [→](#24-streaming) |
| **Frontmatter** | Metadatos al inicio de un archivo markdown | [→](#25-frontmatter) |
| **Skill** | Habilidad reutilizable guardada como archivo | [→](#26-skill-habilidad) |
| **OWASP** | Organización de seguridad web y su lista de vulnerabilidades | [→](#27-owasp) |
| **CI/CD** | Automatización de pruebas y despliegues | [→](#28-cicd) |
| **IDE** | Editor de código con herramientas integradas | [→](#29-ide) |
| **npm** | Gestor de paquetes de JavaScript/Node.js | [→](#30-npm) |

---

## 1. Hook (Gancho)

### ¿Qué es?
Un **hook** es un trozo de código que **se ejecuta automáticamente** antes o después de una acción. Es como poner una "alarma" que salta cuando algo ocurre.

### Analogía
Piensa en la alarma de una tienda: cada vez que alguien cruza la puerta (acción), la alarma suena (hook). Tú no necesitas activarla manualmente.

### Ejemplo cotidiano en Git
```
Quieres que SIEMPRE se pasen los tests antes de hacer un commit.

Sin hook: Tú te acuerdas de ejecutar los tests → a veces se te olvida → subes código roto
Con hook: Configuras un "pre-commit hook" → Git ejecuta los tests automáticamente antes de cada commit → si fallan, el commit se bloquea
```

### En Claude Code
Claude Code tiene hooks que se ejecutan en ciertos momentos:
- **Pre-hook**: Antes de ejecutar una herramienta (ej: antes de cada comando bash)
- **Post-hook**: Después de ejecutar una herramienta
- **Condition evaluator**: Un agente que decide si el hook debe activarse o no

### Archivos relacionados en los prompts
- `agent-prompt-agent-hook.md` — Prompt del hook de agente
- `agent-prompt-hook-condition-evaluator.md` — Evalúa si un hook debe ejecutarse
- `system-prompt-hooks-configuration.md` — Configuración de hooks

---

## 2. Command / Comando

### ¿Qué es?
Un **comando** es una instrucción de texto que le das a la **terminal** (también llamada consola, shell, o línea de comandos) para que el ordenador ejecute algo.

### Ejemplos
```bash
# Listar archivos en la carpeta actual
ls

# Crear una carpeta
mkdir mi-carpeta

# Instalar dependencias de un proyecto
npm install

# Ver el estado de git
git status

# Ejecutar un servidor de desarrollo
npm run dev
```

### ¿Qué es una Terminal?
Es la ventana negra (o de colores) donde escribes comandos. En Windows se llama **PowerShell** o **CMD**. En Mac/Linux se llama **Terminal** o **Bash**.

### En Claude Code
Claude Code tiene una herramienta llamada **Bash** que le permite ejecutar comandos en la terminal. Los prompts de seguridad se encargan de que no ejecute comandos peligrosos (como borrar todo tu disco).

---

## 3. Lint / Linter

### ¿Qué es?
Un **linter** es un programa que **revisa tu código** línea por línea buscando:
- Errores de sintaxis (como olvidar un `;` o un `}`)
- Malas prácticas (como variables sin usar)
- Inconsistencias de estilo (como mezclar comillas simples y dobles)

### Analogía
Es como el **corrector ortográfico** de Word, pero para código. No ejecuta el programa, solo lo lee y señala problemas.

### Ejemplo
```javascript
// Tu código:
var x = 5
var y = 10
console.log(x)

// El linter te dice:
// ⚠️ Línea 2: 'y' se declaró pero nunca se usa
// ⚠️ Línea 1: Usa 'const' en vez de 'var' (best practice)
```

### Linters populares
| Lenguaje | Linter |
|---|---|
| JavaScript/TypeScript | ESLint |
| Python | Pylint, Ruff, Flake8 |
| CSS | Stylelint |
| Markdown | markdownlint |

### En Claude Code
El prompt `system-reminder-new-diagnostics-detected.md` se inyecta cuando el linter detecta nuevos errores, para que Claude los corrija.

---

## 4. Workflow (Flujo de trabajo)

### ¿Qué es?
Un **workflow** es una **secuencia de pasos automatizados** que se ejecutan en orden cuando ocurre algo. Es como una receta: "cuando pase X, haz paso 1, luego paso 2, luego paso 3".

### Ejemplo: GitHub Actions Workflow
```
EVENTO: Alguien sube código al repositorio (push)
  → Paso 1: Instalar dependencias
  → Paso 2: Ejecutar los tests
  → Paso 3: Si pasan, desplegar la aplicación
  → Paso 4: Si fallan, enviar notificación al equipo
```

### ¿Dónde se definen?
Normalmente en archivos YAML dentro de `.github/workflows/`:
```yaml
name: CI
on: push
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm install
      - run: npm test
```

### En Claude Code
El archivo `data-github-actions-workflow-for-claude-mentions.md` contiene un workflow template para que Claude responda automáticamente a @menciones en GitHub.

---

## 5. Rules (Reglas)

### ¿Qué es?
Las **rules** son instrucciones de configuración que definen **cómo debe comportarse** una herramienta. Son las "leyes" que el sistema sigue.

### Tipos de reglas en Claude Code

| Tipo | Dónde se definen | Ejemplo |
|---|---|---|
| **System rules** | Los system prompts | "No ejecutar comandos destructivos sin confirmación" |
| **User rules** | Archivo `CLAUDE.md` o settings | "Responder siempre en español" |
| **Tool rules** | Cada Tool Description | "Preferir la herramienta Edit en vez de usar sed" |
| **Permission rules** | Configuración de permisos | "Bash en modo sandbox obligatorio" |

### Analogía
Las rules son como las **normas de un colegio**: están escritas en algún sitio, todo el mundo las sigue, y si alguien intenta saltárselas, el sistema lo impide.

---

## 6. Sandbox (Caja de arena)

### ¿Qué es?
Un **sandbox** es un **entorno aislado y seguro** donde se ejecuta código sin que pueda afectar al resto del sistema. Como una burbuja protectora.

### Analogía  
Imagina que dejas a un niño jugar solo. Un sandbox es como un **parque infantil vallado**: puede jugar libremente dentro, pero no puede salir a la carretera.

### ¿Qué bloquea?
- ❌ Acceder a archivos fuera de la carpeta del proyecto
- ❌ Conectarse a internet (excepto sitios permitidos)
- ❌ Modificar archivos del sistema
- ❌ Instalar programas globalmente

### En Claude Code
Todos los comandos bash se ejecutan en sandbox por defecto. Los 16 prompts `tool-description-bash-sandbox-*.md` definen las reglas del sandbox. Si un comando falla por el sandbox, Claude te explica por qué y te pide permiso para ejecutarlo fuera.

---

## 7. Prompt / System Prompt

### ¿Qué es?
Un **prompt** es el **texto de instrucciones** que se le envía a una IA para que haga algo. Es como darle un briefing con sus órdenes.

| Tipo | Quién lo escribe | Ejemplo |
|---|---|---|
| **User prompt** | Tú (el usuario) | "Crea una función que calcule el IVA" |
| **System prompt** | El desarrollador de la IA | "Eres un experto en programación. No ejecutes comandos destructivos..." |
| **Agent prompt** | El sistema, para subagentes | "Eres un ingeniero de seguridad. Revisa este código..." |

### System Prompt vs User Prompt
```
System Prompt (invisible para ti): "Eres Claude Code, un asistente de programación.
  Reglas: no borrar archivos sin permiso, usar sandbox, responder en el idioma del usuario..."

User Prompt (lo que tú escribes): "Crea una app de lista de tareas en React"
```

El system prompt es como la **personalidad y las normas** de la IA. El user prompt es lo que tú le pides.

---

## 8. Token

### ¿Qué es?
Un **token** es la **unidad mínima de texto** que procesa una IA. No es exactamente una palabra — es más como un "trozo" de texto.

### Regla aproximada
- 1 token ≈ ¾ de una palabra en inglés
- 100 tokens ≈ 75 palabras
- 1 página de texto ≈ 300-400 tokens

### Ejemplos
```
"Hello"          → 1 token
"Hello, world!"  → 4 tokens  
"Programación"   → 2-3 tokens (las palabras largas se dividen)
```

### ¿Por qué importa?
- Las IAs tienen un **límite máximo** de tokens (la context window)
- Cuantos más tokens uses, más **cuesta** económicamente
- Los system prompts de Claude Code suman miles de tokens antes de que tú escribas nada

---

## 9. API

### ¿Qué es?
**API** (Application Programming Interface) es una **interfaz estandarizada** para que dos programas se comuniquen entre sí. Es como un menú de restaurante: tú pides del menú, y la cocina te lo prepara.

### Analogía
```
TÚ (programa)  →  "Quiero el plato #3"  →  RESTAURANTE (API)  →  🍝 (respuesta)
TU APP          →  "Dame el clima de Madrid" →  API del Tiempo   →  "22°C, soleado"
```

### Ejemplo real
```python
# Tu programa le pide a la API de Claude que genere texto
response = client.messages.create(
    model="claude-sonnet-4-6",
    messages=[{"role": "user", "content": "¿Qué es una API?"}]
)
```

### En Claude Code
Los prompts `data-claude-api-reference-*.md` contienen documentación sobre cómo usar la API de Claude en diferentes lenguajes.

---

## 10. SDK

### ¿Qué es?
**SDK** (Software Development Kit) es un **kit de herramientas** que facilita usar una API. Si la API es la "cocina del restaurante", el SDK es un **robot de cocina** que te simplifica la receta.

### API vs SDK
| | API | SDK |
|---|---|---|
| Qué es | Interfaz de comunicación | Kit de herramientas |
| Cómo se usa | Peticiones HTTP manuales | Funciones y clases preparadas |
| Dificultad | Mayor | Menor |
| Ejemplo | `fetch("https://api.anthropic.com/...")` | `client.messages.create(...)` |

### En Claude Code
El **Agent SDK** es un kit que permite crear agentes AI con herramientas integradas (leer archivos, buscar en web, ejecutar terminales) sin tener que programar todo desde cero.

---

## 11. MCP (Model Context Protocol)

### ¿Qué es?
**MCP** es un **protocolo estándar** (creado por Anthropic) que permite conectar IAs con herramientas externas de forma estandarizada. Es como un "enchufe universal" para que las IAs puedan usar cualquier herramienta.

### Analogía
Sin MCP: Cada IA necesita su propio cable especial para cada herramienta (como los cargadores de móvil antiguos).
Con MCP: Todas las IAs usan el mismo tipo de enchufe (como USB-C).

### Ejemplo
```
Claude Code + MCP Server de Playwright = Claude puede controlar un navegador
Claude Code + MCP Server de Supabase = Claude puede consultar tu base de datos
Claude Code + MCP Server de Slack = Claude puede enviar mensajes por Slack
```

---

## 12. Agent / Agente

### ¿Qué es?
Un **agente** es una IA que puede **tomar decisiones y ejecutar acciones** de forma autónoma. A diferencia de un chatbot simple que solo responde preguntas, un agente puede:
- Decidir qué herramientas usar
- Ejecutar código
- Leer y escribir archivos
- Corregir errores y reintentar

### Chatbot vs Agente
| | Chatbot | Agente |
|---|---|---|
| Función | Responde preguntas | Realiza tareas complejas |
| Autonomía | Baja (solo responde) | Alta (decide y actúa) |
| Herramientas | No tiene | Lee, escribe, ejecuta, busca |
| Ejemplo | ChatGPT básico | Claude Code |

### Claude Code como agente
Claude Code es un **agente de codificación**: puede leer tu proyecto, escribir código, ejecutar tests, crear commits y pull requests — todo de forma autónoma.

---

## 13. Subagent / Subagente

### ¿Qué es?
Un **subagente** es un agente que trabaja **dentro de otro agente** (el agente principal). Es como contratar a un ayudante temporal para una tarea específica.

### Analogía
```
JEFE (agente principal): "Necesito que alguien investigue cómo funciona la autenticación en este proyecto"
  → Contrata a un INVESTIGADOR (subagente Explore)
  → El investigador investiga y le devuelve un informe
  → El jefe continúa con su trabajo usando ese informe
```

### En Claude Code
- **Task tool**: Lanza subagentes para tareas específicas (investigar, planificar, etc.)
- Los subagentes **no ven** los mensajes del usuario directamente — solo ven lo que el agente principal les pasa
- Son **efímeros**: nacen, hacen su tarea y desaparecen

---

## 14. Swarm (Enjambre)

### ¿Qué es?
Un **swarm** es un grupo de **agentes que trabajan en equipo** de forma coordinada, como un equipo de desarrolladores.

### Analogía
```
EQUIPO DE DESARROLLO:
  - Ana (frontend) trabaja en la interfaz
  - Pedro (backend) trabaja en el servidor
  - Laura (QA) ejecuta los tests
  
Todos comparten una lista de tareas y se envían mensajes cuando necesitan algo del otro.
```

### En Claude Code
La herramienta **TeammateTool** crea equipos de agentes que:
- Comparten una lista de tareas
- Se envían mensajes entre sí (via SendMessage)
- Trabajan **en paralelo** en diferentes aspectos del proyecto
- Coordinan su trabajo automáticamente

---

## 15. Slash Command

### ¿Qué es?
Un **slash command** es un comando que empieza por `/` y activa una función especial. Es un atajo rápido.

### Ejemplos en Claude Code
```
/security-review    → Lanza una auditoría de seguridad
/review-pr          → Revisa un pull request
/pr-comments        → Muestra comentarios de un PR
/init               → Inicializa un archivo CLAUDE.md
/init-verifiers     → Crea skills de verificación
/skillify           → Convierte la sesión actual en un skill reutilizable
```

---

## 16. Pull Request (PR)

### ¿Qué es?
Un **Pull Request** es una **petición formal** para integrar tus cambios en el proyecto principal. Es como decir: "He terminado mi trabajo, ¿alguien puede revisarlo antes de que lo incluyamos?".

### Flujo
```
1. Tú trabajas en tu rama (branch) → haces cambios
2. Creas un PR → "Quiero integrar estos cambios"
3. Tus compañeros revisan el código → comentan, sugieren mejoras
4. Si todo está bien → se aprueba y se "merge" (fusiona)
5. Los cambios ya están en el proyecto principal
```

---

## 17. Git

### ¿Qué es?
**Git** es un sistema de **control de versiones**. Piensa en él como una **máquina del tiempo para tu código**: guarda cada cambio que haces y te permite volver atrás si algo sale mal.

### Analogía
Es como el "Historial de versiones" de Google Docs, pero mucho más potente y para código.

---

## 18. Commit

### ¿Qué es?
Un **commit** es como hacer una **foto de tu código** en un momento dado, con una nota que explica qué cambiaste.

### Ejemplo
```bash
git commit -m "Añadir función de cálculo de IVA"
```

Esto guarda todos los cambios que hayas preparado con una nota descriptiva. Cada commit tiene un **identificador único** (hash) para poder volver a él.

---

## 19. Branch / Rama

### ¿Qué es?
Una **rama** es una **línea paralela de desarrollo**. Te permite trabajar en algo nuevo sin estropear el código principal.

### Analogía
```
main (rama principal):        A — B — C — D (código estable)
                                    \
feature/login (tu rama):             E — F (tu trabajo nuevo)
```

Trabajas en tu rama tranquilamente. Cuando terminas, la fusionas (merge) con la principal.

---

## 20. Push / Pull

### ¿Qué es?
| Acción | Dirección | Qué hace |
|---|---|---|
| **Push** | Tu PC → Servidor (GitHub) | Enviar tus commits al repositorio remoto |
| **Pull** | Servidor (GitHub) → Tu PC | Descargar los últimos cambios del servidor |

```bash
git push    # "Subir" tus cambios
git pull    # "Bajar" los cambios de otros
```

---

## 21. Worktree

### ¿Qué es?
Un **worktree** es una **copia aislada** de tu repositorio en otra carpeta. Te permite trabajar en dos ramas a la vez sin tener que cambiar de rama constantemente.

### En Claude Code
La herramienta Task puede lanzar subagentes en un worktree aislado, para que hagan cambios sin afectar tu copia de trabajo principal. Si el subagente hace algo mal, la copia se descarta.

---

## 22. Context Window (Ventana de Contexto)

### ¿Qué es?
La **context window** es la **cantidad máxima de texto** que una IA puede "recordar" en una conversación. Todo lo que le has dicho + todo lo que te ha respondido + el system prompt tiene que caber en esta ventana.

### Tamaños actuales
| Modelo | Context Window |
|---|---|
| Claude Opus 4.6 | 200.000 tokens (1M en beta) |
| Claude Sonnet 4.6 | 200.000 tokens (1M en beta) |
| Claude Haiku 4.5 | 200.000 tokens |

### ¿Qué pasa si se llena?
Cuando la ventana se llena, Claude Code usa **compactación**: resume la conversación anterior para liberar espacio.

---

## 23. Compaction / Compactación

### ¿Qué es?
La **compactación** es el proceso de **resumir** el historial de la conversación para liberar espacio en la context window. Es como hacer "resumen de lo visto" para poder seguir hablando.

### En Claude Code
El prompt `agent-prompt-conversation-summarization.md` define cómo se hace la compactación: preservando archivos modificados, decisiones tomadas, errores encontrados y la tarea actual.

---

## 24. Streaming

### ¿Qué es?
**Streaming** significa recibir la respuesta de la IA **palabra por palabra** según se va generando, en vez de esperar a que termine toda la respuesta.

### Analogía
- **Sin streaming**: Le mandas un email y esperas la respuesta completa
- **Con streaming**: Chateas en WhatsApp y ves cómo escribe en tiempo real

---

## 25. Frontmatter

### ¿Qué es?
El **frontmatter** es un bloque de **metadatos al inicio** de un archivo (normalmente markdown o YAML) que contiene información sobre el archivo.

### Ejemplo
```markdown
---
name: mi-skill
description: Automatiza la creación de PRs
allowed-tools:
  - Bash(git:*)
  - Read
---

# Aquí empieza el contenido real del archivo
```

En los system prompts de Claude Code, el frontmatter se escribe como comentario HTML:
```html
<!--
name: 'System Prompt: Security'
description: Avoid security vulnerabilities
ccVersion: 2.1.62
-->
```

---

## 26. Skill (Habilidad)

### ¿Qué es?
Un **skill** es una **habilidad reutilizable** que Claude Code puede ejecutar. Se guarda como un archivo `SKILL.md` con instrucciones paso a paso, herramientas permitidas y criterios de éxito.

### Analogía
Es como una **receta guardada**: la escribes una vez y la puedes usar siempre que necesites hacer lo mismo.

### Ejemplo
```
Skill: "test-runner"
Descripción: Ejecuta todos los tests del proyecto y reporta errores
Pasos:
  1. Detectar framework de tests
  2. Ejecutar los tests
  3. Reportar resultados
```

---

## 27. OWASP

### ¿Qué es?
**OWASP** (Open Web Application Security Project) es una organización que publica la lista de las **10 vulnerabilidades más peligrosas** en aplicaciones web, conocida como el **OWASP Top 10**.

### Top vulnerabilidades
| # | Vulnerabilidad | Explicación simple |
|---|---|---|
| 1 | **Inyección SQL** | Insertar comandos maliciosos en formularios que van a la base de datos |
| 2 | **XSS** | Insertar scripts maliciosos en páginas web que otros usuarios ven |
| 3 | **Autenticación rota** | Fallos que permiten acceder sin contraseña válida |
| 4 | **CSRF** | Engañar al navegador para que haga cosas que el usuario no quiere |

### En Claude Code
El prompt `system-prompt-doing-tasks-security.md` indica a Claude que evite introducir estas vulnerabilidades al generar código.

---

## 28. CI/CD

### ¿Qué es?
- **CI** (Continuous Integration): Cada vez que subes código, se ejecutan tests automáticamente
- **CD** (Continuous Deployment): Si los tests pasan, se despliega automáticamente

### Analogía
```
CI: Un inspector de calidad que revisa cada producto que sale de la fábrica
CD: Si el inspector da el OK, el producto se envía al cliente automáticamente
```

---

## 29. IDE

### ¿Qué es?
**IDE** (Integrated Development Environment) es un **editor de código con superpoderes**: autocompletado, detección de errores, terminal integrada, debugger, etc.

### Ejemplos
| IDE | Para qué |
|---|---|
| **VS Code** | Todo tipo de desarrollo (el más popular) |
| **WebStorm** | JavaScript/TypeScript |
| **PyCharm** | Python |
| **IntelliJ IDEA** | Java |

### En Claude Code
Claude Code se integra con tu IDE: detecta qué archivos tienes abiertos, qué líneas has seleccionado, y puede recibir errores de diagnóstico del linter de tu IDE.

---

## 30. npm

### ¿Qué es?
**npm** (Node Package Manager) es el **gestor de paquetes** más popular para JavaScript/Node.js. Es como una **tienda de apps** pero para bibliotecas de código.

### Comandos comunes
```bash
npm install          # Instalar todas las dependencias del proyecto
npm install lodash   # Instalar una biblioteca específica
npm run dev          # Ejecutar el servidor de desarrollo
npm run build        # Compilar el proyecto para producción
npm test             # Ejecutar los tests
```

### En Claude Code
Claude Code mismo se distribuye como un paquete npm (`@anthropic-ai/claude-code`). Los system prompts se extraen del código compilado de ese paquete.
