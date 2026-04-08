# 6. El contexto como sistema de registro

Hay una frase del equipo de Codex de OpenAI que vale la pena pintar en la pared del equipo: **"lo que el agente no puede ver, no existe"**. Es una afirmación literal sobre cómo opera un agente de codificación, y tiene consecuencias directas sobre dónde tiene que vivir el conocimiento de un proyecto. La respuesta corta: en el repo, versionado, mecánicamente accesible. La respuesta larga es este capítulo.

## El cambio de mentalidad

En equipos humanos, el conocimiento vive distribuido en muchos lugares: en el README, sí, pero también en Slack del miércoles pasado, en un Google Doc enlazado en algún correo, en la cabeza del senior, en un Linear ticket de hace seis meses, en una conversación de pasillo. Esta distribución funciona — mal pero funciona — porque los humanos sabemos preguntar, cruzar fuentes, y usar contexto implícito.

Un agente no. El agente solo ve lo que está en su contexto efectivo en el momento de actuar. Si la decisión de "no toques este módulo, lo vamos a deprecar" vive solo en un Slack del 14 de marzo, para el agente esa decisión no existe. No es que la ignore: es que no es accesible.

La consecuencia es incómoda y liberadora a la vez: **todo lo que quieras que el agente sepa tiene que estar materializado en el repo o accesible mecánicamente desde él**. No "documentado en algún sitio". No "mencionado en una reunión". Materializado, en markdown, indexado, mantenido.

## El error del "AGENTS.md gigante"

La primera reacción de muchos equipos al descubrir esto es razonable: meter todo lo importante en un AGENTS.md enorme. Es exactamente la trampa que OpenAI describe haber probado y abandonado. Cuatro razones por las que falla:

**1. El contexto es un recurso escaso.** Un manual de 1.000 líneas eclipsa la tarea, el código relevante y las herramientas. El modelo tiene dos opciones: ignorar restricciones clave, u optimizar las equivocadas. Las dos son malas.

**2. Si todo es importante, nada lo es.** Cuando todo en el AGENTS.md está en negrita, el agente no tiene cómo priorizar. Acaba aplicando reglas locales sin entender la intención global.

**3. Se descompone instantáneamente.** Un manual monolítico se convierte en un cementerio de reglas obsoletas. El agente no sabe qué sigue siendo verdad, los humanos dejan de mantenerlo, y lentamente se vuelve un riesgo activo.

**4. No es verificable.** Un bloque grande de prosa no se presta a checks automáticos: no puedes validar cobertura, freshness, propiedad o referencias cruzadas. El drift es inevitable.

La conclusión de OpenAI es elegante: **trata el AGENTS.md como un índice, no como una enciclopedia**. ~100 líneas, apuntando a sitios donde vive cada cosa.

## El repo como sistema de registro

Lo que va al repo, organizado, es lo siguiente. Esto no es prescriptivo — el detalle varía por equipo — pero la forma se repite:

- **Documentos de diseño versionados**, indexados, con un estado de verificación. Nada de "doc enlazado en Notion, Confluence o Google Docs". El doc vive en el repo, junto al código que describe.
- **Planes de ejecución como artefactos de primera clase.** Para cambios pequeños, planes ligeros y efímeros. Para trabajo complejo, planes versionados con estado: activos, completados, abandonados, con razones. Esto le permite al agente ver no solo qué está pasando ahora, sino qué se intentó antes y por qué se descartó. Es una herramienta de memoria a meses vista.
- **Tracker de deuda técnica.** Un archivo, en el repo, que enumera lo que está mal a propósito. Sin esto, el agente "limpia" cosas que no debería tocar, o ignora cosas que debería arreglar. Con esto, las dos categorías son legibles.
- **Especificaciones de producto.** No las épicas de Linear, Jira o GitHub Issues. Las specs *como markdown en el repo*, junto al código que las implementa. Si el ticket es la única fuente, el agente solo ve lo que está en el ticket cuando se lo pasas.
- **Referencias externas en formato `llms.txt`.** Documentación de librerías externas, normas internas de seguridad, guías de la plataforma. Importadas al repo, no enlazadas. Una URL en un AGENTS.md es una promesa rota; un archivo en `references/` es una fuente.
- **Mapa de arquitectura.** Capas, dominios, responsabilidades, direcciones de dependencia. Más sobre esto en el capítulo 7.
- **Score de calidad por dominio.** OpenAI mantiene un `QUALITY_SCORE.md` por dominio que registra las lagunas conocidas. Es a la vez documentación y backlog priorizable mecánicamente.

La estructura concreta importa menos que el principio: **carpetas en el repo, no servicios externos; markdown, no PDFs; mantenidos por linters y agentes recurrentes, no por buena voluntad humana**.

## Cómo hacer que esto no se descomponga

El miedo razonable es: "si meto todo esto en el repo, en seis meses estará obsoleto". Es un miedo correcto, y sin solución se vuelve realidad. Lo que funciona:

