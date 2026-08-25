---
fecha: 2026-08-25
temas: [event-analyzer, correlador, influxdb, mseed]
entorno: [tig]
autor: Milton
---

# Actividad del 2026-08-25

**Hitos de la jornada:**
Se implementó y validó integralmente el plan de acción de 4 pasos derivado del ADR-016 para solucionar inconsistencias temporales y de resolución de trazas en el ecosistema sísmico RSA (Event Analyzer, InfluxDB y Correlador Regional).

En primer lugar, se segmentó la búsqueda de trazas MiniSEED en `reader.py` para aislar los datos de registro continuo en `/mseed/`. Posteriormente, se desacopló la búsqueda de estaciones en `app.py` pasando `stations=None` a `scan_event()`, permitiendo al sismólogo visualizar las trazas de toda la red de estaciones que respondieron a la extracción por broadcast, clarificando las métricas en la interfaz (`N det.` vs. estaciones resueltas).

En el Correlador Regional (`regional_event_correlator.py`), se unificó la generación de `req_id` / `event_id` a partir de `dt_min` (timestamp de la detección física más temprana) en lugar de la hora de procesamiento del servidor, validándose en producción tras una detección real multiestación. Finalmente, se desarrolló y ejecutó el script `scripts/db_sync/fix_event_ids.py` a través del contenedor `rsa-event-analyzer`, migrando exitosamente 165 eventos históricos en InfluxDB bucket `rsa_events` de forma transaccional y sin pérdida de datos.

**Decisiones y Cambios:**
- **Aislamiento de Registro Continuo**: Patrones glob restringidos a `*/events/*.mseed` sin recursión en `reader.py`.
- **Desacoplamiento Multiestación en UI**: Invocación de `scan_event(stations=None)` y diferenciación de estaciones detectoras vs. resueltas en `app.py`.
- **IDs Temporales Deterministas**: `req_id = f"corr-{dt_min.strftime('%Y%m%d-%H%M%S')}"` en `regional_event_correlator.py`.
- **Migración Retroactiva**: Creación y ejecución de `scripts/db_sync/fix_event_ids.py` actualizando 165 registros en InfluxDB.
- **Documentación**: Actualización de contextos técnicos (`event_analyzer_context.md`, `regional_event_correlator_context.md`), nuevo contexto `fix_event_ids_context.md`, actualización de ADR-016 y transición técnica.

**Scripts/Comandos relevantes:**
```bash
# Reconstrucción de contenedores tras modificaciones
docker compose up -d --build event-analyzer
docker compose up -d --build correlator

# Migración retroactiva InfluxDB a través del contenedor
docker exec -i rsa-event-analyzer python3 - --dry-run < scripts/db_sync/fix_event_ids.py
docker exec -i rsa-event-analyzer python3 - < scripts/db_sync/fix_event_ids.py
```
---
