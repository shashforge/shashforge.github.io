---
layout: post
title: "The forge, audited"
date: 2026-08-30
topics: [meta]
status: meta
---

Four weeks ago [this site lit](/log/lighting-the-forge/) on a
premise: claims are cheap, so ship the evidence, dated and
versioned, where it can embarrass me. Four weeks is long enough for
the premise to face its own method. This entry audits the forge the
way the forge audits everything else: numbers first, gaps named,
excuses attributed to their owner.

## What exists that did not

Fifteen entries, counting today's two.
[agent-harness](https://github.com/shashforge/agent-harness): 952
lines of Python at version 0.3.4, 292 tests green in under a second,
CI across CPython 3.10 through 3.13 on every push, two architecture
decision records in `docs/adr/`, a golden trace pinning the
executor's canonical behavior, a 250-seed invariant sweep, and a
trace inspector that reads any run back as a timeline. As of today,
[edge-bench](https://github.com/shashforge/edge-bench): the
[benchmark protocol](/log/edge-ai-benchmark-protocol/) published on
August 4 turned into code that enforces it: 232 lines, 12 tests,
including one that fails if anyone ever adds a `mean()`. As of this
audit, 113 commits across the site and agent-harness, each one
small enough to read.

## What the method bought

The compounding was the point, and it showed up on schedule. The
golden trace, built on August 5 before anything needed it, is what
made [the August 9 refactor](/log/same-trace-different-guts/) boring
instead of risky. The failure taxonomy, argued on paper in
[the spine post](/log/the-harness-gets-a-spine/), became the literal
dispatch of the executor. The invariant sweep found no bugs and
forced [an ADR](/log/adr-002-seeds-over-hypothesis/) anyway. The
best entries came from the worst weeks: a three-day GitHub outage
sat on this site's deploys and produced
[the CI post's](/log/the-badge-is-a-claim/) whole argument about
handing guard duty to infrastructure that had just failed me.

## What did not happen, and whose fault it is

The roadmap's oldest open item is the live-model trace. The run kit
has needed one key and one human since August 6, which is
twenty-four days, and both belong to me. The Edge AI lane now has a
protocol and a harness and still no numbers, because the phone they
require is in my pocket and has stayed there. The pattern across
both gaps is not subtle: the tooling half of this operation has
out-shipped the half that holds the credentials. The audit records
this the way the bench records an empty cell: reason attached,
nothing silent.

One habit needs correcting too, and it is subtler than an errand.
The roadmap struck "LLM-backed planner" as done on August 5 on the
strength of tests that script the transport seam, while the live
proof sat unstruck one line below. Those are different claims.
Verified in the lab and proven in the field deserve different marks,
and a roadmap that blurs them is borrowing credibility from its own
future. When the live trace lands, the strike-through earns its
ink; until then, the honest reading of that line is "built, not yet
witnessed."

## The next four weeks

Live trace first; it unblocks the post this log has owed longest.
Then the CPU column of the benchmark matrix, on the two devices the
protocol promises to name. C++ enters where ADR-001 said it would: when a
measurement points at a hot path, and not before. The cadence stays
what it has been, entries when there is evidence, quiet when there
is not.

## Ledger

Twenty-seven days. Fifteen entries. Two code repos, 1,184 lines,
304 tests, every one green this morning. Two ADRs. One outage survived.
Two items blocked, both on the same person, who keeps writing that
down instead of doing something about it.
