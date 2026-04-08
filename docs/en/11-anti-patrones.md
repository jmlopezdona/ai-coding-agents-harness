# 11. Anti-patterns

This chapter closes the guide by listing the traps teams fall into most often when introducing agents seriously. Each one is reasonable in isolation, most have good intentions behind them, and each one ends up producing a harness that doesn't scale. The structure is simple: what it is, why people fall into it, and what to do instead.

## Anti-pattern 1 — The encyclopedic AGENTS.md

**The symptom.** A single `AGENTS.md` file of hundreds or thousands of lines trying to capture everything important about the project: conventions, architecture, glossary, warnings, history, business rules.

**Why it seems like a good idea.** It's tempting to concentrate context in one place so the agent "has it all". It feels like meticulous documentation.

**Why it fails.** Four reasons, already covered in chapter 6 but worth remembering: it eclipses relevant context, dilutes important signals, decays silently, and isn't verifiable. The agent ends up ignoring entire sections, or worse, applying obsolete rules.

**What to do instead.** AGENTS.md as a short index (~100 lines) pointing to a structured `docs/`. Each piece of context in its own file, with minimal metadata, linters that validate it isn't degrading, and doc-gardening agents that maintain it.

## Anti-pattern 2 — Reviewing every PR like in the old days

**The symptom.** A "two human approvals before merge" policy or similar, inherited from the traditional flow, applied indiscriminately to agent-generated PRs.

**Why it seems like a good idea.** It sounds responsible. "Humans still have the last word" sounds good in meetings.

**Why it fails.** Arithmetic: if the agent produces 50 PRs a day and each review takes 15 minutes, humans are a permanent bottleneck. Worse, reviews under those conditions become superficial and don't catch anything; they turn into ritual, not control.

**What to do instead.** Categorize PRs (chapter 8): trivial auto-mergeable, routine reviewed by agents, sensitive with human, decisions with human from the start. Reserve human attention for the cases where it adds real value.

## Anti-pattern 3 — Treating the agent as glorified autocomplete

**The symptom.** Engineers use the agent to generate code snippets inside their editor, copy, paste, adjust manually. No loop, no sandbox, no automatic sensors.

**Why it seems like a good idea.** It's the softest adoption, nothing has to be restructured. It feels like a small but safe productivity gain.

**Why it fails.** It's the opposite of a harness. You accumulate nothing, you improve nothing, you unblock no parallelism. The agent is useful at 5% of its capacity and the engineers convince themselves "agents are useful but limited" based on a sample that is exactly the worst case.

**What to do instead.** Invest in the loop from the start (chapter 4). The agent works in its sandbox, executes, validates, iterates. If you're still using the agent primarily as autocomplete, you're not in the world this guide describes.

## Anti-pattern 4 — Ignoring environment isolation

**The symptom.** Agents run directly on the developer's machine, on the main checkout, with the local database shared with everything else.

**Why it seems like a good idea.** "Why bother with devboxes, we have Docker". Or: "it's just a test, we'll see if we invest later".

**Why it fails.** Without isolation there's no parallelism, no reproducibility, no ephemeral observability, no security. It's the investment that multiplies the others the most, and all the others stay at half their potential until it exists.

**What to do instead.** Disposable, fast, complete, reproducible sandboxes (chapter 5). Even with a simple implementation at first. The metric: can you launch 5 agents in parallel without them stepping on each other? If not, there's work to do here.

## Anti-pattern 5 — Human documentation as harness

**The symptom.** "We have it documented in the team wiki", "it's in the style guide", "it's explained on the onboarding page". The team trusts that the agent will respect conventions that live in documents that aren't in the repo.

**Why it seems like a good idea.** The documentation already exists; why duplicate it?

**Why it fails.** What the agent can't see, doesn't exist (chapter 6). The wiki, Confluence, Notion, the Google Doc — all that is invisible to the agent when it runs. The team assumes the agent "knows" things it has actually never seen.

**What to do instead.** Materialize the relevant documentation in the repo. And when the documentation describes mandatory rules, promote them to code (lints, validators) so enforcement doesn't depend on anyone reading it.

## Anti-pattern 6 — Correcting the agent instead of correcting the harness

**The symptom.** The same mistake appears repeatedly. Each time, the engineer tells the agent "don't do that, do it like this" and moves on. The repo accumulates PRs with corrections of the same pattern over and over.

**Why it seems like a good idea.** It's faster to fix the immediate symptom than to write a lint or update a guide. It feels like progress.

