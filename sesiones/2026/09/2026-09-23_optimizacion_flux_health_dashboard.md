---
fecha: 2026-09-23
temas: [grafana, flux, telemetria, tig]
entorno: [tig]
autor: Milton
---

# Actividad del 2026-09-23 (Consolidación y Optimización Flux de Health Dashboard v13)

**Hitos de la jornada:**

Durante esta jornada se llevó a cabo el análisis, depuración y consolidación definitiva de los cambios realizados en el dashboard analítico de diagnóstico por estación (**Health Dashboard**), avanzando desde la versión original v9 hasta la versión v13 estable en `services/grafana/provisioning/dashboards/health.json` del repositorio `RSA-Intern-TIG-MQTT`. Los cambios iniciales, explorados interactivamente en la interfaz web de Grafana para incorporar las nuevas fuentes de telemetría especializada (adquisición, sensor triaxial y Google Drive desplegados bajo los ADR-020 y ADR-021), presentaban errores críticos de consulta Flux y desajustes visuales que fueron sistemáticamente rectificados.

Entre las correcciones fundamentales, se erradicó un fallo HTTP 500 en InfluxDB causado por conversiones inválidas de cadenas hexadecimales (`int(v: "0xd0000")`) en las métricas de `throttled`, sustituyéndolas por mapeos discretos deterministas. Asimismo, se unificó la nomenclatura de etiquetas de filtrado a `station_id` en todas las consultas y se blindaron los paneles de métricas singulares (Stat) y series temporales agregando agrupaciones explícitas `group(columns: ["_measurement", "_field", "station_id"])`, impidiendo la fragmentación de paneles en múltiples tarjetas no deseadas por la coexistencia transitoria de tags legados (`station`, `host`).

En una segunda fase, se diseñó e implementó un patrón canónico en Flux para la deduplicación de transiciones de estado en tablas históricas. Dado que la función nativa `monitor.stateChanges()` no se encuentra disponible en Grafana, se formuló una técnica basada en codificación numérica de estados, cálculo de diferencias con `difference(columns: ["state_code"], keepFirst: true)` y filtrado de deltas no nulos. Este patrón se aplicó exitosamente en los paneles de Estado General (`id: 9`), Throttled (`id: 6`) e Historial de Sincronización Google Drive (`id: 29`), eliminando la redundancia de registros nominales idénticos cada 5 minutos y expandiendo la profundidad a 30 eventos significativos con ordenamiento `organize` de columnas (`Estado` y `Razón` al final). Finalmente, para el Historial de Integridad Triaxial y Reloj (`id: 25`), se preservó intencionalmente el muestreo continuo a 5 minutos a petición del operador para auditar de forma ininterrumpida las aceleraciones físicas en reposo ($Z, N, E$), manteniendo igualmente la capacidad ampliada de 30 filas y el alineamiento uniforme de columnas.

**Decisiones y Cambios:**

- **Mapeo Determinista Hexadecimal en Throttled**: Reemplazo de conversiones directas `int(v: r._value)` por condicionales `if/else` exhaustivos para valores de `vcgencmd` (`"0x0" -> 0`, `"0xd0000" -> 851968`, `"0x50005" -> 327685`, etc.), eliminando caídas HTTP 500 en InfluxDB v2.
- **Blindaje de Agrupación Flux contra Fragmentación de Paneles**: Adición de `|> group(columns: ["_measurement", "_field", "station_id"])` en Stat panels (`id: 21, 24, 27, 28`) y series de latencia (`id: 22`), forzando una serie única unificada por estación.
- **Patrón de Deduplicación de Transiciones en Tablas Históricas**: Implementación de mapeo discreto (`state_code`) + `difference()` + `filter(r.state_code != 0)` en paneles `id: 9` (Estados), `id: 6` (Throttled) e `id: 29` (Sincronización Drive). En el panel 29 se construyó un `state_code` ponderado combinando `status`, `reason`, `pending_mseed` y `failed_uploads_protected`, ignorando intencionalmente `free_disk_percent` para evitar falsos cambios de estado por variaciones marginales de disco.
- **Preservación de Muestreo Continuo en Panel de Integridad Triaxial (Panel 25)**: Mantenimiento del flujo continuo de telemetría cada 5 minutos en el Historial Triaxial y Reloj, permitiendo la trazabilidad ininterrumpida de las componentes $Z, N, E$ y la deriva de reloj sin filtrado por deltas.
- **Estandarización de Interfaz y Profundidad de Tablas**: Aumento de capacidad de 10 a 30 filas tanto en Historial de Sincronización Google Drive como en Historial de Integridad Triaxial, con reorganización de columnas mediante transformación Grafana `organize` (`indexByName`), ubicando `Estado` y `Diagnóstico/Razón` en las columnas finales de visualización.
- **Documentación y Trazabilidad**: Actualización del contexto técnico `docs/context/health_context.md`, generación de la nota de transición técnica `docs/progress/2026-09-23_contexto-agente.md` e indexación federada en `indice_tematico.md`.

**Scripts/Comandos relevantes:**

```flux
// Patrón canónico de deduplicación de transiciones de estado en Grafana / InfluxDB Flux
from(bucket: "telegraf")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r._measurement == "drive_watchdog" and (r.station_id == "${station}" or r.station == "${station}"))
  |> pivot(rowKey: ["_time"], columnKey: ["_field"], valueColumn: "_value")
  |> sort(columns: ["_time"], desc: false)
  |> map(fn: (r) => ({
      r with
      status_code: if r.status == "OK" then 1 else if r.status == "WARNING" then 2 else if r.status == "CRITICAL" then 3 else 0,
      reason_code: if r.reason == "nominal" then 1 else if r.reason == "backlog_warning" then 2 else if r.reason == "upload_failed" then 3 else if r.reason == "disk_warning" then 4 else if r.reason == "disk_critical" then 5 else 0
  }))
  |> map(fn: (r) => ({
      r with
      state_code: (r.status_code * 1000000) + (r.reason_code * 10000) + (int(v: r.pending_mseed) * 100) + int(v: r.failed_uploads_protected)
  }))
  |> difference(columns: ["state_code"], keepFirst: true)
  |> filter(fn: (r) => not exists r.state_code or r.state_code != 0)
  |> drop(columns: ["state_code", "status_code", "reason_code"])
  |> sort(columns: ["_time"], desc: true)
  |> limit(n: 30)
```
---
