---
layout: post
title: "ADR-001: Python for the reference, C++ where it counts"
date: 2026-08-04
topics: [agent-harness, adr]
status: designed
---

I promised myself this argument in public, so here it is — the first
architecture decision record for
[agent-harness](https://github.com/shashforge/agent-harness). A career
of C++ instincts versus the ecosystem reality of 2026.

**Status:** accepted. **Date:** 2026-08-04. **Decider:** me, arguing with
myself.

## Context

The harness needs a reference implementation. I've written C++ for a
living for most of my career; every LLM SDK, eval framework, and serving
integration worth touching is Python-first. The goal of this repo is a
*reference architecture* people can read in an afternoon, and evidence I
can ship monthly, not a performance record.

## Options

**A. Python end-to-end.** Fast to write, everyone can read it, plugs
straight into the model SDKs. Slower per operation, and it spends none of
my C++ credibility.

**B. C++ core with Python bindings from day one.** Home turf, fast,
impressive. Also: binding maintenance, slower iteration, and a smaller
audience for a repo whose purpose is to be read.

**C. Python reference now; port hot paths to C++ only when a benchmark
says so.** The hybrid.

| Dimension | A: Python | B: C++ core | C: Hybrid |
|---|---|---|---|
| Iteration speed | high | low | high now |
| Readability as reference | high | medium | high |
| Ecosystem fit (SDKs, evals) | native | bindings | native |
| Runtime performance | fine | best | fine, path to best |
| Uses my C++ depth | no | yes | yes, where measured |

## Decision

Option C. Here's the argument that settled it. **The harness is
control flow, not compute.** A run spends seconds waiting on model calls
and tool I/O; a state transition costs microseconds. Writing the executor
in C++ optimizes the one part of the system that is never the
bottleneck. That's premature optimization committed at the architecture
level, which is the expensive kind.

My C++ gets spent where the measurements will actually point:
the Edge AI lane. On-device inference runtimes (ONNX Runtime, LiteRT,
ExecuTorch) are C++ under the hood, and that benchmark work is where
native depth pays. Right language, right layer.

## Consequences

Easier: shipping the skeleton (it exists: 339 lines,
[8 passing tests](https://github.com/shashforge/agent-harness), written
in an afternoon), wiring in a real LLM planner, letting others read it.
Harder: nothing yet, honestly. What I'll revisit: if trace storage or
replay ever shows up hot in a profile, that module becomes the first C++
port, with the benchmark published here first.

## Action items

1. Executor skeleton in Python: done, tested
2. Checkpoint persistence + replay from trace: next
3. LLM-backed planner behind the same callable interface
4. Profile before porting anything, ever