**Why it fails.** You accumulate nothing. The second time you see the same error, your question should be "what should have prevented this?". If you don't ask it, you'll be fixing this pattern forever.

**What to do instead.** The two-times rule is a tolerance ceiling, not a recommendation to patch first. If the failure is obviously systemic from the first occurrence (a missing convention, clear architectural drift, a pattern you can already sense is going to repeat), you promote it to the harness immediately. If it looks like a genuine oddity, you fix it with minimum spend and note it — but the second time there's no decision: you stop and promote it to the harness (chapter 9), because now you have evidence it's a pattern. The rule isn't "manual fix the first time, systematize the second" — it's "doubt at most once; twice is certainty". There's no well-handled third time.

## Anti-pattern 7 — Megalomania of the initial plan

**The symptom.** Before letting the agent touch anything, the team spends weeks designing the "perfect" harness: detailed architecture, exhaustive templates, twenty custom lints, a CI pipeline with twelve phases. When it finally gets running, half the decisions are miscalibrated.

**Why it seems like a good idea.** It feels responsible. "Let's do things right first".

**Why it fails.** The harness is an adaptive system. Many of the correct decisions can only be made after seeing the agent fail in specific ways. A harness designed in a vacuum optimizes imaginary problems and leaves out the real ones.

**What to do instead.** Start with the minimum viable — basic sandbox, short AGENTS.md, existing computational sensors — and let real problems drive the investment. Every new lint, new guide, new sensor should be justified by an observed failure, not a hypothetical one.

## Anti-pattern 8 — Never investing in the harness

**The symptom.** The opposite of the previous one. "First let's see if it works, then we'll think about the harness". Three months later, the team is still operating with an improvised setup, the same errors keep repeating, and nobody dares to stop and invest because "we're shipping".

**Why it seems like a good idea.** Visible short-term speed. "Immediate" results.

**Why it fails.** Harness debt is like any technical debt but amplified. In the classic model, debt slows down engineers. In the agentic model, debt slows you down *and* produces exponential code drift. What looked like speed at month 1 turns into paralysis at month 6.

**What to do instead.** Part of the team's work is always improving the harness. An explicit proportion — 20%, one day per sprint, a rotating member — dedicated to guides, sensors, recurring agents. Non-negotiable. If it's negotiable, it always gets negotiated down.

## Anti-pattern 9 — The harness as a side project

**The symptom.** A separate "platform team" builds the harness; the rest of the team consumes it without understanding it. When something fails, the answer is "ask the platform team".

**Why it seems like a good idea.** Specialization. Let the "experts" build the infrastructure and the rest ship product.

**Why it fails.** The harness has to evolve based on the failures observed by those using it. If those who observe the failures don't modify it, the feedback is lost. The platform team works on what it imagines matters; the product team suffers what actually matters.

**What to do instead.** Everyone contributes to the harness. Custom lints are written by whoever observed the failure. Guides are updated by whoever tripped over the ambiguity. A platform team can exist as an accelerator, but not as a monopoly.

## Anti-pattern 10 — Believing the model is the problem

**The symptom.** When the agent fails, the default conversation is "we need a better model" or "let's wait for the next version".

**Why it seems like a good idea.** Sometimes it's true. Models improve.

**Why it fails.** Most of the time, the failure isn't in the model — it's in what the model could see (context), in how its output was evaluated (sensors), in the constraints it had available (guides), or in the environment where it executed (sandbox). Waiting for a better model to fix a broken harness is spending time in the wrong place. When the new model arrives, the harness will still be broken and the problems will still be there.

**What to do instead.** First reflex: in the face of any failure, ask "what part of the harness isn't doing its job?". If the honest answer is "the harness is fine, it's the model", then yes, wait for the model. It happens; it's just less common than your team probably thinks.

---

## Closing the guide

If this guide has a single transferable conclusion it's this: **the quality of an agent isn't in the model you use; it's in the system you build around the model**. The model is a commodity. The harness is your competitive advantage.

This is, at bottom, good news. You don't control the model — OpenAI, Anthropic, Google or whoever controls it. You do control the harness. And like everything you control in engineering, it improves with intent, with discipline, and with a team that understands what it's building and why.

Chapters 1 through 10 describe the pillars. This chapter 11 describes the traps. The next version of this guide will add executable templates (example AGENTS.md, `docs/` structure, linters, hooks, sandbox configurations) so you can go from essay to implementation without starting from zero.

In the meantime: start with the loop, invest in isolation, materialize context in the repo, codify the invariants that matter most to you, and when something fails twice, promote it to the harness. The rest comes out by gravity.
