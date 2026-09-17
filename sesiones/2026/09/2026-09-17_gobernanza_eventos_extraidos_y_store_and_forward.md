---
fecha: 2026-09-17
temas: [eventos, drive, almacenamiento, resiliencia]
entorno: [acelerografo]
autor: Milton
---

# Actividad del 2026-09-17 (Gobernanza del Ciclo de Vida de Eventos Extraídos y Store & Forward)

**Hitos de la jornada:**

Durante esta sesión se diseñó, implementó y validó en producción (`DEV00`) la gobernanza completa del ciclo de vida de los eventos sísmicos extraídos (`/home/rsa/data/eventos-extraidos/`) dentro de `gestor_archivos_acq.py`. Anteriormente, los eventos extraídos dependían de una subida puntual inmediata vía `subir_archivo.py --event`, careciendo de reintentos asíncronos ante fallos de conectividad (ausencia de Store & Forward) y acumulando miles de archivos indefinidamente en disco (2.505 eventos en DEV00).

Para solucionar ambas limitaciones de forma estructural, se ejecutó un blueprint integral:
1. **Arquitectura Store & Forward (Rescate Desatendido)**: En modo `online`, el gestor audita periódicamente `eventos_extraidos/`. Si encuentra archivos `.mseed` no indexados en `archivos_exitosos.event`, los sube automáticamente a Google Drive con reintentos exponenciales.
2. **Retención Temporal Configurable**: Se incorporó el parámetro `politicas[modo].retener_dias.event` en `configuracion_dispositivo.json` (configurado en 7 días para DEV00). Los eventos cuya antigüedad supere este límite y que cuenten con confirmación de subida son purgados de forma desatendida.
3. **Jerarquía Estricta de Desalojo ante Espacio Crítico (< 5%)**: Se estableció que ante saturación de almacenamiento, los eventos extraídos se protegen como datos sismológicos de alta relevancia, desalojándose únicamente como último recurso tras purgar primero los archivos binarios continuos `.dat` (Prioridad 1) y los MiniSEED continuos `.mseed` (Prioridad 2).
4. **Sinergia con el Registro Espejo (ADR-021)**: Todo evento purgado por retención es desindexado automáticamente del archivo JSON en disco, manteniendo una sincronización exacta 1:1.
5. **Suite de Pruebas Unitarias**: Se desarrolló `scripts/operation/drive/test_gestor_eventos.py` con 5 pruebas unitarias independientes, alcanzando 100% de aprobación en la estación.

La validación en vivo en `DEV00` demostró inmediatamente el valor de la arquitectura: el gestor detectó y subió a Google Drive **9 eventos pendientes del 7 de septiembre** que nunca habían sido enviados a la nube (`upload_resumen=9/9`), eliminó **1.546 eventos caducados** (> 7 días) liberando espacio en la tarjeta microSD, y podó el registro JSON de 2.908 a **1.371 entradas activas**, coincidiendo exactamente con los archivos físicos en disco.

La jornada culminó con la formalización y registro del **ADR-022** (`022_gobernanza_ciclo_vida_eventos_extraidos_y_store_and_forward.md`) en el repositorio local y en el exocortex `RSA-Metodologias`, junto con la actualización de la transición técnica en `docs/progress/`.

**Decisiones y Cambios:**

- **Gobernanza de Eventos y Store & Forward (ADR-022)**: Centralización del ciclo de vida de `eventos-extraidos/` en `gestor_archivos_acq.py` con subida asíncrona automática para eventos no transmitidos en caliente.
- **Retención Temporal de Eventos**: Soporte para la clave `event` en `politicas[modo].retener_dias`, configurada a 7 días en `configuracion_dispositivo.json`.
- **Prioridad de Desalojo Sismológico**: Preservación prioritaria de eventos extraídos ante niveles críticos de disco (< 5%), sacrificando antes el registro continuo ya procesado.
- **Protección Preventiva de Subidas Fallidas**: Bloqueo estricto del borrado local si un evento se encuentra marcado en `archivos_fallidos.event`.
- **Trazabilidad y Simulación**: Métricas granulares de eventos evaluados y expirados en el log `SUMMARY` y soporte total para `--dry-run`.

**Scripts/Comandos relevantes:**

```bash
# 1. Ejecutar la suite de pruebas unitarias de gobernanza de eventos
python3 scripts/operation/drive/test_gestor_eventos.py

# 2. Simular ciclo de gestión con evaluación de retención de eventos (--dry-run)
python3 scripts/operation/drive/gestor_archivos_acq.py --dry-run

# 3. Ejecutar ciclo completo de almacenamiento y Store & Forward en producción
python3 scripts/operation/drive/gestor_archivos_acq.py

# 4. Inspeccionar el resumen diagnóstico de Drive y balance del registro espejo
diagnostico drive
```
---
