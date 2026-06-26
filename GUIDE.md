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

### Flujo Paso a Paso para Código Existente

Aquí está el workflow concreto para aplicar SDD a un proyecto que ya tiene código funcionando:

```
1. Analizar el código actual → 2. Crear constitución retroactiva → 3. Feature spec de lo próximo → 4. Implementar
```

#### Paso 1: Análisis del Estado Actual (sin agente)

Antes de involucrar al agente, tú mismo haz un inventario rápido:

- **Stack real:** ¿qué lenguajes, frameworks, DB, APIs externas usa el proyecto?
- **Features existentes:** haz una lista de lo que ya funciona
- **Deuda técnica:** lo que sabes que está mal y necesita arreglo
- **README o docs:** si hay documentación existente, úsala como insumo

#### Paso 2: Crear la Constitución (con el agente)

Le das al agente un prompt como:

> "Analiza este proyecto existente. Lee el código, package.json, estructura de carpetas. Luego crea una constitución en `specs/` con `mission.md`, `tech-stack.md` y `roadmap.md` que describa el proyecto tal como existe hoy. Antes de escribir, pregúntame lo que necesites."

El agente debería:
1. Explorar `package.json`, `tsconfig.json`, `Dockerfile`, `composer.json`, `Cargo.toml`, etc.
2. Leer archivos fuente para entender la arquitectura
3. Revisar commits recientes para entender el ritmo de trabajo
4. Preguntarte lo que no pueda deducir del código
5. Escribir la constitución

> **Importante:** la constitución retroactiva describe el proyecto **tal cual es**, no tal como te gustaría que fuera. El `roadmap.md` sí mira hacia adelante — las features pendientes, no las ya hechas.

#### Paso 3: Feature Spec de lo Próximo

Con la constitución lista, usas el feature-spec skill normalmente. La diferencia es que el feature-spec ahora **convive con código legacy** — y está bien.

Algunas situaciones comunes:

| Situación | Cómo manejarlo |
|-----------|---------------|
| La nueva feature toca código legacy | El `plan.md` debe incluir tareas de refactor previo o adaptación |
| El stack real no es ideal para la nueva feature | Pregúntale al agente: "¿conviene migrar este módulo o agregar al lado?" |
| No sabes por dónde empezar | El feature-spec interview te guía: scope → decisiones → contexto |
| El proyecto no tiene tests | La primera fase del roadmap puede ser "Agregar tests a módulo X" |

#### Paso 4: Implementación en un Proyecto Mixto

Tres reglas para sobrevivir:

1. **No mezcles refactor con features nuevas.** Si necesitas refactorizar, que sea una fase separada en el roadmap o una tarea explícita en el plan.md.
2. **No toques código que no está en la spec.** Si el feature-spec dice "agregar endpoint GET /api/users", solo tocas eso. Mejorar el código legacy al lado es tentador pero te saca del foco.
3. **Valida contra lo que existe.** Si el código legacy no tiene tests, la validación manual (probar la feature existente + la nueva) es suficiente.

### Errores Comunes al Empezar SDD en Proyectos Legacy

| Error | Consecuencia | Solución |
|-------|-------------|----------|
| Querer refactorizar todo antes de empezar | Nunca empiezas | Haz una fase "refactor" en el roadmap si es urgente, pero no la hagas requisito |
| Poner todo el código legacy en el roadmap | Roadmap eterno | El roadmap es para lo que **falta**, no para lo que ya existe |
| Ignorar el stack real en la constitución | Los specs describen features imposibles de implementar | Sé honesto. Si el stack apesta, el roadmap mostrará las migraciones necesarias |
| No hacer preguntas al agente | El agente alucina decisiones que no le diste | Siempre responde las 3 preguntas del feature-spec antes de que escriba |

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
