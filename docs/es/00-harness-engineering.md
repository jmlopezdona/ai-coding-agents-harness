# Harness engineering: producir software a escala con agentes

Llevamos un par de años en una conversación pública sobre agentes de codificación que está casi enteramente centrada en el sitio equivocado: el modelo. Qué versión, qué benchmark, qué context window, qué proveedor, qué precio por millón de tokens. Es una conversación legítima, pero deja fuera lo que de verdad explica por qué algunos equipos están entregando software con agentes a una escala difícil de creer y otros equipos siguen tratándolos como un autocomplete glorificado. La diferencia no está en el modelo. Está en todo lo que hay alrededor del modelo.

A ese "todo lo demás" la comunidad le ha puesto un nombre que merece quedarse: **el harness**. El término emergió en torno a LangChain — con la fórmula *"Agent = Model + Harness"* — y lo articuló con más detalle Birgitta Böckeler en *Harness Engineering* (Thoughtworks, publicado en martinfowler.com). Este texto es sobre por qué el harness importa más que el modelo, qué forma tiene cuando está bien hecho, y por dónde empezar si tu equipo ya tiene experiencia con agentes pero siente que no está sacándoles partido.

## La aritmética que cambia todo

Antes de entrar en materia, conviene dejar tres datos sobre la mesa.

**Stripe** mergea **más de 1.000 pull requests por semana** generados por agentes. La base de código tiene cientos de millones de líneas, en Ruby con Sorbet, con librerías internas que ningún modelo público conoce. Y aún así, los agentes funcionan.

**OpenAI** construyó un producto interno con **un millón de líneas de código y 1.500 PRs en cinco meses**, con un equipo que creció de 3 a 7 ingenieros, bajo una regla absoluta: **cero líneas escritas a mano**. Ninguna. Ni la lógica, ni los tests, ni la configuración de CI, ni la documentación, ni los scripts de herramientas. Todo Codex.

**Geoffrey Huntley** describe un patrón que llama "Ralph loop" — un orquestador minúsculo y monolítico que ejecuta una iteración, observa el resultado, y vuelve a empezar — y lo presenta no como un truco, sino como la primitiva más importante del trabajo agéntico.

Si lo único que te separa de estos equipos fuera el modelo, todos tendríamos sus números. No los tenemos. Lo que ellos tienen y la mayoría no es **infraestructura interna alrededor del modelo**: sandboxes desechables, contexto estructurado, validadores automáticos, bucles de feedback rápidos, agentes revisando a otros agentes, y una mentalidad que trata al agente como un sistema que hay que ingeniar, no como una caja mágica que hay que rezarle.

## La tesis, en una frase

> El modelo es el motor. El harness es lo que convierte un LLM en un agente del que un equipo puede depender.

El modelo es intercambiable. Sale uno nuevo cada seis meses, todos parecidos en lo importante. El arnés no. El arnés lo construyes tú, evoluciona con tu proyecto, captura el conocimiento de tu equipo, y se acumula sobre sí mismo a meses vista. Es, en términos directos, **donde está tu ventaja competitiva**. El modelo está donde está la commodity.

Esto es, en el fondo, una buena noticia. El modelo no lo controlas — lo controla un proveedor que va a optimizarlo para todos por igual. El arnés sí. Y como casi todo lo que controlas en ingeniería, mejora con intención, con disciplina, y con un equipo que entiende qué está construyendo y por qué.

## Qué es exactamente un harness

Böckeler descompone el arnés en dos mecanismos, y la distinción es la herramienta más útil que vas a llevarte de este texto:

- **Guías (feedforward).** Indicaciones que orientan al agente *antes* de que actúe. Convenciones, plantillas, schemas, generadores de módulos, documentos de arquitectura legibles, AGENTS.md como mapa del repo. Le dicen al agente cómo proceder.
- **Sensores (feedback).** Guardarraíles que actúan *después*. Tests, type checkers, linters, observabilidad efímera, evals, otros agentes revisando. Detectan cuándo el agente se ha salido del camino con suficiente velocidad y precisión para que el arnés pueda corregirlo sin tu intervención.