**Linters específicos para la documentación.** Validan que los enlaces internos no estén rotos, que los nombres de archivos referenciados existan, que los planes activos no lleven más de N días sin update, que los documentos tengan los campos de metadata mínimos. Es trabajo aburrido y multiplica el ROI de todo lo demás.

**Agentes de doc-gardening recurrentes.** Esto es un patrón que aparece tanto en OpenAI como, implícitamente, en el flujo de Stripe. Un agente que escanea documentación contra el comportamiento real del código y abre PRs cuando detecta divergencia. No "elimina" la doc obsoleta automáticamente: la marca y deja que un humano (o el agente que la modificó) decida.

**Promueve la regla al código.** Cuando una pieza de documentación describe un patrón que debería ser obligatorio, conviértela en lint. El doc deja de ser la fuente y se vuelve la explicación; el lint es el enforcement. La regla original de OpenAI: *"cuando la documentación no basta, promovemos la regla al código"*.

## El test definitivo

Cuando dudes si una pieza de información debe ir al repo o puede quedarse fuera, hazte esta pregunta: **si un nuevo ingeniero se incorporara mañana sin acceso a Slack, sin acceso a Google Docs y sin poder hablar con nadie del equipo, ¿podría ser productivo solo con lo que hay en el repo?**

Si la respuesta es no, esa información es invisible para el agente también. Y en un equipo serio que opera con agentes, ese nuevo "ingeniero" se incorpora cien veces al día. Cada ejecución de agente es una incorporación desde cero.

## El upside oculto

Hay un efecto secundario que casi nadie anticipa: **al hacer el repo legible para un agente, lo haces enormemente más legible para humanos también**. El doc-gardening te beneficia. Los planes versionados te benefician. El tracker de deuda te beneficia. Los humanos que se incorporan al equipo se incorporan más rápido. Los seniors dejan de ser cuellos de botella de contexto. Lo que parecía una concesión a la máquina termina siendo, en buena parte, ingeniería de plataforma para el propio equipo.

Stripe lo dice de forma directa, y vale la pena cerrar con su frase: *"lo que es bueno para humanos es bueno para agentes"*. La inversión histórica en developer experience rinde dividendos también para el agente, y la inversión en agente rinde dividendos para el developer experience. El bucle es virtuoso si lo dejas serlo.

## ¿Y si uso un MCP server en lugar de meterlo en el repo?

Esta es la pregunta que aparece en cuanto un equipo tiene buena documentación viva en Notion, Confluence o Google Docs y se enfrenta al coste de moverla. La intuición es razonable: si existe un MCP server que expone esa documentación al agente, ¿por qué duplicarla en el repo? El agente la consulta en vivo, los humanos siguen editando donde editan, cero sincronización, cero drift. Parece la solución elegante.

La respuesta no es "no lo hagas". Es "entiende qué precio estás pagando y dónde aplica". Hay cuatro estrategias posibles para que un agente acceda a documentación que no nace en el repo, y cada una paga un precio distinto:

### Las cuatro opciones, en orden de "pureza"

**1. Doc solo en el repo (markdown versionado).** Una sola fuente. El humano edita en el editor o por GitHub web. El agente lee del filesystem. Cero ambigüedad, cero drift, todo auditado por commit. Es lo que el resto de este capítulo defiende. **Coste:** los no-ingenieros (PMs, diseñadores, legal, soporte) tienen que aprender git o usar la web de GitHub, y pierdes features de los wikis modernos (comentarios inline, mentions, embeds, plantillas ricas). Para muchos equipos, políticamente inviable.

**2. Importación puntual (copiar al repo cuando se cierra el doc).** El doc nace en Notion, los humanos colaboran ahí, y cuando se considera "estable" se exporta a markdown y se mete al repo. **Coste:** funciona para docs *terminados* (RFCs cerrados, post-mortems, decisiones tomadas). Para docs *vivos* es trampa: el día siguiente Notion ya divergió y nadie va a volver a exportar.

**3. Sincronización automática (mirror Notion → repo).** Un job o webhook detecta cambios en Notion y los exporta a markdown en el repo de forma continua. Humanos siguen editando donde ya editan; el agente lee del repo (rápido, determinista, versionado). Lo mejor de los dos mundos *en apariencia*. **Coste:** la sincronización tiene ventana — entre push humano y pull al repo, el repo está stale. Más infra que mantener (hooks, conversores, manejo de errores). La conversión de Notion/Confluence a markdown pierde matices: bases de datos embebidas, comentarios, plantillas dinámicas. Cuando el sync falla, falla en silencio y el agente sigue leyendo datos viejos sin saber que son viejos.

**4. MCP server con búsqueda semántica en vivo.** El agente, en tiempo de ejecución, llama a un MCP que consulta Notion (o lo que toque) y devuelve los fragmentos más relevantes para su query. Cero duplicación, cero sincronización, cero drift. La doc vive donde nació. **Coste:** es donde más se subestiman los problemas. Vamos a desarrollarlo.

### Por qué un MCP server no es un sustituto del repo (para lo crítico)

La búsqueda semántica vía MCP introduce varias propiedades que son aceptables para algunos usos y desastrosas para otros:

