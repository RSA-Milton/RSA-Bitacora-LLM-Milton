# Guía para Agentes de IA en RSA-Bitacora-LLM-Milton

Este repositorio contiene la **bitácora personal de sesiones de programación asistida con LLM** de Milton, miembro de la Red Sísmica del Austro (RSA).

Cada archivo en `sesiones/` es un registro técnico generado automáticamente por el skill `volcado_bitacora` al finalizar una sesión de trabajo con un agente IA.

---

## 📂 Estructura

```text
RSA-Bitacora-LLM-Milton/
├── sesiones/
│   └── YYYY/
│       └── MM/
│           └── YYYY-MM-DD_tema_de_la_sesion.md
├── AGENTS.md    # Este archivo
└── README.md
```

---

## ⚙️ Flujo de Trabajo

Este repo **no se edita manualmente**. El agente IA escribe en él usando el skill `volcado_bitacora`.

Para que el agente tenga acceso:
1. El repo debe estar clonado en `git/institucional/RSA-Bitacora-LLM-Milton/`.
2. El workspace del IDE debe ser `git/` (el directorio padre).
3. El catálogo de contribuidores en `RSA-Metodologias/indice/catalogo_contribuidores.md` debe tener la entrada de `@Milton`.

---

## ⚙️ Habilidades y Flujos de Trabajo (Skills)

Los skills específicos de este repositorio están en `.agents/skills/`. Se activan con comandos específicos del usuario:

| Skill | Activación | Descripción |
|-------|-----------|-------------|
| `generar_informe` | "genera el informe de [mes] de [año]" | Genera el informe mensual consolidando las sesiones y las clasifica en los objetos contractuales |

---

## 🔗 Repositorios Relacionados

- **Toolkit**: [RSA-Agent-Toolkit](https://github.com/RedSismicaAustro/RSA-Agent-Toolkit) — Skills y reglas del agente.
- **Metodologías**: [RSA-Metodologias](https://github.com/RedSismicaAustro/RSA-Metodologias) — Índice temático y ADRs.

