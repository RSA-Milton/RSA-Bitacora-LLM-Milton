---
fecha: 2026-10-19
temas: [acelerografo, almacenamiento, drive, resiliencia]
entorno: [acelerografo]
autor: Milton
---

# Actividad del 2026-10-19

**Hitos de la jornada:**
Se diagnosticó y resolvió de forma integral la acumulación crítica de datos y el fallo en el control de espacio en disco en las estaciones acelerográficas `DEV00` y `TES00`. En `DEV00`, la partición `/dev/root` alcanzó un 96% de ocupación (solo 572 MB libres / 3.92% de espacio libre), acumulando más de 744 archivos MiniSEED y cientos de binarios `.dat` pese a los umbrales de alerta del 10% (mínimo) y 5% (crítico). El análisis forense documentado en el diagnóstico técnico identificó cuatro defectos interconectados: un `NameError` determinista en línea 830 por la variable no declarada `archivo_mas_reciente_continuous`, desalojo insuficiente unitario (`break` incondicional tras eliminar un solo archivo de 4.2 MB), falso positivo de concurrencia en `adquirir_cerrojo()` que enmascaraba excepciones `PermissionError` como si otra instancia estuviese activa (deadlock operativo ante lockfiles pertenecientes a `root`), y sobredimensionamiento de la retención temporal de binarios brutos (30 días requerían ~10.1 GB en microSDs de 16 GB).

Siguiendo el flujo metodológico (`planning_guide` → `execution_guide` en 6 fases), se diseñó y ejecutó el blueprint `2026-10-19_plan_correccion_acumulacion_y_robustecimiento_gestor_sync.md`. Se unificó la salvaguarda de archivos activos con `archivos_activos_continuous`, se implementó un bucle de desalojo dinámico iterativo con reevaluación continua de espacio libre y envoltura `try/except Exception` para garantizar el saneamiento de `uploaded_files_registry.json`, se aisló el cerrojo de proceso mediante `obtener_ruta_cerrojo()` priorizando `$PROJECT_LOCAL_ROOT/tmp-files/` (`rsa:rsa`) con segregación estricta de `BlockingIOError` (salida limpia 0) y `PermissionError` (salida con código 1), se añadió purga proactiva de cerrojos en `registrocontinuo.sh stop`, se ajustó la retención de binarios `continuous` a 7 días en la plantilla de configuración, y se amplió la suite unitaria a 10 pruebas automatizadas.

La solución se certificó en la estación física `DEV00`: la simulación `--dry-run` validó la eliminación calculada de 106 archivos `.dat`, y la corrida definitiva recuperó el espacio libre de 3.77% a 10.0% (1.5 GB disponibles), saneando 297 registros huérfanos. Se formalizó la decisión técnica en el `ADR-025`, se redactó la transición técnica `2026-10-19_contexto-agente.md`, se actualizaron 3 contextos técnicos y se indexó la resolución del diagnóstico en el índice federado.

**Decisiones y Cambios:**
- Erradicación de `archivo_mas_reciente_continuous` en líneas 830, 864 y 930 de `gestor_archivos_acq.py`, unificando la verificación con `archivos_activos_continuous`.
- Implementación de desalojo dinámico iterativo (`while free_space < umbral_minimo: borrar()`) y envoltura `try...except Exception as e:` con reporte estructurado (`logger.error`).
- Aislamiento del cerrojo en `obtener_ruta_cerrojo()` priorizando `$PROJECT_LOCAL_ROOT/tmp-files/gestor_archivos_acq.lock` (propiedad de `rsa`) con fallback a `/tmp/gestor_archivos_acq_${USER}.lock`.
- Desacoplamiento de excepciones en `adquirir_cerrojo()`: discriminación estricta entre `BlockingIOError` (código de salida 0) y `PermissionError` (código de salida 1 con error en `sys.stderr`).
- Purga proactiva de archivos lock en `scripts/task/registrocontinuo.sh stop`.
- Estandarización de la retención de binarios continuos brutos a 7 días en la política `online` de `configuration/configuracion_dispositivo.json.template`.
- Ampliación de la suite de pruebas unitarias `test_gestor_archivos_acq.py` de 6 a 10 pruebas automatizadas (10/10 OK en 0.040s).
- Emisión y aceptación formal de `ADR-025`, cierre de diagnóstico técnico `2026-10-19` en el índice federado y actualización de contextos técnicos.

**Scripts/Comandos relevantes:**
```bash
# Ejecución de la suite unitaria ampliada del gestor (10 tests aislados):
$PROJECT_LOCAL_ROOT/.venv/bin/python3 scripts/drive/test_gestor_archivos_acq.py

# Simulación completa del desalojo dinámico y nueva retención (--dry-run):
$PROJECT_LOCAL_ROOT/.venv/bin/python3 scripts/drive/gestor_archivos_acq.py --dry-run

# Ejecución real de purga y saneamiento de registros en DEV00:
$PROJECT_LOCAL_ROOT/.venv/bin/python3 scripts/drive/gestor_archivos_acq.py

# Parada y arranque limpio con purga de cerrojos:
/home/rsa/projects/acelerografo/scripts/task/registrocontinuo.sh stop
/home/rsa/projects/acelerografo/scripts/task/registrocontinuo.sh start
```
