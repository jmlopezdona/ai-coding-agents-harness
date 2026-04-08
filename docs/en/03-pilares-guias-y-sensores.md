# 3. Guides and Sensors: feedforward and feedback

Birgitta Böckeler splits the harness into two mechanisms. A simple but useful distinction, because it forces you to classify every investment you make: does this prevent a mistake before it happens (guide), or catch it after (sensor)? If it's neither, it probably doesn't belong in the harness.

## Guides — feedforward

A guide acts **before** the agent acts. Its job is to steer the action toward the right decision by giving the agent, upfront, the information it needs: the shape of the data, the structure of the module, the pattern to follow, where to look. A good guide isn't a prohibition; it's a cue the agent can read and imitate.

Typical examples:

- **Versioned AGENTS.md / system prompts.** Not as a place to dump "everything important", but as a short index pointing to where each thing lives (see ch. 6). The difference between a 100-line AGENTS.md and a 1,000-line one isn't scale: it's philosophy.
- **Conventions embedded in the scaffolding.** Service templates, module generators, component bootstraps. If every new module is created by a command, the agent inherits the structure without having to guess it.
- **Schemas and types at the edge.** Validating inputs with Zod, Pydantic, JSON Schema, or whatever fits. This isn't just runtime defense: it's a readable contract the agent can read and imitate. OpenAI explicitly mentions that they require parsing data shapes at the edge without being prescriptive about the tool.
- **Mechanized architecture rules.** Dependency directions between layers, forbidden modules, disallowed imports. Validated structurally, not in a review. (Note: the *rule* is the guide; the linter that validates it is a sensor — see below.)
- **PR and plan templates.** If all execution plans have the same shape, the agent knows how to write them and how to read the old ones.

The test for a good guide is this: **could a brand-new agent, with no prior context, do the right thing just by following it?** If the answer is "depends on judgment", the guide is incomplete.

### The most common mistake with guides

Confusing guide with documentation. A guide is executable or nearly executable. Documentation is read optionally. If your "guide" is a document the agent *could* consult but isn't required to obey mechanically, it's not a guide: it's a suggestion. And suggestions get ignored when the context fills up.

## Sensors — feedback

A sensor acts **after**. Its job is to detect when something's wrong with enough precision and speed that the agent itself (or the harness) can correct it without human intervention.

Böckeler distinguishes two types of execution:

- **Computational** — deterministic, fast, cheap. Tests, type checkers, linters, schema validation, builds, smoke tests.
- **Inferential** — non-deterministic, slower, capable of semantic evaluation. LLM evals, review by another agent, comparing outputs against a rubric.

Both are necessary. The mistake is using the second where the first would do (expensive, noisy) or trying to use the first where you need the second (impossible). Most teams err by not taking full advantage of the computational ones.

Practical examples:

- **Fast, reproducible tests.** Not "integration tests that take 8 minutes and sometimes fail from flake". Tests the agent can run without thinking twice and that it trusts.
- **Type checks as a primary sensor.** A strict type checker is one of the cheapest sensors and one of the most informative. If your stack allows it, crank up the restrictions.
- **Custom linters with messages aimed at the agent.** This is underrated. A custom lint doesn't just flag the bad pattern: its error message can contain the remedy instruction. The agent reads the error, understands the correct pattern, applies it on the next try. A good custom linter is a passive tutor: it acts after the failure (that's why it's a sensor), but its signal is didactic enough that the agent converges in one or two iterations.
- **Ephemeral observability locally.** This is what OpenAI did with the Vector + Victoria + LogQL/PromQL/TraceQL stack per worktree. It lets the agent ask questions like "does startup take less than 800ms?" and get a verifiable answer. Observability stops being a prod luxury and becomes part of the dev loop.
- **UI smoke tests via DevTools.** OpenAI wires Chrome DevTools Protocol into the agent runtime: the agent can take screenshots, navigate the DOM, reproduce visual failures. The UI stops being a black box to the agent.
- **Automatic reviews by another agent.** One agent reviews another's PR against a rubric. It doesn't replace the human, but it filters 80% of the noise before it reaches one.
- **Evals with frozen sets.** For tasks where there's no deterministic oracle (e.g. the quality of a generated message), an eval with frozen cases.

### The criterion for adding a sensor

A sensor is only useful if its signal feeds back into the loop. A test that fails and nobody fixes isn't a sensor: it's noise. A dashboard a human looks at once a month isn't a sensor for the agent: it's a memorial.

The right question before adding a sensor is: **when this sensor fires, what happens? Who (or what) reads the signal and what do they do with it?** If there's no concrete answer, don't add the sensor yet; design the loop first.

## The interaction between guides and sensors

The two pillars aren't independent. When a sensor repeatedly detects a bad pattern, that's information: there's probably a missing guide. And the other way around, when a guide blocks something legitimate, it's probably miscalibrated and needs to be relaxed or moved to a sensor.

The healthy flow looks something like this:

1. A sensor (test, lint, eval) detects a recurring failure.
2. The team identifies the pattern: the agent keeps trying to do X when the right thing is Y.
3. The rule gets promoted: a guide is added (a lint, a convention in AGENTS.md, a template) that makes X impossible or trivially detectable.
4. The sensor stays there as a safety net, but it almost never fires for that specific case anymore.

This is exactly what OpenAI describes as *"when documentation isn't enough, we promote the rule to code"*. There's no trick: it's platform engineering applied to a non-human collaborator.

## A warning about balance

It's possible to over-restrict. A harness with too many strict guides and too many hypersensitive sensors paralyzes the agent: any reasonable action trips five alarms and the loop stalls. The useful rule is the same one that applies to a human team: **strict limits where consequences are irreversible, autonomy where they're cheap to reverse**.

The next chapters apply these two pillars to concrete problems: the loop (ch. 4), isolated environments (ch. 5), context (ch. 6), architecture (ch. 7), and the PR flow (ch. 8). In each one you'll see guides and sensors working together.
