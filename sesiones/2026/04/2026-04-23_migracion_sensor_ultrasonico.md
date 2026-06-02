---
fecha: 2026-04-23
temas: [sensor, ultrasonico, dsp, esp32]
entorno: [edge-device]
---
# Registro de Sesión: Evolución y Migración de Sensor Ultrasónico a Arquitectura Edge DSP

# Actividad del 2026-04-23

# Hitos de la jornada:
Se analizó la metodología heredada de un sensor ultrasónico de nivel diseñado inicialmente con un microcontrolador dsPIC33FJ32MC202. Se identificó que el enfoque analítico original, basado en el filtrado digital IIR y la estimación del Tiempo de Vuelo (TOF) mediante interpolación parabólica del pico de la envolvente, era susceptible a distorsiones de fase y al fenómeno de timing walk. Estos factores provocaban inestabilidad temporal en el eco, obligando a usar promedios estadísticos pesados que elevaban el tiempo de respuesta ineficientemente.

Posteriormente, se evaluó una iteración del proyecto que empleaba correlación cruzada en Python ejecutada en una Raspberry Pi alimentada vía RS485. Aunque matemáticamente superior por aislar el eco mediante remuestreo sub-muestral, la topología de hardware presentaba cuellos de botella severos en el bus de datos y una alta fragilidad al depender de un sistema operativo de propósito general. Para resolver esto y retornar a una arquitectura de sensor industrial autocontenido (Edge Computing), se propuso la migración total del procesamiento de señales digitales (DSP) a un microcontrolador moderno.

Se descartó el uso del ESP32-C6 debido a su arquitectura RISC-V carente de Unidad de Punto Flotante (FPU), lo cual penalizaría severamente los cálculos intensivos requeridos para la interpolación (FFT) y la correlación cruzada matemática. En su lugar, se validó la selección del ESP32-S3, aprovechando sus instrucciones vectoriales nativas. Se estableció una hoja de ruta para la adquisición analógica que prioriza el uso del ADC interno mediante acceso directo a memoria (DMA) continuo, con la contingencia de integrar un ADC externo de topología SAR (como el MCP33131) si la no-linealidad inherente del chip degrada la nitidez del pico de correlación.

**Decisiones y Cambios:**

- Abandono de Detección de Pico: Sustitución definitiva del cálculo de TOF basado en envolventes por algoritmos de Correlación Cruzada digital.
- Topología Edge DSP: Eliminación del esquema distribuido (Sensor -> RS485 -> Raspberry Pi) para procesar la señal de forma nativa en el dispositivo de captura.
- Descarte de Hardware Incompatible: El ESP32-C6 fue rechazado por su falta de FPU y alto nivel de ruido en el ADC, lo cual es inviable para operaciones de remuestreo matemático.
- Selección de Hardware: Aprobación del ESP32-S3 como núcleo de procesamiento principal por su soporte de aceleración matemática para DSP.
- Estrategia de Adquisición Analógica en 2 Fases: 
  - Fase 1: Muestreo de la señal ultrasónica de 40KHz  utilizando el ADC interno del ESP32-S3 en modo continuo acoplado a I2S/DMA.
  - Fase 2: Integración de un ADC externo vía SPI dedicado de alta linealidad si el ruido y distorsión (INL/DNL) del ADC interno impiden alcanzar la precisión milimétrica objetivo.

# Scripts/Comandos relevantes:

A continuación, se documenta el núcleo del script de procesamiento histórico en Python evaluado durante la sesión, el cual define el estándar matemático (remuestreo, correlación y compensación de temperatura) que deberá ser portado a C/C++ aprovechando las extensiones del ESP32-S3:

```bash
import numpy as np
from scipy import signal
from scipy.signal import correlate
import math

# Constantes de configuración de la trama 
sizeTramaShort = 702
periodoMuestreo = 5
factorResample = 10
periodoResample = periodoMuestreo / factorResample

# Función fundamental de remuestreo para resolución sub-muestral 
def RemuestrearOffset(arrayMuestras, offset):
    npMuestras = np.array(arrayMuestras)
    # Multiplica la resolución de la señal para correlación fina
    resampleMuestras = signal.resample(npMuestras, len(npMuestras) * factorResample)
    resampleMuestras = resampleMuestras - offset
    return resampleMuestras

# Lógica de cálculo de velocidad del sonido y correlación [cite: 893, 894, 895]
def calcular_distancia(senal_cruda, temperatura_sensor, referencia_ideal):
    # Compensación de temperatura física
    Vsonido = 331.45 * math.sqrt(1 + (temperatura_sensor / 273))
    
    # Preparación de señales
    senal1 = RemuestrearOffset(referencia_ideal, 0)
    senal2 = RemuestrearOffset(senal_cruda, 517)
    nsamples = np.size(senal1)
    
    # Motor de Correlación Cruzada Matemática
    xcorr = correlate(senal1, senal2)
    dt = np.arange(1 - nsamples, nsamples)
    recovered_time_shift = dt[xcorr.argmax()]
    
    # Extracción de TOF sub-microsegundo
    desfaseTiempo = recovered_time_shift * -1 * periodoResample
    dA = -80.5
    TOF = 1375 + desfaseTiempo + dA
    
    # Retorno de distancia en milímetros
    Distancia = 0.5 * (TOF / 1e6) * Vsonido * 1000
    return Distancia
```