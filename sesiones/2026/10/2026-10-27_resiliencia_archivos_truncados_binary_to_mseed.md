---
fecha: 2026-10-27
temas: [binary_to_mseed, resiliencia, apagones, desacople_timestamp]
entorno: [acelerografo]
autor: Milton
---

# Actividad del 2026-10-27

**Hitos de la jornada:**
Se resolvió la falla en cascada por desacople de timestamp y truncamiento en `binary_to_mseed.py`, detonada durante la sesión previa en el archivo real `CHA2_261001-150002.dat` con la excepción `Error inesperado al procesar CHA2_261001-150002.dat: 'anio'`. El fallo se originaba por una caída abrupta de energía en la estación sísmica durante la hora 15, interrumpiendo la escritura física del archivo en disco y dejando una última trama truncada ($< 2506$ bytes). La función auxiliar `extraer_timestamp_completo_binario` fallaba con una excepción interna al intentar leer la cola del archivo, retornando `(None, None)` e impidiendo que `leer_archivo_binario` inyectara las llaves `'anio'`, `'mes'` y `'dia'` en el diccionario de tiempo validado.

Se implementó el blueprint formal de 4 fases para dotar al motor de conversión de autonomía y resiliencia sismológica absoluta. En la Fase 1, `leer_archivo_binario` adquiere autonomía completa para construir `tiempo_binario_validado` con sus 7 campos requeridos a partir del nombre del archivo (si `usar_fecha_filename=True` según ADR-007) o de la primera trama válida en memoria (`chunk_valido[0]`). En la Fase 2, se desacoplaron las lecturas de inicio y fin en `extraer_timestamp_completo_binario` con bloques `try/except` independientes, garantizando que un truncamiento en la cola nunca invalide el timestamp de inicio ni la conversión de las tramas sísmicas previas al apagón. En la Fase 3, se eliminó el aborto prematuro por ruido en la trama cero en `procesar_archivo_individual`, se agregaron alias cortos `-f`/`-d` en el CLI y se corrigió el enrutamiento de salida en el modo `--dir` (`path_archivo_salida`) para respetar estrictamente `-o` y blindar los directorios institucionales de producción.

En la Fase 4, se implementó el test unitario T12 (`test_archivo_truncado_por_apagon_conserva_fecha_t12`) en `test_binary_to_mseed.py`, certificando que la suite sintética de 13 pruebas unitarias pase al 100% en 0.176 s en DEV00. Posteriormente, se validó el archivo del corte de luz recuperando 52 minutos y 58 segundos de señal acelerográfica continua previa al apagón (794,500 muestras a 250 Hz en 0.83 s), y se procesó el lote completo de 30 archivos en `/home/rsa/data/test-bin` (26 de `CHA02` y 4 de `DEV0`) logrando un 100% de tasa de éxito (30 exitosos, 0 fallidos). Se documentó el ADR-027, se actualizaron los contextos técnicos y se marcó el diagnóstico como resuelto en el índice federado.

**Decisiones y Cambios:**
- Autonomía total de fecha y hora en `leer_archivo_binario`: garantía de estructura completa de 7 campos en `tiempo_binario_validado` independientemente del estado de la cola en disco.
- Aislamiento de bloques en `extraer_timestamp_completo_binario`: colas incompletas ($< 2506$ bytes) por corte de suministro eléctrico no anulan `ts_inicio`.
- Tolerancia a ruido de encendido en trama física 0 en `procesar_archivo_individual`, delegando el filtrado a la memoria.
- Corrección de enrutamiento en bucle de modo directorio (L989): paso de `path_archivo_salida` en lugar de `path_archivos_mseed`, protegiendo `/home/rsa/data/mseed/` y `events/`.
- Incorporación del test unitario T12 (`test_archivo_truncado_por_apagon_conserva_fecha_t12`).
- Aprobación y documentación del ADR-027 en el exocortex institucional y en el repositorio del acelerógrafo.
- Actualización de los contextos técnicos (`binary_to_mseed_context.md`, `test_binary_to_mseed_context.md`) y del diagnóstico técnico `2026-10-26` a estado resuelto.

**Scripts/Comandos relevantes:**
```bash
# Sincronización y ejecución de suite de 13 tests unitarios en DEV00
cd /home/rsa/projects/acelerografo
cp -v /home/rsa/git/RSA-Acelerografo/scripts/operation/mseed/*.py scripts/mseed/
.venv/bin/python3 scripts/mseed/test_binary_to_mseed.py
# Ran 13 tests in 0.176s - OK

# Conversión y certificación del archivo patológico de apagón
.venv/bin/python3 scripts/mseed/binary_to_mseed.py \
  -f /home/rsa/data/test-bin/CHA2_261001-150002.dat \
  -o /home/rsa/projects/acelerografo/tmp-files/test_mseed_resiliencia/

# Conversión masiva en lote del repositorio completo (30 archivos)
.venv/bin/python3 scripts/mseed/binary_to_mseed.py \
  -d /home/rsa/data/test-bin/ \
  -o /home/rsa/projects/acelerografo/tmp-files/test_mseed_resiliencia/ \
  > /home/rsa/projects/acelerografo/tmp-files/test_mseed_resiliencia/logs.tmp 2>&1

# Resultado obtenido:
# Total de archivos: 30
# Exitosos: 30 (100%)
# Fallidos: 0 (0%)
```
