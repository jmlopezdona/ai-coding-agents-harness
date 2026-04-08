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

## What if I use an MCP server instead of putting it in the repo?

This is the question that comes up the moment a team has good living documentation in Notion, Confluence or Google Docs and faces the cost of moving it. The intuition is reasonable: if there's an MCP server that exposes that documentation to the agent, why duplicate it in the repo? The agent queries it live, humans keep editing where they already edit, zero sync, zero drift. It looks like the elegant solution.

The answer isn't "don't do it". It's "understand the price you're paying and where it applies". There are four possible strategies for getting an agent access to documentation that doesn't originate in the repo, and each one pays a different price:

### The four options, in order of "purity"

**1. Doc only in the repo (versioned markdown).** A single source. Humans edit in the editor or via the GitHub web UI. The agent reads from the filesystem. Zero ambiguity, zero drift, everything audited by commit. It's what the rest of this chapter argues for. **Cost:** non-engineers (PMs, designers, legal, support) have to learn git or use the GitHub web UI, and you lose features of modern wikis (inline comments, mentions, embeds, rich templates). For many teams, politically unviable.

**2. Point-in-time import (copy to repo when the doc is closed).** The doc is born in Notion, humans collaborate there, and once it's considered "stable" it's exported to markdown and pushed to the repo. **Cost:** works for *finished* docs (closed RFCs, post-mortems, settled decisions). For *living* docs it's a trap: the next day Notion has already diverged and nobody is going to re-export.

**3. Automatic sync (mirror Notion → repo).** A job or webhook detects changes in Notion and exports them to markdown in the repo continuously. Humans keep editing where they already edit; the agent reads from the repo (fast, deterministic, versioned). The best of both worlds *in appearance*. **Cost:** sync has a window — between the human's push and the pull to the repo, the repo is stale. More infra to maintain (hooks, converters, error handling). Notion/Confluence to markdown conversion loses nuance: embedded databases, comments, dynamic templates. When sync fails, it fails silently and the agent keeps reading old data without knowing it's old.

**4. MCP server with live semantic search.** At runtime, the agent calls an MCP that queries Notion (or whatever) and returns the most relevant fragments for its query. Zero duplication, zero sync, zero drift. The doc lives where it was born. **Cost:** this is where the problems are most underestimated. Let's get into it.

### Why an MCP server isn't a substitute for the repo (for what's critical)

Semantic search via MCP introduces several properties that are acceptable for some uses and disastrous for others:

**Non-deterministic.** Two identical queries can return different results. Semantic search depends on the embedding model, the ranking, the corpus indexed at that moment. The agent may read the right doc in one execution and *not read it* in the next. For a sensor or an invariant, this is unacceptable: "the agent sometimes sees the security rule and sometimes doesn't" isn't a rule, it's Russian roulette.

**Not versioned.** There's no commit that says "the agent's context changed here". If the MCP's response changes tomorrow because someone edited a doc in Notion, and that breaks the agent, there's no easy way to bisect. In the repo model, every change that affects the agent lands in `git log`; in the MCP model, changes are invisible until they produce a failure.

**Latency and external dependency.** Each query is a round-trip to an external API that can be down, slow, or rate-limited. If Notion goes down, your agent is partially blind — and doesn't necessarily know it. The repo, on the other hand, is local to the sandbox: it's essentially instant and never "goes down".

**The decision of what to include is made at query-time, not authoring-time.** This is subtle but it's the most important point. When you *write* a doc for the repo, you actively decide what information will be accessible to the agent and what shape it has. That contract is explicit and reviewable in code review. When the agent *searches*, the subset it receives is decided by an opaque ranker at runtime. You may have written the perfect doc and the ranker decides another one is more relevant. For invariants — where you need to guarantee the agent sees X no matter what — this breaks the model.

**Permissions and authentication complicate the sandbox.** For the MCP to work, the agent needs credentials with scopes you would normally reserve for humans. That widens the attack surface of the sandbox and puts the agent in a perimeter where a prompt injection can pivot against the doc store. It's a real security trade-off, not a theoretical one.

**Token cost and time on every execution.** Each MCP call burns agent context and API tokens. The repo is free: reading a file is essentially zero cost. At scale (hundreds or thousands of invocations per day), the difference adds up.

### When yes, when no

The heuristic that comes out of all this is simple and worth internalizing:

> **The critical stuff in the repo. The optional stuff via MCP. Sync only when the critical stuff can't be moved and you accept paying the staleness window.**

Operationally:

- **If the agent has to see a doc to avoid breaking something** — architecture invariants, irreversible decisions, golden principles, schema contracts, security rules — that doc goes in the repo, no exceptions. It is not entrusted to probabilistic search, because probabilistic search fails silently 1% of the time, and 1% of failures at agent scale is a lot of failures.
- **If the doc is "nice to have, helps the agent contextualize"** — the history of a module, the rationale of an old decision, notes from a broad domain where the agent can benefit but not depend — an MCP server against Notion is a valid and cheap option. Semantic search shines in this long-tail: it finds things you hadn't manually indexed.
- **If the critical stuff lives in an external tool you can't move for political or organizational reasons**, sync to the repo and add a freshness lint that fails if the mirror has gone more than N hours without updating. The lint turns the sync into a visible invariant: when sync breaks, you find out because the build fails, not because the agent starts making decisions on weeks-old data.

### The most dangerous trap

It's worth naming because it's where most teams fall: **syncing without a freshness check**. It's the combination that looks responsible — "we have a job that copies Notion to the repo every hour" — and gives the false sense that you're covered. But between the last sync and the next, anything can happen, and when the sync breaks (and all syncs break sooner or later), nobody finds out until an agent makes a decision based on three-week-old information. If you're going with sync, the freshness lint is not optional. It's what distinguishes a serious sync from a ceremonial one.

### The principle behind all of this

The whole of chapter 6 defends one idea: the repo is the system of record because it is **deterministic, versioned, auditable, and always available**. Any alternative that loses one of those four properties is acceptable only where that property wasn't critical. An MCP server with semantic search loses all four at once, and that's why it only fits as a complement — never as a replacement — for the knowledge the agent *must* have in front of it to avoid breaking something.

Put differently: MCP is a good way to *enrich* the agent's context. It's not a good way to *guarantee* the agent's context. The difference between enriching and guaranteeing is the difference between an assistant that is sometimes brilliant and a system a team can rely on.
