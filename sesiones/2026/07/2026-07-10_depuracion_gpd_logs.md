---
fecha: 2026-07-10
temas: [gpd, logging, dependencias, bugfix]
entorno: [acelerografo]
autor: Milton
---

# Actividad del 2026-07-10 al 2026-07-14

**Hitos de la jornada:**
Durante este período de trabajo continuo, se logró depurar y operacionalizar con éxito el worker de inteligencia artificial de la Fase 5 del pipeline GPD (`gpd_stream_worker.py`) en la Raspberry Pi de la estación `acelerografo-DEV00`. Se solucionaron problemas críticos que bloqueaban por completo su arranque y hacían que las inferencias retornaran probabilidades constantes redundantes.

El primer hito consistió en resolver un conflicto de dependencias de NumPy en el entorno virtual de producción que impedía la inicialización de `tflite-runtime`. Posteriormente, se diagnosticó que los resultados constantes de las inferencias eran causados por un offset de DC masivo (aceleración de la gravedad terrestre en el eje Z) y un bug de compatibilidad del motor de remuestreo `resample_poly` de SciPy con arrays enteros (`int32`) en la arquitectura ARM. Finalmente, se optimizó el esquema de logs del demonio para evitar el crecimiento excesivo de archivos al reducir la verbosidad por defecto a `INFO`, habilitando opciones para activar `DEBUG` por CLI o JSON.

**Decisiones y Cambios:**
- **Restricción de Versión de NumPy**: Se fijó `numpy<2.0.0` en `requirements.txt` para garantizar la compatibilidad binaria del intérprete de TFLite con ObsPy.
- **Remoción de Media (Demean)**: Se agregó la supresión de la media de la señal en `SignalPreprocessor.prepare_window()` para erradicar las perturbaciones transitorias del filtro IIR provocadas por la aceleración gravitacional.
- **Remuestreo en Float64**: Se añadió una conversión de tipos explícita a `float64` de las muestras crudas en `SignalPreprocessor.resample_frame()` para evadir el fallo de remuestreo en procesadores ARM.
- **Control de Logs**: Se añadió el parámetro `"debug": false` en `configuracion_dispositivo.json.template` y la bandera `--debug` en `gpd_stream_worker.py` para regular la generación de trazas.

**Scripts/Comandos relevantes:**
```bash
# Aplicar cambios en el equipo remoto
bash scripts/setup/update.sh

# Ejecutar diagnóstico de compatibilidad de remuestreo
.venv/bin/python3 scripts/streaming/check_preprocessor.py

# Arrancar worker en modo debug
.venv/bin/python3 scripts/streaming/gpd_stream_worker.py --debug

# Reiniciar servicios de Supervisor
sudo supervisorctl restart gpd_worker
```
---
