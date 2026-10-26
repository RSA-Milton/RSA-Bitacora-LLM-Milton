---
fecha: 2026-10-23
temas: [mseed, gaps, livelock, resiliencia]
entorno: [acelerografo]
autor: Milton
---

# Actividad del 2026-10-23

**Hitos de la jornada:**
Se completó la implementación y validación integral de la Fase 6 del blueprint de refactorización de `binary_to_mseed.py` en la estación de desarrollo `DEV00`. El objetivo primordial fue erradicar definitivamente el livelock de CPU $O(M \cdot N)$ al rellenar gaps de ceros, corregir la corrupción de fase sismológica y la inflación dimensional, y neutralizar la vulnerabilidad ante ruido de bus SPI detectada en el archivo real `CHA2_260928-170000.dat` (glitch `04:04:04` que inflaba artificialmente el MiniSEED a 25 horas).

Se implementó una triple barrera en el conversor: detección y purga en cabecera cruda de tramas con bytes idénticos (`glitch_bytes_identicos`), filtro de coherencia cronológica para tramas outlier aisladas, y acotamiento estricto del cruce de medianoche a la ventana de la hora 23 ($t \ge 86000$ s) a la hora 00 ($t \le 3600$ s). Se incorporó el test unitario T11 en `test_binary_to_mseed.py`, validando que toda la suite sintética de 12 tests pase en 0.096 s en la Raspberry Pi 3B+.

Asimismo, se procesó el archivo patológico de CHA02 en disco local utilizando el nuevo flag `--output-dir /home/rsa/data/test-bin`, certificando con ObsPy que el MiniSEED generado dura exactamente 3599.996 s (1.00 hora nominal) con 900,000 muestras exactas a 250 Hz, reduciendo el tamaño del archivo de 81 MB a 8.6 MB y el tiempo de conversión a 0.95 s.

**Decisiones y Cambios:**
- Refactorización de `obtenerTraza` con preallocado directo de ceros continuos y dispersión indexada de bloques de 250 muestras en $O(N)$.
- Entrega de `tiempo_binario_validado` en `leer_archivo_binario` (6to valor de retorno) para sincronizar el tiempo canónico del archivo con la primera trama que superó los filtros de integridad.
- Deduplicación vectorial tridimensional de tramas con timestamps repetidos en NumPy $O(1)$.
- Incorporación del test unitario T11 (`test_purga_glitch_spi_y_outlier_temporal_t11`).
- Soporte CLI para `--output-dir` / `-o` en `binary_to_mseed.py`.

**Scripts/Comandos relevantes:**
```bash
# Sincronización y ejecución de suite unitaria en DEV00
cd /home/rsa/projects/acelerografo
cp -v /home/rsa/git/RSA-Acelerografo/scripts/operation/mseed/*.py scripts/mseed/
.venv/bin/python3 scripts/mseed/test_binary_to_mseed.py
# Ran 12 tests in 0.096s - OK

# Reprocesamiento de archivo patológico de CHA02
time .venv/bin/python3 scripts/mseed/binary_to_mseed.py 3 /home/rsa/data/test-bin/CHA2_260928-170000.dat --output-dir /home/rsa/data/test-bin

# Inspección sismológica de verificación en ObsPy
.venv/bin/python3 -c "
import obspy
st = obspy.read('/home/rsa/data/test-bin/DEV0_20260928_170000.mseed')
print(st)
for tr in st:
    dur_s = tr.stats.endtime - tr.stats.starttime
    print('Canal: {}, Npts: {}, Duracion: {:.3f} s ({:.2f} h)'.format(
        tr.stats.channel, tr.stats.npts, dur_s, dur_s / 3600.0
    ))
"
```

---

# Actividad del 2026-10-26

**Hitos de la jornada:**
Se ejecutó la prueba de procesamiento masivo en lote sobre los 25 archivos binarios reales de la estación `CHA02` en `/home/rsa/data/test-bin` (`CHA2_260928-170000.dat` y los 24 archivos de la jornada completa `CHA2_261001-*.dat`). El 96% de los archivos (24 de 25) convirtieron exitosamente con tiempos promedio de ~0.95 s por hora en Raspberry Pi y duraciones exactas de 1.00 h (900,000 muestras) o duraciones coherentes en segmentos de reinicio (`010001.dat`, `020202.dat`, `070000.dat`, `160648.dat`, `160752.dat`).

