# 1. Why the harness matters more than the model

The public conversation about coding agents almost always revolves around the model: which version, which benchmark, which context window. It's the wrong question for a team that already uses these tools daily. The useful question is a different one: **what's around the model?**

The community calls that "around" the *harness*: the set of prompts, tools, rules, sandboxes, validators and feedback loops that surround the LLM when it acts as an agent. The term emerged around LangChain (*"Agent = Model + Harness"*) and Birgitta Böckeler articulated it in more detail in *Harness Engineering* (Thoughtworks, published on martinfowler.com). The model is interchangeable. The harness isn't.

And worth clarifying from the start: when we say "harness" there are two layers. The one your AI tool ships with — Claude Code, Copilot, Cursor and similar tools **are already a harness** on top of the LLM — and the one your team builds on top to adapt it to your repo, your domain and your conventions. The first is shared by everyone, and each new release of these tools moves more capabilities from the second group into the first. The second is what sets you apart, and it's what this guide is about.

## The engine analogy

An F1 engine inside a compact car doesn't win races. What wins races is the chassis, the aerodynamics, the tires, the telemetry and the pit crew. The model is the engine; everything else is where the race is won or lost. And the important part: that "everything else" is something you build, not the model vendor.

When a team complains that "the agent hallucinates" or "doesn't understand the code", the problem is almost never the model. The problem is that the agent is operating in an environment where it would be unreasonable to expect a new human to understand anything either. It has no map, no fast tests, can't reproduce the bug, doesn't know the team convention and nobody tells it when what it did is wrong.

## What's inside the harness (a brief reminder)

The introductory chapter breaks the harness into two mechanisms worth keeping in mind: **guides** (feedforward — conventions, templates, AGENTS.md, schemas) that orient the agent before it acts, and **sensors** (feedback — tests, type checkers, linters, observability, evals) that correct it after. We take that distinction as given; we'll use it without redefining it for the rest of the guide.

## What they actually built

The headline numbers — Stripe's >1,000 PRs/week, OpenAI's ~1M lines and ~1,500 PRs in five months — are already in the introductory chapter. What's worth adding here is what they *specifically* built, because that's where the shape of the work we're proposing lives:

- **Stripe** didn't win with a better model. They won with **Toolshed** (~500 internal tools exposed to the agent via MCP), **devboxes pre-instantiated in 10 seconds**, and a **blueprints** system that combines deterministic nodes with agentic nodes depending on the kind of work. All three pieces are harness, not model.
- **OpenAI** did it with an **app bootable per worktree** (one agent per feature to implement, running in parallel, no collisions), **Chrome DevTools Protocol wired to the runtime** (the agent sees the UI the way the user sees it), **ephemeral observability queryable with LogQL/PromQL**, and a `docs/` tree structured as a system of record (more in ch. 6).
- **Geoffrey Huntley** isolates the primitive that reappears across almost all these teams: a monolithic orchestrator running one iteration at a time. It's not a feature of the model; it's an architectural decision of the team operating it (ch. 4).

None of this requires a secret model. Everything is decided by your team. Everything compounds.

## Why this changes the team's work

Once you accept that the harness is where quality is decided, you stop doing certain things and start doing others. You stop looking for the "perfect prompt" — prompts live in the harness now, versioned as code. You stop copy-pasting errors into the chat — errors enter the loop via automatic sensors. You stop asking the agent to guess the context — the context lives structured in the repo, mechanically accessible.

And you start doing things that used to look like over-engineering: custom linters for a single convention, versioned execution plans, local observability, "doc-gardening" agents that keep documentation up to date.

What in a human team would be nitpicky, in a team with agents **is a multiplier**. A rule codified once applies forever, on every PR, without wear, without debate, without having to remind anyone at the next standup.

## The principle that runs through the whole guide

The introductory chapter stated it: when something fails, the useful question isn't "how do I tell the agent this is wrong?" but **"what is missing from the harness so this can't happen, or so it gets detected automatically when it happens?"**. The chapters that follow are, in large part, a catalog of what you can add to the harness when you ask yourself that question — guide by guide, sensor by sensor.
