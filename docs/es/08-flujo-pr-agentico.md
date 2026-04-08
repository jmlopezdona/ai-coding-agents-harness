# 8. El flujo de PR agéntico

El pull request es la unidad atómica del trabajo en software. Y cuando los agentes empiezan a generarlos en serio — Stripe mergea más de mil por semana, OpenAI ha mergeado mil quinientos en cinco meses con un equipo pequeño — todos los supuestos del flujo de PR tradicional saltan por los aires. Este capítulo es sobre qué reemplazarlos por.

## Lo que rompe primero

El primer supuesto que se cae es **"todo PR pasa por un humano"**. No por ideología, sino por aritmética. Si tienes 7 ingenieros y el agente abre 20 PRs al día por persona, son 140 PRs diarios. Asumiendo que un humano pueda hacer una revisión decente en 15 minutos, son 35 horas-humano de revisión por día. No salen las cuentas.

La respuesta intuitiva — "entonces ralentizamos al agente" — es exactamente la equivocada. Lo que tienes que hacer es lo contrario: **delegar la mayoría de la revisión a otros agentes**, y reservar la atención humana para los casos donde realmente importa el criterio.

## El "Ralph loop" aplicado al PR

OpenAI describe su flujo como una iteración continua que parece escrita por un guionista de los Simpson y funciona mejor de lo que parece. El agente:

1. Hace su cambio en su worktree aislado.
2. Revisa sus propios cambios localmente, antes de abrir nada.
3. Solicita revisiones a otros agentes (locales y en la nube), cada uno con foco distinto: arquitectura, seguridad, tests, performance.
4. Lee los comentarios — vengan de otros agentes o de humanos — y responde en línea.
5. Aplica las correcciones que considere válidas, argumenta contra las que no.
6. Vuelve al paso 3 hasta que todos los revisores están satisfechos.
7. Mergea (sí, mergea él mismo) cuando todos los gates pasan.

La revisión humana en este flujo no desaparece, pero **se vuelve opcional para la mayoría de PRs**. El humano interviene cuando un agente revisor escala explícitamente, cuando el cambio toca un área marcada como sensible, o cuando el ingeniero responsable elige mirar.

## Las cuatro categorías de PR

En la práctica, los equipos que llegan a este punto acaban categorizando los PRs en cuatro grupos, aunque no siempre lo hagan explícito:

**1. Trivial — auto-mergeable.** Refactors mecánicos, corrección de typos, actualización de docs, fixes de lint. El agente abre, los sensores pasan, mergea. Cero atención humana.

**2. Rutina — revisado por otros agentes.** La mayoría del trabajo. Implementación de una funcionalidad pequeña, fix de un fallo bien delimitado, tests adicionales. Un agente revisor lee, comenta, el agente original itera. Si convergen, mergean. El humano podría mirar pero no tiene por qué.

**3. Sensible — humano en el bucle.** Cambios en módulos marcados como críticos (auth, billing, migrations, contratos públicos). El humano revisa, decide, posiblemente delega de vuelta al agente para iterar. La marca de "sensible" no la decide cada PR; la decide la arquitectura, codificada en metadatos del repo.

**4. Decisión — humano dirige desde antes.** Cambios que requieren criterio (qué endpoint exponer, qué precio cobrar, qué deprecar). El humano no revisa el PR: define el plan antes de que el agente abra nada. El agente solo ejecuta.

La proporción típica que se observa en equipos maduros: 60% rutina, 25% trivial, 10% sensible, 5% decisión. Y los porcentajes de las dos primeras siguen creciendo a medida que el harness madura.

## Merge gates ligeros

Otro supuesto que se rompe: **"el merge tiene que ser cuidadoso"**. OpenAI describe explícitamente que sus merge gates son "mínimamente bloqueantes". Las PRs son cortas, los tests intermitentes se reintentan en lugar de bloquear progreso, y los fixes para fallos pequeños llegan en PRs de seguimiento en vez de detener el flujo.

Esto suena irresponsable. En un equipo de bajo rendimiento lo sería. En un equipo donde el agente puede arreglar lo que rompa en minutos, **el coste de esperar es mayor que el coste de romper**. La disciplina se reubica (otra vez) hacia detección rápida y reversión barata. Si esos dos no están, no apliques este patrón. Si están, no aplicarlo es dejar dinero en la mesa.

