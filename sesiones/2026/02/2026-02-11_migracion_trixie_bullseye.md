---
fecha: 2026-02-11
temas: [migracion, os, dependencias, venv]
entorno: [acelerografo]
---
# Registro de Sesión: Migración de Sistema Operativo y Resolución de Librerías

# Actividad del 2026-02-11

**Hitos de la jornada:**
Análisis de los errores obtenidos al ejecutar el script `instalar_librerias.sh` en un nuevo entorno con Debian 13 (Trixie).
Se redactó un plan de migración (`Plan_de_Migracion_Trixie.md`) que incluye cuatro fases clave.

**Decisiones y Cambios:**
- Identificación de bloqueo del gestor de paquetes por dependencia obsoleta de `wiringpi` (requiere versión antigua de `libc6`).
- Detección de incompatibilidad con pip a causa del PEP 668 en Trixie (`externally-managed-environment`).
- Formulación de dos posibles caminos: compilar `wiringpi` desde un fork o migrar código C a `lgpio`.
- Decisión de sugerir la migración del proyecto Python a un entorno virtual (`venv`).

**Scripts/Comandos relevantes:**
```bash
# Comandos recomendados para la creación del entorno virtual (Fase 2 del plan)
python3 -m venv $PROJECT_LOCAL_ROOT/venv
source $PROJECT_LOCAL_ROOT/venv/bin/activate
pip install paho-mqtt google-api-python-client obspy
```
---

# Actividad del 2026-02-12

**Hitos de la jornada:**
Evaluación de fallos de instalación del script en la versión actual del sistema operativo, Raspbian 11 (Bullseye). Se examinaron múltiples logs con distintos problemas.

**Decisiones y Cambios:**
- Identificación de un fallo crítico: instalación automática de una versión incompatible de `cryptography` (v46.0.5) que rompía por completo `pip3` con el error `AttributeError: module 'lib' has no attribute 'X509_V_FLAG_NOTIFY_POLICY'`.
- Instrucción para limpiar el directorio `/usr/local/lib/python3.9/dist-packages/` y recuperar el `pip3` original del sistema (`apt`).
- Recomendación de instalar paquetes problemáticos usando `--no-deps` para evitar sobreescribir librerías seguras del sistema.
- Decisión final del usuario de adoptar el uso de entornos virtuales (`venv`) para aislar los procesos, en línea con la arquitectura futura, solucionando de paso los problemas presentes en Bullseye.

**Scripts/Comandos relevantes:**
```bash
# Limpieza manual de paquetes conflictivos en el sistema global
sudo rm -rf /usr/local/lib/python3.9/dist-packages/cryptography*
sudo rm -rf /usr/local/lib/python3.9/dist-packages/OpenSSL*
sudo rm -rf /usr/local/lib/python3.9/dist-packages/pyOpenSSL*

# Instalación segura de dependencias pesadas
sudo apt-get install -y python3-scipy python3-matplotlib python3-numpy python3-pandas python3-obspy
```
---

# Actividad del 2026-02-23

**Hitos de la jornada:**
Se generó una síntesis del contexto de toda la migración y resolución de problemas para referencia futura.

**Decisiones y Cambios:**
- Escritura del documento base `/home/rsa/docs/acelerografo/Contexto_migracion_Trixie.md` documentando los impactos en cada componente del proyecto.
- Creación de un artifact de tipo task list para hacer seguimiento de la documentación.

**Scripts/Comandos relevantes:**
```bash
# Ningún comando específico
```
---

# Actividad del 2026-04-22

**Hitos de la jornada:**
Resumen y auditoría final de la conversación siguiendo la Skill de Registro de Sesión.

**Decisiones y Cambios:**
- Creación del Knowledge Item (KI) interno con un resumen de toda la sesión.
- Consolidación de toda la cronología y decisiones de arquitectura en este documento oficial de sesión.

**Scripts/Comandos relevantes:**
```bash
# Creación del directorio para el registro de la sesión
mkdir -p /home/rsa/RSA-Agent-Workspace/sesiones/2026/02/
```
---
