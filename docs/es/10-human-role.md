# 10. Dónde sigue importando el humano

Después de nueve capítulos describiendo cómo automatizar todo lo automatizable, conviene parar y hacer la pregunta opuesta: **¿qué tiene que seguir haciendo el humano?**. La respuesta no es "supervisar al agente", aunque eso es lo que muchos equipos terminan haciendo por inercia. La respuesta correcta requiere entender que el papel del ingeniero cambia, no se reduce.

## Dónde está el humano: un recordatorio rápido

El Cap. 0 introdujo los tres modos de delegación que articula Kief Morris — *outside the loop* (vibe coding), *in the loop* (supervisión por cada artefacto) y *on the loop* (supervisión del sistema) — y argumentó que "on the loop" es el único régimen que escala: el humano no inspecciona cada salida, sino que modifica el sistema que las produce cuando algo va mal.

Damos esa discusión por cerrada. La pregunta que organiza este capítulo es la siguiente: **si estás "on the loop", ¿qué haces exactamente con tu tiempo?** Porque "supervisar el sistema" es una frase bonita que, sin desglosar, deja a la mitad de los ingenieros sin saber qué hacer el lunes por la mañana.

## Los dos ciclos: why y how

Otra distinción de Morris que ayuda a localizar el rol humano: hay dos ciclos en juego, y conviene no confundirlos.

- El **ciclo "why"** es donde se transforma una idea en software que importa. Implica entender qué se quiere lograr, para quién, bajo qué restricciones, con qué compromisos, en qué orden. Es trabajo profundamente humano y no se delega.
- El **ciclo "how"** es donde se producen los artefactos intermedios — código, tests, configs, docs — que materializan el "why". Es el ciclo donde los agentes brillan y donde la mayor parte de esta guía aplica.

La trampa más común es ocupar tiempo humano en el "how" y descuidar el "why". Un equipo que delega bien el "how" libera el cuello de botella escaso (atención humana) para invertirlo donde realmente importa (entender qué construir y por qué). Un equipo que se queda atrapado en revisar artefactos del "how" no solo es lento; está usando mal a sus humanos.

## Lo que cambia, no lo que desaparece

Böckeler insiste en un punto que es fácil malinterpretar: el harness no elimina al humano, lo *redirige*. Los desarrolladores aportan tres cosas que el agente, por construcción, no tiene:

- **Experiencia implícita:** patrones aprendidos en otros proyectos, intuiciones formadas en errores anteriores, sensación de "esto va a doler en seis meses" que no se deriva del código que estás mirando.
- **Conciencia organizacional:** saber qué quiere el equipo de producto, qué le importa al CFO, qué decisiones tienen contexto político, qué stakeholder se va a oponer.
- **Responsabilidad social:** alguien tiene que firmar cuando algo va a producción que afecta a usuarios reales. Esa firma no la puede dar un agente.

Estas tres cosas no son automatizables, no porque el modelo no sea capaz, sino porque no son problemas técnicos. Son problemas humanos.

## El nuevo trabajo

El Cap. 0 ya enumeró el cambio — de escribir código a diseñar entornos, de revisar PRs a diseñar los revisores, de implementar el dominio a especificarlo, de arreglar fallos a diagnosticar por qué el agente los cometió — y lo resumió con la frase de OpenAI: *"las personas dirigen, los agentes ejecutan"*. Es meta-ingeniería, y la descripción del puesto se parece más a "ingeniero de plataforma para colaboradores no humanos" que a "programador".

Lo que el Cap. 0 no hizo es desglosarlo al nivel de "qué ocupa tus horas". Eso es lo siguiente.

## Las cuatro intervenciones humanas que importan

Cuando filtras todo el ruido y miras lo que los humanos efectivamente hacen en equipos maduros con agentes, aparecen cuatro categorías de intervención. Las tres primeras son las que aportan valor; la cuarta es la que conviene minimizar.

### 1. Especificar la intención

El agente puede implementar casi cualquier cosa. Lo que no puede es decidir qué *vale la pena* implementar. Convertir un problema vago en un objetivo accionable, con criterios de aceptación verificables, sigue siendo trabajo humano. Y es probablemente el trabajo más palanca: una especificación clara ahorra horas de bucles mal dirigidos.

La forma concreta varía — puede ser un plan de ejecución versionado, un ticket bien escrito, un prompt detallado — pero el contenido tiene que tener tres cosas: qué se quiere lograr, cómo se sabrá que está hecho, y qué *no* hacer. La tercera es la que más a menudo se omite y la que más drift previene.

### 2. Calibrar el harness

