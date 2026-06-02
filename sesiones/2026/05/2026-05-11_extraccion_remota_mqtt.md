---
fecha: 2026-05-11
temas: [mqtt, mseed, telemetria, scripts]
entorno: [acelerografo]
---
# Actividad del 2026-05-11

**Hitos de la jornada:**
Se implementó la funcionalidad de extracción remota de eventos sísmicos a través de comandos MQTT. Se desarrolló un script orquestador (`event_extractor.py`) que se encarga de ejecutar el extractor de miniSEED (`extract_segment.py`) en un entorno virtual aislado y la subida de los archivos a Google Drive (`subir_archivo.py`) de forma transparente para el broker.

**Decisiones y Cambios:**
- Se decidió aislar la ejecución de la lógica de mseed y drive utilizando subprocesos (`subprocess.run`) en lugar de importaciones directas para evitar conflictos de dependencias (ObsPy requiere `.venv`, mientras que `mqtt_coordinator.py` usa el Python base del sistema).
- Se diseñó el nuevo comando `extract_event` en `mqtt_coordinator.py` de forma asíncrona mediante `threading.Thread` para no bloquear el bucle principal.
- Se agregó el script `test_event_extractor.py` para verificar rutas locales y diagnosticar el entorno de manera aislada antes de conectar el módulo al daemon.
- Se corrigió un error de compatibilidad de tipos (uso de `Optional[str]` en vez de `str | None`) para soportar correctamente las versiones de Python en la Raspberry Pi.


**Scripts/Comandos relevantes:**
```bash
# Prueba del script extractor de forma local sin usar drive
python3 $PROJECT_LOCAL_ROOT/scripts/mqtt/test_event_extractor.py --start "2026-05-10Z14:30:00" --duration 30

# Petición MQTT para extracción y subida
mosquitto_pub -t "rsa/seismic/smart/NOM00/cmd/extract_event" -m '{"start":"2026-05-10Z14:30:00","duration":120,"upload":true,"request_id":"evt-001"}'
```
---
