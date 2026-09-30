---
fecha: 2026-09-28
temas: [pasantias, planificacion, adquisicion, esp32]
entorno: [pasantias]
autor: Milton
---

# Actividad del 2026-09-28

**Hitos de la jornada:**
Se ejecutó una revisión sistemática y auditoría técnica de las 11 planificaciones históricas de prácticas preprofesionales y laborales de la Red Sísmica del Austro (RSA) alojadas en el repositorio institucional, abarcando desde las iniciativas pioneras en la Presa Chanlud (Plan 1) hasta las validaciones recientes de nodos sensores SHM (Plan 11). A partir de este diagnóstico, se identificó el estado de avance, cuellos de botella y líneas de continuidad en cuatro ejes tecnológicos: red distribuida SHM en topología Daisy Chain (dsPIC), acelerógrafo autónomo basado en ESP32 con migración a KiCad 10, instrumentación geotécnica e hidrométrica de presas, y la plataforma de telemetría/observabilidad TIG-MQTT.

Posteriormente, se analizaron tres nuevos documentos técnicos de ideas situados en `docs/ideas`: `informe_arquitectura_firmware.md`, `plan_implementacion_firmware_hardware.md` y `plan_implementacion_sistema_adquisicion.md`. Dicho análisis permitió identificar dos proyectos de alto impacto institucional para auscultación geotécnica de presas: (1) un sistema electromecánico multiplexado de 24 canales para galgas extensométricas de 4 hilos basado en arquitectura distribuida ESP32 (maestro) y PIC16F628A (esclavo con puente H L293D y encoders ópticos), y (2) un sistema de instrumentación virtual DAQ de 16 canales en Python con tarjeta National Instruments NI USB-6210, gobernando 2 canales de puentes de Wheatstone con amplificadores INA114AP y 14 sensores potenciométricos lineales de desplazamiento.

Se procedió a formular y redactar formalmente ambos planes de trabajo bajo el estándar institucional de la RSA (`Plantilla.md`), fijando una duración de 144 horas distribuidas en 10.5 semanas (14 h/semana) con 7 fases estructuradas mediante checkpoints cuantitativos (CP X.X). Los documentos generados fueron `planificacion-12.md` (`RSA-PPP-2026-12`) y `planificacion-13.md` (`RSA-PPP-2026-13`) en el directorio institucional, manteniendo los campos de pasantes y repositorios en estado "Por Asignar".

**Decisiones y Cambios:**
- Auditoría exhaustiva de los 11 planes precedentes y mapeo de sinergias técnicas con los nuevos requerimientos institucionales de auscultación en presas.
- Formalización del Proyecto 1 como `RSA-PPP-2026-12`: Arquitectura distribuida PIC16F628A (control determinista de motores, Homing, encoders y frenado dinámico) + ESP32 (ADC HX711 de 24 bits, relés K1..K5 para alternancia deformación/temperatura con *dead-time* de 20 ms, datalogger SD FAT32/CSV, display LCD y protocolo SCADA RS-232).
- Formalización del Proyecto 2 como `RSA-PPP-2026-13`: Sistema DAQ en Python orientado a objetos (`nidaqmx`, `scipy`, `PyQtGraph`) con configuración desacoplada en YAML/JSON, equilibrado y tara de cero para INA114AP, calibración micrométrica de 14 potenciómetros, filtrado pasabajas de 60 Hz y prueba continua de estrés de 24 horas.
- Estructuración de ambos planes en bloques de 144 horas con matrices de cronograma semanal, criterios de aceptación estrictos y tolerancia a fallos en banco.
- Despliegue de los archivos `planificacion-12.md` y `planificacion-13.md` en `H:\Mi unidad\Pasantias\Planificaciones\planificaciones`.

**Scripts/Comandos relevantes:**
```markdown
# Estructura de Fases - Planificacion 12 (RSA-PPP-2026-12, 144 h):
Fase 1 (15 h / Sem 1): Diagnóstico de hardware, verificación eléctrica y caracterización de encoders en osciloscopio.
Fase 2 (30 h / Sem 2-3): Firmware PIC: Homing absoluto, conteo de ranuras por interrupciones y frenado dinámico L293D.
Fase 3 (15 h / Sem 4): Protocolo serie binario robusto ESP32-PIC a 9600 bps con checksum XOR y máquina de reintentos.
Fase 4 (30 h / Sem 5-6): Driver HX711 (24 bits), secuenciamiento de relés K1..K5 con tiempos muertos y calibración estática.
Fase 5 (24 h / Sem 7-8): Datalogger en MicroSD (FAT32/CSV), menú informativo en LCD y comandos ASCII por RS-232 / SCADA.
Fase 6 (18 h / Sem 9-10): Barrido automático completo de 24 canales (<= 90 s), tolerancia a atascos y prueba de 100 ciclos.
Fase 7 (12 h / Sem 10-11): Guía de troubleshooting, consolidación de anexos, repositorio y redacción del Informe Final.

# Estructura de Fases - Planificacion 13 (RSA-PPP-2026-13, 144 h):
Fase 1 (15 h / Sem 1): Auditoría circuital, cálculo de ganancias de INA114AP, seguridad eléctrica y conexión NI-DAQmx.
Fase 2 (30 h / Sem 2-3): Arquitectura modular en Python (POO), archivo de configuración YAML desacoplado y buffer DAQ.
Fase 3 (15 h / Sem 4): Equilibrado, tara de cero y ecuaciones analíticas de microdeformación para puentes de Wheatstone.
Fase 4 (30 h / Sem 5-6): Adquisición de 14 potenciómetros, análisis de diafonía (cross-talk) y calibración micrométrica.
Fase 5 (24 h / Sem 7-8): Filtro pasabajas contra 60 Hz, detección proactiva de fallas/saturación y persistencia CSV/log.
Fase 6 (18 h / Sem 9-10): Interfaz gráfica en tiempo real (PyQtGraph) y ensayo continuo de estrés de 24 horas.
Fase 7 (12 h / Sem 10-11): Manual de calibración metrológica, guía de troubleshooting, repositorio e Informe Final.
```
---

# Actividad del 2026-09-30

**Hitos de la jornada:**
Se ejecutó el volcado formal de bitácora institucional mediante el skill `volcado_bitacora`, consolidando las actividades de análisis y elaboración de las planificaciones de pasantías `RSA-PPP-2026-12` y `RSA-PPP-2026-13`. Se actualizó el índice maestro federado (`indice_tematico.md`) en el repositorio `RSA-Metodologias`, indexando la nueva sesión tanto en la sección de entornos (`pasantias`) como en las categorías temáticas correspondientes (`pasantias`, `planificacion`, `adquisicion`, `esp32`). Finalmente, se redactaron los mensajes de commit en formato estandarizado para los repositorios involucrados.

**Decisiones y Cambios:**
- Registro cronológico en la bitácora personal de Milton en `institucional/RSA-Bitacora-LLM-Milton/sesiones/2026/09/`.
- Actualización de `indice_tematico.md` incorporando la referencia a `2026-09-28_planificacion_pasantias_multiplexor_daq.md`.
- Generación de mensajes de commit convencionales en minúsculas (`docs:`).

---
