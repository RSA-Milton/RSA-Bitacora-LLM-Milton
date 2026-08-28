---
fecha: 2026-08-27
temas: [influxdb, backup, systemd, multi-servidor, mqtt, docker, rclone]
entorno: [tig]
autor: Milton
---

# Actividad del 2026-08-27

**Hitos de la jornada:**
Se implementó y validó la Subfase 5A completa del protocolo de respaldo y recuperación para el catálogo de eventos sísmicos en InfluxDB v2 (`rsa_events`). En primer lugar (5A.1), se creó el microservicio Docker `rsa-db-sync` para aislar las dependencias de notificación MQTT y utilidades sin requerir entornos virtuales en el host.

Posteriormente (5A.2), se desarrollaron los scripts core `backup_events.sh` (snapshot binario TSM + exportación tabular a CSV con Flux `pivot()` + sincronización a Google Drive vía `rclone` + rotación estricta de 7 días) y `restore_events.sh` (restauración destructiva interactiva). La prueba en `rsa-server` respaldó **415 eventos** y subió los archivos a Google Drive en solo 10 segundos, emitiendo telemetría exitosa al tópico `rsa/seismic/smart/system/backup` con QoS 1.

Finalmente (5A.3), se diseñó e implementó un sistema de automatización portable con `systemd` (`manage_backup_timer.sh`, `rsa-backup-events.service.template` y `rsa-backup-events.timer.template`) que resuelve dinámicamente el usuario (`SUDO_USER`), grupo y rutas absolutas sin valores fijos en el repositorio. El timer se programó a las 02:00 UTC con persistencia (`Persistent=true`) y se validó mediante disparo sincrónico con código de salida exitoso en 11 segundos.

**Decisiones y Cambios:**
- Se adoptó el contenedor ligero `rsa-db-sync` (`python:3.11-slim`) bajo demanda en `services/docker-unified/docker-compose.yml`.
- Se implementó `scripts/db_sync/backup_events.sh` con sanitización estricta de variables `.env` y manejo de señales (`trap cleanup EXIT`).
- Se implementó `scripts/db_sync/restore_events.sh` con confirmación destructiva explícita (`SI`) y validación de conteo post-restauración.
- Se implementó `scripts/db_sync/mqtt_notify.py` para publicación estructurada de telemetría de respaldo con QoS 1.
- Se implementó el gestor `services/systemd/manage_backup_timer.sh` con soporte para `--install`, `--uninstall`, `--status`, `--run-now` y `--logs`.

**Scripts/Comandos relevantes:**
```bash
# Respaldo manual de prueba
bash scripts/db_sync/backup_events.sh

# Instalación y validación de automatización systemd
sudo ./services/systemd/manage_backup_timer.sh --install
./services/systemd/manage_backup_timer.sh --status
sudo ./services/systemd/manage_backup_timer.sh --run-now
```

---

# Actividad del 2026-08-28

**Hitos de la jornada:**
Se implementó y validó la Subfase 5B de arquitectura multi-servidor y resiliencia ante cortes de energía prolongados (escenario R4). Se configuró el servicio `correlator` con el perfil `profiles: ["primary"]` en Docker Compose para restringir su ejecución exclusiva a `rsa-server` (servidor primario) y permitir que `home-server` opere como espejo pasivo sin emitir órdenes broadcast duplicadas a las estaciones.

Se refactorizó `regional_event_correlator.py` para utilizar sesión persistente en Mosquitto (`clean_session=False`) y `client_id` estático (`rsa-correlator-primary`). Se realizó una prueba real de resiliencia apagando el correlador durante 6 minutos mientras las estaciones emitían alertas reales (`DEV0`, `CHA2`); al volver a encender el contenedor, Mosquitto entregó instantáneamente en ráfaga todos los mensajes encolados, validando que ninguna detección sísmica se pierde ante fallos de energía.

Para concluir la fase, se actualizó de forma integral el `README.md` del repositorio con una guía universal de despliegue paso a paso, se generaron 4 nuevos contextos técnicos (`docs/context/`), se actualizaron 2 existentes y se extrajo el **ADR-017** formalizando el protocolo de respaldo híbrido y la arquitectura multi-servidor.

**Decisiones y Cambios:**
- Se configuró `profiles: ["primary"]` y variable `RSA_CORRELATOR_CLIENT_ID` en `services/docker-unified/docker-compose.yml`.
- Se modificó la inicialización MQTT en `scripts/correlator/regional_event_correlator.py` con `clean_session=False`.
- Se documentaron las variables `RSA_SERVER_ROLE` y `RSA_CORRELATOR_CLIENT_ID` en `services/docker-unified/.env.example`.
- Se actualizó el `README.md` con arquitectura completa, tabla de roles y procedimientos de operación.
- Se crearon contextos técnicos para `backup_events.sh`, `restore_events.sh`, `mqtt_notify.py` y `manage_backup_timer.sh`.
- Se extrajo el `ADR-017: Protocolo de Respaldo Híbrido de InfluxDB, Automatización Portable y Arquitectura Multi-Servidor con Sesión Persistente MQTT`.

**Scripts/Comandos relevantes:**
```bash
# Reconstrucción e inicio del correlador con perfil primary
docker compose --profile primary up -d --build correlator

# Prueba de sesión persistente (con correlador apagado)
docker compose --profile primary stop correlator
docker compose run --rm db-sync -c "
import json, paho.mqtt.client as mqtt, os, time
client = mqtt.Client(client_id='test-sim')
if os.getenv('MQTT_USERNAME'): client.username_pw_set(os.getenv('MQTT_USERNAME'), os.getenv('MQTT_PASSWORD'))
client.connect(os.getenv('MQTT_BROKER', 'localhost'), int(os.getenv('MQTT_PORT', 1883)))
client.loop_start()
payload = json.dumps({'station': 'SIM01', 'timestamp_utc': '2026-08-28T16:30:00Z', 'status': 'detected'})
msg = client.publish('rsa/seismic/smart/SIM01/events/detected', payload, qos=1)
msg.wait_for_publish(timeout=5)
client.loop_stop()
client.disconnect()
"
docker compose --profile primary start correlator
docker compose --profile primary logs --tail=25 correlator
```
