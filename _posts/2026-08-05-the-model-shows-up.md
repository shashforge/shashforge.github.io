---
layout: post
title: "The model shows up"
date: 2026-08-05
topics: [agent-harness, llm]
status: built
---

Two days ago I claimed the model is a component, not the product.
Cheap claim to make in prose. Today the component actually arrived:
[`llm_planner.py`](https://github.com/shashforge/agent-harness), 147
lines that put a model behind the planner callable.

Here's the part I care about: the executor did not change to
accommodate it. Zero lines. It still calls `planner(checkpoints)` and
gets back a step or `None`, exactly as it did when the planner was a
three-line function in a test. That non-diff is the design claim,
demonstrated. If wiring in an LLM had required touching the state
machine, everything this site has argued so far would have been wrong.

## The contract is rigid on purpose

The model answers with one JSON object per turn: a tool step, or
`{"done": true}`. That's the whole protocol. Prose, half-JSON,
reasoning out loud, a tool that isn't in the catalog — every one of
those raises `PlannerProtocolError` and stops the run.

I allow exactly one tolerance: a markdown code fence around the JSON,
because models love fences and stripping one is unambiguous. Nothing
else. The failure taxonomy applies to the model the same way it
applies to tools. When a tool breaks its contract we don't guess what
it meant, and the model gets no more charity than the tools do.

A detail that fell out of the state machine for free: the planner is
prompted with *verified* checkpoints only. Failed attempts, retries,
unverified junk — none of it reaches the model's context, because
none of it ever became a checkpoint. The persistence rule from this
morning's post turns out to be a prompt-hygiene rule too.

## Tested at the seam, and honest about it

The API call is one injectable function: request payload in, response
body out. The tests script that seam. They prove everything on my side
of the wire: the model is shown the goal, the tool catalog, and the
verified checkpoints; the parser refuses prose; an out-of-catalog tool
never reaches the executor; the full loop runs to `DONE` on scripted
decisions.

![Architecture diagram of the transport seam. Executor calls the LLMPlanner callable, which speaks to a single injectable transport function, which speaks to the Messages API. A dashed line marks the seam between transport and API. Everything left of the seam is pinned by seven tests using a scripted transport; the reply contract allows one JSON tool step or done-true, anything else raises PlannerProtocolError. Right of the seam needs an API key; the live run is pending and there is no fake fallback.](/assets/img/transport-seam.svg)

What these tests prove about the model itself: nothing. The code
refuses to blur that line: there is no offline fallback faking model
responses, the live transport won't run without an API key, and a
test pins the refusal. Testing against your own mock and calling it a
result is the vendor-benchmark move, and I published a whole
[protocol](/log/edge-ai-benchmark-protocol/) about not doing that.

## Next

The first live run, with the resulting JSONL trace published unedited:
which steps the model planned, where it got retried, what the
transition log looks like when the planner has opinions. The recorder
went in first so the run itself can be published, not my summary of
it.
