---
fecha: 2026-07-07
temas: [gpd, mqtt, configuracion, telemetria]
entorno: [acelerografo]
autor: Milton
---

# Actividad del 2026-07-07 al 2026-07-10

**Hitos de la jornada:**
Durante esta sesión se completó e integró la **Fase 5 del pipeline GPD** (Configuración, Supervisor y Pruebas de Integración) para la estación `DEV0` (`NOM0` en desarrollo). Esto incluyó la creación del archivo de definición de Supervisor para el daemon `gpd_stream_worker.py`, permitiendo que el proceso corra en segundo plano con persistencia y tolerancia ante la latencia de carga del modelo TensorFlow Lite en la CPU de la Raspberry Pi.

Asimismo, se resolvió una discrepancia de integración crítica relacionada con el formato de fecha/hora de inicio (`start_str`) del evento. Los componentes nuevos formateaban los timestamps en el estándar ISO8601 clásico (`YYYY-MM-DDTHH:MM:SS.mmmZ`), mientras que el núcleo de extracción heredado del acelerógrafo requería un formato propietario con `'Z'` intermedia (`YYYY-MM-DDZHH:MM:SS.mmm`). Esta discrepancia causaba fallos en el parsing de la función `strptime()`, bloqueando la generación de archivos MiniSEED. La solución consistió en adaptar el formateador de fecha en la frontera de integración de los nuevos procesos.

Por último, se validó el comportamiento del pipeline asíncrono y la base de datos CSV thread-safe en el hardware de producción mediante un protocolo de pruebas exhaustivo en 4 etapas, simulando detecciones locales GPD y comandos remotos de red (`network_cmd`) mediante MQTT Explorer. Todas las pruebas resultaron exitosas, confirmando la estabilidad del sistema.

**Decisiones y Cambios:**
* **Creación de `gpd_worker.conf`**: Definición de la tarea en Supervisor con `startsecs=10` para tolerar la carga inicial de `gpd.tflite` e inicialización de librerías.
* **Automatización del Despliegue**: Modificación de `update.sh` para copiar y templatedizar la configuración de Supervisor de `gpd_worker` y copiar de forma automática el modelo `gpd.tflite` a producción (`$PROJECT_LOCAL_ROOT/models/`).
* **Resolución de Formato de Timestamp (ADR-012)**: Ajuste en `mqtt_coordinator.py` y `gpd_stream_worker.py` para generar la fecha de inicio en el formato con `'Z'` intermedia esperada por el extractor.
* **Robustez en Logging**: Se añadió el método `debug(self, msg)` en `StructuredLogger` para cumplir con la interfaz del logger nativo de Python y evitar excepciones de atributo no encontrado (`'StructuredLogger' object has no attribute 'debug'`). Se modificó `event_extractor.py` con `hasattr()` para degradar graciosamente si se le pasan loggers incompletos.
* **Búsqueda Adaptativa de Configuración**: Se corrigió `gpd_stream_worker.py` para que busque la configuración de forma flexible tanto en el directorio `configuracion/` (producción) como en `configuration/` (desarrollo/Git).

**Scripts/Comandos relevantes:**

* Comando de servicio para el worker GPD en `gpd_worker.conf`:
```ini
[program:gpd_worker]
command={{PROJECT_LOCAL_ROOT}}/.venv/bin/python3 {{PROJECT_LOCAL_ROOT}}/scripts/streaming/gpd_stream_worker.py
directory={{PROJECT_LOCAL_ROOT}}/scripts/streaming/
environment=PROJECT_LOCAL_ROOT="{{PROJECT_LOCAL_ROOT}}"
autostart=true
autorestart=true
startretries=3
startsecs=10
user=rsa
```

* Simulación de comandos de red externos desde MQTT Explorer:
```bash
# Publicar comando de red en: rsa/seismic/smart/DEV0/cmd/extract_event
# Payload:
{
    "start": "2026-07-09Z19:18:00.000",
    "duration": 120,
    "upload": false,
    "delete_after_upload": false,
    "request_id": "test-manual-001"
}
```

* Validación de registros en el CSV mensual:
```bash
cat /home/rsa/data/eventos-detectados/2026-07_detecciones.csv
# Salida:
timestamp_centro,fase,probabilidad,timestamp_local,confirmado,archivo_mseed,metodo
2026-07-09T19:19:00.000Z,P,0.0,2026-07-09T21:30:33.749Z,True,DEV0_20260709_191800.mseed,local_gpd
2026-07-09Z19:18:00.000,EXTERNAL,0.0,2026-07-10T17:36:59.091Z,True,DEV0_20260709_191800.mseed,network_cmd
```
