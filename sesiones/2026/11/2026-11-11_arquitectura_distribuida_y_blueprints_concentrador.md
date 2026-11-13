---
fecha: 2026-11-11
temas: [arquitectura, redes, telemetria, adquisicion]
entorno: [concentrador]
autor: Milton
---

# Actividad del 2026-11-11

**Hitos de la jornada:**
Se dio inicio al rediseño arquitectónico integral del sistema `RSA-Concentrador-Presas`. El sistema histórico operaba como una entidad monolítica centralizada donde un solo concentrador ejecutaba la adquisición directa de sensores de vertedero, el cálculo acústico por correlación cruzada digital, la pasarela inalámbrica Ad-Hoc 802.11 IBSS y la subida a la nube. Durante el análisis del código legado en C (`LeerSensorNivel.c`, `ResetConcentrador.c`), se identificó la obsolescencia técnica de las librerías `libbcm2835` y `wiringPi`, las cuales interactuaban mediante bus SPI propietario hacia una placa HAT basada en el microcontrolador PIC18F25K22.

Se acordó evolucionar hacia una arquitectura jerárquica distribuida por roles (`central`, `periferico`, `gateway_adhoc`), adoptando formalmente las denominaciones **Concentrador Central** para el nodo maestro perimetral y **Concentrador Periférico** para los nodos distribuidos por galería. Se evaluó y aprobó la transición física del bus serie hacia adaptadores industriales USB-RS485 modelo ZK-U485 (chipsets CH340G + MAX485), garantizando soporte nativo en el kernel de Linux (`ch341.ko`) y permitiendo operar un maestro Modbus RTU estándar en Python, eliminando dependencias de bajo nivel en C y pines GPIO propietarios.

Asimismo, se analizó la coexistencia en el acelerógrafo sísmico `CHA2` (Raspberry Pi 3B+ en el margen derecho de la presa). Revisando su arquitectura (`RSA-Acelerografo`), se determinó que CHA2 asumirá el rol `gateway_adhoc` exclusivamente para habilitar la interfaz Ad-Hoc y recolectar fotografías de coordinómetros de ese sector, sin bus RS-485 ni cálculos pesados, asegurando un consumo marginal (< 1% CPU, < 15 MB RAM) con impacto cero en la adquisición sísmica continua.

**Decisiones y Cambios:**
- Adopción de la nomenclatura estándar: Concentrador Central y Concentrador Periférico.
- Aprobación del módulo USB ZK-U485 (CH340G / MAX485) para sustituir la placa HAT SPI con PIC18F25K22.
- Selección de Modbus RTU en Python como protocolo estándar para el bus de sensores en campo.
- Estrategia de aislamiento en CHA2: rol de pasarela Ad-Hoc pura para no interferir con el pipeline sísmico de alta frecuencia.
- Esquema de almacenamiento local resiliente en archivos CSV mensuales rotativos (*Store-and-Forward*).

---

# Actividad del 2026-11-12

**Hitos de la jornada:**
Se formalizó la decisión técnica mediante la redacción y aprobación del **ADR-001** (`001_arquitectura_distribuida_roles_concentradores.md`) y la actualización integral de `docs/context/contexto_concentrador.md` y `README.md`. Se estableció el modelo de seguridad perimetral tipo bastión: el Concentrador Central es el único equipo con salida a internet (NAT con puerto SSH hacia IP pública), mientras que las redes internas de galerías y coordinómetros quedan aisladas de internet para minimizar la superficie de ataque, canalizando la administración mediante `ProxyJump` en OpenSSH.

A partir del análisis de requerimientos con el equipo, se diseñó y formalizó la suite completa de 5 Blueprints de ingeniería en `docs/blueprints/`: Blueprint 1 (Red Ad-Hoc, coordinómetros y respaldo Drive), Blueprint 2 (Telemetría MQTT y broker bridge a VPS), Blueprint 3 (Bus RS-485 Modbus RTU y buffer CSV), Blueprint 4 (Servicio NTP Estrato 1 con GPS/PPS, aislamiento y orquestación de actualizaciones) y Blueprint 5 (Resiliencia operativa, watchdogs y housekeeping de disco adaptados de `RSA-Acelerografo`).

Posteriormente, se examinó la documentación histórica de campo (`Izquierda.pdf`, `Derecha.pdf` y `Coordinómetros.pdf`) y se ejecutaron comprobaciones remotas en `CON01`. Se constató que la red Ad-Hoc opera en Canal 5 (ESSID `REDRSAIOT`) con una topología multihop vertical dentro del ducto de 1 m de diámetro por donde baja el cable del péndulo invertido (2 Raspberry Pi Zero por nivel: eje X y eje Y). Las pruebas en `CON01` demostraron conectividad directa excelente con Nivel 2 (RTT ~8.5 ms) y multihop hacia Nivel 1 vía N2 (RTT ~25 ms). Con estos hallazgos, se refinó el Blueprint 1: se acordó mantener la IP `192.168.10.5` en ambas pasarelas (`CON01` y `CHA2`) para evitar reprogramar las 12 tarjetas MicroSD en campo, se sustituyó el crontab `sleep 60` por un hook `post-up` en `interfaces.d/wlan0`, y se desacopló el ciclo de captura (cada 2 h en Pi Zero) de la extracción (diaria por lotes en el concentrador).

**Decisiones y Cambios:**
- Creación del ADR-001 y actualización de la documentación técnica y diagramas Mermaid del repositorio.
- Redacción de los 5 blueprints de ingeniería con sus respectivos checkpoints de validación.
- Conservación de la IP estática `192.168.10.5` en ambas pasarelas para compatibilidad con las imágenes SD existentes.
- Integración de rutas estáticas multihop mediante directiva `post-up` en `/etc/network/interfaces.d/wlan0`.
- Desacople de periodicidades: captura autónoma cada 2 horas en coordinómetros y descarga diaria automatizada con purga segura.

**Scripts/Comandos relevantes:**
```bash
# Verificación de interfaz Ad-Hoc en concentrador CON01
iwconfig wlan0
# wlan0: ESSID:"REDRSAIOT" Mode:Ad-Hoc Frequency:2.432 GHz Cell: 46:34:D6:59:B2:CB

# Comprobación de tabla de enrutamiento estático multihop
route -n
# 192.168.10.30/31 gw 192.168.10.5 metric 1 (Nivel 2)
# 192.168.10.20/21 gw 192.168.10.30/31 metric 2 (Nivel 1)
# 192.168.10.40/41 gw 192.168.10.30/31 metric 2 (Nivel 3)

# Prueba de ping directo hacia N2-X (8.5 ms)
ping -c 2 192.168.10.31

# Prueba de ping multihop hacia N1-X vía N2-X (Redirect Host, 25.0 ms, TTL=63)
ping -c 2 192.168.10.21
```

---

# Actividad del 2026-11-13

**Hitos de la jornada:**
Se completó la consolidación documental de la sesión técnica. Se generó el documento formal de transición semántica en `docs/progress/2026-11-13_contexto-agente.md` para transferir el contexto de diseño, los entornos, las decisiones y los pendientes al siguiente agente de desarrollo. Se ejecutó el volcado cronológico en la bitácora personal y la actualización de referencias en el índice temático federado del exocortex de la Red Sísmica del Austro.

**Decisiones y Cambios:**
- Emisión de la transición técnica semántica en `docs/progress/`.
- Registro cronológico consolidado (11 al 13 de noviembre) en la bitácora institucional.
- Indexación de la sesión bajo el nuevo entorno `concentrador` y los temas `arquitectura`, `redes`, `telemetria` y `adquisicion`.
