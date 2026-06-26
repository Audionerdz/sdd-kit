# Spec-Driven Development — Guía de Trabajo

## ¿Qué es SDD?

Spec-Driven Development es un flujo donde **primero escribes la especificación, después el código**, usando un AI coding agent (opencode, Claude Code, etc.) que te guía con preguntas en cada paso.

## El Pipeline Completo

```
Constitución (fundación)
    ↓
Feature Specs (qué construir)
    ↓
Implementación (código)
    ↓
Validación (tests + revisión)
    ↓
Merge a main
    ↓
Siguiente feature → vuelve al paso 2
```

---

## 1. Constitución — Los Documentos Fundacionales

Son 3 archivos en `specs/` que definen el proyecto antes de escribir código:

| Archivo | Responde a |
|---------|------------|
| `specs/mission.md` | ¿Por qué existe? Propósito, stakeholders, audiencia objetivo |
| `specs/tech-stack.md` | ¿Con qué lo construimos? Lenguaje, framework, base de datos, arquitectura |
| `specs/roadmap.md` | ¿En qué orden? Fases pequeñas e implementables |

### ¿Cómo se crea?

Le das al agente el **Prompt 1**: contexto del proyecto y stakeholder input. Luego el **Prompt 2**: "crea la constitución en `specs/`".

El agente usa su herramienta de preguntas (equivalente a `AskUserQuestion`) para preguntarte sobre:

1. **Misión** — propósito, problema que resuelve, usuarios
2. **Tech Stack** — tecnologías, frameworks, base de datos
3. **Roadmap** — fases de implementación en orden

Tú respondes y el agente escribe los 3 archivos.

> **Regla importante:** el agente debe preguntar ANTES de escribir. Si escribe directo, está alucinando decisiones que deberías tomar tú.

---

## 2. Feature Spec — De la Constitución a la Especificación

Una vez que la constitución está lista, cada nueva feature sigue el **feature-spec skill** (`skills/feature-spec/`):

1. El agente lee `specs/roadmap.md` y encuentra la primera fase incompleta (`[ ]`)
2. Crea un branch: `git checkout -b phase-N-nombre-de-la-fase`
3. Te hace 3 preguntas agrupadas:
   - **Scope** — qué incluye/excluye la feature, datos, comportamiento
   - **Decisiones** — opciones de implementación, almacenamiento, UX
   - **Contexto** — tono, restricciones, preguntas abiertas
4. Lee `specs/mission.md` y `specs/tech-stack.md` para alinear la spec
5. Crea `specs/YYYY-MM-DD-nombre/` con 3 archivos:
   - `requirements.md` — alcance, decisiones, contexto
   - `plan.md` — tareas agrupadas e implementables
   - `validation.md` — cómo validar (tests, walkthrough, definition of done)

> El branch se crea **antes** de escribir cualquier spec, para que todo el trabajo de la feature quede aislado.

---

## 3. Implementación — De la Spec al Código

Con la spec lista, implementas siguiendo el `plan.md`. El agente:

1. Lee `requirements.md` y `plan.md`
2. Implementa cada grupo de tareas
3. Commitea frecuentemente en el mismo branch
4. Cuando termina, ejecuta validación

---

## 4. Validación — ¿Cumple la Spec?

Se ejecuta `validation.md`:

- **Automática** — tests, typecheck, lint
- **Manual** — walkthrough de la feature, casos borde
- Si algo falla → vuelve a implementación
- Si todo pasa → merge a main

---

## 5. Merge y Siguiente Feature

```bash
git checkout main          # te mueves a la rama destino (la que recibe los cambios)
git merge phase-N-nombre   # fusiona la feature hacia main
git branch -d phase-N-nombre
```

> **¿Por qué `git checkout main` primero?** `git merge` fusiona **hacia la rama en la que estás parado**. Si estás en `phase-N-nombre` y haces `git merge phase-N-nombre`, estarías fusionando la rama contra sí misma. Siempre párate en la rama que va a **recibir** los cambios, no en la que los tiene.

Actualiza `roadmap.md` marcando la fase como `[x]`. Repite desde el paso 2.

---

## Proyectos con Código Existente (Retroactivo)

Si el proyecto ya existe y nunca usaste SDD, no puedes ejecutar feature-spec porque no hay `mission.md` ni `tech-stack.md`. La solución:

