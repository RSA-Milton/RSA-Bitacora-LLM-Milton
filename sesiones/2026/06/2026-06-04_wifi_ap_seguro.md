---
fecha: 2026-06-04
temas: [wifi, hostapd, dnsmasq, seguridad]
entorno: [acelerografo]
autor: Milton
---
# Actividad del 2026-06-04 — Punto de Acceso WiFi AP Seguro y Oculto

**Hitos de la jornada:**
Se implementó y probó con éxito la funcionalidad de Punto de Acceso WiFi (AP) seguro, privado y oculto en la Raspberry Pi 3B+ del acelerógrafo. Esto permite que el personal de campo se conecte directamente con su teléfono móvil al WiFi del dispositivo para configurarlo a través del panel web, sin depender de redes de laboratorio ni conectividad externa.

El AP se diseñó en base a los servicios `hostapd` y `dnsmasq`. Dado que los acelerógrafos están conectados permanentemente a internet de forma cableada vía Ethernet (`eth0`), la interfaz WiFi (`wlan0`) queda completamente liberada para el AP. Se creó un script de administración robusto `/usr/local/bin/wifiap` que permite instalar dependencias, activar/desactivar el AP de manera controlada en caliente y consultar el estado general.

**Decisiones y Cambios:**
- **Seguridad en Credenciales:** El archivo `/etc/hostapd/hostapd.conf` almacena la clave de red en texto claro. El script de control corrige esto aplicando permisos `600` y cambiando el dueño a `root:root` inmediatamente después de copiarlo, protegiendo las credenciales contra accesos no autorizados.
- **SSID Oculto:** Uso del parámetro `ignore_broadcast_ssid=1` en hostapd para mitigar ataques oportunistas al no anunciar públicamente el nombre de la red del acelerógrafo.
- **Firewall de Aislamiento de Puerto:** Al habilitar el AP (`wifiap enable`), el script añade dinámicamente una regla en `iptables` que bloquea el tráfico entrante al puerto `5000` (panel Flask) por la interfaz cableada Ethernet `eth0`, permitiéndolo únicamente en `lo` y `wlan0`. Esto evita que el panel sea visible desde la red externa o de internet.
- **Principio de Menor Privilegio (Sudoers):** Se restringe la escalada de privilegios a través de sudoers al usuario `rsa` permitiéndole ejecutar exclusivamente `/usr/local/bin/wifiap` sin contraseña, eliminando la necesidad de dar acceso root generalizado.
- **Integración con la Hidratación:** La configuración del AP (SSID dinámico resuelto con el `estacion_id`, contraseña, canal, etc.) se almacena en la configuración maestra y es hidratada automáticamente a partir de una plantilla, manteniendo la coherencia de despliegues.

**Scripts/Comandos relevantes:**
Fragmento clave del script de control `/usr/local/bin/wifiap` para alternar la IP fija y el estado de la red:
```bash
# Activar el AP
if ! grep -q "# --- RSA WIFI AP START ---" /etc/dhcpcd.conf; then
    cat >> /etc/dhcpcd.conf <<EOF
# --- RSA WIFI AP START ---
interface wlan0
    static ip_address=192.168.4.1/24
    nohook wpa_supplicant
# --- RSA WIFI AP END ---
EOF
fi
rfkill unblock wlan
ip link set wlan0 down
systemctl restart dhcpcd
sleep 2
ip link set wlan0 up
systemctl restart dnsmasq hostapd
iptables -A INPUT -p tcp -i eth0 --dport 5000 -j DROP
```
---

## Sesión de Depuración y Ajustes (Tarde)

**Hitos de la jornada:**
Se corrigió un fallo crítico que impedía iniciar el servicio `hostapd` debido a que systemd lo tenía marcado como enmascarado (`masked`). Se modificó el script de control `wifiap.sh` para que ejecute `systemctl unmask hostapd` de forma transparente durante el proceso de instalación y activación, logrando levantar exitosamente el AP y obtener asignación IP en `wlan0`.

Adicionalmente, se modificó la configuración maestra para cambiar el SSID del WiFi AP de un formato dinámico (`ACEL-{{ESTACION_ID}}-CONFIG`) a uno fijo (`ACEL-CONFIG`), previniendo desconexiones accidentales del personal técnico de campo si deciden modificar el identificador de la estación desde el panel web de configuración.

**Decisiones y Cambios:**
- **Resolución de Servicio Enmascarado:** Integración de la llamada a `unmask` en el flujo de inicialización y habilitación del script de control.
- **SSID Estático:** Eliminación del marcador de estación dinámico en el campo `ssid` del bloque `wifi_ap` en [configuracion_maestra.json](file:///home/rsa/git/montajes/acelerografo-DEV00/configuration/configuracion_maestra.json).

**Scripts/Comandos relevantes:**
Comandos ejecutados en el equipo remoto para aplicar y validar los cambios:
```bash
sudo supervisorctl restart config_server
sudo wifiap disable
bash menu.sh  # Opción 3 (Actualizar) para re-hidratar hostapd.conf
sudo wifiap enable
sudo wifiap status
```
---
