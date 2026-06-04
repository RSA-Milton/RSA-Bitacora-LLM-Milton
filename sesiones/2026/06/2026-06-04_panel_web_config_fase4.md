---
fecha: 2026-06-04
temas: [web, flask, configuracion]
entorno: [acelerografo]
autor: Milton
---
# Actividad del 2026-06-04 — Panel Web de Configuración (Fase 4)

**Hitos de la jornada:**
Se completó el desarrollo e integración de la Fase 4 del plan de unificación de configuración del acelerógrafo. Esto incluye la creación de un panel web interactivo basado en Flask (`config_server.py`) y una interfaz de usuario HTML5/CSS3 responsiva y segura. El servidor permite visualizar el estado de sincronización y de los servicios (`registro_continuo` y `mqtt_coordinator`), editar la configuración maestra del acelerógrafo en caliente de forma visual, comparar cambios y aplicarlos de manera segura (desencadenando la hidratación y reiniciando los daemons del sistema).

El diseño de la aplicación cumple rigurosamente con las directrices de seguridad web: sanitización rigurosa de inputs con un esquema de allow-list, protección de escritura mediante file-lock, resiliencia con backups automáticos y rollback ante fallos en la hidratación, y bloqueo estricto del uso de `innerHTML` en el frontend para evitar inyecciones XSS.

**Decisiones y Cambios:**
- **Selección de Flask:** Elegido sobre FastAPI por tener una menor huella de memoria RAM activa en la Raspberry Pi 3B+ (~15MB vs ~45MB).
- **Esquema de Validación de Inputs:** Validación dual en frontend y backend. El backend implementa allow-list y expresiones regulares estrictas por cada campo para evitar cualquier intento de inyección de comandos o alteración de rutas.
- **Resiliencia en Escritura:** Antes de modificar `configuracion_maestra.json`, se crea automáticamente un archivo de respaldo `.bak`. Si la posterior hidratación de los archivos del runtime falla, el servidor realiza un rollback transparente y restaura la configuración previa estable.
- **Control Concurrente y Bloqueo:** Uso de `fcntl.flock` con exclusión mutua para evitar corrupción del JSON en caso de escrituras concurrentes.
- **Cabeceras de Seguridad:** Inyección dinámica de cabeceras HTTP de seguridad (CSP estricta sin inline scripts, `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, y bloqueo de APIs de dispositivos con `Permissions-Policy`).

**Scripts/Comandos relevantes:**
Inicio del servicio web de configuración gestionado por Supervisor (`config_server.conf`):
```ini
[program:config_server]
command=/home/rsa/projects/acelerografo-rsa/.venv/bin/python3 /home/rsa/projects/acelerografo-rsa/scripts/web/config_server.py
directory=/home/rsa/projects/acelerografo-rsa/scripts/web
autostart=true
autorestart=true
user=rsa
environment=PROJECT_LOCAL_ROOT="/home/rsa/projects/acelerografo-rsa",PROJECT_GIT_ROOT="/home/rsa/git/RSA-Acelerografo"
stderr_logfile=/home/rsa/projects/acelerografo-rsa/log-files/config_server.err.log
stdout_logfile=/home/rsa/projects/acelerografo-rsa/log-files/config_server.out.log
```
---