Cuando el agente falla repetidamente, alguien tiene que hacer la pregunta importante: ¿qué le falta al harness? Esto requiere entender el fallo a un nivel que no es solo "el código es incorrecto", sino "el agente no tenía la información que necesitaba" o "el sensor no detectó esto" o "la convención no estaba codificada". El humano hace este diagnóstico y promueve la lección al arnés — como un nuevo lint, una nueva guía, un nuevo agente revisor, o una sección nueva en docs.

Este es el trabajo donde un senior multiplica más su impacto. Cada hora invertida aquí se acumula sobre miles de ejecuciones futuras del agente. Es la inversión con mejor ROI del equipo.

### 3. Decidir lo irreversible

Hay decisiones cuyo coste de error es asimétrico: deprecar una API pública, cambiar un schema de base de datos en producción, modificar un contrato con un proveedor, aceptar un compromiso de seguridad. En estos casos no quieres velocidad, quieres deliberación. Y la deliberación necesita criterio, contexto organizacional, y responsabilidad — las tres cosas humanas que enumeramos al principio.

La regla operativa: **cuanto más irreversible la decisión, más humano debe haber en el bucle, más temprano**. No al final, revisando un PR que ya se escribió. Al principio, eligiendo si hacerlo y cómo.

### 4. Revisar lo que ya se filtró

Esta es la categoría que conviene minimizar, no eliminar. El humano revisa cuando:

- Un agente revisor escala explícitamente porque no está seguro.
- El cambio toca un área marcada como sensible.
- El ingeniero responsable elige mirar para mantener su modelo mental al día.

Lo importante: **no es la primera línea de defensa**. Si el humano está siendo el filtro principal de calidad, los sensores están incompletos. La revisión humana es la red de seguridad final, no el mecanismo primario.

## Lo que conviene parar de hacer

Hay tres hábitos que muchos equipos arrastran del flujo pre-agente y que conviene desmontar conscientemente:

**Revisar cada PR.** Como ya dijimos en el capítulo 8, no escala. Más importante: **da una falsa sensación de control**. Si revisas todo en piloto automático, no estás revisando nada bien. Mejor revisar el 10% que importa con atención que el 100% por encima.

**Discutir estilo.** Si la salida del agente no respeta una preferencia estilística y no la captura un lint, tienes dos opciones: codificar la preferencia como lint, o aceptar la salida. Discutirlo en cada PR es desperdicio puro.

**Pedirle al agente que "haga las cosas como tú las harías".** El agente no es tú. Lo que quieres es que las cosas que hace satisfagan los invariantes que importan. Si los invariantes están bien capturados, el "estilo" del agente es irrelevante. Si no lo están, codificarlos es más útil que pedir.

## La pregunta de identidad

Hay un momento incómodo que casi todos los ingenieros experimentan al adoptar este modelo: si ya no escribo código, ¿qué soy? La respuesta sincera es que sigues siendo ingeniero, pero el medio del que sale tu trabajo ha cambiado. Antes producías código; ahora produces **las condiciones que hacen posible que el código correcto exista**. Es ingeniería de un orden distinto, no menos.

Las habilidades que se vuelven valiosas son las que ya valoraban los buenos seniors: gusto técnico, capacidad de pensar en sistemas, claridad al especificar, habilidad para diseñar interfaces (humanas o no humanas), juicio sobre cuándo invertir en infraestructura. Las habilidades que pierden valor relativo son las puramente artesanales: tipeo rápido, memorización de APIs, dominio de syntax exótica.

No es un mal trade en general. Pero requiere un realineamiento de identidad profesional que no debería minimizarse. Equipos que ignoran esto tienen ingenieros descontentos haciendo "el trabajo nuevo" con resentimiento. Equipos que lo abordan explícitamente — "este es el nuevo trabajo, esta es la palanca, estos son los nuevos valores" — convierten a sus seniors en multiplicadores en vez de cuellos de botella.

## El cierre

El humano sigue siendo el centro del sistema. Pero el centro ha cambiado de forma. Ya no estás en el centro porque escribas todo el código; estás en el centro porque diseñas el sistema dentro del cual el código se escribe a sí mismo, y porque cuando ese sistema falla, eres tú quien decide qué falló y qué arreglarlo significa.

Las máquinas pueden generar cantidad. Solo los humanos pueden decidir qué cantidad merece existir, y bajo qué condiciones se le permite existir. Eso no se va a automatizar pronto. Y mientras siga siendo verdad, el ingeniero sigue siendo el centro — solo en un nivel de abstracción más alto del que está acostumbrado.
