---
fecha: 2026-06-03
temas: [configuracion, unificacion, plantillas, automatizacion]
entorno: [acelerografo]
autor: Milton
---

# Actividad del 2026-06-03

**Hitos de la jornada:**
Se diseñó e implementó un sistema de unificación de configuración para la estación acelerográfica `NOM00` (DEV00) bajo la Alternativa B (Maestro + Hidratación por Plantillas). Esto soluciona la redundancia de datos (identidad de la estación duplicada en tres archivos JSON diferentes) y prepara la arquitectura para un futuro panel web de administración en caliente.

El nuevo flujo aísla por completo el repositorio Git de las modificaciones operacionales de la estación. Para ello, se almacena una configuración maestra única per-estación en la máquina local (`$PROJECT_LOCAL_ROOT/configuracion/configuracion_maestra.json`), la cual alimenta mediante un script de hidratación en Python las plantillas `.template` para generar los archivos de configuración de producción finales.

**Decisiones y Cambios:**
- Creación de [configuracion_maestra.json](file:///home/rsa/git/montajes/acelerografo-DEV00/configuration/configuracion_maestra.json) con metadatos clave de estación, coordenadas, adquisición y Drive.
- Creación de plantillas `.template` para `configuracion_dispositivo.json`, `configuracion_mqtt.json` y `configuracion_mseed.json` en [configuration/](file:///home/rsa/git/montajes/acelerografo-DEV00/configuration/).
- Implementación de [hidratar_configuracion.py](file:///home/rsa/git/montajes/acelerografo-DEV00/scripts/setup/hidratar_configuracion.py) en Python que prioriza la lectura de la configuración local en `$PROJECT_LOCAL_ROOT` para evitar la modificación de archivos bajo control de versiones.
- Integración del script generador en [deploy.sh](file:///home/rsa/git/montajes/acelerografo-DEV00/scripts/setup/deploy.sh) y [update.sh](file:///home/rsa/git/montajes/acelerografo-DEV00/scripts/setup/update.sh).

**Scripts/Comandos relevantes:**
```bash
# Copiar configuración maestra inicial al local root
cp montajes/acelerografo-DEV00/configuration/configuracion_maestra.json $PROJECT_LOCAL_ROOT/configuracion/

# Ejecutar actualización e hidratación automática
bash montajes/acelerografo-DEV00/scripts/setup/update.sh

# Ejecutar hidratación manualmente
python3 montajes/acelerografo-DEV00/scripts/setup/hidratar_configuracion.py
```
