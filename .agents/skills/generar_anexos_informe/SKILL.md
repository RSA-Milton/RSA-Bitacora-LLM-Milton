---
name: generar_anexos_informe
description: "Genera los anexos técnicos independientes en formato HTML y PDF por cada objeto contractual ejecutado en el mes según la directriz institucional FOR-10."
---

# Skill: Generación de Anexos Técnicos de Evidencias (FOR-10)

**Descripción de Activación:** Ejecuta este flujo cuando el usuario solicite la generación o compilación de los anexos técnicos del mes (ej. *"genera los anexos de septiembre de 2026"*, *"genera los anexos técnicos del informe de [mes] [año]"*, *"compila las evidencias de [mes]"*).

**Objetivo:** Generar reportes técnicos ejecutivos periciales independientes (en HTML y compilados a PDF) por cada objeto del contrato ejecutado durante el mes reportado, bajo el formato normativo **FOR-10** de la Dirección Financiera de la Universidad de Cuenca, guardando copia HTML versionada en el repositorio local y compilando los PDFs probatorios en Google Drive.

---

## ⚙️ Variables de Configuración por Defecto

Estas variables rigen la parametrización institucional y contractual, y pueden sobreescribirse si el contrato, profesional o rutas cambian en el futuro:

```text
PROFESIONAL_NOMBRE = "Ing. Milton Rodrigo Muñoz Calle"
NUMERO_CONTRATO = "CO-01056-2026"
ENTIDAD_CONTRATANTE = "Universidad de Cuenca"
DEPENDENCIA = "Departamento de Ingeniería Civil | Red Sísmica del Austro (RSA)"

RUTA_LOCAL_HTML = "institucional/RSA-Bitacora-LLM-Milton/informes/YYYY/anexos/"
RUTA_DRIVE_EVIDENCIAS = "H:\Mi unidad\Informes RSA\Evidencias\YYYY\MM [NombreMes]\"
PLANTILLA_HTML = "templates/anexo_template.html"

# Rutas de ejecutables headless para exportacion PDF en Windows
EJECUTABLE_EDGE = "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe"
EJECUTABLE_EDGE_FALLBACK = "C:\Program Files\Microsoft\Edge\Application\msedge.exe"
EJECUTABLE_CHROME = "C:\Program Files\Google\Chrome\Application\chrome.exe"
```

### Objetos del Contrato por Defecto

La clasificación de los entregables se rige por los siguientes objetos contractuales predeterminados:

1. **Ensamblado, configuración e instalación de estaciones digitales con 6 componentes** para la actualización y ampliación de la red de monitoreo sísmico, integrando arquitecturas IoT para la transmisión distribuida de datos.
2. **Implementación de funcionalidades de autodetección de eventos** mediante técnicas basadas en machine learning y acceso a datos en tiempo real.
3. **Ensamblado, configuración e instalación de acelerógrafos de 3 componentes** con integración a la red de digitalizadores de 6 componentes.
4. **Configuración, instalación y mantenimiento de los sistemas desarrollados** para el monitoreo automatizado de variables estáticas en las presas propiedad de Elecaustro.
5. **Administración de la red de sensores distribuidos en la presa**, garantizando la recolección y transmisión precisa de datos en tiempo real.
6. **Implementación de una solución de gestión y preservación de datos** para el manejo del almacenamiento en las estaciones de monitoreo.
7. **Implementación de un sistema de visualización y monitoreo de métricas operativas** basado en protocolos de telemetría ligera.

---

## 🛡️ Reglas Institucionales Críticas de Formato y Estilo

1. **Denominación Institucional Estricta:**
   - Debe figurar **únicamente**: `Universidad de Cuenca` y `Departamento de Ingeniería Civil | Red Sísmica del Austro (RSA)`.
   - **PROHIBIDO:** Incluir `"Facultad de Ingeniería"` o `"Dirección Financiera"` en el membrete institucional.
