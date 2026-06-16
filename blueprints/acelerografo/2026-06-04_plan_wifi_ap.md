# Plan de Implementación: Punto de Acceso WiFi (AP) Seguro y Oculto

Este plan define las tareas necesarias para dotar al acelerógrafo de la capacidad de operar como un Punto de Acceso WiFi (AP) temporal, seguro y oculto. Esto permitirá realizar la configuración del dispositivo desde un dispositivo móvil a través del panel web en el campo, sin necesidad de conexión externa de red.

---

## 🛠️ Arquitectura y Flujo del AP

El acelerógrafo siempre estará conectado a internet de forma cableada a través de la interfaz Ethernet (`eth0`). Esto significa que la interfaz inalámbrica (`wlan0`) está completamente disponible y dedicada de manera exclusiva a operar como un Punto de Acceso WiFi (AP) local para configuración.

Aun así, por seguridad y eficiencia de consumo energético, el AP no tiene que estar activo de forma permanente si no se está usando. Diseñaremos el script de control `/usr/local/bin/wifiap` para gestionar el estado del AP:

```mermaid
graph TD
    A[Inicio] --> B{Comando wifiap}
    B -- enable -- > C[Agregar IP fija y nohook wpa_supplicant a dhcpcd.conf]
    C --> D[Detener cualquier instancia de wpa_supplicant en wlan0]
    D --> E[Reiniciar dhcpcd]
    E --> F[Iniciar hostapd y dnsmasq]
    F --> G[AP Oculto Activo en 192.168.4.1 (wlan0)]
    
    B -- disable -- > H[Detener hostapd y dnsmasq]
    H --> I[Remover IP fija y nohook de dhcpcd.conf]
    I --> J[Reiniciar dhcpcd]
    J --> K[wlan0 inactiva/sin asignar IP]
```

---

## 📋 Tareas Planificadas

### 1. Modificar la Configuración Maestra
* **Archivo:** `configuration/configuracion_maestra.json`
* **Cambio:** Agregar sección de WiFi AP con valores predeterminados seguros.
```json
    "wifi_ap": {
        "ssid": "ACEL-{{ESTACION_ID}}-CONFIG",
        "wpa_passphrase": "CambiarEstaContrasenaSegura123",
        "ocultar_red": "si",
        "canal": "7"
    }
```

### 2. Crear la Plantilla de hostapd
* **Archivo nuevo:** `configuration/hostapd.conf.template`
* **Contenido:** Plantilla del servicio `hostapd` para `wlan0`.
```ini
interface=wlan0
driver=nl80211
ssid={{WIFI_SSID}}
hw_mode=g
channel={{WIFI_CHANNEL}}
wmm_enabled=1
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid={{WIFI_HIDDEN}}
wpa=2
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
wpa_passphrase={{WIFI_PASSPHRASE}}
```

### 3. Actualizar el Script de Hidratación
* **Archivo:** `scripts/setup/hidratar_configuracion.py`
* **Cambio:** 
  - Leer la sección `wifi_ap` de la configuración maestra.
  - Reemplazar marcadores (`{{WIFI_SSID}}`, `{{WIFI_PASSPHRASE}}`, `{{WIFI_CHANNEL}}`, `{{WIFI_HIDDEN}}` [si `ocultar_red` es "si", valor "1", sino "0"]).
  - Generar el archivo final `configuracion/hostapd.conf` en `$PROJECT_LOCAL_ROOT/configuracion/`.

