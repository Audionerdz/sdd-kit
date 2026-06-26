# Spec-Driven Development Kit

**Plantilla para iniciar cualquier proyecto usando Spec-Driven Development con AI coding agents.**

Este repo contiene los elementos esenciales del curso [Spec-Driven Development de DeepLearning.AI](https://www.deeplearning.ai/courses/), limpios de código específico de AgentClinic. Úsalo como punto de partida para nuevos proyectos.

## ¿Qué incluye?

| Carpeta | Contenido |
|---------|-----------|
| `prompts/` | Todos los prompts del curso organizados por lección |
| `example_specs/` | Ejemplos de documentos de especificación (misión, tech stack) |
| `skills/` | Skills reutilizables para el agente (changelog, feature-spec) |

## Cómo usar

1. Copia esta carpeta como base para tu proyecto:

```bash
cp -r sc-spec-driven-development-files/ mi-nuevo-proyecto/
cd mi-nuevo-proyecto
```

2. Escribe tu `README.md` con el contexto de tu proyecto y stakeholder input.

3. Usa un AI coding agent (opencode, Claude Code, etc.) con los prompts de `prompts/` para crear tu constitución (`specs/mission.md`, `tech-stack.md`, `roadmap.md`).

4. Sigue el flujo SDD: Constitución → Feature Specs → Implementación → Validación.

## Requisitos

- Node.js (v18+)
- Git
- Un AI coding agent
