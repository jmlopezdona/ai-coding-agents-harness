# 12. Diagnóstico y madurez: dónde estás y qué tocar primero

Los once capítulos anteriores describen el qué y el porqué. Este capítulo es el dónde-estás-tú. Está pensado para que un equipo se siente media hora, responda con honestidad un puñado de preguntas, identifique en qué dimensión del harness está más cojo, y salga con una idea clara de qué inversión va primero. No es un paso-a-paso universal — esos no funcionan, porque cada equipo arranca desde un sitio distinto. Es un mapa con tu posición marcada y la flecha hacia el siguiente puerto.

Una nota importante antes de empezar: **resiste la tentación de subir todas las dimensiones a la vez**. Las dimensiones tienen dependencias entre sí; algunas desbloquean a las otras. Subir una dimensión dos niveles produce más valor que subir cinco dimensiones un nivel. La asimetría es real y casi todos los equipos la subestiman.

## Las cinco dimensiones del harness

Después de comprimir todo lo descrito en la guía, el harness se reduce a cinco dimensiones que se pueden evaluar y mejorar de forma relativamente independiente:

1. **Aislamiento** — la calidad de los entornos donde el agente ejecuta (cap. 5).
2. **Contexto** — cuánto del conocimiento relevante vive en el repo de forma legible para el agente (cap. 6).
3. **Invariantes** — cuánta de la disciplina del equipo está codificada como guides ejecutables, no como cultura implícita (cap. 3, 7).
4. **Bucle de feedback** — cuán rápido y rico es el ciclo de validación, corrección y revisión, incluido el flujo de PR (cap. 4, 8).
5. **Mantenimiento** — qué mecanismos sostienen la base de código contra el drift a meses vista (cap. 9).

Cada una se evalúa en cinco niveles, del 0 al 4. Los niveles no son notas: son **descripciones de comportamiento**. El nivel correcto para tu equipo no es necesariamente "el más alto". Para algunas dimensiones, nivel 2 es razonable y suficiente. Para otras, nivel 2 es donde empieza el dolor.

## Niveles, en una tabla

| Nivel | Aislamiento | Contexto | Invariantes | Bucle / PR | Mantenimiento |
|---|---|---|---|---|---|
| **0** | Agente corre en máquina del dev sobre el checkout principal | Conocimiento en Slack, Notion, cabezas | Disciplina cultural, sin lints custom | Bucle manual: copiar/pegar al chat | Limpieza humana ad hoc |
| **1** | Contenedor reusado entre tareas | README + AGENTS.md monolítico | Linters genéricos del lenguaje | Agente ejecuta y reporta, humano valida cada paso | Viernes de cleanup |
| **2** | Sandboxes dedicados por tarea, creados manualmente | AGENTS.md como índice + carpeta `docs/` parcial | Algunos lints custom, mensajes para humanos | Bucle interno automatizado, PR revisado humano-a-humano | Agentes recurrentes para casos puntuales |
| **3** | Sandboxes desechables, creación en segundos, paralelizables | `docs/` estructurado, planes versionados, doc-gardening básico | Lints custom con mensajes dirigidos al agente, arquitectura mecanizada | Revisiones agente-a-agente para PRs rutinarios, merge gates ligeros | Golden principles + agentes recurrentes que abren PRs de refactor |
| **4** | Aislamiento + observabilidad efímera por entorno (logs/metrics/traces consultables por el agente) | Repo como sistema de registro completo, agent-readable end-to-end | Invariantes ejecutables que cubren toda la arquitectura, escritos por el propio agente | Flujo de PR mayoritariamente agente-a-agente, humano por excepción | Agentic flywheel: agentes proponen mejoras al propio harness |

Lee cada columna como una escalera. La mayoría de equipos que han adoptado agentes están en el rango **0-2** en casi todo. Los equipos que aparecen en los artículos de referencia (OpenAI, Stripe) están en el rango **3-4**, y no en todas las dimensiones por igual.

## Autodiagnóstico: las preguntas

Para cada dimensión, responde con honestidad. La trampa es leer las preguntas y responder lo que te gustaría que fuera verdad. La señal útil aparece cuando respondes lo que es verdad.

### Aislamiento

1. ¿Puedes lanzar 5 agentes en paralelo, cada uno en su propio entorno, sin que se pisen?
2. ¿Cuánto tarda en estar listo un entorno nuevo, desde que lo pides hasta que el agente puede ejecutar dentro? (segundos / minutos / horas)
3. Cuando un entorno se destruye, ¿queda algún recurso huérfano? ¿Algún paso manual de limpieza?
4. ¿Puede el agente ejecutar tests, levantar la app, consultar logs *dentro* del entorno aislado?
5. ¿Dos creaciones del mismo entorno producen un estado idéntico?

**Nivel = el más bajo donde fallas alguna pregunta**. Si fallas la 1, estás en 0-1. Si solo fallas la 5, estás en 3.

### Contexto

