# 7. Architecture for agents

This is the chapter that generates the most resistance in experienced teams. The idea — that software architecture should be designed around how agents are going to read and modify it, not just humans — sounds like tail-wagging-the-dog. But when you start operating agents seriously, the observation becomes hard to avoid: **codebases with mechanically verifiable invariants and strict boundaries produce better agent results than "elegant" codebases with implicit discipline**. It's not an aesthetic preference. It's a measurable property.

## The uncomfortable observation

OpenAI says it explicitly and it's worth quoting: *"It's the kind of architecture you usually postpone until you have hundreds of engineers. With coding agents, it's one of the initial prerequisites: the constraints are what allow for speed, without degradation or architectural drift."*

Read it again. What's normally considered over-engineering — strict layers, structural validation, custom linters for every convention — becomes an *enabling condition* when the code's writer is an agent. The reason is simple: constraints are the only way the agent "knows" what the architecture is. It has no taste, no history, no memory of the decision we made two years ago. It has what the code makes mechanically clear.

## Layers as invariant, not suggestion

OpenAI describes a concrete model worth understanding, not to copy literally, but to capture the shape. Each business domain is split into fixed layers with validated dependency directions:

**Types → Config → Repository → Service → Runtime → UI**

Dependencies only go forward. Cross-cutting concerns (auth, telemetry, feature flags) enter exclusively via an explicit **Providers** layer. Any import that violates this is a build error.

The important part isn't the exact list of layers. It's that **the architectural model is executable**. A custom linter (written by the agent itself, no less) reads the imports and rejects the ones that cross boundaries. No debate, no "this time is an exception", no human review deciding case by case. The architecture is in the code that validates the code.

For a human team this can feel rigid. For an agent, it's liberating: it no longer has to guess the pattern, it discovers it by trying something and reading the error. And custom lint errors can contain remediation instructions aimed explicitly at the agent — "this layer can only import from X and Y; move this function to layer Z" — turning every lint into a passive tutor.

## "Boring" dependencies

Another observation from OpenAI: they prefer "boring" libraries — composable, with stable APIs, well represented in the model's training data. The reason isn't nostalgia. It's that those libraries are **mechanically easier for the agent to reason about**: there are more examples in the corpus, conventions are stable, behavior is predictable.

There's a limit case that deserves attention because it breaks intuitions: sometimes it's cheaper for the agent to reimplement a subset of a library than to integrate it. OpenAI gives a concrete example: instead of using `p-limit` (a tiny concurrency helper), Codex implemented its own version integrated with their OpenTelemetry instrumentation, with 100% coverage. Why? Because the external library was a black box whose opaque behavior generated more friction in the loop than the code that replaces it.

This runs against the "don't reinvent the wheel" reflex, and it's important to calibrate it. The useful rule is: **if the agent can read, modify and validate the complete code of something in its effective context, that's a net architectural advantage**. External dependencies the agent can't inspect are permanent friction. The ones it can inspect are collaborators. The difference shows up months down the road.

This doesn't mean reimplementing React. It means that when you're about to pull in a 200-line utility as a transitive dependency, pause and ask yourself if it's worth more to write it as project code.

## Style as invariant

OpenAI mentions "style invariants" enforced statically: mandatory structured logging, naming conventions for schemas and types, file size limits, platform-specific reliability requirements. Not "style guide in a Confluence". Lints. In the code. Blocking merge.

The difference is decisive. A human style guide depends on the reviewer remembering and applying it. A lint depends on nothing: either it's there, or it isn't. And given that the agent can generate thousands of lines per day, the only way to maintain consistency is to give up on some human maintaining it.

An important tactical detail: **the lint's output is for the agent, not just for the human**. When you write a custom lint, write the error message thinking that an agent is going to read it: explain the correct pattern, give an example, indicate where to look. A lint with the message "incorrect import" is useless. A lint with the message "this module can only import from Providers; move the dependency to `providers/auth.ts` or rewrite using the existing provider" is active teaching.

## Strict boundaries, local autonomy

The balance OpenAI articulates is worth copying literally: *"lead like a large engineering platform organization: setting boundaries centrally and allowing autonomy at the local level. You care a lot about boundaries, correctness, and reproducibility. Within those boundaries, you allow teams (or agents) considerable freedom in how solutions are expressed."*

In practice this means:

