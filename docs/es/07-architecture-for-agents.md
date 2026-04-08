# 7. Arquitectura para agentes

Este es el capítulo que más resistencia genera en equipos con experiencia. La idea — que la arquitectura del software debería diseñarse pensando en cómo la van a leer y modificar los agentes, no solo los humanos — suena a poner al agente por delante del humano. Pero cuando empiezas a operar agentes en serio, la observación se vuelve difícil de evitar: **bases de código con invariantes mecánicamente verificables y límites estrictos producen mejores resultados de agente que bases de código "elegantes" con disciplina implícita**. No es una preferencia estética. Es una propiedad medible.

## La observación incómoda

OpenAI lo dice explícitamente y vale la pena citarlo: *"Es el tipo de arquitectura que sueles posponer hasta que tienes cientos de ingenieros. Con agentes de codificación, es uno de los prerrequisitos iniciales: las restricciones son lo que permite la velocidad, sin deterioros ni desviaciones arquitectónicas."*

Léelo otra vez. Lo que normalmente se considera over-engineering — capas estrictas, validación estructural, linters custom para cada convención — se convierte en *condición habilitadora* cuando el escritor del código es un agente. La razón es simple: las restricciones son la única forma en que el agente "sabe" qué es la arquitectura. No tiene gusto, no tiene historia, no tiene memoria de la decisión que tomamos hace dos años. Tiene lo que el código deja claro mecánicamente.

## Capas como invariante, no como sugerencia

OpenAI describe un modelo concreto que vale la pena entender, no para copiarlo literalmente, sino para captar la forma. Cada dominio de negocio se divide en capas fijas con direcciones de dependencia validadas:

**Types → Config → Repository → Service → Runtime → UI**

Las dependencias solo van hacia adelante. Los cross-cutting concerns (auth, telemetría, feature flags) entran exclusivamente vía una capa explícita de **Providers**. Cualquier import que viole esto es un error de build.

La parte importante no es la lista exacta de capas. Es que **el modelo arquitectónico es ejecutable**. Un linter custom (escrito por el propio agente, además) lee los imports y rechaza los que cruzan límites. No hay debate, no hay "esta vez es excepción", no hay revisión humana decidiendo caso por caso. La arquitectura está en el código que valida el código.

Para un equipo humano esto puede sentirse rígido. Para un agente, es liberador: ya no tiene que adivinar el patrón, lo descubre intentando algo y leyendo el error. Y los errores de los lints custom pueden contener instrucciones de remedio dirigidas explícitamente al agente — "esta capa solo puede importar de X y Y; mueve esta función a la capa Z" — convirtiendo cada lint en un tutor pasivo.

## Dependencias "aburridas"

Otra observación de OpenAI: prefieren librerías "aburridas" — modulares, con APIs estables, bien representadas en los datos de entrenamiento del modelo. La razón no es nostalgia. Es que esas librerías son **mecánicamente más fáciles de razonar para el agente**: hay más ejemplos en el corpus, las convenciones son estables, el comportamiento es predecible.

Hay un caso límite que merece atención porque rompe intuiciones: a veces es más barato que el agente reimplemente un subset de una librería que integrarla. OpenAI da un ejemplo concreto: en lugar de usar `p-limit` (un helper minúsculo de concurrencia), Codex implementó su propia versión integrada con su instrumentación de OpenTelemetry, con 100% de cobertura. ¿Por qué? Porque la librería externa era una caja negra cuyo comportamiento opaco generaba más fricción en el bucle que el código que la reemplaza.

Esto va contra el reflejo "no reinventes la rueda", y es importante calibrarlo. La regla útil es: **si el agente puede leer, modificar y validar el código completo de algo en su contexto efectivo, eso es una ventaja arquitectónica neta**. Las dependencias externas que el agente no puede inspeccionar son fricción permanente. Las que puede inspeccionar son colaboradores. La diferencia se nota a meses vista.

Esto no significa reimplementar React. Significa que cuando estés a punto de meter una utility de 200 líneas como dependencia transitiva, hagas una pausa y te preguntes si vale más escribirla como código del proyecto.

## Estilo como invariante

OpenAI menciona "invariantes de estilo" enforcadas estáticamente: registro estructurado obligatorio, convenciones de nomenclatura para schemas y tipos, límites de tamaño de archivo, requisitos de fiabilidad específicos por plataforma. No "style guide en un Confluence". Lints. En el código. Bloqueando merge.

La diferencia es decisiva. Un style guide humano descansa en que el revisor lo recuerde y lo aplique. Un lint no descansa en nada: o está, o no está. Y dado que el agente puede generar miles de líneas por día, la única forma de mantener consistencia es renunciando a que algún humano la mantenga.

Un detalle táctico importante: **la salida del lint es para el agente, no solo para el humano**. Cuando escribas un lint custom, escribe el mensaje de error pensando que lo va a leer un agente: explica el patrón correcto, da un ejemplo, indica dónde mirar. Un lint con mensaje "incorrect import" es inútil. Un lint con mensaje "este módulo solo puede importar de los Providers; mueve la dependencia a `providers/auth.ts` o reescribe usando el provider existente" es enseñanza activa.

## Límites estrictos, autonomía local

El equilibrio que OpenAI articula merece copiarse literalmente: *"liderar como una gran organización de plataformas de ingeniería: estableciendo límites de manera centralizada y permitiendo la autonomía a nivel local. Te importan mucho los límites, la corrección y la reproducibilidad. Dentro de esos límites, permites a los equipos (o agentes) una libertad considerable en cómo se expresan las soluciones."*

