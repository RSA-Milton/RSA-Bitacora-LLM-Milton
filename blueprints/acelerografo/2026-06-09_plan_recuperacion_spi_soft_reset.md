# Blueprint — Plan de Recuperación de SPI por Software (Soft-Reset)

*   **Fecha:** 2026-06-09
*   **Entorno:** Acelerógrafo RSA
*   **Autor:** Agente Antigravity
*   **Estado:** Listo para Implementación / Compilación Manual

---

## 1. Contexto y Problema Técnico

En una de las estaciones acelerográficas instaladas en una locación remota (sin acceso físico), se omitió la instalación del jumper físico que conecta el pin GPIO `MCLR` (Pin 28 en WiringPi / Pin 38 físico) de la Raspberry Pi con el pin `MCLR` del microcontrolador dsPIC33EP.

### El Síntoma:
Al detener el proceso de adquisición continua (`registro_continuo`) y volverlo a arrancar (por ejemplo, con `registrocontinuo stop` y `registrocontinuo start`), la comunicación SPI se rompe por completo y el acelerógrafo queda inoperable.

### El Diagnóstico:
1.  **Falta de Reset Físico:** Al no estar conectado el jumper de `MCLR`, la ejecución de `reset_master` (que conmuta el pin `MCLR` a `LOW` y luego a `HIGH`) no tiene ningún efecto físico sobre el dsPIC. El microcontrolador sigue encendido y ejecutando su firmware en caliente.
2.  **Bloqueo de la Máquina de Estados SPI (Deadlock):**
    *   Si la Raspberry Pi se detiene mientras lee datos (por ejemplo, en medio de la función `NuevoCiclo()` leyendo los 2506 bytes de la trama de datos), el dsPIC se queda en el estado `banLec = 2` esperando recibir el byte delimitador de cierre **`0xF3`** para finalizar la lectura y limpiar la bandera.
    *   Al reiniciar la RPi, esta inicia el protocolo de enlace enviando nuevos comandos como `0xA6` (Obtener Referencia de Tiempo) o `0xA4` (Enviar Tiempo Local).
    *   Como el dsPIC sigue atrapado en `banLec == 2` esperando `0xF3`, toma cualquier comando nuevo (`0xA6`, `0xA4`) y sus datos asociados como si fueran bytes dummy sobrantes del ciclo de lectura de datos anterior, incrementando su contador interno de lectura e ignorando por completo la semántica del nuevo comando.
    *   Esto rompe de forma permanente el sincronismo lógico de la comunicación en el bus SPI.

---

## 2. Solución Propuesta (Soft-Reset SPI)

Para solucionar el bloqueo sin modificar el firmware del dsPIC (que no puede reprogramarse remotamente) y sin acceso físico, la Raspberry Pi debe forzar al dsPIC a salir de cualquier estado de comunicación SPI incompleto.

### Mecanismo de Escape:
Se envía una ráfaga de bytes correspondientes a los **delimitadores de fin de comando** de todas las funciones del dsPIC. Esto garantiza que sin importar en qué estado estuviera el buffer de recepción SPI del microcontrolador, este consumirá el delimitador esperado y restaurará sus banderas lógicas a su estado inicial (`IDLE` / Listo para recibir comandos).

Los bytes que se transmiten consecutivamente son:
*   **`0xF0`**: Cierra `banOperacion` (la limpia a `0`).
*   **`0xF1`**: Cierra `banMuestrear` (la limpia a `0`).
*   **`0xF3`**: Cierra `banLec` (sale de la lectura de trama, `banLec = 0`).
*   **`0xF4`**: Cierra `banEsc` de la hora local (restablece `banEsc = 0` y calcula el tiempo).
*   **`0xF5`**: Cierra `banSetReloj` (restablece `banSetReloj = 1`).
*   **`0xF6`**: Cierra `banEsc` de la referencia de tiempo (restablece `banEsc = 0` y configura referencia).

---

## 3. Estructura del Script de Emergencia

