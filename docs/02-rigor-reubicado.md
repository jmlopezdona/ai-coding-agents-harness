# 2. La reubicación del rigor

Cada vez que el software ha dado un salto importante, alguien ha aparecido para anunciar que "ya no hace falta tanto rigor". Y cada vez se ha equivocado. El rigor no desaparece; **se traslada a otro sitio**. Esta es probablemente la observación más útil que Chad Fowler hace sobre la era de la IA generativa, y conviene tenerla en mente antes de empezar a desmantelar prácticas que parecen "ya no necesarias".

## El patrón histórico

Tres ejemplos suelen citarse:

1. **Lenguajes dinámicos vs sistemas de tipos estáticos.** Cuando Ruby y Python se hicieron populares en los 2000s, mucha gente declaró muerto al tipado. No murió: el rigor se mudó al *test suite*. Los equipos serios de Ruby acabaron escribiendo más tests por línea de código que los equipos de Java de la misma época. La disciplina no se evaporó, cambió de lugar.
2. **Extreme Programming vs planificación waterfall.** Eliminar planes detallados y fases largas no significó "improvisar". Significó mover la rigurosidad a integración continua, refactor disciplinado y feedback inmediato. Lo que dejaste de hacer en un Gantt lo empezaste a hacer en cada commit.
3. **Despliegue continuo vs ventanas de release.** Quitar la ceremonia del release no relajó nada: exigió observabilidad de primera, reversibilidad inmediata, automatización exhaustiva y feature flags. El rigor se mudó al runtime y a la plataforma.

En los tres casos, quien interpretó el cambio como "ahora podemos ser más relajados" perdió. Quien entendió dónde se había mudado la disciplina, ganó.

## Dónde se mueve el rigor con los agentes de codificación

Con LLMs escribiendo código, el rigor sale de dos sitios y entra en otros dos:

**Sale de:**

- Escribir cada línea con cuidado artesanal.
- Revisar cada PR humano-a-humano como mecanismo principal de calidad.

**Entra en:**

- **Especificación de la intención.** Lo que antes eran "criterios de aceptación informales" en un ticket ahora tiene que ser legible mecánicamente, porque es lo que el agente va a interpretar. La imprecisión en el ticket se convierte en imprecisión en el código. Antes podías compensarlo con conversación de pasillo; con un agente que ejecuta de noche, no.
- **Evaluación verificable.** Si no puedes evaluar mecánicamente si la salida del agente es correcta, no tienes harness, tienes una ruleta. Tests, type checks, evals, validación de schemas, observabilidad, asserts en runtime — todo lo que convierte un "parece que funciona" en una señal binaria.

Chad Fowler lo dice mejor: *"generative systems only work if invariants are explicit rather than implicit"*. Un sistema generativo necesita que las cosas que tienen que ser ciertas estén dichas en algún lugar que la máquina pueda comprobar. Lo implícito — el conocimiento que vivía en la cabeza del senior, en el README de hace dos años, en la conversación de Slack del lunes — deja de funcionar como mecanismo de control.

## La consecuencia incómoda

Esto significa que los equipos que tenían más de su disciplina **codificada** (tipos, tests, schemas, IaC, observabilidad estructurada) están mejor posicionados para usar agentes que los equipos que confiaban en disciplina **cultural** (revisiones cuidadosas, seniors atentos, estándares no escritos). No porque los segundos sean peores, sino porque el agente no puede acceder a su disciplina. Es invisible para él.

El corolario práctico es que **dos repos con calidad humana equivalente pueden estar muy desigualmente preparados para agentes**. El que tiene la disciplina materializada en linters, schemas y tests funciona; el que la tiene en la cabeza de los seniors, no. La diferencia no se ve hasta que metes un agente y lo compruebas — y entonces es tarde para fingir que no estaba ahí.

## La señal de alarma

Chad Fowler propone una heurística que vale la pena adoptar: **cuando algo parezca dejar ir el rigor, busca dónde se reubicó. Si no lo encuentras, preocúpate**.

Aplicada a agentes:

- "Ahora el agente escribe los tests." ¿Quién valida que los tests *realmente* prueban lo que dicen probar? ¿Mediante qué mecanismo? Si la respuesta es "que los revisa un ingeniero", el rigor no ha desaparecido en teoría — pero a los volúmenes que produce un agente, en la práctica nadie los va a leer con la atención necesaria. No has reubicado el rigor a un sitio donde pueda sostenerse; lo has dejado en un sitio donde se va a evaporar por abandono.
- "Ya no hace falta documentar, el agente lee el código." ¿Cómo va a saber el agente qué decisiones de diseño *no* puede revertir? El código no te dice por qué *no* hiciste algo. Si esa información no está en alguna parte legible, el rigor desapareció.
- "Mergeamos rápido y el agente arregla lo que rompa." Vale, pero entonces necesitas un sistema de detección de regresiones más rápido y más sensible que antes. Si lo tienes, perfecto. Si no, has cambiado disciplina por velocidad y la velocidad va a desaparecer en tres meses, cuando la deuda te alcance.

## El corolario práctico

Antes de adoptar un agente en serio, haz inventario de las cosas en las que tu equipo confiaba implícitamente: convenciones, "el senior siempre revisa esto", "todos sabemos que no se toca este módulo", "Slack del 14 de marzo donde decidimos X". Cada una de esas piezas tiene que mudarse a un sitio donde el agente pueda verlas. Si no lo haces, no es que el agente "no esté listo" — es que tu disciplina sigue viviendo en sitios donde no puede leerse mecánicamente, y por tanto, para el agente, no existe.

El resto de la guía es, en buena medida, un catálogo de dónde reubicar cada tipo de rigor: en guías explícitas (cap. 3), en sensores automáticos (cap. 3), en la arquitectura mecánicamente validada (cap. 7), en el contexto versionado del repo (cap. 6), en bucles de mantenimiento continuo (cap. 9). Ninguna de estas inversiones es opcional. Son los nuevos sitios donde el rigor tiene que vivir.
