---
fecha: 2026-02-23
temas: [vpn, cloudflare, venv, git]
entorno: [acelerografo]
---
# Registro de Sesión: Conectividad Remota y Organización de Código

# Actividad del 2026-02-23

**Hitos de la jornada:**
Abordamos la crisis de dependencias y compatibilidad del sistema de adquisición sísmica frente a las restricciones de Debian Trixie (PEP 668) y los fallos de librerías (cryptography, OpenSSL) en Raspbian Bullseye. Se descartó la contenerización completa con Docker debido a las limitaciones de hardware de la Raspberry Pi 3B+ (1GB RAM) y la criticidad de la latencia en la lectura del bus SPI a 250Hz. En su lugar, se definió una estrategia táctica basada en el aislamiento estricto de procesos Python mediante Entornos Virtuales (venv), permitiendo la herencia de paquetes del sistema para evitar compilaciones pesadas en arquitecturas ARM.

Se resolvió el reto de conectividad remota para las estaciones de campo ubicadas detrás de firewalls restrictivos (ej. Fortinet institucional y GADs cantonales). Tras diagnosticar que el firewall bloqueaba conexiones VPN P2P (Tailscale) mediante Inspección Profunda de Paquetes (SSL/TLS), realizamos una prueba de concepto exitosa utilizando Cloudflare Tunnels (Zero Trust). El agente logró evadir el bloqueo estableciendo túneles de salida (Outbound) camuflados como tráfico web HTTPS/QUIC.

Finalmente, establecimos el marco de trabajo colaborativo y de control de versiones. Diseñamos un flujo Git adaptado para integrar la migración de entornos virtuales y desacoplar el desarrollo de la nueva funcionalidad de inferencia con Machine Learning (modelo GPD Keras). A nivel de documentación, se estandarizó la estructura del repositorio para la interacción con agentes de Inteligencia Artificial, separando el contexto técnico de los planes de ejecución táctica.

**Decisiones y Cambios:**

Mantener la imagen maestra clonada de Debian Bullseye para el despliegue a corto plazo, posponiendo la migración a Trixie para evitar riesgos al código de adquisición en C dependiente de WiringPi.

Reemplazar las instalaciones globales de Python por un entorno virtual (venv) creado con el flag --system-site-packages, permitiendo aprovechar binarios precompilados vía apt-get (ej. numpy, scipy) y aislando dependencias sensibles (obspy, tensorflow, paho-mqtt).

Adoptar Cloudflare Tunnels (cloudflared) y la adquisición de un dominio propio (ej. .org o .net) como estándar para el acceso SSH remoto, descartando alternativas como Tailscale o túneles inversos clásicos.

Implementar la política estricta de Git "Merge & Delete": las ramas de características (feature/) y tareas (task/) deben ser eliminadas del repositorio local y remoto inmediatamente tras su integración a develop.

Crear el directorio docs/blueprints/ para almacenar exclusivamente los planes de implementación y prompts estructurados para el trabajo con agentes de IA, manteniendo docs/context/ para documentación técnica pasiva.

**Scripts/Comandos relevantes:**

```bash
# Script de creación del Entorno Virtual (Estrategia de herencia de paquetes)
sudo apt-get update
sudo apt-get install -y python3-numpy python3-scipy python3-matplotlib python3-lxml python3-setuptools python3-sqlalchemy libatlas-base-dev

# Crear venv heredando librerías base para evitar compilaciones en ARM
python3 -m venv $PROJECT_LOCAL_ROOT/venv --system-site-packages

# Instalar dependencias críticas de forma aislada
$PROJECT_LOCAL_ROOT/venv/bin/pip install obspy==1.4.0 paho-mqtt==1.6.1 tensorflow==2.12.0 pandas
```

```bash
# Prueba de Concepto (PoC) exitosa de Evasión de Firewall (Cloudflare Tunnels)
# Terminal 1: Servidor web local
python3 -m http.server 8080

# Terminal 2: Agente Cloudflare
curl -L --output cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared.deb
cloudflared tunnel --url http://localhost:8080
```

```bash
# Flujo de Git establecido para la Migración y Modelo GPD
# 1. Integrar migración y eliminar rama (Merge & Delete)
git checkout develop
git merge feature/venv-migration
git push origin develop
git branch -d feature/venv-migration
git push origin --delete feature/venv-migration

# 2. Iniciar nueva tarea de Machine Learning sobre el entorno estabilizado
git checkout -b task/gpd
```

```markdown
# Nueva Estructura de Directorios para IA (docs/organizacion_ia.md)
RSA-Acelerografo/
├── AGENTS.md
├── docs/
│   ├── context/          # Documentación técnica de scripts existentes
│   └── blueprints/       # Planes de acción para nuevas tareas de IA
│       ├── venv_migration.md
│       └── gpd_implementation.md
```