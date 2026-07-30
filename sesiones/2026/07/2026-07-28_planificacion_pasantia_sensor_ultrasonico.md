---
fecha: 2026-07-28
temas: [sensor, ultrasonico, micropython, pasantias]
entorno: [edge-device, pasantias]
autor: Milton
---

# Actividad del 2026-07-28

**Hitos de la jornada:**
Se estructuró el plan de trabajo de 96 horas para pasantías enfocado en la actualización y migración de un sensor ultrasónico de precisión milimétrica. La planificación partió del diseño previo basado en dsPIC33FJ32MC202 (tesis) y del esquema de procesamiento de señales por correlación cruzada digital, definiendo como plataforma objetivo un microcontrolador ESP32 con firmware en MicroPython.

Se acordó una estrategia de migración en 4 fases alcanzables dentro del tiempo disponible. El prototipo sustituirá temporalmente al dsPIC mediante una placa adaptadora provisional soldada en tarjeta perforada con pines compatibles al zócalo dip original, permitiendo acoplar la tarjeta ESP32 a la placa analógica de la maqueta (driver de excitación Tx 40 kHz, acondicionamiento receptores con TL974 y sensor de temperatura DS18B20 1-Wire).

El desarrollo en MicroPython aprovechará la librería `ulab` para la ejecución de FFT y correlación cruzada en el dispositivo, junto con `machine.PWM` para generar las ráfagas a 40 kHz y `machine.I2S`/DMA para la adquisición analógica a tasas $\ge 200\text{ kSPS}$. Las pruebas experimentales se enfocarán en la validación metrológica y evaluación de la repetibilidad/precisión milimétrica ($\le \pm 1\text{ mm}$).

**Decisiones y Cambios:**
- Enfoque de Pasantía: Estructuración en 4 fases (16h, 36h, 24h, 20h = 96h) orientadas a la migración y comprobación de la precisión milimétrica del sensor.
- Adopción de MicroPython: Selección de MicroPython (usando Thonny IDE y librería `ulab`) por su rapidez de prototipado, depuración interactiva y soporte directo de funciones vectoriales.
- Construcción de Adaptador Físico: Sustitución de KiCad/diseño PCB en Fase 3 por la fabricación e inspección de una placa adaptadora provisional en tarjeta perforada con tiras de pines para el zócalo del dsPIC33.
- Delimitación del Alcance: Exclusión de tareas de integración de vertedero o monitoreo en campo para acotar el entregable exclusivamente a la validación de precisión milimétrica en laboratorio/maqueta.

**Scripts/Comandos relevantes:**
```python
import machine
import onewire
import ds18x20
import ulab
from ulab import numpy as np

# Configuración de pulso de excitación a 40 kHz
pwm_tx = machine.PWM(machine.Pin(18), freq=40000, duty=512)

# Lectura de sensor de temperatura DS18B20 (1-Wire)
ow = onewire.OneWire(machine.Pin(4))
ds = ds18x20.DS18X20(ow)
roms = ds.scan()
ds.convert_temp()
temp = ds.read_temp(roms[0])

# Velocidad del sonido compensada por temperatura (m/s)
v_sonido = 331.45 * (1 + temp / 273.15)**0.5

# Correlación cruzada usando ulab (NumPy en MicroPython)
def correlacion_eco(referencia, senal_capturada):
    ref = np.array(referencia)
    sig = np.array(senal_capturada)
    # FFT y cálculo de desfase
    fft_ref = ulab.scipy.signal.fft.fft(ref)
    fft_sig = ulab.scipy.signal.fft.fft(sig)
    corr = ulab.scipy.signal.fft.ifft(fft_ref * np.conj(fft_sig))
    return np.argmax(corr)
```
---
