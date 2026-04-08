# 9. Entropy and continuous maintenance

This is where most teams discover, around month three or four, that they had a problem they hadn't anticipated. The innocent question — "if the agent writes so fast, what's the code going to look like in six months?" — has an unsettling default answer: **chaotic, inconsistent and fragile**, unless you design an explicit system to prevent it. That system is what this chapter describes.

## The source of the problem

Agents are remarkably good at imitating the patterns that already exist in the repo. That's a virtue — they inherit conventions, maintain local consistency, don't invent new styles every hour — and it's also the source of the problem. Because they imitate **all** the patterns, including the irregular ones, the obsolete ones, the ones somebody introduced on a Friday at 6pm.

OpenAI describes the cycle like this: the agent replicates existing patterns, even the suboptimal ones. Over time, this produces drift. And drift in an agent-generated codebase is qualitatively different from drift in a human one: it's faster, more consistent (that is, errors are replicated more faithfully), and harder to catch by casual review because the code still "looks" coherent.

It's a bit like a photocopier that picks up a small mark on the glass and starts putting it on every copy. If nobody cleans the glass, in six months all the copies have the mark and nobody remembers why.

## The naive response and why it fails

The typical first reaction is cleanup Friday. One day a week, the team sits down to clean up what the agent did wrong. OpenAI describes trying exactly this and abandoning it: they used to dedicate Fridays (20% of the week) to "AI slop cleanup", and it simply didn't scale.

The reasons are obvious in hindsight:

- **Agent throughput grows faster than human cleanup capacity.** The better the harness, the worse this model scales.
- **The knowledge of "what's wrong" stays in the head of whoever is cleaning.** It doesn't translate into harness improvements. The next week, the agent makes the same mistakes.
- **Cleanup is low-morale work.** The most senior engineers avoid it or do it poorly. Cleanup quality decays until it's no longer worth it.

The inevitable conclusion: **maintenance has to be part of the loop, not a separate event**.

## Continuous garbage collection

OpenAI calls their solution "garbage collection" and the analogy is exact. Instead of accumulating trash and dedicating time to throwing it out, you have a process that picks it up continuously, in small amounts, while the codebase keeps working.

In practice this looks like:

**Codified Golden Principles.** Mechanical, deliberately prescriptive rules that capture the team's "taste" in a form the agent can read. Not "we prefer clean code", but "we prefer shared utility packages over ad-hoc helpers, to keep invariants centralized". Each principle is documented *with its why*, so the agent can apply judgment in edge cases.

**Recurring background agents.** Codex tasks (or equivalents) that run on a regular cadence — every night, every week — and do one thing each: detect drift against a specific principle, update quality scores, open small refactor PRs. Each PR is reviewable in under a minute and auto-merges if it passes the sensors.

**Doc-gardening.** A recurring agent that scans documentation and compares it to the actual behavior of the code. When it detects divergence, it opens a PR that updates the doc or (if the doc is the correct source) flags it to the human. It's the equivalent of a doc linter, but inferential.

**Explicit debt tracker.** A file in the repo (`tech-debt-tracker.md` in OpenAI's case) that lists what's wrong on purpose. This serves two functions: it tells the agent what *not* to fix (what's wrong on purpose) and what *to* fix when it has capacity. Without this file, the doc-gardener "fixes" things the team had decided to leave alone.

## The philosophy: debt as a high-interest loan

The metaphor worth internalizing (again from OpenAI): **technical debt is a high-interest loan; it's almost always better to pay continuously in small increments than to let it accumulate and tackle it in painful mountains**.

This changes several operational things:

- **You don't wait for "the next cleanup sprint".** It doesn't exist. Cleanup is ambient.
- **You don't batch big refactors.** Small refactor today + small refactor tomorrow > huge refactor in six weeks.
- **The value of the small refactor isn't just better code.** It's that it keeps the codebase in a state where the agent's next change is more likely to be correct. Refactors are investments in the harness, not just in the code.

## "Human taste" captured once

There's a phrase worth reading twice: *"human taste is captured once and then applied continuously on every line of code"*. It's the difference between a team where the senior has to watch everything, and a team where the senior dedicates time to **translating their taste into executable rules** and then lets the system apply them.

This makes every hour of senior work radically more valuable. In a traditional flow, a review consumes 30 minutes of the senior and improves *one* PR. In a well-done agentic flow, those same 30 minutes are invested in writing a golden principle or a custom lint, and they improve *all future PRs*. The multiplier is huge and compounds over time.

The diagnostic question for a senior: how much of the time you spend on quality ends up as code that runs automatically, and how much ends up as knowledge in your head that evaporates when you leave the team? The higher the first ratio, the healthier the harness.

## The three layers of maintenance

It's useful to think of maintenance as three layers that operate on different time scales:

**Layer 1 — sensors in CI (seconds to minutes).** They detect the obvious problems at PR time. Lints, tests, type checks. This layer is the first line and the cheapest.

**Layer 2 — recurring agents (hours to days).** They detect accumulated drift, doc rot, replicated patterns, consolidation opportunities. They run in background, don't block anyone, open reviewable PRs.

**Layer 3 — occasional re-architecture (weeks).** Structural changes that require human judgment. When the golden principles change, when a whole layer needs redesigning, when a library has to be replaced. These still need humans in the driver seat — but they're rare if the first two layers work.

If you're missing layer 1, the agent introduces errors constantly. If you're missing layer 2, the errors layer 1 doesn't catch accumulate silently. If you're missing layer 3, eventually you hit an architectural ceiling no agent can break through. All three are necessary; none is sufficient.

## The signal that it's working

A team that has this right usually notices two things within a few months:

1. **Agent PRs get shorter, not longer.** Because the context is better structured, most changes are small and targeted. Thousand-line mega-PRs are a symptom of an immature harness.

2. **Recurring problems disappear without anyone chasing them.** Because every recurring problem produced a guide or a sensor when it was spotted the second time. The team's memory lives in the harness, not in people's heads.

If instead you observe PRs getting longer, the same errors still showing up, and the team spending more and more time "fixing what the agent broke", the harness is losing. The right investment isn't "more human cleanup" — it's promoting what you learn during cleanup into the harness, until cleanup isn't needed.

## The agentic flywheel: when the harness maintains itself

There's a point of maturation worth naming because it's where everything described in this chapter naturally points. Kief Morris calls it the **agentic flywheel**: agents that don't just write code, but **analyze results and recommend improvements to the harness itself**, and even implement them under strategic human supervision.

The loop becomes something like:

1. Sensors detect a recurring pattern (a type of failure, a class of drift, a doc-gardening miss).
2. A meta agent reads the accumulated signal and proposes an improvement: "we've fixed this pattern seven times this month; I suggest the following custom lint" or "this doc has diverged from the code three times; I suggest this restructuring".
3. The proposal arrives at the human as a PR on the harness itself — not on the application code.
4. The human approves, adjusts or rejects with arguments, and the harness improves.
5. The next detection cycle starts from a more capable harness.

This sounds ambitious and should be calibrated: it's not the starting point, it's the direction. A team that starts by trying to build the complete flywheel falls into anti-pattern 7 (megalomania of the initial plan). But a team that disciplines itself into promoting lessons into the harness ends up, almost by gravity, reaching a point where improvement proposals also get generated automatically. The flywheel isn't designed; it's discovered once you were already doing almost everything it requires.

The signal you're close: more than 50% of the PRs touching the harness are opened by an agent, and the human's work is reduced to approving, adjusting or vetoing with judgment. When you get there, the human role has completed its transition — and what you've built is a system that learns to improve itself.
