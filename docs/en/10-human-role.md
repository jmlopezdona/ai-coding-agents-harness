# 10. Where humans still matter

After nine chapters describing how to automate everything automatable, it's worth stopping and asking the opposite question: **what does the human still have to do?**. The answer isn't "supervise the agent", even though that's what many teams end up doing by inertia. The right answer requires understanding that the engineer's role changes, it doesn't shrink.

## Where the human is: a quick recap

Ch. 0 introduced the three delegation modes Kief Morris articulates — *outside the loop* (vibe coding), *in the loop* (per-artifact supervision), and *on the loop* (system supervision) — and argued that "on the loop" is the only regime that scales: the human doesn't inspect each output but modifies the system that produces them when things go wrong.

We take that discussion as closed. The question this chapter organizes is the next one: **if you're "on the loop", what exactly do you do with your time?** Because "supervise the system" is a nice phrase that, left unbroken-down, leaves half the engineers on the team not knowing what to do on Monday morning.

## The two cycles: why and how

Another distinction from Morris that helps locate the human role: there are two cycles at play, and it's worth not confusing them.

- The **"why" cycle** is where an idea gets turned into software that matters. It involves understanding what you're trying to achieve, for whom, under what constraints, with what trade-offs, in what order. It's deeply human work and it doesn't get delegated.
- The **"how" cycle** is where the intermediate artifacts get produced — code, tests, configs, docs — that materialize the "why". It's the cycle where agents shine and where most of this guide applies.

The most common trap is filling human time with the "how" and neglecting the "why". A team that delegates the "how" well frees up the scarce bottleneck (human attention) to invest it where it really matters (understanding what to build and why). A team stuck reviewing "how" artifacts isn't just slow; it's using its humans wrong.

## What changes, not what disappears

Böckeler insists on a point that's easy to misread: the harness doesn't eliminate the human, it *redirects* them. Developers bring three things the agent, by construction, does not have:

- **Implicit experience:** patterns learned on other projects, intuitions formed through previous mistakes, the gut feeling of "this is going to hurt in six months" that doesn't derive from the code you're looking at.
- **Organizational awareness:** knowing what product wants, what the CFO cares about, which decisions have political context, which stakeholder will push back.
- **Social accountability:** somebody has to sign off when something goes to production that affects real users. An agent can't sign off.

These three things aren't automatable, not because the model isn't capable, but because they aren't technical problems. They're human problems.

## The new work

Ch. 0 already enumerated the shift — from writing code to designing environments, from reviewing PRs to designing the reviewers, from implementing the domain to specifying it, from fixing bugs to diagnosing why the agent made them — and summed it up with OpenAI's phrase: *"people direct, agents execute"*. It's meta-engineering, and the job description starts to look more like "platform engineer for non-human collaborators" than "programmer".

What Ch. 0 didn't do is break it down to the level of "what fills your hours". That's next.

## The four human interventions that matter

When you filter out all the noise and look at what humans actually do in mature teams with agents, four categories of intervention emerge. The first three are the ones that add value; the fourth is the one to minimize.

### 1. Specify intent

The agent can implement almost anything. What it can't do is decide what's *worth* implementing. Turning a vague problem into an actionable goal, with verifiable acceptance criteria, is still human work. And it's probably the highest-leverage work: a clear specification saves hours of misdirected loops.

The concrete form varies — it can be a versioned execution plan, a well-written ticket, a detailed prompt — but the content has to have three things: what you want to achieve, how you'll know it's done, and what *not* to do. The third is the one most often omitted and the one that prevents the most drift.

### 2. Calibrate the harness

When the agent fails repeatedly, somebody has to ask the important question: what is the harness missing? This requires understanding the failure at a level that isn't just "the code is wrong", but "the agent didn't have the information it needed" or "the sensor didn't catch this" or "the convention wasn't codified". The human makes this diagnosis and promotes the lesson into the harness — as a new lint, a new guide, a new reviewer agent, or a new section in docs.

This is the work where a senior multiplies their impact the most. Every hour invested here compounds over thousands of future agent runs. It's the best-ROI investment on the team.

### 3. Decide the irreversible

There are decisions whose cost of error is asymmetric: deprecating a public API, changing a database schema in production, modifying a contract with a vendor, accepting a security trade-off. In these cases you don't want speed, you want deliberation. And deliberation needs judgment, organizational context, and accountability — the three human things we listed at the start.

The operational rule: **the more irreversible the decision, the more human has to be in the loop, and earlier**. Not at the end, reviewing a PR that's already been written. At the beginning, choosing whether to do it and how.

### 4. Review what already got through the filters

This is the category to minimize, not eliminate. The human reviews when:

- A reviewer agent explicitly escalates because it isn't sure.
- The change touches an area marked as sensitive.
- The engineer on point chooses to look in order to keep their mental model current.

The key thing: **it's not the first line of defense**. If the human is the main quality filter, the sensors are incomplete. Human review is the final safety net, not the primary mechanism.

## What to stop doing

There are three habits many teams drag over from the pre-agent flow and that should be consciously dismantled:

**Reviewing every PR.** As we said in chapter 8, it doesn't scale. More importantly: **it gives a false sense of control**. If you review everything on autopilot, you're not reviewing anything well. Better to review the 10% that matters with care than 100% superficially.

**Arguing over style.** If the agent's output doesn't respect a stylistic preference and no lint captures it, you have two options: codify the preference as a lint, or accept the output. Arguing about it on every PR is pure waste.

**Asking the agent to "do things the way you would".** The agent isn't you. What you want is for the things it does to satisfy the invariants that matter. If the invariants are well captured, the agent's "style" is irrelevant. If they aren't, codifying them is more useful than asking.

## The identity question

There's an uncomfortable moment almost every engineer experiences when adopting this model: if I'm not writing code anymore, what am I? The honest answer is you're still an engineer, but the medium your work comes out of has changed. Before, you produced code; now you produce **the conditions that make correct code possible**. It's engineering of a different order, not less.

The skills that become valuable are the ones good seniors already valued: technical taste, systems thinking, clarity when specifying, ability to design interfaces (human or non-human), judgment about when to invest in infrastructure. The skills that lose relative value are the purely artisanal ones: fast typing, API memorization, mastery of exotic syntax.

It's not a bad trade overall. But it requires a realignment of professional identity that shouldn't be minimized. Teams that ignore this have unhappy engineers doing "the new work" resentfully. Teams that address it explicitly — "this is the new work, this is the leverage, these are the new values" — turn their seniors into multipliers instead of bottlenecks.

## The closing

The human is still the center of the system. But the center has changed shape. You're not in the center anymore because you write all the code; you're in the center because you design the system within which the code writes itself, and because when that system fails, you're the one who decides what failed and what fixing it means.

Machines can generate quantity. Only humans can decide what quantity deserves to exist, and under what conditions it's allowed to exist. That isn't going to be automated soon. And while it stays true, the engineer stays at the center — only at a higher level of abstraction than they're used to.
