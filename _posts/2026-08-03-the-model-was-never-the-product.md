---
layout: post
title: "The model was never the product"
day: 2
date: 2026-08-03
topics: [agents, harness-engineering, platform]
status: analysis
---

In 2023 I watched AutoGPT burn through $40 of API credits trying to order a
pizza. Everyone laughed. Everyone forked it anyway, and "agent" went on to
become the most overloaded word in the industry. Three years later the
correction has arrived, and to me it's the most important shift in AI
engineering right now: we stopped building agents and started building
harnesses.

You could see it all over [AI Engineer World's Fair this year](https://www.latent.space/p/aiewf26trends).
The talks weren't about autonomy. They were about the machinery around the
model: context management, permission boundaries, evaluation loops,
continuous improvement. Lilian Weng's updated framework even gives the
thing a name, harness engineering. The model is an engine component. The
harness is the vehicle. Nobody ships an engine to a customer.

## Why autonomy lost

The math was never on autonomy's side. I think most of us knew it before we
admitted it. If each step of an agent chain succeeds 95% of the time
(generous), a 20-step unsupervised run completes correctly about 36% of the
time. That's not a product. That's a slot machine with a system prompt.

![Line chart: probability that a chain of steps all succeed, for per-step success rates of 0.99, 0.95 and 0.90. At 20 steps, a 95% per-step rate gives only 36%.](/assets/img/chain-reliability.svg)
*The exponent is the enemy. Even at 99% per step you lose a fifth of your
runs by step 25.*

The teams that actually got things into production didn't wait for a
smarter model to fix the exponent. They restructured the problem so the
exponent stops mattering: shorter loops, checkpointed state, verification
after every step that can be verified, and a human placed exactly where
judgment is needed. The outer-loop/inner-loop pattern that
[got so much airtime this year](https://www.latent.space/p/aiewf26trends)
is this idea formalized. Autonomous inner loops, humans supervising from an
outer loop of evals and feedback signals. It's control theory arriving in
AI systems, about a decade after SREs started doing the same thing with
error budgets. We're not inventing anything here. We're finally applying
our own discipline to our newest dependency.

![Diagram: a harness wrapping the model, containing a context manager, tools and permissions, verifier and memory, and evals — an autonomous inner loop inside, a human outer loop for oversight.](/assets/img/harness-loops.svg)
*My mental model of the 2026 stack. The box worth studying is not the one
labelled "model".*

## Context is a systems resource. Treat it like one.

The other half of this shift is that
[context engineering](https://sourcegraph.com/blog/context-engineering)
grew up. What started as prompt-hacking folklore is now a real discipline,
and it's the part I enjoy most, because it behaves exactly like the
resource-management problems backend people have been dealing with for
decades.

The context window is memory. It fragments. It degrades.
[Chroma's context-rot study](https://www.trychroma.com/research/context-rot)
shows accuracy falling as token counts grow, and the
[lost-in-the-middle effect](https://arxiv.org/abs/2307.03172) means
position matters as much as presence. So the winning patterns look
suspiciously like an operating system: admission control before anything
enters the window, compaction when pressure builds (garbage collection,
basically), just-in-time retrieval instead of preloading, re-rankers
enforcing hard top-k caps.

And the numbers say retrieval quality dominates everything else. A
[benchmark Sourcegraph published in March](https://sourcegraph.com/blog/context-engineering)
compared naive grep against indexed, code-aware retrieval on enterprise
tasks. Vendor numbers, measuring their own product, so salt to taste. The
direction still holds. Precision@5 went from 0.14 to 0.478. A cross-file refactor dropped
from 96 tool calls over 84 minutes to 5 calls in under five minutes.
That's a 19x win with no model change. When did a model upgrade last give you
19x? The leverage moved into the harness. Most budgets haven't noticed
yet.

Tools tell the same story. The most common failure mode in agent systems
isn't hallucination, it's bloated tool sets. Every tool definition you
register is a context tax and a decision surface. Ten sharp tools beat
forty vague ones. This is API design. We've known how to do it for twenty
years; we just keep pretending it's new.

## The part I'm skeptical about

"Skills" were everywhere this year: portable, Markdown-encoded workflows
that agents load and execute, now on every major platform. I like the
idea. Encoding team knowledge as versioned, reviewable text files beats
burying it in prompts. But I've seen this movie in config management.
Today's elegant skill library is next year's `utils/` folder. One speaker
at World's Fair [warned about "skills hell"](https://www.latent.space/p/aiewf26trends)
and he's right. Without ownership, review discipline and a deprecation
policy, we're inventing prompt sprawl with better marketing.

Geoffrey Huntley's caution from the same event stuck with me too: loops
are frontier thinking we still haven't figured out. Agreed. Anyone selling
you a solved agentic platform in 2026 is selling you their roadmap.

## What this means if you build platforms

The uncomfortable conclusion is that the differentiating work in AI right
now is boring, and that's exactly why it's valuable. State management.
Retrieval indexes. Eval suites that run like regression tests.
Observability that can answer "why did the agent do that" three weeks
after it did it. Permission models. The people best positioned for this
aren't prompt artists. They're distributed-systems engineers who took the
time to learn how models fail.

That's the bet this whole site is built on, frankly.

What I'm watching next: whether harnesses standardize the way web
frameworks did, or stay bespoke per company. MCP pushed the tool-interface
layer toward a standard. Nothing equivalent exists yet for loops, evals or
memory. If that standard shows up in the next 18 months, it will matter
more than whatever model release gets announced next to it.

The model was never the product. The system around it always was. It just
took the industry three years and a lot of pizza money to admit it.

*Building in this layer too? My inbox is open — details on the
[about page](/about/).*