- **Boundaries are sacred:** dependency directions, schema contracts, mandatory observability, error handling at edges. You block merge.
- **What's inside is free:** how an internal variable is named, how a private function is structured, what specific pattern it uses to iterate a list. Not the lint's business, not the review's business.

There's a consequence many teams find hard to accept: **the resulting code isn't always going to match your stylistic preferences**. And that's fine. As long as it's correct, maintainable and legible for future agent runs, it meets the standard. Arguing about the name of an internal function that a lint doesn't catch is, in this context, pure waste of human attention — and human attention is now the scarce resource.

## The brown-field paradox

There's an uncomfortable observation worth naming before talking tactics: **brown-field projects are at the same time where a good harness is most needed and where it's hardest to build**. Both things at once, and for the same reasons.

**Why it's more needed.** A large, old codebase with years of organic growth, modules inherited from three different teams, and zones nobody on the current team fully understands, is exactly the kind of environment where an agent can multiply the team's output — *if* it can operate safely. A greenfield is trivial for any senior engineer; a brown-field is where the agent, well-directed, saves you weeks of archaeology every month. The potential ROI is maximum.

**Why it's harder.** The same properties that make the agent valuable in brown-field are the ones that make it hard to build a harness for it:

- Discipline lives as culture, not as code. The "rules" are in the heads of seniors, in past PR reviews, in Slack threads from two years ago. Materializing them as lints is archaeological work that has to happen before any investment in the harness.
- The architecture is heterogeneous. Different domains follow different conventions, layer boundaries exist in some places and not in others. You can't write *one* dependency-direction lint: you have to write *N* lints, one per coherent zone, and accept that some zones aren't codifiable until you refactor them first.
- Entropy is real and pushes back. Every new rule you introduce hits code that already violates it, and you have to decide case by case whether it's a legitimate exception or debt that has to be paid. That's not the agent's work; it's human calibration work that has to happen first.
- The team is used to the imperfections. What would be an obvious bug in a greenfield is "it's always been this way" in a brown-field. Turning that into mandatory invariants generates social friction, not just technical.

**And there's an aggravating factor: the agent amplifies existing entropy.** This matters because it turns difficulty into a feedback loop. An agent learns from the code in front of it: if the base is full of bad practices, it imitates them, propagates them faster than a human would, and reinforces the feeling that "this is how we do things here" because the new code looks like the old code. Without a sensor that detects the bad pattern or a guide that forbids it, the agent has no way to distinguish between healthy convention and inherited debt — for the agent, everything it sees in the repo is "what the team does". The loop is silent: it looks like the agent is being productive and consistent, and it is; only the consistency is with what's wrong. In a brown-field without a harness, **introducing an agent accelerates entropy instead of fighting it**. That's the strongest reason not to postpone the harness in this kind of codebase.

**The operational consequence**: in brown-field, harness investment isn't optional — it's a prerequisite. And it can't be done in a week. The question isn't "do we want a harness?" but "what speed of harness construction can we sustain without stopping delivery?". The healthy answer is usually: one new rule per week, one zone codified per month, no big-bang. What the next section describes applies to both cases, but **in brown-field it's the only way that works**.

## How to introduce this in an existing repo

Doing a big-bang on an existing repo is a bad idea. What works:

1. **Start with the layers that are already clear.** If your repo already has a reasonable distinction between domains or modules, write the lint that makes it mandatory. You're not imposing new architecture, you're freezing the one you already have.
2. **Promote one rule at a time.** A new lint a week, not twelve at once. Every new lint is a guide, and new guides require adjustment from the agent and the team.
3. **Start as warning, escalate to error.** A new lint in warning mode teaches you how noisy it's going to be. When the noise drops to zero, escalate it to error.
4. **Write the linter with the agent.** This is meta-level and worth it: having the agent itself write its constraints (under your supervision) makes the intent explicit and produces code the agent itself can read and modify later.

## The shift in horizon

The conclusion worth internalizing: when you invest in mechanically verifiable architecture, you're not doing it for "today". You're doing it for the next 18 months, during which that same architecture is going to sustain tens of thousands of agent modifications without wear. It's the investment with the best ROI months down the road of any that appear in this guide. It's also the easiest to underestimate in the short term, because its benefit accumulates slowly and becomes visible when an equivalent team without this investment is already drowning in drift.
