---
fecha: 2026-08-06
temas: [event-analyzer, streamlit, plotly, dsp, downsampling]
entorno: [tig]
autor: Milton
---

# Actividad del 2026-08-06

**Hitos de la jornada:**
En esta sesión se optimizó la arquitectura de la interfaz web y el motor de renderizado gráfico de la aplicación **Event Analyzer** (`services/event-analyzer`), desplegada en Streamlit. Se implementó una selección interactiva por fecha utilizando un widget de calendario (`st.date_input`) que filtra dinámicamente el catálogo de eventos regionales por día, junto con un estado inicial en blanco (`st.stop()`) previniendo auto-renderizados pesados y cargas de memoria involuntarias.

Adicionalmente, se diagnosticó y resolvió el error crítico `MessageSizeError: Data of size 218.2 MB exceeds the message size limit` (saturación de WebSockets en Streamlit y congelamiento de UI en Plotly) mediante un algoritmo de **diezmado dinámico (Downsampling)** en la capa de visualización (`visualizer.py`). Al acotar las trazas a un máximo de 3,000 puntos por canal (`MAX_POINTS_PER_TRACE`) usando slicing eficiente en C con NumPy (`data[::step]`), el tamaño del payload JSON enviado al navegador se redujo de >200MB a menos de 2MB, acelerando los gráficos de varios minutos a menos de 2 segundos. Los cálculos DSP (Detrending, filtro pasabanda Butterworth y estimación de PGA Z) continúan ejecutándose sobre la señal cruda completa a máxima resolución.

**Decisiones y Cambios:**
- Encapsulado de la selección de eventos en un panel interactivo con widget de fecha (`st.date_input`) y botón de aplicación (`✅ Aplicar Selección de Eventos`).
- Implementación del estado inicial en blanco usando `applied_event_label = None` y `st.stop()`.
- Incorporación de diezmado dinámico (`MAX_POINTS_PER_TRACE = 3000`) en `src/modules/visualizer.py` y optimización de conversión de timestamps UTC.
- Adición del montaje bind `../event-analyzer:/app` en `services/docker-unified/docker-compose.yml` para desarrollo en caliente.
- Documentación de la decisión de arquitectura en `ADR-013` e inserción en el índice temático federado.

**Scripts/Comandos relevantes:**
```bash
# Reiniciar contenedor event-analyzer tras los cambios de volumen y código fuente
docker compose -f services/docker-unified/docker-compose.yml restart event-analyzer

# Reconstruir el servicio si fuera necesario
docker compose -f services/docker-unified/docker-compose.yml up -d --build event-analyzer
```
---
