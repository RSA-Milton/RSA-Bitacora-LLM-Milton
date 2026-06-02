---
fecha: 2026-02-13
temas: [grafana, influxdb, dashboards, flux]
entorno: [tig]
---
# Actividad del 2026-02-13

**Hitos de la jornada:**
Se diagnosticó un problema de "No Data" en los paneles de Grafana al intentar usar consultas por tags. Se identificó que Telegraf aún no había sido reiniciado para generar los tags indexados. Como solución temporal (Consulta de Emergencia), se implementó una consulta basada en el topic utilizando la función `drop()` para mantener los nombres de los campos limpios y evitar problemas visuales al cambiar de estación.

**Decisiones y Cambios:**
- Se proporcionó una consulta de "emergencia" usando filtros por `topic`.
- Se configuró el panel "Estado" usando `drop(columns: ["host", "topic", "_measurement"])`.
- Se indicó configurar "Text mode" a `Value` en Grafana para evitar cambios en el tamaño del texto.

**Scripts/Comandos relevantes:**
```flux
from(bucket: "telemetry")
  |> range(start: -5d) 
  |> filter(fn: (r) => r._measurement == "mqtt_consumer")
  |> filter(fn: (r) => r.topic == "rsa/seismic/smart/${station}/telemetry/state")
  |> filter(fn: (r) => r._field == "status")
  |> last()
  |> drop(columns: ["host", "topic", "_measurement", "_start", "_stop", "station_id", "data_type"])
```
---

# Actividad del 2026-02-18

**Hitos de la jornada:**
Se diagnosticó el cambio estructural en InfluxDB tras habilitar el *Topic Parsing* en Telegraf. Se detectó que el `_measurement` cambió de `mqtt_consumer` a `rsa`. Además, se explicó el funcionamiento de `aggregateWindow` y `yield` para optimizar el rendimiento de las consultas y se documentó el comportamiento de la provisión de Grafana cuando `allowUiUpdates` está activo.

**Decisiones y Cambios:**
- Se actualizó el nombre de la medición objetivo a `rsa`.
- Se verificó la configuración de Telegraf confirmando el uso de una medición personalizada (derivada del topic parsing).
- Se confirmó que Grafana prioriza la UI local sobre los archivos debido a `allowUiUpdates: true` en `dashboards.yml`.
- Se documentó el procedimiento de prueba para forzar la recarga desde el archivo (borrar el dashboard desde la UI y esperar el ciclo de `updateIntervalSeconds`).

**Scripts/Comandos relevantes:**
```flux
from(bucket: "telemetry")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["station_id"] == "CHA01")
  |> filter(fn: (r) => r["_field"] == "cpu_temp_c")
  |> filter(fn: (r) => r["_measurement"] == "rsa")
  |> filter(fn: (r) => r["data_type"] == "health")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> yield(name: "mean")
```
---

# Actividad del 2026-02-19

**Hitos de la jornada:**
Se reestructuró la arquitectura de los dashboards de Grafana hacia un modelo de "Hub + Detalle". El dashboard principal ahora actúa como un centro de monitoreo general (`seismic_monitor.json`), y se agregó un dashboard detallado (`health.json`) para profundizar en las métricas de hardware de cada estación de manera individual.

**Decisiones y Cambios:**
- Transición a arquitectura "Hub + Detalle".
- Renombrado de archivos exportados manualmente a `health.json` y `seismic_monitor.json` para eliminar los timestamps generados por Grafana y asegurar un aprovisionamiento limpio en `services/grafana/provisioning/dashboards/`.
- Generación del mensaje de commit documentando la separación de responsabilidades y la optimización de las consultas.
- Actualización final de la consulta global usando `group()` vacío para fusionar las tablas y `keep()` para seleccionar columnas relevantes.

**Scripts/Comandos relevantes:**
```flux
from(bucket: "telemetry")
  |> range(start: -7d)
  |> filter(fn: (r) => r._measurement == "rsa")
  |> filter(fn: (r) => r.data_type == "state")   
  |> filter(fn: (r) => r._field == "status")
  |> last()
  |> group() 
  |> keep(columns: ["station_id", "_value"]) 
  |> rename(columns: {station_id: "Estación", _value: "Status"})
```
---
