---
fecha: 2026-09-01
temas: [git, deploy, mqtt, automatizacion]
entorno: [acelerografo]
autor: Milton
---

# Actividad del 2026-09-01

**Hitos de la jornada:**
Se abordó el saneamiento de control de versiones y la arquitectura de despliegue para la flota de acelerógrafos de la RSA (`ACEL-DEVP-UNIV-01` / `DEV00`), estableciendo una estrategia limpia para desacoplar el ciclo de desarrollo (`develop`) del código operativo en producción (`main`).

Se definió y aplicó un protocolo de sincronización selectiva hacia `main` mediante `git checkout develop -- ...`, purgando la documentación interna (`docs/`) y reglas del agente (`AGENTS.md`) de los dispositivos de campo, manteniendo en producción únicamente los módulos de ejecución (`scripts/`, `models/`, `configuration/`, `main-libraries/`, `menu.sh`, `requirements.txt`). Asimismo, se saneó el repositorio remoto eliminando masivamente ramas locales huérfanas acumuladas (`task/gpd`) con comandos combinados de poda (`git fetch --prune`, `git reset --hard` y `git branch -D`).

Finalmente, se analizó la viabilidad y seguridad técnica de automatizar las actualizaciones remotas Over-The-Air (OTA) a través del broker MQTT institucional y `mqtt_coordinator.py`, evaluando los riesgos de suicidio de procesos en Supervisor, la necesidad de ejecución desacoplada en nueva sesión, validaciones de compilación con rollback automático y configuración estricta de ACLs en el broker. Todo el análisis fue documentado formalmente en `montajes/acelerografo-DEV00/docs/analysis/diagnostico_automatizacion_despliegue.md`.

**Decisiones y Cambios:**
- Se estableció la política de exclusión de `docs/` y `AGENTS.md` en la rama `main` mediante checkout selectivo desde `develop`.
- Se ejecutó la poda y saneamiento de ramas locales obsoletas en la estación de prueba `ACEL-DEVP-UNIV-01`.
- Se analizó el diseño del mecanismo de actualización remota OTA vía MQTT (`rsa/seismic/smart/{station}/cmd/update`) con subproceso desacoplado (`start_new_session=True`).
- Se redactó el documento técnico `docs/analysis/diagnostico_automatizacion_despliegue.md`.

**Scripts/Comandos relevantes:**
```bash
# Sincronización selectiva de producción desde develop
git checkout main
git checkout develop -- configuration/ main-libraries/ menu.sh models/ README.md requirements.txt scripts/
git commit -m "chore: sincronizar produccion desde develop excluyendo docs y agents"
git push origin main

# Saneamiento rápido de ramas huérfanas en estaciones
git fetch origin --prune
git checkout main
git reset --hard origin/main
git branch | grep -v "main" | xargs git branch -D 2>/dev/null || true
```