2. **Restricción de Términos PPP y Gobernanza de Labores:**
   - **PROHIBIDO:** Mencionar *"prácticas preprofesionales"*, *"pasantías"*, *"pasantes"*, *"cargas horarias"* (horas de dedicación, semanas) o nomenclatura de *"checkpoints"* (CP X.X). La planificación y especificación técnica forman parte de las labores contractuales del profesional, pero la administración de prácticas preprofesionales no lo es. Todo entregable debe redactarse estrictamente como **planificación técnica, especificación de ingeniería y desarrollo de sistemas**.
3. **Extensión Acotada y Control de Páginas:**
   - Cada anexo debe estructurarse para tener **exactamente de 2 a 3 páginas** (formato A4 vertical). Esto optimiza la impresión en ventanilla y previene el rechazo por saturación en el sistema Quipux.
4. **Eliminación Absoluta de Encabezados/Pies de Navegador:**
   - En el archivo HTML, la etiqueta `<title></title>` debe permanecer estrictamente vacía.
   - En la invocación del comando de impresión headless, es **obligatorio** el flag `--no-pdf-header-footer` para suprimir la impresión de la fecha, hora, título y URLs locales (ej. `file:///...`).
5. **Filtrado Exclusivo de Objetos:**
   - **Únicamente generan anexo los objetos marcados como `Ejecutado`** en el informe mensual correspondiente (`XX_informe.txt`). Los objetos `No ejecutado` no generan documento probatorio.
6. **Numeración Secuencial:**
   - Los anexos se numeran secuencialmente a partir del 1 (ej. *Anexo 1*, *Anexo 2*, *Anexo 3*, etc.), en el orden en que aparecen los objetos ejecutados en el contrato.
7. **Prohibición de Notación LaTeX (`$...$`):**
   - El documento HTML es interpretado directamente por un motor de navegador web offline sin bibliotecas KaTeX o MathJax. **Está terminantemente prohibido usar delimitadores de dólar `$ ... $` o sintaxis LaTeX** (ej. `$O(M \cdot N)$`, `$a_x$`, `$\mu\epsilon$`, `$\le$`). Toda notación matemática o técnica debe escribirse con HTML y Unicode nativo: `O(M·N)`, `O(N)`, `a<sub>x</sub> ≈ 0.64 g`, `µε`, `°C`, `≤`, `≥`, `Δt`, `m/s²`.
8. **Espacio para Firma Electrónica Digital:**
   - En la última página, la caja de firma (`.sign-box`) debe contar con un espacio vertical libre superior de al menos **50 a 60 px** (`margin: 55px auto 4px auto`) por encima de la línea divisoria para permitir estampar con holgura el sello visual del certificado de firma digital (FirmaEC, Quipux, Adobe Acrobat) sin solapar textos ni tablas precedentes.

---

## 📋 Pasos de Ejecución

### 1. Extracción del Período y Triaje de Objetos Ejecutados
- Identifica el **mes** (con prefijo de dos dígitos `XX`, ej. `09`) y el **año** (`YYYY`, ej. `2026`).
- Lee el archivo `institucional/RSA-Bitacora-LLM-Milton/informes/YYYY/XX_informe.txt`.
- Identifica los objetos que se encuentran en estado `Estado: Ejecutado` (admitiendo prefijos como `•\tEstado: Ejecutado`, `• Estado: Ejecutado` o `- Estado: Ejecutado`).
- Asigna la numeración secuencial de anexos:
  - Primer objeto ejecutado &rarr; **Anexo 1**
  - Segundo objeto ejecutado &rarr; **Anexo 2**
  - Tercer objeto ejecutado &rarr; **Anexo 3**, etc.

