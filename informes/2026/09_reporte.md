# Reporte de evidencias - Septiembre 2026

---

# Introducción

El presente documento corresponde al detalle de las actividades desarrolladas durante el mes de septiembre de 2026 en el marco de las tareas de la "Red Sísmica del Austro (RSA)". Específicamente se aportan los medios de verificación de las tareas realizadas y su correspondencia con el objeto del contrato.

---

# Avances en los objetos del contrato

Durante el mes de septiembre de 2026 se trabajó en los siguientes objetos del contrato:

🔲 Ensamblado, configuración e instalación de estaciones digitales con 6 componentes para la actualización y ampliación de la red de monitoreo sísmico, integrando arquitecturas IoT para la transmisión distribuida de datos. (objeto 1)

🔲 Implementación de funcionalidades de autodetección de eventos mediante técnicas basadas en machine learning y acceso a datos en tiempo real. (objeto 2)

✅ Ensamblado, configuración e instalación de acelerógrafos de 3 componentes con integración a la red de digitalizadores de 6 componentes.  (objeto 3)

✅ Configuración, instalación y mantenimiento de los sistemas desarrollados para el monitoreo automatizado de variables estáticas en las presas propiedad de Elecaustro.  (objeto 4)

🔲 Administración de la red de sensores distribuidos en la presa, garantizando la recolección y transmisión precisa de datos en tiempo real. (objeto 5)

✅ Implementación de una solución de gestión y preservación de datos para el manejo del almacenamiento en las estaciones de monitoreo.  (objeto 6)

✅ Implementación de un sistema de visualización y monitoreo de métricas operativas basado en protocolos de telemetría ligera.  (objeto 7)

---

## Ensamblado, configuración e instalación de acelerógrafos de 3 componentes con integración a la red de digitalizadores de 6 componentes.

- Se implementó y validó en hardware real el plan de resiliencia del pipeline de adquisición (`ADR-018`) tras el diagnóstico del incidente en la estación `CHA01`, estableciendo una arquitectura de defensa en cuatro capas bajo `systemd` (`rsa-acelerografo.service`) con auto-reinicio en 5 segundos, reseteo físico por hardware del microcontrolador dsPIC33 vía pin MCLR (`reset_master`) en `ExecStartPre`, purga y fijación de permisos `0666` en el named pipe (`/tmp/my_pipe`) y reintentos con backoff exponencial en `stream_processor.py`.
- Se desarrolló e instaló el script helper `wait_for_ntp.sh` en `/usr/local/bin/wait_for_ntp` para ejecutar una espera activa dinámica mediante `ntpstat` (con timeout de 120 s) en la directiva `ExecStartPre`, sincronizando el reloj antes del arranque de `registro_continuo`, eliminando desfasajes temporales en archivos `.dat` y en el Ring Buffer y optimizando el tiempo de inicio tras reinicio en un 44%.
- Se estableció el protocolo de saneamiento y control de versiones en la flota de acelerógrafos desacoplando desarrollo de producción mediante sincronización selectiva hacia la rama `main`, excluyendo deliberadamente la documentación técnica interna (`docs/`) y las reglas del agente para optimizar el almacenamiento flash en las Raspberry Pi.
- Se formalizó y promovió el release de producción bajo SemVer `v4.5.2` con tag Git inmutable, ejecutando el despliegue escalonado en las estaciones piloto de campo `CHA01` y `CHA02`, incorporando en `scripts/setup/update.sh` el saneamiento incondicional de permisos de usuario (`rsa:rsa` y `u+rwX,g+rwX`) en `log-files/` para erradicar fallos de escritura (`PermissionError: [Errno 13]`) en `gestor_archivos_acq.py`.
- Se detectó y catalogó en supervisión operativa una desviación física de nivelación en el eje X del sensor acelerométrico de `CHA02` ($a_x \approx 0.64\text{ g}$), y se creó la skill interactiva institucional `despliegue_release.md` en `RSA-Agent-Toolkit` para estandarizar futuras promociones a producción respetando la directiva de seguridad SSHFS.
- Se diagnosticó y mitigó una contingencia crítica en `CHA02` provocada por fallas selectivas de DNS corporativo hacia `googleapis.com` (`192.170.100.250`), acumulación de 761 archivos MiniSEED ($\approx 2.6\text{ GB}$), concurrencia horaria de instancias del gestor sin exclusión mutua, livelock de memoria RAM (98.5%) y swap thrashing con asfixia de red y bloqueo de acceso SSH, mitigándose temporalmente desde Google Drive (movimiento a papelera para devolver error 404 instantáneo y reabrir puertos TCP 22 y 5000) y diseñándose medidas correctivas con exclusión mutua atómica (`fcntl.flock`), procesamiento por lotes acotado y comandos de reinicio por MQTT.
- Se auditó y reestructuró la planificación técnica de la red acelerográfica de monitoreo de salud estructural (SHM - Acelerógrafo V1.4) en topología Daisy Chain con RS485 a 2 Mbps y sincronización física (retardos de 1.2 µs Concentrador-Nodo y 14.0 ns entre nodos), formalizando la `Planificacion 3.md` (144 horas) con reemplazo de zócalos MicroSD con contacto mecánico de *card-detect*, diseño de arquitectura de Doble Búfer (Ping-Pong) en RAM de 512 bytes en microcontroladores dsPIC33EP256MC202 para desacoplar pulsos de tiempo de la escritura flash, integración del acelerómetro ADXL355 vía SPI y desarrollo de software en Python para volcado y graficación en pruebas de co-localización (*shake/tap test*).

