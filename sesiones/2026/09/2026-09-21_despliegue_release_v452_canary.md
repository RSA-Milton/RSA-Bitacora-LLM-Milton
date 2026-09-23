---
fecha: 2026-09-21
temas: [deploy, git, diagnostico, automatizacion]
entorno: [acelerografo]
autor: Milton
---

# Actividad del 2026-09-21 (Despliegue de Release v4.5.2, Aislamiento de Docs y Despliegue en Campo CHA01/CHA02)

**Hitos de la jornada:**

Tras completarse exitosamente una prueba de remojo (*Soak Test*) de 3 días en el banco de pruebas `TEST-01` que certificó la estabilidad del latido periódico (*heartbeat*) de `telemetry/state` y la resiliencia LWT en reconexiones MQTT (desarrollados originalmente en la rama `feat/telemetry` e integrados en `develop`), se procedió con la ejecución formal del protocolo institucional de despliegue y release (`protocolo-despliegue-release.md`) para promover la versión a la rama de producción `main`.

Se definió la versión correspondiente bajo SemVer como **`v4.5.2`** (incremento de tipo `PATCH`), sellando el registro en `CHANGELOG.md`. Con el objetivo de proteger la memoria flash limitada de las estaciones de campo, se refinó la secuencia de sincronización hacia `main` excluyendo deliberadamente el directorio de desarrollo `docs/`. Al detectarse que versiones previas habían subido `docs/` a `main`, se saneó el índice mediante `git rm -r docs/`, confirmando los cambios con la convención institucional de commits (`chore:`) y re-generando el tag oficial inmutable `v4.5.2` tanto a nivel local como en el repositorio remoto.

Posteriormente se ejecutó el despliegue escalonado en campo en las estaciones piloto:
1. **Estación CHA01**: Al reiniciar el pipeline con `registrocontinuo start`, se produjo una excepción `PermissionError: [Errno 13] Permission denied` sobre `/home/rsa/projects/acelerografo/log-files/gestor_acq.log`. El diagnóstico determinó que ejecuciones previas ejecutadas con `sudo` habían fijado el archivo con propietario `root:root`, impidiendo la escritura por parte del usuario operativo `rsa`. Se resolvió restaurando la titularidad y permisos con `chown -R rsa:rsa` sobre `log-files/`.
2. **Estación CHA02**: Se aplicó la actualización a `v4.5.2`, comprobando el arranque limpio de `rsa-acelerografo.service` y la sincronización dinámica instantánea de reloj vía `wait_for_ntp`. En la auditoría de telemetría del sensor, el coordinador MQTT registró advertencias recurrentes `[SENSOR_ANOMALY]` en el eje X (`ax_out_of_range ≈ 0.637 g` a `0.641 g`) con ejes Y y Z nominales, identificando una anomalía de nivelación física o calibración en sitio que quedó catalogada para supervisión operativa.

**Decisiones y Cambios:**

- **Release SemVer v4.5.2**: Promoción formal de mejoras de resiliencia MQTT (ADR-023) desde `develop` a `main`.
- **Aislamiento de `docs/` en Producción**: Eliminación del directorio `docs/` en la rama `main` y en el tag `v4.5.2` para economizar espacio flash en hardware remoto, manteniéndolo íntegro y versionado en `develop`.
- **Saneamiento de Permisos de Logs**: Regularización de permisos y propiedad `rsa:rsa` en `$PROJECT_LOCAL_ROOT/log-files/` para garantizar la ejecución no privilegiada del gestor de archivos de adquisición (`gestor_archivos_acq.py`).
- **Detección de Desviación en CHA02**: Diagnóstico y registro de comportamiento anómalo en el eje X del acelerómetro triaxial de la estación `CHA02`.

**Scripts/Comandos relevantes:**

```bash
# 1. Saneamiento de docs/ en main y etiquetado inmutable de release
git checkout main
git rm -r docs/
git commit -m "chore: eliminar directorio docs de la rama main
- Se removió docs/ para optimizar almacenamiento en estaciones de campo.
- La documentación técnica permanece preservada e íntegra en develop."
git push origin main

# Re-generar tag oficial v4.5.2
git tag -d v4.5.2
git push origin --delete v4.5.2
git tag -a v4.5.2 -m "Release v4.5.2: Latido periódico para telemetry/state, resiliencia LWT y mitigación de desincronización MQTT"
git push origin v4.5.2

# 2. Corrección de permisos de logs en estación remota (ej. CHA01)
sudo chown -R rsa:rsa /home/rsa/projects/acelerografo/log-files/
sudo chmod -R u+rwX /home/rsa/projects/acelerografo/log-files/
registrocontinuo restart

# 3. Verificación de diagnóstico de sensor en CHA02
comprobar
tail -n 25 $PROJECT_LOCAL_ROOT/log-files/mqtt_coordinator.log
```
---

