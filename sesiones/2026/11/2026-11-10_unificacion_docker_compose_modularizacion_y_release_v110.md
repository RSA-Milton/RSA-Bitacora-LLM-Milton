---
fecha: 2026-11-10
temas: [docker-compose, orquestacion, telegraf, nodered]
entorno: [tig]
autor: Milton
---

# Actividad del 2026-11-10

**Hitos de la jornada:**
Se llevó a cabo una refactorización estructural profunda del repositorio institucional `RSA-Monitor-Sismico` con el fin de consolidar la orquestación de microservicios y eliminar la dispersión de archivos compose. Se centralizó la ejecución del panel de control Node-RED dentro de `services/docker-unified/docker-compose.yml`, eliminando el archivo independiente `services/node-red/docker-compose.yml` y garantizando que todo el stack (InfluxDB, Telegraf, Grafana, Event Analyzer, Node-RED, Correlator Regional y DB Sync) opere coordinadamente bajo una única red Docker (`rsa_network`).

De forma complementaria, se implementó una arquitectura simétrica y modular para las configuraciones de los servicios: se creó el módulo `services/telegraf/` para alojar `telegraf.conf` y su documentación; se estructuró `services/influxdb/` incorporando un script Bash desatendido (`init/01-init-buckets.sh`) montado en `/docker-entrypoint-initdb.d/` para la creación automática e idempotente del bucket permanente `rsa_events`; y se blindó `services/node-red/` con un `.gitignore` estricto y un README operativo para la sincronización de flujos con el volumen `${DATA_DIR}/nodered`. Para prevenir problemas de concurrencia y desconexiones intermitentes ("Client Stealing") en el broker MQTT, se parametrizaron las variables `NODE_RED_PORT` y `NODE_RED_CLIENT_ID`.

Finalmente, se formuló el blueprint de ejecución por fases, se validó el stack en el entorno local WSL 2 con respuestas nominales en todos los puertos expuestos (:8086, :3000, :8501, :1880), se documentó el histórico en `CHANGELOG.md` y se formalizó el release oficial `v1.1.0` en `main`. Durante el despliegue en el servidor central de producción (`server-ubuntu`), se diagnosticó y resolvió un conflicto de nombres por la persistencia del contenedor anterior huérfano de Node-RED, logrando restablecer la operación nominal de los 6 microservicios con perfil `primary`. La decisión técnica quedó registrada en el ADR-028.

**Decisiones y Cambios:**
- Unificación de Node-RED en el Docker Compose maestro (`services/docker-unified/docker-compose.yml`) y remoción del compose redundante en `services/node-red/`.
- Modularización de carpetas de configuración: creación de `services/telegraf/`, `services/influxdb/` (con auto-creación de `rsa_events`) y `services/node-red/` como módulo puro.
- Parametrización de `NODE_RED_CLIENT_ID` y `NODE_RED_PORT` en `.env.example` y `.env` para evitar colisiones de sesión MQTT entre desarrollo y producción.
- Creación de `CHANGELOG.md` institucional bajo el estándar Keep a Changelog y Semantic Versioning (SemVer 2.0.0).
- Sincronización selectiva a `main` aislando `docs/` y publicación del Git Tag oficial anotado `v1.1.0`.
- Saneamiento en producción `server-ubuntu`: detención y remoción del contenedor huérfano previo `/rsa-nodered` y despliegue exitoso del stack unificado.
- Documentación formal de la decisión de arquitectura en el ADR-028.

**Scripts/Comandos relevantes:**
```bash
# 1. Validación de configuración y levantamiento unificado en local (WSL 2)
cd services/docker-unified
docker compose config
docker compose up -d

# 2. Inyección de flujos institucionales y dependencias en Node-RED
docker cp services/node-red/flows.json rsa-nodered:/data/flows.json
docker cp services/node-red/settings.js rsa-nodered:/data/settings.js
docker cp services/node-red/package.json rsa-nodered:/data/package.json
docker exec -w /data rsa-nodered npm --no-audit --no-fund install
docker restart rsa-nodered

# 3. Creación y publicación de release v1.1.0 (en develop y sincronización a main)
git checkout main && git pull origin main
git checkout develop -- services/ scripts/ README.md CHANGELOG.md .gitignore
git rm -r docs/ 2>/dev/null || true
git commit -m "chore: despliegue de componentes para release v1.1.0 en main"
git tag -a v1.1.0 -m "Release v1.1.0: Unificacion de orquestacion Docker Compose y modularizacion de Telegraf, InfluxDB y Node-RED"
git push origin main && git push origin v1.1.0

# 4. Despliegue y resolución de colisión de contenedor en server-ubuntu
docker stop rsa-nodered && docker rm rsa-nodered
cd ~/git/RSA-Monitor-Sismico/services/docker-unified
docker compose --profile primary up -d --build
```
---
