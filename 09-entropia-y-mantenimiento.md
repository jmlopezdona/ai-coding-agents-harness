# 9. Entropía y mantenimiento continuo

Aquí es donde la mayoría de equipos descubren, mes tres o cuatro, que tenían un problema que no habían anticipado. La pregunta inocente — "si el agente escribe tan rápido, ¿cómo va a ser el código en seis meses?" — tiene una respuesta inquietante por defecto: **caótico, inconsistente y frágil**, a menos que diseñes un sistema explícito para evitarlo. Ese sistema es lo que este capítulo describe.

## La fuente del problema

Los agentes son notablemente buenos imitando los patrones que ya existen en el repo. Esto es una virtud — heredan convenciones, mantienen consistencia local, no inventan estilos nuevos cada hora — y también es la fuente del problema. Porque imitan **todos** los patrones, incluidos los irregulares, los obsoletos, los que alguien introdujo un viernes a las 6pm.

OpenAI describe el ciclo así: el agente replica patrones existentes incluso los subóptimos. Con el tiempo, esto produce drift. Y el drift en una base de código generada por agentes es cualitativamente distinto del drift en una base humana: es más rápido, más consistente (es decir, los errores se replican más fielmente), y más difícil de detectar mediante revisión casual porque el código sigue "pareciendo" coherente.

Es un poco como una fotocopiadora que copia una marca pequeña en el cristal y la empieza a poner en todas las copias. Si nadie limpia el cristal, en seis meses todas las copias tienen la marca y nadie recuerda por qué.

## La respuesta ingenua y por qué falla

La primera reacción típica es la viernes-de-limpieza. Un día a la semana, el equipo se sienta a limpiar lo que el agente ha hecho mal. OpenAI describe haber probado exactamente esto y abandonado: solían dedicar los viernes (un 20% de la semana) a "AI slop cleanup", y simplemente no escalaba.

Las razones son obvias en retrospectiva:

- **El throughput del agente sube más rápido que la capacidad humana de limpieza.** Cuanto mejor el harness, peor escala este modelo.
- **El conocimiento de "qué está mal" se queda en la cabeza del que limpia.** No se traduce en mejoras del harness. La semana siguiente, el agente comete los mismos errores.
- **Limpiar es trabajo de baja moral.** Los ingenieros más senior lo evitan o lo hacen mal. La calidad del cleanup decae hasta que ya no compensa.

La conclusión inevitable: **el mantenimiento tiene que ser parte del loop, no un evento separado**.

## Garbage collection continuo

OpenAI llama a su solución "garbage collection" y la analogía es exacta. En vez de acumular basura y dedicar tiempo a tirarla, tienes un proceso que la recoge continuamente, en pequeñas cantidades, mientras la base de código sigue funcionando.

En la práctica esto se ve así:

**Golden Principles codificados.** Reglas mecánicas, deliberadamente prescriptivas, que capturan el "gusto" del equipo de forma legible para el agente. No "preferimos código limpio", sino "preferimos los paquetes de utilidades compartidos sobre helpers ad-hoc, para mantener invariantes centralizadas". Cada principio está documentado *con su porqué*, para que el agente pueda aplicar juicio en casos límite.

**Agentes recurrentes en background.** Tareas Codex (o equivalentes) que corren en una cadencia regular — cada noche, cada semana — y hacen una sola cosa cada uno: detectar drift respecto a un principio concreto, actualizar quality scores, abrir PRs de refactor pequeños. Cada PR es revisable en menos de un minuto y mergea automáticamente si pasa los sensores.

**Doc-gardening.** Un agente recurrente que escanea documentación y la compara con el comportamiento real del código. Cuando detecta divergencia, abre un PR que actualiza el doc o (si la doc es la fuente correcta) flag al humano. Es el equivalente a un linter de docs, pero inferencial.

**Tracker de deuda explícito.** Un archivo en el repo (`tech-debt-tracker.md` en el caso de OpenAI) que enumera lo que está mal a propósito. Esto sirve dos funciones: le dice al agente qué *no* arreglar (lo que está mal a propósito) y qué *sí* arreglar cuando tenga capacidad. Sin este archivo, el doc-gardener "arregla" cosas que el equipo había decidido dejar como estaban.

## La filosofía: deuda como préstamo de altos intereses

La metáfora que vale la pena interiorizar (y otra vez es de OpenAI): **la deuda técnica es un préstamo de altos intereses; casi siempre es mejor pagar continuamente en pequeños incrementos que dejarla acumular para abordarla en montañas dolorosas**.

Esto cambia varias cosas operativas:

- **No esperas a "la próxima sprint de cleanup".** No existe. El cleanup es ambient.
- **No batchas refactors grandes.** Refactor pequeño hoy + refactor pequeño mañana > refactor enorme en seis semanas.
- **El valor del refactor pequeño no es solo el código mejor.** Es que mantiene la base de código en un estado donde el próximo cambio del agente es más probable que sea correcto. Los refactors son inversiones en el harness, no solo en el código.

