# Harness engineering — a guide to building software at scale with agents

A guide, in 13 chapters, on how technical teams can build a serious *harness* around AI coding agents (Claude Code, Codex, Copilot, Cursor, etc.) and stop treating them as glorified autocomplete.

> The model is the engine. The harness — guides, sensors, sandboxes, loops, structured context — is what turns an LLM into an agent a team can rely on.

## 📖 Read the web version

**[https://jmlopezdona.github.io/ai-coding-agents-harness/](https://jmlopezdona.github.io/ai-coding-agents-harness/)**

Available in English and Spanish, built with MkDocs Material: chapter navigation, integrated search, light/dark mode, language switcher in the top bar.

## 📁 Source

The 13 chapters live in [`docs/`](docs/), organized by language: [`docs/en/`](docs/en/) for English and [`docs/es/`](docs/es/) for Spanish. If you'd rather read them directly on GitHub:

- [Chapter 0 — Producing software at scale with agents](docs/en/00-harness-engineering.md)
- [Full guide index](docs/en/index.md)

## About the guide

Written for technical teams already operating agents and skills, who understand how to use them and want to take the next step: **design the system** instead of hunting for the magic prompt.

It synthesizes ideas from Birgitta Böckeler (Thoughtworks), Chad Fowler, Geoffrey Huntley, Kief Morris, Ryan Lopopolo (OpenAI Codex team) and Alistair Gray (Stripe Leverage team). Original URLs are in the [sources section](docs/en/index.md#sources) of the index.

## Roadmap for v2

Things planned for the next version of the guide:

- **Executable templates.** Minimal sample `AGENTS.md`, full `docs/` structure, custom linter examples with agent-targeted messages, sandbox config skeleton, execution plan templates.
- **Roadmap template as a separate file** (extracted from ch. 12) so teams can copy and fill it in a single session without touching the rest of the guide.
- **Short glossary** of terms (harness, guide, sensor, loop, sandbox, blueprint, doc-gardening, golden principles, flywheel) for teams sharing the guide with stakeholders outside the immediate technical circle.
- **Chapter 0 as a publishable blog post**, with its own frontmatter, optimized for external distribution.
- **Comparative case studies** — a side-by-side analysis of how OpenAI, Stripe, and other public teams solve each of the five dimensions in ch. 12, with an equivalence table.
- **Printable diagnostic checklist** (1 page) extracted from ch. 12 to stick on the team's wall.
- **Section on security and adversarial isolation** — what changes in the harness when the agent can be the target of prompt injection via context, how to reason about the tool supply chain, MCP servers, etc.

## License

[CC-BY-4.0](LICENSE). You can share and adapt the content freely, even commercially, as long as you keep attribution.
