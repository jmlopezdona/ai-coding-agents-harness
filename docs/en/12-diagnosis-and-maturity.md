# 12. Diagnosis and maturity: where you are and what to tackle first

The previous eleven chapters describe the what and the why. This chapter is the where-you-are. It's designed so a team can sit down for half an hour, answer a handful of questions honestly, identify which dimension of the harness is weakest, and leave with a clear idea of which investment comes first. It's not a universal step-by-step — those don't work, because each team starts from a different place. It's a map with your position marked and an arrow toward the next port.

An important note before we start: **resist the temptation to raise all the dimensions at once**. The dimensions have dependencies among each other; some unblock others. Raising one dimension two levels produces more value than raising five dimensions one level. The asymmetry is real and almost every team underestimates it.

## The five dimensions of the harness

After compressing everything described in the guide, the harness reduces to five dimensions that can be evaluated and improved relatively independently:

1. **Isolation** — the quality of the environments where the agent executes (ch. 5).
2. **Context** — how much of the relevant knowledge lives in the repo in an agent-readable form (ch. 6).
3. **Invariants** — how much of the team's discipline is codified as executable guides, not as implicit culture (ch. 3, 7).
4. **Feedback loop** — how fast and rich the cycle of validation, correction and review is, including the PR flow (ch. 4, 8).
5. **Maintenance** — what mechanisms sustain the codebase against drift months down the road (ch. 9).

Each one is evaluated on five levels, from 0 to 4. The levels aren't grades: they're **behavior descriptions**. The right level for your team isn't necessarily "the highest". For some dimensions, level 2 is reasonable and sufficient. For others, level 2 is where the pain starts.

## Levels, in a table

| Level | Isolation | Context | Invariants | Loop / PR | Maintenance |
|---|---|---|---|---|---|
| **0** | Agent runs on dev machine against the main checkout | Knowledge in Slack, Notion, heads | Cultural discipline, no custom lints | Manual loop: copy/paste to chat | Ad-hoc human cleanup |
| **1** | Container reused across tasks | README + monolithic AGENTS.md | Generic language linters | Agent executes and reports, human validates each step | Cleanup Fridays |
| **2** | Dedicated sandboxes per task, manually created | AGENTS.md as index + partial `docs/` folder | Some custom lints, messages for humans | Automated internal loop, PR reviewed human-to-human | Recurring agents for specific cases |
| **3** | Disposable sandboxes, created in seconds, parallelizable | Structured `docs/`, versioned plans, basic doc-gardening | Custom lints with messages aimed at the agent, mechanized architecture | Agent-to-agent reviews for routine PRs, light merge gates | Golden principles + recurring agents that open refactor PRs |
| **4** | Isolation + ephemeral observability per environment (logs/metrics/traces queryable by the agent) | Repo as complete system of record, agent-readable end-to-end | Executable invariants covering the whole architecture, written by the agent itself | PR flow mostly agent-to-agent, human by exception | Agentic flywheel: agents propose improvements to the harness itself |

Read each column as a ladder. Most teams that have adopted agents are in the **0-2** range on almost everything. The teams that show up in the reference articles (OpenAI, Stripe) are in the **3-4** range, and not uniformly across all dimensions.

## Self-diagnosis: the questions

For each dimension, answer honestly. The trap is to read the questions and answer what you'd like to be true. The useful signal shows up when you answer what is true.

### Isolation

1. Can you launch 5 agents in parallel, each in its own environment, without them stepping on each other?
2. How long does it take for a new environment to be ready, from when you ask for it until the agent can execute inside? (seconds / minutes / hours)
3. When an environment is destroyed, are there any orphaned resources? Any manual cleanup step?
4. Can the agent run tests, bring up the app, query logs *inside* the isolated environment?
5. Do two creations of the same environment produce an identical state?

**Level = the lowest one where you fail any question**. If you fail #1, you're in 0-1. If you only fail #5, you're at 3.

### Context

