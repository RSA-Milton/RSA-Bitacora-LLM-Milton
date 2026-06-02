---
fecha: 2026-04-23
temas: [vpn, tailscale, redes, conectividad]
entorno: [acelerografo]
---
# Registro de Sesión: Conectividad Remota con Tailscale en Estaciones Sísmicas

# Actividad del 2026-04-23

# Hitos de la jornada:
Abordamos el reto de establecer un acceso SSH confiable y de baja latencia a las estaciones de campo (Raspberry Pi 3B+) desplegadas detrás de firewalls restrictivos (Fortinet/GADs). Tras analizar el impacto en el rendimiento, se descartó el uso de Cloudflare Tunnels debido a que su agente basado en HTTPS/QUIC introducía sobrecarga inaceptable de CPU/RAM y latencia asimétrica, amenazando la estabilidad del proceso crítico de lectura SPI a 250Hz.


En su lugar, implementamos una arquitectura de red superpuesta (overlay network) utilizando Tailscale (WireGuard). Para sortear las restricciones de salida de la red de la universidad sin incurrir en prácticas de Shadow IT, se definió una estrategia táctica de administración empleando un mobile hotspot temporal. Esto permite perforación NAT (UDP hole punching) para lograr una conexión P2P directa entre el nodo de administración (Windows 11) y la estación en campo, minimizando el uso de recursos y mejorando el tiempo de respuesta de la terminal.


A nivel de configuración de sistema y red, se aseguró la persistencia del acceso deshabilitando la caducidad de claves criptográficas y blindando la configuración inalámbrica mediante el cifrado de la clave PSK. Finalmente, se estableció el comportamiento esperado del enrutamiento roaming-aware de Tailscale al alternar entre el hotspot y la conexión cableada institucional, priorizando la interfaz Ethernet para la telemetría.


# Decisiones y Cambios:

- Adopción de Tailscale sobre Cloudflare: Migración a Tailscale para garantizar conexiones P2P de baja latencia y menor consumo de recursos (15MB-40MB RAM), vital para soportar simultáneamente la adquisición de datos y futuros modelos ML en hardware limitado de 1GB de RAM.

- Estrategia de evasión de Firewall (Administrador): Utilización de redes móviles (Tethering) exclusivamente en el equipo de administración para establecer los túneles VPN, evitando bloqueos de inspección profunda (DPI) en la red universitaria.

- Desactivación de Key Expiry: Se estableció como política obligatoria deshabilitar la caducidad de claves a los 180 días desde el panel web de Tailscale para todas las estaciones headless de la red sísmica (RSA).

- Aislamiento de servicios de red en el cliente: Ejecución de tailscaled en las estaciones de campo con la instrucción estricta de no modificar el DNS ni aceptar rutas externas (--accept-dns=false --accept-routes=false) para evitar disrupciones en la red del GAD anfitrión.

- Seguridad en credenciales Wi-Fi: Uso exclusivo de wpa_passphrase para generar el bloque de configuración del hotspot, evitando almacenar contraseñas en texto plano en /etc/wpa_supplicant/wpa_supplicant.conf.

- Priorización de interfaces (Métrica): Establecimiento de políticas de red donde la interfaz cableada (eth0) siempre tendrá prioridad de salida sobre la inalámbrica (wlan0) para garantizar la continuidad en la transmisión de datos.


# Scripts/Comandos relevantes:

```bash
# Instalación optimizada de Tailscale en Debian Bullseye (Raspberry Pi)
# Agregar llaves y repositorios oficiales
curl -fsSL https://pkgs.tailscale.com/stable/debian/bullseye.noarmor.gpg | sudo tee /usr/share/keyrings/tailscale-archive-keyring.gpg >/dev/null
curl -fsSL https://pkgs.tailscale.com/stable/debian/bullseye.tailscale-keyring.list | sudo tee /etc/apt/sources.list.d/tailscale.list

# Actualización e instalación
sudo apt-get update
sudo apt-get install -y tailscale

# Inicialización segura (evita conflictos de DNS y ruteo en red local)
sudo tailscale up --accept-dns=false --accept-routes=false

# Verificación de IP local asignada en la Tailnet
tailscale ip -4
Bash
# Generación de configuración cifrada para red Wi-Fi (Hotspot Móvil)
# Ejecutar en consola, NO guardar el comando en scripts expuestos
wpa_passphrase "Nombre_Red_Hotspot" "Contraseña_Plana"

# Resultado a inyectar en /etc/wpa_supplicant/wpa_supplicant.conf
network={
    ssid="Nombre_Red_Hotspot"
    #psk="Contraseña_Plana"
    psk=a1b2c3d4e5f6g7h8i9j0... # (Cadena hexadecimal a utilizar)
}
```

```dos
:: Comandos de verificación desde el nodo de administración (Windows 11)
:: Verificar estado de nodos (Online/Offline) y topología
tailscale status

:: Diagnóstico de calidad de enlace P2P vs DERP (Relay)
tailscale ping [IP_DE_LA_RASPBERRY]

:: Conexión SSH a través del túnel VPN
ssh rsa@[IP_DE_LA_RASPBERRY]
```