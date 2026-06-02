---
fecha: 2026-05-12
temas: [node-red, docker, mqtt, ui]
entorno: [tig]
---
# Actividad del 2026-05-12

**Hitos de la jornada:**
Inicio de la implementación del panel de control de Node-RED para la red sísmica RSA. Se configuró el contenedor de Docker para Node-RED de forma independiente al stack TIG y se inició la creación del flujo para la extracción de eventos remotos.

**Decisiones y Cambios:**
- Se creó el archivo `docker-compose.yml` para Node-RED en la red compartida `rsa_network`.
- Se externalizaron las configuraciones en `settings.js` para tener logs persistentes en `/data/nodered.log`.
- Se diseñó el flujo MQTT básico para el comando de `extract_event`.

**Scripts/Comandos relevantes:**
```yaml
# docker-compose.yml inicial
services:
  nodered:
    image: nodered/node-red:latest
    container_name: rsa-nodered
    volumes:
      - /home/rsa/data/nodered:/data
```
---
# Actividad del 2026-05-13

**Hitos de la jornada:**
Resolución de los problemas de renderizado visual en el UI y superposición de widgets en el Dashboard de Node-RED. Refactorización inicial de la estructura de dependencias de NPM en Docker.

**Decisiones y Cambios:**
- Se corrigió el layout de la cuadrícula gráfica asignando un ancho de 8 columnas explícito.
- Se separó la fecha y la hora en nodos distintos (`ui_date_picker` y `ui_text_input` modo *time*).
- Se sustituyó `ui_text` por `ui_template` para mostrar el historial de logs (soluciona colapso flexbox).
- Se intentó mapear `package.json` en Docker, lo que generó problemas de permisos (`EBUSY` y bloqueo de inodos durante `npm install`).

**Scripts/Comandos relevantes:**
```javascript
// Lógica de validación matemática de hora y fecha para build command
const dt = new Date(startDate);
let hours = Math.floor(startTime / 3600000);
let minutes = Math.floor((startTime % 3600000) / 60000);
```
---
# Actividad del 2026-05-15

**Hitos de la jornada:**
Consolidación definitiva de la arquitectura de Node-RED. Estabilización de los montajes de Docker y delegación total de persistencia y credenciales al contenedor para evitar bloqueos.

**Decisiones y Cambios:**
- Se eliminaron todos los montajes individuales (`flows.json`, `package.json`, `settings.js`) del archivo `docker-compose.yml`. Todo queda delegado en el único volumen persistente `- /home/rsa/data/nodered:/data`.
- Se eliminaron del repositorio los archivos `.env` y `flows_cred.json`, ya que las credenciales ahora se configuran directamente en la web de Node-RED (Pestaña Security) y se encriptan nativamente en el volumen.
- Se definió que la dependencia `node-red-dashboard` se gestiona completamente vía la interfaz Manage Palette de Node-RED y se persiste internamente, sin versionar en Git los `node_modules`.
- Se actualizaron las instrucciones de despliegue en `README.md` y la estructura del directorio en `AGENTS.md`.

**Scripts/Comandos relevantes:**
```bash
# Instrucciones de inicialización definidas para despliegues futuros
sudo chown -R 1000:1000 /home/rsa/data/nodered
cp settings.js package.json /home/rsa/data/nodered/
docker compose up -d
```
---
