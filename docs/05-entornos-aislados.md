# 5. Entornos aislados y reproducibles

Si tuviera que elegir una sola inversión de infraestructura que más diferencia hace al introducir agentes en un equipo, sería esta: **entornos aislados, baratos de crear y baratos de tirar**. No es la más glamurosa, no aparece en demos, y casi siempre se subestima. Pero es la que multiplica todo lo demás.

## El problema que resuelve

Sin aislamiento, un agente trabajando en tu repo es un riesgo y un cuello de botella al mismo tiempo. Riesgo, porque cualquier comando que ejecute (instalar dependencias, modificar la base de datos, tocar variables de entorno) afecta a tu sistema y al de tus compañeros. Cuello de botella, porque solo puedes tener un agente corriendo a la vez sin que se pisen.

Con aislamiento, los dos problemas desaparecen a la vez. Y abren una tercera capacidad que es la que cambia el juego: **paralelismo**. Puedes lanzar diez agentes a la vez en diez tareas, cada uno en su propio entorno, sin coordinación.

## Las dos referencias

**Stripe — devboxes.** Stripe usa instancias EC2 estandarizadas que están "hot and ready" en 10 segundos. Cada minion (su agente interno) se levanta en uno, hace su trabajo, abre un PR, y el devbox se desecha. La latencia de creación es importante: si tardara 5 minutos, el agente esperaría en cada tarea y el sistema sería mucho menos eficiente. 10 segundos es lo bastante rápido como para tratar el entorno como desechable de verdad.

**OpenAI — worktrees git booteables.** OpenAI tomó otra ruta: cada git worktree puede arrancar la aplicación entera, con su propia pila de observabilidad efímera (Vector, Victoria metrics/logs/traces). Cada cambio se valida en su propio worktree, con sus propios logs. Cuando el worktree se borra, todo desaparece con él.

Las dos soluciones son distintas en mecánica pero idénticas en filosofía: **el entorno es desechable, instantáneo y completo**.

## Las propiedades que importan

Un entorno aislado para agentes debe ser:

**Aislado de verdad.** Si dos agentes corriendo en paralelo pueden colisionar (en la misma base de datos, en el mismo puerto, en el mismo archivo de cache), no es aislamiento, es ilusión. El test es brutal: lanza 5 agentes a la vez y mira si alguno falla por el otro. Si pasa, no tienes aislamiento.

**Barato de crear.** "Barato" significa segundos, no minutos. Un entorno que tarda 3 minutos en estar listo te empuja a reusarlo, y reusarlo te lleva a contaminación cruzada, que rompe el aislamiento. La velocidad de arranque no es comodidad: es lo que sostiene la disciplina del aislamiento.

**Barato de tirar.** Sin pasos manuales, sin "acuérdate de limpiar X", sin recursos huérfanos en la nube. La operación normal es: crear, usar, destruir. Si destruir es caro, los entornos se acumulan y el coste se descontrola.

**Completo.** El agente tiene que poder hacer en el entorno aislado *todo* lo que necesita: ejecutar tests, levantar la app, consultar logs, navegar la UI, hablar con dependencias mockeadas o reales. Un entorno donde el agente "casi puede" hacer todo es peor que ninguno: las cosas que no puede se convierten en bloqueos invisibles.

**Reproducible.** Dos creaciones del mismo entorno deben dar el mismo resultado. Si los entornos divergen (versiones, semillas, datos), el agente empieza a ver fallos intermitentes y los achaca a su propio código. Pierdes confianza en el sensor.

## Lo que el aislamiento desbloquea

Una vez tienes esto, varias cosas se vuelven posibles que antes parecían demasiado caras:

- **Observabilidad efímera.** Como en OpenAI, puedes correr una pila completa de logs/metrics/traces *por entorno*. El agente formula preguntas como "¿este request tarda menos de 200ms?" y obtiene respuesta verificable, sin contaminar producción ni los entornos compartidos del equipo.
- **UI reproducible.** El agente puede levantar la app entera, navegarla con DevTools Protocol, sacar screenshots, comparar antes/después. La UI deja de ser un terreno donde solo los humanos pueden validar.
- **Pruebas destructivas.** Operaciones que normalmente nadie quiere ejecutar (drop tables, reset migrations, simular fallos de red) se vuelven seguras dentro de un entorno desechable.
- **Tareas largas sin culpabilidad.** Si el agente tarda 6 horas en una tarea (OpenAI menciona ejemplos de eso), nadie se preocupa, porque está corriendo en su propio entorno y no bloquea a nadie.
- **Fan-out trivial.** "Prueba estas cinco implementaciones distintas en paralelo y dime cuál es mejor" deja de ser un ejercicio teórico. Cinco entornos, cinco bucles, una comparación al final.

## Cuánto invertir

La pregunta razonable es: ¿hasta dónde llevar esto? La respuesta práctica es **hasta el punto donde lanzar un nuevo entorno deja de costarte una decisión consciente**. Si te lo piensas dos veces antes de lanzar un agente porque "vaya, otro entorno", todavía no estás ahí. El aislamiento ha funcionado cuando el coste mental de lanzar un agente es indistinguible de el coste mental de abrir una pestaña del navegador.

## Lo que no es aislamiento

Conviene desambiguar. Estas cosas suelen confundirse con aislamiento y *no* lo son:

- **"El agente corre en mi máquina":** no es aislamiento; es lo contrario. Estás compartiendo entorno con todo lo que tienes abierto.
- **Una rama de git:** una rama es aislamiento de cambios, no de runtime. Dos agentes en dos ramas pueden seguir pisándose si comparten DB local, puertos, o cache.
- **Un contenedor que se reusa entre tareas:** mejor que nada, pero no es desechable. La contaminación cruzada acumulada se vuelve fuente de fallos intermitentes.

El estándar a alcanzar es: cada tarea, su propio entorno; al terminar, se desecha; arranca en segundos; reproducible byte-a-byte. Cuando llegas ahí, casi todos los demás capítulos de esta guía se vuelven más fáciles de aplicar.
