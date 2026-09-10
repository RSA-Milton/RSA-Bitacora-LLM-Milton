---
fecha: 2026-09-10
temas: [grafana, alertas, mqtt, telegraf, observabilidad]
entorno: [tig, acelerografo]
autor: Milton
---

# Actividad del 2026-09-09 y 2026-09-10

**Hitos de la jornada:**

Durante estas dos jornadas se diseñó, estructuró y formalizó la arquitectura de visualización centralizada de alertas y telemetría especializada para la Red Sísmica del Austro (RSA), abordando de forma integral tanto el servidor central (`RSA-Intern-TIG-MQTT`) como las estaciones acelerográficas de campo (`acelerografo-DEV00`). El objetivo primordial consistió en erradicar la ceguera operativa del monitor de guardia 24/7 (`SeismicMonitor`), el cual operaba bajo un esquema binario de conexión (`online`/`offline`) que mantenía filas en verde aun cuando las estaciones sufrían paradas silenciosas de adquisición, incoherencias físicas en el sensor acelerométrico o acumulación de archivos sin sincronizar en Google Drive.

Se consolidó un modelo jerárquico de alertas fundamentado en la criticidad de los datos sismológicos: nivel **Rojo (Crítico - pérdida o corrupción irreparable de datos)** para caídas de enlace (`offline`), adquisición congelada en el Ring Buffer (`stale_data`), aceleraciones anómalas del sensor triaxial y disco lleno; nivel **Amarillo (Advertencia - atención requerida sin pérdida inmediata)** para demoras en Google Drive, estrés térmico y disco bajo; y nivel **Verde (Nominal)** para operación óptima. En `SeismicMonitor`, la tabla se simplificó a 3 columnas (`Estación`, `Conectividad`, `Diagnóstico Principal`) con diagnóstico en una sola palabra (`OK`, `Offline`, `Adquisicion`, `Sensor`, `Drive`, `Disco`, `Hardware`) y coloreado de fila completa (`applyToRow: true`), transfiriendo el detalle analítico al dashboard `Health` mediante enlaces interactivos.

Para las estaciones acelerográficas, se estableció la unificación de la cadencia de evaluación y publicación MQTT a **5 minutos (300 s)**, acoplada directamente al ciclo habitual de `telemetry/health` en `mqtt_coordinator.py`. Esto previene la saturación del canal de comunicaciones en campo mediante publicaciones con QoS 1 y `retain = true`. Asimismo, se formalizó el protocolo de remediabilidad operativa y se diseñó el comando de contingencia `cmd/stop_acquisition_safety` para detener la adquisición remota si el sensor físico reporta daño irreversible, protegiendo el almacenamiento local y evitando contaminar el Correlador Regional Central con eventos espurios.

**Decisiones y Cambios:**

- **Diagnóstico Técnico Formal**: Redactado y actualizado en `montajes/server-ubuntu/rsa/RSA-Intern-TIG-MQTT/docs/analysis/2026-09-09_diagnostico_modelo_alertas_visualizacion_grafana.md`.
- **Cálculo de Salud Server-Side (Opción B)**: Adoptada la agregación jerárquica en Grafana/Flux en lugar de procesamiento en el edge, garantizando que el servidor detecte automáticamente el estado `offline` vía LWT.
- **Rediseño Minimalista de `SeismicMonitor`**: Eliminada la columna redundante de estado operativo; se adoptó la tabla de 3 columnas (`Estación`, `Conectividad`, `Diagnóstico Principal`) con diagnósticos de una palabra.
- **Enriquecimiento de `Health`**: Diseñadas tres nuevas secciones analíticas en `health.json`: Google Drive (pendientes y protegidos), Integridad del Acelerómetro (tabla triaxial $A_x, A_y, A_z$ y fuente de reloj), y Salud de Adquisición (semáforo Ring Buffer y curva de latencia `age_seconds`).
- **Cadencia Unificada a 5 Minutos**: Unificada la evaluación de Adquisición y Sensor a 300 s acoplada a `telemetry/health` en las estaciones.
- **Blueprints de Implementación**:
  - Servidor TIG: `montajes/server-ubuntu/rsa/RSA-Intern-TIG-MQTT/docs/blueprints/2026-09-10_plan_implementacion_alertas_visualizacion_grafana.md`.
  - Estaciones: `montajes/acelerografo-DEV00/docs/blueprints/2026-09-10_plan_implementacion_telemetria_sensor_adquisicion_drive.md`.
- **Transiciones Técnicas**: Generados los documentos de transición en ambos repositorios (`docs/progress/2026-09-10_contexto-agente.md`).

**Scripts/Comandos relevantes:**

```bash
# 1. Verificación de prueba de inyección MQTT para estado nominal (Servidor TIG)
docker exec -it rsa-mosquitto mosquitto_pub -h localhost -p 1883 \
  -t "rsa/seismic/smart/TEST/status/acquisition" \
  -m '{"status":"ok","age_seconds":1.2,"threshold_seconds":300,"last_frame_utc":"2026-09-10T16:00:00Z","station_id":"TEST","timestamp":"2026-09-10T16:00:01Z"}' -q 1 -r

# 2. Verificación de prueba de alerta roja por sensor anómalo
docker exec -it rsa-mosquitto mosquitto_pub -h localhost -p 1883 \
  -t "rsa/seismic/smart/TEST/status/sensor" \
  -m '{"status":"error","ax":-0.85,"ay":2.14,"az":0.10,"clock_source":"GPS","clock_error":"accelerometer_anomaly","reason":"accelerometer_anomaly","station_id":"TEST","timestamp":"2026-09-10T16:10:00Z"}' -q 1 -r

# 3. Monitoreo de sincronía de los 4 tópicos en estación de campo
mosquitto_sub -h 174.138.41.251 -t "rsa/seismic/smart/DEV0/#" -v
```
---
