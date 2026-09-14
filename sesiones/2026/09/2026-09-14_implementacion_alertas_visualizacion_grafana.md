---
fecha: 2026-09-14
temas: [grafana, alertas, mqtt, testing]
entorno: [tig]
autor: Milton
---

# Actividad del 2026-09-14

**Hitos de la jornada:**

Durante la jornada se implementó, depuró y validó en producción la arquitectura completa de visualización centralizada y jerárquica de alertas para la Red Sísmica del Austro en el Stack TIG (`RSA-Intern-TIG-MQTT`), completando las cuatro fases del blueprint previamente diseñado. Se activó en `telegraf.conf` la ingesta de telemetría especializada mediante tres consumidores MQTT dedicados con QoS 1 (`station_acquisition`, `station_sensor`, `station_drive`), ruteando de forma dual hacia el bucket temporal `telemetry` de InfluxDB v2.

En Grafana, se actualizó `seismic_monitor.json` a la versión 8 integrando el motor de votación jerárquico Flux con 7 niveles de severidad (`Offline` > `Adquisicion` > `Sensor` > `Disco` > `Drive` > `Temperatura` = `Memoria` > `OK`), presentando una matriz minimalista de 3 columnas (`Estación`, `Conectividad`, `Diagnóstico Principal`) con color degradado en toda la fila y Data Links directos hacia el dashboard `Health`. Se refinaron las reglas operativas desvinculando el estado `throttled` de las alertas de la matriz (evitando falsas alarmas por fluctuaciones térmicas habituales en las Raspberry Pi) y ajustando el umbral de disco para activarse estrictamente sobre el 90% de uso.

En el dashboard `Health` (versión 9), se solucionó la saturación visual mediante la encapsulación de los 18 paneles analíticos dentro de 4 filas temáticas colapsables por defecto (`Hardware y Sistema`, `Watchdog de Adquisición`, `Integridad del Sensor y Reloj`, `Sincronización con Google Drive`). Para las pruebas de integración (Fase 4), se desarrolló un simulador interactivo en Python con Paho MQTT (`simulador_alertas_mqtt.py`) y su wrapper Bash (`ejecutar_simulador.sh`), implementando una estrategia de publicación simultánea de 5 canales (4 nominales limpios + 1 anómalo) con marcas de tiempo UTC ISO 8601 dinámicas. Esto permitió validar exitosamente los 8 escenarios de prueba sin traslapes ni procesos residuales. Finalmente, se documentaron 4 contextos técnicos, se redactó el ADR-019 y se actualizó la transición técnica.

**Decisiones y Cambios:**

- **Ingesta en Telegraf**: Suscripción dedicada a `status/acquisition`, `status/sensor` y `status/drive` con parseo JSON y tipado estricto.
- **Motor de Votación Jerárquico Flux (`seismic_monitor.json`)**: Agregación con `union()`, ordenamiento descendente y `limit(n: 1)` para resolver un único diagnóstico no ambiguo por estación.
- **Desvinculación de Throttled**: Desacoplado del monitor principal y presentado en `Health` como texto puramente informativo sin alertas en rojo.
- **Calibración de Alertas de Disco**: Activación de alerta de disco únicamente cuando el uso supere el 90% (<10% de espacio disponible).
- **Filas Colapsables en `Health` (`health.json`)**: Estructuración en 4 filas temáticas (`Salud del Sistema`, `Adquisición`, `Sensor`, `Google Drive`) con `"collapsed": true` y panels anidados en `row.panels`. Se mantuvo la expansión manual para preservar recursos y compatibilidad con Grafana v11.
- **Simulador Aislado de 5 Canales (`simulador_alertas_mqtt.py`)**: Publicación conjunta de la línea base completa para erradicar traslapes por mensajes retenidos o tráfico de equipos físicos en campo.
- **Wrapper con Venv Efímero (`ejecutar_simulador.sh`)**: Creación automática de venv en `/tmp/rsa_simulator_venv` y limpieza garantizada con `trap cleanup EXIT`.
- **Memoria Semántica y Gobernanza**: Creación de 4 contextos técnicos (`telegraf_context.md`, `seismic_monitor_context.md`, `health_context.md`, `simulador_alertas_mqtt_context.md`), registro de `ADR-019` y emisión de transición técnica `2026-09-14_contexto-agente.md`.

**Scripts/Comandos relevantes:**

```bash
# 1. Ejecución interactiva guiada del simulador de alertas MQTT
./scripts/testing/ejecutar_simulador.sh

# 2. Ejecución puntual de un escenario específico (ej. Falla de Adquisición en Ring Buffer)
./scripts/testing/ejecutar_simulador.sh -e 4

# 3. Restauración final a estado nominal limpio en la estación de pruebas
./scripts/testing/ejecutar_simulador.sh -e 8 -s TEST

# 4. Verificación de sintaxis y reinicio de contenedores en el servidor
docker compose config
docker compose restart telegraf grafana
```
---
