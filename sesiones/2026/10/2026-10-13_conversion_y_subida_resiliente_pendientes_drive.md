---
fecha: 2026-10-13
temas: [acelerografo, drive, conversion, resiliencia]
entorno: [acelerografo]
autor: Milton
---

# Actividad del 2026-10-13

**Hitos de la jornada:**
Se diagnosticó la acumulación masiva de archivos binarios `.dat` no convertidos y registros miniSEED tras la suspensión preventiva del cron en estaciones remotas (`CHA01`, `CHA02`), provocada por fallas del servidor DNS corporativo (`192.170.100.250`). El script histórico `gestor_archivos_acq.py` colapsaba la memoria RAM por bucles continuos de reintentos ante la imposibilidad de resolver los dominios de Google Drive.

Se planificó e implementó una herramienta modular y desacoplada de contingencia: `procesar_y_subir_pendientes.py` junto con el wrapper ejecutable de sistema `/usr/local/bin/procesarpendientes` (`procesarpendientes.sh`). Se estructuró en dos fases secuenciales independientes: conversión local protegida con ObsPy (excluyendo el archivo activo del bus SPI y omitiendo archivos ya procesados) y subida controlada por lotes (`--batch-size`) con cerrojo atómico `flock` y pausas anti-bufferbloat.

Se incorporó la suite de pruebas unitarias `test_procesar_y_subir_pendientes.py` con aislamiento en `DEV0`, se aplicó el blindaje incondicional de permisos en `update.sh` y se publicó el release oficial `v4.5.3` en `main` aislando `docs/`. Se ejecutó con éxito la prueba en la estación piloto `CHA01`, convirtiendo 270 archivos pendientes y subiendo el primer lote de 30 a Google Drive sin saturar la RAM.

**Decisiones y Cambios:**
- Desacoplamiento estricto entre la fase de conversión local y la fase de subida a la nube.
- Rechazo de reinvocaciones automáticas internas de Python (`_asegurar_entorno_virtual`); el entorno se garantiza en el wrapper bash de sistema.
- Exclusión incondicional del archivo activo en grabación SPI por tiempo de modificación y registro temporal.
- Blindaje de permisos `chown/chmod rsa:rsa` incondicional en `update.sh`.

**Scripts/Comandos relevantes:**
```bash
# Ejecución de prueba y despliegue en CHA01
procesarpendientes --dry-run
procesarpendientes --batch-size 30
```

---

# Actividad del 2026-10-14

**Hitos de la jornada:**
Se identificó y corrigió un riesgo crítico de agotamiento de almacenamiento en disco: en estaciones desatendidas desconectadas, convertir todo el backlog de golpe duplicaba el espacio ocupado por cada hora en la tarjeta SD (~18 MB por hora: `.dat` + `.mseed`). 

Se implementó el mecanismo de **Control de Flujo por Retro-Presión (Backpressure)** con cálculo de cupo dinámico (`max(0, batch_size - en_disco)`) y verificación previa de DNS (*fail-fast* inicial) antes de iniciar cualquier conversión. Adicionalmente, se incorporaron las banderas de liberación de almacenamiento `--delete-uploaded` (eliminación inmediata del `.mseed` local tras recibir HTTP 200 en Google Drive) y `--purge-dat` (Fase 3 de purga de archivos binarios `.dat` cuyos miniSEED ya consten confirmados en la nube).

Se ampliaron las pruebas unitarias a 9/9 tests aprobados en `DEV0`, se publicó el release oficial `v4.5.4` y se desplegó en `CHA01` configurando la automatización en crontab cada hora al minuto 05 (`5 * * * *`). Se generó el documento de contexto técnico `docs/context/procesar_y_subir_pendientes_context.md`, la transición técnica `docs/progress/2026-10-14_contexto-agente.md` y se actualizó el índice temático federado en `RSA-Metodologias`.

**Decisiones y Cambios:**
- Implementación de Fail-Fast DNS inicial: Si no hay resolución a Google, se aborta antes de convertir para no generar `.mseed` huérfanos.
- Cupo dinámico con retro-presión: Si la cola en disco ya tiene suficientes archivos esperando subida, se frena la conversión de nuevos `.dat`.
- Adición de banderas `--delete-uploaded` y `--purge-dat` para estaciones con almacenamiento crítico.
- Registro y publicación del release SemVer `v4.5.4` en `main`.

**Scripts/Comandos relevantes:**
```bash
# Configuración en crontab de CHA01:
5 * * * * /usr/local/bin/procesarpendientes --batch-size 30 >> /home/rsa/projects/acelerografo/log-files/procesarpendientes_cron.log 2>&1

# Ejecución combinada con liberación de almacenamiento:
procesarpendientes --batch-size 30 --delete-uploaded --purge-dat
```
