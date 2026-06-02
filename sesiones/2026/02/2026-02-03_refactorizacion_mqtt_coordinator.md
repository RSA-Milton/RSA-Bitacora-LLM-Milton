---
fecha: 2026-02-03
temas: [mqtt, refactorizacion, telemetria]
entorno: [acelerografo]
---
# Actividad del 2026-02-03

**Hitos de la jornada:**
Refactorización del cliente MQTT (`cliente.py`) hacia un nuevo agente reactivo modular (`mqtt_coordinator.py`). Implementación de una arquitectura de tópicos jerárquicos y telemetría estructurada, migración a configuración por variables de entorno (.env) y estandarización de rutas.

**Decisiones y Cambios:**
- Se eliminó el script antiguo `cliente.py`.
- Se creó `mqtt_coordinator.py` incorporando utilidades, un `CommandDispatcher` y un manejador para métricas de telemetría y salud.
- Se modificó `configuracion_mqtt.json` para definir la nueva jerarquía de tópicos (org/app/cap/id).
- Se extendió la clase `StructuredLogger` para soportar de manera nativa los eventos MQTT (`mqtt_connect`, `mqtt_publish`, etc.).
- Se implementó la resolución de rutas en base a la variable de entorno `PROJECT_LOCAL_ROOT` siguiendo el patrón usado en otros scripts del proyecto.
- Se aseguró la retrocompatibilidad con las distintas versiones de Paho-MQTT (v1 y v2) corrigiendo las firmas de los callbacks (`on_connect`, `on_disconnect`).

**Scripts/Comandos relevantes:**
```bash
# Sincronización manual en entorno local
bash menu.sh # -> Opción 3 (Actualizar)
```
---
# Actividad del 2026-02-04

**Hitos de la jornada:**
Refinamiento profundo de las métricas de hardware de la telemetría (Health) y parametrización de la configuración de Supervisor para asegurar un despliegue sin conflictos por rutas quemadas.

**Decisiones y Cambios:**
- Se eliminó la lógica de Heartbeat del script MQTT, renombrando su respectivo tópico en el JSON de configuración a `events_heartbeat` para manejo futuro.
- Se añadieron nuevas métricas de salud usando utilidades del sistema Raspberry Pi: `disk_percent`, `ram_available_mb`, `load_avg_15m`, `cpu_temp_c` (con `vcgencmd measure_temp`), y estado `throttled` (con `vcgencmd get_throttled`).
- Se modificó `mqtt_coordinator.conf` para el Supervisor de forma que use la plantilla `{{PROJECT_LOCAL_ROOT}}` y la directiva `environment` para proveer la ruta base.
- Se automatizó la sustitución de plantillas en la configuración de Supervisor empleando `sed` dentro de los scripts `deploy.sh` y `update.sh` (nueva función `update_supervisor_config`).

**Scripts/Comandos relevantes:**
```bash
# Ejemplo del reemplazo automatizado de Supervisor
sed "s|{{PROJECT_LOCAL_ROOT}}|$PROJECT_LOCAL_ROOT|g" $PROJECT_GIT_ROOT/scripts/task/mqtt_coordinator.conf > $PROJECT_LOCAL_ROOT/tmp-files/mqtt_coordinator.conf
sudo cp $PROJECT_LOCAL_ROOT/tmp-files/mqtt_coordinator.conf /etc/supervisor/conf.d/
```
---
