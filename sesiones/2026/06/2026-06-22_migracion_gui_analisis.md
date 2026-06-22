---
fecha: 2026-06-22
temas: [gui, entornos-virtuales, windows, refactorizacion]
entorno: [acelerografo]
autor: Milton
---
# Actividad del 2026-06-22

**Hitos de la jornada:**
Se llevó a cabo la exportación e independización de las interfaces gráficas (GUI) de sismología (`mseed_event_extractor.py` y `mseed_event_inspector.py`) desde el repositorio `RSA-InferenciaGPD` al nuevo repositorio exclusivo para análisis de datos `RSA-Analisis-Datos`. Se estructuró este repositorio en una arquitectura multi-entorno utilizando la carpeta `envs/acelerografo/` para dependencias, `src/acelerografos/` para el código y `bin/` para lanzadores automatizados de Windows.

Adicionalmente, se configuró e instaló un entorno virtual limpio de Conda/Micromamba llamado `rsa_acelerografo` con Python 3.9. Se implementó una estrategia de instalación mixta para sortear la restricción de compiladores C++ (Microsoft Visual C++ 14.0) en Windows 11 Enterprise: ObsPy y sus dependencias complejas se cargaron precompiladas desde conda-forge, mientras que TensorFlow 2.12.0 se instaló desde pip mediante su wheel binaria. Finalmente, se eliminó la dependencia a la librería `python-dotenv` y la carga de archivos `.env` en los scripts GUI, reemplazándola con resoluciones dinámicas nativas mediante la librería estándar `os`.

**Decisiones y Cambios:**
- Se migró e independizó el extractor y el inspector de visualización de archivos miniSEED.
- Se reestructuró la configuración de dependencias a un subdirectorio `envs/acelerografo/` en lugar de la raíz para admitir múltiples proyectos futuros en el monorrepo.
- Se implementó un lanzador Windows Batch `bin/run_acelerografo.bat` que levanta la GUI en el subproceso del entorno virtual de forma transparente y segura usando `micromamba run` y corrige la ruta de ejecución activa mediante `cd /d "%~dp0.."`.
- Se eliminaron las llamadas de `dotenv` y los archivos de configuración `.env` en el código de las GUI, simplificando el arranque y removiendo dependencias de terceros no deseadas en Windows 11.
- Se solucionó el error de compilación de binarios C++ delegando dependencias científicas a Conda e instalando TensorFlow mediante pip.

**Scripts/Comandos relevantes:**

**Eliminar entorno virtual obsoleto:**
```cmd
C:\Users\miltonrsa\micromamba.exe env remove -n gpd_py39 -y
```

**Crear e instalar el entorno virtual rsa_acelerografo:**
```cmd
C:\Users\miltonrsa\micromamba.exe env create -f envs/acelerografo/environment.yml -y
```

**Comando de lanzamiento aislado del menú de análisis:**
```cmd
C:\Users\miltonrsa\micromamba.exe run -n rsa_acelerografo python src/acelerografos/main.py
```
