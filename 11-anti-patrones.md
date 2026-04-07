# 11. Anti-patrones

Este capítulo cierra la guía listando las trampas que los equipos caen más a menudo al introducir agentes en serio. Cada una es razonable en aislamiento, la mayoría tiene buena intención detrás, y cada una termina produciendo un harness que no escala. La estructura es simple: qué es, por qué se cae en ella, y qué hacer en su lugar.

## Anti-patrón 1 — El AGENTS.md enciclopédico

**El síntoma.** Un único archivo `AGENTS.md` de cientos o miles de líneas que intenta capturar todo lo importante del proyecto: convenciones, arquitectura, glosario, advertencias, historia, reglas de negocio.

**Por qué parece buena idea.** Es tentador concentrar el contexto en un sitio para que el agente "lo tenga todo". Se siente como documentación meticulosa.

**Por qué falla.** Cuatro razones, ya cubiertas en el capítulo 6 pero vale la pena recordar: eclipsa el contexto relevante, diluye las señales importantes, se descompone silenciosamente, y no es verificable. El agente acaba ignorando secciones enteras, o peor, aplicando reglas obsoletas.

**Qué hacer en su lugar.** AGENTS.md como índice corto (~100 líneas) que apunta a `docs/` estructurado. Cada pieza de contexto en su propio archivo, con metadatos mínimos, linters que validen que no se degrada, y agentes de doc-gardening que la mantengan.

## Anti-patrón 2 — Revisar cada PR como en los viejos tiempos

**El síntoma.** Política de "dos aprobaciones humanas antes de merge" o similar, heredada del flujo tradicional, aplicada indiscriminadamente a los PRs generados por agentes.

**Por qué parece buena idea.** Parece responsable. "Los humanos siguen teniendo la última palabra" suena bien en reuniones.

**Por qué falla.** Aritmética: si el agente produce 50 PRs al día y cada review lleva 15 minutos, los humanos son el cuello de botella permanente. Peor, las reviews en esas condiciones se vuelven superficiales y no pillan nada; se convierten en ritual, no en control.

**Qué hacer en su lugar.** Categorizar PRs (capítulo 8): trivial auto-mergeable, rutinarios revisados por agentes, sensibles con humano, decisiones con humano desde antes. Reserva la atención humana para los casos donde aporta valor real.

## Anti-patrón 3 — Tratar al agente como autocomplete glorificado

**El síntoma.** Los ingenieros usan al agente para generar fragmentos de código dentro de su editor, copian, pegan, ajustan manualmente. No hay loop, no hay sandbox, no hay sensores automáticos.

**Por qué parece buena idea.** Es la adopción más suave, sin reestructurar nada. Se siente como una productividad pequeña pero segura.

**Por qué falla.** Es lo contrario de un harness. No acumulas nada, no mejoras nada, no desbloqueas el paralelismo. El agente es útil al 5% de su capacidad y los ingenieros se convencen de que "los agentes son útiles pero limitados" basándose en una muestra que es exactamente el peor caso.

**Qué hacer en su lugar.** Invertir en el loop desde el principio (capítulo 4). El agente trabaja en su sandbox, ejecuta, valida, itera. Si todavía usas el agente principalmente como autocomplete, no estás en el mundo que describe esta guía.

## Anti-patrón 4 — Ignorar el aislamiento de entornos

**El síntoma.** Los agentes corren directamente en la máquina del desarrollador, en el checkout principal, con la base de datos local compartida con todo lo demás.

**Por qué parece buena idea.** "Para qué complicarse con devboxes, tenemos Docker". O: "es solo una prueba, ya veremos si inviertimos".

**Por qué falla.** Sin aislamiento no hay paralelismo, no hay reproducibilidad, no hay observabilidad efímera, no hay seguridad. Es la inversión que más multiplica las otras, y todas las demás se quedan en la mitad de su potencial hasta que exista.

**Qué hacer en su lugar.** Sandboxes desechables, rápidos, completos, reproducibles (capítulo 5). Aunque sea con una implementación simple al principio. La métrica: ¿puedes lanzar 5 agentes en paralelo sin que se pisen? Si no, falta trabajo aquí.

## Anti-patrón 5 — Documentación humana como harness

**El síntoma.** "Lo tenemos documentado en el wiki del equipo", "está en el style guide", "se explica en la página de onboarding". El equipo confía en que el agente respetará convenciones que viven en documentos que no están en el repo.

**Por qué parece buena idea.** Ya existe la documentación; ¿por qué duplicarla?

**Por qué falla.** Lo que el agente no puede ver, no existe (capítulo 6). El wiki, Confluence, Notion, el Google Doc — todo eso es invisible para el agente cuando ejecuta. El equipo asume que el agente "sabe" cosas que en realidad nunca ha visto.

**Qué hacer en su lugar.** Materializa la documentación relevante en el repo. Y cuando la documentación describa reglas obligatorias, promuévelas a código (lints, validadores) para que el enforcement no dependa de que nadie las lea.

## Anti-patrón 6 — Corregir al agente en lugar de corregir el harness

**El síntoma.** El mismo error aparece repetidamente. Cada vez, el ingeniero le dice al agente "no hagas eso, hazlo así" y sigue adelante. El repo acumula PRs con correcciones del mismo patrón una y otra vez.

**Por qué parece buena idea.** Es más rápido arreglar el síntoma inmediato que escribir un lint o actualizar una guide. Se siente como progreso.

