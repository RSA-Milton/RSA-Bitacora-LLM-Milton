---
fecha: 2026-09-23
temas: [pasantias, shm, dspic, planificacion]
entorno: [pasantias]
autor: Milton
---

# Actividad del 2026-09-23

**Hitos de la jornada:**
Se realizó la auditoría técnica exhaustiva de los informes y antecedentes del proyecto de Ensamblaje y Validación de la Red de Monitorización de Salud Estructural (SHM - Acelerógrafo V1.4). En primer término, se revisaron los documentos de contexto `Planificacion 1.pdf` (ensamblaje general), `Planificacion 2.pdf` (prueba de concepto de almacenamiento MicroSD), `Descripcion libreria SD.pdf` e informe técnico previo `Validacion SHM RS485.pdf`.

A partir de la confrontación entre la Planificación 2 y los resultados experimentales documentados en el informe técnico, se identificó que la infraestructura de hardware, el cableado UTP T-568B en Daisy Chain, la comunicación RS485 a 2 Mbps y la sincronización física mediante transceptores diferenciales quedaron plenamente validados (retardos de 1.2 µs Concentrador-Nodo y 14.0 ns entre Nodos A y B). Sin embargo, el subsistema de almacenamiento en MicroSD quedó bloqueado en hardware: el zócalo soldado en la PCB carecía físicamente del pin mecánico de detección de tarjeta (*card-detect*). Pese a mitigar el problema por software forzando la bandera `sdflags.detected = 1`, la inicialización y escritura de bloques crudos de 512 bytes no operó con repetibilidad ni confiabilidad.

Con base en este diagnóstico, se elaboró una propuesta inicial de trabajo para subsanar los bloqueos y abordar la integración pendiente del acelerómetro triaxial ADXL355.

**Decisiones y Cambios:**
- Auditoría fase por fase de `Planificacion 2.pdf` frente a los resultados de `Validacion SHM RS485.pdf`, catalogando hitos completados y causas de desvío.
- Identificación de la ausencia de pin físico de *card-detect* en los zócalos MicroSD como causa raíz del fallo en la escritura de sectores crudos.
- Decisión de estructurar un nuevo plan de trabajo enfocado en superar el almacenamiento local e integrar el sensor de aceleración.

---

# Actividad del 2026-09-24

**Hitos de la jornada:**
Se discutió y reestructuró estratégicamente la nueva planificación de trabajo para el proyecto de red de monitorización SHM, delimitando un alcance de 144 horas de dedicación distribuidas en bloques semanales de 14 horas (~10.5 semanas). Para garantizar una revisión continua y avances cuantificables, se establecieron checkpoints medibles para cada entrega semanal/quincenal.

A nivel de hardware y alcance del proyecto, se tomó la decisión de reemplazar los zócalos MicroSD por componentes que sí cuenten con el contacto físico de *card-detect*, y posponer la compleja interfaz SPI entre el Concentrador y la Raspberry Pi para un ciclo de desarrollo posterior. El esfuerzo se enfocó al 100% en consolidar la autonomía y funcionalidad de los Nodos Sensores: inicialización y escritura cruda en tarjetas Kingston y SanDisk verificadas con HxD, recepción de marcas de tiempo vía RS485, implementación de la arquitectura de Doble Búfer (Ping-Pong) en RAM de 512 bytes para evitar bloqueos durante la escritura flash ante pulsos de sincronismo, e integración del acelerómetro triaxial ADXL355 utilizando la librería existente del proyecto.

Para la validación experimental final del sincronismo y la persistencia de datos, se diseñó un ensayo de co-localización (*shake/tap test*) situando dos nodos sensores sobre la misma superficie rígida sometidos a perturbaciones mecánicas periódicas controladas (golpes cada 5 s). Asimismo, se programó el desarrollo de una herramienta en Python para PC encargada de volcar sectores crudos desde las tarjetas MicroSD y graficar series de tiempo alineadas por intervalos, permitiendo verificar visual y cuantitativamente la ausencia de desfases o pérdidas de paquetes. Finalmente, se redactó y generó formalmente el documento `Planificacion 3.md` en el directorio de trabajo del proyecto.

**Decisiones y Cambios:**
- Adopción de una duración total de 144 horas estructurada en 7 fases semanales con checkpoints cuantificables para revisión.
- Retrabajo físico de hardware: soldadura de nuevos zócalos MicroSD con contacto mecánico de *card-detect* en ambos nodos sensores.
- Desacoplamiento de la interfaz Raspberry Pi-Concentrador para asegurar la entrega de nodos sensores 100% autónomos y funcionales.
- Integración de arquitectura de Doble Búfer (Ping-Pong) en memoria RAM del dsPIC33EP256MC202 para desacoplar la interrupción de sincronismo (INT1) de la latencia de escritura en sectores de 512 bytes.
- Fusión de las tareas del acelerómetro ADXL355 en una fase quincenal (28 h) aprovechando la librería SPI existente.
- Diseño del ensayo experimental de co-localización con golpes periódicos y desarrollo de software en Python para volcado y graficación por intervalos como criterio de aceptación final.
- Redacción y despliegue del documento formal `Planificacion 3.md` en el repositorio del proyecto.

**Scripts/Comandos relevantes:**
```markdown
# Resumen de Fases y Checkpoints - Planificacion 3 (144 h):
Fase 1 (14 h / Sem 1): Adecuación de hardware, zócalos con card-detect y consumo eléctrico.
Fase 2 (28 h / Sem 2-3): Inicialización, escritura y lectura cruda en SD (Kingston/SanDisk) y HxD.
Fase 3 (14 h / Sem 4): Transmisión de tiempo Concentrador-Nodos por RS485, guardado en SD y HxD.
Fase 4 (28 h / Sem 5-6): Doble búfer (Ping-Pong), trama 512 B con datos sintéticos y HxD.
Fase 5 (28 h / Sem 7-8): Integración del ADXL355, doble búfer con datos reales y guardado en SD.
Fase 6 (20 h / Sem 9-10): Suite en Python (volcado/graficación) y validación por co-localización.
Fase 7 (12 h / Sem 10-11): Guía de troubleshooting, anexos y redacción del Informe Técnico Final.
```
---
