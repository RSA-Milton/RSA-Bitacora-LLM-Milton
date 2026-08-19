---
fecha: 2026-08-19
temas: [influxdb, event-analyzer, nodered, mqtt]
entorno: [tig]
autor: Milton
---

# Actividad del 2026-08-19

**Hitos de la jornada:**
Se completó la implementación y validación de la Fase 4 del plan de integración de eventos sísmicos en el stack TIG-MQTT. Se transformó el Event Analyzer en una herramienta desacoplada cuyo catálogo y fechas se consultan en milisegundos directamente desde InfluxDB (`rsa_events`), reduciendo drásticamente el consumo de I/O y eliminando los escaneos recursivos sobre el almacenamiento de Google Drive montado por FUSE. Se integró un ciclo cerrado de clasificación sismológica interactiva (Confirmar / Descartar) con actualización optimista inmediata en Streamlit y persistencia centralizada mediante publicaciones MQTT con QoS 1.

Se resolvieron problemas críticos de consistencia temporal en Telegraf: se configuró `timestamp_path = "timestamp_utc"` en el parser `json_v2` para evitar que las actualizaciones de clasificación sobrescribieran la fecha del sismo con la hora del clic. Asimismo, se adaptó el módulo `MseedReader` con el método `scan_event()` para la resolución puntual de trazas (*Lazy Loading*) y normalización de variantes de códigos de estación (`CHA2` $\leftrightarrow$ `CHA02`, `DEV0` $\leftrightarrow$ `DEV00`).

Finalmente, se integró el panel de control de Node-RED (`flows.json`) para que, al disparar una extracción manual de eventos, emita simultáneamente la orden `cmd/extract_event` a los acelerógrafos y el registro de metadatos al tópico central `rsa/seismic/smart/events/metadata` con `event_type: "manual"` y `source: "nodered"`. Se generaron los documentos de contexto técnico, el ADR-015 y la transición técnica del proyecto.

**Decisiones y Cambios:**
- **InfluxDB como Fuente Única de Verdad para el Catálogo:** El calendario y lista de eventos se leen exclusivamente desde InfluxDB (`get_recorded_dates` y `get_events_by_date`). El botón "Recargar Catálogo" invalida la caché Flux sin tocar disco ni Google Drive.
- **Lazy Loading Dirigido con ObsPy:** Google Drive se consulta únicamente bajo demanda en `reader.scan_event()` cuando el operador selecciona un evento para graficar sus trazas.
- **Persistencia Temporal Exacta en Telegraf:** Inclusión de `timestamp_path = "timestamp_utc"` y `timestamp_format = "2006-01-02T15:04:05.000Z07:00"` en `telegraf.conf` para evitar que los eventos cambien de fecha u hora tras ser clasificados.
- **Selector Inmutable con `format_func`:** Adaptación del menú desplegable en Streamlit para evitar que la aplicación pierda la selección del evento al actualizar los badges visuales (`🤖`, `👤`, `✅`, `❌`).
- **Emisión Dual en Node-RED:** Publicación en paralelo de la orden de extracción hacia estaciones y los metadatos hacia InfluxDB vía Telegraf.

**Scripts/Comandos relevantes:**
```bash
# Reconstruir y reiniciar servicios modificados en el stack unificado
cd ~/git/rsa/RSA-Intern-TIG-MQTT/services/docker-unified
docker compose up -d --build event-analyzer
docker compose restart telegraf

# Actualizar flujos en contenedor Node-RED
docker cp ~/git/rsa/RSA-Intern-TIG-MQTT/services/node-red/flows.json rsa-nodered:/data/flows.json
docker restart rsa-nodered

# Consultar eventos registrados en InfluxDB
docker exec rsa-influxdb influx query 'from(bucket: "rsa_events") |> range(start: 0) |> filter(fn: (r) => r._measurement == "seismic_event")'
```
---