**Por qué falla.** No acumulas nada. La segunda vez que ves el mismo error, tu pregunta debe ser "¿qué debería haber impedido esto?". Si no la haces, estarás arreglando este patrón toda la vida.

**Qué hacer en su lugar.** La regla de las dos veces: la primera vez que un error ocurre, lo arreglas puntualmente. La segunda vez, paras y lo promueves al harness (capítulo 9). No hay tercera vez bien gestionada.

## Anti-patrón 7 — Megalomanía del plan inicial

**El síntoma.** Antes de dejar que el agente toque nada, el equipo se pasa semanas diseñando el harness "perfecto": arquitectura detallada, plantillas exhaustivas, veinte lints custom, pipeline de CI con doce fases. Cuando por fin se pone a funcionar, la mitad de las decisiones están mal calibradas.

**Por qué parece buena idea.** Se siente responsable. "Primero hagamos las cosas bien".

**Por qué falla.** El harness es un sistema adaptativo. Muchas de las decisiones correctas solo se pueden tomar después de ver al agente fallar de formas específicas. Un harness diseñado en vacío optimiza problemas imaginarios y deja fuera los reales.

**Qué hacer en su lugar.** Empezar con lo mínimo viable — sandbox básico, AGENTS.md corto, sensores computacionales existentes — y dejar que los problemas reales dirijan la inversión. Cada nuevo lint, nuevo guide, nuevo sensor debería estar justificado por un fallo observado, no por uno hipotético.

## Anti-patrón 8 — No invertir nunca en el harness

**El síntoma.** El opuesto del anterior. "Primero veamos si funciona, luego pensaremos en el harness". Tres meses después, el equipo sigue operando con un setup improvisado, los mismos errores se repiten, y nadie se atreve a parar a invertir porque "estamos entregando".

**Por qué parece buena idea.** Velocidad visible a corto plazo. Resultados "inmediatos".

**Por qué falla.** La deuda de harness es como cualquier deuda técnica pero amplificada. En el modelo clásico, la deuda ralentiza a los ingenieros. En el modelo agéntico, la deuda ralentiza *y* produce código drift exponencial. Lo que parecía velocidad al mes 1 se convierte en parálisis al mes 6.

**Qué hacer en su lugar.** Parte del trabajo del equipo es siempre mejorar el harness. Una proporción explícita — 20%, un día por sprint, un miembro rotativo — dedicada a guides, sensors, agentes recurrentes. No negociable. Si es negociable, siempre se negocia en su contra.

## Anti-patrón 9 — El harness como proyecto paralelo

**El síntoma.** Un "equipo de plataforma" separado construye el harness; el resto del equipo lo consume sin entenderlo. Cuando algo falla, la respuesta es "pedidlo al equipo de plataforma".

**Por qué parece buena idea.** Especialización. Deja que los "expertos" construyan la infraestructura y los demás entregan producto.

**Por qué falla.** El harness tiene que evolucionar basándose en los fallos que observan los que lo usan. Si los que observan los fallos no lo modifican, la retroalimentación se pierde. El equipo de plataforma trabaja en lo que imagina que importa; el equipo de producto sufre lo que realmente importa.

**Qué hacer en su lugar.** Todos contribuyen al harness. Los lints custom los escribe quien observó el fallo. Los guides los actualiza quien tropezó con la ambigüedad. Un equipo de plataforma puede existir como acelerador, pero no como monopolio.

## Anti-patrón 10 — Creer que el modelo es el problema

**El síntoma.** Cuando el agente falla, la conversación por defecto es "necesitamos un modelo mejor" o "esperemos a la próxima versión".

**Por qué parece buena idea.** A veces es verdad. Los modelos mejoran.

**Por qué falla.** La mayoría de las veces, el fallo no está en el modelo — está en lo que el modelo podía ver (contexto), en cómo se evaluó su output (sensores), en las restricciones que tenía disponibles (guides), o en el entorno donde ejecutó (sandbox). Esperar un modelo mejor para arreglar un harness roto es gastar el tiempo en el sitio equivocado. Cuando el modelo nuevo llegue, el harness seguirá roto y los problemas seguirán.

**Qué hacer en su lugar.** Primero reflejo: ante cualquier fallo, preguntar "¿qué del harness no está haciendo su trabajo?". Si la respuesta honesta es "el harness está bien, es el modelo", entonces sí, espera al modelo. Ocurre; solo es menos frecuente de lo que tu equipo probablemente cree.

---

## Cierre de la guía

Si esta guía tiene una única conclusión transferible es esta: **la calidad de un agente no está en el modelo que usas; está en el sistema que construyes alrededor del modelo**. El modelo es una commodity. El harness es tu ventaja competitiva.

Esto es, en el fondo, una buena noticia. El modelo no lo controlas tú — lo controla OpenAI, Anthropic, Google o quien sea. El harness sí. Y como todas las cosas que controlas en ingeniería, mejora con intención, con disciplina, y con un equipo que entiende qué está construyendo y por qué.

Los capítulos 1 al 10 describen los pilares. Este capítulo 11 describe las trampas. La siguiente versión de esta guía añadirá plantillas ejecutables (AGENTS.md de ejemplo, estructura de `docs/`, linters, hooks, configuraciones de sandbox) para que puedas ir del ensayo a la implementación sin empezar desde cero.

Mientras tanto: empieza con el loop, invierte en el aislamiento, materializa el contexto en el repo, codifica los invariantes que más te importan, y cuando algo falle dos veces, promuévelo al harness. El resto sale por gravedad.
