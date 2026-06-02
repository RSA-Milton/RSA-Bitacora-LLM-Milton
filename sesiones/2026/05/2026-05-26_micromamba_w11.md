---
fecha: 2026-05-26
temas: micromamba, windows, gpo, shell, vscode
entorno: entornos-virtuales
--- 
# Registro de Sesión: Instalación y Configuración de Micromamba en Entorno Corporativo Windows con GPO Restrictivas

# Actividad del 2026-05-26
**Hitos de la jornada:**
Se resolvió la instalación y configuración funcional de micromamba en una estación de trabajo Windows 11 Enterprise sujeta a políticas de grupo (GPO) corporativas altamente restrictivas. El problema principal era múltiple: (1) el instalador estándar vía winget fallaba por restricciones de red/certificados, (2) PowerShell estaba bloqueado por GPO impidiendo la ejecución de cualquier script de perfil o inicialización, y (3) las versiones manuales del binario descargadas inicialmente resultaron ser compilaciones no estáticas que dependían de zlib.dll, generando errores silenciosos y fallos de ejecución.

El enfoque de solución implicó primero identificar que el binario descargado manualmente desde fuentes no oficiales era dinámico, lo que causaba la dependencia faltante. Se localizó la URL correcta del binario estático auto-contenido (micromamba-win-64.exe) desde el repositorio oficial de mamba-org/micromamba-releases. Posteriormente, se ejecutó la inicialización del shell CMD con el flag --root-prefix apuntando al perfil de usuario, respondiendo afirmativamente a la habilitación de rutas largas (long paths support), lo que finalmente permitió que micromamba activate funcionara sin errores. Se documentaron los comandos alternativos con micromamba run como respaldo y se configuró la integración con VS Code.

** Decisiones y Cambios:**

- Se descartó el uso de PowerShell como shell principal debido al bloqueo absoluto de ejecución de scripts por GPO; se adoptó CMD como terminal primaria para todas las operaciones con micromamba.

- Se eligió el binario estático auto-contenido (micromamba-win-64.exe) en lugar de versiones dinámicas para eliminar dependencias externas como zlib.dll.

- Se habilitó long paths support en el sistema (respondiendo Y durante shell init) para prevenir futuros errores de rutas anidadas, a pesar de las dudas iniciales sobre permisos corporativos.

- Se estableció que el comando micromamba run -n <env> <command> es el mecanismo de respaldo robusto cuando activate falla por restricciones del shell padre.

- Se configuró VS Code para usar la ruta directa del intérprete (C:\Users\miltonrsa\micromamba\envs\gpd_py39\python.exe) como estrategia para evitar dependencias de scripts de activación.

**Scripts/Comandos relevantes:**

**Instalación del binario estático**
```cmd
:: Descargar desde la URL oficial
:: https://github.com/mamba-org/micromamba-releases/releases/latest/download/micromamba-win-64.exe

:: Guardar en el perfil de usuario
copy micromamba-win-64.exe C:\Users\miltonrsa\micromamba.exe

:: Verificar versión
C:\Users\miltonrsa\micromamba.exe --version
```
**Inicialización del shell CMD con long paths**
```cmd
C:\Users\miltonrsa\micromamba.exe shell init --shell cmd.exe --root-prefix=C:\Users\miltonrsa\micromamba
:: Responder 'Y' cuando pregunte: "Enter admin mode to enable long paths support?"
```
**Creación del entorno virtual**
```cmd
C:\Users\miltonrsa\micromamba.exe create -n gpd_py39 python=3.9 -c conda-forge -y
```
**Activación del entorno (una vez inicializado)**
```cmd
micromamba activate gpd_py39
```
**Comandos de respaldo sin activación (micromamba run)**
```cmd
:: Instalar paquetes
C:\Users\miltonrsa\micromamba.exe run -n gpd_py39 pip install tensorflow==2.12 obspy

:: Ejecutar script Python
C:\Users\miltonrsa\micromamba.exe run -n gpd_py39 python script.py

:: Abrir VS Code con el entorno
C:\Users\miltonrsa\micromamba.exe run -n gpd_py39 code .
```
**Script de inicio rápido para VS Code (vscode-mamba.bat)**
```batch
@echo off
title VS Code con Micromamba
echo Iniciando VS Code con el entorno 'gpd_py39'...
call C:\Users\miltonrsa\micromamba.exe activate gpd_py39
code .
```
**Diagnóstico de políticas de ejecución en PowerShell**
```powershell
# Verificar políticas de ejecución (todas Undefined)
Get-ExecutionPolicy -List

# Verificar modo de lenguaje (FullLanguage)
$ExecutionContext.SessionState.LanguageMode
```
**Verificación final de instalación**
```cmd
:: Con el entorno activado (gpd_py39)
python -c "import tensorflow as tf; import obspy; print('TensorFlow:', tf.__version__); print('ObsPy: OK')"
```