Un arnés sin guías produce un agente que se desvía. Un arnés sin sensores produce un agente que se desvía y nunca se entera. Los dos juntos, bien calibrados, producen algo que empieza a parecerse a un colaborador.

Cada vez que tu agente falla, deberías poder decir si el fallo se debe a una guía ausente o a un sensor ausente. Si no puedes, no tienes arnés. Tienes prompts y suerte.

## El harness que ya tienes y el que tienes que construir

Conviene aclarar algo antes de seguir: cuando hoy hablas de un "agente de codificación" rara vez te refieres a un LLM desnudo. Claude Code, GitHub Copilot, Cursor, Codex y similares **ya son un harness** sobre el modelo — tool calls, ejecución en sandbox, lectura y escritura de archivos, retrieval del repo, memoria de sesión, reglas de stop, subagentes, hooks, MCP, skills. Cada versión nueva empuja más capacidades dentro del producto: lo que hace doce meses tenías que construir a mano hoy viene de fábrica, y esa tendencia va a continuar a medida que modelos y herramientas maduran.

Esto no elimina la ingeniería de harness; la desplaza. La capa que el vendor te da sirve para todos los equipos por igual — es una commodity compartida, igual que lo es el modelo. La capa que *tu* equipo construye — convenciones de tu repo, sensores sobre tu dominio, contexto materializado en tu markdown, validadores de tus invariantes, flujo de PR específico — es la que solo puedes construir tú, y es donde sigue estando la ventaja. La pregunta no es si usar un AI tool con harness incorporado (lo vas a hacer), sino **qué capa de harness engineering construyes encima**. Cuando una capacidad nueva sube al harness del vendor, retira esa parte de tu trabajo y empuja tu inversión hacia lo que aún queda específico de tu contexto.

## La pregunta que hay que aprender a hacerse

Hay una pregunta concreta que vale más que cualquier framework. Cuando el agente falla — y va a fallar — la mayoría de equipos preguntan: *"¿qué le digo al agente, en este chat, para que esto no pase?"*. Es la pregunta equivocada. Lleva a instrucciones efímeras que se evaporan al cerrar la sesión, microcorrecciones que no quedan registradas, una espiral de prompt-tweaking que no acumula nada.

La pregunta correcta es esta:

> **¿Qué le falta al harness para que este fallo sea imposible, o para que se detecte automáticamente cuando ocurra?**

La respuesta puede ser muchas cosas: una sección nueva en AGENTS.md o en un doc del repo, una plantilla de plan actualizada, un linter nuevo, un sensor inferencial que revise un patrón concreto, una restructuración del contexto que el agente recibe, una capa de aislamiento que faltaba. Cualquiera de estas es válida — incluidas las que tocan prompts versionados, porque **un prompt commiteado al repo es arnés; un prompt en el chat que desaparece, no**. Lo que distingue una respuesta buena de una mala no es si toca prompts o código, sino si la lección queda registrada en algún sitio que el agente vaya a leer la próxima vez. Y lo importante: una vez que la implementas, **ese fallo concreto no vuelve a ocurrir**. Nunca. En miles de ejecuciones futuras del agente.

Es el mismo cambio mental que distingue a un equipo de plataforma maduro de uno que sigue apagando incendios. No "cómo enseño a este equipo a no romper esto en esta conversación", sino "cómo hago imposible (o trivialmente detectable) que rompan esto en cualquier conversación futura". La diferencia es que con agentes el ROI es brutal: cada hora invertida en el arnés mejora todas las ejecuciones futuras, y las ejecuciones futuras son muchas más de las que tendrías con un equipo humano.

## El rigor no desaparece, se reubica

Hay una observación de Chad Fowler que conviene tener delante antes de empezar a desmantelar prácticas. Cada vez que el software ha dado un salto importante — lenguajes dinámicos, extreme programming, despliegue continuo — alguien ha anunciado "ya no hace falta tanto rigor". Y cada vez se ha equivocado. **El rigor no desaparece; se traslada a otro sitio**. Los lenguajes dinámicos lo movieron al test suite; XP lo movió a la integración continua; el despliegue continuo lo movió a la observabilidad y a la reversión automática.

