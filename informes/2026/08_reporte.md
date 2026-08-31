# Reporte de evidencias - Agosto 2026

---

# Introducción

El presente documento corresponde al detalle de las actividades desarrolladas durante el mes de agosto de 2026 en el marco de las tareas de la "Red Sísmica del Austro (RSA)". Específicamente se aportan los medios de verificación de las tareas realizadas y su correspondencia con el objeto del contrato.

---

# Avances en los objetos del contrato

Durante el mes de agosto de 2026 se trabajó en los siguientes objetos del contrato:

🔲 Ensamblado, configuración e instalación de estaciones digitales con 6 componentes para la actualización y ampliación de la red de monitoreo sísmico, integrando arquitecturas IoT para la transmisión distribuida de datos. (objeto 1)

✅ Implementación de funcionalidades de autodetección de eventos mediante técnicas basadas en machine learning y acceso a datos en tiempo real.  (objeto 2)

🔲 Ensamblado, configuración e instalación de acelerógrafos de 3 componentes con integración a la red de digitalizadores de 6 componentes. (objeto 3)

🔲 Configuración, instalación y mantenimiento de los sistemas desarrollados para el monitoreo automatizado de variables estáticas en las presas propiedad de Elecaustro. (objeto 4)

🔲 Administración de la red de sensores distribuidos en la presa, garantizando la recolección y transmisión precisa de datos en tiempo real. (objeto 5)

✅ Implementación de una solución de gestión y preservación de datos para el manejo del almacenamiento en las estaciones de monitoreo.  (objeto 6)

✅ Implementación de un sistema de visualización y monitoreo de métricas operativas basado en protocolos de telemetría ligera.  (objeto 7)

---

## Implementación de funcionalidades de autodetección de eventos mediante técnicas basadas en machine learning y acceso a datos en tiempo real.

- Se optimizó y robusteció la arquitectura del correlador regional de eventos sísmicos (`regional_event_correlator.py`) en el ecosistema TIG-MQTT, unificando la determinación unívoca y determinista del identificador de evento (`event_id` / `req_id`) a partir de la marca de tiempo de la detección física más temprana (`dt_min`) en lugar del tiempo de cómputo del servidor, eliminando discrepancias temporales con las trazas MiniSEED y los registros en base de datos.
- Se implementó resiliencia ante contingencias de pérdida de conectividad y cortes prolongados de energía mediante la configuración de sesiones persistentes en el broker Mosquitto (`clean_session=False`, QoS 1 y `client_id` estático `rsa-correlator-primary`), garantizando la retención en broker y el posterior procesamiento ordenado de alertas sísmicas sin pérdida de eventos tras la recuperación operativa.
- Se configuró la ejecución exclusiva del servicio correlador en el servidor primario mediante perfiles de Docker Compose (`profiles: ["primary"]`), permitiendo que el servidor secundario actúe como réplica pasiva sin emitir órdenes broadcast redundantes a las estaciones de monitoreo.

---

## Implementación de una solución de gestión y preservación de datos para el manejo del almacenamiento en las estaciones de monitoreo.

- Se diseñó e implementó integralmente el protocolo de respaldo, recuperación y preservación de datos para el catálogo de eventos sísmicos en InfluxDB v2 (bucket `rsa_events`).
- Se desarrolló una solución de respaldo híbrido mediante scripts automatizados (`backup_events.sh` y `restore_events.sh`) que combina instantáneas binarias nativas del motor TSM con exportaciones tabulares en formato CSV generadas mediante consultas Flux con `pivot()`, incorporando sincronización y políticas estrictas de rotación a 7 días hacia Google Drive mediante `rclone`.
- Se automatizó la ejecución periódica mediante unidades de `systemd` (`rsa-backup-events.service` y `rsa-backup-events.timer` a las 02:00 UTC con persistencia) gestionadas por una herramienta portable de administración (`manage_backup_timer.sh`).
- Se reubicó de forma transparente y sin retransferencia de datos el repositorio en la nube hacia el directorio consolidado de estaciones (`drive/DIA/Datos Estaciones/RSA-Backups/influxdb/`) mediante operaciones *server-side move* con `rclone`.
- Se corrigió el aislamiento en el almacenamiento de trazas MiniSEED en `reader.py` para excluir registros continuos de `/mseed/`, y se ejecutó con éxito el script transaccional `fix_event_ids.py` para migrar 165 registros históricos en InfluxDB sin pérdida de información.

---

## Implementación de un sistema de visualización y monitoreo de métricas operativas basado en protocolos de telemetría ligera.

- Se optimizó y repotenció la aplicación web de análisis sismológico Event Analyzer en Streamlit, resolviendo la saturación de WebSockets mediante la integración de `plotly-resampler` y un micro-servidor asíncrono en Dash (puerto 8050), lo que redujo el payload inicial de más de 200 MB a menos de 2 MB y habilitó el remuestreo dinámico en alta resolución (100–200 Hz) en ventanas de zoom para el picado de fases P y S.
- Se integró InfluxDB como fuente única de verdad para la consulta del catálogo en milisegundos con selección por calendario (`st.date_input`), aplicando carga diferida (*lazy loading*) de trazas con ObsPy bajo demanda y habilitando un ciclo interactivo de clasificación sismológica (Confirmar/Descartar) con actualización optimista y persistencia en tiempo real vía MQTT con QoS 1.
- Se desacopló la visualización en la interfaz permitiendo la resolución global de formas de onda de toda la red (`scan_event(stations=None)`), diferenciando estaciones detectoras de resueltas.
- Se integró el panel de control de Node-RED (`flows.json`) para la emisión dual sincronizada de órdenes de extracción y metadatos, y se implementó el microservicio Docker `rsa-db-sync` junto al script `mqtt_notify.py` para la emisión estructurada de telemetría operativa del estado de los respaldos al tópico `rsa/seismic/smart/system/backup`.

---

# Conclusiones del mes

- Se consolidó la plataforma de análisis y clasificación sismológica mediante la optimización de renderizado con `plotly-resampler` en el Event Analyzer, la consulta en milisegundos sobre InfluxDB y la integración bidireccional con Node-RED y MQTT.
- Se garantizó la consistencia temporal y la resiliencia operativa de la red de autodetección al fijar identificadores deterministas basados en la física del sismo (`dt_min`) y habilitar sesiones persistentes en el correlador regional ante cortes de energía y arquitecturas multi-servidor.
- Se desplegó un protocolo robusto y automatizado de respaldo híbrido y recuperación de datos de InfluxDB hacia Google Drive con rotación a 7 días, telemetría MQTT y migración exitosa de los registros históricos.
