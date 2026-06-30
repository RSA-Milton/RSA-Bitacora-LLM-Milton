---
fecha: 2026-06-30
temas: [streaming, memoria-compartida, dsp, gpd]
entorno: [acelerografo]
autor: Milton
---

# Actividad del 2026-06-30

**Hitos de la jornada:**
Se completaron de forma exitosa las dos primeras fases del Plan de Implementación de Inferencia GPD en Tiempo Real para el acelerógrafo RSA. Esto incluyó la implementación y validación de una capa de memoria compartida de ultra-baja latencia y un módulo de procesamiento digital de señales optimizado para Machine Learning.

En la Fase 1, se diseñó e integró un publicador y lector de memoria compartida (`/dev/shm/rsa_current_frame`) usando el protocolo Seqlock. Esta arquitectura libre de locks permite que el daemon de adquisición continua (`stream_processor.py`) publique muestras en tiempo real sin riesgo de bloqueos y que los lectores realicen doble lectura de coherencia de forma rápida. Además, se implementó una lógica de auto-reconexión robusta que detecta el cambio de inodo si el segmento es recreado tras un reinicio del escritor.

En la Fase 2, se creó el módulo de preprocesamiento de señal (`signal_preprocessor.py`). Se configuró un downsampling polifásico de 250 Hz a 100 Hz (`resample_poly`), un filtro pasabanda Butterworth (3-20 Hz) de fase cero (`sosfiltfilt`) para conservar la alineación temporal de arribos de ondas sísmicas, y normalización per-channel. Se adoptó la "Opción A (Filtrar con padding)", la cual procesa una ventana de 8 segundos y recorta la porción central de 4 segundos limpia de artefactos de borde. Las pruebas unitarias de ambos componentes fueron ejecutadas en el hardware, resultando aprobadas con éxito en su totalidad.

**Decisiones y Cambios:**
- **Sincronización sin bloqueos (Seqlock)**: Se implementó un contador de secuencia par/impar para sincronizar la memoria compartida entre procesos, evitando el uso de semáforos o Mutexes pesados que degradarían el rendimiento del procesador de adquisición en la Raspberry Pi.
- **Detección de recreación por inode**: Se añadió una verificación de inodo en el lector para asegurar que no quede enganchado a descriptores unlinked si el publicador se reinicia.
- **Filtrado pasabanda con fase cero (`sosfiltfilt`)**: Se optó por un filtrado bidireccional local sobre ventanas temporales para evitar introducir retardos de grupo, lo cual desplazaría artificialmente los picks de fase.
- **Mitigación de efectos de borde (Opción A)**: Se confirmó la alimentación de ventanas de 8 segundos con descarte de extremos de 2 segundos para asegurar que el modelo GPD reciba ondas limpias de transitorios.

**Scripts/Comandos relevantes:**
Para ejecutar las pruebas unitarias y de integración de estas fases en el dispositivo remoto:
```bash
# Ejecutar tests de memoria compartida y Seqlock
python3 scripts/operation/streaming/test_shared_memory.py

# Ejecutar tests del preprocesador de señal (resampling, filtrado y normalización)
python3 scripts/operation/core/test_signal_preprocessor.py
```
