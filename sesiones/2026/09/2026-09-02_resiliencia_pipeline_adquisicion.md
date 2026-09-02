---
fecha: 2026-09-02
temas: [adquisicion, resiliencia, systemd, streaming, mqtt, named-pipe]
entorno: [acelerografo]
autor: Milton
---

# Actividad del 2026-09-02

**Hitos de la jornada:**

Durante esta jornada se ejecutó de forma completa y validó en hardware de campo el **Plan de Resiliencia del Pipeline de Adquisición Post-Incidente CHA01** ([ADR-018](../../../rsa/RSA-Metodologias/decisiones/018_resiliencia_pipeline_adquisicion_acelerografo.md)). El plan mitigó integralmente las vulnerabilidades identificadas en el diagnóstico del 2026-09-01, eliminando la cascada de fallos que provocaba desfasajes en el bus SPI, caídas fatales en Supervisor y paradas silenciosas de adquisición.

Se implementó una arquitectura de defensa en profundidad en 4 capas: (1) Gobernanza de `registro_continuo` bajo systemd (`rsa-acelerografo.service`) con auto-reinicio en 5 s, reseteo físico del microcontrolador dsPIC33 vía `reset_master` (pin MCLR) en `ExecStartPre` y purga de FIFO; (2) Verificación de permisos `0666` en named pipe C; (3) Reintentos con backoff exponencial (`0.5s` a `8.0s`, máx `120s`) en `stream_processor.py` para tolerar arranques asíncronos sin crashear bajo Supervisor; y (4) Módulo `AcquisitionWatchdog` integrado en `mqtt_coordinator.py` que audita cada 60 s la frescura del Ring Buffer y emite alertas estructuradas en el broker MQTT (`status/acquisition`).

Las 4 fases fueron sometidas a una rigurosa batería de pruebas en la estación de desarrollo (`ACEL-DEVP-UNIV-01`), incluyendo simulación de caída abrupta (`kill -9`), corrupción de permisos en el FIFO (`chmod 000`), ráfagas de 3 caídas sucesivas, reinicio de `stream_processor` sin FIFO previo y validación de la telemetría en vivo vía MQTT Explorer (`age_seconds: 1.6s`).

**Decisiones y Cambios:**

- **Capa Systemd / Hardware**: Creada plantilla `rsa-acelerografo.service.template` con `Restart=always`, `RestartSec=5`, `ExecStartPre=reset_master` y purga de `/tmp/my_pipe`. Se eliminaron las tareas `@reboot` desfasadas en `crontab.txt` y se actualizó `registrocontinuo.sh` para controlar el servicio mediante `systemctl`.
- **Automatización de Despliegues**: Actualizados `scripts/setup/deploy.sh` y `scripts/setup/update.sh` (función `update_systemd_service`) para instalar y sincronizar automáticamente cambios en la unidad systemd en nuevas estaciones.
- **Resiliencia en Streaming**: Modificado `scripts/operation/streaming/stream_processor.py` implementando `_abrir_pipe_con_retry()` con backoff exponencial y salida limpia ante timeout. Ampliada la suite de pruebas unitarias en `test_stream_processor.py` (20/20 tests aprobados).
- **Watchdog de Latencia y Telemetría**: Creado `scripts/operation/mqtt/acquisition_watchdog.py` y su suite de tests `test_acquisition_watchdog.py` (5/5 aprobados). Integrado timer de 60 s y comando `get_acquisition_status` en `scripts/operation/mqtt/mqtt_coordinator.py`.
- **Documentación y Ecosistema Semántico**: Generado el blueprint `2026-09-02_plan_resiliencia_pipeline_adquisicion.md`, formalizado `ADR-018`, creados/actualizados contextos técnicos (`acquisition_watchdog`, `stream_processor`, `mqtt_coordinator`, `registro_continuo`), generado el documento de transición técnica `docs/progress/2026-09-02_contexto-agente.md` y sincronizado el índice temático federado.

**Scripts/Comandos relevantes:**

```bash
# 1. Actualización y despliegue del proyecto en estación
cd /home/rsa/git/RSA-Acelerografo
./menu.sh  # Opción 3 (Actualizar el proyecto)

# 2. Control del daemon de adquisición continua
registrocontinuo stop
registrocontinuo start
registrocontinuo restart

# 3. Comprobación en caliente de flujo triaxial y archivo .dat
comprobar

# 4. Auditoría de telemetría de adquisición en broker MQTT
# Topic: rsa/seismic/smart/DEV0/status/acquisition
# Payload: {"status": "ok", "last_frame_utc": "...", "age_seconds": 1.6, "station_id": "DEV0", ...}
```
---
