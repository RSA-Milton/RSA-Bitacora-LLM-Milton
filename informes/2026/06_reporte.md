# Reporte de evidencias - Junio 2026

---

# Introducción

El presente documento corresponde al detalle de las actividades desarrolladas durante el mes de junio de 2026 en el marco de las tareas de la "Red Sísmica del Austro (RSA)". Específicamente se aportan los medios de verificación de las tareas realizadas y su correspondencia con el objeto del contrato.

---

# Avances en los objetos del contrato

Durante el mes de junio de 2026 se trabajó en los siguientes objetos del contrato:

🔲 Ensamblado, configuración e instalación de estaciones digitales con 6 componentes para la actualización y ampliación de la red de monitoreo sísmico, integrando arquitecturas IoT para la transmisión distribuida de datos. (objeto 1)

✅ Implementación de funcionalidades de autodetección de eventos mediante técnicas basadas en machine learning y acceso a datos en tiempo real.  (objeto 2)

✅ Ensamblado, configuración e instalación de acelerógrafos de 3 componentes con integración a la red de digitalizadores de 6 componentes.  (objeto 3)

🔲 Configuración, instalación y mantenimiento de los sistemas desarrollados para el monitoreo automatizado de variables estáticas en la presas propiedad de Elecaustro.  (objeto 4)

🔲 Administración de la red de sensores distribuidos en la presa, garantizando la recolección y transmisión precisa de datos en tiempo real.  (objeto 5)

✅ Implementación de una solución de gestión y preservación de datos para el manejo del almacenamiento en las estaciones de monitoreo. (objeto 6)

🔲 Implementación de un sistema de visualización  y monitoreo de métativas operativas basado en protocolos de telemetría ligera.  (objeto 7)

---

## Implementación de funcionalidades de autodetección de eventos mediante técnicas basadas en machine learning y acceso a datos en tiempo real.

- Se realizó la migración y la independización de las interfaces gráficas y herramientas de análisis sísmico (`mseed_event_extractor.py` y `mseed_event_inspector.py`) desde el repositorio de inferencia `RSA-InferenciaGPD` hacia un repositorio exclusivo denominado `RSA-Analisis-Datos` estructurado en una arquitectura modular multi-entorno.
- Se configuró y desplegó un entorno virtual limpio de Conda/Micromamba (`rsa_acelerografo`) bajo Python 3.9 en el sistema operativo Windows 11 Enterprise, resolviendo las restricciones locales de compilación C++ mediante una estrategia híbrida con conda-forge para la librería ObsPy y pip para la wheel de TensorFlow 2.12.0.
- Se simplificó el arranque de las aplicaciones eliminando la dependencia de carga de archivos de entorno `.env` en favor de resoluciones de sistema dinámicas nativas mediante la librería estándar `os`.
- Se implementó un script de lanzamiento automatizado en Windows Batch para una ejecución segura y transparente para el usuario técnico.

---

## Ensamblado, configuración e instalación de acelerógrafos de 3 componentes con integración a la red de digitalizadores de 6 componentes.

- Se diseñó e implementó un sistema unificado de gestión de configuración de estaciones acelerográficas mediante un esquema maestro descentralizado e hidratación dinámica de plantillas en caliente, aislando el código operativo del control de versiones de Git.
- Se desarrolló un panel web interactivo basado en Flask (`config_server.py`) que expone endpoints seguros y cuenta con validación estricta de inputs, file-lock para la persistencia del archivo de configuración JSON y un sistema de respaldos y rollbacks automáticos.
- Se integró y configuró un Punto de Acceso WiFi AP privado y oculto administrado en caliente mediante un script de control en Linux (`wifiap`), resolviendo conflictos del servicio enmascarado en systemd y configurando un SSID estático seguro.
- Se diseñó e integró un botón de diagnóstico dinámico en la consola web para la ejecución segura en background de un binario nativo en C (`comprobar_registro`) mediante un wrapper de Python expuesto en un endpoint estructurado.
- Se realizó el ensamblado físico y la configuración básica de hardware para una nueva estación de 3 componentes destinada a la expansión de la red de monitoreo.

---

## Implementación de una solución de gestión y preservación de datos para el manejo del almacenamiento en las estaciones de monitoreo.

- Se desarrolló e implementó el sistema de streaming continuo en tiempo real y almacenamiento en búfer circular (`RingBufferStore`) en el acelerógrafo para la persistencia de tramas binarias crudas y la extracción rápida de segmentos en formato miniSEED.
- Se implementó el daemon del procesador de flujo (`stream_processor.py`) resolviendo bloqueos de lectura mediante descriptors no bloqueantes, control asíncrono de señales en hilos secundarios y corrección de permisos del named pipe en C para permitir la comunicación segura entre procesos.
- Se integró el extractor de eventos bajo MQTT con un esquema de fallback automático hacia el almacenamiento horario tradicional de archivos `.mseed`.
- Se corrigió el fallo de crecimiento indefinido de archivos del búfer circular derivado del desfase temporal del hardware dsPIC, aplicando validación de días generalizada, nombres de archivos indexados anti-colisión y optimización de cuota de disco reducida a un máximo de 210 MB.

---

# Conclusiones del mes

- Se logró consolidar el sistema de adquisición y preservación de datos de los acelerógrafos mediante el diseño y despliegue del almacenamiento en búfer circular y el daemon de streaming en tiempo real, resolviendo problemas de sincronización de reloj y crecimiento indefinido de archivos.
- Se robusteció la interfaz de administración y el diagnóstico de los dispositivos acelerográficos en campo a través del panel web en Flask, el diagnóstico dinámico en background de adquisición, y un punto de acceso WiFi local seguro y oculto.
- Se optimizó y estructuró el entorno de desarrollo y ejecución de herramientas de análisis sismológico en Windows 11 Enterprise mediante la migración a un repositorio dedicado y la configuración limpia de dependencias complejas bajo Conda/Micromamba.
