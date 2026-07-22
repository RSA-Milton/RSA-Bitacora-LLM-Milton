---
fecha: 2026-07-22
temas: [node-red, mqtt, zona-horaria, ui-dashboard]
entorno: [node-red, tig]
autor: Milton
---

# Actividad del 2026-07-22

**Hitos de la jornada:**
Se implementó la funcionalidad de selección de zona horaria en el panel de control remoto de Node-RED (`services/node-red/flows.json`). Se reemplazó el control original de `Upload` por un desplegable de selección de zona horaria (Tiempo Local UTC-5 de Ecuador vs Tiempo UTC). Asimismo, se actualizaron los parámetros por defecto de los comandos emitidos por MQTT para asegurar la inclusión fija de `upload: true` y `delete_after_upload: true`.

Posteriormente, se ajustó el nodo de entrada de hora (`Start Time`) a formato de texto para permitir el ingreso explícito de hora con segundos (`HH:mm:ss`). Se mejoró la lógica de conversión matemática en el nodo `Build Command` para parsear horas, minutos y segundos, agregar validaciones de rango seguro (0-23h, 0-59m, 0-59s) y realizar la conversión precisa a cadenas de tiempo ISO UTC (`YYYY-MM-DDZHH:MM:SS`). Finalmente, se actualizó la documentación técnica de contexto del servicio.

**Decisiones y Cambios:**
- **Reemplazo de Control UI**: Sustitución del switch `Upload` por un `ui_dropdown` denominado `Zona Horaria` (`node-timezone-select`) con opciones `LOCAL` (UTC-5) y `UTC`.
- **Valores por Defecto**: Inclusión permanente de `upload: true` y `delete_after_upload: true` en la carga útil del comando `extract_event`.
- **Entrada de Hora con Segundos**: Configuración de `node-start-time` a modo `text` con etiqueta `hh:mm:ss`, permitiendo la entrada en formatos `HH:mm:ss` y `HH:mm`.
- **Validación de Rangos Horarios**: Incorporación de verificaciones numéricas en `Build Command` para evitar comandos corruptos ante entradas fuera de rango.
- **Documentación de Contexto**: Actualización de `docs/context/docker-compose-nodered_context.md`.

**Scripts/Comandos relevantes:**
```javascript
// Conversión matemática a UTC con segundos y validación de rangos
const parts = startTime.trim().split(':');
hours = parseInt(parts[0], 10) || 0;
minutes = parseInt(parts[1], 10) || 0;
seconds = parseInt(parts[2], 10) || 0;

if (hours < 0 || hours > 23 || minutes < 0 || minutes > 59 || seconds < 0 || seconds > 59) {
    node.warn(`⚠️ Hora inválida ingresada: ${hours}:${minutes}:${seconds}`);
    return null;
}

if (timeMode === 'LOCAL') {
    const localIsoStr = `${year}-${month}-${day}T${hh}:${mm}:${ss}-05:00`;
    const utcDt = new Date(localIsoStr);
    isoDate = `${utcDt.getUTCFullYear()}-${pad(utcDt.getUTCMonth() + 1)}-${pad(utcDt.getUTCDate())}Z${pad(utcDt.getUTCHours())}:${pad(utcDt.getUTCMinutes())}:${pad(utcDt.getUTCSeconds())}`;
}
```

```bash
# Copia del flujo actualizado al volumen persistente y reinicio del servicio
cp services/node-red/flows.json /home/rsa/data/nodered/flows.json
docker restart rsa-nodered
```
---
