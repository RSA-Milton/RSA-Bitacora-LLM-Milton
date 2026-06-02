---
fecha: 2026-04-27
temas: [deploy, scripts, bash]
entorno: [acelerografo]
---
# Actividad del 2026-04-27

**Hitos de la jornada:**
Establecimiento del workspace en el directorio `montajes/acelerografo-DEV01` y corrección de un fallo crítico en el script de despliegue (`deploy.sh`) que impedía la configuración de nuevas estaciones debido a referencias estáticas a archivos inexistentes o renombrados (como `cliente.py`).

**Decisiones y Cambios:**
- Se refactorizó `scripts/setup/deploy.sh` reemplazando la transferencia de scripts Python archivo por archivo por una transferencia dinámica mediante comodines (`*.py`).
- Se aplicó esta lógica a los directorios de componentes `mqtt`, `mseed` y `drive`.
- Se homologó el mecanismo de copia en `deploy.sh` para que se comporte igual que el script `update.sh`.

**Scripts/Comandos relevantes:**
```bash
cp $PROJECT_GIT_ROOT/scripts/operation/mqtt/*.py $PROJECT_LOCAL_ROOT/scripts/mqtt/
cp $PROJECT_GIT_ROOT/scripts/operation/mseed/*.py $PROJECT_LOCAL_ROOT/scripts/mseed/
cp $PROJECT_GIT_ROOT/scripts/operation/drive/*.py $PROJECT_LOCAL_ROOT/scripts/drive/
```
---
