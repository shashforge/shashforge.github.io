---
layout: post
title: "Lighting the forge"
date: 2026-08-03
topics: [meta, commitment]
status: meta
---

Today I bought **shashforge.dev** and lit the forge.

This site is a standing commitment: **a public log of engineering
artefacts**, documenting the real work of building AI systems — not hot takes, not
link roundups. What I designed, what I built, what broke, what I measured,
and what I'd do differently.

## Why a public log

Three reasons:

1. **Writing is compression.** If I can't explain a design decision in a
   few paragraphs, I don't understand it yet.
2. **Consistency is the signal.** Anyone can write one great post. A
   steady public stream of artefacts is much harder to fake — and much
   more honest about how engineering actually happens: incrementally.
3. **The field moves fast, so the log has to.** Agentic systems, LLM
   serving, evals, retrieval — the ground shifts every week. Publishing
   as I build forces me to stay in contact with the ground.

## What this stack is

Deliberately boring, which is the point:

- **Jekyll** static site — a new post is just a Markdown file
- **GitHub Pages** for hosting — every post is a commit, deploys on push
- **shashforge.dev** via Namecheap DNS — `.dev` enforces HTTPS at the TLD level

Total moving parts: approximately zero. All the energy goes into the
content, none into the plumbing. The interesting infrastructure writing
belongs in the entries, not under them.

## What's coming

The log will orbit the problems I care about professionally: **agentic
system architecture, LLM platform engineering, retrieval done properly,
and the distributed-systems reality underneath AI products.** Expect
diagrams, benchmarks, and honest failure reports.

The first entry is the easiest one. More soon.
