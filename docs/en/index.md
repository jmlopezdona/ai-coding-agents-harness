# Harness engineering

This is the third guide in a trilogy. In [Fundamentals](https://jmlopezdona.github.io/ai-coding-agents-fundamentals/) we covered how to pilot a coding agent; in [Spec-Driven Development](https://jmlopezdona.github.io/ai-coding-agents-sdd/) we learned to specify intent so the agent can execute it well. Now comes what's left: **designing the system that surrounds the agent** — guides, sensors, sandboxes, loops, structured context — so a team can rely on it sustainably.

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

- [Harness engineering: producing software at scale with agents](00-harness-engineering.md)

### Part 1 — Mindset

- [1. Why the harness matters more than the model](01-why-harness.md)
- [2. Relocating rigor](02-relocating-rigor.md)

### Part 2 — The two pillars

- [3. Guides and Sensors: feedforward and feedback](03-guides-and-sensors.md)
- [4. The loop: iteration as primitive](04-the-loop.md)

### Part 3 — Practice

- [5. Isolated and reproducible environments](05-isolated-environments.md)
- [6. Context as a system of record](06-context-system-of-record.md)
- [7. Architecture for agents](07-architecture-for-agents.md)
- [8. The agentic PR flow](08-agentic-pr-flow.md)

### Part 4 — Sustainability

- [9. Entropy and continuous maintenance](09-entropy-and-maintenance.md)
- [10. Where humans still matter](10-human-role.md)

### Part 5 — Pitfalls

- [11. Anti-patterns](11-anti-patterns.md)

### Part 6 — Bringing it to your team

- [12. Diagnosis and maturity: where you are and what to fix first](12-diagnosis-and-maturity.md)

## What you *won't* find in this version

- Ready-to-copy templates (sample `AGENTS.md`, hooks, CI configs). Coming in a later version.
- Comparisons between specific tools. The guide is deliberately tool-agnostic.
- Productivity promises. Stripe's or OpenAI's numbers are references, not guarantees.

## Sources

This guide synthesizes ideas from:

- Birgitta Böckeler (Thoughtworks, published on martinfowler.com) — [*Harness Engineering*](https://martinfowler.com/articles/harness-engineering.html). The term "harness" in this sense emerged around LangChain with the formula *"Agent = Model + Harness"*.
- Kief Morris (on martinfowler.com) — [*Humans and Agents in Software Engineering Cycles*](https://martinfowler.com/articles/exploring-gen-ai/humans-and-agents.html)
- Chad Fowler — [*Relocating Rigor — The Phoenix Architecture*](https://aicoding.leaflet.pub/3mbrvhyye4k2e)
- Geoffrey Huntley — [*everything is a ralph loop*](https://ghuntley.com/loop/)
- Ryan Lopopolo (OpenAI) — [*Harness Engineering: leveraging Codex in an agent-first world*](https://openai.com/index/harness-engineering/)
- Alistair Gray (Stripe, Leverage team) — *Minions: Stripe's one-shot, end-to-end coding agents* — [part 1](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents) · [part 2](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2)
