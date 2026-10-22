---
fecha: 2026-10-21
temas: [exportacion, empaquetado, contingencias, optimizacion_memoria]
entorno: [acelerografo]
autor: Milton
---

# Actividad del 2026-10-21

**Hitos de la jornada:**
Se implementó y desplegó en la estación de desarrollo `DEV00` el plan de contingencia offline de conversión y empaquetado diario (`.tar.gz` + `.sha256`) para la exportación de datos sísmicos mediante SFTP. Se unificaron los flags en `procesar_y_subir_pendientes.py` (`--convert-only`, `--upload-only`), desacoplando la fase de conversión de la dependencia de conectividad DNS con Google Drive e implementando la guarda preventiva de espacio libre en disco ($\ge 10\%$). Asimismo, se desarrolló desde cero el módulo `empaquetar_archivos.py`, incorporando cerrojo exclusivo no bloqueante (`flock`), exclusión del archivo binario activo SPI, comprobación preventiva de espacio libre y sumas de verificación criptográficas en formato estándar GNU.

Tras realizar una prueba de empaquetado en la estación de producción `CHA01` (13 días de datos continuos, 871 MB procesados), se observó que la estación (con 0.9 GB de RAM) elevaba su consumo del 28% normal a picos cercanos al 100%. Se identificó que MiniSEED ya cuenta con compresión interna Steim-2 en origen (reducción adicional por gzip de ~18%) y que la saturación de memoria era causada por acumulación de *dirty page cache* del kernel Linux al escribir cientos de megabytes secuenciales sin drenaje a la MicroSD. Se implementaron tres optimizaciones internas críticas: vaciado forzado de buffers (`os.sync()` y `fsync()`), verificación unificada de integridad y cálculo de hash en un único barrido de 64 KB (`verificar_integridad_y_sha256()`), y pausas de drenaje I/O con recolección de basura (`time.sleep(1.0)` y `gc.collect()`) entre cada día calendario en el bucle principal. La suite unitaria (`test_empaquetar_archivos.py`) certificó 10/10 pruebas exitosas en `DEV00`.

**Decisiones y Cambios:**
- Se crearon los wrappers globales `/usr/local/bin/empaquetar` y `/usr/local/bin/procesarpendientes`, integrándolos en `scripts/setup/deploy.sh` y `scripts/setup/update.sh` (opción 3 de `menu.sh`).
- Se amplió la guía interactiva `/usr/local/bin/ayuda` (`ayuda.sh`) con secciones específicas para la conversión offline de pendientes y el empaquetado diario SFTP.
- Se formalizó el blueprint de contingencia en `docs/blueprints/2026-10-21_plan_contingencia_conversion_y_empaquetado_sftp.md`.

**Scripts/Comandos relevantes:**
```bash
# Ejecución de suite de pruebas unitarias en DEV00:
.venv/bin/python3 scripts/export/test_empaquetar_archivos.py

# Empaquetado por rango de fechas con purga segura de fuentes:
empaquetar --start-date 20260903 --end-date 20260930 --delete-source --verbose
```

---

# Actividad del 2026-10-22

**Hitos de la jornada:**
Se ejecutaron pruebas en la estación de producción `CHA02` utilizando `procesarpendientes --convert-only --purge-dat --dry-run`. El script confirmó la presencia de 503 binarios `.dat` acumulados (~21 días), protegió el archivo activo en grabación SPI (`CHA2_261021-210000.dat`), aplicó el *Fail-Fast DNS* ante la falta de resolución a Google Drive sin saturar memoria, y programó la purga segura de archivos `.dat` cuyo `.mseed` ya constaba verificado localmente.

Al iniciar la conversión real, el proceso sufrió un congelamiento indefinido en el primer archivo (`CHA2_261001-000000.dat`). Mediante interrupción por teclado (`^C`), el traceback reveló que el bloqueo residía en `binary_to_mseed.py:obtenerTraza` (línea 457) al invocar `data_completo = np.insert(data_completo, indice_muestra_faltante, lista_ceros)` dentro de un bucle sobre `segundos_faltantes`. Debido a que `np.insert` clona y reasigna arreglos completos de 360,000 muestras en cada iteración, la presencia de huecos temporales (*gaps*) desató un livelock computacional en la CPU de la Raspberry Pi. Como mitigación operativa inmediata, se aislaron los binarios del día 2026-10-01 en una carpeta de cuarentena, permitiendo que el resto de archivos continuos completaran su conversión en menos de 5 segundos por archivo y fueran extraídos exitosamente por SFTP.

Posteriormente, se resolvió un incidente de subidas duplicadas provocado por una tarea en crontab (`procesarpendientes --delete-uploaded`), la cual intentaba re-subir archivos a Google Drive debido a que los respaldos manuales no constaban en `uploaded_files_registry.json`. Se ejecutó una rutina interactiva en Python que pobló el registro de archivos exitosos mediante `drive_status_manager.marcar_como_exitoso()`. Finalmente, se formalizó el diagnóstico técnico en `docs/analysis/`, se crearon y actualizaron los contextos técnicos en `docs/context/` y se indexaron en el catálogo federado de `RSA-Metodologias`.

**Decisiones y Cambios:**
- Se documentó el análisis formal del bloqueo de CPU en `docs/analysis/2026-10-22_diagnostico_bloqueo_cpu_relleno_gaps_binary_to_mseed.md`, catalogando las opciones de solución (asignación vectorizada por *slicing* $O(N)$ vs. segmentación canónica en `obspy.Stream`).
- Se crearon los contextos técnicos `empaquetar_archivos_context.md` y `empaquetar_context.md`, y se actualizaron `procesar_y_subir_pendientes_context.md` y `ayuda_context.md`.
- Se generó el documento de transición técnica `docs/progress/2026-10-22_contexto-agente.md`.

**Scripts/Comandos relevantes:**
```bash
# Rutina de saneamiento masivo para registrar archivos en uploaded_files_registry.json:
cd /home/rsa/projects/acelerografo
.venv/bin/python3 -c "
import os, glob, sys
sys.path.insert(0, 'scripts/drive')
import drive_status_manager as dsm
log_dir = '/home/rsa/projects/acelerografo/log-files'
mseed_dir = '/home/rsa/data/mseed'
for f in sorted(glob.glob(os.path.join(mseed_dir, '*.mseed'))):
    fname = os.path.basename(f)
    if not dsm.ya_fue_subido(log_dir, fname, 'mseed'):
        dsm.marcar_como_exitoso(log_dir, fname, 'mseed')
print('Registro de exitosos actualizado correctamente.')
"
```
