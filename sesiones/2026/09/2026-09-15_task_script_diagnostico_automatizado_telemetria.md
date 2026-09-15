---
fecha: 2026-09-15
temas: [diagnostico, telemetria, automatizacion]
entorno: [acelerografo]
autor: Milton
---

# Actividad del 2026-09-15 (Sesión 2: Task-Script de Diagnóstico Automatizado de Telemetría)

**Hitos de la jornada:**

Durante esta sesión se diseñó, implementó, desplegó y validó en producción un nuevo task-script de diagnóstico automatizado denominado `diagnostico.sh`. El propósito es dotar a las estaciones acelerográficas de la RSA de una herramienta CLI integral que asista tanto a operadores de campo como a agentes de IA en la identificación inmediata de la causa raíz ante alertas o fallas notificadas en los tópicos MQTT especializados (`rsa/seismic/smart/*/status/*`). Se elaboró inicialmente un blueprint estructurado de cuatro fases (`docs/blueprints/2026-09-15_plan_diagnostico_automatizado_telemetria.md`), catalogando exhaustivamente todas las razones posibles de degradación (`stale_data`, `ring_buffer_dir_not_found`, `no_data_available`, `wrapper_not_found`, `accelerometer_anomaly`, `null_readings`, `timeout`, `pending_backlog`, `upload_retry_retained`).

El script implementado (`scripts/task/diagnostico.sh`) opera bajo arquitectura modular en Bash puro, sin dependencias externas fuera de herramientas estándar del sistema y el entorno virtual local (`.venv`). Permite diagnósticos acotados (`diagnostico [all|acquisition|sensor|drive]`) y ejecución desatendida mediante el flag silencioso `--quiet` / `-q`. Recolecta telemetría general de hardware y servicios (temperatura Raspberry Pi, estados de Supervisor y systemd), auditoría profunda del pipeline de adquisición SPI (estado del named pipe `/tmp/my_pipe`, descriptores de archivo abiertos por `stream_processor`, secuencia de archivos en el Ring Buffer), verificación física y de reloj del acelerómetro mediante la invocación directa de `comprobar_registro_wrapper.py`, y estado de sincronización con Google Drive (conteo de MiniSEED pendientes, registro JSON y conectividad de red). La salida se consolida en `$PROJECT_LOCAL_ROOT/log-files/diagnostico_report.log`, sobrescribiéndose en cada ejecución para proveer un snapshot limpio al agente de análisis.

Se completó el ciclo completo de validación en la estación de pruebas `DEV00`: el script se desplegó automáticamente a `/usr/local/bin/diagnostico` a través del mecanismo existente `update_task_scripts` de `menu.sh` (Opción 3), se ejecutó en caliente en el equipo remoto y se verificó que el reporte generado reflejara con total exactitud la condición nominal del sistema. Asimismo, se aclararon dudas operativas respecto a trazas históricas de inicio en `supervisor_stream_processor.err`, confirmando que el servicio mantenía un descriptor activo y un uptime ininterrumpido superior a 18 horas. La sesión concluyó con la integración de la documentación de uso en `ayuda.sh` (verificado en `/usr/local/bin/ayuda`) y la generación del documento de contexto técnico `docs/context/diagnostico_context.md`.

**Decisiones y Cambios:**

- **Implementación de `diagnostico.sh` (`scripts/task/diagnostico.sh`)**: Creación de las rutinas `diagnostico_general()`, `diagnostico_acquisition()`, `diagnostico_sensor()` y `diagnostico_drive()`. Centralización de la salida en `$PROJECT_LOCAL_ROOT/log-files/diagnostico_report.log` con soporte para stdout y flag silencioso (`--quiet`).
- **Compatibilidad con Despliegue Automatizado**: El script se integra nativamente con `update.sh` (`update_task_scripts`), desplegándose sin modificaciones en `/usr/local/bin/diagnostico`.
- **Actualización de Documentación de Campo (`scripts/task/ayuda.sh`)**: Inclusión de la sección "Diagnostico Automatizado (Reporte para Agente IA)" con sintaxis de ejecución completa, acotada y silenciosa.
- **Contexto Técnico para Agentes IA (`docs/context/diagnostico_context.md`)**: Creación de la guía técnica con diagrama de flujo Mermaid, matriz de mapeo entre `reason` de MQTT y rutinas de inspección, y directrices para discernir errores históricos de anomalías activas.
- **Archivo de Blueprint (`docs/blueprints/2026-09-15_plan_diagnostico_automatizado_telemetria.md`)**: Registro formal de la planificación técnica en el repositorio del proyecto.

**Scripts/Comandos relevantes:**

```bash
# 1. Ejecutar diagnóstico completo en la estación remota
diagnostico

# 2. Ejecutar diagnóstico de un canal específico
diagnostico acquisition
diagnostico sensor
diagnostico drive

# 3. Ejecutar diagnóstico en modo silencioso (solo escribe en log para consumo de IA)
diagnostico --quiet

# 4. Inspeccionar el reporte de diagnóstico generado
cat $PROJECT_LOCAL_ROOT/log-files/diagnostico_report.log

# 5. Visualizar la ayuda actualizada con las nuevas opciones
ayuda
```
---
