---
fecha: 2026-09-21
temas: [deploy, git, diagnostico]
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
