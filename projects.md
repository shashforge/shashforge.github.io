---
layout: default
title: Projects
permalink: /projects/
---

# Projects

Flagship builds, in the open. Each ships with architecture notes, code,
measurements, failure analysis, and a decision record — not just a
README. The status labels are honest: nothing here claims to be more
finished than it is.

## In progress

**Agent Harness Reference Architecture** <span class="status-badge">built</span>

The system argued for in
[the harness essay](/log/the-model-was-never-the-product/), turned into
working code at
[github.com/shashforge/agent-harness](https://github.com/shashforge/agent-harness):
the executor state machine, tool contracts, failure taxonomy, and
replayable transition trace, each pinned by a behavioral test. Next up:
checkpoint persistence, an LLM-backed planner, and an eval harness.
Decisions get recorded as ADRs, starting with
[ADR-001](/log/adr-001-python-vs-cpp/).

**Edge AI Runtime Benchmark** <span class="status-badge">designed</span>

One representative model, measured properly across ExecuTorch, LiteRT,
ONNX Runtime, and llama.cpp on Android hardware: latency, throughput,
memory, binary size, FP16/INT8/INT4 quantization, and accuracy impact
across CPU, GPU, and NPU. The
[measurement protocol](/log/edge-ai-benchmark-protocol/) is published
and frozen before any numbers exist; reproducible scripts ship with the
results.

## Queued

**Production AI Gateway**

A provider-neutral inference gateway: model routing, fallback, caching,
retries and timeouts, cost attribution, tracing, SLOs, and chaos testing
against provider failure.

---

More on [GitHub](https://github.com/shashforge). If one of these overlaps
with a problem you're hiring for, [email me](mailto:shashi@shashforge.dev).