Con agentes pasa lo mismo. El rigor sale de "escribir cada línea con cuidado" y de "revisar humano-a-humano cada PR", y entra en otros dos sitios:

- **Especificación de la intención.** Lo que antes eran criterios informales en un ticket ahora tiene que ser legible mecánicamente, porque es lo que el agente va a interpretar. La imprecisión en el ticket se convierte en imprecisión en el código.
- **Evaluación verificable.** Si no puedes evaluar mecánicamente si la salida del agente es correcta, no tienes arnés, tienes una ruleta.

La heurística que vale la pena adoptar: cuando algo parezca dejar ir el rigor, busca dónde se reubicó. Si no lo encuentras, preocúpate. Las pertenencias quedan en otro sitio o se han perdido.

Esto explica una observación que vale la pena tener delante: **los repos con disciplina codificada funcionan mejor con agentes que los repos con disciplina cultural**, aunque ambos produzcan código humano de calidad equivalente. Disciplina codificada significa linters agresivos, schemas validados en el borde, tests rápidos, docs versionadas. Disciplina cultural significa revisiones cuidadosas, seniors atentos, convenciones no escritas. La razón es directa: el agente no puede acceder a la cultura. Solo ve lo que el código deja claro mecánicamente.

## Lo que el agente no puede ver, no existe

Hay una frase del equipo de Codex de OpenAI que vale la pena pintar en la pared: **"lo que el agente no puede ver, no existe"**. Es una afirmación literal sobre cómo opera un agente de codificación, y tiene una consecuencia que cambia cómo ves el repositorio.

En equipos humanos, el conocimiento vive distribuido: en el README sí, pero también en Slack, en un Google Doc enlazado en algún correo, en la cabeza del senior, en una conversación de pasillo. Esta distribución funciona — mal, pero funciona — porque los humanos sabemos preguntar y cruzar fuentes.

Un agente no. El agente solo ve lo que está en su contexto efectivo en el momento de actuar. Si la decisión de "no toques este módulo, lo vamos a deprecar" vive solo en un Slack del 14 de marzo, para el agente esa decisión no existe. No es que la ignore: es que no es accesible.

La consecuencia es liberadora una vez que la aceptas: **todo lo que quieras que el agente sepa tiene que estar materializado en el repo o accesible mecánicamente desde él**. No "documentado en algún sitio". Materializado, en markdown, indexado, mantenido por linters y por agentes recurrentes que detectan documentación obsoleta.

Y aquí ocurre un efecto secundario que casi nadie anticipa: **al hacer el repo legible para un agente, lo haces enormemente más legible para humanos también**. La inversión en developer experience histórica rinde dividendos para el agente, y la inversión en agente rinde dividendos para los humanos que se incorporan al equipo. Es el bucle virtuoso que Stripe resume mejor: *"lo que es bueno para humanos es bueno para agentes"*.

## Tres modos de delegación, y solo uno escala

Kief Morris articula una distinción que evita la mitad de las discusiones improductivas que hay sobre supervisión humana. Hay tres modos de meter humanos en el bucle con agentes:

- **Humans Outside the Loop ("vibe coding").** El humano define el resultado y mira qué sale. Funciona para prototipos. Falla en producción seria.
- **Humans In the Loop.** El humano revisa cada artefacto, línea por línea. Es aritméticamente insostenible cuando el rendimiento del agente sube. El humano se vuelve cuello de botella.
- **Humans On the Loop.** El humano supervisa el sistema, no los artefactos. Cuando algo va mal, **no corrige el artefacto: modifica el sistema que produjo el artefacto**.

El único modo que escala a meses vista es "on the loop". El "in the loop" es el régimen al que llega un equipo conservador por defecto y donde se queda atascado. La transición saludable es de "in" hacia "on", invirtiendo en el arnés hasta que la supervisión del sistema sea posible. La transición a "outside" — saltarse el arnés directamente — es donde más equipos se queman intentando ahorrarse las fases intermedias.

## El tipo de trabajo que esto pide

Cuando aceptas todo lo anterior, el trabajo del ingeniero cambia. Sigues siendo el centro del sistema, pero el centro tiene una forma distinta:

