# 5. Isolated and reproducible environments

If I had to pick a single infrastructure investment that makes the biggest difference when introducing agents to a team, it would be this: **isolated environments, cheap to create and cheap to throw away**. It's not the most glamorous, it doesn't show up in demos, and it's almost always underestimated. But it's the one that multiplies everything else.

## The problem it solves

Without isolation, an agent working in your repo is both a risk and a bottleneck. A risk, because any command it runs (installing dependencies, modifying the database, touching environment variables) affects your system and your teammates'. A bottleneck, because you can only have one agent running at a time without them stepping on each other.

With isolation, both problems disappear at once. And they unlock a third capability which is the real game-changer: **parallelism**. You can launch ten agents at once on ten tasks, each in its own environment, without coordination.

## The two reference points

**Stripe — devboxes.** Stripe uses standardized EC2 instances that are "hot and ready" in 10 seconds. Each minion (their internal agent) spins up on one, does its work, opens a PR, and the devbox gets tossed. Creation latency matters: if it took 5 minutes, the agent would wait on every task and the system would be much less efficient. 10 seconds is fast enough to treat the environment as truly disposable.

**OpenAI — bootable git worktrees.** OpenAI took another route: each git worktree can boot the whole app, with its own ephemeral observability stack (Vector, Victoria metrics/logs/traces). Each change is validated in its own worktree, with its own logs. When the worktree is deleted, everything disappears with it.

Important clarification: the worktree model doesn't imply "local machine". OpenAI runs them on powerful remote dev hosts, not on engineers' laptops. The real distinction with Stripe's model isn't *local vs cloud*, it's **many small VMs vs one big machine with many worktrees in parallel**. Each model scales differently: VMs horizontally, worktrees vertically up to what the host can take.

The two solutions differ in mechanics but are identical in philosophy: **the environment is disposable, instant, and complete**.

### When to pick which

The right question isn't "what do the big players do?", but **what's my bottleneck?**:

- If what you care about most is **production parity and massive horizontal scale** (hundreds of agents in parallel, huge monorepo, mature infra), the Stripe model fits better.
- If what you care about most is **iteration loop speed and observability close to the agent** (dozens of agents in parallel, a repo where the app boots fast), the OpenAI model is usually cheaper and faster to set up.

Most teams are closer to the second case than they think, and start copying the first one because "that's the serious way". They end up with expensive infra the agent doesn't make use of.

### The gotcha of ports and shared dependencies

If you go the worktree route, the first real pain isn't filesystem isolation (git gives you that for free), it's **shared resources at the process level**: two worktrees trying to open the same port, touch the same local database, write to the same `node_modules` cache, or register on the same socket. If you don't design for it from the start, you launch the second worktree and everything blows up.

What works: dynamically-assigned ports when the app starts, a database per worktree (a schema or an ephemeral DB per environment), any cache path parameterized by worktree. It's a day's work and unblocks the rest of the model. If you leave it for "later", "later" is when five parallel agents prove to you that you didn't have real isolation.

### You don't have to choose: hybrid models

The two approaches aren't mutually exclusive. A reasonable pattern is to use **worktrees for the agent's inner loop** — fast, cheap, with observability stuck to it — and **devboxes (or equivalent) only for the PR/CI loop**, where what matters is production parity before the merge. You keep the iteration speed of the first and the guarantee of the second, without paying the full cost of either.

## The properties that matter

An isolated environment for agents must be:

**Actually isolated.** If two agents running in parallel can collide (on the same database, on the same port, on the same cache file), it's not isolation, it's an illusion. The test is brutal: launch 5 agents at once and see if any fail because of another. If that happens, you don't have isolation.

**Cheap to create.** "Cheap" means seconds, not minutes. An environment that takes 3 minutes to be ready pushes you to reuse it, and reusing it leads to cross-contamination, which breaks isolation. Boot speed isn't a convenience: it's what sustains the discipline of isolation.

**Cheap to throw away.** No manual steps, no "remember to clean up X", no orphan resources in the cloud. Normal operation is: create, use, destroy. If destroying is expensive, environments pile up and cost runs away.

**Complete.** The agent has to be able to do *everything* it needs inside the isolated environment: run tests, boot the app, query logs, navigate the UI, talk to mocked or real dependencies. An environment where the agent "almost can" do everything is worse than none: the things it can't do become invisible blockers.

**Reproducible.** Two creations of the same environment should yield the same result. If environments diverge (versions, seeds, data), the agent starts seeing intermittent failures and blames its own code. You lose trust in the sensor.

## What isolation unlocks

Once you have this, several things become possible that previously seemed too expensive:

- **Ephemeral observability.** Like at OpenAI, you can run a full logs/metrics/traces stack *per environment*. The agent asks questions like "does this request take less than 200ms?" and gets a verifiable answer, without contaminating production or the team's shared environments.
- **Reproducible UI.** The agent can boot the whole app, navigate it with DevTools Protocol, take screenshots, compare before/after. The UI stops being terrain where only humans can validate.
- **Destructive testing.** Operations nobody normally wants to run (drop tables, reset migrations, simulate network failures) become safe inside a disposable environment.
- **Long tasks without guilt.** If the agent takes 6 hours on a task (OpenAI mentions examples of that), nobody worries, because it's running in its own environment and doesn't block anyone.
- **Trivial fan-out.** "Try these five different implementations in parallel and tell me which is best" stops being a theoretical exercise. Five environments, five loops, one comparison at the end.

## How much to invest

The reasonable question is: how far should you take this? The practical answer is **to the point where launching a new environment stops being a conscious decision**. If you think twice before launching an agent because "ugh, another environment", you're not there yet. Isolation has worked when the mental cost of launching an agent is indistinguishable from the mental cost of opening a browser tab.

## What isolation isn't

It's worth disambiguating. These things often get confused with isolation and *aren't*:

- **"The agent runs on my machine":** not isolation; the opposite. You're sharing an environment with everything else you have open.
- **A git branch:** a branch is isolation of changes, not of runtime. Two agents on two branches can still step on each other if they share a local DB, ports, or cache.
- **A container reused across tasks:** better than nothing, but not disposable. Accumulated cross-contamination becomes a source of intermittent failures.

The standard to hit is: every task, its own environment; when it's done, it's discarded; it boots in seconds; reproducible byte-for-byte. When you get there, almost every other chapter in this guide becomes easier to apply.
