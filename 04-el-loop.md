# 4. El loop: iteración como primitiva

Hay una observación que se repite en todos los equipos que operan agentes en serio, y que Geoffrey Huntley articula mejor que nadie: **el agente no es una función, es un loop**. Tratarlo como una función — entrada, salida, listo — es la fuente de la mitad de los problemas que ves en equipos que están empezando.

## El cambio de modelo mental

Cuando llamas a un LLM "puro" — chat, completar un mensaje — es razonable pensar en él como una función. Le das un prompt, devuelve un texto. Pero un agente no funciona así. Un agente:

1. Recibe un objetivo.
2. Mira el entorno (lee archivos, ejecuta comandos, consulta herramientas).
3. Decide una acción.
4. La ejecuta.
5. Observa el resultado.
6. Decide si continuar, retroceder o terminar.
7. Vuelve al paso 2.

Eso es un loop. Y lo importante es que **la calidad del agente no depende solo de qué decisión toma en cada paso, sino de cuántas iteraciones puede sostener antes de descarrilar**. Un modelo regular en un loop bien diseñado supera a un modelo excelente en un loop mal diseñado. Esto es contraintuitivo y es donde se pierde más dinero en la industria ahora mismo.

## El "Ralph loop" como primitivo

Huntley describe lo que llama el *Ralph loop* (en honor al Ralph Wiggum de los Simpson): un orquestador monolítico que ejecuta una iteración del loop y vuelve a empezar. Sin lógica complicada de planificación, sin árboles de decisión exquisitos. Solo: ejecuta una vuelta, observa, ejecuta otra.

La intuición que hay detrás es importante. Cuando un agente falla, hay dos respuestas posibles:

- **Respuesta frágil:** añadir lógica que detecte ese fallo concreto y lo evite. Cada bug genera código de orquestación.
- **Respuesta robusta:** asegurarte de que el loop puede *darse cuenta* de que falló y reintentar de forma diferente. El bug genera un sensor mejor o un guide mejor, no más lógica.

Huntley llama a esto "watch the loop": mirar dónde el loop falla repetidamente y, en vez de parchearlo, ponerse el sombrero de ingeniero y arreglar la condición sistémica que lo causa. Es la misma filosofía que SRE: no celebres el héroe que apaga el incendio cada noche, arregla la causa del incendio.

## Tres propiedades de un buen loop

Si vas a invertir en el loop (y deberías), busca tres propiedades:

**1. Observable.** Cada iteración debe producir señales legibles. No basta con que el agente "termine"; tiene que dejar rastro de qué intentó, qué vio, qué decidió. Los logs del loop son tu único mecanismo de depuración cuando algo va mal a las 3am.

**2. Interrumpible y reanudable.** Un loop que necesita correr ininterrumpidamente seis horas y se pierde si lo paras es frágil. Un loop que puede pararse en cualquier punto, persistir su estado, y reanudarse, es resiliente. OpenAI menciona que ejecuciones individuales de Codex pueden trabajar más de seis horas en una sola tarea — eso solo es posible si el loop es persistente.

**3. Autoterminante por progreso, no por intentos.** "Reintenta hasta 5 veces" es una mala condición de parada. "Reintenta mientras estés haciendo progreso medible" es una buena. La diferencia se ve cuando el agente lleva 4 vueltas trabajando bien en una tarea difícil y el loop lo mata por timeout absurdo.

## Loops anidados

En equipos serios, raramente hay un único loop. Lo que ves es una jerarquía:

- **Loop interno (segundos):** edición → typecheck → test rápido → siguiente edición. Es el bucle más caliente, el que el agente recorre cientos de veces por tarea.
- **Loop intermedio (minutos):** PR → CI → revisión automática → respuesta a comentarios → re-merge. Es el loop que Stripe describe en sus "blueprints", combinando nodos determinísticos (linters, tests) con nodos agénticos (implementar, refactorizar).
- **Loop externo (horas o días):** doc-gardening, refactor agents, eliminación de drift, actualización de quality scores. Es el loop que mantiene la base de código habitable a meses vista.

Los tres loops se influyen. Un fallo recurrente en el loop interno se convierte en un guide nuevo en el loop externo. Una decisión del loop externo (subir el umbral de cobertura) cambia el comportamiento del loop interno. Diseñarlos como un sistema, no como tres scripts inconexos, es lo que distingue un harness real de una colección de automatizaciones.

## El loop como filosofía, no solo como código

Huntley insiste en que el loop también es una mentalidad. Cuando ves al agente fallar, no preguntes "¿qué le digo para que esto no pase?". Pregunta "¿qué bucle necesito construir para que esto se detecte y se corrija sin mí?". El primer reflejo es agotador y no escala. El segundo es ingeniería.

Hay un peligro asociado: enamorarse del loop hasta el punto de no terminar nunca de construirlo. El loop existe para resolver tareas reales; si pasas tres semanas sin entregar nada porque estás "perfeccionando el loop", el loop se ha convertido en otro proyecto huérfano. La regla de oro: cada inversión en el loop debe estar justificada por un fallo real, recurrente y reciente. No por uno hipotético.

## Una nota sobre paralelismo

El loop natural de un agente es secuencial: una iteración después de otra. Pero a nivel de equipo, lo interesante es **lanzar muchos loops en paralelo**. Stripe describe esto explícitamente: los ingenieros lanzan múltiples minions a la vez, cada uno trabajando en una tarea distinta en su propio devbox. La paralelización no ocurre dentro del loop, ocurre *entre* loops, y depende enteramente de tener entornos aislados (próximo capítulo) en los que varios loops puedan correr sin pisarse.

Esto es la diferencia entre tener "un agente" y tener "agentes". El primero es una herramienta. El segundo es capacidad de equipo. Y la transición de uno al otro pasa, casi siempre, por aceptar que la primitiva no es el modelo, ni el prompt, ni la herramienta: es el loop.
