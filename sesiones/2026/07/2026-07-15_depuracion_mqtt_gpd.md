---
fecha: 2026-07-15
temas: [gpd, mqtt, bugfix, pipeline]
entorno: [acelerografo]
autor: Milton
---

# Actividad del 2026-07-15

**Hitos de la jornada:**
Se resolvió por completo el fallo de integración en tiempo real entre el worker de inferencia GPD (`gpd_stream_worker.py`) y el coordinador de red (`mqtt_coordinator.py`) en modo `online`. Se logró establecer una conexión MQTT segura y autenticada desde el worker hacia el broker externo (`174.138.41.251`), y se adaptó la nomenclatura de los tópicos para cumplir con la jerarquía institucional de la Red Sísmica del Austro (RSA). Esto permitió reactivar con éxito el flujo automático de detección, extracción de datos miniSEED (.mseed) y respaldo en Google Drive.

**Decisiones y Cambios:**
- **Autenticación en MQTT**: Se integró la carga de variables de entorno mediante `python-dotenv` y se configuró al cliente MQTT del worker para aplicar credenciales (`username` y `password`) con compatibilidad nativa para Paho-MQTT v2.x.
- **Resolución de Station ID**: Se corrigió el bug de lectura de configuración en el script del worker, apuntando a `dispositivo.id` en lugar del nivel raíz.
- **Jerarquía de Tópicos RSA**: Se implementó la resolución del tópico dinámico a partir de `configuracion_mqtt.json`, asegurando que el worker publique en la ruta `{org}/{app}/{cap}/{station_id}/events/detected` (ej: `rsa/seismic/smart/DEV0/events/detected`) para alinearse con la suscripción del coordinador.
- **Validación del Flujo Completo**: Se verificó la correcta extracción asíncrona de miniSEED (`DEV0_20260713_215813.mseed`), su subida a Drive y la actualización de los registros locales en `YYYY-MM_detecciones.csv` marcando el evento como `confirmado=True`.

**Scripts/Comandos relevantes:**
```bash
# Sincronizar cambios modificados del repositorio al entorno de ejecución local
bash scripts/setup/update.sh

# Reiniciar el daemon del worker GPD administrado por Supervisor
sudo supervisorctl restart gpd_worker

# Monitorear logs de inicio del worker y verificación de conexión MQTT
tail -n 20 /home/rsa/projects/acelerografo/log-files/gpd_stream_worker.log

# Suscripción de depuración en Broker MQTT para verificar confirmación
# Canal: rsa/seismic/smart/DEV0/cmd/extract_event/res
```
---
