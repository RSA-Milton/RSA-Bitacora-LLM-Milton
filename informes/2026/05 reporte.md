# Reporte de evidencias - Mayo 2026

---

# Introducción

El presente documento corresponde al detalle de las actividades desarrolladas durante el mes de mayo de 2026 en el marco de las tareas de la "Red Sísmica del Austro (RSA)". Específicamente se aportan los medios de verificación de las tareas realizadas y su correspondencia con el objeto del contrato.

---

# Avances en los objetos del contrato

Durante el mes de mayo de 2026 se trabajó en los siguientes objetos del contrato:

🔲 Ensamblado, configuración e instalación de estaciones digitales con 6 componentes para la actualización y ampliación de la red de monitoreo sísmico, integrando arquitecturas IoT para la transmisión distribuida de datos. (objeto 1)

✅ Implementación de funcionalidades de autodetección de eventos mediante técnicas basadas en machine learning y acceso a datos en tiempo real.  (objeto 2)

✅ Ensamblado, configuración e instalación de acelerógrafos de 3 componentes con integración a la red de digitalizadores de 6 componentes.  (objeto 3)

✅ Configuración, instalación y mantenimiento de los sistemas desarrollados para el monitoreo automatizado de variables estáticas en la presas propiedad de Elecaustro.  (objeto 4)

🔲 Administración de la red de sensores distribuidos en la presa, garantizando la recolección y transmisión precisa de datos en tiempo real.  (objeto 5)

🔲 Implementación de una solución de gestión y preservación de datos para el manejo del almacenamiento en las estaciones de monitoreo. (objeto 6)

✅ Implementación de un sistema de visualización  y monitoreo de métricas operativas basado en protocolos de telemetría ligera.  (objeto 7)

---

## Implementación de funcionalidades de autodetección de eventos mediante técnicas basadas en machine learning y acceso a datos en tiempo real.

- Se diseñó un mecanismo asíncrono para la extracción remota de registros acelerográficos, el cual emplea orquestadores y subprocesos independientes para evitar conflictos entre los entornos virtuales de procesamiento y los procesos globales de telemetría.
- Se integró a la red la capacidad de procesar comandos de difusión global, permitiendo la interacción simultánea con múltiples dispositivos mediante suscripciones duales, lo cual prepara la infraestructura para algoritmos automatizados.
- Se configuraron entornos de desarrollo resilientes en estaciones locales corporativas con políticas de seguridad restrictivas para dar soporte a la ejecución de los scripts analíticos.

---

## Ensamblado, configuración e instalación de acelerógrafos de 3 componentes con integración a la red de digitalizadores de 6 componentes.

- Se completó la fase de ensamblaje electrónico para una nueva estación acelerográfica destinada a operar en Tenguel.
- Se realizaron los trabajos correspondientes de configuración de red, asegurando la conectividad del dispositivo.
- Se validó su operatividad a través de conexiones seguras y el monitoreo remoto de sus flujos de datos.

---

## Configuración, instalación y mantenimiento de los sistemas desarrollados para el monitoreo automatizado de variables estáticas en la presas propiedad de Elecaustro

- Se realizó una prospección geoeléctrica en las inmediaciones de la infraestructura de la Presa de Labrado.

---

## Implementación de un sistema de visualización  y monitoreo de métricas operativas basado en protocolos de telemetría ligera.

- Se consolidó la interfaz gráfica operativa al reestructurar la gestión de volúmenes virtuales, lo que solucionó inconvenientes de almacenamiento persistente, gestión de credenciales y solapamiento visual de componentes.
- Se subsanó un error en los paneles de estado de red al refactorizar las consultas de datos para ignorar identificadores efímeros que generaban métricas duplicadas.
- Se desplegó exitosamente una solución de monitoreo tipo quiosco en un servidor de misión crítica, sorteando bucles de sesión y garantizando la visualización continua sin interferir con los servicios sísmicos locales.

---

# Conclusiones del mes

- Se logró implementar la telemetría de control asíncrono y los comandos globales en las estaciones remotas, estableciendo los requerimientos técnicos necesarios para la posterior autodetección de eventos sísmicos.
- Se consolidó la fiabilidad del entorno de visualización y control operativo, resolviendo inconsistencias históricas de datos y garantizando un despliegue de monitoreo seguro en infraestructura crítica.