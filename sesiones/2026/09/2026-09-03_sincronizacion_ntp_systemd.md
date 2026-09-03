---
fecha: 2026-09-03
temas: [systemd, ntp, resiliencia, adquisicion]
entorno: [acelerografo]
autor: Milton
---

# Actividad del 2026-09-03

**Hitos de la jornada:**

Durante la jornada de hoy se diagnosticaron y solucionaron dos problemas operacionales observados tras el reinicio en caliente (`sudo reboot`) de la estación acelerográfica `ACEL-DEVP-UNIV-01`. El primer problema consistía en la falta de auto-arranque del servicio de adquisición (`rsa-acelerografo.service`), el cual permanecía en estado `disabled` tras actualizaciones con `update.sh` (Opción 3 de `menu.sh`). El segundo problema era el arranque prematuro en frío de `registro_continuo` en apenas 1-2 segundos tras el boot, antes de que el demonio `ntpd` completara la negociación con los servidores de tiempo, generando advertencias en los logs y arriesgando saltos temporales en los archivos `.dat` y el Ring Buffer.

Para resolver ambas situaciones de forma definitiva, se implementó el script helper `wait_for_ntp.sh` (instalado en `/usr/local/bin/wait_for_ntp`), el cual realiza una espera activa dinámica mediante `ntpstat` (con fallbacks a `timedatectl` y `chronyc`) con un timeout de 120 segundos en la directiva `ExecStartPre` de Systemd. Asimismo, se modificó `update.sh` para ejecutar `systemctl enable` de manera incondicional.

La solución fue validada empíricamente en hardware real mediante un reinicio completo (`sudo reboot`). Se comprobó que `wait_for_ntp` retuvo el arranque durante ~93 segundos hasta que `ntpd` completó su sincronización, permitiendo que `registro_continuo` arrancara inmediatamente con `INFO - Sincronizacion NTP: Si` y eliminando por completo el `WARNING`. La nueva implementación dinámica redujo el tiempo muerto tras reinicio a 108 segundos, ahorrando 85 segundos (44% más rápido) frente al viejo cron de 180 segundos fijos.

**Decisiones y Cambios:**

- **Script Helper `wait_for_ntp.sh`**: Creado en `scripts/task/wait_for_ntp.sh` con timeout dinámico de 120 s. Retorna inmediatamente en cuanto el reloj se sincroniza y sale con `exit 0` no bloqueante si vence el timeout (modo offline).
- **Plantilla de Servicio Systemd**: Actualizado `scripts/task/rsa-acelerografo.service.template` con `ExecStartPre=/usr/local/bin/wait_for_ntp 120` y `After=network.target local-fs.target`.
- **Automatización de Despliegues**: Modificado `scripts/setup/update.sh` (función `update_systemd_service`) añadiendo la auto-habilitación incondicional `sudo systemctl enable rsa-acelerografo.service`.
- **Documentación Técnica**: Generado el contexto técnico `docs/context/wait_for_ntp_context.md`, actualizado `docs/context/registro_continuo_context.md`, redactado el documento de transición `docs/progress/2026-09-03_contexto-agente.md` y sincronizado el índice temático en `RSA-Metodologias`.

**Scripts/Comandos relevantes:**

```bash
# 1. Actualización y despliegue del helper y servicio systemd
cd /home/rsa/git/RSA-Acelerografo
./menu.sh  # Opción 3 (Actualizar el proyecto)

# 2. Verificación de auto-arranque en systemd
systemctl is-enabled rsa-acelerografo.service
# -> enabled

# 3. Prueba manual del helper de sincronización
/usr/local/bin/wait_for_ntp 10
# -> [WAIT_NTP] Reloj sincronizado con éxito vía ntpstat...

# 4. Verificación de logs tras reboot
tail -n 20 /home/rsa/projects/acelerografo/log-files/registro_continuo.log
# -> INFO - Sincronizacion NTP: Si
```
---
