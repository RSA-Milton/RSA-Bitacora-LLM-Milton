---
fecha: 2026-10-16
temas: [acelerografo, drive, memoria, resiliencia]
entorno: [acelerografo]
autor: Milton
---

# Actividad del 2026-10-16

**Hitos de la jornada:**
Se abordó y resolvió de raíz la causa del livelock de memoria, contienda de concurrencia y saturación de swap en las estaciones acelerográficas Raspberry Pi 3B+ (`CHA01`, `CHA02`), derivado de la ejecución horaria por cron de `scripts/task/registrocontinuo.sh` e invocación de `gestor_archivos_acq.py`. Reutilizando los principios probados en `procesar_y_subir_pendientes.py`, se diseñó y aprobó el blueprint `2026-10-14_plan_blindaje_scripts_livelock_memoria.md` para blindar el pipeline regular continuo.

Se implementaron salvaguardas de defensa en profundidad: doble cerrojo de exclusión mutua (guarda `is_running` en bash + cerrojo atómico exclusivo `fcntl.flock` no bloqueante en `/tmp/gestor_archivos_acq.lock` con liberación en `finally:`), resolución DNS real contra `www.googleapis.com:443` con fail-fast temprano eliminando falsos positivos de red, colas prioritarias desacopladas (`event` → `mseed` → `continuous`), procesamiento acotado por lotes (`batch_size=20`), pausas de 1.5s anti-bufferbloat para preservar enlaces SSH/módem celular, recolección forzada de memoria `gc.collect()`, protección de binarios `.dat` activos (por `NombreArchivoRegistroContinuo.tmp` y `mtime < 2h`) y reducción del timeout de socket HTTP de 300s a 60s en `subir_archivo.py`.

Se desarrolló la suite de pruebas unitarias `test_gestor_archivos_acq.py` (6 pruebas con aislamiento tempfile y mocks de DNS/locks), validando la suite combinada completa de 18 pruebas (`logs.tmp`) en 1.816s sin errores. Se documentaron 4 contextos técnicos estandarizados, se formalizó y aceptó el `ADR-024` tanto en el repositorio del proyecto como en `RSA-Metodologias`, y se actualizó el índice federado cerrando el diagnóstico técnico `2026-09-30` como resuelto.

**Decisiones y Cambios:**
- Implementación de cerrojo exclusivo atómico `fcntl.flock` en `/tmp/gestor_archivos_acq.lock` con cierre explícito de descriptor ante contienda.
- Sustitución de comprobación TCP IP `8.8.8.8:53` por resolución de nombres FQDN real vía `socket.getaddrinfo("www.googleapis.com", 443, timeout=5)`.
- Priorización estricta de subida: eventos sísmicos disparados primero (`event`), miniSEED continuo después (`mseed`) y binarios al final (`continuous`).
- Acotamiento por lotes (`batch_size: 20`), pausas de 1.5 s anti-bufferbloat y recolección forzada de basura `gc.collect()`.
- Reducción de timeout de sockets en `subir_archivo.py` a 60 segundos (`Http(timeout=60)`).
- Emisión y aceptación formal de `ADR-024` y cierre del diagnóstico técnico de saturación de memoria en el índice federado.

**Scripts/Comandos relevantes:**
```bash
# Ejecución de pruebas unitarias combinadas de subida a Drive (18 tests totales):
$PROJECT_LOCAL_ROOT/.venv/bin/python3 -m unittest \
  scripts/drive/test_gestor_archivos_acq.py \
  scripts/drive/test_procesar_y_subir_pendientes.py

# Verificación de cerrojo y ejecución limpia en background vía orquestador:
/home/rsa/projects/acelerografo/scripts/task/registrocontinuo.sh start
```
