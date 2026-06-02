---
fecha: 2026-05-14
temas: [grafana, telegraf, influxdb, bugfix]
entorno: [tig]
---
# Actividad del 2026-05-14

**Hitos de la jornada:**
Se resolvieron problemas de datos duplicados en los paneles de Grafana ocasionados por el reinicio del contenedor Docker de Telegraf, el cual generaba nuevos IDs de contenedor usados como tags `host` en InfluxDB. Adicionalmente, se revirtió el cambio de nombre y UID en la exportación de Grafana para evitar dashboards duplicados en el aprovisionamiento.

**Decisiones y Cambios:**
- Se deshabilitó la propagación del hostname en `services/docker-unified/telegraf.conf` estableciendo `omit_hostname = true`.
- Se actualizaron las consultas Flux del dashboard `health.json`, añadiendo la función `|> group()` a los paneles de Uptime, Disco, RAM, Load Average y Temperatura para unificar métricas históricas duplicadas.
- Se restauró manualmente el UID y el Title del dashboard en el archivo JSON exportado para mantener la persistencia.

**Scripts/Comandos relevantes:**
```bash
docker restart rsa-telegraf
```
---
