---
fecha: 2026-09-15
temas: [streaming, drive, resiliencia, diagnostico]
entorno: [acelerografo]
autor: Milton
---

# Actividad del 2026-09-15

**Hitos de la jornada:**

Durante la jornada se abordaron dos anomalías operativas reportadas por los mecanismos de telemetría MQTT en la estación de pruebas `DEV00`: el estancamiento de la adquisición en el Ring Buffer (`status: warning, reason: stale_data`) y una alerta persistente de sincronización con Google Drive (`failed_uploads_protected: 1, reason: upload_retry_retained`). El diagnóstico exhaustivo del pipeline de streaming reveló que, tras reinicios o ejecuciones de la parada de seguridad (`cmd/stop_acquisition_safety`), el servicio en systemd (`rsa-acelerografo.service`) elimina y vuelve a crear el named pipe `/tmp/my_pipe` con un inodo nuevo. Aunque `stream_processor.py` mantenía el descriptor abierto con `os.O_RDWR` (según ADR-006), dicho descriptor quedaba enlazado al inodo desvinculado (`deleted`), dejando al daemon sordo de forma indefinida mientras continuaba en ejecución en Supervisor.

Para erradicar este desacoplamiento sin requerir intervenciones operativas manuales, se diseñó e implementó un mecanismo autónomo de auto-recuperación (Self-Healing) dentro de `StreamProcessor`. Este mecanismo inspecciona periódicamente la validez del inodo mediante `os.fstat(fd).st_ino == os.stat(pipe_path).st_ino` y, ante discrepancias o eliminaciones del archivo FIFO, cierra el descriptor obsoleto, purga los acumuladores internos y reconecta en caliente con backoff exponencial. La solución fue respaldada con 23 pruebas unitarias exhaustivas (100% aprobadas) y validada exitosamente en vivo sobre la Raspberry Pi simulando reinicios forzados del servicio en C, observando la reconexión autónoma inmediata y la reactivación instantánea del flujo al Ring Buffer.

En paralelo, se investigó la alerta en `DriveWatchdog`, determinando que el archivo reportado como protegido retenido (`DEV0_20260907_155830.mseed`) era un falso positivo histórico que ya no existía físicamente en disco. Se robusteció `drive_watchdog.py` incorporando un filtro de existencia física (`f in archivos_disco`) que descarta claves huérfanas en el registro JSON, validado con una suite de 8 pruebas unitarias. Asimismo, se realizó un saneamiento del registro mediante las utilidades nativas de `drive_status_manager.py` y se actualizó integralmente el script de operaciones de campo `ayuda.sh`, modernizando los comandos de adquisición, Supervisor y Google Drive. La sesión concluyó con la actualización de los contextos técnicos (`stream_processor_context.md`, `drive_watchdog_context.md`, `ayuda_context.md`), la enmienda de los registros de arquitectura `ADR-006` y `ADR-020` en el repositorio local y central de metodologías, y la catalogación en el índice federado.

**Decisiones y Cambios:**

- **Auto-Recuperación (Self-Healing) en `StreamProcessor` (`stream_processor.py`)**: Implementación de `_pipe_es_valido()` y `_reconectar_pipe()`. Verificación de inodo cada 1 segundo ante ausencia de datos o `BlockingIOError`. Reconexión transparente en caliente ante recreaciones del FIFO por systemd.
- **Suite de Pruebas de Streaming (`test_stream_processor.py`)**: Ampliación a 23 pruebas unitarias simulando desvinculación de inodos, errores de I/O, reconexión con backoff y acumulación de tramas.
- **Inmunidad a Falsos Positivos en Google Drive (`drive_watchdog.py`)**: Filtrado estricto contra el sistema de ficheros real (`f in archivos_disco`) para evitar que fallos transitorios históricos retengan alarmas cuando los archivos ya no existen en disco.
- **Modernización del Script de Campo (`ayuda.sh`)**: Eliminación de utilidades obsoletas (`extraerevento`, `conversor_mseed.py`), incorporación de comandos de reinicio de adquisición, estado de systemd (`rsa-acelerografo.service`), subida manual a Google Drive con aislamiento del virtualenv (`gestor_archivos_acq.py`), visualización de logs de subida y control completo de servicios en Supervisor (`config_server`, `gpd_worker`, `mqtt_coordinator`, `stream_processor`).
- **Contextos Técnicos y ADRs**: Actualización de `stream_processor_context.md` y `drive_watchdog_context.md`, creación de `ayuda_context.md`, incorporación de la sección de Self-Healing en `ADR-006`, enmienda de la sección 4 y consecuencias en `ADR-020`, y sincronización con `RSA-Metodologias/decisiones/` e `indice_tematico.md`.

**Scripts/Comandos relevantes:**

```bash
# 1. Validación de suite de pruebas de auto-recuperación en la estación
/home/rsa/projects/acelerografo/.venv/bin/python3 -m unittest scripts/operation/streaming/test_stream_processor.py

# 2. Validación de pruebas del auditor de Google Drive
/home/rsa/projects/acelerografo/.venv/bin/python3 -m unittest scripts/operation/mqtt/test_drive_watchdog.py

# 3. Saneamiento del registro de subidas fallidas de Google Drive
/home/rsa/projects/acelerografo/.venv/bin/python3 -c "
import sys; sys.path.insert(0, 'scripts/operation/drive');
from drive_status_manager import DriveStatusManager;
mgr = DriveStatusManager('/home/rsa/data/mseed/uploaded_files_registry.json', '/home/rsa/data/mseed');
mgr.limpiar_archivos_inexistentes()
"

# 4. Prueba en vivo de auto-recuperación del stream processor tras reinicio de C
sudo systemctl stop rsa-acelerografo.service
sudo systemctl start rsa-acelerografo.service
sudo supervisorctl tail stream_processor
```
---
