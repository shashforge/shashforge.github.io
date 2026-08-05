---
layout: post
title: "Replay is the feature"
date: 2026-08-05
topics: [agent-harness, persistence]
status: built
---

The skeleton I published yesterday had a memory problem. The trace was
data, the checkpoints were data, and every byte of it lived in RAM.
Kill the process and the run's whole story died with it. For a harness
whose sales pitch is auditability, that's embarrassing. In embedded
work you learn early that state living only in RAM is state you've
already agreed to lose. Today I stopped agreeing.

## Persistence, the boring way

Transitions and checkpoints now stream to an append-only JSONL file as
they happen, not when the run ends. One file, two record kinds, no
database. [`persistence.py`](https://github.com/shashforge/agent-harness)
is 126 lines and most of them are documentation.

The constraint this buys is real: checkpoint results must be
JSON-serializable, and the store throws if they aren't. Deliberate. A
checkpoint you can't serialize is a checkpoint you can't resume from,
and I want that failure at the moment of writing, while it still costs
nothing. There's a test that pins exactly this.

## The crash test

My favorite test of the day gives the executor a tool that works twice
and then raises `RuntimeError("power cut")`. Not a taxonomy error —
the harness only catches failures it understands, so this one kills
the run the way a real crash would. The file left behind shows two
verified checkpoints and nothing more.

Then a second executor opens the same file with `resume=True`. The
planner sees the two verified checkpoints, plans step two, and
finishes the four-step goal. The sequence numbers continue from where
the dead process stopped: one continuous trace, two processes, and the
seam is visible in the file if you know where to look.

![Timeline of the crash-resume test: process A records sequence numbers 1 through 7 with verified checkpoints for steps 0 and 1, then dies on a RuntimeError with step 2 planned but never verified. Everything streams into one append-only run.jsonl file. Process B opens the same file with resume=True, continues at sequence 8, checkpoints steps 2 and 3, and reaches DONE at sequence 14.](/assets/img/crash-resume-trace.svg)

## Replay refuses bad stories

`replay()` reads the file and reconstructs the run without executing a
single tool: final state, verified progress, transition count. It also
enforces the invariants the executor promises. Sequence numbers must
strictly increase, because append-only means append-only. Every
checkpoint must have a `VERIFY -> CHECKPOINT` transition vouching for
it, because nothing unverified counts as progress.

Two tests tamper with the file to prove the point. One forges a
checkpoint no verifier ever saw; one re-appends an old record so the
sequence runs backwards. Replay rejects both with `TraceCorruption`.
A trace that can't support its own story is a corrupt flight
recorder, and replay treats it like one.

## Golden traces, or: evals for the harness itself

Once wall-clock time is stripped, a deterministic run is a snapshot of
the executor's behavior. So the repo now ships one:
`tests/golden/happy_path.json`, ten transitions, the canonical
three-step run. A test replays the executor against it on every run.

Change the retry policy, reorder verification, touch the checkpoint
rule, and that test fails naming the exact transition that moved and
which fields changed. Recording a new golden is a reviewable diff in
version control, not silent drift. Regression tests for control flow,
not just for outputs. This is the eval harness the README promised,
and it cost 82 lines because replay had already paid for the hard
part.

## Ledger

Yesterday: 339 lines, 8 tests. Tonight: 713 lines, 25 tests, one
golden trace committed. The planner also stopped being a stub today,
but that's its own post.
