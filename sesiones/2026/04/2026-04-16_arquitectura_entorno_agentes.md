---
fecha: 2026-04-16
temas: [antigravity, exocortex, agentes, prompt_engineering]
entorno: [agentes]
---

# Registro de Sesión: Arquitectura del Entorno Agéntico y Sistema Exocortex
# Actividad del 2026-04-16

#Hitos de la jornada:
Se diseñó y estructuró un repositorio centralizado (RSA-Agent-Workspace) para gobernar el comportamiento de entornos de desarrollo asistidos por IA (como Google Antigravity). La arquitectura implementada resuelve el problema de la gestión del contexto a largo plazo mediante la división de la información en "Memoria Episódica" (bitácoras cronológicas de sesiones) y "Memoria Semántica" (Exocortex indexado), permitiendo mantener un registro técnico continuo sin redundancia de datos y optimizando el consumo de tokens en consultas futuras.

Se desarrollaron e iteraron políticas de control (Rules) y flujos de automatización (Skills) que dictan cómo interactúa el agente con el sistema de archivos local y con hardware remoto montado por red. Esto incluyó la resolución de un problema crítico de interferencia entre las instrucciones personalizadas y la memoria nativa del IDE (Knowledge Items), aplicando anti-prompts y comandos de activación específicos para garantizar la ejecución determinista de los flujos de trabajo.


# Decisiones y Cambios:

- Se estableció el directorio raíz git/ como entorno de trabajo global (Workspace), utilizando enlaces simbólicos o patrones Glob para aplicar reglas a subdirectorios específicos de proyectos.

- Se definió la regla restriccion_sshfs.md con alcance limitado a la ruta montajes/, prohibiendo terminantemente al agente la ejecución autónoma de comandos sobre hardware remoto y obligando la delegación de estas tareas al usuario.

- Se implementó un sistema de indexación ligero basado en bloques YAML (frontmatter) y un archivo enrutador central (exocortex/indice_tematico.md) para que el agente recupere información histórica de forma selectiva, previniendo el desbordamiento de la ventana de contexto.

- Se estableció el uso de "ejecuta el volcado de bitácora" como comando de activación estricto para evitar colisiones semánticas con el sistema de memoria nativo del agente.

# Scripts/Comandos relevantes:
```md
# Archivo: .agents/rules/restriccion_sshfs.md
**Alcance de la Regla:** ESTA REGLA SOLO APLICA para archivos y acciones dentro de la ruta relativa `montajes/` y sus subdirectorios. No aplica para otros directorios del workspace.

**Reglas de Comportamiento (Cumplimiento Obligatorio):**
1. **PROHIBICIÓN DE EJECUCIÓN AUTÓNOMA:** Si el contexto de tu tarea actual involucra la ruta `montajes/`, tienes estrictamente prohibido tomar el control de la terminal para ejecutar comandos de forma autónoma.
2. **DELEGACIÓN OBLIGATORIA:** Cada vez que tu razonamiento determine que es necesario ejecutar un comando, DEBES detenerte y solicitar al usuario que lo ejecute.
3. **FORMATO DE ENTREGA:** Debes proporcionar el comando exacto en un bloque de código Bash e instruir al usuario para que lo ejecute en su sesión remota.
```

```md
# Archivo: .agents/skills/consulta_historica.md
**Descripción de Activación:** Ejecuta este flujo cuando el usuario realice preguntas sobre el estado pasado de un proyecto.

**Objetivo:** Minimizar el consumo de tokens cargando únicamente los archivos estrictamente necesarios.

**Pasos de Ejecución (Cumplimiento Estricto):**
1. **Consulta del Índice:** Tienes prohibido usar comandos de búsqueda general. Tu primera acción OBLIGATORIA es leer `exocortex/indice_tematico.md`.
2. **Selección de Archivos:** Cruza la consulta del usuario con las categorías del índice. Extrae únicamente las rutas de los archivos `.md` coincidentes.
3. **Carga Estrictamente Necesaria:** Lee el contenido de los archivos específicos identificados (máximo 3 a 5 archivos).
4. **Síntesis y Respuesta:** Responde de forma directa y técnica, incluyendo referencias a los archivos consultados.
```

```md
# Archivo: .agents/skills/volcado_bitacora.md (Fragmento de Indexación Automática)
**Extracción Automática de Metadatos:**
- Extrae de 1 a 4 palabras clave que definan tecnologías o conceptos tratados.
- Prepara el bloque frontmatter en formato YAML.

**Actualización del Índice Temático (exocortex/indice_tematico.md):**
- Verifica las categorías de `temas` y `entorno`.
- Si la categoría ya existe, añade una coma, un espacio y la nueva ruta relativa al final de esa misma línea.
- Si la categoría no existe, crea una nueva línea con la viñeta y la ruta bajo la sección correspondiente.
- No elimines ni sobrescribas ninguna ruta existente.
```