Hay dos pre-requisitos no negociables para que esto funcione:

- **Detección rápida.** Sensores en CI que descubren regresiones en minutos, no en horas. Smoke tests, contract tests, type checks.
- **Reversión trivial.** Capacidad de revertir un PR sin ceremonia, idealmente automática si una alerta se dispara dentro de N minutos del merge.

Si tienes los dos, los merge gates pueden ser ligeros. Si no los tienes, los merge gates ligeros son una receta para descontrol.

## La revisión agente-a-agente como sensor inferencial

En el lenguaje del capítulo 3, la revisión por otro agente es un sensor *inferencial*: no es determinista, pero captura cosas que los sensores computacionales no pueden. Cosas como:

- "Este nombre es confuso dado el dominio."
- "Esta función no maneja el caso vacío y el contexto sugiere que es importante."
- "Este cambio contradice la decisión del plan de ejecución activo en `docs/exec-plans/active/billing-rewrite.md`."
- "Esta solución es válida pero hay un patrón ya establecido en `domains/orders/repository.ts` que deberías reusar."

Estos comentarios un humano los haría, pero un humano cuesta caro y no escala. Un agente revisor con buen contexto los hace bien la mayoría del tiempo, y los falsos positivos los filtra el agente original argumentando.

Una propiedad importante: **las revisiones agente-a-agente deben tener foco**. Un único agente intentando revisar "todo" produce revisiones mediocres. Múltiples agentes revisores con prompts especializados — uno para arquitectura, otro para seguridad, otro para tests, otro para coherencia con docs activas — producen señal mucho mejor. Stripe lo formaliza con su modelo de "blueprints" combinando nodos determinísticos y agénticos en pipelines explícitos.

## El humano como revisor de excepción

Cuando un humano sí entra a revisar, su papel cambia. Ya no es "el sistema de calidad", porque hay sensores antes que él. Su papel es:

- **Aplicar criterio que no se puede codificar.** Decisiones de producto, compromisos estratégicos, juicio sobre si una solución "encaja" con la dirección del proyecto.
- **Identificar patrones que merecen subir al harness.** Si está corrigiendo lo mismo dos veces, eso no es revisión, es señal de que falta una guía o un sensor. Su trabajo más valioso es traducir esa observación en una mejora del harness.
- **Validar áreas marcadas como sensibles** donde el coste de un fallo justifica la latencia.

Lo que el humano *no* tiene que hacer en este modelo: leer cada línea, discutir naming, validar que los tests existen, asegurarse de que el lint pasa. Eso es trabajo de los sensores. Si el humano lo está haciendo, los sensores están incompletos.

## El PR como evidencia, no como conversación

Un cambio sutil que aparece en estos flujos: el PR deja de ser una conversación entre humanos y se vuelve **un registro de evidencia**. El agente original adjunta:

- El plan que siguió (referenciado al `exec-plans/active/`).
- Los logs relevantes del worktree donde validó.
- Screenshots o vídeos del comportamiento antes y después (cuando es UI).
- Las preguntas que se hizo a sí mismo y cómo las resolvió.

OpenAI menciona explícitamente que su agente puede grabar un vídeo del fallo, otro de la corrección, y adjuntarlos al PR. Esto cambia la naturaleza de la revisión: el revisor (humano o agente) no tiene que reproducir nada para entender el cambio. La evidencia ya está. Si la evidencia falta, la revisión es legítimamente bloqueada y se le pide al agente que la añada.

## Lo que esto pide del repo

Para que este flujo funcione, el repo necesita estar preparado: marcas de "área sensible" legibles mecánicamente (un metadatos.yml por dominio, por ejemplo), planes de ejecución vivos en una ubicación conocida, agentes revisores con prompts versionados, y sensores rápidos en CI. Es trabajo. Pero es trabajo que se amortiza muy rápido cuando el rendimiento de PRs sube.

La pregunta diagnóstica final: **si mañana tu agente abre 50 PRs en un día, ¿qué pasa?** Si la respuesta es "tendríamos que parar al agente porque no podemos revisarlos", el flujo de PR no está listo. Si la respuesta es "los sensores filtrarían los problemas y los humanos solo verían los 3-4 que importan", estás cerca.