### 4. Crear el Script de Control `/usr/local/bin/wifiap`
* **Archivo nuevo:** `scripts/task/wifiap.sh`
* **Funciones del script:**
  - `wifiap install`:
    - Verifica si `hostapd` y `dnsmasq` están instalados en la RPi. Si no, alerta al usuario.
    - Crea el archivo `/etc/dnsmasq.d/wifiap.conf` con el rango IP `192.168.4.2-20` en `wlan0`.
    - Deshabilita los servicios `hostapd` y `dnsmasq` para que no arranquen por defecto al encender la RPi (el AP se inicia a demanda o mediante comando).
  - `wifiap enable`:
    - Genera el backup de `/etc/dhcpcd.conf`.
    - Inserta al final de `/etc/dhcpcd.conf` la IP estática y el bloqueo de wpa_supplicant.
    - Copia `hostapd.conf` hidratado a `/etc/hostapd/hostapd.conf` y aplica `chmod 600`.
    - Detiene la conexión actual y levanta los servicios: `hostapd`, `dnsmasq`, reinicia `dhcpcd`.
  - `wifiap disable`:
    - Detiene `hostapd` y `dnsmasq`.
    - Elimina la configuración del AP de `/etc/dhcpcd.conf`.
    - Reinicia `dhcpcd` para reactivar el comportamiento cliente de `wlan0`.
  - `wifiap status`:
    - Comprueba si el servicio `hostapd` y `dnsmasq` están levantados y si `wlan0` tiene la IP `192.168.4.1`.

### 5. Configuración de Sudoers para Ejecución Segura
El panel web Flask (`config_server.py`) corre bajo el usuario `rsa`. Para cambiar el estado de red o arrancar servicios mediante comandos de sistema, se requiere elevación de privilegios de forma segura.
* **Instrucciones para el usuario:** Crear un archivo `/etc/sudoers.d/rsa-wifiap` que permita ejecutar solo el script de control:
  ```text
  rsa ALL=(ALL) NOPASSWD: /usr/local/bin/wifiap
  ```

---

## 🛡️ Medidas de Seguridad Aplicadas

1. **Permisos de Archivos de Red**: El archivo `/etc/hostapd/hostapd.conf` contiene la frase de contraseña WPA en texto claro. El script de control cambiará el propietario a `root:root` y establecerá los permisos en `600` inmediatamente después de copiarlo, asegurando que ningún usuario no root (o atacante que comprometa el servidor web) pueda leer la clave del WiFi.
2. **Principio de Menor Privilegio (Sudoers)**: En lugar de otorgar permisos `sudo` generalizados o permitir ejecutar comandos genéricos de red como `systemctl` o `ifconfig`, la interfaz web solo estará autorizada a ejecutar el script específico `/usr/local/bin/wifiap`.
3. **SSID Oculto**: El uso de `ignore_broadcast_ssid=1` oculta el SSID del Punto de Acceso para mitigar que atacantes casuales intenten conectarse a la RPi mientras está en funcionamiento.
4. **Validación de Inputs en Shell**: El script `/usr/local/bin/wifiap` no recibirá parámetros que se concatenen directamente en comandos shell vulnerables a inyección de código.

---

## 🧪 Plan de Verificación

### Automated Security Check

- **Security Scanner**: Run a scan on all newly created files to identify common vulnerabilities (e.g., shell injection, insecure file permissions). If findings are detected, auto-apply the fix and document the results.
- **Security Audit**: Audit the new code for design-level security issues (input validation, secrets handling, auth checks). Document findings and remediations in the `walkthrough.md` artifact using the `generate_security_audit_report` skill.

### Pruebas de Funcionamiento Local (vía comandos que el usuario ejecutará)
1. **Instalación e Hidratación**:
   - `python3 scripts/setup/hidratar_configuracion.py` -> Verificar la generación del archivo `configuracion/hostapd.conf`.
   - `sudo wifiap install` -> Verificar que los paquetes estén instalados y el servicio `dnsmasq` configurado.
2. **Activación de AP**:
   - `sudo wifiap enable` -> Confirmar que el servicio `hostapd` arranca, `dnsmasq` asigna IPs y `wlan0` tiene la IP `192.168.4.1`.
   - Verificar si la red WiFi es visible en dispositivos (estando el SSID oculto, requerirá añadirla manualmente).
3. **Desactivación de AP**:
   - `sudo wifiap disable` -> Confirmar que los servicios se detienen, el archivo `/etc/dhcpcd.conf` se limpia, y `wlan0` vuelve a asociarse a la red WiFi cliente si existe.
