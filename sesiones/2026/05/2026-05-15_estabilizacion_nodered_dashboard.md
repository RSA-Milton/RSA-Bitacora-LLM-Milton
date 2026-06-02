---
fecha: 2026-05-15
temas: [node-red, docker, mqtt, dashboard]
entorno: tig
---

# Registro de Sesión: Estabilización del Dashboard Node-RED para RSA

# Hitos de la jornada:
Durante la sesión se resolvió una serie de conflictos estructurales en el despliegue del panel de control de Node-RED encargado de gestionar comandos hacia la Red Sísmica del Austro. El problema central radicaba en un conflicto de bloqueo de inodos a nivel del sistema operativo (EBUSY) provocado por montajes individuales de archivos con permisos de solo lectura (:ro) en Docker, lo que impedía a Node-RED persistir cambios en el flujo, credenciales y dependencias. Se rediseñó el archivo docker-compose.yml para montar únicamente el directorio raíz /data, delegando la gestión de archivos internamente al contenedor.

Adicionalmente, se corrigieron problemas críticos de renderizado del módulo node-red-dashboard. Se mitigó la superposición de widgets forzando el uso estricto de la herramienta gráfica Layout (fijando anchos absolutos en el grid) y se resolvió el desbordamiento visual del panel de logs migrando de un nodo ui_text (afectado por el centrado vertical forzado de Flexbox) a un nodo ui_template con un contenedor HTML personalizado que habilita el posicionamiento absoluto y el scroll dinámico. Finalmente, se refactorizó el código de los nodos Function para implementar tipado estricto en el procesamiento de variables temporales.

# Decisiones y Cambios:

- Gestión de volúmenes Docker: Eliminación de montajes granulares (settings.js, flows.json, flows_cred.json, package.json). Se consolidó un montaje único para permitir lectura/escritura nativa del motor Node.js.

- Gestión de dependencias y obsolescencia: Instalación de node-red-dashboard gestionada exclusivamente mediante la interfaz Manage Palette para asegurar su persistencia. Se tomó la decisión técnica de mantener este paquete (Dashboard Clásico), a pesar de estar obsoleto, para culminar la Prueba de Concepto (PoC) sin reescribir la UI en una nueva librería.

- Gestión de Credenciales MQTT: Abandono de la inyección de credenciales por variables de entorno dentro del archivo flows.json. Las credenciales ahora se ingresan mediante el panel de seguridad de la UI de Node-RED, permitiendo su encriptación nativa en flows_cred.json.

- Refactorización de UI Inputs: Reemplazo de ui_numeric por ui_text_input (modo number) para captura directa de duraciones. Separación del selector temporal en ui_date_picker (startDate) y ui_text_input modo time (startTime).

- Defensa contra tipos dinámicos: Implementación de lógica defensiva en el nodo "Build Command" para capturar la hora (startTime), interceptando el comportamiento del dashboard que inyectaba el valor en milisegundos en lugar de texto.

# Scripts/Comandos relevantes:

```bash
# Comandos de respaldo en el servidor (Sincronización manual del volumen hacia el repositorio)
cp /home/rsa/data/nodered/flows.json montajes/server-ubuntu/RSA-Intern-TIG-MQTT/services/node-red/flows.json
```