- Antes escribías código. Ahora **diseñas entornos donde el código correcto puede escribirse a sí mismo**.
- Antes revisabas PRs. Ahora **diseñas las reglas, sensores y revisores que revisan PRs**.
- Antes entendías el dominio para implementarlo. Ahora **entiendes el dominio para especificarlo de forma que el agente pueda implementarlo correctamente**.
- Antes arreglabas fallos. Ahora **diagnosticas por qué el agente cometió el fallo y modificas el arnés para que no vuelva a pasar**.

Las habilidades que ganan valor son las que ya distinguían a los buenos seniors: gusto técnico, pensamiento sistémico, claridad al especificar, juicio sobre cuándo invertir en infraestructura. Las que pierden valor relativo son las puramente artesanales: tipear rápido, memorizar APIs, dominar syntax exótica.

Hay un momento incómodo en esta transición — *si ya no escribo código, ¿qué soy?* — y conviene nombrarlo en lugar de fingir que no existe. La respuesta sincera es que sigues siendo ingeniero, pero el medio ha cambiado. Antes producías código; ahora produces **las condiciones que hacen posible que el código correcto exista**. Es ingeniería de un orden más alto, no menos.

## Por dónde empezar el lunes

Si has llegado hasta aquí y te interesa moverte de la teoría a la práctica, hay una secuencia que funciona mejor que la mayoría de alternativas:

1. **Empieza por el aislamiento.** Sandboxes desechables, baratos de crear, baratos de tirar. Es la inversión que más multiplica todas las demás. Sin esto, los siguientes pasos rinden la mitad.
2. **Materializa el contexto en el repo.** Lo importante que vive en Slack, Google Docs, o cabezas, mudado a markdown versionado. AGENTS.md como índice de 100 líneas, no como enciclopedia de mil.
3. **Codifica los invariantes que más te importan.** Linters custom con mensajes dirigidos al agente. Direcciones de dependencia validadas mecánicamente. Schemas en el borde.
4. **Promueve cada fallo recurrente al arnés.** Si el fallo es obviamente sistémico desde el principio, lo conviertes en guía o sensor de inmediato. Si parece una rareza, lo arreglas con el mínimo gasto y lo anotas — pero la segunda vez no hay decisión que tomar: paras y lo promueves, porque ahora ya sabes que es un patrón. No hay tercera vez bien gestionada.
5. **Diseña el flujo de PR para que la mayoría se revisen agente-a-agente**, y reserva la atención humana para los casos donde el criterio aporta valor real.

No tienes que hacer todo a la vez. Empieza con los pasos 1 y 2; los demás caen por gravedad cuando los dos primeros están en su sitio.

## El cierre

La línea que separa a los equipos que están haciendo cosas notables con agentes de los que siguen frustrados es una línea de inversión en infraestructura. No de talento, no de modelo, no de presupuesto: de **decisión de tratar al agente como un sistema que se ingenía, no como una herramienta a la que se le pide bien**.

Los modelos van a seguir mejorando. Eso es seguro. Lo que está menos claro es cuántos equipos van a aprovecharlos. Los que estén invirtiendo en su arnés mientras los demás esperan al próximo modelo van a llegar al próximo modelo con un multiplicador encima. Los que esperen al próximo modelo van a descubrir que el problema no era el modelo y van a volver a esperar al siguiente.

Si esta lectura te ha resonado, los **capítulos 1 a 11 de la guía** desarrollan cada una de las ideas con más detalle: la mentalidad (cap. 1–2), los pilares de guías y sensores (cap. 3–4), la práctica concreta de entornos, contexto, arquitectura y flujo de PR (cap. 5–8), y el mantenimiento sostenible y los anti-patrones (cap. 9–11). Léelos como ensayos independientes; no hace falta orden. Lo que sí hace falta es empezar.

---

*Este texto sintetiza ideas de Birgitta Böckeler (Thoughtworks), Chad Fowler, Geoffrey Huntley, Kief Morris, Ryan Lopopolo (equipo Codex de OpenAI) y Alistair Gray (equipo Leverage de Stripe, Minions). El término "harness" en este sentido se popularizó en torno a LangChain. URLs en la sección de fuentes del [README de la guía](README.md#fuentes).*
