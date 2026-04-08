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

- **Documentos de diseño versionados**, indexados, con un estado de verificación. Nada de "doc en Notion enlazado". El doc vive en el repo, junto al código que describe.
- **Planes de ejecución como artefactos de primera clase.** Para cambios pequeños, planes ligeros y efímeros. Para trabajo complejo, planes versionados con estado: activos, completados, abandonados, con razones. Esto le permite al agente ver no solo qué está pasando ahora, sino qué se intentó antes y por qué se descartó. Es una herramienta de memoria a meses vista.
- **Tracker de deuda técnica.** Un archivo, en el repo, que enumera lo que está mal a propósito. Sin esto, el agente "limpia" cosas que no debería tocar, o ignora cosas que debería arreglar. Con esto, las dos categorías son legibles.
- **Especificaciones de producto.** No las épicas de Linear. Las specs *como markdown en el repo*, junto al código que las implementa. Si el ticket es la única fuente, el agente solo ve lo que está en el ticket cuando se lo pasas.
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
