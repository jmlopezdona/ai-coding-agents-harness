# 6. Context as a system of record

There's a line from OpenAI's Codex team worth painting on the team wall: **"what the agent can't see doesn't exist"**. It's a literal statement about how a coding agent operates, and it has direct consequences for where a project's knowledge needs to live. The short answer: in the repo, versioned, mechanically accessible. The long answer is this chapter.

## The mindset shift

On human teams, knowledge lives scattered across many places: in the README, sure, but also in last Wednesday's Slack, in a Google Doc linked from some email, in the senior's head, in a Linear ticket from six months ago, in a hallway conversation. This distribution works — badly, but it works — because humans know how to ask, cross-reference sources, and use implicit context.

An agent doesn't. The agent only sees what's in its effective context at the moment it acts. If the decision "don't touch this module, we're going to deprecate it" lives only in a Slack message from March 14th, for the agent that decision doesn't exist. It's not that it ignores it: it's that it's not accessible.

The consequence is uncomfortable and liberating at once: **everything you want the agent to know has to be materialized in the repo or mechanically accessible from it**. Not "documented somewhere". Not "mentioned in a meeting". Materialized, in markdown, indexed, maintained.

## The "giant AGENTS.md" mistake

Many teams' first reaction when they discover this is reasonable: cram everything important into a huge AGENTS.md. It's exactly the trap OpenAI describes having tried and abandoned. Four reasons it fails:

**1. Context is a scarce resource.** A 1,000-line manual overshadows the task, the relevant code, and the tools. The model has two options: ignore key constraints, or optimize the wrong ones. Both are bad.

**2. If everything is important, nothing is.** When everything in the AGENTS.md is in bold, the agent has no way to prioritize. It ends up applying local rules without understanding the global intent.

**3. It rots instantly.** A monolithic manual becomes a graveyard of obsolete rules. The agent doesn't know what's still true, humans stop maintaining it, and slowly it becomes an active risk.

**4. It's not verifiable.** A large block of prose doesn't lend itself to automatic checks: you can't validate coverage, freshness, ownership, or cross-references. Drift is inevitable.

OpenAI's conclusion is elegant: **treat AGENTS.md as an index, not an encyclopedia**. ~100 lines, pointing to where each thing actually lives.

## The repo as system of record

What goes into the repo, organized, is the following. This isn't prescriptive — the details vary by team — but the shape recurs:

- **Versioned design docs**, indexed, with a verification status. No "doc linked in Notion, Confluence or Google Docs". The doc lives in the repo, next to the code it describes.
- **Execution plans as first-class artifacts.** For small changes, lightweight ephemeral plans. For complex work, versioned plans with state: active, completed, abandoned, with reasons. This lets the agent see not just what's happening now, but what was tried before and why it was discarded. It's a memory tool months down the road.
- **Tech debt tracker.** A file, in the repo, that lists what's wrong on purpose. Without this, the agent "cleans up" things it shouldn't touch, or ignores things it should fix. With this, both categories are legible.
- **Product specs.** Not Linear, Jira or GitHub Issues epics. The specs *as markdown in the repo*, next to the code that implements them. If the ticket is the only source, the agent only sees what's in the ticket when you pass it in.
- **External references in `llms.txt` format.** External library documentation, internal security standards, platform guides. Imported into the repo, not linked. A URL in an AGENTS.md is a broken promise; a file in `references/` is a source.
- **Architecture map.** Layers, domains, responsibilities, dependency directions. More on this in chapter 7.
- **Quality score per domain.** OpenAI maintains a `QUALITY_SCORE.md` per domain that records known gaps. It's both documentation and a mechanically prioritizable backlog.

The concrete structure matters less than the principle: **folders in the repo, not external services; markdown, not PDFs; maintained by linters and recurring agents, not by human goodwill**.

## How to keep this from rotting

The reasonable fear is: "if I put all this in the repo, in six months it'll be stale". It's a correct fear, and without a solution it becomes reality. What works:

**Documentation-specific linters.** They validate that internal links aren't broken, that referenced file names exist, that active plans haven't gone more than N days without an update, that documents have the minimum metadata fields. It's boring work and it multiplies the ROI of everything else.

**Recurring doc-gardening agents.** This is a pattern that shows up both in OpenAI and, implicitly, in Stripe's flow. An agent that scans documentation against the real behavior of the code and opens PRs when it detects divergence. It doesn't "delete" stale docs automatically: it flags them and lets a human (or the agent that modified it) decide.

**Promote the rule to code.** When a piece of documentation describes a pattern that should be mandatory, turn it into a lint. The doc stops being the source and becomes the explanation; the lint is the enforcement. OpenAI's original rule: *"when documentation isn't enough, we promote the rule to code"*.

## The definitive test

When you're in doubt whether a piece of information should go in the repo or can live outside it, ask yourself this: **if a new engineer joined tomorrow without access to Slack, without access to Google Docs and without being able to talk to anyone on the team, could they be productive with only what's in the repo?**

If the answer is no, that information is invisible to the agent too. And on a serious team operating with agents, that new "engineer" joins a hundred times a day. Every agent run is an onboarding from zero.

## The hidden upside

There's a side effect almost nobody anticipates: **by making the repo legible for an agent, you make it enormously more legible for humans too**. Doc-gardening benefits you. Versioned plans benefit you. The debt tracker benefits you. Humans joining the team onboard faster. Seniors stop being context bottlenecks. What looked like a concession to the machine ends up being, in large part, platform engineering for the team itself.

Stripe puts it directly, and it's worth closing with their line: *"what's good for humans is good for agents"*. The historical investment in developer experience pays dividends for the agent too, and the investment in the agent pays dividends for developer experience. The loop is virtuous if you let it be.
