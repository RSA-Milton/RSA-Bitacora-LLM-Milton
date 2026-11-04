---
name: generar_informe
description: "Recopila y clasifica las actividades mensuales de la bitácora en los objetos contractuales y genera el informe de labores en texto plano (FOR-10)."
---

# Skill: Generación de Informes de Labores Mensuales

**Descripción de Activación:** Ejecuta este flujo cuando el usuario solicite la generación de un informe de labores mensual (ej. *"genera el informe de septiembre de 2026"* o *"genera el informe de labores de [mes] [año]"*), con o sin la indicación de actividades adicionales a incluir de forma explícita.

**Objetivo:** Recopilar y consolidar de forma automatizada las actividades de la bitácora de un mes determinado, clasificarlas en los objetos del contrato, redactar el informe en texto plano formal continuo con trazabilidad respecto a meses previos, y almacenarlo en el repositorio local y en Google Drive según la directriz institucional FOR-10.

---

## ⚙️ Variables de Configuración por Defecto

Estas variables rigen la parametrización contractual y pueden sobreescribirse si el contrato o profesional cambian en el futuro:

```text
PROFESIONAL_NOMBRE = "Ing. Milton Rodrigo Muñoz Calle"
NUMERO_CONTRATO = "CO-01056-2026"
ENTIDAD_CONTRATANTE = "Universidad de Cuenca"
DEPENDENCIA = "Departamento de Ingeniería Civil | Red Sísmica del Austro (RSA)"
USUARIO_OBJETIVO = "@Milton"

RUTA_BASE_INFORMES = "institucional/RSA-Bitacora-LLM-Milton/informes/"
RUTA_DRIVE_INFORMES = "H:\Mi unidad\Informes RSA\Informes YYYY\MM mes\"
INDICE_MAESTRO = "rsa/RSA-Metodologias/indice/indice_tematico.md"
CATALOGO_CONTRIBUIDORES = "rsa/RSA-Metodologias/indice/catalogo_contribuidores.md"
PLANTILLA_INFORME = "templates/informe_template.txt"
```

### Objetos del Contrato por Defecto

Las actividades del mes deben clasificarse estrictamente en uno de los siguientes objetos contractuales configurados:

1. **Ensamblado, configuración e instalación de estaciones digitales con 6 componentes** para la actualización y ampliación de la red de monitoreo sísmico, integrando arquitecturas IoT para la transmisión distribuida de datos.
2. **Implementación de funcionalidades de autodetección de eventos** mediante técnicas basadas en machine learning y acceso a datos en tiempo real.
3. **Ensamblado, configuración e instalación de acelerógrafos de 3 componentes** con integración a la red de digitalizadores de 6 componentes.
4. **Configuración, instalación y mantenimiento de los sistemas desarrollados** para el monitoreo automatizado de variables estáticas en las presas propiedad de Elecaustro.
5. **Administración de la red de sensores distribuidos en la presa**, garantizando la recolección y transmisión precisa de datos en tiempo real.
6. **Implementación de una solución de gestión y preservación de datos** para el manejo del almacenamiento en las estaciones de monitoreo.
7. **Implementación de un sistema de visualización y monitoreo de métricas operativas** basado en protocolos de telemetría ligera.

---

## 📋 Pasos de Ejecución

### 1. Extracción del Período y Datos Adicionales
- Identifica el **mes** y el **año** solicitados por el usuario.
- Extrae cualquier actividad adicional proporcionada en el prompt o como archivo adjunto por el usuario.

### 2. Consulta y Lectura del Historial de Sesiones
- Abre `rsa/RSA-Metodologias/indice/indice_tematico.md` y busca las sesiones de `USUARIO_OBJETIVO` que correspondan al año y mes solicitados (ej. `2026/09/` para septiembre de 2026).
- Utiliza `rsa/RSA-Metodologias/indice/catalogo_contribuidores.md` para resolver la ruta local de las sesiones (típicamente `institucional/RSA-Bitacora-LLM-Milton/sesiones/`).
- Lee el contenido de todos los archivos de sesión correspondientes al mes solicitado para extraer todas las labores, hitos técnicos, diagnósticos y decisiones de arquitectura tomadas.

