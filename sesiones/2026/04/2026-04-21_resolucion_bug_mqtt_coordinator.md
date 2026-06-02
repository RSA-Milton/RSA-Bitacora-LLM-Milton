---
fecha: 2026-04-21
temas: [mqtt, bugfix, telemetria]
entorno: [acelerografo]
---
## Registro del 2026-04-21 a las 17:03:44

**Resumen Técnico:**
Resolución del bug de falsos reportes de estado `online` y `offline` solapados en el coordinador MQTT. Se descartó el uso de temporizadores pasivos de conexión en favor de verificación estricta de red debido a encolamientos internos engañosos del socket TCP de Linux y la librería paho-mqtt durante cortes físicos de LAN. 

**Contexto de Hardware/Software:**
- Cambios en: `montajes/acelerografo-DEV00/scripts/operation/mqtt/mqtt_coordinator.py`
  - Implementación del callback `on_publish` para exigir Quality of Service 1 (`QoS=1`) y validación bidireccional mediante acuse de recibo explícito del Broker (`PUBACK`) antes de persistir de forma local el estado "online".
  - Implementación de mecanismo de debouncing de 10 segundos en `on_disconnect` para absorber fluctuaciones rápidas o micro-cortes de red antes de registrar el estado "offline" de forma definitiva.
  - Ajuste de firma con `*args, **kwargs` en `on_publish` para garantizar compatibilidad absoluta con el API Paho-MQTT v2, previniendo así errores de tipado (TypeErrors) asíncronos y silenciosos.
  - Bloqueo temprano de duplicación en el Log activando la bandera anti-spam (`userdata["is_disconnected_logged"] = True`) en el momento inmediato de la desconexión inicial.
- Comandos clave: `sudo supervisorctl restart mqtt_coordinator` utilizado para aplicar en caliente los parches del script dentro del ecosistema del demonio supervisor.
---
