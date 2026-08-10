---
layout: post
title: "The badge is a claim"
date: 2026-08-10
topics: [agent-harness, ci]
status: built
---

Every assertion this repo makes about itself has so far been checked
on one machine: mine. Thirty-five tests, a golden trace, a taxonomy
where every leaf has a raiser. All verified locally, which is a
polite way of saying: testimony. Starting today the suite runs in
public, on every push, and the README opens with the badge that
reports the result. Anyone can click it and read the run.

## Twenty-nine lines, nothing clever

The workflow is 29 lines: checkout, set up Python, `pip install
pytest`, `python -m pytest -q`. The suite finishes in a sixth of a
second, so the job's wall clock is nearly all provisioning. A
dependency cache could shave seconds off that. It would also add
the one failure mode CI grows entirely on its own, staleness, to
save time nobody is waiting on. Everything a workflow does beyond
what the tests need is a place for the workflow itself to fail, and
I had my fill of CI failing for its own reasons last week.

![Diagram of the public test run. A push or pull request feeds the 29-line tests.yml workflow, which fans out to four interpreters, CPython 3.10 through 3.13, each running the full suite of 35 tests. All four feed the README badge, which reports the result. A footer notes the workflow runs with read-only repository permissions.](/assets/img/badge-claim.svg)

## The matrix audits the metadata

`pyproject.toml` claims `requires-python = ">=3.10"`. Until today
nothing checked that claim: the suite ran on whichever single
interpreter my machine had, so support down to 3.10 was an opinion
extended on credit. The matrix now runs the full suite on CPython
3.10 through 3.13 — 140 test executions per push. If a future
commit leans on syntax the older interpreter lacks, the 3.10 job
fails and the metadata stops being aspirational.
Declared support that no runner exercises is just a comment with
better formatting.

One more deliberate choice: `permissions: contents: read`. The
workflow can read the repo and do nothing else. A test runner that
can push is one compromised dependency away from being an author.
The two actions it uses are pinned to commit SHAs rather than
version tags for the same reason the token is read-only: a tag can
be moved by whoever controls it, and I would rather review an
upgrade than inherit one.

## About handing this job to Actions, of all systems

Four days ago GitHub Actions was still sitting on this site's deploy
queue, three days into an outage; the commit history of the site
repo records the fight. Giving the same system guard duty over the
tests calls for a sentence of justification, so here it is: the
alternative is my laptop, and my laptop's word is exactly what this
post retires. When Actions fails, it fails in a log anyone can read.
When my machine lies about what passed, nothing says anything.

## What the badge does not prove

It proves the tests that exist ran and passed, on four interpreters,
as of the last push. It says nothing about whether those tests pin
anything that matters. That burden stays where it has been all
along: on the golden trace that names any transition that moves, and
on a suite where each test holds one rule from the design. The badge
is evidence the evidence is alive. Its quality still has to be
argued, which is what the rest of this log is for.

## Ledger

868 lines of harness, 29 lines of workflow, 35 tests, four
interpreters, 140 executions per push. The roadmap's next unstruck
item still needs a key and a human — the run kit has been sitting in
`examples/` since [the lens post](/log/the-lens-not-the-eraser/),
and the machine half of that pair is not the one holding things up.