Se ha creado un script operativo de emergencia independiente:
*   **Ruta del archivo:** [registro_continuo_4.5.0_soft_reset.c](file:///c:/Users/miltonrsa/Documents/git/rsa/RSA-Acelerografo/scripts/operation/acelerografo/registro_continuo_4.5.0_soft_reset.c)

Este script mantiene el 100% de la funcionalidad de `registro_continuo_4.5.0.c` e incorpora los siguientes cambios:

### A. Prototipo y Definición de la Función de Purga:
```c
void PurgarEstadoSPIdsdPIC();

// Definición:
void PurgarEstadoSPIdsdPIC()
{
    printf("Ejecutando Soft-Reset SPI de rescate en el dsPIC...\n");
    write_log("INFO", "Ejecutando Soft-Reset SPI de rescate en el dsPIC...");
    
    // Delimitadores de fin de comandos del dsPIC (0xF0 - 0xF6)
    unsigned char bytes_escape[] = {0xF0, 0xF1, 0xF3, 0xF4, 0xF5, 0xF6};
    
    for (int k = 0; k < sizeof(bytes_escape); k++)
    {
        bcm2835_spi_transfer(bytes_escape[k]);
        bcm2835_delayMicroseconds(TIEMPO_SPI); // Retardo de 10us requerido por el dsPIC
    }
    
    delay(50); // Tiempo de asentamiento para que el dsPIC actualice sus registros
}
```

### B. Inserción de la Llamada de Limpieza:
Dentro de `ConfiguracionPrincipal()`, justo después de inicializar el bus SPI master y sus parámetros de reloj/polaridad, pero **antes** de habilitar la interrupción GPIO del pin `P1` (`wiringPiISR`):

```c
    // ... Parámetros SPI del bcm2835 ...
    bcm2835_spi_chipSelect(BCM2835_SPI_CS0);
    bcm2835_spi_setChipSelectPolarity(BCM2835_SPI_CS0, LOW);

    // NUEVO: Purga de la máquina de estados SPI del dsPIC para rescate en caliente
    PurgarEstadoSPIdsdPIC();

    // Configuracion libreria WiringPi e Interrupción GPIO
    wiringPiSetup();
    pinMode(P1, INPUT);
    pinMode(MCLR, OUTPUT);
    pinMode(LedTest, OUTPUT);
    wiringPiISR(P1, INT_EDGE_RISING, ObtenerOperacion);
```

---

## 4. Instrucciones de Compilación y Ejecución Manual

Dado que no se desean alterar los scripts de despliegue generales (`update.sh` / `deploy.sh`) para evitar compilaciones accidentales en otras estaciones que sí disponen de jumper físico, la compilación de este binario de emergencia debe realizarse **manualmente** en el equipo remoto.

### Paso 1: Acceder al directorio de adquisición
En la terminal de la Raspberry Pi de la estación remota, navega al directorio donde residen las librerías locales necesarias para el JSON del acelerógrafo:
```bash
cd /home/rsa/projects/acelerografo-rsa/scripts/operation/acelerografo
```
*(Nota: Ajusta la ruta si el directorio local difiere según la variable `$PROJECT_LOCAL_ROOT`).*

### Paso 2: Compilar el script de emergencia
Ejecuta el compilador `gcc` enlazando las librerías dinámicas del sistema (`bcm2835`, `wiringPi`, `jansson`) y la librería local de lectura JSON (`liblector_json.a` ubicada en el subdirectorio `libraries/`):
```bash
gcc -o registro_continuo_soft_reset registro_continuo_4.5.0_soft_reset.c \
    -I./libraries -L./libraries \
    -llector_json -lbcm2835 -lwiringPi -ljansson -lm -lpthread -O2 -Wall
```

### Paso 3: Reemplazar el binario en caliente (con precaución)
Para implementar el script de rescate en la estación remota:
1.  Detener el servicio de adquisición actual:
    ```bash
    sudo supervisorctl stop registro_continuo   # Si se gestiona por Supervisor
    # O alternativamente:
    # sudo killall -q registro_continuo
    ```
2.  Respaldar el ejecutable original:
    ```bash
    mv /home/rsa/projects/acelerografo-rsa/scripts/acelerografo/ejecutables/registro_continuo /home/rsa/projects/acelerografo-rsa/scripts/acelerografo/ejecutables/registro_continuo.orig
    ```
3.  Copiar el nuevo binario compilado al directorio de ejecutables de producción:
    ```bash
    cp registro_continuo_soft_reset /home/rsa/projects/acelerografo-rsa/scripts/acelerografo/ejecutables/registro_continuo
    ```
4.  Iniciar el servicio de nuevo:
    ```bash
    sudo supervisorctl start registro_continuo
    ```
    *Al arrancar, el nuevo ejecutable purgará la comunicación SPI del dsPIC y se acoplará de inmediato al ciclo de muestreo activo del microcontrolador.*

---

## 5. Plan de Verificación

*   **Prueba de Compilación:** Compilar localmente en la RPi usando el comando provisto en el Paso 2 para asegurar que no existan errores de sintaxis, tipos de datos o prototipos faltantes.
*   **Verificación de logs:** Tras iniciar el servicio con el ejecutable modificado, revisar el archivo de log:
    ```bash
    tail -n 50 /home/rsa/projects/acelerografo/log-files/registro_continuo.log
    ```
    Debe observarse la línea:
    `[Timestamp] - INFO - Ejecutando Soft-Reset SPI de rescate en el dsPIC...`
    seguida del enlace y sincronización de hora exitosa sin bloqueos.
