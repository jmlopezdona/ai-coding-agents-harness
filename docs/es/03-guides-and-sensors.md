# 3. Guías y Sensores: feedforward y feedback

Birgitta Böckeler divide el harness en dos mecanismos. Una distinción simple pero útil, porque te obliga a clasificar cada inversión que haces: ¿esto evita un error antes de que ocurra (guía), o lo detecta después (sensor)? Si no es ninguna de las dos cosas, probablemente no pertenece al harness.

## Guías — feedforward

Una guía actúa **antes** de que el agente actúe. Su trabajo es orientar la acción hacia la decisión correcta dándole al agente, por adelantado, la información que necesita: la forma de los datos, la estructura del módulo, el patrón a seguir, dónde mirar. Una buena guía no es una prohibición; es una indicación que el agente puede leer e imitar.

Ejemplos típicos:

- **AGENTS.md / system prompts versionados.** No como lugar donde meter "todo lo importante", sino como índice corto que apunta a los sitios donde vive cada cosa (ver cap. 6). La diferencia entre un AGENTS.md de 100 líneas y uno de 1.000 no es de escala: es de filosofía.
- **Convenciones embebidas en el scaffolding.** Plantillas de servicio, generadores de módulos, bootstraps de componentes. Si cada nuevo módulo lo crea un comando, el agente hereda la estructura sin tener que adivinarla.
- **Schemas y tipos en el borde.** Validar entradas con Zod, Pydantic, JSON Schema o lo que toque. Esto no es solo defensa runtime: es un contrato legible que el agente puede leer e imitar. OpenAI menciona explícitamente que exigen parsear las formas de datos en el borde sin ser prescriptivos sobre la herramienta.
- **Reglas de arquitectura mecanizadas.** Direcciones de dependencia entre capas, módulos prohibidos, imports no permitidos. Validadas estructuralmente, no en una revisión. (Nota: la *regla* es la guía; el linter que la valida es un sensor — ver más abajo.)
- **Plantillas de PR y de plan.** Si todos los planes de ejecución tienen la misma forma, el agente sabe cómo escribirlos y cómo leer los antiguos.

El test de una buena guía es este: **¿podría un agente nuevo, sin contexto previo, hacer lo correcto solo siguiéndola?** Si la respuesta es "depende del juicio", la guía está incompleta.

### El error más común con las guías

Confundir guía con documentación. Una guía es ejecutable o casi ejecutable. La documentación es leída opcionalmente. Si tu "guía" es un documento que el agente *podría* consultar pero no está obligado a obedecer mecánicamente, no es una guía: es una sugerencia. Y las sugerencias se ignoran cuando el contexto se llena.

## Sensores — feedback

Un sensor actúa **después**. Su trabajo es detectar cuándo algo está mal con suficiente precisión y velocidad para que el propio agente (o el arnés) pueda corregirlo sin intervención humana.

Böckeler distingue dos tipos de ejecución:

- **Computacional** — determinístico, rápido, barato. Tests, type checkers, linters, validación de schemas, builds, smoke tests.
- **Inferencial** — no determinístico, más lento, capaz de evaluación semántica. Evals con LLM, revisión por otro agente, comparación de salidas contra una rúbrica.

Los dos son necesarios. El error es usar el segundo donde basta con el primero (caro, ruidoso) o intentar usar el primero donde necesitas el segundo (imposible). La mayoría de equipos pecan de no aprovechar bien los computacionales.

Ejemplos prácticos:

- **Tests rápidos y reproducibles.** No "tests de integración que tardan 8 minutos y a veces fallan por flaky". Tests que el agente puede correr sin pensárselo y de los que se fía.
- **Type checks como sensor primario.** Un type checker estricto es uno de los sensores más baratos y de los que más información aportan. Si tu stack lo permite, súbele las restricciones.
- **Linters custom con mensajes dirigidos al agente.** Esto es subestimado. Un lint custom no solo señala el patrón malo: su mensaje de error puede contener la instrucción de remedio. El agente lee el error, entiende el patrón correcto, lo aplica en el siguiente intento. Un buen linter custom es un tutor pasivo: actúa después del fallo (por eso es sensor), pero su señal es lo bastante didáctica como para que el agente converja en una o dos iteraciones.
- **Observabilidad efímera en local.** Esto es lo que OpenAI hizo con la pila Vector + Victoria + LogQL/PromQL/TraceQL por worktree. Permite que el agente formule preguntas como "¿el arranque tarda menos de 800ms?" y obtenga una respuesta verificable. La observabilidad deja de ser un lujo de prod y se vuelve parte del bucle de desarrollo.
- **Smoke tests del UI vía DevTools.** OpenAI conecta Chrome DevTools Protocol al runtime del agente: el agente puede sacar screenshots, navegar el DOM, reproducir fallos visuales. La UI deja de ser una caja negra para el agente.
- **Revisiones automáticas de otro agente.** Un agente revisa el PR de otro contra una rúbrica. No reemplaza al humano, pero filtra el 80% del ruido antes de que llegue a un humano.
- **Evals con conjuntos congelados.** Para tareas donde no hay oráculo determinístico (ej. calidad de un mensaje generado), un eval con casos congelados.

### El criterio para añadir un sensor

Un sensor solo es útil si su señal entra de vuelta al bucle. Un test que falla y nadie corrige no es un sensor: es ruido. Un dashboard que un humano mira una vez al mes no es un sensor para el agente: es un memorial.

La pregunta correcta antes de añadir un sensor es: **¿cuándo este sensor se dispare, qué pasa? ¿Quién (o qué) lee la señal y qué hace con ella?** Si no hay una respuesta concreta, no añadas el sensor todavía; primero diseña el bucle.

## La interacción entre guías y sensores

Los dos pilares no son independientes. Cuando un sensor detecta un patrón malo repetidamente, eso es información: probablemente falta una guía. Y al revés, cuando una guía bloquea algo legítimo, probablemente está mal calibrada y necesita relajarse o moverse a un sensor.

El flujo sano es algo así:

1. Un sensor (test, lint, eval) detecta un fallo recurrente.
2. El equipo identifica el patrón: el agente sigue intentando hacer X cuando lo correcto es Y.
3. Se promueve la regla: se añade una guía (un lint, una convención en AGENTS.md, una plantilla) que hace que X sea imposible o trivialmente detectable.
4. El sensor sigue ahí como red de seguridad, pero ya casi nunca se dispara para ese caso concreto.

Esto es exactamente lo que OpenAI describe como *"cuando la documentación no basta, promovemos la regla al código"*. No hay ningún truco: es ingeniería de plataforma aplicada a un colaborador no humano.

## Una advertencia sobre el equilibrio

Es posible sobre-restringir. Un arnés con demasiadas guías estrictas y demasiados sensores hipersensibles paraliza al agente: cualquier acción razonable dispara cinco alarmas y el bucle se atasca. La regla útil es la misma que aplica a un equipo humano: **límites estrictos donde las consecuencias son irreversibles, autonomía donde son baratas de revertir**.

Los próximos capítulos aplican estos dos pilares a problemas concretos: el bucle (cap. 4), los entornos aislados (cap. 5), el contexto (cap. 6), la arquitectura (cap. 7) y el flujo de PR (cap. 8). En cada uno verás guías y sensores trabajando juntos.
