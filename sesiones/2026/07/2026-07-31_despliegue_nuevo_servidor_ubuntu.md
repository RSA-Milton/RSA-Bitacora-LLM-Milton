---
fecha: 2026-07-31
temas: [docker, ubuntu, tig, node-red, grafana, influxdb, mqtt]
entorno: [tig]
autor: Milton
---

# Actividad del 2026-07-31

**Hitos de la jornada:**
Despliegue y depuración del stack completo de monitoreo y control (`RSA-Intern-TIG-MQTT`) en un nuevo servidor Ubuntu 22.04.5 LTS (`192.168.10.200`). Se verificó la operatividad de Docker Engine 29.7.1 y Docker Compose v5.3.1. Se identificaron y solucionaron conflictos de permisos en los volúmenes persistentes de Grafana (UID 472) y Node-RED (UID 1000).

Para lograr un despliegue portátil e inmune a las rutas del usuario host (`/home/rsa/` vs `/home/parmont/`), se parametrizaron las rutas de volúmenes en `docker-compose.yml` utilizando las variables `${DATA_DIR}` y `${DRIVE_DIR}`. Todos los servicios principales (Grafana `:3000`, Node-RED `:1880`, Event Analyzer `:8501`, InfluxDB `:8086`) quedaron en estado `Up` y accesibles en la red local. Finalmente, mediante consulta histórica en el exocortex, se resolvió la causa del error `unknown: ui_dropdown` identificando la necesidad de instalar el paquete `node-red-dashboard` en la paleta de Node-RED.

**Decisiones y Cambios:**
- Parametrización de volúmenes Docker: Reemplazo de las rutas hardcodeadas `/home/rsa/data/` y `/home/rsa/datos_estaciones_drive` por `${DATA_DIR}` y `${DRIVE_DIR}` en `services/docker-unified/docker-compose.yml` y `services/node-red/docker-compose.yml`.
- Gestión de rutas en `.env`: Adición de las variables `DATA_DIR` y `DRIVE_DIR` a `services/docker-unified/.env.example` y a los archivos `.env` locales.
- Asignación de permisos de usuario host: Aplicación de `sudo chown -R 472:472 /home/parmont/data/grafana` y `sudo chown -R 1000:1000 /home/parmont/data/nodered` para permitir a Grafana y Node-RED escribir en sus volúmenes persistentes.
- Gestión de dependencias en Node-RED: Confirmación de que `node-red-dashboard` debe instalarse en el volumen `/data` vía Manage Palette en la interfaz web de Node-RED o mediante `npm install --prefix /data node-red-dashboard`.

**Scripts/Comandos relevantes:**
```bash
# Configuración de variables de ruta en .env
DATA_DIR=/home/parmont/data
DRIVE_DIR=/home/parmont/datos_estaciones_drive

# Corrección de permisos de usuario para contenedores sin privilegios
sudo chown -R 472:472 /home/parmont/data/grafana
sudo chown -R 1000:1000 /home/parmont/data/nodered

# Recreación y despliegue del stack unificado y Node-RED
cd ~/git/rsa/RSA-Intern-TIG-MQTT/services/docker-unified
docker compose up -d --force-recreate

cd ~/git/rsa/RSA-Intern-TIG-MQTT/services/node-red
docker compose up -d --force-recreate

# Instalación de paleta Dashboard en Node-RED desde terminal
docker exec -it rsa-nodered npm install --prefix /data node-red-dashboard
```
---
