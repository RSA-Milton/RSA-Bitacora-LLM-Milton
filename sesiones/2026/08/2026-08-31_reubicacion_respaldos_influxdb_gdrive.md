---
fecha: 2026-08-31
temas: [influxdb, backup, rclone, gdrive, docker]
entorno: [tig]
autor: Milton
---

# Actividad del 2026-08-31

**Hitos de la jornada:**
Se llevó a cabo la reubicación del directorio de almacenamiento de respaldos del catálogo de InfluxDB v2 (`rsa_events`) en Google Drive, trasladándolo desde la raíz (`drive/RSA-Backups/influxdb/`) hacia el directorio consolidado de las estaciones sísmicas (`drive/DIA/Datos Estaciones/RSA-Backups/influxdb/`). 

Los archivos históricos de respaldo (`.tar.gz` y `.csv`) fueron migrados de forma directa e instantánea mediante operaciones del lado del servidor (*server-side move*) con `rclone`, eliminando la necesidad de retransferir datos a través de la red y manteniendo la integridad de la retención histórica de 7 días.

A nivel de código y arquitectura, se actualizaron los scripts y plantillas de configuración del repositorio (`backup_events.sh`, `restore_events.sh`, `mqtt_notify.py`, `.env.example`) para adoptar la nueva ruta `"gdrive:DIA/Datos Estaciones/RSA-Backups/influxdb"`. Se garantizó la compatibilidad con espacios en la ruta mediante entrecomillado robusto en Bash y se parametrizó la emisión del campo `drive_path` en la telemetría MQTT con QoS 1. Finalmente, se actualizó la documentación en `README.md` y en los contextos técnicos de `docs/context/`, confirmando que los servicios principales en producción no requieren reinicio.

**Decisiones y Cambios:**
- Se migró el almacenamiento en la nube a `gdrive:DIA/Datos Estaciones/RSA-Backups/influxdb`.
- Se actualizó `services/docker-unified/.env.example` con la nueva ruta entrecomillada.
- Se actualizaron los fallbacks por defecto en `scripts/db_sync/backup_events.sh` y `scripts/db_sync/restore_events.sh`.
- Se configuró el envío explícito de `--drive-path "$RCLONE_DEST"` a `mqtt_notify.py` desde `backup_events.sh`.
- Se actualizó el valor por defecto de `--drive-path` en `scripts/db_sync/mqtt_notify.py`.
- Se actualizaron diagramas y referencias en `README.md`, `docker-compose-tig-mqtt_context.md`, `backup_events_context.md` y `mqtt_notify_context.md`.

**Scripts/Comandos relevantes:**
```bash
# Migración server-side en Google Drive con rclone
rclone move "gdrive:RSA-Backups/influxdb" "gdrive:DIA/Datos Estaciones/RSA-Backups/influxdb" --progress
rclone rmdirs "gdrive:RSA-Backups"

# Verificación de contenidos en el nuevo destino
rclone ls "gdrive:DIA/Datos Estaciones/RSA-Backups/influxdb/"

# Ejecución de prueba del respaldo en el host
bash scripts/db_sync/backup_events.sh

# Reconstrucción opcional de la imagen ligera de utilidades
docker compose build db-sync
```
