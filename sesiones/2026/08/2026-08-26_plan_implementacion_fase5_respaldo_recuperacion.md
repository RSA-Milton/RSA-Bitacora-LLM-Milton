---
fecha: 2026-08-26
temas: [influxdb, backups, rclone, mqtt]
entorno: [tig]
autor: Milton
---

# Actividad del 2026-08-26

**Hitos de la jornada:**
Se elaboró el diagnóstico técnico exhaustivo y el plan de implementación formal para la Fase 5 del ecosistema RSA (Protocolo de Respaldo, Recuperación y Arquitectura Multi-Servidor), estructurando las estrategias de mitigación ante contingencias de pérdida de datos y cortes de energía prolongados (escenario R4).

Se analizaron las decisiones arquitectónicas para la sincronización híbrida: delimitación del alcance al bucket `rsa_events`, estrategia dual de snapshot binario nativo (`influx backup`) y exportación tabular CSV para análisis interno, retención rotativa de 7 días hacia Google Drive mediante `rclone`, y automatización a través de unidades `systemd timer` (02:00 UTC diario). Asimismo, se diseñó la notificación de estado mediante `paho-mqtt` en entorno virtual, integrando dinámicamente el `${TELEGRAF_CLIENT_ID}` del archivo `.env`.

Para la arquitectura multi-servidor (`rsa-server` primario vs `home-server` espejo pasivo), se resolvió la duplicación del correlador utilizando `profiles: ["primary"]` de Docker Compose y habilitando sesiones persistentes (`clean_session=False`, QoS 1, `client_id` estático) en el correlador regional para retención en broker durante desconexiones. Se redactó y guardó el plan maestro en `docs/blueprints/2026-08-26_fase5_implementation_plan.md` con 22 checkpoints de validación y la guía operativa de comandos Docker.

**Decisiones y Cambios:**
- **Alcance y Formato de Backup**: Respaldo focalizado en `rsa_events` combinando snapshot binario `influx backup` con exportación CSV vía consulta Flux `pivot()`.
- **Rotación y Automatización**: Políticas de purga automática a 7 días y programación con `systemd timer` (`Persistent=true`, `OnCalendar=02:00:00 UTC`).
- **Arquitectura Multi-Servidor**: Control de servicio `correlator` por perfiles Docker (`--profile primary`), manteniendo `home-server` como espejo pasivo habilitado para clasificación manual tipo 3/4.
- **Retención MQTT en Broker**: Configuración de `clean_session=False` y `client_id` fijo en `regional_event_correlator.py` para procesar alertas encoladas tras cortes de energía.
- **Notificación Desacoplada**: Script auxiliar `mqtt_notify.py` utilizando `paho-mqtt` en entorno virtual `scripts/db_sync/.venv`.
- **Documentación y Transición**: Creación de `docs/blueprints/2026-08-26_fase5_implementation_plan.md` y registro en `docs/progress/2026-08-26_contexto-agente.md`.

**Scripts/Comandos relevantes:**
```bash
# Operación en rsa-server (primario con correlador)
docker compose --profile primary up -d
docker compose --profile primary ps

# Operación en home-server (espejo sin correlador)
docker compose up -d

# Verificación de timer systemd y ejecución manual
systemctl status rsa-backup-events.timer
sudo systemctl start rsa-backup-events.service
journalctl -u rsa-backup-events.service -n 50 --no-pager
```
---