**No determinista.** Dos consultas idénticas pueden devolver resultados distintos. La búsqueda semántica depende del embedding model, del ranking, del corpus indexado en ese momento. El agente puede leer el doc correcto en una ejecución y *no leerlo* en la siguiente. Para un sensor o una invariante, esto es inaceptable: "el agente a veces ve la regla de seguridad y a veces no" no es una regla, es ruleta rusa.

**No versionado.** No hay commit que diga "el contexto del agente cambió aquí". Si la respuesta del MCP cambia mañana porque alguien editó un doc en Notion, y eso rompe al agente, no hay forma trivial de bisecar. En el modelo del repo, cada cambio que afecta al agente queda en `git log`; en el modelo MCP, los cambios son invisibles hasta que producen un fallo.

**Latencia y dependencia externa.** Cada consulta es un round-trip a una API externa que puede estar caída, lenta o rate-limitada. Si Notion se cae, tu agente está parcialmente ciego — y no necesariamente sabe que lo está. El repo, en cambio, es local al sandbox: es esencialmente instantáneo y nunca "se cae".

**La decisión de qué incluir se toma en query-time, no en authoring-time.** Esto es sutil pero es lo más importante. Cuando *escribes* un doc para el repo, decides activamente qué información va a ser accesible al agente y qué forma tiene. Ese contrato es explícito y revisable en code review. Cuando el agente *busca*, el subset que recibe lo decide un ranking opaco en tiempo de ejecución. Puedes haber escrito el doc perfecto y el ranker decidir que otro es más relevante. Para invariantes — donde necesitas garantizar que el agente vea X sí o sí — esto rompe el modelo.

**Permisos y autenticación complican el sandbox.** Para que el MCP funcione, el agente necesita credenciales con scopes que normalmente reservarías solo para humanos. Eso amplía la superficie de ataque del sandbox y mete al agente en un perímetro donde un prompt injection puede pivotar contra el doc store. Es un trade-off de seguridad real, no teórico.

**Coste de tokens y de tiempo en cada ejecución.** Cada llamada al MCP gasta contexto del agente y tokens de la API. El repo es gratis: leer un archivo es esencialmente coste cero. A escala (cientos o miles de invocaciones por día), la diferencia se nota.

### Cuándo sí, cuándo no

La heurística que sale de todo esto es simple y vale la pena interiorizarla:

> **Lo crítico, en el repo. Lo opcional, vía MCP. Sincronizar solo cuando lo crítico no se puede mover y aceptas pagar el precio de la ventana de staleness.**

Operativamente:

- **Si el agente tiene que ver un doc para no romper algo** — invariantes de arquitectura, decisiones irreversibles, golden principles, contratos de schema, reglas de seguridad — ese doc va al repo, sin excepciones. No se confía a búsqueda probabilística porque la búsqueda probabilística falla en silencio el 1% de las veces, y el 1% de fallos a escala de un agente son muchos fallos.
- **Si el doc es "nice to have, ayuda al agente a contextualizar"** — historia de un módulo, racional de una decisión vieja, notas de un dominio amplio donde el agente puede beneficiarse pero no depender — un MCP server contra Notion es una opción válida y barata. La búsqueda semántica brilla en este long-tail: encuentra cosas que tú no habías indexado manualmente.
- **Si lo crítico vive en una herramienta externa que no puedes mover por razones políticas u organizativas**, sincroniza al repo y mete un freshness lint que falle si el mirror lleva más de N horas sin actualizarse. El lint convierte el sync en una invariante visible: cuando el sync se rompe, lo descubres porque el build falla, no porque el agente empieza a tomar decisiones con datos viejos.

### La trampa más peligrosa

Vale la pena nombrarla porque es donde más equipos caen: **sincronizar sin freshness check**. Es la combinación que parece responsable — "tenemos un job que copia Notion al repo cada hora" — y que da la falsa sensación de que estás cubierto. Pero entre el último sync y el siguiente puede pasar cualquier cosa, y cuando el sync se rompe (y todos los syncs se rompen tarde o temprano), nadie se entera hasta que un agente toma una decisión basada en información de hace tres semanas. Si vas por sync, el lint de freshness no es opcional. Es lo que distingue una sincronización seria de una sincronización ceremonial.

### El principio detrás de todo esto

El cap. 6 entero defiende una idea: el repo es el sistema de registro porque es **determinista, versionado, auditable y siempre disponible**. Cualquier alternativa que pierda alguna de esas cuatro propiedades es aceptable solo donde esa propiedad no era crítica. Un MCP server con búsqueda semántica pierde las cuatro a la vez, y por eso solo encaja como complemento — nunca como reemplazo — para el conocimiento que el agente *debe* tener delante para no romper nada.

Dicho de otra forma: MCP es una buena forma de *enriquecer* el contexto del agente. No es una buena forma de *garantizar* el contexto del agente. La diferencia entre enriquecer y garantizar es la diferencia entre un asistente que a veces es brillante y un sistema del que un equipo puede depender.
