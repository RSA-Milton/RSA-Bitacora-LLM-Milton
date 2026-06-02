---
fecha: 2026-05-15
temas: [mqtt, telemetria, arquitectura, broadcast]
entorno: [acelerografo]
---

# Actividad del 2026-05-15

**Hitos de la jornada:**
Planificación e implementación del soporte para comandos MQTT globales (broadcast). Esto permite a los operadores enviar un único mensaje a un tópico dedicado para que todas las estaciones de la red ejecuten una tarea de manera simultánea (por ejemplo, `extract_event`), minimizando el tráfico en la red y optimizando la coordinación de múltiples dispositivos.

**Decisiones y Cambios:**
- Se adoptó el patrón de **suscripción dual**. Cada estación se suscribe a su tópico individual y a un tópico global (`cmd_broadcast`) cuyo ID es literalmente la palabra `broadcast`.
- Se descartó el uso de wildcards `+` literales en los tópicos de publicación, adhiriéndose a las mejores prácticas de la arquitectura de colas MQTT.
- Se añadió la clave `cmd_broadcast` (`{org}/{app}/{cap}/broadcast/cmd/+`) en `configuracion_mqtt.json` y se incluyó en la lista de `subscriptions`.
- Tras un análisis detallado, se confirmó que el agente `mqtt_coordinator.py` no requería modificaciones lógicas en el código. El handler de extracción asíncrona utiliza `resolver_topico` junto con la configuración local (`config["id"]`) para generar su respectivo tópico de respuesta (ej. `.../NOM00/cmd/extract_event/res`). De esta forma, aunque un comando ingrese a través de la vía global (broadcast), la confirmación de la orden siempre se rutea usando el identificador particular y único de cada estación.

**Scripts/Comandos relevantes:**
```bash
# Publicar un comando broadcast hacia todas las estaciones de la red
mosquitto_pub -t "rsa/seismic/smart/broadcast/cmd/extract_event" -m '{"start":"2026-05-10Z14:30:00","duration":120,"upload":true}'

# Suscribirse de manera general a las respuestas individuales generadas por las estaciones
mosquitto_sub -t "rsa/seismic/smart/+/cmd/extract_event/res"
```
