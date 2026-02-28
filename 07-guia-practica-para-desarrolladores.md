# Guía Práctica: Cómo Usar los System Prompts y Plugins en tu Trabajo Diario

> **Audiencia**: Desarrolladores de software que quieren aprovechar al máximo los system prompts de Claude Code, los plugins y los patrones de diseño documentados en este repositorio.

---

## Índice

1. [¿Qué tienes a tu disposición?](#1-qué-tienes-a-tu-disposición)
2. [Usar los Plugins existentes](#2-usar-los-plugins-existentes)
3. [Mejorar tu CLAUDE.md con los patrones](#3-mejorar-tu-claudemd-con-los-patrones)
4. [Diseñar tus propios System Prompts](#4-diseñar-tus-propios-system-prompts)
5. [Crear tus propios Plugins](#5-crear-tus-propios-plugins)
6. [Crear Workflows reutilizables](#6-crear-workflows-reutilizables)
7. [Integrar herramientas con MCP](#7-integrar-herramientas-con-mcp)
8. [Aplicar los patrones a otros agentes IA](#8-aplicar-los-patrones-a-otros-agentes-ia)
9. [FAQ](#9-faq)
10. [Sección Técnica](#10-sección-técnica)
11. [Seguridad](#11-seguridad)

---

## 1. ¿Qué tienes a tu disposición?

Este repositorio contiene tres categorías de recursos:

| Recurso | Qué es | Dónde está | Para qué sirve |
|---------|--------|------------|-----------------|
| **System Prompts** | 223 prompts extraídos de Claude Code v2.1.62 | `system-prompts/` | Entender cómo Anthropic diseña su agente de producción |
| **Plugins** | 19 plugins oficiales del marketplace | `plugins/` | Instalarlos o usarlos como plantilla para crear los tuyos |
| **Documentos de análisis** | Análisis de seguridad, patrones, glosario, storytelling | `01-*.md` a `06-*.md` | Aprender los patrones y aplicarlos a tus proyectos |

### Mapa rápido de documentos

```
01-analisis-seguridad.md          → ¿Son seguros los prompts?
02-clasificacion-y-casos-de-uso.md → ¿Qué hace cada prompt y cuándo se activa?
03-patrones-de-diseno.md          → ¿Qué patrones usan y cómo replicarlos?
04-glosario-tecnico.md            → ¿Qué significan los términos técnicos?
05-storytelling-como-funciona.md  → ¿Cómo funciona Claude Code por dentro?
06-analisis-plugins.md            → ¿Qué plugins existen y cómo funcionan?
07-guia-practica (este doc)       → ¿Cómo uso todo esto en mi trabajo?
```

---

## 2. Usar los Plugins existentes

### 2.1 Plugins más útiles para desarrolladores

#### Plugin `engineering` — Tu copiloto de ingeniería

Es el plugin más relevante. Incluye **6 skills automáticas** y **6 slash commands**:

| Skill (se activa automáticamente) | ¿Cuándo se activa? |
|-----------------------------------|---------------------|
| `code-review` | Cuando pides que revise código |
| `documentation` | Cuando pides documentación |
| `incident-response` | Cuando describes un incidente |
| `system-design` | Cuando discutes arquitectura |
| `tech-debt` | Cuando mencionas deuda técnica |
| `testing-strategy` | Cuando hablas de tests |

| Command (lo invocas tú) | Qué hace |
|--------------------------|----------|
| `/engineering:review` | Revisión de código estructurada |
| `/engineering:architecture` | Diseño de arquitectura |
| `/engineering:debug` | Depuración guiada paso a paso |
| `/engineering:deploy-checklist` | Checklist de despliegue |
| `/engineering:incident` | Gestión de incidentes |
| `/engineering:standup` | Genera resumen de standup |

**Conectores disponibles**: GitHub, GitLab, Linear, Jira, Datadog, PagerDuty, Slack, Notion, Confluence.

#### Plugin `productivity` — Gestión de tareas

| Command | Qué hace |
|---------|----------|
| `/productivity:start` | Inicia una sesión de trabajo organizada |
| `/productivity:update` | Actualiza progreso y planifica próximos pasos |

**Conectores**: Slack, Notion, Asana, Linear, Jira, Google Calendar, Gmail, Microsoft 365.

#### Plugin `data` — Análisis de datos

| Command | Qué hace |
|---------|----------|
| `/data:write-query` | Genera SQL optimizado |
| `/data:explore-data` | Exploración estadística de datos |
| `/data:build-dashboard` | Crea dashboards interactivos |
| `/data:validate` | Valida calidad de datos |

### 2.2 Cómo instalar plugins

#### En Claude Code (terminal)

```bash
# Añadir el marketplace de plugins
claude plugin marketplace add anthropics/knowledge-work-plugins

# Instalar un plugin específico
claude plugin install engineering@knowledge-work-plugins
claude plugin install productivity@knowledge-work-plugins
claude plugin install data@knowledge-work-plugins
```

#### En Claude Cowork (web)

Navega a [claude.com/plugins](https://claude.com/plugins) e instala desde la interfaz gráfica.

### 2.3 Cómo usar los plugins una vez instalados

**Skills**: No necesitas hacer nada. Se activan automáticamente cuando el contexto es relevante. Si hablas de "revisar código", la skill `code-review` del plugin `engineering` se activa sola.

**Commands**: Invócalos explícitamente:

```
/engineering:review       ← Revisa el código actual
/engineering:debug        ← Inicia depuración guiada
/data:write-query         ← Genera una query SQL
```

**Sin conectores (modo standalone)**: Los plugins funcionan perfectamente sin conectar herramientas externas. Tú proporcionas el contexto (pegar código, describir situación, subir archivos).

**Con conectores (modo supercharged)**: Si conectas herramientas vía MCP (ver sección 7), el agente accede directamente a tus datos de Jira, GitHub, Slack, etc.

---

## 3. Mejorar tu CLAUDE.md con los patrones

El archivo `CLAUDE.md` es la forma más directa de personalizar cómo trabaja Claude Code en tu proyecto. Usa los patrones del documento [03-patrones-de-diseno.md](./03-patrones-de-diseno.md) para crearlo profesionalmente.

### 3.1 Plantilla profesional de CLAUDE.md

```markdown
# CLAUDE.md

## Proyecto
[Nombre del proyecto] — [Descripción de una línea]
Stack: [lenguajes, frameworks, bases de datos]

## Reglas obligatorias (MUST)
- MUST usar TypeScript estricto (`strict: true`)
- MUST validar inputs con Zod en los endpoints de la API
- MUST escribir tests para toda lógica de negocio nueva
- MUST NOT subir credenciales ni tokens al repositorio

## Reglas recomendadas (SHOULD)
- SHOULD seguir la estructura de carpetas existente
- SHOULD usar los componentes de `src/components/ui/`
- SHOULD documentar funciones públicas con JSDoc

## Estructura del proyecto
src/
├── components/   # Componentes React reutilizables
├── services/     # Lógica de negocio y acceso a datos
├── hooks/        # Custom hooks
├── types/        # Tipos TypeScript compartidos
└── utils/        # Funciones utilitarias

## Herramientas y servicios
- Base de datos: Supabase (usar MCP para consultar tablas)
- Despliegue: Coolify en VPS Hetzner
- CI/CD: GitHub Actions

## Patrones del proyecto
- Servicios con patrón Repository
- Componentes React con barrel exports (`index.ts`)
- Estado global con Zustand
- Formularios con React Hook Form + Zod

## Convenciones de código
- Nombres de archivos: kebab-case
- Nombres de componentes: PascalCase
- Nombres de funciones: camelCase
- Commits: Conventional Commits (feat:, fix:, docs:, etc.)
```

### 3.2 Patrones aplicados

| Patrón | Cómo se aplica en CLAUDE.md |
|--------|------------------------------|
| **Directivas con Prioridad** (Patrón 5) | Usar MUST / SHOULD / MAY para clasificar las reglas |
| **Restricciones Negativas** (Patrón 10) | Incluir secciones "MUST NOT" explícitas |
| **Modularidad Atómica** (Patrón 2) | Cada regla es una línea autocontenida |

---

## 4. Diseñar tus propios System Prompts

Cuando necesites crear instrucciones para un agente IA (chatbot, asistente, automatización), usa esta **plantilla basada en los 10 patrones**:

### 4.1 Plantilla de System Prompt

```markdown
<!--
name: 'Agent Prompt: [Nombre descriptivo]'
description: [Descripción de una línea]
version: 1.0.0
variables:
  - VARIABLE_1
  - VARIABLE_2
-->

## Identidad
Eres un [rol] especializado en [dominio]. Tu objetivo principal es [objetivo].

## Directivas de comportamiento

### Obligatorias (MUST)
- MUST [regla crítica 1]
- MUST [regla crítica 2]
- MUST NOT [prohibición explícita]

### Recomendadas (SHOULD)
- SHOULD [buena práctica 1]
- SHOULD [buena práctica 2]

## Fases de ejecución (para tareas complejas)

### Fase 1: [Nombre]
**Goal**: [Qué debe lograr esta fase]
**Steps**: [Pasos concretos]
**Success criteria**: [Cómo saber que terminó]

### Fase 2: [Nombre]
...

## Ejemplos

<example>
User: [Pregunta ejemplo]
Assistant: [Respuesta modelo]

<reasoning>
[Explicación de por qué esta es la respuesta correcta]
</reasoning>
</example>

<example>
User: [Pregunta que NO debe responder]
Assistant: [Cómo rechazar o redirigir]

<reasoning>
[Explicación de por qué no se debe responder]
</reasoning>
</example>

## Restricciones negativas

### HARD EXCLUSIONS — NO hacer bajo ninguna circunstancia:
1. [Cosa que no debe hacer 1]
2. [Cosa que no debe hacer 2]

## Formato de salida
[Cómo debe estructurar sus respuestas]

## Seguridad
- Capa 1: [Verificación de intención]
- Capa 2: [Validación de datos]
- Capa 3: [Confirmación del usuario si acción irreversible]
```

### 4.2 Checklist de calidad

Antes de dar por terminado un system prompt, verifica:

- [ ] ¿Tiene metadatos (nombre, versión, variables)?
- [ ] ¿Cada directiva es atómica (una idea por línea)?
- [ ] ¿Usa variables `${VAR}` para contenido dinámico?
- [ ] ¿Incluye al menos 3 ejemplos positivos y 3 negativos?
- [ ] ¿Las directivas usan MUST/SHOULD/MAY consistentemente?
- [ ] ¿Implementa al menos 3 capas de seguridad?
- [ ] ¿Las tareas complejas están divididas en fases?
- [ ] ¿Incluye restricciones negativas explícitas?

---

## 5. Crear tus propios Plugins

### 5.1 Estructura mínima de un plugin

```
mi-plugin/
├── .claude-plugin/
│   └── plugin.json          # Manifiesto del plugin
├── .mcp.json                # Conexiones MCP (opcional)
├── README.md                # Documentación
├── commands/
│   └── mi-comando.md        # Un slash command
└── skills/
    └── mi-skill/
        └── SKILL.md          # Una skill automática
```

### 5.2 Ejemplo: Plugin para desarrollo educativo

#### `plugin.json`
```json
{
  "name": "educacion-fp",
  "version": "1.0.0",
  "description": "Plugin para gestión de FP en la Región de Murcia",
  "author": "Lucas Nieto",
  "skills": ["normativa-fp", "consulta-centros"],
  "commands": ["consultar-centro", "generar-informe"]
}
```

#### `skills/normativa-fp/SKILL.md`
```yaml
---
name: normativa-fp
description: Conocimiento sobre normativa de Formación Profesional en la Región de Murcia. Activar cuando el usuario pregunte sobre ciclos, centros o legislación educativa.
---

# Normativa de Formación Profesional

## Fuentes de datos
- Base de datos Supabase con centros educativos
- Normativa vigente del BORM

## Instrucciones
1. Siempre consulta Supabase para datos de centros (no inventes)
2. Cita la fuente normativa cuando sea posible
3. Si no estás seguro de un dato, indícalo claramente

## Restricciones
- NO inventes centros ni ciclos que no existan
- NO des información de otras comunidades autónomas
- NO proporciones datos personales de docentes
```

#### `commands/consultar-centro.md`
```markdown
# /educacion:consultar-centro

Consulta información detallada sobre un centro educativo de FP.

## Pasos
1. Identificar el centro por nombre o localidad
2. Consultar Supabase para obtener datos actualizados
3. Presentar: nombre, dirección, ciclos ofertados, contacto
4. Si hay ambigüedad, mostrar opciones y pedir aclaración

## Formato de salida
| Campo | Valor |
|-------|-------|
| Centro | ... |
| Localidad | ... |
| Ciclos | ... |
| Contacto | ... |
```

### 5.3 Usar el plugin de creación de plugins

Si tienes Claude Code con el plugin `cowork-plugin-management` instalado, puedes crear plugins interactivamente:

```
claude> Quiero crear un plugin para gestionar mi flujo de desarrollo

# Claude activará la skill `create-cowork-plugin` y te guiará paso a paso
```

---

## 6. Crear Workflows reutilizables

Los workflows son scripts de pasos que automatizan tareas repetitivas. Se guardan en `.agent/workflows/` y se invocan como slash commands.

### 6.1 Ejemplo: Workflow de despliegue

Archivo: `.agent/workflows/deploy.md`

```markdown
---
description: Desplegar cambios a producción
---
// turbo-all

1. Ejecutar los tests: `npm test`
2. Hacer build de producción: `npm run build`
3. Verificar que no hay errores de TypeScript: `npx tsc --noEmit`
4. Verificar que no hay errores de lint: `npm run lint`
5. Crear commit: `git add . && git commit -m "deploy: preparar release"`
6. Push al repositorio: `git push origin main`
```

### 6.2 Ejemplo: Workflow de revisión de código

Archivo: `.agent/workflows/code-review.md`

```markdown
---
description: Revisar código antes de hacer PR
---

1. Ejecutar `npx tsc --noEmit` para verificar tipos
2. Ejecutar `npm run lint` para verificar estilo
3. Ejecutar `npm test` para verificar tests
4. Revisar los cambios con `git diff --stat`
5. Identificar archivos modificados y verificar que cada cambio es intencional
6. Generar un resumen de la revisión
```

### 6.3 Ejemplo: Workflow de nuevo componente

Archivo: `.agent/workflows/new-component.md`

```markdown
---
description: Crear un nuevo componente React siguiendo los patrones del proyecto
---

1. Solicitar al usuario: nombre del componente, props esperadas, ubicación
2. Crear el archivo del componente en la carpeta indicada
3. Crear el archivo de estilos (CSS modules o styled-components)
4. Crear el archivo de tests (`*.test.tsx`)
5. Añadir el export al barrel file (`index.ts`) más cercano
6. Verificar con `npx tsc --noEmit`
```

---

## 7. Integrar herramientas con MCP

MCP (Model Context Protocol) es el estándar que usan los plugins para conectarse a herramientas externas. Puedes configurar conexiones en el archivo `.mcp.json` de tu proyecto.

### 7.1 Ejemplo de configuración MCP

```json
{
  "mcpServers": {
    "supabase": {
      "type": "http",
      "url": "https://tu-proyecto.supabase.co/mcp"
    },
    "slack": {
      "type": "http",
      "url": "https://mcp.slack.com/mcp"
    },
    "github": {
      "type": "http",
      "url": "https://mcp.github.com/mcp"
    },
    "notion": {
      "type": "http",
      "url": "https://mcp.notion.com/mcp"
    }
  }
}
```

### 7.2 Servidores MCP disponibles

| Servicio | URL del endpoint MCP | Útil para |
|----------|---------------------|-----------|
| Slack | `https://mcp.slack.com/mcp` | Comunicación de equipo |
| Notion | `https://mcp.notion.com/mcp` | Documentación y wikis |
| GitHub | `https://mcp.github.com/mcp` | Gestión de código |
| Linear | `https://mcp.linear.app/mcp` | Gestión de proyectos |
| Jira | Vía Atlassian MCP | Tickets y sprints |
| Figma | Referenciado en CONNECTORS.md | Diseño |
| Google Calendar | `https://gcal.mcp.claude.com/mcp` | Calendario |
| Gmail | `https://gmail.mcp.claude.com/mcp` | Correo |

> **Nota de seguridad**: Los conectores MCP requieren autenticación OAuth. Solo se activan cuando tú conectas tus credenciales. Nunca envían datos sin tu autorización.

---

## 8. Aplicar los patrones a otros agentes IA

Los 10 patrones de diseño de [03-patrones-de-diseno.md](./03-patrones-de-diseno.md) funcionan con **cualquier agente IA**, no solo Claude Code:

### 8.1 GitHub Copilot

Crea un archivo `.github/copilot-instructions.md` usando los patrones:
- **Modularidad atómica**: Una instrucción por línea
- **Restricciones negativas**: "No generes código con `any` en TypeScript"
- **Directivas con prioridad**: "MUST usar arrow functions para componentes React"

### 8.2 Cursor

Crea un archivo `.cursorrules` aplicando:
- **Ejemplificación few-shot**: Incluye ejemplos de código bueno y malo
- **Fases de ejecución**: Define pasos para tareas como "crear endpoint"
- **Seguridad en capas**: "Siempre valida inputs antes de consultar la base de datos"

### 8.3 Antigravity (Gemini)

Las instrucciones de workspace y las user rules se benefician de:
- **Metadatos estructurados**: Organiza las reglas por categoría
- **Variables de plantilla**: Usa placeholders para adaptar instrucciones

### 8.4 n8n (automatizaciones)

Los nodos de IA en n8n aceptan system prompts. Usa:
- **Identidad y rol**: "Eres un agente que clasifica tickets de soporte"
- **Formato de salida**: "Responde siempre en JSON con los campos: categoría, prioridad, resumen"
- **Restricciones negativas**: "No clasifiques como urgente si no hay impacto en producción"

---

## 9. FAQ

### ¿Necesito Claude Code para usar los plugins?
**Sí y no.** Los plugins están diseñados para Claude Code y Claude Cowork, pero puedes **copiar el contenido de las skills** (archivos SKILL.md) y usarlos como system prompts en cualquier interfaz de Claude o incluso en otros LLMs.

### ¿Los plugins ejecutan código en mi máquina?
**No.** Los plugins son archivos markdown y JSON. No contienen código ejecutable. Las únicas excepciones son los scripts Python del plugin `bio-research` y `data`, que se ejecutan solo si tú lo solicitas.

### ¿Puedo modificar los plugins oficiales?
**Sí.** Todos están bajo licencia Apache 2.0. Puedes copiar un plugin, modificar sus skills/commands, y usarlo como propio. Es la forma recomendada de crear plugins especializados.

### ¿Qué es MCP y necesito configurarlo?
**MCP (Model Context Protocol)** es el protocolo que permite a Claude conectarse a herramientas externas (Slack, Jira, GitHub, etc.). **No es obligatorio.** Todos los plugins funcionan en modo "standalone" sin conectores.

### ¿Puedo usar los patrones de diseño para otros LLMs (GPT, Gemini, Llama)?
**Sí.** Los patrones son agnósticos al modelo. Funcionan especialmente bien con modelos que siguen instrucciones de forma robusta (GPT-4, Claude, Gemini Pro). Los patrones de few-shot y restricciones negativas son universales.

### ¿Cómo sé qué system prompt se activa en cada momento?
Consulta el documento [02-clasificacion-y-casos-de-uso.md](./02-clasificacion-y-casos-de-uso.md). Incluye una tabla con cada prompt, su categoría, cuándo se activa y si está siempre activo o es condicional.

### ¿Los system prompts contienen datos sensibles o malware?
**No.** El análisis completo de seguridad está en [01-analisis-seguridad.md](./01-analisis-seguridad.md). Son archivos de texto plano sin credenciales, tokens ni código malicioso.

### ¿Cómo creo un CLAUDE.md efectivo?
Usa la plantilla de la [sección 3](#3-mejorar-tu-claudemd-con-los-patrones) de este documento. Lo más importante: sé específico con tus reglas y usa la jerarquía MUST > SHOULD > MAY.

---

## 10. Sección Técnica

### 10.1 Tecnologías y herramientas del ecosistema

| Tecnología | Propósito | Documentación |
|-----------|-----------|--------------|
| **Claude Code** | Agente de programación en terminal | [docs.anthropic.com](https://docs.anthropic.com) |
| **Claude Cowork** | Agente de productividad en web | [claude.com/product/cowork](https://claude.com/product/cowork) |
| **MCP** | Protocolo de conexión a herramientas | [modelcontextprotocol.io](https://modelcontextprotocol.io) |
| **tweakcc** | Personalizador de system prompts | [github.com/Piebald-AI/tweakcc](https://github.com/Piebald-AI/tweakcc) |
| **Piebald** | IDE con IA de los creadores de este repo | [piebald.ai](https://piebald.ai) |

### 10.2 Formato de los archivos del sistema

| Formato | Extensión | Uso |
|---------|-----------|-----|
| Markdown | `.md` | System prompts, skills, commands, documentación |
| JSON | `.json` | Manifiestos de plugins, configuración MCP, settings |
| YAML (frontmatter) | Dentro de `.md` | Metadatos de skills (name, description) |

### 10.3 Estructura interna de los system prompts

Los prompts se almacenan internamente como **template literals** fragmentados:

```javascript
{
  pieces: ["Use the ", " tool to create files"],
  identifiers: [42],
  identifierMap: { "42": "WRITE_TOOL_NAME" }
}
// Se reconstruye como: "Use the ${WRITE_TOOL_NAME} tool to create files"
```

Esto permite que Anthropic inyecte nombres de herramientas y estado del sistema en runtime sin reescribir el prompt completo.

### 10.4 Estadísticas del repositorio

| Métrica | Valor |
|---------|-------|
| System prompts | 223 archivos |
| Plugins | 19 (15 Anthropic + 4 partners) |
| Skills totales | ~80 |
| Commands totales | ~70 |
| Servicios MCP | ~40 únicos |
| Versión de Claude Code | v2.1.62 |
| Licencia | Apache License 2.0 |

---

## 11. Seguridad

### 11.1 Seguridad de los System Prompts

| Aspecto | Estado | Detalle |
|---------|--------|---------|
| Código malicioso | ✅ Seguro | Son archivos de texto plano (markdown/JSON) |
| Credenciales | ✅ Seguro | No contienen tokens, API keys ni passwords |
| Inyección de prompts | ✅ Seguro | No contienen payloads maliciosos |
| Exfiltración de datos | ✅ Seguro | No transmiten datos a terceros |

### 11.2 Seguridad de los Plugins

| Aspecto | Estado | Detalle |
|---------|--------|---------|
| Código ejecutable | ✅ Sin riesgo | Los plugins son solo markdown y JSON |
| Scripts Python | ⚠️ Bajo riesgo | Solo en `bio-research` y `data`. No se ejecutan automáticamente |
| Conectores MCP | ⚠️ Atención | Requieren autenticación OAuth. Solo se activan si el usuario conecta sus credenciales |

### 11.3 Recomendaciones de seguridad

1. **No subas tu `.mcp.json` a repositorios públicos** si contiene URLs de servicios internos
2. **Revisa los scripts Python** del plugin `bio-research` antes de ejecutarlos
3. **No compartas tu `settings.local.json`** si contiene datos personales o de empresa
4. **Usa el modo sandbox** siempre que ejecutes código generado por IA
5. **Revisa los permisos MCP** antes de conectar servicios: asegúrate de entender qué acceso das al agente

### 11.4 Modelo de amenazas para plugins personalizados

Si creas tus propios plugins, plantéate:

| Pregunta | Si la respuesta es "sí" |
|----------|------------------------|
| ¿El plugin accede a datos sensibles vía MCP? | Documenta qué datos accede y por qué |
| ¿El plugin incluye scripts ejecutables? | Documenta qué hacen y requiere aprobación del usuario |
| ¿El plugin se compartirá públicamente? | Verifica que no contenga datos internos ni credenciales |
| ¿El plugin modifica archivos del sistema? | Implementa confirmación explícita antes de cada modificación |

---

*Documento creado como parte del análisis de los System Prompts de Claude Code v2.1.62.*
*Repositorio fuente: [claude-code-system-prompts](https://github.com/Piebald-AI/claude-code-system-prompts)*
