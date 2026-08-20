---
layout: post
title: "ADR-002: Seeds over Hypothesis"
date: 2026-08-17
topics: [agent-harness, adr]
status: built
---

Friday's [invariant sweep](/log/nobody-chose-these-tests/) spent one
paragraph on why it uses a seeded `random.Random` and not a
property-testing framework. A paragraph in a post is where decisions
go to be forgotten. This is the record, and as of today it lives in
the repo at `docs/adr/`, next to a backfilled ADR-001, so the code
carries its own reasons.

**Status:** accepted. **Date:** 2026-08-17. **Decider:** me, having
already shipped the code on Friday and owing it a defense.

## Context

The executor promises invariants: a terminal state always; a gapless,
strictly increasing trace; a checkpoint one for one with a recorded
verification; no tool called past its budget; every trace replays.
Those needed testing against behavior I did not hand-pick, which
means something has to generate planners, tools, and verifiers with
conduct I never chose. Two candidate generators. The repo has zero
runtime dependencies and one test dependency, and its whole purpose
is to be read as a reference in an afternoon.

## Options

**A. Hypothesis.** Strategies for planners, tools, and verifiers,
`@given` over the executor, shrinking on failure. Battle-tested, and
the shrinker is real engineering I would not want to write myself.

**B. Seeded stdlib random.** `random.Random(seed)` builds each world,
`@pytest.mark.parametrize("seed", range(N))` runs them all, and a
failing seed number is the whole reproduction.

**C. Both.** Hypothesis for depth, seeds for the fast CI lane. Two
mechanisms for one job.

| Dimension | A: Hypothesis | B: Seeds | C: Both |
|---|---|---|---|
| New dependency | yes | no | yes |
| Failure minimization | shrinker | none | shrinker |
| Reproduction | example database + seed | one integer | mixed |
| CI change | pin, cache the database | none | yes |
| Reader follows the file cold | needs framework fluency | yes | no |
| 250 worlds, wall clock | seconds, tunable | 0.7 s | more |

## Decision

Option B. The argument that settled it: **the failure I most need to
defend against is a reader not trusting the test, and every line of
framework between the reader and the executor costs trust.** The
sweep is 169 lines a stranger can read top to bottom and see exactly
what gets generated and exactly what gets asserted. Nothing is
imported that a Python programmer has to go learn first.

Shrinking is the real thing given up, and I want to be precise about
why that is bearable here rather than pretend it costs nothing. A
failing seed in this codebase yields a trace of a few dozen
transitions at most, each carrying its reason string. That is already
a counterexample small enough to read unminimized. Shrinking earns
its keep when the failing input is large and mostly irrelevant; the
executor's inputs are neither.

![Diagram of ADR-002. Three options are weighed: Hypothesis with its shrinker and example database, a seeded stdlib random with one-integer reproduction, and both together. The seeded option is chosen, with the deciding argument that framework layers between a reader and the executor cost trust. A revisit clause lists the two conditions that would reopen it: an unreadable failing trace, or generated worlds needing structure the hand-rolled generators cannot express cleanly.](/assets/img/adr-002-seeds.svg)

## Consequences

Easier: widening the sweep is one number; the CI file stays at 29
lines; the test file doubles as the plainest statement of the
invariants in the repo. Harder: no automatic minimization; the
distributions are hand-written and exactly as hostile as I made them,
no more; 250 draws cover what 250 draws cover, and a strategy-aware
engine would explore differently.

Revisit when either of two things happens. A seed fails and its trace
is too large to read, which is the day shrinking stops being a luxury.
Or the generated worlds need structure the hand-rolled generators
make ugly: nested plans, stateful tools, dependent draws. Either one
justifies Hypothesis as an additional dev dependency, kept out of the
default CI job so the badge keeps meaning what it means today.

## Why the record moved into the repo

ADR-001 lived only on this site until today. A decision record that
sits outside the codebase it governs is a blog post with a template;
the next person to open the repo has no reason to know it exists.
Both records now sit in `docs/adr/`, numbered, with an index that
states the one rule for the folder: never rewrite. A superseded record
keeps its place, changes its status, and points forward. Hindsight
gets a new number.

## Ledger

Two ADRs in the repo, 147 lines between them and their index. Version
0.3.3. The roadmap's ADR item is struck; the live-run item is not,
and I notice the README has said "it needs a key and a human" since
August 6, which is eleven days. The key is mine. So is the human.
