---
fecha: 2026-11-06
temas: [migracion, docker, backups, node-red]
entorno: [tig]
autor: Milton
---

# Actividad del 2026-11-06

**Hitos de la jornada:**
Se ejecutó la migración integral y puesta en producción del stack de monitoreo, telemetría y análisis sísmico desde el repositorio precedente `RSA-Intern-TIG-MQTT` hacia el repositorio consolidado institucional `RSA-Monitor-Sismico` en el servidor central Ubuntu (`server-ubuntu`). La intervención contempló la detención ordenada de los contenedores Docker en ejecución, la migración de archivos de configuración y variables de entorno (`.env`), la reconstrucción de imágenes y el despliegue del stack unificado garantizando la preservación total de los datos históricos en `/home/rsa/data` y trazas MiniSEED en `/home/rsa/datos_estaciones_drive`.

Tras el arranque del stack con perfil `primary`, se constató la operatividad plena de los microservicios: `rsa-influxdb` y `rsa-grafana` alcanzaron estado saludable (`healthy`), `rsa-telegraf` estableció conexión con sus 5 consumidores MQTT hacia el broker central (`174.138.41.251:1883`) y sus salidas InfluxDB duales (`telemetry` y `rsa_events`), y el servicio web `rsa-event-analyzer` quedó activo en el puerto 8501. Asimismo, se comprobó en vivo la reactividad del daemon `rsa-correlator`, el cual procesó de inmediato una detección sísmica real de la estación `DEV0` (Fase P, prob=0.9923) orquestando la extracción y recepción del archivo MiniSEED correspondiente (`DEV0_20261106_163847.mseed`).

Finalmente, se reconfiguró la automatización a nivel de sistema operativo para el respaldo diario del catálogo de eventos. Mediante `manage_backup_timer.sh --install`, se regeneraron y recargaron las unidades `/etc/systemd/system/rsa-backup-events.service` y `rsa-backup-events.timer`, vinculándolas estrictamente al nuevo repositorio. En el stack complementario de Node-RED se identificó y saneó una omisión de la variable `DATA_DIR` en su archivo `.env`, corrigiendo el bind mount a `/home/rsa/data/nodered -> /data` y autorizando el desacople/eliminación definitiva del repositorio histórico.

**Decisiones y Cambios:**
- Migración de variables de entorno `.env` de `docker-unified`, `node-red` y `correlator` hacia `RSA-Monitor-Sismico`.
- Parada controlada de contenedores anteriores para evitar colisiones de nombres y puertos (8086, 3000, 8501, 1880).
- Despliegue del stack unificado bajo perfil primario: `docker compose --profile primary up -d --build`.
- Certificación de recepción en tiempo real de alertas y confirmaciones MiniSEED en `rsa-correlator` con sesión persistente (`client_id=rsa-correlator-primary`).
- Reinstalación de unidades systemd de respaldo diario mediante `manage_backup_timer.sh --install`, actualizando `WorkingDirectory`, `ExecStart` y `Environment` a las rutas del nuevo repositorio.
- Saneamiento del entorno Node-RED (`services/node-red/.env`) agregando `DATA_DIR=/home/rsa/data` para asegurar la persistencia en el volumen host unificado.
- Dictamen favorable y protocolo de retiro/renombrado preventivo para `RSA-Intern-TIG-MQTT`.

**Scripts/Comandos relevantes:**
```bash
# 1. Copia de configuraciones al nuevo repositorio
cp RSA-Intern-TIG-MQTT/services/docker-unified/.env RSA-Monitor-Sismico/services/docker-unified/.env
cp RSA-Intern-TIG-MQTT/services/node-red/.env RSA-Monitor-Sismico/services/node-red/.env
cp RSA-Intern-TIG-MQTT/scripts/correlator/.env RSA-Monitor-Sismico/scripts/correlator/.env

# 2. Detención controlada del sistema anterior
cd ~/git/rsa/RSA-Intern-TIG-MQTT/services/docker-unified
docker compose --profile primary down

# 3. Construcción y levantamiento del nuevo stack
cd ~/git/rsa/RSA-Monitor-Sismico/services/docker-unified
docker compose --profile primary up -d --build

# 4. Reinstalación y validación del temporizador de respaldo diario en Systemd
cd ~/git/rsa/RSA-Monitor-Sismico/services/systemd
sudo ./manage_backup_timer.sh --install
./manage_backup_timer.sh --status

# 5. Corrección de persistencia en Node-RED
cd ~/git/rsa/RSA-Monitor-Sismico/services/node-red
echo "DATA_DIR=/home/rsa/data" >> .env
docker compose up -d
docker inspect rsa-nodered --format '{{ range .Mounts }}{{ .Source }} -> {{ .Destination }}{{ "\n" }}{{ end }}'
# Salida verificada: /home/rsa/data/nodered -> /data
```
---
