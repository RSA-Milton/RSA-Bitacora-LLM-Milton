---
fecha: 2026-11-16
temas: [red-adhoc, nl80211, coordinometros, multi-rol]
entorno: [concentrador]
autor: Milton
---

# Actividad del 2026-11-16

**Hitos de la jornada:**
Se inició la implementación técnica del Blueprint 1 (Red Ad-Hoc 802.11 e infraestructura de coordinómetros). Se revisó y actualizó la plantilla de configuración unificada `configuration/concentrador.example.json` incorporando soporte multi-rol mediante listas (`"roles": ["periferico", "gateway_adhoc"]`), lo que permite que el concentrador `CON01` asuma simultáneamente el rol de periférico Modbus RTU y pasarela Ad-Hoc. Se generaron las plantillas persistentes de interfaces de red con sufijos geográficos (`wlan0.izq` y `wlan0.der`) y los scripts de rutas estáticas correspondientes (`rutas_estaticas_izq.sh` y `rutas_estaticas_der.sh`), eliminando la dependencia de scripts en `crontab @reboot sleep 60`.

Asimismo, se estructuró la plantilla `configuration/ssh_config.example` mapeando los 12 coordinómetros ópticos de ambos márgenes (`IZQ-N1-X` .. `IZQ-N3-Y` y `DER-N1-X` .. `DER-N3-Y`) con políticas de resiliencia y timeouts (`ConnectTimeout 5`, `ServerAliveInterval 15`). Se iniciaron las pruebas de despliegue en campo sobre el acelerógrafo sísmico `CHA2` (Raspberry Pi 3B+ en el margen derecho). Durante el levantamiento de `wlan0` en CHA2 se identificaron bloqueos por `rfkill`, colisiones con `wpa_supplicant`/`dhcpcd` y fallos con `wireless-tools` (`iwconfig`), acordándose la transición hacia comandos nativos `iw` (nl80211).

**Decisiones y Cambios:**
- Adopción de convención multi-rol: `"roles": ["periferico", "gateway_adhoc"]` en `concentrador.json`.
- Adopción de sufijos geográficos `izq` y `der` para plantillas de red y scripts de rutas en lugar de nombres de host.
- Estandarización de `ssh_config` para acceso simplificado a los 12 coordinómetros.
- Determinación de inhabilitar `wpa_supplicant` en nodos gateway para evitar que retenga la interfaz en modo cliente (`managed`).

**Scripts/Comandos relevantes:**
```bash
# Checkout de rama y copia de plantillas en CHA02
git checkout feature/bp1-red-adhoc-coordinometros
cp configuration/concentrador.example.json configuration/concentrador.json

# Desbloqueo de radio Wi-Fi
sudo rfkill unblock wifi
```

---

# Actividad del 2026-11-17

**Hitos de la jornada:**
Se completó con éxito el despliegue y validación en campo de la red inalámbrica Ad-Hoc en el acelerógrafo `CHA2`. Se identificó la causa raíz de la persistencia de `wpa_supplicant`: el hook `/lib/dhcpcd/dhcpcd-hooks/10-wpa_supplicant` de Raspberry Pi OS, el cual forzaba a la interfaz a modo `managed`. Al inhabilitar este hook (`sudo mv 10-wpa_supplicant 10-wpa_supplicant.bak`), la interfaz `wlan0` conmutó limpiamente a modo `IBSS` mediante comandos nativos `iw`, asociándose con éxito a la celda oficial de la presa `46:34:D6:59:B2:CB` en Canal 5 (2432 MHz) con IP `192.168.10.5`.

Se ejecutaron pruebas de conectividad multihop vertical desde `CHA2` hacia los coordinómetros del margen derecho:
- **`DER-N3-Y` (`192.168.10.30`):** Enlace directo de radio 100% operativo (RTT 9.0 ms, 0% packet loss, Uptime 31 días).
- **`DER-N3-X` (`192.168.10.31`):** Enlace directo de radio 100% operativo (RTT 8.0 ms, 0% packet loss).
- **`DER-N2-Y` (`192.168.10.20`):** Enlace multihop a través de N3 (.30) 100% operativo (RTT 13.6 ms, TTL=63, Uptime 31 días).
- **`DER-N1`:** Confirmados físicamente desconectados en galería.

Se configuró la autenticación SSH por claves públicas sin contraseña desde `CHA2` hacia los coordinómetros activos. En `DER-N3-X`, se diagnosticó disco 100% lleno (0 bytes libres) y sectores defectuosos en la tarjeta MicroSD (`print_req_error: I/O error dev mmcblk0 sector 12188863`). Se recuperó ~1 GB de almacenamiento purgando logs y caché, reduciendo el load average de 4.03 a 0.32, y se documentó la necesidad de reemplazo físico preventivo de su tarjeta MicroSD. Finalmente, se confirmó en `DER-N3-Y` la ejecución periódica de capturas cada 2 horas en crontab (`0 */2 * * * /home/pi/ejecutables/app_python.sh`) y la ruta activa de resultados `/home/pi/resultados/` (467 MB acumulados), dejando todo listo para la implementación del orquestador en Python (`src/cloud/orquestador_descargas.py`).

**Decisiones y Cambios:**
- Formalización del **ADR-002** (`002_estandarizacion_adhoc_nl80211_multirol.md`): estandarización de interfaz Ad-Hoc con `iw` (nl80211), red abierta sin claves WEP inválidas y soporte multi-rol.
- Generación del documento de contexto técnico [`docs/context/red_adhoc_context.md`](file:///home/rsa/git/montajes/server-pi/RSA-Concentrador-Presas/docs/context/red_adhoc_context.md).
- Adopción de par de claves RSA 4096 sin passphrase para automatización SSH desatendida del orquestador.
- Registro de anomalía de hardware: degradación física de MicroSD en `DER-N3-X` programada para sustitución en próxima visita de campo.

**Scripts/Comandos relevantes:**
```bash
# Configuración nativa Ad-Hoc en CHA02 (acelerógrafo)
sudo mv /lib/dhcpcd/dhcpcd-hooks/10-wpa_supplicant /lib/dhcpcd/dhcpcd-hooks/10-wpa_supplicant.bak
sudo killall -9 wpa_supplicant

sudo ip link set wlan0 down
sudo iw dev wlan0 set type ibss
sudo ip link set wlan0 up
sudo iw dev wlan0 ibss join REDRSAIOT 2432
sudo ip addr add 192.168.10.5/24 dev wlan0
sudo /home/rsa/git/RSA-Concentrador-Presas/src/network/rutas_estaticas_der.sh

# Estado resultante en CHA02
iwconfig wlan0
# wlan0: ESSID:"REDRSAIOT" Mode:Ad-Hoc Frequency:2.432 GHz Cell: 46:34:D6:59:B2:CB

# Conectividad multihop confirmada
ping -c 3 192.168.10.30   # Nivel 3 Y (RTT 9.0 ms)
ping -c 3 192.168.10.20   # Nivel 2 Y via N3 (RTT 13.6 ms)
```
