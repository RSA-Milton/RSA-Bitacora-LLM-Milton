---
fecha: 2026-09-01
temas: [adquisicion, drive, supervisor, named-pipe, diagnostico]
entorno: [acelerografo]
autor: Milton
---

# Actividad del 2026-09-01

**Hitos de la jornada:**
Se realizó una sesión de depuración y diagnóstico técnico profundo sobre la estación acelerográfica `CHA1` (configurada como `CHA01`), la cual reportaba una aparente falla en la subida de datos continuos y eventos sísmicos a Google Drive.

Mediante el cruce y análisis cronológico de los registros (`drive.log`, `gestor_acq.log`, `mseed.log`, `registro_continuo.log`, `supervisor_stream_processor.err`, `supervisor_gpd_worker.err`, `mqtt_coordinator.log` y `mqtt_state.json`), se demostró que el servicio de Google Drive y sus credenciales operaban con normalidad (habiendo procesado 27/27 archivos pendientes tras resolver una intermitencia de DNS el 28 de agosto). La causa raíz real fue una parada total del proceso de adquisición base en C (`registro_continuo_4.5.0.c`) tras un reinicio del sistema el 27 de agosto a las 19:17 UTC.

Dicha detención generó una falla en cascada: la ausencia del named pipe `/tmp/my_pipe` provocó excepciones de permisos (`PermissionError`) y de archivo no encontrado (`FileNotFoundError`) en `stream_processor`, impidiendo la creación del segmento de memoria compartida `/dev/shm/rsa_current_frame` y congelando el Ring Buffer. Esto mantuvo a `gpd_stream_worker` en un bucle continuo de reinicios, impidió a `binary_to_mseed.py` generar nuevos MiniSEED y provocó fallos en las extracciones de sismos solicitadas remotamente por MQTT. Tras la intervención técnica (purga de pipes residuales, reinicio de systemd y Supervisor), se validó en `logs.tmp` el restablecimiento nominal de la adquisición, memoria compartida, streaming e inferencia GPD en tiempo real.

**Decisiones y Cambios:**
- Se diagnosticó el efecto dominó en el stack de adquisición, streaming y subida de datos de la estación `CHA1`.
- Se aplicó la mitigación local mediante limpieza de `/tmp/my_pipe` y reinicio coordinado de `rsa-acelerografo.service` y Supervisor.
- Se documentó formalmente el análisis técnico en `montajes/acelerografo-DEV00/docs/analysis/diagnostico_parada_adquisicion_cha01.md`.
- Se catalogaron recomendaciones preventivas para la flota: fijación explícita de permisos `0666` en `registro_continuo.c`, políticas de auto-recuperación (`Restart=always`) y `ExecStartPre` en el servicio systemd, y reintentos defensivos en `stream_processor.py`.

**Scripts/Comandos relevantes:**
```bash
# Limpieza de pipe corrupto y reinicio de adquisición en la estación
sudo rm -f /tmp/my_pipe
sudo systemctl restart rsa-acelerografo.service

# Reinicio coordinado de procesos en Supervisor
sudo supervisorctl restart stream_processor gpd_worker mqtt_coordinator

# Verificación de restablecimiento de streaming y memoria compartida
tail -n 20 /home/rsa/projects/acelerografo/log-files/stream_processor.log
tail -n 20 /home/rsa/projects/acelerografo/log-files/gpd_stream_worker.log
```
