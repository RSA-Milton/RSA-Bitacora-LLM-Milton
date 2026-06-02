---
fecha: 2026-05-22
temas: ubuntu, grafana, wayland, kiosk
entorno: tig
---

# Registro de Sesión: Quiosco de Grafana Seguro en Ubuntu con SeisComP3

# Actividad del 2026-05-22

**Hitos de la jornada:**
Se diseñó e implementó una solución no invasiva para desplegar un quiosco de Grafana en un monitor físico conectado localmente a un servidor con Ubuntu Desktop 24.04 LTS. La máquina debe ser administrada exclusivamente mediante SSH de manera remota, sin contar con periféricos locales (teclado/ratón) conectados de forma permanente. El objetivo principal era garantizar que la pantalla permaneciera encendida indefinidamente mostrando el panel de control sin dejar la sesión gráfica de GNOME vulnerable a accesos físicos ni requerir interacciones manuales tras un reinicio.

Debido a que el servidor ejecuta **SeisComP3** (un software crítico para monitoreo sísmico con interfaces basadas en Qt), se descartaron soluciones invasivas como la desactivación global de Wayland para forzar X11 o el uso de gestores de ventanas ligeros como Openbox, los cuales ponían en riesgo la estabilidad del software sísmico. En su lugar, se optó por un enfoque a nivel de usuario que respeta el servidor gráfico GNOME sobre Wayland por defecto, utilizando el mecanismo de GNOME Autostart para lanzar la instancia de Google Chrome en modo quiosco y ajustando las políticas de suspensión de pantalla a través de `gsettings`.

Finalmente, se resolvió un problema de bucle de autenticación en la interfaz del quiosco. Este comportamiento se originaba porque la dirección de carga inicial apuntaba a `localhost`, pero las redirecciones internas del panel (como el botón "Home") utilizaban la IP de red estática del servidor (`10.22.147.102`). Al diferir los dominios de origen, el navegador descartaba las cookies de sesión y volvía a solicitar credenciales. La solución consistió en unificar la dirección de carga del script del quiosco utilizando directamente la IP estática del servidor y estructurar los enlaces del panel de Grafana con rutas relativas.

**Decisiones y Cambios:**
- **Implementación a nivel de usuario (No invasiva):** Se mantuvo el servidor gráfico GNOME y Wayland activos por defecto para proteger la compatibilidad y estabilidad del entorno de ejecución de SeisComP3.
- **Exclusión de VNC:** Se eliminó la configuración de servidores de escritorio remoto (como TigerVNC) para reducir la superficie de exposición y simplificar la arquitectura, manteniendo SSH como único canal de administración remota.
- **Inhabilitación de la suspensión mediante GSettings:** Se configuró el comportamiento de la pantalla directamente sobre la sesión del usuario para evitar el bloqueo o apagado de la señal de video.
- **Consistencia de dominios para cookies de sesión:** Se determinó inicializar el quiosco apuntando directamente a la IP estática del servidor (`10.22.147.102`) en lugar de `localhost` para evitar pérdidas de sesión por redirecciones cruzadas.

**Scripts/Comandos relevantes:**

*Script de inicialización del quiosco (`~/local/bin/grafana-kiosk.sh`):*
```bash
#!/bin/bash
# Exportar display gráfico para aplicar cambios a la sesión de GNOME
export DISPLAY=:0

# Desactivar la suspensión de pantalla y el apagado automático por inactividad
gsettings set org.gnome.desktop.session idle-delay 0
gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-ac-timeout 0
gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-ac-type 'nothing'

# Esperar a que el entorno gráfico y la red se inicialicen por completo
sleep 10

# Lanzar Google Chrome en modo quiosco apuntando a la IP estática
google-chrome --kiosk --no-first-run --new-window "http://10.22.147.102:3000/d/your-dashboard-uid?orgId=1&refresh=5s"
```

*Archivo de autoinicio de GNOME (`~/.config/autostart/grafana-kiosk.desktop`):*
```ini
[Desktop Entry]
Type=Application
Exec=/home/usuario/local/bin/grafana-kiosk.sh
Hidden=false
NoDisplay=false
X-GNOME-Autostart-enabled=true
Name=Grafana Kiosk
Comment=Inicia el navegador Chrome en modo quiosco al iniciar la sesión gráfica
```

*Comandos de control y diagnóstico remotos (vía SSH):*
```bash
# Exportar display para interactuar con la sesión gráfica
export DISPLAY=:0

# Verificar la configuración actual del tiempo de inactividad de pantalla
gsettings get org.gnome.desktop.session idle-delay

# Verificar la acción por inactividad de energía en CA
gsettings get org.gnome.settings-daemon.plugins.power sleep-inactive-ac-type

# Forzar el cierre y reinicio del quiosco de Chrome
pkill chrome
```
