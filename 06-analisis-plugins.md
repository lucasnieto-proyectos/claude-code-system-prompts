# Análisis de Plugins de Claude Cowork / Claude Code

Este documento analiza los **19 plugins oficiales** del repositorio [PUBLICO-claude-cowork-plugins](https://github.com/lucasnieto-proyectos/PUBLICO-claude-cowork-plugins), diseñados para convertir a Claude en un especialista para diferentes roles profesionales. Están construidos para [Claude Cowork](https://claude.com/product/cowork) y son compatibles con [Claude Code](https://claude.com/product/claude-code).

---

## 1. ¿Qué son los plugins?

Los plugins son **paquetes de conocimiento de dominio** que especializan a Claude para funciones profesionales concretas. A diferencia de los system prompts (que configuran el comportamiento base del agente), los plugins añaden:

- **Skills**: conocimiento de dominio que Claude activa automáticamente cuando es relevante
- **Commands**: workflows explícitos que el usuario invoca con slash commands (ej: `/sales:call-prep`)
- **Connectors**: conexiones MCP a herramientas externas (CRMs, project trackers, etc.)

> **Relación con los system prompts**: Los system prompts del documento [02-clasificacion-y-casos-de-uso.md](./02-clasificacion-y-casos-de-uso.md) definen *cómo* opera Claude internamente. Los plugins definen *qué sabe* Claude sobre un dominio específico y *con qué herramientas* puede interactuar.

---

## 2. Arquitectura de un Plugin

Todos los plugins siguen la misma estructura de archivos:

```
plugin-name/
├── .claude-plugin/plugin.json   # Manifiesto del plugin
├── .mcp.json                    # Servidores MCP (conexiones a herramientas)
├── CONNECTORS.md                # Documentación de conectores alternativos
├── README.md                    # Documentación del plugin
├── LICENSE                      # Licencia (Apache 2.0)
├── commands/                    # Slash commands invocables por el usuario
│   └── command-name.md          # Instrucciones del comando en markdown
└── skills/                      # Conocimiento de dominio automático
    └── skill-name/
        ├── SKILL.md             # Instrucciones de la skill (frontmatter YAML + markdown)
        ├── references/          # Material de referencia adicional
        └── scripts/             # Scripts auxiliares (Python, etc.)
```

### 2.1 SKILL.md — Formato

Las skills usan un formato estándar con frontmatter YAML:

```yaml
---
name: nombre-de-la-skill
description: Descripción corta de cuándo activarla
---

# Título de la Skill

[Instrucciones detalladas en markdown]
```

### 2.2 .mcp.json — Conectores

Define las conexiones a herramientas externas via MCP (Model Context Protocol):

```json
{
  "mcpServers": {
    "slack": {
      "type": "http",
      "url": "https://mcp.slack.com/mcp"
    },
    "notion": {
      "type": "http",
      "url": "https://mcp.notion.com/mcp"
    }
  }
}
```

### 2.3 marketplace.json — Registro centralizado

El archivo `.claude-plugin/marketplace.json` en la raíz registra todos los plugins disponibles con nombre, fuente y descripción.

---

## 3. Catálogo Completo de Plugins

### 3.1 Plugins de Anthropic (15)

| Plugin | Skills | Commands | Conectores MCP |
|--------|--------|----------|---------------|
| **productivity** | `memory-management`, `task-management` | `/start`, `/update` | Slack, Notion, Asana, Linear, Atlassian, MS365, Monday, ClickUp, Google Calendar, Gmail |
| **sales** | `account-research`, `call-prep`, `daily-briefing`, `draft-outreach`, `competitive-intelligence`, `create-an-asset` | `/call-summary`, `/forecast`, `/pipeline-review` | Slack, HubSpot, Close, Clay, ZoomInfo, Notion, Jira, Fireflies, MS365 |
| **customer-support** | `customer-research`, `escalation`, `knowledge-management`, `response-drafting`, `ticket-triage` | `/draft-response`, `/escalate`, `/kb-article`, `/research`, `/triage` | Slack, Intercom, HubSpot, Guru, Jira, Notion, MS365 |
| **product-management** | `competitive-analysis`, `feature-specification`, `metrics-analytics`, `release-planning`, `roadmap-planning`, `user-research-synthesis` | `/competitive-brief`, `/feature-spec`, `/launch-plan`, `/roadmap`, `/stakeholder-update`, `/user-research` | Slack, Linear, Asana, Monday, ClickUp, Jira, Notion, Figma, Amplitude, Pendo, Intercom, Fireflies |
| **marketing** | `brand-voice`, `campaign-planning`, `competitive-analysis`, `content-creation`, `performance-analytics` | `/brand-review`, `/campaign-plan`, `/competitive-brief`, `/draft-content`, `/email-sequence`, `/performance-report`, `/seo-audit` | Slack, Canva, Figma, HubSpot, Amplitude, Notion, Ahrefs, SimilarWeb, Klaviyo |
| **legal** | `canned-responses`, `compliance`, `contract-review`, `legal-risk-assessment`, `meeting-briefing`, `nda-triage` | `/brief`, `/compliance-check`, `/respond`, `/review-contract`, `/signature-request`, `/triage-nda`, `/vendor-check` | Slack, Box, Egnyte, Jira, MS365 |
| **finance** | `audit-support`, `close-management`, `financial-statements`, `journal-entry-prep`, `reconciliation`, `variance-analysis` | `/income-statement`, `/journal-entry`, `/reconciliation`, `/sox-testing`, `/variance-analysis` | Snowflake, Databricks, BigQuery, Slack, MS365 |
| **data** | `data-context-extractor`, `data-exploration`, `data-validation`, `data-visualization`, `interactive-dashboard-builder`, `sql-queries`, `statistical-analysis` | `/analyze`, `/build-dashboard`, `/create-viz`, `/explore-data`, `/validate`, `/write-query` | Snowflake, Databricks, BigQuery, Hex, Amplitude, Jira |
| **enterprise-search** | `knowledge-synthesis`, `search-strategy`, `source-management` | `/digest`, `/search` | Slack, Notion, Guru, Jira, Asana, MS365 |
| **bio-research** | `instrument-data-to-allotrope`, `nextflow-development`, `scientific-problem-selection` | `/start` | PubMed, BioRender, bioRxiv, ClinicalTrials.gov, ChEMBL, Synapse, Wiley, Owkin, Open Targets, Benchling |
| **cowork-plugin-management** | `cowork-plugin-customizer`, `create-cowork-plugin` | — | — |
| **engineering** | `code-review`, `documentation`, `incident-response`, `system-design`, `tech-debt`, `testing-strategy` | `/architecture`, `/debug`, `/deploy-checklist`, `/incident`, `/review`, `/standup` | GitHub, GitLab, Linear, Jira, Asana, Datadog, New Relic, PagerDuty, Opsgenie, Slack, Notion, Confluence |
| **human-resources** | `compensation-benchmarking`, `employee-handbook`, `interview-prep`, `org-planning`, `people-analytics`, `recruiting-pipeline` | `/comp-analysis`, `/draft-offer`, `/onboarding`, `/people-report`, `/performance-review`, `/policy-lookup` | Workday, BambooHR, Rippling, Greenhouse, Lever, Ashby, Pave, Radford, Slack, MS365 |
| **design** | `accessibility-review`, `design-critique`, `design-handoff`, `design-system-management`, `user-research`, `ux-writing` | `/accessibility`, `/critique`, `/design-system`, `/handoff`, `/research-synthesis`, `/ux-copy` | Figma, Intercom, Productboard, Linear, Asana, Jira, Notion, Amplitude, Mixpanel |
| **operations** | `vendor-management`, `process-optimization`, `change-management`, `risk-assessment`, `compliance-tracking`, `resource-planning` | `/vendor-review`, `/process-doc`, `/change-request`, `/capacity-plan`, `/status-report`, `/runbook` | ServiceNow, Zendesk, Asana, Jira, Monday, Notion, Confluence, Slack, MS365 |

### 3.2 Plugins de Partners (4)

| Plugin | Autor | Descripción |
|--------|-------|-------------|
| **slack** | Salesforce | Búsqueda de mensajes, comunicaciones, gestión de canvases en Slack |
| **apollo** | Apollo.io | Prospección, enriquecimiento de leads y secuencias de outreach |
| **common-room** | Common Room | Copiloto GTM con investigación de cuentas y contactos |
| **brand-voice** | Tribe AI | Descubrimiento de voz de marca y validación de contenido |

---

## 4. Análisis de Seguridad

### 4.1 Conectores MCP

Los plugins se conectan a herramientas externas mediante **MCP (Model Context Protocol)**. Las URLs de los servidores MCP son endpoints HTTP públicos:

| Servicio | URL del endpoint MCP |
|----------|---------------------|
| Slack | `https://mcp.slack.com/mcp` |
| Notion | `https://mcp.notion.com/mcp` |
| Asana | `https://mcp.asana.com/v2/mcp` |
| Linear | `https://mcp.linear.app/mcp` |
| Atlassian | `https://mcp.atlassian.com/v1/mcp` |
| Microsoft 365 | `https://microsoft365.mcp.claude.com/mcp` |
| Monday | `https://mcp.monday.com/mcp` |
| ClickUp | `https://mcp.clickup.com/mcp` |
| Google Calendar | `https://gcal.mcp.claude.com/mcp` |
| Gmail | `https://gmail.mcp.claude.com/mcp` |
| HubSpot | Referenciado en CONNECTORS.md |
| Figma | Referenciado en CONNECTORS.md |

#### Evaluación de seguridad

| Aspecto | Evaluación | Notas |
|---------|------------|-------|
| **Código malicioso** | ✅ Sin riesgo | Los plugins son solo archivos markdown y JSON, sin código ejecutable |
| **Scripts auxiliares** | ⚠️ Bajo riesgo | El plugin `bio-research` incluye scripts Python; el plugin `data` incluye un script Python para empaquetado. Estos scripts no se ejecutan automáticamente |
| **Conexiones externas** | ⚠️ Atención | Los `.mcp.json` definen conexiones a servicios externos que requieren autenticación. Solo se activan si el usuario conecta sus credenciales |
| **Datos sensibles** | ✅ Sin datos sensibles hardcodeados | No contienen tokens, API keys ni credenciales |
| **Inyección de prompts** | ✅ Sin riesgo detectado | Las instrucciones son directas y no contienen payloads maliciosos |
| **Exfiltración de datos** | ✅ Sin riesgo | Los plugins no envían datos a terceros por sí mismos |

### 4.2 Scripts auxiliares encontrados

Los siguientes plugins incluyen scripts Python:

- **bio-research**:
  - `scripts/convert_to_asm.py`, `export_parser.py`, `flatten_asm.py`, `validate_asm.py` (conversión Allotrope)
  - `scripts/check_environment.py`, `detect_data_type.py`, `generate_samplesheet.py`, etc. (Nextflow)
  - `scripts/qc_plotting.py` (QC)
- **data**:
  - `scripts/package_data_skill.py` (empaquetado de skills de datos)

> Estos scripts son herramientas auxiliares que el agente puede invocar opcionalmente, pero no se ejecutan de forma autónoma ni sin intervención del usuario.

---

## 5. Patrones de Diseño Identificados

### 5.1 Patrón Skill + Command

Cada plugin separa el conocimiento en dos capas:

```
┌─────────────────────────────────────────┐
│              USUARIO                     │
│                                          │
│  Invocación explícita    Uso natural     │
│  /sales:call-prep        "investiga       │
│                           esta empresa"   │
│          │                     │          │
│          ▼                     ▼          │
│     COMMANDS              SKILLS          │
│     (explícitos)          (automáticos)   │
│                                          │
│          └──────┐  ┌──────┘              │
│                 ▼  ▼                      │
│           CONNECTORS (MCP)                │
│         herramientas externas             │
└─────────────────────────────────────────┘
```

### 5.2 Patrón "Standalone + Supercharged"

Todos los plugins están diseñados para funcionar en **dos modos**:

1. **Standalone**: el usuario proporciona contexto manualmente (pegar notas, subir CSV, describir situación)
2. **Supercharged**: con conectores MCP activos, el agente accede automáticamente a datos de CRMs, email, calendarios, etc.

### 5.3 Patrón de Personalización

Los plugins son genéricos por defecto pero personalizables:

- **Swap connectors**: editar `.mcp.json` para apuntar a las herramientas específicas
- **Add company context**: añadir terminología, estructura organizativa y procesos propios en los archivos skill
- **Adjust workflows**: modificar instrucciones de skills para ajustar a los procesos reales del equipo
- **Settings file**: archivo `settings.local.json` para datos de personalización (nombre, equipo, empresa, etc.)

### 5.4 Patrón File-Based

Todo el sistema de plugins está basado en archivos:

- **Sin código compilado**: markdown y JSON exclusivamente
- **Sin infraestructura**: no requiere servidores, bases de datos ni despliegues
- **Sin build steps**: los archivos se usan directamente tal como están
- **Versionable**: al ser archivos de texto, se integran naturalmente con git

---

## 6. Relación con los System Prompts de Claude Code

### Puntos de integración

Los plugins interactúan con varios system prompts documentados en este repositorio:

| System Prompt | Relación con plugins |
|---------------|---------------------|
| [Tool Usage: Skill Invocation](./system-prompts/system-prompt-tool-usage-skill-invocation.md) | Define cómo los slash commands invocan skills vía la herramienta Skill |
| [Tool Description: Skill](./system-prompts/tool-description-skill.md) | Descripción de la herramienta que ejecuta skills en la conversación |
| [System Prompt: Skillify Current Session](./system-prompts/system-prompt-skillify-current-session.md) | Permite convertir una sesión actual en una skill reutilizable |
| [System Reminder: Invoked Skills](./system-prompts/system-reminder-invoked-skills.md) | Lista de skills invocadas en la sesión activa |

### Diferencias clave

| Aspecto | System Prompts | Plugins |
|---------|---------------|---------|
| **Alcance** | Comportamiento general del agente | Conocimiento específico de dominio |
| **Activación** | Siempre presentes | Se activan por relevancia o invocación |
| **Modificación** | Requiere parchear el código de Claude Code | Solo editar archivos markdown |
| **Distribución** | Embedded en el binario | Instalables desde el marketplace |
| **Personalización** | Vía CLAUDE.md y configuración | Vía skill files y settings.json |

---

## 7. Estadísticas del Repositorio de Plugins

| Métrica | Valor |
|---------|-------|
| Total de plugins | 19 (15 Anthropic + 4 Partner) |
| Total de skills | ~80 |
| Total de commands | ~70 |
| Servicios MCP referenciados | ~40 únicos |
| Lenguaje de los archivos | Markdown + JSON |
| Scripts auxiliares | ~15 (Python) |
| Licencia | Apache License 2.0 |

---

## 8. Cómo instalar y usar los plugins

### En Claude Cowork

```
Instalar desde claude.com/plugins
```

### En Claude Code

```bash
# Añadir el marketplace
claude plugin marketplace add anthropics/knowledge-work-plugins

# Instalar un plugin específico
claude plugin install sales@knowledge-work-plugins
```

### Uso

Una vez instalados, los plugins se activan automáticamente:

- **Skills**: se activan por contexto cuando el usuario interactúa de forma natural
- **Commands**: se invocan explícitamente con slash commands (ej: `/sales:call-prep`, `/finance:reconciliation`)
