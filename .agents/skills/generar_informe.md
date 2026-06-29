# Skill: Generación de Informes de Labores Mensuales

**Descripción de Activación:** Ejecuta este flujo cuando el usuario solicite la generación de un informe de labores mensual (ej. *"genera el informe de junio de 2026"* o *"genera el informe de labores de [mes] [año]"*), con o sin la indicación de actividades adicionales a incluir de forma explícita.

**Objetivo:** Recopilar y consolidar de forma automatizada las actividades de la bitácora de un mes determinado, clasificarlas en los objetos del contrato, redactar el informe en texto plano y en markdown con continuidad técnica respecto a meses previos, y almacenarlos en el directorio correspondiente con el formato requerido.

---

## Variables del Skill

- **Índice maestro:** `rsa/RSA-Metodologias/indice/indice_tematico.md`
- **Catálogo de contribuidores:** `rsa/RSA-Metodologias/indice/catalogo_contribuidores.md`
- **Ruta base de informes:** `institucional/RSA-Bitacora-LLM-Milton/informes/`
- **Usuario objetivo:** `@Milton` (usuario institucional)

---

## Objetos del Contrato (Clasificación)

Las actividades del mes deben clasificarse estrictamente en uno de los siguientes 7 objetos contractuales:

1. **Ensamblado, configuración e instalación de estaciones digitales con 6 componentes** para la actualización y ampliación de la red de monitoreo sísmico, integrando arquitecturas IoT para la transmisión distribuida de datos.
2. **Implementación de funcionalidades de autodetección de eventos** mediante técnicas basadas en machine learning y acceso a datos en tiempo real.
3. **Ensamblado, configuración e instalación de acelerógrafos de 3 componentes** con integración a la red de digitalizadores de 6 componentes.
4. **Configuración, instalación y mantenimiento de los sistemas desarrollados** para el monitoreo automatizado de variables estáticas en las presas propiedad de Elecaustro.
5. **Administración de la red de sensores distribuidos en la presa**, garantizando la recolección y transmisión precisa de datos en tiempo real.
6. **Implementación de una solución de gestión y preservación de datos** para el manejo del almacenamiento en las estaciones de monitoreo.
7. **Implementación de un sistema de visualización y monitoreo de métricas operativas** basado en protocolos de telemetría ligera.

---

## Pasos de Ejecución

### 1. Extracción del Período y Datos Adicionales
- Identifica el **mes** y el **año** solicitados por el usuario.
- Extrae cualquier actividad adicional proporcionada en el prompt o como archivo adjunto por el usuario.

### 2. Consulta y Lectura del Historial de Sesiones
- Abre `rsa/RSA-Metodologias/indice/indice_tematico.md` y busca las sesiones de `@Milton` que correspondan al año y mes solicitados (ej. `2026/06/` para junio de 2026).
- Utiliza `rsa/RSA-Metodologias/indice/catalogo_contribuidores.md` para resolver la ruta local de las sesiones de `@Milton` (típicamente `institucional/RSA-Bitacora-LLM-Milton/sesiones/`).
- Lee el contenido de todos los archivos de sesión correspondientes al mes solicitado para extraer todas las labores, hitos técnicos y decisiones tomadas.

### 3. Consulta de Informes Previos para Contexto y Continuidad
- Revisa el directorio `institucional/RSA-Bitacora-LLM-Milton/informes/YYYY/` (donde YYYY es el año solicitado).
- Abre el informe del mes inmediatamente anterior al solicitado en formato texto plano (ej. `05_informe.txt` o `05 informe.txt` si se genera el de Junio).
- Analiza:
  - El estilo y vocabulario de redacción técnico formal.
  - La continuidad de las justificaciones de los objetos marcados como `No ejecutado en el presente mes.`.
  - El campo `Mes previsto de cumplimiento` para verificar si un objeto debía ejecutarse en el mes actual y verificar en las sesiones si se cumplió o no.

### 4. Clasificación y Redacción de Entregables
Clasifica las actividades recopiladas de las sesiones del mes y las actividades adicionales especificadas por el usuario en los 7 objetos contractuales. A partir de esto, genera dos representaciones del informe:

