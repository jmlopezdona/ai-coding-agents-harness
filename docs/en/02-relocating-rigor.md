# 2. Relocating rigor

Every time software has taken a major leap, someone has shown up to announce that "we don't need so much rigor anymore". And every time they've been wrong. Rigor doesn't disappear; **it relocates**. This is probably the most useful observation Chad Fowler makes about the generative-AI era, and it's worth keeping in mind before you start dismantling practices that look "no longer necessary".

## The historical pattern

Three examples tend to be cited:

1. **Dynamic languages vs static type systems.** When Ruby and Python became popular in the 2000s, plenty of people declared typing dead. It didn't die: rigor moved to the *test suite*. Serious Ruby teams ended up writing more tests per line of code than Java teams of the same era. Discipline didn't evaporate, it changed location.
2. **Extreme Programming vs waterfall planning.** Eliminating detailed plans and long phases didn't mean "winging it". It meant moving rigor to continuous integration, disciplined refactoring and immediate feedback. What you stopped doing in a Gantt you started doing on every commit.
3. **Continuous deployment vs release windows.** Removing the release ceremony didn't relax anything: it demanded first-rate observability, immediate reversibility, exhaustive automation and feature flags. Rigor moved to the runtime and to the platform.

In all three cases, whoever interpreted the change as "now we can be more relaxed" lost. Whoever understood where discipline had moved to, won.

## Where rigor moves with coding agents

With LLMs writing code, rigor moves out of two places and into two others:

**Moves out of:**

- Writing every line with artisanal care.
- Reviewing every PR human-to-human as the primary mechanism of quality.

**Moves into:**

- **Specification of intent.** What used to be "informal acceptance criteria" in a ticket now has to be mechanically readable, because that's what the agent is going to interpret. Imprecision in the ticket becomes imprecision in the code. Before, you could compensate for it with a hallway conversation; with an agent running overnight, you can't.
- **Verifiable evaluation.** If you can't mechanically evaluate whether the agent's output is correct, you don't have a harness, you have a roulette wheel. Tests, type checks, evals, schema validation, observability, runtime asserts — everything that turns a "looks like it works" into a binary signal.

Chad Fowler says it better: *"generative systems only work if invariants are explicit rather than implicit"*. A generative system needs the things that have to be true to be stated somewhere the machine can check. The implicit — the knowledge that lived in the senior's head, in the README from two years ago, in Monday's Slack conversation — stops working as a control mechanism.

## The uncomfortable consequence

This means that the teams that had more of their discipline **codified** (types, tests, schemas, IaC, structured observability) are better positioned to use agents than the teams that relied on **cultural** discipline (careful reviews, attentive seniors, unwritten standards). Not because the latter are worse, but because the agent can't access their discipline. It's invisible to it.

The practical corollary is that **two repos with equivalent human quality can be very unequally prepared for agents**. The one with discipline materialized in linters, schemas and tests works; the one with it in the seniors' heads doesn't. The difference isn't visible until you bring in an agent and find out — and by then it's too late to pretend it wasn't there.

## The warning sign

Chad Fowler proposes a heuristic worth adopting: **when something seems to let go of rigor, look for where it relocated. If you can't find it, worry**.

Applied to agents:

- "The agent writes the tests now." Who validates that the tests *actually* test what they say they test? Through what mechanism? If the answer is "an engineer reviews them", rigor hasn't disappeared in theory — but at the volumes an agent produces, in practice nobody is going to read them with the attention required. You haven't relocated rigor to a place where it can be sustained; you've left it in a place where it's going to evaporate through neglect.
- "We don't need docs anymore, the agent reads the code." How is the agent going to know which design decisions it *cannot* revert? Code doesn't tell you why you *didn't* do something. If that information isn't in some readable place, rigor is gone.
- "We merge fast and the agent fixes what breaks." Fine, but then you need a regression-detection system that's faster and more sensitive than before. If you have it, great. If not, you've traded discipline for speed and the speed is going to disappear in three months, when the debt catches up with you.

## The practical corollary

Before adopting an agent seriously, take inventory of the things your team was implicitly relying on: conventions, "the senior always reviews this", "we all know you don't touch this module", "the Slack from March 14 where we decided X". Every one of those pieces has to move to a place where the agent can see it. If you don't do it, it's not that the agent "isn't ready" — it's that your discipline still lives in places where it can't be mechanically read, and therefore, for the agent, it doesn't exist.

The rest of the guide is, in large part, a catalog of where to relocate each kind of rigor: into explicit guides (ch. 3), into automatic sensors (ch. 3), into mechanically validated architecture (ch. 7), into the repo's versioned context (ch. 6), into continuous maintenance loops (ch. 9). None of these investments is optional. They are the new places where rigor has to live.
