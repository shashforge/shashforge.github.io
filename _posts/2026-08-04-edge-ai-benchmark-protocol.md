---
layout: post
title: "The Edge AI benchmark: protocol first, numbers later"
date: 2026-08-04
topics: [edge-ai, benchmark, android]
status: designed
---

Opening the Edge AI lane the way benchmarks should open: by publishing
the methodology *before* any numbers exist. A benchmark designed after
seeing results is a press release. This entry is the protocol I'm
committing to; the numbers will have to live with it.

## Candidates

Four runtimes, current as of this week:

- [ExecuTorch](https://github.com/pytorch/executorch) 1.0.0: PyTorch's
  on-device stack, ahead-of-time compiled to hardware backends
- [LiteRT](https://ai.google.dev/edge/litert) (LiteRT-LM 0.9.0): Google's runtime, JIT graph rewriting for GPU at runtime
- [ONNX Runtime](https://onnxruntime.ai/) 1.23.2 with onnxruntime-genai
  0.10.0: the portability play
- [llama.cpp](https://github.com/ggml-org/llama.cpp): the community
  baseline everything gets measured against, like it or not

The AOT-versus-JIT split between ExecuTorch and LiteRT is the
architecturally interesting comparison: compile-time knowledge of the
hardware versus runtime adaptation. I have a prediction about which wins
on cold start and which on portability. I'm writing it down privately
and will publish it with the results, so past-me can be wrong in public.

## Model

Primary: **Llama-3.2-1B-Instruct**. It exists in every ecosystem's
native format (GGUF, ONNX, ExecuTorch's SpinQuant INT4 export).
Secondary: **Gemma3-1B** where Google's tooling treats it as the paved
road. Where a runtime can't run the primary model, that gets reported as
a result, not silently swapped.

## What gets measured

| Metric | How |
|---|---|
| Time to first token | p50 / p95 over 10 runs |
| Decode speed | steady-state tokens/sec, 256-token generations |
| Peak memory | RSS high-water mark during generation |
| Size cost | runtime libraries added to APK + model artifact |
| Cold start | process launch to first token, cache cleared |
| Quality delta | fixed 200-prompt set, output diff vs FP16 reference |
| Thermal behavior | 5-minute sustained generation, throttle onset time |

## The matrix

Quantization: FP16, INT8, INT4, as each runtime supports them, on CPU
(XNNPACK), GPU (Vulkan/OpenCL), and NPU where the device exposes one.
Every cell in the matrix gets either numbers or a documented reason
there are none: conversion failed, delegate unsupported, crash.
Negative results are results. Silent gaps are how vendor benchmarks lie.

## The discipline

Rules for every measured run, because mobile numbers without protocol
are noise: airplane mode, screen on at fixed brightness, battery above
80%, device cooled to ambient between configurations, 3 warmup runs
discarded, 10 measured, medians with IQR reported — never means, which
one thermal throttle event can ruin. Hardware: the Android devices I
actually own, named in the results post, one mid-range and one flagship.
No emulators. Emulator inference numbers are fiction.

## Why this order

Measurement methodology is where credibility lives or dies. I learned
that in automotive, where a wrong number outlives everyone's memory of
how it was measured. Anyone can produce a chart. Committing
to the protocol first costs me the ability to quietly drop a runtime
that embarrasses my prediction. That's the point.

First results target: the CPU column, both models, all four runtimes.
