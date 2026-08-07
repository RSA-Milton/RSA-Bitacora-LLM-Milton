---
fecha: 2026-08-06
temas: [event-analyzer, streamlit, plotly, dsp, downsampling, plotly-resampler]
entorno: [tig]
autor: Milton
---

# Actividad del 2026-08-06

**Hitos de la jornada:**
En esta sesión se optimizó la arquitectura de la interfaz web y el motor de renderizado gráfico de la aplicación **Event Analyzer** (`services/event-analyzer`), desplegada en Streamlit. Se implementó una selección interactiva por fecha utilizando un widget de calendario (`st.date_input`) que filtra dinámicamente el catálogo de eventos regionales por día, junto con un estado inicial en blanco (`st.stop()`) previniendo auto-renderizados pesados y cargas de memoria involuntarias.

Adicionalmente, se resolvió el fallo crítico `MessageSizeError: Data of size 218.2 MB exceeds the message size limit` (saturación de WebSockets en Streamlit) e integró **`plotly-resampler`** para habilitar el remuestreo dinámico en alta resolución al hacer zoom. La vista inicial del gráfico se sirve diezmada a 3,000 puntos (reduciendo el payload de >200MB a <2MB y cargando en <2s), mientras que al realizar zoom sobre una región de interés (ej. 3 segundos), un micro-servidor Dash asíncrono en el puerto `8050` calcula y sirve el 100% de las muestras crudas nativas (100-200 Hz). Esto permite a los sismólogos realizar la picada exacta de ondas P y S con precisión milimétrica sin congelar el navegador.

**Decisiones y Cambios:**
- Encapsulado de la selección de eventos en un panel interactivo con widget de fecha (`st.date_input`) y botón de aplicación (`✅ Aplicar Selección de Eventos`).
- Implementación del estado inicial en blanco usando `applied_event_label = None` y `st.stop()`.
- Integración de `plotly-resampler>=0.9.1` con `FigureResampler(sub_fig, default_n_shown_samples=3000)` e inyección de trazas `hf_x` / `hf_y` en `src/modules/visualizer.py`.
- Registro de `register_plotly_resampler(mode="Dash", port=8050, host="0.0.0.0")` en `app.py`.
- Exposición del puerto `8050:8050` e inyección de `RESAMPLER_HOST` junto al volumen bind en `services/docker-unified/docker-compose.yml`.
- Verificación técnica mediante Consola JS (`F12`), confirmando 3025 puntos en vista inicial e inyección completa en ventana de zoom.
- Documentación de las decisiones de arquitectura en **`ADR-013`** y **`ADR-014`**, además de la actualización del contexto técnico e índice temático federado.

**Scripts/Comandos relevantes:**
```bash
# Reconstruir el contenedor event-analyzer para instalar la nueva dependencia plotly-resampler y exponer el puerto 8050
cd services/docker-unified
docker compose up -d --build event-analyzer

# Monitorear logs en tiempo real
docker compose logs -f event-analyzer
```
---
