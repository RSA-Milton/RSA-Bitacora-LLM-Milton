---
fecha: 2026-06-16
temas: [mseed, telemetria, streaming]
entorno: [acelerografo]
autor: Milton
---

# Actividad del 2026-06-16

**Hitos de la jornada:**
Se dio inicio al desarrollo del nuevo sistema de streaming en tiempo real y búfer circular para el acelerógrafo DEV00. El objetivo principal de este sistema es almacenar de forma continua tramas binarias crudas de 2506 bytes del sensor y permitir la extracción bajo demanda de segmentos miniSEED sin interrumpir la adquisición continua de datos.

En esta jornada se completó la Fase 1 y la Fase 2 del plan de implementación. La Fase 1 consistió en rediseñar y corregir el decodificador de tramas (`frame_decoder.py`) para adaptarlo a la realidad del hardware, puesto que se detectó que el firmware del dsPIC sobreescribe el byte 0 con el ID de la primera muestra, invalidando la fuente de reloj teórica y desplazando los offsets de la trama. La Fase 2 consistió en implementar el almacén rotativo en disco `RingBufferStore` (`ring_buffer_store.py`) que gestiona archivos `.bin` de 5 minutos, garantizando un límite de tamaño máximo (FIFO) y permitiendo consultas eficientes por rangos de tiempo de forma thread-safe.

Ambas fases fueron completamente validadas mediante pruebas unitarias detalladas (27 tests para el decodificador y 19 tests para el ring buffer) ejecutadas en el equipo remoto.

**Decisiones y Cambios:**
- **Corrección de offsets en tramas:** Las muestras ocupan los bytes 0 a 2499 (donde el byte 0 es el ID de muestra 0), mientras que el timestamp se ubica en los bytes 2500 a 2505. El campo `clock_source` siempre decodifica como `0` por la sobreescritura del firmware.
- **Soporte de fecha dual:** Se añadió soporte para extraer el año, mes y día desde el nombre del archivo del buffer (`usar_fecha_filename=True`) como mitigación al bug de fecha del dsPIC, manteniendo el método tradicional desde la trama como fallback.
- **Implementación de RingBufferStore:** Se estructuró un índice en memoria (`RingFileEntry`) ordenado cronológicamente que se reconstruye automáticamente al iniciar y es protegido por exclusión mutua (`threading.Lock`) para dar soporte concurrente.

**Scripts/Comandos relevantes:**
```bash
# Ejecutar tests de decodificación de tramas (Fase 1)
cd /home/rsa/git/RSA-Acelerografo/scripts/operation/core
/home/rsa/projects/acelerografo/.venv/bin/python3 test_frame_decoder.py

# Ejecutar tests del Ring Buffer Store (Fase 2)
cd /home/rsa/git/RSA-Acelerografo/scripts/operation
/home/rsa/projects/acelerografo/.venv/bin/python3 streaming/test_ring_buffer_store.py
```
---