1. Si un nuevo ingeniero se incorporara mañana sin acceso a Slack, Google Docs ni reuniones, ¿podría ser productivo solo con el repo?
2. Cuando una decisión arquitectónica se toma, ¿dónde queda registrada? ¿Es legible mecánicamente?
3. Tu AGENTS.md, ¿tiene más de 200 líneas?
4. ¿Existe un tracker explícito de deuda técnica que el agente pueda leer?
5. ¿Hay algún proceso (lint, agente recurrente) que detecte cuando un doc divergió del código?

### Invariantes

1. ¿Cuántos lints custom tienes específicos a tu proyecto? (0 / 1-5 / 5-20 / 20+)
2. La regla "esta capa solo puede importar de estas otras", ¿está validada por código o por convención humana?
3. Cuando el agente comete un error de patrón, ¿cuántas veces lo arreglas antes de promoverlo a lint?
4. Los mensajes de error de tus linters custom, ¿están escritos para que un agente los entienda y aplique el remedio?
5. Las invariantes de estilo (logging estructurado, naming, tamaño de archivos), ¿están ejecutadas o documentadas?

### Bucle / Flujo de PR

1. ¿El agente puede ejecutar tests, ver su salida e iterar sin que un humano intervenga?
2. ¿Cuánto tarda el bucle interno típico (edición → typecheck → test → siguiente edición)?
3. ¿Hay revisiones agente-a-agente en tu flujo de PR, o todo lo revisa un humano?
4. ¿Cuántas categorías de PR tienes definidas (trivial/rutinario/sensible/decisión), o todos los PRs van por el mismo camino?
5. Cuando un test es flaky, ¿lo reintentan automáticamente o bloquea el merge indefinidamente?

### Mantenimiento

1. ¿Qué proporción del tiempo del equipo se dedica a "limpiar lo que el agente hizo mal"?
2. ¿Hay agentes recurrentes que corren en background y abren PRs de refactor pequeños?
3. Tienes "golden principles" / convenciones críticas, ¿están escritas en algún sitio que el agente lea automáticamente?
4. Cuando el mismo error aparece por tercera vez, ¿qué pasa?
5. ¿Algún agente analiza el harness mismo y propone mejoras?

## Matriz de inversiones: nivel actual → siguiente paso

Una vez tengas tus cinco niveles aproximados, esta matriz te dice qué inversión concreta desbloquea el siguiente nivel para cada dimensión. La idea no es subir todas a la vez: es identificar la dimensión donde estás más bajo y empezar por ahí.

### Aislamiento

| De | A | Inversión |
|---|---|---|
| 0 → 1 | Algo de aislamiento básico | Contenedor del proyecto, documentado, que cualquier dev puede levantar. Nada elegante. |
| 1 → 2 | Sandboxes dedicados | Un script (o herramienta) que crea un entorno por tarea. Manual está bien. |
| 2 → 3 | Sandboxes desechables y paralelizables | Automatización de creación/destrucción. Velocidad < 30s. Test: lanza 5 en paralelo y mira si alguno colisiona. |
| 3 → 4 | Observabilidad efímera por entorno | Logs/metrics/traces consultables por el agente *dentro* del sandbox. Pila como Vector + Victoria, o equivalente. |

### Contexto

| De | A | Inversión |
|---|---|---|
| 0 → 1 | AGENTS.md básico + README | Escribir un AGENTS.md, aunque sea monolítico. Cualquier punto de partida es mejor que ninguno. |
| 1 → 2 | Reducir AGENTS.md a índice, mover el resto a `docs/` | Crear `docs/` con secciones (`design-docs`, `product-specs`, `references`, `tech-debt-tracker.md`). |
| 2 → 3 | Planes versionados + doc-gardening | Mover los planes de ejecución activos al repo. Lanzar un agente recurrente que detecte docs obsoletos. |
| 3 → 4 | Repo como sistema de registro completo | Cualquier conocimiento que vive fuera del repo (Slack, Notion, GDocs) se materializa. Lints validan freshness. |

### Invariantes

| De | A | Inversión |
|---|---|---|
| 0 → 1 | Linters genéricos en CI | Habilitar y endurecer las herramientas estándar de tu stack (eslint, ruff, golangci-lint, etc.). |
| 1 → 2 | Primeros lints custom | Identificar 2-3 patrones que se repiten en revisiones y convertirlos en reglas. Empezar en warning. |
| 2 → 3 | Lints custom con mensajes para el agente + arquitectura mecanizada | Reescribir mensajes de error como instrucciones. Validar direcciones de dependencia entre capas. |
| 3 → 4 | Cobertura completa de invariantes, escritas por el agente | El propio agente propone y mantiene los lints. Cobertura de invariantes arquitectónicos al 100%. |

### Bucle / Flujo de PR

