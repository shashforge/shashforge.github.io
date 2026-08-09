---
layout: post
title: "Same trace, different guts"
date: 2026-08-09
topics: [agent-harness, refactoring]
status: built
---

There was an `if` in the executor that had no business being an `if`.
Verification failure, a card-carrying member of the failure taxonomy
with its own exception class since
[the spine post](/log/the-harness-gets-a-spine/), was being decided
by a plain conditional. `BudgetExhausted` had it worse: a boolean
return value. The README promises that the taxonomy decides
responses. Inside the executor, a conditional and a boolean were
doing the deciding.

[The lens post](/log/the-lens-not-the-eraser/) ended by admitting two
taxonomy entries sat unemployed, with a note that I'd rather stay
honest than invent work for them. What follows is not invented work.
The refactor was sitting there waiting to be seen: `_act_and_verify`
is now one `try` block, and every branch of control flow is an
`except` clause on an exception from `errors.py`. The permission gate
raises. The budget raises. So, at last, does the verifier. The
dispatch *is* the taxonomy, arm for arm.

## The part that makes this publishable

Rewiring the guts of a state machine is routine work. Proving that
nothing observable moved is the part worth writing down, and the
proof was already sitting in the repo: the golden trace from
[the replay post](/log/replay-is-the-feature/). The shipped
`happy_path.json` pins every transition of the canonical run, in
order, with reasons and fields. If this refactor had shifted a single
edge of the state machine, that test would have failed and named the
transition that moved.

It didn't move. Thirty-one pre-existing tests, untouched, all green,
the golden among them. Building the regression harness before I
needed it felt close to process theater on August 5. Today it turned
a risky internal rewrite into a boring one, which is the highest
compliment a refactoring can receive.

![Diagram of the executor's dispatch after the refactor. One try pipeline runs the permission gate, the call budget, the tool, and the verifier. Five except arms map exceptions to responses: PermissionDenied escalates and never retries, BudgetExhausted escalates, TransientToolError retries the same step, PermanentToolError re-plans, VerificationFailure retries then escalates. A footer notes the shipped golden trace came through unchanged, ten of ten transitions identical.](/assets/img/taxonomy-dispatch.svg)

## One rule got wider, on purpose

Old behavior: the taxonomy applied only when the machinery raised it.
New behavior: it applies no matter who raises it. A tool that throws
`VerificationFailure` from its own body now gets exactly the
treatment a failed verifier gets — retry within budget, then
escalate. The new test has the tool announce "i checked my own work
and it is bad," and the executor answers with two retries and an
escalation, per the contract. Self-rejecting tools were never in the
design. The dispatch handles them anyway, because it no longer cares
where an exception was born.

The call-budget path also picked up its first behavioral test. That
path always worked; nothing pinned it. Now something does.

## Ledger

868 lines, 35 tests, and every leaf of the taxonomy now has a
raiser whose job it is: the budget raises `BudgetExhausted`, the
verifier raises `VerificationFailure`, tools own the transient and
permanent pair, and nothing's fate is decided by a lonely `if`. The
two-day gap before this entry was GitHub's outage sitting on my
deploy queue; the record of that fight lives in the site repo's
commit history, where records belong.
