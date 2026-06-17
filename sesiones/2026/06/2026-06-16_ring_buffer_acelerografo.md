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

# Actividad del 2026-06-17

**Hitos de la jornada:**
Se implementó, probó y desplegó con éxito la Fase 3 del plan de implementación: el daemon procesador de flujo continuo (`stream_processor.py`), que lee tramas binarias de 2506 bytes del named pipe `/tmp/my_pipe` y las persiste secuencialmente en el almacén rotativo en disco (`RingBufferStore`).

Durante la validación y despliegue real en producción, se resolvieron tres desafíos críticos:
1. **Manejo Seguro de Señales en Entornos Multihilo**: Las llamadas a `signal.signal()` arrojaban un error `ValueError` debido a que CPython restringe el registro de señales únicamente al hilo principal. Esto se solucionó validando que `threading.current_thread() is threading.main_thread()` antes de registrar manejadores de señales, permitiendo una parada limpia del daemon a través de un `threading.Event` en subprocesos.
2. **Lecturas Bloqueantes en Named Pipe**: Por defecto, abrir el pipe FIFO y llamar a `os.read()` causaba bloqueos indefinidos que impedían que el bucle de ejecución respondiera al flag de parada. Esto se resolvió abriendo el descriptor de archivo con los flags `os.O_RDWR | os.O_NONBLOCK` y controlando la excepción `BlockingIOError` mediante esperas controladas (`time.sleep(0.005)`).
3. **Denegación de Permisos del Named Pipe en Producción**: Al ejecutarse el daemon de Python bajo el usuario `rsa`, el intento de apertura en modo `O_RDWR` fallaba con `PermissionError` porque el pipe `/tmp/my_pipe` es creado por `registro_continuo` (que corre como `root`) con permisos heredados restrictivos por su `umask`. Se corrigió modificando el programa C `registro_continuo_4.5.0.c` para que ejecute una llamada explícita a `chmod(PIPE_NAME, 0666)` tras la creación/verificación del pipe, garantizando de raíz el acceso en lectura y escritura para cualquier lector.

Adicionalmente, se preparó y actualizó el flujo de despliegue en producción:
* Se creó la plantilla de configuración de Supervisor [[stream_processor.conf](file:///home/rsa/git/montajes/acelerografo-DEV00/scripts/task/stream_processor.conf)].
* Se modificó el script de actualización en producción [[update.sh](file:///home/rsa/git/montajes/acelerografo-DEV00/scripts/setup/update.sh)] para automatizar la copia de los directorios `core/` y `streaming/` a la ruta local y configurar/cargar el daemon en Supervisor de forma autónoma.
* Se validó el funcionamiento del demonio en producción (`sudo supervisorctl status` → `stream_processor` en estado `RUNNING`) comprobando la escritura activa de archivos `.bin` de 751,800 bytes (exactamente 5 minutos/300 tramas) en `/home/rsa/data/ring-buffer`.

**Decisiones y Cambios:**
- **Lectura FIFO No Bloqueante (Opción B del diseño):** Uso de `os.O_RDWR | os.O_NONBLOCK` para prevenir bloqueos persistentes en lecturas de named pipes vacíos.
- **Registro Condicional de Señales:** Restricción de `signal.signal` al hilo principal, usando variables de control internas (`threading.Event`) para la señalización entre hilos.
- **Permisos Abiertos en Pipe (0666):** Forzar permisos del FIFO a nivel de C para permitir comunicación segura entre procesos inter-usuario (root -> rsa).
- **Integración de Despliegue en update.sh:** Copia de módulos compartidos y registro de Supervisor en la hidratación de scripts de despliegue.

**Scripts/Comandos relevantes:**
```bash
# Comprobar estado del servicio daemon en Supervisor
sudo supervisorctl status stream_processor

# Monitorear la escritura de tramas en el directorio del ring buffer
ls -lh /home/rsa/data/ring-buffer/
```
---


