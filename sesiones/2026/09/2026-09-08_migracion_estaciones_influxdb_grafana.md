---
fecha: 2026-09-08
temas: [influxdb, grafana, db-sync, migracion]
entorno: [tig]
autor: Milton
---

# Actividad del 2026-09-08

**Hitos de la jornada:**

Durante la jornada de hoy se llevó a cabo el saneamiento, depuración y normalización de los registros de telemetría y metadatos de eventos en el servidor central (`RSA-Intern-TIG-MQTT`). En primer lugar, se ejecutó la eliminación definitiva de las series temporales de 4 estaciones dadas de baja o creadas durante fases de prueba (`ACL1`, `SIM01`, `TEST01`, `TEST02`), verificando que no queden registros residuales en el bucket `telemetry`.

En segundo lugar, se implementó el script portable `migrate_stations.py` (ejecutado bajo demanda mediante el contenedor `rsa-db-sync`) para realizar la migración y consolidación determinista de nombres de estaciones que evolucionaron a lo largo del proyecto (`DEV01` ➔ `TEST`, `TEN01`/`TEN1` ➔ `TENG`). El script incorporó tolerancia a la política de retención de 90 días de InfluxDB v2, evitando errores HTTP 422 por puntos en la frontera temporal. Se migraron exitosamente 234.439 puntos de telemetría y se actualizaron 12 eventos multidetección en el catálogo `rsa_events` preservando intactos sus timestamps `_time` y estructuras JSON.

Finalmente, se resolvieron dos inconsistencias visuales en los paneles aprovisionados de Grafana: en `SeismicMonitor` (`seismic_monitor.json`) se corrigió la agregación agrupando por `station_id` antes de `last()`, eliminando filas duplicadas para `TENG`; y en `Health` (`health.json`) se optimizó la variable `station` (versión 3) pre-poblando las 6 estaciones activas (`CHA1`, `CHA2`, `DEV0`, `FERR`, `TENG`, `TEST`) y aplicando un filtro regex estricto para una selección directa e inmediata en la interfaz web.

**Decisiones y Cambios:**

- **Eliminación Definitiva de Estaciones Obsoletas**: Purgadas las series de `ACL1`, `SIM01`, `TEST01` y `TEST02` en InfluxDB v2 mediante `influx delete` por predicado.
- **Script `migrate_stations.py`**: Creado en `scripts/db_sync/migrate_stations.py` con soporte `--dry-run`, detección automática del corte de retención del bucket y reintentos seguros en lotes (`safe_write_batch`).
- **Migración de Telemetría y Eventos**: Reescritos 234.439 puntos a `TEST` y `TENG` y purgadas las series antiguas. Actualizadas las referencias en los campos `stations` y `details` del bucket `rsa_events`.
- **Corrección en `SeismicMonitor`**: Actualizada la consulta Flux en `services/grafana/provisioning/dashboards/seismic_monitor.json` con `|> group(columns: ["station_id"]) |> last()` para consolidar el estado de cada acelerógrafo en una sola fila.
- **Optimización de Variable en `Health`**: Actualizado `services/grafana/provisioning/dashboards/health.json` (v3) con opciones pre-pobladas, `start: -7d` y regex `/^(CHA1|CHA2|DEV0|FERR|TENG|TEST)$/`.
- **Transición Técnica**: Generado el documento `docs/progress/2026-09-08_contexto-agente.md` en el repositorio de trabajo.

**Scripts/Comandos relevantes:**

```bash
# 1. Eliminación definitiva de estaciones obsoletas
for st in ACL1 SIM01 TEST01 TEST02; do
  docker exec rsa-influxdb influx delete \
    --bucket telemetry \
    --org rsa \
    --token "$INFLUXDB_TOKEN" \
    --start '1970-01-01T00:00:00Z' \
    --stop '2030-01-01T00:00:00Z' \
    --predicate "station_id=\"$st\""
done

# 2. Migración de telemetría y eventos vía contenedor db-sync
cd /home/rsa/git/rsa/RSA-Intern-TIG-MQTT/services/docker-unified
docker compose run --rm \
  -v "$(pwd)/../../scripts/db_sync:/app" \
  db-sync migrate_stations.py \
    --url http://influxdb:8086 \
    --token "$INFLUXDB_TOKEN"

# 3. Reinicio de Grafana para aplicar aprovisionamiento
docker restart rsa-grafana
```
---
