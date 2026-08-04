---
layout: post
title: "The harness gets a spine"
date: 2026-08-04
topics: [agent-harness, architecture]
status: designed
---

Yesterday I argued the harness is the product. Easy to say. Today I
started paying for it: first design drop of the
[Agent Harness Reference Architecture](/projects/) — the execution state
machine, a failure taxonomy, and a first cut of the tool contract. No
code runs yet, which is why the label on this entry says *designed* and
not *built*.

## Why a state machine and not a loop

Most agent frameworks are a `while` loop with vibes. The loop calls the
model, the model picks a tool, repeat until something looks done. It
works in demos and it's unreplayable in production, because the loop's
actual state lives in a transcript nobody can query.

Fifteen years of embedded work left me with one reflex: you can't debug
what you can't enumerate. So the harness gets an explicit state machine.
When every transition is data, you can store it, replay it, diff two
runs, and answer "why did the agent do that" three weeks later. That
question was the whole point of
[yesterday's essay](/log/the-model-was-never-the-product/).

![State machine: PLAN to ACT to VERIFY to CHECKPOINT, looping while steps remain. Verification failure retries within budget, then escalates to a human outer loop that can steer, resume, or abort.](/assets/img/harness-state-machine.svg)
*Verification is a state, not an afterthought. Checkpoints only happen
after it passes, so replay always restarts from known-good.*

Two decisions worth defending:

**Verify sits between act and checkpoint.** Nothing gets persisted as
progress until it survives verification. A checkpoint of unverified
state is just a saved bug.

**The human is a state, not an exception handler.** Escalation is a
normal transition with normal data attached — what failed, what was
tried, what the budget was. Treating it as a crash path is how you get
agents that fail silently.

## The failure taxonomy

Every failure the executor can see, and what it's allowed to do about it:

| Failure class | Detected by | Response |
|---|---|---|
| Transient tool error | timeout, 5xx | retry same step, backoff |
| Permanent tool error | 4xx, schema mismatch | re-plan with different tool |
| Verification failure | verifier | retry ≤ budget, then escalate |
| Budget exhausted | step / cost meter | escalate to human |
| Permission denied | scope check before call | escalate — never retry |
| Context overflow | token watermark | compact, then re-plan |

The rule hiding in that table: *retries are for the world being flaky;
escalation is for the plan being wrong.* Mixing those two up is where
most agent loops burn their budget.

## Tool contract, first cut

```json
{
  "name": "search_codebase",
  "args_schema": { "$ref": "schemas/search_args.json" },
  "scope": ["repo:read"],
  "idempotent": true,
  "budget": { "timeout_ms": 8000, "max_calls_per_run": 20 },
  "verify": "verifiers/search_nonempty"
}
```

Permissions live on the contract, not in the runtime. The runtime
enforces; the contract declares. That makes a scope change a reviewable
diff in version control instead of a config surprise in production —
same reason capability lists in embedded systems live in the manifest,
not the driver.

## Tomorrow

The executor skeleton, and ADR-001: reference implementation in Python
for iteration speed, or a C++ core from day one? The embedded half of me
has opinions. The part of me that wants to ship this month has different
ones. Should be a fun argument to have with myself in public.

