---
fecha: 2026-02-20
temas: [tig, mqtt, docker, documentacion]
entorno: [tig]
---
# Actividad del 2026-02-20

**Hitos de la jornada:**
- Actualización de métricas de salud en el agente `mqtt_coordinator.py` (`disk_percent` y `ram_percent`).
- Inicio de la limpieza de archivos innecesarios en el proyecto TIG-MQTT (Zone.Identifier, directorios vacíos).

**Decisiones y Cambios:**
- Se modificó `mqtt_coordinator.py` para reportar métricas de uso de RAM y Disco en porcentaje, en lugar de en megabytes.
- El usuario decidió actualizar manualmente los dashboards de Grafana (Stage 2) omitiendo la automatización por parte del agente.
- Se eliminaron directorios legacy vacíos y configuraciones temporales de Windows.

**Scripts/Comandos relevantes:**
```bash
rm -v "services/grafana/docker-compose.yml:Zone.Identifier"
```
---

# Actividad del 2026-02-23

**Hitos de la jornada:**
- Consolidación del stack Docker al entorno unificado (`services/docker-unified/`).
- Limpieza profunda de servicios individuales legacy (`services/grafana`, `services/telegraf`, `services/influxdb`, `services/agent`).
- Unificación y reescritura total de la documentación principal (`README.md` y `AGENTS.md`).
- Realización del merge de la rama `rev-milton` hacia `develop`, resolviendo conflictos de documentación.

**Decisiones y Cambios:**
- Se comprobó que el stack activo real estaba en `services/docker-unified` y no en la raíz.
- Se eliminaron todos los stacks individuales legacy para evitar confusión y redundancias.
- Se sobrescribió el `README.md` con un enfoque unificado indicando el estado del proyecto como "100% completado - ENTREGADO".
- Se resolvió el conflicto de git en `README.md` dando precedencia a la documentación consolidada nueva durante el merge.
- Se purgaron archivos y directorios de desarrollo inútiles en la rama `develop` (`config`, `env`, `examples`).

**Scripts/Comandos relevantes:**
```bash
rm -rv docker-compose.yml .env.example services/grafana/docker-compose.yml services/docker-unified/start.sh && rm -rv services/telegraf services/influxdb services/agent
git checkout develop
git merge rev-milton
git add README.md && git commit -m "merge: resolver conflicto en README.md y unificar documentacion"
git rm -r config env examples CLAUDE.md prueba_4.txt
git commit -m "style: limpieza final y unificacion de README en develop"
```
---