### Crear la Constitución Retroactiva

1. El agente analiza el código existente:
   - Lee `package.json`, `tsconfig.json`, `Dockerfile`, etc.
   - Examina la estructura de carpetas, dependencias, scripts
   - Revisa el README y documentación existente
   - Mira commits recientes para entender el estado actual

2. El agente te pregunta para afinar:
   - **Misión** — ¿para qué existe realmente el proyecto? (puede diferir de lo que dice el README)
   - **Tech Stack** — ¿esto es lo que usas o lo que heredaste?
   - **Roadmap** — ¿qué features faltan? ¿qué deuda técnica hay?

3. Con la constitución lista, el feature-spec ya funciona normalmente.

### Tips para Retroactivo

- **No rewritees historia.** La constitución describe el presente, no idealiza el pasado.
- **Sé honesto en tech-stack.** Si el proyecto usa jQuery y PHP, eso pone el skill. Lo cambias cuando toque migrar.
- **Roadmap empieza desde hoy.** Las fases miran hacia adelante, no documentan features ya hechas.

---

## ¿Cuándo Crear Branches?

| Momento | Acción |
|---------|--------|
| Nueva feature | `git checkout -b phase-N-nombre` (lo hace feature-spec) |
| Hotfix en producción | Branch desde main, no desde tu feature branch |
| Experimentar | Branch aparte, sin spec. Si funciona, lo conviertes en feature spec |
| Cambiar la constitución | `git checkout -b update-constitution`. Se mergea como cualquier cambio |

**Regla:** cada feature = un branch. No trabajes varias features en el mismo branch.

---

## Preguntas Frecuentes de Este Thread

### ¿Qué es la Constitución?
Son 3 documentos que definen por qué existe el proyecto, con qué tecnologías se construye y en qué orden se implementa. Es la base de todo el flujo SDD.

### ¿Qué contiene el AskUserQuestion tool?
Es la herramienta del agente para hacerte preguntas. En la constitución pregunta sobre misión, tech stack y roadmap. En feature-spec pregunta sobre scope, decisiones y contexto. El agente **nunca debe escribir hasta que respondas**.

### ¿Para qué necesito Node.js si solo escribo Markdown?
No lo necesitas para la constitución. Node.js está porque el curso construye una app real (AgentClinic). La constitución vive en el repo junto al código, pero son solo archivos Markdown que puedes escribir en cualquier editor.

### ¿Y si quiero empezar un proyecto nuevo, digamos de ciberseguridad?
Usas el mismo flujo: scaffold mínimo (package.json, tsconfig.json, src/index.ts, README.md), creas la constitución con el agente, y sigues el pipeline SDD. El tema del proyecto (ciberseguridad, web, CLI, lo que sea) no cambia el flujo — solo cambia el contenido de los specs.

### ¿Qué pasa si el proyecto no empezó con SDD?
Vuelta a la sección "Proyectos con Código Existente (Retroactivo)" — el agente analiza tu código existente y deduce la constitución.

### ¿El feature-spec sirve para proyectos sin constitución?
No. El feature-spec necesita `mission.md` y `tech-stack.md` para alinear la spec. Sin constitución, no tiene contexto. Primero crea la constitución retroactiva.

---

## Estructura Final del Proyecto

```
mi-proyecto/
├── specs/
│   ├── mission.md              ← constitución
│   ├── tech-stack.md           ← constitución
│   ├── roadmap.md              ← constitución
│   ├── YYYY-MM-DD-feature-1/   ← feature spec
│   │   ├── requirements.md
│   │   ├── plan.md
│   │   └── validation.md
│   └── YYYY-MM-DD-feature-2/   ← siguiente feature
│       ├── requirements.md
│       ├── plan.md
│       └── validation.md
├── src/                        ← código real
├── skills/                     ← skills del agente
│   ├── changelog/
│   │   ├── SKILL.md
│   │   └── scripts/changelog.py
│   └── feature-spec/
│       └── SKILL.md
├── prompts.md                  ← los prompts que usas con el agente
├── CHANGELOG.md                ← generado por el skill
├── GUIDE.md                    ← este archivo
└── README.md                   ← stakeholder input + contexto
```
