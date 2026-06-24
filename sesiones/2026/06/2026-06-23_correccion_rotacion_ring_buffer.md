---
fecha: 2026-06-23
temas: [streaming, bugfix, configuracion]
entorno: [acelerografo]
autor: Milton
---

# Actividad del 2026-06-23

**Hitos de la jornada:**
Se realizó un diagnóstico en caliente del problema de crecimiento indefinido del archivo del buffer circular (`ring_20260618_235730.bin`), el cual acumuló 409,987 tramas (~979 MB) sin rotar ni aplicar la política FIFO. Se confirmó que el origen técnico del fallo radica en que el dsPIC mantiene fija la fecha en el día 2026-06-18. La corrección original estaba diseñada únicamente para `diff_dias == 1` (crossover de medianoche), por lo que a partir del segundo día (`diff_dias >= 2`), la generación de nombres calculados volvía a basarse en la fecha del dsPIC. Esto generó colisiones de nombres periódicas que truncaban silenciosamente los archivos activos.

Se elaboró un plan de acción definitivo que incluyó 5 correcciones prioritarias: generalizar la condición de desfase a `diff_dias >= 1`, implementar un sufijo incremental anti-colisión, reanudar la escritura en modo append en `_rebuild_index()`, enlazar el StructuredLogger para hacer visible la operación del búfer circular, y configurar pruebas unitarias correspondientes. Se implementaron estas correcciones en el repositorio de desarrollo y se añadieron 4 pruebas unitarias en `test_ring_buffer_store.py` para cubrir todos los escenarios críticos.

**Decisiones y Cambios:**
- **Generalización de Desfase Temporal:** Cambio de la validación a `diff_dias >= 1` para obligar al renombrado con la fecha del sistema host independientemente del número de días de desfase del dsPIC.
- **Sufijo Incremental Anti-Colisiones:** Agregar lógica de sufijos (`_001`, `_002`) si `nuevo_path` ya existe en disco para proteger archivos de truncamientos accidentales.
- **Reanudación No Destructiva en Rebuild:** Modificación en `_rebuild_index()` para abrir el último archivo del índice en modo `"ab"` y rellenar las variables del archivo activo (`self._archivo_activo_inicio`, etc.), garantizando que no se pierdan datos al reiniciar el daemon.
- **Logger Activo:** Paso de la instancia `self._logger` de `StreamProcessor` a `RingBufferStore` para registrar operaciones críticas (`RING_ROTATE`, `RING_CLEANUP`, etc.).

**Scripts/Comandos relevantes:**
```bash
# Detener el servicio daemon en producción
sudo supervisorctl stop stream_processor

# Sincronizar y desplegar archivos modificados
bash /home/rsa/projects/acelerografo-rsa/menu.sh
```

---

# Actividad del 2026-06-24

**Hitos de la jornada:**
Se corrigieron fallos de aserción en los tests unitarios (`test_naming_archivos_ring` y `test_colision_nombre_sufijo`) causados porque las marcas de tiempo históricas de los tests activaban la corrección `diff_dias >= 1`, provocando desfases con la fecha actual del host y evitando la colisión de nombres debido a diferencias de segundos. Las pruebas fueron ajustadas para utilizar marcas de tiempo dinámicas (`utcnow()`), logrando una ejecución exitosa de la suite completa (24/24 tests pasados en verde).

Se procedió al restablecimiento de la producción, eliminando el archivo corrupto gigante y permitiendo que el daemon operara. Tras 20 horas de ejecución continua, se verificó el correcto funcionamiento en producción: la rotación se realiza cada 5 minutos exactos, el cambio de día se procesó exitosamente a medianoche ajustándose a `ring_20260624_000000.bin` y la reanudación del servicio tras un reinicio funcionó de forma correcta (generando un archivo de 1.3 MB en modo append sin truncamientos).

Finalmente, a petición del usuario, se ajustó la cuota máxima del búfer circular a **210 MB** (equivalente a ~24.4 horas de datos continuos) modificando el parámetro `max_size_mb` en la plantilla de configuración del dispositivo para evitar consumos de disco innecesarios.

**Decisiones y Cambios:**
- **Optimización de Cuota del Búfer Circular:** Reducción de la cuota máxima a `210 MB` en `configuracion_dispositivo.json.template` (aproximadamente 24 horas continuas a la tasa real de 8.6 MB/hora), evitando el sobredimensionamiento original de 500 MB (~2.4 días).
- **Pruebas Dinámicas Robustas:** Modificación de aserciones en los tests unitarios para usar `datetime.datetime.utcnow()`, eliminando falsos positivos en entornos donde el desfase de fecha es variable.

**Scripts/Comandos relevantes:**
```bash
# Ejecutar tests unitarios en el entorno virtual de producción
/home/rsa/projects/acelerografo-rsa/.venv/bin/python3 /home/rsa/projects/acelerografo-rsa/scripts/streaming/test_ring_buffer_store.py

# Reiniciar daemons de Supervisor para recargar la nueva configuración
sudo supervisorctl restart stream_processor
sudo supervisorctl restart mqtt_coordinator

# Monitorear cantidad de archivos y tamaño del búfer
ls -1 /home/rsa/data/ring-buffer/ | wc -l
du -sh /home/rsa/data/ring-buffer/
```