| De | A | Inversión |
|---|---|---|
| 0 → 1 | Bucle interno mínimo | El agente puede correr `test` y ver el resultado. Sin intervención humana en cada paso. |
| 1 → 2 | Bucle interno completo + PR automatizado | El agente ejecuta typecheck/test/lint en bucle hasta verde. Abre PR. Humano revisa antes de merge. |
| 2 → 3 | Categorización de PRs + revisiones agente-a-agente | Definir las 4 categorías (trivial/rutinario/sensible/decisión). Introducir un agente revisor para los rutinarios. |
| 3 → 4 | Flujo de PR mayoritariamente agente-a-agente | Múltiples agentes revisores especializados. Humano interviene por excepción. Merge gates ligeros con reversión rápida. |

### Mantenimiento

| De | A | Inversión |
|---|---|---|
| 0 → 1 | Cleanup explícito (peor que nada, pero es un paso) | Reservar tiempo recurrente para limpiar drift. Aceptar que no escala — es punto de partida. |
| 1 → 2 | Agentes recurrentes para casos puntuales | Empezar por uno: doc-gardening, o un agente que detecta TODOs huérfanos, o uno que actualiza dependencias. |
| 2 → 3 | Golden principles + flota de agentes recurrentes | Codificar las 5-10 reglas no negociables. Varios agentes en background, cada uno con foco. |
| 3 → 4 | Agentic flywheel | Un agente meta lee la señal acumulada de los sensores y propone mejoras al propio harness. |

## Cómo priorizar entre dimensiones

Si estás bajo en varias dimensiones a la vez (la mayoría de equipos lo están), usa esta heurística — en este orden — para decidir cuál atacar primero.

**1. Si estás en nivel 0 o 1 en Aislamiento, empieza ahí.** Sin aislamiento, casi todas las demás inversiones rinden la mitad. Es la dimensión con más dependencias hacia las otras y con más ROI multiplicador. No es negociable: si no tienes sandboxes, lo demás se queda atascado.

**2. Si Aislamiento ≥ 2 pero Contexto está en 0 o 1, ataca Contexto.** Un agente con buen sandbox pero mal contexto produce código sintácticamente correcto e ideológicamente equivocado. Subir Contexto a 2 ya elimina la mayoría de las divergencias arquitectónicas.

**3. Si Aislamiento y Contexto ≥ 2, mira en cuál estás más bajo: Invariantes o Bucle.** Las dos son importantes y se refuerzan mutuamente. Como heurística: si tu equipo se queja más de "el agente no sigue las convenciones", sube Invariantes. Si se queja más de "el agente tarda demasiado en iterar" o "tengo que revisar todo a mano", sube Bucle.

**4. Mantenimiento es la última, pero no la opcional.** Por debajo de nivel 2 en Mantenimiento, todo lo que has construido se va a degradar. La buena noticia: cuando subes las otras dimensiones a 2-3, Mantenimiento sube parcialmente sola, porque los agentes recurrentes tienen sobre qué operar.

**Regla negativa:** no intentes saltar dos niveles en una dimensión a la vez. Cada salto cambia los procesos del equipo, y tu equipo necesita tiempo para asentar el nivel anterior antes de subir. Un mes por salto es una velocidad razonable. Más rápido y se rompe.

## Tu hoja de ruta personal

Si quieres llevarte un único artefacto de este capítulo, es esta plantilla. Pásala con tu equipo en una sesión de 60-90 minutos y rellénala con honestidad:

```
Dimensión           Nivel actual   Próximo nivel   Inversión concreta   Plazo estimado
─────────────────   ────────────   ─────────────   ──────────────────   ──────────────
Aislamiento         _              _               _                    _
Contexto            _              _               _                    _
Invariantes         _              _               _                    _
Bucle / PR          _              _               _                    _
Mantenimiento       _              _               _                    _
```

Y luego, sin discusión, **encierra en un círculo la fila que vas a atacar este mes**. Solo una. La regla es disciplinaria: si encierras dos, no encierras ninguna.

Cuando termines esa inversión, vuelve a esta plantilla, vuelve a evaluar (probablemente tu nivel actual ha subido en algunas dimensiones colateralmente), y elige la siguiente. No hay un "harness terminado" — hay un harness en mejora continua, que es exactamente el tipo de sistema que esta guía describe en el resto de los capítulos.

## El cierre real de la guía

Hay una propiedad interesante de los harnesses bien construidos: **no se notan cuando funcionan**. El equipo entrega rápido, el código se mantiene coherente, los agentes producen cosas útiles, los humanos se concentran en lo importante, y nada de eso se siente como "estamos haciendo algo especial". Solo se siente como ingeniería bien hecha. Los equipos que lo tienen rara vez escriben sobre ello porque desde dentro parece evidente.

Los equipos que aún no lo tienen miran desde fuera y atribuyen la diferencia al modelo. El modelo es lo más visible — tiene marca, versión, número de parámetros. El harness es invisible. Y precisamente por eso es donde está la palanca: porque todos los demás están mirando al sitio equivocado.

Si solo te llevas una cosa de los doce capítulos, llévate esta: **tu próxima inversión no es esperar a un modelo mejor. Es construir el harness que tu modelo actual ya puede aprovechar**. Hay un siguiente paso concreto. Está en la fila que acabas de encerrar en un círculo. Hazlo este mes.