# Actividad del 2026-09-22 (Blindaje de Permisos en update.sh y Skill Interactiva despliegue_release en RSA-Agent-Toolkit)

**Hitos de la jornada:**

Tras confirmarse el despliegue satisfactorio de la versión `v4.5.2` en todas las estaciones acelerográficas de la red, se evaluaron estrategias para estandarizar y blindar el flujo de releases futuros. Se diseñó un plan de implementación estructurado en un artifact de arquitectura (`plan_despliegue_estandarizado.md`), el cual fue iterado y refinado incorporando retroalimentación técnica: se descartaron scripts remotos autónomos y orquestaciones centralizadas complejas, optando por una solución desacoplada, asistida y libre de riesgos basada en una skill interactiva alimentada por el archivo de intercambio `logs.tmp`.

En la primera fase técnica, se resolvió de raíz la anomalía de permisos en los archivos de registro (`PermissionError: [Errno 13]` en `gestor_acq.log`). En `scripts/setup/update.sh` se implementó una rutina incondicional de saneamiento al término de la actualización que detecta al usuario operativo (`rsa` vía `$SUDO_USER` o comprobación de sistema) y aplica `chown -R` y `chmod -R u+rwX,g+rwX` sobre `$PROJECT_LOCAL_ROOT/log-files/`. La solución fue validada en hardware real en la estación `DEV00`, verificando mediante `logs.tmp` que la totalidad de los archivos y subdirectorios de log quedaron con titularidad homogénea `rsa:rsa` y permisos de lectura/escritura íntegros.

Posteriormente, en la segunda y tercera fases, se creó e integró la nueva skill [`despliegue_release.md`](rsa/RSA-Agent-Toolkit/.agents/skills/despliegue_release.md) en el repositorio institucional `rsa/RSA-Agent-Toolkit`. La habilidad asiste interactivamente al desarrollador leyendo el estado del repositorio y resultados de pruebas desde `logs.tmp`, calculando de forma justificada el incremento SemVer (`PATCH`, `MINOR`, `MAJOR`), redactando la entrada para `CHANGELOG.md` bajo *Keep a Changelog*, entregando los bloques de comandos Git para `DEV00` con exclusión forzada de `docs/` en `main`, y generando los comandos textuales para copiar y pegar vía SSH en estaciones remotas sin violar la restricción SSHFS. La jornada culminó registrando la skill en `AGENTS.md`, `README.md` y `sincronizar_toolkit.md` de `RSA-Agent-Toolkit`, y sincronizándola hacia el workspace raíz en `git/AGENTS.md` y `git/.agents/skills/`.

**Decisiones y Cambios:**

- **Blindaje Incondicional de Permisos (`update.sh`)**: Inclusión de saneamiento de titularidad `rsa:rsa` y permisos `u+rwX,g+rwX` en `$PROJECT_LOCAL_ROOT/log-files` al final de la actualización, evitando bloqueos tras ejecuciones con privilegios elevados.
- **Skill Interactiva `despliegue_release`**: Implementación de la skill en `rsa/RSA-Agent-Toolkit/.agents/skills/despliegue_release.md` con soporte para ingesta pasiva de datos vía `logs.tmp` y entrega de guías de comandos copy-paste.
- **Descarte de Automatizaciones Autónomas en Remoto**: Preservación estricta de la regla de no ejecución autónoma bajo SSHFS (`restriccion_sshfs.md`), descartando scripts de actualización desatendida u orquestación por Tailscale.
- **Gobernanza y Sincronización del Exocortex**: Actualización de la matriz de habilidades en `AGENTS.md` de `RSA-Agent-Toolkit` y replicación al workspace raíz `git/` con `sincronizar_toolkit`.

**Scripts/Comandos relevantes:**

```bash
# 1. Blindaje de permisos en scripts/setup/update.sh
TARGET_USER="${SUDO_USER:-$USER}"
if [ "$TARGET_USER" = "root" ] && id "rsa" &>/dev/null; then
    TARGET_USER="rsa"
fi
sudo chown -R "$TARGET_USER:$TARGET_USER" "$PROJECT_LOCAL_ROOT"
sudo chmod -R u+rwX "$PROJECT_LOCAL_ROOT"
sudo chmod -R u+rwX,g+rwX "$PROJECT_LOCAL_ROOT/log-files"

# 2. Activación de la nueva skill en el agente
# "prepara el release" o "despliega a producción"

# 3. Sincronización del Toolkit hacia el workspace raíz
# "sincroniza el toolkit"
```
---