## El "gusto humano" capturado una vez

Hay una frase que vale la pena leer dos veces: *"el gusto humano se captura una vez y luego se aplica continuamente en cada línea de código"*. Es la diferencia entre un equipo donde el senior tiene que estar mirando todo, y un equipo donde el senior dedica tiempo a **traducir su gusto en reglas ejecutables** y luego deja que el sistema lo aplique.

Esto vuelve cada hora de trabajo del senior radicalmente más valiosa. En un flujo tradicional, una review consume 30 minutos del senior y mejora *un* PR. En un flujo agéntico bien hecho, esos mismos 30 minutos se invierten en escribir un golden principle o un lint custom, y mejoran *todos los PRs futuros*. El multiplicador es enorme y compounding.

La pregunta diagnóstica para un senior: ¿cuánto del tiempo que dedicas a calidad termina como código que se ejecuta automáticamente, y cuánto termina como conocimiento en tu cabeza que se evapora cuando dejes el equipo? Cuanto más alta la primera proporción, más sano el harness.

## Las tres capas de mantenimiento

Es útil pensar el mantenimiento como tres capas que actúan en escalas de tiempo distintas:

**Capa 1 — sensores en CI (segundos a minutos).** Detectan los problemas obvios en el momento del PR. Lints, tests, type checks. Esta capa es la primera línea y es la más barata.

**Capa 2 — agentes recurrentes (horas a días).** Detectan drift acumulado, doc rot, patrones replicados, oportunidades de consolidación. Corren en background, no bloquean a nadie, abren PRs revisables.

**Capa 3 — re-arquitectura puntual (semanas).** Cambios estructurales que requieren criterio humano. Cuando los golden principles cambian, cuando una capa entera necesita rediseñarse, cuando una librería tiene que reemplazarse. Estos siguen necesitando humanos en el driver seat — pero son raros si las dos primeras capas funcionan.

Si te falta la capa 1, el agente introduce errores constantemente. Si te falta la capa 2, los errores que la capa 1 no captura se acumulan en silencio. Si te falta la capa 3, eventualmente llegas a un techo arquitectónico que ningún agente puede romper. Las tres son necesarias; ninguna es suficiente.

## La señal de que está funcionando

Un equipo que tiene esto bien suele notar dos cosas a los pocos meses:

1. **Las PRs de los agentes se vuelven más cortas, no más largas.** Porque el contexto está mejor estructurado, la mayoría de cambios son pequeños y dirigidos. Las megapull requests de mil líneas son síntoma de harness inmaduro.

2. **Los problemas recurrentes desaparecen sin que nadie los persiga.** Porque cada problema recurrente generó un guide o un sensor cuando se detectó la segunda vez. La memoria del equipo vive en el harness, no en la cabeza de la gente.

Si en cambio observas que las PRs son cada vez más largas, que los mismos errores siguen apareciendo, y que el equipo dedica cada vez más tiempo a "arreglar lo que el agente rompió", el harness está perdiendo. La inversión correcta no es "más limpieza humana" — es promover lo que se aprende durante la limpieza al harness, hasta que la limpieza no haga falta.

## El flywheel agéntico: cuando el harness se mantiene a sí mismo

Hay un punto de maduración que vale la pena nombrar porque es hacia donde apunta naturalmente todo lo descrito en este capítulo. Kief Morris lo llama el **agentic flywheel**: agentes que no solo escriben código, sino que **analizan resultados y recomiendan mejoras al propio harness**, e incluso las implementan bajo supervisión humana estratégica.

El bucle se vuelve algo así:

1. Sensores detectan un patrón recurrente (un tipo de bug, una clase de drift, un fallo de doc-gardening).
2. Un agente meta lee la señal acumulada y propone una mejora: "este patrón lo hemos arreglado siete veces este mes; sugiero el siguiente lint custom" o "este doc ha desviado del código tres veces; sugiero esta restructuración".
3. La propuesta llega al humano como un PR sobre el harness mismo — no sobre el código de la aplicación.
4. El humano aprueba, ajusta o rechaza con argumentos, y el harness mejora.
5. El siguiente ciclo de detección parte de un harness más capaz.

Esto suena ambicioso y conviene calibrarlo: no es el punto de partida, es la dirección. Un equipo que empieza intentando construir el flywheel completo cae en el anti-patrón 7 (megalomanía del plan inicial). Pero un equipo que va promoviendo aprendizajes al harness con disciplina termina, casi por gravedad, llegando a un punto donde las propuestas de mejora también se generan automáticamente. El flywheel no se diseña; se descubre cuando ya estabas haciendo casi todo lo que requiere.

La señal de que estás cerca: más del 50% de los PRs que tocan el harness los abre un agente, y el trabajo del humano se reduce a aprobar, ajustar o vetar con criterio. Cuando llegues ahí, el rol humano ha completado su transición — y lo que has construido es un sistema que aprende a mejorarse solo.