---

## Configuración, instalación y mantenimiento de los sistemas desarrollados para el monitoreo automatizado de variables estáticas en las presas propiedad de Elecaustro.

- Se ejecutó una auditoría técnica y revisión sistemática de los antecedentes institucionales de la RSA en auscultación geotécnica e instrumentación de presas de Elecaustro (Chanlud y Labrado), identificando cuellos de botella y requerimientos de modernización.
- Se estructuró y redactó formalmente la planificación y especificación técnica de ingeniería (`planificacion-12.md`) para el desarrollo de un sistema electromecánico multiplexado de 24 canales para galgas extensométricas de 4 hilos orientado al monitoreo automatizado de deformaciones en presas, gobernado por una arquitectura distribuida con microcontrolador maestro ESP32 y esclavo PIC16F628A para el control determinista de motores con puente H L293D y encoders ópticos por interrupciones de hardware.
- Se especificó la cadena de digitalización analógica diferencial de 24 bits mediante el convertidor ADC HX711 acoplado a un banco de relés K1..K5 con conmutación bimodal (deformación unitaria y compensación de temperatura) y tiempos muertos defensivos de 20 ms para supresión de transitorios, integrando almacenamiento autónomo en tarjeta MicroSD (FAT32/CSV), menús LCD y telemetría SCADA por RS-232, estructurando siete fases de desarrollo técnico con criterios cuantitativos de conformidad metrológica.

---

## Implementación de una solución de gestión y preservación de datos para el manejo del almacenamiento en las estaciones de monitoreo.

- Se implementó en producción la gobernanza integral del ciclo de vida de los eventos sísmicos extraídos (`/home/rsa/data/eventos-extraidos/`) y la arquitectura de rescate desatendido *Store and Forward* en `gestor_archivos_acq.py` (`ADR-022`), subiendo automáticamente a Google Drive con reintentos exponenciales 9 eventos pendientes no transmitidos en caliente, configurando retención temporal a 7 días y estableciendo una jerarquía estricta de desalojo ante almacenamiento crítico (< 5%) que protege prioritariamente los eventos sísmicos frente a datos continuos, purgando desatendidamente 1.546 archivos caducados en disco.
- Se formalizó y validó el Principio de Registro Espejo (`ADR-021`) en `uploaded_files_registry.json`, acotando la caché de deduplicación de Google Drive exclusivamente a los archivos físicamente existentes en el almacenamiento local mediante validación preventiva de rutas (`os.path.isdir`), poda periódica automática de entradas huérfanas en `drive_status_manager.py` y soporte del comando CLI `--purge-registry`, reduciendo el archivo de 2.908 a 1.371 entradas activas sincronizadas 1:1 con disco.
- Se implementó y validó en hardware el mecanismo autónomo de auto-recuperación (*Self-Healing*) en el procesador de flujo (`stream_processor.py` / `ADR-006`), incorporando verificación periódica de inodos (`os.fstat(fd).st_ino == os.stat(pipe_path).st_ino`) para detectar la recreación del named pipe (`/tmp/my_pipe`) tras reinicios de la adquisición en C, cerrando descriptores desvinculados y reconectando en caliente con retroceso exponencial para asegurar la continuidad del streaming hacia el Ring Buffer y la generación de MiniSEED.
- Se robusteció el módulo auditor `DriveWatchdog` (`drive_watchdog.py`) con filtrado estricto contra el sistema de ficheros real para erradicar falsas alarmas por registros históricos huérfanos (`failed_uploads_protected`), auditoría de backlog de subidas pendientes (> 3 archivos) y telemetría de almacenamiento libre en disco mediante `shutil.disk_usage`.

