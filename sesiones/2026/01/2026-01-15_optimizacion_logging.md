---
fecha: 2026-01-15
temas: [logging, mseed]
entorno: [acelerografo]
---
# Actividad del 2026-01-15

**Hitos de la jornada:**
Se optimizó el sistema de logging para los scripts `gestor_archivos_acq.py` y `binary_to_mseed.py`. Se implementó una nueva librería reutilizable `structured_logger.py` que permite la rotación automática de logs, niveles de verbosidad configurables (DEBUG, INFO, SUMMARY) y un formato de mensajes estructurado con etiquetas (tags) para facilitar la lectura y filtrado de información.

**Decisiones y Cambios:**
- Se creó `scripts/operation/structured_logger.py` para estandarizar el registro de eventos y errores, utilizando `RotatingFileHandler` (tamaño máximo 5MB, 3 backups) para evitar el sobrecrecimiento de los archivos de log.
- Se modificó `gestor_archivos_acq.py` para utilizar el nuevo logger estructurado. Se agruparon mensajes repetitivos, como los correspondientes a "archivos ya subidos", en un único resumen global al final del proceso para reducir la redundancia en el archivo de logs.
- Se actualizaron los tags del logger en `gestor_archivos_acq.py` para operaciones de subida de archivos y control de espacio (`[INIT]`, `[UPLOAD_OK]`, `[DELETE_AGE]`, `[DELETE_SPACE]`, `[SUMMARY]`, etc.).
- Se extendió `StructuredLogger` para incluir funcionalidad específica de conversión de binario a miniSEED y se integró en `binary_to_mseed.py` mediante tags como `[CONVERT_START]`, `[CONVERT_OK]`, `[CONVERT_FAIL]`, `[READ_OK]`, `[DATA_WARNING]` y `[CONFIG_ERROR]`.
- Se reemplazaron todas las llamadas a `logging` no estructuradas heredadas en `binary_to_mseed.py` por los métodos estandarizados.
- El usuario actualizó manualmente el script `deploy.sh` para añadir la copia del nuevo archivo `structured_logger.py`.

**Scripts/Comandos relevantes:**
```python
# Ejemplo de inicialización de la nueva librería estructurada en binary_to_mseed.py
logger = obtener_logger_estructurado(dispositivo_id, log_directory, "mseed.log", verbosity=VERBOSITY_LEVEL)

# Uso de tags de conversión en binary_to_mseed.py
logger.convert_start("RC", binary_filename)
logger.data_warning(os.path.basename(archivo_binario), "trama_invalida", f"{h:02}:{m:02}:{s:02}")
logger.convert_ok(binary_filename, nombre_archivo_mseed, tiempo_lectura)
```
---