Se identificó y diagnosticó a profundidad un fallo aislado en `CHA2_261001-150002.dat` (`KeyError: 'anio'`), correspondiente a un corte abrupto de energía en la estación durante la hora 15 que dejó la última trama física truncada. Al fallar `extraer_timestamp_completo_binario` con `(None, None)`, `leer_archivo_binario` omitió inyectar las claves de fecha en `tiempo_binario_validado`, a pesar de que todas las tramas previas en memoria eran válidas.

Bajo la premisa sismológica institucional de que **es mejor no tener datos a tener datos falsos o con tiempo aproximado**, se analizó y consensuó que la fecha debe extraerse de forma estricta y autónoma de la primera trama válida en memoria (`chunk_valido[0]`), purgando cualquier trama rota sin inventar tiempos. Se formalizó el diagnóstico técnico `2026-10-26`, se documentó formalmente el `ADR-026` de dispersión vectorizada de gaps y blindaje SPI en el exocortex, se actualizaron los contextos técnicos (`binary_to_mseed_context.md` y `test_binary_to_mseed_context.md`) y se generó la transición técnica de la sesión.

**Decisiones y Cambios:**
- Formalización del diagnóstico técnico [`2026-10-26_diagnostico_desacople_timestamp_archivos_incompletos_binary_to_mseed.md`](file:///home/rsa/git/montajes/acelerografo-DEV00/docs/analysis/2026-10-26_diagnostico_desacople_timestamp_archivos_incompletos_binary_to_mseed.md) en el proyecto del acelerógrafo.
- Aprobación y documentación del [`ADR-026: Dispersión Vectorizada O(N) para Gaps Temporales, Preservación de Fase y Blindaje de Ruido SPI en binary_to_mseed`](file:///home/rsa/git/rsa/RSA-Metodologias/decisiones/026_dispersion_vectorizada_gaps_y_blindaje_spi_binary_to_mseed.md) en `RSA-Metodologias`.
- Actualización de `binary_to_mseed_context.md` (1044 LOC, complejidad $O(N)$, flags CLI y triple barrera SPI) y creación de `test_binary_to_mseed_context.md` (429 LOC, suite unitaria sintética T1 a T11).
- Generación de la transición técnica semántica [`2026-10-26_contexto-agente.md`](file:///home/rsa/git/montajes/acelerografo-DEV00/docs/progress/2026-10-26_contexto-agente.md) para guiar la resolución del desacople de fecha en la siguiente sesión.

**Scripts/Comandos relevantes:**
```bash
# Procesamiento en lote de archivos reales de CHA02 en DEV00
for f in /home/rsa/data/test-bin/CHA2_*.dat; do
    echo "Procesando $f..."
    .venv/bin/python3 scripts/mseed/binary_to_mseed.py 3 "$f" --output-dir /home/rsa/data/test-bin
done

# Inspección global de trazas generadas con ObsPy
.venv/bin/python3 -c "
import glob, os, obspy
mseed_files = sorted(glob.glob('/home/rsa/data/test-bin/DEV0_*.mseed'))
print('{:<35} | {:<4} | {:<8} | {:<12} | {}'.format('ARCHIVO', 'CH', 'NPTS', 'DURACION (s)', 'RANGO TEMPORAL'))
print('-' * 95)
for mf in mseed_files:
    try:
        st = obspy.read(mf)
        base = os.path.basename(mf)
        for tr in st:
            dur = tr.stats.endtime - tr.stats.starttime
            print('{:<35} | {:<4} | {:<8} | {:<12.3f} | {} -> {}'.format(
                base, tr.stats.channel, tr.stats.npts, dur, tr.stats.starttime, tr.stats.endtime
            ))
    except Exception as e:
        print('Error leyendo {}: {}'.format(mf, e))
"
```