1. If a new engineer joined tomorrow without access to Slack, Google Docs or meetings, could they be productive with just the repo?
2. When an architectural decision is made, where is it recorded? Is it machine-readable?
3. Is your AGENTS.md more than 200 lines long?
4. Is there an explicit technical debt tracker the agent can read?
5. Is there any process (lint, recurring agent) that detects when a doc has diverged from the code?

### Invariants

1. How many custom lints do you have specific to your project? (0 / 1-5 / 5-20 / 20+)
2. The rule "this layer can only import from these others", is it validated by code or by human convention?
3. When the agent makes a pattern mistake, how many times do you fix it before promoting it to a lint?
4. Are the error messages of your custom linters written so an agent can understand them and apply the remedy?
5. Are style invariants (structured logging, naming, file size) enforced or documented?

### Loop / PR flow

1. Can the agent run tests, see the output and iterate without a human intervening?
2. How long does the typical internal loop take (edit → typecheck → test → next edit)?
3. Are there agent-to-agent reviews in your PR flow, or is everything reviewed by a human?
4. How many PR categories do you have defined (trivial/routine/sensitive/decision), or do all PRs go through the same path?
5. When a test is flaky, does it retry automatically or does it block merge indefinitely?

### Maintenance

1. What proportion of the team's time is spent "cleaning up what the agent did wrong"?
2. Are there recurring agents running in background that open small refactor PRs?
3. Do you have "golden principles" / critical conventions, written in some place the agent reads automatically?
4. When the same error appears for the third time, what happens?
5. Does any agent analyze the harness itself and propose improvements?

## Investment matrix: current level → next step

Once you have your five approximate levels, this matrix tells you which concrete investment unlocks the next level for each dimension. The idea isn't to raise them all at once: it's to identify the dimension where you're lowest and start there.

### Isolation

| From | To | Investment |
|---|---|---|
| 0 → 1 | Some basic isolation | Project container, documented, that any dev can bring up. Nothing fancy. |
| 1 → 2 | Dedicated sandboxes | A script (or tool) that creates one environment per task. Manual is fine. |
| 2 → 3 | Disposable and parallelizable sandboxes | Automated creation/destruction. Speed < 30s. Test: launch 5 in parallel and see if any collide. |
| 3 → 4 | Ephemeral observability per environment | Logs/metrics/traces queryable by the agent *inside* the sandbox. Stack like Vector + Victoria, or equivalent. |

### Context

| From | To | Investment |
|---|---|---|
| 0 → 1 | Basic AGENTS.md + README | Write an AGENTS.md, even a monolithic one. Any starting point is better than none. |
| 1 → 2 | Reduce AGENTS.md to an index, move the rest to `docs/` | Create `docs/` with sections (`design-docs`, `product-specs`, `references`, `tech-debt-tracker.md`). |
| 2 → 3 | Versioned plans + doc-gardening | Move active execution plans into the repo. Launch a recurring agent that detects obsolete docs. |
| 3 → 4 | Repo as complete system of record | Any knowledge living outside the repo (Slack, Notion, GDocs) gets materialized. Lints validate freshness. |

### Invariants

| From | To | Investment |
|---|---|---|
| 0 → 1 | Generic linters in CI | Enable and harden the standard tools of your stack (eslint, ruff, golangci-lint, etc.). |
| 1 → 2 | First custom lints | Identify 2-3 patterns that recur in reviews and turn them into rules. Start as warnings. |
| 2 → 3 | Custom lints with messages for the agent + mechanized architecture | Rewrite error messages as instructions. Validate dependency directions between layers. |
| 3 → 4 | Complete invariant coverage, written by the agent | The agent itself proposes and maintains the lints. 100% coverage of architectural invariants. |

### Loop / PR flow

