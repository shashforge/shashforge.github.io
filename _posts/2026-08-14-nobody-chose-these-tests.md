---
layout: post
title: "Nobody chose these tests"
date: 2026-08-14
topics: [agent-harness, testing]
status: built
---

All 35 tests in the harness were, until now, cases I picked:
behaviors I could already name, pinned one at a time. That is their
job, and also their ceiling. A suite I chose inherits my
imagination, blind spots included. Today the repo gained a file
where the cases are picked by a seeded `random.Random`, and my only
contribution is the list of things that must be true no matter what
comes out of it.

## The world the seed builds

Each of 250 seeds constructs a small hostile world. One to four
tools, some of which demand a scope the run was never granted. Tool
bodies that flake with a `TransientToolError`, refuse with a
permanent one, or reject their own work outright, at fixed odds per
call. A verifier that is a weighted coin. A planner drawing random
steps toward a random goal. And a human with finite patience: zero
to three resumes, then abort.

The executor runs each world with a store attached, and then the
invariants get their say. The run ends in `DONE` or `FAILED`, never
anywhere else. The transition sequence is gapless and strictly
increasing. A checkpoint exists exactly when a verification was
recorded, one for one. No tool is ever called past its declared
budget. And `replay()` reads the file the run left behind and
reaches the same conclusions without executing anything. Until now
replay had only ever been shown tidy runs and hand-forged
corruptions; these 250 traces arrive full of denials, retries,
escalations, and aborts.

![Diagram of the invariant sweep. A seed builds a random world of misbehaving tools, a random-goal planner, a coin-flip verifier, and a human with finite patience. The executor runs it and the trace must satisfy four invariants: terminal state always, gapless sequence, checkpoint only with verification, budgets respected. replay() then re-derives the run from the stored trace, agreeing 250 times out of 250.](/assets/img/invariant-sweep.svg)

## Why a seed and not a framework

Hypothesis would do this with more sophistication, and its failure
shrinking is real value I am walking away from. What a seed gives
back is a perfect reproduction for free: if seed 217 fails, you run
seed 217, forever, on any machine. It also costs nothing: no new
dependency, no change to the CI file whose 29 lines I bragged about
four days ago. For a reference implementation whose whole argument
is a small dependency surface, that trade goes one way.

Randomness got a veto on everything except termination. A generated
human that resumes forever would earn a run that runs forever, which
is the design working as stated — the human is a state, and states
can loop. So every generated human has finite patience, and a
tripwire fails any seed whose planner is consulted ten thousand
times. The sweep is allowed to be hostile. It is not allowed to be
eternal.

## What it found

No invariant violations. 250 seeds, zero counterexamples, and I
will not pretend that proves the executor correct; it proves the
executor survives 250 worlds drawn from the distributions I wrote,
which is a smaller and more honest claim.

What writing it actually produced was precision. Stating invariants
formally forced distinctions the prose versions blurred. One became
a new targeted test on the spot: the permission gate sits before the
call meter, so a denied call is never billed against the tool's
budget. True in the code all along, pinned by nothing until today.
A second test sharpens an old pin: the suite already checked that a
tireless planner triggers the step-budget escalation; it now also
checks what such a run banks on the way out, exactly `max_steps`
verified checkpoints and not one more.

## Ledger

868 lines of harness, unchanged. One new 169-line test file, four
test functions, one of which carries all 250 seeds. The suite pytest
collects went from 35 to 288 and still finishes in half a second.
Version 0.3.2, and the README's rules list gained its eleventh
entry: the invariants hold under behavior nobody chose.
