---
fecha: 2026-07-02
temas: [gpd, inferencia, streaming, tflite]
entorno: acelerografo
autor: Milton
---

# Actividad del 2026-07-02

**Hitos de la jornada:**
Se implementó con éxito la Fase 3 del plan de inferencia GPD en tiempo real en el dispositivo acelerógrafo DEV00. La pieza central del desarrollo fue el daemon `gpd_stream_worker.py` que consume tramas de aceleración desde la memoria compartida mediante `SharedMemoryReader`. Este script preprocesa los datos, realiza el downsampling de 250 Hz a 100 Hz, almacena un buffer circular de 8 segundos y ejecuta inferencia a 1 Hz sobre el bloque central de 4 segundos utilizando el runtime de TensorFlow Lite. Las detecciones de arribo de fases P y S se reportan mediante mensajes MQTT bajo el tópico local de la estación.

Se diseñó y ejecutó una suite de pruebas unitarias robusta en `test_gpd_stream_worker.py` que consta de 29 pruebas. Abarca desde la verificación matemática de atenuación del filtro y el remuestreo de sinusoides sintéticas, pasando por la validación del buffer circular deslizante, hasta simular el flujo completo de detección y el temporizador de cooldown de 30 segundos (mecanismo anti-spam). Todos los tests pasaron exitosamente. También se actualizó la plantilla de configuración del acelerógrafo, se generó un documento exhaustivo de contexto técnico y se redactó el ADR-010 para registrar formalmente las decisiones de arquitectura sobre stride, padding del filtro, umbrales y cooldown.

**Decisiones y Cambios:**
- **Inferencia de 1 Hz (Stride de 1 s)**: Se eligió realizar una inferencia por segundo en lugar del stride de 0.1 s del script batch. Esto disminuye significativamente la carga de CPU (a menos del 5% en la Raspberry Pi 3B+).
- **Buffer de 8 segundos (Padding pasabanda)**: Se implementó un buffer de 8 s (800 muestras) en lugar de 4 s, extrayendo las 400 muestras centrales tras el filtrado. Esto previene eficazmente que los artefactos de borde del filtro Butterworth contaminen los onsets de la señal.
- **Configuración dinámica en JSON**: Se integraron parámetros como `umbral_p`, `umbral_s` y `cooldown_s` dentro de `streaming.gpd` en `configuracion_dispositivo.json.template` para su calibración en caliente sin requerir cambios de código en producción.
- **Carga robusta y tolerante a fallos de SHM**: En el arranque, el worker intenta conectarse a la memoria compartida con backoff exponencial. Si no lo logra tras 30 s de reintentos, el script finaliza ordenadamente en vez de crashear, permitiendo una gestión más limpia por parte de Supervisor.

**Scripts/Comandos relevantes:**
Comando para la ejecución de la suite de tests unitarios:
```bash
cd /home/rsa/git/montajes/acelerografo-DEV00/scripts/operation/streaming/
python3 -m pytest test_gpd_stream_worker.py -v
```

Estructura de la llamada al intérprete TFLite para inferencia en tiempo real con 2 hilos:
```python
interpreter = TFLiteInterpreter(model_path=self._model_path, num_threads=self._tflite_threads)
input_details = interpreter.get_input_details()
interpreter.resize_tensor_input(input_details[0]["index"], [1, 400, 3])
interpreter.allocate_tensors()
```
