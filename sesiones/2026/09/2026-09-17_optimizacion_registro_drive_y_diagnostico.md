---
fecha: 2026-09-17
temas: [drive, diagnostico, automatizacion]
entorno: [acelerografo]
autor: Milton
---

# Actividad del 2026-09-17 (Optimización del Registro de Google Drive y Síntesis de Diagnóstico)

**Hitos de la jornada:**

Durante esta jornada se resolvió de forma integral y estructural la saturación operativa provocada por el crecimiento acumulativo e infinito del archivo de registro de sincronización de Google Drive (`uploaded_files_registry.json`) y la consecuente inundación de terminal al ejecutar el script de diagnóstico (`diagnostico.sh`). Se formalizó arquitectónicamente el "Principio de Registro Espejo", estableciendo que el registro JSON de subidas no debe comportarse como una bitácora histórica infinita, sino como una caché de deduplicación acotada estrictamente a los archivos físicos que residen en el almacenamiento local de la estación.

Para ello, se ejecutó un blueprint en cuatro fases:
1. **Robustecimiento de `drive_status_manager.py`**: Se reforzó la función `limpiar_archivos_inexistentes()` con validación preventiva de directorios (`os.path.isdir`), previniendo que volúmenes desmontados o rutas transitoriamente no disponibles vacíen erróneamente el registro. Se incorporaron métricas detalladas de balance y la función `obtener_resumen_diagnostico()`, validada con una nueva batería de pruebas unitarias (`scripts/operation/drive/test_drive_status_manager.py`, 7/7 tests aprobados).
2. **Automatización del Ciclo de Vida en `gestor_archivos_acq.py`**: Se integró la poda automática de entradas huérfanas al término de las rutinas de retención temporal y espacio en disco, con soporte para simulación `--dry-run` y trazabilidad estructurada en `gestor_acq.log`. Se validó la no regresión con la suite `test_drive_watchdog.py` (8/8 tests aprobados).
3. **Síntesis Ejecutiva en `diagnostico.sh`**: Se erradicó el volcado ciego con `cat` del archivo JSON (que alcanzaba miles de líneas), sustituyéndolo por un reporte sintético de ~25 líneas generado mediante Python embebido consumiendo `obtener_resumen_diagnostico()`. Se añadió la bandera `--raw` (`-r`) para auditorías crudas y se documentó en `ayuda.sh`.
4. **Saneamiento bajo Demanda e Inventario**: Se dotó a `gestor_archivos_acq.py` del flag `--purge-registry` (combinable con `--dry-run`), permitiendo auditorías de inventario manual. Se validó en producción en la estación `DEV00`, confirmando que sus 2.903 archivos registrados (407 mseed y 2.496 eventos) existen físicamente en disco y reportando 0 huérfanos.

La jornada culminó con la extracción formal del ADR-021 (`021_sincronizacion_espejo_registro_drive_y_sintesis_diagnostico.md`) en el repositorio local y en el exocortex federado `RSA-Metodologias`, la actualización de los documentos de contexto técnico (`drive_status_manager_context.md`, `gestor_archivos_acq_context.md`, `diagnostico_context.md`, `ayuda_context.md`) y la generación de la transición técnica en `docs/progress/`.

**Decisiones y Cambios:**

- **Principio de Registro Espejo (ADR-021)**: El registro JSON indexa única y exclusivamente archivos físicamente existentes en disco para evitar re-subidas duplicadas; cualquier archivo purgado por retención es eliminado del registro de forma automática.
- **Protección ante Rutas Inválidas**: Chequeo estricto `os.path.isdir(directorio)` antes de evaluar ausencias de archivos en `drive_status_manager.py`.
- **Poda Periódica Desatendida**: Invocación de `limpiar_archivos_inexistentes()` tras la retención de datos en `gestor_archivos_acq.py`.
- **Mecanismo CLI de Poda Manual (`--purge-registry`)**: Argumento opcional con inventario desglosado en pantalla y soporte de simulación sin escrituras.
- **Síntesis en Diagnóstico (`diagnostico.sh`)**: Reemplazo de salida cruda de miles de líneas por resumen compacto de 25 líneas con opción `--raw`.
- **Actualización de Ayuda de Campo (`scripts/task/ayuda.sh`)**: Documentación interactiva de las banderas `--raw` y `--purge-registry`.

**Scripts/Comandos relevantes:**

```bash
# 1. Ejecutar tests unitarios del gestor de estado de Drive
python3 scripts/operation/drive/test_drive_status_manager.py

# 2. Simular ciclo de gestión y poda automática de Drive
python3 scripts/operation/drive/gestor_archivos_acq.py --dry-run

# 3. Purgar huérfanos del registro de Drive bajo demanda (simulación y real)
python3 scripts/operation/drive/gestor_archivos_acq.py --purge-registry --dry-run
python3 scripts/operation/drive/gestor_archivos_acq.py --purge-registry

# 4. Diagnosticar canal de Drive con salida ejecutiva sintetizada (~25 líneas)
diagnostico drive

# 5. Diagnosticar con volcado crudo del JSON si se requiere auditoría completa
diagnostico drive --raw
```
---
