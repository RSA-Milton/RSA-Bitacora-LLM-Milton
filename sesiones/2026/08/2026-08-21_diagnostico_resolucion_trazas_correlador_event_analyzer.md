---
fecha: 2026-08-21
temas: [event-analyzer, correlador, influxdb, miniseed]
entorno: [tig]
autor: Milton
---

# Actividad del 2026-08-21

**Hitos de la jornada:**
Se realizó un diagnóstico exhaustivo sobre el flujo de detección, correlación y resolución de trazas MiniSEED en el stack `RSA-Intern-TIG-MQTT`. A partir del análisis de datos reales y consultas en InfluxDB, se identificaron cuatro problemas arquitectónicos y de consistencia temporal: 1) el filtrado restrictivo de estaciones en `app.py` que limitaba la visualización a las estaciones detectoras en vez de todas las que respondieron al broadcast; 2) la contaminación por trazas duplicadas provenientes de subdirectorios `/mseed/` (datos de registro continuo por estación); 3) la discrepancia temporal en la generación del `event_id` del correlador (uso de `datetime.now()` en lugar del timestamp de la detección más temprana `dt_min`), generando desfases de varios segundos respecto a `_time` en InfluxDB y al nombre del archivo `.mseed` (`dt_min - 60s`); y 4) la inconsistencia histórica de los `event_id` ya almacenados en InfluxDB.

Se elaboró y formalizó el **ADR-016** (*Resolución Global de Trazas MiniSEED, Exclusión de Registro Continuo y Determinación Temporal del event_id*) en el repositorio metodológico institucional y en la raíz del proyecto. Asimismo, se redactó un plan de implementación detallado en 4 fases secuenciales con checkpoints de validación y comandos manuales de verificación antes de avanzar a la Fase 5 (Protocolo de Respaldo y Recuperación).

**Decisiones y Cambios:**
- **Resolución Global de Trazas Desacoplada:** Modificación planificada en `app.py` para invocar `reader.scan_event(stations=None)`, permitiendo cargar las formas de onda de toda la red de estaciones para cualquier evento.
- **Aislamiento Estricto de Rutas `/events/`:** Restricción del patrón glob en `reader.py` a `*/events/*.mseed` para evitar la lectura accidental de archivos de registro continuo ubicados en `/mseed/`.
- **Determinación Temporal Unívoca del `event_id`:** Actualización planificada en `regional_event_correlator.py` para construir el identificador con `dt_min` (`corr-{dt_min.strftime('%Y%m%d-%H%M%S')}`), garantizando correspondencia directa con `_time` y los archivos en disco (`dt_min - 60s`).
- **Diferenciación de Métricas en UI:** Claridad operativa en Streamlit distinguiendo "Estaciones Detectoras" (`(N det.)` / `n_stations`) de "Estaciones Resueltas" (`M/M` en disco).
- **Migración Retroactiva Planificada:** Definición de un script puntual (`fix_event_ids.py`) para reescribir los registros históricos en InfluxDB previo respaldo.

**Scripts/Comandos relevantes:**
```bash
# Consultar eventos registrados en InfluxDB con desglose temporal
docker exec -i rsa-influxdb influx query '
from(bucket: "rsa_events")
  |> range(start: 2026-08-19T00:00:00Z, stop: 2026-08-19T23:59:59Z)
  |> filter(fn: (r) => r._measurement == "seismic_event")
  |> pivot(rowKey: ["_time", "event_id"], columnKey: ["_field"], valueColumn: "_value")
  |> keep(columns: ["_time", "event_id", "event_type", "source", "stations", "n_stations", "duration_s"])
  |> sort(columns: ["_time"])
' --org rsa

# Verificar archivos de eventos en disco excluyendo registro continuo
ls /home/rsa/datos_estaciones_drive/*/events/*20260819*.mseed
```
---
