---
fecha: 2026-09-18
temas: [mqtt, telemetria, resiliencia]
entorno: [acelerografo]
autor: Milton
---

# Actividad del 2026-09-18 (Latido Periódico de Telemetría State y Mitigación de Condiciones de Carrera LWT)

**Hitos de la jornada:**

Durante esta jornada se diagnosticó y erradicó una anomalía recurrente en la observabilidad de las estaciones acelerográficas: tras parpadeos de red o microcortes de conectividad con el broker MQTT, la plataforma central de monitoreo mostraba a la estación como `"offline"`, permaneciendo en ese estado falso durante horas (hasta el refresco diario a las 00:00 UTC) a pesar de que la estación continuaba operando normalmente, adquiriendo datos sísmicos y transmitiendo métricas de salud (`telemetry/health`) y diagnóstico de watchdogs (`status/*`) cada 300 segundos.

El análisis formalizó el diagnóstico técnico (`docs/analysis/2026-09-18_diagnostico_falsos_offline_telemetry_state.md`), identificando que el cierre de sockets huérfanos por *session takeover* en el broker disparaba el Will Message (**LWT - Last Will and Testament**) fijando `"offline"` con `retain=True`, lo que colisionaba con el paquete `"online"` emitido de forma mono-disparo en `on_connect()`. 

Para resolver esta vulnerabilidad sistémica, se elaboró y ejecutó un plan de implementación en cuatro fases:
1. **Priorización en `on_connect()` y Control de Conexión en `mqtt_coordinator.py`**: Se reordenó el flujo para que el estado `"online"` se transmita y registre de forma inmediata al autenticarse, antes de suscribirse a los 5 tópicos de comandos y eventos. Se integró el rastreo determinista de `userdata["is_connected"]` en los callbacks de conexión y desconexión.
2. **Latido Periódico de 300 s (Heartbeat - Opción A)**: Se integró en el bucle principal de `mqtt_coordinator.py` la retransmisión periódica de `telemetry/state` con QoS 1 y `retain=True` acoplada a la ráfaga de 5 minutos, reutilizando el timestamp original de la sesión activa (`last_state_change`). Esto garantiza la autorrecuperación de cualquier anomalía de retención en un tiempo máximo de 5 minutos sin alterar el cálculo del tiempo de actividad ininterrumpido (*uptime*).
3. **Expansión de la Suite de Pruebas**: Se agregaron 3 nuevos tests unitarios en `scripts/operation/mqtt/test_mqtt_coordinator_integration.py` para certificar QoS 1, `retain=True`, preservación del timestamp de sesión y descarte de latidos ante desconexión, logrando un resultado de **10/10 tests pasados exitosamente (`Todo OK ✅`)**.
4. **Despliegue y Validación en Vivo en `DEV00`**: Tras reiniciar el servicio con `sudo supervisorctl restart mqtt_coordinator`, se validó en los logs la emisión prioritaria de `'online'` y el disparo inmediato del latido corrector `[HEARTBEAT_STATE]`, operando con parámetros nominales de hardware, adquisición (`age=0.7s`), sensor y Google Drive.

La jornada culminó con la formalización del **ADR-023** (`023_heartbeat_periodico_telemetria_state_y_resiliencia_lwt.md`) en el repositorio local y en el repositorio federado `rsa/RSA-Metodologias/`, la actualización del contexto técnico `mqtt_coordinator_context.md`, la indexación en `indice_tematico.md` y la generación de la transición técnica en `docs/progress/`.

**Decisiones y Cambios:**

- **Latido Periódico de State con Preservación de Sesión (ADR-023 - Opción A)**: El estado `"online"` en `telemetry/state` se re-publica con `retain=True` cada 300 segundos, preservando el timestamp fijado al inicio de la conexión activa (`last_state_change`) para mantener la semántica de uptime.
- **Priorización en `on_connect()`**: Publicación inmediata de `"online"` antes de iterar sobre el bucle de suscripciones a los 5 tópicos.
- **Rastreo Determinista de Enlace**: Gestión del flag `userdata["is_connected"]` en `on_connect()` y `on_disconnect()` para evitar emisiones espurias de latido durante interrupciones de red.
- **Actualización de Documentación de Contexto**: Reflejo del comportamiento de cadencia híbrida (conexión + heartbeat 300 s) en `mqtt_coordinator_context.md`.
- **Suite de Pruebas Ampliada**: Cobertura de tests unitarios de integración ampliada a 10 pruebas automatizadas.

**Scripts/Comandos relevantes:**

```bash
# 1. Compilación sintáctica del coordinador en la Raspberry Pi
$PROJECT_LOCAL_ROOT/.venv/bin/python3 -m py_compile $PROJECT_LOCAL_ROOT/scripts/mqtt/mqtt_coordinator.py

# 2. Ejecución de la suite de pruebas unitarias/integración de MQTT (10/10 tests)
$PROJECT_LOCAL_ROOT/.venv/bin/python3 $PROJECT_LOCAL_ROOT/scripts/mqtt/test_mqtt_coordinator_integration.py

# 3. Reiniciar el daemon del coordinador bajo Supervisor
sudo supervisorctl restart mqtt_coordinator

# 4. Monitorear los logs en vivo y verificar el latido periódico
tail -n 25 $PROJECT_LOCAL_ROOT/log-files/mqtt_coordinator.log
```
---
