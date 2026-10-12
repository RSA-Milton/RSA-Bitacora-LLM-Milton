---
fecha: 2026-10-01
temas: [drive, memoria, dns, diagnostico, resiliencia]
entorno: [acelerografo]
autor: Milton
---

# Actividad del 2026-10-01

**Hitos de la jornada:**
Se dio seguimiento a la estación acelerográfica de campo `CHA02` tras el reseteo físico ejecutado presencialmente por un colega. El diagnóstico automatizado post-reinicio confirmó el restablecimiento pleno de la salud del sistema: carga de CPU baja (`0.56`), temperatura de 48.9 °C, memoria RAM estabilizada (630 MiB libres, 0 swap), pipeline SPI en estado activo y sensor triaxial respondiendo en submilisegundos con reloj sincronizado por NTP (`0.0s`).

Tras el vaciado accidental de la papelera en Google Drive por parte de un operador, se comprobó que los 2,066 archivos acumulados (~2.65 GB entre miniSEED continuos y eventos extraídos) seguían intactos en el disco local de la estación. Asimismo, se descubrió que la estación generaba hasta 9 copias duplicadas de cada archivo en Google Drive debido a la ejecución concurrente descontrolada de `gestor_archivos_acq.py &` disparada cada hora por el cron sin cerrojo `flock`, lo que saturaba la cuota de almacenamiento remoto y provocaba colisiones en `uploaded_files_registry.json`.

Para rescatar la información de forma segura, se configuraron nuevas carpetas de destino en `configuracion_dispositivo.json` y se desarrolló e implementó el script de contingencia `evacuacion_controlada_drive.py`. Dicho script, blindado con cerrojo atómico `fcntl.flock`, inmunidad a desconexión SSH (`signal.SIGHUP`/`nohup`), priorización en dos fases (eventos primero, continuos después), pausas de 1.5 s anti-*bufferbloat* y recolección periódica `gc.collect()`, logró evacuar con éxito **1,934 archivos** en una sola noche. La política de retención posterior purgó 131 archivos expirados, recuperando el espacio libre en disco a **7.8 GB (72% de uso)**.

**Decisiones y Cambios:**
- Se crearon nuevos identificadores de carpeta en Google Drive (`mseed_id` y `events_id`) y se actualizaron en `configuracion_dispositivo.json`.
- Se corrigió la propiedad y permisos de `uploaded_files_registry.json` para el usuario `rsa`.
- Se implementó `evacuacion_controlada_drive.py` en `/home/rsa/projects/acelerografo/scripts/drive/`.
- Se identificó la ruta correcta del entorno virtual de la estación (`.venv`, con punto inicial).

**Scripts/Comandos relevantes:**
```bash
# Lanzamiento desatendido e inmune a desconexión del script de evacuación controlada
nohup /home/rsa/projects/acelerografo/.venv/bin/python3 /home/rsa/projects/acelerografo/scripts/drive/evacuacion_controlada_drive.py > /home/rsa/projects/acelerografo/log-files/evacuacion_stdout.log 2>&1 &

# Monitoreo en tiempo real del progreso de subida
tail -f /home/rsa/projects/acelerografo/log-files/evacuacion_drive.log
```

---

# Actividad del 2026-10-02

**Hitos de la jornada:**
Al generarse los nuevos registros horarios del día (15:00 y 16:00 UTC), se constató mediante `diagnostico drive` que ambos archivos quedaron retenidos como fallidos tras 5 reintentos con el error `Unable to find the server at www.googleapis.com` y `Temporary failure in name resolution`.

El análisis de red reveló que la salida IP hacia Internet estaba 100% operativa (`ping 8.8.8.8` respondió en 28 ms con 0% de pérdida), pero el servidor DNS corporativo asignado institucionalmente (`192.170.100.250`) fallaba sistemáticamente al resolver nombres de dominio de Google. Esto descubrió un defecto crítico de diseño en `check_internet_connection()` de `gestor_archivos_acq.py`, el cual validaba conectividad únicamente con un socket a la IP `8.8.8.8:53`. Al devolver `True`, el gestor iniciaba la subida y se entrampaba en bucles de timeouts de 30-60 segundos por archivo, demorando la corrida por más de una hora.

Se evaluó la situación en la estación hermana `CHA01` en la misma red corporativa, confirmando que la ejecución horaria del cron también genera picos periódicos de consumo de RAM al intentar subir archivos sin resolución DNS. Se acordó respetar la gobernanza de red institucional sin forzar DNS públicos externos, manteniendo el cron horario pausado en `CHA02` y comprobando que el disco local tiene autonomía para almacenar datos de forma segura durante más de 90 días. Se identificó el efecto colateral de que pausar el cron detiene temporalmente la conversión horaria de `binary_to_mseed.py`.

**Decisiones y Cambios:**
- Política de espera prudencial respetando el servidor DNS corporativo `192.170.100.250`.
- Pausa preventiva de la línea horaria de `crontab -e` en `CHA02` para evitar acumulación de procesos y livelock de memoria.
- Validación de que el disco con 7.8 GB libres garantiza retención local segura (>90 días).

**Scripts/Comandos relevantes:**
```bash
# Verificación de resolución DNS institucional en la estación
ping -c 3 8.8.8.8
ping -c 3 www.googleapis.com

# Comprobación de procesos acumulados en CHA01
pgrep -a -f gestor_archivos_acq.py
free -h
```

---

# Actividad del 2026-10-12

**Hitos de la jornada:**
Se consolidó la investigación forense actualizando el documento formal de diagnóstico técnico `docs/analysis/2026-09-30_diagnostico_saturacion_memoria_y_bloqueo_subidas_drive.md`, detallando el falso positivo en la detección de conectividad, el mecanismo de duplicación masiva en Drive, la falla del DNS corporativo, el resultado de la evacuación de 1,934 archivos y la hoja de ruta en 5 fases hacia la refactorización definitiva. Asimismo, se generó el documento de transición técnica de la sesión en `docs/progress/2026-10-12_contexto-agente.md` para transferir el contexto completo al siguiente agente.

**Decisiones y Cambios:**
- Actualización de estado del diagnóstico técnico en `indice_tematico.md` a "en mitigación".
- Elaboración de documento de transición técnica en `docs/progress/`.
- Definición de hoja de ruta para el próximo blueprint: desacoplamiento de `binary_to_mseed.py` en `registrocontinuo.sh`, cerrojo nativo `fcntl.flock`, validación DNS real a dominios Google con aborto temprano (*fail-fast*), y control por lotes (`max_batch_size`).
