# 1. Por qué el harness importa más que el modelo

La conversación pública sobre agentes de codificación gira casi siempre alrededor del modelo: qué versión, qué benchmark, qué ventana de contexto. Es la pregunta equivocada para un equipo que ya usa estas herramientas a diario. La pregunta útil es otra: **¿qué hay alrededor del modelo?**

A ese "alrededor" la comunidad lo llama el *harness*: el conjunto de prompts, herramientas, reglas, sandboxes, validadores y bucles de feedback que rodean al LLM cuando actúa como agente. El término emergió en torno a LangChain (*"Agent = Model + Harness"*) y Birgitta Böckeler lo articuló con más detalle en *Harness Engineering* (Thoughtworks, publicado en martinfowler.com). El modelo es intercambiable. El harness, no.

Y conviene precisar desde el inicio: cuando hablamos de "harness" hay dos capas. La que trae tu AI tool de serie — Claude Code, Copilot, Cursor y similares **ya son un arnés** sobre el LLM — y la que construye tu equipo encima para adaptarlo a tu repo, tu dominio y tus convenciones. Lo primero lo comparte todo el mundo, y cada versión nueva de estas herramientas mueve más capacidades del segundo grupo al primero. Lo segundo es lo que te diferencia, y es de lo que trata esta guía.

## La analogía del motor

Un motor de Fórmula 1 dentro de un utilitario no gana carreras. Lo que gana carreras es el chasis, la aerodinámica, los neumáticos, la telemetría y el equipo de boxes. El modelo es el motor; todo lo demás es donde se gana o se pierde la carrera. Y lo importante: ese "todo lo demás" lo construyes tú, no el proveedor del modelo.

Cuando un equipo se queja de que "el agente alucina" o "no entiende el código", el problema casi nunca es el modelo. El problema es que el agente está operando en un entorno en el que sería irrazonable esperar que un humano nuevo tampoco entendiera nada. No tiene mapa, no tiene tests rápidos, no puede reproducir el fallo, no sabe cuál es la convención del equipo y nadie le dice cuándo lo que hizo está mal.

## Lo que hay dentro del harness (recordatorio)

El capítulo introductorio descompone el arnés en dos mecanismos que conviene tener delante: **guías** (feedforward — convenciones, plantillas, AGENTS.md, schemas) que orientan al agente antes de actuar, y **sensores** (feedback — tests, type checkers, linters, observabilidad, evals) que lo corrigen después. Damos esa distinción por sabida; la usaremos sin redefinirla el resto de la guía.

## Qué construyeron, exactamente

Las cifras de referencia — Stripe con >1.000 PRs/semana, OpenAI con ~1M de líneas y ~1.500 PRs en cinco meses — ya están en el capítulo introductorio. Lo que conviene añadir aquí es qué *específicamente* construyeron, porque ahí está la forma del trabajo que proponemos:

- **Stripe** no ganó con un modelo mejor. Ganó con **Toolshed** (~500 herramientas internas expuestas al agente vía MCP), **devboxes pre-instanciados en 10 segundos**, y un sistema de **blueprints** que combina nodos determinísticos con nodos agénticos según el tipo de trabajo. Las tres piezas son arnés, no modelo.
- **OpenAI** lo hizo con **app booteable por worktree** (un agente por feature, ejecutándose en paralelo, sin colisiones), **Chrome DevTools Protocol conectado al runtime** (el agente ve la UI como la ve el usuario), **observabilidad efímera consultable con LogQL/PromQL**, y `docs/` estructurado como sistema de registro (más en el cap. 6).
- **Geoffrey Huntley** aísla la primitiva que reaparece en casi todos estos equipos: un orquestador monolítico ejecutando una iteración a la vez. No es un feature del modelo; es una decisión arquitectónica del equipo que lo opera (cap. 4).

Nada de esto requiere un modelo secreto. Todo lo decide tu equipo. Todo se acumula.

## Por qué esto cambia el trabajo del equipo

Cuando aceptas que el arnés es donde se juega la calidad, dejas de hacer ciertas cosas y empiezas a hacer otras. Dejas de buscar el "prompt perfecto" — los prompts viven en el arnés ahora, versionados como código. Dejas de copiar y pegar errores al chat — los errores entran al bucle por sensores automáticos. Dejas de pedirle al agente que adivine el contexto — el contexto vive estructurado en el repo, accesible mecánicamente.

Y empiezas a hacer cosas que antes parecían over-engineering: linters custom para una sola convención, planes de ejecución versionados, observabilidad en local, agentes de "doc-gardening" que mantienen la documentación al día.

Lo que en un equipo humano sería quisquilloso, en un equipo con agentes **es un multiplicador**. Una regla codificada una vez se aplica para siempre, en cada PR, sin desgaste, sin discusión, sin necesidad de recordársela a nadie en la siguiente reunión diaria.

## El principio que recorre toda la guía

El capítulo introductorio lo enunció: cuando algo falla, la pregunta útil no es "¿cómo le digo al agente que esto está mal?" sino **"¿qué le falta al harness para que esto no pueda ocurrir, o para que se detecte automáticamente cuando ocurra?"**. Los capítulos que siguen son, en buena parte, un catálogo de qué puedes añadir al arnés cuando te haces esa pregunta — guía a guía, sensor a sensor.
