---
fecha: 2026-07-29
temas: [pasantias, shm, microc, dspic]
entorno: [pasantias]
autor: Milton
---

# Actividad del 2026-07-29

**Hitos de la jornada:**
Se estructuró, organizó y puso en marcha el nuevo repositorio de trabajo `RSA-Intern-Ensamblaje_SHM` destinado al pasante encargados del plan de trabajo para el ensamblaje, programación y validación de la red de monitorización de salud estructural (SHM). En primer lugar, se revisó el plan de trabajo de 96 horas y la memoria técnica de diseño de circuitos impresos (`InformeFinal_JhonatanCambisaca.pdf`), identificando los requerimientos de fabricación de hardware (1 Nodo Concentrador y 2 Nodos Sensores con dsPIC33EP256MC202), la interconexión en cascada (Daisy Chain) bajo norma T-568B y los criterios de evaluación comparativa de ruido entre la versión con plano de masa y sin plano de masa.

Posteriormente, se realizó una selección y auditoría de los archivos necesarios desde el repositorio base `SaludEstructuralCS`, identificando las librerías de firmware en MikroC (`ADXL355_SPI.c`, `RS485.c`, `sdcard.c`, `TIEMPO_GPS.c`, `TIEMPO_RTC.c`), los códigos fuente para los microcontroladores (`concentrador.c`, `NodoAcelerometro.c`), los scripts de procesamiento/prueba en C y Python (`LeerAceleracion_V40.c`, `SincronizarTiempoSistema.c`, `LeerSD.py`, `GraficarRegistroContinuo.py`) y los documentos esquemáticos de referencia.

Finalmente, se creó el archivo de documentación principal `README.md` dentro del nuevo repositorio `RSA-Intern-Ensamblaje_SHM`, explicando en detalle los objetivos, la estructura modular de directorios (`docs/`, `firmware/`, `results/`), el plan de trabajo por fases y los requisitos del entorno de desarrollo. Adicionalmente, se redactaron las instrucciones Git para la creación de la rama `dev-pasantias` y la gestión de accesos para colaboradores en GitHub.

**Decisiones y Cambios:**
- Selección del nombre estandarizado `RSA-Intern-Ensamblaje_SHM` según el patrón de nomenclatura institucional `RSA-Intern-Nombre_proyecto`.
- Organización de la estructura de archivos en el repositorio dividida en `docs/` (documentación y esquemáticos), `firmware/` (código fuente MikroC para dsPIC33EP256MC202) y `results/` (entregables finales del pasante).
- Redacción del archivo `README.md` explicativo siguiendo los lineamientos técnicos del proyecto.
- Definición de la estrategia de ramificación colaborativa mediante la rama `dev-pasantias` y asignación de permisos de colaborador en GitHub.

**Scripts/Comandos relevantes:**
```bash
# Creación e inicialización de la rama de desarrollo para pasantías
git checkout -b dev-pasantias
git push -u origin dev-pasantias
```
---
