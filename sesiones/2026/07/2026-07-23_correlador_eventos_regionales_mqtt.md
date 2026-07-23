---
fecha: 2026-07-23
temas: [mqtt, correlador, eventos-regionales, docker, python, broadcast]
entorno: [server-ubuntu, rsa-intern-tig-mqtt, acelerografo-dev00]
autor: Milton
---

# Actividad del 2026-07-23

**Hitos de la jornada:**
Se implementó el servicio centralizado de correlación de eventos sísmicos regionales MQTT en el servidor Ubuntu (`RSA-Intern-TIG-MQTT`). La arquitectura transicionó desde un modelo de extracción autónoma hiper-sensible en cada estación acelerográfica (`"auto_extract": true`) a un modelo coordinado en red donde las estaciones operan en modo pasivo (`"auto_extract": false`, `"auto_upload": false`), y la extracción masiva/subida a Google Drive solo se dispara cuando el servidor confirma la coincidencia de 2 o más estaciones distintas dentro de una ventana temporal de 10 segundos.

El servicio fue empaquetado dentro de un contenedor Docker (`rsa-correlator`) e integrado a la pila unificada de Docker Compose (`services/docker-unified/docker-compose.yml`). Se probó con éxito en producción recibiendo alertas reales de estaciones en la red, confirmando la coincidencia regional y disparando el comando broadcast `extract_event` con `"delete_after_upload": true`.

**Decisiones y Cambios:**
- **Servicio Correlador Python**: Implementación de `regional_event_correlator.py` en `scripts/correlator/` que suscribe a `rsa/seismic/smart/+/events/detected`, filtra duplicados de la misma estación y emite comandos en `rsa/seismic/smart/broadcast/cmd/extract_event`.
- **Configuración JSON**: Creación de `scripts/correlator/config.json` especificando `ventana_coincidencia_s: 10.0`, `min_estaciones: 2`, `cooldown_evento_s: 60.0` y `delete_after_upload: true`.
- **Contenedorización**: Creación de `Dockerfile` (Python 3.11-slim unbuffered) y adición del servicio `correlator` a `services/docker-unified/docker-compose.yml`.
- **Documentación Técnica**: Actualización del `README.md` principal y creación de `docs/context/regional_event_correlator_context.md` y `docs/progress/2026-07-23_contexto-agente.md`.

**Scripts/Comandos relevantes:**
```python
# Lógica principal de correlación y desduplicación por estación
def _evaluar_evento_regional(self, dt_referencia: datetime) -> None:
    detecciones_coincidentes = []
    estaciones_unicas = set()

    for d in self.buffer_detecciones:
        delta = abs((d["dt"] - dt_referencia).total_seconds())
        if delta <= self.ventana_coincidencia_s:
            detecciones_coincidentes.append(d)
            estaciones_unicas.add(d["station_id"])

    if len(estaciones_unicas) >= self.min_estaciones:
        self._disparar_extraccion_broadcast(detecciones_coincidentes, list(estaciones_unicas))
        self.buffer_detecciones.clear()
```

```bash
# Compilar y desplegar el servicio correlador en el servidor
cd services/docker-unified
docker compose up -d --build correlator
docker compose logs -f correlator
```
---
