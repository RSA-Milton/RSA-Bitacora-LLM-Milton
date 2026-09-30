---
fecha: 2026-09-23
temas: [diagnostico, drive, dns, memoria, resiliencia]
entorno: [acelerografo]
autor: Milton
---

# Actividad del 2026-09-23 a 2026-09-30 (Diagnóstico de Red DNS, Livelock de Memoria por Concurrencia Drive y Bloqueo de Acceso SSH en Estaciones Acelerográficas)

## Actividad del 2026-09-23 (Diagnóstico de Fallo de Resolución DNS Corporativo en Google APIs)

**Hitos de la jornada:**

Durante esta jornada se investigó la interrupción del servicio de subida automática de archivos hacia Google Drive en estaciones acelerográficas conectadas a una red corporativa administrada (`CHA01` y `CHA02`). Mediante la herramienta de diagnóstico automatizado `diagnostico drive`, se detectaron 11 fallos consecutivos de subida protegidos contra borrado y una pérdida absoluta de conectividad con `www.googleapis.com` con el mensaje `Temporary failure in name resolution`.

El análisis técnico demostró que la conectividad IP y la salida a Internet funcionaban con normalidad (ping a `8.8.8.8` con 0% de pérdida y 29 ms), pero el servidor DNS corporativo asignado por DHCP (`192.170.100.250`) presentaba una falla selectiva: resolvía con éxito dominios como `github.com` y `pool.ntp.org`, pero denegaba o fallaba al resolver la zona completa de Google (`www.google.com` y `www.googleapis.com`). Se constató que las políticas de retención y resiliencia mantuvieron los datos sísmicos 100% seguros en disco (14 GB disponibles, equivalentes a más de 4 meses de autonomía local), protegiendo los archivos contra descarte hasta que el equipo de TI corporativo regularizara los reenviadores o reglas perimetrales del DNS.

**Decisiones y Cambios:**
- Identificación de falla selectiva en servidor DNS corporativo `192.170.100.250` afectando únicamente al ecosistema Google.
- Comprobación de integridad de los datos sísmicos locales y el reloj NTP (`pool.ntp.org`).
- Decisión operativa de respetar las directivas de seguridad corporativa sin alterar configuraciones de red locales, dejando que el equipo de TI repare el servicio.

---

## Actividad del 2026-09-29 (Livelock de Memoria, Concurrencia en Drive y Asfixia de Red en CHA02)

**Hitos de la jornada:**

Se atendió una degradación crítica en la estación remota `CHA02`: la estación reportaba 98.5% de consumo de RAM, dejó de responder conexiones SSH y comenzó a parpadear en telemetría MQTT. Al investigar a fondo, se descubrió que `CHA02` había acumulado una cola masiva de **761 archivos MiniSEED pendientes ($\approx 2.6\text{ GB}$)** tras la interrupción previa de subidas.

Al reanudarse el acceso o restablecerse el espacio, el script `gestor_archivos_acq.py` entró en un bucle voraz e ininterrumpido de transferencias HTTPS. Al carecer de mecanismo de exclusión mutua (*file lock*), el cron horario de las 16:00 (`registrocontinuo.sh`) lanzó una segunda instancia concurrente en segundo plano (`&`), empujando la memoria al 100% y desatando un **livelock por memoria (*swap thrashing*)** en la tarjeta microSD. La saturación del canal de subida provocó *bufferbloat*, impidiendo que el puerto 22 respondiera a tiempo a las conexiones SSH entrantes (`Connection timed out`).

Se aplicaron maniobras de mitigación desde la red y la nube:
1. Se disparó por MQTT el comando `stop_acquisition_safety`, logrando una descompresión inmediata del 2% de RAM al detener temporalmente `rsa-acelerografo.service`.
2. Se envió temporalmente la carpeta de Google Drive de `CHA02` a la **Papelera** desde el navegador. Esto obligó a la API de Drive a responder con error `404 Not Found` en $<100\text{ ms}$ sin transferir bytes masivos, cortando la asfixia de red y permitiendo que los sockets locales de los puertos 22 y 5000 volvieran a responder `ABIERTO` en pruebas TCP desde `concentrador-CON00`. No obstante, el estado de livelock de memoria en los daemons de Supervisor persistió, haciendo inviable el reinicio remoto desatendido.

**Decisiones y Cambios:**
- Mitigación en la nube mediante traslado a la papelera de la carpeta de Drive para frenar la saturación de enlace.
- Verificación de reapertura de sockets TCP 22 y 5000 a nivel de stack de red en `CHA02`.
- Planificación de visita presencial a la estación para la semana siguiente para realizar el ciclo de energía físico limpio.

---

## Actividad del 2026-09-30 (Formalización del Diagnóstico Técnico y Plan de Mitigación)

**Hitos de la jornada:**

Se elaboró y registró el documento formal de diagnóstico técnico (`docs/analysis/2026-09-30_diagnostico_saturacion_memoria_y_bloqueo_subidas_drive.md`), consolidando el análisis forense del efecto dominó: DNS/cuota llena → acumulación masiva (761 archivos) → concurrencia de instancias sin *file lock* → livelock de RAM y swap thrashing → asfixia de red y pérdida de SSH.

Se estructuró el backlog de mejoras preventivas para blindar la flota de acelerógrafos:
1. **Exclusión mutua atómica**: Integración de `fcntl.flock` en `gestor_archivos_acq.py` y validación de proceso en `registrocontinuo.sh`.
2. **Dosificación por lotes (*Batch limit*)**: Procesamiento de un máximo configurable de archivos por ciclo (ej. 15 archivos/hora) para evitar saturar el ancho de banda y la memoria.
3. **Comando de reinicio de emergencia en MQTT**: Inclusión del handler `cmd/reboot_system` en `mqtt_coordinator.py`.
4. **Bandera de mantenimiento**: Bloqueo del auto-reinicio horario en `registrocontinuo.sh` si la adquisición fue detenida por comando de seguridad.

**Scripts/Comandos relevantes:**

```bash
# 1. Comprobación de resolución DNS selectiva nativa en Python
python3 -c "
import socket
for host in ['www.google.com', 'github.com', 'pool.ntp.org', 'www.googleapis.com']:
    try:
        print(f'✅ {host:20} -> {socket.gethostbyname(host)}')
    except Exception as e:
        print(f'❌ {host:20} -> Error: {e}')
"

# 2. Diagnóstico de sockets TCP en red local desde concentrador
python3 -c "
import socket
for port in [22, 5000, 80]:
    s = socket.socket(); s.settimeout(2.0)
    res = s.connect_ex(('172.22.150.53', port)); s.close()
    print(f'Puerto {port:5}: {\"ABIERTO\" if res == 0 else \"CERRADO/TIMEOUT\"}')
"

# 3. Comando SSH optimizado de bajo consumo de memoria para reinicio forzado
ssh -o ConnectTimeout=30 -T CHA02 "sudo /sbin/reboot -f"
```
---
