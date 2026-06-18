---
fecha: 2026-06-17
temas: [ring-buffer, mqtt, mseed, testing, logging]
entorno: [acelerografo]
autor: Milton
---
# Actividad del 2026-06-17

**Hitos de la jornada:**
Se implementó y estabilizó la Fase 4 del sistema Ring Buffer, que integra la extracción bajo demanda solicitada vía MQTT con el almacén rotativo en disco. Ahora, cuando llega un comando `extract_event` al coordinador, el módulo `event_extractor.py` consulta primero en el `RingBufferStore` si el rango de tiempo solicitado está disponible. En caso afirmativo, extrae las tramas crudas, genera un archivo binario temporal y lo convierte a miniSEED. Si el rango de tiempo no está cubierto, realiza un fallback automático hacia el almacenamiento horario tradicional de archivos `.mseed`.

Además, se completaron los requerimientos de la Fase 5 integrando los métodos de logging de streaming (`ring_write`, `ring_rotate`, `ring_cleanup`, `ring_query`, `pipe_read`, `pipe_error`) en el log estructurado de producción y añadiendo la sección de configuración de `streaming` a la plantilla oficial `configuracion_dispositivo.json.template`.

**Decisiones y Cambios:**
- **Extractor de Eventos Dual**: Modificación en `event_extractor.py` para consultar en `RingBufferStore` de forma concurrente y no bloqueante. Las tramas obtenidas se guardan con un nombre que emula el patrón `{CODIGO}_{YYMMDD}-{HHMMSS}.dat` para permitir que el convertidor `binary_to_mseed.py` extraiga la fecha del nombre si la variable `USAR_FECHA_FILENAME` está en `True` (mitigando el bug del dsPIC en producción).
- **Trazabilidad en MQTT**: Adición del campo `"source"` con valores `"ring_buffer"` o `"mseed_archive"` a la respuesta enviada de vuelta por el coordinador.
- **Suite de Pruebas Automatizado**: Rediseño de `test_event_extractor.py` como un suite de pruebas unitarias que simula mediante mocks las llamadas a subprocesos y valida aisladamente la extracción de ring buffer, el fallback por rango y el comportamiento con streaming inactivo.
- **Log de Streaming**: Extensión de `structured_logger.py` con métodos dedicados a operaciones de ring buffer y pipe para auditoría.
- **Plantilla de Configuración**: Modificación en `configuracion_dispositivo.json.template` para incluir parámetros por defecto del buffer (directorio, duración, tamaño máximo).

**Scripts/Comandos relevantes:**
Para verificar el funcionamiento del extractor y toda la suite simulada de fallback y configuración:
```bash
cd /home/rsa/git/montajes/acelerografo-DEV00
python3 scripts/operation/mqtt/test_event_extractor.py
```
---
