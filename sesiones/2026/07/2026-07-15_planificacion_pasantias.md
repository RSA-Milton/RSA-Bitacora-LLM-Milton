---
fecha: 2026-07-15
temas: [pasantias, kicad, eagle, bom, shm]
entorno: [pasantias]
autor: Milton
---

# Actividad del 2026-07-15

**Hitos de la jornada:**
Se estructuraron y actualizaron las planificaciones de tareas de pasantías para el área de hardware y diseño de circuitos impresos de la RSA. En primer lugar, se elaboró un plan de trabajo de 96 horas para la migración nativa de proyectos en EAGLE hacia KiCad 10, enfocado en el PCB del sistema de adquisición de datos acelerométricos con ESP32. El plan incluye auditoría estricta de asignación de pines (*Pinout Audit*) contra el firmware original del prototipo en protoboard, estandarización de componentes SMD para PCBA en JLCPCB, librerías locales relativas con `jlc2kicadlib`, optimización de planos de desacoplo/masa y generación de archivos de producción con *Fabrication Toolkit*.

En segundo lugar, se reestructuró la planificación para el ensamblaje y validación de la red de monitorización de salud estructural (SHM), ajustando la duración a 96 horas reales. Se desglosó la manufactura de 3 placas desarrolladas previamente (1 Nodo Concentrador y 2 Nodos Sensores con dsPIC33EP256MC202), integrando el segundo módulo MAX485 para sincronización en el Concentrador, la interconexión Daisy Chain en norma T-568B, el firmware en MikroC y la prueba comparativa de ruido entre la versión con plano de masa y sin plano de masa.

Finalmente, se generó la Lista de Materiales (BOM) técnica completa para la adquisición de componentes (incorporando explícitamente bases/sockets DIP-28 y DIP-8 para protección de los circuitos integrados) y se automatizó la conversión de dicha BOM a un documento PDF mediante Microsoft Edge en modo headless.

**Decisiones y Cambios:**
- Anonimización de la documentación de pasantías reemplazando nombres de estudiantes por la denominación formal de los proyectos.
- Ajuste del tiempo de ensamblaje manual a 16 horas para 3 placas (1 Concentrador + 2 Nodos Sensores).
- Inclusión obligatoria de zócalos/sockets DIP en la BOM para proteger los dsPIC33EP256MC202 y transceptores MAX485 en soldadura manual.
- Automatización de generación PDF de documentos Markdown usando Microsoft Edge en modo headless (`--headless --print-to-pdf`).

**Scripts/Comandos relevantes:**
```bash
# Exportación automatizada de HTML/Markdown a PDF mediante Microsoft Edge Headless
cmd /c '"C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe" --headless --print-to-pdf="c:\Users\miltonrsa\Documents\Proyectos-RSA\Pasantias\planificacion-pasantias\Lista_de_Materiales_Sistema_Monitorizacion.pdf" "c:\Users\miltonrsa\Documents\Proyectos-RSA\Pasantias\planificacion-pasantias\temp_bom.html"'
```
---