### 3. Consulta de Informes Previos para Contexto y Continuidad
- Revisa el directorio `institucional/RSA-Bitacora-LLM-Milton/informes/YYYY/` (donde YYYY es el año solicitado).
- Abre el informe del mes inmediatamente anterior al solicitado en formato texto plano (ej. `08_informe.txt` si se genera el de septiembre).
- Analiza:
  - El estilo y vocabulario de redacción técnico formal.
  - La continuidad de las justificaciones de los objetos marcados como `No ejecutado en el presente mes.`.
  - El campo `Mes previsto de cumplimiento` para verificar si un objeto debía ejecutarse en el mes actual y verificar en las sesiones si se cumplió o no.

### 4. Clasificación y Redacción del Entregable en Texto Plano (`XX_informe.txt`)
Utiliza la estructura base de `templates/informe_template.txt` y clasifica las actividades recopiladas en los objetos contractuales siguiendo estas reglas estrictas de formato:

- **Si el objeto tuvo actividades (Ejecutado):**
  - Encabezado con el número y texto exacto del objeto contractual.
  - Línea de estado con viñeta y tabulación: `•	Estado: Ejecutado`
  - Línea en blanco de separación.
  - Lista de actividades separadas por viñetas (`•	` con tabulación), donde cada tarea inicia con un **subtítulo corto seguido de dos puntos** (`[Subtítulo corto]: [Descripción sintética con métricas y logros técnicos clave]`).
  - **Límite de extensión estricto:** El conjunto total de tareas del objeto debe tener un **máximo de 200 palabras**, priorizando concisión, métricas reales y rigor técnico.
  - Formato exacto:
    ```text
    N. [Texto exacto del objeto del contrato]
    •	Estado: Ejecutado   

    •	[Subtítulo corto 1]: [Descripción sintética y concisa de la labor, métricas verificadas y herramientas empleadas].
    •	[Subtítulo corto 2]: [Descripción sintética y concisa de la labor, métricas verificadas y herramientas empleadas].
    •	[Subtítulo corto 3]: [Descripción sintética y concisa de la labor, métricas verificadas y herramientas empleadas].
    ```

- **Si el objeto NO tuvo actividades (No ejecutado):**
  - Encabezado con el número y texto exacto del objeto contractual.
  - Línea de estado con viñeta y tabulación: `•	Estado: No ejecutado en el presente mes.`
  - Línea de justificación técnica formal: `•	Justificación: [Redacción en español formal que justifique técnicamente el porqué no se ejecutó, en base a dependencias, importación de componentes o priorización de tareas. Evita justificaciones idénticas o repetitivas].`
  - Línea de mes previsto: `•	Mes previsto de cumplimiento: [Mes previsto, ej: Noviembre].`
  - Formato exacto:
    ```text
    N. [Texto exacto del objeto del contrato]
    •	Estado: No ejecutado en el presente mes.
    •	Justificación: [Redacción técnica formal de justificación].
    •	Mes previsto de cumplimiento: [Mes previsto].
    ```

> ⚠️ **Nota Normativa Institucional (FOR-10):** Ya no se genera el archivo Markdown de evidencias para Notion (`XX_reporte.md`). Según la disposición de la Dirección Financiera, las evidencias documentales de los objetos ejecutados se gestionan como **anexos técnicos independientes en PDF** mediante la skill `generar_anexos_informe`.

### 5. Escritura y Respaldo de los Archivos
- Calcula el prefijo numérico de dos dígitos del mes solicitado (ej. `09` para septiembre).
- Guarda el informe de texto plano en el repositorio local:
  `institucional/RSA-Bitacora-LLM-Milton/informes/YYYY/XX_informe.txt` (ej. `informes/2026/09_informe.txt`).
- Si la unidad de Google Drive institucional está disponible (`H:\Mi unidad\Informes RSA\Informes YYYY\XX [mes]\`), copia o respalda el archivo `XX_informe.txt` en dicho directorio.

### 6. Confirmación al Usuario y Siguiente Paso
- Presenta al usuario un resumen en el chat indicando:
  - La ruta del informe de texto plano generado localmente y su respaldo en Google Drive.
  - La tabla resumen de objetos ejecutados y no ejecutados en el mes.
  - La recomendación explícita de ejecutar a continuación la skill `generar_anexos_informe` para compilar los anexos técnicos en PDF correspondientes a los objetos que resultaron en estado **Ejecutado**.
