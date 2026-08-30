---
layout: post
title: "800 kilobytes per token"
date: 2026-08-26
topics: [analysis, serving, memory]
status: analysis
---

One token of context in OPT-13B costs 800 kilobytes of KV cache: two
vectors, 5120 wide, across 40 layers, in FP16. Hold a 2,000-token
conversation and the model is dragging 1.6 gigabytes of state behind
it, per request, before a single new token is computed. The
[PagedAttention paper](https://arxiv.org/abs/2309.06180) (Kwon et
al., SOSP 2023, the paper behind vLLM) starts from that arithmetic,
and the arithmetic is the whole argument: LLM serving looks like a
compute problem and bills like a memory problem.

## The measured indictment

Before proposing anything, the authors measure what existing serving
systems did with that memory, and the number deserves to be quoted
exactly: 61.8 to 79.6 percent of KV cache memory wasted. Only a
fifth to a third of it held actual token state. The waste has three
named sources, and anyone who has run a memory allocator will
recognize all of them: slots reserved for tokens not yet generated,
internal fragmentation from provisioning every request for its
maximum possible length, and external fragmentation from the
allocator itself. Serving systems were burning most of their
scarcest resource on bookkeeping failures that operating systems
classified, named, and solved decades ago.

## The old answer, imported honestly

The fix is virtual memory, applied without embarrassment. Chop the
KV cache into fixed blocks of sixteen tokens, keep a block table per
sequence mapping logical to physical, allocate on demand, share
read-only blocks across sequences, copy on write when a shared block
diverges. Beam search stops copying whole caches and starts sharing
prefixes. Every sentence of that description could have come from an
OS textbook, and the paper is candid about the lineage.

What makes it architecture rather than borrowing is where the cost
went. Contiguity was never a law of nature; it was a contract the
attention kernel imposed on the storage layer. The authors moved
that constraint out of the contract and paid for it inside a
rewritten kernel that walks the block table. Utilization became
near-total, and the price is indirection on every attention read:
20 to 26 percent higher attention-kernel latency by their own
microbenchmark, diluted to little at system level because attention
is one operator among many.
Naming the price is what makes the trade honest, and the outcome at
system level is 2 to 4 times the throughput of FasterTransformer and
Orca at equal latency, widening with longer sequences and larger
models.

![Diagram comparing KV cache layouts. On the left, contiguous per-request allocation shows large hatched regions of waste: reserved slots, internal fragmentation, external fragmentation, totaling 61.8 to 79.6 percent. On the right, paged allocation maps logical blocks through a block table to scattered fixed-size physical blocks of 16 tokens, with near-zero waste and copy-on-write sharing between sequences. A footer records 800 KB per token for OPT-13B and 2 to 4 times throughput at equal latency.](/assets/img/paged-kv.svg)

## What survives the trip to the edge

My lane is the device, so the honest question is which of these
gains travel. The uncomfortable answer: the headline one mostly does
not. vLLM's throughput wins come from packing more concurrent
requests into a fixed pool of GPU memory, and a phone has exactly
one user. Batching gains that justify the indirection in a
datacenter have nothing to batch on-device.

What does travel is the diagnosis. On-device KV cache competes with
the entire OS for RAM, so the fragmentation and over-reservation the
paper measured are, proportionally, a worse crime on an 8-gigabyte
phone than on an 80-gigabyte GPU. Block-based allocation still
belongs there, for different reasons: long contexts that grow without
pre-reserving worst case, and a shared system-prompt prefix kept
once across an app's sessions rather than per conversation. Same
mechanism, different justification. Importing the mechanism without
re-deriving the justification is how edge runtimes end up carrying
datacenter machinery they cannot afford, which is a mistake I intend
to measure rather than assert when the
[benchmark protocol](/log/edge-ai-benchmark-protocol/) gets its
numbers.

## Ledger

This entry ships no code of its own; it is the analysis lane doing
its job. Every number above is from the paper itself, checked
against it today: 800 KB per token, 61.8 to 79.6 percent measured
waste, blocks of sixteen, 20 to 26 percent kernel overhead, 2 to 4
times throughput at equal latency. The Edge AI lane, meanwhile, got
its protocol turned into runnable code, so the only thing still
standing between that lane and numbers is a phone in my pocket.
