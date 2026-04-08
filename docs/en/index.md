# Harness engineering

A guide for technical teams already using AI coding agents (Claude Code, Codex, Copilot, Cursor, etc.) who want to take the next step: **stop treating them as autocomplete and start building a serious harness around them**.

This isn't a step-by-step tutorial or a manual for any particular product. It's an essay, organized in chapters, about the principles and patterns emerging in teams already operating at this scale (OpenAI, Stripe, Thoughtworks/Böckeler, Geoffrey Huntley) — patterns that show up regardless of stack or tool.

## Premises

- You work in a technical team with experience operating agents and skills.
- You want to **design the system**, not find the magic prompt.
- You're willing to invest in internal infrastructure (linters, sandboxes, observability, versioned docs) if it multiplies the agent's output.
- You care about sustainability over months, not just the first impressive commit.

## Central thesis (in one sentence)

> The model is the engine. The harness — guides, sensors, sandboxes, loops, structured context — is what turns an LLM into an agent a team can rely on.

## How to read this guide

The chapters are ordered as **mindset → pillars → practice → maintenance → anti-patterns → diagnosis**, but each one stands on its own. If you already get the "why", jump directly to the chapter that solves a concrete problem for you.

## Index

### Chapter 0 — Entry point

- [Harness engineering: producing software at scale with agents](00-blog-harness-engineering.md)

### Part 1 — Mindset

- [1. Why the harness matters more than the model](01-por-que-harness.md)
- [2. Relocating rigor](02-rigor-reubicado.md)

### Part 2 — The two pillars

- [3. Guides and Sensors: feedforward and feedback](03-pilares-guias-y-sensores.md)
- [4. The loop: iteration as primitive](04-el-loop.md)

### Part 3 — Practice

- [5. Isolated and reproducible environments](05-entornos-aislados.md)
- [6. Context as a system of record](06-contexto-sistema-de-registro.md)
- [7. Architecture for agents](07-arquitectura-para-agentes.md)
- [8. The agentic PR flow](08-flujo-pr-agentico.md)

### Part 4 — Sustainability

- [9. Entropy and continuous maintenance](09-entropia-y-mantenimiento.md)
- [10. Where humans still matter](10-rol-humano.md)

### Part 5 — Pitfalls

- [11. Anti-patterns](11-anti-patrones.md)

### Part 6 — Bringing it to your team

- [12. Diagnosis and maturity: where you are and what to fix first](12-diagnostico-y-madurez.md)

## What you *won't* find in this version

- Ready-to-copy templates (sample `AGENTS.md`, hooks, CI configs). Coming in a later version.
- Comparisons between specific tools. The guide is deliberately tool-agnostic.
- Productivity promises. Stripe's or OpenAI's numbers are references, not guarantees.

## Roadmap for v2

Things planned for the next version of the guide:

- **Executable templates.** Minimal sample `AGENTS.md`, full `docs/` structure, custom linter examples with agent-targeted messages, sandbox config skeleton, execution plan templates.
- **Roadmap template as a separate file** (extracted from ch. 12) so teams can copy and fill it in a single session without touching the rest of the guide.
- **Short glossary** of terms (harness, guide, sensor, loop, sandbox, blueprint, doc-gardening, golden principles, flywheel) for teams sharing the guide with stakeholders outside the immediate technical circle.
- **Chapter 0 as a publishable blog post**, with its own frontmatter, optimized for external distribution.
- **Comparative case studies** — a side-by-side analysis of how OpenAI, Stripe, and other public teams solve each of the five dimensions in ch. 12, with an equivalence table.
- **Printable diagnostic checklist** (1 page) extracted from ch. 12 to stick on the team's wall.
- **Section on security and adversarial isolation** — what changes in the harness when the agent can be the target of prompt injection via context, how to reason about the tool supply chain, MCP servers, etc.

## Sources

This guide synthesizes ideas from:

- Birgitta Böckeler (Thoughtworks, published on martinfowler.com) — [*Harness Engineering*](https://martinfowler.com/articles/harness-engineering.html). The term "harness" in this sense emerged around LangChain with the formula *"Agent = Model + Harness"*.
- Kief Morris (on martinfowler.com) — [*Humans and Agents in Software Engineering Cycles*](https://martinfowler.com/articles/exploring-gen-ai/humans-and-agents.html)
- Chad Fowler — [*Relocating Rigor — The Phoenix Architecture*](https://aicoding.leaflet.pub/3mbrvhyye4k2e)
- Geoffrey Huntley — [*everything is a ralph loop*](https://ghuntley.com/loop/)
- Ryan Lopopolo (OpenAI) — [*Harness Engineering: leveraging Codex in an agent-first world*](https://openai.com/index/harness-engineering/)
- Alistair Gray (Stripe, Leverage team) — *Minions: Stripe's one-shot, end-to-end coding agents* — [part 1](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents) · [part 2](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2)
