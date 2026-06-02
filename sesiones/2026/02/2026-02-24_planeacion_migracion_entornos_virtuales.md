---
fecha: 2026-02-24
temas: [venv, dependencias, documentacion]
entorno: [acelerografo]
---
# Actividad del 2026-02-24

**Hitos de la jornada:**
Se planificó e inició la migración de la gestión de dependencias Python del proyecto acelerógrafo desde instalaciones globales a un entorno virtual único (`.venv`) usando `--system-site-packages`. Se resolvió la incompatibilidad de dependencias (pyOpenSSL, matplotlib, tflite-runtime). Se documentó el proceso de despliegue y uso del menú de configuración. 

**Decisiones y Cambios:**
- Se creó `requirements.txt` y los scripts de instalación de librerías y de entorno virtual en `scripts/setup/`.
- Se modificaron `registrocontinuo.sh` y `mqtt_coordinator.conf` para apuntar a la ruta del intérprete en el entorno virtual.
- Se agregó documentación sobre dependencias y arquitectura usando diagramas de flujo y arquitectura Mermaid en `README.md` y `AGENTS.md`.

**Scripts/Comandos relevantes:**
```bash
# Script crear_entorno_virtual.sh
python3 -m venv --system-site-packages $PROJECT_LOCAL_ROOT/.venv
source $PROJECT_LOCAL_ROOT/.venv/bin/activate
pip install --upgrade -r requirements.txt
```
---


