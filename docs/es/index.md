# Harness engineering

Una guía para equipos técnicos que ya usan agentes de codificación (Claude Code, Codex, Copilot, Cursor, etc.) y quieren dar el siguiente paso: **dejar de tratarlos como autocomplete y empezar a montar un harness serio alrededor de ellos**.

No es un tutorial paso a paso ni un manual de un producto concreto. Es un ensayo, organizado por capítulos, sobre los principios y patrones que están emergiendo en equipos que ya operan a esta escala (OpenAI, Stripe, Thoughtworks/Böckeler, Geoffrey Huntley) y que se repiten independientemente del stack o de la herramienta.

## Premisas

- Trabajas en un equipo técnico con experiencia operando agentes y skills.
- Te interesa **diseñar el sistema**, no encontrar el prompt mágico.
- Estás dispuesto a invertir en infraestructura interna (linters, sandboxes, observabilidad, docs versionadas) si eso multiplica el rendimiento del agente.
- Te importa la sostenibilidad a meses vista, no solo el primer commit impresionante.

## Tesis central (en una frase)

> El modelo es el motor. El harness — guías, sensores, sandboxes, bucles, contexto estructurado — es lo que convierte a un LLM en un agente del que un equipo puede depender.

## Cómo leer esta guía

Los capítulos están ordenados de **mentalidad → pilares → práctica → mantenimiento → anti-patrones → diagnóstico**, pero cada uno se sostiene por sí solo. Si ya tienes claro el "por qué", puedes saltar directamente al capítulo que te resuelva un problema concreto.

## Índice

### Capítulo 0 — Puerta de entrada

- [Harness engineering: producir software a escala con agentes](00-harness-engineering.md)

### Parte 1 — Mentalidad

- [1. Por qué el harness importa más que el modelo](01-why-harness.md)
- [2. La reubicación del rigor](02-relocating-rigor.md)

### Parte 2 — Los dos pilares

- [3. Guías y Sensores: feedforward y feedback](03-guides-and-sensors.md)
- [4. El bucle: iteración como primitiva](04-the-loop.md)

### Parte 3 — Práctica

- [5. Entornos aislados y reproducibles](05-isolated-environments.md)
- [6. El contexto como sistema de registro](06-context-system-of-record.md)
- [7. Arquitectura para agentes](07-architecture-for-agents.md)
- [8. El flujo de PR agéntico](08-agentic-pr-flow.md)

### Parte 4 — Sostenibilidad

- [9. Entropía y mantenimiento continuo](09-entropy-and-maintenance.md)
- [10. Dónde sigue importando el humano](10-human-role.md)

### Parte 5 — Errores comunes

- [11. Anti-patrones](11-anti-patterns.md)

### Parte 6 — Llevarlo a tu equipo

- [12. Diagnóstico y madurez: dónde estás y qué tocar primero](12-diagnosis-and-maturity.md)

## Qué *no* encontrarás en esta versión

- Plantillas listas para copiar (`AGENTS.md` de ejemplo, hooks, configuraciones de CI). Vendrán en una versión posterior.
- Comparativas entre herramientas concretas. La guía es deliberadamente agnóstica.
- Promesas de productividad. Los números de Stripe o de OpenAI son referencias, no garantías.

## Roadmap para v2

Cosas que están planificadas para la próxima versión de la guía:

- **Plantillas ejecutables.** `AGENTS.md` mínimo de ejemplo, estructura completa de `docs/`, ejemplos de linters custom con mensajes dirigidos al agente, esqueleto de configuración de sandbox, plantillas de plan de ejecución.
- **Plantilla de hoja de ruta como archivo separado** (extraída del cap. 12) para que los equipos puedan copiarla y rellenarla en una sesión sin tocar el resto de la guía.
- **Glosario corto** de términos (harness, guía, sensor, bucle, sandbox, blueprint, doc-gardening, golden principles, flywheel) para equipos que comparten la guía con stakeholders fuera del círculo técnico inmediato.
- **Capítulo 0 como post de blog publicable**, con su propio frontmatter, optimizado para distribución externa.
- **Estudios de caso comparados** — un análisis lado a lado de cómo OpenAI, Stripe y otros equipos públicos resuelven cada una de las cinco dimensiones del cap. 12, con tabla de equivalencias.
- **Checklist diagnóstico imprimible** (1 página) extraído del cap. 12 para pegar en la pared del equipo.
- **Sección sobre seguridad y aislamiento adversarial** — qué cambia en el harness cuando el agente puede ser objetivo de prompt injection vía contexto, cómo razonar sobre la cadena de suministro de herramientas, MCP servers, etc.

## Fuentes

Esta guía sintetiza ideas de:

- Birgitta Böckeler (Thoughtworks, publicado en martinfowler.com) — [*Harness Engineering*](https://martinfowler.com/articles/harness-engineering.html). El término "harness" en este sentido emergió en torno a LangChain con la fórmula *"Agent = Model + Harness"*.
- Kief Morris (en martinfowler.com) — [*Humans and Agents in Software Engineering Cycles*](https://martinfowler.com/articles/exploring-gen-ai/humans-and-agents.html)
- Chad Fowler — [*Relocating Rigor — The Phoenix Architecture*](https://aicoding.leaflet.pub/3mbrvhyye4k2e)
- Geoffrey Huntley — [*everything is a ralph loop*](https://ghuntley.com/loop/)
- Ryan Lopopolo (OpenAI) — [*Harness Engineering: leveraging Codex in an agent-first world*](https://openai.com/es-ES/index/harness-engineering/)
- Alistair Gray (Stripe, equipo Leverage) — *Minions: Stripe's one-shot, end-to-end coding agents* — [parte 1](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents) · [parte 2](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2)
