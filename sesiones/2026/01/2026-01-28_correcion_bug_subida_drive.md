---
fecha: 2026-01-28
temas: [drive, logging, bugfix]
entorno: [acelerografo]
---
# Actividad del 2026-01-28

**Hitos de la jornada:**
Análisis de un error reportado en producción que provocaba un "crash" (`TypeError`) en el script `gestor_archivos_acq.py` durante la subida a Google Drive. Se determinó que el error original era un `socket.timeout` por fallos de red, y el fallo del programa se debía a una llamada incompatible a `logger.error` en `subir_archivo.py` que esperaba 1 argumento, pero usaba la instancia de `StructuredLogger` que requería 2 argumentos de manera estricta.

**Decisiones y Cambios:**
- Análisis de la causa raíz de la excepción `socket.timeout` y `TypeError`.

**Scripts/Comandos relevantes:**
```bash
# Análisis de logs y traza de subir_archivo.py y gestor_archivos_acq.py
```
---
# Actividad del 2026-02-02

**Hitos de la jornada:**
Planificación y ejecución de las correcciones para la resiliencia en red y el sistema de logs. Se ajustó el sistema de logging para soportar ambas firmas de uso (estándar y estructurado) y se incrementó el timeout para estabilizar las subidas de Google Drive en entornos de producción.

**Decisiones y Cambios:**
- Modificación de `scripts/operation/structured_logger.py`: Se hizo opcional el argumento secundario en `error(self, operation, error=None)` para soportar compatibilidad con `logging` estándar y evitar futuras fallas por llamadas con 1 parámetro.
- Modificación de `scripts/operation/drive/subir_archivo.py`: Se estableció un `timeout=300` al objeto HTTP (httplib2) en el método `get_authenticated` para lidiar mejor con conexiones lentas.
- Creación del script `scripts/operation/dev-tests/drive/test_logger_compatibility.py` para simular y validar las llamadas de 1 y 2 parámetros en el logger estructurado.

**Scripts/Comandos relevantes:**
```bash
# Prueba para validar que el bug TypeError fue resuelto
python3 /home/rsa/git/montajes/acelerografo/scripts/operation/dev-tests/drive/test_logger_compatibility.py
```
---
