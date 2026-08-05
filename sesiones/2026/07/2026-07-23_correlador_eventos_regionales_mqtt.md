---
fecha: 2026-07-23
temas: [mqtt, correlador, eventos-regionales, docker, python, broadcast, rclone, google-drive, systemd, fuse]
entorno: [server-ubuntu, rsa-intern-tig-mqtt, acelerografo-dev00]
autor: Milton
---

# Actividad del 2026-07-23

**Hitos de la jornada:**
1. **Correlador Regional de Eventos MQTT**:
   Se implementó el servicio centralizado de correlación de eventos sísmicos regionales MQTT en el servidor Ubuntu (`RSA-Intern-TIG-MQTT`). La arquitectura transicionó desde un modelo de extracción autónoma hiper-sensible en cada estación acelerográfica (`"auto_extract": true`) a un modelo coordinado en red donde las estaciones operan en modo pasivo (`"auto_extract": false`, `"auto_upload": false`), y la extracción masiva/subida a Google Drive solo se dispara cuando el servidor confirma la coincidencia de 2 o más estaciones distintas dentro de una ventana temporal de 10 segundos.

   El servicio fue empaquetado dentro de un contenedor Docker (`rsa-correlator`) e integrado a la pila unificada de Docker Compose (`services/docker-unified/docker-compose.yml`). Se probó con éxito en producción recibiendo alertas reales de estaciones en la red, confirmando la coincidencia regional y disparando el comando broadcast `extract_event` con `"delete_after_upload": true`.

2. **Configuración de Google Drive y Montaje Persistente (`rclone` + `systemd`)**:
   Se configuró e integró la sincronización/montaje de Google Drive en el servidor Ubuntu 24.04 LTS para acceder localmente a las formas de onda MiniSEED subidas por las estaciones en la carpeta `"DIA/Datos Estaciones"`.
   
   Se solucionó la caducidad del token OAuth2 ejecutando la re-autenticación de `rclone` (`rclone config reconnect gdrive:`). Se verificó el acceso de lectura a los subdirectorios de las estaciones (`CHA01`, `CHA02`, `DEV00`, `DEV01`, `FERR`, `LAB01`, `OBSID`, `PRM01`, `PRM02`, `TENG`, `TST1`).
   
   Para garantizar la disponibilidad continua ante reinicios del servidor y permitir la lectura desde contenedores Docker (como `event-analyzer`), se activó `user_allow_other` en `/etc/fuse.conf` y se creó el servicio de sistema `rclone-gdrive.service` montando la unidad en `/home/rsa/datos_estaciones_drive`.

**Decisiones y Cambios:**
- **Servicio Correlador Python**: Implementación de `regional_event_correlator.py` en `scripts/correlator/` que suscribe a `rsa/seismic/smart/+/events/detected`, filtra duplicados de la misma estación y emite comandos en `rsa/seismic/smart/broadcast/cmd/extract_event`.
- **Configuración JSON Correlador**: Creación de `scripts/correlator/config.json` especificando `ventana_coincidencia_s: 10.0`, `min_estaciones: 2`, `cooldown_evento_s: 60.0` y `delete_after_upload: true`.
- **Re-autenticación y Montaje Rclone**: Reconexión de la cuenta `gdrive:` y montaje selectivo de `"DIA/Datos Estaciones"` en la ruta del servidor `/home/rsa/datos_estaciones_drive`.
- **Persistencia Systemd FUSE**: Creación del archivo de servicio `/etc/systemd/system/rclone-gdrive.service` configurado con `--vfs-cache-mode full`, `--vfs-cache-max-size 10G`, `--allow-other` y `--umask 002`. Habilitado y arrancado con `systemctl enable --now rclone-gdrive.service`.
- **Documentación Técnica**: Actualización del `README.md` principal y creación de `docs/context/regional_event_correlator_context.md` y `docs/progress/2026-07-23_contexto-agente.md`.

**Scripts/Comandos relevantes:**
```python
# Lógica principal de correlación y desduplicación por estación
def _evaluar_evento_regional(self, dt_referencia: datetime) -> None:
    detecciones_coincidentes = []
    estaciones_unicas = set()

    for d in self.buffer_detecciones:
        delta = abs((d["dt"] - dt_referencia).total_seconds())
        if delta <= self.ventana_coincidencia_s:
            detecciones_coincidentes.append(d)
            estaciones_unicas.add(d["station_id"])

    if len(estaciones_unicas) >= self.min_estaciones:
        self._disparar_extraccion_broadcast(detecciones_coincidentes, list(estaciones_unicas))
        self.buffer_detecciones.clear()
```

```bash
# Reconectar token expirado de rclone
rclone config reconnect gdrive:

# Verificar listado de estaciones en Google Drive
rclone lsd gdrive:"DIA/Datos Estaciones"

# Habilitar permisos FUSE para Docker y crear servicio Systemd
echo "user_allow_other" | sudo tee -a /etc/fuse.conf

# Activar e iniciar servicio automontado en Ubuntu
sudo systemctl daemon-reload
sudo systemctl enable --now rclone-gdrive.service
sudo systemctl status rclone-gdrive.service
```

```ini
# /etc/systemd/system/rclone-gdrive.service
[Unit]
Description=Montaje Google Drive Datos Estaciones RSA
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
User=rsa
ExecStart=/usr/bin/rclone mount gdrive:"DIA/Datos Estaciones" /home/rsa/datos_estaciones_drive \
  --config /home/rsa/.config/rclone/rclone.conf \
  --vfs-cache-mode full \
  --vfs-cache-max-size 10G \
  --allow-other \
  --umask 002
ExecStop=/bin/fusermount -u /home/rsa/datos_estaciones_drive
Restart=on-failure
RestartSec=10s

[Install]
WantedBy=multi-user.target
```
---

