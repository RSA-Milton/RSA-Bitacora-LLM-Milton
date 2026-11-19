---
fecha: 2026-11-19
temas: [concentrador, coordinometro, desacoplamiento-runtime, adhoc, zip, rsync]
entorno: [concentrador, coordinometro]
autor: Milton
---

# Actividad del 2026-11-19

**Hitos de la jornada:**
Se ejecutó de manera completa y exitosa la implementación, depuración y validación en banco de pruebas de laboratorio del Blueprint de Desacoplamiento de Runtime, Red Ad-Hoc 802.11 y Empaquetado ZIP Mensual para auscultación óptica de presas, interconectando el concentrador `SERVER-Pi` (Raspberry Pi con Bullseye) y el coordinómetro remoto `DER-N3-X` (`192.168.10.31`, Raspberry Pi con Stretch).

En primer lugar, se consolidó la jerarquía tripartita de runtime (`*_GIT_ROOT`, `*_LOCAL_ROOT`, `*_DATA_DIR`) en ambos repositorios (`RSA-Concentrador-Presas` y `RSA-Coordinometro-Optico`), permitiendo actualizar código vía `update.sh` sin alterar configuraciones de campo ni duplicar almacenamiento. Se estandarizó la resolución dinámica de variables `${RSA_*}` en Python y se creó el deserializador tolerante `LeerArchivoJSON.py`. Posteriormente, se solventó la incompatibilidad crítica de nomenclatura fotográfica con Windows 11 ejecutando la migración in situ de 5.805 fotos en `DER-N3-X` mediante `migrar_nombres_fotos.py` (`os.rename()` sobre inodos ext4 con cero consumo adicional de MicroSD).

Finalmente, se desarrolló e integró el pipeline en dos etapas: la sincronización de transporte Ad-Hoc (`sincronizar_fotos_coordinometros.py`) con `rsync -avz --partial` y barra de progreso interactiva, seguida de la purga atómica remota condicionada estrictamente a código de retorno 0; y la etapa de empaquetado (`empaquetador_fotos.py`) que organizó las 5.805 fotos en 17 archivos `.zip` mensuales (`2022-01.zip` a `2023-05.zip`, ~520 MB) con verificación previa CRC-32 (`zf.testzip()`). Todo el proceso fue verificado al 100% en el hardware real, se formalizó el **ADR-029** institucional (**ADR-003** de proyecto) y se generaron los contextos técnicos correspondientes.

**Decisiones y Cambios:**
- Formalización del **ADR-029** (institucional) y **ADR-003** (proyecto): desacoplamiento tripartito de runtime y pipeline de empaquetado mensual ZIP con soporte append (`'a'`) y verificación CRC-32 previa a la limpieza de buffer `inbox/`.
- Estandarización de nombres de fotos a `<ESTACION>_YYYYMMDD_HHMMSS.jpg`, eliminando espacios y dos puntos que bloqueaban Windows 11.
- Purga remota atómica en coordinómetros condicionada exclusivamente al código de salida exitoso (0) de `rsync`.
- Aislamiento de registros de log dentro del bucle de escritura ZIP para mantener una barra de progreso limpia en terminal.
- Generación de contextos técnicos en `docs/context/` para `sincronizar_fotos_coordinometros.py`, `empaquetador_fotos.py`, `setup_adhoc_concentrador.py`, `config.py`, `setup_adhoc_coordinometro.py`, `migrar_nombres_fotos.py` y `LeerArchivoJSON.py`.

**Scripts/Comandos relevantes:**
```bash
# 1. Migración atómica in-place en coordinómetro DER-N3-X (5.805 fotos)
python3 scripts/migrar_nombres_fotos.py --dir /home/pi/data/coordinometro/fotos --estacion DER-N3-X

# 2. Descarga incremental y purga atómica remota desde el concentrador
python3 scripts/sincronizar_fotos_coordinometros.py --purge

# 3. Empaquetado mensual en ZIP estándar con test de integridad CRC-32
python3 scripts/empaquetador_fotos.py --clean-inbox

# 4. Verificación de integridad de los 17 archivos ZIP generados
for f in /home/rsa/data/concentrador/fotos/DER-N3-X/*/*.zip; do
    python3 -m zipfile -t "$f"
done
```
---
