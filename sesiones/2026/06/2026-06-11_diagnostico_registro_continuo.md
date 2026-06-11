---
fecha: 2026-06-11
temas: [web, diagnostico, frontend, json]
entorno: [acelerografo]
autor: Milton
---

# Actividad del 2026-06-11

**Hitos de la jornada:**
Se implementó de forma completa y segura el botón "Comprobar" en la interfaz web de configuración del acelerógrafo. Esta nueva funcionalidad ejecuta un binario en C (`comprobar_registro`) para comprobar el estado actual del registro continuo y visualizar los resultados estructurados en campos específicos dentro de la interfaz web, en lugar de mostrar texto plano.

Para lograrlo, se construyó un script wrapper en Python (`comprobar_registro_wrapper.py`) que ejecuta el binario mediante `subprocess` con un timeout seguro de 10 segundos, parsea el texto con expresiones regulares y retorna un objeto JSON estructurado. El servidor Flask (`config_server.py`) expone estos datos en el nuevo endpoint `/api/check`. En el frontend, se diseñó una sección dedicada a "Diagnóstico del Registro" (separada del formulario de configuración) utilizando flexbox/grid responsivo y renderizando de manera segura (prevención de XSS mediante `textContent` y `createElement`) la información sobre nombre del archivo, tamaño, hora del sistema, hora dsPIC, fuente de reloj y aceleraciones por eje.

**Decisiones y Cambios:**
- Crear [comprobar_registro_wrapper.py](file:///home/rsa/git/montajes/acelerografo-DEV00/scripts/operation/acelerografo/comprobar_registro_wrapper.py) para encapsular la ejecución y parsear los datos en el backend (evitando parseo complejo o inseguro en JavaScript).
- Exponer endpoint GET `/api/check` en [config_server.py](file:///home/rsa/git/montajes/acelerografo-DEV00/scripts/operation/web/config_server.py) con manejo de timeout del wrapper de hasta 15 segundos.
- Separar visualmente la sección de diagnóstico de la de configuración en [index.html](file:///home/rsa/git/montajes/acelerografo-DEV00/scripts/operation/web/templates/index.html) y diseñar tarjetas semánticas responsivas en [styles.css](file:///home/rsa/git/montajes/acelerografo-DEV00/scripts/operation/web/static/css/styles.css).
- Implementar la lógica del botón en [app.js](file:///home/rsa/git/montajes/acelerografo-DEV00/scripts/operation/web/static/js/app.js), renombrando la etiqueta técnica de "Hora uC" a "Hora dsPIC" de acuerdo con los requerimientos.
- Actualizar la documentación de arquitectura en [web_context.md](file:///home/rsa/git/montajes/acelerografo-DEV00/docs/context/web_context.md) incluyendo diagramas de flujo en Mermaid y tablas de endpoints actualizados.

**Scripts/Comandos relevantes:**
Para levantar y verificar los servicios en la Raspberry Pi tras la actualización:
```bash
# Actualizar el entorno en la Raspberry Pi
bash menu.sh  # Seleccionar opción 3 para actualizar

# Reiniciar el servidor de configuración
sudo supervisorctl restart config_server
```
