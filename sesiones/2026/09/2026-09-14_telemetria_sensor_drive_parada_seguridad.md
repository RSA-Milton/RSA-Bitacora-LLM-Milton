---
fecha: 2026-09-14
temas: [telemetria, mqtt, sensor, drive]
entorno: [acelerografo]
autor: Milton
---

# Actividad del 2026-09-14

**Hitos de la jornada:**

Durante la jornada se implementaron, desplegaron y validaron empíricamente en la estación acelerográfica de pruebas `DEV00` las cinco fases del plan de observabilidad y resiliencia de la Red Sísmica del Austro. Se desarrollaron dos nuevos módulos auditores: `SensorWatchdog` (para la verificación periódica de la integridad física del acelerómetro triaxial en reposo y sincronía del reloj) y `DriveWatchdog` (para la auditoría de backlog de archivos MiniSEED en disco, detección de reintentos retenidos de subida y porcentaje de almacenamiento libre). Ambos componentes fueron acompañados de suites exhaustivas de pruebas unitarias (16 pruebas en total) alcanzando 100% de cobertura funcional.

En el orquestador principal (`mqtt_coordinator.py`), se unificó la cadencia temporal a 5 minutos (300 segundos), sincronizando en una ráfaga coherente los canales de salud de hardware (`telemetry/health`), adquisición continua (`status/acquisition`), integridad del sensor (`status/sensor`) y sincronización de Google Drive (`status/drive`), configurando publicación persistente (`retain = true`) y `QoS 1`. Asimismo, se incorporó al despachador de comandos la orden remota de contingencia `stop_acquisition_safety`, permitiendo detener inmediatamente la adquisición continua ante fallas físicas graves del sensor para proteger la integridad del almacenamiento y la red. Se completaron 7 pruebas de integración para validar la ráfaga de publicación y el enrutamiento de comandos.

Finalmente, en la validación en vivo sobre la estación `DEV0`, se depuró y corrigió un fallo de localización del script de diagnóstico (`wrapper_not_found`), actualizando el script de despliegue `update.sh` para sincronizar `scripts/operation/acelerografo/` hacia el entorno productivo y forzando a `SensorWatchdog` a resolver exclusivamente en `$PROJECT_LOCAL_ROOT`. Se verificó la recepción simultánea de telemetría nominal vía MQTT Explorer ($A_x = 0.4414, A_y = 0.1484, A_z = 9.5807\text{ m/s}^2$) y se ejecutó exitosamente la prueba en vivo del comando remoto `cmd/stop_acquisition_safety`, deteniendo limpiamente `rsa-acelerografo.service` al segundo exacto de recepción. La jornada concluyó con la creación de los contextos técnicos, el registro de ADR-020 y la transición técnica de la sesión.

**Decisiones y Cambios:**

- **Auditor de Sensor Triaxial (`sensor_watchdog.py`)**: Evaluación con timeout defensivo (12 s) de aceleraciones en reposo ($|A_x|, |A_y| \le 0.5\text{ m/s}^2$, $|A_z - 9.81| \le 0.8\text{ m/s}^2$) y estado del reloj (`GPS`/`RPi`).
- **Resolución Estricta de Rutas en Producción**: Búsqueda del wrapper de verificación restringida únicamente a `$PROJECT_LOCAL_ROOT/scripts/acelerografo/comprobar_registro_wrapper.py`, sin rutas relativas al árbol Git ni rutas absolutas hardcodeadas.
- **Auditor de Google Drive (`drive_watchdog.py`)**: Soporte dual de registros de subida, detección de backlog (>3 archivos) y alerta preventiva ante archivos protegidos retenidos (`failed_uploads_protected > 0`), combinada con telemetría de espacio en disco vía `shutil.disk_usage`.
- **Unificación de Cadencia a 300 s (`mqtt_coordinator.py`)**: Sincronización de ciclos de publicación de salud y estados en una ráfaga temporal idéntica con retención MQTT habilitada.
- **Comando de Seguridad Remoto (`stop_acquisition_safety`)**: Detención controlada de `rsa-acelerografo.service` vía `systemctl stop` con respuesta en `cmd/stop_acquisition_safety/res`.
- **Despliegue y Plantillas**: Inclusión de nuevos tópicos en `configuracion_mqtt.json.template` y actualización de `scripts/setup/update.sh` para automatizar la sincronización de scripts operativos del acelerógrafo.
- **Documentación y Exocortex**: Elaboración de `sensor_watchdog_context.md`, `drive_watchdog_context.md`, actualización de `mqtt_coordinator_context.md`, registro de `ADR-020` y emisión de transición técnica `2026-09-14_contexto-agente.md`.

**Scripts/Comandos relevantes:**

```bash
# 1. Ejecución de suites de pruebas en la estación remota
/home/rsa/projects/acelerografo/.venv/bin/python3 -m unittest scripts/operation/mqtt/test_sensor_watchdog.py
/home/rsa/projects/acelerografo/.venv/bin/python3 -m unittest scripts/operation/mqtt/test_drive_watchdog.py
/home/rsa/projects/acelerografo/.venv/bin/python3 -m unittest scripts/operation/mqtt/test_mqtt_coordinator_integration.py

# 2. Despliegue de actualización y reinicio del coordinador MQTT
cd /home/rsa/git/RSA-Acelerografo && ./menu.sh # Opción 3: Actualizar
sudo supervisorctl restart mqtt_coordinator

# 3. Emisión de comando remoto de parada de seguridad vía MQTT
mosquitto_pub -h "$MQTT_HOST" -p "$MQTT_PORT" -u "$MQTT_USER" -P "$MQTT_PASS" \
  -t "rsa/acelerografo/estaciones/DEV0/cmd/stop_acquisition_safety" \
  -m '{"action":"stop"}'

# 4. Reactivación nominal del servicio de adquisición tras prueba de contingencia
sudo systemctl start rsa-acelerografo.service
sudo systemctl status rsa-acelerografo.service
```
---