---

## Implementación de un sistema de visualización y monitoreo de métricas operativas basado en protocolos de telemetría ligera.

- Se saneó y normalizó el repositorio central de telemetría en el Stack TIG (`RSA-Intern-TIG-MQTT`), eliminando series de 4 estaciones de prueba en InfluxDB v2 (`ACL1`, `SIM01`, `TEST01`, `TEST02`) y migrando exitosamente 234.439 puntos de telemetría y 12 eventos multidetección hacia identificadores consolidados (`TEST`, `TENG`) mediante el script `migrate_stations.py` en el contenedor `rsa-db-sync`.
- Se diseñó y desplegó el modelo jerárquico de alertas en Grafana (`ADR-019`), actualizando el monitor central 24/7 `SeismicMonitor` a la versión 8 con una matriz minimalista de 3 columnas (`Estación`, `Conectividad`, `Diagnóstico Principal`), motor de votación Flux con 7 niveles de severidad (`Offline` > `Adquisicion` > `Sensor` > `Disco` > `Drive` > `Temperatura` = `Memoria` > `OK`), coloreado integral de fila y enlaces de salto analítico hacia el dashboard `Health`.
- Se consolidó el dashboard `Health` a la versión 13 en `services/grafana/provisioning/dashboards/health.json`, estructurando sus 18 paneles en 4 filas colapsables temáticas, eliminando errores HTTP 500 en InfluxDB mediante mapeo determinista discreto de cadenas hexadecimales en `throttled`, implementando el patrón canónico de deduplicación de transiciones de estado en Flux (`difference()` y filtrado de deltas) en paneles de estado y Google Drive, y preservando el muestreo continuo a 5 minutos en el panel de integridad triaxial y deriva de reloj con capacidad de 30 filas.
- Se unificó la cadencia de evaluación y publicación en `mqtt_coordinator.py` a 300 segundos (5 minutos) con QoS 1 y mensajes retenidos, sincronizando en una ráfaga coherente la salud del hardware (`telemetry/health`), adquisición continua (`status/acquisition`), integridad física del sensor ADXL355 en reposo (`status/sensor`) mediante `SensorWatchdog` y sincronización con Google Drive (`status/drive`) mediante `DriveWatchdog`, habilitando además la orden remota `cmd/stop_acquisition_safety` para detención de contingencia por hardware.
- Se mitigaron condiciones de carrera LWT ante microcortes de red implementando el latido periódico (*Heartbeat*) de `telemetry/state` cada 300 segundos con retención (`ADR-023`), preservando el timestamp de inicio de sesión (`last_state_change`) para garantizar la exactitud en el cómputo de tiempo de actividad ininterrumpido (*uptime*) y erradicar falsos reportes de desconexión.
- Se desarrolló y desplegó el task-script `diagnostico.sh` (`/usr/local/bin/diagnostico`) y se actualizó `ayuda.sh`, proporcionando diagnósticos modulares y reportes ejecutivos sintetizados de 25 líneas en `diagnostico_report.log`, validándose la plataforma mediante un simulador interactivo de alertas MQTT de 5 canales (`simulador_alertas_mqtt.py`) ejecutado con entorno virtual efímero (`ejecutar_simulador.sh`).

---

# Conclusiones del mes

- Se fortaleció de forma integral la resiliencia operativa y la gobernanza de software de la red de acelerógrafos de 3 componentes mediante la implementación de defensa en profundidad en adquisición (ADR-018), sincronización dinámica NTP previa al arranque, el release de producción v4.5.2 desplegado en campo, el auto-recuperamiento en caliente de pipes de streaming (ADR-006) y la planificación de la red SHM (Planificación 3).
- Se garantizó la soberanía y preservación local de los datos sismológicos en las estaciones remotas mediante la arquitectura Store & Forward y retención de eventos extraídos (ADR-022), el principio de Registro Espejo 1:1 con disco (ADR-021) y el diagnóstico forense de contingencias de red corporativa y livelock de memoria en CHA02.
- Se consolidó la plataforma de observabilidad y monitoreo centralizado con la implementación del modelo de alertas jerárquicas en Grafana (ADR-019), la optimización en Flux de los dashboards SeismicMonitor y Health v13, la unificación de telemetría a 5 minutos, la resiliencia LWT con latido periódico (ADR-023) y la formulación técnica del nuevo sistema automatizado de auscultación multiplexada en presas (planificacion-12.md).