#### A. Archivo de Texto Plano (`XX_informe.txt`)
Redacta cada uno de los 7 objetos siguiendo las siguientes reglas:
- **Si el objeto tuvo actividades (Ejecutado):**
  ```text
  N. [Texto exacto del objeto del contrato]
  - Estado: Ejecutado
  - Actividades realizadas: [Párrafo unificado, fluido y formal en español que detalle las labores técnicas, usando jerga técnica precisa. NO usar viñetas, listas ni guiones para describir las tareas aquí].
  ```
- **Si el objeto NO tuvo actividades (No ejecutado):**
  ```text
  N. [Texto exacto del objeto del contrato]
  - Estado: No ejecutado en el presente mes.
  - Justificación: [Redacción en español formal que justifique técnicamente el porqué no se ejecutó, en base a la continuidad o dependencias. Evita justificaciones repetitivas].
  - Mes previsto de cumplimiento: [Mes previsto, ej: Agosto].
  ```

#### B. Archivo Markdown (`XX_reporte.md`)
El archivo Markdown debe ser directamente importable en Notion y **NO debe contener ningún frontmatter YAML** al inicio. Su estructura debe ser la siguiente:
1. **Título Principal:** `# Reporte de evidencias - [Nombre de Mes con Mayúscula] [Año]` (ej. `# Reporte de evidencias - Junio 2026`).
2. **Divisor:** `---`
3. **Sección Introducción:**
   ```markdown
   # Introducción

   El presente documento corresponde al detalle de las actividades desarrolladas durante el mes de [mes en minúscula] de [año] en el marco de las tareas de la "Red Sísmica del Austro (RSA)". Específicamente se aportan los medios de verificación de las tareas realizadas y su correspondencia con el objeto del contrato.
   ```
4. **Divisor:** `---`
5. **Sección Avances:**
   ```markdown
   # Avances en los objetos del contrato

   Durante el mes de [mes en minúscula] de [año] se trabajó en los siguientes objetos del contrato:
   ```
   Seguido por la lista completa de los 7 objetos contractuales, cada uno en su propio renglón separado por líneas en blanco, utilizando los emojis:
   - `✅ [Texto del objeto del contrato]  (objeto N)` si fue ejecutado (notar el doble espacio antes del paréntesis del objeto).
   - `🔲 [Texto del objeto del contrato] (objeto N)` si no fue ejecutado (notar el espacio simple antes del paréntesis del objeto).
6. **Divisor:** `---`
7. **Sección Detalle de Objetos Cumplidos:**
   - **Únicamente lista los objetos que fueron ejecutados (marcados con ✅).** Los objetos no ejecutados no deben figurar en esta sección.
   - Cada objeto cumplido se representa con un título de nivel 2: `## [Texto exacto del objeto del contrato]` (sin el prefijo numérico de objeto en el título `##`, y sin el sufijo `(objeto N)`).
   - Las actividades de cada objeto se listan como viñetas con guiones (`- [Actividad]`), dividiendo las distintas oraciones o tareas técnicas del párrafo del informe en elementos individuales de la lista.
   - Inserta un divisor `---` después de cada objeto cumplido.
8. **Sección Conclusiones:**
   - Termina con un encabezado de primer nivel: `# Conclusiones del mes`
   - Lista con guiones (`- `) de 2 a 3 conclusiones técnicas generales del mes que sinteticen los principales logros alcanzados de forma consolidada.

### 5. Escritura de los Archivos
- Calcula el prefijo numérico de dos dígitos del mes solicitado (ej. `06` para junio).
- Guarda el informe de texto plano en `institucional/RSA-Bitacora-LLM-Milton/informes/YYYY/XX_informe.txt` (ej. `informes/2026/06_informe.txt`).
- Guarda el reporte Markdown en `institucional/RSA-Bitacora-LLM-Milton/informes/YYYY/XX_reporte.md` (ej. `informes/2026/06_reporte.md`).
- Si las carpetas del año no existen, créalas de forma automática.

### 6. Confirmación al Usuario
- Presenta al usuario un resumen en el chat indicando:
  - La ruta del informe de texto plano generado.
  - La ruta del reporte Markdown generado.
  - Una confirmación de la correcta aplicación del formato de Notion (checklists con emojis, sección de introducción, desglose de objetos cumplidos y conclusiones).