En la práctica esto significa:

- **Los límites son sagrados:** direcciones de dependencia, contratos de schemas, observabilidad obligatoria, gestión de errores en bordes. Bloqueas merge.
- **Lo de dentro es libre:** cómo se nombra una variable interna, cómo se estructura una función privada, qué patrón concreto usa para iterar una lista. No es asunto del lint ni de la revisión.

Hay una consecuencia que a muchos equipos les cuesta aceptar: **el código resultante no siempre va a coincidir con tus preferencias estilísticas**. Y no pasa nada. Mientras sea correcto, mantenible y legible para futuras ejecuciones del agente, cumple el estándar. Discutir sobre el nombre de una función interna que un lint no captura es, en este contexto, gasto puro de atención humana — y la atención humana es ahora el recurso escaso.

## La paradoja del brown-field

Hay una observación incómoda que merece nombrarse antes de hablar de tácticas: **los proyectos brown-field son a la vez donde más se necesita un buen harness y donde más cuesta construirlo**. Las dos cosas a la vez, y por las mismas razones.

**Por qué se necesita más.** Una base de código grande, vieja, con años de crecimiento orgánico, módulos heredados de tres equipos distintos y zonas que nadie del equipo actual entiende del todo, es exactamente el tipo de entorno donde un agente puede multiplicar el rendimiento del equipo — *si* puede operar con seguridad. Un greenfield es trivial para cualquier ingeniero senior; un brown-field es donde el agente, bien dirigido, te ahorra semanas de arqueología cada mes. El ROI potencial es máximo.

**Por qué cuesta más.** Las mismas propiedades que hacen valioso al agente en brown-field son las que hacen difícil construirle un harness:

- La disciplina vive como cultura, no como código. Las "reglas" están en la cabeza de los seniors, en revisiones de PR pasadas, en hilos de Slack de hace dos años. Materializarlas en lints es trabajo arqueológico previo a cualquier inversión en el harness.
- La arquitectura es heterogénea. Distintos dominios siguen distintas convenciones, los límites entre capas existen en algunos sitios y en otros no. No puedes escribir *un* lint de direcciones de dependencia: tienes que escribir *N* lints, uno por zona coherente, y aceptar que algunas zonas no son codificables hasta que las refactorices primero.
- La entropía es real y se resiste. Cada regla nueva que introduces choca con código que ya la viola, y tienes que decidir caso por caso si es excepción legítima o deuda que toca pagar. No es trabajo del agente; es trabajo humano de calibración previa.
- El equipo está acostumbrado a las imperfecciones. Lo que en un greenfield sería un bug evidente, en brown-field es "siempre ha sido así". Convertir eso en invariantes obligatorios genera fricción social, no solo técnica.

**Y hay un agravante: el agente amplifica la entropía existente.** Esto es importante porque convierte la dificultad en bucle de retroalimentación. Un agente aprende del código que tiene delante: si la base está llena de malas prácticas, las imita, las propaga más rápido de lo que un humano lo haría, y refuerza la sensación de que "así se hace aquí" porque el código nuevo se parece al viejo. Sin un sensor que detecte el patrón malo o una guía que lo prohíba, el agente no tiene cómo distinguir entre convención sana y deuda heredada — para él, todo lo que ve en el repo es "lo que el equipo hace". El bucle es silencioso: parece que el agente está siendo productivo y consistente, y lo es; solo que la consistencia es con lo equivocado. En un brown-field sin harness, **introducir un agente acelera la entropía en lugar de combatirla**. Esa es la razón más fuerte para no posponer el harness en este tipo de bases.

**La consecuencia operativa**: en brown-field, la inversión en harness no es opcional — es prerrequisito. Y no se puede hacer en una semana. La pregunta no es "¿queremos el harness?" sino "¿qué velocidad de construcción del harness podemos sostener sin parar de entregar?". La respuesta sana suele ser: una regla nueva por semana, una zona codificada al mes, ningún big-bang. Lo que el siguiente apartado describe vale para ambos casos, pero **en brown-field es la única forma que funciona**.

## Cómo introducir esto en un repo existente

Hacer un big-bang sobre un repo existente es mala idea. Lo que funciona:

1. **Empieza por las capas que ya están claras.** Si tu repo ya tiene una distinción razonable entre dominios o módulos, escribe el lint que la haga obligatoria. No estás imponiendo arquitectura nueva, estás congelando la que ya tienes.
2. **Promueve una regla cada vez.** Un lint nuevo a la semana, no doce a la vez. Cada lint nuevo es una guía, y las guías nuevas requieren ajuste del agente y del equipo.
3. **Empieza por warning, sube a error.** Un lint nuevo en modo warning te enseña qué tan ruidoso va a ser. Cuando el ruido baje a cero, súbelo a error.
4. **Escribe el linter con el agente.** Esto es metanivel y vale la pena: que el propio agente escriba sus restricciones (con tu supervisión) hace explícita la intención y produce código que el propio agente puede leer y modificar después.

## El cambio de horizonte

La conclusión que vale la pena interiorizar: cuando inviertes en arquitectura mecánicamente verificable, no estás haciéndolo para "hoy". Lo estás haciendo para los próximos 18 meses, en los que esa misma arquitectura va a sostener decenas de miles de modificaciones del agente sin desgaste. Es la inversión con mejor ROI a meses vista de las que aparecen en esta guía. También es la más fácil de subestimar a corto plazo, porque su beneficio se acumula lentamente y se nota cuando un equipo equivalente sin esta inversión ya está hundido en drift.
