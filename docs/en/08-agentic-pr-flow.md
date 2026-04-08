# 8. The agentic PR flow

The pull request is the atomic unit of work in software. And when agents start generating them seriously — Stripe merges more than a thousand a week, OpenAI has merged fifteen hundred in five months with a small team — all the assumptions of the traditional PR flow blow up. This chapter is about what to replace them with.

## What breaks first

The first assumption that falls is **"every PR goes through a human"**. Not out of ideology, but out of arithmetic. If you have 7 engineers and the agent opens 20 PRs a day per person, that's 140 PRs daily. Assuming a human can do a decent review in 15 minutes, that's 35 human-hours of review per day. The math doesn't work.

The intuitive answer — "then we slow the agent down" — is exactly the wrong one. What you have to do is the opposite: **delegate most of the review to other agents**, and save human attention for cases where judgment actually matters.

## The "Ralph loop" applied to the PR

OpenAI describes its flow as a continuous iteration that sounds like it was written by a Simpsons screenwriter and works better than it sounds. The agent:

1. Makes its change in its isolated worktree.
2. Reviews its own changes locally, before opening anything.
3. Requests reviews from other agents (local and in the cloud), each with a different focus: architecture, security, tests, performance.
4. Reads the comments — whether from other agents or from humans — and responds inline.
5. Applies the corrections it considers valid, argues against the ones it doesn't.
6. Goes back to step 3 until all reviewers are satisfied.
7. Merges (yes, merges itself) when all gates pass.

Human review in this flow doesn't disappear, but **it becomes optional for most PRs**. The human intervenes when a reviewer agent explicitly escalates, when the change touches an area marked as sensitive, or when the responsible engineer chooses to look.

## The four PR categories

In practice, teams that reach this point end up categorizing PRs into four groups, even if they don't always make it explicit:

**1. Trivial — auto-mergeable.** Mechanical refactors, typo corrections, doc updates, lint fixes. The agent opens, the sensors pass, it merges. Zero human attention.

**2. Routine — reviewed by other agents.** The majority of the work. Implementation of a small feature, a fix for a well-bounded bug, additional tests. A reviewer agent reads, comments, the original agent iterates. If they converge, they merge. The human could look but doesn't have to.

**3. Sensitive — human in the loop.** Changes to modules marked as critical (auth, billing, migrations, public contracts). The human reviews, decides, possibly delegates back to the agent to iterate. The "sensitive" mark isn't decided per PR; it's decided by the architecture, encoded in repo metadata.

**4. Decision — human drives from the start.** Changes that require judgment (which endpoint to expose, what price to charge, what to deprecate). The human doesn't review the PR: they define the plan before the agent opens anything. The agent just executes.

The typical proportion observed in mature teams: 60% routine, 25% trivial, 10% sensitive, 5% decision. And the percentages of the first two keep growing as the harness matures.

## Lightweight merge gates

Another assumption that breaks: **"the merge has to be careful"**. OpenAI explicitly describes its merge gates as "minimally blocking". PRs are short, flaky tests are retried instead of blocking progress, and fixes for small breakages arrive in follow-up PRs instead of stopping the flow.

This sounds irresponsible. On a low-performing team it would be. On a team where the agent can fix whatever breaks in minutes, **the cost of waiting is greater than the cost of breaking**. Discipline relocates (again) toward fast detection and cheap reversion. If those two aren't there, don't apply this pattern. If they are, not applying it is leaving money on the table.

There are two non-negotiable prerequisites for this to work:

- **Fast detection.** Sensors in CI that discover regressions in minutes, not hours. Smoke tests, contract tests, type checks.
- **Trivial reversion.** The ability to revert a PR without ceremony, ideally automatic if an alert fires within N minutes of merge.

If you have both, merge gates can be lightweight. If you don't, lightweight merge gates are a recipe for chaos.

## Agent-to-agent review as an inferential sensor

In the language of chapter 3, review by another agent is an *inferential* sensor: it's not deterministic, but it captures things that computational sensors can't. Things like:

- "This name is confusing given the domain."
- "This function doesn't handle the empty case and the context suggests it's important."
- "This change contradicts the decision from the active execution plan in `docs/exec-plans/active/billing-rewrite.md`."
- "This solution is valid but there's an established pattern in `domains/orders/repository.ts` you should reuse."

A human would make these comments, but a human is expensive and doesn't scale. A reviewer agent with good context makes them well most of the time, and the original agent filters the false positives by arguing back.

One important property: **agent-to-agent reviews must have focus**. A single agent trying to review "everything" produces mediocre reviews. Multiple reviewer agents with specialized prompts — one for architecture, another for security, another for tests, another for coherence with active docs — produce much better signal. Stripe formalizes this with its "blueprints" model combining deterministic and agentic nodes in explicit pipelines.

## The human as exception reviewer

When a human does come in to review, their role changes. They're no longer "the quality system", because there are sensors before them. Their role is:

- **Apply judgment that can't be codified.** Product decisions, strategic trade-offs, judgment about whether a solution "fits" the direction of the project.
- **Identify patterns that deserve to be promoted into the harness.** If they're correcting the same thing twice, that's not review, it's a signal that a guide or a sensor is missing. Their most valuable work is translating that observation into an improvement to the harness.
- **Validate areas marked as sensitive** where the cost of a failure justifies the latency.

What the human does *not* have to do in this model: read every line, argue about naming, verify the tests exist, make sure the lint passes. That's sensor work. If the human is doing it, the sensors are incomplete.

## The PR as evidence, not conversation

A subtle shift that appears in these flows: the PR stops being a conversation between humans and becomes **an evidence record**. The original agent attaches:

- The plan it followed (referenced from `exec-plans/active/`).
- The relevant logs from the worktree where it validated.
- Screenshots or videos of the behavior before and after (when it's UI).
- The questions it asked itself and how it resolved them.

OpenAI explicitly mentions that its agent can record a video of the bug, another of the fix, and attach them to the PR. This changes the nature of the review: the reviewer (human or agent) doesn't have to reproduce anything to understand the change. The evidence is already there. If the evidence is missing, the review is legitimately blocked and the agent is asked to add it.

## What this asks of the repo

For this flow to work, the repo needs to be prepared: mechanically-readable "sensitive area" marks (a metadata.yml per domain, for example), live execution plans in a known location, reviewer agents with versioned prompts, and fast sensors in CI. It's work. But it's work that amortizes very quickly when PR throughput goes up.

The final diagnostic question: **if tomorrow your agent opens 50 PRs in a day, what happens?** If the answer is "we'd have to stop the agent because we can't review them", the PR flow isn't ready. If the answer is "the sensors would filter the problems and humans would only see the 3-4 that matter", you're close.
