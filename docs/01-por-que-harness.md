# 1. Por qué el harness importa más que el modelo

La conversación pública sobre agentes de codificación gira casi siempre alrededor del modelo: qué versión, qué benchmark, qué context window. Es la pregunta equivocada para un equipo que ya usa estas herramientas a diario. La pregunta útil es otra: **¿qué hay alrededor del modelo?**

A ese "alrededor" la comunidad lo llama el *harness*: el conjunto de prompts, herramientas, reglas, sandboxes, validadores y bucles de feedback que rodean al LLM cuando actúa como agente. El término emergió en torno a LangChain (*"Agent = Model + Harness"*) y Birgitta Böckeler lo articuló con más detalle en *Harness Engineering* (Thoughtworks, publicado en martinfowler.com). El modelo es intercambiable. El harness, no.

## La analogía del motor

Un motor de Fórmula 1 dentro de un utilitario no gana carreras. Lo que gana carreras es el chasis, la aerodinámica, los neumáticos, la telemetría y el equipo de boxes. El modelo es el motor; todo lo demás es donde se gana o se pierde la carrera. Y lo importante: ese "todo lo demás" lo construyes tú, no el proveedor del modelo.

Cuando un equipo se queja de que "el agente alucina" o "no entiende el código", el problema casi nunca es el modelo. El problema es que el agente está operando en un entorno en el que sería irrazonable esperar que un humano nuevo tampoco entendiera nada. No tiene mapa, no tiene tests rápidos, no puede reproducir el fallo, no sabe cuál es la convención del equipo y nadie le dice cuándo lo que hizo está mal.

## La definición operativa

El harness es todo aquello que aumenta la probabilidad de que el agente haga lo correcto a la primera, y que cuando no lo haga, lo descubra y lo corrija sin intervención humana. Böckeler lo descompone en dos mecanismos:

- **Guías (feedforward)** — indicaciones que orientan al agente *antes* de que actúe: convenciones, plantillas, prompts del sistema, AGENTS.md, schemas.
- **Sensores (feedback)** — guardarraíles que actúan *después*: tests, type checkers, linters, observabilidad, revisores automáticos, evals.

Un harness sin guías produce un agente que se desvía. Un harness sin sensores produce un agente que se desvía y nunca se entera. Un harness con ambos produce algo que empieza a parecerse a un colaborador.

## La evidencia empírica

Tres datos que conviene tener delante cuando alguien dice "los agentes no están listos":

- **Stripe** mergea más de **1.000 PRs por semana** generados por agentes en una base de cientos de millones de líneas de Ruby/Sorbet, con stack interno poco común. Lo logran no porque tengan un modelo mejor que nadie, sino porque construyeron Toolshed (~500 herramientas internas vía MCP), devboxes preinstanciados en 10 segundos, y un sistema de "blueprints" que combina nodos determinísticos con nodos agénticos.
- **OpenAI** construyó un producto interno con **~1M de líneas y ~1.500 PRs en 5 meses**, con un equipo que creció de 3 a 7 ingenieros, y con la regla absoluta de que ninguna línea fuera escrita a mano. La clave no fue Codex. La clave fue el harness alrededor de Codex: app booteable por worktree, Chrome DevTools Protocol conectado al runtime, observabilidad efímera consultable con LogQL/PromQL, AGENTS.md de 100 líneas como índice y `docs/` estructurado como sistema de registro.
- **Geoffrey Huntley** describe el "Ralph loop" — un orquestador monolítico que ejecuta una tarea por iteración — como el primitivo del que parte cualquier agente serio. No es una característica del modelo; es una decisión arquitectónica del equipo que lo opera.

Ninguno de estos equipos ganó porque tuviera acceso a un modelo secreto. Ganaron porque trataron al agente como un sistema que hay que ingeniar.

## Por qué esto cambia el trabajo del equipo

Cuando aceptas que el harness es donde se juega la calidad, dejas de hacer ciertas cosas y empiezas a hacer otras. Dejas de buscar el "prompt perfecto" — los prompts viven en el harness ahora, versionados como código. Dejas de copiar y pegar errores al chat — los errores entran al bucle por sensores automáticos. Dejas de pedirle al agente que adivine el contexto — el contexto vive estructurado en el repo, accesible mecánicamente.

Y empiezas a hacer cosas que antes parecían over-engineering: linters custom para una sola convención, planes de ejecución versionados, observabilidad en local, agentes de "doc-gardening" que mantienen la documentación al día.

Lo que en un equipo humano sería quisquilloso, en un equipo con agentes **es un multiplicador**. Una regla codificada una vez se aplica para siempre, en cada PR, sin desgaste, sin discusión, sin necesidad de recordársela a nadie en la siguiente reunión diaria.

## El principio que recorre toda la guía

> Cuando algo falla, la pregunta no es "¿cómo le digo al agente que esto está mal?". La pregunta es "**¿qué le falta al harness para que esto no pueda ocurrir, o para que se detecte automáticamente cuando ocurra?**".

Es la misma pregunta que se hace una buena plataforma de ingeniería interna sobre sus equipos de producto. No "cómo enseño a este equipo a no romper esto", sino "cómo hago imposible (o trivialmente detectable) que rompan esto". El cambio mental es el mismo. La diferencia es que con agentes el ROI es inmediato y el bucle de mejora es de horas, no de trimestres.