| From | To | Investment |
|---|---|---|
| 0 → 1 | Minimal internal loop | The agent can run `test` and see the result. No human intervention at every step. |
| 1 → 2 | Full internal loop + automated PR | The agent runs typecheck/test/lint in a loop until green. Opens a PR. Human reviews before merge. |
| 2 → 3 | PR categorization + agent-to-agent reviews | Define the 4 categories (trivial/routine/sensitive/decision). Introduce a reviewer agent for the routine ones. |
| 3 → 4 | PR flow mostly agent-to-agent | Multiple specialized reviewer agents. Human intervenes by exception. Light merge gates with fast rollback. |

### Maintenance

| From | To | Investment |
|---|---|---|
| 0 → 1 | Explicit cleanup (worse than nothing, but it's a step) | Reserve recurring time to clean drift. Accept it doesn't scale — it's a starting point. |
| 1 → 2 | Recurring agents for specific cases | Start with one: doc-gardening, or an agent that detects orphan TODOs, or one that updates dependencies. |
| 2 → 3 | Golden principles + fleet of recurring agents | Codify the 5-10 non-negotiable rules. Several background agents, each with a focus. |
| 3 → 4 | Agentic flywheel | A meta agent reads the accumulated signal from the sensors and proposes improvements to the harness itself. |

## How to prioritize between dimensions

If you're low on several dimensions at once (most teams are), use this heuristic — in this order — to decide which to attack first.

**1. If you're at level 0 or 1 on Isolation, start there.** Without isolation, almost every other investment returns half what it should. It's the dimension with the most dependencies into the others and the one with the highest multiplier ROI. It's not negotiable: if you don't have sandboxes, everything else stays stuck.

**2. If Isolation ≥ 2 but Context is at 0 or 1, attack Context.** An agent with a good sandbox but bad context produces code that's syntactically correct and ideologically wrong. Raising Context to 2 already eliminates most architectural divergences.

**3. If Isolation and Context ≥ 2, look at which you're lowest on: Invariants or Loop.** Both are important and reinforce each other. As a heuristic: if your team complains more about "the agent doesn't follow conventions", raise Invariants. If they complain more about "the agent takes too long to iterate" or "I have to review everything by hand", raise Loop.

**4. Maintenance is last, but not optional.** Below level 2 in Maintenance, everything you've built is going to degrade. The good news: when you raise the other dimensions to 2-3, Maintenance goes up partially on its own, because the recurring agents have something to operate on.

**Negative rule:** don't try to jump two levels in a dimension at once. Each jump changes the team's processes, and your team needs time to settle at the previous level before going up. One month per jump is a reasonable pace. Faster and it breaks.

## Your personal roadmap

If you want to take a single artifact away from this chapter, it's this template. Run it with your team in a 60-90 minute session and fill it in honestly:

| Dimension | Current level | Next level | Concrete investment | Estimated timeline |
|---|---|---|---|---|
| Isolation | | | | |
| Context | | | | |
| Invariants | | | | |
| Loop / PR | | | | |
| Maintenance | | | | |

And then, without discussion, **circle the row you're going to attack this month**. Only one. The rule is disciplinary: if you circle two, you circle none.

When you finish that investment, come back to this template, re-evaluate (probably your current level has gone up on some dimensions as a side effect), and pick the next. There's no "finished harness" — there's a harness in continuous improvement, which is exactly the kind of system this guide describes in the rest of the chapters.

## The real closing of the guide

There's an interesting property of well-built harnesses: **they don't get noticed when they work**. The team ships fast, the code stays coherent, the agents produce useful things, the humans focus on what matters, and none of it feels like "we're doing something special". It just feels like engineering done well. The teams that have it rarely write about it because from the inside it looks obvious.

The teams that don't have it yet look from the outside and attribute the difference to the model. The model is the most visible thing — it has a brand, a version, a parameter count. The harness is invisible. And precisely for that reason it's where the leverage is: because everyone else is looking in the wrong place.

If you only take one thing from the twelve chapters, take this: **your next investment isn't waiting for a better model. It's building the harness your current model can already take advantage of**. There's a concrete next step. It's in the row you just circled. Do it this month.