### 2. Recopilación de Evidencias Técnicas Profundas
Por cada objeto ejecutado, consulta las fuentes primarias del ecosistema técnico para extraer datos cuantificables, diagramas y fragmentos de configuración:
- Bitácoras de sesión en `institucional/RSA-Bitacora-LLM-Milton/sesiones/YYYY/XX/`.
- Repositorios de proyectos (`RSA-Acelerografo`, `RSA-Intern-TIG-MQTT`, `RSA-Metodologias`, `RSA-PPP-Toolkit`).
- Documentos de arquitectura en `docs/blueprints/`, notas de transición en `docs/progress/` y decisiones en `docs/analysis/` o ADRs.
- Extrae: métricas cuantitativas reales (latencias, consumos de RAM, recuentos de puntos de datos, porcentajes de optimización), tablas de pruebas, códigos representativos y esquemas de conexión.

### 3. Redacción del Documento HTML por Anexo
Utiliza como base la plantilla `templates/anexo_template.html`:
- Sustituye las variables de metadatos:
  - `{{ENTIDAD_CONTRATANTE}}`: `Universidad de Cuenca`
  - `{{DEPENDENCIA}}`: `Departamento de Ingeniería Civil | Red Sísmica del Austro (RSA)`
  - `{{PROFESIONAL_NOMBRE}}`: `Ing. Milton Rodrigo Muñoz Calle`
  - `{{NUMERO_CONTRATO}}`: `CO-01056-2026`
  - `{{PERIODO_REPORTADO}}`: Fechas del 01 al fin de mes (ej. `01/09/2026 – 30/09/2026`).
  - `{{CODIGO_DOCUMENTAL}}`: `RSA-EVI-YYYY-XX-ANX0N` (ej. `RSA-EVI-2026-09-ANX01`).
- Estructura el cuerpo técnico en las secciones normativas:
  1. **Objeto Contractual y Alcance Técnico Pericial**: Justificación contractual formal.
  2. **Diagnóstico Técnico, Arquitectura y Estado Inicial**: Contexto del problema y solución.
  3. **Desarrollo Técnico, Implementación y Métricas de Validación**: Tablas `data-table`, bloques `code-block` y cajas de alerta técnica `callout`.
  4. **Especificaciones Técnicas y Criterios Cuantitativos de Conformidad**: Criterios de aceptación comprobables y cuantitativos.
  5. **Certificación Pericial de Responsabilidad Técnica**: Bloque formal de cierre con firma del profesional.
- Inserta los saltos de página `<div class="page-break"></div>` para distribuir el contenido en exactamente 2 o 3 páginas.

### 4. Guardado Local del Archivo HTML
- Crea el directorio si no existe: `institucional/RSA-Bitacora-LLM-Milton/informes/YYYY/anexos/`.
- Guarda el archivo HTML en:
  `institucional/RSA-Bitacora-LLM-Milton/informes/YYYY/anexos/XX_anexo_N.html`
  (ej. `informes/2026/anexos/09_anexo_1.html`).

### 5. Compilación del Documento en PDF hacia Google Drive
- Define el directorio de destino en Google Drive:
  `H:\Mi unidad\Informes RSA\Evidencias\YYYY\XX [NombreMes]\`
  (ej. `H:\Mi unidad\Informes RSA\Evidencias\2026\09 Septiembre\`). Créalo si no existe.
- Nombra el archivo PDF resultante:
  `Anexo N - [Título Descriptivo Corto y Formal].pdf`
- Detecta el binario del navegador disponible en el sistema operativo (Edge o Chrome).
- Ejecuta la compilación silenciosa en PowerShell:
  ```powershell
  Start-Process -FilePath $edge -ArgumentList "--headless --disable-gpu --run-all-compositor-stages-before-draw --no-pdf-header-footer --print-to-pdf=`"$pdfPath`" `"$htmlPath`"" -Wait
  ```
- Comprueba que el PDF generado exista y verifica que el conteo de páginas sea exactamente de 2 o 3 páginas.

### 6. Confirmación al Usuario
Presenta un reporte ejecutivo en el chat con:
- La lista de anexos generados con su número, objeto contractual vinculado, título y conteo de páginas.
- La ruta local de los archivos HTML versionados en Git (`informes/YYYY/anexos/`).
- La ruta en Google Drive de los archivos PDF probatorios listos para Quipux e impresión en ventanilla.
