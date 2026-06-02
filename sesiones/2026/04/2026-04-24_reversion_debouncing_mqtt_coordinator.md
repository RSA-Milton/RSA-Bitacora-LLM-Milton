---
fecha: 2026-04-24
temas: [mqtt, bugfix, redes]
entorno: [acelerografo]
---
# Actividad del 2026-04-24

**Hitos de la jornada:**
Se resolvió la solicitud de revertir la lógica de debouncing en la capa de aplicación implementada previamente en el coordinador MQTT, tras identificar que los reportes duplicados de estado eran causados por conflictos de enrutamiento entre interfaces (eth0 y wlan0). Adicionalmente, se mitigó un problema persistente de red (Errno 101 - Network is unreachable) durante arranques posteriores a fallos eléctricos, mediante la implementación de un mecanismo de resiliencia en la inicialización de la conexión.

**Decisiones y Cambios:**
- **Reversión de Lógica (Capa 7):** Se eliminó la función `_persistir_offline` y el uso del temporizador (`threading.Timer`) en el callback `on_disconnect`, permitiendo un reporte de desconexión reactivo e inmediato.
- **Simplificación de Callbacks:** Se delegó la validación de entrega QoS 1 íntegramente a Paho-MQTT, suprimiendo la gestión manual de IDs de mensajes (`pending_online_mids`) y las validaciones cruzadas manuales en los callbacks `on_publish` y `on_connect`.
- **Implementación de Resiliencia (Capa 3):** Se modificó la función `iniciar_cliente` para encapsular la instrucción `client.connect()` en un bucle de reintentos infinitos. Se capturó específicamente la excepción `OSError` (Errno 101) implementando una estrategia de espera de exponential backoff, iniciando en 2 segundos y duplicándose hasta un máximo de 60 segundos por intento fallido.

**Scripts/Comandos relevantes:**
```bash
# Comando ejecutado manualmente en el servidor remoto por el usuario para aplicar cambios en caliente:
sudo supervisorctl restart mqtt_coordinator
```
---

