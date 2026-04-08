# 4. The loop: iteration as primitive

There's an observation that shows up in every team running agents seriously, and Geoffrey Huntley articulates it better than anyone: **the agent isn't a function, it's a loop**. Treating it as a function — input, output, done — is the source of half the problems you see in teams that are just starting out.

## The mental model shift

When you call a "pure" LLM — chat, completing a message — it's reasonable to think of it as a function. You give it a prompt, it returns text. But an agent doesn't work that way. An agent:

1. Receives a goal.
2. Looks at the environment (reads files, runs commands, queries tools).
3. Decides an action.
4. Executes it.
5. Observes the result.
6. Decides whether to continue, back off, or finish.
7. Goes back to step 2.

That's a loop. And the important thing is that **the quality of the agent doesn't just depend on what decision it makes at each step, but on how many iterations it can sustain before going off the rails**. An average model in a well-designed loop beats an excellent model in a badly-designed one. This is counterintuitive and it's where most money is being lost in the industry right now.

## The "Ralph loop" as primitive

Huntley describes what he calls the *Ralph loop* (named after Ralph Wiggum from the Simpsons): a monolithic orchestrator that runs one iteration of the loop and starts over. No complicated planning logic, no exquisite decision trees. Just: run a turn, observe, run another.

The intuition behind it matters. When an agent fails, there are two possible responses:

- **Brittle response:** add logic that detects that specific failure and avoids it. Every failure spawns orchestration code.
- **Robust response:** make sure the loop can *notice* it failed and retry differently. The failure spawns a better sensor or a better guide, not more logic.

Huntley calls this "watch the loop": watch where the loop fails repeatedly and, instead of patching it, put on your engineer hat and fix the systemic condition that causes it. It's the same philosophy as SRE: don't celebrate the hero who puts out the fire every night, fix the cause of the fire.

## Three properties of a good loop

If you're going to invest in the loop (and you should), look for three properties:

**1. Observable.** Each iteration should produce readable signals. It's not enough for the agent to "finish"; it has to leave a trail of what it tried, what it saw, what it decided. The loop's logs are your only debugging mechanism when something goes wrong at 3am.

**2. Interruptible and resumable.** A loop that needs to run uninterrupted for six hours and gets lost if you stop it is brittle. A loop that can stop at any point, persist its state, and resume, is resilient. OpenAI mentions that individual Codex runs can work for over six hours on a single task — that's only possible if the loop is persistent.

**3. Self-terminating by progress, not by attempts.** "Retry up to 5 times" is a bad stop condition. "Retry while you're making measurable progress" is a good one. The difference shows up when the agent has spent 4 rounds working well on a hard task and the loop kills it on an absurd timeout.

## Nested loops

In serious teams, there's rarely a single loop. What you see is a hierarchy:

- **Inner loop (seconds):** edit → typecheck → fast test → next edit. It's the hottest loop, the one the agent goes around hundreds of times per task.
- **Middle loop (minutes):** PR → CI → automatic review → response to comments → re-merge. It's the loop Stripe describes in its "blueprints", combining deterministic nodes (linters, tests) with agentic nodes (implement, refactor).
- **Outer loop (hours or days):** doc-gardening, refactor agents, drift elimination, quality score updates. It's the loop that keeps the codebase livable months down the road.

The three loops influence each other. A recurring failure in the inner loop becomes a new guide in the outer loop. A decision in the outer loop (raise the coverage threshold) changes the behavior of the inner loop. Designing them as a system, not as three disconnected scripts, is what distinguishes a real harness from a collection of automations.

## The loop as a philosophy, not just code

Huntley insists that the loop is also a mindset. When you see the agent fail, don't ask "what do I tell it so this doesn't happen?". Ask "what loop do I need to build so this gets detected and corrected without me?". The first reflex is exhausting and doesn't scale. The second is engineering.

There's an associated danger: falling in love with the loop to the point of never finishing building it. The loop exists to solve real tasks; if you spend three weeks without shipping anything because you're "perfecting the loop", the loop has become another orphan project. The golden rule: every investment in the loop must be justified by a real, recurring, recent failure. Not a hypothetical one.

## A note on parallelism

An agent's natural loop is sequential: one iteration after another. But at the team level, the interesting thing is **launching many loops in parallel**. Stripe describes this explicitly: engineers launch multiple minions at a time, each working on a different task in its own devbox. Parallelization doesn't happen inside the loop, it happens *between* loops, and it depends entirely on having isolated environments (next chapter) where multiple loops can run without stepping on each other.

This is the difference between having "an agent" and having "agents". The first is a tool. The second is team capacity. And the transition from one to the other almost always goes through accepting that the primitive isn't the model, or the prompt, or the tool: it's the loop.
