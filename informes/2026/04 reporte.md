# Reporte de evidencias - Abril 2026

---

# Introducción

El presente reporte detalla las actividades ejecutadas durante el mes de Abril de 2026 en el marco de las tareas de la "Red Sísmica del Austro (RSA)". Específicamente se aportan los medios de verificación de las tareas realizadas y su correspondencia con el objeto del contrato

---

# Avances en los objetos del contrato

Durante el mes de abril de 2026 se trabajó en los siguientes objetos del contrato:

🔲 Ensamblado, configuración e instalación de estaciones digitales con 6 componentes para la actualización y ampliación de la red de monitoreo sísmico, integrando arquitecturas IoT para la transmisión distribuida de datos.

🔲 Implementación de funcionalidades de autodetección de eventos mediante técnicas basadas en machine learning y acceso a datos en tiempo real.

✅ Ensamblado, configuración e instalación de acelerógrafos de 3 componentes con integración a la red de digitalizadores de 6 componentes.

✅Configuración, instalación y mantenimiento de los sistemas desarrollados para el monitoreo automatizado de variables estáticas en la presas propiedad de Elecaustro 

✅ Administración de la red de sensores distribuidos en la presa, garantizando la recolección y transmisión precisa de datos en tiempo real.

🔲 Implementación de una solución de gestión y preservación de datos para el manejo del almacenamiento en las estaciones de monitoreo.

🔲 Implementación de un sistema de visualización  y monitoreo de métricas operativas basado en protocolos de telemetría ligera. 

---

## Ensamblado, configuración e instalación de acelerógrafos de 3 componentes con integración a la red de digitalizadores de 6 componentes

- Se corrigió un error crítico en el script de aprovisionamiento automatizado, el cual obstaculizaba la configuración y puesta a punto de nuevas estaciones.
- Se refactorizó la lógica interna de despliegue para los módulos del sistema, implementando transferencias dinámicas por lotes en sustitución de los enrutamientos estáticos hacia archivos obsoletos.

---

## Configuración, instalación y mantenimiento de los sistemas desarrollados para el monitoreo automatizado de variables estáticas en las presas propiedad de Elecaustro

- Se evaluó arquitectónicamente el histórico del sensor ultrasónico, determinándose la necesidad de migrar la carga de procesamiento matemático hacia un esquema local de Edge DSP. Para este propósito, se seleccionó el microcontrolador ESP32-S3 debido a su unidad de coma flotante (FPU), habilitando así la ejecución precisa de algoritmos de correlación cruzada en detrimento de la antigua medición analógica de tiempo de vuelo.
- El dia de 28 de abril de 2026 se realizo una prospección geoeléctrica en la Presa de Labrado.

---

## Administración de la red de sensores distribuidos en la presa, garantizando la recolección y transmisión precisa de datos en tiempo real

- Se estabilizó el módulo de coordinación MQTT tras presentarse incidentes de reportes falsos de conectividad derivados de fluctuaciones en la red local y conflictos de enrutamiento de interfaces.
- Se incorporó a nivel de software una estrategia de resiliencia frente a conexiones inaccesibles, gestionando las excepciones mediante ciclos de reintentos con retroceso exponencial (*exponential backoff*)  y delegando las verificaciones de mensajes a la librería nativa para garantizar la integridad de los registros telemétricos.
- Se rediseñó el acceso remoto a las estaciones mediante la migración a una red VPN basada en Tailscale, estableciendo túneles de baja latencia capaces de evadir las inspecciones de firewall institucionales mientras se optimizó el consumo de memoria RAM y se forzó la prioridad de salida por la interfaz ethernet.

---

# Conclusiones del mes

- Se logró estabilizar los sistemas de telemetría implementando estrategias de resiliencia de red avanzadas que evitan la corrupción de datos y los falsos reportes operativos.
- Se modernizó la infraestructura de acceso remoto seguro a las estaciones a través de túneles VPN optimizados, superando las limitaciones impuestas por firewalls perimetrales sin comprometer los recursos de hardware.
- Se estableció una ruta clara de actualización para los sensores de nivel mediante el diseño de una arquitectura Edge DSP orientada a mejorar la exactitud de los cálculos ultrasónicos.